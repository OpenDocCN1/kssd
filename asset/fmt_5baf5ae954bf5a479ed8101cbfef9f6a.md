
`006639AC` `.` `880411` `mov byte ptr ds:[ecx+edx],al` ; `buf3` 第一位填 `0x80`

`006639AF` `.` `42` `inc edx` ; 此值固定为 `1`

`006639B0` `.` `3BD3` `cmp edx,ebx`

`006639B2` `.` `77 1F` `ja short PDFEdit.006639D3` ; 跳转始终不实现

`006639B4` `.` `8B7E 18` `mov edi,dword ptr ds:[esi+18]` ; 小于

`006639B7` `.` `8BCB` `mov ecx,ebx` ; `0x38`

`006639B9` `.` `2BCA` `sub ecx,edx`

`006639BB` `.` `03FA` `add edi,edx`

`006639BD` `.` `8BD1` `mov edx,ecx`

`006639BF` `.` `33C0` `xor eax,eax`

`006639C1` `.` `C1E9 02` `shr ecx,2`

`006639C4` `.` `F3:AB` `rep stos dword ptr es:[edi]` ; `buf3` 以后各位填 `0`

`006639C6` `.` `8BCA` `mov ecx,edx`

`006639C8` `.` `83E1 03` `and ecx,3`

`006639CB` `.` `F3:AA` `rep stos byte ptr es:[edi]`

`006639CD` `.` `5F` `pop edi`

`006639CE` `.` `5E` `pop esi`

`006639CF` `.` `5B` `pop ebx`

`006639D0` `.` `C2 0800` `retn 8`

可见同样调用了核心计算函数 `call PDFEdit.00665340`

`00665340` `/$` `83EC 3C` `sub esp,3C` ; 关键计算

`00665343` `.` `8B4424 40` `mov eax,dword ptr ss:[esp+40]` ; 取第二参数 `tempBuf`

`00665347` `.` `53` `push ebx`

`00665348` `.` `55` `push ebp`

`00665349` `.` `56` `push esi`

`0066534A` `.` `8B50 0C` `mov edx,dword ptr ds:[eax+C]`

`0066534D` `.` `8B70 08` `mov esi,dword ptr ds:[eax+8]`

`00665350` `.` `8B08` `mov ecx,dword ptr ds:[eax]`

`00665352` `.` `8BEA` `mov ebp,edx`

`00665354` `.` `57` `push edi`

`00665355` `.` `8B78 04` `mov edi,dword ptr ds:[eax+4]`

`00665358` `.` `8B4424 54` `mov eax,dword ptr ss:[esp+54]` ; 取第二参数 `tempBuf`

`0066535C` `.` `33EE` `xor ebp,esi`

`0066535E` `.` `23EF` `and ebp,edi`

`00665360` `.` `8B18` `mov ebx,dword ptr ds:[eax]`

`00665362` `.` `33EA` `xor ebp,edx`

`00665364` `.` `03EB` `add ebp,ebx`

`00665366` `.` `895C24 54` `mov dword ptr ss:[esp+54],ebx`

`0066536A` `.` `8B58 04` `mov ebx,dword ptr ds:[eax+4]`

`0066536D` `.` `8D8C29 78A46AD7` `lea ecx,dword ptr ds:[ecx+ebp+D76AA478]`

`00665374` `.` `8BEE` `mov ebp,esi`

`00665376` `.` `C1C1 07` `rol ecx,7`

`00665379` `.` `33EF` `xor ebp,edi`

`0066537B` `.` `03CF` `add ecx,edi`

`0066537D` `.` `23E9` `and ebp,ecx`

`0066537F` `.` `895C24 20` `mov dword ptr ss:[esp+20],ebx`

`00665383` `.` `33EE` `xor ebp,esi`

`00665385` `.` `03EB` `add ebp,ebx`

`00665387` `.` `8B58 08` `mov ebx,dword ptr ds:[eax+8]`

`0066538A` `.` `895C24 40` `mov dword ptr ss:[esp+40],ebx`

`0066538E` `.` `8D942A 56B7C7E8` `lea edx,dword ptr ds:[edx+ebp+E8C7B756]`

`00665395` `.` `8BEF` `mov ebp,edi`

`00665397` `.` `C1C2 0C` `rol edx,0C`

`0066539A` `.` `03D1` `add edx,ecx`

`0066539C` `33E9` `xor ebp,ecx`

`0066539E` `.` `23EA` `and ebp,edx`

`006653A0` `.` `33EF` `xor ebp,edi`

`006653A2` `.` `03EB` `add ebp,ebx`

`006653A4` `.` `8B58 0C` `mov ebx,dword ptr ds:[eax+C]`

`006653A7` `.` `895C24 18` `mov dword ptr ss:[esp+18],ebx`

`006653AB` `.` `8DB42E DB702024` `lea esi,dword ptr ds:[esi+ebp+242070DB]`

`006653B2` `.` `8BEA` `mov ebp,edx`

`006653B4` `.` `C1C6 11` `rol esi,11`

`006653B7` `.` `03F2` `add esi,edx`

`006653B9` `.` `33E9` `xor ebp,ecx`

`006653BB` `.` `23EE` `and ebp,esi`

`006653BD` `.` `33E9` `xor ebp,ecx`

`006653BF` `.` `03EB` `add ebp,ebx`

`006653C1` `.` `8B58 10` `mov ebx,dword ptr ds:[eax+10]`

`006653C4` `.` `895C24 30` `mov dword ptr ss:[esp+30],ebx`

`006653C8` `.` `8DBC2F EECEBDC1` `lea edi,dword ptr ds:[edi+ebp+C1BDCEEE]`

`006653CF` `.` `8BEA` `mov ebp,edx`

`006653D1` `.` `C1C7 16` `rol edi,16`

`006653D4` `.` `03FE` `add edi,esi`

`006653D6` `.` `33EE` `xor ebp,esi`

`006653D8` `.` `23EF` `and ebp,edi`

`006653DA` `.` `33EA` `xor ebp,edx`

`006653DC` `.` `03EB` `add ebp,ebx`

`006653DE` `.` `8B58 14` `mov ebx,dword ptr ds:[eax+14]`

`006653E1` `.` `895C24 14` `mov dword ptr ss:[esp+14],ebx`

`006653E5` `.` `8D8C29 AF0F7CF5` `lea ecx,dword ptr ds:[ecx+ebp+F57C0FAF]`

`006653EC` `.` `8BEE` `mov ebp,esi`

`006653EE` `.` `C1C1 07` `rol ecx,7`

`006653F1` `.` `03CF` `add ecx,edi`

`006653F3` `.` `33EF` `xor ebp,edi`

`006653F5` `.` `23E9` `and ebp,ecx`

`006653F7` `.` `33EE` `xor ebp,esi`

`006653F9` `.` `03EB` `add ebp,ebx`

`006653FB` `.` `8B58 18` `mov ebx,dword ptr ds:[eax+18]`

`006653FE` `.` `895C24 28` `mov dword ptr ss:[esp+28],ebx`

`00665402` `.` `8D942A 2AC68747` `lea edx,dword ptr ds:[edx+ebp+4787C62A]`

`00665409` `.` `8BEF` `mov ebp,edi`

`0066540B` `.` `C1C2 0C` `rol edx,0C`

`0066540E` `.` `03D1` `add edx,ecx`

`00665410` `.` `33E9` `xor ebp,ecx`

`00665412` `.` `23EA` `and ebp,edx`

`00665414` `.` `33EF` `xor ebp,edi`

`00665416` `.` `03EB` `add ebp,ebx`

`00665418` `.` `8B58 1C` `mov ebx,dword ptr ds:[eax+1C]`

`0066541B` `.` `8DB42E 134630A8` `lea esi,dword ptr ds:[esi+ebp+A8304613]`

`00665422` `.` `C1C6 11` `rol esi,11`

`00665425` `.` `03F2` `add esi,edx`

`00665427` `.` `8BEA` `mov ebp,edx`

`00665429` `.` `895C24 10` `mov dword ptr ss:[esp+10],ebx`

`0066542D` `.` `33E9` `xor ebp,ecx`

`0066542F` `.` `23EE` `and ebp,esi`

`00665431` `.` `33E9` `xor ebp,ecx`

`00665433` `.` `03EB` `add ebp,ebx`

`00665435` `.` `8B58 20` `mov ebx,dword ptr ds:[eax+20]`

`00665438` `.` `895C24 24` `mov dword ptr ss:[esp+24],ebx`

`0066543C` `.` `8DBC2F 019546FD` `lea edi,dword ptr ds:[edi+ebp+FD469501]`

`00665443` `.` `8BEA` `mov ebp,edx`

`00665445` `.` `C1C7 16` `rol edi,16`

`00665448` `.` `33EE` `xor ebp,esi`

`0066544A` `.` `03FE` `add edi,esi`

`0066544C` `.` `23EF` `and ebp,edi`

`0066544E` `.` `33EA` `xor ebp,edx`

`00665450` `.` `03EB` `add ebp,ebx`

`00665452` `.` `8B58 24` `mov ebx,dword ptr ds:[eax+24]`

`00665455` `.` `895C24 38` `mov dword ptr ss:[esp+38],ebx`

`00665459` `.` `8D8C29 D8988069` `lea ecx,dword ptr ds:[ecx+ebp+698098D8]`

`00665460` `.` `8BEE` `mov ebp,esi`

`00665462` `.` `C1C1 07` `rol ecx,7`

`00665465` `.` `33EF` `xor ebp,edi`

`00665467` `.` `03CF` `add ecx,edi`

`00665469` `.` `23E9` `and ebp,ecx`

`0066546B` `.` `33EE` `xor ebp,esi`

`0066546D` `.` `03EB` `add ebp,ebx`

`0066546F` `.` `8B58 28` `mov ebx,dword ptr ds:[eax+28]`

`00665472` `.` `895C24 1C` `mov dword ptr ss:[esp+1C],ebx`

`00665476` `.` `8D942A AFF7448B` `lea edx,dword ptr ds:[edx+ebp+8B44F7AF]`

`0066547D` `.` `8BEF` `mov ebp,edi`

`0066547F` `.` `C1C2 0C` `rol edx,0C`

`00665482` `.` `03D1` `add edx,ecx`

`00665484` `.` `33E9` `xor ebp,ecx`

`00665486` `.` `23EA` `and ebp,edx`

`00665488` `.` `33EF` `xor ebp,edi`

`0066548A` `.` `03EB` `add ebp,ebx`

`0066548C` `.` `8B58 2C` `mov ebx,dword ptr ds:[eax+2C]`

`0066548F` `.` `895C24 34` `mov dword ptr ss:[esp+34],ebx`

`00665493` `.` `8DB42E B15BFFFF` `lea esi,dword ptr ds:[esi+ebp+FFFF5BB1]`

`0066549A` `.` `8BEA` `mov ebp,edx`

`0066549C` `.` `C1C6 11` `rol esi,11`

`0066549F` `.` `03F2` `add esi,edx`
