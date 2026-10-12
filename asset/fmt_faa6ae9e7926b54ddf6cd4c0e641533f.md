
```
xor ecx, ecx

@@move_cycle:    jecxz   @@return

                movsb

                and     byte ptr [esi-1], 0
                dec     ecx
                jmp     @@move_cycle

; ----------------------------------------------------------------------------

@@done:          pop     eax
                cmp     level, 0
                jnz     @@return
                popa
                leave

@@return:
                retn

restruc_rsrc     endp

; ----------------------------------------------------------------------------

                end start
```

注释

> ```
> 代码:
>
> * * *
>
> ;
> ; RESOURCE RESTURCE ENGINE v1.00 
> ;
> ; forgot / iPB 2006
> ;
> ;
> ; 1. swap root entries, make GROUP ICON first
> ; 2. when Id=24/16/14, move 1st one
> ; 3. when Id=3, query GROUP ICON table, if being in, move it.
> ; 4. otherwise would be preserved
> ; 5. the subrountine makes a compressable rsrc section.
> ;    it's not made for ResEditor, take care!
> ; 6. I just scan once, with a little optimized(size) code
>
> restruc_rsrc    proc  C  \\
>        ibuf,  \\
>        imgbase,\\
>        rva,  \\
>        obuf
>
>       local  permuted
>       local  level
>       local  rtype
>       local  id
>       local  gicon
>       local  numoficons
>       local  rmask
>
> ; ---------------------------------------------------------------------------
>
>       pusha
>
>       xor  eax, eax
>       mov  level, eax
>       mov  gicon, eax
>       mov  numoficons, eax
>       mov  permuted, eax
>       
>       mov  rmask, 1000000010100000000001000b
>               ; 1<<3 | 1<<14 | 1<<16 | 1<<24
>
>       mov  ebx, ibuf
>       mov  edi, obuf       
>
> ; ---------------------------------------------------------------------------
>
> @@recursive:    movzx  eax, word ptr [ebx+0Ch]
>       movzx  ecx, word ptr [ebx+0Eh]
>       add  eax, ecx
>
>       mov  esi, ebx
>       lea  ebx, [edi+10h]    ; ebx -> copied entries
>       lea  ecx, [eax+2]    ; size = 10h + eax * 8
>       shl  ecx, 3
>       call  @@move
>
>       push  eax      ; # of entries
>
> ; ---------------------------------------------------------------------------
>
> @@cycle:    dec  dword ptr [esp]
>       jl  @@done
>
> ; ---------------------------------------------------------------------------
>
>       cmp  permuted, 0
>       jne  @@permuted
>
>       mov  ecx, [esp]
>
>       pusha
>       inc  permuted
>
>       mov  edx, ebx    ; edx -> 1st entry
>
> @@permute_cycle:  mov  eax, [ebx]
>       cmp  eax, 14
>       jne  @@dont_swap
>       cmp  ebx, edx
>       je  @@dont_swap
>
>       xchg  eax, [edx]    ; [edx] <--> [ebx]
>       mov  [ebx], eax
>
>       mov  eax, [ebx+4]
>       xchg  eax, [edx+4]
>       mov  [ebx+4], eax
>       
> @@dont_swap:    add  ebx, 8
>       loop  @@permute_cycle
>       popa
>
> @@permuted:
>
> ; ---------------------------------------------------------------------------
>
>       mov  esi, [ebx]
>       btr  esi, 31
>       jnc  @@id
>
>       mov  eax, edi    ; update name ofs
>       sub  eax, obuf
>       bts  eax, 31
>       mov  [ebx], eax
>
>       add  esi, ibuf
>       movzx  ecx, word ptr [esi]  ; unicode
>       lea  ecx, [ecx+ecx+2]
>       call  @@move
>
>       jmp  @@x1
>
> ; ---------------------------------------------------------------------------
>
> @@id:      cmp  level, 1
>       jg  @@x1
>       mov  id, esi      ; <= 1
>       je  @@x1
>       mov  rtype, esi    ; == 0
> @@x1:
>
> ; ---------------------------------------------------------------------------
>
>       mov  edx, [ebx+4]
>       add  edx, ibuf
>       btr  edx, 31
>       jnc  @@rsrc
>
>       mov  eax, edi    ; next level directory
>       sub  eax, obuf
>       bts  eax, 31
>       push  eax      ; update l8r
>
>       inc  level
>
>       push  ebx       
>       xchg  ebx, edx
>       call  @@recursive
>       pop  ebx
>
>       dec  level
>       jmp  @@fixup
>
> ; ---------------------------------------------------------------------------
>
> @@rsrc:      mov  eax, rtype
>       btr  rmask, eax
>       jnc  @@dont_move
>
>       cmp  eax, 3
>       jne  @@x3
>
>       mov  esi, gicon
>       test  esi, esi
>       je  @@dont_move
>
>       bts  rmask, eax
>
>       mov  eax, id
>       mov  ecx, numoficons
>
> @@icon_cycle:    cmp  ax, [esi+0Ch]
>       je  @@do_move
>       add  esi, 0Eh
>       loop  @@icon_cycle
>
>       jmp  @@dont_move
>
>
> ; ---------------------------------------------------------------------------
>
> @@x3:      cmp  eax, 14
>       jne  @@do_move
>
>       push  edx
>       mov  eax, [edx+4]
>       sub  eax, 6
>       cdq
>       mov  ecx, 0Eh
>       div  ecx
>       mov  numoficons, eax
>       pop  edx
>
> ; ---------------------------------------------------------------------------
>
> @@do_move:    mov  esi, edi
>       sub  esi, obuf
>       add  esi, rva
>       xchg  esi, [edx]
>       add  esi, imgbase
>       mov  ecx, [edx+4]
>       call  @@move
>
> ; ---------------------------------------------------------------------------
>
> @@dont_move:    mov  eax, edi
>       sub  eax, obuf
>       push  eax      ; update l8r
>
>       xchg  esi, edx
>       mov  edx, edi
>
>       push  10h
>       pop  ecx
>       call  @@move
>
> ; ---------------------------------------------------------------------------
>
>       cmp  numoficons, 0
>       je  @@fixup
>       cmp  gicon, 0
>       jne  @@fixup
>
>       mov  eax, [edx]
>       add  eax, imgbase
>       add  eax, 6
>       mov  gicon, eax
>
> ; ---------------------------------------------------------------------------
>
> @@fixup:    pop  eax
>       mov  [ebx+4], eax
>
>       add  ebx, 8
>       jmp  @@cycle
>
> ; ---------------------------------------------------------------------------
>
> @@move:      cmp  esi, edi
>       jne  @@move_cycle
>
>       add  edi, ecx
>       xor  ecx, ecx
>
> @@move_cycle:    jecxz  @@return
>       movsb
>       and  byte ptr [esi-1], 0
>       dec  ecx
>       jmp  @@move_cycle
>
> ; ---------------------------------------------------------------------------
>
> @@done:      pop  eax
>
>       cmp  level, 0
>       jne  @@return
>
>       popa
>       leave
>
> @@return:    retn
>
> ; ---------------------------------------------------------------------------
>
> restruc_rsrc    endp
>
> * * *
>
>
> ```
