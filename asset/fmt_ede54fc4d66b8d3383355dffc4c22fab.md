
# 在 rsatools 中已知 (Q-1)*(P-1),E,求 D 的反汇编过程及分析

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-19.htm`](https://www.pediy.com/kssd/pediy09/pediy09-19.htm)

> 使用工具 `softice`

> 调试环境：`windwos2000`

> 原 `RSATOOLS.exe` 在论坛的工具中

> 调试时使用的数据：

> `D2F1F1429A4A565657B25A75341392C5` ＝Ｑ

> `D2F1F1429A4A565657B25A75341392D7` ＝Ｐ

> `ADD1D47967E6ED701852554F88EEE22C416B71A26F3961AE922CFD6ECEC39D73` ＝Ｎ

> `D2F1F1429A4A565657B25A75341392CD`

> `79C5ABBE161D81306BC99A7CAAE8F847734867E9C940C4927AA767AE6218B219` ＝Ｄ

=====================打这里Ｑ Ｐ Ｅ 已经得到============================

```assembly
00405B67   6A00              PUSH      00
00405B69   E8B2200000        CALL      00407C20   ；申请内存
00405B6E   6A00              PUSH      00
00405B70   8BE8              MOV       EBP,EAX
00405B72   E8A9200000        CALL      00407C20   ；申请内存
00405B77   50                PUSH      EAX
00405B78   57                PUSH      EDI
00405B79   56                PUSH      ESI
00405B7A   89442424          MOV       [ESP+24],EAX
00405B7E   E81D390000        CALL      004094A0   ；EDI*ESI-->EAX
00405B83   56                PUSH      ESI         ；相当于 Q*P
00405B84   6A01              PUSH      01
00405B86   56                PUSH      ESI
00405B87   E8944A0000        CALL      0040A620   ；ESI-1-->ESI=[Q-1]
00405B8C   57                PUSH      EDI
00405B8D   6A01              PUSH      01
00405B8F   57                PUSH      EDI
00405B90   E88B4A0000        CALL      0040A620   ；EDI-1-->EDI=[P-1]
00405B95   55                PUSH      EBP
00405B96   57                PUSH      EDI
00405B97   56                PUSH      ESI
00405B98   E803390000        CALL      004094A0   ；相当于(Q-1)*(P-1)
00405B9D   57                PUSH      EDI
00405B9E   56                PUSH      ESI
00405B9F   56                PUSH      ESI
00405BA0   55                PUSH      EBP-------->(Q-1)*(P-1)
00405BA1   53                PUSH      EBX-------->共钥 E
00405BA2   E879540000        CALL      0040B020   ；根据(Q-1)*(P-1)，共钥 E，计算私钥 D
00405BA7   83C44C            ADD       ESP,4C
00405BAA   83F801            CMP       EAX,01      ；无法计算出私钥
00405BAD   0F85C2000000      JNZ       00405C75
00405BB3   53                PUSH      EBX
00405BB4   53                PUSH      EBX
00405BB5   53                PUSH      EBX
00405BB6   55                PUSH      EBP-------->(Q-1)*(P-1)
00405BB7   53                PUSH      EBX-------->共钥 E
00405BB8   E863540000        CALL      0040B020   ；根据(Q-1)*(P-1)，共钥 E，计算私钥 D
00405BBD   A100E44100        MOV       EAX,[0041E400]
00405BC2   83C414            ADD       ESP,14
00405BC5   83F803            CMP       EAX,03
00405BC8   7526              JNZ       00405BF0
00405BCA   8D9424F0050000    LEA       EDX,[ESP+000005F0]
00405BD1   52                PUSH      EDX
00405BD2   53                PUSH      EBX
00405BD3   E8881C0000        CALL      00407860
00405BD8   50                PUSH      EAX
00405BD9   8D442420          LEA       EAX,[ESP+20]
00405BDD   8D8C24FC050000    LEA       ECX,[ESP+000005FC]
00405BE4   50                PUSH      EAX
00405BE5   51                PUSH      ECX
00405BE6   E865B5FFFF        CALL      00401150
00405BEB   83C414            ADD       ESP,14
00405BEE   EB0E              JMP       00405BFE
00405BF0   8D542414          LEA       EDX,[ESP+14]
00405BF4   52                PUSH      EDX
00405BF5   53                PUSH      EBX
00405BF6   E8651C0000        CALL      00407860----->把数据转化为可显示的字符串
00405BFB   83C408            ADD       ESP,08
00405BFE   8B8C24D00B0000    MOV       ECX,[ESP+00000BD0]
00405C05   8D442414          LEA       EAX,[ESP+14]
00405C09   50                PUSH      EAX   --------->已经算出来的Ｄ
00405C0A   68F5030000        PUSH      000003F5
00405C0F   51                PUSH      ECX
00405C10   FF1520614100      CALL      [USER32!SetDlgItemTextA]---显示
00405C16   833D00E4410003    CMP       DWORD PTR [0041E400],03
00405C1D   752A              JNZ       00405C49
00405C1F   8B442410          MOV       EAX,[ESP+10]
```

数据的储格式如下：第一个 dword 的是数据的位数，其中的 D31 位表示这个数的正负,以 100000000H 为进值表示的

```
DD  EBX  l24----->E 共钥
0089D5F8 00000001  00010001  00000000  00000000      ................

DD  EBP  l24----->(Q-1)*(P-1)
0089A718 00000008  669C77D8  E2C84884  3AA4B501      .....w.f.H.....:
0089A728 9B878F1D  88EEE22A  1852554F  67E6ED70      ....*...OUR.p..g
0089A738 ADD1D479  00000000  00000000  00000000      y...............

DD  ESI  l24----->Q-1
0089DAE0 00000004  341392C4  57B25A75  9A4A5656      .......4uZ.WVVJ.
0089DAF0 D2F1F142  00000000  00000000  00000000      B...............

DD  EDI  l24----->P-1
0089A230 00000004  341392D6  57B25A75  9A4A5656      .......4uZ.WVVJ.
0089A240 D2F1F142  00000000  00000000  00000000      B...............
```

`[ESI+000001D8]`------>X0 用于保存临时的计算结果

`[ESI+000001DC]`------>X1 初始化时等于 E

`[ESI+000001E0]`------>X2 初始化时等于 (Q-1)*(P-1)

`[ESI+000001E4]`------>X3 初始化时等于 1----->最后结果就是私钥

`[ESI+000001E8]`------>X4 初始化时等于 0

`[ESI+000001EC]`------>X5 用于保存临时的计算结果

`[EBP-04]`------------>Y0 初始化时等于 1 Y3-Y0*商

`[EBP-08]`------------>Y1 初始化时等于 0 Y2-Y1*商

`[EBP-14]`------------>Y2 初始化时等于 1 Y1-->Y2

`[EBP-18]`------------>Y3 初始化时等于 0 Y0-->Y3

=========验证是否能得到私有密钥\[上面是调用该程序时积存器所指内存的值================

```assembly
0040B020   55                PUSH      EBP
0040B021   8BEC              MOV       EBP,ESP
0040B023   83EC24            SUB       ESP,24
0040B026   56                PUSH      ESI
0040B027   57                PUSH      EDI
0040B028   E8C3C8FFFF        CALL      004078F0------>取得一个地址
0040B02D   8BF0              MOV       ESI,EAX       ；该地址存了一些计算时需要的，已经申请的
0040B02F   F8                CLC                     ；内存
0040B030   8B8630020000      MOV       EAX,[ESI+00000230]
0040B036   33FF              XOR       EDI,EDI
0040B038   3BC7              CMP       EAX,EDI
0040B03A   7408              JZ        0040B044
0040B03C   5F                POP       EDI
0040B03D   33C0              XOR       EAX,EAX
0040B03F   5E                POP       ESI
0040B040   8BE5              MOV       ESP,EBP
0040B042   5D                POP       EBP
0040B043   C3                RET
0040B044   8B86DC010000      MOV       EAX,[ESI+000001DC] X1
0040B04A   8B4D08            MOV       ECX,[EBP+08]
0040B04D   53                PUSH      EBX
0040B04E   50                PUSH      EAX
0040B04F   51                PUSH      ECX
0040B050   E8CBD5FFFF        CALL      00408620--->共钥Ｅ--->X1
0040B055   8B96E0010000      MOV       EDX,[ESI+000001E0] X2
0040B05B   8B450C            MOV       EAX,[EBP+0C]
0040B05E   52                PUSH      EDX
0040B05F   50                PUSH      EAX
0040B060   E8BBD5FFFF        CALL      00408620--->(Q-1)*(P-1)--->X2
0040B065   8B8EDC010000      MOV       ECX,[ESI+000001DC] X1
0040B06B   51                PUSH      ECX
0040B06C   E89FD3FFFF        CALL      00408410--->检查数的正负
0040B071   8B96DC010000      MOV       EDX,[ESI+000001DC] X1
0040B077   8945DC            MOV       [EBP-24],EAX----->存符号
0040B07A   52                PUSH      EDX
0040B07B   6A01              PUSH      01
0040B07D   E8AED3FFFF        CALL      00408430--->清掉Ｄ31 位[求绝对值]
0040B082   8B86E0010000      MOV       EAX,[ESI+000001E0] X2
```
