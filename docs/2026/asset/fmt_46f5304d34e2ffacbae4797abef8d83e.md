
```asm
:0041E77F 8B5018                  mov edx, dword ptr [eax+18]

:0041E782 03CF                    add ecx, edi

:0041E784 89542430                mov dword ptr [esp+30], edx

:0041E788 8BD1                    mov edx, ecx

:0041E78A F7D2                    not edx

:0041E78C 23D5                    and edx, ebp

:0041E78E 8BE9                    mov ebp, ecx

:0041E790 23EF                    and ebp, edi

:0041E792 0BD5                    or edx, ebp

:0041E794 8B6C2430                mov ebp, dword ptr [esp+30]

:0041E798 03D5                    add edx, ebp

:0041E79A 8BE9                    mov ebp, ecx

:0041E79C 8DB416134630A8          lea esi, dword ptr [esi+edx-57CFB9ED]

:0041E7A3 8BD6                    mov edx, esi

:0041E7A5 C1EA0F                  shr edx, 0F

:0041E7A8 C1E611                  shl esi, 11

:0041E7AB 0BD6                    or edx, esi

:0041E7AD 8B701C                  mov esi, dword ptr [eax+1C]

:0041E7B0 03D1                    add edx, ecx

:0041E7B2 89742410                mov dword ptr [esp+10], esi

:0041E7B6 8BF2                    mov esi, edx

:0041E7B8 23EA                    and ebp, edx

:0041E7BA F7D6                    not esi

:0041E7BC 23F7                    and esi, edi

:0041E7BE 0BF5                    or esi, ebp

:0041E7C0 8B6C2410                mov ebp, dword ptr [esp+10]

:0041E7C4 03F5                    add esi, ebp

:0041E7C6 8B6C2458                mov ebp, dword ptr [esp+58]

:0041E7CA 8DAC2E019546FD          lea ebp, dword ptr [esi+ebp-02B96AFF]

:0041E7D1 8BF5                    mov esi, ebp

:0041E7D3 C1E616                  shl esi, 16

:0041E7D6 C1ED0A                  shr ebp, 0A

:0041E7D9 0BF5                    or esi, ebp

:0041E7DB 8B6820                  mov ebp, dword ptr [eax+20]

:0041E7DE 03F2                    add esi, edx

:0041E7E0 896C242C                mov dword ptr [esp+2C], ebp

:0041E7E4 89742458                mov dword ptr [esp+58], esi

:0041E7E8 8BEA                    mov ebp, edx

:0041E7EA 236C2458                and ebp, dword ptr [esp+58]

:0041E7EE F7D6                    not esi

:0041E7F0 23F1                    and esi, ecx

:0041E7F2 0BF5                    or esi, ebp

:0041E7F4 8B6C242C                mov ebp, dword ptr [esp+2C]

:0041E7F8 03F5                    add esi, ebp

:0041E7FA 8B6824                  mov ebp, dword ptr [eax+24]

:0041E7FD 896C2444                mov dword ptr [esp+44], ebp

:0041E801 8DBC37D8988069          lea edi, dword ptr [edi+esi+698098D8]

:0041E808 8BF7                    mov esi, edi

:0041E80A C1EE19                  shr esi, 19

:0041E80D C1E707                  shl edi, 07

:0041E810 0BF7                    or esi, edi

:0041E812 8B7C2458                mov edi, dword ptr [esp+58]

:0041E816 03F7                    add esi, edi

:0041E818 8BEE                    mov ebp, esi

:0041E81A F7D5                    not ebp

:0041E81C 23EA                    and ebp, edx

:0041E81E 23FE                    and edi, esi

:0041E820 0BEF                    or ebp, edi

:0041E822 8B7C2444                mov edi, dword ptr [esp+44]

:0041E826 03EF                    add ebp, edi

:0041E828 8D8C29AFF7448B          lea ecx, dword ptr [ecx+ebp-74BB0851]

:0041E82F 8B6C2458                mov ebp, dword ptr [esp+58]

:0041E833 8BF9                    mov edi, ecx

:0041E835 C1EF14                  shr edi, 14

:0041E838 C1E10C                  shl ecx, 0C

:0041E83B 0BF9                    or edi, ecx

:0041E83D 8B4828                  mov ecx, dword ptr [eax+28]

:0041E840 03FE                    add edi, esi

:0041E842 894C2424                mov dword ptr [esp+24], ecx

:0041E846 8BCF                    mov ecx, edi

:0041E848 F7D1                    not ecx

:0041E84A 23CD                    and ecx, ebp

:0041E84C 8BEF                    mov ebp, edi

:0041E84E 23EE                    and ebp, esi

:0041E850 0BCD                    or ecx, ebp

:0041E852 8B6C2424                mov ebp, dword ptr [esp+24]

:0041E856 03CD                    add ecx, ebp

:0041E858 8BEF                    mov ebp, edi

:0041E85A 8D940AB15BFFFF          lea edx, dword ptr [edx+ecx-0000A44F]

:0041E861 8BCA                    mov ecx, edx

:0041E863 C1E90F                  shr ecx, 0F

:0041E866 C1E211                  shl edx, 11

:0041E869 0BCA                    or ecx, edx

:0041E86B 8B502C                  mov edx, dword ptr [eax+2C]

:0041E86E 03CF                    add ecx, edi

:0041E870 8954243C                mov dword ptr [esp+3C], edx

:0041E874 8BD1                    mov edx, ecx

:0041E876 23E9                    and ebp, ecx

:0041E878 F7D2                    not edx

:0041E87A 23D6                    and edx, esi

:0041E87C 0BD5                    or edx, ebp

:0041E87E 8B6C243C                mov ebp, dword ptr [esp+3C]

:0041E882 03D5                    add edx, ebp

:0041E884 8B6C2458                mov ebp, dword ptr [esp+58]

:0041E888 8DAC2ABED75C89          lea ebp, dword ptr [edx+ebp-76A32842]

:0041E88F 8BD5                    mov edx, ebp

:0041E891 C1E216                  shl edx, 16

:0041E894 C1ED0A                  shr ebp, 0A

:0041E897 0BD5                    or edx, ebp

:0041E899 8B6830                  mov ebp, dword ptr [eax+30]

:0041E89C 03D1                    add edx, ecx

:0041E89E 896C241C                mov dword ptr [esp+1C], ebp

:0041E8A2 89542458                mov dword ptr [esp+58], edx

:0041E8A6 8BE9                    mov ebp, ecx

:0041E8A8 236C2458                and ebp, dword ptr [esp+58]

:0041E8AC F7D2                    not edx

:0041E8AE 23D7                    and edx, edi

:0041E8B0 0BD5                    or edx, ebp

:0041E8B2 8B6C241C                mov ebp, dword ptr [esp+1C]

:0041E8B6 03D5                    add edx, ebp

:0041E8B8 8B6834                  mov ebp, dword ptr [eax+34]

:0041E8BB 896C2434                mov dword ptr [esp+34], ebp

:0041E8BF 8DB4162211906B          lea esi, dword ptr [esi+edx+6B901122]

:0041E8C6 8BD6                    mov edx, esi

:0041E8C8 C1EA19                  shr edx, 19

:0041E8CB C1E607                  shl esi, 07

:0041E8CE 0BD6                    or edx, esi

:0041E8D0 8B742458                mov esi, dword ptr [esp+58]

:0041E8D4 03D6                    add edx, esi

:0041E8D6 8BEA                    mov ebp, edx

:0041E8D8 23F2                    and esi, edx

:0041E8DA F7D5                    not ebp

:0041E8DC 23E9                    and ebp, ecx

:0041E8DE 0BEE                    or ebp, esi

:0041E8E0 8B742434                mov esi, dword ptr [esp+34]

:0041E8E4 03EE                    add ebp, esi

:0041E8E6 8DBC2F937198FD          lea edi, dword ptr [edi+ebp-02678E6D]

:0041E8ED 8BF7                    mov esi, edi

:0041E8EF C1EE14                  shr esi, 14

:0041E8F2 C1E70C                  shl edi, 0C

:0041E8F5 0BF7                    or esi, edi

:0041E8F7 03F2                    add esi, edx

:0041E8F9 8BFE                    mov edi, esi

:0041E8FB F7D7                    not edi

:0041E8FD 897C244C                mov dword ptr [esp+4C], edi

:0041E901 8B6838                  mov ebp, dword ptr [eax+38]

:0041E904 8B403C                  mov eax, dword ptr [eax+3C]

:0041E907 896C2414                mov dword ptr [esp+14], ebp

:0041E90B 8B6C2458                mov ebp, dword ptr [esp+58]

:0041E90F 23FD                    and edi, ebp

:0041E911 8BEE                    mov ebp, esi

:0041E913 23EA                    and ebp, edx

:0041E915 89442448                mov dword ptr [esp+48], eax

:0041E919 0BFD                    or edi, ebp

:0041E91B 8B6C2414                mov ebp, dword ptr [esp+14]

:0041E91F 03FD                    add edi, ebp

:0041E921 8BEE                    mov ebp, esi

:0041E923 8D8C398E4379A6          lea ecx, dword ptr [ecx+edi-5986BC72]

:0041E92A 8BF9                    mov edi, ecx

:0041E92C C1EF0F                  shr edi, 0F

:0041E92F C1E111                  shl ecx, 11
```
