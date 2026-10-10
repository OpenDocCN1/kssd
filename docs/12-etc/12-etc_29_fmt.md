
# 介绍 Symbian E32Image 文件处理工具：elftran

> 原文：[`www.pediy.com/kssd/pediy10/93525.html`](https://www.pediy.com/kssd/pediy10/93525.html)

如果我们要深入分析 Symbian 平台的可执行文件就需要掌握其文件格式-`E32Image`。
前面的文章中翻译了 `E32Image` 格式的基本资料，可以给学习、研究者一些简单的参考。
在 `WIN32/X86` 平台针对 `PE` 文件格式有很多很好用的分析、修改工具比如 `PE Tools`，`PEdit` 等等，这些工具大大降低了 `PE` 文件分析的难度。
实际上在 Symbian 平台同样有命令行工具可以用于 `E32Image` 文件的分析和修改。本文介绍的就是这样的一款工具：`elftran.exe`

`elftran.exe` 是 Symbian 公司的产品，它的基本功能是将 `ELF` 格式的文件转换成 `E32Image` 格式的文件。在前面的翻译资料中有介绍：Symbian 平台的可执行文件编译连接之后，要使用 `elftran.exe` 将其转换为 `E32Image` 格式。

```
使用方法：

elftran [options] inputfile outputfile

elftran [options] e32imagefile
```

从使用方法中可以看到该工具还可以直接对 `E32Image` 格式的文件进行操作。
参数如下：
`option: [-v] [[-no]call[entrypoint]] [-priority <priority>] [-stack <size>] [-heap <min> <max>] [-uid<n> <uid>] [-allowdlldata] [-datalinkaddress <base>] [-fixed] [-moving] [-align-const-section] [-const-section-address-mask <mask>] [-[no]compress] [-compressionmethod none|deflate|bytepair] [-capability "<list>"] [-version M.m] [-vid <id>] [-fpu <softvfp|vfpv2>] [-[default|un]paged] [-dump [h][s][c][d][e][i]]`

可以看到 `elftran.exe` 提供了非常丰富的设置功能。可以设置程序的运行权限、堆栈大小、堆的最大最小值、`uid` 等等。可以压缩和解压 `E32Image` 文件。可以将文件信息 `dump` 到屏幕上等等。

```
一些简单示例：

01、解压 E32Image 文件
    elftran -nocompress dvdplayer.exe

02、压缩文件
    elftran -compress dvdplayer.exe

03、设置堆栈大小
    elftran -stack 1024 dvdplayer.exe

04、Dump E32Image 文件头
    elftran -dump h dvdplayer.exe
```

`-dump` 对应参数含义。
`h` Header
`s` Security info
`c` Code section
`d` Data section
`e` Export info
`i` Import table

这几个参数可以同时使用。
通过实验还证明，可以使用该命令可以修改 `uid` 的数值，相应的 `CRC` 会由程序自动调整。

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || elftran 使用.rar |
