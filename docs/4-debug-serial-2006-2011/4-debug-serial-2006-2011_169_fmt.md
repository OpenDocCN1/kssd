
# 汉之源美容美发管理系统算法分析

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-336.htm`](https://www.pediy.com/kssd/pediy08/pediy8-336.htm)

- **【破文标题】** 汉之源美容美发管理系统算法分析
- **【破文作者】** 学习破解
- **【作者邮箱】** `wxh9833@163.com`
- **【作者主页】** `wxh9833`
- **【破解工具】** `PEiD`, `OD`
- **【破解平台】** `Windows 2K & XP`
- **【软件大小】** `5.71M`
- **【原版下载】** [`http://www.newhua.com/soft/40296.htm`](http://www.newhua.com/soft/40296.htm)
- **【保护方式】** 无壳，注册码
- **【软件简介】**
- **【破解声明】**

## 破解过程

1. 安装软件，并用 `PEiD` 查壳，无壳。
2. 运行软件有“注册”失败提示。`OD` 载入，打到以下注册提示开始的地方。

```
00622844   /.   55            PUSH EBP                       ;  注册程序的入口
00622845   |.   8BEC          MOV EBP,ESP
00622847   |.   33C9          XOR ECX,ECX
00622849   |.   51            PUSH ECX
0062284A   |.   51            PUSH ECX
0062284B   |.   51            PUSH ECX
0062284C   |.   51            PUSH ECX
0062284D   |.   51            PUSH ECX
0062284E   |.   51            PUSH ECX
0062284F   |.   51            PUSH ECX
00622850   |.   53            PUSH EBX
00622851   |.   8BD8          MOV EBX,EAX
00622853   |.   33C0          XOR EAX,EAX
00622855   |.   55            PUSH EBP
00622856   |.   68 84296200   PUSH MeiRong.00622984
0062285B   |.   64:FF30       PUSH DWORD PTR FS:[EAX]
0062285E   |.   64:8920       MOV DWORD PTR FS:[EAX],ESP
00622861   |.   8D55 F4       LEA EDX,DWORD PTR SS:[EBP-C]
00622864   |.   8B83 1C030000 MOV EAX,DWORD PTR DS:[EBX+31C]
0062286A   |.   E8 B597E5FF   CALL MeiRong.0047C024
0062286F   |.   8B45 F4       MOV EAX,DWORD PTR SS:[EBP-C]                ;  试练码送 EAX 中.
00622872   |.   8D55 F8       LEA EDX,DWORD PTR SS:[EBP-8]
00622875   |.   E8 0E70DEFF   CALL MeiRong.00409888
0062287A   |.   8B45 F8       MOV EAX,DWORD PTR SS:[EBP-8]                ;  试练码送 EAX 中.
0062287D   |.   50            PUSH EAX                                     ;  试练码入栈
0062287E   |.   8D55 EC       LEA EDX,DWORD PTR SS:[EBP-14]
00622881   |.   8B83 18030000 MOV EAX,DWORD PTR DS:[EBX+318]             ;  试练码送 EAX 中.
00622887   |.   E8 9897E5FF   CALL MeiRong.0047C024
0062288C   |.   8B45 EC       MOV EAX,DWORD PTR SS:[EBP-14]              ;  出现一组字符,是预设的字符!!!!!
0062288F   |.   8D55 F0       LEA EDX,DWORD PTR SS:[EBP-10]
00622892   |.   E8 F16FDEFF   CALL MeiRong.00409888                      ;  TOPTHINK  这个就是预设字符,准备用到的.
00622897   |.   8B45 F0       MOV EAX,DWORD PTR SS:[EBP-10]              ;  把出现的字符串送 EAX 中.
0062289A   |.   50            PUSH EAX
0062289B   |.   8D55 E4       LEA EDX,DWORD PTR SS:[EBP-1C]
0062289E   |.   8B83 14030000 MOV EAX,DWORD PTR DS:[EBX+314]
006228A4   |.   E8 7B97E5FF   CALL MeiRong.0047C024
006228A9   |.   8B45 E4       MOV EAX,DWORD PTR SS:[EBP-1C]
006228AC   |.   8D55 E8       LEA EDX,DWORD PTR SS:[EBP-18]
006228AF   |.   E8 D46FDEFF   CALL MeiRong.00409888
006228B4   |.   8B55 E8       MOV EDX,DWORD PTR SS:[EBP-18]              ;  预设字符串
006228B7   |.   8B83 2C030000 MOV EAX,DWORD PTR DS:[EBX+32C]
006228BD   |.   59            POP ECX                                    ;  预设字符放 ECX 和 EDX 中.
006228BE   |.   E8 B1EFFFFF   CALL MeiRong.00621874                      ;  这个 CALL 是关键,是算法在.要 F7 跟进.
006228C3   |.   84C0          TEST AL,AL                                 ;  这里是标志位测试.
006228C5      75 2C           JNZ SHORT MeiRong.006228F3                ;  这里是关键跳转,爆破点！！！
006228C7   |.   8D45 FC       LEA EAX,DWORD PTR SS:[EBP-4]
006228CA   |.   BA 98296200   MOV EDX,MeiRong.00622998
006228CF   |.   E8 4423DEFF   CALL MeiRong.00404C18
006228D4   |.   6A 40         PUSH 40
006228D6   |.   8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
006228D9   |.   E8 6227DEFF   CALL MeiRong.00405040
006228DE   |.   8BD0          MOV EDX,EAX
006228E0   |.   B9 B4296200   MOV ECX,MeiRong.006229B4
006228E5   |.   A1 F8646900   MOV EAX,DWORD PTR DS:[6964F8]
006228EA   |.   8B00          MOV EAX,DWORD PTR DS:[EAX]
006228EC   |.   E8 63A8E7FF   CALL MeiRong.0049D154                      ;  这里是报错误,注册失败!!
006228F1   |.   EB 4E         JMP SHORT MeiRong.00622941
006228F3   |>   68 C8296200   PUSH MeiRong.006229C8
006228F8   |.   8B83 2C030000 MOV EAX,DWORD PTR DS:[EBX+32C]
006228FE   |.   FF70 5C       PUSH DWORD PTR DS:[EAX+5C]
00622901   |.   68 F4296200   PUSH MeiRong.006229F4
00622906   |.   68 002A6200   PUSH MeiRong.00622A00
0062290B   |.   8D45 FC       LEA EAX,DWORD PTR SS:[EBP-4]
0062290E   |.   BA 04000000   MOV EDX,4
00622913   |.   E8 E825DEFF   CALL MeiRong.00404F00
00622918   |.   6A 40         PUSH 40
0062291A   |.   8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
0062291D   |.   E8 1E27DEFF   CALL MeiRong.00405040
00622922   |.   8BD0          MOV EDX,EAX
00622924   |.   B9 342A6200   MOV ECX,MeiRong.00622A34
00622929   |.   A1 F8646900   MOV EAX,DWORD PTR DS:[6964F8]
0062292E   |.   8B00          MOV EAX,DWORD PTR DS:[EAX]
00622930   |.   E8 1FA8E7FF   CALL MeiRong.0049D154                      ;  这里是注册成功的提示信息。
00622935   |.   A1 F8646900   MOV EAX,DWORD PTR DS:[6964F8]
0062293A   |.   8B00          MOV EAX,DWORD PTR DS:[EAX]
0062293C   |.   E8 6FA7E7FF   CALL MeiRong.0049D0B0
00622941   |>   33C0          XOR EAX,EAX
00622943   |.   5A            POP EDX
00622944   |.   59            POP ECX
00622945   |.   59            POP ECX
00622946   |.   64:8910       MOV DWORD PTR FS:[EAX],EDX
00622949   |.   68 8B296200   PUSH MeiRong.0062298B
0062294E   |>   8D45 E4       LEA EAX,DWORD PTR SS:[EBP-1C]
00622951   |.   E8 2A22DEFF   CALL MeiRong.00404B80
00622956   |.   8D45 E8       LEA EAX,DWORD PTR SS:[EBP-18]
00622959   |.   E8 2222DEFF   CALL MeiRong.00404B80
0062295E   |.   8D45 EC       LEA EAX,DWORD PTR SS:[EBP-14]
00622961   |.   E8 1A22DEFF   CALL MeiRong.00404B80
00622966   |.   8D45 F0       LEA EAX,DWORD PTR SS:[EBP-10]
00622969   |>   E8 1222DEFF   CALL MeiRong.00404B80
0062296E   |.   8D45 F4       LEA EAX,DWORD PTR SS:[EBP-C]
00622971   |.   E8 0A22DEFF   CALL MeiRong.00404B80
00622976   |.   8D45 F8       LEA EAX,DWORD PTR SS:[EBP-8]
00622979   |.   BA 02000000   MOV EDX,2
0062297E   |.   E8 2122DEFF   CALL MeiRong.00404BA4
00622983   \\.   C3            RETN
00622984    .^ E9 1B1BDEFF   JMP MeiRong.004044A4
00622989    .^ EB C3          JMP SHORT MeiRong.0062294E
0062298B    .  5B            POP EBX
0062298C    .  8BE5          MOV ESP,EBP
0062298E    .  5D            POP EBP
0062298F    .  C3            RETN
```

第一次 `CALL` 算法。跟进。

```
00621874   /$   55            PUSH EBP                       ;  往下走。！！！！
00621875   |.   8BEC          MOV EBP,ESP
00621877   |.   83C4 F0       ADD ESP,-10
0062187A   |.   53            PUSH EBX
0062187B   |.   33DB          XOR EBX,EBX
0062187D   |.   895D F0       MOV DWORD PTR SS:[EBP-10],EBX
00621880   |.   895D F4       MOV DWORD PTR SS:[EBP-C],EBX
00621883   |.   894D F8       MOV DWORD PTR SS:[EBP-8],ECX
00621886   |.   8955 FC       MOV DWORD PTR SS:[EBP-4],EDX
00621889   |.   8BD8          MOV EBX,EAX
0062188B   |.   8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
0062188E   |.   E8 9D37DEFF   CALL MeiRong.00405030
00621893   |.   8B45 F8       MOV EAX,DWORD PTR SS:[EBP-8]    ;  预设字符
00621896   |.   E8 9537DEFF   CALL MeiRong.00405030
0062189B   |.   8B45 08       MOV EAX,DWORD PTR SS:[EBP+8]    ;  我们输入的试练码！！！如果是小写字符转为大写。
0062189E   |.   E8 8D37DEFF   CALL MeiRong.00405030
006218A3   |.   33C0          XOR EAX,EAX
006218A5   |.   55            PUSH EBP
006218A6   |.   68 5E196200   PUSH MeiRong.0062195E
006218AB   |.   64:FF30       PUSH DWORD PTR FS:[EAX]
006218AE   |.   64:8920       MOV DWORD PTR FS:[EAX],ESP
006218B1   |.   8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
006218B4   |.   E8 8735DEFF   CALL MeiRong.00404E40
```



## 006218B9 汇编代码段

```
006218B9  |.  3B43 4C       CMP EAX,DWORD PTR DS:[EBX+4C]  

006218BC  |.  7F 19         JG SHORT MeiRong.006218D7  

006218BE  |.  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]  

006218C1  |.  E8 7A35DEFF   CALL MeiRong.00404E40  

006218C6  |.  3B43 50       CMP EAX,DWORD PTR DS:[EBX+50]  

006218C9  |.  7C 0C         JL SHORT MeiRong.006218D7  

006218CB  |.  8B45 08       MOV EAX,DWORD PTR SS:[EBP+8]          ;  试练码！！！  

006218CE  |.  E8 6D35DEFF   CALL MeiRong.00404E40  

006218D3  |.  85C0          TEST EAX,EAX  

006218D5  |.  75 04         JNZ SHORT MeiRong.006218DB  

006218D7  |>  33DB          XOR EBX,EBX  

006218D9  |.  EB 60         JMP SHORT MeiRong.0062193B  

006218DB  |>  8D55 F4       LEA EDX,DWORD PTR SS:[EBP-C]  

006218DE  |.  8B45 08       MOV EAX,DWORD PTR SS:[EBP+8]  

006218E1  |.  E8 327DDEFF   CALL MeiRong.00409618  

006218E6  |.  8B55 F4       MOV EDX,DWORD PTR SS:[EBP-C]          ;  转为大写后的试练码！！！无用不用管他。  

006218E9  |.  8D45 08       LEA EAX,DWORD PTR SS:[EBP+8]  

006218EC  |.  E8 2733DEFF   CALL MeiRong.00404C18  

006218F1  |.  8D4D F0       LEA ECX,DWORD PTR SS:[EBP-10]  

006218F4  |.  8B55 FC       MOV EDX,DWORD PTR SS:[EBP-4]  

006218F7  |.  8BC3          MOV EAX,EBX  

006218F9  |.  E8 46FBFFFF   CALL MeiRong.00621444                ;  又是一个关键 CALL 跟进！！！  

006218FE  |.  8B45 F0       MOV EAX,DWORD PTR SS:[EBP-10]        ;  这里就出现真正的注册码！！！！那上面的 CALL 是关键  

00621901  |.  8B55 08       MOV EDX,DWORD PTR SS:[EBP+8]  

00621904  |.  E8 877DDEFF   CALL MeiRong.00409690  

00621909  |.  85C0          TEST EAX,EAX  

0062190B  |.  74 04         JE SHORT MeiRong.00621911  

0062190D  |.  33DB          XOR EBX,EBX  

0062190F  |.  EB 2A         JMP SHORT MeiRong.0062193B  

00621911  |>  8D43 48       LEA EAX,DWORD PTR DS:[EBX+48]  

00621914  |.  8B55 FC       MOV EDX,DWORD PTR SS:[EBP-4]  

00621917  |.  E8 B832DEFF   CALL MeiRong.00404BD4  

0062191C  |.  8D43 54       LEA EAX,DWORD PTR DS:[EBX+54]  

0062191F  |.  8B55 F8       MOV EDX,DWORD PTR SS:[EBP-8]  

00621922  |.  E8 AD32DEFF   CALL MeiRong.00404BD4  

00621927  |.  8D43 5C       LEA EAX,DWORD PTR DS:[EBX+5C]  

0062192A  |.  8B55 08       MOV EDX,DWORD PTR SS:[EBP+8]  

0062192D  |.  E8 A232DEFF   CALL MeiRong.00404BD4  

00621932  |.  8BC3          MOV EAX,EBX  

00621934  |.  E8 5B020000   CALL MeiRong.00621B94  

00621939  |.  B3 01         MOV BL,1  

0062193B  |>  33C0          XOR EAX,EAX  

0062193D  |.  5A            POP EDX  

0062193E  |.  59            POP ECX  

0062193F  |.  59            POP ECX  

00621940  |.  64:8910       MOV DWORD PTR FS:[EAX],EDX  

00621943  |.  68 65196200   PUSH MeiRong.00621965  

00621948  |>  8D45 F0       LEA EAX,DWORD PTR SS:[EBP-10]  

0062194B  |.  BA 04000000   MOV EDX,4  

00621950  |.  E8 4F32DEFF   CALL MeiRong.00404BA4  

00621955  |.  8D45 08       LEA EAX,DWORD PTR SS:[EBP+8]  

00621958  |.  E8 2332DEFF   CALL MeiRong.00404B80  

0062195D  \.  C3            RETN  
```

+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++  

### 第二次 CALL 算法跟进  

+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++  

```
00621444  /$  55            PUSH EBP  

00621445  |.  8BEC          MOV EBP,ESP  

00621447  |.  51            PUSH ECX  

00621448  |.  B9 04000000   MOV ECX,4  

0062144D  |>  6A 00         /PUSH 0  

0062144F  |.  6A 00         |PUSH 0  

00621451  |.  49            |DEC ECX  

00621452  |.^ 75 F9         \JNZ SHORT MeiRong.0062144D  

00621454  |.  874D FC       XCHG DWORD PTR SS:[EBP-4],ECX  

00621457  |.  53            PUSH EBX  

00621458  |.  56            PUSH ESI  

00621459  |.  57            PUSH EDI  

0062145A  |.  8BF9          MOV EDI,ECX  

0062145C  |.  8955 FC       MOV DWORD PTR SS:[EBP-4],EDX  

0062145F  |.  8BF0          MOV ESI,EAX  

00621461  |.  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]  

00621464  |.  E8 C73BDEFF   CALL MeiRong.00405030  

00621469  |.  33C0          XOR EAX,EAX  

0062146B  |.  55            PUSH EBP  

0062146C  |.  68 E4156200   PUSH MeiRong.006215E4  

00621471  |.  64:FF30       PUSH DWORD PTR FS:[EAX]  

00621474  |.  64:8920       MOV DWORD PTR FS:[EAX],ESP  

00621477  |.  8D55 DC       LEA EDX,DWORD PTR SS:[EBP-24]  

0062147A  |.  8BC6          MOV EAX,ESI  

0062147C  |.  E8 070F0000   CALL MeiRong.00622388  

00621481  |.  8B45 DC       MOV EAX,DWORD PTR SS:[EBP-24]        ;  这里是取的机器码！！！！  

00621484  |.  8D55 EC       LEA EDX,DWORD PTR SS:[EBP-14]  

00621487  |.  E8 FC83DEFF   CALL MeiRong.00409888  

0062148C  |.  837D EC 00    CMP DWORD PTR SS:[EBP-14],0          ;  机器与 0 比较。  

00621490  |.  75 0D         JNZ SHORT MeiRong.0062149F  

00621492  |.  8D45 E0       LEA EAX,DWORD PTR SS:[EBP-20]  

00621495  |.  8B55 FC       MOV EDX,DWORD PTR SS:[EBP-4]  

00621498  |.  E8 7B37DEFF   CALL MeiRong.00404C18  

0062149D  |.  EB 5D         JMP SHORT MeiRong.006214FC  

0062149F  |>  8B45 EC       MOV EAX,DWORD PTR SS:[EBP-14]        ;  机器码！！！！  

006214A2  |.  E8 9939DEFF   CALL MeiRong.00404E40  

006214A7  |.  8BD8          MOV EBX,EAX  

006214A9  |.  8D45 E8       LEA EAX,DWORD PTR SS:[EBP-18]  

006214AC  |.  50            PUSH EAX  

006214AD  |.  8BCB          MOV ECX,EBX  

006214AF  |.  D1F9          SAR ECX,1  

006214B1  |.  79 03         JNS SHORT MeiRong.006214B6  

006214B3  |.  83D1 00       ADC ECX,0  

006214B6  |>  BA 01000000   MOV EDX,1  

006214BB  |.  8B45 EC       MOV EAX,DWORD PTR SS:[EBP-14]        ;  机器码送 EAX 中。  

006214BE  |.  E8 DD3BDEFF   CALL MeiRong.004050A0  

006214C3  |.  8D45 E4       LEA EAX,DWORD PTR SS:[EBP-1C]  

006214C6  |.  50            PUSH EAX  

006214C7  |.  8BC3          MOV EAX,EBX  

006214C9  |.  D1F8          SAR EAX,1  

006214CB  |.  79 03         JNS SHORT MeiRong.006214D0  

006214CD  |.  83D0 00       ADC EAX,0  

006214D0  |>  8BCB          MOV ECX,EBX  

006214D2  |.  2BC8          SUB ECX,EAX  

006214D4  |.  8BD3          MOV EDX,EBX  

006214D6  |.  D1FA          SAR EDX,1  

006214D8  |.  79 03         JNS SHORT MeiRong.006214DD  

006214DA  |.  83D2 00       ADC EDX,0  

006214DD  |>  42            INC EDX  

006214DE  |.  8B45 EC       MOV EAX,DWORD PTR SS:[EBP-14]        ;  机器码！！  

006214E1  |.  E8 BA3BDEFF   CALL MeiRong.004050A0                ;  这个 CALL 是取得机器前六位。  

006214E6  |.  FF75 E8       PUSH DWORD PTR SS:[EBP-18]           ;  机器码的前六位！！！  

006214E9  |.  FF75 FC       PUSH DWORD PTR SS:[EBP-4]            ;  预设字符串  

006214EC  |.  FF75 E4       PUSH DWORD PTR SS:[EBP-1C]          ;  机器码后六位  

006214EF  |.  8D45 E0       LEA EAX,DWORD PTR SS:[EBP-20]  

006214F2  |.  BA 03000000   MOV EDX,3  

006214F7  |.  E8 043ADEFF   CALL MeiRong.00404F00  

006214FC  |>  C745 F0 00000>MOV DWORD PTR SS:[EBP-10],0  

00621503  |.  C745 F4 00000>MOV DWORD PTR SS:[EBP-C],0  

0062150A  |.  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]  

0062150D  |.  E8 2E39DEFF   CALL MeiRong.00404E40  

00621512  |.  3B46 4C       CMP EAX,DWORD PTR DS:[ESI+4C]  

00621515  |.  7F 0D         JG SHORT MeiRong.00621524            ;  预设字符  

00621517  |.  8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]  

0062151A  |.  E8 2139DEFF   CALL MeiRong.00404E40  

0062151F  |.  3B46 50       CMP EAX,DWORD PTR DS:[ESI+50]  

00621522  |.  7D 0C         JGE SHORT MeiRong.00621530  

00621524  |>  8BC7          MOV EAX,EDI  

00621526  |.  E8 5536DEFF   CALL MeiRong.00404B80  

0062152B  |.  E9 91000000   JMP MeiRong.006215C1  

00621530  |>  8B45 E0       MOV EAX,DWORD PTR SS:[EBP-20]        ;  原来这里把机器码前六位与预设字相连再连接上机器码后六位。  

00621533  |.  E8 0839DEFF   CALL MeiRong.00404E40  

00621538  |.  8BD8          MOV EBX,EAX  

0062153A  |.  EB 37         JMP SHORT MeiRong.00621573  

0062153C  |>  8B45 F0       /MOV EAX,DWORD PTR SS:[EBP-10]       ;  这里又是一个次关键！！！大家要看好，算法。  
```



```
0062153F  |.  8B55 F4       |MOV EDX,DWORD PTR SS:[EBP-C]
00621542  |.  0346 68       |ADD EAX,DWORD PTR DS:[ESI+68]          ;  这里有一个预设的数值，075BCD15 （123456789）准备做减法运算。
00621545  |.  1356 6C       |ADC EDX,DWORD PTR DS:[ESI+6C]          ;  带位加法。
00621548  |.  52            |PUSH EDX
00621549  |.  50            |PUSH EAX
0062154A  |.  8B45 E0       |MOV EAX,DWORD PTR SS:[EBP-20]         ;  把连接后的字符串放到 EAX 中。
0062154D  |.  0FB64418 FF   |MOVZX EAX,BYTE PTR DS:[EAX+EBX-1]     ;  从后面逐位取 ASCII 的值。
00621552  |.  50            |PUSH EAX                               ;  把取得值放到 EAX 中。
00621553  |.  B8 59040000   |MOV EAX,459                             ;  一个预设数 459 也就是 1113 放到 EAX 中。
00621558  |.  5A            |POP EDX                                 ;  把每位的 ASCII 放到 EDX 中。
00621559  |.  8BCA          |MOV ECX,EDX
0062155B  |.  33D2          |XOR EDX,EDX                             ;  异或 EDX
0062155D  |.  F7F1          |DIV ECX                                 ;  除完取余！！！放入 EDX 中。
0062155F  |.  8BC2          |MOV EAX,EDX                             ;  把余数放到 EAX 中。
00621561  |.  33D2          |XOR EDX,EDX                             ;  异或 EDX！！！
00621563  |.  290424        |SUB DWORD PTR SS:[ESP],EAX             ;  用预设的 075BCD15（123456789）每次减余数。
00621566  |.  195424 04     |SBB DWORD PTR SS:[ESP+4],EDX
0062156A  |.  58            |POP EAX
0062156B  |.  5A            |POP EDX
0062156C  |.  8945 F0       |MOV DWORD PTR SS:[EBP-10],EAX          ;  把 EAX 的值放到[EBP-10],EAX 中。
0062156F  |.  8955 F4       |MOV DWORD PTR SS:[EBP-C],EDX
00621572  |.  4B            |DEC EBX                                 ;  EBX 做为循环变量，减 1
00621573  |>  8B45 E0        MOV EAX,DWORD PTR SS:[EBP-20]          ;  又一次把连接后的机器码字符放到 EAX 中。
00621576  |.  E8 C538DEFF   |CALL MeiRong.00404E40
0062157B  |.  3BD8          |CMP EBX,EAX                             ;  比较循环变量与连接后机器码的长度
0062157D  |.  7F 04         |JG SHORT MeiRong.00621583
0062157F  |.  85DB         |TEST EBX,EBX
00621581  |.^ 7F B9         \JG SHORT MeiRong.0062153C              ;  跳回重新计算。通过以上的计算，然后转换为十六进制。前面加四个 0 就是注册码！！！
00621583      8B5E 60       MOV EBX,DWORD PTR DS:[ESI+60]
00621586  |.  85DB         TEST EBX,EBX
00621588  |.  7F 11         JG SHORT MeiRong.0062159B
0062158A  |.  FF75 F4       PUSH DWORD PTR SS:[EBP-C]              ;  /Arg2
0062158D  |.  FF75 F0       PUSH DWORD PTR SS:[EBP-10]             ;  |Arg1
00621590  |.  8BD7          MOV EDX,EDI                             ;  |
00621592  |.  33C0          XOR EAX,EAX                             ;  |
00621594  |.  E8 B788DEFF   CALL MeiRong.00409E50                  ;  \MeiRong.00409E50
00621599  |.  EB 26         JMP SHORT MeiRong.006215C1            ；这里就出现真正的注册码了！！！
0062159B  |>  FF75 F4       PUSH DWORD PTR SS:[EBP-C]              ;  /Arg2
0062159E  |.  FF75 F0       PUSH DWORD PTR SS:[EBP-10]             ;  |Arg1
006215A1  |.  8BD7          MOV EDX,EDI                             ;  |
006215A3  |.  8BC3          MOV EAX,EBX                             ;  |
006215A5      E8 A688DEFF   CALL MeiRong.00409E50
006215AA  |.  8B07          MOV EAX,DWORD PTR DS:[EDI]
006215AC  |.  E8 8F38DEFF   CALL MeiRong.00404E40
006215B1  |.  8BC8          MOV ECX,EAX
006215B3  |.  2B4E 60       SUB ECX,DWORD PTR DS:[ESI+60]
006215B6  |.  8B56 60       MOV EDX,DWORD PTR DS:[ESI+60]
006215B9  |.  42            INC EDX
006215BA  |.  8BC7          MOV EAX,EDI
006215BC  |.  E8 1F3BDEFF   CALL MeiRong.004050E0
006215C1  |>  33C0          XOR EAX,EAX
006215C3  |.  5A            POP EDX
006215C4  |.  59            POP ECX
006215C5  |.  59            POP ECX
006215C6  |.  64:8910       MOV DWORD PTR FS:[EAX],EDX
006215C9  |.  68 EB156200  PUSH MeiRong.006215EB
006215CE  |>  8D45 DC       LEA EAX,DWORD PTR SS:[EBP-24]
006215D1  |.  BA 05000000  MOV EDX,5
006215D6  |.  E8 C935DEFF   CALL MeiRong.00404BA4
006215DB  |.  8D45 FC       LEA EAX,DWORD PTR SS:[EBP-4]
006215DE  |.  E8 9D35DEFF   CALL MeiRong.00404B80
006215E3  \.  C3            RETN
```

以上结束后就出现明码的注册码！！！

### 破解总结

1. 这个是取的机器码的前六位与一个固定的再连是机器码后六位。
2. 先把一个固定的数放到 `EAX` 中，准备累减 `075BCD15` (123456789)
3. 从后面取连接后的机器码字符取 `ASCII` 值
4. 把一个固定数 `459` (十六) `1113` 存上，除每一位的 `ASCII` 值取余数
6. 用 `EAX` 累加值减去余数。

得到的值以十六进制形式出现，呵呵，前面补上四个 `0000`，所谓补 0 其实就是十六进制前面的四个 0。

### 版权声明

技术交流，转载请注明作者！！！！！！！！！！！！！！！

`4dd466d0e1e0cc123afbe504da2462ca`
