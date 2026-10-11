
# Debug Tutorial (完全翻译)

> 原文：[`www.pediy.com/kssd/pediy12/104828-6.html`](https://www.pediy.com/kssd/pediy12/104828-6.html)

## 使用内核调试器

### 介绍

在这篇教程中，我们将会讲述内核调试器的几个基本特性，然后去掌握它。很明显，我们不能涉及到方方面面，所以我们只会概括到其中几个方面，然后让你熟悉调试器的使用。我希望这篇文章能对你有帮助。

### 设置

为了设置内核调试器，你需要像第一章那样修改 `BOOT.INI` 文件。`/DEBUG /DEBUGPORT=COM1 /BAUDRATE=115200` 是我们需要设置的选项。如果你不设置速度，默认的将会是 19200。在 `boot.ini` 中增加一个选项，这样就可以在开机的时候选择是否要调试了。

```ini
timeout=30

default=multi(0)disk(0)rdisk(0)partition(1)\\WINDOWS

[operating systems]

multi(0)disk(0)rdisk(0)partition(1)\\WINDOWS="Microsoft Windows XP Professional" 
/fastdetect

multi(0)disk(0)rdisk(0)partition(1)\\WINDOWS="Microsoft Windows XP Professional" 
/fastdetect /debug /debugport=com1 /baudrate=115200
```
(Shown on seperate line to prevent long lines in this article. Should be on the same line in `boot.ini`.)

你可以使用 IEEE-1394 来实现设置。你也可以使用对绞线来设置用来调试的 COM 端口。

另外还有在本地实现的内核调试器，比如 Softice，你还可以使用 LiveKD。这篇文章只会涉及到微软提供的一些工具。

为了连接机器，你需要打开 WINDBG，从文件菜单中选择”Kernel Debug”，然后选择 COM 选项卡，然后确认选项。这里设置的 COM 端口是你本地连接上的，而不是你在另外一台机器上的 `BOOT.INI` 中设置的那个。如果你使用 VMWare，当你设置 COM 端口的时候，你可以将它的输出指向一个管道。你可以在这个网页看设置说明 `http://www.vmware.com/support/gsx3/doc/devices_serial_debug_gsx.html`。

如果机器已经启动，你需要键入”Control+C”来中断到调试器中。

如果你连接的时候有问题，可以试试下面的方法：

1. 在调试菜单中选择 Control+Break 或 Break。
2. 重新同步一下(`Debug->Kernel Connection`)。
3. 重新设置波特率。
4. 试试其它的 COM 端口，并保证线连接在正确的端口上。
5. 关闭调试器，并再次打开试试。
6. 确保 `boot.ini` 中内容是正确，启动的时候，应该看到”DEBUGGER ENABLED”。

### 准备开始

连接好后，我们就开始了。首先，确保你已经使用 `!symfix` 或其他方法将服务器指向了微软的符号服务器。使用环境变量设置你的符号路径，比如，增加其他的符号路径。

我们来看看一些简单的东西。找到系统中所有的进程，可以中断到内核调试器中，然后使用”`!process 0 0`”。记住，如果你得到了错误信息，必须确保你指定了正确的微软符号服务器。调试器需要根据不同的变量做不同的动作。

对每个进程，你会看到如下这样的信息。

```text
PROCESS fcdbe2c0  SessionId: 0  Cid: 00a4    Peb: 7ffdf000  ParentCid: 008c
    DirBase: 0401d000  ObjectTable: fcdc39c8  TableSize: 354.
Image: winlogon.exe
```

那么，这些都是什么意思呢？

### 进程

第一个是 `EPROCESS` 结构的地址，它是描述进程的一个内核数据结构。这个进程的地址是”fcdbe2c0”。如果你有符号，你可以使用 `DT` 命令来观察数据结构。

```text
dt _EPROCESS fcdbe2c0
```

或者

```text
dt nt!_EPROCESS fcdbe2c0
```

如果这没有得到想象中的结构，并且你正在调试 Windows2000 的机器，你可以使用”`!processfields`”命令代替。它会显示 `fcdbe2c0` 中的偏移，然后你就可以 Dump 你想要的地址了。

### 会话 ID

下面我们来讲讲会话 ID。在一个多用户的环境中，不同用户会登陆到机器中他们自己的会话中。一般来说，Session 0 就是控制台，虽然这在有”快速用户切换”特征的 WindowsXP 中会有些不同。当同时有几个用户登录 WindowsXP，然后使用”`qwinsta`”命令，你将会得到会话 ID 和用户的一些输出。

### CID

`cid` 是进程标识符。你在任务管理器中显示 PID，你就会得到这个值，也就是进程的 PID。

### PEB

PEB 就是进程环境块。这是一个用户态数据结构，那它代表什么呢？一般来说，这个结构在进程中的地址都是相同的。在内核中，系统地址空间是被映射到所有进程的，但是用户态的地址对每个进程都是各自映射的。因此，一个进程中的相同地址对应的是不同的物理地址。因此，我们需要切换到那个进程的上下文中，这样我们就能得到正确的页目录表，然后就能得到进程中的正确数据。下面我们来看看如何设置。

```text
dt _PEB 7ffdf000
```

或者

```text
!peb.
```

### PARENT CID

Parent CID 就和它的语义一样，表示它的父进程的 ID。

### DIRBASE

Dirbase 就是目录基址。某个进程运行的时候，CR3 中的值就是这个值。这个地址就是进行虚拟地址和物理地址转换所需要的一个地址。事实上，你可以使用 `!ptov` 完成这个功能。因此，页目录是 0401d000，你必须去掉最后 3 个数字(总是 0)。

```text
!ptov 0401d
```

```text
kd> !ptov 0401d

4195000 10000
40d6000 20000
3a5d000 6d000
40f9000 6e000
4117000 6f000
405a000 70000
409b000 71000
409c000 72000
412f000 73000
413a000 74000
42d1000 76000
426f000 77000
4416000 78000
4458000 7a000
439d000 7b000
43b6000 7c000
4880000 7d000
4831000 7e000
4ac3000 7f000
4b66000 80000
4a67000 81000
4f42000 84000
50a7000 89000
4f81000 8d000
523f000 94000
5240000 a8000
524f000 ac000
5226000 b3000
52c4000 bc000
4fda000 d6000
52af000 dc000
5403000 e2000
51cc000 e5000
509f000 ec000
5325000 ee000
52aa000 f0000
512c000 f2000
522e000 f3000
52f1000 f4000
5174000 f6000
5249000 f7000
575e000 f8000
56df000 f9000
5721000 fb000
58e3000 fd000
576c000 fe000
57ad000 ff000
5833000 105000
583c000 10e000
...
```

左边的是物理地址，右边的是程序中用到的虚拟地址。观察物理地址不用切换上下文。对虚拟地址，你需要使用 `!dc`, `!db` 等，而不是使用 `dc`, `db`。你还可以使用 `!eb` 命令改变物理地址中存储的数据。

```text
kd> !dc 5833000

# 5833000 72656d6d 6c616963 666f5320 72617774 mmercial Softwar
# 5833010 75502065 73696c62 73726568 30414320 e Publishers CA0
# 5833020 0d309f81 862a0906 0df78648 05010101 ..0...\*.H.......
# 5833030 8d810300 89813000 00818102 6569d3c3 .....0........ie
# 5833040 54940152 62c628ab 5554b318 458744c5 R..T.(.b..TU.D.E
# 5833050 7ec23b4a c8d7d3d8 d88d8680 9c16f10c J;.~............
# 5833060 29a96bcc 73768fb2 62c5c892 1eed3ca6 .k.)..vs...b.<..
# 5833070 13f07505 4d146c00 079098d4 817369be .u...l.M.....is.
```

### OBJECT TABLE

这个是进程中的句柄映射表。这个数据结构包含了句柄列表的大小和指针，这也就是如何如何将用户态句柄转换为相应的内核对象的方法。

### TABLE SIZE

这是表的大小。它显示了句柄表中有多少项，打开任务管理器，并看 Handle Count 这项，就是这里说的这个数。

### IMAGE

这个很简单，就是进程的镜像，在这里就是 `WINLOGON.EXE`。

### 改变上下文

有很多种方法可以改变上下文。你可以使用 `.context <DirBase>`，还有 `.trap <trap address>`，但是这里我会给你展示如何切换线程。

首先，我们来 Dump 出 WinLogon 的线程。

```text
!process fcdbe2c0  ff
```

`EPROCESS` 结构必须在内核调试器中使用，这里的 ff 表示显示所有可能的东西。

```text
kd> !process fcdbe2c0  ff

PROCESS fcdbe2c0  SessionId: 0  Cid: 00a4    Peb: 7ffdf000  ParentCid: 008c
    DirBase: 0401d000  ObjectTable: fcdc39c8  TableSize: 354.
    Image: winlogon.exe
    VadRoot fcda2ce8 Clone 0 Private 798. Modified 1085. Locked 0.
    DeviceMap fcebfa48
    Token                       e1b762d0
    ElapsedTime                 21:07:55.0882
    UserTime                    0:00:01.0241
    KernelTime                  0:00:03.0805
    QuotaPoolUsage[PagedPool]      36408
    QuotaPoolUsage[NonPagedPool]    62184
    Working Set Sizes (now,min,max)  (652, 50, 345) (2608KB, 200KB, 1380KB)
    PeakWorkingSetSize            2034
    VirtualSize                   34 Mb
    PeakVirtualSize               36 Mb
    PageFaultCount                4919
    MemoryPriority                BACKGROUND
    BasePriority                  13
    CommitCharge                  1357
        THREAD fcdbd020  Cid a4.84  Teb: 7ffde000  Win32Thread: e1b7b8e8 

WAIT: (WrUserRequest) UserMode Non-Alertable
            fcdb9820  SynchronizationEvent
        Not impersonating
        Owning Process fcdbe2c0
        Wait Start TickCount    49551906      Elapsed Ticks: 13726
        Context Switch Count    3360                 LargeStack
        UserTime                 0:00:00.0290
        KernelTime               0:00:01.0041
        Start Address 0x01001674
        Stack Init f7660000 Current f765fcc8 Base f7660000 Limit f765d000 Call 0
        Priority 15 BasePriority 15 PriorityDecrement 0 DecrementCount 0
Kernel stack not resident.
        ChildEBP RetAddr  Args to Child
        f765fce0 8042d61c 00000000 e1b7b8e8 00000001 nt!KiSwapThread+0xc5
        f765fd08 a00159cb fcdb9820 0000000d 00000001 nt!KeWaitForSingleObject+0x1a1
        f765fd44 a0014f6c 000020ff 00000000 00000001 win32k!xxxSleepThread+0x183
        f765fd54 a0014f53 0006fde8 80461691 00000000 win32k!xxxWaitMessage+0xe
        f765fd5c 80461691 00000000 00000000 00000018 win32k!NtUserWaitMessage+0xb
        f765fd5c 77e14b53 00000000 00000000 00000018 nt!KiSystemService+0xc4
        0006fe14 77e23570 00070022 00000000 00000010 +0x77e14b53
        0006fe38 77e2381e 01000000 01024c10 00000000 +0x77e23570
        0006fe58 77e3dcf8 01000000 01024c10 00000000 +0x77e2381e
        0006fe7c 01006052 01000000 00000578 00000000 +0x77e3dcf8
        0006feb8 01005fa8 000766b8 01000000 00000578 +0x1006052
        0006fef0 010099c7 000766b8 01000000 00000578 +0x1005fa8
        0006ff28 010019e4 000766b8 00000005 0007366c +0x10099c7
        0006ff58 0100179e 01000000 00000000 0007366c +0x10019e4
        0006fff4 00000000 7ffdf000 000000c8 00000100 +0x100179e
        THREAD fcdb8b80  Cid a4.c0  Teb: 7ffdd000  Win32Thread: 00000000 

WAIT: (DelayExecution) UserMode Alertable
            fcdb8c68  NotificationTimer
        Not impersonating
        Owning Process fcdbe2c0
        Wait Start TickCount    49564648      Elapsed Ticks: 984
```



### Context Switch Count

`Context Switch Count` `33085`

`UserTime` `0:00:00.0000`

`KernelTime` `0:00:00.0000`

`Start Address` `0x77e92c50`

`Win32 Start Address` `0x77f88164`

`Stack Init` `f78f4000` `Current` `f78f3cc4` `Base` `f78f4000` `Limit` `f78f1000` `Call` `0`

`Priority` `13` `BasePriority` `13` `PriorityDecrement` `0` `DecrementCount` `0`

Kernel stack not resident.

`ChildEBP` `RetAddr` `Args to Child`

`f78f3cdc` `8042d00e` `f78f3d64` `003dffac` `003dffac` `nt!KiSwapThread+0xc5`

`f78f3d04` `804c04e6` `003dfc01` `00000001` `f78f3d34` `nt!KeDelayExecutionThread+0x180`

`f78f3d54` `80461691` `00000001` `003dffac` `00000000` `nt!NtDelayExecution+0x7f`

`f78f3d54` `77f90333` `00000001` `003dffac` `00000000` `nt!KiSystemService+0xc4`

`003dffb4` `77e92ca8` `0006feb8` `00000000` `00000000` `+0x77f90333`

`003dffec` `00000000` `77f88164` `0006feb8` `00000000` `+0x77e92ca8`

... (Continues on with more information)

不要被 `Kernel stack not resident` 吓倒，内存有可能被换出到磁盘上去了。在上面的例子中，我们得到了栈回溯，但并不总是正确的。上面的情况就是说栈的内存当前被换出了。

下面，我们将上下文设置为第一个线程。

```
kd> .process fcdbe2c0

Implicit process is now fcdbe2c0

WARNING: .cache forcedecodeuser is not enabled

kd> .cache forcedecodeuser

Max cache size is     : 1024000 bytes (0x3e8 KB) 

Total memory in cache   : 0 bytes (0 KB) 

Number of regions cached: 0 

0 full reads broken into 0 partial reads 

    counts: 0 cached/0 uncached, 0.00% cached 

    bytes : 0 cached/0 uncached, 0.00% cached 

** Transition PTEs are implicitly decoded 

** User virtual addresses are translated to physical addresses before access 

kd> .thread fcdbd020   

Implicit thread is now fcdbd020 

kd> kb

  *** Stack trace for last set context - .thread/.cxr resets it 

ChildEBP RetAddr  Args to Child           

f765fce0 8042d61c 00000000 e1b7b8e8 00000001 nt!KiSwapThread+0xc5 

f765fd08 a00159cb fcdb9820 0000000d 00000001 nt!KeWaitForSingleObject+0x1a1 

f765fd44 a0014f6c 000020ff 00000000 00000001 win32k!xxxSleepThread+0x183 

f765fd54 a0014f53 0006fde8 80461691 00000000 win32k!xxxWaitMessage+0xe 

f765fd5c 80461691 00000000 00000000 00000018 win32k!NtUserWaitMessage+0xb 

f765fd5c 77e14b53 00000000 00000000 00000018 nt!KiSystemService+0xc4 

WARNING: Frame IP not in any known module. Following frames may be wrong. 

0006fe14 77e23570 00070022 00000000 00000010 0x77e14b53 

0006fe38 77e2381e 01000000 01024c10 00000000 0x77e23570 

0006fe58 77e3dcf8 01000000 01024c10 00000000 0x77e2381e 

0006fe7c 01006052 01000000 00000578 00000000 0x77e3dcf8 

0006feb8 01005fa8 000766b8 01000000 00000578 0x1006052 

0006fef0 010099c7 000766b8 01000000 00000578 0x1005fa8 

0006ff28 010019e4 000766b8 00000005 0007366c 0x10099c7 

0006ff58 0100179e 01000000 00000000 0007366c 0x10019e4 

0006fff4 00000000 7ffdf000 000000c8 00000100 0x100179e 

kd> !reload

Connected to Windows 2000 2195 x86 compatible target, ptr64 FALSE 

Loading Kernel Symbols 

................................................................................... 

Loading unloaded module list 

No unloaded module list present 

Loading User Symbols 

.................................................. 

kd> kb

  *** Stack trace for last set context - .thread/.cxr resets it 

ChildEBP RetAddr  Args to Child           

f765fce0 8042d61c 00000000 e1b7b8e8 00000001 nt!KiSwapThread+0xc5 

f765fd08 a00159cb fcdb9820 0000000d 00000001 nt!KeWaitForSingleObject+0x1a1 

f765fd44 a0014f6c 000020ff 00000000 00000001 win32k!xxxSleepThread+0x183 

f765fd54 a0014f53 0006fde8 80461691 00000000 win32k!xxxWaitMessage+0xe 

f765fd5c 80461691 00000000 00000000 00000018 win32k!NtUserWaitMessage+0xb 

f765fd5c 77e14b53 00000000 00000000 00000018 nt!KiSystemService+0xc4 

0006fde0 77e23680 00000000 00000000 0000ffff USER32!NtUserWaitMessage+0xb 

0006fe14 77e23570 00070022 00000000 00000010 USER32!DialogBox2+0x216 

0006fe38 77e2381e 01000000 01024c10 00000000 USER32!InternalDialogBox+0xd1 

0006fe58 77e3dcf8 01000000 01024c10 00000000 USER32!DialogBoxIndirectParamAorW+0x34 

0006fe7c 01006052 01000000 00000578 00000000 USER32!DialogBoxParamW+0x3d 

0006feb8 01005fa8 000766b8 01000000 00000578 winlogon!TimeoutDialogBoxParam+0x27 

0006fef0 010099c7 000766b8 01000000 00000578 winlogon!WlxDialogBoxParam+0x7b 

0006ff10 01004bdc 000766b8 00071fc8 000766b8 winlogon!BlockWaitForUserAction+0x3c 

0006ff28 010019e4 000766b8 00000005 0007366c winlogon!MainLoop+0x1bf 

0006ff58 0100179e 01000000 00000000 0007366c winlogon!WinMain+0x2a5 

0006fff4 00000000 7ffdf000 000000c8 00000100 winlogon!WinMainCRTStartup+0x156 
```

可以看到，我设置了上下文，并且重新加载了符号。因为我们需要让内核调试器读取新的上下文，并为这个进程加载新符号。如果你多次切换进程，你可能会注意到，内核调试器可能会告诉你前一个进程的符号。这就是你必须重新加载符号的原因。

你还要注意，内核栈和用户栈是不同的。当一个线程进入内核后，它必须切换相应的栈。内核有它自己的栈，用户态也有它自己的栈。

你可能会问 `PROCESS` 和 `THREAD` 列表中这些参数都代表什么意思。大致的情况上面我们已经解释过了，其中大部分信息对你来说都是没用的。

`VadRoot` `fcda2ce8` `Clone` `0` `Private` `798.` `Modified` `1085.` `Locked` `0.`

“`VadRoot`”是进程中表示所有虚拟地址的一个二叉树，包括开始和结束范围，也可以叫做“虚拟地址描述符”。如果你遍历这个树，如果在用户态调试器下使用 `x *!`，你将会得到很多相同的信息，但也会有不同。因为这些地址不仅包含模块，还有其他所有的已经映射的虚拟地址和范围。比如，你使用 `VirtualAlloc` 分配的一大块内存，也会在这个树中。

试试使用 `!vad fcda2ce8` 命令(记住要在内核调试器中使用)。

`DeviceMap` `fcebfa48`

这是系统中的 Object Device Map。内核中在全局中存储了这样一个东西。

```
kd> x nt!ObSystemDeviceMap

8047f79c nt!ObSystemDeviceMap = <NO type information> 

kd> dd nt!ObSystemDeviceMap L 1

8047f79c  fcebfa48 
```

这篇文章不会详细讲述设备栈，只是做一个简单介绍。

`Token` `e1b762d0`

这是进程的用户令牌。你可以使用 `!token e1b762d0` 命令查看信息。

`ElapsedTime` `21:07:55.0882`

`UserTime` `0:00:01.0241`

`KernelTime` `0:00:03.0805`

这是花费在内核和用户态下的时间。

`QuotaPoolUsage[PagedPool]` `36408`

`QuotaPoolUsage[NonPagedPool]` `62184`

`Working Set Sizes (now,min,max)` `(652, 50, 345) (2608KB, 200KB, 1380KB)`

`PeakWorkingSetSize` `2034`

`VirtualSize` `34 Mb`

`PeakVirtualSize` `36 Mb`

`PageFaultCount` `4919`

`MemoryPriority` `BACKGROUND`

`BasePriority` `13`

`CommitCharge` `1357`

这里就是显示内存使用量和相应统计。

下面，我们来看看线程信息。我们可以使用 `!thread <thread>` 命令。

```
kd> !thread fcdbd020   

THREAD fcdbd020  Cid a4.84  Teb: 7ffde000  Win32Thread: e1b7b8e8 WAIT: 

(WrUserRequest) UserMode Non-Alertable 

    fcdb9820  SynchronizationEvent 

Not impersonating 

Owning Process fcdbe2c0 

Wait Start TickCount    49551906      Elapsed Ticks: 14128 

Context Switch Count    3360                LargeStack 

UserTime                0:00:00.0290 

KernelTime              0:00:01.0041 

Start Address winlogon!WinMainCRTStartup (0x01001674) 

Stack Init f7660000 Current f765fcc8 Base f7660000 Limit f765d000 Call 0 

Priority 15 BasePriority 15 PriorityDecrement 0 DecrementCount 0 

Kernel stack not resident. 

ChildEBP RetAddr  Args to Child 

f765fce0 8042d61c 00000000 e1b7b8e8 00000001 nt!KiSwapThread+0xc5 

f765fd08 a00159cb fcdb9820 0000000d 00000001 nt!KeWaitForSingleObject+0x1a1 

f765fd44 a0014f6c 000020ff 00000000 00000001 win32k!xxxSleepThread+0x183 

f765fd54 a0014f53 0006fde8 80461691 00000000 win32k!xxxWaitMessage+0xe 

f765fd5c 80461691 00000000 00000000 00000018 win32k!NtUserWaitMessage+0xb 

f765fd5c 77e14b53 00000000 00000000 00000018 nt!KiSystemService+0xc4 

0006fde0 77e23680 00000000 00000000 0000ffff USER32!NtUserWaitMessage+0xb 

0006fe14 77e23570 00070022 00000000 00000010 USER32!DialogBox2+0x216 

0006fe38 77e2381e 01000000 01024c10 00000000 USER32!InternalDialogBox+0xd1 

0006fe58 77e3dcf8 01000000 01024c10 00000000 USER32!DialogBoxIndirectParamAorW+0x34 

0006fe7c 01006052 01000000 00000578 00000000 USER32!DialogBoxParamW+0x3d 

0006feb8 01005fa8 000766b8 01000000 00000578 winlogon!TimeoutDialogBoxParam+0x27 

0006fef0 010099c7 000766b8 01000000 00000578 winlogon!WlxDialogBoxParam+0x7b 
```



```
0006ff10 01004bdc 000766b8 00071fc8 000766b8 winlogon!BlockWaitForUserAction+0x3c  
0006ff28 010019e4 000766b8 00000005 0007366c winlogon!MainLoop+0x1bf  
0006ff58 0100179e 01000000 00000000 0007366c winlogon!WinMain+0x2a5  
0006fff4 00000000 7ffdf000 000000c8 00000100 winlogon!WinMainCRTStartup+0x156  
```

我们来看看这些信息。前面我们说过，进程地址就是 `EPROCESS` 结构的地址。这里就是 `ETHREAD` 结构的地址。我们可以使用 `dt nt!\_ETHREAD <address>` 或 `!threadfields`。

```
THREAD fcdbd020  Cid a4.84  Teb: 7ffde000  

       Win32Thread: e1b7b8e8 WAIT: (WrUserRequest)  
UserMode Non-Alertable  
    fcdb9820  SynchronizationEvent  
```

第一个地址是 `ETHREAD` 结构的地址，第二个是进程和线程 ID，进程 ID 是 `A4h`，线程 ID 是 `84h`。当你正在查找死锁的时候，线程 ID 很有用，查看窗口信息的时候也会很有用。这些线程 ID 是可以在用户态下的调试器中使用的。

`TEB`，就是线程环境块。这里包含了线程特有的信息，因此线程可以很简单的得到自己的信息，栈的限制和大小也在这里。`!teb` 或 `dt \_TEB<address>` 命令可以显示具体信息。显然，你必须切换到进程和线程的上下文中再使用这些命令。如果 `TEB` 是空的，意味着这个线程是系统线程，这种线程只在内核中运行。

下面一个是”Win32Thread”，这是 `WIN32k.sys` 中包含的每个线程的数据结构。

最后一个部分就是线程的状态。这个线程当前处于等待状态，更多关于等待方面的信息可以看 MSDN 里关于 `KeWaitForMultipleObjects` 或 `APC` 的文章。

`Not impersonating`

如果你对 RPC 熟悉的话，服务端可以模拟调用者来执行操作或者得到调用者的信息。如果线程处于 `impersonating` 状态，你会看到一个 `impersonation token`，并且可以使用 `!token <address>` 命令来得到信息。有一些进入内核的调用会查看这个令牌。

`Owning Process fcdbe2c0`

这是拥有这个线程的进程信息，在这里就是 `Winlogon`。有时你也会看到它属于另外一个进程，因为内核中可以使用 `KeAttachProcess` 来改变上下文。

```
Wait Start TickCount     49551906       Elapsed Ticks: 14128  
Context Switch Count     3360           LargeStack  
UserTime                 0:00:00.0290  
KernelTime               0:00:01.0041  
```

这个线程在用户态和内核态下花费的时间。”Elapsed Ticks”是线程从执行到现在的间隔时间。

`Start Address winlogon!WinMainCRTStartup (0x01001674)`

这是县城的开始地址。

`Stack Init f7660000 Current f765fcc8 Base f7660000 Limit f765d000 Call 0`

这是内核栈的信息，当前的位置和它的限制，还有开始地址。

`Priority 15 BasePriority 15 PriorityDecrement 0 DecrementCount 0`

这是线程的优先级。

另外，你还会看到其他许多东西，这里只是简单的列举一些。你还可以使用 `!lpc message <message id>` 来得到这个 LPC 调用的发起者。

我们还可以设置断点，改变内存等等。记住，改变驱动中的内存对于每个进程来说都是可见的。设置断点也是类似的，在内核中设置对每个进程也都是可见的。因为内核驱动是系统全局性的。

## 我们还能干什么？

我们还可以做其他很多事情，但是就像我前面提到的，我们只会概括到一些基本的东西。这还要看你的工作方向，比如，有人是开发硬件驱动的，他们就不用去看任何关于用户态甚至高处的驱动栈，他们只要知道一些硬件命令就可以了。有些人的工作只和用户态的程序有关，它们就只需要使用内核调试器得到一些额外的信息就可以了。

## 页表项

使用内核调试器的时候，我们会发现有些信息并不在你想象的地方，它可能被页换出了。那么，我们该如何知道内存是否有效呢？

在 X86 的机器上，当将虚拟地址转换为物理地址，包括了很多级的转换。页目录项指向页表项，如果你想找到页表项，你可以通过页目录去找。记住，我们已经知道如何使用页目录找到虚拟地址对应的物理地址。

`!pte` 可以帮你找到页表项。

```
kd> !pte 80461691  

80461691  - PDE at C0300804        PTE at C0201184  
          contains 00034163      contains 00461121  
        pfn 34 G-DA--KWV    pfn 461 G--A--KRV  

kd> dc C0300804  L1  
c0300804  00034163  

kd> dc C0201184 L1  
c0201184  00461121  
```

虚拟地址首先被加到 `GDT/LDT` 中，然后转换。虚拟地址的一部分作为得到页目录项的便宜，下一部分作为得到页表项的便宜，最后 12 位作为真实地址的真正的最后 12 位。下面我们来看看：

```
kd> !dc 461691  

#  461690 8be58bd3 dff1240d 3c558bff 01289189 .....$....U<..(.  
#  4616a0 f7fa0000 00007045 06750002 016c45f6 ....Ep....u..El.  
#  4616b0 1d8b5874 ffdff124 002e43c6 004a7b80 tX..$....C...{J.  
#  4616c0 dd8b4874 c7444389 003b5043 43c70000 tH...CD.CP;....C  
#  4616d0 00002338 3443c700 00000023 003043c7 8#....C4#....C0.  
#  4616e0 b9000000 00000001 059815ff fb508040 ............@.P.  
#  4616f0 6a006a53 f132e801 ff59fffc 40059c15 Sj.j..2...Y....@  
#  461700 44438b80 90abebfa 548bff8b 8b644c24 ..CD.......T$Ld.  

kd> dc 80461691  

80461691  0d8be58b ffdff124 893c558b 00012891  ....$....U<..(..  
804616a1  45f7fa00 02000070 f6067500 74016c45  ...Ep....u..El.t  
804616b1  241d8b58 c6ffdff1 80002e43 74004a7b  X..$....C...{J.t  
804616c1  89dd8b48 43c74443 00003b50 3843c700  H...CD.CP;....C8  
804616d1  00000023 233443c7 c7000000 00003043  #....C4#....C0..  
804616e1  01b90000 ff000000 40059815 53fb5080  ...........@.P.S  
804616f1  016a006a fcf132e8 15ff59ff 8040059c  j.j..2...Y....@.  
80461701  fa44438b 8b90abeb 24548bff 1d8b644c  .CD.......T$Ld..  
```

你可以看到，显示的东西是一样的，只不过开始的位置不一样，`dc 80461690` 得到的信息将会是一模一样的。

你可以回去得到 `PTE` 地址，找到相应的虚拟地址。

```
kd> !pte C0201184  

80461000  - PDE at C0300804        PTE at C0201184  
          contains 00034163      contains 00461121  
        pfn 34 G-DA--KWV    pfn 461 G--AKRV  
```

无效地址会怎么样呢？

```
kd> !pte 40  

00000040  - PDE at C0300000        PTE at C0000000  
          contains 01800067      contains 00000000  
        pfn 1800 --DA--UWV       not valid  

kd> dc C0300000  L1  
c0300000  01800067  

kd> dc C0000000 L1  
c0000000  00000000  

kd> !pte 66740020   

66740020  - PDE at C0300664        PTE at C0199D00  
          contains 00000000        unavailable  

kd> dc C0300664 L1  
c0300664  00000000  

kd> dc C0199D00 L1  
c0199d00  00a8d4c0  
```

“Not Valid”代表这个是一个无效的虚拟地址。(有些地址是无效的，因为你并没有用到它们，所以就没有映射)。”Unavailable”意味着不可用，但是这是有效的。如果一个地址是”unavailable”，那么可能会列举出一个在页交换文件的偏移。可执行文件的载入都是以内存映射文件实现的。你可以用 `!memusage` 来看。

`!sysptes` 命令可以 Dump 出所有的系统 `PTE`。

## 句柄信息

在内核状态下显示句柄信息很简单。只要你将上下文设置为对应的进程，指定进程，使用 `!handle` 就可以了。

```
kd> !handle 0 0 fcdbe2c0  

processor number 0  

PROCESS fcdbe2c0  SessionId: 0  Cid: 00a4    Peb: 7ffdf000  ParentCid: 008c  
    DirBase: 0401d000  ObjectTable: fcdc39c8  TableSize: 354.  
    Image: winlogon.exe  

Handle Table at e1b78000 with 354 Entries in use  

0004: Object: e1267030  GrantedAccess: 000f001f  
0008: Object: fcdee160  GrantedAccess: 00100003  
000c: Object: fcdee120  GrantedAccess: 00100003  
0010: Object: fcdbe280  GrantedAccess: 00100003  
```

第一个十用户态下使用的句柄，第二个是对象的内核地址，最后一个是这个对象的访问权限。你也可以设置一些标志来打印你想要的信息。

```
kd> !handle 0 ff fcdbe2c0  

processor number 0  

PROCESS fcdbe2c0  SessionId: 0  Cid: 00a4    Peb: 7ffdf000  ParentCid: 008c  
    DirBase: 0401d000  ObjectTable: fcdc39c8  TableSize: 354.  
    Image: winlogon.exe  

Handle Table at e1b78000 with 354 Entries in use  

0000: free handle  
0004: Object: e1267030  GrantedAccess: 000f001f  
Object: e1267030  Type: (fcebc580) Section  
    ObjectHeader: e1267018  
        HandleCount: 1  PointerCount: 1  
0008: Object: fcdee160  GrantedAccess: 00100003  
Object: fcdee160  Type: (fcebf420) Event  
    ObjectHeader: fcdee148  
        HandleCount: 1  PointerCount: 1  
000c: Object: fcdee120  GrantedAccess: 00100003  
Object: fcdee120  Type: (fcebf420) Event  
    ObjectHeader: fcdee108  
        HandleCount: 1  PointerCount: 1  
0010: Object: fcdbe280  GrantedAccess: 00100003  
Object: fcdbe280  Type: (fcebf420) Event  
    ObjectHeader: fcdbe268  
        HandleCount: 1  PointerCount: 1  
0014: Object: fcec6c50  GrantedAccess: 00000003  
Object: fcec6c50  Type: (fcee4b40) Directory  
    ObjectHeader: fcec6c38  
        HandleCount: 15  PointerCount: 44  
        Directory Object: fcebfab0  Name: KnownDlls  
        HashBucket[ 00 ]: e137ca00 Section 'gdi32.dll'  
                         e134b900 Section 'imagehlp.dll'  
                         e13154a0 Section 'url.dll'  
        HashBucket[ 02 ]: e13dee00 Section 'MPR.dll'  
        HashBucket[ 03 ]: e135b040 Section 'ole32.dll'  
                         e135d2a0 Section 'urlmon.dll'  
        HashBucket[ 04 ]: e1373600 Section 'lz32.dll'  
                         e138bde0 Section 'olesvr32.dll'  
        HashBucket[ 06 ]: e137ef40 Section 'shell32.dll'  
                         e12c7880 Section 'wldap32.dll'  
        HashBucket[ 09 ]: e13126e0 Section 'user32.dll'  
                         e13118a0 Section 'version.dll'  
        HashBucket[ 10 ]: e135af80 Section 'olecli32.dll'  
        HashBucket[ 14 ]: e13713e0 Section 'MSASN1.DLL'  
        HashBucket[ 16 ]: e1314c60 Section 'COMCTL32.DLL'  
                         fcde6c50 SymbolicLink 'KnownDllPath'  
        HashBucket[ 17 ]: e13171c0 Section 'CRYPT32.dll'  
        HashBucket[ 18 ]: e1316e60 Section 'advapi32.dll'  
                         e13583a0 Section 'oleaut32.dll'  
        HashBucket[ 19 ]: e135c980 Section 'SHLWAPI.DLL'  
                         e12c2fc0 Section 'wow32.dll'  
                         e1316460 Section 'olecnv32.dll'  
        HashBucket[ 23 ]: e131a520 Section 'comdlg32.dll'  
```



```
HashBucket[ 26 ]: e1317b80 Section 'wininet.dll'  
HashBucket[ 27 ]: e12d5fc0 Section 'olethk32.dll'  
HashBucket[ 28 ]: e124c800 Section 'MSVCRT.DLL'  
HashBucket[ 31 ]: e134dbe0 Section 'rpcrt4.dll'  
HashBucket[ 32 ]: e130b460 Section 'kernel32.dll'  

0018: Object: fccc3c88  GrantedAccess: 00100020 (Inherit)  

Object: fccc3c88  Type: (fced7c40) File  
    ObjectHeader: fccc3c70  
        HandleCount: 1  PointerCount: 1  
        Directory Object: 00000000  Name: \\WINNT\\system32 {HarddiskVolume1}  

001c: Object: fcdfb730  GrantedAccess: 000f000f  

Object: fcdfb730  Type: (fceeb420) Directory  
    ObjectHeader: fcdfb718  
        HandleCount: 14  PointerCount: 18  
        Directory Object: fcebfab0  Name: Windows  
        HashBucket[ 04 ]: e1b76d40 Port 'SbApiPort'  
        HashBucket[ 09 ]: e1b65a00 Port 'ApiPort'  
        HashBucket[ 32 ]: fcdc1750 Directory 'WindowStations'  

0020: Object: fcdb9de0  GrantedAccess: 00100003  

Object: fcdb9de0  Type: (fcebf420) Event  
    ObjectHeader: fcdb9dc8  
        HandleCount: 1  PointerCount: 1  

0024: Object: e1b7be30  GrantedAccess: 001f0001 (Protected)  

Object: e1b7be30  Type: (fceb7640) Port  
    ObjectHeader: e1b7be18  
        HandleCount: 1  PointerCount: 18  

0028: Object: fceca3c0  GrantedAccess: 00000001  

Object: fceca3c0  Type: (fcebda40) Mutant  
    ObjectHeader: fceca3a8  
        HandleCount: 15  PointerCount: 16  
        Directory Object: fcebfab0  Name: NlsCacheMutant  

002c: Object: fcdb99a0  GrantedAccess: 00100003  

Object: fcdb99a0  Type: (fcebf420) Event  
    ObjectHeader: fcdb9988  
        HandleCount: 1  PointerCount: 1  
```

你也可以指定一个句柄，显示这个句柄的信息。

```
kd> !handle 2c ff fcdbe2c0  

processor number 0  

PROCESS fcdbe2c0  SessionId: 0  Cid: 00a4    Peb: 7ffdf000  ParentCid: 008c  
    DirBase: 0401d000  ObjectTable: fcdc39c8  TableSize: 354.  
    Image: winlogon.exe  

Handle Table at e1b78000 with 354 Entries in use  

002c: Object: fcdb99a0  GrantedAccess: 00100003  

Object: fcdb99a0  Type: (fcebf420) Event  
    ObjectHeader: fcdb9988  
        HandleCount: 1  PointerCount: 1  
```

打印对象信息还有很多其他方法。对象有一个地址，你可以使用 `!object` 命令对这个地址显示相应信息。所有的内核调试器。内核调试器所做的就是找到句柄表，然后找到 `ObjectTable` 地址，然后打印数据结构信息。

下面我们来自己解析这个表。首先，`ObjectTable` 在 `fcdc39c8`。

```
kd> dc fcdc39c8  

fcdc39c8  00000000 00000162 e1b78000 fcdbe2c0  ....b...........  
fcdc39d8  000000a4 0000016f 00000300 fcdbe0a8  ....o...........  
fcdc39e8  fcdbe5a8 00000000 00000000 00000000  ................  
fcdc39f8  00000000 00000000 00000000 00000000  ................  
fcdc3a08  00000000 00000000 00000000 00000000  ................  
fcdc3a18  00000000 fcdb079c fcdc42bc 00040000  .........B......  
fcdc3a28  00000000 fcdc3a2c fcdc3a2c 0030005c  ....,:..,:..\\.0.  
fcdc3a38  00300030 00000030 07018004 69436d4d  0.0.0.......MmCi  

kd> ? 162  

Evaluate expression: 354 = 00000162  
```

可以看到，我打印出了正在使用的句柄数量。下一个值就是 `!handle` 告诉我们的句柄表的地址。

我们来看看:

```
kd> dc e1b78000  

e1b78000  e1b78400 00000000 00000000 00000000  ................  
e1b78010  00000000 00000000 00000000 00000000  ................  
e1b78020  00000000 00000000 00000000 00000000  ................  
e1b78030  00000000 00000000 00000000 00000000  ................  
e1b78040  00000000 00000000 00000000 00000000  ................  
e1b78050  00000000 00000000 00000000 00000000  ................  
e1b78060  00000000 00000000 00000000 00000000  ................  
e1b78070  00000000 00000000 00000000 00000000  ................  

kd> dc e1b78400  

e1b78400  e1b78800 e1d47000 e1d47800 00000000  .....p...x......  
e1b78410  00000000 00000000 00000000 00000000  ................  
e1b78420  00000000 00000000 00000000 00000000  ................  
e1b78430  00000000 00000000 00000000 00000000  ................  
e1b78440  00000000 00000000 00000000 00000000  ................  
e1b78450  00000000 00000000 00000000 00000000  ................  
e1b78460  00000000 00000000 00000000 00000000  ................  
e1b78470  00000000 00000000 00000000 00000000  ................  

kd> dc e1b78800  

e1b78800  00000000 00000001 61267018 000f001f  .......p&a....  
e1b78810  7cdee148 00100003 7cdee108 00100003  H..|.......|....  
e1b78820  7cdbe268 00100003 7cec6c38 00000003  h..|....8l.|....  
e1b78830  7ccc3c72 00100020 7cdfb718 000f000f  r<.| ......|....  
e1b78840  7cdb9dc8 00100003 61b7be19 001f0001  ...|.......a....  
e1b78850  7ceca3a8 00000001 7cdb9988 00100003  ...|.......|....  
e1b78860  613f5ec8 000f003f 6125dcd8 000f001f  .^?a?.....%a....  
e1b78870  7cdb9809 001f0003 7cdc3778 0002000f  ...|....x7.|....  
```

在这个地址上可以看到貌似还是一个地址，再继续 Dump 这个地址，又有三个地址，我 Dump 出第一个。我们会找到什么呢？我们看到了这个表，每 2 个一组，可以看到，第二个和对象的 `access mask` 一样，`000f001f`，跟着 `00100003`。那么，第一个数和对象有什么关系呢？我 Dump 这个地方，可以看到它们不是有效的地址。有两种方法可以继续，第一种就是调试这个内核调试器，看看它是怎么转化的。第二种就是使用其中一个一个对象进行一次调用，然后看看是如何转化的。下面我们对其中一个设置”`ba r1`”，并且祈祷 Winlogon 会去使用它。我们也可以自己写一个程序，但我有点懒。我们可以找一个 Winlogon 经常使用的对象。

我还是发现在记事本中作一个这样的试验会更简单，我们只要找到打开的文件句柄，然后设置断电。

我已经在记事本的 `ObjectTable` 中设置了 4 个”`ba r1 <address>`”，我们只能最多设置 4 个，因此我随机选择了几个，然后我关闭记事本，我们在”`ExMapHandleToPointer`”中断下来了。

```
eax=e1dfb8f0 ebx=00000000 ecx=e1dfb800 edx=00000000 esi=61caf508 edi=fb6e6bd0  

eip=804b3971 esp=fb6e6af8 ebp=fb6e6b08 iopl=0         nv up ei pl zr na po nc  

cs=0008  ss=0010  ds=0023  es=0023  fs=0030  gs=0000             efl=00000246  

nt!ExLockHandleTableEntry+0xe:  

804b3971 85f6             test    esi,esi  

kd> p;r  

eax=e1dfb8f0 ebx=00000000 ecx=e1dfb800 edx=00000000 esi=61caf508 edi=fb6e6bd0  

eip=804b3973 esp=fb6e6af8 ebp=fb6e6b08 iopl=0         nv up ei pl nz na pe nc  

cs=0008  ss=0010  ds=0023  es=0023  fs=0030  gs=0000             efl=00000202  

nt!ExLockHandleTableEntry+0x10:  

804b3973 8975f8           mov     [ebp-0x8],esi      ss:0010:fb6e6b00=e1dfb800  

kd>  

eax=e1dfb8f0 ebx=00000000 ecx=e1dfb800 edx=00000000 esi=61caf508 edi=fb6e6bd0  

eip=804b3976 esp=fb6e6af8 ebp=fb6e6b08 iopl=0         nv up ei pl nz na pe nc  

cs=0008  ss=0010  ds=0023  es=0023  fs=0030  gs=0000             efl=00000202  

nt!ExLockHandleTableEntry+0x13:  

804b3976 0f8449f0ffff     je nt!ExLockHandleTableEntry+0x5c (804b29c5)  

kd>  

eax=e1dfb8f0 ebx=00000000 ecx=e1dfb800 edx=00000000 esi=61caf508 edi=fb6e6bd0  

eip=804b397c esp=fb6e6af8 ebp=fb6e6b08 iopl=0         nv up ei pl nz na pe nc  

cs=0008  ss=0010  ds=0023  es=0023  fs=0030  gs=0000             efl=00000202  

nt!ExLockHandleTableEntry+0x15:  

804b397c 0f8eabdb0000     jle nt!ExLockHandleTableEntry+0x31 (804c152d)  

kd>  

eax=e1dfb8f0 ebx=00000000 ecx=e1dfb800 edx=00000000 esi=61caf508 edi=fb6e6bd0  

eip=804b3982 esp=fb6e6af8 ebp=fb6e6b08 iopl=0         nv up ei pl nz na pe nc  

cs=0008  ss=0010  ds=0023  es=0023  fs=0030  gs=0000             efl=00000202  

nt!ExLockHandleTableEntry+0x17:  

804b3982 8bc6             mov     eax,esi  

kd>  

eax=61caf508 ebx=00000000 ecx=e1dfb800 edx=00000000 esi=61caf508 edi=fb6e6bd0  

eip=804b3984 esp=fb6e6af8 ebp=fb6e6b08 iopl=0         nv up ei pl nz na pe nc  

cs=0008  ss=0010  ds=0023  es=0023  fs=0030  gs=0000             efl=00000202  

nt!ExLockHandleTableEntry+0x19:  

804b3984 0d00000080       or      eax,0x80000000  

kd>  

eax=e1caf508 ebx=00000000 ecx=e1dfb800 edx=00000000 esi=61caf508 edi=fb6e6bd0  

eip=804b3989 esp=fb6e6af8 ebp=fb6e6b08 iopl=0         nv up ei ng nz na pe nc  

cs=0008  ss=0010  ds=0023  es=0023  fs=0030  gs=0000             efl=00000282  

nt!ExLockHandleTableEntry+0x1e:  

804b3989 8945fc           mov     [ebp-0x4],eax      ss:0010:fb6e6b04=e1dfb800  

kd>  

eax=e1caf508 ebx=00000000 ecx=e1dfb800 edx=00000000 esi=61caf508 edi=fb6e6bd0  

eip=804b398c esp=fb6e6af8 ebp=fb6e6b08 iopl=0         nv up ei ng nz na pe nc  

cs=0008  ss=0010  ds=0023  es=0023  fs=0030  gs=0000             efl=00000282  

nt!ExLockHandleTableEntry+0x21:  

804b398c 8b45f8           mov     eax,[ebp-0x8]      ss:0010:fb6e6b00=61caf508  
```



### 调试器输出

```
kd> 

eax=61caf508 ebx=00000000 ecx=e1dfb800 edx=00000000 esi=61caf508 edi=fb6e6bd0 

eip=804b398f esp=fb6e6af8 ebp=fb6e6b08 iopl=0 nv up ei ng nz na pe nc 

cs=0008 ss=0010 ds=0023 es=0023 fs=0030 gs=0000 efl=00000282 

nt!ExLockHandleTableEntry+0x24: 

804b398f 8b4d0c mov ecx,[ebp+0xc] ss:0010:fb6e6b14=e1dfb8f0 

kd> 

eax=61caf508 ebx=00000000 ecx=e1dfb8f0 edx=00000000 esi=61caf508 edi=fb6e6bd0 

eip=804b3992 esp=fb6e6af8 ebp=fb6e6b08 iopl=0 nv up ei ng nz na pe nc 

cs=0008 ss=0010 ds=0023 es=0023 fs=0030 gs=0000 efl=00000282 

nt!ExLockHandleTableEntry+0x27: 

804b3992 8b55fc mov edx,[ebp-0x4] ss:0010:fb6e6b04=e1caf508 

kd> 

eax=61caf508 ebx=00000000 ecx=e1dfb8f0 edx=e1caf508 esi=61caf508 edi=fb6e6bd0 

eip=804b3995 esp=fb6e6af8 ebp=fb6e6b08 iopl=0 nv up ei ng nz na pe nc 

cs=0008 ss=0010 ds=0023 es=0023 fs=0030 gs=0000 efl=00000282 

nt!ExLockHandleTableEntry+0x2a: 

804b3995 0fb111 cmpxchg [ecx],edx ds:0023:e1dfb8f0=61caf508 

kd> 

Breakpoint 1 hit 

eax=61caf508 ebx=00000000 ecx=e1dfb8f0 edx=e1caf508 esi=61caf508 edi=fb6e6bd0 

eip=804b3998 esp=fb6e6af8 ebp=fb6e6b08 iopl=0 nv up ei pl zr na po nc 

cs=0008 ss=0010 ds=0023 es=0023 fs=0030 gs=0000 efl=00000246 

nt!ExLockHandleTableEntry+0x2d: 

804b3998 3bc6 cmp eax,esi 

kd> 

eax=61caf508 ebx=00000000 ecx=e1dfb8f0 edx=e1caf508 esi=61caf508 edi=fb6e6bd0 

eip=804b399a esp=fb6e6af8 ebp=fb6e6b08 iopl=0 nv up ei pl zr na po nc 

cs=0008 ss=0010 ds=0023 es=0023 fs=0030 gs=0000 efl=00000246 

nt!ExLockHandleTableEntry+0x2f: 

804b399a 0f858ddb0000 jne nt!ExLockHandleTableEntry+0x31 (804c152d) 

kd> 

eax=61caf508 ebx=00000000 ecx=e1dfb8f0 edx=e1caf508 esi=61caf508 edi=fb6e6bd0 

eip=804b39a0 esp=fb6e6af8 ebp=fb6e6b08 iopl=0 nv up ei pl zr na po nc 

cs=0008 ss=0010 ds=0023 es=0023 fs=0030 gs=0000 efl=00000246 

nt!ExLockHandleTableEntry+0x60: 

804b39a0 b001 mov al,0x1 

kd> 

eax=61caf501 ebx=00000000 ecx=e1dfb8f0 edx=e1caf508 esi=61caf508 edi=fb6e6bd0 

eip=804b39a2 esp=fb6e6af8 ebp=fb6e6b08 iopl=0 nv up ei pl zr na po nc 

cs=0008 ss=0010 ds=0023 es=0023 fs=0030 gs=0000 efl=00000246 

nt!ExLockHandleTableEntry+0x62: 

804b39a2 5e pop esi 

kd> 

eax=61caf501 ebx=00000000 ecx=e1dfb8f0 edx=e1caf508 esi=e1dfb8f0 edi=fb6e6bd0 

eip=804b39a3 esp=fb6e6afc ebp=fb6e6b08 iopl=0 nv up ei pl zr na po nc 

cs=0008 ss=0010 ds=0023 es=0023 fs=0030 gs=0000 efl=00000246 

nt!ExLockHandleTableEntry+0x63: 

804b39a3 5b pop ebx 

kd> 

eax=61caf501 ebx=00000078 ecx=e1dfb8f0 edx=e1caf508 esi=e1dfb8f0 edi=fb6e6bd0 

eip=804b39a4 esp=fb6e6b00 ebp=fb6e6b08 iopl=0 nv up ei pl zr na po nc 

cs=0008 ss=0010 ds=0023 es=0023 fs=0030 gs=0000 efl=00000246 

nt!ExLockHandleTableEntry+0x64: 

804b39a4 c9 leave 

kd> 

eax=61caf501 ebx=00000078 ecx=e1dfb8f0 edx=e1caf508 esi=e1dfb8f0 edi=fb6e6bd0 

eip=804b39a5 esp=fb6e6b0c ebp=fb6e6bc4 iopl=0 nv up ei pl zr na po nc 

cs=0008 ss=0010 ds=0023 es=0023 fs=0030 gs=0000 efl=00000246 

nt!ExLockHandleTableEntry+0x65: 

804b39a5 c20800 ret 0x8 

kd> 

eax=61caf501 ebx=00000078 ecx=e1dfb8f0 edx=e1caf508 esi=e1dfb8f0 edi=fb6e6bd0 

eip=804b3a17 esp=fb6e6b18 ebp=fb6e6bc4 iopl=0 nv up ei pl zr na po nc 

cs=0008 ss=0010 ds=0023 es=0023 fs=0030 gs=0000 efl=00000246 

nt!ExMapHandleToPointer+0x1e: 

804b3a17 f6d8 neg al 

kd> 

eax=61caf5ff ebx=00000078 ecx=e1dfb8f0 edx=e1caf508 esi=e1dfb8f0 edi=fb6e6bd0 

eip=804b3a19 esp=fb6e6b18 ebp=fb6e6bc4 iopl=0 nv up ei ng nz ac po cy 

cs=0008 ss=0010 ds=0023 es=0023 fs=0030 gs=0000 efl=00000297 

nt!ExMapHandleToPointer+0x20: 

804b3a19 1bc0 sbb eax,eax 

kd> 

eax=ffffffff ebx=00000078 ecx=e1dfb8f0 edx=e1caf508 esi=e1dfb8f0 edi=fb6e6bd0 

eip=804b3a1b esp=fb6e6b18 ebp=fb6e6bc4 iopl=0 nv up ei ng nz ac po cy 

cs=0008 ss=0010 ds=0023 es=0023 fs=0030 gs=0000 efl=00000297 

nt!ExMapHandleToPointer+0x22: 

804b3a1b 23c6 and eax,esi 

kd> 

eax=e1dfb8f0 ebx=00000078 ecx=e1dfb8f0 edx=e1caf508 esi=e1dfb8f0 edi=fb6e6bd0 

eip=804b3a1d esp=fb6e6b18 ebp=fb6e6bc4 iopl=0 nv up ei ng nz na po nc 

cs=0008 ss=0010 ds=0023 es=0023 fs=0030 gs=0000 efl=00000286 

nt!ExMapHandleToPointer+0x24: 

804b3a1d 5e pop esi 

kd> 

eax=e1dfb8f0 ebx=00000078 ecx=e1dfb8f0 edx=e1caf508 esi=00000000 edi=fb6e6bd0 

eip=804b3a1e esp=fb6e6b1c ebp=fb6e6bc4 iopl=0 nv up ei ng nz na po nc 

cs=0008 ss=0010 ds=0023 es=0023 fs=0030 gs=0000 efl=00000286 

nt!ExMapHandleToPointer+0x25: 

804b3a1e c20800 ret 0x8 
```

### 句柄与对象信息

```
kd> !handle 78 ff fcc654e0

processor number 0 

PROCESS fcc654e0 SessionId: 0 Cid: 03ac Peb: 7ffdf000 ParentCid: 02e0 

DirBase: 03cc6000 ObjectTable: fcc59988 TableSize: 16. 

Image: NOTEPAD.EXE 

Handle Table at e1dfb000 with 16 Entries in use 

0078: Object: e1caf520 GrantedAccess: 000f003f (Locked) 

Object: e1caf520 Type: (fcebc160) Key 

ObjectHeader: e1caf508 

HandleCount: 1 PointerCount: 1 

Directory Object: 00000000 Name: \\REGISTRY\\USER 
```

我们得到了什么呢？我们可以看到我们进入了句柄表中，并找到了一些奇怪的数字。那个奇怪的数字而后被对象头 `e1caf50` 替换，真正的对象在 `ObjectHeader+18h` 处，`e1caf508+18=e1caf520`。

```
kd> !object e1caf520

Object: e1caf520 Type: (fcebc160) Key 

ObjectHeader: e1caf508 

HandleCount: 1 PointerCount: 1 

Directory Object: 00000000 Name: \\REGISTRY\\USER 
```

哪个计算能得到这个值呢？很简单，`or eax ,0x80000000`。我们来试试其它的对象，比如 `7cdee148`，加上 `18h`，然后使用 `!object` 命令。

```
kd> !object fcdee160

Object: fcdee160 Type: (fcebf420) Event 

ObjectHeader: fcdee148 

HandleCount: 1 PointerCount: 1 
```

这很简单，不是吗？

### 堆信息

我们可以在内核态下显示某个进程的堆信息。把上下文设置为对应进程，然后打印出 `PEB` 信息。`PEB` 会显示堆的个数和一个使用的堆列表，每个堆都有一个 `VirtualAlloc` 链表。我们当然也可以在用户态下得到这些消息。最大的不同就是你可能不能打印出所有信息，如果它被页换出的话。但是你并不需要自己去完成这些工作，还是和第三篇那样在内核态下使用 `!heap`，设置上下文就可以了。

### Pools

内核驱动一般都使用分页和非分页内存。你可以使用 `!pool` 命令来确定它的大小，tag 信息等等。如果你开发过驱动，你就会知道，在内核中分配的内存是来自于全局的，而不是对于某个进程而言的。因此，如果你破坏了内存，你也会使其他驱动崩溃。当一个驱动分配内存时，它们可以指定一个 `pool tag`，你就可以根据这个来确定是哪个驱动分配了内存。你可以还使用 `!poolfind *` 来找到你想要的那块内存。网上也有一些关于这方面的资料。

下面我们来看看对一个对象使用 `!pool` 命令。

```
kd> !pool e1caf520

e1caf000 size: 60 previous size: 0 (Allocated) NtFd 

e1caf060 size: 20 previous size: 60 (Free) .... 

e1caf080 size: 40 previous size: 20 (Allocated) NtFs 

e1caf0c0 size: 20 previous size: 40 (Free) Key 

e1caf0e0 size: 20 previous size: 20 (Allocated) CMVI 

e1caf100 size: 40 previous size: 20 (Allocated) CMVa 

e1caf140 size: 20 previous size: 40 (Free) CMVa 

e1caf160 size: 40 previous size: 20 (Lookaside) Key (Protected) 

e1caf1a0 size: 20 previous size: 40 (Allocated) Ntf0 

e1caf1c0 size: 40 previous size: 20 (Allocated) IoNm 

e1caf200 size: 20 previous size: 40 (Free) Gla8 

e1caf220 size: 40 previous size: 20 (Allocated) CMVa 

e1caf260 size: 40 previous size: 40 (Allocated) CMVa 

e1caf2a0 size: 20 previous size: 40 (Free) Key 

e1caf2c0 size: 60 previous size: 20 (Allocated) Ntfo 

e1caf320 size: 40 previous size: 60 (Allocated) CMNb (Protected) 

e1caf360 size: 40 previous size: 40 (Allocated) CMIn (Protected) 

e1caf3a0 size: a0 previous size: 40 (Allocated) SeSd 

e1caf440 size: 20 previous size: a0 (Free) Usqm 

e1caf460 size: 40 previous size: 20 (Allocated) CMVa 

e1caf4a0 size: 40 previous size: 40 (Allocated) CMVa 
```



# 内核调试器内存与线程分析

## 内存分配列表示例

e1caf4e0 size: 20 previous size: 40 (Allocated) Ntf0
e1caf500 size: 40 previous size: 20 (Allocated) Key (Protected)
e1caf540 size: 20 previous size: 40 (Free) Gla8
e1caf560 size: 40 previous size: 20 (Allocated) CMVa
e1caf5a0 size: 20 previous size: 40 (Free) SYSA
e1caf5c0 size: 20 previous size: 20 (Allocated) CMNb (Protected)
e1caf5e0 size: 40 previous size: 20 (Allocated) CMNb (Protected)
e1caf620 size: 40 previous size: 40 (Allocated) CMNb (Protected)
e1caf660 size: a0 previous size: 40 (Allocated) CMDa
e1caf700 size: 40 previous size: a0 (Allocated) NtFL
e1caf740 size: 40 previous size: 40 (Allocated) CMNb (Protected)
e1caf780 size: a0 previous size: 40 (Allocated) MmSt
e1caf820 size: 40 previous size: a0 (Allocated) CMNb (Protected)
e1caf860 size: 20 previous size: 40 (Allocated) ObDi
e1caf880 size: 20 previous size: 20 (Free) CMDa
e1caf8a0 size: 20 previous size: 20 (Allocated) CMVI
e1caf8c0 size: 40 previous size: 20 (Allocated) IoNm
e1caf900 size: 80 previous size: 40 (Allocated) FSim
e1caf980 size: 20 previous size: 80 (Free) MmSt
e1caf9a0 size: 60 previous size: 20 (Allocated) CMDa
e1cafa00 size: 3e0 previous size: 60 (Allocated) NtfF
e1cafde0 size: 60 previous size: 3e0 (Allocated) CMDa
e1cafe40 size: 40 previous size: 60 (Allocated) CMVa
e1cafe80 size: 40 previous size: 40 (Allocated) CMVa
e1cafec0 size: 80 previous size: 40 (Allocated) FSim
e1caff40 size: 40 previous size: 80 (Allocated) Key (Protected)
e1caff80 size: 80 previous size: 40 (Allocated) Gla@

调试器将所给地址的最后 3 个数字去掉，换成 0，在这里就是将 `e1caf520` 替换成 `e1caf000`。然后，就开始遍历这块内存，它是一个链表，就像用户态堆那样，但这里不是用指针连接起来，而是以 `size` 连接起来。这些 `size` 可以指向前面或后面，这个例子中，`size` 左移 5 位。下面我们举一个例子。

```
kd> db e1caff80
e1caff80  02 81 02 04 47 6c 61 40-59 01 10 08 00 00 00 00  ....Gla@Y.......
e1caff90  00 00 00 80 00 00 00 00-07 00 00 00 00 00 00 00  ................
e1caffa0  00 00 00 00 14 00 00 80-8b 00 00 00 f0 00 75 00  ..u.
e1caffb0  00 00 00 00 d4 d0 c8 00-00 00 00 00 00 00 00 00  .d..c..
e1caffc0  01 00 00 00 00 00 00 00-ff ff ff 00 04 00 00 00  ..
e1caffd0  08 00 00 00 c8 d0 d4 00-00 00 00 00 d4 d0 c8 00  ...d...
e1caffe0  01 05 00 00 00 00 00 05-15 00 00 00 85 e7 7e 2f  ..../
e1cafff0  23 f3 f6 63 f8 9f b4 74-01 02 00 00 00 00 00 00  #..c...t...

kd> ? 02 << 5
Evaluate expression: 64 = 00000040

kd> ? 04 << 5
Evaluate expression: 128 = 00000080
```

中间的标志表示这个地址是否正在使用。下面的 4 个字节是 `tag`，然后就是分配的内存。总共有 8 字节的头部，和用户态一样。当然，内核中能分配很大一块内存，这是如何实现的？你会得到内存中的一些页，然后你会得到那个地址，像 `e1caf000`，用户态下会有 32 字节的头部，而这里没有。

## 就绪线程

就绪线程是那些准备运行的线程，处于就绪状态。

```
kd> !ready 0
Ready Threads at priority 0
  THREAD fcebf020  Cid 8.4  Teb: 00000000  Win32Thread: 00000000 READY
```

只使用 `!ready` 将会显示所有的线程信息。在 Windows XP/2003 下，`!running` 命令可以显示多处理器系统中的正在运行的线程。用户态下可以使用 `~<number><number>` 来切换线程，内核态下切换处理器。

## CPU ID

另外还有 `!cpuid` 这个命令。

```
kd> !cpuid
CP  F/M/S  Manufacturer    MHz
 0  5,8,12 AuthenticAMD    350
```

## 虚拟内存使用

你可以使用 `!vm` 显示内存使用状况。

```
kd> !vm
*** Virtual Memory Usage ***
  Physical Memory:   31613   (  126452 Kb)
  Page File: \\??\C:\pagefile.sys
     Current:    184320Kb Free Space:    180464Kb
     Minimum:    184320Kb Maximum:       368640Kb
  Available Pages:   17617   (   70468 Kb)
  ResAvail Pages:     26203   (  104812 Kb)
  Modified Pages:       543   (    2172 Kb)
  NonPagedPool Usage:   677   (    2708 Kb)
  NonPagedPool Max:    12528   (   50112 Kb)
  PagedPool 0 Usage:    2993   (   11972 Kb)
  PagedPool 1 Usage:     439   (    1756 Kb)
  PagedPool 2 Usage:     451   (    1804 Kb)
  PagedPool Usage:      3883   (   15532 Kb)
  PagedPool Maximum:   25600   (  102400 Kb)
  Shared Commit:         324   (    1296 Kb)
  Special Pool:           0   (       0 Kb)
  Free System PTEs:   14442   (   57768 Kb)
  Shared Process:       1125   (    4500 Kb)
  PagedPool Commit:     3883   (   15532 Kb)
  Driver Commit:         912   (    3648 Kb)
  Committed pages:     11945   (   47780 Kb)
  Commit limit:        73862   (  295448 Kb)
  Total Private:        5662   (   22648 Kb)
  00a4 winlogon.exe     1358 (    5432 Kb)
  00d8 services.exe      587 (    2348 Kb)
  02e4 explorer.exe      557 (    2228 Kb)
  01b4 SPOOLSV.EXE       503 (    2012 Kb)
  00e4 lsass.exe         446 (    1784 Kb)
  01e8 svchost.exe       412 (    1648 Kb)
  00a8 csrss.exe         329 (    1316 Kb)
  0374 mspaint.exe       303 (    1212 Kb)
  008c smss.exe          273 (    1092 Kb)
  018c svchost.exe       227 (     908 Kb)
  0290 winvnc.exe        164 (     656 Kb)
  0264 winmgmt.exe       154 (     616 Kb)
  02e0 cmd.exe           143 (     572 Kb)
  0228 mstask.exe        141 (     564 Kb)
  0210 regsvc.exe         59 (     236 Kb)
  0008 System              6 (      24 Kb)
```

如果你想看看系统是否已经用完了内存，你只要比较 `committed pages` 和 `commit limit` 就可以了。提交的页表示你现在使用了多少页，提交限制表示你总共可以使用多少。

你还可以使用 `!memusage` 查看所有内存映射文件。

```
kd> !memusage
  loading PFN database
loading (99% complete)
             Zeroed:   3163 ( 12652 kb)
               Free:     20 (    80 kb)
            Standby:  14434 ( 57736 kb)
           Modified:    543 (  2172 kb)
    ModifiedNoWrite:      0 (     0 kb)
       Active/Valid:  13567 ( 54268 kb)
         Transition:      0 (     0 kb)
            Unknown:      0 (     0 kb)
              TOTAL:  31727 (126908 kb)
  Building kernel map
  Finished building kernel map
  Usage Summary (in Kb):
Control Valid Standby Dirty Shared Locked PageTables  name
fce7a3c8   468  16776     0     0     0     0    No Name for File
fccf6148     0   2876     0     0     0     0  mapped_file( CIM.REP )
fcd43388     0     96     0     0     0     0  mapped_file( chord.wav )
fcc40568     0     48     0     0     0     0  mapped_file( index.dat )
fcc64308     0   2420     0     0     0     0  mapped_file( mshtml.dll )
fcce8b28     0    188     0     0     0     0  mapped_file( rasppp.dll )
fcc76728     0    208     0     0     0     0  mapped_file( h323.tsp )
fccec3c8   220     92     0     0     0     0  mapped_file( netcfgx.dll )
fce48e88  1152    180     0     0     0     0  mapped_file( win32k.sys )
fcde4f28    16    460     0     0     0     0  mapped_file( wininet.dll )
fcc7f808     0     32     0     0     0     0  mapped_file( wbemcomn.dll )
fcc4d7a8     0    388     0     0     0     0  mapped_file( ntvdm.exe )
fcceb128    80    200     0     0     0     0  mapped_file( rasdlg.dll )
fcde5a08   200    896     0   120     0     0  mapped_file( shell32.dll )
fceca188  3196   1280     0     0     0     0    No Name for File
fccecd88   108     36     0     0     0     0  mapped_file( tapisrv.dll )
fcda5a28     8    204     0     0     0     0  mapped_file( msgina.dll )
fcc83348     0    300     0     0     0     0  mapped_file( wbemess.dll )
fcce9248    40    756     0     0     0     0  mapped_file( shdocvw.dll )
fcd10a68     0     32     0     0     0     0  mapped_file( notepad.exe )
fcd7dd08     0    504     0     0     0     0  mapped_file( jscript.dll )
fcc7cc48     0     32     0     0     0     0  mapped_file( mswsock.dll )
fcc4a4a8     0    116     0     0     0     0  mapped_file( dxtmsft.dll )
fcce95e8    32     40     0     0     0     0  mapped_file( ntmarta.dll )
fcce89a8     0     12     0     0     0     0  mapped_file( ntlsapi.dll )
fcc821a8     0     68     0     0     0     0  mapped_file( modemui.dll )
```



### 内存映射文件与进程列表

```
fcc7a4e8    16    196     0     0     0     0  mapped\_file( msi.dll )
fccec688   112     32     0     0     0     0  mapped\_file( rasmans.dll )
fcc4ea68   240     64     0     0     0     0  mapped\_file( mspaint.exe )
fcd64ba8     0     32     0     0     0     0  mapped\_file( tapisrv.dll )
fcde4aa8     0     36     0     0     0     0  mapped\_file( mpr.dll )
fcdc33a8   164     64     0     0     0     0  mapped\_file( winsrv.dll )
fcceeb08     0     32     0     0     0     0  mapped\_file( wkssvc.dll )
fccf3ee8     0     32     0     0     0     0  mapped\_file( winlogon.exe )
fcc54a28   112     36     0     0     0     0  mapped\_file( cmd.exe )
fcc60408     0    148     0     0     0     0  mapped\_file( dxtrans.dll )
fccee4a8     0     32     0     0     0     0  mapped\_file( cryptsvc.dll )
fcc7fe08     0     32     0     0     0     0  mapped\_file( SPOOLSV.EXE )
fcd7e788    80    512     0     0     0     0  mapped\_file( browseui.dll )
fcc7ed28     0      0     4     0     0     0  mapped\_file( software.LOG )
fcc7f128    40    264     0     8     0     0  mapped\_file( netshell.dll )
fcc51ac8     0     32     0     0     0     0  mapped\_file( jscript.dll )
...
\--------    12      0     0 ----- -----     8  pagefile section (78a7)
\--------   456      0     0 ----- -----    72  process ( mstask.exe )
\--------   452    536   116 ----- -----    92  process ( lsass.exe )
\--------   140      0     0 ----- -----    28  process ( smss.exe )
\--------    88    452     0 ----- -----    52  process ( winmgmt.exe )
\--------   236      0     0 ----- -----    36  process ( regsvc.exe )
\--------   520      0     0 ----- -----    72  process ( winvnc.exe )
\--------    12      0     0 ----- -----     8  pagefile section (6933)
\--------  1320    100     0 -----     0 -----  driver ( ntoskrnl.exe )
\--------    60     16     0 -----     0 -----  driver ( hal.dll )
...
```

可以看到，加载到系统中的文件都是以内存映射文件的方式进行的。这就是说，操作系统是使用从磁盘上加载到内存中的方式，而不是直接拷贝到内存，然后放入分页文件。为了得到内存映射文件的信息，比如当前有多少次映射等等，可以使用 `!ca` 命令。

```
kd> !ca fcdee008

ControlArea @fcdee008

  Segment:    e1b764c8    Flink               0   Blink:               0
  Section Ref        1    Pfn Ref             1e   Mapped Views:        1
  User Ref           2    Subsections          4   Flush Count:         0
  File Object fcdbe728    ModWriteCount        0   System Views:        0
  WaitForDel         0    Paged Usage        100   NonPaged Usage       c0
  Flags (90000a0) Image File HadUserReference Accessed
   File: \\WINNT\\system32\\winlogon.exe

Segment @ e1b764c8:
   ControlArea  fcdee008  Total Ptes          0  NonExtendPtes:        2d
   WriteUserRef       2d  Extend Info         0  SizeOfSegment: 2d000
   Image Base           0  Committed   e1b765b8  Based Addr   2d36f438

PTE Template:          3
   Image commit  1000000  Image Info:           0
   ProtoPtes   e1b76500

Subsection 1. @ fcdee040
   ControlArea: fcdee008  Starting Sector 0 Number Of Sectors 2
   Base Pte     e1b76500  Ptes In subsect         1 Unused Ptes          0
   Flags              15  Sector Offset           0 Protection           1
    ReadOnly SubsectionStatic

Subsection 2. @ fcdee060
   ControlArea: fcdee008  Starting Sector 2 Number Of Sectors ff
   Base Pte     e1b76504  Ptes In subsect        20 Unused Ptes          0
   Flags              35  Sector Offset           0 Protection           3
    ReadOnly SubsectionStatic

Subsection 3. @ fcdee080
   ControlArea: fcdee008  Starting Sector 101 Number Of Sectors 11
   Base Pte     e1b76584  Ptes In subsect         3 Unused Ptes          0
   Flags              55  Sector Offset           0 Protection           5
    ReadOnly SubsectionStatic

Subsection 4. @ fcdee0a0
   ControlArea: fcdee008  Starting Sector 112 Number Of Sectors 48
   Base Pte     e1b76590  Ptes In subsect         9 Unused Ptes          0
   Flags              15  Sector Offset           0 Protection           1
    ReadOnly SubsectionStatic
```

### 分析

注意到，机器中只有一个 `Winlogon` 的 `mapped view`。如果程序在运行的时候，你却看见 `0` 个 `view`，不要担心，可以看到这个文件有两个列表，另外一个就是正在使用的 `mapview`。

### 总结

这里只是简单介绍了一下内核调试器和一些你可以使用的命令。我并没有讲述所有的东西，我希望你至少知道了一些基本的东西，并将它作为一个导航。你还可以从搜索引擎中得到更多你想要的东西。我总是确保信息是准确的，但是总会有些许错误，如果你发现了，请告诉我。以后我会加上更多的关于内核调试器的信息。

你每次使用命令得到的信息，其中只有一部分是你想要的，其它的你可以不关心。如果你不清楚这些数据代表什么，不要灰心。如果你真的想知道它的意义，你可以在这些地址上设置 `ba`，然后找出谁正在使用它。
