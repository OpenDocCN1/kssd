
# 破解`sonique`的插件Dee2的核心`dee2.dll`

`sonique`的插件Dee2

`dee2.dll`   `UPX0.99-1.23`

`OD`载入`dee2.dll`

```
1017C9C0 >  807C24 08 01    cmp     byte ptr ss:[esp+8], 1  <=========下断点
1017C9C5    0F85 7E010000   jnz     dee2.1017CB49 <=====第一次没跳,解码,第二次跳往dee2.1000BDA7
1017C9CB    60              pushad
                  --------------------------------------------------------
1000BDA7    55              push    ebp                              ; 进来了~~~~~~~Dee2.dll
1000BDA8    8BEC               mov     ebp, esp
1000BDAA    53                 push    ebx                              ; Dee2.<ModuleEntryPoint>
1000BDAB    8B5D 08            mov     ebx, dword ptr ss:[ebp+8]        ; Dee2.<ModuleEntryPoint>
1000BDAE    56                 push    esi
1000BDAF    8B75 0C            mov     esi, dword ptr ss:[ebp+C]        ; Dee2.10000000
1000BDB2    57                 push    edi
1000BDB3    8B7D 10            mov     edi, dword ptr ss:[ebp+10]
1000BDB6    85F6               test    esi, esi
1000BDB8    75 09              jnz     short Dee2.1000BDC3
1000BDBA    833D B8F80510 00   cmp     dword ptr ds:[1005F8B8], 0
1000BDC1    EB 26              jmp     short Dee2.1000BDE9
1000BDC3    83FE 01            cmp     esi, 1
1000BDC6    74 05              je      short Dee2.1000BDCD
1000BDC8    83FE 02            cmp     esi, 2
1000BDCB    75 22              jnz     short Dee2.1000BDEF
1000BDCD    A1 C80F0610        mov     eax, dword ptr ds:[10060FC8]
1000BDD2    85C0               test    eax, eax
1000BDD4    74 09              je      short Dee2.1000BDDF
```

右键|搜索|全部交互调用

`USER32.MessageBoxA`        `ADVAPI32.RegQueryValueExA`         `USER32.ShowWindow`

在这三类上全部下断点(关闭)

```
------------------------------------------------------------------------
OD载入sonique.exe

运行...呵呵,听到歌声~~~~~~~:)
开断点.

输入试炼码166925-222222-222222-222222,点确定~!

中断在:

015C742D    FF15 44515D01   call    near dword ptr ds:[15D5144]      ; USER32.MessageBoxA (*)

看看中断处前面的代码~~~:)

015C7313    390D 6CF86101   cmp     dword ptr ds:[161F86C], ecx
015C7319    0F85 BD020000   jnz     dee2.015C75DC
015C731F    8B75 08         mov     esi, dword ptr ss:[ebp+8]
015C7322    8B1D 50515D01   mov     ebx, dword ptr ds:[15D5150]      ; USER32.LoadBitmapA
015C7328    81FF 3A010000   cmp     edi, 13A
015C732E    0F8C 55010000   jl      dee2.015C7489
015C7334    81FF 7C010000   cmp     edi, 17C
015C733A    0F8F 49010000   jg      dee2.015C7489
015C7340    83F8 5C         cmp     eax, 5C
015C7343    0F8C 40010000   jl      dee2.015C7489
015C7349    83F8 76         cmp     eax, 76
015C734C    0F8F 37010000   jg      dee2.015C7489
015C7352    6A 06           push    6
015C7354    8D4C24 24       lea     ecx, dword ptr ss:[esp+24]
015C7358    68 74F86101     push    dee2.0161F874                    ; ASCII "166925-222222-222222-222222"
015C735D    51              push    ecx
015C735E    C705 6CF86101 0>mov     dword ptr ds:[161F86C], 0
015C7368    C64424 32 00    mov     byte ptr ss:[esp+32], 0
015C736D    E8 5E480000     call    dee2.015CBBD0
015C7372    8D5424 2C       lea     edx, dword ptr ss:[esp+2C]
015C7376    52              push    edx
015C7377    E8 16480000     call    dee2.015CBB92 <================== ---S---
015C737C    A3 58F86101     mov     dword ptr ds:[161F858], eax <==== 记录K
015C7381    6A 06           push    6
015C7383    8D4424 34       lea     eax, dword ptr ss:[esp+34]
015C7387    68 7BF86101     push    dee2.0161F87B                    ; ASCII "222222-222222-222222"
015C738C    50              push    eax
015C738D    E8 3E480000     call    dee2.015CBBD0
015C7392    8D4C24 3C       lea     ecx, dword ptr ss:[esp+3C]
015C7396    51              push    ecx
015C7397    E8 F6470000     call    dee2.015CBB92
015C739C    6A 06           push    6
015C739E    8D5424 44       lea     edx, dword ptr ss:[esp+44]
015C73A2    68 82F86101     push    dee2.0161F882                    ; ASCII "222222-222222"
015C73A7    52              push    edx
015C73A8    A3 5CF86101     mov     dword ptr ds:[161F85C], eax <==== 记录K
015C73AD    E8 1E480000     call    dee2.015CBBD0
015C73B2    8D4424 4C       lea     eax, dword ptr ss:[esp+4C]
015C73B6    50              push    eax
015C73B7    E8 D6470000     call    dee2.015CBB92
015C73BC    6A 06           push    6
015C73BE    8D4C24 54       lea     ecx, dword ptr ss:[esp+54]
015C73C2    68 89F86101     push    dee2.0161F889                    ; ASCII "222222"
015C73C7    51              push    ecx
015C73C8    A3 60F86101     mov     dword ptr ds:[161F860], eax <==== 记录K
015C73CD    E8 FE470000     call    dee2.015CBBD0
015C73D2    8D5424 5C       lea     edx, dword ptr ss:[esp+5C]
015C73D6    52              push    edx
015C73D7    E8 B6470000     call    dee2.015CBB92
015C73DC    8B0D 5CF86101   mov     ecx, dword ptr ds:[161F85C]
015C73E2    8B15 58F86101   mov     edx, dword ptr ds:[161F858]
015C73E8    83C4 40         add     esp, 40
015C73EB    A3 64F86101     mov     dword ptr ds:[161F864], eax
015C73F0    50              push    eax
015C73F1    A1 60F86101     mov     eax, dword ptr ds:[161F860]
015C73F6    50              push    eax
015C73F7    51              push    ecx
015C73F8    52              push    edx
015C73F9    E8 32C0FFFF     call    dee2.015C3430 <========== ---H---
015C73FE    83C4 10         add     esp, 10
015C7401    83F8 01         cmp     eax, 1 <=================eax=1就K.O.
015C7404    75 1A           jnz     short dee2.015C7420 <====^\_^可爱的三步曲
015C7406    6A 00           push    0
015C7408    68 B8AC5D01     push    dee2.015DACB8                    ; ASCII ":)))))))))))))"
015C740D    68 9CAC5D01     push    dee2.015DAC9C                    ; ASCII "Thanks for registering Dee2"
015C7412    56              push    esi
015C7413    A3 70F86101     mov     dword ptr ds:[161F870], eax
015C7418    FF15 44515D01   call    near dword ptr ds:[15D5144]      ; USER32.MessageBoxA
015C741E    EB 25           jmp     short dee2.015C7445
015C7420    6A 00           push    0
015C7422    68 8CAC5D01     push    dee2.015DAC8C                    ; ASCII ":((((((((((((("
015C7427    68 80AC5D01     push    dee2.015DAC80                    ; ASCII "Invalid key"
015C742C    56              push    esi
015C742D    FF15 44515D01   call    near dword ptr ds:[15D5144]      ; USER32.MessageBoxA (*)
```

```
------------------------------------------------------------------------
---H---
---------------------------------------------------------------------

015C3430    8B4424 08       mov     eax, dword ptr ss:[esp+8] <====第二段试炼码的计算值  y
015C3434    8B4C24 04       mov     ecx, dword ptr ss:[esp+4] <====第一段试炼码的计算值  x
015C3438    35 4B1B3558     xor     eax, 58351B4B
015C343D    81F1 76823327   xor     ecx, 27338276
015C3443    0FAFC1          imul    eax, ecx
015C3446    8B5424 0C       mov     edx, dword ptr ss:[esp+C] <====第三段试炼码的计算值  t
015C344A    B9 635E3100     mov     ecx, 315E63
015C344F    33C2            xor     eax, edx
015C3451    83EC 40         sub     esp, 40 <==================载入第四段试炼码的计算值  z
015C3454    35 8A11DE1D     xor     eax, 1DDE118A <===========(y xor 58351B4B)*(x xor 27338276)xor t  xor 1DDE118A ---->eax
015C3459    99              cdq
015C345A    F7F9            idiv    ecx
015C345C    395424 50       cmp     dword ptr ss:[esp+50], edx <=======eax MOD 315E63 CMP z
015C3460    0F85 F6000000   jnz     dee2.015C355C <===NOP就成功了,不幸的是有蛋壳,晕~~~,还要找算法~~~~
015C3466    8B0D 5CF86101   mov     ecx, dword ptr ds:[161F85C]
015C346C    8B15 60F86101   mov     edx, dword ptr ds:[161F860]
015C3472    A1 58F86101     mov     eax, dword ptr ds:[161F858]
015C3477    81F1 93CA8E00   xor     ecx, 8ECA93
015C347D    81F2 95CE5135   xor     edx, 3551CE95
015C3483    35 B75BE9CC     xor     eax, CCE95BB7
015C3488    890D 5CF86101   mov     dword ptr ds:[161F85C], ecx
```
