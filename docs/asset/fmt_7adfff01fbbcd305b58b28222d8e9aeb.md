
```
//函数 00493B60(GenLv2_NumStr_RegCodeCI)
00493B60   /$  6A FF         push    -1
00493B62   |.  68 679E5800   push    00589E67                      ;  SE 处理程序安装
00493B67   |.  64:A1 000000>mov     eax, dword ptr fs:[0]
;;;;;;;;;;;;;;;;;;;;;;;;省略部分;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
00493BDF   |.  8B7C24 58     mov     edi, dword ptr [esp+58]
00493BE3   |>  8B4424 54     /mov     eax, dword ptr [esp+54]         ;  JASS: 由字串"xxxx"生成一整数
00493BE7   |.  0FAFD5        |imul    edx, ebp
00493BEA   |.  DB4424 18     |fild    dword ptr [esp+18]
00493BEE   |.  DD5C24 18     |fstp    qword ptr [esp+18]
00493BF2   |.  8A5C30 FF     |mov     bl, byte ptr [eax+esi-1]
00493BF6   |.  0FAF5424 10   |imul    edx, dword ptr [esp+10]
00493BFB   |.  0FBECB        |movsx   ecx, bl
00493BFE   |.  0FAFD7        |imul    edx, edi
00493C01   |.  894C24 58     |mov     dword ptr [esp+58], ecx
00493C05   |.  DB4424 58     |fild    dword ptr [esp+58]
00493C09   |.  0FBEC3        |movsx   eax, bl
00493C0C   |.  D9FA          |fsqrt
00493C0E   |.  0FAFC6        |imul    eax, esi
00493C11   |.  DC4C24 18     |fmul    qword ptr [esp+18]
00493C15   |.  DC05 B89A5900 |fadd    qword ptr [599AB8]
00493C1B   |.  0FAFC6        |imul    eax, esi
00493C1E   |.  894424 58     |mov     dword ptr [esp+58], eax
00493C22   |.  DB4424 58     |fild    dword ptr [esp+58]
00493C26   |.  895424 58     |mov     dword ptr [esp+58], edx
00493C2A   |.  DEC9          |fmulp   st(1), st
00493C2C   |.  DB4424 58     |fild    dword ptr [esp+58]
00493C30   |.  DEC1          |faddp   st(1), st
00493C32   |.  E8 75CD0B00   |call    005509AC                      ;  JASS: _ftol 浮点转长整数:arg ST0--return eax
00493C37   |.  99            |cdq
00493C38   |.  B9 A0860100   |mov     ecx, 186A0
00493C3D   |.  F7F9          |idiv    ecx
00493C3F   |.  895424 10     |mov     dword ptr [esp+10], edx       ;  JASS-DEBUG: edx - IterValA
00493C43   |.  DB4424 10     |fild    dword ptr [esp+10]
00493C47   |.  0FBED3        |movsx   edx, bl
00493C4A   |.  DD5C24 28     |fstp    qword ptr [esp+28]
00493C4E   |.  895424 58     |mov     dword ptr [esp+58], edx
00493C52   |.  DB4424 58     |fild    dword ptr [esp+58]
00493C56   |.  DD05 F8BA5900 |fld     qword ptr [59BAF8]
00493C5C   |.  E8 3FE20B00   |call    00551EA0                      ;  JASS: _CIpow
00493C61   |.  DC4C24 18     |fmul    qword ptr [esp+18]
00493C65   |.  E8 42CD0B00   |call    005509AC
00493C6A   |.  DD4424 28     |fld     qword ptr [esp+28]
00493C6E   |.  D9FE          |fsin
00493C70   |.  8BD8          |mov     ebx, eax
00493C72   |.  E8 35CD0B00   |call    005509AC
00493C77   |.  DD4424 28     |fld     qword ptr [esp+28]
00493C7B   |.  03D8          |add     ebx, eax
00493C7D   |.  8BC6          |mov     eax, esi
00493C7F   |.  0FAFC5        |imul    eax, ebp
00493C82   |.  D9FA          |fsqrt
00493C84   |.  B9 A0860100   |mov     ecx, 186A0
00493C89   |.  0FAFC7        |imul    eax, edi
00493C8C   |.  03C3          |add     eax, ebx
00493C8E   |.  99            |cdq
00493C8F   |.  F7F9          |idiv    ecx
00493C91   |.  8BEA          |mov     ebp, edx                      ;  JASS-DEBUG: ebp - IterValB
00493C93   |.  E8 14CD0B00   |call    005509AC
00493C98   |.  8BD5          |mov     edx, ebp
00493C9A   |.  8B4C24 10     |mov     ecx, dword ptr [esp+10]
00493C9E   |.  0FAFD5        |imul    edx, ebp
00493CA1   |.  03D1          |add     edx, ecx
00493CA3   |.  B9 A0860100   |mov     ecx, 186A0
00493CA8   |.  0FAFD6        |imul    edx, esi
00493CAB   |.  0FAFD6        |imul    edx, esi
00493CAE   |.  0FAFD7        |imul    edx, edi
00493CB1   |.  03C2          |add     eax, edx
00493CB3   |.  99            |cdq
00493CB4   |.  F7F9          |idiv    ecx                           ;  JASS-DEBUG: edx - IterValC (除法之后)
00493CB6   |.  8B4424 20     |mov     eax, dword ptr [esp+20]
00493CBA   |.  46            |inc     esi
00493CBB   |.  3BF0          |cmp     esi, eax
00493CBD   |.  897424 18     |mov     dword ptr [esp+18], esi
00493CC1   |.^ 0F8E 1CFFFFFF \jle     00493BE3
00493CC7   |.  8B5C24 10     mov     ebx, dword ptr [esp+10]        ;  JASS: 生成的整数：dword ptr [esp+10],ebp,edx
00493CCB   |>  33C0          xor     eax, eax
00493CCD   |>  8BC8          /mov     ecx, eax
00493CCF   |.  0FAFC8        |imul    ecx, eax
00493CD2   |.  0FAFC8        |imul    ecx, eax
00493CD5   |.  8D4C19 1F     |lea     ecx, dword ptr [ecx+ebx+1F]
00493CD9   |.  81E1 7F000080 |and     ecx, 8000007F
00493CDF   |.  79 05         |jns     short 00493CE6
00493CE1   |.  49            |dec     ecx
00493CE2   |.  83C9 80       |or      ecx, FFFFFF80
00493CE5   |.  41            |inc     ecx
00493CE6   |>  884C04 30     |mov     byte ptr [esp+eax+30], cl
00493CEA   |.  40            |inc     eax
00493CEB   |.  83F8 05       |cmp     eax, 5
00493CEE   |.^ 7C DD         \jl      short 00493CCD
00493CF0   |.  B8 05000000   mov     eax, 5
00493CF5   |>  8BC8          /mov     ecx, eax
00493CF7   |.  0FAFC8        |imul    ecx, eax
00493CFA   |.  0FAFC8        |imul    ecx, eax
00493CFD   |.  8D4C29 1F     |lea     ecx, dword ptr [ecx+ebp+1F]
00493D01   |.  81E1 7F000080 |and     ecx, 8000007F
00493D07   |.  79 05         |jns     short 00493D0E
00493D09   |.  49            |dec     ecx
00493D0A   |.  83C9 80       |or      ecx, FFFFFF80
00493D0D   |.  41            |inc     ecx
00493D0E   |>  884C04 30     |mov     byte ptr [esp+eax+30], cl
00493D12   |.  40            |inc     eax
00493D13   |.  83F8 0A       |cmp     eax, 0A
00493D16   |.^ 7C DD         \jl      short 00493CF5
00493D18   |.  B8 0A000000   mov     eax, 0A
00493D1D   |>  8BC8          /mov     ecx, eax
00493D1F   |.  0FAFC8        |imul    ecx, eax
00493D22   |.  0FAFC8        |imul    ecx, eax
00493D25   |.  8D4C11 1F     |lea     ecx, dword ptr [ecx+edx+1F]
00493D29   |.  81E1 7F000080 |and     ecx, 8000007F
00493D2F   |.  79 05         |jns     short 00493D36
00493D31   |.  49            |dec     ecx
00493D32   |.  83C9 80       |or      ecx, FFFFFF80
00493D35   |.  41            |inc     ecx
00493D36   |>  884C04 30     |mov     byte ptr [esp+eax+30], cl
00493D3A   |.  40            |inc     eax
00493D3B   |.  83F8 0F       |cmp     eax, 0F
00493D3E   |.^ 7C DD         \jl      short 00493D1D
00493D40   |.  33D2          xor     edx, edx
00493D42   |.  C64424 3F 00  mov     byte ptr [esp+3F], 0
00493D47   |.  33C9          xor     ecx, ecx
00493D49   |>  8A4414 30     /mov     al, byte ptr [esp+edx+30]
00493D4D   |.  3C 30         |cmp     al, 30
00493D4F   |.  7C 04         |jl      short 00493D55
00493D51   |.  3C 39         |cmp     al, 39
00493D53   |.  7E 29         |jle     short 00493D7E
00493D55   |>  3C 41         |cmp     al, 41
00493D57   |.  7C 04         |jl      short 00493D5D
00493D59   |.  3C 5A         |cmp     al, 5A
00493D5B   |.  7E 21         |jle     short 00493D7E
00493D5D   |>  3C 61         |cmp     al, 61
00493D5F   |.  7C 04         |jl      short 00493D65
00493D61   |.  3C 7A         |cmp     al, 7A
00493D63   |.  7E 19         |jle     short 00493D7E
00493D65   |>  0FBEC0        |movsx   eax, al
00493D68   |.  8D4408 1F     |lea     eax, dword ptr [eax+ecx+1F]
00493D6C   |.  25 7F000080   |and     eax, 8000007F
00493D71   |.  79 05         |jns     short 00493D78
00493D73   |.  48            |dec     eax
00493D74   |.  83C8 80       |or      eax, FFFFFF80
00493D77   |.  40            |inc     eax
00493D78   |>  884414 30     |mov     byte ptr [esp+edx+30], al
00493D7C   |.^ EB CB         |jmp     short 00493D49
00493D7E   |>  83C1 07       |add     ecx, 7
00493D81   |.  42            |inc     edx
00493D82   |.  83F9 69       |cmp     ecx, 69
00493D85   |.^ 7C C2         \jl      short 00493D49
00493D87   |.  8D4C24 30     lea     ecx, dword ptr [esp+30]
00493D8B   |.  8D5424 14     lea     edx, dword ptr [esp+14]
;;;;;;;;;;;;;;;;;;;;;;;;省略部分;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
00493DE6   \.  C3            retn
```
