
# [原创]某企业版APK加固原理分析

## 0x01. 加固后APP简单分析

1. 目前市面上的Android平台的APK加固己经很多了，而被加固的APK己很多，当我们拿到一个APK后首先要判断它是否加壳了、加了那家的的壳，一般有如下几种方法，最直接的方法可借住一些别人写好的查壳工具，我们本次分析的APK查壳情况如下图1所示：

   ![](https://bbs.kanxue.com/upload/attach/201604/706212_7d7ciznq2enca50.png)

   图1

2. 上面这种方法比较方便，便有时出现新的壳时可能会不能被查出来，这时就须要我们对目标APK进行反编译然后详细观察是否被加壳了，目前反编译APK个人觉得比较好用的工具主要有`APK改之理`、`AndroidKiller`，下面我们用`AndroidKiller`对本次要分析的APK进程反编译，如图2所示反编译时会提示“Invalid debug offset”，有这种提示一般说明`DEX`中的`debuginfo`值被改过了。

   ![](https://bbs.kanxue.com/upload/attach/201604/706212_xvarh2er03it0xt.jpg)

   图2

3. 下面我们来看看代码是否正常，`AndroidKille`反编译后它己经给我们找到到APK的入口类，如图3所示。

   ![](https://bbs.kanxue.com/upload/attach/201604/706212_3gbuxnlaynh2aun.png)

   图3

4. 我们将APK入口类转换在java代码方便阅读，转换后图4所示。

   ![](https://bbs.kanxue.com/upload/attach/201604/706212_t1ahdfs3b69csd1.png)

   图4

从图4可以看到其中`onClick`、`onCreate`等函数都变成了`native`函数了，这明显不太正常，我们从其它类中也发现类似的情况，大部分java函数都变成了`native`函数，而且同样调用了一个相同的图5所示函数。

![](https://bbs.kanxue.com/upload/attach/201604/706212_cjxcz53y8d82t5q.jpg)

图5

图5所示的类它会加载So库并进入So库中的`init`函数，如图6所示。

![](https://bbs.kanxue.com/upload/attach/201604/706212_289q5c8u2r4p457.jpg)

图6

从以上反编译后的分析情况，我们可以大致猜测出壳的运行流程如图7所示。

![](https://bbs.kanxue.com/upload/attach/201604/706212_k0150gpwnjybzxk.jpg)

图7

从上面只是大概了解了壳的基本情况，它是怎么修复？怎么加载的得下面详细分析SO程序了。

## 0x02. S0脱壳分析

1. 同样当我们要分析SO程序时也要判断它是否被加了壳，目前没有直接可视化的查壳工具了，判断方式主要是反编译观察代码情况(反编译后代码是否正常)并查看SO的`init`或`init_array`入口是否为空来做判断，因为加了壳的SO的壳入口一般都在`INIT_ARRAY`段和`INIT`段，用`readelf`查看SO信息，如图8所示。

   ![](https://bbs.kanxue.com/upload/attach/201604/706212_pvbuyh8tzjqrwm0.jpg)

   图8

2. 下面开始动态调试so模块，如何使用`IDA`调试android的SO模块，网上有很多教程，这里就不细说了，附加上进程后在壳入口下好断点开始分析，如图9所示：

   ![](https://bbs.kanxue.com/upload/attach/201604/706212_7tw22sg872cnr9v.jpg)

   图9

3. 继续往下分析须要过掉它的反调试，它的反调主要是以下3种：

   a. 检测进程的状态，打开`/proc/PID/status`，查找`TracerPid`。

   b. 在关键函数地方通过时间差值来检测是否中间有被单步调试执行，如图10所示，这是在解密代码时做的检测。

   ![](https://bbs.kanxue.com/upload/attach/201604/706212_wlkf6vl5bmuo9ae.jpg)

   图10

   c. 3个进程相互调试，以防止被附中，如图11、图12所示：

   ![](https://bbs.kanxue.com/upload/attach/201604/706212_qam4sxcm4qj9eex.jpg)

   ![](https://bbs.kanxue.com/upload/attach/201604/706212_ck72zk3ervvzjee.jpg)

   图11

   ![](https://bbs.kanxue.com/upload/attach/201604/706212_3ntmv6pc7y1urm7.jpg)

   图12

4. 代码解密就是解密导出的函数，循环解密了`0xEE`次，解密算法如图13所示：

   ![](https://bbs.kanxue.com/upload/attach/201604/706212_rimr88xolhnvbkx.jpg)

   图13

## 0x03. 编写SO静态脱壳机

1. 解密流程与算法都知道了，(详细解密过程请看`idb`)，就开始写程序解密了，部分代码如图14，详细请看附件工程。

   ![](https://bbs.kanxue.com/upload/attach/201604/706212_nexly8lv8dksr0m.jpg)

   图14

2. 将解密后的代码放到`IDA`中反编译，代码正常。

## 0x04. DEX壳流程分析

1. 当SO壳执行完成后，该解密的代码己经解密了，这时就执行到了`JNI_OnLoad`函数，首先注册`init`函数与获取`classes.dex`基址，如图15。

   ![](https://bbs.kanxue.com/upload/attach/201604/706212_5aycbrq3a9k3cpj.jpg)

   图15

2. 从数据节得到加密后的密码并解密数，该密钥做为后面解密数据的密钥，解密算法为`RC4`，解密后的密钥为`094K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6g2m8S2P5f1g2Y4K9i4y4Q4x3X3g2U0L8$3#2Q4x3V1k6K6K9r3q4J5K9%4c8W2j5h3#2Q4x3V1k6D9K9i4g2B7K9h3q4B7N6h3&6Q4c8f1k6Q4b7V1y4Q4z5p5y4Q4c8e0g2Q4b7e0k6Q4z5o6u0Q4c8e0g2Q4z5f1u0Q4b7V1f1I4y4W2!0q4y4W2)9^5z5g2)9^5x3q4!0q4y4#2!0m8y4q4!0n7b7g2!0q4x3#2)9^5x3q4)9^5x3R3`.`.

   ![](https://bbs.kanxue.com/upload/attach/201604/706212_b0b4v5flzsdeip8.jpg)

   图16

3. 为还原方法属性做准备，从数据节得到两个数据块，该数据在修复方法时会用到，解密第一块数据，如图17所示。

   ![](https://bbs.kanxue.com/upload/attach/201604/706212_4md7akao0lvebw4.jpg)

   图17

解密数据后从使用情况来分析，可以得到该数据的结构如图18所示：

![](https://bbs.kanxue.com/upload/attach/201604/706212_0jrjfum9b9sz9m8.jpg)

图18

4. 从上一个结构可以看到最后一个成员就是要修复方法的数据结构偏移值，而第二块要解密的数据就是该结构，如图19所示。

   ![](https://bbs.kanxue.com/upload/attach/201604/706212_lgfnhl01brjc0xr.jpg)

   图19

结构大致如图20所示：

![](https://bbs.kanxue.com/upload/attach/201604/706212_cv8ti8is7hu3m93.jpg)

图20

5. 把该准备好的工作都完成后就进入了`init`函数，函数先是解密java层传进来的`Base64`字符，如图21所示。

   ![](https://bbs.kanxue.com/upload/attach/201604/706212_55ai8v1y1hkzu7p.jpg)

   图21

6. 根据解密后的类名判断是否须要修复，判断方式为将类名计算得到一个值与第一块解密的数据中第一个成员比较，如果相等就是须要修复的，如图22所示。

   ![](https://bbs.kanxue.com/upload/attach/201604/706212_sdfnqpff6e9wqt5.jpg)

   图22

7. 如果须要修复就进入方法修复流程，根据第一块数据结构中的最后一个成员值得到第二块数据的结构，然后`FindClass`->`GetxxcMethodID`->`GetMethodID`进行方法属性还原。

## 0x05. 总结

1. 最后再来回顾下壳的流程，java层`Splash` -> `ProxyApplication.init` -> `SO壳` -> `JNI_OnLoad` -> 解密修复方法须要的数据 -> `init`函数 -> 修复执行。

附件下载 `ae3K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4m8S2L8W2)9J5k6h3u0S2K9h3c8#2i4K6u0W2j5$3!0E0i4K6u0r3M7#2)9J5c8U0q4E0K9f1W2f1x3r3c8y4`

教程己被买断版权，不再对外开放培训！！！

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [15.jpg](javascript:void(0);) （74.59kb，15次下载）
- [17.jpg](javascript:void(0);) （14.54kb，11次下载）
- [18.jpg](javascript:void(0);) （58.31kb，9次下载）
- [19.jpg](javascript:void(0);) （52.52kb，10次下载）
- [0.jpg](javascript:void(0);) （40.75kb，9次下载）
- [-1.jpg](javascript:void(0);) （173.81kb，10次下载）
- [2.jpg](javascript:void(0);) （27.49kb，12次下载）
- [3.jpg](javascript:void(0);) （19.84kb，9次下载）
- [4.jpg](javascript:void(0);) （33.64kb，10次下载）
- [23.jpg](javascript:void(0);) （41.42kb，8次下载）
- [24.jpg](javascript:void(0);) （74.68kb，11次下载）
- [25.jpg](javascript:void(0);) （87.86kb，10次下载）
- [5.jpg](javascript:void(0);) （46.80kb，11次下载）
- [6.jpg](javascript:void(0);) （36.49kb，9次下载）
- [7.jpg](javascript:void(0);) （31.66kb，11次下载）
- [26.jpg](javascript:void(0);) （53.29kb，9次下载）
- [8.jpg](javascript:void(0);) （33.56kb，7次下载）
- [27.jpg](javascript:void(0);) （50.49kb，7次下载）
- [9.jpg](javascript:void(0);) （44.13kb，10次下载）
- [12.jpg](javascript:void(0);) （51.08kb，11次下载）
- [app1.png](javascript:void(0);) （57.07kb，9次下载）
- [app2.png](javascript:void(0);) （111.19kb，7次下载）
- [app3.png](javascript:void(0);) （138.79kb，8次下载）
