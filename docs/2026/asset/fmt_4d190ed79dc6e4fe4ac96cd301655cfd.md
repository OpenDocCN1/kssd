
## 静态分析

```scp root@192.168.2.3:/private/var/root/Yelpv7 .
```

解密过程可以手动实现，通过启动应用程序，附加`gdb`到进程然后`dump`内存未加密程序到文件。如果用解密工具失败了，这是个值得考虑的方法。手动解密过程不包含在这篇文章中。

现在有了未加密的可执行文件，就可以用Hopper、IDA或其他反编译工具去分析它。

下面，Hopper分析后的几张截图：

![](upload/attach/202002/687147_MSKQKFCQD2HW8KT.jpg)
![](upload/attach/202002/687147_7W4MEPDRG9GNEZB.jpg)
![](upload/attach/202002/687147_84RYJ7WWGQRQGK4.jpg)

### Check for Plist Files

[Jailbreak unnecessary]应用程序可能使用`plist`文件来存储配置信息。`Plist`是一个XML结构的属性列表文本文件。当评估一个iOS应用程序,其中一个检查是存储在`plist`文件的硬编码凭证。访问这些文件不需要越狱设备。 [i-funbox](http://www.i-funbox.com/)是一个工具,可以导出iOS设备上应用沙盒中的数据,包括`plist`文件。

### Check for Keychain Data and SQLite DBs

[Jailbreak needed]钥匙链是iOS提供的一个允许应用程序存储敏感数据服务，如凭证或API访问令牌。钥匙链会在数据存储在文件系统之前加密。越狱的设备可以完全访问存储在钥匙链的数据。 [Keychain-Dumper](https://github.com/ptoomey3/Keychain-Dumper) 是在越狱设备上导出所有钥匙链数据的工具。当评估一个应用程序，试着查看下它在钥匙链存储的数据。值得称赞的是，数据被一个操作系统提供的额外加密方法加密存储在密钥链中。即使加密密钥可以通过逆向应用得到，也会推迟阻拦一些没经验的攻击者。

[Jailbreak unnecessary]在iOS，可以在SQLite数据库中存储数据。`sqlite`文件可在应用程序沙盒数据目录中。默认情况下，SQLite数据库不加密。[SQLcipher](https://www.zetetic.net/sqlcipher/)是一个可以加密的好选择。当评估一个应用程序时，一定要检查在SQLite数据库中是否保存任何数据和什么样的数据，看看是否启用加密。

### Check for HTTP Caching Responses

[Jailbreak unnecessary]如果应用程序使用了内置`NSURLRequest`执行HTTP请求，响应可能会被缓存在与应用相关的`Cache.db` SQLite文件。远程API / Web服务器的响应不包含`Cache - Control`头字段，敏感信息可能会被存储在设备上。注意，缓存数据存储在这个文件是不加密的。

### MobFS

[Jailbreak needed][MobFS](https://github.com/ajinabraham/Mobile-Security-Framework-MobSF)是一个进行静态/动态分析iOS和Android应用程序的开源工具。在这篇文章中不会涉及这个工具，但审查时还是值得试一下。注意，越狱是在没有IPA需要从设备提取时才需要。

## 动态分析

动态分析是在运行时对应用的过程进行操纵，为了试图实现如下目的：

- 绕过越狱检测
- 窃取加密密钥或敏感数据
- 绕过某些控制加载ViewControllers
- 攻击本地身份验证(如果存在)
- 当接触企业应用时转向内部网络
- 检查自定义加密算法

### Instrumentation

[Jailbreak needed]有一些工具来执行所谓的`instrumentation`，在运行时探索和修改应用过程的流程。比较有名的有：

- Cycript
- Frida
- Cydia substrate
- GDB
- 动态链接器来覆盖函数（`LD_PRELOAD`）

### Cycript

[Jailbreak needed] [Cycript](http://www.cycript.org/)是一个很棒的工具来探索和修改在iOS上运行的应用程序。它使用一个混合的语法，Objective-C和JavaScript语法。它可以通过应用程序的进程名称或PID开始：

```cycript -p Yelp
```

一旦附加到进程，就可以探索应用程序组件：

```
cy# UIApp
#"<UIApplication: 0x1667a760>"
cy# [UIApplication sharedApplication]
#"<UIApplication: 0x1667a760>"
cy# UIApp.keyWindow
#"<UIWindow: 0x17bdfd70; frame = (0 0; 768 1024); gestureRecognizers = <NSArray: 0x17f14270>; layer = <UIWindowLayer: 0x17d23150>>"
cy# UIApp.keyWindow.rootViewController
#"<YPUITabBarViewController_iPad: 0x17c30b90>"
cy# UIApp.delegate
#"<YPUIApplicationDelegate_iPad: 0x1667fd30>"
```

可以手动调用方法，改变当前的`ViewController`等等。

在以下示例中，我们假装`YPUIApplicationDelegate_iPad`类实现一个方法称为`correctCredentials`验证用户输入凭据，如果凭证是正确的，返回`true`。Cycript允许在运行时重新定义方法。在这种情况下，当调用此方法时我们要始终返回`true`：

```
cy# YPUIApplicationDelegate_iPad.prototype['correctCredentials'] = function(){return true;}
function () {return!0;}
```

相同的方法可以用到应用的越狱保护手段。

当评估一个应用程序，检查什么样的绕过手法可以通过操纵运行时过程完成。如果绕过某些函数可以很容易地实现破解，这可能是应用程序本身的设计问题。

### Snoop-it

[Jailbreak needed] <a href="elink@d47K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0L8$3c8W2i4K6u0W2k6$3!0G2k6$3I4W2i4K6u0W2j5$3!0E0i4K6u0r3j5i4u0U0K9r3W2$3k6g2)9J5c8Y4m8Q4x3V1k6K6L8X3!0G2M7q4)9J5k6r3W2@1i4K6u0r3">Snoop-it</a>是另一个很棒的工具来执行iOS应用程序的动态分析。我个人觉得它有点不稳定，Snoop-it附加应用时经常会导致应用崩溃。但除此之外，它提供了一个用户友好的web界面和极好的功能，下面描述。

#### Snoop-it Filesystem Module

这个模块监控应用程序的文件访问：

![](upload/attach/202002/687147_DRDSV3NDN5BAZAA.jpg)

#### Snoop-it Crypto Module

这个模块跟踪常见的加密API调用：

![](upload/attach/202002/687147_SBMFUV4YZNRDZR8.jpg)

#### Snoop-it Fake Location Module

这个模块允许应用程序请求当前设备GPS坐标时提供一个假的位置：

![](upload/attach/202002/687147_8EUU3ENK3V55397.jpg)

#### Snoop-it Hardware Spoof Module

这个模块允许重写设备的UDID和mac地址：

![](upload/attach/202002/687147_VN5KPRX5H8J9M4A.jpg)

#### Snoop-it Keychain Module

这个模块跟踪访问或存储在钥匙链的数据：

![](upload/attach/202002/687147_RCFZ3AF4X28XJYK.jpg)

#### Snoop-it Method Invoke Module

这个模块允许在运行时调用方法，和之前描述Cycript的方法相似：

![](upload/attach/202002/687147_GK28563EPGVF6RK.jpg)

#### Snoop-it Method Tracing Module

这个模块跟踪应用程序执行过程中调用的方法：

![](upload/attach/202002/687147_CFCTC7YBWFBUDRR.jpg)

#### Snoop-it Network Module

这个模块跟踪应用程序的HTTP和HTTPS请求：

![](upload/attach/202002/687147_5B3F9UNDPUHBG44.jpg)

#### Snoop-it Sensitive API Module

这个模块跟踪敏感的API调用，比如当前设备位置，无线网络接口的Mac地址，等等：

![](upload/attach/202002/687147_7K2RDUP3K7FKV82.jpg)

#### Snoop-it URL Schemes Module

URL schemes是与其他应用通信的一种方法。该模块显示URL schemes在应用程序中实现：

![](upload/attach/202002/687147_K3DGMNHKCHRH9JR.jpg)

### IDB

[Jailbreak needed] <a href="elink@aa0K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3W2V1j5Y4c8G2L8$3I4Q4x3X3g2U0L8$3#2Q4x3V1j5`.">IDB</a>是另一个开源工具，用Ruby编写，和Snoop-it以类似的方式工作。它不允许像Snoop-it那样动态调用方法，但它有其他的功能，如检查系统日志和粘贴板，和测试URL schemes。

#### IDB App Info

![](upload/attach/202002/687147_US2FS7C53JKHJNZ.jpg)

#### IDB Keychain

![](upload/attach/202002/687147_CV5GTVK3HFTZA52.jpg)

#### IDB Log

![](upload/attach/202002/687147_34SCSUZEVAKP26Y.jpg)

#### IDB Pasteboard

![](upload/attach/202002/687147_SMVVP2XA7AV67P2.jpg)

#### IDB Shared Libraries

![](upload/attach/202002/687147_5BMASW79FFYUSG9.jpg)

#### IDB Storage

![](upload/attach/202002/687147_Y9ZD864A8V7KEYV.jpg)

#### IDB URL Handlers

![](upload/attach/202002/687147_WXNB79UK2B66D9D.jpg)

### introspy-ios / introspy-analyzer

[Jailbreak needed] <a href="elink@5a6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6A6f1@1g2o6f1r3q4J5N6r3&6W2M7Y4y4Q4x3V1k6u0L8Y4c8J5L8%4y4H3P5g2)9J5k6r3W2a6f1H3`.`.">Introspy-iOS</a> 帮助自动化应用的运行时分析和识别潜在的安全问题。Introspy-iOS(`tracer`)hook和检查应用的敏感API调用。它将结果存储在设备上的本地SQLite数据库中。 <a href="elink@180K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6A6f1@1g2o6f1r3q4J5N6r3&6W2M7Y4y4Q4x3V1k6u0L8Y4c8J5L8%4y4H3P5g2)9J5k6p5q4F1j5h3I4&6P5X3g2J5">Introspy-Analyzer</a>是一个工具，帮助格式化tracer的数据并生成HTML报告。下面的例子是这个工具hook Yelp应用的结果。

![](upload/attach/202002/687147_JDJS7JFFXPBK5VV.jpg)

![](upload/attach/202002/687147_K8BDDQQ96UDZYW8.jpg)

### Snapshot（快照）

[Jailbreak unnecessary]当应用进入后台，按设备`home`按钮，iOS将对当前应用的屏幕截图。截图保存到下一次设备重启。当评估一个应用程序，检查是否显示任何敏感信息;如果是的，当信息显示时让应用进入后台然后检查iOS的屏幕截图。如果是越狱设备，Needle，先前在这篇文章中提到的工具，提供了一种简便的方法来执行这个测试并自动下载截图给你。否则，手动定位文件在应用沙盒目录中位置，使用i-funbox导出。

### Jailbreak Detection（越狱检测）

[Jailbreak needed]作为深入的安全措施，应用程序可能会执行越狱设备检测并拒绝在越狱设备上运行。如果敏感数据存储在设备上，应用程序一旦检测到越狱可能会删除这些数据。当评估一个应用程序，检查越狱检测是否执行以及其有效程度。

### Anti-Debugging Protection(反调试保护)

[Jailbreak needed]作为深入的另一个安全措施，应用程序可能会发现它在调试模式下运行，被调试器连接，如`GDB`，然后停止运行。即使有经验的人能够绕过这样的保护，缺乏经验的人可能会被阻拦而转向下一个目标。

### Keyboard Cache(键盘缓存)

[Jailbreak needed]当在iOS应用内部的输入框输入文本时，数据为了自动校正会被缓存，除了密码类输入框和一些其它特定的字符串。当评估一个应用时，检查哪些数据被存储在缓存中。

### Buffer Overflow / Format Strings / Memory Corruption(缓冲区溢出/格式字符串/内存崩溃)

[Jailbreak unnecessary]iOS应用程序易受到内存崩溃漏洞的攻击，如缓冲区溢出和格式字符串。在进行审查时，值得试着模糊测试用户输入，检查应用程序如果崩溃，这可能表明应用程序是脆弱的。如果测试越狱设备，可以用调试器进一步分析崩溃。

## 参考

- Apple Sandbox: <mark class="encrypted">90bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2S2M7s2m8D9k6g2)9J5k6h3y4G2L8g2)9J5c8X3I4A6j5Y4u0S2M7Y4W2Q4x3V1k6U0L8$3&6@1k6h3&6@1i4K6u0r3k6r3!0U0N6h3#2W2L8Y4c8S2N6r3W2G2L8W2)9J5c8W2y4W2j5%4g2J5K9i4c8&6i4K6u0r3b7$3!0F1j5$3g2H3N6s2g2S2L8q4)9J5c8V1q4H3M7q4y4S2L8X3c8T1L8%4S2p5k6i4y4A6k6$3&6s2N6h3W2V1k6g2)9J5c8V1q4H3M7q4y4S2L8X3c8T1L8%4S2u0L8V1c8W2M7s2c8Z5i4K6u0r3b7i4m8H3f1$3q4F1k6r3u0G2P5p5W2F1c8r3g2H3N6r3S2Q4x3X3g2Z5N6r3#2D9</mark>，
  <mark class="encrypted">d60K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2S2M7s2m8D9k6g2)9J5k6h3y4G2L8g2)9J5c8Y4k6A6k6r3g2G2M7#2)9J5c8Y4m8D9j5i4W2Q4x3V1k6%4N6$3c8U0x3U0l9I4x3#2)9J5c8U0M7I4x3q4)9J5c8R3`.`.</mark>，
