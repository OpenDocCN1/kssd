
# (备忘)PE 文件格式的 RVA 概念

> 原文：[`www.pediy.com/kssd/pediy10/62348.html`](https://www.pediy.com/kssd/pediy10/62348.html)

## (备忘)PE 文件格式的 RVA 概念

我们常说的 `RVA`，很容易让我们理解成这是相对于节的 `RVA`，其实不然。

要理解 `RVA` 的概念，首先还必须理解 `Section Header` 结构（由 `Section Header` 构成节表）。

```c
typedef struct _IMAGE_SECTION_HEADER {
    BYTE    Name[IMAGE_SIZEOF_SHORT_NAME];
    union {
        DWORD   PhysicalAddress;
        DWORD   VirtualSize;
    } Misc;
    DWORD   VirtualAddress;
    DWORD   SizeOfRawData;
    DWORD   PointerToRawData;
    DWORD   PointerToRelocations;
    DWORD   PointerToLinenumbers;
    WORD    NumberOfRelocations;
    WORD    NumberOfLinenumbers;
    DWORD   Characteristics;
} IMAGE_SECTION_HEADER, *PIMAGE_SECTION_HEADER;
```

注意其中的 `VirtualAddress` 和 `PointerToRawData`，这两个都是节的起始地址。不同之处在于：

`VirtualAddress` 用在内存中指明该节的起始地址，而 `PointerToRawData` 用在文件中指明该节的起始地址。

由于文件的对齐值和内存的对齐值不一样，所以，如果以直接用 `RVA`（即 `iRVA`）去定位的话，那么，可能是错误的。

这个 `RVA` 我把它分成三种情况，在不同情况下计算得出的 `RVA` 各给个名字：

为了方便理解，使用例子数据来说明。

```c
typedef struct _IMAGE_SECTION_HEADER {
    BYTE    Name[IMAGE_SIZEOF_SHORT_NAME];  // .text
    union {
        DWORD   PhysicalAddress;
        DWORD   VirtualSize;    // 00069f9b
    } Misc;
    DWORD   VirtualAddress;     // 00000540
    DWORD   SizeOfRawData;      // 00069fc0
    DWORD   PointerToRawData;   // 00000540
    DWORD   PointerToRelocations;
    DWORD   PointerToLinenumbers;
    WORD    NumberOfRelocations;
    WORD    NumberOfLinenumbers;
    DWORD   Characteristics;    // 68000020
} IMAGE_SECTION_HEADER, *PIMAGE_SECTION_HEADER;
```

整理成和 `LordPE` 的形式如下：

名称 `Name` `VOffset` `VSize` `ROffset` `RSize` 标志

`.text` `0000540` `00069f9b` `00000540` `00069fc0` `68000020`

虚拟地址 `VA = 00049586`，`ImageBase = 00040000`

### 1、该 `RVA` 是以 `ImageBase` 为参考的

即是通过 `RVA = VA - ImageBase = 00049586 - 00040000 = 9586` 我们称其为 `iRVA`（我们常说为 `RVA`）

### 2、该 `RVA` 是以 `VOffset`（如 `.text` 节的起始地址）为参考的

即是通过 `RVA = VA - VOffset`，我们称其为 `mRVA`（内存节偏移）

### 3、真正的文件偏移为 `fRVA`

我们很清楚的知道，唯有它们以节的起始地址为参考的相对偏移才是固定的。我们就根据这个来计算。

我们以 `VA = 00049586` 为例子，为了得到它的文件偏移，我们需要经过以下的步骤：

先把 `VA` 转换成 `iRVA`，即 `iRVA = VA - ImageBase = 00049586 - 00040000 = 9586`

把 `iRVA` 转换成 `mRVA`，即 `mRVA = iRVA - VOffset = 9586 - 540 = 9046`

得到了 `mRVA` 后，就很容易得到 `fRVA` 了，`fRVA = mRVA + ROffset = 9046 + 540 = 9586`

所以，`VA = 00049586` 的文件偏移为 `fRVA = 9586`

上面由于 `ROffset` 和 `VOffset` 值一样，所以，你别以为直接计算就行了。

## 最后总结：

### 1、虚拟地址转文件偏移地址 `fRVA`

`fRVA = mRVA + ROffset`

由于 `mRVA = iRVA - VOffset`

再由于 `iRVA = 虚拟地址 - ImageBase`

所以，`mRVA = (虚拟地址 - ImageBase) - VOffset`

所以，`fRVA = (虚拟地址 - ImageBase) - VOffset + ROffset`

最终得到：

`fRVA = (虚拟地址 - ImageBase) - VOffset + ROffset`

一个简单的代入法！

### 2、`RVA` 转文件偏移地址 `fRVA`

同上
