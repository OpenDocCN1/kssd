
# RegSnap 2.6

> 原文：[`www.pediy.com/kssd/pediy03/forum631.htm`](https://www.pediy.com/kssd/pediy03/forum631.htm)

\-----------------------先从 NAG 窗口入手

用 `W32dsm89` 反汇编，

Find Text :"This software has not been registered"

从 `4045CD` 往上，"* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0040456E(C) "

说明 NAG 窗口由 `40456E` 调用，

从 `40456E` 往上，"* Referenced by a CALL at Addresses:

|:0040148F , :004058BD , :00411FE7 "

这表明了在这个软件中共有三次识别软件是否注册的过程 . 因此我们在 CRACK 时 , 要注意在这三个地方进行 , 少了其中一个都不成功 .

将`:004058BD E89EECFFFF call 00404560`

等三处改为 `90 90 90 90 90 nop`

这样就 NAG 窗口就变为"Stand version"

`OK 4dd466d0e1e0cc123afbe504da2462ca`
