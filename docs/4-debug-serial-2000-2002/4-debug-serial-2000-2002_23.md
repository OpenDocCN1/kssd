
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



*   Referenced by a CALL at Address:
    *   `:004643CA`

*   Referenced by a (U)nconditional or (C)onditional Jump at Address:
    *   `:004644A6(U)`

```asm
:0046448D 33D2                xor edx, edx
:0046448F 68AB444600          push 004644AB
:00464494 64FF32              push dword ptr fs:[edx]
:00464497 892524AF4900        mov dword ptr [0049AF24], esp
:0046449D 892D28AF4900        mov dword ptr [0049AF28], ebp
:004644A3 648922              mov dword ptr fs:[edx], esp
:004644A6 EB1A                jmp 004644C2
:004644A8 90                  nop
:004644A9 90                  nop
:004644AA 90                  nop
:004644AB 8B2524AF4900        mov esp, dword ptr [0049AF24]
:004644B1 8B2D28AF4900        mov ebp, dword ptr [0049AF28]
:004644B7 5A                  pop edx
:004644B8 646789160000        mov fs:[0000], edx
:004644BE 58                  pop eax
:004644BF 33C0                xor eax, eax
:004644C1 C3                  ret

:004644C2 B804000000          mov eax, 00000004
:004644C7 BD4B484342          mov ebp, 4243484B
:004644CC CC                  int 03
:004644CD 5A                  pop edx
:004644CE 646789160000        mov fs:[0000], edx
:004644D4 5A                  pop edx
:004644D5 B801000000          mov eax, 00000001
:004644DA C3                  ret
```

---

*   Referenced by a CALL at Address:
    *   `:004643B5`

*   Referenced by a (U)nconditional or (C)onditional Jump at Address:
    *   `:004644F4(U)`

```asm
:004644DB 33D2                xor edx, edx
:004644DD 68F9444600          push 004644F9
:004644E2 64FF32              push dword ptr fs:[edx]
:004644E5 892524AF4900        mov dword ptr [0049AF24], esp
:004644EB 892D28AF4900        mov dword ptr [0049AF28], ebp
:004644F1 648922              mov dword ptr fs:[edx], esp
:004644F4 EB1A                jmp 00464510
:004644F6 90                  nop
:004644F7 90                  nop
:004644F8 90                  nop
:004644F9 8B2524AF4900        mov esp, dword ptr [0049AF24]
:004644FF 8B2D28AF4900        mov ebp, dword ptr [0049AF28]
:00464505 5A                  pop edx
:00464506 646789160000        mov fs:[0000], edx
:0046450C 58                  pop eax
:0046450D 33C0                xor eax, eax
:0046450F C3                  ret

:00464510 B443                mov ah, 43
:00464512 CD68                int 68
:00464514 5A                  pop edx
:00464515 646789160000        mov fs:[0000], edx
:0046451B 5A                  pop edx
:0046451C 663D86F3            cmp ax, F386
:00464520 7407                je 00464529
:00464522 90                  nop
:00464523 90                  nop
:00464524 90                  nop
:00464525 90                  nop
:00464526 33C0                xor eax, eax
:00464528 C3                  ret
```

*   Referenced by a (U)nconditional or (C)onditional Jump at Address:
    *   `:00464520(C)`

```asm
:00464529 B801000000          mov eax, 00000001
:0046452E C3                  ret
```

---

*   Referenced by a CALL at Address:
    *   `:004643EF`

*   Referenced by a (U)nconditional or (C)onditional Jump at Address:
    *   `:00464548(U)`

*   Possible StringData Ref from Data Obj -> `C:\ntice\nmtrans.dll`

*   Reference To: `KERNEL32.LoadLibraryA`, Ord:0000h

*   Possible StringData Ref from Data Obj -> `NmSymIsSoftICELoaded`

*   Reference To: `KERNEL32.GetProcAddress`, Ord:0000h

*   Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
    *   `:004645A7(C), :004645BA(C)`

*   Possible StringData Ref from Data Obj -> `C:\ntice\nmtrans.dll`

*   Reference To: `KERNEL32.LoadLibraryA`, Ord:0000h

*   Possible StringData Ref from Data Obj -> `NmSymIsSoftICELoaded`

*   Reference To: `KERNEL32.GetProcAddress`, Ord:0000h

```asm
:00464564 0F010D2CAF4900      sidt [0049AF2C]
:0046456B A12EAF4900          mov eax, dword ptr [0049AF2E]
:00464570 83C008              add eax, 00000008
:00464573 8B18                mov ebx, dword ptr [eax]
:00464575 83C010              add eax, 00000010
:00464578 8B00                mov eax, dword ptr [eax]
:0046457A 25FFFF0000          and eax, 0000FFFF
:0046457F 81E3FFFF0000        and ebx, 0000FFFF
:00464585 2BC3                sub eax, ebx
:00464587 83F81E              cmp eax, 0000001E
:0046458A 7406                je 00464592
:0046458C 90                  nop
:0046458D 90                  nop
:0046458E 90                  nop
:0046458F 90                  nop
:00464590 33C0                xor eax, eax

:00464592 5A                  pop edx
:00464593 646789160000        mov fs:[0000], edx
:00464599 5A                  pop edx
:0046459A C3                  ret
```

*   Referenced by a CALL at Address:
    *   `:00464404`

*   Possible StringData Ref from Data Obj -> `C:\ntice\nmtrans.dll`

*   Reference To: `KERNEL32.LoadLibraryA`, Ord:0000h

*   Possible StringData Ref from Data Obj -> `NmSymIsSoftICELoaded`

*   Reference To: `KERNEL32.GetProcAddress`, Ord:0000h

*   Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
    *   `:004645A7(C), :004645BA(C)`

```asm
:0046459B 683DAF4900          push 0049AF3D
:004645A0 E8BF20FAFF          Call 00406664
:004645A5 85C0                test eax, eax
:004645A7 7419                je 004645C2
:004645A9 90                  nop
:004645AA 90                  nop
:004645AB 90                  nop
:004645AC 90                  nop
:004645AD 6852AF4900          push 0049AF52
:004645B2 50                  push eax
:004645B3 E81420FAFF          Call 004065CC
:004645B8 85C0                test eax, eax
:004645BA 7406                je 004645C2
:004645BC 90                  nop
:004645BD 90                  nop
:004645BE 90                  nop
:004645BF 90                  nop
:004645C0 FFD0                call eax
:004645C2 C3                  ret
```

和（2）完全一样的代码，难怪现在的软件奇大无比，实在是垃圾太多。

```asm
:00492D32 B057                mov al, 57
:00492D34 BF00000100          mov edi, 00010000
:00492D39 B900003F00          mov ecx, 003F0000
:00492D3E 33D2                xor edx, edx
:00492D40 685C2D4900          push 00492D5C
:00492D45 64FF32              push dword ptr fs:[edx]
:00492D48 8925D0A84A00        mov dword ptr [004AA8D0], esp
:00492D4E 892DD4A84A00        mov dword ptr [004AA8D4], ebp
:00492D54 648922              mov dword ptr fs:[edx], esp
:00492D57 EB1A                jmp 00492D73
:00492D59 90                  nop
```



### 00492D5A - 00492D72

```
:00492D5A 90                          nop

:00492D5B 90                          nop

:00492D5C 8B25D0A84A00                mov esp, dword ptr [004AA8D0]

:00492D62 8B2DD4A84A00                mov ebp, dword ptr [004AA8D4]

:00492D68 5A                          pop edx

:00492D69 646789160000                mov fs:[0000], edx

:00492D6F 58                          pop eax

:00492D70 33C0                        xor eax, eax

:00492D72 C3                          ret
```

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

| `:00492D57(U), :00492D8A(U), :00492D95(C)` |

### 00492D73 - 00492D8A

```
:00492D73 F2                          repnz

:00492D74 AE                          scasb

:00492D75 E325                        jcxz 00492D9C

:00492D77 90                          nop

:00492D78 90                          nop

:00492D79 90                          nop

:00492D7A 90                          nop

:00492D7B 90                          nop

:00492D7C 90                          nop

:00492D7D 90                          nop

:00492D7E 813F494E4943              cmp dword ptr [edi], 43494E49

:00492D84 7406                        je 00492D8C

:00492D86 90                          nop

:00492D87 90                          nop

:00492D88 90                          nop

:00492D89 90                          nop

:00492D8A EBE7                        jmp 00492D73
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| `:00492D84(C)` |

### 00492D8C - 00492DA3

```
:00492D8C 83C704                      add edi, 00000004

:00492D8F 813F452E4252              cmp dword ptr [edi], 52422E45

:00492D95 75DC                        jne 00492D73

:00492D97 B801000000                  mov eax, 00000001

:00492D9C 5A                          pop edx

:00492D9D 646789160000                mov fs:[0000], edx

:00492DA3 5A                          pop edx

:00492DA4 C3                          ret
```

;--------------------------------------------------------------------

* Referenced by a CALL at Address:

| `:00492CE2` |

### 00492DA5 - 00492DC2

```
:00492DA5 33D2                        xor edx, edx

:00492DA7 68C32D4900                  push 00492DC3

:00492DAC 64FF32                      push dword ptr fs:[edx]

:00492DAF 8925D0A84A00                mov dword ptr [004AA8D0], esp

:00492DB5 892DD4A84A00                mov dword ptr [004AA8D4], ebp

:00492DBB 648922                      mov dword ptr fs:[edx], esp

:00492DBE EB1A                        jmp 00492DDA

:00492DC0 90                          nop

:00492DC1 90                          nop

:00492DC2 90                          nop
```

### 00492DC3 - 00492DD9

```
:00492DC3 8B25D0A84A00                mov esp, dword ptr [004AA8D0]

:00492DC9 8B2DD4A84A00                mov ebp, dword ptr [004AA8D4]

:00492DCF 5A                          pop edx

:00492DD0 646789160000                mov fs:[0000], edx

:00492DD6 58                          pop eax

:00492DD7 33C0                        xor eax, eax

:00492DD9 C3                          ret
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| `:00492DBE(U)` |

### 00492DDA - 00492E00

```
:00492DDA B804000000                  mov eax, 00000004

:00492DDF BD4B484342                  mov ebp, 4243484B

:00492DE4 CC                          int 03

:00492DE5 5A                          pop edx

:00492DE6 646789160000                mov fs:[0000], edx

:00492DEC 5A                          pop edx

:00492DED B801000000                  mov eax, 00000001

:00492DF2 C3                          ret
```

;--------------------------------------------------------------------

* Referenced by a CALL at Address:

| `:00492CCD` |

### 00492DF3 - 00492E0C

```
:00492DF3 33D2                        xor edx, edx

:00492DF5 68112E4900                  push 00492E11

:00492DFA 64FF32                      push dword ptr fs:[edx]

:00492DFD 8925D0A84A00                mov dword ptr [004AA8D0], esp

:00492E03 892DD4A84A00                mov dword ptr [004AA8D4], ebp

:00492E09 648922                      mov dword ptr fs:[edx], esp

:00492E0C EB1A                        jmp 00492E28

:00492E0E 90                          nop

:00492E0F 90                          nop

:00492E10 90                          nop
```

### 00492E11 - 00492E27

```
:00492E11 8B25D0A84A00                mov esp, dword ptr [004AA8D0]

:00492E17 8B2DD4A84A00                mov ebp, dword ptr [004AA8D4]

:00492E1D 5A                          pop edx

:00492E1E 646789160000                mov fs:[0000], edx

:00492E24 58                          pop eax

:00492E25 33C0                        xor eax, eax

:00492E27 C3                          ret
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| `:00492E0C(U)` |

### 00492E28 - 00492E40

```
:00492E28 B443                        mov ah, 43

:00492E2A CD68                        int 68

:00492E2C 5A                          pop edx

:00492E2D 646789160000                mov fs:[0000], edx

:00492E33 5A                          pop edx

:00492E34 663D86F3                    cmp ax, F386

:00492E38 7407                        je 00492E41

:00492E3A 90                          nop

:00492E3B 90                          nop

:00492E3C 90                          nop

:00492E3D 90                          nop

:00492E3E 33C0                        xor eax, eax

:00492E40 C3                          ret
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| `:00492E38(C)` |

### 00492E41 - 00492E46

```
:00492E41 B801000000                  mov eax, 00000001

:00492E46 C3                          ret
```

;--------------------------------------------------------------------

* Referenced by a CALL at Address:

| `:00492D07` |

### 00492E47 - 00492E60

```
:00492E47 33D2                        xor edx, edx

:00492E49 68652E4900                  push 00492E65

:00492E4E 64FF32                      push dword ptr fs:[edx]

:00492E51 8925D0A84A00                mov dword ptr [004AA8D0], esp

:00492E57 892DD4A84A00                mov dword ptr [004AA8D4], ebp

:00492E5D 648922                      mov dword ptr fs:[edx], esp

:00492E60 EB1A                        jmp 00492E7C

:00492E62 90                          nop

:00492E63 90                          nop

:00492E64 90                          nop
```

### 00492E65 - 00492E7B

```
:00492E65 8B25D0A84A00                mov esp, dword ptr [004AA8D0]

:00492E6B 8B2DD4A84A00                mov ebp, dword ptr [004AA8D4]

:00492E71 5A                          pop edx

:00492E72 646789160000                mov fs:[0000], edx

:00492E78 58                          pop eax

:00492E79 33C0                        xor eax, eax

:00492E7B C3                          ret
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| `:00492E60(U)` |

### 00492E7C - 00492EA8

```
:00492E7C 0F010DD8A84A00              sidt [004AA8D8]

:00492E83 A1DAA84A00                  mov eax, dword ptr [004AA8DA]

:00492E88 83C008                      add eax, 00000008

:00492E8B 8B18                        mov ebx, dword ptr [eax]

:00492E8D 83C010                      add eax, 00000010

:00492E90 8B00                        mov eax, dword ptr [eax]

:00492E92 25FFFF0000                  and eax, 0000FFFF

:00492E97 81E3FFFF0000                and ebx, 0000FFFF

:00492E9D 2BC3                        sub eax, ebx

:00492E9F 83F81E                      cmp eax, 0000001E

:00492EA2 7406                        je 00492EAA

:00492EA4 90                          nop

:00492EA5 90                          nop

:00492EA6 90                          nop

:00492EA7 90                          nop

:00492EA8 33C0                        xor eax, eax
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| `:00492EA2(C)` |

### 00492EAA - 00492EB2

```
:00492EAA 5A                          pop edx

:00492EAB 646789160000                mov fs:[0000], edx

:00492EB1 5A                          pop edx

:00492EB2 C3                          ret
```

;--------------------------------------------------------------------

* Referenced by a CALL at Address:

| `:00492D1C` |

* Possible StringData Ref from Data Obj -> `C:\ntice\nmtrans.dll`



`4dd466d0e1e0cc123afbe504da2462ca`

```
:00492EB3 68E9A84A00     push 004AA8E9

* Reference To: KERNEL32.LoadLibraryA, Ord:0000h

:00492EB8 E8A737F7FF     Call 00406664

:00492EBD 85C0          test eax, eax

:00492EBF 7419          je 00492EDA

:00492EC1 90            nop

:00492EC2 90            nop

:00492EC3 90            nop

:00492EC4 90            nop

* Possible StringData Ref from Data Obj ->"NmSymIsSoftICELoaded"

:00492EC5 68FEA84A00     push 004AA8FE

:00492ECA 50            push eax

* Reference To: KERNEL32.GetProcAddress, Ord:0000h

:00492ECB E8FC36F7FF     Call 004065CC

:00492ED0 85C0          test eax, eax

:00492ED2 7406          je 00492EDA

:00492ED4 90            nop

:00492ED5 90            nop

:00492ED6 90            nop

:00492ED7 90            nop

:00492ED8 FFD0         call eax

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:00492EBF(C), :00492ED2(C)

:00492EDA C3            ret
```
