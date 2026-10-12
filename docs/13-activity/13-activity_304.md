
# 利用 MBRPORT 漏洞从应用层穿透奇虎 360 第 1 题

> 原文：[`www.pediy.com/kssd/pediy10/79004.html`](https://www.pediy.com/kssd/pediy10/79004.html)

这个驱动其实有一个漏洞，这个漏洞就是判断 `QueueSortKey` 是否为 1。如果是 1，则会将输入的数据全部填 0，也就是，`WINHEX` 写的时候，其实不是没写，而是全部写 0 了。

但是，如果 `QueueSortKey` 不是 1，`MBRPORT` 就直接通过了。

利用这个漏洞，可以利用 `CreateFile` 直接写数据到 `MBR+1`，几个普通的函数完全就可以搞定，不需要驱动。

这是我提交的方案之一。

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || AntiMbr-ring3.rar |
