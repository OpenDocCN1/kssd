
# 体彩霸主 2.8 的调试（浅谈 DOS 下全屏模式软件的调试方法）

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-477.htm`](https://www.pediy.com/kssd/pediy07/pediy7-477.htm)

【破解作者】 CCdebuger

【使用工具】 `TRW2000` 1.23

【破解平台】 Win98

【软件名称】 体彩霸主 2.8

--------------------------------------------------------------------------------

## 【破解内容】

这个软件是两年前应一个朋友的要求破的，现在版本好像是 3.0 了，发 2.8 版的破解应该对它没影响吧？主要是在论坛上看到 acafeel 兄弟谈到此类程序的破解，感觉这类程序还是有一定的特殊性（DOS 下全屏运行，独占模式，无法用 OllyDBG 这类 Ring-3 级调试器来跟踪）。现在论坛上很多兄弟刚接触的破解工具就是 OllyDBG，对 `TRW`、`SoftICE` 缺乏了解，也算是顺便补补课吧。主要是当时调试的时候笔记并不详细，写的比较乱，现在又不大想再看 `TRW` 那张黑脸，大家主要是看看调试这类程序的方法。

启动 `TRW2000`，运行体彩霸主，密码中第一个为 5 其余 11 个数随便输入(都是数字)，回车，显示密码错误。不要关闭体彩霸主，切换出其 DOS 窗口，运行 `WINHEX`，选定体彩霸主所在内存，查找你刚才随便输的数，找到后向上看，可看到如‘00000XXXXXX’的一组数字，把后六位记下来备用。重新启动体彩霸主，输入密码，前七位为 `5XXXXXX`（此处的 `XXXXXX` 就是刚才在 `WINHEX` 中看到的数），后面随便输，按 `Ctrl＋M`（Ring-0 级调试，`Ctrl+N` 的 Ring-3 级无效）调出 `TRW`，输入命令：`S 0 L FFFFFFFF '5XXXXXX'`，回车，得到你输入密码的内存地址，下 `BPM XXXX R|W`（此处 `XXXX` 为输入密码内存地址），`F5` 继续运行体彩霸主，回车（若是用 `TRW`，按回车没反应时请先按一下 `CTRL` 键），被 `TRW` 断下：

```
5C6C:1341 8D36C015         LEA      SI,[15C0]
5C6C:1345 803C35           CMP      BYTE [SI],35    ;比较第一个数是不是“5”
5C6C:1348 7503             JNZ      134D             (NO JUMP)  ;输入注册码按回车后中断的地方
5C6C:134A EB0B             JMP      SHORT 1357
5C6C:134C 90               NOP
```

先按 `F12` 从这个 CALL 里返回，在 `TRW2000` 的反汇编窗口中往上翻，在下面的 `08B6` 这一行上设个断点，重新启动体彩霸主，输入试炼码，下面开始分析：

```
5C6C:08B6 58               POP      AX
5C6C:08B7 5A               POP      DX
5C6C:08B8 1F               POP      DS
5C6C:08B9 2EC606BC1526     MOV      BYTE [CS:15BC],26
5C6C:08BF 90               NOP
5C6C:08C0 2EC606BE150C     MOV      BYTE [CS:15BE],0C
5C6C:08C6 90               NOP
5C6C:08C7 E8F30B           CALL     14BD
5C6C:08CA 2E803EBF150C     CMP      BYTE [CS:15BF],0C    比较密码是否为 12 位
5C6C:08D0 7512             JNZ      08E4      不等则出错
5C6C:08D2 E8620A           CALL     1337      跟进到下面的 1341 行处理
5C6C:08D5 2EC706BE120200   MOV      WORD [CS:12BE],02
5C6C:08DC E88705           CALL     0E66      跟进到下面的 0E66 行处理
5C6C:08DF B8014C           MOV      AX,4C01
5C6C:08E2 CD21             INT      21        密码检测完后的显示结果
5C6C:1341 8D36C015         LEA      SI,[15C0]      把输入的密码送到 SI
5C6C:1345 803C35           CMP      BYTE [SI],35    比较第一个数是不是 5
5C6C:1348 7503             JNZ      134D      不等则出错
5C6C:134A EB0B             JMP      SHORT 1357
5C6C:134C 90               NOP
5C6C:134D 2EC606A61300     MOV      BYTE [CS:13A6],00
5C6C:1353 90               NOP
5C6C:1354 EB47             JMP      SHORT 139D
5C6C:1356 90               NOP
5C6C:1357 BB0000           MOV      BX,00
5C6C:135A B90700           MOV      CX,07
5C6C:135D AC               LODSB           把输入密码的 ASCII 码一个个送到 AL
5C6C:135E 2C30             SUB      AL,30      输入密码的每位数字
5C6C:1360 02F8             ADD      BH,AL
5C6C:1362 32D8             XOR      BL,AL
5C6C:1364 E2F7             LOOP     135D      如果 CX 不等于 0 则再到 135D 处理，即取输入的前 8 个密码。
5C6C:1366 8D36C015         LEA      SI,[15C0]      再把输入的密码送到 SI
5C6C:136A 80FF09           CMP      BH,09      处理 BX 高位
5C6C:136D 7605             JNA      1374
5C6C:136F 80EF0A           SUB      BH,0A
5C6C:1372 EBF6             JMP      SHORT 136A
5C6C:1374 80C730           ADD      BH,30
5C6C:1377 8AC7             MOV      AL,BH      把处理后的 ASCII 码送到 AL
5C6C:1379 3A4407           CMP      AL,[SI+07]      同输入密码的第 8 位进行比较
5C6C:137C 7402             JZ       1380      不等则错
5C6C:137E EBCD             JMP      SHORT 134D
5C6C:1380 80FB09           CMP      BL,09      处理 BX 低位
5C6C:1383 7605             JNA      138A
5C6C:1385 80EB0A           SUB      BL,0A
5C6C:1388 EBF6             JMP      SHORT 1380
5C6C:138A 80C330           ADD      BL,30
5C6C:138D 8AC3             MOV      AL,BL
5C6C:138F 3A4408           CMP      AL,[SI+08]      把处理后的 BX 低位送 AL，同输入密码的第 9 位比较
5C6C:1392 7402             JZ       1396
5C6C:1394 EBB7             JMP      SHORT 134D
5C6C:1396 2EC606A61301     MOV      BYTE [CS:13A6],01
5C6C:139C 90               NOP
5C6C:139D 5A               POP      DX
5C6C:139E 59               POP      CX
5C6C:139F 5B               POP      BX
5C6C:13A0 58               POP      AX
5C6C:13A1 5F               POP      DI
5C6C:13A2 5E               POP      SI
5C6C:13A3 07               POP      ES
5C6C:13A4 1F               POP      DS
5C6C:13A5 C3               RET
5C6C:0E66 2E833EBE1201     CMP      WORD [CS:12BE],BYTE +01
5C6C:0E6C 7410             JZ       0E7E
5C6C:0E6E 2E833EBE1202     CMP      WORD [CS:12BE],BYTE +02
5C6C:0E74 740E             JZ       0E84
5C6C:0E76 2E833EBE1203     CMP      WORD [CS:12BE],BYTE +03
5C6C:0E7C 740C             JZ       0E8A
5C6C:0E7E B80000           MOV      AX,00
5C6C:0E81 EB0D             JMP      SHORT 0E90
5C6C:0E83 90               NOP
5C6C:0E84 B88509           MOV      AX,0985      送 0985H 到 AX
5C6C:0E87 EB07             JMP      SHORT 0E90
5C6C:0E89 90               NOP
5C6C:0E8A B80A13           MOV      AX,130A
5C6C:0E8D EB01             JMP      SHORT 0E90
5C6C:0E8F 90               NOP
5C6C:0E90 FC               CLD
5C6C:0E91 BB3412           MOV      BX,1234      1234H 到 BX
5C6C:0E94 BA2340           MOV      DX,4023      4023H 到 DX
5C6C:0E97 B9F0FF           MOV      CX,FFF0      FFF0H 到 CX，设置循环次数
5C6C:0E9A 058509           ADD      AX,0985
5C6C:0E9D 51               PUSH     CX
5C6C:0E9E D1C8             ROR      AX,1
5C6C:0EA0 03C1             ADD      AX,CX
5C6C:0EA2 33D9             XOR      BX,CX
5C6C:0EA4 D1CB             ROR      BX,1
5C6C:0EA6 03D0             ADD      DX,AX
5C6C:0EA8 F7D1             NOT      CX
5C6C:0EAA D3C2             ROL      DX,CL
5C6C:0EAC 33C2             XOR      AX,DX
5C6C:0EAE 03D8             ADD      BX,AX
5C6C:0EB0 2EFF06BC12       INC      WORD [CS:12BC]    把 12BC 的值加 1，在我机器中初始值为 07B0
5C6C:0EB5 81FBAC96         CMP      BX,96AC      把 BX 同表中值一一比较
5C6C:0EB9 7503             JNZ      0EBE
5C6C:0EBB E9BDFD           JMP      0C7B
5C6C:0EBE 81FBAB97         CMP      BX,97AB
5C6C:0EC2 7503             JNZ      0EC7
5C6C:0EC4 E97E0B           JMP      1A45
5C6C:0EC7 81FB8898         CMP      BX,9888
5C6C:0ECB 7503             JNZ      0ED0
5C6C:0ECD E9A713           JMP      2277
5C6C:0ED0 81FB7699         CMP      BX,9976
5C6C:0ED4 7503             JNZ      0ED9
5C6C:0ED6 E99E13           JMP      2277
5C6C:0ED9 81FB209A         CMP      BX,9A20
5C6C:0EDD 7503             JNZ      0EE2
5C6C:0EDF E9AA13           JMP      228C
5C6C:0EE2 81FB726A         CMP      BX,6A72
5C6C:0EE6 7503             JNZ      0EEB
5C6C:0EE8 E9B916           JMP      25A4
5C6C:0EEB 81FBB96A         CMP      BX,6AB9
5C6C:0EEF 7503             JNZ      0EF4
5C6C:0EF1 E9ADFE           JMP      0DA1
5C6C:0EF4 81FB4F6B         CMP      BX,6B4F
5C6C:0EF8 7503             JNZ      0EFD
5C6C:0EFA E9B916           JMP      25B6
5C6C:0EFD 81FB1B6C         CMP      BX,6C1B
5C6C:0F01 7503             JNZ      0F06
5C6C:0F03 E9B016           JMP      25B6
5C6C:0F06 81FB2B6D         CMP      BX,6D2B
5C6C:0F0A 7503             JNZ      0F0F
5C6C:0F0C E9A716           JMP      25B6
5C6C:0F0F 81FB4C6E         CMP      BX,6E4C
5C6C:0F13 7503             JNZ      0F18
5C6C:0F15 E91517           JMP      262D
5C6C:0F18 81FB3A6F         CMP      BX,6F3A
5C6C:0F1C 7503             JNZ      0F21
5C6C:0F1E E96C19           JMP      288D
5C6C:0F21 81FB1770         CMP      BX,7017
5C6C:0F25 7503             JNZ      0F2A
5C6C:0F27 E9BA19           JMP      28E4
5C6C:0F2A 81FB529B         CMP      BX,9B52
5C6C:0F2E 7503             JNZ      0F33
5C6C:0F30 E9B119           JMP      28E4
```



```
5C6C:0F33	81FB1E9C		CMP		BX,9C1E  
5C6C:0F37	7503		JNZ		0F3C  
5C6C:0F39	E96AF4		JMP		03A6  
5C6C:0F3C	81FBD99C		CMP		BX,9CD9  
5C6C:0F40	7503		JNZ		0F45  
5C6C:0F42	E9C919		JMP		290E  
5C6C:0F45	81FBC79D		CMP		BX,9DC7  
5C6C:0F49	7503		JNZ		0F4E  
5C6C:0F4B	E9861A		JMP		29D4  
5C6C:0F4E	81FB939E		CMP		BX,9E93  
5C6C:0F52	7503		JNZ		0F57  
5C6C:0F54	E9121B		JMP		2A69  
5C6C:0F57	81FBC59F		CMP		BX,9FC5  
5C6C:0F5B	7503		JNZ		0F60  
5C6C:0F5D	E9851D		JMP		2CE5  
5C6C:0F60	81FBA2A0		CMP		BX,A0A2  
5C6C:0F64	7503		JNZ		0F69  
5C6C:0F66	E91C1D		JMP		2C85  
5C6C:0F69	81FBE56E		CMP		BX,6EE5  
5C6C:0F6D	7503		JNZ		0F72  
5C6C:0F6F	E9751D		JMP		2CE7  
5C6C:0F72	81FB8F6F		CMP		BX,6F8F  
5C6C:0F76	7503		JNZ		0F7B  
5C6C:0F78	E9CF1D		JMP		2D4A  
5C6C:0F7B	81FB6C70		CMP		BX,706C  
5C6C:0F7F	7503		JNZ		0F84  
5C6C:0F81	E9801E		JMP		2E04  
5C6C:0F84	81FB3871		CMP		BX,7138  
5C6C:0F88	7503		JNZ		0F8D  
5C6C:0F8A	E9491F		JMP		2ED6  
5C6C:0F8D	81FB3772		CMP		BX,7237  
5C6C:0F91	7503		JNZ		0F96  
5C6C:0F93	E9401F		JMP		2ED6  
5C6C:0F96	81FB1473		CMP		BX,7314  
5C6C:0F9A	7503		JNZ		0F9F  
5C6C:0F9C	E9681F		JMP		2F07  
5C6C:0F9F	81FB4674		CMP		BX,7446  
5C6C:0FA3	7503		JNZ		0FA8  
5C6C:0FA5	E96D1F		JMP		2F15  
5C6C:0FA8	81FB8678		CMP		BX,7886  
5C6C:0FAC	7503		JNZ		0FB1  
5C6C:0FAE	E9A120		JMP		3052  
5C6C:0FB1	81FBD578		CMP		BX,78D5  
5C6C:0FB5	7503		JNZ		0FBA  
5C6C:0FB7	E911FC		JMP		0BCB  
5C6C:0FBA	81FBEC78		CMP		BX,78EC  
5C6C:0FBE	7503		JNZ		0FC3  
5C6C:0FC0	E9F520		JMP		30B8  
5C6C:0FC3	81FB7B7F		CMP		BX,7F7B  
5C6C:0FC7	7503		JNZ		0FCC  
5C6C:0FC9	E92DFC		JMP		0BF9  
5C6C:0FCC	81FB7F7F		CMP		BX,7F7F  
5C6C:0FD0	7503		JNZ		0FD5  
5C6C:0FD2	E91D21		JMP		30F2  
5C6C:0FD5	81FBA17F		CMP		BX,7FA1  
5C6C:0FD9	7503		JNZ		0FDE  
5C6C:0FDB	E91421		JMP		30F2  
5C6C:0FDE	81FB197F		CMP		BX,7F19  
5C6C:0FE2	7503		JNZ		0FE7  
5C6C:0FE4	E91121		JMP		30F8  
5C6C:0FE7	81FBE74A		CMP		BX,4AE7  
5C6C:0FEB	7503		JNZ		0FF0  
5C6C:0FED	E90821		JMP		30F8  
5C6C:0FF0	81FBD54B		CMP		BX,4BD5  
5C6C:0FF4	7503		JNZ		0FF9  
5C6C:0FF6	E9FF20		JMP		30F8  
5C6C:0FF9	81FB6852		CMP		BX,5268  
5C6C:0FFD	7503		JNZ		1002  
5C6C:0FFF	E94621		JMP		3148  
5C6C:1002	81FB3453		CMP		BX,5334  
5C6C:1006	7503		JNZ		100B  
5C6C:1008	E93722		JMP		3242  
5C6C:100B	81FB3354		CMP		BX,5433  
5C6C:100F	7503		JNZ		1014  
5C6C:1011	E9E522		JMP		32F9  
5C6C:1014	81FB5158		CMP		BX,5851  
5C6C:1018	7503		JNZ		101D  
5C6C:101A	E9DC22		JMP		32F9  
5C6C:101D	81FB2F58		CMP		BX,582F  
5C6C:1021	7503		JNZ		1026  
5C6C:1023	E9D322		JMP		32F9  
5C6C:1026	81FB2D58		CMP		BX,582D  
5C6C:102A	7503		JNZ		102F  
5C6C:102C	E9C9F5		JMP		05F8  
5C6C:102F	81FB7358		CMP		BX,5873  
5C6C:1033	7503		JNZ		1038  
5C6C:1035	E9C122		JMP		32F9  
5C6C:1038	81FBE959		CMP		BX,59E9  
5C6C:103C	7503		JNZ		1041  
5C6C:103E	E9B822		JMP		32F9  
5C6C:1041	81FB3189		CMP		BX,8931  
5C6C:1045	7503		JNZ		104A  
5C6C:1047	E9AF22		JMP		32F9  
5C6C:104A	81FB8689		CMP		BX,8986  
5C6C:104E	7503		JNZ		1053  
5C6C:1050	E9A622		JMP		32F9  
5C6C:1053	81FBCA89		CMP		BX,89CA  
5C6C:1057	7503		JNZ		105C  
5C6C:1059	E99D22		JMP		32F9  
5C6C:105C	81FBED89		CMP		BX,89ED  
5C6C:1060	7503		JNZ		1065  
5C6C:1062	E916FC		JMP		0C7B  
5C6C:1065	81FB4DB1		CMP		BX,B14D  
5C6C:1069	7503		JNZ		106E  
5C6C:106B	E9B059		JMP		6A1E  
5C6C:106E	81FB4CB2		CMP		BX,B24C  
5C6C:1072	7503		JNZ		1077  
5C6C:1074	E91798		JMP		A88E  
5C6C:1077	81FB3AB3		CMP		BX,B33A  
5C6C:107B	7503		JNZ		1080  
5C6C:107D	E9198E		JMP		9E99  
5C6C:1080	81FB28B4		CMP		BX,B428  
5C6C:1084	7503		JNZ		1089  
5C6C:1086	E9758E		JMP		9EFE  
5C6C:1089	81FBE3B4		CMP		BX,B4E3  
5C6C:108D	7503		JNZ		1092  
5C6C:108F	E9C08E		JMP		9F52  
5C6C:1092	81FBD1B5		CMP		BX,B5D1  
5C6C:1096	7503		JNZ		109B  
5C6C:1098	E9EB8E		JMP		9F86  
5C6C:109B	81FBE1B6		CMP		BX,B6E1  
5C6C:109F	7503		JNZ		10A4  
5C6C:10A1	E9F08E		JMP		9F94  
5C6C:10A4	81FB4ED2		CMP		BX,D24E  
5C6C:10A8	7503		JNZ		10AD  
5C6C:10AA	E9278F		JMP		9FD4  
5C6C:10AD	81FB80D3		CMP		BX,D380  
5C6C:10B1	7503		JNZ		10B6  
5C6C:10B3	E9908F		JMP		A046  
5C6C:10B6	81FB6EA1		CMP		BX,A16E  
5C6C:10BA	7503		JNZ		10BF  
5C6C:10BC	E9C78F		JMP		A086  
5C6C:10BF	81FB7FA1		CMP		BX,A17F  
5C6C:10C3	7503		JNZ		10C8  
5C6C:10C5	E9E692		JMP		A3AE  
5C6C:10C8	81FB8BA1		CMP		BX,A18B  
5C6C:10CC	7503		JNZ		10D1  
5C6C:10CE	E92206		JMP		16F3  
5C6C:10D1	81FBB2A1		CMP		BX,A1B2  
5C6C:10D5	7503		JNZ		10DA  
5C6C:10D7	E93393		JMP		A40D  
5C6C:10DA	81FBCC43		CMP		BX,43CC  
5C6C:10DE	7503		JNZ		10E3  
5C6C:10E0	E9D493		JMP		A4B7  
5C6C:10E3	81FBA944		CMP		BX,44A9  
5C6C:10E7	7503		JNZ		10EC  
5C6C:10E9	E9BF95		JMP		A6AB  
5C6C:10EC	81FB6445		CMP		BX,4564  
5C6C:10F0	7503		JNZ		10F5  
5C6C:10F2	E96B98		JMP		A960  
5C6C:10F5	81FB8546		CMP		BX,4685  
5C6C:10F9	7503		JNZ		10FE  
5C6C:10FB	E9AB98		JMP		A9A9  
5C6C:10FE	81FBB747		CMP		BX,47B7  
5C6C:1102	7503		JNZ		1107  
5C6C:1104	E9DC98		JMP		A9E3  
5C6C:1107	81FB1C49		CMP		BX,491C  
5C6C:110B	7503		JNZ		1110  
5C6C:110D	E96F7C		JMP		8D7F  
5C6C:1110	81FB4E4A		CMP		BX,4A4E  
5C6C:1114	7503		JNZ		1119  
5C6C:1116	E9BC7C		JMP		8DD5  
5C6C:1119	81FB4D4B		CMP		BX,4B4D  
5C6C:111D	7503		JNZ		1122  
5C6C:111F	E9EA7C		JMP		8E0C  
5C6C:1122	81FB5D4C		CMP		BX,4C5D  
5C6C:1126	7503		JNZ		112B  
5C6C:1128	E92E7D		JMP		8E59  
5C6C:112B	81FBA03C		CMP		BX,3CA0  
5C6C:112F	7503		JNZ		1134  
5C6C:1131	E9737D		JMP		8EA7  
5C6C:1134	81FBB13C		CMP		BX,3CB1  
5C6C:1138	7503		JNZ		113D  
5C6C:113A	E9EC7D		JMP		8F29  
5C6C:113D	81FBF13C		CMP		BX,3CF1		我的机器上运算后为此值  
5C6C:1141	7503		JNZ		1146  
5C6C:1143	E9DBFC		JMP		0E21		跳到 0E21 行执行
```



```
5C6C:1146 81FB063D         CMP      BX,3D06  

5C6C:114A 7503             JNZ      114F  

5C6C:114C E9157E           JMP      8F64  

5C6C:114F 81FB8524         CMP      BX,2485  

5C6C:1153 7503             JNZ      1158  

5C6C:1155 E9DC7E           JMP      9034  

5C6C:1158 81FB5125         CMP      BX,2551  

5C6C:115C 7503             JNZ      1161  

5C6C:115E E9867F           JMP      90E7  

5C6C:1161 81FB8326         CMP      BX,2683  

5C6C:1165 7503             JNZ      116A  

5C6C:1167 E9DA7F           JMP      9144  

5C6C:116A 81FB9327         CMP      BX,2793  

5C6C:116E 7503             JNZ      1173  

5C6C:1170 E91D80           JMP      9190  

5C6C:1173 81FB3D28         CMP      BX,283D  

5C6C:1177 7503             JNZ      117C  

5C6C:1179 E93681           JMP      92B2  

5C6C:117C 81FB4D29         CMP      BX,294D  

5C6C:1180 7503             JNZ      1185  

5C6C:1182 E9D181           JMP      9356  

5C6C:1185 81FB192A         CMP      BX,2A19  

5C6C:1189 7503             JNZ      118E  

5C6C:118B E9FE81           JMP      938C  

5C6C:118E 81FB292B         CMP      BX,2B29  

5C6C:1192 7503             JNZ      1197  

5C6C:1194 E92982           JMP      93C0  

5C6C:1197 81FB172C         CMP      BX,2C17  

5C6C:119B 7503             JNZ      11A0  

5C6C:119D E9CC98           JMP      AA6C  

5C6C:11A0 81FB7C2D         CMP      BX,2D7C  

5C6C:11A4 7503             JNZ      11A9  

5C6C:11A6 E91C99           JMP      AAC5  

5C6C:11A9 81FB372E         CMP      BX,2E37  

5C6C:11AD 7503             JNZ      11B2  

5C6C:11AF E9EF99           JMP      ABA1  

5C6C:11B2 81FB252F         CMP      BX,2F25  

5C6C:11B6 7503             JNZ      11BB  

5C6C:11B8 E9799A           JMP      AC34  

5C6C:11BB 81FB0241         CMP      BX,4102  

5C6C:11BF 7503             JNZ      11C4  

5C6C:11C1 E9289B           JMP      ACEC  

5C6C:11C4 81FBBC42         CMP      BX,42BC  

5C6C:11C8 7503             JNZ      11CD  

5C6C:11CA E9859B           JMP      AD52  

5C6C:11CD 81FB7743         CMP      BX,4377  

5C6C:11D1 7503             JNZ      11D6  

5C6C:11D3 E9059C           JMP      ADDB  

5C6C:11D6 81FB8744         CMP      BX,4487  

5C6C:11DA 7503             JNZ      11DF  

5C6C:11DC E9449B           JMP      AD23  

5C6C:11DF 81FB3145         CMP      BX,4531  

5C6C:11E3 7503             JNZ      11E8  

5C6C:11E5 E9989A           JMP      AC80  

5C6C:11E8 81FB6346         CMP      BX,4663  

5C6C:11EC 7503             JNZ      11F1  

5C6C:11EE E97199           JMP      AB62  

5C6C:11F1 81FBFB25         CMP      BX,25FB  

5C6C:11F5 7503             JNZ      11FA  

5C6C:11F7 E9FB97           JMP      A9F5  

5C6C:11FA 81FB1C27         CMP      BX,271C  

5C6C:11FE 7503             JNZ      1203  

5C6C:1200 E9708F           JMP      A173  

5C6C:1203 81FBD727         CMP      BX,27D7  

5C6C:1207 7503             JNZ      120C  

5C6C:1209 E9E18F           JMP      A1ED  

5C6C:120C 81FB9228         CMP      BX,2892  

5C6C:1210 7503             JNZ      1215  

5C6C:1212 E90390           JMP      A218  

5C6C:1215 81FB9129         CMP      BX,2991  

5C6C:1219 7503             JNZ      121E  

5C6C:121B E9C090           JMP      A2DE  

5C6C:121E 81FBC32A         CMP      BX,2AC3  

5C6C:1222 7503             JNZ      1227  

5C6C:1224 E95191           JMP      A378  

5C6C:1227 81FBA02B         CMP      BX,2BA0  

5C6C:122B 7503             JNZ      1230  

5C6C:122D E9AF9B           JMP      ADDF  

5C6C:1230 81FB7D2C         CMP      BX,2C7D  

5C6C:1234 7503             JNZ      1239  

5C6C:1236 E9A69B           JMP      ADDF  

5C6C:1239 81FB3ED1         CMP      BX,D13E  

5C6C:123D 7503             JNZ      1242  

5C6C:123F E9629C           JMP      AEA4  

5C6C:1242 81FBF9D1         CMP      BX,D1F9  

5C6C:1246 7503             JNZ      124B  

5C6C:1248 E9479E           JMP      B092  

5C6C:124B 81FBF8D2         CMP      BX,D2F8  

5C6C:124F 7503             JNZ      1254  

5C6C:1251 E98AA0           JMP      B2DE  

5C6C:1254 81FBF7D3         CMP      BX,D3F7  

5C6C:1258 7503             JNZ      125D  

5C6C:125A E9F6A0           JMP      B353  

5C6C:125D 81FBC3D4         CMP      BX,D4C3  

5C6C:1261 7503             JNZ      1266  

5C6C:1263 E92EA1           JMP      B394  

5C6C:1266 81FBA0D5         CMP      BX,D5A0  

5C6C:126A 7503             JNZ      126F  

5C6C:126C E9C1A2           JMP      B530  

5C6C:126F 81FBD12D         CMP      BX,2DD1  

5C6C:1273 7503             JNZ      1278  

5C6C:1275 E955A4           JMP      B6CD  

5C6C:1278 81FBAE2E         CMP      BX,2EAE  

5C6C:127C 7503             JNZ      1281  

5C6C:127E E980A3           JMP      B601  

5C6C:1281 81FB692F         CMP      BX,2F69  

5C6C:1285 7503             JNZ      128A  

5C6C:1287 E9E1A6           JMP      B96B  

5C6C:128A 81FBBD30         CMP      BX,30BD  

5C6C:128E 7503             JNZ      1293  

5C6C:1290 E973C1           JMP      D406  

5C6C:1293 2E8B0EC012       MOV      CX,\[CS:12C0\]    把 12C0 内值送 CX，我机器中初始值为 50B8  

5C6C:1298 2E330EC212       XOR      CX,\[CS:12C2\]    12C2 内值与 CX 异或后送 CX，我机器中初始值为 5443  

5C6C:129D 2E030EBC12       ADD      CX,\[CS:12BC\]  

5C6C:12A2 2AE9             SUB      CH,CL  

5C6C:12A4 2E330EBC12       XOR      CX,\[CS:12BC\]  

5C6C:12A9 2E890EC012       MOV      \[CS:12C0\],CX    再把处理后的 CX 内值送到 12C0  

5C6C:12AE E81300           CALL     12C4  

5C6C:12B1 59               POP      CX  

5C6C:12B2 49               DEC      CX        循环次数减 1  

5C6C:12B3 83F900           CMP      CX,BYTE +00  

5C6C:12B6 7403             JZ       12BB  

5C6C:12B8 E9E2FB           JMP      0E9D  

5C6C:12BB C3               RET       

\* Referenced by a CALL at Address:  

|5C6C:12AE  

|  

5C6C:12C4 2E833EBE1202           CMP WORD PTR CS:\[12BE\], 0002  

5C6C:12CA 7501                   JNE 12CD  

5C6C:12CC C3                     RET  

5C6C:0DAD 8D363809               LEA SI, \[0938\]      关键断点，把机器码的第三到第八位送到 SI  

5C6C:0DB1 E8F305                 CALL 13A7      进行运算，参见调用①  

5C6C:0DB4 8D369314               LEA SI, \[1493\]      第二到第七位的真注册码  

5C6C:0DB8 8D3EC115               LEA DI, \[15C1\]      假码  

5C6C:0DBC B90700                 MOV CX, 0007  

5C6C:0DBF F3                     REPZ  

5C6C:0DC0 A6                     CMPSB        比较  

5C6C:0DC1 E30A                   JCXZ 0DCD      相等则转到下一步  

5C6C:0DC3 2EC6065D0E00           MOV BYTE PTR CS:\[0E5D\], 00  

5C6C:0DC9 90                     NOP  

5C6C:0DCA EB4C                   JMP 0E18  

5C6C:0DCC 90                     NOP  

5C6C:0DCD 2EA1C012               MOV AX, WORD PTR CS:\[12C0\]  

5C6C:0DD1 2EA3C7EB               MOV WORD PTR CS:\[EBC7\], AX  

5C6C:0DD5 2EC706C5EB0000         MOV WORD PTR CS:\[EBC5\], 0000  

5C6C:0DDC E851A7                 CALL B530      进行运算，参见调用②  

5C6C:0DDF 8D36CDEB               LEA SI, \[EBCD\]  

5C6C:0DE3 2E833EC3EB03           CMP WORD PTR CS:\[EBC3\], 0003  

5C6C:0DE9 7311                   JNB 0DFC  

5C6C:0DEB C744FE3030             MOV WORD PTR \[SI-02\], 3030  

5C6C:0DF0 C644FD30               MOV BYTE PTR \[SI-03\], 30  

5C6C:0DF4 83EE03                 SUB SI, 0003  

5C6C:0DF7 2E0336C3EB             ADD SI, CS:\[EBC3\]  

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:  

|5C6C:0DE9(C)  

|  

5C6C:0DFC 8D3EC915               LEA DI, \[15C9\]      最后三位的假码送 DI，SI 内为真注册码  

5C6C:0E00 B90400                 MOV CX, 0004  

5C6C:0E03 F3                     REPZ  

5C6C:0E04 A6                     CMPSB        比较  

5C6C:0E05 E30A                   JCXZ 0E11      不等则出错  

5C6C:0E07 2EC6065D0E00           MOV BYTE PTR CS:\[0E5D\], 00  

5C6C:0E0D 90                     NOP  

5C6C:0E0E EB08                   JMP 0E18  

\* Referenced by a CALL at Addresses:        调用①  

|5C6C:0659, 5C6C:0DB1  

|  

5C6C:13A7 1E                     PUSH DS  

5C6C:13A8 06                     PUSH ES  

5C6C:13A9 56                     PUSH SI  
```



```
5C6C:13AA 57                     PUSH DI
5C6C:13AB 50                     PUSH AX
5C6C:13AC 53                     PUSH BX
5C6C:13AD 51                     PUSH CX
5C6C:13AE 52                     PUSH DX
5C6C:13AF 0E                     PUSH CS
5C6C:13B0 07                     POP ES
5C6C:13B1 8D3ECDEB               LEA DI, [EBCD]
5C6C:13B5 B90500                 MOV CX, 0005
5C6C:13B8 F3                     REPZ
5C6C:13B9 A4                     MOVSB
5C6C:13BA 0E                     PUSH CS
5C6C:13BB 1F                     POP DS
5C6C:13BC 2EC706C3EB0500         MOV WORD PTR CS:[EBC3], 0005
5C6C:13C3 E8CCA0                 CALL B492      参见调用③
5C6C:13C6 8D3E9A14               LEA DI, [149A]
5C6C:13CA B000                   MOV AL, 00
5C6C:13CC 2E833EC5EB00           CMP WORD PTR CS:[EBC5], 0000
5C6C:13D2 7402                   JE 13D6
5C6C:13D4 B001                   MOV AL, 01
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|5C6C:13D2(C)
|
5C6C:13D6 AA                     STOSB
5C6C:13D7 2E8B1EC7EB             MOV BX, CS:[EBC7]
5C6C:13DC B91000                 MOV CX, 0010
5C6C:13DF D1E3                   SHL BX, 01
5C6C:13E1 B001                   MOV AL, 01
5C6C:13E3 7202                   JB 13E7
5C6C:13E5 B000                   MOV AL, 00
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|5C6C:13E3(C)
|
5C6C:13E7 AA                     STOSB
5C6C:13E8 E2F5                   LOOP 13DF
5C6C:13EA 8D36AB14               LEA SI, [14AB]
5C6C:13EE B90900                 MOV CX, 0009
5C6C:13F1 AC                     LODSB
5C6C:13F2 B700                   MOV BH, 00
5C6C:13F4 B400                   MOV AH, 00
5C6C:13F6 8AD8                   MOV BL , AL
5C6C:13F8 AC                     LODSB
5C6C:13F9 8D3E9914               LEA DI, [1499]
5C6C:13FD 03FB                   ADD DI, BX
5C6C:13FF 268A35                 MOV DH, ES:[DI]
5C6C:1402 8D3E9914               LEA DI, [1499]
5C6C:1406 03F8                   ADD DI, AX
5C6C:1408 268A15                 MOV DL, ES:[DI]
5C6C:140B 268835                 MOV ES:[DI], DH
5C6C:140E 8D3E9914               LEA DI, [1499]
5C6C:1412 03FB                   ADD DI, BX
5C6C:1414 268815                 MOV ES:[DI], DL
5C6C:1417 E2D8                   LOOP 13F1
5C6C:1419 8D369A14               LEA SI, [149A]
5C6C:141D AC                     LODSB
5C6C:141E 2EC706C5EB0000         MOV WORD PTR CS:[EBC5], 0000
5C6C:1425 3C00                   CMP AL, 00
5C6C:1427 7407                   JE 1430
5C6C:1429 2EC706C5EB0100         MOV WORD PTR CS:[EBC5], 0001
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|5C6C:1427(C)
|
5C6C:1430 B91000                 MOV CX, 0010
5C6C:1433 BB0000                 MOV BX, 0000
5C6C:1436 F8                     CLC
5C6C:1437 AC                     LODSB
5C6C:1438 3C00                   CMP AL, 00
5C6C:143A 7401                   JE 143D
5C6C:143C F9                     STC
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|5C6C:143A(C)
|
5C6C:143D D1D3                   RCL BX, 01
5C6C:143F E2F5                   LOOP 1436
5C6C:1441 2E891EC7EB             MOV CS:[EBC7], BX
5C6C:1446 E8E7A0                 CALL B530      参见调用②
5C6C:1449 8D36CDEB               LEA SI, [EBCD]
5C6C:144D 8D3E9314               LEA DI, [1493]
5C6C:1451 2E833EC3EB00           CMP WORD PTR CS:[EBC3], 0000
5C6C:1457 750A                   JNE 1463
5C6C:1459 B90600                 MOV CX, 0006
5C6C:145C B030                   MOV AL, 30
5C6C:145E F3                     REPZ
5C6C:145F AA                     STOSB
5C6C:1460 EB23                   JMP 1485
5C6C:1462 90                     NOP
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|5C6C:1457(C)
|
5C6C:1463 B90600                 MOV CX, 0006
5C6C:1466 2E833EC3EB06           CMP WORD PTR CS:[EBC3], 0006
5C6C:146C 7607                   JBE 1475
5C6C:146E 2EC706C3EB0600         MOV WORD PTR CS:[EBC3], 0006
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|5C6C:146C(C)
|
5C6C:1475 2E2B0EC3EB             SUB CX, CS:[EBC3]
5C6C:147A B030                   MOV AL, 30
5C6C:147C F3                     REPZ
5C6C:147D AA                     STOSB
5C6C:147E 2E8B0EC3EB             MOV CX, CS:[EBC3]
5C6C:1483 F3                     REPZ
5C6C:1484 A4                     MOVSB
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|5C6C:1460(U)
|
5C6C:1485 5A                     POP DX
5C6C:1486 59                     POP CX
5C6C:1487 5B                     POP BX
5C6C:1488 58                     POP AX
5C6C:1489 5F                     POP DI
5C6C:148A 5E                     POP SI
5C6C:148B 07                     POP ES
5C6C:148C 1F                     POP DS
5C6C:148D C3                     RET
* Referenced by a CALL at Addresses:        调用②
|5C6C:04F9, 5C6C:0652, 5C6C:0DDC, 5C6C:1446, 5C6C:1803
|5C6C:2E26, 5C6C:2E3F, 5C6C:2E60, 5C6C:3B8E, 5C6C:489E
|5C6C:5453, 5C6C:5648, 5C6C:56AF, 5C6C:570A, 5C6C:5C1F
|5C6C:5F97, 5C6C:633E, 5C6C:66BE, 5C6C:84AF, 5C6C:88F6
|5C6C:9478, 5C6C:9526, 5C6C:9D77, 5C6C:9FA3, 5C6C:A979
|5C6C:AF51
|
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|5C6C:126C(U)
|
5C6C:B530 1E                     PUSH DS
5C6C:B531 06                     PUSH ES
5C6C:B532 50                     PUSH AX
5C6C:B533 53                     PUSH BX
5C6C:B534 51                     PUSH CX
5C6C:B535 52                     PUSH DX
5C6C:B536 56                     PUSH SI
5C6C:B537 57                     PUSH DI
5C6C:B538 2EA1C5EB               MOV AX, WORD PTR CS:[EBC5]
5C6C:B53C 2E0306C7EB             ADD AX, CS:[EBC7]
5C6C:B541 3D0000                 CMP AX, 0000
5C6C:B544 750A                   JNE B550
5C6C:B546 2EC706C3EB0000         MOV WORD PTR CS:[EBC3], 0000
5C6C:B54D E9A800                 JMP B5F8
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|5C6C:B544(C)
|
5C6C:B550 2E813EC5EB9800         CMP WORD PTR CS:[EBC5], 0098
5C6C:B557 7220                   JB B579
5C6C:B559 2E813EC5EB9800         CMP WORD PTR CS:[EBC5], 0098
5C6C:B560 7709                   JA B56B
5C6C:B562 2E813EC7EB7F96         CMP WORD PTR CS:[EBC7], 967F
5C6C:B569 760E                   JBE B579
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|5C6C:B560(C)
|
5C6C:B56B 2EC706C5EB9800         MOV WORD PTR CS:[EBC5], 0098
5C6C:B572 2EC706C7EB7F96         MOV WORD PTR CS:[EBC7], 967F
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|5C6C:B557(C), 5C6C:B569(C)
|
5C6C:B579 0E                     PUSH CS
5C6C:B57A 1F                     POP DS
5C6C:B57B 0E                     PUSH CS
5C6C:B57C 07                     POP ES
5C6C:B57D 2EC706C3EB0700         MOV WORD PTR CS:[EBC3], 0007
5C6C:B584 8D3ECDEB               LEA DI, [EBCD]
5C6C:B588 BE0000                 MOV SI, 0000
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|5C6C:B5CD(U)
|
5C6C:B58B 46                     INC SI
5C6C:B58C 2E8B0EC3EB             MOV CX, CS:[EBC3]
5C6C:B591 2BCE                   SUB CX, SI
5C6C:B593 83F900                 CMP CX, 0000
5C6C:B596 7437                   JE B5CF
5C6C:B598 B80100                 MOV AX, 0001
5C6C:B59B BB0A00                 MOV BX, 000A
5C6C:B59E F7E3                   MUL BX
5C6C:B5A0 E2F9                   LOOP B59B
5C6C:B5A2 83FA05                 CMP DX, 0005
```



`5C6C:B5A5` `7503` `JNE` `B5AA`

`5C6C:B5A7` `BA0F00` `MOV` `DX, 000F`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| `5C6C:B5A5(C)`

|

`5C6C:B5AA` `BB0000` `MOV` `BX, 0000`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| `5C6C:B5C8(U)`

|

`5C6C:B5AD` `2E3B16C5EB` `CMP` `DX, CS:[EBC5]`

`5C6C:B5B2` `7209` `JB` `B5BD`

`5C6C:B5B4` `7714` `JA` `B5CA`

`5C6C:B5B6` `2E3B06C7EB` `CMP` `AX, CS:[EBC7]`

`5C6C:B5BB` `770D` `JA` `B5CA`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| `5C6C:B5B2(C)`

|

`5C6C:B5BD` `2E2906C7EB` `SUB` `CS:[EBC7], AX`

`5C6C:B5C2` `2E1916C5EB` `SBB` `CS:[EBC5], DX`

`5C6C:B5C7` `43` `INC` `BX`

`5C6C:B5C8` `EBE3` `JMP` `B5AD`

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

| `5C6C:B5B4(C), 5C6C:B5BB(C)`

|

`5C6C:B5CA` `8BC3` `MOV` `AX, BX`

`5C6C:B5CC` `AA` `STOSB`

`5C6C:B5CD` `EBBC` `JMP` `B58B`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| `5C6C:B596(C)`

|

`5C6C:B5CF` `2EA1C7EB` `MOV` `AX, WORD PTR CS:[EBC7]`

`5C6C:B5D3` `AA` `STOSB`

`5C6C:B5D4` `8D3ECDEB` `LEA` `DI, [EBCD]`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| `5C6C:B5E5(U)`

|

`5C6C:B5D8` `268A05` `MOV` `AL , ES:[DI]`

`5C6C:B5DB` `3C00` `CMP` `AL, 00`

`5C6C:B5DD` `7508` `JNE` `B5E7`

`5C6C:B5DF` `2EFF0EC3EB` `DEC` `WORD PTR CS:[EBC3]`

`5C6C:B5E4` `47` `INC` `DI`

`5C6C:B5E5` `EBF1` `JMP` `B5D8`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| `5C6C:B5DD(C)`

|

`5C6C:B5E7` `2E8B0EC3EB` `MOV` `CX, CS:[EBC3]`

`5C6C:B5EC` `8BF7` `MOV` `SI, DI`

`5C6C:B5EE` `8D3ECDEB` `LEA` `DI, [EBCD]`

`5C6C:B5F2` `AC` `LODSB`

`5C6C:B5F3` `0430` `ADD` `AL, 30`

`5C6C:B5F5` `AA` `STOSB`

`5C6C:B5F6` `E2FA` `LOOP` `B5F2`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| `5C6C:B54D(U)`

|

`5C6C:B5F8` `5F` `POP` `DI`

`5C6C:B5F9` `5E` `POP` `SI`

`5C6C:B5FA` `5A` `POP` `DX`

`5C6C:B5FB` `59` `POP` `CX`

`5C6C:B5FC` `5B` `POP` `BX`

`5C6C:B5FD` `58` `POP` `AX`

`5C6C:B5FE` `07` `POP` `ES`

`5C6C:B5FF` `1F` `POP` `DS`

`5C6C:B600` `C3` `RET`

* Referenced by a CALL at Addresses: 调用③

| `5C6C:0670, 5C6C:0C95, 5C6C:13C3, 5C6C:2E8F, 5C6C:42A0`

| `5C6C:A104`

|

`5C6C:B492` `1E` `PUSH` `DS`

`5C6C:B493` `06` `PUSH` `ES`

`5C6C:B494` `50` `PUSH` `AX`

`5C6C:B495` `53` `PUSH` `BX`

`5C6C:B496` `51` `PUSH` `CX`

`5C6C:B497` `52` `PUSH` `DX`

`5C6C:B498` `56` `PUSH` `SI`

`5C6C:B499` `57` `PUSH` `DI`

`5C6C:B49A` `2E833EC3EB00` `CMP` `WORD PTR CS:[EBC3], 0000`

`5C6C:B4A0` `7511` `JNE` `B4B3`

`5C6C:B4A2` `2EC706C5EB0000` `MOV` `WORD PTR CS:[EBC5], 0000`

`5C6C:B4A9` `2EC706C7EB0000` `MOV` `WORD PTR CS:[EBC7], 0000`

`5C6C:B4B0` `EB75` `JMP` `B527`

`5C6C:B4B2` `90` `NOP`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| `5C6C:B4A0(C)`

|

`5C6C:B4B3` `0E` `PUSH` `CS`

`5C6C:B4B4` `1F` `POP` `DS`

`5C6C:B4B5` `8D36CDEB` `LEA` `SI, [EBCD]`

`5C6C:B4B9` `803C09` `CMP` `BYTE PTR [SI], 09`

`5C6C:B4BC` `7610` `JBE` `B4CE`

`5C6C:B4BE` `2E8B0EC3EB` `MOV` `CX, CS:[EBC3]`

`5C6C:B4C3` `56` `PUSH` `SI`

`5C6C:B4C4` `FC` `CLD`

`5C6C:B4C5` `AC` `LODSB`

`5C6C:B4C6` `2C30` `SUB` `AL, 30`

`5C6C:B4C8` `8844FF` `MOV` `[SI-01], AL`

`5C6C:B4CB` `E2F8` `LOOP` `B4C5`

`5C6C:B4CD` `5E` `POP` `SI`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| `5C6C:B4BC(C)`

|

`5C6C:B4CE` `2EC706C5EB0000` `MOV` `WORD PTR CS:[EBC5], 0000`

`5C6C:B4D5` `2EC706C7EB0000` `MOV` `WORD PTR CS:[EBC7], 0000`

`5C6C:B4DC` `BF0000` `MOV` `DI, 0000`

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

| `5C6C:B509(C), 5C6C:B517(U)`

|

`5C6C:B4DF` `47` `INC` `DI`

`5C6C:B4E0` `2E8B0EC3EB` `MOV` `CX, CS:[EBC3]`

`5C6C:B4E5` `2BCF` `SUB` `CX, DI`

`5C6C:B4E7` `83F900` `CMP` `CX, 0000`

`5C6C:B4EA` `742D` `JE` `B519`

`5C6C:B4EC` `B80100` `MOV` `AX, 0001`

`5C6C:B4EF` `BB0A00` `MOV` `BX, 000A`

`5C6C:B4F2` `F7E3` `MUL` `BX`

`5C6C:B4F4` `E2F9` `LOOP` `B4EF`

`5C6C:B4F6` `83FA05` `CMP` `DX, 0005`

`5C6C:B4F9` `7503` `JNE` `B4FE`

`5C6C:B4FB` `BA0F00` `MOV` `DX, 000F`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| `5C6C:B4F9(C)`

|

`5C6C:B4FE` `50` `PUSH` `AX`

`5C6C:B4FF` `B80000` `MOV` `AX, 0000`

`5C6C:B502` `AC` `LODSB`

`5C6C:B503` `8BC8` `MOV` `CX, AX`

`5C6C:B505` `58` `POP` `AX`

`5C6C:B506` `83F900` `CMP` `CX, 0000`

`5C6C:B509` `74D4` `JE` `B4DF`

`5C6C:B50B` `2E0106C7EB` `ADD` `CS:[EBC7], AX`

`5C6C:B510` `2E1116C5EB` `ADC` `CS:[EBC5], DX`

`5C6C:B515` `E2F4` `LOOP` `B50B`

`5C6C:B517` `EBC6` `JMP` `B4DF`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| `5C6C:B4EA(C)`

|

`5C6C:B519` `B400` `MOV` `AH, 00`

`5C6C:B51B` `AC` `LODSB`

`5C6C:B51C` `2E0106C7EB` `ADD` `CS:[EBC7], AX`

`5C6C:B521` `2E8316C5EB00` `ADC` `WORD PTR CS:[EBC5], 0000`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| `5C6C:B4B0(U)`

|

`5C6C:B527` `5F` `POP` `DI`

`5C6C:B528` `5E` `POP` `SI`

`5C6C:B529` `5A` `POP` `DX`

`5C6C:B52A` `59` `POP` `CX`

`5C6C:B52B` `5B` `POP` `BX`

`5C6C:B52C` `58` `POP` `AX`

`5C6C:B52D` `07` `POP` `ES`

`5C6C:B52E` `1F` `POP` `DS`

`5C6C:B52F` `C3` `RET`

--------------------------------------------------------------------------------

感谢您耐着性子看完！这篇文章只是作为技术交流，请勿把本文中的内容用于非法用途！

> 引用:

>* * *

>*最初由 wyuu 发布*

>**谢谢分享！！！学习下，老版了，新版不一样了，属过滤霸主进球霸主难分析哦！![](img/b16bec246e75299c87cf57e1fb808ac9.jpg) **

>* * *

不知 wyuu 兄弟说的进球霸主是什么，是不是双色球霸主？正好前天 acafeel 兄弟问到这个软件时，我去这个软件的网站上下载了一个最新版本，简单跟了一下，把跟踪结果作为邮件发给 acafeel 兄弟了。顺便也贴上来吧。我没做具体分析，有兴趣的自己跟。

几个关键的地方：

```
67D9:076B 2E803E7C130C     CMP      BYTE [CS:137C],0C    ;比较输入注册码的长度，下面的两个 CALL 很关键。
67D9:0771 7403             JZ       0776
67D9:0773 E9A200           JMP      0818
67D9:0776 8D367D13         LEA      SI,[137D]
67D9:077A E86805           CALL     0CE5
67D9:077D E8D50C           CALL     1455
67D9:0780 2E803E931501     CMP      BYTE [CS:1593],01
67D9:0786 7403             JZ       078B
```

输入注册码转换部分：

```
67D9:0CF9 51               PUSH     CX
67D9:0CFA AC               LODSB           ;载入表中数据
67D9:0CFB 56               PUSH     SI
67D9:0CFC 8AD8             MOV      BL,AL
67D9:0CFE B104             MOV      CL,04
67D9:0D00 D2EB             SHR      BL,CL
67D9:0D02 8BF2             MOV      SI,DX
67D9:0D04 03F3             ADD      SI,BX
67D9:0D06 240F             AND      AL,0F
67D9:0D08 8BFA             MOV      DI,DX
67D9:0D0A 03F8             ADD      DI,AX
67D9:0D0C 8A1C             MOV      BL,[SI]
67D9:0D0E 8A05             MOV      AL,[DI]
```



### 注册码转换相关汇编

```
67D9:0D10 8804             MOV      [SI],AL
67D9:0D12 26881D           MOV      [ES:DI],BL
67D9:0D15 5E               POP      SI
67D9:0D16 59               POP      CX
67D9:0D17 E2E0             LOOP     0CF9
67D9:0D19 8BF2             MOV      SI,DX
67D9:0D1B 46               INC      SI
67D9:0D1C B90B00           MOV      CX,0B
67D9:0D1F 8A04             MOV      AL,[SI]
67D9:0D21 3C35             CMP      AL,35
67D9:0D23 7302             JNC      0D27
67D9:0D25 040A             ADD      AL,0A
67D9:0D27 2C05             SUB      AL,05
67D9:0D29 8804             MOV      [SI],AL
67D9:0D2B E2F2             LOOP     0D1F
67D9:0D2D E8D4CC           CALL     DA04
67D9:0D30 C3               RET
```

把转换后的注册码进行计算的部分：

```
67D9:145A 8D367D13         LEA      SI,[137D]
67D9:145E 803C37           CMP      BYTE [SI],37    ;第一个字符必须是“7”
67D9:1461 7503             JNZ      1466      ;设内存断点时断在这里
67D9:1463 EB15             JMP      SHORT 147A
67D9:1465 90               NOP
67D9:1466 2EC606931500     MOV      BYTE [CS:1593],00
67D9:146C 90               NOP
67D9:146D 8D368015         LEA      SI,[1580]
67D9:1471 894403           MOV      [SI+03],AX
67D9:1474 895C06           MOV      [SI+06],BX
67D9:1477 EB43             JMP      SHORT 14BC
67D9:1479 90               NOP
67D9:147A 8D367F13         LEA      SI,[137F]
67D9:147E B90A00           MOV      CX,0A
67D9:1481 B300             MOV      BL,00
67D9:1483 B753             MOV      BH,53
67D9:1485 AC               LODSB
67D9:1486 32C7             XOR      AL,BH
67D9:1488 02C7             ADD      AL,BH
67D9:148A 02D8             ADD      BL,AL
67D9:148C E2F7             LOOP     1485
67D9:148E 8AC3             MOV      AL,BL
67D9:1490 B400             MOV      AH,00
67D9:1492 B30A             MOV      BL,0A
67D9:1494 F6F3             DIV      BL
67D9:1496 8D367E13         LEA      SI,[137E]
67D9:149A 8A04             MOV      AL,[SI]
67D9:149C 3C30             CMP      AL,30
67D9:149E 7507             JNZ      14A7
67D9:14A0 80FC00           CMP      AH,00
67D9:14A3 7410             JZ       14B5
67D9:14A5 B03A             MOV      AL,3A
67D9:14A7 2AC4             SUB      AL,AH
67D9:14A9 3C30             CMP      AL,30
67D9:14AB 7406             JZ       14B3
67D9:14AD 3C31             CMP      AL,31
67D9:14AF 7402             JZ       14B3
67D9:14B1 EBB3             JMP      SHORT 1466
67D9:14B3 8804             MOV      [SI],AL
67D9:14B5 2EC606931501     MOV      BYTE [CS:1593],01
67D9:14BB 90               NOP
67D9:14BC E845C5           CALL     DA04
67D9:14BF C3               RET
```

对这类软件我不感兴趣，主要是觉得自己没那么好运气，不可能人人都中奖的。

`4dd466d0e1e0cc123afbe504da2462ca`
