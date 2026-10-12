
# rootkit 直接访问硬件之[三]

> 原文：[`www.pediy.com/kssd/pediy10/62081.html`](https://www.pediy.com/kssd/pediy10/62081.html)

关于 ring3 直接访问硬件的办法，前面已经介绍了 2 篇，本篇算是结束篇了。篇幅较长，大家慢慢阅读。高手飘过。

基本上我们能依据 8 条保护规则来进行访问的情况都概括了。还是跟前面一样，我们先贴出保护规则来。

1. 若 `CPL<=IOPL`，则直接转步骤(8)；
2. 取得 I/O 位图开始偏移；
3. 计算 I/O 地址对应位所在字节在 I/O 许可位图内的偏移；
4. 计算位偏移以形成屏蔽码值，即计算 I/O 地址对应位在字节中的第几位；
5. 把字节偏移加上位图开始偏移，再加 1，所得值与 `TSS` 界限比较，若越界，则产生出错码为 0 的通用保护故障；
6. 若不越界，则从位图中读对应字节及下一个字节；
7. 把读出的两个字节与屏蔽码进行与运算，若结果不为 0 表示检查未通过，则产生出错码为 0 的通用保护故障；
8. 进行 I/O 访问。

回顾下保护模式中介绍：`80386` 采用 I/O 特权级 `IPOL` 和 I/O 许可位图的方法来控制输入/输出，实现输入/输出保护。I/O 许可位图位于任务状态段 `TSS` 中。I/O 特权级 `IPOL` 就是 `EFLAGS` 寄存器中 `IOPL` 位。

我们先看看 I/O 许可位图，它位于任务状态段 `TSS` 中。我们可以在 `GDT` 中根据找到任务状态段描述符，根据这个描述符，我们可以找到 `TSS` 在内存中的位置。`TSS` 是保存一个任务重要信息的特殊段。我们先看看其内存结构。

```
lkd> dt _ktss

nt!_KTSS
  +0x000 Backlink        : Uint2B
  +0x002 Reserved0       : Uint2B
  +0x004 Esp0            : Uint4B
  +0x008 Ss0             : Uint2B
  +0x00a Reserved1       : Uint2B
  +0x00c NotUsed1        : [4] Uint4B
  +0x01c CR3             : Uint4B
  +0x020 Eip             : Uint4B
  +0x024 EFlags          : Uint4B
  +0x028 Eax             : Uint4B
  +0x02c Ecx             : Uint4B
  +0x030 Edx             : Uint4B
  +0x034 Ebx             : Uint4B
  +0x038 Esp             : Uint4B
  +0x03c Ebp             : Uint4B
  +0x040 Esi             : Uint4B
  +0x044 Edi             : Uint4B
  +0x048 Es              : Uint2B
  +0x04a Reserved2       : Uint2B
  +0x04c Cs              : Uint2B
  +0x04e Reserved3       : Uint2B
  +0x050 Ss              : Uint2B
  +0x052 Reserved4       : Uint2B
  +0x054 Ds              : Uint2B
  +0x056 Reserved5       : Uint2B
  +0x058 Fs              : Uint2B
  +0x05a Reserved6       : Uint2B
  +0x05c Gs              : Uint2B
  +0x05e Reserved7       : Uint2B
  +0x060 LDT             : Uint2B
  +0x062 Reserved8       : Uint2B
  +0x064 Flags           : Uint2B
  +0x066 IoMapBase       : Uint2B
  +0x068 IoMaps          : [1] _KiIoAccessMap
  +0x208c IntDirectionMap : [32] UChar

lkd> dt _KiIoAccessMap

nt!_KiIoAccessMap
  +0x000 DirectionMap    : [32] UChar
  +0x020 IoMap           : [8196] UChar
```

其中的 `IoMapBase` 就是 I/O 许可位图在 `TSS` 段中的偏移位置。也就是说在 `TSS` 段中从这个位置开始就是 I/O 许可位图了。另外在这个结构中还有 `IoMaps.IoMap` 一项，这是个长度为 8196 字节的数组。在 Windows 32 位操作系统中，端口是由 `word` 类型来描述的，也就是说最大端口数量就是 65536，即 64k。由于每个端口使用一个 `bit` 位来描述，因此 64k 的端口占用字节数就是 `65536/8 = 8192` 字节。因此，我们完全可以使用 `IoMaps.IoMap[8196]` 作为我们 I/O 许可位图区域。从结构上看，`IoMaps.IoMap` 位于 `tss` 段偏移 `0x88` 位置处。

也就是说，如果我们把 I/O 许可位图放到 `IoMaps.IoMap` 中，然后让 `IoMapBase` 和进程中的 `IopmOffset` 指向这个位置就行了。这个是我们的整体思路。

windows 提供以下三个内核函数：

```
BOOLEAN Ke386QueryIoAccessMap(ULONG MapNumber, PKIO_ACCESS_MAP IoAccessMap);

BOOLEAN Ke386SetIoAccessMap(ULONG MapNumber, PKIO_ACCESS_MAP IoAccessMap);

BOOLEAN Ke386IoSetAccessProcess(PKPROCESS Process, ULONG MapNumber);
```

其中第一个函数，是查询端口访问许可用的。第二个，第三个函数是用于设置 `tss` I/O 许可位图的，我们可以分析下第二、第三个函数，来验证下我们的思路。

```
lkd> u Ke386SetIoAccessMap L 30

nt!Ke386SetIoAccessMap:
804f80fe 8bff            mov     edi,edi
804f8100 55              push    ebp
804f8101 8bec            mov     ebp,esp
804f8103 57              push    edi
804f8104 8b7d08          mov     edi,dword ptr [ebp+8]
804f8107 83ff01          cmp     edi,1
804f810a 7759            ja      nt!Ke386SetIoAccessMap+0x67 (804f8165)
804f810c 85ff            test    edi,edi
804f810e 7455            je      nt!Ke386SetIoAccessMap+0x67 (804f8165)
804f8110 53              push    ebx
804f8111 56              push    esi
804f8112 ff158c864d80    call    dword ptr [nt!_imp__KeRaiseIrqlToSynchLevel (804d868c)]
804f8118 8ad8            mov     bl,al
804f811a 3ea120f0dfff    mov     eax,dword ptr ds:[FFDFF020h]
804f8120 8bd0            mov     edx,eax
804f8122 b800f0dfff      mov     eax,0FFDFF000h
804f8127 69ff24200000    imul    edi,edi,2024h    ; edi = 2024h
804f812d 8b4040          mov     eax,dword ptr [eax+40h]
804f8130 8b750c          mov     esi,dword ptr [ebp+0Ch]
804f8133 8dbc0764e0ffff  lea     edi,[edi+eax-1F9Ch]
804f813a b900080000      mov     ecx,800h   ;这里奇怪，IOPM 数组总长为 2000h。
为什么拷贝一部分？
804f813f f3a5            rep movs dword ptr es:[edi],dword ptr [esi]
804f8141 8b4204          mov     eax,dword ptr [edx+4] ; eax = CurrentThread
804f8144 8b4844          mov     ecx,dword ptr [eax+44h] ;ecx = CurrentProcess
804f8147 b800f0dfff      mov     eax,0FFDFF000h
804f814c 668b4930        mov     cx,word ptr [ecx+30h] ;cx = IopmOffset
804f8150 8b4040          mov     eax,dword ptr [eax+40h] ;tss
804f8153 66894866        mov     word ptr [eax+66h],cx
804f8157 8acb            mov     cl,bl
804f8159 ff151c874d80    call    dword ptr [nt!_imp_KfLowerIrql (804d871c)]
804f815f 5e              pop     esi
804f8160 b001            mov     al,1
804f8162 5b              pop     ebx
804f8163 eb02            jmp     nt!Ke386SetIoAccessMap+0x69 (804f8167)
804f8165 32c0            xor     al,al
804f8167 5f              pop     edi
804f8168 5d              pop     ebp
804f8169 c20800          ret     8
804f816c cc              int     3
804f816d cc              int     3
804f816e cc              int     3
804f816f cc              int     3
804f8170 cc              int     3
```

上述代码中的一些重要偏移值解释如下：

1. `0FFDFF000h` 对应于 `KPCR`, 用 `WINDBG` 看

```
lkd> !PCR

KPCR for Processor 0 at ffdff000:

    Major 1 Minor 1
  NtTib.ExceptionList: b2616c7c
      NtTib.StackBase: b2616df0
     NtTib.StackLimit: b2614000
   NtTib.SubSystemTib: 00000000
       NtTib.Version: 00000000
    NtTib.UserPointer: 00000000
        NtTib.SelfTib: 7ffde000
            SelfPcr: ffdff000
                 Prcb: ffdff120
                 Irql: 00000000
                  IRR: 00000000
                  IDR: ffffffff
        InterruptMode: 00000000
                   IDT: 8003f400
                   GDT: 8003f000
                   TSS: 80042000
        CurrentThread: 88fb6da8
         NextThread: 00000000
         IdleThread: 80552d20
          DpcQueue:
```

2. 看 `KPCR` 结构，`ds:[FFDFF020h]` 对应于 `Prcb`，`KPCR` 偏移 `0x40` 位置对应于 `TSS`

```
lkd> dt _kpcr

nt!_KPCR
  +0x000 NtTib           : _NT_TIB
  +0x01c SelfPcr         : Ptr32 _KPCR
  +0x020 Prcb           : Ptr32 _KPRCB
  +0x024 Irql           : UChar
  +0x028 IRR            : Uint4B
  +0x02c IrrActive      : Uint4B
  +0x030 IDR            : Uint4B
  +0x034 KdVersionBlock : Ptr32 Void
  +0x038 IDT            : Ptr32 _KIDTENTRY
  +0x03c GDT            : Ptr32 _KGDTENTRY
  +0x040 TSS            : Ptr32 _KTSS
  +0x044 MajorVersion   : Uint2B
  +0x046 MinorVersion   : Uint2B
  +0x048 SetMember      : Uint4B
  +0x04c StallScaleFactor : Uint4B
  +0x050 DebugActive    : UChar
  +0x051 Number         : UChar
  +0x052 Spare0         : UChar
  +0x053 SecondLevelCacheAssociativity : UChar
  +0x054 VdmAlert       : Uint4B
  +0x058 KernelReserved : [14] Uint4B
  +0x090 SecondLevelCacheSize : Uint4B
  +0x094 HalReserved    : [16] Uint4B
  +0x0d4 InterruptMode  : Uint4B
  +0x0d8 Spare1         : UChar
  +0x0dc KernelReserved2 : [17] Uint4B
  +0x120 PrcbData       : _KPRCB
```

3. 看 `tss` 结构，偏移 `0x88` 处对应于 `IoMaps[0].IoMap`，偏移 `0x66` 处，对应 `IoMapBase`。

```
lkd> dt _ktss

nt!_KTSS
  +0x000 Backlink        : Uint2B
  +0x002 Reserved0       : Uint2B
  +0x004 Esp0            : Uint4B
  +0x008 Ss0             : Uint2B
  +0x00a Reserved1       : Uint2B
  +0x00c NotUsed1        : [4] Uint4B
  +0x01c CR3             : Uint4B
  +0x020 Eip             : Uint4B
```



```
+0x024 EFlags         : Uint4B
+0x028 Eax            : Uint4B
+0x02c Ecx            : Uint4B
+0x030 Edx            : Uint4B
+0x034 Ebx            : Uint4B
+0x038 Esp            : Uint4B
+0x03c Ebp            : Uint4B
+0x040 Esi            : Uint4B
+0x044 Edi            : Uint4B
+0x048 Es             : Uint2B
+0x04a Reserved2      : Uint2B
+0x04c Cs             : Uint2B
+0x04e Reserved3      : Uint2B
+0x050 Ss             : Uint2B
+0x052 Reserved4      : Uint2B
+0x054 Ds             : Uint2B
+0x056 Reserved5      : Uint2B
+0x058 Fs             : Uint2B
+0x05a Reserved6      : Uint2B
+0x05c Gs             : Uint2B
+0x05e Reserved7      : Uint2B
+0x060 LDT            : Uint2B
+0x062 Reserved8      : Uint2B
+0x064 Flags          : Uint2B
+0x066 IoMapBase      : Uint2B
+0x068 IoMaps         : [1] _KiIoAccessMap
+0x208c IntDirectionMap : [32] UChar
```

lkd> dt `_KiIoAccessMap`

```
nt!_KiIoAccessMap
+0x000 DirectionMap   : [32] UChar
+0x020 IoMap          : [8196] UChar
```

4. Prcb 对应的\_KPRCB，其偏移 0x4 字节对应于 CurrentThread

lkd> dt `_KPRCB`

```
ntdll!_KPRCB
+0x000 MinorVersion         : Uint2B
+0x002 MajorVersion         : Uint2B
+0x004 CurrentThread        : Ptr32 _KTHREAD
+0x008 NextThread           : Ptr32 _KTHREAD
+0x00c IdleThread           : Ptr32 _KTHREAD
+0x010 Number               : Char
+0x011 Reserved             : Char
+0x012 BuildType            : Uint2B
+0x014 SetMember            : Uint4B
+0x018 CpuType              : Char
+0x019 CpuID                : Char
+0x01a CpuStep              : Uint2B
+0x01c ProcessorState       : _KPROCESSOR_STATE
+0x33c KernelReserved       : [16] Uint4B
+0x37c HalReserved          : [16] Uint4B
+0x3bc PrcbPad0             : [92] UChar
+0x418 LockQueue            : [16] _KSPIN_LOCK_QUEUE
+0x498 PrcbPad1             : [8] UChar
+0x4a0 NpxThread            : Ptr32 _KTHREAD
+0x4a4 InterruptCount       : Uint4B
+0x4a8 KernelTime           : Uint4B
+0x4ac UserTime             : Uint4B
+0x4b0 DpcTime              : Uint4B
+0x4b4 DebugDpcTime         : Uint4B
+0x4b8 InterruptTime        : Uint4B
+0x4bc AdjustDpcThreshold   : Uint4B
+0x4c0 PageColor            : Uint4B
+0x4c4 SkipTick             : Uint4B
+0x4c8 MultiThreadSetBusy   : UChar
+0x4c9 Spare2               : [3] UChar
+0x4cc ParentNode           : Ptr32 _KNODE
+0x4d0 MultiThreadProcessorSet : Uint4B
+0x4d4 MultiThreadSetMaster : Ptr32 _KPRCB
+0x4d8 ThreadStartCount     : [2] Uint4B
+0x4e0 CcFastReadNoWait     : Uint4B
+0x4e4 CcFastReadWait       : Uint4B
+0x4e8 CcFastReadNotPossible : Uint4B
+0x4ec CcCopyReadNoWait     : Uint4B
+0x4f0 CcCopyReadWait       : Uint4B
+0x4f4 CcCopyReadNoWaitMiss : Uint4B
+0x4f8 KeAlignmentFixupCount : Uint4B
+0x4fc KeContextSwitches    : Uint4B
+0x500 KeDcacheFlushCount   : Uint4B
+0x504 KeExceptionDispatchCount : Uint4B
+0x508 KeFirstLevelTbFills  : Uint4B
+0x50c KeFloatingEmulationCount : Uint4B
+0x510 KeIcacheFlushCount   : Uint4B
+0x514 KeSecondLevelTbFills : Uint4B
+0x518 KeSystemCalls        : Uint4B
+0x51c SpareCounter0        : [1] Uint4B
+0x520 PPLookasideList      : [16] _PP_LOOKASIDE_LIST
+0x5a0 PPNPagedLookasideList : [32] _PP_LOOKASIDE_LIST
+0x6a0 PPPagedLookasideList : [32] _PP_LOOKASIDE_LIST
+0x7a0 PacketBarrier        : Uint4B
+0x7a4 ReverseStall         : Uint4B
+0x7a8 IpiFrame             : Ptr32 Void
+0x7ac PrcbPad2             : [52] UChar
+0x7e0 CurrentPacket        : [3] Ptr32 Void
+0x7ec TargetSet            : Uint4B
+0x7f0 WorkerRoutine        : Ptr32 void
+0x7f4 IpiFrozen            : Uint4B
+0x7f8 PrcbPad3             : [40] UChar
+0x820 RequestSummary       : Uint4B
+0x824 SignalDone           : Ptr32 _KPRCB
+0x828 PrcbPad4             : [56] UChar
+0x860 DpcListHead          : _LIST_ENTRY
+0x868 DpcStack             : Ptr32 Void
+0x86c DpcCount             : Uint4B
+0x870 DpcQueueDepth        : Uint4B
+0x874 DpcRoutineActive     : Uint4B
+0x878 DpcInterruptRequested : Uint4B
+0x87c DpcLastCount         : Uint4B
+0x880 DpcRequestRate       : Uint4B
+0x884 MaximumDpcQueueDepth : Uint4B
+0x888 MinimumDpcRate       : Uint4B
+0x88c QuantumEnd           : Uint4B
+0x890 PrcbPad5             : [16] UChar
+0x8a0 DpcLock              : Uint4B
+0x8a4 PrcbPad6             : [28] UChar
+0x8c0 CallDpc              : _KDPC
+0x8e0 ChainedInterruptList : Ptr32 Void
+0x8e4 LookasideIrpFloat    : Int4B
+0x8e8 SpareFields0         : [6] Uint4B
+0x900 VendorString         : [13] UChar
+0x90d InitialApicId        : UChar
+0x90e LogicalProcessorsPerPhysicalProcessor : UChar
+0x910 MHz                  : Uint4B
+0x914 FeatureBits          : Uint4B
+0x918 UpdateSignature      : _LARGE_INTEGER
+0x920 NpxSaveArea          : _FX_SAVE_AREA
+0xb30 PowerState           : _PROCESSOR_POWER_STATE
```

5. `CurrentThread` 偏移 0x44 处对应于 `ApcState.Process`，即当前进程。

lkd> dt `_kthread`

```
ntdll!_KTHREAD
+0x000 Header               : _DISPATCHER_HEADER
+0x010 MutantListHead       : _LIST_ENTRY
+0x018 InitialStack         : Ptr32 Void
+0x01c StackLimit           : Ptr32 Void
+0x020 Teb                  : Ptr32 Void
+0x024 TlsArray             : Ptr32 Void
+0x028 KernelStack          : Ptr32 Void
+0x02c DebugActive          : UChar
+0x02d State                : UChar
+0x02e Alerted              : [2] UChar
+0x030 Iopl                 : UChar
+0x031 NpxState             : UChar
+0x032 Saturation           : Char
+0x033 Priority             : Char
+0x034 ApcState             : _KAPC_STATE
+0x04c ContextSwitches      : Uint4B
+0x050 IdleSwapBlock        : UChar
+0x051 Spare0               : [3] UChar
+0x054 WaitStatus           : Int4B
+0x058 WaitIrql             : UChar
+0x059 WaitMode             : Char
+0x05a WaitNext             : UChar
+0x05b WaitReason           : UChar
+0x05c WaitBlockList        : Ptr32 _KWAIT_BLOCK
+0x060 WaitListEntry        : _LIST_ENTRY
+0x060 SwapListEntry        : _SINGLE_LIST_ENTRY
+0x068 WaitTime             : Uint4B
+0x06c BasePriority         : Char
+0x06d DecrementCount       : UChar
+0x06e PriorityDecrement    : Char
+0x06f Quantum              : Char
+0x070 WaitBlock            : [4] _KWAIT_BLOCK
+0x0d0 LegoData             : Ptr32 Void
+0x0d4 KernelApcDisable     : Uint4B
+0x0d8 UserAffinity         : Uint4B
+0x0dc SystemAffinityActive : UChar
+0x0dd PowerState           : UChar
+0x0de NpxIrql              : UChar
+0x0df InitialNode          : UChar
+0x0e0 ServiceTable         : Ptr32 Void
+0x0e4 Queue                : Ptr32 _KQUEUE
+0x0e8 ApcQueueLock         : Uint4B
+0x0f0 Timer                : _KTIMER
+0x118 QueueListEntry       : _LIST_ENTRY
+0x120 SoftAffinity         : Uint4B
+0x124 Affinity             : Uint4B
+0x128 Preempted            : UChar
+0x129 ProcessReadyQueue    : UChar
+0x12a KernelStackResident  : UChar
+0x12b NextProcessor        : UChar
+0x12c CallbackStack        : Ptr32 Void
+0x130 Win32Thread          : Ptr32 Void
+0x134 TrapFrame            : Ptr32 _KTRAP_FRAME
+0x138 ApcStatePointer      : [2] Ptr32 _KAPC_STATE
+0x140 PreviousMode         : Char
+0x141 EnableStackSwap      : UChar
+0x142 LargeStack           : UChar
+0x143 ResourceIndex        : UChar
+0x144 KernelTime           : Uint4B
+0x148 UserTime             : Uint4B
+0x14c SavedApcState        : _KAPC_STATE
+0x164 Alertable            : UChar
+0x165 ApcStateIndex        : UChar
+0x166 ApcQueueable         : UChar
```



```
   +0x167 AutoAlignment    : UChar
   +0x168 StackBase        : Ptr32 Void
   +0x16c SuspendApc       : _KAPC
   +0x19c SuspendSemaphore : _KSEMAPHORE
   +0x1b0 ThreadListEntry  : _LIST_ENTRY
   +0x1b8 FreezeCount      : Char
   +0x1b9 SuspendCount     : Char
   +0x1ba IdealProcessor   : UChar
   +0x1bb DisableBoost     : UChar
```

`lkd> dt _KAPC_STATE`

`ntdll!_KAPC_STATE`

```
   +0x000 ApcListHead      : [2] _LIST_ENTRY
   +0x010 Process          : Ptr32 _KPROCESS
   +0x014 KernelApcInProgress : UChar
   +0x015 KernelApcPending : UChar
   +0x016 UserApcPending   : UChar
```

6。在 `Process` 偏移 `0x30` 处，对应于 `IopmOffset`。

`lkd> dt _KPROCESS`

`ntdll!_KPROCESS`

```
   +0x000 Header           : _DISPATCHER_HEADER
   +0x010 ProfileListHead  : _LIST_ENTRY
   +0x018 DirectoryTableBase : [2] Uint4B
   +0x020 LdtDescriptor    : _KGDTENTRY
   +0x028 Int21Descriptor  : _KIDTENTRY
   +0x030 IopmOffset       : Uint2B
   +0x032 Iopl             : UChar
   +0x033 Unused           : UChar
   +0x034 ActiveProcessors : Uint4B
   +0x038 KernelTime       : Uint4B
   +0x03c UserTime         : Uint4B
   +0x040 ReadyListHead    : _LIST_ENTRY
   +0x048 SwapListEntry    : _SINGLE_LIST_ENTRY
   +0x04c VdmTrapcHandler  : Ptr32 Void
   +0x050 ThreadListHead   : _LIST_ENTRY
   +0x058 ProcessLock      : Uint4B
   +0x05c Affinity         : Uint4B
   +0x060 StackCount       : Uint2B
   +0x062 BasePriority     : Char
   +0x063 ThreadQuantum    : Char
   +0x064 AutoAlignment    : UChar
   +0x065 State            : UChar
   +0x066 ThreadSeed       : UChar
   +0x067 DisableBoost     : UChar
   +0x068 PowerState       : UChar
   +0x069 DisableQuantum   : UChar
   +0x06a IdealNode        : UChar
   +0x06b Flags            : _KEXECUTE_OPTIONS
   +0x06b ExecuteOptions   : UChar
```

了解了关键的值，我们把这段汇编代码还原为 `c` 的代码如下：

```c
BOOLEAN

Ke386SetIoAccessMap (
    ULONG MapNumber,
    PKIO_ACCESS_MAP IoAccessMap
    )

{
    KIRQL oldIrql;
    if ((MapNumber >1) || (MapNumber == 0)) {
        return FALSE;
    oldIrql = KeRaiseIrqlToSynchLevel();
    PKPCR KiPcr = (PKPCR)0xffdff000;
    void *pIOPM = &(KiPcr ->TSS->IoMaps[0].IoMap);
    memcpy(pIOPM, IoAccessMap, 0x800);
    KiPcr->TSS->IoMapBase = PsGetCurrentProcess ()->IopmOffset;
    KfLowerIrql(oldIrql);
    return TRUE;
}
```

`lkd> u Ke386IoSetAccessProcess l 30`

`nt!Ke386IoSetAccessProcess:`

```asm
804f81d4 8bff             mov     edi,edi
804f81d6 55               push    ebp
804f81d7 8bec             mov     ebp,esp
804f81d9 56               push    esi
804f81da 8b750c           mov     esi,dword ptr [ebp+0Ch] //参数 2
804f81dd 83fe01           cmp     esi,1
804f81e0 7604             jbe     nt!Ke386IoSetAccessProcess+0x12 (804f81e6) //小于等于 1
804f81e2 32c0             xor     al,al
804f81e4 eb48             jmp     nt!Ke386IoSetAccessProcess+0x5a (804f822e) //大于 1
804f81e6 85f6             test    esi,esi
804f81e8 7507             jne     nt!Ke386IoSetAccessProcess+0x1d (804f81f1) ; //参数 2!=0
804f81ea beac200000       mov     esi,20ACh
804f81ef eb0c             jmp     nt!Ke386IoSetAccessProcess+0x29 (804f81fd)
804f81f1 69f624200000     imul    esi,esi,2024h
804f81f7 81ee9c1f0000     sub     esi,1F9Ch
804f81fd ff158c864d80     call    dword ptr [nt!_imp__KeRaiseIrqlToSynchLevel (804d868c)]
804f8203 8ac8             mov     cl,al
804f8205 8b4508           mov     eax,dword ptr [ebp+8]
804f8208 8b5034           mov     edx,dword ptr [eax+34h]
804f820b 66897030         mov     word ptr [eax+30h],si   //Process->IopmOffset
804f820f 3ea120f0dfff     mov     eax,dword ptr ds:[FFDFF020h]
804f8215 855014           test    dword ptr [eax+14h],edx
804f8218 740c             je      nt!Ke386IoSetAccessProcess+0x52 (804f8226)
804f821a b800f0dfff       mov     eax,0FFDFF000h
804f821f 8b4040           mov     eax,dword ptr [eax+40h]
804f8222 66897066         mov     word ptr [eax+66h],si
804f8226 ff151c874d80     call    dword ptr [nt!_imp_KfLowerIrql (804d871c)]
804f822c b001             mov     al,1
804f822e 5e               pop     esi
804f822f 5d               pop     ebp
804f8230 c20800           ret     8
804f8233 cc               int     3
```

还原为 `c` 的代码

```c
BOOLEAN  Ke386IoSetAccessProcess (
    PKPROCESS Process,
    ULONG MapNumber
    )

{
      WORD IOPMoffset;
      KIRQL oldIrql;
      if (MapNumber > 1)
      {
           return FALSE;
      }
      if(MapNumber == 0)
     {
         IOPMoffset = 0x20AC;
     }
     else
     {
         IOPMoffset = 0x2024 - 0x1F9C;
     }
     oldIrql = KeRaiseIrqlToSynchLevel();
     Process->IopmOffset = IOPMoffset;
     PKPCR KiPcr = (PKPCR)0xffdff000;
     if(KiPcr->Prcb->SetMember != Process->Pcb.ActiveProcessors)
     {
           KiPcr->TSS->IoMapBase = IOPMoffset ;
     }
     KfLowerIrql(oldIrql);
     return TRUE;
}
```

后面贴上一个著名的代码 `porttalk`，供大家学习。后面也附上偶的一个涂鸦之作。

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | `porttalk.rar` |

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | `modifyiopmbase.rar` |
