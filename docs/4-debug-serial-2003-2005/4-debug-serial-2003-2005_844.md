
# 大掌柜—电脑配件进销存系统 V3.5__ 注册流程分析(高手莫进)

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-750.htm`](https://www.pediy.com/kssd/pediy07/pediy7-750.htm)

【破文标题】《大掌柜》—电脑配件进销存系统 V3.5__ 注册流程分析

【破文作者】`fcrjzmd`

【破解工具】`PEID`, `OD`,

【破解平台】`WIN XP`

【软件大小】7.63 MB

【原版下载】`http://www.hktk.com/soft/soft_pro_trade/dzgdnjxc.html`

【保护方式】`UPX 0.89.6 - 1.02 / 1.05 - 1.24 (Delphi) stub -> Markus & Laszlo`

软件名称：《大掌柜》—电脑配件进销存系统 [国产]

软件授权：共享软件

注册费用：单机版：320 元人民币

使用平台：`Win9X/Me/NT/2000/XP`

软件简介：

《大掌柜》——电脑配件进销存系统是在实际运用中逐步开发完成的一套管理系统，经过了实践检验，适合于电脑市场的计算机公司或个体柜台对电脑配件的进、销、存管理和组装兼容机的销售与客户管理。功能强大，简单易用！欢迎使用！

## 破解过程

这个是 V3.5 旧版本网上有很多破解版，有 V3.6 版(也有破解过的)但是没有找着于是正拿来练手吧我菜鸟没有办法，捡过软柿子吧。没想到我竞用 2 天时间惨亏哦(当然晚上要用来睡觉了~~呵呵 。)

总算把注册码算法流程分析完整了。好久没发过破文因为学艺不精。破文必有破漏敬请原谅哦。。不善言语转入正题吧。。

用 `PEiD` 检测显示《`Borland Delphi 6.0 - 7.0`》无壳，适合我这种菜鸟。

运行软件，注册有提示。用 `OLLYDBG` 载入之后，使用字符串参考。或者可以通过传说中的 `OD` 万能断点方法都快速找要点。

看到一些字符：

`Ultra String Reference`，项目 1000

`Address=0054CCB7`
`Disassembly=MOV EDX,ItManage.0054CD3C`
`Text String=注册失败`

```
0054CC34    55              PUSH    EBP                               ; 下断 F2
0054CC35    8BEC            MOV     EBP,ESP
0054CC37    6A 00           PUSH    0
0054CC39    6A 00           PUSH    0
0054CC3B    53              PUSH    EBX
0054CC3C    8BD8            MOV     EBX,EAX
0054CC3E    33C0            XOR     EAX,EAX
0054CC40    55              PUSH    EBP
0054CC41    68 E1CC5400     PUSH    ItManage.0054CCE1
0054CC46    64:FF30        PUSH    DWORD PTR FS:[EAX]
0054CC49    64:8920        MOV     DWORD PTR FS:[EAX],ESP
0054CC4C    8D55 FC         LEA     EDX,DWORD PTR SS:[EBP-4]
0054CC4F    8B83 F8020000  MOV     EAX,DWORD PTR DS:[EBX+2F8]
0054CC55    E8 4614F0FF     CALL    ItManage.0044E0A0                ; 获取假码和长度 B 并送入 EAX
0054CC5A    8B45 FC         MOV     EAX,DWORD PTR SS:[EBP-4]         ; 假码送入 EAX___《下万能断 77D3353D(我的机器)返回到这里》
0054CC5D    50              PUSH    EAX                               ; 压入假码
0054CC5E    E8 91FFFFFF     CALL    ItManage.0054CBF4                ; 获取硬盘 ID 我的 C 盘序列号为 E8317C71
0054CC63    8D55 F8         LEA     EDX,DWORD PTR SS:[EBP-8]          ; 将地址 0012F210 送入 EDX
0054CC66    E8 71C7EBFF     CALL    ItManage.004093DC                ; 关键 CALL 跟进。E8317C71 取反除 A，余数加 30。得到-399410063
0054CC6B    8B55 F8         MOV     EDX,DWORD PTR SS:[EBP-8]          ; 从上面 CALL 运算返回的值 EDX=-399410063
0054CC6E    A1 104A5500     MOV     EAX,DWORD PTR DS:[554A10]         ;
0054CC73    8B00            MOV     EAX,DWORD PTR DS:[EAX]            ;
0054CC75    8B80 AC030000  MOV     EAX,DWORD PTR DS:[EAX+3AC]        ;
0054CC7B    B9 F8CC5400     MOV     ECX,ItManage.0054CCF8             ; ASCII "www.luckguest.com"
0054CC80    E8 57DBF7FF     CALL    ItManage.004CA7DC                ; 关键 CALL 跟进。。
0054CC85    84C0            TEST    AL,AL
0054CC87    75 0A           JNZ     SHORT ItManage.0054CC93           ; 相等 OVER!爆破第 1 点 NOP 吧!!
0054CC89    B8 14CD5400     MOV     EAX,ItManage.0054CD14
0054CC8E    E8 E9A5EFFF     CALL    ItManage.0044727C                ; 注册码错误!
0054CC93    A1 104A5500     MOV     EAX,DWORD PTR DS:[554A10]
0054CC98    8B00            MOV     EAX,DWORD PTR DS:[EAX]
0054CC9A    8B80 AC030000  MOV     EAX,DWORD PTR DS:[EAX+3AC]
0054CCA0    E8 2BD8F7FF     CALL    ItManage.004CA4D0                ; 这里还有一处注册码校验 CALL
0054CCA5    84C0            TEST    AL,AL
0054CCA7    74 0E           JE      SHORT ItManage.0054CCB7           ; 不相等则成功，反之 OVER!爆破第 2 点 NOP 吧!!
0054CCA9    BA 28CD5400     MOV     EDX,ItManage.0054CD28             ; 注册成功!
0054CCAE    8BC3            MOV     EAX,EBX
0054CCB0    E8 1B14F0FF     CALL    ItManage.0044E0D0
0054CCB5    EB 0C           JMP     SHORT ItManage.0054CCC3
0054CCB7    BA 3CCD5400     MOV     EDX,ItManage.0054CD3C             ; 注册失败!
0054CCBC    8BC3            MOV     EAX,EBX
0054CCBE    E8 0D14F0FF     CALL    ItManage.0044E0D0
```

当然我主要分析的是算法跟进 `004093DC`

```
004093E1    894424 04       MOV     DWORD PTR SS:[ESP+4],EAX        ; 将 EAX 保存
004093E5    C64424 08 00    MOV     BYTE PTR SS:[ESP+8],0           ; 将 0 送入 0012F1F4
004093EA    8D4C24 04       LEA     ECX,DWORD PTR SS:[ESP+4]        ; 把地址 0012F1F4 送入 ECX
004093EE    8BC2            MOV     EAX,EDX                          ;
004093F0    BA 08944000     MOV     EDX,ItManage.00409408           ; ASCII "%d"
004093F5    E8 8E110000     CALL    ItManage.0040A588                ; 关键 CALL 跟进。。
004093FA    59              POP     ECX
004093FB    5A              POP     EDX
```

跟进 `0040A588`

```
0040A588    55              PUSH    EBP
0040A589    8BEC            MOV     EBP,ESP
0040A58B    81C4 04F0FFFF   ADD     ESP,-0FFC
0040A591    50              PUSH    EAX
0040A592    83C4 F8         ADD     ESP,-8
0040A595    53              PUSH    EBX
0040A596    56              PUSH    ESI
0040A597    894D F8         MOV     DWORD PTR SS:[EBP-8],ECX
0040A59A    8955 FC         MOV     DWORD PTR SS:[EBP-4],EDX
0040A59D    8BF0            MOV     ESI,EAX
0040A59F    BB 00100000     MOV     EBX,1000
0040A5A4    8B45 FC         MOV     EAX,DWORD PTR SS:[EBP-4]
0040A5A7    E8 00A5FFFF     CALL    ItManage.00404AAC                ; 取%d 的长度
0040A5AC    3D 000C0000     CMP     EAX,0C00                          ; %d 的长度 2 和 0C00 做比较
0040A5B1    7D 26           JGE     SHORT ItManage.0040A5D9          ; 大于或等于则跳
0040A5B3    8B45 FC         MOV     EAX,DWORD PTR SS:[EBP-4]         ; %d 送入 EAX
0040A5B6    E8 F1A4FFFF     CALL    ItManage.00404AAC                ; 再取%d 的长度
0040A5BB    50              PUSH    EAX                               ; 压入长度
0040A5BC    8B45 F8         MOV     EAX,DWORD PTR SS:[EBP-8]         ; EAX=0012F1F0
0040A5BF    50              PUSH    EAX                               ; 压入 EAX=0012F1F0=E8317C71
0040A5C0    8B45 08         MOV     EAX,DWORD PTR SS:[EBP+8]         ; EAX=0
0040A5C3    50              PUSH    EAX                               ; 压入 EAX
0040A5C4    8B4D FC         MOV     ECX,DWORD PTR SS:[EBP-4]         ; ECX=%d
0040A5C7    8D85 F8EFFFFF   LEA     EAX,DWORD PTR SS:[EBP-1008]      ; EAX=0012E1DC
0040A5CD    BA FF0F0000     MOV     EDX,0FFF                          ; EDX=0FFF
0040A5D2    E8 09FBFFFF     CALL    ItManage.0040A0E0                ; 关键 CALL 跟进。。
```

跟进 `0040A0E0`

```
0040A0E0    55              PUSH    EBP
0040A0E1    8BEC            MOV     EBP,ESP
0040A0E3    83C4 8C         ADD     ESP,-74
0040A0E6    53              PUSH    EBX
0040A0E7    33DB            XOR     EBX,EBX
0040A0E9    895D EC         MOV     DWORD PTR SS:[EBP-14],EBX
0040A0EC    53              PUSH    EBX
0040A0ED    56              PUSH    ESI
0040A0EE    57              PUSH    EDI
0040A0EF    89C7            MOV     EDI,EAX
0040A0F1    89CE            MOV     ESI,ECX
0040A0F3    31C0            XOR     EAX,EAX
0040A0F5    8945 FC         MOV     DWORD PTR SS:[EBP-4],EAX
0040A0F8    034D 10         ADD     ECX,DWORD PTR SS:[EBP+10]        ; %d+2=0040940A
0040A0FB    897D F8         MOV     DWORD PTR SS:[EBP-8],EDI
0040A0FE    31C0            XOR     EAX,EAX
0040A100    8945 F4         MOV     DWORD PTR SS:[EBP-C],EAX
0040A103    8945 F0         MOV     DWORD PTR SS:[EBP-10],EAX
0040A106    8945 EC         MOV     DWORD PTR SS:[EBP-14],EAX
0040A109    09D2            OR      EDX,EDX
0040A10B    74 0D           JE      SHORT ItManage.0040A11A
0040A10D    39CE            CMP     ESI,ECX
0040A10F    74 09           JE      SHORT ItManage.0040A11A
0040A111    AC              LODS    BYTE PTR DS:[ESI]
```



`0040A112` `3C 25` `CMP` `AL,25`  

`0040A114` `74 0E` `JE` `SHORT ItManage.0040A124`  

`0040A116` `AA` `STOS` `BYTE PTR ES:[EDI]`  

`0040A117` `4A` `DEC` `EDX`  

`0040A118` `^ 75 F3` `JNZ` `SHORT ItManage.0040A10D`  

`0040A11A` `89F8` `MOV` `EAX,EDI`  

`0040A11C` `2B45 F8` `SUB` `EAX,DWORD PTR SS:[EBP-8]`  

`0040A11F` `E9 C0030000` `JMP` `ItManage.0040A4E4`  

`0040A124` `39CE` `CMP` `ESI,ECX`  

`0040A126` `^ 74 F2` `JE` `SHORT ItManage.0040A11A`  

`0040A128` `AC` `LODS` `BYTE PTR DS:[ESI]`  

`0040A129` `3C 25` `CMP` `AL,25`  

`0040A12B` `^ 74 E9` `JE` `SHORT ItManage.0040A116`  

`0040A12D` `8D5E FE` `LEA` `EBX,DWORD PTR DS:[ESI-2]`  

`0040A130` `895D E8` `MOV` `DWORD PTR SS:[EBP-18],EBX`  

`0040A133` `8845 E7` `MOV` `BYTE PTR SS:[EBP-19],AL`  

`0040A136` `3C 2D` `CMP` `AL,2D`  

`0040A138` `75 05` `JNZ` `SHORT ItManage.0040A13F`  

`0040A13A` `39CE` `CMP` `ESI,ECX`  

`0040A13C` `^ 74 DC` `JE` `SHORT ItManage.0040A11A`  

`0040A13E` `AC` `LODS` `BYTE PTR DS:[ESI]`  

`0040A13F` `E8 7E000000` `CALL` `ItManage.0040A1C2`  

`0040A144` `3C 3A` `CMP` `AL,3A`  

`0040A146` `75 0A` `JNZ` `SHORT ItManage.0040A152`  

`0040A148` `895D F4` `MOV` `DWORD PTR SS:[EBP-C],EBX`  

`0040A14B` `39CE` `CMP` `ESI,ECX`  

`0040A14D` `^ 74 CB` `JE` `SHORT ItManage.0040A11A`  

`0040A14F` `AC` `LODS` `BYTE PTR DS:[ESI]`  

`0040A150` `^ EB E1` `JMP` `SHORT ItManage.0040A133`  

`0040A152` `895D E0` `MOV` `DWORD PTR SS:[EBP-20],EBX`  

`0040A155` `BB FFFFFFFF` `MOV` `EBX,-1`  

`0040A15A` `3C 2E` `CMP` `AL,2E`  

`0040A15C` `75 0A` `JNZ` `SHORT ItManage.0040A168`  

`0040A15E` `39CE` `CMP` `ESI,ECX`  

`0040A160` `^ 74 B8` `JE` `SHORT ItManage.0040A11A`  

`0040A162` `AC` `LODS` `BYTE PTR DS:[ESI]`  

`0040A163` `E8 5A000000` `CALL` `ItManage.0040A1C2`  

`0040A168` `895D DC` `MOV` `DWORD PTR SS:[EBP-24],EBX`  

`0040A16B` `8975 D8` `MOV` `DWORD PTR SS:[EBP-28],ESI`  

`0040A16E` `51` `PUSH` `ECX`  

`0040A16F` `52` `PUSH` `EDX`  

`0040A170` `E8 95000000` `CALL` `ItManage.0040A20A` ; 关键 `CALL` 跟进。。

`0040A175` `5A` `POP` `EDX`  

`0040A176` `8B5D E0` `MOV` `EBX,DWORD PTR SS:[EBP-20]`  

`0040A179` `29CB` `SUB` `EBX,ECX`  

`0040A17B` `73 02` `JNB` `SHORT ItManage.0040A17F`  

\---------------------------------------------------------------------

跟进 `0040A20A`

`0040A20A` `24 DF` `AND` `AL,0DF`

`0040A20C` `88C1` `MOV` `CL,AL`

`0040A20E` `B8 01000000` `MOV` `EAX,1`

`0040A213` `8B5D F4` `MOV` `EBX,DWORD PTR SS:[EBP-C]`

`0040A216` `3B5D 08` `CMP` `EBX,DWORD PTR SS:[EBP+8]`

`0040A219` `77 5C` `JA` `SHORT ItManage.0040A277`

`0040A21B` `FF45 F4` `INC` `DWORD PTR SS:[EBP-C]`

`0040A21E` `8B75 0C` `MOV` `ESI,DWORD PTR SS:[EBP+C]`

`0040A221` `8D34DE` `LEA` `ESI,DWORD PTR DS:[ESI+EBX*8]`

`0040A224` `8B06` `MOV` `EAX,DWORD PTR DS:[ESI]`

`0040A226` `0FB656 04` `MOVZX` `EDX,BYTE PTR DS:[ESI+4]`

`0040A22A` `FF2495 31A24000` `JMP` `DWORD PTR DS:[EDX*4+40A231]` ; 跳走

...........省略部分代码....................................

`0040A332` `80F9 44` `CMP` `CL,44`

`0040A335` `74 15` `JE` `SHORT ItManage.0040A34C`

`0040A337` `80F9 55` `CMP` `CL,55`

`0040A33A` `74 22` `JE` `SHORT ItManage.0040A35E`

`0040A33C` `80F9 58` `CMP` `CL,58`

`0040A33F` `^ 0F85 30FFFFFF` `JNZ` `ItManage.0040A275`

`0040A345` `B9 10000000` `MOV` `ECX,10`

`0040A34A` `EB 17` `JMP` `SHORT ItManage.0040A363`

`0040A34C` `09C0` `OR` `EAX,EAX`

`0040A34E` `79 0E` `JNS` `SHORT ItManage.0040A35E`

`0040A350` `F7D8` `NEG` `EAX` ; `E8317C71` 求反得到 `17CE838F`

`0040A352` `E8 07000000` `CALL` `ItManage.0040A35E` ; 关键 `CALL` 跟进。。

`0040A357` `B0 2D` `MOV` `AL,2D` ; `AL=2D`

`0040A359` `41` `INC` `ECX`

`0040A35A` `4E` `DEC` `ESI` ; `ESI-1`

`0040A35B` `8806` `MOV` `BYTE PTR DS:[ESI],AL` ; `0012E155=AL=2D` (得到 `-399410063`)

`0040A35D` `C3` `RETN`

\---------------------------------------------------------------------

跟进 `0040A35E`

`0040A35E` `B9 0A000000` `MOV` `ECX,0A` ; `ECX=0A`

`0040A363` `8D75 9F` `LEA` `ESI,DWORD PTR SS:[EBP-61]` ; `ESI=0012F1F0`

`0040A366` `31D2` `XOR` `EDX,EDX` ; `EDX` 清 `0`

`0040A368` `F7F1` `DIV` `ECX` ; 除法运算，商存入 `EAX`，余数存入 `EDX`。除尽为止

`0040A36A` `80C2 30` `ADD` `DL,30` ; 余数和 `30` 相加

`0040A36D` `80FA 3A` `CMP` `DL,3A` ; 相加的数和 `3A` 做比较

`0040A370` `72 03` `JB` `SHORT ItManage.0040A375` ; 小于 `3A` 跳走

`0040A372` `80C2 07` `ADD` `DL,7`

`0040A375` `4E` `DEC` `ESI` ; `ESI-1`

`0040A376` `8816` `MOV` `BYTE PTR DS:[ESI],DL` ; 每除得到值保存在 `0012E15E`

`0040A378` `09C0` `OR` `EAX,EAX` ; `EAX OR EAX`

`0040A37A` `^ 75 EA` `JNZ` `SHORT ItManage.0040A366` ; 不相等则跳走继续循环运算

`0040A37C` `8D4D 9F` `LEA` `ECX,DWORD PTR SS:[EBP-61]`

`0040A37F` `29F1` `SUB` `ECX,ESI`

`0040A381` `8B55 DC` `MOV` `EDX,DWORD PTR SS:[EBP-24]`

`0040A384` `83FA 10` `CMP` `EDX,10`

`0040A387` `76 01` `JBE` `SHORT ItManage.0040A38A`

\---------------------------------------------------------------------

就是这样弯山曲水来到这里运算得一个值 `399410063`，返回到 `0040A357` 和 `2D(-)` 合并最后得出 `-399410063`。好了到这里我再返回来到上面 `0054CC80` 吧

跟进 `004CA7DC`

`004CA7DC` `55` `PUSH` `EBP`

`004CA7DD` `8BEC` `MOV` `EBP,ESP`

`004CA7DF` `83C4 F0` `ADD` `ESP,-10`

`004CA7E2` `53` `PUSH` `EBX`

`004CA7E3` `33DB` `XOR` `EBX,EBX`

`004CA7E5` `895D F0` `MOV` `DWORD PTR SS:[EBP-10],EBX` ; `0012F1E0=0`

`004CA7E8` `895D F4` `MOV` `DWORD PTR SS:[EBP-C],EBX` ; `0012F1E8=0`

`004CA7EB` `894D F8` `MOV` `DWORD PTR SS:[EBP-8],ECX` ; `0012F1EC=www.luckguest.com`

`004CA7EE` `8955 FC` `MOV` `DWORD PTR SS:[EBP-4],EDX` ; `0012F1F0=-399410063`

`004CA7F1` `8BD8` `MOV` `EBX,EAX` ; `EBX=EAX=00FF50EC`

`004CA7F3` `8B45 FC` `MOV` `EAX,DWORD PTR SS:[EBP-4]` ; `EAX=-399410063`

`004CA7F6` `E8 99A4F3FF` `CALL` `ItManage.00404C94` ; 获取网址

`004CA7FB` `8B45 F8` `MOV` `EAX,DWORD PTR SS:[EBP-8]` ; `EAX=www.luckguest.com`

`004CA7FE` `E8 91A4F3FF` `CALL` `ItManage.00404C94` ; 获取假码

`004CA803` `8B45 08` `MOV` `EAX,DWORD PTR SS:[EBP+8]` ; `EAX=假码`

`004CA806` `E8 89A4F3FF` `CALL` `ItManage.00404C94`

`004CA80B` `33C0` `XOR` `EAX,EAX` ; `EAX` 清 `0`

`004CA80D` `55` `PUSH` `EBP`

`004CA80E` `68 C6A84C00` `PUSH` `ItManage.004CA8C6`

`004CA813` `64:FF30` `PUSH` `DWORD PTR FS:[EAX]`

`004CA816` `64:8920` `MOV` `DWORD PTR FS:[EAX],ESP` ; `0012F200=0012F1D4`

`004CA819` `8B45 FC` `MOV` `EAX,DWORD PTR SS:[EBP-4]` ; `EAX=-399410063`

`004CA81C` `E8 8BA2F3FF` `CALL` `ItManage.00404AAC` ; 获取 `-399410063` 的长度 `A` 送入 `EAX`

`004CA821` `3B43 4C` `CMP` `EAX,DWORD PTR DS:[EBX+4C]` ; `-399410063` 的长度 `A` 和 `19` 做比较

`004CA824` `7F 19` `JG` `SHORT ItManage.004CA83F` ; 大于 `19` 跳走



## 004CA826

`004CA826    8B45 FC           MOV     EAX,DWORD PTR SS:[EBP-4]        ; 再次 EAX=-399410063`

`004CA829    E8 7EA2F3FF     CALL    ItManage.00404AAC                ; 再次获取-399410063 的长度 A 送入 EAX`

`004CA82E    3B43 50         CMP     EAX,DWORD PTR DS:[EBX+50]       ; -399410063 的长度 A 和 3 做比较`

`004CA831    7C 0C           JL      SHORT ItManage.004CA83F         ; 是否小于 3 跳走`

`004CA833    8B45 08         MOV     EAX,DWORD PTR SS:[EBP+8]        ; EAX=假码`

`004CA836    E8 71A2F3FF     CALL    ItManage.00404AAC                ; 获取假码长度 B，并送入 EAX`

`004CA83B    85C0            TEST    EAX,EAX                          ; 测试假码长度 B`

`004CA83D    75 04           JNZ     SHORT ItManage.004CA843         ; 不等则跳走`

`004CA83F    33DB            XOR     EBX,EBX`

`004CA841    EB 60           JMP     SHORT ItManage.004CA8A3`

`004CA843    8D55 F4         LEA     EDX,DWORD PTR SS:[EBP-C]        ; 将地址 0012F1E8 送入 EDX`

`004CA846    8B45 08         MOV     EAX,DWORD PTR SS:[EBP+8]        ; EAX=假码`

`004CA849    E8 02E6F3FF     CALL    ItManage.00408E50                ; 获取假码`

`004CA84E    8B55 F4         MOV     EDX,DWORD PTR SS:[EBP-C]        ; EDX=假码`

`004CA851    8D45 08         LEA     EAX,DWORD PTR SS:[EBP+8]        ; 将地址 0012F1FC 送入 EAX`

`004CA854    E8 33A0F3FF     CALL    ItManage.0040488C                ; 将假码地址和 01011750 交换`

`004CA859    8D4D F0         LEA     ECX,DWORD PTR SS:[EBP-10]       ; 将地址 0012F1E4 送入 ECX`

`004CA85C    8B55 FC         MOV     EDX,DWORD PTR SS:[EBP-4]        ; EDX=-399410063`

`004CA85F    8BC3            MOV     EAX,EBX                          ; EAX=EBX`

`004CA861    E8 72FCFFFF     CALL    ItManage.004CA4D8                ; ***关键 CALL 跟进____-399410063 运算成真码跟进***`

`004CA866    8B45 F0         MOV     EAX,DWORD PTR SS:[EBP-10]       ; EAX=02078BD9B8FA`

`004CA869    8B55 08         MOV     EDX,DWORD PTR SS:[EBP+8]        ; EDX=假码`

`004CA86C    E8 57E6F3FF     CALL    ItManage.00408EC8                ; 真假码对比 CALL`

`004CA871    85C0            TEST    EAX,EAX                          ; 测试 EAX`

`004CA873    74 04           JE      SHORT ItManage.004CA879         ; 不相等则 OVER!相等则写入注册表信息!`

`004CA875    33DB            XOR     EBX,EBX`

`004CA877    EB 2A           JMP     SHORT ItManage.004CA8A3`

`004CA879    8D43 48         LEA     EAX,DWORD PTR DS:[EBX+48]`

`004CA87C    8B55 FC         MOV     EDX,DWORD PTR SS:[EBP-4]`

`004CA87F    E8 C49FF3FF     CALL    ItManage.00404848`

`004CA884    8D43 54         LEA     EAX,DWORD PTR DS:[EBX+54]`

`004CA887    8B55 F8         MOV     EDX,DWORD PTR SS:[EBP-8]`

`004CA88A    E8 B99FF3FF     CALL    ItManage.00404848`

`004CA88F    8D43 5C         LEA     EAX,DWORD PTR DS:[EBX+5C]`

`004CA892    8B55 08         MOV     EDX,DWORD PTR SS:[EBP+8]`

`004CA895    E8 AE9FF3FF     CALL    ItManage.00404848`

`004CA89A    8BC3            MOV     EAX,EBX`

`004CA89C    E8 B3010000     CALL    ItManage.004CAA54                ; 这个 CALL 作用是把注册成功后写入注册表信息!!`

`004CA8A1    B3 01           MOV     BL,1                             ; 注册标志将 1 送 BL`

`004CA8A3    33C0            XOR     EAX,EAX`

`004CA8A5    5A              POP     EDX`

`004CA8A6    59              POP     ECX`

`004CA8A7    59              POP     ECX`

`004CA8A8    64:8910         MOV     DWORD PTR FS:[EAX],EDX`

`004CA8AB    68 CDA84C00     PUSH    ItManage.004CA8CD`

`004CA8B0    8D45 F0         LEA     EAX,DWORD PTR SS:[EBP-10]`

`004CA8B3    BA 04000000     MOV     EDX,4`

`004CA8B8    E8 5B9FF3FF     CALL    ItManage.00404818`

`004CA8BD    8D45 08         LEA     EAX,DWORD PTR SS:[EBP+8]`

`004CA8C0    E8 2F9FF3FF     CALL    ItManage.004047F4`

`004CA8C5    C3              RETN`

---

## 跟进 004CA4D8

`004CA4D8    55              PUSH    EBP`

`004CA4D9    8BEC            MOV     EBP,ESP`

`004CA4DB    83C4 E4         ADD     ESP,-1C`

`004CA4DE    53              PUSH    EBX`

`004CA4DF    56              PUSH    ESI`

`004CA4E0    57              PUSH    EDI`

`004CA4E1    33DB            XOR     EBX,EBX`

`004CA4E3    895D E4         MOV     DWORD PTR SS:[EBP-1C],EBX      ; 0012F1B0=0`

`004CA4E6    895D F4         MOV     DWORD PTR SS:[EBP-C],EBX       ; 0012F1C0=0`

`004CA4E9    8BF9            MOV     EDI,ECX                          ; EDI=0012F1E4`

`004CA4EB    8955 FC         MOV     DWORD PTR SS:[EBP-4],EDX       ; 保存到 0012F1C8=-399410063`

`004CA4EE    8BF0            MOV     ESI,EAX                          ; ESI=EAX`

`004CA4F0    8B45 FC         MOV     EAX,DWORD PTR SS:[EBP-4]        ; EAX=-399410063`

`004CA4F3    E8 9CA7F3FF     CALL    ItManage.00404C94`

`004CA4F8    33C0            XOR     EAX,EAX                          ; EAX=0`

`004CA4FA    55              PUSH    EBP`

`004CA4FB    68 F1A54C00     PUSH    ItManage.004CA5F1`

`004CA500    64:FF30         PUSH    DWORD PTR FS:[EAX]`

`004CA503    64:8920         MOV     DWORD PTR FS:[EAX],ESP`

`004CA506    8B45 FC         MOV     EAX,DWORD PTR SS:[EBP-4]        ; EAX=-399410063`

`004CA509    E8 9EA5F3FF     CALL    ItManage.00404AAC                ; 获取-399410063 的长度，并送入 EAX=A`

`004CA50E    3B46 4C         CMP     EAX,DWORD PTR DS:[ESI+4C]      ; A 和 19 做比较`

`004CA511    7F 0D           JG      SHORT ItManage.004CA520         ; 大于就跳走`

`004CA513    8B45 FC         MOV     EAX,DWORD PTR SS:[EBP-4]        ; EAX=-399410063`

`004CA516    E8 91A5F3FF     CALL    ItManage.00404AAC                ; 再次获取-399410063 的长度，并送入 EAX=A`

`004CA51B    3B46 50         CMP     EAX,DWORD PTR DS:[ESI+50]      ; A 和 3 做比较`

`004CA51E    7D 0C           JGE     SHORT ItManage.004CA52C         ; 大于或等于则跳走`

`004CA520    8BC7            MOV     EAX,EDI`

`004CA522    E8 CDA2F3FF     CALL    ItManage.004047F4`

`004CA527    E9 9F000000     JMP     ItManage.004CA5CB`

`004CA52C    8B45 FC         MOV     EAX,DWORD PTR SS:[EBP-4]        ; EAX=-399410063`

`004CA52F    E8 78A5F3FF     CALL    ItManage.00404AAC                ; 获取-399410063 的长度，并送入 EAX=A`

`004CA534    8BD8            MOV     EBX,EAX                          ; EBX=EAX=A`

`004CA536    EB 31           JMP     SHORT ItManage.004CA569         ; 无条件跳走`

`004CA538    8B45 FC         MOV     EAX,DWORD PTR SS:[EBP-4]        ; EAX=-399410063`

`004CA53B    8A4418 FF       MOV     AL,BYTE PTR DS:[EAX+EBX-1]     ; 从假码最后一位从右到左取出假码 ASCII 码送入 AL`

`004CA53F    25 FF000000     AND     EAX,0FF                            ; EAX 和 OFF 做 AND 运算`

`004CA544    33D2            XOR     EDX,EDX                          ; EDX 清 0`

`004CA546    52              PUSH    EDX                              ; 压入 EDX`

`004CA547    50              PUSH    EAX                              ; 压入 EAX`

`004CA548    8B46 68         MOV     EAX,DWORD PTR DS:[ESI+68]      ; EAX=573EF7BB`

`004CA54B    8B56 6C         MOV     EDX,DWORD PTR DS:[ESI+6C]`

`004CA54E    E8 C5B3F3FF     CALL    ItManage.00405918                ; 这个 CALL 的作用是 573EF7BB 除以 399410063(-不参以运算)逐位 ASCII 码，并取余数送入 EAX。`

`004CA553    52              PUSH    EDX                              ; 压入 EDX`

`004CA554    50              PUSH    EAX                              ; 压入 EAX`

`004CA555    8D45 E4         LEA     EAX,DWORD PTR SS:[EBP-1C]      ; 将地址 0012F1B0 送入 EAX`

`004CA558    E8 AFEEF3FF     CALL    ItManage.0040940C                ; 这个 CALL 作用是***从上面得的余数除 A 取余+30；除 A 取商再除 A 取余+30；`

`004CA55D    8B55 E4         MOV     EDX,DWORD PTR SS:[EBP-1C]`

`004CA560    8D45 F4         LEA     EAX,DWORD PTR SS:[EBP-C]`

`004CA563    E8 4CA5F3FF     CALL    ItManage.00404AB4                ; 合并上面 CALL 计算的值到 00FF9164 里面`

`004CA568    4B              DEC     EBX                              ; 计数器 EBX-1`

`004CA569    8B45 FC         MOV     EAX,DWORD PTR SS:[EBP-4]        ; EAX=-399410063`

`004CA56C    E8 3BA5F3FF     CALL    ItManage.00404AAC                ; 获取-399410063 的长度，并送入 EAX=A`



```asm
004CA571    83E8 06         SUB     EAX,6                           ; EAX=-399410063 的长度 A-6=4
004CA574    3BD8            CMP     EBX,EAX                         ; EBX(-399410063 的长度 A)和 EAX(从上面 A-6 得值)做比较
004CA576    7C 04           JL      SHORT ItManage.004CA57C         ; 小于就跳
004CA578    85DB            TEST    EBX,EBX                         ; 测试 EBX
004CA57A    7F BC           JG      SHORT ItManage.004CA538         ; 大于就跳
004CA57C    8D55 F8         LEA     EDX,DWORD PTR SS:[EBP-8]
004CA57F    8B45 F4         MOV     EAX,DWORD PTR SS:[EBP-C]        ; EAX=2231434328314
004CA582    E8 79B5F3FF     CALL    ItManage.00405B00
004CA587    8945 E8         MOV     DWORD PTR SS:[EBP-18],EAX
004CA58A    8955 EC         MOV     DWORD PTR SS:[EBP-14],EDX
004CA58D    8B5E 60         MOV     EBX,DWORD PTR DS:[ESI+60]
004CA590    85DB            TEST    EBX,EBX
004CA592    7F 11           JG      SHORT ItManage.004CA5A5
004CA594    FF75 EC         PUSH    DWORD PTR SS:[EBP-14]
004CA597    FF75 E8         PUSH    DWORD PTR SS:[EBP-18]
004CA59A    8BD7            MOV     EDX,EDI
004CA59C    33C0            XOR     EAX,EAX
004CA59E    E8 D9EEF3FF     CALL    ItManage.0040947C
004CA5A3    EB 26           JMP     SHORT ItManage.004CA5CB
004CA5A5    FF75 EC         PUSH    DWORD PTR SS:[EBP-14]
004CA5A8    FF75 E8         PUSH    DWORD PTR SS:[EBP-18]
004CA5AB    8BD7            MOV     EDX,EDI
004CA5AD    8BC3            MOV     EAX,EBX
004CA5AF    E8 C8EEF3FF     CALL    ItManage.0040947C               ; 这个 CALL 做将 2231434328314 转换成 16 进制
004CA5B4    8B07            MOV     EAX,DWORD PTR DS:[EDI]          ; 从上面返加到 EAX=02078BD9B8FA 这个就是真码了!。。做内存注册机吧!
004CA5B6    E8 F1A4F3FF     CALL    ItManage.00404AAC               ; 获取真码长度 C
004CA5BB    8BC8            MOV     ECX,EAX
004CA5BD    2B4E 60         SUB     ECX,DWORD PTR DS:[ESI+60]
004CA5C0    8B56 60         MOV     EDX,DWORD PTR DS:[ESI+60]
004CA5C3    42              INC     EDX
004CA5C4    8BC7            MOV     EAX,EDI
004CA5C6    E8 79A7F3FF     CALL    ItManage.00404D44
004CA5CB    33C0            XOR     EAX,EAX
004CA5CD    5A              POP     EDX
004CA5CE    59              POP     ECX
004CA5CF    59              POP     ECX
004CA5D0    64:8910         MOV     DWORD PTR FS:[EAX],EDX
004CA5D3    68 F8A54C00     PUSH    ItManage.004CA5F8
004CA5D8    8D45 E4         LEA     EAX,DWORD PTR SS:[EBP-1C]
004CA5DB    E8 14A2F3FF     CALL    ItManage.004047F4
004CA5E0    8D45 F4         LEA     EAX,DWORD PTR SS:[EBP-C]
004CA5E3    E8 0CA2F3FF     CALL    ItManage.004047F4
004CA5E8    8D45 FC         LEA     EAX,DWORD PTR SS:[EBP-4]
004CA5EB    E8 04A2F3FF     CALL    ItManage.004047F4
004CA5F0    C3              RETN
```

## 总结注册分析流程

1. 首先获取 `C` 盘分区序列号。(我的 `C` 盘分区序列号是 `E317C71`)

2. 将分区序列号 `E317C71` 取反除以 `A`，每次除 `A` 得出的余数和 `30` 相加，除完得出的值和 `2D(-)` 合并。(我的得出值 `-399410063`)

3. 得到 `-399410063` 后，从倒数最后一位(从右到左顺序)逐个取出 `ASCII` 码除以 `573EF7BB`，只取余数!
   1. 从上面得到余数再除以 `A`，只取余数和 `30` 相加。
   2. 从上面得到余数再除以 `A` 只取商。取得商除以 `A` 只取余数，余数和 `30` 相加。
   3. `-399410063` 每个数经上面 2 步得出的值(当然是 `ASCII` 码的)合并转换成 `10` 进制数。`“-”` 符号不参予运算

4. 最后得值为 `2231434328314`，转换成 `16` 制进就是注册码了!

5. 输入硬盘 `ID` 这栏，输不输入无所谓了作者用来做障眼法的呵呵 。。硬盘 `ID` 和输入的假码都没有参予运算，真码对比可以内存注册机!^\_^

6. 再注册表注册成功的信息(我机器的注册表信息)：

   ```reg
   [HKEY_LOCAL_MACHINE\SOFTWARE\Classes\CLSID\{0C45C661-6005-11D5-8FF3-0040D005E5DC}\Info]
   "Data"=hex:0a,2d,33,39,39,34,31,30,30,36,33,00,00,00,00,00,2c,fd,01,01,0e,04,\
      00,00,a0,fc,12,00,18,ee,92,7c,5c,fd,12,00,a4,ba,d1,77,11,a9,00,00,11,a9,00,\
      00,70,fd,12,00,46,ba,d1,77,98,d9,93,00,11,a9,00,00,00,00,00,00,88,fd,12,00,\
      0a,bb,d1,77,98,d9,93,00,8c,fd,12,00,00,00,00,00,c8,05,93,7c,98,7d,15,00,58,\
      fe,12,00,a0,11,77,77,77,2e,6c,75,63,6b,67,75,65,73,74,2e,63,6f,6d,00,51,05,\
      93,7c,78,07,14,00,6d,05,93,7c,24,46,ff,00,74,96,ff,00,24,46,ff,00,00,00,00,\
      00,64,68,42,00,06,49,6d,61,24,46,ff,00,b8,91,ff,00,24,46,ff,00,00,00,00,00,\
      64,68,42,00,06,50,61,6e,65,6c,31,00,54,da,92,7c,5f,9b,80,7c,ff,ff,ff,ff,10,\
      fe,12,00,14,fe,0c,30,32,30,37,38,42,44,39,42,38,46,41,00,1c,fe,12,00,29,9b,\
      80,7c,28,00,00,00,00,c0,01,01,00,40,00,00,4c,fe,12,00,f8,55,55,00,00,00,14,\
      00,54,fe,12,00,ff,ff,ff,ff,00,00,00,00,00,00,0f,01,30,ae,ff,00,78,64,42,00,\
      30,ae,ff,00,01,96,ff,00,9b,64,42,00,01,96,ff,00,30,ae,ff,00,78,64,42,00,30,\
      ae,ff,00,01,91,ff,00,9b,64,42,00,01,91,ff,00,bc,ca,45,00,b8,91,ff,00,30,ae,\
      ff,00,01,91,ff,00,b8,91,ff,00,78,64,42,00,b8,91,ff,00,01,91,ff,00,9b,64,42,\
      00,01,91,ff,00,08,27,44,00,b8,91,ff,00,b8,91,ff,00,01,91,ff,00,78,d5,44,00,\
      b8,91,ff,00,24,46,ff,00,00,00,00,00,be,64,42,00,01,91,ff,00,78,ff,12,00,b8,\
      91,ff,00,58,91,ff,00,04,32,2e,30,30,00,00,00,29,25,40,00,00,00,00,00,6c,2e,\
      ff,00,40,2e,ff,00,2b,00,00,00,00,00,00,00,00,00,00,00,ec,50,ff,00,2b,00,00,\
      00
   ```

   把注册表信息删了就变成未注册版了!

7. 本人菜鸟一个不会编程所经不会写注册机(我烦哦!)准备啃编程书，没有多少笔墨文风不好请 `XD` 们不要见笑。这是我在看雪发的第 `3` 篇破文希望能和大大学习,

   由于本人原因学习破解时间断断续续所以学技不精菜鸟一个。

2. 将分区序列号 `E317C71` 取反除以 `A`，每次除 `A` 得出的余数和 `30` 相加，除完得出的值和 `2D(-)` 合并。(我的得出值 `-399410063`)

   除以 `A` 再和 `30` 相加，实际上就是把十六进制转化为十进制而已

3. 得到 `-399410063` 后，从倒数最后一位(从右到左顺序)逐个取出 `ASCII` 码除以 `573EF7BB`，只取余数!

   看不太懂，一位的 `ASCII` 码，除以 `573EF7BB` 这么大的数？

   又看了一下，上面写的好像是 `573EF7BB` 除以 `ASCII` 码吧？楼主写反了

   谢谢波兄指点。。看得眼花弄混了。。真不好意思。呵呵 。

`4dd466d0e1e0cc123afbe504da2462ca`
