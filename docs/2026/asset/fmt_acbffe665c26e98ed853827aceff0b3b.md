
```asm
.text:00488F66 -630                mov     [ebp+var_F8], esi
.text:00488F6C -630                mov     [ebp+var_100], 4008h
.text:00488F76 -630                call    ds:rtcLeftCharVar ; Indirect Call Near Procedure
.text:00488F76
.text:00488F7C -630                lea     edx, [ebp+var_D0] ; Load Effective Address
.text:00488F82 -630                push    edx
.text:00488F83 -62C                call    ds:__vbaStrVarMove ; Indirect Call Near Procedure
.text:00488F83
.text:00488F89 -62C                mov     ebx, ds:__vbaStrMove
.text:00488F8F -62C                mov     edx, eax
.text:00488F91 -62C                lea     ecx, [ebp+var_B8] ; Load Effective Address
.text:00488F97 -62C                call    ebx ; __vbaStrMove ; Indirect Call Near Procedure
.text:00488F97
.text:00488F99 -62C                lea     ecx, [ebp+var_D0] ; Load Effective Address
.text:00488F9F -62C                call    ds:__vbaFreeVar ; Indirect Call Near Procedure
.text:00488F9F
.text:00488FA5 -62C                jmp     loc_48902D      ; Jump
.text:00488FA5
.text:00488FAA     ; ---------------------------------------------------------------------------
.text:00488FAA
.text:00488FAA     loc_488FAA:                             ; CODE XREF: sub_488E70+E4j
.text:00488FAA -63C                dec     eax             ; Decrement by 1
.text:00488FAB -63C                cmp     eax, 8          ; switch 9 cases
.text:00488FAE -63C                ja      short loc_489027 ; default
.text:00488FAE
.text:00488FB0 -63C                jmp     ds:off_489BFC[eax*4] ; switch jump
.text:00488FB0
.text:00488FB7
.text:00488FB7     loc_488FB7:                             ; DATA XREF: sub_488E70+D8Co
.text:00488FB7 -63C                mov     eax, [esi]      ; case 0x0
.text:00488FB9 -63C                push    eax
.text:00488FBA -638                push    offset a123456789 ; "123456789"
.text:00488FBF -634                jmp     short loc_48900F ; Jump
.text:00488FBF
.text:00488FC1     ; ---------------------------------------------------------------------------
.text:00488FC1
.text:00488FC1     loc_488FC1:                             ; CODE XREF: sub_488E70+140j
.text:00488FC1                                             ; DATA XREF: sub_488E70+D8Co
.text:00488FC1 -63C                mov     ecx, [esi]      ; case 0x1
.text:00488FC3 -63C                push    ecx
.text:00488FC4 -638                push    offset a23456789 ; "23456789"
.text:00488FC9 -634                jmp     short loc_48900F ; Jump
.text:00488FC9
.text:00488FCB     ; ---------------------------------------------------------------------------
.text:00488FCB
.text:00488FCB     loc_488FCB:                             ; CODE XREF: sub_488E70+140j
.text:00488FCB                                             ; DATA XREF: sub_488E70+D8Co
.text:00488FCB -63C                mov     edx, [esi]      ; case 0x2
.text:00488FCD -63C                push    edx
.text:00488FCE -638                push    offset a3456789 ; "3456789"
.text:00488FD3 -634                jmp     short loc_48900F ; Jump
.text:00488FD3
.text:00488FD5     ; ---------------------------------------------------------------------------
.text:00488FD5
.text:00488FD5     loc_488FD5:                             ; CODE XREF: sub_488E70+140j
.text:00488FD5                                             ; DATA XREF: sub_488E70+D8Co
.text:00488FD5 -63C                mov     eax, [esi]      ; case 0x3
.text:00488FD7 -63C                push    eax
.text:00488FD8 -638                push    offset a456789  ; "456789"
.text:00488FDD -634                jmp     short loc_48900F ; Jump
.text:00488FDD
.text:00488FDF     ; ---------------------------------------------------------------------------
.text:00488FDF
.text:00488FDF     loc_488FDF:                             ; CODE XREF: sub_488E70+140j
.text:00488FDF                                             ; DATA XREF: sub_488E70+D8Co
.text:00488FDF -63C                mov     ecx, [esi]      ; case 0x4
.text:00488FE1 -63C                push    ecx
.text:00488FE2 -638                push    offset a56789   ; "56789"
.text:00488FE7 -634                jmp     short loc_48900F ; Jump
.text:00488FE7
.text:00488FE9     ; ---------------------------------------------------------------------------
.text:00488FE9
.text:00488FE9     loc_488FE9:                             ; CODE XREF: sub_488E70+140j
.text:00488FE9                                             ; DATA XREF: sub_488E70+D8Co
.text:00488FE9 -63C                mov     edx, [esi]      ; case 0x5
.text:00488FEB -63C                push    edx
.text:00488FEC -638                push    offset a6789    ; "6789"
.text:00488FF1 -634                jmp     short loc_48900F ; Jump
.text:00488FF1
.text:00488FF3     ; ---------------------------------------------------------------------------
.text:00488FF3
.text:00488FF3     loc_488FF3:                             ; CODE XREF: sub_488E70+140j
.text:00488FF3                                             ; DATA XREF: sub_488E70+D8Co
.text:00488FF3 -63C                mov     eax, [esi]      ; case 0x6
.text:00488FF5 -63C                push    eax
.text:00488FF6 -638                push    offset a789     ; "789"
.text:00488FFB -634                jmp     short loc_48900F ; Jump
.text:00488FFB
.text:00488FFD     ; ---------------------------------------------------------------------------
.text:00488FFD
.text:00488FFD     loc_488FFD:                             ; CODE XREF: sub_488E70+140j
.text:00488FFD                                             ; DATA XREF: sub_488E70+D8Co
.text:00488FFD -63C                mov     ecx, [esi]      ; case 0x7
.text:00488FFF -63C                push    ecx
.text:00489000 -638                push    offset a89      ; "89"
.text:00489005 -634                jmp     short loc_48900F ; Jump
.text:00489005
.text:00489007     ; ---------------------------------------------------------------------------
.text:00489007
.text:00489007     loc_489007:                             ; CODE XREF: sub_488E70+140j
.text:00489007                                             ; DATA XREF: sub_488E70+D8Co
.text:00489007 -63C                mov     edx, [esi]      ; case 0x8
.text:00489009 -63C                push    edx
.text:0048900A -638                push    offset dword_436BFC
.text:0048900F
.text:0048900F     loc_48900F:                             ; CODE XREF: sub_488E70+14Fj
.text:0048900F                                             ; sub_488E70+159j ...
.text:0048900F -634                call    ds:__vbaStrCat  ; Indirect Call Near Procedure
.text:0048900F
.text:00489015 -638                mov     ebx, ds:__vbaStrMove
.text:0048901B -638                mov     edx, eax
.text:0048901D -638                lea     ecx, [ebp+var_B8] ; Load Effective Address
.text:00489023 -638                call    ebx ; __vbaStrMove ; Indirect Call Near Procedure
.text:00489023
.text:00489025 -638                jmp     short loc_48902D ; Jump
.text:00489025
.text:00489027     ; ---------------------------------------------------------------------------
.text:00489027
.text:00489027     loc_489027:                             ; CODE XREF: sub_488E70+13Ej
.text:00489027 -63C                mov     ebx, ds:__vbaStrMove ; default
.text:0048902D
.text:0048902D     loc_48902D:                             ; CODE XREF: sub_488E70+135j
.text:0048902D                                             ; sub_488E70+1B5j
.text:0048902D -63C                mov     esi, 1
.text:00489032
.text:00489032     loc_489032:                             ; CODE XREF: sub_488E70+3BFj
.text:00489032 -63C                mov     eax, 0Ah
.text:00489037 -63C                cmp     esi, eax        ; Compare Two Operands
```
