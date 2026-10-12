
`SDPI:00481137`
`mov edi, [eax+14h]`

`SDPI:0048113A`
`mov ebp, [eax+8]`

`SDPI:0048113D`
`add edi, edx`

`SDPI:0048113F`
`mov [eax+0Ch], ecx`

`SDPI:00481142`
`mov [eax+14h], edi`

`SDPI:00481145`
`mov edi, 1000h`

`SDPI:0048114A`
`sub edi, esi`

`SDPI:0048114C`
`add eax, 28h`

`SDPI:0048114F`
`add ebp, edi`

`SDPI:00481151`
`mov [eax-20h], ebp`

`SDPI:00481154`
`mov edi, ebp`

`SDPI:00481156`
`add ecx, edi`

`SDPI:00481158`
`mov edi, [eax+14h]`

`SDPI:0048115B`
`mov ebp, [eax+8]`

`SDPI:0048115E`
`add edi, edx`

`SDPI:00481160`
`mov [eax+14h], edi`

`SDPI:00481163`
`mov edi, 1000h`

`SDPI:00481168`
`sub edi, esi`

`SDPI:0048116A`
`mov [eax+0Ch], ecx`

`SDPI:0048116D`
`add ebp, edi`

`SDPI:0048116F`
`mov [eax+8], ebp`

`SDPI:00481172`
`mov edi, ebp`

`SDPI:00481174`
`add eax, 28h`

`SDPI:00481177`
`add ecx, edi`

`SDPI:00481179`
`mov edi, [eax+14h]`

`SDPI:0048117C`
`mov [eax+0Ch], ecx`

`SDPI:0048117F`
`mov ecx, 1000h`

`SDPI:00481184`
`add edi, edx`

`SDPI:00481186`
`mov edx, [eax+8]`

`SDPI:00481189`
`sub ecx, esi`

`SDPI:0048118B`
`mov [eax+14h], edi`

`SDPI:0048118E`
`add edx, ecx`

`SDPI:00481190`
`mov [eax+8], edx`

`SDPI:00481193`
`mov edx, [esp+10h]`

`SDPI:00481197`
`push 0 ; lpflOldProtect`

`SDPI:00481199`
`push edx ; flNewProtect`

`SDPI:0048119A`
`push 200h ; dwSize`

`SDPI:0048119F`
`push ebx ; lpAddress`

`SDPI:004811A0`
`call VirtualProtect`

`SDPI:004811A5`
`pop esi ; 改完后再把文件头属性还原回去`

`SDPI:004811A6`
`pop ebp`

`SDPI:004811A7`

`SDPI:004811A7`
`Done_4811A7: ; CODE XREF: Erase_PEHEADER+10 j`

`SDPI:004811A7`
`; Erase_PEHEADER+1B j ...`

`SDPI:004811A7`
`pop edi`

`SDPI:004811A8`
`pop ebx`

`SDPI:004811A9`
`add esp, 20h`

`SDPI:004811AC`
`retn`

`SDPI:004811AC`
`Erase_PEHEADER endp ; sp = -24h`

`SDPI:004811F8`
`; **** S U B R O U T I N E ****`

`SDPI:004811F8`

`SDPI:004811F8`

`SDPI:004811F8`
`; int __stdcall DeCrypt_Functions(int fromDeAddr,int siz2,int,int size1,DWORD Stack_to,DWORD Stack_from)`

`SDPI:004811F8`
`DeCrypt_Functions proc near ; CODE XREF: Disposal_IMP+152 p`

`SDPI:004811F8`
`; Disposal_IMP+193 p ...`

`SDPI:004811F8`

`SDPI:004811F8`
`fromDeAddr = dword ptr 4`

`SDPI:004811F8`
`siz2 = dword ptr 8`

`SDPI:004811F8`
`arg_8 = dword ptr 0Ch`

`SDPI:004811F8`
`size1 = dword ptr 10h`

`SDPI:004811F8`
`Stack_to = dword ptr 14h`

`SDPI:004811F8`
`Stack_from = dword ptr 18h`

`SDPI:004811F8`

`SDPI:004811F8`
`mov eax, [esp+Stack_from]`

`SDPI:004811FC`
`push esi`

`SDPI:004811FD`
`mov esi, [esp+4+Stack_to]`

`SDPI:00481201`
`push edi`

`SDPI:00481202`
`push 102h ; size`

`SDPI:00481207`
`push eax ; from`

`SDPI:00481208`
`push esi ; to`

`SDPI:00481209`
`call RtlMoveMemory ; 复制内存`

`SDPI:0048120E`
`push esi`

`SDPI:0048120F`
`mov esi, [esp+0Ch+siz2]`

`SDPI:00481213`
`mov edi, [esp+0Ch+fromDeAddr]`

`SDPI:00481217`
`push esi`

`SDPI:00481218`
`push edi`

`SDPI:00481219`
`call sub_4805C2`

`SDPI:0048121E`
`mov ecx, [esp+14h+size1]`

`SDPI:00481222`
`mov edx, [esp+14h+arg_8]`

`SDPI:00481226`
`add esp, 0Ch`

`SDPI:00481229`
`push ecx`

`SDPI:0048122A`
`push edx`

`SDPI:0048122B`
`push esi`

`SDPI:0048122C`
`push edi`

`SDPI:0048122D`
`call sub_4811AD`

`SDPI:00481232`
`pop edi`

`SDPI:00481233`
`mov eax, 1`

`SDPI:00481238`
`pop esi`

`SDPI:00481239`
`retn 18h`

`SDPI:00481239`
`DeCrypt_Functions endp`

`SDPI:00481275`
`; **** S U B R O U T I N E ****`

`SDPI:00481275`

`SDPI:00481275`
`; 判断 MD5 效验值是否相等`

`SDPI:00481275`
`; 如果相等则 eax==edi 的值`

`SDPI:00481275`
`; 不相等返回 FE5F3AFE`

`SDPI:00481275`

`SDPI:00481275`
`CMP_HASH_481275 proc near ; CODE XREF: SDPI:0047B458 j`

`SDPI:00481275`

`SDPI:00481275`
`RetBuffer = dword ptr -68h`

`SDPI:00481275`
`InBuffer = dword ptr -58h`

`SDPI:00481275`

`SDPI:00481275`
`sub esp, 68h`

`SDPI:00481278`
`push edi`

`SDPI:00481279`
`call GetTickCount`

`SDPI:0048127E`
`mov edi, eax`

`SDPI:00481280`
`cmp edi, 0FE5F3AFEh`

`SDPI:00481286`
`jnz short loc_48128D`

`SDPI:00481288`
`mov edi, 0FE5F3AFFh`

`SDPI:0048128D`

`SDPI:0048128D`
`loc_48128D: ; CODE XREF: CMP_HASH_481275+11 j`

`SDPI:0048128D`
`push esi`

`SDPI:0048128E`
`call Get_BASE`

`SDPI:00481293`
`mov esi, eax`

`SDPI:00481295`
`lea eax, [esp+70h+InBuffer]`

`SDPI:00481299`
`push eax ; initBuffer`

`SDPI:0048129A`
`call INIT_MD5`

`SDPI:0048129F`
`lea ecx, [esi+0B8h]`

`SDPI:004812A5`
`push 10h ; BufferSize`

`SDPI:004812A7`
`lea edx, [esp+78h+InBuffer]`

`SDPI:004812AB`
`push ecx ; CalcBuffer`

`SDPI:004812AC`
`push edx ; RetBuffer`

`SDPI:004812AD`
`call Hash_MD5`

`SDPI:004812B2`
`lea eax, [esp+80h+InBuffer]`

`SDPI:004812B6`
`lea ecx, [esp+80h+RetBuffer]`

`SDPI:004812BA`
`push eax ; InBuffer`

`SDPI:004812BB`
`push ecx ; RetBuffer`

`SDPI:004812BC`
`call Calculate_MD5`

`SDPI:004812C1`
`add esp, 18h`

`SDPI:004812C4`
`add esi, 0D0h`

`SDPI:004812CA`
`lea edx, [esp+70h+RetBuffer]`

`SDPI:004812CE`
`push 10h ; size`

`SDPI:004812D0`
`push esi ; Original_value`

`SDPI:004812D1`
`push edx ; Calculate_value`

`SDPI:004812D2`
`call Compare_HASH ; 把 Calculate_value 改为 Original_value`

`SDPI:004812D2`
`; 就通过检测了`

`SDPI:004812D7`
`test eax, eax`

`SDPI:004812D9`
`pop esi`

`SDPI:004812DA`
`mov eax, 0FE5F3AFEh`
