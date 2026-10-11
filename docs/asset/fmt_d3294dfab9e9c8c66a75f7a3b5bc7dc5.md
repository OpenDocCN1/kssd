
```asm
;   si=0x033E  // 初始值
;   si=0xF416
;   si=0xA1DB
;   si=0x77AA
;   si=0xBDFA
;   si=0xEAC5
;   si=0xE320

;
004C82AA  |.  C1E9 08          |shr     ecx, 8                     ;  逻辑右移 2 位给 ECX
004C82AD  |.  32D1             |xor     dl, cl                     ;  异或运算，cl xor dl
;   0x03 xor 0x4B = 0x48
;   0xF4 xor 0x55 = 0xA1
;   0xA1 xor 0x4E = 0xEF
;   0x77 xor 0x47 = 0x30
;   0xBD xor 0x42 = 0xFF
;   0xEA xor 0x49 = 0xA3
;   0xE3 xor 0x4D = 0xAE

;
004C82AF  |.  885438 FF        |mov     byte ptr [eax+edi-1], dl   ;  ★得到第二次运算值★
;   dl=0x48 ('H')
;   dl=0xA1
;   dl=0xEF ('?')
;   dl=0x30 ('0')
;   dl=0xFF
;   dl=0xA3 ('?')
;   dl=0xAE ('?')

;
004C82B3  |. |8B45 00         |mov     eax, dword ptr [ebp]
004C82B6  |. |0FB64438 FF     |movzx   eax, byte ptr [eax+edi-1]
004C82BB  |. |66:03F0         |add     si, ax                     ;  加法运算，ax+si
;   0x48+0x33E=0x386
;   0xA1+0xF416=0xFFB7
;   0xEF+0xA1DB=0xA134
;   0x30+0x77AA=0x77DA
;   0xFF+0xBDFA=0xBEF9
;   0xA3+0xEAC5=0xEB68
;   0xAE+0xE320=0xE3CE

;
004C82BE  |. |66:69C6 D500    |imul    ax, si, 0D5                ;  整数乘法，0xD5*si*ax
004C82C3  |. |66:05 9805      |add     ax, 598                    ;  加法运算，0x598+ax
;   ax=0xF416
;   ax=0x9C43
;   ax=0x7212
;   ax=0xB862
;   ax=0xE52D
;   ax=0xDD88
;   ax=0x8A66
004C82C7  |. |8BF0            |mov     esi, eax
004C82C9  |. |43              |inc     ebx
004C82CA  |. |FE4C24 04       |dec     byte ptr [esp+4]
004C82CE  |.^\\75 C1          \\jnz     short 004C8291             ;  用户名循环运算
004C82D0  |> \\59            pop     ecx
004C82D1  |.  5A             pop     edx
004C82D2  |.  5D             pop     ebp
004C82D3  |.  5F             pop     edi
004C82D4  |.  5E             pop     esi
004C82D5  |.  5B             pop     ebx
004C82D6  \\.  C3            retn                       ;  返回，准备第二次运算

★第二次加密（转换）运算★

004C82D8  /$  55             push    ebp                    ;  call 004C82D8 来到这里
004C82D9  |.  8BEC          mov     ebp, esp
004C82DB  |.  33C9          xor     ecx, ecx
004C82DD  |.  51             push    ecx
004C82DE  |.  51             push    ecx
004C82DF  |.  51             push    ecx
004C82E0  |.  51             push    ecx
004C82E1  |.  51             push    ecx
004C82E2  |.  51             push    ecx
004C82E3  |.  53             push    ebx
004C82E4  |.  56             push    esi
004C82E5  |.  8955 F8       mov     dword ptr [ebp-8], edx
004C82E8  |.  8945 FC       mov     dword ptr [ebp-4], eax
004C82EB  |.  8B45 FC       mov     eax, dword ptr [ebp-4]
004C82EE  |.  E8 CDCBF3FF    call    00404EC0
004C82F3  |.  33C0          xor     eax, eax
004C82F5  |.  55             push    ebp
004C82F6  |.  68 B0834C00    push    004C83B0
004C82FB  |.  64:FF30      push    dword ptr fs:[eax]
004C82FE  |.  64:8920      mov     dword ptr fs:[eax], esp
004C8301  |.  8B45 FC       mov     eax, dword ptr [ebp-4]
004C8304  |.  E8 C7C9F3FF    call    00404CD0
004C8309  |.  8BF0          mov     esi, eax
004C830B  |.  85F6          test    esi, esi
004C830D  |.  7E 73          jle     short 004C8382
004C830F  |.  BB 01000000    mov     ebx, 1
004C8314  |>  8B45 FC        /mov    eax, dword ptr [ebp-4]       ;  依次取第一次用户名加密变换结果
004C8317  |.  8A4418 FF      |mov    al, byte ptr [eax+ebx-1]     ;  dl=0x48 ('H')
;   dl=0xA1
;   dl=0xEF ('?')
;   dl=0x30 ('0')
;   dl=0xFF
;   dl=0xA3 ('?')
;   dl=0xAE ('?')
004C831B  |.  3C 7A          |cmp    al, 7A
004C831D  |.  73 04          |jnb    short 004C8323
004C831F  |.  3C 61          |cmp    al, 61
004C8321  |.  77 11          |ja     short 004C8334
004C8323  |>  8B45 FC        |mov    eax, dword ptr [ebp-4]
004C8326  |.  8A5418 FF     |mov    dl, byte ptr [eax+ebx-1]
004C832A  |.  80FA 5A       |cmp    dl, 5A
004C832D  |.  73 21          |jnb    short 004C8350
004C832F  |.  80FA 41       |cmp    dl, 41
```
