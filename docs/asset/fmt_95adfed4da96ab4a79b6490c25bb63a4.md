
### 3. 一个比较关键的 call：`Registra.00403410`

功能：验证注册码的前四个字符是不是"RWMR"。

```text
00403410   /$  55                   push ebp
00403411   |.  8BEC                 mov ebp,esp
00403413   |.  57                   push edi
00403414   |.  56                   push esi
00403415   |.  53                   push ebx
00403416   |.  8B4D 10             mov ecx,dword ptr ss:[ebp+10]
00403419   |.  0BC9                or ecx,ecx
0040341B   |.  0F84 E9000000      je Registra.0040350A
00403421   |.  8B75 08             mov esi,dword ptr ss:[ebp+8]            ;  ESI 0012F6CC ASCII "RWMR1A-517"
00403424   |.  8B7D 0C             mov edi,dword ptr ss:[ebp+C]            ;  EDI 0041E158 ASCII "RWMR"
00403427   |.  8D05 005A4200      lea eax,dword ptr ds:[425A00]             ;  eax=00425A00
0040342D   |.  8378 08 00         cmp dword ptr ds:[eax+8],0              ; [eax+8]=0
00403431   |.  75 4E              jnz short Registra.00403481
00403433   |.  B7 41              mov bh,41                                 ;  asc(41)=A
00403435   |.  B3 5A              mov bl,5A                                 ;  asc(5A)=Z
00403437   |.  B6 20              mov dh,20                                 ;  asc(20)=Space
00403439   |.  2E:8BC0            mov eax,eax
0040343C   |>  8A26               /mov ah,byte ptr ds:[esi]                 ; 假码
0040343E   |.  0AE4              |or ah,ah
00403440   |.  8A07              |mov al,byte ptr ds:[edi]                 ; 常数字符串"RWMR"
00403442   |.  74 21             |je short Registra.00403465
00403444   |.  0AC0              |or al,al
00403446   |.  74 1D             |je short Registra.00403465
00403448   |.  46                |inc esi
00403449   |.  47                |inc edi
0040344A   |.  38FC              |cmp ah,bh                                 ; 与"A"比较
0040344C   |.  72 06             |jb short Registra.00403454
0040344E   |.  38DC              |cmp ah,bl                                 ; 与"Z"比较
00403450   |.  77 02             |ja short Registra.00403454
00403452   |.  02E6              |add ah,dh                                 ; 变为小写
00403454   |>  38F8              |cmp al,bh                                 ; 与"A"比较
00403456   |.  72 06             |jb short Registra.0040345E
00403458   |.  38D8              |cmp al,bl                                 ; 与"Z"比较
0040345A   |.  77 02             |ja short Registra.0040345E
0040345C   |.  02C6              |add al,dh                                 ; 变成小写
0040345E   |>  38C4              |cmp ah,al                                 ; 假码与常数串相比较
00403460   |.  75 0D             |jnz short Registra.0040346F
00403462   |.  49                |dec ecx                                   ; 计数器-1，初值 4，即常数串的长度
00403463   |.^ 75 D7             \jnz short Registra.0040343C
00403465   |>  33C9              xor ecx,ecx                                 ; 置 0，下面有置-1 的语句，这是一个 BOOL 参数
00403467   |.  38C4              cmp ah,al
00403469   |.  0F84 9B000000    je Registra.0040350A                          ; 跳
0040346F   |>  B9 FFFFFFFF       mov ecx,-1
00403474   |.  0F82 90000000    jb Registra.0040350A
0040347A   |.  F7D9              neg ecx
0040347C   |.  E9 89000000      jmp Registra.0040350A
00403481   |>  F0:FF05 E86D4200 lock inc dword ptr ds:[426DE8]
00403488   |.  833D E46D4200 00  cmp dword ptr ds:[426DE4],0
0040348F   |.  7F 04              jg short Registra.00403495
00403491   |.  6A 00              push 0
00403493   |.  EB 19              jmp short Registra.004034AE
00403495   |>  F0:FF0D E86D4200 lock dec dword ptr ds:[426DE8]
0040349C   |.  8BD9              mov ebx,ecx
0040349E   |.  6A 13              push 13
004034A0   |.  E8 2B360000       call Registra.00406AD0
004034A5   |.  C70424 01000000   mov dword ptr ss:[esp],1
004034AC   |.  8BCB              mov ecx,ebx
004034AE   |>  33C0              xor eax,eax
004034B0   |.  33DB              xor ebx,ebx
004034B2   |.  8BC0              mov eax,eax
004034B4   |>  8A06              /mov al,byte ptr ds:[esi]
004034B6   |.  0BC0              |or eax,eax
004034B8   |.  8A1F              |mov bl,byte ptr ds:[edi]
004034BA   |.  74 23             |je short Registra.004034DF
004034BC   |.  0BDB              |or ebx,ebx
004034BE   |.  74 1F             |je short Registra.004034DF
004034C0   |.  46                |inc esi
004034C1   |.  47                |inc edi
004034C2   |.  51                |push ecx
004034C3   |.  50                |push eax
004034C4   |.  53                |push ebx
004034C5   |.  E8 96370000       |call Registra.00406C60
004034CA   |.  8BD8              |mov ebx,eax
004034CC   |.  83C4 04           |add esp,4
004034CF   |.  E8 8C370000       |call Registra.00406C60
004034D4   |.  83C4 04           |add esp,4
004034D7   |.  59                |pop ecx
004034D8   |.  3BC3              |cmp eax,ebx
004034DA   |.  75 09             |jnz short Registra.004034E5
004034DC   |.  49                |dec ecx
004034DD   |.^ 75 D5             \jnz short Registra.004034B4
004034DF   |>  33C9              xor ecx,ecx
004034E1   |.  3BC3              cmp eax,ebx
004034E3   |.  74 09             je short Registra.004034EE
004034E5   |>  B9 FFFFFFFF       mov ecx,-1
004034EA   |.  72 02             jb short Registra.004034EE
004034EC   |.  F7D9              neg ecx
004034EE   |>  58                pop eax
004034EF   |.  0BC0              or eax,eax
004034F1   |.  75 09             jnz short Registra.004034FC
004034F3   |.  F0:FF0D E86D4200 lock dec dword ptr ds:[426DE8]
004034FA   |.  EB 0E             jmp short Registra.0040350A
004034FC   |>  8BD9              mov ebx,ecx
004034FE   |.  6A 13              push 13                                   ; /Arg1 = 00000013
00403500   |.  E8 4B360000       call Registra.00406B50                      ; \\Registra.00406B50
00403505   |.  83C4 04           add esp,4
00403508   |.  8BCB              mov ecx,ebx
0040350A   |>  8BC1              mov eax,ecx                                 ; Bool 参数到 eax，程序返回后根据 eax 决定流程
0040350C   |.  5B                pop ebx
0040350D   |.  5E                pop esi
0040350E   |.  5F                pop edi
0040350F   |.  C9                leave
00403510   \\.  C3                retn
```

### 4. 又一个比较关键的 call：`Registra.00403350`

功能：

返回值：`eax`，应该不为 0。`eax` 指向`"-"`。
