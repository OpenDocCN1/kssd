
# OllyDBG 中快速定位 VB 按钮的处理程序

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-799.htm`](https://www.pediy.com/kssd/pediy07/pediy7-799.htm)

今天看到 AndreaGeddon 的关于 VB 反汇编的一篇教程(`http://www.reteam.org/papers/e46.pdf`),根据他的介绍,我总结出了以下的快速定位 VB 按钮 `OnClick` 的处理程序.

我以 windayjian 的伊朗程序员的 VB `CrackMe` 为例示范一下步骤(因为刚看过,就没有找其它的例子).

1. 运行原程序,知道注册的按钮是”OK”

2. 所以就在 `OllyDBG` 中搜索字符串”OK”,结果找到如下的地方:`0040BA1D`

```
0040BA1D | ASCII "&OK",0
```

3. 然后双击,我们就在 `CPU` 窗口中了,往上看,我们看到”Command1”,好,我们知道了 `OK` 按钮的名字是”Command1”.

```
0040BA10 . 43 6F 6D 6D 61 6E 64 31 00 ASCII "Command1",0
0040BA19 04 DB 04
0040BA1A 01 DB 01
0040BA1B 03 DB 03
0040BA1C 00 DB 00
0040BA1D . 26 4F 4B 00 ASCII "&OK",0
0040BA21 04 DB 04
0040BA22 B8 DB B8
0040BA23 0B DB 0B
0040BA24 80 DB 80
```

4. 然后回到字符串参考窗口,搜索”Command1”,注意要在右边的那一栏,结果我们找到了两个,一个在 `00407474`, 另一个在 `00407828`. 这里因为我们不知道到底是哪一个,所以两个都试试. 先来 `00407474` 的那个,双击它.

```
00407474 | DD CrackMe2.00407FFC | ASCII "Command1"
00407828 | DD CrackMe2.00407FFC | ASCII "Command1"
```

5. 在 `CPU` 窗口中往上隔一个 `DD` 的位置 `0040746C`,我们把光标移到这一行,`OllyDBG` 自动帮我们加了跳转.按回车键,就自动带到了目的地 `00407658`

```
0040746C /58764000 DD CrackMe2.00407658
00407470 |E0 DB E0
00407471 |B4 DB B4
00407472 |19 DB 19
00407473 |00 DB 00
00407474 |FC7F4000 DD CrackMe2.00407FFC ; ASCII "Command1"
00407478 |01 DB 01
00407479 |00 DB 00
0040747A |03004000 DD CrackMe2.00400003
```

6. 然后我们就看到了整整齐齐的一排地址,注意`<JMP.&MSVBVM60.EVENT_SINK_Release>`下面的那一行 `0040767D`, 所指示的地址 `0040771D` 就是对应的处理程序

```
00407658 00 DB 00
00407659 00 DB 00
0040765A 00 DB 00
0040765B 00 DB 00
0040765C 54744000 DD CrackMe2.00407454
00407660 30734000 DD CrackMe2.00407330
00407664 88194000 DD <JMP.&MSVBVM60.EVENT_SINK_QueryInterface>
00407668 8E194000 DD <JMP.&MSVBVM60.EVENT_SINK_AddRef>
0040766C 94194000 DD <JMP.&MSVBVM60.EVENT_SINK_Release>
00407670 1D774000 DD CrackMe2.0040771D
00407674 00 DB 00
00407675 00 DB 00
00407676 00 DB 00
00407677 00 DB 00
```

7. 我们再按回车键,就到了可以下断点的地方了,好了在 `0040771D` 下断点

```
00407710 . 816C24 04 3B000000 SUB DWORD PTR SS:[ESP+4],3B
00407718 . E9 63460000 JMP CrackMe2.0040BD80
0040771D . 816C24 04 43000000 SUB DWORD PTR SS:[ESP+4],43
00407725 . E9 C6460000 JMP CrackMe2.0040BDF0
0040772A . 816C24 04 47000000 SUB DWORD PTR SS:[ESP+4],47
00407732 . E9 A9470000 JMP CrackMe2.0040BEE0
00407737 . 816C24 04 37000000 SUB DWORD PTR SS:[ESP+4],37
0040773F . E9 0C480000 JMP CrackMe2.0040BF50
00407744 . 816C24 04 FFFF0000 SUB DWORD PTR SS:[ESP+4],0FFFF
0040774C . E9 8F4B0000 JMP CrackMe2.0040C2E0
00407751 . 816C24 04 3B000000 SUB DWORD PTR SS:[ESP+4],3B
00407759 . E9 124D0000 JMP CrackMe2.0040C470
```

8. 对于另外一个”Command1”, 我们做同样的处理. 然后就可以跑程序了,试试点点按钮,是不是被 `OllyDBG` 断下来了,试了之后知道其中一个”Command1”是对应”Register”的

还有,如果是中文的按钮,在字符串参考中找不到,这时候可以在 `CPU` 窗口中 `Ctrl+B` 来搜索.

### 这样是不是更快

1. `od` 下运行 VB 程序，`F12` 暂停；
2. `View` 菜单中选击 `Windows` 项，在打开的窗口中可以从 `Title` 栏看到目标按钮，从而找到它的 `Handle(000xxxxx)` ;
3. 在 `CallWindowProcA` 入口下条件断点 `[esp+8]==xxxxx && [esp+0c]==202`；
4. `F9` 继续程序，点击目标按钮，程序中断；
5. `Alt+F4`，在代码段（`.text`）上下访问断点；
6. `F9` 执行程序，（`xp sp1` 下）程序断在下面

```
73442FA4 FF3418 PUSH DWORD PTR DS:[EAX+EBX] ; yyyyyyy
```

上面`[EAX+EBX]`的值（`yyyyyy`）就是我们要找到位置啦。

用活 `OD`，事半功倍。

`4dd466d0e1e0cc123afbe504da2462ca`
