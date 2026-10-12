
# Ida Pro Advanced 6.0 中木马分析

> 原文：[`www.pediy.com/kssd/pediy12/131195.html`](https://www.pediy.com/kssd/pediy12/131195.html)

今天看到论坛上有人发 `Ida Pro Advanced 6.0`，我也转载了一下，结果 N 多人说有木马，当时我也安装了一下，也没在意，后来分析了一下真的有木马。

点击 `IDA.Pro.Advanced.6.Installer.exe` 后在 `C:\WINDOWS\TXUONU\` 目录下创建几个文件

![](img/28afde17e8436b2fddcb1c3f73356599.jpg)

`OD` 分析后发现此木马会监视浏览器和通讯软件，对键盘进行记录，还有网页截图

![](img/7c82b72509671ffeff5c0ac538dabc1c.jpg)

修改注册表开机自动启动

![](img/337e9d01f98091cc54d35e3ca1d4d748.jpg)

还在进行更深入的分析。

木马样本在此，没事的可以分析玩玩。

**上传的附件**

| 附件 |
| :--- |
| `木马文件.rar` (解压密码：`muma`) |
