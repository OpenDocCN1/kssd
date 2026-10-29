
# HTTP/3 的前世今生

HTTP 是为 Web 提供支持的应用层协议。它始于 1991 年的 HTTP/0.9 协议，到 1999 年已演变为 HTTP/1.1，由 IETF（互联网工程任务组）实现标准化。HTTP/1.1 已经使用了很长一段时间，但随着 Web 需求不断变化，所以需要更好的协议，在 2015 年出现了 HTTP/2。最近宣布 IETF 打算提供新的版本——HTTP/3。对某些人来说，可能不仅感到惊喜，也感到有些困惑。如果你没有密切地追踪 IETF 的工作，似乎 HTTP/3 出乎意料。但是，我们可以通过一系列实验和 Web 协议的演变来追溯它的起源，特别是 QUIC 传输协议。

如果你不熟悉 QUIC，我的同事已经在相关的方面写了一些文章。John 的<a href="elink@f2fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5$3I4G2N6h3c8X3L8r3q4J5k6g2)9J5k6h3y4G2L8g2)9J5c8Y4c8Z5k6g2)9J5k6s2q4#2K9h3y4W2L8X3W2F1k6#2)9J5c8R3`.`.">博客</a>描述了当今 HTTP 的一些现实世界的烦恼，Alessandro 的<a href="elink@9c6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5$3I4G2N6h3c8X3L8r3q4J5k6g2)9J5k6h3y4G2L8g2)9J5c8Y4c8Z5k6g2)9J5k6s2u0G2j5h3c8Q4x3X3c8@1L8#2)9J5k6s2q4#2K9h3y4Q4x3V1j5`.">博客</a>谈到了一些传输层协议的细节，而 Nick 的<a href="elink@4faK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5$3I4G2N6h3c8X3L8r3q4J5k6g2)9J5k6h3y4G2L8g2)9J5c8X3S2W2j5h3c8Q4x3X3c8K6N6r3q4J5N6q4)9J5k6s2N6A6N6r3S2Q4x3X3c8I4N6h3W2U0i4K6u0r3">博客</a>介绍了如何开展一些测试。所有这些文章都可以在 <mark class="encrypted">a0cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0L8r3!0#2k6r3k6D9j5i4u0W2i4K6u0V1M7i4g2A6j5#2)9J5k6h3y4G2L8b7`.`.</mark> 上找到。如果你感兴趣的话，可以看下我们用 Rust 编写的 QUIC 协议的开源实现——<a href="elink@3cfK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5$3I4G2N6h3c8X3L8r3q4J5k6g2)9J5k6h3y4G2L8g2)9J5c8X3g2F1K9X3!0&6i4K6u0V1j5g2)9J5k6s2y4D9K9h3y4W2i4K6u0V1L8$3k6Q4x3X3c8I4N6h3W2U0i4K6u0V1j5h3&6V1i4K6u0V1M7Y4g2K6N6q4)9J5c8R3`.`.">quiche</a>。

HTTP/3 是映射到 QUIC 传输层的 HTTP 应用程序。这个名称在最近的草案版本 17（<a href="elink@165K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1L8$3!0D9M7#2)9J5k6h3W2W2N6r3k6Q4x3X3g2G2M7X3N6Q4x3V1k6Z5N6r3#2D9i4K6u0r3k6s2u0S2k6Y4c8Q4x3X3c8A6k6i4c8X3i4K6u0V1M7i4g2A6j5#2)9J5k6r3S2@1N6s2m8Q4x3X3b7I4y4H3`.`.">draft-ietf-quic-http-17</a>）中正式提出。草案于 2018 年 10 月底提出，11 月在曼谷举行的 IETF 103 会议期间进行了讨论并达成初步共识。HTTP/3 最初来源于 SPDY over gQUIC，之后是 HTTP/2 over gQUIC，再后来是 HTTP/2 over QUIC，最后才是 HTTP over QUIC，而现在被称为 HTTP/3。其实，HTTP/3 只是一种新的 HTTP 语法，可以在 IETF-QUIC（一种基于 UDP 的多路传输和安全传输）上工作。

在这篇博客文章中，我们将探讨一些 HTTP/3 之前的名称由来，并介绍最近更改名称的缘由。我们将回到 HTTP 的早期阶段，并且沿路探寻这期间所有的优秀工作。如果您希望全面了解整个过程，可以跳到文章末尾或打开这个<a href="elink@5d8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5$3I4G2N6h3c8X3L8r3q4J5k6g2)9J5k6h3y4G2L8g2)9J5c8X3y4G2L8Y4c8W2L8Y4c8Q4x3V1k6A6L8h3q4Y4k6i4y4Q4x3V1j5J5x3o6p5&6i4K6u0r3x3o6q4Q4x3V1k6%4k6h3u0Q4y4h3k6@1K9h3#2W2L8r3W2F1k6g2)9#2k6X3I4S2M7X3N6W2x3g2)9J5k6i4y4$3k6H3`.`.">非常详细的 SVG 版本</a>。

![img](upload/attach/202001/726271\_Y984MY7QS5HVXHE.jpg) HTTP/3 蛋糕图表示

## 设置场景

在我们关注 HTTP 之前，要注意有两个协议共享了 QUIC 的名称。正如我们<a href="elink@147K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5$3I4G2N6h3c8X3L8r3q4J5k6g2)9J5k6h3y4G2L8g2)9J5c8Y4c8Z5k6g2)9J5k6s2u0G2j5h3c8Q4x3X3c8@1L8#2)9J5k6s2q4#2K9h3y4Q4x3V1j5`.">前面</a>所解释的，gQUIC 通常用于标识 Google-QUIC（原始协议），而 QUIC 通常用于表示与 gQUIC 不同的 IETF 标准进行中的版本。

自 90 年代初期以来，网络的需求已经发生了变化。我们已经有了新版本的 HTTP，并以传输层安全性（TLS）的形式添加了用户安全性。我们只会在这篇文章中涉及到 TLS，如果您想更详细地了解这一领域，我们的其他<a href="elink@f03K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5$3I4G2N6h3c8X3L8r3q4J5k6g2)9J5k6h3y4G2L8g2)9J5c8Y4c8S2k6#2)9J5c8Y4c8D9M7#2)9J5c8R3`.`.">博客文章</a>是一个很好的资源。

为了有助于我解释 HTTP 和 TLS 的历史，我开始整理协议规范和日期的详细信息。这些信息通常以文本形式显示，如按日期排序的项目符号点列表，说明文档标题。然而，存在分支标准，时间上有重叠，并且一个简单的列表有时候并不能表达关系的真正复杂性。在 HTTP 中，有一些并行工作可以重构核心协议定义以便于使用，扩展协议以供新用途，并重新定义协议如何在 Internet 上交换数据以获得性能。当你试图在近 30 年的互联网历史中跨越不同分支的工作流加入这些点时，你需要可视化。所以我做了一个 CloudFlare 安全 Web 时间表。（注：从技术上讲，它是一个<a href="elink@bbfK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8W2)9J5k6i4N6A6K9$3W2H3k6h3c8A6j5g2)9J5k6h3!0J5k6#2)9J5c8Y4N6A6K9$3W2Q4x3V1k6o6L8r3q4V1L8$3N6J5j5h3@1`.">进化树</a>，但术语“时间表”更广为人知）。

I have applied some artistic license when creating this, choosing to focus on the successful branches in the IETF space. Some of the things not shown include efforts in the W3 Consortium [HTTP-NG](https://www.w3.org/Protocols/HTTP-NG/) working group, along with some exotic ideas that their authors are keen on explaining how to pronounce: <a href="elink@7ffK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2K9X3N6U0i4K6u0W2L8%4u0Y4i4K6u0r3x3U0l9I4x3W2)9J5c8U0p5J5i4K6u0r3M7%4m8W2k6h3c8A6L8X3N6Q4x3X3c8#2M7q4)9J5k6r3S2@1N6s2m8Q4x3X3c8%4K9i4c8Z5i4K6u0V1L8h3W2F1K9h3#2S2L8q4)9J5k6s2m8J5L8%4c8G2j5$3!0D9i4K6u0W2K9s2c8E0L8l9`.`.">HMURR (pronounced 'hammer')</a> and <a href="elink@667K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6t1g2q4c8b7g2$3!0J5K9%4y4Z5L8%4m8Q4x3V1k6%4L8%4u0C8M7$3S2G2M7o6t1H3x3e0N6Q4x3V1k6T1L8r3!0T1i4K6u0r3L8h3q4K6N6r3g2J5i4K6u0r3N6r3q4D9K9%4y4Q4x3V1k6%4j5h3E0S2i4K6u0W2M7r3c8X3">WAKA (pronounced “wah-kah”)</a>.

我在创建时申请了创作者授权，选择专注于 IETF 空间中成功的分支。一些未显示的内容包括 W3 联盟 [HTTP-NG](https://www.w3.org/Protocols/HTTP-NG/) 工作组的努力，以及他们的作者热衷于解释如何发音的一些奇特的想法：<a href="elink@174K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2K9X3N6U0i4K6u0W2L8%4u0Y4i4K6u0r3x3U0l9I4x3W2)9J5c8U0p5J5i4K6u0r3M7%4m8W2k6h3c8A6L8X3N6Q4x3X3c8#2M7q4)9J5k6r3S2@1N6s2m8Q4x3X3c8%4K9i4c8Z5i4K6u0V1L8h3W2F1K9h3#2S2L8q4)9J5k6s2m8J5L8%4c8G2j5$3!0D9i4K6u0W2K9s2c8E0L8l9`.`.">HMURR（发音为 'hammer'）</a> 和 <a href="elink@f70K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6t1g2q4c8b7g2$3!0J5K9%4y4Z5L8%4m8Q4x3V1k6%4L8%4u0C8M7$3S2G2M7o6t1H3x3e0N6Q4x3V1k6T1L8r3!0T1i4K6u0r3L8h3q4K6N6r3g2J5i4K6u0r3N6r3q4D9K9%4y4Q4x3V1k6%4j5h3E0S2i4K6u0W2M7r3c8X3">WAKA（发音为“wah-kah”）</a>。

在接下来的几节中，我将按照这个时间表来解释 HTTP 历史中的关键章节。当你理解为什么标准化是有益的，以及 IETF 是如何处理它的之后，你会更好地享受这篇文章的内容。因此，在回到时间轴之前，首先简要介绍该主题。如果您已熟悉 IETF，可以直接跳过下一部分。

## 互联网标准的类型

通常，标准定义了共同的参考术语、范围、约束、适用性和其他考虑因素。标准形态各异，可以是非正式的（也就是事实上的）或正式的（由标准定义组织，例如 IETF、ISO 或 MPEG，同意或发布）。许多领域都存在标准，甚至有一个正式的英国制茶标准——BS 6008。

早期 Web 使用在 IETF 外部发布的 HTTP 和 SSL 协议定义，这些定义在安全 Web 时间表上标记为**红线**。客户端和服务器采用这些协议，从而使它们成为事实上的标准。

在某些时候，决定将这些协议正式化（一些激励性的原因将在后面的部分中描述）。互联网标准通常在 IETF 中定义，它遵循“粗略共识和运行代码”的非正式原则。这是基于在互联网上开发和部署内容的经验，而这恰恰与尝试在纯理论中开发完美方案的方法形成了对比。

IETF 互联网标准通常称为 RFC。这个区域解释起来很复杂，因此我建议您阅读由 QUIC 工作组联合主席 Mark Nottingham 撰写的博客文章“<a href="elink@f11K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2A6k6i4c8X3i4K6u0W2L8%4u0Y4i4K6u0r3j5X3I4G2k6#2)9J5c8X3S2G2N6#2)9J5k6s2u0W2j5h3c8Q4x3X3c8J5k6X3y4Q4x3V1j5`.">如何阅读 RFC</a>”。工作组，或者说 WG，就是一个邮件列表。

IETF 每年举行三次会议，为所有工作组提供必要的时间和设施，如果他们愿意，可以亲自见面讨论。这几周的议程可能会非常繁忙，没有充足的时间可以深入讨论这些高度技术性的领域。为了解决这个问题，一些工作组选择在 IETF 全体会议之间的几个月内举行临时会议，这有助于保持规范开发的势头。自 2017 年以来，QUIC WG 已举行了几次临时会议，<a href="elink@874K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1j5i4c8S2N6s2u0S2j5$3E0W2M7W2)9J5k6h3W2W2N6r3k6Q4x3X3g2G2M7X3N6Q4x3V1k6%4k6#2)9J5c8Y4q4#2K9h3y4Q4x3V1k6E0k6h3g2@1K9h3&6Y4M7#2)9J5c8R3`.`.">会议页面</a>上提供了完整的列表。

这些 IETF 会议还为其他与 IETF 相关的人员会议提供了机会，例如<a href="elink@014K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2A6j5h3u0Q4x3X3g2G2M7X3N6Q4x3V1j5`.">互联网架构委员会</a>或<a href="elink@670K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6M7Y4c8X3i4K6u0W2L8%4u0Y4i4K6u0r3">互联网研究工作组</a>。近年来，在 IETF 会议之前的周末会举办一场<a href="elink@9deK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2A6k6i4c8X3i4K6u0W2L8%4u0Y4i4K6u0r3K9r3!0%4i4K6u0r3M7Y4g2F1L8X3W2F1k6$3y4G2k6r3g2Q4x3V1k6Z5j5h3y4C8j5i4c8Z5L8$3&6K6i4K6u0r3">IETF 黑客大会</a>。这为社区提供了开发运行代码的机会，更重要的是，可以和其他人在一起进行互操作性测试。这有助于在接下来的几天中找到可以讨论的规范中的问题。

这篇博客的目的是要让你理解重要的是 RFC 不是突然出现的。相反，他们通常会经历一个以 IETF 互联网草案（I-D）格式开始的过程。在已经发布了规范的情况下，准备 I-D 可能只是一个简单的重新格式化操作。自发布之日起，I-D 的有效期为 6 个月。为了保持活跃状态，需要持续发布新版本。在实践中，让一个 I-D 消失并没有什么后果，因为它经常发生。这些文档将继续托管在<a href="elink@f08K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1j5i4c8S2N6s2u0S2j5$3E0W2M7W2)9J5k6h3W2W2N6r3k6Q4x3X3g2G2M7X3N6Q4x3V1k6V1L8$3y4Q4x3V1k6J5k6h3y4W2L8Y4b7`.">IETF 文档</a>的网站上，供任何想要阅读它们的人使用。

I-D 在安全 Web 时间表上表示为**紫色线条**。每一个都有一个唯一的名称，格式为 `draft-{作者姓名}-{工作组}-{主题}-{版本}`。工作组字段是可选的，它可能会预测 IETF WG 将在该文件上工作，有时这会发生变化。如果 IETF 采用 I-D，或者 I-D 直接在 IETF 内启动，则名称为 `draft-ietf-{工作组}-{主题}-{版本}`。I-D 可能会分支、合并或消亡。版本从 `00` 开始，每次发布新草案时增加 `1`。例如，I-D 的第 4 个草案版本号为 `03`。每当 I-D 更改名称，其版本将重置为 `00`。
