
# `PictureToIcon` 1.914

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-134.htm`](https://www.pediy.com/kssd/pediy08/pediy8-134.htm)

【破解软件】`Picture` To `ICON` 1.914  
【下载地址】`http://www.onlinedown.net/soft/45891.htm`  
【运行环境】`Win9x/Me/NT/2000/XP/2003`  
【保护方式】`ASPack` 壳、注册码  
【作者声明】初学 `Crack`，只是感兴趣，没有其他目的。错误之处敬请诸位前辈不吝赐教  
【调试环境】`Winxp`、`OllyDBD`、`PEiD`  
【软件信息】能将图片或屏幕的一部分转化为 `ICON` 图标，调整图标大小以及从资源库中提取 `icon`。支持 `BMP`, `JPEG`, `GIF`, `CUR`, `WMF`。  
【破解过程】常在论坛学习 `Crack`，找了个简单的软件练习，增加学习的兴趣。

## 一、脱壳

`PEiD` 查壳：`ASPack` 2.001 -> `Alexey Solodovnikov`　　`OD` 载入，提示“入口点超出代码范围......”确定后，提示“代码段可能被压缩、加密....”选否。停在下面：

```
00613001  >  60                PUSHAD                                ; 入口代码
00613002    E8 72050000     CALL Pic2Ico.00613579                ; F8 到此 ESP=12FFA4
00613007    EB 4C           JMP SHORT Pic2Ico.00613055

下断：HR ESP  F9

006134F4   /75 08          JNZ SHORT Pic2Ico.006134FE         ; 断在此，F8 跳
006134F6   |B8 01000000     MOV EAX,1
006134FB   |C2 0C00         RETN 0C
006134FE   \68 CC154000     PUSH Pic2Ico.004015CC              ; 跳到这儿
00613503    C3              RETN                               ; F8 返回到 004015CC
004015CC   /EB 10          JMP SHORT Pic2Ico.004015DE        ; 到达 OEP
004015CE   |66:623A        BOUND DI,DWORD PTR DS:\[EDX\]
004015D1   |43            INC EBX
004015D2   |2B2B          SUB EBP,DWORD PTR DS:\[EBX\]
004015D4   |48            DEC EAX
004015D5   |4F            DEC EDI
004015D6   |4F            DEC EDI
004015D7   |4B            DEC EBX
```

在 `004015CC` 处 `OllyDump` 方式 1 脱壳，保存文件后，运行正常。`PEiD` 再查: `Borland C++ 1999`

## 二、追码

`OD` 载入脱壳后文件，查找字串参考，找到：`register successfully!\\nthank you`，双击来到:`00422BED` 处，向上翻很长一段代码，在

`004228E8` 处下断，`F9`　在注册框里填用户名：`wzwgp`　注册码：`127A31682B`　点“`OK`”

```
004228E8  /.  55            PUSH EBP                           ;  断下
004228E9  |.  8BEC          MOV EBP,ESP
004228EB  |.  83C4 9C       ADD ESP,-64
    ------------中间省略------------------
00422926  |.  8B4D A4       MOV ECX,DWORD PTR SS:\[EBP-5C\]
00422929  |.  8B81 00030000 MOV EAX,DWORD PTR DS:\[ECX+300\]
0042292F  |.  E8 E4FE0900   CALL zt.004C2818
00422934  |.  8D45 FC       LEA EAX,DWORD PTR SS:\[EBP-4\]      ;  用户名地址指针入 EAX
00422937  |.  E8 6032FEFF   CALL zt.00405B9C                   ;  取得用户名位数
0042293C  |.  83F8 03       CMP EAX,3                           ;  用户名位数与 3 比较大小
0042293F  |.  0F9CC2        SETL DL
00422942  |.  83E2 01       AND EDX,1
00422945  |.  52            PUSH EDX                            
00422946  |.  FF4D C4       DEC DWORD PTR SS:\[EBP-3C\]        
00422949  |.  8D45 FC       LEA EAX,DWORD PTR SS:\[EBP-4\]      
0042294C  |.  BA 02000000   MOV EDX,2                           
00422951  |.  E8 1E0F0D00   CALL zt.004F3874                   
00422956  |.  59            POP ECX
00422957  |.  84C9          TEST CL,CL
00422959  |.  74 3C         JE SHORT zt.00422997               ;  用户名位数大于 3 位跳
0042295B  |.  66:C745 B8 14>MOV WORD PTR SS:\[EBP-48\],14
00422961  |.  BA 6D275100   MOV EDX,zt.0051276D                ;  please input your full name!
00422997  |>  \68 F4010000   PUSH 1F4                          ;/跳到此
0042299C  |.  E8 B7A40E00   CALL <JMP.&kernel32.Sleep>        ;\Sleep
004229A1  |.  66:C745 B8 20>MOV WORD PTR SS:\[EBP-48\],20
004229A7  |.  8D45 F4       LEA EAX,DWORD PTR SS:\[EBP-C\]
004229AA  |.  E8 CD11FEFF   CALL zt.00403B7C
004229AF  |.  8BD0          MOV EDX,EAX
004229B1  |.  FF45 C4       INC DWORD PTR SS:\[EBP-3C\]
004229B4  |.  8B4D A4       MOV ECX,DWORD PTR SS:\[EBP-5C\]
004229B7  |.  8B81 04030000 MOV EAX,DWORD PTR DS:\[ECX+304\]
004229BD  |.  E8 56FE0900   CALL zt.004C2818                   ;  取假码
004229C2  |.  8D55 F4       LEA EDX,DWORD PTR SS:\[EBP-C\]      ;  假码地址指针入 EDX
004229C5  |.  FF32          PUSH DWORD PTR DS:\[EDX\]           ;  Arg1 = 00BA3684 (ASCII "127A31682B")
004229C7  |.  E8 24E7FFFF   CALL zt.004210F0                   ;  F7 进入算法
```

`004229C7` 处 `F7` 来到下面：

```
004210F0  /$  55            PUSH EBP
004210F1  |.  8BEC          MOV EBP,ESP
004210F3  |.  81C4 74FFFFFF ADD ESP,-8C
004210F9  |.  56            PUSH ESI
004210FA  |.  57            PUSH EDI
004210FB  |.  B8 C0285100   MOV EAX,zt.005128C0
00421100  |.  E8 7B760C00   CALL zt.004E8780
00421105  |.  C745 F8 01000>MOV DWORD PTR SS:\[EBP-8\],1
0042110C  |.  8D55 08       LEA EDX,DWORD PTR SS:\[EBP+8\]
0042110F  |.  8D45 08       LEA EAX,DWORD PTR SS:\[EBP+8\]
00421112  |.  E8 DD260D00   CALL zt.004F37F4
00421117  |.  FF45 F8       INC DWORD PTR SS:\[EBP-8\]
0042111A  |.  66:C745 EC 08>MOV WORD PTR SS:\[EBP-14\],8
00421120  |.  C645 DB 00    MOV BYTE PTR SS:\[EBP-25\],0
00421124  |.  8D45 08       LEA EAX,DWORD PTR SS:\[EBP+8\]
00421127  |.  E8 704AFEFF   CALL zt.00405B9C                   ;  取假码位数
0042112C  |.  83F8 2C       CMP EAX,2C                          ;  注册码要 44 位，填入 44 位注册码重新来到此
0042112F      0F85 3E020000 JNZ zt.00421373
00421135  |.  BE D6245100   MOV ESI,zt.005124D6               ;  常数组入 ESI (1z1h+2a0n-0g8y\*9a1n|)
0042113A  |.  8D7D 88       LEA EDI,DWORD PTR SS:\[EBP-78\]
0042113D  |.  B9 05000000   MOV ECX,5
00421142  |.  F3:A5         REP MOVS DWORD PTR ES:\[EDI\],DWOR>; 常数入栈（12FA30-40）   
00421144  |.  A4            MOVS BYTE PTR ES:\[EDI\],BYTE PTR >
00421145  |.  8D45 08       LEA EAX,DWORD PTR SS:\[EBP+8\]     ; 假码地址入 EAX
00421148  |.  E8 2B10FEFF   CALL zt.00402178                   ; 取假码
0042114D  |.  0FBE50 28     MOVSX EDX,BYTE PTR DS:\[EAX+28\]   ; 假码 41 位入 EDX
00421151  |.  83FA 50       CMP EDX,50                          ; 第 41 位注册码(50)
00421154      74 23        JE SHORT zt.00421179
00421156  |.  33C0          XOR EAX,EAX
00421158  |.  50            PUSH EAX
00421159  |.  FF4D F8       DEC DWORD PTR SS:\[EBP-8\]
0042115C  |.  8D45 08       LEA EAX,DWORD PTR SS:\[EBP+8\]
0042115F  |.  BA 02000000   MOV EDX,2
00421164  |.  E8 0B270D00   CALL zt.004F3874
00421169  |.  58            POP EAX
0042116A  |.  8B55 DC       MOV EDX,DWORD PTR SS:\[EBP-24\]
0042116D  |.  64:8915 00000>MOV DWORD PTR FS:\[0\],EDX
00421174      E9 19020000  JMP zt.00421392
00421179  |>  8D45 08       LEA EAX,DWORD PTR SS:\[EBP+8\]
0042117C  |.  E8 F70FFEFF   CALL zt.00402178
00421181  |.  0FBE50 29     MOVSX EDX,BYTE PTR DS:\[EAX+29\]
00421185  |.  83FA 32       CMP EDX,32                          ; 第 42 位注册码(32)
00421188  |.  74 23         JE SHORT zt.004211AD
0042118A  |.  33C0          XOR EAX,EAX
0042118C  |.  50            PUSH EAX
0042118D  |.  FF4D F8       DEC DWORD PTR SS:\[EBP-8\]
00421190  |.  8D45 08       LEA EAX,DWORD PTR SS:\[EBP+8\]
00421193  |.  BA 02000000   MOV EDX,2
00421198  |.  E8 D7260D00   CALL zt.004F3874
0042119D  |.  58            POP EAX
0042119E  |.  8B55 DC       MOV EDX,DWORD PTR SS:\[EBP-24\]
004211A1  |.  64:8915 00000>MOV DWORD PTR FS:\[0\],EDX
004211A8  |.  E9 E5010000  JMP zt.00421392
004211AD  |>  8D45 08       LEA EAX,DWORD PTR SS:\[EBP+8\]
004211B0  |.  E8 C30FFEFF   CALL zt.00402178
004211B5  |.  0FBE50 2A     MOVSX EDX,BYTE PTR DS:\[EAX+2A\]
004211B9  |.  83FA 49       CMP EDX,49                          ; 第 43 位注册码(49)
004211BC  |.  74 23         JE SHORT zt.004211E1
004211BE  |.  33C0          XOR EAX,EAX
004211C0  |.  50            PUSH EAX
004211C1  |.  FF4D F8       DEC DWORD PTR SS:\[EBP-8\]
004211C4  |.  8D45 08       LEA EAX,DWORD PTR SS:\[EBP+8\]
004211C7  |.  BA 02000000   MOV EDX,2
004211CC  |.  E8 A3260D00   CALL zt.004F3874
004211D1  |.  58            POP EAX
004211D2  |.  8B55 DC       MOV EDX,DWORD PTR SS:\[EBP-24\]
004211D5  |.  64:8915 00000>MOV DWORD PTR FS:\[0\],EDX
004211DC  |.  E9 B1010000  JMP zt.00421392
004211E1  |>  8D45 08       LEA EAX,DWORD PTR SS:\[EBP+8\]
004211E4  |.  E8 8F0FFEFF   CALL zt.00402178
004211E9  |.  0FBE50 2B     MOVSX EDX,BYTE PTR DS:\[EAX+2B\]
004211ED  |.  83FA 31       CMP EDX,31                          ;  第 44 位注册码(31)
```



## 汇编代码分析

```
004211F0  |.  74 23         JE SHORT zt.00421215               ; 相等就跳  
00421215  |> \8D45 08       LEA EAX,DWORD PTR SS:[EBP+8]       ; 跳到此  
00421218  |.  E8 5B0FFEFF   CALL zt.00402178  
0042121D  |.  50            PUSH EAX                            
0042121E  |.  8D55 A0       LEA EDX,DWORD PTR SS:[EBP-60]      
00421221  |.  52            PUSH EDX                            
00421222  |.  E8 B5720C00   CALL zt.004E84DC                    
00421227  |.  83C4 08       ADD ESP,8  
0042122A  |.  0FBE4D A1     MOVSX ECX,BYTE PTR SS:[EBP-5F]     ; 取假码第 2 位  
0042122E  |.  83F9 30       CMP ECX,30                          ; 第 2 位注册码(30)  
00421231      0F85 3C010000 JNZ zt.00421373                    
00421237  |.  C645 A1 23    MOV BYTE PTR SS:[EBP-5F],23        ; 常数 32 替换第 2 位注册码  
0042123B  |.  C645 DB 01    MOV BYTE PTR SS:[EBP-25],1  
0042123F  |.  C745 D4 02000>MOV DWORD PTR SS:[EBP-2C],2  
```

进入第一个循环

```
00421246  |> /8B45 D4       /MOV EAX,DWORD PTR SS:[EBP-2C]  
00421249  |. |0FBE5405 88   |MOVSX EDX,BYTE PTR SS:[EBP+EAX-78]     ;  edx=31(常数组第三位)(逐位取常数组)  
0042124E  |. |8B4D D4       |MOV ECX,DWORD PTR SS:[EBP-2C]         ;  [ebp-2c]=2  
00421251  |. |0FBE440D 9F   |MOVSX EAX,BYTE PTR SS:[EBP+ECX-61]    ;  eax=23（替换后的假码第二位、逐位取假码）  
00421256  |. |03D0          |ADD EDX,EAX                            ;  edx=54  
00421258  |. |8B4D D4       |MOV ECX,DWORD PTR SS:[EBP-2C]  
0042125B  |. |0FBE440D A0   |MOVSX EAX,BYTE PTR SS:[EBP+ECX-60]    ;  eax=37（假码第三位、逐位取假码）  
00421260  |. |33D0          |XOR EDX,EAX                            ;  edx=63  
00421262  |. |8B4D D4       |MOV ECX,DWORD PTR SS:[EBP-2C]  
00421265  |. |0FBE440D 88   |MOVSX EAX,BYTE PTR SS:[EBP+ECX-78]    ;  eax=31  
0042126A  |. |33D0          |XOR EDX,EAX                            ;  edx=52  
0042126C  |. |52            |PUSH EDX                               
0042126D  |. |E8 26010000   |CALL zt.00421398                       
00421272  |. |59            |POP ECX  
00421273  |. |B9 1A000000   |MOV ECX,1A  
00421278  |. |99            |CDQ  
00421279  |. |F7F9          |IDIV ECX                               ;  eax=3  edx=4 (52 IDIV 1A)  
0042127B  |. |83C2 41       |ADD EDX,41                             ;  edx=4＋41＝45  
0042127E  |. |8B45 D4       |MOV EAX,DWORD PTR SS:[EBP-2C]  
00421281  |. |0FBE4C05 A9   |MOVSX ECX,BYTE PTR SS:[EBP+EAX-57]     ;  假码第 12 位入 ecx  
00421286  |. |3BD1          |CMP EDX,ECX                            ;  注册码第 12 位=45  
00421288      |75 06        JZ SHORT zt.00421290                   ;  将 JN 改成 JNZ 可以走完循环  
0042128A  |. |C645 DB 00    |MOV BYTE PTR SS:[EBP-25],0  
0042128E  |. |EB 09         |JMP SHORT zt.00421299  
00421290  |> |FF45 D4       |INC DWORD PTR SS:[EBP-2C]              ;  [ebp-2c]=2+1(计数器)  
00421293  |. |837D D4 0A    |CMP DWORD PTR SS:[EBP-2C],0A  
00421297  |.^\\7C AD         \\JL SHORT zt.00421246                ;  [ebp-2c]小于 0A 循环  
```

通过该循环计算出 44 位注册码中的 8 个数(45 41 4D 59 50 47 51 57)，第 12 位到第 19 位。

走出第一个循环来到第二个循环,将已知的注册码填到对应位置,来到第二个循环。

```
004212B6  |> /8B55 D0       /MOV EDX,DWORD PTR SS:[EBP-30]         ;  18 入 EDX　[EBP-30]计数器  
004212B9  |. |0FBE4415 89   |MOVSX EAX,BYTE PTR SS:[EBP+EDX-77]    ;  替换后的假码第二位入 EAX (逐位取假码)  
004212BE  |. |B9 06000000   |MOV ECX,6  
004212C3  |. |99            |CDQ  
004212C4  |. |F7F9          |IDIV ECX                               ;  EAX=5 EDX=5 (23 IDIV 6)  
004212C6  |. |8BCA          |MOV ECX,EDX                            ;  ECX=5  
004212C8  |. |8B45 D0       |MOV EAX,DWORD PTR SS:[EBP-30]  
004212CB  |. |0FBE5405 8A   |MOVSX EDX,BYTE PTR SS:[EBP+EAX-76]    ;  假码第 3 位入 EDX (逐位取假码)  
004212D0  |. |D3E2          |SHL EDX,CL                             ;  EDX=6E0  Cl=5 (逻辑右移)  
004212D2  |. |8B45 D0       |MOV EAX,DWORD PTR SS:[EBP-30]  
004212D5  |. |0FBE4C05 8B   |MOVSX ECX,BYTE PTR SS:[EBP+EAX-75]     ;  假码第 4 位入 ECX=41 (A)  
004212DA  |. |0BD1          |OR EDX,ECX                             ;  EDX=6E1 (6E0 OR 41)  
004212DC  |. |52            |PUSH EDX                               
004212DD  |. |E8 B6000000   |CALL zt.00421398                       ;  EDX 的值存入 EAX  
004212E2  |. |59            |POP ECX  
004212E3  |. |B9 1A000000   |MOV ECX,1A  
004212E8  |. |99            |CDQ  
004212E9  |. |F7F9          |IDIV ECX                               ;  eax=43 edx=13 (6E1 IDIV 1A)  
004212EB  |. |80C2 61       |ADD DL,61                              ;  DL=13+61=74  
004212EE  |. |8B45 D0       |MOV EAX,DWORD PTR SS:[EBP-30]  
004212F1  |. |889405 5CFFFF>|MOV BYTE PTR SS:[EBP+EAX-A4],DL        ;  保存计算结果  
004212F8  |. |FF45 D0       |INC DWORD PTR SS:[EBP-30]             ;  计数器加 1  
004212FB  |. |837D D0 28    |CMP DWORD PTR SS:[EBP-30],28          ;  计数器的数值小于 28 继续循环  
004212FF  |.^\\7C B5         \\JL SHORT zt.004212B6                ;  循环  
```

通过第二个循环计算出一组中间数，共 16 个(74 78 6C 65 75 67 75 68 70 6C 72 66 71 66 6C 62 )。

```
00421301  |> \C645 84 5A    MOV BYTE PTR SS:[EBP-7C],5A            ; 5A 加入中间数第 17 个  
00421305  |.  C645 85 59    MOV BYTE PTR SS:[EBP-7B],59            ; 59 加入中间数第 18 个  
00421309  |.  C745 CC 18000>MOV DWORD PTR SS:[EBP-34],18  
00421310  |.  66:C745 EC 08>MOV WORD PTR SS:[EBP-14],8  
00421316  |.  837D CC 28    CMP DWORD PTR SS:[EBP-34],28  
0042131A  |.  7D 4A         JGE SHORT zt.00421366  
```

走出第二个循环来到第三个循环。

```
0042131C  |> /8B55 CC       /MOV EDX,DWORD PTR SS:[EBP-34]         ;  [ebp-34]=18　计数器  
0042131F  |. |0FBE8415 5CFF>|MOVSX EAX,BYTE PTR SS:[EBP+EDX-A4]    ;  中间数第一位入 EAX＝74(逐位取数)  
00421327  |. |C1E0 04       |SHL EAX,4                              ;  逻辑左移 4 位  
0042132A  |. |8B55 CC       |MOV EDX,DWORD PTR SS:[EBP-34]  
0042132D  |. |0FBE8C15 5DFF>|MOVSX ECX,BYTE PTR SS:[EBP+EDX-A3]     ;  中间数第 2 位入 ECX=78(逐位取数)  
00421335  |. |D1F9          |SAR ECX,1                              ;  算术右移 1 位  
00421337  |. |33C1          |XOR EAX,ECX                            ;  EAX=77C  
00421339  |. |50            |PUSH EAX                               
0042133A  |. |E8 59000000   |CALL zt.00421398                       
0042133F  |. |59            |POP ECX  
00421340  |. |B9 1A000000   |MOV ECX,1A  
00421345  |. |99            |CDQ  
00421346  |. |F7F9          |IDIV ECX                               ;  EAX=49 EDX=12  
00421348  |. |83C2 41       |ADD EDX,41                             ;  EDX=12+41=53  
0042134B  |. |8B45 CC       |MOV EAX,DWORD PTR SS:[EBP-34]  
0042134E  |. |0FBE4405 A0   |MOVSX EAX,BYTE PTR SS:[EBP+EAX-60]    ;  取第 25 位假码(逐位比较注册码)  
00421353  |. |3BD0          |CMP EDX,EAX  
00421355      |74 06        JE SHORT zt.0042135D  
00421357  |. |C645 DB 00    |MOV BYTE PTR SS:[EBP-25],0  
0042135B  |. |EB 09         |JMP SHORT zt.00421366  
0042135D  |> |FF45 CC       |INC DWORD PTR SS:[EBP-34]  
00421360  |. |837D CC 28    |CMP DWORD PTR SS:[EBP-34],28  
00421364  |.^\\7C B6         \\JL SHORT zt.0042131C               ;  [ebp-34]小于 28 继续循环  
```

通过第三个循环计算出 44 位注册码中的 16 个数(53 59 4B 45 54 59 55 45 41 52 52 4D 48 4B 4A 50)，第 25 位到第 40 位。

```
00421366  |> \0FBE55 AA     MOVSX EDX,BYTE PTR SS:[EBP-56]         ;  取假码 11 位入 EDX  
0042136A  |.  83FA 59       CMP EDX,59                              ;  注册码第 11 位=59  
0042136D      74 04         JE SHORT zt.00421373  
0042136F  |.  C645 DB 00    MOV BYTE PTR SS:[EBP-25],0  
00421373  |>  8A45 DB       MOV AL,BYTE PTR SS:[EBP-25]  
00421376  |.  50            PUSH EAX  
00421377  |.  FF4D F8       DEC DWORD PTR SS:[EBP-8]  
0042137A  |.  8D45 08       LEA EAX,DWORD PTR SS:[EBP+8]            ;  取假码地址入 EAX  
0042137D  |.  BA 02000000   MOV EDX,2  
00421382  |.  E8 ED240D00   CALL zt.004F3874  
00421387  |.  58            POP EAX  
00421388  |.  8B55 DC       MOV EDX,DWORD PTR SS:[EBP-24]  
0042138B  |.  64:8915 00000>MOV DWORD PTR FS:[0],EDX  
```



```
00421392  |>  5F            POP EDI
00421393  |.  5E            POP ESI
00421394  |.  8BE5          MOV ESP,EBP
00421396  |.  5D            POP EBP
00421397  \.  C3            RETN                                     ; 返回到 004229CC
```

走完`004229C7`处`CALL zt.004210F0`返回到下面：

```
004229C7  |.  E8 24E7FFFF   CALL zt.004210F0
004229CC  |.  59            POP ECX                                  ; 返回到此
004229CD  |.  8B0D FCAE5100 MOV ECX,DWORD PTR DS:[51AEFC]
004229D3  |.  8B11          MOV EDX,DWORD PTR DS:[ECX]
004229D5  |.  8882 E4030000 MOV BYTE PTR DS:[EDX+3E4],AL

--------------中间省略------------------

00422A48  |.  E8 2BF7FDFF   CALL zt.00402178
00422A4D  |.  0FBE50 17     MOVSX EDX,BYTE PTR DS:[EAX+17]           ; 取假码第 24 位入 EDX
00422A51  |.  83FA 30       CMP EDX,30
00422A54  |.  7C 16         JL SHORT zt.00422A6C
00422A56  |.  8B45 A4       MOV EAX,DWORD PTR SS:[EBP-5C]
00422A59  |.  05 20030000   ADD EAX,320
00422A5E  |.  E8 15F7FDFF   CALL zt.00402178
00422A63  |.  0FBE50 17     MOVSX EDX,BYTE PTR DS:[EAX+17]           ; 取假码第 24 位入 EDX
00422A67  |.  83FA 39       CMP EDX,39
00422A6A      7E 0F         JLE SHORT zt.00422A7B
```

注册码第 24 位是`0-9`(`十进制`)之间的任意数。

一路往下就到了`"register successfully!\nthank you."`

```
00422BDE  |.  FF51 FC       CALL NEAR DWORD PTR DS:[ECX-4]
00422BE1  |.  66:C745 B8 5C>MOV WORD PTR SS:[EBP-48],5C
00422BE7  |>  66:C745 B8 74>MOV WORD PTR SS:[EBP-48],74
00422BED  |.  BA A8275100   MOV EDX,zt.005127A8                      ;  register successfully!\nthank you.
```

### 三、追码总结

1. 用户名要`3`位以上，与注册码无关。
2. 注册码位数是`44`位(`ASCII`字符)。
3. 注册码第`1`位、第`3`到`10`位(参与不固定注册码运算)、第`20`到`23`位可以取任意字符。
4. 注册码第`2`位、第`41`到`44`位是固定字符：`0 P 2 I 1`
5. 注册码第`24`位是`0-9`(`十进制`)之间的任意字符
6. 第`2`位〔常数`23(h)`替换〕到`10`位注册码与常数组(`1z1h+2a0n-0g8y*9a1n|`)通过循环逐位运算得到注册码第`12`到`19`位注册码。
7. 第`2`位〔常数`23(h)`替换〕到`19`位注册码通过循环逐位运算得到一组中间数`16`个，再加上固定数`5A 59`，中间数组共计有`18`个。
8. `18`个中间数通过循环逐位运算得到第`25`位到第`40`位注册码。

我的用户名：`wzwgp`

注册码：`107A31682BYEAMYPGQW56787SYKETYUEARRMHKJPP2I1`

注册信息保存在注册表：`HKEY_USERS\S-1-5-21-1123561945-492894223-1060284298-1002\Software\XTZY\Pic2Ico`

破文写得很乱，感谢你看完。

`4dd466d0e1e0cc123afbe504da2462ca`
