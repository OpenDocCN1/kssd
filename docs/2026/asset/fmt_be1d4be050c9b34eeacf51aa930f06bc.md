
# [原创]使用 RouterSploit 攻击路由器的方法介绍

路由器是网络连接的核心设备，但是普通用户并不会太注意路由器的安全配置问题。老的设备固件系统，默认弱口令密码和其他配置问题都会被黑客利用。而且这些利用漏洞非常简单，以至于创建自动化工具来利用这些漏洞就变得轻而易举。

在这篇文章中，我会讲述如何使用RouterSploit，这是一种自动化路由器漏洞利用工具。

## 路由器漏洞利用的基础知识

路由器漏洞利用的原理是破坏路由器的Wi-Fi安全性，绕过管理登录页面和访问管理功能。然后，熟练的攻击者可以在“rootkitting”中定位路由器的固件信息，其中自定义的固件信息可以被用于高级恶意功能。

根据攻击者的攻击目标分析，攻击可能包括监视用户和连接的设备，将恶意软件注入Web管理中以利用连接的设备，实现高级鱼叉式网络钓鱼攻击，并通过被利用的路由器为非法流量路由提供犯罪活动。

## Cherry Blossom路由器漏洞利用工具

美国国家安全局和中央情报局等政府机构对路由器漏洞进行了囤积，ShadowBrokers在Window SMB漏洞被利用于WanaCry病毒之后发布这些路由器漏洞。现在像Cherry Blossom这样的工具在以后可能会成为路由器漏洞主流工具。

NSA和CIA的这些工具可以控制受感染路由器所在的整个局域网络，可以将它们转换为先进的无线间谍设备。

Cherry Blossom是一个rootkitting框架，其中路由器被自动利用并转换为“flytraps”。

flytraps

是一种被使用特殊固件更新的路由器，可以防止用户更新或修改新固件，被黑客广泛使用。

![](https://bbs.kanxue.com/upload/attach/201904/823237_KBJQ47UVGA7U972.jpg)

Cherry Blossom路由器漏洞利用工具可以控制许多“

flytraps （

捕蝇草

）”，称为访问位于家中或目标网络中的间谍设备。

flytraps （

捕蝇草

）将“

beacon

”回连到“Cherryweb”的命令和控制服务器，然后由操作者通过加密的VPN隧道分配“任务”。高级模块下，比如“Windex”，可以对任何连接的目标执行恶意软件攻击，可以将

flytraps （捕蝇草）

变成一个能够从任何地方进行控制的高级远程间谍平台。

![](https://bbs.kanxue.com/upload/attach/201904/823237_JBE7XKPZXZ3GAF7.jpg)

如图是Cherry Blossom显示要发送到

flytraps （捕蝇草）

设备的任务命令，包括shell代码，侦察脚本和漏洞利用代码

## 路由器黑客攻击活动

除了CIA关注的间谍工具之外，可以被利用的路由器和物联网设备因其路由能力而成为重要目标。RouterSploit，今天要使用的工具，不仅可以攻击路由器，它还可以用于网络摄像头和其他连接设备。

虽然CIA使用VPN连接来分析和控制服务器之间的流量，但网络犯罪分子将使用这些设备代理恶意流量以避免检测。实际上，这些被感染的路由器和物联网设备的网络作为代理攻击工具在黑市被出售，用于隐藏信用卡盗窃，暗网交易和DDoS攻击等非法活动。

![](https://bbs.kanxue.com/upload/attach/201904/823237_AJGAYVYNB5CRAK4.jpg)

## 路由器黑客攻击实战入门

如果拿到路由器的控制权，你将拥有对网络的完全访问权限。你可以控制目标设备的网络状况并将其路由到你想要的任何地或任何地方，或转发端口以进行远程访问。

初学者只需在RouterSploit上运行Autopwn扫描程序，也会自动检测出针对目标IP地址的一系列漏洞。

## 什么是RouterSploit？

RouterSploit是一个用Python写的框架，可以自动完成与路由器相关的大多数漏洞利用任务。它以Metasploit为模型，任何习惯Metasploit框架的人都会熟悉它的命令。它包含扫描和利用模块，可用于Kali Linux安装下载

与目标网络联网后，扫描后将显示是否可以通过框架轻松利用路由器，我将通过Autopwn功能快速识别路由器和连接设备上的漏洞。

![](https://bbs.kanxue.com/upload/attach/201904/823237_AVB76JYUUV3PSP8.jpg)

## 安装准备

RouterSploit可以在Kali Linux下安装，也可以在Kali Raspberry Pi，macOS或Mac OS X，Windows上运行，甚至可以在root后的Android手机上运行。首先，我们需要处理一些依赖项并确保安装了Python。

### 第1步：安装Python和依赖项

需要确保安装了Python，并且您还需要以下一些软件包。

```
Python3 (with pip)

Requests

Paramiko

Beautifulsoup4

Pysnmp

Gnureadline (macOS / Mac OS X only)
```

你可以使用`apt-get`安装它们：

```
apt-get install python3-pip requests paramiko beautifulsoup4 pysnmp
```

### 第2步：在Mac，Kali上安装RouterSploit

要在Kali Linux上安装，请打开终端窗口并键入以下命令：

```
git clone ee5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6@1K9s2u0W2j5i4b7&6i4K6u0r3M7X3!0#2N6r3g2J5M7%4m8D9L8$3W2@1

cd routersploit

python3 -m pip install -r requirements.txt

python3 rsf.py
```

在macOS或Mac OS X上，方法类似。在终端窗口中，键入：

```
git clone fa2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6@1K9s2u0W2j5i4b7&6i4K6u0r3M7X3!0#2N6r3g2J5M7%4m8D9L8$3W2@1

cd routersploit

sudo easy_install pip

sudo pip install -r requirements.txt
```

### 第3步：运行RouterSploit

首次运行请将计算机连接到有你要扫描的路由器的网络中，导航到RouterSploit文件夹并通过键入以下命令运行RouterSploit。

```
cd

cd routersploit

sudo python ./rsf.py
```

RouterSploit框架界面风格和Metasploit框架很相似

![](https://bbs.kanxue.com/upload/attach/201904/823237_MYN8HC6ZFPN9T4P.jpg)

命令行界面可以输入简单的命令来扫描和利用路由器漏洞，可以通过键入以下命令查看RouterSploit提供的所有功能：

```
show all
```

下面的输出中看到的有很多漏洞利用代码和扫描脚本
