
# ARM 反汇编器

> 原文：[`www.pediy.com/kssd/pediy10/85698.html`](https://www.pediy.com/kssd/pediy10/85698.html)

别人基础上做的。
关键是没时间弄了。本来想弄出来，如果能卖些钱就卖些钱的![](img/f43c1e514327696dccbb0da997496b2a.jpg "扮鬼脸") 。后来也懒得弄了，觉得用来学习倒是不错了。做的完善了，可以作为 IDA 的反汇编的插件。
更新了一下附件，把工程文件加上去了。

上传的附件

| ![](img/fb8ade91a9e54b7bd6690c3ae5b1d1cc.jpg) || TestArm.zip  |

在 VC 中加入这些文件就可以编译了。
函数好像是调用 `disarm2`
参数是 `disarm2(汇编指令, 汇编地址）`
参考 `TestArm.cpp` 中的例子。
本来我想把 thumb 指令也写了的。后来由于时间关系写了个结构体定义。其他的没有时间弄了。
希望有心人能够把它完成。
了解这个方面的人会知道这些代码虽然不多，但是很重要。

上传的附件

| ![](img/fb8ade91a9e54b7bd6690c3ae5b1d1cc.jpg) || disthumb.zip  |
