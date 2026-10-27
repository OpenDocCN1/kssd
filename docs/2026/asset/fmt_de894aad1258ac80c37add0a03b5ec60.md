
```
 0041371D: 803E00                    cmp byte ptr [esi],00
 00413720: 7403                      jz 00413725
 00413722: 46                        inc esi
 00413723: EBF8                      jmp 0041371D
 00413725: 46                        inc esi
 00413726: EBBF                      jmp 004136E7
 00413728: 61                        popad
 00413729: 8B8581BA4000              mov eax,dword ptr [ebp+0040BA81]
 0041372F: C3                        ret
 00413730: 0000                      add byte ptr [eax],al
 00413732: 0000                      add byte ptr [eax],al
 00413734: 0100                      add dword ptr [eax],eax
 00413736: 0000                      add byte ptr [eax],al
 00413738: E3BA                      jecxz 004136F4
 0041373A: 40                        inc eax
 0041373B: 009FBB40006B              add byte ptr [edi+6B0040BB],bl
 00413741: 65726E                    jb 004137B2
 00413744: 656C                      insb es:[edi],dx
 00413746: 3332                      xor esi,dword ptr [edx]
 00413748: 2E646C                    insb es:[edi],dx
 0041374B: 6C                        insb es:[edi],dx
 0041374C: 00D3                      add bl,dl
 0041374E: BB400000BC                mov ebx,BC000040
 00413753: 40                        inc eax
 00413754: 007573                    add byte ptr [ebp+73],dh
 00413757: 657233                    jb 0041378D
 0041375A: 322E                      xor ch,byte ptr [esi]
 0041375C: 646C                      insb es:[edi],dx
 0041375E: 6C                        insb es:[edi],dx
 0041375F: 0014BC                    add byte ptr [esp+edi\*4],dl
 00413762: 40                        inc eax
 00413763: 0072BC                    add byte ptr [edx-44],dh
 00413766: 40                        inc eax
 00413767: 006164                    add byte ptr [ecx+64],ah
 0041376A: 7661                      jbe 004137CD
 0041376C: 7069                      jo 004137D7
 0041376E: 3332                      xor esi,dword ptr [edx]
 00413770: 2E646C                    insb es:[edi],dx
 00413773: 6C                        insb es:[edi],dx
 00413774: 008EBC4000B8              add byte ptr [esi-47FFBF44],cl
 0041377A: BC40006469                mov esp,69640040
 0041377F: 61                        popad
 00413780: 6C                        insb es:[edi],dx
 00413781: 2E646C                    insb es:[edi],dx
 00413784: 6C                        insb es:[edi],dx
 00413785: 0000                      add byte ptr [eax],al
 00413787: 0000                      add byte ptr [eax],al
 00413789: 0000                      add byte ptr [eax],al
 0041378B: 0000                      add byte ptr [eax],al
 0041378D: 0000                      add byte ptr [eax],al
 0041378F: 0020                      add byte ptr [eax],ah
 00413791: 2020                      and byte ptr [eax],ah
 00413793: 2020                      and byte ptr [eax],ah
 00413795: 3237                      xor dh,byte ptr [edi]
 00413797: 3133                      xor dword ptr [ebx],esi
 00413799: 3800                      cmp byte ptr [eax],al
 0041379B: 0000                      add byte ptr [eax],al
 0041379D: 0000                      add byte ptr [eax],al
 0041379F: 0000                      add byte ptr [eax],al
 00413780: 6C                        insb es:[edi],dx
 00413781: 2E646C                    insb es:[edi],dx
 00413784: 6C                        insb es:[edi],dx
 00413785: 0000                      add byte ptr [eax],al
 00413787: 0000                      add byte ptr [eax],al
 00413789: 0000                      add byte ptr [eax],al
 0041378B: 0000                      add byte ptr [eax],al
 0041378D: 0000                      add byte ptr [eax],al
 0041378F: 0000                      add byte ptr [eax],al
 00413791: 0000                      add byte ptr [eax],al
 00413793: 0000                      add byte ptr [eax],al
 00413795: 004765                    add byte ptr [edi+65],al
 00413798: 7453                      jz 004137ED
 0041379A: 7973                      jns 0041380F
 0041379C: 7465                      jz 00413803
 0041379E: 6D                        insd es:[edi],dx
 0041379F: 54                        push esp
 004137A0: 696D6500437265            imul ebp,dword ptr [ebp+65],65724300
 004137A7: 61                        popad
 004137A8: 7465                      jz 0041380F
 004137AA: 46                        inc esi
 004137AB: 696C654100577269          imul ebp,dword ptr [ebp+41],69725700
 004137B3: 7465                      jz 0041381A
 004137B5: 46                        inc esi
 004137B6: 696C6500436C6F73          imul ebp,dword ptr [ebp],736F6C43
 004137BE: 6548                      dec eax
 004137C0: 61                        popad
 004137C1: 6E                        outsb dx,ds:[esi]
 004137C2: 646C                      insb es:[edi],dx
 004137C4: 65004372                  add byte ptr gs:[ebx+72],al
 004137C8: 6561                      popad
 004137CA: 7465                      jz 00413831
 004137CC: 50                        push eax
 004137CD: 726F                      jb 0041383E
 004137CF: 636573                    arpl word ptr [ebp+73],sp
 004137D2: 7341                      jnb 00413815
 004137D4: 004765                    add byte ptr [edi+65],al
 004137D7: 7445                      jz 0041381E
 004137D9: 7869                      js 00413844
 004137DB: 7443                      jz 00413820
 004137DD: 6F                        outsd dx,ds:[esi]
 004137DE: 646550                    push eax
 004137E1: 726F                      jb 00413852
 004137E3: 636573                    arpl word ptr [ebp+73],sp
 004137E6: 7300                      jnb 004137E8
 004137E8: 47                        inc edi
 004137E9: 657444                    jz 00413830
 004137EC: 69736B46726565            imul esi,dword ptr [ebx+6B],65657246
 004137F3: 53                        push ebx
 004137F4: 7061                      jo 00413857
 004137F6: 636545                    arpl word ptr [ebp+45],sp
 004137F9: 7841                      js 0041383C
 004137FB: 0044656C                  add byte ptr [ebp+6C],al
 004137FF: 657465                    jz 00413867
 00413802: 46                        inc esi
 00413803: 696C654100476574          imul ebp,dword ptr [ebp+41],74654700
 0041380B: 56                        push esi
 0041380C: 6F                        outsd dx,ds:[esi]
 0041380D: 6C                        insb es:[edi],dx
 0041380E: 756D                      jnz 0041387D
 00413810: 6549                      dec ecx
 00413812: 6E                        outsb dx,ds:[esi]
 00413813: 666F                      outsw dx,ds:[esi]
 00413815: 726D                      jb 00413884
 00413817: 61                        popad
 00413818: 7469                      jz 00413883
 0041381A: 6F                        outsd dx,ds:[esi]
 0041381B: 6E                        outsb dx,ds:[esi]
 0041381C: 41                        inc ecx
 0041381D: 004765                    add byte ptr [edi+65],al
 00413820: 744D                      jz 0041386F
 00413822: 6F                        outsd dx,ds:[esi]
 00413823: 64756C                    jnz 00413892
 00413826: 6548                      dec eax
 00413828: 61                        popad
 00413829: 6E                        outsb dx,ds:[esi]
 0041382A: 646C                      insb es:[edi],dx
 0041382C: 6541                      inc ecx
 0041382E: 005265                    add byte ptr [edx+65],dl
 00413831: 61                        popad
 00413832: 6450                      push eax
 00413834: 726F                      jb 004138A5
 00413836: 636573                    arpl word ptr [ebp+73],sp
 00413839: 734D                      jnb 00413888
 0041383B: 656D                      insd es:[edi],dx
 0041383D: 6F                        outsd dx,ds:[esi]
```
