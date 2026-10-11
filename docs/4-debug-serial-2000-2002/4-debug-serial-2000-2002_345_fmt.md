
# HAPPYEO 电子琴 2.31

> 原文：[`www.pediy.com/kssd/pediy03/forum1026.htm`](https://www.pediy.com/kssd/pediy03/forum1026.htm)

软件名称：HAPPYEO 电子琴

整理日期：2001.5.21

最新版本：2.31

文件大小：1532KB

软件授权：共享软件

使用平台：Win9x/Me/NT/2000

发布公司：home page

软件简介：

一个用电脑的声卡和键盘来模拟电子琴的程序。主要功能特点：有两个独立的键盘，用左右手分别演奏。可分别设置不同的音色、音区和效果。128 种音色，复音数仅受声卡限制。12 种效果！(包括自动双音、自动和弦、双音色混合，甚至还有模拟弹拨乐器的音色）。有自动伴奏功能！12 种自动伴奏风格，支持节奏填充、节奏同步启动。可以录音，放音，暂停，快放，循环放音。放音的同时仍然可以正常演奏！同时支持声卡和喇叭。即使使用喇叭，也有四种不同的“音色”。界面逼真，操作方便，同时支持鼠标和键盘。帮助详尽。绿色软件，无需安装。2.0 正式版主要功能特点：(New!)支持 MIDI 文件的输入和输出！(New!)支持 MIDI 键盘和带 MIDI 接口的电子琴！支持多指和弦！(New!)支持多音轨录音！(New!)同时支持 GM、XG、GS 标准。可以自由地编辑 XG 和 GS 里的近千种音色！有两个独立的键盘，用左右手分别演奏。可分别设置不同的音色、音区和效果。128 种音色，复音数仅受声卡限制。11 种效果!(包括自动双音、自动和弦、双音色混合，甚至还有模拟弹拨乐器的音色)。有自动伴奏功能！12 种自动伴奏风格，支持节奏填充、节奏同步启动。可以录音，放音，快放，循环放音。可以“边放边弹”和“边放边录”！同时支持声卡和喇叭。即使使用喇叭，也有四种不同的“音色”。界面逼真，操作方便，同时支持鼠标和键盘。绿色软件，无需安装。

破解工具：`trw2000`

1. 添注册名，地址，注册码。其中名字不能少于 4 个字符，地址不能少于 12 个字符注册码为 6 个字符

   呵呵，我怎么知道的？因为我跟了一边，他告诉我的，呵呵

2. 下断点 `hmemcpy`，`F5`，`bc * pmodule`

3. `F10` 单步跟踪，可以看出它把你输入的东东做一下变换，压入堆栈，最后的一个 `call` 是算码的 `call`

   看见最后一个 `call` 以后 `F8` 进入，`F10` 一直到 `5109885e` 下 `d edx` 看见注册码

我的注册信息：

`name`：`yahoo`

`addr`:`hebeidaxuejisuanjikexuexueyuan`(长吧）

`code`：`3E5NCG`

写个 `crackcode.ini`

`crackcode.ini`

```ini
[Options]
CommandLine=happyeo.exe
Mode=2
First_Break_Address=5109885F
First_Break_Address_Code=E8
First_Break_Address_Code_Lenth=5
Second_Break_Address=5100403F
Second_Break_Address_Code_Lenth=2
Save_Code_Address=EDX 4dd466d0e1e0cc123afbe504da2462ca
```
