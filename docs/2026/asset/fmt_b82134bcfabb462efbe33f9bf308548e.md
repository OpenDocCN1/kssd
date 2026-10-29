
一个好的攻击者几乎可以静悄悄在你的网络上移动，甚至当你以SMB知识武装自己后，他们有时也能逃避检测。Windows Management Instrumentation（WMI）和PowerShell远程处理是两种可以避免使用SMB的技术，且可以执行我们之前讨论过的一些相同的功能。（WMI可以使用基于TCP的RPC，并且可以通过基于HTTP的Windows远程管理（WinRM）运行PowerShell远程处理。有关PowerShell远程处理的更多详细信息，请参阅[这里](https://bbs.kanxue.com/elink@807K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1k6h3y4Z5L8X3g2@1i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8X3I4A6j5Y4u0S2M7Y4W2Q4x3V1k6X3k6U0M7H3x3o6t1J5y4#2)9J5k6h3q4K6M7s2R3`.)）我们在这篇文章中没有看到任何一个指标表明恶意行为本身，但是与任何协议一样，对于正常的SMB流量在您的网络中看起来是什么样的较好理解对于发现可能与攻击者有关的异常行为是至关重要的。我希望这个指南会成为一个有用的介绍，然后你可以使用你在这里学到的一些工具来在你的网络中开始发动追捕。（[8a0K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3S2S2M7X3#2B7x3s2W2Q4x3X3g2F1k6i4c8Q4x3V1k6T1L8r3!0Y4i4K6u0r3](https://bbs.kanxue.com/elink@59dK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3S2S2M7X3#2B7x3s2W2Q4x3X3g2F1k6i4c8Q4x3V1k6T1L8r3!0Y4i4K6u0r3)是开始扩展Active Directory攻击技术知识的绝佳资源，[f95K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3q4V1M7$3g2U0N6i4u0A6N6s2W2Q4x3X3g2G2M7X3N6Q4x3V1j5`.](https://bbs.kanxue.com/elink@3b0K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3q4V1M7$3g2U0N6i4u0A6N6s2W2Q4x3X3g2G2M7X3N6Q4x3V1j5`.)是另一个很好的来源）

译注：

来源：[8dfK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1j5@1x3o6q4@1M7X3N6Q4x3X3g2H3N6#2)9J5c8X3q4F1i4K6u0V1K9h3&6@1M7X3!0V1N6h3y4@1K9h3!0F1i4K6u0V1N6r3!0Q4x3X3c8K6L8h3u0Q4x3X3c8X3L8%4u0Q4x3X3c8F1k6i4c8%4L8%4u0C8i4K6u0V1M7$3g2U0N6i4u0A6N6s2W2Q4x3X3c8S2L8X3q4D9P5i4y4@1M7#2)9J5c8R3`.`.](https://bbs.kanxue.com/elink@1c0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1j5@1x3o6q4@1M7X3N6Q4x3X3g2H3N6#2)9J5c8X3q4F1i4K6u0V1K9h3&6@1M7X3!0V1N6h3y4@1K9h3!0F1i4K6u0V1N6r3!0Q4x3X3c8K6L8h3u0Q4x3X3c8X3L8%4u0Q4x3X3c8F1k6i4c8%4L8%4u0C8i4K6u0V1M7$3g2U0N6i4u0A6N6s2W2Q4x3X3c8S2L8X3q4D9P5i4y4@1M7#2)9J5c8R3`.`.) 2017年12月20日

本文由 看雪翻译小组 [hanbingxzy](http://bbs.pediy.com/user-505507.htm) 编译

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
