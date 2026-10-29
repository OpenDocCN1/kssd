
使用 `drozer` 的优势在于它能够自动执行多个任务，并且可以通过模块进行扩展。这些模块非常有帮助并且涵盖了不同的类别，其中包括扫描器类别，该类别使你可以使用简单的命令扫描已知的缺陷，例如模块 `scanner.provider.injection` 可以检测系统中安装的所有应用程序的内容提供者中的 SQL 注入 。 如果不使用 `drozer`，则简单的任务（例如列出应用程序的权限）需要几个步骤，包括反编译 APK 和手动分析结果。

### 安装 Drozer

你可以参考 [drozer GitHub 页面](https://bbs.kanxue.com/elink@425K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6r3f1$3g2U0N6i4u0W2e0p5q4n7f1#2)9J5c8X3c8J5L8%4A6W2M7R3`.`.)（对于 Linux 和 Windows，macOS 请参考[这篇博客文章](https://bbs.kanxue.com/elink@2cdK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2M7X3!0H3L8X3!0H3i4K6u0W2j5$3!0E0i4K6u0r3K9h3&6K6N6r3q4D9L8r3W2F1k6#2)9J5k6r3c8J5L8%4A6W2M7W2)9J5k6r3!0F1i4K6u0V1L8%4y4Q4x3X3c8^5i4K6u0V1k6h3I4Q4x3X3c8U0j5i4m8A6N6r3q4F1i4K6u0r3)）和 [drozer 的网站](https://bbs.kanxue.com/elink@e5eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9j5h3u0K6i4K6u0W2k6W2)9J5k6s2y4W2j5%4g2J5k6g2)9J5k6h3y4G2L8g2)9J5c8Y4c8G2L8$3I4K6i4K6u0r3k6s2u0G2P5X3g2J5i4K6u0r3)了解必备条件和安装说明。

`drozer` 在 Unix、Linux 和 Windows 上的安装说明在 [drozer Github 页面](https://bbs.kanxue.com/elink@99dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6r3f1$3g2U0N6i4u0W2e0p5q4n7f1#2)9J5c8X3c8J5L8%4A6W2M7R3`.`.)中有解释。对于macOS，[这篇博客](https://bbs.kanxue.com/elink@2deK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6r3f1$3g2U0N6i4u0W2e0p5q4n7f1#2)9J5c8X3c8J5L8%4A6W2M7R3`.`.)演示了所有的安装说明。

### 使用 Drozer

在开始使用 `drozer` 之前，还需要在 Android设备上运行 `drozer` 代理。从 [GitHub 发布页面](https://bbs.kanxue.com/elink@fffK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6r3f1$3g2U0N6i4u0W2e0p5q4n7f1#2)9J5c8X3c8J5L8%4A6W2M7W2)9J5c8Y4u0W2L8r3g2S2M7$3g2K6i4K6u0r3)下载最新的 `drozer` 代理，并用 `adb install drozer.apk` 安装它。

一旦安装完成，你可以通过运行 `adb forward tcp:31415 tcp:31415` 和 `drozer console connect` 启动一个会话到模拟器或 USB 连接的设备。这被称为直接模式，你可以在[用户指南的“开始一个会话”](https://bbs.kanxue.com/elink@412K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9j5h3u0K6i4K6u0W2k6W2)9J5k6s2y4W2j5%4g2J5k6g2)9J5k6h3y4G2L8g2)9J5c8X3q4K6M7$3g2@1M7#2)9J5c8V1u0D9L8$3N6r3K9h3I4W2M7#2)9J5c8X3#2%4M7X3W2Q4x3X3c8V1M7X3!0*7k6i4u0Q4x3X3c8#2M7$3g2J5i4K6u0V1k6%4g2A6k6r3g2Q4x3X3b7J5x3o6p5#2i4K6u0V1x3o6y4Q4x3X3b7J5x3#2)9J5k6i4m8V1k6R3`.`.)一节中看到完整的说明。另一种选择是在基础设施模式下运行Drozer，在这种模式下，你运行一个 Drozer 服务器，它可以处理多个控制台和代理，并在它们之间路由会话。你可以在用户指南的[“基础设施模式”](https://bbs.kanxue.com/elink@cdcK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9j5h3u0K6i4K6u0W2k6W2)9J5k6s2y4W2j5%4g2J5k6g2)9J5k6h3y4G2L8g2)9J5c8X3q4K6M7$3g2@1M7#2)9J5c8V1u0D9L8$3N6r3K9h3I4W2M7#2)9J5c8X3#2%4M7X3W2Q4x3X3c8V1M7X3!0*7k6i4u0Q4x3X3c8#2M7$3g2J5i4K6u0V1k6%4g2A6k6r3g2Q4x3X3b7J5x3o6p5#2i4K6u0V1x3o6y4Q4x3X3b7J5x3#2)9J5k6i4m8V1k6R3`.`.)一节中找到如何在此模式下设置 `drozer` 的详细信息。

现在您可以开始分析应用程序了。一个好的开端是列举一个应用程序的攻击面，可以通过以下命令很容易地完成：
```
$ dz> run app.package.attacksurface <package>
```
同样，如果没有 `drozer`，这将需要几个步骤。 `app.package.attacksurface` 模块列出了活动，广播接收者，内容提供者和导出的服务，因此它们是公共的并且可以通过其他应用程序进行访问。 一旦确定了攻击面，就可以通过 `drozer` 与 IPC 端点进行交互，而无需编写单独的独立应用程序，某些任务会需要它，比如与内容提供者通信。

例如，如果应用程序的导出活动泄漏了敏感信息，我们可以使用 Drozer 模块 `app.activity.start` 来调用它：
```
$ dz> run app.activity.start --component <package> <component name>
```
前面的命令将启动活动，希望泄漏一些敏感信息。Drozer 有针对每一种 IPC 机制的模块，如果你想尝试带有故意易受攻击的应用程序的模块，请下载 [InsecureBankv2](https://bbs.kanxue.com/elink@d55K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6V1K9h3&6W2M7$3S2K6K9r3g2@1N6s2W2Q4x3V1k6m8L8X3c8J5L8$3W2V1i4K6u0V1d9h3&6K6k6h3y4#2M7X3g2n7j5h3&6C8N6U0t1`.)，该应用程序演示了与 IPC 端点相关的常见问题。请密切注意扫描器类别中的模块，因为它们对于自动检测系统包中的漏洞非常有用，特别是如果你使用的是手机公司提供的 ROM。在过去甚至使用 `drozer` 识别出过 Google 系统软件包中的 SQL 注入漏洞。

### 其他的 Drozer 命令

这里有一个非详尽的命令列表，你可以用来开始探索 Android：
```
# List all the installed packages
$ dz> run app.package.list
```
```
# Find the package name of a specific app
$ dz> run app.package.list –f (string to be searched)
```
```
# See basic information
$ dz> run app.package.info –a (package name)
```
```
# Identify the exported application components
$ dz> run app.package.attacksurface (package name)
```
```
# Identify the list of exported Activities
$ dz> run app.activity.info -a (package name)
```
```
# Launch the exported Activities
$ dz> run app.activity.start --component (package name) (component name)
```
```
# Identify the list of exported Broadcast receivers
$ dz> run app.broadcast.info -a (package name)
```
```
# Send a message to a Broadcast receiver
$ dz> run app.broadcast.send --action (broadcast receiver name) -- extra (number of arguments)
```
```
# Detect SQL injections in content providers
$ dz> run scanner.provider.injection -a (package name)
```

### 其他的 Drozer 资源

你可能会发现其他有用信息的资源有：

- [Official drozer User Guide](https://bbs.kanxue.com/elink@e53K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9j5h3u0K6i4K6u0W2k6W2)9J5k6s2y4W2j5%4g2J5k6g2)9J5k6h3y4G2L8g2)9J5c8X3q4K6M7$3g2@1M7#2)9J5c8V1u0D9L8$3N6r3K9h3I4W2M7#2)9J5c8X3#2%4M7X3W2Q4x3X3c8V1M7X3!0*7k6i4u0Q4x3X3c8#2M7$3g2J5i4K6u0V1k6%4g2A6k6r3g2Q4x3X3b7J5x3o6p5#2i4K6u0V1x3o6y4Q4x3X3b7J5x3#2)9J5k6i4m8V1k6R3`.`.).
- [drozer GitHub page](https://bbs.kanxue.com/elink@e2aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6r3f1$3g2U0N6i4u0W2e0p5q4n7f1#2)9J5c8X3c8J5L8%4A6W2M7R3`.`. )
- [drozer Wiki](https://bbs.kanxue.com/elink@81fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6r3f1$3g2U0N6i4u0W2e0p5q4n7f1#2)9J5c8X3c8J5L8%4A6W2M7W2)9J5c8Y4N6A6K9$3V1`.)

#### Frida

Frida 是一个免费和开源的动态代码工具包，它允许你在本地应用程序中执行 JavaScript 脚本，在通用测试指南的“篡改与逆向工程”一章中已经介绍了。

Frida 通过 [Java API](https://bbs.kanxue.com/elink@7dfK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6X3M7X3W2V1j5g2)9J5k6i4u0W2i4K6u0r3k6r3!0U0M7#2)9J5c8X3A6S2N6X3q4K6j5%4u0A6M7s2c8Q4x3X3c8S2M7r3W2Q4x3V1k6Q4x3U0y4B7j5i4k6S2) 支持与 Android Java 运行时的交互。你能够在hook 和调用进程的 Java 和原生函数，以及它的原生库。你的 JavaScript 脚本对内存有完全的访问权限，例如读取或写入任何结构化数据。

下面是 Frida api 提供的一些任务，在 Android 上是相关的或独家的:

- 实例化 Java 对象并调用静态和非静态类方法（[Java API](https://bbs.kanxue.com/elink@829K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6X3M7X3W2V1j5g2)9J5k6i4u0W2i4K6u0r3k6r3!0U0M7#2)9J5c8X3A6S2N6X3q4K6j5%4u0A6M7s2c8Q4x3X3c8S2M7r3W2Q4x3V1k6Q4x3U0y4B7j5i4k6S2) ）。
- 替换Java方法实现（[Java API](https://bbs.kanxue.com/elink@9a3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6X3M7X3W2V1j5g2)9J5k6i4u0W2i4K6u0r3k6r3!0U0M7#2)9J5c8X3A6S2N6X3q4K6j5%4u0A6M7s2c8Q4x3X3c8S2M7r3W2Q4x3V1k6Q4x3U0y4B7j5i4k6S2) ）。
- 通过扫描 Java 堆枚举特定类的活动实例（[Java API](https://bbs.kanxue.com/elink@6b6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6X3M7X3W2V1j5g2)9J5k6i4u0W2i4K6u0r3k6r3!0U0M7#2)9J5c8X3A6S2N6X3q4K6j5%4u0A6M7s2c8Q4x3X3c8S2M7r3W2Q4x3V1k6Q4x3U0y4B7j5i4k6S2) ）。
- 扫描进程内存中出现的字符串（[Memory API](https://bbs.kanxue.com/elink@888K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6X3M7X3W2V1j5g2)9J5k6i4u0W2i4K6u0r3k6r3!0U0M7#2)9J5c8X3A6S2N6X3q4K6j5%4u0A6M7s2c8Q4x3X3c8S2M7r3W2Q4x3V1k6Q4x3U0y4E0k6h3#2G2M7Y4V1`.) ）。
- 拦截原生函数调用以在函数入口和出口运行自己的代码（[Interceptor API](https://bbs.kanxue.com/elink@2c2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6X3M7X3W2V1j5g2)9J5k6i4u0W2i4K6u0r3k6r3!0U0M7#2)9J5c8X3A6S2N6X3q4K6j5%4u0A6M7s2c8Q4x3X3c8S2M7r3W2Q4x3V1k6Q4x3U0y4A6L8Y4c8W2M7X3y4W2M7s2c8G2M7R3`.`.)）。

记住，在Android上，你还可以从安装 Frida 时提供的内置工具中获益，包括 Frida CLI（`frida`）、`frida-ps`、`frida-ls-devices` 和 `frida-trace`。

Frida 经常被拿来和 Xposed 进行比较，但是这种比较并不公平，因为这两个框架的设计目标是不同的。作为一个应用程序安全测试人员，了解这一点很重要，这样你就可以知道在什么情况下使用哪个框架：

- Frida 是独立的，你所需要做的就是从目标 Android 设备中的已知位置运行 `frida-server` 二进制文件（请参阅下面的“安装Frida”）。这意味着，与 Xposed 相比，它没有深入安装在目标 OS 中。
- 逆向一个应用程序是一个反复的过程。由于上一点的影响，在测试时，你获得了更短的反馈循环，因不需要（软）重启来应用钩子或简单地更新钩子。同时在实现更持久的钩子时，你可能更喜欢使用 Xposed。
- 你可以在进程运行期间的任何时候动态注入和更新 Frida JavaScript 代码（类似于 iOS 上的 Cycript）。这样，可以通过让 Frida 来 spwan 你的应用程序执行所谓的早期检测，或者你可能更喜欢附加到一个进入特定状态的正在运行的应用程序。
- Frida 能够处理 Java 以及原生代码（JNI），允许你修改它们。不幸的是，这正是 Xposed 的局限性，缺乏原生代码支持。

> 值得注意的是，到 2019 年初，Xposed 还不能在 Android 9（API 等级28）上运行。

##### 安装 Frida

要在本地安装 Frida，只需运行：
```
$ pip install frida-tools
```
参考[安装页面](https://bbs.kanxue.com/elink@a20K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6X3M7X3W2V1j5g2)9J5k6i4u0W2i4K6u0r3k6r3!0U0M7#2)9J5c8X3W2F1M7%4c8S2L8r3I4S2N6r3W2G2L8W2)9J5c8R3`.`.)了解更多细节。

下一步就是在你的 Android 设备上设置 Frida：

- 如果您的设备没有 root，你也可以使用Frida，请参考“逆向工程和篡改”章节的“非 root 设备的动态分析”。
