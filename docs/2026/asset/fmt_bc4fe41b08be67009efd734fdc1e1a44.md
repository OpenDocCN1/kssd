
```asm
movsx eax, byte ptr [esi]
:00402C0F 8D0480 lea eax, dword ptr [eax+4*eax]
:00402C12 8D0480 lea eax, dword ptr [eax+4*eax]
:00402C15 8D0480 lea eax, dword ptr [eax+4*eax]
:00402C18 8D0C80 lea ecx, dword ptr [eax+4*eax]
:00402C1B 0FBE4604 movsx eax, byte ptr [esi+04]
:00402C1F C1E104 shl ecx, 04
:00402C22 8D14C500000000 lea edx, dword ptr [8*eax+00000000]
:00402C29 2BD0 sub edx, eax
:00402C2B C1E208 shl edx, 08
:00402C2E 2BD0 sub edx, eax
:00402C30 8D0490 lea eax, dword ptr [eax+4*edx]
:00402C33 8D1441 lea edx, dword ptr [ecx+2*eax]
:00402C36 0FBE4E01 movsx ecx, byte ptr [esi+01]
:00402C3A 8BC1 mov eax, ecx
:00402C3C C1E006 shl eax, 06
:00402C3F 2BC1 sub eax, ecx
:00402C41 C1E004 shl eax, 04
:00402C44 03C1 add eax, ecx
:00402C46 8D0440 lea eax, dword ptr [eax+2*eax]
:00402C49 8D0C41 lea ecx, dword ptr [ecx+2*eax]
:00402C4C 8D144A lea edx, dword ptr [edx+2*ecx]
:00402C4F 0FBE4E02 movsx ecx, byte ptr [esi+02]
:00402C53 8BC1 mov eax, ecx
:00402C55 C1E005 shl eax, 05
:00402C58 03C1 add eax, ecx
:00402C5A 8D0480 lea eax, dword ptr [eax+4*eax]
:00402C5D 8D0481 lea eax, dword ptr [ecx+4*eax]
:00402C60 8D0C80 lea ecx, dword ptr [eax+4*eax]
:00402C63 8D848AF2430B00 lea eax, dword ptr [edx+4*ecx+000B43F2]
:00402C6A 8D542410 lea edx, dword ptr [esp+10]
:00402C6E 50 push eax
* Possible Reference to Dialog:
            |
:00402C6F 6804364800 push 00483604
:00402C74 52 push edx
:00402C75 E86F380300 call 004364E9
:00402C7A 83C40C add esp, 0000000C
      << dd ecx '6169530'
:00402C7D 8D442410 lea eax, dword ptr [esp+10]
:00402C81 8D4C2414 lea ecx, dword ptr [esp+14]
:00402C85 50 push eax
:00402C86 E8AAAA0300 call 0043D735
:00402C8B 8D4C2430 lea ecx, dword ptr [esp+30]
  << dd eax = good regcode
:00402C8F E868380300 call 004364FC
      << YATNGO-99875-6169530
:00402C94 8B742430 mov esi, dword ptr [esp+30]
:00402C98 8B442414 mov eax, dword ptr [esp+14]
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00402CBE(C)
|
:00402C9C 8A10 mov dl, byte ptr [eax]
:00402C9E 8A1E mov bl, byte ptr [esi]
:00402CA0 8ACA mov cl, dl
:00402CA2 3AD3 cmp dl, bl
:00402CA4 751E jne 00402CC4
:00402CA6 84C9 test cl, cl
:00402CA8 7416 je 00402CC0
:00402CAA 8A5001 mov dl, byte ptr [eax+01]
:00402CAD 8A5E01 mov bl, byte ptr [esi+01]
:00402CB0 8ACA mov cl, dl
:00402CB2 3AD3 cmp dl, bl
:00402CB4 750E jne 00402CC4
:00402CB6 83C002 add eax, 00000002
:00402CB9 83C602 add esi, 00000002
:00402CBC 84C9 test cl, cl
:00402CBE 75DC jne 00402C9C
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00402CA8(C)
|
:00402CC0 33C0 xor eax, eax
:00402CC2 EB04 jmp 00402CC8
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00402CA4(C), :00402CB4(C)
|
:00402CC4 1BC0 sbb eax, eax
:00402CC6 1BC5 sbb eax, ebp
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00402CC2(U)
|
:00402CC8 85C0 test eax, eax
:00402CCA 7530 jne 00402CFC
:00402CCC 8D4C2414 lea ecx, dword ptr [esp+14]
:00402CD0 C644242401 mov [esp+24], 01
:00402CD5 E843A60300 call 0043D31D
:00402CDA 8D4C2410 lea ecx, dword ptr [esp+10]
:00402CDE C644242400 mov [esp+24], 00
:00402CE3 E835A60300 call 0043D31D
:00402CE8 8D4C2430 lea ecx, dword ptr [esp+30]
:00402CEC 896C2424 mov dword ptr [esp+24], ebp
:00402CF0 E828A60300 call 0043D31D
:00402CF5 B801000000 mov eax, 00000001
:00402CFA EB2B jmp 00402D27
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00402B32(C), :00402B49(C), :00402CCA(C)
|
:00402CFC 8D4C2414 lea ecx, dword ptr [esp+14]
:00402D00 C644242401 mov [esp+24], 01
:00402D05 E813A60300 call 0043D31D
:00402D0A 8D4C2410 lea ecx, dword ptr [esp+10]
:00402D0E C644242400 mov [esp+24], 00
:00402D13 E805A60300 call 0043D31D
:00402D18 8D4C2430 lea ecx, dword ptr [esp+30]
:00402D1C 896C2424 mov dword ptr [esp+24], ebp
:00402D20 E8F8A50300 call 0043D31D
:00402D25 33C0 xor eax, eax
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00402AF8(U), :00402CFA(U)
|
:00402D27 8B4C241C mov ecx, dword ptr [esp+1C]
:00402D2B 5F pop edi
:00402D2C 5E pop esi
:00402D2D 5D pop ebp
:00402D2E 5B pop ebx
:00402D2F 64890D00000000 mov dword ptr fs:[00000000], ecx
:00402D36 83C418 add esp, 00000018
:00402D39 C20800 ret 0008
*****************************************************************************************
```

整个过程不是很复杂,这里面还有一对注册码,
`name`: SpongeUk
`key`: YTNAGO-29775-5868020
用这对注册码注册,也会提示OK(参看上面的过程),但是,下次启动后还是"unregister version"
我当时没有仔细看在刚刚安装后,注册表`local_machine->software->long->reg1`中的信息,还没有研究过这对注册码的作用.
另外,就是在填写注册信息中的`name`时,不能填写类似这样的"abba",程序上会比较`name`的1和2字母,以及0和3字母,如果相同好像也不行,这儿,我没看仔细,挺奇怪的,^_^,还好我的`name`: fengy不是这种组合
本想,多花点时间,搞清楚是如何计算注册码的.可惜暂时没弄明白.

---fengy----
