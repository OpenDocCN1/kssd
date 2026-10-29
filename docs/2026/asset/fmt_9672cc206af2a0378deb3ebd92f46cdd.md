
8、使用`macho_edit`插入动态库

9、重新打包ipa

10、Impactor的使用

## 其它

1、如果有苹果开发者账号，可以使用 iOS App Signer 对ipa重签名，然后使用Xcode安装即可。

2、少许的汇编知识有助于使用Hopper Disassembler理解方法的实现逻辑，便于更快地找到破解的入口。

3、避免重复造轮子，做一名调包侠没什么不对。当好奇轮子是怎么造出来的时候，再去深入研究它。

4、基础一定要打好，但不是等基础打好了再去干活，而是边干活边打基础。

5、阅读他人代码的最佳方式就是帮他改Bug。

6、走弯路是必然的，但也是值得的，你接触到了走直路接触不到的东西。

# 前言

我是个新手，接触iOS逆向破解没几个月，网上教程资料看了很多，知识点比较零散，使用的工具大都是针对iOS 10之前的，在iOS 11上基本无法使用，弯路走了不少，经过多次试验，最终摸索出了自己的破解思路和风格。

网上已经有很多基于CydiaSubstrate的越狱开发教程，使用theos、MonkeyDev进行开发非常方便、高效，而本文则是脱离CydiaSubstrate，以非Unity游戏《干掉月亮》(Shoot The Moon)为例，纯手工打造动态库，注入到游戏中实现破解，虽说效率不比使用CydiaSubstrate来的高，但是能学到很多基本的知识，非常适合新手入门。

# 准备工作

## 设备

- 一部越狱的iPhone手机(iOS 11.0~11.1.2)

- 一台Mac机

## macOS上的工具

- Xcode

- Terminal

- Console

- `ssh`

- `scp`

- Homebrew

- `usbmuxd`

- `macho_edit`

- Hopper Disassembler

- Impactor

## iOS上的工具

- `bfinject`

- `applst`

## App

- 《干掉月亮》(Shoot The Moon)

# 下载安装

## Xcode

从App Store下载，或者从苹果开发者页面下载都可以，下载最新版即可。

## Terminal

macOS自带。许多命令需要在终端上执行。

## Console

macOS自带。查看手机日志。

## ssh

macOS自带。通过电脑登录手机。

## scp

macOS自带。在手机和电脑间互传文件。

## Homebrew

从[Homebrew官网（brew.sh）](https://bbs.kanxue.com/elink@e28K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1M7X3g2%4i4K6u0W2M7$3R3`.)下载安装。

用于安装各种命令行工具。

## usbmuxd

打开Terminal，使用Homebrew安装。

```
$ brew install usbmuxd

# 或者

$ brew install libidevicemobile

```

将iPhone上的端口通过usb线映射到电脑上的端口。

## macho_edit

从[cbeK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6f1P5h3W2D9L8#2)9J5c8X3#2S2j5$3S2G2i4K6g2X3k6h3c8A6N6l9`.`.](https://bbs.kanxue.com/elink@e14K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6f1P5h3W2D9L8#2)9J5c8X3#2S2j5$3S2G2i4K6g2X3k6h3c8A6N6l9`.`.)下载源码。

编译后可以得到`macho_edit`可执行文件。

用于编辑App的可执行文件。

## Hopper Disassembler

请自行搜索下载安装。

用于反编译App。

## Impactor

从[Cydia Impactor官网（28dN6%4N6%4i4K6u0W2j5%4W2V1K9h3q4A6L8i4m8S2j5%4c8G2M7W2)9J5k6h3y4G2L8b7`.`.）](https://bbs.kanxue.com/elink@b0fK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3y4&6k6r3W2S2K9h3#2H3j5h3y4@1L8%4u0Q4x3X3g2U0L8$3@1`.)下载。

用于安装ipa到手机。

## bfinject

从[33dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6n7K9i4y4Z5L8%4m8r3L8%4S2Q4x3V1k6T1k6X3W2F1K9X3g2U0N6l9`.`.](https://bbs.kanxue.com/elink@32fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6n7K9i4y4Z5L8%4m8r3L8%4S2Q4x3V1k6T1k6X3W2F1K9X3g2U0N6l9`.`.)下载。

根据说明文档，将`bfinject`放到iPhone中。

用来砸壳、动态库注入等。

## applst

从[5c0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6p5k6i4k6A6e0r3g2G2i4K6u0r3b7i4m8H3e0r3W2K6N6l9`.`.](https://bbs.kanxue.com/elink@b40K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6p5k6i4k6A6e0r3g2G2i4K6u0r3b7i4m8H3e0r3W2K6N6l9`.`.)下载。

根据说明文档，将命令行版的`applst`放到iPhone中。

用于列出手机上安装的App及文档目录。

## 《干掉月亮》(Shoot The Moon)

从[App Store](https://bbs.kanxue.com/elink@bc0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6N6s2g2F1k6i4y4Q4x3X3g2S2M7s2m8D9k6g2)9J5k6h3y4G2L8g2)9J5c8X3y4F1i4K6u0r3j5i4m8H3i4K6u0r3K9h3b7^5x3o6V1K6z5e0l9^5z5e0x3`.)下载。

这是一款有趣好玩的游戏。

# 破解

开局一台机，过程全靠猜。

下面正式进入破解阶段，我会非常啰嗦地写下所有的步骤。

如果您有一定地开发经验，尽情跳过所有已会的步骤。

## 砸壳

从App Store下载下来的App都是加密的，直接复制出来是无法使用Hopper Disassembler看到任何代码的，所以我们需要砸壳解密。

### 使用usbmuxd

如果你的Wifi环境不是很理想，可以使用`usbmuxd`映射端口走usb线。

此处我们映射iPhone上的`ssh`端口22到macOS上的2222端口，输入以下命令。

```
$ iproxy 2222 22

```

### ssh登录iPhone

默认密码是 `alpine`。

```
# -p 后面跟端口，默认是22

$ ssh root@localhost -p 2222

root@localhost's password: # 输入密码，默认密码是alpine

```

### 查询名称及进程ID

启动《干掉月亮》，查询App名称和进程ID。

因为《干掉月亮》的英文名是《Shoot The Moon》，所以我们在搜索进程的时候使用`grep`来筛选出带有moon的输出行。

```
$ ps -e | grep moon

1975 ??         0:59.42 /var/containers/Bundle/Application/24546F40-53DA-4C49-81C0-DED0C13DB814/shootthemoon.app/shootthemoon

```

### 砸壳

有了App名称和进程ID，我们就可以使用`bfinject`来砸壳了。

```
# 以下两条命令二选一，推荐使用App名称



# 使用App名称

$ bash bfinject -P shootthemoon -L decrypt



# 使用进程ID

$ bash bfinject -p 1975 -L decrypt



# 输出结果

[+] Liberios detected

[+] Injecting into '/var/containers/Bundle/Application/24546F40-53DA-4C49-81C0-DED0C13DB814/shootthemoon.app/shootthemoon'

[+] Getting Team ID from target application...

[+] Thinning dylib into non-fat arm64 image

[+] Signing injectable .dylib with Team ID A6DVR4V967 and platform entitlements...

[bfinject4realz] Calling task_for_pid() for PID 1975.

[bfinject4realz] Calling thread_create() on PID 1975

[bfinject4realz] Looking for ROP gadget... found at 0x1826574e0

[bfinject4realz] Fake stack frame at 0x10b624000

[bfinject4realz] Calling _pthread_set_self() at 0x182897804...

[bfinject4realz] Returned from '_pthread_set_self'

[bfinject4realz] Calling dlopen() at 0x182657460...

[bfinject4realz] Returned from 'dlopen'

[bfinject4realz] Success! Library was loaded at 0x1c41e2d00

[+] So long and thanks for all the fish.

```

命令执行完成并不表示砸壳已经完成，需要以App界面的提示为准。

下图为砸壳中，务必保持屏幕常亮。

![](https://bbs.kanxue.com/upload/attach/201803/320395_25GJHW5W5CG8RTX.png)

下图为砸壳完成。

![](https://bbs.kanxue.com/upload/attach/201803/320395_CBK6RCGN8E78AFW.png)

如果你会使用NetCat，选择YES，将砸壳后的App传到电脑上。否则选择No，稍后使用`scp`来传输文件。

砸壳后的文件在App的文档目录下，名字为`decrypted.ipa`。

**注意**：

1、`bfinject`命令必须在`bfinject`目录下执行，在其它目录下执行会出错。

2、如果砸壳过程中出现闪退，按Ctrl+C退出，重启App，再执行一遍砸壳命令即可。

### 找到《干掉月亮》的App文档目录

此处使用工具`applst`查询App的文档目录。

```
# applst默认会列出所有用户安装的App以及BundleID，

# 使用grep过滤只显示《干掉月亮》的BundleID

$ applst | grep shoot

$ applst com.pipsqueakgames.shootmoon

# 此处会显示《干掉月亮》的相关信息

localizedName:

Shoot Moon



shortVersionString:

1.6



vendorName:

Shaun Coleman



applicationIdentifier:

com.pipsqueakgames.shootmoon



itemID:

809390893



itemName:

Shoot The Moon



teamID:

A6DVR4V967



bundleURL:

file:///private/var/containers/Bundle/Application/24546F40-53DA-4C49-81C0-DED0C13DB814/shootthemoon.app



dataContainerURL:

file:///private/var/mobile/Containers/Data/Application/99B25BD3-77BA-47F0-9503-46F912C5A2E4



applicationType:

User

```

其中`bundleURL`就是App的目录，`dataContainerURL`是App的文档目录，数据、缓存、临时文件都存储在这个目录。

**注意**：少数情况下，一个App可能拥有多个`dataContainerURL`，`applst`只能列出其中一个。

### 传输砸壳后的文件

电脑上输入以下命令，将`Documents`目录下的`decrypted-app.ipa`传到电脑上。

```
# -P 后面跟端口，默认是22

$ scp -P 2222 root@localhost:/private/var/mobile/Containers/Data/Application/99B25BD3-77BA-47F0-9503-46F912C5A2E4/Documents/decrypted-app.ipa ./

root@localhost's password: # 输入密码，与ssh登录时的密码相同

```

## 破解

### 解压砸壳后的ipa

如下图，选中ipa文件，右键选择解压工具解压，使用系统默认的解压工具即可。

![](https://bbs.kanxue.com/upload/attach/201803/320395_42V4GCJ676QNF4K.png)

### 备份可执行文件

解压后得到一个文件夹叫`Payload`，里面有`shootthemoon.app`，其实是一个文件夹。

如下图，选中后右键选择显示包内容。

![](https://bbs.kanxue.com/upload/attach/201803/320395_94EMFQHAZ8DCG7V.png)

找到`shootthemoon`没有后缀名的文件，这是一个砸壳后的Mach-O格式的可执行文件。

复制一份到其它目录，准备拿它开刀。

### 使用Hopper Disassembler

启动Hopper Disassembler，菜单栏中选择File->Read Executable to Disassembler...，或者按下快捷键Command+Shift+O，弹出的对话框中选择`shootthemoon`文件，再点击OK。

紧接着会弹出如下图所示的对话框，保持默认选项，点击OK。

![](https://bbs.kanxue.com/upload/attach/201803/320395_MNJZYT6JAUFVJ9M.png)

等待解析完成后，File->Save或Command+S保存一下，以免闪退后又要重新解析。

### 猜猜猜

虽然Hopper Disassembler已经为我们解析出来大部分的方法名，但是我们不知道该从何下手。

与其瞎忙活，不如先玩一盘再说。

![](https://bbs.kanxue.com/upload/attach/201803/320395_X2SP8Q25RQC8Z58.png)

哇哦，10分！在游戏中靠近顶部的地方有一条隐隐约约的横线，当目标正好压在这根线上时，击中会得到10分，连着击中3次10分可以获得双倍分数的加成，而在那根线下面击中时分数会递减，加成也会消失，那就有思路了，我们把那根线移到最下面，这样我们就可以不管目标在哪个位置时，击中都能获得10分。

让我们回到Hopper Disassembler，在左侧列表中输入“ten”，看看能不能搜索到有关设置10分线位置的方法。

![](https://bbs.kanxue.com/upload/attach/201803/320395_W5NC6ASKD7B9RYK.png)

哇哦，运气太好了，这么容易就找到了，`MainEnemy`类里面有一个方法名叫做`tenPointLineY`的方法，看名字应该是10分线的Y坐标，如果能将Y坐标改为屏幕最底部就能达到目的了。

嗯？你想直接改汇编代码？这对于新手来说太难了，我们用高级语言来实现我们想法。

### 编写动态库

打开Xcode，创建一个Project，选择Cocoa Touch Framework，名字就叫`ShootMoonHacker`。

首先新建文件，选择Cocoa Touch Class，名字叫`HackerLoader`，继承`NSObject`类，动态库加载时初始化的代码写在这个类中。

再新建一个文件，选择Objective-C File，文件类型选择Category，类选择`NSObject`，名字叫`Hacker`，所有的破解代码都写在这个分类中。

工程目录结构如下图。

![](https://bbs.kanxue.com/upload/attach/201803/320395_UDTKYWYRJYFGU6M.png)

#### ShootMoonHacker.h

这个文件是创建工程时自动创建的，我们将需要公开的类全部写在这个文件里即可。
