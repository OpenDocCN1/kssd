
```nasm
PAGE:004AD9AC mov al, [esi+\_KTHREAD.PreviousMode]
PAGE:004AD9B2 push 0 ; HandleInformation
PAGE:004AD9B4 mov [ebp+AccessMode], al
PAGE:004AD9B7 lea eax, [ebp+Object]
PAGE:004AD9BA push eax ; Object
PAGE:004AD9BB push dword ptr [ebp+AccessMode] ; AccessMode
PAGE:004AD9BE push \_PsThreadType ; ObjectType
PAGE:004AD9C4 push 1 ; DesiredAccess
PAGE:004AD9C6 push [ebp+ThreadHandle] ; Handle
PAGE:004AD9C9 call \_ObReferenceObjectByHandle@24 ; ObReferenceObjectByHandle(x,x,x,x,x,x)
PAGE:004AD9CE mov edi, eax
PAGE:004AD9D0 test edi, edi
PAGE:004AD9D2 jl short loc\_4AD9F1
```

接着在调用`PspTerminateThreadPointer`来结束线程

```nasm
PAGE:004AD9D4 mov ebx, [ebp+Object]
PAGE:004AD9D7 cmp ebx, esi
PAGE:004AD9D9 jz loc\_52E6BA
PAGE:004AD9DF push [ebp+ExitStatus]
PAGE:004AD9E2 push ebx ; Response
PAGE:004AD9E3 call \_PspTerminateThreadByPointer@8 ; PspTerminateThreadByPointer(x,x)
PAGE:004AD9E8 mov ecx, ebx ; Object
PAGE:004AD9EA mov edi, eax
PAGE:004AD9EC call @ObfDereferenceObject@4
```

### B.`PsTerminateSystemThread`

`PsTerminateSystemThread`函数则非常简单，它就直接取出线程对象以后调用函数结束线程

```nasm
PAGE:004AD44D ; NTSTATUS \_\_stdcall PsTerminateSystemThread(NTSTATUS ExitStatus)
PAGE:004AD44D public \_PsTerminateSystemThread@4
PAGE:004AD44D \_PsTerminateSystemThread@4 proc near ; CODE XREF: InbvRotateGuiBootDisplay(x)+4A↑p
PAGE:004AD44D ; ExpWorkerThread(x)+4A96C↑p ...
PAGE:004AD44D
PAGE:004AD44D ExitStatus = dword ptr 8
PAGE:004AD44D
PAGE:004AD44D ; FUNCTION CHUNK AT PAGE:0052E6CB SIZE 0000000A BYTES
PAGE:004AD44D
PAGE:004AD44D mov edi, edi
PAGE:004AD44F push ebp
PAGE:004AD450 mov ebp, esp
PAGE:004AD452 mov eax, large fs:124h ; 将当前线程赋给eax
PAGE:004AD458 test byte ptr [eax+\_ETHREAD.\_\_\_u24.CrossThreadFlags], 10h
PAGE:004AD45F jz loc\_52E6CB
PAGE:004AD465 push [ebp+ExitStatus]
PAGE:004AD468 push eax ; Response
PAGE:004AD469 call \_PspTerminateThreadByPointer@8 ; PspTerminateThreadByPointer(x,x)
PAGE:004AD46E
PAGE:004AD46E loc\_4AD46E: ; CODE XREF: PsTerminateSystemThread(x)+81283↓j
PAGE:004AD46E pop ebp
PAGE:004AD46F retn 4
PAGE:004AD46F \_PsTerminateSystemThread@4
```

### C.`PspTermiateThreadByPointer`

该函数首先判断结束的线程的`CrossTheadFlags`是否等于0x40，以及要结束的线程是否是当前线程对象，如果都是的话就会调用`PspExitThread`来结束线程

```nasm
PAGE:0049EB6F ; int \_\_stdcall PspTerminateThreadByPointer(PETHREAD Thread, NTSTATUS ExitStatus)
PAGE:0049EB6F \_PspTerminateThreadByPointer@8 proc near
PAGE:0049EB6F ; CODE XREF: PspSystemThreadStartup(x,x)+3B↓p
PAGE:0049EB6F ; PsTerminateSystemThread(x)+1C↓p ...
PAGE:0049EB6F
PAGE:0049EB6F Interval = LARGE\_INTEGER ptr -0Ch
PAGE:0049EB6F var\_4 = dword ptr -4
PAGE:0049EB6F Thread = byte ptr 8
PAGE:0049EB6F ExitStatus = dword ptr 0Ch
PAGE:0049EB6F mov edi, edi
PAGE:0049EB71 push ebp
PAGE:0049EB72 mov ebp, esp
PAGE:0049EB74 sub esp, 0Ch
PAGE:0049EB77 or dword ptr [ebp-8], 0FFFFFFFFh
PAGE:0049EB7B push esi
PAGE:0049EB7C push edi
PAGE:0049EB7D mov edi, dword ptr [ebp+Thread] ; 将线程对象赋给edi
PAGE:0049EB80 lea esi, [edi+248h] ; 将CrossThreadFlags赋给edi
PAGE:0049EB86 test byte ptr [esi], 40h
PAGE:0049EB89 mov dword ptr [ebp+Interval], 0FFF0BDC0h
PAGE:0049EB90 jnz loc\_52E5AF
PAGE:0049EB96
PAGE:0049EB96 loc\_49EB96: ; CODE XREF: PspTerminateThreadByPointer(x,x)+8FA57↓j
PAGE:0049EB96 mov eax, large fs:124h ; 将当前线程赋给eax
PAGE:0049EB9C cmp edi, eax ; 判断要结束的线程是否是当前线程
PAGE:0049EB9E jnz loc\_4B7CE7
PAGE:0049EBA4 xor eax, eax
PAGE:0049EBA6 inc eax
PAGE:0049EBA7 lock or [esi], eax
PAGE:0049EBAA push [ebp+ExitStatus]
PAGE:0049EBAD call \_PspExitThread@4 ; PspExitThread(x)
PAGE:0049EBB2 nop
PAGE:0049EBB3 nop
PAGE:0049EBB4 nop
PAGE:0049EBB5 nop
PAGE:0049EBB6 nop
PAGE:0049EBB6 \_PspTerminateThreadByPointer@8
```

> **下篇:[Windows内核学习笔记之线程（中）](https://bbs.pediy.com/thread-270896.htm)**

# 一.线程基本概念

在操作系统中，进程只是提供资源，它并没有执行指令的作用。真正来执行指令的是线程，线程是一个指令执行序列，它可以直接访问所属进程中的资源。每个进程至少要有一个线程，这样才可以使用进程的资源来执行各种指令，而每个线程在任一时刻一定属于某一特定的进程。

线程不仅仅是一个控制流，它还有更多的内容。线程的调用栈记录了它作为控制流的状态信息，包括每一层函数调用和返回指令地址。线程一定隶属于某个进程，其控制流可以访问这个进程中的资源，包括所有的内存数据以及系统分配给此进程的其他资源。一个进程可以有多个线程，由于这些线程隶属于同一进程，所以它们之间相互通信要方便很多，毕竟几乎所有的资源（不是全部）对它们来说都是共享的。因此，线程概念的引入有时候也称为轻量进程。

# 二.线程数据结构

为了保存线程的各种信息，在用户层和内核层都有相应的数据结构用来存储线程的信息。

## 1.用户层数据结构

与描述进程用户空间信息的`PEB`类型，NT内核定义了线程环境块(`TEB`)来描述线程的用户空间信息，包括用户栈，异常处理，错误码，线程局部存储等，该结构定义如下：

```text
kd> dt _TEB
nt!_TEB
   +0x000 NtTib            : _NT_TIB
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

| **偏移** | **名称** | **作用** |
| --- | --- | --- |
| 0x000 | `NtTib` | 包含异常，线程栈等信息 |
| 0x020 | `ClientId` | 线程所属进程的PID |
| 0x030 | `ProcessEnvironmentPointer` | 保存了进程`PEB`地址 |

其中`NtTib`的结构如下：
