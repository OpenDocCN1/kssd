
```
00411B49  |.  8965 E4            mov     dword ptr [ebp-0x1C], esp

00411B4C  |.  8BCC               mov     ecx, esp

00411B4E  |.  50                 push    eax                                 ; /Arg1

00411B4F  |.  E8 B3E1FFFF        call    0040FD07                            ; \7-不问少.0040FD07

00411B54  |.  C645 FC 01         mov     byte ptr [ebp-0x4], 0x1

00411B58  |.  885D FC            mov     byte ptr [ebp-0x4], bl

00411B5B  |.  8BCE               mov     ecx, esi

00411B5D  |.  E8 7A010000        call    00411CDC

00411B62  |.  84C0               test    al, al

00411B64  |.  0F84 5B010000      je      00411CC5

00411B6A  |.  8D4E 24            lea     ecx, dword ptr [esi+0x24]

00411B6D  |.  8D45 08            lea     eax, dword ptr [ebp+0x8]

00411B70  |.  3BC8               cmp     ecx, eax

00411B72  |.  74 09              je      short 00411B7D

00411B74  |.  6A FF              push    -0x1                                ; /Arg3 = FFFFFFFF

00411B76  |.  53                 push    ebx                                 ; |Arg2

00411B77  |.  50                 push    eax                                 ; |Arg1

00411B78  |.  E8 BBEFFFFF        call    00410B38                            ; \7-不问少.00410B38

00411B7D  |>  8B45 18            mov     eax, dword ptr [ebp+0x18]

00411B80  |.  8D4D F0            lea     ecx, dword ptr [ebp-0x10]

00411B83  |.  51                 push    ecx

00411B84  |.  50                 push    eax

00411B85  |.  8D4D D8            lea     ecx, dword ptr [ebp-0x28]

00411B88  |.  8945 D0            mov     dword ptr [ebp-0x30], eax

00411B8B  |.  885D F0            mov     byte ptr [ebp-0x10], bl

00411B8E  |.  895D D8            mov     dword ptr [ebp-0x28], ebx

00411B91  |.  895D DC            mov     dword ptr [ebp-0x24], ebx

00411B94  |.  895D E0            mov     dword ptr [ebp-0x20], ebx

00411B97  |.  E8 B5010000        call    00411D51

00411B9C  |.  C645 FC 02         mov     byte ptr [ebp-0x4], 0x2

00411BA0  |.  8D4D 08            lea     ecx, dword ptr [ebp+0x8]

00411BA3  |.  8B5D 1C            mov     ebx, dword ptr [ebp+0x1C]

00411BA6  |.  83FB 10            cmp     ebx, 0x10

00411BA9  |.  8B45 18            mov     eax, dword ptr [ebp+0x18]

00411BAC  |.  0F434D 08          cmovnb  ecx, dword ptr [ebp+0x8]

00411BB0  |.  03C1               add     eax, ecx

00411BB2  |.  83FB 10            cmp     ebx, 0x10

00411BB5  |.  8D4D 08            lea     ecx, dword ptr [ebp+0x8]

00411BB8  |.  8BF8               mov     edi, eax

00411BBA  |.  0F434D 08          cmovnb  ecx, dword ptr [ebp+0x8]

00411BBE  |.  33D2               xor     edx, edx

00411BC0  |.  2155 EC            and     dword ptr [ebp-0x14], edx

00411BC3  |.  2BF9               sub     edi, ecx

00411BC5  |.  3BC8               cmp     ecx, eax

00411BC7  |.  0F477D EC          cmova   edi, dword ptr [ebp-0x14]

00411BCB  |.  85FF               test    edi, edi

00411BCD  |.  74 0C              je      short 00411BDB

00411BCF  |>  8031 CC            /xor     byte ptr [ecx], 0xCC

00411BD2  |.  41                 |inc     ecx

00411BD3  |.  42                 |inc     edx

00411BD4  |.  3BD7               |cmp     edx, edi

00411BD6  |.^ 75 F7              \jnz     short 00411BCF

00411BD8  |.  8B5D 1C            mov     ebx, dword ptr [ebp+0x1C]

00411BDB  |>  33C9               xor     ecx, ecx

00411BDD  |.  894D E8            mov     dword ptr [ebp-0x18], ecx

00411BE0  |.  394D D0            cmp     dword ptr [ebp-0x30], ecx

00411BE3  |.  0F8E B1000000      jle     00411C9A

00411BE9  |.  8B7D D8            mov     edi, dword ptr [ebp-0x28]

00411BEC  |.  33C0               xor     eax, eax

00411BEE  |.  40                 inc     eax

00411BEF  |.  897D EC            mov     dword ptr [ebp-0x14], edi

00411BF2  |.  2BC7               sub     eax, edi

00411BF4  |.  C745 E4 05000000   mov     dword ptr [ebp-0x1C], 0x5

00411BFB  |.  8945 CC            mov     dword ptr [ebp-0x34], eax

00411BFE  |>  FF75 E4            /push    dword ptr [ebp-0x1C]               ; /Arg4

00411C01  |.  83FB 10            |cmp     ebx, 0x10                          ; |

00411C04  |.  8D45 08            |lea     eax, dword ptr [ebp+0x8]           ; |

00411C07  |.  0F4345 08          |cmovnb  eax, dword ptr [ebp+0x8]           ; |

00411C0B  |.  03C1               |add     eax, ecx                           ; |

00411C0D  |.  50                 |push    eax                                ; |Arg3

00411C0E  |.  FF76 04            |push    dword ptr [esi+0x4]                ; |Arg2

00411C11  |.  FF36               |push    dword ptr [esi]                    ; |Arg1

00411C13  |.  E8 34F3FFFF        |call    <\_FindCharInStr>                   ; \get index in str

00411C18  |.  8BC8               |mov     ecx, eax

00411C1A  |.  83C4 10            |add     esp, 0x10

00411C1D  |.  3B4E 04            |cmp     ecx, dword ptr [esi+0x4]

00411C20  |.  74 60              |je      short 00411C82

00411C22  |.  8AC1               |mov     al, cl

00411C24  |.  2A06               |sub     al, byte ptr [esi]

00411C26  |.  8807               |mov     byte ptr [edi], al

00411C28  |.  75 05              |jnz     short 00411C2F

00411C2A  |.  33DB               |xor     ebx, ebx

00411C2C  |.  43                 |inc     ebx

00411C2D  |.  EB 03              |jmp     short 00411C32

00411C2F  |>  0FBED8             |movsx   ebx, al

00411C32  |>  837D E8 00         |cmp     dword ptr [ebp-0x18], 0x0

00411C36  |.  881F               |mov     byte ptr [edi], bl

00411C38  |.  7C 3E              |jl      short 00411C78

00411C3A  |.  8B45 CC            |mov     eax, dword ptr [ebp-0x34]

00411C3D  |.  03C7               |add     eax, edi

00411C3F  |.  8945 D4            |mov     dword ptr [ebp-0x2C], eax

00411C42  |>  33FF               |/xor     edi, edi

00411C44  |>  41                 ||/inc     ecx

00411C45  |.  3B4E 04            |||cmp     ecx, dword ptr [esi+0x4]

00411C48  |.  75 02              |||jnz     short 00411C4C

00411C4A  |.  8B0E               |||mov     ecx, dword ptr [esi]

00411C4C  |>  0FBEC3             |||movsx   eax, bl

00411C4F  |.  47                 |||inc     edi

00411C50  |.  99                 |||cdq

00411C51  |.  F77D E4            |||idiv    dword ptr [ebp-0x1C]

00411C54  |.  6A 05              |||push    0x5

00411C56  |.  5A                 |||pop     edx

00411C57  |.  03C2               |||add     eax, edx

00411C59  |.  3BF8               |||cmp     edi, eax

00411C5B  |.^ 75 E7              ||\jnz     short 00411C44

00411C5D  |.  8B7D EC            ||mov     edi, dword ptr [ebp-0x14]

00411C60  |.  8AC1               ||mov     al, cl

00411C62  |.  2A06               ||sub     al, byte ptr [esi]

00411C64  |.  8807               ||mov     byte ptr [edi], al

00411C66  |.  75 05              ||jnz     short 00411C6D

00411C68  |.  33DB               ||xor     ebx, ebx

00411C6A  |.  43                 ||inc     ebx

00411C6B  |.  EB 03              ||jmp     short 00411C70

00411C6D  |>  0FBED8             ||movsx   ebx, al

00411C70  |>  836D D4 01         ||sub     dword ptr [ebp-0x2C], 0x1

00411C74  |.  881F               ||mov     byte ptr [edi], bl

00411C76  |.^ 75 CA              |\jnz     short 00411C42

00411C78  |>  0FBE0F             |movsx   ecx, byte ptr [edi]

00411C7B  |.  8B06               |mov     eax, dword ptr [esi]

00411C7D  |.  8A0401             |mov     al, byte ptr [ecx+eax]

00411C80  |.  8807               |mov     byte ptr [edi], al

00411C82  |>  8B4D E8            |mov     ecx, dword ptr [ebp-0x18]

00411C85  |.  41                 |inc     ecx

00411C86  |.  47                 |inc     edi

00411C87  |.  894D E8            |mov     dword ptr [ebp-0x18], ecx

00411C8A  |.  897D EC            |mov     dword ptr [ebp-0x14], edi

00411C8D  |.  3B4D D0            |cmp     ecx, dword ptr [ebp-0x30]

00411C90  |.  7D 08              |jge     short 00411C9A
```
