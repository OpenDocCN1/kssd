
# 请教 IDA 中 IDS 的用途，TIL 是否有制作工具

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-307.htm`](https://www.pediy.com/kssd/pediy09/pediy09-307.htm)

> `IDA` 目录下有一堆 `IDS` 文件，载入之后看不出有什么区别，也找不到说明文件，请问各位前辈 `IDS` 文件的用途是什么呢？

> 另外一个问题就是 `IDA` 中的类型库（`*.til` 文件），是不是没有官方工具制作？

> 简单的 `google` 过，也上 `IDA` 的主页看过，好象没有这方面的相关信息，倒是在官方下载主页上看到一个制作 `IDS` 的工具，不过我连 `IDS` 是干什么的都不知道，好晕![](img/d40b5575a143b1b5e17e7f31e3f35d26.jpg "恐怖!") ![](img/d40b5575a143b1b5e17e7f31e3f35d26.jpg "恐怖!") ![](img/d40b5575a143b1b5e17e7f31e3f35d26.jpg "恐怖!")

> `The IDS files are used automatically by IDA to assign meaningful names,`
> `comments, and additional information to the imported DLL functions.`

> `DLL2IDT creates an IDT file from a DLL file`
> `The IDT file is a text file. Its format is described below.`

> `AR2IDT creates an IDT file from an AR library file`

> 就是说有写 `xxx.dll` 导出的是
> `Install @1`
> `Uninstall @2`
> 结果 `ida` 反汇编的时候只能反汇编出 `xxx_1`，没有函数名对应，你可以理解成类似的 `OD` 识别 `MFC`，
> 不能识别出来有写函数，需要借助 `MFC` 的 `lib` 文件一样

> 正好 `openrce` 上有三篇讨论文章
> `http://www.openrce.org/blog/view/769`
> `http://www.openrce.org/blog/view/770`
> 还有一篇文章，是应用，我忘了
> 本论坛还有 `codez` 发贴找这个东西的 `5.0` 版

> `til` 不知道有没有工具
> 在 `5.1` 的新改变中有一条就是
> `VISTA We have added new types from the latest MS Windows SDK. This includes, among other`
> `additions, Vista types.`
> 然而我反汇编 `WDF` 的过滤驱动时，没有找到新添加的结构如 `FLT_CALLBACK_DATA`，`FLT_IO_PARAMETER_BLOCK` 等，
> 这些结构在头函数 `fltKernel.h`，不能添加上着说明 `WDF` 类型库没有搞上，这也是我很郁闷的地方。
> 唯一的方法就是把结构单独提出来放在一个 `.h` 中，然后识别，当然可能过不去，只能一点一点的处理。
> 当然你可能看见 `http://www.datarescue.com/idabase/idadown.htm` 上有个小工具 `h2enum`，功能为：
> `an IDC script by Leonid Lisovsky converts C/C++ header files to IDA enums. Nice if you have no TIL files.`
> 不懂其意思。也没有用过。也许自己写个工具或者插件的可以处理。还有就是有篇关于 `til` 原理的文章
> `A Simple Type System for Program Reengineering`。可惜是 `IEEE` 的，要看要收费的。谁要是在学校，
> 可以免费看的话。希望能下一下，传上来.
> 这也是工具作者发表的两篇文章之一，见 `http://www.hexblog.com/about.html`
