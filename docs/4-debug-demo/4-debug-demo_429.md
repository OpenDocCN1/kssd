
# 破解怪怪水族馆的带 pet 数限制

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-76.htm`](https://www.pediy.com/kssd/pediy07/pediy7-76.htm)

小弟初学破解，书是看了不少，可是始终没有什么真正的实践，最近春节放假在家 happy，发现一个小游戏 `Insaniquarium Deluxe`（幻想游戏中的怪怪水族馆）挺有意思，可是越玩到后面越手忙脚乱，这么多的 `pet` 每次却只能带 3 个，真是不爽，于是决定拿它开刀，小搞一下。

目的很明确：我要所有 `pet` 全上阵。目标明确了，开始行动。

掏出 `OllyDbg` 打开游戏的 `exe`，HOHO，没有壳，于是大致浏览了一下，开始寻找切入点，观察游戏中 `choose your pets` 这一步，发现当选择的 `pet` 数超过 3 个时候是不会有任何提示的，可就是选不上，但是，如果你选的 `pet` 数不够 3 个的话，它就会问你是不是要 `select more pets?` 嘿嘿，这个提示窗口就是切入点了，到 `OllyDbg` 中去搜它窗口中的关键字，找到这里：

```
00553DF8   41 72 65 20 79 6F 75 20   Are you
00553E00   73 75 72 65 20 79 6F 75    sure you
00553E08   20 77 61 6E 74 20 74 6F    want to
00553E10   20 63 6F 6E 74 69 6E 75    continu
00553E18   65 20 74 6F 20 74 68 65    e to the
00553E20   20 6E 65 78 74 20 6C 65     next le
00553E28   76 65 6C 20 77 69 74 68    vel with
00553E30   6F 75 74 20 73 65 6C 65    out sele
00553E38   63 74 69 6E 67 20 61 74    cting at
00553E40   20 6C 65 61 73 74 20 33     least 3
00553E48   20 70 65 74 73 3F 00 00     pets?..
```

然后在 `00553DF8` 处 `Find Reference`，找到 2 处，跟过去，来到这 2 处引用的地方：

```
0044E6C7   |. 2BC2             SUB EAX,EDX
0044E6C9   |. 50               PUSH EAX                     ; /Arg2
0044E6CA   |. 68 09835400      PUSH Insaniqu.00548309       ; |Arg1 = 00548309
0044E6CF   |. 8D8D 64FFFFFF    LEA ECX,DWORD PTR SS:[EBP-9C] ; |
0044E6D5   |. E8 86C8FCFF      CALL Insaniqu.0041AF60       ; \Insaniqu.0041AF60
0044E6DA   |. B8 F83D5500      MOV EAX,Insaniqu.00553DF8    ;  ASCII "Are you sure you want to continue to the next level without selecting at least 3 pets?"
0044E6DF   |. 897D B4          MOV DWORD PTR SS:[EBP-4C],EDI
0044E6E2   |. 895D FC          MOV DWORD PTR SS:[EBP-4],EBX
0044E6E5   |. 895D B0          MOV DWORD PTR SS:[EBP-50],EBX
0044E6E8   |. 885D A0          MOV BYTE PTR SS:[EBP-60],BL
0044E6EB   |. 8D78 01          LEA EDI,DWORD PTR DS:[EAX+1]
0044E6EE   |. 8BFF             MOV EDI,EDI
0044E6F0   |> 8A08            /MOV CL,BYTE PTR DS:[EAX]
0044E6F2   |. 40               |INC EAX
0044E6F3   |. 3ACB             |CMP CL,BL
0044E6F5   |. 75 F9            \JNZ SHORT Insaniqu.0044E6F0
0044E6F7   |. 2BC7             SUB EAX,EDI
0044E6F9   |. 50               PUSH EAX                     ; /Arg2
0044E6FA   |. 68 F83D5500      PUSH Insaniqu.00553DF8       ; |Arg1 = 00553DF8 ASCII "Are you sure you want to continue to the next level without selecting at least 3 pets?"
0044E6FF   |. 8D4D 9C          LEA ECX,DWORD PTR SS:[EBP-64] ; |
0044E702   |. E8 59C8FCFF      CALL Insaniqu.0041AF60       ; \Insaniqu.0041AF60
```

断点确认了一下，这块应该就是弹出那个窗口的代码，然后反向往回走，一直来到这里，在开始处下了个断点，发现在选择 `pet` 那里按下任何一个 `pet` 都会来到这里，猜想这里应该离我找的东西不远了，于是单步跟踪了一遍这段代码，找到了一些眉目。

```
0044E530  /. 55             PUSH EBP
0044E531  |. 8BEC           MOV EBP,ESP
0044E533  |. 6A FF          PUSH -1
0044E535  |. 68 36DA5300    PUSH Insaniqu.0053DA36          ; SE handler installation
0044E53A  |. 64:A1 00000000 MOV EAX,DWORD PTR FS:[0]
0044E540  |. 50             PUSH EAX
0044E541  |. 64:8925 000000>MOV DWORD PTR FS:[0],ESP
0044E548  |. 81EC AC000000  SUB ESP,0AC
0044E54E  |. 8B45 08        MOV EAX,DWORD PTR SS:[EBP+8]
0044E551  |. 53             PUSH EBX
0044E552  |. 56             PUSH ESI
0044E553  |. 33DB           XOR EBX,EBX
0044E555  |. 3BC3           CMP EAX,EBX
0044E557  |. 57             PUSH EDI
0044E558  |. 8BF1           MOV ESI,ECX
0044E55A  |. 7C 79          JL SHORT Insaniqu.0044E5D5
0044E55C  |. 83F8 18        CMP EAX,18                     ; 一共 24 个 pet,eax 保存的是当前点到的 pet 号
0044E55F  |. 7D 74          JGE SHORT Insaniqu.0044E5D5
0044E561  |. 8B4C86 14      MOV ECX,DWORD PTR DS:[ESI+EAX*4+14]
0044E565  |. 3899 00010000  CMP BYTE PTR DS:[ECX+100],BL     ; 这个 DS:[ECX+100]处保存的就是当前选择的 pet 是否被选中标志
0044E56B  |. 75 4F          JNZ SHORT Insaniqu.0044E5BC
```

点了一下 1 号 `pet`，查看 `DS:[ECX+100]` 找到这里 `04E86FA8`（所选 `pet` 的选中标志）：

```
04E86FA8   00 F0 AD BA 10 42 3F 00   .瓠?B?.
```

1 号 `pet` 当前的选中标志是 `00`，因为我当时已经选了 3 个了，再选这第 4 个就选不上了。

于是回到游戏调整了一下选择的 `pet`，让 1 号 `pet` 能够被选上，然后再点一下 1 号 `pet`，发现在进入 `0044E530` 之前 `04E86FA8` 这个 `byte` 已经被改成 `01` 了（被选中），于是判断这里还不是关键，应该是前面的某段程序来决定这个 `pet` 是否能被选中的，于是对 `04E86FA8` 这里下 `Memory, on access` 断点，看看是哪里修改了这个标志，然后在游戏中反复点选了几次 `pet`，最后找到了关键的一段代码：

```
00458350  /$ 55             PUSH EBP
00458351  |. 8BEC           MOV EBP,ESP
00458353  |. 8A45 08        MOV AL,BYTE PTR SS:[EBP+8]
00458356  |. 56             PUSH ESI
00458357  |. 8BF1           MOV ESI,ECX
00458359  |. 3A86 00010000  CMP AL,BYTE PTR DS:[ESI+100]
0045835F  |. 74 39          JE SHORT Insaniqu.0045839A
00458361  |. 8886 00010000  MOV BYTE PTR DS:[ESI+100],AL     ; 就是这里改变了 pet 的选中标志
00458367  |. 8D86 90000000  LEA EAX,DWORD PTR DS:[ESI+90]
0045836D  |. 50             PUSH EAX
0045836E  |. 8D8E B0000000  LEA ECX,DWORD PTR DS:[ESI+B0]
00458374  |. 51             PUSH ECX
00458375  |. E8 86FEFFFF    CALL Insaniqu.00458200
0045837A  |. 8D96 F0000000  LEA EDX,DWORD PTR DS:[ESI+F0]     ; |
00458380  |. 52             PUSH EDX                      ; |Arg2
00458381  |. 8D86 A0000000  LEA EAX,DWORD PTR DS:[ESI+A0]     ; |
00458387  |. 50             PUSH EAX                      ; |Arg1
00458388  |. E8 73FEFFFF    CALL Insaniqu.00458200        ; \Insaniqu.00458200
0045838D  |. 8B16           MOV EDX,DWORD PTR DS:[ESI]
0045838F  |. 83C4 10        ADD ESP,10
00458392  |. 8BCE           MOV ECX,ESI
00458394  |. FF92 B8000000  CALL DWORD PTR DS:[EDX+B8]
0045839A  |> 5E             POP ESI
0045839B  |. 5D             POP EBP
0045839C  \. C2 0400        RETN 4
```

这段就是调整 `pet` 选中标志的代码，随即反复试验了几次，只要是选上了某个 `pet`，或是取消选中的某个 `pet` 都会来到这里来调整 `pet` 的选中标志。这么说，调用这个过程的代码的附近应该就是判断是否能选中的地方，于是查看此段代码的被调用情况：

`Local calls from 0044D8BD, 00458572, 004585AE`

有 3 处，逐一跟踪查看，发现这第 3 处就是我要找的地方：

```
0045859C   |. 8B88 F4000000  MOV ECX,DWORD PTR DS:[EAX+F4]
004585A2   |. 3B88 00010000  CMP ECX,DWORD PTR DS:[EAX+100]   ; 这个 cmp 就是关键，DWORD PTR DS:[EAX+100] 这个就是保存的可选 pet 数(3)，而这个 ECX 保存的就是当前选择的 pet 数
004585A8   |. 7D 22          JGE SHORT Insaniqu.004585CC      ; 超过3个就跳走，否则就调用上面那段把所选 pet 的标志置 1，被选定或者 0，取消选定
004585AA   |. 6A 01          PUSH 1                     ; /Arg1 = 00000001
004585AC   |. 8BCE           MOV ECX,ESI                 ; |
004585AE   |. E8 9DFDFFFF    CALL Insaniqu.00458350        ; \Insaniqu.00458350，这里调用上面那段调整所选 pet 的选定标志
```

找到了关键点，于是下手修改，把这个 `DWORD PTR DS:[EAX+100]` 改大，改成 `18`，让可选 `pet` 数量增大到 24，再返回游戏，嘿嘿，这下爽了，所有 `pet` 全可以选了，目的达到了，任务完成。

小弟语文不好，写的不好，还望各位高手大虾指点。

