
```
004B329C  |.  E8 B7C5FAFF    call tic_exe.0045F858

004B32A1  |.  837D C4 00     cmp dword ptr ss:[ebp-3C],0

004B32A5  |.  0F84 FA000000 je tic_exe.004B33A5

004B32AB  |.  8D55 C0        lea edx,dword ptr ss:[ebp-40]

004B32AE  |.  8B83 38030000 mov eax,dword ptr ds:[ebx+338]

004B32B4  |.  E8 9FC5FAFF    call tic_exe.0045F858

004B32B9  |.  8B45 C0        mov eax,dword ptr ss:[ebp-40]

004B32BC  |.  E8 1718F5FF    call tic_exe.00404AD8

004B32C1  |.  48             dec eax

004B32C2  |.  0F8E DD000000 jle tic_exe.004B33A5

004B32C8  |.  8D55 BC        lea edx,dword ptr ss:[ebp-44]

004B32CB  |.  8B83 24030000 mov eax,dword ptr ds:[ebx+324]

004B32D1  |.  E8 82C5FAFF    call tic_exe.0045F858

004B32D6  |.  837D BC 00     cmp dword ptr ss:[ebp-44],0

004B32DA  |.  0F84 C5000000 je tic_exe.004B33A5

004B32E0  |.  8D55 B8        lea edx,dword ptr ss:[ebp-48]

004B32E3  |.  8B83 28030000 mov eax,dword ptr ds:[ebx+328]

004B32E9  |.  E8 6AC5FAFF    call tic_exe.0045F858

004B32EE  |.  837D B8 00     cmp dword ptr ss:[ebp-48],0

004B32F2  |.  0F84 AD000000 je tic_exe.004B33A5

004B32F8  |.  8D55 B4        lea edx,dword ptr ss:[ebp-4C]

004B32FB  |.  8B83 2C030000 mov eax,dword ptr ds:[ebx+32C]

004B3301  |.  E8 52C5FAFF    call tic_exe.0045F858

004B3306  |.  837D B4 00     cmp dword ptr ss:[ebp-4C],0

004B330A  |.  0F84 95000000 je tic_exe.004B33A5

004B3310  |.  8D55 B0        lea edx,dword ptr ss:[ebp-50]

004B3313  |.  8B83 20030000 mov eax,dword ptr ds:[ebx+320]

004B3319  |.  E8 3AC5FAFF    call tic_exe.0045F858

004B331E  |.  8B45 B0        mov eax,dword ptr ss:[ebp-50]

004B3321  |.  BA 84344B00    mov edx,tic_exe.004B3484             ;  镱礤

004B3326  |.  E8 F918F5FF    call tic_exe.00404C24

004B332B  |.  75 78          jnz short tic_exe.004B33A5

004B332D  |.  8D55 AC        lea edx,dword ptr ss:[ebp-54]

004B3330  |.  8B83 20030000 mov eax,dword ptr ds:[ebx+320]

004B3336  |.  E8 1DC5FAFF    call tic_exe.0045F858

004B333B  |.  8B45 AC        mov eax,dword ptr ss:[ebp-54]

004B333E  |.  BA 84344B00    mov edx,tic_exe.004B3484             ;  镱礤

004B3343  |.  E8 DC18F5FF    call tic_exe.00404C24

004B3348  |.  75 01          jnz short tic_exe.004B334B

004B334A  |.  46             inc esi

004B334B  |>  8D55 A8        lea edx,dword ptr ss:[ebp-58]

004B334E  |.  8B83 24030000 mov eax,dword ptr ds:[ebx+324]

004B3354  |.  E8 FFC4FAFF    call tic_exe.0045F858

004B3359  |.  8B45 A8        mov eax,dword ptr ss:[ebp-58]

004B335C  |.  BA B4344B00    mov edx,tic_exe.004B34B4             ;  怛铕

004B3361  |.  E8 BE18F5FF    call tic_exe.00404C24

004B3366  |.  75 01          jnz short tic_exe.004B3369

004B3368  |.  46             inc esi

004B3369  |>  8D55 A4        lea edx,dword ptr ss:[ebp-5C]

004B336C  |.  8B83 28030000 mov eax,dword ptr ds:[ebx+328]

004B3372  |.  E8 E1C4FAFF    call tic_exe.0045F858

004B3377  |.  8B45 A4        mov eax,dword ptr ss:[ebp-5C]

004B337A  |.  BA C4344B00    mov edx,tic_exe.004B34C4             ;  耩邃

004B337F  |.  E8 A018F5FF    call tic_exe.00404C24

004B3384  |.  75 01          jnz short tic_exe.004B3387

004B3386  |.  46             inc esi

004B3387  |>  8D55 A0        lea edx,dword ptr ss:[ebp-60]

004B338A  |.  8B83 2C030000 mov eax,dword ptr ds:[ebx+32C]

004B3390  |.  E8 C3C4FAFF    call tic_exe.0045F858

004B3395  |.  8B45 A0        mov eax,dword ptr ss:[ebp-60]

004B3398  |.  BA D4344B00    mov edx,tic_exe.004B34D4             ;  麇蜮

004B339D  |.  E8 8218F5FF    call tic_exe.00404C24

004B33A2  |.  75 01          jnz short tic_exe.004B33A5

004B33A4  |.  46             inc esi

004B33A5  |>  83FE 04        cmp esi,4

004B33A8  |.  0F85 A6000000 jnz tic_exe.004B3454

004B33AE  |.  8D55 9C        lea edx,dword ptr ss:[ebp-64]

004B33B1  |.  8B83 38030000 mov eax,dword ptr ds:[ebx+338]

004B33B7  |.  E8 9CC4FAFF    call tic_exe.0045F858

004B33BC  |.  8B55 9C        mov edx,dword ptr ss:[ebp-64]

004B33BF  |.  A1 34FC4B00    mov eax,dword ptr ds:[4BFC34]

004B33C4  |.  8B00           mov eax,dword ptr ds:[eax]

004B33C6  |.  8B80 0C030000 mov eax,dword ptr ds:[eax+30C]

004B33CC  |.  E8 B7C4FAFF    call tic_exe.0045F888

004B33D1  |.  A1 841C4C00    mov eax,dword ptr ds:[4C1C84]

004B33D6  |.  E8 2197FCFF    call tic_exe.0047CAFC

004B33DB  |.  B2 01          mov dl,1

004B33DD  |.  A1 4CA74300    mov eax,dword ptr ds:[43A74C]

004B33E2  |.  E8 6574F8FF    call tic_exe.0043A84C

004B33E7  |.  8BF0           mov esi,eax

004B33E9  |.  BA 02000080    mov edx,80000002

004B33EE  |.  8BC6           mov eax,esi

004B33F0  |.  E8 F774F8FF    call tic_exe.0043A8EC

004B33F5  |.  B1 01          mov cl,1

004B33F7  |.  BA E4344B00    mov edx,tic_exe.004B34E4             ;  software\image collection

004B33FC  |.  8BC6           mov eax,esi

004B33FE  |.  E8 4D75F8FF    call tic_exe.0043A950

004B3403  |.  8D55 98        lea edx,dword ptr ss:[ebp-68]

004B3406  |.  8B83 38030000 mov eax,dword ptr ds:[ebx+338]

004B340C  |.  E8 47C4FAFF    call tic_exe.0045F858

004B3411  |.  8B4D 98        mov ecx,dword ptr ss:[ebp-68]

004B3414  |.  BA 08354B00    mov edx,tic_exe.004B3508             ;  name

004B3419  |.  8BC6           mov eax,esi

004B341B  |.  E8 CC76F8FF    call tic_exe.0043AAEC

004B3420  |.  B9 18354B00    mov ecx,tic_exe.004B3518             ;  a:

004B3425  |.  BA 24354B00    mov edx,tic_exe.004B3524             ;  drive

004B342A  |.  8BC6           mov eax,esi

004B342C  |.  E8 BB76F8FF    call tic_exe.0043AAEC

004B3431  |.  8BC6           mov eax,esi

004B3433  |.  E8 8474F8FF    call tic_exe.0043A8BC

004B3438  |.  8BC6           mov eax,esi

004B343A  |.  E8 8906F5FF    call tic_exe.00403AC8

004B343F  |.  6A 00          push 0

004B3441  |.  66:8B0D 2C354>mov cx,word ptr ds:[4B352C]

004B3448  |.  B2 02          mov dl,2

004B344A  |.  B8 38354B00    mov eax,tic_exe.004B3538             ;  注册成功提示 registration successfully complete. thank you for using this software!

004B344F  |.  E8 48E5F7FF    call tic_exe.0043199C

004B3454  |>  33C0           xor eax,eax

004B3456  |.  5A             pop edx

004B3457  |.  59             pop ecx

004B3458  |.  59             pop ecx

004B3459  |.  64:8910        mov dword ptr fs:[eax],edx

004B345C  |.  68 76344B00    push tic_exe.004B3476

004B3461  |>  8D45 98        lea eax,dword ptr ss:[ebp-68]

004B3464  |.  BA 1A000000    mov edx,1A

004B3469  |.  E8 CE13F5FF    call tic_exe.0040483C

004B346E  \\.  C3             retn
```

算法 `call(1)` `call(2)` `call(3)` `call(4)`

```
00408E14    53             push ebx

00408E15    56             push esi

00408E16    83C4 F4        add esp,-0C

00408E19    8BD8           mov ebx,eax

00408E1B    8BD4           mov edx,esp

00408E1D    8BC3           mov eax,ebx

00408E1F    E8 24A5FFFF    call tic_exe.00403348             ; 算法 call(5)

00408E24    8BF0           mov esi,eax

00408E26    833C24 00      cmp dword ptr ss:[esp],0

00408E2A    74 19          je short tic_exe.00408E45

00408E2C    895C24 04      mov dword ptr ss:[esp+4],ebx

00408E30    C64424 08 0B   mov byte ptr ss:[esp+8],0B

00408E35    8D5424 04      lea edx,dword ptr ss:[esp+4]

00408E39    A1 38FD4B00    mov eax,dword ptr ds:[4BFD38]

00408E3E    33C9           xor ecx,ecx

00408E40    E8 9BFAFFFF    call tic_exe.004088E0

00408E45    8BC6           mov eax,esi

00408E47    83C4 0C        add esp,0C

00408E4A    5E             pop esi

00408E4B    5B             pop ebx
```
