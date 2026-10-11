
# 汇编代码片段

```
:004C7CAD 8B55D4                  mov edx, dword ptr [ebp-2C]

:004C7CB0 E8A7C2F3FF              call 00403F5C    ; 这一段是把计算得到的后四位字符串取反 此字符串设为 `S_2`

:004C7CB5 43                      inc ebx

:004C7CB6 4E                      dec esi

:004C7CB7 75DA                    jne 004C7C93     ; 未取完，继续

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004C7C8C(C)

|

:004C7CB9 8D45F8                  lea eax, dword ptr [ebp-08]

:004C7CBC 8B55D8                  mov edx, dword ptr [ebp-28]

:004C7CBF E8A8C0F3FF              call 00403D6C

:004C7CC4 8D45D8                  lea eax, dword ptr [ebp-28]

:004C7CC7 B9FF000000              mov ecx, 000000FF

:004C7CCC BA01000000              mov edx, 00000001

:004C7CD1 E8C6C4F3FF              call 0040419C

:004C7CD6 8D55D8                  lea edx, dword ptr [ebp-28]

:004C7CD9 A140034D00              mov eax, dword ptr [004D0340]

:004C7CDE 8B00                    mov eax, dword ptr [eax]

:004C7CE0 8B80D4020000            mov eax, dword ptr [eax+000002D4]

:004C7CE6 E8A1B4F6FF              call 0043318C

:004C7CEB 8B75EC                  mov esi, dword ptr [ebp-14]

:004C7CEE 85F6                    test esi, esi

:004C7CF0 0F8EB1000000            jle 004C7DA7

:004C7CF6 BB01000000              mov ebx, 00000001

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004C7DA1(C)

|

:004C7CFB 8B45F8                  mov eax, dword ptr [ebp-08]

:004C7CFE 0FB67C18FF              movzx edi, byte ptr [eax+ebx-01] ; 依次取 `S_2`

:004C7D03 037DF0                  add edi, dword ptr [ebp-10]; 依次把 `S_2` 与 `ebp-10` 里的内容相加 `ebp-10` 就是上面保存的 `0x63`

:004C7D06 81FFFF000000            cmp edi, 000000FF                 ; 相加的结果是否>0xff

:004C7D0C 7E06                    jle 004C7D14

:004C7D0E 81EFFF000000            sub edi, 000000FF

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004C7D0C(C)

|

:004C7D14 85FF                    test edi, edi

:004C7D16 7D06                    jge 004C7D1E        ; 比较 `edi` 是否<0，不小于则跳

:004C7D18 81C7FF000000            add edi, 000000FF  ; 否则 `edi+0xff`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004C7D16(C)

|

:004C7D1E 8D45C4                  lea eax, dword ptr [ebp-3C]

:004C7D21 8BD3                    mov edx, ebx

:004C7D23 4A                      dec edx

:004C7D24 03D2                    add edx, edx

:004C7D26 8B4DD8                  mov ecx, dword ptr [ebp-28]

:004C7D29 8A1411                  mov dl, byte ptr [ecx+edx]; 取 `x1` 的奇数位，即 1,3,5,7 位

:004C7D2C 885001                  mov byte ptr [eax+01], dl

:004C7D2F C60001                  mov byte ptr [eax], 01

:004C7D32 8D55C4                  lea edx, dword ptr [ebp-3C]

:004C7D35 8D45C0                  lea eax, dword ptr [ebp-40]

:004C7D38 E88BADF3FF              call 00402AC8

:004C7D3D 8D45BC                  lea eax, dword ptr [ebp-44]

:004C7D40 8BD3                    mov edx, ebx

:004C7D42 4A                      dec edx

:004C7D43 03D2                    add edx, edx

:004C7D45 8B4DD8                  mov ecx, dword ptr [ebp-28]

:004C7D48 8A541101                mov dl, byte ptr [ecx+edx+01]; 取 `x1` 偶数位，即 2,4,6,8 位

:004C7D4C 885001                  mov byte ptr [eax+01], dl

:004C7D4F C60001                  mov byte ptr [eax], 01

:004C7D52 8D55BC                  lea edx, dword ptr [ebp-44]

:004C7D55 8D45C0                  lea eax, dword ptr [ebp-40]

:004C7D58 B102                    mov cl, 02

:004C7D5A E839ADF3FF              call 00402A98

:004C7D5F 8D55C0                  lea edx, dword ptr [ebp-40]

:004C7D62 8D45C8                  lea eax, dword ptr [ebp-38]

:004C7D65 E88EC1F3FF              call 00403EF8

:004C7D6A 8B45C8                  mov eax, dword ptr [ebp-38]

:004C7D6D 50                      push eax

:004C7D6E 8D45B8                  lea eax, dword ptr [ebp-48]

:004C7D71 50                      push eax

:004C7D72 897DB0                  mov dword ptr [ebp-50], edi

:004C7D75 C645B400                mov [ebp-4C], 00

:004C7D79 8D55B0                  lea edx, dword ptr [ebp-50]

:004C7D7C 33C9                    xor ecx, ecx

* Possible StringData Ref from Code Obj ->`"%1.2x"`

                                |

:004C7D7E B838854C00              mov eax, 004C8538

:004C7D83 E8201AF4FF              call 004097A8

:004C7D88 8B55B8                  mov edx, dword ptr [ebp-48]; 把 `S_2` 加 `0x63` 的十六进进制结果做为注册码的第一部分

:004C7D8B 58                      pop eax                     ; 取 `x1` 每两位

:004C7D8C E8D3C2F3FF              call 00404064

:004C7D91 0F9445D3                sete byte ptr [ebp-2D]

:004C7D95 807DD300                cmp byte ptr [ebp-2D], 00

:004C7D99 0F84F8060000            je 004C8497       ; 如果注册码相等则不跳，否则 game over

:004C7D9F 43                      inc ebx

:004C7DA0 4E                      dec esi

:004C7DA1 0F8554FFFFFF            jne 004C7CFB      ; 未取完 `S_2`，继续

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004C7CF0(C)

|

:004C7DA7 8D55F8                  lea edx, dword ptr [ebp-08]

:004C7DAA A140034D00              mov eax, dword ptr [004D0340]

:004C7DAF 8B00                    mov eax, dword ptr [eax]

:004C7DB1 8B80D0020000            mov eax, dword ptr [eax+000002D0]

:004C7DB7 E8D0B3F6FF              call 0043318C

:004C7DBC 8D45F4                  lea eax, dword ptr [ebp-0C]

* Possible StringData Ref from Code Obj ->`"yangyi"`

                                |

:004C7DBF BA48854C00              mov edx, 004C8548

:004C7DC4 E8A3BFF3FF              call 00403D6C

:004C7DC9 8D55F8                  lea edx, dword ptr [ebp-08]

:004C7DCC 8B45FC                  mov eax, dword ptr [ebp-04]

:004C7DCF E8C069FFFF              call 004BE794

:004C7DD4 8B45F8                  mov eax, dword ptr [ebp-08]

:004C7DD7 E878C1F3FF              call 00403F54

:004C7DDC 8945EC                  mov dword ptr [ebp-14], eax

:004C7DDF 8B45F4                  mov eax, dword ptr [ebp-0C]

:004C7DE2 E86DC1F3FF              call 00403F54

:004C7DE7 8945E8                  mov dword ptr [ebp-18], eax

:004C7DEA C745F014000000          mov [ebp-10], 00000014

:004C7DF1 8D45D8                  lea eax, dword ptr [ebp-28]

:004C7DF4 B9FF000000              mov ecx, 000000FF

:004C7DF9 BA01000000              mov edx, 00000001

:004C7DFE E899C3F3FF              call 0040419C

:004C7E03 8B75EC                  mov esi, dword ptr [ebp-14]

:004C7E06 85F6                    test esi, esi

:004C7E08 7E2B                    jle 004C7E35

:004C7E0A BB01000000              mov ebx, 00000001

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004C7E33(C)

|

:004C7E0F 8D45D4                  lea eax, dword ptr [ebp-2C]

:004C7E12 50                      push eax

:004C7E13 8B55EC                  mov edx, dword ptr [ebp-14]

:004C7E16 2BD3                    sub edx, ebx

:004C7E18 42                      inc edx

:004C7E19 B901000000              mov ecx, 00000001

:004C7E1E 8B45F8                  mov eax, dword ptr [ebp-08]

:004C7E21 E836C3F3FF              call 0040415C

:004C7E26 8D45D8                  lea eax, dword ptr [ebp-28]

:004C7E29 8B55D4                  mov edx, dword ptr [ebp-2C]

:004C7E2C E82BC1F3FF              call 00403F5C

:004C7E31 43                      inc ebx

:004C7E32 4E                      dec esi

:004C7E33 75DA                    jne 004C7E0F    ; 这一段是把 `C_2` 取反，取完为止

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004C7E08(C)

|

:004C7E35 8D45F8                  lea eax, dword ptr [ebp-08]
```
