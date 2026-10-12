
# 神算刘半仙 2003 Build 09.01

> 原文：[`www.pediy.com/kssd/pediy06/pediy6248.htm`](https://www.pediy.com/kssd/pediy06/pediy6248.htm)

神算刘半仙 2003 Build 09.01 算法简析

机器码：`92495650(d)`---`5835F22(h)`

注册码：`36b3a175da3637`

试验码：`87654321`

主文件 `ssbx.exe`，无壳，`delphi` 编程。

```
:004C850E 8D45FC              lea eax, dword ptr [ebp-04]
:004C8511 8D957BFFFFFF        lea edx, dword ptr [ebp+FFFFFF7B]
:004C8517 B981000000          mov ecx, 00000081
:004C851C E89BC4F3FF          call 004049BC
:004C8521 8D9574FFFFFF        lea edx, dword ptr [ebp+FFFFFF74]
:004C8527 8B8328030000        mov eax, dword ptr [ebx+00000328]
:004C852D E84272F7FF          call 0043F774
:004C8532 8B8574FFFFFF        mov eax, dword ptr [ebp+FFFFFF74]      //eax=假码
:004C8538 50                  push eax
:004C8539 8D956CFFFFFF        lea edx, dword ptr [ebp+FFFFFF6C]
:004C853F 8B8324030000        mov eax, dword ptr [ebx+00000324]
:004C8545 E82A72F7FF          call 0043F774
:004C854A 8B856CFFFFFF        mov eax, dword ptr [ebp+FFFFFF6C]      //eax=机器码
:004C8550 E85B0CF4FF          call 004091B0
:004C8555 8D9570FFFFFF        lea edx, dword ptr [ebp+FFFFFF70]
:004C855B E804FEFFFF          call 004C8364         //算法 call
:004C8560 8B9570FFFFFF        mov edx, dword ptr [ebp+FFFFFF70]
:004C8566 58                  pop eax
:004C8567 E8E4C5F3FF          call 00404B50         //关键比较，eax 假码，edx 真码
:004C856C 0F858F000000        jne 004C8601         //跳则 over

* Possible StringData Ref from Code Obj ->"注册成功，谢谢你的注册！"
                          |
:004C8572 B874864C00          mov eax, 004C8674
:004C8577 E89405F7FF          call 00438B10
:004C857C A1A4474D00          mov eax, dword ptr [004D47A4]
:004C8581 8B00                mov eax, dword ptr [eax]
:004C8583 8B80F0020000        mov eax, dword ptr [eax+000002F0]
:004C8589 8B8008020000        mov eax, dword ptr [eax+00000208]
:004C858F BA01000000          mov edx, 00000001
:004C8594 E8138AFDFF          call 004A0FAC

* Possible StringData Ref from Code Obj ->"软件已注册"
                          |
:004C8599 BA98864C00          mov edx, 004C8698
:004C859E E86589FDFF          call 004A0F08
:004C85A3 8D8568FFFFFF        lea eax, dword ptr [ebp+FFFFFF68]

* Possible StringData Ref from Code Obj ->"FGHkey1.sys"        //注册码存放的文件
                          |
:004C85A9 B9AC864C00          mov ecx, 004C86AC
:004C85AE 8B55FC              mov edx, dword ptr [ebp-04]
:004C85B1 E8A2C4F3FF          call 00404A58
:004C85B6 8B8D68FFFFFF        mov ecx, dword ptr [ebp+FFFFFF68]
:004C85BC B201                mov dl, 01

* Possible StringData Ref from Code Obj ->"?F"
                          |
:004C85BE A1D4274600          mov eax, dword ptr [004627D4]
:004C85C3 E8BCA2F9FF          call 00462884

* Possible StringData Ref from Code Obj ->"44134co642ls3058372"
                          |
:004C85C8 68C4864C00          push 004C86C4

* Possible StringData Ref from Code Obj ->"sepop"
                          |
:004C85CD B9E0864C00          mov ecx, 004C86E0

* Possible StringData Ref from Code Obj ->"syetup"
                          |
:004C85D2 BAF0864C00          mov edx, 004C86F0
:004C85D7 8B18                mov ebx, dword ptr [eax]
:004C85D9 FF5304              call [ebx+04]
:004C85DC 8D8564FFFFFF        lea eax, dword ptr [ebp+FFFFFF64]

* Possible StringData Ref from Code Obj ->"FGHkey1.sys"
                          |
:004C85E2 B9AC864C00          mov ecx, 004C86AC
:004C85E7 8B55FC              mov edx, dword ptr [ebp-04]
:004C85EA E869C4F3FF          call 00404A58
:004C85EF 8B8564FFFFFF        mov eax, dword ptr [ebp+FFFFFF64]
:004C85F5 BA03000000          mov edx, 00000003
:004C85FA E8F90DF4FF          call 004093F8
:004C85FF EB17                jmp 004C8618

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004C856C(C)
|
* Possible StringData Ref from Code Obj ->"注册码错误，请重新输入！"
                          |
:004C8601 B800874C00          mov eax, 004C8700

*********************************************************************

* Referenced by a CALL at Address:
|:004C855B
|
:004C8364 55                  push ebp
:004C8365 8BEC                mov ebp, esp
:004C8367 33C9                xor ecx, ecx
:004C8369 51                  push ecx
:004C836A 51                  push ecx
:004C836B 51                  push ecx
:004C836C 51                  push ecx
:004C836D 51                  push ecx
:004C836E 51                  push ecx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004C82FE(C)
|
:004C836F 53                  push ebx
:004C8370 56                  push esi
:004C8371 8BF2                mov esi, edx
:004C8373 8BD8                mov ebx, eax      //ebx=eax=机器码的 16 进制
:004C8375 33C0                xor eax, eax
:004C8377 55                  push ebp
:004C8378 68B0844C00          push 004C84B0
:004C837D 64FF30              push dword ptr fs:[eax]
:004C8380 648920              mov dword ptr fs:[eax], esp
:004C8383 81F38776FBDD        xor ebx, DDFB7687      //ebx=机器码 xor DDFB7687 =D87829A5
:004C8389 8BC3                mov eax, ebx        //eax=ebx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004C831C(C)
|
:004C838B 33D2                xor edx, edx
:004C838D 52                  push edx
:004C838E 50                  push eax
:004C838F 8D45FC              lea eax, dword ptr [ebp-04]
:004C8392 E8E50DF4FF          call 0040917C
:004C8397 8B45FC              mov eax, dword ptr [ebp-04]        //eax=ebx 的 10 进制值 3631753637
:004C839A 0FB600              movzx eax, byte ptr [eax]        //取 3631753637 的首位字符 hex 值 33 送 eax
:004C839D 8B55FC              mov edx, dword ptr [ebp-04]
:004C83A0 0FB65201            movzx edx, byte ptr [edx+01]        //取 3631753637 的第二位字符 hex 值 36 送 edx
:004C83A4 03C2                add eax, edx        //eax=eax+edx=69
:004C83A6 B905000000          mov ecx, 00000005      //ecx=5
:004C83AB 99                  cdq
:004C83AC F7F9                idiv ecx        //eax / ecx
:004C83AE 80C261              add dl, 61      //dl=余数 dl 加上 61=61
:004C83B1 8855F8              mov byte ptr [ebp-08], dl        //字符"a"(0x61)送[ebp-8]
:004C83B4 8B45FC              mov eax, dword ptr [ebp-04]
:004C83B7 0FB64002            movzx eax, byte ptr [eax+02]        //取 3631753637 的第三位字符 hex 值 33 送 eax
:004C83BB 8B55FC              mov edx, dword ptr [ebp-04]
:004C83BE 0FB65203            movzx edx, byte ptr [edx+03]        //取 3631753637 的第四位字符 hex 值 31 送 edx
:004C83C2 03C2                add eax, edx        //eax=eax+edx=64
:004C83C4 B905000000          mov ecx, 00000005
:004C83C9 99                  cdq
:004C83CA F7F9                idiv ecx
:004C83CC 80C261              add dl, 61      //dl=余数 dl 加上 61=61
:004C83CF 8855F9              mov byte ptr [ebp-07], dl        //字符"a"(0x61)送[ebp-7]
:004C83D2 8B45FC              mov eax, dword ptr [ebp-04]
```
