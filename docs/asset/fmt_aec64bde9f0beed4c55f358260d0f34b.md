
# 新版机器狗代码逆向

> 原文：[`www.pediy.com/kssd/pediy12/95102.html`](https://www.pediy.com/kssd/pediy12/95102.html)

`http://bbs.pediy.com/showthread.php?t=94609`

这个帖子中提到的是一个 Agent 类型的木马，它运行时会释放两个驱动程序，一个新版的机器狗代码，一个是利用 `PspTerminateProcess` 来结束进程的驱动，然后到 `http://0519qq.cn/ceshi/qq.txt` 下载这个木马列表

`http://0519qq.cn/zzx/qq1.exe`

`http://0519qq.cn/zzx/qq2.exe`

`http://0519qq.cn/zzx/qq4.exe`

`http://0519qq.cn/zzx/qq5.exe`

`http://0519qq.cn/zzx/qq6.exe`

`http://0519qq.cn/zzx/qq7.exe`

`http://0519qq.cn/zzx/qq8.exe`

`http://0519qq.cn/zzx/qq9.exe`

`http://0519qq.cn/zzx/qq10.exe`

`http://0519qq.cn/zzx/qq11.exe`

`http://0519qq.cn/zzx/qq12.exe`

`http://0519qq.cn/zzx/qq13.exe`

`http://0519qq.cn/zzx/qq14.exe`

`http://0519qq.cn/zzx/qq16.exe`

`http://0519qq.cn/zzx/qq15.exe`

`http://0519qq.cn/zzx/qq17.exe`

`http://0519qq.cn/zzx/qq18.exe`

`http://0519qq.cn/zzx/qq19.exe`

`http://0519qq.cn/zzx/qq20.exe`

`http://0519qq.cn/zzx/qq21.exe`

`http://0519qq.cn/zzx/qq22.exe`

（链接现在依然有效）

我将机器狗的代码逆向了一下，附件中是逆向得到的代码。`ANTI` 代码被注释掉了，搜索 `ANTI` 即可![](img/c1d8523a7842d7b67917c210ac6dcf18.jpg "微笑")

它只覆盖 `C:\WINDOWS\explorer.exe`。

上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) | | `scsirw.rar` |
