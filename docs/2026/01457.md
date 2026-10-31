
# [原创]沙箱Sandboxie v3.40 逆向完整源码

## 【前序】

[**Sandboxie**](https://bbs.kanxue.com/elink@3c1K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4y4S2L8X3c8T1L8%4S2A6k6g2)9J5k6h3y4G2L8g2)9J5c8R3`.`.) 是**土耳其**人2006年开发的一个**轻量级小型沙箱**，至今仍有很多人使用，作者也一直在维护更新。当时国内做沙箱的还没有几家，大概有Comodo、卡巴斯基等大的安全厂商做过内置沙箱，不过功能性偏重不同，用户体验跟sandboxie有很大差距。

我在09年的时候使用过Sandboxie一段时间，感觉做的不错，对其实现非常感兴趣，于是当时花了大量的业余时间进行逆向重写，整个过程耗时1年多。后来由于工作繁忙，此工程不再关注，代码就这样被搁置了三年多。

今天翻翻代码，觉得里面有些东西可拿出来分享，遂发此贴，旨在抛砖引玉。

## 【正题】

这个沙箱的名字叫：[**ProteinBox**](https://bbs.kanxue.com/elink@b70K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3S2A6i4K6u0W2j5X3q4A6k6s2g2Q4x3X3g2U0L8$3#2Q4x3V1k6H3M7X3!0@1k6h3W2F1j5X3!0^5i4K6u0r3K9i4c8W2L8g2)9J5c8U0c8U0y4r3b7%4j5$3j5$3k6h3q4T1x3e0V1%4x3o6u0U0y4X3c8U0y4o6g2U0z5b7`.`.)，完全基于Sandboxie v3.40版本调试逆向后重写的工程，完成度90%，可在XP SP3干净环境下运行，被沙箱化的程序的一切行为都限定在沙箱中。

![](https://bbs.kanxue.com/upload/attach/201308/57632_yxqj7037x51un4l.gif)

以下是对Sandboxie原理、整体构架的一段总结：

> 做沙箱不容易，因为要接管的地方太多，文件、注册表重定向只是最基本的，还有**窗口、类名、消息、服务、rpc、Token、COM**、etc。
>
> 在实现过程中需要逆向sandboxie的全部文件：sys、dll、exe，动态调试每个过滤函数的细节。逆向的目的不是抄袭，是为了了解其中的构架，而后再按照自己的思路优化改进重写。以前粗略的总结了sandboxie的原理：
>
> `start.exe`中有些内容较为有趣.其中解析命令行的部分可以不必看,但有些命令的实现还是值得参考,其中的降权&和驱动通信部分有点儿意思.
>
> EXE中复杂的地方是`SbieSvc.exe`的通信部分. 绝大都是RPC,这是沙箱模拟系统通信机制,在自己的小沙盘内部实现了进程间的通信.
>
> 剩下几个exe诸如`SandboxieDcomLaunch.exe` / `SandboxieRpcSs.exe` 内容大同小异.不是关键部分.
>
> 至于`sbieDll.dll` 的文件/注册表/窗口/消息等重定向没什么难度,关键在于细节. 稍微可以借鉴之处在于其对COM部分的处理.
>
> `sbiddrv.sys`完全可以自己重写.
>
> object hook + shadow ssdt hook + image/process notify
>
> object hook 主要处理文件/管道等操作
>
> shadow ssdt hook处理窗口/消息隔离
>
> image notify 负责`sbieDll.dll`注入
>
> process notify负责建立总节点.填充各种信息.
>
> 驱动中的Inline hook 引擎很有趣,解析配置文件`*.ini`部分可以不看.

其实最关键的部分全部在`sbieDll.dll`中，它几乎接管了被沙箱化程序的所有调用，将需要关注的R3层面的各种函数基本都Hook了，进行各种重定向。

`sbieDll.dll` 注入到被沙箱化的进程，用到的方式是驱动`sbiedrv.sys`监控进程创建，在内存中动态感染`IAT`表的方式实现的，x86,x64通用。此时`sbieDll.dll`是除了`ntdll.dll`、`kernel32.dll`（`kernelbase.dll`）之后第三个被加载的模块，此时它便开始接管当前进程的各种调用（这部分注入DLL的代码参考工程中`ProteinBoxDrv`代码）

服务端进程`SbieSvc.exe`用来模拟rpc通信，这个稍微有些复杂，需要对系统进行RPC交互的各种消息有较为深刻的理解。

驱动`sbiddrv.sys`干的事情不多，主要用于控制权限、读取配置文件、限制窗口消息等。

这份代码里面将Sandboxie的exe/dll/sys全部逆向重写了一遍，并包含用IDA 5.0逆向后添加过注释的IDB文件，还有零零散散的调试细节和各种测试工程，风格类似于：

![](https://bbs.kanxue.com/upload/attach/201308/57632_mipc00iw79n3gcr.gif)

![](https://bbs.kanxue.com/upload/attach/201308/57632_e4m3h59w6g8vebc.gif)

最主要的几个工程如下：

![](https://bbs.kanxue.com/upload/attach/201308/57632_nee4e9b8gtj3n84.gif)

[`Sandboxie_Reserved_sudami.7z`](https://bbs.kanxue.com/attach-download-81743.htm) [`沙箱_Proteibox_1.0.0.1001_sudami.7z`](https://bbs.kanxue.com/attach-download-81744.htm)

其他技术实现细节请自行围观代码![](https://bbs.kanxue.com/view/img/face/1.gif)

**解压密码**：sudami

### 镜像下载

请点击下载

![](http://bbs.pediy.com//images/attach/rar.gif)[`Proteibox_1.0.0.1001_sudami.7z`](https://bbs.kanxue.com/elink@483K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0t1I4z5q4)9J5k6e0S2Q4x3X3f1J5y4o6g2Q4x3X3f1J5x3U0W2Q4x3@1p5I4x3U0p5J5i4K6u0r3N6r3!0G2L8s2y4Q4x3V1k6T1j5Y4y4Q4x3V1k6b7M7X3!0@1k6h3W2T1L8%4S2Q4y4h3j5I4i4K6u0W2x3q4)9J5k6e0m8Q4x3X3f1I4x3o6l9I4i4K6g2X3M7%4g2V1j5h3#2A6i4K6u0W2y4%4Z5`.)

![](http://bbs.pediy.com//images/attach/rar.gif)[`Sandboxie_Reserved_sudami.7z`](https://bbs.kanxue.com/elink@b61K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0t1I4z5q4)9J5k6e0S2Q4x3X3f1J5y4o6g2Q4x3X3f1J5x3U0W2Q4x3@1p5I4x3U0p5J5i4K6u0r3N6r3!0G2L8s2y4Q4x3V1k6T1j5Y4y4Q4x3V1k6e0j5h3&6V1j5X3!0^5K9h3g2Q4y4h3k6d9k6i4y4W2M7Y4k6W2k6q4)9#2k6Y4y4#2k6r3q4E0K9g2)9J5k6e0N6*7)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

[#系统底层](https://bbs.kanxue.com/forum-4-1-2.htm)

### 上传的附件：

- [`Snap1.gif`](javascript:void(0);)

  （24.23kb，571次下载）

- [`Snap2.gif`](javascript:void(0);)

  （27.88kb，257次下载）

- [`Snap3.gif`](javascript:void(0);)

  （4.93kb，31次下载）

- [`attachment.gif`](javascript:void(0);)

  （70.99kb，170次下载）

- [`Sandboxie_Reserved_sudami.7z`](javascript:void(0);)

  （3.97MB，2344次下载）

- [`沙箱_Proteibox_1.0.0.1001_sudami.7z`](javascript:void(0);)

  （1.76MB，1982次下载）
