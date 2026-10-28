
# 4 种常见“无文件”攻击技术解析

---

![4 techniques_1042x1042.jpg](https://bbs.kanxue.com/upload/attach/202002/747320_EKXQTQYAARK7UKH.jpg)

为了应对愈演愈烈的**无文件攻击**，我们需要明确地定义“无文件”这个词的含义，以便深入剖析此类攻击的攻击手段及其对现有环境和防御系统的影响。

在安全领域，无文件攻击意味着极其严重的威胁。“无文件”一词，是在探讨绕过恶意文件检测技术的方法时诞生的术语。

无文件攻击技术，利用一系列手段，使得攻击者可以攻陷带有反病毒和应用白名单策略防护的系统。下面，是对该攻击手段的一个概况，包括无文件攻击威胁的准确、清晰的描述。

- 攻击者可能将编译的可执行程序捆绑在脚本里面，从而在脚本运行时把恶意攻击载荷释放到内存。该攻击手段的常用（一个样例）工具为 <a href="elink@740K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6b7L8%4N6W2M7W2y4Z5k6h3I4D9e0h3q4X3K9h3q4Q4x3V1k6b7L8%4N6W2M7W2y4H3L8r3!0A6N6l9`.`.">PowerSploit</a>, 操作过程可以阅读 Asaf Aprozper 和 Gal Bitensky 的 <a href="elink@22cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2L8h3W2F1k6i4u0$3j5g2)9J5k6r3I4S2j5Y4y4Q4x3X3g2U0L8$3#2Q4x3V1k6Y4K9r3!0K6N6r3#2A6L8X3g2J5i4K6u0V1j5%4u0&6M7s2c8G2L8h3W2F1K9h3&6Y4i4K6u0V1L8h3q4D9N6$3q4J5k6g2)9J5k6r3N6G2k6i4y4Q4x3X3c8X3K9h3I4W2L8r3g2K6M7H3`.`."> the GhostMiner analysis</a>（注 2 ）。 Chris Truncer 的 <a href="elink@928K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2$3k6h3W2D9i4K6u0V1k6Y4u0S2L8h3g2%4L8%4u0C8i4K6u0W2j5$3!0E0i4K6u0r3"> Veil Framework</a> 是另一实例。

- `Process Doppelgänging` 是另一种无文件技术，且无需涉及经典的内存注入。取而代之的是，攻击者利用 Microsoft Windows NTFS 的事物（访问）特性：临时修改受信任的内存文件而不同步更改至磁盘。Anton Ivanov, Fedor Sinitsyn 和 Orkhan Mamedov 在 <a href="elink@10eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6k6h3y4#2M7X3g2D9K9i4y4@1i4K6u0W2j5$3!0E0i4K6u0r3M7%4W2F1j5h3y4C8i4K6u0V1N6r3q4J5k6$3g2@1k6h3c8Q4x3X3c8J5j5h3&6K6L8$3#2%4j5i4u0W2i4K6u0V1N6i4y4W2M7#2)9J5k6s2c8Z5k6g2)9J5k6r3c8G2M7s2m8W2L8r3N6S2L8X3N6A6L8X3N6Q4x3X3c8@1k6h3y4Z5L8X3W2I4N6h3g2Q4x3V1j5^5y4e0b7K6x3g2)9J5c8R3`.`."> SynAck malware</a>中解析了该技术。

内存滞留技术，使得攻击者可以绕过众多的反恶意技术的控制，包括应用白名单策略。尽管反病毒工具试图追踪内存注入行为，但是攻击者的感染手段使得其防不胜防。Asaf Aprozper的<a href="elink@806K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2L8h3W2F1k6i4u0$3j5g2)9J5k6r3I4S2j5Y4y4Q4x3X3g2U0L8$3#2Q4x3V1k6U0L8$3k6X3k6h3g2K6K9r3!0@1i4K6u0V1j5i4k6G2K9h3c8Q4x3X3c8V1k6i4c8W2j5%4c8A6L8$3&6Q4x3X3c8%4K9i4c8Z5i4K6u0V1L8h3g2E0L8%4u0&6i4K6u0V1K9h3&6B7k6h3y4@1K9h3!0F1"> <mark class="encrypted">94fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2L8h3W2F1k6i4u0$3j5g2)9J5k6r3I4S2j5Y4y4Q4x3X3g2U0L8$3#2Q4x3V1k6U0L8$3k6X3k6h3g2K6K9r3!0@1i4K6u0V1j5i4k6G2K9h3c8Q4x3X3c8V1k6i4c8W2j5%4c8A6L8$3&6Q4x3X3c8%4K9i4c8Z5i4K6u0V1L8h3g2E0L8%4u0&6i4K6u0V1K9h3&6B7k6h3y4@1K9h3!0F1</mark> </a>利用JAVA实现的一个注入方法说明了各种检测方法多么脆弱。

## “无文件”攻击已成为常态

尽管只有少数攻击和恶意软件家族是全程“无文件”的形式，但是大多数恶意软件为了逃避防御系统检测会加入一些无文件的特性。相比于确定威胁是否是全程“无文件”的，我们考虑恶意攻击是采用何种策略、办法绕过防御系统的，对于日后改进防护的工作，或许会更有成效。

攻击者可能结合上面提到的包括利用恶意文本文档，脚本，“平地起飞”以及内存注入等。这些无文件攻击技术，借助应用程序和操作系统的功能特性，以及反恶意软件技术无法有效地侦测和阻止系统工具的滥用的弱点，使得“无文件”攻击成为攻击的新常态。这是越来越多企业关注<a href="elink@5bdK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2E0K9h3&6W2M7Y4k6S2i4K6u0V1L8r3q4T1M7#2)9J5k6h3y4G2L8g2)9J5c8X3#2A6L8X3g2J5N6X3q4Q4x3X3c8S2L8Y4c8A6i4K6u0V1k6i4k6S2M7$3W2G2L8W2)9J5k6s2m8D9j5i4c8X3L8%4u0E0"> <mark class="encrypted">196K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2E0K9h3&6W2M7Y4k6S2i4K6u0V1L8r3q4T1M7#2)9J5k6h3y4G2L8g2)9J5c8X3#2A6L8X3g2J5N6X3q4Q4x3X3c8S2L8Y4c8A6i4K6u0V1k6i4k6S2M7$3W2G2L8W2)9J5k6s2m8D9j5i4c8X3L8%4u0E0</mark> </a>的原因，这是专门设计来对抗无文件攻击策略的，<a href="elink@ce9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2L8h3W2F1k6i4u0$3j5g2)9J5k6r3I4S2j5Y4y4Q4x3X3g2U0L8$3#2Q4x3V1k6U0L8$3&6K6K9h3c8W2M7X3q4@1K9h3!0F1M7#2)9J5k6r3k6G2M7W2)9J5k6r3q4#2k6$3#2W2L8Y4c8A6L8X3N6Q4x3X3c8S2L8Y4c8A6N6X3W2J5N6i4y4Q4x3X3c8H3M7X3!0@1k6h3y4@1K9h3!0F1i4K6u0V1L8$3&6Q4x3X3c8@1K9r3g2Q4x3X3c8W2L8X3c8H3L8$3W2F1N6l9`.`."> <mark class="encrypted">86bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2L8h3W2F1k6i4u0$3j5g2)9J5k6r3I4S2j5Y4y4Q4x3X3g2U0L8$3#2Q4x3V1k6U0L8$3&6K6K9h3c8W2M7X3q4@1K9h3!0F1M7#2)9J5k6r3k6G2M7W2)9J5k6r3q4#2k6$3#2W2L8Y4c8A6L8X3N6Q4x3X3c8S2L8Y4c8A6N6X3W2J5N6i4y4Q4x3X3c8H3M7X3!0@1k6h3y4@1K9h3!0F1i4K6u0V1L8$3&6Q4x3X3c8@1K9r3g2Q4x3X3c8W2L8X3c8H3L8$3W2F1N6l9`.`.</mark> </a>。

## 手段 #1 ：恶意文档

在许多被分类为无文件攻击的案例中，大部分都有文本文件的影子。在此类攻击中，对方往往是利用恶意文本文档（最常见的是电子邮件的附件），以达到以下的目的：

- 文本文档可作为其他文件的灵活载体、容器。

  例如，攻击者把 JavaScript 文件嵌入 Microsoft Office 文档中，利用社会工程学诱使文档接收者双击执行脚本。可内嵌文件的其它文档类型包括PDF、RTF。因为这些特性是各个应用程序的正常特性，所以通常情况下，反恶意软件技术也不会对这些功能的使用进行干扰。

- 文本文档可以携带执行恶意代码的漏洞利用。

  当前，文档功能的复杂性，为漏洞利用提供了广阔的攻击面。通常，exploit 可以触发捆绑的 shellcode 在应用程序的内存空间执行。这样一来，攻击者无需将恶意代码存放到本地文件系统上即可取得终端设备的控制权。

- 文本文档可以执行恶意感染的逻辑。

  现代文档支持强大的脚本能力，例如 Microsoft Office 支持 VBA 宏。此类功能特性，利用大多数反恶意软件无法区分恶意脚本与良性脚本的弱点，使得攻击者无需通过编译程序即可实施恶意感染。文档脚本拥有的能力包括了执行程序和下载恶意代码。

尽管文本文档本身还是存储在终端设备的本地文件系统中，但是，避免了传统的直接将恶意可执行文件存放于磁盘之上。在大多数情况下，文本文档引导恶意代码直接在终端设备的内存中执行。攻击者利用文本文档发起攻击的案例，见 Omri Moyal 的 write-up [the misuse of Microsoft Office support for DDE](https://bbs.kanxue.com/elink@4f7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2L8h3W2F1k6i4u0$3j5g2)9J5k6r3I4S2j5Y4y4Q4x3X3g2U0L8$3#2Q4x3V1k6H3M7X3g2$3k6h3&6@1k6h3c8Q4x3X3c8T1P5g2)9J5k6r3#2A6L8X3g2J5N6X3q4Q4x3X3c8D9j5h3u0K6i4K6u0V1L8X3g2%4i4K6u0V1k6i4k6S2M7$3W2$3k6g2)9J5k6r3#2S2L8s2N6S2M7X3g2Q4x3X3c8@1k6h3y4Z5L8X3W2I4N6h3g2Q4x3X3c8W2P5s2m8D9L8$3W2@1M7#2)9J5k6r3#2A6j5%4u0G2M7$3!0X3N6q4)9J5k6r3c8&6L8X3q4E0K9h3y4Q4x3X3c8W2P5r3y4Z5j5h3&6Y4k6b7`.`.) 。另一种情景，可阅读文章：[attackers inserting themselves into the conversation to spread malware](https://bbs.kanxue.com/elink@f77K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2L8h3W2F1k6i4u0$3j5g2)9J5k6r3I4S2j5Y4y4Q4x3X3g2U0L8$3#2Q4x3V1k6S2N6s2c8S2j5$3E0W2M7Y4y4Q4x3X3c8A6L8Y4y4W2M7Y4c8Q4x3X3c8@1K9r3g2E0M7$3g2D9N6X3g2K6i4K6u0V1K9h3&6@1L8#2)9J5k6s2c8Z5k6g2)9J5k6r3g2E0j5h3W2D9i4K6u0V1j5$3!0F1N6X3g2J5M7$3q4@1K9h3!0F1i4K6u0V1N6r3!0Q4x3X3c8K6M7s2u0W2j5h3c8Q4x3X3c8E0j5h3I4%4j5i4u0W2)。

## 手段 #2 ：恶意脚本

按照一贯的做法，恶意软件创作者倾向于“无文件”方式的脚本运行攻击，而非传统的将代码编译成可执行二进制文件。除了上面提到的文本文档对脚本的原生支持外，直接在 Microsoft Windows 下运行脚本还有以下几条优点：

- 直接与系统进行交互，无需受其他应用程序的限制，比如浏览器对脚本的检查。

- 与编译的可执行恶意文件相比，更不容易被反恶意软件技术探测到。

- 为躲避应用行为检测，可以灵活地将恶意逻辑分割到数个进程中。

- 可以利用混淆技术，延缓和更好地避免被反恶意软件技术分析、侦测。

## 参考

> 注 1 ：中文译文 <a href="elink@124K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2X3M7X3g2W2j5Y4g2X3i4K6u0W2j5$3!0E0i4K6u0r3j5$3!0D9N6h3#2F1i4K6u0r3x3e0j5#2x3U0x3#2i4K6u0W2K9s2c8E0L8l9`.`.">利用WMI构建无文件后门（基础篇） <mark class="encrypted">7deK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2X3M7X3g2W2j5Y4g2X3i4K6u0W2j5$3!0E0i4K6u0r3j5$3!0D9N6h3#2F1i4K6u0r3x3e0j5#2x3U0x3#2i4K6u0W2K9s2c8E0L8l9`.`.</mark></a>

> 注 2 ：中文译文 <a href="elink@054K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2X3M7X3g2W2j5Y4g2X3i4K6u0W2j5$3!0E0i4K6u0r3j5i4u0@1K9h3y4D9k6i4y4Q4x3V1k6T1L8r3!0U0K9$3y4Z5j5h3W2F1i4K6u0V1j5i4u0@1K9h3y4D9k6i4y4Q4x3V1j5I4y4K6l9^5x3o6N6Q4x3X3g2Z5N6r3#2D9">GhostMiner解析：无文件挖矿的新姿势 <mark class="encrypted">57fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2X3M7X3g2W2j5Y4g2X3i4K6u0W2j5$3!0E0i4K6u0r3j5i4u0@1K9h3y4D9k6i4y4Q4x3V1k6T1L8r3!0U0K9$3y4Z5j5h3W2F1i4K6u0V1j5i4u0@1K9h3y4D9k6i4y4Q4x3V1j5I4y4K6l9^5x3o6N6Q4x3X3g2Z5N6r3#2D9</mark> </a>

> 引用：来去无踪：针对JS\_POWMET无文件恶意软件的分析 [https://bbs.pediy.com/thread-220646.htm](https://bbs.pediy.com/thread-220646.htm)

----

> 原文：<a href="elink@f8cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2L8h3W2F1k6i4u0$3j5g2)9J5k6r3I4S2j5Y4y4Q4x3X3g2U0L8$3#2Q4x3V1k6V1k6h3y4G2L8Y4y4@1M7Y4g2U0N6r3W2F1k6#2)9J5k6r3k6A6L8r3g2D9k6i4y4K6i4K6u0V1j5i4c8@1j5h3y4C8M7#2)9J5k6r3W2F1N6r3!0Q4x3X3b7@1i4K6u0V1N6h3&6V1k6i4u0D9P5h3W2F1k6#2)9J5k6s2c8W2j5$3S2F1K9i4q4#2k6i4x3`."><mark class="encrypted">5bbK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2L8h3W2F1k6i4u0$3j5g2)9J5k6r3I4S2j5Y4y4Q4x3X3g2U0L8$3#2Q4x3V1k6V1k6h3y4G2L8Y4y4@1M7Y4g2U0N6r3W2F1k6#2)9J5k6r3k6A6L8r3g2D9k6i4y4K6i4K6u0V1j5i4c8@1j5h3y4C8M7#2)9J5k6r3W2F1N6r3!0Q4x3X3b7@1i4K6u0V1N6h3&6V1k6i4u0D9P5h3W2F1k6#2)9J5k6s2c8W2j5$3S2F1K9i4q4#2k6i4x3`.</mark></a>

> 编译: 看雪翻译小组 [StrokMitream](https://bbs.pediy.com/user-747320.htm)

> 校对: 看雪翻译小组 [ 玉林小学生 ](https://bbs.pediy.com/user-805111.htm)
