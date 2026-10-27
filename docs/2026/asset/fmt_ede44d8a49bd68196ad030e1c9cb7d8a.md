
# [原创]获取VEH SEH VCH UEF异常的函数地址

我们知道不管是在逆向破解还是脱壳,绝大部分的反调试机制是基于异常机制的，如果我们能够知道异常处理函数的地址，那么我们也就知道了敌人的巢穴，剩下的就看我们的能力能不能够摧垮敌人的巢穴了。

我只是简单告诉你怎么找到异常函数的地址,并不会去分析异常的实现机制,因为这就会和主题不符合了.

在找地址之前我先贴出两个至关重要的结构体 `TEB` 和 `PEB`。

## `TEB` (Thread Environment Block，线程环境块)

```
nt!\_TEB
+0x000 NtTib : Ptr32 \_NT\_TIB
+0x01c EnvironmentPointer : Ptr32 Void
+0x020 ClientId : \_CLIENT\_ID
+0x028 ActiveRpcHandle : Ptr32 Void
+0x02c ThreadLocalStoragePointer : Ptr32 Void
+0x030 ProcessEnvironmentBlock : Ptr32 \_PEB
+0x034 LastErrorValue : Uint4B
+0x038 CountOfOwnedCriticalSections : Uint4B
+0x03c CsrClientThread : Ptr32 Void
+0x040 Win32ThreadInfo : Ptr32 Void
+0x044 User32Reserved : [26] Uint4B
+0x0ac UserReserved : [5] Uint4B
+0x0c0 WOW32Reserved : Ptr32 Void
+0x0c4 CurrentLocale : Uint4B
+0x0c8 FpSoftwareStatusRegister : Uint4B
+0x0cc SystemReserved1 : [54] Ptr32 Void
+0x1a4 ExceptionCode : Int4B
+0x1a8 ActivationContextStack : \_ACTIVATION\_CONTEXT\_STACK
+0x1bc SpareBytes1 : [24] UChar
+0x1d4 GdiTebBatch : \_GDI\_TEB\_BATCH
+0x6b4 RealClientId : \_CLIENT\_ID
+0x6bc GdiCachedProcessHandle : Ptr32 Void
+0x6c0 GdiClientPID : Uint4B
+0x6c4 GdiClientTID : Uint4B
+0x6c8 GdiThreadLocalInfo : Ptr32 Void
+0x6cc Win32ClientInfo : [62] Uint4B
+0x7c4 glDispatchTable : [233] Ptr32 Void
+0xb68 glReserved1 : [29] Uint4B
+0xbdc glReserved2 : Ptr32 Void
+0xbe0 glSectionInfo : Ptr32 Void
+0xbe4 glSection : Ptr32 Void
+0xbe8 glTable : Ptr32 Void
+0xbec glCurrentRC : Ptr32 Void
+0xbf0 glContext : Ptr32 Void
+0xbf4 LastStatusValue : Uint4B
+0xbf8 StaticUnicodeString : \_UNICODE\_STRING
+0xc00 StaticUnicodeBuffer : [261] Uint2B
+0xe0c DeallocationStack : Ptr32 Void
+0xe10 TlsSlots : [64] Ptr32 Void
+0xf10 TlsLinks : \_LIST\_ENTRY
+0xf18 Vdm : Ptr32 Void
+0xf1c ReservedForNtRpc : Ptr32 Void
+0xf20 DbgSsReserved : [2] Ptr32 Void
+0xf28 HardErrorsAreDisabled : Uint4B
+0xf2c Instrumentation : [16] Ptr32 Void
+0xf6c WinSockData : Ptr32 Void
+0xf70 GdiBatchCount : Uint4B
+0xf74 InDbgPrint : UChar
+0xf75 FreeStackOnTermination : UChar
+0xf76 HasFiberData : UChar
+0xf77 IdealProcessor : UChar
+0xf78 Spare3 : Uint4B
+0xf7c ReservedForPerf : Ptr32 Void
+0xf80 ReservedForOle : Ptr32 Void
+0xf84 WaitingOnLoaderLock : Uint4B
+0xf88 Wx86Thread : \_Wx86ThreadState
+0xf94 TlsExpansionSlots : Ptr32 Ptr32 Void
+0xf98 ImpersonationLocale : Uint4B
+0xf9c IsImpersonating : Uint4B
+0xfa0 NlsCache : Ptr32 Void
+0xfa4 pShimData : Ptr32 Void
+0xfa8 HeapVirtualAffinity : Uint4B
+0xfac CurrentTransactionHandle : Ptr32 Void
+0xfb0 ActiveFrame : Ptr32 \_TEB\_ACTIVE\_FRAME
+0xfb4 SafeThunkCall : UChar
+0xfb5 BooleanSpare : [3] UChar
```

## `PEB` (Process Envirorment Block，进程环境块)

```
nt!\_PEB
+0x000 InheritedAddressSpace : UChar
+0x001 ReadImageFileExecOptions : UChar
+0x002 BeingDebugged : UChar
+0x003 SpareBool : UChar
+0x004 Mutant : Ptr32 Void
+0x008 ImageBaseAddress : Ptr32 Void
+0x00c Ldr : Ptr32 \_PEB\_LDR\_DATA
+0x010 ProcessParameters : Ptr32 \_RTL\_USER\_PROCESS\_PARAMETERS
+0x014 SubSystemData : Ptr32 Void
+0x018 ProcessHeap : Ptr32 Void
+0x01c FastPebLock : Ptr32 \_RTL\_CRITICAL\_SECTION
+0x020 FastPebLockRoutine : Ptr32 Void
+0x024 FastPebUnlockRoutine : Ptr32 Void
+0x028 EnvironmentUpdateCount : Uint4B
+0x02c KernelCallbackTable : Ptr32 Void
+0x030 SystemReserved : [1] Uint4B
+0x034 AtlThunkSListPtr32 : Uint4B
+0x038 FreeList : Ptr32 \_PEB\_FREE\_BLOCK
+0x03c TlsExpansionCounter : Uint4B
+0x040 TlsBitmap : Ptr32 Void
+0x044 TlsBitmapBits : [2] Uint4B
+0x04c ReadOnlySharedMemoryBase : Ptr32 Void
+0x050 ReadOnlySharedMemoryHeap : Ptr32 Void
+0x054 ReadOnlyStaticServerData : Ptr32 Ptr32 Void
+0x058 AnsiCodePageData : Ptr32 Void
+0x05c OemCodePageData : Ptr32 Void
+0x060 UnicodeCaseTableData : Ptr32 Void
+0x064 NumberOfProcessors : Uint4B
+0x068 NtGlobalFlag : Uint4B
+0x070 CriticalSectionTimeout : \_LARGE\_INTEGER
+0x078 HeapSegmentReserve : Uint4B
+0x07c HeapSegmentCommit : Uint4B
+0x080 HeapDeCommitTotalFreeThreshold : Uint4B
+0x084 HeapDeCommitFreeBlockThreshold : Uint4B
+0x088 NumberOfHeaps : Uint4B
+0x08c MaximumNumberOfHeaps : Uint4B
+0x090 ProcessHeaps : Ptr32 Ptr32 Void
+0x094 GdiSharedHandleTable : Ptr32 Void
+0x098 ProcessStarterHelper : Ptr32 Void
+0x09c GdiDCAttributeList : Uint4B
+0x0a0 LoaderLock : Ptr32 Void
+0x0a4 OSMajorVersion : Uint4B
+0x0a8 OSMinorVersion : Uint4B
+0x0ac OSBuildNumber : Uint2B
+0x0ae OSCSDVersion : Uint2B
+0x0b0 OSPlatformId : Uint4B
+0x0b4 ImageSubsystem : Uint4B
+0x0b8 ImageSubsystemMajorVersion : Uint4B
+0x0bc ImageSubsystemMinorVersion : Uint4B
+0x0c0 ImageProcessAffinityMask : Uint4B
+0x0c4 GdiHandleBuffer : [34] Uint4B
+0x14c PostProcessInitRoutine : Ptr32 void
+0x150 TlsExpansionBitmap : Ptr32 Void
+0x154 TlsExpansionBitmapBits : [32] Uint4B
+0x1d4 SessionId : Uint4B
+0x1d8 AppCompatFlags : \_ULARGE\_INTEGER
+0x1e0 AppCompatFlagsUser : \_ULARGE\_INTEGER
+0x1e8 pShimData : Ptr32 Void
+0x1ec AppCompatInfo : Ptr32 Void
+0x1f0 CSDVersion : \_UNICODE\_STRING
+0x1f8 ActivationContextData : Ptr32 Void
+0x1fc ProcessAssemblyStorageMap : Ptr32 Void
+0x200 SystemDefaultActivationContextData : Ptr32 Void
+0x204 SystemAssemblyStorageMap : Ptr32 Void
+0x208 MinimumStackCommit : Uint4B
```

## SEH 异常函数地址

`fs:[0]` 指向 `TEB` 结构的开始，其中第一个字段是一个结构体 `NtTib`，`NtTib` 的结构为：

```
typedef struct \_NT\_TIB //sizeof 1ch
{
00h struct \_EXCEPTION\_REGISTRATION\_RECORD \*ExceptionList;
04h PVOID StackBase;
08h PVOID StackLimit;
0ch PVOID SubSystemTib;
10h union {
PVOID FiberData;
DWORD Version;
};
14h PVOID ArbitraryUserPointer;
18h struct \_NT\_TIB \*Self;
}NT\_TIB;
```

其中 `NtTib` 的第一个字段 `ExceptionList` 是一个指针指向记录 SEH 的信息的结构体 `_EXCEPTION_REGISTRATION_RECORD`，该结构体结构为：

```
typedef struct \_EXCEPTION\_REGISTRATION\_RECORD
{
struct \_EXCEPTION\_REGISTRATION\_RECORD \*Next；//指向下一个SEH异常处理函数
PEXCEPTION\_ROUTINE Handler;//SEH 异常处理函数
}
```

看来这个结构是一个单向链表，当 `Next = 0xFFFFFFFF` 时，表示链表结束。

总结一下找 SEH：

1. 从 `FS:[0]` 得到 `ExceptionList`
2. `ExceptionList->Handler` 获取一个 SEH 异常处理函数地址
3. `ExceptionList->Next` 如果不是 `0xFFFFFFFF` 获取下一个 `ExceptionList`，重复第 2 步

## 寻找VEH VCH异常的函数地址

我创建一个 demo：

```
#include "stdafx.h"
#include <windows.h>
DWORD d;
LONG WINAPI VCHCallBack(PEXCEPTION\_POINTERS ExceptionInfo)
{
ExceptionInfo->ContextRecord->Eax = (DWORD)&d;
return EXCEPTION\_CONTINUE\_EXECUTION;
}
LONG WINAPI VEHCallBack(PEXCEPTION\_POINTERS ExceptionInfo)
{
ExceptionInfo->ContextRecord->Eax = (DWORD)&d;
return EXCEPTION\_CONTINUE\_EXECUTION;
}
int main()
{
AddVectoredExceptionHandler(0, VEHCallBack);
AddVectoredContinueHandler(0, VCHCallBack);
\_\_asm
{
mov eax,0
mov [eax],0
}
getchar();
return 0;
}
```

用 OD 附加查看 `main` 函数：

![VEH VCH调用](upload/attach/202002/793792_P8WAS84H9YG2TJZ.png)

我们可以看到添加 VEH、VCH 调用的是：

- `RtlAddVectoredExceptionHandler`
- `RtlAddVectoredContinueHandler`

用 IDA 打开 `ntdll.dll` 查看这两个函数：

![VCH](upload/attach/201804/793792_FPVYWVMQA2DC4RT.png)

![VCH](upload/attach/201804/793792_WMDHNJDAB7P4N77.png)

这两个函数最终都调用了 `RtlpAddVectoredHandler`。

这个函数有 3 个参数，其中第 3 个参数代表是 VEH 还是 VCH。

可以看到 VEH 时 `RtlpAddVectoredHandler` 的第 3 个参数为 `0`。VCH 时 `RtlpAddVectoredHandler` 的第 3 个参数为 `1`。

继续用 IDA 查看 `RtlpAddVectoredHandler` 函数。
