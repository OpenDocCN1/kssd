
# WinME

> 原文：[`www.pediy.com/kssd/pediy03/forum320.htm`](https://www.pediy.com/kssd/pediy03/forum320.htm)

`MS`在`ME`上将`real mode`清除，改动`io.sys`的功能。不运行`config.sys`、`command.com`与`win.com`，直接跟`vmm32.vxd`连通，因此没有`autoexec.bat`与`INT68`（`real debugger debugger`）interface。

而`S-ICE`需要在`real mode`装载自己，但因`win.com`不运行（`win.com`会找寻并装载`real mode debugger`），又没有`autoexec.bat`，`Numega`没法运行`winice.exe`。

不知是`Numega`找到或`MS`通知，在`debug build`的`io.sys`会运行一个叫`debugger.exe`的程序，故在`Numega`站有`debugger.exe`下载。（跟`MS`提供的有异，会装载`winice.exe`）

另`Win98MEDDK`裹有`debug`用的`io.sys`（`checked io.sys or winboot.sys`），以上是原厂方法。

但另外有人发现原`io.sys`会装载另一各字的`debugger`（`loader.exe`），象`wmeldr.zip`裹便是，有源码可参考。此为土法，但比原厂的方便，`0.3`版可选装或不装`winice.exe`，现已有人说`hmemcpy`这断点不能在`ME`上工作，我没用它`debug`，`only`安装。

`4dd466d0e1e0cc123afbe504da2462ca`
