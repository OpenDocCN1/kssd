
# GIF2SWF2.1

> 原文：[`www.pediy.com/kssd/pediy04/kanxue281.htm`](https://www.pediy.com/kssd/pediy04/kanxue281.htm)

用 `W32dasm`

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00404164(C)
|
:0040416F 83FB01                   cmp ebx, 00000001
:00404172 0F85AD000000             jne 00404225
:00404178 E8B3F5FFFF               call 00403730  <--关键 CALL，进入。
:0040417D 85C0                     test eax, eax
:0040417F 7463                     je 004041E4  <---来到这
:00404181 6A00                     push 00000000
:00404183 8BCF                     mov ecx, edi
:00404185 891D6CC54100             mov dword ptr [0041C56C], ebx

* Reference To: MFC42.Ordinal:0A55, Ord:0A55h
                        |
:0040418B E81AB30000               Call 0040F4AA
:00404190 A168C54100               mov eax, dword ptr [0041C568]
:00404195 85C0                     test eax, eax
:00404197 742B                     je 004041C4
:00404199 A1E03A4100               mov eax, dword ptr [00413AE0]
:0040419E 6A00                     push 00000000
:004041A0 6A00                     push 00000000
:004041A2 6810010000               push 00000110
:004041A7 50                       push eax

* Reference To: USER32.SendMessageA, Ord:0214h
                        |
:004041A8 FF1500044100             Call dword ptr [00410400]
:004041AE 6A40                     push 00000040

* Possible StringData Ref from Data Obj ->"Congratulations"
                        |
:004041B0 6800374100               push 00413700

* Possible StringData Ref from Data Obj ->"GIF2SWF has been successfuly registered "
                        ->"!"
                        |
:004041B5 68D4364100               push 004136D4
:004041BA 6A00                     push 00000000

* Reference To: USER32.MessageBoxA, Ord:01BEh
                        |
:004041BC FF15FC034100             Call dword ptr [004103FC]
:004041C2 EB61                     jmp 00404225

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00404197(C)
|
:004041C4 6A40                     push 00000040

* Possible StringData Ref from Data Obj ->"Congratulations"
                        |
:004041C6 6800374100               push 00413700

* Possible StringData Ref from Data Obj ->"GIF2SWF has been successfuly registered "
                        ->"!"
                        |
:004041CB 68D4364100               push 004136D4
:004041D0 6A00                     push 00000000

* Reference To: USER32.MessageBoxA, Ord:01BEh
                        |
:004041D2 FF15FC034100             Call dword ptr [004103FC]
:004041D8 6A00                     push 00000000
:004041DA E871F0FFFF               call 00403250
:004041DF 83C404                   add esp, 00000004
:004041E2 EB41                     jmp 00404225

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040417F(C)<-------从 0040417F 跳过来的，向上找。
|
:004041E4 6A30                     push 00000030

* Possible Reference to Dialog:
                        |
:004041E6 68C4364100               push 004136C4

* Possible StringData Ref from Data Obj ->"You have entered and incorrect "
                        ->"Name or Serial number
Please "
                        ->"try again !"
                        |
:004041EB 6878364100               push 00413678
```

来到 `call 00403730` 中

```
:00403730 6AFF                     push FFFFFFFF
:00403732 68F8F94000               push 0040F9F8
:00403737 64A100000000             mov eax, dword ptr fs:[00000000]
:0040373D 50                       push eax
:0040373E 64892500000000           mov dword ptr fs:[00000000], esp
:00403745 83EC08                   sub esp, 00000008
:00403748 51                       push ecx
:00403749 8BCC                     mov ecx, esp
:0040374B 89642404                 mov dword ptr [esp+04], esp
:0040374F 68D83A4100               push 00413AD8

* Reference To: MFC42.Ordinal:0217, Ord:0217h
                        |
:00403754 E827BD0000               Call 0040F480
:00403759 51                       push ecx
:0040375A C744241800000000         mov [esp+18], 00000000
:00403762 8BCC                     mov ecx, esp
:00403764 8964240C                 mov dword ptr [esp+0C], esp
:00403768 68DC3A4100               push 00413ADC

* Reference To: MFC42.Ordinal:0217, Ord:0217h
                        |
:0040376D E80EBD0000               Call 0040F480
:00403772 C7442418FFFFFFFF         mov [esp+18], FFFFFFFF
:0040377A E831FEFFFF               call 004035B0 〈-注册码比较 CALL
:0040377F 83C408                   add esp, 00000008
:00403782 85C0                     test eax, eax
:00403784 7544                     jne 004037CA
```

来到 `call 004035B0` 中。

```
:004035B0 6AFF                     push FFFFFFFF
:004035B2 68E0F94000               push 0040F9E0
:004035B7 64A100000000             mov eax, dword ptr fs:[00000000]
:004035BD 50                       push eax
:004035BE 64892500000000           mov dword ptr fs:[00000000], esp
:004035C5 83EC68                   sub esp, 00000068
:004035C8 53                       push ebx
:004035C9 56                       push esi
:004035CA 57                       push edi
:004035CB 8D8C2484000000           lea ecx, dword ptr [esp+00000084]
:004035D2 C744247C01000000         mov [esp+7C], 00000001

* Reference To: MFC42.Ordinal:106C, Ord:106Ch
                        |
:004035DA E8B7BD0000               Call 0040F396
:004035DF 8D8C2484000000           lea ecx, dword ptr [esp+00000084]

* Reference To: MFC42.Ordinal:104B, Ord:104Bh
                        |
:004035E6 E88FBE0000               Call 0040F47A
:004035EB 8D8C2488000000           lea ecx, dword ptr [esp+00000088]
:004035F2 8BF0                     mov esi, eax

* Reference To: MFC42.Ordinal:104B, Ord:104Bh
                        |
:004035F4 E881BE0000               Call 0040F47A
:004035F9 8BD8                     mov ebx, eax
:004035FB 8BFE                     mov edi, esi
:004035FD 83C9FF                   or ecx, FFFFFFFF
:00403600 33C0                     xor eax, eax
:00403602 F2                       repnz
:00403603 AE                       scasb
:00403604 F7D1                     not ecx
:00403606 49                       dec ecx
:00403607 895C240C                 mov dword ptr [esp+0C], ebx
:0040360B 0F84DC000000             je 004036ED
:00403611 8BFB                     mov edi, ebx
:00403613 83C9FF                   or ecx, FFFFFFFF
:00403616 F2                       repnz
:00403617 AE                       scasb
:00403618 F7D1                     not ecx
:0040361A 49                       dec ecx
:0040361B 0F84CC000000             je 004036ED
:00403621 55                       push ebp
:00403622 8BFE                     mov edi, esi
:00403624 83C9FF                   or ecx, FFFFFFFF
:00403627 33ED                     xor ebp, ebp
:00403629 F2                       repnz
:0040362A AE                       scasb
:0040362B F7D1                     not ecx
:0040362D 49                       dec ecx
:0040362E 83F920                   cmp ecx, 00000020
:00403631 7E05                     jle 00403638
:00403633 B920000000               mov ecx, 00000020

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00403631(C)
|
:00403638 33F6                     xor esi, esi
```
