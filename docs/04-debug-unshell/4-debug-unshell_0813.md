
# hying 变形壳脱壳过程

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-583.htm`](https://www.pediy.com/kssd/pediy07/pediy7-583.htm)

hying 变形壳脱壳过程

日期：2005 年 2 月 17 日 脱壳人：csjwaman[DFCG]

【软件名称】：hying 变形壳加壳的记事本

【软件大小】： KB

【下载地址】：`http://bbs.pediy.com/showthread.php?s=&threadid=11184`

【脱壳声明】：初学 Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！

【操作系统】：`win2k`

【脱壳工具】：`OD` 等传统工具

【脱壳过程】：

附件中含加壳文件和本人脱壳的文件。加壳文件为龙族兄弟的作品。本人脱壳后有些问题，后经 `forgot`、`fly` 等大侠指点。终于完成。在此向他们表示感谢!现将脱壳过程整理出来也算是一种回报吧:)

### 一、找入口：

用 `OD` 载入，隐藏 `OD`，然后 `bp VirtualAlloc`，运行程序后断在：

```
77E64DB9 >  55            PUSH EBP///断在函数入口。
77E64DBA   8BEC           MOV EBP,ESP
77E64DBC   FF75 14        PUSH DWORD PTR SS:[EBP+14]
77E64DBF   FF75 10        PUSH DWORD PTR SS:[EBP+10]
77E64DC2   FF75 0C        PUSH DWORD PTR SS:[EBP+C]
77E64DC5   FF75 08        PUSH DWORD PTR SS:[EBP+8]
77E64DC8   6A FF          PUSH -1
```

观察堆栈数据：

```
0006FFB0   010102F3  指针到下一个 SEH 记录
0006FFB4   00000000  SE 句柄
0006FFB8   00004255  |Size = 4255 (16981.)///申请空间大小。
0006FFBC   00001000  |AllocationType = MEM_COMMIT
0006FFC0   00000004  \Protect = PAGE_READWRITE
0006FFC4   77E71AF6  返回到 KERNEL32.77E71AF6
```

`ALT+F9` 返回：

```
010102F3   /74 08          JE SHORT notepad.010102FD///返回到这里。此时 EAX=00280000
010102F5   |75 06          JNZ SHORT notepad.010102FD
010102F7   |9A 00E03D00 E8>CALL FAR 50E8:003DE000           ; 远距呼叫
010102FE    76 07          JBE SHORT notepad.01010307
01010300    77 05          JA SHORT notepad.01010307
01010302    E7 BB          OUT 0BB,EAX                      ; I/O 命令
```

返回时 `EAX=00280000`，可见程序申请了一个首地址为 `00280000`，大小为 `4255H` 的空间。后面解码时许多代码都往这里放，以抵抗 DUMP。现在往下找，一直来到：

```
0101052D    33C9          XOR ECX,ECX
0101052F    41            INC ECX
01010530    E8 EEFFFFFF   CALL notepad.01010523
01010535    13C9          ADC ECX,ECX
01010537    E8 E7FFFFFF   CALL notepad.01010523
0101053C  ^ 72 F2         JB SHORT notepad.01010530
0101053E    C3            RETN
0101053F    2B7C24 28     SUB EDI,DWORD PTR SS:[ESP+28]
01010543    897C24 1C     MOV DWORD PTR SS:[ESP+1C],EDI
01010547    61            POPAD
01010548    C2 0800       RETN 8////在这里下断。
0101054B    2800          SUB BYTE PTR DS:[EAX],AL
0101054D    0000          ADD BYTE PTR DS:[EAX],AL
0101054F    1000          ADC BYTE PTR DS:[EAX],AL
```

在 `01010548` 断下 2 次后，看到返回地址为 `002823D4`。此时解码结束。`F8` 返回：

```
002823D4    8B0C2B         MOV ECX,DWORD PTR DS:[EBX+EBP]////返回这里。
002823D7    56             PUSH ESI
002823D8    F3:A4          REP MOVS BYTE PTR ES:[EDI],BYTE PTR DS:[>
002823DA    5E             POP ESI
002823DB    53             PUSH EBX
002823DC    68 00800000    PUSH 8000
002823E1    6A 00          PUSH 0
002823E3    56             PUSH ESI
002823E4    FF95 A8090000  CALL NEAR DWORD PTR SS:[EBP+9A8]
002823EA    5B             POP EBX
002823EB    83C3 0C        ADD EBX,0C
```

一直往下找，来到：

```
00282551   /75 0A          JNZ SHORT 0028255D
00282553   |8B52 04        MOV EDX,DWORD PTR DS:[EDX+4]
00282556   |C742 50 001000>MOV DWORD PTR DS:[EDX+50],1000
0028255D   \89AD 17090000  MOV DWORD PTR SS:[EBP+917],EBP
00282563    8B85 87090000  MOV EAX,DWORD PTR SS:[EBP+987]
00282569    0385 73090000  ADD EAX,DWORD PTR SS:[EBP+973]
0028256F  - FFE0           JMP NEAR EAX ////找到这里。直接 F4 下来，程序停下时 EAX=01006420。这就是入口了。
00282571    50             PUSH EAX
00282572    8BC4           MOV EAX,ESP
00282574    60             PUSHAD
00282575    8BD8           MOV EBX,EAX
00282577    E8 04000000    CALL 00282580
0028257C    65:1C 28       SBB AL,28                         ; 多余的前缀
0028257F    005D 8B        ADD BYTE PTR SS:[EBP-75],BL
```

`F7` 来到：

```
01006420    55             PUSH EBP////入口。
01006421    8BEC           MOV EBP,ESP
01006423    6A FF          PUSH -1
01006425    68 88180001    PUSH notepad.01001888
0100642A    68 D0650001    PUSH notepad.010065D0
0100642F    64:A1 00000000 MOV EAX,DWORD PTR FS:[0]
01006435    50             PUSH EAX
01006436    64:8925 000000>MOV DWORD PTR FS:[0],ESP
0100643D    83C4 98        ADD ESP,-68
01006440    53             PUSH EBX
01006441    56             PUSH ESI
01006442    57             PUSH EDI
01006443    8965 E8        MOV DWORD PTR SS:[EBP-18],ESP
01006446    C745 FC 000000>MOV DWORD PTR SS:[EBP-4],0
0100644D    6A 02          PUSH 2
0100644F    90             NOP
01006450    E8 1CC127FF    CALL 00282571
01006455    83C4 04        ADD ESP,4
01006458    C705 38990001 F>MOV DWORD PTR DS:[1009938],-1
01006462    C705 3C990001 F>MOV DWORD PTR DS:[100993C],-1
0100646C    90             NOP
0100646D    E8 FFC027FF    CALL 00282571
01006472    8B0D 44880001  MOV ECX,DWORD PTR DS:[1008844]
01006478    8908           MOV DWORD PTR DS:[EAX],ECX
0100647A    90             NOP
0100647B    E8 F1C027FF    CALL 00282571
01006480    8B15 40880001  MOV EDX,DWORD PTR DS:[1008840]
01006486    8910           MOV DWORD PTR DS:[EAX],EDX
01006488    A1 54110001    MOV EAX,DWORD PTR DS:[1001154]
0100648D    8B08           MOV ECX,DWORD PTR DS:[EAX]
```

到入口后可以发现不少 `CALL 00282571` 之类的指令。从前面的跟踪可知 `00282571` 是壳中的地址。其实就是加密了 `IAT`。

### 二、解密 IAT：

进入 `01006450` 处的 `CALL 00282571`：

```
00282571    50             PUSH EAX
00282572    8BC4           MOV EAX,ESP
00282574    60             PUSHAD
00282575    8BD8           MOV EBX,EAX
00282577    E8 04000000    CALL 00282580
0028257C    65:1C 28       SBB AL,28                         ; 多余的前缀
0028257F    005D 8B        ADD BYTE PTR SS:[EBP-75],BL
00282582    6D             INS DWORD PTR ES:[EDI],DX         ; I/O 命令
00282583    008B 7B048BB5  ADD BYTE PTR DS:[EBX+B58B047B],CL
00282589    8309 00        OR DWORD PTR DS:[ECX],0
0028258C    0003           ADD BYTE PTR DS:[EBX],AL
0028258E    F5             CMC
0028258F    8B06           MOV EAX,DWORD PTR DS:[ESI]
00282591    33D2           XOR EDX,EDX
00282593    B9 02000000    MOV ECX,2
00282598    F7E1           MUL ECX
0028259A    D1E8           SHR EAX,1
0028259C    3BF8           CMP EDI,EAX
0028259E    75 0A          JNZ SHORT 002825AA
002825A0    0AD2           OR DL,DL
002825A2    75 04          JNZ SHORT 002825A8
002825A4    EB 09          JMP SHORT 002825AF
002825A6    EB 02          JMP SHORT 002825AA
002825A8    EB 10          JMP SHORT 002825BA
002825AA    83C6 08        ADD ESI,8
002825AD  ^ EB E0          JMP SHORT 0028258F
002825AF    8B46 04        MOV EAX,DWORD PTR DS:[ESI+4]
002825B2    8903           MOV DWORD PTR DS:[EBX],EAX
002825B4    61             POPAD
002825B5    58             POP EAX
002825B6    8B00           MOV EAX,DWORD PTR DS:[EAX]////F4 到这里。
002825B8    FFE0           JMP NEAR EAX////找到这里!
002825BA    8B46 04        MOV EAX,DWORD PTR DS:[ESI+4]
002825BD    8903           MOV DWORD PTR DS:[EBX],EAX
002825BF    61             POPAD
002825C0    58             POP EAX
002825C1    83C4 04        ADD ESP,4
002825C4    8B00           MOV EAX,DWORD PTR DS:[EAX]
```

`F4` 到 `002825B6` 后观察内存数据区：

```
DS:[01001160]=7800776N (msvcrt.__set_app_type)
EAX=01001160 (notepad.01001160), ASCII "nw"
```

看来 `01001160` 是函数地址。到 `01001160` 处看看：

```
01001158  C9 1E 00 78 D7 7F 00 78  ?.x?.x
01001160  6E 77 00 78 20 A0 03 78  nw.x ?x
01001168  53 7C 00 78 3E C0 00 78  S|.x>?x
01001170  DA 7C 00 78 6A BD 00 78  趞.xj?x
```

显然是函数地址。上下观察一下：

```
01001000  BC C7 6E 79 7D 2A 6D 79  记 ny}\*my
01001008  DB 1D 6D 79 8D 2B 6D 79  ?my?my
01001010  B9 24 6D 79 9C 2C 6D 79  ?my?my
```



### 1. IAT 内存区域分析

01001018  BF 18 6D 79 00 00 00 00   ?my....
。。。。。。
010012D8  9B ED AF 76 3F ED AF 76   涰痸?懑 v
010012E0  3C E8 B0 76 7C 55 B0 76   <璋 v|U 皏
010012E8  E2 16 AF 76 BD 5A B0 76   ?痸絑皏
010012F0  7A 98 B0 76 E8 DF AF 76   z 槹 v 柽痸
010012F8  00 00 00 00 00 00 00 00   ........

可知 `IAT` 从 `01001000` 到 `010012F7` 结束。大小为 `2F8`。

接下来要找到程序是如何加密 `IAT` 的。重装载入程序，隐藏 `OD`，在 `DUMP` 区 `01006450` 处的 4 个字节上下内存访问断点。`F9` 后程序断在：

```
002823D8    F3:A4           REP MOVS BYTE PTR ES:\[EDI\],BYTE PTR DS:\[ESI\]////断在这里。
002823DA    5E              POP ESI
002823DB    53              PUSH EBX
002823DC    68 00800000     PUSH 8000
002823E1    6A 00           PUSH 0
002823E3    56              PUSH ESI
002823E4    FF95 A8090000   CALL NEAR DWORD PTR SS:\[EBP+9A8\]
```

此时 `01006450` 处还未解码。继续 `F9`，直到断在：

```
00282515    25 FFFFFF7F     AND EAX,7FFFFFFF
0028251A    8BDE            MOV EBX,ESI
0028251C    2BD8            SUB EBX,EAX
0028251E    8958 FC         MOV DWORD PTR DS:\[EAX-4\],EBX////断在这里。
00282521    83C7 08         ADD EDI,8
00282524  ^ EB E7           JMP SHORT 0028250D
00282526    64:FF35 3000000>PUSH DWORD PTR FS:\[30\]
0028252D    58              POP EAX
0028252E    85C0            TEST EAX,EAX
00282530    78 0F           JS SHORT 00282541
00282532    8B40 0C         MOV EAX,DWORD PTR DS:\[EAX+C\]
00282535    8B40 0C         MOV EAX,DWORD PTR DS:\[EAX+C\]
```

断下时 `EBX=FF27C11C` `EAX=01006455` 显然程序正准备将加密后的数据移入该地址。看来这里就是关键处了！

再跟一下：

```
0028250D    8B07            MOV EAX,DWORD PTR DS:\[EDI\] ////取地址放入 EAX。此时 EDI=28370D。
0028250F    0BC0            OR EAX,EAX
00282511    75 02           JNZ SHORT 00282515
00282513    EB 11           JMP SHORT 00282526
00282515    25 FFFFFF7F     AND EAX,7FFFFFFF
0028251A    8BDE            MOV EBX,ESI
0028251C    2BD8            SUB EBX,EAX
0028251E    8958 FC         MOV DWORD PTR DS:\[EAX-4\],EBX
00282521    83C7 08         ADD EDI,8
00282524  ^ EB E7           JMP SHORT 0028250D////跳回继续。
00282526    64:FF35 3000000>PUSH DWORD PTR FS:\[30\]
```

### 2. 解密表分析

到 `DUMP` 区 `28370D` 处看看：

```
00283705  55 64 00 01 60 11 00 01  Ud.\`.
0028370D  80 65 00 01 68 11 00 01  €e.h.
00283715  DC 65 00 81 6C 11 00 01  躤.乴.
```

原来就是解密表，跳转地址一一对应，`[EDI+4]` 处就是正确函数地址指针。

整个解密表如下：

```
00283675  F0 30 00 81 D4 12 00 01  ?.佋.
0028367D  EA 30 00 81 D8 12 00 01  ?.佖.
00283685  E4 30 00 81 DC 12 00 01  ?.佨.
0028368D  3C 66 00 81 E0 12 00 01  <f.佮.
。。。。。。
0028422D  3C 57 00 01 BC 12 00 01  <W.?.
00284235  48 66 00 81 C4 12 00 01  Hf.伳.
0028423D  4E 66 00 81 C8 12 00 01  Nf.伻.
00284245  42 66 00 81 CC 12 00 01  Bf.佁.
```

### 3. 路径（PATH）处理

下面来 `PATH`：

重新载入程序，隐藏 `OD`，然后 `bp VirtualAlloc`，返回后直接 `F4` 到 `01010548`。

`CTRL+G` 来到：

```
00282515    25 FFFFFF7F      AND EAX,7FFFFFFF////从此处开始 PATH。
0028251A    8BDE             MOV EBX,ESI
0028251C    2BD8             SUB EBX,EAX
0028251E    8958 FC          MOV DWORD PTR DS:\[EAX-4\],EBX
00282521    83C7 08          ADD EDI,8
00282524  ^ EB E7           JMP SHORT 0028250D
00282526    64:FF35 30000000 PUSH DWORD PTR FS:\[30\]
```

修改为：

```
00282515   /E9 E6280000      JMP 00284E00
```

`Path` 代码：

```
00284E00    807F 03 01       CMP BYTE PTR DS:\[EDI+3\],1////比较最高位是否为 1。
00284E04    75 11            JNZ SHORT 00284E17
00284E06    8B5F 04          MOV EBX,DWORD PTR DS:\[EDI+4\]////正确的函数地址指针移入 EBX。
00284E09    66:C740 FA FF15 MOV WORD PTR DS:\[EAX-6\],15FF////加入指令 CALL\[XXXXXXXX\]
00284E0F    8958 FC          MOV DWORD PTR DS:\[EAX-4\],EBX
00284E12  ^ E9 0AD7FFFF      JMP 00282521////跳回继续。
00284E17    25 FFFFFF7F      AND EAX,7FFFFFFF////取消最高位。
00284E1C    8B5F 04          MOV EBX,DWORD PTR DS:\[EDI+4\]
00284E1F    66:C740 FA FF25 MOV WORD PTR DS:\[EAX-6\],25FF////加入指令 JMP\[XXXXXXXX\]
00284E25    8958 FC          MOV DWORD PTR DS:\[EAX-4\],EBX
00284E28  ^ E9 F4D6FFFF      JMP 00282521////跳回继续。
00284E2D    90               NOP
```

这个壳加密的 `IAT` 地址最高位有 `81` 和 `01` 两类。据 `forgot` 称如果是 `81` 则对应的要 `25FF`，如果是 `01` 则要 `15FF`。本人照 `forgot` 的指点做了，但还不知其所以然。请 `forgot` 能解释一下原因。

按上述方法 `PATH` 后就可以到入口 `DUMP` 了，然后用 `ImportREC` 修复就 `OK` 了。

### 4. 黑名单纯正与绕过

开始用 `OD` 加载后总是不能正常运行。后来发现，原来他在个黑名单，在运行时会检查 `"OllYDBG"`、`"UkillOD"`、`"Cnh☆DO"`，即原版 `OD` 与两种修改版的 `OD`。但开始我想用 `bp FindWindowA` 断点又不行，因为他应该是动态加载调用的这个 `API`，所以在原文件里没有这个 `API` 的调用，也就不能下这个断点。

但可以用 `he GetProcAddress+5` 做为第一个断点，之后边按 `F9` 边观察堆栈，直到出现对 `FindWindowA` 函数的加载后，按 `Ctrl+f9` 返回到主程序中后，就可以用 `bp FindWindowA`，做为断点来观察他的黑名单内容了。

这也给我们提了个醒，网上高手的修改版 `OD` 是好用，但很容易就进了黑名单了，所以还是自己修改的好用。

![img](img/1a550ccb7245757838a4f08fc220b787.jpg)

`4dd466d0e1e0cc123afbe504da2462ca`
