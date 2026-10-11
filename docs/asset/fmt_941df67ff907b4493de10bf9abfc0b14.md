
# 用 OllyDBG 调试 Explorer.exe 的两种办法

> 原文：[`www.pediy.com/kssd/pediy10/74872.html`](https://www.pediy.com/kssd/pediy10/74872.html)

## 前言

腾讯比赛中第一阶段第四题是要利用 `StretchDIBits` 函数的漏洞，漏洞还算好找，用 `IDA` 打开很快就能看到可疑点，但难的是调试验证。因为 `Explorer.exe` 进程在 Windows 中的特殊性，所以调试起来不太容易，如果采用 `OD` 直接 `Open` 或 `Attach` 的方式，要么是断不下来，要么就是导致系统没有响应。尽管 `forgot` 等大牛已经提示过了要修改 `DesktopProcess`，但本人脑瓜较木，一直不明白是啥意思，这团阴影缠绕了我一星期。刚好昨天下午还算空闲，就决心要把这个问题搞定，于是在 `Google` 中找啊找，先是用了 `shoooo` 大的“可注入 `Explorer` 的 `OD`”，可是不知是否我的用法不对，还是不管用，最后终于看到了这篇 MS 的文档：`Debugging with the Shell`。

[`http://msdn.microsoft.com/en-us/library/cc144064(VS.85).aspx`](http://msdn.microsoft.com/en-us/library/cc144064(VS.85).aspx)，经过反复验证确定可行后，想到调试 `Explorer.exe` 的方法在网上很难找到，于是便有了下文。

## 第一种：神奇的 CTRL+ALT+SHIFT

`OD` 菜单：`File -> Open`，选择 Windows 目录下的 `Explorer.exe`，打开之后，`OD` 将会自动中断在入口点，这时在系统中会存在 2 个 `Explorer.exe` 进程，如下图：

![](img/fdc0d6c0bed0053965d45b48adebddba.jpg)

接下来从“开始”菜单或者“任务管理器”中选择“关机”，系统弹出关机对话框，如下图：

![](img/572e308b5018b6b22ae9bcae0d1a3dd0.jpg)

此时很关键了，要在按住 `CTRL+ALT+SHIFT` 三个键的同时，点击“取消”按钮，注意：必须是同时才有效，即先按下三个键，然后再点击。屏幕会在短暂的灰屏后重新亮起，原来的 `Explorer.exe` 已经被关闭，但是被 `OD` 调试的那个还在，如下图：

![](img/c3adcd9708389446ba8b0d948f5ee2b9.jpg)

之后你会发现桌面和任务栏都不见了，但不用担心，这是正常的，你原来打开的所有应用程序都还健康地活着，在 `OD` 下需要的断点，然后按 `F9` 运行，这时的 `Explorer` 就像普通进程一样开始运行，可以被 `OD` 任意调试了。

按下 `CTRL+ALT+SHIFT` 三个键表示什么意思呢？我也不知道，在 `Google` 上也没找到，要是有谁知道的，请一定要告诉我。![](img/792ac17acdd6fd3e39fc1aacdcf4c3d9.jpg "124.gif")

## 第二种：修改注册表

注册表 `HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer` 键下新建一个 `DWORD` 值为 1，修改之后如下图：

![](img/dfc9a6d9d2c920dbff0d95dc40cc03dc.jpg)

这个值的意思是让 Windows 中的桌面和任务栏使用一个单独的 `Explorer.exe` 进程，而其它的文件浏览等操作则使用另外一个 `Explorer.exe` 进程。修改后进行注销再重新登录，使修改生效。之后就和第一种方法一样了，操作 `OD` 菜单：`File -> Open`，选择 Windows 目录下的 `Explorer.exe`，在 `OD` 下需要的断点，按 `F9` 运行，用被 `OD` 调试的这个 `Explorer.exe` 进行文件浏览等，当触发了断点后就会被断下。

如果用 `IDA` 创建了系统 DLL 的 `MAP` 文件，还可以用 `OD` 插件 `LoadMapEx` 加载到 `OD` 中，这样在 `OD` 中也能看到 MS 的调试符号，会把系统对 `Explorer.exe` 的调用流程看得更清楚。以 `Gdi32.dll` 中的 `StretchDIBits` 函数为例，在 `Explorer.exe` 中下了 `StretchDIBits` 断点后，当该断点被触发时，从我机器上的 `CallStack` 窗口可以看到这样的流程：`Shell32 -> User32 -> Gdi32.StretchDIBits`。如下图：

![](img/1cb3542ec99816edd42297f8b845e8d7.jpg)

## 总结

这两种方法比较，尽管第一种方法感觉有点酷，不过从稳定性方面考虑，个人还是推荐第二种方法。
