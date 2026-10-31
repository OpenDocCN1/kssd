
# [原创]使用 VMware + win10 + VirtualKD + windbg 从零搭建双机内核调试环境

## 前言

当我们没有两台物理机时，又想做双机内核调试怎么办？当然是装虚拟机啦！本文总结了使用VMware15.5 + win10 + virtualkd + windbg搭建双机内核调试环境。

## 安装环境

VMware版本：15.5 pro。可以到VMware 官网下载地址下载最新版本。

我的物理主机系统：win10 1909。其它版本的windows也可以，VMware15.5最低支持win7系统。

虚拟机系统镜像：win10 1809 x64。其它版本的系统也可以。我们可以在msdn i tell you上面找到各个版本的windows官方镜像。除了系统镜像，还包括其它很多工具的官方镜像，比如vs,office等。

![](https://bbs.kanxue.com/upload/attach/202003/873494_GQEH6ANATUPQWG9.jpg)

windbg版本：10.0.18362.1，其它版本的也可以。windbg包含在windows sdk中。可以到微软官网 SDK 下载地址下载。如果只需要windbg，推荐下载.exe形式的安装包。在安装的时候，可以只勾选Debugging Tools for Windows。

![](https://bbs.kanxue.com/upload/attach/202003/873494_Q6V4H4P6A6S852T.jpg)

## 安装VMware& 安装win10

下载VMware到本地后，直接双击即可安装，一路点击下一步即可。需要注意以下两点：

尽量下载最新版本的VMware，早期版本的VMware在win10 1909上可能有兼容性问题。

安装完成后，最好重启系统！

安装好VMware后，就可以安装系统镜像了。通过文件(F) -> 新建虚拟机(N)即可开始新建虚拟机向导。一般我们通过安装程序光盘映像文件(iso)(M)的形式选择下载好的系统镜像安装，然后一路下一步(N)即可。

![](https://bbs.kanxue.com/upload/attach/202003/873494_5BUEW8MHE49TGXJ.jpg)

安装完成后，可以启动了。但是可能遇到一些问题，我做了简单的总结。

## 可能遇到的问题

提示Intel VT-x被禁用。

![](https://bbs.kanxue.com/upload/attach/202003/873494_8BZGRQACWV94NW6.jpg)

根据提示，在BIOS里开启Intel VT-x。

重启进入BIOS，打开Intel Virutal Technology，按F10保存退出。

![](https://bbs.kanxue.com/upload/attach/202003/873494_ZFA2QPA5HQNTZ83.jpg)

提示VMware Workstation与Device/Credential Guard不兼容。

![](https://bbs.kanxue.com/upload/attach/202003/873494_M37ARBSH6KRFZFM.jpg)

强烈建议先检查下Hyper-V是否开启，如果开启了，请先关闭Hyper-V。

![](https://bbs.kanxue.com/upload/attach/202003/873494_YHB82XEXF3H8ZYP.jpg)

如果关闭Hyper-V后还是出现上图的提示。请参考禁用 Device/Credential Guard 的微软官方文档。

点击工具下载链接，下载相应的脚本文件到本地。如果我们的系统不是英文系统，按照提示改变脚本里的 `$OSArch = $(gwmi win32_operatingsystem).OSArchitecture` 为 `$OSArch = $((gwmi win32_operatingsystem).OSArchitecture).tolower()`。

然后以管理员权限执行`DG_Readiness_Tool_v3.6.ps1 -Disable -AutoReboot`即可。

-Disable表示关闭。

-AutoReboot表示自动重启系统，根据自己的需要指定。

![](https://bbs.kanxue.com/upload/attach/202003/873494_54B6FABSNBPT8H2.jpg)

重启后，会有提示是否要真的禁用此特性。按Windows键或者F3键禁用，按Esc跳过。

![](https://bbs.kanxue.com/upload/attach/202003/873494_U37E25RZJ7EWSX2.jpg)

顺利安装完系统过后，我们就可以开始进行双机内核调试了。

## 通过虚拟串口方式连接

如果我们希望通过虚拟串口的形式连接，我们在系统设置前，一定要注意：VMware中的打印机会占用默认的串口 1。如果不移除打印机，我们新建的串口会是串口 2（这样，我们在被调试系统中设置的时候，需要设定`debugport`的值为2）。

![](https://bbs.kanxue.com/upload/attach/202003/873494_Q4QF9YW9QH9QVC5.jpg)

当然，我们可以移除打印机，然后新建串口，这样新建的串口就是默认的串口 1了。

![](https://bbs.kanxue.com/upload/attach/202003/873494_M4QV7S4PFFQWYND.jpg)

我以保留打印机，使用COM2为例，录制了一份屏幕录像。这里对整个操作过程做一个简要描述：

新建串口，因为打印机占用了串口 1，所以我们新建的串口是串口 2。

设置`pipe`参数，我设置的是`\\.\pipe\bcn_com`，其他参数默认即可。

启动虚拟机系统，设置调试参数，命令如下：

```
bcdedit /debug on
bcdedit /dbgsettings serial debugport:1 baudrate:115200
```

以管理员权限运行windbg，通过界面连接。

![](https://bbs.kanxue.com/upload/attach/202003/873494_MB66NZEFCKWE822.jpg)

重启虚拟机系统，发现连接不上。因为我们新建的串口是串口 2，而我们在虚拟机系统里设置的是串口 1。

重启虚拟机系统，设置调试参数，命令如下：

```
bcdedit /debug on
bcdedit /dbgsettings serial debugport:2 baudrate:115200
```

重启虚拟机系统，这次成功连接上了。

最后，还演示了windbg通过命令行直接连接。命令行如下：

```
windbg.exe -k com:pipe,port=\\.\pipe\bcn_com,reconnect
```

![](https://bbs.kanxue.com/upload/attach/202003/873494_H7RP7482XX3SF8N.jpg)

## 通过 VirtualKD

经常做双机内核调试的小伙伴儿一定对VirtualKD不陌生。可以大大提高双机内核调试的速度，可谓神器。

几年前，用VirtualKD搭建双机内核调试环境很简单。没想到这次居然各种连接不上。通过google搜索关键字VMware15 VirtualKD，在VirtualKD github issues和看雪论坛『求助问答』的帖子看到已经有小伙伴儿遇到这个问题了。应该是VirtualKD3.0与VMware15.5不兼容。真是个坏消息，不过也有好消息 —— 已经有大佬解决了这个问题，github上对应的仓库是VirtualKD-Redux。

在查找解决方案的过程，发现有人遇到另外一个问题。在虚拟机里执行`bcdedit /debug on`失败，提示值被保护，如下图：

![](https://bbs.kanxue.com/upload/attach/202003/873494_9MRDERZZVJSDD5M.jpg)

可以在虚拟机设置的高级选项里，禁用安全引导。

![](https://bbs.kanxue.com/upload/attach/202003/873494_8YJHGVABQUGMAAA.jpg)

VirutalKD-Redux的使用方法和VirutalKD一样，简要步骤如下：

在调试主机端运行`vmmon64.exe`，点击Debugger Path...选择调试器路径。

把`target64`拷贝到虚拟机中（如果是32位系统，请拷贝`target32`）。

在虚拟机中执行`target64`目录下的`vminstall.exe`安装VirtualKD-Redux。

重启虚拟机系统，选择新建立的Entry，并且按F8选择禁用强制签名。

如果顺利，调试器会自动弹出并连接到被调试系统（因为Start debugger automatically默认是勾选状态）。

如果调试器没自动弹出，或者我们意外的关闭了调试器，我们可以通过点击Run Debugger按钮，手动启动调试器。

整个过程，非常简单。具体过程录屏如下：

![](https://bbs.kanxue.com/upload/attach/202003/873494_VB4GWGKQ2F3YNYJ.jpg)

## windbg 符号路径设置

当我们执行一些命令的时候（比如，`!thread`），需要符号文件的帮助。我们可以通过`_NT_SYMBOL_PATH`环境变量设置符号路径。如果还没有设置，可以在windbg中执行`.symfix path/to/symbol_cache`自动设置微软符号服务器为上游符号服务器。当需要符号时，windbg会先到`path/to/symbol_cache`中查找符号，如果找不到，会继续到微软符号服务器上查找，如果在微软符号服务器上找到了，会缓存到`path/to/symbol_cache`，然后从`path/to/symbol_cache`加载符号。

温馨提示：

`.symfix`命令会删除windbg中现有的符号路径设置。

目前，好像需要科学上网才能成功连接微软符号服务器。

## 总结

- 使用VMware前，最好禁用Hyper-V，禁用Security Boot，开启Intel VT-x。

- 一定要注意，VMware默认带的打印机会占用串口 1，如果不删除打印机就新建串口的话，新建的串口是串口 2。

- VirtualKD3.0还不支持VMware15.5，我们可以使用VirtualKD-Redux。

- 执行`.symfix path/to/symbol_cache`使用微软符号服务器。

## 参考资料

VirtualKD official install tutorial

"VMware Workstation and Device/Credential Guard are not compatible" error in VMware Workstation on Windows 10 host (2146361)

Setting up kernel debugging (VirtualKD)

VMware+Windgb+Win7内核驱动调试

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
