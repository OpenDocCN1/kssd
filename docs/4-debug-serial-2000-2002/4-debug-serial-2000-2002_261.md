
# `cpucool` 6.1x

> 原文：[`www.pediy.com/kssd/pediy03/forum774.htm`](https://www.pediy.com/kssd/pediy03/forum774.htm)

很早以前就想拿 `cupcool` 练练，今天在来试试，果然有点眉目了！可是还是不理想。请高手们赐教哦！

填好注册码 `123456789`

下断：

`bpm 016f：006eecb8`

按几下 `f10` 来到这：

```
:0041CE07 8B4C241C              mov ecx, dword ptr [esp+1C] *你输入的注册码，ecx＝075bcd15(十六进制的)

:0041CE0B BF83000000            mov edi, 00000083

:0041CE10 8BC1                  mov eax, ecx

:0041CE12 83C40C                add esp, 0000000C

:0041CE15 99                    cdq

:0041CE16 F7FF                  idiv edi *做除法，余数在 edx 中。

:0041CE18 5F                    pop edi

:0041CE19 85D2                  test edx, edx *余数为零？那你就成功了！

:0041CE1B 0F85EF000000          jne 0041CF10

:0041CE21 B83DCB883E            mov eax, 3E88CB3D

:0041CE26 F7E9                  imul ecx

:0041CE28 C1FA05                sar edx, 05 *edx＝商

:0041CE2B 8BC2                  mov eax, edx

:0041CE2D C1E81F                shr eax, 1F

:0041CE30 03D0                  add edx, eax

:0041CE32 81FA83000000          cmp edx, 00000083 *是不是 83 的倍数呢？想蒙混过关？没门！呵呵...

:0041CE38 0F8ED2000000          jle 0041CF10

:0041CE3E 81F980470100          cmp ecx, 00014780 *这不是黑名单吗？14780＝83840 十进制的

:0041CE44 0F8489000000          je 0041CED3 *这里跳到警告框了，挺吓人的。

:0041CE4A 8BC1                  mov eax, ecx

:0041CE4C B97F000000            mov ecx, 0000007F

:0041CE51 99                    cdq

:0041CE52 F7F9                  idiv ecx *再被 7f 除

:0041CE54 85D2                  test edx, edx *余数是零吗？

:0041CE56 747B                  je 0041CED3 *跳到警告框了！

:0041CE58 8D542474              lea edx, dword ptr [esp+74]

:0041CE5C B9A4064A00            mov ecx, 004A06A4

:0041CE61 52                    push edx

:0041CE62 881DF3074A00          mov byte ptr [004A07F3], bl

:0041CE68 E867C10400            call 00468FD4
```

下面我们来自己做个注册码吧！找几个数乘以 `131` 吧。比如 `641*131＝83971`，填入试试吧！别用 `640*131` 啊！为什么？你试试便知！呵呵！可是有个问题。下次运行时还是提示未注册！百思不得其解！请高手指点一二。小弟在此先谢过了！！！

小球[`CCG`]

程序启动时有另一套判断程序，我做的是 `cpucool6.0.9`，在注册后，它在 `4a1fbb` 置 `1`，表示已注册，下 `bpm 4a1fbb`，重新启动 `cpucool`，然后很快就能找到它的判断点，是这里

```
440305 jz 440403 ->nop  //跳过去的话就 die 了，所以 nop 掉它，没空细分析，好忙:(

44030b mov byte ptr [4a1fbb],01
```

`ok` 了

`cracked by zombieys`

`my oicq is 1789655`

`my hp is zombieys.126.com`

`thanks for your supports`

经过反复的试验，发现程序在开始的时候，也要检验我输的注册码。只不过是又经过烦琐的计算，我很难算出来！（我是不行了）不过我发现了一点算码的踪迹！希望有高人写写注册机！

为大众造福哦！

运行程序，输入注册码！随便吧，比如：`83971`，`85150`，`85281`。这里我输入 `85281`。

注册成功！退出程序。用 `ice` 或 `trw` 把它载入！`load` 它！

按 `F10` 来到这里：

```
:0045427A FF15A8424700          Call dword ptr [004742A8]

:00454280 50                    push eax

:00454281 E8F1F90000            call 00463C77  *这里进入，不然会出错的！

:00454286 8945A0                mov dword ptr [ebp-60], eax

:00454289 50                    push eax

:0045428A E829F1FFFF            call 004533B8

:0045428F 8B45EC                mov eax, dword ptr [ebp-14]

:00454292 8B08                  mov ecx, dword ptr [eax]
```

由上面 `call` 到此：

```
:00463C77 FF742410              push [esp+10]

:00463C7B FF742410              push [esp+10]

:00463C7F FF742410              push [esp+10]

:00463C83 FF742410              push [esp+10]

:00463C87 E8516A0000            call 0046A6DD *这里进入

:00463C8C C21000                ret 0010
```

到这儿：

```
:0046A6DD 53                    push ebx

:0046A6DE 56                    push esi

:0046A6DF 57                    push edi

:0046A6E0 83CBFF                or ebx, FFFFFFFF

:0046A6E3 E89FEDFFFF            call 00469487

:0046A6E8 8BF0                  mov esi, eax

:0046A6EA E867310000            call 0046D856

:0046A6EF FF74241C              push [esp+1C]

:0046A6F3 8B7804                mov edi, dword ptr [eax+04]

:0046A6F6 FF74241C              push [esp+1C]

:0046A6FA FF74241C              push [esp+1C]

:0046A6FE FF74241C              push [esp+1C]

:0046A702 E8C8470000            call 0046EECF

:0046A707 85C0                  test eax, eax

:0046A709 743B                  je 0046A746

:0046A70B 85FF                  test edi, edi

:0046A70D 740E                  je 0046A71D

:0046A70F 8B07                  mov eax, dword ptr [edi]

:0046A711 8BCF                  mov ecx, edi

:0046A713 FF9084000000          call dword ptr [eax+00000084]

:0046A719 85C0                  test eax, eax

:0046A71B 7429                  je 0046A746

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0046A70D(C)

|

:0046A71D 8B06                  mov eax, dword ptr [esi]

:0046A71F 8BCE                  mov ecx, esi

:0046A721 FF5050                call [eax+50] *这里进入，不然会出错的！

:0046A724 85C0                  test eax, eax

:0046A726 7515                  jne 0046A73D

:0046A728 8B4E1C                mov ecx, dword ptr [esi+1C]
```

按 `F10` 到这儿：

```
:0044A915 C7457840624800        mov [ebp+78], 00486240

:0044A91C E8BF3DFFFF            call 0043E6E0 *在由这里进入！

:0044A921 3BF7                  cmp esi, edi

:0044A923 7410                  je 0044A935

:0044A925 8BCE                  mov ecx, esi

:0044A927 E8447DFEFF            call 00432670

:0044A92C 56                    push esi

:0044A92D E863E30100            call 00468C95

:0044A932 83C404                add esp, 00000004
```

为什么要在 `0044a91c` 进入呢？如果是高手就会说：“凭感觉了！”其实不然。哪一个 `call` 要进，哪个不要进，是有一定的规律的。

就那本程序来说，有出错框的地方一定要进（废话）。出错的 `call` 上面的 `call` 你也要进！还有象 `0044a921` 处的 `call`，如果你按 `F10` 代过的话，程序将运行一段。你会怎么想？对，这里有可能是算码的 `call`！根据上面种种迹象表明你的猜想是 `80%` 正确的，（感觉？）那还犹豫什么？按 `F8` 进入吧！

总之了，关于该进入哪个 `call` 的问题，确实让人头疼。你只有不断的、反复的调用调试，才能摸透这个程序！嗨！说了半天，告诉你一个秘诀：一个字------------试！

呵呵！！！！！言归正传

由上面进入的庞大的算码过程！我倒.......

```
:0043E6E0 6AFF                  push FFFFFFFF

:0043E6E2 68D8284700            push 004728D8

:0043E6E7 64A100000000          mov eax, dword ptr fs:[00000000]

:0043E6ED 50                    push eax

:0043E6EE 64892500000000        mov dword ptr fs:[00000000], esp

:0043E6F5 81EC9C050000          sub esp, 0000059C

:0043E6FB A1B0D44900            mov eax, dword ptr [0049D4B0]

:0043E700 53                    push ebx

:0043E701 55                    push ebp

:0043E702 8BE9                  mov ebp, ecx

:0043E704 56                    push esi

:0043E705 57                    push edi

:0043E706 896C2424              mov dword ptr [esp+24], ebp

:0043E70A 89442418              mov dword ptr [esp+18], eax

:0043E70E 33DB                  xor ebx, ebx

:0043E710 8D8C2464010000        lea ecx, dword ptr [esp+00000164]

:0043E717 53                    push ebx

:0043E718 899C24B8050000        mov dword ptr [esp+000005B8], ebx

:0043E71F E82CF50000            call 0044DC50

:0043E724 53                    push ebx

:0043E725 8D8C24FC000000        lea ecx, dword ptr [esp+000000FC]

:0043E72C C68424B805000001      mov byte ptr [esp+000005B8], 01

:0043E734 E8A7E20000            call 0044C9E0

:0043E739 53                    push ebx

:0043E73A 8D4C2474              lea ecx, dword ptr [esp+74]

:0043E73E C68424B805000002      mov byte ptr [esp+000005B8], 02

:0043E746 E865AD0000            call 004494B0

:0043E74B 6A01                  push 00000001

:0043E74D C68424B805000003      mov byte ptr [esp+000005B8], 03

:0043E755 881DEA054A00          mov byte ptr [004A05EA], bl

:0043E75B 891DB4054A00          mov dword ptr [004A05B4], ebx

:0043E761 E806A50200            call 00468C6C

:0043E766 83C404                add esp, 00000004
```



:0043E769 8944241C `mov dword ptr [esp+1C], eax`

:0043E76D 3BC3 `cmp eax, ebx`

:0043E76F C68424B405000004 `mov byte ptr [esp+000005B4], 04`

:0043E777 740B `je 0043E784`

:0043E779 8BC8 `mov ecx, eax`

:0043E77B E8D07B0000 `call 00446350`

:0043E780 8BF0 `mov esi, eax`

:0043E782 EB02 `jmp 0043E786`

.
.
.

这里没有稿费，所以略去中间无数过程。为节省大家时间，一直按 `F10` 代过吧，到了这里：

:0043ED89 68E8654800 `push 004865E8`

:0043ED8E 56 `push esi`

:0043ED8F E869390100 `call 004526FD`

:0043ED94 8B4C2420 `mov ecx, dword ptr [esp+20]` *取到算码后的结果，本例中是：`3AF5CDFE=989187582`（十进制的）你把 `989187582` 改为 `989187419` 试试！*

:0043ED98 BE83000000 `mov esi, 00000083` *好像很熟哦！这不是 `131` 吗？（十进制）*

:0043ED9D 8BC1 `mov eax, ecx` *结果送到 `eax` 中做除数*

:0043ED9F 83C40C `add esp, 0000000C`

:0043EDA2 99 `cdq`

:0043EDA3 F7FE `idiv esi` *除法，余数在 `edx` 中。*

:0043EDA5 85D2 `test edx, edx` *又检验是否为零。*

:0043EDA7 7433 `jne 0043EDDC` *不能整除的跳走，当然是出错了！我猜不出注册码来。只有该这儿了。改为 `je`*

:0043EDA9 B83DCB883E `mov eax, 3E88CB3D`

:0043EDAE F7E9 `imul ecx`

:0043EDB0 C1FA05 `sar edx, 05`

:0043EDB3 8BC2 `mov eax, edx`

:0043EDB5 C1E81F `shr eax, 1F`

:0043EDB8 03D0 `add edx, eax`

:0043EDBA 81FAFF000000 `cmp edx, 000000FF`

:0043EDC0 7E1A `jle 0043EDDC`

:0043EDC2 51 `push ecx`

到次成功了！在启动试试！哈哈！注册版了吧！呵呵....

不对之处请您指教！

小球[CCG]
2001/4/7 4dd466d0e1e0cc123afbe504da2462ca
