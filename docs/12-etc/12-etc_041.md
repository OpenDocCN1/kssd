
# Android 联系人数据库分析及获取联系人方法

> 原文：[`www.pediy.com/kssd/pediy12/136435.html`](https://www.pediy.com/kssd/pediy12/136435.html)

第一次发贴，方家见笑。![](img/afde35a5f3b3a4675abb3221cf07c6d1.jpg "124.gif")

查找获取联系人方法时，找到以下代码

`Contacts_.rar`，

可以获取系统中的联系人（2.2 系统），但程序处理得不太好，如下图：

![](img/0fa30db9617c5f4387b3c9cee0a23d45.jpg)

同时代码没有注释，有些地方看不懂，于是自己分析了一下 Android 系统联系人数据库，写了一份总结，比较长，有 28 页，就不贴出来了。

总结如下：![](img/081868d20f20fe7a9b4b88ab0d0d33c6.jpg "狂笑")

Android 联系人总结.pdf

这是我对系统联系人数据库的分析结果，希望对大家有所帮助，如果有错误之处，烦请指出。![](img/081868d20f20fe7a9b4b88ab0d0d33c6.jpg "狂笑")

我将以上代码进行了修改，获取联系人后显示效果如下图：

![](img/3dbab25f1dd3587ef07be30f10d19db6.jpg)

以下为修改过的代码：

上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || `GetContacts_zy.rar` |
