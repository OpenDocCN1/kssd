
```assembly
:0059AD52 8BFE                     mov edi, esi
:0059AD54 83C9FF                   or ecx, FFFFFFFF
:0059AD57 F2                       repnz
:0059AD58 AE                       scasb
:0059AD59 F7D1                     not ecx
:0059AD5B 49                       dec ecx
:0059AD5C 88040A                   mov byte ptr [edx+ecx], al
:0059AD5F 8BCE                     mov ecx, esi

                                  ==== >ECX=123456       呵呵，是试炼码的中间 6 位！

:0059AD61 E84A5B0000               call 005A08B0

                                  ==== >检测上面的中间几位是否为数字？

:0059AD66 85C0                     test eax, eax
:0059AD68 750A                     jne 0059AD74

                                  ==== >不是数字则不跳则 OVER！

:0059AD6A B8FDFFFFFF               mov eax, FFFFFFFD
:0059AD6F E9F9000000               jmp 0059AE6D

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0059AD68(C)
|

:0059AD74 BA64315A00               mov edx, 005A3164

                                  ==== >EDX=0604         呵呵，程序自给的！

:0059AD79 8BCE                     mov ecx, esi

                                  ==== >ECX=123456       呵呵，是试炼码的中间 6 位！

:0059AD7B E8705B0000               call 005A08F0

                                  ==== >再次检测 123456 是否为数字？

                                  不是数字则“invalid digital number！”。并且把 123456 转化为用 16 进制值表示！

:0059AD80 8BF8                     mov edi, eax

                                  ==== >EDI=0001E240（H）=123456（D）

:0059AD82 66A1546C5A00             mov ax, word ptr [005A6C54]
:0059AD88 663D0100                 cmp ax, 0001
:0059AD8C 7546                     jne 0059ADD4
:0059AD8E 66A1FA6B5A00             mov ax, word ptr [005A6BFA]
:0059AD94 8B15006C5A00             mov edx, dword ptr [005A6C00]
:0059AD9A 33C9                     xor ecx, ecx
:0059AD9C 8ACC                     mov cl, ah
:0059AD9E 25FF000000               and eax, 000000FF
:0059ADA3 8BF1                     mov esi, ecx
:0059ADA5 8BC8                     mov ecx, eax
:0059ADA7 E854FBFFFF               call 0059A900
:0059ADAC 8B15FC6B5A00             mov edx, dword ptr [005A6BFC]
:0059ADB2 03F8                     add edi, eax
:0059ADB4 6685F6                   test si, si
:0059ADB7 7504                     jne 0059ADBD
:0059ADB9 33C9                     xor ecx, ecx
:0059ADBB EB03                     jmp 0059ADC0

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0059ADB7(C)
|

:0059ADBD 8D4E01                   lea ecx, dword ptr [esi+01]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0059ADBB(U)
|

:0059ADC0 E83BFBFFFF               call 0059A900
:0059ADC5 8BC8                     mov ecx, eax
:0059ADC7 85C9                     test ecx, ecx
:0059ADC9 7438                     je 0059AE03
:0059ADCB 8BC7                     mov eax, edi
:0059ADCD 99                       cdq
:0059ADCE F7F9                     idiv ecx
:0059ADD0 8BC2                     mov eax, edx
:0059ADD2 EB27                     jmp 0059ADFB

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0059AD8C(C)
|

:0059ADD4 663D0200                 cmp ax, 0002
:0059ADD8 7529                     jne 0059AE03
:0059ADDA 8B15FC6B5A00             mov edx, dword ptr [005A6BFC]

                                  ==== >EDX=[005A6BFC]=564g5fiofj9&⁶kfd

:0059ADE0 A1006C5A00               mov eax, dword ptr [005A6C00]

                                  ==== >EAX=[005A6C00]=dkjfkdu5f2g445]

:0059ADE5 8B0D386D5A00             mov ecx, dword ptr [005A6D38]

                                  ==== >ECX=[005A6D38]=17359（H）=95605（D）呵呵，我的 ID 号

:0059ADEB 52                       push edx
:0059ADEC 668B15FA6B5A00           mov dx, word ptr [005A6BFA]
:0059ADF3 50                       push eax
:0059ADF4 E897FBFFFF               call 0059A990

                                  ==== >算法 CALL！得出下面的 EAX 值。进入！

:0059ADF9 2BC7                     sub eax, edi

                                  ==== >EAX=5DECC - 1E240=3FC8C

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0059ADD2(U)
|

:0059ADFB 85C0                     test eax, eax

                                  ==== >相减结果是否为 0？即：上面 2 部分是否相等？

                                  三、       ==== >所以：注册码的中间部分=5DECC（H）=384716（D）

:0059ADFD 0F848E000000             je 0059AE91

                                  ==== >不为 0 则不跳则 OVER！

:0059B013 FF1500725A00             call dword ptr [005A7200]

                                  ==== >BAD BOY！

—————————————————————————————————

进入算法 CALL：`59ADF4 call 0059A990`

* Referenced by a CALL at Addresses:
|:0059ADF4 , :0059B1DA
|

```assembly
:0059A990 53                       push ebx
:0059A991 56                       push esi
:0059A992 668BDA                   mov bx, dx
:0059A995 8BF1                     mov esi, ecx
:0059A997 8B54240C                 mov edx, dword ptr [esp+0C]

                                  ==== >EDX=[esp+0C]=dkjfkdu5f2g445]

:0059A99B 8ACB                     mov cl, bl
:0059A99D 57                       push edi
:0059A99E 81E1FF000000             and ecx, 000000FF
:0059A9A4 E857FFFFFF               call 0059A900

                                  ==== >对程序给的 dkjfkdu5f2g445]进行运算得出下面的 EAX 值！进入！

:0059A9A9 8B542414                 mov edx, dword ptr [esp+14]

                                  ==== >EDX=dkjfkdu5f2g445]

:0059A9AD 8BF8                     mov edi, eax

                                  ==== >EDI=EAX=1B86B 上面 CALL 对 dkjfkdu5f2g445]进行运算得出的值!

:0059A9AF 33C0                     xor eax, eax
:0059A9B1 8AC7                     mov al, bh
:0059A9B3 6685C0                   test ax, ax
:0059A9B6 7512                     jne 0059A9CA

                                  ==== >跳下去!

:0059A9B8 33C9                     xor ecx, ecx
:0059A9BA E841FFFFFF               call 0059A900
:0059A9BF 03FE                     add edi, esi
:0059A9C1 0FAFC7                   imul eax, edi
:0059A9C4 5F                       pop edi
:0059A9C5 5E                       pop esi
:0059A9C6 5B                       pop ebx
:0059A9C7 C20800                   ret 0008

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0059A9B6(C)
|

:0059A9CA 8D4801                   lea ecx, dword ptr [eax+01]

                                  ==== >EAX=63

:0059A9CD E82EFFFFFF               call 0059A900

                                  ==== >对程序给的 564g5fiofj9&⁶kfd 进行运算得出下面的 EAX 值!

此处与 `59A9A4 call 0059A900` 的运算流程相同，只是参数不同，因此就不再记录了。循环得出 `EAX=2B308`

:0059A9D2 03C7                     add eax, edi

                                  ==== >EAX=2B308 + 1B86B=46B73

:0059A9D4 03C6                     add eax, esi

                                  ==== >EAX=46B73 + 17359=5DECC

                                  呵呵，把上面运算的结果 5DECC（H）转化成 10 进制值 384716（D），就是注册码的中间部分!

:0059A9D6 5F                       pop edi
:0059A9D7 5E                       pop esi
:0059A9D8 5B                       pop ebx
:0059A9D9 C20800                   ret 0008
```

—————————————————————————————————

进入关键 CALL：`59A9A4 call 0059A900`

* Referenced by a CALL at Addresses:
|:0059A9A4 , :0059A9BA , :0059A9CD , :0059ADA7 , :0059ADC0
|

```assembly
:0059A900 53                       push ebx
:0059A901 8BDA                     mov ebx, edx

                                  ==== >EBX=EDX=dkjfkdu5f2g445]

:0059A903 56                       push esi
:0059A904 8BF1                     mov esi, ecx

                                  ==== >ESI=ECX=31

:0059A906 85DB                     test ebx, ebx
:0059A908 7472                     je 0059A97C
:0059A90A 803B00                   cmp byte ptr [ebx], 00
:0059A90D 746D                     je 0059A97C
:0059A90F 57                       push edi
:0059A910 8BFB                     mov edi, ebx
:0059A912 83C9FF                   or ecx, FFFFFFFF
```
