
```assembly
.text:00406DB9 8D 8D 9C EA FF FF lea ecx, [ebp+var_1564] ;
.text:00406DB9 ; 用 2517220091423618851 字符串作为 key 来
.text:00406DBF E8 70 EF 00 00 call blowfish_init ; 初始化 key_pbox 和 key_sbox
.text:00406DC4 8B 55 D0 mov edx, [ebp+var_30] ; 其中还有一个地址值参与运算
.text:00406DC7 83 EC 08 sub esp, 8
.text:00406DCA 8B C4 mov eax, esp
.text:00406DCC 89 10 mov [eax], edx
.text:00406DCE 8B 4D D4 mov ecx, [ebp+var_2C]
.text:00406DD1 89 48 04 mov [eax+4], ecx
.text:00406DD4 8B 45 D4 mov eax, [ebp+var_2C]
.text:00406DD7 3B C7 cmp eax, edi
.text:00406DD9 89 65 08 mov [ebp+arg_0], esp
.text:00406DDC 74 02 jz short loc_406DE0
.text:00406DDE FF 00 inc dword ptr [eax]
.text:00406DE0
.text:00406DE0 loc_406DE0: ; CODE XREF: sub_406580+85Cj
.text:00406DE0 8D 55 E8 lea edx, [ebp+var_18]
.text:00406DE3 52 push edx
.text:00406DE4 8D 8D 9C EA FF FF lea ecx, [ebp+var_1564]
.text:00406DEA E8 3F EF 00 00 call ?Decrypt@uti_Blowfish@@QAE?AVcow_String@@V2@@Z ;对由上面 BASE64 解码的数据处理进行 blowfish 解码
.text:00406DEF 8B 4D C0 mov ecx, [ebp+var_40] ;该函数在对数据解码前先进行高低的字节交换,解码
.text:00406DF2 83 EC 08 sub esp, 8 ;后再进行高低的字节交换.
.text:00406DF5 8B C4 mov eax, esp
.text:00406DF7 89 08 mov [eax], ecx
.text:00406DF9 8B 55 C4 mov edx, [ebp+var_3C]
.text:00406DFC 89 50 04 mov [eax+4], edx
.text:00406DFF 8B 45 C4 mov eax, [ebp+var_3C]
.text:00406E02 3B C7 cmp eax, edi
.text:00406E04 B3 34 mov bl, 34h
.text:00406E06 88 5D FC mov byte ptr [ebp+var_4], bl
.text:00406E09 89 65 08 mov [ebp+arg_0], esp
.text:00406E0C 74 02 jz short loc_406E10
.text:00406E0E FF 00 inc dword ptr [eax]
.text:00406E10
.text:00406E10 loc_406E10: ; CODE XREF: sub_406580+88Cj
.text:00406E10 8D 8D 9C EA FF FF lea ecx, [ebp+var_1564] ; 用 3161117539399797711211784101822777 字符串作为 key
.text:00406E16 E8 19 EF 00 00 call blowfish_init ; 来初始化 key_pbox 和 key_sbox,其中还有一个地址值参与
.text:00406E1B 8B 4D E8 mov ecx, [ebp+var_18] ; 运算
.text:00406E1E 83 EC 08 sub esp, 8
.text:00406E21 8B C4 mov eax, esp
.text:00406E23 89 08 mov [eax], ecx
.text:00406E25 8B 55 EC mov edx, [ebp+var_14]
.text:00406E28 89 50 04 mov [eax+4], edx
.text:00406E2B 8B 45 EC mov eax, [ebp+var_14]
.text:00406E2E 3B C7 cmp eax, edi
.text:00406E30 89 65 08 mov [ebp+arg_0], esp
.text:00406E33 74 02 jz short loc_406E37
.text:00406E35 FF 00 inc dword ptr [eax]
.text:00406E37
.text:00406E37 loc_406E37: ; CODE XREF: sub_406580+8B3j
.text:00406E37 8D 45 A0 lea eax, [ebp+var_60]
.text:00406E3A 50 push eax
.text:00406E3B 8D 8D 9C EA FF FF lea ecx, [ebp+var_1564] ; 再对上次 blowfish_de 的结果进行解码,后得到一个长度为 16 的字符串
.text:00406E41 E8 E8 EE 00 00 call ?Decrypt@uti_Blowfish@@QAE?AVcow_String@@V2@@Z ; uti_Blowfish::Decrypt(cow_String)
.text:00406E46 8B F0 mov esi, eax
.text:00406E48 8B 46 04 mov eax, [esi+4]
.text:00406E4B 3B C7 cmp eax, edi
.text:00406E4D C6 45 FC 35 mov byte ptr [ebp+var_4], 35h
.text:00406E51 74 02 jz short loc_406E55
.text:00406E53 FF 00 inc dword ptr [eax]
.text:00406E55
.text:00406E55 loc_406E55: ; CODE XREF: sub_406580+8D1j
.text:00406E55 8B 45 DC mov eax, [ebp+var_24]
.text:00406E58 3B C7 cmp eax, edi
.text:00406E5A 74 0E jz short loc_406E6A
.text:00406E5C FF 08 dec dword ptr [eax]
.text:00406E5E 39 38 cmp [eax], edi
.text:00406E60 75 08 jnz short loc_406E6A
.text:00406E62 8D 4D D8 lea ecx, [ebp+var_28]
.text:00406E65 E8 50 F0 00 00 call atr_StringBase__Delete_void_
.text:00406E6A
.text:00406E6A loc_406E6A: ; CODE XREF: sub_406580+8DAj
.text:00406E6A ; sub_406580+8E0j
.text:00406E6A 8B 0E mov ecx, [esi]
.text:00406E6C 8B 45 A4 mov eax, [ebp+var_5C]
.text:00406E6F 3B C7 cmp eax, edi
.text:00406E71 89 4D D8 mov [ebp+var_28], ecx
.text:00406E74 8B 56 04 mov edx, [esi+4]
.text:00406E77 89 55 DC mov [ebp+var_24], edx
.text:00406E7A 88 5D FC mov byte ptr [ebp+var_4], bl
.text:00406E7D 74 0E jz short loc_406E8D
.text:00406E7F FF 08 dec dword ptr [eax]
.text:00406E81 39 38 cmp [eax], edi
.text:00406E83 75 08 jnz short loc_406E8D
.text:00406E85 8D 4D A0 lea ecx, [ebp+var_60]
.text:00406E88 E8 2D F0 00 00 call atr_StringBase__Delete_void_
.text:00406E8D
.text:00406E8D loc_406E8D: ; CODE XREF: sub_406580+8FDj
.text:00406E8D ; sub_406580+903j
.text:00406E8D 8B 45 EC mov eax, [ebp+var_14]
.text:00406E90 3B C7 cmp eax, edi
.text:00406E92 C6 45 FC 33 mov byte ptr [ebp+var_4], 33h
.text:00406E96 74 0E jz short loc_406EA6
.text:00406E98 FF 08 dec dword ptr [eax]
.text:00406E9A 39 38 cmp [eax], edi
.text:00406E9C 75 08 jnz short loc_406EA6
.text:00406E9E 8D 4D E8 lea ecx, [ebp+var_18]
.text:00406EA1 E8 14 F0 00 00 call atr_StringBase__Delete_void_
.text:00406EA6
.text:00406EA6 loc_406EA6: ; CODE XREF: sub_406580+916j
.text:00406EA6 ; sub_406580+91Cj
.text:00406EA6 8B 45 D4 mov eax, [ebp+var_2C]
.text:00406EA9 3B C7 cmp eax, edi
```
