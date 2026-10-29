
# [原创]发个Android平台上的注入代码

rt,其实本质上就是linux下面的library注入

只不过处理器是arm的，需要额外处理一些东西

不少代码是从网上或xx直接借鉴的，谢谢这些代码的作者

整体流程分3步

1.在目标进程中分配内存，用来写shellcode和参数

2.往目标进程中写入shellcode, shellcode会调用`dlopen`来载入我们的library

3.运行目标进程中的shellcode

当然注入是需要root权限的，模拟器上可以直接用，但是真机的话，先exploit吧

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [`LibInject.zip`](javascript:void(0);)（3.91kb，6182次下载）
