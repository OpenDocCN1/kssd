
```c
UCHAR VendorString[13]; //0x336c
UCHAR InitialApicId; //0x3379
UCHAR LogicalProcessorsPerPhysicalProcessor; //0x337a
UCHAR PrcbPad9[5]; //0x337b
ULONG FeatureBits; //0x3380
union _LARGE_INTEGER UpdateSignature; //0x3388
volatile ULONGLONG IsrTime; //0x3390
ULONGLONG RuntimeAccumulation; //0x3398
struct _PROCESSOR_POWER_STATE PowerState; //0x33a0
struct _KDPC DpcWatchdogDpc; //0x3468
struct _KTIMER DpcWatchdogTimer; //0x3488
VOID* WheaInfo; //0x34b0
VOID* EtwSupport; //0x34b4
union _SLIST_HEADER InterruptObjectPool; //0x34b8
union _SLIST_HEADER HypercallPageList; //0x34c0
VOID* HypercallPageVirtual; //0x34c8
VOID* VirtualApicAssist; //0x34cc
ULONGLONG* StatisticsPage; //0x34d0
VOID* RateControl; //0x34d4
struct _CACHE_DESCRIPTOR Cache[5]; //0x34d8
ULONG CacheCount; //0x3514
ULONG CacheProcessorMask[5]; //0x3518
struct _KAFFINITY_EX PackageProcessorSet; //0x352c
ULONG PrcbPad91[1]; //0x3538
ULONG CoreProcessorSet; //0x353c
struct _KDPC TimerExpirationDpc; //0x3540
ULONG SpinLockAcquireCount; //0x3560
ULONG SpinLockContentionCount; //0x3564
ULONG SpinLockSpinCount; //0x3568
ULONG IpiSendRequestBroadcastCount; //0x356c
ULONG IpiSendRequestRoutineCount; //0x3570
ULONG IpiSendSoftwareInterruptCount; //0x3574
ULONG ExInitializeResourceCount; //0x3578
ULONG ExReInitializeResourceCount; //0x357c
ULONG ExDeleteResourceCount; //0x3580
ULONG ExecutiveResourceAcquiresCount; //0x3584
ULONG ExecutiveResourceContentionsCount; //0x3588
ULONG ExecutiveResourceReleaseExclusiveCount; //0x358c
ULONG ExecutiveResourceReleaseSharedCount; //0x3590
ULONG ExecutiveResourceConvertsCount; //0x3594
ULONG ExAcqResExclusiveAttempts; //0x3598
ULONG ExAcqResExclusiveAcquiresExclusive; //0x359c
ULONG ExAcqResExclusiveAcquiresExclusiveRecursive; //0x35a0
ULONG ExAcqResExclusiveWaits; //0x35a4
ULONG ExAcqResExclusiveNotAcquires; //0x35a8
ULONG ExAcqResSharedAttempts; //0x35ac
ULONG ExAcqResSharedAcquiresExclusive; //0x35b0
ULONG ExAcqResSharedAcquiresShared; //0x35b4
ULONG ExAcqResSharedAcquiresSharedRecursive; //0x35b8
ULONG ExAcqResSharedWaits; //0x35bc
ULONG ExAcqResSharedNotAcquires; //0x35c0
ULONG ExAcqResSharedStarveExclusiveAttempts; //0x35c4
ULONG ExAcqResSharedStarveExclusiveAcquiresExclusive; //0x35c8
ULONG ExAcqResSharedStarveExclusiveAcquiresShared; //0x35cc
ULONG ExAcqResSharedStarveExclusiveAcquiresSharedRecursive; //0x35d0
ULONG ExAcqResSharedStarveExclusiveWaits; //0x35d4
ULONG ExAcqResSharedStarveExclusiveNotAcquires; //0x35d8
ULONG ExAcqResSharedWaitForExclusiveAttempts; //0x35dc
ULONG ExAcqResSharedWaitForExclusiveAcquiresExclusive; //0x35e0
ULONG ExAcqResSharedWaitForExclusiveAcquiresShared; //0x35e4
ULONG ExAcqResSharedWaitForExclusiveAcquiresSharedRecursive; //0x35e8
ULONG ExAcqResSharedWaitForExclusiveWaits; //0x35ec
ULONG ExAcqResSharedWaitForExclusiveNotAcquires; //0x35f0
ULONG ExSetResOwnerPointerExclusive; //0x35f4
ULONG ExSetResOwnerPointerSharedNew; //0x35f8
ULONG ExSetResOwnerPointerSharedOld; //0x35fc
ULONG ExTryToAcqExclusiveAttempts; //0x3600
ULONG ExTryToAcqExclusiveAcquires; //0x3604
ULONG ExBoostExclusiveOwner; //0x3608
ULONG ExBoostSharedOwners; //0x360c
ULONG ExEtwSynchTrackingNotificationsCount; //0x3610
ULONG ExEtwSynchTrackingNotificationsAccountedCount; //0x3614
struct _CONTEXT* Context; //0x3618
ULONG ContextFlags; //0x361c
struct _XSAVE_AREA* ExtendedState; //0x3620
};
```

- +4 `CurrentThread`：当前CPU正在跑的线程。
- +8 `NextThread`：将要切换的线程。
- +C `IdleThread`：如果没有要切换的线程，CPU将要跑的空闲线程。

其余字段在后续分析内核代码时再了解。

# 2.`EPROCESS`结构

在3环下，每个进程都有一个`PEB`结构用来描述这个进程的一些信息。这些信息仅是为了3环更好的操作进程。而在0环，每个进程又有一个结构体用于描述该进程的所有信息：`EPROCESS`。

结构体成员列表：

```c
//0x2c0 bytes (sizeof)
struct _EPROCESS
{
struct _KPROCESS Pcb; //0x0
struct _EX_PUSH_LOCK ProcessLock; //0x98
union _LARGE_INTEGER CreateTime; //0xa0
union _LARGE_INTEGER ExitTime; //0xa8
struct _EX_RUNDOWN_REF RundownProtect; //0xb0
VOID* UniqueProcessId; //0xb4
struct _LIST_ENTRY ActiveProcessLinks; //0xb8
ULONG ProcessQuotaUsage[2]; //0xc0
ULONG ProcessQuotaPeak[2]; //0xc8
volatile ULONG CommitCharge; //0xd0
struct _EPROCESS_QUOTA_BLOCK* QuotaBlock; //0xd4
struct _PS_CPU_QUOTA_BLOCK* CpuQuotaBlock; //0xd8
ULONG PeakVirtualSize; //0xdc
ULONG VirtualSize; //0xe0
struct _LIST_ENTRY SessionProcessLinks; //0xe4
VOID* DebugPort; //0xec
union
{
VOID* ExceptionPortData; //0xf0
ULONG ExceptionPortValue; //0xf0
ULONG ExceptionPortState:3; //0xf0
};
struct _HANDLE_TABLE* ObjectTable; //0xf4
struct _EX_FAST_REF Token; //0xf8
ULONG WorkingSetPage; //0xfc
struct _EX_PUSH_LOCK AddressCreationLock; //0x100
struct _ETHREAD* RotateInProgress; //0x104
struct _ETHREAD* ForkInProgress; //0x108
ULONG HardwareTrigger; //0x10c
struct _MM_AVL_TABLE* PhysicalVadRoot; //0x110
VOID* CloneRoot; //0x114
volatile ULONG NumberOfPrivatePages; //0x118
volatile ULONG NumberOfLockedPages; //0x11c
VOID* Win32Process; //0x120
struct _EJOB* volatile Job; //0x124
VOID* SectionObject; //0x128
VOID* SectionBaseAddress; //0x12c
ULONG Cookie; //0x130
ULONG Spare8; //0x134
struct _PAGEFAULT_HISTORY* WorkingSetWatch; //0x138
VOID* Win32WindowStation; //0x13c
VOID* InheritedFromUniqueProcessId; //0x140
VOID* LdtInformation; //0x144
VOID* VdmObjects; //0x148
ULONG ConsoleHostProcess; //0x14c
VOID* DeviceMap; //0x150
VOID* EtwDataSource; //0x154
VOID* FreeTebHint; //0x158
union
{
struct _HARDWARE_PTE PageDirectoryPte; //0x160
ULONGLONG Filler; //0x160
};
VOID* Session; //0x168
UCHAR ImageFileName[15]; //0x16c
UCHAR PriorityClass; //0x17b
struct _LIST_ENTRY JobLinks; //0x17c
VOID* LockedPagesList; //0x184
struct _LIST_ENTRY ThreadListHead; //0x188
VOID* SecurityPort; //0x190
VOID* PaeTop; //0x194
volatile ULONG ActiveThreads; //0x198
ULONG ImagePathHash; //0x19c
ULONG DefaultHardErrorProcessing; //0x1a0
LONG LastThreadExitStatus; //0x1a4
struct _PEB* Peb; //0x1a8
struct _EX_FAST_REF PrefetchTrace; //0x1ac
union _LARGE_INTEGER ReadOperationCount; //0x1b0
union _LARGE_INTEGER WriteOperationCount; //0x1b8
union _LARGE_INTEGER OtherOperationCount; //0x1c0
union _LARGE_INTEGER ReadTransferCount; //0x1c8
union _LARGE_INTEGER WriteTransferCount; //0x1d0
union _LARGE_INTEGER OtherTransferCount; //0x1d8
ULONG CommitChargeLimit; //0x1e0
volatile ULONG CommitChargePeak; //0x1e4
VOID* AweInfo; //0x1e8
struct _SE_AUDIT_PROCESS_CREATION_INFO SeAuditProcessCreationInfo; //0x1ec
struct _MMSUPPORT Vm; //0x1f0
struct _LIST_ENTRY MmProcessLinks; //0x25c
VOID* HighestUserAddress; //0x264
ULONG ModifiedPageCount; //0x268
union
{
ULONG Flags2; //0x26c
struct
{
ULONG JobNotReallyActive:1; //0x26c
ULONG AccountingFolded:1; //0x26c
ULONG NewProcessReported:1; //0x26c
ULONG ExitProcessReported:1; //0x26c
ULONG ReportCommitChanges:1; //0x26c
ULONG LastReportMemory:1; //0x26c
ULONG ReportPhysicalPageChanges:1; //0x26c
ULONG HandleTableRundown:1; //0x26c
ULONG NeedsHandleRundown:1; //0x26c
ULONG RefTraceEnabled:1; //0x26c
ULONG NumaAware:1; //0x26c
ULONG ProtectedProcess:1; //0x26c
ULONG DefaultPagePriority:3; //0x26c
ULONG PrimaryTokenFrozen:1; //0x26c
ULONG ProcessVerifierTarget:1; //0x26c
ULONG StackRandomizationDisabled:1; //0x26c
ULONG AffinityPermanent:1; //0x26c
ULONG AffinityUpdateEnable:1; //0x26c
ULONG PropagateNode:1; //0x26c
ULONG ExplicitAffinity:1; //0x26c
};
};
union
{
ULONG Flags; //0x270
struct
{
ULONG CreateReported:1; //0x270
ULONG NoDebugInherit:1; //0x270
ULONG ProcessExiting:1; //0x270
ULONG ProcessDelete:1; //0x270
ULONG Wow64SplitPages:1; //0x270
ULONG VmDeleted:1; //0x270
ULONG OutswapEnabled:1; //0x270
ULONG Outswapped:1; //0x270
ULONG ForkFailed:1; //0x270
ULONG Wow64VaSpace4Gb:1; //0x270
ULONG AddressSpaceInitialized:2; //0x270
ULONG SetTimerResolution:1; //0x270
ULONG BreakOnTermination:1; //0x270
ULONG DeprioritizeViews:1; //0x270
```
