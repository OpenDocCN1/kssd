
```text
004DA1CF  |. |BA 4CA44D00   mov     edx, 004DA44C                    ;  “MF”和“CB”比较。

004DA1D4  |. |E8 4F97F2FF   call    00403928

004DA1D9  |. |0F85 80000000 jnz     004DA25F                         ;  当然不等，跳

004DA1DF  |. |8D45 EC       lea     eax, [ebp-14]

004DA1E2  |. |50            push    eax

004DA1E3  |. |B9 20000000   mov     ecx, 20

004DA1E8  |. |BA 58A44D00   mov     edx, 004DA458                    ;  jtf

004DA1ED  |. |8B45 FC       mov     eax, [ebp-4]

004DA1F0  |. |E8 F7FDFFFF   call    004D9FEC

004DA1F5  |. |8D45 F0       lea     eax, [ebp-10]

004DA1F8  |. |50            push    eax

004DA1F9  |. |B9 10000000   mov     ecx, 10

004DA1FE  |. |BA 58A44D00   mov     edx, 004DA458                    ;  jtf

004DA203  |. |8B45 FC       mov     eax, [ebp-4]

004DA206  |. |E8 E1FDFFFF   call    004D9FEC

004DA20B  |. |8D55 E0       lea     edx, [ebp-20]

004DA20E  |. |8B45 F8       mov     eax, [ebp-8]

004DA211  |. |E8 5EDCFFFF   call    004D7E74

004DA216  |. |8B45 E0       mov     eax, [ebp-20]

004DA219  |. |8D55 E4       lea     edx, [ebp-1C]

004DA21C  |. |E8 D3DBFFFF   call    004D7DF4

004DA221  |. |8B55 E4       mov     edx, [ebp-1C]

004DA224  |. |8B45 EC       mov     eax, [ebp-14]

004DA227  |. |E8 FC96F2FF   call    00403928

004DA22C  |. |74 2A         je      short 004DA258

004DA22E  |. |8D55 E0       lea     edx, [ebp-20]

004DA231  |. |8B45 F8       mov     eax, [ebp-8]

004DA234  |. |E8 3BDCFFFF   call    004D7E74

004DA239  |. |8B45 E0       mov     eax, [ebp-20]

004DA23C  |. |8D55 E4       lea     edx, [ebp-1C]

004DA23F  |. |E8 B0DBFFFF   call    004D7DF4

004DA244  |. |8B55 E4       mov     edx, [ebp-1C]

004DA247  |. |8B45 F0       mov     eax, [ebp-10]

004DA24A  |. |E8 D996F2FF   call    00403928

004DA24F  |. |74 07         je      short 004DA258

004DA251  |. |33DB          xor     ebx, ebx

004DA253  |. |E9 8E000000   jmp     004DA2E6

004DA258  |> |B3 01         mov     bl, 1

004DA25A  |. |E9 87000000   jmp     004DA2E6

004DA25F  |> |8B45 F4       mov     eax, [ebp-C]

004DA262  |. |BA 58A44D00   mov     edx, 004DA458                    ;  jtf

004DA267  |. |E8 BC96F2FF   call    00403928

004DA26C  |. |75 78         jnz     short 004DA2E6                   ;  和“JTF”比较当然也不等。跳

004DA26E  |. |8D45 EC       lea     eax, [ebp-14]

004DA271  |. |50            push    eax

004DA272  |. |B9 20000000   mov     ecx, 20

004DA277  |. |BA 4CA44D00   mov     edx, 004DA44C                    ;  mf

004DA27C  |. |8B45 FC       mov     eax, [ebp-4]

004DA27F  |. |E8 68FDFFFF   call    004D9FEC

004DA284  |. |8D45 F0       lea     eax, [ebp-10]

004DA287  |. |50            push    eax

004DA288  |. |B9 10000000   mov     ecx, 10

004DA28D  |. |BA 4CA44D00   mov     edx, 004DA44C                    ;  mf

004DA292  |. |8B45 FC       mov     eax, [ebp-4]

004DA295  |. |E8 52FDFFFF   call    004D9FEC

004DA29A  |. |8D55 E0       lea     edx, [ebp-20]

004DA29D  |. |8B45 F8       mov     eax, [ebp-8]

004DA2A0  |. |E8 CFDBFFFF   call    004D7E74

004DA2A5  |. |8B45 E0       mov     eax, [ebp-20]

004DA2A8  |. |8D55 E4       lea     edx, [ebp-1C]

004DA2AB  |. |E8 44DBFFFF   call    004D7DF4

004DA2B0  |. |8B55 E4       mov     edx, [ebp-1C]

004DA2B3  |. |8B45 EC       mov     eax, [ebp-14]

004DA2B6  |. |E8 6D96F2FF   call    00403928

004DA2BB  |. |74 27         je      short 004DA2E4

004DA2BD  |. |8D55 E0       lea     edx, [ebp-20]

004DA2C0  |. |8B45 F8       mov     eax, [ebp-8]

004DA2C3  |. |E8 ACDBFFFF   call    004D7E74

004DA2C8  |. |8B45 E0       mov     eax, [ebp-20]

004DA2CB  |. |8D55 E4       lea     edx, [ebp-1C]

004DA2CE  |. |E8 21DBFFFF   call    004D7DF4

004DA2D3  |. |8B55 E4       mov     edx, [ebp-1C]

004DA2D6  |. |8B45 F0       mov     eax, [ebp-10]

004DA2D9  |. |E8 4A96F2FF   call    00403928

004DA2DE  |. |74 04         je      short 004DA2E4

004DA2E0  |. |33DB          xor     ebx, ebx

004DA2E2  |. |EB 02         jmp     short 004DA2E6

004DA2E4  |> |B3 01         mov     bl, 1

004DA2E6  |> \84DB          test    bl, bl

004DA2E8  |.  0F85 1A010000 jnz     004DA408

004DA2EE  |.  8B45 F4       mov     eax, [ebp-C]

004DA2F1  |.  BA 64A44D00   mov     edx, 004DA464                    ;  flm

004DA2F6  |.  E8 2D96F2FF   call    00403928

004DA2FB  |.  0F85 80000000 jnz     004DA381                         ;  和“FLM”比较，不等，跳

004DA301  |.  8D45 EC       lea     eax, [ebp-14]

004DA304  |.  50            push    eax

004DA305  |.  B9 20000000   mov     ecx, 20

004DA30A  |.  BA 70A44D00   mov     edx, 004DA470                    ;  mlm

004DA30F  |.  8B45 FC       mov     eax, [ebp-4]

004DA312  |.  E8 D5FCFFFF   call    004D9FEC

004DA317  |.  8D45 F0       lea     eax, [ebp-10]

004DA31A  |.  50            push    eax

004DA31B  |.  B9 10000000   mov     ecx, 10

004DA320  |.  BA 70A44D00   mov     edx, 004DA470                    ;  mlm

004DA325  |.  8B45 FC       mov     eax, [ebp-4]

004DA328  |.  E8 BFFCFFFF   call    004D9FEC

004DA32D  |.  8D55 E0       lea     edx, [ebp-20]

004DA330  |.  8B45 F8       mov     eax, [ebp-8]

004DA333  |.  E8 3CDBFFFF   call    004D7E74

004DA338  |.  8B45 E0       mov     eax, [ebp-20]

004DA33B  |.  8D55 E4       lea     edx, [ebp-1C]

004DA33E  |.  E8 B1DAFFFF   call    004D7DF4

004DA343  |.  8B55 E4       mov     edx, [ebp-1C]

004DA346  |.  8B45 EC       mov     eax, [ebp-14]

004DA349  |.  E8 DA95F2FF   call    00403928

004DA34E  |.  74 2A         je      short 004DA37A

004DA350  |.  8D55 E0       lea     edx, [ebp-20]

004DA353  |.  8B45 F8       mov     eax, [ebp-8]

004DA356  |.  E8 19DBFFFF   call    004D7E74

004DA35B  |.  8B45 E0       mov     eax, [ebp-20]

004DA35E  |.  8D55 E4       lea     edx, [ebp-1C]

004DA361  |.  E8 8EDAFFFF   call    004D7DF4

004DA366  |.  8B55 E4       mov     edx, [ebp-1C]

004DA369  |.  8B45 F0       mov     eax, [ebp-10]

004DA36C  |.  E8 B795F2FF   call    00403928

004DA371  |.  74 07         je      short 004DA37A

004DA373  |.  33DB          xor     ebx, ebx

004DA375  |.  E9 8E000000   jmp     004DA408

004DA37A  |>  B3 01         mov     bl, 1

004DA37C  |.  E9 87000000   jmp     004DA408

004DA381  |>  8B45 F4       mov     eax, [ebp-C]

004DA384  |.  BA 70A44D00   mov     edx, 004DA470                    ;  mlm

004DA389  |.  E8 9A95F2FF   call    00403928

004DA38E  |.  75 78         jnz     short 004DA408                   ;  和“MLM”比较

004DA390  |.  8D45 EC       lea     eax, [ebp-14]

004DA393  |.  50            push    eax

004DA394  |.  B9 20000000   mov     ecx, 20

004DA399  |.  BA 64A44D00   mov     edx, 004DA464                    ;  flm

004DA39E  |.  8B45 FC       mov     eax, [ebp-4]

004DA3A1  |.  E8 46FCFFFF   call    004D9FEC

004DA3A6  |.  8D45 F0       lea     eax, [ebp-10]

004DA3A9  |.  50            push    eax

004DA3AA  |.  B9 10000000   mov     ecx, 10

004DA3AF  |.  BA 64A44D00   mov     edx, 004DA464                    ;  flm

004DA3B4  |.  8B45 FC       mov     eax, [ebp-4]

004DA3B7  |.  E8 30FCFFFF   call    004D9FEC

004DA3BC  |.  8D55 E0       lea     edx, [ebp-20]

004DA3BF  |.  8B45 F8       mov     eax, [ebp-8]

004DA3C2  |.  E8 ADDAFFFF   call    004D7E74

004DA3C7  |.  8B45 E0       mov     eax, [ebp-20]

004DA3CA  |.  8D55 E4       lea     edx, [ebp-1C]

004DA3CD  |.  E8 22DAFFFF   call    004D7DF4

004DA3D2  |.  8B55 E4       mov     edx, [ebp-1C]

004DA3D5  |.  8B45 EC       mov     eax, [ebp-14]

004DA3D8  |.  E8 4B95F2FF   call    00403928

004DA3DD  |.  74 27         je      short 004DA406

004DA3DF  |.  8D55 E0       lea     edx, [ebp-20]

004DA3E2  |.  8B45 F8       mov     eax, [ebp-8]

004DA3E5  |.  E8 8ADAFFFF   call    004D7E74

004DA3EA  |.  8B45 E0       mov     eax, [ebp-20]

004DA3ED  |.  8D55 E4       lea     edx, [ebp-1C]

004DA3F0  |.  E8 FFD9FFFF   call    004D7DF4

004DA3F5  |.  8B55 E4       mov     edx, [ebp-1C]

004DA3F8  |.  8B45 F0       mov     eax, [ebp-10]

004DA3FB  |.  E8 2895F2FF   call    00403928
```
