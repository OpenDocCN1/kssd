
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
