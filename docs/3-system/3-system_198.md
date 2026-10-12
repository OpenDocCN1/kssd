
# SEH 分析笔记（X86 篇）

> 原文：[`www.pediy.com/kssd/pediy12/140970.html`](https://www.pediy.com/kssd/pediy12/140970.html)

**历史：**

- `v1.0.0`, 2011-10-05：最初版本。
- `v1.0.1`, 2011-10-06：补充总结之前的资源泄漏示例。
- `v1.0.2`, 2011-10-31：调整若干描述语句；增加 `ExceptionNestedException` 返回值的补充说明。

**注**：本帖内容没有同步更新，请下载附件。

\[不介意转载，但请注明出处 `boxcounter.com`。附件里有本文的原始稿，一样的内容，更好的高亮和排版。分别是 `html` 和 `rtl` 格式。后面的部分代码可能会因为自动换行变得很乱，需要的朋友手动复制到自己的代码编辑器就可以正常显示了\]

这两天琢磨了下 `SEH`，这里记录一下自己琢磨的一些心得。

`SEH` 这个概念我就不嗦了，任何一个介绍 `SEH` 的资料都有讲。我主要记录一些自己的理解。可能有一些概念理解的不够清晰，有一些说法比较狭隘，欢迎看到本文的朋友一起讨论、修正，非常感谢。

首先，`SEH` 是针对于异常的一种处理机制，这个异常分为硬件异常和软件异常，这里所说的硬件异常是狭义的异常，也就是 `CPU` 产生的异常。比如除零操作，`CPU` 执行除零操作时候，会自主启动异常处理机制。软件异常，就是程序模拟的异常，比如调用 `RaiseException` 函数。软件异常是可以随意触发的，`windows` 系统内部遇到问题会触发，开发人员高兴了也可以触发。

抛出了问题，就要有解决方案。那这么多问题和解决方案，如何管理呢？`windows` 系统当仁不让的提供了它管理方案 `SEH`。我看一些资料有详细的讨论 `SEH` 的确切含义，这里我不参与讨论，而只是简单的理解为“系统提供的异常处理机制，以及编译器对其进行增强的部分”。

### 来说说系统提供的异常处理机制

`windows` 提供的异常处理机制实际上只是一个简单的框架，一般情况下开发人员都不会直接用到。咱通常所用的异常处理（比如 `C++` 的 `throw`、`try`、`catch`）都是编译器在系统提供的异常处理机制上进行加工了的增强版本。这里先抛开增强版的不提，继续说原始版本。

原始版本的机制很简单：谁都可以触发异常，谁都可以处理异常（只要它能看得见）。但是不管是触发还是处理都得先登记。系统把这些登记信息保存在一个链表里，并且这个链表保存在线程的数据结构里。也就是说，异常所涉及的一些行为都是线程相关的。比如，线程 `T1` 触发的异常就只能由线程 `T1` 来处理，其他线程根本就不知道 `T1` 发生了什么事，更不会狗拿耗子。

等登记完毕后，线程就可以抛出或处理异常了，系统也可以做相应的管理工作了。（这里嗦一句，系统提供的 `SEH` 其实是一个针对于“触发异常-解决异常”的管理机制，系统自身是不提供任何具体异常的解决方案的。解决方案还是要由用户自身来提供（增强版里编译器也会来提供解决方案，来帮“不负责”的程序猿擦屁股，这是后话））

系统提供的管理工作简单来说包括（但不限于）：找到触发异常的线程的异常处理链表（前头登记的那个），然后按照规则（具体的规则后续再说）对该异常进行分发，根据分发后的处理结果再进行下一步的分发或者结束处理。

### 系统管理所使用的数据结构和宏

```c
#define EXCEPTION_CHAIN_END ((struct _EXCEPTION_REGISTRATION_RECORD * POINTER_32)-1)

typedef enum _EXCEPTION_DISPOSITION {
    ExceptionContinueExecution,
    ExceptionContinueSearch,
    ExceptionNestedException,
    ExceptionCollidedUnwind
} EXCEPTION_DISPOSITION;

typedef struct _EXCEPTION_RECORD {
    DWORD ExceptionCode;
    DWORD ExceptionFlags;
    struct _EXCEPTION_RECORD *ExceptionRecord;
    PVOID ExceptionAddress;
    DWORD NumberParameters;
    ULONG_PTR ExceptionInformation[EXCEPTION_MAXIMUM_PARAMETERS];
} EXCEPTION_RECORD;

typedef EXCEPTION_RECORD *PEXCEPTION_RECORD;

typedef
EXCEPTION_DISPOSITION
(*PEXCEPTION_ROUTINE) (
    IN struct _EXCEPTION_RECORD *ExceptionRecord,
    IN PVOID EstablisherFrame,
    IN OUT struct _CONTEXT *ContextRecord,
    IN OUT PVOID DispatcherContext
    );

typedef struct _EXCEPTION_REGISTRATION_RECORD {
    struct _EXCEPTION_REGISTRATION_RECORD *Next;
    PEXCEPTION_ROUTINE Handler;
} EXCEPTION_REGISTRATION_RECORD;

typedef EXCEPTION_REGISTRATION_RECORD *PEXCEPTION_REGISTRATION_RECORD;
```

其中 `EXCEPTION_REGISTRATION_RECORD` 结构就是登记信息。来介绍下它的成员：

1. `EXCEPTION_REGISTRATION_RECORD::Next` 域指向下一个 `EXCEPTION_REGISTRATION_RECORD`，由此构成一个异常登记信息（从字面上说，应该叫做“异常注册记录”更恰当）链表。链表中的最后一个结点会将 `Next` 置为 `EXCEPTION_CHAIN_END`，表示链表到此结束。
2. `EXCEPTION_REGISTRATION_RECORD::Handler` 指向异常处理函数。

前面有简单的说过原始版本 `SEH` 的管理工作，这里再根据以上列出的相关数据结构稍微详细一点说说。

当接收到异常后，系统找到当前线程（还记不记得，前面有说过，异常是线程相关的。系统接收到的异常就是当前正在运行的线程触发的。其实这个说法还不准确，`DPC` 也会触发异常，而它是线程无关的，这里为了方便理解，先只考虑线程）的异常链表，从链表中的第一个结点开始遍历，找到一个 `EXCEPTION_REGISTRATION_RECORD` 就调用它的 `Handler`，并把该异常（由第一个类型为 `EXCEPTION_RECORD` 的参数表示）传递给该 `Handler`，`Handler` 处理并返回一个类型为 `EXCEPTION_DISPOSITION` 的枚举值。该返回值指示系统下一步该做什么：

- `ExceptionContinueExecution` 表示：“我已修正了此异常的故障，请你从事发点重新执行，谢谢”。
- `ExceptionContinueSearch` 表示：“我没有处理此异常，请你继续搜索其他的解决方案，抱歉”。
- `ExceptionNestedException` 和 `ExceptionCollidedUnwind` 这里先不做解释，后面会细说。

这样系统根据不同的返回值来继续遍历异常链表或者回到触发点继续执行。

需要说明一下，本文主要以内核模式下的异常来说，因为相比用户模式下的异常处理流程，内核模式少了模式切换、栈切换以及反向回调等步骤。

我们现在来看看详细的内核异常流程。

首先，`CPU` 执行的指令触发了异常，`CPU` 改执行 `IDT` 中 `KiTrap??`，`KiTrap??` 会调用 `KiDispatchException`。该函数原型如下：

```c
VOID
KiDispatchException (
    IN PEXCEPTION_RECORD ExceptionRecord,
    IN PKEXCEPTION_FRAME ExceptionFrame,
    IN PKTRAP_FRAME TrapFrame,
    IN KPROCESSOR_MODE PreviousMode,
    IN BOOLEAN FirstChance
    );
```

其名称明白的说明了函数的主要功能：分派异常。其实现可以参考 `$wrk-v1.2\\base\\ntos\\ke\\i386\\exceptn.c:1033`。我的笔记：

在当前栈中分配一个 `CONTEXT`，调用 `KeContextFromKframes` 初始化它。

检查 `ExceptionRecord->ExceptionCode`，如果：
- 是 `STATUS_BREAKPOINT`，那么将 `CONTEXT::Eip` 减一；
- 是 `KI_EXCEPTION_ACCESS_VIOLATION`，那么将检查是否是由 `AtlThunk` 触发（这个小环节没有深究），如果是触发 `NX`（不可执行），那么将 `ExceptionRecord->ExceptionInformation[0]` 置为 `0`（貌似表示触发操作的类型，`0` 表示读、`1` 表示写）；
  - 如果 `PreviousMode` 是 `KernelMode`，那么，
    - 如果 `FirstChance` 为 `TRUE`，那么将该异常传达给内核调试器，如果内核调试器没有处理，那么调用 `RtlDispatchException` 进行处理。
    - 如果 `FirstChance` 为 `FALSE`，那么再次将该异常传达给内核调试器，如果内核调试器没有处理，那么 `BUGCHECK`。
  - 如果 `PreviousMode` 是 `UserMode`，那么，
    - 如果 `FirstChance` 为 `TRUE`，那么将该异常传达给内核调试器，如果内核调试器没有处理，那么将异常传达给应用层调试器。如果仍然没有处理，那么将 `KTRAP_FRAME` 和 `EXCEPTION_RECORD` 拷贝到 `UserMode` 的栈中，并设置 `KTRAP_FRAME::Eip` 设置为 `ntdll!KiUserExceptionDispatcher`，返回（将该异常交由应用层异常处理程序进行处理）。
    - 如果 `FirstChance` 为 `FALSE`，那么再次将异常传达给应用层调试器，如果仍然没有处理，那么调用 `ZwTerminateProcess` 结束进程，并 `BUGCHECK`。

抛开应用层异常不说，我们来看 `PreviousMode` 是 `KernelMode` 的情况，其重点是调用 `RtlDispatchException` 的操作。我们来看一下这个函数：

```c
BOOLEAN
RtlDispatchException (
    IN PEXCEPTION_RECORD ExceptionRecord,
    IN PCONTEXT ContextRecord
    );
```

它的实现可以参考 `$wrk-v1.2\\base\\ntos\\rtl\\i386\\exdsptch.c:126`。我的笔记：

遍历当前线程的异常链表，挨个调用 `RtlpExecuteHandlerForException`，`RtlpExecuteHandlerForException` 会调用异常处理函数。再根据返回值做出不同的处理：

- 对于 `ExceptionContinueExecution`，结束遍历，返回。（对于标记为 `EXCEPTION_NONCONTINUABLE` 的异常，会调用 `RtlRaiseException`。）
- 对于 `ExceptionContinueSearch`，继续遍历下一个结点。
- 对于 `ExceptionNestedException`，则从指定的新异常继续遍历。
- 只有正确处理 `ExceptionContinueExecution` 才会返回 `TRUE`，其他情况都返回 `FALSE`。

在继续讲述异常处理机制之前，咱们需要先来认识一下异常链表。

之前有提到过：系统将异常链表头保存在线程结构里。来看看具体的数据结构：

线程的内核数据结构体现是 `_ETHREAD`，从它开始进入，直到咱们关注的异常链表。

```console
kd> dt _ETHREAD
ntdll!_ETHREAD
   +0x000 Tcb              : _KTHREAD
   ... 省略之后的成员

kd> dt _KTHREAD
ntdll!_KTHREAD
   ... 省略的域成员
   +0x074 Teb              : Ptr32 Void
   ... 省略的域成员
```

`Teb` 成员的类型实际是 `_TEB`，来看看：

```console
kd> dt _TEB
ntdll!_TEB
   +0x000 NtTib            : _NT_TIB
   ... 省略的域成员

kd> dt _NT_TIB
ntdll!_NT_TIB
   +0x000 ExceptionList    : Ptr32 _EXCEPTION_REGISTRATION_RECORD
   +0x004 StackBase        : Ptr32 Void
   +0x008 StackLimit       : Ptr32 Void
   +0x00c SubSystemTib     : Ptr32 Void
   +0x010 FiberData        : Ptr32 Void
   +0x010 Version          : Uint4B
   +0x014 ArbitraryUserPointer : Ptr32 Void
   +0x018 Self             : Ptr32 _NT_TIB
```

`_NT_TIB` 的第一个域成员 `ExceptionList` 就是异常链表头。

但是系统不是这么一步一步找的，而是借助 `FS` 寄存器来加速寻找。先来说说系统对 `FS` 的使用。

在应用层，`FS` 寄存器“指向”当前执行线程的 `_TEB` 结构体。在内核层，`FS` 寄存器“指向”另一个跟 `CPU` 相关的结构体：`_KPCR`，来看看它的结构：

```console
nt!_KPCR
   +0x000 NtTib            : _NT_TIB
   +0x000 Used_ExceptionList : Ptr32 _EXCEPTION_REGISTRATION_RECORD
   ... 省略的域成员
```

与 `_TEB` 一样，它的第一个域成员也是 `_NT_TIB`，只不过此时是 `nt!_NT_TIB`，而在应用层是 `ntdll!_NT_TIB`，但它们的结构是一样的。

这样，不论在应用层还是在内核层，系统都可以使用 `FS:[0]` 找到异常链表。

### 总结硬件异常处理流程

到这里，咱们已经聊完了 `CPU` 触发的异常的处理流程，总结一下它的调用流程：

`CPU` 检测到异常 -> `KiTrap??` -> `KiDispatchException` -> `RtlDispatchException` -> `RtlpExecuteHandlerForException`

这是硬件异常，咱们再来看看软件异常。

### 软件异常

软件异常跟硬件异常的处理流程非常接近，只有触发点的不同，调用流程是：

`RtlRaiseException` -> `RtlDispatchException` -> `RtlpExecuteHandlerForException`

后面两个被调用的函数咱已经聊过了，主要来看看 `RtlRaiseException`。这个函数从其名字上就能看出是用来触发异常的。原型如下：

```c
VOID
RtlRaiseException (
    IN PEXCEPTION_RECORD ExceptionRecord
);
```

其实现可以参考 `$wrk-v1.2\\base\\ntos\\rtl\\i386\\raise.asm:71`。我的笔记：

`RtlRaiseException` 首先调用 `RtlDispatchException` 分发异常，如果 `RtlDispatchException` 成功分发（有处理函数处理了这个异常），那么结束本函数。

如果没有成功分发，那么调用 `ZwRaiseException` 再次触发该异常，这次传入的异常的 `FirstChance` 被置为 `FALSE`。

### 原始版本 SEH 机制总结

到这里，系统提供的 `SEH` 机制（本文又称之为原始版本）大致讲解完毕。咱可以回味一下：

1. 原始版本的实现较简单，代码量不大，而且 `wrk` 基本上有所有关键函数的实现代码。



## 2. 原始版本与增强版

原始版本的功能过于简单，实际过程中很难直接使用。整个异常处理过程无非就是遍历异常链表，挨个调用异常注册信息的处理函数，如果其中有某个处理函数处理了该异常（返回值为 `ExceptionContinueExecution`），那么就从异常触发点（如果是断点异常，则要回退一个字节的指令（`int 3` 指令本身））重新执行。否则不管是整个链表中没有找到合适的处理函数（返回值为 `ExceptionContinueSearch`），或者遍历过程中出现问题（返回值为 `ExceptionNestedException`），系统都会简单粗暴的 `BUGCHECK`。而这也带来一个问题：

线程运行过程中会调用很多个函数，每个函数都有可能注册异常处理，它们提供的异常处理函数既可能处理该函数自身触发的异常，又可能需要处理其子孙函数触发的异常。前者还好说，自己出了问题，多少还有可能自己修复。而后者就很头疼了，它无法了解所有其调用的子孙函数内部的实现，要想修复子孙函数触发的异常，太困难了。而一旦没有正确处理，或者没人处理，系统就崩掉。这个后果太严重。于是实际上现实程序设计中，基本上没有直接使用原始版本的 `SEH`，而是使用编译器提供的增强版本。

下面咱们就来聊聊编译器提供的增强版本。

首先要说明，增强版本有很多个，不同的编译器提供的 `SEH` 增强版本或多或少都有不同处。但是，他们一般都是基于 `windows` 系统提供的原始版本进行完善的。一个典型的增强版就是微软的编译器（后面简称为 `MSC`）里提供的 `__try`、`__finally`，`__except`。咱们接下来就用这个增强版作为目标进行分析。我使用的 `MSC` 是 `WDK 7600.16385.1`，内置的 `cl` 的版本是 `15.00.30729.207`，`link` 的版本是 `9.00.30729.207`，测试虚拟机系统为 32 位 `Win2k3sp1` + `wrk`。

### 增强版数据结构

咱们先看看增强版的数据结构，跟之前的原始版本有很多相似之处：

```c
typedef struct _EXCEPTION_REGISTRATION PEXCEPTION_REGISTRATION;
struct _EXCEPTION_REGISTRATION{
    struct _EXCEPTION_REGISTRATION *prev;
    void (*handler)(PEXCEPTION_RECORD, PEXCEPTION_REGISTRATION, PCONTEXT, PEXCEPTION_RECORD);
    struct scopetable_entry *scopetable;
    int trylevel;
    int _ebp;
    PEXCEPTION_POINTERS xpointers;
};
```

这个 `EXCEPTION_REGISTRATION` 在增强版中就相当于原始版本中的 `EXCEPTION_REGISTRATION_RECORD`。可以这么理解它：

```c
struct _EXCEPTION_REGISTRATION{
    struct _EXCEPTION_REGISTRATION_RECORD ExceptionRegistrationRecord;
    struct scopetable_entry *scopetable;
    int trylevel;
    int _ebp;
    PEXCEPTION_POINTERS xpointers;
}; // 注：本结构体只用于理解原始版和增强版的区别，实际代码中并没有这种形式的定义
```

也就是说它沿用了老版本的注册信息结构，只是在域成员名称上做了些改动，把 `Next` 改名为 `prev`，把 `Handler` 改为 `handler`。除此之外，在原始版本基础上增加了 4 个域成员（`scopetable`、`trylevel`、`_ebp`、`xpointers`），用来支持它的增强功能。

需要说明的是，这结构体来源于 `MSC` 的 `crt` 源码里的 `exsup.inc`，这个文件使用的是汇编语法，该结构体定义是从该文件的注释中提取出来。在实际的分析过程中，发现它的定义有一些问题：最后一个域成员 `xpointers` 实际上存放在 `prev` 之前，也就是说，实际中 `__try` 增强版用的结构体是这样的：

```c
typedef struct _EXCEPTION_REGISTRATION PEXCEPTION_REGISTRATION;
struct _EXCEPTION_REGISTRATION{
    PEXCEPTION_POINTERS xpointers;
    struct _EXCEPTION_REGISTRATION *prev;
    void (*handler)(PEXCEPTION_RECORD, PEXCEPTION_REGISTRATION, PCONTEXT, PEXCEPTION_RECORD);
    struct scopetable_entry *scopetable;
    int trylevel;
    int _ebp;
};
```

### 相关的宏和结构

```asm
TRYLEVEL_NONE         equ     -1
TRYLEVEL_INVALID      equ     -2
```

`scopetable_entry`

+ `+0x000 previousTryLevel : Uint4B`
+ `+0x004 lpfnFilter       : Ptr32 int`
+ `+0x008 lpfnHandler      : Ptr32 int`

咱们先来简单的看一下增强版中出现的几个新域成员。

`EXCEPTION_REGISTRATION::scopetable` 是类型为 `scopetable_entry` 的数组。

`EXCEPTION_REGISTRATION::trylevel` 是数组下标，用来索引 `scopetable` 中的数组成员。

`_ebp` 是包含该 `_EXCEPTION_REGISTRATION` 结构体的函数的栈帧指针。对于没有 `FPO` 优化过的函数，一开头通常有个 `push ebp` 的操作，`_ebp` 的值就是被压入的 `ebp` 的值，后续咱们通过代码就再看实际的应用。

### 处理多个 `__try`/`__except`

按照原始版本的设计，每一对“触发异常-处理异常”都会有一个注册信息即 `EXCEPTION_REGISTRATION_RECORD`。也就是说，如果按照原始的设计，每一个 `__try/__except(__finally)` 都应该对应一个 `EXCEPTION_REGISTRATION`。但是实际的 `MSC` 实现不是这样的。

真正的实现是：

每个使用 `__try/__except(__finally)` 的函数，不管其内部嵌套或反复使用多少 `__try/__except(__finally)`，都只注册一遍，即只将一个 `EXCEPTION_REGISTRATION` 挂入当前线程的异常链表中（对于递归函数，每一次调用都会创建一个 `EXCEPTION_REGISTRATION`，并挂入线程的异常链表中，这是另外一回事）。

那如何处理函数内部出现的多个 `__try/__except(__finally)` 呢？这多个 `__except` 代码块的功能可能大不相同，而注册信息 `EXCEPTION_REGISTRATION` 中只能提供一个处理函数 `handler`，怎么办？

`MSC` 的做法是，`MSC` 提供一个处理函数，即 `EXCEPTION_REGISTRATION::handler` 被设置为 `MSC` 的某个函数，而不是程序猿提供的 `__except` 代码块。程序猿提供的多个 `__except` 块被存储在 `EXCEPTION_REGISTRATION::scopetable` 数组中。我们看看上面的 `scopetable_entry` 定义，由于我没有找到它的定义代码，所以就贴了 `windbg` 中 `dt` 输出结果。

其中 `scopetable_entry::lpfnHandler` 就是程序猿提供的 `__except` 异常处理块代码。而 `lpfnFilter` 就是 `__except` 的过滤块代码。对于 `__finally` 代码块，其 `lpfnFilter` 被置为 `NULL`，`lpfnHandler` 就是其包含的代码块。

下面，我们用一小段简单的伪代码来详细说明。

```cpp
1      VOID SimpleSeh()
2      {
3          __try
4          {
5          }
6          __except(ExceptionFilter_0(...))
7          {
8              ExceptCodeBlock_0;
9          }
10
11         __try
12         {
13             __try
14             {
15             }
16             __except(ExceptionFilter_1(...))
17             {
18                 ExceptCodeBlock_1;
19             }
20         }
21         __except(ExceptionFilter_2(...))
22         {
23             ExceptCodeBlock_2;
24         }
25     }
```

编译时，编译器会为 `SimpleSeh` 分配一个 `EXCEPTION_REGISTRATION` 和一个拥有 3 个成员的 `scopetable` 数组，并将 `EXCEPTION_REGISTRATION::scopetable` 指向该数组（请留意：`EXCEPTION_REGISTRATION::scopetable` 只是一个指针，不是数组）。然后按照 `__try` 关键字出现的顺序，将对应的 `__except/__finally` 都存入该数组，步骤如下：

`scopetable[0].lpfnFilter = ExceptionFilter_0;`

`scopetable[0].lpfnHandler = ExceptCodeBlock_0;`

`scopetable[1].lpfnFilter = ExceptionFilter_1;`

`scopetable[1].lpfnHandler = ExceptCodeBlock_1;`

`scopetable[2].lpfnFilter = ExceptionFilter_2;`

`scopetable[2].lpfnHandler = ExceptCodeBlock_2;`

我们假象当前开始执行 `SimpleSeh` 函数，在行 14 和行 15 之间触发了异常。

根据之前我们的讨论的流程：`RtlRaiseException` -> `RtlDispatchException` -> `RtlpExecuteHandlerForException`。

`RtlpExecuteHandlerForException` 会调用注册信息中的处理函数，即 `EXCEPTION_REGISTRATION::handler`。该函数是由 `MSC` 提供的，内部会依次调用 `scopetable` 中的 `lpfnHandler`。

那咱们来模拟执行一下，在 14 和 15 行之前触发异常，那应该先从 `scopetable[2]` 的 `ExceptionFilter_2` 开始执行，假设该函数返回 `EXCEPTION_CONTINUE_SEARCH`。那接下来应该是 `scopetable[1]`，假设 `ExceptionFilter_1` 也返回 `EXCEPTION_CONTINUE_SEARCH`。那么接下来是不是就应该轮到 `scopetable[0]` 了？不是。咱们再看看上面的伪代码，行 14 和行 15 之间的代码并没处于第一个 `__try/__except` 的范围中，该异常轮不到 `scopetable[0]` 来处理。那怎么办？`SimpleSeh` 执行的过程中怎么知道到 `scopetable[1]` 就应该停止？

`MSC` 是通过 `scopetable_entry::previousTryLevel` 来解决这个问题的。上面数组的设置，完整的形式其实是这样：

`scopetable[0].previousTryLevel = TRYLEVEL_NONE;`

`scopetable[0].lpfnFilter = ExceptionFilter_0;`

`scopetable[0].lpfnHandler = ExceptCodeBlock_0;`

`scopetable[1].previousTryLevel = TRYLEVEL_NONE;`

`scopetable[1].lpfnFilter = ExceptionFilter_1;`

`scopetable[1].lpfnHandler = ExceptCodeBlock_1;`

`scopetable[2].previousTryLevel = 1;`

`scopetable[2].lpfnFilter = ExceptionFilter_2;`

`scopetable[2].lpfnHandler = ExceptCodeBlock_2;`

`scopetable_entry::previousTryLevel` 包含的意思是“下一个该轮到数组下标为 `previousTryLevel` 的单元了”。当 `scopetable_entry::previousTryLevel` 等于 `TRYLEVEL_NONE`(-1) 时，就会停止遍历 `scopetable`。

咱再来模拟执行一遍，当 14 和 15 行之间触发异常时，首先遍历到 `scopetable[2]`，处理完后，找到 `scopetable[2].previousTryLevel`，发现其值为 1，那么遍历到 `scopetable[1]`，处理完后，找到 `scopetable[1].previousTryLevel`，发现其值为 `TRYLEVEL_NONE`，于是停止遍历。

好像挺圆满的，是吧。

咱们再假设下，如果行 4 和行 5 之间触发了同样的异常，执行流程应该如何。首先，执行 `scopetable[2]`，然后在 `scopetable[1]`，然后……（省略若干同上字）。停！这次的异常是在第一个 `__try/__except` 中触发的，轮不到 `scopetable[2]` 来处理，怎么办？

这个时候就轮到 `EXCEPTION_REGISTRATION::trylevel` 出场了~。`EXCEPTION_REGISTRATION::trylevel` 的作用就是标识从那个数组单元开始遍历。

与 `scopetable_entry::previousTryLevel` 不同，`EXCEPTION_REGISTRATION::trylevel` 是动态变化的，也就是说，这个值在 `SimpleSeh` 执行过程中是会经常改变的。比如，

+ 执行到行 4 和行 5 之间，该值就会被修改为 0；
+ 执行到第 12 行，该值被修改为 1；
+ 执行到 14 行，该值为 2。

这样，当异常触发时候，`MSC` 就能正确的遍历 `scopetable` 了。

### 结构示意图

这里我画了一幅草图来帮助理解:

图中下方是低地址端，上方是高地址端。

（boxcounter: 这幅图是我借助 `vim` 的列操作手绘的，哪位朋友知道有专门画这类文本图的工具吗（除了 `emacs` 的图操作模式，这玩意太臃肿了，我不太喜欢）？欢迎告知我 `ns.boxcounter[a]gmail.com`。非常感谢。）

![SEH结构草图](http://www.boxcounter.com/wp-content/uploads/2011/10/SEH%E7%BB%93%E6%9E%84%E8%8D%89%E5%9B%BE.jpg)

```text
     4G   |--------------------------|        ...
          |  ...                     |        |
       --> |--------------------------|        |
     /     | ret_addr                 |        |
  func1    | _EXCEPTION_REGISTRATION  |    _EXCEPTION_REGISTRATION      /  previousTryLevel = TRYLEVEL_NONE        \
```



```
\    | ...                    |         |                      -> scopetable[0] |  lpfnFilter       = ExceptionFilter_0    |  

        --> |--------------------------|         |                    /              \  lpfnHandler      = ExceptionCodeBlock_0 /  

       /    | ret_addr                 |         |                   /              /  previousTryLevel = TRYLEVEL_NONE        \   <-  

     func2  | _EXCEPTION_REGISTRATION  |    _EXCEPTION_REGISTRATION      scopetable[1] |  lpfnFilter       = ExceptionFilter_1    |    |  

       \    | ...                    |         |                   \              \  lpfnHandler      = ExceptionCodeBlock_1 /    |  

        --> |--------------------------|         |                    \              /  previousTryLevel = 1                     \   -^  

       /    | ret_addr                 |         |                      -> scopetable[2] |  lpfnFilter       = ExceptionFilter_2    |  

     func3  | _EXCEPTION_REGISTRATION  |    _EXCEPTION_REGISTRATION                    \  lpfnHandler      = ExceptionCodeBlock_2 /  

       \    | ...                    |         |  

        --> |--------------------------|         |  

       /    | ret_addr                 |         |  

     func4  | _EXCEPTION_REGISTRATION  |    _EXCEPTION_REGISTRATION  

       \    | ...                    |           ^  

        --> |                      |           |  

            |                      |        FS:[0]  

       0 -> |--------------------------|  
```

这幅图中的函数关系是： `func1` -> `func2` -> `func3` -> `func4`。

到目前位置，咱们已经熟悉了增强版的概要流程。下面结合真实代码来分析。代码分为三块：SEH 创建代码、MSC 提供的 `handler` 函数，以及展开函数。

在开始看分析代码之前，先把后面分析过程中需要用的宏和结构体列出来：

代码:

```
#define EXCEPTION_NONCONTINUABLE 0x1    // Noncontinuable exception
#define EXCEPTION_UNWINDING 0x2         // Unwind is in progress
#define EXCEPTION_EXIT_UNWIND 0x4       // Exit unwind is in progress
#define EXCEPTION_STACK_INVALID 0x8     // Stack out of limits or unaligned
#define EXCEPTION_NESTED_CALL 0x10      // Nested exception handler call
#define EXCEPTION_TARGET_UNWIND 0x20    // Target unwind in progress
#define EXCEPTION_COLLIDED_UNWIND 0x40  // Collided exception handler call

#define EXCEPTION_UNWIND (EXCEPTION_UNWINDING | EXCEPTION_EXIT_UNWIND | \
                          EXCEPTION_TARGET_UNWIND | EXCEPTION_COLLIDED_UNWIND)

nt!_EXCEPTION_RECORD
   +0x000 ExceptionCode    : Int4B
   +0x004 ExceptionFlags   : Uint4B
   +0x008 ExceptionRecord  : Ptr32 _EXCEPTION_RECORD
   +0x00c ExceptionAddress : Ptr32 Void
   +0x010 NumberParameters : Uint4B
   +0x014 ExceptionInformation : [15] Uint4B

typedef enum _EXCEPTION_DISPOSITION {
    ExceptionContinueExecution,
    ExceptionContinueSearch,
    ExceptionNestedException,
    ExceptionCollidedUnwind
} EXCEPTION_DISPOSITION;

// scopetable_entry::lpfnFilter 的返回值，也就是 __except 过滤块的返回值
#define EXCEPTION_EXECUTE_HANDLER       1
#define EXCEPTION_CONTINUE_SEARCH       0
#define EXCEPTION_CONTINUE_EXECUTION    -1
```

### 一、SEH 创建代码

这里我没有继续使用上面的 `SimpleSeh` 进行分析，而是新写了一个简单的 `SehTest` 函数。

代码:

```
VOID SehTest()
{
    ULONG ulVal = 0;

    __try // 第一个 __try 域
    {
        ulVal = 0x11111111; // 最后一位为 1 表示“在 __try 代码块中”
    }
    __except(Filter_0())
    {
        ulVal = 0x11111110; // 最后一位为 0 表示“在 __except/__finally 代码块中”
    }

    __try // 第二个 __try 域
    {
        ulVal = 0x22222222;

        __try // 第三个 __try 域
        {
            ulVal = 0x33333333;

            *((ULONG*)NULL) = ulVal; // 触发异常
        }
        __finally
        {
            ulVal = 0x33333330;
        }
    }
    __except(Filter_2())
    {
        ulVal = 0x22222220;
    }

    return;
}
```

反汇编代码如下：

（需要说明一下我的命名习惯是，"l_"前缀的变量是函数的局部变量，没有任何前缀的变量是传入的参数）

引用:

```
kd> uf passthrough!SehTest

    ;PassThrough!SehTest [d:\workspace\code\mycode\r0\passthrough\passthrough.c @ 604]:

      f8720040  mov     edi,edi

      f8720042  push    ebp           ; l_ExceptionRegistration->_ebp

      f8720043  mov     ebp,esp

      f8720045  push    0FFFFFFFEh    ; l_ExceptionRegistration->trylevel = TRYLEVEL_INVALID (-2)

      f8720047  push    offset PassThrough!__safe_se_handler_table+0x8 (f8721468)   ; l_ExceptionRegistration->scopetable

      f872004c  push    offset PassThrough!_except_handler4 (f8720390)               ; l_ExceptionRegistration->handler

      f8720051  mov     eax,dword ptr fs:[00000000h]

      f8720057  push    eax           ; _EXCEPTION_REGISTRATION::prev

      f8720058  add     esp,0FFFFFFF4h  ; 这里分配了 0xc 字节的栈空间，其中紧贴着 l_ExceptionRegistration->prev

                                       ; 的 4 个字节存放着 l_ExceptionRegistration->xpointers

      f872005b  push    ebx

      f872005c  push    esi

      f872005d  push    edi

      f872005e  mov     eax,dword ptr [PassThrough!__security_cookie (f87220b0)]

      f8720063  xor     dword ptr [ebp-8],eax  ; 对 scopetable 进行异或加密

      f8720066  xor     eax,ebp              ; 对 __security_cookie 进行加密

      f8720068  push    eax                ; 把加密了的 __security_cookie 也压入栈中，后面用来对 scopetable 进行解密

      f8720069  lea     eax,[ebp-10h]

      f872006c  mov     dword ptr fs:[00000000h],eax   ; 将 l_ExceptionRegistration 挂入线程异常链表中

      f8720072  mov     dword ptr [ebp-18h],esp

      f8720075  mov     dword ptr [ebp-1Ch],0

      f872007c  mov     dword ptr [ebp-4],0      ; 进入第一个 __try 域，l_ExceptionRegistration->trylevel = 0

      f8720083  mov     dword ptr [ebp-1Ch],11111111h

      f872008a  mov     dword ptr [ebp-4],0FFFFFFFEh   ; 离开第一个 __try 域，l_ExceptionRegistration->trylevel = TRYLEVEL_NONE (-2)

      f8720091  jmp     PassThrough!SehTest+0x6a (f87200aa)

        ; ------------------------------------------------------------------------

        ; 这里有个空洞，用 uf 命令是不会显示的。范围是 f8720093 到 f87200a9，汇编码如下

        ;

        ; PassThrough!SehTest+0x53 [d:\workspace\code\mycode\r0\passthrough\passthrough.c @ 611]:

        ; f8720093  call    PassThrough!Filter_0 (f8720010)

        ; f8720098  ret

        ;

        ; f8720099  mov     esp,dword ptr [ebp-18h]       ; 第一个 __except 处理域

        ; f872009c  mov     dword ptr [ebp-1Ch],11111110h

        ; f87200a3  mov     dword ptr [ebp-4],0FFFFFFFEh   ; 离开第一个 __try 域，l_ExceptionRegistration->trylevel = TRYLEVEL_NONE (-2)

        ; ------------------------------------------------------------------------

    ;PassThrough!SehTest+0x6a [d:\workspace\code\mycode\r0\passthrough\passthrough.c @ 616]:

      f87200aa  mov     dword ptr [ebp-4],1     ; 进入第二个 __try 域，l_ExceptionRegistration->trylevel = 1

      f87200b1  mov     dword ptr [ebp-1Ch],22222222h

      f87200b8  mov     dword ptr [ebp-4],2     ; 进入第三个 __try 域，l_ExceptionRegistration->trylevel = 2

      f87200bf  mov     dword ptr [ebp-1Ch],33333333h

      f87200c6  mov     eax,dword ptr [ebp-1Ch]

      f87200c9  mov     dword ptr ds:[00000000h],eax   ; 触发异常

      f87200ce  mov     dword ptr [ebp-4],1     ; 离开第三个 __try 域，l_ExceptionRegistration->trylevel = 1

      f87200d5  call    PassThrough!SehTest+0x9c (f87200dc)
```



```asm
f87200da  jmp     PassThrough!SehTest+0xa4 (f87200e4)  

    ; ---------------------------------------------------------------------------------  
    ; 空洞，范围是 f87200dc 到 f87200e3，汇编码如下  
    ;   
    ; PassThrough!SehTest+0x9c [d:\workspace\code\mycode\r0\passthrough\passthrough.c @ 628]:  
    ; f87200dc c745e430333333  mov     dword ptr [ebp-1Ch],33333330h    ; __finally 域  
    ; ---------------------------------------------------------------------------------  

; PassThrough!SehTest+0xa4 [d:\workspace\code\mycode\r0\passthrough\passthrough.c @ 630]:  
  f87200e4  mov     dword ptr [ebp-4],0FFFFFFFEh    ; 离开第二个 __try 域，l_ExecutionRegistration->trylevel = TRYLEVEL_NONE (-2)  
  f87200eb  jmp     PassThrough!SehTest+0xc4 (f8720104)  

    ; ---------------------------------------------------------------------------------  
    ; 空洞，范围是 f87200ed 到 f8720103，汇编码如下  
    ;   
    ; PassThrough!SehTest+0xad [d:\workspace\code\mycode\r0\passthrough\passthrough.c @ 631]:  
    ; f87200ed e83effffff      call    PassThrough!Filter_2 (f8720030)  
    ; f87200f2 c3              ret  
    ;  
    ; f87200f3 8b65e8          mov     esp,dword ptr [ebp-18h]          ; 第二个 __except 处理域  
    ; f87200f6 c745e420222222  mov     dword ptr [ebp-1Ch],22222220h  
    ; f87200fd c745fcfeffffff  mov     dword ptr [ebp-4],0FFFFFFFEh     ; 离开第三个 __try 域，l_ExceptionRegistration->trylevel = TRYLEVEL_NONE (-2)  
    ; ---------------------------------------------------------------------------------  

; PassThrough!SehTest+0xc4 [d:\workspace\code\mycode\r0\passthrough\passthrough.c @ 637]:  
  f8720104  mov     ecx,dword ptr [ebp-10h]  
  f8720107  mov     dword ptr fs:[0],ecx            ; 恢复旧的 EXCEPTION_REGISTRATION。即从线程异常链表中摘除 l_ExceptionRegistration  
  f872010e  pop     ecx  
  f872010f  pop     edi  
  f8720110  pop     esi  
  f8720111  pop     ebx  
  f8720112  mov     esp,ebp  
  f8720114  pop     ebp  
  f8720115  ret 
```

来看看`scopetable`的内容：

```text
kd> dd f8721468

f8721468  fffffffe 00000000 ffffffd4 00000000  < 16 个字节的坑  
f8721478  [fffffffe f8720093 f8720099] [fffffffe  
f8721488  f87200ed f87200f3] [00000001 00000000  
f8721498  f87200dc] 00000000 00000000 00000000  
f87214a8  00000000 00000000 00000000 00000000  
f87214b8  00000000 00000000 00000000 00000000  
f87214c8  00000000 00000000 00000000 00000000  
f87214d8  00000000 00000000 00000000 00000000  
```

前 16 个字节是坑，坑的作用晚点再说。之后就是三个`scopetable_entry`，被我用大括号扩起来了。对照前面的汇编码可以发现，`scopetable_entry::lpfnFilter`和`scopetable_entry::lpfnHandler`就是汇编码中的空洞处的代码。其中，第三个`scopetable_entry::lpfnFilter`是`NULL`，对照代码可以发现，是因为这是一个`__try`&`__finally`块，没有`lpfnFilter`。

汇编代码很简单，注释里也有详细的分析过程了，我不再多嗦。只提两点：

1. `EXCEPTION_REGISTRATION::scopetable`指针被用`__security_cookie`进行了异或加密。
2. `EXCEPTION_REGISTRATION::scopetable`并不直接指向`scopetable_entry`数组，在第一个`scopetable_entry`之前有 16 个字节的坑。后续分析中会看到，它的主要作用是帮助验证`scopetable`是否被破坏。第三个`DWORD`，即上文中的`ffffffd4`是一个偏移量，后续的分析过程中会看得很清楚。

## 二、MSC 提供的 `EXCEPTION_REGISTRATION::handler` 函数

之前咱们有说过`EXCEPTION_REGISTRATION::handler`指向由`MSC`编译器提供的一个函数，这个函数内部负责调用`scopetable[?]->lpfnFilter/lpfnHandler`。这个函数通常为`module!_except_handler?`，其中`module`为模块名，`?`表示某数字。在分析过程中发现，有的模块完整实现了该函数，有的模块直接使用了系统提供的`nt!_except_handler?`。不知道是因为版本的问题，还是编译选项的问题。我没有深究。

我继续以`passhThrough`模块为例，来说明这个函数。

首先需要再次说明一下我的注释习惯：后续列出来的反汇编代码中，最左边的`:`<>符号表示跳转，是我在分析的过程中为了方便理解流程手写的。其中`<`表示跳转源，`>`是跳转目标，`:`用于组成连接线。

为了方便对照，我把经过整理的`PassThrough!_except_handler4`的原型列在这里：

```c
EXCEPTION_DISPOSITION _except_handler4 (
    PEXCEPTION_RECORD pExceptionRecord,
    PEXCEPTION_REGISTRATION pExceptionRegistration,
    PCONTEXT,
    PDISPATCHER_CONTEXT
);
```

反汇编代码：



```
    kd> uf PassThrough!_except_handler4
                PassThrough!_except_handler4 [d:\5359\minkernel\crts\crtw32\misc\i386\chandler4.c @ 255]:
                    f8720390  mov     edi,edi
                    f8720392  push    ebp
                    f8720393  mov     ebp,esp
                    f8720395  sub     esp,14h
                    f8720398  push    ebx
                    f8720399  mov     ebx,dword ptr [ebp+0Ch] ; ebx = pExceptionRegistration
                    f872039c  push    esi
                    f872039d  mov     esi,dword ptr [ebx+8]   ; esi = pExceptionRegistration->scopetable
                    f87203a0  xor     esi,dword ptr [PassThrough!__security_cookie (f87220b0)]    ; 用 __security_cookie 对 scopetable 解密
                    f87203a6  push    edi
                    f87203a7  mov     eax,dword ptr [esi]
                    f87203a9  mov     byte ptr [ebp-1],0   ; ebp-1 存放的是一个 BOOLEAN 值，用来表示是否执行过任何 scopetable_entry::lpfnFilter
                    f87203ad  mov     dword ptr [ebp-8],1  ; ebp-8 被用来存放本函数的返回值，这里初始化为 ExceptionContinueSearch (1)
                    f87203b4  lea     edi,[ebx+10h]        ; edi = pExceptionRegistration->_ebp 
                    f87203b7  cmp     eax,0FFFFFFFEh       ; 检查 scopetable 中坑的第一个 DWORD 值，后续来做相关的安全处理
<                  f87203ba  je      PassThrough!_except_handler4+0x39 (f87203c9)
:                  
:                PassThrough!_except_handler4+0x2c [d:\5359\minkernel\crts\crtw32\misc\i386\chandler4.c @ 315]:
:                  ; 校验 scopetable 完整性（1）
:                  f87203bc  mov     ecx,dword ptr [esi+4]
:                  f87203bf  add     ecx,edi
:                  f87203c1  xor     ecx,dword ptr [eax+edi]
:                  f87203c4  call    PassThrough!__security_check_cookie (f8720638)
:                
:                PassThrough!_except_handler4+0x39 [d:\5359\minkernel\crts\crtw32\misc\i386\chandler4.c @ 315]:
:                  ; 校验 scopetable 完整性（2）
>                 f87203c9  mov     ecx,dword ptr [esi+0Ch]
                   f87203cc  mov     eax,dword ptr [esi+8]
                   f87203cf  add     ecx,edi
                   f87203d1  xor     ecx,dword ptr [eax+edi]
                   f87203d4  call    PassThrough!__security_check_cookie (f8720638)
                    ; 安全工作处理完毕，开始进入异常处理流程
                   f87203d9  mov     eax,dword ptr [ebp+8]     ; eax = pExceptionRecord
                   f87203dc  test    byte ptr [eax+4],66h      ; pExceptionRecord->ExceptionFlags & EXCEPTION_UNWIND，判断是异常处理过程还是展开过程
<                 f87203e0  jne     PassThrough!_except_handler4+0x138 (f87204c8)
:                
:                PassThrough!_except_handler4+0x56 [d:\5359\minkernel\crts\crtw32\misc\i386\chandler4.c @ 331]:
:                  ; 异常处理过程
:                  f87203e6  mov     ecx,dword ptr [ebp+10h]   ; ecx = pContext
:                  f87203e9  lea     edx,[ebp-14h]
:                  f87203ec  mov     dword ptr [ebx-4],edx     ; ebx-4 是创建 EXCEPTION_REGISTRATION 时候预留的 xpointers 的空间，这里给它赋值
:                  f87203ef  mov     ebx,dword ptr [ebx+0Ch]   ; ebx = pExceptionRegistration->trylevel
:                  f87203f2  mov     dword ptr [ebp-14h],eax   ; [ebp-14] = pExceptionRecord，即 xpointers->ExceptionRecord = pExceptionRecord
:                  f87203f5  mov     dword ptr [ebp-10h],ecx   ; [ebp-10] = pContext，即 xpointers->ContextRecord = pContext
:                  f87203f8  cmp     ebx,0FFFFFFFEh            ; cmp pExceptionRegistration->trylevel, TRYLEVEL_INVALID
: <                 f87203fb  je      PassThrough!_except_handler4+0xcc (f872045c)
: :                
: :               PassThrough!_except_handler4+0x6d [d:\5359\minkernel\crts\crtw32\misc\i386\chandler4.c @ 341]:
: :                 f87203fd  lea     ecx,[ecx]
: :                
: :               PassThrough!_except_handler4+0x70 [d:\5359\minkernel\crts\crtw32\misc\i386\chandler4.c @ 343]:
: :   >             f8720400  lea     eax,[ebx+ebx*2] ; 这里 *3，下面紧接着 *4，即 *12，实际上是为了跳过 x 个 scopetable_entry (大小为 12 个字节)
: :   :             f8720403  mov     ecx,dword ptr [esi+eax*4+14h] ; ecx = scopetable[i].lpfnFilter, 这里 14h 是为了跳过 10h 大小的坑
: :   :             f8720407  lea     eax,[esi+eax*4+10h] ; eax = &scopetable[i]
: :   :             f872040b  mov     dword ptr [ebp-0Ch],eax ; ebp-0Ch 存放的是 pCurrentScopeTableEntry
: :   :             f872040e  mov     eax,dword ptr [eax] ; eax = scopetable[i].previousTryLevel
: :   :             f8720410  mov     dword ptr [ebp+8],eax ; 这里是将 ebp+8 当作局部变量使用, [ebp+8] = scopetable[i].previousTryLevel
: :   :             f8720413  test    ecx,ecx ; 判断 scopetable[i].lpfnFilter 是否为 NULL
: : <     :         f8720415  je      PassThrough!_except_handler4+0x9b (f872042b)
: : :     :        
: : :     :       PassThrough!_except_handler4+0x87 [d:\5359\minkernel\crts\crtw32\misc\i386\chandler4.c @ 355]:
: : :     :         ; scopetable[i].lpfnFilter 不为 NULL，调用它
: : :     :         f8720417  mov     edx,edi ; edx = pExceptionRegistration->_ebp
: : :     :         f8720419  call    PassThrough!_EH4_CallFilterFunc (f87205d1)
: : :     :         f872041e  mov     byte ptr [ebp-1],1 ; [ebp-1] 表示是否执行过 lpfnFilter，它是个 BOOLEAN 值
: : :     :         f8720422  test    eax,eax
: : : <   :         f8720424  jl      PassThrough!_except_handler4+0xd6 (f8720466) ; 如果是 EXCEPTION_CONTINUE_EXECUTION (-1) 就跳
: : : :   :        
: : : :   :       PassThrough!_except_handler4+0x96 [d:\5359\minkernel\crts\crtw32\misc\i386\chandler4.c @ 370]:
: : : : < :         f8720426  jg      PassThrough!_except_handler4+0xdf (f872046f) ; 如果是 EXCEPTION_EXECUTE_HANDLER (1) 就跳
: : : : : :        
: : : : : :       PassThrough!_except_handler4+0x98 [d:\5359\minkernel\crts\crtw32\misc\i386\chandler4.c @ 370]:
: : : : : :         ; lpfnFilter 返回 EXCEPTION_CONTINUE_SEARCH
: : : : : :         f8720428  mov     eax,dword ptr [ebp+8] ; eax = scopetable[i].previousTryLevel 
: : : : : :        
: : : : : :       PassThrough!_except_handler4+0x9b [d:\5359\minkernel\crts\crtw32\misc\i386\chandler4.c @ 341]:
: : > : : :         f872042b  mov     ebx,eax
: :   : : :         f872042d  cmp     eax,0FFFFFFFEh ; cmp scopetable[i].previousTryLevel, TRYLEVEL_INVALID
: :   : : <         f8720430  jne     PassThrough!_except_handler4+0x70 (f8720400)
: :   : :         
: :   : :         PassThrough!_except_handler4+0xa2 [d:\5359\minkernel\crts\crtw32\misc\i386\chandler4.c @ 478]:
: :   : :           f8720432  cmp     byte ptr [ebp-1],0 ; 没有执行过 lpfnFilter，无需进行安全检查
: : < : :           f8720436  je      PassThrough!_except_handler4+0xcc (f872045c)
: : : : :         
: : : : :         PassThrough!_except_handler4+0xa8 [d:\5359\minkernel\crts\crtw32\misc\i386\chandler4.c @ 486]:
: : : : :           ; 执行过 lpfnFilter，需要校验完整性
: : : : :   >   >   f8720438  mov     eax,dword ptr [esi] 
: : : : :   :   :   f872043a  cmp     eax,0FFFFFFFEh ; 根据 scopetable 坑的第一个 DWORD 值判断是否需要做进一步的安全检查
: : : : : < :   :   f872043d  je      PassThrough!_except_handler4+0xbc (f872044c)
: : : : : : :   : 
: : : : : : :   : PassThrough!_except_handler4+0xaf [d:\5359\minkernel\crts\crtw32\misc\i386\chandler4.c @ 486]:
: : : : : : :   :   ; 校验 scopetable 完整性（1）
: : : : : : :   :   f872043f  mov     ecx,dword ptr [esi+4]
: : : : : : :   :   f8720442  add     ecx,edi
: : : : : : :   :   f8720444  xor     ecx,dword ptr [eax+edi]
: : : : : : :   :   f8720447  call    PassThrough!__security_check_cookie (f8720638)
: : : : : : :   : 
: : : : : : :   : PassThrough!_except_handler4+0xbc [d:\5359\minkernel\crts\crtw32\misc\i386\chandler4.c @ 486]:
: : : : : : :   :   ; 校验 scopetable 完整性（2）
: : : : : > :   :   f872044c  mov     ecx,dword ptr [esi+0Ch]
: : : : :   :   :   f872044f  mov     edx,dword ptr [esi+8]
: : : : :   :   :   f8720452  add     ecx,edi
: : : : :   :   :   f8720454  xor     ecx,dword ptr [edx+edi] ; 正常情况下 [edx+edi] 保存的是 __security_cookie 的值
: : : : :   :   :   f8720457  call    PassThrough!__security_check_cookie (f8720638)
: : : : :   :   : 
: : : : :   :   : PassThrough!_except_handler4+0xcc [d:\5359\minkernel\crts\crtw32\misc\i386\chandler4.c @ 495]:
: > > : :   : > :   f872045c  mov     eax,dword ptr [ebp-8]
:     : :   : : :   f872045f  pop     edi
:     : :   : : :   f8720460  pop     esi
:     : :   : : :   f8720461  pop     ebx
:     : :   : : :   f8720462  mov     esp,ebp
:     : :   : : :   f8720464  pop     ebp
:     : :   : : :   f8720465  ret
:     : :   : : : 
:     : :   : : : PassThrough!_except_handler4+0xd6 [d:\5359\minkernel\crts\crtw32\misc\i386\chandler4.c @ 367]:
:     : :   : : :   ; lpfnFilter 返回 EXCEPTION_CONTINUE_EXECUTION
:     > :   : : :   f8720466  mov     dword ptr [ebp-8],0 ; [ebp-8] = ExceptionContinueExecution (0)
:       :   < : :   f872046d  jmp     PassThrough!_except_handler4+0xa8 (f8720438)
:       :     : : 
:       :     : : PassThrough!_except_handler4+0xdf [d:\5359\minkernel\crts\crtw32\misc\i386\chandler4.c @ 396]:
:       :     : :   ; lpfnFilter 返回 EXCEPTION_EXECUTE_HANDLER
:       >     : :   f872046f  mov     ecx,dword ptr [ebp+0Ch] ; ecx = pExceptionRegistration
:             : :   f8720472  call    PassThrough!_EH4_GlobalUnwind (f87205fa)
:             : :   f8720477  mov     eax,dword ptr [ebp+0Ch] ; eax = pExceptionRegistration
:             : :   f872047a  cmp     dword ptr [eax+0Ch],ebx ; cmp pExceptionRegistration->trylevel
: <           : :   f872047d  je      PassThrough!_except_handler4+0x101 (f8720491)
: :           : : 
: :           : : PassThrough!_except_handler4+0xef [d:\5359\minkernel\crts\crtw32\misc\i386\chandler4.c @ 408]:
: :           : :   f872047f  push    offset PassThrough!__security_cookie (f87220b0)
: :           : :   f8720484  push    edi
: :           : :   f8720485  mov     edx,ebx
: :           : :   f8720487  mov     ecx,eax ; ecx = pExceptionRegistration
: :           : :   f8720489  call    PassThrough!_EH4_LocalUnwind (f8720614)
: :           : :   f872048e  mov     eax,dword ptr [ebp+0Ch]
: :           : : 
: :           : : PassThrough!_except_handler4+0x101 [d:\5359\minkernel\crts\crtw32\misc\i386\chandler4.c @ 417]:
: >           : :   f8720491  mov     ecx,dword ptr [ebp+8]   ; ecx = scopetable[i].previousTryLevel 
:             : :   f8720494  mov     dword ptr [eax+0Ch],ecx ; pExceptionRegistration->trylevel = scopetable[i].previousTryLevel 
:             : :   f8720497  mov     eax,dword ptr [esi]
:             : :   f8720499  cmp     eax,0FFFFFFFEh
: <           : :   f872049c  je      PassThrough!_except_handler4+0x11b (f87204ab)
: :           : : 
: :           : : PassThrough!_except_handler4+0x10e [d:\5359\minkernel\crts\crtw32\misc\i386\chandler4.c @ 431]:
: :           : :   ; 校验 scopetable 完整性（1）
: :           : :   f872049e  mov     ecx,dword ptr [esi+4]
: :           : :   f87204a1  add     ecx,edi
: :           : :   f87204a3  xor     ecx,dword ptr [eax+edi]
: :           : :   f87204a6  call    PassThrough!__security_check_cookie (f8720638)
: :           : : 
: :           : : PassThrough!_except_handler4+0x11b [d:\5359\minkernel\crts\crtw32\misc\i386\chandler4.c @ 431]:
: :           : :   ; 校验 scopetable 完整性（2）
: >           : :   f87204ab  mov     ecx,dword ptr [esi+0Ch]
:             : :   f87204ae  mov     edx,dword ptr [esi+8]
:             : :   f87204b1  add     ecx,edi
:             : :   f87204b3  xor     ecx,dword ptr [edx+edi]
:             : :   f87204b6  call    PassThrough!__security_check_cookie (f8720638)
:             : :   ; 调用 lpfnHandler
:             : :   f87204bb  mov     eax,dword ptr [ebp-0Ch] ; eax = l_pCurrentScopeTableEntry
:             : :   f87204be  mov     ecx,dword ptr [eax+8]   ; ecx = l_pCurrentScopeTableEntry->lpfnHandler
:             : :   f87204c1  mov     edx,edi                 ; edx = pExceptionRegistration->_ebp
:             : :   f87204c3  call    PassThrough!_EH4_TransferToHandler (f87205e8) ; 这里不会返回！！
:             : : 
:             : : PassThrough!_except_handler4+0x138 [d:\5359\minkernel\crts\crtw32\misc\i386\chandler4.c @ 456]:
>             : :   f87204c8  mov     edx,0FFFFFFFEh
              : :   f87204cd  cmp     dword ptr [ebx+0Ch],edx ; cmp pExceptionRegistration->trylevel, TRYLEVEL_INVALID
              < :   f87204d0  je      PassThrough!_except_handler4+0xcc (f872045c)
                : 
                : PassThrough!_except_handler4+0x142 [d:\5359\minkernel\crts\crtw32\misc\i386\chandler4.c @ 467]:
                :   ; pExceptionRegistration->trylevel 不等于 TRYLEVEL_INVALID，开始局部展开
                :   f87204d2  push    offset PassThrough!__security_cookie (f87220b0)
                :   f87204d7  push    edi     ; pExceptionRegistration->_ebp
                :   f87204d8  mov     ecx,ebx  ; ecx = pExceptionRegistration
                :   f87204da  call    PassThrough!_EH4_LocalUnwind (f8720614)
                <   f87204df  jmp     PassThrough!_except_handler4+0xa8 (f8720438)
```



这个函数代码不长。主要分为两个大分支，一个分支处理异常，一个分支展开（参考地址`f87203dc`处的`test`指令）。

处理异常的代码负责遍历`scopetable`，依次调用`scopetable_entry::lpfnFilter`（参考`f8720419`处代码)，并针对不同的返回值做出不同的处理：

1.  返回`EXCEPTION_CONTINUE_EXECUTION`，则说明异常已经被刚刚调用的`lpfnFilter`修复。返回`ExceptionContinueExecution`。
2.  返回`EXCEPTION_CONTINUE_SEARCH`，则继续遍历下一个`scopetable_entry`。
3.  返回`EXCEPTION_EXECUTE_HANDLER`，则说明当前`scopetable_entry::lpfnHandler`负责处理该异常。于是调用它。

对于展开的代码，则直接开始局部展开，即对`scopetable`进行展开。

更具体的信息，请参考上面反汇编代码中我附的注释。

有几个小点需要说一下：

1.  该函数多处使用`scopetable`中坑内的数据进行安全检查。这些操作对理解该函数流程没有帮助，可以忽略。如果觉得上面代码不纯净，可以参考我后续的附录 1《Ntfs!_except_handler3 的反汇编代码》，这个函数流程更清晰简单。
2.  一旦有某`scopetable_entry::lpfnFilter`返回`EXCEPTION_EXECUTE_HANDLER`，就会进行全局展开和局部展开。展开结束后会调用该`scopetable_entry::lpfnHandler`，该函数即为`_except`处理域，该函数形式是一个函数，实际上只是一段不返回的代码。即这段代码中没有`ret`指令。执行完整个`_except`处理域后，会接着执行其后的指令，并不会返回`_except_handler4`。
3.  该函数既启动展开，又负责展开，于是会出现类似于“重入”的现象。理解的过程中容易扰乱思路。

`PassThrough!_except_handler4`在执行过程中可能会调用这几个函数：

`PassThrough!_EH4_CallFilterFunc`、

`PassThrough!_EH4_TransferToHandler`、

`PassThrough!_EH4_GlobalUnwind`、

`PassThrough!_EH4_LocalUnwind`

这几个函数名很明白的说明了它们的功能：

`PassThrough!_EH4_CallFilterFunc` 负责调用 `scopetable_entry::lpfnFilter`；

`PassThrough!_EH4_TransferToHandler` 负责调用 `scopetable_entry::lpfnFilter`；

`PassThrough!_EH4_GlobalUnwind` 负责全局展开；

`PassThrough!_EH4_LocalUnwind` 负责局部展开。

来看看`_EH4_CallFilterFunc`和`_EH4_TransferToHandler`的反汇编代码，都很短。剩余两个展开相关的函数稍后咱们再来分析。

```
    kd> uf PassThrough!_EH4_CallFilterFunc
        PassThrough!_EH4_CallFilterFunc [d:\\5359\\minkernel\\crts\\crtw32\\misc\\i386\\exsup4.asm @ 408]:
          f87205d1  push    ebp
          f87205d2  push    esi
          f87205d3  push    edi
          f87205d4  push    ebx
          f87205d5  mov     ebp,edx
          f87205d7  xor     eax,eax
          f87205d9  xor     ebx,ebx
          f87205db  xor     edx,edx
          f87205dd  xor     esi,esi
          f87205df  xor     edi,edi
          f87205e1  call    ecx ; lpfnFilter();
          f87205e3  pop     ebx
          f87205e4  pop     edi
          f87205e5  pop     esi
          f87205e6  pop     ebp
          f87205e7  ret

    kd> uf PassThrough!_EH4_TransferToHandler
        PassThrough!_EH4_TransferToHandler [d:\\5359\\minkernel\\crts\\crtw32\\misc\\i386\\exsup4.asm @ 450]:
          f87205e8  mov     ebp,edx
          f87205ea  mov     esi,ecx ; esi = lpfnHandler
          f87205ec  mov     eax,ecx
          f87205ee  xor     eax,eax
          f87205f0  xor     ebx,ebx
          f87205f2  xor     ecx,ecx
          f87205f4  xor     edx,edx
          f87205f6  xor     edi,edi
          f87205f8  jmp     esi     ; jmp lpfnHandler
```

到这里，咱们就以`PassThrough!_except_handler4`为例分析完了`MSC`提供的`EXCEPTION_REGISTRATION::handler`函数。这个过程中多次接触到一个名为“展开”的概念，这就是咱们要讲的第三个部分。

### 三、展开（unwind）

为了说明这个概念，需要先回顾下异常发生后的处理流程。

我们假设一系列使用`SEH`的函数调用流程：

`func1` -> `func2` -> `func3`。在`func3`执行的过程中触发了异常。

看看分发异常流程`RtlRaiseException` -> `RtlDispatchException` -> `RtlpExecuteHandlerForException`

`RtlDispatchException`会遍历异常链表，对每个`EXCEPTION_REGISTRATION`都调用`RtlpExecuteHandlerForException`。

`RtlpExecuteHandlerForException`会调用`EXCEPTION_REGISTRATION::handler`，也就是`PassThrough!_except_handler4`。如咱们上面分析，该函数内部遍历`EXCEPTION_REGISTRATION::scopetable`，如果遇到有`scopetable_entry::lpfnFilter`返回`EXCEPTION_EXECUTE_HANDLER`，那么`scopetable_entry::lpfnHandler`就会被调用，来处理该异常。

因为`lpfnHandler`不会返回到`PassThrough!_except_handler4`，于是执行完`lpfnHandler`后，就会从`lpfnHandler`之后的代码继续执行下去。也就是说，假设`func3`中触发了一个异常，该异常被`func1`中的`__except`处理块处理了，那`__except`处理块执行完毕后，就从其后的指令继续执行下去，即异常处理完毕后，接着执行的就是`func1`的代码。不会再回到`func2`或者`func3`，这样就有个问题，`func2`和`func3`中占用的资源怎么办？这些资源比如申请的内存是不会自动释放的，岂不是会有资源泄漏问题？

这就需要用到“展开”了。

说白了，所谓“展开”就是进行清理。（注：这里的清理主要包含动态分配的资源的清理，栈空间是由`func1`的“`mov esp,ebp`”这类操作顺手清理的。当时我被“谁来清理栈空间”这个问题困扰了很久……）

那这个展开工作由谁来完成呢？由`func1`来完成肯定不合适，毕竟`func2`和`func3`有没有申请资源、申请了哪些资源，`func1`无从得知。于是这个展开工作还得要交给`func2`和`func3`自己来完成。

展开分为两种：“全局展开”和“局部展开”。

全局展开是指针对异常链表中的某一段，局部展开针对指定`EXCEPTION_REGISTRATION`。用上面的例子来讲，局部展开就是针对`func3`或`func2`（某一个函数）内部进行清理，全局展开就是`func2`和`func3`的局部清理的总和。再归纳一下，局部展开是指具体某一函数内部的清理，而全局展开是指，从异常触发点（`func3`）到异常处理点（`func1`）之间所有函数（包含异常触发点`func3`）的局部清理的总和。

来看反汇编代码：

```
    kd> uf PassThrough!_EH4_GlobalUnwind
        PassThrough!_EH4_GlobalUnwind [d:\\5359\\minkernel\\crts\\crtw32\\misc\\i386\\exsup4.asm @ 485]:
          f87205fa   push    ebp
          f87205fb   mov     ebp,esp
          f87205fd   push    ebx
          f87205fe   push    esi
          f87205ff   push    edi
          f8720600   push    0   ; pReturnValue
          f8720602   push    0   ; pExceptionRecord
          f8720604   push    offset PassThrough!_EH4_GlobalUnwind+0x15 (f872060f) ; pReturnEip
          f8720609   push    ecx ; pExceptionRegistration
          f872060a   call    PassThrough!RtlUnwind (f8720678)
          f872060f   pop     edi
          f8720610   pop     esi
          f8720611   pop     ebx
          f8720612   pop     ebp
          f8720613   ret
```

`RtlUnwind`的原型：

```
        VOID RtlUnwind(
            PEXCEPTION_REGISTRATION pExceptionRegistration
            PVOID pReturnEip
            PEXCEPTION_RECORD pExceptionRecord,
            PVOID pReturnValue
        );
```



```
    kd> uf PassThrough!RtlUnwind ; 提示：此函数 wrk 提供了实现源码
               nt!RtlUnwind [c:\wrk\wrk-v1.2\base\ntos\rtl\i386\exdsptch.c @ 423]:
                  80867358  push    ebp
                  80867359  mov     ebp,esp
                  8086735b  sub     esp,37Ch
                  80867361  mov     eax,dword ptr [nt!__security_cookie (80895388)]
                  80867366  push    esi
                  80867367  mov     esi,dword ptr [ebp+10h] ; esi = pExceptionRecord
                  8086736a  mov     dword ptr [ebp-4],eax
                  8086736d  push    edi
                  8086736e  lea     eax,[ebp-2D8h] ; [ebp-2D8h] = l_pHighLimit
                  80867374  push    eax
                  80867375  lea     eax,[ebp-2D4h] ; [ebp-2D4h] = l_pLowLimit
                  8086737b  push    eax
                  8086737c  call    nt!RtlpGetStackLimits (80887cdc)
                  80867381  xor     edi,edi
                  80867383  cmp     esi,edi ; pExceptionRecord 是否为 NULL
<                 80867385  jne     nt!RtlUnwind+0x5a (808673b2)
:                 
:                 nt!RtlUnwind+0x2f [c:\wrk\wrk-v1.2\base\ntos\rtl\i386\exdsptch.c @ 452]:
:                   80867387  mov     eax,dword ptr [ebp+4]
:                   8086738a  lea     esi,[ebp-37Ch] ; ebp-37Ch 是局部变量 l_ExceptionRecord
:                   80867390  mov     dword ptr [ebp-37Ch],0C0000027h ; l_ExceptionRecord.ExceptionCode = STATUS_UNWIND
:                   8086739a  mov     dword ptr [ebp-378h],edi ; l_ExceptionRecord.ExceptionFlags = 0
:                   808673a0  mov     dword ptr [ebp-374h],edi ; l_ExceptionRecord.ExceptionRecord = NULL
:                   808673a6  mov     dword ptr [ebp-370h],eax ; l_ExceptionRecord.ExceptionAddress = ret_addr （PassThrough!RtlUnwind 执行完毕后的返回地址）
:                   808673ac  mov     dword ptr [ebp-36Ch],edi ; l_ExceptionRecord.ExceptionInformation[0] = 0;
:                   
:                 nt!RtlUnwind+0x5a [c:\wrk\wrk-v1.2\base\ntos\rtl\i386\exdsptch.c @ 462]:
>                   808673b2  cmp     dword ptr [ebp+8],edi ; pExceptionRegistration 是否为 NULL
 <                 808673b5  je      nt!RtlUnwind+0x65 (808673bd)
 :                 
 :                 nt!RtlUnwind+0x5f [c:\wrk\wrk-v1.2\base\ntos\rtl\i386\exdsptch.c @ 463]:
 :                   808673b7  or      dword ptr [esi+4],2 ; l_ExceptionRecord.ExceptionFlags |= EXCEPTION_UNWINDING (0x2)
 :<                  808673bb  jmp     nt!RtlUnwind+0x69 (808673c1)
 ::                 
 ::                nt!RtlUnwind+0x65 [c:\wrk\wrk-v1.2\base\ntos\rtl\i386\exdsptch.c @ 466]:
 >:                  808673bd  or      dword ptr [esi+4],6 ; l_ExceptionRecord.ExceptionFlags |= EXCEPTION_UNWINDING | EXCEPTION_EXIT_UNWIND (0x2 | 0x4)
  :                 
  :                nt!RtlUnwind+0x69 [c:\wrk\wrk-v1.2\base\ntos\rtl\i386\exdsptch.c @ 466]:
  >                  808673c1  push    ebx
                  808673c2  lea     eax,[ebp-2D0h] ; ebp-2D0 是局部变量 l_Context
                  808673c8  push    eax
                  808673c9  mov     dword ptr [ebp-2D0h],10007h ; lContext.ContextFlags = CONTEXT_i386 | CONTEXT_CONTROL | CONTEXT_INTEGER | CONTEXT_SEGMENTS (0x10000 | 0x1 | 0x2 | 0x3)

                  808673d3  call    nt!RtlpCaptureContext (80887c50)
                  808673d8  mov     eax,dword ptr [ebp+14h] ; eax = ReturnValue
                  808673db  add     dword ptr [ebp-20Ch],10h ; -20C = -2D0+C4, lContext.Esp += 0x10
                  808673e2  mov     dword ptr [ebp-220h],eax ; -220 = -2D0+B0, lContext.Eax = ReturnValue
                  808673e8  call    nt!RtlpGetRegistrationHead (80887d04)
                  808673ed  mov     ebx,eax ; ebx = 异常链表头，这里暂命名为 l_pExceptionRegistration
<                 808673ef  jmp     nt!RtlUnwind+0x1d1 (80867529)
:                 
:                 nt!RtlUnwind+0x9c [c:\wrk\wrk-v1.2\base\ntos\rtl\i386\exdsptch.c @ 500]:
:            >    808673f4  cmp     ebx,dword ptr [ebp+8] ; cmp l_pExceptionRegistration,pExceptionRegistration 
:<              808673f7  jne     nt!RtlUnwind+0xb0 (80867408)
::              :   
::              :  nt!RtlUnwind+0xa1 [c:\wrk\wrk-v1.2\base\ntos\rtl\i386\exdsptch.c @ 501]:
::              :    ; pExceptionRegistration 等于 l_pExceptionRegistration，说明展开完毕
::              :    808673f9  push    edi ; TestAlert = FALSE
::              :    808673fa  lea     eax,[ebp-2D0h] ; eax = &l_Context
::              :    80867400  push    eax
::              :    80867401  call    nt!ZwContinue (8082c0b8) ;  这里不会返回
::<             :    80867406  jmp     nt!RtlUnwind+0xe6 (8086743e)
:::             :   
:::             :  nt!RtlUnwind+0xb0 [c:\wrk\wrk-v1.2\base\ntos\rtl\i386\exdsptch.c @ 509]:
:>:             :    80867408  cmp     dword ptr [ebp+8],edi ; cmp pExceptionRegistration, NULL
: :<            :    8086740b  je      nt!RtlUnwind+0xe6 (8086743e)
: ::            :   
: ::            :  nt!RtlUnwind+0xb5 [c:\wrk\wrk-v1.2\base\ntos\rtl\i386\exdsptch.c @ 509]:
: ::            :    8086740d  cmp     dword ptr [ebp+8],ebx ; cmp pExceptionRegistration,l_pExceptionRegistration
: ::<           :    80867410  jae     nt!RtlUnwind+0xe6 (8086743e)
: :::           :   
: :::           :  nt!RtlUnwind+0xba [c:\wrk\wrk-v1.2\base\ntos\rtl\i386\exdsptch.c @ 514]:
: :::           :    ; pExceptionRegistration 在 l_pExceptionRegistration 之下，即 pExceptionRegistration 超出了异常链表的栈范围
: :::           :    80867412  lea     eax,[ebp-32Ch] ; ebp-32Ch 是局部变量 l_ExceptionRecord
: :::           :    80867418  push    eax
: :::           :    80867419  mov     dword ptr [ebp-32Ch],0C0000029h ; l_ExceptionRecord.ExceptionCode = STATUS_INVALID_UNWIND_TARGET
: :::           :    80867423  mov     dword ptr [ebp-328h],1          ; l_ExceptionRecord.ExceptionFlags = EXCEPTION_NONCONTINUABLE (1)
: :::           :    8086742d  mov     dword ptr [ebp-324h],esi        ; l_ExceptionRecord.ExceptionRecord = &l_ExceptionRecord
: :::           :    80867433  mov     dword ptr [ebp-31Ch],edi        ; l_ExceptionRecord.NumberParameters = 0
: :::           :    80867439  call    nt!RtlRaiseException (80887a94)
: :::           :   
: :::           :  nt!RtlUnwind+0xe6 [c:\wrk\wrk-v1.2\base\ntos\rtl\i386\exdsptch.c @ 530]:
: >>>           :    8086743e  cmp     ebx,dword ptr [ebp-2D4h] ; cmp l_pExceptionRegistration, l_pLowLimit
:                :    80867444  lea     edi,[ebx+8] ; edi = l_pExceptionRegistration->scopetable_entry
:    <           :    80867447  jb      nt!RtlUnwind+0x162 (808674ba) ; l_pExceptionRegistration 低于线程栈底，跳到错误处理
:    :           :   
:    :           :  nt!RtlUnwind+0xf1 [c:\wrk\wrk-v1.2\base\ntos\rtl\i386\exdsptch.c @ 530]:
:    :           :    80867449  cmp     edi,dword ptr [ebp-2D8h] ; cmp l_pExceptionRegistration->scopetable_entry,l_pHighLimit
:    :<          :    8086744f  ja      nt!RtlUnwind+0x162 (808674ba) ; l_pExceptionRegistration 高于线程栈顶，跳到错误处理
:    ::          :   
:    ::          :  nt!RtlUnwind+0xf9 [c:\wrk\wrk-v1.2\base\ntos\rtl\i386\exdsptch.c @ 530]:
:    ::          :    ; l_pExceptionRegistration 处于合法栈中，检查它是否正确对齐
:    ::          :    80867451  test    bl,3 ; 检查 l_pExceptionRegistration 是否 4 字节对齐
:    :<          :    80867454  jne     nt!RtlUnwind+0x1a2 (808674fa) ; 没对齐，跳到错误处理
:    :::          :   
:    :::          :  nt!RtlUnwind+0x102 [c:\wrk\wrk-v1.2\base\ntos\rtl\i386\exdsptch.c @ 580]:
:    :::          :    ; 进行局部展开
:    :::          :    8086745a  push    dword ptr [ebx+4] ; l_pExceptionRegistration->Handler
:    :::          :    8086745d  lea     eax,[ebp-2DCh] ; 这里是局部变量 l_DispatchContext
:    :::          :    80867463  push    eax ;
:    :::          :    80867464  lea     eax,[ebp-2D0h] ; eax = &l_Context
:    :::          :    8086746a  push    eax ; l_Context
:    :::          :    8086746b  push    ebx ; l_pExceptionRegistration
:    :::          :    8086746c  push    esi ; &l_ExceptionRecord
:    :::          :    8086746d  call    nt!RtlpExecuteHandlerForUnwind (80887b54) ; 内部调用 l_pExceptionRegistration->Handler
:    :::          :    80867472  dec     eax
:    ::<          :    80867473  je      nt!RtlUnwind+0x156 (808674ae) ; 如果返回 ExceptionContinueSearch 则跳转
:    ::::         :   
:    ::::         :  nt!RtlUnwind+0x11d [c:\wrk\wrk-v1.2\base\ntos\rtl\i386\exdsptch.c @ 586]:
:    ::::         :    80867475  dec     eax
:    ::::         :    80867476  dec     eax
:    ::: <        :    80867477  je      nt!RtlUnwind+0x150 (808674a8) ; 如果返回 ExceptionCollidedUnwind 则跳转
:    ::: :        :   
:    ::: :        :  nt!RtlUnwind+0x121 [c:\wrk\wrk-v1.2\base\ntos\rtl\i386\exdsptch.c @ 621]:
:    ::: :        :    ; 返回 ExceptionContinueSearch 和 ExceptionCollidedUnwind 之外的非法值
:    ::: :        :    80867479  and     dword ptr [ebp-31Ch],0          ; l_ExceptionRecord.NumberParameters = 0
:    ::: :        :    80867480  lea     eax,[ebp-32Ch] ; eax = &l_ExceptionRecord
:    ::: :        :    80867486  push    eax
:    ::: :        :    80867487  mov     dword ptr [ebp-32Ch],0C0000026h ; l_ExceptionRecord.ExceptionCode = STATUS_INVALID_DISPOSITION
:    ::: :        :    80867491  mov     dword ptr [ebp-328h],1          ; l_ExceptionRecord.ExceptionFlags = EXCEPTION_NONCONTINUABLE (1)
:    ::: :        :    8086749b  mov     dword ptr [ebp-324h],esi        ; l_ExceptionRecord.ExceptionRecord = &l_ExceptionRecord
:    ::: :        :    808674a1  call    nt!RtlRaiseException (80887a94)
:    ::::: <      :    808674a6  jmp     nt!RtlUnwind+0x156 (808674ae)
:    ::::::::      :   
:    ::::::::      :  nt!RtlUnwind+0x150 [c:\wrk\wrk-v1.2\base\ntos\rtl\i386\exdsptch.c @ 608]:
:    ::::::::      :    ; RtlpExecuteHandlerForUnwind 返回 ExceptionCollidedUnwind，从 l_DispatchContext.RegistrationPointer 继续处理
:    ::::::>       :    808674a8  mov     ebx,dword ptr [ebp-2DCh] ; l_pExceptionRegistration = l_DispatchContext.RegistrationPointer
:    :::::         :   
:    :::::         :  nt!RtlUnwind+0x156 [c:\wrk\wrk-v1.2\base\ntos\rtl\i386\exdsptch.c @ 630]:
:    :::::         :    ; 局部展开完毕，从异常链表中摘除 l_pExceptionRegistration
:    :::>>         :    808674ae  mov     eax,ebx ; eax = l_pExceptionRegistration
:    :::           :    808674b0  mov     ebx,dword ptr [ebx] ; l_pExceptionRegistration = l_pExceptionRegistration.prev
:    :::           :    808674b2  push    eax
:    :::           :    808674b3  call    nt!RtlpUnlinkHandler (80887c10) ; 摘除
:    ::<           :    808674b8  jmp     nt!RtlUnwind+0x1cf (80867527)
:    ::::          :   
:    ::::          :  nt!RtlUnwind+0x162 [c:\wrk\wrk-v1.2\base\ntos\rtl\i386\exdsptch.c @ 540]:
:    ::::          :    ; l_pExceptionRegistration 并不处在当前线程栈中，这种情况不一定是出错，有可能当前正在执行 DPC。
:    ::::          :    ; 当时还需要检查是否对齐，不对齐就一定是出错
:    >::>         :    808674ba  test    bl,3 ; 检查 l_pExceptionRegistration 是否 4 字节对齐
:      ::<        :    808674bd  jne     nt!RtlUnwind+0x1a2 (808674fa)
:      :::         :   
:      :::         :  nt!RtlUnwind+0x167 [c:\wrk\wrk-v1.2\base\ntos\rtl\i386\exdsptch.c @ 540]:
:      :::         :    ; 如果当前正在执行 DPC，那 IRQL 一定不得低于 DISPATCH_LEVEL（难道不是应该一定等于 DISPATCH_LEVEL ？）
:      :::         :    808674bf  call    dword ptr [nt!_imp__KeGetCurrentIrql (8080102c)]
:      :::         :    808674c5  cmp     al,2 ; DISPATCH_LEVEL (2)
:      :<          :    808674c7  jb      nt!RtlUnwind+0x1a2 (808674fa) ; 低于 DISPATCH_LEVEL，跳到错误处理点
:      ::::        :   
:      ::::        :  nt!RtlUnwind+0x171 [c:\wrk\wrk-v1.2\base\ntos\rtl\i386\exdsptch.c @ 542]:
:      ::::        :    808674c9  mov     eax,dword ptr fs:[00000020h] ; eax = 当前 CPU 的 KPCR::Prcb
:      ::::        :    808674cf  cmp     byte ptr [eax+95Ah],0 ; KPCR::Prcb->DpcRoutineActive (BOOLEAN 类型)
:      ::::        :    808674d6  mov     ecx,dword ptr [eax+948h] ; ecx = KPCR::Prcb->DpcStack
:      ::::: <     :    808674dc  je      nt!RtlUnwind+0x1a2 (808674fa) ; 当前不是在执行 DPC，跳到错误处理点
:      ::::::      :   
:      ::::::      :  nt!RtlUnwind+0x186 [c:\wrk\wrk-v1.2\base\ntos\rtl\i386\exdsptch.c @ 547]:
:      ::::::      :    ; KPCR::Prcb->DpcRoutineActive 为 TRUE，当前异常是由 DpcRoutine 触发
:      ::::::      :    808674de  cmp     edi,ecx ; 比较 l_pExceptionRegistration->scopetable_entry 和 DPC 栈顶
:      ::::: <     :    808674e0  ja      nt!RtlUnwind+0x1a2 (808674fa) ; 高出 DPC 栈顶，跳到错误处理点
:      :::::::     :   
:      :::::::     :  nt!RtlUnwind+0x18a [c:\wrk\wrk-v1.2\base\ntos\rtl\i386\exdsptch.c @ 547]:
:      :::::::     :    808674e2  lea     eax,[ecx-3000h] ; eax = KPCR::Prcb->DpcStack - KERNEL_STACK_SIZE, 即 DPC 栈底
:      :::::::     :    808674e8  cmp     ebx,eax ; 比较 l_pExceptionRegistration 和 DPC 栈底
:      ::::::: <   :    808674ea  jb      nt!RtlUnwind+0x1a2 (808674fa) ; 低于 DPC 栈底，跳到错误处理点
:      :::::::     :   
:      :::::::     :  nt!RtlUnwind+0x194 [c:\wrk\wrk-v1.2\base\ntos\rtl\i386\exdsptch.c @ 555]:
:      :::::::     :    808674ec  mov     dword ptr [ebp-2D8h],ecx ; l_pHighLimit = DPC 栈顶
:      :::::::     :    808674f2  mov     dword ptr [ebp-2D4h],eax ; l_pLowLimit = DPC 栈底
:      :::::: <    :    808674f8  jmp     nt!RtlUnwind+0x1cf (80867527)
:      :::::::::   
:      :::::::::   nt!RtlUnwind+0x1a2 [c:\wrk\wrk-v1.2\base\ntos\rtl\i386\exdsptch.c @ 564]:
:      :::::::::    ; 栈错误
:      :>>>>> >:     808674fa  and     dword ptr [ebp-31Ch],0
:       :     ::    80867501  lea     eax,[ebp-32Ch]   ; eax = &l_ExceptionRecord
:       :     ::    80867507  push    eax
:       :     ::    80867508  mov     dword ptr [ebp-32Ch],0C0000028h ; l_ExceptionRecord.ExceptionCode = STATUS_BAD_STACK
:       :     ::    80867512  mov     dword ptr [ebp-328h],1          ; l_ExceptionRecord.ExceptionFlags = EXCEPTION_NONCONTINUABLE (1)
:       :     ::    8086751c  mov     dword ptr [ebp-324h],esi        ; l_ExceptionRecord.ExceptionRecord = &l_ExceptionRecord
:       :     ::    80867522  call    nt!RtlRaiseException (80887a94)
:       :     ::   
:       :     ::  nt!RtlUnwind+0x1cf [c:\wrk\wrk-v1.2\base\ntos\rtl\i386\exdsptch.c @ 493]:
:       >     >:    80867527  xor     edi,edi
:                :   
:                :  nt!RtlUnwind+0x1d1 [c:\wrk\wrk-v1.2\base\ntos\rtl\i386\exdsptch.c @ 493]:
>               :    80867529  cmp     ebx,0FFFFFFFFh ; cmp l_pExceptionRegistration, EXCEPTION_CHAIN_END
             <    8086752c  jne     nt!RtlUnwind+0x9c (808673f4)
             
               nt!RtlUnwind+0x1da [c:\wrk\wrk-v1.2\base\ntos\rtl\i386\exdsptch.c @ 647]:
                  80867532  cmp     dword ptr [ebp+8],0FFFFFFFFh ; cmp pExceptionRegistration, EXCEPTION_CHAIN_END
                  80867536  pop     ebx
                  80867537  lea     eax,[ebp-2D0h] ; eax = &l_Context
                  8086753d  push    edi ; bTestAlert = FALSE 或 bFirstChance = FALSE
                  8086753e  push    eax
<                 8086753f  jne     nt!RtlUnwind+0x1f0 (80867548)
:                 
:                 nt!RtlUnwind+0x1e9 [c:\wrk\wrk-v1.2\base\ntos\rtl\i386\exdsptch.c @ 656]:
:                   ; 展开整个异常链表
:                   80867541  call    nt!ZwContinue (8082c0b8)
:<                  80867546  jmp     nt!RtlUnwind+0x1f6 (8086754e)
::                 
::                nt!RtlUnwind+0x1f0 [c:\wrk\wrk-v1.2\base\ntos\rtl\i386\exdsptch.c @ 667]:
::                  ; pExceptionRegistration 等于 EXCEPTION_CHAIN_END。
::                  ; 没有理解这里触发异常是为什么，但是不妨碍分析异常处理流程。
>:                  80867548  push    esi ; &l_ExceptionRecord
 :                  80867549  call    nt!ZwRaiseException (8082ccd4)
 :                 
 :                nt!RtlUnwind+0x1f6 [c:\wrk\wrk-v1.2\base\ntos\rtl\i386\exdsptch.c @ 671]:
 >                  8086754e  mov     ecx,dword ptr [ebp-4]
                  80867551  pop     edi
                  80867552  pop     esi
                  80867553  call    nt!__security_check_cookie (80874e87)
                  80867558  leave
                  80867559  ret     10h

        kd> uf 80887be9 ; RtlpExecuteHandlerForUnwind 的异常处理函数 
        nt!ExecuteHandler2+0x61 [C:\wrk\wrk-v1.2\base\ntos\rtl\i386\xcptmisc.asm @ 329]:
          80887be9  mov     ecx,dword ptr [esp+4] ; ecx = pExceptionRecord
          80887bed  test    dword ptr [ecx+4],6   ; test pExceptionRecord->ExceptionFlags, EXCEPTION_UNWINDING | EXCEPTION_EXIT_UNWIND
          80887bf4  mov     eax,1 ; eax = ExceptionContinueSearch
          80887bf9  je      nt!ExecuteHandler2+0x85 (80887c0d)

        nt!ExecuteHandler2+0x73 [C:\wrk\wrk-v1.2\base\ntos\rtl\i386\xcptmisc.asm @ 341]:
          ; 展开过程中 Handler 触发异常
          80887bfb  mov     ecx,dword ptr [esp+8]   ; ecx = pExceptionRegistration
          80887bff  mov     edx,dword ptr [esp+10h] ; edx = pDispatcherContext
          80887c03  mov     eax,dword ptr [ecx+8]   ; eax = pExceptionRegistration->prev
          80887c06  mov     dword ptr [edx],eax     ; pDispatcherContext->RegistrationPointer = pExceptionRegistration->prev，
                                                   ; 然后 pDispatcherContext->RegistrationPointer 就是展开过程中正在被展开的
                                                   ; 那个 EXCEPTION_REGISTRATION_RECORD
          80887c08  mov     eax,3 ; eax = ExceptionCollidedUnwind

        nt!ExecuteHandler2+0x85 [C:\wrk\wrk-v1.2\base\ntos\rtl\i386\xcptmisc.asm @ 349]:
          80887c0d  ret     10h


    kd> uf RtlpExecuteHandlerForUnwind
        nt!RtlpExecuteHandlerForUnwind [C:\wrk\wrk-v1.2\base\ntos\rtl\i386\xcptmisc.asm @ 142]:
          80887b54  mov     edx,offset nt!ExecuteHandler2+0x61 (80887be9)
          80887b59  lea     ecx,[ecx]
          80887b5c  push    ebx
          80887b5d  push    esi
          80887b5e  push    edi
          80887b5f  xor     eax,eax
          80887b61  xor     ebx,ebx
          80887b63  xor     esi,esi
          80887b65  xor     edi,edi
          80887b67  push    dword ptr [esp+20h] ; pExceptionRoutine
          80887b6b  push    dword ptr [esp+20h] ; pDispatcherContext
          80887b6f  push    dword ptr [esp+20h] ; pContext
          80887b73  push    dword ptr [esp+20h] ; pExceptionRegistration
          80887b77  push    dword ptr [esp+20h] ; pExceptionRecord
          80887b7b  call    nt!ExecuteHandler2 (80887b88)
          80887b80  pop     edi
          80887b81  pop     esi
          80887b82  pop     ebx
          80887b83  ret     14h

    kd> uf nt!ExecuteHandler2
        nt!ExecuteHandler2 [C:\wrk\wrk-v1.2\base\ntos\rtl\i386\xcptmisc.asm @ 188]:
          80887b88  push    ebp
          80887b89  mov     ebp,esp
          80887b8b  push    dword ptr [ebp+0Ch] ; pExceptionRegistration
          80887b8e  push    edx               ; _EXCEPTION_REGISTRATION_RECORD::Handler (nt!ExecuteHandler2+0x61 (80887be9))
          80887b8f  push    dword ptr fs:[0]    ; _EXCEPTION_REGISTRATION_RECORD::Next
          80887b96  mov     dword ptr fs:[0],esp; 注意：这里使用的是原始版本的异常处理
          80887b9d  push    dword ptr [ebp+14h] ; pDispatcherContext
          80887ba0  push    dword ptr [ebp+10h] ; pContext
          80887ba3  push    dword ptr [ebp+0Ch] ; pExceptionRegistration
          80887ba6  push    dword ptr [ebp+8]   ; pExceptionRecord
          80887ba9  mov     ecx,dword ptr [ebp+18h] ; ecx = pExceptionRoutine
          80887bac  call    ecx ; pExceptionRoutine(...)，即 pExceptionRegistration->handler(...)
          80887bae  mov     esp,dword ptr fs:[0]
          80887bb5  pop     dword ptr fs:[0]    ; 摘掉当前 _EXCEPTION_REGISTRATION_RECORD
          80887bbc  mov     esp,ebp
          80887bbe  pop     ebp
          80887bbf  ret     14h

```



代码不长，主要功能也不复杂：从异常链表头开始遍历，一直遍历到指定 `EXCEPTION_REGISTRATION_RECORD`，对每个遍历到的 `EXCEPTION_REGISTRATION_RECORD`，执行 `RtlpExecuteHandlerForUnwind` 进行局部展开。

这段代码里有一个细节我没想明白，或者我想复杂了。在 `nt!RtlUnwind` 地址 `80867401` 处，当展开到指定 `EXCEPTION_REGISTRATION` 后，`RtlUnwind` 通过 `ZwContinue` 返回，而不是使用 `ret` 指令。通过静态分析和动态分析我都没有找到用 `ZwContinue` 的好处，或者不可替代的原因。如果有朋友有不同的结论，请分享一下。

在分析全局展开时发生一件很的事，我反汇编完成，梳理流程的时候，总感觉“这个逻辑怎么这么熟悉，貌似在哪见过~ 难道 `wrk` 里有源码？”，翻开 `wrk`，果然有…… 不过话说回来，在反汇编分析过程让我对一些细节理解的更深刻了（也只能这么安慰自己了……）。

下面我们来看看局部展开。

在前面讲 `PassThrough!_except_handler4` 时，有提到该函数既负责处理异常也负责局部展开。其区分功能的标志就是判断 `EXCEPTION_RECORD::ExceptionFlags` 是否包含 `EXCEPTION_UNWIND` 标志位。可以参考 `PassThrough!_except_handler4` 中地址为 `f87203dc` 的指令：

```
f87203dc  test    byte  ptr  [eax+4],66h      ; pExceptionRecord->ExceptionFlags & EXCEPTION_UNWIND，判断是异常处理过程还是展开过程
```

该标志是在 `RtlUnwind` 中被设置的，可以参考 `RtlUnwind` 中地址为 `808673b7` 和 `808673bd` 处的指令：

```
808673b7  or      dword  ptr [esi+4],2  ; l_ExceptionRecord.ExceptionFlags |= EXCEPTION_UNWINDING (0x2)

808673bd  or      dword  ptr [esi+4],6  ; l_ExceptionRecord.ExceptionFlags |= EXCEPTION_UNWINDING | EXCEPTION_EXIT_UNWIND (0x2 | 0x4)
```

### 反汇编代码

首先是我整理的 `_EH4_LocalUnwind` 的原型：

```
VOID fastcall _EH4_LocalUnwind(
    PEXCEPTION_REGISTRATION pExceptionRegistartion,
    ULONG ulUntilTryLevel
)
```

代码:

```
    kd> uf PassThrough!_EH4_LocalUnwind
        PassThrough!_EH4_LocalUnwind [d:\\5359\\minkernel\\crts\\crtw32\\misc\\i386\\exsup4.asm @ 529]:
          f8720614  push    ebp
          f8720615  mov     ebp,dword  ptr  [esp+8]  ;  ebp = pExceptionRegistartion->_ebp
          f8720619  push    edx  ;  ulUntilTryLevel
          f872061a  push    ecx  ;  pExceptionRegistartion
          f872061b  push    dword  ptr  [esp+14h]  ;  push __security_cookie
          f872061f  call    PassThrough!_local_unwind4  (f87204ec)
          f8720624  add     esp,0Ch
          f8720627  pop     ebp
          f8720628  ret     8

    PassThrough!_EH4_LocalUnwind 的原型：
       VOID local_unwind4(
           PEXCEPTION_REGSTRATION pExceptionRegstration,
           ULONG ulUntilTryLevel
           )

    kd> uf PassThrough!_local_unwind4
      PassThrough!_local_unwind4 [d:\\5359\\minkernel\\crts\\crtw32\\misc\\i386\\exsup4.asm @ 177]:
        f87204ec  push    ebx
        f87204ed  push    esi
        f87204ee  push    edi
        f87204ef  mov     edx,dword  ptr  [esp+10h]  ;  edx = __security_cookie
        f87204f3  mov     eax,dword  ptr  [esp+14h]  ;  eax = pExceptionRegistartion
        f87204f7  mov     ecx,dword  ptr  [esp+18h]  ;  ecx = ulUntilTryLevel
        f87204fb  push    ebp
        f87204fc  push    edx  ;  __security_cookie
        f87204fd  push    eax  ;  pExceptionRegistartion
        f87204fe  push    ecx  ;  ulUntilTryLevel
        f87204ff  push    ecx  ;  这里压入的值后续被 f8720513 处的指令修改
        f8720500  push    offset PassThrough!_unwind_handler4  (f872056f)  ;  _EXCEPTION_REGISTRATION_RECORD::handler
        f8720505  push    dword  ptr  fs:[0]  ;  _EXCEPTION_REGISTRATION_RECORD::prev, 注意这里使用的是原始版本
        f872050c  mov     eax,dword  ptr  [PassThrough!__security_cookie  (f87220b0)]
        f8720511  xor     eax,esp  ;  这里是用 esp 来异或，而 esp 此时指向新 _EXCEPTION_REGISTRATION_RECORD
        f8720513  mov     dword  ptr  [esp+8],eax  ;  [esp+8] 被赋予加密后的 __security_cookie
        f8720517  mov     dword  ptr  fs:[0],esp
       
      PassThrough!_local_unwind4+0x32 [d:\\5359\\minkernel\\crts\\crtw32\\misc\\i386\\exsup4.asm @ 209]:
   >>   f872051e  mov     eax,dword  ptr  [esp+30h]  ;  eax = pExceptionRegistartion
   ::   f8720522  mov     ebx,dword  ptr  [eax+8]   ;  ebx = pExceptionRegistration->scopetable, 将 ebx 假称为 l_pCurScopeEntry
   ::   f8720525  mov     ecx,dword  ptr  [esp+2Ch]  ;  ecx = &__security_cookie
   ::   f8720529  xor     ebx,dword  ptr  [ecx]     ;  解密 scopetable
   ::   f872052b  mov     esi,dword  ptr  [eax+0Ch]  ;  esi = pExceptionRegistration->trylevel
   ::   f872052e  cmp     esi,0FFFFFFFEh  ;  cmp pExceptionRegistration->trylevel,TRYLEVEL_NONE
<  ::   f8720531  je      PassThrough!_local_unwind4+0x75  (f8720561)  ;  pExceptionRegistration->trylevel 等于 TRYLEVEL_NONE，无需遍历，返回
:  :: 
:  :: PassThrough!_local_unwind4+0x47 [d:\\5359\\minkernel\\crts\\crtw32\\misc\\i386\\exsup4.asm @ 216]:
:  ::   f8720533  mov     edx,dword  ptr  [esp+34h]  ;  edx = ulUntilTryLevel
:  ::   f8720537  cmp     edx,0FFFFFFFEh  ;  cmp ulUntilTryLevel, TRYLEVEL_NONE
:< ::   f872053a  je      PassThrough!_local_unwind4+0x54  (f8720540)
:: :: 
:: :: PassThrough!_local_unwind4+0x50 [d:\\5359\\minkernel\\crts\\crtw32\\misc\\i386\\exsup4.asm @ 219]:
:: ::   f872053c  cmp     esi,edx  ;  cmp pExceptionRegistration->trylevel, ulUntilTryLevel
::<::   f872053e  jbe     PassThrough!_local_unwind4+0x75  (f8720561)  ;  已经遍历到指定 scopetable_entry 了，返回
::::: 
::::: PassThrough!_local_unwind4+0x54 [d:\\5359\\minkernel\\crts\\crtw32\\misc\\i386\\exsup4.asm @ 221]:
:>:::   f8720540  lea     esi,[esi+esi*2]
: :::   f8720543  lea     ebx,[ebx+esi*4+10h]  ;  ebx - pExceptionRegistration->scopetable[pExceptionRegistration->trylevel]
: :::   f8720547  mov     ecx,dword  ptr  [ebx]  ;  ecx = l_pCurScopeEntry->previousTryLevel
: :::   f8720549  mov     dword  ptr  [eax+0Ch],ecx  ;  pExceptionRegistration->trylevel = ebx->l_pCurScopeEntry->previousTryLevel
: :::   f872054c  cmp     dword  ptr  [ebx+4],0  ;  cmp pExceptionRegistration->lpfnFilter,NULL
: <:   f8720550  jne     PassThrough!_local_unwind4+0x32  (f872051e)
: : : 
: : : PassThrough!_local_unwind4+0x66 [d:\\5359\\minkernel\\crts\\crtw32\\misc\\i386\\exsup4.asm @ 240]:
: : :   f8720552  mov     ecx,1
: : :   f8720557  mov     eax,dword  ptr  [ebx+8]  ;  eax = pExceptionRegistration->lpfnHandler
: : :   f872055a  call    PassThrough!_NLG_Call  (f8720630)  ;  调用 pExceptionRegistration->lpfnHandler
: : <   f872055f  jmp     PassThrough!_local_unwind4+0x32  (f872051e)
: :    
: :   PassThrough!_local_unwind4+0x75 [d:\\5359\\minkernel\\crts\\crtw32\\misc\\i386\\exsup4.asm @ 248]:
> >     f8720561  pop     dword  ptr  fs:[0]
        f8720568  add     esp,18h
        f872056b  pop     edi
        f872056c  pop     esi
        f872056d  pop     ebx
        f872056e  ret
       
        kd> uf PassThrough!_NLG_Call
        PassThrough!_NLG_Call [d:\\5359\\minkernel\\crts\\crtw32\\misc\\i386\\nlgsupp.asm @ 120]:
          f8720630  call    eax
          f8720632  ret
```

`PassThrough!_EH4_LocalUnwind` 中 `f8720500` 处的指令用到的异常处理函数 `PassThrough!_unwind_handler4` 的反汇编代码：

代码:



```
    kd> uf PassThrough!_unwind_handler4
        PassThrough!_unwind_handler4 [d:\5359\minkernel\crts\crtw32\misc\i386\exsup4.asm @ 294]:
          f872056f  mov     ecx,dword ptr [esp+4] ; ecx = pExceptionRecord
          f8720573  test    dword ptr [ecx+4],6   ; test pExceptionRecord->ExceptionFlags, EXCEPTION_UNWINDING | EXCEPTION_EXIT_UNWIND
          f872057a  mov     eax,1 ; eax = ExceptionContinueSearch (1)
          f872057f  je      PassThrough!_unwind_handler4+0x45 (f87205b4)

        PassThrough!_unwind_handler4+0x12 [d:\5359\minkernel\crts\crtw32\misc\i386\exsup4.asm @ 307]:
          ; 在展开过程中又触发了异常，那么处理该新异常（实际操作只是做展开，即清理掉该异常占用的资源，并没有真正处理该异常）
          f8720581  mov     eax,dword ptr [esp+8] ; eax = pExceptionRegistration
          f8720585  mov     ecx,dword ptr [eax+8] ; ecx = __security_cookie, 注意这里不是 __pExceptionRegistration->scopetable
          f8720588  xor     ecx,eax ; 这里用 pExceptionRegistration 解密，PassThrough!_local_unwind4 中也正是用它进行加密的
                                        ; 参考 f8720511 处的指令
          f872058a  call    PassThrough!__security_check_cookie (f8720638)
          f872058f  push    ebp
          f8720590  mov     ebp,dword ptr [eax+18h] ; eax+18h 是被 PassThrough!_local_unwind4 压入栈的 ebp，
                                                    ; 参考 f87204fb 处的指令
          f8720593  push    dword ptr [eax+0Ch] ; push ulUntilTryLevel
          f8720596  push    dword ptr [eax+10h] ; pExceptionRegistartion
          f8720599  push    dword ptr [eax+14h] ; __security_cookie
          f872059c  call    PassThrough!_local_unwind4 (f87204ec)
          f87205a1  add     esp,0Ch
          f87205a4  pop     ebp
          f87205a5  mov     eax,dword ptr [esp+8] ; eax = pExceptionRegistration
          f87205a9  mov     edx,dword ptr [esp+10h] ; edx = pDispatcherContext （调整 pDispatcherContext，改变遍历顺序）
          f87205ad  mov     dword ptr [edx],eax   ; pDispatcherContext->RegistrationPointer = pExceptionRegistration
          f87205af  mov     eax,3 ; eax = ExceptionCollidedUnwind (3)

        PassThrough!_unwind_handler4+0x45 [d:\5359\minkernel\crts\crtw32\misc\i386\exsup4.asm @ 336]:
          f87205b4  ret

```

到这里概要流程就讲完了。在处理异常和展开过程中多处涉及到遍历操作，咱们来总结一下这些遍历操作。

1. 在异常处理过程中，每个被"卷入是非"的异常都至少会遍历异常链表两次（如果发生嵌套异常，比如在展开过程中 `EXCEPTION_REGISTRATION_RECORD::Handler` 又触发异常，则会遍历更多次。不过这也可以算作是一个新异常了。看如何理解。）。

   一次是在 `RtlDispatchException` 中，遍历的目的是找到愿意处理该异常的 `_EXCEPTION_REGISTRATION_RECORD`。

   另一次是在展开过程中、`RtlUnwind` 函数内，遍历的目录是为了对每个遍历到的 `EXCEPTION_REGISTRATION_RECORD` 进行局部展开。

2. 同样的，每个被"卷入是非"的异常的 `scopetable` 也会被遍历至少两次，

   一次是在 `modulename!_except_handler?` 中，遍历目的也是找到愿意处理该异常的 `scopetable_entry`。

   另一次是在展开过程中、`_local_unwind4` 函数内，遍历的目的是找到所有指定范围内的 `scopetable_entry::lpfnFilter` 为 `NULL` 的 `scopetable_entry`，调用它们的 `lpfnHandler` （即 `__finally` 处理块）。

   在展开过程中，`__finally` 代码块会被执行，在执行过程中有可能触发新的异常，增强版通过返回 `ExceptionCollidedUnwind` (3) 来标识这种情况（参考 `PassThrough!_unwind_handler4` 中 `f87205af` 处的指令）。咱来回顾下这类返回值：

   代码:

```
    typedef enum _EXCEPTION_DISPOSITION {
        ExceptionContinueExecution,
        ExceptionContinueSearch,
        ExceptionNestedException,
        ExceptionCollidedUnwind
    } EXCEPTION_DISPOSITION;

```

   前面的代码中已经展示了上述 4 中的三种：

   `ExceptionContinueExecution` 表示继续执行（异常已经被修复）；

   `ExceptionContinueSearch` 表示继续搜索（当前搜索到的异常注册信息不处理）；

   `ExceptionCollidedUnwind` 表示在展开过程中再次触发异常。

   `ExceptionNestedException` 呢？到目前咱们还没有遇到过，什么情况会用到它？

   从字面上看 `ExceptionNestedException` 大概意思是“嵌套的异常”，是不是可以理解为“处理异常过程中再次触发的异常”？比如类似于 `ExceptionCollidedUnwind`，只不过 `ExceptionCollidedUnwind` 是在展开过程。而 `ExceptionNestedException` 是在处理异常过程中？

   咱们顺着这个思路去寻找，首先来看 `PassThrough!_except_handler4`，它是异常处理的入口了。处理和展开都是它负责的。可是翻遍了它和它的工具函数的反汇编码也没有找到它直接返回或者通过注册异常处理信息来间接返回。于是我决定继续向上层即调用者方向搜寻，于是找到了如下汇编码：

   代码:

```
        kd> uf RtlpExecuteHandlerForException
        nt!RtlpExecuteHandlerForException [C:\wrk\wrk-v1.2\base\ntos\rtl\i386\xcptmisc.asm @ 87]:
          80887b4c  mov     edx,offset nt!ExecuteHandler2+0x3a (80887bc2)
          80887b51  jmp     nt!ExecuteHandler (80887b5c)

        nt!ExecuteHandler [C:\wrk\wrk-v1.2\base\ntos\rtl\i386\xcptmisc.asm @ 160]:
          80887b5c  push    ebx
          80887b5d  push    esi
          80887b5e  push    edi
          80887b5f  xor     eax,eax
          80887b61  xor     ebx,ebx
          80887b63  xor     esi,esi
          80887b65  xor     edi,edi
          80887b67  push    dword ptr [esp+20h]
          80887b6b  push    dword ptr [esp+20h]
          80887b6f  push    dword ptr [esp+20h]
          80887b73  push    dword ptr [esp+20h]
          80887b77  push    dword ptr [esp+20h]
          80887b7b  call    nt!ExecuteHandler2 (80887b88)
          80887b80  pop     edi
          80887b81  pop     esi
          80887b82  pop     ebx
          80887b83  ret     14h

        kd> uf ExecuteHandler2
        nt!ExecuteHandler2 [C:\wrk\wrk-v1.2\base\ntos\rtl\i386\xcptmisc.asm @ 188]:
          80887b88  push    ebp
          80887b89  mov     ebp,esp
          80887b8b  push    dword ptr [ebp+0Ch]
          80887b8e  push    edx
          80887b8f  push    dword ptr fs:[0]
          80887b96  mov     dword ptr fs:[0],esp ; 注册新的异常处理结构
          80887b9d  push    dword ptr [ebp+14h]
          80887ba0  push    dword ptr [ebp+10h]
          80887ba3  push    dword ptr [ebp+0Ch]
          80887ba6  push    dword ptr [ebp+8]
          80887ba9  mov     ecx,dword ptr [ebp+18h]
          80887bac  call    ecx
          80887bae  mov     esp,dword ptr fs:[0]
          80887bb5  pop     dword ptr fs:[0]
          80887bbc  mov     esp,ebp
          80887bbe  pop     ebp
          80887bbf  ret     14h

        kd> uf 80887bc2 ; ExecuteHandler2 使用的异常处理函数
        nt!ExecuteHandler2+0x3a [C:\wrk\wrk-v1.2\base\ntos\rtl\i386\xcptmisc.asm @ 268]:
          80887bc2  mov     ecx,dword ptr [esp+4]
          80887bc6  test    dword ptr [ecx+4],6
          80887bcd  mov     eax,1 ; eax = ExceptionContinueSearch (1)
          80887bd2  jne     nt!ExecuteHandler2+0x5e (80887be6)

        nt!ExecuteHandler2+0x4c [C:\wrk\wrk-v1.2\base\ntos\rtl\i386\xcptmisc.asm @ 279]:
          80887bd4  mov     ecx,dword ptr [esp+8]
          80887bd8  mov     edx,dword ptr [esp+10h]
          80887bdc  mov     eax,dword ptr [ecx+8]
          80887bdf  mov     dword ptr [edx],eax
          80887be1  mov     eax,2 ; eax = ExceptionNestedException (2)

        nt!ExecuteHandler2+0x5e [C:\wrk\wrk-v1.2\base\ntos\rtl\i386\xcptmisc.asm @ 287]:
          80887be6  ret     10h

```

   `RtlpExecuteHandlerForException` 函数是在 `RtlDispatchException` 中被调用。`RtlDispatchException` 在遍历异常链过程中并不直接调用 `EXCEPTION_REGISTRATION_RECORD::Handler`，而是通过 `RtlpExecuteHandlerForException` 来间接调用。`RtlpExecuteHandlerForException` 通过 `ExecuteHandler2` 建立一个异常处理块。

   `PassThrough!_except_handler4` 处理异常的过程中，如果再次触发异常，则会由地址为 `80887bc2` 的异常处理函数返回 `ExceptionNestedException`。因为这几个函数都很简单，而且跟之前分析的函数很类似，不再赘述。

   分析完这些，我有个疑问还是没有解开，我个人一直很奇怪增强版的使用方式为何不能这样：

```
        __try

        {

        }

        __except()

        {

        }

        __finally

        {

        }

```

   其中 `__except` 过滤块和处理块可以省略。

   但是很遗憾，MSC 中 `__except` 和 `__finally` 只能选其一。（当然，咱们可以用双层 `__try` 来实现同样的效果，但是总感觉不太爽，特别是会导致代码块被迫缩进两次）

   在分析的过程中我也发现我希望的这种方式更合理一些，`__except` 负责处理异常，在处理异常代码中被执行。如果没有处理异常，那么在展开过程中 `__finally` 代码块被执行，做一些清理操作。这样两者都存在，各负责各的，不是更好吗？不知道是不是什么历史原因。

   还有一个地方我也觉得不太完美。我们分析原始版本的时候发现，原始版本自身并没有直接使用展开，`RtlDispatchException` 等异常处理函数并没有直接调用 `RtlUnwind`。后者实际上是在增强版中才用到。我个人感觉这种模型并不完美，原始版本并没有自成体系，而是与增强版纠结在一起，没有很好的分层。

   另外，在实际应用中 `SEH` 机制可能会导致很难分析的内存泄漏。我们来看一个例子，

   调用流程：`func1` -> `func2`

   其中，`func1` 的代码如下：

```
        VOID Func1()

        {

            __try

            {

                Func2();

            }

            __except(EXCEPTION_EXECUTE_HANDLER)

            {

                // 一些善后处理
```






```
kd> uf Ntfs!_except_handler3
;nt!_except_handler3 [d:\dnsrv\base\crts\crtw32\misc\i386\exsup3.asm @ 172]:
   80872c00  push    ebp
   80872c01  mov     ebp,esp
   80872c03  sub     esp,8
   80872c06  push    ebx
   80872c07  push    esi
   80872c08  push    edi
   80872c09  push    ebp
   80872c0a  cld
   80872c0b  mov     ebx,dword ptr [ebp+0Ch] ; pExceptionRegistration
   80872c0e  mov     eax,dword ptr [ebp+8]   ; pExceptionRecord
   80872c11  test    dword ptr [eax+4],6     ; test pExceptionRecord->ExceptionFlags, (EXCEPTION_UNWINDING | EXCEPTION_EXIT_UNWIND)
<     80872c18  jne     nt!_except_handler3+0xc9 (80872cc9)
:
:
:
:  ;nt!_except_handler3+0x1e [d:\dnsrv\base\crts\crtw32\misc\i386\exsup3.asm @ 202]:
:  ; ebp-8 和 ebp-4 是一个类型为 PEXCEPTION_POINTERS 的结构体，称之为 l_ExceptionPointers
:  80872c1e  mov     dword ptr [ebp-8],eax   ; l_ExceptionPointers->ExceptionRecord = pExceptionRecord
:  80872c21  mov     eax,dword ptr [ebp+10h] ; eax = pContext
:  80872c24  mov     dword ptr [ebp-4],eax   ; l_ExceptionPointers->ContextRecord = pContext
:  80872c27  lea     eax,[ebp-8]             ; eax = lException
:  80872c2a  mov     dword ptr [ebx-4],eax   ; ebx-4 指向 pExceptionRegistration 所在栈上类型为 PEXCEPTION_POINTERS 的变量
:                         ; 具体栈的构造形式请参考当时建立 pExceptionRegistration 的代码
:                         ; 这里是赋值给该 PEXCEPTION_POINTERS 变量，以提供给 GetExceptionInformation 和 GetExceptionCode 使用
:  80872c2d  mov     esi,dword ptr [ebx+0Ch] ; esi = pExceptionRegistration->trylevel
:  80872c30  mov     edi,dword ptr [ebx+8]   ; edi = pExceptionRegistration->scopetable
:  80872c33  push    ebx
:  80872c34  call    nt!_ValidateEH3RN (8087cde8)
:  80872c39  add     esp,4
:  80872c3c  or      eax,eax
:<  80872c3e  je      nt!_except_handler3+0xbb (80872cbb)
::
::
::   ;nt!_except_handler3+0x40 [d:\dnsrv\base\crts\crtw32\misc\i386\exsup3.asm @ 218]:
::  >  80872c40  cmp     esi,0FFFFFFFFh  ; cmp pExceptionRegistration->trylevel, TRYLEVEL_NONE
::<  :  80872c43  je      nt!_except_handler3+0xc2 (80872cc2)
:::  :
:::  : ;nt!_except_handler3+0x45 [d:\dnsrv\base\crts\crtw32\misc\i386\exsup3.asm @ 220]:
:::  :  80872c45  lea     ecx,[esi+esi*2]         ; esi *= 3; 下面要将 eis*4，总共 esi*12，这是因为 scopetable_entry 大小是 12
:::  :  80872c48  mov     eax,dword ptr [edi+ecx*4+4]  ; eax = pExceptionRegistration->scopetable[i].lpfnFilter
:::  :  80872c4c  or      eax,eax
::<  :  80872c4e  je      nt!_except_handler3+0xa9 (80872ca9) ; lpfnFilter 为 NULL 则跳转
::::  :
::::  : ;nt!_except_handler3+0x50 [d:\dnsrv\base\crts\crtw32\misc\i386\exsup3.asm @ 226]:
::::  :  80872c50  push    esi
::::  :  80872c51  push    ebp
::::  :  80872c52  lea     ebp,[ebx+10h] ; ebp = pExceptionRegistration->_ebp
::::  :  80872c55  xor     ebx,ebx
::::  :  80872c57  xor     ecx,ecx
::::  :  80872c59  xor     edx,edx
::::  :  80872c5b  xor     esi,esi
::::  :  80872c5d  xor     edi,edi
::::  :  80872c5f  call    eax           ; pExceptionRegistration->scopetable[i].lpfnFilter()
::::  :  80872c61  pop     ebp
::::  :  80872c62  pop     esi
::::  :  80872c63  mov     ebx,dword ptr [ebp+0Ch] ; ebx = pExceptionRegistration
::::  :  80872c66  or      eax,eax
::::< :  80872c68  je      nt!_except_handler3+0xa9 (80872ca9) ; EXCEPTION_CONTINUE_SEARCH
::::: :
::::: : ;nt!_except_handler3+0x6a [d:\dnsrv\base\crts\crtw32\misc\i386\exsup3.asm @ 245]:
::::: :  ; 如果 lpfnFilter 返回 EXCEPTION_CONTINUE_EXECUTION，跳过下面的展开操作
::::<:  80872c6a  js      nt!_except_handler3+0xb4 (80872cb4) ; EXCEPTION_CONTINUE_EXECUTION
::::::
:::::: ;nt!_except_handler3+0x6c [d:\dnsrv\base\crts\crtw32\misc\i386\exsup3.asm @ 249]:
:::::: ; lpfnFilter 返回 EXCEPTION_EXECUTE_HANDLER，开始展开
::::::  80872c6c  mov     edi,dword ptr [ebx+8] ; edi = pExceptionRegistration->scopetable
::::::  80872c6f  push    ebx
::::::  80872c70  call    nt!__global_unwind2 (80872520)
::::::  80872c75  add     esp,4
::::::
::::::  80872c78  lea     ebp,[ebx+10h] ; ebp = pExceptionRegistration->_ebp
::::::  80872c7b  push    esi ; 展开到当前 trylevel 为止（不包含本 scopetable_entry）
::::::  80872c7c  push    ebx
::::::  80872c7d  call    nt!__local_unwind2 (8087257b)
::::::  80872c82  add     esp,8
::::::
::::::  80872c85  lea     ecx,[esi+esi*2]
::::::  80872c88  push    1
::::::  80872c8a  mov     eax,dword ptr [edi+ecx*4+8]  ; pExceptionRegistration->scopetable[i].lpfnHandler
::::::  80872c8e  call    nt!_NLG_Notify (80872617)
::::::  80872c93  mov     eax,dword ptr [edi+ecx*4]  ;
::::::  80872c96  mov     dword ptr [ebx+0Ch],eax     ; pExceptionRegistration->trylevel = RegistrationPointer->scopetable[i].previousTryLevel
::::::  80872c99  mov     eax,dword ptr [edi+ecx*4+8]  ; pExceptionRegistration->scopetable[i].lpfnHandler
::::::  80872c9d  xor     ebx,ebx
::::::  80872c9f  xor     ecx,ecx
::::::  80872ca1  xor     edx,edx
::::::  80872ca3  xor     esi,esi
::::::  80872ca5  xor     edi,edi
::::::  80872ca7  call    eax ; pExceptionRegistration->scopetable[i].lpfnHandler(); 这里不会返回的！！
::::::
:::::: ;nt!_except_handler3+0xa9 [d:\dnsrv\base\crts\crtw32\misc\i386\exsup3.asm @ 285]:
:::::: ; 找到 scopetable 中的下一个 scopetable_entry，继续循环
:::>>::  80872ca9  mov     edi,dword ptr [ebx+8]
:::  ::  80872cac  lea     ecx,[esi+esi*2]
:::  ::  80872caf  mov     esi,dword ptr [edi+ecx*4]
:::  :<  80872cb2  jmp     nt!_except_handler3+0x40 (80872c40)
:::  :
:::  : ;nt!_except_handler3+0xb4 [d:\dnsrv\base\crts\crtw32\misc\i386\exsup3.asm @ 291]:
:::  >   80872cb4  mov     eax,0 ; eax = ExceptionContinueExecution (0)
:::   <   80872cb9  jmp     nt!_except_handler3+0xde (80872cde)
:::   :
:::   : ;nt!_except_handler3+0xbb [d:\dnsrv\base\crts\crtw32\misc\i386\exsup3.asm @ 295]:
:>:   :  80872cbb  mov     eax,dword ptr [ebp+8]
: :   :  80872cbe  or      dword ptr [eax+4],8
: :   :
: :   : ;nt!_except_handler3+0xc2 [d:\dnsrv\base\crts\crtw32\misc\i386\exsup3.asm @ 298]:
: >   :  80872cc2  mov     eax,1 ; eax = ExceptionContinueSearch (1)
:     :<  80872cc7  jmp     nt!_except_handler3+0xde (80872cde)
:     ::
>     :: ;nt!_except_handler3+0xc9 [d:\dnsrv\base\crts\crtw32\misc\i386\exsup3.asm @ 302]:
      ::  ; 设置了(EXCEPTION_UNWINDING | EXCEPTION_EXIT_UNWIND)，开始展开
      ::  80872cc9  push    ebp
      ::  80872cca  lea     ebp,[ebx+10h] ;ebp = pExceptionRegistration->_ebp
      ::  80872ccd  push    0FFFFFFFFh
      ::  80872ccf  push    ebx
      ::  80872cd0  call    nt!__local_unwind2 (8087257b)
      ::  80872cd5  add     esp,8
      ::  80872cd8  pop     ebp
      ::  80872cd9  mov     eax,1 eax = ExceptionContinueSearch (1)
      ::
      :: ;nt!_except_handler3+0xde [d:\dnsrv\base\crts\crtw32\misc\i386\exsup3.asm @ 313]:
      >>  80872cde  pop     ebp
      80872cdf  pop     edi
      80872ce0  pop     esi
      80872ce1  pop     ebx
      80872ce2  mov     esp,ebp
      80872ce4  pop     ebp
      80872ce5  ret

```

## 附录 2 `nt!__local_unwind2` 的反汇编代码

`代码`



```
      kd> uf __local_unwind2
      nt!__local_unwind2 [d:\dnsrv\base\crts\crtw32\misc\i386\exsup.asm @ 205]:
        8087257b  push    ebx
        8087257c  push    esi
        8087257d  push    edi
        8087257e  mov     eax,dword ptr [esp+10h] ; eax = pExceptionRegistration
        80872582  push    ebp
        80872583  push    eax
        80872584  push    0FFFFFFFEh
        80872586  push    offset nt!__unwind_handler (80872540)
        8087258b  push    dword ptr fs:[0]
        80872592  mov     dword ptr fs:[0],esp ; __local_unwind2 自身也会构建 _EXCEPTION_REGISTRATION

      nt!__local_unwind2+0x1e [d:\dnsrv\base\crts\crtw32\misc\i386\exsup.asm @ 222]:
>       80872599  mov     eax,dword ptr [esp+24h] ; eax = pExceptionRegistration
:       8087259d  mov     ebx,dword ptr [eax+8]   ; ebx = pExceptionRegistration->scopetable
:       808725a0  mov     esi,dword ptr [eax+0Ch] ; esi = pExceptionRegistration->trylevel
:       808725a3  cmp     esi,0FFFFFFFFh          ; cmp pExceptionRegistration->trylevel, TRYLEVEL_NONE
:<      808725a6  je      nt!_NLG_Return2+0x2 (808725dd)
::      
::    nt!__local_unwind2+0x2d [d:\dnsrv\base\crts\crtw32\misc\i386\exsup.asm @ 228]:
::      808725a8  cmp     dword ptr [esp+28h],0FFFFFFFFh
::<     808725ad  je      nt!__local_unwind2+0x3a (808725b5)
:::    
:::   nt!__local_unwind2+0x34 [d:\dnsrv\base\crts\crtw32\misc\i386\exsup.asm @ 230]:
:::     808725af  cmp     esi,dword ptr [esp+28h]
:::<    808725b3  jbe     nt!_NLG_Return2+0x2 (808725dd)
::::   
::::  nt!__local_unwind2+0x3a [d:\dnsrv\base\crts\crtw32\misc\i386\exsup.asm @ 234]:
::>:    808725b5  lea     esi,[esi+esi*2]
:: :    808725b8  mov     ecx,dword ptr [ebx+esi*4]  ; move ecx, [pExceptionRegistration->scopetable[i].previousTryLevel]
:: :    808725bb  mov     dword ptr [esp+8],ecx      ; 这个 esp+8 只写没读，什么情况？
:: :    808725bf  mov     dword ptr [eax+0Ch],ecx    ; pExceptionRegistration->trylevel = ecx
:: :    808725c2  cmp     dword ptr [ebx+esi*4+4],0  ; cmp pExceptionRegistration->scopetable->lpfnFilter, NULL
:: :<   808725c7  jne     nt!_NLG_Return2 (808725db)
:: :: 
:: :: nt!__local_unwind2+0x4e [d:\dnsrv\base\crts\crtw32\misc\i386\exsup.asm @ 243]:
:: ::   ; RegistrationPointer->scopetable->lpfnFilter 等于 NULL，即这里是 __try & __finally 的组合
:: ::   808725c9  push    101h
:: ::   808725ce  mov     eax,dword ptr [ebx+esi*4+8]
:: ::   808725d2  call    nt!_NLG_Notify (80872617)  ; 这个函数对理解 SEH 不重要，可以暂时忽略
:: ::   808725d7  call    dword ptr [ebx+esi*4+8]    ; pExceptionRegistration->scopetable->lpfnHandler()
:: :: 
:: :: nt!_NLG_Return2 [d:\dnsrv\base\crts\crtw32\misc\i386\exsup.asm @ 251]:
: :>   808725db  jmp     nt!__local_unwind2+0x1e (80872599) ; 循环
 : :
 : :  nt!_NLG_Return2+0x2 [d:\dnsrv\base\crts\crtw32\misc\i386\exsup.asm @ 253]:
 > >    808725dd  pop     dword ptr fs:[0]
        808725e4  add     esp,10h
        808725e7  pop     edi
        808725e8  pop     esi
        808725e9  pop     ebx
        808725ea  ret

```

参考资料：  

`[1]` wrk 源码  

`[2]` A Crash Course on the Depths of Win32? Structured Exception Handling, Matt Pietrek  

`[3]` 《Windows 内核原理与实现》潘爱民  

`[4]` 《软件调试》张银奎
