
# [原创]64位CreateProcess逆向:（五）创建线程的过程

点击下面进入总目录：

[64位Windows创建64位进程逆向分析（总目录）](http://bbs.pediy.com/showthread.php?t=207430)

上一篇文章我们讨论了进程创建的过程。这篇文章会介绍线程创建的过程。

64位Windows 7，在`CreateProcess`中，调用了`PspAllocateThread`来创建线程。它的原型如下：

**`PspAllocateThread`**

```
NTSTATUS
PspAllocateThread(PEPROCESS newProcess,
POBJECT_ATTRIBUTES   ObjectAttributes,
KPROCESSOR_MODE     AccessMode,
CreateProcessContext ProcessContext,
CONTEXT         context,
PINITIAL_TEB       pInitTeb,
PVOID           StartRoutine,
PVOID           StartContext,
PULONG         ptrProcessFlag,
PETHREAD *        pptrEthread,
DIYStruct*       unknownStruct,
ACCESS_STATE_EXPAND* pAccessStateExpand
)
```

它的主要参数的作用如下：

`newProcess`里包含很多进程信息，根据进程信息如进程是32位或者64位，填充不一样的结构体，初始化`_ETHREAD`成员时，需要填充`_ETHREAD`部分成员。`Context`的成员`Rip`, `Rcx`用于初始化给`ETHREAD.StartAddress`, `Win32StartAddress`, 调用`KeInitThread`初始化线程时，需要`context`，`StartRoutine`，`StartContext`来初始化线程上下文。

`PspAllocateThread`内部的流程比较简单，对于熟悉线程结构体的人来说，可以概括成一句话，就是“创建线程结构体，并初始化”，然而，对于我们这些学习者而言，就有必要先了解下线程相关的结构体。

## 线程相关结构体

Windows中，用`ETHREAD`结构体来承载线程信息，`ETHREAD`结构体保存在0环的内存空间，即归整个操作系统统一管理。此外，每一个线程还对应了`TEB`（Thread Environment Block），`TEB`是保存在各个进程的内存空间中的，即3环。

在通过WinDbg详细看他们的成员组成前，我们可以自己通过线程、进程的基础知识，推断出`ETHREAD`中应该包含哪些信息：(核心成员都在`kthread`中，`ethread`与本篇分析关系较小，故略过)

- 因为一个进程可以拥有多个线程，所以对与每个线程，应该包含它所属的进程的相关信息
- 同一个进程的多个线程，要轮流占用cpu时间片，因此互相应该有彼此信息，即一个线程应该能够遍历与自己“相关”的其它线程
- 同一个进程的多个线程，虽然共用内存空间，但是不能共用栈空间（否则函数调用就乱了），因此每个线程应该记录自己栈对应的信息
- 每个`APC`都是在特定的线程环境中执行的，从而也一定在特定的进程环境中执行，因此每个线程都有自己特有的`APC`链表

有了这些基本认识，我们再来具体看相关的结构体。

```
KTHREAD结构体
dt !_KTHREAD
ntdll!_KTHREAD
   +0x000 Header           : _DISPATCHER_HEADER
   +0x018 CycleTime        : Uint8B
   +0x020 QuantumTarget    : Uint8B
   +0x028 InitialStack     : Ptr64 Void
   +0x030 StackLimit       : Ptr64 Void
   +0x038 KernelStack      : Ptr64 Void
   +0x040 ThreadLock       : Uint8B
   +0x048 WaitRegister     : _KWAIT_STATUS_REGISTER
   +0x049 Running          : UChar
   +0x04a Alerted          : [2] UChar
   +0x04c KernelStackResident : Pos 0, 1 Bit
   +0x04c ReadyTransition  : Pos 1, 1 Bit
   +0x04c ProcessReadyQueue : Pos 2, 1 Bit
   +0x04c WaitNext         : Pos 3, 1 Bit
   +0x04c SystemAffinityActive : Pos 4, 1 Bit
   +0x04c Alertable        : Pos 5, 1 Bit
   +0x04c GdiFlushActive   : Pos 6, 1 Bit
   +0x04c UserStackWalkActive : Pos 7, 1 Bit
   +0x04c ApcInterruptRequest : Pos 8, 1 Bit
   +0x04c ForceDeferSchedule : Pos 9, 1 Bit
   +0x04c QuantumEndMigrate : Pos 10, 1 Bit
   +0x04c UmsDirectedSwitchEnable : Pos 11, 1 Bit
   +0x04c TimerActive      : Pos 12, 1 Bit
   +0x04c SystemThread     : Pos 13, 1 Bit
   +0x04c Reserved         : Pos 14, 18 Bits
   +0x04c MiscFlags        : Int4B
   +0x050 ApcState         : _KAPC_STATE
   +0x050 ApcStateFill     : [43] UChar
   +0x07b Priority         : Char
   +0x07c NextProcessor    : Uint4B
   +0x080 DeferredProcessor : Uint4B
   +0x088 ApcQueueLock     : Uint8B
   +0x090 WaitStatus       : Int8B
   +0x098 WaitBlockList    : Ptr64 _KWAIT_BLOCK
   +0x0a0 WaitListEntry    : _LIST_ENTRY
   +0x0a0 SwapListEntry    : _SINGLE_LIST_ENTRY
   +0x0b0 Queue            : Ptr64 _KQUEUE
   +0x0b8 Teb              : Ptr64 Void
   +0x0c0 Timer            : _KTIMER
   +0x100 AutoAlignment    : Pos 0, 1 Bit
   +0x100 DisableBoost     : Pos 1, 1 Bit
   +0x100 EtwStackTraceApc1Inserted : Pos 2, 1 Bit
   +0x100 EtwStackTraceApc2Inserted : Pos 3, 1 Bit
   +0x100 CalloutActive    : Pos 4, 1 Bit
   +0x100 ApcQueueable     : Pos 5, 1 Bit
   +0x100 EnableStackSwap  : Pos 6, 1 Bit
   +0x100 GuiThread        : Pos 7, 1 Bit
   +0x100 UmsPerformingSyscall : Pos 8, 1 Bit
   +0x100 VdmSafe          : Pos 9, 1 Bit
   +0x100 UmsDispatched    : Pos 10, 1 Bit
   +0x100 ReservedFlags    : Pos 11, 21 Bits
   +0x100 ThreadFlags      : Int4B
   +0x104 Spare0           : Uint4B
   +0x108 WaitBlock        : [4] _KWAIT_BLOCK
   +0x108 WaitBlockFill4   : [44] UChar
   +0x134 ContextSwitches  : Uint4B
   +0x108 WaitBlockFill5   : [92] UChar
   +0x164 State            : UChar
   +0x165 NpxState         : Char
   +0x166 WaitIrql         : UChar
   +0x167 WaitMode         : Char
   +0x108 WaitBlockFill6   : [140] UChar
   +0x194 WaitTime         : Uint4B
   +0x108 WaitBlockFill7   : [168] UChar
   +0x1b0 TebMappedLowVa   : Ptr64 Void
   +0x1b8 Ucb              : Ptr64 _UMS_CONTROL_BLOCK
   +0x108 WaitBlockFill8   : [188] UChar
   +0x1c4 KernelApcDisable : Int2B
   +0x1c6 SpecialApcDisable : Int2B
   +0x1c4 CombinedApcDisable : Uint4B
   +0x1c8 QueueListEntry   : _LIST_ENTRY
   +0x1d8 TrapFrame        : Ptr64 _KTRAP_FRAME
   +0x1e0 FirstArgument    : Ptr64 Void
   +0x1e8 CallbackStack    : Ptr64 Void
   +0x1e8 CallbackDepth    : Uint8B
   +0x1f0 ApcStateIndex    : UChar
   +0x1f1 BasePriority     : Char
   +0x1f2 PriorityDecrement : Char
   +0x1f2 ForegroundBoost  : Pos 0, 4 Bits
   +0x1f2 UnusualBoost     : Pos 4, 4 Bits
   +0x1f3 Preempted        : UChar
   +0x1f4 AdjustReason     : UChar
   +0x1f5 AdjustIncrement  : Char
   +0x1f6 PreviousMode     : Char
   +0x1f7 Saturation       : Char
   +0x1f8 SystemCallNumber : Uint4B
   +0x1fc FreezeCount      : Uint4B
   +0x200 UserAffinity     : _GROUP_AFFINITY
   +0x210 Process          : Ptr64 _KPROCESS
   +0x218 Affinity         : _GROUP_AFFINITY
   +0x228 IdealProcessor   : Uint4B
   +0x22c UserIdealProcessor : Uint4B
   +0x230 ApcStatePointer  : [2] Ptr64 _KAPC_STATE
   +0x240 SavedApcState    : _KAPC_STATE
   +0x240 SavedApcStateFill : [43] UChar
   +0x26b WaitReason       : UChar
   +0x26c SuspendCount     : Char
   +0x26d Spare1           : Char
   +0x26e CodePatchInProgress : UChar
   +0x270 Win32Thread      : Ptr64 Void
   +0x278 StackBase        : Ptr64 Void
   +0x280 SuspendApc       : _KAPC
   +0x280 SuspendApcFill0  : [1] UChar
   +0x281 ResourceIndex    : UChar
   +0x280 SuspendApcFill1  : [3] UChar
   +0x283 QuantumReset     : UChar
   +0x280 SuspendApcFill2  : [4] UChar
   +0x284 KernelTime       : Uint4B
   +0x280 SuspendApcFill3  : [64] UChar
   +0x2c0 WaitPrcb         : Ptr64 _KPRCB
   +0x280 SuspendApcFill4  : [72] UChar
   +0x2c8 LegoData         : Ptr64 Void
   +0x280 SuspendApcFill5  : [83] UChar
   +0x2d3 LargeStack       : UChar
   +0x2d4 UserTime         : Uint4B
   +0x2d8 SuspendSemaphore : _KSEMAPHORE
   +0x2d8 SuspendSemaphorefill : [28] UChar
   +0x2f4 SListFaultCount  : Uint4B
   +0x2f8 ThreadListEntry  : _LIST_ENTRY
   +0x308 MutantListHead   : _LIST_ENTRY
   +0x318 SListFaultAddress : Ptr64 Void
   +0x320 ReadOperationCount : Int8B
   +0x328 WriteOperationCount : Int8B
   +0x330 OtherOperationCount : Int8B
   +0x338 ReadTransferCount : Int8B
   +0x340 WriteTransferCount : Int8B
   +0x348 OtherTransferCount : Int8B
   +0x350 ThreadCounters   : Ptr64 _KTHREAD_COUNTERS
   +0x358 XStateSave       : Ptr64 _XSTATE_SAVE
```

正如我们之前所推测的，在实际`ETHREAD`结构体中，`Tcb.Process`成员，就是负责保存线程所属进程信息的；而`Tcb.ThreadListEntry`，可以遍历进程下的所有线程；`Tcb.StackBase`、`Tcb.InitialStack`、`Tcb.InitialStack`、`Tcb.KernelStack` `Tcb.IdealProcessor`等保存了线程的栈信息和`CPU`信息；`Tcb.ApcState`包含`APC`链表信息。

通过以下对函数的详细分析，我们可以看出`PspAllocateThread`内部，对各个结构体创建、初始化的时机和流程。

## 函数流程详细实现:

![](https://bbs.kanxue.com/upload/attach/201603/406681_wrb53qkwmx47jjb.png)

首先对参数`context`进行判断，如果不为空，并且参数`ProcessContext`中的标志位与`0x8000`后成立，再看当前`eprocess`中的`Wow64Process`，如果是32位程序，则直接返回`STATUS_INVALID_PARAMETER`错误。

![](https://bbs.kanxue.com/upload/attach/201603/406681_5xphrk2fv5515s2.png)

![](https://bbs.kanxue.com/upload/attach/201603/406681_izvsmiarqeixjij.png)

填充输出参数`pAccessStateExpand`的成员

![](https://bbs.kanxue.com/upload/attach/201603/406681_yy35paupii0jmp1.png)

将当前`EPROCESS`和`PsInitialSystemProcess`作比较，若相等，直接返回`STATUS_INVALID_PARAMETER`

![](https://bbs.kanxue.com/upload/attach/201603/406681_s66qnosykx5a8n2.png)

调用`obCreateObject`，指定对象类型为`PsThreadType`，创建`ETHREAD`对象，创建的对象类型是在系统初始时通过`ObCreateObjectType`创建的。

![](https://bbs.kanxue.com/upload/attach/201603/406681_7ma7tvpo97dcl1i.png)

初始化`ETHREAD`中的`ClientID`成员

![](https://bbs.kanxue.com/upload/attach/201603/406681_2e6arsowh3c1p1x.png)
