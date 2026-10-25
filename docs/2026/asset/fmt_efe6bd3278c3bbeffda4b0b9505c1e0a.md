
`mov dword ptr [esp+1C], edi`====>累加结果

`:10001613 72E9`

`jb 100015FE`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:100015FC(C)

|

`:10001615 8BFA`

`mov edi, edx`

`:10001617 83E703`

`and edi, 00000003` 是否为4的倍数

`:1000161A 7420`

`je 1000163C`=======>是则转

`:1000161C 33F6`

`xor esi, esi`

`:1000161E 33C9`

`xor ecx, ecx`

`:10001620 33C0`

`xor eax, eax`

`:10001622 85FF`

`test edi, edi`

`:10001624 7622`

`jbe 10001648`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:10001638(C)

|

`:10001626 0FBEAC04A8000000`

`movsx ebp, byte ptr [esp+eax+000000A8]`====>Name串

`:1000162E`

`D3E5` `shl ebp, cl`

`:10001630 83C108`

`add ecx, 00000008`

`:10001633 0BF5`

`or esi, ebp`

`:10001635 40`

`inc eax`

`:10001636 3BC7`

`cmp eax, edi`

`:10001638 72EC`

`jb 10001626`

`:1000163A EB0C`

`jmp 10001648`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:1000161A(C)

|

`:1000163C 8BB424A8000000` `mov esi, dword ptr [esp+000000A8]`

`:10001643 B804000000`

`mov eax, 00000004`

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:10001624(C), :1000163A(U)

|

`:10001648 3BC2`

`cmp eax, edx`

`:1000164A 7310`

`jnb 1000165C`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:1000165A(C)

|

`:1000164C 8BAC04A8000000`

`mov ebp, dword ptr [esp+eax+000000A8]`

`:10001653`

`83C004` `add eax, 00000004`

`:10001656 33F5`

`xor esi, ebp`=========>ESI的值261B0109(我的)

`:10001658 3BC2`

`cmp eax, edx`

`:1000165A 72F0`

`jb 1000164C`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:1000164A(C)

|

`:1000165C 8B542418`

`mov edx, dword ptr [esp+18]`

`:10001660`

`83E203` `and edx, 00000003`

`:10001663 741D`

`je 10001682`

`:10001665 33ED`

`xor ebp, ebp`

`:10001667 33C9`

`xor ecx, ecx`

`:10001669`

`33C0` `xor eax, eax`

`:1000166B 85D2`

`test edx, edx`

`:1000166D 761C`

`jbe 1000168B`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:1000167E(C)

|

`:1000166F 0FBE7C0428`

`movsx edi, byte ptr [esp+eax+28]`===>Code串1+用户名余串

`:10001674 D3E7`

`shl edi, cl`

`:10001676 83C108`

`add ecx, 00000008`

`:10001679 0BEF`

`or ebp, edi`

`:1000167B 40`

`inc eax`

`:1000167C 3BC2`

`cmp eax, edx`

`:1000167E 72EF`

`jb 1000166F`

`:10001680 EB09`

`jmp 1000168B`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:10001663(C)

|

`:10001682 8B6C2428`

`mov ebp, dword ptr [esp+28]`=======>串1

`:10001686 B804000000`

`mov eax, 00000004`

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:1000166D(C), :10001680(U)

|

`:1000168B 8B4C2418` `mov ecx, dword ptr [esp+18]`

`:1000168F 3BC1`

`cmp eax, ecx`

`:10001691 730D`

`jnb 100016A0`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:1000169E(C)

|

`:10001693 8B540428`

`mov edx, dword ptr [esp+eax+28]`

`:10001697 83C004`

`add eax, 00000004`

`:1000169A 33EA`

`xor ebp, edx`============>结果63382C54(我的)

`:1000169C 3BC1`

`cmp eax, ecx`

`:1000169E 72F3`

`jb 10001693`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:10001691(C)

|

`:100016A0 8D542468`

`lea edx, dword ptr [esp+68]`

`:100016A4 6A0A`

`push 0000000A`

`:100016A6 52`

`push edx`

`:100016A7 8BC6`

`mov eax, esi`

`:100016A9 33D2`

`xor edx, edx`

`:100016AB B9FFFFFF7F` `mov ecx, 7FFFFFFF`

`:100016B0 F7F1`

`div ecx`

`:100016B2 33EB`

`xor ebp, ebx`==========>ebp

`:100016B4 52`

`push edx`===>

`:100016B5 E805100100` `call 100126BF`=======>

`EAX=960575048`

`ECX=75048`

`:100016BA 8B5C2428`

`mov ebx, dword ptr [esp+28]`

`:100016BE 8B3D388F0110` `mov edi, dword ptr [10018F38]`

`:100016C4 8D9424B4010000`

`lea edx, dword ptr [esp+000001B4]`

`:100016CB 6A0A`

`push 0000000A`

`:100016CD 52`

`push edx`

`:100016CE 53`

`push ebx`=================>累加和

`:100016CF 33F7`

`xor esi, edi`==========>esi

`:100016D1 E8E90F0100` `call 100126BF`

`EAX=12159`

`ECX=159`

取后3个

`:100016D6 8DBC2480000000`

`lea edi, dword ptr [esp+00000080]`

`:100016DD 83C9FF`

`or ecx, FFFFFFFF`

`:100016E0 33C0`

`xor eax, eax`

`:100016E2 83C418`

`add esp, 00000018`

`:100016E5 F2`

`repnz`

`:100016E6 AE`

`scasb`

`:100016E7`

`F7D1` `not ecx`

`:100016E9 49`

`dec ecx`

`:100016EA 8DBC24A8010000`

`lea edi, dword ptr [esp+000001A8]`

`:100016F1 894C2420`

`mov dword ptr [esp+20], ecx`

`:100016F5`

`83C9FF` `or ecx, FFFFFFFF`

`:100016F8 F2`

`repnz`

`:100016F9 AE`

`scasb`

`:100016FA F7D1`

`not ecx`

`:100016FC 49`

`dec ecx`

`:100016FD 8BC3`

`mov eax, ebx`==========>累加和

`:100016FF 894C2418`

`mov dword ptr [esp+18], ecx`

`:10001703`

`33D2` `xor edx, edx`

`:10001705 B905000000`

`mov ecx, 00000005`

`:1000170A F7F1`

`div ecx`

`:1000170C 85D2`

`test edx, edx`

`:1000170E 8954241C`

`mov dword ptr [esp+1C], edx`

`:10001712 7504`

`jne 10001718`========是5的倍数则=0否则=5

`:10001714 894C241C`

`mov dword ptr [esp+1C], ecx`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:10001712(C)

|

`:10001718 33DB`

`xor ebx, ebx`

`:1000171A 33FF`

`xor edi, edi`

`:1000171C 3BF5`

`cmp esi, ebp`

`:1000171E C644241400`

`mov [esp+14], 00`

`:10001723 0F85F0000000`

`jne 10001819`============>错误8

====================================================================

00DE171A

====================================================================

[这段循环检测条件]

共14次检测

这儿使用的是ODBG中的代码，因为我分析使用的

====================================================================

====================================================================

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:100017FA(C),

:10001802(U)

|

00DE1729 8A443C 68

`MOV AL,BYTE PTR SS:[ESP+EDI+68]`

00DE172D 8A4C24 1C

`MOV CL,BYTE PTR SS:[ESP+1C]`=5NameAdd的和

00DE1731 02C1

`ADD AL,CL`

00DE1733 0FBED0

`MOVSX EDX,AL`

00DE1736 52

`PUSH EDX`

00DE1737 884424 16

`MOV BYTE PTR SS:[ESP+16],AL`==========

00DE173B

`E8 06990000` `CALL Srch32_d.00DEB046`

+

00DE1740 83C4 04

`ADD ESP,4`

+===>同一位置

00DE1743 85C0

`TEST EAX,EAX`

+

00DE1745 75 05

`JNZ SHORT Srch32_d.00DE174C`==========

00DE1747

`804424 12 F7` `ADD BYTE PTR SS:[ESP+12],F7`

00DE174C

`8A4424 12` `MOV AL,BYTE PTR SS:[ESP+12]`

00DE1750

`8A4C1C 28` `MOV CL,BYTE PTR SS:[ESP+EBX+28]`===>奇数位值的串

00DE1754 3AC1 `CMP AL,CL`

00DE1756 0F85 BD000000 `JNZ Srch32_d.00DE1819`============>错误9

00DE175C 43

`INC EBX`

00DE175D 83FB 0E `CMP EBX,E`

00DE1760 0F83 A1000000 `JNB Srch32_d.00DE1807`=====================>正确1

00DE1766 0FBE4C1C 28 `MOVSX ECX,BYTE PTR SS:[ESP+EBX+28]`

00DE176B 51

`PUSH ECX`

00DE176C E8 D5980000 `CALL Srch32_d.00DEB046`

00DE1771 83C4 04 `ADD ESP,4`

00DE1774

`85C0` `TEST EAX,EAX`

00DE1776

`75 0A` `JNZ SHORT Srch32_d.00DE1782`

00DE1778 43 `INC EBX`

00DE1779 83FB 0E `CMP EBX,E`

00DE177C

`0F83 85000000` `JNB Srch32_d.00DE1807`=====================>正确2

00DE1782 8B7424 20 `MOV ESI,DWORD PTR SS:[ESP+20]`=====>NameXor串长度

00DE1786 47

`INC EDI`

00DE1787 3BFE

`CMP EDI,ESI`

00DE1789 72 02

`JB SHORT Srch32_d.00DE178D`

00DE178B 33FF
