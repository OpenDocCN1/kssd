
# Flash Cut1.7

> 原文：[`www.pediy.com/kssd/pediy06/pediy6235.htm`](https://www.pediy.com/kssd/pediy06/pediy6235.htm)

Flash Cut1.7

ASP 加壳，使用 `AspackDie 1.4` 脱壳。

是 `Delphi6` 的作品。

反汇编查看字符串参考

## ① 注册码输入处

```
:004ADE1E 8BC0                     mov eax, eax
:004ADE20 55                       push ebp
:004ADE21 8BEC                     mov ebp, esp
:004ADE23 33C9                     xor ecx, ecx
:004ADE25 51                       push ecx
:004ADE26 51                       push ecx
:004ADE27 51                       push ecx
:004ADE28 51                       push ecx
:004ADE29 51                       push ecx
:004ADE2A 51                       push ecx
:004ADE2B 51                       push ecx
:004ADE2C 8945FC                   mov dword ptr [ebp-04], eax
:004ADE2F 33C0                     xor eax, eax
:004ADE31 55                       push ebp
:004ADE32 689FDF4A00               push 004ADF9F
:004ADE37 64FF30                   push dword ptr fs:[eax]
:004ADE3A 648920                   mov dword ptr fs:[eax], esp
:004ADE3D 8D55F4                   lea edx, dword ptr [ebp-0C]
:004ADE40 8B45FC                   mov eax, dword ptr [ebp-04]
:004ADE43 8B800C030000             mov eax, dword ptr [eax+0000030C]
:004ADE49 E8DEEDFBFF               call 0046CC2C
:004ADE4E 8B45F4                   mov eax, dword ptr [ebp-0C]
:004ADE51 50                       push eax
:004ADE52 8D55EC                   lea edx, dword ptr [ebp-14]
:004ADE55 8B45FC                   mov eax, dword ptr [ebp-04]
:004ADE58 8B8000030000             mov eax, dword ptr [eax+00000300]
:004ADE5E E8C9EDFBFF               call 0046CC2C
:004ADE63 8B55EC                   mov edx, dword ptr [ebp-14]
:004ADE66 8D4DF0                   lea ecx, dword ptr [ebp-10]
:004ADE69 A1FCE14C00               mov eax, dword ptr [004CE1FC]
:004ADE6E 8B00                     mov eax, dword ptr [eax]
:004ADE70 E887C10100               call 004C9FFC
:004ADE75 8B55F0                   mov edx, dword ptr [ebp-10]-------------------->真注册码 0042599729
:004ADE78 58                       pop eax\---------------------------------------->假注册码
:004ADE79 E8F26DF5FF               call 00404C70---------------------------------->验证
:004ADE7E 0F85CE000000             jne 004ADF52----------------------------------->跳到"注册失败"
:004ADE84 B201                     mov dl, 01
:004ADE86 A1F0C24300               mov eax, dword ptr [0043C2F0]
:004ADE8B E8CCE5F8FF               call 0043C45C
:004ADE90 8945F8                   mov dword ptr [ebp-08], eax
:004ADE93 33C0                     xor eax, eax
:004ADE95 55                       push ebp
:004ADE96 68FCDE4A00               push 004ADEFC
:004ADE9B 64FF30                   push dword ptr fs:[eax]
:004ADE9E 648920                   mov dword ptr fs:[eax], esp
:004ADEA1 BA01000080               mov edx, 80000001
:004ADEA6 8B45F8                   mov eax, dword ptr [ebp-08]
:004ADEA9 E88AE6F8FF               call 0043C538
:004ADEAE B101                     mov cl, 01

* Possible StringData Ref from Code Obj ->"softwaremicrosoftFSC"
                    |
:004ADEB0 BAB4DF4A00               mov edx, 004ADFB4
:004ADEB5 8B45F8                   mov eax, dword ptr [ebp-08]
:004ADEB8 E8BFE7F8FF               call 0043C67C
:004ADEBD 8D55E8                   lea edx, dword ptr [ebp-18]
:004ADEC0 8B45FC                   mov eax, dword ptr [ebp-04]
:004ADEC3 8B800C030000             mov eax, dword ptr [eax+0000030C]
:004ADEC9 E85EEDFBFF               call 0046CC2C
:004ADECE 8B4DE8                   mov ecx, dword ptr [ebp-18]

* Possible StringData Ref from Code Obj ->"RegValue"
                    |
:004ADED1 BAD4DF4A00               mov edx, 004ADFD4
:004ADED6 8B45F8                   mov eax, dword ptr [ebp-08]
:004ADED9 E83AE9F8FF               call 0043C818
:004ADEDE 8B45F8                   mov eax, dword ptr [ebp-08]
:004ADEE1 E822E6F8FF               call 0043C508
:004ADEE6 33C0                     xor eax, eax
:004ADEE8 5A                       pop edx
:004ADEE9 59                       pop ecx
:004ADEEA 59                       pop ecx
:004ADEEB 648910                   mov dword ptr fs:[eax], edx
:004ADEEE 6803DF4A00               push 004ADF03

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004ADF01(U)
|
:004ADEF3 8B45F8                   mov eax, dword ptr [ebp-08]
:004ADEF6 E8555BF5FF               call 00403A50
:004ADEFB C3                       ret
:004ADEFC E9E362F5FF               jmp 004041E4
:004ADF01 EBF0                     jmp 004ADEF3
:004ADF03 6A40                     push 00000040

* Possible StringData Ref from Code Obj ->"注册成功!"
                    |
:004ADF05 68E0DF4A00               push 004ADFE0

* Possible StringData Ref from Code Obj ->"注册成功,谢谢您的注册!"
                    |
:004ADF0A 68ECDF4A00               push 004ADFEC
:004ADF0F 8B45FC                   mov eax, dword ptr [ebp-04]
:004ADF12 E8A956FCFF               call 004735C0
:004ADF17 50                       push eax

* Reference To: user32.MessageBoxA, Ord:0000h
                    |
:004ADF18 E81398F5FF               Call 00407730
:004ADF1D 8D55E4                   lea edx, dword ptr [ebp-1C]
:004ADF20 A178E24C00               mov eax, dword ptr [004CE278]
:004ADF25 8B00                     mov eax, dword ptr [eax]
:004ADF27 E848EAFDFF               call 0048C974
:004ADF2C 8D45E4                   lea eax, dword ptr [ebp-1C]

* Possible StringData Ref from Code Obj ->"(以注册！，谢谢您的注册！)"
                    |
:004ADF2F BA0CE04A00               mov edx, 004AE00C
:004ADF34 E8F36BF5FF               call 00404B2C
:004ADF39 8B55E4                   mov edx, dword ptr [ebp-1C]
:004ADF3C A1FCE14C00               mov eax, dword ptr [004CE1FC]
:004ADF41 8B00                     mov eax, dword ptr [eax]
:004ADF43 E814EDFBFF               call 0046CC5C
:004ADF48 8B45FC                   mov eax, dword ptr [ebp-04]
:004ADF4B E858B7FDFF               call 004896A8
:004ADF50 EB1A                     jmp 004ADF6C

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004ADE7E(C)
|
:004ADF52 6A30                     push 00000030

* Possible StringData Ref from Code Obj ->"注册失败!"
                    |
:004ADF54 6828E04A00               push 004AE028

* Possible StringData Ref from Code Obj ->"注册失败,请确认您的注册码是否正确!"
                    |                                     /8B55F0      MOV edx, dword ptr[ebp-10]
:004ADF59 6834E04A00               push 004AE034--------------------------------->|52         Push EDX
:004ADF5E 8B45FC                   mov eax, dword ptr [ebp-04]                    90         Nop
:004ADF61 E85A56FCFF               call 004735C0
:004ADF66 50                       push eax

* Reference To: user32.MessageBoxA, Ord:0000h
                    |
:004ADF67 E8C497F5FF               Call 00407730

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004ADF50(U)
|
:004ADF6C 33C0                     xor eax, eax
:004ADF6E 5A                       pop edx
:004ADF6F 59                       pop ecx
:004ADF70 59                       pop ecx
:004ADF71 648910                   mov dword ptr fs:[eax], edx
:004ADF74 68A6DF4A00               push 004ADFA6

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004ADFA4(U)
|
```



```
:004ADF79 8D45E4             lea eax, dword ptr [ebp-1C]

:004ADF7C E8E368F5FF        call 00404864

:004ADF81 8D45E8             lea eax, dword ptr [ebp-18]

:004ADF84 BA02000000        mov edx, 00000002

:004ADF89 E8FA68F5FF        call 00404888

:004ADF8E 8D45F0             lea eax, dword ptr [ebp-10]

:004ADF91 E8CE68F5FF        call 00404864

:004ADF96 8D45F4             lea eax, dword ptr [ebp-0C]

:004ADF99 E8C668F5FF        call 00404864

:004ADF9E C3                 ret

****************************************************************
```

新手破解,功力不足,注册机实在是难写。用 `OD` 跟踪发现，到出错对话框时真注册码地位置是 `EBP－F`，所以我们把出错对话框改成上面的样子。

出错对话框就可以变成注册码输出对话框啦！

`4dd466d0e1e0cc123afbe504da2462ca`
