
```asm
00000232: 81 7D C4 00 00 00  cmp         dword ptr [ebp-3Ch],1000000h
            01

00000239: 74 08              je          00000243

0000023B: 83 C8 FF           or          eax,0FFFFFFFFh

0000023E: E9 A3 01 00 00     jmp         000003E6

00000243: 8B 45 C0           mov         eax,dword ptr [ebp-40h]

00000246: 25 CC 00 00 00     and         eax,0CCh

0000024B: 75 05              jne         00000252

0000024D: E9 C0 00 00 00     jmp         $exit_success$19845

00000252: 8B 4D B0           mov         ecx,dword ptr [ebp-50h]

00000255: 89 4D F0           mov         dword ptr [ebp-10h],ecx

00000258: 8B 55 F0           mov         edx,dword ptr [ebp-10h]

0000025B: 0F B7 02           movzx       eax,word ptr [edx]

0000025E: 3D 4D 5A 00 00     cmp         eax,5A4Dh

00000263: 74 08              je          0000026D

00000265: 83 C8 FF           or          eax,0FFFFFFFFh

00000268: E9 79 01 00 00     jmp         000003E6

0000026D: 8B 4D F0           mov         ecx,dword ptr [ebp-10h]

00000270: 8B 55 F0           mov         edx,dword ptr [ebp-10h]

00000273: 03 51 3C           add         edx,dword ptr [ecx+3Ch]

00000276: 89 55 EC           mov         dword ptr [ebp-14h],edx

00000279: 8B 45 EC           mov         eax,dword ptr [ebp-14h]

0000027C: 81 38 50 45 00 00  cmp         dword ptr [eax],4550h

00000282: 74 08              je          0000028C

00000284: 83 C8 FF           or          eax,0FFFFFFFFh

00000287: E9 5A 01 00 00     jmp         000003E6

0000028C: 8B 4D EC           mov         ecx,dword ptr [ebp-14h]

0000028F: 83 C1 18           add         ecx,18h

00000292: 89 4D F4           mov         dword ptr [ebp-0Ch],ecx

00000295: 8B 55 F4           mov         edx,dword ptr [ebp-0Ch]

00000298: 0F B7 02           movzx       eax,word ptr [edx]

0000029B: 3D 0B 01 00 00     cmp         eax,10Bh

000002A0: 74 08              je          000002AA

000002A2: 83 C8 FF           or          eax,0FFFFFFFFh

000002A5: E9 3C 01 00 00     jmp         000003E6

000002AA: 8B 4D E0           mov         ecx,dword ptr [ebp-20h]

000002AD: 2B 4D F0           sub         ecx,dword ptr [ebp-10h]

000002B0: 89 4D FC           mov         dword ptr [ebp-4],ecx

000002B3: C7 45 D4 00 00 00  mov         dword ptr [ebp-2Ch],0
            00

000002BA: 8B 55 EC           mov         edx,dword ptr [ebp-14h]

000002BD: 0F B7 42 14        movzx       eax,word ptr [edx+14h]

000002C1: 8B 4D EC           mov         ecx,dword ptr [ebp-14h]

000002C4: 8D 54 01 18        lea         edx,[ecx+eax+18h]

000002C8: 89 55 C8           mov         dword ptr [ebp-38h],edx

000002CB: 8B 45 EC           mov         eax,dword ptr [ebp-14h]

000002CE: 0F B7 48 06        movzx       ecx,word ptr [eax+6]

000002D2: 39 4D D4           cmp         dword ptr [ebp-2Ch],ecx

000002D5: 73 33              jae         0000030A

000002D7: 8B 55 C8           mov         edx,dword ptr [ebp-38h]

000002DA: 8B 45 FC           mov         eax,dword ptr [ebp-4]

000002DD: 3B 42 0C           cmp         eax,dword ptr [edx+0Ch]

000002E0: 72 26              jb          00000308

000002E2: 8B 4D C8           mov         ecx,dword ptr [ebp-38h]

000002E5: 8B 51 0C           mov         edx,dword ptr [ecx+0Ch]

000002E8: 8B 45 C8           mov         eax,dword ptr [ebp-38h]

000002EB: 03 50 08           add         edx,dword ptr [eax+8]

000002EE: 39 55 FC           cmp         dword ptr [ebp-4],edx

000002F1: 73 15              jae         00000308

000002F3: 8B 4D C8           mov         ecx,dword ptr [ebp-38h]

000002F6: 8B 51 24           mov         edx,dword ptr [ecx+24h]

000002F9: 81 E2 00 00 00 80  and         edx,80000000h

000002FF: 74 07              je          00000308

00000301: 33 C0              xor         eax,eax

00000303: E9 DE 00 00 00     jmp         000003E6

00000308: EB 08              jmp         $exit_success$19845

0000030A: 83 C8 FF           or          eax,0FFFFFFFFh

0000030D: E9 D4 00 00 00     jmp         000003E6

$exit_success$19845:

00000312: 6A 01              push        1

00000314: 68 00 00 00 00     push        offset _lModifying

00000319: FF 15 00 00 00 00  call        dword ptr [__imp__InterlockedExchange@8]

0000031F: 85 C0              test        eax,eax

00000321: 74 0A              je          0000032D

00000323: B8 01 00 00 00     mov         eax,1

00000328: E9 B9 00 00 00     jmp         000003E6

0000032D: A1 00 00 00 00     mov         eax,dword ptr [_nValidPages]

00000332: 89 45 D0           mov         dword ptr [ebp-30h],eax

00000335: EB 09              jmp         00000340

00000337: 8B 4D D0           mov         ecx,dword ptr [ebp-30h]

0000033A: 83 E9 01           sub         ecx,1

0000033D: 89 4D D0           mov         dword ptr [ebp-30h],ecx

00000340: 83 7D D0 00        cmp         dword ptr [ebp-30h],0

00000344: 7E 13              jle         00000359

00000346: 8B 55 D0           mov         edx,dword ptr [ebp-30h]

00000349: 8B 04 95 FC FF FF  mov         eax,dword ptr [edx*4-4]
            FF

00000350: 3B 45 CC           cmp         eax,dword ptr [ebp-34h]

00000353: 75 02              jne         00000357

00000355: EB 02              jmp         00000359

00000357: EB DE              jmp         00000337

00000359: 83 7D D0 00        cmp         dword ptr [ebp-30h],0

0000035D: 75 75              jne         000003D4

0000035F: 83 3D 00 00 00 00  cmp         dword ptr [_nValidPages],0Fh
            0F

00000366: 7E 09              jle         00000371

00000368: C7 45 A4 0F 00 00  mov         dword ptr [ebp-5Ch],0Fh
            00

0000036F: EB 09              jmp         0000037A

00000371: 8B 0D 00 00 00 00  mov         ecx,dword ptr [_nValidPages]

00000377: 89 4D A4           mov         dword ptr [ebp-5Ch],ecx

0000037A: 8B 55 A4           mov         edx,dword ptr [ebp-5Ch]

0000037D: 89 55 D0           mov         dword ptr [ebp-30h],edx

00000380: C7 45 D8 00 00 00  mov         dword ptr [ebp-28h],0
            00

00000387: EB 09              jmp         00000392

00000389: 8B 45 D8           mov         eax,dword ptr [ebp-28h]

0000038C: 83 C0 01           add         eax,1

0000038F: 89 45 D8           mov         dword ptr [ebp-28h],eax

00000392: 8B 4D D8           mov         ecx,dword ptr [ebp-28h]

00000395: 3B 4D D0           cmp         ecx,dword ptr [ebp-30h]

00000398: 7F 22              jg          000003BC

0000039A: 8B 55 D8           mov         edx,dword ptr [ebp-28h]

0000039D: 8B 04 95 00 00 00  mov         eax,dword ptr [edx*4]
            00

000003A4: 89 45 E8           mov         dword ptr [ebp-18h],eax

000003A7: 8B 4D D8           mov         ecx,dword ptr [ebp-28h]

000003AA: 8B 55 CC           mov         edx,dword ptr [ebp-34h]

000003AD: 89 14 8D 00 00 00  mov         dword ptr [ecx*4],edx
            00

000003B4: 8B 45 E8           mov         eax,dword ptr [ebp-18h]

000003B7: 89 45 CC           mov         dword ptr [ebp-34h],eax

000003BA: EB CD              jmp         00000389

000003BC: 83 3D 00 00 00 00  cmp         dword ptr [_nValidPages],10h
            10

000003C3: 7D 0F              jge         000003D4

000003C5: 8B 0D 00 00 00 00  mov         ecx,dword ptr [_nValidPages]

000003CB: 83 C1 01           add         ecx,1

000003CE: 89 0D 00 00 00 00  mov         dword ptr [_nValidPages],ecx

000003D4: 6A 00              push        0

000003D6: 68 00 00 00 00     push        offset _lModifying

000003DB: FF 15 00 00 00 00  call        dword ptr [__imp__InterlockedExchange@8]

000003E1: B8 01 00 00 00     mov         eax,1

000003E6: 8B E5              mov         esp,ebp

000003E8: 5D                 pop         ebp

000003E9: C3                 ret
```

## Summary
