
```
00469F36     .  C745 AC 02000000   mov     dword ptr [ebp-54], 2

00469F3D     .  C785 6CFFFFFF 0840>mov     dword ptr [ebp-94], 4008

00469F47     .  FF15 B8834A00      call    dword ptr [<&MSVBVM50.#632>]    ;  MSVBVM50.rtcMidCharVar

00469F4D     .  8D55 9C            lea     edx, dword ptr [ebp-64]

00469F50     .  52                 push    edx

00469F51     .  FFD6               call    esi

00469F53     .  8BD0               mov     edx, eax

00469F55     .  8D4D E0            lea     ecx, dword ptr [ebp-20]

00469F58     .  FFD7               call    edi

00469F5A     .  8D45 9C            lea     eax, dword ptr [ebp-64]

00469F5D     .  8D4D AC            lea     ecx, dword ptr [ebp-54]

00469F60     .  50                 push    eax

00469F61     .  51                 push    ecx

00469F62     .  6A 02              push    2

00469F64     .  FFD3               call    ebx

00469F66     .  8B55 E0            mov     edx, dword ptr [ebp-20]

00469F69     .  83C4 0C            add     esp, 0C

00469F6C     .  52                 push    edx

00469F6D     .  FF15 10834A00      call    dword ptr [<&MSVBVM50.#516>]    ;  MSVBVM50.rtcAnsiValueBstr

00469F73     .  66:8B4D D8         mov     cx, word ptr [ebp-28]           ;  先将注册名转为大写，再取每一位注册名进行运算

00469F77     .  66:6BC9 06         imul    cx, cx, 6                       ;  cx=6*cx

00469F7B     .  0F80 C5050000      jo      0046A546

00469F81     .  66:03C1            add     ax, cx                          ;  ax=ax+cx

00469F84     .  0F80 BC050000      jo      0046A546

00469F8A     .  66:2D 0F00         sub     ax, 0F                          ;  ax=ax-6f

00469F8E     .  0F80 B2050000      jo      0046A546

00469F94     .  66:3D 2100         cmp     ax, 21                          ;  和33比较，是否在asc字符区域

00469F98     .  7C 56              jl      short 00469FF0

00469F9A     .  66:3D 7E00         cmp     ax, 7E                          ;  和126比较

00469F9E     .  7F 4A              jg      short 00469FEA

00469FA0     .  8B55 DC            mov     edx, dword ptr [ebp-24]

00469FA3     .  8D4D AC            lea     ecx, dword ptr [ebp-54]

00469FA6     .  0FBFC0             movsx   eax, ax

00469FA9     .  50                 push    eax

00469FAA     .  51                 push    ecx

00469FAB     .  8995 74FFFFFF      mov     dword ptr [ebp-8C], edx

00469FB1     .  C785 6CFFFFFF 0800>mov     dword ptr [ebp-94], 8

00469FBB     .  FF15 64844A00      call    dword ptr [<&MSVBVM50.#608>]    ;  MSVBVM50.rtcVarBstrFromAnsi

00469FC1     .  8D95 6CFFFFFF      lea     edx, dword ptr [ebp-94]

00469FC7     .  8D45 AC            lea     eax, dword ptr [ebp-54]

00469FCA     .  52                 push    edx

00469FCB     .  8D4D 9C            lea     ecx, dword ptr [ebp-64]

00469FCE     .  50                 push    eax

00469FCF     .  51                 push    ecx

00469FD0     .  FF15 08854A00      call    dword ptr [<&MSVBVM50.\\_\\_vbaVarA>;  MSVBVM50.\\_\\_vbaVarAdd

00469FD6     .  50                 push    eax

00469FD7     .  FFD6               call    esi

00469FD9     .  8BD0               mov     edx, eax                        ;  注册码每次连接起来

00469FDB     .  8D4D DC            lea     ecx, dword ptr [ebp-24]

00469FDE     .  FFD7               call    edi

00469FE0     .  8D55 9C            lea     edx, dword ptr [ebp-64]

00469FE3     .  8D45 AC            lea     eax, dword ptr [ebp-54]

00469FE6     .  52                 push    edx

00469FE7     .  50                 push    eax

00469FE8     .  EB 5E              jmp     short 0046A048

00469FEA     >  66:3D 2100         cmp     ax, 21

00469FEE     .  7D 05              jge     short 00469FF5

00469FF0     >  B8 65000000        mov     eax, 65

00469FF5     >  66:3D 7E00         cmp     ax, 7E

00469FF9     .  7E 05              jle     short 0046A000

00469FFB     .  B8 2A000000        mov     eax, 2A

0046A000     >  8B4D DC            mov     ecx, dword ptr [ebp-24]

0046A003     .  C785 6CFFFFFF 0800>mov     dword ptr [ebp-94], 8

0046A00D     .  0FBFD0             movsx   edx, ax

0046A010     .  8D45 AC            lea     eax, dword ptr [ebp-54]

0046A013     .  52                 push    edx

0046A014     .  50                 push    eax

0046A015     .  898D 74FFFFFF      mov     dword ptr [ebp-8C], ecx

0046A01B     .  FF15 64844A00      call    dword ptr [<&MSVBVM50.#608>]    ;  MSVBVM50.rtcVarBstrFromAnsi

0046A021     .  8D8D 6CFFFFFF      lea     ecx, dword ptr [ebp-94]

0046A027     .  8D55 AC            lea     edx, dword ptr [ebp-54]

0046A02A     .  51                 push    ecx

0046A02B     .  8D45 9C            lea     eax, dword ptr [ebp-64]

0046A02E     .  52                 push    edx

0046A02F     .  50                 push    eax

0046A030     .  FF15 08854A00      call    dword ptr [<&MSVBVM50.\\_\\_vbaVarA>;  MSVBVM50.\\_\\_vbaVarAdd

0046A036     .  50                 push    eax

0046A037     .  FFD6               call    esi

0046A039     .  8BD0               mov     edx, eax

0046A03B     .  8D4D DC            lea     ecx, dword ptr [ebp-24]

0046A03E     .  FFD7               call    edi

0046A040     .  8D4D 9C            lea     ecx, dword ptr [ebp-64]

0046A043     .  8D55 AC            lea     edx, dword ptr [ebp-54]

0046A046     .  51                 push    ecx

0046A047     .  52                 push    edx

0046A048     >  6A 02              push    2

0046A04A     .  FFD3               call    ebx

0046A04C     .  66:8B45 D8         mov     ax, word ptr [ebp-28]

0046A050     .  83C4 0C            add     esp, 0C

0046A053     .  66:40              inc     ax

0046A055     .  0F80 EB040000      jo      0046A546

0046A05B     .  8945 D8            mov     dword ptr [ebp-28], eax

0046A05E     .^ E9 91FEFFFF        jmp     00469EF4

0046A063     >  8B7D 08            mov     edi, dword ptr [ebp+8]

0046A066     .  57                 push    edi

0046A067     .  8B07               mov     eax, dword ptr [edi]

0046A069     .  8B80 0C030000      mov     eax, dword ptr [eax+30C]

0046A06F     .  8985 18FFFFFF      mov     dword ptr [ebp-E8], eax

0046A075     .  FFD0               call    eax

0046A077     .  8D4D C0            lea     ecx, dword ptr [ebp-40]

0046A07A     .  50                 push    eax

0046A07B     .  51                 push    ecx

0046A07C     .  FF15 84834A00      call    dword ptr [<&MSVBVM50.\\_\\_vbaObjS>;  MSVBVM50.\\_\\_vbaObjSet

0046A082     .  8BF0               mov     esi, eax

0046A084     .  8D45 D4            lea     eax, dword ptr [ebp-2C]

0046A087     .  50                 push    eax

0046A088     .  56                 push    esi

0046A089     .  8B16               mov     edx, dword ptr [esi]

0046A08B     .  FF92 A0000000      call    dword ptr [edx+A0]

0046A091     .  85C0               test    eax, eax

0046A093     .  7D 12              jge     short 0046A0A7

0046A095     .  68 A0000000        push    0A0

0046A09A     .  68 C0F04000        push    0040F0C0

0046A09F     .  56                 push    esi

0046A0A0     .  50                 push    eax

0046A0A1     .  FF15 50834A00      call    dword ptr [<&MSVBVM50.\\_\\_vbaHres>;  MSVBVM50.\\_\\_vbaHresultCheckObj

0046A0A7     >  57                 push    edi

0046A0A8     .  FF95 18FFFFFF      call    dword ptr [ebp-E8]

0046A0AE     .  8D4D BC            lea     ecx, dword ptr [ebp-44]

0046A0B1     .  50                 push    eax

0046A0B2     .  51                 push    ecx

0046A0B3     .  FF15 84834A00      call    dword ptr [<&MSVBVM50.\\_\\_vbaObjS>;  MSVBVM50.\\_\\_vbaObjSet

0046A0B9     .  8BF0               mov     esi, eax

0046A0BB     .  8D45 D0            lea     eax, dword ptr [ebp-30]

0046A0BE     .  50                 push    eax
```
