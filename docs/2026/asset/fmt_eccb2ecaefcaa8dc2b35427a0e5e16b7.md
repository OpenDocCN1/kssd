
![图片描述](upload/attach/201912/832445_JZRMPS469KRDKBJ.png)

第五层

由上层解出来的可见`HE()`函数，之前在`powershell`里定义了该函数，可以直接删除`ms`并转储一下为`7.ps1`。

![图片描述](upload/attach/201912/832445_XBTZ2QXRU4HRFAT.png)

现在解出来的代码勉强可读，需要自己整理一下。看的不清楚的可以自己在`powershell`中执行一下会更加直观。

# 总结

`powershell`解混淆其实并不太难，只是刚接触的时候会不太明白，无从入手。而且比较费眼神，“（）”等成对出现的符号都要一一配对，否则就会出错。

并且对于通常常用的混淆方法

![图片描述](upload/attach/201912/832445_3GHBZZEPYAS9E2Q.png)

我们可以直接在`powershell`里逐条运行，`powershell`就会反馈给我们它本身的样子。

最后还有大佬们还有什么好的`powershell`反混淆的方法或者工具吗？这手工反混淆太需要耐心、太费眼睛了

# 目录

- [前言](https://bbs.kanxue.com/#前言)
- [自动化反混淆](https://bbs.kanxue.com/#自动化反混淆)
- [混淆原理](https://bbs.kanxue.com/#混淆原理)
- [自动化反混淆工具](https://bbs.kanxue.com/#自动化反混淆工具)
- [手动反混淆](https://bbs.kanxue.com/#手动反混淆)
- [总结](https://bbs.kanxue.com/#总结)

# 前言

Windows PowerShell 是一种命令行外壳程序和脚本环境，使命令行用户和脚本编写者可以利用 .NET Framework的强大功能。

它引入了许多非常有用的新概念，从而进一步扩展了您在 Windows 命令提示符和 Windows Script Host 环境中获得的知识和创建的脚本。

Windows PowerShell v3将伴随着Microsoft Hyper-V 3.0和Windows Server 2012发布。PowerShell v3是一个Windows任务自动化的框架，它由一个命令行shell和内置在这个.NET框架上的编程语言组成。

PowerShell v3采用新的cmdlet让管理员能够更深入到系统进程中，这些进程可以制作成可执行的文件或脚本（script）。一条cmdlet是一条轻量命令，Windows PowerShell运行时间在自动化脚本的环境里调用它。

目前在流行的病毒里，还有许多使用了powershell。office宏病毒通过宏代码调用powershell执行恶意行为、powershell无文件挖矿等。特别是无文件挖矿的反射型注入，被许多人称为老掉牙的技术了，但是在9102年了还有许多用户中招。分析这类powershell病毒时，无法避免的是面对一堆乱码————经混淆的powershell代码。一些经过高度混淆的代码，简直可以称为“非人般的操作”了。

# 自动化反混淆

## 混淆原理

网上帖子太多了，可以参考下看雪论坛用户发的翻译贴：

https://bbs.pediy.com/thread-248034.htm

## 自动化反混淆工具

目前搜了搜，网上能自动反混淆的工具

Unit42安全团队编写的`PowerShellProfiler.py`：

github地址：

58aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6H3j5h3&6Q4x3X3c8#2L8X3W2@1y4o6u0Q4x3V1k6H3N6h3u0D9K9h3y4Q4y4h3k6@1L8$3!0D9M7#2)9J5c8Y4c8J5k6h3g2Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3V1k6H3L8%4N6W2M7Y4y4Z5k6h3I4D9M7s2u0G2k6X3W2D9k6i4t1`.

用法及原理参考：

be9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2X3M7X3g2W2j5Y4g2X3i4K6u0W2j5$3!0E0i4K6u0r3M7$3g2U0N6r3!0G2L8q4)9J5c8U0t1I4z5e0l9#2y4#2)9J5k6h3S2@1L8h3H3`.

在线解码工具CyberChef：

6b9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4j5$3S2I4i4K6u0W2k6$3W2@1K9s2g2T1i4K6u0W2K9h3!0Q4x3V1k6o6P5h3u0W2M7V1y4Z5k6h3k6Q4x3V1j5`.

使用方法参考：

94fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2K6k6h3y4H3N6h3I4K6k6g2)9J5k6h3y4G2L8g2)9J5c8X3q4J5j5$3S2A6N6X3g2K6i4K6u0r3x3e0l9&6z5e0p5J5i4K6u0W2K9s2c8E0L8l9`.`.

以上两款工具试着使用了下

一些简单的混淆能够解开，但是一些经过好几层的混淆代码就不太好使了。

# 手动反混淆

上次遇到了一个混淆次数较多，对我来说算得上是“高度混淆”样本，记录一下我的手动反混淆过程。

原混淆代码：
