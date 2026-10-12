
`:00424395` `7509` `jne` `004243A0`

`:00424397` `8B510C` `mov` `edx,` `dword` `ptr` `[ecx+0C]` <--`edx`=`c1`

`:0042439A` `33C7` `xor` `eax,` `edi`

`:0042439C` `3BD0` `cmp` `edx,` `eax`

`:0042439E` `7413` `je` `004243B3` <--这个则一定要跳转

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

`:00424379(C)`, `:00424387(C)`, `:00424395(C)`

`:004243A0` `6A00` `push` `00000000`

`:004243A2` `6A00` `push` `00000000`

`:004243A4` `6A00` `push` `00000000`

`:004243A6` `6838FFFFFF` `push` `FFFFFF38`

`:004243AB` `E8F0360000` `call` `00427AA0` <--弹出注册失败窗口

`:004243B0` `83C410` `add` `esp,` `00000010`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:0042439E(C)`

`:004243B3` `8B6C2418` `mov` `ebp,` `dword` `ptr` `[esp+18]`

`:004243B7` `8B7C2414` `mov` `edi,` `dword` `ptr` `[esp+14]`

`:004243BB` `56` `push` `esi`

`:004243BC` `55` `push` `ebp`

`:004243BD` `57` `push` `edi`

`:004243BE` `E8ED040000` `call` `004248B0` <--按 F8 进入

`:004243C3` `83C40C` `add` `esp,` `0000000C`

`:004243C6` `85C0` `test` `eax,` `eax`

`:004243C8` `7510` `jne` `004243DA`

`:004243CA` `50` `push` `eax`

`:004243CB` `50` `push` `eax`

`:004243CC` `50` `push` `eax`

`:004243CD` `6837FFFFFF` `push` `FFFFFF37`

`:004243D2` `E8C9360000` `call` `00427AA0` <--弹出注册失败窗口

`:004243D7` `83C410` `add` `esp,` `00000010`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:004243C8(C)`

`:004243DA` `8B1E` `mov` `ebx,` `dword` `ptr` `[esi]`

`:004243DC` `C1EB10` `shr` `ebx,` `10`

`:004243DF` `66335E04` `xor` `bx,` `word` `ptr` `[esi+04]`

`:004243E3` `81E3FFFF0000` `and` `ebx,` `0000FFFF`

`:004243E9` `8D43F1` `lea` `eax,` `dword` `ptr` `[ebx-0F]`

`:004243EC` `3DBC040000` `cmp` `eax,` `000004BC`

`:004243F1` `7E13` `jle` `00424406` <--这个一定要跳转

`:004243F3` `6A00` `push` `00000000`

`:004243F5` `6A00` `push` `00000000`

`:004243F7` `6A00` `push` `00000000`

`:004243F9` `6834FFFFFF` `push` `FFFFFF34`

`:004243FE` `E89D360000` `call` `00427AA0` <--弹出注册失败窗口

`:00424403` `83C410` `add` `esp,` `00000010`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:004243F1(C)`

`:00424406` `83C30F` `add` `ebx,` `0000000F`

`:00424409` `81FBBC040000` `cmp` `ebx,` `000004BC`

`:0042440F` `7D13` `jge` `00424424` <--这个也一定要跳转

`:00424411` `6A00` `push` `00000000`

`:00424413` `6A00` `push` `00000000`

`:00424415` `6A00` `push` `00000000`

`:00424417` `6835FFFFFF` `push` `FFFFFF35`

`:0042441C` `E87F360000` `call` `00427AA0` <--弹出注册失败窗口

`:00424421` `83C410` `add` `esp,` `00000010`

在那个 `call` `00427AA0` 按 F8 进入后来到以下：

* Referenced by a CALL at Addresses:

`:004242D8`, `:004243BE`

`:004248B0` `8B44240C` `mov` `eax,` `dword` `ptr` `[esp+0C]`

`:004248B4` `8B542404` `mov` `edx,` `dword` `ptr` `[esp+04]`

`:004248B8` `53` `push` `ebx`

`:004248B9` `55` `push` `ebp`

`:004248BA` `8B18` `mov` `ebx,` `dword` `ptr` `[eax]` <--`ebx`=`54321`

`:004248BC` `57` `push` `edi`

`:004248BD` `8BFA` `mov` `edi,` `edx`

`:004248BF` `83C9FF` `or` `ecx,` `FFFFFFF F`

`:004248C2` `33C0` `xor` `eax,` `eax`

`:004248C4` `F2` `repnz`

`:004248C5` `AE` `scasb`

`:004248C6` `F7D1` `not` `ecx`

`:004248C8` `49` `dec` `ecx`

`:004248C9` `83F906` `cmp` `ecx,` `00000006` <--检查 name 的长度是否大于等于 6，

`:004248CC` `726B` `jb` `00424939` name 的长度应该>=6

`:004248CE` `8B6C2414` `mov` `ebp,` `dword` `ptr` `[esp+14]`

`:004248D2` `83C9FF` `or` `ecx,` `FFFFFFF F`

`:004248D5` `8BFD` `mov` `edi,` `ebp`

`:004248D7` `F2` `repnz`

`:004248D8` `AE` `scasb`

`:004248D9` `F7D1` `not` `ecx`

`:004248DB` `49` `dec` `ecx`

`:004248DC` `83F906` `cmp` `ecx,` `00000006` <--检查 company 的长度是否大于等于 6,

`:004248DF` `7258` `jb` `00424939` company 的长度应该>=6

`:004248E1` `F7C3FFFF0000` `test` `ebx,` `0000FFFF`

`:004248E7` `7450` `je` `00424939`

`:004248E9` `F7C30000FFFF` `test` `ebx,` `FFFF0000`

`:004248EF` `7448` `je` `00424939`

`:004248F1` `56` `push` `esi`

`:004248F2` `6A02` `push` `00000002`

`:004248F4` `52` `push` `edx`

`:004248F5` `E846000000` `call` `00424940` <--按 F8 进入

`:004248FA` `6A02` `push` `00000002`

`:004248FC` `55` `push` `ebp`

`:004248FD` `8BF0` `mov` `esi,` `eax`

`:004248FF` `E83C000000` `call` `00424940` <--按 F8 进入

`:00424904` `8BC8` `mov` `ecx,` `eax`

`:00424906` `8BD0` `mov` `edx,` `eax`

`:00424908` `81E10000FF00` `and` `ecx,` `00FF0000`

`:0042490E` `83C410` `add` `esp,` `00000010`

`:00424911` `C1EA10` `shr` `edx,` `10`

`:00424914` `0BCA` `or` `ecx,` `edx`

`:00424916` `8BD0` `mov` `edx,` `eax`

`:00424918` `81E200FF0000` `and` `edx,` `0000FF00`

`:0042491E` `C1E010` `shl` `eax,` `10`

`:00424921` `0BD0` `or` `edx,` `eax`

`:00424923` `33C0` `xor` `eax,` `eax`

`:00424925` `C1E908` `shr` `ecx,` `08`

`:00424928` `C1E208` `shl` `edx,` `08`

`:0042492B` `0BCA` `or` `ecx,` `edx`

`:0042492D` `33CE` `xor` `ecx,` `esi`

`:0042492F` `5E` `pop` `esi`

`:00424930` `3BCB` `cmp` `ecx,` `ebx`

`:00424932` `5F` `pop` `edi`

`:00424933` `5D` `pop` `ebp`

`:00424934` `5B` `pop` `ebx`

`:00424935` `0F94C0` `sete` `al`

`:00424938` `C3` `ret`

第一个 `call` `00424940` 是把 name 进行计算，第二个 `call` `00424940` 是把 company 进行计算。

* Referenced by a CALL at Addresses:

`:004248F5`, `:004248FF`

`:00424940` `8B442408` `mov` `eax,` `dword` `ptr` `[esp+08]`

`:00424944` `56` `push` `esi`

`:00424945` `48` `dec` `eax`

`:00424946` `740E` `je` `00424956`

`:00424948` `48` `dec` `eax`

`:00424949` `7404` `je` `0042494F` <--在这里一般都会跳转，而且应该要跳转

`:0042494B` `33C0` `xor` `eax,` `eax`

`:0042494D` `5E` `pop` `esi`

`:0042494E` `C3` `ret`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:00424949(C)`

`:0042494F` `BEA0804A00` `mov` `esi,` `004A80A0`

`:00424954` `EB05` `jmp` `0042495B`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:00424946(C)`

`:00424956` `BE60804A00` `mov` `esi,` `004A8060`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:00424954(U)`

`:0042495B` `8B542408` `mov` `edx,` `dword` `ptr` `[esp+08]`

`:0042495F` `57` `push` `edi`

`:00424960` `8BFA` `mov` `edi,` `edx`

`:00424962` `83C9FF` `or` `ecx,` `FFFFFFF F`

`:00424965` `33C0` `xor` `eax,` `eax`

`:00424967` `6A00` `push` `00000000`

`:00424969` `F2` `repnz`
