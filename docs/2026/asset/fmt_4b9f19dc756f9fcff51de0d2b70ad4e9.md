
## 汇编转储

`00013996: B810000000`                `mov eax,00000010`
`0001399B: 50`                        `push eax`
`0001399C: B8D4AD4000`                `mov eax,0040ADD4`
`000139A1: 03C5`                      `add eax,ebp`
`000139A3: 50`                        `push eax`
`000139A4: B8B4AD4000`                `mov eax,0040ADB4`
`000139A9: 03C5`                      `add eax,ebp`
`000139AB: 50`                        `push eax`     <--`"C:\"`
`000139AC: FF95BFBB4000`              `call dword ptr [ebp+0040BBBF]`   <--`GetVolumeInformationA`

### 获取C盘空余磁盘信息

`000139B2: B8A4AD4000`                `mov eax,0040ADA4`
`000139B7: 03C5`                      `add eax,ebp`
`000139B9: BBA8AD4000`                `mov ebx,0040ADA8`
`000139BE: 03DD`                      `add ebx,ebp`
`000139C0: B9ACAD4000`                `mov ecx,0040ADAC`
`000139C5: 03CD`                      `add ecx,ebp`
`000139C7: C701633A5C00`              `mov dword ptr [ecx],005C3A63`
`000139CD: 50`                        `push eax`
`000139CE: 53`                        `push ebx`
`000139CF: 6A00`                      `push 00000000`
`000139D1: 51`                        `push ecx`
`000139D2: FF95B7BB4000`              `call dword ptr [ebp+0040BBB7]`   <--`GetDiskFreeSpaceExA`

### 从注册表获取信息

`000139D8: 8B85A8AD4000`              `mov eax,dword ptr [ebp+0040ADA8]`
`000139DE: F7D0`                      `not eax`
`000139E0: 3385D0AD4000`              `xor eax,dword ptr [ebp+0040ADD0]`
`000139E6: 8985A8AD4000`              `mov dword ptr [ebp+0040ADA8],eax`
`000139EC: E844020000`                `call 00013C35`     <--从注册表中获取信息

### 好象是还原信息

`000139F1: B801000000`                `mov eax,00000001`
`000139F6: 33DB`                      `xor ebx,ebx`
`000139F8: BE65B14000`                `mov esi,0040B165`
`000139FD: 03F5`                      `add esi,ebp`
`000139FF: 8A1E`                      `mov bl,byte ptr [esi]`
`00013A01: 80FB00`                    `cmp bl,00`
`00013A04: 7405`                      `jz 00013A0B`
`00013A06: F7E3`                      `mul ebx`
`00013A08: 46`                        `inc esi`
`00013A09: EBF4`                      `jmp 000139FF`
`00013A0B: 33855DB74000`              `xor eax,dword ptr [ebp+0040B75D]`
`00013A11: 3385A8AD4000`              `xor eax,dword ptr [ebp+0040ADA8]`
`00013A17: 86E0`                      `xchg al,ah`
`00013A19: 8BD8`                      `mov ebx,eax`

### 处理信息

`00013A1B: BE79B14000`                `mov esi,0040B179`
`00013A20: 03F5`                      `add esi,ebp`
`00013A22: E80E000000`                `call 00013A35`   <--处理信息
`00013A27: 3BD8`                      `cmp ebx,eax`
`00013A29: 7405`                      `jz 00013A30`
`00013A2B: 66B80000`                  `mov ax,0000`
`00013A2F: C3`                        `ret`
`00013A30: 66B80100`                  `mov ax,0001`
`00013A34: C3`                        `ret`

----------子程序结束-----------

-----------子程序开始----------

`00013A35: 53`                        `push ebx`
`00013A36: 56`                        `push esi`
`00013A37: 33DB`                      `xor ebx,ebx`
`00013A39: 8A06`                      `mov al,byte ptr [esi]`
`00013A3B: 3C39`                      `cmp al,39`
`00013A3D: 7606`                      `jbe 00013A45`
`00013A3F: 24CF`                      `and al,CF`
`00013A41: 2C37`                      `sub al,37`
`00013A43: EB02`                      `jmp 00013A47`
`00013A45: 240F`                      `and al,0F`
`00013A47: C1E304`                    `shl ebx,04`
`00013A4A: 0AD8`                      `or bl,al`
`00013A4C: 46`                        `inc esi`
`00013A4D: 803E00`                    `cmp byte ptr [esi],00`
`00013A50: 75E7`                      `jnz 00013A39`
`00013A52: 8BC3`                      `mov eax,ebx`
`00013A54: 5E`                        `pop esi`
`00013A55: 5B`                        `pop ebx`
`00013A56: C3`                        `ret`

---------子程序结束-----------

`00013A57: 0000`                      `add byte ptr [eax],al`
`00013A59: 0000`                      `add byte ptr [eax],al`
`00013A5B: 0000`                      `add byte ptr [eax],al`
`00013A5D: 0000`                      `add byte ptr [eax],al`
`00013A5F: 633A`                      `arpl word ptr [edx],di`
`00013A61: 5C`                        `pop esp`
`00013A62: 00256C580043`              `add byte ptr [4300586C],ah`
`00013A68: 3A5C0000`                  `cmp bl,byte ptr [eax+eax]`
`00013A6C: 0000`                      `add byte ptr [eax],al`
`00013A6E: 0000`                      `add byte ptr [eax],al`
`00013A70: 0000`                      `add byte ptr [eax],al`
`00013A72: 0000`                      `add byte ptr [eax],al`
`00013A74: 0000`                      `add byte ptr [eax],al`
`00013A76: 0000`                      `add byte ptr [eax],al`
`00013A78: 0000`                      `add byte ptr [eax],al`
`00013A7A: 0000`                      `add byte ptr [eax],al`
`00013A7C: 0000`                      `add byte ptr [eax],al`
`00013A7E: 0000`                      `add byte ptr [eax],al`
`00013A80: 0000`                      `add byte ptr [eax],al`
`00013A82: 0000`                      `add byte ptr [eax],al`
`00013A84: 0000`                      `add byte ptr [eax],al`
`00013A86: 0000`                      `add byte ptr [eax],al`
`00013A88: 0000`                      `add byte ptr [eax],al`
`00013A8A: 0000`                      `add byte ptr [eax],al`
`00013A8C: 0000`                      `add byte ptr [eax],al`
`00013A8E: 0000`                      `add byte ptr [eax],al`
`00013A90: 0000`                      `add byte ptr [eax],al`
`00013A92: 0000`                      `add byte ptr [eax],al`
`00013A94: 0000`                      `add byte ptr [eax],al`

----子程序----------

`00013A96: 00608B`                    `add byte ptr [eax-75],ah`
`00013A99: 9D`                        `popfd`
`00013A9A: 69B4400083FB000F84A600`    `imul esi,dword ptr [eax+eax*2+00FB8300],00A6840F`
`00013AA5: 0000`                      `add byte ptr [eax],al`
`00013AA7: 039DD3B24000`              `add ebx,dword ptr [ebp+0040B2D3]`
`00013AAD: 8B430C`                    `mov eax,dword ptr [ebx+0C]`
`00013AB0: 83F800`                    `cmp eax,00000000`
`00013AB3: 0F8494000000`              `jz 00013B4D`
`00013AB9: 0385D3B24000`              `add eax,dword ptr [ebp+0040B2D3]`
`00013ABF: 8BF0`                      `mov esi,eax`
`00013AC1: 50`                        `push eax`
`00013AC2: FF9564BF4000`              `call dword ptr [ebp+0040BF64]`   <--`LoadLibraryA`
`00013AC8: 89857DBA4000`              `mov dword ptr [ebp+0040BA7D],eax`
`00013ACE: 8B33`                      `mov esi,dword ptr [ebx]`
`00013AD0: 8B7B10`                    `mov edi,dword ptr [ebx+10]`
`00013AD3: 83FE00`                    `cmp esi,00000000`
`00013AD6: 7502`                      `jnz 00013ADA`
`00013AD8: 8BF7`                      `mov esi,edi`
`00013ADA: 03B5D3B24000`              `add esi,dword ptr [ebp+0040B2D3]`
`00013AE0: 03BDD3B24000`              `add edi,dword ptr [ebp+0040B2D3]`
`00013AE6: 8B06`                      `mov eax,dword ptr [esi]`
`00013AE8: 83F800`                    `cmp eax,00000000`
`00013AEB: 7505`                      `jnz 00013AF2`
`00013AED: 83C314`                    `add ebx,00000014`
`00013AF0: EBBB`                      `jmp 00013AAD`
`00013AF2: 807E0380`                  `cmp byte ptr [esi+03],80`
`00013AF6: 7514`                      `jnz 00013B0C`
`00013AF8: 33C0`                      `xor eax,eax`
`00013AFA: 668706`                    `xchg word ptr [esi],ax`
`00013AFD: 50`                        `push eax`
`00013AFE: FFB57DBA4000`              `push dword ptr [ebp+0040BA7D]`
`00013B04: FF9568BF4000`              `call dword ptr [ebp+0040BF68]`
`00013B0A: EB37`                      `jmp 00013B43`
`00013B0C: 33C0`                      `xor eax,eax`
`00013B0E: 8706`                      `xchg dword ptr [esi],eax`
`00013B10: 0385D3B24000`              `add eax,dword ptr [ebp+0040B2D3]`
`00013B16: 83C002`                    `add eax,00000002`
`00013B19: 8BC8`                      `mov ecx,eax`
