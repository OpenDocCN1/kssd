
# [原创]修改微信实现防撤回、自动抢红包功能

昨天看到大神skyun分享了一个实现微信`主动发消息 防撤回 抢红包 骰子作弊 模拟位置 步数最高`的模块。自己之前也搞过类似的事情，但是实现的方法不同，仅实现了`防撤回 抢红包 修改步数`的功能。其中修改步数是通过[StepChanger](https://bbs.kanxue.com/elink@eccK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6A6N6$3g2A6P5X3W2E0k6g2)9J5c8W2y4@1k6i4m8o6K9r3q4F1k6$3g2J5)实现的。防撤回、抢红包是通过修改微信APK实现的。这里主要描述一下防撤回、抢红包功能的实现思路。恰巧我修改微信的最新版本也是微信6.6.1。

## 效果图

![](https://bbs.kanxue.com/upload/attach/201804/677218_3K2VDFDSRWA94N8.png)

## 基本思路

一来是因为不想装Xposed，二来是因为觉得通过Hook实现的方法不够优雅，所以想通过直接修改微信的方式来实现防撤回和抢红包的功能。

在修改微信的过程中，一个重要的原则就是尽量重用微信自带的界面和组件，以求添加的界面和原界面保持风格一致。

### 1 过掉校验和资源混淆

研究过微信的人肯定都知道，用`apktool`反编译微信，是不能重打包的，因为微信使用[AndResGuard](https://bbs.kanxue.com/elink@addK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K6K9s2N6W2L8Y4A6Z5j5h3&6Y4i4K6u0r3b7h3&6V1f1X3g2K6c8%4g2S2M7X3b7`.)对资源做了混淆。即便用`apktool d -r`命令保持资源文件不变，重打包后的微信也不能运行，这是因为微信启动后会对自己的`dex`文件进行校验，校验失败就不能运行。

要过掉这两个问题也很简单，针对资源混淆的问题，使用[ShakaApktool](https://bbs.kanxue.com/elink@42cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6J5L8%4k6W2M7U0p5J5y4o6t1I4i4K6u0r3f1$3S2S2K9$3q4m8M7r3E0@1L8$3!0D9)。

针对校验问题，自己找到校验的位置然后把那一段代码patch掉。例如在微信6.6.1版本中，校验代码位于`smali/com/tencent/mm/f/a.smali`。

```smali
    .line 532

    const v2, 0x19000

    :try_start_2

    invoke-static {v3, v2}, Lcom/tencent/mm/a/g;->a(Ljava/io/InputStream;I)Ljava/lang/String;

    move-result-object v2

    .line 533

    if-eqz v2, :cond_2

    iget-object v4, p0, Lcom/tencent/mm/f/a$a;->eHC:Ljava/lang/String;

    invoke-virtual {v2, v4}, Ljava/lang/String;->equalsIgnoreCase(Ljava/lang/String;)Z

    :try_end_2

    .catch Ljava/io/IOException; {:try_start_2 .. :try_end_2} :catch_6

    .catchall {:try_start_2 .. :try_end_2} :catchall_2

    move-result v2

    if-eqz v2, :cond_2

    .line 534

    :try_start_3

    invoke-virtual {v5}, Lcom/tencent/tinker/loader/shareutil/ShareFileLockHelper;->close()V

    :try_end_3

    .catch Ljava/lang/Exception; {:try_start_3 .. :try_end_3} :catch_0

    .line 551

    :goto_1

    invoke-static {v3}, Lcom/tencent/mm/f/a;->b(Ljava/io/Closeable;)V

    goto :goto_0
```

### 2 研究微信的代码逻辑

这一部分主要是通过调试和Hook完成的。Hook框架用的[Frida](https://bbs.kanxue.com/elink@06eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6X3M7X3W2V1j5g2)9J5c8X3k6J5K9h3c8S2)。调试用的Android Studio + [smalidea](https://bbs.kanxue.com/elink@790K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6v1k6i4y4#2M7@1k6J5k6h3E0W2i4K6u0r3M7$3#2S2L8r3W2Q4x3V1k6%4K9h3E0A6)。

当然root权限是进行hook和调试的基础。有了root权限，开启调试权限的方法有两种。

一是修改`AndroidManifest.xml`里面的`android:debuggable`属性为`true`，开启某个app的调试权限。

二是使用大神提供的工具[`mprop`](https://bbs.pediy.com/thread-215311.htm)开启整个系统的调试权限。

调试和Hook的具体工作很繁杂，就不说了。

### 3 复用微信组件和界面

在研究完微信的代码逻辑后，就可以开始添加自己的代码了。反编译微信后得到的都是smali代码，手写那么多smali代码肯定是不太现实的。

这里采取的方式是用Android Studio新建一个工程，然后把微信的`dex`文件变成`jar`文件，再把这些`jar`文件当成库导入到工程中。接下来就可以写Java代码了。Java代码写好后，先编译成`class`文件，然后打包成`dex`文件，再反编译成`smali`文件。然后把这些`smali`文件复制到微信的代码目录中（这个过程好像有点蠢）。最后修改微信代码，在合适的地方调用自己添加的这些代码。

思路基本上是这样的，具体步骤太繁杂，这个就不写了。具体代码已经上传到[Github](https://bbs.kanxue.com/elink@3d6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6A6N6$3g2A6P5X3W2E0k6g2)9J5c8Y4N6W2j5$3S2S2N6l9`.`.)，成品就不上传了。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于  2018-4-12 13:44 被iweizime编辑，原因：
