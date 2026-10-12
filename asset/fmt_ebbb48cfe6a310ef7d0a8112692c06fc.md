
```
                push    [obuf]
                push    [fhandle]
                call    WriteFile
                call    __restart
                mov     byte ptr [esi], 0e9h            ; disable xref scannin'
                jmp     __disasm_next
__x3_0:                 xor     eax, eax                        ; grab bl
                jmp     __grab_bl
__x3_1:                 mov     al, [esi+1]
__grab_bl:              movzx   eax, al
                mov     [xbl], eax
                jmp     __disasm_next
__x4_0:                 xor     eax, eax
                jmp     __grab_dl
__x4_1:                 mov     al, [esi+1]
__grab_dl:              movzx   eax, al
                mov     [xdl], eax
                jmp     __disasm_next
__disasm_end:           popad
                retn
__restart:              mov     edi, [obuf]                     ; and restart
                pusha
                mov     ecx, [isize]
                xor     eax, eax
                rep     stosb
                popa
                or      [fuckr], 1
                retn
; -----------------------------------------------------------------------------
__parse_call:           add     eax, [eax+1]
                add     eax, 5
                retn
strsux                  endp
                        end     start
```
4dd466d0e1e0cc123afbe504da2462ca
