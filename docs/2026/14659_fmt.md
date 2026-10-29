
# [原创]DIY新浪微博Android手机客户端（一）（二）（三）完

作者：地球仪

原文：[74fK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3S2A6i4K6u0W2j5X3q4A6k6s2g2Q4x3X3g2U0L8$3#2Q4x3V1k6S2M7i4c8S2N6r3p5^5x3e0N6Q4x3V1k6T1L8r3!0Y4i4K6u0r3K9i4c8W2L8g2)9J5c8U0M7%4y4U0t1@1y4U0l9J5x3h3j5^5x3h3t1$3k6o6R3%4j5U0R3&6y4o6N6T1j5W2)9J5k6h3S2@1L8h3H3`.](https://bbs.kanxue.com/elink@567K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3S2A6i4K6u0W2j5X3q4A6k6s2g2Q4x3X3g2U0L8$3#2Q4x3V1k6S2M7i4c8S2N6r3p5^5x3e0N6Q4x3V1k6T1L8r3!0Y4i4K6u0r3K9i4c8W2L8g2)9J5c8U0M7%4y4U0t1@1y4U0l9J5x3h3j5^5x3h3t1$3k6o6R3%4j5U0R3&6y4o6N6T1j5W2)9J5k6h3S2@1L8h3H3`.)

玩过新浪微博的人都知道，在每条微博的下方都会显示出该条微博来自于什么样的客户端

比如网页的、手机版的、Android的、iPhone的、iPad的、Symbian的、Kjava的以及很多第三方应用等等

![](https://bbs.kanxue.com/upload/attach/201103/713_fvv5i8uptnoez5p.png)

本次教程的最终目的是“偷龙转凤”，通过修改Android客户端让其发表的微博显示“来自iPhone客户端”

iPhone多火呀！用iPhone多有面子呀！呵呵

第一件事还是准备好所需要的工具。工欲善其事，必先利其器！

## 工具准备

【手机一部】

当然必须是Android系统的

【新浪微博Android客户端】

有了手机当然也要有客户端安装文件了，下载地址：03aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4c8Q4x3X3g2K6K9h3&6S2i4K6u0W2j5$3!0E0i4K6u0W2j5$3&6Q4x3V1k6E0L8$3u0A6L8r3g2Q4x3V1k6S2L8X3c8J5L8$3W2V1i4K6u0W2M7r3S2H3

【JDK】

后面要用到一些Java程序，所以必须安装Java环境

我这里装的1.6版，下载地址：fb8K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3A6S2N6X3q4Q4x3X3g2F1k6i4c8Q4x3V1k6V1L8%4N6F1L8r3!0S2k6q4)9J5c8X3A6V1K9K6k6Q4x3V1j5$3N6e0p5H3i4K6u0r3M7s2u0G2L8h3!0@1k6h3c8Q4x3V1k6T1x3K6u0Q4x3V1k6T1K9h3&6S2M7X3W2W2M7#2)9J5c8X3A6V1K9#2)9J5k6o6k6#2x3e0m8Q4x3X3c8J5j5K6u0Q4x3X3c8T1K9h3&6Q4x3X3c8T1x3K6u0Q4x3X3c8%4K9h3&6V1L8%4N6K6i4K6u0V1K9e0f1^5y4W2)9J5k6s2m8Q4x3X3b7I4x3W2)9#2k6Y4y4W2M7q4)9#2k6U0t1H3x3o6S2Q4x3X3g2W2P5r3f1`.

【ADB】

这个安装了Android的SDK都会有的，或着单独下载提取出来的：b97K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3#2G2j5X3W2D9k6g2)9J5k6i4W2W2M7$3E0&6i4K6u0W2j5$3!0E0i4K6u0r3K9h3#2S2k6$3g2D9K9i4y4@1i4K6u0r3x3U0l9H3z5g2)9J5c8U0x3H3x3g2)9J5c8X3u0&6K9K6R3%4j5U0b7&6M7o6k6Y4M7W2)9J5k6i4u0S2M7R3`.`.

这里面带有USB驱动，可以不用理会

【Tcpdump】

Linux下的网络监视工具，下载：7ccK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4g2#2M7X3!0A6k6q4)9J5k6h3y4G2L8g2)9J5c8Y4g2H3L8r3!0S2k6q4)9J5c8Y4c8U0M7r3c8#2L8i4m8Q4x3X3g2\*7K9i4l9`.

【dex2jar】

Java程序，可以把dex文件反编译成jar文件，下载：5f6K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3I4S2K9h3y4Z5j5h3!0Q4x3X3g2Y4L8$3!0Y4L8r3g2U0L8$3c8W2i4K6u0W2j5$3!0E0i4K6u0r3k6X3W2D9k6i4y4Q4x3V1k6V1k6i4R3J5K9X3q4J5i4K6u0V1x3q4)9J5k6e0m8Q4x3X3f1%4i4K6u0V1f1@1&6m8f1q4y4t1e0#2c8Q4x3X3g2\*7K9i4l9`.

【JD-GUI】

Java反编译程序，可以查看jar文件源码，下载：b74K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3c8#2L8%4c8W2i4K6u0W2j5$3!0E0i4K6u0r3M7$3!0X3N6q4)9J5c8U0M7%4z5e0y4Q4x3X3g2Z5N6r3#2D9

【baksmali和smali】

解包打包java源码的工具，下载：2e5K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3y4G2k6r3g2Q4x3X3g2Y4L8$3!0Y4L8r3g2Q4x3X3g2U0L8$3#2Q4x3V1k6H3i4K6u0r3M7$3#2S2L8r3W2Q4x3V1k6V1L8%4N6F1L8r3!0S2k6s2y4Q4x3V1k6D9K9i4y4@1

【IDA】

强大的反汇编工具，不用解释了吧，我这用的5.5版

【91手机助手】

主要用它来传输文件，当然你用豌豆荚等工具也是可以的

【EFFind】

文本搜索工具，这个不是必须的，只是有它的话能更快地从大量的源码中找到自己想要的数据

## 授权机制

新浪微博开放了API接口，官方网站：456K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4N6W2K9h3u0G2i4K6u0W2j5$3&6Q4x3V1k6%4K9h3E0A6i4K6u0r3K9h3&6V1k6i4S2Q4x3X3g2H3K9s2l9`.

这里只需要简单地看看“授权机制说明”那部分，有两种授权方式

其中的`Basic Authentication`方式需要一个`Appkey`值，而新浪就是通过这个key来判断你是什么客户端

比如“优酷连接分享”的`Appkey`是`2684493555`，这个值在优酷网点击转贴到新浪微博后，在URL地址栏都能看到

![](https://bbs.kanxue.com/upload/attach/201103/713_i3dbykjuw17pbh1.png)

再看浏览器的地址栏

![](https://bbs.kanxue.com/upload/attach/201103/713_gwlouup15dkkgr7.png)

也就是说，我们只要在客户端上修改对应的`Appkey`即可达到欺骗服务器的目的

这也是本教程的最终目的

不过后来分析发现，新浪官方的客户端是没有`Appkey`的，官方的客户端并不访问这个公开的API接口

而是有自己内部的访问接口，当然我们也不必去关心内部接口，因为我们不是要去做客户端

第一篇就暂时到这，做好准备工作，下篇实战

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [q1.png](javascript:void(0);)

  （0.97kb，1096次下载）

- [q2.png](javascript:void(0);)

  （28.35kb，1088次下载）

- [q3.png](javascript:void(0);)

  （1.39kb，1087次下载）
