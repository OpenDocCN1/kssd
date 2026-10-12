
```
0051D1E7   .  8D45 B0       lea eax,dword ptr ss:[ebp-50]   ; |  
0051D1EA   .  E8 45DCEEFF   call Down.0040AE34               ; \Down.0040AE34  
0051D1EF   .  FF75 B0       push dword ptr ss:[ebp-50]  
0051D1F2   .  68 DCD35100   push Down.0051D3DC               ;   天)  
0051D1F7   .  8D45 B4       lea eax,dword ptr ss:[ebp-4C]  
0051D1FA   .  BA 04000000   mov edx,4  
0051D1FF   .  E8 087CEEFF   call Down.00404E0C  
0051D204   .  8B55 B4       mov edx,dword ptr ss:[ebp-4C]  
0051D207   .  8B45 FC       mov eax,dword ptr ss:[ebp-4]  
0051D20A   .  E8 55CAF4FF   call Down.00469C64  
0051D20F   .  DD45 D0       fld qword ptr ss:[ebp-30]  
0051D212   .  D81D B4D35100 fcomp dword ptr ds:[51D3B4]  
0051D218   .  DFE0          fstsw ax  
0051D21A   .  9E            sahf  

………  

………  

………  

0051D285   .  E8 2678EEFF   call Down.00404AB0  
0051D28A   .  C3            retn  

*******************************************************************

跟进算法 call，来到这里

0051AC90  /$  55            push ebp  
0051AC91  |.  8BEC          mov ebp,esp  
0051AC93  |.  51            push ecx  
0051AC94  |.  B9 04000000   mov ecx,4  
0051AC99  |>  6A 00         /push 0  
0051AC9B  |.  6A 00         |push 0  
0051AC9D  |.  49            |dec ecx  
0051AC9E  |.^ 75 F9         \jnz short Down.0051AC99  
0051ACA0  |.  51            push ecx  
0051ACA1  |.  874D FC       xchg dword ptr ss:[ebp-4],ecx  
0051ACA4  |.  53            push ebx  
0051ACA5  |.  56            push esi  
0051ACA6  |.  57            push edi  
0051ACA7  |.  8BF9          mov edi,ecx  
0051ACA9  |.  8955 FC       mov dword ptr ss:[ebp-4],edx     ;  ebp-4=tigerisme  
0051ACAC  |.  8B45 FC       mov eax,dword ptr ss:[ebp-4]     ;  eax=tigerisme  
0051ACAF  |.  E8 88A2EEFF   call Down.00404F3C  
0051ACB4  |.  33C0          xor eax,eax                     ;  eax 清零  
0051ACB6  |.  55            push ebp  
0051ACB7  |.  68 51AE5100   push Down.0051AE51  
0051ACBC  |.  64:FF30       push dword ptr fs:[eax]  
0051ACBF  |.  64:8920       mov dword ptr fs:[eax],esp  
0051ACC2  |.  8BC7          mov eax,edi  
0051ACC4  |.  E8 C39DEEFF   call Down.00404A8C  
0051ACC9  |.  8B45 FC       mov eax,dword ptr ss:[ebp-4]     ;  eax=tigerisme  
0051ACCC  |.  E8 7BA0EEFF   call Down.00404D4C  
0051ACD1  |.  8BF0          mov esi,eax                     ;  注册名位数 9 送 esi  
0051ACD3  |.  85F6          test esi,esi  
0051ACD5  |.  7E 26         jle short Down.0051ACFD  
0051ACD7  |.  BB 01000000   mov ebx,1  
0051ACDC  |>  8D4D EC       /lea ecx,dword ptr ss:[ebp-14]  
0051ACDF  |.  8B45 FC       |mov eax,dword ptr ss:[ebp-4]    ;  tigerisme 送 eax  
0051ACE2  |.  0FB64418 FF   |movzx eax,byte ptr ds:[eax+ebx] ;  逐位将注册名送 eax  
0051ACE7  |.  33D2          |xor edx,edx  
0051ACE9  |.  E8 32EAEEFF   |call Down.00409720              ;  将注册名逐位字符转成十六进制  
0051ACEE  |.  8B55 EC       |mov edx,dword ptr ss:[ebp-14]  
0051ACF1  |.  8D45 F8       |lea eax,dword ptr ss:[ebp-8]  
0051ACF4  |.  E8 5BA0EEFF   |call Down.00404D54  
0051ACF9  |.  43            |inc ebx  
0051ACFA  |.  4E            |dec esi                         ;  esi-1  
0051ACFB  |.^ 75 DF         \jnz short Down.0051ACDC  
0051ACFD  |>  8B45 F8       mov eax,dword ptr ss:[ebp-8]     ;  最后形成"746967657269736D65"记做 codeA，即将注册名逐位转成十六进制的 ascii 码  
0051AD00  |.  E8 47A0EEFF   call Down.00404D4C  
0051AD05  |.  8BF0          mov esi,eax                     ;  eax=12（十六进制），即 codeA 的位数  
0051AD07  |.  85F6          test esi,esi  
0051AD09  |.  7E 2C         jle short Down.0051AD37  
0051AD0B  |.  BB 01000000   mov ebx,1  
0051AD10  |>  8B45 F8       /mov eax,dword ptr ss:[ebp-8]    ;  codeA=ebp-8  
0051AD13  |.  E8 34A0EEFF   |call Down.00404D4C              ;  eax=codeA  
0051AD18  |.  2BC3          |sub eax,ebx                     ;  eax=12（十六进制），循环减 1  
0051AD1A  |.  8B55 F8       |mov edx,dword ptr ss:[ebp-8]    ;  codeA 送 edx  
0051AD1D  |.  8A1402        |mov dl,byte ptr ds:[edx+eax]    ;  反向将 codeA 逐位送 dl  
0051AD20  |.  8D45 E8       |lea eax,dword ptr ss:[ebp-18]  
0051AD23  |.  E8 4C9FEEFF   |call Down.00404C74  
0051AD28  |.  8B55 E8       |mov edx,dword ptr ss:[ebp-18]  
0051AD2B  |.  8D45 F4       |lea eax,dword ptr ss:[ebp-C]  
0051AD2E  |.  E8 21A0EEFF   |call Down.00404D54  
0051AD33  |.  43            |inc ebx  
0051AD34  |.  4E            |dec esi                         ;  esi-1  
0051AD35  |.^ 75 D9         \jnz short Down.0051AD10  
0051AD37  |>  8D45 F8       lea eax,dword ptr ss:[ebp-8]  
0051AD3A  |.  50            push eax  
0051AD3B  |.  B9 04000000   mov ecx,4  
0051AD40  |.  BA 01000000   mov edx,1  
0051AD45  |.  8B45 F4       mov eax,dword ptr ss:[ebp-C]     ;  ebp-c="56D637962756769647"出现，即上面的 codeA 倒转的结果，记做 codeB  
0051AD48  |.  E8 5FA2EEFF   call Down.00404FAC  
0051AD4D  |.  8D45 F4       lea eax,dword ptr ss:[ebp-C]  
0051AD50  |.  50            push eax  
0051AD51  |.  B9 04000000   mov ecx,4  
0051AD56  |.  BA 05000000   mov edx,5  
0051AD5B  |.  8B45 F4       mov eax,dword ptr ss:[ebp-C]     ;  codeB 送 eax  
0051AD5E  |.  E8 49A2EEFF   call Down.00404FAC  
0051AD63  |.  8B45 F8       mov eax,dword ptr ss:[ebp-8]     ;  codeB 的前 4 位，即 56D6 送 eax  
0051AD66  |.  E8 E19FEEFF   call Down.00404D4C  
0051AD6B  |.  83F8 04       cmp eax,4  
0051AD6E  |.  7D 2F         jge short Down.0051AD9F  
0051AD70  |.  8B45 F8       mov eax,dword ptr ss:[ebp-8]  
0051AD73  |.  E8 D49FEEFF   call Down.00404D4C  
0051AD78  |.  8BD8          mov ebx,eax  
0051AD7A  |.  83FB 03       cmp ebx,3  
0051AD7D  |.  7F 20         jg short Down.0051AD9F  
0051AD7F  |>  8D4D E4       /lea ecx,dword ptr ss:[ebp-1C]  
0051AD82  |.  8BC3          |mov eax,ebx  
0051AD84  |.  C1E0 02       |shl eax,2  
0051AD87  |.  33D2          |xor edx,edx  
0051AD89  |.  E8 92E9EEFF   |call Down.00409720  
0051AD8E  |.  8B55 E4       |mov edx,dword ptr ss:[ebp-1C]  
0051AD91  |.  8D45 F8       |lea eax,dword ptr ss:[ebp-8]  
0051AD94  |.  E8 BB9FEEFF   |call Down.00404D54  
0051AD99  |.  43            |inc ebx  
0051AD9A  |.  83FB 04       |cmp ebx,4  
0051AD9D  |.^ 75 E0         \jnz short Down.0051AD7F  
0051AD9F  |>  8B45 F4       mov eax,dword ptr ss:[ebp-C]     ;  codeB 的第 4-8 位，即 3796 送 eax  
0051ADA2  |.  E8 A59FEEFF   call Down.00404D4C  
0051ADA7  |.  83F8 04       cmp eax,4  
0051ADAA  |.  7D 2F         jge short Down.0051ADDB  
0051ADAC  |.  8B45 F4       mov eax,dword ptr ss:[ebp-C]  
0051ADAF  |.  E8 989FEEFF   call Down.00404D4C  
0051ADB4  |.  8BD8          mov ebx,eax  
0051ADB6  |.  83FB 03       cmp ebx,3  
0051ADB9  |.  7F 20         jg short Down.0051ADDB  
0051ADBB  |>  8D4D E0       /lea ecx,dword ptr ss:[ebp-20]  
0051ADBE  |.  8BC3          |mov eax,ebx  
0051ADC0  |.  C1E0 02       |shl eax,2  
0051ADC3  |.  33D2          |xor edx,edx  
0051ADC5  |.  E8 56E9EEFF   |call Down.00409720  
0051ADCA  |.  8B55 E0       |mov edx,dword ptr ss:[ebp-20]  
0051ADCD  |.  8D45 F4       |lea eax,dword ptr ss:[ebp-C]  
0051ADD0  |.  E8 7F9FEEFF   |call Down.00404D54  
0051ADD5  |.  43            |inc ebx  
0051ADD6  |.  83FB 04       |cmp ebx,4  
0051ADD9  |.^ 75 E0         \jnz short Down.0051ADBB  
0051ADDB  |>  8D45 F0       lea eax,dword ptr ss:[ebp-10]  
0051ADDE  |.  BA 68AE5100   mov edx,Down.0051AE68            ;  pic4ei8espr，固定值，计算注册码需要用到  
0051ADE3  |.  E8 3C9DEEFF   call Down.00404B24  
0051ADE8  |.  8D45 DC       lea eax,dword ptr ss:[ebp-24]  
0051ADEB  |.  50            push eax  
0051ADEC  |.  B9 04000000   mov ecx,4  
0051ADF1  |.  BA 01000000   mov edx,1  
0051ADF6  |.  8B45 F0       mov eax,dword ptr ss:[ebp-10]  
0051ADF9  |.  E8 AEA1EEFF   call Down.00404FAC  
```
