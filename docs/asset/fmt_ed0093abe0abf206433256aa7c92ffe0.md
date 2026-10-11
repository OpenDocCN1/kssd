
# 电子小说阅读器

> 原文：[`www.pediy.com/kssd/pediy04/kanxue239.htm`](https://www.pediy.com/kssd/pediy04/kanxue239.htm)

电子小说阅读器黄金版破解手记

作者：`crackYY[CNCG]`

工具：`FileInfo v2.45` `trw2000 v1.22` `W32Dasm8.93` 黄金版

这是一个很别制的文本阅读器，我很喜欢它，不过不注册只能使用 30 次（真小气）。

首先用 `FI` 检测它是否加壳，又是没壳（难到现在流行不加壳？？）。好，请出 `TRW` 大侠，开工！运行电子小说阅读器，马上会出现一个注册窗口，随便添上一个用户名和一个注册码。我添的用户名是：`crackYY` 注册码：`78787878`，然后 `Ctrl+N` 呼出 `TRW`，下断点：`bpx hmemcpy`，按 `F5` 把控制权交给电子小说阅读器。点击确定按钮，`TRW` 弹出，下命令：`bc *` 去除断点，现下命令：`pmodule` 直接来到程序领空，看到如下代码：

```
0167:00438A1A  CALL    `USER32!CallWindowProcA`
0167:00438A1F  MOV     [EBX+0C],EAX
0167:00438A22  MOV     EAX,[EBX]
0167:00438A24  CMP     EAX,BYTE +0C
0167:00438A27  JNZ     00438A44
0167:00438A29  MOV     EDX,[EBX+08]
0167:00438A2C  PUSH    EDX
0167:00438A2D  MOV     ECX,[EBX+04]
0167:00438A30  MOV     EDX,EAX
0167:00438A32  MOV     EAX,ESI
0167:00438A34  CALL    00434870
0167:00438A39  JMP     SHORT 00438A44
0167:00438A3B  MOV     EDX,EBX
0167:00438A3D  MOV     EAX,ESI
0167:00438A3F  CALL    00435E00
```

然后按了几十次 `F12` 也没出错，气死我啦！那好，我今天就换个方法来破解这个小东东（等死吧！）。拿出我的"屠龙宝刀"——`W32Dasm`，然后反汇编电子小说阅读器，用串式参考找到："注册失败，请重新注册册！"。再向上找 `je` `jz` 等跳转指令，嗯，发现一个：

```
* Possible StringData Ref from Code Obj ->"350512"
                        |
:004B605E BA18624B00   mov edx, 004B6218
:004B6063 E84CDFF4FF   call 00403FB4 ———在这里下断点:bpx 004b6063
:004B6068 752E         jne 004B6098  ————就是这个啦！
:004B606A 6A40         push 00000040
* Possible StringData Ref from Code Obj ->"提示"
                        |
:004B606C B9E0614B00   mov ecx, 004B61E0
* Possible StringData Ref from Code Obj ->"注册失败，请重新注册！"
```

既然已经找到到啦，那还等什么？退出 `W32Dasm`，然后运行电子小说阅读器，`Ctrl+N` 呼出 `TRW`，下断点:`bpx 004b6063`，用户名：`crackYY` 注册码：`78787878`，点击确定按钮。这里 `TRW` 会弹出，拦到以下代码：

```
0167:004B605E  MOV     EDX,004B6218
0167:004B6063  CALL    00403FB4————光标停在这一行，按 F8 进入。
0167:004B6068  JNZ     004B6098
0167:004B606A  PUSH    BYTE +40
0167:004B606C  MOV     ECX,004B61E0
0167:004B6071  MOV     EDX,004B6330
0167:004B6076  MOV     EAX,[004E8C6C]
0167:004B607B  MOV     EAX,[EAX]
0167:004B607D  CALL    00453078—————————走到这里出错。
0167:004B6082  MOV     EAX,[004E8A20]
0167:004B6087  MOV     EAX,[EAX]
0167:004B6089  MOV     EAX,[EAX+0460]
0167:004B608F  MOV     DL,01
0167:004B6091  CALL    0044541C
0167:004B6096  JMP     SHORT 004B6114
```

按 `F8` 跟进后，会看到以下代码：

```
0167:00403FB1  LEA     EAX,[EAX+00]
0167:00403FB4  PUSH    EBX——————下命令：d edx 会看到一组数字：350512
0167:00403FB5  PUSH    ESI
0167:00403FB6  PUSH    EDI
0167:00403FB7  MOV     ESI,EAX
0167:00403FB9  MOV     EDI,EDX
0167:00403FBB  CMP     EAX,EDX
0167:00403FBD  JZ      NEAR 00404052
0167:00403FC3  TEST    ESI,ESI
0167:00403FC5  JZ      0040402F
0167:00403FC7  TEST    EDI,EDI
0167:00403FC9  JZ      00404036
0167:00403FCB  MOV     EAX,[ESI-04]
0167:00403FCE  MOV     EDX,[EDI-04]
0167:00403FD1  SUB     EAX,EDX
```

下命令：`bc *` 清除断点 按 `F5` 反回程序，输入用户名：`crackYY` 注册号：`350512`，点击确定。怎么回事儿？还是注册不成功？难到这个不是注册码？呵呵，这点小计量就想击退我？嗯…想一想，还有哪儿可疑。哦，对了，可以到出错的 `call` 里看看。说干就干，`Ctrl+N` 呼出 `TRW`，下断点：`bpx 004b607d`，接 `F5` 反回，点确定按钮，`TRW` 弹出，光标停在 `004B607D` 处。接看按 `F8` 跟进，看到如下代码：

```
0167:00453076  MOV     EAX,EAX
0167:00453078  PUSH    EBP————————在这里下命令:d ecx 会看到刚才的那组数字和一组字母：liyy
0167:00453079  MOV     EBP,ESP
0167:0045307B  ADD     ESP,BYTE -50
0167:0045307E  PUSH    EBX
0167:0045307F  PUSH    ESI
0167:00453080  PUSH    EDI
0167:00453081  MOV     EDI,ECX
0167:00453083  MOV     ESI,EDX
0167:00453085  MOV     [EBP-04],EAX
0167:00453088  MOV     EBX,[EBP+08]
0167:0045308B  CALL    `USER32!GetActiveWindow`
0167:00453090  MOV     [EBP-0C],EAX
0167:00453093  PUSH    BYTE +02
0167:00453095  MOV     EAX,[EBP-0C]
```

这回看到的还是刚才的那组数字，不过多了一组英文字母(`liyy`)，这个会不会是用户名呢？不管那么多啦，记下字母，按 `F5` 返回电子小说阅读器，用户名添上：`liyy` 注册码：`350512`，点击确定。"恭喜您，注册成功！您以成为本程序的正式用户…………"。哈哈，我爽!

好啦，结束战斗！啊，2 点多啦，好困呀。

`crackYY[CNCG]`

E-mail:`crackYY@hotmail.com`

写的不好，请多指教！ `4dd466d0e1e0cc123afbe504da2462ca`
