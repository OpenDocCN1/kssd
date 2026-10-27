
# [原创]HackShield分析(1)

我学习研究了一段时间的hackshield，写一些资料，与大家共享。共同学习，共同进步。

主要有三部分。

## 第1 HackShield的组成

1. `EhSvc.dll`：游戏和Hack Shield的通信dll。
2. `V3Pro32s.dll`：HackShield的anti tools侦测dll。
3. `3N.mhe`：样式库，可以通过一些规则来查找hack tool。
4. `psapi.dll`：进程信息库。
5. `V3Warp(d)(n)s.v3d`：和`3N.mhe`类似，需要`3N.mhe`工作。
6. `EagleNT.sys`：地球人都知道的，HS的驱动库，呵呵，hook `ntopenprocesss`, `read/writeprocessmemory`,...等函数。

## 第2，HackShiled的流程

![](https://bbs.kanxue.com/upload/attach/201001/102836_xpt2zg1gklqtc3l.jpg)

这个图是HackShield的工作流程。

![](https://bbs.kanxue.com/upload/attach/201001/102836_4pkgtgyhdnfboma.jpg)

## 第3，byPass的一些想法

HackShiled通过`CallBack`函数，通知主程序一些异常信息，比如"侦测到...工具"；可以hook这个函数，来bypass。

查找方法，IDA，找关键字符，比如"侦测到违法工具"，然后找到处理这些字符的函数。`.nop`掉。通常游戏就不会再弹出对话框，也不会关掉游戏了。

(....内容有些多，稍后我会不定期更新下，）

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [w1.JPG](javascript:void(0);)（26.45kb，2406次下载）
- [w2.JPG](javascript:void(0);)（19.01kb，2397次下载）
