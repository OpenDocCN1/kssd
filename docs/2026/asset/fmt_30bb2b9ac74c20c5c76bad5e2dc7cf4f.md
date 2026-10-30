
```
00407429   .  E8 46C20000   call    <jmp.&MFC42.#CWnd::GetWindowTextA_3874>   ;  在GetWindowTextA上下断点，在聊天输入框随便输入点东西，来到这里

0040742E   .  8D4C24 10     lea     ecx, dword ptr [esp+10]

00407432   .  E8 97C20000   call    <jmp.&MFC42.#CString::TrimRight_6283>

00407437   .  8B4C24 10     mov     ecx, dword ptr [esp+10]

0040743B   .  8B41 F8       mov     eax, dword ptr [ecx-8]

0040743E   .  85C0          test    eax, eax

00407440   .  75 3A         jnz     short 0040747C

00407442   .  8BAD D4000000 mov     ebp, dword ptr [ebp+D4]

00407448   .  85ED          test    ebp, ebp

0040744A   .  0F84 58060000 je      00407AA8

00407450   .  8B4D 48       mov     ecx, dword ptr [ebp+48]

00407453   .  85C9          test    ecx, ecx

00407455   .  0F84 4D060000 je      00407AA8

0040745B   .  8B11          mov     edx, dword ptr [ecx]

0040745D   .  FF52 1C       call    dword ptr [edx+1C]

00407460   .  8B10          mov     edx, dword ptr [eax]

00407462   .  68 B8EC4100   push    0041ECB8

00407467   .  8BC8          mov     ecx, eax

00407469   .  FF52 18       call    dword ptr [edx+18]

0040746C   .  C78424 3C0700>mov     dword ptr [esp+73C], -1

00407477   .  E9 99070000   jmp     00407C15

0040747C   >  3D 00010000   cmp     eax, 100                                  ;  限制发送的内容不能超过0x100字节

00407481   .  7E 5F         jle     short 004074E2

00407483   .  8D4424 24     lea     eax, dword ptr [esp+24]

00407487   .  68 FA000000   push    0FA                                       ;  把聊天内容前0xfa以外的数据截掉

0040748C   .  50            push    eax

0040748D   .  8D4C24 18     lea     ecx, dword ptr [esp+18]

00407491   .  E8 8AC10000   call    <jmp.&MFC42.#CString::Left_4129>

00407496   .  68 2CF44100   push    0041F42C

0040749B   .  8D4C24 24     lea     ecx, dword ptr [esp+24]

0040749F   .  50            push    eax

004074A0   .  51            push    ecx

004074A1   .  C68424 480700>mov     byte ptr [esp+748], 1

004074A9   .  E8 D0C00000   call    <jmp.&MFC42.#operator+_924>

004074AE   .  50            push    eax

004074AF   .  8D4C24 14     lea     ecx, dword ptr [esp+14]

004074B3   .  C68424 400700>mov     byte ptr [esp+740], 2

004074BB   .  E8 5AC10000   call    <jmp.&MFC42.#CString::operator=_858>

004074C0   .  8D4C24 20     lea     ecx, dword ptr [esp+20]

004074C4   .  C68424 3C0700>mov     byte ptr [esp+73C], 1

004074CC   .  E8 8DBF0000   call    <jmp.&MFC42.#CString::~CString_800>

004074D1   .  8D4C24 24     lea     ecx, dword ptr [esp+24]

004074D5   .  C68424 3C0700>mov     byte ptr [esp+73C], 0

004074DD   .  E8 7CBF0000   call    <jmp.&MFC42.#CString::~CString_800>

004074E2   >  8D4C24 14     lea     ecx, dword ptr [esp+14]

004074E6   .  E8 85BF0000   call    <jmp.&MFC42.#CString::CString_540>

004074EB   .  8D5424 14     lea     edx, dword ptr [esp+14]

004074EF   .  8D8D 380B0000 lea     ecx, dword ptr [ebp+B38]

004074F5   .  52            push    edx

004074F6   .  C68424 400700>mov     byte ptr [esp+740], 3

004074FE   .  E8 71C10000   call    <jmp.&MFC42.#CWnd::GetWindowTextA_3874>   ;  获取消息发送对象，例如是“所有人”

00407503   .  8B4424 14     mov     eax, dword ptr [esp+14]

00407507   .  8B48 F8       mov     ecx, dword ptr [eax-8]

0040750A   .  85C9          test    ecx, ecx

0040750C   .  0F84 A6050000 je      00407AB8

00407512   .  68 F0E34100   push    0041E3F0                                  ; /s2 = "所有人"

00407517   .  50            push    eax                                       ; |s1

00407518   .  FF15 44764100 call    dword ptr [<&MSVCRT._mbscmp>]             ; \_mbscmp

0040751E   .  83C4 08       add     esp, 8

00407521   .  85C0          test    eax, eax

00407523   .  0F84 8F050000 je      00407AB8

…………省略一段代码…………

00407AB8   >  8D4C24 18     lea     ecx, dword ptr [esp+18]

00407ABC   .  E8 AFB90000   call    <jmp.&MFC42.#CString::CString_540>

00407AC1   .  A1 64F54100   mov     eax, dword ptr [41F564]

00407AC6   .  B9 44F54100   mov     ecx, 0041F544

00407ACB   .  50            push    eax

00407ACC   .  C68424 400700>mov     byte ptr [esp+740], 4

00407AD4   .  E8 97660000   call    0040E170

00407AD9   .  85C0          test    eax, eax

00407ADB   .  74 16         je      short 00407AF3

00407ADD   .  83C0 0A       add     eax, 0A

00407AE0   .  8D4C24 18     lea     ecx, dword ptr [esp+18]

00407AE4   .  50            push    eax

00407AE5   .  68 B0EC4100   push    0041ECB0                                  ;  ASCII "%s"

00407AEA   .  51            push    ecx

00407AEB   .  E8 74B90000   call    <jmp.&MFC42.#CString::Format_2818>

00407AF0   .  83C4 0C       add     esp, 0C

00407AF3   >  8D5424 18     lea     edx, dword ptr [esp+18]

00407AF7   .  68 40EC4100   push    0041EC40                                  ;  ASCII "-->所有人:"

00407AFC   .  8D4424 3C     lea     eax, dword ptr [esp+3C]

00407B00   .  52            push    edx

00407B01   .  50            push    eax

00407B02   .  E8 77BA0000   call    <jmp.&MFC42.#operator+_924>

00407B07   .  8D4C24 10     lea     ecx, dword ptr [esp+10]

00407B0B   .  8D5424 30     lea     edx, dword ptr [esp+30]

00407B0F   .  51            push    ecx

00407B10   .  B3 05         mov     bl, 5

00407B12   .  50            push    eax

00407B13   .  52            push    edx

00407B14   .  889C24 480700>mov     byte ptr [esp+748], bl

00407B1B   .  E8 90BB0000   call    <jmp.&MFC42.#operator+_922>

00407B20   .  50            push    eax

00407B21   .  8D4C24 1C     lea     ecx, dword ptr [esp+1C]

00407B25   .  C68424 400700>mov     byte ptr [esp+740], 6

00407B2D   .  E8 E8BA0000   call    <jmp.&MFC42.#CString::operator=_858>

00407B32   .  8D4C24 30     lea     ecx, dword ptr [esp+30]

00407B36   .  889C24 3C0700>mov     byte ptr [esp+73C], bl

00407B3D   .  E8 1CB90000   call    <jmp.&MFC42.#CString::~CString_800>

00407B42   .  8D4C24 38     lea     ecx, dword ptr [esp+38]

00407B46   .  C68424 3C0700>mov     byte ptr [esp+73C], 4

00407B4E   .  E8 0BB90000   call    <jmp.&MFC42.#CString::~CString_800>

00407B53   .  8D4C24 18     lea     ecx, dword ptr [esp+18]

00407B57   .  E8 72BB0000   call    <jmp.&MFC42.#CString::TrimRight_6283>

00407B5C   .  68 28F44100   push    0041F428

00407B61   .  8BCF          mov     ecx, edi

00407B63   .  E8 20B90000   call    <jmp.&MFC42.#CWnd::SetWindowTextA_6199>   ;  这里是自己说的话显示到自己的窗口上

00407B68   .  8B5424 18     mov     edx, dword ptr [esp+18]

00407B6C   .  B9 DF000000   mov     ecx, 0DF

00407B71   .  33C0          xor     eax, eax

00407B73   .  8D7C24 3C     lea     edi, dword ptr [esp+3C]

00407B77   .  F3:AB         rep     stos dword ptr es:[edi]

00407B79   .  83CB FF       or      ebx, FFFFFFFF

00407B7C   .  8BFA          mov     edi, edx

00407B7E   .  8BCB          mov     ecx, ebx

00407B80   .  C64424 3C 27  mov     byte ptr [esp+3C], 27

00407B85   .  895C24 40     mov     dword ptr [esp+40], ebx

00407B89   .  8D7424 48     lea     esi, dword ptr [esp+48]

00407B8D   .  F2:AE         repne   scas byte ptr es:[edi]

00407B8F   .  F7D1          not     ecx

00407B91   .  2BF9          sub     edi, ecx

00407B93   .  897424 30     mov     dword ptr [esp+30], esi

00407B97   .  8BC1          mov     eax, ecx

00407B99   .  8BF7          mov     esi, edi

00407B9B   .  8B7C24 30     mov     edi, dword ptr [esp+30]

00407B9F   .  C1E9 02       shr     ecx, 2

00407BA2   .  F3:A5         rep     movs dword ptr es:[edi], dword ptr [esi]

00407BA4   .  8BC8          mov     ecx, eax

00407BA6   .  A1 0CF54100   mov     eax, dword ptr [41F50C]

00407BAB   .  83E1 03       and     ecx, 3

00407BAE   .  F3:A4         rep     movs byte ptr es:[edi], byte ptr [esi]

00407BB0   .  8B52 F8       mov     edx, dword ptr [edx-8]

00407BB3   .  895424 44     mov     dword ptr [esp+44], edx

00407BB7   .  8B08          mov     ecx, dword ptr [eax]

00407BB9   .  8D5424 3C     lea     edx, dword ptr [esp+3C]

00407BBD   .  52            push    edx

00407BBE   .  68 7C030000   push    37C

00407BC3   .  50            push    eax

00407BC4   .  FF51 0C       call    dword ptr [ecx+C]                         ;  这里准备要跟游戏大厅通讯
```
