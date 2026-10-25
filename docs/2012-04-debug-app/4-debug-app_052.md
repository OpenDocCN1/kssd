
# 基于最小 XLS 文件格式的分析

> 原文：[`www.pediy.com/kssd/pediy12/130658.html`](https://www.pediy.com/kssd/pediy12/130658.html)

标题:基于最小 XLS 文件格式的分析

发信人:落幕 CC

时间:2011-03-11

联系方式:247228162@qq.com

## 详细信息

第一次发表文章，不对的地方大家多多指正，基于前辈的研究+自己的一点努力，算是弄懂了复合文档的文件格式本篇文章基于最小 XLS 文件格式的分析，不对之处大家多多指教，废话不多说

## XLS 文件格式

`XLS` 文件作为一个文件流（`FileStream`），是由多个子流（`SubStream`）构成的，下面这张图是将各个 `SubStream` 按顺序依次展现出来，这些 `SubStream` 都是有若干个整数倍 Block（1 `block` ＝ 512 bytes）组成，所以 `XLS` 文件最小也为 13,824bytes（即一个表格时）（其实不管是 word 还是 ppt 其实都是基于复合文档格式，`WPS` 其实也一样），自己随便建立一个 `XLS` 文件，用 `UltraEdit` 打开就可以看见，我这里是基于最小的，也就是一个 sheet 了！

主要组成如下：

![](img/9d899a60ff962666982d5b067061b0a7.jpg)

其中 `WorkBook` 为数据变化区，它会随着文件的内容的增大而增大，但始终保持 Block 的整数倍。

## `XLS` Header

最小 `XLS` 文件 Header

![](img/c64df6f76b715a581d02b3a42df0f8aa.jpg)

表 1

\* `00000000h` 到 `00000010h` 这 32 个字节基本不变，表示是复合文档，前 8 个字节 `D0 CF 11 E0 A1 B1 1A E1` 表示复合文档标识，后面 16 个字节全部为 0,后面 2 个字节 `3E 00` 代表文件格式修订号，后面 2 个字节代表文件格式版本号，后面 2 个字节 `FE FF` 代表字节顺序号（高存高低存低）后面连个字节 `09 00` 代表标准扇区的大小（公式= 2 的 9 次幂（`09 00` 十进制=9） = 512 bytes

\* `00000020h` 行的 `01 00 00 00` 表示：BigBlock pointer 区占的 block 个数。表 1 中 BigBlock pointer 区 Block 个数为：01（十进制）。

\* `00000030h` 行的 `19 00 00 00` 表示：指向 `RootEntry` 所在 Block 序号。表 1 中 `RootEntry` 的所在的起始 block 序号为：25（十进制）。

\* `00000040h` 行的 `FE FF FF FF` 表示：指向 `XLS` header 扩展区所在 Block 序号。`FE FF FF FF` 表示没有 `XLS` header 扩展区

\* `00000040h` 行的 `00 00 00 00` 表示：`XLS` header 扩展区所占的 block 的个数。如果没有 `XLS` header 扩展区这里就是 `00 00 00 00`。

\* `00000040h` 行的 `18 00 00 00` 表示：指向 BigBlock pointer 所在 Block 序号。表 1 中 BigBlock pointer 的所在的起始 block 序号为：24（十进制）。

\* `00000050h` 行开始到 `00 00 01 f0h`：4 字节组。因为没有 `XLS` header 扩展区，所以连续填充 `FF FF FF FF`

## `WorkBook`

存放 execl 表数据信息所有 sheet 数据都在这里面 ,里面的东西重要（那要看对那些对象，其实是对我不重要，呵呵）

`09 08 10 00 00 06 05 00 86 20 CD 07 C9 C0 00`

第 1 个字, `0×0010`, 按照 BIFF 记录结构，当然是长度了

第 2 个字，`0×0600`, BIFF 版本，BIFF7 总是 `0×0500`, BIFF8/BIFF8X 总是 `0×0600`,

第 3 个字, `0×0005`, 数据类型, `0×0010` 表示 Sheet, `0×0005` 表示 Workbook Global 第 4 个字，Build Id 不能是 0, 但可以被忽略第 5 个字，Build Year 不能是 0

## Summary Information

属性，路作者，工作表类型等，

Excel 中点击文件>属性的部分内容存在这里（不重要）

## Document Summary Information

文档的属性（不重要）

## BigBlock pointer

\* 从 `01 00 00 00` 开始，连续，注意最后一个 4 字节组，也是连续的，但到一个 substream 结束时 ,`FE FF FF FF` 为结尾标识

\* `WorkBook` 和 Summary Information 和 Document Summary Information 中共有几个 block，BigBlock pointer 就有几个 4 字节组，BigBlock pointer 所占的字节数一定为 512 的整数倍。

\* 每当一 SubStream 结束时就加结尾标识 `FE FF FF FF`（数值：-2），不是以 block 为一个单位，所以在 BigBlock pointer 里至少要有 4 个 `FE FF FF FF`（数值：-2），即 4 个 substream 结束标识。

\* 在最后一个 `FE FF FF FF` 后，跟着是 `FC FF FF FF` 或 `FD FF FF FF` 或两者同时都有。当有 `XLS` Header 扩展区时，是两者都有，当无 `XLS` Header 扩展区时，只个 `FD FF FF FF`。`FD FF FF FF` 的个数为 BigBlock pointer 所占的 block 个数。未用的用 `FF FF FF FF`（数值：-1）填满。

## RootEntry

1. 存放文件中 `RootEntry` 属性、地址和大小
2. `Workbook` 属性，地址和大小，占有实际数据的内容
3. `SummaryInformation` 属性，地址和大小
4. `DocumentSummaryInformation` 的属性，地址和大小

\* 每个 128 字节以 name（占 64 字节）的 Unicode Strings 开始。如“`52 00 6F 00 …………6E 00 74 00 72 00 79 00`”是“Root Entry”的 16 进制形式。不足 64 字节的用 00 补足 64 字节。

\* 接下 2 字节表示 name 中 64 字节被使用字节个数，可以反映出 name 中字符个数（包括 null 结束符）。如“`16 00`”表示 name 被使用字节数为 22 个，反映出 name“Root Entry”有 9 个字符（包括 null 结束符）。

\* 再接下来 2 字节为 type。Root Entry：`05 01`，其余三个为：`02 01`；之后 8 个字节是数值-1、数值-1：`FF FF FF FF FF FF FF FF`。但 Summary Information substream 为 `01 00 00 00 03 00 00 00`

\* `00 00 00 00 00 10 00 00` 表示：前四个字节表示指向 WorkBook 所在 Block 序号（0），后四个字节表示 WorkBook 的大小（4096byte）。下面的都一样

\* 每个 128 字节内容，其偏移量 116 后的 4 个字节为每个区块的开始位置，之后的 4 个字节为每个区块的大小，地址和大小数据会随着内容的变化而变化。

分析就到这里吧，第一次发，大虾们多多指教！
