
起先我看到这里的时候并不知道是进行了 hook 操作，在进行不断搜索的过程找到了一位大佬的博客（<<mark class="encrypted">5d9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3P5r3W2S2L8$3u0S2K9i4W2W2P5g2)9J5c8Y4m8Q4x3V1j5$3y4e0f1%4z5o6f1@1i4K6u0W2K9s2c8E0L8l9`.`.</mark>>）和他的 github项目（<<mark class="encrypted">700K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6^5K9h3q4G2j5X3q4A6P5h3g2&6i4K6u0r3k6r3g2^5L8r3!0S2k6q4)9J5c8X3u0D9L8$3u0Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3V1k6V1k6i4S2D9L8$3q4V1i4K6u0r3f1V1g2m8c8p5#2q4i4K6u0W2L8h3b7`.`.</mark>>）。

![](upload/attach/202006/802108\_3E48YRU3DMEZ6AQ.png)

进行了一番学习和分析后，才确定这里是进行了 hook 操作，感谢大佬们的无私分享精神！有空要花时间好好研究下 ELF 文件的 hook。

然后再往下面分析，会走到调用`sub_35E0`函数的地方，里面就是调用 Java 层中`MultiDex`类中的`installDexes`地方。我们可以通过调试很容易判断要走哪些代码。

![](upload/attach/202006/802108\_XVRREYP8G5AFC7X.png)

然后来看`installDexes`方法，

![](upload/attach/202006/802108\_DFWX624HURGYYNH.png)

![](upload/attach/202006/802108\_8B5KMYBG5NSP88U.png)

看到会调用到`install`方法，然后可以看到又调用了`V19.makeDexElements`，查看方法具体实现就是通过反射调用了`makeDexElements`方法来加载 Dex 文件，再后面的过程就是在前面说`ClassLoader`已经说到了的。

`ClassLoader`加载了两个 Dex 文件，前面也说到实际上加载的这两个文件是空文件，但是由于进行了 hook 操作，`ClassLoader`在底层函数会加载前面再内存中解压出来的抽取了指令的 Dex 文件。

## 4.8 `sub_6DC0`——抽取指令还原

下面让我们回过头看`sub_58000`函数中调用`sub_6DC0`的地方，也就是我们说到的还原填充了内存中 Dex 文件被抽取的指令的地方。

![](upload/attach/202006/802108\_BZWK57V9H9HG3Y8.png)

这个函数很长，描述起来多少会比较模糊，慢慢调试分析可能会感受得更清晰些。函数可以分为三部分，先提下的就是`salsa20`加密函数，我是搜索后进行确认的，具体的加密可以看大佬的这篇博客（<<mark class="encrypted">ff1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3j5i4q4#2j5i4u0Q4x3V1k6H3i4K6u0r3z5o6b7K6y4K6p5%4x3W2)9J5k6h3S2@1L8h3H3`.</mark>>）。

第一部分是处理前面说的`data1`和`data2`数据。

![](upload/attach/202006/802108\_7AP6PZS57BEJBWJ.png)

![](upload/attach/202006/802108\_UWKQT4MKDNJ8GBP.png)

数据解压后还会进行读取，得到指导抽取指令数据进行填充的重要数据。

![](upload/attach/202006/802108\_XBKC84FGBEASV3M.png)

![](upload/attach/202006/802108\_CZBPNVTQK4DPRB5.png)

第二部分是处理前面说的`data3`和`data4`数据。

![](upload/attach/202006/802108\_V7FMDP9D8XD24AJ.png)

第三部分就是通过`memcpy`在`do-while`循环中根据得到的指导数据和解压数据，进行对内存中被抽取指令的 Dex 文件的填充了。

![](upload/attach/202006/802108\_WYSAKPRXU2TQMYW.png)

`sub_6DC0`到此结束，当执行完这个函数后，我们再对内存中的 Dex 进行 dump 的时候，就会发现指令已经被还原了！

![](upload/attach/202006/802108\_R3439VUYUNEUP6V.png)

## 4.9 完成 `Application` 的替换

Orz 终于要结束了，现在已经完成了抽取指令的还原了，让我们来看`sub_58000`最后一部分吧，就是完成`ProxyApplication`到`DelegateApplication`的替换过程。

![](upload/attach/202006/802108\_GJFCPD56MQGWVGR.png)

主要进行了三个操作，设置`ClassLoader`、创建`DelegateApplication`的实例和进行`baseContext`的`attach`操作，这时候控制权就已经从`ProxyApplication`交给`DelegateApplication`了，具体的函数我们就不细看了。

然后整个`JNI_OnLoad`函数要做的事情差不多就做完了，但是`Application`的替换还没结束。

别忘了 Java 层执行了的只是`attachBaseContext`，还有`onCreate`要执行。

![](upload/attach/202006/802108\_9TWHJMUEYSN7JWR.png)

我们来看下发现执行了原生函数`Ooo0ooO0oO`，也就是在开始注册的 so 层中的`sub_30E0`函数。

![](upload/attach/202006/802108\_FMRSB2VRAGQJWGH.png)

看下`sub_A380`函数，可以看出是替换了 API 层的所有`Application`引用。

![](upload/attach/202006/802108\_SUJAM2WQQ296XSU.png)

随后`sub_A2F0`中又处理`ContentProviders`，最后调用了我们的`DelegateApplication`的`onCreate`方法，也就是用户`Application`。

到此壳的运行结束了。

# 5. 结语

写到这里其实已经写不动什么了，写文章总结纪录不是一件容易的事，十分地消耗精力和时间...

希望这篇文章能帮助像我一样入门学习 so 层分析的新手，如果觉得学习到了有用的东西，请务必点赞，也不枉我写了这么多。

# 6. 一些学习资料

这次分析中还有很大的收获就是找到了很多优秀的文章和博客，耐心分析学习着慢慢看懂了，再次感谢大佬们的无私分享精神！

Android的Proxy/Delegate Application框架

<<mark class="encrypted">852K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4M7#2)9J5k6e0x3$3x3q4)9J5k6h3y4F1i4K6u0r3M7r3!0K6N6q4)9J5c8Y4m8J5L8%4S2&6k6r3g2D9k6h3N6S2N6r3g2Q4x3X3c8S2M7s2m8D9K9h3y4S2N6r3W2G2L8W2)9J5k6h3S2@1L8h3I4Q4x3U0y4U0L8$3#2E0k6h3&6@1i4K6u0V1y4K6M7`.</mark>>

性能优化 (八) APK 加固之动态替换 Application

<<mark class="encrypted">aadK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6B7N6h3g2B7K9h3&6Q4x3X3g2A6L8g2)9J5c8Y4m8G2M7%4c8Q4x3V1j5#2j5$3j5$3z5h3b7K6x3r3j5J5y4U0g2V1j5e0q4T1z5o6V1%4j5h3u0V1y4e0y4Q4x3U0y4Z5k6h3q4V1K9h3&6Y4i4K6u0V1y4b7`.`.</mark>>

利用动态加载技术加固APK原理解析

<<mark class="encrypted">7b6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6B7N6h3g2B7K9h3&6Q4x3X3g2A6L8g2)9J5c8Y4m8G2M7%4c8Q4x3V1j5#2j5$3j5$3z5h3b7K6x3r3j5J5y4U0g2V1j5e0q4T1z5o6V1%4j5h3u0V1y4e0y4Q4x3U0y4Z5k6h3q4V1K9h3&6Y4i4K6u0V1y4b7`.`.</mark>>

基于代理Application机制的Anddroid应用加壳方法

<<mark class="encrypted">6f6K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3c8#2L8%4u0W2L8Y4N6W2K9g2)9J5k6h3y4G2L8g2)9J5c8X3&6W2N6%4y4Q4x3V1j5@1y4e0x3&6i4K6u0W2K9s2c8E0L8l9`.`.</mark>>

第十二章 软件壳（四）（代码抽取型壳）

<<mark class="encrypted">5f9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4A6D9L8h3@1%4y4o6q4Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8U0p5H3y4U0p5%4x3K6M7%4z5q4)9J5c8R3`.`.</mark>>

Android中apk加固完善篇之内存加载dex方案实现原理(不落地方式加载)

<<mark class="encrypted">eb1K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6e0f1J5x3r3#2G2L8X3E0W2P5g2)9J5k6h3y4G2L8g2)9J5c8X3q4J5j5$3S2A6N6X3g2K6i4K6u0r3y4U0t1&6</mark>>

阿里早期Android加固代码的实现分析

<<mark class="encrypted">ef5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4q4I4x3e0l9^5y4o6t1^5x3K6p5%4x3W2)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3y4K6R3K6x3U0l9@1y4o6g2Q4x3@1k6#2N6r3#2Q4y4h3k6K6L8%4g2J5j5$3g2Q4x3@1c8Y4L8$3I4V1i4K6g2X3j5Y4u0G2N6%4y4W2M7W2)9#2k6X3g2^5N6r3g2F1M7$3W2G2L8R3`.`.</mark>>

android动态加载dex支持art

<<mark class="encrypted">901K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3f1#2x3Y4m8G2K9X3W2W2i4K6u0W2j5$3&6Q4x3V1k6@1K9s2u0W2j5h3c8Q4x3X3b7#2z5e0l9I4z5e0u0Q4x3X3b7I4i4K6u0V1x3g2)9J5k6h3S2@1L8h3H3`.</mark>>

Android APK加固-内存加载dex

<<mark class="encrypted">2bfK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3L8s2c8&6j5h3&6V1P5g2)9J5c8Y4m8Q4x3V1j5I4x3e0j5@1x3U0p5H3z5q4)9J5k6h3S2@1L8h3H3`.</mark>>

Android APK加固-完善内存dex

<<mark class="encrypted">0a4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3L8s2c8&6j5h3&6V1P5g2)9J5c8Y4m8Q4x3V1j5I4x3e0j5@1y4o6l9#2y4W2)9J5k6h3S2@1L8h3H3`.</mark>>

APK加壳【2】内存加载dex实现详解

<<mark class="encrypted">41dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6k6h3N6E0k6h3&6@1k6X3q4#2L8s2c8Q4x3X3g2U0L8$3#2Q4x3V1k6S2i4K6u0r3x3e0p5&6x3o6l9H3x3o6l9H3x3U0f1%4z5o6M7#2y4b7`.`.</mark>>

Android4.0内存Dex数据动态加载技术

<<mark class="encrypted">81aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8X3q4F1k6s2u0G2K9h3c8K6k6h3y4#2M7X3W2@1P5g2)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3z5e0j5%4y4o6t1#2x3b7`.`.</mark>>

Android安全分析挑战：运行时篡改Dalvik字节码

<<mark class="encrypted">95aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8X3q4F1k6s2u0G2K9h3c8K6k6h3y4#2M7X3W2@1P5g2)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3z5o6R3K6x3K6M7I4x3l9`.`.</mark>>

阿里早期加固代码还原4.4-6.0

<https://bbs.pediy.com/thread-215078.htm>

Android第二代加固（support 4.4-8.1）

<https://bbs.pediy.com/thread-225303.htm>

如何修复使用NOP指令抹去关键方法的DEX文件

<<mark class="encrypted">d2dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2L8Y4q4#2j5h3&6C8k6g2)9J5k6h3y4G2L8g2)9J5c8Y4m8G2M7%4c8Q4x3V1k6A6k6q4)9J5c8U0R3#2z5o6R3$3</mark>>

DEX保护之指令抽取

<<mark class="encrypted">0ffK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2X3M7X3g2W2j5Y4g2X3i4K6u0W2j5$3!0E0i4K6u0r3j5i4u0@1K9h3y4D9k6i4y4Q4x3V1k6@1k6i4u0E0K9h3&6S2L8q4)9J5c8U0t1H3y4e0l9%4z5g2)9J5k6h3S2@1L8h3H3`.</mark>>

dvm，art模式下的dex文件加载流程

<<mark class="encrypted">ba9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8X3@1H3i4K6g2X3x3K6M7K6y4o6b7%4z5e0m8Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8U0M7^5y4e0t1K6x3e0b7%4</mark>>

Android DEX加壳

<<mark class="encrypted">e65K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6&6M7g2)9J5k6h3q4D9K9i4W2#2L8W2)9J5k6h3y4G2L8g2)9J5c8X3q4J5N6r3W2U0L8r3g2K6i4K6u0r3y4U0j5K6x3U0V1@1</mark>>

Salsa20笔记

<<mark class="encrypted">536K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3j5i4q4#2j5i4u0Q4x3V1k6H3i4K6u0r3z5o6b7K6y4K6p5%4x3W2)9J5k6h3S2@1L8h3H3`.</mark>>

Bangcle
