
# Terminal Overdrive 2000

> 原文：[`www.pediy.com/kssd/pediy03/forum545.htm`](https://www.pediy.com/kssd/pediy03/forum545.htm)

## 破 Terminal Overdrive 2000 (TO2000)

如有雷同，纯属巧合！

忘了在哪下载，好像 `NEWHUA`, `YESKEY`......

反正《电脑报》吹得响亮，大家可用搜索引擎查找。

`TO2000.exe` 933KB

难度：有点。

我用 `TRW2K1.03` demo 不用 1.23

我吹：

> `TO2000` is the worlds best internet accelerator and searching tool available. You will also find that the `TO2000` Connection Manager is unmatched in its ability to sustain the fastest connections possible without the hassles associated with idle time outs.

此软件是用 `Delphi` 编写，用了很多第三方的控件。

分析注册表(`regmon`)后，得知注册信息保存在：

*   `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Time Zones\GMT`
*   `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\explorer\Tips`

我的键值:

`4u8h 儎厗=kC>nqHDJEFGXHK 枟槞 MMMQOSRST￥ウZZZ^\\\`\\\`a 氨渤 f 刀犯 lnlqoorz|}媚牌|絸八掏嗡菲扔哉`

两处的键值是完全一模一样的，这就相当于在其中的一处做了备份。

键值的内容是加密的，包括注册信息，难怪用 `W32DASM` 好像～～～～？$\_$

## Let's begin:

运行 `TO2000`，对其任务栏图标点右键，点 `UNLOCK`。

我的 `SERIAL NUMBER` 为 `627120875`

`NAME` 填上 `zest`

`COMPANY NAME` 不填

`UNLOCK CODE` 填 `9876543210abcdef`（一定要 `>=14` 位，我原取 `10` 位，浪费 `3` 天时间跟踪，你可试试，详见下）

点击 `OK`，一点鸟叫都没有。

```
s 0 -1 '9876543210abcdef'

bpm 8068ac92（你的可能不同）

F5

bd

bpm es:0

F5

bd
```

到：

（在 `SHRLK21.DLL` 中）

```
0167:004F767D  MOV   AL,[EDX]
0167:004F767F  CMP   AL,61       ==>这里，小写字母变大写
0167:004F7681  JB    004F7689
0167:004F7683  CMP   AL,7A
0167:004F7685  JA    004F7689
0167:004F7687  SUB   AL,20
0167:004F7689  MOV   [ESI],AL
0167:004F768B  INC   EDX
0167:004F768C  INC   ESI
0167:004F768D  DEC   EBX
0167:004F768E  TEST  EBX,EBX
0167:004F7690  JNZ   004F767D
0167:004F7692  POP   EDI
0167:004F7693  POP   ESI
0167:004F7694  POP   EBX
0167:004F7695  RET                 ==>RET 后,到：00534dfe

0167:00534DE5 E80EEDFBFF          CALL  004F3AF8
0167:00534DEA 83F80E              CMP   EAX,0000000E  ==>比较 UNLOCK CODE 位数>=14
0167:00534DED 0F8CEA000000        JL    00534EDD      ==>跳 bye!
0167:00534DF3 8D55E8              LEA   EDX,[EBP-18]
0167:00534DF6 8B45FC              MOV   EAX,[EBP-04]
0167:00534DF9 E85E28FCFF          CALL  004F765C      ==>此 CALL 变大写
0167:00534DFE 8B55E8              MOV   EDX,[EBP-18]  ==>RET 后到此
0167:00534E01 8D45FC              LEA   EAX,[EBP-04]  ==>d edx=9876543210ABCDEF
0167:00534E04 E80BEBFBFF          CALL  004F3914
0167:00534E09 8B45FC              MOV   EAX,[EBP-04]
0167:00534E0C 8A5801              MOV   BL,[EAX+01]   ==>取第二位 8
0167:00534E0F 8B45FC              MOV   EAX,[EBP-04]
0167:00534E12 8A4004              MOV   AL,[EAX+04]   ==>取第五位 5
0167:00534E15 8845EF              MOV   [EBP-11],AL
0167:00534E18 8B45FC              MOV   EAX,[EBP-04]
0167:00534E1B 8A4005              MOV   AL,[EAX+05]   ==>取第六位 4
0167:00534E1E 8845EE              MOV   [EBP-12],AL
0167:00534E21 8D45FC              LEA   EAX,[EBP-04]
0167:00534E24 B901000000          MOV   ECX,00000001
0167:00534E29 BA02000000          MOV   EDX,00000002
0167:00534E2E E809EFFBFF          CALL  004F3D3C
0167:00534E33 8D45FC              LEA   EAX,[EBP-04]  ==>976543210ABCDEF
0167:00534E36 B901000000          MOV   ECX,00000001
0167:00534E3B BA04000000          MOV   EDX,00000004
0167:00534E40 E8F7EEFBFF          CALL  004F3D3C
0167:00534E45 8D45FC              LEA   EAX,[EBP-04]  ==>9763210ABCDEF
0167:00534E48 B901000000          MOV   ECX,00000001
0167:00534E4D BA04000000          MOV   EDX,00000004
0167:00534E52 E8E5EEFBFF          CALL  004F3D3C
0167:00534E57 33C0                XOR   EAX,EAX
0167:00534E59 8AC3                MOV   AL,BL         ==>8 （必须为 A）
0167:00534E5B 83E841              SUB   EAX,00000041
0167:00534E5E 6BC01A              IMUL  EAX,EAX,0000001A
0167:00534E61 33D2                XOR   EDX,EDX
0167:00534E63 8A55EF              MOV   DL,[EBP-11]   ==>5 （必须为 A）
0167:00534E66 83EA41              SUB   EDX,00000041
0167:00534E69 03C2                ADD   EAX,EDX       ==>,必须为 0,见下 534ecc
0167:00534E6B 8B5508              MOV   EDX,[EBP+08]
0167:00534E6E 8902                MOV   [EDX],EAX     ==>0 存入 77f918
0167:00534E70 33DB                XOR   EBX,EBX
0167:00534E72 8D55F0              LEA   EDX,[EBP-10]
0167:00534E75 A174A85300          MOV   EAX,[0053A874]
0167:00534E7A E8DD27FCFF          CALL  004F765C
0167:00534E7F 8B45F0              MOV   EAX,[EBP-10]
0167:00534E82 E871ECFBFF          CALL  004F3AF8
0167:00534E87 85C0                TEST  EAX,EAX
0167:00534E89 7E13                JLE   00534E9E
0167:00534E8B BA01000000          MOV   EDX,00000001
0167:00534E90 8B4DF0              MOV   ECX,[EBP-10]  ==>A9B9C5
0167:00534E93 0FB64C11FF          MOVZX ECX,Byte Ptr [ECX+1*EDX-01]
0167:00534E98 03D9                ADD   EBX,ECX
0167:00534E9A 42                  INC   EDX
0167:00534E9B 48                  DEC   EAX
0167:00534E9C 75F2                JNZ   00534E90
0167:00534E9E 8D55E4              LEA   EDX,[EBP-1C]
0167:00534EA1 8B45F8              MOV   EAX,[EBP-08]  ==>MY SERIAL NUMBER 627120875
0167:00534EA4 E8B327FCFF          CALL  004F765C
0167:00534EA9 8B45E4              MOV   EAX,[EBP-1C]
0167:00534EAC 8D4DE8              LEA   ECX,[EBP-18]
0167:00534EAF 8BD3                MOV   EDX,EBX
0167:00534EB1 E8BA070000          CALL  00535670      ==>计算
0167:00534EB6 8B55E8              MOV   EDX,[EBP-18]  ==>482949333C24    OK！
0167:00534EB9 8B45FC              MOV   EAX,[EBP-04]  ==>9763210ABCDEF
0167:00534EBC E847EDFBFF          CALL  004F3C08      ==>对比 CALL，你可知 MACU......
0167:00534EC1 7506                JNZ   00534EC9      ==>no!no!!
0167:00534EC3 8B45F4              MOV   EAX,[EBP-0C]
0167:00534EC6 C60001              MOV   Byte Ptr [EAX],01     ==>ok!!
0167:00534EC9 8B4508              MOV   EAX,[EBP+08]
0167:00534ECC 833800              CMP   DWord Ptr [EAX],00000000 ==>77f918 之值必须为 0,why?见下
0167:00534ECF 750C                JNZ   00534EDD          ==>*&^%$#@###ZZZzzz?
0167:00534ED1 807DEE43            CMP   Byte Ptr [EBP-12],43    ==>第六位 4 必须为 C
0167:00534ED5 7406                JZ    00534EDD          ==>yes,jump!!!
0167:00534ED7 8B45F4              MOV   EAX,[EBP-0C]
0167:00534EDA C60000              MOV   Byte Ptr [EAX],00     ==>don't come here
0167:00534EDD 33C0                XOR   EAX,EAX
0167:00534EDF 5A                  POP   EDX
0167:00534EE0 59                  POP   ECX
0167:00534EE1 59                  POP   ECX
0167:00534EE2 648910              MOV   FS:[EAX],EDX
0167:00534EE5 68144F5300          PUSH  00534F14
0167:00534EEA 8D45E4              LEA   EAX,[EBP-1C]
0167:00534EED BA02000000          MOV   EDX,00000002
0167:00534EF2 E8A9E9FBFF          CALL  004F38A0
0167:00534EF7 8D45F0              LEA   EAX,[EBP-10]
0167:00534EFA E87DE9FBFF          CALL  004F387C
0167:00534EFF 8D45F8              LEA   EAX,[EBP-08]
0167:00534F02 BA02000000          MOV   EDX,00000002
0167:00534F07 E894E9FBFF          CALL  004F38A0
0167:00534F0C C3                  RET
```

`bpm 77f918` 看看

`F5`

```
0167:00534C87  CMP   DWord Ptr [EBP-14],00000000   ==>还有比较!所以必须为 0!
0167:00534C8B  JNZ   00534CA5                 ==>don't jump!
0167:00534C8D  MOV   ECX,[EBP-0C]
0167:00534C90  MOV   EDX,[EBP-08]
0167:00534C93  MOV   EAX,[EBP-04]
0167:00534C96  CALL  00534FA8                 ==>thank you......
0167:00534C9B  CALL  00535258
0167:00534CA0  JMP   00534D2C
0167:00534CA5  CALL  00534F1C
0167:00534CAA  TEST  AL,AL
0167:00534CAC  JNZ   00534CD9
```

所以 `my UNLOCK CODE`：

`9876543210abcdef`

`^ ^^`



`4a82ac949333c24` `oooooook!!!!!!!!`

`TO2000======>TO2000 PRO` 省 MONEY `$19.95 US`。

附：用到`SHRLK21.DLL`的程序比较广，如`CLEANTIDY`，`WEBZIP(SHRLK20.DLL)??`等等
`4dd466d0e1e0cc123afbe504da2462ca`
