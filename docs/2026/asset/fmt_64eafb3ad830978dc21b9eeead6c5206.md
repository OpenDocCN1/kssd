
```
004896C3    .  51            push ecx
004896C4    .  53            push ebx
004896C5    .  56            push esi
004896C6    .  57            push edi
004896C7    .  8955 F8       mov dword ptr ss:[ebp-8],edx          ;  SerialNo
004896CA    .  8945 FC       mov dword ptr ss:[ebp-4],eax          ;  Name
004896CD    .  8B45 FC       mov eax,dword ptr ss:[ebp-4]
004896D0    .  E8 3BAEF7FF   call UnPacked.00404510
004896D5    .  8B45 F8       mov eax,dword ptr ss:[ebp-8]
004896D8    .  E8 33AEF7FF   call UnPacked.00404510
004896DD    .  33C0          xor eax,eax
004896DF    .  55            push ebp
004896E0    .  68 589B4800   push UnPacked.00489B58
004896E5    .  64:FF30       push dword ptr fs:[eax]
004896E8    .  64:8920       mov dword ptr fs:[eax],esp
004896EB    .  33C0          xor eax,eax
004896ED    .  55            push ebp
004896EE    .  68 0C9B4800   push UnPacked.00489B0C
004896F3    .  64:FF30       push dword ptr fs:[eax]
004896F6    .  64:8920       mov dword ptr fs:[eax],esp
004896F9    .  837D FC 00    cmp dword ptr ss:[ebp-4],0            ;  Name,用户名不能为空
004896FD    .  74 73         je short UnPacked.00489772
004896FF    .  837D F8 00    cmp dword ptr ss:[ebp-8],0            ;  SerialNo，序列号不能为空
00489703    .  74 6D         je short UnPacked.00489772
```

注册信息不为空时在注册表`\Software\SWIFTDOG\PcThrust`中保存注册信息

```
00489705    .  33C0          xor eax,eax
00489707    .  55            push ebp
00489708    .  68 66974800   push UnPacked.00489766
0048970D    .  64:FF30       push dword ptr fs:[eax]
00489710    .  64:8920       mov dword ptr fs:[eax],esp
00489713    .  B2 01         mov dl,1
00489715    .  A1 B01A4200   mov eax,dword ptr ds:[421AB0]
0048971A    .  E8 9184F9FF   call UnPacked.00421BB0
0048971F    .  8BD8          mov ebx,eax
00489721    .  BA 02000080   mov edx,80000002
00489726    .  8BC3          mov eax,ebx
00489728    .  E8 2385F9FF   call UnPacked.00421C50                ;  开始在注册表中保存注册信息
0048972D    .  B1 01         mov cl,1
0048972F    .  BA 749B4800   mov edx,UnPacked.00489B74             ;  ASCII "\Software\SWIFTDOG\PcThrust"
00489734    .  8BC3          mov eax,ebx
00489736    .  E8 7985F9FF   call UnPacked.00421CB4
0048973B    .  8B4D FC       mov ecx,dword ptr ss:[ebp-4]          ;  Name
0048973E    .  BA 989B4800   mov edx,UnPacked.00489B98             ;  ASCII "Name"
00489743    .  8BC3          mov eax,ebx
00489745    .  E8 2687F9FF   call UnPacked.00421E70
0048974A    .  8B4D F8       mov ecx,dword ptr ss:[ebp-8]          ;  SerialNo
0048974D    .  BA A89B4800   mov edx,UnPacked.00489BA8             ;  ASCII "Serial"
00489752    .  8BC3          mov eax,ebx
00489754    .  E8 1787F9FF   call UnPacked.00421E70
00489759    .  33C0          xor eax,eax
0048975B    .  5A            pop edx
0048975C    .  59            pop ecx
0048975D    .  59            pop ecx
0048975E    .  64:8910       mov dword ptr fs:[eax],edx
00489761    .  E9 89000000   jmp UnPacked.004897EF
00489766    .^ E9 C19FF7FF   jmp UnPacked.0040372C
0048976B    .  E8 24A3F7FF   call UnPacked.00403A94
00489770    .  EB 7D         jmp short UnPacked.004897EF
```

注册信息为空时清空注册信息

```
00489772    >  33C0          xor eax,eax
00489774    .  55            push ebp
00489775    .  68 E5974800   push UnPacked.004897E5
0048977A    .  64:FF30       push dword ptr fs:[eax]
0048977D    .  64:8920       mov dword ptr fs:[eax],esp
00489780    .  B2 01         mov dl,1
00489782    .  A1 B01A4200   mov eax,dword ptr ds:[421AB0]
00489787    .  E8 2484F9FF   call UnPacked.00421BB0
0048978C    .  8BD8          mov ebx,eax
0048978E    .  BA 02000080   mov edx,80000002
00489793    .  8BC3          mov eax,ebx
00489795    .  E8 B684F9FF   call UnPacked.00421C50
0048979A    .  C743 18 19000>mov dword ptr ds:[ebx+18],20019
004897A1    .  33C9          xor ecx,ecx
004897A3    .  BA 749B4800   mov edx,UnPacked.00489B74             ;  ASCII "\Software\SWIFTDOG\PcThrust"
004897A8    .  8BC3          mov eax,ebx
004897AA    .  E8 0585F9FF   call UnPacked.00421CB4
004897AF    .  8D4D FC       lea ecx,dword ptr ss:[ebp-4]
004897B2    .  BA 989B4800   mov edx,UnPacked.00489B98             ;  ASCII "Name"
004897B7    .  8BC3          mov eax,ebx
004897B9    .  E8 DE86F9FF   call UnPacked.00421E9C
004897BE    .  8D4D F8       lea ecx,dword ptr ss:[ebp-8]
004897C1    .  BA A89B4800   mov edx,UnPacked.00489BA8             ;  ASCII "Serial"
004897C6    .  8BC3          mov eax,ebx
004897C8    .  E8 CF86F9FF   call UnPacked.00421E9C
004897CD    .  8BC3          mov eax,ebx
004897CF    .  E8 4C84F9FF   call UnPacked.00421C20
004897D4    .  8BC3          mov eax,ebx
004897D6    .  E8 B19AF7FF   call UnPacked.0040328C
004897DB    .  33C0          xor eax,eax
004897DD    .  5A            pop edx
004897DE    .  59            pop ecx
004897DF    .  59            pop ecx
004897E0    .  64:8910       mov dword ptr fs:[eax],edx
004897E3    .  EB 0A         jmp short UnPacked.004897EF
004897E5    .^ E9 429FF7FF   jmp UnPacked.0040372C
004897EA    .  E8 A5A2F7FF   call UnPacked.00403A94
```

开始验证啦

```
004897EF    >  33C0          xor eax,eax
004897F1    .  55            push ebp
004897F2    .  68 F49A4800   push UnPacked.00489AF4
004897F7    .  64:FF30       push dword ptr fs:[eax]
004897FA    .  64:8920       mov dword ptr fs:[eax],esp
004897FD    .  837D FC 00    cmp dword ptr ss:[ebp-4],0            ;  Name
00489801    .  0F84 E3020000 je UnPacked.00489AEA
00489807    .  837D F8 00    cmp dword ptr ss:[ebp-8],0            ;  SerialNo
0048980B    .  0F84 D9020000 je UnPacked.00489AEA
00489811    .  B8 781F4900   mov eax,UnPacked.00491F78
00489816    .  8B55 FC       mov edx,dword ptr ss:[ebp-4]
00489819    .  E8 96A8F7FF   call UnPacked.004040B4
0048981E    .  68 7C1F4900   push UnPacked.00491F7C
00489823    .  8D45 E4       lea eax,dword ptr ss:[ebp-1C]
00489826    .  50            push eax
00489827    .  8B55 F8       mov edx,dword ptr ss:[ebp-8]          ;  SerialNo
0048982A    .  B8 B89B4800   mov eax,UnPacked.00489BB8             ;  '+'
0048982F    .  E8 30AEF7FF   call UnPacked.00404664                ;  System.@LStrPos;
00489834    .  40            inc eax                               ;  分隔字符在序列号中的位置+1
00489835    .  50            push eax                              ;  入栈
00489836    .  8B45 F8       mov eax,dword ptr ss:[ebp-8]          ;  SerialNo
00489839    .  E8 E2AAF7FF   call UnPacked.00404320                ;  System.@LStrLen(String):Integer;
0048983E    .  8BC8          mov ecx,eax                           ;  注册码长度
00489840    .  8B45 F8       mov eax,dword ptr ss:[ebp-8]          ;  SerialNo
00489843    .  5A            pop edx                               ;  出栈
00489844    .  E8 37ADF7FF   call UnPacked.00404580                ;  System.@LStrCopy;
00489849    .  8B55 E4       mov edx,dword ptr ss:[ebp-1C]         ;  取得第一个分隔字符后的序列号
0048984C    .  B8 B89B4800   mov eax,UnPacked.00489BB8             ;  '+'
00489851    .  E8 0EAEF7FF   call UnPacked.00404664                ;  System.@LStrPos;
00489856    .  48            dec eax                               ;  第2个分隔字符在序列号中的位置-1
00489857    .  50            push eax                              ;  入栈
00489858    .  8D45 E0       lea eax,dword ptr ss:[ebp-20]
0048985B    .  50            push eax
0048985C    .  8B55 F8       mov edx,dword ptr ss:[ebp-8]          ;  SerialNo
0048985F    .  B8 B89B4800   mov eax,UnPacked.00489BB8             ;  '+'
00489864    .  E8 FBADF7FF   call UnPacked.00404664                ;  System.@LStrPos;
00489869    .  40            inc eax
0048986A    .  50            push eax
0048986B    .  8B45 F8       mov eax,dword ptr ss:[ebp-8]          ;  SerialNo
0048986E    .  E8 ADAAF7FF   call UnPacked.00404320                ;  System.@LStrLen(String):Integer;
00489873    .  8BC8          mov ecx,eax
00489875    .  8B45 F8       mov eax,dword ptr ss:[ebp-8]          ;  SerialNo
```
