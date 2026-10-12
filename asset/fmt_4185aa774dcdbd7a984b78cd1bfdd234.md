
```
01002ED5  /$  B8 60030000    mov eax,360
01002EDA  |>  48             /dec eax
01002EDB  |.  C680 40530001>|mov byte ptr ds:\[eax+1005340\],0F
01002EE2  |.^ 75 F6          \\jnz short winmine.01002EDA
01002EE4  |.  8B0D 34530001  mov ecx,dword ptr ds:\[1005334\]
01002EEA  |.  8B15 38530001  mov edx,dword ptr ds:\[1005338\]
01002EF0  |.  8D41 02        lea eax,dword ptr ds:\[ecx+2\]
01002EF3  |.  85C0           test eax,eax
01002EF5  |.  56             push esi
01002EF6  |.  74 19          je short winmine.01002F11
01002EF8  |.  8BF2           mov esi,edx
01002EFA  |.  C1E6 05        shl esi,5
01002EFD  |.  8DB6 60530001  lea esi,dword ptr ds:\[esi+1005360\]
01002F03  |>  48             /dec eax
01002F04  |.  C680 40530001>|mov byte ptr ds:\[eax+1005340\],10
01002F0B  |.  C60406 10      |mov byte ptr ds:\[esi+eax\],10
01002F0F  |.^ 75 F2          \\jnz short winmine.01002F03
01002F11  |>  8D72 02        lea esi,dword ptr ds:\[edx+2\]
01002F14  |.  85F6           test esi,esi
01002F16  |.  74 21          je short winmine.01002F39
01002F18  |.  8BC6           mov eax,esi
01002F1A  |.  C1E0 05        shl eax,5
01002F1D  |.  8D90 40530001  lea edx,dword ptr ds:\[eax+1005340\]
01002F23  |.  8D8408 415300>lea eax,dword ptr ds:\[eax+ecx+1005341\]
01002F2A  |>  83EA 20        /sub edx,20
01002F2D  |.  83E8 20        |sub eax,20
01002F30  |.  4E             |dec esi
01002F31  |.  C602 10        |mov byte ptr ds:\[edx\],10
01002F34  |.  C600 10        |mov byte ptr ds:\[eax\],10
01002F37  |.^ 75 F1          \\jnz short winmine.01002F2A
01002F39  |>  5E             pop esi
01002F3A  \\.  C3            retn
```

删除内存断点（记得一定要删哦，亲 ），`ctrl+f9`：  

代码:

```
0100367A  /$  A1 AC560001    mov eax,dword ptr ds:\[10056AC\]
0100367F  |.  8B0D A8560001  mov ecx,dword ptr ds:\[10056A8\]
01003685  |.  53             push ebx
01003686  |.  56             push esi
01003687  |.  57             push edi
01003688  |.  33FF           xor edi,edi
0100368A  |.  3B05 34530001  cmp eax,dword ptr ds:\[1005334\]
01003690  |.  893D 64510001  mov dword ptr ds:\[1005164\],edi
01003696  |.  75 0C          jnz short winmine.010036A4
01003698  |.  3B0D 38530001  cmp ecx,dword ptr ds:\[1005338\]
0100369E  |.  75 04          jnz short winmine.010036A4
010036A0  |.  6A 04          push 4
010036A2  |.  EB 02          jmp short winmine.010036A6
010036A4  |>  6A 06          push 6
010036A6  |>  5B             pop ebx
010036A7  |.  A3 34530001    mov dword ptr ds:\[1005334\],eax
010036AC  |.  890D 38530001  mov dword ptr ds:\[1005338\],ecx
010036B2  |.  E8 1EF8FFFF    call winmine.01002ED5                     ;  init game
010036B7  |.  A1 A4560001    mov eax,dword ptr ds:\[10056A4\]
010036BC  |.  893D 60510001  mov dword ptr ds:\[1005160\],edi
010036C2  |.  A3 30530001    mov dword ptr ds:\[1005330\],eax
010036C7  |>  FF35 34530001  push dword ptr ds:\[1005334\]
010036CD  |.  E8 6E020000    call <winmine.rand>                       ;  rand
010036D2  |.  FF35 38530001  push dword ptr ds:\[1005338\]
010036D8  |.  8BF0           mov esi,eax
010036DA  |.  46             inc esi
010036DB  |.  E8 60020000    call <winmine.rand>
010036E0  |.  40             inc eax
010036E1  |.  8BC8           mov ecx,eax
010036E3  |.  C1E1 05        shl ecx,5
010036E6  |.  F68431 405300>test byte ptr ds:\[ecx+esi+1005340\],80
010036EE  |.^ 75 D7          jnz short winmine.010036C7
010036F0  |.  C1E0 05        shl eax,5
010036F3  |.  8D8430 405300>lea eax,dword ptr ds:\[eax+esi+1005340\]
010036FA  |.  8008 80        or byte ptr ds:\[eax\],80
010036FD  |.  FF0D 30530001  dec dword ptr ds:\[1005330\]
01003703  |.^ 75 C2          jnz short winmine.010036C7
01003705  |.  8B0D 38530001  mov ecx,dword ptr ds:\[1005338\]
0100370B  |.  0FAF0D 345300>imul ecx,dword ptr ds:\[1005334\]
01003712  |.  A1 A4560001    mov eax,dword ptr ds:\[10056A4\]
01003717  |.  2BC8           sub ecx,eax
01003719  |.  57             push edi
0100371A  |.  893D 9C570001  mov dword ptr ds:\[100579C\],edi
01003720  |.  A3 30530001    mov dword ptr ds:\[1005330\],eax
01003725  |.  A3 94510001    mov dword ptr ds:\[1005194\],eax
0100372A  |.  893D A4570001  mov dword ptr ds:\[10057A4\],edi
01003730  |.  890D A0570001  mov dword ptr ds:\[10057A0\],ecx
01003736  |.  C705 00500001>mov dword ptr ds:\[1005000\],1
01003740  |.  E8 25FDFFFF    call winmine.0100346A
01003745  |.  53             push ebx
01003746  |.  E8 05E2FFFF    call winmine.01001950
0100374B  |.  5F             pop edi
0100374C  |.  5E             pop esi
0100374D  |.  5B             pop ebx
0100374E  \\.  C3            retn
```

至于你信不信上面是初始化代码片段，反正我是信了![](img/f3773294bf3a914c9a8c0c242528a003.jpg "使眼色") 。剩下的大家去参考邓韬牛牛的帖子吧。
