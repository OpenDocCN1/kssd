
```
004B7630    .  8D45 C4               lea eax,dword ptr ss:[ebp-3C]

004B7633    .  50                    push eax

004B7634    .  B9 01000000           mov ecx,1

004B7639    .  BA 03000000           mov edx,3

004B763E    .  8B45 FC               mov eax,dword ptr ss:[ebp-4]

004B7641    .  E8 FECCF4FF           call unpacked.00404344               ;  取用户名第3位

004B7646    .  8B45 C4               mov eax,dword ptr ss:[ebp-3C]

004B7649    .  E8 2AF9FFFF           call unpacked.004B6F78

004B764E    .  25 FF000000           and eax,0FF

004B7653    .  8945 EC               mov dword ptr ss:[ebp-14],eax        ;  保存用户名第3位

004B7656    .  8D45 C0               lea eax,dword ptr ss:[ebp-40]

004B7659    .  50                    push eax

004B765A    .  B9 01000000           mov ecx,1

004B765F    .  BA 04000000           mov edx,4

004B7664    .  8B45 FC               mov eax,dword ptr ss:[ebp-4]

004B7667    .  E8 D8CCF4FF           call unpacked.00404344               ;  取用户名第4位

004B766C    .  8B45 C0               mov eax,dword ptr ss:[ebp-40]

004B766F    .  E8 04F9FFFF           call unpacked.004B6F78

004B7674    .  25 FF000000           and eax,0FF

004B7679    .  8945 E8               mov dword ptr ss:[ebp-18],eax        ;  保存用户名第4位

004B767C    .  8D45 BC               lea eax,dword ptr ss:[ebp-44]

004B767F    .  50                    push eax

004B7680    .  B9 01000000           mov ecx,1

004B7685    .  BA 05000000           mov edx,5

004B768A    .  8B45 FC               mov eax,dword ptr ss:[ebp-4]

004B768D    .  E8 B2CCF4FF           call unpacked.00404344

004B7692    .  8B45 BC               mov eax,dword ptr ss:[ebp-44]

004B7695    .  E8 DEF8FFFF           call unpacked.004B6F78

004B769A    .  25 FF000000           and eax,0FF

004B769F    .  8945 E4               mov dword ptr ss:[ebp-1C],eax        ;  保存用户名第5位

004B76A2    .  8D45 B8               lea eax,dword ptr ss:[ebp-48]

004B76A5    .  50                    push eax

004B76A6    .  B9 01000000           mov ecx,1

004B76AB    .  BA 06000000           mov edx,6

004B76B0    .  8B45 FC               mov eax,dword ptr ss:[ebp-4]

004B76B3    .  E8 8CCCF4FF           call unpacked.00404344

004B76B8    .  8B45 B8               mov eax,dword ptr ss:[ebp-48]

004B76BB    .  E8 B8F8FFFF           call unpacked.004B6F78

004B76C0    .  25 FF000000           and eax,0FF

004B76C5    .  8945 E0               mov dword ptr ss:[ebp-20],eax        ;  保存用户名第6位

004B76C8    .  8D45 B4               lea eax,dword ptr ss:[ebp-4C]

004B76CB    .  50                    push eax

004B76CC    .  B9 01000000           mov ecx,1

004B76D1    .  BA 07000000           mov edx,7

004B76D6    .  8B45 FC               mov eax,dword ptr ss:[ebp-4]

004B76D9    .  E8 66CCF4FF           call unpacked.00404344

004B76DE    .  8B45 B4               mov eax,dword ptr ss:[ebp-4C]

004B76E1    .  E8 92F8FFFF           call unpacked.004B6F78

004B76E6    .  25 FF000000           and eax,0FF

004B76EB    .  8945 DC               mov dword ptr ss:[ebp-24],eax        ;  保存用户名第7位

004B76EE    .  8D45 B0               lea eax,dword ptr ss:[ebp-50]

004B76F1    .  50                    push eax

004B76F2    .  B9 01000000           mov ecx,1

004B76F7    .  BA 08000000           mov edx,8

004B76FC    .  8B45 FC               mov eax,dword ptr ss:[ebp-4]

004B76FF    .  E8 40CCF4FF           call unpacked.00404344

004B7704    .  8B45 B0               mov eax,dword ptr ss:[ebp-50]

004B7707    .  E8 6CF8FFFF           call unpacked.004B6F78

004B770C    .  25 FF000000           and eax,0FF

004B7711    .  8945 D8               mov dword ptr ss:[ebp-28],eax        ;  保存用户名第8位

004B7714    .  33C0                  xor eax,eax

004B7716    .  55                    push ebp

004B7717    .  68 5C774B00           push unpacked.004B775C

004B771C    .  64:FF30               push dword ptr fs:[eax]

004B771F    .  64:8920               mov dword ptr fs:[eax],esp

004B7722    .  8D45 A8               lea eax,dword ptr ss:[ebp-58]

004B7725    .  50                    push eax

004B7726    .  B9 08000000           mov ecx,8

004B772B    .  BA 07000000           mov edx,7

004B7730    .  8B45 F8               mov eax,dword ptr ss:[ebp-8]         ;  试炼码

004B7733    .  E8 0CCCF4FF           call unpacked.00404344               ;  字符串截取函数

004B7738    .  8B4D A8               mov ecx,dword ptr ss:[ebp-58]        ;  取得试炼码第7-14位,共8位

004B773B    .  8D45 AC               lea eax,dword ptr ss:[ebp-54]

004B773E    .  BA 20794B00           mov edx,unpacked.004B7920

004B7743    .  E8 40CAF4FF           call unpacked.00404188

004B7748    .  8B45 AC               mov eax,dword ptr ss:[ebp-54]

004B774B    .  E8 D828F5FF           call unpacked.0040A028

004B7750    .  8BD8                  mov ebx,eax                          ;  转成16进制

004B7752    .  33C0                  xor eax,eax

004B7754    .  5A                    pop edx

004B7755    .  59                    pop ecx

004B7756    .  59                    pop ecx

004B7757    .  64:8910               mov dword ptr fs:[eax],edx

004B775A    .  EB 14                 jmp short unpacked.004B7770

004B775C    .^ E9 13BFF4FF           jmp unpacked.00403674

004B7761    .  E8 6AC2F4FF           call unpacked.004039D0

004B7766    .  E9 57010000           jmp unpacked.004B78C2

004B776B    .  E8 60C2F4FF           call unpacked.004039D0

004B7770    >  2B5D D8               sub ebx,dword ptr ss:[ebp-28]        ;  减去用户名第8位

004B7773    .  8BC3                  mov eax,ebx

004B7775    .  B9 07000000           mov ecx,7                            ;  除7

004B777A    .  33D2                  xor edx,edx

004B777C    .  F7F1                  div ecx

004B777E    .  85D2                  test edx,edx                         ;  不能被7整除,over

004B7780    .  0F85 3C010000         jnz unpacked.004B78C2

004B7786    .  895D A0               mov dword ptr ss:[ebp-60],ebx

004B7789    .  33C0                  xor eax,eax

004B778B    .  8945 A4               mov dword ptr ss:[ebp-5C],eax

004B778E    .  DF6D A0               fild qword ptr ss:[ebp-60]

004B7791    .  D835 24794B00         fdiv dword ptr ds:[4B7924]

004B7797    .  E8 84B4F4FF           call unpacked.00402C20

004B779C    .  8BD8                  mov ebx,eax                          ;  余数传给EBX

004B779E    .  2B5D DC               sub ebx,dword ptr ss:[ebp-24]        ;  减去用户名第7位

004B77A1    .  8BC3                  mov eax,ebx

004B77A3    .  B9 07000000           mov ecx,7                            ;  除7

004B77A8    .  33D2                  xor edx,edx

004B77AA    .  F7F1                  div ecx

004B77AC    .  85D2                  test edx,edx

004B77AE    .  0F85 0E010000         jnz unpacked.004B78C2                ;  不能被7整除，OVER

004B77B4    .  895D A0               mov dword ptr ss:[ebp-60],ebx

004B77B7    .  33C0                  xor eax,eax

004B77B9    .  8945 A4               mov dword ptr ss:[ebp-5C],eax

004B77BC    .  DF6D A0               fild qword ptr ss:[ebp-60]

004B77BF    .  D835 24794B00         fdiv dword ptr ds:[4B7924]

004B77C5    .  E8 56B4F4FF           call unpacked.00402C20

004B77CA    .  8BD8                  mov ebx,eax                          ;  余数传给EBX

004B77CC    .  2B5D E0               sub ebx,dword ptr ss:[ebp-20]        ;  减去用户名第6位

004B77CF    .  8BC3                  mov eax,ebx

004B77D1    .  B9 07000000           mov ecx,7

004B77D6    .  33D2                  xor edx,edx

004B77D8    .  F7F1                  div ecx

004B77DA    .  85D2                  test edx,edx

004B77DC    .  0F85 E0000000         jnz unpacked.004B78C2                ;  不能被7整除，OVER

004B77E2    .  895D A0               mov dword ptr ss:[ebp-60],ebx

004B77E5    .  33C0                  xor eax,eax

004B77E7    .  8945 A4               mov dword ptr ss:[ebp-5C],eax

004B77EA    .  DF6D A0               fild qword ptr ss:[ebp-60]

004B77ED    .  D835 24794B00         fdiv dword ptr ds:[4B7924]
```
