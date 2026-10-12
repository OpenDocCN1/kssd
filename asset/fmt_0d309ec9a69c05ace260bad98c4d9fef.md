
```
101EDFF9       FF7424 1C                     push dword ptr ss:[esp+1C]

101EDFFD       68 B4856A10                  push CVI_1.106A85B4               ; ASCII "CVI_ADE_080000"

101EE002       56                           push esi

101EE003       56                           push esi

101EE004       6A 08                        push 8

101EE006       FF3485 10EB6F10              push dword ptr ds:[eax*4+106FEB10]

101EE00D       E8 CEAD3A00                  call CVI_1.10598DE0              ; F7 进去

----------------------------------------------------------------------------------------------

10598DE0       6A FF                        push -1

10598DE2       68 144C6910                  push CVI_1.10694C14

10598DE7       64:A1 00000000               mov eax,dword ptr fs:[0]

10598DED       50                           push eax

10598DEE       64:8925 00000000             mov dword ptr fs:[0],esp

10598DF5       81EC F8000000                sub esp,0F8

10598DFB       8B9424 20010000             mov edx,dword ptr ss:[esp+120]

10598E02       53                           push ebx

10598E03       55                           push ebp

10598E04       56                           push esi

10598E05       33DB                         xor ebx,ebx

10598E07       BD 0F000000                  mov ebp,0F

10598E0C       8BC2                         mov eax,edx

10598E0E       57                           push edi

10598E0F       894C24 10                    mov dword ptr ss:[esp+10],ecx

10598E13       89AC24 90000000             mov dword ptr ss:[esp+90],ebp

10598E1A       899C24 8C000000             mov dword ptr ss:[esp+8C],ebx

10598E21       885C24 7C                    mov byte ptr ss:[esp+7C],bl

10598E25       8D70 01                      lea esi,dword ptr ds:[eax+1]

10598E28       8A08                         mov cl,byte ptr ds:[eax]

10598E2A       40                           inc eax

10598E2B       3ACB                         cmp cl,bl

10598E2D    ^ 75 F9                         jnz short CVI_1.10598E28

10598E2F       2BC6                         sub eax,esi

10598E31       50                           push eax

10598E32       52                           push edx

10598E33       8D8C24 80000000             lea ecx,dword ptr ss:[esp+80]

10598E3A       E8 0430A9FF                 call CVI_1.1002BE43

10598E3F       8BB424 2C010000             mov esi,dword ptr ss:[esp+12C]

10598E46       8BC6                         mov eax,esi

10598E48       899C24 10010000             mov dword ptr ss:[esp+110],ebx

10598E4F       896C24 64                    mov dword ptr ss:[esp+64],ebp

10598E53       895C24 60                    mov dword ptr ss:[esp+60],ebx

10598E57       885C24 50                    mov byte ptr ss:[esp+50],bl

10598E5B       8D78 01                      lea edi,dword ptr ds:[eax+1]

10598E5E       8BFF                         mov edi,edi

10598E60       8A08                         mov cl,byte ptr ds:[eax]

10598E62       40                           inc eax

10598E63       3ACB                         cmp cl,bl

10598E65    ^ 75 F9                         jnz short CVI_1.10598E60

10598E67       2BC7                         sub eax,edi

10598E69       50                           push eax

10598E6A       56                           push esi

10598E6B       8D4C24 54                    lea ecx,dword ptr ss:[esp+54]

10598E6F       E8 CF2FA9FF                 call CVI_1.1002BE43

10598E74       8BB424 28010000             mov esi,dword ptr ss:[esp+128]

10598E7B       8BC6                         mov eax,esi

10598E7D       C68424 10010000 01          mov byte ptr ss:[esp+110],1

10598E85       896C24 48                    mov dword ptr ss:[esp+48],ebp

10598E89       895C24 44                    mov dword ptr ss:[esp+44],ebx

10598E8D       885C24 34                    mov byte ptr ss:[esp+34],bl

10598E91       8D78 01                      lea edi,dword ptr ds:[eax+1]

10598E94       8A08                         mov cl,byte ptr ds:[eax]

10598E96       40                           inc eax

10598E97       3ACB                         cmp cl,bl

10598E99    ^ 75 F9                         jnz short CVI_1.10598E94

10598E9B       2BC7                         sub eax,edi

10598E9D       50                           push eax

10598E9E       56                           push esi

10598E9F       8D4C24 38                    lea ecx,dword ptr ss:[esp+38]

10598EA3       E8 9B2FA9FF                 call CVI_1.1002BE43

10598EA8       8D8424 24010000             lea eax,dword ptr ss:[esp+124]

10598EAF       50                           push eax

10598EB0       8D4C24 74                    lea ecx,dword ptr ss:[esp+74]

10598EB4       C68424 14010000 02          mov byte ptr ss:[esp+114],2

10598EBC       E8 2F5D0000                 call CVI_1.1059EBF0

10598EC1       8D8C24 20010000             lea ecx,dword ptr ss:[esp+120]

10598EC8       51                           push ecx

10598EC9       8D4C24 6C                    lea ecx,dword ptr ss:[esp+6C]

10598ECD       E8 1E5D0000                 call CVI_1.1059EBF0

10598ED2       8D9424 1C010000             lea edx,dword ptr ss:[esp+11C]

10598ED9       52                           push edx

10598EDA       8D8C24 98000000             lea ecx,dword ptr ss:[esp+98]

10598EE1       E8 0A5D0000                 call CVI_1.1059EBF0

10598EE6       8BB424 18010000             mov esi,dword ptr ss:[esp+118]         ; Feature "CVI_FDS"

10598EED       8BC6                         mov eax,esi

10598EEF       896C24 2C                    mov dword ptr ss:[esp+2C],ebp

10598EF3       895C24 28                    mov dword ptr ss:[esp+28],ebx

10598EF7       885C24 18                    mov byte ptr ss:[esp+18],bl

10598EFB       8D78 01                      lea edi,dword ptr ds:[eax+1]

10598EFE       8BFF                         mov edi,edi

10598F00       8A08                         mov cl,byte ptr ds:[eax]

10598F02       40                           inc eax

10598F03       3ACB                         cmp cl,bl

10598F05    ^ 75 F9                         jnz short CVI_1.10598F00

10598F07       2BC7                         sub eax,edi                          ; Feature "CVI_Base"

10598F09       50                           push eax

10598F0A       56                           push esi

10598F0B       8D4C24 1C                    lea ecx,dword ptr ss:[esp+1C]

10598F0F       E8 2F2FA9FF                 call CVI_1.1002BE43

10598F14       8D4424 70                    lea eax,dword ptr ss:[esp+70]

10598F18       50                           push eax

10598F19       8D4C24 6C                    lea ecx,dword ptr ss:[esp+6C]

10598F1D       51                           push ecx

10598F1E       8D9424 9C000000             lea edx,dword ptr ss:[esp+9C]

10598F25       52                           push edx

10598F26       8D8C24 A8000000             lea ecx,dword ptr ss:[esp+A8]

10598F2D       C68424 1C010000 03          mov byte ptr ss:[esp+11C],3

10598F35       E8 865D0000                 call CVI_1.1059ECC0

10598F3A       8B8C24 3C010000             mov ecx,dword ptr ss:[esp+13C]

10598F41       8B9424 38010000             mov edx,dword ptr ss:[esp+138]

10598F48       51                           push ecx

10598F49       8B8C24 38010000             mov ecx,dword ptr ss:[esp+138]

10598F50       52                           push edx

10598F51       51                           push ecx

10598F52       8D9424 84000000             lea edx,dword ptr ss:[esp+84]

10598F59       52                           push edx

10598F5A       8D4C24 5C                    lea ecx,dword ptr ss:[esp+5C]
```
