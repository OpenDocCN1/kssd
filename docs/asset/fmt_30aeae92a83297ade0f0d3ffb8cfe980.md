
```
 00412EF4: 2032                      and byte ptr [edx],dh  

 00412EF6: 37                        aaa  

 00412EF7: 3133                      xor dword ptr [ebx],esi  

 00412EF9: 3800                      cmp byte ptr [eax],al  

 00412EFB: 0000                      add byte ptr [eax],al  

 00412EFD: 0000                      add byte ptr [eax],al  

 00412EFF: 0000                      add byte ptr [eax],al  

 00412EE0: 20CE                      and dh,cl  

 00412EE2: B4D7                      mov ah,D7  

 00412EE4: A2B2E1B0E6                mov [E6B0E1B2],al  

 00412EE9: B1BE                      mov cl,BE  

 00412EEB: BCD3C3DC20                mov esp,20DCC3D3  

 00412EF0: 2020                      and byte ptr [eax],ah  

 00412EF2: 2020                      and byte ptr [eax],ah  

 00412EF4: 2020                      and byte ptr [eax],ah  

 00412EF6: 2020                      and byte ptr [eax],ah  

 00412EF8: 2020                      and byte ptr [eax],ah  

 00412EFA: 0D0A0D0A20                or eax,200A0D0A  

 00412EFF: 54                        push esp  

 00412F00: 6869732070                push 70207369  

 00412F05: 726F                      jb 00412F76  

 00412F07: 677261                    jb 00412F6B  

 00412F0A: 6D                        insd es:[edi],dx  

 00412F0B: 207072                    and byte ptr [eax+72],dh  

 00412F0E: 6F                        outsd dx,ds:[esi]  

 00412F0F: 7465                      jz 00412F76  

 00412F11: 63746564                  arpl word ptr [ebp+64],si  

 00412F15: 206279                    and byte ptr [edx+79],ah  

 00412F18: 20444250                  and byte ptr [edx+eax*2+50],al  

 00412F1C: 45                        inc ebp  

 00412F1D: 207631                    and byte ptr [esi+31],dh  

 00412F20: 2E352020556E              xor eax,6E552020  

 00412F26: 7265                      jb 00412F8D  

 00412F28: 6769737465726564          imul esi,dword ptr [bp+di+74],64657265  

 00412F30: 207665                    and byte ptr [esi+65],dh  

 00412F33: 7273                      jb 00412FA8  

 00412F35: 696F6E0D0A0D0A            imul ebp,dword ptr [edi+6E],0A0D0A0D  

 00412F3C: 0D0A0D0A20                or eax,200A0D0A  

 00412F41: 48                        dec eax  

 00412F42: 6F                        outsd dx,ds:[esi]  

 00412F43: 6D                        insd es:[edi],dx  

 00412F44: 657061                    jo 00412FA8  

 00412F47: 67653A20                  cmp ah,byte ptr gs:[bx+si]  

 00412F4B: 687474703A                push 3A707474  

 00412F50: 2F                        das  

 00412F51: 2F                        das  

 00412F52: 667364                    jnb 00412FB9  

 00412F55: 622E                      bound ebp,dword ptr [esi]  

 00412F57: 7965                      jns 00412FBE  

 00412F59: 61                        popad  

 00412F5A: 682E6E6574                push 74656E2E  

 00412F5F: 2020                      and byte ptr [eax],ah  

 00412F61: 2020                      and byte ptr [eax],ah  

 00412F63: 45                        inc ebp  

 00412F64: 2D6D61696C                sub eax,6C69616D  

 00412F69: 3A20                      cmp ah,byte ptr [eax]  

 00412F6B: 44                        inc esp  

 00412F6C: 2E42                      inc edx  

 00412F6E: 6F                        outsd dx,ds:[esi]  

 00412F6F: 7940                      jns 00412FB1  

 00412F71: 3132                      xor dword ptr [edx],esi  

 00412F73: 362E636F6D                arpl word ptr cs:[edi+6D],bp  

 00412F78: 200D0A007C7C              and byte ptr [7C7C000A],cl  

 00412F7E: 00D0                      add al,dl  

 00412F80: 0000                      add byte ptr [eax],al  

 00412F82: CC                        int 3  

 00412F83: 1000                      adc byte ptr [eax],al  

 00412F85: 0000                      add byte ptr [eax],al  

 00412F87: 0000                      add byte ptr [eax],al  

 00412F89: 0000                      add byte ptr [eax],al  

 00412F8B: 1000                      adc byte ptr [eax],al  

 00412F8D: 0000                      add byte ptr [eax],al  

 00412F8F: 40                        inc eax  

 00412F90: 0000                      add byte ptr [eax],al  

 00412F92: 005000                    add byte ptr [eax],dl  

 00412F95: 0000                      add byte ptr [eax],al  

 00412F97: 1000                      adc byte ptr [eax],al  

 00412F99: 0000                      add byte ptr [eax],al  

 00412F9B: 60                        pushad  

 00412F9C: 0000                      add byte ptr [eax],al  

 00412F9E: 0010                      add byte ptr [eax],dl  

 00412FA0: 0000                      add byte ptr [eax],al  

 00412FA2: 00C0                      add al,al  

 00412FA4: 0000                      add byte ptr [eax],al  

 00412FA6: 0010                      add byte ptr [eax],dl  

 00412FA8: 0000                      add byte ptr [eax],al  

 00412FAA: 0000                      add byte ptr [eax],al  

 00412FAC: 0000                      add byte ptr [eax],al  

 00412FAE: 0000                      add byte ptr [eax],al  

 00412FB0: 0000                      add byte ptr [eax],al  

 00412FB2: 0000                      add byte ptr [eax],al  

 00412FB4: 0000                      add byte ptr [eax],al  

 00412FB6: 0000                      add byte ptr [eax],al  

 00412FB8: 0000                      add byte ptr [eax],al  

 00412FBA: 0000                      add byte ptr [eax],al  

 00412FBC: 0000                      add byte ptr [eax],al  

 00412FBE: 0000                      add byte ptr [eax],al  

 00412FC0: 0000                      add byte ptr [eax],al  

 00412FC2: 0000                      add byte ptr [eax],al  

 00412FC4: 0000                      add byte ptr [eax],al  

 00412FC6: 0000                      add byte ptr [eax],al  

 00412FC8: 0000                      add byte ptr [eax],al  

 00412FCA: 0000                      add byte ptr [eax],al  

 00412FCC: 0000                      add byte ptr [eax],al  

 00412FCE: 0000                      add byte ptr [eax],al  

 00412FD0: 0000                      add byte ptr [eax],al  

 00412FD2: 0000                      add byte ptr [eax],al  

 00412FD4: 0000                      add byte ptr [eax],al  

 00412FD6: 0000                      add byte ptr [eax],al  

 00412FD8: 0000                      add byte ptr [eax],al  

 00412FDA: 0000                      add byte ptr [eax],al  

 00412FDC: 0000                      add byte ptr [eax],al  

 00412FDE: 0000                      add byte ptr [eax],al  

 00412FE0: 0000                      add byte ptr [eax],al  

 00412FE2: 0000                      add byte ptr [eax],al  

 00412FE4: 0000                      add byte ptr [eax],al  

 00412FE6: 0000                      add byte ptr [eax],al  

 00412FE8: 0000                      add byte ptr [eax],al  

 00412FEA: 0000                      add byte ptr [eax],al  

 00412FEC: 0000                      add byte ptr [eax],al  

 00412FEE: 0000                      add byte ptr [eax],al  

 00412FF0: 0000                      add byte ptr [eax],al  

 00412FF2: 0000                      add byte ptr [eax],al  

 00412FF4: 0000                      add byte ptr [eax],al  

 00412FF6: 0000                      add byte ptr [eax],al  

 00412FF8: 0000                      add byte ptr [eax],al  

 00412FFA: 0000                      add byte ptr [eax],al  

 00412FFC: 0000                      add byte ptr [eax],al  

 00412FFE: 0000                      add byte ptr [eax],al
```
