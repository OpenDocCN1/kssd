
```asm
8B6C2428 mov ebp, dword ptr [esp+28]
:00415EB9 C1E912 shr ecx, 12
:00415EBC 0BCA or ecx, edx
:00415EBE 03CF add ecx, edi
:00415EC0 8BD7 mov edx, edi
:00415EC2 33D1 xor edx, ecx
:00415EC4 23D6 and edx, esi
:00415EC6 33D7 xor edx, edi
:00415EC8 03D5 add edx, ebp
:00415ECA 8D9410ED145A45 lea edx, dword ptr [eax+edx+455A14ED]
:00415ED1 8BC2 mov eax, edx
:00415ED3 8B6C2430 mov ebp, dword ptr [esp+30]
:00415ED7 C1EA0C shr edx, 0C
:00415EDA 8B5C2444 mov ebx, dword ptr [esp+44]
:00415EDE C1E014 shl eax, 14
:00415EE1 0BC2 or eax, edx
:00415EE3 03C1 add eax, ecx
:00415EE5 8BD1 mov edx, ecx
:00415EE7 33D0 xor edx, eax
:00415EE9 23D7 and edx, edi
:00415EEB 33D1 xor edx, ecx
:00415EED 03D5 add edx, ebp
:00415EEF 8D941605E9E3A9 lea edx, dword ptr [esi+edx-561C16FB]
:00415EF6 8BF2 mov esi, edx
:00415EF8 C1E205 shl edx, 05
:00415EFB C1EE1B shr esi, 1B
:00415EFE 0BF2 or esi, edx
:00415F00 03F0 add esi, eax
:00415F02 8BD0 mov edx, eax
:00415F04 33D6 xor edx, esi
:00415F06 23D1 and edx, ecx
:00415F08 33D0 xor edx, eax
:00415F0A 03D3 add edx, ebx
:00415F0C 8DBC17F8A3EFFC lea edi, dword ptr [edi+edx-03105C08]
:00415F13 8BDF mov ebx, edi
:00415F15 C1E709 shl edi, 09
:00415F18 C1EB17 shr ebx, 17
:00415F1B 0BDF or ebx, edi
:00415F1D 8B7C2414 mov edi, dword ptr [esp+14]
:00415F21 8B6C2440 mov ebp, dword ptr [esp+40]
:00415F25 03DE add ebx, esi
:00415F27 8BD3 mov edx, ebx
:00415F29 33D6 xor edx, esi
:00415F2B 23D0 and edx, eax
:00415F2D 33D6 xor edx, esi
:00415F2F 03D7 add edx, edi
:00415F31 8D8C11D9026F67 lea ecx, dword ptr [ecx+edx+676F02D9]
:00415F38 8BF9 mov edi, ecx
:00415F3A C1E10E shl ecx, 0E
:00415F3D C1EF12 shr edi, 12
:00415F40 0BF9 or edi, ecx
:00415F42 03FB add edi, ebx
:00415F44 8BD3 mov edx, ebx
:00415F46 33D7 xor edx, edi
:00415F48 8BCA mov ecx, edx
:00415F4A 23CE and ecx, esi
:00415F4C 33CB xor ecx, ebx
:00415F4E 03CD add ecx, ebp
:00415F50 8B6C2418 mov ebp, dword ptr [esp+18]
:00415F54 8D84088A4C2A8D lea eax, dword ptr [eax+ecx-72D5B376]
:00415F5B 8BC8 mov ecx, eax
:00415F5D C1E114 shl ecx, 14
:00415F60 C1E80C shr eax, 0C
:00415F63 0BC8 or ecx, eax
:00415F65 03CF add ecx, edi
:00415F67 33D1 xor edx, ecx
:00415F69 03D5 add edx, ebp
:00415F6B 8DB4164239FAFF lea esi, dword ptr [esi+edx-0005C6BE]
:00415F72 8BC6 mov eax, esi
:00415F74 C1E604 shl esi, 04
:00415F77 8B6C2438 mov ebp, dword ptr [esp+38]
:00415F7B C1E81C shr eax, 1C
:00415F7E 0BC6 or eax, esi
:00415F80 8B742428 mov esi, dword ptr [esp+28]
:00415F84 03C1 add eax, ecx
:00415F86 8BD7 mov edx, edi
:00415F88 33D1 xor edx, ecx
:00415F8A 33D0 xor edx, eax
:00415F8C 03D6 add edx, esi
:00415F8E 8D9C1381F67187 lea ebx, dword ptr [ebx+edx-788E097F]
:00415F95 8BF3 mov esi, ebx
:00415F97 C1E30B shl ebx, 0B
:00415F9A C1EE15 shr esi, 15
:00415F9D 0BF3 or esi, ebx
:00415F9F 03F0 add esi, eax
:00415FA1 8BD6 mov edx, esi
:00415FA3 33D1 xor edx, ecx
:00415FA5 33D0 xor edx, eax
:00415FA7 03D5 add edx, ebp
:00415FA9 8D941722619D6D lea edx, dword ptr [edi+edx+6D9D6122]
:00415FB0 8BFA mov edi, edx
:00415FB2 8B6C2448 mov ebp, dword ptr [esp+48]
:00415FB6 C1EA10 shr edx, 10
:00415FB9 C1E710 shl edi, 10
:00415FBC 0BFA or edi, edx
:00415FBE 03FE add edi, esi
:00415FC0 8BD6 mov edx, esi
:00415FC2 33D7 xor edx, edi
:00415FC4 8BDA mov ebx, edx
:00415FC6 33D8 xor ebx, eax
:00415FC8 03DD add ebx, ebp
:00415FCA 8B6C2424 mov ebp, dword ptr [esp+24]
:00415FCE 8D9C190C38E5FD lea ebx, dword ptr [ecx+ebx-021AC7F4]
:00415FD5 8BCB mov ecx, ebx
:00415FD7 C1EB09 shr ebx, 09
:00415FDA C1E117 shl ecx, 17
:00415FDD 0BCB or ecx, ebx
:00415FDF 03CF add ecx, edi
:00415FE1 33D1 xor edx, ecx
:00415FE3 03D5 add edx, ebp
:00415FE5 8B6C2434 mov ebp, dword ptr [esp+34]
:00415FE9 8D941044EABEA4 lea edx, dword ptr [eax+edx-5B4115BC]
:00415FF0 8BC2 mov eax, edx
:00415FF2 C1E204 shl edx, 04
:00415FF5 C1E81C shr eax, 1C
:00415FF8 0BC2 or eax, edx
:00415FFA 03C1 add eax, ecx
:00415FFC 8BD7 mov edx, edi
:00415FFE 33D1 xor edx, ecx
:00416000 33D0 xor edx, eax
:00416002 03D5 add edx, ebp
:00416004 8B6C2414 mov ebp, dword ptr [esp+14]
:00416008 8D9416A9CFDE4B lea edx, dword ptr [esi+edx+4BDECFA9]
:0041600F 8BF2 mov esi, edx
:00416011 C1E20B shl edx, 0B
:00416014 C1EE15 shr esi, 15
:00416017 0BF2 or esi, edx
:00416019 03F0 add esi, eax
:0041601B 8BD6 mov edx, esi
:0041601D 33D1 xor edx, ecx
:0041601F 33D0 xor edx, eax
:00416021 03D5 add edx, ebp
:00416023 8B6C2420 mov ebp, dword ptr [esp+20]
:00416027 8D9417604BBBF6 lea edx, dword ptr [edi+edx-0944B4A0]
:0041602E 8BFA mov edi, edx
:00416030 C1EA10 shr edx, 10
:00416033 C1E710 shl edi, 10
:00416036 0BFA or edi, edx
:00416038 03FE add edi, esi
:0041603A 8BD6 mov edx, esi
:0041603C 33D7 xor edx, edi
:0041603E 8BDA mov ebx, edx
:00416040 33D8 xor ebx, eax
:00416042 03DD add ebx, ebp
:00416044 8D9C1970BCBFBE lea ebx, dword ptr [ecx+ebx-41404390]
:0041604B 8BCB mov ecx, ebx
:0041604D C1E117 shl ecx, 17
:00416050 C1EB09 shr ebx, 09
:00416053 8B6C2430 mov ebp, dword ptr [esp+30]
:00416057 0BCB or ecx, ebx
:00416059 03CF add ecx, edi
:0041605B 33D1 xor edx, ecx
:0041605D 03D5 add edx, ebp
:0041605F 8D9410C67E9B28 lea edx, dword ptr [eax+edx+289B7EC6]
:00416066 8BC2 mov eax, edx
:00416068 C1E204 shl edx, 04
:0041606B 8B6C2410 mov ebp, dword ptr [esp+10]
:0041606F C1E81C shr eax, 1C
:00416072 0BC2 or eax, edx
:00416074 03C1 add eax, ecx
:00416076 8BD7 mov edx, edi
:00416078 33D1 xor edx, ecx
:0041607A 33D0 xor edx, eax
:0041607C 03D5 add edx, ebp
:0041607E 8D9416FA27A1EA lea edx, dword ptr [esi+edx-155ED806]
:00416085 8BF2 mov esi, edx
:00416087 C1E20B shl edx, 0B
:0041608A 8B6C241C mov ebp, dword ptr [esp+1C]
:0041608E C1EE15 shr esi, 15
:00416091 0BF2 or esi, edx
:00416093 03F0 add esi, eax
:00416095 8BD6 mov edx, esi
:00416097 33D1 xor edx, ecx
:00416099 33D0 xor edx, eax
:0041609B 03D5
```
