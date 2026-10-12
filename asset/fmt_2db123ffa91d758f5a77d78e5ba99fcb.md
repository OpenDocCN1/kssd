
100594E0    8D8E EC00000    lea     ecx, dword ptr [esi+EC]  
100594E6    52            push    edx  
100594E7    C607 00       mov     byte ptr [edi], 0  
100594EA    E8 BD840500   call    100B19AC  
100594EF    47            inc     edi  
100594F0    57            push    edi  
100594F1    EB 12         jmp     short 10059505  
10059505    8D8E F000000  lea     ecx, dword ptr [esi+F0]  
1005950B    E8 9C840500   call    100B19AC                      ; 返回 EAX==100ED058 krnln.100ED058  
10059510    837B 2C 02    cmp     dword ptr [ebx+2C], 2  
10059514    74 0F         je      short 10059525  
10059516    8B86 EC00000  mov     eax, dword ptr [esi+EC]  
1005951C    50            push    eax  
1005951D    FF15 8C330C10 call    dword ptr [<&KERNEL32.SetCurrentDirectoryA>]  ; kernel32.SetCurrentDirectoryA  
10059523    EB 4C         jmp     short 10059571  
10059571    8B46 68       mov     eax, dword ptr [esi+68]  
10059574    68 CC450E10   push    100E45CC                      ; ASCII "mp3"  
10059579    50            push    eax  
1005957A    8D8E FC030000 lea     ecx, dword ptr [esi+3FC]  
10059580    E8 6B9FFBFF   call    100134F0  
10059585    8B4E 68       mov     ecx, dword ptr [esi+68]  
10059588    68 44410E10   push    100E4144                      ; ASCII "odbcdb"  
1005958D    51            push    ecx  
1005958E    8D8E 0C040000 lea     ecx, dword ptr [esi+40C]  
10059594    E8 579FFBFF   call    100134F0  
10059599    8B56 68       mov     edx, dword ptr [esi+68]  
1005959C    68 C4450E10   push    100E45C4                      ; ASCII "ComInf"  
100595A1    52            push    edx  
100595A2    8D8E 3C040000 lea     ecx, dword ptr [esi+43C]  
100595A8    E8 439FFBFF   call    100134F0  
100595AD    33FF          xor     edi, edi  
100595AF    68 007F0000   push    7F00  
100595B4    57            push    edi  
100595B5    FF15 D0360C10 call    dword ptr [<&USER32.LoadCursorA>]             ; USER32.LoadCursorA  
100595BB    57            push    edi  
100595BC    57            push    edi  
100595BD    57            push    edi  
100595BE    57            push    edi  
100595BF    57            push    edi  
100595C0    57            push    edi  
100595C1    57            push    edi  
100595C2    57            push    edi  
100595C3    68 90C90E10   push    100EC990  
100595C8    57            push    edi  
100595C9    6A 05         push    5  
100595CB    8945 D0       mov     dword ptr [ebp-30], eax  
100595CE    FF15 48310C10 call    dword ptr [<&GDI32.GetStockObject>]           ; GDI32.GetStockObject  
100595D4    50            push    eax  
100595D5    8B45 D0       mov     eax, dword ptr [ebp-30]  
100595D8    50            push    eax  
100595D9    57            push    edi  
100595DA    E8 D55C0500   call    100AF2B4  
100595DF    50            push    eax  
100595E0    68 80000000   push    80  
100595E5    8D8E 48040000 lea     ecx, dword ptr [esi+448]  
100595EB    E8 E0550500   call    100AEBD0  
100595F0    FF15 98330C10 call    dword ptr [<&KERNEL32.GetCurrentThreadId>]     ; kernel32.GetCurrentThreadId  
100595F6    8986 B4020000 mov     dword ptr [esi+2B4], eax  
100595FC    EB 1B         jmp     short 10059619  
10059619    8B43 48       mov     eax, dword ptr [ebx+48]  
1005961C    897D 98       mov     dword ptr [ebp-68], edi  
1005961F    83F8 FF       cmp     eax, -1  
10059622    897D 9C       mov     dword ptr [ebp-64], edi  
10059625    897D A0       mov     dword ptr [ebp-60], edi  
10059628    897D A4       mov     dword ptr [ebp-5C], edi  
1005962B    74 2F         je      short 1005965C  
1005962D    8B7C18 2C     mov     edi, dword ptr [eax+ebx+2C]  
10059631    03FB          add     edi, ebx  
10059633    897D 98       mov     dword ptr [ebp-68], edi  
10059636    8B4418 24     mov     eax, dword ptr [eax+ebx+24]  
1005963A    83F8 34       cmp     eax, 34  
1005963D    72 05         jb      short 10059644  
1005963F    B8 34000000   mov     eax, 34  
10059644    8BC8          mov     ecx, eax  
10059646    BE E8370E10   mov     esi, 100E37E8  
1005964B    8BD1          mov     edx, ecx  
1005964D    C1E9 02       shr     ecx, 2  
10059650    F3:A5         rep     movs dword ptr es:[edi], dword ptr [esi]  
10059652    8BCA          mov     ecx, edx  
10059654    83E1 03       and     ecx, 3  
10059657    F3:A4         rep     movs byte ptr es:[edi], byte ptr [esi]  
10059659    8B75 EC       mov     esi, dword ptr [ebp-14]  
1005965C    8B86 DC000000 mov     eax, dword ptr [esi+DC]  
10059662    8D8E 18020000 lea     ecx, dword ptr [esi+218]  
10059668    50            push    eax  
10059669    E8 C205FDFF   call    10029C30  
1005966E    8B43 40       mov     eax, dword ptr [ebx+40]  
10059671    83F8 FF       cmp     eax, -1  
10059674    8945 E4       mov     dword ptr [ebp-1C], eax  
10059677    74 2C         je      short 100596A5  
10059679    8B7C18 2C     mov     edi, dword ptr [eax+ebx+2C]  
1005967D    8D8E 04020000 lea     ecx, dword ptr [esi+204]  
10059683    03FB          add     edi, ebx  
10059685    57            push    edi  
10059686    E8 A505FDFF   call    10029C30  
1005968B    8B4D E4       mov     ecx, dword ptr [ebp-1C]  
1005968E    8B4419 24     mov     eax, dword ptr [ecx+ebx+24]  
10059692    8D8E 04020000 lea     ecx, dword ptr [esi+204]  
10059698    03C7          add     eax, edi  
1005969A    50            push    eax  
1005969B    E8 9005FDFF   call    10029C30  
100596A0    897D 9C       mov     dword ptr [ebp-64], edi  
100596A3    EB 18         jmp     short 100596BD  
100596BD    8B43 4C       mov     eax, dword ptr [ebx+4C]  
100596C0    C745 C4 0000000 mov   dword ptr [ebp-3C], 0  
100596C7    83F8 FF       cmp     eax, -1  
100596CA    74 0C         je      short 100596D8  
100596CC    8B4418 2C     mov     eax, dword ptr [eax+ebx+2C]  
100596D0    03C3          add     eax, ebx  
100596D2    8945 C4       mov     dword ptr [ebp-3C], eax  
100596D5    8945 A4       mov     dword ptr [ebp-5C], eax  
100596D8    8B43 50       mov     eax, dword ptr [ebx+50]  
100596DB    83F8 FF       cmp     eax, -1  
100596DE    8945 D8       mov     dword ptr [ebp-28], eax  
100596E1    74 60         je      short 10059743  
100596E3    8B4418 24     mov     eax, dword ptr [eax+ebx+24]  
100596E7    85C0          test    eax, eax  
100596E9    7E 58         jle     short 10059743  
100596EB    50            push    eax  
100596EC    E8 3C4A0500   call    100AE12D  
100596F1    8B55 D8       mov     edx, dword ptr [ebp-28]  
100596F4    8BF8          mov     edi, eax  
100596F6    33C0          xor     eax, eax  
100596F8    897D E4       mov     dword ptr [ebp-1C], edi  
100596FB    8B4C1A 24     mov     ecx, dword ptr [edx+ebx+24]  
100596FF    83C4 04       add     esp, 4  
10059702    8BD1          mov     edx, ecx  
10059704    81C6 F8000000 add     esi, 0F8  
1005970A    C1E9 02       shr     ecx, 2  
1005970D    F3:AB         rep     stos dword ptr es:[edi]  
1005970F    8BCA          mov     ecx, edx  
10059711    83E1 03       and     ecx, 3  
10059714    F3:AA         rep     stos byte ptr es:[edi]  
10059716    8B45 E4       mov     eax, dword ptr [ebp-1C]  
10059719    8BCE          mov     ecx, esi  
1005971B    50            push    eax  
1005971C    E8 0F05FDFF   call    10029C30  
10059721    8B45 E4       mov     eax, dword ptr [ebp-1C]  
