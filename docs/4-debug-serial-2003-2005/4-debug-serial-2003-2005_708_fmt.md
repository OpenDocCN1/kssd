
# CoolTV V2.4.002 的计算部分

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-216.htm`](https://www.pediy.com/kssd/pediy07/pediy7-216.htm)

【破文作者】 Rdsnow\[PYG\]  
【 E-mail 】 RDSNOW@163.COM  
【 作者 QQ 】 83757177  
【文章题目】 CoolTV 的计算部分  
【软件名称】 CoolTV V2.4.002  
【下载地址】 [`www2.skycn.com/soft/22907.html`](http://www2.skycn.com/soft/22907.html)  

----------------------------------------------------------------------------------------------  

【加密方式】 序列号、在线验证  
【破解工具】 FlyOD V1.10  
【软件限制】 功能限制  
【破解平台】 Windows XP SP2  

【文章简介】

程序已经升级，所以不会对作者造成影响，程序加了双保险，如果 `Cracker` 跟出注册码，程序会提示你注册成功，但是程序从新启动之后，会对注册码进行在线验证，最近我运气不好，连续碰到好几个这样的软件，现在着重看看这个程序的注册部分。

----------------------------------------------------------------------------------------------  

【破解过程】

脱掉程序 `Aspack2.12` 的外壳，用 `OD` 载入，输入用户名：`XiaoLing` 和假注册码:`abc987654321`，来到这儿：

```
004B536B  |.  55             PUSH EBP
004B536C  |.  68 73544B00    PUSH Unpacked.004B5473
004B5371  |.  64:FF30        PUSH DWORD PTR FS:[EAX]
004B5374  |.  64:8920        MOV DWORD PTR FS:[EAX],ESP
004B5377  |.  8D55 F8        LEA EDX,DWORD PTR SS:[EBP-8]
004B537A  |.  8B83 A4030000  MOV EAX,DWORD PTR DS:[EBX+3A4]
004B5380  |.  E8 3B65F8FF    CALL Unpacked.0043B8C0         ;  取出假注册码
004B5385  |.  8B45 F8        MOV EAX,DWORD PTR SS:[EBP-8]
004B5388  |.  50             PUSH EAX
004B5389  |.  8D55 F4        LEA EDX,DWORD PTR SS:[EBP-C]
004B538C  |.  8B83 94030000  MOV EAX,DWORD PTR DS:[EBX+394]
004B5392  |.  E8 2965F8FF    CALL Unpacked.0043B8C0
004B5397  |.  8B45 F4        MOV EAX,DWORD PTR SS:[EBP-C]
004B539A  |.  50             PUSH EAX
004B539B  |.  8D55 F0        LEA EDX,DWORD PTR SS:[EBP-10]
004B539E  |.  8B83 88030000  MOV EAX,DWORD PTR DS:[EBX+388]
004B53A4  |.  E8 1765F8FF    CALL Unpacked.0043B8C0         ;  取出用户名
004B53A9  |.  8B55 F0        MOV EDX,DWORD PTR SS:[EBP-10]
004B53AC  |.  8B83 A0030000  MOV EAX,DWORD PTR DS:[EBX+3A0]
004B53B2  |.  59             POP ECX
004B53B3  |.  E8 D0F1FFFF    CALL Unpacked.004B4588         ;  关键 call
004B53B8  |.  84C0           TEST AL,AL
004B53BA  |.  75 2C          JNZ SHORT Unpacked.004B53E8
004B53BC  |.  8D45 FC        LEA EAX,DWORD PTR SS:[EBP-4]
004B53BF  |.  BA 88544B00    MOV EDX,Unpacked.004B5488      ;  提示：未输入注册码或者输入的注册码不正确,请重新输入或者与软件供应商联系！
004B53C4  |.  E8 4BF3F4FF    CALL Unpacked.00404714
004B53C9  |.  6A 40          PUSH 40
004B53CB  |.  8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]
004B53CE  |.  E8 59F7F4FF    CALL Unpacked.00404B2C
004B53D3  |.  8BD0           MOV EDX,EAX
004B53D5  |.  B9 D4544B00    MOV ECX,Unpacked.004B54D4
004B53DA  |.  A1 ECE34D00    MOV EAX,DWORD PTR DS:[4DE3EC]
004B53DF  |.  8B00           MOV EAX,DWORD PTR DS:[EAX]
004B53E1  |.  E8 8E65FAFF    CALL Unpacked.0045B974
004B53E6  |.  EB 68          JMP SHORT Unpacked.004B5450
004B53E8  |>  68 E8544B00    PUSH Unpacked.004B54E8         ;  注册成功！\n 注册信息为：\n 用户姓名：
004B53ED  |.  8BB3 A0030000  MOV ESI,DWORD PTR DS:[EBX+3A0]
004B53F3  |.  FF76 48        PUSH DWORD PTR DS:[ESI+48]
004B53F6  |.  68 14554B00    PUSH Unpacked.004B5514
004B53FB  |.  68 20554B00    PUSH Unpacked.004B5520
004B5400  |.  FF76 54        PUSH DWORD PTR DS:[ESI+54]
004B5403  |.  68 14554B00    PUSH Unpacked.004B5514
004B5408  |.  68 34554B00    PUSH Unpacked.004B5534
004B540D  |.  FF76 5C        PUSH DWORD PTR DS:[ESI+5C]
004B5410  |.  68 14554B00    PUSH Unpacked.004B5514
004B5415  |.  68 48554B00    PUSH Unpacked.004B5548         ;  感谢您对我们的支持！请重新启动软件以验证注册码。
004B541A  |.  8D45 FC        LEA EAX,DWORD PTR SS:[EBP-4]
004B541D  |.  BA 0A000000    MOV EDX,0A
004B5422  |.  E8 CDF5F4FF    CALL Unpacked.004049F4
004B5427  |.  6A 40          PUSH 40
004B5429  |.  8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]
004B542C  |.  E8 FBF6F4FF    CALL Unpacked.00404B2C
004B5431  |.  8BD0           MOV EDX,EAX
004B5433  |.  B9 D4544B00    MOV ECX,Unpacked.004B54D4
004B5438  |.  A1 ECE34D00    MOV EAX,DWORD PTR DS:[4DE3EC]
004B543D  |.  8B00           MOV EAX,DWORD PTR DS:[EAX]
004B543F  |.  E8 3065FAFF    CALL Unpacked.0045B974
```

进入关键 `Call`:

```
004B4588  /$  55             PUSH EBP
004B4589  |.  8BEC           MOV EBP,ESP
004B458B  |.  83C4 F0        ADD ESP,-10
004B458E  |.  53             PUSH EBX
004B458F  |.  33DB           XOR EBX,EBX
004B4591  |.  895D F0        MOV DWORD PTR SS:[EBP-10],EBX
004B4594  |.  895D F4        MOV DWORD PTR SS:[EBP-C],EBX
004B4597  |.  894D F8        MOV DWORD PTR SS:[EBP-8],ECX
004B459A  |.  8955 FC        MOV DWORD PTR SS:[EBP-4],EDX
004B459D  |.  8BD8           MOV EBX,EAX
004B459F  |.  8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]
004B45A2  |.  E8 7505F5FF    CALL Unpacked.00404B1C
004B45A7  |.  8B45 F8        MOV EAX,DWORD PTR SS:[EBP-8]
004B45AA  |.  E8 6D05F5FF    CALL Unpacked.00404B1C
004B45AF  |.  8B45 08        MOV EAX,DWORD PTR SS:[EBP+8]
004B45B2  |.  E8 6505F5FF    CALL Unpacked.00404B1C
004B45B7  |.  33C0           XOR EAX,EAX
004B45B9  |.  55             PUSH EBP
004B45BA  |.  68 72464B00    PUSH Unpacked.004B4672
004B45BF  |.  64:FF30        PUSH DWORD PTR FS:[EAX]
004B45C2  |.  64:8920        MOV DWORD PTR FS:[EAX],ESP
004B45C5  |.  8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]
004B45C8  |.  E8 6703F5FF    CALL Unpacked.00404934         ;  用户名的长度
004B45CD  |.  3B43 4C        CMP EAX,DWORD PTR DS:[EBX+4C]
004B45D0  |.  7F 19          JG SHORT Unpacked.004B45EB     ;  用户名长度不大于 100
004B45D2  |.  8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]
004B45D5  |.  E8 5A03F5FF    CALL Unpacked.00404934
004B45DA  |.  3B43 50        CMP EAX,DWORD PTR DS:[EBX+50]
004B45DD  |.  7C 0C          JL SHORT Unpacked.004B45EB     ;  用户名长度不小于 3
004B45DF  |.  8B45 08        MOV EAX,DWORD PTR SS:[EBP+8]
004B45E2  |.  E8 4D03F5FF    CALL Unpacked.00404934
004B45E7  |.  85C0           TEST EAX,EAX
004B45E9  |.  75 04          JNZ SHORT Unpacked.004B45EF
004B45EB  |>  33DB           XOR EBX,EBX
004B45ED  |.  EB 60          JMP SHORT Unpacked.004B464F
004B45EF  |>  8D55 F4        LEA EDX,DWORD PTR SS:[EBP-C]
004B45F2  |.  8B45 08        MOV EAX,DWORD PTR SS:[EBP+8]
004B45F5  |.  E8 AA43F5FF    CALL Unpacked.004089A4         ;  注册码转换为大写
004B45FA  |.  8B55 F4        MOV EDX,DWORD PTR SS:[EBP-C]
004B45FD  |.  8D45 08        LEA EAX,DWORD PTR SS:[EBP+8]
004B4600  |.  E8 0F01F5FF    CALL Unpacked.00404714
004B4605  |.  8D4D F0        LEA ECX,DWORD PTR SS:[EBP-10]
004B4608  |.  8B55 FC        MOV EDX,DWORD PTR SS:[EBP-4]
004B460B  |.  8BC3           MOV EAX,EBX
004B460D  |.  E8 BAFBFFFF    CALL Unpacked.004B41CC         ;  通过用户名计算得到"00780B37BE"
004B4612  |.  8B45 F0        MOV EAX,DWORD PTR SS:[EBP-10]
004B4615  |.  8B55 08        MOV EDX,DWORD PTR SS:[EBP+8]
004B4618  |.  E8 FF43F5FF    CALL Unpacked.00408A1C
004B461D  |.  85C0           TEST EAX,EAX                   ;  真假注册码进行比较
004B461F  |.  74 04          JE SHORT Unpacked.004B4625     ;  关键跳，跳走就让此 CALL 返回 1，不跳则返回 0
004B4621  |.  33DB           XOR EBX,EBX
004B4623  |.  EB 2A          JMP SHORT Unpacked.004B464F
004B4625  |>  8D43 48        LEA EAX,DWORD PTR DS:[EBX+48]
004B4628  |.  8B55 FC        MOV EDX,DWORD PTR SS:[EBP-4]
004B462B  |.  E8 A000F5FF    CALL Unpacked.004046D0
004B4630  |.  8D43 54        LEA EAX,DWORD PTR DS:[EBX+54]
004B4633  |.  8B55 F8        MOV EDX,DWORD PTR SS:[EBP-8]
004B4636  |.  E8 9500F5FF    CALL Unpacked.004046D0
004B463B  |.  8D43 5C        LEA EAX,DWORD PTR DS:[EBX+5C]
004B463E  |.  8B55 08        MOV EDX,DWORD PTR SS:[EBP+8]
004B4641  |.  E8 8A00F5FF    CALL Unpacked.004046D0
004B4646  |.  8BC3           MOV EAX,EBX
004B4648  |.  E8 DF010000    CALL Unpacked.004B482C
004B464D  |.  B3 01          MOV BL,1
004B464F  |>  33C0           XOR EAX,EAX
004B4651  |.  5A             POP EDX
```



```
004B4652   |.   59             POP ECX
004B4653   |.   59             POP ECX
004B4654   |.   64:8910        MOV DWORD PTR FS:[EAX],EDX
004B4657   |.   68 79464B00    PUSH Unpacked.004B4679
004B465C   |>   8D45 F0        LEA EAX,DWORD PTR SS:[EBP-10]
004B465F   |.   BA 04000000    MOV EDX,4
004B4664   |.   E8 3700F5FF    CALL Unpacked.004046A0
004B4669   |.   8D45 08        LEA EAX,DWORD PTR SS:[EBP+8]
004B466C   |.   E8 0B00F5FF    CALL Unpacked.0040467C
004B4671   \.   C3             RETN
```

如何将用户名 `"XiaoLing"` 计算得到注册码 `"00780B37BE"` 的呢？要知道答案，进入 `CALL 004B41CC`

```
004B41CC   /$   55             PUSH EBP
004B41CD   |.   8BEC           MOV EBP,ESP
004B41CF   |.   83C4 E0        ADD ESP,-20
004B41D2   |.   53             PUSH EBX
004B41D3   |.   56             PUSH ESI
004B41D4   |.   57             PUSH EDI
004B41D5   |.   33DB           XOR EBX,EBX
004B41D7   |.   895D EC        MOV DWORD PTR SS:[EBP-14],EBX
004B41DA   |.   895D E8        MOV DWORD PTR SS:[EBP-18],EBX
004B41DD   |.   895D E4        MOV DWORD PTR SS:[EBP-1C],EBX
004B41E0   |.   895D E0        MOV DWORD PTR SS:[EBP-20],EBX
004B41E3   |.   8BF9           MOV EDI,ECX
004B41E5   |.   8955 FC        MOV DWORD PTR SS:[EBP-4],EDX
004B41E8   |.   8BF0           MOV ESI,EAX
004B41EA   |.   8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]
004B41ED   |.   E8 2A09F5FF    CALL Unpacked.00404B1C
004B41F2   |.   33C0           XOR EAX,EAX
004B41F4   |.   55             PUSH EBP
004B41F5   |.   68 65434B00    PUSH Unpacked.004B4365
004B41FA   |.   64:FF30        PUSH DWORD PTR FS:[EAX]
004B41FD   |.   64:8920        MOV DWORD PTR FS:[EAX],ESP
004B4200   |.   8D45 EC        LEA EAX,DWORD PTR SS:[EBP-14]
004B4203   |.   BA 7C434B00    MOV EDX,Unpacked.004B437C        ;  ASCII "gf258369gf"
004B4208   |.   E8 0705F5FF    CALL Unpacked.00404714
004B420D   |.   837D EC 00     CMP DWORD PTR SS:[EBP-14],0
004B4211   |.   75 0D          JNZ SHORT Unpacked.004B4220
004B4213   |.   8D45 E0        LEA EAX,DWORD PTR SS:[EBP-20]
004B4216   |.   8B55 FC        MOV EDX,DWORD PTR SS:[EBP-4]
004B4219   |.   E8 F604F5FF    CALL Unpacked.00404714
004B421E   |.   EB 5D          JMP SHORT Unpacked.004B427D
004B4220   |>   8B45 EC        MOV EAX,DWORD PTR SS:[EBP-14]
004B4223   |.   E8 0C07F5FF    CALL Unpacked.00404934
004B4228   |.   8BD8           MOV EBX,EAX
004B422A   |.   8D45 E8        LEA EAX,DWORD PTR SS:[EBP-18]
004B422D   |.   50             PUSH EAX
004B422E   |.   8BCB           MOV ECX,EBX
004B4230   |.   D1F9           SAR ECX,1
004B4232   |.   79 03         JNS SHORT Unpacked.004B4237
004B4234   |.   83D1 00        ADC ECX,0
004B4237   |>   BA 01000000    MOV EDX,1
004B423C   |.   8B45 EC        MOV EAX,DWORD PTR SS:[EBP-14]
004B423F   |.   E8 4809F5FF    CALL Unpacked.00404B8C           ;  取出"gf258369gf"前面五个字符"gf258"
004B4244   |.   8D45 E4        LEA EAX,DWORD PTR SS:[EBP-1C]
004B4247   |.   50             PUSH EAX
004B4248   |.   8BC3           MOV EAX,EBX
004B424A   |.   D1F8           SAR EAX,1
004B424C   |.   79 03         JNS SHORT Unpacked.004B4251
004B424E   |.   83D0 00        ADC EAX,0
004B4251   |>   8BCB           MOV ECX,EBX
004B4253   |.   2BC8           SUB ECX,EAX
004B4255   |.   8BD3           MOV EDX,EBX
004B4257   |.   D1FA           SAR EDX,1
004B4259   |.   79 03         JNS SHORT Unpacked.004B425E
004B425B   |.   83D2 00        ADC EDX,0
004B425E   |>   42             INC EDX
004B425F   |.   8B45 EC        MOV EAX,DWORD PTR SS:[EBP-14]
004B4262   |.   E8 2509F5FF    CALL Unpacked.00404B8C           ;  取出"gf258369gf"后面五个字符"369gf"
004B4267   |.   FF75 E8        PUSH DWORD PTR SS:[EBP-18]
004B426A   |.   FF75 FC        PUSH DWORD PTR SS:[EBP-4]
004B426D   |.   FF75 E4        PUSH DWORD PTR SS:[EBP-1C]
004B4270   |.   8D45 E0        LEA EAX,DWORD PTR SS:[EBP-20]
004B4273   |.   BA 03000000    MOV EDX,3
004B4278   |.   E8 7707F5FF    CALL Unpacked.004049F4           ;  将"gf258"、用户名、 "369gf"连接成"gf258XiaoLing369gf"
004B427D   |>   C745 F0 00000>MOV DWORD PTR SS:[EBP-10],0
004B4284   |.   C745 F4 00000>MOV DWORD PTR SS:[EBP-C],0
004B428B   |.   8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]
004B428E   |.   E8 A106F5FF    CALL Unpacked.00404934
004B4293   |.   3B46 4C        CMP EAX,DWORD PTR DS:[ESI+4C]
004B4296   |.   7F 0D          JG SHORT Unpacked.004B42A5       ;  用户名长度不超过 100
004B4298   |.   8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]
004B429B   |.   E8 9406F5FF    CALL Unpacked.00404934
004B42A0   |.   3B46 50        CMP EAX,DWORD PTR DS:[ESI+50]
004B42A3   |.   7D 0C          JGE SHORT Unpacked.004B42B1      ;  用户名长度大于 3
004B42A5   |>   8BC7           MOV EAX,EDI
004B42A7   |.   E8 D003F5FF    CALL Unpacked.0040467C
004B42AC   |.   E9 91000000    JMP Unpacked.004B4342
004B42B1   |>   8B45 E0        MOV EAX,DWORD PTR SS:[EBP-20]
004B42B4   |.   E8 7B06F5FF    CALL Unpacked.00404934           ;  取变形后用户名"gf258XiaoLing369gf"的长度
004B42B9   |.   8BD8           MOV EBX,EAX                      ;  i=用户名长度，准备循环
004B42BB   |.   EB 37          JMP SHORT Unpacked.004B42F4
004B42BD   |>   8B46 68        /MOV EAX,DWORD PTR DS:[ESI+68]    ;  0x6AB4A67
004B42C0   |.   8B56 6C        |MOV EDX,DWORD PTR DS:[ESI+6C]
004B42C3   |.   0345 F0        |ADD EAX,DWORD PTR SS:[EBP-10]    ;  累加求和
004B42C6   |.   1355 F4        |ADC EDX,DWORD PTR SS:[EBP-C]
004B42C9   |.   52             |PUSH EDX
004B42CA   |.   50             |PUSH EAX
004B42CB   |.   8B45 E0        |MOV EAX,DWORD PTR SS:[EBP-20]
004B42CE   |.   0FB64418 FF    |MOVZX EAX,BYTE PTR DS:[EAX+EBX-1]  ;  EAX=ASC[i]
004B42D3   |.   50             |PUSH EAX
004B42D4   |.   B8 59040000    |MOV EAX,459
004B42D9   |.   5A             |POP EDX
004B42DA   |.   8BCA           |MOV ECX,EDX
004B42DC   |.   33D2           |XOR EDX,EDX
004B42DE   |.   F7F1           |DIV ECX                     ;  0x459 mod ASC[i]
004B42E0   |.   8BC2           |MOV EAX,EDX
004B42E2   |.   33D2           |XOR EDX,EDX
004B42E4   |.   290424         |SUB DWORD PTR SS:[ESP],EAX     ;  0x6AB4A67-余数
004B42E7   |.   195424 04      |SBB DWORD PTR SS:[ESP+4],EDX
004B42EB   |.   58             |POP EAX
004B42EC   |.   5A             |POP EDX
004B42ED   |.   8945 F0        |MOV DWORD PTR SS:[EBP-10],EAX
004B42F0   |.   8955 F4        |MOV DWORD PTR SS:[EBP-C],EDX
004B42F3   |.   4B             |DEC EBX                     ;  i-1
004B42F4   |>   8B45 E0        MOV EAX,DWORD PTR SS:[EBP-20]
004B42F7   |.   E8 3806F5FF    |CALL Unpacked.00404934
004B42FC   |.   3BD8           |CMP EBX,EAX
004B42FE   |.   7F 04          |JG SHORT Unpacked.004B4304
004B4300   |.   85DB           |TEST EBX,EBX
004B4302   |.^ 7F B9          \\JG SHORT Unpacked.004B42BD      ;  循坏，用 0x6AB4A67-0x459%ASC[i]，然后累加求和
004B4304   |>   8B5E 60        MOV EBX,DWORD PTR DS:[ESI+60]
004B4307   |.   85DB           TEST EBX,EBX
004B4309   |.   7F 11          JG SHORT Unpacked.004B431C
004B430B   |.   FF75 F4        PUSH DWORD PTR SS:[EBP-C]        ;  /Arg2
004B430E   |.   FF75 F0        PUSH DWORD PTR SS:[EBP-10]       ;  |Arg1
004B4311   |.   8BD7           MOV EDX,EDI                       ;  |
004B4313   |.   33C0           XOR EAX,EAX                       ;  |
004B4315   |.   E8 F249F5FF    CALL Unpacked.00408D0C           ;  \\Unpacked.00408D0C
004B431A   |.   EB 26          JMP SHORT Unpacked.004B4342
004B431C   |>   FF75 F4        PUSH DWORD PTR SS:[EBP-C]        ;  /Arg2
004B431F   |.   FF75 F0        PUSH DWORD PTR SS:[EBP-10]       ;  |Arg1
004B4322   |.   8BD7           MOV EDX,EDI                       ;  |
004B4324   |.   8BC3           MOV EAX,EBX                       ;  |转换为长度为 10 的字符串得到"00780B37BE",他就是注册码了
004B4326   |.   E8 E149F5FF    CALL Unpacked.00408D0C           ;  \\Unpacked.00408D0C
004B432B   |.   8B07           MOV EAX,DWORD PTR DS:[EDI]
004B432D   |.   E8 0206F5FF    CALL Unpacked.00404934
004B4332   |.   8BC8           MOV ECX,EAX
004B4334   |.   2B4E 60        SUB ECX,DWORD PTR DS:[ESI+60]
```



```
004B4337  |.  8B56 60       MOV EDX,DWORD PTR DS:[ESI+60]
004B433A  |.  42            INC EDX
004B433B  |.  8BC7          MOV EAX,EDI
004B433D  |.  E8 8A08F5FF   CALL Unpacked.00404BCC
004B4342  |>  33C0          XOR EAX,EAX
004B4344  |.  5A            POP EDX
004B4345  |.  59            POP ECX
004B4346  |.  59            POP ECX
004B4347  |.  64:8910       MOV DWORD PTR FS:[EAX],EDX
004B434A  |.  68 6C434B00   PUSH Unpacked.004B436C
004B434F  |>  8D45 E0       LEA EAX,DWORD PTR SS:[EBP-20]
004B4352  |.  BA 04000000   MOV EDX,4
004B4357  |.  E8 4403F5FF   CALL Unpacked.004046A0
004B435C  |.  8D45 FC       LEA EAX,DWORD PTR SS:[EBP-4]
004B435F  |.  E8 1803F5FF   CALL Unpacked.0040467C
004B4364  \\.  C3            RETN
004B4365   .^ E9 96FCF4FF   JMP Unpacked.00404000
004B436A   .^ EB E3         JMP SHORT Unpacked.004B434F
004B436C   .  5F            POP EDI
004B436D   .  5E            POP ESI
004B436E   .  5B            POP EBX
004B436F   .  8BE5          MOV ESP,EBP
004B4371   .  5D            POP EBP
004B4372   .  C3            RETN
```

## 破解心得

大致注册过程是这样的，将用户名`XiaoLing`插入`gf258369gf`中间，得到`gf258XiaoLing369gf`，然后对每一个字符用`0x6AB4A67 - 0x459 mod ASC[i]`计算，将结果累加求和，得到的结果`0x780B37BE`，转换为长度为 10 的十六进制文本，不足十位前面加两个`0`，就得到注册码`00780B37BE`。

不要高兴太早，程序重启后，经过在线验证，显示为`未授权`，将下面代码中的`JE 004B5BEE`改为`JMP 004B5BEE`跳过

```
004B58CA  |> \8B86 A0030000 MOV EAX,DWORD PTR DS:[ESI+3A0]
004B58D0  |.  E8 EFE8FFFF   CALL Unpacked.004B41C4
004B58D5  |.  84C0          TEST AL,AL
004B58D7     0F84 11030000 JE Unpacked.004B5BEE
```

内存注册机：

- 中断地址：`4B4615`
- 中断次数：`1`
- 第一字节：`8B`
- 指令长度：`3`

## 破解声明

我是一只小菜鸟，偶得一点心得，愿与大家分享:)

## 版权声明

本文纯属技术交流，转载请注明作者并保持文章的完整，谢谢!

文章写于 2005-5-10 22:58:20

`4dd466d0e1e0cc123afbe504da2462ca`
