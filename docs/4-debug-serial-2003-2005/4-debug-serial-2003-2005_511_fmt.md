
# 网络精确时钟 2.25

> 原文：[`www.pediy.com/kssd/pediy06/pediy6173.htm`](https://www.pediy.com/kssd/pediy06/pediy6173.htm)

## 破解过程

### 详细过程

1. 用 `FI2.5` 查壳，无壳。用 `w32dasm` 反汇编，查找有用信息："注册成功，感谢使用！"，双击来到此处：

```
00405288 51                  push ecx
:00405289 E8C6130000          call 00406654
:0040528E 83C404             add esp, 00000004
:00405291 8D95D4FBFFFF       lea edx, dword ptr [ebp+FFFFFBD4]
:00405297 52                 push edx
:00405298 B970B84100         mov ecx, 0041B870
:0040529D E88BDFFFFF         call 0040322D--------在此 CALL 设断!
:004052A2 85C0               test eax, eax----------这是经典比较!
:004052A4 7458               je 004052FE
:004052A6 8D85D4FBFFFF       lea eax, dword ptr [ebp+FFFFFBD4]
:004052AC 50                 push eax
:004052AD B970B84100         mov ecx, 0041B870
:004052B2 E8DCE2FFFF         call 00403593
:004052B7 85C0               test eax, eax
:004052B9 7418               je 004052D3
:004052BB 6A00               push 00000000
:004052BD 6860DB4100         push 0041DB60
* Possible StringData Ref from Data Obj ->"注册成功，感谢使用！"
                          |
:004052C2 6898664100         push 00416698-------停在此！往上找！第一个跳转！
```

2. 用 `0llydbg` 动态调试，设断在：`:0040529D E88BDFFFFF call 0040322D`，以下是程序代码：

```
00405289  |.  E8 C6130000    CALL   STANDARD.00406654                  ; \\STANDARD.00406654
0040528E  |.  83C4 04        ADD    ESP, 4
00405291  |.  8D95 D4FBFFFF  LEA    EDX, [LOCAL.267]
00405297  |.  52             PUSH   EDX                                ; /Arg1 = 0068EBE4 ASCII "123456789"
00405298  |.  B9 70B84100    MOV    ECX, STANDARD.0041B870 ; |
0040529D  |.  E8 8BDFFFFF    CALL   STANDARD.0040322D         ; \\STANDARD.0040322D--------在此 CALL 设断！按 F7 追进此 CALL!
004052A2  |.  85C0           TEST   EAX, EAX\---------------------------这是经典比较句式，所以上面的 CALL 我们要追进!
004052A4  |.  74 58          JE     SHORT STANDARD.004052FE
004052A6  |.  8D85 D4FBFFFF  LEA    EAX, [LOCAL.267]
004052AC  |.  50             PUSH   EAX                                ; /Arg1 = 0068EBE4 ASCII "123456789"
004052AD  |.  B9 70B84100    MOV    ECX, STANDARD.0041B870          ; |
004052B2  |.  E8 DCE2FFFF    CALL   STANDARD.00403593               ; \\STANDARD.00403593
004052B7  |.  85C0           TEST   EAX, EAX
004052B9  |.  74 18          JE     SHORT STANDARD.004052D3
004052BB  |.  6A 00          PUSH   0                                  ; /style = MB_OK|MB_APPLMODAL
004052BD  |.  68 60DB4100    PUSH   STANDARD.0041DB60                ; |Title = "网络精确时钟"
004052C2  |.  68 98664100    PUSH   STANDARD.00416698                ; |Text = "注册成功，感谢使用！"
004052C7  |.  8B4D 08        MOV    ECX, [ARG.1]                       ; |
004052CA  |.  51             PUSH   ECX                                ; |hOwner = 0041B870
004052CB  |.  FF15 80424100  CALL   DWORD PTR DS:[<&USER32.MessageBoxA>; \\MessageBoxA
004052D1  |.  EB 16          JMP    SHORT STANDARD.
```

以下是这个 `CALL` 的代码，追进后：

```
0040322D  /$  55             PUSH   EBP\----停在此!
0040322E  |.  8BEC           MOV    EBP, ESP
00403230  |.  81EC DC000000  SUB    ESP, 0DC
00403236  |.  898D 24FFFFFF  MOV    [LOCAL.55], ECX                   ;  STANDARD.0041B870
0040323C  |.  6A 29          PUSH   29                                 ; /Arg2 = 00000029
0040323E  |.  8D85 28FFFFFF  LEA    EAX, [LOCAL.54]                   ; |
00403244  |.  50             PUSH   EAX                                ; |Arg1 = 0068EBE4 ASCII "123456789"
00403245  |.  8B8D 24FFFFFF  MOV    ECX, [LOCAL.55]                   ; |
0040324B  |.  E8 6EFDFFFF    CALL   STANDARD.00402FBE               ; \\STANDARD.00402FBE
00403250  |.  85C0           TEST   EAX, EAX
00403252  |.  75 04          JNZ    SHORT STANDARD.00403258
00403254  |.  33C0           XOR    EAX, EAX
00403256  |.  EB 43          JMP    SHORT STANDARD.0040329B
00403258  |>  8D8D 54FFFFFF  LEA    ECX, [LOCAL.43]
0040325E  |.  51             PUSH   ECX                                ; /Arg2 = 0041B870 ASCII "`CA"
0040325F  |.  8D95 28FFFFFF  LEA    EDX, [LOCAL.54]                   ; |
00403265  |.  52             PUSH   EDX                                ; |Arg1 = 0068EBE4 ASCII "123456789"
00403266  |.  8B8D 24FFFFFF  MOV    ECX, [LOCAL.55]                   ; |
0040326C  |.  E8 CA010000    CALL   STANDARD.0040343B               ; \\STANDARD.0040343B---算法 CALL1，按 F7 追进此 CALL!
00403271  |.  8B45 08        MOV    EAX, [ARG.1]
00403274  |.  50             PUSH   EAX                                ; /Arg1 = 0068EBE4 ASCII "123456789"
00403275  |.  E8 DA330000    CALL   STANDARD.00406654               ; \\STANDARD.00406654
0040327A  |.  83C4 04        ADD    ESP, 4
0040327D  |.  8D8D 54FFFFFF  LEA    ECX, [LOCAL.43]
00403283  |.  51             PUSH   ECX                                ; /String2 = "`CA"
00403284  |.  8B55 08        MOV    EDX, [ARG.1]                       ; |
00403287  |.  52             PUSH   EDX                                ; |String1 = "123456789"
00403288  |.  FF15 B8404100   CALL   DWORD PTR DS:[<&KERNEL32.lstrcmpA>; \\lstrcmpA
0040328E  |.  85C0           TEST   EAX, EAX
00403290  |.  75 07          JNZ    SHORT STANDARD.00403299
```

以下是算法 `CALL1` 的代码，追进后：

```
0040343B  /$  55             PUSH   EBP\--追进后，停在此!
0040343C  |.  8BEC           MOV    EBP, ESP
0040343E  |.  81EC E8000000  SUB    ESP, 0E8
00403444  |.  56             PUSH   ESI
00403445  |.  57             PUSH   EDI
00403446  |.  898D 18FFFFFF  MOV    [LOCAL.58], ECX
0040344C  |.  A1 94624100    MOV    EAX, DWORD PTR DS:[416294]
00403451  |.  8985 1CFFFFFF  MOV    [LOCAL.57], EAX
00403457  |.  8B0D 98624100   MOV    ECX, DWORD PTR DS:[416298]
0040345D  |.  898D 20FFFFFF  MOV    [LOCAL.56], ECX
00403463  |.  8A15 9C624100   MOV    DL, BYTE PTR DS:[41629C]
00403469  |.  8895 24FFFFFF   MOV    BYTE PTR SS:[EBP-DC], DL
0040346F  |.  8B85 1CFFFFFF   MOV    EAX, [LOCAL.57]
00403475  |.  8985 40FFFFFF   MOV    [LOCAL.48], EAX
0040347B  |.  8B8D 20FFFFFF   MOV    ECX, [LOCAL.56]
00403481  |.  898D 44FFFFFF   MOV    [LOCAL.47], ECX
00403487  |.  C745 FC 000000  MOV    [LOCAL.1], 0
0040348E  |.  837D 08 00      CMP    [ARG.1], 0
00403492  |.  75 07           JNZ    SHORT STANDARD.0040349B
00403494  |.  33C0            XOR    EAX, EAX
00403496  |.  E9 F0000000     JMP    STANDARD.0040358B
0040349B  |>  68 AB000000     PUSH   0AB
004034A0  |.  6A 00           PUSH   0
004034A2  |.  8D95 50FFFFFF   LEA    EDX, [LOCAL.44]
004034A8  |.  52              PUSH   EDX
004034A9  |.  E8 22730000     CALL   STANDARD.0040A7D0
004034AE  |.  83C4 0C         ADD    ESP, 0C
004034B1  |.  C785 3CFFFFFF   MOV    [LOCAL.49], 0
004034BB  |.  EB 0F           JMP    SHORT STANDARD.004034CC
004034BD  |>  8B85 3CFFFFFF   /MOV    EAX, [LOCAL.49]------这里开始计算注册码!
004034C3  |.  83C0 04         |ADD    EAX, 4--------------- EAX=EAX+4,这说明每次取四位机器码进行计算！
004034C6  |.  8985 3CFFFFFF   |MOV    [LOCAL.49], EAX
004034CC  |>  83BD 3CFFFFFF   > CMP    [LOCAL.49], 28-------与 40 比较，28 的十六进制值就是 40，因为机器码是 40 位!
004034D3  |.  0F8D 82000000   |JGE    STANDARD.0040355B----大于等于就转移!当然没取完就得继续!
004034D9  |.  8B4D 08         |MOV    ECX, [ARG.1]---------把机器码送 ECX!我的机器码是："{2181533-91212-94319-592381135-11335993}"
004034DC  |.  038D 3CFFFFFF   |ADD    ECX, [LOCAL.49]
004034E2  |.  8B11            |MOV    EDX, DWORD PTR DS:[ECX]----取机器码计算!并经过转换!
004034E4  |.  8955 FC         |MOV    [LOCAL.1], EDX\--------取机器码的第一组前四位：{218，经
```



过转换为：`3831327B`! 转换是这样的：取各位的 ASCII 码，`{`为`7B`，`2`为`32`,`1`为`31`,`8`为`38`，然后从最后一位开始

，把它们组合起来就是：`3831327B`!

```
004034E7  |.  8B45 FC       |MOV     EAX, [LOCAL.1]-------把机器码送 EAX!EAX 为 3831327B

004034EA  |.  33C9          |XOR     ECX, ECX\--------------ECX 清零!

004034EC  |.  8B95 44FFFFFF |MOV     EDX, [LOCAL.47]-------把常数：6B636F6C 送 EDX!这个常数是

参与计算注册码的!

004034F2  |.  52           |PUSH    EDX\-------------------EDX 入栈!

004034F3  |.  8B95 40FFFFFF |MOV     EDX, [LOCAL.48]-------把常数：6374656E 送 EDX!这个常数也

是参与计算注册码的!

004034F9  |.  52           |PUSH    EDX\-------------------常数：6374656E 也入栈!

004034FA  |.  51           |PUSH    ECX

004034FB  |.  50           |PUSH    EAX

004034FC  |.  E8 8F770000   |CALL    STANDARD.0040AC90----算法 CALL2，要追进!

00403501  |.  81CA 00000040 |OR      EDX, 40000000--------EDX 或 40000000=0E0A33F7 OR 4000000

0=4E0A33F7，这一步我们设为算法 3

00403507  |.  8985 48FFFFFF |MOV     [LOCAL.46], EAX\------把 EAX 送入[LOCAL.46]=FECA37DA

0040350D  |.  8995 4CFFFFFF |MOV     [LOCAL.45], EDX\------把 EDX 送入[LOCAL.45]=4E0A33F7

00403513  |.  8D85 28FFFFFF |LEA     EAX, [LOCAL.54]

00403519  |.  50           |PUSH    EAX                             ; /Arg3 = 0068EA34

0040351A  |.  8B8D 4CFFFFFF |MOV     ECX, [LOCAL.45]                 ; |

00403520  |.  51           |PUSH    ECX                             ; |Arg2 = 00000000

00403521  |.  8B95 48FFFFFF |MOV     EDX, [LOCAL.46]                 ; |

00403527  |.  52           |PUSH    EDX                             ; |Arg1 = 00000000

00403528  |.  E8 74FDFFFF   |CALL    STANDARD.004032A1               ; \STANDARD.004032A1--

\-此 CALL 把[LOCAL.45]和[LOCAL.46]的值连接起来组成第一组注册码，注意字母大写转换成小写：4e0a33f

7feca37da

0040352D  |.  83C4 0C       |ADD     ESP, 0C

00403530  |.  8D85 28FFFFFF |LEA     EAX, [LOCAL.54]

00403536  |.  50           |PUSH    EAX                             ; /StringToAdd = ""

00403537  |.  8D8D 50FFFFFF |LEA     ECX, [LOCAL.44]                 ; |

0040353D  |.  51           |PUSH    ECX                             ; |ConcatString = NULL

0040353E  |.  FF15 9C404100 |CALL    DWORD PTR DS:[<&KERNEL32.lstrca>; \lstrcatA

00403544  |.  68 A0624100   |PUSH    STANDARD.004162A0               ; /StringToAdd = " "

00403549  |.  8D95 50FFFFFF |LEA     EDX, [LOCAL.44]                 ; |

0040354F  |.  52           |PUSH    EDX                             ; |ConcatString = NULL

00403550  |.  FF15 9C404100 |CALL    DWORD PTR DS:[<&KERNEL32.lstrca>; \lstrcatA

00403556  |.^ E9 62FFFFFF   \JMP     STANDARD.004034BD------跳回继续!直至机器码取完为止!

0040355B  |>  C785 3CFFFFFF >MOV     [LOCAL.49], 0A9

00403565  |.  8B85 3CFFFFFF MOV     EAX, [LOCAL.49]

0040356B  |.  C68405 50FFFFF>MOV     BYTE PTR SS:[EBP+EAX-B0], 0

00403573  |.  B9 2A000000   MOV     ECX, 2A

00403578  |.  8DB5 50FFFFFF LEA     ESI, [LOCAL.44]

0040357E  |.  8B7D 0C       MOV     EDI, [ARG.2]

00403581  |.  F3:A5         REP     MOVS DWORD PTR ES:[EDI], DWORD P>

00403583  |.  66:A5        MOVS    WORD PTR ES:[EDI], WORD PTR DS:[>

00403585  |.  A4          MOVS    BYTE PTR ES:[EDI], BYTE PTR DS:[>

00403586  |.  B8 01000000  MOV     EAX, 1

0040358B  |>  5F          POP     EDI                             ;  0068F018

0040358C  |.  5E          POP     ESI                             ;  0068F018

0040358D  |.  8BE5        MOV     ESP, EBP

0040358F  |.  5D          POP     EBP                             ;  0068F018

00403590  \\.  C2 0800     RETN    8
```

以下是算法 `CALL2` 的代码，追进后：

```
0040AC90  /$  8B4424 08    MOV     EAX, DWORD PTR SS:[ESP+8]----  EAX 清零!

0040AC94  |.  8B4C24 10    MOV     ECX, DWORD PTR SS:[ESP+10]---把常数 6B636FC 送入 ECX

0040AC98  |.  0BC8         OR      ECX, EAX\----ECX 或 EAX，ECX 为 6B636FC！

0040AC9A  |.  8B4C24 0C    MOV     ECX, DWORD PTR SS:[ESP+C]----把常数 6374656E 移入 ECX

0040AC9E  |.  75 09        JNZ     SHORT STANDARD.0040ACA9------不等则跳！

0040ACA0  |.  8B4424 04    MOV     EAX, DWORD PTR SS:[ESP+4]      ;  STANDARD.00403501

0040ACA4  |.  F7E1         MUL     ECX

0040ACA6  |.  C2 1000      RETN    10

0040ACA9  |>  53          PUSH    EBX－－－－－－－跳到此处！

0040ACAA  |.  F7E1        MUL     ECX\----ECX=ECX*1=6374656E*1=6374656E

0040ACAC  |.  8BD8        MOV     EBX, EAX\---------EBX 清零!

0040ACAE  |.  8B4424 08   MOV     EAX, DWORD PTR SS:[ESP+8]---把第一组机器码送入 EAX：38313

27B

0040ACB2  |.  F76424 14   MUL     DWORD PTR SS:[ESP+14]---6B636FC*3831327B=F835A0E4 r 1792

6394，高位送入 EAX，低位送入 EDX，故 EAX 的值为：F835A0E4；EDX 的值为：17926394！

0040ACB6  |.  03D8        ADD     EBX, EAX\--－－－－－EBX=EBX+EAX=F835A0E4+0=F835A0E4

0040ACB8  |.  8B4424 08   MOV     EAX, DWORD PTR SS:[ESP+8]-----把 3831327B 送入 EAX

0040ACBC  |.  F7E1        MUL     ECX\---－－－－－－－ECX=ECX*3831327B=6374656E*3831327B=FECA37DA r 15D49

313，高位送入 EAX，低位送入 EDX，故 EAX 的值变为：FECA37DA；EDX 的值为：15D49313！

0040ACBE  |.  03D3        ADD     EDX, EBX\--－－－EDX=EBX＋EDX＝F835A0E4+15D49313=0E0A33F7，EDX 的值参与第 3 步的运算！

0040ACC0  |.  5B          POP     EBX                             ;  0068F064
```

－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－－

算法基本搞清楚，下面进行总结：

1、每次取机器码的四位进行计算，如我的机器码是：`{2181533-91212-94319-592381135-11335993}`

第 1 次四位是：`{218`；第 2 次四位是：`1533`；第 3 次四位是：`-912`；第 4 次四位是：`12-9`

第 5 次四位是：`4319`；第 6 次四位是：`-592`；第 7 次四位是：`3811`；第 8 次四位是：`35-1`

第 9 次四位是：`1335`　第 10 次四位是：`993}`。然后对取出的四位机器码进行转换，转换是这样的：先取每位机器码的 ASCII 码，然后从最

后一位开始把它组合起来形成一组新值，这组新值参与计算注册码！如：第 1 次四位是：`{218`，取各位的 ASCII 码为：`{`为`7B`，`2`为`32`,`1`为

`31`,`8`为`38`，把它们组合起来是：`3831327B`!第 10 次四位是：`993}`，取各位的 ASCII 码为：`9`为`39`,`9`为`39`,`3`为`33`，`}`为`7D`，把它们组合起来就

是：`7D333939`。

2、第二步运算是：把四位机器码的新值与两个常数：`6B636F6C`和`6374656E`进行计算形成 16 位一组的注册码，故注册码是 160 位，又长又

臭。前 8 位的注册码(我们假定是`A`，机器码的转换值我们假定为`C`)，它的计算是这样的：`A=(6B636F6C*C 的高 8 位+6374656E*C 的低 8 位) OR

40000000`，如第一组的前 8 位计算是：`A=(6B636F6C*3831327B 的高 8 位+6374656E*3831327B 的低 8 位) OR 40000000=(F835A0E4+15D49313)

OR 40000000=0E0A33F7 OR 40000000=4E0A33F7`，最后一组注册码的计算：`A=(6B636F6C*7D333939 的高 8 位+6374656E*7D333939 的低 8 位)

OR 40000000=(FB76DB0C+30A3BBF1) OR 40000000=2C1A96FD OR 40000000=6C1A96FD`；

后 8 位的注册码(我们假定是`B`)，`B=6374656E*C 的高 8 位`。如第一组的后 8 位：`B=6374656E*3831327B 的高 8 位=FECA37DA`；

最后一组的后 8 位：`B=6374656E*7D333939 的高 8 位=FA6A137E`，把`A`、`B`组合起来构成一组 16 位的注册码，注意大写字母转换成小写字母；

即每组注册码由`AB`构成，如我的第一组注册码就是：`4e0a33f7feca37da`;我的最后一组注册码就是：`6c1a96fdfa6a137e`.每组注册码之间

空一格。

破解小结：

这个软件破解是非常容易的，是明码!但研究它的算法却费了我不少时间，搞清楚它的算法，对我等菜鸟来说，很有帮助!

`4dd466d0e1e0cc123afbe504da2462ca`
