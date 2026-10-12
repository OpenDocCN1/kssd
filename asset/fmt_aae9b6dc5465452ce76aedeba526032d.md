
`00922293` `3B95 30FFFFFF` `cmp edx,dword ptr ss:[ebp-D0]` `// 遍历链表模块信息，查找自身模块`  

`00922299` `/74 57` `je short 009222F2`  

`0092229B` `|8B85 28FEFFFF` `mov eax,dword ptr ss:[ebp-1D8]`  

`009222A1` `|8985 34FFFFFF` `mov dword ptr ss:[ebp-CC],eax`  

`009222A7` `|8B8D 34FFFFFF` `mov ecx,dword ptr ss:[ebp-CC]`  

`009222AD` `|8B51 18` `mov edx,dword ptr ds:[ecx+18]`  

`009222B0` `|3B95 48FFFFFF` `cmp edx,dword ptr ss:[ebp-B8]` `// 比较是不是自身模块 0x400000`  

`009222B6` `|75 2A` `jnz short 009222E2`  

`009222B8` `|8B85 C0FEFFFF` `mov eax,dword ptr ss:[ebp-140]`  

`009222BE` `|8B8D 48FFFFFF` `mov ecx,dword ptr ss:[ebp-B8]`  

`009222C4` `|0348 10` `add ecx,dword ptr ds:[eax+10]` `// 获得 bound_pe 的入口点，但这个是经过修正的，已经被改为 0x407f30`  

`009222C7` `|8B95 34FFFFFF` `mov edx,dword ptr ss:[ebp-CC]`  

`009222CD` `|894A 1C` `mov dword ptr ds:[edx+1C],ecx` `// 改写原自身的入口点 0x40109c 为 0x407f30`  

`009222D0` `|8B85 34FFFFFF` `mov eax,dword ptr ss:[ebp-CC]` `// eax 保存当前 _LDR_DATA_TABLE_ENTRY 信息`  

`009222D6` `|8B8D C0FEFFFF` `mov ecx,dword ptr ss:[ebp-140]`  

`009222DC` `|8B51 38` `mov edx,dword ptr ds:[ecx+38]` `// SizeOfImage`  

`009222DF` `|8950 20` `mov dword ptr ds:[eax+20],edx` `// 修改 SizeOfImage 为 0xA000，原为 0x19000`  

`009222E2` `|8B85 28FEFFFF` `mov eax,dword ptr ss:[ebp-1D8]`  

`009222E8` `|8B08` `mov ecx,dword ptr ds:[eax]`  

`009222EA` `|898D 28FEFFFF` `mov dword ptr ss:[ebp-1D8],ecx`  

`009222F0` `^|EB 9B` `jmp short 0092228D` `// 如果没有发现继续 next`  

`// 找到，跳出了链表循环`  

`009222F2` `8D95 70FEFFFF` `lea edx,dword ptr ss:[ebp-190]`  

`009222F8` `52` `push edx`  

`009222F9` `8B85 48FFFFFF` `mov eax,dword ptr ss:[ebp-B8]`  

`009222FF` `50` `push eax` `// 压入基址 0x400000`  

`00922300` `E8 2BEDFFFF` `call 00921030`  

`// 填充一些 PE 信息`  

`00921030` `55` `push ebp`  

`00921031` `8BEC` `mov ebp,esp`  

`00921033` `83EC 0C` `sub esp,0C`  

`00921036` `8D45 F8` `lea eax,dword ptr ss:[ebp-8]`  

`00921039` `50` `push eax`  

`0092103A` `8D4D FC` `lea ecx,dword ptr ss:[ebp-4]`  

`0092103D` `51` `push ecx`  

`0092103E` `8D55 F4` `lea edx,dword ptr ss:[ebp-C]`  

`00921041` `52` `push edx`  

`00921042` `8B45 08` `mov eax,dword ptr ss:[ebp+8]`  

`00921045` `50` `push eax`  

`00921046` `E8 D5F4FFFF` `call 00920520` `// 获得 MagicNumber，第一个节的虚拟地址，`  

`0092104B` `8B4D 0C` `mov ecx,dword ptr ss:[ebp+C]`  

`0092104E` `8B55 08` `mov edx,dword ptr ss:[ebp+8]`  

`00921051` `8951 18` `mov dword ptr ds:[ecx+18],edx`  

`00921054` `8B45 0C` `mov eax,dword ptr ss:[ebp+C]`  

`00921057` `8B4D FC` `mov ecx,dword ptr ss:[ebp-4]`  

`0092105A` `8B51 10` `mov edx,dword ptr ds:[ecx+10]`  

`0092105D` `8950 1C` `mov dword ptr ds:[eax+1C],edx` `// 写新的入口地址`  

`00921060` `8B45 0C` `mov eax,dword ptr ss:[ebp+C]`  

`00921063` `8B4D FC` `mov ecx,dword ptr ss:[ebp-4]`  

`00921066` `8B51 38` `mov edx,dword ptr ds:[ecx+38]` `// 写入新的 Size0fImage`  

`00921069` `8950 20` `mov dword ptr ds:[eax+20],edx`  

`0092106C` `8B45 0C` `mov eax,dword ptr ss:[ebp+C]`  

`0092106F` `8B4D F4` `mov ecx,dword ptr ss:[ebp-C]`  

`00921072` `8B51 04` `mov edx,dword ptr ds:[ecx+4]`  

`00921075` `8950 44` `mov dword ptr ds:[eax+44],edx` `// 写入 TimeDateStamp`  

`00921078` `8BE5` `mov esp,ebp`  

`0092107A` `5D` `pop ebp`  

`0092107B` `C2 0800` `retn 8`  

`// 压入导出的 API 列表`  

`00922305` `8B8D CCFEFFFF` `mov ecx,dword ptr ss:[ebp-134]`  

`0092230B` `51` `push ecx`  

`0092230C` `8D95 70FEFFFF` `lea edx,dword ptr ss:[ebp-190]`  

`00922312` `52` `push edx`  

`00922313` `6A 00` `push 0`  

`00922315` `E8 66EDFFFF` `call 00921080`  

`00920A52` `E8 59FCFFFF` `call <get_export_table_address>` `// 查找对应的 Kernel32，ntdll 导出表`  

`00920A57` `8945 EC` `mov dword ptr ss:[ebp-14],eax`  

`00920A5A` `837D EC 00` `cmp dword ptr ss:[ebp-14],0`  

`00920A5E` `75 07` `jnz short 00920A67`  

`00920A60` `33C0` `xor eax,eax`  

`00920A62` `E9 17020000` `jmp 00920C7E`  

`00920A67` `8B55 EC` `mov edx,dword ptr ss:[ebp-14]`  

`00920A6A` `837A 20 00` `cmp dword ptr ds:[edx+20],0`  

`00920A6E` `75 07` `jnz short 00920A77`  

`00920A70` `33C0` `xor eax,eax`  

`// 找到了需要的 API`  

`00920C44` `837D E8 00` `cmp dword ptr ss:[ebp-18],0`  

`00920C48` `74 05` `je short 00920C4F`  

`00920C4A` `8B45 E8` `mov eax,dword ptr ss:[ebp-18]`  

`00920C4D` `EB 2F` `jmp short 00920C7E`  

`00920C4F` `EB 26` `jmp short 00920C77`  

`// 用自己获得的 API 填充自己已经被修改过的 IAT`  

`00920DA2` `E8 99FCFFFF` `call 00920A40` `// 找 (WININET.InternetOpenA) ok`  

`00920DA7` `8B4D E4` `mov ecx,dword ptr ss:[ebp-1C]` `; __krap_.00409100`  

`00920DAA` `8901` `mov dword ptr ds:[ecx],eax`  

`00920DAC` `8B55 E4` `mov edx,dword ptr ss:[ebp-1C]`  

`00920DAF` `833A 00` `cmp dword ptr ds:[edx],0`  

`// 清除工作`  

`00922351` `E8 EA020000` `call 00922640`  

`00922356` `8BE5` `mov esp,ebp`  

`00922358` `5D` `pop ebp`  

`00922359` `C2 1800` `retn 18`  

`009226A3` `8B7D 1C` `mov edi,dword ptr ss:[ebp+1C]`  

`009226A6` `8B75 18` `mov esi,dword ptr ss:[ebp+18]`  

`009226A9` `8B5D 14` `mov ebx,dword ptr ss:[ebp+14]`  

`009226AC` `FF65 F0` `jmp dword ptr ss:[ebp-10]` `// 注意这里，跳向了新的 EOP 处（00407f30）`  

`// 至此二次解密的前期准备工作完毕，下面是解密`  

`// 用 407000 数据按一定的格式去改写 401000 开始的数据`  

`00407F48` `8A06` `mov al,byte ptr ds:[esi]`  

`00407F4A` `46` `inc esi`  

`00407F4B` `8807` `mov byte ptr ds:[edi],al`  

`00407F4D` `47` `inc edi`  

`00407F4E` `01DB` `add ebx,ebx`  

`00407F50` `75 07` `jnz short __krap_.00407F59`  

`00407F52` `8B1E` `mov ebx,dword ptr ds:[esi]`  

`00407F54` `83EE FC` `sub esi,-4`  

`00407F57` `11DB` `adc ebx,ebx`  

`00407F59` `^ 72 ED` `jb short __krap_.00407F48`  

`...`  

`// 从 406000 开始,搜索导入表中函数名称的后面的信息。406420 开始为一块重定位的信息，利用这个重定位信息去修正 401000 中的代码,这里面记录的是要重定位数据的索引。`  

`004063E2` `41 00 01 48 74 74 70 51 75 65 72 79 49 6E 66 6F` `A.HttpQueryInfo`  

`004063F2` `41 00 01 49 6E 74 65 72 6E 65 74 43 6C 6F 73 65` `A.InternetClose`  

`00406402` `48 61 6E 64 6C 65 00 01 49 6E 74 65 72 6E 65 74` `Handle.Internet`  

`00406412` `52 65 61 64 46 69 6C 65 00 00 00 00 00 00 9D 08` `ReadFile......?`  

`00406422` `0F 11 1D 23 0F 0D 06 05 0D 06 15 33 0F 0E 0D 0B` `#..3.`  

`00406432` `07 12 07 27 35 21 A3 0B 34 70 0D 06 28 20 14 11` `'5!?4p.( `  

`00406442` `11 21 28 32 3B 39 09 1D 0B 0C 54 07 16 06 07 07` `!(2;9..T`  

`00406452` `07 07 07 07 13 09 12 06 05 0E 0F 10 0C 30 09 11` `..0.`  

`00406462` `19 15 15 14 26 15 15 14 19 15 15 14 19 15 15 14` `&`  

`00406472` `19 15 15 14 19 15 15 14 19 15 15 14 19 15 15 14`  

`00406482` `19 15 15 14 19 15 15 14 17 10 1B 10 1B 15 15 14`  

`00406492` `10 09 1D 0B 0C 17 0D 2D 2D 0D 27 52 24 18 14 0F` `...--.'R$`  

`//`  

`00408084` `31C0` `xor eax,eax`  

`00408086` `8A07` `mov al,byte ptr ds:[edi]` `// edi -> 00406424`  

`00408088` `47` `inc edi`  

`00408089` `09C0` `or eax,eax`  
