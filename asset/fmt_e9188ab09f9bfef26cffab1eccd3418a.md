
```
0055367A   |.   80F9 61               |cmp cl,61  
0055367D   |.   72 0D                |jb short Camtasia.0055368C  
0055367F   |.   80F9 7A              |cmp cl,7A  
00553682   |.   77 08                |ja short Camtasia.0055368C  
00553684   |.   0FB6C9               |movzx ecx,cl  
00553687   |.   83E9 20              |sub ecx,20  
0055368A   |.   EB 03                |jmp short Camtasia.0055368F  
0055368C   |>   0FB6C9               |movzx ecx,cl  
0055368F   |>   66:0FB6C9            |movzx cx,cl  
00553693   |.   66:83F9 30           |cmp cx,30  
00553697   |.   72 0E                |jb short Camtasia.005536A7  
00553699   |.   66:83F9 39           |cmp cx,39  
0055369D   |.   77 08                |ja short Camtasia.005536A7  
0055369F   |.   81C1 D0FF0000        |add ecx,0FFD0  
005536A5   |.   EB 12                |jmp short Camtasia.005536B9  
005536A7   |>   66:83F9 41           |cmp cx,41  
005536AB   |.   72 1D                |jb short Camtasia.005536CA  
005536AD   |.   66:83F9 46           |cmp cx,46  
005536B1   |.   77 17                |ja short Camtasia.005536CA  
005536B3   |.   81C1 C9FF0000        |add ecx,0FFC9  
005536B9   |>   C1E0 04              |shl eax,4  
005536BC   |.   0FB7C9               |movzx ecx,cx  
005536BF   |.   0BC1                 |or eax,ecx  
005536C1   |.   42                   |inc edx  
005536C2   |.   66:3BD7              |cmp dx,di  
005536C5   |.^ 72 AD                 \jb short Camtasia.00553674  
005536C7   |.   5F                   |pop edi  
005536C8   |.   5E                   |pop esi  
005536C9   |.   C3                   |retn  
005536CA   |>   83C8 FF              |or eax,FFFFFFFF  
005536CD   |>   5F                   |pop edi  
005536CE   |.   5E                   |pop esi  
005536CF   |\   C3                   |retn  
005536D0   /$   83EC 7C              |sub esp,7C  
005536D3   |.   33C0                 |xor eax,eax  
005536D5   |.   B9 30000000          |mov ecx,30  
005536DA   |.   8D9B 00000000        |lea ebx,dword ptr ds:[ebx]  
005536E0   |>   0FB7D0               /movzx edx,ax  
005536E3   |.   40                   |inc eax  
005536E4   |.   884C14 04            |mov byte ptr ss:[esp+edx+4],cl  
005536E8   |.   41                   |inc ecx  
005536E9   |.   66:83F9 39           |cmp cx,39  
005536ED   |.^ 76 F1                 \jbe short Camtasia.005536E0  
005536EF   |.   B9 41000000          |mov ecx,41  
005536F4   |>   0FB7D0               /movzx edx,ax  
005536F7   |.   40                   |inc eax  
005536F8   |.   884C14 04            |mov byte ptr ss:[esp+edx+4],cl  
005536FC   |.   41                   |inc ecx  
005536FD   |.   66:83F9 46           |cmp cx,46  
00553701   |.^ 76 F1                 \jbe short Camtasia.005536F4  
00553703   |.   55                   |push ebp  
00553704   |.   8BAC24 88000000      |mov ebp,dword ptr ss:[esp+88]  
0055370B   |.   56                   |push esi  
0055370C   |.   8D45 0C              |lea eax,dword ptr ss:[ebp+C]  
0055370F   |.   6A 02                |push 2  
00553711   |.   50                   |push eax  
00553712   |.   E8 49FFFFFF          |call Camtasia.00553660  
00553717   |.   8BB424 90000000      |mov esi,dword ptr ss:[esp+90]  
0055371E   |.   894424 10            |mov dword ptr ss:[esp+10],eax  
00553722   |.   0FB706               |movzx eax,word ptr ds:[esi]  
00553725   |.   8D48 02              |lea ecx,dword ptr ds:[eax+2]  
00553728   |.   83C4 08              |add esp,8  
0055372B   |.   83F9 32              |cmp ecx,32  
0055372E   |.   7F 0E                |jg short Camtasia.0055373E  
00553730   |.   66:8B5424 08         |mov dx,word ptr ss:[esp+8]  
00553735   |.   66:895430 02         |mov word ptr ds:[eax+esi+2],dx  
0055373A   |.   66:8306 02           |add word ptr ds:[esi],2  
0055373E   |>   8D45 08              |lea eax,dword ptr ss:[ebp+8]  
00553741   |.   6A 04                |push 4  
00553743   |.   50                   |push eax  
00553744   |.   E8 17FFFFFF          |call Camtasia.00553660  
00553749   |.   894424 10            |mov dword ptr ss:[esp+10],eax  
0055374D   |.   0FB706               |movzx eax,word ptr ds:[esi]  
00553750   |.   8D48 02              |lea ecx,dword ptr ds:[eax+2]  
00553753   |.   83C4 08              |add esp,8  
00553756   |.   83F9 32              |cmp ecx,32  
00553759   |.   7F 0E                |jg short Camtasia.00553769  
0055375B   |.   66:8B5424 08         |mov dx,word ptr ss:[esp+8]  
00553760   |.   66:895430 02         |mov word ptr ds:[eax+esi+2],dx  
00553765   |.   66:8306 02           |add word ptr ds:[esi],2  
00553769   |>   8D4424 2C            |lea eax,dword ptr ss:[esp+2C]  
0055376D   |.   57                   |push edi  
0055376E   |.   50                   |push eax  
0055376F   |.   E8 8C000000          |call Camtasia.00553800  
00553774   |.   0FB70E               |movzx ecx,word ptr ds:[esi]  
00553777   |.   51                   |push ecx  
00553778   |.   8D7E 02              |lea edi,dword ptr ds:[esi+2]  
0055377B   |.   8D5424 38            |lea edx,dword ptr ss:[esp+38]  
0055377F   |.   57                   |push edi  
00553780   |.   52                   |push edx  
00553781   |.   E8 8A090000          |call Camtasia.00554110  
00553786   |.   8D4424 40            |lea eax,dword ptr ss:[esp+40]  
0055378A   |.   50                   |push eax  
0055378B   |.   8D4C24 34            |lea ecx,dword ptr ss:[esp+34]  
0055378F   |.   51                   |push ecx  
00553790   |.   E8 3B0A0000          |call Camtasia.005541D0  
00553795   |.   33C0                 |xor eax,eax  
00553797   |.   B9 0C000000          |mov ecx,0C  
0055379C   |.   F3:AB                |rep stos dword ptr es:[edi]  
0055379E   |.   83C4 18              |add esp,18  
005537A1   |.   66:AB                |stos word ptr es:[edi]  
005537A3   |.   33D2                 |xor edx,edx  
005537A5   |.   5F                   |pop edi  
005537A6   |>   0FB7CA               /movzx ecx,dx  
005537A9   |.   8BC1                 |mov eax,ecx  
005537AB   |.   D1E8                 |shr eax,1  
005537AD   |.   8A0428               |mov al,byte ptr ds:[eax+ebp]  
005537B0   |.   3C 61                |cmp al,61  
005537B2   |.   72 0C                |jb short Camtasia.005537C0  
005537B4   |.   3C 7A                |cmp al,7A  
005537B6   |.   77 08                |ja short Camtasia.005537C0  
005537B8   |.   0FB6C0               |movzx eax,al  
005537BB   |.   83E8 20              |sub eax,20  
005537BE   |.   EB 03                |jmp short Camtasia.005537C3  
005537C0   |>   0FB6C0               |movzx eax,al  
005537C3   |>   0FB64C0C 1C          |movzx ecx,byte ptr ss:[esp+ecx+1C]  
005537C8   |.   83E1 0F              |and ecx,0F  
005537CB   |.   0FBE4C0C 0C          |movsx ecx,byte ptr ss:[esp+ecx+C]  
005537D0   |.   0FB6C0               |movzx eax,al  
005537D3   |.   3BC8                 |cmp ecx,eax --->在这里可以读出前 8 位注册码了 5BFD9EA0  
005537D5   |.   75 14                |jnz short Camtasia.005537EB  
005537D7   |.   83C2 02              |add edx,2  
005537DA   |.   66:83FA 10           |cmp dx,10  
005537DE   |.^ 72 C6                 \jb short Camtasia.005537A6  
005537E0   |.   5E                   |pop esi  
005537E1   |.   B8 01000000          |mov eax,1  
005537E6   |.   5D                   |pop ebp  
005537E7   |.   83C4 7C              |add esp,7C  
005537EA   |.   C3                   |retn  
005537EB   |>   5E                   |pop esi  
005537EC   |.   33C0                 |xor eax,eax  
005537EE   |.   5D                   |pop ebp  
005537EF   |.   83C4 7C              |add esp,7C
```

`-->完。。。。`

可用的注册码：

用户名：`ftts[BCG]`

注册码：`5BFD9EA09012615678`

--------------------------------------------------------------------------------

【版权声明】 本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!

`4dd466d0e1e0cc123afbe504da2462ca`
