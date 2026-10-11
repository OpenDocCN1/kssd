
:10007629 8B542410 `mov edx, dword ptr [esp+10]`   

:1000762D 33C0 `xor eax, eax`   

:1000762F 53 `push ebx`   

:10007630 89442404 `mov dword ptr [esp+04], eax`   

:10007634 56 `push esi`   

:10007635 8944240C `mov dword ptr [esp+0C], eax`   

:10007639 51 `push ecx`   

:1000763A 6689442414 `mov word ptr [esp+14], ax`   

:1000763F 68CC410310 `push 100341CC`   

:10007644 8D442410 `lea eax, dword ptr [esp+10]`   

:10007648 52 `push edx`   

:10007649 50 `push eax`   

:1000764A E8119D0000 `call 10011360` <= 详细在下面   

* Possible StringData Ref from Data Obj -> `"$LS98"` <= 看是否为此   

:1000764F 8B35F8410310 `mov esi, dword ptr [100341F8]`   

:10007655 83C410 `add esp, 00000010`   

:10007658 8D442408 `lea eax, dword ptr [esp+08]` <= 由 IRC 求得的码   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:1000767E(C)

|   

:1000765C 8A10 `mov dl, byte ptr [eax]`   

:1000765E 8A1E `mov bl, byte ptr [esi]`   

:10007660 8ACA `mov cl, dl`   

:10007662 3AD3 `cmp dl, bl`   

:10007664 751E `jne 10007684`   

:10007666 84C9 `test cl, cl`   

:10007668 7416 `je 10007680`   

:1000766A 8A5001 `mov dl, byte ptr [eax+01]`   

:1000766D 8A5E01 `mov bl, byte ptr [esi+01]`   

:10007670 8ACA `mov cl, dl`   

:10007672 3AD3 `cmp dl, bl`   

:10007674 750E `jne 10007684`   

:10007676 83C002 `add eax, 00000002`   

:10007679 83C602 `add esi, 00000002`   

:1000767C 84C9 `test cl, cl`   

:1000767E 75DC `jne 1000765C`   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:10007668(C)

|   

:10007680 33C0 `xor eax, eax`   

:10007682 EB05 `jmp 10007689`   

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:10007664(C), :10007674(C)

|   

:10007684 1BC0 `sbb eax, eax`   

:10007686 83D8FF `sbb eax, FFFFFFFF`   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:10007682(U)

|   

:10007689 85C0 `test eax, eax` <- 此处改为 `CMP EAX,EAX` 就可暴破   

:1000768B 7443 `je 100076D0`   

* Possible StringData Ref from Data Obj -> `"$BCWP"`   

:1000768D 8B35FC410310 `mov esi, dword ptr [100341FC]` <- 另一次机会   

:10007693 8D442408 `lea eax, dword ptr [esp+08]`   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:100076B9(C)

|   

:10007697 8A10 `mov dl, byte ptr [eax]`   

:10007699 8A1E `mov bl, byte ptr [esi]`   

:1000769B 8ACA `mov cl, dl`   

:1000769D 3AD3 `cmp dl, bl`   

:1000769F 751E `jne 100076BF` <-   

:100076A1 84C9 `test cl, cl`   

:100076A3 7416 `je 100076BB`   

:100076A5 8A5001 `mov dl, byte ptr [eax+01]`   

:100076A8 8A5E01 `mov bl, byte ptr [esi+01]`   

:100076AB 8ACA `mov cl, dl`   

:100076AD 3AD3 `cmp dl, bl`   

:100076AF 750E `jne 100076BF`   

:100076B1 83C002 `add eax, 00000002`   

:100076B4 83C602 `add esi, 00000002`   

:100076B7 84C9 `test cl, cl`   

:100076B9 75DC `jne 10007697`   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:100076A3(C)

|   

:100076BB 33C0 `xor eax, eax`   

:100076BD EB05 `jmp 100076C4`   

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:1000769F(C), :100076AF(C)

|   

:100076BF 1BC0 `sbb eax, eax`   

:100076C1 83D8FF `sbb eax, FFFFFFFF`   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:100076BD(U)

|   

:100076C4 85C0 `test eax, eax`   

:100076C6 7408 `je 100076D0`   

:100076C8 5E `pop esi`   

:100076C9 33C0 `xor eax, eax`   

:100076CB 5B `pop ebx`   

:100076CC 83C40C `add esp, 0000000C`   

:100076CF C3 `ret`   

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:1000768B(C), :100076C6(C)

|   

:100076D0 8B44241C `mov eax, dword ptr [esp+1C]`   

:100076D4 8B4C240E `mov ecx, dword ptr [esp+0E]`   

:100076D8 5E `pop esi`   

:100076D9 5B `pop ebx`   

:100076DA 8908 `mov dword ptr [eax], ecx`   

* Possible Reference to String Resource ID=00001: `"%d Hours"`   

:100076DC B801000000 `mov eax, 00000001`   

:100076E1 83C40C `add esp, 0000000C`   

:100076E4 C3 `ret`   

\====================================================================   

在 `:1000764A` 处的调用   

* Referenced by a CALL at Addresses:

| `:1000764A` , `:1000788A` 

|   

:10011360 6AFF `push FFFFFFFF`   

:10011362 6880A40210 `push 1002A480`   

:10011367 64A100000000 `mov eax, dword ptr fs:[00000000]`   

:1001136D 50 `push eax`   

:1001136E 64892500000000 `mov dword ptr fs:[00000000], esp`   

:10011375 83EC14 `sub esp, 00000014`   

:10011378 8B442430 `mov eax, dword ptr [esp+30]`   

:1001137C 8B4C2428 `mov ecx, dword ptr [esp+28]`   

:10011380 56 `push esi`   

:10011381 8B742428 `mov esi, dword ptr [esp+28]`   

:10011385 50 `push eax`   

:10011386 51 `push ecx`   

:10011387 56 `push esi`   

:10011388 C744241000000000 `mov [esp+10], 00000000`   

:10011390 E89B180000 `call 10012C30` <- 见下面   

:10011395 83C40C `add esp, 0000000C`   

:10011398 85C0 `test eax, eax` <- 不能让 `Eax` 返回值为 0   

:1001139A 7412 `je 100113AE` <- 跳下去就有大麻烦了，因为它会按别的方式计算 `Rc`，我实在是不想跟下去   

:1001139C 33C0 `xor eax, eax` <- 大家不信可以跟一跟   

:1001139E 5E `pop esi`   

:1001139F 8B4C2414 `mov ecx, dword ptr [esp+14]`   

:100113A3 64890D00000000 `mov dword ptr fs:[00000000], ecx`   

:100113AA 83C420 `add esp, 00000020`   

:100113AD C3 `ret`   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| `:1001139A(C)` 

|   

:100113AE 33D2 `xor edx, edx`   

:100113B0 89542410 `mov dword ptr [esp+10], edx`   

:100113B4 89542414 `mov dword ptr [esp+14], edx`   

:100113B8 33C0 `xor eax, eax`   

:100113BA 89542420 `mov dword ptr [esp+20], edx`   

:100113BE 89442408 `mov dword ptr [esp+08], eax`   

:100113C2 8944240C `mov dword ptr [esp+0C], eax`   

:100113C6 8B542430 `mov edx, dword ptr [esp+30]`   

:100113CA 8D4C2434 `lea ecx, dword ptr [esp+34]`   

:100113CE 51 `push ecx`   

:100113CF 8D44240C `lea eax, dword ptr [esp+0C]`   

:100113D3 52 `push edx`   
