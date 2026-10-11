
# 中华压缩 V7.0

> 原文：[`www.pediy.com/kssd/pediy04/kanxue255.htm`](https://www.pediy.com/kssd/pediy04/kanxue255.htm)

**作者**：`crackYY[CNCG]`

**工具**：`FileInfo v2.45` `TRW2000 V1.22` `procdump v1.62` `keymake v1.34`

这个东东不注册只能用 30 天，每次运行都会提示你去注册它，功能方面我还不清楚，反正我也不打算用它。下面我就介绍一下它的破解经过。

首先用 `FI` 检测一下是否加了壳，我靠，是用 `ASPack2001` 加的壳。用我的 `Procdump` 脱壳（这个壳是我后加的，原版没有）。请出 `TRW` 大侠上场（鼓掌呀，啊…什么东西砸我！），再请出今天的主角中华压缩（不准乱扔瓜果皮壳！）。好，现在它提示你注册啦！当然是点击注册按钮啦（不过我不打算花钱: D），用户名和注册码随便添一个就行，我写的用户名是：`crackYY`，注册码：`78787878`，然后 `Ctrl+N` 呼出 `TRW`，下断点：`bpx hmemcpy`，再按 `F5` 把控制权交给中华压缩。接着点击确定按钮，`TRW` 会弹出，这时看到的是系统的领空，所以下命令：`bc *` 清除断点，再下命令：`pmodule` 直接跳到程序的领空，这时会看到如下代码：

```
0167:00436D0E  CALL    `USER32!CallWindowProcA`
0167:00436D13  MOV     [EBX+0C],EAX
0167:00436D16  MOV     EAX,[EBX]
0167:00436D18  CMP     EAX,BYTE +0C
0167:00436D1B  JNZ     00436D38
0167:00436D1D  MOV     EDX,[EBX+08]
0167:00436D20  PUSH    EDX
0167:00436D21  MOV     ECX,[EBX+04]
0167:00436D24  MOV     EDX,EAX
0167:00436D26  MOV     EAX,ESI
0167:00436D28  CALL    00432B24
0167:00436D2D  JMP     SHORT 00436D38
0167:00436D2F  MOV     EDX,EBX
0167:00436D31  MOV     EAX,ESI
0167:00436D33  CALL    004340F4
```

由于我们下的断点是：`bpx hmemcpy`，所以要继续向下走，一直找到出错的地方，然后再向回分析，找到正确的注册码。按 7 次 `F12`（超过 7 次就会出错），然后再向下分析：

```
0167:00432F51  POP     EBX
0167:00432F52  RET
0167:00432F53  NOP
0167:00432F54  PUSH    EBP
0167:00432F55  MOV     EBP,ESP
0167:00432F57  PUSH    BYTE +00
0167:00432F59  PUSH    EBX
0167:00432F5A  PUSH    ESI
0167:00432F5B  MOV     ESI,EDX
0167:00432F5D  MOV     EBX,EAX
0167:00432F5F  XOR     EAX,EAX
0167:00432F61  PUSH    EBP
0167:00432F62  PUSH    DWORD 00432FA9
0167:00432F67  PUSH    DWORD [FS:EAX]
0167:00432F6A  MOV     [FS:EAX],ESP
0167:00432F6D  LEA     EDX,[EBP-04]
0167:00432F70  MOV     EAX,EBX
0167:00432F72  CALL    00432F24
0167:00432F77  MOV     EAX,[EBP-04]
0167:00432F7A  MOV     EDX,ESI
0167:00432F7C  CALL    0040411C
```

按 `F10` 来到如下代码处：

```
0167:004F4DBF  CALL    00432F24
0167:004F4DC4  MOV     EAX,[EBP-08]
0167:004F4DC7  PUSH    EAX
0167:004F4DC8  LEA     EDX,[EBP-10]
0167:004F4DCB  MOV     EAX,[EBX+02E0]
0167:004F4DD1  CALL    00432F24
0167:004F4DD6  MOV     EDX,[EBP-10]
0167:004F4DD9  LEA     ECX,[EBP-0C]
0167:004F4DDC  MOV     EAX,EBX
0167:004F4DDE  CALL    004F4FAC
0167:004F4DE3  MOV     EDX,[EBP-0C]
0167:004F4DE6  POP     EAX
0167:004F4DE7  CALL    0040411C  这里按 F8 进入
0167:004F4DEC  JNZ     004F4E64  跳过去就完蛋！
0167:004F4DEE  MOV     DL,01
0167:004F4DF0  MOV     EAX,[00452558]
0167:004F4DF5  CALL    00452658
0167:004F4DFA  MOV     [EBP-04],EAX
0167:004F4DFD  XOR     EAX,EAX
0167:004F4DFF  PUSH    EBP
0167:004F4E00  PUSH    DWORD 004F4E5D
0167:004F4E05  PUSH    DWORD [FS:EAX]
0167:004F4E08  MOV     [FS:EAX],ESP
0167:004F4E0B  MOV     CL,01
0167:004F4E0D  MOV     EDX,004F4EA8
0167:004F4E12  MOV     EAX,[EBP-04]
```

按 `F10` 走到 `004F4DE7` 处，用 `F8` 跟进，看到如下代码：

```
0167:00404119  LEA     EAX,[EAX+00]
0167:0040411C  PUSH    EBX  这里可以看到真的注册码
0167:0040411D  PUSH    ESI
0167:0040411E  PUSH    EDI
0167:0040411F  MOV     ESI,EAX
0167:00404121  MOV     EDI,EDX
0167:00404123  CMP     EAX,EDX
0167:00404125  JZ      NEAR 004041BA
0167:0040412B  TEST    ESI,ESI
0167:0040412D  JZ      00404197
0167:0040412F  TEST    EDI,EDI
0167:00404131  JZ      0040419E
0167:00404133  MOV     EAX,[ESI-04]
0167:00404136  MOV     EDX,[EDI-04]
0167:00404139  SUB     EAX,EDX
```

在 `0040411c` 处，下命令：`d edx` 会看到真正的注册码，下命令：`d eax` 会看到我刚才输入的假注册码。

好了，到了最后一步，来个注册机玩（谁让我看你不顺眼，让你永世不得翻身！）。请出 `keymake`，选择另类注册机，添法如下：

第一个地址：
*   中断地址：`4F4DE7`
*   中断次数：`1`
*   第一字节：`E8`
*   指令长度：`5`

第二个地址：
*   中断地址：`40411C`
*   中断次数：`1`
*   第一字节：`53`
*   指令长度：`1`

终于结束战斗啦，对啦，这个东东可是加了壳的哦，别忘了选上"经过加壳"呀！

`E-mail:crackYY@hotmail.com`
`oicq:20651482` `4dd466d0e1e0cc123afbe504da2462ca`
