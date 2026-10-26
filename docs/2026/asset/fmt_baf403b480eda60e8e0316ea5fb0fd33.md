
```asm
.text:00408221                 mov     esi, ds:Sleep

.text:00408227                 push    0C8h            ; dwMilliseconds

.text:0040822C                 call    esi ; Sleep

.text:0040822E                 mov     ecx, [esp+0F0h+arg\_4]

.text:00408235                 push    ecx

.text:00408236                 push    ebp

.text:00408237                 call    sub\_40D7D0

.text:0040823C                 add     esp, 8

.text:0040823F                 test    eax, eax

.text:00408241                 jnz     loc\_40881F

.text:00408247                 mov     edx, dword ptr pszPath+8A0h

.text:0040824D                 lea     ecx, [edx+6864h]

.text:00408253                 call    sub\_4115E0

.text:00408258                 cmp     eax, ebx

.text:0040825A                 jnz     loc\_40881F

.text:00408260                 mov     eax, [esp+0F0h+var\_E0]

.text:00408264                 lea     ecx, [esp+0F0h+var\_D8]

.text:00408268                 push    eax

.text:00408269                 lea     edx, [esp+0F4h+var\_D0]

.text:0040826D                 push    ecx

.text:0040826E                 mov     ecx, [esp+0F8h+var\_C8]

.text:00408272                 lea     eax, [esp+0F8h+var\_CC]

.text:00408276                 push    edx

.text:00408277                 push    eax

.text:00408278                 push    ecx

.text:00408279                 push    ebx

.text:0040827A                 call    sub\_415EC0      ; 地图\_外挂验证\_2

.text:0040827F                 add     esp, 18h

.text:00408282                 test    al, al

.text:00408284                 jz      loc\_4087B0

.text:0040828A                 mov     esi, [esp+0F0h+var\_CC]

.text:0040828E                 mov     ecx, [esp+0F0h+var\_D0]

.text:00408292                 cmp     esi, 0FFFFFFFFh

.text:00408295                 jnz     short loc\_4082A0

.text:00408297                 cmp     ecx, 0FFFFFFFFh

.text:0040829A                 jz      loc\_40881F

.text:004082A0

.text:004082A0 loc\_4082A0:                             ; CODE XREF: sub\_408160+135j

.text:004082A0                 mov     edi, [esp+0F0h+var\_D8]

.text:004082A4                 cmp     edi, 58h

.text:004082A7                 jz      loc\_4087C8

.text:004082AD                 cmp     edi, 399h

.text:004082B3                 jnz     short loc\_4082EB

.text:004082B5                 cmp     word ptr [ebp+3F6h], 2Dh

.text:004082BD                 jge     short loc\_4082EB

.text:004082BF                 mov     edx, dword ptr pszPath+8A0h

.text:004082C5                 push    0

.text:004082C7                 push    offset aD\_1     ; "幻境凭证"

.text:004082CC                 lea     ecx, [edx+6864h]

.text:004082D2                 call    sub\_411A40

.text:004082D7                 test    eax, eax

.text:004082D9                 jz      loc\_4087D3

.text:004082DF                 mov     edi, [esp+0F0h+var\_D8]

.text:004082E3                 mov     ecx, [esp+0F0h+var\_D0]

.text:004082E7                 mov     esi, [esp+0F0h+var\_CC]

.text:004082EB

.text:004082EB loc\_4082EB:                             ; CODE XREF: sub\_408160+153j

.text:004082EB                                         ; sub\_408160+15Dj

.text:004082EB                 mov     eax, 51EB851Fh

.text:004082F0                 imul    edi

.text:004082F2                 sar     edx, 5

.text:004082F5                 mov     eax, edx

.text:004082F7                 shr     eax, 1Fh

.text:004082FA                 add     edx, eax

.text:004082FC                 cmp     edx, 9

.text:004082FF                 jnz     loc\_4085DB

//////////////////////////////////////////////////////////////////////////////////////////////////

.text:00415CA0

.text:00415CA0 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

.text:00415CA0

.text:00415CA0 ; 地图\_外挂验证\_1

.text:00415CA0

.text:00415CA0 sub\_415CA0      proc near               ; CODE XREF: sub\_408160+58p

.text:00415CA0

.text:00415CA0 var\_180         = dword ptr -180h

.text:00415CA0 var\_17C         = dword ptr -17Ch

.text:00415CA0 var\_178         = word ptr -178h

.text:00415CA0 var\_176         = word ptr -176h

.text:00415CA0 var\_174         = word ptr -174h

.text:00415CA0 var\_172         = word ptr -172h

.text:00415CA0 var\_170         = dword ptr -170h

.text:00415CA0 var\_140         = dword ptr -140h

.text:00415CA0 var\_138         = word ptr -138h

.text:00415CA0 var\_136         = word ptr -136h

.text:00415CA0 var\_12C         = word ptr -12Ch

.text:00415CA0 var\_12A         = word ptr -12Ah

.text:00415CA0 var\_100         = dword ptr -100h

.text:00415CA0 var\_80          = dword ptr -80h

.text:00415CA0 arg\_4           = dword ptr  8

.text:00415CA0 arg\_8           = dword ptr  0Ch

.text:00415CA0 arg\_C           = dword ptr  10h

.text:00415CA0

.text:00415CA0                 sub     esp, 180h

.text:00415CA6                 push    ebx

.text:00415CA7                 push    ebp

.text:00415CA8                 push    esi

.text:00415CA9                 mov     esi, [esp+18Ch+arg\_4]

.text:00415CB0                 push    edi

.text:00415CB1                 push    esi

.text:00415CB2                 call    sub\_415C80

.text:00415CB7                 add     esp, 4

.text:00415CBA                 test    al, al

.text:00415CBC                 jz      loc\_415EB3

.text:00415CC2                 mov     ebp, [esp+190h+arg\_8]

.text:00415CC9                 push    ebp

.text:00415CCA                 call    sub\_415C80

.text:00415CCF                 add     esp, 4

.text:00415CD2                 test    al, al

.text:00415CD4                 jz      loc\_415EB3

.text:00415CDA                 mov     edx, [esp+190h+arg\_C]

.text:00415CE1                 xor     ecx, ecx

.text:00415CE3                 mov     eax, 59D6CAh

.text:00415CE8                 mov     dword ptr [edx], 0FFFFFFFFh

.text:00415CEE

.text:00415CEE loc\_415CEE:                             ; CODE XREF: sub\_415CA0+6Aj

.text:00415CEE                 movsx   edi, word ptr [eax-2]

.text:00415CF2                 cmp     edi, esi

.text:00415CF4                 jnz     short loc\_415D01

.text:00415CF6                 movsx   edi, word ptr [eax]

.text:00415CF9                 cmp     edi, ebp

.text:00415CFB                 jz      loc\_415DF8

.text:00415D01

.text:00415D01 loc\_415D01:                             ; CODE XREF: sub\_415CA0+54j

.text:00415D01                 add     eax, 0Eh

.text:00415D04                 inc     ecx

.text:00415D05                 cmp     eax, 59DDCAh

.text:00415D0A                 jl      short loc\_415CEE

.text:00415D0C                 mov     edx, dword ptr pszPath+8A0h

.text:00415D12                 lea     ecx, [edx+6864h]

.text:00415D18                 call    sub\_4115B0

.text:00415D1D                 mov     ebx, eax

.text:00415D1F                 mov     ecx, 10h

.text:00415D24                 xor     eax, eax

.text:00415D26                 lea     edi, [esp+190h+var\_180]

.text:00415D2A                 rep stosd

.text:00415D2C                 push    eax             ; time\_t \*

.text:00415D2D                 call    \_time           ; 生成加密用的种子

.text:00415D32                 push    eax             ; unsigned int

.text:00415D33                 call    \_srand

.text:00415D38                 call    \_rand           ; 随机函数

.text:00415D3D                 mov     [esp+198h+var\_180], eax

.text:00415D41                 call    \_rand           ; 随机函数

.text:00415D46                 mov     [esp+198h+var\_17C], eax

.text:00415D4A                 mov     [esp+198h+var\_178], 64h

.text:00415D51                 mov     [esp+198h+var\_174], si

.text:00415D56                 mov     [esp+198h+var\_176], bp

.text:00415D5B                 mov     ax, [ebx+16h]

.text:00415D5F                 lea     edi, [ebx+24h]
```
