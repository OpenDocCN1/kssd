
# `[mod]EasySYS 0.3.2.6` for `vs2008`/`WDK7`

> 原文：[`www.pediy.com/kssd/pediy11/106632.html`](https://www.pediy.com/kssd/pediy11/106632.html)

这个软件的作用是

一键建立 `VC+DDK` 的驱动开发环境，

并且创建好应用层和驱动层的开发模板～～～

![](img/6c3e4dbfdaf406bad4a735324b5929a6.jpg)

我是在在苏大米大牛和 `ChiChou` 大牛和 `bobo` 大牛的的基础上修改的，

原来的版本支持到 `WDK6` 和 `VS2005`，

我加了 `vs2008`/`WDK7` 的对应，

并且增加了 `SeLoadDriverPrivilege` 提权的操作，否则驱动 `load` 不上来，

还有注册表操作的那一块感觉有点乱，稍微整理了一下。

大家如果要自己定制 `EasySYS` 代码的话，记得要装一个 `VS2008` 的补丁

对应 `KB948127` - `LNK1000` error when incrementally linking a Visual C++ Project 的错误，

补丁的地址是

`http://code.msdn.microsoft.com/KB948127/Release/ProjectReleases.aspx?ReleaseId=871`

希望对大家开发驱动能有帮助，谢谢～～～

## 上传的附件

| ![](img/1baac1a3eff5ac15dae6f33dccc134d9.jpg) || `EasySYS_src_0.3.2.6.2.zip` |
