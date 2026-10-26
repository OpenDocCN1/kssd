
```asm
:0041BE93 F3                      repz

:0041BE94 A5                      movsd

:0041BE95 8BCA                    mov ecx, edx

:0041BE97 83E103                  and ecx, 00000003

:0041BE9A F3                      repz

:0041BE9B A4                      movsb

:0041BE9C 8A08                    mov cl, byte ptr [eax]

:0041BE9E 84C9                    test cl, cl

:0041BEA0 740D                    je 0041BEAF

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0041BEAD(C)

|

:0041BEA2 80E97F                  sub cl, 7F  <--加密字符串的解密算法

:0041BEA5 8808                    mov byte ptr [eax], cl

:0041BEA7 8A4801                  mov cl, byte ptr [eax+01]

:0041BEAA 40                      inc eax

:0041BEAB 84C9                    test cl, cl

:0041BEAD 75F3                    jne 0041BEA2

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0041BEA0(C)

|

:0041BEAF 8D83D4060000            lea eax, dword ptr [ebx+000006D4]

:0041BEB5 50                      push eax

:0041BEB6 8D442414                lea eax, dword ptr [esp+14]

:0041BEBA 50                      push eax

:0041BEBB E810590000              call 004217D0  <--检查注册文件中是否有黑名单中的"XTERM"

:0041BEC0 83C408                  add esp, 00000008

:0041BEC3 85C0                    test eax, eax

:0041BEC5 7414                    je 0041BEDB  <--应该跳转

:0041BEC7 C783B405000020A94100    mov dword ptr [ebx+000005B4], 0041A920  <--送注册失败处理子程序的地址

\* Possible Reference to String Resource ID=00001: "b

\*"

                                  |

:0041BED1 C783B805000001000000    mov dword ptr [ebx+000005B8], 00000001  <--把注册失败值送全局变量reg

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0041BEC5(C)

|

:0041BEDB 5F                      pop edi

:0041BEDC 5E                      pop esi

:0041BEDD 5B                      pop ebx

:0041BEDE 81C404040000            add esp, 00000404

:0041BEE4 C3                      ret

//\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

    第七次调用call eax会来到如下地方：

:0041B560 81EC04040000            sub esp, 00000404

:0041B566 53                      push ebx

:0041B567 8BD9                    mov ebx, ecx

:0041B569 56                      push esi

:0041B56A C783B405000040B74100    mov dword ptr [ebx+000005B4], 0041B740  <--送下一个检查子程序的地址

:0041B574 8B8344050000            mov eax, dword ptr [ebx+00000544]

:0041B57A 8DB348050000            lea esi, dword ptr [ebx+00000548]

:0041B580 6A00                    push 00000000

:0041B582 8A08                    mov cl, byte ptr [eax]

:0041B584 40                      inc eax

:0041B585 51                      push ecx

:0041B586 8BCE                    mov ecx, esi

:0041B588 898344050000            mov dword ptr [ebx+00000544], eax

:0041B58E E8FD2F0000              call 0041E590  <--这个call比对第9个字符

:0041B593 85C0                    test eax, eax

:0041B595 751D                    jne 0041B5B4  <--不应该跳转

:0041B597 8B8344050000            mov eax, dword ptr [ebx+00000544]

:0041B59D 6A00                    push 00000000

:0041B59F 8A08                    mov cl, byte ptr [eax]

:0041B5A1 40                      inc eax

:0041B5A2 51                      push ecx

:0041B5A3 8BCE                    mov ecx, esi

:0041B5A5 898344050000            mov dword ptr [ebx+00000544], eax

:0041B5AB E8E02F0000              call 0041E590  <--这个call比对第10个字符

:0041B5B0 85C0                    test eax, eax

:0041B5B2 7414                    je 0041B5C8  <--应该跳转

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0041B595(C)

|

:0041B5B4 C783B405000020A94100    mov dword ptr [ebx+000005B4], 0041A920  <--送注册失败处理子程序的地址

\* Possible Reference to String Resource ID=00001: "b

\*"

                                  |

:0041B5BE C783B805000001000000    mov dword ptr [ebx+000005B8], 00000001  <--把注册失败值送全局变量reg

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0041B5B2(C)

|

:0041B5C8 55                      push ebp

:0041B5C9 8DAB40010000            lea ebp, dword ptr [ebx+00000140]

:0041B5CF 57                      push edi

:0041B5D0 8BFD                    mov edi, ebp

:0041B5D2 83C9FF                  or ecx, FFFFFFFF

:0041B5D5 33C0                    xor eax, eax

:0041B5D7 F2                      repnz

:0041B5D8 AE                      scasb

:0041B5D9 F7D1                    not ecx

:0041B5DB 2BF9                    sub edi, ecx

:0041B5DD 8D542414                lea edx, dword ptr [esp+14]

:0041B5E1 8BC1                    mov eax, ecx

:0041B5E3 8BF7                    mov esi, edi

:0041B5E5 8BFA                    mov edi, edx

:0041B5E7 C1E902                  shr ecx, 02

:0041B5EA F3                      repz

:0041B5EB A5                      movsd

:0041B5EC 8BC8                    mov ecx, eax

:0041B5EE 83E103                  and ecx, 00000003

:0041B5F1 F3                      repz

:0041B5F2 A4                      movsb

:0041B5F3 8D4C2414                lea ecx, dword ptr [esp+14]

:0041B5F7 51                      push ecx

:0041B5F8 E8F44A0100              call 004300F1

\* Possible StringData Ref from Data Obj ->"晌虑耐悄乙"  <--该字符串解密后为"JOCHEN HESSE"

                                  |

:0041B5FD BF7C464500              mov edi, 0045467C

:0041B602 83C9FF                  or ecx, FFFFFFFF

:0041B605 33C0                    xor eax, eax

:0041B607 83C404                  add esp, 00000004

:0041B60A F2                      repnz

:0041B60B AE                      scasb

:0041B60C F7D1                    not ecx

:0041B60E 2BF9                    sub edi, ecx

:0041B610 8D93D4060000            lea edx, dword ptr [ebx+000006D4]

:0041B616 8BC1                    mov eax, ecx

:0041B618 8BF7                    mov esi, edi

:0041B61A 8BFA                    mov edi, edx

:0041B61C 8BD0                    mov edx, eax

:0041B61E 8BC7                    mov eax, edi

:0041B620 C1E902                  shr ecx, 02

:0041B623 F3                      repz

:0041B624 A5                      movsd

:0041B625 8BCA                    mov ecx, edx

:0041B627 83E103                  and ecx, 00000003

:0041B62A F3                      repz

:0041B62B A4                      movsb

:0041B62C 8A08                    mov cl, byte ptr [eax]

:0041B62E 84C9                    test cl, cl

:0041B630 740D                    je 0041B63F

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0041B63D(C)

|

:0041B632 80E97F                  sub cl, 7F  <--加密字符串的解密算法

:0041B635 8808                    mov byte ptr [eax], cl

:0041B637 8A4801                  mov cl, byte ptr [eax+01]

:0041B63A 40                      inc eax

:0041B63B 84C9                    test cl, cl

:0041B63D 75F3                    jne 0041B632

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0041B630(C)

|

:0041B63F 8D83D4060000            lea eax, dword ptr [ebx+000006D4]

:0041B645 50                      push eax

:0041B646 8D442418                lea eax, dword ptr [esp+18]

:0041B64A 50                      push eax

:0041B64B E880610000              call 004217D0  <--检查注册文件中是否有黑名单中的"JOCHEN HESSE"

:0041B650 83C408                  add esp, 00000008

:0041B653 85C0                    test eax, eax

:0041B655 7414                    je 0041B66B  <--应该跳转

:0041B657 C783B405000020A94100    mov dword ptr [ebx+000005B4], 0041A920  <--送注册失败处理子程序的地址

\* Possible Reference to String Resource ID=00001: "b

\*"

                                  |

:0041B661 C783B805000001000000    mov dword ptr [ebx+000005B8], 00000001  <--把注册失败值送全局变量reg

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0041B655(C)

|

:0041B66B 8BFD                    mov edi, ebp

:0041B66D 83C9FF                  or ecx, FFFFFFFF

:0041B670 33C0                    xor eax, eax
```
