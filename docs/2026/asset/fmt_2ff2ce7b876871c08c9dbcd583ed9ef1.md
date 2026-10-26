
```
                        and     edx, $3F
                        xor     ecx, dword ptr [esi+edx*$4+$100]
                        shr     eax, $1A
                        mov     edx, dword ptr [esi+eax*$4+$700]
                        mov     eax, ebp
                        xor     ecx, edx
                        shr     eax, $1A
                        mov     edx, dword ptr [esi+eax*$4+$600]
                        shr     ebp, $2
                        and     ebp, $3F
                        mov     eax, dword ptr [esi+ebp*$4]
                        xor     ecx, edx
                        xor     ecx, eax
                        xor     ebx, ecx
                        mov     ecx, dword ptr [esp+$38]
                        add     esp, $20
                        add     ecx, $20
                        mov     eax, dword ptr [esp+$20]
                        dec     eax
                        mov     dword ptr [esp+$18], ecx
                        mov     dword ptr [esp+$20], eax
                        jnz @L061
                        jmp @L402
@L230:
                        add     eax, $70
                        mov     dword ptr [esp+$18], eax
                        mov     dword ptr [esp+$20], $4
                        jmp @L235
@L234:
                        mov     eax, dword ptr [esp+$18]
@L235:
                        mov     ebp, dword ptr [eax+$8]
                        mov     eax, dword ptr [eax+$C]
                        xor     eax, ebx
                        push    $4
                        push    eax
                        xor     ebp, ebx
                        call    __0089F57B
                        mov     ecx, eax
                        shr     ecx, $12
                        and     ecx, $3F
                        mov     ecx, dword ptr [esi+ecx*$4+$500]
                        mov     edx, ebp
                        shr     edx, $12
                        and     edx, $3F
                        xor     ecx, dword ptr [esi+edx*$4+$400]
                        mov     edx, eax
                        shr     edx, $0A
                        and     edx, $3F
                        xor     ecx, dword ptr [esi+edx*$4+$300]
                        mov     edx, ebp
                        shr     edx, $0A
                        and     edx, $3F
                        xor     ecx, dword ptr [esi+edx*$4+$200]
                        mov     edx, eax
                        shr     eax, $1A
                        shr     edx, $2
                        and     edx, $3F
                        xor     ecx, dword ptr [esi+edx*$4+$100]
                        mov     edx, dword ptr [esi+eax*$4+$700]
                        mov     eax, ebp
                        shr     eax, $1A
                        xor     ecx, edx
                        mov     edx, dword ptr [esi+eax*$4+$600]
                        shr     ebp, $2
                        and     ebp, $3F
                        mov     eax, dword ptr [esi+ebp*$4]
                        xor     ecx, edx
                        xor     ecx, eax
                        mov     eax, dword ptr [esp+$20]
                        mov     ebp, dword ptr [eax]
                        mov     eax, dword ptr [eax+$4]
                        xor     edi, ecx
                        xor     eax, edi
                        push    $4
                        push    eax
                        xor     ebp, edi
                        call    __0089F57B
                        mov     ecx, eax
                        shr     ecx, $12
                        and     ecx, $3F
                        mov     ecx, dword ptr [esi+ecx*$4+$500]
                        mov     edx, ebp
                        shr     edx, $12
                        and     edx, $3F
                        xor     ecx, dword ptr [esi+edx*$4+$400]
                        mov     edx, eax
                        shr     edx, $0A
                        and     edx, $3F
                        xor     ecx, dword ptr [esi+edx*$4+$300]
                        mov     edx, ebp
                        shr     edx, $0A
                        and     edx, $3F
                        xor     ecx, dword ptr [esi+edx*$4+$200]
                        mov     edx, eax
                        shr     edx, $2
                        shr     eax, $1A
                        and     edx, $3F
                        xor     ecx, dword ptr [esi+edx*$4+$100]
                        mov     edx, dword ptr [esi+eax*$4+$700]
                        mov     eax, ebp
                        shr     eax, $1A
                        xor     ecx, edx
                        mov     edx, dword ptr [esi+eax*$4+$600]
                        shr     ebp, $2
                        and     ebp, $3F
                        mov     eax, dword ptr [esi+ebp*$4]
                        xor     ecx, edx
                        xor     ecx, eax
                        mov     eax, dword ptr [esp+$28]
                        mov     ebp, dword ptr [eax-$8]
                        xor     ebx, ecx
                        mov     eax, dword ptr [eax-$4]
                        xor     eax, ebx
                        push    $4
                        push    eax
                        xor     ebp, ebx
                        call    __0089F57B
                        mov     ecx, eax
                        shr     ecx, $12
                        and     ecx, $3F
                        mov     ecx, dword ptr [esi+ecx*$4+$500]
                        mov     edx, ebp
                        shr     edx, $12
                        and     edx, $3F
                        xor     ecx, dword ptr [esi+edx*$4+$400]
                        mov     edx, eax
                        shr     edx, $0A
                        and     edx, $3F
                        xor     ecx, dword ptr [esi+edx*$4+$300]
                        mov     edx, ebp
                        shr     edx, $0A
                        and     edx, $3F
                        xor     ecx, dword ptr [esi+edx*$4+$200]
                        mov     edx, eax
                        shr     eax, $1A
                        shr     edx, $2
                        and     edx, $3F
                        xor     ecx, dword ptr [esi+edx*$4+$100]
                        mov     edx, dword ptr [esi+eax*$4+$700]
                        mov     eax, ebp
                        shr     eax, $1A
                        xor     ecx, edx
                        mov     edx, dword ptr [esi+eax*$4+$600]
                        shr     ebp, $2
                        and     ebp, $3F
                        mov     eax, dword ptr [esi+ebp*$4]
                        xor     ecx, edx
                        xor     ecx, eax
                        mov     eax, dword ptr [esp+$30]
                        mov     ebp, dword ptr [eax-$10]
                        mov     eax, dword ptr [eax-$C]
                        xor     edi, ecx
                        xor     eax, edi
                        push    $4
                        push    eax
                        xor     ebp, edi
                        call    __0089F57B
                        mov     ecx, eax
                        shr     ecx, $12
                        and     ecx, $3F
                        mov     ecx, dword ptr [esi+ecx*$4+$500]
                        mov     edx, ebp
                        shr     edx, $12
                        and     edx, $3F
                        xor     ecx, dword ptr [esi+edx*$4+$400]
                        mov     edx, eax
                        shr     edx, $0A
                        and     edx, $3F
                        xor     ecx, dword ptr [esi+edx*$4+$300]
                        mov     edx, ebp
                        shr     edx, $0A
                        and     edx, $3F
                        xor     ecx, dword ptr [esi+edx*$4+$200]
                        mov     edx, eax
                        shr     eax, $1A
                        shr     edx, $2
                        and     edx, $3F
```
