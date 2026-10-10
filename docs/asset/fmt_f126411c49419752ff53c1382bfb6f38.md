
# 谈谈 mac 机器和 Sun 的解密

> 原文：[`www.pediy.com/kssd/pediy06/pediy6531.htm`](https://www.pediy.com/kssd/pediy06/pediy6531.htm)

## 1、开篇

本人研究解密已经有 10 来年了，从未间断，可一直没有什么长进，

除了当年在 DOS 下脱个壳，解密个水浒 1，什么三国 1 之类。看看

周围的人都在搞什么.net,java 之类，NE,PE,PPC,asm 之类，大家

都在出书，什么大全，那个秘诀之类，感觉自己已经落后了许多，

Windows 已经被大家玩烂了，什么壳估计也存不了 1 年，什么狗 2 年

也要升级，关键是有系统级调试工具，而且资料也很齐全，执行文件格式

等相当齐全。我就把以前看过的一些资料整理一下，加上自己的一点

理解拿出来，让大家也熟悉其它的系统。先谈一下 MAC 吧，如果说错了

欢迎跟贴指正。

## 2、MAC 就是苹果机，现在可能是昨日黄花了，没办法他是一个封闭的操作系统

熟悉他必须有苹果机，一般的新机都要 1W，二手机都要 5K，而且软件多数

都是正版，一套上 W，这也限制了它的普及以及国内应用，如果向深入研究

当然最好有一部 MAC，当然是废话了。

苹果机的调试工具一般都是收费的，而且还需要两台，我可没有多余$，买一台

已经让老婆骂了（//sigh）。所以就只好用那个破 `macsbug` 了，这个工具我稍后讲

先讲一下我破 mac 的一般流程，用我的 PC(Win2K)没办法，还是 Win2k 的软件多，

先讲一下静态破解的吧

PC 上面安装 `PC MACLAN`，和苹果机网络共享的软件，可以方便存取 MAC 上的文件

比如有一个**软件（说了要被起诉的哦）

执行无狗就弹出对话框然后退出，有狗才能正常运行，（这是废话一堆）

用 `IDA4.30` 正式版本，btw demo 还不行，不能反编译 PPC 格式，打开这个执行文件，

```
seg000:00010168 .ExitToShell:                           # CODE XREF: sub_128568+B0p
seg000:00010168                                         # sub_144828+80p ...
seg000:00010168

seg000:00010168 .set arg_14,  0x14

seg000:00010168

seg000:00010168                 lwz     r12, TC_ExitToShell # ExitToShell
seg000:0001016C                 stw     rtoc, arg_14(sp)
seg000:00010170                 lwz     r0, 0(r12)
seg000:00010174                 lwz     rtoc, 4(r12)
seg000:00010178                 mtctr   r0
seg000:0001017C                 bctr
seg000:0001017C # End of function .ExitToShell
```

退出一般都调用 `ExitToShell` 函数就和 WIndows 调用 `CloseWindow`...只类差不多

那么看那个地方调用了呢

`sub_128568`

`sub_144828`

`sub_16ea98`

....

....

总共有 7 处，不多，一处处修改

```
seg000:00128618 loc_128618:                             # CODE XREF: sub_128568+84j
seg000:00128618                 bl      .ExitToShell
seg000:0012861C                 lwz     rtoc, 0x40+var_2C(sp)
```

找到类似上面的地方把

`bl      .ExitToShell`

替换成 `60000000` 就是 NOP 了，当你改到 `001809C8` 这个地方，哦程序终于不退出了，但是那个

讨厌的对话框还在，

```
seg000:001809BC

seg000:001809BC                 mflr    r0
seg000:001809C0                 stw     r0, arg_8(sp)
seg000:001809C4                 stwu    sp, var_38(sp)
seg000:001809C8                 bl      .ExitToShell
seg000:001809CC                 lwz     rtoc, 0x38+var_24(sp)
seg000:001809D0                 lwz     r0, 0x38+arg_8(sp)
seg000:001809D4                 addic   sp, sp, 0x38
seg000:001809D8                 mtlr    r0
seg000:001809DC                 blr
```

那还要检查哪里调用了 `001809BC`，一看是下面的

```
-------------------------
seg000:0012CDE0                 cmplw   cr1, r29, r31
seg000:0012CDE4                 bne     cr1, loc_12CE04  # 检查狗
seg000:0012CDE4                                         #
seg000:0012CDE8                 mr      r3, r31
seg000:0012CDEC                 li      r4, 1
seg000:0012CDF0                 bl      sub_12CB58  对话框
seg000:0012CDF4                 nop
seg000:0012CDF8                 bl      sub_1809BC 退出
seg000:0012CDFC                 nop
seg000:0012CE00                 b       loc_12CE48
seg000:0012CE04 # 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
seg000:0012CE04

seg000:0012CE04 loc_12CE04:                             # CODE XREF: seg000:0012CDDCj
seg000:0012CE04                                         # seg000:0012CDE4j
seg000:0012CE04                 cmpwi   r30, 0x3E9
seg000:0012CE08                 bne     loc_12CE30
seg000:0012CE0C                 cmplw   cr1, r29, r31
seg000:0012CE10                 bne     cr1, loc_12CE30
seg000:0012CE14                 mr      r3, r31
seg000:0012CE18                 li      r4, 1
seg000:0012CE1C                 bl      sub_12CB58  对话框
seg000:0012CE20                 nop
seg000:0012CE24                 bl      sub_1809BC 退出
seg000:0012CE28                 nop
```

讲到这里，大家就知道了把 `0012CDF0`,`0012CDF8`,`0012CE1C`,`0012CE24` 都修改为 NOP 就 OK 了

现在程序能够进入并且能够执行大部分的功能了。

其实就和 PC 里面的断点原理差不多，而且苹果机一般都没有壳，所以作起来相对容易一些

当然也看你对系统的熟悉程度了，如果大家喜欢，我就接着写。

进入系统后发现文件是可以正常编辑了，可是文件存盘功能不行，那个菜单按钮总是灰色的

用 `ResCafe` 看一下，菜单资源不是灰的 Enable，那么程序中一般都调用 `DisableItem` 函数来使能

菜单，同理，找到

```
seg000:00012340                 .csect  .DisableItem[GL]
seg000:00012340

seg000:00012340 # 圹圹圹圹圹圹圹?S U B R O U T I N E 圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹?
seg000:00012340

seg000:00012340

seg000:00012340 .DisableItem:                           # CODE XREF: sub_1863F0+58p
seg000:00012340                                         # sub_186464+5Cp
seg000:00012340

seg000:00012340 .set arg_14,  0x14

seg000:00012340

seg000:00012340                 lwz     r12, TC_DisableItem # DisableItem
seg000:00012344                 stw     rtoc, arg_14(sp)
seg000:00012348                 lwz     r0, 0(r12)
seg000:0001234C                 lwz     rtoc, 4(r12)
seg000:00012350                 mtctr   r0
seg000:00012354                 bctr
seg000:00012354 # End of function .DisableItem
```

`sub_1863F0`，和 `sub_186464` 两个位置 `seg000:00186428                 beq     cr1, loc_186440`

把 `beq` 修改为 NOP 之后，菜单已经能够不是灰色的了并且可以正常进入。

```
seg000:00186424                 cmpwi   cr1, r30, 0
seg000:00186428                 beq     cr1, loc_186440
seg000:0018642C                 mr      r3, r9
seg000:00186430                 lha     r4, 0(r31)
seg000:00186434                 bl      .EnableItem
seg000:00186438                 lwz     rtoc, 0x48+var_34(sp)
seg000:0018643C                 b       loc_186450
seg000:00186440 # 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
seg000:00186440

seg000:00186440 loc_186440:                             # CODE XREF: sub_1863F0+38j
seg000:00186440                 mr      r3, r9
seg000:00186444                 lha     r4, 0(r31)
seg000:00186448                 bl      .DisableItem
seg000:0018644C                 lwz     rtoc, 0x48+var_34(sp)
seg000:00186450

seg000:00186450 loc_186450:                             # CODE XREF: sub_1863F0+30j
```

上面的方法对付比较善良的 MAC 程序一般都有效，其实 DOS 下

以及 Linux,pocketpc,windows 等等都可以适用这种方法，君不见

目前注册码直接比较的软件满天飞么，就是软件作者总以为没有必要

当然如果程序比较恶，那么就需要动态调试以及静态分析程序结合了

谈到动态调试，一个好的调试工具必不可少。想想如果没有 sice，

没有 tr。。。。。等等，如果你对 `windowsSDK` 一无所知,那么来谈

Windows 的解密就是无从谈起，同样其他操作系统环境下的也一样

一般来说都是 SDK,静态分析工具，动态调试结合才能够直接处理

完绝大部分的问题，下篇讲一下 mac 的 SDK 吧，还是最后讲动态调试工具吧

毕竟这个东西有心搞得同学看看帮助也就差不多了，类似 Windows 下的

调试技术论坛(`www.pediy.com`) 并保持文章的完整性！

`4dd466d0e1e0cc123afbe504da2462ca`
