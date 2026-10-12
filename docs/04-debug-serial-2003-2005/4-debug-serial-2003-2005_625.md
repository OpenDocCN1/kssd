
# Focus Magic V3.01

> 原文：[`www.pediy.com/kssd/pediy06/pediy6853.htm`](https://www.pediy.com/kssd/pediy06/pediy6853.htm)

【破解作者】GoOdLeiSuRe  
【作者邮箱】zhmwf@sohu.com  
【作者主页】[`sstudio.home.xdjm.net`](http://sstudio.home.xdjm.net)  
【破解日期】2004/11/22  
【软件名称】Focus Magic V3.01  
【软件大小】4.18MB  
【下载地址】不记得在哪下载了，应该很容易搜索到。  
【软件简介】它能提供一些独特的相片处理功能让模糊或缺失的图片在经过修整之后能被修复成接近原状，其特效与一般影像处理软件不同，着重在图片的清晰化、以及缺裂部分的推导，能与市面上大部分的商业美工软件做结合。Focus Magic 目前已经广泛地被数字摄影师、广告和出版行业所使用，也用在建文件组织（像美国国家档案局）、法医鉴定专家和相片修复专家使用。  
【加壳方式】没有加壳  
【使用工具】OllyDbg v1.10  
【破解平台】Windows XP SP2  
【破解声明】我水平很菜，偶得一点心得，愿与大家分享，错误难免，肯请指正。  

## 破解过程

输入注册信息：

User Name: GoOdLeiSuRe  
Registration Code: 123456789

下断点：`BPX MessageBoxA`

点击“Register”进行注册，程序在显示出错对话框时拦截下来。

幸运的是中断所在的过程就是进行注册计算与保存的过程。

```
00408330  /$  81EC 14010000      SUB ESP,114
00408336  |.  8D8424 B0000000    LEA EAX,DWORD PTR SS:[ESP+B0]
0040833D  |.  53                PUSH EBX
0040833E  |.  55                PUSH EBP
0040833F  |.  8BAC24 20010000    MOV EBP,DWORD PTR SS:[ESP+120]
00408346  |.  56                PUSH ESI
00408347  |.  8B35 5CB24200      MOV ESI,DWORD PTR DS:[<&USER32.GetDlgItemT>;  USER32.GetDlgItemTextA
0040834D  |.  57                PUSH EDI
0040834E  |.  6A 64             PUSH 64                       ; /Count = 64 (100.)
00408350  |.  50                PUSH EAX                      ; |Buffer
00408351  |.  68 F5030000       PUSH 3F5                      ; |ControlID = 3F5 (1013.)
00408356  |.  55                PUSH EBP                      ; |hWnd
00408357  |.  FFD6              CALL ESI                      ; \GetDlgItemTextA    取得用户名 "GoOdLeiSuRe"
00408359  |.  8D4C24 28        LEA ECX,DWORD PTR SS:[ESP+28]
0040835D  |.  6A 64            PUSH 64                        ; /Count = 64 (100.)
0040835F  |.  51               PUSH ECX                       ; |Buffer
00408360  |.  68 F6030000      PUSH 3F6                       ; |ControlID = 3F6 (1014.)
00408365  |.  55               PUSH EBP                       ; |hWnd
00408366  |.  FFD6             CALL ESI                       ; \GetDlgItemTextA    取得输入的注册码 "123456789"
00408368  |.  8A4424 28        MOV AL,BYTE PTR SS:[ESP+28]
0040836C  |.  84C0             TEST AL,AL                     ;  是否输入空注册码？
0040836E  |.  0F84 0E030000    JE FocusMag.00408682
00408374  |.  8D7C24 28        LEA EDI,DWORD PTR SS:[ESP+28]    ;  输入的注册码
00408378  |.  83C9 FF          OR ECX,FFFFFFFF
0040837B  |.  33C0             XOR EAX,EAX
0040837D  |.  F2:AE            REPNE SCAS BYTE PTR ES:[EDI]
0040837F  |.  F7D1             NOT ECX
00408381  |.  49               DEC ECX
00408382  |.  8BD9             MOV EBX,ECX                    ;  以上几行计算输入的注册码长度
00408384  |.  83FB 05          CMP EBX,5                      ;  输入的注册码 < 5 ？
00408387  |.  0F8C B7010000    JL FocusMag.00408544
0040838D  |.  83FB 09          CMP EBX,9                      ;  输入的注册码 <> 9 ？
00408390  |.  75 12            JNZ SHORT FocusMag.004083A4
00408392  |.  8D7C24 28        LEA EDI,DWORD PTR SS:[ESP+28]
00408396  |.  83C9 FF          OR ECX,FFFFFFFF
00408399  |.  884424 2D        MOV BYTE PTR SS:[ESP+2D],AL      ;  取输入注册码的前五位
0040839D  |.  F2:AE            REPNE SCAS BYTE PTR ES:[EDI]
0040839F  |.  F7D1             NOT ECX
004083A1  |.  49               DEC ECX
004083A2  |.  8BD9             MOV EBX,ECX                    ;  以上几行计算取得的输入码的长度（5 位）
004083A4  |>  33C9             XOR ECX,ECX
004083A6  |.  85DB             TEST EBX,EBX                   ;  新取得码的长度 <= 0 ？
004083A8  |.  7E 19            JLE SHORT FocusMag.004083C3
004083AA  |>  8A440C 28       /MOV AL,BYTE PTR SS:[ESP+ECX+28]    ;  循环取新码的各个字符的 ASCII 码
004083AE  |.  3C 30            |CMP AL,30                     ;  取得的 ASCII 码 < 30 ？
004083B0  |.  0F8C 8E010000    |JL FocusMag.00408544
004083B6  |.  3C 39            |CMP AL,39                     ;  取得的 ASCII 码 > 39 ？
004083B8  |.  0F8F 86010000    |JG FocusMag.00408544
004083BE  |.  41               |INC ECX
004083BF  |.  3BCB             |CMP ECX,EBX
004083C1  |.^ 7C E7            \JL SHORT FocusMag.004083AA        ;  这个循环是判断新码是否为数字
004083C3  |>  83FB 05          CMP EBX,5                      ;  新码长度 <= 5 ？
004083C6  |.  7E 6C            JLE SHORT FocusMag.00408434
004083C8  |.  8B541C 23        MOV EDX,DWORD PTR SS:[ESP+EBX+23]
004083CC  |.  8A441C 27        MOV AL,BYTE PTR SS:[ESP+EBX+27]
004083D0  |.  8D4B FB          LEA ECX,DWORD PTR DS:[EBX-5]
004083D3  |.  899424 8C000000  MOV DWORD PTR SS:[ESP+8C],EDX
004083DA  |.  8BD1             MOV EDX,ECX
004083DC  |.  8D7424 28        LEA ESI,DWORD PTR SS:[ESP+28]
004083E0  |.  8DBC24 91000000  LEA EDI,DWORD PTR SS:[ESP+91]
004083E7  |.  888424 90000000  MOV BYTE PTR SS:[ESP+90],AL
004083EE  |.  C1E9 02          SHR ECX,2
004083F1  |.  F3:A5            REP MOVS DWORD PTR ES:[EDI],DWORD PTR DS:[>
004083F3  |.  8BCA             MOV ECX,EDX
004083F5  |.  83E1 03          AND ECX,3
004083F8  |.  33C0             XOR EAX,EAX
004083FA  |.  F3:A4            REP MOVS BYTE PTR ES:[EDI],BYTE PTR DS:[ES>
004083FC  |>  40               /INC EAX
004083FD  |.  8D48 05          |LEA ECX,DWORD PTR DS:[EAX+5]
00408400  |.  3BCB             |CMP ECX,EBX
00408402  |.^ 7C F8            \JL SHORT FocusMag.004083FC
00408404  |.  8DBC24 8C000000  LEA EDI,DWORD PTR SS:[ESP+8C]
0040840B  |.  83C9 FF          OR ECX,FFFFFFFF
0040840E  |.  33C0             XOR EAX,EAX
00408410  |.  C6841C 8C000000 00 MOV BYTE PTR SS:[ESP+EBX+8C],0
00408418  |.  F2:AE            REPNE SCAS BYTE PTR ES:[EDI]
0040841A  |.  F7D1             NOT ECX
0040841C  |.  2BF9             SUB EDI,ECX
0040841E  |.  8D5424 28        LEA EDX,DWORD PTR SS:[ESP+28]
00408422  |.  8BC1             MOV EAX,ECX
00408424  |.  8BF7             MOV ESI,EDI
00408426  |.  8BFA             MOV EDI,EDX
00408428  |.  C1E9 02          SHR ECX,2
0040842B  |.  F3:A5            REP MOVS DWORD PTR ES:[EDI],DWORD PTR DS:[>
0040842D  |.  8BC8             MOV ECX,EAX
0040842F  |.  83E1 03          AND ECX,3
00408432  |.  F3:A4            REP MOVS BYTE PTR ES:[EDI],BYTE PTR DS:[ES>
00408434  |>  0FBE4424 2C      MOVSX EAX,BYTE PTR SS:[ESP+2C]      ;  "5" 第 5（长度）位字符
00408439  |.  0FBE5424 28      MOVSX EDX,BYTE PTR SS:[ESP+28]      ;  "1" 第 1 位字符
0040843E  |.  8D0C80           LEA ECX,DWORD PTR DS:[EAX+EAX*4]     ;  ECX = ASC("5") + ASC("5")*4 = 109
00408441  |.  8D0448           LEA EAX,DWORD PTR DS:[EAX+ECX*2]     ;  EAX = ASC("5") + ECX*2 = B*ASC("5") = 247
00408444  |.  0FBE4C24 29      MOVSX ECX,BYTE PTR SS:[ESP+29]      ;  "2" 第 2 位字符
00408449  |.  0FBEAC02 7CF14200 MOVSX EBP,BYTE PTR DS:[EDX+EAX+42F17C]    ;  "9" Form "3982015674" 由第（长度）位字符定位这组数字串；由（EDX+1）定位数字
00408451  |.  0FBEBC01 7CF14200 MOVSX EDI,BYTE PTR DS:[ECX+EAX+42F17C]    ;  "8" Form "3982015674" 由（ECX+1）定位数字
00408459  |.  8B5424 2A        MOV EDX,DWORD PTR SS:[ESP+2A]      ;  "3" 第 3 位字符
0040845D  |.  8B4424 2C        MOV EAX,DWORD PTR SS:[ESP+2C]      ;  "5" 第 5 位字符
00408461  |.  52               PUSH EDX                      ;  Arg2
```



```
00408462  |.  50                     PUSH EAX                                   ;  Arg1  
00408463  |.  83ED 30                SUB EBP,30                                 ;  字符 -> 数字  
00408466  |.  83EF 30                SUB EDI,30                                 ;  字符 -> 数字  
00408469  |.  E8 A2FEFFFF            CALL FocusMag.00408310  
0040846E  |.  8B4C24 33              MOV ECX,DWORD PTR SS:[ESP+33]              ;  "4" 第 4 位字符  
00408472  |.  8B5424 34              MOV EDX,DWORD PTR SS:[ESP+34]              ;  "5" 第 5 位字符  
00408476  |.  8D3480                 LEA ESI,DWORD PTR DS:[EAX+EAX*4]           ;  ESI = 5*EAX  其中 EAX 为对应的第(Arg2-30+1)位数字  
00408479  |.  51                     PUSH ECX                                   ;  Arg2  
0040847A  |.  52                     PUSH EDX                                   ;  Arg1  
0040847B  |.  D1E6                   SHL ESI,1                                  ;  加倍 ESI = A*EAX  
0040847D  |.  E8 8EFEFFFF            CALL FocusMag.00408310  
00408482  |.  83C4 10                ADD ESP,10  
00408485  |.  03F0                   ADD ESI,EAX                                ;  ESI = ESI + EAX  
00408487  |.  8D4424 18              LEA EAX,DWORD PTR SS:[ESP+18]  
0040848B  |.  50                     PUSH EAX                                   ;  /pSystemTime  
0040848C  |.  FF15 A0B04200          CALL DWORD PTR DS:[<&KERNEL32.GetSystemTim>; \GetSystemTime  
00408492  |.  8B4424 18              MOV EAX,DWORD PTR SS:[ESP+18]              ;  年份/月份  
00408496  |.  B9 0A000000            MOV ECX,0A  
0040849B  |.  25 FFFF0000            AND EAX,0FFFF                              ;  年份  
004084A0  |.  99                     CDQ                                      ;  EDX 清零  
004084A1  |.  F7F9                   IDIV ECX                                   ;  EAX = EAX \ A ; EDX = EAX | A  
004084A3  |.  8B4424 1A              MOV EAX,DWORD PTR SS:[ESP+1A]              ;  月份/星期  
004084A7  |.  33C9                   XOR ECX,ECX  
004084A9  |.  25 FFFF0000            AND EAX,0FFFF                              ;  月份  
004084AE  |.  3BD7                   CMP EDX,EDI                                ;  （年份/A）的余数 <> EDI ？  
004084B0  |.  75 09                  JNZ SHORT FocusMag.004084BB  
004084B2  |.  3BC6                   CMP EAX,ESI                                ;  月份 <> ESI ？  
004084B4  |.  75 05                  JNZ SHORT FocusMag.004084BB  
004084B6  |.  B9 01000000            MOV ECX,1                                  ;  上面条件相等时置 ECX=1  
004084BB  |>  66:837C24 1E 0E        CMP WORD PTR SS:[ESP+1E],0E                ;  日 > E ？  
004084C1  |.  77 1B                  JA SHORT FocusMag.004084DE  
004084C3  |.  46                     INC ESI  
004084C4  |.  83FE 0D                CMP ESI,0D                                 ;  ESI <> D ？  
004084C7  |.  75 0D                  JNZ SHORT FocusMag.004084D6  
004084C9  |.  47                     INC EDI  
004084CA  |.  83FF 0A                CMP EDI,0A                                 ;  EDI <> A ？  
004084CD  |.  75 02                  JNZ SHORT FocusMag.004084D1  
004084CF  |.  33FF                   XOR EDI,EDI  
004084D1  |>  BE 01000000            MOV ESI,1  
004084D6  |>  3BD7                   CMP EDX,EDI  
004084D8  |.  75 04                  JNZ SHORT FocusMag.004084DE  
004084DA  |.  3BC6                   CMP EAX,ESI                                ;  EAX = ESI ？  
004084DC  |.  74 04                  JE SHORT FocusMag.004084E2                 ;  若 ECX=0 则要跳，否则可不跳  
004084DE  |>  85C9                   TEST ECX,ECX  
004084E0  |.  74 5B                  JE SHORT FocusMag.0040853D                 ;  不能跳，ECX <> 0  
004084E2  |>  83FB 05                CMP EBX,5                                  ;  EBX <= 5 ？  
004084E5  |.  7E 52                  JLE SHORT FocusMag.00408539                ;  最好跳  
004084E7  |.  BE 05000000            MOV ESI,5  
004084EC  |>  8DBC24 C0000000        /LEA EDI,DWORD PTR SS:[ESP+C0]  
004084F3  |.  83C9 FF                |OR ECX,FFFFFFFF  
004084F6  |.  33C0                   |XOR EAX,EAX  
004084F8  |.  8D56 FC                |LEA EDX,DWORD PTR DS:[ESI-4]  
004084FB  |.  F2:AE                  |REPNE SCAS BYTE PTR ES:[EDI]  
004084FD  |.  F7D1                   |NOT ECX  
004084FF  |.  49                     |DEC ECX  
00408500  |.  3BCA                   |CMP ECX,EDX  
00408502  |.  7C 39                  |JL SHORT FocusMag.0040853D  
00408504  |.  8A8434 BB000000        |MOV AL,BYTE PTR SS:[ESP+ESI+BB]  
0040850B  |.  3C 60                  |CMP AL,60  
0040850D  |.  7E 02                  |JLE SHORT FocusMag.00408511  
0040850F  |.  04 E0                  |ADD AL,0E0  
00408511  |>  3C 41                  |CMP AL,41  
00408513  |.  7C 17                  |JL SHORT FocusMag.0040852C  
00408515  |.  3C 5A                  |CMP AL,5A  
00408517  |.  7F 13                  |JG SHORT FocusMag.0040852C  
00408519  |.  0FBEC0                 |MOVSX EAX,AL  
0040851C  |.  83E8 40                |SUB EAX,40  
0040851F  |.  B9 0A000000            |MOV ECX,0A  
00408524  |.  99                     |CDQ  
00408525  |.  F7F9                   |IDIV ECX  
00408527  |.  80C2 30                |ADD DL,30  
0040852A  |.  EB 02                  |JMP SHORT FocusMag.0040852E  
0040852C  |>  B2 30                  |MOV DL,30  
0040852E  |>  385434 28              |CMP BYTE PTR SS:[ESP+ESI+28],DL  
00408532  |.  75 09                  |JNZ SHORT FocusMag.0040853D  
00408534  |.  46                     |INC ESI  
00408535  |.  3BF3                   |CMP ESI,EBX  
00408537  |.^ 7C B3                  \JL SHORT FocusMag.004084EC  
00408539  |>  85ED                   TEST EBP,EBP                               ;  EBP <> 0 ？正确值应为：EBP = 1  
0040853B  |.  75 27                  JNZ SHORT FocusMag.00408564                ;  一定要跳  
0040853D  |>  8BAC24 28010000        MOV EBP,DWORD PTR SS:[ESP+128]  
00408544  |>  6A 30                  PUSH 30                                    ;  /Style = MB_OK|MB_ICONEXCLAMATION|MB_APPLMODAL  
00408546  |.  68 CCEE4200            PUSH FocusMag.0042EECC                     ;  |Title = "Focus Magic"  
0040854B  |.  68 68F54200            PUSH FocusMag.0042F568                     ;  |Text = "Invalid Registration Code !  
                                                                                   ;  |        If you have lost your registration code, or it  
                                                                                   ;  |        does not work, then you can get your registration  
                                                                                   ;  |        details emailed to you by going to :-(
                                                                                   ;  |        [`www.focusmagic.com/newcode`](http://www.focusmagic.com/newcode)"  
00408550  |.  55                     PUSH EBP                                   ;  |hOwner  
00408551  |.  FF15 94B14200          CALL DWORD PTR DS:[<&USER32.MessageBoxA>]  ;  \MessageBoxA  
00408557  |.  5F                     POP EDI  
00408558  |.  5E                     POP ESI  
00408559  |.  5D                     POP EBP  
0040855A  |.  32C0                   XOR AL,AL  
0040855C  |.  5B                     POP EBX  
0040855D  |.  81C4 14010000          ADD ESP,114  
00408563  |.  C3                     RETN  
00408564  |>  83FD 01                CMP EBP,1                                  ;  EBP <> 1 ？  
00408567  |.  0F85 F6000000          JNZ FocusMag.00408663                      ;  这儿不能跳 EBP = 1 若 EBP<>1、3，则显示"专业版"？并非成功注册。  
0040856D  |.  8D5424 14              LEA EDX,DWORD PTR SS:[ESP+14]              ;  只要前面能跳至这儿就可以顺利完成自动注册了。  
00408571  |.  52                     PUSH EDX                                   ;  /pHandle  
00408572  |.  68 8CE04200            PUSH FocusMag.0042E08C                     ;  |Subkey = "SOFTWARE\\Acclaim Software Ltd\\Focus Magic"  
00408577  |.  68 02000080            PUSH 80000002                              ;  |hKey = HKEY_LOCAL_MACHINE  
0040857C  |.  FF15 04B04200          CALL DWORD PTR DS:[<&ADVAPI32.RegOpenKeyA>];  \RegOpenKeyA  
```



# 00408582 FocusMag 逆向分析

## 汇编代码

```
00408582  |.  85C0             TEST EAX,EAX  

00408584  |.  74 27            JE SHORT FocusMag.004085AD              ;  这儿要跳  

00408586  |.  8B8424 28010000  MOV EAX,DWORD PTR SS:[ESP+128]  

0040858D  |.  6A 30            PUSH 30                                 ; /Style = MB_OK|MB_ICONEXCLAMATION|MB_APPLMODAL  

0040858F  |.  68 4CF54200      PUSH FocusMag.0042F54C                 ; |Title = "Registry Entries Missing"  

00408594  |.  68 0CF54200      PUSH FocusMag.0042F50C                 ; |Text = "Focus Magic is not installed properly.  Please re-install it"  

00408599  |.  50               PUSH EAX                                ; |hOwner  

0040859A  |.  FF15 94B14200     CALL DWORD PTR DS:[<&USER32.MessageBoxA>]  ; \MessageBoxA  

004085A0  |.  5F               POP EDI  

004085A1  |.  5E               POP ESI  

004085A2  |.  5D               POP EBP  

004085A3  |.  32C0             XOR AL,AL  

004085A5  |.  5B               POP EBX  

004085A6  |.  81C4 14010000     ADD ESP,114  

004085AC  |.  C3               RETN  

004085AD  |>  8DBC24 C0000000  LEA EDI,DWORD PTR SS:[ESP+C0]          ;  以下应该是把输入的正确注册信息写入注册表  

004085B4  |.  83C9 FF          OR ECX,FFFFFFFF  

004085B7  |.  33C0             XOR EAX,EAX  

004085B9  |.  8B5424 14        MOV EDX,DWORD PTR SS:[ESP+14]  

004085BD  |.  F2:AE            REPNE SCAS BYTE PTR ES:[EDI]  

004085BF  |.  F7D1             NOT ECX  

004085C1  |.  8B35 08B04200     MOV ESI,DWORD PTR DS:[<&ADVAPI32.RegSetVal>;  ADVAPI32.RegSetValueExA  

004085C7  |.  51               PUSH ECX                                ; /BufSize  

004085C8  |.  8D8C24 C4000000  LEA ECX,DWORD PTR SS:[ESP+C4]          ; |  

004085CF  |.  51               PUSH ECX                                ; |Buffer  

004085D0  |.  6A 01            PUSH 1                                  ; |ValueType = REG_SZ  

004085D2  |.  50               PUSH EAX                                ; |Reserved => 0  

004085D3  |.  68 80E04200      PUSH FocusMag.0042E080                 ; |ValueName = "User Name"  

004085D8  |.  52               PUSH EDX                                ; |hKey  

004085D9  |.  FFD6             CALL ESI                                ; \RegSetValueExA  

004085DB  |.  0FBE05 883E4300  MOVSX EAX,BYTE PTR DS:[433E88]         ;  38 "8"  

004085E2  |.  83E8 30          SUB EAX,30  

004085E5  |.  8D7C24 28        LEA EDI,DWORD PTR SS:[ESP+28]          ;  新码 "12345"  

004085E9  |.  C64424 13 00     MOV BYTE PTR SS:[ESP+13],0  

004085EE  |.  8D0C80           LEA ECX,DWORD PTR DS:[EAX+EAX*4]       ;  ECX = 5 * EAX  

004085F1  |.  8D0448           LEA EAX,DWORD PTR DS:[EAX+ECX*2]       ;  EAX = B * EAX  

004085F4  |.  8A90 4DF34200    MOV DL,BYTE PTR DS:[EAX+42F34D]        ;  "1" Form "2190835647"  

004085FA  |.  8A88 4EF34200    MOV CL,BYTE PTR DS:[EAX+42F34E]        ;  "9" Form "2190835647"  

00408600  |.  885424 10        MOV BYTE PTR SS:[ESP+10],DL  

00408604  |.  8A90 4FF34200    MOV DL,BYTE PTR DS:[EAX+42F34F]        ;  "0" Form "2190835647"  

0040860A  |.  884C24 11        MOV BYTE PTR SS:[ESP+11],CL  

0040860E  |.  83C9 FF          OR ECX,FFFFFFFF  

00408611  |.  33C0             XOR EAX,EAX  

00408613  |.  885424 12        MOV BYTE PTR SS:[ESP+12],DL  

00408617  |.  F2:AE            REPNE SCAS BYTE PTR ES:[EDI]  

00408619  |.  F7D1             NOT ECX  

0040861B  |.  8D4424 10        LEA EAX,DWORD PTR SS:[ESP+10]  

0040861F  |.  51               PUSH ECX                                ; /BufSize  

00408620  |.  8B4C24 18        MOV ECX,DWORD PTR SS:[ESP+18]          ; |  

00408624  |.  50               PUSH EAX                                ; |Buffer  

00408625  |.  6A 01            PUSH 1                                  ; |ValueType = REG_SZ  

00408627  |.  6A 00            PUSH 0                                  ; |Reserved = 0  

00408629  |.  68 A8E34200      PUSH FocusMag.0042E3A8                 ; |ValueName = "RCode"  

0040862E  |.  51               PUSH ECX                                ; |hKey  

0040862F  |.  FFD6             CALL ESI                                ; \RegSetValueExA  

00408631  |.  8B5424 14        MOV EDX,DWORD PTR SS:[ESP+14]  

00408635  |.  52               PUSH EDX                                ; /hKey  

00408636  |.  FF15 0CB04200     CALL DWORD PTR DS:[<&ADVAPI32.RegCloseKey>>; \RegCloseKey  

0040863C  |.  8B8424 28010000  MOV EAX,DWORD PTR SS:[ESP+128]         ;  以下这个对话框才是真正注册成功时显示的  

00408643  |.  6A 30            PUSH 30                                 ; /Style = MB_OK|MB_ICONEXCLAMATION|MB_APPLMODAL  

00408645  |.  68 CCEE4200      PUSH FocusMag.0042EECC                 ; |Title = "Focus Magic"  

0040864A  |.  68 E4F44200      PUSH FocusMag.0042F4E4                 ; |Text = "Thankyou for registering Focus Magic"  

0040864F  |.  50               PUSH EAX                                ; |hOwner = 0029025C  

00408650  |.  FF15 94B14200     CALL DWORD PTR DS:[<&USER32.MessageBoxA>]  ; \MessageBoxA  

00408656  |.  5F               POP EDI  

00408657  |.  5E               POP ESI  

00408658  |.  5D               POP EBP  

00408659  |.  B0 01            MOV AL,1  

0040865B  |.  5B               POP EBX  

0040865C  |.  81C4 14010000    ADD ESP,114  

00408662  |.  C3               RETN  

00408663  |>  83FD 03          CMP EBP,3                               ;  EBP <> 3 ？显示"专业版"，估计是用来误导的。  

00408666  |.  75 1A            JNZ SHORT FocusMag.00408682  

00408668  |.  8B8C24 28010000  MOV ECX,DWORD PTR SS:[ESP+128]  

0040866F  |.  6A 00            PUSH 0                                  ; /Style = MB_OK|MB_APPLMODAL  

00408671  |.  68 DCF44200      PUSH FocusMag.0042F4DC                 ; |Title = "RegType"  

00408676  |.  68 CCF44200      PUSH FocusMag.0042F4CC                 ; |Text = "Professional"  

0040867B  |.  51               PUSH ECX                                ; |hOwner  

0040867C  |.  FF15 94B14200     CALL DWORD PTR DS:[<&USER32.MessageBoxA>]  ; \MessageBoxA  

00408682  |>  5F               POP EDI  

00408683  |.  5E               POP ESI  

00408684  |.  5D               POP EBP  

00408685  |.  32C0             XOR AL,AL  

00408687  |.  5B               POP EBX  

00408688  |.  81C4 14010000    ADD ESP,114  

0040868E  |\. C3               RETN  
```

## 算法分析

程序在进行注册时使用了二组数据：

### 第一组数据（用来生成 `RegCode` 记录在注册表中）

```
0  5279631048
1  9275683014
2  3845721096
3  0234857961
4  5432986701
5  4530967821
6  3081497652
7  3140985672
8  2190835647
9  5476310928
```

### 第二组数据（用来验证注册码是否成立）

```
0  7615804293
1  7816934250
2  7650239418
3  0912358647
4  8932106754
5  3982015674
6  1390487625
7  3190267854
8  3105867942
9  6584103297
```

现在分析时间为“`2004/11`”，利用这些就够了。设注册码为 `ABCDEFGHI`（每个字母代表一个数字）。

1. 使用第二组中的一串数字，如“`8932106754`”；
2. 这串数字位于第 `4` 串，则 `E=4`；
3. 为了使 `EBP=1`，“`1`”位于“`8932106754`”的第 `5` 位，实际就由数字 `4` 定位，所以 `A=4`；
4. `2004/10` 得余数为 `4`，“`4`”位于“`8932106754`”的第 `9` 位，故 `B=9`；
5. 立一方程组：`11`（月份）= `X` + `Y`；`X=10Z`；`Z=K1`；`Y=K2` => 设 `K1=K2`（这样一设就很容易解了）

   解得：`K1=K2=1`，“`1`”位于“`8932106754`”的第 `5` 位，实际就由数字 `4` 定位。

   这样便确定了注册码的第 `3`、`4` 两位：`C=4`，`D=4`；
6. `FGHI` 未用到，实际注册码仅用了 `5` 位；
7. 注册码为：`49444`

由以上分析，很容易就写出注册机了，在此略。

## 破解总结

这个程序注册时读取了系统时间，所以注册码也会因时间不同而不同，这样使得程序最终没有依靠输入的注册码为判断依据，必将导致因为轻松爆破而使得软件自注册。

## 版权声明

本文纯属技术交流，转载请注明作者并保持文章的完整，谢谢！

`4dd466d0e1e0cc123afbe504da2462ca`
