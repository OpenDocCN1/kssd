
# 分享曾经写的更换 EXE 显示图标源码 （本人菜鸟大牛飘过）

> 原文：[`www.pediy.com/kssd/pediy12/130321.html`](https://www.pediy.com/kssd/pediy12/130321.html)

来看雪很长时间了，但从未贡献过什么`![](img/906f120cfe711ccbc8463fd6eb86eee9.jpg "尴尬")`。现分享自己参考很多大牛的文章后写的一个更改 `EXE` 文件显示图标的代码，本人菜鸟，大牛飘过啊`![](img/755d44573ea0045d4eb19f2164c82d4e.jpg "扮鬼脸")`。有什么不对的地方，大家提出来一起进步啊。

代码很简单：先调用 `GetLagerIconFromIco` 或 `GetLagerIconFromExe` 载入图标资源，然后调用 `UpdataExeIcon`，目标 `EXE` 文件的显示图标就修改成功了，哈哈。

上传的附件

| 附件图片 | 文件名 |
| :--- | :--- |
| `![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg)` | `Icon.rar` |
