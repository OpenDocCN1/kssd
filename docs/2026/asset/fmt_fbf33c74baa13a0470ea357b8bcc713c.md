
# [原创]扫雷游戏代码分析及我的扫雷专家asm源码

【工具】 ollice, PEiD

【平台】 WinXP　SP2

【软件】 XP自带的扫雷游戏“`winmine.exe`”

【编程】 masm9.0+Radasm2.209

【简介】 分析游戏代码，编写出了地雷位置查看和自动快速扫雷功能的asm代码

　　前几天在看雪论坛看到有人把XP的“红心大战”游戏爽爽地玩了一通，我也凑了一阵热闹，觉得还不过瘾，就打算拿扫雷游戏来练练。这个程序相对较简单，适合新手练习。

## 一.游戏代码分析

　　用PEiD看一下为：Microsoft Visual C++ 7.0 Method2 [Debug]，无壳，直接用OD调试“`winmine.exe`”。

　　本游戏要用鼠标左、右键来操作，所以跟踪鼠标消息可以找到关键点，而跟踪鼠标消息的方法有许多，这里说说我的方法：

　　用OD载入，Ctrl+A分析代码后，右键菜单：查找－>所有分支，打开“已识别的分支”窗口

```
已识别的分支位于 winmine:.text

地址       反汇编                                    注释

010015AC   sub     eax, 53                           Switch (cases 53..111)　;里面有　WM\_INITDIALOG　消息

010015D5   dec     eax                               Switch (cases 1..6D)

01001700   sub     eax, 53                           Switch (cases 53..111)

0100182B   sub     eax, 110                          Switch (cases 110..111)

01001919   sub     eax, 0                            Switch (cases 0..1)

01001C05   dec     eax                               Switch (cases 2..47)

01001C9D   sub     eax, 10                           Switch (cases 10..75)

01001D5B   sub     eax, 111                          Switch (cases 111..113)

01001DE6   cmp     eax, 208                          Switch (cases 209..20F)

01001EDC   sub     eax, 211                          Switch (cases 211..251)

01001F5F   lea     eax, dword ptr ds:[edx-201]       Switch (cases 201..212)　;在这句上右键菜单：列出switch的case，又打开一个窗口，在里面找到 202(WM\_LBUTTONUP) 这一行，双击就来到下面，在下面这行设断点，F9运行，在游戏窗口点击鼠标左键就断在这里

01001FDF  |> \33FF          xor     edi, edi                                    ;  Cases 202 (WM\_LBUTTONUP),205 (WM\_RBUTTONUP),208 (WM\_MBUTTONUP) of switch 01001F5F

01001FE1  |.  393D 40510001 cmp     dword ptr ds:[1005140], edi

01001FE7  |.  0F84 BC010000 je      winmine.010021A9

01001FED  |>  893D 40510001 mov     dword ptr ds:[1005140], edi

01001FF3  |.  FF15 D8100001 call    near dword ptr ds:[<&USER32.ReleaseCapture>>; [ReleaseCapture

01001FF9  |.  841D 00500001 test    byte ptr ds:[<游戏是否开始的标志？>], bl

01001FFF  |.  0F84 B6000000 je      winmine.010020BB

01002005  |.  E8 D7170000   call    <winmine.处理你的左键点击>                  ;跟进去就好了^O^

0100200A  |.  E9 9A010000   jmp     winmine.010021A9
```

```
处理你的左键点击:

010037E1   /$  A1 18510001        mov     eax, dword ptr ds:[1005118]              ;  列号，比如你点在第３列则让EAX=３

010037E6   |.  85C0               test    eax, eax

010037E8   |.  0F8E C8000000      jle     winmine.010038B6                         ;  EAX<=0则跳

010037EE   |.  8B0D 1C510001      mov     ecx, dword ptr ds:[100511C]              ;  行号，比如你点在第９列则让EAX=９

010037F4   |.  85C9               test    ecx, ecx                                 ;  ECX<=0则跳

010037F6   |.  0F8E BA000000      jle     winmine.010038B6

010037FC   |.  3B05 34530001      cmp     eax, dword ptr ds:[<ColMax>]             ;  最大列数，比如９列

01003802   |.  0F8F AE000000      jg      winmine.010038B6                         ;  你点击的列号大于最大列数则跳

01003808   |.  3B0D 38530001      cmp     ecx, dword ptr ds:[<LineMax>]            ;  你点击的行号大于最大行数则跳

0100380E   |.  0F8F A2000000      jg      winmine.010038B6

01003814   |.  53                 push    ebx

01003815   |.  33DB               xor     ebx, ebx                                 ;  EBX=0

01003817   |.  43                 inc     ebx                                      ;  EBX=1

01003818   |.  833D A4570001 00   cmp     dword ptr ds:[10057A4], 0                ;  此处为已经探明的空格数

0100381F   |.  75 4A              jnz     short winmine.0100386B

01003821   |.  833D 9C570001 00   cmp     dword ptr ds:[100579C], 0                ;  游戏是否开始的标志

01003828   |.  75 41              jnz     short winmine.0100386B

0100382A   |.  53                 push    ebx

0100382B   |.  E8 BD000000        call    winmine.010038ED

01003830   |.  FF05 9C570001      inc     dword ptr ds:[100579C]

01003836   |.  E8 7AF0FFFF        call    winmine.010028B5                         ;  绘图相关调用

0100383B   |.  6A 00              push    0                                        ; /Timerproc = NULL

0100383D   |.  68 E8030000        push    3E8                                      ; |Timeout = 1000. ms

01003842   |.  53                 push    ebx                                      ; |TimerID

01003843   |.  FF35 245B0001      push    dword ptr ds:[1005B24]                   ; |hWnd = 0014014C ('扫雷',class='扫雷')

01003849   |.  891D 64510001      mov     dword ptr ds:[1005164], ebx              ; |TimerID

0100384F   |.  FF15 B4100001      call    near dword ptr ds:[<&USER32.SetTimer>]   ; \SetTimer

01003855   |.  85C0               test    eax, eax

01003857   |.  75 07              jnz     short winmine.01003860

01003859   |.  6A 04              push    4                                        ; /Arg1 = 00000004

0100385B   |.  E8 F0000000        call    winmine.01003950                         ; \winmine.01003950

01003860   |>  A1 18510001        mov     eax, dword ptr ds:[1005118]

01003865   |.  8B0D 1C510001      mov     ecx, dword ptr ds:[100511C]

0100386B   |>  841D 00500001      test    byte ptr ds:[<什么标志？>], bl

01003871   |.  5B                 pop     ebx

01003872   |.  75 10              jnz     short winmine.01003884

01003874   |.  6A FE              push    -2

01003876   |.  59                 pop     ecx

01003877   |.  8BC1               mov     eax, ecx

01003879   |.  890D 1C510001      mov     dword ptr ds:[100511C], ecx

0100387F   |.  A3 18510001        mov     dword ptr ds:[1005118], eax

01003884   |>  833D 44510001 00   cmp     dword ptr ds:[<雷数？>], 0

0100388B   |.  74 09              je      short winmine.01003896

0100388D   |.  51                 push    ecx

0100388E   |.  50                 push    eax

0100388F   |.  E8 23FDFFFF        call    winmine.010035B7

01003894   |.  EB 20              jmp     short winmine.010038B6

01003896   |>  8BD1               mov     edx, ecx                                 ;  行号

01003898   |.  C1E2 05            shl     edx, 5                                   ;  EDX\*=32

0100389B   |.  8A9402 40530001    mov     dl, byte ptr ds:[edx+eax+<byteITEM>]     ;  item[line\*32+col]

010038A2   |.  F6C2 40            test    dl, 40

010038A5   |.  75 0F              jnz     short winmine.010038B6

010038A7   |.  80E2 1F            and     dl, 1F

010038AA   |.  80FA 0E            cmp     dl, 0E

010038AD   |.  74 07              je      short winmine.010038B6

010038AF   |.  51                 push    ecx

010038B0   |.  50                 push    eax

010038B1   |.  E8 5CFCFFFF        call    winmine.01003512

010038B6   |>  FF35 60510001      push    dword ptr ds:[1005160]

010038BC   |.  E8 52F0FFFF        call    winmine.01002913

010038C1   \.  C3                 retn
```

　　同样用跟踪 `WM_INITDIALOG` 消息的方法可以找到游戏初始化的相关代码，具体方法同上，就不多说了。我这里就直接给出代码：

```
0100367A >/$  A1 AC560001   mov     eax, dword ptr ds:[<设置的最大宽度>]

0100367F  |.  8B0D A8560001 mov     ecx, dword ptr ds:[<设置的最大高度>]

01003685  |.  53            push    ebx

01003686  |.  56            push    esi

01003687  |.  57            push    edi

01003688  |.  33FF          xor     edi, edi

0100368A  |.  3B05 34530001 cmp     eax, dword ptr ds:[<ColMax>]

01003690  |.  893D 64510001 mov     dword ptr ds:[1005164], edi

01003696  |.  75 0C         jnz     short winmine.010036A4

01003698  |.  3B0D 38530001 cmp     ecx, dword ptr ds:[<LineMax>]

0100369E  |.  75 04         jnz     short winmine.010036A4

010036A0  |.  6A 04         push    4

010036A2  |.  EB 02         jmp     short winmine.010036A6

010036A4  |>  6A 06         push    6

010036A6  |>  5B            pop     ebx

010036A7  |.  A3 34530001   mov     dword ptr ds:[<ColMax>], eax

010036AC  |.  890D 38530001 mov     dword ptr ds:[<LineMax>], ecx

010036B2  |.  E8 1EF8FFFF   call    winmine.01002ED5

010036B7  |.  A1 A4560001   mov     eax, dword ptr ds:[10056A4]

010036BC  |.  893D 60510001 mov     dword ptr ds:[1005160], edi
```
