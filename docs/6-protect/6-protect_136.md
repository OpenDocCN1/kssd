
# 彩虹狗

> 原文：[`www.pediy.com/kssd/pediy04/kanxue119.htm`](https://www.pediy.com/kssd/pediy04/kanxue119.htm)

如果软件在安装的时候不需要狗，而是运行的时候需要，并且用`bpx 378`无法拦截，那么需要删掉`system`目录下面的`sentinel.vxd`这个虚拟设备驱动程序，这样一来，`sx32w.dll`会使用自己带的端口读写模块，那么，`bpx 378`等就可以使用。

4dd466d0e1e0cc123afbe504da2462ca
