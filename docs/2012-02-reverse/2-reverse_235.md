
# 小技巧

> 原文：[`www.pediy.com/kssd/pediy04/kanxue134.htm`](https://www.pediy.com/kssd/pediy04/kanxue134.htm)

由于加过壳的程序的 `section characteristics` 一般都被改了，所以用 `SoftICE symbol loader` 加载被加壳了的 `exe` 后无法停在程序的入口处，而是立即运行了。

一般是采用修改 `section characteristics` 为 `E0000020` 或者将入口指令改为 `int 3` 的办法，但难于躲过 CRC 校验。

在 `Win2K` 下可以按照下面的方法使得程序在入口处停下：

1、用 `pe editor` 得到加壳后的入口为 `xxxxxxxx`。

2、`BPX LoadLibraryA`

3、用 `symbol loader` 加载被加壳了的 `exe`，第一次中断后处在系统的 `rpcrt4.dll` 中。

4、清除断点，设断点 `BPM CS:xxxxxxxx X` 即可。
