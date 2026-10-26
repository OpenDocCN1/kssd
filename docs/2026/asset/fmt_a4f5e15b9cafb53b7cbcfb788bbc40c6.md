
# [原创] 使用iMonitor分析Visual Assist X的试用期存储位置

最近升级VS到了VS2022，VS已经是64位的了，智能提示非常流畅，但是还是想安装Visual Assist X看看有啥增强的，安装完发现试用期过期了。找了一圈发现没有针对64位的和谐补丁。于是就尝试能不能清理掉Visual Assist X保存的试用期配置信息，再试用一次。（试用发现好用再购买正版！！！）

于是开启`procmon`，想看看Visual Assist X保存的数据在哪里，神奇的事情发生了，竟然没有试用期提示了。好吧，原来`VAX`内部判断了如果开启了`procmon`，就做了一些特殊处理，不再操作数据了。发现只要使用`procmon`相同的窗口类名也可以达到相同的效果。

是的，事情到这里就结束了。

出于好奇，还是想知道`VAX`到底把数据存储在哪里，还害怕别人用`procmon`。把`procmon`的窗口信息隐藏了，就可以开始监控了。但是想了想，觉得还是自己实现一个类似`procmon`的工具更有成就一点。于是乎，开源版本的`procmon`就诞生了。

`iMonitor`（冰镜 - 终端行为分析系统）源码：9f3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6%4k6h3y4G2L8%4m8W2M7X3q4@1k6g2)9J5c8X3W2y4L8$3&6A6N6r3!0J5

启动`iMonitor`，设置进程为`devenv.exe`，过滤堆栈模块包括`VAX`（因为`VAX`加壳了，实际模块地址没有路径，为`NULL`）的写操作（文件、注册表）。

于是乎，`VAX`的试用期信息保存马上就出来了。

注册表的`Licenses`、`Whole Tomato`、临时目录的`1489AFE4.TMP`。

还有一个特别猥琐的位置，一个随机的`CLSID`位置（每个机器不一样，从系统的`CLSID`复制了一份配置，然后加了加密字段保存数据）。

把这几个位置的信息清理了，发现过期提醒就没有了，又能开心写代码了。

![图片描述](https://bbs.kanxue.com/upload/attach/202111/941522_7FSEXCETHHZHEGM.png)

详细过滤规则如下：

![图片描述](https://bbs.kanxue.com/upload/attach/202111/941522_UQT9K85SHCYA2FB.png)

这个分析是出于学习的目的，反对盗版，喜欢`VAX`的请购买正版软件。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2021-11-27 19:32 被iMonitor编辑，原因：

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm) [#系统底层](https://bbs.kanxue.com/forum-4-1-2.htm)
