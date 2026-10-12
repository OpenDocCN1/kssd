
```
0046F89A     8B4432 10             mov eax,dword ptr ds:[edx+esi+10]

0046F8A6     8BF0                mov esi,eax

0046F8AE     F783 D8204000 FFFFFFFF test dword ptr ds:[ebx+4020D8],FFFFFFFF

0046F8BF     74 0E               je short 0046F8CF

0046F8C8     03F2                add esi,edx

0046F8D2     833E 00             cmp dword ptr ds:[esi],0

0046F8DA     0F84 BA000000       je 0046F99A

0046F8E3     8B06                mov eax,dword ptr ds:[esi]

0046F8EA     A9 00000080         test eax,80000000

0046F8F4     75 1E               jnz short 0046F914

0046F8FB     3BC1                cmp eax,ecx

0046F900     73 12               jnb short 0046F914

0046F907     03C2                add eax,edx

0046F90C     83C0 02             add eax,2

0046F917     50                  push eax

0046F91D     50                  push eax

0046F921     51                  push ecx

0046F926     E8 0DDCFFFF         call 0046D538

0046F92E     E8 25F9FFFF         call 0046F258

0046F937     83F8 00             cmp eax,0

0046F93E     74 13               je short 0046F953

0046F944     E8 79F4FFFF         call 0046EDC2

0046F94D     8907                mov dword ptr ds:[edi],eax///加密后的 IAT 地址移入地址表。

0046F956     8B83 AC204000       mov eax,dword ptr ds:[ebx+4020AC]

0046F960     0107                add dword ptr ds:[edi],eax

0046F967     83C7 04             add edi,4

0046F971     F783 E0204000 FFFFFFFF test dword ptr ds:[ebx+4020E0],FFFFFFFF

0046F983     74 15               je short 0046F99A

0046F98B     83C6 04             add esi,4

0046F991     E9 39FFFFFF         jmp 0046F8CF///循环。

0046F99D     5E                  pop esi

0046F9A3     83C6 14             add esi,14

0046F9A9     E9 81FEFFFF         jmp 0046F82F///循环。

0046F9B9     8B83 AC204000       mov eax,dword ptr ds:[ebx+4020AC]///F4 到这里。

0046F9C2     8BBB C4204000       mov edi,dword ptr ds:[ebx+4020C4]///EAX=60000 即输入表地址。

0046F9CD     03FA                add edi,edx///EDI=460000

0046F9D2     8B8B C8204000       mov ecx,dword ptr ds:[ebx+4020C8]

0046F9DD     3183 B71D4000       xor dword ptr ds:[ebx+401DB7],eax

0046F9E6     B0 00               mov al,0

0046F9FF     8B93 BF1D4000       mov edx,dword ptr ds:[ebx+401DBF]

0046FA0A     8B72 3C             mov esi,dword ptr ds:[edx+3C]

0046FA10     8BB3 CC204000       mov esi,dword ptr ds:[ebx+4020CC]

0046FA27     83FE 00             cmp esi,0

0046FA2F     0F84 D5000000       je 0046FB0A

0046FA38     8D3432              lea esi,dword ptr ds:[edx+esi]

0046FA43     833E 00             cmp dword ptr ds:[esi],0

0046FA4B     0F84 B9000000       je 0046FB0A

0046FA54     56                  push esi

0046FA5A     8B3E                mov edi,dword ptr ds:[esi]

0046FA5F     8B4E 04             mov ecx,dword ptr ds:[esi+4]

0046FA67     83E9 08             sub ecx,8

0046FA6D     D1E9                shr ecx,1

0046FA77     0FB746 08           movzx eax,word ptr ds:[esi+8]

0046FA80     50                  push eax

0046FA84     66:25 0030          and ax,3000

0046FA8C     66:3D 0030          cmp ax,3000

0046FA94     58                  pop eax

0046FA99     75 4B               jnz short 0046FAE6

0046FA9F     66:25 FF0F          and ax,0FFF

0046FAA7     03C7                add eax,edi

0046FAAC     011402              add dword ptr ds:[edx+eax],edx

0046FAB3     51                  push ecx

0046FAB7     8B8B D4204000       mov ecx,dword ptr ds:[ebx+4020D4]

0046FAC1     290C02              sub dword ptr ds:[edx+eax],ecx

0046FAC8     8B8B C0204000       mov ecx,dword ptr ds:[ebx+4020C0]

0046FAD2     038B BB1D4000       add ecx,dword ptr ds:[ebx+401DBB]

0046FADB     290C02              sub dword ptr ds:[edx+eax],ecx

0046FAE2     59                  pop ecx

0046FAEA     83C6 02             add esi,2

0046FAF1     E2 81               loopd short 0046FA74

0046FAF7     5E                  pop esi

0046FAFC     0376 04             add esi,dword ptr ds:[esi+4]

0046FB02     E9 39FFFFFF         jmp 0046FA40

0046FB21     8BBB CC204000       mov edi,dword ptr ds:[ebx+4020CC]

0046FB2C     03FA                add edi,edx

0046FB31     8B8B D0204000       mov ecx,dword ptr ds:[ebx+4020D0]

0046FB3C     B0 00               mov al,0

0046FB4D     8B83 B4204000       mov eax,dword ptr ds:[ebx+4020B4]

0046FB5B     8B93 BF1D4000       mov edx,dword ptr ds:[ebx+401DBF]

0046FB64     8B72 3C             mov esi,dword ptr ds:[edx+3C]

0046FB6C     8DB432 F8000000     lea esi,dword ptr ds:[edx+esi+F8]

00046FB76    8BFE                mov edi,esi

0046FB7C     3183 B71D4000       xor dword ptr ds:[ebx+401DB7],eax

0046FB89     6BB3 76214000 28    imul esi,dword ptr ds:[ebx+402176],28

0046FB93     03F7                add esi,edi

0046FB9A     EB 44               jmp short 0046FBE0

0046FBB7     837E 0C 00          cmp dword ptr ds:[esi+C],0

0046FBBE     74 20               je short 0046FBE0

0046FBC5     837E 08 00          cmp dword ptr ds:[esi+8],0

0046FBCC     74 12               je short 0046FBE0

0046FBD3     83C6 28             add esi,28

0046FBD9     EB AA               jmp short 0046FB85

0046FBE3     83EE 28             sub esi,28

0046FBEB     87FE                xchg esi,edi

0046FBFA     3BF7                cmp esi,edi

0046FC01     77 69               ja short 0046FC6C

0046FC0A     59                  pop ecx

0046FC0F     52                  push edx

0046FC19     8D83 BC204000       lea eax,dword ptr ds:[ebx+4020BC]

0046FC23     50                  push eax

0046FC2F     51                  push ecx

0046FC34     FF76 08             push dword ptr ds:[esi+8]

0046FC3B     8B46 0C             mov eax,dword ptr ds:[esi+C]

0046FC42     03C2                add eax,edx

0046FC47     50                  push eax

0046FC4C     FF93 D31E4000       call near dword ptr ds:[ebx+401ED3]     ; kernel32.VirtualProtect

0046FC57     5A                  pop edx

0046FC5E     83C6 28             add esi,28

0046FC67     EB 89               jmp short 0046FBF2

0046FC73     8B83 B71D4000       mov eax,dword ptr ds:[ebx+401DB7]///eax=0004B720

0046FC7E     03C2                add eax,edx///edx=00400000 (ccc.00400000)

0046FC87     0383 C0204000       add eax,dword ptr ds:[ebx+4020C0]///eax=44b720 入口地址!

0046FC93     3383 BB1D4000       xor eax,dword ptr ds:[ebx+401DBB]

0046FCA0     50                  push eax

0046FCB1     838B E4204000 07    or dword ptr ds:[ebx+4020E4],7

0046FCBC     818B E4204000 00000080 or dword ptr ds:[ebx+4020E4],80000000

0046FCCE     8B83 F0204000       mov eax,dword ptr ds:[ebx+4020F0]

0046FCD7     25 07000080         and eax,80000007

0046FCE0     35 07000080         xor eax,80000007

0046FCE8     75 DF               jnz short 0046FCC9///死循环。

0046FCF1     58                  pop eax///新建 EIP 跳出循环，弹出入口地址。

0046FCFA     8983 B33D4000       mov dword ptr ds:[ebx+403DB3],eax

0046FD08     6A 00               push 0

0046FD0A     6A 00               push 0

0046FD0C     E8 CDDBFFFF         call 0046D8DE///检测 CC

0046FD22     838B 20214000 01    or dword ptr ds:[ebx+402120],1
```
