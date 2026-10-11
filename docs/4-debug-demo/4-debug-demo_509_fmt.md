
# 星际争霸母巢之战 1.08b

> 原文：[`www.pediy.com/kssd/pediy04/kanxue180.htm`](https://www.pediy.com/kssd/pediy04/kanxue180.htm)

改一个字节破解星际争霸母巢之战 1.08b 的光盘保护

近来朋友将正版星际争霸母巢之战升级至 1.08b 版后，要放光盘才能玩，真不爽，苦于网上也找不到补丁，于是要我帮忙看看。

拿来后，要先装星际争霸老版本，忘了问朋友要 `CD-KEY`，懒得问朋友要了，用 `TRW` 改一个跳转就过了，很简单，我就不罗嗦了。

再装母巢之战，这个游戏的光盘保护和其他的有些不太一样，但也只要改一处跳转即可，用 `TRW` 下 `bpx getdrivetypea`，运行游戏，拦断后，`BD *`，按数次 `F12` 和 `F10`，来到 `storm.dll` 的领空，发现关键比较点如下：

```
:15018675 6681F9001F              cmp cx, 1F00

:1501867A 740B                    je 15018687       ；就是这里，改成 JMP 即可

:1501867C 6681F90508              cmp cx, 0805
```

用 `HIEW` 找： `66 81 F9 00 1F 74 0B 66 81`

改为： `66 81 F9 00 1F EB 0B 66 81`

另外，注册表中的 `CD` 盘付改成你的安装盘，我这里安装在 `D:` 盘下

```
REGEDIT4

[HKEY_LOCAL_MACHINE\Software\Blizzard Entertainment\Starcraft\]

"StarCD"="D:"
```

最后将光盘上的 `INSTALL.EXE`（540M）拷贝到安装盘的根目录下，运行游戏，动画和背景音乐和光盘版一样。

搞定！

大波罗`[BCG]``[FCG]`

Company: `Beginner's Cracking Group`

`http://wdfdiablo.myetang.com`

2001.10.26  `4dd466d0e1e0cc123afbe504da2462ca`
