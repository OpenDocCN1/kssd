
SDPI:0047BABF `;` `----------------------------------------------------------------------------`  

SDPI:0047BAC4 `dd 90909090h`  

SDPI:0047BAC8 `;` `----------------------------------------------------------------------------`  

SDPI:0047BAC8 `push 7`  

SDPI:0047BACA `call loc_47BAD0`  

SDPI:0047BACF `nop`  

SDPI:0047BAD0  

SDPI:0047BAD0 `loc_47BAD0:` `; CODE XREF: FNDDBG_47BA78+52 p`  

SDPI:0047BAD0 `pop eax`  

SDPI:0047BAD1 `add eax, 11h`  

SDPI:0047BAD6 `push eax`  

SDPI:0047BAD7 `jmp ExitProcess`  

SDPI:0047BAD7 `;` `----------------------------------------------------------------------------`  

SDPI:0047BADC `dd 90909090h`  

SDPI:0047BADC `FNDDBG_47BA78 endp`  

SDPI:0047BADC  

SDPI:0047BAE0 `;` `----------------------------------------------------------------------------`  

SDPI:0047BAE0 `pop edx`  

SDPI:0047BAE1 `mov eax, ecx`  

SDPI:0047BAE3 `add eax, edx`  

SDPI:0047BAE5 `inc ecx`  

SDPI:0047BAE6 `push eax`  

SDPI:0047BAE7 `inc ecx`  

SDPI:0047BAE8 `pop ebx`  

SDPI:0047BAE9 `pop ecx`  

SDPI:0047BAEA `push eax`  

SDPI:0047BAEB `sub eax, 8`  

SDPI:0047BAEE `pop ebx`  

SDPI:0047BAEF `pop ebx`  

SDPI:0047BAF0 `inc eax`  

SDPI:0047BAF1 `add eax, ebx`  

SDPI:0047BAF3 `pop eax`  

SDPI:0047BAF4 `pushfw`  

SDPI:0047BAF6 `popfw`  

SDPI:0047BAF8 `popfw`  

SDPI:0047BAFA `pop es`  

SDPI:0047BAFB `mov eax, 12345678h`  

SDPI:0047BB00 `push eax`  

SDPI:0047BB01 `call loc_47BB07`  

SDPI:0047BB06 `nop`  

SDPI:0047BB07  

SDPI:0047BB07 `loc_47BB07:` `; CODE XREF: SDPI:0047BB01 p`  

SDPI:0047BB07 `pop eax`  

SDPI:0047BB08 `add eax, 12Ch`  

SDPI:0047BB0D `push eax`  

SDPI:0047BB0E `pop ebx`  

SDPI:0047BB0F `add eax, 12h`  

SDPI:0047BB12 `pop edx`  

SDPI:0047BB13 `add eax, edx`  

SDPI:0047BB15 `mov edx, eax`  

SDPI:0047BB17 `push ebx`  

SDPI:0047BB18 `mov ebx, es:[ecx+100h]`  

SDPI:0047BB1F `push ebx`  

SDPI:0047BB20 `mov eax, esp`  

SDPI:0047BB22 `mov ebx, eax`  

SDPI:0047BB24 `push ebx`  

SDPI:0047BB25 `pop edx`  

SDPI:0047BB26 `mov es:[ecx+100h], eax`  

SDPI:0047BB2D `xor eax, eax`  

SDPI:0047BB2F `jle short loc_47BB38`  

SDPI:0047BB31 `jg short loc_47BB38`  

SDPI:0047BB31 `;` `----------------------------------------------------------------------------`  

SDPI:0047BB33 `dd 401000h`  

SDPI:0047BB37 `db 0E8h` `; ?`  

SDPI:0047BB38 `;` `----------------------------------------------------------------------------`  

SDPI:0047BB38  

SDPI:0047BB38 `loc_47BB38:` `; CODE XREF: SDPI:0047BB2F j`  

SDPI:0047BB38 `; SDPI:0047BB31 j`  

SDPI:0047BB38 `pushfw`  

SDPI:0047BB3A `push ecx`  

SDPI:0047BB3B `xor ecx, ecx`  

SDPI:0047BB3D `jcxz loc_47BB45`  

SDPI:0047BB40 `add [eax], dl`  

SDPI:0047BB42 `inc eax`  

SDPI:0047BB43 `add al, ch`  

SDPI:0047BB45  

SDPI:0047BB45 `loc_47BB45:` `; CODE XREF: SDPI:0047BB3D j`  

SDPI:0047BB45 `pop ecx`  

SDPI:0047BB46 `nop`  

SDPI:0047BB47 `nop`  

SDPI:0047BB48 `nop`  

SDPI:0047BB49 `nop`  

SDPI:0047BB4A `nop`  

SDPI:0047BB4B `nop`  

SDPI:0047BB4C `nop`  

SDPI:0047BB4D `nop`  

SDPI:0047BB4E `nop`  

SDPI:0047BB4F `nop`  

SDPI:0047BB50 `nop`  

SDPI:0047BB51 `nop`  

SDPI:0047BB52 `nop`  

SDPI:0047BB53 `nop`  

SDPI:0047BB54 `nop`  

SDPI:0047BB55 `nop`  

SDPI:0047BB56 `nop`  

SDPI:0047BB57 `nop`  

SDPI:0047BB58 `nop`  

SDPI:0047BB59 `nop`  

SDPI:0047BB5A `nop`  

SDPI:0047BB5B `nop`  

SDPI:0047BB5C `nop`  

SDPI:0047BB5D `popfw`  

SDPI:0047BB5F `jo short loc_47BB67`  

SDPI:0047BB61 `jno short loc_47BB67`  

SDPI:0047BB61 `;` `----------------------------------------------------------------------------`  

SDPI:0047BB63 `dd 401000h`  

SDPI:0047BB67 `;` `----------------------------------------------------------------------------`  

SDPI:0047BB67  

SDPI:0047BB67 `loc_47BB67:` `; CODE XREF: SDPI:0047BB5F j`  

SDPI:0047BB67 `; SDPI:0047BB61 j`  

SDPI:0047BB67 `int 3` `; Trap to Debugger`  

SDPI:0047BB68 `nop` `; SEH 位置:0047BC32`  

SDPI:0047BB69 `xor eax, eax`  

SDPI:0047BB6B `mov dword ptr [eax], 402C6Bh`  

SDPI:0047BB71 `jp short loc_47BB7D`  

SDPI:0047BB73 `jnp short loc_47BB7D`  

SDPI:0047BB73 `;` `----------------------------------------------------------------------------`  

SDPI:0047BB75 `dd 401000h`  

SDPI:0047BB79 `dd 403D7Bh`  

SDPI:0047BB7D `;` `----------------------------------------------------------------------------`  

SDPI:0047BB7D  

SDPI:0047BB7D `loc_0047BB7D:` `; CODE XREF: SDPI:0047B99B p`  

SDPI:0047BB7D `; SDPI:0047BB71 j ...`  

SDPI:0047BB7D `call loc_0047BB83`  

SDPI:0047BB82 `nop`  

SDPI:0047BB83  

SDPI:0047BB83 `loc_0047BB83:` `; CODE XREF: SDPI:loc_0047BB7D p`  

SDPI:0047BB83 `pop eax`  

SDPI:0047BB84 `add eax, 11h`  

SDPI:0047BB89 `push eax`  

SDPI:0047BB8A `jmp GetTickCount`  

SDPI:0047BB8A `;` `----------------------------------------------------------------------------`  

SDPI:0047BB8F `dd 90909090h`  

SDPI:0047BB93 `;` `----------------------------------------------------------------------------`  

SDPI:0047BB93 `call loc_0047BB99`  

SDPI:0047BB98 `nop`  

SDPI:0047BB99  

SDPI:0047BB99 `loc_0047BB99:` `; CODE XREF: SDPI:0047BB93 p`  

SDPI:0047BB99 `pop edx`  

SDPI:0047BB9A `add edx, 0FFFFFB04h`  

SDPI:0047BBA0 `mov [edx], eax`  

SDPI:0047BBA2 `pop ebp`  

SDPI:0047BBA3 `add eax, edx`  

SDPI:0047BBA5 `push eax`  

SDPI:0047BBA6 `call loc_0047B9A8`  

SDPI:0047BBAB  

SDPI:0047BBAB `loc_0047BBAB:` `; CODE XREF: SDPI:0047B9AA j`  

SDPI:0047BBAB `call loc_0047BBB1`  

SDPI:0047BBB0 `nop`  

SDPI:0047BBB1  

SDPI:0047BBB1 `loc_0047BBB1:` `; CODE XREF: SDPI:loc_0047BBAB p`  

SDPI:0047BBB1 `pop edx`
