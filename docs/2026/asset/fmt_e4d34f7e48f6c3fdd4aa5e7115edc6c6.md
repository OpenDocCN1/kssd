
# EPROCESS 结构

## EPROCESS 结构的第三部分

`Win32Process`：指向由Windows子系统管理的进程区域，此值不为NULL，说明这是一个Windows 子系统进程(GUI 进程)

`Job`：指向一个`_EJOB`对象

`SectionObject`：进程的内存区对象（进程的可执行映像文件的内存区对象）

`SectionBaseAddress`：进程内存区对象的基地址

`QuotaBlock`：指向进程的配额块

`WorkingSetWatch`：用于监视一个进程的页面错误（由全局变量`PsWatchEnabled`开关来控制）

`Win32WindowStation`：一个进程所属的窗口站的句柄

`InheritedFromUniqueProcessId`：父进程的标识符

`LdtInformation`：用来维护一个进程的LDT(局部描述符表)信息

`VadFreeHint`：指向一个提示VAD(虚拟地址描述符)节点，用于加速在VAD树中执行查找操作

`VdmObjects`：指向当前进程的VDM数据区

`DeviceMap`：指向进程使用的设备表

![图片描述](upload/attach/202009/550779_Y57TF9ZXJ7PMAJ9.png) ![图片描述](upload/attach/202009/550779_AV7PF8U5UHNSQW7.png)

`Spare0`：一个备用域，wrk 没有使用

`PageDirectoryPte`：顶级目录页面的页表项

`Session`：指向进程所在的系统会话，实际指向`MM_SESSION_SPACE`

`ImageFileName`：进程的映像文件名

`JobLinks`：一个双链表节点，一个`Job`的所有进程构成了一个链表

`LockedPagesList`：一个指向`LOCK_HEADER`结构指针

`ThreadListHead`：一个双链表的头节点，包含了一个进程中的所有线程

`SecurityPort`：指向该进程与`Lsass`进程之间的跨进程通信端口

`PaeTop`：用于支持PAE内存访问机制

`ActiveThreads`：记录了当前进程有多少个活动线程

`GrantedAccess`：包含了进程的访问权限

`DefaultHardErrorProcessing`：指定了默认的硬件错误处理，默认为1

`LastThreadExitStatus`：记录了刚才最后一个线程的退出状态

`Peb`：一个进程的进程环境块（PEB，Process Environment Block）

## EPROCESS 结构的最后一部分

![图片描述](upload/attach/202009/550779_YMTM49CM8BWYKPG.png)

`PrefetchTrace`：指向与该进程关联的一个预取痕迹结构，以支持进程的预取

`ReadOperationCount`、`WriteOperationCount`：记录了当前进程`NtReadFile`和`NtWriteFile`系统服务被调用的次数

`OtherOperationCount`：记录了除读和写操作以外的其他I/O服务的次数

`ReadTransferCount`、`WriteTransferCount`：记录了I/O 读写操完成的次数

`OtherTransferCount`：记录了非读写操作完成的次数

`AweInfo`：一个指向`AWEINFO`结构的指针，目的是支持`AWE`(Adderss Windowing Extension,地址窗口扩展)

`SeAuditProcessCreationInfo`：包含了创建进程映像全路径名，`ImageFile` 就是从这里提取的

`Vm`：是Windows为每个进程管理虚拟内存的重要数据结构成员，类型为`MMSUPPORT`

`MmProcessLinks`：一个双链表节点，所有拥有自己地址空间的进程都将加入到一个双链表中，链表头是全局变量`MmProcessList`

`ModifiedPageCount`：记录了该进程中已修改页面的数量

`JobStatus`：记录了进程所属`Job`的状态

`Flags`：包含了进程的标志位

`ExitStatus`：包含了进程的退出状态

`NextPageColor`：用于物理页面分配算法

`SubSystemVersion`：记录了一个进程的子系统主版本和次版本号

`PriorityClass`：一个进程的优先级程度，参考`public\sdk\inc\ntpsapi.h`

`PROCESS_PRIORITY_CLASS_<XXX>`

`VadRoot`：指向一个平衡二叉树的根，用于管理该进程的虚拟地址空间

`Cookie`：一个代表该进程的随机值

## ETHREAD 结构

### ETHREAD 结构，第一部分

![图片描述](upload/attach/202009/550779_CMEQ788EFSZDEYW.png)

`CreateTime`：线程的创建时间

`ExitTime`：线程的退出时间

`LpcReplyChain`、`KeyedWaitChain`：分别用于跨进程通信(LPC)和带键事件的等待链表

`ExitStatus`：线程的退出状态

`OfsChain`：wrk 没有使用

`PostBlockList`：一个类型为`PCM_POST_BLOCK`的双链表头节点,用于一个线程向配置管理器等级注册表键的变化通知

`TerminationPort`：一个链表头，当一个线程退出时，系统会通知所有已经登记过要接收其终止事件的那些端口

`ReaperLink`：一个单链表节点，仅在线程退出时使用

`KeyedWaitVaule`：带键事件的键值

`ActiveTimerListHead`：一个双链表的头，包含了当前线程的所有定时器

`ActiveTimeListLock`：操作`ActiveTimerListHead`链表的自旋锁

`Cid`：包含了线程的唯一标识符

`LpcReplySemaphore`：用于应答LPC通知

`KeyedWaitSemaphore`：用于处理带键的事件

`LpcReplyMessage`：指向`LPCP_MESSAGE`的指针

`LpcWaitingOnPort`：说明在哪个端口对象上等待

`ImpersonationInfo`：指向线程的模仿信息

`IrpList`：一个双链表头，包含了当前线程所有正在处理但尚未完成的I/O请求(IRP对象)

`TopLevelIrp`：指向线程的顶级IRP

`DeviceToVerify`：指向一个待检验的设备对象

### ETHREAD 结构的第二部分

![图片描述](upload/attach/202009/550779_U6APDYXVUP9Q66Z.png)

`ThreadsProcess`：指向当前线程所属的进程

`StartAddress`：包含了线程的启动地址

`Win32StartAddress`：是Windows子系统的启动地址

`LpcReceivedMessageId`：包含了接收到的LPC消息ID

`ThreadList`：一个双链表的节点，每个线程都会加入到它所属进程`EPROCESS`结构的`ThreadListHead` 中

`RundownProtect`：线程的停止保护锁

`ThreadLock`：是一把推锁，用户保护线程的数据属性

`LpcReplyMessageId`：指明了当前线程正在等待对一个消息的应答

`ReadClusterSize`：指明了在一次I/O 操作中读取多少个页面，用于页面交换文件和内存映射文件的读操作

`GrantedAccess`：保护了线程的访问权限

`CrossThreadFlags`：一些针对跨线程访问的标志位

`SameThreadPassiveFlags`：一些只有在最低中断级别(被动级别)上才可以访问的标志位，并且只能被线程自身访问

`SameThreadApcFlags`：一些在APC中断级别上被该线程自身访问的标志位

`DisablePageFaultClustering`：用于控制页面的聚集与否

`ForwardClusterOnly`：指示是否仅仅前向聚集

`ActiveFaultCount`：包含了正在进行之中的页面错误数量

# 3.4 Windows的进程和线程管理

## 3.4.1 Windows进程中的句柄表

句柄是一个对象引用，同一个对象在不同的环境下可能有不同的引用（句柄）值。句柄仅在一个进程范围内有效。

`HANDLE_TABLE` 结构的定义：

![图片描述](upload/attach/202009/550779_CPJ849QPY6UN2XY.png)

![图片描述](upload/attach/202009/550779_4JCWZVC6YQQU8DF.png)

`TableCode`：指向句柄表的最高层表项页面，低2位的值表示当前句柄表的层数。

如果最低2位为0，说明句柄表只有1层，最多能容纳512个句柄。

如果最低2位为1，说明句柄表有2层，最多能容纳512\*1024个句柄。

如果最低2位为2，说明句柄表有3层，最多能容纳512\*1024\*1024个句柄。

Windows 进程的句柄表结构

![图片描述](upload/attach/202009/550779_RUJPSV2RBKXQ66Y.png)

执行体在创建进程时，首先为新进程分配一个单层句柄表。由`ExCreateHandleTable` 函数来完成，该函数调用`ExpAllocateHandleTable` 来构造初始的句柄表。由函数 `ExpAllocateHandleTableEntryShow` 来扩展句柄表。代码见`base\ntos\ex\handle.c`.

`FirstFree`：记录了当前句柄表中的空闲句柄链。通过句柄索引值来链接。

`HANDLE_TABLE_ENTRY` 结构定义如下：

![图片描述](upload/attach/202009/550779_6WWXV7FMXJHVET9.png)

`Object`：指向句柄所代表的内核对象，最低3位含义：第0 位`OBJ_PROTECT_CLOSE`，表示调用者是否允许关闭该句柄；第1位`OBJ_INHERIT`,指示该进程所创建的子进程是否可以继承该句柄，即是否将该句柄项拷贝到它们的句柄表中；第2位`OBJ_AUDIT_OBJECT_CLOSE`,指示关闭该对象时是否产生一个审计事件

第二个union中，如果句柄表项指向一个有效的对象，则`GrantedAccess` 记录了该句柄的访问掩码；如果这是一个空闲的句柄表项，则`NextFreeTableEntry` 将加入到句柄表的空闲单链表中。

一个有效的句柄有4中可能：-1，表示当前进程；-2表示当前线程；负值，其绝对值为内核句柄表中的索引；不超过226的正值，当前进程的句柄表中的索引。

内核句柄表即系统的全局句柄表，wrk 中即变量`ObpKernelHandleTable`,也是System 进程的句柄表。解析句柄函数`ObReferenceObjectByHandle`,见`base\ntos\ob\obref.c`。

将一个对象插入到句柄表中的函数是`ObInsertObject`,见`base\ntos\ob\obinsert.c`。

对象的引用有两种，一种在内核中之间通过对象地址来引用，通过 `ObReferenceObjectByPoint`记录一次新的引用，第二种，通过句柄来引用对象，通过 `ObpIncrementHandleCount`检查并记录一次句柄引用。

在一个句柄上调用了`ObReferenceObjectByHandle`后，若该对象不在使用，必须调用`ObDereferenceObject` 函数。

进程唯一ID，即`UniqueProcessId`;线程有一个`CLIENT_ID`成员`Cid`,包含了所属进程的唯一ID和线程的唯一ID。

## 3.4.2 获得当前线程和进程

Wrk 中，`KeGetCurrentThread` 实现：

![图片描述](upload/attach/202009/550779_9KMX6TEFMSM59DG.png)

Intel X86 处理器中，`fs` 指向处理器控制区(`PCR`,Processor Control Region)的内存，类型为`KPCR`，其成员`PrcbData`是当前处理器的控制块(Processor Control Block),其中包含当前线程的`KTHREAD`结构指针。`KPCR`、`KPCRCB` 结构见`base\ntos\inc\i386.h`。

`PsGetCurrentThread` ,`PsGetCurrentProcess` 实现：

![图片描述](upload/attach/202009/550779_V5EX8Y8ZR2H3SWE.png)

![图片描述](upload/attach/202009/550779_FZTN9EHK735NRB6.png)

## 3.4.3 进程和线程的创建过程

参考看雪资料 ：https://bbs.pediy.com/thread-114611.htm

http://bbs.pediy.com/showthread.php?t=114958

## 3.4.4 进程和线程的结束

在执行体层，线程的终止函数是`NtTerminateThread`,内部调用`PspTerminateThreadByPointer`完成终止处理。系统线程的终止函数是`PsTerminateSystemThread`,内部调用 `PspTerminateThreadByPointer`完成终止处理。

三个函数原型如下：

![图片描述](upload/attach/202009/550779_CWWUMBAWPX3AWPJ.png)

`NtTerminateThread` 先把参数`ThreadHandle`解出来，然后调用 `PspTerminateThreadByPointer`来结束指定的线程。如果是当前线程 `PspTerminateThreadByPointer` 参数`DirectTerminate` 为TRUE，函数不返回；否则为FALSE,返回后，线程的句柄数减1。

`PspTerminateThreadByPointer` 函数，如果是当前线程被终止，则设置结束标志 `PS_CROSS_THREAD_FLAGS_TERMINATED`,调用`PspExitThread` 退出线程。如果不是当前线程，则在该线程中插入一个内核模式APC，为它指定`PsExitSpecialApc`、`PspExitApcRundown`和 `PspExitNormalApc`，他们完成终止过程。

`NtTerminateProcess` 遍历非当前线程，调用`PspTerminateThreadByPointer`终止线程。当前进程调用`PspTerminateThreadByPointer`终止自己。

`PsTerminateProcess` 循环调用`PspTerminateThreadByPointer`以删除指定进程中的所有线程。

## 3.4.5 系统初始进程和线程

# 3.5 Windows中的线程调度

## 3.5.1 线程优先级

线程优先级分为3种类别：实时类别：16-31；动态类别：1-15；系统类别：0

线程对象得`KTHREAD` 的`BasePriority` 和`Priority` 分别为线程的静态和动态优先级。

`KiComputeNewPriority` 是计算`Priority` 值的函数，代码如下：

![图片描述](upload/attach/202009/550779_5XYBMER6MBJBGKZ.png)

![图片描述](upload/attach/202009/550779_UD9ZAY7Z9VEW79N.png)

线程的优先级重新计算只针对非实时优先级类别的线程，并且新的优先级不会低于基本优先级。

执行体在创建进程时，调用`PspComputeQuantumAndPriority` 来计算进程的优先级。

`PspComputeQuantumAndPriority` 根据`EPROCESS`对象的`PrioriytClass` 查表获得内核层优先级定义，即全局变量`PspPriorityTable`,相关定义如下：

![图片描述](upload/attach/202009/550779_YPWBK79Y5NJ8TQK.png)

执行体层的优先级经过全局表`PspPriorityTable`变换后，就成了内核层的0~31优先级值。

带有“KPRIORITY Increment”参数，函数包括`KeInsertQueueApc`、`KePulseEvent`、`KeSetEvent`、`KeReleaseMutant`、`KeReleaseSemaphore`、`KeSetProcess`、`KeBoostProirityThread`、`KeTeminateThread`。在Windows中优先级提升的典型情形：

1. 当一个I/O 操作完成时，正在等待此I/O 的线程有更多的机会被立即执行
2. 一个线程在等待到事件和信号量以后，其优先级得到提升
3. 前台线程从等待状态中醒来时，优先级提升
4. 在平衡集管理器系统线程中，扫描那些已金融就绪状态4s 但尚未被执行的线程，将它们的优先级提升至15
5. 窗口线程因窗口的活动(比如窗口消息到来)而醒来时，优先级提升

## 3.5.2 线程状态转移

`KTHREAD` 的`State` 反映了线程的当前调度状态，类型为`KTHREAD_STATE`，见`base\ntos\inc\ke.h`

![图片描述](upload/attach/202009/550779_NEW28DWBVNWQD8T.png)

线程状态说明：

1. 已初始化(Initialized): 一个线程对象的内部已初始化，是线程创建过程中的一个内部状态，此时线程尚未加入到进程的线程链表中，也没有启动
2. 就绪(Ready): 该线程已经准备就绪，等待被调度执行
3. 运行(Running) : 线程正在运行
4. 备用(Standby) : 处于备用状态的线程已经被选中作为某个处理器上下一个要运行的线程
5. 已终止(Terminated): 表示线程已经完成任务，正在进行资源回收
6. 等待(Waiting) : 表示一个线程正在等待某个条件。
7. 转移(Transition) : 处于转移状态的线程已经准备好运行，但是它的内核栈不在内存中
8. 延迟的就绪(DeferredReady) : 处于延迟的就绪状态的线程也已经准备好可以运行了，但是，与就绪状态不同的是，它尚未确定在哪个处理器上运行
9. 门等待(GateWait) : 线程正在等待一个门对象
