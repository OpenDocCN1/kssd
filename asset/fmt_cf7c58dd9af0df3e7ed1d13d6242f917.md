
```asm
.text:00489ADA -634                lea     eax, [ebp+var_20] ; Load Effective Address  

.text:00489ADD -634                push    ecx  

.text:00489ADE -630                push    edx  

.text:00489ADF -62C                mov     [ebp+var_F8], eax  

.text:00489AE5 -62C                mov     [ebp+var_100], edi  

.text:00489AEB -62C                call    ds:rtcTrimVar   ; Indirect Call Near Procedure  

.text:00489AEB    



.text:00489AF1 -62C                lea     eax, [ebp+var_D0] ; Load Effective Address  

.text:00489AF7 -62C                push    eax  

.text:00489AF8 -628                call    esi ; __vbaStrVarMove ; Indirect Call Near Procedure  

.text:00489AF8    



.text:00489AFA -628                mov     edx, eax  

.text:00489AFC -628                lea     ecx, [ebp+var_B4] ; Load Effective Address  

.text:00489B02 -628                call    ebx ; __vbaStrMove ; Indirect Call Near Procedure  

.text:00489B02    



.text:00489B04 -628                lea     ecx, [ebp+var_D0] ; Load Effective Address  

.text:00489B0A -628                call    ds:__vbaFreeVar ; Indirect Call Near Procedure  

.text:00489B0A    



.text:00489B10 -628                wait                    ; Wait until BUSY# Pin is Inactive (HIGH)  

.text:00489B11 -628                push    offset loc_489BE0  

.text:00489B16 -624                jmp     short loc_489B57 ; Jump  

.text:00489B16    



.text:00489B18     ; ---------------------------------------------------------------------------  

.text:00489B18 -624                test    byte ptr [ebp-4], 4 ; Logical Compare  

.text:00489B1C -624                jz      short loc_489B2A ; Jump if Zero (ZF=1)  

.text:00489B1C    



.text:00489B1E -624                lea     ecx, [ebp-0B4h] ; Load Effective Address  

.text:00489B24 -624                call    ds:__vbaFreeStr ; Indirect Call Near Procedure  

.text:00489B24    



.text:00489B2A    



.text:00489B2A     loc_489B2A:                         ; CODE XREF: sub_488E70+CACj  

.text:00489B2A -624                lea     ecx, [ebp-0C0h] ; Load Effective Address  

.text:00489B30 -624                call    ds:__vbaFreeStr ; Indirect Call Near Procedure  

.text:00489B30    



.text:00489B36 -624                lea     ecx, [ebp-0F0h] ; Load Effective Address  

.text:00489B3C -624                lea     edx, [ebp-0E0h] ; Load Effective Address  

.text:00489B42 -624                push    ecx  

.text:00489B43 -624                lea     eax, [ebp-0D0h] ; Load Effective Address  

.text:00489B49 -624                push    edx  

.text:00489B4A -624                push    eax  

.text:00489B4B -624                push    3  

.text:00489B4D -624                call    ds:__vbaFreeVarList ; Indirect Call Near Procedure  

.text:00489B4D    



.text:00489B53 -630                add     esp, 10h        ; Add  

.text:00489B56 -640                retn                    ; Return Near from Procedure  

.text:00489B56    



.text:00489B57     ; ---------------------------------------------------------------------------  

.text:00489B57    



.text:00489B57     loc_489B57:                         ; CODE XREF: sub_488E70+CA6j  

.text:00489B57 -624                mov     edi, ds:__vbaFreeStr  

.text:00489B5D -624                lea     ecx, [ebp+var_1C] ; Load Effective Address  

.text:00489B60 -624                call    edi ; __vbaFreeStr ; Indirect Call Near Procedure  

.text:00489B60    



.text:00489B62 -624                lea     ecx, [ebp+var_20] ; Load Effective Address  

.text:00489B65 -624                call    edi ; __vbaFreeStr ; Indirect Call Near Procedure  

.text:00489B65    



.text:00489B67 -624                mov     esi, ds:__vbaAryDestruct  

.text:00489B6D -624                lea     edx, [ebp+var_114] ; Load Effective Address  

.text:00489B73 -624                lea     ecx, [ebp+var_3C] ; Load Effective Address  

.text:00489B76 -624                push    edx  

.text:00489B77 -620                push    0  

.text:00489B79 -61C                mov     [ebp+var_114], ecx  

.text:00489B7F -61C                call    esi ; __vbaAryDestruct ; Indirect Call Near Procedure  

.text:00489B7F    



.text:00489B81 -61C                lea     ecx, [ebp+var_118] ; Load Effective Address  

.text:00489B87 -61C                lea     eax, [ebp+var_58] ; Load Effective Address  

.text:00489B8A -61C                push    ecx  

.text:00489B8B -618                push    0  

.text:00489B8D -614                mov     [ebp+var_118], eax  

.text:00489B93 -614                call    esi ; __vbaAryDestruct ; Indirect Call Near Procedure  

.text:00489B93    



.text:00489B95 -614                lea     eax, [ebp+var_11C] ; Load Effective Address  

.text:00489B9B -614                lea     edx, [ebp+var_74] ; Load Effective Address  

.text:00489B9E -614                push    eax  

.text:00489B9F -610                push    0  

.text:00489BA1 -60C                mov     [ebp+var_11C], edx  

.text:00489BA7 -60C                call    esi ; __vbaAryDestruct ; Indirect Call Near Procedure  

.text:00489BA7    



.text:00489BA9 -60C                lea     edx, [ebp+var_120] ; Load Effective Address  

.text:00489BAF -60C                lea     ecx, [ebp+var_90] ; Load Effective Address  

.text:00489BB5 -60C                push    edx  

.text:00489BB6 -608                push    0  

.text:00489BB8 -604                mov     [ebp+var_120], ecx  

.text:00489BBE -604                call    esi ; __vbaAryDestruct ; Indirect Call Near Procedure  

.text:00489BBE    



.text:00489BC0 -604                lea     ecx, [ebp+var_124] ; Load Effective Address  

.text:00489BC6 -604                lea     eax, [ebp+var_AC] ; Load Effective Address  

.text:00489BCC -604                push    ecx  

.text:00489BCD -600                push    0  

.text:00489BCF -5FC                mov     [ebp+var_124], eax  

.text:00489BD5 -5FC                call    esi ; __vbaAryDestruct ; Indirect Call Near Procedure  

.text:00489BD5    



.text:00489BD7 -5FC                lea     ecx, [ebp+var_B8] ; Load Effective Address  

.text:00489BDD -5FC                call    edi ; __vbaFreeStr ; Indirect Call Near Procedure  

.text:00489BDD    



.text:00489BDF -5FC                retn                    ; Return Near from Procedure  

.text:00489BDF    



.text:00489BE0     ; ---------------------------------------------------------------------------  

.text:00489BE0    



.text:00489BE0     loc_489BE0:                         ; DATA XREF: sub_488E70+CA1o  

.text:00489BE0 -5FC                mov     ecx, [ebp-14h]  

.text:00489BE3 -5FC                mov     eax, [ebp-0B4h]  

.text:00489BE9 -5FC                pop     edi  

.text:00489BEA -5FC                pop     esi  

.text:00489BEB -5FC                mov     large fs:0, ecx  

.text:00489BF2 -5FC                pop     ebx  

.text:00489BF3 -5FC                mov     esp, ebp  

.text:00489BF5 -5FC                pop     ebp  

.text:00489BF6 -5FC                retn    8               ; Return Near from Procedure  

.text:00489BF6    



.text:00489BF6     ; ---------------------------------------------------------------------------  

.text:00489BF9 -5FC                align 4  

.text:00489BFC -5FC off_489BFC     dd  offset loc_488FB7    ; DATA XREF: sub_488E70+140r  

.text:00489BFC                  dd  offset loc_488FC1    ; jump table for switch statement  

.text:00489BFC                  dd  offset loc_488FCB  

.text:00489BFC                  dd  offset loc_488FD5  

.text:00489BFC                  dd  offset loc_488FDF  

.text:00489BFC                  dd  offset loc_488FE9  

.text:00489BFC                  dd  offset loc_488FF3
```
