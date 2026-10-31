
# [原创]Frida加载和启动XServer

## 背景

XServer是我常用的通信协议分析工具，是一个Xposed插件，可能有些同学也在用。

然而Xposed的使用空间受限，不但很多应用对其做了检测，去做绕过有时很麻烦，而且Xposed无法在一些特殊的定制安卓系统上使用。

Frida在这些环境下较为方便，但是在工具开发上有难度，因为：

1. Frida的JS脚本里缺少各类支持的库，想要做个HTTP请求都要自己写。
2. Frida对Java方法的Implementation里应该是native状态，不方便进行异步操作。

当我在使用Frida时，并没有一个现成的既支持实时篡改方法调用，又能很好地处理应用内各类复杂Java对象的工具。

所以，作为一个Xposed的遗民，我决定挣扎一下，用Frida加载和驱动XServer，冒充Frida工具，让这个旧时代的Xposed插件焕发新春。

## 效果及XServer功能介绍

XServer启动，长得完全就像是个Frida工具

![图片描述](upload/attach/202007/776971\_RSND7TQ3CAVD4JK.png)

使用XServer查看应用已加载的类

![图片描述](upload/attach/202007/776971\_RZQHUA28SGGD8YS.png)

在图形化界面Hook特定的方法~~（好了我知道界面很丑）~~

![图片描述](upload/attach/202007/776971\_PA2A4NAPF3FXBWC.png)

跟踪应用内的方法调用

![图片描述](upload/attach/202007/776971\_8VSAPUBW6YWW2GT.png)

点进去还能跟踪调用的堆栈信息

![图片描述](upload/attach/202007/776971\_6788Y4E86Q9JYR6.png)

并通过Burp动态拦截应用的方法调用，并实时篡改参数或结果

![图片描述](upload/attach/202007/776971\_9B594J8KMVWR7FW.png)

![图片描述](upload/attach/202007/776971\_TQMKVFRUNJZS9EY.png)

并且可以在Burp里重放请求，从而直接调用对应方法（用于通信协议或加解密）

![图片描述](upload/attach/202007/776971\_36NY5XFM9H69ARW.png)

再就是可以直接读取/修改/Dump内存，这个功能还很简陋

![图片描述](upload/attach/202007/776971\_SEWDXTNZPKV3GBZ.png)

以上，XServer的功能全部都能用Frida正常实现。

## 原理

XServer本身是一个Xposed插件，但其实依赖Xposed的地方只是注入目标应用和Hook，大部分功能其实都是Java层面的奇技淫巧。

所以我对Hook部分进行了封装，使其既可以被Xposed实现（默认），也可以被其他Hook框架实现（比如Frida），从而使一个Xposed插件可以脱离Xposed Framework工作。

然后我用Frida实现了XServer的注入和Hook功能。然后，就正常工作了。这部分实现可以在[XServer.js](elink@971K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6E0L8$3&6C8k6i4W2D9L8%4u0V1i4K6u0r3h3q4y4W2M7Y4k6W2M7W2)9J5c8X3u0D9L8$3u0Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3V1k6j5f1$3g2J5N6X3g2J5i4K6u0W2K9Y4x3`.)中看到，使用了一些技巧。

（实际上，我还用Frida实现了Xposed API，从而可以加载一般的Xposed插件，但还不稳定，所以不拿出来了）

## 总结

使用Frida的时候，比起JS+Python，JS+Java也有它的优势，比如说Java可以很容易在安卓应用内部玩花活，这对熟悉安卓的人十分友好。

用Java编写工具的逻辑，然后用Frida去实现它的底层功能，至少，我这个Xposed遗民觉得，挺香。

XServer：9c1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6E0L8$3&6C8k6i4W2D9L8%4u0V1i4K6u0r3h3q4y4W2M7Y4k6W2M7R3`.

（另附操作视频，见附件）

## 背景

XServer是我常用的通信协议分析工具，是一个Xposed插件，可能有些同学也在用。

然而Xposed的使用空间受限，不但很多应用对其做了检测，去做绕过有时很麻烦，而且Xposed无法在一些特殊的定制安卓系统上使用。

Frida在这些环境下较为方便，但是在工具开发上有难度，因为：

1. Frida的JS脚本里缺少各类支持的库，想要做个HTTP请求都要自己写。
2. Frida对Java方法的Implementation里应该是native状态，不方便进行异步操作。

当我在使用Frida时，并没有一个现成的既支持实时篡改方法调用，又能很好地处理应用内各类复杂Java对象的工具。

所以，作为一个Xposed的遗民，我决定挣扎一下，用Frida加载和驱动XServer，冒充Frida工具，让这个旧时代的Xposed插件焕发新春。

## 效果及XServer功能介绍

XServer启动，长得完全就像是个Frida工具

 ![图片描述](https://bbs.kanxue.com/upload/attach/202007/776971_RSND7TQ3CAVD4JK.png)

使用XServer查看应用已加载的类

![图片描述](https://bbs.kanxue.com/upload/attach/202007/776971_RZQHUA28SGGD8YS.png)

在图形化界面Hook特定的方法

 ![图片描述](https://bbs.kanxue.com/upload/attach/202007/776971_PA2A4NAPF3FXBWC.png)

跟踪应用内的方法调用

![图片描述](https://bbs.kanxue.com/upload/attach/202007/776971_8VSAPUBW6YWW2GT.png)

点进去还能跟踪调用的堆栈信息

![图片描述](https://bbs.kanxue.com/upload/attach/202007/776971_6788Y4E86Q9JYR6.png)

并通过Burp动态拦截应用的方法调用，并实时篡改参数或结果

![图片描述](https://bbs.kanxue.com/upload/attach/202007/776971_9B594J8KMVWR7FW.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202007/776971_TQMKVFRUNJZS9EY.png)

并且可以在Burp里重放请求，从而直接调用对应方法（用于通信协议或加解密）

 ![图片描述](https://bbs.kanxue.com/upload/attach/202007/776971_36NY5XFM9H69ARW.png)

再就是可以直接读取/修改/Dump内存，这个功能还很简陋

 ![图片描述](https://bbs.kanxue.com/upload/attach/202007/776971_SEWDXTNZPKV3GBZ.png)

以上，XServer的功能全部都能用Frida正常实现。

## 原理

XServer本身是一个Xposed插件，但其实依赖Xposed的地方只是注入目标应用和Hook，大部分功能其实都是Java层面的奇技淫巧。

所以我对Hook部分进行了封装，使其既可以被Xposed实现（默认），也可以被其他Hook框架实现（比如Frida），从而使一个Xposed插件可以脱离Xposed Framework工作。

然后我用Frida实现了XServer的注入和Hook功能。然后，就正常工作了。这部分实现可以在[XServer.js](https://bbs.kanxue.com/elink@37fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6E0L8$3&6C8k6i4W2D9L8%4u0V1i4K6u0r3h3q4y4W2M7Y4k6W2M7W2)9J5c8X3u0D9L8$3u0Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3V1k6j5f1$3g2J5N6X3g2J5i4K6u0W2K9Y4x3`.)中看到，使用了一些技巧。

（实际上，我还用Frida实现了Xposed API，从而可以加载一般的Xposed插件，但还不稳定，所以不拿出来了）

## 总结

使用Frida的时候，比起JS+Python，JS+Java也有它的优势，比如说Java可以很容易在安卓应用内部玩花活，这对熟悉安卓的人十分友好。

用Java编写工具的逻辑，然后用Frida去实现它的底层功能，至少，我这个Xposed遗民觉得，挺香。

XServer：751K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6E0L8$3&6C8k6i4W2D9L8%4u0V1i4K6u0r3h3q4y4W2M7Y4k6W2M7R3`.

（另附操作视频，见附件）

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于  2020-7-7 21:55

被Monkeylord编辑

，原因： 补充附件

上传的附件：

- [XServer V0.7.1.apk](javascript:void(0);)（2.52MB，171次下载）
- [XServer.js](javascript:void(0);)（12.56kb，138次下载）
- [使用Frida加载XServer.mp4](javascript:void(0);)（6.01MB，217次下载）
