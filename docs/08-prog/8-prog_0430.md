
# 一个即将成形的 rootkit，愿与大家共享，相互学习，共同进步

> 原文：[`www.pediy.com/kssd/pediy10/83964.html`](https://www.pediy.com/kssd/pediy10/83964.html)

由于本人才疏学浅，经过 N 天的努力，总算弄成现在这个样子，传上来与大家共享，相互学习，共同进步！现在应该还不能叫做 rootkit,只能算是一个 ndis 的协议驱动。实现了如下功能：

1. 盗用别人的代码，实现一个 NDIS 协议驱动。
2. 盗用 `rk_044` 的代码，实现了 `telnet`，很不稳定；原因是实现的 TCP/IP 协议不稳定，这将是我下一步努力的重点，有兴趣的朋友，咱们可以共同努力。我 QQ:282461168。

这个程序使用 `2000DDK+VC6` 开发，在 `vmware+winxp` 下测试，你若想跑起来，需要做如下工作：

1. 把文件 `/ProtoDrv/ndisprot.cpp` 178 行改成你要测试机器网卡的 `ServiceName`。
2. 编译生成，安装驱动。
3. 安装的服务名若不是 `HelloDDK`，更改文件 `/RawEthernet/protoutils.cpp` 20 和 57 行为正确的服务名。
4. 启动 `windbg`，可查看输出的调试信息，或单步调试程序。

说说开发目标吧，因为现在完成的真的没什么可以说的，惭愧，惭愧！！！

1. 稳定的 `tcp/ip` 协议栈。
2. 内核 `Shell`。
3. 隐藏进程，通信端口和驱动程序，实现键盘 `Sniffer`。
4. 声卡过滤驱动和显卡过滤驱动，这两个没整过，可能表达的就不专业，向大家多多学习。

源代码下载：`download.csdn.net/source/1115409`

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | | `rootkit.rar` |
