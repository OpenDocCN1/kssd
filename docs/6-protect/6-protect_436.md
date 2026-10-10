

# 还原 hying 最后的八连环 SEH

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-668.htm`](https://www.pediy.com/kssd/pediy07/pediy7-668.htm)

虽然没啥作用，不过看看也是挺有意思的事情。  
  
  
; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
;   
; CHAINDRIVE.ASM - reversing hying's SEH chain drive  
;  
;                       by forgot/iPB  
;  
; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  
  
                        .386  
                        .model  flat, stdcall  
                        option  casemap:none  
  
                        assume  fs : flat  
  
                        .code  
  
start:                  call    delta  
delta:                  pop     ebp  
                        sub     ebp, delta  
                        call    chain\_drive             ; test it  
  
                        retn  
  
; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
;  
; seh chain structure  
;  
; +0x00                 DWORD   except\_code  
; +0x04                 DWORD   new\_origin  
; +0x08                 DWORD   dr0  
; +0x0c                 DWORD   dr1  
; +0x10                 DWORD   dr2  
; +0x14                 DWORD   dr3  
; +0x18                 DWORD   dr6  
; +0x1c                 DWORD   dr7  
;  
;                       total size = 4\*8 = 0x20  
;  
; \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  
  
chain\_drive:  
;                       mov     esi, codebase\[ebp\]      ; calculate hash of user code  
;                       add     esi, imagebase\[ebp\]  
;                       mov     ecx, codesize\[ebp\]  
;                       call    crc32\_esi\_ecx  
;                       mov     saved\_hash\[ebp\], eax  
  
                        mov     eax, ebp                ; eax = ebp = delta  
                        lea     esi, sehchain\[ebp\]  
  
                        add     \[esi+4\], eax            ; fixup  
                        add     \[esi+8\], eax  
                        add     esi, 20h                ; next  
  
                        add     \[esi+4\], eax  
                        add     esi, 20h  
  
                        add     \[esi+4\], eax  
                        add     \[esi+8\], eax  
                        add     esi, 20h  
  
                        add     \[esi+4\], eax  
                        add     esi, 20h  
  
                        add     \[esi+4\], eax  
                        add     esi, 20h  
  
                        add     \[esi+4\], eax  
                        add     esi, 20h  
  
                        add     \[esi+4\], eax  
                        add     esi, 20h  
  
                        add     \[esi+4\], eax  
  
                        lea     esi, sehchain\_ptr\[ebp\]  
                        add     \[esi\], eax  
  
                        lea     eax, sehchain\_handler\[ebp\]  
                        push    eax  
  
                        push    dword ptr fs:\[0\]  
                        mov     dword ptr fs:\[0\], esp  
  
                        xor     eax, eax  
except\_1:               mov     eax, \[eax\]  
  
                        db      "FIGHT WITH THE BEST, AND DIE LIKE THE REST!"  
  
origin\_1:               nop  
  
except\_2:               nop  
  
                        int     3  
except\_3:               jmp     origin\_1                ; trash  
  
                        db      "HOW CAN I PUT SOMEONE TO THE TEST WITH I THOUGHT I GOT THE BEST?"  
  
  
origin\_2:               pushfd  
except\_4:               pushfd  
                        pop     eax  
                        or      ah, 1  
                        push    eax                     ; set TF = 1  
                        popfd  
except\_5:               popfd  
                        jmp     origin\_2  
  
                        db      "UNTIL THE TASTE OF BITTERNESS THEN I REGRET."  
  
  
origin\_3:               xor     eax, eax  
except\_6:               div     eax  
                        jmp     origin\_3                ; simplified  
  
                        db      "CHRISTINA OH MY GODDESS!"  
  
  
origin\_4:               inc     eax  
                        ror     eax, 1                  ; set OF = 1  
                        into  
except\_7:               jmp     origin\_4  
  
                        db      "MUAHAHA STRINGS EMULATED THE JUNK INSTRUCTIONS~"  
  
  
except\_8:               bound   eax, boundlimit\[ebp\]  
                        jmp     except\_8  
  
                        db      "YOU SAY YOU LET YOUR GUARD DOWN? I THINK YOU'RE JUST A WEENIE!"  
  
  
sehchain\_done:          pop     dword ptr fs:\[0\]  
                        pop     eax  
  
  
  
  
                        retn                            ; return  
  
  
  
  
sehchain\_handler:       mov     edx, esp                ; edx = current stack ptr  
                        pusha  
  
                        mov     edi, \[edx+4\*3\]  
                        mov     ebp, \[edi+0b4h\]         ; ctx.ebp  
  
                        mov     esi, sehchain\_ptr\[ebp\]  
  
                        mov     ebx, \[edx+4\]            ; exception record  
  
                        lodsd                           ; exception code  
                        cmp     eax, \[ebx\]  
                        jne     \_\_ignore  
  
                        mov     dword ptr \[edi\], 10017h ; ctx.ctxflags = ctrl | drx | segs | integer  
  
                        lodsd                           ; new origin  
                        mov     \[edi+0b8h\], eax  
  
                        lea     edi, \[edi+4\]            ; skip ctx flags  
  
                        ; (sucked from stack magic)  
                        movsd                           ; dr0  
                        movsd                           ; dr1  
                        movsd                           ; dr2  
                        movsd                           ; dr3  
                        movsd                           ; dr6  
                        movsd                           ; dr7  
  
                        add     sehchain\_ptr\[ebp\], 20h  ; point to next structure  
  
                        popa                            ; continue executing  
                        xor     eax, eax  
                        retn                            ; i optimized ;-)  
  
\_\_ignore:               popa                            ; unknown  
                        sub     eax, eax  
                        inc     eax  
                        retn  
  
  
sehchain\_ptr            dd      sehchain  
  
                        ; memory access violation  
  
sehchain                dd      0c0000005h  
                        dd      origin\_1  
                        dd      except\_2  
                        dd      0  
                        dd      0  
                        dd      0  
                        dd      0  
                        dd      101h  
  
                        ; single step  
  
                        dd      80000004h  
                        dd      except\_2  
                        dd      0  
                        dd      0  
                        dd      0  
                        dd      0  
                        dd      0  
                        dd      0  
  
                        ; int 3 command  
  
                        dd      80000003h  
                        dd      origin\_2  
                        dd      except\_4  
                        dd      0  
                        dd      0  
                        dd      0  
                        dd      0  
                        dd      101h  
  
                        ; single step  
  
                        dd      80000004h  
                        dd      except\_4  
                        dd      0  
                        dd      0  
                        dd      0  
                        dd      0  
                        dd      0  
                        dd      0  
  
                        ; trap  
  
                        dd      80000004h  
                        dd      origin\_3  
                        dd      0  
                        dd      0  
                        dd      0  
                        dd      0  
                        dd      0  
                        dd      0  
  
                        ; integer division by zero  
  
                        dd      0c0000094h  
                        dd      origin\_4  
                        dd      0  
                        dd      0  
                        dd      0  
                        dd      0  
                        dd      0  
                        dd      0  
  
                        ; integer overflow  
  
                        dd      0c0000095h  
                        dd      except\_8  
                        dd      0  
                        dd      0  
                        dd      0  
                        dd      0  
                        dd      0  
                        dd      0  
  
                        ; array bounds exceeded  
  
                        dd      0c000008ch  
                        dd      sehchain\_done  
                        dd      0  
                        dd      0  
                        dd      0  
                        dd      0  
boundlimit              dd      0                       ; a little optimization X-D  
                        dd      0  
  
  
                        end     start 

 4dd466d0e1e0cc123afbe504da2462ca