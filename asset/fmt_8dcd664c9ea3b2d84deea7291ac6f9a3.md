
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
