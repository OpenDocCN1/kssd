

# 斗胆发一个辅助使用 WinDbg 获得内核数据结构的小工具

> 原文：[`www.pediy.com/kssd/pediy10/65415.html`](https://www.pediy.com/kssd/pediy10/65415.html)

来看雪一段时间了，得到高手们不少帮助，看着大家的问题也都帮不上什么忙，实在是惭愧啊，最近在学习 WinDbg 的使用，学到用 WinDbg 获得内核数据结构的时候发现有些内核数据结构比如 EPROCESS 都是老大的一段，如果要自己把这些代码都转成 C 语言的结构体定义的话那实在是太麻烦了，于是就有了这个小工具了，代码丑陋，就不拿上来献丑了，希望能够对那些和我一样正在学习 WinDbg 的同仁们有所帮助。我也不能保证得到的代码都是正确的，不过我测试了一下，把和 EPROCESS 有关的所有结构体都转成了 C 语言的结构体，sizeof 了一下发现没有错误。  
  

代码:

```
#pragma pack(push)
#pragma pack(1)  
  typedef ULONG UINT4B;
  typedef INT64 UINT8B;
  typedef INT INT4B;
  typedef SHORT UINT2B;
  typedef int \_UNNAMED;
  typedef char BYTE;

  typedef struct
  {
    UINT2B LimitLow; //+0x000
    UINT2B BaseLow; //+0x002
    \_UNNAMED HighWord; //+0x004
  }KGDTENTRY;
  typedef struct
  {
    UINT2B Offset; //+0x000
    UINT2B Selector; //+0x002
    UINT2B Access; //+0x004
    UINT2B ExtendedOffset; //+0x006
  }KIDTENTRY;
  typedef struct
  {
    union {
      BYTE ExecuteDisable:1; //+0x000,Pos 0
      BYTE ExecuteEnable:1; //+0x000,Pos 1
      BYTE DisableThunkEmulation:1; //+0x000,Pos 2
      BYTE Permanent:1; //+0x000,Pos 3
      BYTE ExecuteDispatchEnable:1; //+0x000,Pos 4
      BYTE ImageDispatchEnable:1; //+0x000,Pos 5
      BYTE Spare:2; //+0x000,Pos 6
    };
  }KEXECUTE\_OPTIONS;

  typedef struct
  {
    DISPATCHER\_HEADER Header; //+0x000
    LIST\_ENTRY ProfileListHead; //+0x010
    UINT4B DirectoryTableBase\[2\]; //+0x018
    KGDTENTRY LdtDescriptor; //+0x020
    KIDTENTRY Int21Descriptor; //+0x028
    UINT2B IopmOffset; //+0x030
    UCHAR Iopl; //+0x032
    UCHAR Unused; //+0x033
    UINT4B ActiveProcessors; //+0x034
    UINT4B KernelTime; //+0x038
    UINT4B UserTime; //+0x03c
    LIST\_ENTRY ReadyListHead; //+0x040
    SINGLE\_LIST\_ENTRY SwapListEntry; //+0x048
    PVOID VdmTrapcHandler; //+0x04c
    LIST\_ENTRY ThreadListHead; //+0x050
    UINT4B ProcessLock; //+0x058
    UINT4B Affinity; //+0x05c
    UINT2B StackCount; //+0x060
    CHAR BasePriority; //+0x062
    CHAR ThreadQuantum; //+0x063
    UCHAR AutoAlignment; //+0x064
    UCHAR State; //+0x065
    UCHAR ThreadSeed; //+0x066
    UCHAR DisableBoost; //+0x067
    UCHAR PowerState; //+0x068
    UCHAR DisableQuantum; //+0x069
    UCHAR IdealNode; //+0x06a
    union {
      KEXECUTE\_OPTIONS Flags; //+0x06b
      UCHAR ExecuteOptions; //+0x06b
    };
  }KPROCESS;
  
  typedef struct
  {
    union {
      ULONG Waiting:1; //+0x000,Pos 0
      ULONG Exclusive:1; //+0x000,Pos 1
      ULONG Shared:30; //+0x000,Pos 2
      UINT4B Value; //+0x000
      PVOID Ptr; //+0x000
    };
  }EX\_PUSH\_LOCK;

  typedef struct
  {
    CLIENT\_ID ClientId; //+0x000
    PVOID Handle; //+0x008
    UINT4B Type; //+0x00c
    PVOID StackTrace\[16\]; //+0x010
  }HANDLE\_TRACE\_DB\_ENTRY;

  typedef struct
  {
    UINT4B CurrentStackIndex; //+0x000
    HANDLE\_TRACE\_DB\_ENTRY TraceDb\[4096\]; //+0x004
  }HANDLE\_TRACE\_DEBUG\_INFO,\*PHANDLE\_TRACE\_DEBUG\_INFO;

  typedef struct
  {
    UINT4B TableCode; //+0x000
    PEPROCESS QuotaProcess; //+0x004
    PVOID UniqueProcessId; //+0x008
    EX\_PUSH\_LOCK HandleTableLock\[4\]; //+0x00c
    LIST\_ENTRY HandleTableList; //+0x01c
    EX\_PUSH\_LOCK HandleContentionEvent; //+0x024
    PHANDLE\_TRACE\_DEBUG\_INFO DebugInfo; //+0x028
    INT4B ExtraInfoPages; //+0x02c
    UINT4B FirstFree; //+0x030
    UINT4B LastFree; //+0x034
    UINT4B NextHandleNeedingPool; //+0x038
    INT4B HandleCount; //+0x03c
    union {
      UINT4B Flags; //+0x040
      UINT4B StrictFIFO:1; //+0x040,Pos 0
    };
  }HANDLE\_TABLE,\*PHANDLE\_TABLE;

  typedef struct
  {
    union {
      PVOID Object; //+0x000
      UINT4B RefCnt:3; //+0x000,Pos 0
      UINT4B Value; //+0x000
    };
  }EX\_FAST\_REF;

  typedef struct
  {
    PVOID Sid; //+0x000
    UINT4B Attributes; //+0x004
  }SID\_AND\_ATTRIBUTES,\*PSID\_AND\_ATTRIBUTES;

  typedef struct
  {
    UINT4B CapturedSidCount; //+0x000
    PSID\_AND\_ATTRIBUTES CapturedSids; //+0x004
    UINT4B CapturedSidsLength; //+0x008
    UINT4B CapturedGroupCount; //+0x00c
    PSID\_AND\_ATTRIBUTES CapturedGroups; //+0x010
    UINT4B CapturedGroupsLength; //+0x014
    UINT4B CapturedPrivilegeCount; //+0x018
    PLUID\_AND\_ATTRIBUTES CapturedPrivileges; //+0x01c
    UINT4B CapturedPrivilegesLength; //+0x020
  }PS\_JOB\_TOKEN\_FILTER,\*PPS\_JOB\_TOKEN\_FILTER;

  typedef struct
  {
    KEVENT Event; //+0x000
    LIST\_ENTRY JobLinks; //+0x010
    LIST\_ENTRY ProcessListHead; //+0x018
    ERESOURCE JobLock; //+0x020
    LARGE\_INTEGER TotalUserTime; //+0x058
    LARGE\_INTEGER TotalKernelTime; //+0x060
    LARGE\_INTEGER ThisPeriodTotalUserTime; //+0x068
    LARGE\_INTEGER ThisPeriodTotalKernelTime; //+0x070
    UINT4B TotalPageFaultCount; //+0x078
    UINT4B TotalProcesses; //+0x07c
    UINT4B ActiveProcesses; //+0x080
    UINT4B TotalTerminatedProcesses; //+0x084
    LARGE\_INTEGER PerProcessUserTimeLimit; //+0x088
    LARGE\_INTEGER PerJobUserTimeLimit; //+0x090
    UINT4B LimitFlags; //+0x098
    UINT4B MinimumWorkingSetSize; //+0x09c
    UINT4B MaximumWorkingSetSize; //+0x0a0
    UINT4B ActiveProcessLimit; //+0x0a4
    UINT4B Affinity; //+0x0a8
    UCHAR PriorityClass; //+0x0ac
    UINT4B UIRestrictionsClass; //+0x0b0
    UINT4B SecurityLimitFlags; //+0x0b4
    PVOID Token; //+0x0b8
    PPS\_JOB\_TOKEN\_FILTER Filter; //+0x0bc
    UINT4B EndOfJobTimeAction; //+0x0c0
    PVOID CompletionPort; //+0x0c4
    PVOID CompletionKey; //+0x0c8
    UINT4B SessionId; //+0x0cc
    UINT4B SchedulingClass; //+0x0d0
    UINT8B ReadOperationCount; //+0x0d8
    UINT8B WriteOperationCount; //+0x0e0
    UINT8B OtherOperationCount; //+0x0e8
    UINT8B ReadTransferCount; //+0x0f0
    UINT8B WriteTransferCount; //+0x0f8
    UINT8B OtherTransferCount; //+0x100
    IO\_COUNTERS IoInfo; //+0x108
    UINT4B ProcessMemoryLimit; //+0x138
    UINT4B JobMemoryLimit; //+0x13c
    UINT4B PeakProcessMemoryUsed; //+0x140
    UINT4B PeakJobMemoryUsed; //+0x144
    UINT4B CurrentJobMemoryUsed; //+0x148
    FAST\_MUTEX MemoryLimitsLock; //+0x14c
    LIST\_ENTRY JobSetLinks; //+0x16c
    UINT4B MemberLevel; //+0x174
    UINT4B JobFlags; //+0x178
  }EJOB,\*PEJOB;

  typedef struct
  {
    UINT4B Usage; //+0x000
    UINT4B Limit; //+0x004
    UINT4B Peak; //+0x008
    UINT4B Return; //+0x00c
  }EPROCESS\_QUOTA\_ENTRY;

  typedef struct
  {
    EPROCESS\_QUOTA\_ENTRY QuotaEntry\[3\]; //+0x000
    LIST\_ENTRY QuotaList; //+0x030
    UINT4B ReferenceCount; //+0x038
    UINT4B ProcessCount; //+0x03c
  }EPROCESS\_QUOTA\_BLOCK,\*PEPROCESS\_QUOTA\_BLOCK;

  typedef struct
  {
    UINT4B CurrentIndex; //+0x000
    UINT4B MaxIndex; //+0x004
    UINT4B SpinLock; //+0x008
    PVOID Reserved; //+0x00c
    PROCESS\_WS\_WATCH\_INFORMATION WatchInfo\[1\]; //+0x010
  }PAGEFAULT\_HISTORY,\*PPAGEFAULT\_HISTORY;

  typedef struct
  {
    union {
      ULONG Valid:1; //+0x000,Pos 0
      ULONG Write:1; //+0x000,Pos 1
      ULONG Owner:1; //+0x000,Pos 2
      ULONG WriteThrough:1; //+0x000,Pos 3
      ULONG CacheDisable:1; //+0x000,Pos 4
      ULONG Accessed:1; //+0x000,Pos 5
      ULONG Dirty:1; //+0x000,Pos 6
      ULONG LargePage:1; //+0x000,Pos 7
      ULONG Global:1; //+0x000,Pos 8
      ULONG CopyOnWrite:1; //+0x000,Pos 9
      ULONG Prototype:1; //+0x000,Pos 10
      ULONG reserved:1; //+0x000,Pos 11
      ULONG PageFrameNumber:20; //+0x000,Pos 12
    };
  }HARDWARE\_PTE;

  typedef struct
  {
    POBJECT\_NAME\_INFORMATION ImageFileName; //+0x000
  }SE\_AUDIT\_PROCESS\_CREATION\_INFO;

  typedef struct
  {
    union {
      ULONG SessionSpace:1; //+0x000,Pos 0
      ULONG BeingTrimmed:1; //+0x000,Pos 1
      ULONG SessionLeader:1; //+0x000,Pos 2
      ULONG TrimHard:1; //+0x000,Pos 3
      ULONG WorkingSetHard:1; //+0x000,Pos 4
      ULONG AddressSpaceBeingDeleted:1; //+0x000,Pos 5
      ULONG Available:10; //+0x000,Pos 6
      ULONG AllowWorkingSetAdjustment:8; //+0x000,Pos 16
      ULONG MemoryPriority:8; //+0x000,Pos 24
    };
  }MMSUPPORT\_FLAGS;

  typedef struct
  {
    \_UNNAMED u1; //+0x000
  }MMWSLE,\*PMMWSLE;


  typedef struct
  {
    PVOID Key; //+0x000
    UINT4B Index; //+0x004
  }MMWSLE\_HASH,\*PMMWSLE\_HASH;

  typedef struct
  {
    UINT4B Quota; //+0x000
    UINT4B FirstFree; //+0x004
    UINT4B FirstDynamic; //+0x008
    UINT4B LastEntry; //+0x00c
    UINT4B NextSlot; //+0x010
    PMMWSLE Wsle; //+0x014
    UINT4B LastInitializedWsle; //+0x018
    UINT4B NonDirectCount; //+0x01c
    PMMWSLE\_HASH HashTable; //+0x020
    UINT4B HashTableSize; //+0x024
    UINT4B NumberOfCommittedPageTables; //+0x028
    PVOID HashTableStart; //+0x02c
    PVOID HighestPermittedHashAddress; //+0x030
    UINT4B NumberOfImageWaiters; //+0x034
    UINT4B VadBitMapHint; //+0x038
    UINT2B UsedPageTableEntries\[768\]; //+0x03c
    UINT4B CommittedPageTables\[24\]; //+0x63c
  }MMWSL,\*PMMWSL;

  typedef struct
  {
    LARGE\_INTEGER LastTrimTime; //+0x000
    MMSUPPORT\_FLAGS Flags; //+0x008
    UINT4B PageFaultCount; //+0x00c
    UINT4B PeakWorkingSetSize; //+0x010
    UINT4B WorkingSetSize; //+0x014
    UINT4B MinimumWorkingSetSize; //+0x018
    UINT4B MaximumWorkingSetSize; //+0x01c
    PMMWSL VmWorkingSetList; //+0x020
    LIST\_ENTRY WorkingSetExpansionLinks; //+0x024
    UINT4B Claim; //+0x02c
    UINT4B NextEstimationSlot; //+0x030
    UINT4B NextAgingSlot; //+0x034
    UINT4B EstimatedAvailable; //+0x038
    UINT4B GrowthSinceLastEstimate; //+0x03c
  }MMSUPPORT;

  typedef struct
  {
    KPROCESS Pcb; //+0x000
    EX\_PUSH\_LOCK ProcessLock; //+0x06c
    LARGE\_INTEGER CreateTime; //+0x070
    LARGE\_INTEGER ExitTime; //+0x078
    EX\_RUNDOWN\_REF RundownProtect; //+0x080
    PVOID UniqueProcessId; //+0x084
    LIST\_ENTRY ActiveProcessLinks; //+0x088
    UINT4B QuotaUsage\[3\]; //+0x090
    UINT4B QuotaPeak\[3\]; //+0x09c
    UINT4B CommitCharge; //+0x0a8
    UINT4B PeakVirtualSize; //+0x0ac
    UINT4B VirtualSize; //+0x0b0
    LIST\_ENTRY SessionProcessLinks; //+0x0b4
    PVOID DebugPort; //+0x0bc
    PVOID ExceptionPort; //+0x0c0
    PHANDLE\_TABLE ObjectTable; //+0x0c4
    EX\_FAST\_REF Token; //+0x0c8
    FAST\_MUTEX WorkingSetLock; //+0x0cc
    UINT4B WorkingSetPage; //+0x0ec
    FAST\_MUTEX AddressCreationLock; //+0x0f0
    UINT4B HyperSpaceLock; //+0x110
    PETHREAD ForkInProgress; //+0x114
    UINT4B HardwareTrigger; //+0x118
    PVOID VadRoot; //+0x11c
    PVOID VadHint; //+0x120
    PVOID CloneRoot; //+0x124
    UINT4B NumberOfPrivatePages; //+0x128
    UINT4B NumberOfLockedPages; //+0x12c
    PVOID Win32Process; //+0x130
    PEJOB Job; //+0x134
    PVOID SectionObject; //+0x138
    PVOID SectionBaseAddress; //+0x13c
    PEPROCESS\_QUOTA\_BLOCK QuotaBlock; //+0x140
    PPAGEFAULT\_HISTORY WorkingSetWatch; //+0x144
    PVOID Win32WindowStation; //+0x148
    PVOID InheritedFromUniqueProcessId; //+0x14c
    PVOID LdtInformation; //+0x150
    PVOID VadFreeHint; //+0x154
    PVOID VdmObjects; //+0x158
    PVOID DeviceMap; //+0x15c
    LIST\_ENTRY PhysicalVadList; //+0x160
    union {
      HARDWARE\_PTE PageDirectoryPte; //+0x168
      UINT8B Filler; //+0x168
    };
    PVOID Session; //+0x170
    UCHAR ImageFileName\[16\]; //+0x174
    LIST\_ENTRY JobLinks; //+0x184
    PVOID LockedPagesList; //+0x18c
    LIST\_ENTRY ThreadListHead; //+0x190
    PVOID SecurityPort; //+0x198
    PVOID PaeTop; //+0x19c
    UINT4B ActiveThreads; //+0x1a0
    UINT4B GrantedAccess; //+0x1a4
    UINT4B DefaultHardErrorProcessing; //+0x1a8
    INT4B LastThreadExitStatus; //+0x1ac
    PPEB Peb; //+0x1b0
    EX\_FAST\_REF PrefetchTrace; //+0x1b4
    LARGE\_INTEGER ReadOperationCount; //+0x1b8
    LARGE\_INTEGER WriteOperationCount; //+0x1c0
    LARGE\_INTEGER OtherOperationCount; //+0x1c8
    LARGE\_INTEGER ReadTransferCount; //+0x1d0
    LARGE\_INTEGER WriteTransferCount; //+0x1d8
    LARGE\_INTEGER OtherTransferCount; //+0x1e0
    UINT4B CommitChargeLimit; //+0x1e8
    UINT4B CommitChargePeak; //+0x1ec
    PVOID AweInfo; //+0x1f0
    SE\_AUDIT\_PROCESS\_CREATION\_INFO SeAuditProcessCreationInfo; //+0x1f4
    MMSUPPORT Vm; //+0x1f8
    UINT4B LastFaultCount; //+0x238
    UINT4B ModifiedPageCount; //+0x23c
    UINT4B NumberOfVads; //+0x240
    UINT4B JobStatus; //+0x244
    union {
      UINT4B Flags; //+0x248
      ULONG CreateReported:1; //+0x248,Pos 0
      ULONG NoDebugInherit:1; //+0x248,Pos 1
      ULONG ProcessExiting:1; //+0x248,Pos 2
      ULONG ProcessDelete:1; //+0x248,Pos 3
      ULONG Wow64SplitPages:1; //+0x248,Pos 4
      ULONG VmDeleted:1; //+0x248,Pos 5
      ULONG OutswapEnabled:1; //+0x248,Pos 6
      ULONG Outswapped:1; //+0x248,Pos 7
      ULONG ForkFailed:1; //+0x248,Pos 8
      ULONG HasPhysicalVad:1; //+0x248,Pos 9
      ULONG AddressSpaceInitialized:2; //+0x248,Pos 10
      ULONG SetTimerResolution:1; //+0x248,Pos 12
      ULONG BreakOnTermination:1; //+0x248,Pos 13
      ULONG SessionCreationUnderway:1; //+0x248,Pos 14
      ULONG WriteWatch:1; //+0x248,Pos 15
      ULONG ProcessInSession:1; //+0x248,Pos 16
      ULONG OverrideAddressSpace:1; //+0x248,Pos 17
      ULONG HasAddressSpace:1; //+0x248,Pos 18
      ULONG LaunchPrefetched:1; //+0x248,Pos 19
      ULONG InjectInpageErrors:1; //+0x248,Pos 20
      ULONG VmTopDown:1; //+0x248,Pos 21
      ULONG Unused3:1; //+0x248,Pos 22
      ULONG Unused4:1; //+0x248,Pos 23
      ULONG VdmAllowed:1; //+0x248,Pos 24
      ULONG Unused:5; //+0x248,Pos 25
      ULONG Unused1:1; //+0x248,Pos 30
      ULONG Unused2:1; //+0x248,Pos 31
    };
    INT4B ExitStatus; //+0x24c
    UINT2B NextPageColor; //+0x250
    UCHAR SubSystemMinorVersion; //+0x252
    UCHAR SubSystemMajorVersion; //+0x253
    UINT2B SubSystemVersion; //+0x252
    UCHAR PriorityClass; //+0x254
    UCHAR WorkingSetAcquiredUnsafe; //+0x255
    UINT4B Cookie; //+0x258
  }EPROCESS,\*\_PEPROCESS;

#pragma pack(pop)

```

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || CodeTransformer.rar  
 |
| ![](img/0f24970173caf34cf302ee22e88648bc.jpg) || 我得到的和 EPROCESS 有关的结构体定义.txt  
 |