
# [翻译] Google Zero 团队：Android 流行通讯 APP 中的 BUG

原文地址:<mark class="encrypted">b34K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4L8$3!0Y4L8r3g2H3M7X3!0B7k6h3y4@1P5X3g2J5L8#2)9J5k6h3u0D9L8$3N6K6M7r3!0@1i4K6u0W2j5$3!0E0i4K6u0r3x3U0l9I4z5g2)9J5c8U0l9K6i4K6u0r3j5h3&6V1M7X3!0A6k6q4)9J5k6r3#2W2M7%4y4S2k6$3W2F1k6#2)9J5k6r3k6W2N6#2)9J5k6r3u0#2k6%4y4Q4x3X3c8K6K9r3!0J5N6q4)9J5k6r3!0X3i4K6u0W2K9s2c8E0L8l9`.`.</mark>

## 前言

大概一年半以前, 我对 Android 的通讯和邮件客户端进行了一些测试。当时, 在这些客户端中发现了一些 bug, 然而没能把这些 bug 组合成有效的攻击链, 所以没有将成果发到博客上。不过, 和当初想公布研究成果的想法不同, 我现在决定将研究成果分享出来。毕竟, 了解（客户端的）设计上的抉择会对安全造成什么影响, 也是一件有趣的事情。

## 实例

在研究过程中, 我对 Messages（Android 默认的短信客户端）、Android 上的 Gmail 客户端、微软 Outlook、Facebook Messenger、WhatsApp、Signal 和 Telegram 这些 Google Play 上最流行的通讯和邮件客户端进行了测试。

## 通讯客户端说明（体系结构、攻击面、测试方法）

从高层次上看, 通讯客户端具有类似的体系结构, 其功能是与服务器进行通讯, 负责定位要与用户通讯的收件人, 通常还要保留通讯过程中的 session。消息需要通过服务器, 再由服务器直接发给收件人, 支持端到端加密的客户端可以将消息用某种信封加密技术密封起来。

不同客户端使用的通讯协议是不一样的, 尽管如此, Facebook Messenger、WhatsApp 和 Telegram 还是有相似的地方的, 这些客户端通讯协议都是采用分层方法实现的, 使用该方法发送的消息可以在一个层次结构中包含类型不同的字段。Telegram 使用的<a href="elink@58cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0L8%4u0W2i4K6u0W2N6r3g2D9k6h3N6J5j5h3#2Q4x3X3g2G2M7X3N6Q4x3V1k6K6j5$3S2W2L8h3p5`.">通讯协议</a>是公开的, 用它发送出的一条消息就同时包含了发送给服务器端和接收者的信息。

一开始, 我从 Facebook Messenger 处理未加密信息的方式入手, 没想到, 这个攻击思路颇具挑战性。发送消息的协议（如 mobile 和 WEB API）有挺多可以用的, 再加上在目标设备接收到消息之前消息还会经过大量处理, 这两点大大提升了找到客户端中可攻击点的难度。因此, 我决定将重点放在加密后的消息上, 因为服务器不能过滤这些邮件。

我没有 Facebook Messenger 和 WhatsApp 的源码, 所以我只能用 `apktool` 从这些应用的 APK 文件中提取出 smali 代码来进行分析。Signal 协议的普及简化了找这些应用程序的加密部分的步骤, 因为可以直接在 `libsignal` 中搜索字符串。之后, 我添加了一个发送消息的函数，将消息发送到我的服务器上, 该函数还允许在加密之前对消息进行更改的选项, 这样我就可以用格式错误的加密消息来进行测试。

比起上面两个客户端, 测试邮件和 SMS 客户端会显得稍微容易些。在测试 SMS 客户端时, 我调用了<a href="elink@058K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2j5$3!0E0i4K6u0r3M7X3g2X3k6i4u0W2L8X3y4W2i4K6u0r3j5h3&6V1M7X3!0A6k6q4)9J5c8Y4c8W2L8r3g2H3K9r3!0F1P5g2)9J5c8W2y4E0M7@1#2S2L8X3q4Y4k6i4u0Q4x3X3g2Z5N6r3#2D9i4K6t1K6M7$3g2F1k6p5c8S2N6r3q4y4k6i4y4K6j5h3N6W2i4K6t1^5K9X3q4$3j5g2)9J5k6h3I4S2L8X3N6Q4x3X3g2e0N6s2u0A6L8X3N6Q4x3V1y4Q4x3U0f1J5x3r3A6S2N6X3q4Q4x3X3g2D9j5h3&6Y4i4K6u0W2f1%4c8J5K9h3&6Y4i4K6u0o6i4K6t1#2x3U0m8K6K9r3!0J5N6q4)9J5b7#2)9J5y4e0t1H3j5Y4W2@1k6g2)9J5y4e0g2n7i4K6t1#2y4f1c8Q4x3V1y4Q4x3U0f1J5x3r3q4F1k6s2u0G2K9h3c8Q4x3X3g2S2M7s2m8Q4x3X3g2b7k6h3&6V1K9h3&6Y4d9h3&6@1k6h3&6@1i4K6u0o6i4K6t1#2x3U0m8S2L8X3c8J5L8$3W2V1i4K6u0W2j5i4m8H3i4K6u0W2f1r3g2F1k6r3W2F1k6@1W2F1N6r3g2F1N6l9`.`."><code>sendDataMessage</code></a> API 发送原始的 SMS 消息。而在测试邮件客户端时, 可以直接使用 SMTP 服务器发送 MIME 邮件。

## 研究结果提要

有一点需要注意一下, 这些结果已超过一年, 并且通讯 app 的客户端的代码已有较大改动, 因此现在测试结果可能会有所不同。本文中提及漏洞均已修复。

### Messages

Android 默认的 Messages 客户端主要是用 Java 写的, 使用 AOSP 的 SMS 和 MMS 来解析消息。因此, Messages 几乎不存在内存损坏漏洞。不过 Android 会以不同的方式处理一些特殊的 SMS 消息, 解析这些特殊 SMS 消息有可能会产生漏洞。某些特殊的消息会被解析为符合可视化语音信箱的<a href="elink@fd8K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3!0E0N6s2m8Q4x3X3g2G2M7X3N6Q4x3V1k6a6e0g2c8b7i4K6g2X3g2W2k6y4i4K6g2X3f1%4m8W2j5$3W2X3K9h3y4S2N6r3W2G2L8W2)9#2k6Y4j5I4i4K6g2X3x3#2)9#2k6V1k6A6L8X3q4D9i4K6u0W2M7r3c8X3">OMTP</a>规范的信息, 而处理这些消息的过程以及可视化语音信箱的 IMAP 功能是 Java 实现的。在过去, Android 曾经支持 SUPL 协议（用于 GPS 星历更新的 SMS 协议）在用户空间中解析, 并且进行解析工作的 `Intent` 对象是存储在 SMS 客户端堆栈中。但是, 现在高通的基带就可以直接处理这些消息, 因此处理过程不会在新的设备上的用户空间中进行。Messages 客户端中存在的漏洞的主要是由处理媒体文件的库引发的, 过去大多漏洞都是在这些组件中发现的, 然而与之前相比, 最近爆出的这样的漏洞少多了, 再加上 media 是由低特权进程处理的。所以, 目前我没有在 Messages 中发现任何漏洞。

### Gmail

Gmail 应用程序可处理由 IMAP、POP 或其他服务器发出的 MIME 消息, 这些消息的内容通常直接由非 Gmail 用户发出, 不过 Gmail 的邮件服务器还是能过滤掉一些邮件的。Gmail 的客户端会调用 AOSP 中的实现邮件协议底层 API, 这些 API 和 Gmail 的客户端都是用 Java 实现的。在 Gmail 的客户端中, 我发现了一个由附件上传引发的目录遍历的<a href="elink@72fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1N6h3N6K6i4K6u0W2j5$3S2J5L8$3#2A6N6h3#2Q4x3X3g2G2M7X3N6Q4x3V1k6H3i4K6u0r3M7s2u0G2K9X3g2U0N6q4)9J5k6s2A6W2M7X3!0Q4x3V1k6A6M7%4y4#2k6i4y4Q4x3V1k6V1k6i4c8S2K9h3I4Q4x3@1k6A6k6q4)9K6c8o6p5K6y4o6t1`.">bug</a>。

### Outlook

微软 Outlook 客户端的功能和 Gmail 的差不太多, 同样会处理另一台服务器发送的 MIME 消息, 但其所有协议的处理过程都是由客户端中的代码完成的。Outlook 主要是用 Java 写的, 其中的 native 的功能和邮件本身没多大关系。和在 Gmail 中一样, Outlook 也存在着相似<a href="elink@1f9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1N6h3N6K6i4K6u0W2j5$3S2J5L8$3#2A6N6h3#2Q4x3X3g2G2M7X3N6Q4x3V1k6H3i4K6u0r3M7s2u0G2K9X3g2U0N6q4)9J5k6s2A6W2M7X3!0Q4x3V1k6A6M7%4y4#2k6i4y4Q4x3V1k6V1k6i4c8S2K9h3I4Q4x3@1k6A6k6q4)9K6c8o6p5K6y4e0j5`.">目录遍历漏洞</a>。

### Facebook Messenger

与我测试的其他通讯客户端相比, Facebook Messenger 的工程量挺大的了, 很多代码很难明确其功能。该应用中包含大量的 native 库, 几乎每天都要从 Facebook 服务器重新加载数据动态更新。不过, 这些库好像很少直接参与消息的处理。

Facebook Messenger 使用一种叫 "消息队列遥测传输（<a href="elink@f2aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3#2I4N6s2c8Q4x3X3g2G2M7X3N6Q4x3V1j5`.">MQTT</a>）" 的协议与服务器进行通信——MQTT 的底层是开源的<a href="elink@9ffK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6X3j5h3y4W2j5X3!0G2K9#2)9J5c8X3k6T1N6r3S2J5K9h3k6@1i4K6u0r3N6s2u0W2k6g2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8Y4c8Z5M7X3W2X3N6q4)9J5c8X3I4A6j5W2)9J5c8X3A6S2N6X3q4Q4x3V1k6@1K9s2u0A6k6Y4c8Q4x3V1k6K6M7X3y4Q4x3V1k6E0j5h3W2F1i4K6u0r3K9X3q4$3j5g2)9J5c8X3y4G2L8g2)9J5c8X3k6S2j5$3g2T1L8$3!0C8i4K6u0r3N6r3S2J5K9h3k6@1i4K6u0r3M7s2u0G2N6r3!0U0L8$3H3`.">Thrift 协议</a>。应用中实现该协议的有 Java 和 c++ 两种的库, 但好像只用了 Java 的库。我检查了协议相关代码, 没有发现 Facebook Messenger 的漏洞。

### WhatsApp

WhatsApp 使用的分层协议主要是用 Java 实现的, 该应用处理语音和视频呼叫请求的消息代码是在 native 代码中实现的, 我在我写的<a href="elink@23aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4L8$3!0Y4L8r3g2H3M7X3!0B7k6h3y4@1P5X3g2J5L8#2)9J5k6h3u0D9L8$3N6K6M7r3!0@1i4K6u0W2j5$3!0E0i4K6u0r3x3U0l9I4z5q4)9J5c8U0p5J5i4K6u0r3j5h3c8$3k6h3&6@1N6i4u0W2M7#2)9J5k6r3W2F1i4K6u0V1N6X3W2V1k6h3!0Q4x3X3c8U0L8$3&6X3k6i4u0W2L8X3y4A6L8X3N6Q4x3X3c8H3j5i4u0@1i4K6u0V1y4q4)9J5k6h3S2@1L8h3H3`.">一篇文章</a>中讨论了相关内容。目前, 我没有在 WhatsApp 发现任何漏洞。

### Telegram

Telegram 是<a href="elink@98fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6p5M7V1E0x3e0#2)9J5c8W2c8W2L8r3g2Y4M7X3q4E0">开源</a>的, 其 API 代码是公开的。其中有相当多的 native 代码, 但该部分代码主要用于设备到服务器通信, 而不是端对端通信。因此, 远程攻击者可用的攻击面几乎都在 Java 中。在 Telegram 中, 我发现了一个<a href="elink@1eeK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1N6h3N6K6i4K6u0W2j5$3S2J5L8$3#2A6N6h3#2Q4x3X3g2G2M7X3N6Q4x3V1k6H3i4K6u0r3M7s2u0G2K9X3g2U0N6q4)9J5k6s2A6W2M7X3!0Q4x3V1k6A6M7%4y4#2k6i4y4Q4x3V1k6V1k6i4c8S2K9h3I4Q4x3@1k6A6k6q4)9K6c8o6p5@1y4K6l9`.">bug</a>, 同样是一个与附件相关的目录遍历漏洞。

### Signal

Signal 主要由 Java 实现, 攻击面相当小。虽然, 它使用的通讯协议有很好的文档记录。但是, 我没有在其中的任何漏洞。

## 漏洞影响

总的来说, 我在这些客户端中一共发现了三个目录遍历漏洞。这些漏洞惊人地相似, 都与附件名有关, 是由附件在下载时允许附件名包含特殊字符造成的。目录遍历问题在 Android 应用中非常常见, 造成该漏洞的部分原因应该是 Java API 在文件名方面处理得不够严谨。

不过, 这些漏洞具有局限性。首先, Gmail 和 Outlook 邮件客户端中的 bug 仅在客户端与非标准电子邮件地址一起使用时才起作用（即 Gmail 与非 Gmail 帐户一起使用, 或者 Outlook 与非 outlook 电子邮件地址一起使用）。虽然, Gmail 和 Outlook 邮件客户端都经常与非标准电子邮件地址一起使用, 但这些漏洞不会影响每个用户。并且想要利用这些漏洞, 还需要用户进行下载附件的操作。

该漏洞的利用还有一些限制条件。比如, 想要利用这些漏洞不能覆盖存在的文件, 只能往里面写入新文件。因此, 攻击者需要在某个地方放置系统可以处理但不存在的文件。但 Android 上的应用程序权限有限, 通常不能在其应用程序目录之外写入。

上面失败之后, 又有了一个绝妙的 idea 进入到我脑子里, 通过利用 SQLite 中漏洞实现攻击, 因为 Gmail 和 Outlook 都使用 SQLite 数据库来存储数据。SQLite 支持日志记录, 这意味着当 SQLite 数据库发生更改时, 会先写到日志文件里面, 防止在写入过程中系统崩溃或断电时数据库的数据丢失。如果发生系统崩溃或断电的情况, 在下次访问数据库时, 就会存在一个保存着更改信息日志文件, 这时 SQLite 就知道要还原已更改信息。一旦信息还原完成, 日志文件就会被删除。这意味着, 可以通过在同一目录下写入具有特定名称的日志文件来更改 SQLite 数据库, 也满足此文件在一般情况下不存在的条件。

我尝试过这个方法, 并且更改了数据库, 但只能改变用户在应用中看到的邮件, 不能进行提权或执行非法操作。为了寻找更通用的方法, 我想到了可以从 SQLite 解析日志文件或数据库文件的方式下手。

上面失败之后, 又有了一个绝妙的 idea 进入到我脑子里, 通过利用 SQLite 中漏洞实现攻击, 因为 Gmail 和 Outlook 都使用 SQLite 数据库来存储数据。SQLite 支持日志记录, 这意味着当 SQLite 数据库发生更改时, 会先写到日志文件里面, 防止在写入过程中系统崩溃或断电时数据库的数据丢失。如果发生系统崩溃或断电的情况, 在下次访问数据库时, 就会存在一个保存着更改信息日志文件, 这时 SQLite 就知道要还原已更改信息。一旦信息还原完成, 日志文件就会被删除。这意味着, 可以通过在同一目录下写入具有特定名称的日志文件来更改 SQLite 数据库, 也满足此文件在一般情况下不存在的条件。

我尝试过这个方法, 并且更改了数据库, 但只能改变用户在应用中看到的邮件, 不能进行提权或执行非法操作。为了寻找更通用的方法, 我想到了可以从 SQLite 解析日志文件或数据库文件的方式下手。

我用 `AFL` 对数据库的加载和日志处理进行了 Fuzzing 测试。我发现<a href="elink@7fdK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6M7h3I4A6N6r3g2Q4x3X3g2G2M7X3N6Q4x3V1k6K6M7X3y4Q4x3V1k6A6L8X3k6G2i4K6u0r3x3o6t1^5x3U0S2V1y4K6p5%4k6e0u0V1z5e0N6T1x3b7`.`.">四个漏洞</a>, 其中三个我认为是不可利用的, 剩下一个利用的可能性非常小。

我找不到更多的方法来利用目录遍历, 因为不能覆盖 Android 应用程序中的文件, 所以我不认为这些目录遍历漏洞对攻击者来说会特别有价值。

## 总结

我测试了几个 Android 通讯和邮件客户端, 发现了三个目录遍历的漏洞, 但这些漏洞的利用存在一些限制, 现在还不清楚在一般情况下这些漏洞能否被利用。
