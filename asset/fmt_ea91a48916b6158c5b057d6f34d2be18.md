
# 软件安全测试（fuzz）之大家一起学 2：认识 PNG 文件格式

> 原文：[`www.pediy.com/kssd/pediy10/75181.html`](https://www.pediy.com/kssd/pediy10/75181.html)

首先想和大家一起讨论的，是 `filefuzz`。在我们开始继续深入 Peach 的数据定义之前，我决定先独立于 peach，用一点篇幅介绍下文件格式。

学习 `filefuzz` 就要非常清楚要测试的文件格式。我们以 PNG (Portable Network Graphics)为例，在附件中我给出了 PNG 图片文件格式的详细参考文档。

附件：PNG 文件格式 `PNG_format.zip`

除了附件中的参考之外，在 `http://www.libpng.org/pub/png/` 能找到更多关于 PNG 的资源。

简单的总结一下文档中关于 PNG 文件格式的描述。

![](img/98d59256ebcb586bb6408404d47a722c.jpg)

如上图：PNG 文件可以简单的理解成一个文件头和一系列数据块，具体说来：

文件头：由八个字节组成，`0x89504e470d0a1a0a`

数据块：每个数据块由四部分构成，他们的描述依次如下：

`Length`：占四字节，表示数据块 `data` 域占多少个字节。（注意这里不包括 `length` 自身）

`Type`：占四字节，表示当前块的类型。一般是英文大小写字母的 ASCII 码（65～90 或者 97～122）

`Data`：数据区。大小可以是 0 字节

`CRC`：占四个字节，整个 `chunk` 的 CRC 校验码（`Length+Type+Data`）

下面我们打开一个 PNG 文件一起对照着文件格式在二进制编辑器中更加直观的来学习一下。这里我介绍一个解析文件结构很牛 X 的二进制编辑器 `010editor` 给大家。

可以去 `http://www.sweetscape.com/010editor/` 下载到。

上面这个 PNG 文件格式的示意图就是一个 PNG 格式的图片，这里就以它为例了，将其另存在本地文件 `PNG_chunk.png`。

安装好 `010editor`，用 `010editor` 打开 `PNG_chunk.png`

![](img/b9b5345a77eaf5d70cb1a9cb2623d855.jpg)

如上图，界面看起来和 Ultraedit 也差不了多少，大家应该比较熟悉。

我们主要想用 `010editor` 的二进制文件模板，套用这些文件格式解析脚本，可以更容易更直观的搞清楚文件格式。

注意上图红色标出的几个模板按钮，打开下拉框，默认安装 `010editor` 只能识别出 bmp,zip,wav 这三种文件。为了能够识别 PNG 文件，我们需要去 `http://www.sweetscape.com/010editor/templates/` 下载额外的文件模板。

下载需要的模板，我们这次至少需要 `PNG12Template.bt` 模板。

按 `ctr+F5` 或者点第二个模板按钮，打开刚下载的 `PNG12Template.bt` 模板。

按 `F5` 或者点击第四个模板按钮，套用 PNG 文件模板。

![](img/55ad19b1ac82506a64ae4e13ae8dda59.jpg)

如上图，运行成功后，可以看到 `010editor` 解析出了若干个 `chunk`，并且能够用不同颜色标注出你所选中的某一个 `chunk` 的某一个字段。

这些工具和知识会为后面调试 peach 的数据定义带来很多帮助。

下一讲我们就一起来用 peach 定义这样一个 PNG 文件。

最后，小声说一下，《0day 安全：软件漏洞分析技术》第二次印刷刚结束。第二次印刷总共勘误了 50 余处，包括了大家反映的插图不清，一个重要示意图的重复，若干出地址和指针的标注，以及错别字等等。感谢 0day 版朋友的勘误，书籍质量的提高离不开你们的支持（最后一句话是大声说的）。
