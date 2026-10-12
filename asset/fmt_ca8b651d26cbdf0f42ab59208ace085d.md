
```
0066582D   |.  03D7            add edx,edi

0066582F   |.  F7D5            not ebp

00665831   |.  0BEA            or ebp,edx

00665833   |.  33EF            xor ebp,edi

00665835   |.  036C24 54       add ebp,dword ptr ss:[esp+54]

00665839   |.  8D8C29 442229F4 lea ecx,dword ptr ds:[ecx+ebp+F4292244]

00665840   |.  8BEF            mov ebp,edi

00665842   |.  C1C1 06         rol ecx,6

00665845   |.  03CA            add ecx,edx

00665847   |.  F7D5            not ebp

00665849   |.  0BE9            or ebp,ecx

0066584B   |.  33EA            xor ebp,edx

0066584D   |.  036C24 10       add ebp,dword ptr ss:[esp+10]

00665851   |.  8DB42E 97FF2A43 lea esi,dword ptr ds:[esi+ebp+432AFF97]

00665858   |.  8BEA            mov ebp,edx

0066585A   |.  C1C6 0A         rol esi,0A

0066585D   |.  03F1            add esi,ecx

0066585F   |.  F7D5            not ebp

00665861   |.  0BEE            or ebp,esi

00665863   |.  33E9            xor ebp,ecx

00665865   |.  036C24 44       add ebp,dword ptr ss:[esp+44]

00665869   |.  8DBC2F A72394AB lea edi,dword ptr ds:[edi+ebp+AB9423A7]

00665870   |.  8BE9            mov ebp,ecx

00665872   |.  C1C7 0F         rol edi,0F

00665875   |.  03FE            add edi,esi

00665877   |.  F7D5            not ebp

00665879   |.  0BEF            or ebp,edi

0066587B   |.  33EE            xor ebp,esi

0066587D   |.  036C24 14       add ebp,dword ptr ss:[esp+14]

00665881   |.  8D942A 39A093FC lea edx,dword ptr ds:[edx+ebp+FC93A039]

00665888   |.  8BEE            mov ebp,esi

0066588A   |.  C1C2 15         rol edx,15

0066588D   |.  03D7            add edx,edi

0066588F   |.  F7D5            not ebp

00665891   |.  0BEA            or ebp,edx

00665893   |.  33EF            xor ebp,edi

00665895   |.  03E8            add ebp,eax

00665897   |.  8BC7            mov eax,edi

00665899   |.  F7D0            not eax

0066589B   |.  8D8C29 C3595B65 lea ecx,dword ptr ds:[ecx+ebp+655B59C3]

006658A2   |.  8B6C24 18       mov ebp,dword ptr ss:[esp+18]

006658A6   |.  C1C1 06         rol ecx,6

006658A9   |.  03CA            add ecx,edx

006658AB   |.  0BC1            or eax,ecx

006658AD   |.  33C2            xor eax,edx

006658AF   |.  03C5            add eax,ebp

006658B1   |.  8B6C24 1C       mov ebp,dword ptr ss:[esp+1C]

006658B5   |.  8DB430 92CC0C8F lea esi,dword ptr ds:[eax+esi+8F0CCC92]

006658BC   |.  8BC2            mov eax,edx

006658BE   |.  C1C6 0A         rol esi,0A

006658C1   |.  03F1            add esi,ecx

006658C3   |.  F7D0            not eax

006658C5   |.  0BC6            or eax,esi

006658C7   |.  33C1            xor eax,ecx

006658C9   |.  03C5            add eax,ebp

006658CB   |.  8DBC38 7DF4EFFF lea edi,dword ptr ds:[eax+edi+FFEFF47D]

006658D2   |.  8B6C24 20       mov ebp,dword ptr ss:[esp+20]

006658D6   |.  8BC1            mov eax,ecx

006658D8   |.  C1C7 0F         rol edi,0F

006658DB   |.  03FE            add edi,esi

006658DD   |.  F7D0            not eax

006658DF   |.  0BC7            or eax,edi

006658E1   |.  33C6            xor eax,esi

006658E3   |.  03C5            add eax,ebp

006658E5   |.  8B6C24 24       mov ebp,dword ptr ss:[esp+24]

006658E9   |.  8D9410 D15D8485 lea edx,dword ptr ds:[eax+edx+85845DD1]

006658F0   |.  8BC6            mov eax,esi

006658F2   |.  C1C2 15         rol edx,15

006658F5   |.  03D7            add edx,edi

006658F7   |.  F7D0            not eax

006658F9   |.  0BC2            or eax,edx

006658FB   |.  33C7            xor eax,edi

006658FD   |.  03C5            add eax,ebp

006658FF   |.  8B6C24 48       mov ebp,dword ptr ss:[esp+48]

00665903   |.  8D8C08 4F7EA86F lea ecx,dword ptr ds:[eax+ecx+6FA87E4F]

0066590A   |.  8BC7            mov eax,edi

0066590C   |.  C1C1 06         rol ecx,6

0066590F   |.  03CA            add ecx,edx

00665911   |.  F7D0            not eax

00665913   |.  0BC1            or eax,ecx

00665915   |.  33C2            xor eax,edx

00665917   |.  03C5            add eax,ebp

00665919   |.  8B6C24 28       mov ebp,dword ptr ss:[esp+28]

0066591D   |.  8D8430 E0E62CFE lea eax,dword ptr ds:[eax+esi+FE2CE6E0]

00665924   |.  8BF2            mov esi,edx

00665926   |.  C1C0 0A         rol eax,0A

00665929   |.  03C1            add eax,ecx

0066592B   |.  F7D6            not esi

0066592D   |.  0BF0            or esi,eax

0066592F   |.  33F1            xor esi,ecx

00665931   |.  03F5            add esi,ebp

00665933   |.  8B6C24 2C       mov ebp,dword ptr ss:[esp+2C]

00665937   |.  8DB43E 144301A3 lea esi,dword ptr ds:[esi+edi+A3014314]

0066593E   |.  8BF9            mov edi,ecx

00665940   |.  C1C6 0F         rol esi,0F

00665943   |.  03F0            add esi,eax

00665945   |.  F7D7            not edi

00665947   |.  0BFE            or edi,esi

00665949   |.  33F8            xor edi,eax

0066594B   |.  03FD            add edi,ebp

0066594D   |.  8B6C24 30       mov ebp,dword ptr ss:[esp+30]

00665951   |.  8D9417 A111084E lea edx,dword ptr ds:[edi+edx+4E0811A1]

00665958   |.  8BF8            mov edi,eax

0066595A   |.  C1C2 15         rol edx,15

0066595D   |.  03D6            add edx,esi

0066595F   |.  F7D7            not edi

00665961   |.  0BFA            or edi,edx

00665963   |.  33FE            xor edi,esi

00665965   |.  03FD            add edi,ebp

00665967   |.  8B6C24 34       mov ebp,dword ptr ss:[esp+34]

0066596B   |.  8D8C0F 827E53F7 lea ecx,dword ptr ds:[edi+ecx+F7537E82]

00665972   |.  8BFE            mov edi,esi

00665974   |.  C1C1 06         rol ecx,6

00665977   |.  03CA            add ecx,edx

00665979   |.  F7D7            not edi

0066597B   |.  0BF9            or edi,ecx

0066597D   |.  33FA            xor edi,edx

0066597F   |.  03FD            add edi,ebp

00665981   |.  8B6C24 38       mov ebp,dword ptr ss:[esp+38]

00665985   |.  8DBC07 35F23ABD lea edi,dword ptr ds:[edi+eax+BD3AF235]

0066598C   |.  8BC2            mov eax,edx

0066598E   |.  C1C7 0A         rol edi,0A

00665991   |.  03F9            add edi,ecx

00665993   |.  F7D0            not eax

00665995   |.  0BC7            or eax,edi

00665997   |.  33C1            xor eax,ecx

00665999   |.  03C3            add eax,ebx

0066599B   |.  8DB430 BBD2D72A lea esi,dword ptr ds:[eax+esi+2AD7D2BB]

006659A2   |.  8BC1            mov eax,ecx

006659A4   |.  C1C6 0F         rol esi,0F

006659A7   |.  03F7            add esi,edi

006659A9   |.  F7D0            not eax

006659AB   |.  0BC6            or eax,esi

006659AD   |.  33C7            xor eax,edi

006659AF   |.  03C5            add eax,ebp

006659B1   |.  8D9410 91D386EB lea edx,dword ptr ds:[eax+edx+EB86D391]

006659B8   |.  C1C2 15         rol edx,15

006659BB   |.  8B4424 50       mov eax,dword ptr ss:[esp+50]

006659BF   |.  03D6            add edx,esi

006659C1   |.  8B18            mov ebx,dword ptr ds:[eax]

006659C3   |.  03D9            add ebx,ecx

006659C5   |.  8B48 04         mov ecx,dword ptr ds:[eax+4]

006659C8   |.  03CA            add ecx,edx

006659CA   |.  8918            mov dword ptr ds:[eax],ebx

006659CC   |.  8948 04         mov dword ptr ds:[eax+4],ecx

006659CF   |.  8B48 08         mov ecx,dword ptr ds:[eax+8]

006659D2   |.  03CE            add ecx,esi

006659D4   |.  8948 08         mov dword ptr ds:[eax+8],ecx

006659D7   |.  8B48 0C         mov ecx,dword ptr ds:[eax+C]

006659DA   |.  03CF            add ecx,edi

006659DC   |.  5F              pop edi

006659DD   |.  5E              pop esi

006659DE   |.  5D              pop ebp

006659DF   |.  8948 0C         mov dword ptr ds:[eax+C],ecx

006659E2   |.  5B              pop ebx

006659E3   |.  83C4 3C         add esp,3C
```
