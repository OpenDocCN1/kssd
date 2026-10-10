

# The Headless Horsemen Engine v1.03

> 原文：[`www.pediy.com/kssd/pediy10/61317.html`](https://www.pediy.com/kssd/pediy10/61317.html)

代码:

```
; ---------------------------------------------------------------------------
;
; The Headless Horsemen Engine v1.03
;
; (x) forgot 2k7 - 2k8
;
; Size = 94 bytes
;
; input:                esi = input buffer
;                       edi = output buffer
;                       edx = virtual address delta
;
; output:               eax = output size
;
; ---------------------------------------------------------------------------
hhe                     proc
 
                        push    esi
                        push    edi
 
                        lodsb
 
                        cmp     al, 0E8h
                        jz      \_\_e8e9
 
                        cmp     al, 0E9h
                        jnz     \_\_x0
 
\_\_e8e9:                 stosb
 
\_\_fixup\_dword:          lodsd
 
\_\_store\_dword:          sub     eax, edx
                        stosd
 
\_\_exit:                 pop     eax
                        sub     edi, eax
                        pop     esi
                        xchg    eax, edi
                        retn
 
; ---------------------------------------------------------------------------
 
\_\_x0:                   cmp     al, 0Fh
                        jnz     \_\_x1
 
                        lodsb
                        and     al, 0F0h
                        cmp     al, 80h
                        jnz     \_\_exit
 
                        dec     esi
                        dec     esi
                        movsw
                        jmp     \_\_fixup\_dword
 
; ---------------------------------------------------------------------------
 
\_\_x1:                   add     edx, 3
 
                        cmp     al, 0EBh
                        jnz     \_\_x2
 
\_\_eb:                   mov     al, 0E9h
                        stosb
 
\_\_fixup\_byte:           lodsb
                        movsx   eax, al
                        jmp     \_\_store\_dword
 
; ---------------------------------------------------------------------------
\_\_x2:                   inc     edx
 
                        mov     ah, al
                        and     al, 0FCh
                        cmp     al, 0E0h
                        jz      \_\_x3
 
                        and     al, 0F3h
                        cmp     al, 70h
                        jnz     \_\_exit
 
                        xor     ah, 0F0h
                        mov     al, 0Fh
                        stosw
                        jmp     \_\_fixup\_byte
 
; ---------------------------------------------------------------------------
 
\_\_x3:                   add     edx, 4
                        movzx   eax, ah
                        or      eax, 5EB0200h
                        stosd
                        jmp     \_\_eb
 
hhe                     endp
; ---------------------------------------------------------------------------

```

经典应用 pseudocode：  

代码:

```
while (copied\_size < 5)
{
  n = lde(src);
  nx = hhe(src, dst, dst - src);
  if ( nx == 0 )
  {
    memcpy(dst, src, n);
    dst += n;
  }
  else
  {
    dst += nx;
  }
  src += n;  
  copied\_size += n;
}

```