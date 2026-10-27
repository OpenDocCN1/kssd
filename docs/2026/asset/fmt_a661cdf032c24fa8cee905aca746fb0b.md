
```
.text:004891FF -61C                jmp     short loc_489220 ; Jump
.text:004891FF
.text:00489201     ; ---------------------------------------------------------------------------
.text:00489201
.text:00489201     loc_489201:                             ; CODE XREF: sub_488E70+241j
.text:00489201 -634                lea     edi, [esi-1]    ; Load Effective Address
.text:00489204 -634                cmp     edi, 0Ah        ; Compare Two Operands
.text:00489207 -634                jb      short loc_48920F ; Jump if Below (CF=1)
.text:00489207
.text:00489209 -634                call    ds:__vbaGenerateBoundsError ; Indirect Call Near Procedure
.text:00489209
.text:0048920F
.text:0048920F     loc_48920F:                             ; CODE XREF: sub_488E70+397j
.text:0048920F -634                mov     ecx, [ebp+var_68]
.text:00489212 -634                mov     edx, offset dword_436AF4
.text:00489217 -634                lea     ecx, [ecx+edi*4] ; Load Effective Address
.text:0048921A -634                call    ds:__vbaStrCopy ; Indirect Call Near Procedure
.text:0048921A
.text:00489220
.text:00489220     loc_489220:                             ; CODE XREF: sub_488E70+38Fj
.text:00489220 -634                mov     eax, 1
.text:00489225 -634                add     eax, esi        ; Add
.text:00489227 -634                jo      loc_489C25      ; Jump if Overflow (OF=1)
.text:00489227
.text:0048922D -634                mov     esi, eax
.text:0048922F -634                jmp     loc_489032      ; Jump
.text:0048922F
.text:00489234     ; ---------------------------------------------------------------------------
.text:00489234
.text:00489234     loc_489234:                             ; CODE XREF: sub_488E70+1C9j
.text:00489234 -63C                mov     esi, 1
.text:00489239 -63C                mov     [ebp+var_18], esi
.text:0048923C
.text:0048923C     loc_48923C:                             ; CODE XREF: sub_488E70+78Dj
.text:0048923C -63C                mov     eax, 9
.text:00489241 -63C                cmp     esi, eax        ; Compare Two Operands
.text:00489243 -63C                jg      loc_489602      ; Jump if Greater (ZF=0 & SF=OF)
.text:00489243
.text:00489249 -63C                dec     esi             ; Decrement by 1
.text:0048924A -63C                cmp     esi, 0Ah        ; Compare Two Operands
.text:0048924D -63C                jb      short loc_489255 ; Jump if Below (CF=1)
.text:0048924D
.text:0048924F -63C                call    ds:__vbaGenerateBoundsError ; Indirect Call Near Procedure
.text:0048924F
.text:00489255
.text:00489255     loc_489255:                             ; CODE XREF: sub_488E70+3DDj
.text:00489255 -63C                mov     edi, [ebp+var_18]
.text:00489258 -63C                add     edi, 1          ; Add
.text:0048925B -63C                jo      loc_489C25      ; Jump if Overflow (OF=1)
.text:0048925B
.text:00489261 -63C                dec     edi             ; Decrement by 1
.text:00489262 -63C                cmp     edi, 0Ah        ; Compare Two Operands
.text:00489265 -63C                jb      short loc_48926D ; Jump if Below (CF=1)
.text:00489265
.text:00489267 -63C                call    ds:__vbaGenerateBoundsError ; Indirect Call Near Procedure
.text:00489267
.text:0048926D
.text:0048926D     loc_48926D:                             ; CODE XREF: sub_488E70+3F5j
.text:0048926D -63C                mov     edx, [ebp+var_68]
.text:00489270 -63C                mov     eax, [edx+esi*4]
.text:00489273 -63C                push    eax
.text:00489274 -638                call    ds:__vbaR8Str   ; Indirect Call Near Procedure
.text:00489274
.text:0048927A -638                fmul    ds:dbl_4017C0   ; Multiply Real
.text:00489280 -638                mov     ecx, [ebp+var_68]
.text:00489283 -638                mov     edx, [ecx+edi*4]
.text:00489286 -638                fnstsw  ax              ; Store Status Word (no wait)
.text:00489288 -638                test    al, 0Dh         ; Logical Compare
.text:0048928A -638                jnz     loc_489C20      ; Jump if Not Zero (ZF=0)
.text:0048928A
.text:00489290 -638                fstp    [ebp+var_170]   ; Store Real and Pop
.text:00489296 -638                push    edx
.text:00489297 -634                call    ds:__vbaR8Str   ; Indirect Call Near Procedure
.text:00489297
.text:0048929D -634                fadd    [ebp+var_170]   ; Add Real
.text:004892A3 -634                sub     esp, 8          ; Integer Subtraction
.text:004892A6 -62C                fnstsw  ax              ; Store Status Word (no wait)
.text:004892A8 -62C                test    al, 0Dh         ; Logical Compare
.text:004892AA -62C                jnz     loc_489C20      ; Jump if Not Zero (ZF=0)
.text:004892AA
.text:004892B0 -62C                fstp    qword ptr [esp+0] ; Store Real and Pop
.text:004892B3 -62C                call    ds:__vbaStrR8   ; Indirect Call Near Procedure
.text:004892B3
.text:004892B9 -62C                mov     edx, eax
.text:004892BB -62C                lea     ecx, [ebp+var_C0] ; Load Effective Address
.text:004892C1 -62C                call    ebx ; __vbaStrMove ; Indirect Call Near Procedure
.text:004892C1
.text:004892C3 -62C                push    eax
.text:004892C4 -628                call    ds:rtcR8ValFromBstr ; Indirect Call Near Procedure
.text:004892C4
.text:004892CA -628                call    ds:__vbaFpI4    ; Indirect Call Near Procedure
.text:004892CA
.text:004892D0 -628                mov     edi, eax
.text:004892D2 -628                and     edi, 80000001h  ; Logical AND
.text:004892D8 -628                jns     short loc_4892DF ; Jump if Not Sign (SF=0)
.text:004892D8
.text:004892DA -628                dec     edi             ; Decrement by 1
.text:004892DB -628                or      edi, 0FFFFFFFEh ; Logical Inclusive OR
.text:004892DE -628                inc     edi             ; Increment by 1
.text:004892DF
.text:004892DF     loc_4892DF:                             ; CODE XREF: sub_488E70+468j
.text:004892DF -628                neg     edi             ; Two's Complement Negation
.text:004892E1 -628                sbb     edi, edi        ; Integer Subtraction with Borrow
.text:004892E3 -628                lea     ecx, [ebp+var_C0] ; Load Effective Address
.text:004892E9 -628                inc     edi             ; Increment by 1
.text:004892EA -628                neg     edi             ; Two's Complement Negation
.text:004892EC -628                call    ds:__vbaFreeStr ; Indirect Call Near Procedure
.text:004892EC
.text:004892F2 -628                test    di, di          ; Logical Compare
.text:004892F5 -628                jz      short loc_48935D ; Jump if Zero (ZF=1)
.text:004892F5
.text:004892F7 -628                cmp     esi, 0Ah        ; Compare Two Operands
.text:004892FA -628                jb      short loc_489302 ; Jump if Below (CF=1)
.text:004892FA
.text:004892FC -628                call    ds:__vbaGenerateBoundsError ; Indirect Call Near Procedure
.text:004892FC
.text:00489302
.text:00489302     loc_489302:                             ; CODE XREF: sub_488E70+48Aj
.text:00489302 -628                mov     edi, [ebp+var_18]
.text:00489305 -628                add     edi, 1          ; Add
.text:00489308 -628                jo      loc_489C25      ; Jump if Overflow (OF=1)
.text:00489308
.text:0048930E -628                dec     edi             ; Decrement by 1
.text:0048930F -628                cmp     edi, 0Ah        ; Compare Two Operands
.text:00489312 -628                jb      short loc_48931A ; Jump if Below (CF=1)
.text:00489312
.text:00489314 -628                call    ds:__vbaGenerateBoundsError ; Indirect Call Near Procedure
.text:00489314
.text:0048931A
```
