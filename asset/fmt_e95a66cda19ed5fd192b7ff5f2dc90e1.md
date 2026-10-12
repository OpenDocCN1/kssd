
# FAR

> 原文：[`www.pediy.com/kssd/pediy03/forum888.htm`](https://www.pediy.com/kssd/pediy03/forum888.htm)

想注册 `swish 2.0`（特好的 flash 字体特效软件），终因功力太浅，未能如愿，很痛苦！！！

发了几次帖子，向各个大虾们求助，没有人回应，更加痛苦！！！！！！！！！！！

特找来一个简单的软件（`Far.exe`，制作帮助文件），增强点信心。大虾们就此掉头，可下一个 `swish v2.0` 一试身手！初学者，想看请继续。

```
:005311F0 50                             push eax
:005311F1 8B4DF4                         mov ecx, dword ptr [ebp-0C]
:005311F4 8B55F8                         mov edx, dword ptr [ebp-08]
:005311F7 8B45FC                         mov eax, dword ptr [ebp-04]
:005311FA E82D17FAFF                     call 004D292C---------------------关键 Call!  记为*2
:005311FF 8806                           mov byte ptr [esi], al
:00531201 803E00                         cmp byte ptr [esi], 00
:00531204 744D                           je 00531253                          ------------------------跳就死！
:00531206 8B45F4                         mov eax, dword ptr [ebp-0C]
:00531209 E82610FAFF                     call 004D2234
:0053120E 8B158C105700                   mov edx, dword ptr [0057108C]
:00531214 52                             push edx
:00531215 50                             push eax
:00531216 8D45E8                         lea eax, dword ptr [ebp-18]
:00531219 50                             push eax
:0053121A 8B0D8CEA5600                   mov ecx, dword ptr [0056EA8C]
:00531220 8B09                           mov ecx, dword ptr [ecx]
:00531222 8B55F8                         mov edx, dword ptr [ebp-08]
:00531225 8B45FC                         mov eax, dword ptr [ebp-04]
:00531228 E89F15FAFF                     call 004D27CC
* Possible StringData Ref from Data Obj ->"?M"
                                        |
:0053122D A108E95600                     mov eax, dword ptr [0056E908]
:00531232 8B158C105700                   mov edx, dword ptr [0057108C]
:00531238 8B5490FC                       mov edx, dword ptr [eax+4*edx-04]
:0053123C 8D45E4                         lea eax, dword ptr [ebp-1C]
* Possible StringData Ref from Data Obj ->" Registered OK."
                                        |
:0053123F B908135300                     mov ecx, 00531308
:00531244 E81F2EEDFF                     call 00404068
:00531249 8B45E4                         mov eax, dword ptr [ebp-1C]
:0053124C E883B4F2FF                     call 0045C6D4
:00531251 EB0A                           jmp 0053125D
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00531204(C)
|
* Possible StringData Ref from Data Obj ->"Invalid entry. Please enter the "     -----------记为*1
                                             ->"correct codes."
                                        |
:00531253 B820135300                     mov eax, 00531320
:00531258 E877B4F2FF                     call 0045C6D4
```

追入关键 Call:

```
:004D292C 55                             push ebp
:004D292D 8BEC                           mov ebp, esp
:004D292F 83C4B8                         add esp, FFFFFFB8
:004D2932 53                             push ebx
:004D2933 56                             push esi
:004D2934 33DB                           xor ebx, ebx
:004D2936 895DF0                         mov dword ptr [ebp-10], ebx
:004D2939 894DF4                         mov dword ptr [ebp-0C], ecx
:004D293C 8955F8                         mov dword ptr [ebp-08], edx
:004D293F 8945FC                         mov dword ptr [ebp-04], eax
:004D2942 8B7508                         mov esi, dword ptr [ebp+08]
:004D2945 8B45FC                         mov eax, dword ptr [ebp-04]
:004D2948 E88318F3FF                     call 004041D0
:004D294D 8B45F8                         mov eax, dword ptr [ebp-08]
:004D2950 E87B18F3FF                     call 004041D0
:004D2955 8B45F4                         mov eax, dword ptr [ebp-0C]
:004D2958 E87318F3FF                     call 004041D0
:004D295D 8B450C                         mov eax, dword ptr [ebp+0C]
:004D2960 E86B18F3FF                     call 004041D0
:004D2965 33C0                           xor eax, eax
:004D2967 55                             push ebp
:004D2968 68262A4D00                     push 004D2A26
:004D296D 64FF30                         push dword ptr fs:[eax]
:004D2970 648920                         mov dword ptr fs:[eax], esp
:004D2973 8B45F4                         mov eax, dword ptr [ebp-0C]
:004D2976 E8B9F8FFFF                     call 004D2234
:004D297B 8BD8                           mov ebx, eax
:004D297D 8B45FC                         mov eax, dword ptr [ebp-04]
:004D2980 8945C8                         mov dword ptr [ebp-38], eax
:004D2983 C645CC0B                       mov [ebp-34], 0B
:004D2987 8B45F8                         mov eax, dword ptr [ebp-08]
:004D298A 8945D0                         mov dword ptr [ebp-30], eax
:004D298D C645D40B                       mov [ebp-2C], 0B
:004D2991 8B45F4                         mov eax, dword ptr [ebp-0C]
:004D2994 8945D8                         mov dword ptr [ebp-28], eax
:004D2997 C645DC0B                       mov [ebp-24], 0B
:004D299B 8B450C                         mov eax, dword ptr [ebp+0C]
:004D299E 8945E0                         mov dword ptr [ebp-20], eax
:004D29A1 C645E40B                       mov [ebp-1C], 0B
:004D29A5 8975E8                         mov dword ptr [ebp-18], esi
:004D29A8 C645EC00                       mov [ebp-14], 00
:004D29AC 8D55C8                         lea edx, dword ptr [ebp-38]
:004D29AF B904000000                     mov ecx, 00000004
* Possible StringData Ref from Data Obj ->"ValidateRegistrationInfo(%s,%s,%s,%s,%d)"
                                        |
:004D29B4 B8402A4D00                     mov eax, 004D2A40
:004D29B9 E87A86FBFF                     call 0048B038
:004D29BE 8B45F4                         mov eax, dword ptr [ebp-0C]
:004D29C1 8945B8                         mov dword ptr [ebp-48], eax
:004D29C4 C645BC0B                       mov [ebp-44], 0B
:004D29C8 895DC0                         mov dword ptr [ebp-40], ebx
:004D29CB C645C400                       mov [ebp-3C], 00
:004D29CF 8D55B8                         lea edx, dword ptr [ebp-48]
:004D29D2 B901000000                     mov ecx, 00000001
* Possible StringData Ref from Data Obj ->"  GetUserNumberFromKey(%s) = %d"
                                        |
:004D29D7 B8742A4D00                     mov eax, 004D2A74
:004D29DC E85786FBFF                     call 0048B038
:004D29E1 53                             push ebx
:004D29E2 8D45F0                         lea eax, dword ptr [ebp-10]
:004D29E5 50                             push eax
:004D29E6 8BCE                           mov ecx, esi
:004D29E8 8B55F8                         mov edx, dword ptr [ebp-08]
:004D29EB 8B45FC                         mov eax, dword ptr [ebp-04]
:004D29EE E8A9F9FFFF                     call 004D239C
:004D29F3 8B55F4                         mov edx, dword ptr [ebp-0C]
:004D29F6 8B45F0                         mov eax, dword ptr [ebp-10]   -----------------注意啊！！！！！
:004D29F9 E88E6FF3FF                     call 0040998C   -----------------db eax  看注册码！！记为*3
:004D29FE 85C0                           test eax, eax
:004D2A00 0F94C3                         sete bl
:004D2A03 33C0                           xor eax, eax
:004D2A05 5A                             pop edx
:004D2A06 59                             pop ecx
:004D2A07 59                             pop ecx
:004D2A08 648910                         mov dword ptr fs:[eax], edx
:004D2A0B 682D2A4D00                     push 004D2A2D
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004D2A2B(U)
|
:004D2A10 8D45F0                         lea eax, dword ptr [ebp-10]
:004D2A13 BA04000000                     mov edx, 00000004
:004D2A18 E8A313F3FF                     call 00403DC0
:004D2A1D 8D450C                         lea eax, dword ptr [ebp+0C]
:004D2A20 E87713F3FF                     call 00403D9C
:004D2A25 C3                             ret
```

总结：

1. `fi` 查看 `far.exe` 用 `upx` 压缩，脱壳后由 727k 变为 2.40M,(压缩的厉害）
2. `w32asm` 反汇编，找“invalid entry”来到\*1 处，向上来到\*2 处，跟入 `:005311FA E82D17FAFF call 004D292C`
   来到\*3 处，`db eax` 看注册码。
