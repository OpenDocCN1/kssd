
- [`draft-hamilton-quic-transport-protocol-00`](https://bbs.kanxue.com/elink@e4aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1L8$3!0D9M7#2)9J5k6h3W2W2N6r3k6Q4x3X3g2G2M7X3N6Q4x3V1k6Z5N6r3#2D9i4K6u0r3k6s2u0S2k6Y4c8Q4x3X3c8Z5j5h3#2A6L8s2c8G2L8W2)9J5k6s2q4#2K9h3y4Q4x3X3c8@1M7X3q4F1M7%4m8G2M7Y4c8Q4x3X3c8H3M7X3!0@1L8$3y4G2L8q4)9J5k6o6l9H3)
- [`draft-thomson-quic-tls-00`](https://bbs.kanxue.com/elink@67aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1L8$3!0D9M7#2)9J5k6h3W2W2N6r3k6Q4x3X3g2G2M7X3N6Q4x3V1k6Z5N6r3#2D9i4K6u0r3k6s2u0S2k6Y4c8Q4x3X3c8@1K9r3!0E0M7$3!0F1i4K6u0V1M7i4g2A6j5#2)9J5k6s2c8D9M7#2)9J5k6o6l9H3)
- [`draft-iyengar-quic-loss-recovery-00`](https://bbs.kanxue.com/elink@b97K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1L8$3!0D9M7#2)9J5k6h3W2W2N6r3k6Q4x3X3g2G2M7X3N6Q4x3V1k6Z5N6r3#2D9i4K6u0r3k6s2u0S2k6Y4c8Q4x3X3c8A6P5h3g2F1k6$3q4J5i4K6u0V1M7i4g2A6j5#2)9J5k6r3I4G2M7%4y4Q4x3X3c8J5k6h3y4G2N6X3g2J5P5g2)9J5k6o6l9H3)
- [`draft-shade-quic-http2-mapping-00`](https://bbs.kanxue.com/elink@d51K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1L8$3!0D9M7#2)9J5k6h3W2W2N6r3k6Q4x3X3g2G2M7X3N6Q4x3V1k6Z5N6r3#2D9i4K6u0r3k6s2u0S2k6Y4c8Q4x3X3c8K6K9r3q4V1k6g2)9J5k6s2q4#2K9h3y4Q4x3X3c8Z5N6s2c8H3x3W2)9J5k6r3#2S2M7s2m8A6L8X3N6Q4x3X3b7H3x3l9)

这里是另一个关于 http 和 quic 的困惑来源。[`draft-shade-quic-http2-mapping-00`](https://bbs.kanxue.com/elink@d51K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1L8$3!0D9M7#2)9J5k6h3W2W2N6r3k6Q4x3X3g2G2M7X3N6Q4x3V1k6Z5N6r3#2D9i4K6u0r3k6s2u0S2k6Y4c8Q4x3X3c8K6K9r3q4V1k6g2)9J5k6s2q4#2K9h3y4Q4x3X3c8Z5N6s2c8H3x3W2)9J5k6r3#2S2M7s2m8A6L8X3N6Q4x3X3b7H3x3l9) 的标题是“使用QUIC传输协议的HTTP/2语义”，它将自己描述为“ QUIC 上 HTTP/2语义的映射”。然而，这是用词不当。 HTTP/2是关于在保持语义的同时改变语法。此外，由于我之前概述的原因，“基于 gQUIC 的 HTTP/2 ”从未对语法进行过准确描述。记住这些想法。

这个 IETF 版本的 QUIC 是一个全新的传输协议。这是一项艰巨的任务，在首先投入这些承诺之前，IETF 喜欢衡量其成员的实际利益。为此，2016年在柏林举行的IETF 96会议上举行了正式的 “[物以类聚](https://bbs.kanxue.com/elink@12aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2A6k6i4c8X3i4K6u0W2L8%4u0Y4i4K6u0r3K9r3!0%4i4K6u0r3j5X3!0X3M7#2)9J5c8R3)”会议。我很幸运能够亲自参加会议而[幻灯片](https://bbs.kanxue.com/elink@092K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1j5i4c8S2N6s2u0S2j5$3E0W2M7W2)9J5k6h3W2W2N6r3k6Q4x3X3g2G2M7X3N6Q4x3V1k6E0k6h3g2@1K9h3&6Y4i4K6u0r3z5e0k6Q4x3V1k6E0j5i4c8W2M7X3W2S2L8s2y4Q4x3V1k6K6L8r3W2V1k6i4y4Q4x3X3b7&6y4W2)9J5k6s2q4#2K9h3y4Q4x3X3b7H3)并不能证明这一点。如Adam Roach的[照片](https://bbs.kanxue.com/elink@257K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2X3L8r3W2U0K9%4u0Q4x3X3g2U0L8$3#2Q4x3V1k6H3K9r3!0@1L8%4y4Q4x3V1k6S2k6r3q4E0i4K6u0V1M7X3!0S2j5$3S2Q4x3V1j5J5z5o6x3@1x3K6M7&6y4U0M7J5x3W2)9J5c8X3W2F1i4K6u0r3M7r3S2G2N6r3!0K6N6s2u0W2j5h3#2Q4x3V1j5)所示，数百人参加了会议。在会议结束时达成了共识; QUIC 将被 IETF 采用并标准化。

第一个用于将HTTP映射到QUIC的 IETF QUIC I-D是[`draft-ietf-quic-http-00`](https://bbs.kanxue.com/elink@c60K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1L8$3!0D9M7#2)9J5k6h3W2W2N6r3k6Q4x3X3g2G2M7X3N6Q4x3V1k6Z5N6r3#2D9i4K6u0r3k6s2u0S2k6Y4c8Q4x3X3c8A6k6i4c8X3i4K6u0V1M7i4g2A6j5#2)9J5k6r3S2@1N6s2m8Q4x3X3b7H3x3l9)，采用 Ronseal 方法并将其名称简化为“HTTP over QUIC”。不幸的是，它没有完全完成工作，并且在整个机构中存在许多HTTP/2术语的实例。 I-Ds新编辑Mike Bishop发现了这一点，并开始修复 HTTP/2 的误称。在01草案中，描述改为“ HTTP 语义在 QUIC 上的映射”。

随着时间和版本的推移，术语“http/2”的使用逐渐减少，这些实例变成了对 [RFC7540](https://bbs.kanxue.com/elink@8aaK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1L8$3!0D9M7#2)9J5k6h3W2W2N6r3k6Q4x3X3g2G2M7X3N6Q4x3V1k6Z5N6r3#2D9i4K6u0r3M7X3k6U0y4K6f1@1x3l9) 部分的引用。向前推进两年到2018年10月，ID 版本 是16。尽管quic上的HTTP与HTTP/2存在相似之处，但它最终是一种独立的、不向后兼容的HTTP语法。然而，对于那些不密切跟踪IETF发展的人（地球人口的很大一部分），文档名称并没有捕捉到这种差异。标准化的一个要点是帮助沟通和互操作。然而，像命名这样简单的事情是社区混乱的主要原因。

回想一下2012年所说的“HTTP / 2.0仅表示线上与HTTP / 1.x 不兼容的那部分”。 IETF 遵循现有的提示。在 IETF 103 之前和期间经过深思熟虑后，达成了共识，将“HTTP over QUIC”重命名为 HTTP/3。世界现在处于一个更好的地方，我们可以继续进行更重要的辩论。

## 但是，RFC 7230 和 7231 与 您对语义和语法的定义不一致！

有时文档标题可能会令人困惑。目前描述语法和语义的HTTP文档有：

- [RFC 7230](https://bbs.kanxue.com/elink@e6aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1L8$3!0D9M7#2)9J5k6h3W2W2N6r3k6Q4x3X3g2G2M7X3N6Q4x3V1k6Z5N6r3#2D9i4K6u0r3M7X3k6U0y4K6t1K6x3l9) - 超文本传输协议（HTTP / 1.1）：消息语法和路由
- [RFC 7231](https://bbs.kanxue.com/elink@d41K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1L8$3!0D9M7#2)9J5k6h3W2W2N6r3k6Q4x3X3g2G2M7X3N6Q4x3V1k6Z5N6r3#2D9i4K6u0r3M7X3k6U0y4K6t1K6x3b7) - 超文本传输协议（HTTP / 1.1）：语义和内容

太多地阅读这些名称，就会认为基本的HTTP语义特定于HTTP的版本，即HTTP / 1.1。但是，这是HTTP系列树的意外副作用。好消息是 HTTPbis 工作组正试图解决这个问题。一些勇敢的成员正在进行另一轮的文件修订，正如罗伊菲尔丁所说，“再来一次！”。这项工作正在进行中，被称为HTTP核心活动（您可能也在名称HTTPtre或HTTPter下听说过这个，命名很难）。这将把六个草案缩减为三个：

- HTTP语义（`draft-ietf-httpbis-semantics`）
- HTTP缓存（`draft-ietf-httpbis-caching`）
- HTTP / 1.1消息语法和路由（`draft-ietf-httpbis-messaging`）

在这种新结构下，HTTP/2 和 HTTP/3 显然是通用HTTP语义的语法定义。这并不意味着它们除了语法之外没有自己的特性，但它应该有助于框架讨论的进行。

## 总结一下

这篇文章简要介绍了在过去的三十年时间中 IETF 中 HTTP 的标准化过程。 在没有触及许多技术细节的情况下，我试图解释我们今天如何是到达 HTTP/3 的。 如果您跳过中间的某些细节部分来寻找总结性的话语，那么它是：HTTP/3 只是一种新的HTTP语法，在 IETF-QUIC（一种基于UDP的多路传输和安全传输）上工作。我们下次可以进一步探索一些许多其他有趣的技术领域。

在这篇文章中，我们探讨了HTTP和TLS开发中的重要章节，但这些章节是孤立的。 我们通过将它们全部整合到下面提供的完整安全 Web 时间表中来结束这篇博客。 您可以使用它来自行调查详细的历史记录。 对于那些喜欢刨根问底的人，请务必查看完整版本，包括草稿编号。

![img](https://bbs.kanxue.com/upload/attach/202001/726271_XSEC6C44MDPZUZY.jpg)

原文链接：[cdcK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5$3I4G2N6h3c8X3L8r3q4J5k6g2)9J5k6h3y4G2L8g2)9J5c8X3S2@1N6s2m8Q4x3X3b7K6i4K6u0V1k6Y4u0G2L8g2)9J5k6s2u0G2L8%4c8Q4x3X3c8@1L8#2)9J5k6s2c8A6M7q4)9J5c8R3](https://bbs.kanxue.com/elink@980K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5$3I4G2N6h3c8X3L8r3q4J5k6g2)9J5k6h3y4G2L8g2)9J5c8X3S2@1N6s2m8Q4x3X3b7K6i4K6u0V1k6Y4u0G2L8g2)9J5k6s2u0G2L8%4c8Q4x3X3c8@1L8#2)9J5k6s2c8A6M7q4)9J5c8R3)

编译： 看雪翻译小组 fyb 波

校对： 看雪翻译小组 欢歌笑语

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
