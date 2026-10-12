
# 无驱 RING3 超稳定注册表还原保护

> 原文：[`www.pediy.com/kssd/pediy10/86501.html`](https://www.pediy.com/kssd/pediy10/86501.html)

无驱 RING3 超稳定注册表还原保护。

最近很多同学都在完注册表保护，有的用磁盘&总线过滤，有的用 HOOK 和猥琐的 CCB~

可惜驱动总是不稳定呀~

我也放一个 RING3 无驱、超稳定的注册表还原保护，`EXE` 仅 `20K` 大小~

仔细看过源码中的注册表处理的，就应该能留意到这点 ，呵呵

运行后重启，所有对 `SYSTEM` 键下面的操作都会被重启还原~

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `easyRegProt.rar` |
