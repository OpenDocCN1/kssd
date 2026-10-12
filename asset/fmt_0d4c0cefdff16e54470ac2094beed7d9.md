
> `00407B4F`       . `59`               `pop ecx`                             ; `kernel32.7C816FD7`  

> `00407B50`       . `59`               `pop ecx`                             ; `kernel32.7C816FD7`  

> `00407B51`       . `C3`               `retn`                               

>   

> `6A 60 68 B0 24 42 00 E8 F5 E1 FF FF BF 94 00 00 00 8B C7 E8 65 FC FF FF 89 65 E8 8B F4 89 3E 56`  

> `FF 15 70 02 42 00 8B 4E 10 89 0D 28 BB 42 00 8B 46 04 A3 34 BB 42 00 8B 56 08 89 15 38 BB 42 00`  

> `8B 76 0C 81 E6 FF 7F 00 00 89 35 2C BB 42 00 83 F9 02 74 0C 81 CE 00 80 00 00 89 35 2C BB 42 00`  

> `C1 E0 08 03 C2 A3 30 BB 42 00 33 F6 56 8B 3D 88 01 42 00 FF D7 66 81 38 4D 5A 75 1F 8B 48 3C 03`  

> `C8 81 39 50 45 00 00 75 12 0F B7 41 18 3D 0B 01 00 00 74 1F 3D 0B 02 00 00 74 05 89 75 E4 EB 27`  

> `83 B9 84 00 00 00 0E 76 F2 33 C0 39 B1 F8 00 00 00 EB 0E 83 79 74 0E 76 E2 33 C0 39 B1 E8 00 00`  

> `00 0F 95 C0 89 45 E4 6A 01 E8 A3 55 00 00 59 85 C0 75 08 6A 1C E8 02 FF FF FF 59 E8 57 2A 00 00`  

> `85 C0 75 08 6A 10 E8 F1 FE FF FF 59 E8 75 6A 00 00 89 75 FC E8 6F 68 00 00 85 C0 7D 08 6A 1B E8`  

> `B3 FE FF FF 59 FF 15 0C 02 42 00 A3 10 D4 42 00 E8 31 67 00 00 A3 14 BB 42 00 E8 85 66 00 00 85`  

> `C0 7D 08 6A 08 E8 8D FE FF FF 59 E8 41 64 00 00 85 C0 7D 08 6A 09 E8 7C FE FF FF 59 6A 01 E8 72`  

> `03 00 00 59 89 45 D8 3B C6 74 07 50 E8 66 FE FF FF 59 89 75 BC 8D 45 90 50 FF 15 B4 00 42 00 E8`  

> `B0 63 00 00 89 45 E0 F6 45 BC 01 74 06 0F B7 45 C0 EB 03 6A 0A 58 50 FF 75 E0 56 56 FF D7 50 E8`  

> `33 B5 00 00 8B F8 89 7D D4 39 75 E4 75 06 57 E8 4E 04 00 00 E8 6B 04 00 00 EB 2B 8B 45 EC 8B 08`  

> `8B 09 89 4D DC 50 51 E8 04 62 00 00 59 59 C3`  

>   

`****************************************************************`  

`Microsoft Visual C++ 7.0 Method2 [Debug]`  

>   

`.text`  

`.data`  

`.rsrc`  

>   

`0100739D` note>  `6A 70`               `push 70`  

`0100739F`         `68 98180001`           `push notepad.01001898`  

`010073A4`         `E8 BF010000`           `call notepad.01007568`  

`010073A9`         `33DB`                 `xor ebx,ebx`  

`010073AB`         `53`                   `push ebx`  

`010073AC`         `8B3D CC100001`        `mov edi,dword ptr ds:[<&KERNEL32.GetModuleHandleA>]`    ; `kernel32.GetModuleHandleA`  

`010073B2`         `FFD7`                 `call near edi`                                       ; `ntdll.7C930738`  

`010073B4`         `66:8138 4D5A`         `cmp word ptr ds:[eax],5A4D`  

`010073B9`         `75 1F`                `jnz short notepad.010073DA`  

`010073BB`         `8B48 3C`              `mov ecx,dword ptr ds:[eax+3C]`  

`010073BE`         `03C8`                 `add ecx,eax`  

`010073C0`         `8139 50450000`        `cmp dword ptr ds:[ecx],4550`  

`010073C6`         `75 12`                `jnz short notepad.010073DA`  

`010073C8`         `0FB741 18`            `movzx eax,word ptr ds:[ecx+18]`  

`010073CC`         `3D 0B010000`          `cmp eax,10B`  

`010073D1`         `74 1F`                `je short notepad.010073F2`  

`010073D3`         `3D 0B020000`          `cmp eax,20B`  

`010073D8`         `74 05`                `je short notepad.010073DF`  

`010073DA`         `895D E4`              `mov dword ptr ss:[ebp-1C],ebx`  

`010073DD`         `EB 27`                `jmp short notepad.01007406`  

`010073DF`         `83B9 84000000 0E`     `cmp dword ptr ds:[ecx+84],0E`  

`010073E6`     ^   `76 F2`                `jbe short notepad.010073DA`  

`010073E8`         `33C0`                 `xor eax,eax`  

`010073EA`         `3999 F8000000`        `cmp dword ptr ds:[ecx+F8],ebx`  

`010073F0`         `EB 0E`                `jmp short notepad.01007400`  

`010073F2`         `8379 74 0E`           `cmp dword ptr ds:[ecx+74],0E`  

`010073F6`     ^   `76 E2`                `jbe short notepad.010073DA`  

`010073F8`         `33C0`                 `xor eax,eax`  

`010073FA`         `3999 E8000000`        `cmp dword ptr ds:[ecx+E8],ebx`  

`01007400`         `0F95C0`               `setne al`  

`01007403`         `8945 E4`              `mov dword ptr ss:[ebp-1C],eax`  

`01007406`         `895D FC`              `mov dword ptr ss:[ebp-4],ebx`  

`01007409`         `6A 02`                `push 2`  

`0100740B`         `FF15 38130001`        `call near dword ptr ds:[<&msvcrt.__set_app_type>]`       ; `msvcrt.__set_app_type`  

`01007411`         `59`                   `pop ecx`                               ; `kernel32.7C816FD7`  

`01007412`         `830D 9CAB0001 FF`     `or dword ptr ds:[100AB9C],FFFFFFFF`  

`01007419`         `830D A0AB0001 FF`     `or dword ptr ds:[100ABA0],FFFFFFFF`  

`01007420`         `FF15 34130001`        `call near dword ptr ds:[<&msvcrt.__p__fmode>]`           ; `msvcrt.__p__fmode`  

`01007426`         `8B0D B89A0001`        `mov ecx,dword ptr ds:[1009AB8]`  

`0100742C`         `8908`                 `mov dword ptr ds:[eax],ecx`  

`0100742E`         `FF15 30130001`        `call near dword ptr ds:[<&msvcrt.__p__commode>]`         ; `msvcrt.__p__commode`  

`01007434`         `8B0D B49A0001`        `mov ecx,dword ptr ds:[1009AB4]`  

`0100743A`         `8908`                 `mov dword ptr ds:[eax],ecx`  

`0100743C`         `A1 2C130001`          `mov eax,dword ptr ds:[<&msvcrt._adjust_fdiv>]`  

`01007441`         `8B00`                 `mov eax,dword ptr ds:[eax]`  

`01007443`         `A3 A4AB0001`          `mov dword ptr ds:[100ABA4],eax`  

`01007448`         `E8 A7010000`          `call notepad.010075F4`  

`0100744D`         `391D 08960001`        `cmp dword ptr ds:[1009608],ebx`  

`01007453`         `75 0C`                `jnz short notepad.01007461`  

`01007455`         `68 F4750001`          `push notepad.010075F4`                         ; `Entry address`  

`0100745A`         `FF15 28130001`        `call near dword ptr ds:[<&msvcrt.__setusermatherr>]`    ; `msvcrt.__setusermatherr`  

`01007460`         `59`                   `pop ecx`                               ; `kernel32.7C816FD7`  

`01007461`         `E8 77010000`          `call notepad.010075DD`  

`01007466`         `68 10900001`          `push notepad.01009010`  

`0100746B`         `68 0C900001`          `push notepad.0100900C`  

`01007470`         `E8 5D010000`          `call <jmp.&msvcrt._initterm>`  

`01007475`         `A1 B09A0001`          `mov eax,dword ptr ds:[1009AB0]`  

`0100747A`         `8945 DC`              `mov dword ptr ss:[ebp-24],eax`  

`0100747D`         `8D45 DC`              `lea eax,dword ptr ss:[ebp-24]`  

`01007480`         `50`                   `push eax`  

`01007481`         `FF35 AC9A0001`        `push dword ptr ds:[1009AAC]`  

`01007487`         `8D45 D4`              `lea eax,dword ptr ss:[ebp-2C]`  

`0100748A`         `50`                   `push eax`  

`0100748B`         `8D45 D0`              `lea eax,dword ptr ss:[ebp-30]`  

`0100748E`         `50`                   `push eax`  

`0100748F`         `8D45 CC`              `lea eax,dword ptr ss:[ebp-34]`  

`01007492`         `50`                   `push eax`  

`01007493`         `FF15 20130001`        `call near dword ptr ds:[<&msvcrt.__getmainargs>]`       ; `msvcrt.__getmainargs`  

`01007499`         `8945 C8`              `mov dword ptr ss:[ebp-38],eax`  

`0100749C`         `68 08900001`          `push notepad.01009008`  

`010074A1`         `68 00900001`          `push notepad.01009000`  

`010074A6`         `E8 27010000`          `call <jmp.&msvcrt._initterm>`  

`010074AB`         `83C4 24`              `add esp,24`  

`010074AE`         `A1 1C130001`          `mov eax,dword ptr ds:[<&msvcrt._acmdln>]`  

`010074B3`         `8B30`                 `mov esi,dword ptr ds:[eax]`  

`010074B5`         `8975 E0`              `mov dword ptr ss:[ebp-20],esi`  

`010074B8`         `803E 22`              `cmp byte ptr ds:[esi],22`  

`010074BB`         `75 3A`                `jnz short notepad.010074F7`  

`010074BD`         `46`                   `inc esi`  

`010074BE`         `8975 E0`              `mov dword ptr ss:[ebp-20],esi`  
