
# 2nd Speech Center V3.30.7.1129 算法分析

> 原文：[`www.pediy.com/kssd/pediy10/73542.html`](https://www.pediy.com/kssd/pediy10/73542.html)

【破文标题】2nd Speech Center V3.30.7.1129 算法分析

【破文作者】tianxj

【作者邮箱】`tianxj_2007@126.com`

【作者主页】`WwW.ChiNaPYG.CoM`

【破解工具】`PEiD`, `OD`

【破解平台】`Windows XP`

【软件名称】2nd Speech Center V3.30.7.1129

【软件大小】4.21 MB

【软件语言】英文

【软件类别】国外软件 / 共享软件 / 电子阅读

【更新时间】2007-12-07

【原版下载】自己找一下

【保护方式】注册码

【软件简介】将任何文本转换成声音或 `MP3/WAVE` 音频文件的工具。2nd Speech Center 是一款率获殊荣的文本到声音的播放器，可以让你听文档，`email` 或网页，而无需再去阅读。除了文本，2nd Speech Center 也支持将文本转换成声音文件以备后用。2nd Speech Center 支持超过 11 种语言，几十种男女声音及机器合成发声。2nd Speech Center 会自动监测 `win` 的剪贴板并处理其中的内容。此外，2nd Speech Center 集成了许多有用的功能，诸如闹铃，事务提醒和强大的命令行。无需担心，直观的 `VCR` 风格界面绝对简单易用。

【破解声明】我是一只小菜鸟，偶得一点心得，愿与大家分享：）

--------------------------------------------------------------

## 破解内容

--------------------------------------------------------------

### 一、运行程序，进行注册，输入错误的注册信息进行检测，有提示信息

`"Registered Code Error!"`

### 二、用 PEiD 对 iisc.exe 查壳，为 ASPack 2.12 -> Alexey Solodovnikov

用 `PEiD` 插件脱壳后，为 `Borland Delphi 6.0 - 7.0`

### 三、运行 OD，打开 iisc.exe.unpacked_.exe，右键超级字串参考查找 ASCII.

==============================================================

代码:
```



```
00601C58  /.  55                PUSH    EBP
00601C59  |.  8BEC              MOV     EBP, ESP
00601C5B  |.  33C9              XOR     ECX, ECX
00601C5D  |.  51                PUSH    ECX
00601C5E  |.  51                PUSH    ECX
00601C5F  |.  51                PUSH    ECX
00601C60  |.  51                PUSH    ECX
00601C61  |.  51                PUSH    ECX
00601C62  |.  51                PUSH    ECX
00601C63  |.  51                PUSH    ECX
00601C64  |.  53                PUSH    EBX
00601C65  |.  8BD8              MOV     EBX, EAX
00601C67  |.  33C0              XOR     EAX, EAX
00601C69  |.  55                PUSH    EBP
00601C6A  |.  68 6B1D6000       PUSH    iisc_exe.00601D6B
00601C6F  |.  64:FF30           PUSH    DWORD PTR FS:[EAX]
00601C72  |.  64:8920           MOV     DWORD PTR FS:[EAX], ESP
00601C75  |.  8D55 F4           LEA     EDX, DWORD PTR [EBP-C]
00601C78  |.  8B83 1C030000     MOV     EAX, DWORD PTR [EBX+31C]
00601C7E  |.  E8 3940EFFF       CALL    iisc_exe.004F5CBC
00601C83  |.  8B45 F4           MOV     EAX, DWORD PTR [EBP-C]
00601C86  |.  8D55 FC           LEA     EDX, DWORD PTR [EBP-4]
00601C89  |.  E8 1E7DE0FF       CALL    iisc_exe.004099AC
00601C8E  |.  8D55 F0           LEA     EDX, DWORD PTR [EBP-10]
00601C91  |.  8B83 20030000     MOV     EAX, DWORD PTR [EBX+320]
00601C97  |.  E8 2040EFFF       CALL    iisc_exe.004F5CBC
00601C9C  |.  8B45 F0           MOV     EAX, DWORD PTR [EBP-10]
00601C9F  |.  8D55 F8           LEA     EDX, DWORD PTR [EBP-8]
00601CA2  |.  E8 057DE0FF       CALL    iisc_exe.004099AC
00601CA7  |.  837D FC 00        CMP     DWORD PTR [EBP-4], 0
00601CAB  |.  0F84 80000000     JE      iisc_exe.00601D31                ;  //用户名为空则跳
00601CB1  |.  837D F8 00        CMP     DWORD PTR [EBP-8], 0
00601CB5  |.  74 7A             JE      SHORT iisc_exe.00601D31          ;  //注册码为空则跳
00601CB7  |.  8D45 EC           LEA     EAX, DWORD PTR [EBP-14]
00601CBA  |.  50                PUSH    EAX
00601CBB  |.  B9 801D6000       MOV     ECX, iisc_exe.00601D80           ;  3
00601CC0  |.  BA 8C1D6000       MOV     EDX, iisc_exe.00601D8C           ;  2nd Speech Center
00601CC5  |.  8B45 FC           MOV     EAX, DWORD PTR [EBP-4]           ;  //用户名
00601CC8  |.  E8 4BFAFFFF       CALL    iisc_exe.00601718                ;  //算法 CALL
00601CCD  |.  8B55 EC           MOV     EDX, DWORD PTR [EBP-14]          ;  //真码
00601CD0  |.  8B45 F8           MOV     EAX, DWORD PTR [EBP-8]           ;  //假码
00601CD3  |.  E8 E033E0FF       CALL    iisc_exe.004050B8                ;  //比较 CALL
00601CD8  |.  75 57             JNZ     SHORT iisc_exe.00601D31          ;  //关键跳转
00601CDA  |.  C605 40BD6100 01  MOV     BYTE PTR [61BD40], 1
00601CE1  |.  8B4D FC           MOV     ECX, DWORD PTR [EBP-4]
00601CE4  |.  BA A81D6000       MOV     EDX, iisc_exe.00601DA8           ;  RegName
00601CE9  |.  B8 B81D6000       MOV     EAX, iisc_exe.00601DB8           ;  \Software\Microsoft\02k\JJTD4
00601CEE  |.  E8 5946EAFF       CALL    iisc_exe.004A634C
00601CF3  |.  8B45 FC           MOV     EAX, DWORD PTR [EBP-4]
00601CF6  |.  8945 E4           MOV     DWORD PTR [EBP-1C], EAX
00601CF9  |.  C645 E8 0B        MOV     BYTE PTR [EBP-18], 0B
00601CFD  |.  8D55 E4           LEA     EDX, DWORD PTR [EBP-1C]
00601D00  |.  33C9              XOR     ECX, ECX
00601D02  |.  B8 E01D6000       MOV     EAX, iisc_exe.00601DE0           ;  Congratulations! \r\nThis software is licensed to %s.\r\nThanks for your supporting.
00601D07  |.  E8 5841EAFF       CALL    iisc_exe.004A5E64
00601D0C  |.  8BC3              MOV     EAX, EBX
00601D0E  |.  E8 81DBE8FF       CALL    iisc_exe.0048F894
00601D13  |.  A1 8C866100       MOV     EAX, DWORD PTR [61868C]
00601D18  |.  8338 00           CMP     DWORD PTR [EAX], 0
00601D1B  |.  74 1E             JE      SHORT iisc_exe.00601D3B
00601D1D  |.  A1 8C866100       MOV     EAX, DWORD PTR [61868C]
00601D22  |.  8B00              MOV     EAX, DWORD PTR [EAX]
00601D24  |.  8A15 40BD6100     MOV     DL, BYTE PTR [61BD40]
00601D2A  |.  E8 C1A20000       CALL    iisc_exe.0060BFF0
00601D2F  |.  EB 0A             JMP     SHORT iisc_exe.00601D3B
00601D31  |>  B8 3C1E6000       MOV     EAX, iisc_exe.00601E3C           ;  Registered Code Error!
00601D36  |.  E8 F941EAFF       CALL    iisc_exe.004A5F34
00601D3B  |>  33C0              XOR     EAX, EAX
00601D3D  |.  5A                POP     EDX
00601D3E  |.  59                POP     ECX
00601D3F  |.  59                POP     ECX
00601D40  |.  64:8910           MOV     DWORD PTR FS:[EAX], EDX
00601D43  |.  68 721D6000       PUSH    iisc_exe.00601D72
00601D48  |>  8D45 EC           LEA     EAX, DWORD PTR [EBP-14]
00601D4B  |.  E8 4C2FE0FF       CALL    iisc_exe.00404C9C
00601D50  |.  8D45 F0           LEA     EAX, DWORD PTR [EBP-10]
00601D53  |.  BA 02000000       MOV     EDX, 2
00601D58  |.  E8 632FE0FF       CALL    iisc_exe.00404CC0
00601D5D  |.  8D45 F8           LEA     EAX, DWORD PTR [EBP-8]
00601D60  |.  BA 02000000       MOV     EDX, 2
00601D65  |.  E8 562FE0FF       CALL    iisc_exe.00404CC0
00601D6A  \\.  C3                RETN
00601D6B   .^ E9 4828E0FF       JMP     iisc_exe.004045B8
00601D70   .^ EB D6             JMP     SHORT iisc_exe.00601D48
00601D72   .  5B                POP     EBX
00601D73   .  8BE5              MOV     ESP, EBP
00601D75   .  5D                POP     EBP
00601D76   .  C3                RETN
\==============================================================
00601718   $  55                PUSH    EBP
00601719   .  8BEC              MOV     EBP, ESP
0060171B   .  51                PUSH    ECX
0060171C   .  B9 06000000       MOV     ECX, 6
00601721   >  6A 00             PUSH    0
00601723   .  6A 00             PUSH    0
00601725   .  49                DEC     ECX
00601726   .^ 75 F9             JNZ     SHORT iisc_exe.00601721
00601728   .  874D FC           XCHG    DWORD PTR [EBP-4], ECX
0060172B   .  53                PUSH    EBX
0060172C   .  56                PUSH    ESI
0060172D   .  57                PUSH    EDI
0060172E   .  894D F4           MOV     DWORD PTR [EBP-C], ECX
00601731   .  8955 F8           MOV     DWORD PTR [EBP-8], EDX
00601734   .  8945 FC           MOV     DWORD PTR [EBP-4], EAX
00601737   .  8B45 FC           MOV     EAX, DWORD PTR [EBP-4]
0060173A   .  E8 1D3AE0FF       CALL    iisc_exe.0040515C
0060173F   .  8B45 F8           MOV     EAX, DWORD PTR [EBP-8]
00601742   .  E8 153AE0FF       CALL    iisc_exe.0040515C
00601747   .  8B45 F4           MOV     EAX, DWORD PTR [EBP-C]
0060174A   .  E8 0D3AE0FF       CALL    iisc_exe.0040515C
0060174F   .  33C0              XOR     EAX, EAX
00601751   .  55                PUSH    EBP
00601752   .  68 8E186000       PUSH    iisc_exe.0060188E
00601757   .  64:FF30           PUSH    DWORD PTR FS:[EAX]
0060175A   .  64:8920           MOV     DWORD PTR FS:[EAX], ESP
0060175D   .  33C0              XOR     EAX, EAX
0060175F   .  55                PUSH    EBP
00601760   .  68 37186000       PUSH    iisc_exe.00601837
00601765   .  64:FF30           PUSH    DWORD PTR FS:[EAX]
00601768   .  64:8920           MOV     DWORD PTR FS:[EAX], ESP
0060176B   .  8D55 E8           LEA     EDX, DWORD PTR [EBP-18]
0060176E   .  8B45 F8           MOV     EAX, DWORD PTR [EBP-8]
00601771   .  E8 3682E0FF       CALL    iisc_exe.004099AC
00601776   .  8B45 E8           MOV     EAX, DWORD PTR [EBP-18]          ;  //字符串"2nd Speech Center"
00601779   .  B1 01             MOV     CL, 1                             ;  //CL=1
0060177B   .  BA 6E740300       MOV     EDX, 3746E                        ;  //EDX=3746E
00601780   .  E8 87FEFFFF       CALL    iisc_exe.0060160C                ;  //运算 CALL
00601785   .  8BD8              MOV     EBX, EAX                          ;  //EBX=EAX=27487h
00601787   .  8D55 E4           LEA     EDX, DWORD PTR [EBP-1C]
0060178A   .  8B45 F4           MOV     EAX, DWORD PTR [EBP-C]
0060178D   .  E8 1A82E0FF       CALL    iisc_exe.004099AC
00601792   .  8B45 E4           MOV     EAX, DWORD PTR [EBP-1C]          ;  //字符串"3"
00601795   .  33C9              XOR     ECX, ECX                          ;  //CL=0
00601797   .  BA 01EB0B00       MOV     EDX, 0BEB01                       ;  //EDX=0BEB01
0060179C   .  E8 6BFEFFFF       CALL    iisc_exe.0060160C                ;  //运算 CALL
006017A1   .  8BF0              MOV     ESI, EAX                          ;  //ESI=EAX=23B52h
006017A3   .  8D55 E0           LEA     EDX, DWORD PTR [EBP-20]
006017A6   .  8B45 FC           MOV     EAX, DWORD PTR [EBP-4]
006017A9   .  E8 FE81E0FF       CALL    iisc_exe.004099AC
006017AE   .  8B45 E0           MOV     EAX, DWORD PTR [EBP-20]          ;  //用户名
006017B1   .  B1 01             MOV     CL, 1                             ;  //CL=1
006017B3   .  8BD3              MOV     EDX, EBX                          ;  //EDX=EBX
006017B5   .  E8 52FEFFFF       CALL    iisc_exe.0060160C                ;  //运算 CALL
006017BA   .  8BD8              MOV     EBX, EAX                          ;  //EBX=EAX
006017BC   .  8D55 DC           LEA     EDX, DWORD PTR [EBP-24]
006017BF   .  8B45 FC           MOV     EAX, DWORD PTR [EBP-4]
006017C2   .  E8 E581E0FF       CALL    iisc_exe.004099AC
006017C7   .  8B45 DC           MOV     EAX, DWORD PTR [EBP-24]          ;  //用户名
006017CA   .  33C9              XOR     ECX, ECX                          ;  //CL=0
006017CC   .  8BD6              MOV     EDX, ESI                          ;  //EDX=ESI
006017CE   .  E8 39FEFFFF       CALL    iisc_exe.0060160C                ;  //运算 CALL
006017D3   .  8BF0              MOV     ESI, EAX                          ;  //ESI=EAX
006017D5   .  8D45 F0           LEA     EAX, DWORD PTR [EBP-10]
006017D8   .  50                PUSH    EAX
006017D9   .  8D55 D8           LEA     EDX, DWORD PTR [EBP-28]
006017DC   .  8BC3              MOV     EAX, EBX
006017DE   .  E8 2586E0FF       CALL    iisc_exe.00409E08                ;  //将 EAX 转 10 进制字符形式
006017E3   .  8B45 D8           MOV     EAX, DWORD PTR [EBP-28]
006017E6   .  B9 05000000       MOV     ECX, 5
006017EB   .  BA 01000000       MOV     EDX, 1
006017F0   .  E8 D739E0FF       CALL    iisc_exe.004051CC                ;  //取 1-5 位
006017F5   .  8D45 EC           LEA     EAX, DWORD PTR [EBP-14]
006017F8   .  50                PUSH    EAX
006017F9   .  8D55 D4           LEA     EDX, DWORD PTR [EBP-2C]
006017FC   .  8BC6              MOV     EAX, ESI
006017FE   .  E8 0586E0FF       CALL    iisc_exe.00409E08                ;  //将 EAX 转 10 进制字符形式
00601803   .  8B45 D4           MOV     EAX, DWORD PTR [EBP-2C]
00601806   .  B9 05000000       MOV     ECX, 5
0060180B   .  BA 02000000       MOV     EDX, 2
00601810   .  E8 B739E0FF       CALL    iisc_exe.004051CC                ;  //取 2-6 位
00601815   .  FF75 EC           PUSH    DWORD PTR [EBP-14]
00601818   .  68 A8186000       PUSH    iisc_exe.006018A8                ;  -
0060181D   .  FF75 F0           PUSH    DWORD PTR [EBP-10]
00601820   .  8B45 08           MOV     EAX, DWORD PTR [EBP+8]
00601823   .  BA 03000000       MOV     EDX, 3
00601828   .  E8 FF37E0FF       CALL    iisc_exe.0040502C                ;  //相连
0060182D   .  33C0              XOR     EAX, EAX
0060182F   .  5A                POP     EDX
00601830   .  59                POP     ECX
00601831   .  59                POP     ECX
00601832   .  64:8910           MOV     DWORD PTR FS:[EAX], EDX
00601835   .  EB 3C             JMP     SHORT iisc_exe.00601873
00601837   .^ E9 C82AE0FF       JMP     iisc_exe.00404304
0060183C   .  8D55 D0           LEA     EDX, DWORD PTR [EBP-30]
0060183F   .  B8 6E740300       MOV     EAX, 3746E
00601844   .  E8 BF85E0FF       CALL    iisc_exe.00409E08
00601849   .  FF75 D0           PUSH    DWORD PTR [EBP-30]
0060184C   .  68 A8186000       PUSH    iisc_exe.006018A8                ;  -
00601851   .  8D55 CC           LEA     EDX, DWORD PTR [EBP-34]
00601854   .  B8 01EB0B00       MOV     EAX, 0BEB01
00601859   .  E8 AA85E0FF       CALL    iisc_exe.00409E08
0060185E   .  FF75 CC           PUSH    DWORD PTR [EBP-34]
00601861   .  8B45 08           MOV     EAX, DWORD PTR [EBP+8]
00601864   .  BA 03000000       MOV     EDX, 3
00601869   .  E8 BE37E0FF       CALL    iisc_exe.0040502C
0060186E   .  E8 F92DE0FF       CALL    iisc_exe.0040466C
00601873   >  33C0              XOR     EAX, EAX
00601875   .  5A                POP     EDX
00601876   .  59                POP     ECX
00601877   .  59                POP     ECX
00601878   .  64:8910           MOV     DWORD PTR FS:[EAX], EDX
0060187B   .  68 95186000       PUSH    iisc_exe.00601895
00601880   >  8D45 CC           LEA     EAX, DWORD PTR [EBP-34]
00601883   .  BA 0D000000       MOV     EDX, 0D
00601888   .  E8 3334E0FF       CALL    iisc_exe.00404CC0
0060188D   .  C3                RETN
0060188E   .^ E9 252DE0FF       JMP     iisc_exe.004045B8
00601893   .^ EB EB             JMP     SHORT iisc_exe.00601880
00601895   .  5F                POP     EDI
00601896   .  5E                POP     ESI
00601897   .  5B                POP     EBX
00601898   .  8BE5              MOV     ESP, EBP
0060189A   .  5D                POP     EBP
0060189B   .  C2 0400           RETN    4
\==============================================================
0060160C   $  55                PUSH    EBP
0060160D   .  8BEC              MOV     EBP, ESP
0060160F   .  83C4 F4           ADD     ESP, -0C
00601612   .  53                PUSH    EBX
00601613   .  56                PUSH    ESI
00601614   .  57                PUSH    EDI
00601615   .  884D F7           MOV     BYTE PTR [EBP-9], CL              ;  //[EBP-9]=CL
00601618   .  8955 F8           MOV     DWORD PTR [EBP-8], EDX            ;  //[EBP-8]=EDX
0060161B   .  8945 FC           MOV     DWORD PTR [EBP-4], EAX
0060161E   .  8B45 FC           MOV     EAX, DWORD PTR [EBP-4]            ;  //字符串"2nd Speech Center","3",用户名
00601621   .  E8 363BE0FF       CALL    iisc_exe.0040515C
00601626   .  33C0              XOR     EAX, EAX
00601628   .  55                PUSH    EBP
00601629   .  68 08176000       PUSH    iisc_exe.00601708
0060162E   .  64:FF30           PUSH    DWORD PTR FS:[EAX]
00601631   .  64:8920           MOV     DWORD PTR FS:[EAX], ESP
00601634   .  33C0              XOR     EAX, EAX
00601636   .  55                PUSH    EBP
00601637   .  68 D6166000       PUSH    iisc_exe.006016D6
0060163C   .  64:FF30           PUSH    DWORD PTR FS:[EAX]
0060163F   .  64:8920           MOV     DWORD PTR FS:[EAX], ESP
00601642   .  33DB              XOR     EBX, EBX                          ;  //EBX=0
00601644   .  837D FC 00        CMP     DWORD PTR [EBP-4], 0
00601648   .  74 70             JE      SHORT iisc_exe.006016BA
0060164A   .  8B45 FC           MOV     EAX, DWORD PTR [EBP-4]            ;  //字符串
0060164D   .  E8 1A39E0FF       CALL    iisc_exe.00404F6C                ;  //取字符串长度
00601652   .  8BF0              MOV     ESI, EAX                          ;  //ESI=EAX=字符串长度
00601654   .  85F6              TEST    ESI, ESI
00601656   .  7E 3B             JLE     SHORT iisc_exe.00601693
00601658   .  B9 01000000       MOV     ECX, 1                            ;  //ECX=1
0060165D   >  807D F7 00        CMP     BYTE PTR [EBP-9], 0               ;  //[EBP-9]与 0 比较
00601661   .  74 15             JE      SHORT iisc_exe.00601678           ;  //相等则跳
00601663   .  8BC1              MOV     EAX, ECX                          ;  //EAX=ECX
00601665   .  F7E8              IMUL    EAX                               ;  //EAX=EAX*EAX
00601667   .  03C0              ADD     EAX, EAX                          ;  //EAX=EAX+EAX
00601669   .  40                INC     EAX                               ;  //EAX=EAX+1
0060166A   .  8B55 FC           MOV     EDX, DWORD PTR [EBP-4]            ;  //字符串
0060166D   .  0FB6540A FF       MOVZX   EDX, BYTE PTR [EDX+ECX-1]         ;  //依次取字符串 ASC 值
00601672   .  F7EA              IMUL    EDX                               ;  //EAX=EAX*EDX
00601674   .  03D8              ADD     EBX, EAX                          ;  //EBX=EBX+EAX
00601676   .  EB 17             JMP     SHORT iisc_exe.0060168F
00601678   >  8B45 FC           MOV     EAX, DWORD PTR [EBP-4]            ;  //字符串
0060167B   .  0FB64408 FF       MOVZX   EAX, BYTE PTR [EAX+ECX-1]         ;  //依次取字符串 ASC 值
00601680   .  F7E8              IMUL    EAX                               ;  //EAX=EAX*EAX
00601682   .  0FBFC0            MOVSX   EAX, AX
00601685   .  BF A0860100       MOV     EDI, 186A0                        ;  //EDI=186A0
0060168A   .  99                CDQ
0060168B   .  F7FF              IDIV    EDI                               ;  //EAX/EDI,商送 EAX,余送 EDX
0060168D   .  03DA              ADD     EBX, EDX                          ;  //EBX=EDX
0060168F   >  41                INC     ECX                               ;  //ECX=ECX+1
00601690   .  4E                DEC     ESI                               ;  //ESI=ESI-1
00601691   .^ 75 CA             JNZ     SHORT iisc_exe.0060165D           ;  //循环
00601693   >  8BC3              MOV     EAX, EBX                          ;  //EAX=EBX
00601695   .  B9 10270000       MOV     ECX, 2710                         ;  //ECX=2710
0060169A   .  99                CDQ
0060169B   .  F7F9              IDIV    ECX                               ;  //EAX/ECX,商送 EAX,余送 EDX
0060169D   .  8BC2              MOV     EAX, EDX                          ;  //EAX=EDX
0060169F   .  F7E8              IMUL    EAX                               ;  //EAX=EAX*EAX
006016A1   .  0345 F8           ADD     EAX, DWORD PTR [EBP-8]            ;  //EAX=EAX+[EBP-8]
006016A4   .  8BD8              MOV     EBX, EAX                          ;  //EBX=EAX
006016A6   .  8BC3              MOV     EAX, EBX                          ;  //EAX=EBX
006016A8   .  B9 A0860100       MOV     ECX, 186A0                        ;  //ECX=186A0
006016AD   .  99                CDQ
006016AE   .  F7F9              IDIV    ECX                               ;  //EAX/ECX,商送 EAX,余送 EDX
006016B0   .  81C2 A0860100     ADD     EDX, 186A0                        ;  //EDX=EDX+186A0
006016B6   .  8BDA              MOV     EBX, EDX                          ;  //EDX=EDX
006016B8   .  EB 12             JMP     SHORT iisc_exe.006016CC
006016BA   >  807D F7 00        CMP     BYTE PTR [EBP-9], 0
006016BE   .  74 07             JE      SHORT iisc_exe.006016C7
006016C0   .  BB 6E740300       MOV     EBX, 3746E
006016C5   .  EB 05             JMP     SHORT iisc_exe.006016CC
006016C7   >  BB 01EB0B00       MOV     EBX, 0BEB01
006016CC   >  33C0              XOR     EAX, EAX
006016CE   .  5A                POP     EDX
006016CF   .  59                POP     ECX
006016D0   .  59                POP     ECX
006016D1   .  64:8910           MOV     DWORD PTR FS:[EAX], EDX
006016D4   .  EB 1C             JMP     SHORT iisc_exe.006016F2
006016D6   .^ E9 292CE0FF       JMP     iisc_exe.00404304
006016DB   .  807D F7 00        CMP     BYTE PTR [EBP-9], 0
006016DF   .  74 07             JE      SHORT iisc_exe.006016E8
006016E1   .  BB 6E740300       MOV     EBX, 3746E
006016E6   .  EB 05             JMP     SHORT iisc_exe.006016ED
006016E8   >  BB 01EB0B00       MOV     EBX, 0BEB01
006016ED   >  E8 7A2FE0FF       CALL    iisc_exe.0040466C
006016F2   >  33C0              XOR     EAX, EAX
006016F4   .  5A                POP     EDX
006016F5   .  59                POP     ECX
006016F6   .  59                POP     ECX
006016F7   .  64:8910           MOV     DWORD PTR FS:[EAX], EDX
006016FA   .  68 0F176000       PUSH    iisc_exe.0060170F
006016FF   >  8D45 FC           LEA     EAX, DWORD PTR [EBP-4]
00601702   .  E8 9535E0FF       CALL    iisc_exe.00404C9C
00601707   .  C3                RETN
00601708   .^ E9 AB2EE0FF       JMP     iisc_exe.004045B8
0060170D   .^ EB F0             JMP     SHORT iisc_exe.006016FF
0060170F   .  8BC3              MOV     EAX, EBX                          ;  //EAX=EBX
00601711   .  5F                POP     EDI
00601712   .  5E                POP     ESI
00601713   .  5B                POP     EBX
00601714   .  8BE5              MOV     ESP, EBP
00601716   .  5D                POP     EBP
00601717   .  C3                RETN
```



# 破解总结

--------------------------------------------------------------

# 算法总结

将字符串`"2nd Speech Center"`, `"3"`, 用户名，通过一个运算 `CALL` 计算，分别取中间的 5 位用`"-"`相连即是注册码。

--------------------------------------------------------------

# 算法注册机

## 易语言代码

```
.版本 2

.子程序 _ 按钮 1_ 被单击

.局部变量 a, 文本型

.局部变量 b, 文本型

.局部变量 c, 文本型

.局部变量 d, 整数型

.局部变量 e, 整数型

.局部变量 f, 整数型

.局部变量 g, 整数型

.局部变量 x, 整数型

.局部变量 y, 整数型

.局部变量 i, 整数型

.如果 (取文本长度 (编辑框 1.内容) ＝ 0)
    信息框 (“请输入用户名！”, 0, “错误”)
.否则
    a ＝ “2nd Speech Center”
    b ＝ “3”
    c ＝ 编辑框 1.内容
    .计次循环首 (取文本长度 (a), i)
        d ＝ i
        d ＝ d × d
        d ＝ d ＋ d
        d ＝ d ＋ 1
        e ＝ 取代码 (a, i)
        d ＝ d × e
        f ＝ f ＋ d
    .计次循环尾 ()
    f ＝ f ％ 十六进制到十进制 (“2710”)
    f ＝ f × f
    f ＝ f ＋ 十六进制到十进制 (“3746E”)
    f ＝ f ％ 十六进制到十进制 (“186A0”)
    f ＝ f ＋ 十六进制到十进制 (“186A0”)
    .计次循环首 (取文本长度 (b), i)
        e ＝ 取代码 (b, i)
        e ＝ e × e
        e ＝ e ％ 十六进制到十进制 (“186A0”)
        g ＝ g ＋ e
    .计次循环尾 ()
    g ＝ g ％ 十六进制到十进制 (“2710”)
    g ＝ g × g
    g ＝ g ＋ 十六进制到十进制 (“BEB01”)
    g ＝ g ％ 十六进制到十进制 (“186A0”)
    g ＝ g ＋ 十六进制到十进制 (“186A0”)
    .计次循环首 (取文本长度 (c), i)
        d ＝ i
        d ＝ d × d
        d ＝ d ＋ d
        d ＝ d ＋ 1
        e ＝ 取代码 (c, i)
        d ＝ d × e
        x ＝ x ＋ d
    .计次循环尾 ()
    x ＝ x ％ 十六进制到十进制 (“2710”)
    x ＝ x × x
    x ＝ x ＋ f
    x ＝ x ％ 十六进制到十进制 (“186A0”)
    x ＝ x ＋ 十六进制到十进制 (“186A0”)
    .计次循环首 (取文本长度 (c), i)
        e ＝ 取代码 (c, i)
        e ＝ e × e
        e ＝ e ％ 十六进制到十进制 (“186A0”)
        y ＝ y ＋ e
    .计次循环尾 ()
    y ＝ y ％ 十六进制到十进制 (“2710”)
    y ＝ y × y
    y ＝ y ＋ g
    y ＝ y ％ 十六进制到十进制 (“186A0”)
    y ＝ y ＋ 十六进制到十进制 (“186A0”)
    编辑框 2.内容 ＝ 取文本中间 (到文本 (y), 2, 5) ＋ “-” ＋ 取文本中间 (到文本 (x), 1, 5)
.如果结束

.子程序 十六进制到十进制, 整数型

.参数 十六进制文本, 文本型

.局部变量 文本, 文本型

.局部变量 最终值, 整数型

.局部变量 i, 整数型, , , 中间循环变量

.局部变量 临时数值, 文本型, , , 每一位上的值

文本 ＝ 到小写 (十六进制文本)

.变量循环首 (1, 取文本长度 (文本), 1, i)
    临时数值 ＝ 取文本中间 (文本, i, 1)
    .如果 (取代码 (临时数值, ) ≥ 取代码 (“a”, ) 且 取代码 (临时数值, ) ≤ 取代码 (“f”, ))
        最终值 ＝ 最终值 ＋ (取代码 (临时数值, ) － 取代码 (“a”, ) ＋ 10) × 求次方 (16, 取文本长度 (文本) － i)
    .否则
        最终值 ＝ 最终值 ＋ 到数值 (临时数值) × 求次方 (16, 取文本长度 (文本) － i)
    .如果结束
.变量循环尾 ()

返回 (最终值)
```

--------------------------------------------------------------

# 注册信息

保存在`[HKEY_USERS\****\\Software\\Microsoft\\02k\\JJTD4]`

--------------------------------------------------------------

感谢飘云老大、猫老大、`Nisy` 老大以及很多前辈们的学习教程以及所有帮助过我的论坛兄弟姐妹们！谢谢

--------------------------------------------------------------

# 版权声明

破文是学习的手记，兴趣是成功的源泉；本破文纯属技术交流，转载请注明作者并保持文章的完整，谢谢!

```
    _/_/__/   _/   _/   _/_/_/
   _/   _/  _/  _/    _/        
  _/_/_/    _/_/     _/_/_/
  _/          _/      _/    _/
 _/          _/      _/_/_/ _/    tianxj
```
