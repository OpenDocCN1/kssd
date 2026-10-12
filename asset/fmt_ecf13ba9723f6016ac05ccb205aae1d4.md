
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
