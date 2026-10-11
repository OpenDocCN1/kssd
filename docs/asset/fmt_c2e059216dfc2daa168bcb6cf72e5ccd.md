
## 2. 根据分页机制进行手工转换

进行手工转换，得到虚拟地址映射的物理地址，通过读取该物理地址获取目标进程虚拟地址的内容。详见附件分析。

获取 `CR3` 后，接下来根据分页机制，将虚拟地址转换为物理地址。

在转换之前，需要判断是否开启 `PAE`，非 `PAE` 和开启 `PAE` 的转换有所不同。

控制寄存器 `CR4` 的第 5 位标记是否开启 `PAE`。

代码：

```
// 获得 CR4 的值
__asm
{
  _emit 0x0F 
  _emit 0x20
  _emit 0xE0
  mov uCR4, eax
}
```

### 未开启 PAE 情况

通过 `CR3` 寄存器定位到页目录的基地址。

线性地址的高 10 位作为获取页目录表项的索引，获得一个页目录的一个表项。

注：Windows 的保护实现基本不使用分段机制，主要是通过分页机制来实现保护，这里的线性地址等于虚拟地址。

代码：

```
// 获得页目录表项(PDE)
dwPageDirIndex = (dwVirtualAddr & 0xffc00000) >> 22;
DWORD dwPageDirEntry = ReadPageDirEntryNoPAE(dwPageDirIndex);
if (dwPageDirEntry == 0)
{
    return;
}
```

根据 `PDE` 获得页表基地址或者页基地址。

当没有开启 `PAE` 时，有两种 `PDE` 格式，分别指向 `4KB` 的页表和 `4MB` 的内存页。

![](img/8332fdb2bc981048ace539e5e21c2fef.jpg)

![](img/b83898e9c97b87af41834492d7a9b866.jpg)

页目录项的第 7 位判断页大小。

代码：

```
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

#### 4KB 情况

`4KB` 时，页目录项的高 20 位为页表基地址。线性地址的第 12 位到 21 位作为选取页表的一个表项（`PTE`）。

![](img/59951de2e91ec74a7dc32bafa2927cb5.jpg)

代码：

```
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

代码：

```
// 页的基地址
DWORD dwPageBaseAddr = dwPageTable & 0xfffff000;
dwPageOffset = dwVirtualAddr & 0x00000fff;
```

这时获得的物理地址就是想要的目标进程虚拟地址映射的物理地址。

代码：

```
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

#### 4MB 情况

`4MB` 时，页目录项的高 10 位作为页的基地址，线性地址的低 22 位作为物理地址在页中的偏移。

![](img/9ad6f1805fd361cec9bfee6bdea556c8.jpg)

代码：

```
// 线性地址的低 22 位是页内偏移
DWORD dwPageOffsetMB = dwVirtualAddr & 0x003fffff;
// 高 10 位,是页基地址
DWORD dwPageBaseAddr = dwPageDirEntry & 0xffc00000;
```

这时获得的物理地址就是想要的目标进程虚拟地址映射的物理地址。

代码：

```
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

### 开启 PAE 情况

`CR4` 中的物理地址扩展（`PAE`）标志可以开启 `PAE` 机制，将物理地址从 32 位扩展到 36 位。

当开启 `PAE` 机制后，处理器支持两种尺寸的页：`4KB` 和 `2MB` 的页。

增加了页目录指针表项。

- 表项的大小从 32 位增加到了 64 位。
- 表项中的物理基地址扩展到了 24 位。
- 寄存器 `CR3` 中的高位页目录基地址被换为 27 位的页目录指针表基地址。

![](img/82062069f0a8bd2b1aebeb04e7647c7b.jpg)

通过 `CR3` 寄存器定位到页目录指针表起始地址，取线性地址的高 2 位作为选取页目录指针表项的索引。

代码：

```
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

取线性地址的第 21 位到 29 位作为页目录索引，页目录指针表项的第 12 位到第 35 位为页目录基地址。

![](img/f7595058d084f184388262dc4eeb7d62.jpg)

代码：

```
// 页目录基地址
DWORD dwDirBaseAddr = (DWORD)(nPageDirPointerTable &0x0000000ffffff000);
// 页目录项索引
DWORD dwPageDirIndex = (dwVirtualAddr & 0x3fe00000) >> 21;

__int64 nPageDirEntry = ReadPageDirEntryPAE(dwPageDirIndex, dwDirBaseAddr);

if (nPageDirEntry == 0)
{
    return;
}
```

根据页目录项的第 7 位判断页大小，`2MB` 还是 `4KB`。

代码：

```
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

#### 4KB 情况

`4KB` 时，页目录项的第 12 位到 35 位作为页表基地址的高 24 位。取线性地址第 12 位到 20 位作为在页表中的偏移。

![](img/40d8dbf297e625d455b9824c1319f64b.jpg)

代码：

```
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

代码：

```
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

所得到的内容就是目标进程虚拟地址内存的数据。

#### 2MB 情况

`2MB` 时，页目录项的第 21 位到 35 位作为页的基地址的高位，取线性地址的第 0 位到第 20 位做为物理地址在页中的偏移。

![](img/36a6c672c487534d274b5a72913ea1ee.jpg)

代码：

```
DWORD dwPageOffsetMB = dwVirtualAddr & 0x000fffff;
// 高 35 位
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

所得到的内容就是目标进程虚拟地址内存的数据。

这里谢谢钱老师，近一年的培养，祝钱老师的培训越办越好！

### 附件

![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg)

`源码.rar`
