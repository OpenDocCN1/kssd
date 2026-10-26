
# [BC][简单]Picture to Icon V1.918 注册算法分析

**标题** Picture to Icon V1.918 注册算法分析
**作者** forever[RCT]
**语言** BC
**保护** `aspack`, 注册码
**难度** 简单
**简介** 能将图片或屏幕的一部分转化为ICON图标，调整图标大小以及从资源库中提取icon。支持BMP, JPEG, GIF, CUR, WMF。
**下载** bf3K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6K6k6Q4x3X3g2K6K9%4W2U0L8W2)9J5k6h3y4G2L8g2)9J5c8Y4y4G2k6Y4c8Q4x3V1j5J5y4K6l9H3y4g2)9J5k6h3S2@1L8h3H3`.

## 正文

这篇帖子没什么技术含量，纯为练手所作。:)

`peid`查看`aspack`加壳。用`UnAspack`脱壳，发现不能运行。不过没关系，`ida`能分析就成。我主要是想用`ida`导出个`map`文件，这样在`od`中好分析。

用`od`加载程序，跟过壳的代码后用`LoadMapEx`加载从`ida`中导出的`map`文件。然后尝试着在注册窗口随便输入点什么，有提示了：

Your registration code is invalid. 呵呵，这个正是我想要的。在`od`中搜索字符串"Your",共有一次，双击这个字符串，来到CPU窗口。

见图：

![](http://bbs.pediy.com/upload/2006/4/image/a1.gif)

向上翻，直到来到这里：

![](http://bbs.pediy.com/upload/2006/4/image/a2.gif)

这个`4210F0`的`call`就是关键了，这里要下个断点。再次输入用户名和密码后来到下面：

```asm
004210F0 >  55              PUSH EBP                                 ; sub_4210F0
004210F1    8BEC            MOV EBP,ESP
004210F3    81C4 74FFFFFF   ADD ESP,-8C
004210F9    56              PUSH ESI
004210FA    57              PUSH EDI
004210FB    B8 C0285100     MOV EAX,<stru_5128C0>
00421100    E8 7B760C00     CALL <@__InitExceptBlockLDTC>
00421105    C745 F8 0100000>MOV DWORD PTR SS:[EBP-8],1
0042110C    8D55 08         LEA EDX,DWORD PTR SS:[EBP+8]
0042110F    8D45 08         LEA EAX,DWORD PTR SS:[EBP+8]
00421112    E8 DD260D00     CALL <System::AnsiString::AnsiString(Sys>
00421117    FF45 F8         INC DWORD PTR SS:[EBP-8]
0042111A    66:C745 EC 0800 MOV WORD PTR SS:[EBP-14],8
00421120    C645 DB 00      MOV BYTE PTR SS:[EBP-25],0
00421124    8D45 08         LEA EAX,DWORD PTR SS:[EBP+8]
00421127    E8 704AFEFF     CALL <System::AnsiString::Length(void)>
0042112C    83F8 2C         CMP EAX,2C                               ; 注册码长度是否44位
0042112F    0F85 3E020000   JNZ <loc_421373>
00421135    BE D6245100     MOV ESI,<a1z1h2a0n0g8y9a>                ; ASCII "1z1h+2a0n-0g8y*9a1n|"
0042113A    8D7D 88         LEA EDI,DWORD PTR SS:[EBP-78]
0042113D    B9 05000000     MOV ECX,5
00421142    F3:A5           REP MOVS DWORD PTR ES:[EDI],DWORD PTR DS>
00421144    A4              MOVS BYTE PTR ES:[EDI],BYTE PTR DS:[ESI]
00421145    8D45 08         LEA EAX,DWORD PTR SS:[EBP+8]
00421148    E8 2B10FEFF     CALL <System::AnsiString::c_str(void)>
0042114D    0FBE50 28       MOVSX EDX,BYTE PTR DS:[EAX+28]           ; 第41位是否等于50h
00421151    83FA 50         CMP EDX,50
00421154    74 23           JE SHORT <loc_421179>
00421156    33C0            XOR EAX,EAX
00421158    50              PUSH EAX
00421159    FF4D F8         DEC DWORD PTR SS:[EBP-8]
0042115C    8D45 08         LEA EAX,DWORD PTR SS:[EBP+8]
0042115F    BA 02000000     MOV EDX,2
00421164    E8 0B270D00     CALL <System::AnsiString::~AnsiString(vo>
00421169    58              POP EAX
0042116A    8B55 DC         MOV EDX,DWORD PTR SS:[EBP-24]
0042116D    64:8915 0000000>MOV DWORD PTR FS:[0],EDX
00421174    E9 19020000     JMP <loc_421392>
00421179 >  8D45 08         LEA EAX,DWORD PTR SS:[EBP+8]             ; loc_421179
0042117C    E8 F70FFEFF     CALL <System::AnsiString::c_str(void)>
00421181    0FBE50 29       MOVSX EDX,BYTE PTR DS:[EAX+29]
00421185    83FA 32         CMP EDX,32                               ; 第42位是否等于32h
00421188    74 23           JE SHORT <loc_4211AD>
0042118A    33C0            XOR EAX,EAX
0042118C    50              PUSH EAX
0042118D    FF4D F8         DEC DWORD PTR SS:[EBP-8]
00421190    8D45 08         LEA EAX,DWORD PTR SS:[EBP+8]
00421193    BA 02000000     MOV EDX,2
00421198    E8 D7260D00     CALL <System::AnsiString::~AnsiString(vo>
0042119D    58              POP EAX
0042119E    8B55 DC         MOV EDX,DWORD PTR SS:[EBP-24]
004211A1    64:8915 0000000>MOV DWORD PTR FS:[0],EDX
004211A8    E9 E5010000     JMP <loc_421392>
004211AD >  8D45 08         LEA EAX,DWORD PTR SS:[EBP+8]             ; loc_4211AD
004211B0    E8 C30FFEFF     CALL <System::AnsiString::c_str(void)>
004211B5    0FBE50 2A       MOVSX EDX,BYTE PTR DS:[EAX+2A]
004211B9    83FA 49         CMP EDX,49                               ; 第43位是否等于49h
004211BC    74 23           JE SHORT <loc_4211E1>
004211BE    33C0            XOR EAX,EAX
004211C0    50              PUSH EAX
004211C1    FF4D F8         DEC DWORD PTR SS:[EBP-8]
004211C4    8D45 08         LEA EAX,DWORD PTR SS:[EBP+8]
004211C7    BA 02000000     MOV EDX,2
004211CC    E8 A3260D00     CALL <System::AnsiString::~AnsiString(vo>
004211D1    58              POP EAX
004211D2    8B55 DC         MOV EDX,DWORD PTR SS:[EBP-24]
004211D5    64:8915 0000000>MOV DWORD PTR FS:[0],EDX
004211DC    E9 B1010000     JMP <loc_421392>
004211E1 >  8D45 08         LEA EAX,DWORD PTR SS:[EBP+8]             ; loc_4211E1
004211E4    E8 8F0FFEFF     CALL <System::AnsiString::c_str(void)>
004211E9    0FBE50 2B       MOVSX EDX,BYTE PTR DS:[EAX+2B]
004211ED    83FA 31         CMP EDX,31                               ; 第44位是否等于31h
004211F0    74 23           JE SHORT <loc_421215>
004211F2    33C0            XOR EAX,EAX
004211F4    50              PUSH EAX
004211F5    FF4D F8         DEC DWORD PTR SS:[EBP-8]
004211F8    8D45 08         LEA EAX,DWORD PTR SS:[EBP+8]
004211FB    BA 02000000     MOV EDX,2
00421200    E8 6F260D00     CALL <System::AnsiString::~AnsiString(vo>
00421205    58              POP EAX
00421206    8B55 DC         MOV EDX,DWORD PTR SS:[EBP-24]
00421209    64:8915 0000000>MOV DWORD PTR FS:[0],EDX
00421210    E9 7D010000     JMP <loc_421392>
00421215 >  8D45 08         LEA EAX,DWORD PTR SS:[EBP+8]             ; loc_421215
00421218    E8 5B0FFEFF     CALL <System::AnsiString::c_str(void)>
0042121D    50              PUSH EAX
0042121E    8D55 A0         LEA EDX,DWORD PTR SS:[EBP-60]
00421221    52              PUSH EDX
00421222    E8 B5720C00     CALL <_strcpy>
00421227    83C4 08         ADD ESP,8
0042122A    0FBE4D A1       MOVSX ECX,BYTE PTR SS:[EBP-5F]
0042122E    83F9 30         CMP ECX,30                               ; 第2位是否等于30h
00421231    0F85 3C010000   JNZ <loc_421373>
00421237    C645 A1 23      MOV BYTE PTR SS:[EBP-5F],23              ; 第2位替换成'#'
0042123B    C645 DB 01      MOV BYTE PTR SS:[EBP-25],1
0042123F    C745 D4 0200000>MOV DWORD PTR SS:[EBP-2C],2              ; 初始化索引
00421246 >  8B45 D4         MOV EAX,DWORD PTR SS:[EBP-2C]
00421249    0FBE5405 88     MOVSX EDX,BYTE PTR SS:[EBP+EAX-78]       ; 取常量字符串中一个字符,位置3开始
0042124E    8B4D D4         MOV ECX,DWORD PTR SS:[EBP-2C]
00421251    0FBE440D 9F     MOVSX EAX,BYTE PTR SS:[EBP+ECX-61]       ; 取注册码中一个字符，位置2开始
00421256    03D0            ADD EDX,EAX                              ; 相加
00421258    8B4D D4         MOV ECX,DWORD PTR SS:[EBP-2C]
0042125B    0FBE440D A0     MOVSX EAX,BYTE PTR SS:[EBP+ECX-60]       ; 取注册码中一个字符，位置3开始
00421260    33D0            XOR EDX,EAX                              ; 异或
00421262    8B4D D4         MOV ECX,DWORD PTR SS:[EBP-2C]
00421265    0FBE440D 88     MOVSX EAX,BYTE PTR SS:[EBP+ECX-78]       ; 取常量字符串中一个字符，位置3开始
0042126A    33D0            XOR EDX,EAX                              ; 异或
0042126C    52              PUSH EDX
0042126D    E8 26010000     CALL <sub_421398>
00421272    59              POP ECX
00421273    B9 1A000000     MOV ECX,1A
00421278    99              CDQ
00421279    F7F9            IDIV ECX                                 ; 除以26
0042127B    83C2 41         ADD EDX,41                               ; 加上'A'
0042127E    8B45 D4         MOV EAX,DWORD PTR SS:[EBP-2C]
00421281    0FBE4C05 A9     MOVSX ECX,BYTE PTR SS:[EBP+EAX-57]       ; 和第12个字符开始比较
00421286    3BD1            CMP EDX,ECX
00421288    74 06           JE SHORT <loc_421290>
0042128A    C645 DB 00      MOV BYTE PTR SS:[EBP-25],0
0042128E    EB 09           JMP SHORT <loc_421299>
00421290 >  FF45 D4         INC DWORD PTR SS:[EBP-2C]                ; 索引加1
```
