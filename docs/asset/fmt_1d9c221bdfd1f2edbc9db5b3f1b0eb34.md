
SDPI:00481C52 `mov` `ecx` `\[esp+30h\]`

SDPI:00481C56 `push` `esi` `; LPCSTR`

SDPI:00481C57 `push` `ecx` `; int`

SDPI:00481C58 `call` `LOadDll_4817B7` `; 载入相应的 DLL`

SDPI:00481C5D `loc_481C5D:` `; CODE XREF: Disposal_IMP+22F j`

SDPI:00481C5D `test` `eax` `eax`

SDPI:00481C5F `mov` `\[esp+444h+hdll\]` `eax`

SDPI:00481C63 `jnz` `short` `loc_481CB3`

SDPI:00481C65 `push` `esi`

SDPI:00481C66 `push` `offset` `strcanntloaddll` `; relativeAddress`

SDPI:00481C6B `call` `GetAbsAddress` `; 获取实际地址`

SDPI:00481C70 `lea` `edx` `\[esp+244h\]`

SDPI:00481C77 `push` `eax` `; strIn`

SDPI:00481C78 `push` `edx` `; strout`

SDPI:00481C79 `call` `wsprintfA` `; 如果不能载入 dll 则提示出错`

SDPI:00481C7E `add` `esp` `0Ch`

SDPI:00481C81 `push` `12h` `; uType`

SDPI:00481C83 `push` `offset` `strError` `; relativeAddress`

SDPI:00481C88 `call` `GetAbsAddress` `; 获取实际地址`

SDPI:00481C8D `push` `eax` `; lpCaption`

SDPI:00481C8E `lea` `eax` `\[esp+248h\]`

SDPI:00481C95 `push` `eax` `; lpText`

SDPI:00481C96 `push` `0` `; hWnd`

SDPI:00481C98 `call` `MessageBoxA` `; MessageBoxA 函数，壳用的宏来处理，`

SDPI:00481C98 `; 判断函数的前 5 位是否为 CC`

SDPI:00481C98 `; 也就是判断有没有下 int3 断点`

SDPI:00481C9D `cmp` `eax` `3`

SDPI:00481CA0 `jz` `short` `loc_481CAC`

SDPI:00481CA2 `cmp` `eax` `4`

SDPI:00481CA5 `jnz` `short` `loc_481CCB`

SDPI:00481CA7 `jmp` `loc_481BBE`

SDPI:00481CAC `; ----------------------------------------------------------------------------`

SDPI:00481CAC `loc_481CAC:` `; CODE XREF: Disposal_IMP+27F j`

SDPI:00481CAC `push` `0` `; uExitCode`

SDPI:00481CAE `call` `ExitProcess`

SDPI:00481CB3 `loc_481CB3:` `; CODE XREF: Disposal_IMP+242 j`

SDPI:00481CB3 `mov` `eax` `\[esp+444h\]`

SDPI:00481CBA `test` `eax` `eax`

SDPI:00481CBC `mov` `eax` `\[edi\]`

SDPI:00481CBE `lea` `ecx` `\[ebx+eax\]`

SDPI:00481CC1 `jz` `short` `loc_481CD3`

SDPI:00481CC3 `add` `ecx` `ebp`

SDPI:00481CC5 `lea` `esi` `\[ebp+ebx+0\]`

SDPI:00481CC9 `jmp` `short` `loc_481CD5`

SDPI:00481CCB `; ----------------------------------------------------------------------------`

SDPI:00481CCB `loc_481CCB:` `; CODE XREF: Disposal_IMP+284 j`

SDPI:00481CCB `add` `edi` `14h`

SDPI:00481CCE `jmp` `loc_481B46` `; 保存输入表起始 VA`

SDPI:00481CD3 `; ----------------------------------------------------------------------------`

SDPI:00481CD3 `loc_481CD3:` `; CODE XREF: Disposal_IMP+2A0 j`

SDPI:00481CD3 `mov` `esi` `ebx`

SDPI:00481CD5 `loc_481CD5:` `; CODE XREF: Disposal_IMP+2A8 j`

SDPI:00481CD5 `mov` `edx` `\[edi+10h\]`

SDPI:00481CD8 `mov` `ebp` `ecx`

SDPI:00481CDA `add` `esi` `edx`

SDPI:00481CDC `test` `eax` `eax`

SDPI:00481CDE `jnz` `short` `loc_481CE2`

SDPI:00481CE0 `mov` `ebp` `esi`

SDPI:00481CE2 `loc_481CE2:` `; CODE XREF: Disposal_IMP+2BD j`

SDPI:00481CE2 `call` `CRC_480467`

SDPI:00481CE7 `loc_481CE7:` `; CODE XREF: Disposal_IMP+607 j`

SDPI:00481CE7 `cmp` `dword ptr [ebp+0]` `0` `; 上面载入 DLL 后，这里填充当前 DLL 的加载函数`

SDPI:00481CE7 `; 填充完跳去处理下一个 DLL`

SDPI:00481CEB `jz` `DoneCurrDll_48202D`

SDPI:00481CF1 `mov` `edi` `\[esp+444h+var_430\]`

SDPI:00481CF5 `lea` `ecx` `\[esp+38h\]`

SDPI:00481CF9 `lea` `edx` `\[esp+13Ch\]`

SDPI:00481D00 `push` `ecx` `; Stack_from`

SDPI:00481D01 `push` `edx` `; Stack_to`

SDPI:00481D02 `push` `10h` `; size1`

SDPI:00481D04 `push` `edi` `; int`

SDPI:00481D05 `push` `4` `; siz2`

SDPI:00481D07 `push` `ebp` `; fromDeAddr`

SDPI:00481D08 `call` `DeCrypt_Functions` `; 解密出 Thunk Value`

SDPI:00481D0D `lea` `eax` `\[esp+38h\]`

SDPI:00481D11 `lea` `ecx` `\[esp+13Ch\]`

SDPI:00481D18 `push` `eax` `; Stack_from`

SDPI:00481D19 `push` `ecx` `; Stack_to`

SDPI:00481D1A `push` `10h` `; size1`

SDPI:00481D1C `push` `edi` `; int`

SDPI:00481D1D `push` `4` `; siz2`

SDPI:00481D1F `push` `esi` `; fromDeAddr`

SDPI:00481D20 `call` `DeCrypt_Functions`

SDPI:00481D25 `mov` `eax` `\[ebp+0\]`

SDPI:00481D28 `mov` `ecx` `\[esp+444h\]`

SDPI:00481D2F `test` `ecx` `ecx`

SDPI:00481D31 `jz` `short` `loc_481D3D`

SDPI:00481D33 `mov` `edx` `\[esp+2Ch\]`

SDPI:00481D37 `add` `ebx` `eax`

SDPI:00481D39 `add` `ebx` `edx`

SDPI:00481D3B `jmp` `short` `loc_481D3F`

SDPI:00481D3D `; ----------------------------------------------------------------------------`

SDPI:00481D3D `loc_481D3D:` `; CODE XREF: Disposal_IMP+310 j`

SDPI:00481D3D `add` `ebx` `eax`

SDPI:00481D3F `loc_481D3F:` `; CODE XREF: Disposal_IMP+31A j`

SDPI:00481D3F `test` `eax` `80000000h`

SDPI:00481D44 `jz` `short` `loc_481DA8` `; 判断是否为字符串`

SDPI:00481D46 `test` `ecx` `ecx`

SDPI:00481D48 `jz` `short` `loc_481D91`

SDPI:00481D4A `mov` `ecx` `\[esp+444h+var_42C\]`

SDPI:00481D4E `test` `ecx` `ecx`

SDPI:00481D50 `jz` `short` `loc_481D6C`

SDPI:00481D52 `mov` `edx` `\[esi\]`

SDPI:00481D54 `mov` `eax` `\[esp+444h+hdll\]`

SDPI:00481D58 `and` `edx` `7FFFFFFFh`

SDPI:00481D5E `push` `edx`

SDPI:00481D5F `push` `eax`

SDPI:00481D60 `call` `sub_48359A`

SDPI:00481D65 `mov` `\[esi\]` `eax`

SDPI:00481D67 `jmp` `loc_48200C` `; 如果不相等则清除 ThunkValue`

SDPI:00481D6C `; ----------------------------------------------------------------------------`

SDPI:00481D6C `loc_481D6C:` `; CODE XREF: Disposal_IMP+32F j`
