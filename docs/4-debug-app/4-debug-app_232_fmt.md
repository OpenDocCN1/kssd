
# 解密 windows 自带游戏"纸牌"的胜利效果

> 原文：[`www.pediy.com/kssd/pediy10/63051.html`](https://www.pediy.com/kssd/pediy10/63051.html)

## 解密 windows 自带游戏"纸牌"的胜利效果

还记得刚刚接触计算机的时候,最先玩的游戏就数"纸牌"了,不过感觉挺有难度的,现在也一直这么觉得.这是个非常经典的游戏,以至于到现在还有人会用它来打发时间,这也就是为什么我们的系统中会带有它的缘故了.我觉得"纸牌"的胜利效果做得很不错,可是难得一见啊(为什么?就是因为很难取得胜利,至少我没有胜利几次).现在有机会分析一下这个程序,看看这个"效果"隐藏在哪里,而后是"让我一次看个够".

但是不好入手啊,你先想一想应该从哪里开始?下什么断点?我分析的是这样的:触发胜利效果的条件是,当把最后一张牌移动到顶端区域时,胜利效果就会出现.也就是说当我们把最后一张牌拖放至顶端区域然后"松开鼠标"时,

目标区域接受并判断是合法牌且为"胜利状态",则调用相应的过程显示胜利效果.所以,我们可以下消息断点:`WM_LBUTTONUP`,目标窗体是主窗体.

下面开始.

### 1.获取窗体的回调函数地址

通常的做法是给 `RegisterClassExW`,`RegisterClassW` 这两个函数下断,通过堆栈提示得到回调函数的地址.

OD 载入,

```
bp RegisterClassW

bp RegisterClassExW
```

完了之后 F9 运行.

断下一个:

```
01001D3A  .  FF15 44110001   call    dword ptr [<&USER32.RegisterClas>; \RegisterClassExW
```

查看堆栈提示:

```
0007FE28   0007FE9C  \pWndClassEx = 0007FE9C
```

数据跟随 `0007FE9C`

```
0007FE9C  00000030
0007FEA0  00002008
0007FEA4  010016BD  sol.010016BD
0007FEA8  00000000
......
```

`010016BD` 就是回调过程.

F9 再运行,又断下一个:

```
01005F6E  .  FF15 C8100001   call    dword ptr [<&USER32.RegisterClas>; \RegisterClassW
```

查看堆栈提示:

```
0007FDF0   0007FDF8  \pWndClass = 0007FDF8
```

数据跟随 `0007FDF8`:

```
0007FDF8  00000000
0007FDFC  01005EBF  sol.01005EBF
0007FE00  00000000
......
```

`01005EBF` 就是回调过程.

### 2.在回调函数里查看处理代码,找到关键点.

首先转到:`010016BD` 查看处理函数代码,可以找到对 `202` 消息的处理.

```
01001A66  .  2D 02020000   sub     eax, 202                ;  Switch (cases 202..204)
01001A6B  .  0F84 AD000000 je      01001B1E
```

Enter 跟随到 `01001B1E`:

```
01001B1E  |> \FF15 FC100001 call    dword ptr [<&USER32.ReleaseCapture>]        ; [ReleaseCapture; Case 202 (WM_LBUTTONUP) of switch 01001A66
01001B24  .  6A 04          push    4
01001B26  .  59            pop     ecx
01001B27  \.^ E9 5CFDFFFF   jmp     01001888
```

就对 `01001B27` 下断,F9 运行,在游戏主界面单击一下,中断在 `01001B27` 处.

跟随至:

```
01001888  |> /A1 70710001   mov     eax, dword ptr [1007170]             ;  Case 200 (WM_MOUSEMOVE) of switch 01001866
0100188D  .  |3970 4C       cmp     dword ptr [eax+4C], esi
01001890  .  |74 1C        je      short 010018AE
01001892  |> |0FBF55 14    movsx   edx, word ptr [ebp+14]
01001896  .  |8955 F8      mov     dword ptr [ebp-8], edx
01001899  .  |8B55 14      mov     edx, dword ptr [ebp+14]
0100189C  .  |C1EA 10      shr     edx, 10
0100189F  .  |0FBFD2       movsx   edx, dx
010018A2  .  |8955 FC      mov     dword ptr [ebp-4], edx
010018A5  .  |56           push    esi
010018A6  .  |8D55 F8      lea     edx, dword ptr [ebp-8]
010018A9  .  |52           push    edx
010018AA  .  |51           push    ecx
010018AB  |> |50           push    eax
010018AC  .  .FF10         call    dword ptr [eax]                     ;  sol.010054A2
010018AE  |> |FF75 14      push    dword ptr [ebp+14]                  ; /lParam; Default case of switch 01001A8A
010018B1  .  |FF75 10      push    dword ptr [ebp+10]                  ; |wParam
010018B4  .  |57           push    edi                                   ; |Message
010018B5  .  |FF75 08      push    dword ptr [ebp+8]                     ; |hWnd
010018B8  .  |FF15 14110001 call    dword ptr [<&USER32.DefWindowProcW>]  ; \DefWindowProcW
010018BE  |> |5F           pop     edi
010018BF  .  |5E           pop     esi
010018C0  .  |C9           leave
010018C1  .  |C2 1000      retn    10
```

有个关键 call: `"call    dword ptr [eax]"`,跟进:

```
010054A2  /$  55            push    ebp
010054A3  .  .8BEC         mov     ebp, esp
010054A5  .  .8B4D 0C      mov     ecx, dword ptr [ebp+C]
010054A8  .  .83F9 0E      cmp     ecx, 0E                      ;  Switch (cases 3..12)
010054AB  .  .7F 7A        jg      short 01005527
010054AD  .  .74 68        je      short 01005517
010054AF  .  .8BC1         mov     eax, ecx
010054B1  .  .83E8 03      sub     eax, 3
010054B4  .  .74 54        je      short 0100550A
010054B6  .  .83E8 03      sub     eax, 3
010054B9  .  .74 36        je      short 010054F1
010054BB  .  .48           dec     eax
010054BC  .  .48           dec     eax
010054BD  .  .74 22        je      short 010054E1
010054BF  .  .83E8 04      sub     eax, 4
010054C2  .  .74 10        je      short 010054D4
010054C4  .  .48           dec     eax
010054C5  .  .75 70        jnz     short 01005537
010054C7  .  .FF75 08      push    dword ptr [ebp+8]             ;  Case D of switch 010054A8
010054CA  .  .E8 21F9FFFF   call    01004DF0
010054CF  .  .E9 A9000000   jmp     0100557D
010054D4  |>  .FF75 08      push    dword ptr [ebp+8]             ;  Case C of switch 010054A8
010054D7  .  .E8 42F8FFFF   call    01004D1E
010054DC  .  .E9 9C000000   jmp     0100557D
010054E1  |>  .FF75 10      push    dword ptr [ebp+10]            ; /Arg2; Case 8 of switch 010054A8
010054E4  .  .FF75 08      push    dword ptr [ebp+8]             ; |Arg1
010054E7  .  .E8 90F5FFFF   call    01004A7C                     ; \sol.01004A7C
010054EC  .  .E9 8C000000   jmp     0100557D
010054F1  |>  .FF75 14      push    dword ptr [ebp+14]            ; /Arg4; Case 6 of switch 010054A8
010054F4  .  .FF75 10      push    dword ptr [ebp+10]            ; |Arg3
010054F7  .  .6A 06        push    6                              ; |Arg2 = 00000006
010054F9  .  .FF75 08      push    dword ptr [ebp+8]             ; |Arg1
010054FC  .  .E8 F8DBFFFF   call    010030F9                     ; \sol.010030F9
01005501  .  .85C0         test    eax, eax
01005503  .  .74 05        je      short 0100550A
01005505  .  .33C0         xor     eax, eax
01005507  .  .40           inc     eax
01005508  .  .EB 73        jmp     short 0100557D
0100550A  |>  .FF75 10      push    dword ptr [ebp+10]            ;  Case 3 of switch 010054A8
0100550D  .  .FF75 08      push    dword ptr [ebp+8]
01005510  .  .E8 C8F6FFFF   call    01004BDD
01005515  .  .EB 66        jmp     short 0100557D
01005517  |>  .FF75 14      push    dword ptr [ebp+14]            ;  Case E of switch 010054A8
0100551A  .  .FF75 10      push    dword ptr [ebp+10]
0100551D  .  .FF75 08      push    dword ptr [ebp+8]
01005520  .  .E8 3AFBFFFF   call    0100505F
01005525  .  .EB 56        jmp     short 0100557D
01005527  |>  .8BC1         mov     eax, ecx
01005529  .  .83E8 0F      sub     eax, 0F
0100552C  .  .74 41        je      short 0100556F
0100552E  .  .48           dec     eax
0100552F  .  .74 31        je      short 01005562
01005531  .  .48           dec     eax
01005532  .  .74 1E        je      short 01005552
01005534  .  .48           dec     eax
01005535  .  .74 11        je      short 01005548
01005537  |>  .FF75 14      push    dword ptr [ebp+14]            ; /Arg4; Default case of switch 010054A8
0100553A  .  .FF75 10      push    dword ptr [ebp+10]            ; |Arg3
0100553D  .  .51           push    ecx                             ; |Arg2
0100553E  .  .FF75 08      push    dword ptr [ebp+8]             ; |Arg1
01005541  .  .E8 B3DBFFFF   call    010030F9                     ; \sol.010030F9
01005546  .  .EB 35        jmp     short 0100557D
01005548  |>  .FF75 08      push    dword ptr [ebp+8]             ;  Case 12 of switch 010054A8
0100554B  .  .E8 66FAFFFF   call    01004FB6
01005550  .  .EB 2B        jmp     short 0100557D
01005552  |>  .FF75 14      push    dword ptr [ebp+14]            ;  Case 11 of switch 010054A8
01005555  .  .FF75 10      push    dword ptr [ebp+10]
01005558  .  .FF75 08      push    dword ptr [ebp+8]
```



```
0100555B   |.  E8 29FCFFFF    call    01005189

01005560   |.  EB 1B          jmp     short 0100557D

01005562   |>  FF75 10       push    dword ptr [ebp+10]                        ;  Case 10 of switch 010054A8

01005565   |.  FF75 08       push    dword ptr [ebp+8]

01005568   |.  E8 96FCFFFF   call    01005203

0100556D   |.  EB 0E         jmp     short 0100557D

0100556F   |>  FF75 14       push    dword ptr [ebp+14]                        ; /Arg3;  Case F of switch 010054A8

01005572   |.  FF75 10       push    dword ptr [ebp+10]                        ; |Arg2

01005575   |.  FF75 08       push    dword ptr [ebp+8]                         ; |Arg1

01005578   |.  E8 42FBFFFF   call    010050BF                                  ; \\sol.010050BF

0100557D   |>  5D            pop     ebp

0100557E   \\.  C2 1000      retn    10
```

主要代码就在这里了,经过我分析发现

```
......

010054C5     /74 70          je      short 01005537

010054C7   |. |FF75 08       push    dword ptr [ebp+8]                         ;   Case D of switch 010054A8

010054CA   |. |E8 21F9FFFF   call    01004DF0

010054CF   |. |E9 A9000000   jmp     0100557D

......
```

这几句就是关键,跟进 `call    01004DF0` :

```
01004DF0   /$  55            push    ebp

01004DF1   |.  8D6C24 8C     lea     ebp, dword ptr [esp-74]

01004DF5   |.  81EC D8000000 sub     esp, 0D8

01004DFB   |.  A1 70710001   mov     eax, dword ptr [1007170]

01004E00   |.  53            push    ebx

01004E01   |.  56            push    esi

01004E02   |.  57            push    edi

01004E03   |.  33FF          xor     edi, edi

01004E05   |.  57            push    edi

01004E06   |.  6A 07         push    7

01004E08   |.  33F6          xor     esi, esi

01004E0A   |.  6A 0F         push    0F

01004E0C   |.  46            inc     esi

01004E0D   |.  50            push    eax

01004E0E   |.  8935 CC710001 mov     dword ptr [10071CC], esi

01004E14   |.  FF10          call    dword ptr [eax]

01004E16   |.  8B5D 7C       mov     ebx, dword ptr [ebp+7C]

01004E19   |.  897B 04       mov     dword ptr [ebx+4], edi

01004E1C   |.  897B 24       mov     dword ptr [ebx+24], edi

01004E1F   |.  8973 2C       mov     dword ptr [ebx+2C], esi

01004E22   |.  813D 28700001>cmp     dword ptr [1007028], 12E

01004E2C   |.  8945 70       mov     dword ptr [ebp+70], eax

01004E2F   |.  75 33         jnz     short 01004E64

01004E31   |.  6A 54         push    54

01004E33   |.  6A 6B         push    6B

01004E35   |.  8D45 9C       lea     eax, dword ptr [ebp-64]

01004E38   |.  50            push    eax

01004E39   |.  E8 BAD5FFFF   call    010023F8

01004E3E   |.  FF75 70       push    dword ptr [ebp+70]                         ; /Arg2

01004E41   |.  8BF0          mov     esi, eax                                    ; |

01004E43   |.  8D7C75 9C     lea     edi, dword ptr [ebp+esi*2-64]             ; |

01004E47   |.  57            push    edi                                        ; |Arg1

01004E48   |.  E8 DED4FFFF   call    0100232B                                   ; \\sol.0100232B

01004E4D   |.  8D0C47        lea     ecx, dword ptr [edi+eax*2]

01004E50   |.  66:C701 2000  mov     word ptr [ecx], 20

01004E55   |.  41            inc     ecx

01004E56   |.  41            inc     ecx

01004E57   |.  66:C701 2000  mov     word ptr [ecx], 20

01004E5C   |.  41            inc     ecx

01004E5D   |.  41            inc     ecx

01004E5E   |.  8D4406 02     lea     eax, dword ptr [esi+eax+2]

01004E62   |.  EB 05         jmp     short 01004E69

01004E64   |>  8D4D 9C       lea     ecx, dword ptr [ebp-64]

01004E67   |.  33C0          xor     eax, eax

01004E69   |>  83F8 54       cmp     eax, 54

01004E6C   |.  73 0E         jnb     short 01004E7C

01004E6E   |.  6A 54         push    54

01004E70   |.  5A            pop     edx

01004E71   |.  2BD0          sub     edx, eax

01004E73   |.  52            push    edx

01004E74   |.  6A 6A         push    6A

01004E76   |.  51            push    ecx

01004E77   |.  E8 7CD5FFFF   call    010023F8

01004E7C   |>  8D45 9C       lea     eax, dword ptr [ebp-64]

01004E7F   |.  50            push    eax                                        ; /Arg1

01004E80   |.  E8 680F0000   call    01005DED                                   ; \\sol.01005DED

01004E85   |.  E8 44C5FFFF   call    010013CE

01004E8A   |.  85C0          test    eax, eax

01004E8C   |.  0F84 FA000000 je      01004F8C

01004E92   |.  8D45 44       lea     eax, dword ptr [ebp+44]

01004E95   |.  50            push    eax                                        ; /pRect

01004E96   |.  FF35 3C730001 push    dword ptr [100733C]                       ; |hWnd = 00C30290 ('纸牌',class='Solitaire')

01004E9C   |.  FF15 24110001 call    dword ptr [<&USER32.GetClientRect>]       ; \\GetClientRect

01004EA2   |.  8B45 4C       mov     eax, dword ptr [ebp+4C]

01004EA5   |.  8945 5C       mov     dword ptr [ebp+5C], eax

01004EA8   |.  8B45 50       mov     eax, dword ptr [ebp+50]

01004EAB   |.  2B05 0C730001 sub     eax, dword ptr [100730C]

01004EB1   |.  C745 7C B4000>mov     dword ptr [ebp+7C], 0B4

01004EB8   |.  8945 60       mov     dword ptr [ebp+60], eax

01004EBB   |>  8D43 74       /lea     eax, dword ptr [ebx+74]

01004EBE   |.  C745 70 02000>|mov     dword ptr [ebp+70], 2

01004EC5   |.  8945 64       |mov     dword ptr [ebp+64], eax

01004EC8   |>  8B35 F8110001 |/mov     esi, dword ptr [<&msvcrt.rand>]         ;   msvcrt.rand

01004ECE   |.  FFD6          ||call    esi                                     ; [rand

01004ED0   |.  99            ||cdq

01004ED1   |.  6A 6E         ||push    6E

01004ED3   |.  59            ||pop     ecx

01004ED4   |.  F7F9          ||idiv    ecx

01004ED6   |.  8BC2          ||mov     eax, edx

01004ED8   |.  83E8 41       ||sub     eax, 41

01004EDB   |.  8945 54       ||mov     dword ptr [ebp+54], eax

01004EDE   |.  99            ||cdq

01004EDF   |.  33C2          ||xor     eax, edx

01004EE1   |.  2BC2          ||sub     eax, edx

01004EE3   |.  83F8 0F       ||cmp     eax, 0F

01004EE6   |.  7D 07         ||jge     short 01004EEF

01004EE8   |.  C745 54 ECFFF>||mov     dword ptr [ebp+54], -14

01004EEF   |>  FFD6          ||call    esi

01004EF1   |.  99            ||cdq

01004EF2   |.  6A 6E         ||push    6E

01004EF4   |.  59            ||pop     ecx

01004EF5   |.  F7F9          ||idiv    ecx

01004EF7   |.  8B45 64       ||mov     eax, dword ptr [ebp+64]

01004EFA   |.  8B30          ||mov     esi, dword ptr [eax]

01004EFC   |.  8BFA          ||mov     edi, edx

01004EFE   |.  83EF 4B       ||sub     edi, 4B

01004F01   |.  0375 7C       ||add     esi, dword ptr [ebp+7C]

01004F04   |.  8B46 04       ||mov     eax, dword ptr [esi+4]

01004F07   |.  8945 68       ||mov     dword ptr [ebp+68], eax

01004F0A   |.  8B4E 08       ||mov     ecx, dword ptr [esi+8]

01004F0D   |.  894D 6C       ||mov     dword ptr [ebp+6C], ecx

01004F10   |.  EB 4A         ||jmp     short 01004F5C

01004F12   |>  3B45 5C       ||/cmp     eax, dword ptr [ebp+5C]

01004F15   |.  7D 51         |||jge     short 01004F68

01004F17   |.  8D45 68       |||lea     eax, dword ptr [ebp+68]

01004F1A   |.  50            |||push    eax

01004F1B   |.  56            |||push    esi

01004F1C   |.  E8 61D1FFFF   |||call    01002082

01004F21   |.  8B45 54       |||mov     eax, dword ptr [ebp+54]

01004F24   |.  99            |||cdq

01004F25   |.  6A 0A         |||push    0A

01004F27   |.  59            |||pop     ecx

01004F28   |.  F7F9          |||idiv    ecx

01004F2A   |.  0145 68       |||add     dword ptr [ebp+68], eax

01004F2D   |.  8BC7          |||mov     eax, edi

01004F2F   |.  99            |||cdq

01004F30   |.  F7F9          |||idiv    ecx

01004F32   |.  83C7 03       |||add     edi, 3

01004F35   |.  0145 6C       |||add     dword ptr [ebp+6C], eax
```



```
01004F38  |.  8B45 6C       |||mov     eax, dword ptr [ebp+6C]
01004F3B  |.  3B45 60       |||cmp     eax, dword ptr [ebp+60]
01004F3E  |.  7E 10         |||jle     short 01004F50
01004F40  |.  85FF         |||test    edi, edi
01004F42  |.  7E 0C         |||jle     short 01004F50
01004F44  |.  8BC7         |||mov     eax, edi
01004F46  |.  F7D8         |||neg     eax
01004F48  |.  C1E0 03      |||shl     eax, 3
01004F4B  |.  99           |||cdq
01004F4C  |.  F7F9         |||idiv    ecx
01004F4E  |.  8BF8         |||mov     edi, eax
01004F50  |>  E8 EEFDFFFF  |||call    01004D43
01004F55  |.  85C0         |||test    eax, eax
01004F57  |.  75 2E        |||jnz     short 01004F87
01004F59  |.  8B45 68      |||mov     eax, dword ptr [ebp+68]
01004F5C  |>  8B0D 08730001|| mov     ecx, dword ptr [1007308]
01004F62  |.  F7D9         |||neg     ecx
01004F64  |.  3BC1         |||cmp     eax, ecx
01004F66  |.^ 7F AA       ||\jg      short 01004F12
01004F68  |>  FF45 70      ||inc     dword ptr [ebp+70]
01004F6B  |.  8345 64 04   ||add     dword ptr [ebp+64], 4
01004F6F  |.  837D 70 06   ||cmp     dword ptr [ebp+70], 6
01004F73  |.^ 0F8C 4FFFFFFF|\jl      01004EC8
01004F79  |.  836D 7C 0C   |sub      dword ptr [ebp+7C], 0C
01004F7D  |.  837D 7C 24   |cmp      dword ptr [ebp+7C], 24
01004F81  |.^ 0F8D 34FFFFFF\jge      01004EBB
01004F87  |>  E8 79C4FFFF  call    01001405
01004F8C  |>  68 FF1F0000  push    1FFF                             ; /Arg1 = 00001FFF
01004F91  |.  E8 F80E0000  call    01005E8E                         ; \sol.01005E8E
01004F96  |.  E8 0FD2FFFF  call    010021AA
01004F9B  |.  33C0         xor     eax, eax
01004F9D  |.  50           push    eax                             ; /Arg4 => 00000000
01004F9E  |.  50           push    eax                             ; |Arg3 => 00000000
01004F9F  |.  6A 0D        push    0D                              ; |Arg2 = 0000000D
01004FA1  |.  53           push    ebx                             ; |Arg1
01004FA2  |.  A3 CC710001  mov     dword ptr [10071CC], eax        ; |
01004FA7  |.  E8 4DE1FFFF  call    010030F9                         ; \sol.010030F9
01004FAC  |.  5F           pop     edi
01004FAD  |.  5E           pop     esi
01004FAE  |.  5B           pop     ebx
01004FAF  |.  83C5 74      add     ebp, 74
01004FB2  |.  C9           leave
01004FB3  \.  C2 0400      retn    4
```

关键代码就在这里了，我就不分析了（有点吃力哦），当然另外一个回调函数也不必看了。看下胜利截图：

![](img/5e35467dc12420e664476a884f4ba665.jpg)
