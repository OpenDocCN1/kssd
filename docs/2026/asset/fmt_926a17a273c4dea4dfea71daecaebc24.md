
# [分享]yoda's protector脱壳思路

yoda's protector主要有两个需要注意的反调试保护：阻塞设备输入和枚举进程pid并终止调试器

## 阻塞设备输入

yoda 使用 `user32.dll`中的`BlockInput`来屏蔽所有的输入设备的消息，比如鼠标键盘。当设备被屏蔽时你无法进行任何操作，只有阻塞设备的线程或进程可以使用相同的API填入不同的参数来解锁它

yoda protector会在使用api阻塞设备后转向脱壳/解密的进程并执行一些反调试的技巧，在脱壳结束并且所有检查通过时，yoda才会解锁设备。

一个非常简单的方法可以解除阻塞，那就是patch掉`BlockInput`，让它什么都不做就返回。

当我们用`od`载入`unpackme`，我们可以看到内存中只载入了两个dll

![](https://bbs.kanxue.com/upload/attach/201712/722644_49fvu0ztysyfpol.jpg)

为了patch掉`BlockInput`，我们需要让`user32.dll`载入内存，在`od`选项中我们设置断在新的dll上

![](https://bbs.kanxue.com/upload/attach/201712/722644_do1yallmldttgfj.jpg)

然后我们继续`F9`，让`user32.dll`导入到内存

![](https://bbs.kanxue.com/upload/attach/201712/722644_sgaw7xerhm5h9ls.jpg)

随后我们就可以取消掉之前断在新dll的选项，然后跳到`BlockInput`的入口代码处

![](https://bbs.kanxue.com/upload/attach/201712/722644_2uc6govoj4w6et5.jpg)

![](https://bbs.kanxue.com/upload/attach/201712/722644_t8tfazaj0srm3p1.jpg)

`BlockInput`的api就是上图的灰色代码，`retn 4`就是结尾，我们只需要将所有这些代码填充为`nop`就行

![](https://bbs.kanxue.com/upload/attach/201712/722644_fvx6n0o1m9526hj.jpg)

这样，api就不会进行任何操作，我们的输入设备也不会被阻塞

## 枚举进程并终止调试器

yoda使用`CreateToolhelp32Snapshot`来获取所有正在运行的进程，然后yoda会搜索启动`unpackme`的进程是否和`unpackme`自己的进程PID是否相同，如果不同，那么yoda就会终止掉该进程(如`od`)。如果我们像先前一样patch掉`CreateToolhelp32Snapshot`，程序则会产生句柄非法(`Invalid_Handle`)异常。这里有另外一种方式来绕过保护。yoda使用`GetCurrentProcessId`来获取PID，因此我们可以控制返回的PID的值来迷惑保护代码。

比如我们启动`unpackme`的进程是`Ollydbg.exe`，我们就要让`GetCurrentProcessId`返回`Ollydbg.exe`的PID，这样检查就不会出现问题（而不是`unpackme`的PID）。

首先我们必须知道`ollydbg.exe`的PID，这点我们可以启动`LordPE`来轻松获得

![](https://bbs.kanxue.com/upload/attach/201712/722644_41esohgprhqad8p.jpg)

这里`ollydbg.exe`的PID是`0x4B4`，而`unpackme`的PID是`0x1CC`，我们要做的就是使得`GetCurrentProcessId`的返回结果是`0x4B4`

像之前一样，来到api的入口代码处

![](https://bbs.kanxue.com/upload/attach/201712/722644_ryflqoyrweunnft.jpg)

函数通过`eax`返回值，我们只需要修改`eax`即可

![](https://bbs.kanxue.com/upload/attach/201712/722644_y2qyqoqyajeca6i.jpg)

这样也就完成了。

在patch完后，还需要使用插件来隐藏`Ollydbg`（用`HideOD`等插件，避免`IsDebuggerPresent`的检查），之后的关键就是找OEP了。

## OEP查找

oep的查找我们可以使用最后一次异常法来跟踪，按下`shift+F9`的最后一次异常，在堆栈窗口中出现的SE Handler，本例中是`00413F50`，我们跟踪进去，在`00413F50`设下断点，然后按下`Shift+F9`到达断点处。我们要记得开启`HideDebugger`或类似插件的反`IsDebuggerPresent`的选项，否则在使用最后一次异常法时会退出`OD`和`unpackme`并且无法再点击任务栏和图标。

![](https://bbs.kanxue.com/upload/attach/201712/722644_57pyeg79zpd92q9.jpg)

然后在`00413F79`处的`00404000`就是我们的OEP了，dump出来修复IAT即可

![](https://bbs.kanxue.com/upload/attach/201712/722644_zyljdhg2gk9cypc.jpg)

主要的脱壳思路就是以上，如果想要练手的话，可以下载附件，附件程序就单纯加上`Hide debugger`的反`IsDebuggerPresent`然后用最后一次异常法就能解决了，分享给大家学习。

ps：附件无毒，由于检测到PID不一致的话会无法点击任务栏，建议在虚拟机下运行。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [files.zip](javascript:void(0);)

  （31.15kb，49次下载）
