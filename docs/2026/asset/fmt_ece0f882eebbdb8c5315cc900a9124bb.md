
## 既然我们在Android7.0以下是信任了我们的用户证书，如果我们可以重打包我们的应用，将应用的目标版本强制修改为6.0或者更加低，我们就可以实现信任用户证书来抓包。

具体修改方法：

比如我们的`platformBuildVersionCode`指定为`23`，版本为`Android 6.0.0`，然后重新打包安装。

当然对抗的方法显然第一种比较容易实现，思路多点也是可以应对不同环境。

### 3、SSL pinning 单边校验

单边校验的意思就是只校验了一端，通常情况下都是客户端去校验服务端。实际上第二种方法就是单边校验。只是我们校验的是系统证书。

那为啥又要引入SSL pinning技术呢？实际上就是为了解决被中间人攻击。

SSL pinning技术又分为两种：锁定证书和锁定公钥。

如果对该技术不太熟悉的可以参考下面两篇文章：

1、<mark class="encrypted">0afK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2E0k6s2m8A6i4K6u0W2j5$3!0E0i4K6u0r3x3e0l9&6z5g2)9J5k6o6b7K6x3o6m8Q4x3V1j5J5x3g2)9J5c8U0p5J5i4K6u0r3x3e0p5K6y4W2)9J5c8X3S2@1L8g2)9J5x3@1t1I4x3#2)9J5k6r3g2F1N6s2u0G2M7s2W2Q4x3X3b7J5x3g2)9J5k6o6l9I4x3e0x3$3</mark>

2、<mark class="encrypted">26eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6B7N6h3g2B7K9h3&6Q4x3X3g2U0L8W2)9J5c8Y4m8G2M7%4c8Q4x3V1j5$3z5o6b7@1z5e0l9K6z5o6l9&6x3o6j5^5y4e0j5@1y4o6V1K6</mark>

![图片描述](upload/attach/202104/849527_2BFTMT4HYC6SZC4.png)

看上面的这张图片我们也可以大致的理解单边校验的概念。

如果没有加SSL pinning技术，那我们依旧可以中间人。相关的图例在第二篇的参考链接中有，同样在第一小节中也有图例，这里就不再重复。

那我们来理解一下SSL pinning技术是怎样避免中间人攻击的。

#### 证书锁定

既然是要锁定证书，那么我们客户端上应该事先存在一个证书，我们才能锁定这个证书来验证我们真正的服务端，而不是代理工具伪造的服务端。

如果是锁定证书，那通常情况下会将证书放置在`app/asset`目录下

具体操作：将APP代码内置仅接受指定域名的证书，而不接受操作系统或者浏览器内置的CA根证书对应的任何证书。通过这种授权方式，保障了APP与服务端通信的唯一性和安全性，因此移动端APP与服务端（例如API网关）之间的通信可以保证绝对的安全。

#### 公钥锁定

具体做法：公钥锁定是提前证书中的公钥并内置到移动端APP内，通过与服务器对比公钥值来验证连接的合法性。

如果是这样的话，我们想要抓包，那么就的突破锁定。

### 对抗方法

对于这种SSL pinning的单边校验，想突破，那我们就需要让这个SSL pinning形同虚设，这样我们就可以抓包。如何实现呢？

1、使用`xposed`工具`JustTrustMe`

2、使用`frida` 比如：<mark class="encrypted">d6dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6n7K9h3N6r3j5h3y4W2b7$3q4@1x3U0l9I4y4#2)9J5c8X3k6J5K9h3c8S2i4K6g2X3M7%4y4D9i4K6g2X3L8r3!0Y4k6$3g2J5</mark>

3、反编译修改逻辑

为了不重复造轮子，这里直接推荐一篇文章

<mark class="encrypted">ab7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6^5P5W2)9J5k6h3q4D9K9i4W2#2L8W2)9J5k6h3y4G2L8g2)9J5c8Y4c8Q4x3V1j5^5x3o6b7%4</mark>

### 4、SSL pinning 双边校验

顾名思义，双边校验就是客户端校验服务端，服务端校验客户端。已经被逼迫到这个地步。我们还是用一张图来看一下双边校验。

![图片描述](upload/attach/202104/849527_7UWVY8T7K9XWDM2.png)

（引用于：<mark class="encrypted">697K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6B7N6h3g2B7K9h3&6Q4x3X3g2U0L8W2)9J5c8Y4m8G2M7%4c8Q4x3V1j5$3z5o6b7@1z5e0l9K6z5o6l9&6x3o6j5^5y4e0j5@1y4o6V1K6i4@1g2r3i4@1u0o6i4K6R3&6</mark>

既然如此，看完上面这张图，可以明白双边校验是在单边校验的基础上进行的，那我们想要突破双边校验就需要先把单边校验给绕过，绕过后我们才能来继续绕过服务端校验客户端的环节。

如果我们能够找到客户端的证书，然后导入到我们的抓包工具伪造的客户端中，是不是就可以绕过我们的校验呢？确实是这样。因为这个校验的代码并不在我们的手上，我们也没办法修改。只能欺骗服务端。

#### 对抗方法

找到证书，导入抓包工具，使用上一种方法绕过单边校验。

具体的实现方法：

发现网上有很多优秀的师傅提供的实战的案例，参考如下：

1、<mark class="encrypted">cbeK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6^5P5W2)9J5k6h3q4D9K9i4W2#2L8W2)9J5k6h3y4G2L8g2)9J5c8Y4c8Q4x3V1j5^5x3o6b7%4i4K6t1K6N6r3!0U0i4K6u0V1y4H3`.`.</mark>

2、<mark class="encrypted">173K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4q4I4i4K6g2X3x3K6R3K6x3e0j5$3y4e0g2Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8U0p5H3y4o6p5%4y4U0R3^5x3R3`.`.</mark>

3、<mark class="encrypted">275K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6^5P5W2)9J5k6h3q4D9K9i4W2#2L8W2)9J5k6h3y4G2L8g2)9J5c8Y4c8Q4x3V1j5$3y4e0f1I4</mark>

上面三篇文章基本上用的都是同一个案例

4、https://bbs.pediy.com/thread-265404.htm

当然还会遇到一种情况就是本地APK中并没有发现证书的情况，处理方法大致有两个途径。

1、使用`r0ysue`写的通杀脚本（<mark class="encrypted">ae7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6J5x3s2W2K6N6h3g2Q4x3V1k6J5x3r3y4S2M7s2c8#2M7X3g2Q4x3U0V1`.</mark>

2、自己逆相关逻辑，寻找到私钥文件

### 5、代理检测

目前的对抗已经进入双向认证的环节了，但是依然可以绕过，那接下来我们还能怎样进行下去呢？对抗永不停止。可以看出，上面几乎所有的对抗办法都是基于代理来完成的，如果没有代理的情况下，那我们中间人攻击就相对来讲更加难以实现，如果我们在对绕过的工具进行专门的对抗，相对情况也是会增加难度的，（<mark class="encrypted">361K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2E0k6s2m8A6i4K6u0W2j5$3!0E0i4K6u0r3x3e0l9&6z5g2)9J5k6o6b7K6x3o6m8Q4x3V1j5J5x3g2)9J5c8U0p5J5i4K6u0r3x3e0p5K6y4W2)9J5c8X3S2@1L8g2)9J5x3@1t1I4x3#2)9J5k6r3g2F1N6s2u0G2M7s2W2Q4x3X3b7J5x3g2)9J5k6o6l9I4x3e0x3$3i4@1g2r3i4@1u0o6i4K6R3&6i4@1f1#2i4K6W2o6i4@1p5^5i4@1f1^5i4@1u0r3i4K6V1&6i4@1f1&6i4K6R3%4i4K6S2o6i4@1f1&6i4K6W2p5i4@1p5J5i4@1f1@1i4@1t1&6i4K6W2r3i4@1f1$3i4K6S2r3i4K6V1H3i4@1f1#2i4K6R3^5i4@1t1H3i4@1f1@1i4@1t1^5i4K6R3H3i4@1f1@1i4@1u0m8i4K6W2n7i4@1f1#2i4@1p5@1i4K6R3@1i4@1f1%4i4K6V1H3i4K6R3$3i4@1f1$3i4K6V1$3i4@1t1&6i4@1f1$3i4@1t1K6i4K6V1#2i4@1g2r3i4@1u0o6i4K6S2o6i4@1f1$3i4@1q4r3i4K6V1@1i4@1f1#2i4@1p5$3i4K6R3J5i4@1g2r3i4@1u0o6i4K6W2m8i4@1f1@1i4@1u0n7i4@1p5K6i4@1f1%4i4@1p5H3i4K6R3I4i4@1f1$3i4@1t1%4i4@1t1%4i4@1f1$3i4@1t1%4i4K6R3$3i4@1f1K6i4K6R3H3i4K6R3I4i4@1f1#2i4K6S2m8i4@1p5H3i4@1f1#2i4K6W2n7i4@1u0m8i4@1f1K6i4K6R3H3i4K6R3I4i4@1f1#2i4K6S2r3i4K6S2p5i4@1f1&6i4K6R3%4i4K6S2p5i4@1f1$3i4K6R3&6i4K6V1K6i4@1f1#2i4K6S2o6i4K6R3#2i4@1f1@1i4@1t1&6i4K6S2n7i4@1f1%4i4@1t1I4i4@1u0n7i4@1f1%4i4K6W2m8i4K6R3@1i4@1f1#2i4@1q4q4i4K6W2q4i4@1f1%4i4K6S2q4i4@1t1H3i4@1f1$3i4K6V1$3i4@1t1&6i4@1f1$3i4@1t1K6i4K6V1#2i4@1f1K6i4K6R3H3i4K6R3J5</mark>

如何实现代理检测呢？又如何绕过代理检测呢？这是一场持续的对抗。

比如在`okhttp`中可以设置无代理模式，不走系统代理

![图片描述](upload/attach/202104/849527_D7RYZ6EW9WU7DR2.png)

比如检测我们是否使用了`http.proxyHost`之类的函数

![图片描述](upload/attach/202104/849527_JC3FSGAG4F9K323.png)

实现的方式多种多样，没有比较固定的方式。

#### 对抗方法

1、hook相关函数

2、反编译修改代码逻辑

3、可以使用`iptables`对请求进行强制转发，`ProxyDroid`全局代理工具就是通过`iptables`实现的，所以使用`ProxyDroid`开启代理，可以比较有效的绕过代理检测。

## 后记

1、首选感谢前辈们的分析，使得更加容易理解

2、尽管提到了这么多校验，但是我相信对抗永不止步，肯定还含有很多针对独特环境的。

3、本人能力确实有限，所能想到的策略也很局限，希望各位大佬们不吝赐教，一起补充。

## 前言

首选抓包工具是非常多的，抓包的手段也是非常多，在本文中主要是结合实际的使用场景和工作需要，选择较为常见的抓包方式和工具。同时，现在开发者的安全意识也是在逐步加强，对流量的保护上也是在下苦功夫，因此针对不同的保护选择不同的绕过方式。本文大致分为以下两个部分：

1、抓包工具的使用和环境的搭建

2、常见的绕过解决疑难杂症

## 抓包工具使用和环境搭建

参考：

1、915K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6^5P5W2)9J5k6h3q4D9K9i4W2#2L8W2)9J5k6h3y4G2L8g2)9J5c8Y4c8Q4x3V1j5$3y4e0f1I4

就上面的参考文章中已经提到很多工具抓包，我这里就直接重复讲解，直接进入正题，我在抓包过程中经常使用的工具。

### burpsuite

#### 环境搭建

这个抓包工具相信都知道，可想而知其功能的强大，并且该工具的社区也是非常活跃，WEB必备工具之一，更新频繁。

首选我们来介绍环境的搭建

我喜欢使用Linux系统来搭建我的分析环境，当然其他环境下搭建是没有任何问题的。

下载地址：

e3dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8%4N6F1i4K6u0W2y4e0u0H3L8$3A6A6k6g2)9J5k6h3y4F1i4K6u0r3g2r3!0G2L8s2y4Q4x3V1k6z5k6i4c8%4L8%4u0C8i4K6g2X3b7h3&6S2L8s2W2\*7k6i4u0Q4x3V1j5`.

至于版本可以根据自己的喜好来选择，尽可能的选择比较新的版本

当然该项目是基于Java开发，那我们肯定需要使用到Java环境，首选对于我们开发人员来讲，可以直接使用JDK就可，笔者使用的JDK11版本

![图片描述](https://bbs.kanxue.com/upload/attach/202104/849527_2P67658K9ZTDVWQ.png)

注：比较新的BP是需要JDK8以上，不太支持JDK8

使用比较新的JDK版本还是有很多好处的，比如可以兼容JEB pro工具

如下图是笔者使用版本

简单的修改一下启动脚本就可以直接运行起来，非常方便

![图片描述](https://bbs.kanxue.com/upload/attach/202104/849527_7C7T8KKUXM6BXJD.png)

具体的注册方式在这里就不过多的讲解。接下来启动工具开始准备抓包环境

#### 代理抓包

![图片描述](https://bbs.kanxue.com/upload/attach/202104/849527_8FACV98YY5YTQS2.png)

1、首选在Options中选择配置我们的代理

2、设置端口

3、配置代理服务的IP

其中重要的是配置我们代理服务器的IP

如果我们是本地抓包，那我们可以直接配置本地地址就可以，浏览器代理上这个IP便可以实现抓包。

如果是借助路由器抓包，也就是手机或者其他设备抓包，只要处于同一局域网中，我们配置的IP就应该是本机的IP地址，然后被抓包的设备代理上就可以实现抓包。

大致的结构图如下：

![图片描述](https://bbs.kanxue.com/upload/attach/202104/849527_57JDT3Q47XUWQ8Y.png)

但是目前可能我们只讨论可以使用WIFI来进行上网服务的应用和设备，如果应用的流量是通过4G、网线上网那我们使用BP就不是这样来进行抓包，BP抓包的数据流量是位于应用层，如果是使用4G上网或者网线可能就需要借助接下来会介绍的TCPdump、wireshark等工具对网卡进行抓包分析。

![图片描述](https://bbs.kanxue.com/upload/attach/202104/849527_DF6FABNPZGX8F2W.png)

随着HTTP进入到HTTPS时代，上面这样简单的配置是没有办法实现抓包的。

我们用一张图片来解释HTTP->HTTPS的差异

![图片描述](https://bbs.kanxue.com/upload/attach/202104/849527_DW69DZB6EDGUXP3.png)

HTTPS的实现就是在传输过程中添加了一个SSL/TLS环节，使得我们的传输的流量是处于加密状态。

那是如何加密的呢？当然这里不是我们深入研究的主题

实际上就是使用RSA、DES等加密方式，在网络建立之前会使用秘钥协商来商定对称加密的秘钥，通常秘钥协商是会使用RSA算法，或者其他非对称的加密算法来实现（比如蓝牙中的秘钥协商就不完全采用RSA），协商完成后我们就可以使用对称加密来进行加密我们的流量。（具体的原理这里就不展开分析，后面开文章进行单独讲解）

![图片描述](https://bbs.kanxue.com/upload/attach/202104/849527_JKSJCVKFV4VAWVT.png)

既然如此就引入了CA证书的概念

证书的作用验证我们客户端和服务端，但是在很多开发者这里就不会很好的使用这样的机制，因此出现安全问题。相关的演变或者难易变化在第二节中讲解。

![图片描述](https://bbs.kanxue.com/upload/attach/202104/849527_GKFCAMH3BX44E9R.png)

#### 证书安装
