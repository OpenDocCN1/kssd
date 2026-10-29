
# [转帖]端口渗透总结

## 0x00 背景

在前段时间的渗透中，我发现通过端口来进行渗透有时会提升我们的效率，所以才有了这篇文章的诞生；

首先分享一份关于端口及他们对应的服务文件：[5eeK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6&6N6h3&6H3j5h3&6Q4x3X3g2U0L8W2)9J5c8X3y4k6P5f1&6j5c8i4m8K9e0W2W2$3P5q4p5`.](https://bbs.kanxue.com/elink@ae5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6&6N6h3&6H3j5h3&6Q4x3X3g2U0L8W2)9J5c8X3y4k6P5f1&6j5c8i4m8K9e0W2W2$3P5q4p5`.) 访问密码 983e

这里再分享一篇我曾经在百度文库提交的端口渗透文章：[请点我](https://bbs.kanxue.com/elink@3bbK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6W2L8X3E0#2i4K6u0W2j5X3q4A6k6s2g2Q4x3X3g2U0L8$3#2Q4x3V1k6D9K9h3&6C8i4K6y4r3N6i4u0D9i4K6y4p5e0g2f1K6j5$3c8a6i4K6g2X3e0K6g2U0y4f1&6x3M7f1S2G2b7f1S2^5y4h3x3#2N6K6q4g2d9#2u0W2b7K6W2S2k6e0j5H3P5V1g2x3x3W2N6g2c8r3&6i4k6g2u0k6x3%4c8D9M7%4A6&6f1@1!0d9c8U0c8K9g2g2y4z5z5q4q4k6d9%4l9I4f1f1S2Q4y4h3k6X3P5Y4S2o6g2i4N6@1y4#2g2%4b7W2m8q4x3q4)9J5k6q4k6v1h3r3!0n7L8$3I4Q4x3X3c8K9P5q4u0J5f1$3y4Y4e0s2m8I4M7h3k6G2k6b7`.`.)

再次看这篇文章发现写的很简单，也只描述了几个常见的端口渗透；而且一般我们都是可以修 改默认端口的，所以平时在渗透过程中，对端口信息的收集就是一个很重要的过程；然后对症下药就可以更快的渗透进入我们需要的服务器；接下来就详细通过渗透 实战对端口的渗透进行更加深入的剖析；

端口渗透过程中我们需要关注几个问题：
1、 端口的banner信息
2、 端口上运行的服务
3、 常见应用的默认端口

当然对于上面这些信息的获取，我们有各式各样的方法，最为常见的应该就是`nmap`了吧！我们也可以结合其他的端口扫描工具，比如专门的`3389`、`1433`等等的端口扫描工具；

### 服务默认端口

公认端口(Well Known Ports)：`0-1023`，他们紧密绑定了一些服务；
注册端口(Registered Ports)：`1024-49151`，他们松散的绑定了一些服务；
动态/私有：`49152-65535`，不为服务分配这些端口；

当然这些端口都可以通过修改来达到欺骗攻击者的目的，但是这就安全了吗？攻击者又可以使用什么攻击方式来攻击这些端口呢？

还需要注明的一点是：很多木马工具也有特定的端口，本文并没有涉及到这块的内容，大家可以自己去收集收集！

### 关于爆破之我见

在对这些端口进行实战讲解时，我需要先阐述一下我对爆破这个方式的一些看法；

爆破：技术最简单，需要的技术能力基本为0，工作效率与网络、硬件等相关，在我看来爆破其实是最强大的攻击方式，特别是结合一些特制的字典，结合社工我们可以在很短的时间达到最大的效果，只不过因为我们的pc或者字典不够强大，所以很多时候我们不能进行一次优秀的爆破攻击；当然现在很多web应用以及服务端口都限制了暴力破解；对于这种做了限制的我们可能就需要利用到本文提到的其他攻击了！

分享一个团队sai总结的字典：[请点击](https://bbs.kanxue.com/elink@7aeK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3S2W2P5i4y4W2j5#2)9J5k6h3!0J5k6#2)9J5c8X3q4J5j5$3S2A6N6X3g2K6i4K6u0r3y4o6M7^5)

声明：本文总结的都是近两年的常见漏洞，以前的老版漏洞以及危害性不大的漏洞没有总结，望大家谅解！

## 0x01 实战测试

### 文件共享服务端口渗透

#### ftp服务

`FTP`服务：ftp服务我分为两种情况，第一种是使用系统软件来配置，比如`IIS`中的`FTP`文件共享或`Linux`中的默认服务软件；第二种是通过第三方软件来配置，比如`Serv-U`还有一些网上写的简易ftp服务器等；

默认端口：`20`（数据端口）；`21`（控制端口）；`69`（tftp小型文件传输协议）

攻击方式：

爆破：ftp的爆破工具有很多，这里我推荐owasp的`[Bruter](https://bbs.kanxue.com/elink@6e9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6L8%4g2J5j5$3g2X3L8%4u0Y4k6g2)9J5k6h3&6W2N6q4)9J5c8Y4m8J5L8$3A6W2j5%4c8K6i4K6u0r3N6$3!0J5j5i4N6A6N6r3q4Q4x3V1j5`.)`以及msf中ftp爆破模块；

匿名访问：用户名：`anonymous` 密码：为空或任意邮箱

用户名：`FTP` 密码：`FTP`或为空

用户名：`USET` 密码：`pass`

当然还有不需要用户名密码直接访问的，一般出现在局域网中；

[![0](https://bbs.kanxue.com/upload/attach/201711/2YK9ZSWB9XN84UA.jpg)](https://bbs.kanxue.com/upload/attach/201711/2YK9ZSWB9XN84UA.jpg)

嗅探：ftp使用明文传输技术（但是嗅探给予局域网并需要欺骗或监听网关）

[![01](https://bbs.kanxue.com/upload/attach/201711/CUW5YSWHZAHWDQB.jpg)](https://bbs.kanxue.com/upload/attach/201711/CUW5YSWHZAHWDQB.jpg)

后门技术：在linux的`vsftp`某一版本中，存在着一个后门程序，只要在用户名后面加上 就会在`6200`上打开一个监听`Shell`，我们可以使用`telnet`直接连接；详细[请点击](https://bbs.kanxue.com/elink@151K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3k6J5k6h3g2T1N6h3k6Q4x3X3g2U0L8$3#2Q4x3V1k6S2M7Y4c8A6j5$3I4W2M7#2)9J5c8Y4y4&6M7%4c8W2L8g2)9J5c8U0x3@1y4e0M7I4i4K6u0W2K9s2c8E0L8l9`.`.)

远程溢出漏洞：`6.10.1 IIS FTP`远程溢出漏洞，在`IIS FTP`服务器中`NLST`命令存在一个缓冲区溢出漏洞，这个漏洞可能是攻击者在服务器运行一条非法命令。

跳转攻击：（Bounce Attacks）攻击者发送一个`FTP`”`PORT`”命令给目标`FTP`服务器，其中包含该主机的网络地址和被攻击的服务的端口号。这样，客户端就能命令`FTP`服务器发一个文件给被攻击的服务。这个文件可能包括根被攻击的服务有关的命令（如`SMTP,NNTP`等）。由于是命令第三方去连接到一种服务，而不是直接连接，就使得跟踪攻击者变得困难，并且还避开了基于网络地址的访问限制。（注：此种情况小白并没有遇到过，只是总结一下，欢迎大牛指教）

案例分享：

[山东电信Serv-U Web客户端弱口令](https://bbs.kanxue.com/elink@973K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4N6G2L8%4W2#2L8W2)9J5k6h3!0J5k6#2)9J5c8X3u0#2k6%4y4Q4x3V1k6%4L8$3!0&6N6h3&6Q4x3X3b7J5x3o6p5#2i4K6u0V1x3o6p5I4y4U0b7&6z5b7`.`.)

[长虹ftp弱口令导致全网数据泄漏](https://bbs.kanxue.com/elink@408K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4N6G2L8%4W2#2L8W2)9J5k6h3!0J5k6#2)9J5c8X3u0#2k6%4y4Q4x3V1k6%4L8$3!0&6N6h3&6Q4x3X3b7J5x3o6p5H3i4K6u0V1x3o6p5H3x3U0f1#2x3R3`.`.)

#### NFS服务

`nfs`：网络文件系统，允许网络中的计算机通过`TCP/IP`网络共享资源。基于`Linux`系统，配置方面很简单，详细配置请参考案例分享。在`nfs`配置中，有不做任何限制的，有限制用户，有限制`IP`，以及在版本`2.x`中我们还可以使用证书来验证用户。当然不同的限制可以采用的攻击方式也不一样；就目前而言网上关于`nfs`的攻击还是比较少的!

默认端口：`2049`

攻击方式：

未授权访问：未限制`IP`以及用户权限设置错误

案例分享：

[Nfs配置不当导致被入侵](https://bbs.kanxue.com/elink@a6dK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3c8J5L8%4m8K6i4K6u0W2N6$3!0G2P5i4g2F1i4K6u0W2L8%4u0Y4i4K6u0r3N6r3W2H3M7#2)9J5c8U0R3$3y4e0V1`.)

[NFS服务全攻略](https://bbs.kanxue.com/elink@66aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6e0x3$3x3r3c8G2j5#2)9J5k6h3y4G2L8g2)9J5c8X3y4G2L8Y4c8W2L8Y4c8Q4x3V1j5I4x3g2)9J5c8U0l9@1x3e0m8Q4x3V1j5I4y4q4)9J5c8U0t1H3y4e0b7J5z5o6g2Q4y4h3j5I4x3o6R3#2z5e0R3@1x3U0k6Q4x3X3g2K6K9s2c8E0L8l9`.`.)

#### Samba服务

`Samba`服务：对于这个可以在windows与`Linux`之间进行共享文件的服务同样是我们攻击的关注点；`samba`登录分为两种方式，一种是需要用户名口令；另一种是不需要用户名口令。在很多时候不光是pc机，还有一些服务器，网络设备都开放着此服务，方便进行文件共享，但是同时也给攻击者提供了便利。

默认端口：`137`（主要用户`NetBIOS Name Service`；`NetBIOS`名称服务）、`139`（`NetBIOS Session Service`，主要提供`samba`服务）

攻击方式：

爆破：弱口令（爆破工具采用`hydra`）`hydra -l username -P PassFile IP smb`

未授权访问：给予`public`用户高权限

远程代码执行漏洞：`CVE-2015-0240`等等

案例分享：

[Samba远程代码执行漏洞](https://bbs.kanxue.com/elink@4aeK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4y4W2j5%4m8#2L8s2y4W2i4K6u0W2j5$3!0E0i4K6u0r3j5i4u0U0K9r3W2$3k6i4y4Q4x3V1j5#2z5e0M7#2i4K6u0W2K9s2c8E0L8l9`.`.)

[未授权访问文件系统漏洞](https://bbs.kanxue.com/elink@145K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4c8W2j5$3S2Q4x3X3g2K6L8$3k6@1y4W2)9J5k6h3y4G2L8g2)9J5c8U0j5$3y4#2)9J5c8U0p5H3i4K6u0r3y4o6f1^5y4U0q4Q4x3X3g2Z5N6r3#2D9)

#### LDAP协议

`ldap`：轻量级目录访问协议，最近几年随着`ldap`的广泛使用被发现的漏洞也越来越多。但是毕竟主流的攻击方式仍旧是那些，比如注入，未授权等等；这些问题的出现也都是因为配置不当而造成的。

默认端口：`389`

攻击方式：

注入攻击：盲注

未授权访问：

爆破：弱口令

案例分享：

[LDAP注入与防御剖析](https://bbs.kanxue.com/elink@d78K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3c8J5L8%4m8K6i4K6u0W2N6$3!0G2P5i4g2F1i4K6u0W2L8%4u0Y4i4K6u0r3N6r3W2H3M7#2)9J5c8U0V1$3y4H3`.`.)

[欧朋LDAP服务匿名访问](https://bbs.kanxue.com/elink@d99K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4N6G2L8%4W2#2L8W2)9J5k6h3!0J5k6#2)9J5c8X3u0#2k6%4y4Q4x3V1k6%4L8$3!0&6N6h3&6Q4x3X3b7J5x3o6p5H3i4K6u0V1x3o6f1I4x3U0x3@1)

[使用LDAP查询快速提升域权限](https://bbs.kanxue.com/elink@916K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3c8J5L8%4m8K6i4K6u0W2N6$3!0G2P5i4g2F1i4K6u0W2L8%4u0Y4i4K6u0r3M7r3g2F1N6r3g2K6N6r3W2F1k6#2)9J5c8U0t1I4x3U0f1`.)

### 远程连接服务端口渗透

#### SSH服务

`SSH`服务：这个服务基本会出现在我们的`Linux`服务器，网络设备，安全设备等设备上，而且很多时候这个服务的配置都是默认的；对于`SSH`服务我们可能使用爆破攻击方式较多。

默认端口：`22`

攻击方式

爆破：弱口令、

漏洞：28退格漏洞、`OpenSSL`漏洞

案例分享：

[安宇创新科技ssh弱口令](https://bbs.kanxue.com/elink@bc6K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4N6G2L8%4W2#2L8W2)9J5k6h3!0J5k6#2)9J5c8X3u0#2k6%4y4Q4x3V1k6%4L8$3!0&6N6h3&6Q4x3X3b7J5x3o6p5#2i4K6u0V1x3o6p5K6y4e0f1K6x3l9`.`.)

[宜信贷某站存在OpenSSL漏洞](https://bbs.kanxue.com/elink@d78K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4N6G2L8%4W2#2L8W2)9J5k6h3!0J5k6#2)9J5c8X3u0#2k6%4y4Q4x3V1k6%4L8$3!0&6N6h3&6Q4x3X3b7J5x3o6p5H3i4K6u0V1x3o6p5#2y4o6R3J5z5b7`.`.)

#### Telnet服务

`Telnet`服务：在`SSH`服务崛起的今天我们已经很难见到使用`telnet`的服务器，但是在很多设备上同样还是有这个服务的；比如`cisco`、`华三`，深信服等厂商的设备；我就有很多次通过`telnet`弱口令控制这些设备；

默认端口：`23`

攻击方式

爆破：弱口令

嗅探：此种情况一般发生在局域网；

案例分享：

[大量惠普打印机远程telnet可被查看和操作](https://bbs.kanxue.com/elink@42cK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4N6G2L8%4W2#2L8W2)9J5k6h3!0J5k6#2)9J5c8X3u0#2k6%4y4Q4x3V1k6%4L8$3!0&6N6h3&6Q4x3X3b7J5x3o6p5#2i4K6u0V1x3o6p5$3x3U0M7H3y4l9`.`.)

#### Windows远程连接

远程桌面连接：作为`windows`上进行远程连接的端口，很多时候我们在得到系统为`windows`的`shell`的时候我们总是希望可以登录`3389`实际操作对方电脑；这个时候我们一般的情况分为两种。一种是内网，需要先将目标机`3389`端口反弹到外网；另一种就是外网，我们可以直接访问；当然这两种情况我们利用起来可能需要很苛刻的条件，比如找到登录密码等等；

默认端口：`3389`

攻击方式：

爆破：`3389`端口爆破工具就有点多了

Shift粘滞键后门：5次shift后门

`3389`漏洞攻击：利用`ms12-020`攻击`3389`端口，导致服务器关机；[请参考](https://bbs.kanxue.com/elink@951K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6Q4x3X3f1I4y4U0y4Q4x3X3g2U0L8$3#2Q4x3V1k6J5P5W2)9#2k6Y4S2A6j5h3!0B7K9h3q4Q4x3V1k6T1L8r3!0Y4i4K6u0r3M7%4c8S2N6r3W2U0i4K6u0r3x3e0p5&6x3U0l9&6x3e0V1^5x3U0l9I4x3U0f1I4x3e0V1#2x3K6b7H3y4U0t1&6i4K6u0r3)

#### VNC服务

`VNC`：一款优秀的远控工具，常用语类`UNIX`系统上，简单功能强大；也

默认端口：`5900+`桌面`ID`（`5901`；`5902`）

攻击方式：

爆破：弱口令

认证口令绕过：

拒绝服务攻击：（`[CVE-2015-5239](https://bbs.kanxue.com/elink@dd6K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3c8J5L8%4m8K6i4K6u0W2N6$3!0G2P5i4g2F1i4K6u0W2L8%4u0Y4i4K6u0r3M7r3q4H3k6i4u0K6i4K6u0r3z5e0l9I4y4l9`.`.)`）

权限提升：（`CVE-2013-6886`）

案例分享：
