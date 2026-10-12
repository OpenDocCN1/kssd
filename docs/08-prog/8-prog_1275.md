
# 文件过滤驱动对特定文件实现只读的小例子

> 原文：[`www.pediy.com/kssd/pediy12/127943.html`](https://www.pediy.com/kssd/pediy12/127943.html)

最近学习了下文件过滤驱动，想做个小东西来巩固下，做了个笔记，于是有了本文。

原理很简单就是从`IRP_MJ_CREATE`中得到文件名然后与我们的文件名比较，然后得到`FILE_OBJECT`，再对`IRP_MJ_WRITE`中的`FILE_OBJECT`进行比较。

本来想做个应用层界面的，太麻烦了就没做了。利用看雪平台给自己保存下 呵呵

上传的附件

| ![](img/620251b33676a8e4b98ca0ece915ee8d.jpg) | `sfilter.zip` |
