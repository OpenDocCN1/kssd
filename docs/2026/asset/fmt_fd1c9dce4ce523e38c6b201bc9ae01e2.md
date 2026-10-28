
```
   +0x050 Ss               : Uint2B

   +0x052 Reserved4        : Uint2B

   +0x054 Ds               : Uint2B

   +0x056 Reserved5        : Uint2B

   +0x058 Fs               : Uint2B

   +0x05a Reserved6        : Uint2B

   +0x05c Gs               : Uint2B

   +0x05e Reserved7        : Uint2B

   +0x060 LDT              : Uint2B

   +0x062 Reserved8        : Uint2B

   +0x064 Flags            : Uint2B

   +0x066 IoMapBase        : Uint2B

   +0x068 IoMaps           : [1] \_KiIoAccessMap

   +0x208c IntDirectionMap  : [32] UChar

lkd> dt \_KiIoAccessMap

nt!\_KiIoAccessMap

   +0x000 DirectionMap     : [32] UChar

   +0x020 IoMap            : [8196] UChar
```

4。`Prcb`对应的`\_KPRCB`，其偏移0x4字节对应于`CurrentThread`

```
lkd> dt \_KPRCB

ntdll!\_KPRCB

   +0x000 MinorVersion     : Uint2B

   +0x002 MajorVersion     : Uint2B

   +0x004 CurrentThread    : Ptr32 \_KTHREAD

   +0x008 NextThread       : Ptr32 \_KTHREAD

   +0x00c IdleThread       : Ptr32 \_KTHREAD

   +0x010 Number           : Char

   +0x011 Reserved         : Char

   +0x012 BuildType        : Uint2B

   +0x014 SetMember        : Uint4B

   +0x018 CpuType          : Char

   +0x019 CpuID            : Char

   +0x01a CpuStep          : Uint2B

   +0x01c ProcessorState   : \_KPROCESSOR\_STATE

   +0x33c KernelReserved   : [16] Uint4B

   +0x37c HalReserved      : [16] Uint4B

   +0x3bc PrcbPad0         : [92] UChar

   +0x418 LockQueue        : [16] \_KSPIN\_LOCK\_QUEUE

   +0x498 PrcbPad1         : [8] UChar

   +0x4a0 NpxThread        : Ptr32 \_KTHREAD

   +0x4a4 InterruptCount   : Uint4B

   +0x4a8 KernelTime       : Uint4B

   +0x4ac UserTime         : Uint4B

   +0x4b0 DpcTime          : Uint4B

   +0x4b4 DebugDpcTime     : Uint4B

   +0x4b8 InterruptTime    : Uint4B

   +0x4bc AdjustDpcThreshold : Uint4B

   +0x4c0 PageColor        : Uint4B

   +0x4c4 SkipTick         : Uint4B

   +0x4c8 MultiThreadSetBusy : UChar

   +0x4c9 Spare2           : [3] UChar

   +0x4cc ParentNode       : Ptr32 \_KNODE

   +0x4d0 MultiThreadProcessorSet : Uint4B

   +0x4d4 MultiThreadSetMaster : Ptr32 \_KPRCB

   +0x4d8 ThreadStartCount : [2] Uint4B

   +0x4e0 CcFastReadNoWait : Uint4B

   +0x4e4 CcFastReadWait   : Uint4B

   +0x4e8 CcFastReadNotPossible : Uint4B

   +0x4ec CcCopyReadNoWait : Uint4B

   +0x4f0 CcCopyReadWait   : Uint4B

   +0x4f4 CcCopyReadNoWaitMiss : Uint4B

   +0x4f8 KeAlignmentFixupCount : Uint4B

   +0x4fc KeContextSwitches : Uint4B

   +0x500 KeDcacheFlushCount : Uint4B

   +0x504 KeExceptionDispatchCount : Uint4B

   +0x508 KeFirstLevelTbFills : Uint4B

   +0x50c KeFloatingEmulationCount : Uint4B

   +0x510 KeIcacheFlushCount : Uint4B

   +0x514 KeSecondLevelTbFills : Uint4B

   +0x518 KeSystemCalls    : Uint4B

   +0x51c SpareCounter0    : [1] Uint4B

   +0x520 PPLookasideList  : [16] \_PP\_LOOKASIDE\_LIST

   +0x5a0 PPNPagedLookasideList : [32] \_PP\_LOOKASIDE\_LIST

   +0x6a0 PPPagedLookasideList : [32] \_PP\_LOOKASIDE\_LIST

   +0x7a0 PacketBarrier    : Uint4B

   +0x7a4 ReverseStall     : Uint4B

   +0x7a8 IpiFrame         : Ptr32 Void

   +0x7ac PrcbPad2         : [52] UChar

   +0x7e0 CurrentPacket    : [3] Ptr32 Void

   +0x7ec TargetSet        : Uint4B

   +0x7f0 WorkerRoutine    : Ptr32     void

   +0x7f4 IpiFrozen        : Uint4B

   +0x7f8 PrcbPad3         : [40] UChar

   +0x820 RequestSummary   : Uint4B

   +0x824 SignalDone       : Ptr32 \_KPRCB

   +0x828 PrcbPad4         : [56] UChar

   +0x860 DpcListHead      : \_LIST\_ENTRY

   +0x868 DpcStack         : Ptr32 Void

   +0x86c DpcCount         : Uint4B

   +0x870 DpcQueueDepth    : Uint4B

   +0x874 DpcRoutineActive : Uint4B

   +0x878 DpcInterruptRequested : Uint4B

   +0x87c DpcLastCount     : Uint4B

   +0x880 DpcRequestRate   : Uint4B

   +0x884 MaximumDpcQueueDepth : Uint4B

   +0x888 MinimumDpcRate   : Uint4B

   +0x88c QuantumEnd       : Uint4B

   +0x890 PrcbPad5         : [16] UChar

   +0x8a0 DpcLock          : Uint4B

   +0x8a4 PrcbPad6         : [28] UChar

   +0x8c0 CallDpc          : \_KDPC

   +0x8e0 ChainedInterruptList : Ptr32 Void

   +0x8e4 LookasideIrpFloat : Int4B

   +0x8e8 SpareFields0     : [6] Uint4B

   +0x900 VendorString     : [13] UChar

   +0x90d InitialApicId    : UChar

   +0x90e LogicalProcessorsPerPhysicalProcessor : UChar

   +0x910 MHz              : Uint4B

   +0x914 FeatureBits      : Uint4B

   +0x918 UpdateSignature  : \_LARGE\_INTEGER

   +0x920 NpxSaveArea      : \_FX\_SAVE\_AREA

   +0xb30 PowerState       : \_PROCESSOR\_POWER\_STATE
```

5。`CurrentThread`偏移0x44处对应于`ApcState.Process`, 即当前进程。

```
lkd> dt \_kthread

ntdll!\_KTHREAD

   +0x000 Header           : \_DISPATCHER\_HEADER

   +0x010 MutantListHead   : \_LIST\_ENTRY

   +0x018 InitialStack     : Ptr32 Void

   +0x01c StackLimit       : Ptr32 Void

   +0x020 Teb              : Ptr32 Void

   +0x024 TlsArray         : Ptr32 Void

   +0x028 KernelStack      : Ptr32 Void

   +0x02c DebugActive      : UChar

   +0x02d State            : UChar

   +0x02e Alerted          : [2] UChar

   +0x030 Iopl             : UChar

   +0x031 NpxState         : UChar

   +0x032 Saturation       : Char

   +0x033 Priority         : Char

   +0x034 ApcState         : \_KAPC\_STATE

   +0x04c ContextSwitches  : Uint4B

   +0x050 IdleSwapBlock    : UChar

   +0x051 Spare0           : [3] UChar

   +0x054 WaitStatus       : Int4B

   +0x058 WaitIrql         : UChar

   +0x059 WaitMode         : Char

   +0x05a WaitNext         : UChar

   +0x05b WaitReason       : UChar

   +0x05c WaitBlockList    : Ptr32 \_KWAIT\_BLOCK

   +0x060 WaitListEntry    : \_LIST\_ENTRY

   +0x060 SwapListEntry    : \_SINGLE\_LIST\_ENTRY

   +0x068 WaitTime         : Uint4B

   +0x06c BasePriority     : Char

   +0x06d DecrementCount   : UChar

   +0x06e PriorityDecrement : Char

   +0x06f Quantum          : Char

   +0x070 WaitBlock        : [4] \_KWAIT\_BLOCK

   +0x0d0 LegoData         : Ptr32 Void

   +0x0d4 KernelApcDisable : Uint4B

   +0x0d8 UserAffinity     : Uint4B

   +0x0dc SystemAffinityActive : UChar

   +0x0dd PowerState       : UChar

   +0x0de NpxIrql          : UChar

   +0x0df InitialNode      : UChar

   +0x0e0 ServiceTable     : Ptr32 Void

   +0x0e4 Queue            : Ptr32 \_KQUEUE

   +0x0e8 ApcQueueLock     : Uint4B

   +0x0f0 Timer            : \_KTIMER

   +0x118 QueueListEntry   : \_LIST\_ENTRY

   +0x120 SoftAffinity     : Uint4B

   +0x124 Affinity         : Uint4B

   +0x128 Preempted        : UChar

   +0x129 ProcessReadyQueue : UChar

   +0x12a KernelStackResident : UChar

   +0x12b NextProcessor    : UChar

   +0x12c CallbackStack    : Ptr32 Void

   +0x130 Win32Thread      : Ptr32 Void

   +0x134 TrapFrame        : Ptr32 \_KTRAP\_FRAME

   +0x138 ApcStatePointer  : [2] Ptr32 \_KAPC\_STATE

   +0x140 PreviousMode     : Char

   +0x141 EnableStackSwap  : UChar

   +0x142 LargeStack       : UChar

   +0x143 ResourceIndex    : UChar

   +0x144 KernelTime       : Uint4B

   +0x148 UserTime         : Uint4B

   +0x14c SavedApcState    : \_KAPC\_STATE

   +0x164 Alertable        : UChar

   +0x165 ApcStateIndex    : UChar

   +0x166 ApcQueueable     : UChar

   +0x167 AutoAlignment    : UChar

   +0x168 StackBase        : Ptr32 Void

   +0x16c SuspendApc       : \_KAPC

   +0x19c SuspendSemaphore : \_KSEMAPHORE

   +0x1b0 ThreadListEntry  : \_LIST\_ENTRY

   +0x1b8 FreezeCount      : Char

   +0x1b9 SuspendCount     : Char

   +0x1ba IdealProcessor   : UChar

   +0x1bb DisableBoost     : UChar

lkd> dt \_KAPC\_STATE

ntdll!\_KAPC\_STATE

   +0x000 ApcListHead      : [2] \_LIST\_ENTRY

   +0x010 Process          : Ptr32 \_KPROCESS

   +0x014 KernelApcInProgress : UChar

   +0x015 KernelApcPending : UChar

   +0x016 UserApcPending   : UChar
```

6。在`Process`偏移0x30处，对应于`IopmOffset`。

```
lkd> dt \_KPROCESS

ntdll!\_KPROCESS

   +0x000 Header           : \_DISPATCHER\_HEADER

   +0x010 ProfileListHead  : \_LIST\_ENTRY

   +0x018 DirectoryTableBase : [2] Uint4B

   +0x020 LdtDescriptor    : \_KGDTENTRY

   +0x028 Int21Descriptor  : \_KIDTENTRY

   +0x030 IopmOffset       : Uint2B
```
