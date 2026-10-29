
# [翻译]滥用User DPAPI进行攻击的操作指南

原文：

670K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3L8%4y4@1M7#2)9J5k6i4y4H3k6h3y4@1k6i4u0G2M7s2y4Q4x3X3g2A6L8#2)9J5c8X3!0H3k6i4u0S2N6r3W2G2L8X3q4D9i4K6u0V1k6%4g2A6k6r3q4F1j5$3g2Q4x3X3c8X3L8%4u0Q4x3X3c8G2k6X3k6W2L8Y4y4A6N6X3g2Q4x3X3c8#2M7$3g2J5i4K6u0V1k6s2m8S2M7r3W2Q4x3X3c8S2j5Y4g2K6k6g2)9J5k6o6q4X3j5U0N6X3j5h3x3^5j5U0p5H3y4H3`.`.

在之前我已经讨论过一些有关DPAPI（数据保护应用编程接口）的内容，其中包括KeePass如何在“Windows User Account”的key参数选项中使用DPAPI。最近我进行了一些有趣的工作，这些工作Benjamin Delpy[已经进行过](https://bbs.kanxue.com/elink@684K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1N6$3W2@1N6r3g2J5i4K6u0W2j5$3!0E0i4K6u0r3k6$3g2F1N6r3W2D9K9$3W2%4K9b7`.`.)，我想在此记录一些有关结合Mimikatz滥用DPAPI的方法。

注：本文关注对用户DPAPI的滥用，但有时也会涉及对机器DPAPI key的滥用。如果在之后我把滥用机器DPAPI搞清楚了，会随后发表一篇文章。

另注：我不是最初提出这些滥用的人，也没有写一些工具来实际滥用它们。这些思想都来自Benjamin和其它本文引用的人。我只是简单地整理各种滥用样例、语法，形成一个操作指南。

# DPAPI速成课

我没有在文章中包含很多DPAPI背景知识，其它文章已经介绍地很好了：

- Benjamin的wiki（e24K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6Y4k6h3&6@1K9h3I4C8K9i4N6A6i4K6u0r3L8h3W2E0K9h3E0S2N6s2A6Q4x3V1k6%4K9h3E0A6i4K6u0r3L8h3!0V1N6h3I4W2i4K6u0V1i4K6N6q4i4K6u0V1k6s2m8S2M7r3W2Q4c8f1k6Q4b7V1y4Q4z5p5y4Z5N6s2c8H3M7#2)9K6b7g2)9J5c8W2)9J5c8X3N6A6N6r3S2#2j5W2)9J5k6h3y4G2L8g2)9J5c8X3N6W2L8Y4c8A6L8r3E0A6N6$3W2Q4x3V1k6E0K9h3#2A6K9$3q4@1P5W2)9J5c8Y4N6A6K9$3W2Q4x3V1k6Z5L8%4N6@1L8#2)9J5k6q4)9%4c8g2)9J5k6r3y4J5k6h3c8W2L8Y4c8A6j5h3I4Q4x3X3c8E0j5h3&6S2k6$3g2J5i4K6u0V1M7$3q4$3k6h3c8Q4x3X3c8U0M7X3g2V1k6h3&6@1K9h3q4D9M7#2!0q4y4g2)9&6x3W2)9^5b7$3S2@1N6s2m8K6i4K6y4m8i4K6u0r3i4K6u0r3k6$3W2@1K9s2g2T1i4K6u0W2j5$3!0E0i4K6u0r3k6$3g2F1N6r3W2D9K9$3W2%4K9g2)9J5c8X3#2A6L8h3W2C8j5i4c8\*7i4K6u0r3N6$3W2C8K9g2)9J5c8X3S2G2N6%4c8G2i4K6u0V1i4K6N6q4i4K6u0V1k6r3g2U0M7Y4W2H3N6q4)9J5k6p5g2r3f1#2)9J5k6r3k6A6L8r3g2K6 ），还有其它各种DPAPI相关的tweet

- Bartosz Inglot在OPCDE 2017的演讲[“DPAPI的黑盒”](https://bbs.kanxue.com/elink@63bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6U0L8$3#2S2k6h3W2G2i4K6u0r3e0#2m8o6c8p5g2Q4x3V1k6T1L8r3!0T1i4K6u0r3L8h3q4K6N6r3g2J5i4K6u0r3x3U0l9I4y4#2)9J5c8W2c8Z5k6g2)9J5y4e0t1H3b7X3I4S2j5$3E0T1L8%4S2Q4x3U0f1J5x3r3!0X3i4K6t1#2x3U0m8p5f1p5q4b7d9g2)9J5y4e0t1H3N6r3S2W2i4K6t1#2x3U0m8Y4K9h3k6@1i4K6t1#2x3U0m8@1K9r3q4@1i4K6t1#2x3U0m8C8k6h3g2H3M7#2)9J5y4e0t1H3L8$3&6Q4x3U0f1J5x3r3N6A6N6X3W2F1k6#2)9J5y4e0t1H3i4K6u0V1i4K6t1#2x3U0m8n7j5i4u0@1L8%4y4*7i4K6t1#2x3U0m8u0L8X3N6D9L8%4c8Q4x3V1k6f1K9r3g2Q4x3U0f1J5x3p5u0D9j5h3y4C8j5X3!0^5i4K6t1#2x3U0m8G2k6W2)9J5y4e0t1H3c8q4m8m8f1p5W2Q4x3U0f1J5x3q4)9J5k6q4)9J5y4e0t1H3b7X3q4J5N6q4)9J5y4e0t1H3d9h3&6Y4L8r3!0@1i4K6u0W2M7r3c8X3)

- Black Hat Europe 2017上Paula J的报告[“DPAPI and DPAPI-NG”](https://bbs.kanxue.com/elink@e5aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2K6L8r3W2V1k6i4y4Z5j5i4u0W2i4K6u0W2L8X3g2@1i4K6u0r3M7r3q4#2L8r3q4B7j5h3&6#2M7%4A6C8K9h3g2%4K9h3y4*7i4K6u0r3j5X3I4S2j5$3E0Q4x3X3c8Z5j5i4c8Q4x3X3c8W2N6i4u0G2M7r3g2Q4x3X3b7J5x3o6p5%4i4K6u0V1k6s2m8S2M7r3W2Q4x3X3c8S2L8X3c8Q4x3X3c8V1M7r3q4H3K9h3&6Y4i4K6u0V1k6r3g2U0M7Y4W2H3N6r3W2G2L8W2)9J5k6s2c8G2L8$3I4C8K9i4b7`.)

- “[DPAPI在渗透测试和密码破解中的利用](https://bbs.kanxue.com/elink@d99K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2K6P5h3&6S2j5$3E0@1K9i4k6Q4x3X3g2U0L8$3#2Q4x3V1k6J5k6i4y4K6L8%4g2J5j5$3g2K6i4K6u0r3N6h3&6A6N6X3g2J5M7$3S2W2L8r3I4Q4y4h3j5J5x3o6p5%4i4K6g2X3k6s2m8S2M7r3W2Q4x3X3g2H3k6r3j5`.)”，作者Jean-Christophe Delaunay

- “[给我密码我将控制整个世界](https://bbs.kanxue.com/elink@a5fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1K9h3N6A6N6r3q4D9i4K6u0V1k6X3!0J5k6h3&6K6K9h3y4K6i4K6u0W2M7$3q4F1M7#2)9J5k6h3!0J5k6#2)9J5c8Y4y4#2L8h3#2A6N6q4)9J5k6r3q4J5j5$3S2A6N6X3g2K6i4K6u0r3k6r3k6A6M7Y4m8J5j5h3N6#2k6e0p5@1i4K6u0r3c8$3W2$3k6g2)9#2k6V1#2W2i4K6g2X3N6r3S2W2i4K6g2X3f1r3q4K6M7%4N6G2M7X3c8Q4y4h3k6S2L8X3c8Q4y4h3k6u0L8r3I4Q4y4h3k6d9N6h3I4W2i4K6g2X3N6r3S2W2i4K6g2X3g2$3!0J5L8r3c8Q4y4h3k6r3M7X3q4F1j5$3g2K6j5$3!0Q4y4h3k6b7K9h3y4S2M7%4y4G2i4K6u0W2M7r3c8X3)”，作者Francesco Picasso，还有他的演讲[“ReVaulting”](https://bbs.kanxue.com/elink@d68K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2K6L8r3W2V1k6i4y4Z5j5i4u0W2i4K6u0W2L8X3g2@1i4K6u0r3M7X3g2S2L8r3W2@1P5h3&6W2N6q4)9J5c8Y4u0W2N6X3q4#2L8s2c8A6L8X3N6Q4x3X3c8V1k6h3y4J5P5i4m8@1K9h3!0F1i4K6u0V1j5h3&6V1i4K6u0V1L8%4m8H3L8%4u0@1N6h3&6A6N6r3W2W2M7H3`.`.)

- Itai Grady的“[在域环境保护浏览器的隐私](https://bbs.kanxue.com/elink@e4cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2K6L8r3W2V1k6i4y4Z5j5i4u0W2i4K6u0W2L8X3g2@1i4K6u0r3d9i4c8S2K9f1N6J5j5h3c8&6i4K6u0r3M7s2u0G2N6r3g2U0N6r3W2F1k6#2)9J5k6r3u0J5L8%4N6K6k6i4u0K6i4K6u0V1M7$3g2U0M7X3g2@1M7#2)9J5k6r3W2F1i4K6u0V1j5h3c8G2L8h3q4A6L8X3g2F1N6X3W2J5L8$3&6E0k6h3&6@1)”演讲

- Jean-Michel Picod和Elie Bursztein的“[解密DPAPI数据](https://bbs.kanxue.com/elink@a7fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8r3W2W2i4K6u0W2L8X3g2@1i4K6u0r3M7%4c8S2N6r3W2U0i4K6u0r3k6X3W2D9k6i4y4Q4x3V1k6J5k6i4k6W2M7Y4y4A6L8X3N6Q4x3X3c8V1M7r3q4H3K9g2)9J5k6r3q4F1k6q4)9J5k6s2y4@1k6h3q4D9K9h3&6Y4i4K6u0V1N6$3W2F1k6r3!0%4M7#2)9J5k6s2y4W2j5%4u0W2N6s2y4Q4x3X3c8G2k6X3k6D9K9h3&6W2i4K6u0r3M7X3g2$3k6i4u0K6K9h3&6Y4i4K6u0V1k6s2m8S2M7r3W2Q4x3X3c8S2L8X3c8Q4x3X3c8K6N6r3g2S2L8r3W2F1k6#2)9J5k6s2N6A6L8X3c8G2N6%4y4Q4x3X3c8K6k6h3y4J5k6i4c8K6i4K6u0V1L8$3k6X3L8r3W2F1k6g2)9J5k6s2y4D9K9h3c8W2M7#2)9J5k6i4m8V1k6R3`.`.)”

- @\_rastamouse关于跨网段的[优秀文章](https://bbs.kanxue.com/elink@a95K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6J5j5i4y4@1j5h3#2G2N6i4y4W2i4K6u0W2L8h3g2Q4x3V1j5J5x3o6p5%4i4K6u0r3x3o6S2Q4x3V1k6B7N6h3#2H3K9h3&6Y4i4K6u0V1L8X3g2@1N6$3!0J5K9#2)9J5k6s2y4W2k6%4u0W2k6$3q4@1K9h3!0F1i4K6u0V1N6$3W2@1K9q4)9J5k6s2u0V1M7q4)9J5c8R3`.`.)，里面有一节讲使用Mimikatz解密DPAPI加密的RDP证书blob。

我确实丢掉了一些有趣的内容，但上面的资料是我了解DPAPI如何工作以及有哪些潜在滥用情况所阅读的资料。

DPAPI提供了一个简单的API集合，使用与用户绑定或与系统绑定的隐含key来透明地加密（`CryptProtectData()`）和解密（`CryptUnprotectData()`）数据blob。这使得应用程序保护用户数据时不用担心key管理等工作。有许多事情要用到DPAPI，本文我只关注Chrome的Cookies和登录数据、Windows证书管理器（如保存IE或Edge登录密码和文件共享（RDP）密码）、远程桌面连接管理器的.rdg文件。

在顶层从用户角度看，用户的密码被用于产生一组用户特有的“Master Key”。这些Key的位置在`C:\Users\<USER>\AppData\Roaming\Microsoft\Protect\<SID>\<GUID>`，`<SID>`是用户的安全描述符，`<GUID>`是Master Key的名字。一个用户可以有多个Master Key。Master Key需要使用用户的密码或域备份key（见第四部分，Chrome）来解密，解密后的Master Key用于解密DPAPI blob。

所以，如果我们试图解密某个用户的被加密的DPAPI blob（如Chrome cookie值），我们需要关注该用户的Master Key。

# Chrome

Chrome使用DPAPI保存两块我们感兴趣的信息：cookie值和登录数据：

- Cookie文件的位置：`%localappdata%\Google\Chrome\User Data\Default\Cookies`

- 登录数据的位置：`%localappdata%\Google\Chrome\User Data\Default\Login Data`

`%localappdata%`在大多数系统对应于`“C:\Users\<USER>\AppData\Local”`。同时，这一节的所有Mimikatz命令对Cookie文件和登录数据文件都有效。

Chrome在一个SQLite数据库中存储cookies，存储的cookie值被加密为DPAPI blob来进行保护。幸运的是，Benjamin在Mimikatz中实现了Chrome的SQLite数据库解析。要列出当前用户的有效cookie值，使用下面Mimikatz命令：

```
mimikatz dpapi::chrome /in:"%localappdata%\Google\Chrome\User Data\Default\Cookies"
```

然而，cookie的值被DPAPI使用用户的master key加密过，master key则被用户的密码保护（或域备份key）。在某些情况下我们能够获得这些cookie值（或登录数据）：

## 情况一：在目标用户上下文执行代码

这是最简单的情况。如果你想在用户上下文执行Beacon、Mimikatz或其它可执行代码，只需在`dpapi::chrome`命令中增加`/unprotect`标志：

该命令只是通知Mimikatz使用`CryptUnprotectData` API来解密我们想要的cookie值。如果我们在用户上下文执行该命令，将默认使用他们的master key执行请求的解密操作。

注：如果在Chrome使用该代码，有时会遇到打开Cookies数据库失败的问题。这时，只需复制Cookie或登录数据文件到当前的操作目录并使用新路径运行`dpapi::chrome`命令即可。

## 情况二：以管理员权限访问目标机器，且目标用户当前已登录

如果你不想在另一个用户的上下文注入一个Beacon，或者你登录了一个当前已有多个用户登录的系统，你可以选择做一些事。

如果你对属于另一个用户的数据库执行`/unprotect`操作，调用`CryptUnprotectData()`进行解密将得到一个错误。较新版本的Mimikatz能识别出解密所需的master key的GUID（在Cobalt Strike更新Mimikatz，就会在输出中看到），形如：`needed masker key is {b8854128-023c-433d-aac9-232b4bca414c}`。

我们可以看到，这个master key是`harmj0y`的Chrome Cookies目录的位置。我们同样可以通过列出用户目录(`C:\Users\<USER>\AppData\Roaming\Microsoft\Protect\<SID>\<GUID>`)内的所有makter key的GUID来得到用户的所有master key，以此来得到所需master key的GUID。如何对每个用户进行该操作，见Seatbelt节。

我们需要想办法获取`harmj0y`的master key。一个选择是运行`sekurlsa::dpapi`从内存提取所有系统内当前登录用户的DPAPI key（有时也在`sekurlsa::msv`命令的输出中显示）。
