
# 一行代码检测程序是否使用 OllyDBG 启动

> 原文：[`www.pediy.com/kssd/pediy11/119484.html`](https://www.pediy.com/kssd/pediy11/119484.html)

测试方法：

1. 使用`OllyDBG`打开附件中的程序，点击`test`按钮，弹出`MessageBox`显示`'程序使用 OllyDBG 启动!!!'`
2. 直接打开程序，点击`test`按钮，则显示程序未使用`OllyDBG`启动。

备注：

打开附件程序后使用`OllyDBG`附加进程无法检测`OD`，此例子只能检测使用`OD`打开本程序的情况![](img/9b2aec7f3d4bb041b2d419a59832192f.jpg "恐怖!")

上传的附件

| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || `check_OD.rar`

其实还是`heapmagic`
