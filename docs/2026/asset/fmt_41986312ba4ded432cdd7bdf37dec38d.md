
```
00412E69: 0000                      add byte ptr [eax],al
00412E6B: 0000                      add byte ptr [eax],al
00412E6D: 0000                      add byte ptr [eax],al
00412E6F: 0000                      add byte ptr [eax],al
00412E71: 0000                      add byte ptr [eax],al
00412E73: 0000                      add byte ptr [eax],al
00412E75: 0000                      add byte ptr [eax],al
00412E77: 0000                      add byte ptr [eax],al
00412E79: 0000                      add byte ptr [eax],al
00412E7B: 0000                      add byte ptr [eax],al
00412E7D: 0000                      add byte ptr [eax],al
00412E7F: 0000                      add byte ptr [eax],al
00412E81: 0000                      add byte ptr [eax],al
00412E83: 0000                      add byte ptr [eax],al
00412E85: 0000                      add byte ptr [eax],al
00412E87: 0000                      add byte ptr [eax],al
00412E89: 0000                      add byte ptr [eax],al
00412E8B: 0000                      add byte ptr [eax],al
00412E8D: 0000                      add byte ptr [eax],al
00412E8F: 0000                      add byte ptr [eax],al
00412E91: 0000                      add byte ptr [eax],al
00412E93: 0000                      add byte ptr [eax],al
00412E95: 0000                      add byte ptr [eax],al
00412E97: 0000                      add byte ptr [eax],al
00412E99: 0000                      add byte ptr [eax],al
00412E9B: 0000                      add byte ptr [eax],al
00412E9D: 0000                      add byte ptr [eax],al
00412E9F: 0000                      add byte ptr [eax],al
00412EA1: 0000                      add byte ptr [eax],al
00412EA3: 0000                      add byte ptr [eax],al
00412EA5: 0000                      add byte ptr [eax],al
00412EA7: 0000                      add byte ptr [eax],al
00412EA9: 0000                      add byte ptr [eax],al
00412EAB: 0000                      add byte ptr [eax],al
00412EAD: 0003                      add byte ptr [ebx],al
00412EAF: 0000                      add byte ptr [eax],al
00412EB1: 0029                      add byte ptr [ecx],ch
00412EB3: 0000                      add byte ptr [eax],al
00412EB5: 002A                      add byte ptr [edx],ch
00412EB7: 00494E                    add byte ptr [ecx+4E],cl
00412EBA: 5F                        pop edi
00412EBB: 43                        inc ebx
00412EBC: 4F                        dec edi
00412EBD: 44                        inc esp
00412EBE: 45                        inc ebp
00412EBF: 0000                      add byte ptr [eax],al
00412EC1: 0000                      add byte ptr [eax],al
00412EC3: 0020                      add byte ptr [eax],ah
00412EC5: 2020                      and byte ptr [eax],ah
00412EC7: 2020                      and byte ptr [eax],ah
00412EC9: 2020                      and byte ptr [eax],ah
00412ECB: 2020                      and byte ptr [eax],ah
00412ECD: 2020                      and byte ptr [eax],ah
00412ECF: B1BE                      mov cl,BE
00412ED1: C8EDBCFE                  enter BCED,FE
00412ED5: D3C9                      ror ecx,cl
00412ED7: 20BBC3D3B076              and byte ptr [ebx+76B0D3C3],bh
00412EDD: 312E                      xor dword ptr [esi],ebp
00412EDF: 3520CEB4D7                xor eax,D7B4CE20
00412EE4: A2B2E1B0E6                mov [E6B0E1B2],al
00412EE9: B1BE                      mov cl,BE
00412EEB: BCD3C3DC20                mov esp,20DCC3D3
00412EF0: 2020                      and byte ptr [eax],ah
00412EF2: 2020                      and byte ptr [eax],ah
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
```
