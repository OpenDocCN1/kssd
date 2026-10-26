
已经面目全非了，在OEP处，跟踪了一阵子就来到

呵呵,System@\_16605没有被搞到壳里面去，来来，我们一步一步走。

### 初步跟踪片段
```
0044DCBB    58              pop     eax
0044DCBC  ^ 76 F0           jbe     short 0044DCAE
0044DCBE    91              xchg    eax, ecx
0044DCBF    49              dec     ecx
0044DCC0    0C 78           or      al, 78
0044DCC2    1F              pop     ds
0044DCC3    3336            xor     esi, dword ptr [esi]
0044DCC5    51              push    ecx
0044DCC6    2C F1           sub     al, 0F1
0044DCC8    4C              dec     esp
0044DCC9  ^ 78 C4           js      short 0044DC8F
0044DCCB    02F1            add     dh, cl
0044DCCD    A5              movs    dword ptr es:[edi], dword ptr [e>
0044DCCE    DCCF            fmul    st(7), st
0044DCD0    25 0C476156     and     eax, 5661470C
0044DCD5    2E:1018         adc     byte ptr cs:[eax], bl
0044DCD8    D898 426F5582   fcomp   dword ptr [eax+82556F42]
0044DCDE    48              dec     eax
0044DCDF    EA 13AE7696 370>jmp     far 0C37:9676AE13
0044DCE6    64:CE           into
0044DCE8    4E              dec     esi
0044DCE9    E8 2133C7C7     call    C80C100F
0044DCEE  ^ 7C F9           jl      short 0044DCE9
0044DCF0    3B15 D8028A5C   cmp     edx, dword ptr [5C8A02D8]
0044DCF6    4D              dec     ebp
0044DCF7    67:49           dec     ecx
0044DCF9    B5 08           mov     ch, 8
0044DCFB    64:0283 35C82EC>add     al, byte ptr fs:[ebx+CD2EC835]
0044DD02  - E9 6D09BABD     jmp     BDFEE674
0044DD07    398A FE24C839   cmp     dword ptr [edx+39C824FE], ecx
0044DD0D    49              dec     ecx
0044DD0E    B6 C6           mov     dh, 0C6
0044DD10    98              cwde
0044DD11    F5              cmc
0044DD12    E7 6A           out     6A, eax
0044DD14    64:99           cdq
0044DD16    E4 D0           in      al, 0D0
0044DD18    E8 2AA10E17     call    17537E47
0044DD1D    DA3A            fidivr  dword ptr [edx]
0044DD1F    2E:34 E8        xor     al, 0E8
0044DD22    42              inc     edx
0044DD23    8B37            mov     esi, dword ptr [edi]
0044DD25    F7A2 263CB312   mul     dword ptr [edx+12B33C26]
0044DD2B    F3:             prefix rep:
0044DD2C    A3 DC603CB6     mov     dword ptr [B63C60DC], eax
0044DD31    EE              out     dx, al
0044DD32  - 0F89 F10BF3C6   jns     C737E929
0044DD38    52              push    edx
0044DD39    DB              ???                                      ; 未知命令
0044DD3A    E4 7B           in      al, 7B
0044DD3C    35 839D5AC0     xor     eax, C05A9D83
0044DD41    EA 707BD1F3 640>jmp     far 0564:F3D17B70
0044DD48    DC57 D1         fcom    qword ptr [edi-2F]
0044DD4B    1B66 45         sbb     esp, dword ptr [esi+45]
0044DD4E    B0 53           mov     al, 53
0044DD50    D211            rcl     byte ptr [ecx], cl
0044DD52    0F02C6          lar     eax, esi
0044DD55    6B7B 2D 63      imul    edi, dword ptr [ebx+2D], 63
0044DD59    014B EB         add     dword ptr [ebx-15], ecx
0044DD5C    FD              std
0044DD5D    8B0A            mov     ecx, dword ptr [edx]
0044DD5F    7F 34           jg      short 0044DD95
0044DD61    90              nop
0044DD62    857A 6C         test    dword ptr [edx+6C], edi
0044DD65    73 34           jnb     short 0044DD9B
```

### OEP 处代码
```
00403F04    55              push    ebp
00403F05    8BEC            mov     ebp, esp
00403F07    53              push    ebx
00403F08    56              push    esi
00403F09    57              push    edi
00403F0A    A1 38F64400     mov     eax, dword ptr [44F638]---->这个值跳转表开始的地方，记下来，用来修复用
00403F0F    85C0            test    eax, eax
00403F11    74 4B           je      short 00403F5E
00403F13    8B30            mov     esi, dword ptr [eax]---->这个值本来应该是0x2D，项目的个数，但是现在变成0x1了
00403F15    33DB            xor     ebx, ebx
00403F17    8B78 04         mov     edi, dword ptr [eax+4]
00403F1A    33D2            xor     edx, edx
00403F1C    55              push    ebp
00403F1D    68 4A3F4000     push    00403F4A
00403F22    64:FF32         push    dword ptr fs:[edx]
00403F25    64:8922         mov     dword ptr fs:[edx], esp
00403F28    3BF3            cmp     esi, ebx
00403F2A    7E 14           jle     short 00403F40
00403F2C    8B04DF          mov     eax, dword ptr [edi+ebx*8]
00403F2F    43              inc     ebx
00403F30    891D 3CF64400   mov     dword ptr [44F63C], ebx
00403F36    85C0            test    eax, eax
00403F38    74 02           je      short 00403F3C
00403F3A    FFD0            call    eax----------->这里进入壳里，初始化函数跳到壳中执行了。OK,F7
00403F3C    3BF3            cmp     esi, ebx
00403F3E  ^ 7F EC           jg      short 00403F2C
00403F40    33C0            xor     eax, eax
00403F42    5A              pop     edx
00403F43    59              pop     ecx
00403F44    59              pop     ecx
00403F45    64:8910         mov     dword ptr fs:[eax], edx
00403F48    EB 14           jmp     short 00403F5E
00403F4A  ^ E9 1DFAFFFF     jmp     0040396C
00403F4F    E8 50FFFFFF     call    00403EA4
00403F54    E8 27FDFFFF     call    00403C80
00403F59    E8 76FDFFFF     call    00403CD4
00403F5E    5F              pop     edi
00403F5F    5E              pop     esi
00403F60    5B              pop     ebx
00403F61    5D              pop     ebp
00403F62    C3              retn
```

F7进入壳里面来到，也许你的地址会不一样

```
00DEC608    53              push    ebx
00DEC609    B3 01           mov     bl, 1
00DEC60B    803D 0436DF00 0>cmp     byte ptr [DF3604], 0 --->这是个标志
00DEC612    74 1A           je      short 00DEC62E
00DEC614    A1 0038DF00     mov     eax, dword ptr [DF3800]
00DEC619    8B00            mov     eax, dword ptr [eax]
00DEC61B    50              push    eax       --->这个是ImageBase=0x400000
00DEC61C    A1 FC35DF00     mov     eax, dword ptr [DF35FC]
00DEC621    50              push    eax       --->Table1地址
00DEC622    A1 F835DF00     mov     eax, dword ptr [DF35F8]
00DEC627    50              push    eax              --->Table2地址
00DEC628    FF15 0036DF00   call    dword ptr [DF3600]  ->F7跟进
00DEC62E    8BC3            mov     eax, ebx
00DEC630    5B              pop     ebx
00DEC631    C3              retn
```

通过后来分析知道,Table1存放加密后的跳转表项的RVA值,Table2是一个效验的表，通过它和code代码，来得到正确的值，也就是说[DF3600]进入的call，不能下普通断点了，好在硬件断点可以用。

### 壳入口代码
```
01070000    52              push    edx
01070001    2BD1            sub     edx, ecx
01070003    8D9411 A08F4800 lea     edx, dword ptr [ecx+edx+488FA0]
0107000A    8D5424 13       lea     edx, dword ptr [esp+13]
0107000E    8D5402 ED       lea     edx, dword ptr [edx+eax-13]
01070012    2BD0            sub     edx, eax
01070014    57              push    edi
01070015    337C24 28       xor     edi, dword ptr [esp+28]
01070019    BF 82A14000     mov     edi, 40A182
0107001E    51              push    ecx
0107001F    81E9 367BAC10   sub     ecx, 10AC7B36
01070025    50              push    eax
01070026    F2:             prefix repne:
01070027    EB 01           jmp     short 0107002A
01070029  - E9 33442428     jmp     292B4461
0107002E    83C8 31         or      eax, 31
01070031    81E7 66F9F3E5   and     edi, E5F3F966
01070037    FF72 08         push    dword ptr [edx+8]
0107003A    BF F6CF4700     mov     edi, 47CFF6
0107003F    83EF D7         sub     edi, -29
01070042    5F              pop     edi
01070043    EB 01           jmp     short 01070046
01070045    698D 4C4B42FF 7>imul    ecx, dword ptr [ebp+FF424B4C], D9C10C72
```

一步一步F7

```
0107005B    8B07            mov     eax, dword ptr [edi]--->取Table1中每项值
0107005D    0BC0            or      eax, eax
0107005F    0F84 FA010000   je      0107025F
01070065   |81F8 8464D6AF   cmp     eax, AFD66484
0107006B   |0F84 DD010000   je      0107024E
```

### Table1 数据示例
我的`table1`地址是`1050000`

```
01050000  40 F6 9B EC 0F B3 2C EE 83 2C 8D F2 2B DE BD C2  @????,?+藿
01050010  06 D4 5F 42 54 3E C6 8A C6 24 E3 30 9C 77 8C DD  赃BT>?????
01050020  39 B9 BA EE 51 3D B7 97 E0 37 60 03 C2 45 75 5B  9购钛=??`屡u[
```
