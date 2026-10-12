
```
.text:0042F877             xor     ebp, [ebx+0Ch]
.text:0042F87A             add     esi, 8F1BBCDCh
.text:0042F880             xor     ebp, [ebx+34h]
.text:0042F883             add     eax_B, esi
.text:0042F885             mov     esi, ecx_D
.text:0042F887             shr     ecx_D, 2
.text:0042F88A             xor     ebp, [ebx+2Ch]
.text:0042F88D             shl     esi, 1Eh
.text:0042F890             or      esi, ecx_D
.text:0042F892             mov     ecx_D, esi      ; R3(C,D,E,A,B,58)
.text:0042F892                              ; **********第 3.18 次运算 END****
.text:0042F894             mov     esi, ecx_D      ; **********第 3.19 次运算开始****
.text:0042F896             mov     edi, ecx_D
.text:0042F898             or      esi, edx_C
.text:0042F89A             and     edi, edx_C
.text:0042F89C             and     esi, [esp+14h+E]
.text:0042F8A0             or      esi, edi
.text:0042F8A2             mov     edi, [ebx+20h]
.text:0042F8A5             xor     edi, [ebx+0Ch]
.text:0042F8A8             xor     edi, [ebx+34h]
.text:0042F8AB             xor     edi, [ebx+2Ch]
.text:0042F8AE             shr     ebp, 1Fh
.text:0042F8B1             add     edi, edi
.text:0042F8B3             or      edi, ebp
.text:0042F8B5             mov     ebp, eax_B
.text:0042F8B7             mov     [ebx+2Ch], edi
.text:0042F8BA             add     esi, edi
.text:0042F8BC             mov     edi, eax_B
.text:0042F8BE             shl     edi, 5
.text:0042F8C1             shr     ebp, 1Bh
.text:0042F8C4             or      edi, ebp
.text:0042F8C6             add     esi, edi
.text:0042F8C8             add     esi, 8F1BBCDCh
.text:0042F8CE             add     [esp+14h+A], esi
.text:0042F8D2             mov     esi, edx_C
.text:0042F8D4             shr     edx_C, 2
.text:0042F8D7             mov     edi, [ebx+UI32._9]
.text:0042F8DA             shl     esi, 1Eh
.text:0042F8DD             xor     edi, [ebx+UI32._4]
.text:0042F8E0             or      esi, edx_C
.text:0042F8E2             xor     edi, [ebx+UI32._14]
.text:0042F8E5             mov     edx_C, esi      ; R3(B,C,D,E,A,59)
.text:0042F8E5                              ; **********第 3.19 次运算 END****
.text:0042F8E5                              ; **********第 3 轮计算 END****
.text:0042F8E7             mov     esi, [ebx+UI32._9] ; **********第 4 轮计算开始****
.text:0042F8E7                              ; **********第 4.0 次运算开始****
.text:0042F8EA             xor     esi, [ebx+UI32._4] ; buf[9] ^ buf[4]
.text:0042F8ED             xor     edi, [ebx+UI32._12]
.text:0042F8F0             xor     esi, [ebx+UI32._14] ; buf[9] ^ buf[4] ^ buf[14]
.text:0042F8F3             xor     esi, [ebx+UI32._12] ; buf[9] ^ buf[4] ^ buf[14] ^ buf[12]
.text:0042F8F6             shr     edi, 1Fh        ; (buf[9] ^ buf[4] ^ buf[14] ^ buf[12])>>31
.text:0042F8F9             add     esi, esi        ; (buf[9] ^ buf[4] ^ buf[14] ^ buf[12])<<1
.text:0042F8FB             or      esi, edi        ; RL((buf[9] ^ buf[4] ^ buf[14] ^ buf[12]),1)
.text:0042F8FD             mov     edi, edx_C
.text:0042F8FF             xor     edi, eax_B      ; B^C
.text:0042F901             mov     [ebx+UI32._12], esi ; RXL[60]
.text:0042F904             xor     edi, ecx_D      ; RX1(B,C,D) = B^C^D
.text:0042F906             mov     ebp, [esp+14h+A]
.text:0042F90A             add     esi, edi        ; RXL[60] + RX1(B,C,D)
.text:0042F90C             mov     edi, [esp+14h+A]
.text:0042F910             shl     edi, 5          ; A<<5
.text:0042F913             shr     ebp, 1Bh        ; A>>27
.text:0042F916             or      edi, ebp        ; RL(A,5)
.text:0042F918             add     esi, edi        ; RXL[60] + RX1(B,C,D) + RL(A,5)
.text:0042F91A             add     esi, 0CA62C1D6h
.text:0042F920             add     [esp+14h+E], esi ; E += RXL[60] + RX1(B,C,D) + RL(A,5) + 0xca62c1d6
.text:0042F924             mov     esi, eax_B
.text:0042F926             shr     eax_B, 2        ; B>>2
.text:0042F929             mov     edi, [ebx+UI32._10]
.text:0042F92C             shl     esi, 1Eh        ; B<<30
.text:0042F92F             xor     edi, [ebx+UI32._5]
.text:0042F932             or      esi, eax_B      ; RL(B,30)
.text:0042F934             xor     edi, [ebx+UI32._15]
.text:0042F937             mov     eax_B, esi      ; B = RL(B,30)
.text:0042F937                              ;
.text:0042F937                              ; 定义宏 R4:
.text:0042F937                              ; #define R4(a,b,c,d,e,i) ( e+=RXL(i)+RX1(b,c,d)+RL(a,5)+0xca62c1d6; b=RL(b,30); )
.text:0042F937                              ;
.text:0042F937                              ; 那么这里用 R4 表示为: R4(A,B,C,D,E,60)
.text:0042F937                              ;
.text:0042F937                              ; **********第 4.0 次运算 END****
.text:0042F939             mov     esi, [ebx+28h]  ; **********第 4.1 次运算开始****
.text:0042F93C             xor     esi, [ebx+14h]
.text:0042F93F             xor     edi, [ebx+34h]
.text:0042F942             xor     esi, [ebx+3Ch]
.text:0042F945             xor     esi, [ebx+34h]
.text:0042F948             shr     edi, 1Fh
.text:0042F94B             add     esi, esi
.text:0042F94D             or      esi, edi
.text:0042F94F             mov     [ebx+34h], esi
.text:0042F952             mov     edi, [esp+14h+A]
.text:0042F956             mov     ebp, [esp+14h+E]
.text:0042F95A             xor     edi, eax_B
.text:0042F95C             xor     edi, edx_C
.text:0042F95E             add     esi, edi
.text:0042F960             mov     edi, [esp+14h+E]
.text:0042F964             shl     edi, 5
.text:0042F967             shr     ebp, 1Bh
.text:0042F96A             or      edi, ebp
.text:0042F96C             add     esi, edi
.text:0042F96E             mov     edi, [esp+14h+A]
.text:0042F972             add     esi, 0CA62C1D6h
.text:0042F978             add     ecx_D, esi
.text:0042F97A             mov     esi, [esp+14h+A]
.text:0042F97E             shl     esi, 1Eh
.text:0042F981             mov     ebp, ecx_D
.text:0042F983             shr     edi, 2
.text:0042F986             or      esi, edi
.text:0042F988             mov     [esp+14h+A], esi ; R4(E,A,B,C,D,61)
.text:0042F988                              ; **********第 4.1 次运算 END****
.text:0042F98C             mov     edi, [ebx+2Ch]  ; **********第 4.2 次运算开始****
.text:0042F98F             mov     esi, [ebx+2Ch]
.text:0042F992             xor     edi, [ebx+18h]
.text:0042F995             xor     esi, [ebx+18h]
.text:0042F998             xor     edi, [ebx]
.text:0042F99A             xor     esi, [ebx]
```
