
```
 000147FF: 657465                    jz 00014867

 00014802: 46                        inc esi

 00014803: 696C654100476574          imul ebp,dword ptr [ebp+41],74654700

 0001480B: 56                        push esi

 0001480C: 6F                        outsd dx,ds:[esi]

 0001480D: 6C                        insb es:[edi],dx

 0001480E: 756D                      jnz 0001487D

 00014810: 6549                      dec ecx

 00014812: 6E                        outsb dx,ds:[esi]

 00014813: 666F                      outsw dx,ds:[esi]

 00014815: 726D                      jb 00014884

 00014817: 61                        popad

 00014818: 7469                      jz 00014883

 0001481A: 6F                        outsd dx,ds:[esi]

 0001481B: 6E                        outsb dx,ds:[esi]

 0001481C: 41                        inc ecx

 0001481D: 004765                    add byte ptr [edi+65],al

 00014820: 744D                      jz 0001486F

 00014822: 6F                        outsd dx,ds:[esi]

 00014823: 64756C                    jnz 00014892

 00014826: 6548                      dec eax

 00014828: 61                        popad

 00014829: 6E                        outsb dx,ds:[esi]

 0001482A: 646C                      insb es:[edi],dx

 0001482C: 6541                      inc ecx

 0001482E: 005265                    add byte ptr [edx+65],dl

 00014831: 61                        popad

 00014832: 6450                      push eax

 00014834: 726F                      jb 000148A5

 00014836: 636573                    arpl word ptr [ebp+73],sp

 00014839: 734D                      jnb 00014888

 0001483B: 656D                      insd es:[edi],dx

 0001483D: 6F                        outsd dx,ds:[esi]

 0001483E: 7279                      jb 000148B9

 00014840: 005669                    add byte ptr [esi+69],dl

 00014843: 7274                      jb 000148B9

 00014845: 7561                      jnz 000148A8

 00014847: 6C                        insb es:[edi],dx

 00014848: 41                        inc ecx

 00014849: 6C                        insb es:[edi],dx

 0001484A: 6C                        insb es:[edi],dx

 0001484B: 6F                        outsd dx,ds:[esi]

 0001484C: 6300                      arpl word ptr [eax],ax

 0001484E: 0000                      add byte ptr [eax],al

 00014850: 0000                      add byte ptr [eax],al

 00014852: 0000                      add byte ptr [eax],al

 00014854: 0000                      add byte ptr [eax],al

 00014856: 0000                      add byte ptr [eax],al

 00014858: 0000                      add byte ptr [eax],al

 0001485A: 0000                      add byte ptr [eax],al

 0001485C: 0000                      add byte ptr [eax],al

 0001485E: 0000                      add byte ptr [eax],al

 00014860: 0000                      add byte ptr [eax],al

 00014862: 0000                      add byte ptr [eax],al

 00014864: 0000                      add byte ptr [eax],al

 00014866: 0000                      add byte ptr [eax],al

 00014868: 0000                      add byte ptr [eax],al

 0001486A: 0000                      add byte ptr [eax],al

 0001486C: 0000                      add byte ptr [eax],al

 0001486E: 0000                      add byte ptr [eax],al

 00014870: 0000                      add byte ptr [eax],al

 00014872: 0000                      add byte ptr [eax],al

 00014874: 0000                      add byte ptr [eax],al

 00014876: 0000                      add byte ptr [eax],al

 00014878: 0000                      add byte ptr [eax],al

 0001487A: 0000                      add byte ptr [eax],al

 0001487C: 0000                      add byte ptr [eax],al

 0001487E: 0000                      add byte ptr [eax],al

 00014880: 0000                      add byte ptr [eax],al

 00014882: 0000                      add byte ptr [eax],al

 00014884: 0000                      add byte ptr [eax],al

 00014886: 4D                        dec ebp

 00014887: 657373                    jnb 000148FD

 0001488A: 61                        popad

 0001488B: 676542                    inc edx

 0001488E: 6F                        outsd dx,ds:[esi]

 0001488F: 7841                      js 000148D2

 00014891: 007773                    add byte ptr [edi+73],dh

 00014894: 7072                      jo 00014908

 00014896: 696E7466410053            imul ebp,dword ptr [esi+74],53004166

 0001489D: 657454                    jz 000148F4

 000148A0: 696D6572004B69            imul ebp,dword ptr [ebp+65],694B0072

 000148A7: 6C                        insb es:[edi],dx

 000148A8: 6C                        insb es:[edi],dx

 000148A9: 54                        push esp

 000148AA: 696D6572000000            imul ebp,dword ptr [ebp+65],00000072

 000148B1: 0000                      add byte ptr [eax],al

 000148B3: 0000                      add byte ptr [eax],al

 000148B5: 0000                      add byte ptr [eax],al

 000148B7: 0000                      add byte ptr [eax],al

 000148B9: 0000                      add byte ptr [eax],al

 000148BB: 0000                      add byte ptr [eax],al

 000148BD: 0000                      add byte ptr [eax],al

 000148BF: 0000                      add byte ptr [eax],al

 000148C1: 0000                      add byte ptr [eax],al

 000148C3: 0000                      add byte ptr [eax],al

 000148C5: 0000                      add byte ptr [eax],al

 000148C7: 52                        push edx

 000148C8: 65674F                    dec edi

 000148CB: 7065                      jo 00014932

 000148CD: 6E                        outsb dx,ds:[esi]

 000148CE: 4B                        dec ebx

 000148CF: 657945                    jns 00014917

 000148D2: 7841                      js 00014915

 000148D4: 005265                    add byte ptr [edx+65],dl

 000148D7: 6743                      inc ebx

 000148D9: 7265                      jb 00014940

 000148DB: 61                        popad

 000148DC: 7465                      jz 00014943

 000148DE: 4B                        dec ebx

 000148DF: 657945                    jns 00014927

 000148E2: 7841                      js 00014925

 000148E4: 005265                    add byte ptr [edx+65],dl

 000148E7: 6743                      inc ebx

 000148E9: 6C                        insb es:[edi],dx

 000148EA: 6F                        outsd dx,ds:[esi]

 000148EB: 7365                      jnb 00014952

 000148ED: 4B                        dec ebx

 000148EE: 657900                    jns 000148F1

 000148F1: 52                        push edx

 000148F2: 656751                    push ecx

 000148F5: 7565                      jnz 0001495C

 000148F7: 7279                      jb 00014972

 000148F9: 56                        push esi

 000148FA: 61                        popad

 000148FB: 6C                        insb es:[edi],dx

 000148FC: 7565                      jnz 00014963

 000148FE: 45                        inc ebp

 000148FF: 7841                      js 00014942

 00014901: 005265                    add byte ptr [edx+65],dl

 00014904: 6753                      push ebx

 00014906: 657456                    jz 0001495F

 00014909: 61                        popad

 0001490A: 6C                        insb es:[edi],dx

 0001490B: 7565                      jnz 00014972

 0001490D: 45                        inc ebp

 0001490E: 7841                      js 00014951

 00014910: 005265                    add byte ptr [edx+65],dl

 00014913: 6744                      inc esp

 00014915: 656C                      insb es:[edi],dx

 00014917: 657465                    jz 0001497F

 0001491A: 56                        push esi

 0001491B: 61                        popad

 0001491C: 6C                        insb es:[edi],dx

 0001491D: 7565                      jnz 00014984

 0001491F: 41                        inc ecx

 00014920: 0000                      add byte ptr [eax],al
```
