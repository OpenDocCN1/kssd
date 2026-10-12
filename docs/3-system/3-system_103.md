
# 程序自动实现写入 Tls CallBack

> 原文：[`www.pediy.com/kssd/pediy10/64848.html`](https://www.pediy.com/kssd/pediy10/64848.html)

这个程序断断续续的写了 N 天了，到今天总算能运行了……

往 IE 和 Notepad 里面写，都成功了，但程序 Bug 巨多，而且思维一点不严密。

现在测试就是手动写的 `MessageBoxA` 的地址，然后调用该函数。

大概的思路如下：

输入 `Tls CallBack` 的内容，然后我的程序取得内容之后，写到 `temp.asm` 文件中，然后调用 `nmake` 命令，执行 `nmake -f Makefile3`，来将 `temp.asm` 编译完成，然后将需要的二进制代码从里面挖出来，然后写到 `Binaray.dat` 文件中。

然后选择你要写入的文件，即可判断 PE，然后将数据写入可执行文件中。

使用就是先 “生成二进制代码”然后再“选择文件”，将代码写入。

现在还有好几个问题

(因为是编程初学者，请大家多多包涵呀～)

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | `Tls.rar` |
| :--- | :--- |
