
关于将这些标记添加回可见的 QUIC 头部已经有[很多讨论](https://bbs.kanxue.com/elink@f64K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6&6L8%4g2@1N6g2)9J5k6h3u0W2i4K6u0r3g2q4q4I4y4W2Z5@1i4K6g2X3d9p5u0S2h3g2)9K6c8Y4c8Q4x3@1b7I4x3K6f1%4)，但最终结果是，只有 1 bit 用于 RTT 测算—— spin bit。这个“轮转位”的概念的是这样的：在每次往返时改变一次值，从而使得网络中间设备可以观察到变化并以这种方式估算 RTT，参见 图 2。虽然有点帮助，但是仍然大大地限制了运营商，特别是初始标识是 [Chrome 和 Firefox 不支持的“轮转位”](https://bbs.kanxue.com/elink@bf6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2L8r3W2@1k6i4y4H3k6h3g2V1N6r3g2U0K9q4)9J5k6h3y4G2L8g2)9J5c8U0t1H3x3e0S2Q4x3V1j5I4x3g2)9J5c8U0p5J5i4K6u0r3K9h3g2@1k6W2)9J5k6o6p5H3x3#2)9J5k6r3S2@1N6s2m8Q4x3X3c8G2N6X3g2J5i4K6u0V1M7i4g2A6j5#2)9J5k6r3W2K6i4K6u0V1L8X3!0%4i4K6u0V1K9s2c8@1M7q4)9J5k6o6y4Q4x3V1j5`.)。QUIC 唯一支持的其他选项是“[显式拥塞提醒](https://bbs.kanxue.com/elink@94aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8W2)9J5k6i4N6A6K9$3W2H3k6h3c8A6j5g2)9J5k6h3!0J5k6#2)9J5c8Y4N6A6K9$3W2Q4x3V1k6q4P5s2m8D9K9h3y4A6N6q4)9#2k6V1y4G2L8X3N6W2M7%4c8A6L8$3&6Q4y4h3k6z5L8%4c8A6k6X3W2U0j5i4c8A6L8$3^5`.)”，它采用 IP 级别的标识来表示拥塞。

![图片描述](https://bbs.kanxue.com/upload/attach/202001/93_5JJAVPXKK2PASXQ.png)

图 2：“轮转位”工作原理简述

### UDP 拦截、alt-svc

不知道你会怎样做？但如果我是一个网络运营商，如果我正在进行 TCP 网络优化或者在应用特殊的安全措施，我会非常想要彻底地封锁 QUIC。对于网页浏览来说，拦截 443 端口上的 UDP 流量，并不会有什么问题。在开发 QUIC 时，[Google 也认识到了该问题](https://bbs.kanxue.com/elink@3efK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2A6k6i4c8X3i4K6u0W2L8%4u0Y4i4K6u0r3M7s2u0G2j5$3g2W2k6r3W2F1k6%4y4Q4x3V1j5&6y4W2)9J5c8Y4y4D9K9h3c8W2M7#2)9J5c8Y4y4D9K9h3c8W2M7#2)9J5k6o6V1$3i4K6u0V1M7i4g2A6j5#2)9J5k6o6y4Q4x3X3g2H3k6r3j5`.)。为了搞清楚当前网络对 UDP/QUIC 的封锁情况，他们研究得出 [3-5% 的网络](https://bbs.kanxue.com/elink@a2bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1L8$3!0D9M7#2)9J5k6h3W2W2N6r3k6Q4x3X3g2G2M7X3N6Q4x3V1k6A6k6q4)9J5c8X3c8J5j5h3k6@1i4K6u0V1K9h3g2@1k6W2)9J5k6s2q4#2K9h3y4Q4x3X3c8S2M7s2m8D9K9h3y4S2j5X3W2D9K9i4c8&6i4K6u0V1x3o6y4Q4x3X3g2Z5N6r3#2D9i4K6t1K6k6X3q4D9L8r3u0S2j5$3D9`.)不允许 QUIC 流量通过。结果看起来还不错，不过该数据并未包含不计其数的企业网络；此外，更重要的问题是，这种状况还会一直延续下去吗？如果 QUIC 应用的更加广泛后，这些网络会停止对 QUIC 的主动拦截吗（至少是在他们更新他们的防火墙和网络管理工具后）？

在使用 quick-tracker conformance 工具测试我们自己的 [QUIC 实现](https://bbs.kanxue.com/elink@3dcK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6J5L8h3q4J5P5q4)9J5c8Y4q4#2K9h3y4C8k6i4t1`.)时，发生一个“有趣”的小故事：当测试一台在加拿大的服务器时，大部分测试突然失效。更进一步测试发现，一些 IP 段主动阻塞 QUIC 流量，导致了测试的失败。

拦截 QUIC 流量并不会对浏览网页的终端用户带来任何影响--网站照常访问。因为，无论如何浏览器（和服务器）必须要解决 UDP 阻塞问题。为此，他们总是把 TCP 留为备用措施，而不是干等着 QUIC 连接 timeout。

服务器将采用 [`alt-svc`](https://bbs.kanxue.com/elink@0f7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0N6i4u0D9i4K6u0W2K9r3q4^5P5q4)9J5k6i4y4W2i4K6u0r3N6X3W2V1k6h3!0Q4x3V1k6U0N6i4u0D9N6i4m8Q4x3X3b7J5x3o6p5^5i4K6u0r3x3U0l9I4z5q4)9J5k6o6l9@1i4K6u0V1x3e0g2Q4y4h3k6x3N6h3y4S2M7#2)9#2k6W2m8S2M7X3c8#2k6g2)9J5k6r3q4D9N6s2y4$3j5#2)9J5k6r3k6G2M7W2)9J5k6s2c8Z5k6g2)9J5k6s2m8W2L8%4m8D9k6g2)9J5k6i4N6W2j5X3@1`.) 机制提供对 QUIC 的支持，但浏览器只能在一定程度上信任它；因为快速变化的网络环境意味着 QUIC 随时有可能被拦截。在 QUIC 流量被拦截的企业网络里，网络管理员无需处理用户的各种问题同时还可以维持对网络的有力控制，他们何乐而不为呢？[他们还无需为此额外地维护一个独立的 QUIC/HTTP3 协议栈](https://bbs.kanxue.com/elink@6f8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6i4K6u0W2K9%4W2E0i4K6u0V1j5$3c8F1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6@1M7X3W2W2M7#2)9J5c8X3W2U0L8$3&6K6i4K6u0r3k6X3q4U0k6h3u0G2L8$3E0Q4x3V1j5H3x3o6m8Q4x3V1j5H3x3e0c8Q4x3V1j5@1z5e0W2Q4x3V1k6K6M7r3q4U0K9$3I4W2M7W2)9J5k6h3A6H3k6H3`.`.)。

最后，有人可能会问了：为什么这些巨头比如 Google 即使是极其不容易的情况下，还要开发部署 QUIC 协议呢？在我看来，像 Google 这样的巨头公司对他们从服务器到各个边缘网络的链路，有着绝对的掌控，同时，还与其他网络运营商有联系。他们或多或少是知道网络上的问题的，并通过调整负载均衡，路由和服务器来缓解这些问题。为了解决这些问题，在开发 QUIC 时，他们可能会做点手脚，比如在 QUIC 的非加密部分编码一些信息 -- 连接 ID。该 [18 字节长的域](https://bbs.kanxue.com/elink@0d8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6I4N6h3W2U0N6$3N6Q4x3V1k6T1j5i4y4W2i4K6u0V1k6s2u0S2k6Y4c8K6i4K6u0r3M7s2g2D9L8q4)9J5c8U0p5H3z5o6R3`.)可以[编码负载均衡信息在里头](https://bbs.kanxue.com/elink@e30K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1L8$3!0D9M7#2)9J5k6h3W2W2N6r3k6Q4x3X3g2G2M7X3N6Q4x3V1k6A6k6q4)9J5c8X3c8J5j5h3k6@1i4K6u0V1K9h3g2@1k6W2)9J5k6s2q4#2K9h3y4Q4x3X3c8S2M7s2m8D9K9h3y4S2j5X3W2D9K9i4c8&6i4K6u0V1x3o6y4Q4x3X3g2Z5N6r3#2D9i4K6t1K6M7$3g2J5N6X3g2J5i4K6u0V1k6$3g2F1k6i4u0S2N6r3g2V1i4K6u0V1j5$3!0F1L8X3g2U0N6r3W2G2L8W2)9J5k6r3W2V1)。他们还可以设想为其数据包添加额外的包头，一旦流量离开企业网络就将其剥离。所以，这样看来，在这个游戏中，这些巨头可能会失去一些，但是那些服务器提供商或是小公司会失去更多。

### 反对观点

- 由于其优秀的性能，终端用户迫切希望用上 QUIC ；
- 基于其自身的拥塞控制及快速启动配置，QUIC 并不需要性能增强的中间设备；
- 大部分网络并没有阻挡 QUIC 流量，大的变革同时也意味着大的机会；
- 运行 QUIC + HTTP/3 只需在现有的 TCP + HTTP/2 的服务器配置文件添加若干行配置即可。

## 2. CPU 问题

到目前为止， QUIC 协议完全是在用户层实现的（而 TCP 协议是典型的运行在操作系统内核空间的）。在用户层实现，可以方便地更新、迭代，而无需升级内核，但同时也带来严重的性能开销（主要是用户层到内核空间的通信）和[潜在的安全问题](https://bbs.kanxue.com/elink@cd8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1N6$3W2@1N6r3g2J5i4K6u0W2j5$3!0E0i4K6u0r3M7X3g2&6K9$3k6D9L8$3g2@1k6i4u0Q4x3V1k6K6N6r3q4@1N6i4y4Q4x3V1j5I4x3o6j5@1y4K6V1@1y4U0f1J5x3o6j5#2x3K6j5I4z5e0t1H3)。

在他们发表的[研究论文](https://bbs.kanxue.com/elink@afcK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8q4)9J5k6h3q4U0L8g2)9J5k6h3!0J5k6#2)9J5c8X3y4A6N6r3q4@1K9h3!0F1i4K6u0W2j5$3k6E0i4K6y4r3K9h3c8Q4x3@1b7K6x3o6V1^5z5o6b7J5)中，Google 表示，相较于同等条件下的 TCP+TLS 协议栈，他们在服务器端的 QUIC 协议栈，需要 2 倍于前者的 CPU 开销。这已经是在经过优化的内核上的较好的结果。换句话来说，为了提供同等质量的网络，至少需要 2 倍性能的硬件。他们也提到 QUIC 协议在移动设备上性能降低的问题，不过并未提供相关数据。还好，我们找到了[另外一份关于移动设备上的测试报告](https://bbs.kanxue.com/elink@e28K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3c8S2N6X3W2V1i4K6u0W2j5$3S2G2k6X3k6F1k6i4y4Q4x3X3g2U0L8$3#2Q4x3V1k6H3N6h3u0K6i4K6u0r3L8r3!0F1k6#2)9J5k6r3I4G2L8$3E0Q4x3X3c8S2N6q4)9J5k6s2q4#2K9h3y4Q4x3X3c8A6L8h3x3I4y4#2)9J5k6i4m8V1k6R3`.`.)。该报告研究显示， Google 的 QUIC 比 TCP 快，只是在不同机型上这一优势并不明显，见图 3。这主要是因为 QUIC 的拥塞控制是基于限制应用程序时间 58%（vs 桌面设备的 7%），这意味着 CPU 不能处理数目过大的接入数据包。

![图片描述](https://bbs.kanxue.com/upload/attach/202001/93_8A33MTFUJG8HRQB.png)

图 3：性能 QUIC vs TCP 。红色 = QUIC 表现较好, 蓝色 = TCP 表现较好。

看来 QUIC 将为 TCP 网络不好的##高端##设备带来改善的机会。不幸的是，[不良的网络往往是由于落后的硬件设备](https://bbs.kanxue.com/elink@ad3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8X3k6J5k6i4q4#2k6h3&6@1L8s2W2Q4x3X3g2G2M7X3N6Q4x3V1j5J5x3o6p5%4i4K6u0r3x3e0m8Q4x3V1k6U0j5h3&6Q4x3X3c8&6L8%4g2Q4x3X3c8S2k6X3k6G2M7X3c8Q4x3X3c8A6N6q4)9J5k6s2u0W2j5h3I4Q4x3X3c8%4L8%4u0D9k6q4)9J5k6s2N6W2j5W2)9J5k6s2m8W2M7X3k6G2M7X3#2S2L8X3y4W2i4K6u0V1j5Y4g2V1k6$3g2@1M7#2)9J5c8R3`.`.)。那么，采用 QUIC 协议为网络带来的改进，将被缓慢的硬件抵消。结合现在的网页也越来越消耗 CPU 资源的事实（[导致网页的整体性能越来越依赖于 JavaScript 的性能而非网络性能](https://bbs.kanxue.com/elink@c6aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2&6L8%4g2@1N6h3u0W2i4K6u0W2j5$3!0E0i4K6u0r3N6$3q4@1j5$3S2Q4x3@1k6$3i4K6y4p5f1Y4N6e0L8s2g2T1g2p5u0F1k6i4M7`.)），这确实是个难题。

### IoT 和 TypeScript

QUIC 另一个常被吹嘘的应用场景是物联网（IoT）设备，因为这些设备经常需要间歇性的（蜂窝）网络接入，而 QUIC 的[低延迟启动、0-RTT和良好的传输可靠性,看起来很适合物联网设备](https://bbs.kanxue.com/elink@df2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2M7Y4S2A6N6W2)9J5k6h3!0J5k6#2)9J5c8X3q4T1M7#2)9J5c8U0p5^5x3e0m8Q4x3X3f1H3y4K6M7K6x3l9`.`.)。但是，这些设备的 CPU 通常相当的慢。尽管，据我所知目前尚未有在该类设备上测试的 QUIC 协议栈，但 QUIC 的设计者提到，在物联网应用场景下仍存在很多的[问题](https://bbs.kanxue.com/elink@838K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6I4N6h3W2U0N6$3N6Q4x3V1k6T1j5i4y4W2i4K6u0V1k6s2u0S2k6Y4c8K6i4K6u0r3K9i4y4K6N6h3g2K6i4K6y4r3N6i4c8X3z5q4)9K6c8q4)9J5y4f1f1J5i4K6t1#2z5f1y4Q4x3U0f1&6x3#2)9J5y4X3q4E0M7q4)9K6b7Y4q4Q4x3@1c8A6M7#2)9J5y4e0y4m8K9i4y4K6N6h3g2Q4x3V1u0A6L8%4b7`.)，任何一个决定都有可能影响到 QUIC 性能表现。类似的，不少人提到了硬件实现 QUIC 的想法，不过，据我的经验来看，这只是某些人一厢情愿的想法罢了。

我是 [Quicer（NodeJS QUIC 的 TypeScript 实现）](https://bbs.kanxue.com/elink@f4aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6J5L8h3q4J5P5q4)9J5c8Y4q4#2K9h3y4C8k6i4t1`.)的合著者。对于那些用 C/C++，Rust 或 Go 实现的协议栈来说，这显得相当离奇。我们选择用 TypeScript 主要是为了评估 QUIC 脚本语言实现的开销和可行性。由于我们的进行试验的时间处于 QUIC 的早期，所以测试结果并不是太好，见图 4。

![图片描述](https://bbs.kanxue.com/upload/attach/202001/93_VBMA9VXQCMWPMU8.png)

图 4：CPU、内存使用率 *Quicker (TypeScript)* vs *ngtcp2 (C/C++)*

### 反对观点

- QUIC 将在内核/硬件层实现。
- 相较于其他开销，TCP+TLS 连接的开销低得不值一提；即使 QUIC 连接的开销是 TCP+TLS 的 2 倍。
- 现在的测试数据是 Google 版的 QUIC,IETF 版的 QUIC 将与此不同。
- [（客户端）硬件将会越来越快](https://bbs.kanxue.com/elink@d66K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1N6$3W2@1N6r3g2J5i4K6u0W2j5$3!0E0i4K6u0r3M7$3I4A6k6$3S2@1L8s2W2D9j5i4c8W2i4K6u0r3M7%4c8S2N6s2g2K6i4K6u0r3z5e0j5K6y4o6j5H3x3o6R3^5x3e0R3#2z5o6M7J5x3K6R3#2)。
- QUIC 的开销并没有高到不可控制。
