
```
00013E9F: 0000                      add byte ptr [eax],al  
00013EA1: 0000                      add byte ptr [eax],al  
00013EA3: 0000                      add byte ptr [eax],al  
00013EA5: 0000                      add byte ptr [eax],al  
00013EA7: 0000                      add byte ptr [eax],al  
00013EA9: 0000                      add byte ptr [eax],al  
00013EAB: 0000                      add byte ptr [eax],al  
00013EAD: 0003                      add byte ptr [ebx],al  
00013EAF: 0000                      add byte ptr [eax],al  
00013EB1: 0029                      add byte ptr [ecx],ch  
00013EB3: 0000                      add byte ptr [eax],al  
00013EB5: 002A                      add byte ptr [edx],ch  
00013EB7: 00494E                   add byte ptr [ecx+4E],cl  
00013EBA: 5F                       pop edi  
00013EBB: 43                       inc ebx  
00013EBC: 4F                       dec edi  
00013EBD: 44                       inc esp  
00013EBE: 45                       inc ebp  
00013EBF: 0000                      add byte ptr [eax],al  
00013EC1: 0000                      add byte ptr [eax],al  
00013EC3: 0020                      add byte ptr [eax],ah  
00013EC5: 2020                      and byte ptr [eax],ah  
00013EC7: 2020                      and byte ptr [eax],ah  
00013EC9: 2020                      and byte ptr [eax],ah  
00013ECB: 2020                      and byte ptr [eax],ah  
00013ECD: 2020                      and byte ptr [eax],ah  
00013ECF: B1BE                      mov cl,BE  
00013ED1: C8EDBCFE                 enter BCED,FE  
00013ED5: D3C9                      ror ecx,cl  
00013ED7: 20BBC3D3B076             and byte ptr [ebx+76B0D3C3],bh  
00013EDD: 312E                      xor dword ptr [esi],ebp  
00013EDF: 3520CEB4D7              xor eax,D7B4CE20  
00013EE4: A2B2E1B0E6              mov [E6B0E1B2],al  
00013EE9: B1BE                      mov cl,BE  
00013EEB: BCD3C3DC20              mov esp,20DCC3D3  
00013EF0: 2020                      and byte ptr [eax],ah  
00013EF2: 2020                      and byte ptr [eax],ah  
00013EF4: 2038                      and byte ptr [eax],bh  
00013EF6: 3530343400              xor eax,00343430  
00013EFB: 0000                      add byte ptr [eax],al  
00013EFD: 0000                      add byte ptr [eax],al  
00013EFF: 0000                      add byte ptr [eax],al  
00013EE0: 20CE                      and dh,cl  
00013EE2: B4D7                      mov ah,D7  
00013EE4: A2B2E1B0E6              mov [E6B0E1B2],al  
00013EE9: B1BE                      mov cl,BE  
00013EEB: BCD3C3DC20              mov esp,20DCC3D3  
00013EF0: 2020                      and byte ptr [eax],ah  
00013EF2: 2020                      and byte ptr [eax],ah  
00013EF4: 2020                      and byte ptr [eax],ah  
00013EF6: 2020                      and byte ptr [eax],ah  
00013EF8: 2020                      and byte ptr [eax],ah  
00013EFA: 0D0A0D0A20              or eax,200A0D0A  
00013EFF: 54                       push esp  
00013F00: 6869732070              push 70207369  
00013F05: 726O                      jb 00013F76  
00013F07: 677261                   jb 00013F6B  
00013F0A: 6D                       insd es:[edi],dx  
00013F0B: 207072                   and byte ptr [eax+72],dh  
00013F0E: 6F                       outsd dx,ds:[esi]  
00013F0F: 7465                    jz 00013F76  
00013F11: 63746564                arpl word ptr [ebp+64],si  
00013F15: 206279                   and byte ptr [edx+79],ah  
00013F18: 20444250                and byte ptr [edx+eax*2+50],al  
00013F1C: 45                       inc ebp  
00013F1D: 207631                   and byte ptr [esi+31],dh  
00013F20: 2E352020556E            xor eax,6E552020  
00013F26: 7265                    jb 00013F8D  
00013F28: 6769737465726564        imul esi,dword ptr [bp+di+74],64657265  
00013F30: 207665                   and byte ptr [esi+65],dh  
00013F33: 7273                    jb 00013FA8  
00013F35: 696F6E0D0A0D0A          imul ebp,dword ptr [edi+6E],0A0D0A0D  
00013F3C: 0D0A0D0A20              or eax,200A0D0A  
00013F41: 48                       dec eax  
00013F42: 6F                       outsd dx,ds:[esi]  
00013F43: 6M                       insd es:[edi],dx  
00013F44: 657061                   jo 00013FA8  
00013F47: 67653A20                 cmp ah,byte ptr gs:[bx+si]  
00013F4B: 687474703A              push 3A707474  
00013F50: 2F                       das  
00013F51: 2F                       das  
00013F52: 667364                   jnb 00013FB9  
00013F55: 622E                     bound ebp,dword ptr [esi]  
00013F57: 7965                     jns 00013FBE  
00013F59: 61                       popad   
00013F5A: 682E6E6574              push 74656E2E  
00013F5F: 2020                      and byte ptr [eax],ah  
00013F61: 2020                      and byte ptr [eax],ah  
00013F63: 45                       inc ebp  
00013F64: 2D6D61696C              sub eax,6C69616D  
00013F69: 3A20                     cmp ah,byte ptr [eax]  
00013F6B: 44                       inc esp  
00013F6C: 2E42                     inc edx  
00013F6E: 6F                       outsd dx,ds:[esi]  
00013F6F: 7940                     jns 00013FB1  
00013F71: 3132                     xor dword ptr [edx],esi  
00013F73: 362E636F6D              arpl word ptr cs:[edi+6D],bp  
00013F78: 200D0A007C7C           and byte ptr [7C7C000A],cl  
00013F7E: 00D0                     add al,dl  
00013F80: 0000                      add byte ptr [eax],al  
00013F82: CC                       int 3  
00013F83: 1000                      adc byte ptr [eax],al  
00013F85: 0000                      add byte ptr [eax],al  
00013F87: 0000                      add byte ptr [eax],al  
00013F89: 0000                      add byte ptr [eax],al  
00013F8B: 1000                      adc byte ptr [eax],al  
00013F8D: 0000                      add byte ptr [eax],al  
00013F8F: 40                       inc eax  
00013F90: 0000                      add byte ptr [eax],al  
00013F92: 005000                   add byte ptr [eax],dl  
00013F95: 0000                      add byte ptr [eax],al  
00013F97: 1000                      adc byte ptr [eax],al  
00013F99: 0000                      add byte ptr [eax],al  
00013F9B: 60                       pushad   
00013F9C: 0000                      add byte ptr [eax],al  
00013F9E: 0010                     add byte ptr [eax],dl  
00013FA0: 0000                      add byte ptr [eax],al  
00013FA2: 00C0                     add al,al  
00013FA4: 0000                      add byte ptr [eax],al  
00013FA6: 0010                     add byte ptr [eax],dl  
00013FA8: 0000                      add byte ptr [eax],al  
00013FAA: 0000                      add byte ptr [eax],al  
00013FAC: 0000                      add byte ptr [eax],al  
00013FAE: 0000                      add byte ptr [eax],al  
00013FB0: 0000                      add byte ptr [eax],al  
00013FB2: 0000                      add byte ptr [eax],al  
00013FB4: 0000                      add byte ptr [eax],al  
00013FB6: 0000                      add byte ptr [eax],al  
00013FB8: 0000                      add byte ptr [eax],al  
00013FBA: 0000                      add byte ptr [eax],al  
00013FBC: 0000                      add byte ptr [eax],al  
```
