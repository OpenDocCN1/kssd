
# Easy CD-DA Extractor 3

> 原文：[`www.pediy.com/kssd/pediy04/kanxue490.htm`](https://www.pediy.com/kssd/pediy04/kanxue490.htm)

## 破解 Easy CD-DA Extractor 3

终于，开始试一下去破解商业软件，说实在，一开始接触这些软件还有点不一样。怎么说那些 `CRACKME` 都大多只有反破解的功能，而那些商业的则里面还有好多不知什么功能的代码！一看上去还一时不能适应。

好了就啰唆到这了，开始进入正题：

安装好软件后（可没稿费的...）...

就是进入注册的界面，填入：

- `NAME`: `Vitamin C`
- `NUM.`: `1234ABCD` (只能大写)

在 `SICE` 里用 `BPX HMEMCPY`，`F5` 后，按'注册'按纽，就会进入程序里，用几次 `F12` 就可以进入核心：

...

失败，失败...我，我也想不起会到哪了，反正是一个 `CALL` 取 `NAME` 一个 `CALL` 取 `NUM.` 的...

反正是进去后会有好多的 `CALL` 一些我搞不明白的 `CALL`... 反正说了怕你笑话，我现在还不知道得到正确的 `NUM.` 后什么时侯 `CMP`，我只知道那个一看就知是 `NUM.` 的东东在哪产生的... 也罢了，就写出是哪几个 `CALL` 后会出现正确的 `NUM.` 吧... (反正是经过好漫长的计算才跑出来的，虽说是自动的出来，但我还是跟了好久好久)...

...

```
:00401350 6846654D00                 push 004D6546
:00401355 52                         push edx
:00401356 E8851E0B00                 call 004B31E0 /*这个 CALL 后就会产生 19A57A45- */
:0040135B 83C40C                     add esp, 0000000C
:0040135E 8D4DE0                     lea ecx, dword ptr [ebp-20]
:00401361 51                         push ecx
:00401362 8B450C                     mov eax, dword ptr [ebp+0C]
:00401365 50                         push eax
:00401366 E809E10A00                 call 004AF474 /*这个 CALL 把'EZCDDAX3-'接上去(它好像是天生的*/
:0040136B 83C408                     add esp, 00000008 /*得到:EZCDDAX3-19A57A45- */
:0040136E 6A20                       push 00000020
:00401370 8B5508                     mov edx, dword ptr [ebp+08]

...

:0040141B 8B550C                     mov edx, dword ptr [ebp+0C]
:0040141E 52                         push edx
:0040141F E850E00A00                 call 004AF474 /*这个 CALL 后变成:EZCDDAX3-19A57A45-BD9F92DE- */
:00401424 83C408                     add esp, 00000008
:00401427 33F6                       xor esi, esi

...

:0040147F 52                         push edx
:00401480 E8EFDF0A00                 call 004AF474 /*这个 CALL 后就全出来了:EZCDDAX3-19A57A45-BD9F92DE-360*/
:00401485 83C408                     add esp, 00000008
:00401488 5F                         pop edi
:00401489 5E                         pop esi
:0040148A 5B                         pop ebx
:0040148B 8BE5                       mov esp, ebp
```

...

那个 `NUM.` 放在和 `NAME` 不远的地方，在 `00E24BE0`，反正你一看就会想到它是 `NUM.`

这个 `NUM.` 是有来头的，开始我记录它的算法，经过了九牛二虎之力，才找出 `-19A57A45-` 的来头：

...

```
:00401326 8B4DEC                     mov ecx, dword ptr [ebp-14]
:00401329 8B45E4                     mov eax, dword ptr [ebp-1C]
:0040132C 0FAF4DF0                   imul ecx, dword ptr [ebp-10]
:00401330 0FAF4DF4                   imul ecx, dword ptr [ebp-0C]
:00401334 0FAF4DF8                   imul ecx, dword ptr [ebp-08]
:00401338 F76DE8                     imul [ebp-18]
:0040133B 0FAF4DFC                   imul ecx, dword ptr [ebp-04]
:0040133F 8B55E0                     mov edx, dword ptr [ebp-20]
:00401342 33C2                       xor eax, edx
:00401344 8D55E0                     lea edx, dword ptr [ebp-20]
:00401347 33C1                       xor eax, ecx
:00401349 69C033A1F594               imul eax, 94F5A133
:0040134F 50                         push eax /*EAX 就是它了*/
```

...

这段代码就是算 `-19A57A45-` 的，用了好多数，(也不知那些数是哪来的...)

其它的我就没有细看了。我几乎是用了 `D 00E24BE0` 后，一路用 `F10`，看着数据窗的变化就是了... 嘻嘻... 也给我看着那些 `NUM.` 一段一段自动跑出来.....

好了，就到这了，一个不成功的 `CRACK`。对了现在已经是年三十了... 明天又要老一岁！:-(

`NAME`: `Vitamin C`
`NUM.`: `EZCDDAX3-19A57A45-BD9F92DE-360`

`Vitamin C[抗坏血酸].2002.2.11.HY.GD.CHI. 4dd466d0e1e0cc123afbe504da2462ca`
