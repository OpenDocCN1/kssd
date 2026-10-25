
# [原创]X86内核笔记_0_配置双机调试环境

# 1.为什么要使用双机调试

当我们调试用户层程序时，调试器会将线程挂起，此时目标程序处于一种“卡死”的状态。

若目标是操作系统，那么被挂起的将是整个操作系统。Windows为了可以与调试器交互，设计了一个`调试子系统`。在调试时，除了调试子系统以外的所有线程全部被挂起，仅剩调试子系统仍可运行。用来与调试器进行信息交互。

信息交互需要通道，操作系统被挂起，声卡网卡等设备无法使用。但可以通过`串口`来向外发送数据。

调试器通过串口与调试子系统建立连接来进行信息交互。

![image-20210330131637315](upload/attach/202106/867232_5G3D4VHVGG87R4K.jpg)

# 2.XP配置双机调试

## 2.0 必备文件

`WDK_7600.iso`：<mark class="encrypted">1dfK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0L8r3!0#2k6q4)9J5k6e0p5^5z5g2)9J5k6h3y4F1i4K6u0r3N6q4)9J5c8X3W2q4y4@1k6K9k6V1W2n7b7U0y4y4K9R3`.`.</mark> (访问码:y20m)

`VirtualKD-Redux`：<mark class="encrypted">902K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0L8r3!0#2k6q4)9J5k6e0p5^5z5g2)9J5k6h3y4F1i4K6u0r3N6q4)9J5c8V1&6d9d9Y4k6y4L8Y4q4&6h3g2j5%4N6R3`.`.</mark> (访问码:1thh)

XP符号包：<mark class="encrypted">c70K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0L8r3!0#2k6q4)9J5k6e0p5^5z5g2)9J5k6h3y4F1i4K6u0r3N6q4)9J5c8U0y4k6f1U0y4g2x3#2q4v1k6V1W2T1P5b7`.`.</mark> (访问码:yz0m)

XP纯净镜像：<mark class="encrypted">301K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0L8r3!0#2k6q4)9J5k6e0p5^5z5g2)9J5k6h3y4F1i4K6u0r3N6q4)9J5c8X3k6E0x3Y4W2E0L8e0u0W2b7f1&6F1j5b7`.`.</mark> (访问码:hgr8)

XP系统激活码：HTXH6-2JJC4-CDB6C-X38B4-C3GF3

## 2.1 虚拟机镜像

这里使用XP作为目标系统来学习内核。因此我安装了一个XP的镜像。推荐安装itellyou上的XP镜像

```
Windows XP Professional with Service Pack 3 (x86) - CD (Chinese-Simplified)

msdn.itellyou.cn

没有下载器的话可以使用 闪电下载 ，支持磁力
```

之前随便找了一个XP镜像，结果符号包解析不了，又重新安装了一个itellyou的镜像。

开机后进入C盘根目录，有一个`boot.ini`文件，这是个隐藏文件，记得去控制面板里设置显示隐藏

![image-20210402150203880](upload/attach/202106/867232_7BUNMRND32YVMXP.jpg)

打开ini文件，加入下面这一行：

```
multi(0)disk(0)rdisk(0)partition(1)\WINDOWS="debug" /noexecute=optin /fastdetect /debug /debugport=com2
```

![image-20210402150258673](upload/attach/202106/867232_PRMRMC2YTHHCJSA.jpg)

然后就可以关机了，镜像就配置完了

## 2.2 虚拟机串口

为镜像添加一个串口

![image-20210402150620232](upload/attach/202106/867232_X2A6SVSXXTERT9V.jpg)

设置为`使用命名的管道`、`该端是服务器`、`另一端是应用程序`，管道命名填写：

```
\\.\pipe\com_2
```

![image-20210402150750148](upload/attach/202106/867232_2AJ7WX3VSGG7KT2.jpg)

然后点击确定。

## 2.3 Windbg安装

`windbg`的安装使用`WDK_7600.ISO`来安装WDK，这样就会附带`windbg`。不要自己装`windbg`，否则可能出现一些错误。

双击ISO文件，等待DVD驱动器挂载成功后，管理员运行`KitSetup.exe`，然后一直下一步。

![image-20210402151847737](upload/attach/202106/867232_833G6U3EZ3PFQKS.jpg)

安装完成后，找到WDK的安装目录（安装的时候会让你选），`Debugger`目录下就是`windbg`的组件，将`windbg.exe`发送到桌面快捷方式。

![image-20210402152127812](upload/attach/202106/867232_UTPZAM85AVDR76K.jpg)

## 2.4 XP符号包安装

双击运行`windowsxp-kb936929-sp3-x86-symbols-full-enu.exe`，什么都不要改，直接下一步等待安装成功。然后你的`C:\Windows`会多出一个`Symbols`文件夹，结构如下图，如果不对那就重新安装。这步绝对不能出错。

![image-20210402152343078](upload/attach/202106/867232_G8A3VFGAW8HYPAS.jpg)

## 2.5 Windbg参数配置

右键`windbg.exe` - 快捷方式，点击属性，在目标那里后面加一段话（记得-b前面有个空格别忘记加）：

```
-b -k com:port=\\.\pipe\com_2,pipe
```

![image-20210402152532054](upload/attach/202106/867232_49TF94FG945BUV7.jpg)

## 2.6 环境变量配置

### 2.6.1 Windbg环境变量配置

右键此电脑->属性->高级系统设置->环境变量，在`Path`里新建一条记录，内容为WDK安装路径下的`debugger`文件夹

![image-20210402152859496](upload/attach/202106/867232_W937KABPAZHSYDX.jpg)

### 2.6.2 符号包环境变量配置

在上面的用户变量处新建一个记录，变量名和变量值直接复制下面的，一定不能错，不然后面`windbg`命令用不了。

```
变量名：
_NT_SYMBOL_PATH

变量值：
C:/WINDOWS/Symbols/acm;C:/WINDOWS/Symbols/cnv;C:/WINDOWS/Symbols/com;C:/WINDOWS/Symbols/cpl;C:/WINDOWS/Symbols/dic;C:/WINDOWS/Symbols/dll;C:/WINDOWS/Symbols/drv;C:/WINDOWS/Symbols/exe;C:/WINDOWS/Symbols/iec;C:/WINDOWS/Symbols/ime;C:/WINDOWS/Symbols/ocx;C:/WINDOWS/Symbols/scr;C:/WINDOWS/Symbols/sys;C:/WINDOWS/Symbols/tpl;C:/WINDOWS/Symbols/tsp;C:/WINDOWS/Symbols/wpc;symsrv*symsrv.dll*C:/mysymbol*<mark class="encrypted">e0dK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3#2K6k6r3I4Q4x3X3g2E0K9h3y4J5L8%4y4G2k6Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6V1L8%4N6F1L8r3!0S2k6q4)9J5c8Y4y4&6L8h3u0G2L8s2x3`.</mark>
```

![image-20210402153056391](upload/attach/202106/867232_XXM3J58RYSYR5C3.jpg)

都弄完后点击确定，保存退出。

## 2.7. 进入debug模式XP

XP开机后选择DEBUG系统，按回车进入。

![image-20210402153227748](upload/attach/202106/867232_CCDSW2MW9D35M8A.jpg)

然后双击`windbg` - 快捷方式，等待`windbg`连接串口。

如果出现`nt!RtlpBreakWithStatusInstruction:`那就是配置没问题了。如果出现的是`BreakWithPoint`那就说明你的符号包配置的有问题。

![image-20210402153334506](upload/attach/202106/867232_4VPDHT5YA8X5MTV.jpg)

输入`!process 0 0`查看进程，如果可以成功查看，那么双机调试就已经完美配置成功了。如果不可以，那就还是符号包配置有问题。

![image-20210402153645842](upload/attach/202106/867232_248FYNCW96NJ5G5.jpg)

# 3.win7~10配置双机调试

> 此处以win7x86演示

## 3.0 必备文件

WIN7-X86-SP1镜像：<mark class="encrypted">097K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0L8r3!0#2k6q4)9J5k6e0p5^5z5g2)9J5k6h3y4F1i4K6u0r3N6q4)9J5c8X3V1$3j5X3g2k6k6Y4u0u0d9X3&6S2P5b7`.`.</mark> (访问码:0xyt)

`VisualStudio2008`（镜像内安装）：<mark class="encrypted">86bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0L8r3!0#2k6q4)9J5k6e0p5^5z5g2)9J5k6h3y4F1i4K6u0r3N6q4)9J5c8U0u0S2k6i4W2#2P5f1g2K9j5Y4g2q4K9R3`.`.</mark> (访问码:e34z)

`WDK_1803`：<mark class="encrypted">2f3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0L8r3!0#2k6q4)9J5k6e0p5^5z5g2)9J5k6h3y4F1i4K6u0r3N6q4)9J5c8V1#2n7N6X3#2#2j5h3k6k6e0V1&6\*7k6b7`.`.</mark> (访问码:kmd0)

`VisualStudio2017_1803`（实体机安装）：<mark class="encrypted">4b4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0L8r3!0#2k6q4)9J5k6e0p5^5z5g2)9J5k6h3y4F1i4K6u0r3N6q4)9J5c8V1q4B7y4X3&6A6k6g2q4T1j5g2g2J5N6b7`.`.</mark> (访问码:4ocr)

## 3.1 镜像内配置

在XP系统中我们在C盘目录下修改了`boot.ini`文件即可开启调试模式。但在win7及以后的版本中，windows为我们提供了一个可视化工具来配置引导模式。

以管理员身份运行`cmd`，可以搜索`cmd`，然后右键以管理员运行。![image-20210603105700789](upload/attach/202106/867232_E3A3TDTHYUHR88F.jpg)

在`CMD`中输入`bcdedit /copy {current} /d debug`来添加名为debug的引导模式，回车执行后记下ID。我这里的ID是`e7fee4cf-c3b4-11eb-9a3d-82405acb2d2e`，后面的命令要用到这个ID。![image-20210603105850449](upload/attach/202106/867232_V59B6DE9ZR6P4Z6.jpg)

执行`bcdedit /displayorder {e7fee4cf-c3b4-11eb-9a3d-82405acb2d2e} /addlast`让新加的模式显示出来。![image-20210603110029839](upload/attach/202106/867232_PSXW6DNEMA3XXXY.jpg)

执行`bcdedit /dbgsettings SERIAL DEBUGPORT:1 BAUDRATE:115200`设置调试端口及波特率。

![image-20210603140354490](upload/attach/202106/867232_UQBD6JKJ3X39HY7.jpg)

执行`bcdedit /bootdebug {e7fee4cf-c3b4-11eb-9a3d-82405acb2d2e} ON`启用该模式

![image-20210603110254776](upload/attach/202106/867232_F7U4Y2XQ5UEMNPD.jpg)

执行`bcdedit /debug {e7fee4cf-c3b4-11eb-9a3d-82405acb2d2e} ON`启用该模式

![image-20210603110330004](upload/attach/202106/867232_BB2F28HGZHCSXYJ.jpg)

执行`bcdedit /timeout 30`设置选择引导模式时的超时时间。写30就够了。

![image-20210603110410441](upload/attach/202106/867232_NBA5ZWRPZYVFFKN.jpg)

重启虚拟机镜像，一定要点虚拟机内的重启，否则设置不会被更新，

![image-20210603110459596](upload/attach/202106/867232_9QWPC3BVSN2YV9N.jpg)

添加成功后在开机时会看到多出来的debug选项

![image-20210603110547820](upload/attach/202106/867232_5QM8XY7BV9QNM9G.jpg)

## 3.2 VMware设置

如图添加一个串行端口，如果有的话就不用填加了。

![image-20210603144255491](upload/attach/202106/867232_UENYK934DAZ3DMX.jpg)

点击新添加的串行端口，设置一下管道名`//./pipe/com_1`，点击确定保存。

![image-20210603140953324](upload/attach/202106/867232_3EDQQKHJ7SBXCM9.jpg)

如果你的镜像里面有个打印机，记得删除，否则会占用`com1`串口。

![image-20210603141046867](upload/attach/202106/867232_TSP63GQB6DNBB4Q.jpg)

## 3.3 VS+WDK安装

我们这里使用VS2017专业版+WDK1803。

VS需要安装的组件如下图所示，路径按自己喜好即可。：

![image-20210603113126447](upload/attach/202106/867232_XXKZSVJ6Z79EX2Q.jpg)

WDK安装时建议一路默认。

## 3.4 WinDbg配置启动参数

在`WinDbg`快捷方式的目标选项中追加` -y SRV*C:\symbol*<mark class="encrypted">0e7K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3#2K6k6r3I4Q4x3X3g2E0K9h3y4J5L8%4y4G2k6Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6V1L8%4N6F1L8r3!0S2k6q4)9J5c8Y4y4&6L8h3u0G2L8s2x3.</mark> -b -k com:port=//./pipe/com_1,baud=115200,pipe`

## 3.5 系统环境变量配置

增加名为`_NT_SYMBOL_PATH`的变量，值为`SRV*C:\symbol* <mark class="encrypted">e34K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0M7$3c8D9i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6r3!0%4L8X3I4G2j5h3c8Q4x3V1k6K6P5h3#2T1L8$3I4K6i4K6j5H3</mark>`

## 3.6 验证

选择Debug系统，管理员运行`windbg`，如下图断下就说明双机调试已经配置成功了。

![image-20210603141520390](upload/attach/202106/867232_UW2T37M4QVYYBM4.jpg)

接下来`windbg`输入`lm`验证符号配置，如下图则说明符号配置成功

![image-20210603141837381](upload/attach/202106/867232_8NKFM3BQE5QW9PM.jpg)
