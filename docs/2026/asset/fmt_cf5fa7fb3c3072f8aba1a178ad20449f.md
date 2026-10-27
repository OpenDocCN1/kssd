
```cpp
__try
{
    ReadMemoryInfo *pInfo = 
        (ReadMemoryInfo *)ExAllocatePool(PagedPool, sizeof(ReadMemoryInfo));
    RtlCopyMemory(pInfo, pIoBuffer, sizeof(ReadMemoryInfo));
    ULONG uOldCr3 = 0;
    ULONG uCurrentCr3 = *(PULONG)(pInfo->nEprocess + 0x18);
    if (pInfo->nMemoryAddr == 0)
    {
        status = STATUS_UNSUCCESSFUL;
        break;
    }
    __asm
    {
        mov eax, cr3
        mov uOldCr3, eax

        mov eax, uCurrentCr3
        mov cr3, eax
    }
    RtlCopyMemory(pIoBuffer, 
                  (PVOID)pInfo->nMemoryAddr, pInfo->nReadSize);
    uOutSize = pInfo->nReadSize;
    __asm
    {
        mov eax, uOldCr3
        mov cr3, eax
    }

    if (pInfo != NULL)
    {
        ExFreePool(pInfo);
        pInfo = NULL;
    }
    // Return success
    status = STATUS_SUCCESS;
}
__except(1)
{
    status = STATUS_UNSUCCESSFUL;
}
```

## 2. 根据分页机制，进行手工转换

得到虚拟地址映射到的物理地址，读取该物理地址得到目标进程虚拟地址的内容，详见附件分析。

获得 `CR3` 后，接下来就是根据分页机制把虚拟地址转换为物理地址。在转换之前要判断是否开启 `PAE`：未开启 `PAE` 和开启 `PAE` 的转换有所不同。控制寄存器 `CR4` 的第 5 位标记是否开启 `PAE`。

```cpp
// 获得 CR4 的值
__asm
{
    _emit 0x0F
    _emit 0x20
    _emit 0xE0
    mov uCR4, eax
}
```

### 未开启 `PAE` 情况

通过 `CR3` 寄存器定位到页目录的基地址。线性地址的高 10 位作为获取页目录表项的索引，获得一个页目录表项。

> 注：Windows 的保护实现基本不使用分段机制，主要是通过分页机制来实现保护，这里的线性地址等于虚拟地址。

```cpp
// 获得页目录表项(PDE)
dwPageDirIndex = (dwVirtualAddr & 0xffc00000) >> 22;
DWORD dwPageDirEntry = ReadPageDirEntryNoPAE(dwPageDirIndex);
if (dwPageDirEntry == 0)
{
    return;
}
```

根据 `PDE` 获得页表基地址或者页基地址。当没有开启 `PAE` 时，有两种 `PDE` 格式，分别指向 4KB 的页表和 4MB 的内存页。

![](https://bbs.kanxue.com/upload/attach/200906/123563_7x9rsfgxqy1k5z7.jpg)
![](https://bbs.kanxue.com/upload/attach/200906/123563_ovmn64xwlzrs0gm.jpg)

页目录项的第 7 位判断页大小。

```cpp
// 获得页大小
DWORD CReadMemoryDlg::GetPageSizeNoPAE(DWORD dwAddr)
{
    if ((dwAddr & 0x00000080) == 0x00000080)
    {
        return MBSIZE;
    }
    else
    {
        return KBSIZE;
    }
}
```

4KB 时，页目录项的高 20 位为页表基地址，线性地址的第 12 位到第 21 位作为选取页表的一个表项（`PTE`）。

![](https://bbs.kanxue.com/upload/attach/200906/123563_zc84cqapft4o5n4.jpg)

```cpp
dwPageTableIndex = (dwVirtualAddr & 0x0003ff000) >> 12;
DWORD dwPageTableBaseAddr = dwPageDirEntry & 0xfffff000;
// 获得页表
DWORD dwPageTable = ReadPageTableNoPAE(dwPageTableBaseAddr, 
                                       dwPageTableIndex);
if (dwPageTable == 0)
{
    return;
}
if (IsPresentNoPAE(dwPageTable) == FALSE)
{
    return;
}
```

取 `PTE` 的高 20 位作为内存页的基地址，线性地址的低 12 位作为页中偏移，得到物理地址。

```cpp
// 页的基地址
DWORD dwPageBaseAddr = dwPageTable & 0xfffff000;
dwPageOffset = dwVirtualAddr & 0x00000fff;
```

这时获得的物理地址就是目标进程虚拟地址所映射的物理地址。

```cpp
PVOID pReadBuf = new BYTE[dwReadSize];

BOOL bRet = ReadPageMemoryNoPAE(pReadBuf, 
                                dwPageOffset, 
                                dwReadSize, 
                                dwPageBaseAddr);
if (bRet == FALSE)
{
    return;
}
```

4MB 时，页目录项的高 10 位作为页的基地址，线性地址的低 22 位作为物理地址在页中的偏移。

![](https://bbs.kanxue.com/upload/attach/200906/123563_x0abfa722naz5ji.jpg)

```cpp
// 线性地址的低 22 位是页内偏移
DWORD dwPageOffsetMB = dwVirtualAddr & 0x003fffff;
// 高 10 位是页基地址
DWORD dwPageBaseAddr = dwPageDirEntry & 0xffc00000;
```

这时获得的物理地址就是目标进程虚拟地址所映射的物理地址。

```cpp
PVOID pReadBuf = new BYTE[dwReadSize];

BOOL bRet = ReadPageMemoryPAE(pReadBuf, 
                              dwPageOffsetMB, 
                              dwReadSize, 
                              dwPageBaseAddr);
if (bRet == FALSE)
{
    return;
}
```

### 开启 `PAE` 情况

`CR4` 中的物理地址扩展（`PAE`）标志可以开启 `PAE` 机制，将物理地址从 32 位扩展到 36 位。开启 `PAE` 机制后，处理器支持两种尺寸的页：4KB 和 2MB 的页，并增加了页目录指针表项。表项的大小从 32 位增加到 64 位，表项中的物理基地址扩展到 24 位，寄存器 `CR3` 中的高位页目录基地址被换为 27 位的页目录指针表基地址。

![](https://bbs.kanxue.com/upload/attach/200906/123563_d1hg3wbn1tapy6j.jpg)

通过 `CR3` 寄存器定位到页目录指针表起始地址，取线性地址的高 2 位作为选取页目录指针表项的索引。

```cpp
DWORD dwDirPointerTableIndex = (dwVirtualAddr & 0xc0000000) >> 30;
DWORD dwDirPointerTableBaseAddr = m_nDirBase & 0xffffffe0;
// 获得页目录指针表
__int64 nPageDirPointerTable = ReadPageDirPAE(dwDirPointerTableIndex, 
                                              dwDirPointerTableBaseAddr);
if (nPageDir == 0)
{
    return;
}
```

取线性地址的第 21 位到第 29 位作为页目录索引，页目录指针表项的第 12 位到第 35 位为页目录基地址。

![](https://bbs.kanxue.com/upload/attach/200906/123563_pjutu7c8doxdwb7.jpg)

```cpp
// 页目录基地址
DWORD dwDirBaseAddr = (DWORD)(nPageDirPointerTable & 0x0000000ffffff000);
// 页目录项索引
DWORD dwPageDirIndex = (dwVirtualAddr & 0x3fe00000) >> 21;

__int64 nPageDirEntry = ReadPageDirEntryPAE(dwPageDirIndex, dwDirBaseAddr);

if (nPageDirEntry == 0)
{
    return;
}
```

根据页目录项的第 7 位判断页大小是 2MB 还是 4KB。

```cpp
// 获得页大小
DWORD CReadMemoryDlg::GetPageSizePAE(__int64 nAddr)
{
    if ((nAddr & 0x0000000000000080) == 0x0000000000000080)
    {
        return MBSIZE;
    }
    else
    {
        return KBSIZE;
    }
}
```

4KB 时，页目录项的第 12 位到第 35 位作为页表基地址的高 24 位，取线性地址第 12 位到第 20 位作为在页表中的偏移。

![](https://bbs.kanxue.com/upload/attach/200906/123563_mo4a9uxj20toof0.jpg)

```cpp
// 获得页表
DWORD dwPageTableBaseAddr = (DWORD)(nPageDirEntry & 0x0000000ffffff000);
DWORD dwPageTableIndex = (dwVirtualAddr & 0x001ff000) >> 12;
__int64 nPageTable = ReadPageTablePAE(dwPageTableIndex, dwPageTableBaseAddr);

if (nPageTable == 0)
{
    return;
}
```

通过页表项的第 12 位到第 35 位作为页基地址的高 24 位，线性地址的低 12 位作为在页中的偏移。

```cpp
// 读取页内容
DWORD dwPageBaseAddr = (DWORD)(nPageTable & 0x0000000ffffff000);
DWORD dwPageOffsetKb = dwVirtualAddr & 0x00000fff;
PVOID pReadBuf = new BYTE[dwReadSize];

BOOL bRet = ReadPageMemoryPAE(pReadBuf, 
                              dwPageOffsetKb, 
                              dwReadSize, 
                              dwPageBaseAddr);
if (bRet == FALSE)
{
    return;
}
```

所得到的内容就是目标进程虚拟地址内存中的数据。

2MB 时，页目录项的第 21 位到第 35 位作为页的基地址的高位，取线性地址的第 0 位到第 20 位作为物理地址在页中的偏移。

![](https://bbs.kanxue.com/upload/attach/200906/123563_n5bofezfc2ibjoe.jpg)

```cpp
DWORD dwPageOffsetMB = dwVirtualAddr & 0x000fffff;
// 高-35位
DWORD dwDirBaseAddr = (DWORD)(nPageDirEntry & 0x00000007fff00000);
PVOID pReadBuf = new BYTE[dwReadSize];

BOOL bRet = ReadPageMemoryPAE(pReadBuf, 
                              dwPageOffsetMB, 
                              dwReadSize, 
                              dwDirBaseAddr);
if (bRet == FALSE)
{
    return;
}
```

所得到的内容就是目标进程虚拟地址内存中的数据。

这里谢谢钱老师这近一年的培养，祝钱老师的培训越办越好！

[\[招生\]科锐逆向工程师培训（2026年11月13日实地，远程教学同时开班，第57期）](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [1.jpg](javascript:void(0);)（19.33kb，3935次下载）
- [819373657206152704.jpg](javascript:void(0);)（17.36kb，3928次下载）
- [3.jpg](javascript:void(0);)（42.90kb，3923次下载）
- [4.jpg](javascript:void(0);)（24.25kb，3913次下载）
- [5.jpg](javascript:void(0);)（9.42kb，3898次下载）
- [6.jpg](javascript:void(0);)（10.69kb，3896次下载）
- [7.jpg](javascript:void(0);)（29.50kb，3908次下载）
- [8.jpg](javascript:void(0);)（25.37kb，3896次下载）
- [源码.rar](javascript:void(0);)（670.73kb，1202次下载）
