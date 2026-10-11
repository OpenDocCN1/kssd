
# mtasia V3.0.2 

> 原文：[`www.pediy.com/kssd/pediy05/pediy50420.htm`](https://www.pediy.com/kssd/pediy05/pediy50420.htm)

破解对象：`Camtasia` V3.0.2

破解工具：`TRW2000` V1.22

软件简介：

`Camtasia` 是套屏幕摄影及影像制造工具，它能帮助你做演出、软件辅助、及自动文件。你可以用它来建立 `AVI` 或影像文件、捕捉整个屏幕或局部放大、加入特效等。它的 `Producer` 功能让你可以修剪、切割、或贴上 `AVI` 影像修剪片段，及记录你的叙述。`Camtasia` 包含一个特别的程序，具有功能强大的压缩功能，能存成 `AVI` 或 `ASF` 格式，且拥有高分辨率。

版本 `2.0` 有实时输出功能，可以在网上传送 desktop activity。你能透过电子邮件、于网站上、在网上、或于 `CD` 片上共享你的 `Camtasia` 屏幕录像。它有 `ScreenPad` 及 `Watermark` 效果，能将批注、标题、标识、及其它图形加入至影片中。

版本 `2.0.1` 主要是错误修改后的版本。然而它也增加了一些强化功能，如支持 `Real Video 8` 及修改了原本支持 `common line options` 的功能。

这个软件我跟踪了很久，直到昨晚才跟出来（我菜啊！毕竟学 `crack` 才半年不到）。这个软件的验证过程是我 `crack` 到现在最难找的一个，能找到注册码对我这样的初学者来说实在是有 90％的运气。不过我想分析过程中的一些思路还是可以给和我一样的初学者借鉴一下，所以在这里把过程贴出来。

老规矩，先填入用户名和假的注册码 `1213141516171819`（为什么要 18 位？后面会解释到），下断点 `bpx hmemcpy` 开始跟踪。

```
017F:00449B09 50               PUSH     EAX  
017F:00449B0A 51               PUSH     ECX  
017F:00449B0B 8B8E94000000     MOV      ECX,[ESI+94]  
017F:00449B11 56               PUSH     ESI  
017F:00449B12 E8E9E3FFFF       CALL     00447F00－－－－call（1），关键，进入  
017F:00449B17 8BD8             MOV      EBX,EAX  
```

`F10` 带过上面这个 `call` 是会出现注册失败提示，按照常规的切入点应往上找可疑的跳转，但我跟踪了很久都没有找到，无奈之下走进了这个 `call`，才发现真正的验证过程就在这个 `call` 中。虽然以前看到过这种软件，但我自己还是第一次碰到。

call（1）：

```
17F:00447F00 53               PUSH     EBX  
017F:00447F01 55               PUSH     EBP  
017F:00447F02 8B6C2414         MOV      EBP,[ESP+14]  
017F:00447F06 56               PUSH     ESI  
017F:00447F07 8BF1             MOV      ESI,ECX  
017F:00447F09 57               PUSH     EDI  
017F:00447F0A 8B4500           MOV      EAX,[EBP+00]－－－－假注册码地址入 eax  
017F:00447F0D 8B48F8           MOV      ECX,[EAX-08]－－－－假注册码长度入 ecx  
017F:00447F10 85C9             TEST     ECX,ECX－－－－测试输入的用户名长度是否为零？  
017F:00447F12 7518             JNZ      00447F2C  
017F:00447F14 6AFF             PUSH     BYTE -01  
017F:00447F16 6A00             PUSH     BYTE +00  
017F:00447F18 683E280000       PUSH     DWORD 283E  
017F:00447F1D E8E31B0600       CALL     004A9B05  
017F:00447F22 5F               POP      EDI  
017F:00447F23 5E               POP      ESI  
017F:00447F24 5D               POP      EBP  
017F:00447F25 6633C0           XOR      AX,AX  
017F:00447F28 5B               POP      EBX  
017F:00447F29 C20C00           RET      0C  
017F:00447F2C 8B7C2418         MOV      EDI,[ESP+18]  
017F:00447F30 8BCF             MOV      ECX,EDI  
017F:00447F32 E881520500       CALL     0049D1B8  
017F:00447F37 8B07             MOV      EAX,[EDI]－－－－注册码地址入 eax  
017F:00447F39 8B58F8           MOV      EBX,[EAX-08]－－－注册码长度入 ebx  
017F:00447F3C 83FB0E           CMP      EBX,BYTE +0E－－－－比较注册码位数是否大于 0Eh  
017F:00447F3F 0F8CB9000000     JL       NEAR 00447FFE  
017F:00447F45 689CD44E00       PUSH     DWORD 004ED49C  
017F:00447F4A 50               PUSH     EAX－－－－注册码地址入栈  
017F:00447F4B E830370100       CALL     0045B680－－－－call(2)，检验注册码中的字符是否为“123456789ABCDEF－”之一，若有不是的就出错  
017F:00447F50 83C408           ADD      ESP,BYTE +08  
017F:00447F53 3BC3             CMP      EAX,EBX－－－－测试注册码有效长度，call（2）结果正确就不跳  
017F:00447F55 0F85A3000000     JNZ      NEAR 00447FFE  
017F:00447F5B 6800374F00       PUSH     DWORD 004F3700  
017F:00447F60 6898D44E00       PUSH     DWORD 004ED498  
017F:00447F65 8BCF             MOV      ECX,EDI  
017F:00447F67 E8BC4A0500       CALL     0049CA28  
017F:00447F6C 8B07             MOV      EAX,[EDI]  
017F:00447F6E 8B4D00           MOV      ECX,[EBP+00]  
017F:00447F71 8B16             MOV      EDX,[ESI]  
017F:00447F73 50               PUSH     EAX  
017F:00447F74 51               PUSH     ECX  
017F:00447F75 8BCE             MOV      ECX,ESI  
017F:00447F77 FF520C           CALL     NEAR [EDX+0C]－－－－－有问题！进入！  
017F:00447F7A 8BD8             MOV      EBX,EAX  
017F:00447F7C 80FB01           CMP      BL,01  
017F:00447F7F 7545             JNZ      00447FC6－－－－－－call（3）关键跳转！跳就完了！  
017F:00447F81 C7460401000000   MOV      DWORD [ESI+04],01  
017F:00447F88 C7460C00000000   MOV      DWORD [ESI+0C],00  
017F:00447F8F 8B07             MOV      EAX,[EDI]  
017F:00447F91 8BCE             MOV      ECX,ESI  
017F:00447F93 50               PUSH     EAX  
017F:00447F94 6894C24E00       PUSH     DWORD 004EC294  
017F:00447F99 E882FDFFFF       CALL     00447D20  
017F:00447F9E 8B4D00           MOV      ECX,[EBP+00]  
017F:00447FA1 51               PUSH     ECX  
017F:00447FA2 68B4C24E00       PUSH     DWORD 004EC2B4  
017F:00447FA7 8BCE             MOV      ECX,ESI  
017F:00447FA9 E872FDFFFF       CALL     00447D20  
017F:00447FAE 8B542414         MOV      EDX,[ESP+14]  
017F:00447FB2 57               PUSH     EDI  
017F:00447FB3 55               PUSH     EBP  
017F:00447FB4 52               PUSH     EDX  
017F:00447FB5 8BCE             MOV      ECX,ESI  
017F:00447FB7 E864000000       CALL     00448020  
017F:00447FBC 5F               POP      EDI  
017F:00447FBD 5E               POP      ESI  
017F:00447FBE 668BC3           MOV      AX,BX  
017F:00447FC1 5D               POP      EBP  
017F:00447FC2 5B               POP      EBX  
017F:00447FC3 C20C00           RET      0C  
017F:00447FC6 53               PUSH     EBX  
017F:00447FC7 8BCE             MOV      ECX,ESI  
017F:00447FC9 E822FDFFFF       CALL     00447CF0  
017F:00447FCE 85C0             TEST     EAX,EAX  
017F:00447FD0 6AFF             PUSH     BYTE -01  
017F:00447FD2 6A00             PUSH     BYTE +00  
017F:00447FD4 7414             JZ       00447FEA－检验是否为老版本的序列号，不是则跳，转向出错处  
017F:00447FD6 6841280000       PUSH     DWORD 2841  
017F:00447FDB E8251B0600       CALL     004A9B05  
017F:00447FE0 5F               POP      EDI  
017F:00447FE1 5E               POP      ESI  
017F:00447FE2 668BC3           MOV      AX,BX  
017F:00447FE5 5D               POP      EBP  
017F:00447FE6 5B               POP      EBX  
017F:00447FE7 C20C00           RET      0C  
017F:00447FEA 683D280000       PUSH     DWORD 283D  
017F:00447FEF E8111B0600       CALL     004A9B05  
017F:00447FF4 5F               POP      EDI  
017F:00447FF5 5E               POP      ESI  
017F:00447FF6 668BC3           MOV      AX,BX  
017F:00447FF9 5D               POP      EBP  
017F:00447FFA 5B               POP      EBX
```

call（2）：

```
017F:0045B680 55               PUSH     EBP  
017F:0045B681 8BEC             MOV      EBP,ESP  
017F:0045B683 56               PUSH     ESI  
017F:0045B684 33C0             XOR      EAX,EAX  
017F:0045B686 50               PUSH     EAX  
017F:0045B687 50               PUSH     EAX  
017F:0045B688 50               PUSH     EAX  
017F:0045B689 50               PUSH     EAX  
017F:0045B68A 50               PUSH     EAX  
017F:0045B68B 50               PUSH     EAX  
017F:0045B68C 50               PUSH     EAX  
017F:0045B68D 50               PUSH     EAX  
017F:0045B68E 8B550C           MOV      EDX,[EBP+0C]  
017F:0045B691 8D4900           LEA      ECX,[ECX+00]  
017F:0045B694 8A02             MOV      AL,[EDX]  
017F:0045B696 0AC0             OR       AL,AL  
017F:0045B698 7407             JZ       0045B6A1  
017F:0045B69A 42               INC      EDX  
017F:0045B69B 0FAB0424         BTS      [ESP],EAX  
017F:0045B69F EBF3             JMP      SHORT 0045B694  
017F:0045B6A1 8B7508           MOV      ESI,[EBP+08]  
017F:0045B6A4 83C9FF           OR       ECX,BYTE -01  
017F:0045B6A7 90               NOP      
017F:0045B6A8 41               INC      ECX  
017F:0045B6A9 8A06             MOV      AL,[ESI]－－－依次取注册码每一位的 ASCII 码  
017F:0045B6AB 0AC0             OR       AL,AL  
017F:0045B6AD 7407             JZ       0045B6B6－－－－是否结束？  
```



017F:0045B6AF 46                  INC      ESI

017F:0045B6B0 0FA30424            BT       [ESP],EAX

017F:0045B6B4 72F2                JC       0045B6A8

017F:0045B6B6 8BC1                MOV      EAX,ECX

017F:0045B6B8 83C420              ADD      ESP,BYTE +20

017F:0045B6BB 5E                  POP      ESI

017F:0045B6BC C9                  LEAVE

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

`call(3)`：

注意：这个 `call` 的后半部分有大量的跳向 `436E7E` 的跳转，可以判断这些是跳向出错处的，所以在跟踪过程中要把握好程序走向。

017F:00436CD0 6AFF                PUSH     BYTE -01

017F:00436CD2 68A0E84B00          PUSH     DWORD 004BE8A0

017F:00436CD7 64A100000000        MOV      EAX,[FS:00]

017F:00436CDD 50                  PUSH     EAX

017F:00436CDE 64892500000000      MOV      [FS:00],ESP

017F:00436CE5 83EC58              SUB      ESP,BYTE +58

017F:00436CE8 8B44246C            MOV      EAX,[ESP+6C]

017F:00436CEC 53                  PUSH     EBX

017F:00436CED 56                  PUSH     ESI

017F:00436CEE 57                  PUSH     EDI

017F:00436CEF 50                  PUSH     EAX

017F:00436CF0 8D4C247C            LEA      ECX,[ESP+7C]

017F:00436CF4 C744241800000000    MOV      DWORD [ESP+18],00

017F:00436CFC C744242006000000    MOV      DWORD [ESP+20],06

017F:00436D04 C644241300          MOV      BYTE [ESP+13],00

017F:00436D09 B301                MOV      BL,01

017F:00436D0B E86FDB0600          CALL     004A487F

017F:00436D10 8B0DB4D84E00        MOV      ECX,[004ED8B4]

017F:00436D16 C744246C00000000    MOV      DWORD [ESP+6C],00

017F:00436D1E 894C2410            MOV      [ESP+10],ECX

017F:00436D22 8D542418            LEA      EDX,[ESP+18]

017F:00436D26 6A04                PUSH     BYTE +04

017F:00436D28 52                  PUSH     EDX

017F:00436D29 8D8C2480000000      LEA      ECX,[ESP+80]

017F:00436D30 885C2474            MOV      [ESP+74],BL

017F:00436D34 E85C5F0600          CALL     0049CC95

017F:00436D39 50                  PUSH     EAX

017F:00436D3A 8D4C2414            LEA      ECX,[ESP+14]

017F:00436D3E C644247002          MOV      BYTE [ESP+70],02

017F:00436D43 E802DC0600          CALL     004A494A

017F:00436D48 8D4C2418            LEA      ECX,[ESP+18]

017F:00436D4C 885C246C            MOV      [ESP+6C],BL

017F:00436D50 E8BCDA0600          CALL     004A4811

017F:00436D55 8B442478            MOV      EAX,[ESP+78]

017F:00436D59 8D4C2418            LEA      ECX,[ESP+18]

017F:00436D5D 8B40F8               MOV      EAX,[EAX-08]

017F:00436D60 83C0FC              ADD      EAX,BYTE -04

017F:00436D63 50                  PUSH     EAX

017F:00436D64 51                  PUSH     ECX

017F:00436D65 8D8C2480000000      LEA      ECX,[ESP+80]

017F:00436D6C E8A05F0600          CALL     0049CD11

017F:00436D71 50                  PUSH     EAX

017F:00436D72 8D4C247C            LEA      ECX,[ESP+7C]

017F:00436D76 C644247003          MOV      BYTE [ESP+70],03

017F:00436D7B E8CADB0600          CALL     004A494A

017F:00436D80 8D4C2418            LEA      ECX,[ESP+18]

017F:00436D84 885C246C            MOV      [ESP+6C],BL

017F:00436D88 E884DA0600          CALL     004A4811

017F:00436D8D 33D2                XOR      EDX,EDX

017F:00436D8F 8B742478            MOV      ESI,[ESP+78]

017F:00436D93 89542420            MOV      [ESP+20],EDX

017F:00436D97 8D7C2420            LEA      EDI,[ESP+20]

017F:00436D9B 89542424            MOV      [ESP+24],EDX

017F:00436D9F 6A10                PUSH     BYTE +10

017F:00436DA1 8954242C            MOV      [ESP+2C],EDX

017F:00436DA5 6689542430          MOV      [ESP+30],DX

017F:00436DAA 88542432            MOV      [ESP+32],DL

017F:00436DAE 8B4EF8               MOV      ECX,[ESI-08]

017F:00436DB1 8BC1                MOV      EAX,ECX

017F:00436DB3 52                  PUSH     EDX

017F:00436DB4 C1E902              SHR      ECX,02

017F:00436DB7 F3A5                REP MOVSD

017F:00436DB9 8BC8                MOV      ECX,EAX

017F:00436DBB 83E103              AND      ECX,BYTE +03

017F:00436DBE F3A4                REP MOVSB

017F:00436DC0 8B4C2418            MOV      ECX,[ESP+18]

017F:00436DC4 51                  PUSH     ECX

017F:00436DC5 E8D7450200          CALL     0045B3A1

017F:00436DCA 8D54243C            LEA      EDX,[ESP+3C]

017F:00436DCE 89442420            MOV      [ESP+20],EAX

017F:00436DD2 52                  PUSH     EDX

017F:00436DD3 E878190100          CALL     00448750

017F:00436DD8 83C410              ADD      ESP,BYTE +10

017F:00436DDB 85C0                TEST     EAX,EAX

017F:00436DDD 7507                JNZ      00436DE6

017F:00436DDF 32DB                XOR      BL,BL

017F:00436DE1 E998000000          JMP      00436E7E

017F:00436DE6 8D44241C            LEA      EAX,[ESP+1C]

017F:00436DEA 6A02                PUSH     BYTE +02

017F:00436DEC 8D4C2434            LEA      ECX,[ESP+34]

017F:00436DF0 50                  PUSH     EAX

017F:00436DF1 51                  PUSH     ECX

017F:00436DF2 E869190100          CALL     00448760

017F:00436DF7 83C40C              ADD      ESP,BYTE +0C

017F:00436DFA 85C0                TEST     EAX,EAX

017F:00436DFC 7504                JNZ      00436E02

017F:00436DFE 32DB                XOR      BL,BL

017F:00436E00 EB7C                JMP      SHORT 00436E7E

017F:00436E02 8D542414            LEA      EDX,[ESP+14]

017F:00436E06 6A02                PUSH     BYTE +02

017F:00436E08 8D442434            LEA      EAX,[ESP+34]

017F:00436E0C 52                  PUSH     EDX

017F:00436E0D 50                  PUSH     EAX

017F:00436E0E E84D190100          CALL     00448760

017F:00436E13 83C40C              ADD      ESP,BYTE +0C

017F:00436E16 85C0                TEST     EAX,EAX

017F:00436E18 7504                JNZ      00436E1E

017F:00436E1A 32DB                XOR      BL,BL

017F:00436E1C EB60                JMP      SHORT 00436E7E

017F:00436E1E 8D4C2420            LEA      ECX,[ESP+20]

017F:00436E22 8D542430            LEA      EDX,[ESP+30]

017F:00436E26 51                  PUSH     ECX

017F:00436E27 52                  PUSH     EDX

017F:00436E28 E8131A0100          CALL     00448840－－`call(4)`，这个调用检验注册码的前八位是否正确

017F:00436E2D 83C408              ADD      ESP,BYTE +08

017F:00436E30 85C0                TEST     EAX,EAX

017F:00436E32 7509                JNZ      00436E3D－－－－正确就跳

017F:00436E34 32DB                XOR      BL,BL

017F:00436E36 C644240F0A          MOV      BYTE [ESP+0F],0A

017F:00436E3B EB41                JMP      SHORT 00436E7E

017F:00436E3D 8B442478            MOV      EAX,[ESP+78]－－注册码地址入 `eax`，此时注册码第 15 位已置`\0`

017F:00436E41 8378F80E            CMP      DWORD [EAX-08],BYTE +0E

017F:00436E45 7C30                JL       00436E77－－若注册码长度为 18 位，这里就不跳

017F:00436E47 83C00C              ADD      EAX,BYTE +0C

017F:00436E4A 6A02                PUSH     BYTE +02

017F:00436E4C 50                  PUSH     EAX

017F:00436E4D E85E190100          CALL     004487B0－－－`call(5)`，取得注册码的 15，16 位，`eax` 返回 15，16 位组成的十六进制值

017F:00436E52 83C408              ADD      ESP,BYTE +08

017F:00436E55 83F841              CMP      EAX,BYTE +41－－－比较上面的返回值是否大于等于 41h

017F:00436E58 7304                JNC      00436E5E－－－大于等于 41h 就跳

017F:00436E5A 33C0                XOR      EAX,EAX

017F:00436E5C EB08                JMP      SHORT 00436E66

017F:00436E5E 83E841              SUB      EAX,BYTE +41－－`eax` = `eax` - 41h

017F:00436E61 83F830              CMP      EAX,BYTE +30－－比较 `eax` 值是否还大于等于 30h

017F:00436E64 7303                JNC      00436E69－－－大于等于 30h 就跳（这里跳就注册成功了）

017F:00436E66 83C00D              ADD      EAX,BYTE +0D－－`eax` = `eax` + 0Dh

017F:00436E69 83F82E              CMP      EAX,BYTE +2E－－－比较 `eax` 值是否大于等于 2Eh

017F:00436E6C 7310                JNC      00436E7E－－－大于等于 2Eh 就跳（这里跳就注册成功了），总起来说，注册码 15，16 两位构成的十六进制值只要大于等于 62h 就可以了。

017F:00436E6E 32DB                XOR      BL,BL

017F:00436E70 C644240F0B          MOV      BYTE [ESP+0F],0B

017F:00436E75 EB07                JMP      SHORT 00436E7E



### 汇编代码段

```
017F:00436E77 32DB             XOR      BL,BL

017F:00436E79 C644240F0C       MOV      BYTE [ESP+0F],0C

017F:00436E7E 33C0             XOR      EAX,EAX

017F:00436E80 8D4C2410         LEA      ECX,[ESP+10]

017F:00436E84 8A64240F         MOV      AH,[ESP+0F]

017F:00436E88 C644246C00       MOV      BYTE [ESP+6C],00

017F:00436E8D 8AC3             MOV      AL,BL

017F:00436E8F 8BF0             MOV      ESI,EAX

017F:00436E91 E87BD90600       CALL     004A4811

017F:00436E96 8D4C2478         LEA      ECX,[ESP+78]

017F:00436E9A C744246CFFFFFFFF MOV      DWORD [ESP+6C],FFFFFFFF

017F:00436EA2 E86AD90600       CALL     004A4811

017F:00436EA7 8B4C2464         MOV      ECX,[ESP+64]

017F:00436EAB 668BC6           MOV      AX,SI

017F:00436EAE 5F               POP      EDI

017F:00436EAF 5E               POP      ESI

017F:00436EB0 5B               POP      EBX

017F:00436EB1 64890D00000000   MOV      [FS:00],ECX
```

### call（4）

```
017F:00448840 83EC7C           SUB      ESP,BYTE +7C
017F:00448843 33C0             XOR      EAX,EAX——eax 清零
017F:00448845 B930000000       MOV      ECX,30——ecx＝30
017F:0044884A 8BD0             MOV      EDX,EAX——edx 初值为零，每次递增 1
017F:0044884C 81E2FFFF0000     AND      EDX,FFFF
017F:00448852 40               INC      EAX——eax＝eax＋1
017F:00448853 884C1404         MOV      [ESP+EDX+04],CL——cl 的值送指定地址，即在那里产生 0－9 的 ASCII 码
017F:00448857 41               INC      ECX
017F:00448858 6683F939         CMP      CX,BYTE +39——是否产生完毕？
017F:0044885C 76EC             JNA      0044884A————这个循环产生“123456789”
017F:0044885E B941000000       MOV      ECX,41
017F:00448863 8BD0             MOV      EDX,EAX
017F:00448865 81E2FFFF0000     AND      EDX,FFFF
017F:0044886B 40               INC      EAX
017F:0044886C 884C1404         MOV      [ESP+EDX+04],CL
017F:00448870 41               INC      ECX
017F:00448871 6683F946         CMP      CX,BYTE +46
017F:00448875 76EC             JNA      00448863——同上，这个循环产生“ABCDEF”
017F:00448877 53               PUSH     EBX
017F:00448878 8B9C2488000000   MOV      EBX,[ESP+88]
017F:0044887F 56               PUSH     ESI
017F:00448880 89442408         MOV      [ESP+08],EAX
017F:00448884 57               PUSH     EDI
017F:00448885 8D430C           LEA      EAX,[EBX+0C]
017F:00448888 6A02             PUSH     BYTE +02
017F:0044888A 50               PUSH     EAX
017F:0044888B E820FFFFFF       CALL     004487B0
017F:00448890 8BB42494000000   MOV      ESI,[ESP+94]
017F:00448897 8D4C2414         LEA      ECX,[ESP+14]
017F:0044889B 6A02             PUSH     BYTE +02
017F:0044889D 51               PUSH     ECX
017F:0044889E 56               PUSH     ESI
017F:0044889F 89442420         MOV      [ESP+20],EAX
017F:004488A3 E8B8FeFFFF       CALL     00448760
017F:004488A8 8D5308           LEA      EDX,[EBX+08]
017F:004488AB 6A04             PUSH     BYTE +04
017F:004488AD 52               PUSH     EDX
017F:004488AE E8FDFeFFFF       CALL     004487B0
017F:004488B3 89442428         MOV      [ESP+28],EAX
017F:004488B7 8D442428         LEA      EAX,[ESP+28]
017F:004488BB 6A02             PUSH     BYTE +02
017F:004488BD 50               PUSH     EAX
017F:004488BE 56               PUSH     ESI
017F:004488BF E89CFEFFFF       CALL     00448760
017F:004488C4 8D4C2458         LEA      ECX,[ESP+58]
017F:004488C8 51               PUSH     ECX
017F:004488C9 E882000000       CALL     00448950
017F:004488CE 33D2             XOR      EDX,EDX
017F:004488D0 8D7E02           LEA      EDI,[ESI+02]
017F:004488D3 668B16           MOV      DX,[ESI]
017F:004488D6 8D44245C         LEA      EAX,[ESP+5C]
017F:004488DA 52               PUSH     EDX
017F:004488DB 57               PUSH     EDI
017F:004488DC 50               PUSH     EAX
017F:004488DD E89E000000       CALL     00448980
017F:004488E2 8D4C2468         LEA      ECX,[ESP+68]
017F:004488E6 8D542458         LEA      EDX,[ESP+58]
017F:004488EA 51               PUSH     ECX
017F:004488EB 52               PUSH     EDX
017F:004488EC E84F010000       CALL     00448A40——call（6），这个 call 产生下面要用到的密码表
017F:004488F1 B90C000000       MOV      ECX,0C
017F:004488F6 33C0             XOR      EAX,EAX
017F:004488F8 F3AB             REP STOSD
017F:004488FA 66AB             STOSW
017F:004488FC 83C440           ADD      ESP,BYTE +40
017F:004488FF 33FF             XOR      EDI,EDI——edi 清零
017F:00448901 8BF7             MOV      ESI,EDI——esi 初值为零，以 2 为递增量
017F:00448903 81E6FFFF0000     AND      ESI,FFFF
017F:00448909 8BC6             MOV      EAX,ESI——eax＝esi
017F:0044890B D1E8             SHR      EAX,1——eax＝eax/2
017F:0044890D 8A0C18           MOV      CL,[EAX+EBX]——依次取得注册码前八位得 ASCII 码
017F:00448910 51               PUSH     ECX——ASCII 码入栈
017F:00448911 E80AFFFFFF       CALL     00448820——call（7），对 ASCII 码进行处理
017F:00448916 8A543424         MOV      DL,[ESP+ESI+24]——从密码表（长度：10h 字节）中取出相应的值送入 dl
017F:0044891A 25FF000000       AND      EAX,FF
017F:0044891F 83E20F           AND      EDX,BYTE +0F——取 dl 的右半个字节
017F:00448922 83C404           ADD      ESP,BYTE +04
017F:00448925 0FBE4C1410       MOVSX    ECX,BYTE [ESP+EDX+10]——从“123456789ABCDEF”中取出对应字符的 ASCII 码送入 ecx 中
017F:0044892A 3BC8             CMP      ECX,EAX——与假注册码中对应位置的字符比较
017F:0044892C 7515             JNZ      00448943——不一样就跳走
017F:0044892E 83C702           ADD      EDI,BYTE +02——edi 为计数器，递增量为 2
017F:00448931 6683FF10         CMP      DI,BYTE +10——直到 edi＝10h，即取前八个字符
017F:00448935 72CA             JC       00448901
017F:00448937 5F               POP      EDI
017F:00448938 5E               POP      ESI
017F:00448939 B801000000       MOV      EAX,01
017F:0044893E 5B               POP      EBX
017F:0044893F 83C47C           ADD      ESP,BYTE +7C
017F:00448942 C3               RET
017F:00448943 5F               POP      EDI
017F:00448944 5E               POP      ESI
017F:00448945 33C0             XOR      EAX,EAX
017F:00448947 5B               POP      EBX
017F:00448948 83C47C           ADD      ESP,BYTE +7C
```

### call（5）

```
017F:004487BD 6685ED           TEST     BP,BP
017F:004487C0 764F             JNA      00448811
017F:004487C2 8B5C2414         MOV      EBX,[ESP+14]
017F:004487C6 8BC6             MOV      EAX,ESI
017F:004487C8 25FFFF0000       AND      EAX,FFFF
017F:004487CD 8A0C18           MOV      CL,[EAX+EBX]
017F:004487D0 51               PUSH     ECX
017F:004487D1 E84A000000       CALL     00448820——又是 call（7），分别对 15，16 位注册码的 ASCII 码进行处理
017F:004487D6 660FB6C0         MOVZX    AX,AL
017F:004487DA 83C404           ADD      ESP,BYTE +04
017F:004487DD 663D3000         CMP      AX,30
017F:004487E1 720D             JC       004487F0
017F:004487E3 663D3900         CMP      AX,39
017F:004487E7 7707             JA       004487F0
017F:004487E9 05D0FF0000       ADD      EAX,FFD0
017F:004487EE EB11             JMP      SHORT 00448801
017F:004487F0 663D4100         CMP      AX,41
017F:004487F4 7222             JC       00448818
017F:004487F6 663D4600         CMP      AX,46
017F:004487FA 771C             JA       00448818
017F:004487FC 05C9FF0000       ADD      EAX,FFC9
017F:00448801 C1E704           SHL      EDI,04
017F:00448804 25FFFF0000       AND      EAX,FFFF
017F:00448809 0BF8             OR       EDI,EAX
017F:0044880B 46               INC      ESI
017F:0044880C 663BF5           CMP      SI,BP
```



`017F:0044880F 72B5` `JC` `004487C6`

`017F:00448811 8BC7` `MOV` `EAX,EDI`

`017F:00448813 5F` `POP` `EDI`

`017F:00448814 5E` `POP` `ESI`

`017F:00448815 5D` `POP` `EBP`

`017F:00448816 5B` `POP` `EBX`

这个 `call` 的功能就是取得注册码第 15，16 位两个数字构成的 16 进制值，并放到 `eax` 中

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

`call(6)`：

`017F:00448A40 83EC08` `SUB` `ESP,BYTE +08`

`017F:00448A43 8D442400` `LEA` `EAX,[ESP+00]`

`017F:00448A47 56` `PUSH` `ESI`

`017F:00448A48 57` `PUSH` `EDI`

`017F:00448A49 8B7C2418` `MOV` `EDI,[ESP+18]`

`017F:00448A4D 6A08` `PUSH` `BYTE +08`

`017F:00448A4F 8D7710` `LEA` `ESI,[EDI+10]`

`017F:00448A52 56` `PUSH` `ESI`

`017F:00448A53 50` `PUSH` `EAX`

`017F:00448A54 E8C70A0000` `CALL` `00449520`

`017F:00448A59 8B06` `MOV` `EAX,[ESI]`

`017F:00448A5B 83C40C` `ADD` `ESP,BYTE +0C`

`017F:00448A5E C1E803` `SHR` `EAX,03`

`017F:00448A61 83E03F` `AND` `EAX,BYTE +3F`

`017F:00448A64 B938000000` `MOV` `ECX,38`

`017F:00448A69 83F838` `CMP` `EAX,BYTE +38`

`017F:00448A6C 7205` `JC` `00448A73`

`017F:00448A6E B978000000` `MOV` `ECX,78`

`017F:00448A73 2BC8` `SUB` `ECX,EAX`

`017F:00448A75 51` `PUSH` `ECX`

`017F:00448A76 68F8D44E00` `PUSH` `DWORD 004ED4F8`

`017F:00448A7B 57` `PUSH` `EDI`

`017F:00448A7C E8FFFEFFFF` `CALL` `00448980`

`017F:00448A81 8D4C2414` `LEA` `ECX,[ESP+14]`

`017F:00448A85 6A08` `PUSH` `BYTE +08`

`017F:00448A87 51` `PUSH` `ECX`

`017F:00448A88 57` `PUSH` `EDI`

`017F:00448A89 E8F2FEFFFF` `CALL` `00448980`－－－密码表在这个 `call` 中产生

`017F:00448A8E 8B54242C` `MOV` `EDX,[ESP+2C]`

`017F:00448A92 6A10` `PUSH` `BYTE +10`

`017F:00448A94 57` `PUSH` `EDI`

`017F:00448A95 52` `PUSH` `EDX`

`017F:00448A96 E8850A0000` `CALL` `00449520`

`017F:00448A9B 83C424` `ADD` `ESP,BYTE +24`

`017F:00448A9E B916000000` `MOV` `ECX,16`

`017F:00448AA3 33C0` `XOR` `EAX,EAX`

`017F:00448AA5 F3AB` `REP STOSD`

`017F:00448AA7 5F` `POP` `EDI`

`017F:00448AA8 5E` `POP` `ESI`

`017F:00448AA9 83C408` `ADD` `ESP,BYTE +08`

`017F:00448AAC C3` `RET`

这个过程比较复杂，我没空分析了，有兴趣的朋友可以跟进去看一看。只要这里搞明白就可以写出注册机了。

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

`call（7）`：

`017F:00448820 8B442404` `MOV` `EAX,[ESP+04]`－－－`eax` 取得当前字符的 `ASCII` 码

`017F:00448824 3C61` `CMP` `AL,61`

`017F:00448826 720D` `JC` `00448835`－－－－小于 `61h` 就跳

`017F:00448828 3C7A` `CMP` `AL,7A`

`017F:0044882A 7709` `JA` `00448835`

`017F:0044882C 25FF000000` `AND` `EAX,FF`

`017F:00448831 83E820` `SUB` `EAX,BYTE +20`

`017F:00448834 C3` `RET`

`017F:00448835 25FF000000` `AND` `EAX,FF`

这个 `call` 其实没什么用，因为能走到这里的注册码，其字符应该都是“123456789ABCDEF－”之一，

这些在 `call（2）` 中就已经检验过了。

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

这是我今年寒假最后一篇破文了，追这个东西我花了不少时间，希望看懂的朋友帮忙顶一下，没看懂的也可以把问题提出来，可能某些地方还写得不清楚,我会尽力解答。

最后感谢看完拙文！

cyclotron

2003.2.11 4dd466d0e1e0cc123afbe504da2462ca
