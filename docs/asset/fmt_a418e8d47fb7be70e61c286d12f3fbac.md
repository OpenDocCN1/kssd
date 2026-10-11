
:00407E46 `BFFC044400` `mov edi, 004404FC` <--------------用户名

:00407E4B `83C9FF` `or ecx, FFFFFFFF`

:00407E4E `F2` `repnz`

:00407E4F `AE` `scasb`

:00407E50 `8BCA` `mov ecx, edx`

:00407E52 `4F` `dec edi`

:00407E53 `C1E902` `shr ecx, 02`

:00407E56 `F3` `repz`

:00407E57 `A5` `movsd` <----------------把'Multi Clipboard'接在小写的用户名后

:00407E58 `8BCA` `mov ecx, edx`

:00407E5A `8B542418` `mov edx, dword ptr [esp+18]` <---------0x9b((2)的计算结果)

:00407E5E `83E103` `and ecx, 00000003`

:00407E61 `F3` `repz`

:00407E62 `A4` `movsb`

:00407E63 `8A0DFC044400` `mov cl, byte ptr [004404FC]` <---------连接后的第 1 位 0x6c('l')

:00407E69 `33F6` `xor esi, esi`

:00407E6B `3ACB` `cmp cl, bl`

:00407E6D `89742410` `mov dword ptr [esp+10], esi`

:00407E71 `7431` `je 00407EA4`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:00407E9A(C)`

:00407E73 `8BC1` `mov eax, ecx`

:00407E75 `25FF000000` `and eax, 000000FF`

:00407E7A `8D3CC0` `lea edi, dword ptr [eax+8*eax]` <---------9*0x6c==0x3cc

:00407E7D `03C6` `add eax, esi` <---------------------------0x6c+0x0(序号)==0x6c

:00407E7F `8D0478` `lea eax, dword ptr [eax+2*edi]` <---------0x6c+2*0x3cc==0x804

:00407E82 `03D0` `add edx, eax` <---------------------------0x9b+0x804==0x89f

:00407E84 `80F960` `cmp cl, 60` <------------------------第 1 位为字母

:00407E87 `7305` `jnb 00407E8E`

:00407E89 `83C215` `add edx, 00000015`

:00407E8C `EB03` `jmp 00407E91`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:00407E87(C)`

:00407E8E `83EA15` `sub edx, 00000015` <-------------------0x89f-0x15==0x88a

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:00407E8C(U)`

:00407E91 `8A8EFD044400` `mov cl, byte ptr [esi+004404FD]` <----------取第 2 位

:00407E97 `46` `inc esi`

:00407E98 `3ACB` `cmp cl, bl` <-------------------------是否为 0

:00407E9A `75D7` `jne 00407E73` <-----------------------不为 0 则继续循环

:00407E9C `89742410` `mov dword ptr [esp+10], esi` <--------0x384

:00407EA0 `89542418` `mov dword ptr [esp+18], edx` <--------0xc28a<---最后结果

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

****

`0xc28a==049802`

`049802` 的 ASCII 值为 `30 34 39 38 30 32`

+0 +1 +2 +3 +4 +5

—————————

`30 35 3B 3B 34 37` <------------注册码

****

=========================================================================================

* Referenced by a CALL at Address:<----------------------------------计算(2)

`:00407E2B`

:00407FE0 `55` `push ebp`

:00407FE1 `8BEC` `mov ebp, esp`

:00407FE3 `83EC08` `sub esp, 00000008`

:00407FE6 `8B450C` `mov eax, dword ptr [ebp+0C]`

:00407FE9 `53` `push ebx`

:00407FEA `56` `push esi`

:00407FEB `57` `push edi`

:00407FEC `8B7D08` `mov edi, dword ptr [ebp+08]`

:00407FEF `668B08` `mov cx, word ptr [eax]`

:00407FF2 `660FB605FD044400` `movzx ax, byte ptr [004404FD]` <-------用户名的第 2 位 0x61('a')

:00407FFA `8B17` `mov edx, dword ptr [edi]`

:00407FFC `894DF8` `mov dword ptr [ebp-08], ecx`

:00407FFF `660FB60DFE044400` `movzx cx, byte ptr [004404FE]` <-------用户名的第 3 位 0x6e('n')

:00408007 `8955FC` `mov dword ptr [ebp-04], edx`

:0040800A `03C8` `add ecx, eax` <------------------------0x6e+0x61==0xcf

:0040800C `660FB615FC044400` `movzx dx, byte ptr [004404FC]` <-------用户名的第 1 位 0x6c('l')

:00408014 `03CA` `add ecx, edx` <------------------------0xcf+0x6c==0x13b

:00408016 `B856555555` `mov eax, 55555556`

:0040801B `81E1FFFF0000` `and ecx, 0000FFFF`

:00408021 `F7E9` `imul ecx` <-----------------------0x55555556*0x13b==0x69000000d2

:00408023 `8BC2` `mov eax, edx` <------------------------0x69

:00408025 `8BC8` `mov ecx, eax`

:00408027 `C1E91F` `shr ecx, 1F` <-------------------------0x69>>0x1f==0x0

:0040802A `03C1` `add eax, ecx` <------------------------0x69+0x0===0x69

:0040802C `3C30` `cmp al, 30`

:0040802E `7208` `jb 00408038`

:00408030 `3C39` `cmp al, 39`

:00408032 `7704` `ja 00408038` <--------------------------是否位数字

:00408034 `2C30` `sub al, 30` <---------------------------是则减 0x30

:00408036 `EB1A` `jmp 00408052`

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

`:0040802E(C), :00408032(C)`

:00408038 `3C61` `cmp al, 61`

:0040803A `7208` `jb 00408044`

:0040803C `3C7A` `cmp al, 7A`

:0040803E `7704` `ja 00408044` <---------------------是否为字母

:00408040 `2C61` `sub al, 61` <----------------------是则减 0x61

:00408042 `EB0E` `jmp 00408052` `0x69-0x61==0x8`

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

`:00408036(U), :00408042(U), :00408046(C), :0040804E(U)`

:00408052 `8AC8` `mov cl, al` <--------------------------0x8

:00408054 `80F91A` `cmp cl, 1A`

:00408057 `884D0C` `mov byte ptr [ebp+0C], cl`

:0040805A `761F` `jbe 0040807B`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:0040805A(C)`

:0040807B `8B450C` `mov eax, dword ptr [ebp+0C]`

:0040807E `8BC8` `mov ecx, eax`

:00408080 `81E1FF000000` `and ecx, 000000FF` <------------------0x8

:00408086 `49` `dec ecx` <----------------------------0x8-0x1==0x7

:00408087 `83F918` `cmp ecx, 00000018`

:0040808A `0F87EA000000` `ja 0040817A`

:00408090 `FF248DEC814000` `jmp dword ptr [4*ecx+004081EC]` <-----jmp [00408208]

=> jmp 0040815a

=======================================================================================

* Referenced by a CALL at Address:

`:0040815B`

:00407780 `8B4C2404` `mov ecx, dword ptr [esp+04]` <--------0x8

:00407784 `8B15FC044400` `mov edx, dword ptr [004404FC]` <------0x63('l')

:0040778A `81E1FF000000` `and ecx, 000000FF`

:00407790 `81E2FF000000` `and edx, 000000FF`

:00407796 `0FAFD1` `imul edx, ecx` <----------------------0x63*0x8==0x360

:00407799 `B8ABAAAAAA` `mov eax, AAAAAAAB`

:0040779E `8D0C49` `lea ecx, dword ptr [ecx+2*ecx]` <-----0x3*0x8==0x18

:004077A1 `F7E2` `mul edx` <----------------0xaaaaaaab*0x360==0x24000000120
