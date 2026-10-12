
# 菜鸟分析宏达公安机关档案管理系统 1.0 算法

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-230.htm`](https://www.pediy.com/kssd/pediy07/pediy7-230.htm)

## 破文信息

- **【破文标题】**: 宏达公安机关档案管理系统 1.0 算法分析
- **【破文作者】**: CDboy
- **【作者邮箱】**: cdboy_cn@yahoo.com.cn
- **【使用工具】**: 二哥修改版 OD，PEiD
- **【软件名称】**: 宏达公安机关档案管理系统 1.0
- **【下载地址】**: [http://sccrc.onlinedown.net/soft/37938.htm](http://sccrc.onlinedown.net/soft/37938.htm)
- **【保护方式】**: 功能限制
- **【软件语言】**: Delphi
- **【软件简介】**: 轻松管理维护公安机关相关档案，对专业档案、户籍档案、重点人口卷、备查档案、文书档案、报表等进行全面管理
- **【破解声明】**: 菜鸟学习技术，仅此而已

## 破解过程

看看软件名，好像是警察叔叔用的，搞这个不违法吧，吓我一跳 ^_^

用 `PEiD` 检测是 `ASPack 2.12` 的壳，`ASPackDie` 脱之

再检测是 `Delphi` 的

用 `OD` 加载 输入注册名注册码 `123456` 后 下断点 `bp MessageBoxA`

```
77D3B064   >  833D D0C3D677 0>  CMP DWORD PTR DS:[77D6C3D0],0
77D3B06B    0F85 405B0100     JNZ user32.77D50BB1
77D3B071    6A 00             PUSH 0
77D3B073    FF7424 14         PUSH DWORD PTR SS:[ESP+14]
77D3B077    FF7424 14         PUSH DWORD PTR SS:[ESP+14]
77D3B07B    FF7424 14         PUSH DWORD PTR SS:[ESP+14]
77D3B07F    FF7424 14         PUSH DWORD PTR SS:[ESP+14]
77D3B083    E8 03000000       CALL user32.MessageBoxExA
77D3B088    C2 1000           RETN 10
```

看堆栈

```
0012EF9C   004AED27  /CALL 到 MessageBoxA 来自 unpacked.004AED22   //直接右键-反汇编中跟随
0012EFA0   001C0220  |hOwner = 001C0220 ('公安机关档案管理系统(非注册用户)',class='TApplication')
0012EFA4   0060B1E4  |Text = "注册码失败！请核对注册名与注册码。"
0012EFA8   0060B198  |Title = "用户注册"
0012EFAC   00000000  \Style = MB_OK|MB_APPLMODAL
```

来到

```
004AED22   |.  E8 4D94F5FF   CALL <JMP.&user32.MessageBoxA>        ; \\MessageBoxA
004AED27   |.  8945 F8       MOV DWORD PTR SS:[EBP-8],EAX
004AED2A   |.  33C0          XOR EAX,EAX
004AED2C   |.  5A            POP EDX
004AED2D   |.  59            POP ECX
004AED2E   |.  59            POP ECX
......省略部分代码
004AED83   |.  E8 AC94F5FF   CALL <JMP.&user32.SetActiveWindow>    ; \\SetActiveWindow
004AED88   |.  8B45 E4       MOV EAX,DWORD PTR SS:[EBP-1C]
004AED8B   |.  E8 EC68FFFF   CALL unpacked.004A567C
004AED90   \.  C3            RETN
004AED91    .^ E9 DA56F5FF   JMP unpacked.00404470
004AED96    .^ EB 9F         JMP SHORT unpacked.004AED37
004AED98    .  8B45 F8       MOV EAX,DWORD PTR SS:[EBP-8]
004AED9B    .  5F            POP EDI
004AED9C    .  5E            POP ESI
004AED9D    .  5B            POP EBX
004AED9E    .  8BE5          MOV ESP,EBP
004AEDA0    .  5D            POP EBP
004AEDA1    .  C2 0400       RETN 4    //单步来到这里就返回了
```

返回到这里

```
0060B058   .  B9 98B16000   MOV ECX,unpacked.0060B198
0060B05D   .  BA E4B16000   MOV EDX,unpacked.0060B1E4
0060B062   .  A1 287C6A00   MOV EAX,DWORD PTR DS:[6A7C28]
0060B067   .  8B00          MOV EAX,DWORD PTR DS:[EAX]
0060B069   .  E8 CE3BEAFF   CALL unpacked.004AEC3C  
0060B06E   >  33C0          XOR EAX,EAX      //返回到这里 那么我们向上看，上面就是验证注册码的地方
```

0060AC8C

```
0060AC8C   .  55            PUSH EBP        //这里下断
0060AC8D   .  8BEC          MOV EBP,ESP
0060AC8F   .  B9 0D000000   MOV ECX,0D
0060AC94   >  6A 00         PUSH 0
0060AC96   .  6A 00         PUSH 0
0060AC98   .  49            DEC ECX
0060AC99   .^ 75 F9         JNZ SHORT unpacked.0060AC94
0060AC9B   .  51            PUSH ECX
0060AC9C   .  53            PUSH EBX
0060AC9D   .  56            PUSH ESI
0060AC9E   .  57            PUSH EDI
0060AC9F   .  8BD8          MOV EBX,EAX
0060ACA1   .  33C0          XOR EAX,EAX
0060ACA3   .  55            PUSH EBP
0060ACA4   .  68 35B16000   PUSH unpacked.0060B135
0060ACA9   .  64:FF30       PUSH DWORD PTR FS:[EAX]
0060ACAC   .  64:8920       MOV DWORD PTR FS:[EAX],ESP
0060ACAF   .  B2 01        MOV DL,1
0060ACB1   .  A1 F0E04400   MOV EAX,DWORD PTR DS:[44E0F0]
0060ACB6   .  E8 3535E4FF   CALL unpacked.0044E1F0
0060ACBB   .  8945 FC       MOV DWORD PTR SS:[EBP-4],EAX
0060ACBE   .  33C0          XOR EAX,EAX
0060ACC0   .  55            PUSH EBP
0060ACC1   .  68 3EB06000   PUSH unpacked.0060B03E
0060ACC6   .  64:FF30       PUSH DWORD PTR FS:[EAX]
0060ACC9   .  64:8920       MOV DWORD PTR FS:[EAX],ESP
0060ACCC   .  8D55 EC       LEA EDX,DWORD PTR SS:[EBP-14]
0060ACCF   .  8B83 08030000 MOV EAX,DWORD PTR DS:[EBX+308]
0060ACD5   .  E8 662CE8FF   CALL unpacked.0048D940    //取假码
0060ACDA   .  8B45 EC       MOV EAX,DWORD PTR SS:[EBP-14]  //[EBP-14]=EAX
0060ACDD   .  8D55 F0       LEA EDX,DWORD PTR SS:[EBP-10]
0060ACE0   .  E8 ABECDFFF   CALL unpacked.00409990
0060ACE5   .  8B45 F0       MOV EAX,DWORD PTR SS:[EBP-10]  //[EBP-10]=EAX=假码
0060ACE8   .  8D4D F4       LEA ECX,DWORD PTR SS:[EBP-C]
0060ACEB   .  BA 50B16000   MOV EDX,unpacked.0060B150     //字符 HDDBIP 入栈          
0060ACF0   .  E8 A37A0600   CALL unpacked.00672798    //关键 CALL1 跟进
0060ACF5   .  8B45 F4       MOV EAX,DWORD PTR SS:[EBP-C]  //[EBP-C]地址 CED738 进 EAX     
0060ACF8   .  50            PUSH EAX
0060ACF9   .  8D55 E4       LEA EDX,DWORD PTR SS:[EBP-1C]
0060ACFC   .  8B83 04030000 MOV EAX,DWORD PTR DS:[EBX+304]
0060AD02   .  E8 392CE8FF   CALL unpacked.0048D940    //取注册名
0060AD07   .  8B45 E4       MOV EAX,DWORD PTR SS:[EBP-1C]  //[EBP-1C]=EAX=注册名
0060AD0A   .  8D55 E8       LEA EDX,DWORD PTR SS:[EBP-18]
0060AD0D   .  E8 7EECDFFF   CALL unpacked.00409990    //进入堆栈
0060AD12   .  8B55 E8       MOV EDX,DWORD PTR SS:[EBP-18]  //[EBP-18]=注册名新地址
0060AD15   .  58            POP EAX
0060AD16   .  E8 C1A2DFFF   CALL unpacked.00404FDC    //关键 CALL2 跟进 这里比较的是注册名为 6 位的情况
0060AD1B   .  0F85 EA000000 JNZ unpacked.0060AE0B    //跳
0060AD21   .  33C0          XOR EAX,EAX
0060AD23   .  55            PUSH EBP
0060AD24   .  68 FCAD6000   PUSH unpacked.0060ADFC
0060AD29   .  64:FF30       PUSH DWORD PTR FS:[EAX]
0060AD2C   .  64:8920       MOV DWORD PTR FS:[EAX],ESP
0060AD2F   .  BA 02000080   MOV EDX,80000002
0060AD34   .  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
0060AD37   .  E8 5435E4FF   CALL unpacked.0044E290
0060AD3C   .  B1 01        MOV CL,1
0060AD3E   .  BA 60B16000   MOV EDX,unpacked.0060B160          ;  ASCII "SoftWare\\Dbimp\\Dbimp1.0"
0060AD43   .  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
0060AD46   .  E8 A935E4FF   CALL unpacked.0044E2F4
0060AD4B   .  84C0          TEST AL,AL
0060AD4D   .  74 29         JE SHORT unpacked.0060AD78
0060AD4F   .  8D55 DC       LEA EDX,DWORD PTR SS:[EBP-24]
0060AD52   .  8B83 04030000 MOV EAX,DWORD PTR DS:[EBX+304]
0060AD58   .  E8 E32BE8FF   CALL unpacked.0048D940
0060AD5D   .  8B45 DC       MOV EAX,DWORD PTR SS:[EBP-24]
0060AD60   .  8D55 E0       LEA EDX,DWORD PTR SS:[EBP-20]
0060AD63   .  E8 28ECDFFF   CALL unpacked.00409990
0060AD68   .  8B4D E0       MOV ECX,DWORD PTR SS:[EBP-20]
0060AD6B   .  BA 80B16000   MOV EDX,unpacked.0060B180          ;  ASCII "RegName"
0060AD70   .  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
0060AD73   .  E8 1837E4FF   CALL unpacked.0044E490
0060AD78   >  8D55 D4       LEA EDX,DWORD PTR SS:[EBP-2C]
0060AD7B   .  8B83 08030000 MOV EAX,DWORD PTR DS:[EBX+308]
0060AD81   .  E8 BA2BE8FF   CALL unpacked.0048D940
0060AD86   .  8B45 D4       MOV EAX,DWORD PTR SS:[EBP-2C]
0060AD89   .  8D55 D8       LEA EDX,DWORD PTR SS:[EBP-28]
0060AD8C   .  E8 FFEBDFFF   CALL unpacked.00409990
0060AD91   .  8B4D D8       MOV ECX,DWORD PTR SS:[EBP-28]
0060AD94   .  BA 90B16000   MOV EDX,unpacked.0060B190          ;  ASCII "RegID"
0060AD99   .  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
0060AD9C   .  E8 EF36E4FF   CALL unpacked.0044E490
0060ADA1   .  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
0060ADA4   .  E8 B734E4FF   CALL unpacked.0044E260
0060ADA9   .  6A 00         PUSH 0
0060ADAB   .  B9 98B16000   MOV ECX,unpacked.0060B198
0060ADB0   .  BA A4B16000   MOV EDX,unpacked.0060B1A4
0060ADB5   .  A1 287C6A00   MOV EAX,DWORD PTR DS:[6A7C28]
0060ADBA   .  8B00          MOV EAX,DWORD PTR DS:[EAX]
0060ADBC   .  E8 7B3EEAFF   CALL unpacked.004AEC3C
```



0060ADC1   .  8B83 F8020000 MOV EAX,DWORD PTR DS:[EBX+2F8]

0060ADC7   .  33D2           XOR EDX,EDX

0060ADC9   .  8B08           MOV ECX,DWORD PTR DS:[EAX]

0060ADCB   .  FF51 64        CALL DWORD PTR DS:[ECX+64]

0060ADCE   .  A1 58776A00    MOV EAX,DWORD PTR DS:[6A7758]

0060ADD3   .  C640 28 01     MOV BYTE PTR DS:[EAX+28],1

0060ADD7   .  8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]

0060ADDA   .  E8 FD8EDFFF    CALL unpacked.00403CDC

0060ADDF   .  33C0           XOR EAX,EAX

0060ADE1   .  5A             POP EDX

0060ADE2   .  59             POP ECX

0060ADE3   .  59             POP ECX

0060ADE4   .  64:8910        MOV DWORD PTR FS:[EAX],EDX

0060ADE7   .  33C0           XOR EAX,EAX

0060ADE9   .  5A             POP EDX

0060ADEA   .  59             POP ECX

0060ADEB   .  59             POP ECX

0060ADEC   .  64:8910        MOV DWORD PTR FS:[EAX],EDX

0060ADEF   .  E9 7A020000    JMP unpacked.0060B06E

0060ADF4   .  33C0           XOR EAX,EAX

0060ADF6   .  5A             POP EDX

0060ADF7   .  59             POP ECX

0060ADF8   .  59             POP ECX

0060ADF9   .  64:8910        MOV DWORD PTR FS:[EAX],EDX

0060ADFC   .^ E9 BB93DFFF    JMP unpacked.004041BC

0060AE01   .  E8 E297DFFF     CALL unpacked.004045E8

0060AE06   .  E9 29020000     JMP unpacked.0060B034

0060AE0B   >  A1 58776A00     MOV EAX,DWORD PTR DS:[6A7758]

0060AE10   .  8338 00         CMP DWORD PTR DS:[EAX],0    //比较文件名

0060AE13   .  0F84 1B020000  JE unpacked.0060B034    //为空则跳

0060AE19   .  8D55 F8         LEA EDX,DWORD PTR SS:[EBP-8]

0060AE1C   .  A1 58776A00     MOV EAX,DWORD PTR DS:[6A7758]

0060AE21   .  8B00           MOV EAX,DWORD PTR DS:[EAX]    //数据库地址进 EAX

0060AE23   .  E8 B8F7DFFF     CALL unpacked.0040A5E0

0060AE28   .  8D45 F8         LEA EAX,DWORD PTR SS:[EBP-8]

0060AE2B   .  50             PUSH EAX

0060AE2C   .  8B55 F8         MOV EDX,DWORD PTR SS:[EBP-8]  //数据库名移入 EDX

0060AE2F   .  B8 BCB16000     MOV EAX,unpacked.0060B1BC    //先定义的数据进 EAX

0060AE34   .  E8 9BA3DFFF     CALL unpacked.004051D4    //取注册名位数

0060AE39   .  8BC8           MOV ECX,EAX        //位数进 ECX

0060AE3B   .  49             DEC ECX        //ECX-1

0060AE3C   .  BA 01000000     MOV EDX,1        //EDX=1

0060AE41   .  8B45 F8         MOV EAX,DWORD PTR SS:[EBP-8]  //数据库名进 EAX

0060AE44   .  E8 A7A2DFFF     CALL unpacked.004050F0    //数据库名保存到堆栈 12f0a4

0060AE49   .  33C0           XOR EAX,EAX

0060AE4B   .  55             PUSH EBP

0060AE4C   .  68 2AB06000     PUSH unpacked.0060B02A

0060AE51   .  64:FF30        PUSH DWORD PTR FS:[EAX]

0060AE54   .  64:8920        MOV DWORD PTR FS:[EAX],ESP

0060AE57   .  8D55 C8         LEA EDX,DWORD PTR SS:[EBP-38]

0060AE5A   .  8B83 08030000   MOV EAX,DWORD PTR DS:[EBX+308]

0060AE60   .  E8 DB2AE8FF     CALL unpacked.0048D940    //取假码

0060AE65   .  8B45 C8         MOV EAX,DWORD PTR SS:[EBP-38]  //[EBP-38]=EAX=假码

0060AE68   .  8D55 CC         LEA EDX,DWORD PTR SS:[EBP-34]

0060AE6B   .  E8 20EBDFFF     CALL unpacked.00409990    //假码进堆栈 12F078

0060AE70   .  8B45 CC         MOV EAX,DWORD PTR SS:[EBP-34]  //[EBP-34]=EAX 新地址

0060AE73   .  8D4D D0         LEA ECX,DWORD PTR SS:[EBP-30]

0060AE76   .  8B55 F8         MOV EDX,DWORD PTR SS:[EBP-8]  //[EBP-8]数据库名 Pams 进 EDX

0060AE79   .  E8 1A790600     CALL unpacked.00672798    //关键 CALL1 跟进

0060AE7E   .  8B45 D0         MOV EAX,DWORD PTR SS:[EBP-30]

0060AE81   .  50             PUSH EAX

0060AE82   .  8D55 C0         LEA EDX,DWORD PTR SS:[EBP-40]

0060AE85   .  8B83 04030000   MOV EAX,DWORD PTR DS:[EBX+304]

0060AE8B   .  E8 B02AE8FF     CALL unpacked.0048D940    //取注册名

0060AE90   .  8B45 C0         MOV EAX,DWORD PTR SS:[EBP-40]  //[EBP-40]=EAX

0060AE93   .  8D55 C4         LEA EDX,DWORD PTR SS:[EBP-3C]

0060AE96   .  E8 F5EADFFF     CALL unpacked.00409990    //存入堆栈

0060AE9B   .  8B55 C4         MOV EDX,DWORD PTR SS:[EBP-3C]  //[EBP-3C]=EDX 注册名新地址

0060AE9E   .  58             POP EAX

0060AE9F   .  E8 38A1DFFF     CALL unpacked.00404FDC    //关键的 CALL2 跟进 这里比较的是注册名是 4 位的情况

0060AEA4   .  74 63          JE SHORT unpacked.0060AF09

0060AEA6   .  8D55 B4         LEA EDX,DWORD PTR SS:[EBP-4C]

0060AEA9   .  8B83 08030000   MOV EAX,DWORD PTR DS:[EBX+308]

0060AEAF   .  E8 8C2AE8FF     CALL unpacked.0048D940    //取假码

0060AEB4   .  8B45 B4         MOV EAX,DWORD PTR SS:[EBP-4C]

0060AEB7   .  8D55 B8         LEA EDX,DWORD PTR SS:[EBP-48]

0060AEBA   .  E8 D1EADFFF     CALL unpacked.00409990

0060AEBF   .  8B45 B8         MOV EAX,DWORD PTR SS:[EBP-48]  //[EBP-48]=EAX 假码新地址

0060AEC2   .  8D4D BC         LEA ECX,DWORD PTR SS:[EBP-44]

0060AEC5   .  8B55 F8         MOV EDX,DWORD PTR SS:[EBP-8]  //[EBP-8]库名进 EDX

0060AEC8   .  E8 CB780600     CALL unpacked.00672798    //关键 CALL1 跟进

0060AECD   .  8B45 BC         MOV EAX,DWORD PTR SS:[EBP-44]

0060AED0   .  50             PUSH EAX

0060AED1   .  8D55 A8         LEA EDX,DWORD PTR SS:[EBP-58]

0060AED4   .  8B83 04030000   MOV EAX,DWORD PTR DS:[EBX+304]

0060AEDA   .  E8 612AE8FF     CALL unpacked.0048D940    //取注册名

0060AEDF   .  8B4D A8         MOV ECX,DWORD PTR SS:[EBP-58]  //[EBP-58]=ECX

0060AEE2   .  8D45 AC         LEA EAX,DWORD PTR SS:[EBP-54]

0060AEE5   .  BA C8B16000     MOV EDX,unpacked.0060B1C8

0060AEEA   .  E8 ED9FDFFF     CALL unpacked.00404EDC    //连接

0060AEEF   .  8B45 AC         MOV EAX,DWORD PTR SS:[EBP-54]  //连接成 NCDboy 入 EAX

0060AEF2   .  8D55 B0         LEA EDX,DWORD PTR SS:[EBP-50]

0060AEF5   .  E8 96EADFFF     CALL unpacked.00409990    //保存进堆栈

0060AEFA   .  8B55 B0         MOV EDX,DWORD PTR SS:[EBP-50]  //[EBP-50]=EDX 新地址

0060AEFD   .  58             POP EAX

0060AEFE   .  E8 D9A0DFFF     CALL unpacked.00404FDC    //关键 CALL2 跟进 这里比较的是注册名是 5 位的情况

0060AF03   .  0F85 17010000  JNZ unpacked.0060B020    //关键跳 跳就 over

0060AF09   >  837D FC 00      CMP DWORD PTR SS:[EBP-4],0

0060AF0D   .  74 0A          JE SHORT unpacked.0060AF19

0060AF0F   .  8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]

0060AF12   .  E8 4933E4FF    CALL unpacked.0044E260

0060AF17   .  EB 0F          JMP SHORT unpacked.0060AF28

0060AF19   >  B2 01          MOV DL,1

0060AF1B   .  A1 F0E04400     MOV EAX,DWORD PTR DS:[44E0F0]

0060AF20   .  E8 CB32E4FF     CALL unpacked.0044E1F0

0060AF25   .  8945 FC        MOV DWORD PTR SS:[EBP-4],EAX

0060AF28   >  33C0           XOR EAX,EAX

0060AF2A   .  55             PUSH EBP

0060AF2B   .  68 16B06000     PUSH unpacked.0060B016

0060AF30   .  64:FF30        PUSH DWORD PTR FS:[EAX]

0060AF33   .  64:8920        MOV DWORD PTR FS:[EAX],ESP

0060AF36   .  BA 02000080     MOV EDX,80000002

0060AF3B   .  8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]

0060AF3E   .  E8 4D33E4FF    CALL unpacked.0044E290

0060AF43   .  8D45 A4         LEA EAX,DWORD PTR SS:[EBP-5C]

0060AF46   .  8B4D F8         MOV ECX,DWORD PTR SS:[EBP-8]

0060AF49   .  BA D4B16000     MOV EDX,unpacked.0060B1D4            ;  ASCII "SoftWare\\Dbimp\\"

0060AF4E   .  E8 899FDFFF     CALL unpacked.00404EDC

0060AF53   .  8B55 A4         MOV EDX,DWORD PTR SS:[EBP-5C]

0060AF56   .  B1 01          MOV CL,1

0060AF58   .  8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]

0060AF5B   .  E8 9433E4FF     CALL unpacked.0044E2F4

0060AF60   .  84C0           TEST AL,AL

0060AF62   .  74 29          JE SHORT unpacked.0060AF8D

0060AF64   .  8D55 9C         LEA EDX,DWORD PTR SS:[EBP-64]

0060AF67   .  8B83 04030000   MOV EAX,DWORD PTR DS:[EBX+304]

0060AF6D   .  E8 CE29E8FF     CALL unpacked.0048D940

0060AF72   .  8B45 9C         MOV EAX,DWORD PTR SS:[EBP-64]

0060AF75   .  8D55 A0         LEA EDX,DWORD PTR SS:[EBP-60]

0060AF78   .  E8 13EADFFF     CALL unpacked.00409990

0060AF7D   .  8B4D A0         MOV ECX,DWORD PTR SS:[EBP-60]

0060AF80   .  BA 80B16000     MOV EDX,unpacked.0060B180            ;  ASCII "RegName"



0060AF85&nbsp;.&nbsp;8B45&nbsp;FC&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MOV&nbsp;EAX,DWORD&nbsp;PTR&nbsp;SS:\[EBP-4\]  

0060AF88&nbsp;.&nbsp;E8&nbsp;0335E4FF&nbsp;&nbsp;CALL&nbsp;unpacked.0044E490  

0060AF8D&nbsp;&gt;&nbsp;&nbsp;8D55&nbsp;94&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;LEA&nbsp;EDX,DWORD&nbsp;PTR&nbsp;SS:\[EBP-6C\]  

0060AF90&nbsp;.&nbsp;8B83&nbsp;08030000&nbsp;MOV&nbsp;EAX,DWORD&nbsp;PTR&nbsp;DS:\[EBX+308\]  

0060AF96&nbsp;.&nbsp;E8&nbsp;A529E8FF&nbsp;&nbsp;CALL&nbsp;unpacked.0048D940  

0060AF9B&nbsp;.&nbsp;8B45&nbsp;94&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MOV&nbsp;EAX,DWORD&nbsp;PTR&nbsp;SS:\[EBP-6C\]  

0060AF9E&nbsp;.&nbsp;8D55&nbsp;98&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;LEA&nbsp;EDX,DWORD&nbsp;PTR&nbsp;SS:\[EBP-68\]  

0060AFA1&nbsp;.&nbsp;E8&nbsp;EAE9DFFF&nbsp;&nbsp;CALL&nbsp;unpacked.00409990  

0060AFA6&nbsp;.&nbsp;8B4D&nbsp;98&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MOV&nbsp;ECX,DWORD&nbsp;PTR&nbsp;SS:\[EBP-68\]  

0060AFA9&nbsp;.&nbsp;BA&nbsp;90B16000&nbsp;&nbsp;MOV&nbsp;EDX,unpacked.0060B190&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.&nbsp;ASCII&nbsp;"RegID"  

0060AFAE&nbsp;.&nbsp;8B45&nbsp;FC&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MOV&nbsp;EAX,DWORD&nbsp;PTR&nbsp;SS:\[EBP-4\]  

0060AFB1&nbsp;.&nbsp;E8&nbsp;DA34E4FF&nbsp;&nbsp;CALL&nbsp;unpacked.0044E490  

0060AFB6&nbsp;.&nbsp;8B45&nbsp;FC&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MOV&nbsp;EAX,DWORD&nbsp;PTR&nbsp;SS:\[EBP-4\]  

0060AFB9&nbsp;.&nbsp;E8&nbsp;A232E4FF&nbsp;&nbsp;CALL&nbsp;unpacked.0044E260  

0060AFBE&nbsp;.&nbsp;6A&nbsp;00&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;PUSH&nbsp;0  

0060AFC0&nbsp;.&nbsp;B9&nbsp;98B16000&nbsp;&nbsp;MOV&nbsp;ECX,unpacked.0060B198  

0060AFC5&nbsp;.&nbsp;BA&nbsp;A4B16000&nbsp;&nbsp;MOV&nbsp;EDX,unpacked.0060B1A4  

0060AFCA&nbsp;.&nbsp;A1&nbsp;287C6A00&nbsp;&nbsp;MOV&nbsp;EAX,DWORD&nbsp;PTR&nbsp;DS:\[6A7C28\]  

0060AFCF&nbsp;.&nbsp;8B00&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MOV&nbsp;EAX,DWORD&nbsp;PTR&nbsp;DS:\[EAX\]  

0060AFD1&nbsp;.&nbsp;E8&nbsp;663CEAFF&nbsp;&nbsp;CALL&nbsp;unpacked.004AEC3C  

0060AFD6&nbsp;.&nbsp;8B83&nbsp;F8020000&nbsp;MOV&nbsp;EAX,DWORD&nbsp;PTR&nbsp;DS:\[EBX+2F8\]  

0060AFDC&nbsp;.&nbsp;33D2&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;XOR&nbsp;EDX,EDX  

0060AFDE&nbsp;.&nbsp;8B08&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MOV&nbsp;ECX,DWORD&nbsp;PTR&nbsp;DS:\[EAX\]  

0060AFE0&nbsp;.&nbsp;FF51&nbsp;64&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;CALL&nbsp;DWORD&nbsp;PTR&nbsp;DS:\[ECX+64\]  

0060AFE3&nbsp;.&nbsp;A1&nbsp;58776A00&nbsp;&nbsp;MOV&nbsp;EAX,DWORD&nbsp;PTR&nbsp;DS:\[6A7758\]  

0060AFE8&nbsp;.&nbsp;C640&nbsp;28&nbsp;01&nbsp;&nbsp;&nbsp;&nbsp;MOV&nbsp;BYTE&nbsp;PTR&nbsp;DS:\[EAX+28\],1  

0060AFEC&nbsp;.&nbsp;8B45&nbsp;FC&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MOV&nbsp;EAX,DWORD&nbsp;PTR&nbsp;SS:\[EBP-4\]  

0060AFEF&nbsp;.&nbsp;E8&nbsp;E88CDFFF&nbsp;&nbsp;CALL&nbsp;unpacked.00403CDC  

0060AFF4&nbsp;.&nbsp;33C0&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;XOR&nbsp;EAX,EAX  

0060AFF6&nbsp;.&nbsp;5A&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;POP&nbsp;EDX  

0060AFF7&nbsp;.&nbsp;59&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;POP&nbsp;ECX  

0060AFF8&nbsp;.&nbsp;59&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;POP&nbsp;ECX  

0060AFF9&nbsp;.&nbsp;64:8910&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MOV&nbsp;DWORD&nbsp;PTR&nbsp;FS:\[EAX\],EDX  

0060AFFC&nbsp;.&nbsp;33C0&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;XOR&nbsp;EAX,EAX  

0060AFFE&nbsp;.&nbsp;5A&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;POP&nbsp;EDX  

0060AFFF&nbsp;.&nbsp;59&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;POP&nbsp;ECX  

0060B000&nbsp;.&nbsp;59&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;POP&nbsp;ECX  

0060B001&nbsp;.&nbsp;64:8910&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MOV&nbsp;DWORD&nbsp;PTR&nbsp;FS:\[EAX\],EDX  

0060B004&nbsp;.&nbsp;33C0&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;XOR&nbsp;EAX,EAX  

0060B006&nbsp;.&nbsp;5A&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;POP&nbsp;EDX  

0060B007&nbsp;.&nbsp;59&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;POP&nbsp;ECX  

0060B008&nbsp;.&nbsp;59&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;POP&nbsp;ECX  

0060B009&nbsp;.&nbsp;64:8910&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MOV&nbsp;DWORD&nbsp;PTR&nbsp;FS:\[EAX\],EDX  

0060B00C&nbsp;.&nbsp;EB&nbsp;60&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;JMP&nbsp;SHORT&nbsp;unpacked.0060B06E  

0060B00E&nbsp;.&nbsp;33C0&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;XOR&nbsp;EAX,EAX  

0060B010&nbsp;.&nbsp;5A&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;POP&nbsp;EDX  

0060B011&nbsp;.&nbsp;59&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;POP&nbsp;ECX  

0060B012&nbsp;.&nbsp;59&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;POP&nbsp;ECX  

0060B013&nbsp;.&nbsp;64:8910&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MOV&nbsp;DWORD&nbsp;PTR&nbsp;FS:\[EAX\],EDX  

0060B016&nbsp;.&nbsp;.^.&nbsp;E9&nbsp;A191DFFF&nbsp;&nbsp;JMP&nbsp;unpacked.004041BC  

0060B01B&nbsp;.&nbsp;E8&nbsp;C895DFFF&nbsp;&nbsp;CALL&nbsp;unpacked.004045E8  

0060B020&nbsp;&gt;&nbsp;&nbsp;33C0&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;XOR&nbsp;EAX,EAX  

0060B022&nbsp;.&nbsp;5A&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;POP&nbsp;EDX  

0060B023&nbsp;.&nbsp;59&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;POP&nbsp;ECX  

0060B024&nbsp;.&nbsp;59&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;POP&nbsp;ECX  

0060B025&nbsp;.&nbsp;64:8910&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MOV&nbsp;DWORD&nbsp;PTR&nbsp;FS:\[EAX\],EDX  

0060B028&nbsp;.&nbsp;EB&nbsp;0A&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;JMP&nbsp;SHORT&nbsp;unpacked.0060B034  

0060B02A&nbsp;.&nbsp;.^.&nbsp;E9&nbsp;8D91DFFF&nbsp;&nbsp;JMP&nbsp;unpacked.004041BC  

0060B02F&nbsp;.&nbsp;E8&nbsp;B495DFFF&nbsp;&nbsp;CALL&nbsp;unpacked.004045E8  

0060B034&nbsp;&gt;&nbsp;&nbsp;33C0&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;XOR&nbsp;EAX,EAX  

0060B036&nbsp;.&nbsp;5A&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;POP&nbsp;EDX  

0060B037&nbsp;.&nbsp;59&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;POP&nbsp;ECX  

0060B038&nbsp;.&nbsp;59&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;POP&nbsp;ECX  

0060B039&nbsp;.&nbsp;64:8910&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MOV&nbsp;DWORD&nbsp;PTR&nbsp;FS:\[EAX\],EDX  

0060B03C&nbsp;.&nbsp;EB&nbsp;0A&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;JMP&nbsp;SHORT&nbsp;unpacked.0060B048  

0060B03E&nbsp;.&nbsp;.^.&nbsp;E9&nbsp;7991DFFF&nbsp;&nbsp;JMP&nbsp;unpacked.004041BC  

0060B043&nbsp;.&nbsp;E8&nbsp;A095DFFF&nbsp;&nbsp;CALL&nbsp;unpacked.004045E8  

0060B048&nbsp;&gt;&nbsp;&nbsp;837D&nbsp;FC&nbsp;00&nbsp;&nbsp;&nbsp;&nbsp;CMP&nbsp;DWORD&nbsp;PTR&nbsp;SS:\[EBP-4\],0  

0060B04C&nbsp;.&nbsp;74&nbsp;08&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;JE&nbsp;SHORT&nbsp;unpacked.0060B056  

0060B04E&nbsp;.&nbsp;8B45&nbsp;FC&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MOV&nbsp;EAX,DWORD&nbsp;PTR&nbsp;SS:\[EBP-4\]  

0060B051&nbsp;.&nbsp;E8&nbsp;868CDFFF&nbsp;&nbsp;CALL&nbsp;unpacked.00403CDC  

0060B056&nbsp;&gt;&nbsp;&nbsp;6A&nbsp;00&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;PUSH&nbsp;0  

0060B058&nbsp;.&nbsp;B9&nbsp;98B16000&nbsp;&nbsp;MOV&nbsp;ECX,unpacked.0060B198  

0060B05D&nbsp;.&nbsp;BA&nbsp;E4B16000&nbsp;&nbsp;MOV&nbsp;EDX,unpacked.0060B1E4  

0060B062&nbsp;.&nbsp;A1&nbsp;287C6A00&nbsp;&nbsp;MOV&nbsp;EAX,DWORD&nbsp;PTR&nbsp;DS:\[6A7C28\]  

0060B067&nbsp;.&nbsp;8B00&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MOV&nbsp;EAX,DWORD&nbsp;PTR&nbsp;DS:\[EAX\]  

0060B069&nbsp;.&nbsp;E8&nbsp;CE3BEAFF&nbsp;&nbsp;CALL&nbsp;unpacked.004AEC3C&nbsp;&nbsp;&nbsp;//这里弹出错误对话匡  

由以上的代码，可以得知注册名必须是&nbsp;4&nbsp;或&nbsp;5&nbsp;或&nbsp;6&nbsp;位。算法就是根据相应的字符和注册名计算出来的&nbsp;  

采用了两组&nbsp;3&nbsp;个相同的&nbsp;`CALL&nbsp;00672798&nbsp;00404FDC`&nbsp;作为关键&nbsp;`CALL`&nbsp;&nbsp;分别跟进&nbsp;`CALL1`&nbsp;  

00672798&nbsp;&nbsp;/&nbsp;&nbsp;55&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;PUSH&nbsp;EBP  

00672799&nbsp;&nbsp;|.&nbsp;8BEC&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MOV&nbsp;EBP,ESP  

0067279B&nbsp;&nbsp;|.&nbsp;83C4&nbsp;D0&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ADD&nbsp;ESP,-30  

0067279E&nbsp;&nbsp;|.&nbsp;53&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;PUSH&nbsp;EBX  

0067279F&nbsp;&nbsp;|.&nbsp;56&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;PUSH&nbsp;ESI  

006727A0&nbsp;&nbsp;|.&nbsp;57&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;PUSH&nbsp;EDI  

006727A1&nbsp;&nbsp;|.&nbsp;33DB&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;XOR&nbsp;EBX,EBX&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//EBX&nbsp;清零做准备  

006727A3&nbsp;&nbsp;|.&nbsp;895D&nbsp;D0&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MOV&nbsp;DWORD&nbsp;PTR&nbsp;SS:\[EBP-30\],EBX&nbsp;&nbsp;//全清零  

006727A6&nbsp;&nbsp;|.&nbsp;895D&nbsp;D8&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MOV&nbsp;DWORD&nbsp;PTR&nbsp;SS:\[EBP-28\],EBX  

006727A9&nbsp;&nbsp;|.&nbsp;895D&nbsp;D4&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MOV&nbsp;DWORD&nbsp;PTR&nbsp;SS:\[EBP-2C\],EBX  

006727AC&nbsp;&nbsp;|.&nbsp;895D&nbsp;E0&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MOV&nbsp;DWORD&nbsp;PTR&nbsp;SS:\[EBP-20\],EBX  

006727AF&nbsp;&nbsp;|.&nbsp;895D&nbsp;DC&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MOV&nbsp;DWORD&nbsp;PTR&nbsp;SS:\[EBP-24\],EBX  

006727B2&nbsp;&nbsp;|.&nbsp;895D&nbsp;EC&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MOV&nbsp;DWORD&nbsp;PTR&nbsp;SS:\[EBP-14\],EBX  

006727B5&nbsp;&nbsp;|.&nbsp;894D&nbsp;F4&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MOV&nbsp;DWORD&nbsp;PTR&nbsp;SS:\[EBP-C\],ECX  

006727B8&nbsp;&nbsp;|.&nbsp;8955&nbsp;F8&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MOV&nbsp;DWORD&nbsp;PTR&nbsp;SS:\[EBP-8\],EDX&nbsp;&nbsp;//相关字符保存第&nbsp;1&nbsp;次是&nbsp;HDDBIP&nbsp;第&nbsp;2&nbsp;次是&nbsp;第&nbsp;3&nbsp;次是库名&nbsp;Pams  

006727BB&nbsp;&nbsp;|.&nbsp;8945&nbsp;FC&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MOV&nbsp;DWORD&nbsp;PTR&nbsp;SS:\[EBP-4\],EAX&nbsp;&nbsp;//注册码保存  

006727BE&nbsp;&nbsp;|.&nbsp;8B45&nbsp;FC&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MOV&nbsp;EAX,DWORD&nbsp;PTR&nbsp;SS:\[EBP-4\]&nbsp;&nbsp;//\[EBP-4\]=EAX=注册码  

006727C1&nbsp;&nbsp;|.&nbsp;E8&nbsp;BA28D9FF&nbsp;&nbsp;CALL&nbsp;unpacked.00405080  

006727C6&nbsp;&nbsp;|.&nbsp;8B45&nbsp;F8&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MOV&nbsp;EAX,DWORD&nbsp;PTR&nbsp;SS:\[EBP-8\]&nbsp;&nbsp;//\[EBP-8\]=EAX=相关字符  

006727C9&nbsp;&nbsp;|.&nbsp;E8&nbsp;B228D9FF&nbsp;&nbsp;CALL&nbsp;unpacked.00405080  

006727CE&nbsp;&nbsp;|.&nbsp;33C0&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;XOR&nbsp;EAX,EAX&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//EAX&nbsp;清零  

006727D0&nbsp;&nbsp;|.&nbsp;55&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;PUSH&nbsp;EBP  

006727D1&nbsp;&nbsp;|.&nbsp;68&nbsp;F3286700&nbsp;&nbsp;PUSH&nbsp;unpacked.006728F3  

006727D6&nbsp;&nbsp;|.&nbsp;64:FF30&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;PUSH&nbsp;DWORD&nbsp;PTR&nbsp;FS:\[EAX\]  

006727D9&nbsp;&nbsp;|.&nbsp;64:8920&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MOV&nbsp;DWORD&nbsp;PTR&nbsp;FS:\[EAX\],ESP  

006727DC&nbsp;&nbsp;|.&nbsp;8B45&nbsp;F8&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MOV&nbsp;EAX,DWORD&nbsp;PTR&nbsp;SS:\[EBP-8\]  

006727DF&nbsp;&nbsp;|.&nbsp;E8&nbsp;AC26D9FF&nbsp;&nbsp;CALL&nbsp;unpacked.00404E90&nbsp;&nbsp;&nbsp;//取相关字符位数&nbsp;6  

006727E4&nbsp;&nbsp;|.&nbsp;8945&nbsp;F0&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MOV&nbsp;DWORD&nbsp;PTR&nbsp;SS:\[EBP-10\],EAX&nbsp;&nbsp;//位数进\[EBP-10\]  

006727E7&nbsp;&nbsp;|.&nbsp;837D&nbsp;F0&nbsp;00&nbsp;&nbsp;&nbsp;&nbsp;CMP&nbsp;DWORD&nbsp;PTR&nbsp;SS:\[EBP-10\],0&nbsp;&nbsp;&nbsp;//比较是否是零  

006727EB&nbsp;&nbsp;|.&nbsp;75&nbsp;0D&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;JNZ&nbsp;SHORT&nbsp;unpacked.006727FA&nbsp;&nbsp;&nbsp;//不是则继续  

006727ED&nbsp;&nbsp;|.&nbsp;8D45&nbsp;F8&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;LEA&nbsp;EAX,DWORD&nbsp;PTR&nbsp;SS:\[EBP-8\]  

006727F0&nbsp;&nbsp;|.&nbsp;BA&nbsp;0C296700&nbsp;&nbsp;MOV&nbsp;EDX,unpacked.0067290C&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.&nbsp;ASCII&nbsp;"Think&nbsp;Space"  

006727F5&nbsp;&nbsp;|.&nbsp;E8&nbsp;6E24D9FF&nbsp;&nbsp;CALL&nbsp;unpacked.00404C68  

006727FA&nbsp;&nbsp;|&gt;&nbsp;&nbsp;33F6&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;XOR&nbsp;ESI,ESI  

006727FC&nbsp;&nbsp;|.&nbsp;8D45&nbsp;DC&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;LEA&nbsp;EAX,DWORD&nbsp;PTR&nbsp;SS:\[EBP-24\]  

006727FF&nbsp;&nbsp;|.&nbsp;50&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;PUSH&nbsp;EAX  

00672800&nbsp;&nbsp;|.&nbsp;B9&nbsp;02000000&nbsp;&nbsp;MOV&nbsp;ECX,2&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//ECX=2  

00672805&nbsp;&nbsp;|.&nbsp;BA&nbsp;01000000&nbsp;&nbsp;MOV&nbsp;EDX,1&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//EDX=1  

0067280A&nbsp;&nbsp;|.&nbsp;8B45&nbsp;FC&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MOV&nbsp;EAX,DWORD&nbsp;PTR&nbsp;SS:\[EBP-4\]&nbsp;&nbsp;//\[EBP-4\]=EAX=假码  

0067280D&nbsp;&nbsp;|.&nbsp;E8&nbsp;DE28D9FF&nbsp;&nbsp;CALL&nbsp;unpacked.004050F0  

00672812&nbsp;&nbsp;|.&nbsp;8B4D&nbsp;DC&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MOV&nbsp;ECX,DWORD&nbsp;PTR&nbsp;SS:\[EBP-24\]&nbsp;&nbsp;//假码前&nbsp;2&nbsp;位进&nbsp;ECX=12  

00672815&nbsp;&nbsp;|.&nbsp;8D45&nbsp;E0&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;LEA&nbsp;EAX,DWORD&nbsp;PTR&nbsp;SS:\[EBP-20\]  

00672818&nbsp;&nbsp;|.&nbsp;BA&nbsp;20296700&nbsp;&nbsp;MOV&nbsp;EDX,unpacked.00672920  

0067281D&nbsp;&nbsp;|.&nbsp;E8&nbsp;BA26D9FF&nbsp;&nbsp;CALL&nbsp;unpacked.00404EDC  

00672822&nbsp;&nbsp;|.&nbsp;8B45&nbsp;E0&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MOV&nbsp;EAX,DWORD&nbsp;PTR&nbsp;SS:\[EBP-20\]&nbsp;&nbsp;//前&nbsp;2&nbsp;位地址入&nbsp;EAX  

00672825&nbsp;&nbsp;|.&nbsp;E8&nbsp;5A76D9FF&nbsp;&nbsp;CALL&nbsp;unpacked.00409E84&nbsp;&nbsp;&nbsp;//转换成数字  

0067282A&nbsp;&nbsp;|.&nbsp;8BF8&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MOV&nbsp;EDI,EAX&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//EAX=EDI=00000012  

0067282C&nbsp;&nbsp;|.&nbsp;C745&nbsp;E8&nbsp;03000&gt;MOV&nbsp;DWORD&nbsp;PTR&nbsp;SS:\[EBP-18\],3  

00672833&nbsp;&nbsp;|&gt;&nbsp;&nbsp;8D45&nbsp;D4&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;/LEA&nbsp;EAX,DWORD&nbsp;PTR&nbsp;SS:\[EBP-2C\]  

00672836&nbsp;&nbsp;|.&nbsp;50&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|PUSH&nbsp;EAX  

00672837&nbsp;&nbsp;|.&nbsp;B9&nbsp;02000000&nbsp;&nbsp;|MOV&nbsp;ECX,2&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//ECX=2  

0067283C&nbsp;&nbsp;|.&nbsp;8B55&nbsp;E8&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|MOV&nbsp;EDX,DWORD&nbsp;PTR&nbsp;SS:\[EBP-18\]  

0067283F&nbsp;&nbsp;|.&nbsp;8B45&nbsp;FC&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|MOV&nbsp;EAX,DWORD&nbsp;PTR&nbsp;SS:\[EBP-4\]&nbsp;&nbsp;//假码进&nbsp;EAX  

00672842&nbsp;&nbsp;|.&nbsp;E8&nbsp;A928D9FF&nbsp;&nbsp;|CALL&nbsp;unpacked.004050F0&nbsp;&nbsp;&nbsp;//取假码每两位  

00672847&nbsp;&nbsp;|.&nbsp;8B4D&nbsp;D4&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|MOV&nbsp;ECX,DWORD&nbsp;PTR&nbsp;SS:\[EBP-2C\]&nbsp;&nbsp;//每两位进&nbsp;ECX&nbsp;&nbsp;  

0067284A&nbsp;&nbsp;|.&nbsp;8D45&nbsp;D8&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|LEA&nbsp;EAX,DWORD&nbsp;PTR&nbsp;SS:\[EBP-28\]  

0067284D&nbsp;&nbsp;|.&nbsp;BA&nbsp;20296700&nbsp;&nbsp;|MOV&nbsp;EDX,unpacked.00672920  

00672852&nbsp;&nbsp;|.&nbsp;E8&nbsp;8526D9FF&nbsp;&nbsp;|CALL&nbsp;unpacked.00404EDC  

00672857&nbsp;&nbsp;|.&nbsp;8B45&nbsp;D8&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;|MOV&nbsp;EAX,DWORD&nbsp;PTR&nbsp;SS:\[EBP-28\]&nbsp;&nbsp;//\[EBP-28\]=EAX&nbsp;每两位地址入&nbsp;EAX  

0067285A&nbsp;&nbsp;|.&nbsp;E8&nbsp;2576D9FF&nbsp;&nbsp;|CALL&nbsp;unpacked.00409E84&nbsp;&nbsp;&nbsp;//转换成数字  



# 逆向分析记录

## 第一段代码分析

```assembly
0067285F  |.  8945 E4       |MOV DWORD PTR SS:[EBP-1C],EAX  //EAX=[EBP-1C]=00000034  
00672862  |.  3B75 F0       |CMP ESI,DWORD PTR SS:[EBP-10]  //ESI=0 和相关字符长度比较  
00672865  |.  7D 03         |JGE SHORT unpacked.0067286A  //大于等于跳  
00672867  |.  46            |INC ESI        //ESI+1  
00672868  |.  EB 05         |JMP SHORT unpacked.0067286F  //跳  
0067286A  |>  BE 01000000   |MOV ESI,1  
0067286F  |>  8B45 F8       |MOV EAX,DWORD PTR SS:[EBP-8]  //相关字符进 EAX  
00672872  |.  33DB          |XOR EBX,EBX      //EBX 清零  
00672874  |.  8A5C30 FF     |MOV BL,BYTE PTR DS:[EAX+ESI-1]  //相关字符的每一位 ASC 依次进 BL48  
00672878  |.  335D E4       |XOR EBX,DWORD PTR SS:[EBP-1C]  //和 34 56 异或 7C 12  
0067287B  |.  3BFB          |CMP EDI,EBX      //和 12 比较  
0067287D  |.  7C 0A         |JL SHORT unpacked.00672889    //小于跳  
0067287F  |.  81C3 FF000000 |ADD EBX,0FF      //EBX+0FF=111  
00672885  |.  2BDF          |SUB EBX,EDI      //EBX-EDI=DD  
00672887  |.  EB 02         |JMP SHORT unpacked.0067288B  
00672889  |>  2BDF          |SUB EBX,EDI      //EBX-EDI=6A  
0067288B  |>  8D45 D0       |LEA EAX,DWORD PTR SS:[EBP-30]  
0067288E  |.  8BD3          |MOV EDX,EBX      //差再移入 EDX  
00672890  |.  E8 2325D9FF   |CALL unpacked.00404DB8    //转换成字符 j  
00672895  |.  8B55 D0       |MOV EDX,DWORD PTR SS:[EBP-30]  //地址进 EDX CED738  
00672898  |.  8D45 EC       |LEA EAX,DWORD PTR SS:[EBP-14]  
0067289B  |.  E8 F825D9FF   |CALL unpacked.00404E98  
006728A0  |.  8B7D E4       |MOV EDI,DWORD PTR SS:[EBP-1C]  //[EBP-1C]里的数入 EDI  
006728A3  |.  8345 E8 02    |ADD DWORD PTR SS:[EBP-18],2  //3+2=5  
006728A7  |.  8B45 FC       |MOV EAX,DWORD PTR SS:[EBP-4]  //假码入 EAX  
006728AA  |.  E8 E125D9FF   |CALL unpacked.00404E90    //取假码位数  
006728AF  |.  3B45 E8       |CMP EAX,DWORD PTR SS:[EBP-18]  //和[EBP-18]中的数比较  
006728B2  |.^ 0F8F 7BFFFFFF \JG unpacked.00672833    //没有完就继续  
006728B8  |.  8B45 F4       MOV EAX,DWORD PTR SS:[EBP-C]  
006728BB  |.  8B55 EC       MOV EDX,DWORD PTR SS:[EBP-14]  //计算结果 6ADD 的地址 CED738 进 EDX  
006728BE  |.  E8 6123D9FF   CALL unpacked.00404C24  
006728C3  |.  33C0          XOR EAX,EAX  
006728C5  |.  5A            POP EDX  
006728C6  |.  59            POP ECX  
006728C7  |.  59            POP ECX  
006728C8  |.  64:8910      MOV DWORD PTR FS:[EAX],EDX  
006728CB  |.  68 FA286700  PUSH unpacked.006728FA  
006728D0  |>  8D45 D0       LEA EAX,DWORD PTR SS:[EBP-30]  
006728D3  |.  BA 05000000   MOV EDX,5  
006728D8  |.  E8 1723D9FF   CALL unpacked.00404BF4  
006728DD  |.  8D45 EC       LEA EAX,DWORD PTR SS:[EBP-14]  
006728E0  |.  E8 EB22D9FF   CALL unpacked.00404BD0  
006728E5  |.  8D45 F8       LEA EAX,DWORD PTR SS:[EBP-8]  
006728E8  |.  BA 02000000   MOV EDX,2  
006728ED  |.  E8 0223D9FF   CALL unpacked.00404BF4  
006728F2  \\.  C3           RETN  
006728F3   .^ E9 781BD9FF   JMP unpacked.00404470  
006728F8   .^ EB D6         JMP SHORT unpacked.006728D0  
006728FA   .  5F            POP EDI  
006728FB   .  5E            POP ESI  
006728FC   .  5B            POP EBX  
006728FD   .  8BE5         MOV ESP,EBP  
006728FF   .  5D            POP EBP  
00672900   .  C3            RETN  
```

以上这段的循环就是依次取相关字符的每一位字符的 `ASC` 码与假码的第 `3` 位开始的每两位异或。

结果和假码的前 `2` 位比较，大于则减注册码前 `2` 位，小于则加 `0FF` 再减前 `2` 位，最后结果是 `ASC` 码。

## 第二段代码分析（CALL2）

再跟进 `CALL2`。

```assembly
00404FDC  /$  53            PUSH EBX  
00404FDD  |.  56            PUSH ESI  
00404FDE  |.  57            PUSH EDI  
00404FDF  |.  89C6         MOV ESI,EAX        //CALL1 计算的结果地址进 ESI  
00404FE1  |.  89D7         MOV EDI,EDX        //注册名进 EDI  
00404FE3  |.  39D0         CMP EAX,EDX        //相等就挂  
00404FE5  |.  0F84 8F000000 JE unpacked.0040507A  
00404FEB  |.  85F6         TEST ESI,ESI  
00404FED  |.  74 68        JE SHORT unpacked.00405057  
00404FEF  |.  85FF         TEST EDI,EDI  
00404FF1  |.  74 6B        JE SHORT unpacked.0040505E  
00404FF3  |.  8B46 FC     MOV EAX,DWORD PTR DS:[ESI-4]  //CALL1 计算的结果的长度 2 进 EAX  
00404FF6  |.  8B57 FC     MOV EDX,DWORD PTR DS:[EDI-4]  //注册名位数 5 移入 EDX  
00404FF9  |.  29D0         SUB EAX,EDX        //EAX=EAX-EDX  
00404FFB  |.  77 02        JA SHORT unpacked.00404FFF    //不小于跳  
00404FFD  |.  01C2         ADD EDX,EAX        //EDX+EAX=2  
00404FFF  |>  52           PUSH EDX  
00405000  |.  C1EA 02     SHR EDX,2        //右移 2 位  
00405003  |.  74 26        JE SHORT unpacked.0040502B    //0 则跳  
00405005  |>  8B0E         /MOV ECX,DWORD PTR DS:[ESI]    //地址[ESI]入 ECX  
00405007  |.  8B1F          |MOV EBX,DWORD PTR DS:[EDI]    //地址[EDI]入 EBX  
00405009  |.  39D9          |CMP ECX,EBX  
0040500B  |.  75 58         |JNZ SHORT unpacked.00405065  
0040500D  |.  4A            |DEC EDX        //EDX-1  
0040500E  |.  74 15         |JE SHORT unpacked.00405025  
00405010  |.  8B4E 04       |MOV ECX,DWORD PTR DS:[ESI+4]  
00405013  |.  8B5F 04       |MOV EBX,DWORD PTR DS:[EDI+4]  
00405016  |.  39D9          |CMP ECX,EBX  
00405018  |.  75 4B         |JNZ SHORT unpacked.00405065  
0040501A  |.  83C6 08       |ADD ESI,8  
0040501D  |.  83C7 08       |ADD EDI,8  
00405020  |.  4A            |DEC EDX  
00405021  |.^ 75 E2         \JNZ SHORT unpacked.00405005  
00405023  |.  EB 06         JMP SHORT unpacked.0040502B  
00405025  |>  83C6 04       ADD ESI,4        //取 ESI 后两位  
00405028  |.  83C7 04       ADD EDI,4        //取 EDI 后两位  
0040502B  |>  5A            POP EDX        //EDX 出栈  
0040502C  |.  83E2 03       AND EDX,3        //EDXand3  
0040502F  |.  74 22         JE SHORT unpacked.00405053  
00405031  |.  8B0E          MOV ECX,DWORD PTR DS:[ESI]    //CALL1 计算的结果地址入 ECX  
00405033  |.  8B1F          MOV EBX,DWORD PTR DS:[EDI]    //注册名地址移入 EBX  
00405035  |.  38D9          CMP CL,BL        //比较 CL BL  
00405037  |.  75 41         JNZ SHORT unpacked.0040507A    //不等跳 跳则挂  
00405039  |.  4A            DEC EDX        //EDX 减 1  
0040503A  |.  74 17         JE SHORT unpacked.00405053  
0040503C  |.  38FD          CMP CH,BH        //比较 CH BH  
0040503E  |.  75 3A         JNZ SHORT unpacked.0040507A    //不等则挂  
00405040  |.  4A            DEC EDX        //EDX-1  
00405041  |.  74 10         JE SHORT unpacked.00405053    //等于 0 就跳  
00405043  |.  81E3 0000FF00 AND EBX,0FF0000     //EBX=EBX+0FF0000     
00405049  |.  81E1 0000FF00 AND ECX,0FF0000     //ECX=ECX+0FF0000  
0040504F  |.  39D9          CMP ECX,EBX        //比较  
00405051  |.  75 27         JNZ SHORT unpacked.0040507A    //不等跳  
00405053  |>  01C0          ADD EAX,EAX        //这里是标志位是 0 则成功  
00405055  |.  EB 23         JMP SHORT unpacked.0040507A    //跳  
00405057  |>  8B57 FC     MOV EDX,DWORD PTR DS:[EDI-4]  
0040505A  |.  29D0          SUB EAX,EDX  
0040505C  |.  EB 1C         JMP SHORT unpacked.0040507A  
0040505E  |>  8B46 FC     MOV EAX,DWORD PTR DS:[ESI-4]  
00405061  |.  29D0          SUB EAX,EDX  
00405063  |.  EB 15         JMP SHORT unpacked.0040507A  
00405065  |>  5A            POP EDX  
00405066  |.  38D9          CMP CL,BL  
00405068  |.  75 10         JNZ SHORT unpacked.0040507A  
0040506A  |.  38FD          CMP CH,BH  
0040506C  |.  75 0C         JNZ SHORT unpacked.0040507A  
0040506E  |.  C1E9 10       SHR ECX,10  
00405071  |.  C1EB 10       SHR EBX,10  
00405074  |.  38D9          CMP CL,BL  
00405076  |.  75 02         JNZ SHORT unpacked.0040507A  
00405078  |.  38FD          CMP CH,BH  
0040507A  |>  5F            POP EDI  
0040507B  |.  5E            POP ESI  
0040507C  |.  5B            POP EBX  
0040507D  \\.  C3           RETN  
```

以上这段代码就是说依次比较每一位在 `CALL1` 中的计算结果和注册名，相等就成功。

因为我的注册名是 `5` 位 `CDboy`，也就在 `0060AEFE . E8 D9A0DFFF CALL unpacked.00404FDC` 做关键的比较。

## 算法总结

1. 依注册名的位数分别计算。
2. `4` 位注册名则取字符 `Pams`（数据库名）的每位字符的 `ASC` 码分别与注册码的第 `3` 位开始的每两位异或。若结果大于注册码的前 `2` 位则减去前 `2` 位的值，若小于前 `2` 位则加上 `OFF` 后再减去前两位的值，最后等于注册名。
3. `5` 位注册名则取字符 `Pams`（数据库名）每位字符的 `ASC` 码分别与注册码的第 `3` 位开始的每两位异或。若结果大于注册码的前 `2` 位则减去前 `2` 位的值，若小于前 `2` 位则加上 `OFF` 后再减去前两位的值，最后等于注册名前加上 `N`。



### 算法步骤

4. 6 位注册名则取字符 `HDDBIP`（固定字串）的每位字符的 `ASC` 码分别与注册码的第 `3` 位开始的每两位异或。

若结果大于注册码的前 2 位则减去前 2 位的值，若小于前 2 位则加上 `OFF` 后再减去前两位的值，最后等于注册名。

按照算法：

我的注册名是 `CDboy`，加上固定字符 `N` 就是 `NCDboy` 与 `Pams` 计算。注册码前 2 位可假定为 `01`：

```
4E(N)+01=4F xor 50(P)=1F
43(C)+1F=62 xor 61(a)=03
44(D)+03=47 xor 6D(m)=2A
62(b)+2A=8C xor 73(s)=FF
6F(o)+FF-FF=6F xor 50(P)=3F
79(y)+3F=B8 xor 61(a)=D9
```

所以 `Code=011F032AFF3FD9`。

可以看出这个算法同一注册名有多个注册码，根据前 2 位注册码生成。

### 注册表操作

注册成功后写入注册表：

```
[HKEY_LOCAL_MACHINE\SOFTWARE\Dbimp\Pams]

"RegName"="CDboy"
"RegID"="011F032AFF3FD9"
```

删掉又可以再玩一次 ^_^。

> 菜鸟写破文，感谢大家看完。

---

【工程声明】本过程只供内部学习之用！如要转载请保持过程完整！

---

`4dd466d0e1e0cc123afbe504da2462ca`
