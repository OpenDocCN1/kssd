
# 小实验：用 OD 改 VB 的 EXE 中 `integer` 变量为 `long`

> 原文：[`www.pediy.com/kssd/pediy11/116764.html`](https://www.pediy.com/kssd/pediy11/116764.html)

几年前我的一个同事，请我帮他改一个 `VB` 写的 `EXE`。说是程序中有个变量定义成 `integer` 了老是会溢出，请我帮忙改成 `long`。还没等我开始研究，他就找到了程序原作者，帮他重写了程序。

最近，想起这件事来，好奇心又上来了，想看看到底要怎样改。

他给我的那个 `EXE` 已经找不到了，同时水平有限也不敢一下搞个复杂的。于是就先写个简单的试试：

1. `Vb` 只加一个按扭和两个 `TEXT` 控件：

```vb
Private Sub Command1_Click()
Dim a1 As Integer
Dim b1 As Long

a1 = 32760
a1 = a1 + 1
a1 = a1 + 1
a1 = a1 + 8
b1 = 32760

Text1.Text = a1
Text2.Text = b1
End Sub
```

2. 点 `command1` 后马上就溢出了。
3. 用 `OD` 打开这个 `exe`。

```asm
00401C00   \55            PUSH EBP
00401C01    8BEC         MOV EBP,ESP
00401C03    83EC 0C      SUB ESP,0C
00401C06    68 B6104000  PUSH <JMP.&MSVBVM60.__vbaExceptHandler>  ; SE 处理程序安装
00401C0B    64:A1 00000000 MOV EAX,DWORD PTR FS:[0]
00401C11    50           PUSH EAX
00401C12    64:8925 0000000>MOV DWORD PTR FS:[0],ESP
00401C19    83EC 24      SUB ESP,24
00401C1C    53           PUSH EBX
00401C1D    56           PUSH ESI
00401C1E    57           PUSH EDI
00401C1F    8965 F4      MOV DWORD PTR SS:[EBP-C],ESP
00401C22    C745 F8 9810400>MOV DWORD PTR SS:[EBP-8],工程 2.00401098
00401C29    8B75 08      MOV ESI,DWORD PTR SS:[EBP+8]
00401C2C    8BC6         MOV EAX,ESI
00401C2E    83E0 01      AND EAX,1
00401C31    8945 FC      MOV DWORD PTR SS:[EBP-4],EAX
00401C34    83E6 FE      AND ESI,FFFFFFFE
00401C37    56           PUSH ESI
00401C38    8975 08      MOV DWORD PTR SS:[EBP+8],ESI
00401C3B    8B0E         MOV ECX,DWORD PTR DS:[ESI]
00401C3D    FF51 04      CALL DWORD PTR DS:[ECX+4]
00401C40    33C0         XOR EAX,EAX
00401C42    8B16         MOV EDX,DWORD PTR DS:[ESI]
00401C44    8945 E0      MOV DWORD PTR SS:[EBP-20],EAX
00401C47    8945 DC      MOV DWORD PTR SS:[EBP-24],EAX
00401C4A    B8 F97F0000  MOV EAX,7FF9         ;就是这里了 7FF9=32760+1
00401C4F    56           PUSH ESI
00401C50    66:05 0100    ADD AX,1             ;加 1
00401C54    0F80 0A010000  JO 工程 2.00401D64    ;判断 OF 是否为 1，就是判断是否溢出
00401C5A    66:05 0800    ADD AX,8             ;加 8
00401C5E    0F80 00010000  JO 工程 2.00401D64   ; 判断是否溢出
00401C64    8BD8         MOV EBX,EAX
00401C66    FF92 FC020000  CALL DWORD PTR DS:[EDX+2FC]
00401C6C    50           PUSH EAX
00401C6D    8D45 DC      LEA EAX,DWORD PTR SS:[EBP-24]
00401C70    50           PUSH EAX
00401C71    FF15 20104000  CALL DWORD PTR DS:[<&MSVBVM60.__vbaObjSe>; MSVBVM60.__vbaObjSet
00401C77    8BF8         MOV EDI,EAX
00401C79    53           PUSH EBX
00401C7A    8B17         MOV EDX,DWORD PTR DS:[EDI]
00401C7C    8955 C8      MOV DWORD PTR SS:[EBP-38],EDX
00401C7F    FF15 00104000  CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrI2>; MSVBVM60.__vbaStrI2
00401C85    8BD0         MOV EDX,EAX
00401C87    8D4D E0      LEA ECX,DWORD PTR SS:[EBP-20]
00401C8A    FF15 78104000  CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrMo>; MSVBVM60.__vbaStrMove
00401C90    8B4D C8      MOV ECX,DWORD PTR SS:[EBP-38]
00401C93    50           PUSH EAX
……
```

`00401C50` `ADD AX,1` 改成 `add eax,1`

`00401C5A` `ADD AX,8` 改成 `add eax,8`

保存后运行，

变成这个样子了，没溢出提示了，但是 `text1` 中显示是`-32766`，这个值是错的，应该是 `32770` 才对。

`od` 重新载入 `00401C4A` 处下断，运行到 `00401C4A` 单步发现执行 `00401C7F` 后 `OD` 界面中出现了`-32766`。呵呵也对`&MSVBVM60.__vbaStrI2` 是把 `integer` 转成字符,还在认为是 `integer`。好的改成 `MSVBVM60.__vbaStrI4` 就应该可以了。在下边找到：

```asm
00401CE4   .  FF15 0C104000 CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrI4>]     ;  MSVBVM60.__vbaStrI4
```

在 `00401CE4` 处记下 `FF15 0C104000` 到 `00401C7F` 处改下保存。

运行看下

呵呵 `OK` 了。

总结下：

1. 以上只是改局部变量，全局变量还没去试。
2. `b1 as long` `Text2.Text = b1` 是刻意构造的，为的是能找到 `MSVBVM60.__vbaStrI4` 否则就要另外引入了。主要是偷个懒，呵呵。
3. 在实际的 `EXE` 去改难度会很大。都说 `vb` 是解释性的语言，谁知道微软在执行前怎样优化的。代码中一小点变化可能在汇编上也会变成另一种方式。

水平有限！高手别笑！

让我改变量的同事离开公司也有几年了，失去联系了，当时他叫我和他创业，我没胆没去，不知他现在混得怎样了。

作者：`QQ 746212099`

时间:`2010-7-10`

`test` 改变量.rar
