
### 汇编代码分析

#### 主函数逻辑片段

```asm
.text:0042BCF2  74  59            jz      short loc_42BD4D

.text:0042BCF4  FF B7 D5 00 00 00 push    dword ptr [edi+0D5h]

.text:0042BCFA  E8 BA 09 00 00    call    sub_42C6B9

.text:0042BCFF  3B F0             cmp     esi, eax

.text:0042BD01  59                pop     ecx

.text:0042BD02  75 0F             jnz     short loc_42BD13

.text:0042BD04  53                push    ebx

.text:0042BD05  53                push    ebx

.text:0042BD06  FF 35 44 7B 48 00 push    dword_487B44

.text:0042BD0C  E8 D1 54 02 00    call    sub_4511E2

.text:0042BD11  EB 75             jmp     short loc_42BD88

.text:0042BD13             ; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

.text:0042BD13

.text:0042BD13             loc_42BD13:             ; CODE XREF: sub_42BCB2+50j

.text:0042BD13  8B BF D5 00 00 00 mov     edi, [edi+0D5h]

.text:0042BD19  39 5F F8          cmp     [edi-8], ebx

.text:0042BD1C  75 0A             jnz     short loc_42BD28

.text:0042BD1E  53                push    ebx

.text:0042BD1F  53                push    ebx

.text:0042BD20  FF 35 BC 7B 48 00 push    dword_487BBC

.text:0042BD26  EB 1B             jmp     short loc_42BD43

.text:0042BD28             ; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

.text:0042BD28

.text:0042BD28             loc_42BD28:             ; CODE XREF: sub_42BCB2+6Aj

.text:0042BD28  57                push    edi

.text:0042BD29  E8 8B 09 00 00    call    sub_42C6B9

.text:0042BD2E  59                pop     ecx

.text:0042BD2F  85 C0             test    eax, eax

.text:0042BD31  53                push    ebx

.text:0042BD32  53                push    ebx

.text:0042BD33  75 08             jnz     short loc_42BD3D

.text:0042BD35  FF 35 C0 7B 48 00 push    dword_487BC0

.text:0042BD3B  EB 06             jmp     short loc_42BD43

.text:0042BD3D             ; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

.text:0042BD3D

.text:0042BD3D             loc_42BD3D:             ; CODE XREF: sub_42BCB2+81j

.text:0042BD3D  FF 35 C4 7B 48 00 push    dword_487BC4

.text:0042BD43

.text:0042BD43             loc_42BD43:             ; CODE XREF: sub_42BCB2+74j

.text:0042BD43                                             ; sub_42BCB2+89j

.text:0042BD43  E8 9A 54 02 00    call    sub_4511E2

.text:0042BD48  E9 D8 00 00 00    jmp     loc_42BE25

.text:0042BD4D             ; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

.text:0042BD4D

.text:0042BD4D             loc_42BD4D:             ; CODE XREF: sub_42BCB2+40j

.text:0042BD4D  8B 87 D9 00 00 00 mov     eax, [edi+0D9h]

.text:0042BD53  BE 34 72 48 00    mov     esi, offset aForEvaluationO ; "For Evaluation Only"

.text:0042BD58  56                push    esi

.text:0042BD59  50                push    eax

.text:0042BD5A  E8 ED 67 00 00    call    __mbscmp

.text:0042BD5F  59                pop     ecx

.text:0042BD60  85 C0             test    eax, eax

.text:0042BD62  59                pop     ecx

.text:0042BD63  0F 85 B5 00 00 00 jnz     loc_42BE1E

.text:0042BD69  53                push    ebx

.text:0042BD6A  53                push    ebx

.text:0042BD6B  FF 35 C8 7B 48 00 push    dword_487BC8

.text:0042BD71  E8 6C 54 02 00    call    sub_4511E2

.text:0042BD76  8B 0D A4 13 49 00 mov     ecx, dword_4913A4

.text:0042BD7C  56                push    esi

.text:0042BD7D  68 48 72 48 00    push    offset aTempcode      ; "Tempcode"

.text:0042BD82  55                push    ebp

.text:0042BD83  E8 08 56 02 00    call    sub_451390

.text:0042BD88

.text:0042BD88             loc_42BD88:             ; CODE XREF: sub_42BCB2+5Fj

.text:0042BD88  A1 A4 13 49 00    mov     eax, dword_4913A4

.text:0042BD8D  88 98 CD 04 00 00 mov     [eax+4CDh], bl

.text:0042BD93  A1 A4 13 49 00    mov     eax, dword_4913A4

.text:0042BD98  88 98 CF 04 00 00 mov     [eax+4CFh], bl

.text:0042BD9E  A1 A0 13 49 00    mov     eax, dword_4913A0

.text:0042BDA3  3B C3             cmp     eax, ebx

.text:0042BDA5  74 06             jz      short loc_42BDAD

.text:0042BDA7  88 98 E4 0F 00 00 mov     [eax+0FE4h], bl

.text:0042BDAD

.text:0042BDAD             loc_42BDAD:             ; CODE XREF: sub_42BCB2+F3j

.text:0042BDAD  FF B7 D5 00 00 00 push    dword ptr [edi+0D5h]

.text:0042BDB3  8D B7 D5 00 00 00 lea     esi, [edi+0D5h]

.text:0042BDB9  68 5C 71 48 00    push    offset aName_1        ; "Name"

.text:0042BDBE  68 B0 72 48 00    push    offset aSoftwareTennys ; "SOFTWARE\\Tennyson Maxwell\\Teleport Pro"

.text:0042BDC3  68 02 00 00 80    push    80000002h

.text:0042BDC8  E8 18 0F 00 00    call    sub_42CCE5

.text:0042BDCD  FF 74 24 20       push    [esp+24h+var_4]

.text:0042BDD1  BB A0 72 48 00    mov     ebx, offset aRegistration ; "Registration"

.text:0042BDD6  53                push    ebx

.text:0042BDD7  68 B0 72 48 00    push    offset aSoftwareTennys ; "SOFTWARE\\Tennyson Maxwell\\Teleport Pro"

.text:0042BDDC  68 02 00 00 80    push    80000002h

.text:0042BDE1  E8 63 0F 00 00    call    sub_42CD49

.text:0042BDE6  8B 0D A4 13 49 00 mov     ecx, dword_4913A4

.text:0042BDEC  83 C4 20          add     esp, 20h

.text:0042BDEF  FF 36             push    dword ptr [esi]

.text:0042BDF1  68 5C 71 48 00    push    offset aName_1        ; "Name"

.text:0042BDF6  55                push    ebp

.text:0042BDF7  E8 94 55 02 00    call    sub_451390

.text:0042BDFC  FF 74 24 10       push    [esp+14h+var_4]

.text:0042BE00  8B 0D A4 13 49 00 mov     ecx, dword_4913A4

.text:0042BE06  53                push    ebx

.text:0042BE07  55                push    ebp

.text:0042BE08  E8 0E 55 02 00    call    sub_45131B

.text:0042BE0D  A1 A4 13 49 00    mov     eax, dword_4913A4

.text:0042BE12  56                push    esi

.text:0042BE13  8D 88 09 05 00 00 lea     ecx, [eax+509h]

.text:0042BE19  E8 F8 D9 01 00    call    sub_449816

.text:0042BE1E

.text:0042BE1E             loc_42BE1E:             ; CODE XREF: sub_42BCB2+33j

.text:0042BE1E                                             ; sub_42BCB2+B1j

.text:0042BE1E  8B CF             mov     ecx, edi

.text:0042BE20  E8 69 A4 01 00    call    ?OnOK@CDialog@@MAEXXZ ; CDialog::OnOK(void)

.text:0042BE25

.text:0042BE25             loc_42BE25:             ; CODE XREF: sub_42BCB2+96j

.text:0042BE25  5F                pop     edi

.text:0042BE26  5E                pop     esi

.text:0042BE27  5D                pop     ebp

.text:0042BE28  5B                pop     ebx

.text:0042BE29  59                pop     ecx

.text:0042BE2A  C3                retn

.text:0042BE2A             sub_42BCB2 endp
```

再进入 `call` `sub_42C6B9`

#### 子例程 sub_42C6B9

```asm
; 〓〓〓〓〓〓〓〓 S U B R O U T I N E  〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓〓

.text:0042C6B9

.text:0042C6B9

.text:0042C6B9             sub_42C6B9 proc near          ; CODE XREF: sub_416EC7+26p

.text:0042C6B9                                             ; sub_418312+690p

.text:0042C6B9                                             ; sub_418312+69Cp

.text:0042C6B9                                             ; sub_42BCB2+48p

.text:0042C6B9                                             ; sub_42BCB2+77p

.text:0042C6B9

.text:0042C6B9             arg_0= dword ptr  8

.text:0042C6B9

.text:0042C6B9  57                push    edi

.text:0042C6BA  8B 7C 24 08       mov     edi, [esp+arg_0]

.text:0042C6BE  85 FF             test    edi, edi
```
