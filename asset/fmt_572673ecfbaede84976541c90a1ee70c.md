
# 键盘过滤驱动与应用通讯代码

> 原文：[`www.pediy.com/kssd/pediy11/124565.html`](https://www.pediy.com/kssd/pediy11/124565.html)

主要功能完成了 不过还有个问题 就是加载完过滤驱动后第一个按键截不到，因为在 attach 到 `KBDCLASS` 之前,已经有一个 read irp 已经发下去了，我还没找到办法如何把之前发下去的这个 IRP 给弄掉，谁有好的建议 小弟在此多谢了。![](img/45dd044a2743704cbb333715eae4a5f6.jpg "狂笑")

对了 忘记说了 这个 DEMO 只适合于 `PS2` 键盘或者笔记本键盘，不支持 `USB` 的键盘:3

![](img/bd681f7c5fd5ed724d0078c5a40a4137.jpg)

上传的附件

| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || `KeyFilter.rar` |
