
## 004133DF

`BB0B000000` `mov ebx, 0000000B` `ebx=B`

`004133E4` `89742440` `mov dword ptr [esp+40], esi`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| `0041341B(C)`
|
`004133E8` `0FBE042E` `movsx eax, byte ptr [esi+ebp]` `先符号扩展,再将注册码 的第二位传送给 eax`

`004133EC` `83F85A` `cmp eax, 0000005A` `比较 eax 和 5A`

`004133EF` `7F17` `jg 00413408` `大于就跳`

`004133F1` `B95A000000` `mov ecx, 0000005A` `小于就 ecx=5A`

`004133F6` `2BC8` `sub ecx, eax` `ecx=ecx-eax`

`004133F8` `B856555555` `mov eax, 55555556` `eax=55555556`

`004133FD` `F7E9` `imul ecx` `eax=eax*ecx`

`004133FF` `8BC2` `mov eax, edx` `eax=edx`

`00413401` `C1EA1F` `shr edx, 1F` `edx 逻辑右移 1F`

`00413404` `03C2` `add eax, edx` `eax=eax+edx`

`00413406` `EB0D` `jmp 00413415` `跳走`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| `004133EF(C)`
|
`00413408` `83F865` `cmp eax, 00000065` `比较 eax 和 65`

`0041340B` `7C08` `jl 00413415` `小于就跳走`

`0041340D` `83C09B` `add eax, FFFFFF9B` `否则 eax=eax+FFFFFF9B`

`00413410` `99` `cdq` `双字扩展(把 EAX 中的字的 符号扩展到 EDX 中去)`

`00413411` `2BC2` `sub eax, edx` `eax=eax-dx`

`00413413` `D1F8` `sar eax, 1` `eax 算术右移 1`

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

| `00413406(U), 0041340B(C)`
|
`00413415` `0430` `add al, 30` `al=al+30`

`00413417` `8806` `mov byte ptr [esi], al` `al 给[esi]`

`00413419` `46` `inc esi` `esi 加一`

`0041341A` `4B` `dec ebx` `ebx 减一`

`0041341B` `75CB` `jne 004133E8` `循环；即得新注册码`

`0041341D` `8A5C2424` `mov bl, byte ptr [esp+24]`

`00413421` `C6470C00` `mov [edi+0C], 00`

`00413425` `881F` `mov byte ptr [edi], bl` `将注册码的第一位补到新 注册码的前面`

`00413427` `33C0` `xor eax, eax`

`00413429` `B901000000` `mov ecx, 00000001` `|` `|` `将新注册码累乘` `|` `值给 eax`

* Referenced by a (U)nconditional or (C)onditional Jump at Address: `|`

| `0041343B(C)` `|`
| `|`
`0041342E` `0FBE1439` `movsx edx, byte ptr [ecx+edi]` `|`

`00413432` `0FAFC1` `imul eax, ecx` `|`

`00413435` `03C2` `add eax, edx` `|`

`00413437` `41` `inc ecx` `|`

`00413438` `83F90C` `cmp ecx, 0000000C` `|`

`0041343B` `7CF1` `jl 0041342E`

`0041343D` `5F` `pop edi`

`0041343E` `5E` `pop esi`

`0041343F` `85C0` `test eax, eax` `检查 eax`

`00413441` `5D` `pop ebp`

`00413442` `7D02` `jge 00413446` `eax 大于等于 43446 就跳`

`00413444` `F7D8` `neg eax` `？？？`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| `00413442(C)`
|
`00413446` `99` `cdq` `双字扩展`

`00413447` `B91A000000` `mov ecx, 0000001A` `ecx=1A`

`0041344C` `F7F9` `idiv ecx` `eax=eax/ecx`

`0041344E` `0FBEC3` `movsx eax, bl` `先符号扩展,再把 bl 给 al`

`00413451` `5B` `pop ebx`

`00413452` `83C241` `add edx, 00000041` `edx=edx+41`

`00413455` `3BC2` `cmp eax, edx` `比较 eax 和 edx`

`00413457` `7512` `jne 0041346B` `不相等就完蛋`

`00413459` `8B4C2430` `mov ecx, dword ptr [esp+30]` `将新注册码的第一位地址 给 ecx`

`0041345D` `803932` `cmp byte ptr [ecx], 32` `比较 ecx 和 32`

`00413460` `7509` `jne 0041346B` `不相等就完蛋`

`00413462` `B801000000` `mov eax, 00000001`

`00413467` `83C428` `add esp, 00000028`

`0041346A` `C3` `ret`

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

| `00413457(C), 00413460(C)`
|
`0041346B` `33C0` `xor eax, eax`

`0041346D` `83C428` `add esp, 00000028`

`00413470` `C3` `ret`

--------------------------------------------------------

* Referenced by a CALL at Addresses:

| `0040808A , 004133CA`
|
`004132F0` `56` `push esi`

`004132F1` `8B74240C` `mov esi, dword ptr [esp+0C]`

`004132F5` `57` `push edi`

`004132F6` `8BFE` `mov edi, esi`

`004132F8` `83C9FF` `or ecx, FFFFFFFF`

`004132FB` `33C0` `xor eax, eax`

`004132FD` `F2` `repnz`

`004132FE` `AE` `scasb`

`004132FF` `F7D1` `not ecx`

`00413301` `49` `dec ecx`

`00413302` `F6C103` `test cl, 03`

`00413305` `7403` `je 0041330A`

`00413307` `5F` `pop edi`

`00413308` `5E` `pop esi`

`00413309` `C3` `ret`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| `00413305(C)`
|
`0041330A` `8A06` `mov al, byte ptr [esi]` `依次取注册码到 al`

`0041330C` `84C0` `test al, al` `检查是否是零`

`0041330E` `7467` `je 00413377` `是就完蛋！`

`00413310` `8B7C240C` `mov edi, dword ptr [esp+0C]`

`00413314` `53` `push ebx`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| `0041336A(C)`
|
`00413315` `50` `push eax`

`00413316` `46` `inc esi`

`00413317` `E894FFFFFF` `call 004132B0` `也需要追入！（是换算）`

`0041331C` `8AD8` `mov bl, al` `al 给 bl`

`0041331E` `8A06` `mov al, byte ptr [esi]` `下一个注册码给 al`

`00413320` `50` `push eax`

`00413321` `46` `inc esi` `esi 加一`

`00413322` `E889FFFFFF` `call 004132B0` `换算`

`00413327` `8AC8` `mov cl, al` `al 给 cl`

`00413329` `240F` `and al, 0F` `al and 0f`

`0041332B` `C0F904` `sar cl, 04` `cl 算术右移 4`
