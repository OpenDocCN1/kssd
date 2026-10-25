
# `xara` `3d4` `4.0`

> 原文：[`www.pediy.com/kssd/pediy01/forum090.htm`](https://www.pediy.com/kssd/pediy01/forum090.htm)

程序下载：

`http://www.newhua.com.cn/down/xara3d4n.exe`

一个作 3D 动力图像的东西!!有网友要求破的^\_^

\*\*\*\*\*方法如下：

一、去掉 `nag`： 查找 `x3d.exe` 中 `3BC77452F6462410` (bpx `00458572` 处设断)

`----75----------`

二、去掉时间：查找 `x3d.exe` 中 `84C00F848B0000006A68` (bpx `00425688` 处设断)

`------85------------`

总结一下：只用改两个字节!^\_^

你作的是个 `quick` `crack`，是最省事的办法。

用 `RegMon` 看一下，发现它启动的时候试图读如下的键：

`HKEY_CURRENT_USER\Software\Xara\X3D4\Install\Key`。

伪造该键，用 `bpx RegQueryValueExA do "d \*(esp+8)"`设断点，

跟踪它启动的过程，发现它是用如下的几个键来生成 `Key` 的，如此

可以写出注册机。

`HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\RegisteredOrganization`

`HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\RegisteredOwner`

`HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\FirstInstallDateTime`

也就是说，它是不会记住 `unlock` `code` 的，启动的时候根本不判断 `unlock` `code`，

只是在输入 `unlock` `code` 之后对 `unlock` `code` 判断一下， 然后生成 `Key`。只要把它生成 `Key` 的代码拷贝下来就成注册机了。

象这样的软件还碰到过不少，即启动的时候不判 `unlock` `code`，而是判断另外的东西。比如 `Oska` `Deskmate`（一个在桌面上跑来跑去的小熊）等。
