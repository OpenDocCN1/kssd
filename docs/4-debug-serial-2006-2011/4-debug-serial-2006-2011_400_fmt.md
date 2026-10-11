
# CDEdit V1.146 算法分析

> 原文：[`www.pediy.com/kssd/pediy10/66249.html`](https://www.pediy.com/kssd/pediy10/66249.html)

【破文标题】`CDEdit` `V1.146` 算法分析

【破文作者】`tianxj`

【作者邮箱】`tianxj_2007@126.com`

【破解工具】`PEiD`、`OD`

【破解平台】`Windows` `XP`

【软件名称】`CDEdit` `V1.146` (2008-06-05 版)

【软件大小】848 `KB`

【软件语言】英文

【软件类别】国外软件 / 共享软件 / 媒体制作

【原版下载】自己搜索下

【保护方式】注册码

【软件简介】帮助你设计 `CD-Audio`、`CD-Rom`、`CD-RW` 等的光盘盒封面，它允许你使用不同字形，色彩和影像，可调整字型大小，影像大小，使用了简单的操作界面，能让你迅速又容易的设计光盘盒封面。

【破解声明】我是一只小菜鸟，偶得一点心得，愿与大家分享：）

初学 `Crack`，只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！

--------------------------------------------------------------

【破解内容】

--------------------------------------------------------------

****************************************************************

一、运行程序，进行注册，输入错误的注册信息进行检测，有提示信息

`"Sorry, this registration code is invalid."`

****************************************************************

二、用 `PEiD` 对这个软件查壳，为 `Borland` `Delphi` `6.0` - `7.0`

****************************************************************

三、运行 `OD`，打开 `CDEdit`，`F9` 运行，输入假注册信息注册，出现错误对话框，`F12` 暂停，`alt+K`

调用堆栈 , 项目 19

地址=`0013F220`

堆栈=`004CE9CC`

程序过程 / 参数=`?` `CDEdit.004540E4`

调用来自=`CDEdit.004CE9C7`

结构=`0013F21C`

================================================================

```
代码:
```



```
004CE818  /.  55            PUSH EBP
004CE819  |.  8BEC          MOV EBP,ESP
004CE81B  |.  B9 06000000   MOV ECX,6
004CE820  |>  6A 00         /PUSH 0
004CE822  |.  6A 00         |PUSH 0
004CE824  |.  49            |DEC ECX
004CE825  |.^ 75 F9         \JNZ SHORT CDEdit.004CE820
004CE827  |.  51            PUSH ECX
004CE828  |.  53            PUSH EBX
004CE829  |.  8BD8          MOV EBX,EAX
004CE82B  |.  33C0          XOR EAX,EAX
004CE82D  |.  55            PUSH EBP
004CE82E  |.  68 1EEA4C00   PUSH CDEdit.004CEA1E
004CE833  |.  64:FF30       PUSH DWORD PTR FS:[EAX]
004CE836  |.  64:8920       MOV DWORD PTR FS:[EAX],ESP
004CE839  |.  8D55 FC       LEA EDX,DWORD PTR SS:[EBP-4]
004CE83C  |.  8B83 F8020000 MOV EAX,DWORD PTR DS:[EBX+2F8]
004CE842  |.  E8 89C3F8FF   CALL CDEdit.0045ABD0
004CE847  |.  837D FC 00    CMP DWORD PTR SS:[EBP-4],0
004CE84B  |.  74 14         JE SHORT CDEdit.004CE861                ;  //用户名为空则跳
004CE84D  |.  8D55 F8       LEA EDX,DWORD PTR SS:[EBP-8]
004CE850  |.  8B83 00030000 MOV EAX,DWORD PTR DS:[EBX+300]
004CE856  |.  E8 75C3F8FF   CALL CDEdit.0045ABD0
004CE85B  |.  837D F8 00    CMP DWORD PTR SS:[EBP-8],0
004CE85F  |.  75 38         JNZ SHORT CDEdit.004CE899               ;  //注册码不为空则跳
004CE861  |>  6A 40         PUSH 40
004CE863  |.  8D55 F4       LEA EDX,DWORD PTR SS:[EBP-C]
004CE866  |.  A1 40675100   MOV EAX,DWORD PTR DS:[516740]
004CE86B  |.  8B00          MOV EAX,DWORD PTR DS:[EAX]
004CE86D  |.  E8 0A52F8FF   CALL CDEdit.00453A7C
004CE872  |.  8B45 F4       MOV EAX,DWORD PTR SS:[EBP-C]
004CE875  |.  E8 B267F3FF   CALL CDEdit.0040502C
004CE87A  |.  50            PUSH EAX
004CE87B  |.  A1 7C8E5100   MOV EAX,DWORD PTR DS:[518E7C]
004CE880  |.  E8 A767F3FF   CALL CDEdit.0040502C
004CE885  |.  8BD0          MOV EDX,EAX
004CE887  |.  A1 40675100   MOV EAX,DWORD PTR DS:[516740]
004CE88C  |.  8B00          MOV EAX,DWORD PTR DS:[EAX]
004CE88E  |.  59            POP ECX
004CE88F  |.  E8 5058F8FF   CALL CDEdit.004540E4
004CE894  |.  E9 33010000   JMP CDEdit.004CE9CC
004CE899  |>  8D55 F0       LEA EDX,DWORD PTR SS:[EBP-10]
004CE89C  |.  8B83 F8020000 MOV EAX,DWORD PTR DS:[EBX+2F8]
004CE8A2  |.  E8 29C3F8FF   CALL CDEdit.0045ABD0
004CE8A7  |.  8B45 F0       MOV EAX,DWORD PTR SS:[EBP-10]         ;  //EAX=[EBP-10]=用户名
004CE8AA  |.  BA 34EA4C00   MOV EDX,CDEdit.004CEA34               ;  //temporary cdedit code
004CE8AF  |.  E8 C466F3FF   CALL CDEdit.00404F78
004CE8B4  |.  75 37         JNZ SHORT CDEdit.004CE8ED             ;  //用户名不为"Temporary CDEdit Code"则跳
004CE8B6  |.  8D55 EC       LEA EDX,DWORD PTR SS:[EBP-14]
004CE8B9  |.  8B83 00030000 MOV EAX,DWORD PTR DS:[EBX+300]
004CE8BF  |.  E8 0CC3F8FF   CALL CDEdit.0045ABD0
004CE8C4  |.  8B45 EC       MOV EAX,DWORD PTR SS:[EBP-14]         ;  //EAX=[EBP-14]=试练码
004CE8C7  |.  50            PUSH EAX
004CE8C8  |.  8D55 E8       LEA EDX,DWORD PTR SS:[EBP-18]
004CE8CB  |.  8B83 F8020000 MOV EAX,DWORD PTR DS:[EBX+2F8]
004CE8D1  |.  E8 FAC2F8FF   CALL CDEdit.0045ABD0
004CE8D6  |.  8B55 E8       MOV EDX,DWORD PTR SS:[EBP-18]
004CE8D9  |.  58            POP EAX
004CE8DA  |.  E8 9966F3FF   CALL CDEdit.00404F78
004CE8DF  |.  75 0C         JNZ SHORT CDEdit.004CE8ED             ;  //试练码不为"Temporary CDEdit Code"则跳
004CE8E1  |.  8BC3          MOV EAX,EBX
004CE8E3  |.  E8 301FF8FF   CALL CDEdit.00450818
004CE8E8  |.  E9 DF000000   JMP CDEdit.004CE9CC
004CE8ED  |>  8D55 E0       LEA EDX,DWORD PTR SS:[EBP-20]
004CE8F0  |.  8B83 F8020000 MOV EAX,DWORD PTR DS:[EBX+2F8]
004CE8F6  |.  E8 D5C2F8FF   CALL CDEdit.0045ABD0
004CE8FB  |.  8B45 E0       MOV EAX,DWORD PTR SS:[EBP-20]         ;  //EAX=EBP-20]=用户名
004CE8FE  |.  8D55 E4       LEA EDX,DWORD PTR SS:[EBP-1C]
004CE901  |.  E8 4AE9FFFF   CALL CDEdit.004CD250                 ;  //算法 CALL
004CE906  |.  8B45 E4       MOV EAX,DWORD PTR SS:[EBP-1C]         ;  //EAX=[EBP-1C]=真码
004CE909  |.  50            PUSH EAX
004CE90A  |.  8D55 DC       LEA EDX,DWORD PTR SS:[EBP-24]
004CE90D  |.  8B83 00030000 MOV EAX,DWORD PTR DS:[EBX+300]
004CE913  |.  E8 B8C2F8FF   CALL CDEdit.0045ABD0
004CE918  |.  8B55 DC       MOV EDX,DWORD PTR SS:[EBP-24]         ;  //EDX=[EBP-24]=试练码
004CE91B  |.  58            POP EAX                                ;  //EAX=真码
004CE91C  |.  E8 5766F3FF   CALL CDEdit.00404F78                  ;  //比较 CALL
004CE921  |.  75 76         JNZ SHORT CDEdit.004CE999             ;  //关键跳转
004CE923  |.  8D55 D8       LEA EDX,DWORD PTR SS:[EBP-28]
004CE926  |.  8B83 00030000 MOV EAX,DWORD PTR DS:[EBX+300]
004CE92C  |.  E8 9FC2F8FF   CALL CDEdit.0045ABD0
004CE931  |.  8B45 D8       MOV EAX,DWORD PTR SS:[EBP-28]
004CE934  |.  50            PUSH EAX
004CE935  |.  8D55 D4       LEA EDX,DWORD PTR SS:[EBP-2C]
004CE938  |.  8B83 F8020000 MOV EAX,DWORD PTR DS:[EBX+2F8]
004CE93E  |.  E8 8DC2F8FF   CALL CDEdit.0045ABD0
004CE943  |.  8B55 D4       MOV EDX,DWORD PTR SS:[EBP-2C]
004CE946  |.  8BC3          MOV EAX,EBX
004CE948  |.  59            POP ECX
004CE949  |.  E8 96010000   CALL CDEdit.004CEAE4
004CE94E  |.  6A 40         PUSH 40
004CE950  |.  8D55 D0       LEA EDX,DWORD PTR SS:[EBP-30]
004CE953  |.  A1 40675100   MOV EAX,DWORD PTR DS:[516740]
004CE958  |.  8B00          MOV EAX,DWORD PTR DS:[EAX]
004CE95A  |.  E8 1D51F8FF   CALL CDEdit.00453A7C
004CE95F  |.  8B45 D0       MOV EAX,DWORD PTR SS:[EBP-30]
004CE962  |.  E8 C566F3FF   CALL CDEdit.0040502C
004CE967  |.  50            PUSH EAX
004CE968  |.  A1 808E5100   MOV EAX,DWORD PTR DS:[518E80]
004CE96D  |.  E8 BA66F3FF   CALL CDEdit.0040502C
004CE972  |.  8BD0          MOV EDX,EAX
004CE974  |.  A1 40675100   MOV EAX,DWORD PTR DS:[516740]
004CE979  |.  8B00          MOV EAX,DWORD PTR DS:[EAX]
004CE97B  |.  59            POP ECX
004CE97C  |.  E8 6357F8FF   CALL CDEdit.004540E4
004CE981  |.  A1 788E5100   MOV EAX,DWORD PTR DS:[518E78]
004CE986  |.  E8 2D20F8FF   CALL CDEdit.004509B8
004CE98B  |.  A1 40675100   MOV EAX,DWORD PTR DS:[516740]
004CE990  |.  8B00          MOV EAX,DWORD PTR DS:[EAX]
004CE992  |.  E8 A956F8FF   CALL CDEdit.00454040
004CE997  |.  EB 33         JMP SHORT CDEdit.004CE9CC
004CE999  |>  6A 40         PUSH 40
004CE99B  |.  8D55 CC       LEA EDX,DWORD PTR SS:[EBP-34]
004CE99E  |.  A1 40675100   MOV EAX,DWORD PTR DS:[516740]
004CE9A3  |.  8B00          MOV EAX,DWORD PTR DS:[EAX]
004CE9A5  |.  E8 D250F8FF   CALL CDEdit.00453A7C
004CE9AA  |.  8B45 CC       MOV EAX,DWORD PTR SS:[EBP-34]
004CE9AD  |.  E8 7A66F3FF   CALL CDEdit.0040502C
004CE9B2  |.  50            PUSH EAX
004CE9B3  |.  A1 7C8E5100   MOV EAX,DWORD PTR DS:[518E7C]
004CE9B8  |.  E8 6F66F3FF   CALL CDEdit.0040502C
004CE9BD  |.  8BD0          MOV EDX,EAX
004CE9BF  |.  A1 40675100   MOV EAX,DWORD PTR DS:[516740]
004CE9C4  |.  8B00          MOV EAX,DWORD PTR DS:[EAX]
004CE9C6  |.  59            POP ECX
004CE9C7  |.  E8 1857F8FF   CALL CDEdit.004540E4
004CE9CC  |>  33C0          XOR EAX,EAX
004CE9CE  |.  5A            POP EDX
004CE9CF  |.  59            POP ECX
004CE9D0  |.  59            POP ECX
004CE9D1  |.  64:8910       MOV DWORD PTR FS:[EAX],EDX
004CE9D4  |.  68 25EA4C00   PUSH CDEdit.004CEA25
004CE9D9  |>  8D45 CC       LEA EAX,DWORD PTR SS:[EBP-34]
004CE9DC  |.  BA 02000000   MOV EDX,2
004CE9E1  |.  E8 BA61F3FF   CALL CDEdit.00404BA0
004CE9E6  |.  8D45 D4       LEA EAX,DWORD PTR SS:[EBP-2C]
004CE9E9  |.  BA 04000000   MOV EDX,4
004CE9EE  |.  E8 AD61F3FF   CALL CDEdit.00404BA0
004CE9F3  |.  8D45 E4       LEA EAX,DWORD PTR SS:[EBP-1C]
004CE9F6  |.  E8 8161F3FF   CALL CDEdit.00404B7C
004CE9FB  |.  8D45 E8       LEA EAX,DWORD PTR SS:[EBP-18]
004CE9FE  |.  BA 03000000   MOV EDX,3
004CEA03  |.  E8 9861F3FF   CALL CDEdit.00404BA0
004CEA08  |.  8D45 F4       LEA EAX,DWORD PTR SS:[EBP-C]
004CEA0B  |.  E8 6C61F3FF   CALL CDEdit.00404B7C
004CEA10  |.  8D45 F8       LEA EAX,DWORD PTR SS:[EBP-8]
004CEA13  |.  BA 02000000   MOV EDX,2
004CEA18  |.  E8 8361F3FF   CALL CDEdit.00404BA0
004CEA1D  \\.  C3            RETN
004CEA1E   .^ E9 5D5BF3FF   JMP CDEdit.00404580
004CEA23   .^ EB B4         JMP SHORT CDEdit.004CE9D9
004CEA25   .  5B            POP EBX
004CEA26   .  8BE5          MOV ESP,EBP
004CEA28   .  5D            POP EBP
004CEA29   .  C3            RETN
==============================================================
004CD250   $  55            PUSH EBP
004CD251   .  8BEC          MOV EBP,ESP
004CD253   .  81C4 E0FDFFFF ADD ESP,-220
004CD259   .  53            PUSH EBX
004CD25A   .  56            PUSH ESI
004CD25B   .  57            PUSH EDI
004CD25C   .  33C9          XOR ECX,ECX
004CD25E   .  898D E0FDFFFF MOV DWORD PTR SS:[EBP-220],ECX
004CD264   .  894D F0       MOV DWORD PTR SS:[EBP-10],ECX
004CD267   .  894D EC       MOV DWORD PTR SS:[EBP-14],ECX
004CD26A   .  8955 F8       MOV DWORD PTR SS:[EBP-8],EDX
004CD26D   .  8945 FC       MOV DWORD PTR SS:[EBP-4],EAX
004CD270   .  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
004CD273   .  E8 A47DF3FF   CALL CDEdit.0040501C
004CD278   .  33C0          XOR EAX,EAX
004CD27A   .  55            PUSH EBP
004CD27B   .  68 F6D34C00   PUSH CDEdit.004CD3F6
004CD280   .  64:FF30       PUSH DWORD PTR FS:[EAX]
004CD283   .  64:8920       MOV DWORD PTR FS:[EAX],ESP
004CD286   .  8D45 F0       LEA EAX,DWORD PTR SS:[EBP-10]
004CD289   .  E8 EE78F3FF   CALL CDEdit.00404B7C
004CD28E   .  EB 0B         JMP SHORT CDEdit.004CD29B
004CD290   >  8D45 FC       LEA EAX,DWORD PTR SS:[EBP-4]
004CD293   .  8B55 FC       MOV EDX,DWORD PTR SS:[EBP-4]
004CD296   .  E8 A17BF3FF   CALL CDEdit.00404E3C
004CD29B   >  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
004CD29E   .  E8 917BF3FF   CALL CDEdit.00404E34
004CD2A3   .  85C0          TEST EAX,EAX
004CD2A5   .  7E 0D         JLE SHORT CDEdit.004CD2B4
004CD2A7   .  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
004CD2AA   .  E8 857BF3FF   CALL CDEdit.00404E34
004CD2AF   .  83F8 09      CMP EAX,9
004CD2B2   .^ 7C DC        JL SHORT CDEdit.004CD290                ;  //取用户名长度与 9 比较,若小于 9,则重复用户名,直到长度

大于等于 9
004CD2B4   >  8D85 E4FEFFFF LEA EAX,DWORD PTR SS:[EBP-11C]
004CD2BA   .  8B55 FC       MOV EDX,DWORD PTR SS:[EBP-4]          ;  //重复后的用户名
004CD2BD   .  E8 A2C9F3FF   CALL CDEdit.00409C64
004CD2C2   .  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
004CD2C5   .  E8 6A7BF3FF   CALL CDEdit.00404E34                  ;  //取重复后的用户名的长度
004CD2CA   .  48            DEC EAX
004CD2CB   .  85C0          TEST EAX,EAX
004CD2CD   .  0F8C EA000000 JL CDEdit.004CD3BD
004CD2D3   .  40            INC EAX
004CD2D4   .  8945 E8       MOV DWORD PTR SS:[EBP-18],EAX         ;  //[EBP-18]=EAX=重复后的用户名的长度
004CD2D7   .  C745 F4 00000>MOV DWORD PTR SS:[EBP-C],0            ;  //[EBP-C]=0
004CD2DE   .  8D85 E4FEFFFF LEA EAX,DWORD PTR SS:[EBP-11C]
004CD2E4   .  8945 E4       MOV DWORD PTR SS:[EBP-1C],EAX
004CD2E7   >  8B45 E4       MOV EAX,DWORD PTR SS:[EBP-1C]         ;  //重复后的用户名
004CD2EA   .  33DB          XOR EBX,EBX                            ;  //EBX=0
004CD2EC   .  8A18          MOV BL,BYTE PTR DS:[EAX]              ;  //依次取重复后的用户名的 ASC 值
004CD2EE   .  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]          ;  //重复后的用户名
004CD2F1   .  E8 3E7BF3FF   CALL CDEdit.00404E34                  ;  //取重复后的用户名的长度
004CD2F6   .  2B45 F4       SUB EAX,DWORD PTR SS:[EBP-C]          ;  //EAX=EAX-[EBP-C],[EBP-C]初始值为 0
004CD2F9   .  8D04C0        LEA EAX,DWORD PTR DS:[EAX+EAX*8]      ;  //EAX=[EAX+EAX*8]
004CD2FC   .  B9 03000000   MOV ECX,3                             ;  //ECX=3
004CD301   .  99            CDQ
004CD302   .  F7F9          IDIV ECX                              ;  //EAX/ECX,商送 EAX,余送 EDX
004CD304   .  03D8          ADD EBX,EAX                           ;  //EBX=EBX+EAX
004CD306   .  83FB 09       CMP EBX,9                             ;  //EBX 与 9 比较
004CD309   .  0F8E 83000000 JLE CDEdit.004CD392                   ;  //小于等于则跳
004CD30F   .  8D55 EC       LEA EDX,DWORD PTR SS:[EBP-14]
004CD312   .  8BC3          MOV EAX,EBX                           ;  //EAX=EBX
004CD314   .  E8 17C0F3FF   CALL CDEdit.00409330                  ;  //将 EAX 转为 10 进制字符形式
004CD319   .  EB 77         JMP SHORT CDEdit.004CD392
004CD31B   >  33FF          XOR EDI,EDI                           ;  //EDI=0
004CD31D   .  8D85 E4FDFFFF LEA EAX,DWORD PTR SS:[EBP-21C]
004CD323   .  8B55 EC       MOV EDX,DWORD PTR SS:[EBP-14]         ;  //10 进制字符串
004CD326   .  E8 39C9F3FF   CALL CDEdit.00409C64
004CD32B   .  33D2          XOR EDX,EDX                           ;  //EDX=0
004CD32D   .  55            PUSH EBP
004CD32E   .  68 81D34C00   PUSH CDEdit.004CD381
004CD333   .  64:FF32       PUSH DWORD PTR FS:[EDX]
004CD336   .  64:8922       MOV DWORD PTR FS:[EDX],ESP
004CD339   .  8B45 EC       MOV EAX,DWORD PTR SS:[EBP-14]         ;  //10 进制字符串
004CD33C   .  E8 F37AF3FF   CALL CDEdit.00404E34                  ;  //取 10 进制字符串的长度
004CD341   .  8BF0          MOV ESI,EAX                           ;  //ESI=EAX=10 进制字符串的长度
004CD343   .  4E            DEC ESI                                ;  //ESI=ESI-1
004CD344   .  85F6          TEST ESI,ESI
004CD346   .  7C 25         JL SHORT CDEdit.004CD36D              ;  //小于则跳
004CD348   .  46            INC ESI                                ;  //ESI=ESI+1
004CD349   .  8D9D E4FDFFFF LEA EBX,DWORD PTR SS:[EBP-21C]        ;  //10 进制字符串
004CD34F   >  8D85 E0FDFFFF LEA EAX,DWORD PTR SS:[EBP-220]
004CD355   .  8A13          MOV DL,BYTE PTR DS:[EBX]              ;  //依次取 10 进制字符串 ASC 值
004CD357   .  E8 007AF3FF   CALL CDEdit.00404D5C
004CD35C   .  8B85 E0FDFFFF MOV EAX,DWORD PTR SS:[EBP-220]
004CD362   .  E8 69C0F3FF   CALL CDEdit.004093D0                  ;  //将 10 进制字符串的数字送入 EAX
004CD367   .  03F8          ADD EDI,EAX                           ;  //EDI=EDI+EAX
004CD369   .  43            INC EBX                                ;  //EBX=EBX+1
004CD36A   .  4E            DEC ESI                                ;  //ESI=ESI-1
004CD36B   .^ 75 E2        JNZ SHORT CDEdit.004CD34F             ;  //循环,将 10 进制字符串的数字逐个相加
004CD36D   >  8D55 EC       LEA EDX,DWORD PTR SS:[EBP-14]
004CD370   .  8BC7          MOV EAX,EDI                           ;  //EAX=EDI
004CD372   .  E8 B9BFF3FF   CALL CDEdit.00409330
004CD377   .  33C0          XOR EAX,EAX
004CD379   .  5A            POP EDX
004CD37A   .  59            POP ECX
004CD37B   .  59            POP ECX
004CD37C   .  64:8910       MOV DWORD PTR FS:[EAX],EDX
004CD37F   .  EB 11         JMP SHORT CDEdit.004CD392
004CD381   .^ E9 466FF3FF   JMP CDEdit.004042CC
004CD386   .  E8 A972F3FF   CALL CDEdit.00404634
004CD38B   .  EB 3B         JMP SHORT CDEdit.004CD3C8
004CD38D   .  E8 A272F3FF   CALL CDEdit.00404634
004CD392   >  8B45 EC       MOV EAX,DWORD PTR SS:[EBP-14]         ;  //10 进制字符串或累加值
004CD395   .  E8 36C0F3FF   CALL CDEdit.004093D0                  ;  //10 进制字符转 16 进制或累加值送入 EAX
004CD39A   .  83F8 09      CMP EAX,9                             ;  //EAX 与 9 比较
004CD39D   .^ 0F8F 78FFFFFF JG CDEdit.004CD31B                   ;  //大于则跳
004CD3A3   .  8D45 F0       LEA EAX,DWORD PTR SS:[EBP-10]
004CD3A6   .  8B55 EC       MOV EDX,DWORD PTR SS:[EBP-14]
004CD3A9   .  E8 8E7AF3FF   CALL CDEdit.00404E3C
004CD3AE   .  FF45 F4       INC DWORD PTR SS:[EBP-C]             ;  //[EBP-C]=[EBP-C]+1
004CD3B1   .  FF45 E4       INC DWORD PTR SS:[EBP-1C]            ;  //[EBP-1C]=[EBP-1C]+1
004CD3B4   .  FF4D E8       DEC DWORD PTR SS:[EBP-18]            ;  //[EBP-18]=[EBP-18]-1
004CD3B7   .^ 0F85 2AFFFFFF JNZ CDEdit.004CD2E7                  ;  //循环
004CD3BD   >  8B45 F8       MOV EAX,DWORD PTR SS:[EBP-8]
004CD3C0   .  8B55 F0       MOV EDX,DWORD PTR SS:[EBP-10]
004CD3C3   .  E8 0878F3FF   CALL CDEdit.00404BD0
004CD3C8   >  33C0          XOR EAX,EAX
004CD3CA   .  5A            POP EDX
004CD3CB   .  59            POP ECX
004CD3CC   .  59            POP ECX
004CD3CD   .  64:8910       MOV DWORD PTR FS:[EAX],EDX
004CD3D0   .  68 FDD34C00   PUSH CDEdit.004CD3FD
004CD3D5   >  8D85 E0FDFFFF LEA EAX,DWORD PTR SS:[EBP-220]
004CD3DB   .  E8 9C77F3FF   CALL CDEdit.00404B7C
004CD3E0   .  8D45 EC       LEA EAX,DWORD PTR SS:[EBP-14]
004CD3E3   .  BA 02000000   MOV EDX,2
004CD3E8   .  E8 B377F3FF   CALL CDEdit.00404BA0
004CD3ED   .  8D45 FC       LEA EAX,DWORD PTR SS:[EBP-4]
004CD3F0   .  E8 8777F3FF   CALL CDEdit.00404B7C
004CD3F5   .  C3            RETN
004CD3F6   .^ E9 8571F3FF   JMP CDEdit.00404580
004CD3FB   .^ EB D8         JMP SHORT CDEdit.004CD3D5
004CD3FD   .  5F            POP EDI
004CD3FE   .  5E            POP ESI
004CD3FF   .  5B            POP EBX
004CD400   .  8BE5          MOV ESP,EBP
004CD402   .  5D            POP EBP
004CD403   .  C3            RETN
==============================================================
右键超级字串参考查找 `ASCII`.发现还有黑名单
004CD404  /$  55            PUSH EBP
004CD405  |.  8BEC          MOV EBP,ESP
004CD407  |.  33C9          XOR ECX,ECX
004CD409  |.  51            PUSH ECX
004CD40A  |.  51            PUSH ECX
004CD40B  |.  51            PUSH ECX
004CD40C  |.  51            PUSH ECX
004CD40D  |.  53            PUSH EBX
004CD40E  |.  8BD8          MOV EBX,EAX
004CD410  |.  33C0          XOR EAX,EAX
004CD412  |.  55            PUSH EBP
004CD413  |.  68 AED54C00   PUSH CDEdit.004CD5AE
004CD418  |.  64:FF30       PUSH DWORD PTR FS:[EAX]
004CD41B  |.  64:8920       MOV DWORD PTR FS:[EAX],ESP
004CD41E  |.  B2 01         MOV DL,1
004CD420  |.  A1 A4534700   MOV EAX,DWORD PTR DS:[4753A4]
004CD425  |.  E8 7A80FAFF   CALL CDEdit.004754A4
004CD42A  |.  8945 FC       MOV DWORD PTR SS:[EBP-4],EAX
004CD42D  |.  33C0          XOR EAX,EAX
004CD42F  |.  55            PUSH EBP
004CD430  |.  68 8CD54C00   PUSH CDEdit.004CD58C
004CD435  |.  64:FF30       PUSH DWORD PTR FS:[EAX]
004CD438  |.  64:8920       MOV DWORD PTR FS:[EAX],ESP
004CD43B  |.  BA 02000080   MOV EDX,80000002
004CD440  |.  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
004CD443  |.  E8 FC80FAFF   CALL CDEdit.00475544
004CD448  |.  B1 01         MOV CL,1
004CD44A  |.  BA C4D54C00   MOV EDX,CDEdit.004CD5C4              ;  \software\stefano falda\cdedit
004CD44F  |.  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
004CD452  |.  E8 2D82FAFF   CALL CDEdit.00475684
004CD457  |.  BA ECD54C00   MOV EDX,CDEdit.004CD5EC              ;  username
004CD45C  |.  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
004CD45F  |.  E8 A487FAFF   CALL CDEdit.00475C08
004CD464  |.  84C0          TEST AL,AL
004CD466  |.  0F84 8F000000 JE CDEdit.004CD4FB
004CD46C  |.  BA 00D64C00   MOV EDX,CDEdit.004CD600              ;  code
004CD471  |.  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
004CD474  |.  E8 8F87FAFF   CALL CDEdit.00475C08
004CD479  |.  84C0          TEST AL,AL
004CD47B  |.  74 7E         JE SHORT CDEdit.004CD4FB
004CD47D  |.  8D4D F8       LEA ECX,DWORD PTR SS:[EBP-8]
004CD480  |.  BA ECD54C00   MOV EDX,CDEdit.004CD5EC              ;  username
004CD485  |.  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
004CD488  |.  E8 EF84FAFF   CALL CDEdit.0047597C
004CD48D  |.  8D4D F4       LEA ECX,DWORD PTR SS:[EBP-C]
004CD490  |.  BA 00D64C00   MOV EDX,CDEdit.004CD600              ;  code
004CD495  |.  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
004CD498  |.  E8 DF84FAFF   CALL CDEdit.0047597C
004CD49D  |.  837D F8 00    CMP DWORD PTR SS:[EBP-8],0
004CD4A1  |.  0F84 85000000 JE CDEdit.004CD52C
004CD4A7  |.  837D F4 00    CMP DWORD PTR SS:[EBP-C],0
004CD4AB  |.  74 7F         JE SHORT CDEdit.004CD52C
004CD4AD  |.  837D F8 00    CMP DWORD PTR SS:[EBP-8],0
004CD4B1  |.  74 79         JE SHORT CDEdit.004CD52C
004CD4B3  |.  8D55 F0       LEA EDX,DWORD PTR SS:[EBP-10]
004CD4B6  |.  8B45 F8       MOV EAX,DWORD PTR SS:[EBP-8]
004CD4B9  |.  E8 92FDFFFF   CALL CDEdit.004CD250
004CD4BE  |.  8B45 F0       MOV EAX,DWORD PTR SS:[EBP-10]
004CD4C1  |.  8B55 F4       MOV EDX,DWORD PTR SS:[EBP-C]
004CD4C4  |.  E8 AF7AF3FF   CALL CDEdit.00404F78
004CD4C9  |.  75 1B         JNZ SHORT CDEdit.004CD4E6
004CD4CB  |.  C603 01       MOV BYTE PTR DS:[EBX],1
004CD4CE  |.  8D43 04       LEA EAX,DWORD PTR DS:[EBX+4]
004CD4D1  |.  8B55 F8       MOV EDX,DWORD PTR SS:[EBP-8]
004CD4D4  |.  E8 F776F3FF   CALL CDEdit.00404BD0
004CD4D9  |.  8D43 08       LEA EAX,DWORD PTR DS:[EBX+8]
004CD4DC  |.  8B55 F4       MOV EDX,DWORD PTR SS:[EBP-C]
004CD4DF  |.  E8 EC76F3FF   CALL CDEdit.00404BD0
004CD4E4  |.  EB 46         JMP SHORT CDEdit.004CD52C
004CD4E6  |>  C603 00       MOV BYTE PTR DS:[EBX],0
004CD4E9  |.  8D43 04       LEA EAX,DWORD PTR DS:[EBX+4]
004CD4EC  |.  E8 8B76F3FF   CALL CDEdit.00404B7C
004CD4F1  |.  8D43 08       LEA EAX,DWORD PTR DS:[EBX+8]
004CD4F4  |.  E8 8376F3FF   CALL CDEdit.00404B7C
004CD4F9  |.  EB 31         JMP SHORT CDEdit.004CD52C
004CD4FB  |>  33C9          XOR ECX,ECX
004CD4FD  |.  BA ECD54C00   MOV EDX,CDEdit.004CD5EC              ;  username
004CD502  |.  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
004CD505  |.  E8 4684FAFF   CALL CDEdit.00475950
004CD50A  |.  33C9          XOR ECX,ECX
004CD50C  |.  BA 00D64C00   MOV EDX,CDEdit.004CD600              ;  code
004CD511  |.  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
004CD514  |.  E8 3784FAFF   CALL CDEdit.00475950
004CD519  |.  C603 00       MOV BYTE PTR DS:[EBX],0
004CD51C  |.  8D43 04       LEA EAX,DWORD PTR DS:[EBX+4]
004CD51F  |.  E8 5876F3FF   CALL CDEdit.00404B7C
004CD524  |.  8D43 08       LEA EAX,DWORD PTR DS:[EBX+8]
004CD527  |.  E8 5076F3FF   CALL CDEdit.00404B7C
004CD52C  |>  8B45 F8       MOV EAX,DWORD PTR SS:[EBP-8]
004CD52F  |.  BA 10D64C00   MOV EDX,CDEdit.004CD610              ;  frenzy [c4a]
004CD534  |.  E8 3F7AF3FF   CALL CDEdit.00404F78
004CD539  |.  74 1E         JE SHORT CDEdit.004CD559
004CD53B  |.  8B45 F8       MOV EAX,DWORD PTR SS:[EBP-8]
004CD53E  |.  BA 28D64C00   MOV EDX,CDEdit.004CD628              ;  spider]pc98
004CD543  |.  E8 307AF3FF   CALL CDEdit.00404F78
004CD548  |.  74 0F         JE SHORT CDEdit.004CD559
004CD54A  |.  8B45 F8       MOV EAX,DWORD PTR SS:[EBP-8]
004CD54D  |.  BA 3CD64C00   MOV EDX,CDEdit.004CD63C              ;  the doctor
004CD552  |.  E8 217AF3FF   CALL CDEdit.00404F78
004CD557  |.  75 1D         JNZ SHORT CDEdit.004CD576
004CD559  |>  B8 50D64C00   MOV EAX,CDEdit.004CD650              ;  instead of using a crack code, register your copy 

of cdedit!
004CD55E  |.  E8 2198F7FF   CALL CDEdit.00446D84
004CD563  |.  C603 00       MOV BYTE PTR DS:[EBX],0
004CD566  |.  8D43 04       LEA EAX,DWORD PTR DS:[EBX+4]
004CD569  |.  E8 0E76F3FF   CALL CDEdit.00404B7C
004CD56E  |.  8D43 08       LEA EAX,DWORD PTR DS:[EBX+8]
004CD571  |.  E8 0676F3FF   CALL CDEdit.00404B7C
004CD576  |>  33C0          XOR EAX,EAX
004CD578  |.  5A            POP EDX
004CD579  |.  59            POP ECX
004CD57A  |.  59            POP ECX
004CD57B  |.  64:8910       MOV DWORD PTR FS:[EAX],EDX
004CD57E  |.  68 93D54C00   PUSH CDEdit.004CD593
004CD583  |>  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
004CD586  |.  E8 6168F3FF   CALL CDEdit.00403DEC
004CD58B  \\.  C3            RETN
004CD58C   .^ E9 EF6FF3FF   JMP CDEdit.00404580
004CD591   .^ EB F0         JMP SHORT CDEdit.004CD583
004CD593   .  33C0          XOR EAX,EAX
004CD595   .  5A            POP EDX
004CD596   .  59            POP ECX
004CD597   .  59            POP ECX
004CD598   .  64:8910       MOV DWORD PTR FS:[EAX],EDX
004CD59B   .  68 B5D54C00   PUSH CDEdit.004CD5B5
004CD5A0   >  8D45 F0       LEA EAX,DWORD PTR SS:[EBP-10]
004CD5A3   .  BA 03000000   MOV EDX,3
004CD5A8   .  E8 F375F3FF   CALL CDEdit.00404BA0
004CD5AD   .  C3            RETN
004CD5AE   .^ E9 CD6FF3FF   JMP CDEdit.00404580
004CD5B3   .^ EB EB         JMP SHORT CDEdit.004CD5A0
004CD5B5   .  5B            POP EBX
004CD5B6   .  8BE5          MOV ESP,EBP
004CD5B8   .  5D            POP EBP
004CD5B9   .  C3            RETN
```



# 破解总结

明码比较，算法有点绕口

---

# 算法总结

1. 用户名不能小于 `9`，否则重复用户名到大于等于 `9` 为止
2. 用变形后的用户名长度与循环次数做一系列运算，加上循环对应的用户名 `ASC` 值
3. 将累加值各个位上的数字相加，若相加的值大于 `9`，则继续将各个位上的数字相加，直到小于等于 `9`
4. 将所得的数字相连即为注册码

---

# 算法注册机

```vb
Private Sub Command1_Click()
Dim Name As String
Dim i, a, b, c, d, n
If Len(Text1.Text) = 0 Then
   Text2.Text = "请输入用户名！"
Else
   If (Len(Text1.Text) = 1) Then
   For i = 1 To 18
   Name = Name & (Text1.Text)
   Next
   Else
        If (Len(Text1.Text) = 2) Then
        For i = 1 To 9
        Name = Name & (Text1.Text)
        Next
        Else
            If (Len(Text1.Text) = 3 Or Len(Text1.Text) = 4) Then
            For i = 1 To 4
            Name = Name & (Text1.Text)
            Next
            Else
                 If (Len(Text1.Text) > 4 And Len(Text1.Text) < 9) Then
                 For i = 1 To 2
                 Name = Name & (Text1.Text)
                 Next
                 Else
                     Name = Text1.Text
   End If
   End If
   End If
   End If

For i = 1 To Len(Name)
a = Len(Name) - (i - 1)
a = a * 9
a = a / 3
b = Asc(Mid(Name, i, 1)) + a
c = 0
      For n = 1 To Len(b)
      c = c + Mid(b, n, 1)
      Next
      b = c
      If b < 10 Then
      d = d & StrConv(b, 1)
      Else
tianxj:
      b = c
      c = 0
      For n = 1 To Len(StrConv(b, 1))
      c = c + Mid(StrConv(b, 1), n, 1)
      Next
      If c < 10 Then
      d = d & StrConv(c, 1)
      Else
      GoTo tianxj
    End If
   End If
 Next
      Text2.Text = d
    End If
End Sub
```

---

# 内存注册机

- 中断地址：`004CE91C`
- 中断次数：`1`
- 第一字节：`E8`
- 指令长度：`5`
- 内存方式：`-寄存器-EAX`

---

# 注册信息

- 用户名：`abcdef`
- 注册码：`753186753186`

---

感谢飘云老大、猫老大、Nisy 老大以及很多前辈们的学习教程以及所有帮助过我的论坛兄弟姐妹们！谢谢

---

# 版权声明

破文是学习的手记，兴趣是成功的源泉；本破文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!

今天是端午节，祝兄弟姐妹们平平安安、顺顺利利！
