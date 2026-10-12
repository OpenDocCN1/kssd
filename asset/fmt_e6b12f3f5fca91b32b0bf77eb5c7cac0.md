
```
004EDDCB  663145FE            xor    [ebp-$02], ax

004EDDCF  8D55FE             lea    edx, [ebp-$02]

004EDDD2  B901000000         mov    ecx, $00000001

004EDDD7  8BC6               mov    eax, esi

004EDDD9  8B38               mov    edi, [eax]

* Reference to method TMemoryStream.Write(Pointer,Longint)

|

004EDDDB  FF5708             call   dword ptr [edi+$08]

004EDDDE  66FF45FC           inc    word ptr [ebp-$04]

004EDE02  E88152F1FF         call   00403088

004EDE07  EB07               jmp    004EDE10

004EDE09  C605AC394F0001     mov    byte ptr [$4F39AC], $01

004EDE10  803DAC394F0000     cmp    byte ptr [$4F39AC], $00

004EDE17  741F               jz     004EDE38

004EDE19  8D45CC             lea    eax, [ebp-$34]

* Possible String Reference to: ' UNREGISTERED'

|

004EDE1C  B960E04E00         mov    ecx, $004EE060

004EDE21  8B15A8394F00       mov    edx, [$4F39A8]

* Reference to: System..LStrCat3()

|

004EDE27  E85062F1FF         call   0040407C

004EDE2C  8B55CC             mov    edx, [ebp-$34]

004EDE2F  8BC3               mov    eax, ebx

* Reference to: Controls.TControl.SetText(System.AnsiString)

|

004EDE31  E81A94F5FF         call   00447250

004EDE36  EB1C               jmp    004EDE54

004EDE38  8B15A8394F00       mov    edx, [$4F39A8]

004EDE3E  8BC3               mov    eax, ebx

* Reference to: Controls.TControl.SetText(System.AnsiString)

|

004EDE40  E80B94F5FF         call   00447250

004EDE45  EB0D               jmp    004EDE54
```

输入注册码时的判断在`TEdit`的`OnChange()`中。启动时的判断只是对`keyfile`进行校验。

**Registered User**

1257167423

```pascal
//-------------------------------------------------------------

procedure TfmRegistration.Edit2Change(Sender: TObject);

begin

{

004BCAA0  55                 push   ebp

004BCAA1  8BEC               mov    ebp, esp

004BCAA3  33C9               xor    ecx, ecx

004BCAA5  51                 push   ecx

004BCAA6  51                 push   ecx

004BCAA7  51                 push   ecx

004BCAA8  51                 push   ecx

004BCAA9  51                 push   ecx

004BCAAA  53                 push   ebx

004BCAAB  8BD8               mov    ebx, eax

004BCAAD  33C0               xor    eax, eax

004BCAAF  55                 push   ebp

004BCAB0  688ACB4B00         push   $004BCB8A

***** TRY

|

004BCAB5  64FF30             push   dword ptr fs:[eax]

004BCAB8  648920             mov    fs:[eax], esp

004BCABB  8D55FC             lea    edx, [ebp-$04]

* Reference to control TfmRegistration.Edit2 : TEdit

|

004BCABE  8B83E4020000       mov    eax, [ebx+$02E4]

* Reference to: Controls.TControl.GetText()

|

004BCAC4  E857A7F8FF         call   00447220

004BCAC9  8B45FC             mov    eax, [ebp-$04]

* Reference to: System..DynArrayLength()

|          or: System..LStrLen()

|

004BCACC  E85F75F4FF         call   00404030

004BCAD1  83F80A             cmp    eax, +$0A       //注册码长度

004BCAD4  0F858D000000       jnz    004BCB67

004BCADA  8D55F8             lea    edx, [ebp-$08]

* Reference to control TfmRegistration.Edit2 : TEdit

|

004BCADD  8B83E4020000       mov    eax, [ebx+$02E4]

* Reference to: Controls.TControl.GetText()

|

004BCAE3  E838A7F8FF         call   00447220

004BCAE8  8B45F8             mov    eax, [ebp-$08]

004BCAEB  8B1504664F00       mov    edx, [$4F6604]

* Reference to: System..LStrCmp()

|

004BCAF1  E84A76F4FF         call   00404140        //比较注册码

004BCAF6  756F               jnz    004BCB67

004BCAF8  8D45F4             lea    eax, [ebp-$0C]

004BCAFB  50                 push   eax

004BCAFC  8D55F0             lea    edx, [ebp-$10]

* Reference to control TfmRegistration.Edit2 : TEdit

|

004BCAFF  8B83E4020000       mov    eax, [ebx+$02E4]

* Reference to: Controls.TControl.GetText()

|

004BCB05  E816A7F8FF         call   00447220

004BCB0A  8B45F0             mov    eax, [ebp-$10]

004BCB0D  B905000000         mov    ecx, $00000005

004BCB12  BA05000000         mov    edx, $00000005

* Reference to: System..LStrCopy()

|

004BCB17  E81C77F4FF         call   00404238

004BCB1C  8B45F4             mov    eax, [ebp-$0C]

004BCB1F  50                 push   eax

004BCB20  8D45EC             lea    eax, [ebp-$14]

004BCB23  50                 push   eax

004BCB24  B905000000         mov    ecx, $00000005

004BCB29  BA05000000         mov    edx, $00000005

004BCB2E  A104664F00         mov    eax, dword ptr [$4F6604]

* Reference to: System..LStrCopy()

|

004BCB33  E80077F4FF         call   00404238

004BCB38  8B55EC             mov    edx, [ebp-$14]

004BCB3B  58                 pop    eax

* Reference to: System..LStrCmp()

|

004BCB3C  E8FF75F4FF         call   00404140      //比较注册码中间的 5 位

004BCB41  7524               jnz    004BCB67

* Reference to control TfmRegistration.BitBtn1 : TBitBtn

|

004BCB43  8B83D0020000       mov    eax, [ebx+$02D0]

* Reference to field TBitBtn.OFFS_010C

|

004BCB49  89980C010000       mov    [eax+$010C], ebx

* Reference to field TBitBtn.OFFS_0108

|

004BCB4F  C7800801000064C74B00 mov  dword ptr [eax+$0108], $004BC764

* Reference to field TfmRegistration.OFFS_02EC

|

004BCB59  C683EC02000001     mov    byte ptr [ebx+$02EC], $01

* Reference to field TfmRegistration.OFFS_02ED

|

004BCB60  C683ED02000001     mov    byte ptr [ebx+$02ED], $01

004BCB67  33C0               xor    eax, eax

004BCB69  5A                 pop    edx

004BCB6A  59                 pop    ecx

004BCB6B  59                 pop    ecx

004BCB6C  648910             mov    fs:[eax], edx

***** FINALLY

|

* Possible String Reference to: '[嬪]脣繳嬱 3 繳 h 伤 K'

|

004BCB6F  6891CB4B00         push   $004BCB91

004BCB74  8D45EC             lea    eax, [ebp-$14]

* Reference to: System..LStrClr(System.AnsiString)

|

004BCB77  E83472F4FF         call   00403DB0

004BCB7C  8D45F0             lea    eax, [ebp-$10]

004BCB7F  BA04000000         mov    edx, $00000004

* Reference to: System..LStrArrayClr()

|

004BCB84  E84B72F4FF         call   00403DD4

004BCB89  C3                 ret

004BCB8A  E9196CF4FF         jmp    004037A8

004BCB8F  EBE3               jmp    004BCB74

***** END

|

004BCB91  5B                 pop    ebx

004BCB92  8BE5               mov    esp, ebp

004BCB94  5D                 pop    ebp

004BCB95  C3                 ret  4dd466d0e1e0cc123afbe504da2462ca
```
