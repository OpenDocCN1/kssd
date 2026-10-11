
```
.text:0043960E             mov     [ebp+var_B0], 0BCh  

.text:00439617             lea     edx, [ebp+Buffer]  

.text:0043961D             lea     eax, [ebp+var_44]  

.text:00439620             call    ToStr_4E84C8  

.text:00439625             inc     [ebp+var_A4]  

.text:0043962B             mov     edx, [eax]  

.text:0043962D             mov     eax, [ebp+var_E0]  

.text:00439633             mov     ecx, [eax]  

.text:00439635             call    dword ptr [ecx+64h]  

.text:00439638             dec     [ebp+var_A4]  

.text:0043963E             lea     eax, [ebp+var_44]  

.text:00439641             mov     edx, 2  

.text:00439646             call    @System@AnsiString@$bdtr$qqrv ; System::AnsiString::~AnsiString(void)  

.text:0043964B             mov     ecx, [ebp+var_E0]  

.text:00439651             mov     [ebp+var_4C], ecx  

.text:00439654             cmp     [ebp+var_4C], 0  

.text:00439658             jz      short loc_439681  

.text:0043965A             mov     eax, [ebp+var_4C]  

.text:0043965D             mov     edx, [eax]  

.text:0043965F             mov     [ebp+var_48], edx  

.text:00439662             mov     [ebp+var_B0], 0D4h  

.text:0043966B             mov     edx, 3  

.text:00439670             mov     eax, [ebp+var_4C]  

.text:00439673             mov     ecx, [eax]  

.text:00439675             call    dword ptr [ecx-4]  

.text:00439678             mov     [ebp+var_B0], 0C8h  

.text:00439681  

.text:00439681 loc_439681:                                       ; CODE XREF: sub_438E8C+7CCj  

.text:00439681             mov     eax, off_524288  

.text:00439686             mov     edx, [eax]  

.text:00439688             mov     eax, [edx+554h]  

.text:0043968E             mov     dl, 1  

.text:00439690             call    @Menus@TMenuItem@SetEnabled$qqro ; Menus::TMenuItem::SetEnabled(bool)  

.text:00439695             mov     ecx, off_524288  

.text:0043969B             mov     eax, [ecx]  

.text:0043969D             mov     eax, [eax+574h]  

.text:004396A3             mov     dl, 1  

.text:004396A5             mov     ecx, [eax]  

.text:004396A7             call    dword ptr [ecx+5Ch]  

.text:004396AA             mov     eax, off_524288  

.text:004396AF             mov     edx, [eax]  

.text:004396B1             mov     eax, [edx+570h]  

.text:004396B7             mov     dl, 1  

.text:004396B9             mov     ecx, [eax]  

.text:004396BB             call    dword ptr [ecx+5Ch]  

.text:004396BE             mov     eax, off_524288  

.text:004396C3             mov     edx, [eax]  

.text:004396C5             mov     eax, [edx+5B8h]  

.text:004396CB             mov     dl, 1  

.text:004396CD             mov     ecx, [eax]  

.text:004396CF             call    dword ptr [ecx+5Ch]  

.text:004396D2             mov     eax, off_524288  

.text:004396D7             mov     edx, [eax]  

.text:004396D9             mov     eax, [edx+5BCh]  

.text:004396DF             mov     dl, 1  

.text:004396E1             mov     ecx, [eax]  

.text:004396E3             call    dword ptr [ecx+5Ch]  

.text:004396E6             mov     dl, 1  

.text:004396E8             mov     eax, ds:off_4AB0B8  

.text:004396ED             call    @TAngles@$bctr$qqrv ; TAngles::TAngles(void)  

.text:004396F2             mov     [ebp+var_E4], eax  

.text:004396F8             mov     [ebp+var_B0], 0E0h  

.text:00439701             lea     eax, [ebp+var_54]  

.text:00439704             call    nop_38  

.text:00439709             push    eax  

.text:0043970A             inc     [ebp+var_A4]  

.text:00439710             mov     edx, offset aSet_txt_9 ; "set.txt"  

.text:00439715             lea     eax, [ebp+var_50]  

.text:00439718             call    ToStr_4E84C8  

.text:0043971D             inc     [ebp+var_A4]  

.text:00439723             lea     edx, [ebp+var_50]  

.text:00439726             mov     ecx, off_524288  

.text:0043972C             mov     eax, [ecx]  

.text:0043972E             add     eax, 1258h  

.text:00439733             pop     ecx  

.text:00439734             call    @System@AnsiString@$badd$xqqrrx17System@AnsiString ; System::AnsiString::operator+(System::AnsiString &)  

.text:00439739             lea     edx, [ebp+var_54]  

.text:0043973C             mov     edx, [edx]  

.text:0043973E             mov     eax, [ebp+var_E4]  

.text:00439744             mov     ecx, [eax]  

.text:00439746             call    dword ptr [ecx+58h]  

.text:00439749             dec     [ebp+var_A4]  

.text:0043974F             lea     eax, [ebp+var_54]  

.text:00439752             mov     edx, 2  

.text:00439757             call    @System@AnsiString@$bdtr$qqrv ; System::AnsiString::~AnsiString(void)  

.text:0043975C             dec     [ebp+var_A4]  

.text:00439762             lea     eax, [ebp+var_50]  

.text:00439765             mov     edx, 2  

.text:0043976A             call    @System@AnsiString@$bdtr$qqrv ; System::AnsiString::~AnsiString(void)  

.text:0043976F             mov     [ebp+var_B0], 0ECh  

.text:00439778             lea     eax, [ebp+var_58]  

.text:0043977B             call    nop_38  

.text:00439780             mov     ecx, eax  

.text:00439782             inc     [ebp+var_A4]  

.text:00439788             mov     edx, 7  

.text:0043978D             mov     eax, [ebp+var_E4]  

.text:00439793             mov     ebx, [eax]  

.text:00439795             call    dword ptr [ebx+0Ch]  

.text:00439798             lea     eax, [ebp+var_58]  

.text:0043979B             call    @System@AnsiString@ToInt$xqqrv ; System::AnsiString::ToInt(void)  

.text:004397A0             mov     edx, off_524288  

.text:004397A6             mov     ecx, [edx]  

.text:004397A8             mov     [ecx+0DC4h], eax  

.text:004397AE             dec     [ebp+var_A4]  

.text:004397B4             lea     eax, [ebp+var_58]  

.text:004397B7             mov     edx, 2  

.text:004397BC             call    @System@AnsiString@$bdtr$qqrv ; System::AnsiString::~AnsiString(void)  

.text:004397C1             mov     [ebp+var_B0], 0F8h  

.text:004397CA             lea     eax, [ebp+var_5C]  

.text:004397CD             call    nop_38  

.text:004397D2             mov     ecx, eax  

.text:004397D4             inc     [ebp+var_A4]  

.text:004397DA             mov     edx, 8  

.text:004397DF             mov     eax, [ebp+var_E4]  

.text:004397E5             mov     ebx, [eax]  

.text:004397E7             call    dword ptr [ebx+0Ch]  

.text:004397EA             lea     eax, [ebp+var_5C]  

.text:004397ED             call    @System@AnsiString@ToInt$xqqrv ; System::AnsiString::ToInt(void)  

.text:004397F2             mov     edx, off_524288  

.text:004397F8             mov     ecx, [edx]  

.text:004397FA             mov     [ecx+0DC8h], eax  

.text:00439800             dec     [ebp+var_A4]  

.text:00439806             lea     eax, [ebp+var_5C]  

.text:00439809             mov     edx, 2  

.text:0043980E             call    @System@AnsiString@$bdtr$qqrv ; System::AnsiString::~AnsiString(void)  

.text:00439813             mov     [ebp+var_B0], 104h  

.text:0043981C             lea     eax, [ebp+var_60]  

.text:0043981F             call    nop_38  
```
