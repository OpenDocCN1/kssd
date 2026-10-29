
进入`/path/to/xxx`目录，找到`xxx.json`。

运行起来就跟系统没有 Xposed 一样；完美契合某些金融/银行类 APP。

太极·Magisk 针对 APP 的模块，`无需重启即刻生效`。

太极·Magisk `更不易被检测`。太极的弱侵入特性不再修改 运行时，也不在全局环境中留下任何踪影；因此只要想要做到，可以轻松逃过各种代码类型的检测。

### 2. 太极 Magisk 模式 (Magisk 模块)

来自作者的一些话：

太极·Magisk 版针对会刷机的用户，如果你不喜欢折腾，请不要刷机，直接使用普通版即可；你依然可以使用常用的 Xposed 模块。并且，太极·Magisk 版和普通版安装包完全一致，后续统一维护。

`我本人针对 9.0 系统做了较多的测试`，8.0 的系统仅做过简单测试。8.0 以下的系统压根没有测试；因此，如无必要，8.0 以下系统的用户无需尝试。9.0 以下系统的用户，如果能刷 Xposed，也不建议使用。

太极的免 Root 模式虽然无需解锁 Bootloader，使用起来异常方便，但是有一些`缺点`，`比如无法修改系统、必须卸载原有 APP`。因此我开发了一个 magisk 模块，这个模块可以通过 Magisk 赋予太极更强大的功能；从而突破免 Root 模式的种种限制。

当你刷入太极提供的 Magisk 模块之后，太极 APP 会自动从 免 Root 模式切换到 Magisk 模式：太极 APP + 太极 Magisk 模块 = 太极·Magisk。当 太极的 magisk 模块被禁用或者卸载之后，太极会自动回到 免 Root 模式运行。

如果你需要使用 Magisk 模式，请仔细阅读 太极·阳 使用说明。

太极·虚拟框架

一句话简介：免解锁,免Root,就能使用Xposed框架

一个可以免 Root 运行的类 Xposed 框架

资料

官网：

<mark class="encrypted">fc6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2@1j5h3W2U0K9r3W2Q4x3X3c8S2M7s2m8Q4x3X3g2U0L8$3#2Q4x3V1j5`.</mark>

<mark class="encrypted">a8bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1j5h3W2U0K9r3W2Q4x3X3g2U0L8$3!0D9i4K6u0r3P5X3S2Q4x3V1j5`.</mark>

文档

<mark class="encrypted">a6dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1j5h3W2U0K9r3W2Q4x3X3g2U0L8$3!0D9i4K6u0r3P5X3S2Q4x3V1k6V1L8$3y4Q4x3V1j5`.</mark>

下载

<mark class="encrypted">170K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1j5h3W2U0K9r3W2Q4x3X3g2U0L8$3!0D9i4K6u0r3P5X3S2Q4x3V1k6V1L8%4N6F1L8r3!0S2k6q4)9J5k6h3S2@1L8h3H3`.</mark>

模块下载

太极官网-太极app，开启全新的XPOSED模块使用体验

<mark class="encrypted">6cfK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2@1j5h3W2U0K9r3W2Q4x3X3c8S2M7s2m8Q4x3X3g2U0L8$3#2Q4x3V1k6Q4x3U0y4Q4x3V1k6V1L8%4N6F1L8r3!0S2k6q4)9J5c8X3q4D9K9i4m8S2P5b7`.`.</mark>

### 3. 三种 magisk 安装方法：

安装 magisk 前要安装 magisk manager？（<mark class="encrypted">89eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0j5h3N6A6M7$3E0E0j5h3&6S2k6$3g2J5i4K6u0W2j5$3!0E0i4K6u0r3i4@1g2r3i4@1u0o6i4K6R3&6</mark>

按顺序所需安装内容：

`MagiskManager-v7.1.1.apk`

`Magisk-v19.0.zip`

`Magisk-uninstaller-20190328.zip` - 这个是备用的卸载文件，如果安装失败无法开机，需要刷入这个

三种安装方法

`直接安装`

由于 MIUI 开发版自带 Root 权限，只要在酷安/Github 下载 Magisk Manager，赋予其 Root 权限，打开 Magisk Manager 首页-》安装-》直接安装即可。说明：这项只会出现在拥有 Root 权限的手机，但是即使一开始就有 Root 权限，也不推荐这种安装方法，因为成功安装后并不能隐藏 Magisk 内置的 Root 权限，会造成 basicIntegrity 检测失败。

`下载安装包（推荐方法）`

打开 Magisk Manager 在首页-》安装-》仅下载安装包/或在 Github 下载 Magisk 安装包并传入手机目录，命令行执行 `adb reboot bootloader` 进入手机的 Fastboot/Bootloader 模式，继续执行 `fastboot boot TWRP.img`，刷入临时 TWRP（第三方 rec），手机会自动重启到 TWRP 中，首页-》安装-》找到 Magisk 安装包点击-》直接右滑底部的滑动条-》重启，打开手机里 Magisk Manager 即可看到安装成功啦。

`修补镜像文件`

在不具备解锁的手机上安装，需要提取手机系统安装包中的 `boot.img` 固件，并通过 Magisk Manager 中，首页-》安装-》修补镜像文件，选择这个 `boot.img`，生成新的镜像后发给电脑，Windows 命令行执行 `adb reboot bootloader` 进入手机的 Fastboot/Bootloader 模式，执行 `fastboot boot Patchedboot.img`，来加载生成后的 boot 分区文件获取临时的 Root 权限，此时打开 Magisk Manager 会提示已经安装，此时别高兴得太早，还得在首页-》安装-》直接安装，才能将临时 Root 转换成永久 Root。

### 3. 刷 SuperSU、magisk 需要的步骤：

1. 解锁 oem（刷第三方 recovery 必须解锁）
2. 刷第三方 recovery，比如 TWRP.

1. `adb reboot bootloader` 进入 bootloader
2. `fastboot devices` - 查看设备是否连接 ok
3. `fastboot flash recovery twrp.img` - 刷入第三方 recovery
3. 刷入 SuperSU 或者 magisk 模块的 `.zip` 文件
1. 拷贝 `.zip` 文件到 twrp 能访问的目录，比如 `sdcard/downloads`
2. `adb reboot bootloader` - 进入 bootloader
3. 选择 recovery mode 进入 twrp 界面
4. 点击 install，选择 `.zip` 文件，不要勾选滑动滑动框进行安装。
4. 刷 xposed
把下载的文件复制进手机去：

`xposed-v90-sdk26-arm64-beta3.zip`

`XposedInstaller_3.1.5.apk`

手机关机，按住【开机键】+【音量减】，按音量键进入 Recovery mode，电源键确认（或命令行 `adb reboot bootloader` 然后选择 recovery mode）。

进入 twrp recovery -> 选择 Install，选中刚刚复制进去的 `xposed-v90-sdk26-arm64-beta3.zip` 文件，拖动三个箭头的图标

完成后选择 reboot system，选择 do not install 重启系统

重启会停在开机动画 15 分钟左右就会进入系统了。

进入系统在安装 `XposedInstaller_3.1.5.apk`，此时 xposed 就安装完成了。

### 4. 附：

刷入 Xposed Framework 然后进入系统在 Xposed Installer 里面发现报错

`Xposed framework version xx is installed, but not active. Please check logs for details.`

Xposed Installer 的 log 里面提示的是关于 `/data/data/de.robv.android.xposed.installer/conf` 的权限 error……

解决办法：使用 RE 管理器进入 `/data/data/de.robv.android.xposed.installer/conf` 将里面的 `modules.list`,我进去后发现没整个文件，就把 cache 目录删除了 删除，重启手机发现 ok 了。（思路来自 <mark class="encrypted">1b4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8W2)9J5k6h3#2A6N6h3W2Q4x3X3g2U0L8$3#2Q4x3V1k6Q4c8e0u0Q4z5o6m8Q4b7e0j5I4i4K6u0W2K9s2c8E0L8q4!0q4c8W2!0n7b7#2)9^5z5b7`.`.</mark>

解决了上面的问题后，打开手机上的 xposed installer，发现现实已经 active，但是黄色字体提示：Could not load available ZIP files.Pull down to try again.

解决办法：联网，下拉刷新一下就 ok。

> Xposed 从 Android 4.0 就开始出现，而现在 Android Q 的版本已经来到了 10.0/11，且 Xposed 框架又没有像 Android 系统版本一样能够快速的更新，从而导致很多 Xposed 用户对于自己究竟该使用哪个版本或者说怎么使用 Xposed 框架感到很迷茫。所以在自己的设备上，特别是 7.0 之上的设备上想要使用 xposed 框架进行 hook 之前，需要对整个 xposed 安装环境有全面的了解。

### 0. 开胃菜

不管是刷机还是刷各种框架都需要对 recovery，系统分区，root 权限这些基础知识有所了解。如果还不了解，可以先看下[这篇文章](https://bbs.kanxue.com/elink@f7bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6*7K9s2g2S2L8X3I4S2L8W2)9J5k6i4A6Z5K9h3S2#2i4K6u0W2j5$3!0E0i4K6u0r3M7q4)9J5c8U0x3I4z5o6R3J5y4o6l9^5)，否则直接跳过就好.

### 1. 适用于 Android 4.0 ~ Android 7.1 的 Xposed 框架

作者 rovo89 的 Xposed 的[官方版本](https://bbs.kanxue.com/elink@435K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6J5L8%4k6G2z5o6W2Q4x3V1k6j5M7r3!0K6k6h3c8Q4x3V1k6J5k6h3I4W2j5i4y4W2M7H3`.`.) 最新的版本停留在 Android [8.0/8.1 beta3 版本](https://bbs.kanxue.com/elink@ae4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2L8X3c8J5L8$3W2V1M7$3q4Y4k6g2)9J5k6h3y4G2L8g2)9J5c8U0t1H3x3e0S2Q4x3V1j5H3x3g2)9J5c8U0l9&6i4K6u0r3k6r3!0%4L8X3I4G2j5h3c8Q4x3X3c8A6L8Y4y4@1j5h3I4D9i4K6u0V1L8r3q4@1k6i4y4@1i4K6u0V1P5s2m8G2M7$3g2V1i4K6u0V1N6U0V1H3i4K6u0V1j5h3&6V1M7X3!0A6k6q4)9J5k6o6S2Q4x3X3b7H3i4K6u0V1z5q4)9J5k6o6q4Q4x3X3c8G2M7X3g2G2i4K6u0r3)。

如果你的系统在 4.0 ~ 7.1（**8.0，和 8.1 也可以，但是不建议**）之间，那么你直接使用官方的框架即可，但由于在 Android 5.0 之后 Xposed 框架的安装方式变得更加简单，因此具体的安装步骤如下：

获取 Root 权限，这是前提条件，通常是使用 SuperSu（7.0 及以下低版本）和 Magmisk（8.0 以上高版本）的方式，当然也可以直接刷编译 root rom。

**卡刷 Xposed.zip 安装**：

如果 Android 系统版本在 Android 5.0（不包括）以下：

你需要通过 recovery 刷入 `Xposed.zip` 文件实现 recovery 卡刷安装 xposed 框架

然后手动安装 `xposedinstaller.apk` 管理程序。

**Xposedinstaller 中安装**：

如果 Android 系统版本在 Android 5.0（包括）以上

首先安装 Xposed installer

然后打开，给予权限，根据自动匹配的文件自动开始安装

当然你仍然可以通过 recovery 来刷入相关文件实现上面的目的，区别是一个自动一个手动

![image](https://bbs.kanxue.com/upload/attach/202012/701425_REQNMEA4BVCVD7Q.jpg)

Download and Install Xposed v90 Beta 3 for Android Oreo 8.0/8.1：aedK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2L8X3c8J5L8$3W2V1M7$3q4Y4k6g2)9J5k6h3y4G2L8g2)9J5c8U0t1H3x3e0S2Q4x3V1j5H3x3g2)9J5c8U0l9&6i4K6u0r3k6r3!0%4L8X3I4G2j5h3c8Q4x3X3c8A6L8Y4y4@1j5h3I4D9i4K6u0V1L8r3q4@1k6i4y4@1i4K6u0V1P5s2m8G2M7$3g2V1i4K6u0V1N6U0V1H3i4K6u0V1j5h3&6V1M7X3!0A6k6q4)9J5k6o6S2Q4x3X3b7H3i4K6u0V1z5q4)9J5k6o6q4Q4x3X3c8G2M7X3g2G2i4K6u0r3i4K6t1K6d9r3!0%4i4K6g2X3N6r3!0Q4y4h3k6u0L8Y4y4@1j5h3I4D9i4K6g2X3h3s2m8G2M7$3g2V1i4K6g2X3c8Y4u0S2L8h3g2%4L8%4u0C8i4K6g2X3L8$3&6@1L8#2)9#2k6V1q4F1k6s2u0G2K9h3c8Q4y4h3j5^5x3o6R3I4i4K6g2X3e0%4u0W2L8H3`.

---

### 2. 适用于 Android 8.0-10.0 的 Xposed 框架
