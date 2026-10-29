
即使要花费巨大的投入，Google 仍决定大规模部署 `QUIC`。这表明其带来收益大于成本，或许更佳的 web 性能将为其营收带来大幅提升。

[TCP 在物联网领域也占有一席之地](https://bbs.kanxue.com/elink@b5bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2J5k6i4y4W2j5i4u0U0K9r3N6S2N6r3g2Q4x3X3g2F1k6i4c8Q4x3V1k6H3N6h3u0D9K9h3y4S2N6r3W2G2L8W2)9J5c8U0x3J5x3o6t1&6y4U0b7K6y4W2)9#2k6W2c8o6f1q4)9#2k6X3W2F1i4K6g2X3N6r3S2W2i4K6g2X3d9h3&6@1k6i4u0F1k6i4c8Q4y4h3k6G2k6W2)9#2k6W2c8Z5K9h3&6Y4M7#2)9#2k6X3k6J5L8$3#2Q4y4h3k6G2M7%4c8J5j5h3y4A6M7$3#2Q4y4h3k6@1L8#2)9#2k6Y4m8J5L8$3#2A6L8X3g2F1j5$3f1`.)。

“我看过你的 `TypeScript` 代码，简直就是一团糟。合格的开发者能让 `QUIC` 运行地更快”

## 3. 0-RTT 的实际应用

`QUIC` 的另一主要卖点（实际上是源于 `TLS 1.3`）是 0-RTT 连接配置：你初始（HTTP）请求可以绑定到第一个握手包上，在第一个回应包就可以得到请求的数据，简直不能再快了！

然而，这有个前提条件：服务器必须是之前通过正常的 1-RTT 连接过的。二次连接的 0-RTT 数据是用首次连接得到的“预共享密钥(pre-shared secret)”加密的。服务器也需要知道该密钥，所以 0-RTT 只能连接同一台服务器，而不是同一个服务器集群。这意味着，负载均衡器要足够智能地将请求路由到正确的服务器。在最初的 `QUIC` 开发中，[Google 测试的正确率是 87%（桌面端）- 67%（移动端）](https://bbs.kanxue.com/elink@5caK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8q4)9J5k6h3q4U0L8g2)9J5k6h3!0J5k6#2)9J5c8X3y4A6N6r3q4@1K9h3!0F1i4K6u0W2j5$3k6E0i4K6y4r3K9h3c8Q4x3@1b7K6x3o6V1^5z5o6b7J5)。这一结果相当令人印象深刻，因为他们要求用户保持他们的 IP 地址不变。

此外，还有其他的缺陷：0-RTT 数据包容易受到“[重放攻击](https://bbs.kanxue.com/elink@b75K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2M7s2u0A6L8Y4c8Q4x3X3g2A6j5h3y4J5i4K6u0W2L8%4u0Y4i4K6u0r3x3U0l9I4y4#2)9J5c8U0l9^5x3W2)9J5k6i4m8V1k6R3`.`.)”，攻击者重复多次发送最初的数据包。由于完整性保护，数据包的内容不会被更改，但是依据数据包内请求的内容不同，[重复多次的请求可能会导致一些不希望的行为](https://bbs.kanxue.com/elink@6e3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1L8$3!0D9M7#2)9J5k6h3W2W2N6r3k6Q4x3X3g2G2M7X3N6Q4x3V1k6A6k6q4)9J5c8X3c8J5j5h3k6@1i4K6u0V1K9h3g2@1k6W2)9J5k6s2q4#2K9h3y4Q4x3X3c8S2M7s2m8D9K9h3y4S2j5X3W2D9K9i4c8&6i4K6u0V1x3o6y4Q4x3X3g2Z5N6r3#2D9i4K6t1K6K9r3g2J5k6g2)9J5k6s2c8Z5k6i4u0W2i4K6u0V1j5X3g2Q4x3X3c8V1M7X3q4Y4L8$3&6K6)（e.g., `POSTbank.com?addToAccount=1000`）。因此，[只有叫做“幂等”的数据可以通过 0-RTT 发送](https://bbs.kanxue.com/elink@d41K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5$3I4G2N6h3c8X3L8r3q4J5k6g2)9J5k6h3y4G2L8g2)9J5c8X3W2F1N6s2u0G2k6s2g2U0K9h3&6Y4i4K6u0V1x3q4)9J5k6s2u0@1N6q4)9J5c8R3`.`.)。根据应用场景，这将会严重限制 0-RTT 发挥作用（例如，IoT 传感器使用 0-RTT POST 数据，这可不是个好主意）。

最后还存在的问题是 IP 地址欺骗和 [UDP 放大攻击](https://bbs.kanxue.com/elink@1e6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2#2M7#2)9J5k6r3y4W2M7Y4c8Q4x3X3g2Y4L8%4k6Q4x3V1k6F1j5$3q4K6i4K6u0r3j5h3I4W2M7Y4c8K6i4K6u0r3g2p5p5I4y4q4)9J5k6o6l9I4y4@1p5`.)。在该场景下，攻击者伪装成受害者的 IP `a.b.c.d` 然后发送一个 UDP 包给服务器。如果服务器响应返回一个相当大的 UDP 包给 `a.b.c.d`，这样，攻击者就以一个相当小的带宽产生了大流量的攻击，见 图 5。为了防止该类攻击，`QUIC` 加入了[两条缓解措施](https://bbs.kanxue.com/elink@322K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1L8$3!0D9M7#2)9J5k6h3W2W2N6r3k6Q4x3X3g2G2M7X3N6Q4x3V1k6Z5N6r3#2D9i4K6u0r3k6s2u0S2k6Y4c8Q4x3X3c8A6k6i4c8X3i4K6u0V1M7i4g2A6j5#2)9J5k6s2c8J5j5h3&6K6M7r3!0J5N6q4)9J5k6o6p5$3i4K6t1K6M7$3g2U0N6r3W2G2L8W2)9J5k6o6S2Q4x3X3f1I4)：客户端的第一个数据包必须至少为 1200 字节（实践中最大为 1460）；同时，在未收到客户端响应时，服务器响应包不得超过 3 倍的请求包大小。所以只有 3600 - 4380 字节，包括 `TLS` 握手和 `QUIC` 开销，只有少量空间留给 `HTTP` 响应（如果有）。你会发送HTML `<head>`吗？header， push 一些东西？这有关系吗？这个确切的问题是我期待深入研究的事情之一。

![图片描述](https://bbs.kanxue.com/upload/attach/202001/93_M87RRTBF25FA28W.png)

图 5：UDP 放大攻击

对 `QUIC` 最后致命一击的是 `TCP+TLS 1.3` 的“Fast Open”选项（不过也存在上述的缺陷）也可以使用 0-RTT 功能。所以 0-RTT 这个功能并不足以让人采用 `QUIC`。

### 反对观点

- [HTTP 服务器应该能免疫重放攻击了](https://bbs.kanxue.com/elink@c30K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5$3I4G2N6h3c8X3L8r3q4J5k6g2)9J5k6h3y4G2L8g2)9J5c8X3W2F1N6s2u0G2k6s2g2U0K9h3&6Y4i4K6u0V1x3q4)9J5k6s2u0@1N6q4)9J5c8R3`.`.)。

- 阻止重放攻击很容易，只需添加一些内容如时间戳或序列号，因为攻击者无法修改数据包内容。

- 1-RTT 仍比 `TCP+TLS 1.2` 的 3-4 RTT 好不少。

- 服务端为了性能考虑可以忽略 RFC 中规定的 3600 字节的限制。

- [有证据显示更大的初始拥塞窗口（initial congestion window）并未给 TCP 的 HTTP/2 整体连接带来大提升](https://bbs.kanxue.com/elink@ea5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2M7Y4S2A6N6W2)9J5k6h3!0J5k6#2)9J5c8X3q4T1M7#2)9J5c8U0p5%4x3o6N6Q4x3X3f1H3y4e0R3K6y4R3`.`.)，所以 `QUIC` 终端限制或许不会对 `HTTP/3` 实际应用带来影响

- [TCP 的“Fast Open”尚不可用，Mozilla 甚至不打算在 Firefox 支持它](https://bbs.kanxue.com/elink@4ffK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6J5L8h3q4J5P5q4)9J5c8Y4q4#2K9h3y4Q4y4h3k6@1L8$3!0T1K9h3N6@1L8$3k6S2K9h3I4Q4x3V1k6A6M7%4y4#2k6i4y4Q4x3V1j5I4i4K6t1K6K9i4y4K6N6h3g2U0L8$3#2E0k6h3&6@1i4K6u0V1y4o6b7I4x3e0x3K6x3U0j5I4)。（反对观点：久而久之，对 TCP Fast Open 的支持也会增多）

## 4. QUIC v3.5.66.6.8.55-Facebook

与 `TCP` 相对的是，`QUIC` 集成有完整的[版本协商配置](https://bbs.kanxue.com/elink@5e2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1L8$3!0D9M7#2)9J5k6h3W2W2N6r3k6Q4x3X3g2G2M7X3N6Q4x3V1k6Z5N6r3#2D9i4K6u0r3k6s2u0S2k6Y4c8Q4x3X3c8A6k6i4c8X3i4K6u0V1M7i4g2A6j5#2)9J5k6s2c8J5j5h3&6K6M7r3!0J5N6q4)9J5k6o6p5$3i4K6t1K6M7$3g2U0N6r3W2G2L8W2)9J5k6o6j5`.)，主要是为了在不破坏现有部署的前提下进行版本演进。客户端使用其最支持的版本发送第一握手包。如果服务端不支持该版本，则返还一个协议版本协商包，列出服务端支持的版本。客户端选择其一版本，尝试重新连接。该过程是必要的，因为各版本的编码实现必然会有些差异的。

### 每一个 RTT 都是多余的

承上所述，每一次版本协商需要额外的 1 RTT。在有限数量的版本的情况下，这并不存在什么问题。只是，实际情况可能是每年有不止一个正式版本，而是一大批不同的版本。一种可能的情况是，甚至因为[某个单一的功能而发布一个新版本](https://bbs.kanxue.com/elink@69bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0j5h3W2D9j5i4u0U0K9r3W2$3k6g2)9J5k6h3W2W2N6r3k6Q4x3X3g2G2M7X3N6Q4x3V1k6S2M7X3y4Z5i4K6u0r3L8i4y4Y4i4K6u0r3M7i4g2A6j5#2)9J5c8X3y4&6k6K6y4C8d9$3N6v1M7$3k6%4j5X3N6b7N6@1b7I4y4h3N6r3P5e0m8D9P5r3y4D9k6H3`.`.)（比如前面提到的 轮转位（spin bit））。另外一种情况是，为了让用户体验不同的，**非标准化**的功能特性。这些都将导致“狂野西部（wild-wild-west）”，不同部分的人运行略微有点区别的 `QUIC`，最终使得版本协商（意味着 1 RTT 开销）的可能性增加。考虑地更全面深入一点，可能[一部分人](https://bbs.kanxue.com/elink@0b7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2&6L8%4g2@1N6h3u0W2i4K6u0W2j5$3!0E0i4K6u0r3N6$3q4@1j5$3S2Q4x3@1k6$3i4K6y4p5h3V1^5#2f1r3!0i4y4#2)9#2k6X3E0V1b7b7`.`.)觉得他们自己的 `QUIC` 版本更好而不愿意更新到新的标准版本.最终，导致了 drop-and-forget 的情况，例如，物联网应用场景里，软件更新频率极低并且严重滞后。

我们可以从[参数传输](https://bbs.kanxue.com/elink@7d1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1L8$3!0D9M7#2)9J5k6h3W2W2N6r3k6Q4x3X3g2G2M7X3N6Q4x3V1k6Z5N6r3#2D9i4K6u0r3k6s2u0S2k6Y4c8Q4x3X3c8A6k6i4c8X3i4K6u0V1M7i4g2A6j5#2)9J5k6s2c8J5j5h3&6K6M7r3!0J5N6q4)9J5k6o6p5$3i4K6t1K6M7$3g2U0N6r3W2G2L8W2)9J5k6o6N6Q4x3X3f1K6)的过程中找到部分的解决思路。这些参数被用来作为握手认证的一部分，同时用来启用/关闭某些功能。例如，`QUIC` 中有 1 个参数用于表示切换连接迁移支持。然而，目前尚不清楚 `QUIC` 实际实现是倾向于版本标记还是参数控制。

对于一个以 0-RTT 标榜的传输协议来说，竟然还需要为了 1 个 RTT 的版本协商而大费脑筋，实在是有点自相矛盾。

### 反对观点

- 浏览器只支持主流的版本，只要服务端也支持，那么就没什么问题。

- 运行他们自己版本的人需要确保客户端和服务端都支持其功能，或者是通过版本协商确定。

- 客户端将会把服务端支持的版本列表缓存下来用于接下来的连接。

- 用于支持独立功能特性的版本可以放在单独的数据库中。如果不支持某个版本，服务端将会智能的不进行版本协商；如果主线功能一致，那么服务端将安全地忽略其他不一致的功能。

- 即使没有版本协商，服务端也总是会将他们支持的版本列表完整的回传。这样，在二次连接时，客户端可以选择其最支持的版本来建立连接。

## 5. 拥塞控制的公平性

`QUIC` 的端到端加密，版本协商和用户空间实现，为用户提供了前所未见的灵活性。到目前为止，[拥塞控制算法](https://bbs.kanxue.com/elink@ef3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8W2)9J5k6i4N6A6K9$3W2H3k6h3c8A6j5g2)9J5k6h3!0J5k6#2)9J5c8Y4N6A6K9$3W2Q4x3V1k6f1b7#2m8Q4y4h3k6U0L8$3&6Y4k6i4y4@1K9h3!0F1i4K6g2X3j5$3!0F1N6s2u0G2L8l9`.`.)(congestion control algorithms (`CCAs`))都是在内核实现的。你可以选择[选择不同的算法](https://bbs.kanxue.com/elink@d01K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5$3I4G2N6h3c8X3L8r3q4J5k6g2)9J5k6h3y4G2L8g2)9J5c8X3S2@1N6s2m8Q4x3X3b7J5i4K6u0V1M7s2u0A6L8%4u0A6N6r3W2*7j5i4c8A6L8$3&6Q4x3X3c8%4K9i4c8Z5i4K6u0V1L8X3N6A6L8Y4S2Q4x3V1j5`.)，但是，该算法将应用到整个服务器。由此，大多数拥塞控制算法都是通用算法，因为它们需要处理的是各种接入链路。在 `QUIC` 中，你可以根据某个连接的具体情况选择合适的算法或者至少可以很容易地试用不同的（新的）算法。比如，我想用 [NetInfo API](https://bbs.kanxue.com/elink@dc0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2E0L8%4A6A6L8r3I4S2i4K6u0W2L8%4u0Y4i4K6u0r3k6h3&6Q4x3X3c8g2f1#2)9J5c8X3c8G2j5%4y4Q4x3V1k6i4k6h3u0Q4x3V1k6m8f1p5W2Q4x3V1k6z5k6i4c8%4L8%4u0C8i4K6g2X3d9h3&6X3L8%4u0E0j5i4c8A6L8$3&6Q4y4h3k6m8f1p5V1`.) 来确定接入链路的类型，并以此来调整拥塞控制算法的参数。

### Calimero

上一例子也明确地表明了潜在的危险：如果任何人都可以决定和修改拥塞控制算法（无需重新编译内核），这将会为滥用行为大开方便之门。毕竟，拥塞控制的很重要一部分是使每个连接得到或多或少尽量公平的带宽，这一原则称为“公平性”。如果一些 `QUIC` 服务器使用更极端的拥塞控制算法，使其连接获得超过公平原则下的带宽，这将会拉低其他非 `QUIC` 连接和使用不同拥塞控制算法的 `QUIC` 连接的带宽。
