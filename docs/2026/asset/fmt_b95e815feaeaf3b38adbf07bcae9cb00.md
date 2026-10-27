
```c
else
{
if ( var\_PoolType == 1 && NeedSize <= 0x20 )
{
KPRCB = KeGetPcr()->Prcb;
var\_LookasideList = KPRCB->PPLookasideList[NeedSize + 15].P;
++\*((\_DWORD \*)var\_LookasideList + 3);
NumberOfBytesa = (SIZE\_T)KPRCB;
v8 = ExInterlockedPopEntrySList(var\_LookasideList);
v9 = v8;
// 省略代码，找到则返回结果
}
if ( (unsigned int)ExpNumberOfNonPagedPools > 1 )
{
tmpPoolIndex = KeGetPcr()->Prcb->ParentNode->Color;
PoolIndex = tmpPoolIndex;
if ( tmpPoolIndex >= ExpNumberOfNonPagedPools )
{
PoolIndex = ExpNumberOfNonPagedPools - 1;
tmpPoolIndex = ExpNumberOfNonPagedPools - 1;
}
// 获取非分页内存对象
PoolDesc = ExpNonPagedPoolDescriptor[tmpPoolIndex];
goto LABEL\_17;
}
}
```

如果是要在非分页内存对象中申请内存，最后会跳转到上面使用分页内存对象来申请内存时候的`LABEL_17`执行，而`LABEL17`则跳转到`LABEL_18`执行剩下的代码

剩下的代码就是先从链表中寻找内存块，如果没有找到可用的内存块，那么就会通过调用函数`MiAllocatePoolPages`来申请新的内存块，而多申请的内存块也会被加入到链表中，而内存块链接进链表的位置是`POOL_HEADER`以后的八字节，也就是说池对象头后就是用来链接进链表的双向链表，以下是`WRK`中关于这部分内容的部分代码：

```c
ListHead = &PoolDesc->ListHeads[ListNumber];
do {
// 检查链表中是否有可用的内存块， FALSE为有
if (PrivateIsListEmpty (ListHead) == FALSE) {
// 获取内存块
Block = PrivateRemoveHeadList (ListHead);
// 指向POOL\_HEADER，此处的POOL\_OVERHEAD为0x8
Entry = (PPOOL\_HEADER)((PCHAR)Block - POOL\_OVERHEAD);
// 内存块大小是和需要的内存大小不一致
if (Entry->BlockSize != NeededSize) {
if (Entry->PreviousSize == 0) {
// 页面开始处，取前面一块作为返回结果的内存块，保存在Entry中
// 剩余部分保存在SplitEntry中
SplitEntry = (PPOOL\_HEADER)((PPOOL\_BLOCK)Entry + NeededSize);
SplitEntry->BlockSize = (USHORT)(Entry->BlockSize - NeededSize);
SplitEntry->PreviousSize = (USHORT) NeededSize;
// 获取下一内存块
NextEntry = (PPOOL\_HEADER)((PPOOL\_BLOCK)SplitEntry + SplitEntry->BlockSize);
if (PAGE\_END(NextEntry) == FALSE) {
NextEntry->PreviousSize = SplitEntry->BlockSize;
}
}
else {
// 非页面开始处，取后面一块作为返回结果的内存块，保存在Entry中
// 剩余部分保存在SplitEntry中
SplitEntry = Entry;
Entry->BlockSize = (USHORT)(Entry->BlockSize - NeededSize);
Entry = (PPOOL\_HEADER)((PPOOL\_BLOCK)Entry + Entry->BlockSize);
Entry->PreviousSize = SplitEntry->BlockSize;
NextEntry = (PPOOL\_HEADER)((PPOOL\_BLOCK)Entry + NeededSize);
if (PAGE\_END(NextEntry) == FALSE) {
NextEntry->PreviousSize = (USHORT) NeededSize;
}
}
// 为内存块赋值
Entry->BlockSize = (USHORT) NeededSize;
SplitEntry->PoolType = 0;
Index = SplitEntry->BlockSize;
if ((POOL\_OVERHEAD != POOL\_SMALLEST\_BLOCK) ||
(SplitEntry->BlockSize != 1)) {
// 将剩余内存块加入到链表中
PrivateInsertTailList(&PoolDesc->ListHeads[Index - 1], ((PLIST\_ENTRY)((PCHAR)SplitEntry + POOL\_OVERHEAD)));
}
}
// 为申请的内存块的POOL\_HEADER赋值
Entry->PoolType = (UCHAR)(((PoolType & (BASE\_POOL\_TYPE\_MASK | POOL\_QUOTA\_MASK | SESSION\_POOL\_MASK | POOL\_VERIFIER\_MASK)) + 1) | POOL\_IN\_USE\_MASK);
InterlockedIncrement ((PLONG)&PoolDesc->RunningAllocs);
InterlockedExchangeAddSizeT (&PoolDesc->TotalBytes,
Entry->BlockSize << POOL\_BLOCK\_SHIFT);
Entry->PoolTag = Tag;
((PULONGLONG)((PCHAR)Entry + CacheOverhead))[0] = 0;
// 将申请到的内存地址返回，此时的CacheOverhead为0x8，所以会略过POOL\_HEADER
return (PCHAR)Entry + CacheOverhead;
}
// 指向下一个链表
ListHead += 1;
// 是否是最后一个链表
} while (ListHead != &PoolDesc->ListHeads[POOL\_LIST\_HEADS]);
// 如果没有找到可用内存块，就通过MiAllocatePoolPages来申请一块内存
Entry = (PPOOL\_HEADER) MiAllocatePoolPages (RequestType, PAGE\_SIZE);
// 为申请的内存块赋值
Entry->Ulong1 = 0;
Entry->PoolIndex = (UCHAR) PoolIndex;
Entry->BlockSize = (USHORT) NeededSize;
Entry->PoolType = (UCHAR)(((PoolType & (BASE\_POOL\_TYPE\_MASK | POOL\_QUOTA\_MASK | SESSION\_POOL\_MASK | POOL\_VERIFIER\_MASK)) + 1) | POOL\_IN\_USE\_MASK);
// 指向剩余内存块
SplitEntry = (PPOOL\_HEADER)((PPOOL\_BLOCK)Entry + NeededSize);
SplitEntry->Ulong1 = 0;
Index = (PAGE\_SIZE / sizeof(POOL\_BLOCK)) - NeededSize;
SplitEntry->BlockSize = (USHORT) Index;
SplitEntry->PreviousSize = (USHORT) NeededSize;
SplitEntry->PoolIndex = (UCHAR) PoolIndex;
InterlockedIncrement ((PLONG)&PoolDesc->TotalPages);
NeededSize <<= POOL\_BLOCK\_SHIFT;
InterlockedExchangeAddSizeT (&PoolDesc->TotalBytes, NeededSize);
if ((POOL\_OVERHEAD != POOL\_SMALLEST\_BLOCK) ||
(SplitEntry->BlockSize != 1)) {
// 将剩余的内存块加入到链表中
PrivateInsertTailList(&PoolDesc->ListHeads[Index - 1], ((PLIST\_ENTRY)((PCHAR)SplitEntry + POOL\_OVERHEAD)));
}
InterlockedIncrement ((PLONG)&PoolDesc->RunningAllocs);
// 指向申请的内存块的内存地址，此时的CacheOverhead为0x8，所以会略过POOL\_HEADER
Block = (PVOID) ((PCHAR)Entry + CacheOverhead);
Entry->PoolTag = Tag;
return Block;
```

因此，可得出`ExAllocatePoolWithTag`函数基本流程如下：

1. 如果申请的是大页面，直接调用`MiAllocatePoolPages`分配池内存
2. 如果请求小页面，尝试从当前处理器控制块后备链表分配
3. 如果后备链表分配失败，在从`PoolVector`指向的池描述符512条链表中分配
4. 如果池描述符链表分配也失败，在调用`MiAllocatePoolPages`分配

## 4.释放池内存

池内存的释放是通过`ExFreePool`来实现的，函数首先会判断要释放的池内存是否是大页面内存，由于大页面内存的地址是按页对齐的，低12位为0，所以此时如果与`0xFFF`与运算以后的结果为0，则说明是大页面池内存

```nasm
POOLCODE:0047A202 test word ptr [ebp+P], 0FFFh ; 是否是大页面内存
POOLCODE:0047A208 jz loc\_47A7E7
```

对于大页面池内存，会通过`MiFreePoolPages`来释放

```nasm
POOLCODE:0047A7E7 loc\_47A7E7: ; CODE XREF: ExFreePoolWithTag(x,x)+22↑j
POOLCODE:0047A7E7 ; ExFreePoolWithTag(x,x)+C53↓j
POOLCODE:0047A7E7 push [ebp+P]
POOLCODE:0047A7EA call \_MmDeterminePoolType@4 ; MmDeterminePoolType(x)
POOLCODE:0047A7EF mov esi, \_ExpSessionPoolDescriptor
POOLCODE:0047A7F5 mov ebx, eax
POOLCODE:0047A7F7 cmp ebx, 21h
POOLCODE:0047A7FA jz short loc\_47A803
POOLCODE:0047A7FC mov esi, \_PoolVector[ebx\*4]
POOLCODE:0047A803
POOLCODE:0047A803 loc\_47A803: ; CODE XREF: ExFreePoolWithTag(x,x)+614↑j
POOLCODE:0047A803 cmp \_PoolTrackTable, 0
POOLCODE:0047A80A jnz loc\_47AEDC
POOLCODE:0047A810
POOLCODE:0047A810 loc\_47A810: ; CODE XREF: ExFreePoolWithTag(x,x)+CF9↓j
POOLCODE:0047A810 ; ExFreePoolWithTag(x,x)+D57↓j
POOLCODE:0047A810 test byte ptr [esi], 1
POOLCODE:0047A813 jz loc\_47A8ED
POOLCODE:0047A819 mov ecx, [esi+1Ch] ; FastMutex
POOLCODE:0047A81C call ds:\_\_imp\_@ExAcquireFastMutex@4 ; ExAcquireFastMutex(x)
POOLCODE:0047A822
POOLCODE:0047A822 loc\_47A822: ; CODE XREF: ExFreePoolWithTag(x,x)+71F↓j
POOLCODE:0047A822 ; ExFreePoolWithTag(x,x)+D68↓j
POOLCODE:0047A822 push [ebp+P] ; BugCheckParameter2
POOLCODE:0047A825 inc dword ptr [esi+0Ch]
POOLCODE:0047A828 call \_MiFreePoolPages@4 ; MiFreePoolPages(x)
POOLCODE:0047A82D jmp short loc\_47A7BE
```

和申请池内存类似，如果释放的是小页面池内存，就会尝试将池内存交还给当前处理器控制块的后备链表

如果不是上面的两种情况，函数就会判断要释放的内存块的上一内存块和下一内存块是否在被使用中，如果没有被使用，就会将它们合并成为一个内存块。合并的内存块如果刚好一个页的大小，函数就会调用`MiFreePoolPages`释放内存，否则的话就会将其加入到链表中
