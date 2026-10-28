
像上面提到的大多数其他技术一样，恶意软件需要触发它编写的代码。在以前讨论的技术中，恶意软件通过调用API（如`CreateRemoteThread`，`QueueUserAPC`或`SetThreadContext`）来实现这一点。在EWMI方法中，恶意软件通过调用`SendNotifyMessage`触发注入的代码。在执行`SendNotifyMessage`之后，`Shell_TrayWnd`接收并将控件传递给由`SetWindowLong`先前设置的值指向的地址。在图13中，名为PowerLoader的恶意软件使用这种技术。

### 9. SHIMS注入

Microsoft向开发人员提供了`Shims`[译者注：Shim是一个工程术语，描述为了让两个物体更好地组装在一起而插入的一块木头或金属。在计算机编程中，shim是一个小型的函数库，用于透明地拦截API调用，修改传递的参数、自身处理操作、或把操作重定向到其他地方。Shim也可以用来在不同的软件平台上运行程序。]，主要是为了向后兼容。`Shims`允许开发人员将修补程序应用于程序，而无需重写代码。通过利用`Shims`，开发人员可以告诉操作系统如何处理其应用程序。`Shims`本质上是一种嵌入API并针对特定可执行文件的方式。恶意软件可以利用`Shims`来实现注入可执行文件并维持注入。Windows运行`Shim`引擎时，它加载二进制文件以检查`shimming`数据库，以便应用适当的修补程序。

有许多可以使用的修复程序，但是恶意软件还是更偏爱那些安全相关的（例如`DisableNX`，`DisableSEH`，`InjectDLL`等）。要安装shimming数据库，恶意软件可以使用各种方法。例如，一个常见的方法是简单执行`sdbinst.exe`，并将其指向恶意的`sdb`文件。在图14中，广告软件“Search Protect by Conduit”使用`Shims`进行注入和维持。它在Google Chrome中执行`InjectDLL` shim以加载`vc32loader.dll`。现在有一些用于分析`sdb`文件的工具，但是对于下面列出的`sdb`的分析，我使用了`python-sdb`，而没有使用现成的工具。

![](https://bbs.kanxue.com/upload/attach/201708/740325_k3n87uh09ckjsft.png)

### 图14 Search Protect用于注入的SDB

### 10. IAT HOOKING和INLINE HOOKING (又名应用层ROOTKITS)

`IAT hooking`以及`inline hooking`通常被称为用户级rootkit. 恶意软件可以利用`IAT HOOKING`技术更改导入地址表。当合法应用程序调用位于`DLL`中的`API`时，将执行替换的函数，而不是原始代码。相比之下，使用`Inline Hooking`，恶意软件可以修改API函数本身。在图15中，恶意软件FinFisher通过修改`CreateWindowEx`指向的位置执行`IAT HOOKING`。

### 图15 FinFisher通过更改CreateWindowEx指向的位置实现IAT HOOKING

### 总结

在这篇文章中，我介绍了恶意软件在另一个进程中隐藏其活动的十种不同技术。通常，恶意软件直接将其shellcode注入到另一个进程中，或者强制另一个进程加载其恶意`DLL`。在表1中，我对各种技术进行了分类，并提供了样品，作为观察本文所涵盖的每种注射技术的参考。整篇文章中的数字将有助于研究人员在反转恶意软件时识别各种技术。

![](https://bbs.kanxue.com/upload/attach/201708/740325_2urz42ufwvjvwt5.png)

表格1: 流程注入可以通过直接将代码注入到另一个进程中，或通过强制将`DLL`加载到另一个进程来完成

攻击者和研究人员经常发现新的技术来实现注入并提供隐藏自身的目的。这篇文章详细介绍了十种常见和新兴的技术，但还有其他的没有提及，如`COMM`劫持。防御者在其使命中将永远不会“完成”检测和防止隐形进程注入的任务，因为入侵者永远不会停止创新。

在EndGame[译者注：原文发表的网站名称为EndGame]，我们不断研究先进的隐藏技术，并将这些技术带入我们的产品中，给产品提供保护。我们的层级功能可以检测加载某些维持注入（如`AppInit DLL`，`COM Hijack`等）的恶意`DLL`，通过我们的shellcode注入防护软件，可以实时防止许多形式的代码注入，并且可以检测在内存中运行的恶意注入的有效负载。我们正在申请专利的无文件攻击检测技术可以让我们的平台比市场上任何其他产品更有效地防止代码注入，同时还通过新兴的代码注入技术最大限度地提高绕过的弹性。

原文作者：

[Ashkan Hosseini](https://bbs.kanxue.com/elink@b29K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2W2L8X3c8Y4j5h3#2W2i4K6u0W2j5$3!0E0i4K6u0r3L8%4g2J5i4K6u0V1k6i4S2H3k6i4u0@1M7#2)9J5c8X3q4K6K9r3E0S2L8W2)9J5k6r3S2G2M7%4y4W2K9h3&6A6)

原文连接： a29K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2W2L8X3c8Y4j5h3#2W2i4K6u0W2j5$3!0E0i4K6u0r3j5X3I4G2k6#2)9J5c8Y4c8W2j5$3S2F1K9h3y4S2L8q4)9J5k6r3u0D9L8$3N6Q4x3V1k6@1k6h3&6Q4x3X3c8H3M7X3!0U0k6i4y4K6i4K6u0V1K9h3&6B7k6h3y4@1K9h3!0F1i4K6u0V1N6r3g2U0K9r3&6A6M7i4g2W2M7#2)9J5k6s2c8W2j5$3S2F1K9h3y4S2L8q4)9J5k6s2y4#2M7Y4k6W2P5g2)9J5k6r3y4G2L8h3#2G2L8W2)9J5k6r3q4F1k6q4)9J5k6s2c8J5k6h3&6V1K9h3&6Y4i4K6u0V1M7s2u0G2j5$3g2K6M7H3`.`.

### 译者总结

一、关于注入的知识，这几天也有翻译组的小伙伴发了一篇关于原理方面的文章写的很好。大家想要学习注入相关知识可以参考，阅读。

二、文中有一些词汇，在计算机类中应该是运用的引申义，而我又确实没有找到比较合适的特定词汇翻译，所以就在文中直接使用的英文 没有强行翻译，希望大家见谅。

三、翻译过程中，遇到了不少新鲜的知识点，也翻阅了一点博客文章，挑选其中比较关键的几个放在最后，大家可以作为扩展阅读。

四、由于个人水平有限，难免有些地方翻译的地方加入了个人的臆测，希望各位大佬可以给出建设性意见，欢迎批评指正。

### 相关阅读

[52dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2W2L8X3c8Y4j5h3#2W2i4K6u0W2j5$3!0E0i4K6u0r3j5X3I4G2k6#2)9J5c8Y4c8W2j5$3S2F1K9h3y4S2L8q4)9J5k6r3u0D9L8$3N6Q4x3V1k6V1M7X3!0H3M7r3W2F1k6#2)9J5k6r3q4@1L8$3#2T1L8$3#2T1M7#2)9J5k6r3c8W2N6r3g2U0N6r3W2F1k6#2)9J5k6r3c8J5K9h3c8W2P5s2j5@1i4K6u0V1N6$3W2D9k6l9`.`.](https://bbs.kanxue.com/elink@622K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2W2L8X3c8Y4j5h3#2W2i4K6u0W2j5$3!0E0i4K6u0r3j5X3I4G2k6#2)9J5c8Y4c8W2j5$3S2F1K9h3y4S2L8q4)9J5k6r3u0D9L8$3N6Q4x3V1k6V1M7X3!0H3M7r3W2F1k6#2)9J5k6r3q4@1L8$3#2T1L8$3#2T1M7#2)9J5k6r3c8W2N6r3g2U0N6r3W2F1k6#2)9J5k6r3c8J5K9h3c8W2P5s2j5@1i4K6u0V1N6$3W2D9k6l9`.`.)

[8c9K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6Q4x3X3g2U0M7$3c8F1i4K6u0W2L8X3g2@1i4K6u0r3M7i4q4Q4y4h3j5J5x3U0b7J5x3K6j5#2z5g2)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3y4e0x3@1x3U0f1^5x3U0R3`.](https://bbs.kanxue.com/elink@e7dK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6Q4x3X3g2U0M7$3c8F1i4K6u0W2L8X3g2@1i4K6u0r3M7i4q4Q4y4h3j5J5x3U0b7J5x3K6j5#2z5g2)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3y4e0x3@1x3U0f1^5x3U0R3`.)

[3a8K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0G2j5X3q4G2i4K6u0W2x3K6j5H3i4K6u0W2j5$3&6Q4x3V1k6D9k6h3q4J5L8X3W2F1k6#2)9J5c8X3c8W2N6r3q4A6L8q4)9J5c8U0x3I4y4o6S2Q4x3X3g2Z5N6r3#2D9](https://bbs.kanxue.com/elink@cefK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0G2j5X3q4G2i4K6u0W2x3K6j5H3i4K6u0W2j5$3&6Q4x3V1k6D9k6h3q4J5L8X3W2F1k6#2)9J5c8X3c8W2N6r3q4A6L8q4)9J5c8U0x3I4y4o6S2Q4x3X3g2Z5N6r3#2D9)

[b6dK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3k6J5k6h3g2T1N6h3k6Q4x3X3g2U0L8$3#2Q4x3V1k6F1k6i4N6K6i4K6u0r3y4o6R3^5y4K6S2Q4x3X3g2Z5N6r3#2D9](https://bbs.kanxue.com/elink@737K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3k6J5k6h3g2T1N6h3k6Q4x3X3g2U0L8$3#2Q4x3V1k6F1k6i4N6K6i4K6u0r3y4o6R3^5y4K6S2Q4x3X3g2Z5N6r3#2D9)

[963K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3k6J5k6h3g2T1N6h3k6Q4x3X3g2U0L8$3#2Q4x3V1k6S2M7Y4c8A6j5$3I4W2M7#2)9J5c8Y4y4&6M7%4c8W2L8g2)9J5c8U0p5I4y4o6t1^5y4#2)9J5k6h3S2@1L8h3H3`.](https://bbs.kanxue.com/elink@a4cK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3k6J5k6h3g2T1N6h3k6Q4x3X3g2U0L8$3#2Q4x3V1k6S2M7Y4c8A6j5$3I4W2M7#2)9J5c8Y4y4&6M7%4c8W2L8g2)9J5c8U0p5I4y4o6t1^5y4#2)9J5k6h3S2@1L8h3H3`.)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
