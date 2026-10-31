
# [分享][原创] 一次针对反 `fart` 的加壳 APK 的脱壳经历

## 一、简介

最近碰到一个加壳的 APK，里面有类抽取，并且有做反 `fart` 脱壳。后来经过修改 `fart` 的源码，幸运地脱壳并还原 `dex` 成功。特将脱壳经历记录于此。

此次遇到的壳中，两个核心的 `dex` 文件中几乎所有的方法的 `codeitem` 都抽取了。

![](upload/attach/202109/892380_868P49JA3E2DJUC.png)

庆幸的是，最后还原出来了：

![](upload/attach/202109/892380_2YUKF7QW8R8KDRP.png)

## 二、学习 `fart` 脱壳机源码

从 `hanbing` 大神的 GitHub 上下载源码：<mark class="encrypted">f54K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6Z5j5h3&6T1K9h3&6Y4L8r3g2F1k6%4W2#2k6g2)9J5c8V1k6m8f1W2c8Q4c8e0y4Q4z5o6m8Q4z5o6t1`.</mark>

`fart` 的代码量也不大，其中一部分代码是原来系统的代码，一部分是作者自己加的代码。要想快速知道作者加了哪些代码，可以直接将 `fart` 代码文件和相应的系统源码文件（GitHub 源码中代码是对应 Android 6.0 系统的）双选拖到 Beyond Compare 中，然后作者自加的代码便一目了然（最左侧小红线的地址就是代码差异的地方）。

![](upload/attach/202109/892380_SZTPBDDXBNG7FV9.png)

修改和编辑系统源码的 IDE，考虑到只是简单地修改一些代码，这里使用了 VS Code，直接将系统源码根目录导入即可，然后根据 VS Code 提示，安装一些插件也能有一些简单的代码提示和定位功能。

## 三、使用原始 `fart` dump 不了 `codeitem` 原因排查

一开始使用 `fart`（`fart` 脱壳机兼 Frida 版 `fart`）时，有两个 `dex` dump 下来是类被抽取了，并且 `codeitem` 没 dump，显然是对 `fart` 做了防护。脱壳机中未 dump 出 `codeitem` 截图（Frida 版 `fart` 也没有，这里就不给出截图），如下：

![](upload/attach/202109/892380_9Z35UTTBPFB3Q33.png)

为了排查原因和找到 `fart` 哪一步被卡住了，本人缩小排查范围，并在 `fart` 源码多个环节打印 log，过程如下：

1. 原始的 `fart` 是对手机上所有的应用脱壳，这其中的日志不是一般的多，手机也比较卡，因此本人对脱壳的包名进行了限制，如下：

   ![](upload/attach/202109/892380_9QMGY4T3WSWVEHU.png)

   另外，即使在同一个应用中，上图中的代码也可能运行两次，因此加上 `mIsRunFartThread` 限制一个程序只进行一次脱壳。

2. 对 `fartwithClassloader` 中 `ClassLoader` 参数进行过滤。因为不是所有的 `dex` 都做了类抽取，所以仅对包含了文章开头给出的两个 `dex` 中的类（各取一个类代表即可）的 `ClassLoader` 进行排查，其他的滤掉，如下：

   ![](upload/attach/202109/892380_RNS2GQW4VZVQAFN.png)

3. 在 `fart` 各个必经函数处打印 log。

最后排查出是 `fart` 在反射调用 `getClassNameList_method` 方法时，返回的类名列表为空。针对这个问题，本人处理的方法是将 `dex` 的类名手动导出到文件（可以通过 010 导出类名列表，也可直接使用 `fart` 中另一个脱壳点 `dumpDexFileByExecute` 导出的类列表，后者更快一些，但前提是得跑一次原始的 `fart`），然后存放到 `sdcard` 中，并修改 `fart` 源码从 `sdcard` 中读取该文件，从而获取 `dex` 的所有类名列表。后面的自动调用类方法的流程没有变。

![](upload/attach/202109/892380_2SX473VS72WAHW7.png)

虽然，目前反 `fart` 的原因找到了，但离最后脱壳还有一些距离。

## 四、剩余问题解决

接下来运行过程中还碰到了一个问题：类的 `<clinit>` 方法的 `codeitem` dump 不下来。

`<clinit>` 的 `methodid` 不能通过 `getDeclaredConstructors` 和 `getDeclaredMethods` 获取到，所以原始的 `fart` 是没有对 `<clinit>` 方法进行主动调用的。后面发现 `Class.forName(cls)` 是可以触发 `cls` 中的 `<clinit>` 调用的，但是使用时，得用 `Class.forName(eachclassname, true, appClassloader);`，不然在 `fart` 代码调用位置默认是使用 `BootClassLoader`，会导致类找不到。因此，把原始 `fart` 中获取类的代码修改如下：

```java
resultclass = Class.forName(eachclassname, true, appClassloader); // 原来是 appClassloader.loadClass(eachclassname);
```

`fart` 中还有另一个脱壳点 `dumpdexfilebyExecute`，会将所有执行过的方法都 dump 下来。至此，所有的 `codeitem` 都获取到了，然后使用 `hanbing` 大神提供的 `fart.py` 可以将 `codeitem` 和 `dex` 中反编译代码通过命令行显示出来。对 `fart.py` 进行简单修改，可以将 `codeitem` 和 `dex` 合并，获取完整 `dex`。

## 五、后序

如果文中有不对的地方，还请批评指正。脱壳中过程文件和修改的脱壳机代码已加至：<a href="elink@e2fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6o6K9r3g2F1P5h3q4F1k6$3#2A6L8X3M7&6i4K6u0r3L8$3&6W2i4K6u0V1j5h3&6@1K9g2)9J5k6r3k6S2M7Y4c8Q4x3X3c8K6K9r3g2D9L8q4)9J5k6s2m8#2L8r3H3`." ><mark class="encrypted">4ecK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6o6K9r3g2F1P5h3q4F1k6$3#2A6L8X3M7&6i4K6u0r3L8$3&6W2i4K6u0V1j5h3&6@1K9g2)9J5k6r3k6S2M7Y4c8Q4x3X3c8K6K9r3g2D9L8q4)9J5k6s2m8#2L8r3H3`.</mark></a>，仅供参考。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2021-9-24 13:12

被 Denny Chen 编辑

，原因：

[#脱壳反混淆](https://bbs.kanxue.com/forum-161-1-122.htm)
