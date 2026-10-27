
### `PgLocatePoolObject`

```c
VOID
NTAPI
PgLocatePoolObject(
    __inout PPGBLOCK PgBlock,
    __in PPGOBJECT Object
)
{
    PFN_NUMBER Index = 0;
    PVOID Establisher = NULL;

    GetCounterBody(&Object->Establisher, &Establisher);

    for (Index = 0;
        Index < PgBlock->PoolBigPageTableSize;
        Index++) {
        if (POOL_BIG_TABLE_ENTRY_FREE !=
            ((ULONG64)PgBlock->PoolBigPageTable[Index].Va & POOL_BIG_TABLE_ENTRY_FREE)) {
            if (NonPagedPool == PgBlock->MmDeterminePoolType(PgBlock->PoolBigPageTable[Index].Va)) {
                if (PgBlock->PoolBigPageTable[Index].NumberOfPages > PgBlock->SizeINITKDBG) {
                    if ((ULONG64)Establisher >= (ULONG64)PgBlock->PoolBigPageTable[Index].Va &&
                        (ULONG64)Establisher < (ULONG64)PgBlock->PoolBigPageTable[Index].Va +
                        PgBlock->PoolBigPageTable[Index].NumberOfPages) {
                        Object->BaseAddress = PgBlock->PoolBigPageTable[Index].Va;
                        Object->RegionSize = PgBlock->PoolBigPageTable[Index].NumberOfPages;

#ifndef PUBLIC
                        DbgPrint(
                            "[Sefirot] [PatchGuard] < %p > found region in pool < %p - %08x >\n",
                            Establisher,
                            Object->BaseAddress,
                            Object->RegionSize);
#endif // !PUBLIC

                        Object->Type = PgPoolBigPage;

                        break;
                    }
                }
            }
        }
    }
}
```

### `PgLocateSystemPtesObject`

```c
VOID
NTAPI
PgLocateSystemPtesObject(
    __inout PPGBLOCK PgBlock,
    __in PPGOBJECT Object
)
{
    PRTL_BITMAP BitMap = NULL;
    ULONG BitMapSize = 0;
    PFN_NUMBER NumberOfPtes = 0;
    ULONG HintIndex = 0;
    ULONG StartingRunIndex = 0;
    PVOID Establisher = NULL;

    NumberOfPtes = PgBlock->NumberOfPtes;

    GetCounterBody(&Object->Establisher, &Establisher);

    BitMapSize =
        sizeof(RTL_BITMAP) +
        (ULONG)((((NumberOfPtes + 1) + 31) / 32) * 4);

    BitMap = ExAllocatePool(NonPagedPool, BitMapSize);

    if (NULL != BitMap) {
        RtlInitializeBitMap(
            BitMap,
            (PULONG)(BitMap + 1),
            (ULONG)(NumberOfPtes + 1));

        RtlClearAllBits(BitMap);

        InitializeSystemPtesBitMap(
            PgBlock->BasePte,
            NumberOfPtes,
            BitMap);

        do {
            HintIndex = RtlFindSetBits(
                BitMap,
                1,
                HintIndex);

            if (MAXULONG != HintIndex) {
                RtlFindNextForwardRunClear(
                    BitMap,
                    HintIndex,
                    &StartingRunIndex);

                RtlClearBits(BitMap, HintIndex, StartingRunIndex - HintIndex);

                if ((ULONG64)Establisher >=
                    (ULONG64)GetVirtualAddressMappedByPte(
                        PgBlock->BasePte + HintIndex) &&
                        (ULONG64)Establisher <
                    (ULONG64)GetVirtualAddressMappedByPte(
                        PgBlock->BasePte + StartingRunIndex) - PgBlock->SizeCmpAppendDllSection) {
                    Object->BaseAddress =
                        GetVirtualAddressMappedByPte(PgBlock->BasePte + HintIndex);

                    Object->RegionSize =
                        (SIZE_T)(StartingRunIndex - HintIndex) * PAGE_SIZE;

#ifndef PUBLIC
                    DbgPrint(
                        "[Sefirot] [PatchGuard] < %p > found region in system ptes < %p - %08x >\n",
                        Establisher,
                        Object->BaseAddress,
                        Object->RegionSize);
#endif // !PUBLIC

                    Object->Type = PgSystemPtes;

                    break;
                }

                HintIndex = StartingRunIndex;
            }
        } while (HintIndex < NumberOfPtes);

        ExFreePool(BitMap);
    }
}
```

### `PgLocateObject`

```c
VOID
NTAPI
PgLocateObject(
    __inout PPGBLOCK PgBlock,
    __out PPGOBJECT Object
)
{
    IpiSingleCall(
        (PPS_APC_ROUTINE)NULL,
        (PKSYSTEM_ROUTINE)PgLocatePoolObject,
        (PUSER_THREAD_START_ROUTINE)PgBlock,
        (PVOID)Object);

    if (-1 == Object->Type) {
        IpiSingleCall(
            (PPS_APC_ROUTINE)NULL,
            (PKSYSTEM_ROUTINE)PgLocateSystemPtesObject,
            (PUSER_THREAD_START_ROUTINE)PgBlock,
            (PVOID)Object);
    }
}
```

然后在内存块搜索 `SdbpCheckDll` 的代码来判断是不是 `PatchGuad` 执行逻辑, 如果是释放掉内存->重启 `WORKER`, 不是则恢复运行环境. 为了便于管理, 这里使用了对象的设计方法. 每个检测到的 `CONTEXT` 分配一个对象.
