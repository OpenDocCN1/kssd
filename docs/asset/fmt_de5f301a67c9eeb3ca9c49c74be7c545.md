
```asm
.text:F4CD402A             lea     eax, [ebp+var_4+3]  

.text:F4CD402D  

.text:F4CD402D loc_F4CD402D:                    ; CODE XREF: sub_F4CD3FD2+66j  

.text:F4CD402D             xor     byte ptr [ebp+edx+var_8], cl ; LicName XOR with 0xD6C2BC0h ,0x0D7C0D6DCh  

.text:F4CD4031             add     [eax], cl  

.text:F4CD4033             inc     edx  

.text:F4CD4034             dec     eax  

.text:F4CD4035             cmp     edx, 8  

.text:F4CD4038             jl      short loc_F4CD402D  

.text:F4CD403A             inc     esi  

.text:F4CD403B             cmp     esi, [ebp+var_10]  

.text:F4CD403E             jl      short loc_F4CD4022  

.text:F4CD4040  

.text:F4CD4040 loc_F4CD4040:                    ; CODE XREF: sub_F4CD3FD2+4Ej  

.text:F4CD4040             push    ebx  

.text:F4CD4041             push    1  

.text:F4CD4043             push    8  

.text:F4CD4045             push    [ebp+arg_4]  

.text:F4CD4048             lea     eax, [ebp+var_8]  

.text:F4CD404B             push    eax  

.text:F4CD404C             call    sub_F4CD3DEE    ; Process LicName //最终将在这儿产生注册码:) -----------------  

.text:F4CD4051             lea     ecx, [ebp+var_18]                  |   

.text:F4CD4054             call    sub_F4C44C60                       |           

.text:F4CD4059             mov     al, 1                              |   

.text:F4CD405B             pop     esi                              |   

.text:F4CD405C             jmp     short loc_F4CD4060                       |   

.text:F4CD405E ; ---------------------------------------------------------------------------                       |   

.text:F4CD405E                             |   

.text:F4CD405E loc_F4CD405E:                    ; CODE XREF: sub_F4CD3FD2+Ej                              |  

.text:F4CD405E                                 ; sub_F4CD3FD2+12j                |  

.text:F4CD405E             xor     al, al                         |       

.text:F4CD4060                            |   

.text:F4CD4060 loc_F4CD4060:                    ; CODE XREF: sub_F4CD3FD2+8Aj                              |  

.text:F4CD4060             pop     ebx                                                     |  

.text:F4CD4061             leave                           |  

.text:F4CD4062             retn    8                        |   

.text:F4CD4062 sub_F4CD3FD2    endp                                   |  

                            |   

                        |  

text:F4283DEE                       |  

.text:F4283DEE             push    ebp                  <------------------------------------------------------  

.text:F4283DEF             mov     ebp, esp  

.text:F4283DF1             sub     esp, 18h  

.text:F4283DF4             push    ebx  

.text:F4283DF5             mov     ebx, [ebp+arg_C]  

.text:F4283DF8             push    esi  

.text:F4283DF9             push    edi  

.text:F4283DFA             xor     edi, edi  

.text:F4283DFC             cmp     [ebp+arg_10], edi  

.text:F4283DFF             lea     esi, [ebx+ebx]  

.text:F4283E02             mov     [ebp+var_4], esi  

.text:F4283E05             mov     [ebp+var_C], edi  

.text:F4283E08             jz      short loc_F4283E18  

.text:F4283E0A             push    [ebp+arg_10]  

.text:F4283E0D             call    sub_F41F2F04  

.text:F4283E12             add     [ebp+var_4], eax  

.text:F4283E15             mov     [ebp+var_C], eax  

.text:F4283E18  

.text:F4283E18 loc_F4283E18:                    ; CODE XREF: sub_F4283DEE+1Aj  

.text:F4283E18             mov     eax, [ebp+arg_8]  

.text:F4283E1B             cdq  

.text:F4283E1C             idiv    ebx  

.text:F4283E1E             test    eax, eax  

.text:F4283E20             mov     ebx, edx  

.text:F4283E22             mov     [ebp+var_18], ebx  

.text:F4283E25             jle     loc_F4283EB4  

.text:F4283E2B             mov     ecx, [ebp+arg_0]  

.text:F4283E2E             mov     [ebp+var_8], ecx  

.text:F4283E31             mov     ecx, [ebp+arg_4]  

.text:F4283E34             add     esi, ecx  

.text:F4283E36             mov     edi, eax  

.text:F4283E38             mov     [ebp+var_10], esi  

.text:F4283E3B             mov     [ebp+var_14], edi  

.text:F4283E3E  

.text:F4283E3E loc_F4283E3E:                    ; CODE XREF: sub_F4283DEE+C4j  

.text:F4283E3E             cmp     [ebp+arg_10], 0  

.text:F4283E42             jz      short loc_F4283E4D  

.text:F4283E44             push    [ebp+arg_10]  

.text:F4283E47             push    esi  

.text:F4283E48             call    sub_F41F2EE8  

.text:F4283E4D  

.text:F4283E4D loc_F4283E4D:                    ; CODE XREF: sub_F4283DEE+54j  

.text:F4283E4D             and     [ebp+arg_8], 0  

.text:F4283E51             cmp     [ebp+arg_C], 0  

.text:F4283E55             jle     short loc_F4283EA0  

.text:F4283E57  

.text:F4283E57 loc_F4283E57:                    ; CODE XREF: sub_F4283DEE+ADj  

.text:F4283E57             mov     eax, [ebp+var_8]  

.text:F4283E5A             mov     ecx, [ebp+arg_8]  

.text:F4283E5D             mov     cl, [eax+ecx]  

.text:F4283E60             movzx   eax, cl  

.text:F4283E63             cdq  

.text:F4283E64             push    10h  

.text:F4283E66             pop     ebx  

.text:F4283E67             idiv    ebx  

.text:F4283E69             dec     esi  

.text:F4283E6A             cmp     dl, 9  

.text:F4283E6D             movzx   eax, dl  

.text:F4283E70             ja      short loc_F4283E77  

.text:F4283E72             add     eax, 30h  

.text:F4283E75             jmp     short loc_F4283E7A  

.text:F4283E77 ; ---------------------------------------------------------------------------  

.text:F4283E77  

.text:F4283E77 loc_F4283E77:                    ; CODE XREF: sub_F4283DEE+82j  

.text:F4283E77             add     eax, 37h  

.text:F4283E7A  

.text:F4283E7A loc_F4283E7A:                    ; CODE XREF: sub_F4283DEE+87j  

.text:F4283E7A             shr     cl, 4  

.text:F4283E7D             mov     [esi], al  

.text:F4283E7F             dec     esi  

.text:F4283E80             cmp     cl, 9  

.text:F4283E83             movzx   eax, cl  

.text:F4283E86             ja      short loc_F4283E8D  

.text:F4283E88             add     eax, 30h  

.text:F4283E8B             jmp     short loc_F4283E90  

.text:F4283E8D ; ---------------------------------------------------------------------------  

.text:F4283E8D  

.text:F4283E8D loc_F4283E8D:                    ; CODE XREF: sub_F4283DEE+98j  

.text:F4283E8D             add     eax, 37h  

.text:F4283E90  

.text:F4283E90 loc_F4283E90:                    ; CODE XREF: sub_F4283DEE+9Dj  

.text:F4283E90             inc     [ebp+arg_8]  

.text:F4283E93             mov     [esi], al  

.text:F4283E95             mov     eax, [ebp+arg_8]  

.text:F4283E98             cmp     eax, [ebp+arg_C]  

.text:F4283E9B             jl      short loc_F4283E57  

.text:F4283E9D             mov     ebx, [ebp+var_18]  

.text:F4283EA0  

.text:F4283EA0 loc_F4283EA0:                    ; CODE XREF: sub_F4283DEE+67j  

.text:F4283EA0             mov     esi, [ebp+var_10]  

.text:F4283EA3             add     esi, [ebp+var_4]  
```
