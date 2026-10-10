
# 写插件让 OD 像 IDA 那样代码高亮(开源)

> 原文：[`www.pediy.com/kssd/pediy12/129937.html`](https://www.pediy.com/kssd/pediy12/129937.html)

想必各位都有这样的经历,在调试某些不太方便用 `IDA` 静态分析的代码时,看多了 `OD` 的各种 `ebp+` 和 `esp+`, 会无比怀念 `IDA` 的高亮和修改符号名的功能,找了一下,好像目前还没有好用的 `OD` 高亮插件

那么,我们自己动手写一个吧

原理很简单,`Hook` `OD` 的打印函数,`Hook` `TextOut` 用来做屏幕取词,然后修改当前鼠标点击字符的颜色就完了

附件中的代码没有做修改符号名的功能,大家有兴趣可以自己改一改~

上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || RAsmDbgPlugin\_bin.rar |
| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || RAsmDbgPlugin\_src.rar |
