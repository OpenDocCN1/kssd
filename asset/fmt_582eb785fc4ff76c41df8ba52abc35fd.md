
```
.text:00439824                mov     ecx, eax  
.text:00439826                inc     [ebp+var_A4]  
.text:0043982C                mov     edx, 9  
.text:00439831                mov     eax, [ebp+var_E4]  
.text:00439837                mov     ebx, [eax]  
.text:00439839                call    dword ptr [ebx+0Ch]  
.text:0043983C                lea     edx, [ebp+var_60]  
.text:0043983F                mov     eax, off_524288  
.text:00439844                mov     eax, [eax]  
.text:00439846                add     eax, 0DCCh  
.text:0043984B                call    @System@AnsiString@$basg$qqrrx17System@AnsiString ; System::AnsiString::operator=(System::AnsiString &)  
.text:00439850                dec     [ebp+var_A4]  
.text:00439856                lea     eax, [ebp+var_60]  
.text:00439859                mov     edx, 2  
.text:0043985E                call    @System@AnsiString@$bdtr$qqrv ; System::AnsiString::~AnsiString(void)  
.text:00439863                mov     [ebp+var_B0], 110h  
.text:0043986C                lea     eax, [ebp+var_64]  
.text:0043986F                call    nop_38  
.text:00439874                mov     ecx, eax  
.text:00439876                inc     [ebp+var_A4]  
.text:0043987C                mov     edx, 3Bh  
.text:00439881                mov     eax, [ebp+var_E4]  
.text:00439887                mov     ebx, [eax]  
.text:00439889                call    dword ptr [ebx+0Ch]  
.text:0043988C                lea     eax, [ebp+var_64]  
.text:0043988F                call    @System@AnsiString@ToInt$xqqrv ; System::AnsiString::ToInt(void)  
.text:00439894                mov     edx, off_524288  
.text:0043989A                mov     ecx, [edx]  
.text:0043989C                mov     [ecx+0DD0h], eax  
.text:004398A2                dec     [ebp+var_A4]  
.text:004398A8                lea     eax, [ebp+var_64]  
.text:004398AB                mov     edx, 2  
.text:004398B0                call    @System@AnsiString@$bdtr$qqrv ; System::AnsiString::~AnsiString(void)  
.text:004398B5                mov     [ebp+var_B0], 11Ch  
.text:004398BE                lea     eax, [ebp+var_68]  
.text:004398C1                call    nop_38  
.text:004398C6                mov     ecx, eax  
.text:004398C8                inc     [ebp+var_A4]  
.text:004398CE                mov     edx, 3Ch  
.text:004398D3                mov     eax, [ebp+var_E4]  
.text:004398D9                mov     ebx, [eax]  
.text:004398DB                call    dword ptr [ebx+0Ch]  
.text:004398DE                lea     eax, [ebp+var_68]  
.text:004398E1                call    @System@AnsiString@ToInt$xqqrv ; System::AnsiString::ToInt(void)  
.text:004398E6                mov     edx, off_524288  
.text:004398EC                mov     ecx, [edx]  
.text:004398EE                mov     [ecx+0DD4h], eax  
.text:004398F4                dec     [ebp+var_A4]  
.text:004398FA                lea     eax, [ebp+var_68]  
.text:004398FD                mov     edx, 2  
.text:00439902                call    @System@AnsiString@$bdtr$qqrv ; System::AnsiString::~AnsiString(void)  
.text:00439907                mov     [ebp+var_B0], 128h  
.text:00439910                lea     eax, [ebp+var_6C]  
.text:00439913                call    nop_38  
.text:00439918                mov     ecx, eax  
.text:0043991A                inc     [ebp+var_A4]  
.text:00439920                mov     edx, 3Dh  
.text:00439925                mov     eax, [ebp+var_E4]  
.text:0043992B                mov     ebx, [eax]  
.text:0043992D                call    dword ptr [ebx+0Ch]  
.text:00439930                lea     edx, [ebp+var_6C]  
.text:00439933                mov     eax, off_524288  
.text:00439938                mov     eax, [eax]  
.text:0043993A                add     eax, 0DD8h  
.text:0043993F                call    @System@AnsiString@$basg$qqrrx17System@AnsiString ; System::AnsiString::operator=(System::AnsiString &)  
.text:00439944                dec     [ebp+var_A4]  
.text:0043994A                lea     eax, [ebp+var_6C]  
.text:0043994D                mov     edx, 2  
.text:00439952                call    @System@AnsiString@$bdtr$qqrv ; System::AnsiString::~AnsiString(void)  
.text:00439957                lea     eax, [ebp-0E5h]  
.text:0043995D                call    @System@_Set$t19Classes@Classes__92$iuc$0$iuc$1_@$bctr$qqrv ; System::Set<Classes::Classes__92,(uchar)0,(uchar)1>::Set(void)  
.text:00439962                mov     edx, eax  
.text:00439964                mov     ecx, off_524288  
.text:0043996A                mov     eax, [ecx]  
.text:0043996C                add     eax, 0DDCh  
.text:00439971                call    sub_40B674  
.text:00439976                mov     [ebp+var_B0], 134h  
.text:0043997F                lea     eax, [ebp+var_70]  
.text:00439982                call    nop_38  
.text:00439987                mov     ecx, eax  
.text:00439989                inc     [ebp+var_A4]  
.text:0043998F                mov     edx, 3Eh  
.text:00439994                mov     eax, [ebp+var_E4]  
.text:0043999A                mov     ebx, [eax]  
.text:0043999C                call    dword ptr [ebx+0Ch]  
.text:0043999F                lea     eax, [ebp+var_70]  
.text:004399A2                push    eax  
.text:004399A3                lea     eax, [ebp+var_74]  
.text:004399A6                mov     edx, 1  
.text:004399AB                call    @System@WideString@$bctr$qqrul ; System::WideString::WideString(ulong)  
.text:004399B0                inc     [ebp+var_A4]  
.text:004399B6                lea     edx, [ebp+var_74]  
.text:004399B9                pop     eax  
.text:004399BA                call    @System@AnsiString@$beql$xqqrrx17System@AnsiString ; System::AnsiString::operator==(System::AnsiString &)  
.text:004399BF                push    eax  
.text:004399C0                dec     [ebp+var_A4]  
.text:004399C6                lea     eax, [ebp+var_74]  
.text:004399C9                mov     edx, 2  
.text:004399CE                call    @System@AnsiString@$bdtr$qqrv ; System::AnsiString::~AnsiString(void)  
.text:004399D3                dec     [ebp+var_A4]  
.text:004399D9                lea     eax, [ebp+var_70]  
.text:004399DC                mov     edx, 2  
.text:004399E1                call    @System@AnsiString@$bdtr$qqrv ; System::AnsiString::~AnsiString(void)  
.text:004399E6                pop     ecx  
.text:004399E7                test    cl, cl  
.text:004399E9                jz      short loc_4399FE  
.text:004399EB                mov     eax, off_524288  
.text:004399F0                mov     eax, [eax]  
.text:004399F2                add     eax, 0DDCh  
.text:004399F7                xor     edx, edx  
.text:004399F9                call    sub_40B6B0  
.text:004399FE  
.text:004399FE loc_4399FE:                             ; CODE XREF: sub_438E8C+B5Dj  
.text:004399FE                mov     [ebp+var_B0], 140h  
.text:00439A07                lea     eax, [ebp+var_78]  
.text:00439A0A                call    nop_38  
.text:00439A0F                mov     ecx, eax  
.text:00439A11                inc     [ebp+var_A4]  
.text:00439A17                mov     edx, 3Fh  
.text:00439A1C                mov     eax, [ebp+var_E4]  
.text:00439A22                mov     ebx, [eax]  
.text:00439A24                call    dword ptr [ebx+0Ch]  
.text:00439A27                lea     eax, [ebp+var_78]  
.text:00439A2A                push    eax  
.text:00439A2B                lea     eax, [ebp+var_7C]  
```
