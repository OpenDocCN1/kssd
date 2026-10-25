
```asm
004E1CE2  movzx eax,byte ptr ds:[eax+4]
004E1CE6  add edi,eax      到此取出申请码的后四位，存于edi中
004E1CE8  xor eax,eax
004E1CEA  mov dword ptr ss:[ebp-18],eax
004E1CED  lea edx,dword ptr ss:[ebp-58]
004E1CF0  mov eax,un-GoodN.004E248C            ;  ASCII "718B1C252E2F7E5E8F6328ED65617C95B47DAF"
004E1CF5  call un-GoodN.004DCB68  某种计算，得一数
004E1CFA  lea eax,dword ptr ss:[ebp-58]
004E1CFD  lea edx,dword ptr ss:[ebp-48]
004E1D00  call un-GoodN.004DCBDC  某种变换，得一字符串"AStCa3pYweA0PZWK"
004E1D05  mov bl,20
004E1D07  add dword ptr ss:[ebp-18],9E3719B5
004E1D0E  mov eax,edi
004E1D10  shl eax,4
004E1D13  add esi,eax
004E1D15  mov eax,dword ptr ss:[ebp-3C]
004E1D18  xor eax,edi
004E1D1A  add esi,eax
004E1D1C  mov eax,edi
004E1D1E  shr eax,5
004E1D21  xor eax,dword ptr ss:[ebp-18]
004E1D24  add esi,eax
004E1D26  add esi,dword ptr ss:[ebp-40]
004E1D29  mov eax,esi
004E1D2B  shl eax,4
004E1D2E  add edi,eax
004E1D30  mov eax,dword ptr ss:[ebp-34]
004E1D33  xor eax,esi
004E1D35  add edi,eax
004E1D37  mov eax,esi
004E1D39  shr eax,5
004E1D3C  xor eax,dword ptr ss:[ebp-18]
004E1D3F  add edi,eax
004E1D41  add edi,dword ptr ss:[ebp-38]
004E1D44  dec bl
004E1D46  jnz short un-GoodN.004E1D07  循环对申请码计算
004E1D48  mov eax,esi
004E1D4A  and eax,3FFFFFFF
004E1D4F  add eax,2
004E1D52  xor edx,edx
004E1D54  mov dword ptr ss:[ebp-20],eax  存申请码变换结果
004E1D57  mov dword ptr ss:[ebp-1C],edx  存申请码变换结果
004E1D5A  mov eax,esi
004E1D5C  shr eax,1E
004E1D5F  add eax,24F80050
004E1D64  add eax,2
004E1D67  xor edx,edx
004E1D69  mov dword ptr ss:[ebp-28],eax  存申请码变换结果
004E1D6C  mov dword ptr ss:[ebp-24],edx  存申请码变换结果
004E1D6F  push dword ptr ss:[ebp-1C]           ; /Arg2
004E1D72  push dword ptr ss:[ebp-20]           ; |Arg1
004E1D75  call un-GoodN.004E0E00               ; \un-GoodN.004E0E00 计算
004E1D7A  mov dword ptr ss:[ebp-20],eax  存结果
004E1D7D  mov dword ptr ss:[ebp-1C],edx
004E1D80  push dword ptr ss:[ebp-24]           ; /Arg2
004E1D83  push dword ptr ss:[ebp-28]           ; |Arg1
004E1D86  call un-GoodN.004E0E00               ; \un-GoodN.004E0E00 计算
004E1D8B  mov dword ptr ss:[ebp-28],eax 存结果
004E1D8E  mov dword ptr ss:[ebp-24],edx
004E1D91  lea edx,dword ptr ss:[ebp-58]
004E1D94  mov eax,un-GoodN.004E24BC     ;ASCII "CCB3EF10FF82041FEC05EBCB3B523362992AB75ADD33CF49F"
004E1D99  call un-GoodN.004DCB68  某种计算，得一数
004E1D9E  lea eax,dword ptr ss:[ebp-58]
004E1DA1  lea edx,dword ptr ss:[ebp-5C]
004E1DA4  call un-GoodN.004DCBDC  某种变换，得一字符串 "wsra022ajLlcEEW9"
004E1DA9  push dword ptr ss:[ebp-24]           ; /Arg2
004E1DAC  push dword ptr ss:[ebp-28]           ; |Arg1
004E1DAF  lea edx,dword ptr ss:[ebp-64]        ; |
004E1DB2  mov eax,8                            ; |
004E1DB7  call un-GoodN.00408FBC               ; \un-GoodN.00408FBC 按16进制打印
004E1DBC  mov eax,dword ptr ss:[ebp-64]
004E1DBF  lea edx,dword ptr ss:[ebp-60]
004E1DC2  call un-GoodN.00408B18  某种计算，得一数
004E1DC7  mov eax,dword ptr ss:[ebp-60]
004E1DCA  push eax
004E1DCB  push dword ptr ss:[ebp-1C]           ; /Arg2
004E1DCE  push dword ptr ss:[ebp-20]           ; |Arg1
004E1DD1  lea edx,dword ptr ss:[ebp-6C]        ; |
004E1DD4  mov eax,8                            ; |
004E1DD9  call un-GoodN.00408FBC               ; \un-GoodN.00408FBC 按16进制打印
004E1DDE  mov eax,dword ptr ss:[ebp-6C]
004E1DE1  lea edx,dword ptr ss:[ebp-68]
004E1DE4  call un-GoodN.00408B18  某种计算，得一数
004E1DE9  mov edx,dword ptr ss:[ebp-68]
004E1DEC  lea eax,dword ptr ss:[ebp-2C]
004E1DEF  pop ecx
004E1DF0  call un-GoodN.004045D0  连接字符串
004E1DF5  lea ecx,dword ptr ss:[ebp-70]
004E1DF8  mov eax,dword ptr ss:[ebp-2C]
004E1DFB  mov dl,byte ptr ds:[eax+2]  '6'
004E1DFE  mov eax,dword ptr ss:[ebp-2C]
004E1E01  call un-GoodN.004E29C4  去掉其中的'6'
004E1E06  mov edx,dword ptr ss:[ebp-70]
004E1E09  lea eax,dword ptr ss:[ebp-2C]
004E1E0C  call un-GoodN.0040435C  没看出什么变化
004E1E11  lea ecx,dword ptr ss:[ebp-74]
004E1E14  mov edx,dword ptr ss:[ebp-30]
004E1E17  mov eax,dword ptr ss:[ebp-2C]
004E1E1A  call un-GoodN.004DD8E8  字符串变换
004E1E1F  mov edx,dword ptr ss:[ebp-74]
004E1E22  lea eax,dword ptr ss:[ebp-30]
004E1E25  call un-GoodN.0040435C
004E1E2A  lea ecx,dword ptr ss:[ebp-78]
004E1E2D  mov eax,dword ptr ss:[ebp-30]
004E1E30  mov dl,byte ptr ds:[eax+D]  '6'
004E1E33  mov eax,dword ptr ss:[ebp-30] 字符串 "FC3C63988AAA46B00FF5D5DB8131CD3F"
004E1E36  call un-GoodN.004E29C4  去掉其中的'6'
004E1E3B  mov edx,dword ptr ss:[ebp-78]
004E1E3E  lea eax,dword ptr ss:[ebp-30]
004E1E41  call un-GoodN.0040435C
004E1E46  mov eax,dword ptr ds:[51B81C]
004E1E4B  mov eax,dword ptr ds:[eax]
004E1E4D  mov eax,dword ptr ds:[eax+430]
004E1E53  cmp byte ptr ds:[eax+40],0
004E1E57  je un-GoodN.004E239D
004E1E5D  lea edx,dword ptr ss:[ebp-2C]  申请码变来的数
004E1E60  mov eax,dword ptr ss:[ebp-8]  注册码     "1234523456345674567856789"
004E1E63  call un-GoodN.004E2744  注册码变成 "4231423546354675867586759"
004E1E68  mov eax,dword ptr ss:[ebp-C]
004E1E6B  mov edx,un-GoodN.004E24F8            ;  ASCII "aaa"
004E1E70  call un-GoodN.004046D0
004E1E75  jnz short un-GoodN.004E1E94
004E1E77  mov eax,dword ptr ds:[51BD30]
004E1E7C  mov eax,dword ptr ds:[eax]
004E1E7E  mov ecx,dword ptr ss:[ebp-30]
004E1E81  mov edx,1
004E1E86  mov ebx,dword ptr ds:[eax]
004E1E88  call dword ptr ds:[ebx+20]
004E1E8B  mov byte ptr ss:[ebp-D],1
004E1E8F  jmp un-GoodN.004E239D
004E1E94  lea eax,dword ptr ss:[ebp-80]
004E1E97  push eax
004E1E98  lea eax,dword ptr ss:[ebp-84]
004E1E9E  mov ecx,un-GoodN.004E2504            ;  ASCII "     "
004E1EA3  mov edx,dword ptr ss:[ebp-2C]
004E1EA6  call un-GoodN.004045D0  加5个空格，变成 "4231423546354675867586759     "
004E1EAB  mov eax,dword ptr ss:[ebp-84]
004E1EB1  mov ecx,7
004E1EB6  mov edx,11
004E1EBB  call un-GoodN.004047E4  从第17位开始取7位， "8675867"
004E1EC0  mov eax,dword ptr ss:[ebp-80]  "8675867"
004E1EC3  lea ecx,dword ptr ss:[ebp-7C]
004E1EC6  mov edx,un-GoodN.004E2514     常数"34A5CC48D1AB3"
004E1ECB  call un-GoodN.004DD8E8  字符串变换，得 "9E4165ADC409583C"
004E1ED0  mov eax,dword ptr ss:[ebp-7C]
004E1ED3  push eax
004E1ED4  lea eax,dword ptr ss:[ebp-8C]
004E1EDA  push eax
004E1EDB  lea eax,dword ptr ss:[ebp-90]
004E1EE1  mov ecx,un-GoodN.004E252C            ;  ASCII "2AB75ADD"
004E1EE6  mov edx,dword ptr ss:[ebp-30]  "FC3C3988AAA4B00FF5D5DB8131CD3F"
004E1EE9  call un-GoodN.004045D0  连接成 "FC3C3988AAA4B00FF5D5DB8131CD3F2AB75ADD"
004E1EEE  mov eax,dword ptr ss:[ebp-90]
004E1EF4  mov ecx,7
004E1EF9  mov edx,1
004E1EFE  call un-GoodN.004047E4  取前7位得 "FC3C398"
004E1F03  mov eax,dword ptr ss:[ebp-8C]  ASCII "FC3C398"
004E1F09  lea ecx,dword ptr ss:[ebp-88]
004E1F0F  mov edx,un-GoodN.004E2514            ;  ASCII "34A5CC48D1AB3"
004E1F14  call un-GoodN.004DD8E8  字符串变换，得 "EF9F176831E87F88"
004E1F19  mov edx,dword ptr ss:[ebp-88]  "EF9F176831E87F88"
004E1F1F  pop eax      "9E4165ADC409583C"
004E1F20  call un-GoodN.004046D0  关键比较，字符串比较
004E1F25  jnz un-GoodN.004E2382    关键跳，这儿不跳就会将信息写入注册表
004E1F2B  mov eax,dword ptr ss:[ebp-C]
004E1F2E  mov edx,un-GoodN.004E2540            ;  ASCII "111"
004E1F33  call un-GoodN.004046D0
004E1F38  jnz un-GoodN.004E237C
004E1F3E  mov dl,1
004E1F40  mov eax,dword ptr ds:[43C144]
004E1F45  call un-GoodN.0043C244
004E1F4A  mov dword ptr ss:[ebp-14],eax
004E1F4D  xor eax,eax
004E1F4F  push ebp
004E1F50  push un-GoodN.004E2375
004E1F55  push dword ptr fs:[eax]
004E1F58  mov dword ptr fs:[eax],esp
004E1F5B  mov edx,80000002
004E1F60  mov eax,dword ptr ss:[ebp-14]
004E1F63  call un-GoodN.0043C2E4
004E1F68  mov cl,1
004E1F6A  mov edx,un-GoodN.004E254C            ;  ASCII "Software\GoodSoft\GoodName"
004E1F6F  mov eax,dword ptr ss:[ebp-14]
004E1F72  call un-GoodN.0043C34C
004E1F77  test al,al
004E1F79  jnz short un-GoodN.004E1F85
004E1F7B  call un-GoodN.00403D70
004E1F80  jmp un-GoodN.004E239D
004E1F85  lea eax,dword ptr ss:[ebp-98]
004E1F8B  call un-GoodN.004E26B0
004E1F90  mov eax,dword ptr ss:[ebp-98]
```
