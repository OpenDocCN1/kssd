
# 快速修改导入表的一种方法

> 原文：[`www.pediy.com/kssd/pediy12/129613.html`](https://www.pediy.com/kssd/pediy12/129613.html)

有这样一种情况：如果程序本身调用的函数是 `MessageBoxW`，现在要把程序中调用 `MessageBoxW` 的地方都改为 `MessageBoxA`。

常规的方法是添加一项新的导入表，然后把程序中 `MessageBoxW` 的 `RVA` 都修改为 `MessageBoxA` 的 `RVA`。

如果仅仅是实现上面所说的情况，还可以有另外一种快速修改的方法，就是直接把导入表中的 `MessageBoxW` 修改为 `MessageBoxA`。

这样只需要修改一处，就可以实现了。

只需要修改 `MessageBoxW` 字符为 `MessageBoxA`。

如果修改为其他函数（与目标函数要在同一模块）的名称，那么在程序中所有调用 `MessageBoxW` 的地方都会变为所修改的函数。

这种修改方法局限性很大，只在某种特定的情况下才能用。

![](img/447f5a3e97eb8f067fed84ae0977c501.jpg)

![](img/8f81ff81bd37f2da08dff1d91b1d44bc.jpg)
