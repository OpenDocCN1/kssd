
# IDA + GDBServer 实现 iPhone 程序远程调试

> 原文：[`www.pediy.com/kssd/pediy12/138472.html`](https://www.pediy.com/kssd/pediy12/138472.html)

IDA + GDBServer 实现 iPhone 程序远程调试  
By：obaby

在早期的 IDA 中包含了一个 `iphoneserver` 的程序，这个程序就是配合 IDA 实现远程调试的。但是在最新版的 IDA 中这个东西已经不复存在了，因而下载的破解版的 IDA 中没有那个文件并不是被删除掉了，而是本来就没有，*^_^*。所以一直以来调试 iPhone 上的二进制程序只能悲催的使用 `ssh+gdb` 进行调试，虽然调试器的功能还算可以，但是每次调试都需要设置显示，只能使用命令进行控制，因而用起来还是不是十分爽。

其实网上关于 IDA 实现 ios 设备远程调试的文章从网上也是可以找到的，但是说的都不是十分具体。本文主要是介绍下 IDA 实现远程 iPhone 程序调试的方法，当然这样调试还存在一些问题，如果大家有什么好的解决方案还请不吝赐教。

需要注意的是要想调试 ios 设备上的程序并不是简单的吧 `gdbserver` 拷贝到 ios 设备上行就可以了，此时如果使用 `gdbserver` 启动进程将得到类似如图 1 所示的提示信息：

![](img/7cffef01a0a0ca33136e350450467ec2.jpg)

图 1

提示文件没有找到，但是如果设备曾经链接过 `xcode` 进行过程序调试则不会出现下面的情况。这是因为 `xcode` 在进行程序调试的时候会将所需要的文件拷贝到 `Develop` 目录下，这样的话不论 `gdbserver` 在哪个目录下程序都是可以正常执行的。同样在上面的命令中看到的 `debugserver` 是苹果自己编译的一个 `gdbserver` 调试服务器，而实际使用则基本是没有任何的差异和影响的。

首先来配置下设备上的调试环境，从 `xcode` 的 `/Developer/Platforms/iPhoneOS.platform/DeviceSupport/3.2` 目录下找到 `DeveloperDiskImage.dmg` 文件，而 `3.2` 则是对应的 ios 设备上的系统版本，在 `DeviceSupport` 目录下通常会包含如如 2 所示的版本列表：

![](img/6392f02e270720fc5c5b07a46c8f8204.jpg)

图 02

在实际的调试过程中只需要选择相应的版本就可以了。在每个目录下都存在一个 `DeveloperDiskImage.dmg` 文件，用 `ultraiso` 打开这个文件可以看到如图 3 的文件目录列表：

![](img/494afe2174c509c64af97939782b75c2.jpg)

图 3

在调试的过程中需要将整个 `dmg` 文件下的所有目录和文件拷贝到设备上的 `/Developer` 目录下，如图 4 所示。这一步可以使用 ios 的文件管理工具，或者使用 `winscp` 上传文件，不建议使用后者，太卡了～，如果使用过 `xcode` 开发调试过程序那么这个目录下的所有东西应该都是存在的，可以无需手工复制。

![](img/0cacf9a956a74d6f2b9b67bdc70ebff3.jpg)

图 4

另外还需要在设备上安装 `ssh` 服务端，这个直接从 `cydia` 源中安装即可。最后就是程序的调试了。

在这里的调试可以使用预先分析文件和直接 `attach` 的方式进行调试。直接 `attach` 的方式比较简单一些，使用 `ssh` 客户端连接到设备（需要无线网络支持，如果木有，那我也办法哦～），在 `win` 下可以使用 `putty`。

登录后默认会在 `root` 目录下，如图 5 所示。

![](img/2808c73d68644394da1dcfd69d74e874.jpg)

切换到 `/Develop/usr/bin/` 目录下，给 `debugserver` 增加执行权限，如图 6 所示。

![](img/f388699e66644809bc88869f26cd9b2d.jpg)

图 6

现在就可以执行一个程序了，以 `AppleAssistantHD` 为例，执行 `./debugserver port:2008 /var/mobile/Applications/ABDEE3BA-02BE-4251-A57B-ECC363813133/AppleAssistantHD.app/AppleAssistantHD`，此时 `debugserver` 会执行目标程序并且同时打开一个端口监听远程连接，如图 7 所示

![](img/f63fb6ba5d834de5704a59adef40c190.jpg)

图 7

现在就可以运行 `ida` 链接远程调试器了。执行 `ida` 直接点 `go` 进入到程序界面即可，如图 8 所示。

![](img/a83eae97808356b5b1c033d234b7c194.jpg)

图 8

进入之后点击菜单的 `Debugger->Attach->Remote GdbDebuger`，如图 9 所示

![](img/c2a5fb5df70beabbd5bae32aa33a1e60.jpg)

执行之后将会打开如图 10 所示的调试器附加窗口。

![](img/2cf5e30f9b86498233e482d26153b091.jpg)

图 10

在 `Hostname` 中输入设备的 `ip`，`port` 中输入 `debugserver` 的监听端口。点击 `Debug options` 打开调试选项，如图 11 所示。

![](img/687dfac27c6eb3b6e2ef95fc93ffd1a8.jpg)

图 11

执行 `Edit exceptions` 打开异常处理选项窗口，如图 12 所示。

![](img/4602db82405d98dc0aaa7d1ccc6b18e0.jpg)

图 12

编辑 `11` 号异常，去掉 `stop program` 的勾，如图 13，否则在调试的过程中会非常的痛苦。

![](img/579bbc4d79cd9434d533d9e7b0274606.jpg)

图 13

关闭所有的设置窗口，现在就可以进行附加了，点击 `ok` 之后将会弹出如图 14 所示的进程列表。

![](img/128fbd8eac40465132241b98d4f07244.jpg)

图 14

选择进程 `id` 为 `0` 的进程进行附加，如果成功将会弹出如图 15 所示的窗口。

![](img/8f0d2eb0fb6c07dc25417c89520bbcfa.jpg)

图 15

现在就可以进行调试了，如图 16 所示。

![](img/dff5299952baebf26cb5ec27a2be85b9.jpg)

图 16

上面的是直接 `attach` 的方法，这种方法能看到的只有内存数据，所以在实际的调试中并不是最好的方法。另外一种方式是直接用 `ida` 载入文件之后采用链接远程调试器的方式。首先将调试器设置为 `Remote gdb debuger` 然后编辑 `ida` 的 `Debug application setup:gdb` 设置，如图 17 所示：

![](img/1308279b30554bd2fa1d72ee09989bf7.jpg)

图 17

在 `application` 和 `input file` 中输入设备上的文件的绝对路径，其余的设置与 `attach` 的方式是完全一致的。设置完成之后就可以启动调试器了，此时会提示已经有远程进程在被调试，是否附加到，如图 18.

![](img/28b1895cedc2bc9d344a35ac6b9a5ec1.jpg)

图 18

点击 `yes` 之后就可以进行调试了，如果附加成功将会弹出提示信息。这样调试的时候代码看起来还是比较清晰的，如图 19

![](img/2a4e8b842b751ed2d50a8731a4e6838a.jpg)

图 19

虽然现在可以调试了，但是还有几个问题。也是我现在没有处理掉的：

1. 在静态分析的时候设置的断点会变为无效，如图 20 所示；
![](img/e78ab11b90ad39681227589a62db0aaf.jpg)
图 20
2. 一旦将进程挂起之后重启启动进程已经卡死了，没有任何的响应。

由于上面的两个问题导致现在 `ida` 虽然可以附加到进程或者启动独立的进程进行调试，但是实际的效果并不理想。明显是一锤子买卖，一旦暂停就完全死翘翘了，这也就是在文章开始处提到的那个问题。所以对于上面的两个问题哪位大大有好的解决办法还望不吝赐教。![](img/d23858dd221cce728a3a61b46badb044.jpg "恐怖!")![](img/d23858dd221cce728a3a61b46badb044.jpg "恐怖!")![](img/d23858dd221cce728a3a61b46badb044.jpg "恐怖!")![](img/d23858dd221cce728a3a61b46badb044.jpg "恐怖!")

Pdf 版本下载：`IDA +DbgServer.pdf`
