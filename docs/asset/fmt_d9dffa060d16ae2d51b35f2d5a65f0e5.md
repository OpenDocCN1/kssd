
0040E9A2 `E8 87FDFFFF` `call cyto-wj.0040E72E` `; (4)`

0040E9A7 `A1 B4CD4F00` `mov eax,dword ptr ds:\[4FCDB4\]`

0040E9AC `8945 D8` `mov dword ptr ss:\[ebp-28\],eax`

0040E9AF `8D45 D8` `lea eax,dword ptr ss:\[ebp-28\]`

0040E9B2 `50` `push eax`

0040E9B3 `FF35 B0CD4F00` `push dword ptr ds:\[4FCDB0\]`

0040E9B9 `8D45 D0` `lea eax,dword ptr ss:\[ebp-30\]`

0040E9BC `50` `push eax`

0040E9BD `8D45 CC` `lea eax,dword ptr ss:\[ebp-34\]`

0040E9C0 `50` `push eax`

0040E9C1 `8D45 C8` `lea eax,dword ptr ss:\[ebp-38\]`

0040E9C4 `50` `push eax`

0040E9C5 `FF15 404F4D00` `call dword ptr ds:\[<&msvcr71.__getmainargs>\]`

0040E9CB `83C4 20` `add esp,20`

0040E9CE `8945 C4` `mov dword ptr ss:\[ebp-3C\],eax`

0040E9D1 `3BC3` `cmp eax,ebx`

0040E9D3 `7D 08` `jge short cyto-wj.0040E9DD`

0040E9D5 `6A 08` `push 8`

0040E9D7 `E8 D8010000` `call <jmp.&msvcr71._amsg_exit>`

0040E9DC `59` `pop ecx`

0040E9DD `68 D8C24F00` `push cyto-wj.004FC2D8`

0040E9E2 `68 00C04F00` `push cyto-wj.004FC000`

0040E9E7 `E8 56020000` `call <jmp.&msvcr71._initterm>`

0040E9EC `59` `pop ecx`

0040E9ED `59` `pop ecx`

0040E9EE `A1 484F4D00` `mov eax,dword ptr ds:\[<&msvcr71._acmdln>\]`

0040E9F3 `8B30` `mov esi,dword ptr ds:\[eax\]`

0040E9F5 `8975 DC` `mov dword ptr ss:\[ebp-24\],esi`

0040E9F8 `8A06` `mov al,byte ptr ds:\[esi\]`

0040E9FA `3C 20` `cmp al,20`

0040E9FC `77 5D` `ja short cyto-wj.0040EA5B`

0040E9FE `3AC3` `cmp al,bl`

0040EA00 `74 05` `je short cyto-wj.0040EA07`

0040EA02 `395D E0` `cmp dword ptr ss:\[ebp-20\],ebx`

0040EA05 `75 54` `jnz short cyto-wj.0040EA5B`

0040EA07 `8A06` `mov al,byte ptr ds:\[esi\]`

0040EA09 `3AC3` `cmp al,bl`

0040EA0B `74 0A` `je short cyto-wj.0040EA17`

0040EA0D `3C 20` `cmp al,20`

0040EA0F `77 06` `ja short cyto-wj.0040EA17`

0040EA11 `46` `inc esi`

0040EA12 `8975 DC` `mov dword ptr ss:\[ebp-24\],esi`

0040EA15 `^ EB F0` `jmp short cyto-wj.0040EA07`

0040EA17 `895D A8` `mov dword ptr ss:\[ebp-58\],ebx`

0040EA1A `8D85 7CFFFFFF` `lea eax,dword ptr ss:\[ebp-84\]`

0040EA20 `50` `push eax`

0040EA21 `FF15 18424D00` `call dword ptr ds:\[<&kernel32.GetStartupInfoA>\]`

0040EA27 `F645 A8 01` `test byte ptr ss:\[ebp-58\],1`

0040EA2B `74 06` `je short cyto-wj.0040EA33`

0040EA2D `0FB745 AC` `movzx eax,word ptr ss:\[ebp-54\]`

0040EA31 `EB 03` `jmp short cyto-wj.0040EA36`

0040EA33 `6A 0A` `push 0A`

0040EA35 `58` `pop eax`

0040EA36 `50` `push eax`

0040EA37 `56` `push esi`

0040EA38 `53` `push ebx`

0040EA39 `53` `push ebx`

0040EA3A `FFD7` `call edi`

0040EA3C `50` `push eax`

0040EA3D `E8 9EF10A00` `call cyto-wj.004BDBE0`

0040EA42 `8BF0` `mov esi,eax`

0040EA44 `8975 C0` `mov dword ptr ss:\[ebp-40\],esi`

0040EA47 `395D E4` `cmp dword ptr ss:\[ebp-1C\],ebx`

0040EA4A `75 07` `jnz short cyto-wj.0040EA53`

0040EA4C `56` `push esi`

0040EA4D `FF15 4C4F4D00` `call dword ptr ds:\[<&msvcr71.exit>\]`

0040EA53 `FF15 504F4D00` `call dword ptr ds:\[<&msvcr71._cexit>\]`

0040EA59 `EB 55` `jmp short cyto-wj.0040EAB0`

0040EA5B `3C 22` `cmp al,22`

0040EA5D `75 0B` `jnz short cyto-wj.0040EA6A`

0040EA5F `33C9` `xor ecx,ecx`

0040EA61 `395D E0` `cmp dword ptr ss:\[ebp-20\],ebx`

0040EA64 `0F94C1` `sete cl`

0040EA67 `894D E0` `mov dword ptr ss:\[ebp-20\],ecx`

0040EA6A `0FB6C0` `movzx eax,al`

0040EA6D `50` `push eax`

0040EA6E `FF15 544F4D00` `call dword ptr ds:\[<&msvcr71._ismbblead>\]`

0040EA74 `59` `pop ecx`

0040EA75 `85C0` `test eax,eax`

0040EA77 `74 04` `je short cyto-wj.0040EA7D`

0040EA79 `46` `inc esi`

0040EA7A `8975 DC` `mov dword ptr ss:\[ebp-24\],esi`

0040EA7D `46` `inc esi`

0040EA7E `^ E9 72FFFFFF` `jmp cyto-wj.0040E9F5`

OEP 中某些 `call` 内容也被偷了：

1) `call dumped_.0040EABC`

0040EABC `E9 DA1F2F01` `jmp 01700A9B`

0040EABC `68 6CE84000` `push <jmp.&msvcr71._except_handler3>`

0040EAC1 `64:A1 00000000` `mov eax,dword ptr fs:\[0\]`

0040EAC7 `50` `push eax`

0040EAC8 `8B4424 10` `mov eax,dword ptr ss:\[esp+10\]`

0040EACC `896C24 10` `mov dword ptr ss:\[esp+10\],ebp`

0040EAD0 `8D6C24 10` `lea ebp,dword ptr ss:\[esp+10\]`

0040EAD4 `2BE0` `sub esp,eax`

0040EAD6 `53` `push ebx`

0040EAD7 `56` `push esi`

0040EAD8 `57` `push edi`

0040EAD9 `8B45 F8` `mov eax,dword ptr ss:\[ebp-8\]`

0040EADC `8965 E8` `mov dword ptr ss:\[ebp-18\],esp`

0040EADF `50` `push eax`

0040EAE0 `8B45 FC` `mov eax,dword ptr ss:\[ebp-4\]`

0040EAE3 `C745 FC FFFFFFFF` `mov dword ptr ss:\[ebp-4\],-1`

0040EAEA `8945 F8` `mov dword ptr ss:\[ebp-8\],eax`

0040EAED `8D45 F0` `lea eax,dword ptr ss:\[ebp-10\]`

0040EAF0 `64:A3 00000000` `mov dword ptr fs:\[0\],eax`

0040EAF6 `C3` `retn`

2) `call cyto-wj.0040EBBA`

0040EBBA `E9 991E2F01` `jmp 01700A58`

0040EBBA `6A 0C` `push 0C`

0040EBBC `68 30894D00` `push cyto-wj.004D8930`

0040EBC1 `E8 F6FEFFFF` `call cyto-wj.0040EABC`

0040EBC6 `C745 E4 48E24E00` `mov dword ptr ss:\[ebp-1C\],cyto-wj.004EE248`

0040EBCD `817D E4 48E24E00` `cmp dword ptr ss:\[ebp-1C\],cyto-wj.004EE248`

0040EBD4 `73 22` `jnb short cyto-wj.0040EBF8`

0040EBD6 `8365 FC 00` `and dword ptr ss:\[ebp-4\],0`

0040EBDA `8B45 E4` `mov eax,dword ptr ss:\[ebp-1C\]`

0040EBDD `8B00` `mov eax,dword ptr ds:\[eax\]`

0040EBDF `85C0` `test eax,eax`

0040EBE1 `74 0B` `je short cyto-wj.0040EBEE`

0040EBE3 `FFD0` `call eax`

0040EBE5 `EB 07` `jmp short cyto-wj.0040EBEE`

0040EBE7 `33C0` `xor eax,eax`

0040EBE9 `40` `inc eax`

0040EBEA `C3` `retn`

0040EBEB `8B65 E8` `mov esp,dword ptr ss:\[ebp-18\]`

0040EBEE `834D FC FF` `or dword ptr ss:\[ebp-4\],FFFFFFFF`

0040EBF2 `8345 E4 04` `add dword ptr ss:\[ebp-1C\],4`

0040EBF6 `^ EB D5` `jmp short cyto-wj.0040EBCD`

0040EBF8 `E8 FAFEFF` `call cyto-wj.0040EAF7`

0040EBFD `C3` `retn`

`call cyto-wj.0040EAF7`

0040EAF7 `E9 7C1B2F01` `jmp 01700678`

0040EAF7 `8B4D F0` `mov ecx,dword ptr ss:\[ebp-10\]`

0040EAFA `64:890D 00000000` `mov dword ptr fs:\[0\],ecx`

0040EB01 `59` `pop ecx`

0040EB02 `5F` `pop edi`

0040EB03 `5E` `pop esi`

0040EB04 `5B` `pop ebx`

0040EB05 `C9` `leave`

0040EB06 `51` `push ecx`

0040EB07 `C3` `retn`

3) `call cyto-wj.0040EC48`

0040EC48 `E9 BB1B2F01` `jmp 01700808`

0040EC48 `68 00000300` `push 30000`
