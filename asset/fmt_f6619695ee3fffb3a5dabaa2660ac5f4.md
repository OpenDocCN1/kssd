
`:004D15F0 E86FB4F7FF call 0044CA64`

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
`:004D130D(U), :004D135E(U), :004D1382(C), :004D13A1(C), :004D1402(C), :004D1417(C), :004D142C(C), :004D1441(C), :004D14D7(U)`

`:004D15F5 33C0 xor eax, eax`
`:004D15F7 5A pop edx`
`:004D15F8 59 pop ecx`
`:004D15F9 59 pop ecx`
`:004D15FA 648910 mov dword ptr fs:[eax], edx`
`:004D15FD 6833164D00 push 004D1633`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
`:004D1631(U)`

`:004D1602 8D8580FDFFFF lea eax, dword ptr [ebp+FFFFFD80]`
`:004D1608 BA09000000 mov edx, 00000009`
`:004D160D E8CA26F3FF call 00403CDC`
`:004D1612 8D85A4FEFFFF lea eax, dword ptr [ebp+FFFFFEA4]`

\* Possible StringData Ref from Code Obj -> `TSearchRecX`

`:004D1618 8B156C7E4000 mov edx, dword ptr [00407E6C]`
`:004D161E E8ED2FF3FF call 00404610`
`:004D1623 8D45FC lea eax, dword ptr [ebp-04]`
`:004D1626 E88D26F3FF call 00403CB8`
`:004D162B C3 ret`

--------------------------------------------------------------------------------------------------
在上面的核心 `call` 中按 `F8` 进入后，会来到如下地方（注意：由于代码在 `DLL` 中，所以地址与内存中的地址会有所不同）：

`:004043AC 55 push ebp`
`:004043AD 68FE434000 push 004043FE`
`:004043B2 64FF30 push dword ptr fs:[eax]`
`:004043B5 648920 mov dword ptr fs:[eax], esp`
`:004043B8 8B4510 mov eax, dword ptr [ebp+10]`
`:004043BB 8945F8 mov dword ptr [ebp-08], eax`
`:004043BE 8B4508 mov eax, dword ptr [ebp+08]`
`:004043C1 8945FC mov dword ptr [ebp-04], eax`
`:004043C4 8D45F0 lea eax, dword ptr [ebp-10]`
`:004043C7 8BD3 mov edx, ebx`
`:004043C9 E876EBFFFF call 00402F44 <--处理申请注册码`
`:004043CE 8B45F0 mov eax, dword ptr [ebp-10]`
`:004043D1 8D55F6 lea edx, dword ptr [ebp-0A]`
`:004043D4 8D4DF8 lea ecx, dword ptr [ebp-08]`
`:004043D7 E8FCFDFFFF call 004041D8 <--进行注册码计算的核心 CALL，按 F8 进入`
`:004043DC 85C0 test eax, eax`
`:004043DE 7504 jne 004043E4 <--一定要跳转`
`:004043E0 33DB xor ebx, ebx`
`:004043E2 EB04 jmp 004043E8`

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
`:0040437D(C), :004043DE(C)`

`:004043E4 0FB75DF6 movzx ebx, word ptr [ebp-0A] <--把第二部分的计算结果的低 16 位送到 EBX 中`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
`:004043E2(U)`

`:004043E8 33C0 xor eax, eax`
`:004043EA 5A pop edx`
`:004043EB 59 pop ecx`
`:004043EC 59 pop ecx`
`:004043ED 648910 mov dword ptr fs:[eax], edx`
`:004043F0 6805444000 push 00404405`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
`:00404403(U)`

`:004043F5 8D45F0 lea eax, dword ptr [ebp-10]`
`:004043F8 E85FEAFFFF call 00402E5C`
`:004043FD C3 ret`

`:004043FE E9D5E4FFFF jmp 004028D8`
`:00404403 EBF0 jmp 004043F5`
`:00404405 8BC3 mov eax, ebx <--把第二部分的计算结果的低 16 位送到 EAX 中`
`:00404407 5B pop ebx`
`:00404408 8BE5 mov esp, ebp`
`:0040440A 5D pop ebp`
`:0040440B C21000 ret 0010`

--------------------------------------------------------------------------------------------------
在上面的核心 `call` 按 `F8` 进入后，会来到如下地方：

`:004041D8 55 push ebp`
`:004041D9 8BEC mov ebp, esp`
`:004041DB 83C4DC add esp, FFFFFFDC`
`:004041DE 53 push ebx`
`:004041DF 56 push esi`
`:004041E0 57 push edi`
`:004041E1 33DB xor ebx, ebx`
`:004041E3 895DEC mov dword ptr [ebp-14], ebx`
`:004041E6 8BF1 mov esi, ecx`
`:004041E8 8D7DF0 lea edi, dword ptr [ebp-10]`
`:004041EB A5 movsd <--把输入的注册码的第一部分的 32 位十六进制数送到 EDI 所指向的内存中`
`:004041EC A5 movsd <--把输入的注册码的第二部分的 32 位十六进制数送到 EDI 所指向的内存中`
`:004041ED 8955F8 mov dword ptr [ebp-08], edx`
`:004041F0 8945FC mov dword ptr [ebp-04], eax`
`:004041F3 8B45FC mov eax, dword ptr [ebp-04]`
`:004041F6 E8C5EDFFFF call 00402FC0`
`:004041FB 33C0 xor eax, eax`
`:004041FD 55 push ebp`
`:004041FE 685F434000 push 0040435F`
`:00404203 64FF30 push dword ptr fs:[eax]`
`:00404206 648920 mov dword ptr fs:[eax], esp`
`:00404209 8D45EC lea eax, dword ptr [ebp-14]`

\* Possible StringData Ref from Code Obj -> `You are big pig.`

`:0040420C BA78434000 mov edx, 00404378`
`:00404211 E8AEECFFFF call 00402EC4`
`:00404216 8B45FC mov eax, dword ptr [ebp-04]`
`:00404219 E856EDFFFF call 00402F74`
`:0040421E 2507000080 and eax, 80000007`
`:00404223 7905 jns 0040422A`
`:00404225 48 dec eax`
`:00404226 83C8F8 or eax, FFFFFFF8`
`:00404229 40 inc eax`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
`:00404223(C)`

`:0040422A 85C0 test eax, eax`
`:0040422C 742C je 0040425A`
`:0040422E 8D45FC lea eax, dword ptr [ebp-04]`

\* Possible StringData Ref from Code Obj -> `1234567`

`:00404231 BA94434000 mov edx, 00404394`
`:00404236 E841EDFFFF call 00402F7C <--把"1234567"连接到注册申请码的后面形成新的字符串`
`:0040423B 8B45FC mov eax, dword ptr [ebp-04]`
`:0040423E E831EDFFFF call 00402F74 <--求新字符串的长度`
`:00404243 85C0 test eax, eax`
`:00404245 7903 jns 0040424A <--一般都会跳转`
`:00404247 83C007 add eax, 00000007`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
`:00404245(C)`

`:0040424A C1F803 sar eax, 03`
`:0040424D 8BD0 mov edx, eax`
`:0040424F C1E203 shl edx, 03`
`:00404252 8D45FC lea eax, dword ptr [ebp-04]`
`:00404255 E8AEEDFFFF call 00403008 <--截断新字符串，使其长度位 8`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
`:0040422C(C)`

`:0040425A 33F6 xor esi, esi`
`:0040425C 8D45FC lea eax, dword ptr [ebp-04]`
`:0040425F E86CEDFFFF call 00402FD0`
`:00404264 8BF8 mov edi, eax`
`:00404266 8D45EC lea eax, dword ptr [ebp-14]`
`:00404269 E862EDFFFF call 00402FD0 <--拷贝字符串"You are big pig."到 EAX 所指的内存中`
`:0040426E 8BD8 mov ebx, eax`
