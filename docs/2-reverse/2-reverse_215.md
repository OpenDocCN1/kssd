
# 编写脚本增强 windbg 堆栈、内存窗口【有码有真相啊】

> 原文：[`www.pediy.com/kssd/pediy12/139816.html`](https://www.pediy.com/kssd/pediy12/139816.html)

[`hi.baidu.com/isuae`](http://hi.baidu.com/isuae)  
[`www.krnldbg.com`](http://www.krnldbg.com)

因为要频繁地使用 windbg 分析堆栈、参数、以及内存状态，但是 windbg 的界面跟 OD、IDA 差距是很大的，对于漏洞分析或者其它用途来说，用它来分析堆栈、内存是很不方便的，需要多次手工输入命令。

为了简化分析工作，自己闲着蛋疼的时候写了两个脚本来增强 windbg 的这些功能。

### 准备：下载以下脚本，放置在 windbg 的安装目录。【有码有真相啊】

`krnldbg.rar`

如上网站打不开，试试这个：`http://115.com/file/bhigox4v#krnldbg.rar`

首先说函数调用栈及参数 dump 脚本，当 windbg 命中断点后，用快捷键”ctrl+n”增加一个命令窗口，输入命令 `$$>a<advstk` ，即可将栈、及调用栈中的函数参数列出来，蓝色带链接的证明这个内存地址是可访问的，所以他极有可能是一个指针，所以做了一个内存“链接”，点击后可直接使用另一个脚本查看该处的内存。

![](img/d9d49c006f41c8a3dd0f53b6c86098ed.jpg)

注: 如果 `advstk` 执行出错,请换用脚本 `advstk1`,因为我使用的方法很笨,一些细节没处理好.

接上，跳转到内存查看脚本后，会把这个内存地址进行不同的方式解析，如 `db`,`dd`,`du`,`da`,目的是减少命令的输入次数，并且还会进一步解析这块内存中储存的是否为指针，如果内存可访问，会进一步再给其加上链接，以便于分析时进一步跟踪。

![](img/691795bd6ef13ebc5c4af5e5f329aaa7.jpg)

![](img/ad00d6fd31c27b38bbedcb25985c4963.jpg)

当然也可以直接在命令窗口中输入命令 `$$>a<advmem 0x06FAEF80` 查看 `0x06FAEF80` 处内存情况

接着下来的问题是怎样让这个命令窗口像真正像 windbg 的 view“窗口”那样即时刷新呢？

答案是在命令窗口上右键，勾选“Auto refresh”，如果不出意外，他就带上了自动刷新的功能了，如下图：

![](img/9f43b34cc456dbcbbf57d10e0007f2c6.jpg)
