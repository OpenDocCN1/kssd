
```
.text:00488FD5     loc_488FD5:                           ; CODE XREF: sub_488E70+140j
.text:00488FD5                                       ; DATA XREF: sub_488E70+D8Co
.text:00488FD5 -63C                 mov     eax, [esi]      ; case 0x3
.text:00488FD7 -63C                 push    eax
.text:00488FD8 -638                 push    offset a456789  ; "456789"
.text:00488FDD -634                 jmp     short loc_48900F ; Jump

.text:00488FDF     ; ---------------------------------------------------------------

.text:00488FDF     loc_488FDF:                           ; CODE XREF: sub_488E70+140j
.text:00488FDF                                       ; DATA XREF: sub_488E70+D8Co
.text:00488FDF -63C                 mov     ecx, [esi]      ; case 0x4
.text:00488FE1 -63C                 push    ecx
.text:00488FE2 -638                 push    offset a56789   ; "56789"
.text:00488FE7 -634                 jmp     short loc_48900F ; Jump

.text:00488FE9     ; ---------------------------------------------------------------

.text:00488FE9     loc_488FE9:                           ; CODE XREF: sub_488E70+140j
.text:00488FE9                                       ; DATA XREF: sub_488E70+D8Co
.text:00488FE9 -63C                 mov     edx, [esi]      ; case 0x5
.text:00488FEB -63C                 push    edx
.text:00488FEC -638                 push    offset a6789    ; "6789"
.text:00488FF1 -634                 jmp     short loc_48900F ; Jump

.text:00488FF3     ; ---------------------------------------------------------------

.text:00488FF3     loc_488FF3:                           ; CODE XREF: sub_488E70+140j
.text:00488FF3                                       ; DATA XREF: sub_488E70+D8Co
.text:00488FF3 -63C                 mov     eax, [esi]      ; case 0x6
.text:00488FF5 -63C                 push    eax
.text:00488FF6 -638                 push    offset a789     ; "789"
.text:00488FFB -634                 jmp     short loc_48900F ; Jump

.text:00488FFD     ; ---------------------------------------------------------------

.text:00488FFD     loc_488FFD:                           ; CODE XREF: sub_488E70+140j
.text:00488FFD                                       ; DATA XREF: sub_488E70+D8Co
.text:00488FFD -63C                 mov     ecx, [esi]      ; case 0x7
.text:00488FFF -63C                 push    ecx
.text:00489000 -638                 push    offset a89      ; "89"
.text:00489005 -634                 jmp     short loc_48900F ; Jump

.text:00489007     ; ---------------------------------------------------------------

.text:00489007     loc_489007:                           ; CODE XREF: sub_488E70+140j
.text:00489007                                       ; DATA XREF: sub_488E70+D8Co
.text:00489007 -63C                 mov     edx, [esi]      ; case 0x8
.text:00489009 -63C                 push    edx
.text:0048900A -638                 push    offset dword_436BFC

.text:0048900F     loc_48900F:                           ; CODE XREF: sub_488E70+14Fj
.text:0048900F                                       ; sub_488E70+159j ...
.text:0048900F -634                 call    ds:__vbaStrCat  ; Indirect Call Near Procedure

.text:00489015 -638                 mov     ebx, ds:__vbaStrMove
.text:0048901B -638                 mov     edx, eax
.text:0048901D -638                 lea     ecx, [ebp+var_B8] ; Load Effective Address
.text:00489023 -638                 call    ebx ; __vbaStrMove ; Indirect Call Near Procedure

.text:00489025 -638                 jmp     short loc_48902D ; Jump

.text:00489027     ; ---------------------------------------------------------------

.text:00489027     loc_489027:                           ; CODE XREF: sub_488E70+13Ej
.text:00489027 -63C                 mov     ebx, ds:__vbaStrMove ; default

.text:0048902D     loc_48902D:                           ; CODE XREF: sub_488E70+135j
.text:0048902D                                       ; sub_488E70+1B5j
.text:0048902D -63C                 mov     esi, 1

.text:00489032     loc_489032:                           ; CODE XREF: sub_488E70+3BFj
.text:00489032 -63C                 mov     eax, 0Ah
.text:00489037 -63C                 cmp     esi, eax        ; Compare Two Operands
.text:00489039 -63C                 jg      loc_489234      ; Jump if Greater (ZF=0 & SF=OF)

.text:0048903F -63C                 lea     eax, [ebp+var_B8] ; Load Effective Address
.text:00489045 -63C                 lea     ecx, [ebp+var_D0] ; Load Effective Address
.text:0048904B -63C                 mov     [ebp+var_F8], eax
.text:00489051 -63C                 push    ecx
.text:00489052 -638                 lea     edx, [ebp+var_100] ; Load Effective Address
.text:00489058 -638                 push    esi
.text:00489059 -634                 lea     eax, [ebp+var_E0] ; Load Effective Address
.text:0048905F -634                 push    edx
.text:00489060 -630                 push    eax
.text:00489061 -62C                 mov     [ebp+var_C8], 1
.text:0048906B -62C                 mov     [ebp+var_D0], 2
.text:00489075 -62C                 mov     [ebp+var_100], 4008h
.text:0048907F -62C                 call    ds:rtcMidCharVar ; Indirect Call Near Procedure

.text:00489085 -62C                 lea     ecx, [ebp+var_E0] ; Load Effective Address
.text:0048908B -62C                 push    ecx
.text:0048908C -628                 call    ds:rtcIsNumeric ; Indirect Call Near Procedure

.text:00489092 -628                 mov     di, ax
.text:00489095 -628                 lea     edx, [ebp+var_E0] ; Load Effective Address
.text:0048909B -628                 lea     eax, [ebp+var_D0] ; Load Effective Address
.text:004890A1 -628                 push    edx
.text:004890A2 -624                 push    eax
.text:004890A3 -620                 push    2
.text:004890A5 -61C                 call    ds:__vbaFreeVarList ; Indirect Call Near Procedure

.text:004890AB -628                 add     esp, 0Ch        ; Add
.text:004890AE -634                 test    di, di          ; Logical Compare
.text:004890B1 -634                 jz      loc_489201      ; Jump if Zero (ZF=1)

.text:004890B7 -634                 lea     ecx, [ebp+var_B8] ; Load Effective Address
.text:004890BD -634                 lea     edx, [ebp+var_D0] ; Load Effective Address
.text:004890C3 -634                 mov     [ebp+var_F8], ecx
.text:004890C9 -634                 push    edx
.text:004890CA -630                 lea     eax, [ebp+var_100] ; Load Effective Address
.text:004890D0 -630                 push    esi
.text:004890D1 -62C                 lea     ecx, [ebp+var_E0] ; Load Effective Address
.text:004890D7 -62C                 push    eax
.text:004890D8 -628                 push    ecx
.text:004890D9 -624                 mov     [ebp+var_C8], 1
.text:004890E3 -624                 mov     [ebp+var_D0], 2
.text:004890ED -624                 mov     [ebp+var_100], 4008h
.text:004890F7 -624                 call    ds:rtcMidCharVar ; Indirect Call Near Procedure
```
