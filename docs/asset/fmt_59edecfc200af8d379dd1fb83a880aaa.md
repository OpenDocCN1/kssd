
:0040D812 `FFD7`                     `call edi`

:0040D814 `83C404`                  `add esp, 00000004`

:0040D817 `85C0`                    `test eax, eax`

:0040D819 `7405`                    `je 0040D820`

:0040D81B `8B580C`                  `mov ebx, dword ptr [eax+0C]`

:0040D81E `EB02`                    `jmp 0040D822`

`* Referenced by a (U)nconditional or (C)onditional Jump at Address:`

`|:0040D819(C)`

`|`

:0040D820 `33DB`                    `xor ebx, ebx`

`* Referenced by a (U)nconditional or (C)onditional Jump at Address:`

`|:0040D81E(U)`

`|`

:0040D822 `68B8FC6000`             `push 0060FCB8`

:0040D827 `FFD7`                    `call edi`

:0040D829 `83C404`                  `add esp, 00000004`

:0040D82C `85C0`                    `test eax, eax`

:0040D82E `7406`                    `je 0040D836`

:0040D830 `8B7010`                  `mov esi, dword ptr [eax+10]`

:0040D833 `46`                      `inc esi`

:0040D834 `EB02`                    `jmp 0040D838`

`* Referenced by a (U)nconditional or (C)onditional Jump at Address:`

`|:0040D82E(C)`

`|`

:0040D836 `33F6`                    `xor esi, esi`

`* Referenced by a (U)nconditional or (C)onditional Jump at Address:`

`|:0040D834(U)`

`|`

:0040D838 `68B8FC6000`             `push 0060FCB8`

:0040D83D `FFD7`                    `call edi`

:0040D83F `83C404`                  `add esp, 00000004`

:0040D842 `85C0`                    `test eax, eax`

:0040D844 `740A`                    `je 0040D850`

:0040D846 `8B4014`                  `mov eax, dword ptr [eax+14]`

:0040D849 `056C070000`             `add eax, 0000076C`

:0040D84E `EB02`                    `jmp 0040D852`

`* Referenced by a (U)nconditional or (C)onditional Jump at Address:`

`|:0040D844(C)`

`|`

:0040D850 `33C0`                    `xor eax, eax`

`* Referenced by a (U)nconditional or (C)onditional Jump at Address:`

`|:0040D84E(U)`

`|`

:0040D852 `8B4D04`                  `mov ecx, dword ptr [ebp+04]`

:0040D855 `53`                      `push ebx`

:0040D856 `56`                      `push esi`

:0040D857 `50`                      `push eax`

:0040D858 `51`                      `push ecx`

:0040D859 `8D54246C`                `lea edx, dword ptr [esp+6C]`

:0040D85D `6838DF5900`             `push 0059DF38`

:0040D862 `52`                      `push edx`

`* Reference To: MSVCR71.sprintf, Ord:0300h`

                     `|`

:0040D863 `FF15F88F5900`           `Call dword ptr [00598FF8]`

:0040D869 `8B442438`                `mov eax, dword ptr [esp+38]`

:0040D86D `83C418`                  `add esp, 00000018`

:0040D870 `85C0`                    `test eax, eax`

:0040D872 `7409`                    `je 0040D87D`

:0040D874 `50`                      `push eax`

`* Reference To: MFC71.Ordinal:010A, Ord:010Ah`

                     `|`

:0040D875 `E8D4FD1600`             `Call 0057D64E`

:0040D87A `83C404`                  `add esp, 00000004`

`* Referenced by a (U)nconditional or (C)onditional Jump at Address:`

`|:0040D872(C)`

`|`

:0040D87D `6A30`                    `push 00000030`

:0040D87F `6818DF5900`             `push 0059DF18`

:0040D884 `8D442464`                `lea eax, dword ptr [esp+64]`

:0040D888 `50`                      `push eax`

:0040D889 `6A00`                    `push 00000000`

`* Reference To: USER32.MessageBoxA, Ord:01DEh`

                     `|`

:0040D88B `FF1560925900`           `Call dword ptr [00599260]`

:0040D891 `C605D4675F0000`         `mov byte ptr [005F67D4], 00`

`* Referenced by a (U)nconditional or (C)onditional Jump at Address:`

`|:0040D7F1(C)`

`|`

:0040D898 `8B4C2414`                `mov ecx, dword ptr [esp+14]`

:0040D89C `8B542410`                `mov edx, dword ptr [esp+10]`

:0040D8A0 `6A00`                    `push 00000000`

:0040D8A2 `6880510100`             `push 00015180`

:0040D8A7 `51`                      `push ecx`

:0040D8A8 `52`                      `push edx`

:0040D8A9 `E8B2061700`             `call 0057DF60`

:0040D8AE `8B8C24A4010000`         `mov ecx, dword ptr [esp+000001A4]`

:0040D8B5 `8901`                    `mov dword ptr [ecx], eax`

:0040D8B7 `8D4C2424`                `lea ecx, dword ptr [esp+24]`

:0040D8BB `C7842494010000FFFFFFFF` `mov dword ptr [esp+00000194], FFFFFFFF`

:0040D8C6 `E865C60F00`             `call 00509F30`

`* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:`

`|:0040D74D(C), :0040D76D(C)`

`|`

:0040D8CB `32C0`                    `xor al, al`

:0040D8CD `E908020000`             `jmp 0040DADA`

`* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:`

`|:0040D7DA(C), :0040D7E4(C)`

`|`

:0040D8D2 `807C2EF40D`             `cmp byte ptr [esi+ebp-0C], 0D`

:0040D8D7 `7401`                    `je 0040D8DA`

:0040D8D9 `45`                      `inc ebp`

`* Referenced by a (U)nconditional or (C)onditional Jump at Address:`

`|:0040D8D7(C)`

`|`

`* Reference To: MSVCR71.strncpy, Ord:030Fh`

                     `|`

:0040D8DA `8B3DFC8F5900`           `mov edi, dword ptr [00598FFC]`

:0040D8E0 `6A04`                    `push 00000004`

:0040D8E2 `8D542EEC`                `lea edx, dword ptr [esi+ebp-14]`

:0040D8E6 `52`                      `push edx`

:0040D8E7 `680CBF5F00`             `push 005FBF0C`

:0040D8EC `FFD7`                    `call edi`

:0040D8EE `6A02`                    `push 00000002`

:0040D8F0 `8D442EF0`                `lea eax, dword ptr [esi+ebp-10]`

:0040D8F4 `50`                      `push eax`

:0040D8F5 `6800BF5F00`             `push 005FBF00`

:0040D8FA `FFD7`                    `call edi`

:0040D8FC `6A02`                    `push 00000002`

:0040D8FE `8D4C2EF2`                `lea ecx, dword ptr [esi+ebp-0E]`

:0040D902 `51`                      `push ecx`

:0040D903 `68F4BE5F00`             `push 005FBEF4`

:0040D908 `FFD7`                    `call edi`

:0040D90A `68F0BE5F00`             `push 005FBEF0`

:0040D90F `680CDF5900`             `push 0059DF0C`

:0040D914 `680CBF5F00`             `push 005FBF0C`

:0040D919 `FFD3`                    `call ebx`

:0040D91B `68ECBE5F00`             `push 005FBEEC`

:0040D920 `680CDF5900`             `push 0059DF0C`

:0040D925 `6800BF5F00`             `push 005FBF00`

:0040D92A `FFD3`                    `call ebx`

:0040D92C `68E8BE5F00`             `push 005FBEE8`

:0040D931 `680CDF5900`             `push 0059DF0C`

:0040D936 `68F4BE5F00`             `push 005FBEF4`

:0040D93B `FFD3`                    `call ebx`

:0040D93D `A1F0BE5F00`             `mov eax, dword ptr [005FBEF0]` `//取注册码中后 19 位头四位数字转成 16 进制数`

:0040D942 `83C448`                  `add esp, 00000048`

:0040D945 `3D540B0000`             `cmp eax, 00000B54` `//要大于 B54 否则出错误`

:0040D94A `7E05`                    `jle 0040D951`

:0040D94C `B8540B0000`             `mov eax, 00000B54`

..............................

................................

`///关键调用结束`

`/////////////////////////////////////////////////////////////////////////////////////////////////////////`

`//插入程序功能限制函数调用`

:0049ABB0 `83EC74`                  `sub esp, 00000074`

:0049ABB3 `A1B0AB5F00`             `mov eax, dword ptr [005FABB0]`

:0049ABB8 `89442470`                `mov dword ptr [esp+70], eax`

:0049ABBC `33C0`                    `xor eax, eax`

:0049ABBE `53`                      `push ebx`

:0049ABBF `89442408`                `mov dword ptr [esp+08], eax`

:0049ABC3 `89442404`                `mov dword ptr [esp+04], eax`

:0049ABC7 `88442410`                `mov byte ptr [esp+10], al`

:0049ABCB `8D44240C`                `lea eax, dword ptr [esp+0C]`

:0049ABCF `50`                      `push eax`

:0049ABD0 `A1F0686000`             `mov eax, dword ptr [006068F0]`

:0049ABD5 `8D4C2408`                `lea ecx, dword ptr [esp+08]`

:0049ABD9 `51`                      `push ecx`

:0049ABDA `8D542410`                `lea edx, dword ptr [esp+10]`

:0049ABDE `52`                      `push edx`

:0049ABDF `50`                      `push eax`

:0049ABE0 `C744241CFEFFFFFF`        `mov [esp+1C], FFFFFFFE`
