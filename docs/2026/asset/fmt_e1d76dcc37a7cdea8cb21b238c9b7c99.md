
进入`/path/to/xxx`目录，找到`xxx.json`。

在表达式`cvar = avar + bvar`中，加法运算符（`+`）将`avar`与`bvar`相加，得到它们的和`cvar`

在`List.of(arg0, arg1, arg2)`中，`List`接口的工厂方法`of()`接受一系列的元素，返回包含它们的只读列表。

之后我们这样调用`cmd`命令：`cmd arg0 arg1 arg2`。

```
if (condVar > someVal) {console.log("xxx")}
```

内核变量`nt!ExWorkerQueue`包含了一个含3个`EX_WORK_QUEUE`结构体成员的数组，分别表示系统中关键的（Critical也叫临界的），延迟的（Delayed）和超临界（HyperCritical）工作队列。`WorkItem`字段用于组织`IO_WORK_ITEM`结构成上面3个中的某一个工作队列。

函数`IoQueueWorkItemEx()`持有一个`IoObject`的引用，它是一个指向驱动或设备对象的指针，用以防止驱动在工作例程执行期间被卸载。

`WORK_QUEUE_ITEM`结构体内嵌的`WorkerRoutine`字段指向了I/O管理器，它提供了名为`IopProcessWorkItem()`的包装器函数，该函数调用驱动程序提供的工作例程，并降低IoObject的引用计数。

`Routine`字段指向了驱动提供的工作例程，它在`PASSIVE_LEVEL`优先级上执行。

调试器的`!exqueue`命令显示了关于工作队列和工作线程的详细信息。

APIs：
- `IoAllocateWorkItem()`
- `IoQueueWorkItem()`
- `IoInitializeWorkItem()`
- `IoQueueWorkItemEx()`
- `IoSizeofWorkItem()`
- `IoUninitializeWorkItem()`
- `IoFreeWorkItem()`

> 译者注：由于本文篇幅实在过巨，且译者英文较poor, 为了防止通篇又臭又长的译文劝退读者，故于此拦腰截断，近日将更新下半部分。

下篇已译毕，[单击此处](https://bbs.pediy.com/thread-224057.htm)。

# 关键的Windows内核数据结构一览（上）

> 译者注：近期一直在整理归纳知识体系，译几叠好文，写数篇心得，旨在温故而知新。放眼四壁，一时浩如烟海，月迷津渡。在翻译fuzzySecurity的[Windows exploit开发系列教程第十部分](https://bbs.pediy.com/thread-223812.htm)时，觅得[此文](https://bbs.kanxue.com/elink@eb8K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3y4G2k6r3g2E0j5h3y4Z5K9h3&6W2i4K6u0W2j5$3!0E0i4K6u0r3j5i4u0@1K9h3y4D9k6g2)9#2k6X3E0W2M7X3&6W2L8s2y4@1M7Y4g2U0N6q4)9J5k6h3S2@1L8h3H3`.)，甚佳，不敢独酌。

在我们的“Windows internals and debugging”课程中，学生经常会问我们这样的一些问题：Windows内核使用哪种数据结构来实现互斥量？。。本文试图通过描述Windows内核和设备驱动所使用的一些关键数据结构来回答这样的问题。

本文重点强调了系统中各种数据结构的关系，帮助读者在内核调试中进行导航。当阅读本文时，读者应该使用一个便捷的内核调试器来尝试调试命令、进行数据结构的实验。本文仅仅是一个参考，而并非新手向导。

对每种结构来说，本文提供了数据结构的一种高层次描述，同时也描述了数据结构中指向其他结构的关键字段的细节。合适的话，可以对该结构使用调试命令，实现对所提供的数据结构的巧妙操纵。大部分文中提到的数据结构均由内核的paged抑或non-paged pool分配空间，这也是内核虚拟地址空间的一部分。

下列数据结构会在文中进行描述，单击以查看详情。

| 描述 | 数据结构 |
| --- | --- |
| 双链表 | `LIST_ENTRY` |
| 进程和线程 | `EPROCESS`, `KPROCESS`, `ETHREAD`, `KTHREAD` |
| 内核&HAL | `KPCR`, `KINTERRUPT`, `CONTEXT`, `KTRAP_FRAME`, `KDPC`, `KAPC`, `KAPC_STATE` |
| 异步对象 | `DISPATCHER_HEADER`, `KEVENT`, `KSEMAPHORE`, `KMUTANT`, `KTIMER`, `KGATE`, `KQUEUE` |
| 执行体&RTL | `IO_WORKITEM` |
| I/O管理器 | `IRP`, `IO_STACK_LOCATION`, `DRIVER_OBJECT`, `DEVICE_OBJECT`, `DEVICE_NODE`, `FILE_OBJECT` |
| 对象和句柄 | `OBJECT_HEADER`, `OBJECT_TYPE`, `HANDLE_TABLE_ENTRY` |
| 内存管理器 | `MDL`, `MMPTE`, `MMPFN`, `MMPFNLIST`, `MMWSL`, `MMWSLE`, `POOL_HEADER`, `MMVAD` |
| 缓存管理器 | `VACB`, `VACB_ARRAY_HEADER`, `SHARED_CACHE_MAP`, `PRIVATE_CACHE_MAP`, `SECTION_OBJECT_POINTERS` |

## 双链表

### `nt!_LIST_ENTRY`

Windows内核中的大部分数据结构都保存在链表中，在链表头中保存着指向链表元素项的指针。`LIST_ENTRY`结构用于实现这些循环双链表。`LIST_ENTRY`结构既可用于链表头也可以用作保存单个链表元素的表项结构。`LIST_ENTRY`结构较为典型的通过嵌入到大数据结构中，维持着链表中元素的关系。

![](https://bbs.kanxue.com/upload/attach/201801/530966_uqzywjac6a8wv04.jpg)

调试命令`dt -l`命令会步过任何内嵌了该双链表的数据结构并显示链表中的所有元素。`dl`和`db`命令会向前和向后步过双链表，也可以使用`!dflink`和`!dblink`。

APIs:
- `InitializeListHead()`
- `IsListEmpty()`
- `InsertHeadList()`
- `InsertTailList()`
- `RemoveHeadList()`
- `RemoveTailList()`
- `RemoveEntryList()`
- `ExInterlockedInsertHeadList()`
- `ExInterlockedInsertTailList()`
- `ExInterlockedRemoveHeadList()`

## 进程和线程

### `nt!_EPROCESS`

Windows内核使用`ERPROCESS`结构体来表示一个进程，其包含了所有内核需要去保存关乎该进程的信息。对每一个运行在系统中的进程包括System Process和System Idle Process来说，都有一个对应的`EPROCESS`结构，System Process和System Idle Process运行在内核中。

`EPROCESS`结构属于内核的执行体层，包含了进程的资源相关信息诸如句柄表、虚拟内存、安全、调试、异常、创建信息、I/O转移统计以及进程计时等。

指向System Process的`EPROCESS`结构的指针保存在`nt!PsInitialSystemProcess`，而System Idle Process的`EPROCESS`指针保存在`nt!PsIdleProcess`。

任何进程都可以同时隶属于多个集合或组。例如，一个进程总是在系统中active进程列表中，一个进程可以属于内部运行着一个会话的进程集合，一个进程也可以是某个job的一部分。为了实现这些集合或组，`EPROCESS`结构通过不同的字段持有数个列表项。

`ActiveProcessLink`字段用于将该`EPROCESS`结构链入系统中active进程链表，该链表的头保存在内核变量中`nt!PsActiveProcessHead`。类似的，`SessionProcessLinks`字段用于将该`EPROCESS`结构链入到一个会话链表，链表头在`MM_SESSION_SPACE.ProcessList`。`JobLinks`字段用于将该`EPROCESS`结构链入到所属的job链表中，链表头在`EJOB.ProcessListHead`。内存管理器全局变量`MmProcessList`通过`MmProcessLinks`字段链入了一个进程链表。该链表可以通过`MiReplicatePteChange()`横贯以更新内核模式中关于进程虚拟地址空间的那部分。

属于进程的所有线程链表保存在`ThreadListHead`中，线程通过`ETHREAD.ThreadListEntry`排队。

内核变量`ExpTimerResolutionListHead`持有一个进程链表，使用`NtSetTimerResolution()`来改变定时器间隔。该链表被`ExpUpdateTimerResolution()`函数使用来更新时间分辨率到所有进程需求值中最小的那个。

`!process`命令从`EPROCESS`结构展示信息。`.process`命令切换调试器的虚拟地址空间上下文到特定的进程，当在一个完全的内核转储中或现场使用内核调试器时进行用户模式虚拟地址的实验时，这是一个非常危险的操作。

APIs:
- `ZwOpenProcess()`
- `ZwTerminateProcess()`
- `ZwQueryInformationProcess()`
- `ZwSetInformationProcess()`
- `PsSuspendProcess()`
- `PsResumeProcess()`
- `PsGetProcessCreateTimeQuadPart()`
- `PsSetCreateProcessNotifyRoutineEx()`
- `PsLookupProcessByProcessId()`
- `PsGetCurrentProcess()`
- `PsGetCurrentProcessId()`
- `PsGetCurrentProcessSessionId()`
- `PsGetCurrentProcessWin32Process()`
- `PsGetCurrentProcessWow64Process()`
- `PsGetCurrentThreadProcess()`
- `PsGetCurrentThreadProcessId()`
- `PsGetProcessCreateTimeQuadPart()`
- `PsGetProcessDebugPort()`
- `PsGetProcessExitProcessCalled()`
- `PsGetProcessExitStatus()`
- `PsGetProcessExitTime()`
- `PsGetProcessId()`
- `PsGetProcessImageFileName()`
- `PsGetProcessInheritedUniqueProcessId()`
- `PsGetProcessJob()`
- `PsGetProcessPeb()`
- `PsGetProcessPriorityClass()`
- `PsGetProcessSectionBaseAddress()`
- `PsGetProcessSecurityPort()`
- `PsGetProcessSessionIdEx()`
- `PsGetProcessWin32Process()`
- `PsGetProcessWin32WindowStation()`
- `PsGetProcessWow64Process()`

### `nt!_KPROCESS`

`KPROCESS`结构内嵌在`EPROCESS`结构体中，保存在`EPROCESS.Pcb`字段，它被执行体层下一级的微内核层使用，包含了线程的调度、配额、优先级以及执行时间等信息。

`ProfileListHead`字段包含了为该进程创建的性能对象链表。该链表被性能中断所使用来记录相关性能的说明。

`ReadyListHead`字段是一个线程链表，保存的是进程中出于就绪状态的线程。只有当进程不在内存中时，该链表才是非空的。链表中每项都是指向`KTHREAD`对象的`WaitListEntry`域的地址。

> 译者注：这里我扩展解释一下该字段：记录了这个进程中处于就绪状态但尚未被加入全局就绪链表的线程。当进程被换出内存后，他所属的线程一旦就绪，则被挂入到此链表，并要求换入该进程；此后当进程被换入内存时，`ReadyListHead`中的所有线程被加入到系统全局的就绪线程链表中。

`ThreadListHead`字段是进程所有线程的链表。`KTHREAD`数据结构通过`KTHREAD.ThreadListEntry`链入。内核用它来遍历进程中所有的线程。

`JobLinks`字段是同属于一个job的进程链表，链表头在`EJOB.ProcessListHead`。

### `nt!_ETHREAD`

Windows内核使用`ETHREAD`结构来表示一个线程，每个线程都有一个`ETHREAD`结构，这也包括在System Idle Process中的线程。

`ETHREAD`结构属于内核的执行体层，它包含了其他执行体组件诸如I/O管理器、安全引用监视器、内存管理、ALPC管理器等需要保存的线程相关信息。

`Tcb`字段包含了`KTHREAD`结构体，它嵌入到`ETHREAD`中并被用来存储线程调度相关信息。

每个进程都存储了一个`ETHREAD`结构体链表，代表了在进程中执行的线程，它在`EPROCESS`结构体的`ThreadListHead`字段中。

`ETHREAD`结构体通过`ThreadListEntry`字段链入到链表。

`KeyedWaitChain`字段用于保存那些正在等待一个特定事件的线程。

`IrpList`是一个IRP链表，用于表示该线程生成的I/O请求，在系统中这些请求在各驱动中正在处理但尚未完成。当线程终止时，这些IRP请求需要被取消。

`CallbackListHead`字段用于保存一个注册表回调函数的链表，它们会被调用以便于通知注册表过滤驱动关于该线程正在执行的注册操作。该字段对前向和后向通知注册表回调函数都是有效的。

`Win32StartAddress`字段是线程顶层函数的地址。该函数会通过`CreateThread()`传递给用户模式线程，通过`PsCreateSsytemThread()`给内核模式线程。

`ActiveTimerListHead`字段是个链表头，链表中包含了所有的当前线程active定时器（在一个确定的间隔后超期）。`ActiveTimerListBlock`字段用于保护链表，函数`ExpSetTimer()`通过`ETIMER.ActiveTimerListEntry`字段将定时器对象插入此链表。

调试命令`!thread`展示了线程的信息。`.thread`命令切换调试器CPU寄存器上下文到一个特定的线程。

APIs:
- `PsCreateSystemThread()`
- `PsTerminateSystemThread()`
- `PsGetCurrentThreadId()`
- `PsSetCreateThreadNotifyRoutine()`
- `PsRemoveCreateThreadNotifyRoutine()`
- `PsLookupThreadByThreadId()`
- `PsIsSystemThread()`
- `PsIsThreadTerminating()`
- `ZwCurrentThread()`

### `nt!_KTHREAD`

`KTHREAD`结构体内嵌在`ETHREAD`结构体中，存储在`ETHREAD.Tcb`字段，它被执行体层的下层微内核层所使用，它包含了线程的堆栈、调度、APC、系统调用、优先级、执行时间等信息。

`QueueListEntry`字段用于将关联到一个`KQUEUE`数据结构的线程链入链表。`KQUEUE.ThreadListHead`是该链表的头。`KQUEUE`结构体用于实现执行体工作队列（`EX_WORK_QUEUE`）以及I/O完成端口。当当前工作线程在此队列中且处于等待状态时，像`KiCommitThreadWait()`这样的函数会使用它来激活工作队列中的其他线程。

`MutantListHead`字段用于保存一个线程所获取的所有互斥量的链表。函数`KiRundownMutants()`使用此链表来检测一个线程在终止前是否释放了所有的互斥量，如果未能释放，则它会使得系统崩溃，bugcheck为`THREAD_TERMINATE_HELD_MUTEX`。

`Win32Thread`字段指向了Win32K.sys结构体W32THREAD（指向由Win32子系统管理的区域）。当一个用模式线程转换到UI线程，它会发起一个到USER32或GDI32中API的调用。函数`PsConvertToGuiThread()`执行这一转换。Win32K.sys函数`AllocateW32Thread()`调用`PsSetThreadWin32Thread()`来设置`Win32Thread`字段的值。每个线程分配的结构体尺寸存储于Win32K.sys中的`W32ThreadSize`变量中。

`WaitBlock`字段是一个4个`KWAIT_BLOCK`数组结构，线程用来等待本地内核对象。`KWAIT_BLOCK`的其中之一是保留的，它用于实现超时等待，因此它只能指向KTIMER对象。

`WaitBlockList`字段指向了`KWAIT_BLOCK`数组结构，下才能用来等待一到多个对象。该字段由函数`KiCommitThreadWait()`于线程刚刚进入到它的等待状态时设置。如果我们的线程等待的对象数量少于`THREAD_WAIT_OBJECTS`(3)，`WaitBlockList`就应该指向内置的`WaitBlock[]`数组，如果等待对象的数量超过了`THREAD_WAIT_OBJECTS`，但少于`MAXIMUM_WAIT_OBJECTS`(64)，`WaitBlockList`应该指向一个外部分配的`KWAIT_BLOCKS`数组。`ObpWaitForMultipleObjects()`是用来分配带有标签'Wait'的`KWAIT_BLOCK`数组的其中一个函数。
