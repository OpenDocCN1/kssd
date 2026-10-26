
## 汇编代码

```
004E1D93   .  8D4C24 20     lea     ecx, [esp+20]
004E1D97   .  C68424 D00301>mov     byte ptr [esp+103D0], 20
004E1D9F   .  E8 A67D0400   call    <jmp.&MFC42.#4278\_CString::Mid>
004E1DA4   .  894424 10     mov     [esp+10], eax
004E1DA8   .  6A 01         push    1
004E1DAA   .  8D4C24 7C     lea     ecx, [esp+7C]
004E1DAE   .  6A 00         push    0
004E1DB0   .  51            push    ecx
004E1DB1   .  8D4C24 20     lea     ecx, [esp+20]
004E1DB5   .  C68424 D00301>mov     byte ptr [esp+103D0], 21
004E1DBD   .  E8 887D0400   call    <jmp.&MFC42.#4278\_CString::Mid>
004E1DC2   .  8B5424 10     mov     edx, [esp+10]
004E1DC6   .  C68424 C40301>mov     byte ptr [esp+103C4], 22
004E1DCE   .  52            push    edx
004E1DCF   .  50            push    eax
004E1DD0   .  8D4424 48     lea     eax, [esp+48]
004E1DD4   .  50            push    eax
004E1DD5   .  E8 B67F0400   call    <jmp.&MFC42.#922\_operator+>
004E1DDA   .  8B4C24 64     mov     ecx, [esp+64]
004E1DDE   .  8D5424 54     lea     edx, [esp+54]
004E1DE2   .  51            push    ecx
004E1DE3   .  50            push    eax
004E1DE4   .  52            push    edx
004E1DE5   .  C68424 D00301>mov     byte ptr [esp+103D0], 23
004E1DED   .  E8 9E7F0400   call    <jmp.&MFC42.#922\_operator+>
004E1DF2   .  55            push    ebp
004E1DF3   .  50            push    eax
004E1DF4   .  8D4424 30     lea     eax, [esp+30]
004E1DF8   .  C68424 CC0301>mov     byte ptr [esp+103CC], 24
004E1E00   .  50            push    eax
004E1E01   .  E8 8A7F0400   call    <jmp.&MFC42.#922\_operator+>
004E1E06   .  57            push    edi
004E1E07   .  8D4C24 30     lea     ecx, [esp+30]
004E1E0B   .  50            push    eax
004E1E0C   .  51            push    ecx
004E1E0D   .  C68424 D00301>mov     byte ptr [esp+103D0], 25
004E1E15   .  E8 767F0400   call    <jmp.&MFC42.#922\_operator+>
004E1E1A   .  50            push    eax
004E1E1B   .  8D4C24 4C     lea     ecx, [esp+4C]
004E1E1F   .  C68424 C80301>mov     byte ptr [esp+103C8], 26
004E1E27   .  E8 047C0400   call    <jmp.&MFC42.#858\_CString::operator=>
004E1E2C   .  8D4C24 2C     lea     ecx, [esp+2C]
004E1E30   .  C68424 C40301>mov     byte ptr [esp+103C4], 25
004E1E38   .  E8 BB7A0400   call    <jmp.&MFC42.#800\_CString::~CString>
004E1E3D   .  8D4C24 28     lea     ecx, [esp+28]
004E1E41   .  C68424 C40301>mov     byte ptr [esp+103C4], 24
004E1E49   .  E8 AA7A0400   call    <jmp.&MFC42.#800\_CString::~CString>
004E1E4E   .  8D4C24 54     lea     ecx, [esp+54]
004E1E52   .  C68424 C40301>mov     byte ptr [esp+103C4], 23
004E1E5A   .  E8 997A0400   call    <jmp.&MFC42.#800\_CString::~CString>
004E1E5F   .  8D4C24 40     lea     ecx, [esp+40]
004E1E63   .  C68424 C40301>mov     byte ptr [esp+103C4], 22
004E1E6B   .  E8 887A0400   call    <jmp.&MFC42.#800\_CString::~CString>
004E1E70   .  8D4C24 78     lea     ecx, [esp+78]
004E1E74   .  C68424 C40301>mov     byte ptr [esp+103C4], 21
004E1E7C   .  E8 777A0400   call    <jmp.&MFC42.#800\_CString::~CString>
004E1E81   .  8D4C24 7C     lea     ecx, [esp+7C]
004E1E85   .  C68424 C40301>mov     byte ptr [esp+103C4], 20
004E1E8D   .  E8 667A0400   call    <jmp.&MFC42.#800\_CString::~CString>
004E1E92   .  8D4C24 60     lea     ecx, [esp+60]
004E1E96   .  C68424 C40301>mov     byte ptr [esp+103C4], 1F
004E1E9E   .  E8 557A0400   call    <jmp.&MFC42.#800\_CString::~CString>
004E1EA3   .  8D4C24 38     lea     ecx, [esp+38]
004E1EA7   .  C68424 C40301>mov     byte ptr [esp+103C4], 1E
004E1EAF   .  E8 447A0400   call    <jmp.&MFC42.#800\_CString::~CString>
004E1EB4   .  8D4C24 30     lea     ecx, [esp+30]
004E1EB8   .  C68424 C40301>mov     byte ptr [esp+103C4], 1C
004E1EC0   .  E8 337A0400   call    <jmp.&MFC42.#800\_CString::~CString>
004E1EC5   .  8D4C24 48     lea     ecx, [esp+48]
004E1EC9   .  E8 3E810400   call    <jmp.&MFC42.#4204\_CString::MakeUpper>
======================================================================
```

## 高级语言分析

到这里又一个小阶段，以上汇编分析如下高级语言：

`str1 = str1 + str3;`

`str5 = str1.Left(0xA);`

`str6 = str1.Mid(0x0A,1);`

`str6.MakeUpper();`

`str5 = str5 + str6;`

`str7 = str1.Mid(0x0B,1);`

`str7.MakeUpper();`

`str5 = str5 + str7;`

`str8 = str1.Mid(0x0C,5);`

`str5 = str5 + str8;`

`str9 = str1.Mid(0x11,1);`

`str9.MakeUpper();`

`str5 = str5 + str9;`

`str10 = str1.Mid(0x12,1);`

`str10.MakeUpper();`

`str5 = str5 + str10;`

`str11 = str1.Mid(0x13,5);`

`str5 = str5 + str11;`

`str12 = str1.Mid(0x18,1);`

`str5 = str5 + str12;`

`str13 = str1.Mid(0x19,1);`

`str13.MakeUpper();`

`str5 = str5 + str13;`

`str14 = str1.Mid(0x1A,1);`

`str14.MakeLower();`

`str5 = str5 + str14;`

`str15 = str1.Mid(0x1B,1);`

`str15.MakeLower();`

`str5 = str5 + str15;`

`str16 = str1.Mid(0x1C,1);`

`str5 = str5 + str16;`

`str17 = str1.Mid(0x1D,1);`

`str17.MakeUpper();`

`str5 = str5 + str17;`

`str18 = str1.Mid(0x1E,1);`

`str18.MakeUpper();`

`str5 = str5 + str18;`

`str19 = str1.Mid(0x1F,3);`

`str5 = str5 + str19;`

`str20 = str5.Mid(0xA);`

`str21 = str20.Mid(0x13,1);//B`

`str22 = str20.Mid(0x0D,1);//;`

`str23 = str20.Mid(0x8,1);//@`

`str24 = str20.Mid(0x7,1);//G`

`str25 = str20.Mid(0,1);//P`

`str25 = str25 + str24;`

`str25 = str25 + str23;`

`str25 = str25 + str22;`

`str25 = str25 + str21;`

`str25.MakeUpper();`

## 后续汇编代码

```
004E1ECE   .  8B5424 14     mov     edx, [esp+14]                               ;  P[]asdfG@hjkl;'ZxcvBNm,.
004E1ED2   .  8D8424 BC0300>lea     eax, [esp+3BC]                              ;  000FADD4
004E1ED9   .  52            push    edx                                         ; /<%s>
004E1EDA   .  68 3C965E00   push    005E963C                                    ; |format = "%s"
004E1EDF   .  50            push    eax                                         ; |s
004E1EE0   .  FF15 104A5A00 call    [<&MSVCRT.sprintf>]                         ; \sprintf
004E1EE6   .  8A8C24 C80300>mov     cl, [esp+3C8]                               ;  000FADD4
004E1EED   .  83C4 0C       add     esp, 0C
004E1EF0   .  33C0          xor     eax, eax
004E1EF2   .  84C9          test    cl, cl
004E1EF4   .  894424 10     mov     [esp+10], eax
004E1EF8   .  74 18         je      short 004E1F12
004E1EFA   .  8D9424 BC0300>lea     edx, [esp+3BC]                              ;  000FADD4
004E1F01   >  0FBEC9        movsx   ecx, cl
004E1F04   .  03C1          add     eax, ecx
004E1F06   .  8A4A 01       mov     cl, [edx+1]
004E1F09   .  42            inc     edx
004E1F0A   .  84C9          test    cl, cl
004E1F0C   .^ 75 F3         jnz     short 004E1F01
004E1F0E   .  894424 10     mov     [esp+10], eax
004E1F12   >  8B4C24 14     mov     ecx, [esp+14]
004E1F16   .  33ED          xor     ebp, ebp
004E1F18   .  8B41 F8       mov     eax, [ecx-8]
004E1F1B   .  99            cdq
004E1F1C   .  2BC2          sub     eax, edx
004E1F1E   .  D1F8          sar     eax, 1
004E1F20   .  8A1408        mov     dl, [eax+ecx]
004E1F23   .  33C0          xor     eax, eax
004E1F25   .  885424 1F     mov     [esp+1F], dl
004E1F29   >  8A8C04 BC0300>mov     cl, [esp+eax+3BC]
004E1F30   .  84C9          test    cl, cl
004E1F32   .  74 0F         je      short 004E1F43
004E1F34   .  0FBEC9        movsx   ecx, cl
004E1F37   .  83C0 02       add     eax, 2
004E1F3A   .  03E9          add     ebp, ecx
004E1F3C   .  3D FFFF0000   cmp     eax, 0FFFF
004E1F41   .^ 7C E6         jl      short 004E1F29
004E1F43   >  D9ED          fldln2 -------------------------->浮点运算.loge2放给st0
004E1F45   .  DB4424 10     fild    dword ptr [esp+10]----->浮点整数运算
```
