
# 如何配置好 Ollydbg

> 原文：[`www.pediy.com/kssd/pediy06/pediy6113.htm`](https://www.pediy.com/kssd/pediy06/pediy6113.htm)

## 五月话题总结

2003 年 5 月话题题目——如何配置好 `Ollydbg`

### 目录

- 话题的前言
- 话题的总评
- 话题的相关提问与解答
- 几个 `Ollydbg` 配置帖子

### 话题的前言

2003 年 5 月话题——如何配置好 `Ollydbg`

最近，`Ollydbg` 成为软件调试的新宠。然而 `Ollydbg` 相关的介绍却比较少，`Ollydbg` 许多问题亟待深入研究。本月的话题就聚焦于 `Ollydbg`。“工欲善其事，必先利其器”。一个好的程序配置，会让调试工作事半功倍。希望大家把自己 `Ollydbg` 配制中值得称道的地方公布出来。新手也可针对配置问题，提出问题。

本次话题的所有配置应基于 `Ollydbg` 1.09b 英文版，必须通过这个版本的测试

注意：由于 Clone 汉化的 `Ollydbg` 有多个版本（配置文件也被汉化），而且存在 Bug；因此本期话题所针对的是 `Ollydbg` 1.09b 英文版。请不要把汉化版的配置帖出来（兼容性太差）。您可以通过英文版 `Option` 菜单中的选项，对比着配置汉化版

**相关信息：**

1. `Ollydbg` 1.09b 英文版
2. 您可以通过菜单 `Option` 下相关选项，进行配置；也可以通过 `ollydbg.ini` 文件，进行配置。
3. 配置中各个选项的含义可以察看英文版自带的帮助。

### 话题的总评

本月的话题是每月话题这个栏目的第一期，涉及了 `Ollydbg` 的设置问题，应该说大家是比较踊跃的加入到话题的讨论中，`winroot`、`doskey`、`zmworm`、`jwh51` 等网友帖出了自己的配置，但我们的水平有限，加上众多高手没有加入到讨论中，使得讨论质量有所下降，一些话题的最初设想没有达到，比如一些配置选项为什么那么设置，他的好处是什么；比较重要的异常中断的问题也没有人提出，等等。但无论如何这次处女秀，算是勉强通过。`doskey` 的 `1024*768` 配置 以及 `zmworm` 的 `800*600` 配置 有一些值得称道的地方。在这个总结帖编写的时候 `Ollydbg` 1.09c 已经出来了（升级好快，不过本次话题提出的配置仍可以使用），不过 `Ollydbg` 2.0 要到明年才能出来，让我们耐心等待把^\_^

### 话题的相关提问与解答

**freeia**:

Q. 怎么把字体设置大点，小的看得不舒服：（

我的是 `1024*768`

**zmworm**:

A. `1024` 下字体可以这样设置

鼠标右击 选 `Appearance->fonts->system fixed font`

**xq**:

Q. 启动时，初始化的那些窗口是做什么用的？

**zmworm**:

A. 就是在启动时提前把所有的窗口都打开来了，并藏在主窗口后面。你也可以在使用时打开。

**李逍遥**:

Q. 插件都有些什么用，如何用，可以给我们菜鸟说说吗？

**zhb8811551**:

A. 呵呵，我知道了，

`李逍遥`，你在 `OD` 目录下建一个 `plugins` 目录，将下载的插件放到该目录中，然后选择

`options -> appearance -> directories`

在 `plugins path` 中选择刚才新建的目录即可！

然后重新启动 `OD`！

呵呵

**entertop**:

Q. 不知道 `winxp` 可以用吗？（问的菜了，不要笑啊！）

**A. zmworm**:

当然可以了，不过能用 `xp` 一般屏幕是 `1024` 的

我那个是 `800` 的配置，你还要自己改改

**code-inject**:

可以把 `ollydbg` 的标题，`class` 改改， `anti-anti`，`thought this tricks is easy to defeat`

The Modified `OD`：

[`crack.zuasoft.com/ut/attach/2003/05/22/6444-CoDeDBG.rar`](http://crack.zuasoft.com/ut/attach/2003/05/22/6444-CoDeDBG.rar)

对比文件就知道改了那些地方。有时间再写个插件把那些 `IsdebuggerPresent`, `Ntquerysysteminformation` 等函数 `XX` 掉就行了，没时间啊

`zworm` 有时间写吗？and about `doskey`?

也可以修改 `OD` 让加载的时候搞掉这些 。。。

**想入非非 1068**:

Q. 请改为自己的 `WinAPI` 帮助文件

```
API help file=Win32.hlp

Text save file=

;符号文件路径

Symbolic data path=Symbolic

[Import libraries]

;添加自己的输入连接库文件

Implib[0]=F:\VC\VC98\MFC\Lib\MFC42.LIB
```

这几个地方我看不懂，我是新手，请大家帮忙！

**zmworm**:

A. `"API help file=Win32.hlp"`

是 `WinAPI` 帮助文件的路径，设置好后遇到不清楚的 `API` 函数用 `Ctrl+F1`，就会显示出该 `API` 的 Reference(使用参考.

`"Implib[0]=F:\VC\VC98\MFC\Lib\MFC42.LIB"`

添加自己的 `Lib` 文件，这些你都可以在 `SDK` 中找到

**海楼**:

Q. 请教版主，我依照您的配置后（不好意思，我是原样照搬您的），插件不能用，而且 `UDD` 文件也没有保存在 `UDD` 目录下，请问如何解决！

**zmworm**:

A. 我写的是相对路径

如果 `UDD` 或 `Plugins` 目录有问题，可以使用绝对路径。

`4dd466d0e1e0cc123afbe504da2462ca`
