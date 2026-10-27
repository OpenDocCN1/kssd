
# [原创]x64内核实验6-进程

[TOC](这里写目录标题)

## 进程结构体头文件（很长可以直接看后面的重要属性介绍）

首先来看一下进程结构体完整的定义，然后我会介绍几个比较重要的字段属性。

下面是我从pdb文件导出的头文件，这个头文件可以直接复制到ida里导入成结构体使用的，其他结构体的头文件后面我会一起放出来给大家下载。

`KEPROCESS.h`

```
typedef struct \_KPROCESS // 54 elements, 0x438 bytes (sizeof)
{
/\*0x000\*/ struct \_DISPATCHER\_HEADER Header; // 59 elements, 0x18 bytes (sizeof)
/\*0x018\*/ struct \_LIST\_ENTRY ProfileListHead; // 2 elements, 0x10 bytes (sizeof)
/\*0x028\*/ UINT64 DirectoryTableBase;
/\*0x030\*/ struct \_LIST\_ENTRY ThreadListHead; // 2 elements, 0x10 bytes (sizeof)
/\*0x040\*/ ULONG32 ProcessLock;
/\*0x044\*/ ULONG32 ProcessTimerDelay;
/\*0x048\*/ UINT64 DeepFreezeStartTime;
/\*0x050\*/ struct \_KAFFINITY\_EX Affinity; // 4 elements, 0xA8 bytes (sizeof)
/\*0x0F8\*/ UINT64 AffinityPadding[12];
/\*0x158\*/ struct \_LIST\_ENTRY ReadyListHead; // 2 elements, 0x10 bytes (sizeof)
/\*0x168\*/ struct \_SINGLE\_LIST\_ENTRY SwapListEntry; // 1 elements, 0x8 bytes (sizeof)
/\*0x170\*/ struct \_KAFFINITY\_EX ActiveProcessors; // 4 elements, 0xA8 bytes (sizeof)
/\*0x218\*/ UINT64 ActiveProcessorsPadding[12];
union // 2 elements, 0x4 bytes (sizeof)
{
struct // 10 elements, 0x4 bytes (sizeof)
{
/\*0x278\*/ ULONG32 AutoAlignment : 1; // 0 BitPosition
/\*0x278\*/ ULONG32 DisableBoost : 1; // 1 BitPosition
/\*0x278\*/ ULONG32 DisableQuantum : 1; // 2 BitPosition
/\*0x278\*/ ULONG32 DeepFreeze : 1; // 3 BitPosition
/\*0x278\*/ ULONG32 TimerVirtualization : 1; // 4 BitPosition
/\*0x278\*/ ULONG32 CheckStackExtents : 1; // 5 BitPosition
/\*0x278\*/ ULONG32 CacheIsolationEnabled : 1; // 6 BitPosition
/\*0x278\*/ ULONG32 PpmPolicy : 3; // 7 BitPosition
/\*0x278\*/ ULONG32 VaSpaceDeleted : 1; // 10 BitPosition
/\*0x278\*/ ULONG32 ReservedFlags : 21; // 11 BitPosition
};
/\*0x278\*/ LONG32 ProcessFlags;
};
/\*0x27C\*/ ULONG32 ActiveGroupsMask;
/\*0x280\*/ CHAR BasePriority;
/\*0x281\*/ CHAR QuantumReset;
/\*0x282\*/ CHAR Visited;
/\*0x283\*/ union \_KEXECUTE\_OPTIONS Flags; // 10 elements, 0x1 bytes (sizeof)
/\*0x284\*/ UINT16 ThreadSeed[20];
/\*0x2AC\*/ UINT16 ThreadSeedPadding[12];
/\*0x2C4\*/ UINT16 IdealProcessor[20];
/\*0x2EC\*/ UINT16 IdealProcessorPadding[12];
/\*0x304\*/ UINT16 IdealNode[20];
/\*0x32C\*/ UINT16 IdealNodePadding[12];
/\*0x344\*/ UINT16 IdealGlobalNode;
/\*0x346\*/ UINT16 Spare1;
/\*0x348\*/ union \_KSTACK\_COUNT StackCount; // 3 elements, 0x4 bytes (sizeof)
/\*0x34C\*/ UINT8 \_PADDING0\_[0x4];
/\*0x350\*/ struct \_LIST\_ENTRY ProcessListEntry; // 2 elements, 0x10 bytes (sizeof)
/\*0x360\*/ UINT64 CycleTime;
/\*0x368\*/ UINT64 ContextSwitches;
/\*0x370\*/ struct \_KSCHEDULING\_GROUP\* SchedulingGroup;
/\*0x378\*/ ULONG32 FreezeCount;
/\*0x37C\*/ ULONG32 KernelTime;
/\*0x380\*/ ULONG32 UserTime;
/\*0x384\*/ ULONG32 ReadyTime;
/\*0x388\*/ UINT64 UserDirectoryTableBase;
/\*0x390\*/ UINT8 AddressPolicy;
/\*0x391\*/ UINT8 Spare2[71];
/\*0x3D8\*/ VOID\* InstrumentationCallback;
/\*0x3E0\*/ union \_<anonymous-tag> SecureState; // 2 elements, 0x8 bytes (sizeof)
/\*0x3E8\*/ UINT64 KernelWaitTime;
/\*0x3F0\*/ UINT64 UserWaitTime;
/\*0x3F8\*/ UINT64 EndPadding[8];
}KPROCESS, \*PKPROCESS;
```

`EPROCESS.h`
