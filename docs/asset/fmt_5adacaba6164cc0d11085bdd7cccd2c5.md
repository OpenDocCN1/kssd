
:0046EA39 8B4C0224          mov ecx, dword ptr [edx+eax+24]

:0046EA3D 890DC48B4700      mov dword ptr [00478BC4], ecx

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

`:0046E999(C), :0046EA0F(U), :0046EA32(U)`

:0046EA43 A14C8C4700        mov eax, dword ptr [00478C4C]

:0046EA48 66837C020400      cmp word ptr [edx+eax+04], 0000

:0046EA4E 7555              jne 0046EAA5

:0046EA50 8D4C241C          lea ecx, dword ptr [esp+1C]

:0046EA54 E8A7F9FFFF        call 0046E400

:0046EA59 33C0              xor eax, eax

:0046EA5B 66A1388C4700      mov ax, word ptr [00478C38]

:0046EA61 85C0              test eax, eax

:0046EA63 740C              je 0046EA71

:0046EA65 83F801            cmp eax, 00000001

:0046EA68 7C3B              jl 0046EAA5

:0046EA6A 83F802            cmp eax, 00000002

:0046EA6D 7E0A              jle 0046EA79

:0046EA6F EB34              jmp 0046EAA5

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:0046EA63(C)`

:0046EA71 8B0D348C4700      mov ecx, dword ptr [00478C34]

:0046EA77 EB27              jmp 0046EAA0

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:0046EA6D(C)`

:0046EA79 8B0D448C4700      mov ecx, dword ptr [00478C44]

:0046EA7F E87CF9FFFF        call 0046E400

:0046EA84 8B0DCC8B4700      mov ecx, dword ptr [00478BCC]

:0046EA8A E871F9FFFF        call 0046E400

:0046EA8F 8B0DC08B4700      mov ecx, dword ptr [00478BC0]

:0046EA95 E866F9FFFF        call 0046E400

:0046EA9A 8B0DC48B4700      mov ecx, dword ptr [00478BC4]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:0046EA77(U)`

:0046EAA0 E85BF9FFFF        call 0046E400

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

`:0046EA4E(C), :0046EA68(C), :0046EA6F(U)`

:0046EAA5 33C0              xor eax, eax

:0046EAA7 66A1388C4700      mov ax, word ptr [00478C38]

:0046EAAD 85C0              test eax, eax

:0046EAAF 7417              je 0046EAC8

:0046EAB1 83F801            cmp eax, 00000001

:0046EAB4 0F8C4B020000      jl 0046ED05

:0046EABA 83F802            cmp eax, 00000002

:0046EABD 0F8E92000000      jle 0046EB55

:0046EAC3 E93D020000        jmp 0046ED05

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:0046EAAF(C)`

:0046EAC8 A1348C4700        mov eax, dword ptr [00478C34]

:0046EACD 803800            cmp byte ptr [eax], 00

:0046EAD0 7519              jne 0046EAEB

:0046EAD2 8B442418          mov eax, dword ptr [esp+18]

:0046EAD6 8A80A3894700      mov al, byte ptr [eax+004789A3]

:0046EADC 3C01              cmp al, 01

:0046EADE 7404              je 0046EAE4

:0046EAE0 3C02              cmp al, 02

:0046EAE2 7507              jne 0046EAEB

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:0046EADE(C)`

:0046EAE4 33ED              xor ebp, ebp

:0046EAE6 E91A020000        jmp 0046ED05

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

`:0046EAD0(C), :0046EAE2(C)`

:0046EAEB 8B3D348C4700      mov edi, dword ptr [00478C34]

:0046EAF1 B9FFFFFFFF        mov ecx, FFFFFFFF

:0046EAF6 2BC0              sub eax, eax

:0046EAF8 F2                repnz

:0046EAF9 AE                scasb

:0046EAFA 0FBF442410        movsx eax, word ptr [esp+10]

:0046EAFF F7D1              not ecx

:0046EB01 49                dec ecx

:0046EB02 3BC8              cmp ecx, eax

:0046EB04 7C15              jl 0046EB1B

:0046EB06 8B3D348C4700      mov edi, dword ptr [00478C34]

:0046EB0C B9FFFFFFFF        mov ecx, FFFFFFFF

:0046EB11 2BC0              sub eax, eax

:0046EB13 F2                repnz

:0046EB14 AE                scasb

:0046EB15 F7D1              not ecx

:0046EB17 49                dec ecx

:0046EB18 668BD9            mov bx, cx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:0046EB04(C)`

:0046EB1B 6633C9            xor cx, cx

:0046EB1E 6685DB            test bx, bx

:0046EB21 7E1E              jle 0046EB41

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:0046EB3F(C)`

:0046EB23 8B15348C4700      mov edx, dword ptr [00478C34]

:0046EB29 0FBFC1            movsx eax, cx

:0046EB2C 8A1402            mov dl, byte ptr [edx+eax]

:0046EB2F 80FA3F            cmp dl, 3F

:0046EB32 7406              je 0046EB3A

:0046EB34 3854041C          cmp byte ptr [esp+eax+1C], dl

:0046EB38 7507              jne 0046EB41

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:0046EB32(C)`

:0046EB3A 6641              inc cx

:0046EB3C 663BCB            cmp cx, bx

:0046EB3F 7CE2              jl 0046EB23

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

`:0046EB21(C), :0046EB38(C)`

:0046EB41 662BCB            sub cx, bx

:0046EB44 BD00000000        mov ebp, 00000000

:0046EB49 6683F901          cmp cx, 0001

:0046EB4D 83D5FF            adc ebp, FFFFFFFF

:0046EB50 E9B0010000        jmp 0046ED05

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:0046EABD(C)`

:0046EB55 8B3D448C4700      mov edi, dword ptr [00478C44]

:0046EB5B B9FFFFFFFF        mov ecx, FFFFFFFF

:0046EB60 2BC0              sub eax, eax

:0046EB62 F2                repnz

:0046EB63 AE                scasb

:0046EB64 F7D1              not ecx

:0046EB66 49                dec ecx

:0046EB67 6649              dec cx

:0046EB69 6683F9FF          cmp cx, FFFF

:0046EB6D 7426              je 0046EB95

:0046EB6F 6685C9            test cx, cx

:0046EB72 7C1B              jl 0046EB8F

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:0046EB8D(C)`

:0046EB74 8B15448C4700      mov edx, dword ptr [00478C44]

:0046EB7A 0FBFC1            movsx eax, cx

:0046EB7D 8A1402            mov dl, byte ptr [edx+eax]

:0046EB80 80FA3F            cmp dl, 3F

:0046EB83 7406              je 0046EB8B

:0046EB85 3854041C          cmp byte ptr [esp+eax+1C], dl

:0046EB89 7504              jne 0046EB8F

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:0046EB83(C)`

:0046EB8B 6649              dec cx

:0046EB8D 79E5              jns 0046EB74

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

`:0046EB72(C), :0046EB89(C)`

:0046EB8F 6683F9FF          cmp cx, FFFF

:0046EB93 7505              jne 0046EB9A

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:0046EB6D(C)`

:0046EB95 BD01000000        mov ebp, 00000001

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:0046EB93(C)`

:0046EB9A 8B3DCC8B4700      mov edi, dword ptr [00478BCC]

:0046EBA0 B9FFFFFFFF        mov ecx, FFFFFFFF

:0046EBA5 2BC0              sub eax, eax

:0046EBA7 F2                repnz

:0046EBA8 AE                scasb

:0046EBA9 F7D1              not ecx

:0046EBAB 49                dec ecx

:0046EBAC 8D7C241C          lea edi, dword ptr [esp+1C]

EDI=13572468          试炼码
