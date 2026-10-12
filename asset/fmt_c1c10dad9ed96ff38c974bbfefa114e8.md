
```
004010E0    81EC 50010000   sub     esp, 150
004010E6    53              push    ebx
004010E7    55              push    ebp
004010E8    56              push    esi                     ; 29.00402340
004010E9    57              push    edi
004010EA    33C0            xor     eax, eax
004010EC    B9 49000000     mov     ecx, 49
004010F1    8D7C24 3C       lea     edi, dword ptr ss:\[esp+3C\]
004010F5    50              push    eax
004010F6    F3:AB           rep     stos dword ptr es:\[edi\]
004010F8    6A 02           push    2
004010FA    C74424 40 28010>mov     dword ptr ss:\[esp+40\], 128
00401102    E8 CB170000     call    004028D2                 ; jmp 到 kernel32.CreateToolhelp32Snapshot
00401107    8BE8            mov     ebp, eax
00401109    83FD FF         cmp     ebp, -1
0040110C    75 0D           jnz     short 0040111B             ; 29.0040111B
0040110E    5F              pop     edi                      ; 0012F838
0040110F    5E              pop     esi                      ; 0012F838
00401110    5D              pop     ebp                      ; 0012F838
00401111    33C0            xor     eax, eax
00401113    5B              pop     ebx                      ; 0012F838
00401114    81C4 50010000   add     esp, 150
0040111A    C3              retn
0040111B    8D4424 38       lea     eax, dword ptr ss:\[esp+38\]
0040111F    50              push    eax
00401120    55              push    ebp
00401121    E8 B8170000     call    004028DE                 ; jmp 到 kernel32.Process32First
00401126    85C0            test    eax, eax
00401128    75 0B           jnz     short 00401135             ; 29.00401135
0040112A    5F              pop     edi                      ; 0012F838
0040112B    5E              pop     esi                      ; 0012F838
0040112C    5D              pop     ebp                      ; 0012F838
0040112D    5B              pop     ebx                      ; 0012F838
0040112E    81C4 50010000   add     esp, 150
00401134    C3              retn
00401135    B9 0A000000     mov     ecx, 0A
0040113A    33C0            xor     eax, eax
0040113C    8D7C24 10       lea     edi, dword ptr ss:\[esp+10\]
00401140    8D5424 10       lea     edx, dword ptr ss:\[esp+10\]
00401144    F3:AB           rep     stos dword ptr es:\[edi\]
00401146    8D7C24 5C       lea     edi, dword ptr ss:\[esp+5C\]
0040114A    83C9 FF         or      ecx, FFFFFFFF
0040114D    F2:AE           repne   scas byte ptr es:\[edi\]
0040114F    F7D1            not     ecx
00401151    2BF9            sub     edi, ecx
00401153    8BC1            mov     eax, ecx
00401155    8BF7            mov     esi, edi
00401157    8BFA            mov     edi, edx
00401159    C1E9 02         shr     ecx, 2
0040115C    F3:A5           rep     movs dword ptr es:\[edi\], dword p>
0040115E    8BC8            mov     ecx, eax
00401160    83E1 03         and     ecx, 3
00401163    F3:A4           rep     movs byte ptr es:\[edi\], byte ptr>
00401165    8A4C24 10       mov     cl, byte ptr ss:\[esp+10\]
00401169    8A5424 13       mov     dl, byte ptr ss:\[esp+13\]
0040116D    8A4424 11       mov     al, byte ptr ss:\[esp+11\]
00401171    80F9 51         cmp     cl, 51
00401174    74 05           je      short 0040117B             ; 29.0040117B
00401176    80F9 71         cmp     cl, 71
00401179    75 35           jnz     short 004011B0             ; 29.004011B0
0040117B    3C 51           cmp     al, 51
0040117D    74 04           je      short 00401183             ; 29.00401183
0040117F    3C 71           cmp     al, 71
00401181    75 2D           jnz     short 004011B0             ; 29.004011B0
00401183    807C24 12 2E    cmp     byte ptr ss:\[esp+12\], 2E
00401188    75 26           jnz     short 004011B0             ; 29.004011B0
0040118A    8A5C24 14       mov     bl, byte ptr ss:\[esp+14\]
0040118E    80FB 58         cmp     bl, 58
00401191    74 05           je      short 00401198             ; 29.00401198
00401193    80FB 78         cmp     bl, 78
00401196    75 18           jnz     short 004011B0             ; 29.004011B0
00401198    80FA 45         cmp     dl, 45
0040119B    74 05           je      short 004011A2             ; 29.004011A2
0040119D    80FA 65         cmp     dl, 65
004011A0    75 0E           jnz     short 004011B0             ; 29.004011B0
004011A2    8A5C24 15       mov     bl, byte ptr ss:\[esp+15\]
004011A6    80FB 45         cmp     bl, 45
004011A9    74 5E           je      short 00401209             ; 29.00401209
004011AB    80FB 65         cmp     bl, 65
004011AE    74 59           je      short 00401209             ; 29.00401209
004011B0    3C 4D           cmp     al, 4D
004011B2    74 04           je      short 004011B8             ; 29.004011B8
004011B4    3C 6D           cmp     al, 6D
004011B6    75 33           jnz     short 004011EB             ; 29.004011EB
004011B8    807C24 12 2E    cmp     byte ptr ss:\[esp+12\], 2E
004011BD    75 2C           jnz     short 004011EB             ; 29.004011EB
004011BF    80FA 45         cmp     dl, 45
004011C2    74 05           je      short 004011C9             ; 29.004011C9
004011C4    80FA 65         cmp     dl, 65
004011C7    75 22           jnz     short 004011EB             ; 29.004011EB
004011C9    8A4424 14       mov     al, byte ptr ss:\[esp+14\]
004011CD    3C 58           cmp     al, 58
004011CF    74 04           je      short 004011D5             ; 29.004011D5
004011D1    3C 78           cmp     al, 78
004011D3    75 16           jnz     short 004011EB             ; 29.004011EB
004011D5    8A4424 15       mov     al, byte ptr ss:\[esp+15\]
004011D9    3C 45           cmp     al, 45
004011DB    74 04           je      short 004011E1             ; 29.004011E1
004011DD    3C 65           cmp     al, 65
004011DF    75 0A           jnz     short 004011EB             ; 29.004011EB
004011E1    80F9 54         cmp     cl, 54
004011E4    74 23           je      short 00401209             ; 29.00401209
004011E6    80F9 74         cmp     cl, 74
004011E9    74 1E           je      short 00401209             ; 29.00401209
004011EB    8D4C24 38       lea     ecx, dword ptr ss:\[esp+38\]
004011EF    51              push    ecx
004011F0    55              push    ebp
004011F1    E8 E2160000     call    004028D8                 ; jmp 到 kernel32.Process32Next
004011F6    85C0            test    eax, eax
004011F8   ^ 0F85 37FFFFFF   jnz     00401135                 ; 29.00401135
004011FE    5F              pop     edi                      ; 0012F838
004011FF    5E              pop     esi                      ; 0012F838
00401200    5D              pop     ebp                      ; 0012F838
00401201    5B              pop     ebx                      ; 0012F838
00401202    81C4 50010000   add     esp, 150
00401208    C3              retn
```
