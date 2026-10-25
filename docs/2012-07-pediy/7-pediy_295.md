
# 『原创』调戏 `SkinSharp`

> 原文：[`www.pediy.com/kssd/pediy12/132247.html`](https://www.pediy.com/kssd/pediy12/132247.html)

最近想玩一下软件界面，百度了半天，发现 `SkinSharp` 挺强的。可是以前写的好多程序都懒得改源码了，所以想调戏一下 `SkinSharp`，在没有源码的情况下，让任何一个 `EXE` 都可以使用 `SkinSharp`。本人是很尊重知识产权的，发这个贴子完全是出于技术交流的目的。如果你想使用 `SkinSharp` 请到官网 [`www.skinsharp.com/htdocs/index.htm`](http://www.skinsharp.com/htdocs/index.htm) 购买。这里玩的 `SkinSharp` 是 1.0.6.6 版的。

为了实现目的，往 `PE` 文件里注入代码是必须的。注入的方法谁都会，程序里用的是扩展最后一个节的方法。

`SkinSharp` 的换肤功能是在 `SkinSharp.dll` 里实现的，通过调用导出的接口：`SkinH_AttachEx` 就可以为当前进程更换皮肤。可是 `EXE` 跑起来时还得带个 `SkinSharp.dll` 这个累赘，就不好玩了。所以得想办法手动把 `SkinSharp.dll` 映射到内存里。映射的方法也是谁都会。说得嗦的话就是这几个步骤（详见源码）:

1. 申请内存，把 `SkinSharp.dll` 中的数据按内存对齐映射到内存里；
2. 修复重导入表；
3. 修复重定位表；
4. 修改映像的内存保护属性；
5. 调用 `SkinSharp.dll` 的入口（如果写得通用一点的话，在调用入口之前，先处理一下 `TLS`）；
6. 收工。

`SkinSharp` 的皮肤数据是放在 `*.she` 文件中的，程序跑着还得带个 `*.she` 文件，那就更没鸟意思了！所以得想办法让皮肤数据只出现在内存里，而不会创建在磁盘上。用 `IDA` 瞅了一眼 `SkinSharp.dll`，发现偏移 `000161D0H` 处是读取皮肤数据的：

![](img/2bd4fd386c9815dc4a324cbf313417e2.jpg)

这个函数很简单，传进来一个皮肤文件名，把这个文件读入后，调用偏移 `00016290H` 处的函数把皮肤数据保存下来。所以自己写了点代码替代了这个函数，不管传入的文件名是什么，都用内存中的皮肤数据去调用偏移 `00016290H` 处的函数，`SkinSharp` 把皮肤数据保存起来。详见代码。

给看雪节省点版面，就嗦这么多。

成品：

`成品.rar`

汇编源码：

`调戏 SkinSharp.rar`
