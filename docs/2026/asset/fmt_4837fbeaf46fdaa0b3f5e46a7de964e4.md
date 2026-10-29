
# Android 安全开发之 ZIP 文件目录遍历

## 1. ZIP 文件目录遍历简介

因为 ZIP 压缩包文件中允许存在`../`的字符串，攻击者可以利用多个`../`在解压时改变 ZIP 包中某个文件的存放位置，覆盖掉应用原有的文件。如果被覆盖掉的文件是动态链接`so`、`dex`或者`odex`文件，轻则产生本地拒绝服务漏洞，影响应用的可用性，重则可能造成任意代码执行漏洞，危害用户的设备安全和信息安全。比如近段时间发现的“寄生兽”漏洞、海豚浏览器远程命令执行漏洞、三星默认输入法远程代码执行漏洞等都与 ZIP 文件目录遍历有关。

阿里聚安全的应用漏洞扫描服务，可以检测出应用的 ZIP 文件目录遍历风险。另外我们发现日本计算机应急响应小组（JPCERT）给出的修复方案存在缺陷。如果使用不当（它提供的示例文档就使用错误），可能起不到防止 ZIP 文件目录遍历的作用，并且国内有修复方案参考了此方案。

## 2. 漏洞原理和风险示例

### 2.1 漏洞原理

在 Linux/Unix 系统中`..`代表的是向上级目录跳转，如果程序在处理到诸如用`../../../../../../../../../../../etc/hosts`表示的文件时没有进行防护，则会跳转出当前工作目录，跳转到其他目录中。

Java 代码在解压 ZIP 文件时会使用到`ZipEntry`类的`getName()`方法。如果 ZIP 文件中包含`../`的字符串，该方法返回值里面会原样返回。如果在这里没有进行防护，继续解压缩操作，就会将解压文件创建到其他目录中。

如我们构造的 ZIP 文件中有如下文件：

![](https://bbs.kanxue.com/upload/attach/201606/666663_060ee9514c2e9462.jpg)

进行解压的代码如下，没有对`getName`进行过滤：

![](https://bbs.kanxue.com/upload/attach/201606/666663_72505bef2e8c6b7f.png)

解压操作时的日志：

![](https://bbs.kanxue.com/upload/attach/201606/666663_9a0ef02b035fcfc9.png)

此 ZIP 文件存放在 SD 卡中，想让解压出来的所有文件也存在 SD 卡中，但是`a_poc.txt`文件却存在了应用的数据目录中：

![](https://bbs.kanxue.com/upload/attach/201606/666663_9e098f0902b74673.png)

### 2.2 风险示例

以海豚浏览器远程代码执行漏洞为例。

海豚浏览器的主题设置中允许用户通过网络下载新的主题进行更换，主题文件其实是一个 ZIP 压缩文件。通过中间人攻击的方法可以替换掉这个 ZIP 文件。替换后的 ZIP 文件中有重新编译过的`libdolphin.so`。此 so 文件重写了`JNI_OnLoad()`函数：

![](https://bbs.kanxue.com/upload/attach/201606/666663_3108344e3405f08c.png)

此 so 以`../../../../../../../../../../data/data/mobi.mgeek.TunnyBrowser/files/libdolphin.so`的形式存在恶意 ZIP 文件中。海豚浏览器解压恶意 ZIP 文件后，重新的`libdolphin.so`就会覆盖掉原有的 so 文件，重新运行海豚浏览器会弹出 Toast 提示框：

![](https://bbs.kanxue.com/upload/attach/201606/666663_9ce542718222f996.png)

能弹出 Toast 说明也就可以执行其他代码。

这里分析下此漏洞产生的原因是：

1. 主题文件其实是一个 ZIP 压缩包，从服务器下载后进行解压，但是解压时没有过滤`getName()`返回的字符串中是否有`../`：

   ![](https://bbs.kanxue.com/upload/attach/201606/666663_158efa148c829d18.png)

2. 动态链接库文件`libdolphin.so`，并没有放在应用数据的 lib 目录下，而是放在了 files 目录中：

   ![](https://bbs.kanxue.com/upload/attach/201606/666663_7eeaba11eb816a76.png)

加载使用的地方是`com.dolphin.browser.search.redirect`包中的`SearchRedirector`：

![](https://bbs.kanxue.com/upload/attach/201606/666663_1b164a630e738a2f.png)

应用使用的是`System.load()`来加载`libdolphin.so`而非`System.loadLibrary()`，在 Android 中，`System.loadLibrary()`是从应用的 lib 目录中加载`.so`文件，而`System.load()`是用某个`.so`文件的绝对路径加载，这个`.so`文件可以不在应用的 lib 目录中，可以在 SD 卡中，或者在应用的 files 目录中，只要应用有读的权限目录中即可。

在 files 目录中，应用具有写入权限，通过网络中间人攻击，同时利用 ZIP 文件目录遍历漏洞，替换掉文件`libdolphin.so`，达到远程命令执行的目的。

应用的 lib 目录是软链接到了`/data/app-lib/应用目录`，如果`libdolphin.so`文件在 lib 目录下就不会被覆盖了，第三方应用在执行时没有写入`/data/app-lib`目录的权限：

![](https://bbs.kanxue.com/upload/attach/201606/666663_0ca226e626007745.png)

## 3. JPCERT 修复方案的研究

在研究中我们发现 JPCERT 提供的修复方案存在缺陷。它是利用 Java 的`File`类提供的`getCanonicalPath()`方法过滤掉`zipEntry.getName()`返回的字符串中所包含的`../`，然后检查这个字符串是否是以要解压到的目标目录字符串为开头，如果是，返回`getCanonicalPath()`获取到的字符串，如果不是，则抛出异常：

![](https://bbs.kanxue.com/upload/attach/201606/666663_229ff46b8cee179d.jpg)

但是在 JPCERT 给出的示例代码中，对`validateFilename()`的调用对于 APP 来说不会达到防止任意目录遍历的目的：

![](https://bbs.kanxue.com/upload/attach/201606/666663_ef3b7db5c5a4da8d.png)

其使用`.`作为要解压到的目的目录，`.`表示当前目录，经测试 APP 进程的当前工作目录是根目录`/`：

![](https://bbs.kanxue.com/upload/attach/201606/666663_70cf9159d272a31e.png)

![](https://bbs.kanxue.com/upload/attach/201606/666663_3b21429ca4bd2337.png)

查看进程状态，得到的 APP 进程的当前工作目录 cwd 是链接到了根目录：

![](https://bbs.kanxue.com/upload/attach/201606/666663_95a6da7e4988a597.png)

如下的 Demo，如果采用 JPCERT 示例中`validateFilename(entry.genName(), ".")`的调用方式，还是会产生目录遍历读到系统配置文件：

![](https://bbs.kanxue.com/upload/attach/201606/666663_c712c32bbf41998a.png)

读到的 hosts 文件内容：

![](https://bbs.kanxue.com/upload/attach/201606/666663_ca48cfb8c11a43b3.png)

正确的调用`validateFilename()`形式是传入的要解压到的目的目录不要用`.`，而是指定一个绝对路径。

## 4. 阿里聚安全对开发者建议

1. 对重要的 ZIP 压缩包文件进行数字签名校验，校验通过才进行解压。
2. 检查 ZIP 压缩包中使用`ZipEntry.getName()`获取的文件名中是否包含`../`或者`..`，检查`../`的时候不必进行 URI Decode（以防通过 URI 编码`..%2F`来进行绕过），测试发现`ZipEntry.getName()`对于 ZIP 包中有`..%2F`的文件路径不会进行处理。
3. 在应用上线前使用阿里聚安全的安全扫描服务，尽早发现应用的安全风险。

### 阿里聚安全扫描器建议修复方案

在使用`java.util.zip`包中`ZipInputStream`类进行解压操作时，进行检查，示例如下：

![](https://bbs.kanxue.com/upload/attach/201606/666663_0b04971040f84ecc.jpg)

也可以使用`java.util.zip`包中的`ZipFile`类，直接读取 ZIP 包中的所有 entries，然后检查`getName()`的返回值是否包含`../`：

![](https://bbs.kanxue.com/upload/attach/201606/666663_0b079c2bc41117b9.jpg)

## 5. 参考

1. d73K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2B7M7r3y4W2M7Y4c8Q4x3X3g2G2M7W2)9J5k6h3A6H3i4K6u0r3M7s2u0W2M7$3g2F1N6q4)9J5c8U0t1H3x3e0c8Q4x3V1j5J5x3o6p5@1x3o6V1I4x3r3q4F1k6s2u0G2K9h3c8Q4x3X3c8K6j5#2)9J5k6i4m8V1k6W2)9J5y4X3&6T1M7%4m8Q4x3@1t1`.
2. 《海豚浏览器与水星浏览器远程代码执行漏洞详解》a13K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3c8J5L8%4m8K6i4K6u0W2N6$3!0G2P5i4g2F1i4K6u0W2L8%4u0Y4i4K6u0r3L8h3!0T1K9h3I4W2i4K6u0r3z5o6t1&6x3#2)9J5y4X3&6T1M7%4m8Q4x3@1t1`.
3. 《影响数千万 APP 的安卓 APP“寄生兽”漏洞技术分析》ed9K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3c8J5L8%4m8K6i4K6u0W2N6$3!0G2P5i4g2F1i4K6u0W2L8%4u0Y4i4K6u0r3L8h3!0T1K9h3I4W2i4K6u0r3y4U0V1I4x3q4)9J5y4X3&6T1M7%4m8Q4x3@1t1`.
4. 《三星默认输入法远程代码执行》4b3K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3c8J5L8%4m8K6i4K6u0W2N6$3!0G2P5i4g2F1i4K6u0W2L8%4u0Y4i4K6u0r3M7r3q4H3k6i4u0K6i4K6u0r3y4U0j5K6x3W2)9J5y4X3&6T1M7%4m8Q4x3@1t1`.
5. 6ddK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3!0J5j5h3y4D9k6g2)9J5k6h3y4G2L8g2)9J5c8Y4c8W2j5$3S2F1k6i4c8%4L8%4u0C8i4K6u0r3j5i4u0@1K9h3y4D9k6i4y4Q4x3V1k6B7j5i4k6S2i4K6u0r3j5$3!0E0M7s2u0W2M7%4y4Q4x3X3b7I4y4e0j5#2x3o6M7$3i4K6u0W2K9s2c8E0L8q4)9J5y4X3&6T1M7%4m8Q4x3@1t1`.
6. b7eK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4y4@1j5h3y4C8L8%4k6W2M7X3k6D9L8%4N6Q4x3X3g2U0L8$3#2Q4x3V1k6I4N6h3g2K6N6r3W2G2L8Y4y4Q4x3V1j5I4x3o6V1&6x3K6l9H3i4K6u0r3N6$3S2S2N6s2y4Q4x3X3c8@1K9r3g2Q4x3X3c8V1K9h3k6X3k6i4u0W2L8X3y4W2i4K6u0V1j5X3g2@1N6$3g2W2L8W2)9J5k6r3N6W2N6s2m8S2N6r3S2Q4x3X3c8Y4k6i4c8S2j5Y4y4G2L8s2g2@1k6i4m8S2N6r3S2Q4x3X3c8S2L8X3c8Q4x3X3c8Y4k6i4c8U0j5h3&6G2L8X3W2U0j5h3I4H3j5i4c8Z5i4K6t1$3L8X3u0K6M7q4)9K6b7R3`.

## 6. Android 安全开发系列

### 目录

- [Android 安全开发之 Provider 组件安全](https://bbs.kanxue.com/elink@f3cK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3A6S2M7g2)9J5k6h3q4D9K9h3u0S2j5X3q4Q4x3X3g2U0L8$3#2Q4x3V1k6U0L8$3#2E0N6h3&6A6N6s2W2Q4x3V1k6S2M7Y4c8Q4x3V1k6K6K9r3!0%4i4K6y4r3M7%4m8E0i4K6y4p5j5e0x3I4x3$3g2Q4x3X3f1%4z5e0p5$3y4U0b7^5i4K6u0W2x3q4)9J5k6e0m8Q4x3X3g2m8P5s2S2I4z5e0u0Q4x3U0k6S2M7Y4c8A6j5$3I4W2K9h3c8Q4x3@1b7K6y4e0t1`.)
- [Android 安全开发之浅谈密钥硬编码](https://bbs.kanxue.com/elink@9b6K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3A6S2M7g2)9J5k6h3q4D9K9h3u0S2j5X3q4Q4x3X3g2U0L8$3#2Q4x3V1k6U0L8$3#2E0N6h3&6A6N6s2W2Q4x3V1k6S2M7Y4c8Q4x3V1k6K6K9r3!0%4i4K6y4r3M7%4m8E0i4K6y4p5j5e0x3I4x3$3g2Q4x3X3f1%4z5e0p5$3y4U0b7^5i4K6u0W2x3q4)9J5k6e0m8Q4x3X3g2m8P5s2S2I4z5e0u0Q4x3U0k6S2M7Y4c8A6j5$3I4W2K9h3c8Q4x3@1b7K6x3U0p5`.)
- [Android 安全开发之浅谈网页打开 APP](https://bbs.kanxue.com/elink@306K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3A6S2M7g2)9J5k6h3q4D9K9h3u0S2j5X3q4Q4x3X3g2U0L8$3#2Q4x3V1k6U0L8$3#2E0N6h3&6A6N6s2W2Q4x3V1k6S2M7Y4c8Q4x3V1k6K6K9r3!0%4i4K6y4r3M7%4m8E0i4K6y4p5j5e0x3I4x3$3g2Q4x3X3f1%4z5e0p5$3y4U0b7^5i4K6u0W2x3q4)9J5k6e0m8Q4x3X3g2m8P5s2S2I4z5e0u0Q4x3U0k6S2M7Y4c8A6j5$3I4W2K9h3c8Q4x3@1b7J5y4U0f1`.)
- [Android 应用安全开发之浅谈加密算法的坑](https://bbs.kanxue.com/elink@400K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3A6S2M7g2)9J5k6h3q4D9K9h3u0S2j5X3q4Q4x3X3g2U0L8$3#2Q4x3V1k6U0L8$3#2E0N6h3&6A6N6s2W2Q4x3V1k6S2M7Y4c8Q4x3V1k6K6K9r3!0%4i4K6y4r3M7%4m8E0i4K6y4p5j5e0x3I4x3$3g2Q4x3X3f1%4z5e0p5$3y4U0b7^5i4K6u0W2x3q4)9J5k6e0m8Q4x3X3g2m8P5s2S2I4z5e0u0Q4x3U0k6S2M7Y4c8A6j5$3I4W2K9h3c8Q4x3@1b7J5x3o6V1`.)

作者：伊樵、呆狐@阿里聚安全，更多安全技术文章，请访问[阿里聚安全博客](https://bbs.kanxue.com/elink@4c5K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3A6S2M7g2)9J5k6h3q4D9K9h3u0S2j5X3q4Q4x3X3g2U0L8$3#2Q4x3V1k6U0L8$3#2E0N6h3&6A6N6s2W2Q4x3V1k6A6L8X3c8W2P5q4)9J5k6h3S2@1L8b7`.

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
