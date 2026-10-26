
```
0043D360   .  68 01030080   push    80000301

0043D365   .  6A 00         push    0

0043D367   .  68 01000000   push    1

0043D36C   .  68 01030080   push    80000301

0043D371   .  6A 00         push    0

0043D373   .  68 08000000   push    8

0043D378   .  68 04000080   push    80000004

0043D37D   .  6A 00         push    0

0043D37F   .  8B45 BC       mov     eax, dword ptr [ebp-44]

0043D382   .  85C0          test    eax, eax

0043D384   .  75 05         jnz     short 0043D38B

0043D386   .  B8 623E5100   mov     eax, 00513E62

0043D38B   >  50            push    eax

0043D38C   .  68 03000000   push    3

0043D391   .  BB 10844500   mov     ebx, 00458410

0043D396   .  E8 3AA70100   call    00457AD5

0043D39B   .  83C4 28       add     esp, 28

0043D39E   .  8945 B8       mov     dword ptr [ebp-48], eax

0043D3A1   .  8B5D BC       mov     ebx, dword ptr [ebp-44]

0043D3A4   .  85DB          test    ebx, ebx

0043D3A6   .  74 09         je      short 0043D3B1

0043D3A8   .  53            push    ebx

0043D3A9   .  E8 33A70100   call    00457AE1

0043D3AE   .  83C4 04       add     esp, 4

0043D3B1   >  68 04000080   push    80000004

0043D3B6   .  6A 00         push    0

0043D3B8   .  8B45 B8       mov     eax, dword ptr [ebp-48]

0043D3BB   .  85C0          test    eax, eax

0043D3BD   .  75 05         jnz     short 0043D3C4

0043D3BF   .  B8 623E5100   mov     eax, 00513E62

0043D3C4   >  50            push    eax

0043D3C5   .  68 01000000   push    1

0043D3CA   .  BB C0894500   mov     ebx, 004589C0

0043D3CF   .  E8 01A70100   call    00457AD5

0043D3D4   .  83C4 10       add     esp, 10

0043D3D7   .  8945 B0       mov     dword ptr [ebp-50], eax

0043D3DA   .  8955 B4       mov     dword ptr [ebp-4C], edx

0043D3DD   .  8B5D B8       mov     ebx, dword ptr [ebp-48]

0043D3E0   .  85DB          test    ebx, ebx

0043D3E2   .  74 09         je      short 0043D3ED

0043D3E4   .  53            push    ebx

0043D3E5   .  E8 F7A60100   call    00457AE1

0043D3EA   .  83C4 04       add     esp, 4

0043D3ED   >  DD45 B0       fld     qword ptr [ebp-50]

0043D3F0   .  E8 104DFCFF   call    00402105

0043D3F5   .  8945 F0       mov     dword ptr [ebp-10], eax

0043D3F8   .  6A 01         push    1

0043D3FA   .  FF75 F0       push    dword ptr [ebp-10]

0043D3FD   .  E8 897B0000   call    00444F8B

0043D402   .  8945 B4       mov     dword ptr [ebp-4C], eax

0043D405   .  DB45 B4       fild    dword ptr [ebp-4C]

0043D408   .  DD5D B4       fstp    qword ptr [ebp-4C]

0043D40B   .  DD45 B4       fld     qword ptr [ebp-4C]

0043D40E   .  DB45 F0       fild    dword ptr [ebp-10]

0043D411   .  DD5D AC       fstp    qword ptr [ebp-54]

0043D414   .  DC4D AC       fmul    qword ptr [ebp-54]

0043D417   .  DB45 0C       fild    dword ptr [ebp+C]

0043D41A   .  DD5D A4       fstp    qword ptr [ebp-5C]

0043D41D   .  DC4D A4       fmul    qword ptr [ebp-5C]

0043D420   .  DD5D 9C       fstp    qword ptr [ebp-64]

0043D423   .  68 01060080   push    80000601

0043D428   .  FF75 A0       push    dword ptr [ebp-60]

0043D42B   .  FF75 9C       push    dword ptr [ebp-64]

0043D42E   .  68 01000000   push    1

0043D433   .  BB 20964500   mov     ebx, 00459620

0043D438   .  E8 98A60100   call    00457AD5

0043D43D   .  83C4 10       add     esp, 10

0043D440   .  8945 98       mov     dword ptr [ebp-68], eax

0043D443   .  8B45 98       mov     eax, dword ptr [ebp-68]

0043D446   .  50            push    eax

0043D447   .  8B5D D0       mov     ebx, dword ptr [ebp-30]

0043D44A   .  85DB          test    ebx, ebx

0043D44C   .  74 09         je      short 0043D457

0043D44E   .  53            push    ebx

0043D44F   .  E8 8DA60100   call    00457AE1

0043D454   .  83C4 04       add     esp, 4

0043D457   >  58            pop     eax

0043D458   .  8945 D0       mov     dword ptr [ebp-30], eax

0043D45B   .  68 02000080   push    80000002

0043D460   .  6A 00         push    0

0043D462   .  68 00000000   push    0

0043D467   .  6A 00         push    0

0043D469   .  6A 00         push    0

0043D46B   .  6A 00         push    0

0043D46D   .  68 04000080   push    80000004

0043D472   .  6A 00         push    0

0043D474   .  8B45 D0       mov     eax, dword ptr [ebp-30]

0043D477   .  85C0          test    eax, eax

0043D479   .  75 05         jnz     short 0043D480

0043D47B   .  B8 623E5100   mov     eax, 00513E62

0043D480   >  50            push    eax

0043D481   .  68 04000080   push    80000004

0043D486   .  6A 00         push    0

0043D488   .  8B5D 08       mov     ebx, dword ptr [ebp+8]

0043D48B   .  8B03          mov     eax, dword ptr [ebx]

0043D48D   .  85C0          test    eax, eax

0043D48F   .  75 05         jnz     short 0043D496

0043D491   .  B8 623E5100   mov     eax, 00513E62

0043D496   >  50            push    eax

0043D497   .  68 04000000   push    4

0043D49C   .  BB 70864500   mov     ebx, 00458670

0043D4A1   .  E8 2FA60100   call    00457AD5

0043D4A6   .  83C4 34       add     esp, 34

0043D4A9   .  8945 B8       mov     dword ptr [ebp-48], eax

0043D4AC   .  837D B8 FF    cmp     dword ptr [ebp-48], -1

0043D4B0   .  0F84 03000000 je      0043D4B9

0043D4B6   .  FF45 F8       inc     dword ptr [ebp-8]

0043D4B9   >  68 01030080   push    80000301

0043D4BE   .  6A 00         push    0

0043D4C0   .  FF75 F4       push    dword ptr [ebp-C]

0043D4C3   .  68 01000000   push    1

0043D4C8   .  BB 20964500   mov     ebx, 00459620

0043D4CD   .  E8 03A60100   call    00457AD5

0043D4D2   .  83C4 10       add     esp, 10

0043D4D5   .  8945 BC       mov     dword ptr [ebp-44], eax

0043D4D8   .  68 01030080   push    80000301

0043D4DD   .  6A 00         push    0

0043D4DF   .  68 01000000   push    1

0043D4E4   .  68 01030080   push    80000301

0043D4E9   .  6A 00         push    0

0043D4EB   .  68 09000000   push    9

0043D4F0   .  68 04000080   push    80000004

0043D4F5   .  6A 00         push    0

0043D4F7   .  8B45 BC       mov     eax, dword ptr [ebp-44]

0043D4FA   .  85C0          test    eax, eax

0043D4FC   .  75 05         jnz     short 0043D503

0043D4FE   .  B8 623E5100   mov     eax, 00513E62

0043D503   >  50            push    eax

0043D504   .  68 03000000   push    3

0043D509   .  BB 10844500   mov     ebx, 00458410

0043D50E   .  E8 C2A50100   call    00457AD5

0043D513   .  83C4 28       add     esp, 28

0043D516   .  8945 B8       mov     dword ptr [ebp-48], eax

0043D519   .  8B5D BC       mov     ebx, dword ptr [ebp-44]

0043D51C   .  85DB          test    ebx, ebx

0043D51E   .  74 09         je      short 0043D529

0043D520   .  53            push    ebx

0043D521   .  E8 BBA50100   call    00457AE1

0043D526   .  83C4 04       add     esp, 4

0043D529   >  68 04000080   push    80000004

0043D52E   .  6A 00         push    0

0043D530   .  8B45 B8       mov     eax, dword ptr [ebp-48]

0043D533   .  85C0          test    eax, eax

0043D535   .  75 05         jnz     short 0043D53C

0043D537   .  B8 623E5100   mov     eax, 00513E62

0043D53C   >  50            push    eax

0043D53D   .  68 01000000   push    1

0043D542   .  BB C0894500   mov     ebx, 004589C0

0043D547   .  E8 89A50100   call    00457AD5

0043D54C   .  83C4 10       add     esp, 10

0043D54F   .  8945 B0       mov     dword ptr [ebp-50], eax

0043D552   .  8955 B4       mov     dword ptr [ebp-4C], edx

0043D555   .  8B5D B8       mov     ebx, dword ptr [ebp-48]

0043D558   .  85DB          test    ebx, ebx

0043D55A   .  74 09         je      short 0043D565

0043D55C   .  53            push    ebx

0043D55D   .  E8 7FA50100   call    00457AE1

0043D562   .  83C4 04       add     esp, 4
```
