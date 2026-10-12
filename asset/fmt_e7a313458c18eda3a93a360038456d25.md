
# 有关句柄分配算法

> 原文：[`www.pediy.com/kssd/pediy10/82982.html`](https://www.pediy.com/kssd/pediy10/82982.html)

一直在想 `windows` 下的句柄是怎么进行分配和管理的。

而这个句柄分配的算法又是 `windows` 不予公布的。

在网上找了下，发现这方面的资料很少啊，于是自己对着 `WRK` 和 `reactOS` 看了一下午。

虽然明白了一点点，但是还是有好多地方很迷糊，希望大牛们指导一下。

从 `ExCreateHandle` 开始，到 `ExpAllocateHandleEntry`，再到 `ExpAllocateHandleTableEntrySlow`。

定位句柄分配算法大概就位于此处。

```
if ( TableLevel == 0 ) {

    NewMidLevel = ExpAllocateMidLevelTable( HandleTable, DoInit, &NewLowLevel );

    if (NewMidLevel == NULL) {
        return FALSE;
    }

    NewMidLevel[1] = NewMidLevel[0];
    NewMidLevel[0] = (PHANDLE_TABLE_ENTRY)CapturedTable;

    CapturedTable = ((ULONG_PTR)NewMidLevel) | 1;
        
    OldValue = InterlockedExchangePointer( (PVOID *)&HandleTable->TableCode, (PVOID)CapturedTable );


} else if (TableLevel == 1) {

    PHANDLE_TABLE_ENTRY *TableLevel2 = (PHANDLE_TABLE_ENTRY *)CapturedTable;

    i = HandleTable->NextHandleNeedingPool / (LOWLEVEL_COUNT * HANDLE_VALUE_INC);

    if (i < MIDLEVEL_COUNT) {

        NewLowLevel = ExpAllocateLowLevelTable( HandleTable, DoInit );

        if (NewLowLevel == NULL) {
            return FALSE;
        }

        OldValue = InterlockedExchangePointer( (PVOID *) (&TableLevel2[i]), NewLowLevel );
        EXASSERT (OldValue == NULL);

    } else {

        NewHighLevel = ExpAllocateTablePagedPool( HandleTable->QuotaProcess,
                                                  HIGHLEVEL_SIZE
                                                 );

        if (NewHighLevel == NULL) {

            return FALSE;
        }
            
        NewMidLevel = ExpAllocateMidLevelTable( HandleTable, DoInit, &NewLowLevel );

        if (NewMidLevel == NULL) {
                
            ExpFreeTablePagedPool( HandleTable->QuotaProcess,
                                   NewHighLevel,
                                   HIGHLEVEL_SIZE
                                  );

            return FALSE;
        }

        NewHighLevel[0] = (PHANDLE_TABLE_ENTRY*)CapturedTable;
        NewHighLevel[1] = NewMidLevel;

        CapturedTable = ((ULONG_PTR)NewHighLevel) | 2;

        OldValue = InterlockedExchangePointer( (PVOID *)&HandleTable->TableCode, (PVOID)CapturedTable );

    }

} else if (TableLevel == 2) {

    ULONG RemainingIndex;
    PHANDLE_TABLE_ENTRY **TableLevel3 = (PHANDLE_TABLE_ENTRY **)CapturedTable;

    i = HandleTable->NextHandleNeedingPool / (MIDLEVEL_THRESHOLD * HANDLE_VALUE_INC);

    if (i >= HIGHLEVEL_COUNT) {

        return FALSE;
    }

    if (TableLevel3[i] == NULL) {

        NewMidLevel = ExpAllocateMidLevelTable( HandleTable, DoInit, &NewLowLevel );
            
        if (NewMidLevel == NULL) {
                
            return FALSE;
        }            

        OldValue = InterlockedExchangePointer( (PVOID *) &(TableLevel3[i]), NewMidLevel );
        EXASSERT (OldValue == NULL);

    } else {

        RemainingIndex = (HandleTable->NextHandleNeedingPool / HANDLE_VALUE_INC) -
                       i * MIDLEVEL_THRESHOLD;
        j = RemainingIndex / LOWLEVEL_COUNT;

        NewLowLevel = ExpAllocateLowLevelTable( HandleTable, DoInit );

        if (NewLowLevel == NULL) {

            return FALSE;
        }

        OldValue = InterlockedExchangePointer( (PVOID *)(&TableLevel3[i][j]) , NewLowLevel );
        EXASSERT (OldValue == NULL);
    }
}
```

大致流程简化如下。

```
if ( TableLevel == 0 ) 
{
    NewMidLevel = ExpAllocateMidLevelTable( HandleTable, DoInit, &NewLowLevel );
}
else if ( TableLevel == 1 )
{
    if (i < MIDLEVEL_COUNT)
    {
        NewLowLevel = ExpAllocateLowLevelTable( HandleTable, DoInit );
    }
    else
    {
        NewHighLevel = ExpAllocateTablePagedPool( HandleTable->QuotaProcess,
                                                  HIGHLEVEL_SIZE
                                                 );
        NewMidLevel = ExpAllocateMidLevelTable( HandleTable, DoInit, &NewLowLevel );
    }
}
else if (TableLevel == 2)
{
    if (i >= HIGHLEVEL_COUNT) 
            return FALSE;

    if (TableLevel3[i] == NULL)
         NewMidLevel = ExpAllocateMidLevelTable( HandleTable, DoInit, &NewLowLevel );
    else
        NewLowLevel = ExpAllocateLowLevelTable( HandleTable, DoInit );
}
```

思路可能是这样的：如果这一级句柄表满了以后就分配更高一级的句柄表，并修改 `HANDLE_TABLE` 结构中相应的部分使其指向高一级的句柄表，在高一级的句柄表中指向满掉的那个句柄表。

这里有一个问题：当 `level=0` 时，并没有做任何判断，就用 `ExpAllocateMidLevelTable` 分配了一个中级表，这是为什么呢？不是应该判断一下 `0` 级表有没有满吗？就像下面的都判断了 `i` 的值，这里为什么没有判断？

`ps`：不知道有没有句柄算法的更详细的一点资料？这是我所找到的关于句柄的文章，你看看吧。

## 上传的附件

`img/c8c74a380a3dea7fa2460187d80ee5e0.jpg`，`句柄相关资料.rar`

它是在这个 `ExpAllocateHandleTableEntry` 函数里判断，通过 `HandleTable->FirstFree` 来判断，如果为 `0` 句柄表就是满了，否则它就是指向可获取的句柄的值。而这个 `HandleTable->FirstFree` 的值来源于 `NextFreeTableEntry`，而 `NextFreeTableEntry` 初始化就看 `ExpAllocateLowLevelTable`。
