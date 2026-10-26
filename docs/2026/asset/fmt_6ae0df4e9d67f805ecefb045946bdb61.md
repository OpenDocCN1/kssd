
```
0060E679  \.  C2 0400       retn    4
```

来到这里：

```
0060E2E0  /$  55            push    ebp
0060E2E1  |.  8BEC          mov     ebp, esp
0060E2E3  |.  51            push    ecx
0060E2E4  |.  B9 06000000   mov     ecx, 6
0060E2E9  |>  6A 00         /push    0
0060E2EB  |.  6A 00         |push    0
0060E2ED  |.  49            |dec     ecx
0060E2EE  |.^ 75 F9         \jnz     short 0060E2E9
0060E2F0  |.  874D FC       xchg    [ebp-4], ecx
0060E2F3  |.  53            push    ebx
0060E2F4  |.  56            push    esi
0060E2F5  |.  57            push    edi
0060E2F6  |.  884D FB       mov     [ebp-5], cl
0060E2F9  |.  8BFA          mov     edi, edx
0060E2FB  |.  8945 FC       mov     [ebp-4], eax
0060E2FE  |.  8B45 FC       mov     eax, [ebp-4]
0060E301  |.  E8 0A7CDFFF   call    00405F10
0060E306  |.  33C0          xor     eax, eax
0060E308  |.  55            push    ebp
0060E309  |.  68 08E56000   push    0060E508
0060E30E  |.  64:FF30       push    dword ptr fs:[eax]
0060E311  |.  64:8920       mov     fs:[eax], esp
0060E314  |.  BB 21000000   mov     ebx, 21                          将21赋予EBX，21是字符!的ASC码值
0060E319  |.  EB 16         jmp     short 0060E331
0060E31B  |>  8D45 EC       /lea     eax, [ebp-14]
0060E31E  |.  8BD3          |mov     edx, ebx
0060E320  |.  E8 F778DFFF   |call    00405C1C
0060E325  |.  8B55 EC       |mov     edx, [ebp-14]
0060E328  |.  8D45 FC       |lea     eax, [ebp-4]
0060E32B  |.  E8 F879DFFF   |call    00405D28
0060E330  |.  43            |inc     ebx                              如果用户名长度不够十位，则从!开始补够。
0060E331  |>  8B45 FC        mov     eax, [ebp-4]
0060E334  |.  E8 E779DFFF   |call    00405D20                            计算用户名的长度
0060E339  |.  83F8 0A       |cmp     eax, 0A                          与A比较，小于十位则跳转到0060E31B
0060E33C  |.^ 7C DD         \jl      short 0060E31B
0060E33E  |.  8D55 E8       lea     edx, [ebp-18]
0060E341  |.  8B45 FC       mov     eax, [ebp-4]
0060E344  |.  E8 A7D0DFFF   call    0040B3F0                          将用户名转换为大写
0060E349  |.  8B55 E8       mov     edx, [ebp-18]
0060E34C  |.  8D45 FC       lea     eax, [ebp-4]
0060E34F  |.  E8 7877DFFF   call    00405ACC
0060E354  |.  8B45 FC       mov     eax, [ebp-4]
0060E357  |.  E8 C479DFFF   call    00405D20                         计算用户名的长度（如果用户名含有中文字符
0060E35C  |.  8BF0          mov     esi, eax                          则一个中文字符按两位算）
0060E35E  |.  85F6          test    esi, esi                          ESI＝用户名的长度
0060E360  |.  7E 5E         jle     short 0060E3C0
0060E362  |.  BB 01000000   mov     ebx, 1
0060E367  |>  8B45 FC       /mov     eax, [ebp-4]
0060E36A  |.  0FB64418 FF   |movzx   eax, byte ptr [eax+ebx-1]        按位取用户名ASC码值赋予EAX
0060E36F  |.  8B55 0C       |mov     edx, [ebp+C]                     [ebp+C]＝1A
0060E372  |.  83C2 46       |add     edx, 46                          1A+46＝EDX＝60
0060E375  |.  3BC2          |cmp     eax, edx                         EAx与EDX比较
0060E377  |.  7E 22         |jle     short 0060E39B                   小于等于零则跳转到0060E39B
0060E379  |.  8B45 FC       |mov     eax, [ebp-4]
0060E37C  |.  0FB64418 FF   |movzx   eax, byte ptr [eax+ebx-1]        按位取用户名ASC码值赋予EAX
0060E381  |.  8D143B        |lea     edx, [ebx+edi]
0060E384  |.  2BC2          |sub     eax, edx                         EAX＝EAX－EDX
0060E386  |.  8D55 E4       |lea     edx, [ebp-1C]
0060E389  |.  E8 BEDBDFFF   |call    0040BF4c                         算法运算部分，将EAx的值转换为字符串。
0060E38E  |.  8B55 E4       |mov     edx, [ebp-1C]
0060E391  |.  8D45 F4       |lea     eax, [ebp-C]
0060E394  |.  E8 8F79DFFF   |call    00405D28
0060E399  |.  EB 20         |jmp     short 0060E3BB
0060E39B  |>  8B45 FC       |mov     eax, [ebp-4]
0060E39E  |.  0FB64418 FF   |movzx   eax, byte ptr [eax+ebx-1]        按位取用户名ASC码值赋予EAX
0060E3A3  |.  8D143B        |lea     edx, [ebx+edi]                   EBX＝1 （专业版时EDI＝10，标准版时EDI=26,移动版时EDI＝1C）  EDX＝EBX＋EDI
0060E3A6  |.  03C2          |add     eax, edx                         EAX＝EAX＋EDX
0060E3A8  |.  8D55 E0       |lea     edx, [ebp-20]
0060E3AB  |.  E8 9CDBDFFF   |call    0040BF4C                         算法运算部分，将EAx的值转换为字符串。
0060E3B0  |.  8B55 E0       |mov     edx, [ebp-20]
0060E3B3  |.  8D45 F4       |lea     eax, [ebp-C]
0060E3B6  |.  E8 6D79DFFF   |call    00405D28
0060E3BB  |>  47            |inc     edi                              EDI＋1
0060E3BC  |.  43            |inc     ebx                              EBX＋1
0060E3BD  |.  4E            |dec     esi
0060E3BE  |.^ 75 A7         \jnz     short 0060E367                   直到将用户名全部运算完毕。
0060E3C0  |>  BB 31000000   mov     ebx, 31
0060E3C5  |.  EB 20         jmp     short 0060E3E7                    跳转到0060E3E7
0060E3C7  |>  8D45 DC       /lea     eax, [ebp-24]
0060E3CA  |.  8BD3          |mov     edx, ebx
0060E3CC  |.  E8 4B78DFFF   |call    00405C1C
0060E3D1  |.  8B55 DC       |mov     edx, [ebp-24]
0060E3D4  |.  8D45 F4       |lea     eax, [ebp-C]
0060E3D7  |.  E8 4C79DFFF   |call    00405D28
0060E3DC  |.  43            |inc     ebx
0060E3DD  |.  83FB 39       |cmp     ebx, 39                          如果算法运算部分产生的字符串长度不够20位
0060E3E0  |.  75 05         |jnz     short 0060E3E7                   则用12345678补够。
0060E3E2  |.  BB 31000000   |mov     ebx, 31
0060E3E7  |>  8B45 F4        mov     eax, [ebp-C]                     算法运算部分产生的字符串赋予EAX
0060E3EA  |.  E8 3179DFFF   |call    00405D20                         计算其长度
0060E3EF  |.  83F8 14       |cmp     eax, 14                          要求最少20位，否者跳转到0060E3C7
0060E3F2  |.^ 7C D3         \jl      short 0060E3C7                   ＝＝＝＝＝＝＝＝＝＝＝＝＝＝
0060E3F4  |.  BB 01000000   mov     ebx, 1
0060E3F9  |>  8D45 D8       /lea     eax, [ebp-28]
0060E3FC  |.  8B55 F4       |mov     edx, [ebp-C]
0060E3FF  |.  8A541A FF     |mov     dl, [edx+ebx-1]
0060E403  |.  E8 1478DFFF   |call    00405C1C                         取EAX的前5位
0060E408  |.  8B55 D8       |mov     edx, [ebp-28]
0060E40B  |.  8D45 F0       |lea     eax, [ebp-10]
0060E40E  |.  E8 1579DFFF   |call    00405D28
0060E413  |.  43            |inc     ebx
0060E414  |.  83FB 06       |cmp     ebx, 6
0060E417  |.^ 75 E0         \jnz     short 0060E3F9                   ＝＝＝＝＝＝＝＝＝＝＝＝＝＝＝
0060E419  |.  8D45 F0       lea     eax, [ebp-10]
0060E41C  |.  BA 20E56000   mov     edx, 0060E520
0060E421  |.  E8 0279DFFF   call    00405D28
0060E426  |.  8D45 F0       lea     eax, [ebp-10]
0060E429  |.  33D2          xor     edx, edx
0060E42B  |.  8A55 FB       mov     dl, [ebp-5]
0060E42E  |.  8B1495 D861AB>mov     edx, [edx\*4+AB61D8]
0060E435  |.  E8 EE78DFFF   call    00405D28
0060E43A  |.  BB 03000000   mov     ebx, 3
0060E43F  |>  8B45 F4       /mov     eax, [ebp-C]
0060E442  |.  E8 D978DFFF   |call    00405D20
0060E447  |.  8945 D0       |mov     [ebp-30], eax
0060E44A  |.  DB45 D0       |fild    dword ptr [ebp-30]               将字串的长度转换为十进制
0060E44D  |.  D835 24E56000 |fdiv    dword ptr [60E524]               将字串的长度除以2
0060E453  |.  E8 9C50DFFF   |call    004034F4
0060E458  |.  83C0 03       |add     eax, 3
0060E45B  |.  83D2 00       |adc     edx, 0
0060E45E  |.  52            |push    edx
0060E45F  |.  50            |push    eax
0060E460  |.  8BC3          |mov     eax, ebx
0060E462  |.  99            |cdq
0060E463  |.  290424        |sub     [esp], eax
0060E466  |.  195424 04     |sbb     [esp+4], edx
0060E46A  |.  58            |pop     eax                              ＝＝＝＝＝＝＝＝＝＝＝＝＝
0060E46B  |.  5A            |pop     edx
0060E46C  |.  8B55 F4       |mov     edx, [ebp-C]
0060E46F  |.  8A5402 FF     |mov     dl, [edx+eax-1]                  从字串的中间向后取三位
0060E473  |.  8D45 D4       |lea     eax, [ebp-2C]
0060E476  |.  E8 A177DFFF   |call    00405C1C
0060E47B  |.  8B55 D4       |mov     edx, [ebp-2C]
0060E47E  |.  8D45 F0       |lea     eax, [ebp-10]
0060E481  |.  E8 A278DFFF   |call    00405D28
0060E486  |.  4B            |dec     ebx
```
