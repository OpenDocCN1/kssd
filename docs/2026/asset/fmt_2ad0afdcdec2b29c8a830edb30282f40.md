
# [原创]替换SharedUserData

版本:1.0

作者: xIkUg/RCT/CCG          xikug.xp [at] gmail [dot] com

我常去的网站：

4dbK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3c8W2j5Y4g2Y4L8h3q4F1i4K6u0W2N6$3W2F1N6r3!0G2L8s2y4H3M7X3!0Q4x3X3g2U0L8$3@1`.

83bK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3k6U0k6$3y4Z5K9h3&6S2i4K6u0W2j5$3!0E0

http://bbs.pediy.com

b46K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4g2F1M7r3q4U0K9#2)9J5k6h3y4F1

说明：为了不至于混淆视线，本文以Win2000专业版，非3G，非PAE系统为蓝本进行讲述。其他系统稍有不同，但不在本文的讲述范围。

SharedUserData 是操作系统为每个进程提供的个共享数据结构，里面存放有很多重要的系统信息，如`TickCount`、系统时间、`SystemRoot`等。。。其在DDK定义为：

```
#define KI_USER_SHARED_DATA         0xffdf0000

#define SharedUserData  ((KUSER_SHARED_DATA * const) KI_USER_SHARED_DATA)
```

他在内核中的地址是`0xffdf0000`，操作系统通过共享映射把这个结构以只读方式映射到每个进程的`0x7ffe0000`的地方。

操作系统为每个进程提供4G虚拟空间的访问能力(空间访问能力不代表空间容量)，不过实际访问不了这么多，因为4G空间中低2G是用户空间，高2G是系统空间（非3G模式的情况下），其中低2G空间中某些地址是受系统保护的，普通用户进程也不能访问，而高2G访问需要ring0权限。

操作系统采用分页机制来映射物理内存，并会为每个进程分配一个`PT`(Pages Table)和`PD`(Pages Directory)，每个进程的`PTE`(Page Table Entry)和`PDE`(Page Directory Entry)通常映射了不同的物理页，所以不同进程的同一虚拟地址的内容是不相干的。

上面说的“通常”也有例外，那就是不同进程的`PTE`或`PDE`也可能会映射相同的物理页，如果他们映射了相同的物理页，说明他们共享了一段内存，`SharedUserData`就是这个例外，每个进程用户空间的`0x7ffe0000`都以只读方式映射到相同的物理页面上，而这个物理页面上就是`KUSER_SHARED_DATA`结构的数据，因此操作系统上的每个进程都有一个这个结构，而操作系统只需要维护这一个结构就行了。

实际上在Windows NT操作系统的实现中大量的使用了这种内存共享机制，分页机制处理也远比这个我说的复杂，具体可参考《JIURL玩玩Win2k内存篇》。

我们回到我们的主题，我们要替换指定进程的`SharedUserData`。了解了上面我说的共享机制，可能你已经知道了，使进程的`0x7ffe0000`不要映射到`0xffdf0000`指向的物理页，而是映射到我们自己的物理内存页就行了。没错。。。话是这么说，但实际上Windows的内存管理机制相当复杂，我在具体实现想法的时候碰到了许多问题，真正彻底解决这些问题花了我足足两周的时间，其中最棘手的问题的是：

1. 死锁问题

2. 内存被置换到`Standby`链表上去的问题

事出必有因，我坚信操作系统能做到的事情我们也同样能做到。通过不断的调试操作系统代码终于解决了各种问题，现在我的代码能在2000-2003的系统下跑起来了。

我们需要定义一些结构，`PTE`和`PDE`的基地址如下：

```
#define PTE_BASE    0xC0000000

#define PDE_BASE    0xC0300000
```

根据虚拟地址得到`PTE`的地址和`PDE`的地址：

```
#define GetPteAddress(va) ((PMMPTE)(((((ULONG)(va)) >> 12) << 2) + PTE_BASE))

#define GetPdeAddress(va)  ((PMMPTE)(((((ULONG)(va)) >> 22) << 2) + PDE_BASE))
```

我们需要分配一段非分页内存，必须以小页边界(4K)对齐，`0x7ffe0000`也以页对齐的：

```
ProcessPTE->p = ExAllocatePool(NonPagedPool, PAGE_SIZE)
```

现在我们就替换`SharedUserData`，并把日期锁定：

```
RtlCopyMemory(ProcessPTE->p, SharedUserData, sizeof(KUSER_SHARED_DATA));

pMySharedData = ProcessPTE->p;

RtlTimeToTimeFields((PLARGE_INTEGER)&pMySharedData->SystemTime, &TimeFields);

TimeFields.Year = 2007;

TimeFields.Month = 1;

TimeFields.Day = 1;

RtlTimeFieldsToTime(&TimeFields, (PLARGE_INTEGER)&pMySharedData->SystemTime);

pMySharedData->SystemTime.High2Time = pMySharedData->SystemTime.High1Time;
```

这样日期就被我们锁定在2007-1-1了。。。，现在我们要让时间继续走，于是建一个系统线程去更新`SystemTime`：

```
pSharedData = ProcessPTE->p;

RtlTimeToTimeFields((PLARGE_INTEGER)&SharedUserData->SystemTime, &TimeFields);

TimeFields.Year = 2007;

TimeFields.Month = 1;

TimeFields.Day = 1;

RtlTimeFieldsToTime(&TimeFields, (PLARGE_INTEGER)&pSharedData->SystemTime);

pSharedData->SystemTime.High2Time = pSharedData->SystemTime.High1Time;
```

测试一下，我以现程序有些问题，在被替换进程的窗口最小化的时候会让系统死锁，是于我中断下来，查看调用堆栈：

```
80473ae0 8046950e 00000001 82060102 000000d1 nt!RtlpBreakWithStatusInstruction
80473ae0 80069b02 00000001 82060102 000000d1 nt!KeUpdateSystemTime+0x13e
80473b64 80464b59 0000000e 00000000 00000000 hal!HalProcessorIdle+0x2
80473b68 00000000 00000000 00000000 00000000 nt!KiIdleLoop+0x10
```

系统在这里空转， 查看我们目标进程的调用堆栈：

```
be482a50 8042d87a 00cc0020 00000000 0000032c nt!KiSwapThread+0xc5
be482a78 a007dbad 00000001 00000000 be482a8c nt!KeDelayExecutionThread+0x180
be482a94 a007e005 00000001 a03106e0 01010057 win32k!UserSleep+0x2b
be482af4 a007e11a 0000032c 01010057 000000c0 win32k!xxxAnimateCaption+0x229
be482b48 a004d809 a0332cf8 00000003 a0332cf8 win32k!xxxDrawAnimatedRects+0xc8
be482c20 a00151d3 e1f32468 00000006 00010000 win32k!xxxMinMaximize+0x1fc
be482c48 a003e872 00000010 00010006 00000112 win32k!xxxShowWindow+0x147
be482c7c a008977d a0332cf8 0000f020 00000000 win32k!xxxSysCommand+0x2a6
be482cdc a00045fa a0332cf8 00000112 0000f020 win32k!xxxDefWindowProc+0xc43
be482cf0 a00045e1 a0332cf8 00000112 0000f020 win32k!xxxWrapDefWindowProc+0x15
be482d0c a0004588 a0332cf8 00000112 0000f020 win32k!NtUserfnDWORD+0x25
be482d40 80465691 00050100 00000112 0000f020 win32k!NtUserMessageCall+0x89
be482d40 77df37e7 00050100 00000112 0000f020 nt!KiSystemService+0xc4
0012fe0c 00000000 00000000 00000000 00000000 +0x77df37e7
```

`xxxAnimateCaption`之上是`UserSleep`，这看起来是一个循环，于是我们转去对`xxxAnimateCaption`进行调度，发现之所以在这里卡住，是因为我们没有更新`SharedUserData`的`TickCountLow`，而`xxxAnimateCaption`会从`0x7ffe0000`取这个值来画出动画效果，具体可看`xxxAnimateCaption`的反汇编代码，比较多，我就不列出来了。。。

当然光是一个死循环还不至于让系统死锁，我们再向上看，`xxxDrawAnimatedRects`的时候调用`GetDCEx`导致锁定一个对象，具体见函数`GreLockDisplay`的代码。而一个死循环导致了系统一直没有机会`ReleaseDC`。系统会不断的等待这个对象，所以导致了系统的死锁。

通过调试，我们发现`GreLockDisplay`锁定的对象是`0x81e1bc08`，

```
kd> dt 0x81e1bc08 _ERESOURCE -b

   +0x000 SystemResourcesList : _LIST_ENTRY [ 0x81e1bbc8 - 0x81e1bd08 ]

      +0x000 Flink            : 0x81e1bbc8 

      +0x004 Blink            : 0x81e1bd08 

   +0x008 OwnerTable       : (null) 

   +0x00c ActiveCount      : 1

   +0x00e Flag             : 0x80

   +0x010 SharedWaiters    : 0x81cf4168 

   +0x014 ExclusiveWaiters : 0x81e0c068 

   +0x018 OwnerThreads     : 

    [00] _OWNER_ENTRY

      +0x000 OwnerThread      : 0x81d0d020

      +0x004 OwnerCount       : 1

      +0x004 TableSize        : 1

    [01] 

      +0x000 OwnerThread      : 0

      +0x004 OwnerCount       : 0

      +0x004 TableSize        : 0

   +0x028 ContentionCount  : 0x5e9

   +0x02c NumberOfSharedWaiters : 0

   +0x02e NumberOfExclusiveWaiters : 3

   +0x030 Address          : (null) 

   +0x030 CreatorBackTraceIndex : 0

   +0x034 SpinLock         : 0
```

`0x81d0d020`就是我们的线程，从这个结构来看，我们的线程占有了一个对象，而还有3个等待者(`NumberOfExclusiveWaiters` : 3)，这3个等待者在`ExclusiveWaiters` : `0x81e0c068`指向的地方，

```
kd> dt 0x81e0c068 _KEVENT -b

   +0x000 Header           : _DISPATCHER_HEADER

      +0x000 Type             : 0x1 ''

      +0x001 Absolute         : 0 ''

      +0x002 Size             : 0x4 ''

      +0x003 Inserted         : 0 ''

      +0x004 SignalState      : 0

      +0x008 WaitListHead     : _LIST_ENTRY [ 0x81cf962c - 0x81da15cc ]

         +0x000 Flink            : 0x81cf962c 

         +0x004 Blink            : 0x81da15cc 
```

```
kd> dt 0x81cf962c _KWAIT_BLOCK -b

   +0x000 WaitListEntry    : _LIST_ENTRY [ 0x81e0ca6c - 0x81e0c070 ]

      +0x000 Flink            : 0x81e0ca6c 

      +0x004 Blink            : 0x81e0c070 

   +0x008 Thread           : 0x81cf95c0 

   +0x00c Object           : 0x81e0c068 

   +0x010 NextWaitBlock    : 0x81cf9674 

   +0x014 WaitKey          : 0

   +0x016 WaitType         : 1
```

```
kd> dt 0x81e0ca6c _KWAIT_BLOCK -b

   +0x000 WaitListEntry    : _LIST_ENTRY [ 0x81da15cc - 0x81cf962c ]

      +0x000 Flink            : 0x81da15cc 

      +0x004 Blink            : 0x81cf962c 

   +0x008 Thread           : 0x81e0ca00 

   +0x00c Object           : 0x81e0c068 

   +0x010 NextWaitBlock    : 0x81e0cab4 

   +0x014 WaitKey          : 0

   +0x016 WaitType         : 1
```

```
kd> dt 0x81da15cc _KWAIT_BLOCK -b

   +0x000 WaitListEntry    : _LIST_ENTRY [ 0x81e0c070 - 0x81e0ca6c ]

      +0x000 Flink            : 0x81e0c070 

      +0x004 Blink            : 0x81e0ca6c 

   +0x008 Thread           : 0x81da1560 

   +0x00c Object           : 0x81e0c068 

   +0x010 NextWaitBlock    : 0x81da1614 

   +0x014 WaitKey          : 0

   +0x016 WaitType         : 1
```

其中两个线程`0x81da1560`和`0x81cf95c0`是VMWare的关键线程（看来VMWare需要跟系统配合运行的，并不是完全模拟硬件），另一个线程`0x81e0ca00`是`csrss.exe`(csrss负责win32子系统)的关键线程。以上就是导致系统死锁的根本原因。

(windbg中可以用`!locks`来搜索系统中所有的锁，当然有时候搜索出来的结果比较多，还没有手工来得直接)

要解决这个问题，我们可采用以下方法：

1. 关闭系统的动画效果

2. 我们来更新这个值

如果采用1的方法，我难保系统的其他地方没有用`SharedUserData`的`TickCountLow`这个值来做循环因子，因此最彻底的方法是我们自己更新这个值，于是我们在我们的系统线程中加入：

```
pSharedData->TickCountLow = SharedUserData->TickCountLow;
```

现在死锁问题解决了，我们又迎来了一个新的问题：我们申请的用于替换的内存会被置换到`Standby`链表上去。当有程序要访问`0x7ffe0000`的时候就会非法访问。用Arm加壳的程序在调用`SetProcessWorkingSetSize`或程序最小化时会出现这个问题。这是因为Windows分页机制造成的，Windows会根据需要调整进程的工作集空间。
