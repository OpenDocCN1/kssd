
```
.text:0042F43B                 or      esi, edi  

.text:0042F43D                 mov     [esp+14h+A], esi ; R3(E,A,B,C,D,46)  

.text:0042F43D                                             ; **********第 3.6 次运算 END****  

.text:0042F441                 mov     esi, [esp+14h+E] ; **********第 3.7 次运算开始****  

.text:0042F445                 mov     edi, [esp+14h+E]  

.text:0042F449                 or      esi, [esp+14h+A]  

.text:0042F44D                 and     edi, [esp+14h+A]  

.text:0042F451                 and     esi, eax_B  

.text:0042F453                 mov     ebp, [ebx+30h]  

.text:0042F456                 or      esi, edi  

.text:0042F458                 mov     edi, [ebx+30h]  

.text:0042F45B                 xor     ebp, [ebx+1Ch]  

.text:0042F45E                 xor     edi, [ebx+1Ch]  

.text:0042F461                 xor     ebp, [ebx+4]  

.text:0042F464                 xor     edi, [ebx+4]  

.text:0042F467                 xor     ebp, [ebx+3Ch]  

.text:0042F46A                 xor     edi, [ebx+3Ch]  

.text:0042F46D                 shr     ebp, 1Fh  

.text:0042F470                 add     edi, edi  

.text:0042F472                 or      edi, ebp  

.text:0042F474                 mov     ebp, ecx_D  

.text:0042F476                 mov     [ebx+3Ch], edi  

.text:0042F479                 add     esi, edi  

.text:0042F47B                 mov     edi, ecx_D  

.text:0042F47D                 shl     edi, 5  

.text:0042F480                 shr     ebp, 1Bh  

.text:0042F483                 or      edi, ebp  

.text:0042F485                 add     esi, edi  

.text:0042F487                 mov     edi, [esp+14h+E]  

.text:0042F48B                 add     esi, 8F1BBCDCh  

.text:0042F491                 add     edx_C, esi  

.text:0042F493                 mov     esi, [esp+14h+E]  

.text:0042F497                 shl     esi, 1Eh  

.text:0042F49A                 shr     edi, 2  

.text:0042F49D                 or      esi, edi  

.text:0042F49F                 mov     [esp+14h+E], esi ; R3(D,E,A,B,C,47)  

.text:0042F49F                                             ; **********第 3.7 次运算 END****  

.text:0042F4A3                 mov     esi, [esp+14h+E] ; **********第 3.8 次运算开始****  

.text:0042F4A7                 mov     edi, [esp+14h+E]  

.text:0042F4AB                 or      esi, ecx_D  

.text:0042F4AD                 and     edi, ecx_D  

.text:0042F4AF                 and     esi, [esp+14h+A]  

.text:0042F4B3                 mov     ebp, [ebx+34h]  

.text:0042F4B6                 or      esi, edi  

.text:0042F4B8                 mov     edi, [ebx+34h]  

.text:0042F4BB                 xor     ebp, [ebx+20h]  

.text:0042F4BE                 xor     edi, [ebx+20h]  

.text:0042F4C1                 xor     ebp, [ebx+8]  

.text:0042F4C4                 xor     edi, [ebx+8]  

.text:0042F4C7                 xor     ebp, [ebx]  

.text:0042F4C9                 xor     edi, [ebx]  

.text:0042F4CB                 shr     ebp, 1Fh  

.text:0042F4CE                 add     edi, edi  

.text:0042F4D0                 or      edi, ebp  

.text:0042F4D2                 mov     ebp, edx_C  

.text:0042F4D4                 add     esi, edi  

.text:0042F4D6                 mov     [ebx], edi  

.text:0042F4D8                 mov     edi, edx_C  

.text:0042F4DA                 shr     ebp, 1Bh  

.text:0042F4DD                 shl     edi, 5  

.text:0042F4E0                 or      edi, ebp  

.text:0042F4E2                 add     esi, edi  

.text:0042F4E4                 add     esi, 8F1BBCDCh  

.text:0042F4EA                 add     eax_B, esi  

.text:0042F4EC                 mov     esi, ecx_D  

.text:0042F4EE                 shl     esi, 1Eh  

.text:0042F4F1                 shr     ecx_D, 2  

.text:0042F4F4                 or      esi, ecx_D  

.text:0042F4F6                 mov     ebp, [ebx+38h]  

.text:0042F4F9                 mov     ecx_D, esi       ; R3(C,D,E,A,B,48)  

.text:0042F4F9                                             ; **********第 3.8 次运算 END****  

.text:0042F4FB                 xor     ebp, [ebx+24h]  ; **********第 3.9 次运算开始****  

.text:0042F4FE                 mov     esi, ecx_D  

.text:0042F500                 mov     edi, ecx_D  

.text:0042F502                 or      esi, edx_C  

.text:0042F504                 and     edi, edx_C  

.text:0042F506                 and     esi, [esp+14h+E]  

.text:0042F50A                 xor     ebp, [ebx+0Ch]  

.text:0042F50D                 or      esi, edi  

.text:0042F50F                 mov     edi, [ebx+38h]  

.text:0042F512                 xor     edi, [ebx+24h]  

.text:0042F515                 xor     ebp, [ebx+4]  

.text:0042F518                 xor     edi, [ebx+0Ch]  

.text:0042F51B                 xor     edi, [ebx+4]  

.text:0042F51E                 shr     ebp, 1Fh  

.text:0042F521                 add     edi, edi  

.text:0042F523                 or      edi, ebp  

.text:0042F525                 mov     ebp, eax_B  

.text:0042F527                 mov     [ebx+4], edi  

.text:0042F52A                 add     esi, edi  

.text:0042F52C                 mov     edi, eax_B  

.text:0042F52E                 shl     edi, 5  

.text:0042F531                 shr     ebp, 1Bh  

.text:0042F534                 or      edi, ebp  

.text:0042F536                 add     esi, edi  

.text:0042F538                 add     esi, 8F1BBCDCh  

.text:0042F53E                 add     [esp+14h+A], esi  

.text:0042F542                 mov     esi, edx_C  

.text:0042F544                 shr     edx_C, 2  

.text:0042F547                 mov     ebp, [ebx+3Ch]  

.text:0042F54A                 shl     esi, 1Eh  

.text:0042F54D                 xor     ebp, [ebx+28h]  

.text:0042F550                 or      esi, edx_C  

.text:0042F552                 xor     ebp, [ebx+10h]  

.text:0042F555                 mov     edx_C, esi       ; R3(B,C,D,E,A,49)  

.text:0042F555                                             ; **********第 3.9 次运算 END****  

.text:0042F557                 xor     ebp, [ebx+8]     ; **********第 3.10 次运算开始****  

.text:0042F55A                 mov     esi, edx_C  

.text:0042F55C                 mov     edi, edx_C  

.text:0042F55E                 or      esi, eax_B  

.text:0042F560                 and     edi, eax_B  

.text:0042F562                 and     esi, ecx_D  

.text:0042F564                 or      esi, edi  

.text:0042F566                 mov     edi, [ebx+3Ch]  

.text:0042F569                 xor     edi, [ebx+28h]  

.text:0042F56C                 xor     edi, [ebx+10h]  

.text:0042F56F                 xor     edi, [ebx+8]  

.text:0042F572                 shr     ebp, 1Fh  

.text:0042F575                 add     edi, edi  

.text:0042F577                 or      edi, ebp  

.text:0042F579                 mov     [ebx+8], edi  

.text:0042F57C                 add     esi, edi  

.text:0042F57E                 mov     edi, [esp+14h+A]  

.text:0042F582                 mov     ebp, [esp+14h+A]  

.text:0042F586                 shr     ebp, 1Bh  

.text:0042F589                 shl     edi, 5  

.text:0042F58C                 or      edi, ebp  

.text:0042F58E                 add     esi, edi  

.text:0042F590                 add     esi, 8F1BBCDCh  

.text:0042F596                 add     [esp+14h+E], esi  

.text:0042F59A                 mov     esi, eax_B  

.text:0042F59C                 shr     eax_B, 2  

.text:0042F59F                 mov     edi, [esp+14h+A]  
```
