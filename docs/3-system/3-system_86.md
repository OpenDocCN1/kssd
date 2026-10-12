
# 资源表重建代码

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-539.htm`](https://www.pediy.com/kssd/pediy07/pediy7-539.htm)

100 行之内吧。

```asm
; Resource table rebuilder  
;     coded by forgot/iPB  
; no optimition yet.        

      .386  
      .Model Flat,StdCall  
      Option CaseMap:None  
      .Data  
imagebuf    db  1024 dup(0)  
resourcebase    dd  0  
newresbase    dd  0  

      .code  
start:      mov  eax, 400000h  
      mov  ecx, [eax+3Ch]  
      add  ecx, eax  
      mov  edx, [ecx+88h]  
      add  edx, eax  
      mov  resourcebase, edx  
      lea  edi, imagebuf  
      mov  newresbase, edi  
      call  __move_rsrc  
      retn  

; input: edx = offset, edi = stream  
; output: edi = new stream  

__move_rsrc:    movzx  ecx, word ptr [edx+0Ch]  
      movzx  eax, word ptr [edx+0Eh]  
      add  ecx, eax  
      push  ecx  
      mov  esi, edx  
      mov  edx, edi  
      shl  ecx, 03h  
      add  ecx, 10h  
      call  __memmov  
      pop  ecx  
      add  edx, 10h  

__rsrc_cycle:    jecxz  __rsrc_end  
      push  ecx  
      mov  eax, [edx]  
      btr  eax, 31  
      jnc  __skip_copy_name  
      add  eax, resourcebase  
      xchg  esi, eax  
      mov  eax, edi  
      sub  eax, newresbase  
      bts  eax, 31  
      push  eax  

__next_char:    lodsb  
      and  byte ptr [esi-1], 0  
      test  al, al  
      jz  __str_end  
      stosb  
      jmp  __next_char  

__str_end:   

      pop  eax  
      mov  [edx], eax  

__skip_copy_name:  mov  eax, [edx+04h]  
      btr  eax, 31  
      xchg  esi, eax  
      jnc  __not_dir  
      add  esi, resourcebase  
      mov  eax, edi  
      sub  eax, newresbase  
      bts  eax, 31  
      push  eax  
      push  edx  
      xchg  edx, esi  
      call  __move_rsrc  
      pop  edx  
      pop  eax  
      jmp  __fix_offs  

__not_dir:    mov  eax, edi  
      sub  eax, newresbase  
      push  eax  
      add  esi, resourcebase  
      mov  ecx, 10h  
      call  __memmov  
      pop  eax  

__fix_offs:    mov  [edx+04h], eax  
      add  edx, 08h  
      pop  ecx  
      dec  ecx  
      jmp  __rsrc_cycle  

__rsrc_end:    retn  

__memmov:    jecxz  __rsrc_end  
      movsb  
      and  byte ptr [esi-1], 0  
      dec  ecx  
      jmp  __memmov  

      end  start 

 4dd466d0e1e0cc123afbe504da2462ca
```
