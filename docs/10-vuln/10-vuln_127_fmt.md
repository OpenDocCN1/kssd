
# Exploiting Windows Device Drivers

> 原文：[`www.pediy.com/kssd/pediy10/74535.html`](https://www.pediy.com/kssd/pediy10/74535.html)

arhat 的悄悄话没办法回![](img/e71d9210a215d5b66755d038631e0c83.jpg "恐怖!") 这个文章是由我翻译，并最早发于[SoftRCE](http://softrce.net)的。

引用: 本来是打算在昨天发布这个译文的，但是由于一些翻译上的问题和联系作者版权的问题，导致了译文发布的延迟。这次不一定是最完美的版本，但是大家先可以凑合下看吧。这篇文章是作为上手文章而作的，是配合着前段时间某个 paper 一起出来的姊妹篇，不过同期的姊妹篇不打算翻译了。文中提到了两篇 paper 也有打算汉化的想法，不过希望不要是无限期拖稿了...呵呵。高手是完全可以无视这篇文章的的。

文章中如果存在什么错误，欢迎在回复中指出。

注意：文章中的代码可能存在一定破坏性，是请使用者酌情处理，以上。代码可能由于合并转换导致存在问题，请自行修正。

下载地址：[本站下载](http://softrce.net/upload/ha_ewdd.pdf) | [RayFile](http://www.rayfile.com/files/aafdbe73-8eac-11dd-b228-0014221b798a/) | [Box.net](http://www.box.net/shared/m9jz588lvf)

本地备档一份，有些英文可能当时输入有误，但是并不影响阅读。![](img/c72cb8eba397ac67dda6e1ac58a9b627.jpg "微笑")

转载希望著名原作者版权，译文已联系原文作者，也请保留相关信息，感谢![](img/c72cb8eba397ac67dda6e1ac58a9b627.jpg "微笑")

上传的附件

| ![](img/d5c539bacaef0c9233cc56abc062f348.jpg) || ha_ewdd.pdf |
