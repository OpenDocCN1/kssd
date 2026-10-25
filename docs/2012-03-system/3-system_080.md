
# 也谈 PE 重定位表

> 原文：[`www.pediy.com/kssd/pediy12/140722.html`](https://www.pediy.com/kssd/pediy12/140722.html)

最近研究脱壳，遇到了 `dll`，所以不可避免的需要修复重定位表。以前研究过也脱过不少壳，但都是 `exe` 从来没有手工修过重定位表。

于是搜索之，恩有这么一篇：《PE 重定位表学习手记》，有刚巧手上有看雪段钢等编著的《加密与解密》，一并阅读。

《PE 重定位表学习手记》中说到：

> 每个块的首部是如下定义：
>
> ```
> typedef struct _IMAGE_BASE_RELOCATION {
>     DWORD VirtualAddress;
>     DWORD SizeOfBlock;
> } IMAGE_BASE_RELOCATION;
> ```
>
> ……
>
> 这个可由每个块结构中的 `Size` 来确定。`Size` 的值是以 `DWORD` 表示的当前整个块的大小，先减去 `_IMAGE_BASE_RELOCATION` 的大小，因为重定位数据是 16 位 `WORD` 的，再除以 2，就得到个重定位数据的个数。由 `Size` 可以直接到达下一个重定位块，如图所示：
>
> `0x5E850000 + 0x000000EC = 0x5E8500EC` 即为第二个重定位块的地址，直至某个块首结构的 `VirtualAddress` 为 0，表明重定位表结束。
>
> ……
>
> ```
> do {
>     //处理一个接一个的重定位块，最后一个重定位块以 RAV=0 结束
>     ...
> } while (pRelocBlock->VirtualAddress);
> ```

《加密与解密》中说到：

> 如果还有更多段，将重复上面数据结构，直到 `VirtualAddress` 为 `NULL`，表示结束。
>
> 如图 13.48 所示是重定位表结构的一个示意图。

OK，照此，就是说如果我们自己手工创建编辑了重定位表，那么需要填充 8 个字节的 0（也就是空的 `_IMAGE_BASE_RELOCATION`）作为重定位表的结束标记。并且 `pe` 文件头目录表中的重定位表尺寸也修改为整个重定位表数据的大小（当然包括这 8 个空字节）。

运行……嚯，`0xC0000005`……（如果 8 个空字节后恰好还有杂乱的随机数据，你还会看到各种有意思的 `Win32 ErrorCode`）。

为啥呢？是修改的时候手抖了吗？眼花？神经衰弱？非也，是这些文章错了！我们先来看看微软官方权威的《Microsoft PE and COFF Specification》

以下文字来自 Microsoft PE and COFF Specification（Updated: September 21, 2010 File name: `pecoff_v8.docx`）

> 5.6 The `.reloc` Section (Image Only)
> The base relocation table contains entries for all base relocations in the image. The Base Relocation Table field in the optional header data directories gives the number of bytes in the base relocation table. For more information, see section 3.4.3, “Optional Header Data Directories (Image Only).” The base relocation table is divided into blocks. Each block represents the base relocations for a 4K page. Each block must start on a 32-bit boundary.

可以看到，微软的文档中并没有说重定位表需要 `padding zero` 来作为 `end` 标志。

下面我们看看到底 `windows` 中是怎么处理这个重定位表的，下面是反汇编 `windows xp sp2` 的 `ntdll.dll`，实际的 `LoadLibraryA/W` 最后会调用到 `LdrLoadDll(x, x, x, x)`

然后沿下列调用链（用 `IDA` 和微软的符号服务器即可方便的分析）：

`LdrLoadDll(x, x, x, x) -> LdrpLoadDll(x,x,x,x,x,x) -> LdrpMapDll(x,x,x,x,x,x) -> LdrRelocateImage(x, x, x, x, x) -> LdrRelocateImageWithBias(x,x,x,x,x,x,x)`

`LdrRelocateImageWithBias` 就是实际执行重定位算法的函数，我们来看看这个函数是怎么做的：

![](img/1b1248a66654fb8df64df0329c708769.jpg)

首先 `RtlImageDirectoryEntryToData` 从目录表中获取重定位表的偏移和尺寸（`TotalCountBytes`）

然后指针移动到重定位表开始，然后循环开始每一项 `_IMAGE_BASE_RELOCATION` 调用一次 `LdrProcessRelocationBlockLongLong`，

`LdrProcessRelocationBlockLongLong` 返回值是下一项的指针，除非访问到了无效内存，否则这个指针我们可以认为永远是有效的，

`7C93D8A7` 这里的跳转不会执行到。`TotalCountBytes` 为零的时候，说明重定位表处理完成，成功返回。

`LdrProcessRelocationBlockLongLong` 是一个非常单纯的函数，里面完全根据读取到的数据来对 `PE` 进行重定位，

也就是说，它永远认为输入数据是正确的！没有任何错误处理！

好了，现在我们回到一开始的问题：`windows` 遇到了重定位表的最后一项，我们手工写的 8 个零，

于是 `LdrProcessRelocationBlockLongLong` 一丝不苟的开始在 `VirtualAddress = 0` 的地方，

开始处理 `(SizeOfBlock - 8) / 2` 项，别忘了 `SizeOfBlock = 0`，`(SizeOfBlock - 8) / 2` 是什么请读者自己算吧……

总结：重定位表实际有多少项就是多少项！
