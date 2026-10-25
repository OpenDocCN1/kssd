
```
kd> !handle 00000200

PROCESS 92a3a988  SessionId: 0  Cid: 05d4    Peb: 7ffdf000  ParentCid: 0230

    DirBase: 3e8a8380  ObjectTable: 9468f140  HandleCount: 1261.

    Image: AvastSvc.exe

Handle table at 9468f140 with 1261 entries in use

0200: Object: 8b3ac728  GrantedAccess: 00100081 Entry: 9468c400

Object: 8b3ac728  Type: (869e3eb0) File

    ObjectHeader: 8b3ac710 (new version)

        HandleCount: 1  PointerCount: 2

        Directory Object: 00000000  Name: \Program Files\AVAST Software\Avast\setup {HarddiskVolume2}
```

又是指向`AvastSvc.exe`进程。我们再查看下这个LPC的消息处理的服务线程是谁

```
kd> !alpc /m a1e498e0

Message a1e498e0

  MessageID             : 0x07E0 (2016)

  CallbackID            : 0x25537 (152887)

  SequenceNumber        : 0x00000026 (38)

  Type                  : LPC\_REQUEST

  DataLength            : 0x0058 (88)

  TotalLength           : 0x0070 (112)

  Canceled              : No

  Release               : No

  ReplyWaitReply        : No

  Continuation          : Yes

  OwnerPort             : 8c8f4f00 [ALPC\_CLIENT\_COMMUNICATION\_PORT]

  WaitingThread         : 88a5ad48

  QueueType             : ALPC\_MSGQUEUE\_PENDING

  QueuePort             : 92b54f00 [ALPC\_CONNECTION\_PORT]

  QueuePortOwnerProcess : 8c93d820 (svchost.exe)

  ServerThread          : 92ac8718

  QuotaCharged          : No

  CancelQueuePort       : 00000000

  CancelSequencePort    : 00000000

  CancelSequenceNumber  : 0x00000000 (0)

  ClientContext         : 001ad168

  ServerContext         : 00000000

  PortContext           : 001ad028

  CancelPortContext     : 00000000

  SecurityData          : 00000000

  View                  : 00000000

  HandleData            : 00000000
```

```
kd> !thread 92ac8718

THREAD 92ac8718  Cid 0190.11c0  Teb: 7ff82000 Win32Thread: 00000000 WAIT: (UserRequest) UserMode Non-Alertable

    885c1f00  Semaphore Limit 0x1

Not impersonating

DeviceMap                 8b67ab10

Owning Process            8c93d820       Image:         svchost.exe

Attached Process          N/A            Image:         N/A

Wait Start TickCount      21361          Ticks: 2102 (0:00:00:32.791)

Context Switch Count      37             IdealProcessor: 0

UserTime                  00:00:00.000

KernelTime                00:00:00.000

Win32 Start Address ntdll!TppWorkerThread (0x770903e7)

Stack Init a68c3ed0 Current a68c3ac8 Base a68c4000 Limit a68c1000 Call 0

Priority 8 BasePriority 8 UnusualBoost 0 ForegroundBoost 0 IoPriority 2 PagePriority 5

ChildEBP RetAddr  Args to Child

a68c3ae0 8408069d 92ac8718 00000000 8412cd20 nt!KiSwapContext+0x26 (FPO: [Uses EBP] [0,0,4])

a68c3b18 8407f4c7 92ac87d8 92ac8718 885c1f00 nt!KiSwapThread+0x266

a68c3b40 840790cf 92ac8718 92ac87d8 000000a0 nt!KiCommitThreadWait+0x1df

a68c3bb8 8422acd7 885c1f00 00000006 84080d01 nt!KeWaitForSingleObject+0x393

a68c3c20 8404028a 000000e4 00000000 a68c3be4 nt!NtWaitForSingleObject+0xc6

a68c3c20 770a70b4 000000e4 00000000 a68c3be4 nt!KiFastCallEntry+0x12a (FPO: [0,3] TrapFrame @ a68c3c34)

0285f240 00000000 00000000 00000000 00000000 ntdll!KiFastSystemCallRet (FPO: [0,0,0])
```

同样这个服务线程也在等待消息处理。在这里我们就得不到其它有用的信息了，此时我在想为什么这个`svchost.exe`没有回复LPC的请求。要想知道为什么`svchost.exe`没有回应请求那我们就需要看下这个`svchost.exe`的所有线程是不是有其他可疑的地方直接`!process 8c93d820`  ，接着就是找出可疑的线程看下

```
   THREAD 96877288  Cid 0190.06b8  Teb: 7ffda000 Win32Thread: 00000000 WAIT: (UserRequest) UserMode Non-Alertable

            96875428  SynchronizationEvent

            968761c8  NotificationEvent

            87666fb0  NotificationEvent

            96876188  NotificationEvent

            87666ff0  NotificationEvent

        IRP List:

            889cd560: (0006,0094) Flags: 00060070  Mdl: 00000000

            92bc9a00: (0006,0094) Flags: 00060070  Mdl: 00000000

            885be9d8: (0006,0094) Flags: 00060070  Mdl: 00000000

            92b11bf8: (0006,0094) Flags: 00060070  Mdl: 00000000

        Not impersonating

        DeviceMap                 8b67ab10

        Owning Process            8c93d820       Image:         svchost.exe

        Attached Process          N/A            Image:         N/A

        Wait Start TickCount      7702           Ticks: 15761 (0:00:04:05.873)

        Context Switch Count      4              IdealProcessor: 0

        UserTime                  00:00:00.000

        KernelTime                00:00:00.000

        Win32 Start Address 0x72414f4c

        Stack Init 97b5eed0 Current 97b5e648 Base 97b5f000 Limit 97b5c000 Call 0

        Priority 8 BasePriority 8 UnusualBoost 0 ForegroundBoost 0 IoPriority 2 PagePriority 5

        Kernel stack not resident.

        ChildEBP RetAddr

        97b5e660 8408069d nt!KiSwapContext+0x26 (FPO: [Uses EBP] [0,0,4])

        97b5e698 8407f4c7 nt!KiSwapThread+0x266

        97b5e6c0 8407b4a4 nt!KiCommitThreadWait+0x1df

        97b5e83c 8422b8c0 nt!KeWaitForMultipleObjects+0x535

        97b5eac8 8422b62d nt!ObpWaitForMultipleObjects+0x262

        97b5ec18 8404028a nt!NtWaitForMultipleObjects+0xcd

        97b5ec18 770a70b4 nt!KiFastCallEntry+0x12a (FPO: [0,3] TrapFrame @ 97b5ec34)

        007df730 00000000 ntdll!KiFastSystemCallRet (FPO: [0,0,0]) THREAD a52175a8  Cid 0190.0f00  Teb: 7ff92000 Win32Thread: 00000000 WAIT: (UserRequest) UserMode Alertable

            a31d7c88  SynchronizationEvent

            a31e99b0  SynchronizationEvent

        IRP List:

            a5258598: (0006,0220) Flags: 00060070  Mdl: 00000000

        Not impersonating

        DeviceMap                 8b67ab10

        Owning Process            8c93d820       Image:         svchost.exe

        Attached Process          N/A            Image:         N/A

        Wait Start TickCount      23399          Ticks: 64 (0:00:00:00.998)

        Context Switch Count      114            IdealProcessor: 0

        UserTime                  00:00:00.000

        KernelTime                00:00:00.015

        Win32 Start Address ADVAPI32!WmipEventPump (0x76fca4aa)

        Stack Init a4107ed0 Current a4107648 Base a4108000 Limit a4105000 Call 0

        Priority 8 BasePriority 8 UnusualBoost 0 ForegroundBoost 0 IoPriority 2 PagePriority 5

        ChildEBP RetAddr

        a4107660 8408069d nt!KiSwapContext+0x26 (FPO: [Uses EBP] [0,0,4])

        a4107698 8407f4c7 nt!KiSwapThread+0x266

        a41076c0 8407b4a4 nt!KiCommitThreadWait+0x1df

        a410783c 8422b8c0 nt!KeWaitForMultipleObjects+0x535

        a4107ac8 8422b62d nt!ObpWaitForMultipleObjects+0x262

        a4107c18 8404028a nt!NtWaitForMultipleObjects+0xcd

        a4107c18 770a70b4 nt!KiFastCallEntry+0x12a (FPO: [0,3] TrapFrame @ a4107c34)

        0197fed8 00000000 ntdll!KiFastSystemCallRet (FPO: [0,0,0])
```

在第一处的线程里面的IRP里面都是类似下面的输出

```
Irp is active with 1 stacks 1 is current (= 0x92bc9a70)

 No Mdl: System buffer=a30e5008: Thread 96877288:  Irp stack trace.

>[  e, 0]   5  1 92b87308 96875038 00000000-00000000    pending

      \Driver\mpsdvr

Args: 00000fc8 00000000 7d008004 00000000
```

可以看到这个IRP处于未决状态，但是由于这个是同步的，其Completion函数是空的，所以这里会一直等待其它地方处理完成才会返回另一处线程里面的IRP如下 ：

```
kd> !irp a5258598

Irp is active with 12 stacks 12 is current (= 0xa5258794)

 No Mdl: System buffer=86aa3000: Thread a52175a8:  Irp stack trace.

 [  0, 0]   0  0 00000000 00000000 00000000-00000000

Args: 00000000 00000000 00000000 00000000

 [  0, 0]   0  0 00000000 00000000 00000000-00000000

Args: 00000000 00000000 00000000 00000000

 [  0, 0]   0  0 00000000 00000000 00000000-00000000

Args: 00000000 00000000 00000000 00000000

 [  0, 0]   0  0 00000000 00000000 00000000-00000000

Args: 00000000 00000000 00000000 00000000

 [  0, 0]   0  0 00000000 00000000 00000000-00000000

Args: 00000000 00000000 00000000 00000000

 [  0, 0]   0  0 00000000 00000000 00000000-00000000

Args: 00000000 00000000 00000000 00000000

 [  0, 0]   0  0 00000000 00000000 00000000-00000000

Args: 00000000 00000000 00000000 00000000

 [  0, 0]   0  0 00000000 00000000 00000000-00000000

Args: 00000000 00000000 00000000 00000000

 [  0, 0]   0  0 00000000 00000000 00000000-00000000

Args: 00000000 00000000 00000000 00000000
```

