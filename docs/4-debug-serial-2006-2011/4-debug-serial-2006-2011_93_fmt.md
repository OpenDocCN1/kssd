
# NeoN Reminder 1.3

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-508.htm`](https://www.pediy.com/kssd/pediy08/pediy8-508.htm)

【破解软件】`NeoN Reminder 1.3`

【软件类别】国外软件/共享版/记事管理

【运行环境】`Win9x`/`Me`/`NT`/`2000`/`XP`/`2003`

【保护方式】注册码 + `ASPack`

【作者声明】初学 Crack，只是感兴趣，消遣业余时间，错误之处敬请诸位前辈不吝赐教。

【下载地址】`http://www.onlinedown.net/soft/51331.htm`

【软件介绍】一个简单易用功能强大的个人备忘录软件，界面简单实用，可以添加各种行程标记，可以使用图片来标记不同类型的事项，简明显著，让你不用再去看文字，就可以知道今天的日程，占用资源很小，对中文有良好支持！这是 1.2 最终版！

## 一、分析过程

`ASPack` 壳，脱壳后编译语言是：`Borland C++ 1999`。用插件 `KANAL` 查看是:`MD5`，在 `MD5` 加密用户名的地方转了半天与注册算法毫无关系，估计是迷惑破解的。输入用户名注册码后，提示重启。再次启动后，`About` 窗口就"Registered to：wzwgp"，天底下肯定没有这等好事。软件运行一段时间后有：`it is a trial version …… can only …… period of time(30 days)……`。找到：`00408680` 处的 `oftware\\neon labs\\neon reminder\\`，向上在 `004085C4` 处下断。`F9` 没有断下，又忙了半天才搞清楚软件原来是要启动后 5 分钟后再开始验证注册信息。

```
004085C4   .  55                  PUSH EBP                    ; F9,5 钟后断下
004085C5   .  8BEC                MOV EBP,ESP
004085C7   .  81C4 74FDFFFF       ADD ESP,-28C
004085CD   .  B8 00F65100         MOV EAX,NeoN_Rem.0051F600
004085D2   .  53                  PUSH EBX
004085D3   .  56                  PUSH ESI
004085D4   .  57                  PUSH EDI
004085D5   .  E8 92C70D00         CALL NeoN_Rem.004E4D6C
004085DA   .  6A 00               PUSH 0                      ; /Arg1 = 00000000
004085DC   .  E8 97550E00         CALL NeoN_Rem.004EDB78      ; \\NeoN_Rem.004EDB78
004085E1   .  59                  POP ECX
004085E2   .  50                  PUSH EAX                    ; /Arg1
004085E3   .  E8 701D0E00         CALL NeoN_Rem.004EA358      ; \\NeoN_Rem.004EA358
004085E8   .  59                  POP ECX
004085E9   .  E8 961D0E00         CALL NeoN_Rem.004EA384
004085EE   .  B9 05000000         MOV ECX,5
004085F3   .  99                  CDQ
004085F4   .  F7F9               IDIV ECX
004085F6   .  8BC2               MOV EAX,EDX
004085F8   .  803D E2D55100 00   CMP BYTE PTR DS:[51D5E2],0
004085FF   .  0F84 5C050000      JE NeoN_Rem.00408B61
00408605   .  803D E1D55100 00   CMP BYTE PTR DS:[51D5E1],0
0040860C   .  0F84 4F050000      JE NeoN_Rem.00408B61
00408612   .  83F8 03            CMP EAX,3
00408615   .  0F85 46050000      JNZ NeoN_Rem.00408B61
0040861B   .  66:C745 B8 0800    MOV WORD PTR SS:[EBP-48],8
00408621   .  33D2               XOR EDX,EDX
00408623   .  33C9               XOR ECX,ECX
00408625   .  8955 FC            MOV DWORD PTR SS:[EBP-4],EDX
00408628   .  B2 01             MOV DL,1
0040862A   .  FF45 C4            INC DWORD PTR SS:[EBP-3C]
0040862D   .  A1 B00E4600        MOV EAX,DWORD PTR DS:[460EB0]
00408632   .  66:C745 B8 1400    MOV WORD PTR SS:[EBP-48],14
00408638   .  66:C745 B8 2000    MOV WORD PTR SS:[EBP-48],20
0040863E   .  894D F8            MOV DWORD PTR SS:[EBP-8],ECX
00408641   .  FF45 C4            INC DWORD PTR SS:[EBP-3C]
00408644   .  66:C745 B8 1400    MOV WORD PTR SS:[EBP-48],14
0040864A   .  66:C745 B8 2C00    MOV WORD PTR SS:[EBP-48],2C
00408650   .  E8 5B890500        CALL NeoN_Rem.00460FB0
00408655   .  8BD8               MOV EBX,EAX
00408657   .  803D B4D25100 00   CMP BYTE PTR DS:[51D2B4],0
0040865E   .  74 0E              JE SHORT NeoN_Rem.0040866E
00408660   .  BA 02000080        MOV EDX,80000002
00408665   .  8BC3               MOV EAX,EBX
00408667   .  E8 F48D0E00        CALL NeoN_Rem.004F1460
0040866C   .  EB 0C              JMP SHORT NeoN_Rem.0040867A
0040866E   >  BA 01000080        MOV EDX,80000001
00408673   .  8BC3               MOV EAX,EBX
00408675   .  E8 E68D0E00        CALL NeoN_Rem.004F1460
0040867A   >  66:C745 B8 4400    MOV WORD PTR SS:[EBP-48],44
00408680   .  BA CCE45100        MOV EDX,NeoN_Rem.0051E4CC      ;  SOFTWARE\NeoN Labs\NeoN
                                                              ;  Reminder\User NameNumber
00408685   .  8D45 EC            LEA EAX,DWORD PTR SS:[EBP-14]
00408688   .  E8 5F8E0E00        CALL NeoN_Rem.004F14EC
0040868D   .  FF45 C4            INC DWORD PTR SS:[EBP-3C]
00408690   .  66:C745 B8 3800    MOV WORD PTR SS:[EBP-48],38
00408696   .  33C9               XOR ECX,ECX
00408698   .  8B55 EC            MOV EDX,DWORD PTR SS:[EBP-14]
0040869B   .  8BC3               MOV EAX,EBX
0040869D   .  E8 168A0500        CALL NeoN_Rem.004610B8
004086A2   .  66:C745 B8 5000    MOV WORD PTR SS:[EBP-48],50
004086A8   .  BA EEE45100        MOV EDX,NeoN_Rem.0051E4EE      ;  User NameNumber
004086AD   .  8D45 E4            LEA EAX,DWORD PTR SS:[EBP-1C]
004086B0   .  E8 378E0E00        CALL NeoN_Rem.004F14EC
004086B5   .  FF45 C4            INC DWORD PTR SS:[EBP-3C]
004086B8   .  8D4D E8            LEA ECX,DWORD PTR SS:[EBP-18]
004086BB   .  8B10               MOV EDX,DWORD PTR DS:[EAX]
004086BD   .  33C0               XOR EAX,EAX
004086BF   .  8945 E8            MOV DWORD PTR SS:[EBP-18],EAX
004086C2   .  8BC3               MOV EAX,EBX
004086C4   .  FF45 C4            INC DWORD PTR SS:[EBP-3C]
004086C7   .  E8 E08D0500        CALL NeoN_Rem.004614AC         ;  取用户名
004086CC   .  FF4D C4            DEC DWORD PTR SS:[EBP-3C]      ;  [EBP-3C]用户名位数
004086CF   .  8D45 E4            LEA EAX,DWORD PTR SS:[EBP-1C]
004086D2   .  BA 02000000        MOV EDX,2
004086D7   .  E8 20900E00        CALL NeoN_Rem.004F16FC
004086DC   .  66:C745 B8 3800    MOV WORD PTR SS:[EBP-48],38
004086E2   .  66:C745 B8 5C00    MOV WORD PTR SS:[EBP-48],5C
004086E8   .  BA F8E45100        MOV EDX,NeoN_Rem.0051E4F8      ;  Number
004086ED   .  8D45 E0            LEA EAX,DWORD PTR SS:[EBP-20]
004086F0   .  E8 F78D0E00        CALL NeoN_Rem.004F14EC
004086F5   .  FF45 C4            INC DWORD PTR SS:[EBP-3C]
004086F8   .  8D4D DC            LEA ECX,DWORD PTR SS:[EBP-24]
004086FB   .  8B10               MOV EDX,DWORD PTR DS:[EAX]
004086FD   .  33C0               XOR EAX,EAX
004086FF   .  8945 DC            MOV DWORD PTR SS:[EBP-24],EAX
00408702   .  8BC3               MOV EAX,EBX
00408704   .  FF45 C4            INC DWORD PTR SS:[EBP-3C]
00408707   .  E8 A08D0500        CALL NeoN_Rem.004614AC         ;  取假码
0040870C   .  8D55 DC            LEA EDX,DWORD PTR SS:[EBP-24]
0040870F   .  8D45 FC            LEA EAX,DWORD PTR SS:[EBP-4]
00408712   .  E8 15900E00        CALL NeoN_Rem.004F172C
00408717   .  FF4D C4            DEC DWORD PTR SS:[EBP-3C]
0040871A   .  8D45 DC            LEA EAX,DWORD PTR SS:[EBP-24]   ;  [EBP-24]假码地址
0040871D   .  BA 02000000        MOV EDX,2
00408722   .  E8 D58F0E00        CALL NeoN_Rem.004F16FC
00408727   .  FF4D C4            DEC DWORD PTR SS:[EBP-3C]
0040872A   .  8D45 E0            LEA EAX,DWORD PTR SS:[EBP-20]
0040872D   .  BA 02000000        MOV EDX,2
00408732   .  E8 C58F0E00        CALL NeoN_Rem.004F16FC
00408737   .  66:C745 B8 6800    MOV WORD PTR SS:[EBP-48],68
0040873D   .  BA FFE45100        MOV EDX,NeoN_Rem.0051E4FF
00408742   .  8D45 D8            LEA EAX,DWORD PTR SS:[EBP-28]
00408745   .  E8 A28D0E00        CALL NeoN_Rem.004F14EC
0040874A   .  FF45 C4            INC DWORD PTR SS:[EBP-3C]
0040874D   .  8D55 D8            LEA EDX,DWORD PTR SS:[EBP-28]
00408750   .  8D45 E8            LEA EAX,DWORD PTR SS:[EBP-18]
00408753   .  E8 A0900E00        CALL NeoN_Rem.004F17F8
00408758   .  50                  PUSH EAX                    ; /Arg1
00408759   .  FF4D C4            DEC DWORD PTR SS:[EBP-3C]      ; |
0040875C   .  8D45 D8            LEA EAX,DWORD PTR SS:[EBP-28]   ; |
0040875F   .  BA 02000000        MOV EDX,2                     ; |
00408764   .  E8 938F0E00        CALL NeoN_Rem.004F16FC        ; \\NeoN_Rem.004F16FC
00408769   .  59                  POP ECX
```



```
0040876A   .  84C9                 TEST CL,CL  

0040876C   .  0F84 27010000        JE NeoN_Rem.00408899  

00408772   .  837D E8 00           CMP DWORD PTR SS:[EBP-18],0        ;  [EBP-18]用户名地址  

00408776   .  74 05                JE SHORT NeoN_Rem.0040877D  

00408778   .  8B45 E8              MOV EAX,DWORD PTR SS:[EBP-18]  

0040877B   .  EB 05                JMP SHORT NeoN_Rem.00408782  

0040877D   >  B8 00E55100          MOV EAX,NeoN_Rem.0051E500  

00408782   >  8BF8                 MOV EDI,EAX  

00408784   .  33C0                 XOR EAX,EAX  

00408786   .  83C9 FF              OR ECX,FFFFFFFF  

00408789   .  8DB5 98FEFFFF        LEA ESI,DWORD PTR SS:[EBP-168]  

0040878F   .  F2:AE                REPNE SCAS BYTE PTR ES:[EDI]  

00408791   .  F7D1                 NOT ECX  

00408793   .  2BF9                 SUB EDI,ECX  

00408795   .  8BD1                 MOV EDX,ECX  

00408797   .  87F7                 XCHG EDI,ESI                       ;  用户名地址交换到 EDI  

00408799   .  C1E9 02              SHR ECX,2                           ;  ECX=用户名位数加 1  

0040879C   .  8BC7                 MOV EAX,EDI  

0040879E   .  F3:A5                REP MOVS DWORD PTR ES:[EDI],DWORD >  

004087A0   .  8BCA                 MOV ECX,EDX  

004087A2   .  8D85 98FEFFFF        LEA EAX,DWORD PTR SS:[EBP-168]  

004087A8   .  83E1 03              AND ECX,3  

004087AB   .  F3:A4                REP MOVS BYTE PTR ES:[EDI],BYTE PT>  

004087AD   .  50                    PUSH EAX                             ;  /Arg1  

004087AE   .  E8 B92A0000          CALL NeoN_Rem.0040B26C               ;  \\NeoN_Rem.0040B26C  

004087B3   .  59                    POP ECX  

004087B4   .  8D95 98FEFFFF        LEA EDX,DWORD PTR SS:[EBP-168]  

004087BA   .  52                    PUSH EDX  

004087BB   .  E8 38C30D00          CALL NeoN_Rem.004E4AF8  

004087C0   .  59                    POP ECX  

004087C1   .  8945 A4              MOV DWORD PTR SS:[EBP-5C],EAX  

004087C4   .  66:C745 B8 3800      MOV WORD PTR SS:[EBP-48],38  

004087CA   .  C745 A0 01000000     MOV DWORD PTR SS:[EBP-60],1         ;  [EBP-60]置初值 1  

004087D1   .  33C0                 XOR EAX,EAX  

004087D3   .  8DB5 98FEFFFF        LEA ESI,DWORD PTR SS:[EBP-168]      ;  [EBP-168]用户名地址  

004087D9   .  3B45 A4              CMP EAX,DWORD PTR SS:[EBP-5C]  

004087DC   .  7D 0D                JGE SHORT NeoN_Rem.004087EB  

004087DE   >  0FBE16               MOVSX EDX,BYTE PTR DS:[ESI]         ;  逐位取用户名  

004087E1   .  0155 A0              ADD DWORD PTR SS:[EBP-60],EDX      ;  [EBP-60]=用户名 16 进制值累  

加和再加 1  

004087E4   .  40                    INC EAX  

004087E5   .  46                    INC ESI  

004087E6   .  3B45 A4              CMP EAX,DWORD PTR SS:[EBP-5C]      ;  [EBP-5C]=用户名位数  

004087E9   .^ 7C F3                JL SHORT NeoN_Rem.004087DE  

004087EB   >  8B45 A0              MOV EAX,DWORD PTR SS:[EBP-60]      ;  累加和入 EAX  

004087EE   .  33C9                 XOR ECX,ECX  

004087F0   .  99                    CDQ  

004087F1   .  F77D A4              IDIV DWORD PTR SS:[EBP-5C]         ;  除用户名位数  

004087F4   .  83C2 0A              ADD EDX,0A                         ;  余数加 A(这是注册码位数)  

004087F7   .  8DB5 98FDFFFF        LEA ESI,DWORD PTR SS:[EBP-268]  

004087FD   .  8955 9C              MOV DWORD PTR SS:[EBP-64],EDX  

00408800   .  66:C745 B8 3800      MOV WORD PTR SS:[EBP-48],38  

00408806   .  3B4D 9C              CMP ECX,DWORD PTR SS:[EBP-64]  

00408809   .  7D 48                JGE SHORT NeoN_Rem.00408853  

0040880B   >  8BF9                 MOV EDI,ECX  

0040880D   .  0FAFF9               IMUL EDI,ECX                       ;  注册码下标相乘  

00408810   .  03F9                 ADD EDI,ECX                        ;  再与乘积相加  

00408812   .  337D A0              XOR EDI,DWORD PTR SS:[EBP-60]     ;  异或累加和  

00408815   .  8BC7                 MOV EAX,EDI  

00408817   .  99                    CDQ  

00408818   .  33C2                 XOR EAX,EDX  

0040881A   .  2BC2                 SUB EAX,EDX  

0040881C   .  BF 4A000000          MOV EDI,4A  

00408821   .  99                    CDQ  

00408822   .  F7FF                 IDIV EDI                           ;  除 4A  

00408824   .  8BC2                 MOV EAX,EDX  

00408826   .  04 30                ADD AL,30                           ;  余数+30  

00408828   .  66:C745 B8 3800      MOV WORD PTR SS:[EBP-48],38  

0040882E   .  0FBED0               MOVSX EDX,AL  

00408831   .  83FA 3A              CMP EDX,3A  

00408834   .  7C 05                JL SHORT NeoN_Rem.0040883B  

00408836   .  83FA 40              CMP EDX,40  

00408839   .  7E 0D                JLE SHORT NeoN_Rem.00408848  

0040883B   >  0FBED0               MOVSX EDX,AL  

0040883E   .  83FA 5B              CMP EDX,5B  

00408841   .  7C 07                JL SHORT NeoN_Rem.0040884A  

00408843   .  83FA 60              CMP EDX,60  

00408846   .  7F 02                JG SHORT NeoN_Rem.0040884A  

00408848   >  04 0F                ADD AL,0F  

0040884A   >  8806                 MOV BYTE PTR DS:[ESI],AL           ;  保存  

0040884C   .  41                    INC ECX                            ;  +1  

0040884D   .  46                    INC ESI  

0040884E   .  3B4D 9C              CMP ECX,DWORD PTR SS:[EBP-64]      ;  [EBP-64]循环次数(注册码位  

数)  

00408851   .^ 7C B8                JL SHORT NeoN_Rem.0040880B  

00408853   >  C6840D 98FDFFFF 00   MOV BYTE PTR SS:[EBP+ECX-268],0  

0040885B   .  8D45 F8              LEA EAX,DWORD PTR SS:[EBP-8]  

0040885E   .  8B55 A4              MOV EDX,DWORD PTR SS:[EBP-5C]  

00408861   .  E8 8A900E00          CALL NeoN_Rem.004F18F0  

00408866   .  66:C745 B8 7400      MOV WORD PTR SS:[EBP-48],74  

0040886C   .  8D95 98FDFFFF        LEA EDX,DWORD PTR SS:[EBP-268]     ;  [EBP-268]计算出的真码  

00408872   .  8D45 D4              LEA EAX,DWORD PTR SS:[EBP-2C]  

00408875   .  E8 728C0E00          CALL NeoN_Rem.004F14EC  

0040887A   .  8BD0                 MOV EDX,EAX  

0040887C   .  FF45 C4              INC DWORD PTR SS:[EBP-3C]  

0040887F   .  8D45 F8              LEA EAX,DWORD PTR SS:[EBP-8]  

00408882   .  E8 A58E0E00          CALL NeoN_Rem.004F172C  

00408887   .  FF4D C4              DEC DWORD PTR SS:[EBP-3C]  

0040888A   .  8D45 D4              LEA EAX,DWORD PTR SS:[EBP-2C]  

0040888D   .  BA 02000000          MOV EDX,2  

00408892   .  E8 658E0E00          CALL NeoN_Rem.004F16FC  

00408897   .  EB 31                JMP SHORT NeoN_Rem.004088CA  

00408899   >  66:C745 B8 8000      MOV WORD PTR SS:[EBP-48],80  

0040889F   .  BA 01E55100          MOV EDX,NeoN_Rem.0051E501         ;  CB72392A  

004088A4   .  8D45 D0              LEA EAX,DWORD PTR SS:[EBP-30]  

004088A7   .  E8 408C0E00          CALL NeoN_Rem.004F14EC  

004088AC   .  FF45 C4              INC DWORD PTR SS:[EBP-3C]  

004088AF   .  8D55 D0              LEA EDX,DWORD PTR SS:[EBP-30]  

004088B2   .  8D45 F8              LEA EAX,DWORD PTR SS:[EBP-8]  

004088B5   .  E8 728E0E00          CALL NeoN_Rem.004F172C  

004088BA   .  FF4D C4              DEC DWORD PTR SS:[EBP-3C]  

004088BD   .  8D45 D0              LEA EAX,DWORD PTR SS:[EBP-30]  

004088C0   .  BA 02000000          MOV EDX,2  

004088C5   .  E8 328E0E00          CALL NeoN_Rem.004F16FC  

004088CA   >  8BC3                 MOV EAX,EBX  

004088CC   .  E8 4F870500          CALL NeoN_Rem.00461020  

004088D1   .  8BC3                 MOV EAX,EBX  

004088D3   .  E8 4CE40600          CALL NeoN_Rem.00476D24  

004088D8   .  FF4D C4              DEC DWORD PTR SS:[EBP-3C]  

004088DB   .  8D45 E8              LEA EAX,DWORD PTR SS:[EBP-18]  

004088DE   .  BA 02000000          MOV EDX,2  

004088E3   .  E8 148E0E00          CALL NeoN_Rem.004F16FC  

004088E8   .  FF4D C4              DEC DWORD PTR SS:[EBP-3C]  

004088EB   .  8D45 EC              LEA EAX,DWORD PTR SS:[EBP-14]  

004088EE   .  BA 02000000          MOV EDX,2  

004088F3   .  E8 048E0E00          CALL NeoN_Rem.004F16FC  

004088F8   .  66:C745 B8 2C00      MOV WORD PTR SS:[EBP-48],2C  

004088FE   .  66:C745 B8 1400      MOV WORD PTR SS:[EBP-48],14  
```



```
00408904   .  EB 0B                JMP SHORT NeoN_Rem.00408911  
00408906   .  66:C745 B8 3400     MOV WORD PTR SS:[EBP-48],34  
0040890C   .  E8 7D600E00         CALL NeoN_Rem.004EE98E  
00408911   >  8D55 F8             LEA EDX,DWORD PTR SS:[EBP-8]  
00408914   .  8D45 FC             LEA EAX,DWORD PTR SS:[EBP-4]  
00408917   .  E8 C48E0E00         CALL NeoN_Rem.004F17E0                ;  比较真假码  
0040891C   .  84C0                TEST AL,AL                           ;  AL=0 注册失败  
0040891E     74 36                JE SHORT NeoN_Rem.00408956  
```

## 二、算法验证

1. 累加用户名：`wzwgp` (0x777A776770) 累加和=23F   23F+1=240
2. 求余 `240 mod 5`(用户名位数)=1   1+A=B  得到注册码位数
3. `0*0=0`   `0+0=0`    `0 xor 240=240`   `240 mod 4A=3A`  `3A+30=6A` (j)  
   `1*1=1`   `1+1=2`    `2 xor 240=242`   `242 mod 4A=3C`  `3C+30=6C` (l)  
   `2*2=4`   `4+2=6`    `6 xor 240=246`   `246 mod 4A=3C`  `40+30=70` (p)  
   …………………………  
   `9*9=51`  `51+9=5A`  `5A xor 240=21A`  `21A mod 4A=14`  `14+30=44` (D)   
   `A*A=64`  `64+A=6E`  `6E xor 240=22E`  `22E mod 4A=28`  `28+30=58` (X)  
   计算结果不在`[0…9，a…z，A…Z]`范围内加 F。
4. 用户名：`wzwgp`   
   注册码：`jlpv4MJX2DX`
5. 注册信息保存在：`HKEY_CURRENT_USER\Software\NeoN Labs\NeoN Reminder`

## 三、注册机代码

Delphi7 编译通过

```
procedure TForm1.Button1Click(Sender: TObject);  
var sum,i,j,n,ip,s:integer;  
    k:string;  
    begin  
    sum := 1;  
    for i := 1 to Length(Edit1.Text) do  
    sum:=sum+(Ord(Edit1.Text[i]));  
    n:=((sum) mod (Length(Edit1.Text)))+$A;  
    for j := 0 to (n-1) do  
    begin  
    ip:=(j*j+j) xor sum ;  
    s:=(ip mod $4A) + $30;  
    if s in [$3A..$40,$5B..$60] then  
    s:=s+$F;  
    k:=k+chr(s);  
    end;  
    begin  
    Edit2.text:=(k);  
    end;  
    end;  
end. 

4dd466d0e1e0cc123afbe504da2462ca
```
