
```
0055D7B4   |>  8BF9         |/mov edi,ecx

0055D7B6   |.  C1E7 0B      ||shl edi,0B

0055D7B9   |.  33D7         ||xor edx,edi

0055D7BB   |.  0155 F4      ||add [local.3],edx

0055D7BE   |.  03CB         ||add ecx,ebx

0055D7C0   |.  8BFB         ||mov edi,ebx

0055D7C2   |.  C1EF 02      ||shr edi,2

0055D7C5   |.  33CF         ||xor ecx,edi

0055D7C7   |.  014D F0      ||add [local.4],ecx

0055D7CA   |.  035D F4      ||add ebx,[local.3]

0055D7CD   |.  8B7D F4      ||mov edi,[local.3]

0055D7D0   |.  C1E7 08      ||shl edi,8

0055D7D3   |.  33DF         ||xor ebx,edi

0055D7D5   |.  015D EC      ||add [local.5],ebx

0055D7D8   |.  8B7D F0      ||mov edi,[local.4]

0055D7DB   |.  017D F4      ||add [local.3],edi

0055D7DE   |.  8B7D F0      ||mov edi,[local.4]

0055D7E1   |.  C1EF 10      ||shr edi,10

0055D7E4   |.  317D F4      ||xor [local.3],edi

0055D7E7   |.  8B7D F4      ||mov edi,[local.3]

0055D7EA   |.  017D E8      ||add [local.6],edi

0055D7ED   |.  8B7D EC      ||mov edi,[local.5]

0055D7F0   |.  017D F0      ||add [local.4],edi

0055D7F3   |.  8B7D EC      ||mov edi,[local.5]

0055D7F6   |.  C1E7 0A      ||shl edi,0A

0055D7F9   |.  317D F0      ||xor [local.4],edi

0055D7FC   |.  8B7D F0      ||mov edi,[local.4]

0055D7FF   |.  017D E4      ||add [local.7],edi

0055D802   |.  8B7D E8      ||mov edi,[local.6]

0055D805   |.  017D EC      ||add [local.5],edi

0055D808   |.  8B7D E8      ||mov edi,[local.6]

0055D80B   |.  C1EF 04      ||shr edi,4

0055D80E   |.  317D EC      ||xor [local.5],edi

0055D811   |.  0355 EC      ||add edx,[local.5]

0055D814   |.  8B7D E4      ||mov edi,[local.7]

0055D817   |.  017D E8      ||add [local.6],edi

0055D81A   |.  8B7D E4      ||mov edi,[local.7]

0055D81D   |.  C1E7 08      ||shl edi,8

0055D820   |.  317D E8      ||xor [local.6],edi

0055D823   |.  034D E8      ||add ecx,[local.6]

0055D826   |.  0155 E4      ||add [local.7],edx

0055D829   |.  8BFA         ||mov edi,edx

0055D82B   |.  C1EF 09      ||shr edi,9

0055D82E   |.  317D E4      ||xor [local.7],edi

0055D831   |.  035D E4      ||add ebx,[local.7]

0055D834   |.  03D1         ||add edx,ecx

0055D836   |.  4E           ||dec esi

0055D837   |.^ 0F85 77FFFFF>|\jnz xx.0055D7B4

0055D83D   |.  8910         |mov dword ptr ds:[eax],edx            ;  存储变换后结果

0055D83F   |.  8948 04      |mov dword ptr ds:[eax+4],ecx

0055D842   |.  8958 08      |mov dword ptr ds:[eax+8],ebx

0055D845   |.  8B55 F4      |mov edx,[local.3]

0055D848   |.  8950 0C      |mov dword ptr ds:[eax+C],edx

0055D84B   |.  8B55 F0      |mov edx,[local.4]

0055D84E   |.  8950 10      |mov dword ptr ds:[eax+10],edx

0055D851   |.  8B55 EC      |mov edx,[local.5]

0055D854   |.  8950 14      |mov dword ptr ds:[eax+14],edx

0055D857   |.  8B55 E8      |mov edx,[local.6]

0055D85A   |.  8950 18      |mov dword ptr ds:[eax+18],edx

0055D85D   |.  8B55 E4      |mov edx,[local.7]

0055D860   |.  8950 1C      |mov dword ptr ds:[eax+1C],edx

0055D863   |.  83C0 20      |add eax,20

0055D866   |.  8345 DC 04   |add [local.9],4

0055D86A   |.  FF4D E0      |dec [local.8]

0055D86D   |.^ 0F85 0FFFFFF>\jnz xx.0055D782

0055D873   |.  807D 08 00   cmp byte ptr ss:[ebp+8],0

0055D877   |.  75 6A        jnz short xx.0055D8E3

0055D879   |.  8B45 FC      mov eax,[local.1]

0055D87C   |.  8B40 04      mov eax,dword ptr ds:[eax+4]

0055D87F   |.  48           dec eax

0055D880   |.  85C0         test eax,eax

0055D882   |.  7C 34        jl short xx.0055D8B8

0055D884   |.  40           inc eax

0055D885   |.  C745 E0 0000>mov [local.8],0

0055D88C   |.  8B55 FC      mov edx,[local.1]

0055D88F   |.  83C2 08      add edx,8

0055D892   |>  8B4D FC      /mov ecx,[local.1]

0055D895   |.  8B49 04      |mov ecx,dword ptr ds:[ecx+4]

0055D898   |.  49           |dec ecx

0055D899   |.  2B4D E0      |sub ecx,[local.8]

0055D89C   |.  8B1A         |mov ebx,dword ptr ds:[edx]

0055D89E   |.  899CCD 5CFFF>|mov dword ptr ss:[ebp+ecx*8-A4],ebx

0055D8A5   |.  8B5A 04      |mov ebx,dword ptr ds:[edx+4]

0055D8A8   |.  899CCD 60FFF>|mov dword ptr ss:[ebp+ecx*8-A0],ebx

0055D8AF   |.  FF45 E0      |inc [local.8]

0055D8B2   |.  83C2 08      |add edx,8

0055D8B5   |.  48           |dec eax

0055D8B6   |.^ 75 DA        \jnz short xx.0055D892

0055D8B8   |>  8B45 FC      mov eax,[local.1]

0055D8BB   |.  8B40 04      mov eax,dword ptr ds:[eax+4]

0055D8BE   |.  48           dec eax

0055D8BF   |.  85C0         test eax,eax

0055D8C1   |.  7C 20        jl short xx.0055D8E3

0055D8C3   |.  40           inc eax

0055D8C4   |.  8D95 5CFFFFF>lea edx,[local.41]

0055D8CA   |.  8B5D FC      mov ebx,[local.1]

0055D8CD   |.  83C3 08      add ebx,8

0055D8D0   |>  8B0A         /mov ecx,dword ptr ds:[edx]

0055D8D2   |.  890B         |mov dword ptr ds:[ebx],ecx

0055D8D4   |.  8B4A 04      |mov ecx,dword ptr ds:[edx+4]

0055D8D7   |.  894B 04      |mov dword ptr ds:[ebx+4],ecx

0055D8DA   |.  83C3 08      |add ebx,8

0055D8DD   |.  83C2 08      |add edx,8

0055D8E0   |.  48           |dec eax

0055D8E1   |.^ 75 ED        \jnz short xx.0055D8D0

0055D8E3   |>  5F           pop edi

0055D8E4   |.  5E           pop esi

0055D8E5   |.  5B           pop ebx

0055D8E6   |.  8BE5         mov esp,ebp

0055D8E8   |.  5D           pop ebp

0055D8E9   \.  C2 0400      retn 4
```

### 跟进 `0055F318` `call xx.Lbcipher::EncryptLBC`

```
0055D36C x>/$  53           push ebx

0055D36D   |.  56           push esi

0055D36E   |.  57           push edi

0055D36F   |.  83C4 D0      add esp,-30

0055D372   |.  891424       mov dword ptr ss:[esp],edx

0055D375   |.  8BD8         mov ebx,eax

0055D377   |.  8D5424 20    lea edx,dword ptr ss:[esp+20]

0055D37B   |.  8B0424       mov eax,dword ptr ss:[esp]

0055D37E   |.  B9 10000000  mov ecx,10

0055D383   |.  E8 F0C60700  call xx.005D9A78                       ;  将[12f790]处的16个数据转存到[12f658]处

0055D388   |.  8B4424 20    mov eax,dword ptr ss:[esp+20]

0055D38C   |.  894424 0C    mov dword ptr ss:[esp+C],eax           ;  把16位值存储到[12f644]处

0055D390   |.  8B4424 24    mov eax,dword ptr ss:[esp+24]

0055D394   |.  894424 10    mov dword ptr ss:[esp+10],eax

0055D398   |.  8B4424 28    mov eax,dword ptr ss:[esp+28]

0055D39C   |.  894424 14    mov dword ptr ss:[esp+14],eax

0055D3A0   |.  8B4424 2C    mov eax,dword ptr ss:[esp+2C]

0055D3A4   |.  894424 18    mov dword ptr ss:[esp+18],eax

0055D3A8   |.  8B43 04      mov eax,dword ptr ds:[ebx+4]

0055D3AB   |.  48           dec eax

0055D3AC   |.  85C0         test eax,eax

0055D3AE   |.  0F8C 1401000>jl xx.0055D4C8

0055D3B4   |.  40           inc eax

0055D3B5   |.  894424 1C    mov dword ptr ss:[esp+1C],eax          ;  [12f654]=eax计数器

0055D3B9   |.  8D43 08      lea eax,dword ptr ds:[ebx+8]

0055D3BC   |.  8BD8         mov ebx,eax

0055D3BE   |>  8B5424 0C    /mov edx,dword ptr ss:[esp+C]

0055D3C2   |.  8B0B         |mov ecx,dword ptr ds:[ebx]            ;  取第3个变量

0055D3C4   |.  8B7424 10    |mov esi,dword ptr ss:[esp+10]

0055D3C8   |.  8B43 04      |mov eax,dword ptr ds:[ebx+4]          ;  取第4个变量

0055D3CB   |.  03D0         |add edx,eax                           ;  继续变换

0055D3CD   |.  03C2         |add eax,edx

0055D3CF   |.  8BFA         |mov edi,edx

0055D3D1   |.  C1EF 07      |shr edi,7

0055D3D4   |.  33D7         |xor edx,edi

0055D3D6   |.  03CA         |add ecx,edx

0055D3D8   |.  03D1         |add edx,ecx

0055D3DA   |.  8BF9         |mov edi,ecx

0055D3DC   |.  C1E7 0D      |shl edi,0D

0055D3DF   |.  33CF         |xor ecx,edi

0055D3E1   |.  03F1         |add esi,ecx

0055D3E3   |.  03CE         |add ecx,esi

0055D3E5   |.  8BFE         |mov edi,esi

0055D3E7   |.  C1EF 11      |shr edi,11

0055D3EA   |.  33F7         |xor esi,edi

0055D3EC   |.  03C6         |add eax,esi

0055D3EE   |.  03F0         |add esi,eax

0055D3F0   |.  8BF8         |mov edi,eax

0055D3F2   |.  C1E7 09      |shl edi,9

0055D3F5   |.  33C7         |xor eax,edi

0055D3F7   |.  03D0         |add edx,eax
```
