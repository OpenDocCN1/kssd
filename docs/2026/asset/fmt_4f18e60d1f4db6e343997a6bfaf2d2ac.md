
```text
004D0B6A   .  8D4D F4       lea     ecx, [ebp-C]                     ; |
004D0B6D   .  BA 600E4D00   mov     edx, 004D0E60                    ; |threadingmodelnn
004D0B72   .  8B45 FC       mov     eax, [ebp-4]                     ; |
004D0B75   .  E8 D62EF9FF   call    00463A50                         ; \CLBD.00463A50
004D0B7A   .  8B45 FC       mov     eax, [ebp-4]
004D0B7D   .  E8 B62BF9FF   call    00463738
004D0B82   .  33C0          xor     eax, eax
004D0B84   .  5A            pop     edx
004D0B85   .  59            pop     ecx
004D0B86   .  59            pop     ecx
004D0B87   .  64:8910       mov     fs:[eax], edx
004D0B8A   .  68 9F0B4D00   push    004D0B9F
004D0B8F   >  8B45 FC       mov     eax, [ebp-4]
004D0B92   .  E8 7D2BF3FF   call    00403714
004D0B97   .  C3            retn
004D0B98   .^ E9 0B33F3FF   jmp     00403EA8
004D0B9D   .^ EB F0         jmp     short 004D0B8F
004D0B9F   .  B2 01         mov     dl, 1
004D0BA1   .  A1 C8354600   mov     eax, [4635C8]
004D0BA6   .  E8 1D2BF9FF   call    004636C8
004D0BAB   .  8945 FC       mov     [ebp-4], eax
004D0BAE   .  33C0          xor     eax, eax
004D0BB0   .  55            push    ebp
004D0BB1   .  68 1A0C4D00   push    004D0C1A
004D0BB6   .  64:FF30       push    dword ptr fs:[eax]
004D0BB9   .  64:8920       mov     fs:[eax], esp
004D0BBC   .  BA 02000080   mov     edx, 80000002
004D0BC1   .  8B45 FC       mov     eax, [ebp-4]
004D0BC4   .  E8 9F2BF9FF   call    00463768
004D0BC9   .  B1 01         mov     cl, 1
004D0BCB   .  BA 7C0E4D00   mov     edx, 004D0E7C                    ;  ASCII "\Software\CLASSES\CLSID\{CD356868-0605-0606-0611-abcd20060428}\InprocServer32"
004D0BD0   .  8B45 FC       mov     eax, [ebp-4]
004D0BD3   .  E8 F42BF9FF   call    004637CC
004D0BD8   .  6A 04         push    4                                ; /Arg1 = 00000004
004D0BDA   .  8D4D F8       lea     ecx, [ebp-8]                     ; |
004D0BDD   .  BA 480E4D00   mov     edx, 004D0E48                    ; |ASCII "ThreadingModel"
004D0BE2   .  8B45 FC       mov     eax, [ebp-4]                     ; |
004D0BE5   .  E8 662EF9FF   call    00463A50                         ; \CLBD.00463A50
004D0BEA   .  6A 04         push    4                                ; /Arg1 = 00000004
004D0BEC   .  8D4D F4       lea     ecx, [ebp-C]                     ; |
004D0BEF   .  BA 600E4D00   mov     edx, 004D0E60                    ; |ASCII "ThreadingModelnn"
004D0BF4   .  8B45 FC       mov     eax, [ebp-4]                     ; |
004D0BF7   .  E8 542EF9FF   call    00463A50                         ; \CLBD.00463A50
004D0BFC   .  8B45 FC       mov     eax, [ebp-4]
004D0BFF   .  E8 342BF9FF   call    00463738
004D0C04   .  33C0          xor     eax, eax
004D0C06   .  5A            pop     edx
004D0C07   .  59            pop     ecx
004D0C08   .  59            pop     ecx
004D0C09   .  64:8910       mov     fs:[eax], edx
004D0C0C   .  68 8F0C4D00   push    004D0C8F
004D0C11   >  8B45 FC       mov     eax, [ebp-4]
004D0C14   .  E8 FB2AF3FF   call    00403714
004D0C19   .  C3            retn
004D0C1A   .^ E9 8932F3FF   jmp     00403EA8
004D0C1F   .^ EB F0         jmp     short 004D0C11
004D0C21   >  8D55 AC       lea     edx, [ebp-54]
004D0C24   .  8B83 04030000 mov     eax, [ebx+304]
004D0C2A   .  E8 510AF7FF   call    00441680
004D0C2F   .  8B45 AC       mov     eax, [ebp-54]
004D0C32   .  8D55 B0       lea     edx, [ebp-50]
004D0C35   .  E8 3A7DF3FF   call    00408974
004D0C3A   .  8B45 B0       mov     eax, [ebp-50]
004D0C3D   .  E8 7A3BF3FF   call    004047BC
004D0C42   .  83F8 04       cmp     eax, 4
004D0C45   .  7D 12         jge     short 004D0C59
004D0C47   .  8B83 00030000 mov     eax, [ebx+300]
004D0C4D   .  BA D40E4D00   mov     edx, 004D0ED4
004D0C52   .  E8 590AF7FF   call    004416B0
004D0C57   .  EB 36         jmp     short 004D0C8F
004D0C59   >  8D55 A4       lea     edx, [ebp-5C]
004D0C5C   .  8B83 0C030000 mov     eax, [ebx+30C]
004D0C62   .  E8 190AF7FF   call    00441680
004D0C67   .  8B45 A4       mov     eax, [ebp-5C]
004D0C6A   .  8D55 A8       lea     edx, [ebp-58]
004D0C6D   .  E8 027DF3FF   call    00408974
004D0C72   .  8B45 A8       mov     eax, [ebp-58]
004D0C75   .  E8 423BF3FF   call    004047BC
004D0C7A   .  83F8 11       cmp     eax, 11
004D0C7D   .  74 10         je      short 004D0C8F
004D0C7F   .  8B83 00030000 mov     eax, [ebx+300]
004D0C85   .  BA 100F4D00   mov     edx, 004D0F10
004D0C8A   .  E8 210AF7FF   call    004416B0
004D0C8F   >  33C0          xor     eax, eax
004D0C91   .  5A            pop     edx
004D0C92   .  59            pop     ecx
004D0C93   .  59            pop     ecx
004D0C94   .  64:8910       mov     fs:[eax], edx
004D0C97   .  68 410D4D00   push    004D0D41
004D0C9C   >  8D45 A4       lea     eax, [ebp-5C]
004D0C9F   .  E8 6038F3FF   call    00404504
004D0CA4   .  8D45 A8       lea     eax, [ebp-58]
004D0CA7   .  E8 5838F3FF   call    00404504
004D0CAC   .  8D45 AC       lea     eax, [ebp-54]
004D0CAF   .  E8 5038F3FF   call    00404504
004D0CB4   .  8D45 B0       lea     eax, [ebp-50]
004D0CB7   .  BA 02000000   mov     edx, 2
004D0CBC   .  E8 6738F3FF   call    00404528
004D0CC1   .  8D45 B8       lea     eax, [ebp-48]
004D0CC4   .  E8 3B38F3FF   call    00404504
004D0CC9   .  8D45 BC       lea     eax, [ebp-44]
004D0CCC   .  E8 3338F3FF   call    00404504
004D0CD1   .  8D45 C0       lea     eax, [ebp-40]
004D0CD4   .  E8 2B38F3FF   call    00404504
004D0CD9   .  8D45 C4       lea     eax, [ebp-3C]
004D0CDC   .  E8 2338F3FF   call    00404504
004D0CE1   .  8D45 C8       lea     eax, [ebp-38]
004D0CE4   .  E8 1B38F3FF   call    00404504
004D0CE9   .  8D45 CC       lea     eax, [ebp-34]
004D0CEC   .  E8 1338F3FF   call    00404504
004D0CF1   .  8D45 D0       lea     eax, [ebp-30]
004D0CF4   .  E8 0B38F3FF   call    00404504
004D0CF9   .  8D45 D4       lea     eax, [ebp-2C]
004D0CFC   .  BA 02000000   mov     edx, 2
004D0D01   .  E8 2238F3FF   call    00404528
004D0D06   .  8D45 DC       lea     eax, [ebp-24]
004D0D09   .  E8 F637F3FF   call    00404504
004D0D0E   .  8D45 E0       lea     eax, [ebp-20]
004D0D11   .  E8 EE37F3FF   call    00404504
004D0D16   .  8D45 E4       lea     eax, [ebp-1C]
004D0D19   .  E8 E637F3FF   call    00404504
004D0D1E   .  8D45 E8       lea     eax, [ebp-18]
004D0D21   .  E8 DE37F3FF   call    00404504
004D0D26   .  8D45 EC       lea     eax, [ebp-14]
004D0D29   .  E8 D637F3FF   call    00404504
004D0D2E   .  8D45 F0       lea     eax, [ebp-10]
004D0D31   .  E8 CE37F3FF   call    00404504
004D0D36   .  C3            retn
004D0D37   .^ E9 6C31F3FF   jmp     00403EA8
004D0D3C   .^ E9 5BFFFFFF   jmp     004D0C9C
004D0D41   .  5E            pop     esi
004D0D42   .  5B            pop     ebx
004D0D43   .  8BE5          mov     esp, ebp
004D0D45   .  5D            pop     ebp
004D0D46   .  C3            retn

-------------- 在这里跟进  004D0A4E   .  E8 01F9FFFF   call    004D0354 到这里 ----------------

004D0354  /$  55            push    ebp
004D0355  |.  8BEC          mov     ebp, esp
004D0357  |.  51            push    ecx
004D0358  |.  B9 05000000   mov     ecx, 5
004D035D  |>  6A 00         /push    0
004D035F  |.  6A 00         |push    0
004D0361  |.  49            |dec     ecx
004D0362  |.^ 75 F9         \jnz     short 004D035D
004D0364  |.  874D FC       xchg    [ebp-4], ecx
004D0367  |.  53            push    ebx
004D0368  |.  56            push    esi
004D0369  |.  57            push    edi
004D036A  |.  8BF9          mov     edi, ecx
004D036C  |.  8955 F8       mov     [ebp-8], edx                     ;  (ASCII "651645186G01")
004D036F  |.  8945 FC       mov     [ebp-4], eax                     ;  用户名
004D0372  |.  8B45 FC       mov     eax, [ebp-4]
004D0375  |.  E8 2A46F3FF   call    004049A4
004D037A  |.  8B45 F8       mov     eax, [ebp-8]
004D037D  |.  E8 2246F3FF   call    004049A4
```
