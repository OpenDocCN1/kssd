
```
00408FE9  |.  A1 10B24100      mov     eax, dword ptr [41B210]
00408FEE  |.  83C4 24         add     esp, 24
00408FF1  |.  40              inc     eax
00408FF2  |.  5F              pop     edi
00408FF3  |.  A3 10B24100     mov     dword ptr [41B210], eax
00408FF8  |.  B8 01000000     mov     eax, 1
00408FFD  |.  5E              pop     esi
00408FFE  |.  5D              pop     ebp
00408FFF  |.  5B              pop     ebx
00409000  |.  8BE5            mov     esp, ebp
00409002  |.  5D              pop     ebp
00409003  \.  C3              retn
```

跟入 `00408FD4` (`E8 07FDFFFF`) 调用 `00408CE0`

```
00408CE0   $  55            push    ebp
00408CE1   .  8BEC         mov     ebp, esp
00408CE3   .  81EC A400000 sub     esp, 0A4
00408CE9   .  8B0D E8624100 mov     ecx, dword ptr [4162E8]
00408CEF   .  53            push    ebx
00408CF0   .  56            push    esi
00408CF1   .  8B75 08       mov     esi, dword ptr [ebp+8]
00408CF4   .  8BC6         mov     eax, esi
00408CF6   .  8908         mov     dword ptr [eax], ecx
00408CF8   .  8B15 EC624100 mov     edx, dword ptr [4162EC]
00408CFE   .  57            push    edi
00408CFF   .  8950 04       mov     dword ptr [eax+4], edx
00408D02   .  C745 EC F8624 mov     dword ptr [ebp-14], 004162F8          ;  ASCII "hello"
00408D09   .  C745 08 D45C4 mov     dword ptr [ebp+8], 00415CD4            ;  ASCII "info"
00408D10   .  74 03         je      short 00408D15
00408D12   .  75 01         jnz     short 00408D15
00408D14     E8            db      E8
00408D15   >  8D45 EC       lea     eax, dword ptr [ebp-14]
00408D18   .  8D5D 08       lea     ebx, dword ptr [ebp+8]
00408D1B   .  8B45 08       mov     eax, dword ptr [ebp+8]
00408D1E   .  8B4D EC       mov     ecx, dword ptr [ebp-14]
00408D21   .  50            push    eax
00408D22   .  51            push    ecx
00408D23   .  68 F0624100   push    004162F0                             ;  ASCII "%s,%s"
00408D28   .  E8 48480000   call    0040D575                             ;  格式化
00408D2D   .  83C4 0C       add     esp, 0C
00408D30   .  E8 CBF7FFFF   call    00408500                             ;  自身 hash
```

hash 值影响 `IdentityCode`，跟入后一路进入到这里

### hash 运算的函数过程

```
004084A0  /$  8B5424 08     mov     edx, dword ptr [esp+8]             ;  文件长度,nLen
004084A4  |.  8B4C24 04     mov     ecx, dword ptr [esp+4]             ;  指向文件数据,pData
004084A8  |.  83C8 FF       or      eax, FFFFFFFF
004084AB  |.  85D2         test    edx, edx
004084AD  |.  74 21        je      short 004084D0
004084AF  |.  56            push    esi
004084B0  |.  57            push    edi
004084B1  |>  0FB639        /movzx  edi, byte ptr [ecx]
004084B4  |.  8BF0         |mov     esi, eax
004084B6     81E6 FF000000 and     esi, 0FF
004084BC  |.  33F7         |xor     esi, edi
004084BE  |.  8B3CB5 889441>|mov     edi, dword ptr [esi*4+419488]
004084C5  |.  C1E8 08       |shr     eax, 8
004084C8  |.  33C7         |xor     eax, edi
004084CA  |.  41            |inc     ecx
004084CB  |.  4A            |dec     edx
004084CC  |.^ 75 E3        \jnz     short 004084B1
004084CE  |.  5F            pop     edi
004084CF  |.  5E            pop     esi
004084D0  |>  F7D0         not     eax
004084D2  \.  C3            retn
```

逻辑伪代码：

```c
int sum = 0xFFFFFFFF;
int i;
for (i = 0; i < nLen; i++) {
    sum = ((sum >> 8) ^ ((sum & 0xFF) ^ *pData)) * 4 + 0x419488;
    pData++;
}
```

hash 值是 `0x34DEB678`，脱壳以后要将整个函数改一下，比如改成：

```
mov eax, 34DEB678
retn
```

解决了自效验后返回到刚才那里继续

```
00408D35   .  8BF8         mov     edi, eax
00408D37   .  A1 60B64100  mov     eax, dword ptr [41B660]
00408D3C   .  33DB         xor     ebx, ebx
00408D3E   .  3BC3         cmp     eax, ebx
00408D40   .  75 06        jnz     short 00408D48
00408D42   .  893D 60B64100 mov    dword ptr [41B660], edi
00408D48   >  E8 93F7FFFF  call    004084E0
00408D4D   .  57           push    edi                              ; /HostLong
00408D4E   .  E8 29400000  call    <jmp.&ws2_32.htonl>              ; \htonl(自身 hash)
00408D53   .  8946 08      mov     dword ptr [esi+8], eax           ;  压入包的"linkage."后面
00408D56   .  E8 15F7FFFF  call    00408470
00408D5B   .  E8 80F7FFFF  call    004084E0
00408D60   .  E8 0BF7FFFF  call    00408470
00408D65   .  90           nop
00408D66   .  EB 00        jmp     short 00408D68
00408D68  / >  55         push    ebp
00408D69  |.  8BEC        mov     ebp, esp
00408D6B  |.  41          inc     ecx
00408D6C  |.  52          push    edx
00408D6D  |.  90          nop
00408D6E  |.  5A          pop     edx
00408D6F  |.  49          dec     ecx
00408D70  |.  5D          pop     ebp
00408D71  |.  41          inc     ecx
00408D72  |.  68 E700E700 push    0E700E7                            ; /NetLong = E700E7
00408D77  |.  E8 00400000 call    <jmp.&ws2_32.htonl>               ; \htonl(0xE700E7);
00408D7C  |.  8946 0C     mov     dword ptr [esi+C], eax             ;  这个也压入包后面
00408D7F  |.  33C0        xor     eax, eax
00408D81  |.  889D 5CFFFFFF mov    byte ptr [ebp-A4], bl
00408D87  |.  B9 1F000000 mov     ecx, 1F
00408D8C  |.  8DBD 5DFFFFFF lea    edi, dword ptr [ebp-A3]
00408D92  |.  F3:AB       rep     stos dword ptr es:[edi]
00408D94  |.  66:AB       stos    word ptr es:[edi]
00408D96  |.  AA          stos    byte ptr es:[edi]
00408D97  |.  33D2        xor     edx, edx
00408D99  |.  33C0        xor     eax, eax
00408D9B  |.  8955 F1     mov     dword ptr [ebp-F], edx
00408D9E  |.  8945 DD     mov     dword ptr [ebp-23], eax
00408DA1  |.  8955 F5     mov     dword ptr [ebp-B], edx
00408DA4  |.  8945 E1     mov     dword ptr [ebp-1F], eax
00408DA7  |.  8955 F9     mov     dword ptr [ebp-7], edx
00408DAA  |.  8945 E5     mov     dword ptr [ebp-1B], eax
00408DAD  |.  66:8955 FD  mov     word ptr [ebp-3], dx
00408DB1  |.  66:8945 E9  mov     word ptr [ebp-17], ax
00408DB5  |.  885D F0     mov     byte ptr [ebp-10], bl
00408DB8  |.  8855 FF     mov     byte ptr [ebp-1], dl
00408DBB  |.  8845 EB     mov     byte ptr [ebp-15], al
00408DBE  |.  8BFF        mov     edi, edi
00408DC0  |>  8A88 F0984100 /mov    cl, byte ptr [eax+4198F0]
00408DC6  |.  888C05 5CFFFF> mov    byte ptr [ebp+eax-A4], cl
00408DCD  |.  40           |inc     eax
00408DCE  |.  3ACB         |cmp     cl, bl
00408DD0  |.^ 75 EE        \jnz     short 00408DC0
00408DD2  |.  B8 F0984100  mov     eax, 004198F0                    ;  ASCII "0123456789012345678901234567890123456789"
00408DD7  |.  8D50 01      lea     edx, dword ptr [eax+1]             ;  记这个字符串为 strMagic
00408DDA  |.  8D9B 00000000 lea     ebx, dword ptr [ebx]
00408DE0  |>  8A08         /mov     cl, byte ptr [eax]
00408DE2  |.  40           |inc     eax
00408DE3  |.  3ACB         |cmp     cl, bl
00408DE5  |.^ 75 F9        \jnz     short 00408DE0
00408DE7  |.  2BC2        sub     eax, edx                           ;  strMagic 长度 n
00408DE9  |.  8BF8        mov     edi, eax
00408DEB  |.  55          push    ebp
00408DEC  |.  8BEC        mov     ebp, esp
00408DEE  |.  6A FF       push    -1
00408DF0  |.  68 07B20100 push    1B207
```
