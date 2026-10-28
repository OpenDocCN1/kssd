
# 玩玩微点——APCKILL

最近学习了APC相关的一些知识 (汗！古老到牛们都可能忘记了它的存在~\_~)，发现special kernal apc还是能做些事情,比如结束某个进程（嘿嘿^\_^）,当然是在R0了, “不要问我都能加载驱动了,什么不能干?因为我太TMD能\*\*了,呸”某大牛的话，同样适用于本贴!-\_-

选择微点做试验，并非偶然。微点主动防御软件是目前主防中的牛B者，她太X了，真正达到HOOK无极限，搞定她，能学到一些XX技巧。貌似网上流行的ARK软件和进程查杀软件不能直接结束她。（要是很容易放倒，也许就不会有太多人玩儿她了。。嘿嘿^\_^！）

## 预备知识

PE结构；`EPROCESS`、`ETHREAD`结构及获取方法；APC相关知识（俺学习总结的文档会放后面，愿与大伙交流学习）。

## 看点

EAT HOOK 恢复方法；special kernal apc使用方法；获取内核未导出函数地址方法。

## 基本流程

1. 恢复`KeInsertQueueApc`的EAT HOOK；
2. 暴搜进程获取微点进程，根据微点进程枚举所有线程；
3. 通过`PsTerminateSystemThread`函数查找到`PspTerminateThreadByPointer`，通过`PspTerminateThreadByPointer`找到`PspExitThread`，编写APC例程，例程内直接调用搜到的`PspExitThread`函数-\_-；
4. 调用已恢复`KeInsertQueueApc`对所有微点线程插APC；
5. 加载驱动，看着最新版免费试用90天的微点主防在XP SP2平台上消失。

## 附注

代码写得很乱，想到哪写到哪。如果在阅读代码时出现不适切勿找偶！正常情况加载驱动，微点退出，如出现BSOD等现象，切勿找偶！

### 以下为俺机子上加载驱动后，windbg输出信息：

```
baseadress:400000
the functionName = KeInsertQueueApc
the functionaddress=0x0040E411
  the functionrav =0x0000E411
the modulefullname is \WINDOWS\system32\ntoskrnl.exe
the database is 0x804D8000
the functionName = KeInsertQueueApc
the functionaddress=0xF9847B80
  the functionrav =0x7936FB80
the hook address at 0x7936fb80
the pkeinsertqueueapc is 0x804e6411the keinsertqueueapc is 0xf9847b80restore the hook address at 0x0000e411
the Thread is 0x81CEBAD0
the Thread 0x81cebad0 is Terminated.
the Thread is 0x81CFBAE0
the Thread 0x81cfbae0 is Terminated.
the Thread is 0x81D2E8E8
the Thread 0x81d2e8e8 is Terminated.
the Thread is 0x81C35DA8
the Thread 0x81c35da8 is Terminated.
the Thread is 0x81C14DA8
the Thread 0x81c14da8 is Terminated.
the Thread is 0x81E23B30
the Thread 0x81e23b30 is Terminated.
the Thread is 0x81E4D9E8
the Thread 0x81e4d9e8 is Terminated.
the Thread is 0x81E444E0
the Thread 0x81e444e0 is Terminated.
the Thread is 0x81E23770
the Thread 0x81e23770 is Terminated.
the Thread is 0x81E234F8
the Thread 0x81e234f8 is Terminated.
the Thread is 0x81DCA348
the Thread 0x81dca348 is Terminated.
the Thread is 0x81D93598
the Thread 0x81d93598 is Terminated.
the Thread is 0x81C6E020
the Thread 0x81c6e020 is Terminated.
the Thread is 0x81C6ED20
the Thread 0x81c6ed20 is Terminated.
the Thread is 0x81BB0020
the Thread 0x81bb0020 is Terminated.
the Thread is 0x81BB0320
the Thread 0x81bb0320 is Terminated.
the Thread is 0x81C6A020
the Thread 0x81c6a020 is Terminated.
the Thread is 0x81E41DA8
the Thread 0x81e41da8 is Terminated.
the Thread is 0x81E3F450
the Thread 0x81e3f450 is Terminated.
the Thread is 0x81E3EB38
the Thread 0x81e3eb38 is Terminated.
the Thread is 0x81E3E8C0
the Thread 0x81e3e8c0 is Terminated.
the Thread is 0x81E3E510
the Thread 0x81e3e510 is Terminated.
the Thread is 0x81E3DB38
the Thread 0x81e3db38 is Terminated.
the Thread is 0x81E3D8C0
the Thread 0x81e3d8c0 is Terminated.
the Thread is 0x81E3D648
the Thread 0x81e3d648 is Terminated.
the Thread is 0x81E3A258
the Thread 0x81e3a258 is Terminated.
the Thread is 0x81E39020
the Thread 0x81e39020 is Terminated.
the Thread is 0x81E39DA8
the Thread 0x81e39da8 is Terminated.
the Thread is 0x81E39AD0
the Thread 0x81e39ad0 is Terminated.
the Thread is 0x81E38020
the Thread 0x81e38020 is Terminated.
the Thread is 0x81E383A0
the Thread 0x81e383a0 is Terminated.
the Thread is 0x81E37020
the Thread 0x81e37020 is Terminated.
the Thread is 0x81E37DA8
the Thread 0x81e37da8 is Terminated.
the Thread is 0x81E36C08
the Thread 0x81e36c08 is Terminated.
the Thread is 0x81E364D8
the Thread 0x81e364d8 is Terminated.
the Thread is 0x81E355B0
the Thread 0x81e355b0 is Terminated.
the Thread is 0x81E2C020
the Thread 0x81e2c020 is Terminated.
the Thread is 0x81E2CD88
the Thread 0x81e2cd88 is Terminated.
the Thread is 0x81E2C918
the Thread 0x81e2c918 is Terminated.
the Thread is 0x81E23020
the Thread 0x81e23020 is Terminated.
the Thread is 0x81E23DA8
the Thread 0x81e23da8 is Terminated.
the Thread is 0x81E18560
the Thread 0x81e18560 is Terminated.
the Thread is 0x81D91950
the Thread 0x81d91950 is Terminated.
the Thread is 0x81D6C020
the Thread 0x81d6c020 is Terminated.
the Thread is 0x81D5BDA8
the Thread 0x81d5bda8 is Terminated.
```

---

正文开始

---

## APC机制学习

1. APC允许用户程序和系统元件在一个进程的地址空间内某个线程的上下文中执行代码。
2. I/O管理器使用APC来完成一个线程发起的异步的I/O操作。例如：当一个设备驱动调用`IoCompleteRequest`来通知I/O管理器，它已经结束处理一个异步I/O请求时，I/O管理器排队一个apc到发起请求的线程。然后线程在一个较低IRQL级别，来执行APC. APC的作用是从系统空间拷贝I/O操作结果和状态信息到线程虚拟内存空间的一个缓冲中。
3. 使用APC可以得到或者设置一个线程的上下文和挂起线程的执行。

## APC对象

在NT中，有两种类型的APC：用户模式和内核模式。用户APCs运行在用户模式下目标线程当前上下文中，并且需要从目标线程得到许可来运行。特别是，用户模式的APCs需要目标线程处在alertable等待状态才能被成功的调度执行。对于用户模式下，可以调用函数`SleepEx`, `SignalObjectAndWait`, `WaitForSingleObjectEx`, `WaitForMultipleObjectsEx`, `MsgWaitForMultipleObjectsEx` 都可以使目标线程处于alertable等待状态，从而让用户模式APC执行,原因是这些函数最终都是调用了内核中的`KeWaitForSingleObject`, `KeWaitForMultipleObjects`, `KeWaitForMutexObject`, `KeDelayExecutionThread`等函数。另外通过调用一个未公开的alert-test服务`KeTestAlertThread`，用户线程可以使用户模式APC执行。

当一个用户模式APC被投递到一个线程，调用上面的等待函数，如果返回等待状态`STATUS_USER_APC`，在返回用户模式时，内核转去控制APC例程，当APC例程完成后，再继续线程的执行.

内核模式APC执行在内核模式下，可以被划分为常规的和特殊的两类。当APC被投递到一个特殊的线程，特殊的内核模式APC不需要从线程得到许可来运行。然而，常规的内核模式APC在他们成功执行前，需要有特定的环境。此外，特殊的内核APC被尽可能快地执行，既只要`APC_LEVEL`级上有可调度的活动。在很多情况下，特殊的内核APC甚至能唤醒阻塞的线程。普通的内核APC仅在所有特殊APC都被执行完，并且目标线程仍在运行，同时该线程中也没有其它内核模式APC正执行时才执行。用户模式APC在所有内核模式APC执行完后才执行，并且仅在目标线程有alertable属性时才执行。

一个等待执行的APC都存在于一个线程执行体，由内核管理的队列中。系统中的每一个线程都包含两个APC队列，一个是为用户模式APC,另一个是为内核模式APC的。

NT通过一个成为`KAPC`的内核控制对象来描述一个ＡＰＣ．

Windbg中`kapc`结构如下：

```
kd> dt -v -r _kapc
Matched ntdll!_KAPC
ntdll!_KAPC
struct _KAPC, 14 elements, 0x30 bytes
   +0x000 Type             : Int2B
   +0x002 Size             : Int2B
   +0x004 Spare0           : Uint4B
   +0x008 Thread           : Ptr32 to struct _KTHREAD, 73 elements, 0x1c0 bytes
      +0x000 Header           : struct _DISPATCHER_HEADER, 6 elements, 0x10 bytes
         +0x000 Type             : UChar
         +0x001 Absolute         : UChar
         +0x002 Size             : UChar
         +0x003 Inserted         : UChar
         +0x004 SignalState      : Int4B
         +0x008 WaitListHead     : struct _LIST_ENTRY, 2 elements, 0x8 bytes
      +0x010 MutantListHead   : struct _LIST_ENTRY, 2 elements, 0x8 bytes
         +0x000 Flink            : Ptr32 to struct _LIST_ENTRY, 2 elements, 0x8 bytes
         +0x004 Blink            : Ptr32 to struct _LIST_ENTRY, 2 elements, 0x8 bytes
      +0x018 InitialStack     : Ptr32 to Void
      +0x01c StackLimit       : Ptr32 to Void
      +0x020 Teb              : Ptr32 to Void
      +0x024 TlsArray         : Ptr32 to Void
      +0x028 KernelStack      : Ptr32 to Void
      +0x02c DebugActive      : UChar
      +0x02d State            : UChar
      +0x02e Alerted          : [2] UChar
      +0x030 Iopl             : UChar
      +0x031 NpxState         : UChar
      +0x032 Saturation       : Char
      +0x033 Priority         : Char
      +0x034 ApcState         : struct _KAPC_STATE, 5 elements, 0x18 bytes
         +0x000 ApcListHead      : [2] struct _LIST_ENTRY, 2 elements, 0x8 bytes
         +0x010 Process          : Ptr32 to struct _KPROCESS, 29 elements, 0x6c bytes
         +0x014 KernelApcInProgress : UChar
         +0x015 KernelApcPending : UChar
         +0x016 UserApcPending   : UChar
      +0x04c ContextSwitches  : Uint4B
      +0x050 IdleSwapBlock    : UChar
      +0x051 Spare0           : [3] UChar
      +0x054 WaitStatus       : Int4B
      +0x058 WaitIrql         : UChar
      +0x059 WaitMode         : Char
      +0x05a WaitNext         : UChar
      +0x05b WaitReason       : UChar
      +0x05c WaitBlockList    : Ptr32 to struct _KWAIT_BLOCK, 6 elements, 0x18 bytes
         +0x000 WaitListEntry    : struct _LIST_ENTRY, 2 elements, 0x8 bytes
         +0x008 Thread           : Ptr32 to struct _KTHREAD, 73 elements, 0x1c0 bytes
         +0x00c Object           : Ptr32 to Void
         +0x010 NextWaitBlock    : Ptr32 to struct _KWAIT_BLOCK, 6 elements, 0x18 bytes
         +0x014 WaitKey          : Uint2B
```
