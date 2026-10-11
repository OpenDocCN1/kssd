
# GIF Movie Gear 3.01

> 原文：[`www.pediy.com/kssd/pediy04/kanxue479.htm`](https://www.pediy.com/kssd/pediy04/kanxue479.htm)

## 破解 GIF Movie Gear 3.01

为了找一个汉化的对象，我找到了它。根据介绍，这个软件功能强大。我不会玩 `GIF` 动画，但在汉化过程中它给我的感觉倒是很好的，小巧的身材，强大的功能... 我喜欢这样的软件。在汉化中我顺便想把它注册了，于是今晚就请出了 `SICE` 把它给破了，呵呵...（也久没有 `CRACK` 了，为了搞安装程序的制作... 哪天我要写一个关于搞安装程序的制作的教程，我在网上找了好久都发现有...）

好了，言归正传。打开注册窗口填入：

`NAME`: `Vitamin C`

`CODE`: `1234abcd1234abcd`

(不知从哪天起喜欢用长的 `NUM.` 了...)

在 `SICE` 里用：`BPX HMEMCPY`。其实这个断点是最好用的，以前我不太懂 `CRACK` 时很怕用它，现在次次都用。（因为用 `GETWINDOWTEXTA` 等有时会拦不到...）...

按 `'OK'` 中断按 `F12` 后你会发现原来它是用 `GetWindowTextA` 的，按多几次 `F12`（我记不清是几次了，好像不是很多次 ^_^）后到达我们的目的地：

```
* Reference To: USER32.GetWindowTextA, Ord:015Eh

:00431959 8B1DF8834400 mov ebx, dword ptr [004483F8] /*它是把要 CALL 的偏移放入 EBX
:0043195F 50 push eax 再调用的*/
:00431960 FFD3 call ebx /*这个 CALL 取 NAME*/
:00431962 8D9424C4000000 lea edx, dword ptr [esp+000000C4] <-我们要在这里停

* Possible Reference to Dialog: DialogID_0064

:00431969 6A64 push 00000064
:0043196B 52 push edx

* Possible Reference to Dialog: DialogID_0091, CONTROL_ID:0450, ""

:0043196C 6850040000 push 00000450
:00431971 56 push esi
:00431972 FFD7 call edi
:00431974 50 push eax
:00431975 FFD3 call ebx /*这个 CALL 取 CODE,和上面的 CALL 调用同一个偏移*/
:00431977 8D8424C4000000 lea eax, dword ptr [esp+000000C4]
:0043197E 8D4C2460 lea ecx, dword ptr [esp+60]
:00431982 50 push eax /*这里是的两个寄存器里有一个是你输入的 CODE 有一个
:00431983 51 push ecx 是 mvg21951736,呵呵不用我说了吧,可是它可不是*/
:00431984 E8F7FBFFFF call 00431580/*这个是关键的 CALL,这里是一个经典的组合,CALL
:00431989 83C408 add esp, 00000008 了之后来一个 TEST EAX,EAX,呵呵...^_^*/
:0043198C 85C0 test eax, eax
:0043198E 0F84AD000000 je 00431A41
:00431994 8D542410 lea edx, dword ptr [esp+10]
```

试一试用 `F10` 跳过关键 `CALL`，`EAX` 就为 `00H`，(那个 `CODE` 是乱来的吗！) 然后就完了...

那么就是要跟进那个 `CALL`：

```
* Referenced by a CALL at Addresses:

|:004316C9 , :00431984

:00431580 53 push ebx
:00431581 55 push ebp/*EBP 是你输入的 CODE 的地址*/
:00431582 8B6C2410 mov ebp, dword ptr [esp+10]
:00431586 56 push esi
:00431587 57 push edi
:00431588 807D006D cmp byte ptr [ebp+00], 6D/*CODE 和第一个字符是:m*/
:0043158C 0F85A0000000 jne 00431632/*跳了就 OVER,下面几个一样的,知道是怎么回事了吧?*/
:00431592 807D0167 cmp byte ptr [ebp+01], 67/*第二个:g*/
:00431596 0F8596000000 jne 00431632
:0043159C 807D0233 cmp byte ptr [ebp+02], 33/*第三个:3*/
:004315A0 0F858C000000 jne 00431632
:004315A6 807D0337 cmp byte ptr [ebp+03], 37/*第四个:7*/
:004315AA 0F8582000000 jne 00431632

* Possible Indirect StringData Ref from Data Obj ->"mvg21951736" <-这个字符串不知有什么用?

:004315B0 BBC4D44400 mov ebx, 0044D4C4

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004315D6(C)

:004315B5 8B13 mov edx, dword ptr [ebx]
:004315B7 83C9FF or ecx, FFFFFFFF
:004315BA 8BFA mov edi, edx
:004315BC 33C0 xor eax, eax
:004315BE F2 repnz
:004315BF AE scasb
:004315C0 F7D1 not ecx
:004315C2 49 dec ecx
:004315C3 8BFA mov edi, edx
:004315C5 8BF5 mov esi, ebp
:004315C7 33C0 xor eax, eax
:004315C9 F3 repz
:004315CA A6 cmpsb
:004315CB 7465 je 00431632
:004315CD 83C304 add ebx, 00000004
:004315D0 81FBC8D44400 cmp ebx, 0044D4C8
:004315D6 7CDD jl 004315B5
:004315D8 807D0473 cmp byte ptr [ebp+04], 73/*第五个字符:s(不是必须的,另有用处)*/
:004315DC 7501 jne 004315DF
:004315DE 45 inc ebp

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004315DC(C)

:004315DF 83C507 add ebp, 00000007 /*取 CODE 第 7 位后的字符!CODE 要>7 位啊!*/
:004315E2 55 push ebp
:004315E3 E8D0DD0000 call 0043F3B8/*这个 CALL 其实作用是这样的:把 CODE 第 8 位起的字符
:004315E8 8B542418 mov edx, dword ptr [esp+18] '1234'->1234D*/
:004315EC 83C404 add esp, 00000004
:004315EF 8BFA mov edi, edx
:004315F1 33C9 xor ecx, ecx
:004315F3 8A12 mov dl, byte ptr [edx]/*这里开始是用 NAME 计算 CODE 了*/
:004315F5 BEDF0B0000 mov esi, 00000BDF
:004315FA 84D2 test dl, dl
:004315FC 7426 je 00431624

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00431622(C)

:004315FE 0FBED2 movsx edx, dl
:00431601 41 inc ecx
:00431602 0FAFD1 imul edx, ecx
:00431605 03F2 add esi, edx
:00431607 81FEBE170000 cmp esi, 000017BE
:0043160D 7E06 jle 00431615
:0043160F 81EEBE170000 sub esi, 000017BE

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0043160D(C)

:00431615 83F90A cmp ecx, 0000000A/*到了名字的第 10 个字符则要跳出去了!*/
:00431618 7E02 jle 0043161C
:0043161A 33C9 xor ecx, ecx

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00431618(C)

:0043161C 8A5701 mov dl, byte ptr [edi+01]
:0043161F 47 inc edi
:00431620 84D2 test dl, dl
:00431622 75DA jne 004315FE

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:004315FC(C)

:00431624 3BF0 cmp esi, eax/*将用 NAME 计算得到的 CODE 和 CODE 第 8 位起的字符->数字进
:00431626 750A jne 00431632 行比较!*/
:00431628 5F pop edi
:00431629 5E pop esi
:0043162A 5D pop ebp
:0043162B B801000000 mov eax, 00000001/*EAX 置 01H*/
:00431630 5B pop ebx
:00431631 C3 ret

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

|:0043158C(C), :00431596(C), :004315A0(C), :004315AA(C), :004315CB(C)
|:00431626(C)

:00431632 5F pop edi
:00431633 5E pop esi
:00431634 5D pop ebp
:00431635 33C0 xor eax, eax/*听笨冬瓜兄说,这是我们的大敌!(EAX=00H)*/
:00431637 5B pop ebx
:00431638 C3 ret
```

知道了 `CODE` 前 4 位是：`mg37`，那就让它 `OVER` 我们把 `CODE` 改一改：`mg37abc1234`，好了，顺利到了：`004315E3` 这个 `CALL`，这个 `CALL` 其实作用是这样的：把 `CODE` 第 8 位起的字符（就是：`'1234'`）变成十进制的数字（就是：`1234D`）！我跟进了那个 `CALL` 但没能找出不所以然来，还有一点，变是第 8 位起的字符不能是字母，要不会成为一个负数的！我一开始就是这样，所以我走了弯路...



## 注册码计算规则

后续步骤是使用 `NAME` 计算得到一个数字，并与 `CODE` 第 8 位起的字符（如：`1234`）转换为十进制数字（即：`1234D`）进行比较。如果相等则验证通过。

例如，`NAME` 为 `Vitamin C` 时，计算结果为 `031DH`（即 `797D`）。因此，正确的 `CODE` 格式为：
*   `mg37xxx797`（`xxx` 可以为任何字符）
*   实际上还有一个 `CODE`，即第 5 位是 `s` 的情况：`mg37sxxx797`（`xxx` 可以为任何字符）。

这两个 `CODE` 分别对应不同的验证类型：
*   站点 `CODE`（`SITE LICENSE`）
*   个人 `CODE`（`GIF Movie Gear` 软件指定）

这就是非必须比较的作用所在。

## 算法说明

上述关于 `NAME` 的计算算法是可见的，并不复杂。若有能力，可以编写一个注册机。

**算法流程：**

针对 `Vitamin C` 为例：
1.  取每个字符的十六进制数乘以它所在的位（第一位 `*1`，第二位 `*2` ...）。
2.  将所有结果相加。
3.  加上 `0BDFH`。
4.  如果结果大于 `17BEH`，则减去 `17BEH`，直到结果小于 `17BEH`（即 `6078D`）。
5.  最终得到的数的十进制值即为该 `NAME` 对应的 `CODE` 部分。

**注意：** 计算名字时，最多只计算前 10 个字符，其余部分不参与计算。

## 示例数据

*   `NAME`: `Vitamin C`
*   `CODE`: `mg37xxx797` / `mg37sxxx797`（`xxx` 可以为任何字符）

```
Vitamin C[抗坏血酸].2002.2.18.HY.GD.CHI.
4dd466d0e1e0cc123afbe504da2462ca
```
