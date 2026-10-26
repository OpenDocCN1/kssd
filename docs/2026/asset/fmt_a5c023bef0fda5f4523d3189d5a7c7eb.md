
# [原创]初次分析一NE软件Koohg

## 文章信息
【文章标题】: 静态分析NE文件

【文章作者】: P_O_K_E_R

【作者邮箱】: p_o_k_e_r@sohu.com

【软件名称】: Koohg V1.0.4

【下载地址】: 962K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3y4Z5K9h3&6W2M7$3g2U0K9r3g2U0K9$3g2J5M7#2)9J5k6i4k6W2k6$3q4J5k6o6u0Q4x3X3g2F1L8#2)9J5c8X3E0G2L8$3S2Y4K9h3&6K6N6q4)9J5k6h3g2^5k6b7`.

【保护方式】: 序列号

【编写语言】: C++

【使用工具】: W32Dasm

【操作平台】: WinXP

【软件介绍】: 以花样为主并可自制棋盘的类跳棋游戏

【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

--------------------------------------------------------------------------------

## 详细过程

用PEiD分析成“不是有效的PE文件”，但文件可执行；用OllyICE动态分析时推测它通过`ntvdm`解释执行，使用W32dasm打开后并未发现三十二位寄存器，并依文件时间判断它是为Win95以前版本所作的十六位程序，然后用eXeScope分析发现是NE文件。

结果是用W32dasm进行静态分析，用W32dasm打开应用程序后参考找不到所愿，细看程序终于找到，问题应该是分析器的参考有毛病或对象程序可能使用了特殊字符造成了影响，又或是程序是十六位的缘故。

注册对话框的是`DialogID_00F7`，注册按键是`ControlID:0001`。

用eXeScope找到对应注册对话框，其编号为`247.`，并记下其中重要控件：

`MC` `static` `437=1079.` 机器码
`UN` `edit` `435=1077.` 姓名编辑框
`SN` `edit` `436=1078.` 序列号输入框
`BT` `button` `001=0001.` 注册按键
`CC` `button` `002=0002.` 取消按键

通过对话框参考找到对话框 `F7` 中的 `433` 控件并很幸运地跟踪到其处理代码，向上稍一卷动便发现了：

```
:0002.1F2A 3D0100     cmp ax, 0001
:0002.1F2D 7503       jne 1F32
:0002.1F2F E9B3F4     jmp 13E5
:0002.1F32 3D0200     cmp ax, 0002
:0002.1F35 7503       jne 1F3A
:0002.1F37 E932FC     jmp 1B6C
```

插说一下，如果不幸运地没找到处理代码，可以通过 `Dialog_ID` 先找到对话框调用：

```
:0002.0F48 FF363629   push word ptr [2936]              ;模块句柄
:0002.0F4C B8F700     mov ax, 00F7                      ;
:0002.0F4F BA0000     mov dx, 0000                      ;
:0002.0F52 52         push dx                           ;
:0002.0F53 50         push ax                           ;对话框标识
:0002.0F54 B80000     mov ax, 0000                      ;
:0002.0F57 50         push ax                           ;所属窗口
:0002.0F58 B8FA0F     mov ax, 0FFA                      ;
:0002.0F5B BA7A0B     mov dx, SEG ADDR of Segment 0002  ;对话框过程
:0002.0F5E 52         push dx                           ;
:0002.0F5F 50         push ax                           ;
:0002.0F60 9AFFFF0000 call USER.DIALOGBOX               ;
```

再到过程中找到分析。

无置疑地跟到 `0002.13E5`。粗一看便见：

```
:0002.141C 9AD68CD613 call 0004.8CD6
:0002.1421 83C404     add sp, 0004
:0002.1424 3D0000     cmp ax, 0000
:0002.1427 7403       je 142C
:0002.1429 E91F00     jmp 144B
```

有理由相信 `0004.8CD6` 就是注册判断。但 `0002.13E5` 这一部分我们还是要细细从头分析的好。

之后会发现不是的。

得分析——

```
:0002.13E5 FF760E     push word ptr [bp+0E]     ;对话框句柄
:0002.13E8 B83604     mov ax, 0436              ;
:0002.13EB 50         push ax                   ;控件ID
:0002.13EC 8D46BC     lea ax, [bp-44]           ;
:0002.13EF 8CD2       mov dx, ss                ;
:0002.13F1 52         push dx                   ;
:0002.13F2 50         push ax                   ;缓冲区为SS: bp-0044=bp-68.
:0002.13F3 B81400     mov ax, 0014              ;
:0002.13F6 50         push ax                   ;序列号限长20.字节
:0002.13F7 9AFFFF0000 call USER.GETDLGITEMTEXT  ;
:0002.13FC FF760E     push word ptr [bp+0E]     ;对话框句柄
:0002.13FF B83504     mov ax, 0435              ;
:0002.1402 50         push ax                   ;控件ID
:0002.1403 8D8626FE   lea ax, [bp+FE26]         ;
:0002.1407 8CD2       mov dx, ss                ;
:0002.1409 52         push dx                   ;
:0002.140A 50         push ax                   ;缓冲区为SS: bp-01DA=bp-474.
:0002.140B B82800     mov ax, 0028              ;
:0002.140E 50         push ax                   ;用户名限长40.字节
:0002.140F 9AF8130000 call USER.GETDLGITEMTEXT  ;
:0002.1414 8D8626FE   lea ax, [bp+FE26]         ;
:0002.1418 8CD2       mov dx, ss                ;
:0002.141A 52         push dx                   ;
:0002.141B 50         push ax                   ;用户名缓冲区
:0002.141C 9AD68CD613 call 0004.8CD6            ;获取用户名长度
:0002.1421 83C404     add sp, 0004              ;平衡堆栈
:0002.1424 3D0000     cmp ax, 0000              ;
:0002.1427 7403       je 142C                   ;
:0002.1429 E91F00     jmp 144B                  ;
:0002.142C
... ... ... ...
:0002.1443 9A610F0000 call USER.DIALOGBOX       ;用户名为空的处理
:0002.1448 E9020B     jmp 1F4D                  ;出错重来
```

补充一下，分析 `0004.8CD6` 子程序时，轻易看出它是统计字符串长，为了了解它计算的是哪一个字符串，则要了解 `bp+6` 是何值，回溯查找不果，细想起 `CALL` 有压下一 `CS:IP` 的，于是便知 `bp+6` 正是堆栈中的用户名缓冲区地址。

```
:0004.8CD6 55         push bp
:0004.8CD7 8BEC       mov bp, sp
:0004.8CD9 8BD7       mov dx, di
:0004.8CDB C47E06     les di, [bp+06]
:0004.8CDE 33C0       xor ax, ax
:0004.8CE0 B9FFFF     mov cx, FFFF
:0004.8CE3 F2         repnz
:0004.8CE4 AE         scasb
:0004.8CE5 F7D1       not cx
:0004.8CE7 7501       jne 8CEA
:0004.8CE9 49         dec cx
:0004.8CEA 91         xchg cx
:0004.8CEB 8BFA       mov di, dx
:0004.8CED 5D         pop bp
:0004.8CEE CB         retf
```

这子过段计算字符串长，空串返回负一，非空则返回包括结尾的字节长（二三四……）。

回过头来继续分析：

```
:0002.144B B80100     mov ax, 0001
:0002.144E 50         push ax
:0002.144F B80100     mov ax, 0001
:0002.1452 BAD312     mov dx, SEG ADDR of Segment 0005
:0002.1455 52         push dx
:0002.1456 50         push ax
:0002.1457 8D46F8     lea ax, [bp-08]
:0002.145A 8CD2       mov dx, ss
:0002.145C 52         push dx
:0002.145D 50         push ax
:0002.145E 9AF08C1F14 call 0004.8CF0            ;字符串传送：0005.0001->ss:bp-08
:0002.1463 83C40A     add sp, 000A
```

插入该传送函数：

```
:0004.8CF0 55         push bp
:0004.8CF1 8BEC       mov bp, sp
:0004.8CF3 57         push di
:0004.8CF4 56         push si
:0004.8CF5 1E         push ds
:0004.8CF6 C47E06     les di, [bp+06]
:0004.8CF9 C5760A     lds si, [bp+0A]
:0004.8FC 8BDF       mov bx, di
:0004.8CFE 8B4E0E     mov cx, [bp+0E]
:0004.8D01 E30C       jcxz 8D0F
:0004.8D03 AC         lodsb
:0004.8D04 0AC0       or al , al
:0004.8D06 7403       je 8D0B
:0004.8D08 AA         stosb
:0004.8D09 E2F8       loop 8D03
:0004.8D0B 32C0       xor al , al
:0004.8D0D F3AA       rep stosb
:0004.8D0F 8BC3       mov ax, bx
:0004.8D11 8CC2       mov dx, es
:0004.8D13 1F         pop ds
:0004.8D14 5E         pop si
:0004.8D15 5F         pop di
:0004.8D16 8BE5       mov sp, bp
:0004.8D18 5D         pop bp
:0004.8D19 CB         retf
```

本程序段的功能是进行串传送。从左向右的参数为——零值不传送而非零值表传送及目标串尾零个数:byte；
源段址:word；源偏址:word；目的段址:word；目的偏址:word。返回值为`ax=目的偏址`；`dx=目的段址`。

回过头我们可以往下继续分析，然而 `0005.0001` 处有何值我们不知道，虽能猜测那一处应该是机器码之类的，但还是调查一番比较好。

`DSEG005`  `File Offset: 00000000`  `Size:0000`  `Flags:0x0C51 -> DATA, MOVEABLE`

现有三个线索，一是利用查找找到对 `DSEG005` 调用的地方；二是查看注册对话框的初始化部分；三是机码文本的消息处理。第三者可能性极小。

按第一个查找会发现调用它的地方极多，它可能就一临时数据区，所以希望寄在第二可能。虽然过程的开头有些怪，但是很快就找到了 `0002.1027`，往下略一滚动很幸运就发现：

```
:0002.1063 BAE00E     mov dx, SEG ADDR of Segment 0005
```

但这一段看得眼花。天无绝人之路，于是搜索 `3704`，结果只有三处，很快就判断出 `0002.133C` 是所需，往它上面一看，有四个地方调用：

`:0002.10F2(U), :0002.1127(U), :0002.1240(U), :0002.1323(U)`

有够麻烦，但看第一个离 `0002.1027` 最近，去看一看，事先记下了 `SS:bp-30` 是机器码所在。

虽然很麻烦，但我们还是要把 `0002.1027` 至 `0002.10F2` 之间的部分细细分析一下。

```
:0002.1027 FF760E     push word ptr [bp+0E]
:0002.102A B82E04     mov ax, 042E
:0002.102D 50         push ax
:0002.102E 9AFFFF0000 call USER.GETDLGITEM
;获取注册对话框上的图标句柄并存一下
:0002.1033 A32E28     mov word ptr [282E], ax
;获取注册对话框上的图标句柄并存一下
:0002.1036 B83028     mov ax, 2830
:0002.1039 8CDA       mov dx, ds
:0002.103B 52         push dx
:0002.103C 50         push ax                   ;缓冲区为DS:2830
:0002.103D B80401     mov ax, 0104
:0002.1040 50         push ax                   ;缓冲区限长104=260.
:0002.1041 9A0B0C0000 call KERNEL.GETWINDOWSDIRECTORY
;查找操作系统目录，我返回的是"C:\WINDOWS"。
:0002.1046 B82B09     mov ax, 092B
; Possible StringData Ref from Data Seg 007 ->"\wingk.dll"
:0002.1049 8CDA       mov dx, ds
:0002.104B 52         push dx
:0002.104C 50         push ax
:0002.104D B83028     mov ax, 2830
:0002.1050 8CDA       mov dx, ds
:0002.1052 52         push dx
:0002.1053 50         push ax
:0002.1054 9A1C8C1F10 call 0004.8C1C
:0002.1059 83C408     add sp, 0008
```
