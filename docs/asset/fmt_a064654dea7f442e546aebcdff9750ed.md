
# BlindRead

> 原文：[`www.pediy.com/kssd/pediy02/forum142.htm`](https://www.pediy.com/kssd/pediy02/forum142.htm)

哪位高手能破解 `BlindWrite` 和 `Search&Replace`？

`BlindWrite`: `ftp://ftp02.softhouse.com.cn/download3/31892BW_Install.exe`

`Search&Replace`: `http://go.163.com/~nihility/down/other/sr32.zip`

最好能把破解过程贴出来，谢谢！

几乎把 `FrogsICE` 中提到的检测 `SoftICE` 的方法都用上了。

* Referenced by a CALL at Address:

`:00492F15`

|

```
:00492B88 6A00                       push 00000000
:00492B8A 6A00                       push 00000000
:00492B8C 6A03                       push 00000003
:00492B8E 6A00                       push 00000000
:00492B90 6A02                       push 00000002
:00492B92 6800000080                 push 80000000

* Possible StringData Ref from Code Obj ->"\\.\\NTICE"
                                |
:00492B97 683C2C4900             push 00492C3C

* Reference To: KERNEL32.CreateFileA, Ord:0000h
                                |
:00492B9C E84B39F7FF            Call 004064EC
:00492BA1 83F8FF                cmp eax, FFFFFFFF
:00492BA4 741D                  je 00492BC3
:00492BA6 6A10                  push 00000010

* Possible StringData Ref from Code Obj ->"Error"
                                |
:00492BA8 68482C4900           push 00492C48

* Possible StringData Ref from Code Obj ->"This software is not designed "
                                  ->"to operate under a debugger. The "
                                  ->"application will stop."
                                |
:00492BAD 68502C4900           push 00492C50
:00492BB2 6A00                  push 00000000

* Reference To: user32.MessageBoxA, Ord:0000h
                                |
:00492BB4 E86B41F7FF            Call 00406D24
:00492BB9 B801000000           mov eax, 00000001
:00492BBE E83110F7FF            call 00403BF4

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00492BA4(C)
|
:00492BC3 6A00                   push 00000000
:00492BC5 6A00                   push 00000000
:00492BC7 6A03                   push 00000003
:00492BC9 6A00                   push 00000000
:00492BCB 6A02                   push 00000002
:00492BCD 6800000080             push 80000000

* Possible StringData Ref from Code Obj ->"\\.\\SIWVID"
                                |
:00492BD2 68A82C4900            push 00492CA8

* Reference To: KERNEL32.CreateFileA, Ord:0000h
                                |
:00492BD7 E81039F7FF            Call 004064EC
:00492BDC 83F8FF                cmp eax, FFFFFFFF
:00492BDF 741D                  je 00492BFE
:00492BE1 6A10                  push 00000010

* Possible StringData Ref from Code Obj ->"Error"
                                |
:00492BE3 68482C4900           push 00492C48

* Possible StringData Ref from Code Obj ->"This software is not designed "
                                  ->"to operate under a debugger. The "
                                  ->"application will stop."
                                |
:00492BE8 68502C4900           push 00492C50
:00492BED 6A00                  push 00000000

* Reference To: user32.MessageBoxA, Ord:0000h
                                |
:00492BEF E83041F7FF            Call 00406D24
:00492BF4 B801000000           mov eax, 00000001
:00492BF9 E8F60FF7FF            call 00403BF4

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00492BDF(C)
|
:00492BFE 6A00                   push 00000000
:00492C00 6A00                   push 00000000
:00492C02 6A03                   push 00000003
:00492C04 6A00                   push 00000000
:00492C06 6A02                   push 00000002
:00492C08 6800000080             push 80000000

* Possible StringData Ref from Code Obj ->"\\.\\SICE"
                                |
:00492C0D 68B42C4900            push 00492CB4

* Reference To: KERNEL32.CreateFileA, Ord:0000h
                                |
:00492C12 E8D538F7FF            Call 004064EC
:00492C17 83F8FF                cmp eax, FFFFFFFF
:00492C1A 741D                  je 00492C39
:00492C1C 6A10                  push 00000010

* Possible StringData Ref from Code Obj ->"Error"
                                |
:00492C1E 68482C4900           push 00492C48

* Possible StringData Ref from Code Obj ->"This software is not designed "
                                  ->"to operate under a debugger. The "
                                  ->"application will stop."
                                |
:00492C23 68502C4900           push 00492C50
:00492C28 6A00                  push 00000000

* Reference To: user32.MessageBoxA, Ord:0000h
                                |
:00492C2A E8F540F7FF            Call 00406D24
:00492C2F B801000000           mov eax, 00000001
:00492C34 E8BB0FF7FF            call 00403BF4

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00492C1A(C)
|
:00492C39 C3                     ret
```

```
:0046441A B057                    mov al, 57
:0046441C BF00000100              mov edi, 00010000
:00464421 B900003F00              mov ecx, 003F0000
:00464426 33D2                    xor edx, edx
:00464428 6844444600              push 00464444
:0046442D 64FF32                  push dword ptr fs:[edx]
:00464430 892524AF4900            mov dword ptr [0049AF24], esp
:00464436 892D28AF4900            mov dword ptr [0049AF28], ebp
:0046443C 648922                  mov dword ptr fs:[edx], esp
:0046443F EB1A                    jmp 0046445B
:00464441 90                      nop
:00464442 90                      nop
:00464443 90                      nop
:00464444 8B2524AF4900            mov esp, dword ptr [0049AF24]
:0046444A 8B2D28AF4900            mov ebp, dword ptr [0049AF28]
:00464450 5A                      pop edx
:00464451 646789160000            mov fs:[0000], edx
:00464457 58                      pop eax
:00464458 33C0                    xor eax, eax
:0046445A C3                      ret
```

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:0046443F(U), :00464472(U), :0046447D(C)

|

```
:0046445B F2                       repnz
:0046445C AE                       scasb
:0046445D E325                     jcxz 00464484
:0046445F 90                       nop
:00464460 90                       nop
:00464461 90                       nop
:00464462 90                       nop
:00464463 90                       nop
:00464464 90                       nop
:00464465 90                       nop
:00464466 90                       nop
:00464466 813F494E4943            cmp dword ptr [edi], 43494E49
:0046446C 7406                     je 00464474
:0046446E 90                       nop
:0046446F 90                       nop
:00464470 90                       nop
:00464471 90                       nop
:00464472 EBE7                     jmp 0046445B
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0046446C(C)

|

```
:00464474 83C704                   add edi, 00000004
:00464477 813F452E4252             cmp dword ptr [edi], 52422E45
:0046447D 75DC                     jne 0046445B
:0046447F B801000000               mov eax, 00000001
:00464484 5A                       pop edx
:00464485 646789160000            mov fs:[0000], edx
:0046448B 5A                       pop edx
:0046448C C3                       ret
```

`;--------------------------------------------------------------------`
