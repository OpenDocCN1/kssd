
# Mouse Lock 1.63 脱壳教程

> 原文：[`www.pediy.com/kssd/pediy05/pediy50121.htm`](https://www.pediy.com/kssd/pediy05/pediy50121.htm)

## 概述

Asprotect 1.2x 加壳的 Mouse Lock 1.63 脱壳（很简单）。

**Mouse Lock V1.63** 软件介绍：

简单好用的鼠标锁定工具，它可设置密码将鼠标锁住、让别人无法移动鼠标、而须密码才可解开，注册版并可设置锁住的时间及自动解开的时间。

**软件下载：** `http://ln.skycn.net/down/ml.zip`

**破解工具：** `loader`，`trw2000`，`superbpm`，`Import REConstructor v1.4.2+`

这个东东是在汉化新世纪论坛上，有朋友请求帮忙脱壳的，看了论坛上诸位大哥脱 Asprotect 1.2x 的破文，手痒了，于是便找来试一试，居然成功了 :)

顺便说一下，这个软件很烂，居然也用 Asprotect 1.2x 加壳，真是受不了！

另外，这个软件是 `VC` 写的。

## 一、找程序的入口点

感谢 `fs0` 大哥的超强工具 `loader`！太厉害了！用 `loader` 载入软件主程序 `Mouslock.EXE`，轻易找到入口 `401000`，嗯？好像这个软件的入口没有被改的乱七八糟的？

## 二、用 TRW 2000 初步脱壳

下面要在入口处脱壳，先打开 `SuperBPM`，选中 `eraser` 前面的复选框，然后用 `TRW` 载入 `Mouslock.EXE`，下 `g 401000`，接着下 `pedump`，将内存中的程序脱出来。

## 三、修复导入表

打开原加壳程序，在 `Import REConstructor v1.4.2+` 的 `Attach to an Active Process` 窗口中选取 `Mouslock.EXE` 的进程，然后在下方的 `OEP` 处填入正确的入口 `0001000`，点 `IAI AutoSearch`，再点 `Get Imports`，然后点 `Show Invalid`，在 `Imported Functions Found` 窗口里的无效地址上点鼠标右键，选 `Trace Leve11(disasm)`，这样 `Import REConstructor` 能够自动修复的项目就搞定了。

再点 `show invaids`，发现还剩下几个没有修复，再次在那几个没有修复的地址上点鼠标右键，选中 `Plugin Tracer(Asprotect 1.2X Emul)`，这样这个插件能修复的项目也全部搞定了。再点 `show invaids`，所有的导入表项目都显示有效了！

现在点击 `Fix Dump`，选择 `TRW` 脱壳出来的文件，会在目录下生成修正了导入表的程序！OK，全部搞定了～

修正后的程序可以跨操作系统平台运行，在 `98se` 及 `XP` 下测试通过。

## 四、破解

这个软件的限制应该就是使用 Asprotect 加的，原程序启动时有 `NAG`，脱壳后就没有了。另外在注册用户里面还是显示未注册，用 `UE` 搜索相应的字串，改掉即可 :)

<div align="right">

炎之川 2002.5.7<br>
skipli@sina.com<br>
`http://skipli.yeah.net` `4dd466d0e1e0cc123afbe504da2462ca`

</div>
