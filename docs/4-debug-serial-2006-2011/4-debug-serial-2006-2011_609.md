
# 精装友情通讯录算法分析

> 原文：[`www.pediy.com/kssd/pediy12/129870.html`](https://www.pediy.com/kssd/pediy12/129870.html)

## 软件信息

- **破文标题**：精装友情通讯录算法分析
- **破文作者**：wangweilll
- **作者邮箱**：309661415@qq.com
- **作者主页**：309661415@qq.com
- **破解工具**：`PEiD`, `OD`
- **破解平台**：Windows XP
- **软件名称**：精装友情通讯录
- **软件大小**：956 KB
- **软件类别**：应用软件/信息管理
- **软件语言**：中文
- **更新时间**：2008-3-6
- **原版下载**：网上找一下
- **保护方式**：注册码

〖精装友情通讯录〗软件是国产最为简洁、实用的通讯录软件，虽然精致但是功能特别强大，软件可能将令你爱不释手。软件界面精美、小巧实用、绿色精致。通讯录可选择简洁列表、名片格式及详细列表格式等三种格式显示，支持鼠标拖放操作，支持模糊查询，支持添加、排序等等，添加修改方便。软件随时升级。

`Delphi6.0` 无壳

```assembly
00515264   /.   55                   PUSH EBP
00515265   |.   8BEC                 MOV EBP,ESP
00515267   |.   B9 17000000          MOV ECX,17
0051526C   |>   6A 00                PUSH 0
0051526E   |.   6A 00                PUSH 0
00515270   |.   49                   DEC ECX
00515271   |.^  75 F9                JNZ SHORT jzyq.0051526C
00515273   |.   53                   PUSH EBX
00515274   |.   56                   PUSH ESI
00515275   |.   8BD8                 MOV EBX,EAX
00515277   |.   33C0                 XOR EAX,EAX
00515279   |.   55                   PUSH EBP
0051527A   |.   68 E9545100          PUSH jzyq.005154E9
0051527F   |.   64:FF30             PUSH DWORD PTR FS:[EAX]
00515282   |.   64:8920             MOV DWORD PTR FS:[EAX],ESP
00515285   |.   68 80000000          PUSH 80                 ; /BufSize = 80 (128.)
0051528A   |.   8D85 7BFFFFFF       LEA EAX,DWORD PTR SS:[EBP-85]      ; |
00515290   |.   50                   PUSH EAX                ; |Buffer
00515291   |.   E8 B220EFFF          CALL <JMP.&kernel32.GetSystemDirect>; \GetSystemDirectoryA
00515296   |.   8D45 FC              LEA EAX,DWORD PTR SS:[EBP-4]
00515299   |.   8D95 7BFFFFFF        LEA EDX,DWORD PTR SS:[EBP-85]
0051529F   |.   B9 81000000          MOV ECX,81
005152A4   |.   E8 DBF9EEFF         CALL jzyq.00404C84
005152A9   |.   8D95 74FFFFFF        LEA EDX,DWORD PTR SS:[EBP-8C]
005152AF   |.   8B83 1C030000        MOV EAX,DWORD PTR DS:[EBX+31C]
005152B5   |.   E8 F2D8F2FF         CALL jzyq.00442BAC                 ;  取注册码
005152BA   |.   83BD 74FFFFFF>CMP DWORD PTR SS:[EBP-8C],0
005152C1   |.   74 1A                JE SHORT jzyq.005152DD
005152C3   |.   8D95 70FFFFFF        LEA EDX,DWORD PTR SS:[EBP-90]
005152C9   |.   8B83 10030000        MOV EAX,DWORD PTR DS:[EBX+310]
005152CF   |.   E8 D8D8F2FF         CALL jzyq.00442BAC                 ;  取订单号
005152D4   |.   83BD 70FFFFFF>CMP DWORD PTR SS:[EBP-90],0
005152DB   |.   75 0F                JNZ SHORT jzyq.005152EC
005152DD   |>   B8 00555100         MOV EAX,jzyq.00515500             ;  注册信息没有填写齐全
005152E2   |.   E8 B96AF2FF         CALL jzyq.0043BDA0
005152E7   |.   E9 80010000         JMP jzyq.0051546C
005152EC   |>   8D95 6CFFFFFF       LEA EDX,DWORD PTR SS:[EBP-94]
005152F2   |.   8B83 1C030000        MOV EAX,DWORD PTR DS:[EBX+31C]
005152F8   |.   E8 AFD8F2FF         CALL jzyq.00442BAC                 ;  取注册码位数
005152FD   |.   8B85 6CFFFFFF        MOV EAX,DWORD PTR SS:[EBP-94]
00515303   |.   50                   PUSH EAX
00515304   |.   8D95 60FFFFFF        LEA EDX,DWORD PTR SS:[EBP-A0]
0051530A   |.   8B83 10030000        MOV EAX,DWORD PTR DS:[EBX+310]
00515310   |.   E8 97D8F2FF         CALL jzyq.00442BAC                 ;  取订单号位数
00515315   |.   8B85 60FFFFFF        MOV EAX,DWORD PTR SS:[EBP-A0]
0051531B   |.   E8 9042EFFF         CALL jzyq.004095B0                 ;  判断字符是不是 0-9，是就通过计算把结果给 EAX
```

`CALL jzyq.004095B0` 跟进去！！！！！！！！！！！！！！！！

```assembly
004095B0   /$   53                   PUSH EBX
004095B1   |.   56                   PUSH ESI
004095B2   |.   83C4 F4              ADD ESP,-0C
004095B5   |.   8BD8                 MOV EBX,EAX
004095B7   |.   8BD4                 MOV EDX,ESP
004095B9   |.   8BC3                 MOV EAX,EBX
004095BB   |.   E8 B89DFFFF         CALL jzyq.00403378          这个 CALL 有问题跟进
```

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~可以找这个关键算法

```assembly
004033CA   |>   80EB 30              /SUB BL,30                  ；BL 是订单号的字符
004033CD   |.  |80FB 09             |CMP BL,9
004033D0   |.  |77 25               |JA SHORT jzyq.004033F7
004033D2   |.  |39F8                |CMP EAX,EDI
004033D4   |.  |77 21               |JA SHORT jzyq.004033F7
004033D6   |.  |8D0480             |LEA EAX,DWORD PTR DS:[EAX+EAX*4]       ;  把计算结果*4+把计算结果
004033D9   |.  |01C0                |ADD EAX,EAX                   ;  把计算结果+把计算结果
004033DB   |.  |01D8                |ADD EAX,EBX                   ;  把计算结果+取出的字符
004033DD   |.  |8A1E                |MOV BL,BYTE PTR DS:[ESI]         ;  循环取出字符给 BL
004033DF   |.  |46                  |INC ESI                     ;  ESI 自 加 1
004033E0   |.  |84DB                |TEST BL,BL
004033E2   |.^ \75 E6              \JNZ SHORT jzyq.004033CA
```

****************************************************************

```assembly
00515320   |.   B9 40080000          MOV ECX,840                 ;  ECX 置 840
00515325   |.   99                   CDQ                       ;  EDX 清 0
00515326   |.   F7F9                 IDIV ECX                    ;  把 CALL 004095B0 EAX 的结果除以 ECX
00515328   |.   8BC2                 MOV EAX,EDX                 ;  把佘数给 EAX
0051532A   |.   8D95 64FFFFFF        LEA EDX,DWORD PTR SS:[EBP-9C]
00515330   |.   E8 1742EFFF         CALL jzyq.0040954C              ;  把上次计算 EAX 的结果放这个 CDLL 运算可以得到注册码的前 N 位
```

****************************************************************

`CALL jzyq.0040954C` 算法 `CALL` 跟进

`CALL 0040A24C` 算法 `CALL` 跟进，`CALL 00409DA4` 算法 `CALL` 跟进，`CALL 00409ECE` 算法 `CALL` 跟进，

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~可以看到算法了

```assembly
0040A022   $  B9 0A000000          MOV ECX,0A                   ;  ECX 置 0A     ; Case 55 ('U') of switch 00409FF6
0040A027   |>   8D75 9F            LEA ESI,DWORD PTR SS:[EBP-61]
0040A02A   |>   31D2               /XOR EDX,EDX                 ;  EDX 清 0
0040A02C   |.   F7F1               |DIV ECX                     ;  EAX 是上次运算的结果/0A
0040A02E   |.   80C2 30            |ADD DL,30                    ;  DL+30
0040A031   |.   80FA 3A            |CMP DL,3A                    ;  DL 与 3A 比较
0040A034   |.   72 03              |JB SHORT jzyq.0040A039          ;  不高于等于时转移
0040A036   |.   80C2 07            |ADD DL,7
0040A039   |>   4E                  |DEC ESI                     ;  ESI 自减 1
0040A03A   |.   8816               |MOV BYTE PTR DS:[ESI],DL         ;  把结果存入 DS：[ESI]
0040A03C   |.   09C0               |OR EAX,EAX
0040A03E   |.^  75 EA              \JNZ SHORT jzyq.0040A02A
```

****************************************************************

```assembly
00515335   |.   FFB5 64FFFFFF       PUSH DWORD PTR SS:[EBP-9C]         ;  把注册码前 N 位压入堆栈
0051533B   |.   8D95 54FFFFFF       LEA EDX,DWORD PTR SS:[EBP-AC]
00515341   |.   8B83 10030000        MOV EAX,DWORD PTR DS:[EBX+310]
00515347   |.   E8 60D8F2FF         CALL jzyq.00442BAC                 ;  在次判断字符是不是 0-9，是就通过计算把结果给 EAX
0051534C   |.   8B85 54FFFFFF        MOV EAX,DWORD PTR SS:[EBP-AC]
00515352   |.   E8 5942EFFF         CALL jzyq.004095B0                 ;  把订单号字符运算
00515357   |.   8D95 58FFFFFF        LEA EDX,DWORD PTR SS:[EBP-A8]
0051535D      E8 C6FCFFFF         CALL jzyq.00515028                 ;  把上次计算 EAX 的结果放这个 CDLL 运算可以得到一串字符
```

****************************************************************

`CALL jzyq.00515028` 跟进

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

```assembly
00515028   /$   55                   PUSH EBP
00515029   |.   8BEC                 MOV EBP,ESP
0051502B   |.   33C9                 XOR ECX,ECX
0051502D   |.   51                   PUSH ECX
0051502E   |.   51                   PUSH ECX
0051502F   |.   51                   PUSH ECX
```



## 00515030 - 00515054

```
00515030 |. 51            PUSH ECX
00515031 |. 53            PUSH EBX
00515032 |. 56            PUSH ESI
00515033 |. 8BF2          MOV ESI,EDX
00515035 |. 8BD8          MOV EBX,EAX                     ; 上次计算 EAX 的结果给我 EBX
00515037 |. 33C0          XOR EAX,EAX
00515039 |. 55            PUSH EBP
0051503A |. 68 F8505100   PUSH jzyq.005150F8
0051503F |. 64:FF30       PUSH DWORD PTR FS:[EAX]
00515042 |. 64:8920       MOV DWORD PTR FS:[EAX],ESP
00515045 |. 81F3 F1250B00 XOR EBX,0B25F1                  ; EBX 与 0B25F1 逻辑异或运算
0051504B |. 8BC3          MOV EAX,EBX                     ; 在把结果存到 EAX 中
0051504D |. 33D2          XOR EDX,EDX
0051504F |. 52            PUSH EDX
00515050 |. 50            PUSH EAX                          ; 在把 EAX 压入堆栈这个数据有用
00515051 |. 8D45 FC       LEA EAX,DWORD PTR SS:[EBP-4]
00515054 |. E8 2345EFFF   CALL jzyq.0040957C                ;  跟进
```

在跟进 `CALL 0040A24C`，在跟进 `CALL 00409DA4`，在跟进 `CALL 00409DA4`，在跟进 `CALL 00409ECE`，最后跟进可以找到这个算法。

## 00409F91 - 00409FD7

```
00409F91 |$ B9 0A000000   MOV ECX,0A                      ;  ECX 置 0A    ; Case 55 ('U') of switch 00409F5D
00409F96 |>  8D75 AF       LEA ESI,DWORD PTR SS:[EBP-51]
00409F99 |  53            /PUSH EBX
00409F9A |. 51            |PUSH ECX
00409F9B |. 6A 00         |PUSH 0
00409F9D |. 51            |PUSH ECX
00409F9E |. 8B03          |MOV EAX,DWORD PTR DS:[EBX]      ;  EBX 与 0B25F1 逻辑异或运算给 EAX
00409FA0 |. 8B53 04       |MOV EDX,DWORD PTR DS:[EBX+4]     ;  EAX=3042
00409FA3 |. 8B5D FC       |MOV EBX,DWORD PTR SS:[EBP-4]
00409FA6 |. E8 15BDFFFF   |CALL jzyq.00405CC0              ;  CALL 的内是 DIV EBX XCHG EAX,EDX XOR EDX,EDX
00409FAB |. 59            |POP ECX
00409FAC |. 5B            |POP EBX
00409FAD |. 92            |XCHG EAX,EDX
00409FAE |. 80C2 30       |ADD DL,30
00409FB1 |. 80FA 3A       |CMP DL,3A
00409FB4 |. 72 03         |JB SHORT jzyq.00409FB9
00409FB6 |. 80C2 07       |ADD DL,7
00409FB9 |  4E            |DEC ESI
00409FBA |. 8816          |MOV BYTE PTR DS:[ESI],DL
00409FBC |. 53            |PUSH EBX
00409FBD |. 51            |PUSH ECX
00409FBE |. 6A 00         |PUSH 0
00409FC0 |. 51            |PUSH ECX
00409FC1 |. 8B03          |MOV EAX,DWORD PTR DS:[EBX]
00409FC3 |. 8B53 04       |MOV EDX,DWORD PTR DS:[EBX+4]
00409FC6 |. 8B5D FC       |MOV EBX,DWORD PTR SS:[EBP-4]
00409FC9 |. E8 2ABCFFFF   |CALL jzyq.00405BF8              ;  CALL 内是 DIV EBX   XOR EDX,EDX
00409FCE |. 59            |POP ECX
00409FCF |. 5B            |POP EBX
00409FD0 |. 8903          |MOV DWORD PTR DS:[EBX],EAX
00409FD2 |. 8953 04       |MOV DWORD PTR DS:[EBX+4],EDX
00409FD5 |. 09D0          |OR EAX,EDX
00409FD7 |.^ 75 C0         \JNZ SHORT jzyq.00409F99
```

## 00515362 - 00515373

```
00515362 |. 8B85 58FFFFFF MOV EAX,DWORD PTR SS:[EBP-A8]
00515368 |. E8 4342EFFF   CALL jzyq.004095B0            ;  把 CALL 00515028 运算得到的字符串放到这个 CALL
0051536D |. 8D95 5CFFFFFF LEA EDX,DWORD PTR SS:[EBP-A4]   ;  把 CALL 00515028 运算结果存到 EAX 中
00515373   E8 90FDFFFF   CALL jzyq.00515108            ;  把 EAX 的数经 CALL 运算可以得到注册码的中间 N 位 跟进
```

`CALL jzyq.00515108` 跟进

## 00515108 - 005151B5

```
00515108 /$ 55            PUSH EBP
00515109 |. 8BEC          MOV EBP,ESP
0051510B |. 33C9          XOR ECX,ECX
0051510D |. 51            PUSH ECX
0051510E |. 51            PUSH ECX
0051510F |. 51            PUSH ECX
00515110 |. 51            PUSH ECX
00515111 |. 51            PUSH ECX
00515112 |. 51            PUSH ECX
00515113 |. 53            PUSH EBX
00515114 |. 56            PUSH ESI
00515115 |. 8BF2          MOV ESI,EDX
00515117 |. 8BD8          MOV EBX,EAX                     上次计算 EAX 的结果给 EBX
00515119 |. 33C0          XOR EAX,EAX
0051511B |. 55            PUSH EBP
0051511C |. 68 54525100   PUSH jzyq.00515254
00515121 |. 64:FF30       PUSH DWORD PTR FS:[EAX]
00515124 |. 64:8920       MOV DWORD PTR FS:[EAX],ESP
00515127 |. 81F3 8776FBDD XOR EBX,DDFB7687                ; EBX 与 DDFB7687 逻辑异或运
0051512D |. 8BC3          MOV EAX,EBX                       ; EBX 与 DDFB7687 逻辑异或运的结果给 EAX
0051512F |. 33D2          XOR EDX,EDX
00515131 |. 52            PUSH EDX
00515132   50            PUSH EAX                          ; 在把 EAX 压入堆栈这个数据有用
00515133   8D45 FC       LEA EAX,DWORD PTR SS:[EBP-4]
00515136   E8 4144EFFF   CALL jzyq.0040957C                ; 在上面我们已经他析了这个 CALL 我就不跟进去了
0051513B   8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
0051513E   0FB600        MOVZX EAX,BYTE PTR DS:[EAX]       ; 取字符串的第 1 位给 EAX
00515141   8B55 FC       MOV EDX,DWORD PTR SS:[EBP-4]      ; 把 CALL jzyq.0040957C 算出的字串给 EDX
00515144   0FB652 01     MOVZX EDX,BYTE PTR DS:[EDX+1]     ; 取字符串的第 2 位给 EDX
00515148 |. 03C2          ADD EAX,EDX                       ; 字符串的第 1 位与字符串的第 2 位逻辑与运算给 EAX
0051514A |. B9 05000000   MOV ECX,5                         ; ECX 置 5
0051514F |. 99            CDQ                                 ; EDX 清 0
00515150 |. F7F9          IDIV ECX                            ; EAX/ECX 带符号数除法 佘数存入 EDX
00515152 |. 80C2 66       ADD DL,66                           ; DL 加 66
00515155 |. 8855 F8       MOV BYTE PTR SS:[EBP-8],DL          ; 把结果存到 DS：[EBP-8]
00515158 |. 8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
0051515B |. 0FB640 02     MOVZX EAX,BYTE PTR DS:[EAX+2]
0051515F |. 8B55 FC       MOV EDX,DWORD PTR SS:[EBP-4]
00515162 |. 0FB652 03     MOVZX EDX,BYTE PTR DS:[EDX+3]
00515166 |. 03C2          ADD EAX,EDX
00515168 |. B9 05000000   MOV ECX,5
0051516D |. 99            CDQ
0051516E |. F7F9          IDIV ECX
00515170 |. 80C2 75       ADD DL,75
00515173 |. 8855 F9       MOV BYTE PTR SS:[EBP-7],DL          ; 把结果存到 DS：[EBP-7]
00515176 |. 8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
00515179 |. 0FB640 04     MOVZX EAX,BYTE PTR DS:[EAX+4]
0051517D |. 8B55 FC       MOV EDX,DWORD PTR SS:[EBP-4]
00515180 |. 0FB652 05     MOVZX EDX,BYTE PTR DS:[EDX+5]
00515184 |. 03C2          ADD EAX,EDX
00515186 |. B9 05000000   MOV ECX,5
0051518B |. 99            CDQ
0051518C |. F7F9          IDIV ECX
0051518E |. 80C2 7A       ADD DL,7A
00515191 |. 8855 FA       MOV BYTE PTR SS:[EBP-6],DL          ; 把结果存到 DS：[EBP-6]
00515194 |. 8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
00515197 |. 0FB640 06     MOVZX EAX,BYTE PTR DS:[EAX+6]
0051519B |. 8B55 FC       MOV EDX,DWORD PTR SS:[EBP-4]
0051519E |. 0FB652 07     MOVZX EDX,BYTE PTR DS:[EDX+7]
005151A2 |. 03C2          ADD EAX,EDX
005151A4 |. 8B55 FC       MOV EDX,DWORD PTR SS:[EBP-4]
005151A7 |. 0FB652 08     MOVZX EDX,BYTE PTR DS:[EDX+8]
005151AB |. 03C2          ADD EAX,EDX
005151AD |. B9 05000000   MOV ECX,5
005151B2 |. 99            CDQ
005151B3 |. F7F9          IDIV ECX
005151B5 |. 80C2 69       ADD DL,69
```



```
005151B8  |.  8855 FB       MOV BYTE PTR SS:[EBP-5],DL                 把结果存到 DS：[EBP-5]  
005151BB  |.  8D45 F4       LEA EAX,DWORD PTR SS:[EBP-C]  
005151BE  |.  8A55 F8       MOV DL,BYTE PTR SS:[EBP-8]  
005151C1  |.  E8 36FAEEFF   CALL jzyq.00404BFC                       
005151C6  |.  8B45 F4       MOV EAX,DWORD PTR SS:[EBP-C]             ; 
005151C9  |.  8D55 FC       LEA EDX,DWORD PTR SS:[EBP-4]             把 `CALL jzyq.0040957C` 算出的字串给 `EAX`  
005151CC  |.  B9 07000000   MOV ECX,7  
005151D1  |.  E8 DEFDEEFF   CALL jzyq.00404FB4         把 `SS:[EBP-8]`中字符插入 `CALL 0040957C` 算出的字串的第 7 位  
005151D6  |.  8D45 F0       LEA EAX,DWORD PTR SS:[EBP-10]  
005151D9  |.  8A55 FB       MOV DL,BYTE PTR SS:[EBP-5]  
005151DC  |.  E8 1BFAEEFF   CALL jzyq.00404BFC                     
005151E1  |.  8B45 F0       MOV EAX,DWORD PTR SS:[EBP-10]  
005151E4  |.  8D55 FC       LEA EDX,DWORD PTR SS:[EBP-4]  
005151E7  |.  B9 03000000   MOV ECX,3  
005151EC  |.  E8 C3FDEEFF   CALL jzyq.00404FB4            把 `SS:[EBP-5]`中字符插入 `CALL 0040957C` 算出的字串的第 3 位  
005151F1  |.  8D45 EC       LEA EAX,DWORD PTR SS:[EBP-14]  
005151F4  |.  8A55 F9       MOV DL,BYTE PTR SS:[EBP-7]  
005151F7  |.  E8 00FAEEFF   CALL jzyq.00404BFC               
005151FC  |.  8B45 EC       MOV EAX,DWORD PTR SS:[EBP-14]  
005151FF  |.  8D55 FC       LEA EDX,DWORD PTR SS:[EBP-4]  
00515202  |.  B9 05000000   MOV ECX,5  
00515207  |.  E8 A8FDEEFF   CALL jzyq.00404FB4           把 `SS:[EBP-7]`中字符插入 `CALL 0040957C` 算出的字串的第 5 位  
0051520C  |.  8D45 E8       LEA EAX,DWORD PTR SS:[EBP-18]  
0051520F  |.  8A55 FA       MOV DL,BYTE PTR SS:[EBP-6]  
00515212  |.  E8 E5F9EEFF   CALL jzyq.00404BFC                 
00515217  |.  8B45 E8       MOV EAX,DWORD PTR SS:[EBP-18]  
0051521A  |.  8D55 FC       LEA EDX,DWORD PTR SS:[EBP-4]  
0051521D  |.  B9 09000000   MOV ECX,9  
00515222  |.  E8 8DFDEEFF   CALL jzyq.00404FB4          把 `SS:[EBP-6]`中字符插入 `CALL 0040957C` 算出的字串的第 9 位  
00515227  |.  8BC6          MOV EAX,ESI  
00515229  |.  8B55 FC       MOV EDX,DWORD PTR SS:[EBP-4]  
0051522C  |.  E8 3FF8EEFF   CALL jzyq.00404A70
00515231  |.  33C0          XOR EAX,EAX  
00515233  |.  5A            POP EDX  
00515234  |.  59            POP ECX  
00515235  |.  59            POP ECX  
00515236  |.  64:8910       MOV DWORD PTR FS:[EAX],EDX  
00515239  |.  68 5B525100   PUSH jzyq.0051525B  
0051523E  |>  8D45 E8       LEA EAX,DWORD PTR SS:[EBP-18]  
00515241  |.  BA 04000000   MOV EDX,4  
00515246  |.  E8 F5F7EEFF   CALL jzyq.00404A40  
0051524B  |.  8D45 FC       LEA EAX,DWORD PTR SS:[EBP-4]  
0051524E  |.  E8 C9F7EEFF   CALL jzyq.00404A1C  
00515253  \.  C3            RETN  
00515254   .^ E9 67F0EEFF   JMP jzyq.004042C0  
00515259   .^ EB E3         JMP SHORT jzyq.0051523E  
0051525B   .  5E            POP ESI  
0051525C   .  5B            POP EBX  
0051525D   .  8BE5          MOV ESP,EBP  
0051525F   .  5D            POP EBP  
00515260   .  C3            RETN  
```

```
00515378  |.  FFB5 5CFFFFFF PUSH DWORD PTR SS:[EBP-A4]      ;  把 `CALL` 得到字串压入堆栈  
0051537E  |.  68 20555100   PUSH jzyq.00515520              ;  放入固定字符 1  
00515383  |.  8D85 68FFFFFF LEA EAX,DWORD PTR SS:[EBP-98]  
00515389  |.  BA 03000000   MOV EDX,3  
0051538E  |.  E8 01FAEEFF   CALL jzyq.00404D94              ;  把 `CALL 0040954C` 计算出的注册码 + `CALL 00515108` 计算出的注册码  
00515393  |.  8B95 68FFFFFF MOV EDX,DWORD PTR SS:[EBP-98]   ;  +1 这就是真的注册码了  
00515399  |.  58            POP EAX  
0051539A  |.  E8 79FAEEFF   CALL jzyq.00404E18  
0051539F  |.  0F85 A3000000 JNZ jzyq.00515448  
005153A5  |.  B8 2C555100   MOV EAX,jzyq.0051552C           ;  注册码正确，感谢你的注册！  
005153AA  |.  E8 F169F2FF   CALL jzyq.0043BDA0  
005153AF  |.  A1 109C5300   MOV EAX,DWORD PTR DS:[539C10]  
005153B4  |.  8B00          MOV EAX,DWORD PTR DS:[EAX]  
005153B6  |.  8B80 C8040000 MOV EAX,DWORD PTR DS:[EAX+4C8]  
005153BC  |.  8B80 08020000 MOV EAX,DWORD PTR DS:[EAX+208]  
005153C2  |.  33D2          XOR EDX,EDX  
005153C4  |.  E8 A79BF5FF   CALL jzyq.0046EF70  
005153C9  |.  BA 50555100   MOV EDX,jzyq.00515550           ;  精装友情-软件已注册  
005153CE  |.  E8 F99AF5FF   CALL jzyq.0046EECC  
005153D3  |.  8D85 50FFFFFF LEA EAX,DWORD PTR SS:[EBP-B0]
```

中间无用略去

```
00515448  |>  B8 B4555100   MOV EAX,jzyq.005155B4           ;  注册失败，请重新注册！  
0051544D  |.  E8 4E69F2FF   CALL jzyq.0043BDA0  
```

总的来说分析得不错，但从技术的角度来说，有些不是在绕圈子，就是你自己也没认真研读。例如：

```
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~可以找这个关键算法
004033CA  |>  80EB 30       /SUB BL,30                       ；`BL` 是订单号的字符
004033CD  |. |80FB 09       |CMP BL,9
004033D0  |. |77 25         |JA SHORT jzyq.004033F7
004033D2  |. |39F8          |CMP EAX,EDI
004033D4  |. |77 21         |JA SHORT jzyq.004033F7
004033D6  |. |8D0480        |LEA EAX,DWORD PTR DS:[EAX+EAX*4]        ;  把计算结果*4+把计算结果
004033D9  |. |01C0          |ADD EAX,EAX                         ;  把计算结果+把计算结果
004033DB  |. |01D8          |ADD EAX,EBX                         ;  把计算结果+取出的字符
004033DD  |. |8A1E          |MOV BL,BYTE PTR DS:[ESI]              ;  循环取出字符给 `BL`
004033DF  |. |46            |INC ESI                             ;  `ESI` 自 加 1
004033E0  |. |84DB          |TEST BL,BL
004033E2  |.^\75 E6         \JNZ SHORT jzyq.004033CA
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
```

这截代码其实就是按十进制模式转换字符串为整数。

```
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
00515320  |.  B9 40080000   MOV ECX,840                   ;  `ECX` 置 840
00515325  |.  99            CDQ                             ;  `EDX` 清 0
00515326  |.  F7F9          IDIV ECX                        ;  把 `CALL 004095B0 EAX` 的结果除以 `ECX`
00515328  |.  8BC2          MOV EAX,EDX                     ;  把佘数给 `EAX`
0051532A  |.  8D95 64FFFFFF LEA EDX,DWORD PTR SS:[EBP-9C]
00515330  |.  E8 1742EFFF   CALL jzyq.0040954C              ;  把上次计算 `EAX` 的结果放这个 CDLL 运算可以得到注册码的前 N 位
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
```

典型的求模运算。

```
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~可以看到算法了
0040A022  |$  B9 0A000000   MOV ECX,0A                     ;  `ECX` 置 0A      ; Case 55 ('U') of switch `00409FF6`
0040A027  |>  8D75 9F       LEA ESI,DWORD PTR SS:[EBP-61]
0040A02A  |>  31D2          /XOR EDX,EDX                     ;  `EDX` 清 0
0040A02C  |.  F7F1          |DIV ECX                        ;  `EAX` 是上次运算的结果/0A
0040A02E  |.  80C2 30       |ADD DL,30                      ;  `DL`+30
0040A031  |.  80FA 3A       |CMP DL,3A                      ;  `DL` 与 3A 比较
0040A034  |.  72 03         |JB SHORT jzyq.0040A039         ;  不高于等于时转移
0040A036  |.  80C2 07       |ADD DL,7
0040A039  |>  4E            |DEC ESI                        ;  `ESI` 自减 1
0040A03A  |.  8816          |MOV BYTE PTR DS:[ESI],DL       ;  把结果存入 `DS:[ESI]`
0040A03C  |.  09C0          |OR EAX,EAX
0040A03E  |.^ 75 EA        \JNZ SHORT jzyq.0040A02A
```

将整数转换成十进制字符。

而经过你的描述，看官是不是有云中雾里的感觉？

仅供参考。![](img/755d44573ea0045d4eb19f2164c82d4e.jpg "扮鬼脸")
