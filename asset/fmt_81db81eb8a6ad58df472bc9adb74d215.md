
```
004AF962    E8 6552F5FF     call    00404BCC
004AF967    83F8 03         cmp     eax, 3
004AF96A    7D 47          jge     short 004AF9B3
004AF96C    68 18FC4A00    push    004AFC18                  ; name must have no less than three symbols!
004AF971    8D45 F0        lea     eax, dword ptr [ebp-10]
004AF974    50             push    eax
004AF975    A1 2CDA4E00    mov     eax, dword ptr [4EDA2C]
004AF97A    8B00           mov     eax, dword ptr [eax]
004AF97C    B9 4CFC4A00    mov     ecx, 004AFC4C               ; message9
004AF981    BA 60FC4A00    mov     edx, 004AFC60               ; evaluation
004AF986    8B18           mov     ebx, dword ptr [eax]
004AF988    FF13           call    dword ptr [ebx]
004AF98A    8B55 F0        mov     edx, dword ptr [ebp-10]
004AF98D    A1 04DB4E00    mov     eax, dword ptr [4EDB04]
004AF992    8B00           mov     eax, dword ptr [eax]
004AF994    8B80 04030000  mov     eax, dword ptr [eax+304]
004AF99A    E8 3D2BFBFF    call    004624DC
004AF99F    A1 04DB4E00    mov     eax, dword ptr [4EDB04]
004AF9A4    8B00           mov     eax, dword ptr [eax]
004AF9A6    8B10           mov     edx, dword ptr [eax]
004AF9A8    FF92 EC000000  call    dword ptr [edx+EC]
004AF9AE    E9 F0010000    jmp     004AFBA3
004AF9B3    8D55 EC        lea     edx, dword ptr [ebp-14]
004AF9B6    8B83 20030000  mov     eax, dword ptr [ebx+320]
004AF9BC    E8 EB2AFBFF    call    004624AC
004AF9C1    8B45 EC        mov     eax, dword ptr [ebp-14]
004AF9C4    E8 0352F5FF    call    00404BCC
004AF9C9    85C0           test    eax, eax
004AF9CB    75 47          jnz     short 004AFA14
004AF9CD    68 74FC4A00    push    004AFC74                  ; fields for license key must not be less three symbols!
004AF9D2    8D45 E8        lea     eax, dword ptr [ebp-18]
004AF9D5    50             push    eax
004AF9D6    A1 2CDA4E00    mov     eax, dword ptr [4EDA2C]
004AF9DB    8B00           mov     eax, dword ptr [eax]
004AF9DD    B9 B4FC4A00    mov     ecx, 004AFCB4               ; message10
004AF9E2    BA 60FC4A00    mov     edx, 004AFC60               ; evaluation
004AF9E7    8B18           mov     ebx, dword ptr [eax]
004AF9E9    FF13           call    dword ptr [ebx]
004AF9EB    8B55 E8        mov     edx, dword ptr [ebp-18]
004AF9EE    A1 04DB4E00    mov     eax, dword ptr [4EDB04]
004AF9F3    8B00           mov     eax, dword ptr [eax]
004AF9F5    8B80 04030000  mov     eax, dword ptr [eax+304]
004AF9FB    E8 DC2AFBFF    call    004624DC
004AFA00    A1 04DB4E00    mov     eax, dword ptr [4EDB04]
004AFA05    8B00           mov     eax, dword ptr [eax]
004AFA07    8B10           mov     edx, dword ptr [eax]
004AFA09    FF92 EC000000  call    dword ptr [edx+EC]
004AFA0F    E9 8F010000    jmp     004AFBA3
004AFA14    8D55 E0        lea     edx, dword ptr [ebp-20]
004AFA17    8B83 1C030000  mov     eax, dword ptr [ebx+31C]
004AFA1D    E8 8A2AFBFF    call    004624AC
004AFA22    8B45 E0        mov     eax, dword ptr [ebp-20]
004AFA25    8D55 E4        lea     edx, dword ptr [ebp-1C]
004AFA28    E8 AF9BF5FF    call    004095DC
004AFA2D    8B55 E4        mov     edx, dword ptr [ebp-1C]
004AFA30    A1 00D74E00    mov     eax, dword ptr [4ED700]
004AFA35    E8 164FF5FF    call    00404950
004AFA3A    8D55 D8        lea     edx, dword ptr [ebp-28]
004AFA3D    8B83 20030000  mov     eax, dword ptr [ebx+320]
004AFA43    E8 642AFBFF    call    004624AC
004AFA48    8B45 D8        mov     eax, dword ptr [ebp-28]
004AFA4B    8D55 DC        lea     edx, dword ptr [ebp-24]
004AFA4E    E8 899BF5FF    call    004095DC
004AFA53    8B55 DC        mov     edx, dword ptr [ebp-24]
004AFA56    A1 4CD54E00    mov     eax, dword ptr [4ED54C]
004AFA5B    E8 F04EF5FF    call    00404950
004AFA60    A1 00D74E00    mov     eax, dword ptr [4ED700]
004AFA65    8B00           mov     eax, dword ptr [eax]
004AFA67    50             push    eax
004AFA68    B9 C8FC4A00    mov     ecx, 004AFCC8               ; licuser
004AFA6D    BA D8FC4A00    mov     edx, 004AFCD8               ; 

software\microsoft\windows\currentversion\settings\smart pc suite
004AFA72    B8 01000080    mov     eax, 80000001
004AFA77    E8 BC46FFFF    call    004A4138
004AFA7C    A1 4CD54E00    mov     eax, dword ptr [4ED54C]
004AFA81    8B00           mov     eax, dword ptr [eax]
004AFA83    50             push    eax
004AFA84    BA D8FC4A00    mov     edx, 004AFCD8               ; 

software\microsoft\windows\currentversion\settings\smart pc suite
004AFA89    B9 24FD4A00    mov     ecx, 004AFD24               ; lickey
004AFA8E    B8 01000080    mov     eax, 80000001
004AFA93    E8 A046FFFF    call    004A4138
004AFA98    E8 FB6BFFFF    call    004A6698                  ; 绝对是关键 call 啊!!!!!
004AFA9D    8B15 CCD94E00  mov     edx, dword ptr [4ED9CC]       
004AFAA3    8802           mov     byte ptr [edx], al
004AFAA5    A1 CCD94E00    mov     eax, dword ptr [4ED9CC]
004AFAAA    8038 00        cmp     byte ptr [eax], 0
004AFAAD    74 75          je      short 004AFB24             ; 关键跳!爆破点
004AFAAF    E8 54C3F5FF    call    0040BE08
004AFAB4    83C4 F8        add     esp, -8
004AFAB7    DD1C24         fstp    qword ptr [esp]
004AFABA    9B             wait
004AFABB    BA D8FC4A00    mov     edx, 004AFCD8               ; 

software\microsoft\windows\currentversion\settings\smart pc suite
004AFAC0    B9 34FD4A00    mov     ecx, 004AFD34               ; timefactor1
004AFAC5    B8 01000080    mov     eax, 80000001
004AFACA    E8 4D48FFFF    call    004A431C
004AFACF    E8 34C3F5FF    call    0040BE08
004AFAD4    83C4 F8        add     esp, -8
004AFAD7    DD1C24         fstp    qword ptr [esp]
004AFADA    9B             wait
004AFADB    BA D8FC4A00    mov     edx, 004AFCD8               ; 

software\microsoft\windows\currentversion\settings\smart pc suite
004AFAE0    B9 48FD4A00    mov     ecx, 004AFD48               ; timefactor2
004AFAE5    B8 01000080    mov     eax, 80000001
004AFAEA    E8 2D48FFFF    call    004A431C
004AFAEF    68 5CFD4A00    push    004AFD5C                  ; thank you for registering smart pc suite!
004AFAF4    8D45 D4        lea     eax, dword ptr [ebp-2C]
004AFAF7    50             push    eax
004AFAF8    A1 2CDA4E00    mov     eax, dword ptr [4EDA2C]
004AFAFD    8B00           mov     eax, dword ptr [eax]
004AFAFF    B9 90FD4A00    mov     ecx, 004AFD90               ; message11
004AFB04    BA 60FC4A00    mov     edx, 004AFC60               ; evaluation
004AFB09    8B30           mov     esi, dword ptr [eax]
004AFB0B    FF16           call    dword ptr [esi]
004AFB0D    8B55 D4        mov     edx, dword ptr [ebp-2C]
004AFB10    A1 04DB4E00    mov     eax, dword ptr [4EDB04]
004AFB15    8B00           mov     eax, dword ptr [eax]
004AFB17    8B80 04030000  mov     eax, dword ptr [eax+304]
004AFB1D    E8 BA29FBFF    call    004624DC
004AFB22    EB 69          jmp     short 004AFB8D
004AFB24    68 A4FD4A00    push    004AFDA4                  ; the registration info is incorrect!
004AFB29    8D45 CC        lea     eax, dword ptr [ebp-34]
004AFB2C    50             push    eax
```

毫无疑问，关键跳和关键 `call` 精确定位，进入 `004AFA98` 的 `call` 分析算法。

代码:
