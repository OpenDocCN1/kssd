
# WinDbg 学习笔记（一）--认识 WinDbg

> 原文：[`www.pediy.com/kssd/pediy10/94457.html`](https://www.pediy.com/kssd/pediy10/94457.html)

## 一、前言

本人学习 `WinDbg` 已经有好几天了，虽说技术掌握的还不太熟练，不过也总算是入门了。在学习 `WinDbg` 的过程中，觉得 `WinDbg` 真的比 `OllyDbg` 难很多（不知道是不是习惯 `OllyDbg` 的缘故），所以就想做点笔记，那以后自己忘了也可以拿出来复习，也为同样在学 `WinDbg` 的兄弟提供一点帮助。文章写得有点嗦，初学者应该可以很快看完，老鸟就可以飘过了…

## 二、WinDbg 简介

`WinDbg` 是微软发布的一款免费而十分强大的调试工具。既然是微软自己发布的调试工具，那它对微软产品的调试当然是十分的强大。但虽说是功能强大，操作起来有时可能十分困难。有时候，可能 `WinDbg` 很多特别的功能我们还没知道，就更别说用得上了。我是个 `WinDbg` 的初学者，开始的时候觉得 `WinDbg` 真的很难入门，不过现在资料看多了，也就算是参悟了点。很多时候，我们习惯用 `OllyDbg` 或 `SoftICE`（`SoftICE` 我没用过），所以，也可以先从我们熟悉的入手，慢慢了解 `WinDbg`。

下面是 `http://bbs.dbgtech.net/` 里一些资料的截图，可以加深我们对 `WinDbg` 的了解。（本人没有侵犯原作者版权的意思，只是想着把资料整理出来方便大家看。如有冒犯，请见谅）

内核调试方面：

![](img/8a41a109b17517ca7ac8c2d8310a9653.jpg)

用户调试方面：

![](img/5e19117d128d2414e835da1f45ed101c.jpg)

![](img/4e72505c04da5a5b6152a1ce951d922b.jpg)

虽说 `WinDbg` 在无源码调试方面确实比较困难，但在调试内核方面却真的有独到之处。我跟大家一样，可能本身对 `OllyDbg` 比较熟悉，所以我会尝试跟着 `CCDebuger` 大牛所写的《`OllyDbg` 入门系列》的模式进行笔记的书写。

## 三、WinDbg 的安装与配置

`WinDbg` 包含在 `WDK` 里，也可以独立的下载。

官方的下载地址：[`www.microsoft.com/whdc/devtoo...g/default.mspx`](http://www.microsoft.com/whdc/devtools/debugging/default.mspx)

汉化版的下载地址：[`bbs.pediy.com/showthread.php?...ghlight=WinDbg`](http://bbs.pediy.com/showthread.php?t=81746&highlight=WinDbg)

下载下来的一般是 `WinDbg` 的安装程序，跟一般程序的安装过程类似。安装完后，若要使用汉化版本，就直接把汉化后的启动文件（例如 `WinDbg_cn.exe`）复制到安装目录下，再直接点击（`WinDbg_cn.exe`）即可。

启动后的界面类似：

![](img/c0c1d88249d279152487a78f5914951d.jpg)

到这里，可能会发现没有其它窗口。但其实，只要你点击对应的窗口按钮（上图红色方框圈着的区域），工作区就会出现相应的窗口。

`WinDbg` 有好几个窗口，功能跟 `OllyDbg` 的差不多。窗口的名称与用途如下：

| 名称 | 热键 | 用途 |
| :--- | :--- | :--- |
| `Command` | `Alt+1` | 输入命令、显示命令结果和调试信息输出 |
| `Watch` | `Alt+2` | 观察指令全局变量、局部变量和寄存器的信息 |
| `Locals` | `Alt+3` | 自动显示当前函数的所有局部变量 |
| `Registers` | `Alt+4` | 观察和修改寄存器的值 |
| `Memory` | `Alt+5` | 观察和修改内存数据 |
| `Call Stack` | `Alt+6` | 栈中记录的函数调用序列 |
| `Disassembly` | `Alt+7` | 反汇编 |
| `Scratch Pad` | `Alt+8` | 白板，可以用来做调试笔记等 |
| `Processes and Threads` | `Alt+9` | 显示所有调试目标的列表，包括进程和线程等 |
| `Command Browser` | `Alt+N` | 执行和浏览命令 |

在弹出需要的窗口后，就可以对窗口进行排列。通过鼠标拖曳窗口或结合右击窗口标题选择 `Dock`（停靠）和 `Always floating`（保持浮动），可以把窗口摆放成最适合我们工作的位置。

![](img/4c2c94c5b4c2b3ce60fb34424cb2651b.jpg)

不知道是不是我自己的问题，本来想着把窗口排列成跟 `OllyDbg` 的界面类似，可怎么弄也达不到目的，最后也就选择了两个我最常用的窗口（`Disassembly` 和 `Command`），然后上下摆放。就像：

![](img/43913be8e5faa5f74bedddbc23c27bde.jpg)

各位如果有摆放窗口的诀窍，麻烦也告之一下….

## 四、符号、源码路径设置

刚才也说过，`WinDbg` 是一款很强大的源码调试工具。如果我们有程序的源码的话，一般可以生成它的符号文件。而在用 `WinDbg` 进行调试时，因为 `WinDbg` 本身不会自动识别符号文件，所以需要我们设置。

可以通过打开菜单 `File > Symbol File Path` 和 `File > Source File Path`，打开对应的对话框进行设置。

![](img/d35d128063cdf3f9de53fd5f892d8c0d.jpg)

![](img/fcbb14d6f0ab078e20587dc32e582412.jpg)

可以直接在 `Source path` 编辑框输入目录，也可以通过按钮 `Browse` 进行选择。多个目录之间用分号隔开。

到这里，可能有人有疑问。符号文件是什么？为什么我们以前用 `OllyDbg` 时不用设置符号文件？

其实我接触到这里的时候也十分的迷惑，甚至当时以为 `WinDbg` 只能调试有源码的文件。但后来，通过试验，逐步清楚了。

按我理解，符号就是用来标识程序的某个函数、变量或标记的东西。例如，通过载入有源代码和程序符号的目标文件，就可以轻松的使用 `bp WinMain`（我用来测试的程序的主函数）和 `bp 函数名`（程序里的某个函数）。

那如果没有源代码怎么办？那只好使用微软的公共符号库。所谓公共符号库，可以理解成 `WinDbg` 对某些微软的系统函数（例如 `MessageBoxA`）的识别，就像 `OllyDbg` 能够识别出 `MessageBoxA` 一样。虽然 `WinDbg` 能够识别出这些系统函数，但却显示的不明显（到后面会接触到）。

要设置公共符号文件，只需在设置符号文件的编辑框里输入路径即可。一般的输入形式为：

`SRV*g:\\symbols*http://msdl.microsoft.com/download/symbols`

意思大概就是从 `http://msdl.microsoft.com/download/symbols` 的地址里下载符号到 `g:\\symbols`（自己设置用来保存符号的目录）。然后设置完可以使用 `.reload` 命令让 `WinDbg` 重新加载新的符号文件。具体的用法可以参考 `WinDbg` 的帮助文档。

## 五、工作空间

工作空间，是 `WinDbg` 用来保存当前工作环境的工具。例如可以保存当前工作的窗口布局、符号路径设置等，当再次载入时就不用重新设置。

例如，当前工作空间还没载入任何的调试文件，那么它就是 `WinDbg` 默认的打开环境。在这里的所有设置，包括窗口布局、符号源码路径设置等，都可以根据用户需要保存起来。`WinDbg` 提供多种保存方式，包括：

`Save Workspace` --- 直接保存（保存到默认文件）

`Save Workspace as` --- 另存为（保存到用户指定的文件）

上面两种方式所说的文件其实并不是真正的文件，而是 `WinDbg` 在注册表里存放工作空间的项。

注册表的位置是：`HKEY_CURRENT_USER\Software\Microsoft\Windbg\Workspaces`

这个键里一般包含四个子键 `User`、`Kernel`、`Dump`、`Explicit`，前 3 个子键用来保存用户态调试、内核态调试、调试转储文件时使用 `Save Workspace` 方式系统默认的保存路径，而 `Explicit` 则用来记录使用 `Save Workspace as` 方式保存的命名工作空间。有时或者会发现没有某个子键，这是因为你的 `WinDbg` 还没调试过使用该种调试方式的文件。

例如，我注册表 `HKEY_CURRENT_USER\Software\Microsoft\Windbg\Workspaces` 里有这些项：

![](img/dee3d8c2eb3162e8afd4e9ab1da5a96c.jpg)

那就说明，我的 `WinDbg` 目前还没进行过 `Dump` 文件的调试。

再展开 `User` 子键看看：

![](img/a9cbcc6fb355f27231c29b6ba57a5205.jpg)

可以看出，我在用户态调试过 3 个程序，它们分别是 `CrackMe3.exe`、`CrackMe32.exe` 和 `adsfsf.exe`，而且使用 `Save WorkSpace` 方式保存。

可能有人会问，有时后我调试完程序后没有选择 `Save WorkSpace` 而是直接关闭 `WinDbg` 也会自动的保存工作空间。这是因为你的 `WinDbg` 使用了“总是保存”的方式。

可以通过打开 菜单----`view`----`Options` 打开设置对话框：

![](img/d8f250abfeba66c96a63517d568538db.jpg)

图中用红色方框圈着的就是工作空间关闭时的提示。

选择 `【Always ask】` 在工作空间关闭时总是询问用户是否保存；

选择 `【Always save】` 在关闭时自动把工作空间保存起来；

选择 `【Never save】` 从不保存。

另外，`WinDbg` 还提供另外一种保存方式 `【Save WorkSpace to File】`，作用跟上面两种的方式差不多，只不过把工作空间以文件的形式保存到磁盘里，那么用户就可以通过 `U` 盘或其它方式把工作空间的环境移植到其它机器上使用。

在打开工作空间方面，当原本是使用默认的保存方式时，`WinDbg` 在打开目标文件的时候同时也会自动打开其对应的工作空间文件。

另外，在载入调试目标后改变了工作空间并保存后，并不改变 `WinDbg` 默认的启动环境，只改变调试目标对应的工作空间保存文件，而只有在 `WinDbg` 还没载入任何调试目标时改变的才会影响原来的启动环境。

## 六、常用操作与命令

![](img/50e9cb4917c5131a240e639d3c1c6689.jpg) `F9`：设置断点，只要在光标定位的位置（上图中灰色条）按 `F9` 键即可，再按一次 `F9` 键则会删除断点。（相当于 `OllyDbg` 中的 `F2`）

![](img/138c9a5e949e0ccee4043d9d96a56cbf.jpg) `F10`：单步步过。每按一次这个键执行一条反汇编窗口中的一条指令，遇到 `CALL` 等子程序不进入其代码。（相当于 `OllyDbg` 中的 `F8`）

![](img/723a7cf26a64f5328cf1bf9cdecb0828.jpg) `F8` or `F11`：单步步入。功能同单步步过(`F10`)类似，区别是遇到 `CALL` 等子程序时会进入其中，进入后首先会停留在子程序的第一条指令上。（相当于 `OllyDbg` 中的 `F7`）

![](img/d6aa2e0f78b2fa9cc1b30a04503a0f5b.jpg) `F7`：运行到选定位置。作用就是直接运行到光标所在位置处暂停。（相当于 `OllyDbg` 中的 `F4`）

![](img/d6110efe0293e994b10af2330259fb1a.jpg) `Shift+F11`：运行到跳出函数。

![](img/e78aa8eccdb157bafea212893378d353.jpg) `F5`：运行。按下这个键如果没有设置相应断点的话，被调试的程序将直接开始运行。（相当于 `OllyDbg` 中的 `F9`）

`WinDbg` 没有类似 `OllyDbg` 的 `【执行到用户代码】` 功能，但可以用 `笨笨雄` 写的插件，又或者可以多次按 `Shift+F11` 直到跳到用户代码。

注意，如果反汇编窗口中的某行显示蓝色，并不一定该行就是当前执行到的地址。在 `WinDbg` 的反汇编窗口，每次显示的代码行数都是固定的，可以通过窗口右上角的 `previous` 和 `next` 按钮或着鼠标滑轮查看前后的代码，而每次显示在中间的必定会显示蓝色。要回到当前执行到的指令，只需点击菜单的 `Edit > Go to Current Instruction` 即可。

## 七、总结

经过 `nightxie` 大牛的提点，知道了 `bu` 下的断点可以保存到 `WorkSpace` 里。谢谢提点![](img/022c916b9660e467a8663bd57a4ae6d1.jpg "狂笑")

> 引用: `Breakpoints that you set by using bu are saved in WinDbg workspaces, but breakpoints that you set by using bp are not saved in workspaces.`

这篇文章很多部分是我自己对 `WinDbg` 的理解，有些说法如果不对的话麻烦提点一下，小弟不胜感激。![](img/c72cb8eba397ac67dda6e1ac58a9b627.jpg "微笑")

下面是一些我学习时的资料，在这里也整理一下

貌似是《软件调试》中的章节

[WinDBG 用法详解.rar](http://bbs.pediy.com/upload/bbs/94457/WinDbg.rar)

来自 `http://bbs.dbgtech.net/` 的资料

`WinDbg` 入门系列.rar
