
# 手脱 ASProtect 2.1 壳

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-547.htm`](https://www.pediy.com/kssd/pediy08/pediy8-547.htm)

**【脱壳文件】** `EPSON 打印机工具`

**【下载地址】** [http://www.ssclg.com/download/sscserve.exe](http://www.ssclg.com/download/sscserve.exe)

**【加壳方式】** `ASProtect 2.1x SKE -> Alexey Solodovnikov`

**【作者声明】**：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教

**【调试环境】**：`Win2003`、`OllyDBD`、`PEiD`、`LordPE`、`ImportREC`

**【脱壳过程】**：近日论坛闲逛，遇此软件，学了 N 久 `ASPR` 了，正愁无软柿可捏，于是照猫画虎一番，算给我等菜鸟写个笔记吧。

## 一、避开加密，得到完整 IAT

`OD` 忽略除 `INT3` 外的所有异常（注意同时忽略以下所有异常前面不要选），过两次异常后在 `CODE` 段下内存断点，到这里:

```
代码:

* * *

004B09F0    55             PUSH EBP
004B09F1    8BEC           MOV EBP,ESP
004B09F3    83C4 F0        ADD ESP,-10
004B09F6    B8 40074B00    MOV EAX,ssc_serv.004B0740
004B09FB    E8 7C5CF5FF    CALL ssc_serv.0040667C
004B0A00    A1 70744B00    MOV EAX,DWORD PTR DS:[4B7470]
004B0A05    8B00           MOV EAX,DWORD PTR DS:[EAX]
004B0A07    E8 D43FFBFF    CALL ssc_serv.004649E0
004B0A0C    A1 70744B00    MOV EAX,DWORD PTR DS:[4B7470]
004B0A11    8B00           MOV EAX,DWORD PTR DS:[EAX]
004B0A13    BA 800A4B00    MOV EDX,ssc_serv.004B0A80            ; ASCII "SSC Service Utility"
004B0A18    E8 BB3BFBFF    CALL ssc_serv.004645D8
004B0A1D    8B0D C0754B00  MOV ECX,DWORD PTR DS:[4B75C0]        ; ssc_serv.004BC344
004B0A23    A1 70744B00    MOV EAX,DWORD PTR DS:[4B7470]
004B0A28    8B00           MOV EAX,DWORD PTR DS:[EAX]

* * *
```

搜索一下 `FF 25`

找到这里

`0040121C - FF25 00D24B00 JMP DWORD PTR DS:[4BD200] ; kernel32.CloseHandle`

数据窗口到 `4BD200` 看一下：

```
代码:

* * *

004BD154  7C96AE65  ntdll.RtlDeleteCriticalSection
004BD158  7C95F2FC  ntdll.RtlLeaveCriticalSection
004BD15C  7C95F337  ntdll.RtlEnterCriticalSection
004BD160  7C8284E0  kernel32.InitializeCriticalSection
004BD164  7C828CFC  kernel32.VirtualFree
004BD168  7C82BEC9  kernel32.VirtualAlloc
004BD16C  7C82BC09  kernel32.LocalFree
004BD170  7C82BB92  kernel32.LocalAlloc
004BD174  7C82BB6D  kernel32.GetTickCount
004BD178  5358C7CE
004BD17C  7C82C07F  kernel32.GetVersion
004BD180  44481099
004BD184  7C82B44F  kernel32.InterlockedDecrement
004BD188  7C82B43B  kernel32.InterlockedIncrement
004BD18C  7C818EA7  kernel32.VirtualQuery
004BD190  7C82DC10  kernel32.WideCharToMultiByte
004BD194  7C82BC7C  kernel32.MultiByteToWideChar
004BD198  CDFA8D2A
004BD19C  7C817702  kernel32.lstrcpynA
004BD1A0  F92B6CBC
004BD1A4  21D669D5
004BD1A8  F915B282

* * *
```

输入表被加密了，重新运行程序，不忽略非法访问内存异常，忽略其他异常后大约十四次后，搜索参考字符串，找到到达 IAT 处理的 `CALL`。

```
代码:

* * *

00E7EBF6   /EB 0A           JMP SHORT 00E7EC02
00E7EBF8   |68 E8F4E700     PUSH 0E7F4E8                           ; ASCII "85
00E7EBFD   |E8 2A62FeFF     CALL 00E64E2C
00E7EC02   \A1 1C37E800     MOV EAX,DWORD PTR DS:[E8371C]
00E7EC07    8B00            MOV EAX,DWORD PTR DS:[EAX]
00E7EC09    E8 0A8CFFFF     CALL 00E77818             //这里进去！
00E7EC0E    84C0            TEST AL,AL
00E7EC10    75 0A           JNZ SHORT 00E7EC1C
00E7EC12    68 E8F4E700     PUSH 0E7F4E8                           ; ASCII "85

* * *
```

进去后，找到 IAT 处理的 `CALL`

```
代码:

* * *

00E77916    50             PUSH EAX
00E77917    56             PUSH ESI
00E77918    E8 9BFCFFFF    CALL 00E775B8               〈〈〈〈————这里进去

* * *
```

再进来后开始了比较了，开始 `PATCH` 吧

```
代码:

* * *

00E775B8    55             PUSH EBP
00E775B9    8BEC           MOV EBP,ESP
00E775BB    81C4 F8FEFFFF  ADD ESP,-108
00E775C1    53             PUSH EBX
00E775C2    56             PUSH ESI
00E775C3    57             PUSH EDI
00E775C4    8B55 14        MOV EDX,DWORD PTR SS:[EBP+14]
00E775C7    8B5D 08        MOV EBX,DWORD PTR SS:[EBP+8]
00E775CA    8DBD FAFEFFFF  LEA EDI,DWORD PTR SS:[EBP-106]
00E775D0    8BC2           MOV EAX,EDX
00E775D2    48             DEC EAX
00E775D3    83E8 02        SUB EAX,2
00E775D6    0FB630         MOVZX ESI,BYTE PTR DS:[EAX]
00E775D9    8B45 10        MOV EAX,DWORD PTR SS:[EBP+10]
00E775DC    83E8 02        SUB EAX,2
00E775DF    0FB600         MOVZX EAX,BYTE PTR DS:[EAX]
00E775E2    3B43 2C        CMP EAX,DWORD PTR DS:[EBX+2C]
00E775E5    76 06          JBE SHORT 00E775ED
00E775E7    8943 2C        MOV DWORD PTR DS:[EBX+2C],EAX
00E775EA    EB 01          JMP SHORT 00E775ED
00E775EC    6933 C08A433B  IMUL ESI,DWORD PTR DS:[EBX],3B438AC0
00E775F2    3BF0           CMP ESI,EAX             〈〈〈〈〈〈〈————这里比较四种情况
00E775F4    75 5E          JNZ SHORT 00E77654             //这里开始 PATCH
00E775F6    EB 01          JMP SHORT 00E775F9
PATCH：
00E775F4  - E9 078A2500    JMP 010D0000             我用内存管理插件申请到的是 10D0000 处

* * *
```

我跟踪后发现 `ESI` 值有三种情况，`BA`、`88`、`91`，其中 `BA`，`91` 两种情况不加密，`88` 需要改成 `91` 才能不加密。

继续 `PATCH`：

```
代码:

* * *

010D0000  - 0F84 F375DAFF  JE 00E775F9
010D0006    81FE 91000000  CMP ESI,91
010D000C  - 0F84 4276DAFF  JE 00E77654
010D0012    BE 91000000    MOV ESI,91
010D0017  - E9 3876DAFF    JMP 00E77654

* * *
```

下面开始运行吧，跑飞了也没关系，反正我们已经拿到正确的 IAT 表了。

## 二、修复 CALL XXXXXXXX。

到 `OEP` 后开始修复吧，搜索一下 `89,45,F0,B8,00,07,00,00` 到达这里：

```
代码:

* * *

00E77188    8945 F0        MOV DWORD PTR SS:[EBP-10],EAX
00E7718B    B8 00070000    MOV EAX,700
00E77190    E8 B7B3FDFF    CALL 00E5254C //就这里改吧

PATCH：
00E77190  - E9 6B8E2200    JMP 010A0043

* * *
```

到 `10A0000` 处写上代码



```
代码:

* * *

010A0000    0000            ADD BYTE PTR DS:[EAX],AL
010A0002    0000            ADD BYTE PTR DS:[EAX],AL
010A0004    0000            ADD BYTE PTR DS:[EAX],AL
010A0006    0000            ADD BYTE PTR DS:[EAX],AL
010A0008    0000            ADD BYTE PTR DS:[EAX],AL
010A000A    BA 00104000     MOV EDX,401000                     \
010A000F    803A E8         CMP BYTE PTR DS:[EDX],0E8           |
010A0012    75 14           JNZ SHORT 010A0028                  |   
010A0014    8B42 01         MOV EAX,DWORD PTR DS:[EDX+1]    | 
010A0017    03C2            ADD EAX,EDX                         |
010A0019    05 05000000     ADD EAX,5                           |    
010A001E    3D 00000601     CMP EAX,1060000                     |
010A0023    75 03           JNZ SHORT 010A0028                  |
010A0025    EB 0C           JMP SHORT 010A0033                  |
010A0027    90              NOP                                 |     
010A0028    42              INC EDX                             |
010A0029    81FA 00104B00   CMP EDX,4B1000                      |
010A002F  ^ 72 DE           JB SHORT 010A000F                 |
010A0031  - EB FE           JMP SHORT 010A0031                  |   
010A0033    8915 00010A01   MOV DWORD PTR DS:[10A0100],EDX   |
010A0039    60              PUSHAD                              |
010A003A    FFE2            JMP EDX                             / 

//这段代码是从 401000 开始扫描代码段,查找 CALL XXXXXX.是则跳到原 CALL 处执行. 
010A003C    90              NOP
010A003D    90              NOP
010A003E    90              NOP
010A003F    90              NOP
010A0040    90              NOP
010A0041    90              NOP
010A0042    90              NOP
010A0043    60              PUSHAD              //这是从原 CALL 处返回点,此时 EDX 值即为指针.
010A0044    B8 54D14B00     MOV EAX,4BD154
010A0049    90              NOP
010A004A    3910            CMP DWORD PTR DS:[EAX],EDX
010A004C    75 23           JNZ SHORT 010A0071
010A004E    8B0D 00010A01   MOV ECX,DWORD PTR DS:[10A0100]
010A0054    C701 FF250000   MOV DWORD PTR DS:[ECX],25FF
010A005A    8941 02         MOV DWORD PTR DS:[ECX+2],EAX
010A005D    61              POPAD
010A005E    90              NOP
010A005F    8B15 00010A01   MOV EDX,DWORD PTR DS:[10A0100]
010A0065    90              NOP
010A0066    90              NOP
010A0067    90              NOP
010A0068  ^ EB BE           JMP SHORT 010A0028
010A006A    90              NOP
010A006B    90              NOP
010A006C    90              NOP
010A006D    90              NOP
010A006E    90              NOP
010A006F    90              NOP
010A0070    90              NOP
010A0071    83C0 04         ADD EAX,4
010A0074    3D 00D94B00     CMP EAX,4BD900
010A0079  ^ 7E CF           JLE SHORT 010A004A
010A007B  ^ EB E0           JMP SHORT 010A005D                 
010A007D    90              NOP
//这段是修复 CALL XXXXX 为 JMP DWORD PTR DS:[XXXXX]

* * *
```

代码大部分是抄的，里面的 `NOP` 是为了留空的，便于以后别的程序用。

在 `010A000A` 新建 `EIP`，`F9` 运行一下，OK。

### 三、

下面就是 `LODEPE` 开始 `DUMP`，`IMR` 里填上 `OEP: 000B09F0` `IATRVA: 000BD150` `IATSize: 00000730` 修复一下。

测试一下，正常了。

【脱壳总结】：这个没有 `OEP` 抽取,而且校验不严格，比较适合菜鸟作练习用。最后感谢 `SYSCOM`，`SHOOOO`，`林海雪原`等大侠们的脱文。

1. 请仔细看教程,输入表一般是连续的,中间用 `000000` 间隔,如果出现教程中的情况就是被加密了.

```
004BD174  7C82BB6D  kernel32.GetTickCount  
004BD178  5358C7CE      //这个就是加密了.  
004BD17C  7C82C07F  kernel32.GetVersion  
004BD180  44481099     //这个也是加密了  
004BD184  7C82B44F  kernel32.InterlockedDecrement  
```

2.

```
00E775F2    3BF0            CMP ESI,EAX       //这里下断点.  
00E775F4    75 5E           JNZ SHORT 00E77654            
00E775F6    EB 01           JMP SHORT 00E775F9  
```

就可以跟踪了,这个程序有三个值,有的是四个值.
