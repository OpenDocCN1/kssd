
# 第二题的办法

> 原文：[`www.pediy.com/kssd/pediy10/78991.html`](https://www.pediy.com/kssd/pediy10/78991.html)

`nids` 驱动俺不是很熟，不过俺跟了一下，最后跟到 `ndisDeviceControlIrpHandler` -> `ndisQueryStatisticsOids`。OK，`HOOK` 之。

现在的问题就是怎么找了，`\\Driver\\NdisWan` 里面就有。俺是懒虫，未仔细研究传入的参数，`HOOK` 后的 `ndisDeviceControlIrpHandler` 直接返回了，不过似乎并不影响网络传输，反正是达到不再闪烁的目标了。

当然也可以 `SSDT` `HOOK` 拦，但其实原理一样。

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || 第二题.rar |
