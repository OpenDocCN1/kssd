
```
> VM_FYL2X        proc near                   ; DATA XREF: ????1536o  
> 
>                 fyl2x  
> 
>                 jmp     VMLoop_EP           ; fetch Instruction Byte  
> 
> VM_FYL2X        endp  
> 
> ; ????S U B R O U T I N E ????
> 
> VM_MUL_F        proc near                   ; DATA XREF: ????1566o  
> 
>                 pop     edx  
> 
>                 pop     eax  
> 
>                 mul     edx  
> 
>                 push    eax  
> 
>                 push    edx  
> 
>                 pushfw  
> 
>                 jmp     VMLoop_EP           ; fetch Instruction Byte  
> 
> VM_MUL_F        endp  
> 
> ; ????
> 
> VM_PUSH_IMMW:                          ; DATA XREF: ????13DEo  
> 
>                 lodsw  
> 
>                 add     ax, bx  
> 
>                 xor     ax, 6609h  
> 
>                 not     ax  
> 
>                 ror     ax, 8  
> 
>                 dec     ax  
> 
>                 add     bx, ax  
> 
>                 cwde  
> 
>                 push    eax  
> 
>                 jmp     VMLoop_EP           ; fetch Instruction Byte  
> 
> ; ????
> 
> VM_NA_NB_AND_F:                          ; DATA XREF: ????1526o  
> 
>                 pop     ecx  
> 
>                 not     ecx  
> 
>                 not     dword ptr [esp]  
> 
>                 and     [esp], ecx  
> 
>                 pushfw  
> 
>                 jmp     VMLoop_EP           ; fetch Instruction Byte  
> 
> ; ????
> 
> VM_RETN:                              ; DATA XREF: ????1406o  
> 
>                 pop     eax  
> 
>                 popa  
> 
>                 popf  
> 
>                 retn  
> 
> ; ????S U B R O U T I N E ????
> 
> ; save status word to dest  
> 
> VM_FNSTSWW      proc near                   ; DATA XREF: ????16BEo  
> 
>                 fnstsw  ax  
> 
>                 push    ax  
> 
>                 jmp     VMLoop_EP           ; fetch Instruction Byte  
> 
> VM_FNSTSWW      endp  
> 
> ; ????S U B R O U T I N E ????
> 
> VM_PUSH_CR5     proc near                   ; DATA XREF: ????174Eo  
> 
>                 mov     edx, cr5  
> 
>                 push    edx  
> 
>                 jmp     VMLoop_EP           ; fetch Instruction Byte  
> 
> VM_PUSH_CR5     endp  
> 
> ; ????S U B R O U T I N E ????
> 
> ; st(0) <- ?(ie, pi)  
> 
> VM_FLDPI        proc near                   ; DATA XREF: ????16AAo  
> 
>                 fldpi  
> 
>                 jmp     VMLoop_EP           ; fetch Instruction Byte  
> 
> 
> VM_FLDPI        endp  
> 
> ; ????S U B R O U T I N E ????
> 
> VM_PUSH_IMM     proc near                   ; DATA XREF: ????1726o  
> 
>                 mov     eax, [esi]  
> 
>                 add     eax, ebx  
> 
>                 add     eax, 23D30F4Fh ; <suspicious>  
> 
>                 rol     eax, 1Fh  
> 
>                 inc     eax  
> 
>                 add     esi, 4  
> 
>                 rol     eax, 0Bh  
> 
>                 sub     eax, 8E5C6C84h  
> 
>                 add     ebx, eax  
> 
>                 push    eax  
> 
>                 jmp     VMLoop_EP           ; fetch Instruction Byte  
> 
> VM_PUSH_IMM     endp  
> 
> ; ????S U B R O U T I N E ????
> 
> VM_PUSHW_CONTEXTBL proc near                ; DATA XREF: ????158Ao  
> 
>                 lodsb  
> 
>                 add     al, bl  
> 
>                 xor     al, 0B9h  
> 
>                 inc     al  
> 
>                 not     al  
> 
>                 ror     al, 6  
> 
>                 add     al, 0F2h  
> 
>                 add     bl, al  
> 
>                 mov     al, [edi+eax*4]  
> 
>                 push    ax  
> 
>                 jmp     VMLoop_EP           ; fetch Instruction Byte  
> 
> VM_PUSHW_CONTEXTBL endp  
> 
> ; ????S U B R O U T I N E ????
> 
> VM_POP_CR7      proc near                   ; DATA XREF: ????14B2o  
> 
>                 pop     edx  
> 
>                 mov     cr7, edx  
> 
>                 jmp     VMLoop_EP           ; fetch Instruction Byte  
> 
> VM_POP_CR7      endp  
> 
> ; ????S U B R O U T I N E ????
> 
> VM_FMULQ        proc near                   ; DATA XREF: ????171Eo  
> 
>                 fmul    qword ptr [esp+0]  
> 
>                 jmp     VMLoop_EP           ; fetch Instruction Byte  
> 
> VM_FMULQ        endp  
> 
> ; ????S U B R O U T I N E ????
> 
> VM_MOV_CSA_TO_B proc near                   ; DATA XREF: ????147Eo  
> 
>                 pop     ecx  
> 
>                 push    dword ptr cs:[ecx]  
> 
>                 jmp     VMLoop_EP           ; fetch Instruction Byte  
> 
> VM_MOV_CSA_TO_B endp  
> 
> ; ????S U B R O U T I N E ????
> 
> VM_MOV_B_TO_GSA proc near                   ; DATA XREF: ????1472o  
> 
>                 pop     eax  
> 
>                 pop     dword ptr gs:[eax]  
> 
>                 jmp     VMLoop_EP           ; fetch Instruction Byte  
> 
> VM_MOV_B_TO_GSA endp  
> 
> ; ????S U B R O U T I N E ????
> 
> VM_POP_CR5      proc near                   ; DATA XREF: ????153Ao  
> 
>                 pop     eax  
> 
>                 mov     cr5, eax  
> 
>                 jmp     VMLoop_EP           ; fetch Instruction Byte  
> 
> 
> VM_POP_CR5      endp  
> 
> ; ????
> 
> VM_ADD_B_F:                              ; DATA XREF: ????13AEo  
> 
>                               ; ????13C2o  
> 
>                 pop     cx  
> 
>                 add     [esp], cl  
> 
>                 pushfw  
> 
>                 jmp     VMLoop_EP           ; fetch Instruction Byte  
> 
> ; ????S U B R O U T I N E ????
> 
> VM_PUSH_CONTEXT proc near                   ; DATA XREF: ????1502o  
> 
>                 lodsb  
> 
>                 add     al, bl  
> 
>                 xor     al, 4Ah  
> 
>                 ror     al, 4  
> 
>                 sub     al, 0B7h  
> 
>                 rol     al, 6  
> 
>                 add     bl, al  
> 
>                 push    dword ptr [edi+eax*4]  
> 
>                 jmp     VMLoop_EP           ; fetch Instruction Byte  
> 
> VM_PUSH_CONTEXT endp  
> 
> ; ????S U B R O U T I N E ????
> 
> VM_MOV_B_TO_CSA proc near                   ; DATA XREF: ????13A2o  
> 
>                 pop     ecx  
> 
>                 pop     ax  
> 
>                 mov     cs:[ecx], al  
> 
>                 jmp     VMLoop_EP           ; fetch Instruction Byte  
> 
> VM_MOV_B_TO_CSA endp  
> 
> ; ????S U B R O U T I N E ????
> 
> VM_SHRW_F       proc near                   ; DATA XREF: ????173Eo  
> 
>                 pop     ax  
> 
>                 pop     cx  
> 
>                 shr     ax, cl  
> 
>                 push    ax  
> 
>                 pushfw  
> 
>                 jmp     VMLoop_EP           ; fetch Instruction Byte  
> 
> VM_SHRW_F       endp  
> 
> ; ????S U B R O U T I N E ????
> 
> VM_FCOMP        proc near                   ; DATA XREF: ????15A6o  
> 
>                 fcomp   dword ptr [esp+0]  
> 
>                 jmp     VMLoop_EP           ; fetch Instruction Byte  
> 
> VM_FCOMP        endp  
> 
> ; ????S U B R O U T I N E ????
> 
> VM_POP_DR2      proc near                   ; DATA XREF: ????13FAo  
> 
>                 pop     ecx  
> 
>                 mov     dr2, ecx  
> 
>                 jmp     VMLoop_EP           ; fetch Instruction Byte  
```
