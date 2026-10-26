
```
004DE388 >/$  55            push    ebp                              ;  BigDecrypt

  004DE389  |.  8BEC          mov     ebp, esp

  004DE38B  |.  53            push    ebx

  004DE38C  |.  8B5D 08       mov     ebx, dword ptr [ebp+8]

  004DE38F  |.  53            push    ebx

  004DE390  |.  6A 00         push    0

  004DE392  |.  92            xchg    eax, edx

  004DE393  |.  E8 60FDFFFF   call    <BigDecrypt0>

  004DE398  |.  5B            pop     ebx

  004DE399  |.  5D            pop     ebp

  004DE39A  \.  C2 0400       retn    4

  004DE0F8 >/$  55            push    ebp                              ;  BigDecrypt0

  004DE0F9  |.  8BEC          mov     ebp, esp

  004DE0FB  |.  83C4 E4       add     esp, -1C

  004DE0FE  |.  53            push    ebx

  004DE0FF  |.  56            push    esi

  004DE100  |.  57            push    edi

  004DE101  |.  894D F4       mov     dword ptr [ebp-C], ecx           ;  [ebp-C]=832位大数

  004DE104  |.  8955 F8       mov     dword ptr [ebp-8], edx           ;  [ebp-8]=要解密的0x68个字节

  004DE107  |.  8945 FC       mov     dword ptr [ebp-4], eax           ;  [ebp-4]=65537

  004DE10A  |.  C605 04336D00>mov     byte ptr [6D3304], 0

  004DE111  |.  33D2          xor     edx, edx

  004DE113  |.  8B45 FC       mov     eax, dword ptr [ebp-4]

  004DE116  |.  E8 9DEFFFFF   call    004DD0B8

  004DE11B  |.  85C0          test    eax, eax

  004DE11D  |.  75 14         jnz     short 004DE133

  004DE11F  |.  8B45 0C       mov     eax, dword ptr [ebp+C]

  004DE122  |.  8B00          mov     eax, dword ptr [eax]

  004DE124  |.  BA 02000000   mov     edx, 2

  004DE129  |.  E8 6EE8FFFF   call    <BigIn16>

  004DE12E  |.  E9 4A020000   jmp     004DE37D

  004DE133  |>  B2 01         mov     dl, 1

  004DE135  |.  B8 48C64D00   mov     eax, 004DC648

  004DE13A  |.  E8 8DFEFFFF   call    <BigCreate>

  004DE13F  |.  8945 F0       mov     dword ptr [ebp-10], eax          ;  [ebp-10]=temp1

  004DE142  |.  B2 01         mov     dl, 1

  004DE144  |.  B8 48C64D00   mov     eax, 004DC648

  004DE149  |.  E8 7EFEFFFF   call    <BigCreate>

  004DE14E  |.  8BF0          mov     esi, eax                         ;  esi=temp2

  004DE150  |.  B2 01         mov     dl, 1

  004DE152  |.  B8 48C64D00   mov     eax, 004DC648

  004DE157  |.  E8 70FEFFFF   call    <BigCreate>

  004DE15C  |.  8BF8          mov     edi, eax                         ;  edi=temp3

  004DE15E  |.  B2 01         mov     dl, 1

  004DE160  |.  B8 48C64D00   mov     eax, 004DC648

  004DE165  |.  E8 62FEFFFF   call    <BigCreate>

  004DE16A  |.  8BD8          mov     ebx, eax                         ;  ebx=temp4

  004DE16C  |.  807D 08 00    cmp     byte ptr [ebp+8], 0

  004DE170  |.  74 1F         je      short 004DE191

  004DE172  |.  C745 E8 803E0>mov     dword ptr [ebp-18], 3E80

  004DE179  |.  8B45 E8       mov     eax, dword ptr [ebp-18]

  004DE17C  |.  E8 0345F2FF   call    00402684

  004DE181  |.  8945 E4       mov     dword ptr [ebp-1C], eax

  004DE184  |.  8B45 E4       mov     eax, dword ptr [ebp-1C]

  004DE187  |.  33C9          xor     ecx, ecx

  004DE189  |.  8B55 E8       mov     edx, dword ptr [ebp-18]

  004DE18C  |.  E8 2B49F2FF   call    00402ABC

  004DE191  |>  8B45 FC       mov     eax, dword ptr [ebp-4]           ;  [ebp-4]=e=0x10001

  004DE194  |.  E8 C7EDFFFF   call    <BigGetBitLength>                ;  得到二进制位长度

  004DE199  |.  48            dec     eax

  004DE19A  |.  8945 EC       mov     dword ptr [ebp-14], eax          ;  [ebp-14]=bitlength-1

  004DE19D  |.  8B55 F8       mov     edx, dword ptr [ebp-8]

  004DE1A0  |.  8BC3          mov     eax, ebx

  004DE1A2  |.  E8 F5EDFFFF   call    <BigCopy>                        ;  temp4=c

  004DE1A7  |.  8B4D F0       mov     ecx, dword ptr [ebp-10]

  004DE1AA  |.  8B55 F4       mov     edx, dword ptr [ebp-C]

  004DE1AD  |.  8BC3          mov     eax, ebx

  004DE1AF  |.  E8 F4F3FFFF   call    <BigDiv>                         ;  temp1=temp4%n=c%n temp4=temp4/n=c/n

  004DE1B4  |.  BA 02000000   mov     edx, 2

  004DE1B9  |.  8BC6          mov     eax, esi

  004DE1BB  |.  E8 DCE7FFFF   call    <BigIn16>                        ;  temp2=2

  004DE1C0  |.  8B55 F0       mov     edx, dword ptr [ebp-10]

  004DE1C3  |.  8BC7          mov     eax, edi

  004DE1C5  |.  E8 D2EDFFFF   call    <BigCopy>                        ;  temp3=temp1=c%n

  004DE1CA  |.  837D EC 00    cmp     dword ptr [ebp-14], 0

  004DE1CE  |.  0F8C 6F010000 jl      004DE343

  004DE1D4  |>  FF05 08336D00 /inc     dword ptr [6D3308]              ;  循环开始

  004DE1DA  |.  66:8B55 EC    |mov     dx, word ptr [ebp-14]

  004DE1DE  |.  8B45 FC       |mov     eax, dword ptr [ebp-4]

  004DE1E1  |.  E8 36EFFFFF   |call    <BigBitTest>                    ;  测试二进制位是否为1

  004DE1E6  |.  84C0          |test    al, al

  004DE1E8  |.  0F84 A0000000 |je      004DE28E

  004DE1EE  |.  8BD7          |mov     edx, edi

  004DE1F0  |.  8BC6          |mov     eax, esi

  004DE1F2  |.  E8 59F2FFFF   |call    <BigMul>                        ;  temp2=temp2*temp3

  004DE1F7  |.  8B55 F4       |mov     edx, dword ptr [ebp-C]

  004DE1FA  |.  8BC3          |mov     eax, ebx

  004DE1FC  |.  E8 9BEDFFFF   |call    <BigCopy>                       ;  temp4=n

  004DE201  |.  8B55 F0       |mov     edx, dword ptr [ebp-10]

  004DE204  |.  8BC3          |mov     eax, ebx

  004DE206  |.  E8 C5F1FFFF   |call    <BigSub>                        ;  temp4=temp4-temp1=n-c%n

  004DE20B  |.  8BD6          |mov     edx, esi

  004DE20D  |.  8BC3          |mov     eax, ebx

  004DE20F  |.  E8 58F1FFFF   |call    <BigAdd>                        ;  temp4=temp4+temp2=n-c%n+temp2*temp3

  004DE214  |.  807D 08 00    |cmp     byte ptr [ebp+8], 0

  004DE218  |.  74 13         |je      short 004DE22D

  004DE21A  |.  56            |push    esi

  004DE21B  |.  8B45 E4       |mov     eax, dword ptr [ebp-1C]

  004DE21E  |.  50            |push    eax

  004DE21F  |.  8BCB          |mov     ecx, ebx

  004DE221  |.  8B55 F4       |mov     edx, dword ptr [ebp-C]

  004DE224  |.  8BC3          |mov     eax, ebx

  004DE226  |.  E8 09F9FFFF   |call    <BigDiv0>

  004DE22B  |.  EB 0C         |jmp     short 004DE239

  004DE22D  |>  8BCE          |mov     ecx, esi

  004DE22F  |.  8B55 F4       |mov     edx, dword ptr [ebp-C]

  004DE232  |.  8BC3          |mov     eax, ebx

  004DE234  |.  E8 6FF3FFFF   |call    <BigDiv>                        ;  temp2=temp4%n=(n-c%n+temp2*temp3)%n=(temp2*temp3-c%n)%n

  004DE239  |>  8BD7          |mov     edx, edi

  004DE23B  |.  8BC7          |mov     eax, edi

  004DE23D  |.  E8 0EF2FFFF   |call    <BigMul>                        ;  temp3=temp3*temp3

  004DE242  |.  8B55 F4       |mov     edx, dword ptr [ebp-C]

  004DE245  |.  8BC3          |mov     eax, ebx

  004DE247  |.  E8 50EDFFFF   |call    <BigCopy>                       ;  temp4=n

  004DE24C  |.  BA 02000000   |mov     edx, 2

  004DE251  |.  8BC3          |mov     eax, ebx

  004DE253  |.  E8 28F5FFFF   |call    <BigSub16>                      ;  temp4=n-2

  004DE258  |.  8BD7          |mov     edx, edi

  004DE25A  |.  8BC3          |mov     eax, ebx

  004DE25C  |.  E8 0BF1FFFF   |call    <BigAdd>                        ;  temp4=temp4+temp3=n-2+temp3*temp3

  004DE261  |.  807D 08 00    |cmp     byte ptr [ebp+8], 0             ;  [ebp+8]==0

  004DE265  |.  74 16         |je      short 004DE27D

  004DE267  |.  57            |push    edi

  004DE268  |.  8B45 E4       |mov     eax, dword ptr [ebp-1C]

  004DE26B  |.  50            |push    eax

  004DE26C  |.  8BCB          |mov     ecx, ebx

  004DE26E  |.  8B55 F4       |mov     edx, dword ptr [ebp-C]

  004DE271  |.  8BC3          |mov     eax, ebx

  004DE273  |.  E8 BCF8FFFF   |call    <BigDiv0>

  004DE278  |.  E9 A9000000   |jmp     004DE326

  004DE27D  |>  8BCF          |mov     ecx, edi

  004DE27F  |.  8B55 F4       |mov     edx, dword ptr [ebp-C]

  004DE282  |.  8BC3          |mov     eax, ebx

  004DE284  |.  E8 1FF3FFFF   |call    <BigDiv>                        ;  temp3=temp4%n=(n-2+temp3*temp3)%n=(temp3*temp3-2)%n

  004DE289  |.  E9 98000000   |jmp     004DE326

  004DE28E  |>  8BD6          |mov     edx, esi                        ;  二进制位为0时跳到这

  004DE290  |.  8BC7          |mov     eax, edi

  004DE292  |.  E8 B9F1FFFF   |call    <BigMul>                        ;  temp3=temp3*temp2

  004DE297  |.  8B55 F4       |mov     edx, dword ptr [ebp-C]

  004DE29A  |.  8BC3          |mov     eax, ebx

  004DE29C  |.  E8 FBECFFFF   |call    <BigCopy>                       ;  temp4=n

  004DE2A1  |.  8B55 F0       |mov     edx, dword ptr [ebp-10]

  004DE2A4  |.  8BC3          |mov     eax, ebx

  004DE2A6  |.  E8 25F1FFFF   |call    <BigSub>                        ;  temp4=n-temp1=n-c%n

  004DE2AB  |.  8BD7          |mov     edx, edi

  004DE2AD  |.  8BC3          |mov     eax, ebx

  004DE2AF  |.  E8 B8F0FFFF   |call    <BigAdd>                        ;  temp4=temp4+temp3=n-c%n+temp3*temp2

  004DE2B4  |.  807D 08 00    |cmp     byte ptr [ebp+8], 0

  004DE2B8  |.  74 13         |je      short 004DE2CD

  004DE2BA  |.  57            |push    edi

  004DE2BB  |.  8B45 E4       |mov     eax, dword ptr [ebp-1C]

  004DE2BE  |.  50            |push    eax

  004DE2BF  |.  8BCB          |mov     ecx, ebx

  004DE2C1  |.  8B55 F4       |mov     edx, dword ptr [ebp-C]

  004DE2C4  |.  8BC3          |mov     eax, ebx

  004DE2C6  |.  E8 69F8FFFF   |call    <BigDiv0>

  004DE2CB  |.  EB 0C         |jmp     short 004DE2D9

  004DE2CD  |>  8BCF          |mov     ecx, edi

  004DE2CF  |.  8B55 F4       |mov     edx, dword ptr [ebp-C]

  004DE2D2  |.  8BC3          |mov     eax, ebx

  004DE2D4  |.  E8 CFF2FFFF   |call    <BigDiv>                        ;  temp3=temp4%n=(n-c%n+temp3*temp2)%n=(temp3*temp2-c%n)%n

  004DE2D9  |>  8BD6          |mov     edx, esi

  004DE2DB  |.  8BC6          |mov     eax, esi

  004DE2DD  |.  E8 6EF1FFFF   |call    <BigMul>                        ;  temp2=temp2*temp2

  004DE2E2  |.  8B55 F4       |mov     edx, dword ptr [ebp-C]

  004DE2E5  |.  8BC3          |mov     eax, ebx

  004DE2E7  |.  E8 B0ECFFFF   |call    <BigCopy>                       ;  temp4=n

  004DE2EC  |.  BA 02000000   |mov     edx, 2

  004DE2F1  |.  8BC3          |mov     eax, ebx

  004DE2F3  |.  E8 88F4FFFF   |call    <BigSub16>                      ;  temp4=tem4-2=n-2

  004DE2F8  |.  8BD6          |mov     edx, esi

  004DE2FA  |.  8BC3          |mov     eax, ebx

  004DE2FC  |.  E8 6BF0FFFF   |call    <BigAdd>                        ;  temp4=temp4+temp2=n-2+temp2*temp2

  004DE301  |.  807D 08 00    |cmp     byte ptr [ebp+8], 0             ;  [ebp+8]==0

  004DE305  |.  74 13         |je      short 004DE31A

  004DE307  |.  56            |push    esi

  004DE308  |.  8B45 E4       |mov     eax, dword ptr [ebp-1C]

  004DE30B  |.  50            |push    eax

  004DE30C  |.  8BCB          |mov     ecx, ebx

  004DE30E  |.  8B55 F4       |mov     edx, dword ptr [ebp-C]

  004DE311  |.  8BC3          |mov     eax, ebx

  004DE313  |.  E8 1CF8FFFF   |call    <BigDiv0>

  004DE318  |.  EB 0C         |jmp     short 004DE326

  004DE31A  |>  8BCE          |mov     ecx, esi

  004DE31C  |.  8B55 F4       |mov     edx, dword ptr [ebp-C]

  004DE31F  |.  8BC3          |mov     eax, ebx

  004DE321  |.  E8 82F2FFFF   |call    <BigDiv>                        ;  temp2=temp4%n=(n-2+temp2*temp2)%n=(temp2*temp2-2)%n

  004DE326  |>  FF4D EC       |dec     dword ptr [ebp-14]

  004DE329  |.  803D 04336D00>|cmp     byte ptr [6D3304], 0

  004DE330  |.  74 07         |je      short 004DE339

  004DE332  |.  C745 EC FFFFF>|mov     dword ptr [ebp-14], -1

  004DE339  |>  837D EC 00    |cmp     dword ptr [ebp-14], 0

  004DE33D  |.^ 0F8D 91FEFFFF \jge     004DE1D4                        ;  e从二进制高位至低位循环

  004DE343  |>  8B45 0C       mov     eax, dword ptr [ebp+C]

  004DE346  |.  8B00          mov     eax, dword ptr [eax]

  004DE348  |.  8BD6          mov     edx, esi

  004DE34A  |.  E8 4DECFFFF   call    <BigCopy>                        ;  return temp2

  004DE34F  |.  807D 08 00    cmp     byte ptr [ebp+8], 0

  004DE353  |.  74 0B         je      short 004DE360

  004DE355  |.  8B55 E8       mov     edx, dword ptr [ebp-18]

  004DE358  |.  8B45 E4       mov     eax, dword ptr [ebp-1C]

  004DE35B  |.  E8 3C43F2FF   call    <FreeMem>

  004DE360  |>  8B45 F0       mov     eax, dword ptr [ebp-10]

  004DE363  |.  E8 E049F2FF   call    <Free>

  004DE368  |.  8BC6          mov     eax, esi

  004DE36A  |.  E8 D949F2FF   call    <Free>

  004DE36F  |.  8BC7          mov     eax, edi

  004DE371  |.  E8 D249F2FF   call    <Free>

  004DE376  |.  8BC3          mov     eax, ebx

  004DE378  |.  E8 CB49F2FF   call    <Free>

  004DE37D  |>  5F            pop     edi

  004DE37E  |.  5E            pop     esi

  004DE37F  |.  5B            pop     ebx

  004DE380  |.  8BE5          mov     esp, ebp

  004DE382  |.  5D            pop     ebp

  004DE383  \.  C2 0800       retn    8
```
