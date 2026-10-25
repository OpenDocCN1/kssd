
# [原创]学硬核知识-硬件断点

上一篇文章我们介绍了`inlinehook`（修改代码的`hook`方式），接下来准备介绍硬件断点+`veh hook`（无需修改代码的`hook`方式）。作为铺垫，本文先介绍硬件断点。

获取本文的实战代码、参考资料，请关注后，在聊天框回复：硬件断点。

## 硬件断点介绍

硬件断点与软件断点类似，都是一种代码调试的手段，可以让代码中断在需要的地方，方便调试。

软件断点是调试器在断点位置插入 `int 3` 汇编指令实现的。硬件断点顾名思义是依赖硬件`cpu`，主要是靠`dr0~dr7`，8个调试寄存器实现的。

硬件断点比软件断点的功能更强，除了函数断点外，还可以数据断点，可以指定当数据被读或写时中断。

硬件断点的本质就是在指定内存下断点，内存可以位于代码段（函数断点）也可以是数据段（数据断点）。可以设置事件有执行、写入、读写时中断。

调试器使用硬件断点实现数据断点功能（例如`gdb`的`watch`命令）

```
Breakpoint 1, main (argc=0, argv=0x7fffffffe3e0) at xxx.cpp:2727      {(gdb) watch argcHardware watchpoint 2: argc
```

一些游戏外挂也会使用硬件断点来实现`hook`效果，介绍完硬件断点以后讲。

## 调试寄存器

硬件断点是`cpu`提供的功能，主要是与`cpu`上的`dr0~dr7`这8个调试寄存器打交道。下面参考`intel`手册详细介绍一下。

`intel-325462-sdm-vol-1-2abcd-3abcd.pdf`，page 3415

### `dr0`，`dr1`，`dr2`，`dr3`（Debug Address Registers）

`dr0~dr3`是调试地址寄存器，储存4个硬件断点的内存地址。硬件限制所以最多只有四个硬件断点。

### `dr4`，`dr5`（保留，暂时没用）

目前没有用，不说了。

### `dr6`（Debug Status Register）

调试状态寄存器，`dr6`按位使用，我们主要关注`B0~B3`位。触发硬件断点后，会将对应序号位设置为1。

### `dr7`（Debug Control Register）

可以按位设置硬件断点的属性，包括：开关位、条件位、长度位。

#### 开关位

`dr7`的开关位控制`dr0~dr3`号硬件断点是否启用。

#### 条件位

`dr7`的条件位控制`dr0~dr3`如何被触发。`00` 执行时触发。`01`写入时触发，`11`读写时触发。

#### 长度位

`dr0~dr3`指定的内存地址，`dr7`的长度位控制内存长度。

如果`dr7`的条件位设置为`00`执行，则对应的长度位必须是`00`。

## 代码实战

### 设置调试寄存器

`windows`提供了`API`，可以设置和获取寄存器。

```
BOOL SetThreadContext(

    _In_ HANDLE hThread,

    _In_ CONST CONTEXT* lpContext

    );BOOL GetThreadContext(

    _In_ HANDLE hThread,

    _Inout_ LPCONTEXT lpContext

    );
```

`CONTEXT`是一个结构体，里面包含所有寄存器。因为每个`cpu`有一套寄存器，所以`API`里需要传线程句柄，设置哪个线程的寄存器。

### `dr7`辅助类

`dr7`寄存器都是位操作，不方便设置也不方便看。写个辅助类。按位操作的都可以这样来弄，不用来回位移了。

```
// <<intel-325462-sdm>> page 3414// Debug control register (DR7)// Specifies the forms of memory or I / O access that cause breakpoints to be generated.struct xx_dr7 {

  uint32_t L0 : 1;

  uint32_t G0 : 1;

  uint32_t L1 : 1;

  uint32_t G1 : 1;

  uint32_t L2 : 1;

  uint32_t G2 : 1;

  uint32_t L3 : 1;

  uint32_t G3 : 1;

  uint32_t LE : 1;

  uint32_t GE : 1;

  uint32_t no_use1 : 1;

  uint32_t RTM : 1;

  uint32_t no_use2 : 1;

  uint32_t GD : 1;

  uint32_t no_use3 : 2;

  uint32_t RW0 : 2;

  uint32_t LEN0 : 2;

  uint32_t RW1 : 2;

  uint32_t LEN1 : 2;

  uint32_t RW2 : 2;

  uint32_t LEN2 : 2;

  uint32_t RW3 : 2;

  uint32_t LEN3 : 2;};
```

### 我们封装的函数

我们封装一个设置硬件断点的函数，利用前面的`dr7`辅助类。

参数有：线程句柄、硬件断点序号（0~3）、内存地址、事件类型（执行、写、读写）、内存长度。

```
#define RW_EXE    0b00#define RW_WRITE  0b01#define RW_RW    0b11#define LEN_1B    0b00#define LEN_2B    0b01#define LEN_4B    0b11static bool xx_set_hw_bp(HANDLE thread, int idx, void* addr, 

  int RW = RW_EXE, int LEN = LEN_4B){  if (RW == RW_EXE) {    //If the corresponding RWn field in register DR7 is 00 (instruction execution), then the LENn field should also be 00. 

    // The effect of using other lengths is undefined.See Section 17.2.5, “Breakpoint Field Recognition, ” below.

    LEN = 0b00;

  }  // get context

  CONTEXT context = { 0 };

  context.ContextFlags = CONTEXT_DEBUG_REGISTERS;

  BOOL get_ret = GetThreadContext(thread, &context);  if (FALSE == get_ret) {    return false;

  }  // set dr7

  xx_dr7 dr7{ 0 };  if (0 == idx) {    memcpy(&context.Dr0, &addr, sizeof(addr));

    dr7.L0 = 1;

    dr7.G0 = 1;

    dr7.RW0 = RW;

    dr7.LEN0 = LEN;

  }  else if (1 == idx) {    memcpy(&context.Dr1, &addr, sizeof(addr));

    dr7.L1 = 1;

    dr7.G1 = 1;

    dr7.RW1 = RW;

    dr7.LEN1 = LEN;

  }  else if (2 == idx) {    memcpy(&context.Dr2, &addr, sizeof(addr));

    dr7.L2 = 1;

    dr7.G2 = 1;

    dr7.RW2 = RW;

    dr7.LEN2 = LEN;

  }  else if (3 == idx) {    memcpy(&context.Dr3, &addr, sizeof(addr));

    dr7.L3 = 1;

    dr7.G3 = 1;

    dr7.RW3 = RW;

    dr7.LEN3 = LEN;

  }  // set context

  context.Dr7 = dr7.get();

  BOOL set_ret = SetThreadContext(thread, &context);  return TRUE == set_ret;

}
```

代码逻辑不复杂，借助`dr7`辅助类，设置对应的控制位。

下面做一些测试

### `vs`的数据断点

测试一下，`vs`中设置数据断点后，调试寄存器的值是怎样的？

测试代码：

```
void test_vs_data_bp() {  int n = 0;// 对n下数据断点

  CONTEXT context = { 0 };

  context.ContextFlags = CONTEXT_DEBUG_REGISTERS;  auto ret = GetThreadContext(GetCurrentThread(), &context);

  xx_dr7 dr7{ 0 };

  dr7.set(context.Dr7);

}
```

在第3行代码下普通断点停住，再对 `n` 下数据断点。

然后看看调试寄存器的值

借助`dr7`辅助类看看`dr7`情况。

控制位`L0`设置为`0b1`，条件位`RW0`设置为`0b01`（写事件），长度位为`0b11`（`0x3`，4字节）。

### 硬件断点执行事件

我们继续测试一下，执行事件的硬件断点。

```
void test_exe_hw_bp_func() {

  xx_set_hw_bp(GetCurrentThread(),1, &func, RW_EXE);

  func();}
```

执行`func`时触发异常，我们没有设置`veh`处理，就被调试器捕捉到

### 硬件断点写事件

接下来试试写事件，测试代码如下：

```
void test_write_hw_bp() {  int n = 0;

  xx_set_hw_bp(GetCurrentThread(), 0, &n, RW_WRITE);

  n = 1;//write}
```

`n=1`时触发断点

### 硬件断点读写事件

继续试试读写事件，为啥不能设置只读事件？

```
void test_read_hw_bp() {  int n = 0;

  xx_set_hw_bp(GetCurrentThread(), 0, &n, RW_RW);  int b = n;//read

  b = b * b;

  n = 5;

}
```

读事件时中断

写事件时中断

### 硬件断点写事件1Byte

接下来试试，长度控制位。先设置写事件、1字节。再设置写事件、4字节。

```
void test_write_hw_bp_1byte() {

  char c[4];

  xx_set_hw_bp(GetCurrentThread(), 0, c, RW_WRITE,LEN_1B);  c[0] = 0;  c[1] = 0;

  xx_set_hw_bp(GetCurrentThread(), 0, c, RW_WRITE, LEN_4B);  c[0] = 0;  c[1] = 0;

}
```

不贴图了，在执行第5、9、10行时发生了中断。第6行没有中断，因为只设置了1字节。

硬件断点、调试寄存器介绍完了，下次就介绍硬件断点`hook`。

最后，求关注、点赞、转发~

东北码农，全网同名，求关~

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)
