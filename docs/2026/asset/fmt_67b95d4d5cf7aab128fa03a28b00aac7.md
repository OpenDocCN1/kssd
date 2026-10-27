
### _KPRCB 结构体

```C++
kd> dt \_KPRCB
ntdll!\_KPRCB
+0x000 MinorVersion : Uint2B
+0x002 MajorVersion : Uint2B
+0x004 CurrentThread : Ptr32 \_KTHREAD // fs:[0x124]
+0x008 NextThread : Ptr32 \_KTHREAD
+0x00c IdleThread : Ptr32 \_KTHREAD
... ...
```

### `_KTHREAD`结构体

```c++
kd> dt \_KTHREAD
ntdll!\_KTHREAD
+0x000 Header : \_DISPATCHER\_HEADER
+0x010 MutantListHead : \_LIST\_ENTRY
+0x018 InitialStack : Ptr32 Void
+0x01c StackLimit : Ptr32 Void
+0x020 Teb : Ptr32 Void
+0x024 TlsArray : Ptr32 Void
+0x028 KernelStack : Ptr32 Void
+0x02c DebugActive : UChar
+0x02d State : UChar
+0x02e Alerted : [2] UChar
+0x030 Iopl : UChar
+0x031 NpxState : UChar
+0x032 Saturation : Char
+0x033 Priority : Char
+0x034 ApcState : \_KAPC\_STATE // \_KAPC\_STATE 结构体
+0x04c ContextSwitches : Uint4B
... ...
```

### `_KAPC_STATE`结构体

```c++
kd> dt \_KAPC\_STATE
ntdll!\_KAPC\_STATE
+0x000 ApcListHead : [2] \_LIST\_ENTRY
+0x010 Process : Ptr32 \_KPROCESS // \_KTHREAD + 0x44 = \_KPROCESS
+0x014 KernelApcInProgress : UChar
+0x015 KernelApcPending : UChar
+0x016 UserApcPending : UChar
```

### `_EPROCESS`结构体

```c++
kd> dt \_eprocess
ntdll!\_EPROCESS
+0x000 Pcb : \_KPROCESS
+0x06c ProcessLock : \_EX\_PUSH\_LOCK
+0x070 CreateTime : \_LARGE\_INTEGER
+0x078 ExitTime : \_LARGE\_INTEGER
+0x080 RundownProtect : \_EX\_RUNDOWN\_REF
+0x084 UniqueProcessId : Ptr32 Void
+0x088 ActiveProcessLinks : \_LIST\_ENTRY // 保存进程的双线循环链表结构体，与PEB中的ldr 结构体一样
+0x090 QuotaUsage : [3] Uint4B
+0x09c QuotaPeak : [3] Uint4B
+0x0a8 CommitCharge : Uint4B
+0x0ac PeakVirtualSize : Uint4B
+0x0b0 VirtualSize : Uint4B
+0x0b4 SessionProcessLinks : \_LIST\_ENTRY
+0x0bc DebugPort : Ptr32 Void
+0x0c0 ExceptionPort : Ptr32 Void
+0x0c4 ObjectTable : Ptr32 \_HANDLE\_TABLE
+0x0c8 Token : \_EX\_FAST\_REF
+0x0cc WorkingSetLock : \_FAST\_MUTEX
+0x0ec WorkingSetPage : Uint4B
+0x0f0 AddressCreationLock : \_FAST\_MUTEX
+0x110 HyperSpaceLock : Uint4B
+0x114 ForkInProgress : Ptr32 \_ETHREAD
+0x118 HardwareTrigger : Uint4B
+0x11c VadRoot : Ptr32 Void
+0x120 VadHint : Ptr32 Void
+0x124 CloneRoot : Ptr32 Void
+0x128 NumberOfPrivatePages : Uint4B
+0x12c NumberOfLockedPages : Uint4B
+0x130 Win32Process : Ptr32 Void
+0x134 Job : Ptr32 \_EJOB
+0x138 SectionObject : Ptr32 Void
+0x13c SectionBaseAddress : Ptr32 Void
+0x140 QuotaBlock : Ptr32 \_EPROCESS\_QUOTA\_BLOCK
+0x144 WorkingSetWatch : Ptr32 \_PAGEFAULT\_HISTORY
+0x148 Win32WindowStation : Ptr32 Void
+0x14c InheritedFromUniqueProcessId : Ptr32 Void
+0x150 LdtInformation : Ptr32 Void
+0x154 VadFreeHint : Ptr32 Void
+0x158 VdmObjects : Ptr32 Void
+0x15c DeviceMap : Ptr32 Void
+0x160 PhysicalVadList : \_LIST\_ENTRY
+0x168 PageDirectoryPte : \_HARDWARE\_PTE\_X86
+0x168 Filler : Uint8B
+0x170 Session : Ptr32 Void
+0x174 ImageFileName : [16] UChar // 进程名称
+0x184 JobLinks : \_LIST\_ENTRY
+0x18c LockedPagesList : Ptr32 Void
+0x190 ThreadListHead : \_LIST\_ENTRY
+0x198 SecurityPort : Ptr32 Void
+0x19c PaeTop : Ptr32 Void
+0x1a0 ActiveThreads : Uint4B
+0x1a4 GrantedAccess : Uint4B
+0x1a8 DefaultHardErrorProcessing : Uint4B
+0x1ac LastThreadExitStatus : Int4B
+0x1b0 Peb : Ptr32 \_PEB // 3环的PEB地址
+0x1b4 PrefetchTrace : \_EX\_FAST\_REF
... ...
```

### EPROCESS断链源码

```c++
#include <ntddk.h>
NTSTATUS DriverEntry(PDRIVER\_OBJECT driver, PUNICODE\_STRING reg\_path);
NTSTATUS DriverUnload(PDRIVER\_OBJECT driver);
NTSTATUS DriverEntry(PDRIVER\_OBJECT driver, PUNICODE\_STRING reg\_path)
{
UNREFERENCED\_PARAMETER(reg\_path);
PEPROCESS pEprocess, pCurProcess;
PCHAR ImageFileName;
DbgBreakPoint();
\_\_asm
{
mov eax, fs: [0x124] ; // 获取指向 \_KTHREAD 的指针
mov eax, [eax + 0x44]; // 获取指向 \_KPROCESS 的指针， 即EPROCESS 的首地址
mov pEprocess, eax;
}
pCurProcess = pEprocess;
do
{
ImageFileName = (PCHAR)pCurProcess + 0x174; // 进程名
if (strcmp(ImageFileName, "notepad.exe") == 0)
{
PLIST\_ENTRY curNode;
curNode = (PLIST\_ENTRY)((ULONG)pCurProcess + 0x88); // +88 指向链表结构体
curNode->Flink->Blink = curNode->Blink;
curNode->Blink->Flink = curNode->Flink;
DbgPrint("断链成功!\n");
break;
}
pCurProcess = (PEPROCESS)(\*(PULONG)((ULONG)pCurProcess + 0x88) - 0x88); // 更新进程
} while (pEprocess != pCurProcess);
driver->DriverUnload = DriverUnload;
return STATUS\_SUCCESS;
}
NTSTATUS DriverUnload(PDRIVER\_OBJECT driver)
{
UNREFERENCED\_PARAMETER(driver);
DbgPrint("驱动卸载成功\n");
return STATUS\_SUCCESS;
}
```

### 关键代码解释

#### （1）更新`pCurProcess`的说明

```c++
// + 0x88：从 \_LIST\_ENTRY结构体中的Flink 指向的是下一个进程的双向循环链表结构体首地址
// - 0x88：链表指向的是下一个进程结构体+88的位置，-0x88让指针指向EPROCESS的首地址
pCurProcess = (PEPROCESS)(\*(PULONG)((ULONG)pCurProcess + 0x88) - 0x88);
```

#### （2）获取EPRCESS的两种办法

第1种：通过内核函数 `PsGetCurrentProcess`。原理已在前面部分解释

```c++
kd> u psgetcurrentprocess
nt!PsGetCurrentProcess:
804ef608 64a124010000 mov eax,dword ptr fs:[00000124h] // 获取指向 \_KTHREAD 的指针
804ef60e 8b4044 mov eax,dword ptr [eax+44h] // 获取指向 \_KPROCESS 的指针， 即EPROCESS 的首地址
```

第2种：

获取进程模块的步骤为：获取 `KPCR` -> `KTHREAD`-> `ETHREAD` -> `EPROCESS`

对应的汇编是：

```c++
\_\_asm
{
mov eax, fs: [0x124] ; // 获取指向 \_KTHREAD 的指针， 即 ETHREAD 的首地址。原理与上面一样
mov eax, [eax + 0x220]; // ETHREAD + 0x220 -> ThreadsProcess : Ptr32 \_EPROCESS
mov pEprocess, eax;
}
```

### `_ETHREAD`结构体

```c++
kd> dt \_EThread
ntdll!\_ETHREAD
+0x000 Tcb : \_KTHREAD // \_KTHREAD 结构体
+0x1c0 CreateTime : \_LARGE\_INTEGER
+0x1c0 NestedFaultCount : Pos 0, 2 Bits
+0x1c0 ApcNeeded : Pos 2, 1 Bit
+0x1c8 ExitTime : \_LARGE\_INTEGER
+0x1c8 LpcReplyChain : \_LIST\_ENTRY
+0x1c8 KeyedWaitChain : \_LIST\_ENTRY
+0x1d0 ExitStatus : Int4B
+0x1d0 OfsChain : Ptr32 Void
+0x1d4 PostBlockList : \_LIST\_ENTRY
+0x1dc TerminationPort : Ptr32 \_TERMINATION\_PORT
+0x1dc ReaperLink : Ptr32 \_ETHREAD
+0x1dc KeyedWaitValue : Ptr32 Void
+0x1e0 ActiveTimerListLock : Uint4B
+0x1e4 ActiveTimerListHead : \_LIST\_ENTRY
+0x1ec Cid : \_CLIENT\_ID
+0x1f4 LpcReplySemaphore : \_KSEMAPHORE
+0x1f4 KeyedWaitSemaphore : \_KSEMAPHORE
+0x208 LpcReplyMessage : Ptr32 Void
+0x208 LpcWaitingOnPort : Ptr32 Void
+0x20c ImpersonationInfo : Ptr32 \_PS\_IMPERSONATION\_INFORMATION
+0x210 IrpList : \_LIST\_ENTRY
+0x218 TopLevelIrp : Uint4B
+0x21c DeviceToVerify : Ptr32 \_DEVICE\_OBJECT
+0x220 ThreadsProcess : Ptr32 \_EPROCESS // PPEB
+0x224 StartAddress : Ptr32 Void
... ...
```

### 总结

（1）隐藏模块或进程就是一个断链的过程，将自身节点从双向循环链表中删除就可以达到隐藏的效果

（2）破坏双向循环链表的连续性就可以达到隐藏全部的效果

（3）为什么删除进程的节点后，程序任然可以运行？这是因为CPU调度的基本单位是线程，和进程无关

（4）找到EPROCESS也有多种方法

ps: 以下实验均在 `windows xp`系统下完成

### 1. 3环断链（PEB断链）

作用：3环断链通常用于在用户层隐藏dll

获取dll模块的步骤为：获取 `TEB` -> `PEB` -> `ldr` -> `dll`模块

（1）`TEB`的获取方法为：`fs:[0]`

（2）通过`TEB`结构体发现，`PPEB`（PEB指针，指向PEB首地址）位于`TEB`偏移`0x30`处，因此`PPEB == fs:[0x30]`

（3）通过`PEB`结构体发现，`ldr`位于`PEB`偏移`0xc`处，因此 `ldr == PPEB + 0xc`

（4）通过`ldr`指向的结构体 `\_PEB\_LDR\_DATA` 结构体找到3个双向循环链表 `\_LIST\_ENTRY`结构体

（5）`\_LIST\_ENTRY`结构体有两个成员，指向前驱模块节点的 `\_LIST\_ENTRY`、指向后继模块节点的 `\_LIST\_ENTRY`

（6）节点指针指向的是 `\_LIST\_ENTRY` 结构体，该结构体存储了模块在链表中的位置信息

（7）通过 `\_LDR\_DATA\_TABLE\_ENTRY` 结构体中偏移 `0x18`的 `DllBase` 或 `0x2c` 的 `BaseDllName` 确定要隐藏的dll，并通过该结构体中的 `\_LIST\_ENTRY`结构体删除该节点在链表中的位置信息（双向循环链表删除节点），即可达到3环下的dll隐藏

`TEB`结构体：3环下的线程环境块结构体

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17 | `kd> dt _TEB`  `ntdll!_TEB`  `+0x000 NtTib            : _NT_TIB`  `+0x01c EnvironmentPointer : Ptr32 Void`  `+0x020 ClientId         : _CLIENT_ID`  `+0x028 ActiveRpcHandle  : Ptr32 Void`  `+0x02c ThreadLocalStoragePointer : Ptr32 Void`  `+0x030 ProcessEnvironmentBlock : Ptr32 _PEB` `// PPEB`  `+0x034 LastErrorValue   : Uint4B`  `+0x038 CountOfOwnedCriticalSections : Uint4B`  `+0x03c CsrClientThread  : Ptr32 Void`  `+0x040 Win32ThreadInfo  : Ptr32 Void`  `+0x044 User32Reserved   : [26] Uint4B`  `+0x0ac UserReserved     : [5] Uint4B`  `+0x0c0 WOW32Reserved    : Ptr32 Void`  `+0x0c4 CurrentLocale    : Uint4B`  `... ...` |

`PEB`结构体：3环下的进程环境块结构体

|  |  |

| --- | --- |
