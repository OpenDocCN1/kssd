
```asm
.text:00421D7B E8 74 8C 00 00 call __sprintf //把 esi 转成长度为 4 位的 16 进制数据，保存到[esi]="1532"

.text:00421D80 83 C4 0C add esp, 0Ch

.text:00421D83 33 C0 xor eax, eax //eax=0

.text:00421D85

.text:00421D85 loc_421D85: ; CODE XREF: sub_421CE0+B5j

.text:00421D85 0F BE 0C 38 movsx ecx, byte ptr [eax+edi] //ecx 指向[edi]第 i 个字符

.text:00421D89 40 inc eax //eax++

.text:00421D8A 8A 54 0C DC mov dl, [esp+ecx+24h+var_48] //取得字符串表"RHTLMWYAEF XJPKCG"中对应的字符

.text:00421D8E 83 F8 04 cmp eax, 4 //判断是否处理完

.text:00421D91 88 54 38 FF mov [eax+edi-1], dl //将取得的字符保存回去

.text:00421D95 7C EE jl short loc_421D85 //循环处理，处理完后[edi]="XGCL"

.text:00421D97 33 C0 xor eax, eax

.text:00421D99

.text:00421D99 loc_421D99: ; CODE XREF: sub_421CE0+C9j

.text:00421D99 0F BE 0C 30 movsx ecx, byte ptr [eax+esi] //ecx 指向[esi]第 i 个字符

.text:00421D9D 40 inc eax

.text:00421D9E 8A 54 0C DC mov dl, [esp+ecx+24h+var_48] //取得字符串表"RHTLMWYAEF XJPKCG"中对应的字符

.text:00421DA2 83 F8 04 cmp eax, 4 //判断是否处理完

.text:00421DA5 88 54 30 FF mov [eax+esi-1], dl //将取得的字符保存回去

.text:00421DA9 7C EE jl short loc_421D99 //循环处理，处理完后[esi]="HWLT"

.text:00421DAB

.text:00421DAB loc_421DAB: ; CODE XREF: sub_421CE0+36j

.text:00421DAB 5F pop edi

.text:00421DAC 5E pop esi

.text:00421DAD 5B pop ebx

.text:00421DAE 83 C4 18 add esp, 18h

.text:00421DB1 C2 0C 00 retn 0Ch

.text:00421DB1 sub_421CE0 endp

}

.text:00422042 8B 83 39 01 00 00 mov eax, [ebx+139h]

.text:00422048 85 C0 test eax, eax

.text:0042204A 74 25 jz short loc_422071

.text:0042204C 8D 7C 24 30 lea edi, [esp+30h]

.text:00422050 83 C9 FF or ecx, 0FFFFFFFFh

.text:00422053 33 C0 xor eax, eax

.text:00422055 8D 54 24 18 lea edx, [esp+18h]

.text:00422059 F2 AE repne scasb

.text:0042205B F7 D1 not ecx

.text:0042205D 2B F9 sub edi, ecx

.text:0042205F 8B C1 mov eax, ecx

.text:00422061 8B F7 mov esi, edi

.text:00422063 8B FA mov edi, edx

.text:00422065 C1 E9 02 shr ecx, 2

.text:00422068 F3 A5 rep movsd

.text:0042206A 8B C8 mov ecx, eax

.text:0042206C 83 E1 03 and ecx, 3

.text:0042206F F3 A4 rep movsb

.text:00422071

.text:00422071 loc_422071: ; CODE XREF: .text:0042204Aj

.text:00422071 8D 74 24 30 lea esi, [esp+30h] //esi 指向"XGCL"

.text:00422075 8D 44 24 18 lea eax, [esp+18h] //eax 指向"3333"

.text:00422079 5F pop edi

.text:0042207A

.text:0042207A loc_42207A: ; CODE XREF: .text:0042209Cj

.text:0042207A 8A 10 mov dl, [eax] //dl 指向字符"3333"第 1 个字符

.text:0042207C 8A 1E mov bl, [esi] //bl 指向字符"XGCL"第 1 个字符

.text:0042207E 8A CA mov cl, dl //cl=dl

.text:00422080 3A D3 cmp dl, bl //比较 dl,bl

.text:00422082 75 1E jnz short loc_4220A2 //不等结束

.text:00422084 84 C9 test cl, cl //测试 cl 是否为 0

.text:00422086 74 16 jz short loc_42209E //为 0 结束

.text:00422088 8A 50 01 mov dl, [eax+1] //dl 指向字符"3333"第 2 个字符

.text:0042208B 8A 5E 01 mov bl, [esi+1] //bl 指向字符"XGCL"第 2 个字符

.text:0042208E 8A CA mov cl, dl

.text:00422090 3A D3 cmp dl, bl //比较 dl,bl

.text:00422092 75 0E jnz short loc_4220A2

.text:00422094 83 C0 02 add eax, 2

.text:00422097 83 C6 02 add esi, 2

.text:0042209A 84 C9 test cl, cl 测试 cl 是否为 0

.text:0042209C 75 DC jnz short loc_42207A //不为 0 跳回继续比较下面 2 个字符

.text:0042209E

.text:0042209E loc_42209E: ; CODE XREF: .text:00422086j

.text:0042209E 33 C0 xor eax, eax //都相等后，由 00422086 跳到这里，eax=0

.text:004220A0 EB 05 jmp short loc_4220A7

.text:004220A2 ; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

.text:004220A2

.text:004220A2 loc_4220A2: ; CODE XREF: .text:00422082j
.text:004220A2 ; .text:00422092j

.text:004220A2 1B C0 sbb eax, eax

.text:004220A4 83 D8 FF sbb eax, 0FFFFFFFFh

.text:004220A7

.text:004220A7 loc_4220A7: ; CODE XREF: .text:004220A0j

.text:004220A7 85 C0 test eax, eax

.text:004220A9 74 7C jz short loc_422127 //跳到 loc_422127

.text:004220AB 8D 74 24 70 lea esi, [esp+70h]

.text:004220AF 8D 44 24 14 lea eax, [esp+14h]

.text:004220B3

.text:004220B3 loc_4220B3: ; CODE XREF: .text:004220D5j

.text:004220B3 8A 10 mov dl, [eax]

.text:004220B5 8A 1E mov bl, [esi]

.text:004220B7 8A CA mov cl, dl

.text:004220B9 3A D3 cmp dl, bl

.text:004220BB 75 1E jnz short loc_4220DB

.text:004220BD 84 C9 test cl, cl

.text:004220BF 74 16 jz short loc_4220D7

.text:004220C1 8A 50 01 mov dl, [eax+1]

.text:004220C4 8A 5E 01 mov bl, [esi+1]

.text:004220C7 8A CA mov cl, dl

.text:004220C9 3A D3 cmp dl, bl

.text:004220CB 75 0E jnz short loc_4220DB

.text:004220CD 83 C0 02 add eax, 2

.text:004220D0 83 C6 02 add esi, 2

.text:004220D3 84 C9 test cl, cl

.text:004220D5 75 DC jnz short loc_4220B3

.text:004220D7

.text:004220D7 loc_4220D7: ; CODE XREF: .text:004220BFj

.text:004220D7 33 C0 xor eax, eax

.text:004220D9 EB 05 jmp short loc_4220E0

.text:004220DB ; ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

.text:004220DB

.text:004220DB loc_4220DB: ; CODE XREF: .text:004220BBj
.text:004220DB ; .text:004220CBj

.text:004220DB 1B C0 sbb eax, eax

.text:004220DD 83 D8 FF sbb eax, 0FFFFFFFFh

.text:004220E0

.text:004220E0 loc_4220E0: ; CODE XREF: .text:004220D9j

.text:004220E0 85 C0 test eax, eax

.text:004220E2 74 43 jz short loc_422127

.text:004220E4 8B 74 24 20 mov esi, [esp+20h]

.text:004220E8 68 60 C4 46 00 push offset ValueName

.text:004220ED 8B CE mov ecx, esi

.text:004220EF E8 6C D3 01 00 call sub_43F460

.text:004220F4 8B 4C 24 28 mov ecx, [esp+28h]

.text:004220F8 68 60 C4 46 00 push offset ValueName

.text:004220FD E8 5E D3 01 00 call sub_43F460
```
