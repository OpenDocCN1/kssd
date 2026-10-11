
# 魔装网神 2.7

> 原文：[`www.pediy.com/kssd/pediy04/kanxue130.htm`](https://www.pediy.com/kssd/pediy04/kanxue130.htm)

破解作者：`gemstone_3[BCG]`

软件作者主页：`http://www.fastale.com/cnindex.html`

软件名称：魔装网神 v3.20

软件下载地址：`http://www.fastale.com/downloads/netmyth.zip`

软件简介：

魔装网神是一个浏览器，主要特点如下：

* 可以更换界面(`Skins`)，软件更漂亮。
* 由于对系统缓存的最佳利用，软件的浏览速度比 IE 要快。
* 魔装网神占用的内存比其他全副武装的软件要少得多。
* 魔装网神平均每开 15 页占用 1% 系统资源，而一个 IE 就 4%。
* 能设置 255 个主页地址，方便打开你最常用的网页。
* 一些网站在状态栏有字幕滚动效果，阻碍了用户观察当前网页读取状况、链接地址，现在你可以关闭掉这种效果。
* 加入了黑名单功能，可以过滤掉指定的广告或不喜欢的网址。

脱壳使用软件：`UnPECompact` v1.31

破解使用软件：`FI` v2.45, `SmartCheck` v6.03

### 破解方法

1. 先用 `FI` 2.45 查出软件壳是 `PECompact` v0.976.1。
2. (脱壳) 运行 `UnPECompact` 1.31，点“开始”，打开 `netmyth.exe` (魔装网神可执行文件)，程序自动找到程序入口。只要把脱壳的文件另存为就可以了，我另存为 `cr_netmyth.exe`。
3. 运行 `cr_netmyth.exe`，点击菜单 -> 帮助 -> 关于 -> 注册。输入注册信息：
   * 用户名: `gemstone`
   * 注册码：`123456789` ( 当然是假的 :P )
4. 启动 `SmartCheck` 并载入运行 `cr_netmyth.exe`，经过很长一段时间的等待。魔装网神进入主界面，然后在 `SmartCheck` 的工具栏单击 `END` 图标结束魔装网神的运行。
5. 看一下 `SmartCheck` 左边的窗口有一个“+ _Load”字样，它实际上是程序启动的过程，点击 + 打开。出来很多内容，继续往下面看还有一个“+ _Load”字样，再点 + 图标，在“UCase”下找到“这是一个什么我忘了”和“gemstone”字样，再往下一看是“HEX”，这才是关键所在。
6. “HEX”在 `SmartCheck` 右边窗口中的内容如下：

```text
- string (variant)
  - signed lpng * .plVal = 006CF1B4
      Long = 这个是什么我忘了 0x0245A5D39 这就是我们要的注册码
```

破解时间：2 天多经过过数次失败，注册错 N 次。最后终于成功了（是在看雪老师的精华 III 里看到，有个 2.5 的破解，就照着做了一下，大家可以参考这个来做），谢谢看雪！！！。

`4dd466d0e1e0cc123afbe504da2462ca`
