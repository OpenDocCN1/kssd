

# SEH 分析笔记（X64 篇）

> 原文：[`www.pediy.com/kssd/pediy12/142371.html`](https://www.pediy.com/kssd/pediy12/142371.html)

SEH 分析笔记（X64 篇）  
  v1.0.0  
  boxcounter  
  
  历史：  
  v1.0.0, 2011-11-4：最初版本。  
  
  \[不介意转载，但请注明出处 [www.boxcounter.com](http://www.boxcounter.com)   
  附件里有本文的原始稿，一样的内容，更好的高亮和排版。  
  本文的部分代码可能会因为论坛的自动换行变得很乱，需要的朋友手动复制到自己的代码编辑器就可以正常显示了\]  
  
  在之前的《SEH 分析笔记（X86 篇）》中，我借助 wrk1.2 介绍了 x86 下 windows 系统内核中的 SEH 实现。这次我们来看看 x64 位 windows 系统内核中 SEH 的实现。  
  本文需要大家熟悉 x64 位系统的一些特性，比如调用约定、Prolog 和 Epilog。可以通过这几篇文章熟悉一下:  
  [Overview of x64 Calling Conventions](http://msdn.microsoft.com/en-us/library/ms235286.aspx), MSDN  
  [The history of calling conventions, part 5: amd64](http://blogs.msdn.com/b/oldnewthing/archive/2004/01/14/58579.aspx) , The Old New Thing  
  [Everything You Need To Know To Start Programming 64-Bit Windows Systems](http://msdn.microsoft.com/en-us/magazine/cc300794.aspx), Matt Pietrek  
  
  首先回顾一下前一篇文章。  
  在 x86 windows 中，函数通过以下几个步骤来参与 SEH ：  
  1. 在自身的栈空间中分配并初始化一个 EXCEPTION\_REGISTRATION(\_RECORD) 结构体。  
  2. 将该 EXCEPTION\_REGISTRATION(\_RECORD) 挂入当前线程的异常链表。  
  
  当某函数触发异常时，系统首先会通过调用 KiDispatchException 来给内核调试器一个机会，如果内核调试器没有处理该异常，则该机会被转给 RtlDispatchException，这个函数就开始分发该异常。分发过程为：  
  从当前线程的异常链表头开始遍历，对于每一个 SEH 注册信息（即 EXCEPTION\_REGISTRATION(\_RECORD)），调用其 Handler。根据 Handler 的返回值做相应的后续处理：  
    1. 返回 ExceptionContinueExecution，表示 Handler 已经修复了异常触发点，从异常触发点继续执行。  
    2. 返回 ExceptionContinueSearch，表示该 Handler 没有处理该异常，继续遍历异常链表。  
    3. Handler 没有修复异常触发点，但是却能处理该异常（某个 \_\_except 过滤代码返回 EXCEPTION\_EXECUTE\_HANDLER）。这种情况下，处理完该异常后就从异常解决代码（\_\_except 代码块）继续执行，Handler 不会返回。  
  以上是简略的 x86 SEH 流程，其中省略了很多细节，比如展开、错误处理、ExceptionNestedException 和 ExceptionCollidedUnwind 等等。  
  
  之所以在这里重温这个流程，是因为 x64 中 SEH 的流程总体思路也是如此，只是细节上做了一些修改。但这并不表示熟悉 x86 SEH 就能很轻松的掌握 x64 SEH。  
  
  本文分为四个部分：“异常注册”、“异常分发”、“展开、解决”和“ExceptionNestedException 和 ExceptionCollidedUnwind”。依然以 MSC 的增强版为分析对象。分析环境为：WDK 7600.16385.1，内置的 cl 的版本是 15.00.30729.207，link 的版本是 9.00.30729.207，测试虚拟机系统为 amd64 WinXP + wrk1.2。  
    
  在讲述之前，需要先定义几个名词，以简化后续的讲述。  
  
  RVA  熟悉 PE 格式的朋友都懂的，表示某个绝对地址相对于所在模块的基地址的偏移。  
  EXCEPT\_POINT  异常触发点。  
  EXCEPT\_FILTER  \_\_except 小括号内的异常过滤代码。  
  EXCEPT\_HANDLER  \_\_except 大括号内的异常解决代码。  
  FINALLY\_HANDLER  \_\_finally 大括号内的代码。  
  
  以下面的伪码为例，  
  

代码:

```
    1  \_\_try
    2  {
    3      \_\_try
    4      {
    5           \*((ULONG\*)NULL) = 0; 
    6      }
    7      \_\_except((STATUS\_INVALID\_PARAMETER \== GetExceptionCode()) ? EXCEPTION\_CONTINUE\_SEARCH : EXCEPTION\_EXECUTE\_HANDLER)
    8      {
    9          ...
    10     }
    11 }
    12 \_\_finally
    13 {
    14     ...
    15 {

```

  EXCEPT\_POINT 指的是行 5 中的代码。  
  EXCEPT\_FILTER 指的是行 7 中的“(STATUS\_INVALID\_PARAMETER == GetExceptionCode()) ? EXCEPTION\_CONTINUE\_SEARCH : EXCEPTION\_EXECUTE\_HANDLER”。  
  EXCEPT\_HANDLER 指的是行 8 到行 10 中所有的代码。  
  FINALLY\_HANDLER 指的是行 13 到行 15 中所有的代码。  
  
  
**一、异常注册**  
  
  在 x64 windows 中，异常注册信息发生了巨大的改变。x86 中异常注册信息是在函数执行过程中在栈中分配并初始化的。x64 中变成这样：  
  异常注册信息不再是动态创建，而是编译过程中生成，链接时写入 PE+ 头中的 ExceptionDirectory（参考 winnt.h 中 IMAGE\_RUNTIME\_FUNCTION\_ENTRY 的定义）。ExceptionDirectory 里包含几乎所有函数的栈操作、异常处理等信息。  
  
  来看看新异常注册信息的数据结构：  
  
代码:

```
    typedef struct \_RUNTIME\_FUNCTION {
        ULONG BeginAddress;
        ULONG EndAddress;
        ULONG UnwindData;
    } RUNTIME\_FUNCTION, \*PRUNTIME\_FUNCTION;

    typedef enum \_UNWIND\_OP\_CODES {
        UWOP\_PUSH\_NONVOL \= 0,
        UWOP\_ALLOC\_LARGE,       // 1
        UWOP\_ALLOC\_SMALL,       // 2
        UWOP\_SET\_FPREG,         // 3
        UWOP\_SAVE\_NONVOL,       // 4
        UWOP\_SAVE\_NONVOL\_FAR,   // 5
        UWOP\_SPARE\_CODE1,       // 6
        UWOP\_SPARE\_CODE2,       // 7
        UWOP\_SAVE\_XMM128,       // 8
        UWOP\_SAVE\_XMM128\_FAR,   // 9
        UWOP\_PUSH\_MACHFRAME     // 10
    } UNWIND\_OP\_CODES, \*PUNWIND\_OP\_CODES;

    typedef union \_UNWIND\_CODE {
        struct {
            UCHAR CodeOffset;
            UCHAR UnwindOp : 4;
            UCHAR OpInfo : 4;
        };
    
        USHORT FrameOffset;
    } UNWIND\_CODE, \*PUNWIND\_CODE;
    
    #define UNW\_FLAG\_NHANDLER 0x0
    #define UNW\_FLAG\_EHANDLER 0x1
    #define UNW\_FLAG\_UHANDLER 0x2
    #define UNW\_FLAG\_CHAININFO 0x4

    typedef struct \_UNWIND\_INFO {
        UCHAR Version : 3;
        UCHAR Flags : 5;
        UCHAR SizeOfProlog;
        UCHAR CountOfCodes;
        UCHAR FrameRegister : 4;
        UCHAR FrameOffset : 4;
        UNWIND\_CODE UnwindCode\[1\];
    
    //
    // The unwind codes are followed by an optional DWORD aligned field that
    // contains the exception handler address or a function table entry if
    // chained unwind information is specified. If an exception handler address
    // is specified, then it is followed by the language specified exception
    // handler data.
    //
    //  union {
    //      struct {
    //          ULONG ExceptionHandler;
    //          ULONG ExceptionData\[\];
    //      };
    //
    //      RUNTIME\_FUNCTION FunctionEntry;
    //  };
    //
    
    } UNWIND\_INFO, \*PUNWIND\_INFO;

    typedef struct \_SCOPE\_TABLE {
        ULONG Count;
        struct
        {
            ULONG BeginAddress;
            ULONG EndAddress;
            ULONG HandlerAddress;
            ULONG JumpTarget;
        } ScopeRecord\[1\];
    } SCOPE\_TABLE, \*PSCOPE\_TABLE;

```

  x64 中，MSC 为几乎所有的函数都登记了完备的信息，用来在展开过程中完整的回滚函数所做的栈、寄存器操作。登记的信息包括：  
  函数是否使用了 SEH、  
  函数使用的是什么组合的 SEH（\_\_try/\_\_except？\_\_try/\_\_finally？）、  
  函数申请了多少栈空间、  
  函数保存了哪些寄存器、  
  函数是否建立了栈帧，  
  等等，  
  同时也记录了这些操作的顺序（以保证回滚的时候不会乱套）。  
  
  这些信息就存储在 UNWIND\_INFO 之中。  
  UNWIND\_INFO 相当于 x86 下的 EXCEPTION\_REGISTRATION。它的成员分别是：  
    Version  结构体的版本。  
    Flags  标志位，可以有这么几种取值：  
      UNW\_FLAG\_NHANDLER (0x0): 表示既没有 EXCEPT\_FILTER 也没有 EXCEPT\_HANDLER。  
      UNW\_FLAG\_EHANDLER (0x1): 表示该函数有 EXCEPT\_FILTER & EXCEPT\_HANDLER。  
      UNW\_FLAG\_UHANDLER (0x2): 表示该函数有 FINALLY\_HANDLER。  
      UNW\_FLAG\_CHAININFO (0x4): 表示该函数有多个 UNWIND\_INFO，它们串接在一起（所谓的 chain）。  
    SizeOfProlog  表示该函数的 Prolog 指令的大小，单位是 byte。  
    CountOfCodes  表示当前 UNWIND\_INFO 包含多少个 UNWIND\_CODE 结构。  
    FrameRegister  如果函数建立了栈帧，它表示栈帧的索引（相对于 CONTEXT::RAX 的偏移，详情参考 RtlVirtualUnwind 源码）。否则该成员的值为 0。  
    FrameOffset  表示 FrameRegister 距离函数最初栈顶（刚进入函数，还没有执行任何指令时的栈顶）的偏移，单位也是 byte。  
    UnwindCode  是一个 UNWIND\_CODE 类型的数组。元素数量由 CountOfCodes 决定。  
  需要说明几点：  
    1. 如果 Flags 设置了 UNW\_FLAG\_EHANDLER 或 UNW\_FLAG\_UHANDLER，那么在最后一个 UNWIND\_CODE 之后存放着 ExceptionHandler（相当于 x86 EXCEPTION\_REGISTRATION::handler）和 ExceptionData（相当于 x86 EXCEPTION\_REGISTRATION::scopetable）。  
    2. UnwindCode 数组详细记录了函数修改栈、保存非易失性寄存器的指令。  
    3. MSDN 中有 UNWIND\_INFO 和 UNWIND\_CODE 的详细说明，推荐阅读。  
  
  那 UNWIND\_INFO 是如何与其描述的函数关联起来的呢？答案是：通过一个 RUNTIME\_FUNCTION 结构体。  
  RUNTIME\_FUNCTION::BeginAddress 同 RUNTIME\_FUNCTION::EndAddress 一起以 RVA 形式描述了函数的范围。  
  RUNTIME\_FUNCTION::UnwindData 就是 UNWIND\_INFO 了，它也是一个 RVA 值。  
  
  PE+ 中的 ExceptionDirectory 中存放着所有函数的 RUNTIME\_FUNCTION，按 RUNTIME\_FUNCTION::BeginAddress 升序排列。一旦触发异常，系统可以通过 EXCEPT\_POINT 的 RVA 在 ExceptionDirectory 中二分查找到 RUNTIME\_FUNCTION，进而找到 UNWIND\_INFO。  
  
  前面有提到，MSC 为几乎所有的函数都登记了完毕的信息，那是不是有一些特殊函数没有登记信息呢？  
  是的。x64 新增了一个概念，叫做“叶函数”。熟悉数据结构的朋友可能第一时间就联想到“叶节点”。没错，“叶函数”的含义跟“叶节点”很类似，叶函数不会有子函数，也就是说它不会再调用任何函数。另外 x64 对这个概念额外加了一些要求：不修改栈指针（比如分配栈空间）、没有使用 SEH。总结下来就是：既不调用函数、又没有修改栈指针，也没有使用 SEH 的函数就叫做“叶函数”。  
  叶函数可以没有登记信息，原因很简单，它根本就没信息需要登记~  
  
  还有一个 SCOPE\_TABLE 结构，熟悉 x86 SEH 的朋友应该很眼熟 :-)，它等同于 x86 SEH 中的 REGISTRATIOIN\_RECORD::scopetable 的类型。其成员有：  
    Count  表示 ScopeRecord 数组的大小。  
    ScopeRecord  等同于 x86 中的 scopetable\_entry 成员。其中，  
      BeginAddress 和 EndAddress 表示某个 \_\_try 保护域的范围。  
      HandlerAddress 和 JumpTarget 表示 EXCEPTION\_FILTER、EXCEPT\_HANDLER 和 FINALLY\_HANDLER。具体对应情况为：  
        对于 \_\_try/\_\_except 组合，HandlerAddress 代表 EXCEPT\_FILTER，JumpTarget 代表 EXCEPT\_HANDLER。  
        对于 \_\_try/\_\_finally 组合，HandlerAddress 代表 FINALLY\_HANDLER，JumpTarget 等于 0。  
      这四个域通常都是 RVA，但当 EXCEPT\_FILTER 简单地返回或等于 EXCEPTION\_EXECUTE\_HANDLER 时，HandlerAddress 可能直接等于 EXCEPTION\_EXECUTE\_HANDLER，而不再是一个 RVA。  
  
  我们可以通过 windbg 中的 .fnent 命令来查看某个函数的异常注册信息。比如，  
  
  1   kd> .fnent passThrough!SehTest  
  2   Debugger function entry 00000000\`00778210 for:  
  3   d:\\workspace\\code\\mycode\\r0\\passthrough\\passthrough.c(51)  
  4   (fffffadf\`f140f020)   PassThrough!SehTest   |  (fffffadf\`f140f0c0)   PassThrough!Caller2  
  5   Exact matches:  
  6       PassThrough!SehTest (void)  
  7     
  8   BeginAddress      = 00000000\`00001020  
  9   EndAddress        = 00000000\`000010b2  
  10  UnwindInfoAddress = 00000000\`00002668  
  11    
  12  Unwind info at fffffadf\`f1410668, 10 bytes  
  13    version 1, flags 1, prolog 4, codes 1  
  14    handler routine: PassThrough!\_C\_specific\_handler (fffffadf\`f140f4ce), data 3  
  15    00: offs 4, unwind op 2, op info 4  UWOP\_ALLOC\_SMALL.  
  
  行 8 到行 10 描述的是 RUNTIME\_FUNCTION。  
  行 12 到行 15 描述的是 UNWIND\_INFO。  
  
  对于叶函数，输出是这样的，  
  
  kd> .fnent passthrough!LeafTest  
  No function entry for fffffadf\`f240c080  
  
  到这里，异常注册就讲完了，我们认识了相关的数据结构和定位方法。下面我们进入异常分发流程。  
  
  
**二、异常分发**  
  
  x64 异常分发过程使用的仍然是 KiDispatchException、RtlDispatchException、RtlpExecuteHandlerForException 等函数。  
  
  其中，KiDispatchException 中有关内核异常部分的代码完全没有变化，这里我偷懒直接拷贝《SEH 分析笔记（X86 篇）》中的部分描述，  
  
  原型：  
代码:

```
  VOID
  KiDispatchException (
      IN PEXCEPTION\_RECORD ExceptionRecord,
      IN PKEXCEPTION\_FRAME ExceptionFrame,
      IN PKTRAP\_FRAME TrapFrame,
      IN KPROCESSOR\_MODE PreviousMode,
      IN BOOLEAN FirstChance
      );

```

  对于内核异常，它的处理步骤如下：  
    如果 FirstChance 为 TRUE，那么,  
      1. 首先将该异常传达给内核调试器（KD），如果 KD 处理了该异常，那么函数返回。  
      2. KD 没有处理，调用 RtlDispatchException 进行异常分发。如果分发成功，RtlDispatchExcetpion 返回 TRUE，或者根本不返回。  
      3. RtlDispatchException 分发失败，那么再给 KD 一次处理机会，如果还是没有处理，那么 BUGCHECK。  
    如果 FirstChance 为 FALSE，那么将该异常传达给 KD，如果 KD 没有处理，那么 BUGCHECK。  
  它的源码实现位于 $WRK-v1.2\\base\\ntos\\ke\\amd64\\exceptn.c:430。  
  
  RtlDispatchException 发生了一些改变。  
  从之前描述的异常注册信息的数据结构可以发现，x64 中已经不存在异常注册链这个概念了（虽然 \_NT\_TIB 结构体中还保留了 ExceptionList 域）。那 x64 中 RtlDispatchException 是如何遍历异常注册信息的呢？前面虽然有提到如何通过 EXCEPT\_POINT 找到 UNWIND\_INFO 结构，但是假如这个 UNWIND\_INFO 没有处理该异常，如何继续遍历呢？  
  
  在解答这个问题之前，我们先来认识一个新函数和相关的数据结构，  
  
代码:

```
    #define UNWIND\_HISTORY\_TABLE\_SIZE 12
    
    typedef struct \_UNWIND\_HISTORY\_TABLE\_ENTRY {
            ULONG64 ImageBase;
            PRUNTIME\_FUNCTION FunctionEntry;
    } UNWIND\_HISTORY\_TABLE\_ENTRY, \*PUNWIND\_HISTORY\_TABLE\_ENTRY;
    
    #define UNWIND\_HISTORY\_TABLE\_NONE 0
    #define UNWIND\_HISTORY\_TABLE\_GLOBAL 1
    #define UNWIND\_HISTORY\_TABLE\_LOCAL 2
    
    typedef struct \_UNWIND\_HISTORY\_TABLE {
            ULONG Count;
            UCHAR Search;
            ULONG64 LowAddress;
            ULONG64 HighAddress;
            UNWIND\_HISTORY\_TABLE\_ENTRY Entry\[UNWIND\_HISTORY\_TABLE\_SIZE\];
    } UNWIND\_HISTORY\_TABLE, \*PUNWIND\_HISTORY\_TABLE;

    PRUNTIME\_FUNCTION
    RtlLookupFunctionEntry (
        IN ULONG64 ControlPc,
        OUT PULONG64 ImageBase,
        IN OUT PUNWIND\_HISTORY\_TABLE HistoryTable OPTIONAL
        );

```

  RtlLookupFunctionEntry 的功能是查找指定地址所在函数的 RUNTIME\_FUNCTION 和所在模块的基地址。它的参数分为为，  
    ControlPc  需要查找的指令地址，  
    ImageBase  返回的模块基地址，  
    HistoryTable  用于加速查找。  
  
  工作流程：  
  RtlLookupFunctionEntry 在搜索的过程会根据是否传入 HistoryTable 而采取不同的搜索方法：  
    如果传入 HistoryTable，则根据 HistoryTable->Search 表示的搜索方式在表中进行搜索：  
      如果搜索方式为 UNWIND\_HISTORY\_TABLE\_NONE，那么不在 HistoryTable 中进行搜索。  
      如果搜索方式为 UNWIND\_HISTORY\_TABLE\_GLOBAL，则首先在全局表 RtlpUnwindHistoryTable 开始搜索，如果搜索到，则结束搜索，函数返回。否则再在 HistoryTable 中搜索。  
      如果搜索方式为 UNWIND\_HISTORY\_TABLE\_LOCAL，那么在 HistoryTable 中搜索。  
    如果上述过程中没有搜索到需要的结果，那么找到模块基地址，从模块的 PE+ 头结构中解析出 RUNTIME\_FUNCTION。如果搜索方式为 UNWIND\_HISTORY\_TABLE\_NONE，还会将解决加入到 HistoryTable。  
  
  之前的描述中 RtlDispatchException 定位 EXCEPT\_POINT 所对应的 RUNTIME\_FUNCTION 就是通过调用 RtlLookupFunctionEntry 实现的。  
  
  回到刚才的问题，如何推动遍历呢？为了解决这个问题，x64 又引进了一个新函数。现在我们有请 x64 SEH 核心成员 RtlVirtualUnwind 登场~  
  
  先来看看它的原型：  
代码:

```
  PEXCEPTION\_ROUTINE
  RtlVirtualUnwind (
      IN ULONG HandlerType,
      IN ULONG64 ImageBase,
      IN ULONG64 ControlPc,
      IN PRUNTIME\_FUNCTION FunctionEntry,
      IN OUT PCONTEXT ContextRecord,
      OUT PVOID \*HandlerData,
      OUT PULONG64 EstablisherFrame,
      IN OUT PKNONVOLATILE\_CONTEXT\_POINTERS ContextPointers OPTIONAL
      );

```

  它的主要功能是：  
    根据传入的 ControlPc 和 ContextRecord 等参数虚拟（模拟）展开该函数，并返回该函数的一些信息，比如 HandlerData（SCOPE\_TABLE）、EstablisherFrame（rsp 或 栈帧）。    
  流程是：  
    1. 通过 FunctionEntry 和 ImageBase 找到 UNWIND\_INFO。根据 UNWIND\_INFO 中记录的信息，查找 EstablisherFrame（即栈帧或者 rsp）。  
    2. 根据 ControlPc 分如下两种情况展开：  
      a. ControlPc >= EpilogOffset，即 ControlPc 在 Epilog 之中。那么把剩余的 Epilog 指令模拟执行完毕即可。  
         所谓模拟是指，如果下一条 EpiLog 指令是“sub rsp, 0x32”，那么将 ContextRecord->rsp 减去 0x32。并不是真正执行 sub 指令。  
      b. ControlPc < EpilogOffset，那么把 Prolog 反向模拟回滚一遍即可。  
         所谓反向回滚是指，如果 Prolog 的指令是  
         mov   \[RSP + 8\], RCX  
         push R15  
         push R14  
         push R13   
         那么反向回滚就是  
              pop ContextRecord->R13 （实际上是从 ContextRecord->Rsp 指向的内存中取出值存入 ContextRecord->R13，然后 ContextRecord->Rsp 加上 8。并不是真正执行 pop）  
              pop ContextRecord->R14  
              pop ContextRecord->R15  
              mov ContextRecord->RCX, \[ContextRecord->RSP+8\]  
       然后把 ContextRecord->Rip 修改为 ControlPc 所在函数的返回地址，即父函数中的某一处 call 的下一条指令。  
       这样，ContextRecord 就被恢复成父函数在调用 ControlPc 所在函数之后的状态了。  
    3. 如果 HandlerType 包含 UNW\_FLAG\_EHANDLER 或 UNW\_FLAG\_UHANDLER，那么将 UNWIND\_INFO::ExceptionData 赋给传出参数 HandlerData，并返回 UNWIND\_INFO::ExceptionRoutine。  
       对于 MSC 编译器生成的模块，UNWIND\_INFO::ExceptionRoutine 一般指向 nt!\_\_C\_specific\_handler。UNWIND\_INFO::ExceptionData 指向 ControlPc 所在函数的 SCOPE\_TABLE。  
  RtlVirtualUnwind 的实现源码位于 $WRK-v1.2\\base\\ntos\\rtl\\amd64\\exdsptch.c:1202。  
  
  RtlVirtualUnwind 返回后，RtlDispatchException 就可以根据 ContextRecord->Rip 找到父函数对应的 RUNTIME\_FUNCTION，进而找到 UNWIND\_INFO。就这样推动整个遍历过程。  
  
  这是一般情况，对于没有 UNWIND\_INFO 的叶函数呢？  
  对于叶函数，RtlLookupFunctionEntry 返回 NULL，于是 RtlDispatchException 知道这是个叶函数，就找到该叶函数的父函数，从父函数继续遍历。也就是完全无视叶函数，因为叶函数对整个异常处理过程没有任何影响。  
  
  RtlDispatchException 调用 UNWIND\_INFO::ExceptionHandler 依然是通过 RtlpExecuteHandlerForException，其函数原型没有变化：  
  
代码:

```
  EXCEPTION\_DISPOSITION
  RtlpExecuteHandlerForException (
      IN PEXCEPTION\_RECORD ExceptionRecord,
      IN PVOID EstablisherFrame,
      IN OUT PCONTEXT ContextRecord,
      IN OUT PVOID DispatcherContext
      );

```

  该函数的实现源码位于 $\\WRK-v1.2\\base\\ntos\\rtl\\amd64\\xcptmisc.asm:84。  
  RtlpExecuteHandlerForException 的逻辑较 x86 版本没什么大变化，内部注册了一个异常处理函数 RtlpExceptionHandler。RtlpExceptionHandler 相当于 x86 中的 nt!ExecuteHandler2，其内部会返回 ExceptionNestedException 或 ExceptionContinueSearch。它的实现源码位于 $\\WRK-v1.2\\base\\ntos\\rtl\\amd64\\xcptmisc.asm:26。  
  
  需要一提的是，最后一个参数 DispatchContext 的类型是 DISPATCHER\_CONTEXT，相对于 x86 版本，它扩充了很多，  
  
代码:

```
    typedef struct \_DISPATCHER\_CONTEXT {
        ULONG64 ControlPc;
        ULONG64 ImageBase;
        PRUNTIME\_FUNCTION FunctionEntry;
        ULONG64 EstablisherFrame;
        ULONG64 TargetIp;
        PCONTEXT ContextRecord;
        PEXCEPTION\_ROUTINE LanguageHandler;
        PVOID HandlerData;
        PUNWIND\_HISTORY\_TABLE HistoryTable;
        ULONG ScopeIndex;
        ULONG Fill0;
    } DISPATCHER\_CONTEXT, \*PDISPATCHER\_CONTEXT;

```

  成员分别为：  
    ControlPc  异常触发点。  
    ImagePase  ControlPc 所在模块的基地址。  
    FunctionEntry  ControlPc 所在函数的 RUNTIME\_FUNCTION。  
    EstablisherFrame  ControlPc 所在函数的栈帧（如果建立了栈帧）或 RSP。  
    TargetIp  解决异常的 EXCEPT\_HANDLER 地址，该成员只在展开的过程中被使用。RtlpExecuteHandlerForException 没有使用它。  
    ContextRecord  供展开过程中使用，只有当展开过程中触发新异常（返回 ExceptionCollidedUnwind）时，才会被 RtlDispatchException 真正的使用到（参考 RtlDispatchException 处理 ExceptionCollidedUnwind 的代码）。  
    LanguageHandler  ControlPc 所在函数的 UNWIND\_INFO::ExceptionRoutine。  
    HandlerData  ControlPc 所在函数的 UNWIND\_INFO::ExceptionData。  
    ScopeIndex  UNWIND\_INFO::ExceptionData 中 SCOPE\_TABLE::ScopeRecord 的索引，通常设置为 0（注：请不要与 x86 中运行时不断改变的 EXCEPTION\_REGISTRATION::trylevel 相混淆，ScopeIndex 不会在在函数执行过程中改变）  
    Fill0  未用。  
  
  再看一下它的 .fnent 输出，  
  
  1  kd> .fnent nt!RtlpExecuteHandlerForException  
  2  Debugger function entry 00000000\`01458210 for:  
  3  (fffff800\`008bd950)   nt!RtlpExecuteHandlerForException   |  (fffff800\`008bd970)   nt!RtlpUnwindHandler  
  4  Exact matches:  
  5      nt!RtlpExecuteHandlerForException (void)  
  6    
  7  BeginAddress      = 00000000\`000bd950  
  8  EndAddress        = 00000000\`000bd963  
  9  UnwindInfoAddress = 00000000\`000dfeb8  
  10   
  11 Unwind info at fffff800\`008dfeb8, 10 bytes  
  12   version 1, flags 3, prolog 4, codes 1  
  13   handler routine: nt!RtlpExceptionHandler (fffff800\`008bd920), data 0  
  14   00: offs 4, unwind op 2, op info 4  UWOP\_ALLOC\_SMALL.  
  
  行 12 中显示 flags 等于 3，即  UNW\_FLAG\_EHANDLER (0x1) | UNW\_FLAG\_UHANDLER (0x2)，说明行 13 中显示的异常处理函数 nt!RtlpExceptionHandler 既负责解决异常，也负责展开。  
  
  RtlpExecuteHandlerForException 会调用 DISPATCHER\_CONTEXT::LanguageHandler。对于 MSC 编译得到的模块，它是 nt!\_\_C\_specific\_handler，我们来看看这个函数，  
  
  原型：  
代码:

```
  EXCEPTION\_DISPOSITION 
  \_\_C\_specific\_handler (
      IN PEXCEPTION\_RECORD pExceptionRecord,
      IN PVOID pEstablisherFrame,
      IN OUT PCONTEXT pContext,
      IN OUT PVOID pDispatcherContext
      );

```

  反汇编码：  
代码:

```
               kd\> uf nt!\_\_C\_specific\_handler
               nt!\_\_C\_specific\_handler:
               fffff800\`008a42d0 mov     qword ptr \[rsp+10h\],rdx ; 在栈上保存 pEstablisherFrame
               fffff800\`008a42d5 mov     rax,rsp
               fffff800\`008a42d8 sub     rsp,88h
               fffff800\`008a42df mov     qword ptr \[rax\-8\],rbx
               fffff800\`008a42e3 mov     qword ptr \[rax\-10h\],rbp
               fffff800\`008a42e7 mov     rbp,qword ptr \[r9\]      ; rbp = pDispatcherContext->ControlPc
               fffff800\`008a42ea mov     qword ptr \[rax\-18h\],rsi
               fffff800\`008a42ee mov     qword ptr \[rax\-20h\],rdi
               fffff800\`008a42f2 mov     qword ptr \[rax\-28h\],r12
               fffff800\`008a42f6 mov     r12,qword ptr \[r9+38h\]  ; r12 = pDispatcherContext->HandlerData
               fffff800\`008a42fa mov     qword ptr \[rax\-30h\],r13
               fffff800\`008a42fe mov     qword ptr \[rax\-38h\],r14
               fffff800\`008a4302 mov     r14,qword ptr \[r9+8\]    ; r14 = pDispatcherContext->ImageBase
               fffff800\`008a4306 mov     qword ptr \[rax\-40h\],r15
               fffff800\`008a430a mov     r13,r9                  ; r13 = pDispatcherContext
               fffff800\`008a430d sub     rbp,r14                 ; l\_OffsetInFunc = pDispatcherContext->ControlPc - pDispatcherContext->ImageBase
               fffff800\`008a4310 test    byte ptr \[rcx+4\],66h    ; pExceptionRecord->ExceptionFlags, EXCEPTION\_UNWIND (0x66)
               fffff800\`008a4314 mov     rsi,rdx                 ; rsi = pEstablisherFrame
               fffff800\`008a4317 mov     r15,rcx                 ; r15 = pExceptionRecord
<              fffff800\`008a431a jne     nt!\_\_C\_specific\_handler+0xf5 (fffff800\`008a43c5)
:              
:              -------------------------------------------------------------------
:              nt!\_\_C\_specific\_handler+0x50:
:              fffff800\`008a4320 movsxd  rdi,dword ptr \[r9+48h\]  ; l\_ScopeIndex (rdi) = pDispatcherContext->ScopeIndex
:              fffff800\`008a4324 mov     qword ptr \[rax\-58h\],rcx ; \[rax-58h\] = pExceptionRecord，供给 GetExceptionCode(Information) 使用
:              fffff800\`008a4328 mov     qword ptr \[rax\-50h\],r8  ; \[rax-50h\] = pContext，供给 GetExceptionCode(Information) 使用
:              fffff800\`008a432c cmp     edi,dword ptr \[r12\]     ; cmp l\_ScopeIndex, pDispatcherContext->HandlerData->Count
:              fffff800\`008a4330 mov     rax,rdi                 ; rax = l\_ScopeIndex
:<             fffff800\`008a4333 jae     nt!\_\_C\_specific\_handler+0x166 (fffff800\`008a4436)
::             
::             nt!\_\_C\_specific\_handler+0x69:
::             fffff800\`008a4339 add     rax,rax             ; 这里 \*2，下面紧接着 \*8，目的是跳过指定数目的 ScopeRecord（大小为 16 字节）
::             fffff800\`008a433c lea     rbx,\[r12+rax\*8+0Ch\] ; rbx = &(pDispatcherContext->HandlerData->ScopeRecord\[l\_ScopeIndex\].HandlerAddress)
::             
::             nt!\_\_C\_specific\_handler+0x71:
::             ; 检查 ControlPc 处于哪个 \_\_try 保护域，之步骤一
::      >      fffff800\`008a4341 mov     eax,dword ptr \[rbx\-8\] ; eax = pDispatcherContext->HandlerData->ScopeRecord\[l\_ScopeIndex\].BeginAddress
::      :      fffff800\`008a4344 cmp     rbp,rax               ; cmp l\_OffsetInFunc, pDispatcherContext->HandlerData->ScopeRecord\[l\_ScopeIndex\].BeginAddress
::<     :      fffff800\`008a4347 jb      nt!\_\_C\_specific\_handler+0xdd (fffff800\`008a43ad)
:::     :      
:::     :      nt!\_\_C\_specific\_handler+0x79:
:::     :      ; 检查 ControlPc 处于哪个 \_\_try 保护域，之步骤二
:::     :      fffff800\`008a4349 mov     eax,dword ptr \[rbx\-4\] ; eax = pDispatcherContext->HandlerData->ScopeRecord\[l\_ScopeIndex\].EndAddress
:::     :      fffff800\`008a434c cmp     rbp,rax               ; cmp l\_OffsetInFunc, pDispatcherContext->HandlerData->ScopeRecord\[l\_ScopeIndex\].EndAddress
:::<    :      fffff800\`008a434f jae     nt!\_\_C\_specific\_handler+0xdd (fffff800\`008a43ad)
::::    :      
::::    :      nt!\_\_C\_specific\_handler+0x81:
::::    :      ; 判断是否是 \_\_try/\_\_finally（JumpTarget 为 NULL）。如果是，那么跳转到下一个 ScopeRecord 继续遍历。
::::    :      fffff800\`008a4351 cmp     dword ptr \[rbx+4\],0 ; cmp pDispatcherContext->HandlerData->ScopeRecord\[l\_ScopeIndex\].JumpTarget, NULL
::::<   :      fffff800\`008a4355 je      nt!\_\_C\_specific\_handler+0xdd (fffff800\`008a43ad)
:::::   :      
:::::   :      nt!\_\_C\_specific\_handler+0x87:
:::::   :      ; 到这里，已经找到与异常地址最匹配的 \_\_try/\_\_except
:::::   :      fffff800\`008a4357 mov     eax,dword ptr \[rbx\] ; eax = pDispatcherContext->HandlerData->ScopeRecord\[l\_ScopeIndex\].HandlerAddress
:::::   :      fffff800\`008a4359 cmp     eax,1               ; cmp pDispatcherContext->HandlerData->ScopeRecord\[l\_ScopeIndex\].HandlerAddress, EXCEPTION\_EXECUTE\_HANDLER (0x1)
:::::<  :      fffff800\`008a435c je      nt!\_\_C\_specific\_handler+0xa3 (fffff800\`008a4373) ; 如果返回 EXCEPTION\_EXECUTE\_HANDLER 则跳转
::::::  :      
::::::  :      nt!\_\_C\_specific\_handler+0x8e:
::::::  :      ; 是 \_\_try/\_\_except，且过滤域并不是 EXCEPTION\_EXECUTE\_HANDLER，执行 HandlerAddress 
::::::  :      ; （注：HandlerAddress 指向的函数仍有可能会返回 EXCEPTION\_EXECUTE\_HANDLER） 
::::::  :      fffff800\`008a435e lea     rcx,\[rsp+30h\]
::::::  :      fffff800\`008a4363 add     rax,r14 ; rax = pDispatcherContext->HandlerData->ScopeRecord\[l\_ScopeIndex\].HandlerAddress + pDispatcherContext->ImageBase
::::::  :      fffff800\`008a4366 mov     rdx,rsi ; rdx = pEstablisherFrame
::::::  :      fffff800\`008a4369 call    rax     ; 调用 EXCEPT\_FILTER
::::::  :      fffff800\`008a436b test    eax,eax
::::::< :      fffff800\`008a436d js      nt!\_\_C\_specific\_handler+0xee (fffff800\`008a43be) ; 返回 EXCEPTION\_CONTINUE\_EXECUTION (-1) 则跳转
::::::: :      
::::::: :      nt!\_\_C\_specific\_handler+0x9f:
::::::: :      fffff800\`008a436f test    eax,eax
:::::::<:      fffff800\`008a4371 jle     nt!\_\_C\_specific\_handler+0xdd (fffff800\`008a43ad) ; 返回 EXCEPTION\_CONTINUE\_SEARCH (0) 则跳转
:::::::::      
:::::::::      nt!\_\_C\_specific\_handler+0xa3:
:::::::::      ; 返回的是 EXCEPTION\_EXECUTE\_HANDLER
:::::>:::      fffff800\`008a4373 mov     ecx,dword ptr \[rbx+4\] ; ecx = pDispatcherContext->HandlerData->ScopeRecord\[l\_ScopeIndex\].JumpTarget
::::: :::      fffff800\`008a4376 mov     r8d,1
::::: :::      fffff800\`008a437c mov     rdx,rsi ; rdx = pEstablisherFrame
::::: :::      fffff800\`008a437f add     rcx,r14 ; rcx = pDispatcherContext->HandlerData->ScopeRecord\[l\_ScopeIndex\].JumpTarget + pDispatcherContext->ImageBase
::::: :::      fffff800\`008a4382 call    nt!\_NLG\_Notify (fffff800\`008b1460)
::::: :::      fffff800\`008a4387 mov     rax,qword ptr \[r13+40h\] ; rax = pDispatcherContext->HistoryTable
::::: :::      fffff800\`008a438b mov     edx,dword ptr \[rbx+4\]   ; edx = pDispatcherContext->HandlerData->ScopeRecord\[l\_ScopeIndex\].JumpTarget
::::: :::      fffff800\`008a438e movsxd  r9,dword ptr \[r15\]      ; r9 = pExceptionRecord->ExceptionCode
::::: :::      fffff800\`008a4391 mov     qword ptr \[rsp+28h\],rax ; \_ARG\_6 = pDispatcherContext->HistoryTable
::::: :::      fffff800\`008a4396 mov     rax,qword ptr \[r13+28h\] ; rax = pDispatcherContext->ContextRecord
::::: :::      fffff800\`008a439a add     rdx,r14 ; rdx = pDispatcherContext->HandlerData->ScopeRecord\[l\_ScopeIndex\].JumpTarget + pDispatcherContext->ImageBase
::::: :::      fffff800\`008a439d mov     r8,r15  ; r8 = pExceptionRecord
::::: :::      fffff800\`008a43a0 mov     rcx,rsi ; rcx = pEstablisherFrame
::::: :::      fffff800\`008a43a3 mov     qword ptr \[rsp+20h\],rax ; \_ARG\_5 = pDispatcherContext->ContextRecord
::::: :::      fffff800\`008a43a8 call    nt!RtlUnwindEx (fffff800\`00891e80) ; 这里不会返回
::::: :::      ; RtlUnwindEx(pEstablisherFrame, 
::::: :::      ;             pDispatcherContext->HandlerData->ScopeRecord\[l\_ScopeIndex\].JumpTarget + pDispatcherContext->ImageBase
::::: :::      ;             pExceptionRecord,
::::: :::      ;             pExceptionRecord->ExceptionCode
::::: :::      ;             pDispatcherContext->ContextRecord,
::::: :::      ;             pDispatcherContext->HistoryTable)
::::: :::      
::::: :::      nt!\_\_C\_specific\_handler+0xdd:
::>>> :>:      fffff800\`008a43ad inc     edi     ; l\_ScopeIndex += 1
::    : :      fffff800\`008a43af add     rbx,10h ; 调整到下一个 ScopeRecord::HandlerAddress
::    : :      fffff800\`008a43b3 cmp     edi,dword ptr \[r12\] ; cmp l\_ScopeIndex, pDispatcherContext->HandlerData->Count
::    : <      fffff800\`008a43b7 jb      nt!\_\_C\_specific\_handler+0x71 (fffff800\`008a4341)
::    :        
::    :        nt!\_\_C\_specific\_handler+0xe9:
::    :        ; pDispatcherContext->HandlerData 遍历完毕
::<   :        fffff800\`008a43b9 jmp     nt!\_\_C\_specific\_handler+0x166 (fffff800\`008a4436)
:::   :        
:::   :        nt!\_\_C\_specific\_handler+0xee:
:::   >        fffff800\`008a43be xor     eax,eax ; eax = ExceptionContinueExecution
:::<           fffff800\`008a43c0 jmp     nt!\_\_C\_specific\_handler+0x16b (fffff800\`008a443b)
::::           
::::           -------------------------------------------------------------------------------------
::::           nt!\_\_C\_specific\_handler+0xf5:
::::           ; 设置了 EXCEPTION\_UNWIND，当前是展开过程
\>:::           fffff800\`008a43c5 movsxd  rdi,dword ptr \[r9+48h\] ; l\_ScopeIndex (rdi) = pDispatcherContext->ScopeIndex
 :::           fffff800\`008a43c9 mov     rsi,qword ptr \[r9+20h\] ; rsi = pDispatcherContext->TargetIp
 :::           fffff800\`008a43cd sub     rsi,r14                ; rsi = pDispatcherContext->TargetIp - pDispatcherContext->ImageBase
 :::           fffff800\`008a43d0 cmp     edi,dword ptr \[r12\]    ; cmp l\_ScopeIndex, pDispatcherContext->HandlerData->Count
 :::           fffff800\`008a43d4 mov     rax,rdi                ; rax = l\_ScopeIndex
 :::<          fffff800\`008a43d7 jae     nt!\_\_C\_specific\_handler+0x166 (fffff800\`008a4436)
 ::::          
 ::::          nt!\_\_C\_specific\_handler+0x109:
 ::::          fffff800\`008a43d9 add     rax,rax ; 
 ::::          fffff800\`008a43dc lea     rbx,\[r12+rax\*8+8\] ; rbx = &(pDispatcherContext->HandlerData->ScopeRecord\[l\_ScopeIndex\].EndAddress)
 ::::          
 ::::          nt!\_\_C\_specific\_handler+0x111:
 ::::          ; 检查 ControlPc 处于哪个 \_\_try 保护域，之步骤一
 ::::        > fffff800\`008a43e1 mov     eax,dword ptr \[rbx\-4\] ; eax = pDispatcherContext->HandlerData->ScopeRecord\[l\_ScopeIndex\].BeginAddress
 ::::        : fffff800\`008a43e4 cmp     rbp,rax ; cmp l\_OffsetInFunc, pDispatcherContext->HandlerData->ScopeRecord\[l\_ScopeIndex\].BeginAddress
 ::::<       : fffff800\`008a43e7 jb      nt!\_\_C\_specific\_handler+0x15a (fffff800\`008a442a)
 :::::       : 
 :::::       : nt!\_\_C\_specific\_handler+0x119:
 :::::       : ; 检查 ControlPc 处于哪个 \_\_try 保护域，之步骤二
 :::::       : fffff800\`008a43e9 mov     ecx,dword ptr \[rbx\] ; ecx = pDispatcherContext->HandlerData->ScopeRecord\[l\_ScopeIndex\].EndAddress
 :::::       : fffff800\`008a43eb cmp     rbp,rcx ; cmp l\_OffsetInFunc, pDispatcherContext->HandlerData->ScopeRecord\[l\_ScopeIndex\].EndAddress
 :::::<      : fffff800\`008a43ee jae     nt!\_\_C\_specific\_handler+0x15a (fffff800\`008a442a)
 ::::::      : 
 ::::::      : nt!\_\_C\_specific\_handler+0x120:
 ::::::      : ; 到这里，已经找到与异常地址匹配的最内层（如果有多层） \_\_try/\_\_except
 ::::::      : fffff800\`008a43f0 cmp     rsi,rax ; cmp pDispatcherContext->TargetIp - pDispatcherContext->ImageBase, pDispatcherContext->HandlerData->ScopeRecord\[l\_ScopeIndex\].BeginAddress
 ::::::<     : fffff800\`008a43f3 jb      nt!\_\_C\_specific\_handler+0x131 (fffff800\`008a4401)
 :::::::     : 
 :::::::     : nt!\_\_C\_specific\_handler+0x125:
 :::::::     : fffff800\`008a43f5 cmp     rsi,rcx ; cmp pDispatcherContext->TargetIp - pDispatcherContext->ImageBase, pDispatcherContext->HandlerData->ScopeRecord\[l\_ScopeIndex\].EndAddress
 :::::::<    : fffff800\`008a43f8 ja      nt!\_\_C\_specific\_handler+0x131 (fffff800\`008a4401)
 ::::::::    : 
 ::::::::    : nt!\_\_C\_specific\_handler+0x12a:
 ::::::::    : ; 如果标记了 EXCEPTION\_TARGET\_UNWIND，说明是最后一个需要局部展开的函数。但是该次局部展开只展开到 EXCEPT\_HANDLER（不包含 EXCEPT\_HANDLER），所以需要判断 TargetIp
 ::::::::    : fffff800\`008a43fa test    byte ptr \[r15+4\],20h ; test pExceptionRecord->ExceptionFlags, EXCEPTION\_TARGET\_UNWIND (0x20)
 ::::::::<   : fffff800\`008a43ff jne     nt!\_\_C\_specific\_handler+0x166 (fffff800\`008a4436)
 :::::::::   : 
 :::::::::   : nt!\_\_C\_specific\_handler+0x131:
 ::::::>>:   : fffff800\`008a4401 mov     eax,dword ptr \[rbx+8\] ; eax = pDispatcherContext->HandlerData->ScopeRecord\[l\_ScopeIndex\].JumpTarget
 ::::::  :   : fffff800\`008a4404 test    eax,eax ; 判断 pDispatcherContext->HandlerData->ScopeRecord\[l\_ScopeIndex\].JumpTarget 是否为 NULL，即是否是 \_\_try/\_\_finally
 ::::::  :<  : fffff800\`008a4406 je      nt!\_\_C\_specific\_handler+0x13f (fffff800\`008a440f) ; 如果是 \_\_try/\_\_finally 则跳转
 ::::::  ::  : 
 ::::::  ::  : nt!\_\_C\_specific\_handler+0x138:
 ::::::  ::  : fffff800\`008a4408 cmp     rsi,rax ; cmp pDispatcherContext->TargetIp, pDispatcherContext->HandlerData->ScopeRecord\[l\_ScopeIndex\].JumpTarget
 ::::::  ::< : fffff800\`008a440b je      nt!\_\_C\_specific\_handler+0x166 (fffff800\`008a4436)
 ::::::  ::: : 
 ::::::  ::: : nt!\_\_C\_specific\_handler+0x13d:
 ::::::  :::<: fffff800\`008a440d jmp     nt!\_\_C\_specific\_handler+0x15a (fffff800\`008a442a)
 ::::::  ::::: 
 ::::::  ::::: nt!\_\_C\_specific\_handler+0x13f:
 ::::::  ::::: ; 注意这里是先修改 pDispatcherContext->ScopeIndex，然后调用 EXCEPT\_HANDLER。这样如果 EXCEPT\_HANDLER 触发异常，后续展开就会跳过这个 EXCEPT\_HANDLER。
 ::::::  :>::: fffff800\`008a440f mov     rdx,qword ptr \[rsp+98h\]
 ::::::  : ::: fffff800\`008a4417 lea     eax,\[rdi+1\]             ; eax = l\_ScopeIndex + 1
 ::::::  : ::: fffff800\`008a441a mov     cl,1
 ::::::  : ::: fffff800\`008a441c mov     dword ptr \[r13+48h\],eax ; pDispatcherContext->ScopeIndex = eax
 ::::::  : ::: fffff800\`008a4420 mov     r8d,dword ptr \[rbx+4\]   ; r8d = pDispatcherContext->HandlerData->ScopeRecord\[i\].HandlerAddress
 ::::::  : ::: fffff800\`008a4424 add     r8,r14                  ; r8 = pDispatcherContext->HandlerData->ScopeRecord\[i\].HandlerAddress + pDispatcherContext->ImageBase
 ::::::  : ::: fffff800\`008a4427 call    r8                      ; 调用 \_\_finally 处理块，会返回（注：对于 \_\_try/\_\_finally，HandlerAddress 保存的是 \_\_finally 代码块的 RVA）
 ::::::  : ::: 
 ::::::  : ::: nt!\_\_C\_specific\_handler+0x15a:
 ::::>>  : :>: fffff800\`008a442a inc     edi                 ; l\_ScopeIndex += 1
 ::::    : : : fffff800\`008a442c add     rbx,10h             ; 调整到下一个 ScopeRecord::HandlerAddress
 ::::    : : : fffff800\`008a4430 cmp     edi,dword ptr \[r12\] ; cmp l\_ScopeIndex, pDispatcherContext->HandlerData->Count
 ::::    : : < fffff800\`008a4434 jb      nt!\_\_C\_specific\_handler+0x111 (fffff800\`008a43e1)
 ::::    : :   
 ::::    : :   nt!\_\_C\_specific\_handler+0x166:
 >>:>    > >   fffff800\`008a4436 mov     eax,1 ; eax = ExceptionContinueSearch (0n1)
   :           
   :           nt!\_\_C\_specific\_handler+0x16b:
   >           fffff800\`008a443b mov     r15,qword ptr \[rsp+48h\]
               fffff800\`008a4440 mov     r14,qword ptr \[rsp+50h\]
               fffff800\`008a4445 mov     r13,qword ptr \[rsp+58h\]
               fffff800\`008a444a mov     r12,qword ptr \[rsp+60h\]
               fffff800\`008a444f mov     rdi,qword ptr \[rsp+68h\]
               fffff800\`008a4454 mov     rsi,qword ptr \[rsp+70h\]
               fffff800\`008a4459 mov     rbp,qword ptr \[rsp+78h\]
               fffff800\`008a445e mov     rbx,qword ptr \[rsp+80h\]
               fffff800\`008a4466 add     rsp,88h
               fffff800\`008a446d ret

```

  nt!\_\_C\_specific\_handler 相当于 x86 中的 nt!\_except\_handler3。从上面的反汇编代码也可以看出它的逻辑跟 nt!\_except\_handler3 基本上一致。  
  函数代码不长。主要分为两个大分支，一个分支处理异常，一个分支处理展开（我用横线分隔开了）。  
  
  异常解决的代码负责遍历 SCOPE\_TABLE，依次调用 SCOPE\_TABLE::ScopeRecord.HandlerAddress 代表的 EXCEPT\_FILTER，并针对返回值做出相应的处理：  
    1. 返回 EXCEPTION\_CONTINUE\_EXECUTION，说明异常已经被 EXCEPT\_FILTER 修复。返回 ExceptionContinueExecution。  
    2. 返回 EXCEPTION\_CONTINUE\_SEARCH，继续遍历下一个 ScopeRecord。  
    3. 返回 EXCEPTION\_EXECUTE\_HANDLER，说明当前 ScopeRecord.JumpTarget 代表的 EXCEPT\_HANDLER 可以处理该异常。那么调用 RtlUnwindEx 进行展开。  
  
  熟悉 x86 的朋友可能会疑惑：在 x86 中 nt!\_except\_handler3 先进行全局展开，然后对本函数自身进行不完全的局部展开，最后执行 EXCEPT\_HANDLER。而在 nt!\_\_C\_specific\_handler 中却找不到执行 EXCEPT\_HANDLER 的指令，这是怎么回事？  
  实际上，x64 对这个流程做了一些调整，EXCEPT\_HANDLER 不是由 nt!\_\_C\_specific\_handler 直接调用，而是作为参数传给 RtlUnwindEx，RtlUnwindEx 处理完展开之后才执行 EXCEPT\_HANDLER。后续我们在讲展开的时候会看到具体的方法。  
  
  \_\_C\_specific\_handler 的展开分支，是对 SCOPE\_TABLE 进行展开，逻辑很简单，不多讲了。  
  
  更详细的信息，请参考上面反汇编代码中我附的注释。  
  
  另外还需要说一下 SCOPE\_TABLE。  
  在 x86 中，遍历 scopetable 时是通过运行时动态改变的 EXCEPTION\_REGISTRATION::trylevel 来确定应该首先遍历哪一个 scopetable\_entry。而 x64 中没有等同于 trylevel 的数据，有的朋友可能会说“SCOPE\_TABLE 中不是有每个 \_\_try 保护域的范围 RVA 吗？通过范围不就可以确定在哪个 \_\_try 中触发了异常吗？”。  
  我们可以先试试这种方法，以下面这段伪码为例，  
  
代码:

```
  1 VOID SehTest()
  2 {
  3     \_\_try // 1
  4     {
  5     }
  6     \_\_except()
  7     {
  8     }
  9 
  10    \_\_try // 2
  11    {
  12        \_\_try // 3
  13        {
  14            ...
  15        }
  16        \_\_except()
  17        {
  18        }
  19    }
  20    \_\_except()
  21    {
  22    }
  23
  24    \_\_try // 4
  25    {
  26    }
  27    \_\_finally()
  28    {
  29    }
  30}

```

  上述伪码中总共有 4 个 \_\_try，按照 x86 中的方法，SCOPE\_TABLE 的内容应该是顺序排列的，像这样：  
  
  SCOPE\_TABLE::Count 等于 4，  
  SCOPE\_TABLE::ScopeRecord\[0\] 表示行 3 开始的 \_\_try/\_\_except，  
  SCOPE\_TABLE::ScopeRecord\[1\] 表示行 10 开始的 \_\_try/\_\_except，  
  SCOPE\_TABLE::ScopeRecord\[2\] 表示行 12 开始的 \_\_try/\_\_except，  
  SCOPE\_TABLE::ScopeRecord\[3\] 表示行 24 开始的 \_\_try/\_\_finally。  
  
  假设行 14 处触发了异常，遍历过程应该是这样，  
  首先检查 ScopeRecord\[0\]，发现其范围不包含 EXCEPT\_POINT，继续下一个，  
  开始检查 ScopeRecord\[1\]，范围匹配了。  
  
  那是不是该把异常交给 ScopeRecord\[1\] 处理呢？  
  不是！从伪码中可以很明显的看出，行 14 触发的异常应该首先由行 12 开始的 \_\_try/\_\_except，即 ScopeRecord\[1\] 处理。  
  
  可见这种方法是行不通的。  
  MSC 通过调整 SCOPE\_TABLE::ScopeRecord 的排列顺序来解决这个问题：  
  
  SCOPE\_TABLE::Count 等于 4，  
  SCOPE\_TABLE::ScopeRecord\[0\] 表示行 3 开始的 \_\_try/\_\_except，  
  SCOPE\_TABLE::ScopeRecord\[1\] 表示行 12 开始的 \_\_try/\_\_except，  
  SCOPE\_TABLE::ScopeRecord\[2\] 表示行 10 开始的 \_\_try/\_\_except，  
  SCOPE\_TABLE::ScopeRecord\[3\] 表示行 24 开始的 \_\_try/\_\_finally。  
  
  即对于嵌套的 \_\_try/\_\_except/\_\_finally，ScopeRecord 的排列顺序是，最内层的 \_\_try 排在前面，其次是次内层的，依次排到最外层。  
  这样就能正确的遍历 SCOPE\_TABLE 了。  
  
  再用伪码完整的展示一下 SCOPE\_TABLE 的布置，  
  
  SCOPE\_TABLE::Count = 4。  
  
  SCOPE\_TABLE::ScopeRecord\[0\].BeginAddress = RVA\_L4; （行 4 的 RVA） // 第一个 \_\_try  
  SCOPE\_TABLE::ScopeRecord\[0\].EndAddress = RVA\_L5;  
  SCOPE\_TABLE::ScopeRecord\[0\].HandlerAddress = RVA\_L6\_EXCEPT\_FILTER; （行 6 \_\_except 过滤代码首地址的 RVA）  
  SCOPE\_TABLE::ScopeRecord\[0\].JumpTarget = RVA\_L7;   
  
  SCOPE\_TABLE::ScopeRecord\[1\].BeginAddress = RVA\_L13;                 // 第三个 \_\_try  
  SCOPE\_TABLE::ScopeRecord\[1\].EndAddress = RVA\_L15;  
  SCOPE\_TABLE::ScopeRecord\[1\].HandlerAddress = RVA\_L16\_EXCEPT\_FILTER;   
  SCOPE\_TABLE::ScopeRecord\[1\].JumpTarget = RVA\_L7;   
  
  SCOPE\_TABLE::ScopeRecord\[2\].BeginAddress = RVA\_L11;                 // 第二个 \_\_try  
  SCOPE\_TABLE::ScopeRecord\[2\].EndAddress = RVA\_L19;  
  SCOPE\_TABLE::ScopeRecord\[2\].HandlerAddress = RVA\_L20\_EXCEPT\_FILTER;  
  SCOPE\_TABLE::ScopeRecord\[2\].JumpTarget = RVA\_L21;   
  
  SCOPE\_TABLE::ScopeRecord\[3\].BeginAddress = RVA\_L25;                 // 第四个 \_\_try  
  SCOPE\_TABLE::ScopeRecord\[3\].EndAddress = RVA\_L26;  
  SCOPE\_TABLE::ScopeRecord\[3\].HandlerAddress = RVA\_L28;  
  SCOPE\_TABLE::ScopeRecord\[3\].JumpTarget = 0;   
  
  我们再模拟一下 nt!\_\_C\_specific\_handler 是如何遍历 SCOPE\_TABLE 的：  
  1. 首先通过传入参数中的 pDispatcherContext->ControlPc 和 pDispatcherContext->ImageBase 计算出异常触发点的 RVA（简称 E\_RVA）。参见 fffff800\`008a430d 处的指令。  
  2. 通过 pDispatcherContext->ScopeIndex 确认是否需要遍历。如果需要遍历，则从它指定的 ScopeRecord 开始遍历。pDispatcherContext->ScopeIndex 一般都为 0，只有返回 ExceptionCollidedUnwind 时，RtlDispatchException 才可能将它设置为其他值。  
  3. 通过比较 E\_RVA 和 ScopeRecord\[?\].BeginAddress、ScopeRecord\[?\].EndAddress 来找到正确的处理函数，  
    首先 ScopeRecord\[0\] 范围不匹配，遍历下一个，  
    然后 ScopeRecord\[1\]，发现范围匹配，并且是 \_\_try/\_\_except 组合。于是调用 ScopeRecord\[1\].HandlerAddress，假设它返回的是 EXCEPTION\_CONTINUE\_SEARCH，那么继续遍历下一个，  
    这次是 ScopeRecord\[2\]，发现范围匹配，并且是 \_\_try/\_\_except 组合。于是调用 ScopeRecord\[2\].HandlerAddress，假设它返回的是 EXCEPTION\_EXECUTE\_HANDLER，那么说明找到了解决方案。  
  4. 调用 RtlUnwindEx，把 ScopeRecord\[2\].JumpTarget 对应的绝对地址作为 TargetIp 参数传给它。RtlUnwindEx 全局展开完毕后执行 TargetIp。  
  
  到这里，异常分发就大致讲述完毕。接下来是关于展开和解决的内容。  
  
  
**三、展开、解决**  
  
  x64 中展开使用的函数有 RtlVirtualUnwind、RtlUnwindEx 和 RtlpExecuteHandlerForUnwind。其中 RtlVirtualUnwind 已经讲了，我们来看看余下的两个。  
  
  首先是 RtlUnwindEx，原型如下：  
  
代码:

```
  VOID
  RtlUnwindEx (
      IN PVOID TargetFrame OPTIONAL,
      IN PVOID TargetIp OPTIONAL,
      IN PEXCEPTION\_RECORD ExceptionRecord OPTIONAL,
      IN PVOID ReturnValue,
      IN PCONTEXT OriginalContext,
      IN PUNWIND\_HISTORY\_TABLE HistoryTable OPTIONAL
      );

```

  参数分别是：  
    TargetFrame  目标帧，即最后一个需要展开的帧。  
    TargetIp  前面有讲过，它就是 ScopeRecord\[?\].JumpTarget 代表的地址，即 EXCEPT\_HANDLER。  
    ExceptionRecord  异常信息。  
    ReturnValue  传递给 TargetIp 的返回值，分析过程中没发现它有什么用处。  
    OriginalContext  虽然被声明为 IN，但是实际上 RtlUnwindEx 并没有使用它内部的数据，  
    HistoryTable  用于加速查找 RUNTIME\_FUNCTION。  
  
  主要功能是：  
    从自身开始展开，到 TargetFrame 停止。然后跳转到 TargetIp 继续执行。  
  流程：  
    1. 申请一个类型为 CONTEXT 的局部变量 l\_Context，调用 RtlCaptureContext 将当前自身的环境复制到 l\_Context。  
    2. 通过 RtlVirtualUnwind 对 l\_Context 进行模拟展开，推动遍历。对每个遍历到的 UNWIND\_INFO，检查 UNWIND\_INFO::Flags 是否包含 UNW\_FLAG\_UHANDLER。如果包含，则调用 UNWIND\_INFO::ExceptionHandler 进行局部展开。否则继续遍历下一个。  
       循环本步骤，直到展开到 TargetFrame，即到达解决异常的 EXCEPT\_HANDLER 所在的函数（简称为 ExceptionHandlerFunc）了。  
    3. 这时 l\_Context 内已经是从 RtlUnwindEx 完整展开到 EXCEPT\_HANDLER 的环境了。即此时 l\_Context 已经是 ExceptionHandlerFunc 的执行环境了。  
       调用 RtlRestoreContext，用 l\_Context 替换当前线程的执行环境，于是就跳转到 EXCEPT\_HANDLER 继续执行。  
  这样就完美的从触发异常的环境跳到了新的环境中。  
  
  这个过程有点类似这种手法：  
    1. 将某台机器的系统 ghost 到一个 bak.gho 文件。  
    2. 把 bak.gho 恢复到一台临时机器，然后对这台临时机器做一些调整。调整完毕后制作一个临时机器的 bak\_mod.gho。  
    3. 将 bak\_mod.gho 恢复到原来那台机器。  
  
  这个流程很重要，我手绘了一副图帮助理解，  
  
  伪码：  
代码:

```
  \_\_try
  {
      ExRaiseStatus(STATUS\_INVALID\_PARAMETER);
  }
  \_\_except(EXCEPTION\_EXECUTE\_HANDLER)
  {
  }

```

  图：  
  1. 异常解决流程，从 EXCEPT\_POINT 到 RtlUnwindEx，途中已经找到能够解决该异常的 EXCEPT\_HANDLER 了（以参数 TargetIp 表示），当前线程状态为 ThreadContext  
     +---------------------------------------+  
     | ......                                |  
     | RtlRaiseStatus                    |调 |  
     | RtlDispatchException              |用 |-> ThreadContext &  
     | RtlpExecuteHandlerForException    |方 |   TargetIp = ExceptionHandler  
     | \_\_C\_specific\_handler              |向 |  
     | RtlUnwindEx                       v   |  
     |                                       |  
     +---------------------------------------+  
  
  2. RtlUnwindEx 将当前自身状态复制到 ThreadContext\_Copy 中  
     +---------------------------------------+                                     +---------------------------------------+  
     | ......                                |                                     | ......                                |  
     | RtlRaiseStatus                    |调 |                                     | RtlRaiseStatus                    |调 |  
     | RtlDispatchException              |用 |-> ThreadContext &                   | RtlDispatchException              |用 |-> ThreadContext\_Copy &  
     | RtlpExecuteHandlerForException    |方 |   TargetIp = ExceptionHandler       | RtlpExecuteHandlerForException    |方 |   TargetIp = ExceptionHandler  
     | \_\_C\_specific\_handler              |向 |                                     | \_\_C\_specific\_handler              |向 |  
     | RtlUnwindEx                       v   |                                     | RtlUnwindEx                       v   |  
     |                                       |                                     |                                       |  
     +---------------------------------------+                                     +---------------------------------------+  
  
  3. 用 ThreadContext\_Copy 进行展开，一直展开到异常触发点停止。  
     +---------------------------------------+                                     +---------------------------------------+  
     | .....                                 |                                     | ......                                |  
     | RtlRaiseStatus                    |调 |                                     | RtlRaiseStatus                    ^展 |  
     | RtlDispatchException              |用 |-> ThreadContext                     | RtlDispatchException              |开 |-> ThreadContext\_Copy &  
     | RtlpExecuteHandlerForException    |方 |   TargetIp = ExceptionHandler       | RtlpExecuteHandlerForException    |方 |   TargetIp = ExceptionHandler  
     | \_\_C\_specific\_handler              |向 |                                     | \_\_C\_specific\_handler              |向 |  
     | RtlUnwindEx                       v   |                                     | RtlUnwindEx                       |   |  
     |                                       |                                     |                                       |  
     +---------------------------------------+                                     +---------------------------------------+  
  
  4. 将 ThreadContext\_Copy.Rip 设置为 TargetIp，以 ThreadContext\_Copy 为参数调用 RtlpRestoreContext。跳转到 TargetIp 继续执行。  
     +---------------------------------------+  
     | ......                                |  
     | EXCEPT\_HANDLER                        |-> ThreadContext (ThreadContext.Rip = TargetIp)  
     |                                       |  
     +---------------------------------------+  
  
  这样就完成了展开和执行 EXCEPT\_HANDLER 的工作。  
  
  RtlpExecuteHandlerForUnwind 没有什么改变，原型依旧：  
  
代码:

```
  EXCEPTION\_DISPOSITION
  RtlpExecuteHandlerForUnwind (
      IN PEXCEPTION\_RECORD ExceptionRecord,
      IN PVOID EstablisherFrame,
      IN OUT PCONTEXT ContextRecord,
      IN OUT PVOID DispatcherContext
      );

```

  它会注册一个异常处理函数 RtlpUnwindHandle，当触发新异常时 RtlpUnwindHandler 会返回 ExceptionCollidedUnwind。关于 ExceptionCollidedUnwind，我们后面还会详细讲述。  
  RtlpExecuteHandlerForUnwind 的实现源码位于 $\\WRK-v1.2\\base\\ntos\\rtl\\amd64\\xcptmisc.asm:199。  
  RtlpUnwindHandle 的实现源码位于 $\\WRK-v1.2\\base\\ntos\\rtl\\amd64\\xcptmisc.asm:136。  
  
  到这里，我们讲完了展开的逻辑。接下来我们要讲述两个比较特殊的返回值： ExceptionNestedException 和 ExceptionCollidedUnwind。  
  
  
  四、ExceptionNestedException 和 ExceptionCollidedUnwind  
  
  之所以专门讲述这两个返回值，是因为在分析过程中，我感觉常规情况的 SEH 流程理解起来并不困难，难理解的是这两种不一般的情况。它们不一般之处在于：在处理异常的过程中又触发了新的异常。  
  先来讲一下这两个返回值的含义：  
  ExceptionNestedException  在异常分发过程中触发新的异常，比如执行 EXCEPT\_FILTER 时触发异常。  
  ExceptionCollidedUnwind  在展开过程中触发新的异常，比如执行 FINALLY\_HANDLER 时触发异常。  
  
  首先来讲讲 ExceptionNestedException，以如下伪码为例：  
  
代码:

```
    1  VOID SehTest()
    2  {
    3      \_\_try
    4      {
    5          ExRaiseStatus();
    6      }
    7      \_\_except(ExRaiseStatus(), EXCEPTION\_CONTINUE\_SEARCH) // EXCEPT\_FILTER\_1
    8      { // EXCEPT\_HANDLER\_1
    9      }
    10 }
    11 
    12 VOID Caller()
    13 {
    14     \_\_try
    15     {
    16         SehTest();
    17     }
    18     \_\_except(EXCEPTION\_EXECUTE\_HANDLER) // EXCEPT\_FILTER\_2
    19     { // EXCEPT\_HANDLER\_2
    20     }
    21 }

```

  上述代码会两次触发异常，第一次是行 5 的 ExRaiseStatus，第二次是行 7 的 ExRaiseStatus。为了方便区分，我将它们分别标记为 EXCEPT\_POINT#1、EXCEPT\_POINT#2。  
  我们来看一下这两个异常的处理流程：  
  
  1. ExRaiseStatus#1 会创建保存 EXCEPT\_POINT#1 触发时的状态 Context#1，并构建一个 EXCEPTION\_RECORD，然后将他们作为参数来调用 RtlDispatchException#1。（注：这种方式的的触发点是 ExRaiseStatus 内部，而非 SehTest 的第 5 行。即 Context#1 记录的异常触发点是 ExRaiseStatus#1 内部）  
  
  2. RtlDispatchException#1 根据 Context#1 首先找到 EXCEPT\_POINT#1 所在函数 ExRaiseStatus#1 的 UNWIND\_INFO，发现其 UNWIND\_INFO::Flags 为 UNW\_FLAG\_NHANDLER，于是继续遍历。  
  
  3. RtlDispatchException#1 遍历到 SehTest，发现其 UNWIND\_INFO::Flags 为 UNW\_FLAG\_EHANDLER，于是调用其 UNWIND\_INFO::ExceptionHandler，即 \_\_C\_specific\_handler$2。\_\_C\_specific\_handler$2 遍历 SehTest 的 SCOPE\_TABLE，发现唯一的一个 ScopeRecord。于是执行 ScopeRecord\[0\].HandlerAddress，即行 7 的 SehTest::EXCEPT\_FILTER\_1#1。此时的调用栈如下（竖线后的内容为函数的 UNWIND\_INFO::Flags 和 UNWIND\_INFO::ExceptionHandler，其中 Flags 缩写为 E、U、N）：  
    (1)  Caller                                 | E  & \_\_C\_specific\_handler$1  
    (2)  SehTest                                | E  & \_\_C\_specific\_handler$2  
    (3)  ExRaiseStatus#1                        | N  
    (4)  RtlDispatchException#1                 | N  
    (5)  RtlpExecuteHandlerForException#1       | EU & RtlpExceptionHandler$5  
    (6)  \_\_C\_specific\_handler$2                 | N  
    (7)  EXCEPT\_FILTER\_1#1                      | N  
  
  4. EXCEPT\_FILTER#1 触发 EXCEPT\_POINT#2。同步骤 1 类似，ExRaiseStatus 会调用 RtlDispatchException#2，这个过程中同样会创建保存 EXCEPT\_POINT#2 的状态，我们称之为 Context#2。  
  
  5. RtlDispatchException#2 根据 Context#2 找到了 EXCEPT\_POINT#2 所在函数 ExRaiseStatus#2，发现其 UNWIND\_INFO::Flags 为 UNW\_FLAG\_NHANDLER，于是继续遍历。  
  
  6. RtlDispatchException#2 遍历到 EXCEPT\_FILTER\_1#1，发现其 UNWIND\_INFO::Flags 为 UNW\_FLAG\_NHANDLER，于是继续遍历。（注：EXCEPT\_FILTER 虽然代码形式上从属于 SehTest 函数，但实际上它是一个单独的函数，有自己的 UNWIND\_INFO，跟 SEH 的 UNWIND\_INFO 并不是同一个）  
  
  7. RtlDispatchException#2 遍历到 \_\_C\_specific\_handler$2、RltpExecuteHandlerForException#1、RtlDispatchException#1、ExRaiseStatus#1，这些函数要么被标记为 UNW\_FLAG\_NHANDLER，要么 UNWIND\_INFO::ExceptionHandler 返回 ExcetpionNestedException，结果都是继续遍历，所以不再一一讲述。继续遍历下一个。  
  
  8. RtlDispatchException#2 遍历到 SehTest，发现其 UNWIND\_INFO::Flags 为 UNW\_FLAG\_EHANDLER，于是调用其 UNWIND\_INFO::ExceptionRoutine，即 \_\_C\_specific\_handler$2，发现范围匹配，于是调用 EXCEPT\_FILTER\_1#1，于是又触发异常，这次是 #3 异常。此时的调用栈如下：  
    (1)  Caller                                 | E  & \_\_C\_specific\_handler$1  
    (2)  SehTest                                | E  & \_\_C\_specific\_handler$2  
    (3)  ExRaiseStatus#1                        | N  
    (4)  RtlDispatchException#1                 | N  
    (5)  RtlpExecuteHandlerForException#1       | EU & RtlpExceptionHandler$5  
    (6)  \_\_C\_specific\_handler$2                 | N  
    (7)  EXCEPT\_FILTER\_1#1                      | N  
    (8)  ExRaiseStatus#2                        | N  
    (9)  RtlDispatchException#2                 | N  
    (10) RtlpExecuteHandlerForException#2       | EU & RtlpExceptionHandler$10  
    (11) \_\_C\_specific\_handler$2                 | N  
    (12) EXCEPT\_FILTER\_1#2                      | N  
    (13) ExRaiseStatus#3                        | N  
  
  9. #3 异常的处理流程同 #2 的处理流程类似，也会再遍历到 \_\_C\_specific\_handler$2，也会再调用 EXCEPT\_FILTER\_1，于是会触发 #4 异常、#5 异常等等。最终内核栈溢出，BSOD。  
  
  以上就是 ExceptionNestedException 的产生以及处理的流程。过程中还有一些细节操作，为了描述简洁，我没有在上述过程中一一讲述。  
  
  
  再来看看 ExceptionCollidedUnwind。它比 ExceptionNestedException 更复杂一些，我们以如下伪码为例，  
  
代码:

```
    1  VOID SehTest()
    2  {
    3      \_\_try
    4      {
    5          ExRaiseStatus();
    6      }
    7      \_\_finally
    8      { // FINALLY\_HANDLER\_1
    9          ExRaiseStatus();
    10     }
    11 }
    12 
    13 VOID Caller()
    14 {
    15     \_\_try
    16     {
    17         SehTest();
    18     }
    19     \_\_except(EXCEPTION\_EXECUTE\_HANDLER) // EXCEPT\_FILTER\_2
    20     { // EXCEPT\_HANDLER\_2
    21     }
    22 }

```

  伪码中也有两处触发异常的地方，第一次在行 5，第二次在行 9。也分别标记为 EXCEPT\_POINT#1 和 EXCEPT\_POINT#2。处理流程：  
  
  1. ExRaiseStatus#1 创建保存 EXCEPT\_POINT#1 的状态 Context#1，并构建一个 EXCEPTION\_RECORD，然后将他们作为参数来调用 RtlDispatchException#1。  
  
  2. RtlDispatchException#1 根据 Context#1 开始遍历：  
    首先遍历到 EXCEPT\_POINT#1 所在函数 ExRaiseStatus，发现其 UNWIND\_INFO::Flags 为 UNW\_FLAG\_NHANDLER，于是遍历下一个。  
    然后遍历到 SehTest，发现其 UNWIND\_INFO::Flags 为 UNW\_FLAG\_UHANDLER，于是继续遍历下一个。  
    然后遍历到 Caller，发现其 UNWIND\_INFO::Flags 为 UNW\_FLAG\_EHANDLER，于是调用其 UNWIND\_INFO::ExceptionRoutine 即 \_\_C\_specific\_handler。\_\_C\_specific\_handler 发现可以处理该异常，于是以 EXCEPT\_HANDLER\_2 为 TargetIp 参数调用 RtlUnwindEx。  
  
  3. RtlUnwindEx 从自身开始展开，展开到 SehTest，执行 FINALLY\_HANDLER\_1 时触发新异常。此时调用栈为：  
     (1)  Caller                              | E  & \_\_C\_specific\_handler$1  
     (2)  SehTest                             | U  & \_\_C\_specific\_handler$2  
     (3)  ExRaiseStatus#1                     | N  
     (4)  RtlDispatchException#1              | N  
     (5)  RtlpExecuteHandlerForException#1    | EU & RtlpExceptionHandler$5  
     (6)  \_\_C\_specific\_handler$1              | N  
     (7)  RtlUnwindEx#1                       | N  
     (8)  RtlpExecuteHandlerForUnwind#1       | EU & RtlpUnwindHandler$8  
     (9)  \_\_C\_specific\_handler$2              | N  
     (10) FINALLY\_HANDLER\_1                   | N  
     (11) ExRaiseStatus#2                     | N  
     需要说明的是，调用栈(7) RtlUnwindEx 创建并初始化了一个 DISPATCHER\_CONTEXT 变量（后续称之为 pDispatcherContextForUnwind），并作为参数传递给调用栈(8) RltpExecuteHandlerForUnwind，后者在调用(9) \_\_C\_specific\_handler$2 之前将 pDispatcherContextForUnwind 保存在自己的栈中。此时 pDispatcherContextForUnwind 的内容表示的是调用栈(2) SehTest 的情况。后续步骤会用到这个 pDispatcherContextForUnwind。  
  
  4. (11) ExRaiseStatus#2 将 EXCEPT\_POINT#2 触发时的状态保存到 Context#2，然后调用 RtlDispatchException#2 进行 EXCEPT\_POINT#2 的分发。  
  
  5. RtlDispatchException#2 根据 Context#2 开始遍历，  
    首先遍历到 EXCEPT\_POINT#2 所在函数 ExRaiseStatus#2，发现其 UNWIND\_INFO::Flags 为 UNW\_FLAG\_NHANDLER，于是遍历下一个。  
    然后遍历到 FINALLY\_HANDLER\_1（同前面提到的 EXCEPT\_FILTER 一样，FINALLY\_HANDLER 实际上也是一个单独的函数，有自己的 RUNTIME\_FUNCTION 和 UNWIND\_INFO），发现其 UNWIND\_INFO::Flags 为  UNW\_FLAG\_NHANDLER，于是遍历下一个。  
       然后遍历到(9) \_\_C\_specific\_handler$2，发现其 UNWIND\_INFO::Flags 为 UNW\_FLAG\_NHANDLER，于是继续遍历。  
    然后遍历到(8) RtlpExecuteHandlerForUnwind#1，发现其 UNWIND\_INFO::Flags 包含 UNW\_FLAG\_EHANDLER。于是调用其 UNWIND\_INFO::ExceptionRoutine 即 RtlpUnwindHandler$8。RtlpUnwindHandler$8 会取出步骤 3 中所提到的 pDispatcherContextForUnwind，将其内容拷贝到自己的传出参数（参考 RtlpUnwindHandler 的函数原型）pDispatcherContext 中，然后返回 ExceptionCollidedUnwind。  
  
  6. RtlDispatchException#2 收到 ExceptionCollidedUnwind 后，从传回来的 pDispatchContext 中取出诸如 ControlPc、EstablisherFrame 等值（如步骤 3 所说，此时这些值反应的是(2) SehTest 的状态），用这些值来继续遍历。  
    首先遍历到(2) SehTest，调用 RtlpExecuteHandlerForException#2，进而调用 \_\_C\_specific\_handler$2，但是发现 pDispatcherContext->ScopeIndex（步骤(9)中在调用(10) FINALLY\_HANDLER\_1 之前+1 了，参考 \_\_C\_specific\_handler 反汇编码）等于 pDispatcherContext->HandlerData->Count。于是继续遍历。  
    然后遍历到(1) Caller，调用 RtlpExecuteHandlerForException#2，进而调用 \_\_C\_specific\_handler$1，发现它可以处理 #2 异常，于是以 EXCEPT\_HANDLER#2 为 TargetIp 参数调用 RtlUnwindEx。此时调用栈如下：  
     (1)  Caller                              | E  & \_\_C\_specific\_handler$1  
     (2)  SehTest                             | U  & \_\_C\_specific\_handler$2  
     (3)  ExRaiseStatus#1                     | N  
     (4)  RtlDispatchException#1              | N  
     (5)  RtlpExecuteHandlerForException#1    | EU & RtlpExceptionHandler$6  
     (6)  \_\_C\_specific\_handler$1              | N  
     (7)  RtlUnwindEx#1                       | N  
     (8)  RtlpExecuteHandlerForUnwind#1       | EU & RtlpUnwindHandler$8  
     (9)  \_\_C\_specific\_handler$2              | N  
     (10) FINALLY\_HANDLER#1                   | N  
     (11) ExRaiseStatus#2                     | N  
     (12) RtlDispatchException#2              | N  
     (13) RtlpExecuteHandlerForException#2    | EU & RtlpExceptionHandler  
     (14) \_\_C\_specific\_handler$1              | N  
     (15) RtlUnwindEx                         | N  
  
  7. (17) RtlUnwindEx 展开完毕后，通过 RtlRestoreContext 跳转到 EXCEPT\_HANDLER#2 继续执行。  
  
  在上述过程中，我们可以发现，遍历过程中 RtlDispatchException 等系统函数被频繁遍历到。于是就有了前面提到的全局展开历史表 RtlpUnwindHistoryTable，这个表中存放着 RtlDispatchException、RtlUnwindEx 等函数的 RUNTIME\_FUNCTION 和 ImageBase 信息，这样就不用每次都去解析 PE+ 中的 ExceptionDirectory，实现了加速。  
  
  
  到这里，我们就讲完了 x64 SEH 的实现。可以发现，x64 和 x86 的 SEH 思想或者说框架是一样的：  
  1. RtlDispatchException 和 RtlUnwindEx 都对异常注册信息进行遍历。前者是为了分发异常而遍历，后者是为了展开而遍历。  
  2. MSC 提供的异常处理函数按照“异常解决”和“展开”两个分支，对 SCOPE\_TABLE/scopetable 进行遍历。前者是为了找到 EXCEPT\_FILTER & EXCEPT\_HANDLER，后者是为了找到 FINALLY\_HANDLER。  
  3. RtlDispatchException 和 RtlUnwindEx 借助 MSC 提供的异常处理函数这个桥梁，配合处理异常。  
  主要的改变有两点：  
  1. RtlDispatchException 和 RtlUnwindEx 通过调用 RtlVirtualUnwind 推动遍历。  
  2. 所有的非叶函数都参与到 SEH，尽管大部分的函数都没有使用到 SEH。  
  
  以上我们主要讲述的是 x64 SEH 的内部实现。对于使用者，也有一个好消息，我们来看看，  
  
  C 代码：  
  
代码:

```
    VOID SehTest()
    {
        \_\_try
        {
            \_\_try
            {
                ExRaiseStatus(STATUS\_INVALID\_PARAMETER);
                DbgPrint("%u \[%s\] \_\_try \\n", \_\_LINE\_\_, \_\_FILE\_\_);
            }
            \_\_except((STATUS\_INVALID\_PARAMETER \== GetExceptionCode()) ? EXCEPTION\_CONTINUE\_SEARCH : EXCEPTION\_EXECUTE\_HANDLER)
            {
                DbgPrint("%u \[%s\] \_\_except \\n", \_\_LINE\_\_, \_\_FILE\_\_);
            }
        }
        \_\_finally
        
            DbgPrint("%u \[%s\] \_\_finally \\n", \_\_LINE\_\_, \_\_FILE\_\_);
        }
    }

```

  反汇编码：  
  
代码:

```
    kd\> uf passthrough!SehTest
    PassThrough!SehTest:
    fffffadf\`f1100020 sub     rsp,38h
    fffffadf\`f1100024 mov     ecx,0C000000Dh
    fffffadf\`f1100029 call    qword ptr \[PassThrough!\_imp\_ExRaiseStatus (fffffadf\`f1101050)\]
    fffffadf\`f110002f lea     r8,\[PassThrough! ?? ::FNODOBFM::\`string\` (fffffadf\`f1100500)\]
    fffffadf\`f1100036 mov     edx,39h
    fffffadf\`f110003b lea     rcx,\[PassThrough! ?? ::FNODOBFM::\`string\` (fffffadf\`f1100540)\]
    fffffadf\`f1100042 call    PassThrough!DbgPrint (fffffadf\`f11004a6)
    fffffadf\`f1100047 jmp     PassThrough!SehTest+0x42 (fffffadf\`f1100062)
    
    PassThrough!SehTest+0x42:
    fffffadf\`f1100062 lea     r8,\[PassThrough! ?? ::FNODOBFM::\`string\` (fffffadf\`f1100500)\]
    fffffadf\`f1100069 mov     edx,42h
    fffffadf\`f110006e lea     rcx,\[PassThrough! ?? ::FNODOBFM::\`string\` (fffffadf\`f1100570)\]
    fffffadf\`f1100075 call    PassThrough!DbgPrint (fffffadf\`f11004a6)
    fffffadf\`f110007a add     rsp,38h
    fffffadf\`f110007e ret

```

  我们发现 SehTest 内部完全没有任何 SEH 的踪迹，不像 x86 那样会有创建、销毁 EXCEPTION\_REGISTRATION\_RECORD 和调整 EXCEPTION\_REGISTRATION\_RECORD::trylevel 等操作。  
  这样的好处就是使用者无需再担心性能损耗，可以放心大胆的使用 SEH 机制了。  
  
  
**附录一 boxr 扩展**  
  
  为了方便自己分析，我写了一个简单的 windbg 扩展，提供了几个 x64 seh 常用功能：  
  
  !boxr.unwindinfo    module-name    unwindinfo\_addr  
  功能：  
    用于查询指定 UNWIND\_INFO 结构的详细信息。  
  参数说明：  
    module-name  待查询的 UNWIND\_INFO 结构对应函数的模块名  
    unwindinfo\_addr  UNWIND\_INFO 结构的绝对地址  
  
  !boxr.rtfn    option    module    runtimefunction\_addr  
  功能：  
    用于查询指定 RUNTIME\_FUNCTION 结构的详细信息。（rtfn 表示 RunTime\_FunctioN）  
  参数说明：  
    option  参数选项，目前支持两种：  
      /a 表示 module 参数为模块基地址  
      /n 表示 module 参数为模块名称  
    module  RUNTIME\_FUNCTION 结构对应函数所在的模块，具体形式根据 option 而定。  
    runtimefunction\_addr  需要查询的 RUNTIME\_FUNCTION 结构体的绝对地址。支持 @rax 操作方式，但不支持复杂的组合，比如 @rax+8。  
  
  使用的方法是：用 .extpath+ 命令将 boxr.dll 所在的目录添加到 windbg 的搜索路径中，然后就可以使用了。需要卸载时就 .unload。  
  
  简单说明一下这两个命令。  
  
  比如我们要查看下面这个函数的 UNWIND\_INFO 信息：  
代码:

```
    VOID SehTest()
    {
        \_\_try
        {
            DbgPrint("%u \[%s\] \_\_try \\n", \_\_LINE\_\_, \_\_FILE\_\_);
        }
        \_\_except(EXCEPTION\_EXECUTE\_HANDLER)
        {
            DbgPrint("%u \[%s\] \_\_except \\n", \_\_LINE\_\_, \_\_FILE\_\_);
        }
    
        \_\_try
        {
            \_\_try
            {
                \_\_try
                {
                    CollidedUnwind();
                }
                \_\_except(EXCEPTION\_EXECUTE\_HANDLER)
                {
                    DbgPrint("%u \[%s\] \_\_except \\n", \_\_LINE\_\_, \_\_FILE\_\_);
                }
            }
            \_\_except(EXCEPTION\_EXECUTE\_HANDLER)
            {
                DbgPrint("%u \[%s\] \_\_except \\n", \_\_LINE\_\_, \_\_FILE\_\_);
            }
        }
        \_\_finally
        {
            DbgPrint("%u \[%s\] \_\_finally \\n", \_\_LINE\_\_, \_\_FILE\_\_);
        }
    
        \_\_try
        {
            DbgPrint("%u \[%s\] \_\_try \\n", \_\_LINE\_\_, \_\_FILE\_\_);
    
            \_\_try
            {
                DbgPrint("%u \[%s\] \_\_try \\n", \_\_LINE\_\_, \_\_FILE\_\_);
            }
            \_\_finally
            {
                DbgPrint("%u \[%s\] \_\_finally \\n", \_\_LINE\_\_, \_\_FILE\_\_);
            }
        }
        \_\_except(EXCEPTION\_EXECUTE\_HANDLER)
        {
            DbgPrint("%u \[%s\] \_\_except \\n", \_\_LINE\_\_, \_\_FILE\_\_);
        }
    
        return;
    }

```

  
  操作步骤：  
  1. 使用 .fnent 命令获得 SehTest 的基本信息，  
    kd> .fnent passthrough!SehTest  
    Debugger function entry 00000000\`00758210 for:  
    (fffffadf\`f135d080)   PassThrough!SehTest   |  (fffffadf\`f135d180)   PassThrough!LeafTest  
    Exact matches:  
        PassThrough!SehTest (void)  
  
    BeginAddress      = 00000000\`00001080  
    EndAddress        = 00000000\`00001175  
    UnwindInfoAddress = 00000000\`000026b8  
  
    Unwind info at fffffadf\`f135e6b8, 10 bytes  
      version 1, flags 3, prolog 4, codes 1  
      handler routine: PassThrough!\_C\_specific\_handler (fffffadf\`f135d5de), data 6  
      00: offs 4, unwind op 2, op info 4  UWOP\_ALLOC\_SMALL.  
  2. 使用 !boxr.unwindinfo 查询详细信息，  
    kd> !boxr.unwindinfo passthrough fffffadf\`f135e6b8  
    \_UNWIND\_INFO for fffffadff135e6b8   
    Flags:  
        EU  
    ExceptionRoutine:  
        PassThrough!\_C\_specific\_handler (fffffadf\`f135d5de)  
    ScopeTable:  
        Count: 6  
        ScopeRecord\[0\]      (fffffadff135e6c8)  
            BeginAddress:   
                PassThrough!SehTest+0x4 (fffffadf\`f135d084)  
            EndAddress:   
                PassThrough!SehTest+0x1e (fffffadf\`f135d09e)  
            HandlerAddress:   
                PassThrough!SehTest$filt$0 (fffffadf\`f135d8a0)  
            JumpTarget:   
                PassThrough!SehTest+0x1e (fffffadf\`f135d09e)  
        \[省略中间 3 个 ScopeRecord 成员\]  
        ScopeRecord\[5\]      (fffffadff135e718)  
            BeginAddress:   
                PassThrough!SehTest+0x8b (fffffadf\`f135d10b)  
            EndAddress:   
                PassThrough!SehTest+0xd7 (fffffadf\`f135d157)  
            HandlerAddress:   
                PassThrough!SehTest$filt$5 (fffffadf\`f135d960)  
            JumpTarget:   
                PassThrough!SehTest+0xd7 (fffffadf\`f135d157)  
  
  !boxr.rtfn 的用法也类似，比如：  
    kd> !box.rtfn /n passThrough @rax  
    \_RUNTIME\_FUNCTION for fffffadff1113000   
    BeginAddress:  
        PassThrough!CollidedUnwind (fffffadf\`f1110020)  
    EndAddress:  
        PassThrough!CollidedUnwind+0x38 (fffffadf\`f1110058)  
    UnwindData:  
        fffffadff1111688  
    \_UNWIND\_INFO for fffffadff1111688   
    Flags:  
        U  
    ExceptionRoutine:  
        PassThrough!\_C\_specific\_handler (fffffadf\`f11104ee)  
    ScopeTable:  
        Count: 1  
        ScopeRecord\[0\]      (fffffadff1111698)  
            BeginAddress:   
                PassThrough!CollidedUnwind+0x4 (fffffadf\`f1110024)  
            EndAddress:   
                PassThrough!CollidedUnwind+0x10 (fffffadf\`f1110030)  
            HandlerAddress:   
                PassThrough!CollidedUnwind$fin$0 (fffffadf\`f1110750)  
            JumpTarget:   
                0    
  
  需要说明的是，我写这个扩展的目的仅仅是为了分析 x64 SEH 过程中能轻松的查看相关数据结构的详细信息，所以并没有在这个扩展上花很多时间。其代码是从 MS 例子代码的基础上增加了我需要的功能。应该有一些 BUG，但是对我来说不重要，已经满足我的需要了。源码也放在附件里，方便分析的朋友根据自己的需要进行修改。  
  另外有一个疑问：我编译的 x64 wrk1.2 内核无法对 .c 代码进行源码调试，对 .asm 代码倒是可以，这是为什么？我看了一下编译选项，没看出什么猫腻。有经验的朋友分享一下吧，感谢 :-)  
  
  
**附录二 RtlUnwindEx 的反汇编代码和注释**  
  
  由于无法源码调试，只好把它反汇编出来加上注释……  
  
代码:

```
             VOID
             RtlUnwindEx (
/\* rcx    \*/     IN PVOID pEstablisherFrame OPTIONAL,
/\* rdx    \*/     IN PVOID pJumpTargetIp OPTIONAL,
/\* r8     \*/     IN PEXCEPTION\_RECORD pExceptionRecord OPTIONAL,
/\* r9     \*/     IN PVOID ReturnValue,
/\* rsp+28 \*/     IN PCONTEXT pOriginalContext,
/\* rsp+30 \*/     IN PUNWIND\_HISTORY\_TABLE pHistoryTable OPTIONAL
                 );

             kd\> uf nt!RtlUnwindEx
             nt!RtlUnwindEx:
             fffff800\`00891e70 mov     qword ptr \[rsp+20h\],r9
             fffff800\`00891e75 mov     qword ptr \[rsp+18h\],r8
             fffff800\`00891e7a mov     qword ptr \[rsp+10h\],rdx
             fffff800\`00891e7f mov     rax,rsp
             fffff800\`00891e82 sub     rsp,678h
             fffff800\`00891e89 mov     qword ptr \[rax\-8\],rbx
             fffff800\`00891e8d mov     qword ptr \[rax\-10h\],rbp
             fffff800\`00891e91 mov     qword ptr \[rax\-18h\],rsi
             fffff800\`00891e95 mov     rsi,qword ptr \[rsp+6A0h\] ; rsi = pOriginalContext
             fffff800\`00891e9d mov     qword ptr \[rax\-20h\],rdi
             fffff800\`00891ea1 mov     qword ptr \[rax\-28h\],r12
             fffff800\`00891ea5 mov     qword ptr \[rax\-40h\],r15
             fffff800\`00891ea9 mov     rbp,rcx
             fffff800\`00891eac mov     r15,rdx
             fffff800\`00891eaf lea     rdx,\[rsp+40h\] ; rsp+40 为 l\_HighLimit
             fffff800\`00891eb4 lea     rcx,\[rsp+50h\] ; rsp+50 为 l\_LowLimit
             fffff800\`00891eb9 mov     rbx,r8
             fffff800\`00891ebc mov     rdi,rsi
             fffff800\`00891ebf lea     r12,\[rax\-518h\] ; r12 = &l\_Context
             fffff800\`00891ec6 call    nt!RtlpGetStackLimits (fffff800\`00890da0)
             fffff800\`00891ecb mov     rcx,rsi
             fffff800\`00891ece call    nt!RtlCaptureContext (fffff800\`008bd150)
             fffff800\`00891ed3 mov     rax,qword ptr \[rsp+6A8h\] ; rax = pHistoryTable
             fffff800\`00891edb test    rax,rax
<            fffff800\`00891ede je      nt!RtlUnwindEx+0x74 (fffff800\`00891ee4)
.            
.            nt!RtlUnwindEx+0x70:
.            fffff800\`00891ee0 mov     byte ptr \[rax+4\],1 ; pHistoryTable->Search = UNWIND\_HISTORY\_TABLE\_GLOBAL
.            
.            nt!RtlUnwindEx+0x74:
\>            fffff800\`00891ee4 xor     ecx,ecx
             fffff800\`00891ee6 test    rbx,rbx ; 判断 pExceptionRecord 是否为 NULL
<            fffff800\`00891ee9 jne     nt!RtlUnwindEx+0xbc (fffff800\`00891f2c)
.            
.            nt!RtlUnwindEx+0x7b:
.            ; pExceptionRecord 等于 NULL
.            fffff800\`00891eeb mov     rax,qword ptr \[rsi+0F8h\]  ; rax = pOriginalContext->Rip
.            fffff800\`00891ef2 lea     rbx,\[rsp+0C0h\]            ; rbx = &l\_ExceptionRecord
.            fffff800\`00891efa mov     dword ptr \[rsp+0C0h\],0C0000027h ; l\_ExceptionRecord.ExceptionCode = STATUS\_UNWIND (0xC0000027)
.            fffff800\`00891f05 mov     qword ptr \[rsp+0D0h\],rax  ; l\_ExceptionRecord.ExceptionAddress = pOriginalContext->Rip
.            fffff800\`00891f0d mov     rax,qword ptr \[rsp+6A8h\]  ; rax = pHistoryTable
.            fffff800\`00891f15 mov     qword ptr \[rsp+690h\],rbx  ; \[r8-home\] = &l\_ExceptionRecord ????
.            fffff800\`00891f1d mov     qword ptr \[rsp+0C8h\],rcx  ; l\_ExceptionRecord.ExceptionRecord = NULL
.            fffff800\`00891f25 mov     dword ptr \[rsp+0D8h\],ecx  ; l\_ExceptionRecord.NumberParameters = 0
.            
.            nt!RtlUnwindEx+0xbc:
\>            fffff800\`00891f2c mov     rbx,qword ptr \[rsp+680h\]  ; rbx = &\[rcx-home\]
             fffff800\`00891f34 mov     esi,2 ; l\_ExceptionFlags(esi) = EXCEPTION\_UNWINDING (2)
             fffff800\`00891f39 mov     ecx,6 ; ecx = EXCEPTION\_EXIT\_UNWIND (6)
             fffff800\`00891f3e test    rbp,rbp ; 判断 pEstablisherFrame 是否为 NULL
             fffff800\`00891f41 mov     qword ptr \[rsp+648h\],r13 ; 保存 r13 
             fffff800\`00891f49 mov     qword ptr \[rsp+640h\],r14 ; 保存 r14
             fffff800\`00891f51 cmove   esi,ecx ; if (NULL == pEstablisherFrame) { l\_ExceptionFlags = EXCEPTION\_EXIT\_UNWIND (6) }
             fffff800\`00891f54 xchg    ax,ax
             fffff800\`00891f58 xchg    ax,ax
             fffff800\`00891f5c xchg    ax,ax
             
             nt!RtlUnwindEx+0xf0:
           > fffff800\`00891f60 mov     r13,qword ptr \[rdi+0F8h\] ; r13 = pOriginalContext->Rip
           . fffff800\`00891f67 lea     rdx,\[rsp+60h\] ; rdx = &l\_pImageBase
           . fffff800\`00891f6c mov     r8,rax        ; r8 = pHistoryTable
           . fffff800\`00891f6f mov     rcx,r13
           . fffff800\`00891f72 mov     qword ptr \[rsp+68h\],r13
           . fffff800\`00891f77 call    nt!RtlLookupFunctionEntry (fffff800\`00890e60)
           .                   ; l\_pFunctionEntry = RtlLookupFunctionEntry(pOriginalContext->Rip, 
           .                   ;                                           &l\_pImageBase,
           .                   ;                                           pHistoryTable)
           . fffff800\`00891f7c test    rax,rax ; 判断 l\_pFunctionEntry (eax) 是否为 NULL
           . fffff800\`00891f7f mov     r14,rax ; r14 = l\_pFunctionEntry
<          . fffff800\`00891f82 je      nt!RtlUnwindEx+0x3ab (fffff800\`0089221b)
.          . 
.          . nt!RtlUnwindEx+0x118:
.          . fffff800\`00891f88 mov     rdx,rdi ; rdx = pOriginalContext
.          . fffff800\`00891f8b mov     rcx,r12 ; rcx = &l\_Context
.          . fffff800\`00891f8e call    nt!RtlpCopyContext (fffff800\`00891080)
.          .                   ; RtlpCopyContext(&l\_Context, pOriginalContext)
.          . fffff800\`00891f93 mov     rdx,qword ptr \[rsp+60h\] ; rdx = l\_pImageBase
.          . fffff800\`00891f98 mov     qword ptr \[rsp+38h\],0   ; \_ARG\_8 = 0
.          . fffff800\`00891fa1 lea     rax,\[rsp+680h\]          ; rax = &\[rcx-home\]，这里被用作局部变量 l\_pEstablisherFrame 空间
.          . fffff800\`00891fa9 mov     r9,r14                  ; r9 = l\_pFunctionEntry
.          . fffff800\`00891fac mov     r8,r13                  ; r8 = pOriginalContext->Rip
.          . fffff800\`00891faf mov     qword ptr \[rsp+30h\],rax ; \_ARG7 = &l\_pEstablisherFrame
.          . fffff800\`00891fb4 lea     rax,\[rsp+58h\]           ; rax = &l\_pHandlerData
.          . fffff800\`00891fb9 mov     ecx,2                   ; ecx = UNW\_FLAG\_UHANDLER (2)
.          . fffff800\`00891fbe mov     qword ptr \[rsp+28h\],rax ; \_ARG\_6 = &l\_pHandlerData
.          . fffff800\`00891fc3 mov     qword ptr \[rsp+20h\],r12 ; \_ARG\_5 = &l\_Context
.          . fffff800\`00891fc8 call    nt!RtlVirtualUnwind (fffff800\`00891380)
.          .                   ; l\_pExceptionRoutine = RtlVirtualUnwind(UNW\_FLAG\_UHANDLER,
.          .                   ;                                        l\_pImageBase,
.          .                   ;                                        pOriginalContext->Rip,
.          .                   ;                                        l\_pFunctionEntry,
.          .                   ;                                        &l\_Context,
.          .                   ;                                        &l\_pHandlerData,
.          .                   ;                                        &l\_pEstablisherFrame,
.          .                   ;                                        NULL);
.          . fffff800\`00891fcd mov     rbx,qword ptr \[rsp+680h\] ; rbx = l\_pEstablisherFrame
.          . fffff800\`00891fd5 mov     rcx,rax                  ; rcx = l\_pExceptionRoutine
.          . fffff800\`00891fd8 mov     qword ptr \[rsp+48h\],rax
.          . fffff800\`00891fdd test    bl,7                     ; 检查 l\_pEstablisherFrame 是否对齐
.<         . fffff800\`00891fe0 jne     nt!RtlUnwindEx+0x431 (fffff800\`008922a1)
..         . 
..         . nt!RtlUnwindEx+0x176:
..         . fffff800\`00891fe6 cmp     rbx,qword ptr \[rsp+50h\]  ; cmp l\_pEstablisherFrame, l\_LowLimit
..<        . fffff800\`00891feb jb      nt!RtlUnwindEx+0x184 (fffff800\`00891ff4)
...        . 
...        . nt!RtlUnwindEx+0x17d:
...        . fffff800\`00891fed cmp     rbx,qword ptr \[rsp+40h\]  ; cmp l\_pEstablisherFrame, l\_HighLimit
...<       . fffff800\`00891ff2 jb      nt!RtlUnwindEx+0x1d3 (fffff800\`00892043)
....       . 
....       . nt!RtlUnwindEx+0x184:
....       . ; 检查 l\_pEstablisherFrame 是否合法
..\>.       . fffff800\`00891ff4 mov     cl,byte ptr gs:\[20DEh\]   ; cl = \_KPCR->DpcRoutineActive
.. .       . fffff800\`00891ffc test    cl,cl                    ; 判断当前是否在执行 DPC
.< .       . fffff800\`00891ffe jne     nt!RtlUnwindEx+0x431 (fffff800\`008922a1) ; 如果是在执行 DPC 则失败？??
.. .       . 
.. .       . nt!RtlUnwindEx+0x194:
.. .       . fffff800\`00892004 mov     rcx,qword ptr \[rsp+40h\] ; rcx = l\_HighLimit
.. .       . fffff800\`00892009 mov     rax,qword ptr \[rcx\-28h\] ; rax = l\_KernelStackCtrl->Previous.StackBase
.. .       . fffff800\`0089200d test    rax,rax                 ; 判断 l\_KernelStackCtrl->Previous.StackBase 是否为 NULL
.< .       . fffff800\`00892010 je      nt!RtlUnwindEx+0x431 (fffff800\`008922a1)
.. .       . 
.. .       . nt!RtlUnwindEx+0x1a6:
.. .       . fffff800\`00892016 mov     rdx,qword ptr \[rcx\-20h\]  ; rdx = l\_KernelStackCtrl->Previous.StackLimit
.. .       . fffff800\`0089201a mov     rbx,qword ptr \[rsp+680h\] ; rbx = l\_pEstablisherFrame
.. .       . fffff800\`00892022 cmp     rbx,rdx                  ; cmp l\_pEstablisherFrame, l\_KernelStackCtrl->Previous.StackLimit
.< .       . fffff800\`00892025 jb      nt!RtlUnwindEx+0x431 (fffff800\`008922a1)
.. .       . 
.. .       . nt!RtlUnwindEx+0x1bb:
.. .       . fffff800\`0089202b cmp     rbx,rax ; cmp l\_pEstablisherFrame, l\_KernelStackCtrl->Previous.StackBase
.< .       . fffff800\`0089202e jae     nt!RtlUnwindEx+0x431 (fffff800\`008922a1)
.. .       . 
.. .       . nt!RtlUnwindEx+0x1c4:
.. .       . fffff800\`00892034 mov     rcx,qword ptr \[rsp+48h\] ; rcx = l\_pExceptionRoutine
.. .       . fffff800\`00892039 mov     qword ptr \[rsp+50h\],rdx ; l\_LowLimit = l\_KernelStackCtrl->Previous.StackLimit
.. .       . fffff800\`0089203e mov     qword ptr \[rsp+40h\],rax ; l\_HighLimit = l\_KernelStackCtrl->Previous.StackBase
.. .       . 
.. .       . nt!RtlUnwindEx+0x1d3:
.. \>       . fffff800\`00892043 test    rbp,rbp ; 判断 pEstablisherFrame 是否为 NULL
.. <       . fffff800\`00892046 je      nt!RtlUnwindEx+0x1e1 (fffff800\`00892051)
.. .       . 
.. .       . nt!RtlUnwindEx+0x1d8:
.. .       . fffff800\`00892048 cmp     rbp,rbx ; cmp pEstablisherFrame, l\_pEstablisherFrame
.< .       . fffff800\`0089204b jb      nt!RtlUnwindEx+0x431 (fffff800\`008922a1)
.. .       . 
.. .       . nt!RtlUnwindEx+0x1e1:
.. \>       . fffff800\`00892051 test    rcx,rcx ; 判断 l\_pExceptionRoutine 是否为 NULL
..  <      . fffff800\`00892054 je      nt!RtlUnwindEx+0x39b (fffff800\`0089220b)
..  .      . 
..  .      . nt!RtlUnwindEx+0x1ea:
..  .      . fffff800\`0089205a mov     r13,qword ptr \[rsp+58h\] ; r13 = l\_pHandlerData
..  .      . fffff800\`0089205f mov     qword ptr \[rsp+90h\],r15 ; \[rsp+90\] = pJumpTargetIp
..  .      . fffff800\`00892067 xor     r15d,r15d
..  .      . fffff800\`0089206a xchg    ax,ax
..  .      . fffff800\`0089206d xchg    ax,ax
..  .      . 
..  .      . nt!RtlUnwindEx+0x200:
..  .    \> . fffff800\`00892070 cmp     rbp,rbx ; cmp pEstablisherFrame, l\_pEstablisherFrame
..  .<   . . fffff800\`00892073 jne     nt!RtlUnwindEx+0x208 (fffff800\`00892078)
..  ..   . . 
..  ..   . . nt!RtlUnwindEx+0x205:
..  ..   . . fffff800\`00892075 or      esi,20h ; l\_ExceptionFlags |= EXCEPTION\_TARGET\_UNWIND (0x20)
..  ..   . . 
..  ..   . . nt!RtlUnwindEx+0x208:
..  .\>   . . fffff800\`00892078 mov     r10,qword ptr \[rsp+690h\] ; r10 = &l\_ExceptionRecord
..  .    . . fffff800\`00892080 mov     rax,qword ptr \[rsp+698h\] ; rax = ReturnValue
..  .    . . fffff800\`00892088 mov     qword ptr \[rsp+0A0h\],rcx ; l\_DispatcherContext.LanguageHandler = l\_pExceptionRoutine
..  .    . . fffff800\`00892090 mov     dword ptr \[r10+4\],esi    ; l\_ExceptionRecord.ExceptionFlags = l\_ExceptionFlags
..  .    . . fffff800\`00892094 mov     qword ptr \[rdi+78h\],rax  ; pOriginalContext->Rax = ReturnValue
..  .    . . fffff800\`00892098 mov     rax,qword ptr \[rsp+68h\]  ; rax = pOriginalContext->Rip
..  .    . . fffff800\`0089209d mov     qword ptr \[rsp+70h\],rax  ; l\_DispatcherContext.ControlPc = pOriginalContext->Rip
..  .    . . fffff800\`008920a2 mov     rax,qword ptr \[rsp+60h\]  ; rax = l\_pImageBase
..  .    . . fffff800\`008920a7 lea     r9,\[rsp+70h\]             ; r9 = &l\_DispatcherContext
..  .    . . fffff800\`008920ac mov     qword ptr \[rsp+78h\],rax  ; l\_DispatcherContext.ImageBase = l\_pImageBase
..  .    . . fffff800\`008920b1 mov     rax,qword ptr \[rsp+6A8h\] ; rax = pHistoryTable
..  .    . . fffff800\`008920b9 mov     r8,rdi                   ; r8 = pOriginalContext
..  .    . . fffff800\`008920bc mov     rdx,rbx                  ; rdx = l\_pEstablisherFrame
..  .    . . fffff800\`008920bf mov     rcx,r10                  ; rcx = &l\_ExceptionRecord
..  .    . . fffff800\`008920c2 mov     qword ptr \[rsp+80h\],r14   ; l\_DispatcherContext.FunctionEntry = l\_pFunctionEntry
..  .    . . fffff800\`008920ca mov     qword ptr \[rsp+0B0h\],rax  ; l\_DispatcherContext.HistoryTable = pHistoryTable
..  .    . . fffff800\`008920d2 mov     qword ptr \[rsp+88h\],rbx   ; l\_DispatcherContext.EstablisherFrame = l\_pEstablisherFrame
..  .    . . fffff800\`008920da mov     qword ptr \[rsp+98h\],rdi   ; l\_DispatcherContext.ContextRecord = pOriginalContext
..  .    . . fffff800\`008920e2 mov     qword ptr \[rsp+0A8h\],r13  ; l\_DispatcherContext.HandlerData = l\_pHandlerData
..  .    . . fffff800\`008920ea mov     dword ptr \[rsp+0B8h\],r15d ; l\_DispatcherContext.ScopeIndex = 0
..  .    . . fffff800\`008920f2 and     esi,0FFFFFF9Fh            ; l\_ExceptionFlags &= ~(EXCEPTION\_TARGET\_UNWIND|EXCEPTION\_COLLIDED\_UNWIND)
..  .    . . fffff800\`008920f5 call    nt!RtlpExecuteHandlerForUnwind (fffff800\`008bd9d0)
..  .    . .                   ; RtlpExecuteHandlerForUnwind(&l\_ExceptionRecord,
..  .    . .                   ;                             l\_pEstablisherFrame,
..  .    . .                   ;                             pOriginalContext,
..  .    . .                   ;                             &l\_DispatcherContext);
..  .    . . fffff800\`008920fa dec     eax
..  .<   . . fffff800\`008920fc je      nt!RtlUnwindEx+0x368 (fffff800\`008921d8) ; 如果返回 ExceptionContinueSearch 则跳转
..  ..   . . 
..  ..   . . nt!RtlUnwindEx+0x292:
..  ..   . . fffff800\`00892102 cmp     eax,2 ; cmp eax, ExceptionCollidedUnwind (3 - 1)
..  ..<  . . fffff800\`00892105 jne     nt!RtlUnwindEx+0x426 (fffff800\`00892296)
..  ...  . . 
..  ...  . . nt!RtlUnwindEx+0x29b:
..  ...  . . ; ExceptionCollidedUnwind 的情况
..  ...  . . fffff800\`0089210b mov     r8,qword ptr \[rsp+70h\]   ; r8 = l\_DispatcherContext.ControlPc
..  ...  . . fffff800\`00892110 mov     r10,qword ptr \[rsp+78h\]  ; r10 = l\_DispatcherContext.ImageBase
..  ...  . . fffff800\`00892115 mov     rdx,qword ptr \[rsp+98h\]  ; rdx = l\_DispatcherContext.ContextRecord
..  ...  . . fffff800\`0089211d mov     rcx,qword ptr \[rsp+6A0h\] ; rcx = pOriginalContext
..  ...  . . fffff800\`00892125 mov     r14,qword ptr \[rsp+80h\]  ; r14 = l\_DispatcherContext.FunctionEntry
..  ...  . . fffff800\`0089212d mov     qword ptr \[rsp+68h\],r8
..  ...  . . fffff800\`00892132 mov     qword ptr \[rsp+60h\],r10
..  ...  . . fffff800\`00892137 call    nt!RtlpCopyContext (fffff800\`00891080)
..  ...  . . ;                 RtlpCopyContext(pOriginalContext, l\_DispatcherContext.ContextRecord);
..  ...  . . fffff800\`0089213c mov     rdx,rcx        ; rdx = pOriginalContext
..  ...  . . fffff800\`0089213f mov     rdi,rcx        ; rdi = pOriginalContext
..  ...  . . fffff800\`00892142 lea     rcx,\[rsp+160h\] ; rcx = &l\_Context
..  ...  . . fffff800\`0089214a lea     r12,\[rsp+160h\] ; r12 = &l\_Context
..  ...  . . fffff800\`00892152 call    nt!RtlpCopyContext (fffff800\`00891080)
..  ...  . . ;                 RtlpCopyContext(&l\_Context, pOriginalContext);
..  ...  . . fffff800\`00892157 mov     qword ptr \[rsp+38h\],0   ; \_ARG\_8 = NULL
..  ...  . . fffff800\`00892160 lea     rax,\[rsp+680h\]          ; rax = &l\_pEstablisherFrame
..  ...  . . fffff800\`00892168 mov     qword ptr \[rsp+30h\],rax ; \_ARG\_7 = &l\_pEstablisherFrame
..  ...  . . fffff800\`0089216d lea     rax,\[rsp+58h\]           ; rax = &l\_pHandlerData
..  ...  . . fffff800\`00892172 mov     r9,r14                  ; r9 = l\_DispatcherContext.FunctionEntry
..  ...  . . fffff800\`00892175 mov     qword ptr \[rsp+28h\],rax ; \_ARG\_6 = &l\_pHandlerData
..  ...  . . fffff800\`0089217a lea     rax,\[rsp+160h\]          ; rax = &l\_Context
..  ...  . . fffff800\`00892182 mov     rdx,r10                 ; rdx = l\_DispatcherContext.ImageBase
..  ...  . . fffff800\`00892185 xor     ecx,ecx                 ; ecx = UNW\_FLAG\_NHANDLER (0)
..  ...  . . fffff800\`00892187 mov     qword ptr \[rsp+20h\],rax ; \_ARG\_5 = &l\_Context
..  ...  . . fffff800\`0089218c call    nt!RtlVirtualUnwind (fffff800\`00891380)
..  ...  . .                   ; RtlVirtualUnwind(UNW\_FLAG\_NHANDLER,
..  ...  . .                   ;                  l\_DispatcherContext.ImageBase,
..  ...  . .                   ;                  l\_DispatcherContext.ControlPc,
..  ...  . .                   ;                  l\_DispatcherContext.FunctionEntry,
..  ...  . .                   ;                  &l\_Context,
..  ...  . .                   ;                  &l\_pHandlerData,
..  ...  . .                   ;                  &l\_pEstablisherFrame,
..  ...  . .                   ;                  NULL);
..  ...  . . fffff800\`00892191 mov     rbx,qword ptr \[rsp+88h\]   ; rbx = l\_DispatcherContext.EstablisherFrame
..  ...  . . fffff800\`00892199 mov     rcx,qword ptr \[rsp+0A0h\]  ; rcx = l\_DispatcherContext.LanguageHandler
..  ...  . . fffff800\`008921a1 mov     r13,qword ptr \[rsp+0A8h\]  ; r13 = l\_DispatcherContext.HandlerData
..  ...  . . fffff800\`008921a9 mov     rax,qword ptr \[rsp+0B0h\]  ; rax = l\_DispatcherContext.HistoryTable
..  ...  . . fffff800\`008921b1 mov     r15d,dword ptr \[rsp+0B8h\] ; r15d = l\_DispatcherContext.ScopeIndex
..  ...  . . fffff800\`008921b9 mov     qword ptr \[rsp+680h\],rbx  ; l\_pEstablisherFrame = l\_DispatcherContext.EstablisherFrame
..  ...  . . fffff800\`008921c1 mov     qword ptr \[rsp+48h\],rcx   ; l\_pExceptionRoutine = l\_DispatcherContext.LanguageHandler
..  ...  . . fffff800\`008921c6 mov     qword ptr \[rsp+58h\],r13   ; l\_pHandlerData = l\_DispatcherContext.HandlerData
..  ...  . . fffff800\`008921cb mov     qword ptr \[rsp+6A8h\],rax  ; pHistoryTable = l\_DispatcherContext.HistoryTable
..  ...  . . fffff800\`008921d3 or      esi,40h                   ; l\_ExceptionFlags |= EXCEPTION\_COLLIDED\_UNWIND (40)
..  ...< . . fffff800\`008921d6 jmp     nt!RtlUnwindEx+0x382 (fffff800\`008921f2)
..  .... . . 
..  .... . . nt!RtlUnwindEx+0x368:
..  .\>.. . . fffff800\`008921d8 cmp     rbx,rbp ; cmp l\_DispatcherContext.EstablisherFrame, pEstablisherFrame
..  . ..<. . fffff800\`008921db je      nt!RtlUnwindEx+0x37d (fffff800\`008921ed)
..  . .... . 
..  . .... . nt!RtlUnwindEx+0x36d:
..  . .... . fffff800\`008921dd mov     rcx,qword ptr \[rsp+48h\] ; rcx = l\_pExceptionRoutine
..  . .... . fffff800\`008921e2 mov     rax,rdi                 ; rax = pOriginalContext
..  . .... . fffff800\`008921e5 mov     rdi,r12                 ; rdi = &l\_Context
..  . .... . fffff800\`008921e8 mov     r12,rax                 ; r12 = pOriginalContext
..  . .v.. . fffff800\`008921eb jmp     nt!RtlUnwindEx+0x382 (fffff800\`008921f2)
..  . .... . 
..  . .... . nt!RtlUnwindEx+0x37d:
..  . ..\>. . fffff800\`008921ed mov     rcx,qword ptr \[rsp+48h\] ; rcx = l\_pExceptionRoutine
..  . .. . . 
..  . .. . . nt!RtlUnwindEx+0x382:
..  . .\> . . fffff800\`008921f2 test    sil,40h  ; test sil, EXCEPTION\_COLLIDED\_UNWIND (40)
..  . .  < . fffff800\`008921f6 jne     nt!RtlUnwindEx+0x200 (fffff800\`00892070)
..  . .    . 
..  . .    . nt!RtlUnwindEx+0x38c:
..  . .    . fffff800\`008921fc mov     r13,qword ptr \[rsp+68h\]  ; r13 = pOriginalContext->Rip
..  . .    . fffff800\`00892201 mov     r15,qword ptr \[rsp+688h\] ; r15 = pJumpTargetIp
..  . .<   . fffff800\`00892209 jmp     nt!RtlUnwindEx+0x3c7 (fffff800\`00892237)
..  . ..   . 
..  . ..   . nt!RtlUnwindEx+0x39b:
..  \> ..   . fffff800\`0089220b cmp     rbx,rbp  ; cmp l\_DispatcherContext.EstablisherFrame, pEstablisherFrame
..    .<   . fffff800\`0089220e je      nt!RtlUnwindEx+0x3c7 (fffff800\`00892237)
..    ..   . 
..    ..   . nt!RtlUnwindEx+0x3a0:
..    ..   . fffff800\`00892210 mov     rax,rdi ; rax = &l\_Context
..    ..   . fffff800\`00892213 mov     rdi,r12 ; rdi = pOriginalContext
..    ..   . fffff800\`00892216 mov     r12,rax ; r12 = &l\_Context
..    .<   . fffff800\`00892219 jmp     nt!RtlUnwindEx+0x3c7 (fffff800\`00892237)
..    ..   . 
..    ..   . nt!RtlUnwindEx+0x3ab:
\>.    ..   . fffff800\`0089221b mov     rcx,qword ptr \[rdi+98h\]  ; rcx = pOriginalContext->Rsp
 .    ..   . fffff800\`00892222 mov     rax,qword ptr \[rcx\]      ; rax = pOriginalContext->Rsp\[0\]
 .    ..   . fffff800\`00892225 mov     qword ptr \[rdi+0F8h\],rax ; pOriginalContext->Rip = pOriginalContext->Rsp\[0\]
 .    ..   . fffff800\`0089222c lea     rax,\[rcx+8\]              ; rax = pOriginalContext->Rsp + 8
 .    ..   . fffff800\`00892230 mov     qword ptr \[rdi+98h\],rax  ; pOriginalContext->Rsp = pOriginalContext->Rsp + 8 （跳过返回值）
 .    ..   .
 .    ..   . nt!RtlUnwindEx+0x3c7:
 .    .\>   . fffff800\`00892237 test    bl,7 ; 检查 l\_DispatcherContext.EstablisherFrame 是否对齐
 .    .<   . fffff800\`0089223a jne     nt!RtlUnwindEx+0x444 (fffff800\`008922b4)
 .    ..   . 
 .    ..   . nt!RtlUnwindEx+0x3cc:
 .    ..   . fffff800\`0089223c cmp     rbx,qword ptr \[rsp+50h\] ; cmp l\_DispatcherContext.EstablisherFrame, l\_LowLimit
 .    ..<  . fffff800\`00892241 jb      nt!RtlUnwindEx+0x3da (fffff800\`0089224a)
 .    ...  . 
 .    ...  . nt!RtlUnwindEx+0x3d3:
 .    ...  . fffff800\`00892243 cmp     rbx,qword ptr \[rsp+40h\] ; cmp l\_DispatcherContext.EstablisherFrame, l\_HighLimit
 .    ...< . fffff800\`00892248 jb      nt!RtlUnwindEx+0x414 (fffff800\`00892284)
 .    .... . 
 .    .... . nt!RtlUnwindEx+0x3da:
 .    ..\>. . fffff800\`0089224a mov     al,byte ptr gs:\[20DEh\] ; al = \_KPCR->DpcRoutineActive
 .    .. . . fffff800\`00892252 test    al,al
 .    .. .<. fffff800\`00892254 jne     nt!RtlUnwindEx+0x43c (fffff800\`008922ac)
 .    .. ... 
 .    .. ... nt!RtlUnwindEx+0x3e6:
 .    .. ... fffff800\`00892256 mov     rcx,qword ptr \[rsp+40h\] ; rcx = l\_HighLimit
 .    .. ... fffff800\`0089225b mov     rax,qword ptr \[rcx\-28h\] ; rax = l\_KernelStackCtrl->Previous.StackBase
 .    .. ... fffff800\`0089225f test    rax,rax
 .    .. .<. fffff800\`00892262 je      nt!RtlUnwindEx+0x43c (fffff800\`008922ac)
 .    .. ... 
 .    .. ... nt!RtlUnwindEx+0x3f4:
 .    .. ... fffff800\`00892264 mov     rdx,qword ptr \[rcx\-20h\]  ; rdx = l\_KernelStackCtrl->Previous.StackLimit
 .    .. ... fffff800\`00892268 mov     rbx,qword ptr \[rsp+680h\] ; rbx = l\_pEstablisherFrame
 .    .. ... fffff800\`00892270 cmp     rbx,rdx                  ; cmp l\_pEstablisherFrame, l\_KernelStackCtrl->Previous.StackLimit
 .    .< ... fffff800\`00892273 jb      nt!RtlUnwindEx+0x444 (fffff800\`008922b4)
 .    .. ... 
 .    .. ... nt!RtlUnwindEx+0x405:
 .    .. ... fffff800\`00892275 cmp     rbx,rax ; cmp l\_pEstablisherFrame, l\_KernelStackCtrl->Previous.StackBase
 .    .< ... fffff800\`00892278 jae     nt!RtlUnwindEx+0x444 (fffff800\`008922b4)
 .    .. ... 
 .    .. ... nt!RtlUnwindEx+0x40a:
 .    .. ... fffff800\`0089227a mov     qword ptr \[rsp+50h\],rdx ; l\_LowLimit = l\_KernelStackCtrl->Previous.StackLimit
 .    .. ... fffff800\`0089227f mov     qword ptr \[rsp+40h\],rax ; l\_HighLimit = l\_KernelStackCtrl->Previous.StackBase
 .    .. ... 
 .    .. ... nt!RtlUnwindEx+0x414:
 .    .. \>.. fffff800\`00892284 cmp     rbx,rbp ; cmp l\_pEstablisherFrame, pEstablisherFrame
 .    .. <.. fffff800\`00892287 je      nt!RtlUnwindEx+0x449 (fffff800\`008922b9)
 .    .. ... 
 .    .. ... nt!RtlUnwindEx+0x419:
 .    .. ... fffff800\`00892289 mov     rax,qword ptr \[rsp+6A8h\] ; rax = pHistoryTable
 .    .. ..^ fffff800\`00892291 jmp     nt!RtlUnwindEx+0xf0 (fffff800\`00891f60)
 .    .. ..  
 .    .. ..  nt!RtlUnwindEx+0x426:
 .    \>. ..  fffff800\`00892296 mov     ecx,0C0000026h ; ecx = STATUS\_INVALID\_DISPOSITION
 .     . ..  fffff800\`0089229b call    nt!RtlRaiseStatus (fffff800\`00889df0)
 .     . ..  fffff800\`008922a0 int     3
 .     . ..  
 .     . ..  nt!RtlUnwindEx+0x431:
 >     . ..  fffff800\`008922a1 mov     ecx,0C0000028h ; ecx = STATUS\_BAD\_STACK
       . ..  fffff800\`008922a6 call    nt!RtlRaiseStatus (fffff800\`00889df0)
       . ..  fffff800\`008922ab int     3
       . ..  
       . ..  nt!RtlUnwindEx+0x43c:
       . .\>  fffff800\`008922ac mov     rbx,qword ptr \[rsp+680h\] ; rbx = l\_pEstablisherFrame
       . .   
       . .   nt!RtlUnwindEx+0x444:
       > .   fffff800\`008922b4 cmp     rbx,rbp ; cmp l\_pEstablisherFrame, pEstablisherFrame
<        .   fffff800\`008922b7 jne     nt!RtlUnwindEx+0x479 (fffff800\`008922e9)
.        .   
.        .   nt!RtlUnwindEx+0x449:
.        \>   fffff800\`008922b9 mov     rax,qword ptr \[rsp+698h\]  ; rax = ReturnValue
.            fffff800\`008922c1 mov     rbx,qword ptr \[rsp+690h\]  ; rbx = pExceptionRecord
.            fffff800\`008922c9 mov     qword ptr \[rdi+78h\],rax   ; pOriginalContext->Rax, ReturnValue
.            fffff800\`008922cd cmp     dword ptr \[rbx\],80000029h ; cmp pExceptionRecord->ExceptionCode, STATUS\_UNWIND\_CONSOLIDATE
.<           fffff800\`008922d3 je      nt!RtlUnwindEx+0x46c (fffff800\`008922dc)
..           
..           nt!RtlUnwindEx+0x465:
..           fffff800\`008922d5 mov     qword ptr \[rdi+0F8h\],r15 ; pOriginalContext->Rip, pJumpTargetIp
..           
..           nt!RtlUnwindEx+0x46c:
.\>           fffff800\`008922dc mov     rdx,rbx ; rdx = pExceptionRecord
.            fffff800\`008922df mov     rcx,rdi ; rcx = pOriginalContext
.            fffff800\`008922e2 call    nt!RtlRestoreContext (fffff800\`008bd290)
.<           fffff800\`008922e7 jmp     nt!RtlUnwindEx+0x4a0 (fffff800\`00892310)
..           
..           nt!RtlUnwindEx+0x479:
\>.           fffff800\`008922e9 cmp     r13,qword ptr \[rdi+0F8h\] ; pOriginalContext->Rip
 .<          fffff800\`008922f0 jne     nt!RtlUnwindEx+0x48d (fffff800\`008922fd)
 ..          
 ..          nt!RtlUnwindEx+0x482:
 ..          fffff800\`008922f2 mov     ecx,0C00000FFh ; ecx = STATUS\_BAD\_FUNCTION\_TABLE
 ..          fffff800\`008922f7 call    nt!RtlRaiseStatus (fffff800\`00889df0)
 ..          fffff800\`008922fc int     3
 ..          
 ..          nt!RtlUnwindEx+0x48d:
 .\>          fffff800\`008922fd mov     rcx,qword ptr \[rsp+690h\] ; rcx = pExceptionRecord
 .           fffff800\`00892305 xor     r8d,r8d                  ; r8d = 0
 .           fffff800\`00892308 mov     rdx,rdi                  ; rdx = pOriginalContext
 .           fffff800\`0089230b call    nt!ZwRaiseException (fffff800\`008b9b80)
 .                             ; ZwRaiseException(pExceptionRecord, pOriginalContext, FALSE);
 .           
 .           nt!RtlUnwindEx+0x4a0:
 >           fffff800\`00892310 mov     r15,qword ptr \[rsp+638h\]
             fffff800\`00892318 mov     r14,qword ptr \[rsp+640h\]
             fffff800\`00892320 mov     r13,qword ptr \[rsp+648h\]
             fffff800\`00892328 mov     r12,qword ptr \[rsp+650h\]
             fffff800\`00892330 mov     rdi,qword ptr \[rsp+658h\]
             fffff800\`00892338 mov     rsi,qword ptr \[rsp+660h\]
             fffff800\`00892340 mov     rbp,qword ptr \[rsp+668h\]
             fffff800\`00892348 mov     rbx,qword ptr \[rsp+670h\]
             fffff800\`00892350 add     rsp,678h
             fffff800\`00892357 ret

```

参考资料  
  
  \[1\] wrk 源码  
  \[2\] Improving Automated Analysis of Windows x64 Binaries, skape  
  \[3\] Programming against the x64 exception handling support, Skywing  
  \[4\] Exceptional Behavior - x64 Structured Exception Handling, The NT Insider  
  
SEH 分析笔记（x64 篇）\_v1.0.0.7z

**参考资料**  
  
  \[1\] wrk 源码  
  \[2\] Improving Automated Analysis of Windows x64 Binaries, skape  
  \[3\] Programming against the x64 exception handling support, Skywing  
  \[4\] Exceptional Behavior - x64 Structured Exception Handling, The NT Insider