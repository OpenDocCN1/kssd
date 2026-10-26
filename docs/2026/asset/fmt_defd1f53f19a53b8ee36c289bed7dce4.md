
```
5650D4B2                              |.  8B49 24       mov ecx, dword ptr [ecx+24]

5650D4B5                              |.  0F95C0        setne al

5650D4B8                              |.  0FB6C0        movzx eax, al

5650D4BB                              |.  50            push eax

5650D4BC                              |.  8BD7          mov edx, edi

5650D4BE                              |.  E8 4DFEFFFF   call CXCentau.5650D310

5650D4C3                              |.  83C4 14       add esp, 14

5650D4C6                              |.  5F            pop edi

5650D4C7                              |.  5E            pop esi

5650D4C8                              |.  C3            retn

5650D4C9                              |>  68 38FBA356   push CXCentau.56A3FB38                                                            ;  ASCII "cxc\_query"

5650D4CE                              |.  50            push eax

5650D4CF                              |.  68 9C5FA356   push CXCentau.56A35F9C                                                            ;  ASCII "Too many arguments(%d) to %s.",LF

5650D4D4                              |.  E8 37831B00   call CXCentau.566C5810

5650D4D9                              |.  CC            int3

5650D4DA                              |.  CC            int3

5650D4DB                              |.  CC            int3

5650D4DC                              |.  CC            int3

5650D4DD                              |.  CC            int3

5650D4DE                              |.  CC            int3

5650D4DF                              |.  CC            int3

5650D4E0                              |.  A1 B055D756   mov eax, dword ptr [56D755B0]

5650D4E5                              |.  8B40 04       mov eax, dword ptr [eax+4]

5650D4E8                              |.  85C0          test eax, eax

5650D4EA                              |.  7F 59         jg short CXCentau.5650D545

5650D4EC                              |.  56            push esi

5650D4ED                              |.  57            push edi

5650D4EE                              |.  50            push eax

5650D4EF                              |.  E8 0C4B1B00   call CXCentau.566C2000

5650D4F4                              |.  8B3D 30E2CA56 mov edi, dword ptr [56CAE230]

5650D4FA                              |.  8B35 34E2CA56 mov esi, dword ptr [56CAE234]

5650D500                              |.  6A 03         push 3

5650D502                              |.  E8 191B1B00   call CXCentau.566BF020

5650D507                              |.  50            push eax

5650D508                              |.  E8 D34D1B00   call CXCentau.566C22E0

5650D50D                              |.  57            push edi

5650D50E                              |.  E8 2D501B00   call CXCentau.566C2540

5650D513                              |.  68 68FCA356   push CXCentau.56A3FC68                                                            ;  ASCII "total\_alloc"

5650D518                              |.  E8 933E1B00   call CXCentau.566C13B0

5650D51D                              |.  56            push esi

5650D51E                              |.  E8 1D501B00   call CXCentau.566C2540

5650D523                              |.  68 74FCA356   push CXCentau.56A3FC74                                                            ;  ASCII "total\_free"

5650D528                              |.  E8 833E1B00   call CXCentau.566C13B0

5650D52D                              |.  2BFE          sub edi, esi

5650D52F                              |.  57            push edi

5650D530                              |.  E8 0B501B00   call CXCentau.566C2540

5650D535                              |.  68 80FCA356   push CXCentau.56A3FC80                                                            ;  ASCII "net"

5650D53A                              |.  E8 713E1B00   call CXCentau.566C13B0

5650D53F                              |.  83C4 24       add esp, 24

5650D542                              |.  5F            pop edi

5650D543                              |.  5E            pop esi

5650D544                              |.  C3            retn

5650D545                              |>  68 38FBA356   push CXCentau.56A3FB38                                                            ;  ASCII "cxc\_query"

5650D54A                              |.  50            push eax

5650D54B                              |.  68 9C5FA356   push CXCentau.56A35F9C                                                            ;  ASCII "Too many arguments(%d) to %s.",LF

5650D550                              |.  E8 BB821B00   call CXCentau.566C5810

5650D555                              |.  CC            int3

5650D556                              |.  CC            int3

5650D557                              |.  CC            int3

5650D558                              |.  CC            int3

5650D559                              |.  CC            int3

5650D55A                              |.  CC            int3

5650D55B                              |.  CC            int3

5650D55C                              |.  CC            int3

5650D55D                              |.  CC            int3

5650D55E                              |.  CC            int3

5650D55F                              |.  CC            int3

5650D560                              |.  68 A0C5B656   push CXCentau.56B6C5A0

5650D565                              |.  68 44FBA356   push CXCentau.56A3FB44                                                            ;  ASCII "system.ext"

5650D56A                              |.  E8 916A1B00   call CXCentau.566C4000

5650D56F                              |.  83C4 08       add esp, 8

5650D572                              |.  B8 01000000   mov eax, 1

5650D577                              \.  C3            retn

55CDCBFF   .  8B7424 10     mov esi, dword ptr [esp+10]

55CDCC03   .  FF44B1 04     inc dword ptr [ecx+esi\*4+4]

55CDCC07   .  FF15 D0E2E055 call dword ptr [<&MSVCR110.clock>]       ;  msvcr110.clock

55CDCC0D   .  8B4C24 08     mov ecx, dword ptr [esp+8]

55CDCC11   .  2B04B1        sub eax, dword ptr [ecx+esi\*4]

55CDCC14   .  66:0F6EC0     movd mm0, eax

55CDCC18   .  F3:           prefix rep:

55CDCC19      0F            db 0F

55CDCC1A   .  E6 C0         out 0C0, al

55CDCC1C   .  8B4424 0C     mov eax, dword ptr [esp+C]

55CDCC20   .  F2:           prefix repne:

55CDCC21   .  0F5E05 D89DE3>divps xmm0, dqword ptr [55E39DD8]

55CDCC28   .  66:0F2F40 68  comiss xmm0, dword ptr [eax+68]

55CDCC2D   .  77 41         ja short CEGUIBas.55CDCC70

55CDCC2F   .  F3:0F1044B1 1>movss xmm0, dword ptr [ecx+esi\*4+10]

55CDCC35   .  F3:0F104C24 4>movss xmm1, dword ptr [esp+40]

55CDCC3B   .  0F2FC1        comiss xmm0, xmm1

55CDCC3E   .  77 30         ja short CEGUIBas.55CDCC70

55CDCC40   .  0F2F4CB1 14   comiss xmm1, dword ptr [ecx+esi\*4+14]

55CDCC45   .  77 29         ja short CEGUIBas.55CDCC70

55CDCC47   .  F3:0F1044B1 0>movss xmm0, dword ptr [ecx+esi\*4+8]

55CDCC4D   .  F3:0F104C24 4>movss xmm1, dword ptr [esp+44]

55CDCC53   .  0F2FC1        comiss xmm0, xmm1

55CDCC56   .  77 18         ja short CEGUIBas.55CDCC70

55CDCC58   .  0F2F4CB1 0C   comiss xmm1, dword ptr [ecx+esi\*4+C]

55CDCC5D   .  77 11         ja short CEGUIBas.55CDCC70

55CDCC5F   .  397CB1 18     cmp dword ptr [ecx+esi\*4+18], edi

55CDCC63   .  75 0B         jnz short CEGUIBas.55CDCC70

55CDCC65   .  837CB1 04 03  cmp dword ptr [ecx+esi\*4+4], 3

55CDCC6A   .  0F8E DB000000 jle CEGUIBas.55CDCD4B

55CDCC70   >  C744B1 04 010>mov dword ptr [ecx+esi\*4+4], 1

55CDCC78   .  F3:0F104424 4>movss xmm0, dword ptr [esp+40]

55CDCC7E   .  F3:0F104CB1 1>movss xmm1, dword ptr [ecx+esi\*4+14]

55CDCC84   .  F3:0F1054B1 0>movss xmm2, dword ptr [ecx+esi\*4+C]

55CDCC8A   .  F3:0F5C4CB1 1>subss xmm1, dword ptr [ecx+esi\*4+10]

55CDCC90   .  F3:0F5C54B1 0>subss xmm2, dword ptr [ecx+esi\*4+8]
```
