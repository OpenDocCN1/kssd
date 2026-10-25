

# [原创] AArch64函数栈的分配,指令生成与GCC实现(上)

本篇主要介绍原理，源码分析见 [AArch64函数栈的分配,指令生成与GCC实现(下)](https://bbs.kanxue.com/elink@3faK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8X3I4A6k6r3q4F1x3e0p5K6L8r3W2V1j5h3&6Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8U0p5J5x3K6V1$3x3e0V1#2y4l9`.`.)

# 一、术语定义

为便于理解，本文中后续术语含义解释如下:

1. 首地址: 一个变量/内存的首地址指的是此变量/内存最低位的地址

2. 尾地址: 一个变量/内存的尾地址指的是此变量/内存最高地址(+1), 如变量存储空间为[0x0, 0x10), 则此变量首地址为0x0, 尾地址为0x10

3. GPR/FPR: AArch64中的通用寄存器(`R0`-`R30`)后面称为GPR(General Purpose Register); 浮点寄存器称为FPR(Floating Point Register)

# 二、AArch64函数栈的组成

aarch64下函数栈是向低地址方向增长, 按照由高=>低的顺序其函数栈可以包含如下区域:

```
/* 从高地址=>低地址, 这些区域包括:

  * incoming stack: 传入栈参数区域 (这段区域实际上属于caller函数, 在callee中作为栈参数使用)

  * varargs: 匿名寄存器参数区域

  * local variables: 局部变量区域

  * callee-saved regs: callee-saved 寄存器存储区域

  * dynamic allocation: 动态栈分配区域

  * outgoing stack: 传出栈参数区域

*/

+-------------------------------+ <- highmem

|                               |

|  incoming stack arguments     |

|                               |

+-------------------------------+ <-- incoming stack (aligned)/caller sp

|                               | 

|  callee-allocated save area   |

|  for register varargs         |

|                               |

+-------------------------------+ 

|  local variables              | 

|                               |

+-------------------------------+

|  padding                      | \

+-------------------------------+  |

|  callee-saved registers       |  | frame.saved_regs_size

+-------------------------------+  |

|  LR'                          |  |

+-------------------------------+  |

|  FP'                          | /

+-------------------------------+ <-- 硬件寄存器fp(x29)指向这里(若有)

|  dynamic allocation           |

+-------------------------------+

|  padding                      |

+-------------------------------+

|  outgoing stack arguments     |

|                               |

+-------------------------------+ <-- 硬件寄存器sp指向这里/callee sp

|                               | 

    				| <- lowmem
```

虽然函数栈整体是由高地址=>低地址增长的，但每个区域内部有自己的变量分配顺序。这些区域的主要作用为:

### 1. incoming stack arguments

即**传入栈参数区域, 内部元素分配方向:** **低地址 => 高地址**。

严格来讲这段区域是caller分配的, 并不属于callee栈帧的一部分, 当caller调用callee时, callee 通过incoming 区域接收caller的参数。

对于caller来说这段区域称为outgoing stack argument(传出栈参数区域), 一个函数的 incoming 区域首地址等于其父函数的outgoing 区域的首地址。

### 2. callee-allocated save area for register varargs

即**匿名寄存器参数区域, 内部元素分配方向:** **低地址 => 高地址**。

对于可变参函数，此区域用来保存函数中可能用到的所有匿名寄存器参数，`va_xxx`系列函数的参数解析依赖于此区域。

从此区域开始(以及向低地址方向的内存)均为callee分配, 此区域按照高地址=>低地址实际上分为GPRs/FPRs两个子区域; 每个区域内部元素分配方向均是低地址=>高地址;

### 3. local variables

即**局部变量区域, 内部元素分配方向: 高地址 => 低地址**。

函数中显示定义的变量，编译器内部生成的临时变量均存放在此区域中。

需要注意的是，编译优化可能导致变量分配顺序与源码中定义顺序不同，在不考虑优化的情况下其内部元素分配方向默认是高地址=>低地址。

### 4. callee-saved registers

即**callee-saved寄存器区, 内部元素分配方向:** **低地址 => 高地址**。

在AAPCS64标准中[3]:

- [`R19`, `R28`] 是callee-saved registers, 这些寄存器在函数调用结束后要保持原始值，如果callee中修改了这些寄存器，则在返回前必须恢复其原始值。

- `LR`(`R30`), `FP`(`R29`) 并非callee-saved寄存器, 但由于操作流程类似, 后续放在callee-saved寄存器中一并讨论。

### 5. dynamic allocation

即**动态栈分配区, 内部元素分配方向: 高地址 => 低地址**。

当函数中调用`alloca`时会在栈中dynamic allocation区域分配需内存，这部分区域在编译期间不存在, 在运行期间大小可以随执行变化。

### 6. outgoing stack arguments

即**栈参数传出区域**, **内部元素分配方向:** **低地址=> 高地址**;

当caller需要通过栈向子函数传递参数(如大于8个GPR参数)时, 栈参数保存在caller的outgoing区域。

一个函数(作为caller)其outgoing区域的大小是固定的，取决于其调用的callees中使用了最多栈参数的那个函数。当函数中存在动态分配时，其outging区域的首地址是可能动态变化的，但总是等于当前硬件寄存器`sp`的值, caller调用callee时也是通过此`sp`将其outgoing区域首地址传递给callee(作为callee的incoming区域首地址)。

# 三、incoming/outgoing区域与栈参数

按照AAPCS64标准，以通用寄存器(GPRs)传参为例(不考虑浮点参数和结构体参数等) :

- 若一个函数调用的参数<=8个则参数会通过`R0`-`R7`传入

- 对于>8的参数，则会通过栈传递

如以下函数:

```c
#define weak __attribute__((weak))



weak int func1(int x0, int x1, int x2, int x3, int x4, int x5, int x6, int x7, int x8, int x9)

{

        printf("func1: incoming:%p ,%p\n",  &x8, &x9);

}



weak int func2(int x0, int x1, int x2, int x3, int x4, int x5, int x6, int x7, int x8, int x9, int x10)

{

        printf("func2: incoming:%p ,%p, %p\n",  &x8, &x9, &x10);

}



int main(void)

{

   int x8 = 8;

   int x9 = 9;

   int x10 = 10;

   register void * sp asm("sp");

   printf("main1: sp:%p, fp:%p\n", sp, __builtin_frame_address(0));

   func1(0, 1, 2, 3, 4, 5, 6, 7, x8, x9);

   func2(0, 1, 2, 3, 4, 5, 6, 7, x8, x9, x10);

}



//aarch64-linux-gnu-gcc main.c -O2 -o main

tangyuan@ubuntu:~/tests/gcc/aarch64_stack/caller_callee_parm_test$ ./main

main1: sp:0x40007ff830, fp:0x40007ff850

func1: incoming:0x40007ff830 ,0x40007ff838

func2: incoming:0x40007ff830 ,0x40007ff838, 0x40007ff840
```

其中:

1. `func1`/`func2`的栈参数来自其caller(`main1`)的函数栈

2. 两次函数调用的前两个寄存器参数在栈中的地址是相同的, 其中:

- 第8个参数(`x8`) 总是在地址`0x40007ff830`

- 第9个参数(`x9`) 总是在地址`0x40007ff838`

在incoming/outgoing区域, 栈参数是基于callee入口时`sp`向高地址方向寻址的, **正是由于参数首地址相同且ABI规则相同(AAPCS64), caller和callee之间才能正确的传递栈参数**.

以上代码的函数栈如图:

![](https://bbs.kanxue.com/upload/attach/202204/490870_57B6QBD9MCMYM4Z.jpg)

这里需要注意的是:

- caller函数在编译期间需要一个固定大小的outgoing区域(否则栈生成逻辑、性能上都会有影响), 一个caller可能调用多个callee, 故对于**caller来说其outgoing区域的大小要以占用outgoing区域最大的那个子函数为准**(细节可见源码分析)。 如上面`func1`中需要存储2个栈参数，`func3`中需要存储3个栈参数，则在caller中需要分配(对齐后)`0x20`大小的空间用来存储outgoing的参数。

- caller在编译期间总是可以确定其需要向每一个调用的callee传递多少个参数，不论callee是否为可变参函数, 因此caller在编译期间即可确定outgoing区域大小。

通过汇编代码可以更好的理解outgoing/incoming区域:

```
.LC2:

        .string "main1: sp:%p, fp:%p\n"

        .......

main:

        sub     sp, sp, #64					## main函数整个栈空间 64 byte，包括局部变量和outgoing区域

        adrp    x0, .LC2					## printf字符串参数

        mov     x1, sp

        add     x0, x0, :lo12:.LC2

        stp     x29, x30, [sp, 32]

        add     x29, sp, 32

        mov     x2, x29

        stp     x19, x20, [sp, 48]

        bl      printf

        mov     w19, 9

        mov     w20, 8

        str     w20, [sp]					## func1的第一个栈参数x8保存到 *sp中,sp为当前outgoing首地址

        str     w19, [sp, 8]				## func1的第二个栈参数x9保存到 *[sp+8]中, outgoing中第二个参数地址

        mov     w7, 7						## 其他为寄存器传参

        mov     w6, 6

        mov     w5, 5

        mov     w4, 4

        mov     w3, 3

        mov     w2, 2

        mov     w1, 1

        mov     w0, 0

        bl      func1						## call func1

        mov     w0, 10

        str     w20, [sp]					## func2的第一个栈参数x8保存到 *sp中, sp为当前outgoing首地址

        str     w19, [sp, 8]				## func2的第二个栈参数x9保存到 *(sp+8)中

        mov     w7, 7

        str     w0, [sp, 16]				## func2的第三个栈参数x10保存到 *(sp+16)中

        mov     w6, 6						## 其余为寄存器参数

        mov     w5, 5

        mov     w4, 4

        mov     w3, 3

        mov     w2, 2

        mov     w1, 1

        mov     w0, 0

        bl      func2						## call func2

        mov     w0, 0

        ldp     x29, x30, [sp, 32]

        ldp     x19, x20, [sp, 48]

        add     sp, sp, 64

        ret



func1:

        stp     x29, x30, [sp, -16]!	    ## func1自身栈大小16 byte,用来保存callee-saved寄存器

        adrp    x0, .LC0

        add     x0, x0, :lo12:.LC0

        mov     x29, sp 

        add     x2, sp, 24					## callee的incoming = callee入口时的sp, 由于函数入口时已经做了sp-=16,

											## 所以此时callee第二个栈参数来自 *(sp + 24), 也就是*(incoming + 8)

        add     x1, sp, 16					## callee的第一个栈参数来自 *(sp + 16), 也就是 *(incoming)

        bl      printf

        ldp     x29, x30, [sp], 16

        ret



func2:

        stp     x29, x30, [sp, -16]!

        adrp    x0, .LC1

        add     x0, x0, :lo12:.LC1

        mov     x29, sp 

        add     x3, sp, 32					## func2同理,第一个栈参数来自 *(sp + 16), 第二个来自 *(sp+24)

            								## 第三个来自 *(sp+32)

        add     x2, sp, 24

        add     x1, sp, 16

        bl      printf

        ldp     x29, x30, [sp], 16

        ret
```

# 四、可变参函数与匿名寄存器参数的存储

## 1. 可变参函数

可变参函数指的是一个可以接受可变个参数的函数, 调用此函数时**只有caller知道为此函数传入了多少个参数, 可变参函数callee只知道caller最少传入了多少个参数**:

- callee中可以确定的参数称为命名参数(Named arguments)

- callee中不确定的参数称为匿名参数(Anonymous arguments)

对于可变参函数(callee), 其需要:

