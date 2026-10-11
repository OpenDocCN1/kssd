
# Advanced eLearning Builder 3.6.7 算法分析

> 原文：[`www.pediy.com/kssd/pediy10/73256.html`](https://www.pediy.com/kssd/pediy10/73256.html)

【破文标题】Advanced eLearning Builder 3.6.7 算法分析
【破文作者】`tianxj`
【作者邮箱】`tianxj_2007@126.com`
【作者主页】`WwW.ChiNaPYG.CoM`
【破解工具】`PEiD`, `OD`
【破解平台】D-Windows XP sp2
【软件名称】`Advanced eLearning Builder 3.6.7`
【软件大小】2209KB
【软件语言】英文
【软件类别】国外软件/考试系统
【更新时间】2008-9-9
【原版下载】自己找一下
【保护方式】注册码

【软件简介】`Advanced eLearning Builder` 是一款创建电子考题，指南，小型测验等电子教学材料的工具。使用该系统可以简单的通过点击几下鼠标创建各种类型的电子考题（可视化设计）等。

【破解声明】我是一只小菜鸟，偶得一点心得，愿与大家分享：）

--------------------------------------------------------------

【破解内容】

--------------------------------------------------------------

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

## 一、运行程序，进行注册，输入错误的注册信息进行检测，有提示信息

`"Invalid Registration Key!"`

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

## 二、用 `PEiD` 对 `advelb.exe` 查壳，为 `Borland Delphi 6.0 - 7.0`

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

## 三、运行 `OD`，打开 `advelb.exe`，输入注册信息，`F12` 暂停，`alt+K`

调用堆栈 , 项目 19

`地址=0012EAA4`
`堆栈=0049BB40`
`程序过程 / 参数=? advelb.0047F990`
`调用来自=advelb.0049BB3B`
`结构=0012EAA0`

==============================================================

### 代码:
```



```
0049BA18  /.  55                 PUSH EBP
0049BA19  |.  8BEC               MOV EBP,ESP
0049BA1B  |.  B9 35000000        MOV ECX,35
0049BA20  |>  6A 00              /PUSH 0
0049BA22  |.  6A 00              |PUSH 0
0049BA24  |.  49                 |DEC ECX
0049BA25  |.^ 75 F9              \JNZ SHORT advelb.0049BA20
0049BA27  |.  51                 PUSH ECX
0049BA28  |.  8955 F8            MOV DWORD PTR SS:[EBP-8],EDX
0049BA2B  |.  8945 FC            MOV DWORD PTR SS:[EBP-4],EAX
0049BA2E  |.  33C0               XOR EAX,EAX
0049BA30  |.  55                 PUSH EBP
0049BA31  |.  68 06BD4900        PUSH advelb.0049BD06
0049BA36  |.  64:FF30            PUSH DWORD PTR FS:[EAX]
0049BA39  |.  64:8920            MOV DWORD PTR FS:[EAX],ESP
0049BA3C  |.  A1 546C4B00        MOV EAX,DWORD PTR DS:[4B6C54]
0049BA41  |.  C600 00            MOV BYTE PTR DS:[EAX],0
0049BA44  |.  8D95 80FEFFFF      LEA EDX,DWORD PTR SS:[EBP-180]
0049BA4A  |.  8B45 FC            MOV EAX,DWORD PTR SS:[EBP-4]
0049BA4D  |.  8B80 14030000      MOV EAX,DWORD PTR DS:[EAX+314]
0049BA53  |.  E8 082FFCFF        CALL advelb.0045E960
0049BA58  |.  8B95 80FEFFFF      MOV EDX,DWORD PTR SS:[EBP-180]          ;  //用户名
0049BA5E  |.  8D85 84FEFFFF      LEA EAX,DWORD PTR SS:[EBP-17C]
0049BA64  |.  B9 FF000000        MOV ECX,0FF
0049BA69  |.  E8 FA8EF6FF        CALL advelb.00404968
0049BA6E  |.  8D95 84FEFFFF      LEA EDX,DWORD PTR SS:[EBP-17C]
0049BA74  |.  8D45 88            LEA EAX,DWORD PTR SS:[EBP-78]
0049BA77  |.  B1 64              MOV CL,64
0049BA79  |.  E8 C275F6FF        CALL advelb.00403040
0049BA7E  |.  8D95 7CFEFFFF      LEA EDX,DWORD PTR SS:[EBP-184]
0049BA84  |.  8B45 FC            MOV EAX,DWORD PTR SS:[EBP-4]
0049BA87  |.  8B80 0C030000      MOV EAX,DWORD PTR DS:[EAX+30C]
0049BA8D  |.  E8 CE2EFCFF        CALL advelb.0045E960
0049BA92  |.  8B95 7CFEFFFF      MOV EDX,DWORD PTR SS:[EBP-184]          ;  //试练码
0049BA98  |.  8D85 84FEFFFF      LEA EAX,DWORD PTR SS:[EBP-17C]
0049BA9E  |.  B9 FF000000        MOV ECX,0FF
0049BAA3  |.  E8 C08EF6FF        CALL advelb.00404968
0049BAA8  |.  8D95 84FEFFFF      LEA EDX,DWORD PTR SS:[EBP-17C]
0049BAAE  |.  8D45 ED            LEA EAX,DWORD PTR SS:[EBP-13]
0049BAB1  |.  B1 0A              MOV CL,0A
0049BAB3  |.  E8 8875F6FF        CALL advelb.00403040
0049BAB8  |.  8D85 78FEFFFF      LEA EAX,DWORD PTR SS:[EBP-188]
0049BABE  |.  8D55 88            LEA EDX,DWORD PTR SS:[EBP-78]
0049BAC1  |.  E8 6A8EF6FF        CALL advelb.00404930
0049BAC6  |.  8B85 78FEFFFF      MOV EAX,DWORD PTR SS:[EBP-188]          ;  //用户名
0049BACC  |.  50                 PUSH EAX
0049BACD  |.  8D85 74FEFFFF      LEA EAX,DWORD PTR SS:[EBP-18C]
0049BAD3  |.  8D55 ED            LEA EDX,DWORD PTR SS:[EBP-13]
0049BAD6  |.  E8 558EF6FF        CALL advelb.00404930
0049BADB  |.  8B85 74FEFFFF      MOV EAX,DWORD PTR SS:[EBP-18C]          ;  //试练码
0049BAE1  |.  B9 7D000000        MOV ECX,7D                              ;  //ECX=7Dh
0049BAE6  |.  5A                 POP EDX
0049BAE7  |.  E8 9CF5FEFF        CALL advelb.0048B088                    ;  //关键 CALL1，第 1 种算法
0049BAEC  |.  84C0               TEST AL,AL
0049BAEE  |.  75 55              JNZ SHORT advelb.0049BB45               ;  //关键跳转
0049BAF0  |.  8D85 70FEFFFF      LEA EAX,DWORD PTR SS:[EBP-190]
0049BAF6  |.  8D55 88            LEA EDX,DWORD PTR SS:[EBP-78]
0049BAF9  |.  E8 328EF6FF        CALL advelb.00404930
0049BAFE  |.  8B85 70FEFFFF      MOV EAX,DWORD PTR SS:[EBP-190]          ;  //用户名
0049BB04  |.  50                 PUSH EAX
0049BB05  |.  8D85 6CFEFFFF      LEA EAX,DWORD PTR SS:[EBP-194]
0049BB0B  |.  8D55 ED            LEA EDX,DWORD PTR SS:[EBP-13]
0049BB0E  |.  E8 1D8EF6FF        CALL advelb.00404930
0049BB13  |.  8B85 6CFEFFFF      MOV EAX,DWORD PTR SS:[EBP-194]          ;  //试练码
0049BB19  |.  B9 87000000        MOV ECX,87                              ;  //ECX=87h
0049BB1E  |.  5A                 POP EDX
0049BB1F  |.  E8 64F5FEFF        CALL advelb.0048B088                    ;  //关键 CALL2，第 2 种算法
0049BB24  |.  84C0               TEST AL,AL
0049BB26  |.  75 1D              JNZ SHORT advelb.0049BB45               ;  //关键跳转
0049BB28  |.  6A 00              PUSH 0
0049BB2A  |.  B9 14BD4900        MOV ECX,advelb.0049BD14                 ;  ASCII "Advanced eLearning Builder"
0049BB2F  |.  BA 30BD4900        MOV EDX,advelb.0049BD30                 ;  ASCII "Invalid Registration Key!"
0049BB34  |.  A1 106F4B00        MOV EAX,DWORD PTR DS:[4B6F10]
0049BB39  |.  8B00               MOV EAX,DWORD PTR DS:[EAX]
0049BB3B  |.  E8 503EFEFF        CALL advelb.0047F990
0049BB40  |.  E9 83010000        JMP advelb.0049BCC8
0049BB45  |>  A1 546C4B00        MOV EAX,DWORD PTR DS:[4B6C54]
0049BB4A  |.  C600 01            MOV BYTE PTR DS:[EAX],1
0049BB4D  |.  A1 AC6C4B00        MOV EAX,DWORD PTR DS:[4B6CAC]
0049BB52  |.  8B00               MOV EAX,DWORD PTR DS:[EAX]
0049BB54  |.  8B80 04030000      MOV EAX,DWORD PTR DS:[EAX+304]
0049BB5A  |.  8078 39 00         CMP BYTE PTR DS:[EAX+39],0
0049BB5E  |.  74 0E              JE SHORT advelb.0049BB6E
0049BB60  |.  A1 AC6C4B00        MOV EAX,DWORD PTR DS:[4B6CAC]
0049BB65  |.  8B00               MOV EAX,DWORD PTR DS:[EAX]
0049BB67  |.  E8 90530000        CALL advelb.004A0EFC
0049BB6C  |.  EB 0C              JMP SHORT advelb.0049BB7A
0049BB6E  |>  A1 AC6C4B00        MOV EAX,DWORD PTR DS:[4B6CAC]
0049BB73  |.  8B00               MOV EAX,DWORD PTR DS:[EAX]
0049BB75  |.  E8 96510000        CALL advelb.004A0D10
0049BB7A  |>  8D95 68FEFFFF      LEA EDX,DWORD PTR SS:[EBP-198]
0049BB80  |.  8B45 FC            MOV EAX,DWORD PTR SS:[EBP-4]
0049BB83  |.  8B80 14030000      MOV EAX,DWORD PTR DS:[EAX+314]
0049BB89  |.  E8 D22DFCFF        CALL advelb.0045E960
0049BB8E  |.  8B85 68FEFFFF      MOV EAX,DWORD PTR SS:[EBP-198]
0049BB94  |.  E8 F38DF6FF        CALL advelb.0040498C
0049BB99  |.  85C0               TEST EAX,EAX
0049BB9B  |.  74 5E              JE SHORT advelb.0049BBFB
0049BB9D  |.  8D95 64FEFFFF      LEA EDX,DWORD PTR SS:[EBP-19C]
0049BBA3  |.  8B45 FC            MOV EAX,DWORD PTR SS:[EBP-4]
0049BBA6  |.  8B80 14030000      MOV EAX,DWORD PTR DS:[EAX+314]
0049BBAC  |.  E8 AF2DFCFF        CALL advelb.0045E960
0049BBB1  |.  8B85 64FEFFFF      MOV EAX,DWORD PTR SS:[EBP-19C]
0049BBB7  |.  50                 PUSH EAX
0049BBB8  |.  8D95 60FEFFFF      LEA EDX,DWORD PTR SS:[EBP-1A0]
0049BBBE  |.  8B45 FC            MOV EAX,DWORD PTR SS:[EBP-4]
0049BBC1  |.  8B80 0C030000      MOV EAX,DWORD PTR DS:[EAX+30C]
0049BBC7  |.  E8 942DFCFF        CALL advelb.0045E960
0049BBCC  |.  8B85 60FEFFFF      MOV EAX,DWORD PTR SS:[EBP-1A0]
0049BBD2  |.  B9 7D000000        MOV ECX,7D
0049BBD7  |.  5A                 POP EDX
0049BBD8  |.  E8 ABF4FEFF        CALL advelb.0048B088
0049BBDD  |.  84C0               TEST AL,AL
0049BBDF  |.  74 1A              JE SHORT advelb.0049BBFB
0049BBE1  |.  BA 54BD4900        MOV EDX,advelb.0049BD54                 ;  ASCII "advelb.dat"
0049BBE6  |.  8D45 88            LEA EAX,DWORD PTR SS:[EBP-78]
0049BBE9  |.  E8 5EF0FEFF        CALL advelb.0048AC4C
0049BBEE  |.  A1 E46B4B00        MOV EAX,DWORD PTR DS:[4B6BE4]
0049BBF3  |.  C700 7D000000      MOV DWORD PTR DS:[EAX],7D
0049BBF9  |.  EB 5C              JMP SHORT advelb.0049BC57
0049BBFB  |>  8D95 5CFEFFFF      LEA EDX,DWORD PTR SS:[EBP-1A4]
0049BC01  |.  8B45 FC            MOV EAX,DWORD PTR SS:[EBP-4]
0049BC04  |.  8B80 14030000      MOV EAX,DWORD PTR DS:[EAX+314]
0049BC0A  |.  E8 512DFCFF        CALL advelb.0045E960
0049BC0F  |.  8B85 5CFEFFFF      MOV EAX,DWORD PTR SS:[EBP-1A4]
0049BC15  |.  50                 PUSH EAX
0049BC16  |.  8D95 58FEFFFF      LEA EDX,DWORD PTR SS:[EBP-1A8]
0049BC1C  |.  8B45 FC            MOV EAX,DWORD PTR SS:[EBP-4]
0049BC1F  |.  8B80 0C030000      MOV EAX,DWORD PTR DS:[EAX+30C]
0049BC25  |.  E8 362DFCFF        CALL advelb.0045E960
0049BC2A  |.  8B85 58FEFFFF      MOV EAX,DWORD PTR SS:[EBP-1A8]
0049BC30  |.  B9 87000000        MOV ECX,87
0049BC35  |.  5A                 POP EDX
0049BC36  |.  E8 4DF4FEFF        CALL advelb.0048B088
0049BC3B  |.  84C0               TEST AL,AL
0049BC3D  |.  74 18              JE SHORT advelb.0049BC57
0049BC3F  |.  BA 54BD4900        MOV EDX,advelb.0049BD54                 ;  ASCII "advelb.dat"
0049BC44  |.  8D45 88            LEA EAX,DWORD PTR SS:[EBP-78]
0049BC47  |.  E8 00F0FEFF        CALL advelb.0048AC4C
0049BC4C  |.  A1 E46B4B00        MOV EAX,DWORD PTR DS:[4B6BE4]
0049BC51  |.  C700 87000000      MOV DWORD PTR DS:[EAX],87
0049BC57  |>  A1 546C4B00        MOV EAX,DWORD PTR DS:[4B6C54]
0049BC5C  |.  8038 00            CMP BYTE PTR DS:[EAX],0
0049BC5F  |.  74 4F              JE SHORT advelb.0049BCB0
0049BC61  |.  6A 00              PUSH 0
0049BC63  |.  B9 14BD4900        MOV ECX,advelb.0049BD14                 ;  ASCII "Advanced eLearning Builder"
0049BC68  |.  BA 60BD4900        MOV EDX,advelb.0049BD60                 ;  ASCII "Successfully registered!"
0049BC6D  |.  A1 106F4B00        MOV EAX,DWORD PTR DS:[4B6F10]
0049BC72  |.  8B00               MOV EAX,DWORD PTR DS:[EAX]
0049BC74  |.  E8 173DFEFF        CALL advelb.0047F990
0049BC79  |.  A1 106F4B00        MOV EAX,DWORD PTR DS:[4B6F10]
0049BC7E  |.  8B00               MOV EAX,DWORD PTR DS:[EAX]
0049BC80  |.  BA 84BD4900        MOV EDX,advelb.0049BD84                 ;  ASCII "Advanced eLearning Builder"
0049BC85  |.  E8 D636FEFF        CALL advelb.0047F360
0049BC8A  |.  8D95 54FEFFFF      LEA EDX,DWORD PTR SS:[EBP-1AC]
0049BC90  |.  8B45 FC            MOV EAX,DWORD PTR SS:[EBP-4]
0049BC93  |.  8B80 14030000      MOV EAX,DWORD PTR DS:[EAX+314]
0049BC99  |.  E8 C22CFCFF        CALL advelb.0045E960
0049BC9E  |.  8B95 54FEFFFF      MOV EDX,DWORD PTR SS:[EBP-1AC]
0049BCA4  |.  A1 546F4B00        MOV EAX,DWORD PTR DS:[4B6F54]
0049BCA9  |.  E8 728AF6FF        CALL advelb.00404720
0049BCAE  |.  EB 18              JMP SHORT advelb.0049BCC8
0049BCB0  |>  6A 00              PUSH 0
0049BCB2  |.  B9 14BD4900        MOV ECX,advelb.0049BD14                 ;  ASCII "Advanced eLearning Builder"
0049BCB7  |.  BA 30BD4900        MOV EDX,advelb.0049BD30                 ;  ASCII "Invalid Registration Key!"
0049BCBC  |.  A1 106F4B00        MOV EAX,DWORD PTR DS:[4B6F10]
0049BCC1  |.  8B00               MOV EAX,DWORD PTR DS:[EAX]
0049BCC3  |.  E8 C83CFEFF        CALL advelb.0047F990
0049BCC8  |>  33C0               XOR EAX,EAX
0049BCCA  |.  5A                 POP EDX
0049BCCB  |.  59                 POP ECX
0049BCCC  |.  59                 POP ECX
0049BCCD  |.  64:8910            MOV DWORD PTR FS:[EAX],EDX
0049BCD0  |.  68 0DBD4900        PUSH advelb.0049BD0D
0049BCD5  |>  8D85 54FEFFFF      LEA EAX,DWORD PTR SS:[EBP-1AC]
0049BCDB  |.  BA 06000000        MOV EDX,6
0049BCE0  |.  E8 0B8AF6FF        CALL advelb.004046F0
0049BCE5  |.  8D85 6CFEFFFF      LEA EAX,DWORD PTR SS:[EBP-194]
0049BCEB  |.  BA 04000000        MOV EDX,4
0049BCF0  |.  E8 FB89F6FF        CALL advelb.004046F0
0049BCF5  |.  8D85 7CFEFFFF      LEA EAX,DWORD PTR SS:[EBP-184]
0049BCFB  |.  BA 02000000        MOV EDX,2
0049BD00  |.  E8 EB89F6FF        CALL advelb.004046F0
0049BD05  \\.  C3                 RETN
0049BD06   .^ E9 C583F6FF        JMP advelb.004040D0
0049BD0B   .^ EB C8              JMP SHORT advelb.0049BCD5
0049BD0D   .  8BE5               MOV ESP,EBP
0049BD0F   .  5D                 POP EBP
0049BD10   .  C3                 RETN
\==============================================================
0048B088  /$  55                 PUSH EBP
0048B089  |.  8BEC               MOV EBP,ESP
0048B08B  |.  81C4 D0FEFFFF      ADD ESP,-130
0048B091  |.  53                 PUSH EBX
0048B092  |.  33DB               XOR EBX,EBX
0048B094  |.  899D D0FEFFFF      MOV DWORD PTR SS:[EBP-130],EBX
0048B09A  |.  894D F4            MOV DWORD PTR SS:[EBP-C],ECX          ;  //[EBP-C]=ECX=7Dh
0048B09D  |.  8955 F8            MOV DWORD PTR SS:[EBP-8],EDX
0048B0A0  |.  8945 FC            MOV DWORD PTR SS:[EBP-4],EAX
0048B0A3  |.  8B45 FC            MOV EAX,DWORD PTR SS:[EBP-4]          ;  //试练码
0048B0A6  |.  E8 D19AF7FF        CALL advelb.00404B7C
0048B0AB  |.  8B45 F8            MOV EAX,DWORD PTR SS:[EBP-8]          ;  //用户名
0048B0AE  |.  E8 C99AF7FF        CALL advelb.00404B7C
0048B0B3  |.  33C0               XOR EAX,EAX
0048B0B5  |.  55                 PUSH EBP
0048B0B6  |.  68 A6B14800        PUSH advelb.0048B1A6
0048B0BB  |.  64:FF30            PUSH DWORD PTR FS:[EAX]
0048B0BE  |.  64:8920            MOV DWORD PTR FS:[EAX],ESP
0048B0C1  |.  C645 EB 01         MOV BYTE PTR SS:[EBP-15],1
0048B0C5  |.  8B45 FC            MOV EAX,DWORD PTR SS:[EBP-4]          ;  //试练码
0048B0C8  |.  E8 BF98F7FF        CALL advelb.0040498C                  ;  //取试练码长度
0048B0CD  |.  83F8 0A            CMP EAX,0A
0048B0D0  |.  0F85 AA000000     JNZ advelb.0048B180                   ;  //试练码长度不等于 0Ah 则跳
0048B0D6  |.  8B45 F8            MOV EAX,DWORD PTR SS:[EBP-8]          ;  //用户名
0048B0D9  |.  E8 AE98F7FF        CALL advelb.0040498C                  ;  //取用户名长度
0048B0DE  |.  48                 DEC EAX
0048B0DF  |.  0F8C 9B000000     JL advelb.0048B180                    ;  //用户名长度小于 1 则跳
0048B0E5  |.  8D8D D0FEFFFF      LEA ECX,DWORD PTR SS:[EBP-130]
0048B0EB  |.  8B55 F4            MOV EDX,DWORD PTR SS:[EBP-C]          ;  //EDX=[EBP-C]=7Dh
0048B0EE  |.  8B45 F8            MOV EAX,DWORD PTR SS:[EBP-8]          ;  //用户名
0048B0F1  |.  E8 92FeFFFF        CALL advelb.0048AF88                  ;  //算法 CALL
0048B0F6  |.  8B95 D0FEFFFF      MOV EDX,DWORD PTR SS:[EBP-130]        ;  //真码
0048B0FC  |.  8D85 D4FEFFFF      LEA EAX,DWORD PTR SS:[EBP-12C]
0048B102  |.  B9 FF000000        MOV ECX,0FF
0048B107  |.  E8 5C98F7FF        CALL advelb.00404968
0048B10C  |.  8D95 D4FEFFFF      LEA EDX,DWORD PTR SS:[EBP-12C]
0048B112  |.  8D45 D5            LEA EAX,DWORD PTR SS:[EBP-2B]
0048B115  |.  B1 0A              MOV CL,0A
0048B117  |.  E8 247FF7FF        CALL advelb.00403040
0048B11C  |.  C745 EC 01000000  MOV DWORD PTR SS:[EBP-14],1
0048B123  |>  837D EC 03         /CMP DWORD PTR SS:[EBP-14],3
0048B127  |.  74 48              |JE SHORT advelb.0048B171
0048B129  |.  837D EC 05         |CMP DWORD PTR SS:[EBP-14],5
0048B12D  |.  74 42              |JE SHORT advelb.0048B171
0048B12F  |.  837D EC 08         |CMP DWORD PTR SS:[EBP-14],8
0048B133  |.  74 3C              |JE SHORT advelb.0048B171
0048B135  |.  807D EB 00         |CMP BYTE PTR SS:[EBP-15],0
0048B139  |.  74 2D              |JE SHORT advelb.0048B168
0048B13B  |.  8B45 EC            |MOV EAX,DWORD PTR SS:[EBP-14]
0048B13E  |.  8B55 FC            |MOV EDX,DWORD PTR SS:[EBP-4]
0048B141  |.  48                 |DEC EAX
0048B142  |.  85D2               |TEST EDX,EDX
0048B144  |.  74 05              |JE SHORT advelb.0048B14B
0048B146  |.  3B42 FC            |CMP EAX,DWORD PTR DS:[EDX-4]
0048B149  |.  72 05              |JB SHORT advelb.0048B150
0048B14B  |>  E8 6487F7FF        |CALL advelb.004038B4
0048B150  |>  40                 |INC EAX
0048B151  |.  8A4402 FF         |MOV AL,BYTE PTR DS:[EDX+EAX-1]
0048B155  |.  8B55 EC            |MOV EDX,DWORD PTR SS:[EBP-14]
0048B158  |.  83FA 0A            |CMP EDX,0A
0048B15B  |.  76 05              |JBE SHORT advelb.0048B162
0048B15D  |.  E8 5287F7FF        |CALL advelb.004038B4
0048B162  |>  3A4415 D5         |CMP AL,BYTE PTR SS:[EBP+EDX-2B]
0048B166  |.  74 04              |JE SHORT advelb.0048B16C
0048B168  |>  33C0               |XOR EAX,EAX
0048B16A  |.  EB 02              |JMP SHORT advelb.0048B16E
0048B16C  |>  B0 01              |MOV AL,1
0048B16E  |>  8845 EB            |MOV BYTE PTR SS:[EBP-15],AL
0048B171  |>  FF45 EC            |INC DWORD PTR SS:[EBP-14]
0048B174  |.  837D EC 0B         |CMP DWORD PTR SS:[EBP-14],0B
0048B178  |.^ 75 A9              \JNZ SHORT advelb.0048B123          ;  //循环,真码假码逐位比较
0048B17A  |.  8A45 EB            MOV AL,BYTE PTR SS:[EBP-15]
0048B17D  |.  8845 F3            MOV BYTE PTR SS:[EBP-D],AL
0048B180  |>  33C0               XOR EAX,EAX
0048B182  |.  5A                 POP EDX
0048B183  |.  59                 POP ECX
0048B184  |.  59                 POP ECX
0048B185  |.  64:8910            MOV DWORD PTR FS:[EAX],EDX
0048B188  |.  68 ADB14800        PUSH advelb.0048B1AD
0048B18D  |>  8D85 D0FEFFFF      LEA EAX,DWORD PTR SS:[EBP-130]
0048B193  |.  E8 3495F7FF        CALL advelb.004046CC
0048B198  |.  8D45 F8            LEA EAX,DWORD PTR SS:[EBP-8]
0048B19B  |.  BA 02000000        MOV EDX,2
0048B1A0  |.  E8 4B95F7FF        CALL advelb.004046F0
0048B1A5  \\.  C3                 RETN
0048B1A6   .^ E9 258FF7FF        JMP advelb.004040D0
0048B1AB   .^ EB E0              JMP SHORT advelb.0048B18D
0048B1AD   .  8A45 F3            MOV AL,BYTE PTR SS:[EBP-D]
0048B1B0   .  5B                 POP EBX
0048B1B1   .  8BE5               MOV ESP,EBP
0048B1B3   .  5D                 POP EBP
0048B1B4   .  C3                 RETN
\==============================================================
0048AF88  /$  55                 PUSH EBP
0048AF89  |.  8BEC               MOV EBP,ESP
0048AF8B  |.  51                 PUSH ECX
0048AF8C  |.  B9 05000000        MOV ECX,5
0048AF91  |>  6A 00              /PUSH 0
0048AF93  |.  6A 00              |PUSH 0
0048AF95  |.  49                 |DEC ECX
0048AF96  |.^ 75 F9              \JNZ SHORT advelb.0048AF91
0048AF98  |.  51                 PUSH ECX
0048AF99  |.  874D FC            XCHG DWORD PTR SS:[EBP-4],ECX
0048AF9C  |.  53                 PUSH EBX
0048AF9D  |.  894D F4            MOV DWORD PTR SS:[EBP-C],ECX
0048AFA0  |.  8955 F8            MOV DWORD PTR SS:[EBP-8],EDX          ;  //[EBP-8]=EDX=7Dh
0048AFA3  |.  8945 FC            MOV DWORD PTR SS:[EBP-4],EAX
0048AFA6  |.  8B45 FC            MOV EAX,DWORD PTR SS:[EBP-4]          ;  //用户名
0048AFA9  |.  E8 CE9BF7FF        CALL advelb.00404B7C
0048AFAE  |.  33C0               XOR EAX,EAX
0048AFB0  |.  55                 PUSH EBP
0048AFB1  |.  68 7BB04800        PUSH advelb.0048B07B
0048AFB6  |.  64:FF30            PUSH DWORD PTR FS:[EAX]
0048AFB9  |.  64:8920            MOV DWORD PTR FS:[EAX],ESP
0048AFBC  |.  8D55 F0            LEA EDX,DWORD PTR SS:[EBP-10]
0048AFBF  |.  8B45 FC            MOV EAX,DWORD PTR SS:[EBP-4]          ;  //用户名
0048AFC2  |.  E8 61D9F7FF        CALL advelb.00408928                  ;  //用户名转大写
0048AFC7  |.  8D55 E0            LEA EDX,DWORD PTR SS:[EBP-20]
0048AFCA  |.  8B45 F0            MOV EAX,DWORD PTR SS:[EBP-10]        ;  //大写用户名
0048AFCD  |.  E8 A2FAFFFF        CALL advelb.0048AA74                  ;  //取大写用户名 MD5 值大写
0048AFD2  |.  8B55 E0            MOV EDX,DWORD PTR SS:[EBP-20]        ;  //大写用户名 MD5 值大写
0048AFD5  |.  8D45 F0            LEA EAX,DWORD PTR SS:[EBP-10]
0048AFD8  |.  E8 8797F7FF        CALL advelb.00404764
0048AFDD  |.  FF75 F0            PUSH DWORD PTR SS:[EBP-10]
0048AFE0  |.  8D55 DC            LEA EDX,DWORD PTR SS:[EBP-24]
0048AFE3  |.  8B45 F8            MOV EAX,DWORD PTR SS:[EBP-8]          ;  //EAX=[EBP-8]=7Dh
0048AFE6  |.  E8 F5DDF7FF        CALL advelb.00408DE0                  ;  //7Dh 转 10 进制字符形式,即 125
0048AFEB  |.  FF75 DC            PUSH DWORD PTR SS:[EBP-24]
0048AFEE  |.  8D55 D4            LEA EDX,DWORD PTR SS:[EBP-2C]
0048AFF1  |.  8B45 F8            MOV EAX,DWORD PTR SS:[EBP-8]
0048AFF4  |.  E8 E7DDF7FF        CALL advelb.00408DE0
0048AFF9  |.  8B45 D4            MOV EAX,DWORD PTR SS:[EBP-2C]        ;  //"125"
0048AFFC  |.  8D55 D8            LEA EDX,DWORD PTR SS:[EBP-28]
0048AFFF  |.  E8 70FAFFFF        CALL advelb.0048AA74                  ;  //取字符串"125"MD5 值大写
0048B004  |.  FF75 D8            PUSH DWORD PTR SS:[EBP-28]            ;  //字符串"125"MD5 值大写
0048B007  |.  8D45 EC            LEA EAX,DWORD PTR SS:[EBP-14]
0048B00A  |.  BA 03000000        MOV EDX,3
0048B00F  |.  E8 389AF7FF        CALL advelb.00404A4C                  ;  //将大写用户名 MD5 值大写和字符串"125"及其 MD5 值大写相连
0048B014  |.  8D55 D0            LEA EDX,DWORD PTR SS:[EBP-30]
0048B017  |.  8B45 EC            MOV EAX,DWORD PTR SS:[EBP-14]        ;  //相连字符串
0048B01A  |.  E8 55FAFFFF        CALL advelb.0048AA74                  ;  //取相连字符串 MD5 值大写
0048B01F  |.  8B55 D0            MOV EDX,DWORD PTR SS:[EBP-30]        ;  //相连字符串 MD5 值大写
0048B022  |.  8D45 EC            LEA EAX,DWORD PTR SS:[EBP-14]
0048B025  |.  E8 3A97F7FF        CALL advelb.00404764
0048B02A  |.  8D45 EC            LEA EAX,DWORD PTR SS:[EBP-14]
0048B02D  |.  50                 PUSH EAX
0048B02E  |.  B9 0A000000        MOV ECX,0A
0048B033  |.  BA 07000000        MOV EDX,7
0048B038  |.  8B45 EC            MOV EAX,DWORD PTR SS:[EBP-14]        ;  //相连字符串 MD5 值大写
0048B03B  |.  E8 AC9BF7FF        CALL advelb.00404BEC                  ;  //取相连字符串 MD5 值大写的 7~16 位
0048B040  |.  8B45 F4            MOV EAX,DWORD PTR SS:[EBP-C]
0048B043  |.  8B55 EC            MOV EDX,DWORD PTR SS:[EBP-14]        ;  //相连字符串 MD5 值大写的 7~16 位,即真码
0048B046  |.  E8 D596F7FF        CALL advelb.00404720
0048B04B  |.  33C0               XOR EAX,EAX
0048B04D  |.  5A                 POP EDX
0048B04E  |.  59                 POP ECX
0048B04F  |.  59                 POP ECX
0048B050  |.  64:8910            MOV DWORD PTR FS:[EAX],EDX
0048B053  |.  68 82B04800        PUSH advelb.0048B082
0048B058  |>  8D45 D0            LEA EAX,DWORD PTR SS:[EBP-30]
0048B05B  |.  BA 05000000        MOV EDX,5
0048B060  |.  E8 8B96F7FF        CALL advelb.004046F0
0048B065  |.  8D45 EC            LEA EAX,DWORD PTR SS:[EBP-14]
0048B068  |.  BA 02000000        MOV EDX,2
0048B06D  |.  E8 7E96F7FF        CALL advelb.004046F0
0048B072  |.  8D45 FC            LEA EAX,DWORD PTR SS:[EBP-4]
0048B075  |.  E8 5296F7FF        CALL advelb.004046CC
0048B07A  \\.  C3                 RETN
0048B07B   .^ E9 5090F7FF        JMP advelb.004040D0
0048B080   .^ EB D6              JMP SHORT advelb.0048B058
0048B082   .  5B                 POP EBX
0048B083   .  8BE5               MOV ESP,EBP
0048B085   .  5D                 POP EBP
0048B086   .  C3                 RETN
```



## 【破解总结】

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

## 【算法总结】

有两种注册算法，分别使用了 3 次标准 MD5 运算

\--------------------------------------------------------------

## 【算法注册机】

VB 代码

```vb
Private Sub Command1_Click()
    If Len(Text1.Text) = 0 Then
        Text2.Text = "请输入用户名!"
    Else
        Text2.Text = Mid((MD5(MD5(UCase(Text1.Text)) & "125" & MD5("125"))), 7, 10)  'MD5 模块，"125"也可以是"135"
    End If
End Sub
```

\--------------------------------------------------------------

## 【注册信息】

保存在`advelb.dat`

\--------------------------------------------------------------

感谢飘云老大、猫老大、`Nisy` 老大以及很多前辈们的学习教程以及所有帮助过我的论坛兄弟姐妹们！谢谢

\--------------------------------------------------------------

## 【版权声明】

破文是学习的手记，兴趣是成功的源泉；本破文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!

```text
      _/_/_/   _/   _/   _/_/_/
     _/   _/  _/  _/    _/
    _/_/_/    _/_/     _/_/_/_/
   _/         _/      _/    _/
  _/         _/      _/_/_/ _/    tianxj
```
