
```
00407742  |> /8D5424 34     /lea     edx, dword ptr [esp+34]
00407746  |. |8D4C24 10     |lea     ecx, dword ptr [esp+10]
0040774A  |. |52            |push    edx
0040774B  |. |E8 FDCB0200   |call    0043434D                        ;  peterpan@2977+peterpan@2977
00407750  |. |8B4424 10     |mov     eax, dword ptr [esp+10]
00407754  |. |3970 F8       |cmp     dword ptr [eax-8], esi          ;  26位
00407757  |.^\7C E9         \jl      short 00407742
00407759  |>  56            push    esi
0040775A  |.  8D4C24 18     lea     ecx, dword ptr [esp+18]
0040775E  |.  6A 00         push    0
00407760  |.  51            push    ecx
00407761  |.  8D4C24 1C     lea     ecx, dword ptr [esp+1C]
00407765  |.  E8 F2740200   call    0042EC5C                         ;  截取前20位
0040776A  |.  50            push    eax
0040776B  |.  8D4C24 38     lea     ecx, dword ptr [esp+38]
0040776F  |.  C64424 2C 05  mov     byte ptr [esp+2C], 5
00407774  |.  E8 F5C80200   call    0043406E
00407779  |.  8D4C24 14     lea     ecx, dword ptr [esp+14]
0040777D  |.  C64424 28 03  mov     byte ptr [esp+28], 3
00407782  |.  E8 AEC70200   call    00433F35
00407787  |.  8B5424 38     mov     edx, dword ptr [esp+38]          ;  假码第一组
0040778B  |.  8D4424 38     lea     eax, dword ptr [esp+38]
0040778F  |.  50            push    eax
00407790  |.  8D4C24 14     lea     ecx, dword ptr [esp+14]
00407794  |.  8B7A F8       mov     edi, dword ptr [edx-8]
00407797  |.  E8 D2C80200   call    0043406E
0040779C  |.  3BFE          cmp     edi, esi
0040779E  |.  7D 3B         jge     short 004077DB
004077A0  |.  B3 06         mov     bl, 6
004077A2  |>  8D4C24 38     /lea     ecx, dword ptr [esp+38]
004077A6  |.  8D5424 10     |lea     edx, dword ptr [esp+10]
004077AA  |.  51            |push    ecx
004077AB  |.  8D4424 18     |lea     eax, dword ptr [esp+18]
004077AF  |.  52            |push    edx
004077B0  |.  50            |push    eax
004077B1  |.  E8 AEC90200   |call    00434164
004077B6  |.  50            |push    eax
004077B7  |.  8D4C24 14     |lea     ecx, dword ptr [esp+14]
004077BB  |.  885C24 2C     |mov     byte ptr [esp+2C], bl
004077BF  |.  E8 AAC80200   |call    0043406E
004077C4  |.  8D4C24 14     |lea     ecx, dword ptr [esp+14]
004077C8  |.  C64424 28 03  |mov     byte ptr [esp+28], 3
004077CD  |.  E8 63C70200   |call    00433F35
004077D2  |.  8B4C24 10     |mov     ecx, dword ptr [esp+10]
004077D6  |.  3971 F8       |cmp     dword ptr [ecx-8], esi
004077D9  |.^ 7C C7         \jl      short 004077A2                  ;  扩展长度
004077DB  |>  56            push    esi
004077DC  |.  8D5424 18     lea     edx, dword ptr [esp+18]
004077E0  |.  6A 00         push    0
004077E2  |.  52            push    edx
004077E3  |.  8D4C24 1C     lea     ecx, dword ptr [esp+1C]
004077E7  |.  E8 70740200   call    0042EC5C                         ;  截取前20位
004077EC  |.  50            push    eax
004077ED  |.  8D4C24 3C     lea     ecx, dword ptr [esp+3C]
004077F1  |.  C64424 2C 07  mov     byte ptr [esp+2C], 7
004077F6  |.  E8 73C80200   call    0043406E
004077FB  |.  8D4C24 14     lea     ecx, dword ptr [esp+14]
004077FF  |.  C64424 28 03  mov     byte ptr [esp+28], 3
00407804  |.  E8 2CC70200   call    00433F35
00407809  |.  33FF          xor     edi, edi
0040780B  |.  85F6          test    esi, esi
0040780D  |.  7E 20         jle     short 0040782F
0040780F  |>  8B4424 34     /mov     eax, dword ptr [esp+34]         ;  扩展后的假码第一组
00407813  |.  8B5424 38     |mov     edx, dword ptr [esp+38]         ;  扩展后的固定字符串
00407817  |.  8A0C07        |mov     cl, byte ptr [edi+eax]
0040781A  |.  8A0417        |mov     al, byte ptr [edi+edx]
0040781D  |.  32C1          |xor     al, cl                          ;  不过是xor
0040781F  |.  8D4C24 34     |lea     ecx, dword ptr [esp+34]
00407823  |.  50            |push    eax
00407824  |.  57            |push    edi
00407825  |.  E8 31CC0200   |call    0043445B                        ;  写回字符串
0040782A  |.  47            |inc     edi
0040782B  |.  3BFE          |cmp     edi, esi
0040782D  |.^ 7C E0         \jl      short 0040780F
0040782F  |>  8B5424 34     mov     edx, dword ptr [esp+34]
00407833  |.  33F6          xor     esi, esi
00407835  |>  32C9          /xor     cl, cl                          ;  每个大循环的初始值为0
00407837  |.  33C0          |xor     eax, eax
00407839  |.  85ED          |test    ebp, ebp
0040783B  |.  7E 0A         |jle     short 00407847
0040783D  |>  8A1C02        |/mov     bl, byte ptr [edx+eax]         ;  变形后的字符串取字符
00407840  |.  02CB          ||add     cl, bl                         ;  byte类型相加
00407842  |.  40            ||inc     eax
00407843  |.  3BC5          ||cmp     eax, ebp                       ;  4个一组作为数值
00407845  |.^ 7C F6         |\jl      short 0040783D
00407847  |>  884C34 14     |mov     byte ptr [esp+esi+14], cl
0040784B  |.  46            |inc     esi
0040784C  |.  03D5          |add     edx, ebp
0040784E  |.  83FE 05       |cmp     esi, 5                          ;  5次
00407851  |.^ 7C E2         \jl      short 00407835                  ;  这里是一个嵌套的For 循环
00407853  |.  68 701B4500   push    00451B70
00407858  |.  8D4C24 14     lea     ecx, dword ptr [esp+14]
0040785C  |.  E8 5DC80200   call    004340BE
00407861  |.  8B7424 18     mov     esi, dword ptr [esp+18]          ;  得出2组数值，这是第二组
00407865  |.  8B5C24 14     mov     ebx, dword ptr [esp+14]          ;  得出2组数值，这是第一组
00407869  |.  BF 08000000   mov     edi, 8
0040786E  |>  8AC3          /mov     al, bl                          ;  转化字符串的过程 select case
00407870  |.  24 1F         |and     al, 1F                          ;  and 1F --》0B11111
00407872  |.  884424 14     |mov     byte ptr [esp+14], al
00407876  |.  7C 08         |jl      short 00407880                  ;  小于0
00407878  |.  3C 03         |cmp     al, 3
0040787A  |.  7F 04         |jg      short 00407880
0040787C  |.  04 49         |add     al, 49                          ;  《=3，+49
0040787E  |.  EB 76         |jmp     short 004078F6
00407880  |>  3C 04         |cmp     al, 4
00407882  |.  7C 0E         |jl      short 00407892
00407884  |.  3C 07         |cmp     al, 7
00407886  |.  7F 0A         |jg      short 00407892
00407888  |.  B1 57         |mov     cl, 57
0040788A  |.  2AC8          |sub     cl, al                          ;  4-7是 57-
0040788C  |.  884C24 14     |mov     byte ptr [esp+14], cl
00407890  |.  EB 68         |jmp     short 004078FA
00407892  |>  3C 08         |cmp     al, 8
00407894  |.  7C 08         |jl      short 0040789E
00407896  |.  3C 0B         |cmp     al, 0B
00407898  |.  7F 04         |jg      short 0040789E
0040789A  |.  04 39         |add     al, 39                          ;  8-11 ，+39
0040789C  |.  EB 58         |jmp     short 004078F6
0040789E  |>  3C 0C         |cmp     al, 0C
004078A0  |.  7C 0E         |jl      short 004078B0
004078A2  |.  3C 0F         |cmp     al, 0F
004078A4  |.  7F 0A         |jg      short 004078B0
004078A6  |.  B2 41         |mov     dl, 41
004078A8  |.  2AD0          |sub     dl, al
004078AA  |.  885424 14     |mov     byte ptr [esp+14], dl
004078AE  |.  EB 4A         |jmp     short 004078FA
004078B0  |>  3C 10         |cmp     al, 10
004078B2  |.  7C 08         |jl      short 004078BC
004078B4  |.  3C 13         |cmp     al, 13
004078B6  |.  7F 04         |jg      short 004078BC
004078B8  |.  04 35         |add     al, 35                          ;  10,11,12,13 时+35
004078BA  |.  EB 3A         |jmp     short 004078F6
004078BC  |>  3C 14         |cmp     al, 14
004078BE  |.  7C 0E         |jl      short 004078CE
004078C0  |.  3C 17         |cmp     al, 17
004078C2  |.  7F 0A         |jg      short 004078CE
004078C4  |.  B1 6B         |mov     cl, 6B
004078C6  |.  2AC8          |sub     cl, al
004078C8  |.  884C24 14     |mov     byte ptr [esp+14], cl
004078CC  |.  EB 2C         |jmp     short 004078FA
004078CE  |>  3C 18         |cmp     al, 18
004078D0  |.  7C 08         |jl      short 004078DA
004078D2  |.  3C 1B         |cmp     al, 1B
004078D4  |.  7F 04         |jg      short 004078DA
004078D6  |.  04 1E         |add     al, 1E
004078D8  |.  EB 1C         |jmp     short 004078F6
004078DA  |>  3C 1C         |cmp     al, 1C
004078DC  |.  7C 0E         |jl      short 004078EC
004078DE  |.  3C 1D         |cmp     al, 1D
004078E0  |.  7F 0A         |jg      short 004078EC
004078E2  |.  B2 75         |mov     dl, 75
004078E4  |.  2AD0          |sub     dl, al
004078E6  |.  885424 14     |mov     byte ptr [esp+14], dl
004078EA  |.  EB 0E         |jmp     short 004078FA
004078EC  |>  3C 1E         |cmp     al, 1E
004078EE  |.  7C 0A         |jl      short 004078FA
004078F0  |.  3C 1F         |cmp     al, 1F
004078F2  |.  7F 06         |jg      short 004078FA
004078F4  |.  04 2F         |add     al, 2F
004078F6  |>  884424 14     |mov     byte ptr [esp+14], al           ;  计算结果放回去
004078FA  |>  8B4424 14     |mov     eax, dword ptr [esp+14]
004078FE  |.  8D4C24 10     |lea     ecx, dword ptr [esp+10]
00407902  |.  50            |push    eax
00407903  |.  E8 30CA0200   |call    00434338                        ;  转回字符串
00407908  |.  B9 05000000   |mov     ecx, 5
0040790D  |.  8BC3          |mov     eax, ebx                        ;  两组数据shrd
0040790F  |.  8BD6          |mov     edx, esi
00407911  |.  E8 BA9A0100   |call    004213D0
00407916  |.  4F            |dec     edi
00407917  |.  8BD8          |mov     ebx, eax
00407919  |.  8BF2          |mov     esi, edx
0040791B  |.^ 0F85 4DFFFFFF \jnz     0040786E
00407921  |.  8B7424 30     mov     esi, dword ptr [esp+30]
```
