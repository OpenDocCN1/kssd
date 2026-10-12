
# TextAloudMp3

> 原文：[`www.pediy.com/kssd/pediy06/pediy6175.htm`](https://www.pediy.com/kssd/pediy06/pediy6175.htm)

〖软件大小〗：4525KB
〖软件语言〗：英文
〖软件类别〗：国外软件/共享版/音频工具
〖运行环境〗：`Win9x`/`Me`/`NT`/`2000`/`XP`
〖加入时间〗：2003-7-30 17:30:44
〖下载地址〗：`http://www.getafile.com/cgi-bin/merlot/get/nextup/TxAl1452.exe`
〖软件评级〗：☆☆☆☆

【软件介绍】：

`TextAloud MP3` 是一个相当新颖的应用程序，它可以转换任何应用程序中的文字成为声音或 `MP3` 文件 (很可惜的，目前并不支持中文字的发声)。平常是常驻在 `Windows` 系统列中，随时等待使用者利用它来读取 `Email`、`Web page` 或其它文件中的文字。`TextAloud MP3` 是利用监视剪贴簿的方式来抓取所要发声的文字 (所以使用者可以直接利用复制文字的方式来启动 `TextAloud MP3`)，而使用者可以决定是要马上聆听，还是转存成 `MP3`、`WAV` 文件，等到有空闲时再利用喜欢的 `MP3` 播放程序来收听。除了 `TextAloud MP3` 内附的声音之外，使用者还可以到作者网页去下载其它的声音，让使用者可以随自己的喜好来变换发声的人喔！

〖破解工具〗：`TRW1.22` 娃娃修改版, `OllyDbgV1.09`, `WdasmV10.0`, `Guw`

〖作者声明〗：初学破解,仅作学习交流之用,失误之处敬请大侠赐教.

【简要过程】:

试验码:`2234567865148455`

好久没来论坛了，发觉鲜有破文，大概大家都太忙了,抽空找了两个简单的软件，把过程

放上来顶顶人气，高手莫见笑！

用 `Pe-scan` 检测是 `aspack2.12` 的壳,用 `pe-scan` 的脱壳功能脱壳成功,但是不能运行!

请求 `GUW` 搞定,`621K`-->`1578K`.照例是用 `TRW` 找断点 (`bpx hmemcpy`),`OD` 载入分析!

..........(略)

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00497BDE(C)
|
:00497C23 8D55FC                       lea edx, dword ptr [ebp-04]
:00497C26 8B835C030000                 mov eax, dword ptr [ebx+0000035C]
:00497C2C E82BF5F9FF                   call 0043715C
:00497C31 689999F13F                   push 3FF19999
:00497C36 689A999999                   push 9999999A
:00497C3B 8B55FC                       mov edx, dword ptr [ebp-04]
                                     <===edx="2234567865148455",假码!
:00497C3E 8B837C030000                 mov eax, dword ptr [ebx+0000037C]
:00497C44 E803E6FFFF                   call 0049624C
                                     <===关键的 Call,追算法就跟进吧!
:00497C49 3C01                         cmp al, 01
:00497C4B 7543                         jne 00497C90

* Possible StringData Ref from Code Obj ->"Thank you for your purchase."
                                     |
:00497C4D B80C7D4900                   mov eax, 00497D0C
:00497C52 E8714AFCFF                   call 0045C6C8
                                     <===是不是很有成就感!
:00497C57 8B837C030000                 mov eax, dword ptr [ebx+0000037C]
:00497C5D 8B10                         mov edx, dword ptr [eax]
:00497C5F FF12                         call dword ptr [edx]
:00497C61 84C0                         test al, al
:00497C63 7409                         je 00497C6E
:00497C65 B201                         mov dl, 01
:00497C67 8BC3                         mov eax, ebx
:00497C69 E8F2F3FFFF                   call 00497060

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00497C63(C)
|
:00497C6E 8B45FC                       mov eax, dword ptr [ebp-04]

* Possible StringData Ref from Code Obj ->"1234123412341234"
                                     |
:00497C71 BA347D4900                   mov edx, 00497D34
:00497C76 E815C6F6FF                   call 00404290
:00497C7B 751D                         jne 00497C9A
:00497C7D C7833402000001000000         mov dword ptr [ebx+00000234], 00000001
:00497C87 8BC3                         mov eax, ebx
:00497C89 E802AAFBFF                   call 00452690
:00497C8E EB0A                         jmp 00497C9A

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00497C4B(C)
|
* Possible StringData Ref from Code Obj ->"Invalid Registration Code."
                                     |
:00497C90 B8507D4900                   mov eax, 00497D50
:00497C95 E82E4AFCFF                   call 0045C6C8
                                     <===:(!真是讨人厌!

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00497BF6(C), :00497C11(C), :00497C21(U), :00497C7B(C), :00497C8E(U)
|
:00497C9A 33C0                         xor eax, eax
:00497C9C 5A                           pop edx
:00497C9D 59                           pop ecx
:00497C9E 59                           pop ecx
:00497C9F 648910                       mov dword ptr fs:[eax], edx
:00497CA2 68C77C4900                   push 00497CC7

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00497CC5(U)
|
:00497CA7 8D45EC                       lea eax, dword ptr [ebp-14]
:00497CAA E835C2F6FF                   call 00403EE4
:00497CAF 8D45F0                       lea eax, dword ptr [ebp-10]
:00497CB2 E82DC2F6FF                   call 00403EE4
:00497CB7 8D45FC                       lea eax, dword ptr [ebp-04]
:00497CBA E825C2F6FF                   call 00403EE4
:00497CBF C3                           ret
```

★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★

①跟进上面的那个关键的 `Call`,来到如下代码:

```
:0049624C 55                           push ebp
:0049624D 8BEC                         mov ebp, esp
-------------------------------------------------------
:0049624F B910000000                   mov ecx, 00000010

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00496259(C)
|
:00496254 6A00                         push 00000000
:00496256 6A00                         push 00000000
:00496258 49                           dec ecx
:00496259 75F9                         jne 00496254
-------------------------------------------------------
清 16 个堆栈单元 0
:0049625B 51                           push ecx
:0049625C 53                           push ebx
:0049625D 56                           push esi
:0049625E 57                           push edi
:0049625F 8955FC                       mov dword ptr [ebp-04], edx
:00496262 8BF0                         mov esi, eax
:00496264 8B45FC                       mov eax, dword ptr [ebp-04]
:00496267 E8C8E0F6FF                   call 00404334
:0049626C 33C0                         xor eax, eax
:0049626E 55                           push ebp
:0049626F 687D644900                   push 0049647D
:00496274 64FF30                       push dword ptr fs:[eax]
:00496277 648920                       mov dword ptr fs:[eax], esp
:0049627A 8BC6                         mov eax, esi
:0049627C E82FF5FFFF                   call 004957B0
:00496281 33DB                         xor ebx, ebx
:00496283 8D55E4                       lea edx, dword ptr [ebp-1C]
:00496286 8B45FC                       mov eax, dword ptr [ebp-04]
:00496289 E85E33F7FF                   call 004095EC
:0049628E 8B55E4                       mov edx, dword ptr [ebp-1C]
:00496291 8D45FC                       lea eax, dword ptr [ebp-04]
:00496294 E8E3DCF6FF                   call 00403F7C
:00496299 8B45FC                       mov eax, dword ptr [ebp-04]
                                     <===eax="2234567865148455"

* Possible StringData Ref from Code Obj ->"2234123412341234"
                                     |
:0049629C BA98644900                   mov edx, 00496498
                                     <===edx="1234123412341234"
                                     猜一下,注册码应该是 16 位
:004962A1 E8EADFF6FF                   call 00404290
:004962A6 7510                         jne 004962B8
:004962A8 B201                         mov dl, 01
:004962AA 8BC6                         mov eax, esi
:004962AC E82BFBFFFF                   call 00495DDC
:004962B1 8BD8                         mov ebx, eax
:004962B3 E961010000                   jmp 00496419

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004962A6(C)
|
:004962B8 8B55FC                       mov edx, dword ptr [ebp-04]
                                     <===edx="2234567865148455"
:004962BB 8BC6                         mov eax, esi
:004962BD E8CAFCFFFF                   call 00495F8C
                                     <====此 Call 当然也要跟进的!②
:004962C2 84C0                         test al, al
:004962C4 0F844F010000                 je 00496419
                                     <===关键跳转!why?因为 00496419 距现在的代码
                                     位置 004962C4 有很大一段嘛!接跟着后面的大概
                                     是注册表操作了
:004962CA 8B55FC                       mov edx, dword ptr [ebp-04]
:004962CD 8BC6                         mov eax, esi
:004962CF E880F3FFFF                   call 00495654
:004962D4 3C01                         cmp al, 01
:004962D6 0F843D010000                 je 00496419
```



:004962DC B301 mov bl, 01

:004962DE E8194FF7FF call `0040B1FC`

:004962E3 DD5DF0 fstp qword ptr `[ebp-10]`

:004962E6 9B wait

:004962E7 FF75F4 push `[ebp-0C]`

:004962EA FF75F0 push `[ebp-10]`

:004962ED 8D45E8 lea eax, dword ptr `[ebp-18]`

:004962F0 E81F4AF7FF call `0040AD14`

:004962F5 33C9 xor ecx, ecx

:004962F7 B201 mov dl, 01

\* Possible StringData Ref from Code Obj -> `"?A"`

:004962F9 A184D74700 mov eax, dword ptr `[0047D784]`

:004962FE E8D975FEFF call `0047D8DC`

:00496303 8BF8 mov edi, eax

:00496305 C6472D00 mov `[edi+2D]`, 00

:00496309 8D4728 lea eax, dword ptr `[edi+28]`

:0049630C 8B4E14 mov ecx, dword ptr `[esi+14]`

\* Possible StringData Ref from Code Obj -> `"Software\\Microsof\\"`

:0049630F BAB4644900 mov edx, `004964B4`

:00496314 E8B3DEF6FF call `004041CC`

:00496319 8D45D4 lea eax, dword ptr `[ebp-2C]`

:0049631C 8B55FC mov edx, dword ptr `[ebp-04]`

:0049631F E844F2F6FF call `00405568`

:00496324 8D4DD4 lea ecx, dword ptr `[ebp-2C]`

\* Possible StringData Ref from Code Obj -> `"SerNum"`

:00496327 BAD0644900 mov edx, `004964D0`

:0049632C 8BC7 mov eax, edi

:0049632E E8C978FEFF call `0047DBFC`

:00496333 C6472D02 mov `[edi+2D]`, 02

:00496337 8D4728 lea eax, dword ptr `[edi+28]`

:0049633A 8B4E14 mov ecx, dword ptr `[esi+14]`

\* Possible StringData Ref from Code Obj -> `"Software\\Microsof\\"`

:0049633D BAB4644900 mov edx, `004964B4`

:00496342 E885DEF6FF call `004041CC`

:00496347 8D45C4 lea eax, dword ptr `[ebp-3C]`

:0049634A 8B55FC mov edx, dword ptr `[ebp-04]`

:0049634D E816F2F6FF call `00405568`

:00496352 8D4DC4 lea ecx, dword ptr `[ebp-3C]`

\* Possible StringData Ref from Code Obj -> `"SerNum"`

:00496355 BAD0644900 mov edx, `004964D0`

:0049635A 8BC7 mov eax, edi

:0049635C E89B78FEFF call `0047DBFC`

:00496361 C6472D00 mov `[edi+2D]`, 00

:00496365 DD4508 fld qword ptr `[ebp+08]`

:00496368 83C4F4 add esp, `FFFFFFF4`

:0049636B DB3C24 fstp tbyte ptr `[esp]`

:0049636E 9B wait

:0049636F 8D45B0 lea eax, dword ptr `[ebp-50]`

:00496372 E8F948F7FF call `0040AC70`

:00496377 8B55B0 mov edx, dword ptr `[ebp-50]`

:0049637A 8D45B4 lea eax, dword ptr `[ebp-4C]`

:0049637D E8E6F1F6FF call `00405568`

:00496382 8D4DB4 lea ecx, dword ptr `[ebp-4C]`

\* Possible StringData Ref from Code Obj -> `"RegVersion"`

:00496385 BAE0644900 mov edx, `004964E0`

:0049638A 8BC7 mov eax, edi

:0049638C E86B78FEFF call `0047DBFC`

:00496391 8D55F8 lea edx, dword ptr `[ebp-08]`

:00496394 8B75EC mov esi, dword ptr `[ebp-14]`

:00496397 8BC6 mov eax, esi

:00496399 E8F235F7FF call `00409990`

:0049639E 8D45A0 lea eax, dword ptr `[ebp-60]`

:004963A1 8B55F8 mov edx, dword ptr `[ebp-08]`

:004963A4 E8BFF1F6FF call `00405568`

:004963A9 8D4DA0 lea ecx, dword ptr `[ebp-60]`

...(略)

★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

②将跟踪进行到底，接着跟进：

```
:00495F8C 55                 push ebp
:00495F8D 8BEC               mov ebp, esp
----------------------------------------
:00495F8F B906000000         mov ecx, `00000006`
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00495F99(C)
|
:00495F94 6A00               push `00000000`
:00495F96 6A00               push `00000000`
:00495F98 49                 dec ecx
:00495F99 75F9               jne `00495F94`
----------------------------------------
:00495F9B 53                 push ebx
:00495F9C 56                 push esi
:00495F9D 57                 push edi
:00495F9E 8955FC             mov dword ptr `[ebp-04]`, edx
:00495FA1 8BD8               mov ebx, eax
:00495FA3 8B45FC             mov eax, dword ptr `[ebp-04]`
:00495FA6 E889E3F6FF         call `00404334`
:00495FAB 33C0               xor eax, eax
:00495FAD 55                 push ebp
:00495FAE 6827624900         push `00496227`
:00495FB3 64FF30             push dword ptr fs:[eax]
:00495FB6 648920             mov dword ptr fs:[eax], esp
:00495FB9 8BC3               mov eax, ebx
:00495FBB E8F0F7FFFF         call `004957B0`
:00495FC0 C645FB00           mov `[ebp-05]`, 00
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00495F4B(C)
|
:00495FC4 8D55E4             lea edx, dword ptr `[ebp-1C]`
:00495FC7 8B45FC             mov eax, dword ptr `[ebp-04]`
:00495FCA E81D36F7FF         call `004095EC`
:00495FCF 8B55E4             mov edx, dword ptr `[ebp-1C]`
:00495FD2 8D45FC             lea eax, dword ptr `[ebp-04]`
:00495FD5 E8A2DFF6FF         call `00403F7C`
:00495FDA 807B1C00           cmp byte ptr `[ebx+1C]`, 00
:00495FDE 7456               je `00496036`
:00495FE0 8B55FC             mov edx, dword ptr `[ebp-04]`
:00495FE3 B840624900         mov eax, `00496240`
:00495FE8 E87FE4F6FF         call `0040446C`
:00495FED 8BF0               mov esi, eax
:00495FEF 85F6               test esi, esi
:00495FF1 0F8400020000       je `004961F7`
:00495FF7 8D45E8             lea eax, dword ptr `[ebp-18]`
:00495FFA 50                 push eax
:00495FFB 8D5601             lea edx, dword ptr `[esi+01]`
:00495FFE B9FFFFFF7F         mov ecx, `7FFFFFFF`
:00496003 8B45FC             mov eax, dword ptr `[ebp-04]`
:00496006 E87DE3F6FF         call `00404388`
:0049600B 8B55E8             mov edx, dword ptr `[ebp-18]`
:0049600E B840624900         mov eax, `00496240`
:00496013 E854E4F6FF         call `0040446C`
:00496018 8BF0               mov esi, eax
:0049601A 85F6               test esi, esi
:0049601C 0F84D5010000       je `004961F7`
:00496022 8D45FC             lea eax, dword ptr `[ebp-04]`
:00496025 50                 push eax
:00496026 8D5601             lea edx, dword ptr `[esi+01]`
:00496029 B9FFFFFF7F         mov ecx, `7FFFFFFF`
:0049602E 8B45E8             mov eax, dword ptr `[ebp-18]`
:00496031 E852E3F6FF         call `00404388`
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00495FDE(C)
|
:00496036 8B45FC             mov eax, dword ptr `[ebp-04]`
  <=== eax = "2234567865148455", 假码!
:00496039 E842E1F6FF         call `00404180`
  <=== 取假码位数
:0049603E 83F810             cmp eax, `00000010`
  <=== 果然注册码应为 16 位
:00496041 0F85B0010000       jne `004961F7`
:00496047 8D45E8             lea eax, dword ptr `[ebp-18]`
:0049604A 50                 push eax
```



```
----------------------------------------------------  
:0049604B B903000000                 mov ecx, 00000003  
:00496050 BA01000000                 mov edx, 00000001  
:00496055 8B45FC                     mov eax, dword ptr [ebp-04]  
                                     <===eax="2234567865148455"  
:00496058 E82BE3F6FF                 call 00404388  
----------------------------------------------------  
```

呵呵，标准的 Api 调用，从假码的第 1 位向后取 3 个字符，`eax`="223"

```
:0049605D 33C0                       xor eax, eax  
:0049605F 55                         push ebp  
:00496060 687F604900                 push 0049607F  
:00496065 64FF30                     push dword ptr fs:[eax]  
:00496068 648920                     mov dword ptr fs:[eax], esp  
:0049606B 8B45E8                     mov eax, dword ptr [ebp-18]  
                                     <===eax="223"  
:0049606E E8BD39F7FF                 call 00409A30  
                                     <===Dec-->Hex，即 223(D)-->DF(H)  
:00496073 8BF0                       mov esi, eax  
:00496075 33C0                       xor eax, eax  
:00496077 5A                         pop edx  
:00496078 59                         pop ecx  
:00496079 59                         pop ecx  
:0049607A 648910                     mov dword ptr fs:[eax], edx  
:0049607D EB14                       jmp 00496093  
:0049607F E900D5F6FF                 jmp 00403584  
:00496084 E817D9F6FF                 call 004039A0  
:00496089 E969010000                 jmp 004961F7  
:0049608E E80DD9F6FF                 call 004039A0 
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:0049607D(U)`

```
:00496093 837B246F                   cmp dword ptr [ebx+24], 0000006F  
                                     <===[ebx+24]中为定值 DF  
:00496097 7506                       jne 0049609F  
:00496099 8B4320                     mov eax, dword ptr [ebx+20]  
:0049609C 894324                     mov dword ptr [ebx+24], eax 
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:00496097(C)`

```
:0049609F 3B7320                     cmp esi, dword ptr [ebx+20]  
                                     <===[ebx+20]中为定值 DE,esi 为 DF(H),222 的十六进制  
:004960A2 7409                       je 004960AD  
:004960A4 3B7324                     cmp esi, dword ptr [ebx+24]  
                                     <===[ebx+24]中为定值 DF,看来注册码的前 3 位  
                                     应该为 223(DF)或 222(DE),幸运!  
:004960A7 0F854A010000               jne 004961F7 
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:004960A2(C)`

```
:004960AD 8D45E8                     lea eax, dword ptr [ebp-18]  
:004960B0 50                         push eax  
-----------------------------------------------------------  
:004960B1 B905000000                 mov ecx, 00000005  
:004960B6 BA04000000                 mov edx, 00000004  
:004960BB 8B45FC                     mov eax, dword ptr [ebp-04]  
:004960BE E8C5E2F6FF                 call 00404388  
-----------------------------------------------------------  
```

不用注释了吧！

```
:004960C3 33C0                       xor eax, eax  
:004960C5 55                         push ebp  
:004960C6 68E5604900                 push 004960E5  
:004960CB 64FF30                     push dword ptr fs:[eax]  
:004960CE 648920                     mov dword ptr fs:[eax], esp  
:004960D1 8B45E8                     mov eax, dword ptr [ebp-18]  
                                     <===eax="45678",假码 4~8 位  
:004960D4 E85739F7FF                 call 00409A30  
:004960D9 8BF8                       mov edi, eax  
                                     <===edi=B26E(H)-->45678(D)  
:004960DB 33C0                       xor eax, eax  
:004960DD 5A                         pop edx  
:004960DE 59                         pop ecx  
:004960DF 59                         pop ecx  
:004960E0 648910                     mov dword ptr fs:[eax], edx  
:004960E3 EB14                       jmp 004960F9  
:004960E5 E99AD4F6FF                 jmp 00403584  
:004960EA E8B1D8F6FF                 call 004039A0  
:004960EF E903010000                 jmp 004961F7  
:004960F4 E8A7D8F6FF                 call 004039A0 
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:004960E3(U)`

```
:004960F9 81FF10270000               cmp edi, 00002710  
                                     <===比较是否小于 10000,即判断第 4 位是否为 0  
:004960FF 0F8CF2000000               jl 004961F7  
:00496105 8D45E8                     lea eax, dword ptr [ebp-18]  
:00496108 50                         push eax  
------------------------------------------------------------  
:00496109 B908000000                 mov ecx, 00000008  
:0049610E BA0A000000                 mov edx, 0000000A  
:00496113 8B45FC                     mov eax, dword ptr [ebp-04]  
:00496116 E86DE2F6FF                 call 00404388  
------------------------------------------------------------  
```

```
:0049611B 33C0                       xor eax, eax  
:0049611D 55                         push ebp  
:0049611E 684A614900                 push 0049614A  
:00496123 64FF30                     push dword ptr fs:[eax]  
:00496126 648920                     mov dword ptr fs:[eax], esp  
:00496129 8D4DE0                     lea ecx, dword ptr [ebp-20]  
:0049612C 8B55E8                     mov edx, dword ptr [ebp-18]  
                                     <===edx="5148455",假码后 7 位  
:0049612F 8BC3                       mov eax, ebx  
:00496131 E8DE040000                 call 00496614  
:00496136 8B45E0                     mov eax, dword ptr [ebp-20]  
                                     <===eax="5148455"  
:00496139 E89A4BF7FF                 call 0040ACD8  
                                     <===这个 Call 大概是将字符型转换成整数型  
:0049613E DDD8                       fstp st(0)  
                                     <===FSTP dest  
                                     dest <- st(0) (mem32/mem64/mem80)；  
                                     然后再执行一次出栈操作  
:00496140 33C0                       xor eax, eax  
:00496142 5A                         pop edx  
:00496143 59                         pop ecx  
:00496144 59                         pop ecx  
:00496145 648910                     mov dword ptr fs:[eax], edx  
:00496148 EB14                       jmp 0049615E  
:0049614A E935D4F6FF                 jmp 00403584  
:0049614F E84CD8F6FF                 call 004039A0  
:00496154 E99E000000                 jmp 004961F7  
:00496159 E842D8F6FF                 call 004039A0 
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:00496148(U)`

```
:0049615E 8D0437                     lea eax, dword ptr [edi+esi]  
                                     <===eax=B26E+DF=B34D(H)-->45901  
                                     B26E 和 DF 应该还有印象吧,对应 45678 和 223 的 Hex  
:00496161 8945DC                     mov dword ptr [ebp-24], eax  
:00496164 DB45DC                     fild dword ptr [ebp-24]  
                                     <===FILD src 装入整数到 st(0)   
                                     st(0) <- src (mem16/mem32/mem64)  
                                     即 45901 装入到 st(0)  
:00496167 D95DF0                     fstp dword ptr [ebp-10]  
:0049616A 9B                         wait  
:0049616B DB4328                     fild dword ptr [ebx+28]  
                                     <===[ebx+28]中为定值 5  
:0049616E D84DF0                     fmul dword ptr [ebp-10]  
                                     <===FMUL st(i) 即 st(0) <- st(0) * st(i)  
                                     5*45091=229505  
:00496171 D95DF0                     fstp dword ptr [ebp-10]  
                                     <===结果存入[ebp-10]中  
:00496174 9B                         wait  
:00496175 EB0A                       jmp 00496181 
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:0049618D(C)`

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

```
:00496177 DB432C                     fild dword ptr [ebx+2C]  
                                     <===[ebx+2c]中为定值 8  
:0049617A D84DF0                     fmul dword ptr [ebp-10]  
```



```
                   <===8*229505=1836040  

:0049617D D95DF0                 fstp dword ptr [ebp-10]  

                   <===结果存入[ebp-10]  

:00496180 9B                     wait 

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  

|:00496175(U)  

|  

:00496181 D945F0                 fld dword ptr [ebp-10]  

                   <===取上面的计算结果 229505  

:00496184 D81D44624900           fcomp dword ptr [00496244]  

                   <===FCOM op   实数比较  

                   将标志位设置为 st(0) - op (mem32/mem64)的  

                   结果标志位   

                   比较是否小于 10000000,小于的话接着乘 8,我最后  

                   得到的值为 14688320  

:0049618A DFE0                   fstsw ax  

                   <===FSTSW dest  保存状态字的值到 dest  

                   dest<-MSW (mem16)  

:0049618C 9E                     sahf  

:0049618D 72E8                   jb 00496177  

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆  

:0049618F 81FF30750000           cmp edi, 00007530  

                   <===比较是否小于 30000,记得 edi 中是什么吗?  

                   B26E(H)--->45678(D),注册码的第 4~8 位  

:00496195 7E0D                   jle 004961A4  

:00496197 D945F0                 fld dword ptr [ebp-10]  

:0049619A D82548624900           fsub dword ptr [00496248]  

                   <===[00496248]中为定值 1  

                   FSUB src 即 st(0) <-st(0) - src (reg/mem)  

                   14688320-1=14688319 

:004961A0 D95DF0                 fstp dword ptr [ebp-10]  

:004961A3 9B                     wait 

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  

|:00496195(C)  

|  

:004961A4 8D55D8                 lea edx, dword ptr [ebp-28]  

:004961A7 8BC6                   mov eax, esi  

                   <===eax=7B  

:004961A9 E8E237F7FF             call 00409990  

                   <===十六进制--->十进制  

:004961AE FF75D8                 push [ebp-28]  

:004961B1 8D55D4                 lea edx, dword ptr [ebp-2C]  

:004961B4 8BC7                   mov eax, edi  

                   <===eax=B26E(H)-->45678(D)  

:004961B6 E8D537F7FF             call 00409990  

:004961BB FF75D4                 push [ebp-2C]  

:004961BE D945F0                 fld dword ptr [ebp-10]  

                   <===取上面的计算结果 14688319  

:004961C1 83C4F4                 add esp, FFFFFFF4  

:004961C4 DB3C24                 fstp tbyte ptr [esp]  

:004961C7 9B                     wait  

:004961C8 8D45D0                 lea eax, dword ptr [ebp-30]  

:004961CB E8A04AF7FF             call 0040AC70  

:004961D0 FF75D0                 push [ebp-30]  

:004961D3 8D45EC                 lea eax, dword ptr [ebp-14]  

:004961D6 BA03000000             mov edx, 00000003  

:004961DB E860E0F6FF             call 00404240  

:004961E0 8B45EC                 mov eax, dword ptr [ebp-14]  

                   <===eax="2234567814688319"  

:004961E3 8B55FC                 mov edx, dword ptr [ebp-04]  

                   <===edx="2234567865148455"  

:004961E6 E8A5E0F6FF             call 00404290  

                   <===不用说是比对的 Call 了  

:004961EB 7406                   je 004961F3  

:004961ED C645FB00               mov [ebp-05], 00  

                   <===置标志位 0,失败!  

:004961F1 EB04                   jmp 004961F7 

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  

|:004961EB(C)  

|  

:004961F3 C645FB01               mov [ebp-05], 01  

                   <===置标志位 1,成功! 

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:  

|:00495FF1(C), :0049601C(C), :00496041(C), :00496089(U), :004960A7(C)  

|:004960EF(U), :004960FF(C), :00496154(U), :004961F1(U)  

|  

:004961F7 33C0                   xor eax, eax  

:004961F9 5A                     pop edx  

:004961FA 59                     pop ecx  

:004961FB 59                     pop ecx  

:004961FC 648910                 mov dword ptr fs:[eax], edx  

:004961FF 682E624900             push 0049622E 

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  

|:0049622C(U)  

|  

:00496204 8D45D0                 lea eax, dword ptr [ebp-30]  

:00496207 BA03000000             mov edx, 00000003  

:0049620C E8F7DCF6FF             call 00403F08  

:00496211 8D45E0                 lea eax, dword ptr [ebp-20]  

:00496214 BA04000000             mov edx, 00000004  

:00496219 E8EADCF6FF             call 00403F08  

:0049621E 8D45FC                 lea eax, dword ptr [ebp-04]  

:00496221 E8BEDCF6FF             call 00403EE4  

:00496226 C3                     ret 

:00496227 E90CD6F6FF             jmp 00403838  

:0049622C EBD6                   jmp 00496204  

:0049622E 8A45FB                 mov al, byte ptr [ebp-05]  

:00496231 5F                     pop edi  

:00496232 5E                     pop esi  

:00496233 5B                     pop ebx  

:00496234 8BE5                   mov esp, ebp  

:00496236 5D                     pop ebp  

:00496237 C3                     ret 
```

### 【总结】

这个软件的算法也很简单，同样有浮点运算。

注册码必须为 `16` 位，前 `3` 位须为 `222` 或 `223`，设为 `a`；设后 `5` 位为 `b`（须大于 `10000`），则注册码的最后 `8` 位 `c` 为：

`c=(a+b)*5*8`，若 `c<10000000`，则 `c` 再乘 `8`，依此往后类推直至 `c>10000000` 为止

如果 `b>30000`，则注册码的后 `8` 位为 `c-1`，若 `b<30000`，则注册码的后 `8` 位为 `c`

软件注册成功后将注册信息保存在注册表的 `HKEY_CURRENT_USER\Software\Microsof\TATrialLoc` 下。

放上 `1` 个可用的注册码：`2234567814688319`

Cracked By ShenGe[BCG] 2003.7.31

`4dd466d0e1e0cc123afbe504da2462ca`
