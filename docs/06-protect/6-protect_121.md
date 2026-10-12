
# ROCKEY2

> 原文：[`www.pediy.com/kssd/pediy06/pediy6576.htm`](https://www.pediy.com/kssd/pediy06/pediy6576.htm)

无狗解狗: 30 秒内脱掉狗壳

作者邮箱: `ym-lp@163.com`

使用工具: `DebuggerKiller1.1`(内部测试版)

系统平台: `Windows 2000 Server`

软件下载: 点击此处下载（或鼠标右键另存为） (`ROCKEY2` 狗)

日期 : `2004-06-18`

**破解声明:**

破解最重要一步就是脱壳，壳就像一件衣服，穿上衣服的软件我们无法看法他的内在，要脱下它的衣服，脱开它。下面我讲解怎么来脱掉这个狗壳！

**声明:**

本文只是技术探讨.

**壳分析:**

运行 `DebuggerKiller.exe` 选择 `Load Driver` 加载调试器,选择 `Load File` 加载目标程序, 弹出内核调试器界面来到壳入口,调试器自动分析发现程序带有花指令,因为是最新的调试器，没有检查到调试器存在是很正常的。

本壳全部是用 `DebuggerKiller1.1`(内部测试版)工具脱成功。 `DebuggerKiller1.1`(内部测试版)不能把汇编导出到文件，只能用 `OD Dump` 出程序来讲解

我们来到加壳程序入口:

```
00416000  ROC>   E8 7BA1FFFF      CALL    ROCKEYNO.00410180  ;壳程序入口点.
00416005        50               PUSH    EAX
00416006        C3               RETN
```

按 `F8` 单步进入来到:

```
                          一值按 F7 单步
00410180        55               PUSH    EBP
00410181        8BEC             MOV     EBP,ESP
00410183        51               PUSH    ECX
00410184        56               PUSH    ESI
00410185        57               PUSH    EDI
00410186        8B45 04          MOV     EAX,DWORD PTR [EBP+4]
00410189        8D40 FB          LEA     EAX,DWORD PTR [EAX-5]
0041018C        8945 FC          MOV     DWORD PTR [EBP-4],EAX
0041018F        8B7D FC          MOV     EDI,DWORD PTR [EBP-4]
00410192        8B87 99000000    MOV     EAX,DWORD PTR [EDI+99]
00410198        8DB7 81000000    LEA     ESI,DWORD PTR [EDI+81]
0041019E        85C0             TEST    EAX,EAX
004101A0        75 1D            JNZ     SHORT ROCKEYNO.004101BF
004101A2        8B4E 08          MOV     ECX,DWORD PTR [ESI+8]
004101A5        8B56 04          MOV     EDX,DWORD PTR [ESI+4]
004101A8        8BC7             MOV     EAX,EDI
004101AA        56               PUSH    ESI
004101AB        2BC1             SUB     EAX,ECX
004101AD        57               PUSH    EDI
004101AE        03D0             ADD     EDX,EAX
004101B0        8956 04          MOV     DWORD PTR [ESI+4],EDX
004101B3        E8 48000000      CALL    ROCKEYNO.00410200
004101B8        C746 18 01000000 MOV     DWORD PTR [ESI+18],1
004101BF        0336             ADD     ESI,DWORD PTR [ESI]
004101C1        8B46 18          MOV     EAX,DWORD PTR [ESI+18]
004101C4        85C0             TEST    EAX,EAX
004101C6        75 23            JNZ     SHORT ROCKEYNO.004101EB
004101C8        8B56 08          MOV     EDX,DWORD PTR [ESI+8]
004101CB        8B46 04          MOV     EAX,DWORD PTR [ESI+4]
004101CE        8BCF             MOV     ECX,EDI
004101D0        56               PUSH    ESI
004101D1        2BCA             SUB     ECX,EDX
004101D3        03C1             ADD     EAX,ECX
004101D5        8946 04          MOV     DWORD PTR [ESI+4],EAX
004101D8        E8 13040000      CALL    ROCKEYNO.004105F0  ;来到这里。这里按 F8 一跳就完了。F7 跟进
004101DD        56               PUSH    ESI
004101DE        57               PUSH    EDI
004101DF        E8 1C000000      CALL    ROCKEYNO.00410200
004101E4        C746 18 01000000 MOV     DWORD PTR [ESI+18],1
004101EB        8B46 0C          MOV     EAX,DWORD PTR [ESI+C]
004101EE        8B4E 04          MOV     ECX,DWORD PTR [ESI+4]
004101F1        5F               POP     EDI
004101F2        03C1             ADD     EAX,ECX
004101F4        5E               POP     ESI
004101F5        8BE5             MOV     ESP,EBP
004101F7        5D               POP     EBP
004101F8        C3               RETN
```

按 `F7` 来到: 到这里了就不忙着单步。先分析下代码

分析发现程序运行到 `00410955` 地址程序就关闭了,所以不能让他执行到这里。

分析发现地址 `0041090C` 里面有一个 `RET 4`。作者感觉到了这是入口点。可不管是怎么设断都不进入。愚昧哟。心想可能不是这里吧，但还是坚持感觉。

好现在单步 `F7` 跟踪强行进入我们预分析的地址 `0041090C`

```
004105F0        83EC 3C          SUB     ESP,3C
004105F3        53               PUSH    EBX
004105F4        55               PUSH    EBP
004105F5        56               PUSH    ESI
004105F6        57               PUSH    EDI
004105F7        8B7C24 50        MOV     EDI,DWORD PTR [ESP+50]
004105FB        33ED             XOR     EBP,EBP
004105FD        33C0             XOR     EAX,EAX
004105FF        66:896C24 44     MOV     WORD PTR [ESP+44],BP
00410604        8B77 34          MOV     ESI,DWORD PTR [EDI+34]
00410607        33C9             XOR     ECX,ECX
00410609        03F7             ADD     ESI,EDI
0041060B        33DB             XOR     EBX,EBX
0041060D        C74424 2C 080000>MOV     DWORD PTR [ESP+2C],8
00410615        66:8B46 04       MOV     AX,WORD PTR [ESI+4]
00410619        894424 14        MOV     DWORD PTR [ESP+14],EAX
0041061D        66:8B4E 06       MOV     CX,WORD PTR [ESI+6]
00410621        C1E0 10          SHL     EAX,10
00410624        0BC1             OR      EAX,ECX
00410626        894424 14        MOV     DWORD PTR [ESP+14],EAX
0041062A        66:8B46 04       MOV     AX,WORD PTR [ESI+4]
0041062E        66:3146 08       XOR     WORD PTR [ESI+8],AX
00410632        8B5424 14        MOV     EDX,DWORD PTR [ESP+14]
00410636        8B4E 0A          MOV     ECX,DWORD PTR [ESI+A]
00410639        F7D0             NOT     EAX
0041063B        66:8946 04       MOV     WORD PTR [ESI+4],AX
0041063F        66:8B46 06       MOV     AX,WORD PTR [ESI+6]
00410643        33CA             XOR     ECX,EDX
00410645        66:F7D0          NOT     AX
00410648        894E 0A          MOV     DWORD PTR [ESI+A],ECX
0041064B        66:8946 06       MOV     WORD PTR [ESI+6],AX
0041064F        66:8B4E 04       MOV     CX,WORD PTR [ESI+4]
00410653        66:3BDD          CMP     BX,BP
00410656        894C24 1C        MOV     DWORD PTR [ESP+1C],ECX
0041065A        66:8B56 06       MOV     DX,WORD PTR [ESI+6]
0041065E        895424 18        MOV     DWORD PTR [ESP+18],EDX
00410662        896C24 24        MOV     DWORD PTR [ESP+24],EBP
00410666        896C24 20        MOV     DWORD PTR [ESP+20],EBP
0041066A        75 42            JNZ     SHORT ROCKEYNO.004106AE
0041066C        8D4424 3C        LEA     EAX,DWORD PTR [ESP+3C]
00410670        8D4C24 20        LEA     ECX,DWORD PTR [ESP+20]
00410674        50               PUSH    EAX
00410675        8D5424 28        LEA     EDX,DWORD PTR [ESP+28]
00410679        51               PUSH    ECX
0041067A        8D4424 20        LEA     EAX,DWORD PTR [ESP+20]
0041067E        52               PUSH    EDX
0041067F        8D4C24 28        LEA     ECX,DWORD PTR [ESP+28]
00410683        50               PUSH    EAX
00410684        8D5424 38        LEA     EDX,DWORD PTR [ESP+38]
00410688        51               PUSH    ECX
00410689        8D4424 28        LEA     EAX,DWORD PTR [ESP+28]
0041068D        52               PUSH    EDX
0041068E        8D4C24 2A        LEA     ECX,DWORD PTR [ESP+2A]
00410692        50               PUSH    EAX
00410693        51               PUSH    ECX
00410694        6A 01            PUSH    1
00410696        E8 05050000      CALL    ROCKEYNO.00410BA0  ;来到第一个 CALL 里面代码作者只花了 10 秒就看懂了,因为作者长年和驱动打交道, 里面实现功能是:先打开设备，没有设备就启动驱动发送控制命令。
0041069B        83C4 24          ADD     ESP,24
0041069E        66:85C0          TEST    AX,AX
004106A1        0F85 9C020000    JNZ     ROCKEYNO.00410943  ;失败就跳。不能给他跳，一跳程序就退出。强行修改标志位 Z=1
004106A7        BB 01000000      MOV     EBX,1
004106AC        EB 3B            JMP     SHORT ROCKEYNO.004106E9
004106AE        8D5424 3C        LEA     EDX,DWORD PTR [ESP+3C]
004106B2        8D4424 20        LEA     EAX,DWORD PTR [ESP+20]
004106B6        52               PUSH    EDX
004106B7        8D4C24 28        LEA     ECX,DWORD PTR [ESP+28]
004106BB        50               PUSH    EAX
004106BC        8D5424 20        LEA     EDX,DWORD PTR [ESP+20]
004106C0        51               PUSH    ECX
```



```
004106C1       8D4424 28        LEA     EAX,DWORD PTR [ESP+28]

004106C5       52               PUSH    EDX

004106C6       8D4C24 38        LEA     ECX,DWORD PTR [ESP+38]

004106CA       50               PUSH    EAX

004106CB       8D5424 28        LEA     EDX,DWORD PTR [ESP+28]

004106CF       51               PUSH    ECX

004106D0       8D4424 2A        LEA     EAX,DWORD PTR [ESP+2A]

004106D4       52               PUSH    EDX

004106D5       50               PUSH    EAX

004106D6       6A 02            PUSH    2

004106D8       E8 C3040000      CALL    ROCKEYNO.00410BA0

004106DD       83C4 24          ADD     ESP,24

004106E0       66:85C0          TEST    AX,AX

004106E3       0F85 5A020000    JNZ     ROCKEYNO.00410943

004106E9       8B46 0A          MOV     EAX,DWORD PTR [ESI+A]  ;JMP SHORT ROCKEYNO.004106E9  这个指令跳转到这里 ;一直按 F8

004106EC       8B4C24 14        MOV     ECX,DWORD PTR [ESP+14]

004106F0       3BC5             CMP     EAX,EBP

004106F2       74 14            JE      SHORT ROCKEYNO.00410708

004106F4       3BC8             CMP     ECX,EAX

004106F6       0F85 9F000000    JNZ     ROCKEYNO.0041079B

004106FC       894C24 46        MOV     DWORD PTR [ESP+46],ECX

00410700       C74424 2C 0E0000>MOV     DWORD PTR [ESP+2C],0E

00410708       894C24 30        MOV     DWORD PTR [ESP+30],ECX

0041070C       8D4C24 3C        LEA     ECX,DWORD PTR [ESP+3C]

00410710       8D5424 20        LEA     EDX,DWORD PTR [ESP+20]

00410714       51               PUSH    ECX

00410715       8D4424 28        LEA     EAX,DWORD PTR [ESP+28]

00410719       52               PUSH    EDX

0041071A       8D4C24 20        LEA     ECX,DWORD PTR [ESP+20]

0041071E       50               PUSH    EAX

0041071F       8D5424 28        LEA     EDX,DWORD PTR [ESP+28]

00410723       51               PUSH    ECX

00410724       8D4424 38        LEA     EAX,DWORD PTR [ESP+38]

00410728       52               PUSH    EDX

00410729       8D4C24 28        LEA     ECX,DWORD PTR [ESP+28]

0041072D       50               PUSH    EAX

0041072E       8D5424 2A        LEA     EDX,DWORD PTR [ESP+2A]

00410732       51               PUSH    ECX

00410733       52               PUSH    EDX

00410734       6A 03            PUSH    3

00410736       E8 65040000      CALL    ROCKEYNO.00410BA0  ;来到这个 CALL. 这个功能和 CALL ROCKEYNO.00410BA0 功能是一样的原理

0041073B       83C4 24          ADD     ESP,24

0041073E       66:85C0          TEST    AX,AX

00410741       0F85 FC010000    JNZ     ROCKEYNO.00410943  ;失败就跳.不能给他跳,一跳程序就退出.强行修改标志位 Z=1 ;一直按 F8

00410747       66:8B46 08       MOV     AX,WORD PTR [ESI+8]

0041074B       66:3D FFFF       CMP     AX,0FFFF

0041074F       74 6F            JE      SHORT ROCKEYNO.004107C0

00410751       83E0 0F          AND     EAX,0F

00410754       8D4C24 20        LEA     ECX,DWORD PTR [ESP+20]

00410758       894424 1C        MOV     DWORD PTR [ESP+1C],EAX

0041075C       8D4424 3C        LEA     EAX,DWORD PTR [ESP+3C]

00410760       50               PUSH    EAX

00410761       8D5424 28        LEA     EDX,DWORD PTR [ESP+28]

00410765       51               PUSH    ECX

00410766       8D4424 20        LEA     EAX,DWORD PTR [ESP+20]

0041076A       52               PUSH    EDX

0041076B       8D4C24 28        LEA     ECX,DWORD PTR [ESP+28]

0041076F       50               PUSH    EAX

00410770       8D5424 38        LEA     EDX,DWORD PTR [ESP+38]

00410774       51               PUSH    ECX

00410775       8D4424 28        LEA     EAX,DWORD PTR [ESP+28]

00410779       52               PUSH    EDX

0041077A       8D4C24 2A        LEA     ECX,DWORD PTR [ESP+2A]

0041077E       50               PUSH    EAX

0041077F       51               PUSH    ECX

00410780       6A 0C            PUSH    0C

00410782       E8 19040000      CALL    ROCKEYNO.00410BA0

00410787       83C4 24          ADD     ESP,24

0041078A       66:85C0          TEST    AX,AX

0041078D       0F85 7E010000    JNZ     ROCKEYNO.00410911

00410793       66:837C24 18 01  CMP     WORD PTR [ESP+18],1

00410799       74 0B            JE      SHORT ROCKEYNO.004107A6

0041079B       66:83FB 02       CMP     BX,2

0041079F       74 1F            JE      SHORT ROCKEYNO.004107C0

004107A1     ^ E9 A9FEFFFF      JMP     ROCKEYNO.0041064F

004107A6       8B4424 2C        MOV     EAX,DWORD PTR [ESP+2C]

004107AA       66:8B56 08       MOV     DX,WORD PTR [ESI+8]

004107AE       83F8 08          CMP     EAX,8

004107B1       66:895424 44     MOV     WORD PTR [ESP+44],DX

004107B6       75 08            JNZ     SHORT ROCKEYNO.004107C0

004107B8       C74424 2C 0A0000>MOV     DWORD PTR [ESP+2C],0A

004107C0       66:8B4424 12     MOV     AX,WORD PTR [ESP+12] ;JE SHORT ROCKEYNO.004107C0 掉转到这里.  继续一直按 F8

004107C5       8B4C24 30        MOV     ECX,DWORD PTR [ESP+30]

004107C9       66:A3 50D14000   MOV     WORD PTR [40D150],AX

004107CF       894E 0A          MOV     DWORD PTR [ESI+A],ECX

004107D2       8B5F 2C          MOV     EBX,DWORD PTR [EDI+2C]

004107D5       33C0             XOR     EAX,EAX

004107D7       03DF             ADD     EBX,EDI

004107D9       896C24 30        MOV     DWORD PTR [ESP+30],EBP

004107DD       66:8B03          MOV     AX,WORD PTR [EBX]

004107E0       83C3 02          ADD     EBX,2

004107E3       3BC5             CMP     EAX,EBP

004107E5       894424 38        MOV     DWORD PTR [ESP+38],EAX

004107E9       0F8E BA000000    JLE     ROCKEYNO.004108A9     ;好.到这里大家注意了,这个是进入口的 强行修改标志 Z=1 让他跳

004107EF       EB 04            JMP     SHORT ROCKEYNO.004107F5;/////////////////////////////////////////////////////////

004107F1       8B7C24 50        MOV     EDI,DWORD PTR [ESP+50]               ;以下代码不重要，我们强行跳过了

004107F5       8B03             MOV     EAX,DWORD PTR [EBX]

004107F7       8B6B 04          MOV     EBP,DWORD PTR [EBX+4]

004107FA       83C3 04          ADD     EBX,4

004107FD       894424 34        MOV     DWORD PTR [ESP+34],EAX

00410801       894424 28        MOV     DWORD PTR [ESP+28],EAX

00410805       8D5424 3C        LEA     EDX,DWORD PTR [ESP+3C]

00410809       8D4424 20        LEA     EAX,DWORD PTR [ESP+20]

0041080D       52               PUSH    EDX

0041080E       8D4C24 28        LEA     ECX,DWORD PTR [ESP+28]

00410812       50               PUSH    EAX

00410813       8D5424 20        LEA     EDX,DWORD PTR [ESP+20]

00410817       51               PUSH    ECX

00410818       8D4424 28        LEA     EAX,DWORD PTR [ESP+28]

0041081C       52               PUSH    EDX

0041081D       8D4C24 38        LEA     ECX,DWORD PTR [ESP+38]

00410821       50               PUSH    EAX

00410822       8D5424 28        LEA     EDX,DWORD PTR [ESP+28]

00410826       51               PUSH    ECX

00410827       8D4424 2A        LEA     EAX,DWORD PTR [ESP+2A]

0041082B       52               PUSH    EDX

0041082C       50               PUSH    EAX

0041082D       6A 08            PUSH    8

0041082F       83C3 04          ADD     EBX,4

00410832       E8 69030000      CALL    ROCKEYNO.00410BA0  ;来到这个 CALL. 这个功能和 CALL ROCKEYNO.00410BA0 功能是一样的原理

00410837       83C4 24          ADD     ESP,24

0041083A       66:85C0          TEST    AX,AX

0041083D       0F85 CC000000    JNZ     ROCKEYNO.0041090F  ;失败就跳.不能给他跳,一跳程序就退出.

00410843       66:8B4C24 1C     MOV     CX,WORD PTR [ESP+1C]

00410848       8B7F 04          MOV     EDI,DWORD PTR [EDI+4]

0041084B       66:8B5424 18     MOV     DX,WORD PTR [ESP+18]

00410850       66:8B4424 24     MOV     AX,WORD PTR [ESP+24]

00410855       66:894C24 3C     MOV     WORD PTR [ESP+3C],CX

0041085A       66:8B4C24 20     MOV     CX,WORD PTR [ESP+20]
```



```
0041085F       66:894C24 42      MOV     WORD PTR [ESP+42],CX

00410864       8B4C24 34         MOV     ECX,DWORD PTR [ESP+34]

00410868       03F9              ADD     EDI,ECX

0041086A       33C9              XOR     ECX,ECX

0041086C       85ED              TEST    EBP,EBP

0041086E       66:895424 3E      MOV     WORD PTR [ESP+3E],DX

00410873       66:894424 40      MOV     WORD PTR [ESP+40],AX

00410878       7E 18             JLE     SHORT ROCKEYNO.00410892

0041087A       8BC1              MOV     EAX,ECX

0041087C       99                CDQ

0041087D       F77C24 2C         IDIV    DWORD PTR [ESP+2C]

00410881       8A0439            MOV     AL,BYTE PTR [ECX+EDI]

00410884       8A5414 3C         MOV     DL,BYTE PTR [ESP+EDX+3C]

00410888       32C2              XOR     AL,DL

0041088A       880439            MOV     BYTE PTR [ECX+EDI],AL

0041088D       41                INC     ECX

0041088E       3BCD              CMP     ECX,EBP

00410890     ^ 7C E8             JL      SHORT ROCKEYNO.0041087A

00410892       8B4424 30         MOV     EAX,DWORD PTR [ESP+30]

00410896       8B4C24 38         MOV     ECX,DWORD PTR [ESP+38]

0041089A       40                INC     EAX

0041089B       3BC1              CMP     EAX,ECX

0041089D       894424 30         MOV     DWORD PTR [ESP+30],EAX

004108A1     ^ 0F8C 4AFFFFFF     JL      ROCKEYNO.004107F1

004108A7       33ED              XOR     EBP,EBP/////////////////////////////////////////////////////////////////////////////////

004108A9       8D4424 3C         LEA     EAX,DWORD PTR [ESP+3C] ;JLE ROCKEYNO.004108A9 跳转到这里. 继续一直按 F8

004108AD       8D4C24 20         LEA     ECX,DWORD PTR [ESP+20]

004108B1       50                PUSH    EAX

004108B2       8D5424 28         LEA     EDX,DWORD PTR [ESP+28]

004108B6       51                PUSH    ECX

004108B7       8D4424 20         LEA     EAX,DWORD PTR [ESP+20]

004108BB       52                PUSH    EDX

004108BC       8D4C24 28         LEA     ECX,DWORD PTR [ESP+28]

004108C0       50                PUSH    EAX

004108C1       8D5424 38         LEA     EDX,DWORD PTR [ESP+38]

004108C5       51                PUSH    ECX

004108C6       8D4424 28         LEA     EAX,DWORD PTR [ESP+28]

004108CA       52                PUSH    EDX

004108CB       8D4C24 2A         LEA     ECX,DWORD PTR [ESP+2A]

004108CF       50                PUSH    EAX

004108D0       51                PUSH    ECX

004108D1       6A 04             PUSH    4

004108D3       E8 C8020000       CALL    ROCKEYNO.00410BA0

004108D8       8B06              MOV     EAX,DWORD PTR [ESI]

004108DA       83C4 24           ADD     ESP,24

004108DD       3BC5              CMP     EAX,EBP

004108DF       74 24             JE      SHORT ROCKEYNO.00410905

004108E1       68 544E4100       PUSH    ROCKEYNO.00414E54

004108E6       55                PUSH    EBP

004108E7       56                PUSH    ESI

004108E8       68 D0034100       PUSH    ROCKEYNO.004103D0

004108ED       55                PUSH    EBP

004108EE       55                PUSH    EBP

004108EF       FF15 44D04000     CALL    DWORD PTR [40D044]

004108F5       A3 504E4100       MOV     DWORD PTR [414E50],EAX

004108FA       FF15 48D04000     CALL    DWORD PTR [40D048]

00410900       A3 54D14000       MOV     DWORD PTR [40D154],EAX

00410905       5F                POP     EDI

00410906       5E                POP     ESI

00410907       5D                POP     EBP

00410908       5B                POP     EBX

00410909       83C4 3C           ADD     ESP,3C

0041090C       C2 0400           RETN    4           ;好 来到了我们预分析的地址 0041090C

0041090F       33ED              XOR     EBP,EBP

00410911       8D4424 3C         LEA     EAX,DWORD PTR [ESP+3C]

00410915       8D4C24 20         LEA     ECX,DWORD PTR [ESP+20]

00410919       50                PUSH    EAX

0041091A       8D5424 28         LEA     EDX,DWORD PTR [ESP+28]

0041091E       51                PUSH    ECX

0041091F       8D4424 20         LEA     EAX,DWORD PTR [ESP+20]

00410923       52                PUSH    EDX

00410924       8D4C24 28         LEA     ECX,DWORD PTR [ESP+28]

00410928       50                PUSH    EAX

00410929       8D5424 38         LEA     EDX,DWORD PTR [ESP+38]

0041092D       51                PUSH    ECX

0041092E       8D4424 28         LEA     EAX,DWORD PTR [ESP+28]

00410932       52                PUSH    EDX

00410933       8D4C24 2A         LEA     ECX,DWORD PTR [ESP+2A]

00410937       50                PUSH    EAX

00410938       51                PUSH    ECX

00410939       6A 04             PUSH    4

0041093B       E8 60020000       CALL    ROCKEYNO.00410BA0

00410940       83C4 24           ADD     ESP,24

00410943       8D56 0E           LEA     EDX,DWORD PTR [ESI+E]

00410946       6A 10             PUSH    10

00410948       83C6 5E           ADD     ESI,5E

0041094B       52                PUSH    EDX

0041094C       56                PUSH    ESI

0041094D       55                PUSH    EBP

0041094E       FF15 20D14000     CALL    DWORD PTR [40D120]

00410954       55                PUSH    EBP

00410955       FF15 54D04000     CALL    DWORD PTR [40D054]
```

`0041090C` 地址 调用 `RETN 4` 来到:

```
004101DD       56                PUSH    ESI               ;继续一直按 F8

004101DE       57                PUSH    EDI

004101DF       E8 1C000000       CALL    ROCKEYNO.00410200

004101E4       C746 18 01000000 MOV     DWORD PTR [ESI+18],1

004101EB       8B46 0C           MOV     EAX,DWORD PTR [ESI+C]

004101EE       8B4E 04           MOV     ECX,DWORD PTR [ESI+4]

004101F1       5F                POP     EDI

004101F2       03C1              ADD     EAX,ECX

004101F4       5E                POP     ESI

004101F5       8BE5              MOV     ESP,EBP

004101F7       5D                POP     EBP

004101F8       C3                RETN
```

`004101F8` 地址 调用 `RETN` 来到:

```
00416005       50                PUSH    EAX

00416006       C3                RETN                ;哈哈。返回就是程序入口了.
```

`00416006` 地址 调用 `RETN` 来到: 程序入口

```
004010CC       55                PUSH    EBP            ;真正的 OEP 在 004010CC

004010CD       8BEC              MOV     EBP,ESP

004010CF       83EC 44           SUB     ESP,44

004010D2       56                PUSH    ESI

004010D3       FF15 E4634000     CALL    DWORD PTR [4063E4]
```

最后。导出内存。 用 `ImportREC` 加载目标程序。进行恢复 `IAT` `OEP` 在 `004010CC`

`4dd466d0e1e0cc123afbe504da2462ca`
