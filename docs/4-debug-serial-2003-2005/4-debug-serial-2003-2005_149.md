
# Gif Movie Gear 3.0.2

> 原文：[`www.pediy.com/kssd/pediy05/pediy50294.htm`](https://www.pediy.com/kssd/pediy05/pediy50294.htm)

一个动画制作软件（`Gif Movie Gear 3.0.2` 的 `PJ`）的 `PJ` 过程与注册码算法。献给初学者。

下载地址：`http://www.skycn.com/download.php?id=2418&url=http://ln.skycn.net/down/gmvgr30.exe`

此版本的注册与 `3.0` 相比有了较大的改进，再也不是 `3.0` 的明码比较了，它的注册码与你所输入的用户名有密切联系，可以通过用户名反推出注册码，当然也可以通过注册码推出用户名。所以要想得出真正的注册码需费一番周折。

软件介绍：它是一个非常优秀的图片处理工具，是对动态 `gif` 压缩效果非常满意，是一个图片处理的小精灵。超级动画制作程序。

工具：`TRW2000`

1、首先运行 `movgear.exe`，填入用户名和注册码。注意注册码的前四位必须为 `mg37`，原因你就看下文吧。（最好前五位是 `mg37s`，当然也无所谓了，这是我的习惯。）

2、再运行 `TRW2000`，设置中断 `BPX HMEMCPY`，

3、回到 `movgear.exe` 注册画面，按 `OK`，`TRW2000` 拦截成功后按 `F12` 按 `9` 次（如果我没记错的话）再按 `F10` 就可到达此处。代码比较多，有关注册码的算法我已经注明了，你仔细看下文吧。

```
:0043197C 6850040000             push 00000450
:00431981 56                   push esi
:00431982 FFD7                 call edi
:00431984 50                   push eax
:00431985 FFD3                 call ebx
:00431987 8D8424C4000000       lea eax, dword ptr [esp+000000C4]
:0043198E 8D4C2460             lea ecx, dword ptr [esp+60]
:00431992 50                   push eax
:00431993 51                   push ecx
:00431994 E8F7FBFFFF           call 00431590 此处的调用是计算你所输入的注册码的数值以及计算你的用户名的数字，然后进行二者比较。见 B 处
:00431999 83C408               add esp, 00000008
:0043199C 85C0                 test eax, eax
:0043199E 0F84AD000000         je 00431A51 失败就跳

下面是成功处。

:004319A4 8D542410             lea edx, dword ptr [esp+10]
:004319A8 8D44240C             lea eax, dword ptr [esp+0C]
:004319AC 52                   push edx
:004319AD 50                   push eax
:004319AE 6A00                 push 00000000
:004319B0 683F000F00           push 000F003F
:004319B5 6A00                 push 00000000
:004319B7 6814ED4400           push 0044ED14
:004319BC 6A00                 push 00000000
```

* Possible StringData Ref from Data Obj -> `"Software\\gamani\\GIFMovieGear\\2.0"`

```
:004319BE 68F8B34400           push 0044B3F8
:004319C3 6801000080           push 80000001
```

* Reference To: `ADVAPI32.RegCreateKeyExA`, Ord:015Fh

把你正确的用户名和注册码存入注册表中

```
:004319C8 FF150C804400         Call dword ptr [0044800C]
```

```
:00431590 53                   push ebx
:00431591 55                   push ebp
:00431592 8B6C2410             mov ebp, dword ptr [esp+10] 你所输入的注册码
:00431596 56                   push esi
:00431597 57                   push edi
:00431598 807D006D             cmp byte ptr [ebp+00], 6D
比较首字节是否为 m
:0043159C 0F85A0000000         jne 00431642 不等于就跳
:004315A2 807D0167             cmp byte ptr [ebp+01], 67
比较第二字节是否为 g
:004315A6 0F8596000000         jne 00431642 不等于就跳
:004315AC 807D0233             cmp byte ptr [ebp+02], 33
比较第三字节是否为 3
:004315B0 0F858C000000         jne 00431642 不等于就跳
:004315B6 807D0337             cmp byte ptr [ebp+03], 37
比较第四字节是否为 7
:004315BA 0F8582000000         jne 00431642 不等于就跳
```

* Possible Indirect StringData Ref from Data Obj -> `"mvg21951736"`

下面是比较字串是否为 `mvg21951736`

```
:004315C0 BBC4D44400           mov ebx, 0044D4C4
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address: `:004315E6(C)`

```
:004315C5 8B13                 mov edx, dword ptr [ebx]
:004315C7 83C9FF               or ecx, FFFFFFFF
:004315CA 8BFA                 mov edi, edx
:004315CC 33C0                 xor eax, eax
:004315CE F2                   repnz
:004315CF AE                   scasb
:004315D0 F7D1                 not ecx
:004315D2 49                   dec ecx
:004315D3 8BFA                 mov edi, edx
:004315D5 8BF5                 mov esi, ebp
:004315D7 33C0                 xor eax, eax
:004315D9 F3                   repz
:004315DA A6                   cmpsb
:004315DB 7465                 je 00431642
:004315DD 83C304               add ebx, 00000004
:004315E0 81FBC8D44400         cmp ebx, 0044D4C8
:004315E6 7CDD                 jl 004315C5
:004315E8 807D0473             cmp byte ptr [ebp+04], 73
比较第五字节是否为 s
:004315EC 7501                 jne 004315EF 不等于就跳
:004315EE 45                   inc ebp 如果等于 ebp 加 1，也就是取你所输入的注册码第八位以后的字符，如果不等于就是取你所输入的注册码第七位以后的字符。
:004315EF 83C507               add ebp, 00000007
:004315F2 55                   push ebp
:004315F3 E8D0DD0000           call 0043F3C8 此处调用的作用是计算你所输入的注册码。见 A 处。
:004315F8 8B542418             mov edx, dword ptr [esp+18] 你所输入的用户名。
:004315FC 83C404               add esp, 00000004
:004315FF 8BFA                 mov edi, edx
:00431601 33C9                 xor ecx, ecx 为零
:00431603 8A12                 mov dl, byte ptr [edx]
:00431605 BEDF0B0000           mov esi, 00000BDF 设 ESI 的初始值为 BDF。
:0043160A 84D2                 test dl, dl 测试用户名是否为空
:0043160C 7426                 je 00431634
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address: `:00431632(C)`

```
:0043160E 0FBED2               movsx edx, dl 取用户名的一个字节 ASCII 值给 EDX
:00431611 41                   inc ecx 第几个字节
:00431612 0FAFD1               imul edx, ecx 二者相乘。
:00431615 03F2                 add esi, edx 乘积的高位字与 ESI 相加
:00431617 81FEBE170000         cmp esi, 000017BE 比较 ESI 是否大于 17BE
:0043161D 7E06                 jle 00431625 低于或等于就跳
:0043161F 81EEBE170000         sub esi, 000017BE 大于就减去 17BE
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address: `:0043161D(C)`

```
:00431625 83F90A               cmp ecx, 0000000A 比较循环数是否大于 A
:00431628 7E02                 jle 0043162C 低于或等于就跳
:0043162A 33C9                 xor ecx, ecx　ECX 为零
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address: `:00431628(C)`

```
:0043162C 8A5701               mov dl, byte ptr [edi+01] 取用户名的一个字节。
:0043162F 47                   inc edi
:00431630 84D2                 test dl, dl 比较用户名是否为空
:00431632 75DA                 jne 0043160E 不等于就继续循环。
循环完毕比较计算出来的你所输入的注册码的数值以及用户名的数值。
:00431634 3BF0                 cmp esi, eax
:00431636 750A                 jne 00431642 二者不等于就跳
:00431638 5F                   pop edi
:00431639 5E                   pop esi
:0043163A 5D                   pop ebp
:0043163B B801000000           mov eax, 00000001 如果相等就设置成功标志 EAX＝1
:00431640 5B                   pop ebx
:00431641 C3                   ret
```

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses: `:0043159C(C), :004315A6(C), :004315B0(C), :004315BA(C), :004315DB(C)`
* Referenced by a (U)nconditional or (C)onditional Jump at Address: `:00431636(C)`

```
:00431642 5F                   pop edi
:00431643 5E                   pop esi
:00431644 5D                   pop ebp
:00431645 33C0                 xor eax, eax 如果不相等就设置失败标志 EAX＝0
:00431647 5B                   pop ebx
:00431648 C3                   ret
```

```
A:
:0043F3C8 FF742404             push [esp+04]
:0043F3CC E86CFFFFFF           call 0043F33D 此处调用的作用是计算你所输入的注册码。见下文。
:0043F3D1 59                   pop ecx
:0043F3D2 C3                   ret
```

下文是计算你所输入的注册码。

```
:0043F33D 53                   push ebx
:0043F33E 55                   push ebp
:0043F33F 56                   push esi
:0043F340 57                   push edi
:0043F341 8B7C2414             mov edi, dword ptr [esp+14]
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address: `:0043F371(U)`

```
:0043F345 833D4CE2440001       cmp dword ptr [0044E24C], 00000001
:0043F34C 7E0F                 jle 0043F35D
:0043F34E 0FB607               movzx eax, byte ptr [edi]
```



```
:0043F351 6A08                   push 00000008
:0043F353 50                   push eax
:0043F354 E816230000           call 0044166F
:0043F359 59                   pop ecx
:0043F35A 59                   pop ecx
:0043F35B EB0F                 jmp 0043F36C
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  - `:0043F34C(C)`

```
:0043F35D 0FB607               movzx eax, byte ptr [edi]
```

* Possible StringData Ref from Data Obj -> `"((((" ... -> "H"`

```
:0043F360 8B0D40E04400         mov ecx, dword ptr [0044E040]
:0043F366 8A0441               mov al, byte ptr [ecx+2*eax]
:0043F369 83E008               and eax, 00000008
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  - `:0043F35B(U)`

```
:0043F36C 85C0                 test eax, eax
:0043F36E 7403                 je 0043F373
:0043F370 47                   inc edi
:0043F371 EBD2                 jmp 0043F345
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  - `:0043F36E(C)`

```
:0043F373 0FB637               movzx esi, byte ptr [edi]
:0043F376 47                   inc edi
:0043F377 83FE2D               cmp esi, 0000002D
:0043F37A 8BEE                 mov ebp, esi
:0043F37C 7405                 je 0043F383
:0043F37E 83FE2B               cmp esi, 0000002B
:0043F381 7504                 jne 0043F387
```

***** 下面是重点。 *****

```
:0043F383 0FB637               movzx esi, byte ptr [edi]
:0043F386 47                   inc edi
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  - `:0043F381(C)`

```
:0043F387 33DB                 xor ebx, ebx 设置 `EBX` 的初始值为零
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  - `:0043F3B8(U)`

```
:0043F389 833D4CE2440001       cmp dword ptr [0044E24C], 00000001
:0043F390 7E0C                 jle 0043F39E
:0043F392 6A04                 push 00000004
:0043F394 56                   push esi
:0043F395 E8D5220000           call 0044166F
:0043F39A 59                   pop ecx
:0043F39B 59                   pop ecx
:0043F39C EB0B                 jmp 0043F3A9
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  - `:0043F390(C)`

* Possible StringData Ref from Data Obj -> `"((((" ... -> "H"`

```
:0043F39E A140E04400           mov eax, dword ptr [0044E040]
:0043F3A3 8A0470               mov al, byte ptr [eax+2*esi]
:0043F3A6 83E004               and eax, 00000004
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  - `:0043F39C(U)`

```
:0043F3A9 85C0                 test eax, eax 测试是否循环完毕。
:0043F3AB 740D                 je 0043F3BA 相等就跳出循环。
```

下面就是计算你所输入的注册码的数值。

```
:0043F3AD 8D049B               lea eax, dword ptr [ebx+4*ebx]
; `eax = ebx + 4*ebx`

:0043F3B0 8D5C46D0             lea ebx, dword ptr [esi+2*eax-30]
; `ebx = esi + 2*eax - 30`

:0043F3B4 0FB637               movzx esi, byte ptr [edi] 取注册码的一个数值(ASCII 值)给 `ESI`。
:0043F3B7 47                   inc edi 加 1
:0043F3B8 EBCF                 jmp 0043F389
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  - `:0043F3AB(C)`

```
:0043F3BA 83FD2D               cmp ebp, 0000002D
:0043F3BD 8BC3                 mov eax, ebx 把计算出的数值传给 `EAX`。
:0043F3BF 7502                 jne 0043F3C3
:0043F3C1 F7D8                  neg eax
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
  - `:0043F3BF(C)`

```
:0043F3C3 5F                   pop edi
:0043F3C4 5E                   pop esi
:0043F3C5 5D                   pop ebp
:0043F3C6 5B                   pop ebx
:0043F3C7 C3                   ret
```

### 总结

此软件的注册码可以是 12 位。形式如下：`mg37s***abcd` 也可以是 `mg37***abcd`。其中 `*` 处可以是任意字符，而 `abcd` 所计算的数值必须与用户名的数值相等。

如：
- 用户名：`飞狐`
- 注册码：`mg37s1112630`

`飞狐` 的数值为 `A46`，而 `2630` 数值也为 `A46`，二者相等，注册成功。 `4dd466d0e1e0cc123afbe504da2462ca`
