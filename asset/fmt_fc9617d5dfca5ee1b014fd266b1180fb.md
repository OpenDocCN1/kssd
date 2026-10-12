
# 逆向 MmGetPhysicalAddress 并正向

> 原文：[`www.pediy.com/kssd/pediy11/125037.html`](https://www.pediy.com/kssd/pediy11/125037.html)

这两天想做点坏事，也没啥，就是 `PTE` 欺骗啦。

其中涉及了 `VirtualAddress(VA)` 转化 `PhysicalAddress(PA)` 的生成原理。

于是网上找了些相关资料，很努力地看。

但看来看去，怎么和 `MmGetPhysicalAddress` 逆向结果不大一样啊。

这才发现，大部分现存的资料都很旧了。虽然应用原理还是差不多，但是已经不能被我们伸手党使用了。

于是决定逆向一下 `MmGetPhysicalAddress`，再正向它，尝试 `C++` 实现之。

## \_HARDWARE\_PTE 结构

代码:

```
lkd> dt \_hardware\_pte
nt!\_HARDWARE\_PTE
   +0x000 Valid            : Pos 0, 1 Bit
   +0x000 Write            : Pos 1, 1 Bit
   +0x000 Owner            : Pos 2, 1 Bit
   +0x000 WriteThrough     : Pos 3, 1 Bit
   +0x000 CacheDisable     : Pos 4, 1 Bit
   +0x000 Accessed         : Pos 5, 1 Bit
   +0x000 Dirty            : Pos 6, 1 Bit
   +0x000 LargePage        : Pos 7, 1 Bit
   +0x000 Global           : Pos 8, 1 Bit
   +0x000 CopyOnWrite      : Pos 9, 1 Bit
   +0x000 Prototype        : Pos 10, 1 Bit
   +0x000 reserved0        : Pos 11, 1 Bit
   +0x000 PageFrameNumber  : Pos 12, 26 Bits
   +0x000 reserved1        : Pos 38, 26 Bits
   +0x000 LowPart          : Uint4B
   +0x004 HighPart         : Uint4B
```

由此，可以列出如下结构：

代码:

```
typedef struct \_HARDWARE\_PTE\_X86PAE
  {
  union
    {
    struct
      {
      ULONGLONG Valid:1;
      ULONGLONG Write:1;
      ULONGLONG Owner:1;
      ULONGLONG WriteThrough:1;
      ULONGLONG CacheDisable:1;
      ULONGLONG Accessed:1;
      ULONGLONG Dirty:1;
      ULONGLONG LargePage:1;
      ULONGLONG Global:1;
      ULONGLONG CopyOnWrite:1;
      ULONGLONG Prototype:1;
      ULONGLONG reserved0:1;
      ULONGLONG PageFrameNumber:26;
      ULONGLONG reserved1:26;
      };
    struct
      {
      ULONG LowPart;
      ULONG HighPart;
      };
    };
  } HARDWARE\_PTE\_X86PAE, \*PHARDWARE\_PTE\_X86PAE,MMPTE\_HARDWARE, \*PMMPTE\_HARDWARE,\_MMPTE\_HARDWARE,HARDWARE\_PTE,\*PHARDWARE\_PTE;
```

注意，由于 `PAE` 模式下，`PTE` 结构突破 `32` 位，所以结构体使用位域的时候，需要用 `__int64` 来做(`2` `byte`)。

之前我做 `PTE` 结构时，使用 `ulong` 来做位域，无论是否强制结构体对齐，结果编译器硬要把这个结构识别为 `3` `byte`。汗死。

直到后面 `xIkUg` 大哥提供旧式结构体后，我才发现之间的差异。

不过 `xIkUg` 大哥提供的 `PTE` 结构指明 `PageFrameNumber:24`;很奇怪。我在 `XP` 和 `2003` 上列表的结构都指明 `PageFrameNumber:26`。所以，在实际应用时，大家还是用 `windbg` 看看自己的 `pte` 结构好了。免得出现意外。

## `MmGetPhysicalAddress` 的 `IDA` 结果

代码:
