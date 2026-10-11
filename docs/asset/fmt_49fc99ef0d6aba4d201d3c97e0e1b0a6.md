
# QQGame浅析系列(3) --- 山重水复疑无路（上）

整体浏览一下发现这里是解读封包参数的。如果有 `call` 到 `Utility` 领空的都不需要跟进，那里是解密的过程，否则就一去不能回了（主要是在代码的海洋里很容易晕的）。例如在 `Utility` 领空有这样的字符串提示：

- `"DecodeInt16 %p"`，`LF`
- `"DecodeBuffer 0x%p %d"`，`LF`
- `"Param Error!"`，`LF`
- `ASCII "Read to end!"`，`LF`
- ……

所以可以暂时认为 `sub_02D2226D` 是解读数据参数的，下一步跟进：

```
02D220C9 E8 F7020000 call 02D223C5

02D423C5 B8 4C7DD502 mov eax, 02D57D4C
02D423CA E8 11560100 call 02D579E0
02D423CF 83EC 24 sub esp, 24
02D423D2 56 push esi
02D423D3 8BF1 mov esi, ecx
02D423D5 8365 E4 00 and dword ptr [ebp-1C], 0
02D423D9 8D4D F3 lea ecx, dword ptr [ebp-D]
02D423DC 8B46 58 mov eax, dword ptr [esi+58]
02D423DF 51 push ecx
02D423E0 8D4D E4 lea ecx, dword ptr [ebp-1C]
02D423E3 51 push ecx
02D423E4 50 push eax
02D423E5 8D4D D0 lea ecx, dword ptr [ebp-30]
02D423E8 E8 D3020000 call 02D426C0
02D423ED 8B46 50 mov eax, dword ptr [esi+50]
02D423F0 8365 FC 00 and dword ptr [ebp-4], 0
02D423F4 8B08 mov ecx, dword ptr [eax]
02D423F6 3BC8 cmp ecx, eax
02D423F8 894D EC mov dword ptr [ebp-14], ecx
02D423FB 74 23 je short 02D42420
02D423FD 57 push edi
02D423FE 33FF xor edi, edi
02D42400 8B45 EC mov eax, dword ptr [ebp-14]
02D42403 8B4D D4 mov ecx, dword ptr [ebp-2C]
02D42406 8B40 0C mov eax, dword ptr [eax+C]
02D42409 89040F mov dword ptr [edi+ecx], eax
02D4240C 8D4D EC lea ecx, dword ptr [ebp-14]
02D4240F E8 0F050000 call 02D42923
02D42414 8B46 50 mov eax, dword ptr [esi+50]
02D42417 83C7 04 add edi, 4
02D4241A 3945 EC cmp dword ptr [ebp-14], eax
02D4241D ^ 75 E1 jnz short 02D42400
02D4241F 5F pop edi
02D42420 837D D4 00 cmp dword ptr [ebp-2C], 0
02D42424 74 60 je short 02D42486
02D42426 8B45 D8 mov eax, dword ptr [ebp-28]
02D42429 8BC8 mov ecx, eax
02D4242B 2B4D D4 sub ecx, dword ptr [ebp-2C]
02D4242E F7C1 FCFFFFFF test ecx, FFFFFFFC
02D42434 74 50 je short 02D42486
02D42436 8B48 FC mov ecx, dword ptr [eax-4]
02D42439 83C0 FC add eax, -4
02D4243C 894D E8 mov dword ptr [ebp-18], ecx
02D4243F 50 push eax
02D42440 8D4D D0 lea ecx, dword ptr [ebp-30]
02D42443 E8 11050000 call 02D42959
02D42448 8D45 E8 lea eax, dword ptr [ebp-18]
02D4244B 8D4E 4C lea ecx, dword ptr [esi+4C]
02D4244E 50 push eax
02D4244F 8D45 E0 lea eax, dword ptr [ebp-20]
02D42452 50 push eax
02D42453 E8 91040000 call 02D428E9
02D42458 8B45 E0 mov eax, dword ptr [ebp-20]
02D4245B 3B46 50 cmp eax, dword ptr [esi+50]
02D4245E ^ 74 C0 je short 02D42420
02D42460 8B4D E8 mov ecx, dword ptr [ebp-18]
02D42463 85C9 test ecx, ecx
02D42465 ^ 74 B9 je short 02D42420
02D42467 FF75 0C push dword ptr [ebp+C]
02D4246A 8B01 mov eax, dword ptr [ecx]
02D4246C FF75 08 push dword ptr [ebp+8]
02D4246F FF50 14 call dword ptr [eax+14]
02D42472 85C0 test eax, eax
02D42474 ^ 74 AA je short 02D42420
02D42476 8B4D E8 mov ecx, dword ptr [ebp-18]
02D42479 FF75 0C push dword ptr [ebp+C]
02D4247C 8B01 mov eax, dword ptr [ecx]
02D4247E FF75 08 push dword ptr [ebp+8]
02D42481 FF50 10 call dword ptr [eax+10] ; 进
02D42484 ^ EB 9A jmp short 02D42420
02D42486 834D FC FF or dword ptr [ebp-4], FFFFFFFF
02D4248A 8D4D D0 lea ecx, dword ptr [ebp-30]
02D4248D E8 7E020000 call 02D42710 ; 善后处理
02D42492 8B4D F4 mov ecx, dword ptr [ebp-C]
02D42495 5E pop esi
02D42496 64:890D 0000000> mov dword ptr fs:[0], ecx
02D4249D C9 leave
02D4249E C2 0800 retn 8
```

分析的结果反应在注释里，我们跟进：

```
02D42481 call dword ptr [eax+10]
```

这时从 `BaseProt` 领空进入了 `ScatProt` 领空。

从 `BaseProt` 领空进入 `ScatProt` 领空：

```
043F60B7 . 56 push esi
043F60B8 . 57 push edi
043F60B9 FF7424 10 push dword ptr [esp+10] ; 数据大小
043F60BD . 8BF1 mov esi, ecx ; |
043F60BF . FF7424 10 push dword ptr [esp+10] ; |Arg1
043F60C3 E8 1268FFFF call 043EC8DA ; \\ScatProt.1000C8DA
043F60C8 . 8BF8 mov edi, eax
043F60CA B8 44D04004 mov eax, 0440D044 ; ASCII "sucessfully"
043F60CF . 85FF test edi, edi
043F60D1 . 75 05 jnz short 043F60D8
043F60D3 . B8 3CD04004 mov eax, 0440D03C ; ASCII "failed"
043F60D8 > 50 push eax
043F60D9 . 83C6 14 add esi, 14
043F60DC . 68 28D04004 push 0440D028 ; ASCII "Package decoded %s"
043F60E1 . 56 push esi
043F60E2 . E8 C2290000 call 043F8AA9
043F60E7 . 83C4 0C add esp, 0C
043F60EA . 8BC7 mov eax, edi
043F60EC . 5F pop edi
043F60ED . 5E pop esi
043F60EE . C2 0800 retn 8
```

看到那个 `"sucessfully"` 串提示了吗？说明前面一个 `call 043EC8DA` 完成了解密工作。当然要跟进了。

```
043EC8DA /$ 55 push ebp
043EC8DB |. 8BEC mov ebp, esp
043EC8DD |. 83EC 10 sub esp, 10
043EC8E0 |. 53 push ebx
043EC8E1 |. 8B5D 0C mov ebx, dword ptr [ebp+C]
043EC8E4 |. 56 push esi
043EC8E5 |. 8BF1 mov esi, ecx
043EC8E7 |. 57 push edi
043EC8E8 |. 53 push ebx
043EC8E9 |. 8B4E 4C mov ecx, dword ptr [esi+4C]
043EC8EC |. FF75 08 push dword ptr [ebp+8]
043EC8EF |. 895E 60 mov dword ptr [esi+60], ebx
043EC8F2 |. 8B01 mov eax, dword ptr [ecx]
043EC8F4 |. FF50 0C call dword ptr [eax+C]
043EC8F7 |. 66:8365 F0 00 and word ptr [ebp-10], 0
043EC8FC |. 33C0 xor eax, eax
043EC8FE |. 8D7D F2 lea edi, dword ptr [ebp-E]
043EC901 |. 8BCE mov ecx, esi
043EC903 |. AB stos dword ptr es:[edi]
043EC904 |. AB stos dword ptr es:[edi]
043EC905 |. AB stos dword ptr es:[edi]
043EC906 |. 66:AB stos word ptr es:[edi]
043EC908 |. 8D45 F0 lea eax, dword ptr [ebp-10]
043EC90B |. 50 push eax
043EC90C |. FF76 4C push dword ptr [esi+4C]
043EC90F |. E8 2573FFFF call 043E3C39
043EC914 |. 8B4E 4C mov ecx, dword ptr [esi+4C]
043EC917 |. 8365 0C 00 and dword ptr [ebp+C], 0
043EC91B |. 8BF8 mov edi, eax
043EC91D |. 8D55 0C lea edx, dword ptr [ebp+C]
043EC920 |. 8B01 mov eax, dword ptr [ecx]
043EC922 |. 52 push edx
043EC923 |. FF50 1C call dword ptr [eax+1C]
043EC926 |. 395D 0C cmp dword ptr [ebp+C], ebx
043EC929 |. 7F 21 jg short 043EC94C
043EC92B |. 8B4E 4C mov ecx, dword ptr [esi+4C]
043EC92E |. 836D 0C 02 sub dword ptr [ebp+C], 2
043EC932 |. 8B1E mov ebx, dword ptr [esi]
043EC934 |. 8B01 mov eax, dword ptr [ecx]
043EC936 |. FF50 30 call dword ptr [eax+30]
043EC939 50 push eax
043EC93A 8D45 F0 lea eax, dword ptr [ebp-10]
043EC93D |. FF75 0C push dword ptr [ebp+C]
043EC940 8BCE mov ecx, esi
043EC942 |. 57 push edi
043EC943 |. 50 push eax
043EC944 |. FF76 4C push dword ptr [esi+4C]
043EC947 |. FF53 24 call dword ptr [ebx+24] ; 进
043EC94A |. EB 37 jmp short 043EC983
043EC94C |> 8D7E 14 lea edi, dword ptr [esi+14]
043EC94F |. 68 D0D04004 push 0440D0D0 ; ASCII LF, LF, "*******************************"
043EC954 |. 57 push edi
043EC955 |. E8 2D91FFFF call 043E5A87
043EC95A |. 59 pop ecx
043EC95B |. 59 pop ecx
043EC95C |. 68 90D04004 push 0440D090 ; ASCII "Length specified in package is bigger than total package size"
043EC961 |. 57 push edi
043EC962 |. E8 2091FFFF call 043E5A87
043EC967 |. 59 pop ecx
043EC968 |. 59 pop ecx
043EC969 |. 68 78D04004 push 0440D078 ; ASCII "Package is discarded!"
043EC96E |. 57 push edi
043EC96F |. E8 1391FFFF call 043E5A87
043EC974 |. 59 pop ecx
043EC975 |. 59 pop ecx
043EC976 |. 68 50D04004 push 0440D050 ; ASCII "*******************************, LF, LF"
043EC97B |. 57 push edi
043EC97C |. E8 0691FFFF call 043E5A87
043EC981 |. 59 pop ecx
043EC982 |. 59 pop ecx
043EC983 |> 8B4E 4C mov ecx, dword ptr [esi+4C]
043EC986 |. 8B01 mov eax, dword ptr [ecx]
043EC988 |. FF50 10 call dword ptr [eax+10] ; 过
043EC98B |. 6A 01 push 1
043EC98D |. 58 pop eax
043EC98E |. 5F pop edi
043EC98F |. 5E pop esi
```
