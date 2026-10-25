
# UltraEdit 15 算法分析

> 原文：[`www.pediy.com/kssd/pediy10/98406.html`](https://www.pediy.com/kssd/pediy10/98406.html)

最近在找资料的时候无意在 ARTeam 论坛上面看到的一个破解 UltraEdit15 算法的教程。分析得相当到位。

寻找算法函数的大概方法：

1. 用 `exescope` 找到 `nag` 窗口的资源 ID 为 `28269`，即是：`6E6D`。
2. 然后在 `OD` 里面找到 `PUSH 6E6D` 的命令。分别下断点。
3. 在第二次中断的时候，看到 `Days to expire` 字样。向上滚动，找到这个子程序的开始。然后 `ctrl + r` 寻找引用。发现只有一个引用，于是跳去。
4. 作者凭借自己破解 14 版的经验，准确判断出关键函数的调用是哪一个。但是，下面他还是做了一些简短的分析。
5. 最后就开始了算法的分析，这个我就不多说了，太多了，自己看哈，我发在附近里面。

由于测试它教程的时候用的中文版（他用的意大利版），所以，有些地址有出入。相信很多朋友也会有一样的感受。

## 上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | | `Uedit15Keygen_ENG.rar` [请到论坛下载：[`bbs.pediy.com/showthread.php?t=98406`](http://bbs.pediy.com/showthread.php?t=98406)] |
|---|---|---|
