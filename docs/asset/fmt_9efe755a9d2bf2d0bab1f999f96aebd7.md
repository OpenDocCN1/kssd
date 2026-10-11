
# Flash Cam v1.78

> 原文：[`www.pediy.com/kssd/pediy05/pediy50094.htm`](https://www.pediy.com/kssd/pediy05/pediy50094.htm)

软件名称：Flash Cam v1.78

破解者：`fhmdw[BCG]`

破解工具：`trw2000` `keymake`

软件大小：1100KB

软件简介：Flash Cam 可以抓取影像为 Flash 文件，还可以设定测鼠移动路径、加入文字、外挂说明声效。

下载地址：`http://newhua.ruyi.com/down/FlashCamInstall.exe`

我在追注册码过程中共找到 4 个注册码，包括个人版的、家庭版的、专业版的以及企业版的。

```
:00512470 8B8F3C080000    mov ecx, dword ptr [edi+0000083C]
:00512476 8B9738080000    mov edx, dword ptr [edi+00000838]
:0051247C 8BC7            mov eax, edi
:0051247E E8A10A0000      call 00512F24<<---------关键 call,F8 追入
:00512483 84C0            test al, al<<---------注册标志
:00512485 7468            je 005124EF<<--------不跳则注册成功
```

====================================>>由 51247e 追入，来到这里

```
:00512F84 B920000000      mov ecx, 00000020
:00512F89 BAF19D0000      mov edx, 00009DF1
:00512F8E B81A000000      mov eax, 0000001A
:00512F93 E870FCFFFF      call 00512C08
:00512F98 8B45E8          mov eax, dword ptr [ebp-18]
:00512F9B 8B55FC          mov edx, dword ptr [ebp-04]<<--------下 d eax 得第一个注册码
:00512F9E E88D12EFFF      call 00404230<<---------真假比较
:00512FA3 7504            jne 00512FA9
:00512FA5 B301            mov bl, 01<<---------注册标志
:00512FA7 EB6D            jmp 00513016
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

`:00512F78(C), :00512FA3(C)`

```
:00512FA9 FF75F4          push [ebp-0C]
:00512FAC FF75F0          push [ebp-10]
:00512FAF 8D45E4          lea eax, dword ptr [ebp-1C]
:00512FB2 50              push eax
:00512FB3 B921000000      mov ecx, 00000021
:00512FB8 BACB9C0000      mov edx, 00009CCB
:00512FBD B81B000000      mov eax, 0000001B
:00512FC2 E841FCFFFF      call 00512C08
:00512FC7 8B45E4          mov eax, dword ptr [ebp-1C]
:00512FCA 8B55FC          mov edx, dword ptr [ebp-04]<<--------下 d eax 得第二个注册码
:00512FCD E85E12EFFF      call 00404230<<---------真假比较
:00512FD2 750B            jne 00512FDF
:00512FD4 B301            mov bl, 01<<---------注册标志
:00512FD6 C6871608000001  mov byte ptr [edi+00000816], 01
:00512FDD EB37            jmp 00513016
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:00512FD2(C)`

```
:00512FDF 837DF800        cmp dword ptr [ebp-08], 00000000
:00512FE3 742F            je 00513014
:00512FE5 8B55FC          mov edx, dword ptr [ebp-04]
:00512FE8 8B45F8          mov eax, dword ptr [ebp-08]
:00512FEB E804FEFFFF      call 00512DF4<<--------第三个注册码，F8 追入
:00512FF0 8BD8            mov ebx, eax
:00512FF2 889F17080000    mov byte ptr [edi+00000817], bl
:00512FF8 84DB            test bl, bl
:00512FFA 750F            jne 0051300B
:00512FFC 8B55FC          mov edx, dword ptr [ebp-04]
:00512FFF 8B45F8          mov eax, dword ptr [ebp-08]
:00513002 E885FEFFFF      call 00512E8C<<--------第四个注册码(企业版的)，F8 追入
:00513007 8BD8            mov ebx, eax
:00513009 EB0B            jmp 00513016
```

=======================================>>由 512feb 追入，来到这里

```
:00512E44 8D4DEC          lea ecx, dword ptr [ebp-14]
:00512E47 BA47010000      mov edx, 00000147
:00512E4C 8B45FC          mov eax, dword ptr [ebp-04]
:00512E4F E888FEFFFF      call 00512CDC
:00512E54 8B45EC          mov eax, dword ptr [ebp-14]
:00512E57 8B55F8          mov edx, dword ptr [ebp-08]<<-------下 d eax 得第三个注册码
:00512E5A E8D113EFFF      call 00404230<<-------真假比较
:00512E5F 0F94C3          sete bl<<--------设标志位
```

=======================================>>由 513002 追入，来到这里

```
:00512EDC 8D4DEC          lea ecx, dword ptr [ebp-14]
:00512EDF BA86000000      mov edx, 00000086
:00512EE4 8B45FC          mov eax, dword ptr [ebp-04]
:00512EE7 E8F0FDFFFF      call 00512CDC<<----------计算注册码的 call
:00512EEC 8B45EC          mov eax, dword ptr [ebp-14]
:00512EEF 8B55F8          mov edx, dword ptr [ebp-08]<<-------下 d eax 得第四个注册码
:00512EF2 E83913EFFF      call 00404230<<-------真假比较
:00512EF7 0F94C3          sete bl<<--------设标志位
```

分析企业版注册码的算法：

======================================>>由 512ee7 追入，来到这里

```
:00512D09 BBDB070000      mov ebx, 000007DB
:00512D0E 8B45FC          mov eax, dword ptr [ebp-04]<<----大写的公司名
:00512D11 E80A14EFFF      call 00404120<<-------取用户名长度
:00512D16 8BF0            mov esi, eax
:00512D18 85F6            test esi, esi
:00512D1A 7E4A            jle 00512D66
:00512D1C BF01000000      mov edi, 00000001
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:00512D64(C)`

```
:00512D21 8B45FC          mov eax, dword ptr [ebp-04]
:00512D24 8A4C38FF        mov cl, byte ptr [eax+edi-01]<<-----公司名各个字符 ASC 码送 cl
:00512D28 33C0            xor eax, eax<<-------清零
:00512D2A 8AC1            mov al, cl
:00512D2C 8D570D          lea edx, dword ptr [edi+0D]
:00512D2F F7EA            imul edx<<------ax 乘以 dx，结果送 ax
:00512D31 03D8            add ebx, eax<<-------ax 加到 bx 中去
:00512D33 8BC3            mov eax, ebx
:00512D35 BBFFC99A3B      mov ebx, 3B9AC9FF<<------0x3b9ac9ff 送入 bx
:00512D3A 99              cdq<<---------扩展
:00512D3B F7FB            idiv ebx<<-------ax 除以 bx，余数送 dx
:00512D3D 8BDA            mov ebx, edx
:00512D3F 8B45FC          mov eax, dword ptr [ebp-04]
:00512D42 80F145          xor cl, 45<<------cl 异或 0x45
:00512D45 33C0            xor eax, eax<<------清零
:00512D47 8AC1            mov al, cl
:00512D49 F76DF8          imul [ebp-08]
:00512D4C 03D8            add ebx, eax<<-------ax 加到 bx 中去
:00512D4E 8BC3            mov eax, ebx
:00512D50 B9FFC99A3B      mov ecx, 3B9AC9FF<<------0x3b9a9c9ff 送入 cx
:00512D55 99              cdq
:00512D56 F7F9            idiv ecx<<-------ax 除以 cx，余数送 dx
:00512D58 8BDA            mov ebx, edx
:00512D5A 69C72B300600    imul eax, edi, 0006302B<<-----edi 乘以 0x6302b,结果送 ax
:00512D60 03D8            add ebx, eax<<-------ax 加到 bx 中去
:00512D62 47              inc edi<<-------计数器
:00512D63 4E              dec esi
:00512D64 75BB            jne 00512D21<<-------未取完则继续取
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:00512D1A(C)`

```
:00512D66 8BC3            mov eax, ebx<<--------bx 入送 ax
:00512D68 B9FFE0F505      mov ecx, 05F5E0FF<<-------0x5f5e0ff 送 cx
:00512D6D 99              cdq
:00512D6E F7F9            idiv ecx<<-------ax 除以 cx，余数送 dx
:00512D70 8BDA            mov ebx, edx
:00512D72 8BC3            mov eax, ebx
:00512D74 B906000000      mov ecx, 00000006<<------6 送入 cx
:00512D79 99              cdq
:00512D7A F7F9            idiv ecx<<-------ax 除以 cx，余数送 dx
:00512D7C 83C241          add edx, 00000041<<------dx 加上 0x41,结果送 dx
:00512D7F 8855F3          mov byte ptr [ebp-0D], dl
:00512D82 895DEC          mov dword ptr [ebp-14], ebx
:00512D85 DB45EC          fild dword ptr [ebp-14]
```
