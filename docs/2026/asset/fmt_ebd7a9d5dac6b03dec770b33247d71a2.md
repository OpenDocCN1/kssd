
```c
#define ADDRESS 0x8000
int32_t add(uc_engine *uc, int32_t a, int32_t b) {
    uint32_t r0, lr;

    // 传参数
    uc_reg_write(uc, UC_ARM_REG_R0, &a);
    uc_reg_write(uc, UC_ARM_REG_R1, &b);

    // 根据函数的调用机制，要求我们必需设置一个返回点，这个返回点正是函数执行完毕的标志
    // 由于函数内部会执行到内存中的什么位置我们是不确定的（在我们这个例子中我们当然知道它会执行到哪里）
    // 并且在uc_emu_start()中也有一个停止点，这个停止点非常强硬，如果pc指针到达这个地址程序就会立刻终止
    // 因此这个地址必需是一个目标函数永远不可能执行到的点，而且这个地址又必需是在已映射的内存范围内
    // 在我这个例子中add函数永远不可能执行到ADDRESS地址，所以我将停止点设置成了ADDRESS，因此当add函数内部经由bx lr返回后
    // pc指针将会到达uc_emu_start()设置的停止点，模拟器才能停止运行，回到我们的代码
    lr = ADDRESS;
    uc_reg_write(uc, UC_ARM_REG_LR, &lr);

    // 在unicorn 1.0.2之前uc_emu_start()在特殊情况下不会在pc==stopAddr时立即停止
    uc_emu_start(uc, 0x8010, lr, 0, 0);

    uc_reg_read(uc, UC_ARM_REG_R0, &r0); // 获取返回值
    return r0;
}

int main() {
    uc_engine *uc;
    uint32_t code[] = {0xE1A0200F, 0xE2823008, 0xE2824010, 0xE12FFF14, 0xE0800001, 0xE12FFF1E, 0xE3A0000B, 0xE3A01016, 0xE12FFF33, 0xE3A01021, 0xE12FFF33, 0xE1A00000};

    uc_open(UC_ARCH_ARM, UC_MODE_ARM, &uc);
    uc_mem_map(uc, ADDRESS, 1024 * 4, UC_PROT_ALL);
    uc_mem_write(uc, ADDRESS, code, sizeof(code));
    printf("%d\n", add(uc, 24, 37)); // 输出61
    printf("%d\n", add(uc, 86, 753)); // 输出839
    uc_close(uc);
    return 0;
}
```

## 拦截函数调用

上一个案例讲述了由我们调用目标函数的方法，实际上目标函数内部是有可能会有一些系统调用的，系统调用正是双向通信中对方调用我们，该怎么做呢？

还是以之前的函数案例来研究，我们发现在`0x8020`和`0x8028`处都调用了`add`函数，这个函数的地址我们是知道的，如果能实现把`add`函数调用替换成调用我们另外实现的一个`add`函数，那不就相当于目标程序调用了系统函数？

CPU是按照`PC`寄存器指向的地址来执行代码的，因此当`PC`寄存器指向`add`函数的地址时，就是我们下手的时机

这里需要增加一个新的unicorn API调用`uc_hook_add()`，通过这个API实现控制目标程序的各种行为，比如内存读写、指令执行，以及获取unicorn本身的运行状态

```c
#define ADDRESS 0x8000

void add(uc_engine *uc) {
    int32_t a, b, ret;
    uint32_t lr;

    // 获取参数值
    uc_reg_read(uc, UC_ARM_REG_R0, &a);
    uc_reg_read(uc, UC_ARM_REG_R1, &b);

    ret = a + b + 1;

    // 设置返回值
    uc_reg_write(uc, UC_ARM_REG_R0, &ret);

    // 模拟实现bx lr的功能
    uc_reg_read(uc, UC_ARM_REG_LR, &lr);
    uc_reg_write(uc, UC_ARM_REG_PC, &lr);
}

void hook(uc_engine *uc, uint64_t address, uint32_t size, void *user_data) {
    if (0x8010 == (uint32_t)address) { // 当模拟器内执行到add函数地址时，进入我们的add函数进行处理
        add(uc);
    }
}

int main() {
    uc_engine *uc;
    uc_hook hh;
    uint32_t r0;

    uint32_t code[] = {0xE1A0200F, 0xE2823008, 0xE2824010, 0xE12FFF14, 0xE0800001, 0xE12FFF1E, 0xE3A0000B, 0xE3A01016, 0xE12FFF33, 0xE3A01021, 0xE12FFF33, 0xE1A00000};

    uc_open(UC_ARCH_ARM, UC_MODE_ARM, &uc);
    uc_mem_map(uc, ADDRESS, 1024 * 4, UC_PROT_ALL);
    uc_mem_write(uc, ADDRESS, code, sizeof(code));

    // 这里我在整个代码地址范围内加上单条指令的hook，每次执行这个地址范围内的指令前都会回调我们的hook函数
    // 如果你可以很明确的知道在哪个地址范围内需要hook，设置一个准确的地址范围能提升程序的运行效率
    uc_hook_add(uc, &hh, UC_HOOK_CODE, hook, NULL, ADDRESS, ADDRESS + sizeof(code));

    uc_emu_start(uc, ADDRESS, ADDRESS + sizeof(code), 0, 0);

    uc_reg_read(uc, UC_ARM_REG_R0, &r0);
    printf("r0 = %d\n", r0);
    uc_close(uc);
    return 0;
}
```

由于我们实现的`add`函数每次都会额外`+1`，所以原本的结果应该是66现在变成了68

之前：`(11+22)+33=66`

之后：`(11+22+1)+33+1=68`

## 可变参数的函数调用

最常见的可变参数函数是`printf()`，如果模拟器内的代码需要调用这样的函数我们实现起来是比较复杂的，一种比较简单的方式是写一个转换函数编译成机器码加载进模拟器，当需要调用这种函数时跳转到转换函数去执行，转换函数将参数处理后转换成固定参数的函数调用，以此实现简化处理

我在遇到这个问题时采用的就是这种策略，将 4a4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6E0M7r3q4D9j5h3&6V1i4K6u0r3M7s2u0A6L8Y4c8X3 编译后加载进模拟器内，这样无论`printf()`的用法有多复杂，我只需要实现`putchar()`就能实现字符串输出功能

## 栈内存

在前面的函数案例中因为是故意设计的简单函数，因此并没有涉及到栈的操作，在实际应用中一个功能稍微复杂点的函数都一定会用到栈内存，什么是栈内存呢？

栈是在CPU硬件级别上直接支持的一种数据结构和算法，通常至少由`push`/`pop`指令和栈指针寄存器组成，实现一种简单的内存管理机制，在编程时函数内的临时变量正是由这一机制自动分配和回收内存。

栈在使用时的表现类似于往一个桶里面放东西，后面放进去的东西总是最先拿出来，在程序中`push a, push b`后第一次`pop`拿出来的是`b`的数据，第二次`pop`拿出来的才是`a`的数据，无论是`push`还是`pop`指令都是往一个内存地址读写数据，因此需要有一个寄存器来保存这个内存地址，这个寄存器就是栈指针寄存器`sp`，每条`push`和`pop`指令都会自动修改`sp`寄存器的值，因此初始化一个栈空间实际上就是设置`sp`寄存器的初始值。

栈的实现方式不同又可以分为不同的类型，在`push`/`pop`执行前`sp`指向的内存有没有存放数据，以此可以分为两种：

- 空栈-先放数据再调指针

- 满栈-先调指针再放数据

然后在`push`/`pop`执行后`sp`指向的地址是增加还是减少，又可以分为递增/递减两种，由此可以组合出四种栈类型：

- 满递减FD

- 空递减ED

- 满递增FA

- 满递增FA

必需要确认程序中使用的是哪一种栈类型，合理的设置`SP`寄存器初始化栈内存程序才能正确运行，在我的模拟器中使用的是满递减类型：

```c
#define ADDRESS         0x8000
#define TOTAL_MEMORY    1024 * 4
#define STACK_ADDRESS   (ADDRESS+TOTAL_MEMORY)  // 栈的开始地址放在代码块之后（可以是任意未使用的内存空间）
#define STACK_SIZE      1024 * 4   // 栈内存的大小根据需要进行调整

uc_mem_map(uc, ADDRESS, TOTAL_MEM, UC_PROT_ALL);
uc_mem_write(uc, ADDRESS, code, sizeof(code));

// 设置栈
uc_mem_map(uc, STACK_ADDRESS, STACK_SIZE, UC_PROT_ALL);
uint32_t value = STACK_ADDRESS + STACK_SIZE; // 由于是满递减类型，因此sp初始化为栈空间的最末尾
uc_reg_write(uc, UC_ARM_REG_SP, &value);

// ... 之后就可以进行函数调用了
```

其中栈内存的大小是可以任意调整的，决定了函数中局部变量最多能使用多少内存，通常如果局部变量少那么几k内存也够

## 内存管理(堆内存)

内存管理是一个重点知识，因为在C语言中`malloc()`和`free()`的地位非常高，我们与模拟器中的函数通信时像`int`之类的基本数据类型是可以直接传递的，因为它们都是值复制形式传递，但是一旦出现字符串、数组、结构体就无能为力了，因为它们传递的都是指针，实际的数据要求我们先放入一块内存中，我们可以通过`uc_mem_map()`再申请一块内存，然后用`uc_mem_write()`把数据写进去，这样做的确可以实现预期的效果，但是有两个弊端：

1. `uc_mem_map()`要求内存必需4k对齐，也就是说哪怕你只需要几个字节它也至少会分配4k内存。

2. `uc_mem_map()`和`uc_mem_write()`都必需手动管理内存地址和长度，使用起来极其不方便。

unicorn给我们提供了另一个分配内存的方式：

```c
uc_err uc_mem_map_ptr(uc_engine *uc, uint64_t address, size_t size, uint32_t perms, void *ptr);
```

这个api与`uc_mem_map()`相比只增加了最后的`ptr`参数，这使我们可以直接将一块存有数据的内存映射到模拟器中，省去了`uc_mem_write()`的操作，似乎这个方法要比之前的要方便，但实际上它同样要求4k对齐和手动管理内存地址。

在arm和x86都是通过内存地址访问内存，我们编程操作内存的方式是完全相同的，唯一的区别可能就是大小端字节序的问题，如果字节序不同，写一个转换程序转换一下就行了，在我们这个模拟器里无论是本机还是模拟器都是小端模式，因此两者在内存中的数据是完全相同的。

```c
#define ADDRESS            0x8000
#define TOTAL_MEMORY       1024 * 1024 * 4

uint8_t *mem = malloc(TOTAL_MEMORY); // 模拟器的全部内存
uc_mem_map_ptr(uc, ADDRESS, TOTAL_MEMORY, UC_PROT_ALL, mem);
```

采用这种方式给模拟器初始化内存后，我们对`mem`的读写会直接影响到模拟器里面运行的程序，同时模拟器里面的程序对内存的读写也会立刻影响到本机代码，也就是说模拟器里外都是同一片内存。

这里面唯一需要注意的是内存地址的区别，在模拟器里面这块内存的首地址是我们设置的`0x8000`，而在模拟器外面它是由系统分配的一个地址，因此如果直接拿模拟器返回的地址当指针用的话一定是会出错的，把地址直接传递给模拟器在里面也是无法使用的，所以在与模拟器内部通信时必需要经过一次地址转换：

```c
// 模拟器内部地址转换成本地指针
void *toPtr(uint32_t addr) {
    return mem + (addr - ADDRESS);
}

// 本地指针转换成模拟器内部地址
uint32_t toAddr(void *ptr) {
    return ((uint8_t *)ptr - mem) + ADDRESS;
}
```

前面说了`uc_mem_map_ptr()`仍然有许多限制，因此要想方便的使用`malloc()`和`free()`必需要自己实现`malloc()`和`free()`，下面是一个超小的内存管理器代码：
