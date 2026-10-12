
```
:004C7FCA C60001                           mov byte ptr [eax], 01
:004C7FCD 8D55C4                           lea edx, dword ptr [ebp-3C]
:004C7FD0 8D45C0                           lea eax, dword ptr [ebp-40]
:004C7FD3 E8F0AAF3FF                        call 00402AC8
:004C7FD8 8D45BC                           lea eax, dword ptr [ebp-44]
:004C7FDB 8BD3                             mov edx, ebx
:004C7FDD 4A                               dec edx
:004C7FDE 03D2                             add edx, edx
:004C7FE0 8B4DD8                           mov ecx, dword ptr [ebp-28]
:004C7FE3 8A541101                          mov dl, byte ptr [ecx+edx+01]; 取第二部分的假码的偶数位
:004C7FE7 885001                           mov byte ptr [eax+01], dl
:004C7FEA C60001                           mov byte ptr [eax], 01
:004C7FED 8D55BC                           lea edx, dword ptr [ebp-44]
:004C7FF0 8D45C0                           lea eax, dword ptr [ebp-40]
:004C7FF3 B102                             mov cl, 02
:004C7FF5 E89EAAF3FF                        call 00402A98
:004C7FFA 8D55C0                           lea edx, dword ptr [ebp-40]
:004C7FFD 8D45A8                           lea eax, dword ptr [ebp-58]
:004C8000 E8F3BEF3FF                        call 00403EF8
:004C8005 8B45A8                           mov eax, dword ptr [ebp-58]
:004C8008 50                               push eax
:004C8009 8D45A4                           lea eax, dword ptr [ebp-5C]
:004C800C 50                               push eax
:004C800D 897DB0                           mov dword ptr [ebp-50], edi
:004C8010 C645B400                          mov [ebp-4C], 00
:004C8014 8D55B0                           lea edx, dword ptr [ebp-50]
:004C8017 33C9                             xor ecx, ecx

* Possible StringData Ref from Code Obj ->"%1.2x"
|

:004C8019 B838854C00                        mov eax, 004C8538
:004C801E E88517F4FF                        call 004097A8
:004C8023 8B55A4                           mov edx, dword ptr [ebp-5C]; 把上面与 edi 相加的结果传入 edx
:004C8026 58                               pop eax
:004C8027 E838C0F3FF                        call 00404064
:004C802C 0F9445D3                          sete byte ptr [ebp-2D]
:004C8030 807DD300                          cmp byte ptr [ebp-2D], 00
:004C8034 0F845D040000                      je 004C8497
:004C803A 43                               inc ebx
:004C803B 4E                               dec esi
:004C803C 0F8554FFFFFF                      jne 004C7F96
; 未取到四位，继续

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004C7F8B(C)
|

:004C8042 8D55F8                           lea edx, dword ptr [ebp-08]
:004C8045 A140034D00                        mov eax, dword ptr [004D0340]
:004C804A 8B00                             mov eax, dword ptr [eax]
:004C804C 8B80D0020000                      mov eax, dword ptr [eax+000002D0]
:004C8052 E835B1F6FF                        call 0043318C
:004C8057 8D45F4                           lea eax, dword ptr [ebp-0C]

* Possible StringData Ref from Code Obj ->"ILoveYang"
|

:004C805A BA58854C00                        mov edx, 004C8558
:004C805F E808BDF3FF                        call 00403D6C
:004C8064 8D55F8                           lea edx, dword ptr [ebp-08]
:004C8067 8B45FC                           mov eax, dword ptr [ebp-04]
:004C806A E82567FFFF                        call 004BE794
:004C806F 8B45F8                           mov eax, dword ptr [ebp-08]
:004C8072 E8DDBEF3FF                        call 00403F54
:004C8077 8945EC                           mov dword ptr [ebp-14], eax
:004C807A 8B45F4                           mov eax, dword ptr [ebp-0C]
:004C807D E8D2BEF3FF                        call 00403F54
:004C8082 8945E8                           mov dword ptr [ebp-18], eax
:004C8085 C745F08C000000                    mov [ebp-10], 0000008C
:004C808C 8D45D8                           lea eax, dword ptr [ebp-28]
:004C808F 8B4DD8                           mov ecx, dword ptr [ebp-28]

* Possible StringData Ref from Code Obj ->"Book"
|

:004C8092 BA6C854C00                        mov edx, 004C856C
:004C8097 E804BFF3FF                        call 00403FA0
:004C809C 8D45D8                           lea eax, dword ptr [ebp-28]
:004C809F B9FF000000                        mov ecx, 000000FF
:004C80A4 BA01000000                        mov edx, 00000001
:004C80A9 E8EEC0F3FF                        call 0040419C
:004C80AE BB01000000                        mov ebx, 00000001

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004C80D9(C)
|

:004C80B3 8D45D4                           lea eax, dword ptr [ebp-2C]
:004C80B6 50                               push eax
:004C80B7 8B55EC                           mov edx, dword ptr [ebp-14]
:004C80BA 2BD3                             sub edx, ebx
:004C80BC 42                               inc edx
:004C80BD B901000000                        mov ecx, 00000001
:004C80C2 8B45F8                           mov eax, dword ptr [ebp-08]
:004C80C5 E892C0F3FF                        call 0040415C
:004C80CA 8D45D8                           lea eax, dword ptr [ebp-28]
:004C80CD 8B55D4                           mov edx, dword ptr [ebp-2C]
:004C80D0 E887BEF3FF                        call 00403F5C
:004C80D5 43                               inc ebx
:004C80D6 83FB05                           cmp ebx, 00000005
:004C80D9 75D8                             jne 004C80B3
; 这一小结是取 C_2 取反后的前四位

:004C80DB 8D45F8                           lea eax, dword ptr [ebp-08]
:004C80DE 8B55D8                           mov edx, dword ptr [ebp-28]
:004C80E1 E886BCF3FF                        call 00403D6C
:004C80E6 8B45F8                           mov eax, dword ptr [ebp-08]
:004C80E9 E866BEF3FF                        call 00403F54
:004C80EE 8945EC                           mov dword ptr [ebp-14], eax
:004C80F1 8D45D8                           lea eax, dword ptr [ebp-28]
:004C80F4 B9FF000000                        mov ecx, 000000FF
:004C80F9 BA01000000                        mov edx, 00000001
:004C80FE E899C0F3FF                        call 0040419C
:004C8103 8B75EC                           mov esi, dword ptr [ebp-14]
:004C8106 85F6                             test esi, esi
:004C8108 7E2B                             jle 004C8135
:004C810A BB01000000                        mov ebx, 00000001

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004C8133(C)
|

:004C810F 8D45D4                           lea eax, dword ptr [ebp-2C]
:004C8112 50                               push eax
:004C8113 8B55EC                           mov edx, dword ptr [ebp-14]
:004C8116 2BD3                             sub edx, ebx
:004C8118 42                               inc edx
:004C8119 B901000000                        mov ecx, 00000001
:004C811E 8B45F8                           mov eax, dword ptr [ebp-08]
:004C8121 E836C0F3FF                        call 0040415C
:004C8126 8D45D8                           lea eax, dword ptr [ebp-28]
:004C8129 8B55D4                           mov edx, dword ptr [ebp-2C]
:004C812C E82BBEF3FF                        call 00403F5C
:004C8131 43                               inc ebx
:004C8132 4E                               dec esi
:004C8133 75DA                             jne 004C810F
; 把取到的四位字符串又倒序，设为 2_C_1

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004C8108(C)
|

:004C8135 8D45F8                           lea eax, dword ptr [ebp-08]
:004C8138 8B55D8                           mov edx, dword ptr [ebp-28]
:004C813B E82CBCF3FF                        call 00403D6C
:004C8140 8D45E0                           lea eax, dword ptr [ebp-20]
:004C8143 B9FF000000                        mov ecx, 000000FF
:004C8148 BA01000000                        mov edx, 00000001
:004C814D E84AC0F3FF                        call 0040419C
:004C8152 8D45D8                           lea eax, dword ptr [ebp-28]
:004C8155 B9FF000000                        mov ecx, 000000FF
:004C815A BA01000000                        mov edx, 00000001
:004C815F E838C0F3FF                        call 0040419C
:004C8164 C745E401000000                   mov [ebp-1C], 00000001
:004C816B C645DE01                          mov [ebp-22], 01

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004C820C(C)
|

:004C816F 8D45E0                           lea eax, dword ptr [ebp-20]
```
