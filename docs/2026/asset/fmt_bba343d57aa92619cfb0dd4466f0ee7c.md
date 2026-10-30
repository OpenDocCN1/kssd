
基本上来说，有一个计数器用来记住发送到键盘中的固件包的数量。如果计数在10到100之间，固件将会处理被损坏的固件并且调用函数来修复他。

修复过程也不难，就是和一个52字节的秘钥进行异或。

![](https://bbs.kanxue.com/upload/attach/201604/302697_d87yzjg2kkisy9i.png)

我快速的用C写了一个加密和解密的函数来模拟这个过程。并将其加入到之前的代码中。

![](https://bbs.kanxue.com/upload/attach/201604/302697_k8t8l2ey1tmyc0p.png)

现在也可以解密固件更新程序为要写入`flash`的数据。生成一个文件并将其发送给`flash`。

### 五、加入贪吃蛇程序

随着之前工作完成，是时候编写贪吃蛇了。首先，我必须在`flash`和`RAM`中找到一个无用的存储区来存储我自己的代码和数据。这并不难：可以很容易的找到一个不少于`64k`的空闲`flash`和`28k`的`RAM`。为了修改键盘功能，我修改原始固件中的变量：比如`PEM`值和`RAM`中按下的键值数组。通过使用`JTAG/SWD`找到`RAM`的这种变量并不难。我在固件中加入一些`HOOK`使得我的代码能够执行。比如原始固件有一个功能是当按键被按下时调用，如果我对这个动作感兴趣，那就可以重定向到我自己函数执行，之后再调用原函数执行。

以下是我加入的`Hook`函数表。

![](https://bbs.kanxue.com/upload/attach/201604/302697_mexjhveya5u2j5b.png)

在这些攻击完成了后，我最终可以在键盘上玩贪吃蛇游戏了。

下面张图是我添加的贪吃蛇的方向控制。

![](https://bbs.kanxue.com/upload/attach/201604/302697_ckl6ud95tsm7xma.png)

### 六、扩展

固件中还有一些东西我做了改善，首先，允许键盘这样更新有一个安全问题。当`Coolermaster`固件更新自身时，会有一个窗口通知固件更新操作，比如我的程序运行`linux`下默认是`root`权限的用户，攻击者会在用户不知情的情况下很容易的重新刷入固件。比如在固件中刷入键盘记录器或者进行类似于`BadUSB`的攻击。为了防止这种情况发生，我在我的固件中加入一个物理条件来开启`flash`模式。具体来说，你需要在按下组合键`fn+f`后10秒钟的时候才能进行固件更新。否则，更新会失败。

![](https://bbs.kanxue.com/upload/attach/201604/302697_xs3ao04jrltow8l.png)

另外，就是关于按键效果自定义和游戏自定义问题目前还没有，但是我想很多人想要写入自定义效果但是并不想拆开他们的键盘通过`bootloader`来修复损坏的固件。我发现键盘的`USB`功能是被禁用了的，因此我决定重用这个功能：在我的固件程序中，通过一定的`cmd`包可以用来覆盖掉固件中设置的`LED`信号值。我还写了一个`PC`上的`demo`来模拟(这个模拟程序作者好像没有提供)。

### 七、总结

在`Rapid-I`(这次`hack`的工具包)中，默认固件并不是一成不变的，你可以用`keyboard`做比`Coolermaster`考虑得更多的事情。采用同样`hack`攻击技术，你除了用来玩儿贪吃蛇游戏，任何程序员理论上来说可以让自己想象任何东西运行在键盘上。我甚至希望是它变得更安全些：加入物理组合键开启更新的功能防止恶意攻击键盘。

一般来说，我一般写的代码都是开源的：固件分割和更新工具使用是请遵循`GPLv3`协议，贪吃蛇程序和其它代码请遵循`Beer-Ware`协议。因为版权法律原因我不得不清除所有`Coolermaster`的所有代码和二进制程序。但是你可以通过到官方下载更新程序来提取和重建。这次`hack`，仅仅使用`USB`键盘作为测试，因为我的键盘就是这个版本。

源码可以在附件下载。

[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)

上传的附件：

- [0.png](javascript:void(0);)（259.47kb，5次下载）
- [1.jpg](javascript:void(0);)（18.20kb，3次下载）
- [2.jpg](javascript:void(0);)（20.39kb，3次下载）
- [3.jpg](javascript:void(0);)（29.82kb，4次下载）
- [4.jpg](javascript:void(0);)（16.64kb，4次下载）
- [5.jpg](javascript:void(0);)（68.81kb，3次下载）
- [6.jpg](javascript:void(0);)（64.94kb，3次下载）
- [7.jpg](javascript:void(0);)（16.54kb，3次下载）
- [8.jpg](javascript:void(0);)（34.44kb，3次下载）
- [9.jpg](javascript:void(0);)（25.85kb，3次下载）
- [10.jpg](javascript:void(0);)（51.92kb，5次下载）
- [11.jpg](javascript:void(0);)（29.12kb，3次下载）
- [12.jpg](javascript:void(0);)（38.53kb，3次下载）
- [13.jpg](javascript:void(0);)（10.49kb，4次下载）
- [14.jpg](javascript:void(0);)（44.70kb，3次下载）
- [15.jpg](javascript:void(0);)（31.77kb，3次下载）
- [16.jpg](javascript:void(0);)（64.46kb，6次下载）
- [17.jpg](javascript:void(0);)（26.44kb，3次下载）
- [18.jpg](javascript:void(0);)（32.13kb，4次下载）
- [19.jpg](javascript:void(0);)（18.77kb，4次下载）
- [20.png](javascript:void(0);)（1.58kb，3次下载）
- [21.png](javascript:void(0);)（13.28kb，6次下载）
- [22.png](javascript:void(0);)（7.59kb，5次下载）
- [23.png](javascript:void(0);)（7.35kb，3次下载）
- [24.png](javascript:void(0);)（2.64kb，4次下载）
- [rapidisnake.rar](javascript:void(0);)（20.49kb，44次下载）
