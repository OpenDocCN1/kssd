
# 驱动开发和调试的环境设置

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-627.htm`](https://www.pediy.com/kssd/pediy09/pediy09-627.htm)

> 【文章标题】: 驱动开发和调试的环境设置

> 【文章作者】: `qqeleven`

> 【使用工具】: `VC2003`,`VMware`,`Visual SoftICE`

> 【操作平台】: `xp sp2`

> 【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

--------------------------------------------------------------------------------

> 【详细过程】

介绍一下如何用 `VC2003` 开发驱动，并且介绍在虚拟机 `VMware` 中用 `Visual SoftICE` 调试驱动。

先安装好 `VC2003`，再安装 `Visual Assist X`（`VC` 的一个插件，高亮、查错、、、）。`VC` 安装好后就要安装驱动开发包 `DDK2003` 了,建议完全安装。之后就安装 `DriverStudio`，我用的是 `3.2`，如图：

![](img/d2cec5859e6bb97e5eaf1fd3f77f10d6.jpg)

打开 `VC2003`，工具－>DriverStudio－>DDK Build Settings…，设置编译环境：

![](img/b94f05ffe4cd04699d9786a4afa715e2.jpg)

然后设置 `Visual Assist X`：

![](img/f7ae1b86a5a735d6067a661b30e41c31.jpg)

开发环境设置完成了。现在编译一个驱动，拿 `DDK` 目录下的一个有现成代码的例子 `ioctl` 讲解。用 `DriverStudio` 提供的一个工具 `DDK sources to vcproj Converter` 建立工程文件。打开 `ioctl` 目录下的 `dirs` 文件（其实是打开 `sources` 文件），Convert-> Convert DDK dirs/sources…，按下 `Convert` 生成工程文件。用 `VC2003` 打开 `ioctl.sln`，现在你可以修改或者编译驱动了，因为这个是现成的例子不用修改，那就直接编译吧。用 `Win32 Checked` 方式编译，编译工程：工具－>DriverStudio－>Build Solution with DDK Build.exe。如果有错误产生可以参考 `log` 文件来修复错误，编译成功后会产生 `ioctlapp.exe` 和 `sioctl.sys` 文件。下面建立调试环境来调试刚生成的驱动文件。

先安装好 `VMware`，再在虚拟机中安装好 `xp sp2`，然后在 `xp` 中安装好 `DriverStudio` 的 `Visual SoftICE`（Target）。之后设置 `Visual SoftICE`，`Startup` 中把 `Visual SoftICE` 设置成 `Boot`，`Settings` 中选中所有，`Connections` 中选用 `COM1` 连接，`Keep Alive` 设定为 `6000`，完成。关闭虚拟机电源设置 `VMware`，为虚拟机添加一个 `COM` 口，如图：

![](img/ba25403f7d27fb3806ad612e351d3638.jpg)

设置完成，下面开始调试。

启动虚拟机把 `ioctlapp.exe` 和 `sioctl.sys` 两个文件复制到虚拟机中，主机打开 `Visual SoftICE`，Debug->Connect->Named Pipe…，填入 `com_1`，连接到虚拟机。Debug->Symbol Settings->Symbol Paths…，设置好 `MS` 的符号下载路径和本地路径，下载调试符号，如图：

![](img/21da80557e5dc7eeeec7641600152791.jpg)

一定要下载系统的调试符号文件，这一步不能省。最后，还要把编译驱动的符号路径 `ioctl\sys\objchk_wxp_x86\i386` 和驱动的原代码路径 `ioctl\sys` 分别加入到 `User Symbol Search Paths` 和 `Source Search Paths` 中去。然后，在命令行 `SI>`中下断点 `bpload sioctl.sys`，当系统加载驱动 `sioctl.sys` 时断下。回到虚拟机中双击 `ioctlapp.exe`，断下，如图：

![](img/4b7d0dc339c2f0f14a6b32eed680cb1e.jpg)

继续，点击标签 `sioctl.c` 在 `DriverEntry` 例程开始处下断点，取消刚才那个断点，`Go`，如图：

![](img/3bc4c0b3d778fb9e1885f1cdc44b10b9.jpg)

可以进行源码调试了，方法和 `WinDbg` 差不多，在这里不再说明了，祝大家好运！

--------------------------------------------------------------------------------

> 【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

把文章放到我的 Blog 上去了：[`blog.csdn.net/qqeleven/`](http://blog.csdn.net/qqeleven/)

`4dd466d0e1e0cc123afbe504da2462ca`
