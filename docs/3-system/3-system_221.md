
# 简单了解 SEH 机制

> 原文：[`www.pediy.com/kssd/pediy12/135462.html`](https://www.pediy.com/kssd/pediy12/135462.html)

在反汇编分析中，有的时候经常会看到 `FS:[0]`，那这个指令作什么用呢，现在就来了解它。

通常 `FS:[0]` 指向的是 `TEB` 结构，`TEB`（`Thread Environment Block`，线程环境块）。

`TEB` 结构又存放着什么，接着往下看：

```
0x000 NtTib                 : _NT_TIB
   +0x01c EnvironmentPointer : Ptr32 Void
   +0x020 ClientId         : _CLIENT_ID
   +0x028 ActiveRpcHandle  : Ptr32 Void
   +0x02c ThreadLocalStoragePointer : Ptr32 Void
   +0x030 ProcessEnvironmentBlock : Ptr32 _PEB
   +0x034 LastErrorValue   : Uint4B
   +0x038 CountOfOwnedCriticalSections : Uint4B
   +0x03c CsrClientThread  : Ptr32 Void
   +0x040 Win32ThreadInfo  : Ptr32 Void
   +0x044 User32Reserved   : [26] Uint4B
   +0x0ac UserReserved     : [5] Uint4B
   +0x0c0 WOW32Reserved    : Ptr32 Void
   +0x0c4 CurrentLocale    : Uint4B
   +0x0c8 FpSoftwareStatusRegister : Uint4B
   +0x0cc SystemReserved1  : [54] Ptr32 Void
   +0x1a4 ExceptionCode    : Int4B
   +0x1a8 ActivationContextStack : _ACTIVATION_CONTEXT_STACK
   +0x1bc SpareBytes1      : [24] UChar
   +0x1d4 GdiTebBatch      : _GDI_TEB_BATCH
   +0x6b4 RealClientId     : _CLIENT_ID
   +0x6bc GdiCachedProcessHandle : Ptr32 Void
   +0x6c0 GdiClientPID     : Uint4B
   +0x6c4 GdiClientTID     : Uint4B
   +0x6c8 GdiThreadLocalInfo : Ptr32 Void
   +0x6cc Win32ClientInfo  : [62] Uint4B
   +0x7c4 glDispatchTable  : [233] Ptr32 Void
   +0xb68 glReserved1      : [29] Uint4B
   +0xbdc glReserved2      : Ptr32 Void
   +0xbe0 glSectionInfo    : Ptr32 Void
   +0xbe4 glSection        : Ptr32 Void
   +0xbe8 glTable          : Ptr32 Void
   +0xbec glCurrentRC      : Ptr32 Void
   +0xbf0 glContext        : Ptr32 Void
   +0xbf4 LastStatusValue  : Uint4B
   +0xbf8 StaticUnicodeString : _UNICODE_STRING
   +0xc00 StaticUnicodeBuffer : [261] Uint2B
   +0xe0c DeallocationStack : Ptr32 Void
   +0xe10 TlsSlots         : [64] Ptr32 Void
   +0xf10 TlsLinks         : _LIST_ENTRY
   +0xf18 Vdm              : Ptr32 Void
   +0xf1c ReservedForNtRpc : Ptr32 Void
   +0xf20 DbgSsReserved    : [2] Ptr32 Void
   +0xf28 HardErrorsAreDisabled : Uint4B
   +0xf2c Instrumentation  : [16] Ptr32 Void
   +0xf6c WinSockData      : Ptr32 Void
   +0xf70 GdiBatchCount    : Uint4B
   +0xf74 InDbgPrint       : UChar
   +0xf75 FreeStackOnTermination : UChar
   +0xf76 HasFiberData     : UChar
   +0xf77 IdealProcessor   : UChar
   +0xf78 Spare3           : Uint4B
   +0xf7c ReservedForPerf  : Ptr32 Void
   +0xf80 ReservedForOle   : Ptr32 Void
   +0xf84 WaitingOnLoaderLock : Uint4B
   +0xf88 Wx86Thread       : _Wx86ThreadState
   +0xf94 TlsExpansionSlots : Ptr32 Ptr32 Void
   +0xf98 ImpersonationLocale : Uint4B
   +0xf9c IsImpersonating  : Uint4B
   +0xfa0 NlsCache         : Ptr32 Void
   +0xfa4 pShimData        : Ptr32 Void
   +0xfa8 HeapVirtualAffinity : Uint4B
   +0xfac CurrentTransactionHandle : Ptr32 Void
   +0xfb0 ActiveFrame      : Ptr32 _TEB_ACTIVE_FRAME
   +0xfb4 SafeThunkCall    : UChar
   +0xfb5 BooleanSpare     : [3] UChar
```

接着看 `NT_TIB` 结构：

```
   +0x000ExceptionList    : Ptr32 _EXCEPTION_REGISTRATION_RECORD
   +0x004 StackBase        : Ptr32 Void
   +0x008 StackLimit       : Ptr32 Void
   +0x00c SubSystemTib     : Ptr32 Void
   +0x010 FiberData        : Ptr32 Void
   +0x010 Version          : Uint4B
   +0x014 ArbitraryUserPointer : Ptr32 Void
   +0x018 Self             : Ptr32 _NT_TIB
```

实际上 `FS:[0]` 指向的是 `ERR` 结构。

而 `TIB` 结构指向的是 `_EXCEPTION_REGISTRATION_RECORD` 结构：

```c
typedef struct _EXCEPTION_REGISTRATION_RECORD
{
  struct _EXCEPTION_REGISTRATION_RECORD* prev;
  DWORD                                 handler;
} _EXCEPTION_REGISTRATION_RECORD;
```

简称 `ERR` 结构，它指向的是个异常处理表，以单链表形式存储。

![`bbs.pediy.com/upload/2006/41/image/seh11.GIF`](img/32ff6f590d48467fd08c57a76b13bdff.jpg)

更多异常信息可参考 `EXCEPTION_RECORD` 结构。

这里做个简单的 `SEH` 实验。

代码：

```c
__try
  {
    ::MessageBoxA(NULL, "test", NULL, NULL);
  }
  __except(EXCEPTION_BREAKPOINT) //下断后执行回调函数
  {
       ::MessageBoxA(NULL, "aa", NULL, NULL);
  }
```

接着以反汇编角度来看会发生什么。

（注：缺图，请联系原作者）图中了解到异常回调函数并不会去调用函数，而是跳到函数的下一地址。

在谈 `SEH` 机制的作用。

`SEH` 机制对于反追踪是非常有效的，一般在分析时都是习惯用 `F2` 下断点及单步下断。

`SEH` 正是利用这种机制，让追踪者在不知不觉跳到异常处理函数中，其中也可以加些花指令。

也可以配合硬断处理来强制引发异常。

更多信息可参考《加密与解密 3》第十一章。
