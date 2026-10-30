
![此处输入图片的描述](https://bbs.kanxue.com/upload/attach/202001/697506_Y8BQSYWYUGSMP3X.jpg)

Bandwidth 800 kHz for 250 kbps rate

所有准备都已经就绪，概括一点频道和“无人机地址”不同之处[校对：简要概括一下，需要修改的地方就是信道和无人机的地址]。其他不需要修改...

### 载荷及其格式

![此处输入图片的描述](https://bbs.kanxue.com/upload/attach/202001/697506_S662NPPFMN33TND.jpg)

我们可以通过阅读传输手册来了解传输数据的格式，上图很好地解释了一切。

接收器用一个字节作为前导码来识别传入的数据包，然后跟着的就是地址码（被用作身份标识），载荷是我们实际感兴趣的传输数据，最后是2字节的CRC校验码。

下图所示是一个载荷的例子和其中重要字节的意义：

![此处输入图片的描述](https://bbs.kanxue.com/upload/attach/202001/697506_E43DZRXYB7W7YMN.jpg)

1. 加速引擎
2. 倾向（向前或向后）(  *)
3. 方向舵（转向轴）(*  )
4. 副翼（左或右倾斜）（  *）
5. CRC（前9个字节和`0x55`的异或值）（*）（高位=方向，剩余位=值）

现在拦截脚本已准备就绪。

`droneTelemetry.py`

1. `DecoderSymaX5SW`类：解析数据包的每个字节，返回一个操作。
2. `DisplayDrone`类：通过使用URWID，这个类显示了一个带有无人机动作的图形界面。

实时遥感：）

[youtube.com/embed/OhET0bT48bQ](https://bbs.kanxue.com/elink@57cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2&6L8%4g2@1N6h3u0W2i4K6u0W2j5$3!0E0i4K6u0r3k6h3#2T1k6h3c8Q4x3V1k6a6K9p5g2f1x3r3u0f1y4o6S2T1f1b7`.`.)

## 目标2：发给无人机的传输指令

真正有趣的部分开始了，我们的想法是创造一个能够向无人机发送指令的发射器，就像原来的控制器那样。

因此我们需要如下一些东西：

1. 一个射频传输模块

`NRF24`模块是大多数设备中常用的一种模块，在我们原始控制器的例子中我们使用了原模块的复制品。

这个模块的优势在于其价格（3-5美元）而且其在`Arduino`, `Raspberry Pi` and `BeagleBone`上都可工作。

![此处输入图片的描述](https://bbs.kanxue.com/upload/attach/202001/697506_QCW8HW7TNMQRD3J.jpg)

上面就是运行代码并连接我们发射器的设备。

我们需要特别注意这一点，因为在其他电子或物联网项目中，在`Arduino`、`Raspberry PI`或`Beaglebone`之间进行选择是个人的爱好。

在本例中，延迟因素和传输速度至关重要，我们需要在代码和传输模块之间的延迟尽可能小，这样才能在几毫秒内完成数据包传输，否则无人机会直接掉到地上。

因此我们选择了`Arduino`，相对的我们缺少了一个足够复杂的、稳定的操作系统来判断执行代码时机器的性能。

![此处输入图片的描述](https://bbs.kanxue.com/upload/attach/202001/697506_GEKXAE95S386S5S.jpg)

既然万事俱备，是时候连线了（这并不轻松，因为网上相关的指导少之又少）。

下表很好地说明了模块的连接方法，并通过反复试验取得成功。

![此处输入图片的描述](https://bbs.kanxue.com/upload/attach/202001/697506_DDQRTAUEJYF7YZB.jpg)

还记得上面提到的盒子的四个信道的问题以及FCC的手册吗？（现在看来）完全不是那么回事。在单通道传输中我们可以向无人机发送命令，而不需要在所有四个信道上进行更改和传输。

下面是发射器的部分代码和最重要部分的简要说明：

![此处输入图片的描述](https://bbs.kanxue.com/upload/attach/202001/697506_MHXDQHH2V6D4TNZ.jpg)

就好像是最初的控制器一样，`sender.c`是以`C`语言开发的用于向无人机发送指令的代码文件。在代码层面而言，最重要的是定义模块的`CE`和`CSN`插脚，在本例中是9和10号。下面是对先前找到的地址及我们将要传输到的信道的定义。

![此处输入图片的描述](https://bbs.kanxue.com/upload/attach/202001/697506_UHRPCVVKXJMBK54.jpg)

波特率是我们为了明确`Arduino`的串行连接及能够获取代码的调试信息而定义的。除了一些我们自己的模块配置（如16字节的CRC），包括传输通道和有效载荷大小在内，最大传输功率和“比特率”为250 kbps的。

由于初始无人机和控制器之间缺乏关联协议以及发射机自身的缘故，“劫持”无人机的关键是要拥有一个比无人机自带的发射器具有更大功率和范围的发射器！

**我们做到了这一点！**

## 目标3：摄像机

我们需要一个应用程序，这个应用程序可以连接到摄像机自身创建的WI-FI接入点，同时我们可以用我们的手机查看视频并截图。

如果我们的目的是“劫持”视频连接，那么我们来稍稍看一下`Android`应用程序以便理解一下程序是怎么连接摄像机的...

对这个APK进行反编译我们发现了一些值得注意的事情：

1. 与硬编码IP关联的连接：`192.168.1.1` 即摄像机的IP。
2. 一个带有硬编码证书的URL：`63dK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5&6x3W2)9J5k6e0p5$3z5q4)9J5k6e0q4Q4x3X3f1I4i4K6u0r3M7X3g2I4N6h3g2K6N6q4)9#2k6X3q4$3i4K6u0W2j5$3N6A6i4K6y4r3N6i4y4W2M7W2)9K6c8r3q4V1L8h3W2F1i4K6t1$3j5h3#2H3i4K6y4n7M7s2N6V1i4K6y4p5`
3. 通过对摄像机进行端口扫描我们发现了两个开放式端口：80和2345。

摄像机厂商已经发布了相关说明文档，在文档中我们发现了激活`telnet`服务的方法，给`e8aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5&6x3W2)9J5k6e0p5$3z5q4)9J5k6e0q4Q4x3X3f1I4i4K6u0r3M7$3g2@1i4K6g2X3M7r3q4J5j5h3#2K6i4K6u0W2j5$3N6A6i4K6y4r3N6r3g2D9L8X3g2@1k6q4)9K6c8o6q4Q4x3U0k6S2L8i4m8Q4x3@1u0K6j5i4k6W2i4K6y4p5x3g2)9J5y4X3q4E0M7q4)9K6b7Y4u0W2j5X3!0G2N6q4)9K6c8o6q4Q4c8e0S2Q4b7V1k6Q4z5e0W2Q4c8e0c8Q4b7U0S2Q4b7f1q4Q4c8e0W2Q4z5e0y4Q4b7V1g2Q4c8e0k6Q4z5p5g2Q4b7e0g2Q4c8e0g2Q4z5p5k6Q4z5e0q4Q4c8e0W2Q4z5o6m8Q4z5o6q4Q4c8e0c8Q4b7U0S2Q4z5o6m8Q4c8e0c8Q4b7U0S2Q4b7f1q4Q4c8e0S2Q4b7f1k6Q4b7U0N6Q4c8e0k6Q4b7U0q4Q4z5o6t1`.

如果我们这么做了...

![此处输入图片的描述](https://bbs.kanxue.com/upload/attach/202001/697506_ZKS5SSPTDS4VGHF.jpg)

我们取得了`Root`权限，就是如此简单...

我们继续阅读说明文档然后发现当我们使用`584K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5&6x3W2)9J5k6e0p5$3z5q4)9J5k6e0q4Q4x3X3f1I4i4K6u0r3M7$3&6S2M7s2y4Z5L8%4c8Q4x3X3g2U0k6$3W2Q4x3@1k6#2M7$3g2J5i4K6y4p5j5h3c8E0K9h3&6Q4x3U0k6S2L8i4m8Q4x3@1u0H3N6$3c8Q4x3@1c8Q4c8e0S2Q4b7V1k6Q4z5e0W2Q4c8e0c8Q4b7U0S2Q4b7f1q4g2f1V1I4Q4c8e0k6Q4z5e0N6Q4b7U0k6Q4c8f1k6Q4b7V1y4Q4z5p5y4Q4c8e0k6Q4z5o6S2Q4z5e0q4Q4c8e0c8Q4b7V1u0Q4b7f1y4Q4c8e0c8Q4b7V1c8Q4b7V1k6Q4c8e0N6Q4z5e0c8Q4b7e0S2Q4c8e0g2Q4z5f1y4Q4b7e0S2m8f1p5E0Q4c8e0W2Q4z5o6N6Q4z5p5y4Q4c8e0g2Q4z5p5k6Q4z5e0q4Q4c8e0N6Q4z5p5g2Q4b7U0m8Q4c8e0N6Q4z5f1q4Q4z5o6c8Q4c8e0W2Q4b7V1u0Q4z5e0S2Q4c8e0S2Q4b7f1g2Q4b7e0c8Q4c8e0N6Q4z5f1q4Q4z5o6c8Q4c8e0S2Q4b7f1k6Q4z5o6q4Q4c8e0c8Q4b7U0W2Q4b7e0k6Q4c8e0g2Q4b7U0m8Q4b7U0q4Q4c8e0g2Q4z5p5k6Q4b7f1k6Q4c8e0c8Q4b7V1u0Q4b7e0g2Q4c8e0k6Q4z5p5u0Q4z5p5c8Q4c8e0N6Q4z5o6g2Q4b7e0N6Q4c8e0y4Q4z5o6m8Q4z5o6t1`.

那么视频流呢？

文档也有提及！

1. 首先我们发送一个流ID：`035K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5&6x3W2)9J5k6e0p5$3z5q4)9J5k6e0q4Q4x3X3f1I4i4K6u0r3M7X3g2I4N6h3g2K6N6q4)9#2k6X3q4$3i4K6u0W2j5$3N6A6i4K6y4r3N6i4y4W2M7W2)9K6c8r3q4V1L8h3W2F1i4K6t1$3j5h3#2H3i4K6y4n7M7s2N6V1i4K6y4p5`
2. 然后，我们通过先前所掌握的ID来请求视频流：`48eK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5&6x3W2)9J5k6e0p5$3z5q4)9J5k6e0q4Q4x3X3f1I4i4K6u0r3N6X3W2V1k6h3!0K6N6s2u0W2j5h3#2Q4x3X3g2U0k6$3W2Q4x3@1k6#2M7$3g2J5i4K6y4p5j5h3c8E0K9h3&6Q4x3U0k6S2L8i4m8Q4x3@1u0H3N6$3c8Q4x3@1c8Q4x3U0k6S2L8i4m8Q4x3@1u0K6N6s2u0W2j5h3#2Q4x3@1b7%4x3e0b7#2y4o6j5J5y4U0p5`.

**这样我们就得到了视频！**

结论：

- 我们对无人机的遥控实施了拦截，我们成功地对无人机本身及视频进行了劫持。
- 但在传输一侧我们仍有一些问题。我们用一个电源组来提供传输模块所需的峰值能量，用`Arduino`和`C`语言来改善传输性能。
- 在通信过程中没有安防措施，既没有加密也没有关联的安全协议，发射功率更大的基本上可以控制无人机。

**我们对无人机的遥控实施了拦截，我们成功地对无人机本身及视频进行了劫持！**

我想声明一点，我开始这个项目的时候我对射频和SDR方面一无所知。带着好奇心和坚韧不拔的毅力，再加上有互联网的帮助，你能做很多事。

现在我们要开始下一次征程了。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2019-7-25 17:55 被jasonk龙莲编辑，原因：
