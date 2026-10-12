
# 关于 `createfile` 研究

> 原文：[`www.pediy.com/kssd/pediy05/pediy50145.htm`](https://www.pediy.com/kssd/pediy05/pediy50145.htm)

## 论“`createfile("\\\\.\\trw2000")` 不能检测 `trw2000` 1.23 版本之存在”

自从用 API 函数 `createfile("\\\\.\\Sice")` 能有效检测 `softice` 存在与否的方法被揭示以来，此方法被推而广之，广泛运用于各种 anti 技术中。比如大家很熟悉的：

```
createfile("\\\\.\\trw2000")   //检测 trw2000
createfile("\\\\.\\trwdebug")  //检测 trw2000
createfile("\\\\.\\filemon")   //检测 filemon
createfile("\\\\.\\regmon")    //检测 regmon
```

等等。

特别是 `createfile("\\\\.\\trw2000")` 更是被多个国产软件所运用，来对付 `trw2000`。但是，实验发现，这种方法不能检测到 `trw2000` 1.23 版本，其余版本可以检测到。

自从见到 `createfile("\\\\.\\Sice")` 检测 `softice` 后，我当然也举一反三，大肆试验，用此函数检测其它程序，但是检测 `filemon`、`regmon` 都成功，唯独不能检测到 `trw2000`（我是 1.23 版本）存在；我也询问其它人，部分人与我有同感；难道这个方法有限制，此疑惑一直存于我心。

直到最近，我从一些 `vxd` 教程中知道 `createfile("\\\\.\\Sice")` 检测的原理，并作了一些试验。

在 `win9x` 下，`win32` 程序与 `vxd` 程序彼此沟通分 2 部分：1 部分是应用程序到 `vxd` 通信；另一层是 `vxd` 至应用程序通信。

`win32` 程序对 `vxd` 程序通信时，是通过调用 `DeviceIoControl` 函数进入 `vxd`，此函数的一个参数就是由 `createfile` 获得的设备句柄。这样，同样生成或打开文件的调用也能够打开一个到 `vxd` 的通道。要用 `createfile` 打开一个 `vxd`，而不是一个通常的文件，在文件名的地方必须使用特殊形式：

```
createfile("vxdname")
```

（摘自《windows vxd 与设备驱动程序权威指南》第二版）

如此，很显然了，`createfile("\\\\.\\Sice")` 能检测是因为 `sice` 驻留后内存中有个 `vxd` 叫 `sice` 形成(当然，实际硬盘上不存在一个叫 `sice.vxd` 的程序）。

各位可以从以下网址下载一个软件 `vxdview`，它用来显示内存中驻留的 `vxd`

`http://www.tssc.de/download/prods/vxdview.exe`

其结果显示 `trw2000` 1.23 版本运行前、后系统中 `vxd` 没有改变；

`trw2000` 1.22、1.03 版本运行前、后比较，系统中增加一个 `vxd`，名字为 `trwdebug`

再用 `filemon` 测试，显示增加一个 `vxd`，名字为 `filemon`。

### 结论 1
`trw2000` 1.23 版本运行后（但没有激活中断情况下），没有生成 `vxd`，所以 `createfile("\\\\.\\trw2000")` 不能成功。

但是用 `ctrl-N` 激活 `trw2000` 1.23 版本后，下命令：

```
vxd //显示所有 vxd
```

可以见到 `trw2000` 名字，说明只有在 `trw2000` 激活，接手系统后才会生成 `vxd`，来实行跟踪、显示等功能；可是此时，`createfile("\\\\.\\trw2000")` 已经无能为力，因为一切在 `trw` 掌握中；等到用 `F5` 结束 `trw2K` 的激活，`vxd` 也就消失了。

### 结论 2
`trw2000` 1.23 版只有在激活后才生成 `vxd`。

所以。最后结论是：“`createfile("\\\\.\\trw2000")` 不能检测 `trw2000` 1.23 版本之存在，但可以用来检测低级版本”。

也许刘滔滔在 1.23 版本中用了 `wdm` 编程技术导致不能检测？但是 1.22 版本也是 `wdm` 编程，为什么可以检测？或者，作者在 1.23 版本中才完全实现了 `wdm` 编程？那么基于 `wdm` 在 `win2K` 的通用性，也许可以期待 `trw2000 for win2k` 的出现！

以上是我的研究过程，不管是赞同还是反对，都请各位提出观点及相应的论据；并请指出我表达中的错误。

谢谢 `4dd466d0e1e0cc123afbe504da2462ca`
