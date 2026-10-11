
# WORD 试卷王 V2.1

> 原文：[`www.pediy.com/kssd/pediy06/pediy6318.htm`](https://www.pediy.com/kssd/pediy06/pediy6318.htm)

第一次手动脱壳-----WORD 试卷王 V2.1 加强版 EXEStealth 2.5 - 2.6 的壳

破解作者：yzez\[DFCG\]\[BCG\]\[FCG\]

破解对象：WORD 试卷王 V2.1 加强版

下载地址：`http://www.skycn.com/soft/12539.html`

破解工具：`FI3.01`、`OLLYDBG1.09`、`ImportREC`

破解目的：只为技术而破解，不为破解而破解，如有失误，请大侠们指点!

说明：现在软件的加壳比较厉害，对于手动脱壳我是盲点，最近粗浅的看了一些脱壳方面的文章，也自己试着手动脱一下壳，我发现脱壳实在是一件麻烦事，可怜我的椅子，一天被我坐七八个小时，痛苦呀! 很感谢 `FLY` 的 `<<EXEStealth V2.5 脱壳+易语言程序——多程序启动管理器 V2.32 >>`的文章，给我很大启发!

### 1、脱壳过程

这篇文章要脱的对象是 `WORD 试卷王` 安装目录下的 `regedit.EXE`，用 `FI3.01` 查壳是：`EXEStealth 2.5 - 2.6`，好! 下面的过程就是打开 `Winamp`，边听音乐边手动脱壳吧! 用 `OD` 载入加壳的程序：`regedit.EXE`，第二个对话框，点否。载入程序后，把 `OD` 设置好，设置忽略所有异常。载入程序后，我们停在这里：

```
0040B060  >  PUSHAD*************************************
载入程序后我们停在这里!
0040B061    JMP     SHORT regedit.0040B085***F8 往下，跳!
0040B063    INC     EBP
0040B064    JS      SHORT regedit.0040B0CB
0040B066    PUSH    EBX
0040B067    JE      SHORT regedit.0040B0CE
0040B069    POPAD
0040B06A    INS     BYTE PTR ES:[EDI], DX            
0040B06B    JE      SHORT regedit.0040B0D5
0040B06D    AND     BYTE PTR DS:[77777720], CH
0040B073    JA      SHORT regedit.0040B0DB            
0040B076    BOUND   ESI, QWORD PTR DS:[EDI+EBP*2+6F]
0040B07A    INS     BYTE PTR ES:[EDI], DX           
0040B07B    INS     DWORD PTR ES:[EDI], DX           
0040B07C    POPAD
0040B07D    JNB     SHORT regedit.0040B0F3
0040B07F    JB      SHORT regedit.0040B0B0            
0040B082    ARPL    DWORD PTR DS:[EDI+6D], EBP
0040B085    CALL    regedit.0040B08A********跳到这里!F7 跟进!变形 JMP
0040B08A    POP     EBP                           
0040B08B    SUB     EBP, regedit.004025F0
0040B091    MOV     ECX, 0C69
0040B096    LEA     EDI, DWORD PTR SS:[EBP+402638]
0040B09C    MOV     ESI, EDI                       
0040B09E    LODS    BYTE PTR DS:[ESI]
0040B09F    SUB     AL, CL
0040B0A1    SUB     AL, 90
0040B0A3    STC
0040B0A4    JMP     SHORT regedit.0040B0A7*****注意这个 JMP 后的地址!
0040B0A6    JMP     EC3A71D5************************花指令!修改：E9---90，如何修改？鼠标点击这一行
***************************右键单击，在出现的对话框中，选择：BInary---Edit,直接在弹出的对话框中把 E9 改成
***************************90，点确定。
0040B0A6    NOP************************************************************修改后的程序变化!继续 F8 往下!
0040B0A7    SUB     AL, CL
0040B0A9    STC
0040B0AA    JMP     SHORT regedit.0040B0AD*****注意这个 JMP 后的地址!
0040B0AC    E8 EB01E8C0     CALL    C128B29C***花指令!E8 改成 90
0040B0AC    90             NOP*****************修改后程序的变化!
0040B0AD    EB 01          JMP     SHORT regedit.0040B0B0***注意这个 JMP 后的地址!
0040B0AF    E8 C0C034C0     CALL    C0757174***花指令!E8 改成 90
0040B0AF    90             NOP*****************修改花指令后程序的变化!
0040B0B0    C0C0 34         ROL     AL, 34                      
0040B0B3    C0C0 B4         ROL     AL, 0B4                     
0040B0B6    04 47          ADD     AL, 47
0040B0B8    F9             STC
0040B0B9    C0C0 2E         ROL     AL, 2E                      
0040B0BC    F8             CLC
0040B0BD    FEC8           DEC     AL
0040B0BF    04 75          ADD     AL, 75
0040B0C1    04 28          ADD     AL, 28
0040B0C3    2AC1           SUB     AL, CL
0040B0C5    90             NOP
0040B0C6    2AC1           SUB     AL, CL
0040B0C8    2AC1           SUB     AL, CL
0040B0CA    C0C8 4A         ROR     AL, 4A                      
0040B0CD    34 8E          XOR     AL, 8E
0040B0CF    AA             STOS    BYTE PTR ES:[EDI]
0040B0D0   ^ E2 CC         LOOPD   SHORT regedit.0040B09E***到这里后程序会变化!
0040B0D2   - E9 3824564E     JMP     4E96D50F
0040B0D2    8B38           MOV     EDI, DWORD PTR DS:[EAX]**变成这个样子!鼠标点击这一行，按 F4 跳到这里!
**********************************************************************************************
跳到这一行后程序会变成下面的样子!
0040B0D2    8B4424 20       MOV     EAX, DWORD PTR SS:[ESP+20]        
0040B0D6    40             INC     EAX
0040B0D7    78 0D          JS      SHORT regedit.0040B0E6
0040B0D9    C785 932D4000 0>MOV     DWORD PTR SS:[EBP+402D93], 1
0040B0E3    90             NOP
0040B0E4    EB 00          JMP     SHORT regedit.0040B0E6
0040B0E6    8D85 C6254000   LEA     EAX, DWORD PTR SS:[EBP+4025C6]
0040B0EC    B9 6C060000     MOV     ECX, 66C
0040B0F1    90             NOP
0040B0F2    E8 41020000     CALL    regedit.0040B338**************************此 CALL F8 带过!
0040B0F7    8985 8F2D4000   MOV     DWORD PTR SS:[EBP+402D8F], EAX
0040B0FD    8B85 872D4000   MOV     EAX, DWORD PTR SS:[EBP+402D87]
0040B103    83E0 01         AND     EAX, 1
0040B106    74 40          JE      SHORT regedit.0040B148
0040B108    8DB5 CF314000   LEA     ESI, DWORD PTR SS:[EBP+4031CF]
0040B10E    8D85 9B264000   LEA     EAX, DWORD PTR SS:[EBP+40269B]
0040B114    8946 08         MOV     DWORD PTR DS:[ESI+8], EAX
0040B117    8BFD           MOV     EDI, EBP
0040B119    8D85 1D2D4000   LEA     EAX, DWORD PTR SS:[EBP+402D1D]
0040B11F    33DB           XOR     EBX, EBX
0040B121    50             PUSH    EAX
0040B122    64:FF33        PUSH    DWORD PTR FS:[EBX]
0040B125    64:8923        MOV     DWORD PTR FS:[EBX], ESP
0040B128    BD 4B484342     MOV     EBP, 4243484B
0040B12D    66:B8 0400      MOV     AX, 4
0040B131    EB 01          JMP     SHORT regedit.0040B134********注意这个 JMP 的地址：0040B134
0040B133    FFCC           DEC     ESP***************************************花指令!FFCC 改成 9090
0040B133    90             NOP************************************************修改后!
0040B134    90             NOP
0040B135    8BEF           MOV     EBP, EDI
0040B137    33DB           XOR     EBX, EBX
0040B139    64:8F03        POP     DWORD PTR FS:[EBX]               
0040B13C    83C4 04        ADD     ESP, 4
0040B13F    3C 04          CMP     AL, 4
0040B141    74 05          JE      SHORT regedit.0040B148
0040B143    EB 01          JMP     SHORT regedit.0040B146*******注意这个 JMP 后的地址!
0040B145   - E9 61C38B85     JMP     85CC74AB****************************花指令，E9 改成 90
0040B145    90             NOP************************************************修改后!
0040B146    61             POPAD
0040B147    C3             RETN
0040B148    8B85 7F2D4000   MOV     EAX, DWORD PTR SS:[EBP+402D7F]   ; regedit.00400000
0040B148    8B85 7F2D4000   MOV     EAX, DWORD PTR SS:[EBP+402D7F]   ; regedit.00400000
0040B14E    0340 3C         ADD     EAX, DWORD PTR DS:[EAX+3C]
0040B151    05 80000000     ADD     EAX, 80
0040B156    8B08           MOV     ECX, DWORD PTR DS:[EAX]
0040B158    038D 7F2D4000   ADD     ECX, DWORD PTR SS:[EBP+402D7F]   ; regedit.00400000
0040B15E    83C1 10         ADD     ECX, 10
0040B161    8B01           MOV     EAX, DWORD PTR DS:[ECX]
0040B163    0385 7F2D4000   ADD     EAX, DWORD PTR SS:[EBP+402D7F]   ; regedit.00400000
0040B169    8B18           MOV     EBX, DWORD PTR DS:[EAX]
0040B16B    899D DB314000   MOV     DWORD PTR SS:[EBP+4031DB], EBX
0040B171    83C0 04         ADD     EAX, 4
0040B174    8B18           MOV     EBX, DWORD PTR DS:[EAX]
0040B176    899D DF314000   MOV     DWORD PTR SS:[EBP+4031DF], EBX
0040B17C    8D85 E3314000   LEA     EAX, DWORD PTR SS:[EBP+4031E3]
```



0040B182    50                   PUSH    EAX                       ; regedit.00400004

0040B183    FF95 DB314000       CALL    DWORD PTR SS:[EBP+4031DB]

0040B189    8BF0                MOV     ESI, EAX                   ; regedit.00400004

0040B18B    8985 F0314000       MOV     DWORD PTR SS:[EBP+4031F0], EAX   ; regedit.00400004

0040B191    8D85 F4314000       LEA     EAX, DWORD PTR SS:[EBP+4031F4]

0040B197    E8 96000000         CALL    regedit.0040B232

0040B19C    8985 05324000       MOV     DWORD PTR SS:[EBP+403205], EAX   ; regedit.00400004

0040B1A2    8D85 09324000       LEA     EAX, DWORD PTR SS:[EBP+403209]

0040B1A8    E8 85000000         CALL    regedit.0040B232

0040B1AD    8985 18324000       MOV     DWORD PTR SS:[EBP+403218], EAX

0040B1B3    8D85 1C324000       LEA     EAX, DWORD PTR SS:[EBP+40321C]

0040B1B9    E8 74000000         CALL    regedit.0040B232

0040B1BE    8985 2F324000       MOV     DWORD PTR SS:[EBP+40322F], EAX

0040B1C4    8D85 33324000       LEA     EAX, DWORD PTR SS:[EBP+403233]

0040B1CA    E8 63000000         CALL    regedit.0040B232

0040B1CF    8985 3F324000       MOV     DWORD PTR SS:[EBP+40323F], EAX

0040B1D5    8D85 43324000       LEA     EAX, DWORD PTR SS:[EBP+403243]

0040B1DB    E8 52000000         CALL    regedit.0040B232

0040B1E0    8985 4F324000       MOV     DWORD PTR SS:[EBP+40324F], EAX

0040B1E6    8D85 53324000       LEA     EAX, DWORD PTR SS:[EBP+403253]

0040B1EC    E8 41000000         CALL    regedit.0040B232

0040B1F1    8985 5E324000       MOV     DWORD PTR SS:[EBP+40325E], EAX

0040B1F7    8D85 62324000       LEA     EAX, DWORD PTR SS:[EBP+403262]

0040B1FD    E8 30000000         CALL    regedit.0040B232

0040B202    8985 6B324000       MOV     DWORD PTR SS:[EBP+40326B], EAX

0040B208    8D85 6F324000       LEA     EAX, DWORD PTR SS:[EBP+40326F]

0040B20E    E8 1F000000         CALL    regedit.0040B232

0040B213    8985 7B324000       MOV     DWORD PTR SS:[EBP+40327B], EAX

0040B219    8D85 7F324000       LEA     EAX, DWORD PTR SS:[EBP+40327F]

0040B21F    E8 0E000000         CALL    regedit.0040B232

0040B224    8985 8B324000       MOV     DWORD PTR SS:[EBP+40328B], EAX

0040B22A    8D85 A1274000       LEA     EAX, DWORD PTR SS:[EBP+4027A1]

0040B230    50                  PUSH    EAX

0040B231    C3                  RETN

0040B23B    F785 872D4000 1>TEST    DWORD PTR SS:[EBP+402D87], 10

0040B245    74 37               JE      SHORT regedit.0040B27E

0040B247    64:FF35 3000000>PUSH    DWORD PTR FS:[30]

0040B24E    58                  POP     EAX

0040B24F    85C0                TEST    EAX, EAX

0040B251    78 0F               JS      SHORT regedit.0040B262

0040B253    8B40 0C             MOV     EAX, DWORD PTR DS:[EAX+C]

0040B256    8B40 0C             MOV     EAX, DWORD PTR DS:[EAX+C]

0040B259    C740 20 0010000>MOV     DWORD PTR DS:[EAX+20], 1000

0040B260    EB 1C               JMP     SHORT regedit.0040B27E

0040B262    6A 00               PUSH    0

0040B264    FF95 05324000       CALL    DWORD PTR SS:[EBP+403205]

0040B26A    85D2                TEST    EDX, EDX

0040B26C    79 10               JNS     SHORT regedit.0040B27E

0040B26E    837A 08 FF          CMP     DWORD PTR DS:[EDX+8], -1

0040B272    75 0A               JNZ     SHORT regedit.0040B27E

0040B274    8B52 04             MOV     EDX, DWORD PTR DS:[EDX+4]

0040B277    C742 50 0010000>MOV     DWORD PTR DS:[EDX+50], 1000

0040B27E    8BBD 7F2D4000       MOV     EDI, DWORD PTR SS:[EBP+402D7F]

0040B284    037F 3C             ADD     EDI, DWORD PTR DS:[EDI+3C]

0040B287    8BB5 7F2D4000       MOV     ESI, DWORD PTR SS:[EBP+402D7F]

0040B28D    8B4F 54             MOV     ECX, DWORD PTR DS:[EDI+54]

0040B290    8D85 BD324000       LEA     EAX, DWORD PTR SS:[EBP+4032BD]

0040B296    50                  PUSH    EAX

0040B297    6A 04               PUSH    4

0040B299    51                  PUSH    ECX

0040B29A    FFB5 7F2D4000       PUSH    DWORD PTR SS:[EBP+402D7F]

0040B2A0    FF95 18324000       CALL    DWORD PTR SS:[EBP+403218]

0040B2A6    F785 872D4000 0>TEST    DWORD PTR SS:[EBP+402D87], 8

0040B2B0    0F84 A7000000       JE      regedit.0040B35D

0040B2B6    68 04010000         PUSH    104

0040B2BB    8DBD BD324000       LEA     EDI, DWORD PTR SS:[EBP+4032BD]

0040B2C1    57                  PUSH    EDI

0040B2C2    6A 00               PUSH    0

0040B2C4    FF95 2F324000       CALL    DWORD PTR SS:[EBP+40322F]

0040B2CA    6A 00               PUSH    0

0040B2CC    68 80000000         PUSH    80

0040B2D1    6A 03               PUSH    3

0040B2D3    6A 00               PUSH    0

0040B2D5    6A 01               PUSH    1

0040B2D7    68 00000080         PUSH    80000000

0040B2DC    57                  PUSH    EDI

0040B2DD    FF95 3F324000       CALL    DWORD PTR SS:[EBP+40323F]

0040B2E3    83F8 FF             CMP     EAX, -1\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*注意下面的跳转指令，这里把 EAX 的

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*值改成 FFFFFFFF

0040B2E6    75 04               JNZ     SHORT regedit.0040B2EC\*\*\*\*\*\*这里不能跳!

0040B2E8    33C0                XOR     EAX, EAX\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*上面一定不能跳，不跳往下!

0040B2EA    EB 71               JMP     SHORT regedit.0040B35D

0040B2EC    8BF8                MOV     EDI, EAX

0040B2EE    6A 00               PUSH    0

0040B2F0    57                  PUSH    EDI

0040B2F1    FF95 7B324000       CALL    DWORD PTR SS:[EBP+40327B]

0040B2F7    83E8 05             SUB     EAX, 5

0040B2FA    96                  XCHG    EAX, ESI

0040B2FB    56                  PUSH    ESI

0040B2FC    6A 40               PUSH    40

0040B2FE    FF95 4F324000       CALL    DWORD PTR SS:[EBP+40324F]

0040B304    0BC0                OR      EAX, EAX

0040B306    75 02               JNZ     SHORT regedit.0040B30A

0040B308    EB 4A               JMP     SHORT regedit.0040B354

0040B30A    93                  XCHG    EAX, EBX

0040B30B    6A 00               PUSH    0

0040B30D    8D85 BD324000       LEA     EAX, DWORD PTR SS:[EBP+4032BD]

0040B313    50                  PUSH    EAX

0040B314    56                  PUSH    ESI

0040B315    53                  PUSH    EBX

0040B316    57                  PUSH    EDI

0040B317    FF95 6B324000       CALL    DWORD PTR SS:[EBP+40326B]

0040B31D    8BC3                MOV     EAX, EBX

0040B31F    8BCE                MOV     ECX, ESI

0040B321    53                  PUSH    EBX

0040B322    57                  PUSH    EDI

0040B323    E8 10000000         CALL    regedit.0040B338

0040B328    8985 8B2D4000       MOV     DWORD PTR SS:[EBP+402D8B], EAX

0040B32E    5F                  POP     EDI

0040B32F    5B                  POP     EBX

0040B330    8D85 B2284000       LEA     EAX, DWORD PTR SS:[EBP+4028B2]

0040B336    50                  PUSH    EAX

0040B337    C3                  RETN

0040B338    8BF8                MOV     EDI, EAX

0040B33A    33C0                XOR     EAX, EAX

0040B33C    33DB                XOR     EBX, EBX

0040B33E    33D2                XOR     EDX, EDX

0040B340    8A07                MOV     AL, BYTE PTR DS:[EDI]

0040B342    F7E2                MUL     EDX

0040B344    03D8                ADD     EBX, EAX

0040B346    42                  INC     EDX
```



```
0040B347    47                INC     EDI                    
0040B348  ^ E2 F6            LOOPD   SHORT regedit.0040B340 
0040B34A    93                XCHG    EAX, EBX                
0040B34B    C3                RETN 
0040B34C    53                PUSH    EBX                       
0040B34D    FF95 5E324000     CALL    DWORD PTR SS:[EBP+40325E]       
0040B353    96                XCHG    EAX, ESI                 
0040B354    50                PUSH    EAX 
0040B355    57                PUSH    EDI                       
0040B356    FF95 8B324000     CALL    DWORD PTR SS:[EBP+40328B]         
0040B35C    58                POP     EAX                       
0040B35D    8B85 7F2D4000     MOV     EAX, DWORD PTR SS:[EBP+402D7F]****跳到这里!    
0040B363    BB 01000000       MOV     EBX, 1 
0040B368    E8 08000000       CALL    regedit.0040B375 
0040B36D    8D85 BE294000     LEA     EAX, DWORD PTR SS:[EBP+4029BE] 
0040B373    50                PUSH    EAX 
0040B374    C3                RETN********************************************返回指令，返回到 0040B458 
0040B458    8B9D 7F2D4000     MOV     EBX, DWORD PTR SS:[EBP+402D7F]****返回到这里! [EBP+402D7F]的值是：00400000   
0040B45E    039D 832D4000     ADD     EBX, DWORD PTR SS:[EBP+402D83]****EBX=00400000+127C=0040127C，这就是 OEP 的值! 
0040B464    C1CB 07           ROR     EBX, 7****************这一行我们把它改成 JMP EBX，直接跳到 OEP 处 
0040B466    90                NOP**********************修改后这里加一个 NOP! 
0040B467    895C24 10         MOV     DWORD PTR SS:[ESP+10], EBX 
0040127C    68 E0154000       PUSH    regedit.004015E0********跳到了 OEP 处，在这里我们用 OD 的 
************************脱壳插件脱出程序，保存!第一步完工! 
00401281    E8 EEFFFFFF       CALL    regedit.00401274 
00401286    0000              ADD     BYTE PTR DS:[EAX], AL 
00401288    0000              ADD     BYTE PTR DS:[EAX], AL 
0040128A    0000              ADD     BYTE PTR DS:[EAX], AL 
0040128C    3000              XOR     BYTE PTR DS:[EAX], AL 
0040128E    0000              ADD     BYTE PTR DS:[EAX], AL 
00401290    3800              CMP     BYTE PTR DS:[EAX], AL 
00401292    0000              ADD     BYTE PTR DS:[EAX], AL 
00401294    0000              ADD     BYTE PTR DS:[EAX], AL 
00401296    0000              ADD     BYTE PTR DS:[EAX], AL 
00401298    ED                IN      EAX, DX                       ; I/O command 
00401299    8E42 F6           MOV     ES, WORD PTR DS:[EDX-A]      ; Modification of segment register 
0040129C    1E                PUSH    DS 
0040129D    BF D71193CC       MOV     EDI, CC9311D7 
004012A2    0001              ADD     BYTE PTR DS:[ECX], AL 
004012A4    E1 03             LOOPDE  SHORT regedit.004012A9 
004012A6    7F 5B             JG      SHORT regedit.00401303 
004012A8    0000              ADD     BYTE PTR DS:[EAX], AL 
```

## 2. 第二步用 ImportREC 修复脱壳后的程序

先运行原程序，然后打开 `ImportREC`，在程序界面选取：`Attach to an Active Process`，找到 `regedit.EXE`，单击!载入原程序!在 OEP 栏内，我脱壳后的是：`0000B060`，

把它修改成真正的 OEP：`0000127C`，再点击：`IAT Auto Search`，会弹出一个对话框，提示 OEP 是否正确，这里是正确的，我们选中：`Get Imports` 修复，看看 `Imported Functions Found` 这个框里的内容，会提示修复的情况，这里全部修复，再用 `Fix Dump` 保存好文件，手动脱壳完成。试运行脱壳后的程序，正常运行!

`4dd466d0e1e0cc123afbe504da2462ca`
