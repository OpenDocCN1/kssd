
```
.aspack:00405134             cmp     al, 0E8h        ; 判断是不是 call

.aspack:00405136             jz      short loc_405142

.aspack:00405138             jmp     short $+2

.aspack:0040513A             cmp     al, 0E9h        ; 判断是不是 jmp

.aspack:0040513C             jz      short loc_405142

.aspack:0040513E

.aspack:0040513E loc_40513E:                       ; CODE XREF: .aspack:00405149j

.aspack:0040513E             inc     ebx

.aspack:0040513F             dec     ecx

.aspack:00405140             jmp     short loc_40512D

.aspack:00405142 ; ---------------------------------------------------------------------------

.aspack:00405142

.aspack:00405142 loc_405142:                       ; CODE XREF: .aspack:00405136j

.aspack:00405142                                 ; .aspack:0040513Cj

.aspack:00405142             mov     eax, [esi]

.aspack:00405144             jmp     short $+2

.aspack:00405146             cmp     byte ptr [esi], 0

.aspack:00405149             jnz     short loc_40513E

.aspack:0040514B             and     al, 0

.aspack:0040514D             rol     eax, 18h

.aspack:00405150             sub     eax, ebx

.aspack:00405152             mov     [esi], eax

.aspack:00405154             add     ebx, 5

.aspack:00405157             add     esi, 4

.aspack:0040515A             sub     ecx, 5

.aspack:0040515D             jmp     short loc_40512D

.aspack:0040515F ; ---------------------------------------------------------------------------

.aspack:0040515F

.aspack:0040515F loc_40515F:                       ; CODE XREF: .aspack:0040512Fj

.aspack:0040515F                                 ; .aspack:00405131j

.aspack:0040515F             pop     ebx

.aspack:00405160             pop     esi

.aspack:00405161             pop     ecx

.aspack:00405162             pop     eax

.aspack:00405163

.aspack:00405163 loc_40405163:                       ; CODE XREF: .aspack:00405103j

.aspack:00405163             jmp     short loc_40516D

.aspack:00405163 ; ---------------------------------------------------------------------------

.aspack:00405165             db    0

.aspack:00405166             db    0

.aspack:00405167             db    0

.aspack:00405168             db    0

.aspack:00405169             db    0

.aspack:0040516A             db    0

.aspack:0040516B             db    0

.aspack:0040516C             db    0

.aspack:0040516D ; ---------------------------------------------------------------------------

.aspack:0040516D

.aspack:0040516D loc_40516D:                       ; CODE XREF: .aspack:loc_405163j

.aspack:0040516D             mov     ecx, eax

.aspack:0040516F             mov     edi, [esi]

.aspack:00405171             add     edi, [ebp+422h] ; 解码后要放入的地址

.aspack:00405177             mov     esi, [ebp+152h] ; 解码后的堆地址

.aspack:0040517D             sar     ecx, 2          ; Size / 4

.aspack:00405180             rep movsd

.aspack:00405182             mov     ecx, eax

.aspack:00405184             and     ecx, 3

.aspack:00405187             rep movsb               ; EQU memcpy

.aspack:00405189             pop     esi

.aspack:0040518A             push    8000h

.aspack:0040518F             push    0

.aspack:00405191             push    dword ptr [ebp+152h]

.aspack:00405197             call    dword ptr [ebp+551h] ; VirtualFree

.aspack:0040519D             add     esi, 8          ; 移到下一结构地址

.aspack:004051A0             cmp     dword ptr [esi], 0 ; 循环解码

.aspack:004051A3             jnz     loc_4050C7      ; 获得原来节区的 VSize

.aspack:004051A9             push    8000h

.aspack:004051AE             push    0

.aspack:004051B0             push    dword ptr [ebp+156h]

.aspack:004051B6             call    dword ptr [ebp+551h] ; 全部解压完毕 VitualFree 掉第一次申请的内存堆

.aspack:004051BC             mov     ebx, [ebp+531h]

.aspack:004051C2             or      ebx, ebx

.aspack:004051C4             jz      short loc_4051CE

.aspack:004051C6             mov     eax, [ebx]

.aspack:004051C8             xchg    eax, [ebp+535h]

\========================================================

以下为判断是否要对 pe 进行重定位

\========================================================

.aspack:004051CE                                 ; .aspack:004051C4j

.aspack:004051CE             mov     edx, [ebp+422h]

.aspack:004051D4             mov     eax, [ebp+52Dh]

.aspack:004051DA             sub     edx, eax        ; 判断下当前的 ImageBase 决定是否要进行重定位

.aspack:004051DC             jz      short loc_405257

.aspack:004051DE             mov     eax, edx        ; 以下为进行重定位操作

.aspack:004051E0             shr     eax, 10h

.aspack:004051E3             xor     ebx, ebx

.aspack:004051E5             mov     esi, [ebp+539h]

.aspack:004051EB             add     esi, [ebp+422h]

.aspack:004051F1

.aspack:004051F1 loc_4051F1:                       ; CODE XREF: .aspack:00405255j

.aspack:004051F1             cmp     dword ptr [esi], 0

.aspack:004051F4             jz      short loc_405257

.aspack:004051F6             mov     ecx, [esi+4]

.aspack:004051F9             sub     ecx, 8

.aspack:004051FC             shr     ecx, 1

.aspack:004051FE             mov     edi, [esi]

.aspack:00405200             add     edi, [ebp+422h]

.aspack:00405206             add     esi, 8

.aspack:00405209

.aspack:00405209 loc_405209:                       ; CODE XREF: .aspack:00405253j

.aspack:00405209             mov     bx, [esi]

.aspack:0040520C             shr     ebx, 0Ch

.aspack:0040520F             cmp     ebx, 1

.aspack:00405212             jz      short loc_405220

.aspack:00405214             cmp     ebx, 2

.aspack:00405217             jz      short loc_40522F

.aspack:00405219             cmp     ebx, 3

.aspack:0040521C             jz      short loc_40523E

.aspack:0040521E             jmp     short loc_40524C

.aspack:00405220 ; ---------------------------------------------------------------------------

.aspack:00405220

.aspack:00405220 loc_405220:                       ; CODE XREF: .aspack:00405212j

.aspack:00405220             mov     bx, [esi]

.aspack:00405223             and     ebx, 0FFFh

.aspack:00405229             add     [edi+ebx], ax

.aspack:0040522D             jmp     short loc_40524C

.aspack:0040522F ; ---------------------------------------------------------------------------

.aspack:0040522F

.aspack:0040522F loc_40522F:                       ; CODE XREF: .aspack:00405217j

.aspack:0040522F             mov     bx, [esi]

.aspack:00405232             and     ebx, 0FFFh

.aspack:00405238             add     [edi+ebx], dx

.aspack:0040523C             jmp     short loc_40524C

.aspack:0040523E ; ---------------------------------------------------------------------------

.aspack:0040523E

.aspack:0040523E loc_40523E:                       ; CODE XREF: .aspack:0040521Cj

.aspack:0040523E             mov     bx, [esi]

.aspack:00405241             and     ebx, 0FFFh

.aspack:00405247             add     [edi+ebx], edx

.aspack:0040524A             jmp     short $+2

.aspack:0040524C

.aspack:0040524C loc_40524C:                       ; CODE XREF: .aspack:0040521Ej

.aspack:0040524C                                 ; .aspack:0040522Dj ...
```
