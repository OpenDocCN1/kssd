
```
.text:0042F99C                xor     edi, [ebx+38h]  
.text:0042F99F                xor     esi, [ebx+38h]  
.text:0042F9A2                shr     edi, 1Fh  
.text:0042F9A5                add     esi, esi  
.text:0042F9A7                or      esi, edi  
.text:0042F9A9                mov     [ebx+38h], esi  
.text:0042F9AC                mov     edi, [esp+14h+E]  
.text:0042F9B0                xor     edi, [esp+14h+A]  
.text:0042F9B4                xor     edi, eax_B  
.text:0042F9B6                add     esi, edi  
.text:0042F9B8                mov     edi, ecx_D  
.text:0042F9BA                shl     edi, 5  
.text:0042F9BD                shr     ebp, 1Bh  
.text:0042F9C0                or      edi, ebp  
.text:0042F9C2                add     esi, edi  
.text:0042F9C4                mov     edi, [esp+14h+E]  
.text:0042F9C8                add     esi, 0CA62C1D6h  
.text:0042F9CE                add     edx_C, esi  
.text:0042F9D0                mov     esi, [esp+14h+E]  
.text:0042F9D4                shl     esi, 1Eh  
.text:0042F9D7                mov     ebp, edx_C  
.text:0042F9D9                shr     edi, 2  
.text:0042F9DC                or      esi, edi  
.text:0042F9DE                mov     [esp+14h+E], esi ; R4(D,E,A,B,C,62)  
.text:0042F9DE                                ; ***************第 4.2 次运算 END****  
.text:0042F9E2                mov     edi, [ebx+30h]  ; ***************第 4.3 次运算开始****  
.text:0042F9E5                mov     esi, [ebx+30h]  
.text:0042F9E8                xor     edi, [ebx+1Ch]  
.text:0042F9EB                xor     esi, [ebx+1Ch]  
.text:0042F9EE                xor     edi, [ebx+4]  
.text:0042F9F1                xor     esi, [ebx+4]  
.text:0042F9F4                xor     edi, [ebx+3Ch]  
.text:0042F9F7                xor     esi, [ebx+3Ch]  
.text:0042F9FA                shr     edi, 1Fh  
.text:0042F9FD                add     esi, esi  
.text:0042F9FF                or      esi, edi  
.text:0042FA01                mov     [ebx+3Ch], esi  
.text:0042FA04                mov     edi, [esp+14h+E]  
.text:0042FA08                xor     edi, ecx_D  
.text:0042FA0A                xor     edi, [esp+14h+A]  
.text:0042FA0E                add     esi, edi  
.text:0042FA10                mov     edi, edx_C  
.text:0042FA12                shl     edi, 5  
.text:0042FA15                shr     ebp, 1Bh  
.text:0042FA18                or      edi, ebp  
.text:0042FA1A                add     esi, edi  
.text:0042FA1C                mov     edi, [ebx+34h]  
.text:0042FA1F                add     esi, 0CA62C1D6h  
.text:0042FA25                xor     edi, [ebx+20h]  
.text:0042FA28                add     eax_B, esi  
.text:0042FA2A                mov     esi, ecx_D  
.text:0042FA2C                shr     ecx_D, 2  
.text:0042FA2F                mov     ebp, eax_B  
.text:0042FA31                shl     esi, 1Eh  
.text:0042FA34                xor     edi, [ebx+8]  
.text:0042FA37                or      esi, ecx_D  
.text:0042FA39                xor     edi, [ebx]  
.text:0042FA3B                mov     ecx_D, esi      ; R4(C,D,E,A,B,63)  
.text:0042FA3B                                ; ***************第 4.3 次运算 END****  
.text:0042FA3D                mov     esi, [ebx+34h]  ; ***************第 4.4 次运算开始****  
.text:0042FA40                xor     esi, [ebx+20h]  
.text:0042FA43                xor     esi, [ebx+8]  
.text:0042FA46                xor     esi, [ebx]  
.text:0042FA48                shr     edi, 1Fh  
.text:0042FA4B                add     esi, esi  
.text:0042FA4D                or      esi, edi  
.text:0042FA4F                mov     edi, ecx_D  
.text:0042FA51                mov     [ebx], esi  
.text:0042FA53                xor     edi, edx_C  
.text:0042FA55                xor     edi, [esp+14h+E]  
.text:0042FA59                add     esi, edi  
.text:0042FA5B                mov     edi, eax_B  
.text:0042FA5D                shr     ebp, 1Bh  
.text:0042FA60                shl     edi, 5  
.text:0042FA63                or      edi, ebp  
.text:0042FA65                add     esi, edi  
.text:0042FA67                add     esi, 0CA62C1D6h  
.text:0042FA6D                add     [esp+14h+A], esi  
.text:0042FA71                mov     esi, edx_C  
.text:0042FA73                shr     edx_C, 2  
.text:0042FA76                shl     esi, 1Eh  
.text:0042FA79                or      esi, edx_C  
.text:0042FA7B                mov     edx_C, esi      ; R4(B,C,D,E,A,64)  
.text:0042FA7B                                ; ***************第 4.4 次运算 END****  
.text:0042FA7D                mov     esi, [ebx+38h]  ; ***************第 4.5 次运算开始****  
.text:0042FA80                xor     esi, [ebx+24h]  
.text:0042FA83                xor     esi, [ebx+0Ch]  
.text:0042FA86                xor     esi, [ebx+4]  
.text:0042FA89                add     esi, esi  
.text:0042FA8B                mov     edi, [ebx+38h]  
.text:0042FA8E                xor     edi, [ebx+24h]  
.text:0042FA91                xor     edi, [ebx+0Ch]  
.text:0042FA94                xor     edi, [ebx+4]  
.text:0042FA97                shr     edi, 1Fh  
.text:0042FA9A                or      esi, edi  
.text:0042FA9C                mov     edi, edx_C  
.text:0042FA9E                xor     edi, eax_B  
.text:0042FAA0                mov     [ebx+4], esi  
.text:0042FAA3                xor     edi, ecx_D  
.text:0042FAA5                mov     ebp, [esp+14h+A]  
.text:0042FAA9                add     esi, edi  
.text:0042FAAB                mov     edi, [esp+14h+A]  
.text:0042FAAF                shl     edi, 5  
.text:0042FAB2                shr     ebp, 1Bh  
.text:0042FAB5                or      edi, ebp  
.text:0042FAB7                add     esi, edi  
.text:0042FAB9                add     esi, 0CA62C1D6h  
.text:0042FABF                add     [esp+14h+E], esi  
.text:0042FAC3                mov     esi, eax_B  
.text:0042FAC5                shr     eax_B, 2  
.text:0042FAC8                mov     edi, [ebx+3Ch]  
.text:0042FACB                shl     esi, 1Eh  
.text:0042FACE                xor     edi, [ebx+28h]  
.text:0042FAD1                or      esi, eax_B  
.text:0042FAD3                xor     edi, [ebx+10h]  
.text:0042FAD6                mov     eax_B, esi      ; R4(A,B,C,D,E,65)  
.text:0042FAD6                                ; ***************第 4.5 次运算 END****  
.text:0042FAD8                mov     esi, [ebx+3Ch]  ; ***************第 4.6 次运算开始****  
.text:0042FADB                xor     esi, [ebx+28h]  
.text:0042FADE                xor     edi, [ebx+8]  
.text:0042FAE1                xor     esi, [ebx+10h]  
.text:0042FAE4                xor     esi, [ebx+8]  
.text:0042FAE7                shr     edi, 1Fh  
.text:0042FAEA                add     esi, esi  
.text:0042FAEC                or      esi, edi  
.text:0042FAEE                mov     [ebx+8], esi  
.text:0042FAF1                mov     edi, [esp+14h+A]  
.text:0042FAF5                mov     ebp, [esp+14h+E]  
.text:0042FAF9                xor     edi, eax_B  
.text:0042FAFB                xor     edi, edx_C  
.text:0042FAFD                add     esi, edi  
.text:0042FAFF                mov     edi, [esp+14h+E]  
.text:0042FB03                shl     edi, 5  
```
