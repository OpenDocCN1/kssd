
# S-SPLINE 的逆向分析

> 原文：[`www.pediy.com/kssd/pediy03/forum1037.htm`](https://www.pediy.com/kssd/pediy03/forum1037.htm)

下载地：`http://www.shortcut.nl/S-Spline/Shortcut_S-SplineDemo.zip`

这个软件注册码非常难算，软件设计者算法到了变态的地步。我看的头都大了。而且这个软件还设了陷阱。当你上网后，它会到它网站的数据库去比较，如果没有注册过，当即将你的注册版改为未注册版。

这个软件用 `Asprotect1.2` 以后的版本加壳，得手动脱壳，还得修复输入表。以下分析部分引 `BLOWFISH` 的分析结果。

## 输入注册码之后的判断

```
0177:0040BB11  PUSH        004BF728                              //DialogRegister::RegisterButton.OnClick()
0177:0040BB16  CALL        0041EBB6
0177:0040BB1B  LEA         ECX,[EBP-01EC]
0177:0040BB21  CALL        0041C0CF
0177:0040BB26  AND         DWORD PTR [EBP-04],00
0177:0040BB2A  PUSH        0040118F
0177:0040BB2F  PUSH        0041EB3F
0177:0040BB34  PUSH        03
0177:0040BB36  PUSH        18
0177:0040BB38  LEA         EAX,[EBP-00F0]
0177:0040BB3E  PUSH        EAX
0177:0040BB3F  CALL        0047E80A
0177:0040BB44  MOV         ECX,[EBP-24]
0177:0040BB47  MOV         BYTE PTR [EBP-04],01
0177:0040BB4B  CALL        00427F9E
0177:0040BB50  MOV         EBX,EAX
0177:0040BB52  CALL        0042FA11
0177:0040BB57  LEA         EAX,[EBP-00A8]
0177:0040BB5D  PUSH        EAX
0177:0040BB5E  LEA         ECX,[EBX+000001E0]
0177:0040BB64  CALL        00428000                               //GetWindowTextA(), Name
0177:0040BB69  MOV         ESI,004BF724
0177:0040BB6E  PUSH        ESI
0177:0040BB6F  LEA         ECX,[EBP-00A8]
0177:0040BB75  MOV         BYTE PTR [EBP-04],02
0177:0040BB79  CALL        00420B42
0177:0040BB7E  LEA         EAX,[EBP-0090]
0177:0040BB84  PUSH        EAX
0177:0040BB85  LEA         ECX,[EBX+00000224]
0177:0040BB8B  CALL        00428000                               //GetWindowTextA(), Company
0177:0040BB90  PUSH        ESI
0177:0040BB91  LEA         ECX,[EBP-0090]
0177:0040BB97  MOV         BYTE PTR [EBP-04],03
0177:0040BB9B  CALL        00420B42
0177:0040BBA0  MOV         ECX,EBX
0177:0040BBA2  CALL        0040B9D9
0177:0040BBA7  LEA         EAX,[EBP-01B8]
0177:0040BBAD  LEA         ECX,[EBX+0000019C]
0177:0040BBB3  PUSH        EAX
0177:0040BBB4  CALL        00428000                               //GetWindowTextA(), Serial[1]
0177:0040BBB9  MOV         ESI,EAX
0177:0040BBBB  LEA         EAX,[EBP-01A0]
0177:0040BBC1  PUSH        EAX
0177:0040BBC2  LEA         ECX,[EBX+00000158]
0177:0040BBC8  MOV         BYTE PTR [EBP-04],04
0177:0040BBCC  CALL        00428000                               //GetWindowTextA(), Serial[2]
0177:0040BBD1  MOV         EDI,EAX
0177:0040BBD3  LEA         EAX,[EBP-0170]
0177:0040BBD9  PUSH        EAX
0177:0040BBDA  LEA         ECX,[EBX+00000114]
0177:0040BBE0  MOV         BYTE PTR [EBP-04],05
0177:0040BBE4  CALL        00428000                               //GetWindowTextA(), Serial[3]
0177:0040BBE9  MOV         [EBP-1C],EAX
0177:0040BBEC  LEA         EAX,[EBP-0158]
0177:0040BBF2  PUSH        EAX
0177:0040BBF3  LEA         ECX,[EBX+000000D0]
0177:0040BBF9  MOV         BYTE PTR [EBP-04],06
0177:0040BBFD  CALL        00428000                               //GetWindowTextA(), Serial[4]
0177:0040BC02  MOV         [EBP-14],EAX
0177:0040BC05  LEA         EAX,[EBP-0188]
0177:0040BC0B  PUSH        EAX
0177:0040BC0C  LEA         ECX,[EBX+0000008C]
0177:0040BC12  MOV         BYTE PTR [EBP-04],07
0177:0040BC16  CALL        00428000                               //GetWindowTextA(), Serial[5]

........

0177:0040BE4E  PUSH        EAX
0177:0040BE4F  MOV         ECX,ESI
0177:0040BE51  CALL        00459A71                               //第一处疯狂计算的地方
0177:0040BE56  TEST        AL,AL
0177:0040BE58  MOV         [EBP-15],AL                            //返回值
0177:0040BE5B  JNZ         0040BF1C
0177:0040BE61  LEA         EAX,[EBP-40]
0177:0040BE64  PUSH        EAX
0177:0040BE65  LEA         ECX,[EBX+48]
0177:0040BE68  CALL        00428000                               //GetWindowTextA
...........
0177:0040BF0F  PUSH        EAX
0177:0040BF10  MOV         ECX,ESI
0177:0040BF12  CALL        00459A71                               //第二处疯狂计算的地方
0177:0040BF17  MOV         [EBP-15],AL                            //返回值
0177:0040BF1A  JMP         0040BF1E
0177:0040BF1C  XOR         EDI,EDI
0177:0040BF1E  LEA         EAX,[EBP-60]
0177:0040BF21  PUSH        EAX
0177:0040BF22  LEA         ECX,[EBX+48]
0177:0040BF25  CALL        00428000
0177:0040BF2A  PUSH        004BF704                               //注册码的第一部分 Serial[1]的黑名单“!AMOK”？
0177:0040BF2F  MOV         ECX,EAX
0177:0040BF31  MOV         BYTE PTR [EBP-04],27
0177:0040BF35  CALL        0041FC03                               //strcmp()
0177:0040BF3A  TEST        AL,AL
0177:0040BF3C  JNZ         0040BF84
0177:0040BF3E  LEA         EAX,[EBP-40]
0177:0040BF41  PUSH        EAX
0177:0040BF42  LEA         ECX,[EBX+0000019C]
0177:0040BF48  CALL        00428000                               //GetWindowTextA(), Serial[6]
0177:0040BF4D  PUSH        004BF6FC                               //注册码的最后一部分的黑名单？（有不可显示字符）
0177:0040BF52  MOV         ECX,EAX
0177:0040BF54  MOV         BYTE PTR [EBP-04],28
0177:0040BF58  CALL        0041FC03                               //strcmp()，跟进去可看到 CMPSB 指令
0177:0040BF5D  MOV         [EBP-0D],AL                             //比较的结果。
0177:0040BF60  LEA         ECX,[EBP-3C]
0177:0040BF63  MOV         BYTE PTR [EBP-04],29
0177:0040BF67  CALL        0040117A
0177:0040BF6C  LEA         ECX,[EBP-40]
0177:0040BF6F  MOV         BYTE PTR [EBP-04],27
0177:0040BF73  CALL        0040117A
0177:0040BF78  CMP         BYTE PTR [EBP-0D],00 //标志位
0177:0040BF7C  JNZ         0040BF84
0177:0040BF7E  AND         BYTE PTR [EBP-0D],00
0177:0040BF82  JMP         0040BF88
0177:0040BF84  MOV         BYTE PTR [EBP-0D],01
```

## 启动时联网检查新版本

启动时通过 `InternetOpenUrlA` 检查新版本（`BPM WININET`）：

```
017F:00AF1870 68 74 74 70 3A 2F 2F 77-77 77 2E 73 68 6F 72 74   http://www.short
017F:00AF1880 63 75 74 2E 6E 6C 2F 63-67 69 2D 62 69 6E 2F 43   cut.nl/cgi-bin/C
017F:00AF1890 68 65 63 6B 2E 63 67 69-3F 75 70 64 61 74 65 25   heck.cgi?update%
017F:00AF18A0 32 30 73 2D 73 70 6C 69-6E 65 5F 32 5F 78 00 00   20s-spline_2_x..
```

## 启动时联网检查注册码

启动时联网检查注册码，如果检查不通过又成为 `unregistered`。这个检查是随机的，即并非每次启动都检查。

```
017F:00AF2C10 68 74 74 70 3A 2F 2F 77-77 77 2E 73 68 6F 72 74   http://www.short
017F:00AF2C20 63 75 74 2E 6E 6C 2F 63-67 69 2D 62 69 6E 2F 43   cut.nl/cgi-bin/C
017F:00AF2C30 68 65 63 6B 2E 63 67 69-3F 4A 56 43 36 35 4A 58   heck.cgi?JVC65JX
017F:00AF2C40 46 56 4F 56 57 36 4E 47-50 50 56 56 45 34 4B 48   FVOVW6NGPPVVE4KH
017F:00AF2C50 49 41 00 2E 6E 6C 2F 53-51 00 00 00 51 00 00 00   IA..nl/SQ...Q...
```

## 相关的代码

```
0177:004159DD  LEA         ECX,[ESI+0000089C]
0177:004159E3  PUSH        ECX
0177:004159E4  LEA         EAX,[ESI+00000884]
0177:004159EA  PUSH        EAX
0177:004159EB  LEA         EAX,[ESI+0000086C]
0177:004159F1  PUSH        EAX
0177:004159F2  MOV         ECX,ESI
0177:004159F4  CALL        00413013                               //联网检查有无新版本
0177:004159F9  TEST        AL,AL
0177:004159FB  JZ          00415A4B
0177:004159FD  LEA         EAX,[ESI+00000884]
0177:00415A03  PUSH        EAX
0177:00415A04  LEA         EAX,[EBP-30]
0177:00415A07  PUSH        004C065C                               //"update"
0177:00415A0C  PUSH        EAX
0177:00415A0D  CALL        00402CF8
0177:00415A12  SUB         ESP,0C
0177:00415A15  MOV         ECX,ESP
0177:00415A17  MOV         [EBP-10],ESP
0177:00415A1A  PUSH        004C0650                               //"available"
0177:00415A1F  PUSH        ECX
0177:00415A20  MOV         ECX,EAX
0177:00415A22  MOV         BYTE PTR [EBP-04],37
0177:00415A26  CALL        0041FFC9
0177:00415A2B  MOV         ECX,[EBP-14]
0177:00415A2E  CALL        0041BADD
0177:00415A33  LEA         ECX,[EBP-2C]
0177:00415A36  MOV         BYTE PTR [EBP-04],38
0177:00415A3A  CALL        0040117A
0177:00415A3F  MOV         BYTE PTR [EBP-04],24
0177:00415A43  LEA         ECX,[EBP-30]
0177:00415A46  JMP         00415AEA
```



### 1. 注册码校验流程入口

```
0177:00415A4B  CMP      BYTE PTR [ESI+000000D4],00 //标志位    
0177:00415A52  PUSH     FF   
0177:00415A54  JZ       00415AA1   
0177:00415A56  PUSH     0000E156   
.......................   
0177:00415A9F  JMP      00415AEA   
0177:00415AA1  PUSH     0000E15E   
.......................   
0177:00415AEA  CALL     0040117A   
0177:00415AEF  MOV      ECX,ESI   
0177:00415AF1  CALL     004161C5                                //本地检查注册码，并随机联网检查注册码   
0177:00415AF6  FLD1     
0177:00415AF8  FSTP     REAL8 PTR [ESI+00000298]   
```

在上面的 `0177:00415AF1` 处跟进去，看见本地判断注册码的地方：

```
0177:0041644C  CALL     [EAX+14]                    //判断注册码，里面是疯狂的计算   
0177:0041644F  MOV      ECX,004D420C   
0177:00416454  MOV      [EBP-0D],AL                 //判断的结果   
0177:00416457  CALL     0041E80A   
0177:0041645C  CMP      BYTE PTR [EBP-0D],00         //注册码错误？    
0177:00416460  JZ       004168BD   
0177:00416466  MOV      ECX,[EDI+00000208]   
0177:0041646C  CMP      ECX,[004A5C78]   
0177:00416472  LEA      EAX,[EDI+00000208]   
0177:00416478  JNZ      00416482   
```

### 2. 注册码比对逻辑分析

再在上面的 `0177:0041644C` 处跟进去，疯狂的计算就开始了。第一处比较注册码的地方如下：

```
0177:004629C8  MOV      EAX,[EBP-0244]   
0177:004629CE  IMUL     EAX,EAX,000343FD              //疯狂的乘法（RSA？）   
0177:004629D4  ADD      EAX,00269EC3   
0177:004629D9  MOV      [EBP-0244],EAX   
0177:004629DF  MOV      EAX,[EBP+FFFFF1A8]   
0177:004629E5  ADD      EAX,[EBP+FFFFF19C]   
0177:004629EB  PUSH     EAX   
0177:004629EC  PUSH     00   
0177:004629EE  PUSH     DWORD PTR [EBP-0244]   
0177:004629F4  CALL     004791DC   
0177:004629F9  ADD      ESP,0C   
0177:004629FC  MOV      [EBP+FFFFF198],EAX   
0177:00462A02  MOV      EAX,[EBP+FFFFF1B4]   
0177:00462A08  SUB      EAX,[EBP+FFFFF198]   
0177:00462A0E  SUB      EAX,0A   
0177:00462A11  CMP      [EBP-0174],EAX   
0177:00462A17  JGE      00462A7D   
0177:00462A19  MOV      EAX,[EBP-0174]   
0177:00462A1F  MOVSX    EAX,BYTE PTR [EAX+EBP-0160]   
0177:00462A27  MOV      [EBP-0178],EAX   
0177:00462A2D  CMP      DWORD PTR [EBP-0178],39   
0177:00462A34  JLE      00462A47   
0177:00462A36  MOV      EAX,[EBP-0178]   
0177:00462A3C  SUB      EAX,41   
0177:00462A3F  MOV      [EBP-0178],EAX   
0177:00462A45  JMP      00462A56   
0177:00462A47  MOV      EAX,[EBP-0178]   
0177:00462A4D  SUB      EAX,30   
0177:00462A50  MOV      [EBP-0178],EAX   
0177:00462A56  MOV      EAX,[EBP-0178]   
0177:00462A5C  MOVSX    EAX,BYTE PTR [EAX+EBP-34]     //真注册码    
0177:00462A61  MOV      ECX,[EBP+10]   
0177:00462A64  ADD      ECX,[EBP-0174]   
0177:00462A6A  MOVSX    ECX,BYTE PTR [ECX]             //假注册码   
0177:00462A6D  CMP      EAX,ECX                       //比较   
0177:00462A6F  JZ       00462A78   
0177:00462A71  XOR      AL,AL                         //bad guy   
0177:00462A73  JMP      0046A910   
0177:00462A78  JMP      004626A5   
```

我共找到跳到 `46A910` 的有 5 个比较分别为：`462A73`，`463069`，`4647F7`，`465BB5`，`46A90C`。

我将其全部 `JZ` 改为 `JNZ`，但程序出错，而且自动启动 `OUTLOOK` 发信给作者。哪位高手给我一个完美的爆破方案？

### 3. 错误处理与补丁方案

```
0177:0041644C  CALL     [EAX+14]                    //判断注册码，里面是疯狂的计算   
0177:0041644F  MOV      ECX,004D420C   
0177:00416454  MOV      [EBP-0D],AL                 //判断的结果   
0177:00416457  CALL     0041E80A   
0177:0041645C  CMP      BYTE PTR [EBP-0D],00         //注册码错误？    
0177:00416460  JZ       004168BD   
```

`:004168AE 6A04             push 00000004   
:004168B0 8D8708020000       lea eax, dword ptr [edi+00000208]   
:004168B6 50                 push eax   
:004168B7 FF9704020000       call dword ptr [edi+00000204]` 

改为：

```
:004168AE 6A04             push 00000004   
:004168B0 8D8708020000       lea eax, dword ptr [edi+00000208]   
:004168B6 50                 push eax   
:004168B7 83C408             add esp, 00000008   
:004168BA 90                 nop   
:004168BB 90                 nop   
:004168BC 90                 nop   
```

最后把 `BLOWFISH` 兄说的那个跳，改一下就成注册版了。

```
0177:0041644C  CALL     [EAX+14]                    //判断注册码，里面是疯狂的计算   
0177:0041644F  MOV      ECX,004D420C   
0177:00416454  MOV      [EBP-0D],AL                 //判断的结果   
0177:00416457  CALL     0041E80A   
0177:0041645C  CMP      BYTE PTR [EBP-0D],00         //注册码错误？    
0177:00416460  JZ       004168BD //改为 JNZ 就可以了。   
```

### 4. 脱壳与调试问题

我怀疑我是脱壳问题。我用的是 `CASPR1.1` 脱的壳。脱完后，只有 `1.92M`。可是用 `TR2000` 脱的壳却有 `2.04M`，`BLOWFISH`，这是为什么？

为什么在 `TR2000` 下设 `BPX 47E910` 时，程并没有停在那个地址，而是停在别的地址，`DB？`，`DB？` 一大堆，这是为什么？`PNEWSEC` 也是如此 `4dd466d0e1e0cc123afbe504da2462ca`
