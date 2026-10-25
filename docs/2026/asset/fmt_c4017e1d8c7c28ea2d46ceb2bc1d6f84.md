
```
.text:00440B9C                 call    zshread          \*\*\*\*\*\*\*\*\*读入E

.text:00440BA1                 add     esp, 8

.text:00440BA4                 lea     edx, [ebp+var\_CC]

.text:00440BAA                 push    edx

.text:00440BAB                 lea     ecx, [ebp+var\_14C]

.text:00440BB1                 push    ecx

.text:00440BB2                 call    zsread            \*\*\*\*\*\*\*\*\*读入数据

.text:00440BB7                 add     esp, 8

.text:00440BBA                 lea     eax, [ebp+var\_C4]

.text:00440BC0                 push    eax             ; int

.text:00440BC1                 push    offset a1f3662faa8e2\_0 ; buffer

.text:00440BC6                 call    zshread           \*\*\*\*\*\*\*\*\*读入N

.text:00440BCB                 add     esp, 8

.text:00440BCE                 lea     edx, [ebp+var\_D4]

.text:00440BD4                 push    edx

.text:00440BD5                 mov     ecx, [ebp+var\_C4]

.text:00440BDB                 push    ecx

.text:00440BDC                 mov     eax, [ebp+var\_C8]

.text:00440BE2                 push    eax

.text:00440BE3                 mov     edx, [ebp+var\_CC]

.text:00440BE9                 push    edx

.text:00440BEA                 call    zexpmod           \*\*\*\*\*\*\*\*\*\*RSA解密

.text:00440BEF                 add     esp, 10h

.text:00440BF2                 lea     ecx, [ebp+var\_D0]

.text:00440BF8                 push    ecx             ; int

.text:00440BF9                 lea     eax, [ebp+buffer]

.text:00440BFF                 push    eax             ; buffer

.text:00440C00                 call    zshread           \*\*\*\*\*\*\*\*\*\*\*读入用户名变换后的数据

.text:00440C05                 add     esp, 8

.text:00440C08                 mov     edx, [ebp+var\_D4]

.text:00440C0E                 push    edx

.text:00440C0F                 mov     ecx, [ebp+var\_D0]

.text:00440C15                 push    ecx

.text:00440C16                 call    sub\_43C80C         \*\*\*\*\*\*\*\*\*\*\*\*比较

.text:00440C1B                 add     esp, 8

.text:00440C1E                 test    eax, eax

.text:00440C20                 jz      short loc\_440C30   \*\*\*\*\*\*\*\*\*\*\*\*相等则继续处理

.text:00440C22                 mov     [ebp+var\_94], 8

.text:00440C2B                 jmp     loc\_440CF2

.text:00440C30 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?

.text:00440C30

.text:00440C30 loc\_440C30:                             ; CODE XREF: \_TForm1\_SpeedButton1Click+F24j

=========================================================下面这段不用理会

.text:00440C30                 mov     eax, dword\_50AAB4

.text:00440C35                 lea     edi, [ebp+s]

.text:00440C3B                 mov     edx, [eax]

.text:00440C3D                 xor     eax, eax

.text:00440C3F                 add     edx, 2111h

.text:00440C45                 or      ecx, 0FFFFFFFFh

.text:00440C48                 repne scasb

.text:00440C4A                 not     ecx

.text:00440C4C                 sub     edi, ecx

.text:00440C4E                 mov     esi, edx

.text:00440C50                 xchg    esi, edi

.text:00440C52                 mov     edx, ecx

.text:00440C54                 mov     eax, edi

.text:00440C56                 shr     ecx, 2

.text:00440C59                 rep movsd

.text:00440C5B                 mov     ecx, edx

.text:00440C5D                 and     ecx, 3

.text:00440C60                 rep movsb

.text:00440C62                 mov     eax, dword\_50AAB4

.text:00440C67                 lea     edi, [ebp+var\_14D+1]

.text:00440C6D                 mov     edx, [eax]

.text:00440C6F                 xor     eax, eax

.text:00440C71                 add     edx, 212Ah

.text:00440C77                 or      ecx, 0FFFFFFFFh

.text:00440C7A                 repne scasb

.text:00440C7C                 not     ecx

.text:00440C7E                 sub     edi, ecx

.text:00440C80                 mov     esi, edx

.text:00440C82                 xchg    esi, edi

.text:00440C84                 mov     edx, ecx

.text:00440C86                 mov     eax, edi

.text:00440C88                 shr     ecx, 2

.text:00440C8B                 rep movsd

.text:00440C8D                 mov     ecx, edx

.text:00440C8F                 and     ecx, 3

.text:00440C92                 rep movsb

========================================================

.text:00440C94                 mov     eax, [ebp+var\_D4]

.text:00440C9A                 push    eax

.text:00440C9B                 mov     edx, [ebp+var\_D0]

.text:00440CA1                 push    edx

.text:00440CA2                 call    sub\_43C80C   \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*这里再比较一次(不放心? @##$$%  ^\_^)

.text:00440CA7                 add     esp, 8

=========================================================下面这段不用理会

.text:00440CAA                 test    eax, eax

.text:00440CAC                 jnz     short loc\_440CF2

.text:00440CAE                 mov     ecx, dword\_50AAB4

.text:00440CB4                 mov     eax, [ecx]

.text:00440CB6                 mov     byte ptr [eax+2161h], 1

.text:00440CBD                 xor     ebx, ebx

.text:00440CBF

.text:00440CBF loc\_440CBF:                             ; CODE XREF: \_TForm1\_SpeedButton1Click+FF4j

.text:00440CBF                 call    sub\_4CD910

.text:00440CC4                 cdq

.text:00440CC5                 mov     ecx, 32h

.text:00440CCA                 idiv    ecx

.text:00440CCC                 lea     eax, [ebx+ebx\*4]

.text:00440CCF                 add     dl, 65h

.text:00440CD2                 mov     ecx, dword\_50AAB4

.text:00440CD8                 lea     eax, [ebx+eax\*2]

.text:00440CDB                 shl     eax, 3

.text:00440CDE                 sub     eax, ebx

.text:00440CE0                 mov     ecx, [ecx]

.text:00440CE2                 lea     eax, [ebx+eax\*4]

.text:00440CE5                 mov     [ecx+eax\*4+0A46h], dl

.text:00440CEC                 inc     ebx

.text:00440CED                 cmp     ebx, 3

.text:00440CF0                 jl      short loc\_440CBF

=========================================================

.text:00440CF2

.text:00440CF2 loc\_440CF2:                             ; CODE XREF: \_TForm1\_SpeedButton1Click+F2Fj

.text:00440CF2                                         ; \_TForm1\_SpeedButton1Click+FB0j

.text:00440CF2                 mov     [ebp+var\_94], 188h

.text:00440CFB                 mov     edx, offset aPVSgb ; "关闭程序重新启动完成注册！"

.text:00440D00                 lea     eax, [ebp+var\_80]

.text:00440D03                 call    tostring

.text:00440D08                 inc     [ebp+var\_88]

.text:00440D0E                 mov     eax, [eax]

.text:00440D10                 call    @Dialogs@ShowMessage$qqrx17System@AnsiString ; Dialogs::ShowMessage(System::AnsiString)

.text:00440D15                 dec     [ebp+var\_88]

.text:00440D1B                 lea     eax, [ebp+var\_80]

.text:00440D1E                 mov     edx, 2

.text:00440D23                 call    @System@AnsiString@$bdtr$qqrv ; System::AnsiString::~AnsiString(void)

.text:00440D28                 mov     ecx, dword\_50AAB4

.text:00440D2E                 mov     eax, [ecx]

.text:00440D30                 call    @Forms@TCustomForm@Close$qqrv ; Forms::TCustomForm::Close(void)

.text:00440D35                 dec     [ebp+var\_88]

.text:00440D3B                 lea     eax, [ebp+var\_28]

.text:00440D3E                 mov     edx, 2

.text:00440D43                 call    @System@AnsiString@$bdtr$qqrv ; System::AnsiString::~AnsiString(void)

.text:00440D48                 dec     [ebp+var\_88]

.text:00440D4E                 lea     eax, [ebp+var\_24]

.text:00440D51                 mov     edx, 2

.text:00440D56                 call    @System@AnsiString@$bdtr$qqrv ; System::AnsiString::~AnsiString(void)

.text:00440D5B                 dec     [ebp+var\_88]

.text:00440D61                 lea     eax, [ebp+var\_20]

.text:00440D64                 mov     edx, 2
```
