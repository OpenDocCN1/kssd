
# Midi 简谱输入系统(version 0.8.0 Build 4)

> 原文：[`www.pediy.com/kssd/pediy04/kanxue353.htm`](https://www.pediy.com/kssd/pediy04/kanxue353.htm)

Midi 简谱输入系统(version 0.8.0 Build 4)注册破解

软件版本：Midi 简谱输入系统 0.8.0 Build 4

下载地址：http://www.widenstage.com/artech/product/midiinput/midiinput.zip

软件大小：396k

软件简介：MIDI 简谱快速输入是一个 MIDI 作曲的辅助输入软件，采用与简谱近似的最为直观的命令行输入方式，并以简谱的形式显示出来，具有删除、插入、修改等基本编辑功能。

破解工具：`Trw2000`

由于没有脱壳和反汇编，所以代码较简单（都是手工输入的呀！）

运行“Midi 简谱输入系统”出现注册窗口。

机器 ID 号：`A9090909090909090`

您的 EMAIL：`Http://esoft2001.51.net`

注册号码：`987654321`

`Ctrl-n`，下命令 `bpx hmemcpy`，`F5` 返回，按“确定”按钮，`Trw2000` 拦下。

下 `bc *` 命令

再下 `Pmodule` 命令，按 7 下 `F12`，再按 `F10` 程序来到下面一行

```
0040A22D  LEA EDX, [EBP+0C]
```

从这里开始一直按 `F10`，不久会弹出“注册码错误”的对话框，代码如下：

```
0040A4F2  CALL 00436D60
```

向上看一下，看从哪里跳过来，发现下面的“经典结构”

```
0040A455  CALL 0040A95C（F8 进入这个 CALL，可能有戏）

0040A45A  TEST AL,AL

0040A45C  JZ 0040A4BA
```

依次进入下面的几个 `CALL`

```
0040AC78  CALL 0045D814

0045E3C5  CALL 0045DEFC

0045DF8C  CALL 0045E027
```

进入 `0045DF8C CALL 0045E027` 后按 `F10`，到下面

```
0045E0EF  PUSH ECX

………………

0045E123  JNZ 0040E0EF
```

这段代码将注册码算出放在 `ESI` 中。所以执行到下面

```
0045E125  LEA ECX，[EBP-51]
```

下命令 `D ESI` 看到正确的注册码 `D1DB64E871152109`。

`esoft2001.51.net`

2002 年 1 月 19 日 `4dd466d0e1e0cc123afbe504da2462ca`
