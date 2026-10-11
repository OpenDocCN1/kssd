
# 深入剖析 Win32 可移植可执行文件格式

> 原文：[`www.pediy.com/kssd/pediy10/61499.html`](https://www.pediy.com/kssd/pediy10/61499.html)

Matt Pietrek 2002 年发表在 MSDN 上的关于 PE 的文章

上传的附件

| ![](img/d5c539bacaef0c9233cc56abc062f348.jpg) | | An In-Depth Look into the Win32 Portable Executable File Format-part 1.pdf |
| ![](img/d5c539bacaef0c9233cc56abc062f348.jpg) | | An In-Depth Look into the Win32 Portable Executable File Format-part 2.pdf |

引用：最初由 `cxd` 发布 ![查看帖子](img/78d57e1ac1f1fc075f84e78a6a789d46.jpg) 就是老了点，要翻译也应该翻译 `PeCoff_v8`。

你想到的都翻译了

现在流行的关于 PE 资料的最终来源不外乎以下几个：

1. Matt Pietrek 在 1994 年在 MSJ 上发表的关于 PE 的文章；
2. Matt Pietrek 在《Windows 95 System Programming Secrets》中关于 PE 的一章内容，实际上 Matt 说是 94 年那篇文章的完整版，因为 Under The Hood 专栏篇幅所限不能把所有内容都发表；
3. Microsoft 的 PECOFF 规范，MSDN Library Visual Studio 6.0 中有其 5.0 版，配合 Visual Studio 97，MSDN Library - October 2001 中有其 6.0 版，配合 Visual Studio 6.0，以及配合 Visual Studio 2005 的 PECOFF 规范 v8；
4. Matt Pietrek 在 2002 年发表在 MSDN Magazine 上两篇关于 PE 的文章，是为了配合 PECOFF 规范，帮助读者理解 PECOFF 规范。

因为《Windows 95 System Programming Secrets》台湾候捷先生已经翻译，其它的几篇文章我都翻译了，本意是想来个 PE 文章系列。其实这几篇文章并不完全重复，都有可取之处，关键是看自己了。
