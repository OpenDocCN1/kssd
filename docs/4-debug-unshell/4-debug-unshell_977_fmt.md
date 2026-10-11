
# SDprotector v1.1 加壳的 MD5 计算器脱壳过程

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-600.htm`](https://www.pediy.com/kssd/pediy07/pediy7-600.htm)

SDprotector v1.1 加壳的 MD5 计算器脱壳过程

日期：2005 年 2 月 22 日 | 脱壳人：`csjwaman[DFCG]`

【软件名称】：`MD5 计算器` | 软件版本：

【下载地址】：附件:`SDP.rar`

【软件限制】：`SDprotector v1.1` 加壳

【破解声明】：初学 `Crack`，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！

【操作系统】：`WIN2K`, `WINXP`

【脱壳工具】：`OD` 等传统工具

【脱壳过程】：

`SDprotector v1.1` 壳，`softworm` 大侠曾写过有关文章。本人认真拜读过。本文就是在 `softworm` 大侠文章指引下完成的。在此谢过 `softworm` 大侠 :)

## 一、查找入口和 stolen code

`OD` 载入，隐藏 `OD`，忽略除 `INT3` 中断外的所有异常。运行程序，异常在：

```
004510B8    90               NOP////异常在这。
004510B9    3C 04            CMP AL,4
004510BB    74 71            JE SHORT Pack.0045112E
004510BD    74 03            JE SHORT Pack.004510C2
004510BF    75 01            JNZ SHORT Pack.004510C2
004510C1    E8 E8010000      CALL Pack.004512AE
004510C6    00FF             ADD BH,BH
004510C8    58               POP EAX
```

`shift+F9` 两次，断在：

```
0044C950    90               NOP////异常在这。
0044C951    33C0             XOR EAX,EAX
0044C953    C700 531A4000    MOV DWORD PTR DS:[EAX],Pack.004>
0044C959    7A 0A            JPE SHORT Pack.0044C965
0044C95B    7B 08            JPO SHORT Pack.0044C965
0044C95D    0010             ADD BYTE PTR DS:[EAX],DL
0044C95F    40               INC EAX
0044C960    007B 3D          ADD BYTE PTR DS:[EBX+3D],BH
0044C963    40               INC EAX
0044C964    00E8             ADD AL,CH
0044C966    0100             ADD DWORD PTR DS:[EAX],EAX
0044C968    0000             ADD BYTE PTR DS:[EAX],AL
0044C96A    FF58 05          CALL FAR FWORD PTR DS:[EAX+5]   ; 远距呼叫
0044C96D    1100             ADC DWORD PTR DS:[EAX],EAX
0044C96F    0000             ADD BYTE PTR DS:[EAX],AL
0044C971    50               PUSH EAX
```

异常后堆栈：

```
0012FFC0   0012FFE0  指针到下一个 SEH 记录
0012FFC4   0044CA1A  SE 句柄
0012FFC8   000000AE
0012FFCC   00000008
```

在 `44CA1A` 处 `F2` 下断，`F9` 运行，程序停在：

```
0044CA1A    8B6424 08       MOV ESP,DWORD PTR SS:[ESP+8]////中断在这里。
0044CA1E    64:8F05 0000000>POP DWORD PTR FS:[0]
0044CA25    E8 01000000     CALL Pack.0044CA2B
0044CA2A    FF58 05         CALL FAR FWORD PTR DS:[EAX+5]   ; 远距呼叫
0044CA2D    1100             ADC DWORD PTR DS:[EAX],EAX
0044CA2F    0000             ADD BYTE PTR DS:[EAX],AL
0044CA31    50               PUSH EAX
```

取消断点后，`F7` 跟踪，来到：

```
00454FE3    58               POP EAX         ; Pack.00454FE2
00454FE4    05 EC070000     ADD EAX,7EC
00454FE9    8B00             MOV EAX,DWORD PTR DS:[EAX]////现在[EAX]中是 GetTickCount 函数地址。
00454FEB    8038 CC          CMP BYTE PTR DS:[EAX],0CC/////检测 API 断点。象这样的检测很多。
00454FEE    74 22            JE SHORT Pack.00455012
00454FF0    8078 01 CC       CMP BYTE PTR DS:[EAX+1],0CC
00454FF4    74 1C            JE SHORT Pack.00455012
00454FF6    8078 02 CC       CMP BYTE PTR DS:[EAX+2],0CC
00454FFA    74 16            JE SHORT Pack.00455012
00454FFC    8078 03 CC       CMP BYTE PTR DS:[EAX+3],0CC
00455000    74 10            JE SHORT Pack.00455012
00455002    8078 04 CC       CMP BYTE PTR DS:[EAX+4],0CC
00455006    74 0A            JE SHORT Pack.00455012
00455008    50               PUSH EAX
00455009    C3               RETN////如果没有下断则返回。
```

现在检测的是 `GetTickCount` 函数是否被下断。没有被下断，返回：

```
0044CA46    FFFF             ???                         ; 未知命令
0044CA48    8B0A             MOV ECX,DWORD PTR DS:[EDX]
0044CA4A    83F9 00          CMP ECX,0
0044CA4D   ^ 0F84 57FFFFFF  JE Pack.0044C9AA/////时间检测，不能跳！现在不跳。
0044CA53    2BC1             SUB EAX,ECX
0044CA55   ^ 0F88 4FFFFFFF  JS Pack.0044C9AA/////时间检测，不能跳！现在不跳。
0044CA5B    2D D0070000      SUB EAX,7D0
0044CA60   ^ 0F89 44FFFFFF  JNS Pack.0044C9AA/////时间检测，不能跳！跳了，要修改 SF 标志为 1，不让跳。
0044CA66    B8 7612800E      MOV EAX,0E801276
0044CA6B    8902             MOV DWORD PTR DS:[EDX],EAX
0044CA6D    E8 01000000      CALL Pack.0044CA73
0044CA72    FF5A 81          CALL FAR FWORD PTR DS:[EDX-7F]  ; 远距呼叫
0044CA75    C2 0F03          RETN 30F
0044CA78    0000             ADD BYTE PTR DS:[EAX],AL
0044CA7A    E8 01000000      CALL Pack.0044CA80
0044CA7F    FF58 05          CALL FAR FWORD PTR DS:[EAX+5]   ; 远距呼叫
```

修改 `SF` 标志后用 `F7` 跟踪，来到：

```
00455C73    53               PUSH EBX///跟踪到这里。
00455C74    56               PUSH ESI
00455C75    57               PUSH EDI
00455C76    8BD8             MOV EBX,EAX
00455C78    8B7424 0C        MOV ESI,DWORD PTR SS:[ESP+C]
00455C7C    51               PUSH ECX
00455C7D    8B0C24           MOV ECX,DWORD PTR SS:[ESP]
00455C80    33FF             XOR EDI,EDI
00455C82    3BF2             CMP ESI,EDX
00455C84    73 0B            JNB SHORT Pack.00455C91
00455C86    8A041F           MOV AL,BYTE PTR DS:[EDI+EBX]
00455C89    3006             XOR BYTE PTR DS:[ESI],AL
00455C8B    46               INC ESI
00455C8C    47               INC EDI
00455C8D   ^ E2 F3           LOOPD SHORT Pack.00455C82
00455C8F   ^ EB EC          JMP SHORT Pack.00455C7D
00455C91    E8 01000000      CALL Pack.00455C97////F4 到这里。
00455C96    FF58 05          CALL FAR FWORD PTR DS:[EAX+5]   ; 远距呼叫
00455C99    0D 00000050      OR EAX,50000000
00455C9E    C3               RETN
```

然后 `F7` 来到：

```
00455C97    58               POP EAX                   ; Pack.00455C96
00455C98    05 0D000000      ADD EAX,0D
00455C9D    50               PUSH EAX
00455C9E    C3               RETN
00455C9F    EF               OUT DX,EAX                  ; I/O 命令
00455CA0    49               DEC ECX
00455CA1    2907             SUB DWORD PTR DS:[EDI],EAX
00455CA3    59               POP ECX
00455CA4    5F               POP EDI
00455CA5    5E               POP ESI
00455CA6    5B               POP EBX
00455CA7    C3               RETN
```

一直 `F7` 来到：

```
00451EB4    8850 02          MOV BYTE PTR DS:[EAX+2],DL
00451EB7    83C0 04          ADD EAX,4
00451EBA    8D1430           LEA EDX,DWORD PTR DS:[EAX+ESI]
00451EBD    3BD7             CMP EDX,EDI
00451EBF   ^ 72 D4          JB SHORT Pack.00451E95
00451EC1    5E               POP ESI
00451EC2    5F               POP EDI
00451EC3    C3               RETN////返回。
00451779    C1E8 03          SHR EAX,3
0045177C    83E0 3F          AND EAX,3F
0045177F    B9 38000000      MOV ECX,38
00451784    83F8 38          CMP EAX,38
00451787    72 05            JB SHORT Pack.0045178E
00451789    B9 78000000      MOV ECX,78
0045178E    2BC8             SUB ECX,EAX
00451790    51               PUSH ECX
00451791    68 D4624000      PUSH Pack.004062D4
00451796    E8 E8F6FFFF      CALL Pack.00450E83
0045179B    50               PUSH EAX
0045179C    57               PUSH EDI
0045179D    E8 13FFFFFF      CALL Pack.004516B5
004517A2    8D4C24 14        LEA ECX,DWORD PTR SS:[ESP+14]
004517A6    6A 08            PUSH 8
004517A8    51               PUSH ECX
004517A9    57               PUSH EDI
004517AA    E8 06FFFFFF      CALL Pack.004516B5
004517AF    8B5424 2C        MOV EDX,DWORD PTR SS:[ESP+2C]
004517B3    6A 10            PUSH 10
004517B5    57               PUSH EDI
004517B6    52               PUSH EDX
004517B7    E8 BF060000      CALL Pack.00451E7B
004517BC    6A 58            PUSH 58
004517BE    6A 00            PUSH 0
004517C0    57               PUSH EDI
004517C1    E8 1C070000      CALL Pack.00451EE2
004517C6    83C4 30          ADD ESP,30
004517C9    5F               POP EDI
004517CA    5E               POP ESI
004517CB    83C4 08          ADD ESP,8
004517CE    C3               RETN///返回。
00452FBB    83C4 24          ADD ESP,24
00452FBE    5F               POP EDI
00452FBF    5E               POP ESI
00452FC0    83C4 58          ADD ESP,58
00452FC3    C2 0400          RETN 4///返回。
```

中间省略跟踪过程。来到：



### 汇编代码分析：00453F60 至 00454FDD

```assembly
00453F60    E5 0D             IN EAX,0D                               ; I/O 命令
00453F62    0000              ADD BYTE PTR DS:[EAX],AL
00453F64    3D 00000080       CMP EAX,80000000
00453F69    73 63             JNB SHORT Pack.00453FCE               ; 这里要跳！修改 CF 标志，让跳。
00453F6B    56                PUSH ESI
00453F6C    57                PUSH EDI
00453F6D    E8 6B100000       CALL Pack.00454FDD
00453F72    68 3A8F4000       PUSH Pack.00408F3A
00453F77    8BF8              MOV EDI,EAX
00453F79    E8 05CFFFFF       CALL Pack.00450E83
00453F7E    50                PUSH EAX
00453F7F    68 548F4000       PUSH Pack.00408F54
00453F84    E8 FACEFFFF       CALL Pack.00450E83
00453F89    50                PUSH EAX
00453F8A    E8 C6100000       CALL Pack.00455055
```

在 `00453F69` 处，重新跟踪，`SHIFT+F9` 直接通过异常。由于不是单步跟踪，所以检测时间会自行通过。当断在 `00453F69` 时，修改 `CF` 标志，让跳。

接着 `CreateProcessA`，然后 `F9`。由于程序检测 `INT3` 断点，所以要下硬件断点。

```assembly
77E41BBC > 55                PUSH EBP                                ; 断下。
77E41BBD  8BEC                MOV EBP,ESP
77E41BBF  6A 00               PUSH 0
77E41BC1  FF75 2C             PUSH DWORD PTR SS:[EBP+2C]
77E41BC4  FF75 28             PUSH DWORD PTR SS:[EBP+28]
77E41BC7  FF75 24             PUSH DWORD PTR SS:[EBP+24]
77E41BCA  FF75 20             PUSH DWORD PTR SS:[EBP+20]
77E41BCD  FF75 1C             PUSH DWORD PTR SS:[EBP+1C]
77E41BD0  FF75 18             PUSH DWORD PTR SS:[EBP+18]
77E41BD3  FF75 14             PUSH DWORD PTR SS:[EBP+14]
77E41BD6  FF75 10             PUSH DWORD PTR SS:[EBP+10]
77E41BD9  FF75 0C             PUSH DWORD PTR SS:[EBP+C]
```

堆栈区：

```text
0012FF98  0044CD69  /CALL 到 CreateProcessA      ; 44CD69
0012FF9C  00000000  |ModuleFileName = NULL
0012FFA0  0044C224  |CommandLine = ""C:\Documents and Settings\csjwaman\桌面\SDP\Pack.exe""
0012FFA4  00000000  |pProcessSecurity = NULL
0012FFA8  00000000  |pThreadSecurity = NULL
0012FFAC  00000001  |InheritHandles = TRUE
0012FFB0  00000000  |CreationFlags = 0
0012FFB4  00000000  |pEnvironment = NULL
0012FFB8  00000000  |CurrentDir = NULL
0012FFBC  0044C328  |pStartupInfo = Pack.0044C328
0012FFC0  0044C214  \pProcessInfo = Pack.0044C214
```

```assembly
0044CD69  6A 00               PUSH 0
0044CD6B  E8 01000000         CALL Pack.0044CD71
0044CD70  FF58 05             CALL FAR FWORD PTR DS:[EAX+5]          ; 远距呼叫
0044CD73  1100                ADC DWORD PTR DS:[EAX],EAX
0044CD75  0000                ADD BYTE PTR DS:[EAX],AL
0044CD77  50                  PUSH EAX
0044CD78  E9 9C820000         JMP Pack.00455019
0044CD7D  0010                ADD BYTE PTR DS:[EAX],DL
```

往上找，可以找到 `JE Pack.0044CD81`，这个跳转可以跳过创建进程。

```assembly
0044CCD8   /75 07            JNZ SHORT Pack.0044CCE1
0044CCDA   |74 05            JE SHORT Pack.0044CCE1
0044CCDC   |0010              ADD BYTE PTR DS:[EAX],DL
0044CCDE   |40                INC EAX
0044CCDF   |00E8              ADD AL,CH
0044CCE1   \0F84 9A000000     JE Pack.0044CD81                      ; 跳转则成单进程!
0044CCE7    E8 01000000        CALL Pack.0044CCED
0044CCEC    FF58 05            CALL FAR FWORD PTR DS:[EAX+5]         ; 远距呼叫
0044CCEF    1100               ADC DWORD PTR DS:[EAX],EAX
0044CCF1    0000               ADD BYTE PTR DS:[EAX],AL
0044CCF3    50                 PUSH EAX
0044CCF4    E9 D4830000        JMP Pack.004550CD
```

在 `44CCE1` 处，然后重新跟踪，`SHIFT+F9` 直接通过异常，再修改 `00453F69` 处的跳转，就会断在 `44CCE1`，然后修改标志位，让跳。

`F9` 运行，会遇到 `INT3` 中断异常，`SHIFT+F9` 通过，直到停在：

```assembly
00450D0A  90                  NOP                                   ; 在这里。
00450D0B  33C0                XOR EAX,EAX
00450D0D  C700 0D5E4000       MOV DWORD PTR DS:[EAX],Pack.00405E0D
00450D13  7A 0A               JPE SHORT Pack.00450D1F
00450D15  7B 08               JPO SHORT Pack.00450D1F
00450D17  0010                ADD BYTE PTR DS:[EAX],DL
00450D19  40                  INC EAX
00450D1A  007B 3D             ADD BYTE PTR DS:[EBX+3D],BH
```

堆栈区：

```text
0012EE10  0012FFE0  指针到下一个 SEH 记录
0012EE14  00450DD4  SE 句柄                                      ; 到 SE 句柄下断。
```

`F9` 运行，程序停在：

```assembly
00450DD4  8B6424 08           MOV ESP,DWORD PTR SS:[ESP+8]        ; 断在这里。
00450DD8  64:8F05 00000000    POP DWORD PTR FS:[0]
00450DDF  E8 01000000         CALL Pack.00450DE5
00450DE4  FF58 05             CALL FAR FWORD PTR DS:[EAX+5]        ; 远距呼叫
00450DE7  1100                ADC DWORD PTR DS:[EAX],EAX
00450DE9  0000                ADD BYTE PTR DS:[EAX],AL
00450DEB  50                  PUSH EAX
```

取消断点后，`F7` 跟踪。来到：

```assembly
00454FE3  58                  POP EAX
00454FE4  05 EC070000         ADD EAX,7EC
00454FE9  8B00                MOV EAX,DWORD PTR DS:[EAX]           ; 又开始检测 API 断点了。
00454FEB  8038 CC             CMP BYTE PTR DS:[EAX],0CC
00454FEE  74 22               JE SHORT Pack.00455012
00454FF0  8078 01 CC          CMP BYTE PTR DS:[EAX+1],0CC
00454FF4  74 1C               JE SHORT Pack.00455012
00454FF6  8078 02 CC          CMP BYTE PTR DS:[EAX+2],0CC
00454FFA  74 16               JE SHORT Pack.00455012
00454FFC  8078 03 CC          CMP BYTE PTR DS:[EAX+3],0CC
00455000  74 10               JE SHORT Pack.00455012
00455002  8078 04 CC          CMP BYTE PTR DS:[EAX+4],0CC
00455006  74 0A               JE SHORT Pack.00455012
00455008  50                  PUSH EAX
00455009  C3                  RETN                                 ; 返回。
```

返回后跟踪到：

```assembly
00450DFB  5A                  POP EDX                                ; Pack.00450DFA
00450DFC  81C2 49FAFFFF      ADD EDX,-5B7
00450E02  8B0A              MOV ECX,DWORD PTR DS:[EDX]
00450E04  83F9 00           CMP ECX,0
00450E07  ^ 0F84 57FFFFFF    JE Pack.00450D64                       ; 时间检测，不能跳!
00450E0D  2BC1              SUB EAX,ECX
00450E0F  ^ 0F88 4FFFFFFF    JS Pack.00450D64                       ; 时间检测，不能跳!
00450E15  2D D0070000        SUB EAX,7D0
00450E1A  ^ 0F89 44FFFFFF    JNS Pack.00450D64                      ; 时间检测，不能跳!
00450E20  B8 7612800E        MOV EAX,0E801276
00450E25  8902              MOV DWORD PTR DS:[EDX],EAX
00450E27  E8 01000000        CALL Pack.00450E2D
00450E2C  FF5A 81            CALL FAR FWORD PTR DS:[EDX-7F]         ; 远距呼叫
00450E2F  C2 4F00            RETN 4F
```

用 `F7` 走到：

```assembly
00455C73  53                  PUSH EBX
00455C74  56                  PUSH ESI
00455C75  57                  PUSH EDI                               ; Pack.004508B0
00455C76  8BD8                MOV EBX,EAX
00455C78  8B7424 0C           MOV ESI,DWORD PTR SS:[ESP+C]
00455C7C  51                  PUSH ECX
00455C7D  8B0C24              MOV ECX,DWORD PTR SS:[ESP]
00455C80  33FF                XOR EDI,EDI
00455C82  3BF2                CMP ESI,EDX
00455C84  73 0B               JNB SHORT Pack.00455C91
00455C86  8A041F              MOV AL,BYTE PTR DS:[EDI+EBX]
00455C89  3006                XOR BYTE PTR DS:[ESI],AL
00455C8B  46                  INC ESI
00455C8C  47                  INC EDI
00455C8D  ^ E2 F3               LOOPD SHORT Pack.00455C82
00455C8F  ^ EB EC               JMP SHORT Pack.00455C7D
00455C91  E8 01000000           CALL Pack.00455C97                  ; 直接 F4 到这里。
00455C96  FF58 05               CALL FAR FWORD PTR DS:[EAX+5]       ; 有花指令。
00455C99  0D 00000050           OR EAX,50000000
00455C9E  C3                    RETN
```

清除花指令后：

```assembly
00455C97  58                  POP EAX       ; Pack.00455C96      ; F7 走。
00455C98  05 0D000000         ADD EAX,0D
00455C9D  50                  PUSH EAX
00455C9E  C3                  RETN

00455C9F  EF                  OUT DX,EAX
00455CA0  49                  DEC ECX
00455CA1  2907                SUB DWORD PTR DS:[EDI],EAX
00455CA3  59                  POP ECX
00455CA4  5F                  POP EDI
00455CA5  5E                  POP ESI
00455CA6  5B                  POP EBX
00455CA7  C3                  RETN               ; 返回。
```

返回后用 `F7` 走，会来到：

```assembly
00450E7A  C3                  RETN               ; 来到这里。
00450E7B  34 62               XOR AL,62
00450E7D  74 00               JE SHORT Pack.00450E7F
```

来到 `00450E7A` 后，就有许多 `RETN`，一直 `F7` 返回到：

```assembly
00401D34  FF15 E8551300       CALL NEAR DWORD PTR DS:[1355E8]      ; 一直返回到这里!F7 进入。
00401D3A  64:A1 00000000      MOV EAX,DWORD PTR FS:[0]
00401D40  50                  PUSH EAX
00401D41  64:8925 00000000    MOV DWORD PTR FS:[0],ESP
```



### 00401D48 83EC 58 SUB ESP,58

F7 进入后来到：

```
00135608    58              POP EAX///来到这里。

00135609    55              PUSH EBP    ////stolen code

0013560A    8BEC            MOV EBP,ESP ////stolen code

0013560C    6A FF           PUSH -1     ////stolen code

0013560E    68 C0504000     PUSH 4050C0 ////stolen code

00135613    68 60284000     PUSH 402860 ////stolen code

00135618  - FFE0            JMP NEAR EAX          ; Pack.00401D3A

0013561A    0000            ADD BYTE PTR DS:[EAX],AL

0013561C    0000            ADD BYTE PTR DS:[EAX],AL

00401D2B    0000            ADD BYTE PTR DS:[EAX],AL////入口地址!

00401D2D    0000            ADD BYTE PTR DS:[EAX],AL

00401D2F    0000            ADD BYTE PTR DS:[EAX],AL

00401D31    0000            ADD BYTE PTR DS:[EAX],AL

00401D33    00FF            ADD BH,BH

00401D35    15 E8551300     ADC EAX,1355E8

00401D3A    64:A1 00000000  MOV EAX,DWORD PTR FS:[0]////00135168 跳到这里。

00401D40    50              PUSH EAX

00401D41    64:8925 000000>MOV DWORD PTR FS:[0],ESP

00401D48    83EC 58         SUB ESP,58

00401D4B    53              PUSH EBX

00401D4C    56              PUSH ESI

00401D4D    57              PUSH EDI

00401D4E    8965 E8         MOV DWORD PTR SS:[EBP-18],ESP

00401D51    FF15 30504000   CALL NEAR DWORD PTR DS:[405030]   ; Pack.00452596

00401D57    33D2            XOR EDX,EDX

00401D59    8AD4            MOV DL,AH

00401D5B    8915 08654000   MOV DWORD PTR DS:[406508],EDX

00401D61    8BC8            MOV ECX,EAX
```

### 二、还原 IAT

到 `00401D3A` 后，观察一下就可发现 `405000` 就是 IAT 表。

```
00404FF8  00 00 00 00 00 00 00 00  ........

00405000  20 3A 13 00 58 3A 13 00   :.X:.

00405008  90 3A 13 00 C8 3A 13 00  ?.?.

00405010  00 3B 13 00 38 3B 13 00  .;.8;.

00405018  70 3B 13 00 A8 3B 13 00  p;.?.

00405020  E0 3B 13 00 18 3C 13 00  ?.<.

00405028  8E 26 45 00 7D 26 45 00  ?E.}&E.

00405030  96 25 45 00 50 3C 13 00  ?E.P<.

00405038  88 3C 13 00 C0 3C 13 00  ?.?.

00405040  F8 3C 13 00 30 3D 13 00  ?.0=.

00405048  68 3D 13 00 A0 3D 13 00  h=.?.

00405050  68 52 13 00 A0 52 13 00  hR.\_R.

00405058  D8 52 13 00 10 53 13 00  豏.S.

00405060  48 53 13 00 80 53 13 00  HS.€S.

00405068  B8 53 13 00 F0 53 13 00  窼.餝.

00405070  28 54 13 00 60 54 13 00  (T.\`T.

00405078  98 54 13 00 D0 54 13 00  楾.蠺.

00405080  08 55 13 00 40 55 13 00  U.@U.

00405088  78 55 13 00 B0 55 13 00  xU.癠.

00405090  00 00 00 00 28 38 13 00  ....(8.

00405098  60 38 13 00 98 38 13 00  \`8.?.

004050A0  D0 38 13 00 08 39 13 00  ?.9.

004050A8  40 39 13 00 78 39 13 00  @9.x9.

004050B0  B0 39 13 00 E8 39 13 00  ?.?.

004050B8  00 00 00 00 00 00 00 00  ........
```

除了 `405028`-`405030` 处的三个外，其他的都到堆栈里了。

重新跟踪程序，在 `405094` 处的四个字节处下内存访问断点，就可以跟踪到修改 IAT 处。

```
00453AB0    50              PUSH EAX   ; user32.SetDlgItemTextA////EAX 中保存的就是正确的函数地址。

00453AB1    E8 B3F8FFFF     CALL Pack.00453369////加密 CALL，将函数地址转换成了堆栈地址了。

00453AB6    EB 0B           JMP SHORT Pack.00453AC3

00453AB8    8B4424 10       MOV EAX,DWORD PTR SS:[ESP+10]

00453ABC    57              PUSH EDI

00453ABD    50              PUSH EAX

00453ABE    E8 22EAFFFF     CALL Pack.004524E5

00453AC3    33C9            XOR ECX,ECX

00453AC5    8906            MOV DWORD PTR DS:[ESI],EAX////EAX 为加密后的地址。
```

为了得到原来的正确地址，偷懒的做法就是将

```
00453AB0 PUSH EAX

00453AB1    E8 B3F8FFFF     CALL Pack.00453369
```

这个句 NOP 掉。完美的方法可以参考 `softworm` 大侠的文章。

这样可以得到：

```
00405000  D1 3E E6 77 9D 63 E6 77  ?鎤漜鎤

00405008  FE 9B E6 77 BB D7 E7 77  䴖鎤蛔鐆

00405010  C4 51 E6 77 CC C3 E7 77  腝鎤堂鐆

00405018  7E 4C E6 77 FB 68 E6 77  ~L 鎤鹔鎤

00405020  CD 61 E6 77 FC 3D E6 77  蚢鎤?鎤

00405028  8E 26 45 00 7D 26 45 00  ?E.}&E.

00405030  96 25 45 00 7D 0E E7 77  ?E.}鐆

00405038  C4 20 E6 77 AB 3A E6 77  ?鎤?鎤

00405040  18 AA E7 77 D0 41 E6 77  蠥鎤

00405048  17 64 E6 77 17 64 E6 77  d 鎤 d 鎤

00405050  2D 54 E6 77 42 69 E7 77  -T 鎤 Bi 鐆

00405058  E3 5E F8 77 09 FB E6 77  鉤鴚.w

00405060  43 87 E6 77 0C 4F E6 77  C 囨 w.O 鎤

00405068  B1 4E E6 77 64 4E E6 77  盢鎤 dN 鎤

00405070  44 C6 FC 77 8B 6C F9 77  D 泣 w 媗鵺

00405078  F3 37 E6 77 C7 FA E6 77  ?鎤曲鎤

00405080  E6 3E E6 77 82 6C E6 77  ?鎤俵鎤

00405088  EF C0 FC 77 B9 4D E6 77  锢黽筂鎤

00405090  00 00 00 00 3B 47 E0 77  ....;G 鄔

00405098  19 47 E0 77 95 B4 E0 77  G 鄔暣鄔

004050A0  8F B7 E0 77 C1 95 DF 77  彿鄔習遷

004050A8  DA 5F DF 77 2C 61 DF 77  赺遷,a 遷

004050B0  5D 7E E0 77 3C 85 E0 77  \]~鄔<呧 w
```

观察 IAT 表，可以发现还有三个地址没有还原，就是 `405028`、`40502C` 和 `405030`，跟踪加壳程序，发现实际只有 `405028`、`40502C` 加密了。而 `405028` 实际调用 `45268E`，`40502C` 实际调用 `45267D`，因此可以得到：

```
00405028  8E 26 45 00 7D 26 45 00  ?E.}&E.

00405030  96 25 45 00             ?E.}
```

在 `00401D3A` 处 DUMP 并修复 stolen code，然后 `ImportREC` 修复 IAT 即可。

`4dd466d0e1e0cc123afbe504da2462ca`
