
完成上述操作之后，就可以把被patch的`davinci`上传至海康萤石的智能网关，此处我们不再采用烧录固件的方法，因为过于麻烦。通过翻阅该设备的文件系统，可以在设备中发现`tftp`程序，这是一个使用`tftp`协议传输文件的程序，可以用它从`tftpd`服务器上传或下载文件。我们使用从这个链接下载到的`tftpd`程序：374K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4c8X3N6s2m8V1x3K6u0Q4x3X3g2B7L8%4g2F1K9h3&6Q4x3X3g2F1k6i4c8Q4x3V1k6@1k6Y4c8H3k6o6j5@1i4K6g2X3k6r3!0%4L8X3I4G2j5h3c8Q4x3X3g2Z5N6r3#2D9i4@1g2r3i4@1u0o6i4K6S2o6i4@1f1^5i4@1u0r3i4K6V1H3i4@1f1^5i4@1p5I4i4K6S2o6i4@1f1#2i4K6V1H3i4K6S2q4i4@1g2r3i4@1u0o6i4K6S2o6i4@1f1^5i4@1t1H3i4K6R3K6i4@1f1$3i4K6V1#2i4@1t1@1N6r3k6@1M7r3c8Q4c8e0k6Q4z5f1y4Q4z5p5c8Q4c8e0g2Q4z5p5q4Q4b7e0q4Q4c8e0g2Q4z5e0W2Q4b7e0S2Q4c8e0N6Q4z5f1q4Q4z5o6c8Q4c8e0k6Q4z5e0k6Q4z5o6N6Q4c8e0c8Q4b7V1u0Q4b7U0k6Q4c8e0N6Q4z5f1u0Q4b7f1g2Q4c8e0g2Q4b7V1c8Q4z5e0g2Q4c8e0g2Q4z5e0u0Q4z5p5y4Q4c8e0N6Q4z5f1u0Q4z5e0q4Q4c8e0g2Q4z5e0m8Q4b7f1y4Q4c8e0N6Q4b7V1c8Q4z5e0q4Q4c8e0g2Q4z5p5c8Q4b7e0q4Q4c8e0y4Q4z5o6m8Q4z5o6u0Q4c8e0N6Q4z5o6c8Q4b7U0k6Q4c8e0g2Q4z5e0m8Q4z5p5g2Q4c8e0g2Q4z5f1y4Q4b7e0S2Q4c8e0k6Q4z5e0W2Q4b7V1q4Q4c8e0S2Q4z5o6y4Q4b7V1c8Q4c8e0N6Q4b7V1c8Q4z5e0q4Q4c8e0g2Q4z5o6g2Q4b7U0y4Q4c8e0S2Q4b7f1g2Q4b7V1g2Q4c8e0g2Q4b7e0c8Q4z5o6N6Q4c8e0c8Q4b7U0S2Q4b7f1c8Q4c8e0c8Q4b7V1c8Q4b7V1k6Q4c8e0N6Q4z5e0c8Q4b7e0S2@1k6Y4c8H3i4@1f1%4i4@1p5^5i4K6S2n7i4@1f1#2i4@1u0m8i4K6S2r3i4@1f1^5i4K6S2q4i4@1t1%4i4@1f1#2i4K6S2r3i4K6V1$3i4@1f1%4i4K6W2n7i4@1t1^5i4@1f1#2i4K6R3#2i4@1t1K6i4@1f1$3i4K6V1$3i4K6R3%4i4@1f1@1i4@1u0n7i4@1t1$3i4@1f1#2i4K6S2p5i4@1t1K6i4@1f1#2i4K6S2r3i4@1q4r3i4@1g2r3i4@1u0o6i4K6S2o6i4@1f1%4i4K6W2n7i4@1t1^5i4@1f1#2i4K6R3#2i4@1t1K6i4@1f1#2i4K6V1I4i4@1u0p5i4@1f1@1i4@1u0n7i4@1p5@1i4@1f1$3i4K6R3^5i4@1q4m8i4@1f1#2i4K6W2n7i4@1u0q4i4@1f1#2i4@1p5$3i4K6R3J5i4@1f1@1i4@1t1^5i4K6S2n7i4@1g2r3i4@1u0o6i4K6W2m8

![](https://bbs.kanxue.com/upload/attach/202008/813243_YKEPSX59XWBU78J.jpg)

图5-4 使用`tftp`下载文件

待相关文件传输到萤石网关中之后，就可以运行`FeedWatchdog.sh`脚本开始喂狗，然后终止原本的`davinci`进程，接着删除`/home/pidfile`文件，该文件相当于互斥体，用于控制`davinci`仅运行了一次，最后启动我们自己的`davinci_1`进程，相关命令如下：

![](https://bbs.kanxue.com/upload/attach/202008/813243_2YM9VJCDE4YCBHM.jpg)

图5-5 启动patch之后的`davinci`进程

待程序运行一段时间之后，就可以查看程序的运行日志，然后通过日志分析程序的各种行为。

## 5.2 日志分析

待程序运行一段时间之后，就会在`/applog/devlog`目录下生成日志文件，每个文件500KB左右，如下图所示：

![](https://bbs.kanxue.com/upload/attach/202008/813243_YHJC99X4DEQAXSK.jpg)

图5-6 程序运行时生成的日志

用`tftp`程序将日志取回，然后打开程序日志，在日志中搜索`litedev.ys7.com`，这个地址是智能网关上电之后第一个访问的地址，我们第二章中介绍过。搜索结果如下图：

![](https://bbs.kanxue.com/upload/attach/202008/813243_5ND8ZWGN6DYTQ8G.jpg)

图5-7 与`litedev.ys7.com`有关的日志

上图中，我们可以看到由`litedev.ys7.com`解析而来的ip地址：115.231.107.14，这与我们用`wireshark`抓包时得到的结果是相同的。

继续翻看日志，在距离上图不远的地方，可以看到另一条日志，看起来像是与海康萤石MQTT服务器相关的日志内容，截图如下：

![](https://bbs.kanxue.com/upload/attach/202008/813243_2ZUZWEJ5JJ957XU.jpg)

图5-8 与MQTT服务器有关的日志

从图中，我们可以看到另一个ip地址：101.71.30.172，该地址同样与我们`wireshark`抓包的结果吻合。MQTT通信协议是构建于TCP/IP协议之上的一种轻量级通信协议，经常出现在IoT设备系统中设备端与云端的通信过程中。

结合这两条日志内容和`wireshark`的抓包结果，我们可以进一步确认设备的工作流程：与`litedev.ys7.com`通信，获取了MQTT服务器的地址；然后与MQTT服务器通信，实现设备的逻辑功能。

在图5-7和图5-8中，我们分别用红框标识了一个关键字符串。在海康萤石智能网关的文件系统中，二进制搜索其中一个字符串“`lbs_connect`”，确认该字符串出现在`libmicrokernel.so.1`中。用IDA加载该so文件，可以找到与`lbs_connect`字符串有关系的函数，查找该函数的交叉引用，可以看到有多处代码调用了此函数，我们随便找一处点开看下，如下图所示：

![](https://bbs.kanxue.com/upload/attach/202008/813243_S4X8BRAHUFHJFKM.jpg)

图5-9 调用`lbs_connect`函数

可以看到在`lbs_connect`函数返回成功之后，就会调用`send_authentication_i`函数。这个函数看起来就是加密和认证相关的函数，接下来我们通过动态调试的方式分析网关加密和认证的流程。

## 5.3 动态调试的准备

对于嵌入式Linux操作系统，我们通常选用`gdb`和`gdbserver`作为调试工具。我们可以直接在设备上使用`gdb`进行本地调试，但`gdb`程序体积比较大，而且直接在设备上运行`gdb`会有很多不方便的地方，所以我们选择通过`gdbserver`进行远程调试。为了完成调试工作，我们首先需要一个可以运行在海康萤石智能网关设备上的`gdbserver`程序。

在4.3节中，我们使用MT7688 SDK中的工具对固件进行了重打包，所以当我们需要与芯片配套的`gdbserver`时，也是去MTK官方查阅资料。事实上，MT7688开发板的固件包中确实内置了一个`gdbserver`，但是当我们提取出这个`gdbserver`，并通过`tftp`传输到网关上开始运行后，发现并没有任何输出，可能是MTK官方定制了`gdbserver`的代码，导致我们无法使用该程序调试。

那么，就只能去找一找有没有其他可以正常运行在海康萤石智能网关中的`gdbserver`程序了。经过一番搜索后，在rapid7官方github账户上发现了已经编译好的`gdbserver`程序，链接如下：2eeK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6J5j5i4m8A6k6o6N6Q4x3V1k6W2L8h3u0W2k6r3c8W2k6q4)9J5k6s2c8G2L8$3I4K6i4K6u0r3N6s2u0W2k6g2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8X3u0A6L8X3q4J5K9h3g2K6i4K6u0r3k6$3c8T1M7$3g2J5N6X3g2J5i4@1g2r3i4@1u0o6i4K6S2o6 这里我们选择下载芯片对应的`mipsle`版本的`gdbserver`。

这里需要说明一下，rapid7提供的`gdbserver`，可能在某些设备上也无法正常使用，所以最靠谱的方法是使用SDK自己编译一个出来，但是这里既然有可用的，我们就不（懒）再（得）去费功夫了。

我们通过`tftp`将`gdbserver`传输到网关中，并使用`gdbserver`启动`davinci`。如果使用附加方式调试，很可能会错过`davinci`程序与服务器通信的认证过程。所以我们选择使用`gdbserver`启动程序，这样一来，在远程调试器连接之前，`davinci`程序将处于挂起状态。命令如下：

![](https://bbs.kanxue.com/upload/attach/202008/813243_BWMXTKMRNXETX5N.jpg)

图5-10 通过`gdbserver`启动程序

此时，`gdbserver`就开始监听23946端口，等待远程调试器的连接。

接下来，我们可以选择`gdb`或IDA作为远程调试器连接`gdbserver`。IDA提供图形界面，可以帮助我们理解程序的逻辑；但`gdb`要更稳定，可以有效避免IDA调试时出现的奇怪错误。我们这里直接选择`gdb`作为远程调试器，在后续的文章中会有介绍用IDA作为调试器的例子。

由于我们需要调试的是MIPS指令集的程序，而`gdb`默认情况下，仅支持调试与当前环境采用相同指令集的程序（i386），所以我们需要安装可以调试MIPS指令集的`gdb`程序。安装方法比较简单，直接输入`sudo apt install gdb-multiarch`即可。还可以给`gdb`程序加一个`pwndbg`插件，用于辅助我们的调试工作，该插件的官方地址是：bd4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6H3N6$3&6V1j5X3N6Q4x3V1k6H3N6$3&6V1j5X3N6Q4c8f1k6Q4b7V1y4Q4z5p5y4Q4c8e0g2Q4z5p5k6Q4b7f1q4Q4c8e0W2Q4z5f1y4Q4z5o6m8Q4c8e0S2Q4b7e0k6Q4z5o6q4Q4c8e0c8Q4b7U0S2Q4z5p5u0Q4c8e0S2Q4b7V1c8Q4b7V1c8Q4c8e0c8Q4b7U0S2Q4z5p5u0Q4c8e0k6Q4z5f1c8Q4b7e0g2Q4c8f1k6Q4b7V1y4Q4z5p5y4Q4c8e0N6Q4z5o6c8Q4b7U0k6Q4c8e0g2Q4z5e0m8Q4z5p5g2Q4c8e0S2Q4b7V1k6Q4z5e0m8Q4c8e0S2Q4b7e0q4Q4z5p5y4Q4x3X3g2Q4x3V1k6K6k6i4c8#2M7q4)9J5k6i4y4Z5i4@1f1#2i4K6S2p5i4@1t1K6i4@1f1#2i4K6S2r3i4@1q4r3i4@1f1K6i4K6R3H3i4K6R3J5i4@1f1$3i4@1q4p5i4@1p5@1i4@1f1$3i4K6S2r3i4K6V1J5i4@1f1@1i4@1u0n7i4@1t1$3i4@1f1#2i4@1t1&6i4@1t1$3i4@1f1&6i4K6W2p5i4K6W2q4i4@1f1#2i4@1u0r3i4K6R3#2i4@1f1&6i4K6W2o6i4K6R3H3i4@1f1#2i4K6V1K6i4K6R3I4i4@1g2r3i4@1u0o6i4K6S2o6i4@1f1@1i4@1u0p5i4K6R3$3i4@1f1$3i4K6V1^5i4@1q4r3i4@1f1$3i4K6S2q4i4@1p5^5i4@1f1^5i4K6S2p5i4K6V1H3i4@1f1^5i4@1p5K6i4K6R3#2i4@1f1@1i4@1t1^5i4K6S2m8i4@1f1K6i4K6R3H3i4K6R3J5

完成`gdb`的配置工作之后，就可以使用`gdb`连接`gdbserver`开始远程调试了。`gdb`的调试命令非常多，可以直接搜索到很多整理好的常用命令，在这里，我们就遇见什么指令就解释什么指令吧。运行`gdb-multiarch`，截图如下：

![](https://bbs.kanxue.com/upload/attach/202008/813243_PYRTKFP4CF43PBK.jpg)

图5-11 运行`gdb-multiarch`程序

上图中，我们分别设置`architecture`为MIPS，读取`davinci`的符号文件，在`main`函数设置断点，并连接远程的`gdbserver`，关键位置已用绿框圈出。然后，我们用快捷键`c`（`continue`），让程序开始执行，过一会程序就会在`main`函数的入口处被断下来。

此时，参考图5-11中的`file`命令加载`libmicrokernel.so.1`的符号文件，然后就可以直接使用函数名`lbs_connect`下断点了，如下图：

![](https://bbs.kanxue.com/upload/attach/202008/813243_9KTFB8PTDSABB82.jpg)

图5-12 在`lbs_connect`函数下断点

稍等片刻，就会看到程序断在`lbs_connect`函数中，等待我们的调试命令，截图如下：

![](https://bbs.kanxue.com/upload/attach/202008/813243_4AYA2FRQ6K67Q5G.jpg)

图5-13 程序命中`lbs_connect`断点

图5-13最下方，打印出了函数调用栈，可以看到，程序是从`lbs_redirect`调用过来的，我们可以继续在外层函数下断点调试，以帮助我们理解程序的逻辑。

## 5.4 加密和认证流程分析

至此，我们已经具备了调试能力，可以调试固件中的`davinci`程序以及`libmicrokernel.so.1`等动态连接库。为了节约篇幅，我们先将一些分析结论与各位分享，然后再详细调试其中的某个细节。

在海康萤石的智能网关中，完整的通信流程涉及到3个密钥，分别是`share key`、`master key`以及`session key`。其中，`share key`是智能网关与`litedev`服务器（与之对应的另一个服务器是MQTT服务器）共有的密钥，由设备序列号等常量经过多次MD5运算得到，每个智能网关设备的`share key`都是独一无二的；`master key`是智能网关与`litedev`服务器经过密钥协商而来，用于加密传输`session key`，关于`master key`的密钥协商过程就是我们将要分析的重点内容；最后`session key`是智能网关与MQTT服务器通信时使用的加密密钥，该密钥由`litedev`服务器直接下发给智能网关。

以上三个密钥在使用过程中，`share key`是与设备绑定，且固定不变的，一旦获取了`share key`，就可以完成所有的认证过程，最终被海康萤石认定为合法的网关设备；使用者通过手机app绑定智能网关时，会经过密钥协商生成`master key`，每绑定一次会就更新一次；最后的`session key`则每次运行`davinci`程序时，都会更新，这也意味着每次重启网关都会更新`session key`。三个不同级别的密钥，三个不同的密钥生存周期。

在对海康萤石智能网关的密钥体系有所了解之后，我们就可以聚焦到某些细节上，下面来着重看一下`master key`和`share key`的生成过程。

### 5.4.1 master key生成流程

为了触发智能网关的密钥协商流程，我们需要先删除当前的`master key`，具体讲就是删除“/cfg/dev_masterkey”文件，如下图所示：

![](https://bbs.kanxue.com/upload/attach/202008/813243_AKAK4SHAZVBDMMS.jpg)

图5-14 删除`dev_masterkey`文件

删除当前`master key`之后，按照前一节所述的方法通过`gdbserver`启动`davinci`，即可开始调试了。

既然我们希望弄清楚加密和认证的流程，那么这个流程中网关和服务器通信数据的处理是我们要重点关注的。我们首先静态分析一下5.2节结尾提到的`send_authentication_i`函数，它调用的`common_serialize`和`authentication_i_serialize`用于构造将要发送的数据。我们就在`common_serialize`函数之前下断点，然后用`gdb`命令查看待发送数据的内容（本次调试时，地址为`0x007d2dd0`），如下图所示：

![](https://bbs.kanxue.com/upload/attach/202008/813243_FBHAJVHAWPBVB7S.jpg)

图5-15 第一次打印发送数据

在上图中，可以看到待发送数据内容还是空的。待函数`common_serialize`执行完毕之后，重新打印该地址的数据，如下图所示：

![](https://bbs.kanxue.com/upload/attach/202008/813243_PC6C6ECJ2BBJC46.jpg)

图5-16 第二次打印发送数据

该地址的前几个字节已经被赋值了。继续调试该程序，在`authentication_i_serialize`函数之后下断点，待程序中断时，重新打印该内存处的数据，如下图所示：
