
# [翻译]通过x64分页机制的PTE Space实现内核漏洞利用

# 通过x64分页机制的PTE Space实现内核漏洞利用

在研究NVIDIA DxgDdiEscape Handler的漏洞时，可以非常明显的感觉到过去几年中讨论的GDI原语的方法对于可靠的利用此漏洞毫无帮助。

因此，我想出了另一个解决方案：可以选择映射一些的特殊的虚拟地址，强制对齐这些地址的页目录，然后使用漏洞来改写这个页目录。

简单来讲，这种操作允许在我们选择的/一直的虚拟地址上`**映射任意的物理地址**`。

在接下来的部分中，我将介绍x64分页表的详细信息以及利用此漏洞的特殊技巧。

## x64中的分页机制

x64使用了4级页表来映射物理内存与虚拟内存。这4级分别是PML4（Page Map Level 4）（俗名：PXE），PDPT（Page Directory Pointers），PD（Page Directory）以及PT（Page Table）。`CR3`（控制寄存器）保存着当前进程的PML4基地址（物理地址）。

下图是x64下从虚拟内存到物理内存寻址的大致流程：

![图片描述](upload/attach/201901/3_3ZBX2AH9FE536MS.png)

举个栗子：

如果我们要遍历虚拟地址为`0x71000000000`（仅仅是个例子。。）：

首先，分解这个虚拟地址：

![图片描述](upload/attach/201901/3_UU8R5J8Q6WEC3N4.png)

虚拟地址的低12位表明了页内偏移。接下来（译注：从低位到高位的顺序）的9位是PT（Page Table）索引，接着是PD（Page Directory）索引，再接下来的9位是PDPT（Page Directory Pointer Table）索引，再往下的9位是PML索引（Page Mapping Level 4）。

我使用了下边这个结构体来表示这些数据：

![图片描述](upload/attach/201901/3_RYSGEUCMJKQ38H7.png)

比如说：

![图片描述](upload/attach/201901/3_EFQJPWV3X9CGZZU.png)

对于我们之前的例子（虚拟地址为0x71000000000）来讲，我们通过上边的结构体获得了`pml4_index=0x0E`，`pdpt_index=x040`，`pd_idnex=0`，`pt_indedx=0,offset=0`。

![图片描述](upload/attach/201901/3_KNM7NNVJHN7MC8T.png)

现在，我们知道了对于`0x71000000000`这个虚拟地址，PML4 目录是`0x38a0000040653867`

这实际上是一个被称为MMPTE的8字节结构，我们需要从中得到PFN（Page Frame Number）。

![图片描述](upload/attach/201901/3_78YBSD24GS8ZMT.png)

现在，得到了PFN是`0x40653`。将PFN*PageSize就得到了下一级结构PDPT的基址，然后使用`pdpt_index`来索引PDPT.

![图片描述](upload/attach/201901/3_HUZY8DVSYXJ7EAV.png)

![图片描述](upload/attach/201901/3_BWED4AHWDJ8F59G.png)

继续将PFN(`0x41cd7`)*PageSize得到了下一级结构PD的基址是`0x41cd7000`。使用`pd_index`来索引这张表。

![图片描述](upload/attach/201901/3_NJE6V9VW6S7EPWR.png)

![图片描述](upload/attach/201901/3_P2UU76ZYYRHBE69.png)

和之前同样的操作使用PFN（`0x3e7d8`）* PageSize得到了PT的基址是`0x3e7d8000`。使用`pt_index`来索引这张表。

![图片描述](upload/attach/201901/3_PANCB5HVMKEMXY3.png)

![图片描述](upload/attach/201901/3_9EZCCUHH9EBUDNG.png)

最终，剩下的就是将这个PFN（`0x3d7d9`）* PageSize + page_offset。

现在我们知道了虚拟地址`0x71000000000`对应的物理地址是`0x3d7d9000`。

如果你想更加深入的了解这些机制，可以参考一下资料：

<<mark class="encrypted">bebK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8%4u0W2M7$3g2U0N6i4u0A6N6s2W2Q4x3X3g2U0L8$3#2Q4x3V1k6T1L8r3!0Y4i4K6u0r3k6$3g2@1N6r3W2F1k6#2)9J5k6s2m8Z5P5i4y4A6j5$3q4D9i4K6u0V1k6i4S2@1M7X3g2E0k6g2)9J5k6r3q4T1N6i4y4W2i4K6u0V1L8$3k6Q4x3X3c8A6L8Y4c8W2L8q4)9J5k6r3u0S2M7$3g2V1i4K6u0V1M7r3q4Y4K9h3&6Y4i4K6u0V1M7%4W2K6N6r3g2E0M7#2)9J5k6s2m8S2M7Y4c8Q4x3X3b7I4</mark>

<<mark class="encrypted">2edK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8%4u0W2M7$3g2U0N6i4u0A6N6s2W2Q4x3X3g2U0L8$3#2Q4x3V1k6T1L8r3!0Y4i4K6u0r3k6$3g2@1N6r3W2F1k6#2)9J5k6s2m8Z5P5i4y4A6j5$3q4D9i4K6u0V1k6i4S2@1M7X3g2E0k6g2)9J5k6r3q4T1N6i4y4W2i4K6u0V1L8$3k6Q4x3X3c8A6L8Y4c8W2L8q4)9J5k6r3u0S2M7$3g2V1i4K6u0V1M7r3q4Y4K9h3&6Y4i4K6u0V1M7%4W2K6N6r3g2E0M7#2)9J5k6s2m8S2M7Y4c8Q4x3X3b7J5i4K6u0V1N6$3W2F1k6r3!0%4M7H3`.`.</mark>

<<mark class="encrypted">c7fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8%4u0W2M7$3g2U0N6i4u0A6N6s2W2Q4x3X3g2U0L8$3#2Q4x3V1k6T1L8r3!0Y4i4K6u0r3k6$3g2@1N6r3W2F1k6#2)9J5k6s2m8Z5P5i4y4A6j5$3q4D9i4K6u0V1k6i4S2@1M7X3g2E0k6g2)9J5k6r3q4T1N6i4y4W2i4K6u0V1L8$3k6Q4x3X3c8A6L8Y4c8W2L8q4)9J5k6r3u0S2M7$3g2V1i4K6u0V1M7r3q4Y4K9h3&6Y4i4K6u0V1M7%4W2K6N6r3g2E0M7#2)9J5k6s2m8S2M7Y4c8Q4x3X3b7K6i4K6u0V1N6$3W2F1k6r3!0%4M7#2)9J5k6r3S2S2L8s2y4Q4x3X3c8Z5k6h3q4H3</mark>

### NVIDIA 的漏洞：

<a href="elink@e3fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1N6h3N6K6i4K6u0W2j5$3S2J5L8$3#2A6N6h3#2Q4x3X3g2G2M7X3N6Q4x3V1k6H3i4K6u0r3M7s2u0G2K9X3g2U0N6q4)9J5k6s2A6W2M7X3!0Q4x3V1k6A6M7%4y4#2k6i4y4Q4x3V1k6D9K9i4y4@1i4K6y4r3j5$3q4F1i4K6y4p5x3g2)9J5y4Y4q4Q4x3@1c8z5g2V1W2p5d9f1q4Q4x3V1u0W2M7$3y4S2M7r3g2Q4x3U0k6U0L8$3I4K6M7r3g2U0i4K6y4p5d9f1c8Q4x3V1u0f1P5i4m8W2i4K6u0n7f1%4c8S2N6s2g2K6i4K6u0n7f1s2u0A6L8%4u0A6N6s2W2Q4x3V1u0y4K9h3I4W2M7%4c8G2L8X3g2Q4x3V1u0a6N6$3&6W2M7W2)9J5b7W2y4#2L8h3#2S2M7Y4W2Q4x3U0k6U0k6h3I4D9M7#2)9K6c8r3W2V1M7H3`.`.">最早被Google Project Zero报告</a>，在PDXGKDDI_ESCAPE各种回调中，有一个分支出现了漏洞。这是一个用户模式display驱动程序与display 微端口驱动之间共享数据的接口。

经过一番研究，我决定把重点放在‘NVIDIA: Unchecked write to user provided pointer in escape 0x600000D’,这条信息上。这个漏洞可以让我们在任意的虚拟地址上写入数据，但是不能控制正在写入的数据或数据的大小。事实上，写入的大部分数据是0，内部代码强制执行了大小检查只允许我们至少写入0x1000（4096）字节。

原始POC<<mark class="encrypted">a05K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1N6h3N6K6i4K6u0W2j5$3S2J5L8$3#2A6N6h3#2Q4x3X3g2G2M7X3N6Q4x3V1k6H3i4K6u0r3M7s2u0G2K9X3g2U0N6q4)9J5k6s2A6W2M7X3!0Q4x3V1k6A6M7%4y4#2k6i4y4Q4x3V1k6V1k6i4c8S2K9h3I4Q4x3@1k6A6k6q4)9K6c8o6V1I4x3g2)9J5y4X3y4S2L8W2)9K6c8o6q4Q4x3U0k6I4i4K6y4p5e0W2k6u0c8p5W2m8i4K6t1#2x3U0m8W2M7$3y4S2M7r3f1`.</mark>

### Exploiting：

由于缺乏对正在写入的数据或者写入数据大小的控制，我不得不放弃最近使用的GDI原语，认识到需要搞点与众不同的事了。

之前讨论的“x64中的分页机制”中的页表有时也存在于被称为"<a href="elink@edeK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2J5k6h3q4U0N6r3!0K6i4K6u0W2L8%4u0Y4i4K6u0r3N6$3W2C8K9g2)9J5c8W2c8W2j5$3S2%4K9h3E0A6i4K6y4m8e0h3g2E0L8%4u0&6i4K6g2X3e0r3q4&6L8%4g2@1i4K6t1K6b7f1#2p5y4U0b7`.">PTE空间</a>"的内存区域，通过对该区域的滥用，我得到了一个解决方案。

PTE空间是Windows内核在需要管理分页结构时使用的虚拟内存区域。（涉及页访问权限，将内容移动到pagefile中，协同内存映射等等。。）

通过一些偏移和掩码，我们可以计算出任何给定虚拟地址（在PTE空间上）的每个表的虚拟地址。

以下代码来自于：<<mark class="encrypted">187K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6v1k6i4u0W2L8i4W2r3k6i4c8A6N6X3g2S2N6g2)9J5c8V1g2^5M7r3I4G2K9i4c8K6i4K6u0r3j5X3I4G2j5W2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8V1u0&6M7r3q4K6M7#2)9#2k6W2y4y4c8g2m8Q4y4h3k6p5c8g2m8Q4x3V1k6E0K9i4c8A6k6$3q4@1K9h3!0F1i4K6t1#2x3U0m8T1P5i4m8S2M7%4y4Q4x3V1k6o6L8$3#2H3N6i4c8S2N6r3W2G2L8Y4y4Q4x3X3g2U0M7s2l9`.</mark>>)

```c
#define PXE_PAGES_START 0xFFFFF6FB7DBED000 // PML4
#define PDPT_PAGES_START 0xFFFFF6FB7DA00000
#define PDE_PAGES_START 0xFFFFF6FB40000000
#define PTE_PAGES_START 0xFFFFF68000000000
ULONG64 GetPML4VirtualAddress(ULONG64 vaddr) {
vaddr >>= 36;
vaddr >>= 3;
vaddr <<= 3;
vaddr &= 0xfffff6fb7dbedfff;
vaddr |= PXE_PAGES_START;
return vaddr;
}
ULONG64 GetPDPTVirtualAddress(ULONG64 vaddr) {
vaddr >>= 27;
vaddr >>= 3;
vaddr <<= 3;
vaddr &= 0xfffff6fb7dbfffff;
vaddr |= PDPT_PAGES_START;
return vaddr;
}
ULONG64 GetPDEVirtualAddress(ULONG64 vaddr) {
vaddr >>= 18;
vaddr >>= 3;
vaddr <<= 3;
vaddr &= 0xfffff6fb7fffffff;
vaddr |= PDE_PAGES_START;
return vaddr;
}
ULONG64 GetPTEVirtualAddress(ULONG64 vaddr) {
vaddr >>= 9;
vaddr >>= 3;
vaddr <<= 3;
vaddr &= 0xfffff6ffffffffff;
vaddr |= PTE_PAGES_START;
return vaddr;
}
```

再来一次，对于我们的例子`0x71000000000`这个虚拟地址来说：

`GetPML4VirtualAddress(0x71000000000)` 将返回 `0xFFFFF6FB7DBED070`

![图片描述](upload/attach/201901/3_AMY726SVQ54WD8K.png)

可以通过WINDBG或者KD来确认：

![图片描述](upload/attach/201901/3_DP8REEK8AHAE9FR.png)

### 重映射原语（概览）

此刻，我们已经具备了攻击计划的全部要素：

- 漏洞允许我们写一堆0到任意虚拟地址

- PTE空间中的页目录可以被漏洞修改

- 篡改页表的PFN可以让我们映射任何物理地址

可以使用`VirtualAlloc`获得一个被映射的虚拟地址（例如：`0x71000000000`，姑且称这个被映射的虚拟地址位ADDR1a）,同时破坏它在PTE空间中的PD目录（例如：FFFFF6FB41C40000）使其指向物理地址0。

看起来就像这样：

![图片描述](upload/attach/201901/3_Y24DHVJE35QYNR2.png)

![图片描述](upload/attach/201901/3_NES9USYFPFAKPSY.png)

此时，我们的原语就已经准备好了。

我们可以通过对`0x71080000000`处的_MMPTE写入一些数据使这个地址有效。然后从`0x71000000000`读取和写入。

完成这项工作的宏定义：

![图片描述](upload/attach/201901/3_5QTXKGTHUDPKAAE.png)

### 物理0地址：

我们使用物理0地址的原因是漏洞只允许我们写0。但是它到底是怎么起作用的呢？？？如果物理地址无效，我们的操作将立即导致BSOD。

### 深入重映射机制：

![图片描述](upload/attach/201901/3_4Z7EQFXMH6EZ6PE.png)

### 一些问题：

1)如何保留_MMPTE的标志位。我们需要至少保留12标志位中的3位以表明我们从RING3中映射的地址是可读可写的。

![图片描述](upload/attach/201901/3_TT46UWUAP8CVN4N.png)

可以通过写数据的时候错位1字节的偏移来解决这个问题（ 写入`FFFFF6FB41C40001`来替代写入`FFFFF6FB41C40000`）。

2)事实上，漏洞需要我们至少写入0x1000字节，这意味着我们需要在`FFFFF6FB41C40001 `来写入`ADDR1a`的PD表。同时，要确保`FFFFF6FB41C41001 `是一个可以写入的地址。同样，可以通过`VirtualAlloc`

来解决这个问题，但是，这次映射的地址变为了`0x71040000000 `（称之为`ADDR1b`）。

分解`0x71040000000`这个地址之后，它看起来像这样（注意：仅仅将`ADDR1a`的`pfpt_index=0x40`改为 `0x41）：

![图片描述](upload/attach/201901/3_WGX3ZAYBMCBKDZX.png)

在函数` GetPDEVirtualAddress(0x71040000000) `执行后，我们得到了`FFFFF6FB41C41000`。所以我们现在解决了第二个问题。

![图片描述](upload/attach/201901/3_NGSH9PUXFK9U42E.png)

![图片描述](upload/attach/201901/3_BGA9ZMRK6V2ZJW8.png)

3)这个问题稍微有点复杂，依赖于硬件或者系统。由于性能原因，分页结构被缓存在TLB（Translation Lookaside Buffer，转换检测缓冲区）中。

在使用完映射原语之后，我们需要某种方式来使TLB无效或者刷新TLB，否则对页表所做的操作将不会立即生效（因为旧的值被缓存了）。
