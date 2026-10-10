

# 第一题 32 字节的

> 原文：[`www.pediy.com/kssd/pediy11/123007.html`](https://www.pediy.com/kssd/pediy11/123007.html)

代码:

```
0012FC78    6A 00           push    0
0012FC7A    68 6C604000     push    40606C                           ; UNICODE "ExploitMe"
0012FC7F    E8 20000000     call    0012FCA4
0012FC84    45              inc     ebp
0012FC85    0078 00         add     byte ptr \[eax\], bh
0012FC88    70 00           jo      short 0012FC8A
0012FC8A    6C              ins     byte ptr es:\[edi\], dx
0012FC8B    006F 00         add     byte ptr \[edi\], ch
0012FC8E    6900 74002000   imul    eax, dword ptr \[eax\], 200074
0012FC94    73 00           jnb     short 0012FC96
0012FC96    75 00           jnz     short 0012FC98
0012FC98    6300            arpl    word ptr \[eax\], ax
0012FC9A    6300            arpl    word ptr \[eax\], ax
0012FC9C    65:0073 00      add     byte ptr gs:\[ebx\], dh
0012FCA0    73 00           jnb     short 0012FCA2
0012FCA2    0000            add     byte ptr \[eax\], al
0012FCA4  - E9 23152D00     jmp     ExploitM.004011CC

```

上传的附件
| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || exploit.rar  
 |