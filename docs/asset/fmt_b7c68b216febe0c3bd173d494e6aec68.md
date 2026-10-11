
0034AF72 `EB 01` `jmp short 0034AF75`

。。。。。。

0034AFA3 `E8 02000000` `call 0034AFAA`

0034AFA8 `0F35` `sysexit`

0034AFAA `0F31` `rdtsc`

0034AFAC `83C4 04` `add esp,4`

0034AFAF `2BC1` `sub eax,ecx`

0034AFB1 `3D 00000200` `cmp eax,20000`

0034AFB6 `76 04` `jbe short 0034AFBC` `////改为 jmp 0034AFBC`

0034AFBC `59` `pop ecx`

0034AFBD `5A` `pop edx`

0034AFBE `58` `pop eax`

0034AFBF `EB 30` `jmp short 0034AFF1`

0034AFF1 `2946 04` `sub dword ptr ds:[esi+4],eax` `//终于不再时间检测了！！`

0034AFF4 `8DB5 36FD4000` `lea esi,dword ptr ss:[ebp+40FD>`

0034AFFA `B8 3AFD4000` `mov eax,40FD3A`

0034AFFF `8906` `mov dword ptr ds:[esi],eax`

0034B001 `8D85 558B4000` `lea eax,dword ptr ss:[ebp+408B>`

0034B007 `50` `push eax`

0034B008 `8B85 05F64000` `mov eax,dword ptr ss:[ebp+40F6>; KERNEL32.GetCurrentThread`

0034B00E `E9 20620000` `jmp 00351233`

0034B014 `6A 00` `push 0`

0034B016 `50` `push eax`

0034B017 `8D85 BA8C4000` `lea eax,dword ptr ss:[ebp+408C>`

0034B01D `50` `push eax`

0034B01E `68 00FE98B4` `push B498FE00`

0034B023 `50` `push eax`

0034B024 `E8 5D000000` `call 0034B086`

。。。。。。。

0034B16D `8B85 B5F64000` `mov eax,dword ptr ss:[ebp+40F6>; KERNEL32.SetThreadPriority`

0034B173 `E9 BB600000` `jmp 00351233`

0034B179 `^\\E9 F6C0FFFF` `jmp 00347274`

00347274 `50` `push eax`

00347275 `64:FF35 0000000>` `push dword ptr fs:[0]`

0034727C `83EC 08` `sub esp,8`

0034727F `B8 1FF44000` `mov eax,40F41F`

00347284 `50` `push eax`

00347285 `64:FF35 0000000>` `push dword ptr fs:[0]`

0034728C `64:8925 0000000>` `mov dword ptr fs:[0],esp`

00347293 `83C4 10` `add esp,10`

00347296 `B8 1FF44000` `mov eax,40F41F`

0034729B `50` `push eax`

0034729C `64:FF35 0000000>` `push dword ptr fs:[0]`

003472A3 `64:8925 0000000>` `mov dword ptr fs:[0],esp`

003472AA `B8 1FF44000` `mov eax,40F41F`

003472AF `50` `push eax`

003472B0 `64:FF35 0000000>` `push dword ptr fs:[0]`

003472B7 `64:8925 0000000>` `mov dword ptr fs:[0],esp`

003472BE `33C0` `xor eax,eax`

003472C0 `40` `inc eax`

003472C1 `83C4 10` `add esp,10`

003472C4 `64:8F05 0000000>` `pop dword ptr fs:[0]`

003472CB `58` `pop eax`

003472CC `FF85 46F44000` `inc dword ptr ss:[ebp+40F446]`

003472D2 `50` `push eax`

003472D3 `52` `push edx`

003472D4 `51` `push ecx`

003472D5 `EB 01` `jmp short 003472D8`

。。。。。。

00347324 `55` `push ebp`

00347325 `8BEC` `mov ebp,esp`

00347327 `810424 87880600` `add dword ptr ss:[esp],68887`

0034732E `81EC 80000000` `sub esp,80`

00347334 `81AC24 80000000>` `sub dword ptr ss:[esp+80],6888>`

0034733F `8BE5` `mov esp,ebp`

00347341 `5D` `pop ebp`

00347342 `C3` `retn` `返回到 003472EC`

。。。。。。

00347306 `E8 02000000` `call 0034730D` `//又开始了时间检测`

0034730B `0F35` `sysexit`

0034730D `0F31` `rdtsc`

0034730F `83C4 04` `add esp,4`

00347312 `2BC1` `sub eax,ecx`

00347314 `3D 00000400` `cmp eax,40000`

00347319 `76 04` `jbe short 0034731F` `//改为 jmp 0034731F`

0034731F `59` `pop ecx`

00347320 `5A` `pop edx`

00347321 `58` `pop eax`

00347322 `EB 33` `jmp short 00347357`

。。。。。。

0034735C `^\\74 FB` `je short 00347359`

0034735E `83BD 96F44000 0>` `cmp dword ptr ss:[ebp+40F496],0`

00347365 `0F84 28010000` `je 00347493`

0034736B `50` `push eax`

0034736C `64:FF35 0000000>` `push dword ptr fs:[0]`

00347373 `83EC 08` `sub esp,8`

00347376 `B8 1FF44000` `mov eax,40F41F`

0034737B `50` `push eax`

0034737C `64:FF35 0000000>` `push dword ptr fs:[0]`

00347383 `64:8925 0000000>` `mov dword ptr fs:[0],esp`

0034738A `83C4 10` `add esp,10`

0034738D `B8 1FF44000` `mov eax,40F41F`

00347392 `50` `push eax`

00347393 `64:FF35 0000000>` `push dword ptr fs:[0]`

0034739A `64:8925 0000000>` `mov dword ptr fs:[0],esp`

003473A1 `B8 1FF44000` `mov eax,40F41F`

003473A6 `50` `push eax`

003473A7 `64:FF35 0000000>` `push dword ptr fs:[0]`

003473AE `64:8925 0000000>` `mov dword ptr fs:[0],esp`

003473B5 `33C0` `xor eax,eax`

003473B7 `40` `inc eax`

003473B8 `83C4 10` `add esp,10`

003473BB `64:8F05 0000000>` `pop dword ptr fs:[0]`

003473C2 `58` `pop eax`

003473C3 `8D85 C91A4000` `lea eax,dword ptr ss:[ebp+401AC9]`

003473C9 `EB 02` `jmp short 003473CD`

003473CD `0385 96F44000` `add eax,dword ptr ss:[ebp+40F496]`

003473D3 `E8 03000000` `call 003473DB`

003473DB `83C4 04` `add esp,4`

003473DE `894424 EC` `mov dword ptr ss:[esp-14],eax`

003473E2 `EB 03` `jmp short 003473E7`

003473E7 `61` `popad`

003473E8 `50` `push eax`

003473E9 `52` `push edx`

003473EA `51` `push ecx`

003473EB `EB 01` `jmp short 003473EE`

。。。。。。

0034743A `55` `push ebp`

0034743B `8BEC` `mov ebp,esp`

0034743D `810424 87880600` `add dword ptr ss:[esp],68887`

00347444 `81EC 80000000` `sub esp,80`

0034744A `81AC24 80000000>` `sub dword ptr ss:[esp+80],68887`

00347455 `8BE5` `mov esp,ebp`

00347457 `5D` `pop ebp`

00347458 `C3` `retn` `返回到 00347402`

。。。。。。。

0034741C `E8 02000000` `call 00347423` `//又来了！！`

00347421 `0F35` `sysexit`

00347423 `0F31` `rdtsc`

00347425 `83C4 04` `add esp,4`

00347428 `2BC1` `sub eax,ecx`

0034742A `3D 00000400` `cmp eax,40000`

0034742F `76 04` `jbe short 00347435` `//改为 jmp 00347435`

00347431 `83C4 0C` `add esp,0C`

00347434 `C3` `retn`

00347435 `59` `pop ecx`

00347436 `5A` `pop edx`

00347437 `58` `pop eax`

00347438 `EB 33` `jmp short 0034746D`

。。。。。。

开始抽代码：

0035526C `55` `push ebp` `★push ebp`

0035526D `EB 00` `jmp short 0035526F`

0035526F `54` `push esp`

00355270 `5D` `pop ebp` `★mov ebp, esp`

00355271 `EB 00` `jmp short 00355273`

00355273 `54` `push esp`

00355274 `830424 F0` `add dword ptr ss:[esp],-10`

00355278 `5C` `pop esp` `★add esp, -10`

00355279 `EB 01` `jmp short 0035527C`

0035527C `68 98FC4C00` `push 4CFC98`

00355281 `58` `pop eax` `★mov eax,4CFC98`

00355282 `EB 02` `jmp short 00355286`

。。。。。

00355429 `68 D8014D00` `push 4D01D8` `//伪 OEP`

0035542E `C3` `retn`

（未完代续）`![img](img/0cf410da109ac47387dd713fcee2cd79.jpg)`

`4dd466d0e1e0cc123afbe504da2462ca`
