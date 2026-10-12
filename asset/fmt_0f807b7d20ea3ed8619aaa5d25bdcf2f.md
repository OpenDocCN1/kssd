
```
.text:0042F055 or edi, ebp
.text:0042F057 add esi, edi
.text:0042F059 add esi, 6ED9EBA1h ; <suspicious>
.text:0042F05F add [esp+14h+E], esi
.text:0042F063 mov esi, eax_B
.text:0042F065 shr eax_B, 2
.text:0042F068 mov edi, [ebx+4]
.text:0042F06B shl esi, 1Eh
.text:0042F06E xor edi, [ebx+30h]
.text:0042F071 or esi, eax_B
.text:0042F073 xor edi, [ebx+18h]
.text:0042F076 mov eax_B, esi ; R2(A,B,C,D,E,35)
.text:0042F076 ; *********************************第 2.15 次运算 END****
.text:0042F078 mov esi, [ebx+4] ; *********************************第 2.16 次运算开始****
.text:0042F07B xor esi, [ebx+30h]
.text:0042F07E xor edi, [ebx+10h]
.text:0042F081 xor esi, [ebx+18h]
.text:0042F084 xor esi, [ebx+10h]
.text:0042F087 shr edi, 1Fh
.text:0042F08A add esi, esi
.text:0042F08C or esi, edi
.text:0042F08E mov [ebx+10h], esi
.text:0042F091 mov edi, [esp+14h+A]
.text:0042F095 mov ebp, [esp+14h+E]
.text:0042F099 xor edi, eax_B
.text:0042F09B xor edi, edx_C
.text:0042F09D add esi, edi
.text:0042F09F mov edi, [esp+14h+E]
.text:0042F0A3 shl edi, 5
.text:0042F0A6 shr ebp, 1Bh
.text:0042F0A9 or edi, ebp
.text:0042F0AB add esi, edi
.text:0042F0AD mov edi, [esp+14h+A]
.text:0042F0B1 add esi, 6ED9EBA1h ; <suspicious>
.text:0042F0B7 add ecx_D, esi
.text:0042F0B9 mov esi, [esp+14h+A]
.text:0042F0BD shl esi, 1Eh
.text:0042F0C0 mov ebp, ecx_D
.text:0042F0C2 shr edi, 2
.text:0042F0C5 or esi, edi
.text:0042F0C7 mov [esp+14h+A], esi ; R2(E,A,B,C,D,36)
.text:0042F0C7 ; *********************************第 2.16 次运算 END****
.text:0042F0CB mov edi, [ebx+8] ; *********************************第 2.17 次运算开始****
.text:0042F0CE mov esi, [ebx+8]
.text:0042F0D1 xor edi, [ebx+34h]
.text:0042F0D4 xor esi, [ebx+34h]
.text:0042F0D7 xor edi, [ebx+1Ch]
.text:0042F0DA xor esi, [ebx+1Ch]
.text:0042F0DD xor edi, [ebx+14h]
.text:0042F0E0 xor esi, [ebx+14h]
.text:0042F0E3 shr edi, 1Fh
.text:0042F0E6 add esi, esi
.text:0042F0E8 or esi, edi
.text:0042F0EA mov [ebx+14h], esi
.text:0042F0ED mov edi, [esp+14h+E]
.text:0042F0F1 xor edi, [esp+14h+A]
.text:0042F0F5 xor edi, eax_B
.text:0042F0F7 add esi, edi
.text:0042F0F9 mov edi, ecx_D
.text:0042F0FB shl edi, 5
.text:0042F0FE shr ebp, 1Bh
.text:0042F101 or edi, ebp
.text:0042F103 add esi, edi
.text:0042F105 mov edi, [esp+14h+E]
.text:0042F109 add esi, 6ED9EBA1h ; <suspicious>
.text:0042F10F add edx_C, esi
.text:0042F111 mov esi, [esp+14h+E]
.text:0042F115 shl esi, 1Eh
.text:0042F118 mov ebp, edx_C
.text:0042F11A shr edi, 2
.text:0042F11D or esi, edi
.text:0042F11F mov [esp+14h+E], esi ; R2(D,E,A,B,C,37)
.text:0042F11F ; *********************************第 2.17 次运算 END****
.text:0042F123 mov edi, [ebx+0Ch] ; *********************************第 2.18 次运算开始****
.text:0042F126 mov esi, [ebx+0Ch]
.text:0042F129 xor edi, [ebx+38h]
.text:0042F12C xor esi, [ebx+38h]
.text:0042F12F xor edi, [ebx+20h]
.text:0042F132 xor esi, [ebx+20h]
.text:0042F135 xor edi, [ebx+18h]
.text:0042F138 xor esi, [ebx+18h]
.text:0042F13B shr edi, 1Fh
.text:0042F13E add esi, esi
.text:0042F140 or esi, edi
.text:0042F142 mov [ebx+18h], esi
.text:0042F145 mov edi, [esp+14h+E]
.text:0042F149 xor edi, ecx_D
.text:0042F14B xor edi, [esp+14h+A]
.text:0042F14F add esi, edi
.text:0042F151 mov edi, edx_C
.text:0042F153 shl edi, 5
.text:0042F156 shr ebp, 1Bh
.text:0042F159 or edi, ebp
.text:0042F15B add esi, edi
.text:0042F15D mov edi, [ebx+10h]
.text:0042F160 add esi, 6ED9EBA1h ; <suspicious>
.text:0042F166 xor edi, [ebx+3Ch]
.text:0042F169 add eax_B, esi
.text:0042F16B mov esi, ecx_D
.text:0042F16D shr ecx_D, 2
.text:0042F170 mov ebp, eax_B
.text:0042F172 shl esi, 1Eh
.text:0042F175 xor edi, [ebx+24h]
.text:0042F178 or esi, ecx_D
.text:0042F17A xor edi, [ebx+1Ch]
.text:0042F17D mov ecx_D, esi ; R2(C,D,E,A,B,38)
.text:0042F17D ; *********************************第 2.18 次运算 END****
.text:0042F17F mov esi, [ebx+10h] ; *********************************第 2.19 次运算开始****
.text:0042F182 xor esi, [ebx+3Ch]
.text:0042F185 xor esi, [ebx+24h]
.text:0042F188 xor esi, [ebx+1Ch]
.text:0042F18B shr edi, 1Fh
.text:0042F18E add esi, esi
.text:0042F190 or esi, edi
.text:0042F192 mov edi, ecx_D
.text:0042F194 mov [ebx+1Ch], esi
.text:0042F197 xor edi, edx_C
.text:0042F199 xor edi, [esp+14h+E]
.text:0042F19D add esi, edi
.text:0042F19F mov edi, eax_B
.text:0042F1A1 shl edi, 5
.text:0042F1A4 shr ebp, 1Bh
.text:0042F1A7 or edi, ebp
.text:0042F1A9 add esi, edi
.text:0042F1AB add esi, 6ED9EBA1h ; <suspicious>
.text:0042F1B1 add [esp+14h+A], esi
.text:0042F1B5 mov esi, edx_C
.text:0042F1B7 shr edx_C, 2
.text:0042F1BA mov ebp, [ebx+UI32._5]
.text:0042F1BD shl esi, 1Eh
.text:0042F1C0 xor ebp, [ebx+UI32._0]
```
