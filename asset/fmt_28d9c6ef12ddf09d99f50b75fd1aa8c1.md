
====>或者≤2EE0(H)

`jbe 0042F68D`

:0042F684 `cmp dword ptr [ebp-04], 00004E20`

====>或者≥4E20(H)

`jb 0042F696`

====>跳则 OVER！

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0042F682(C)

|

:0042F68D `cmp dword ptr [ebp-04], 000055F0`

====>或者≤55F0(H)

`jbe 0042F6A2`

====>应跳！ 

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:0042F631(C), :0042F643(C), :0042F655(C), :0042F667(C), :0042F679(C)

|:0042F68B(C)

====>跳到这儿就 OVER 了！

:0042F696 `mov [ebp-08], 00000000`

:0042F69D `jmp 0042F875`

====>跳向 OVER！

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0042F694(C)

|

:0042F6A2 `mov [ebp-14], 00000000`

:0042F6A9 `jmp 0042F6B4`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0042F6E2(U)

|

:0042F6AB `mov eax, dword ptr [ebp-14]`

:0042F6AE `add eax, 00000001`

:0042F6B1 `mov dword ptr [ebp-14], eax`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0042F6A9(U)

|

:0042F6B4 `mov ecx, dword ptr [ebp-14]`

:0042F6B7 `cmp ecx, dword ptr [ebp-0C]`

====>循环 NAME 的位数次！

:0042F6BA `jge 0042F6E4`

:0042F6BC `mov esi, dword ptr [ebp-14]`

:0042F6BF `add esi, 00000001`

:0042F6C2 `mov edx, dword ptr [ebp-14]`

:0042F6C5 `push edx`

:0042F6C6 `mov ecx, dword ptr [ebp-24]`

:0042F6C9 `add ecx, 00000060`

:0042F6CC `call 004031B0`

====>依次取 NAME 字符的 HEX 值

:0042F6D1 `movsx eax, al`

1、 ====>`EAX=66`

2、 ====>`EAX=6C`

3、 ====>`EAX=79`

:0042F6D4 `imul esi, eax`

1、 ====>`ESI=1 * 66=66`

2、 ====>`ESI=2 * 6C=D8`

3、 ====>`ESI=3 * 79=16B`

:0042F6D7 `add esi, dword ptr [ebp-0C]`

1、 ====>`ESI=66 + 3=69`

2、 ====>`ESI=D8 + 3=DB`

3、 ====>`ESI=16B + 3=16E`

:0042F6DA `mov ecx, dword ptr [ebp-18]`

1、 ====>`ECX=00`

2、 ====>`ECX=69`

3、 ====>`ECX=144`

:0042F6DD `add ecx, esi`

1、 ====>`ECX=00 + 69=69`

2、 ====>`ECX=69 + DB=144`

3、 ====>`ECX=144 + 16E=2B2`

:0042F6DF `mov dword ptr [ebp-18], ecx`

1、 ====>`[ebp-18]=ECX=69`

2、 ====>`[ebp-18]=ECX=144`

2、 ====>`[ebp-18]=ECX=2B2`

:0042F6E2 `jmp 0042F6AB`

====>跳上去继续循环

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0042F6BA(C)

|

:0042F6E4 `mov edx, dword ptr [ebp-0C]`

====>`EDX=3` 即：NAME 位数

:0042F6E7 `imul edx, dword ptr [ebp-04]`

====>`EDX=3 * 4D2=E76`

:0042F6EB `mov eax, dword ptr [ebp-18]`

====>`EAX=2B2`

:0042F6EE `add eax, edx`

====>`EAX=2B2 + E76=1128`

:0042F6F0 `mov dword ptr [ebp-18], eax`

====>`[ebp-18]=EAX=1128` 壹

:0042F6F3 `push 00000010`

:0042F6F5 `lea ecx, dword ptr [ebp-20]`

:0042F6F8 `push ecx`

:0042F6F9 `mov edx, dword ptr [ebp-18]`

:0042F6FC `push edx`

:0042F6FD `call 0044A43B`

:0042F702 `add esp, 0000000C`

:0042F705 `mov [ebp-14], 00000000`

:0042F70C `jmp 0042F717`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0042F73B(U)

|

:0042F70E `mov eax, dword ptr [ebp-14]`

:0042F711 `add eax, 00000001`

:0042F714 `mov dword ptr [ebp-14], eax`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0042F70C(U)

|

:0042F717 `mov ecx, dword ptr [ebp-14]`

:0042F71A `movsx edx, byte ptr [ebp+ecx-20]`

:0042F71F `test edx, edx`

:0042F721 `je 0042F73D`

:0042F723 `mov eax, dword ptr [ebp-14]`

:0042F726 `movsx ecx, byte ptr [ebp+eax-20]`

:0042F72B `push ecx`

:0042F72C `call 00440CB0`

:0042F731 `add esp, 00000004`

:0042F734 `mov edx, dword ptr [ebp-14]`

:0042F737 `mov byte ptr [ebp+edx-20], al`

:0042F73B `jmp 0042F70E`

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:0042F721(C), :0042F796(U)

|

:0042F73D `lea eax, dword ptr [ebp-20]`

====>`EAX=[ebp-20]=1128`

:0042F740 `push eax`

:0042F741 `call 00440AF0`

====>求 1128 的位数

:0042F746 `add esp, 00000004`

:0042F749 `cmp eax, 00000004`

====>是否≥4 位？

:0042F74C `jnb 0042F798`

====>跳下去再运算注册码的前 2 位！

:0042F74E `lea ecx, dword ptr [ebp-20]`

:0042F751 `push ecx`

:0042F752 `call 00440AF0`

:0042F757 `add esp, 00000004`

:0042F75A `mov [ebp+eax-1F], 00`

:0042F75F `lea edx, dword ptr [ebp-20]`

:0042F762 `push edx`

:0042F763 `call 00440AF0`

:0042F768 `add esp, 00000004`

:0042F76B `add eax, 00000001`

:0042F76E `mov dword ptr [ebp-14], eax`

:0042F771 `jmp 0042F77C`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0042F790(U)

|

:0042F773 `mov eax, dword ptr [ebp-14]`

:0042F776 `sub eax, 00000001`

:0042F779 `mov dword ptr [ebp-14], eax`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0042F771(U)

|

:0042F77C `cmp dword ptr [ebp-14], 00000000`

:0042F780 `je 0042F792`

:0042F782 `mov ecx, dword ptr [ebp-14]`

:0042F785 `mov edx, dword ptr [ebp-14]`
