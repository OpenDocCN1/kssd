
# 不用重启更改 IP 地址

> 原文：[`www.pediy.com/kssd/pediy11/117742.html`](https://www.pediy.com/kssd/pediy11/117742.html)

最近在做一个程序，需要修改 IP 地址，但是修改后又不想重新启动，所有就有了如下程序。

原理：通过先在注册表中修改 IP 信息，然后调用 XP 的一个隐含 API 函数`DhcpNotifyConfigChange()`就可以修改 IP 地址了。

本程序有个问题就是修改 不了 DNS， 哪位高手路过帮帮忙啊。

上传的附件

| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || 改变 IP.rar
|
