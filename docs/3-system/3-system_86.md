

# 资源表重建代码

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-539.htm`](https://www.pediy.com/kssd/pediy07/pediy7-539.htm)

100 行之内吧。  
  
; Resource table rebuilder  
;      coded by forgot/iPB  
; no optimition yet.          
        
      .386  
      .Model Flat,StdCall  
      Option CaseMap:None  
  
      .Data  
  
imagebuf    db  1024 dup(0)  
  
resourcebase    dd  0  
newresbase    dd  0  
  
      .code  
  
start:      mov  eax, 400000h  
      mov  ecx, \[eax+3Ch\]  
      add  ecx, eax  
      mov  edx, \[ecx+88h\]  
      add  edx, eax  
      mov  resourcebase, edx  
  
      lea  edi, imagebuf  
      mov  newresbase, edi  
      call  \_\_move\_rsrc  
  
      retn  
  
; input: edx = offset, edi = stream  
; output: edi = new stream  
  
\_\_move\_rsrc:    movzx  ecx, word ptr \[edx+0Ch\]  
      movzx  eax, word ptr \[edx+0Eh\]  
      add  ecx, eax  
  
      push  ecx  
      mov  esi, edx  
      mov  edx, edi  
      shl  ecx, 03h  
      add  ecx, 10h  
      call  \_\_memmov  
      pop  ecx  
  
      add  edx, 10h  
  
\_\_rsrc\_cycle:    jecxz  \_\_rsrc\_end  
      push  ecx  
  
      mov  eax, \[edx\]  
      btr  eax, 31  
      jnc  \_\_skip\_copy\_name  
  
      add  eax, resourcebase  
      xchg  esi, eax  
  
      mov  eax, edi  
      sub  eax, newresbase  
      bts  eax, 31  
      push  eax  
  
\_\_next\_char:    lodsb  
      and  byte ptr \[esi-1\], 0  
      test  al, al  
      jz  \_\_str\_end  
      stosb  
      jmp  \_\_next\_char  
\_\_str\_end:    
      pop  eax  
      mov  \[edx\], eax  
  
\_\_skip\_copy\_name:  mov  eax, \[edx+04h\]  
      btr  eax, 31  
      xchg  esi, eax  
      jnc  \_\_not\_dir  
  
      add  esi, resourcebase  
  
      mov  eax, edi  
      sub  eax, newresbase  
      bts  eax, 31  
      push  eax  
  
      push  edx  
      xchg  edx, esi  
      call  \_\_move\_rsrc  
      pop  edx  
  
      pop  eax  
      jmp  \_\_fix\_offs  
  
\_\_not\_dir:    mov  eax, edi  
      sub  eax, newresbase  
      push  eax  
  
      add  esi, resourcebase  
      mov  ecx, 10h  
      call  \_\_memmov  
        
      pop  eax  
  
\_\_fix\_offs:    mov  \[edx+04h\], eax  
      add  edx, 08h  
  
      pop  ecx  
      dec  ecx  
      jmp  \_\_rsrc\_cycle  
  
\_\_rsrc\_end:    retn  
  
\_\_memmov:    jecxz  \_\_rsrc\_end  
      movsb  
      and  byte ptr \[esi-1\], 0  
      dec  ecx  
      jmp  \_\_memmov  
  
      end  start 

 4dd466d0e1e0cc123afbe504da2462ca