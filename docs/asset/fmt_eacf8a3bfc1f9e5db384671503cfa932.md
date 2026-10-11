
`003B2A72` `03BD DCD34100` `add` `edi, dword ptr ss:[ebp+41D3DC]` `+ImageBase,Get VA`
`; offset to function name ( and hint)`
`003B2A99` `8B06` `mov` `eax, dword ptr ds:[esi]` `; 要 do hint...`
`003B2AA0` `83F8 00` `cmp` `eax, 0`
`003B2AA3` `/75 29` `jnz` `short 003B2ACE` `; 未完?`
`003B2ABC` `83C3 14` `add` `ebx, 14`
`003B2AC4` `^\\E9 13FBFFFF` `jmp` `003B25DC` `; 下一个 IID`
`003B2B00` `807E 03 80` `cmp` `byte ptr ds:[esi+3], 80` `; is imported by ordinal?`
`003B2B04` `75 6C` `jnz` `short 003B2B72`
`; ord...`
`003B2B0B` `33C0` `xor` `eax, eax`
`003B2B12` `66:8706` `xchg` `word ptr ds:[esi], ax`
`003B2B42` `50` `push` `eax`
`003B2B43` `FFB5 7FCC4100` `push` `dword ptr ss:[ebp+41CC7F]`
`003B2B49` `FF95 C53D4200` `call` `dword ptr ss:[ebp+423DC5]`
`003B2B54` `8BC8` `mov` `ecx, eax`
`003B2B6D` `/E9 84000000` `jmp` `003B2BF6`
`; str...`
`003B2B77` `33C0` `xor` `eax, eax`
`003B2B7E` `8706` `xchg` `dword ptr ds:[esi], eax`
`003B2B85` `0385 DCD34100` `add` `eax, dword ptr ss:[ebp+41D3DC]` `+ImageBase,这句话打过无数次了,累!!!`
`003B2BA2` `83C0 02` `add` `eax, 2` `; 跳过 hint 值(sizeof word == 2)`
`003B2BD2` `50` `push` `eax`
`003B2BD3` `FFB5 7FCC4100` `push` `dword ptr ss:[ebp+41CC7F]`
`003B2BD9` `FF95 C53D4200` `call` `dword ptr ss:[ebp+423DC5]` `kernel32.GetProcAddress`
`003B2C00` `80BD 78894100 0>cmp` `byte ptr ss:[ebp+418978], 1` `; 特殊函数?`
`003B2C07` `/0F85 87020000` `jnz` `003B2E94` `; magic jump...改为 jmp 吧,省得心烦`
`; --------------------------- 加密输入表 ----------------------------------------`
`; 具体不说了,跟到这里有点筋疲力尽了`
`003B2C3F` `60` `pushad`
`003B2C45` `8BF8` `mov` `edi, eax`
`003B2C63` `8B85 A7DC4100` `mov` `eax, dword ptr ss:[ebp+41DCA7]`
`003B2C6E` `3D F4010000` `cmp` `eax, 1F4`
`003B2C73` `/75 1A` `jnz` `short 003B2C8F`
`003B2C7A` `89BD 70894100` `mov` `dword ptr ss:[ebp+418970], edi`
`003B2C85` `/E9 C9010000` `jmp` `003B2E53`
`003B2CC1` `B9 04000000` `mov` `ecx, 4`
`003B2CCB` `33D2` `xor` `edx, edx`
`003B2CD2` `F7E1` `mul` `ecx`
`003B2CD9` `BE B1DC4100` `mov` `esi, 41DCB1`
`003B2CE3` `03F5` `add` `esi, ebp`
`003B2CFC` `33BD 20FA4100` `xor` `edi, dword ptr ss:[ebp+41FA20]`
`003B2D07` `893C06` `mov` `dword ptr ds:[esi+eax], edi`
`003B2D26` `8B85 A7DC4100` `mov` `eax, dword ptr ss:[ebp+41DCA7]`
`003B2D31` `B9 0B000000` `mov` `ecx, 0B`
`003B2D3B` `33D2` `xor` `edx, edx`
`003B2D42` `F7E1` `mul` `ecx`
`003B2D49` `BF 85E44100` `mov` `edi, 41E485`
`003B2D7B` `03FD` `add` `edi, ebp`
`003B2DAA` `03F8` `add` `edi, eax`
`003B2DB6` `89BD 70894100` `mov` `dword ptr ss:[ebp+418970], edi`
`003B2DEE` `BE A6DC4100` `mov` `esi, 41DCA6`
`003B2DF8` `03F5` `add` `esi, ebp`
`003B2E27` `B9 0B000000` `mov` `ecx, 0B`
`003B2E31` `F3:A4` `rep movs` `byte ptr es:[edi], byte ptr ds:[esi]`
`003B2E38` `FF85 A7DC4100` `inc` `dword ptr ss:[ebp+41DCA7]` `; 计数器,被加密的应该是 push count...push address...ret`
`003B2E43` `8BBD 70894100` `mov` `edi, dword ptr ss:[ebp+418970]`
`003B2E6A` `61` `popad`
`003B2E70` `8B8D 70894100` `mov` `ecx, dword ptr ss:[ebp+418970]`
`003B2E92` `/EB 1E` `jmp` `short 003B2EB2`
`; -----------------------------------------------------------------------------`
`003B2EAB` `8BC8` `mov` `ecx, eax` `; NETAPI32.Netbios`
`; 函数地址`
`003B2ECE` `39BD 33D64100` `cmp` `dword ptr ss:[ebp+41D633], edi`
`003B2ED4` `76 10` `jbe` `short 003B2EE6` `; 不知道,没跳`
`003B2EDB` `89BD 33D64100` `mov` `dword ptr ss:[ebp+41D633], edi`
`003B2EEB` `39BD 2FD64100` `cmp` `dword ptr ss:[ebp+41D62F], edi`
`003B2EF1` `73 10` `jnb` `short 003B2F03` `; 不知道,还是没跳`
`003B2EF8` `89BD 2FD64100` `mov` `dword ptr ss:[ebp+41D62F], edi`
`003B2F24` `890F` `mov` `dword ptr ds:[edi], ecx` `; 填充 IAT`
`003B2F3D` `83C6 04` `add` `esi, 4`
`003B2F45` `83C7 04` `add` `edi, 4`
`; 移动 Thunk 指针`
`003B2F4D` `^\\E9 42FBFFFF` `jmp` `003B2A94` `; 循环---`
`; ---------------------------------------------------------------------------`
`; 从老上边的 003B25F6 /0F84 B0090000 je 003B2FAC 出来,完成 patch iat`
`003B2FB1` `E8 16000000` `call` `003B2FCC` `; 取 User32.GetClassNameA...居然专门弄个 call...`
`003B2FBB` `61` `popad` `; 解放了`
`003B2FC1` `C3` `ret`

[part 8]
`img/620448a77d884bd6b2c4a7cfef437501.jpg`

`; --------------------------- 不明飞行物 -----------------------------------`
`003B0FD3` `8D85 DA5C4000` `lea` `eax, dword ptr ss:[ebp+405CDA]`
`003B1011` `E8 96200000` `call` `003B30AC` `; 没看懂`

`; 处理 调用 表 & 跳转 表 ( 还原 指针 )`

`; --------------call dword ptr ds:[xxxxxxxx]，其中 xxxxxxxx 是 80xxxxxxxx----------------`
`003B108C` `60` `pushad`
`003B10A4` `8B9D 08D64100` `mov` `ebx, dword ptr ss:[ebp+41D608]`
`003B10AF` `039D DCD34100` `add` `ebx, dword ptr ss:[ebp+41D3DC]` `; 还是 Imagebase`
`003B110F` `BE E4D34100` `mov` `esi, 41D3E4` `; 块表`
`003B1119` `03F5` `add` `esi, ebp`
`003B1120` `8B9D DCD34100` `mov` `ebx, dword ptr ss:[ebp+41D3DC]` `; Try.00400000`
`003B112B` `031E` `add` `ebx, dword ptr ds:[esi]`
`; ebx -> a section,应该是代码段`
`003B1132` `8BFB` `mov` `edi, ebx` `; Try.00401000`
`003B1139` `8B4E 08` `mov` `ecx, dword ptr ds:[esi+8]` `; VSize`
`003B1141` `83F9 00` `cmp` `ecx, 0`
