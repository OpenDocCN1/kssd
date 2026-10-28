
Microsoft Windows 自带的脚本解释器包括 PowerShell、VBScript、批处理文件和 JavaScript。对应的应用程序为 `powershell.exe`、`cscript.exe`、`cmd.exe` 和 `mshta.exe`。此外，Windows 提供的 Linux 子系统，使得其支持的脚本语言更加广泛了。该问题带来的众多挑战中的一个是，企业需要限制这些工具被滥用，见 Gal Bitensky 的文章 [a09K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2L8h3W2F1k6i4u0$3j5g2)9J5k6r3I4S2j5Y4y4Q4x3X3g2U0L8$3#2Q4x3V1k6U0L8$3&6X3M7X3!0F1N6r3W2F1k6#2)9J5k6s2y4F1j5h3E0W2i4K6u0V1L8$3W2D9i4K6u0V1M7$3q4D9k6i4y4Q4x3X3c8@1j5h3y4@1K9h3y4K6i4K6u0V1K9h3&6Q4x3X3c8W2L8X3c8H3L8$3W2F1N6q4)9J5k6s2y4W2j5%4g2J5K9i4c8&6](https://bbs.kanxue.com/elink@b78K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2L8h3W2F1k6i4u0$3j5g2)9J5k6r3I4S2j5Y4y4Q4x3X3g2U0L8$3#2Q4x3V1k6U0L8$3&6X3M7X3!0F1N6r3W2F1k6#2)9J5k6s2y4F1j5h3E0W2i4K6u0V1L8$3W2D9i4K6u0V1M7$3q4D9k6i4y4Q4x3X3c8@1j5h3y4@1K9h3y4K6i4K6u0V1K9h3&6Q4x3X3c8W2L8X3c8H3L8$3W2F1N6q4)9J5k6s2y4W2j5%4g2J5K9i4c8&6)。

然而，攻击者却可以利用各种框架工具轻易地混淆攻击脚本。这些混淆措施包括：Daniel Bohannon 针对 PowerShell 的 [`Invoke-Obfuscation`](https://bbs.kanxue.com/elink@87bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6V1j5h3&6A6k6h3I4T1L8$3S2S2L8X3&6G2L8W2)9J5c8V1W2F1N6X3!0C8k6g2)9J5k6p5!0T1k6Y4g2K6j5$3q4@1K9h3!0F1) 和 [`Invoke-DOSfuscation`](https://bbs.kanxue.com/elink@c12K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6V1j5h3&6A6k6h3I4T1L8$3S2S2L8X3&6G2L8W2)9J5c8V1W2F1N6X3!0C8k6g2)9J5k6p5c8a6f1$3k6#2M7$3y4S2N6r3W2G2L8R3`.`.) 框架。相应的实际操作，可参见 Minerva 关于 [Emotet’s script obfuscation](https://bbs.kanxue.com/elink@85dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2L8h3W2F1k6i4u0$3j5g2)9J5k6r3I4S2j5Y4y4Q4x3X3g2U0L8$3#2Q4x3V1k6@1K9r3g2Q4x3X3c8W2L8h3!0@1k6i4c8Q4x3X3c8Y4M7X3W2F1j5$3S2Q4x3X3c8A6M7#2)9J5k6r3u0S2j5$3D9`.) 的分析。

## 手段 #3：Living off the Land（平地起飞）

对无文件攻击的讨论，通常包括对 Microsoft Windows 众多内建工具的滥用。这些工具使得攻击者轻松地从一个阶段“跳转”到另一个阶段，无需执行任何编译的二进制可执行文件。这种操作模式有时被称为“平地起飞”。

一旦攻击者的恶意代码能与本地程序进行交互，那么攻击者就有可能利用系统的原生工具进行下一步攻击，包括下载附带的其他恶意代码，启动程序，执行脚本，窃取数据，横向扩展，维持访问等等。攻击者为达到这些目的而调用的工具包括 `regsvr32.exe`、`rundll32.exe`、`certutil.exe` 和 `schtasks.exe`。想更加全面地了解系统中被利用的内建程序，库和脚本，参考 Oddvar Moe 的 [LOLBAS project](https://bbs.kanxue.com/elink@8f4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6S2M7r3V1H3j5%4u0S2k6r3I4W2i4K6u0r3e0p5!0x3b7V1q4e0)。

Windows 管理规范（WMI），是 Windows 系统内建工具，为攻击者提供了“平地起飞”的绝好机会。WMI 借助运行 `wmic.exe` 程序和执行脚本（如 PowerShell），使得攻击者可以操作设备的绝大部分配置。这些工具都是系统自带的、受信任的，所以反恶意软件技术也难以侦测和限制。关于利用 WMI 进行无文件攻击的详细内容，请见 Matt Graeber 的文章 [Abusing WMI to Build a Persistent, Asynchronous, and Fileless Backdoor](https://bbs.kanxue.com/elink@710K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2T1L8r3q4U0K9$3S2S2N6q4)9J5k6h3y4G2L8g2)9J5c8X3c8G2j5%4y4Q4x3V1k6#2M7#2)9J5k6o6p5#2i4K6u0r3L8h3q4@1k6i4u0A6j5h3I4K6i4K6u0r3N6i4y4Q4x3X3b7I4y4g2)9J5k6p5N6J5j5h3g2T1k6i4u0Q4x3X3c8m8j5Y4g2K6K9h3&6Y4i4K6u0V1g2$3W2F1k6r3!0%4M7#2)9J5k6p5#2S2L8X3q4Y4k6h3#2W2L8Y4c8Q4x3X3c8u0L8Y4y4@1M7Y4g2E0k6h3&6@1j5i4c8A6L8$3&6Q4x3X3c8i4e0f1W2Q4x3X3c8f1L8#2)9J5k6p5u0#2K9h3I4V1i4K6u0V1b7g2)9J5k6q4m8W2M7Y4y4A6M7%4c8W2L8Y4c8Q4x3U0f1J5x3p5q4K6P5h3&6U0K9s2u0G2L8X3!0#2M7#2)9J5k6p5q4F1k6q4)9J5k6p5k6A6L8r3g2D9k6i4y4K6i4K6u0V1b7X3q4U0K9$3c8G2L8%4u0Q4x3X3c8%4M7q4)9J5k6i4m8V1k6R3`.`.)（注 1）。

攻击者依靠这些系统自带的受信任工具，显著地提高了他们逃避反恶意软件工具及应用白名单限制的机率。更多此类的案件，参考 Matthew Dunwoody 的 [APT29’s use of WMI and PowerShell to plant fileless backdoors](https://bbs.kanxue.com/elink@c72K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2X3K9i4u0W2k6i4W2W2i4K6u0W2j5$3!0E0i4K6u0r3j5X3I4G2k6#2)9J5c8Y4c8Z5M7X3g2S2N6q4)9J5k6s2u0W2M7$3g2S2M7X3y4Z5i4K6u0r3x3U0l9I4y4#2)9J5c8U0l9K6i4K6u0r3k6r3W2K6M7$3g2U0N6r3W2F1k6#2)9#2k6X3!0F1k6g2)9#2k6X3!0X3j5i4m8Q4x3X3g2Z5N6r3#2D9)。

## 手段 #4：内存恶意代码

现今的反病毒产品，对于检测磁盘上的恶意文件，是相当在行的；然而，要想检测到只存在于内存中的恶意代码，他们往往是事倍功半。内存的易失性、动态性，使得恶意软件很容易地改变形态；同时，可以自由地运行，无需顾忌任何反恶意技术的侦测。

一旦攻击者开始在终端设备上执行恶意代码（可能是利用上面提及的方法），那么他就有可能将恶意软件解包到内存，无需在磁盘上留下丝毫痕迹。此过程可能包括解压代码到进程自身的内存空间。其他情况下，恶意程序可能将代码注入到其他受信任的进程或者其他正常进程。

内存攻击技术的例子包括以下一些：

- 在不利用系统脆弱性的情况下，利用系统功能的内存注入。

  例如，包括 `VirtualAllocEx` 和 `WriteProcessMemory` 在内的 API 常被恶意软件滥用于内存注入。详细操作，参阅 Gal Bitensky 的 [overview of the AZORult attack](https://bbs.kanxue.com/elink@f61K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2L8h3W2F1k6i4u0$3j5g2)9J5k6r3I4S2j5Y4y4Q4x3X3g2U0L8$3#2Q4x3V1k6S2L8X3q4D9P5i4A6A6L8X3N6Q4x3X3c8S2L8W2)9J5k6r3q4*7L8%4u0#2L8s2c8Q4x3X3c8S2N6s2c8S2j5$3E0Q4x3X3c8W2N6X3q4K6K9h3!0F1i4K6u0V1K9h3&6Q4x3X3c8S2i4K6u0V1j5$3I4G2j5h3E0Q4x3X3c8G2k6W2)9J5k6r3#2#2L8s2c8A6M7r3I4W2i4K6u0V1L8r3q4&6k6i4u0K6)。

- 攻击者可能将编译的可执行程序捆绑在脚本里面，从而在脚本运行时把恶意攻击载荷释放到内存。该攻击手段的常用（一个样例）工具为 [PowerSploit](https://bbs.kanxue.com/elink@bfbK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6b7L8%4N6W2M7W2y4Z5k6h3I4D9e0h3q4X3K9h3q4Q4x3V1k6b7L8%4N6W2M7W2y4H3L8r3!0A6N6l9`.`.)，操作过程可以阅读 Asaf Aprozper 和 Gal Bitensky 的 [the GhostMiner analysis](https://bbs.kanxue.com/elink@9bdK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2L8h3W2F1k6i4u0$3j5g2)9J5k6r3I4S2j5Y4y4Q4x3X3g2U0L8$3#2Q4x3V1k6Y4K9r3!0K6N6r3#2A6L8X3g2J5i4K6u0V1j5%4u0&6M7s2c8G2L8h3W2F1K9h3&6Y4i4K6u0V1L8h3q4D9N6$3q4J5k6g2)9J5k6r3N6G2k6i4y4Q4x3X3c8X3K9h3I4W2L8r3g2K6M7H3`.`.)（注 2）。Chris Truncer 的 [Veil Framework](https://bbs.kanxue.com/elink@807K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2$3k6h3W2D9i4K6u0V1k6Y4u0S2L8h3g2%4L8%4u0C8i4K6u0W2j5$3!0E0i4K6u0r3) 是另一实例。

- `Process Doppelgänging` 是另一种无文件技术，且无需涉及经典的内存注入。取而代之的是，攻击者利用 Microsoft Windows NTFS 的事物（访问）特性：临时修改受信任的内存文件而不同步更改至磁盘。Anton Ivanov, Fedor Sinitsyn 和 Orkhan Mamedov 在 [SynAck malware](https://bbs.kanxue.com/elink@462K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6k6h3y4#2M7X3g2D9K9i4y4@1i4K6u0W2j5$3!0E0i4K6u0r3M7%4W2F1j5h3y4C8i4K6u0V1N6r3q4J5k6$3g2@1k6h3c8Q4x3X3c8J5j5h3&6K6L8$3#2%4j5i4u0W2i4K6u0V1N6i4y4W2M7#2)9J5k6s2c8Z5k6g2)9J5k6r3c8G2M7s2m8W2L8r3N6S2L8X3N6A6L8X3N6Q4x3X3c8@1k6h3y4Z5L8X3W2I4N6h3g2Q4x3V1j5^5y4e0b7K6x3g2)9J5c8R3`.`.) 中解析了该技术。

内存滞留技术，使得攻击者可以绕过众多的反恶意技术的控制，包括应用白名单策略。尽管反病毒工具试图追踪内存注入行为，但是攻击者的感染手段使得其防不胜防。Asaf Aprozper 的 [fdcK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2L8h3W2F1k6i4u0$3j5g2)9J5k6r3I4S2j5Y4y4Q4x3X3g2U0L8$3#2Q4x3V1k6U0L8$3k6X3k6h3g2K6K9r3!0@1i4K6u0V1j5i4k6G2K9h3c8Q4x3X3c8V1k6i4c8W2j5%4c8A6L8$3&6Q4x3X3c8%4K9i4c8Z5i4K6u0V1L8h3g2E0L8%4u0&6i4K6u0V1K9h3&6B7k6h3y4@1K9h3!0F1](https://bbs.kanxue.com/elink@405K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2L8h3W2F1k6i4u0$3j5g2)9J5k6r3I4S2j5Y4y4Q4x3X3g2U0L8$3#2Q4x3V1k6U0L8$3k6X3k6h3g2K6K9r3!0@1i4K6u0V1j5i4k6G2K9h3c8Q4x3X3c8V1k6i4c8W2j5%4c8A6L8$3&6Q4x3X3c8%4K9i4c8Z5i4K6u0V1L8h3g2E0L8%4u0&6i4K6u0V1K9h3&6B7k6h3y4@1K9h3!0F1) 利用 Java 实现的一个注入方法说明了各种检测方法多么脆弱。

## “无文件”攻击已成为常态

尽管只有少数攻击和恶意软件家族是全程“无文件”的形式，但是大多数恶意软件为了逃避防御系统检测会加入一些无文件的特性。相比于确定威胁是否是全程“无文件”的，我们考虑恶意攻击是采用何种策略、办法绕过防御系统的，对于日后改进防护的工作，或许会更有成效。

攻击者可能结合上面提到的包括利用恶意文本文档，脚本，“平地起飞”以及内存注入等。这些无文件攻击技术，借助应用程序和操作系统的功能特性，以及反恶意软件技术无法有效地侦测和阻止系统工具的滥用的弱点，使得“无文件”攻击成为攻击的新常态。这是越来越多企业关注 [406K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2E0K9h3&6W2M7Y4k6S2i4K6u0V1L8r3q4T1M7#2)9J5k6h3y4G2L8g2)9J5c8X3#2A6L8X3g2J5N6X3q4Q4x3X3c8S2L8Y4c8A6i4K6u0V1k6i4k6S2M7$3W2G2L8W2)9J5k6s2m8D9j5i4c8X3L8%4u0E0](https://bbs.kanxue.com/elink@6b1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2E0K9h3&6W2M7Y4k6S2i4K6u0V1L8r3q4T1M7#2)9J5k6h3y4G2L8g2)9J5c8X3#2A6L8X3g2J5N6X3q4Q4x3X3c8S2L8Y4c8A6i4K6u0V1k6i4k6S2M7$3W2G2L8W2)9J5k6s2m8D9j5i4c8X3L8%4u0E0) 的原因，这是专门设计来对抗无文件攻击策略的， [9ebK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2L8h3W2F1k6i4u0$3j5g2)9J5k6r3I4S2j5Y4y4Q4x3X3g2U0L8$3#2Q4x3V1k6U0L8$3&6K6K9h3c8W2M7X3q4@1K9h3!0F1M7#2)9J5k6r3k6G2M7W2)9J5k6r3q4#2k6$3#2W2L8Y4c8A6L8X3N6Q4x3X3c8S2L8Y4c8A6N6X3W2J5N6i4y4Q4x3X3c8H3M7X3!0@1k6h3y4@1K9h3!0F1i4K6u0V1L8$3&6Q4x3X3c8@1K9r3g2Q4x3X3c8W2L8X3c8H3L8$3W2F1N6l9`.`.](https://bbs.kanxue.com/elink@98dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2L8h3W2F1k6i4u0$3j5g2)9J5k6r3I4S2j5Y4y4Q4x3X3g2U0L8$3#2Q4x3V1k6U0L8$3&6K6K9h3c8W2M7X3q4@1K9h3!0F1M7#2)9J5k6r3k6G2M7W2)9J5k6r3q4#2k6$3#2W2L8Y4c8A6L8X3N6Q4x3X3c8S2L8Y4c8A6N6X3W2J5N6i4y4Q4x3X3c8H3M7X3!0@1k6h3y4@1K9h3!0F1i4K6u0V1L8$3&6Q4x3X3c8@1K9r3g2Q4x3X3c8W2L8X3c8H3L8$3W2F1N6l9`.`.) 。

## 参考
