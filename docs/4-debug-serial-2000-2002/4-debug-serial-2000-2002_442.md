
# PSXVIDEO Ver1.12

> 原文：`www.pediy.com/kssd/pediy03/forum810.htm`

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;注册 PSXVIDEO Ver1.12 详细算法

软件名称：PSXVIDEO Ver1.12

软件下载：等我汉化完传到 `HTTP://ZOMBIEYS.126.COM` 去

主要功能：PSXVIDEO 是一个能将 PS 游戏 CD-ROM 上的动画档案转换成 PC 平台能播放的 AVI 档案的一个转档程式, 是由一位日本人 `tany` 先生所写, 目前为最新版 `v1.12`

已经可以将大部分的 PS 游戏动画转出, 几乎没有转不出的动画

`PSXVIDEO v1.07` 由 `tany` 先生於 97/10/20 完成 , 增加了 `FF7` 的检查选项,以及一些些细部的修正等.

主要目的：寻找合适的注册码

破解难度：非常简单(适合初学者)

&nbsp;&nbsp;&nbsp;&nbsp;正规方法：选注册填自己的大名，注册码填 `78787878`,进 `s-ice` 下 `s 30:0 l ffffffff '78787878'`,下 `bpm "搜到的地址"`,按 `f5` 返回,点注册被拦到，按几次 `f12` 知道进入 `PSXVIDEO` 领空，再按几次 `f10` 到这里

`* Possible Reference to Dialog: REGIST, CONTROL_ID:03F7, ""`

|
:004099FA 68F7030000 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; push 000003F7
:004099FF 56 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; push esi
:00409A00 FFD7 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; call edi
:00409A02 8D4C240C &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; lea ecx, dword ptr [esp+0C]
:00409A06 51 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; push ecx
:00409A07 E844B6FFFF &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; call 00405050 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; //算 KEY 的 CALL
:00409A0C 83C404 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; add esp, 00000004
:00409A0F 85C0 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; test eax, eax
:00409A11 740E &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; je 00409A21 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; //跳下去就成功了
:00409A13 6A10 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; push 00000010

`* Possible StringData Ref from Data Obj ->"ERROR"`

|
:00409A15 68E0514100 &nbsp;&nbsp;&nbsp;&nbsp; push 004151E0

`* Possible StringData Ref from Data Obj ->"Password Error."`

|
:00409A1A 689C664100 &nbsp;&nbsp;&nbsp;&nbsp; push 0041669C
:00409A1F EB18 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; jmp 00409A39 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; //骗子-\_-(TANY 语)

`* Referenced by a (U)nconditional or (C)onditional Jump at Address:`

|:00409A11(C)

|
:00409A21 C7056C9C450001000000 &nbsp;&nbsp; mov dword ptr [00459C6C], 00000001 &nbsp; //成功了，没被抓到^Q^
:00409A2B EB0F &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; jmp 00409A3C

我们来看看它怎码算的 KEY 吧@\_@，还好了很短，也很简单:)

```
:00405050 81EC08010000 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; sub esp, 00000108 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; //准备抓骗子(TANY 语)
:00405056 53 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; push ebx
:00405057 55 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; push ebp
:00405058 56 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; push esi
:00405059 57 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; push edi
:0040505A 8BBC241C010000 &nbsp;&nbsp; mov edi, dword ptr [esp+0000011C] &nbsp; //取你输的注册码->EDI
:00405061 33D2 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; xor edx, edx &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; //准备了:)
:00405063 BE1A000000 &nbsp;&nbsp;&nbsp;&nbsp; mov esi, 0000001A &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; //做除数用
:00405068 BD1A000000 &nbsp;&nbsp;&nbsp;&nbsp; mov ebp, 0000001A &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; //做除数用
:0040506D 0FBE4F04 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; movsx ecx, byte ptr [edi+04] &nbsp;&nbsp; //注册码第 5 位->ECX
:00405071 F7D1 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; not ecx &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; //取反->ECX
:00405073 8A1F &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; mov bl, byte ptr [edi] &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; //注册码第 1 位->BL
:00405075 8D414F &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; lea eax, dword ptr [ecx+4F] &nbsp; //(第 5 位取反结果+4F)的地址->EAX
:00405078 83F050 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; xor eax, 00000050 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; //拿它与 50 做异或->EAX
:0040507B F7F6 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; div esi &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; //接着用 EAX 整除 1A,余数在 EDX
:0040507D 0FBEF3 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; movsx esi, bl &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; //注册码第 1 位->ESI
:00405080 F7D6 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; not esi &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; //取反->ESI
:00405082 8D4645 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; lea eax, dword ptr [esi+45] &nbsp; //(第 1 位取反结果+45)的地址->EAX
:00405085 83F053 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; xor eax, 00000053 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; //那它与 53 做异或->EAX
:00405088 80C241 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; add dl, 41 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; //余数变成 1 个大写字母?->DL
:0040508B 88542419 &nbsp;&nbsp;&nbsp;&nbsp; mov byte ptr [esp+19], dl &nbsp;&nbsp;&nbsp; //好了,这是真注册码的第 2 位->[ESP+19]
:0040508F 33D2 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; xor edx, edx &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; //再来
:00405091 F7F5 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; div ebp &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; //用 EAX(上次整除的商)整除 1A,余数放在 EDX 里
:00405093 8D4144 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; lea eax, dword ptr [ecx+44] &nbsp; //(第 5 位取反结果+44)地址->EAX
:00405096 83F058 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; xor eax, 00000058 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; //拿它与 58 做异或->EAX
:00405099 80C241 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; add dl, 41 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; //余数变成 1 个大写字母?->DL
:0040509C 8854241A &nbsp;&nbsp;&nbsp;&nbsp; mov byte ptr [esp+1A], dl &nbsp;&nbsp;&nbsp; //作为真注册码的第 3 位->[ESP+1A]
:004050A0 33D2 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; xor edx, edx &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; //GOGOGO(TANY 语) YUN~~(我)
:004050A2 F7F5 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; div ebp &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; //用 EAX 整除 1A,余数放 EDX
:004050A4 8D4659 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; lea eax, dword ptr [esi+59] &nbsp; //(注册码第 1 位取反结果+59)的地址->EAX
:004050A7 83F056 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; xor eax, 00000056 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; //拿它与 56 做异或->EAX
:004050AA 80C241 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; add dl, 41 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; //余数变成 1 个大写字母?
:004050AD 8854241B &nbsp;&nbsp;&nbsp;&nbsp; mov byte ptr [esp+1B], dl &nbsp;&nbsp;&nbsp; //作为真注册码的第 4 位->[ESP+1B]
:004050B1 33D2 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; xor edx, edx &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; //GOGOGO(TANY 语) ......(我)
:004050B3 F7F5 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; div ebp &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; //用 EAX(上次整除的商)整除 1A,余数放在 EDX 里
:004050B5 8D4156 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; lea eax, dword ptr [ecx+56] &nbsp; //(第 5 位取反结果+56)的地址->EAX
:004050B8 83F049 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; xor eax, 00000049 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; //拿它与 49 做异或->EAX
:004050BB 80C241 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; add dl, 41 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; //余数变成 1 个大写字母?
:004050BE 8854241D &nbsp;&nbsp;&nbsp;&nbsp; mov byte ptr [esp+1D], dl &nbsp;&nbsp;&nbsp; //作为真注册码的第 6 位->[ESP+1D]
:004050C2 33D2 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; xor edx, edx &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; //晕死你(TANY 语) -\_-|(我)
:004050C4 F7F5 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; div ebp &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; //用 EAX(上次整除的商)整除 1A,余数放在 EDX 里
:004050C6 8D4658 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; lea eax, dword ptr [esi+58] &nbsp; //(注册码第 1 位取反结果+58)的地址-EAX
:004050C9 83F044 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; xor eax, 00000044 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; //拿它与 44 做异或->EAX
:004050CC 80C241 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; add dl, 41 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; //余数变成 1 个大写字母?
:004050CF 8854241E &nbsp;&nbsp;&nbsp;&nbsp; mov byte ptr [esp+1E], dl &nbsp;&nbsp;&nbsp; //作为真注册码的第 7 位->[ESP+1E]
:004050D3 33D2 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; xor edx, edx &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; //...... ......
:004050D5 F7F5 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; div ebp &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; //用 EAX 整除 1A,余数放 EDX
:004050D7 8D4153 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; lea eax, dword ptr [ecx+53] &nbsp; //(第 5 位取反结果+53)的地址->EAX
:004050DA 8BCD &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; mov ecx, ebp &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; //1A->ECX
:004050DC 83F045 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; xor eax, 00000045 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; //EAX(商)与 45 做异或->EAX
:004050DF 80C241 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; add dl, 41 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; //余数变成 1 个大写字母?
:004050E2 8854241F &nbsp;&nbsp;&nbsp;&nbsp; mov byte ptr [esp+1F], dl &nbsp;&nbsp;&nbsp; //作为真注册码的第 8 位->[ESP+1F]
:004050E6 33D2 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; xor edx, edx &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; //:) :(
:004050E8 F7F1 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; div ecx &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; //用 EAX 整除 1A,余数放 EDX
:004050EA 8D4650 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; lea eax, dword ptr [esi+50] &nbsp; //(注册码第 1 位取反结果+50)的地址->EAX
:004050ED 8BF5 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; mov esi, ebp &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; //1A->ESI
:004050EF 83F04F &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; xor eax, 0000004F &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; //EAX(商)与 4F 做异或->EAX
:004050F2 8BCA &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; mov ecx, edx &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; //余数->ECX
:004050F4 33D2 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; xor edx, edx &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; //EDX 清 0
:004050F6 F7F6 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; div esi &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; //EAX(商)整除 1A,余数放 EDX
:004050F8 80C141 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; add cl, 41 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; //真注册码的第 9 位->CL
:004050FB 80C241 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; add dl, 41 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; //真注册码的第 10 位->DL
:004050FE 80FB41 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; cmp bl, 41 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; //注册码第 1 位在 A-Z 之间?
:00405101 0F8CFD000000 &nbsp; jl 00405204
```



```
:00405107 80FB5A              cmp bl, 5A

:0040510A 0F8FF4000000        jg 00405204

:00405110 8A5F01              mov bl, byte ptr [edi+01]    //注册码第 2 位->BL 且在 A-Z 之间?

:00405113 80FB41              cmp bl, 41

:00405116 0F8CE8000000        jl 00405204

:0040511C 80FB5A              cmp bl, 5A

:0040511F 0F8FDF000000        jg 00405204

:00405125 8A4702              mov al, byte ptr [edi+02]    //注册码第 3 位在 A-Z 之间?

:00405128 3C41                cmp al, 41

:0040512A 88442413            mov byte ptr [esp+13], al

:0040512E 0F8CD0000000        jl 00405204

:00405134 3C5A                cmp al, 5A

:00405136 0F8FC8000000        jg 00405204

:0040513C 8A4703              mov al, byte ptr [edi+03]    //注册码第 4 位在 A-Z 之间?

:0040513F 3C41                cmp al, 41

:00405141 88442417            mov byte ptr [esp+17], al

:00405145 0F8CB9000000        jl 00405204

:0040514B 3C5A                cmp al, 5A

:0040514D 0F8FB1000000        jg 00405204

:00405153 8A4704              mov al, byte ptr [edi+04]    //注册码第 5 位在 A-Z 之间?

:00405156 3C41                cmp al, 41

:00405158 0F8CA6000000        jl 00405204

:0040515E 3C5A                cmp al, 5A

:00405160 0F8F9E000000        jg 00405204

:00405166 8A4705              mov al, byte ptr [edi+05]    //注册码第 6 位在 A-Z 之间?

:00405169 3C41                cmp al, 41

:0040516B 88442415            mov byte ptr [esp+15], al

:0040516F 0F8C8F000000        jl 00405204

:00405175 3C5A                cmp al, 5A

:00405177 0F8F87000000        jg 00405204

:0040517D 8A4706              mov al, byte ptr [edi+06]    //注册码第 7 位在 A-Z 之间?

:00405180 3C41                cmp al, 41

:00405182 88442412            mov byte ptr [esp+12], al

:00405186 7C7C                jl 00405204

:00405188 3C5A                cmp al, 5A

:0040518A 7F78                jg 00405204

:0040518C 8A4707              mov al, byte ptr [edi+07]    //注册码第 8 位在 A-Z 之间?

:0040518F 3C41                cmp al, 41

:00405191 88442414            mov byte ptr [esp+14], al

:00405195 7C6D                jl 00405204

:00405197 3C5A                cmp al, 5A

:00405199 7F69                jg 00405204

:0040519B 8A4708              mov al, byte ptr [edi+08]    //注册码第 9 位在 A-Z 之间?

:0040519E 3C41                cmp al, 41

:004051A0 88442416            mov byte ptr [esp+16], al

:004051A4 7C5E                jl 00405204

:004051A6 3C5A                cmp al, 5A

:004051A8 7F5A                jg 00405204

:004051AA 8A4709              mov al, byte ptr [edi+09]    //注册码第 10 位->AL 且在 A-Z 之间?

:004051AD 3C41                cmp al, 41

:004051AF 7C53                jl 00405204

:004051B1 3C5A                cmp al, 5A

:004051B3 7F4F                jg 00405204

:004051B5 3A5C2419            cmp bl, byte ptr [esp+19]    //真假注册码第 2 位比较

:004051B9 7549                jne 00405204

:004051BB 8A5C2413            mov bl, byte ptr [esp+13]    //假注册码第 3 位->[ESP+13]

:004051BF 3A5C241A            cmp bl, byte ptr [esp+1A]    //真假注册码第 3 位比较

:004051C3 753F                jne 00405204

:004051C5 8A5C2417            mov bl, byte ptr [esp+17]    //假注册码第 4 位->[ESP+17]

:004051C9 3A5C241B            cmp bl, byte ptr [esp+1B]    //真假注册码第 3 位比较

:004051CD 7535                jne 00405204

:004051CF 8A5C2415            mov bl, byte ptr [esp+15]    //假注册码第 6 位->[ESP+15]

:004051D3 3A5C241D            cmp bl, byte ptr [esp+1D]    //真假注册码第 6 位比较

:004051D7 752B                jne 00405204

:004051D9 8A5C2412            mov bl, byte ptr [esp+12]    //假注册码第 7 位->[ESP+12]

:004051DD 3A5C241E            cmp bl, byte ptr [esp+1E]    //真假注册码第 7 位比较

:004051E1 7521                jne 00405204

:004051E3 8A5C2414            mov bl, byte ptr [esp+14]    //假注册码第 8 位->[ESP+12]

:004051E7 3A5C241F            cmp bl, byte ptr [esp+1F]    //真假注册码第 8 位比较

:004051EB 7517                jne 00405204

:004051ED 384C2416            cmp byte ptr [esp+16], cl    //真假注册码第 9 位比较

:004051F1 7511                jne 00405204

:004051F3 3AC2                cmp al, dl                   //真假注册码第 10 位比较

:004051F5 750D                jne 00405204                 //想混过去没门(TANY 语)

:004051F7 33C0                xor eax, eax                 //oo 混过来了:)

:004051F9 5F                  pop edi

:004051FA 5E                  pop esi

:004051FB 5D                  pop ebp

:004051FC 5B                  pop ebx

:004051FD 81C408010000        add esp, 00000108

:00405203 C3                  ret
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

```
:00405101(C), :0040510A(C), :00405116(C), :0040511F(C), :0040512E(C)
:00405136(C), :00405145(C), :0040514D(C), :00405158(C), :00405160(C)
:0040516F(C), :00405177(C), :00405186(C), :0040518A(C), :00405195(C)
:00405199(C), :004051A4(C), :004051A8(C), :004051AF(C), :004051B3(C)
:004051B9(C), :004051C3(C), :004051CD(C), :004051D7(C), :004051E1(C)
:004051EB(C), :004051F1(C), :004051F5(C)
```

```
:00405204 5F                  pop edi                        //晕~~~没混过来:(

:00405205 5E                  pop esi

:00405206 5D                  pop ebp

:00405207 83C8FF              or eax, FFFFFFFF

:0040520A 5B                  pop ebx

:0040520B 81C408010000        add esp, 00000108

:00405211 C3                  ret
```

完活-\_-,这是我第一次也是最后一次写详细的了，完全是为了初学者，自己努力吧:)

这个玩艺破解用 10 分钟，写出来花几个小时，我晕~~~~~~

部分注册码：(用户名任意)

`Name:ZOMBIE`

`Code:RNFKCCNNWW`

`Code:ALCKENLEUN`

———————————————————————————————>

```
        .-"     "-.     Cracked by zombieys     >
        /          \     My oicq is 1789655      >
       |            |   My hp is zombieys.yeah.net>
       |, .-. .-. ,|   My hp is zombieys.126.com >
         |)(\_\_/  \\\_\_)(|                        >
         |/   /\\   \\|     Thanks for your supports >
     (@\_)@)   (\_   ^^   \_)                        >
   \_ )\\\_\_\_\_\_\_\_\\\_\_|IIIIII|\_\_/\_\_\_\_\_        >
\_)@8@8{}<\_\_\_\_\_\_\_\_|-\\IIIIII/-|\_\_\_\_zombieys\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_>
```

4dd466d0e1e0cc123afbe504da2462ca
