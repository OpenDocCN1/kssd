
# 万能五笔 2001

> 原文：[`www.pediy.com/kssd/pediy03/forum1063.htm`](https://www.pediy.com/kssd/pediy03/forum1063.htm)

I think Delete those keys may work for you !!

In Registry :

{0IvsUrCYRA}

SystemAppIDs

Under `System/System32` :

`access.ctl`

or just delete the keys in registry, Try it out yourself !!

## 万能五笔 2001 的跟踪

这万能五笔(万能码)从 2000a+ 改版到 2001 时, 注册方法都改了.

我下载的好像没有加壳, 故不知为甚麽会有人问脱壳的问题!?

这程式有 200 次计算器指示, 我想是否在 200 次内都不能破解, 这程式便会自动毁灭, 至於那个动手破解的人.............!

在那里下载, 用甚麽工具我想不用说了.

本想写详尽一点, 但找不到还原的地方, 包涵一下!!

先下 `bpx getdlgitemtexta` 程式会断在下面

:0048D350 FFD3 call ebx

用 `F10` 跟了一下, 提示错误在下面这个 `call` 里跳出

:0048D3A0 E8DBF4FFFF call 0048C880

跟入 `0048C880` 後还是 `F8` 一步一步走, 到下面

:0048CB85 3854041C cmp byte ptr \[esp+eax+1C\], dl

这里是测式前四个字符是否 `tt98`, 而测式後四个字符是否 `0615` 在下面

:0048CBE3 38441C1C cmp byte ptr \[esp+ebx+1C\], al

重要点是下面的 `0048CCFA`

```
:0048CCF3 E828FBFFFF call 0048C820
:0048CCF8 8BE8 mov ebp, eax
:0048CCFA 2BEE sub ebp, esi <------
:0048CCFC 85ED test ebp, ebp
:0048CCFE 7429 je 0048CD29
```

这 `call 0048C820` 返回名字的计算值，再跟 `esi` 比较，相等等于 `OK`!

```
:0048CC65 E8064E0000 call 00491A70
:0048CC6A 66833D286D490001 cmp word ptr \[00496D28\], 0001
:0048CC72 8BF0 mov esi, eax <-------
:0048CC74 7559 jne 0048CCCF
```

这 `call 00491A70` 返回 ID 的计算值, 送到 `esi`。

`ID` 是--> `tt98xx...x0615`

`xx...x = xn....x3,x2,x1,x0` 是 ID 的计算值

计算方法：返回名字的计算值=`(xn*A^n)+(x5*186A0)+(x4*2710)+(x3*3E8)+(x2*64)+(x1*A)+(x0*1)`

如果名字的计算值=`2BFC3`

```
2BFC3 - 1*186A0 = 13923 --->x5=1
13923 - 8*2710 = A3 --->x4=8
A3 - 0*3E8 = A3 --->x3=0
A3 - 1*64 = 3F --->x2=1
3F - 6*A = 3 --->x1=6
3 - 3*1 = 0 --->x0=3
```

`ID` `tt981801630615` `4dd466d0e1e0cc123afbe504da2462ca`
