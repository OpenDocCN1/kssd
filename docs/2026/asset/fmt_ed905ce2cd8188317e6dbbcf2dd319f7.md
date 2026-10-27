
```c
UCHAR SuppressSymbolLoad:1; //0x289

UCHAR Prefetching:1; //0x289

UCHAR OwnsDynamicMemoryShared:1; //0x289

UCHAR OwnsChangeControlAreaExclusive:1; //0x289

UCHAR OwnsChangeControlAreaShared:1; //0x289

UCHAR OwnsPagedPoolWorkingSetExclusive:1; //0x28a

UCHAR OwnsPagedPoolWorkingSetShared:1; //0x28a

UCHAR OwnsSystemPtesWorkingSetExclusive:1; //0x28a

UCHAR OwnsSystemPtesWorkingSetShared:1; //0x28a

UCHAR TrimTrigger:2; //0x28a

UCHAR Spare1:2; //0x28a

UCHAR PriorityRegionActive; //0x28b

};

};

UCHAR CacheManagerActive; //0x28c

UCHAR DisablePageFaultClustering; //0x28d

UCHAR ActiveFaultCount; //0x28e

UCHAR LockOrderState; //0x28f

ULONG AlpcMessageId; //0x290

union

{

VOID* AlpcMessage; //0x294

ULONG AlpcReceiveAttributeSet; //0x294

};

struct _LIST_ENTRY AlpcWaitListEntry; //0x298

ULONG CacheManagerCount; //0x2a0

ULONG IoBoostCount; //0x2a4

ULONG IrpListLock; //0x2a8

VOID* ReservedForSynchTracking; //0x2ac

struct _SINGLE_LIST_ENTRY CmCallbackListHead; //0x2b0

};
```

- `+0 Tcb`：`KTHREAD`成员，见下文。
- `+218 StartAddress`：线程函数起始地址。
- `+22C Cid`：当前线程的线程ID。为`_CLIENT_ID`结构，包含了线程ID和所属的进程ID。
- `+268 ThreadListEntry`：当前进程内所有线程的双向链表。

## KTHREAD结构

与进程一样，第一个成员为`KTHREAD`结构。

`KTHREAD`结构成员列表：

```c
//0x200 bytes (sizeof)

struct _KTHREAD

{

struct _DISPATCHER_HEADER Header; //0x0

volatile ULONGLONG CycleTime; //0x10

volatile ULONG HighCycleTime; //0x18

ULONGLONG QuantumTarget; //0x20

VOID* InitialStack; //0x28

VOID* volatile StackLimit; //0x2c

VOID* KernelStack; //0x30

ULONG ThreadLock; //0x34

union _KWAIT_STATUS_REGISTER WaitRegister; //0x38

volatile UCHAR Running; //0x39

UCHAR Alerted[2]; //0x3a

union

{

struct

{

ULONG KernelStackResident:1; //0x3c

ULONG ReadyTransition:1; //0x3c

ULONG ProcessReadyQueue:1; //0x3c

ULONG WaitNext:1; //0x3c

ULONG SystemAffinityActive:1; //0x3c

ULONG Alertable:1; //0x3c

ULONG GdiFlushActive:1; //0x3c

ULONG UserStackWalkActive:1; //0x3c

ULONG ApcInterruptRequest:1; //0x3c

ULONG ForceDeferSchedule:1; //0x3c

ULONG QuantumEndMigrate:1; //0x3c

ULONG UmsDirectedSwitchEnable:1; //0x3c

ULONG TimerActive:1; //0x3c

ULONG SystemThread:1; //0x3c

ULONG Reserved:18; //0x3c

};

LONG MiscFlags; //0x3c

};

union

{

struct _KAPC_STATE ApcState; //0x40

struct

{

UCHAR ApcStateFill[23]; //0x40

CHAR Priority; //0x57

};

};

volatile ULONG NextProcessor; //0x58

volatile ULONG DeferredProcessor; //0x5c

ULONG ApcQueueLock; //0x60

ULONG ContextSwitches; //0x64

volatile UCHAR State; //0x68

CHAR NpxState; //0x69

UCHAR WaitIrql; //0x6a

CHAR WaitMode; //0x6b

volatile LONG WaitStatus; //0x6c

struct _KWAIT_BLOCK* WaitBlockList; //0x70

union

{

struct _LIST_ENTRY WaitListEntry; //0x74

struct _SINGLE_LIST_ENTRY SwapListEntry; //0x74

};

struct _KQUEUE* volatile Queue; //0x7c

ULONG WaitTime; //0x80

union

{

struct

{

SHORT KernelApcDisable; //0x84

SHORT SpecialApcDisable; //0x86

};

ULONG CombinedApcDisable; //0x84

};

VOID* Teb; //0x88

struct _KTIMER Timer; //0x90

union

{

struct

{

volatile ULONG AutoAlignment:1; //0xb8

volatile ULONG DisableBoost:1; //0xb8

volatile ULONG EtwStackTraceApc1Inserted:1; //0xb8

volatile ULONG EtwStackTraceApc2Inserted:1; //0xb8

volatile ULONG CalloutActive:1; //0xb8

volatile ULONG ApcQueueable:1; //0xb8

volatile ULONG EnableStackSwap:1; //0xb8

volatile ULONG GuiThread:1; //0xb8

volatile ULONG UmsPerformingSyscall:1; //0xb8

volatile ULONG VdmSafe:1; //0xb8

volatile ULONG UmsDispatched:1; //0xb8

volatile ULONG ReservedFlags:21; //0xb8

};

volatile LONG ThreadFlags; //0xb8

};

VOID* ServiceTable; //0xbc

struct _KWAIT_BLOCK WaitBlock[4]; //0xc0

struct _LIST_ENTRY QueueListEntry; //0x120

struct _KTRAP_FRAME* TrapFrame; //0x128

VOID* FirstArgument; //0x12c

union

{

VOID* CallbackStack; //0x130

ULONG CallbackDepth; //0x130

};

UCHAR ApcStateIndex; //0x134

CHAR BasePriority; //0x135

union

{

CHAR PriorityDecrement; //0x136

struct

{

UCHAR ForegroundBoost:4; //0x136

UCHAR UnusualBoost:4; //0x136

};

};

UCHAR Preempted; //0x137

UCHAR AdjustReason; //0x138

CHAR AdjustIncrement; //0x139

CHAR PreviousMode; //0x13a

CHAR Saturation; //0x13b

ULONG SystemCallNumber; //0x13c

ULONG FreezeCount; //0x140

volatile struct _GROUP_AFFINITY UserAffinity; //0x144

struct _KPROCESS* Process; //0x150

volatile struct _GROUP_AFFINITY Affinity; //0x154

ULONG IdealProcessor; //0x160

ULONG UserIdealProcessor; //0x164

struct _KAPC_STATE* ApcStatePointer[2]; //0x168

union

{

struct _KAPC_STATE SavedApcState; //0x170

struct

{

UCHAR SavedApcStateFill[23]; //0x170

UCHAR WaitReason; //0x187

};

};

CHAR SuspendCount; //0x188

CHAR Spare1; //0x189

UCHAR OtherPlatformFill; //0x18a

VOID* volatile Win32Thread; //0x18c

VOID* StackBase; //0x190

union

{

struct _KAPC SuspendApc; //0x194

struct

{

UCHAR SuspendApcFill0[1]; //0x194

UCHAR ResourceIndex; //0x195

};

struct

{

UCHAR SuspendApcFill1[3]; //0x194

UCHAR QuantumReset; //0x197

};

struct

{

UCHAR SuspendApcFill2[4]; //0x194

ULONG KernelTime; //0x198

};

struct

{

UCHAR SuspendApcFill3[36]; //0x194

struct _KPRCB* volatile WaitPrcb; //0x1b8

};

struct

{

UCHAR SuspendApcFill4[40]; //0x194

VOID* LegoData; //0x1bc

};

struct

{

UCHAR SuspendApcFill5[47]; //0x194

UCHAR LargeStack; //0x1c3

};

};

ULONG UserTime; //0x1c4

union

{

struct _KSEMAPHORE SuspendSemaphore; //0x1c8

UCHAR SuspendSemaphorefill[20]; //0x1c8

};

ULONG SListFaultCount; //0x1dc

struct _LIST_ENTRY ThreadListEntry; //0x1e0

struct _LIST_ENTRY MutantListHead; //0x1e8

VOID* SListFaultAddress; //0x1f0

struct _KTHREAD_COUNTERS* ThreadCounters; //0x1f4

struct _XSTATE_SAVE* XStateSave; //0x1f8

};
```

- `+0 Header`：可等待对象头部。
- `+28 InitialStack`：线程切换相关。当前线程的栈底。栈底-29C是`TrapFrame`结构首地址。
- `+2C StackLimit`：线程切换相关。当前线程的最大栈顶。ESP不能小于这个值。
- `+30 KernelStack`：线程切换相关。线程切换时，存储当前线程切换时的ESP，被切换回来时，从这里恢复ESP。
- `+39 Running`：线程状态，正在运行中为1，否则为0。
- `+3A Alerted`：可警惕性。APC相关，后续APC章节学习。
- `+3C MiscFlags`：
  - `KernelStackResident`：堆栈可扩展位。为1时，线程内核堆栈可以被扩大。0时无法扩大。
  - `SystemThread`：为1时，该线程为内核线程，否则为用户线程。
- `+40 ApcState`：APC相关，后续APC章节学习。
- `+57 Priority`：当前线程的优先级。如存储11，则优先级为11，当前线程存储在第11个就绪链表中。优先级数字越大，优先级越低。默认线程优先级为8，存储在第8个就绪链表中。
- `+60 ApcQueueLock`：APC相关，后续APC章节学习。
- `+64 ContextSwitches`：当前线程切换了多少次。
- `+68 State`：线程状态。就绪、等待、运行等。
- `+88 Teb`：3环的TEB。
- `+BC ServiceTable`：系统服务表地址，系统调用章节已学过。
- `+C0 WaitBlock`：当前线程正在等待的对象。
- `+128 TrapFrame`：`TrapFrame`结构，进0环时保存3环寄存器的值，系统调用章节已学过。
- `+135 BasePriority`：线程基础优先级。这个值就是所属进程的`BasePriority`值。
- `+13A PreviousMode`：先前模式。一些内核函数会判断这个值。
- `+150 Process`：该线程的父进程（创建该线程的进程）。
- `+168 ApcStatePointer`：APC相关，后续APC章节学习。
- `+170 SavedApcState`：APC相关，后续APC章节学习。
- `+18C Win32Thread`：win32线程，如果该线程是UI图形线程，就会多一个win32线程结构体。
- `+1E0 ThreadListEntry`：当前进程所有线程的双向链表。

# 4. OBJECT_HEADER

每一个内核对象都有一个`OBJECT_HEADER`结构，大小为`0x18`。将某个内核对象地址-`0x18`就是`OBJECT_HEADER`结构地址。

`OBJECT_HEADER`结构成员列表：

```c
nt!_OBJECT_HEADER

+0x000 PointerCount : Int4B

+0x004 HandleCount : Int4B

+0x004 NextToFree : Ptr32 Void

+0x008 Lock : _EX_PUSH_LOCK

+0x00c TypeIndex : UChar

+0x00d TraceFlags : UChar

+0x00e InfoMask : UChar

+0x00f Flags : UChar

+0x010 ObjectCreateInfo : Ptr32 _OBJECT_CREATE_INFORMATION

+0x010 QuotaBlockCharged : Ptr32 Void

+0x014 SecurityDescriptor : Ptr32 Void

+0x018 Body : _QUAD
```

- `PointerCount`：对应的内核对象被使用了多少次。如调用`ObXXXX`函数都会将该值+1。该值为0时，对应的内核对象就会被释放。
- `HandleCount`：句柄引用。
- `TypeIndex`：对应的内核对象的类型。
- `Flags`：详细作用未知。当此处的值被置为4时（对应的对象为进程对象时），对应的进程被保护，无法打开、附加。且不影响进程功能。仅对3环起作用。

# 5. 线程链表

线程大体上可分为3种状态：等待、正在运行、就绪。系统上的所有未处于正在运行状态的线程都被存储在两种链表中：等待链表+就绪链表。

## 等待链表

全局变量`KiWaitListHead`是一个双向链表的头部，里面是所有等待执行的线程的`ETHREAD`结构，每个核都有一个等待链表。链表中每个节点指向`ETHREAD`中的`KTHREAD->WaitListEntry`。

当调用`Sleep`、`WaitForSingleObject`一类的等待函数时，该线程会被加入到这个链表中。

## 就绪链表（调度链表）

所有就绪的线程都会存储在就绪链表中，共有32个就绪链表（32位），所有核共享这32个链表，每个链表对应一个优先级（0~31）。全局变量`KiDispatcherReadyListHead`存储了这32个链表的起始位置。

## 扩展：进程、线程遍历

操作系统中所有的线程 = 所有核中等待链表中的所有线程 + 32个就绪链表中所有线程 + 所有核的`KPCR`中正在运行的线程。

而通过遍历线程可以找到每个线程所属的进程。即使断链也无法阻挡这种遍历。

# 6. 线程切换-模拟版
