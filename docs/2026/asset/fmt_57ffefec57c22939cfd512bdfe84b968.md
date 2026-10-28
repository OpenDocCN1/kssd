
使用 Microsoft [ML.Net](https://bbs.kanxue.com/elink@2a5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8%4c8F1k6i4c8Q4x3X3g2E0K9h3y4J5L8%4y4G2k6Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6S2M7s2m8K6i4K6u0r3L8h3q4U0K9r3W2F1k6h3I4W2j5i4u0F1K9h3&6Y4i4K6u0V1j5h3W2Q4x3V1k6E0L8q4)9J5k6r3c8G2N6r3&6W2N6l9`.`.) 技术和 [ONNX](https://bbs.kanxue.com/elink@846K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6S2K9g2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3X3c8E0L8q4)9J5c8X3N6W2N6q4)9J5k6r3!0F1L8Y4S2Q4x3X3c8E0L8$3c8W2L8l9`.`.) 格式的深度神经网络，将性能最佳的深度学习模型大规模应用到 Microsoft Defender ATP 通过 [`AMSI`](https://bbs.kanxue.com/elink@0f5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6%4K9h3^5K6x3W2)9J5c8X3q4E0M7$3W2Q4x3V1k6Z5L8%4N6Q4x3X3c8S2L8i4y4A6i4K6u0V1K9r3g2D9M7s2x3`.) 接口捕获到的 PowerShell 脚本。该模型强化了 Microsoft Defender ATP 的 [机器学习模型和启发式工具套件](https://bbs.kanxue.com/elink@eedK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2E0K9h3y4J5L8%4y4G2k6Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6K6k6h3y4#2M7X3W2@1P5g2)9J5c8X3u0D9L8$3N6Q4x3V1j5J5x3o6p5%4i4K6u0r3x3e0u0Q4x3V1j5H3y4q4)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3X3c8V1k6h3k6W2L8X3c8W2M7W2)9J5k6r3q4@1M7q4)9J5k6r3#2S2j5$3S2A6L8X3g2Q4x3X3c8D9k6h3q4J5L8X3W2F1k6#2)9J5k6r3q4F1k6q4)9J5k6r3q4E0M7$3W2Q4x3X3c8#2L8X3g2S2M7Y4c8Z5K9h3&6Y4i4K6u0V1M7$3y4J5K9i4m8@1i4K6u0V1j5X3q4K6k6h3c8Q4x3X3c8S2N6s2c8S2j5$3E0K6i4K6u0V1N6r3S2S2N6q4)9J5k6r3I4A6N6X3g2Q4x3X3c8G2k6X3k6Q4x3X3c8@1K9r3g2Q4x3X3c8D9j5h3&6V1i4K6u0r3)，以及对恶意脚本的检测能力。

自从首次部署以来，该深度学习模型就可以高精度地检测到许多恶意和蓝军团队的 PowerShell 活动，其中有些活动是其他方法发现不了的。通过 PowerShell 获得的信息与各种机器学习模型以及 Microsoft Defender ATP 的信息结合在一起以检测网络攻击。

以下是恶意 PowerShell 脚本的例子，深度学习可以准确地检测到这些恶意脚本，但对于其他检测方法而言是很有挑战性的：

![图5. 高度混淆的恶意脚本](https://bbs.kanxue.com/upload/attach/202001/747358_XKNZ582RQ5PD6PN.jpg)

*图5 高度混淆的恶意脚本*

![图6. 下载并运行payload的混淆脚本](https://bbs.kanxue.com/upload/attach/202001/747358_VQE42GUXN4UVFDA.jpg)

*图6 下载并运行payload的混淆脚本*

![图7. 解密并执行恶意代码的脚本](https://bbs.kanxue.com/upload/attach/202001/747358_QY9NU544Q9BJ2XE.jpg)

*图7 解密并执行恶意代码的脚本*

## 通过深度学习强化 Microsoft Defender ATP

深度学习方法可极大提高威胁检测的能力。此文中，我们讨论了深度学习对一种特别容易绕过检测的威胁的具体应用，即检测恶意 PowerShell 脚本。我们已经并将继续在 [Microsoft Defender ATP](https://bbs.kanxue.com/elink@f96K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2E0K9h3y4J5L8%4y4G2k6Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6W2L8W2)9J5k6s2g2K6i4K6u0r3g2$3W2F1k6r3!0%4M7@1k6G2M7V1u0#2M7$3W2F1k6i4y4K6i4K6u0r3N6$3W2F1k6r3!0%4M7#2)9J5k6r3q4@1M7l9`.`.) 的多种功能中开发基于深度学习的防护。

用于网络防御的深度学习系统的开发和产品化需要大量的数据、计算、资源和工程工作。Microsoft Defender ATP 将从数百万个终端上收集的数据与 Microsoft 计算资源和算法结合在一起，以提供业界领先的攻击防御能力。

基于深度学习，可以增强对终端上的恶意 PowerShell 脚本和其他威胁的检测能力，意味着 [Microsoft Threat Protection](https://bbs.kanxue.com/elink@89fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2E0K9h3y4J5L8%4y4G2k6Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6W2L8W2)9J5k6s2g2K6i4K6u0r3M7$3g2U0N6i4u0A6N6s2W2Q4x3V1k6@1k6h3y4Z5L8X3!0D9L8$3N6&6i4K6u0r3N6r3S2J5k6h3q4@1i4K6u0V1M7s2u0G2N6r3g2U0N6r3W2G2L8R3`.`.) 可以提供更丰富、更深层次的安全性，Microsoft Threat Protection 能够为隐私、终端、电子邮件和数据、应用程序和基础设施提供全面的安全性支持。

原文链接：[ac1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2E0K9h3y4J5L8%4y4G2k6Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6K6k6h3y4#2M7X3W2@1P5g2)9J5c8X3u0D9L8$3N6Q4x3V1j5J5x3o6p5&6i4K6u0r3x3o6W2Q4x3V1j5H3x3#2)9J5c8X3c8W2k6i4m8Q4x3X3c8D9k6h3q4J5L8X3W2F1k6#2)9J5k6s2u0A6M7$3g2K6i4K6u0V1L8X3g2%4i4K6u0V1L8h3g2@1K9r3!0V1M7#2)9J5k6r3k6G2M7W2)9J5k6r3c8W2N6r3g2U0N6r3W2F1k6#2)9J5k6r3#2S2L8r3W2U0K9h3!0#2M7#2)9J5k6s2m8G2N6$3g2J5M7$3S2W2L8r3I4Q4x3V1j5`.](https://bbs.kanxue.com/elink@c6bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2E0K9h3y4J5L8%4y4G2k6Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6K6k6h3y4#2M7X3W2@1P5g2)9J5c8X3u0D9L8$3N6Q4x3V1j5J5x3o6p5&6i4K6u0r3x3o6W2Q4x3V1j5H3x3#2)9J5c8X3c8W2k6i4m8Q4x3X3c8D9k6h3q4J5L8X3W2F1k6#2)9J5k6s2u0A6M7$3g2K6i4K6u0V1L8X3g2%4i4K6u0V1L8h3g2@1K9r3!0V1M7#2)9J5k6r3k6G2M7W2)9J5k6r3c8W2N6r3g2U0N6r3W2F1k6#2)9J5k6r3#2S2L8r3W2U0K9h3!0#2M7#2)9J5k6s2m8G2N6$3g2J5M7$3S2W2L8r3I4Q4x3V1j5`.)

翻译：看雪翻译小组 SpearMint

校对：看雪翻译小组 fyb波

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
