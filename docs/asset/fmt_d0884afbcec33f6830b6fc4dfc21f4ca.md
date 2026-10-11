
0045A686 `<>|.` `E8 AD8BFDFF` `call Unpacked.00433238` `; ->Controls.TControl.SetText(TControl;TCaption);`  

0045A68B `|. ` `8D55 F4` `lea edx,dword ptr ss:[ebp-C]`  

0045A68E `<>|.` `8B83 04030000` `mov eax,dword ptr ds:[ebx+304]` `; *RCode:N.A.`  

0045A694 `<>|.` `E8 6F8BFDFF` `call Unpacked.00433208` `; ->Controls.TControl.GetText(TControl):TCaption;`  

0045A699 `|. ` `8B45 F4` `mov eax,dword ptr ss:[ebp-C]`  

0045A69C `|. ` `BA 01000000` `mov edx,1`  

0045A6A1 `|. ` `E8 7EB6FFFF` `call Unpacked.00455D24` `; 判断注册`  

0045A6A6 `|. ` `84C0` `test al,al`  

0045A6A8 `|. ` `74 26` `je short Unpacked.0045A6D0` `; 判断`  

0045A6AA `|. ` `33D2` `xor edx,edx`  

0045A6AC `<>|.` `8B83 04030000` `mov eax,dword ptr ds:[ebx+304]` `; *RCode:N.A.`  

0045A6B2 `|. ` `8B08` `mov ecx,dword ptr ds:[eax]`  

0045A6B4 `|. ` `FF51 64` `call dword ptr ds:[ecx+64]`  

0045A6B7 `|. ` `33D2` `xor edx,edx`  

0045A6B9 `<>|.` `8B83 00030000` `mov eax,dword ptr ds:[ebx+300]` `; *MCode:N.A.`  

0045A6BF `|. ` `8B08` `mov ecx,dword ptr ds:[eax]`  

0045A6C1 `|. ` `FF51 64` `call dword ptr ds:[ecx+64]`  

0045A6C4 `|. ` `BA 18A74500` `mov edx,Unpacked.0045A718` `; '软件已经注册' ,马脚露了!!`  

0045A6C9 `|. ` `8BC3` `mov eax,ebx`  

0045A6CB `<>|.` `E8 688BFDFF` `call Unpacked.00433238` `; ->Controls.TControl.SetText(TControl;TCaption);`  

0045A6D0 `|> ` `33C0` `xor eax,eax`  

0045A6D2 `|. ` `5A` `pop edx`  

0045A6D3 `|. ` `59` `pop ecx`  

0045A6D4 `|. ` `59` `pop ecx`  

从上面代码很容易看出 `0045A6A1` 处 `call Unpacked.00455D24` 是关键，跟进:

`---------------------------------------------------------`

00455D24 `$ ` `55` `push ebp`  

00455D25 `. ` `8BEC` `mov ebp,esp`  

00455D27 `. ` `83C4 E0` `add esp,-20`  

00455D2A `. ` `53` `push ebx`  

00455D2B `. ` `33C9` `xor ecx,ecx`  

00455D2D `. ` `894D E0` `mov dword ptr ss:[ebp-20],ecx`  

00455D30 `. ` `894D E4` `mov dword ptr ss:[ebp-1C],ecx`  

00455D33 `. ` `894D F0` `mov dword ptr ss:[ebp-10],ecx`  

00455D36 `. ` `8BDA` `mov ebx,edx`  

00455D38 `. ` `8945 FC` `mov dword ptr ss:[ebp-4],eax` `; 试炼码`  

00455D3B `. ` `8B45 FC` `mov eax,dword ptr ss:[ebp-4]`  

00455D3E `. ` `E8 09EBFAFF` `call Unpacked.0040484C`  

00455D43 `. ` `33C0` `xor eax,eax`  

00455D45 `. ` `55` `push ebp`  

00455D46 `. ` `68 055E4500` `push Unpacked.00455E05`  

00455D4B `. ` `64:FF30` `push dword ptr fs:[eax]`  

00455D4E `. ` `64:8920` `mov dword ptr fs:[eax],esp`  

00455D51 `. ` `C645 FB 00` `mov byte ptr ss:[ebp-5],0`  

00455D55 `. ` `837D FC 00` `cmp dword ptr ss:[ebp-4],0` `; 试炼码不能为空`  

00455D59 `. ` `74 7F` `je short Unpacked.00455DDA`  

00455D5B `. ` `8D45 F0` `lea eax,dword ptr ss:[ebp-10]`  

00455D5E `. ` `50` `push eax`  

00455D5F `. ` `8D45 E4` `lea eax,dword ptr ss:[ebp-1C]`  

00455D62 `. ` `E8 B5FDFFFF` `call Unpacked.00455B1C`  

00455D67 `. ` `8B45 E4` `mov eax,dword ptr ss:[ebp-1C]` `; 机器码`  

00455D6A `. ` `8D4D EC` `lea ecx,dword ptr ss:[ebp-14]`  

00455D6D `. ` `8D55 E8` `lea edx,dword ptr ss:[ebp-18]`  

00455D70 `. ` `2BCA` `sub ecx,edx`  

00455D72 `. ` `49` `dec ecx`  

00455D73 `. ` `8BD3` `mov edx,ebx`  

00455D75 `. ` `E8 42EBFAFF` `call Unpacked.004048BC` `; System.@LStrCopy;`  

00455D7A `. ` `33C9` `xor ecx,ecx`  

00455D7C `. ` `B2 01` `mov dl,1`  

00455D7E `. ` `A1 48554500` `mov eax,dword ptr ds:[455548]`  

00455D83 `. ` `E8 DC58FCFF` `call Unpacked.0041B664` `; TComponent.Create`  

00455D88 `. ` `8945 F4` `mov dword ptr ss:[ebp-C],eax`  

00455D8B `. ` `33C0` `xor eax,eax`  

00455D8D `. ` `55` `push ebp`  

00455D8E `. ` `68 D35D4500` `push Unpacked.00455DD3`  

00455D93 `. ` `64:FF30` `push dword ptr fs:[eax]`  

00455D96 `. ` `64:8920` `mov dword ptr fs:[eax],esp`  

00455D99 `. ` `8D45 E0` `lea eax,dword ptr ss:[ebp-20]`  

00455D9C `. ` `50` `push eax`  

00455D9D `. ` `B1 03` `mov cl,3` `; cl=3 重要数据,决定下面一固定字符串来源`  

00455D9F `. ` `8B55 F0` `mov edx,dword ptr ss:[ebp-10]`  

00455DA2 `. ` `8B45 F4` `mov eax,dword ptr ss:[ebp-C]`  

00455DA5 `. ` `E8 C6FBFFFF` `call Unpacked.00455970` `; 关键,根据机器码产生注册码`  

00455DAA `. ` `8B45 E0` `mov eax,dword ptr ss:[ebp-20]` `; 注册码`  

00455DAD `. ` `8B55 FC` `mov edx,dword ptr ss:[ebp-4]` `; 试炼码`  

00455DB0 `. ` `E8 EBEbFAFF` `call Unpacked.004049A0` `; 明码比较`  

00455DB5 `. ` `85C0` `test eax,eax`  

00455DB7 `. ` `7E 04` `jle short Unpacked.00455DBD`  

00455DB9 `. ` `C645 FB 01` `mov byte ptr ss:[ebp-5],1`  

00455DBD `> ` `33C0` `xor eax,eax`  

00455DBF `. ` `5A` `pop edx`  

00455DC0 `. ` `59` `pop ecx`  

00455DC1 `. ` `59` `pop ecx`  

00455DC2 `. ` `64:8910` `mov dword ptr fs:[eax],edx`  

00455DC5 `. ` `68 DA5D4500` `push Unpacked.00455DDA`  

00455DCA `> ` `8B45 F4` `mov eax,dword ptr ss:[ebp-C]`  

00455DCD `. ` `E8 6ED8FAFF` `call Unpacked.00403640`  

00455DD2 `. ` `C3` `retn`  

00455DD3 `.^ ` `E9 FCDFFAFF` `jmp Unpacked.00403DD4`  

00455DD8 `.^ ` `EB F0` `jmp short Unpacked.00455DCA`  

00455DDA `> ` `33C0` `xor eax,eax`  

00455DDC `. ` `5A` `pop edx`  

00455DDD `. ` `59` `pop ecx`  

00455DDE `. ` `59` `pop ecx`  

00455DDF `. ` `64:8910` `mov dword ptr fs:[eax],edx`  

00455DE2 `. ` `68 0C5E4500` `push Unpacked.00455E0C`  

00455DE7 `> ` `8D45 E0` `lea eax,dword ptr ss:[ebp-20]`  

00455DEA `. ` `BA 02000000` `mov edx,2`  

00455DEF `. ` `E8 DCE5FAFF` `call Unpacked.004043D0`  

00455DF4 `. ` `8D45 F0` `lea eax,dword ptr ss:[ebp-10]`  

00455DF7 `. ` `E8 B0E5FAFF` `call Unpacked.004043AC`  

00455DFC `. ` `8D45 FC` `lea eax,dword ptr ss:[ebp-4]`  

00455DFF `. ` `E8 A8E5FAFF` `call Unpacked.004043AC`  

00455E04 `. ` `C3` `retn`  

00455E05 `.^ ` `E9 CADFFAFF` `jmp Unpacked.00403DD4`  

00455E0A `.^ ` `EB DB` `jmp short Unpacked.00455DE7`  

00455E0C `. ` `8A45 FB` `mov al,byte ptr ss:[ebp-5]`  

00455E0F `. ` `5B` `pop ebx`  

00455E10 `. ` `8BE5` `mov esp,ebp`  

00455E12 `. ` `5D` `pop ebp`  

00455E13 `. ` `C3` `retn`  

`---------------------------------------------------------`

哈哈，又是一个明码比较，还是看看 `00455DA5` `call Unpacked.00455970` 的注册码的运算过程吧:

`---------------------------------------------------------`

00455970 `$ ` `55` `push ebp`  

00455971 `. ` `8BEC` `mov ebp,esp`  

00455973 `. ` `83C4 E4` `add esp,-1C`  

00455976 `. ` `53` `push ebx`  

00455977 `. ` `56` `push esi`  

00455978 `. ` `57` `push edi`  

00455979 `. ` `33DB` `xor ebx,ebx`  

0045597B `. ` `895D E8` `mov dword ptr ss:[ebp-18],ebx`  

0045597E `. ` `895D E4` `mov dword ptr ss:[ebp-1C],ebx`  
