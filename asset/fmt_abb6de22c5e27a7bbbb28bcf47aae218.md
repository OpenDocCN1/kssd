
```asm
.text:0042FC75                 xor     edi, [ebx+24h]  

.text:0042FC78                 mov     eax_B, esi      ; R4(A,B,C,D,E,70)  

.text:0042FC78                 ; ***************第 4.10 次运算 END****  

.text:0042FC7A                 mov     esi, [ebx+10h]  ; ***************第 4.11 次运算开始****  

.text:0042FC7D                 xor     esi, [ebx+3Ch]  

.text:0042FC80                 xor     edi, [ebx+1Ch]  

.text:0042FC83                 xor     esi, [ebx+24h]  

.text:0042FC86                 xor     esi, [ebx+1Ch]  

.text:0042FC89                 shr     edi, 1Fh  

.text:0042FC8C                 add     esi, esi  

.text:0042FC8E                 or      esi, edi  

.text:0042FC90                 mov     [ebx+1Ch], esi  

.text:0042FC93                 mov     edi, [esp+14h+A]  

.text:0042FC97                 mov     ebp, [esp+14h+E]  

.text:0042FC9B                 xor     edi, eax_B  

.text:0042FC9D                 xor     edi, edx_C  

.text:0042FC9F                 add     esi, edi  

.text:0042FCA1                 mov     edi, [esp+14h+E]  

.text:0042FCA5                 shl     edi, 5  

.text:0042FCA8                 shr     ebp, 1Bh  

.text:0042FCAB                 or      edi, ebp  

.text:0042FCAD                 add     esi, edi  

.text:0042FCAF                 mov     edi, [esp+14h+A]  

.text:0042FCB3                 add     esi, 0CA62C1D6h  

.text:0042FCB9                 add     ecx_D, esi  

.text:0042FCBB                 mov     esi, [esp+14h+A]  

.text:0042FCBF                 shl     esi, 1Eh  

.text:0042FCC2                 mov     ebp, ecx_D  

.text:0042FCC4                 shr     edi, 2  

.text:0042FCC7                 or      esi, edi  

.text:0042FCC9                 mov     [esp+14h+A], esi ; R4(E,A,B,C,D,71)  

.text:0042FCC9                 ; ***************第 4.11 次运算 END****  

.text:0042FCCD                 mov     edi, [ebx+14h]  ; ***************第 4.12 次运算开始****  

.text:0042FCD0                 mov     esi, [ebx+14h]  

.text:0042FCD3                 xor     edi, [ebx]  

.text:0042FCD5                 xor     esi, [ebx]  

.text:0042FCD7                 xor     edi, [ebx+28h]  

.text:0042FCDA                 xor     esi, [ebx+28h]  

.text:0042FCDD                 xor     edi, [ebx+20h]  

.text:0042FCE0                 xor     esi, [ebx+20h]  

.text:0042FCE3                 shr     edi, 1Fh  

.text:0042FCE6                 add     esi, esi  

.text:0042FCE8                 or      esi, edi  

.text:0042FCEA                 mov     [ebx+20h], esi  

.text:0042FCED                 mov     edi, [esp+14h+E]  

.text:0042FCF1                 xor     edi, [esp+14h+A]  

.text:0042FCF5                 xor     edi, eax_B  

.text:0042FCF7                 add     esi, edi  

.text:0042FCF9                 mov     edi, ecx_D  

.text:0042FCFB                 shl     edi, 5  

.text:0042FCFE                 shr     ebp, 1Bh  

.text:0042FD01                 or      edi, ebp  

.text:0042FD03                 add     esi, edi  

.text:0042FD05                 mov     edi, [esp+14h+E]  

.text:0042FD09                 add     esi, 0CA62C1D6h  

.text:0042FD0F                 add     edx_C, esi  

.text:0042FD11                 mov     esi, [esp+14h+E]  

.text:0042FD15                 shl     esi, 1Eh  

.text:0042FD18                 mov     ebp, edx_C  

.text:0042FD1A                 shr     edi, 2  

.text:0042FD1D                 or      esi, edi  

.text:0042FD1F                 mov     [esp+14h+E], esi ; R4(D,E,A,B,C,72)  

.text:0042FD1F                 ; ***************第 4.12 次运算 END****  

.text:0042FD23                 mov     edi, [ebx+18h]  ; ***************第 4.13 次运算开始****  

.text:0042FD26                 mov     esi, [ebx+18h]  

.text:0042FD29                 xor     edi, [ebx+4]  

.text:0042FD2C                 xor     esi, [ebx+4]  

.text:0042FD2F                 xor     edi, [ebx+2Ch]  

.text:0042FD32                 xor     esi, [ebx+2Ch]  

.text:0042FD35                 xor     edi, [ebx+24h]  

.text:0042FD38                 xor     esi, [ebx+24h]  

.text:0042FD3B                 shr     edi, 1Fh  

.text:0042FD3E                 add     esi, esi  

.text:0042FD40                 or      esi, edi  

.text:0042FD42                 mov     [ebx+24h], esi  

.text:0042FD45                 mov     edi, [esp+14h+E]  

.text:0042FD49                 xor     edi, ecx_D  

.text:0042FD4B                 xor     edi, [esp+14h+A]  

.text:0042FD4F                 add     esi, edi  

.text:0042FD51                 mov     edi, edx_C  

.text:0042FD53                 shr     ebp, 1Bh  

.text:0042FD56                 shl     edi, 5  

.text:0042FD59                 or      edi, ebp  

.text:0042FD5B                 add     esi, edi  

.text:0042FD5D                 add     esi, 0CA62C1D6h  

.text:0042FD63                 add     eax_B, esi  

.text:0042FD65                 mov     esi, ecx_D  

.text:0042FD67                 mov     edi, [ebx+1Ch]  

.text:0042FD6A                 shr     ecx_D, 2  

.text:0042FD6D                 mov     ebp, eax_B  

.text:0042FD6F                 shl     esi, 1Eh  

.text:0042FD72                 xor     edi, [ebx+8]  

.text:0042FD75                 or      esi, ecx_D  

.text:0042FD77                 xor     edi, [ebx+30h]  

.text:0042FD7A                 mov     ecx_D, esi      ; R4(C,D,E,A,B,73)  

.text:0042FD7A                 ; ***************第 4.13 次运算 END****  

.text:0042FD7C                 mov     esi, [ebx+1Ch]  ; ***************第 4.14 次运算开始****  

.text:0042FD7F                 xor     esi, [ebx+8]  

.text:0042FD82                 xor     edi, [ebx+28h]  

.text:0042FD85                 xor     esi, [ebx+30h]  

.text:0042FD88                 xor     esi, [ebx+28h]  

.text:0042FD8B                 shr     edi, 1Fh  

.text:0042FD8E                 add     esi, esi  

.text:0042FD90                 or      esi, edi  

.text:0042FD92                 mov     edi, ecx_D  

.text:0042FD94                 mov     [ebx+28h], esi  

.text:0042FD97                 xor     edi, edx_C  

.text:0042FD99                 xor     edi, [esp+14h+E]  

.text:0042FD9D                 add     esi, edi  

.text:0042FD9F                 mov     edi, eax_B  

.text:0042FDA1                 shl     edi, 5  

.text:0042FDA4                 shr     ebp, 1Bh  

.text:0042FDA7                 or      edi, ebp  

.text:0042FDA9                 add     esi, edi  

.text:0042FDAB                 add     esi, 0CA62C1D6h  

.text:0042FDB1                 add     [esp+14h+A], esi  

.text:0042FDB5                 mov     esi, edx_C  

.text:0042FDB7                 shr     edx_C, 2  

.text:0042FDBA                 mov     edi, [ebx+20h]  

.text:0042FDBD                 shl     esi, 1Eh  

.text:0042FDC0                 xor     edi, [ebx+0Ch]  

.text:0042FDC3                 or      esi, edx_C  

.text:0042FDC5                 xor     edi, [ebx+34h]  

.text:0042FDC8                 mov     edx_C, esi      ; R4(B,C,D,E,A,74)  

.text:0042FDC8                 ; ***************第 4.14 次运算 END****  

.text:0042FDCA                 mov     esi, [ebx+20h]  ; ***************第 4.15 次运算开始****  

.text:0042FDCD                 xor     esi, [ebx+0Ch]  

.text:0042FDD0                 xor     edi, [ebx+2Ch]  

.text:0042FDD3                 xor     esi, [ebx+34h]  
```
