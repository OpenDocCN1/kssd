
为了帮助传播 DNS 重绑定攻击的意识，我构建了其他人可以用来创建自己的 DNS 重绑定攻击的工具和库。除了针对特定设备或服务编写的 JavaScript 有效负载之外，交付该有效负载、与恶意 DNS 服务器交互以及枚举受害者专用网络上的主机的过程，在不同的攻击目标之间几乎相同。为了降低了解和执行 DNS 重绑定攻击的门槛，我已经开源了：

- 一个名为 [whonow](https://bbs.kanxue.com/elink@bc8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6T1M7X3q4F1L8X3!0F1k6r3!0J5M7$3g2&6i4K6u0r3N6$3S2G2L8X3!0%4) 的“恶意”DNS 服务器。
- 一个名为 [DNS Rebind Toolkit](https://bbs.kanxue.com/elink@6deK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6T1M7X3q4F1L8X3!0F1k6r3!0J5M7$3g2&6i4K6u0r3k6r3&6K6i4K6u0V1M7X3g2T1K9h3&6V1i4K6u0V1N6r3!0G2L8r3E0A6N6l9`.`.) 的前端 JavaScript 库。

## Whonow DNS 服务器

[Whonow](https://bbs.kanxue.com/elink@780K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6T1M7X3q4F1L8X3!0F1k6r3!0J5M7$3g2&6i4K6u0r3N6$3S2G2L8X3!0%4) 是一个自定义 DNS 服务器，允许你使用域请求本身动态指定 DNS 响应和重新绑定规则。这是一个工作示例：

```text
# respond to DNS queries for this domain with 34.192.228.43 the first

# time it is requested and then 192.168.1.1 every time after that.

A.34.192.228.43.1time.192.168.1.1.forever.rebind.network



# respond first with 34.192.228.43, then 192.168.1.1 the next five

# times, and then start all over again (1, then 5, forever…)

A.34.192.228.43.1time.192.168.1.1.5times.repeat.rebind.network
```

动态 DNS 重新绑定规则的优点在于，你无需启动自己的恶意 DNS 服务器即可开始利用浏览器的同源策略。相反，每个人都可以共享在 `rebind.network` 的 53 端口上运行的同一公共 Whonow 服务器。

如果你愿意，可以立即尝试，只需使用 `dig` 查询公共 Whonow 实例。它响应对 `*.rebind.network` 域名的请求。

```bash
# dig is a unix command for making DNS requests

dig A.10.10.10.50.forever.rebind.network

;; QUESTION SECTION:

;10.10.10.50.forever.rebind.network. IN A

;; ANSWER SECTION:

10.10.10.50.forever.rebind.network. 1 IN A 10.10.10.50
```

Whonow 总是将 TTL 值设置为 1 秒，因此你可以预期它的响应将不会长时间停留在 DNS 解析器的缓存中。

![](https://bbs.kanxue.com/upload/attach/201807/665727_HE8MW6J6KBFKJF5.jpg)

## DNS Rebind Toolkit

[DNS Rebind Toolkit](https://bbs.kanxue.com/elink@10cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6T1M7X3q4F1L8X3!0F1k6r3!0J5M7$3g2&6i4K6u0r3k6r3&6K6i4K6u0V1M7X3g2T1K9h3&6V1i4K6u0V1N6r3!0G2L8r3E0A6N6l9`.`.) 是一个实用程序库，可自动执行 DNS 重绑定攻击的过程。一旦你编写了一个 JavaScript 有效载荷来定位特定服务，DNS Rebind Toolkit 就可以帮助你在受害者的网络上部署该攻击。

- 使用 [WebRTC IP 地址泄漏](https://bbs.kanxue.com/elink@af7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6V1K9h3q4X3P5h3N6A6i4K6u0r3N6$3g2T1M7Y4c8U0i4K6u0V1K9i4m8K6) 来发现受害者的本地 IP 地址。
- 可以自动执行网络子网枚举，这样即使你不知道专用网络上的 IP 地址，也可以定位设备。
- 自动化攻击的 DNS 重新绑定部分。你得到一个 `Promise`，一旦重新绑定完成就会解决。
- 在幕后使用 Whonow，因此你不必担心 DNS 重绑定攻击的 DNS 服务器组件。
- 带有几个用于攻击常见物联网设备的有效负载和记录良好的示例，以便你学会编写自己的漏洞。

在 `192.168.1.1/24` 子网上启动对 Google Home 设备的攻击，就像在 `index.html` 页面中嵌入代码段一样简单。

![](https://bbs.kanxue.com/upload/attach/201807/665727_RT8XY8MWGJCYGGF.png)

`index.html` 文件将启动攻击，为传递给 `rebind.attack(...)` 的数组中的每个 IP 地址生成一个 `iframe`。每个 `iframe` 都使用以下 JavaScript 代码段嵌入有效内容 `/google-home.html`：

![](https://bbs.kanxue.com/upload/attach/201807/665727_4P9R8UZVDF6FTWW.png)

# 链接和感谢

感谢 Lily Hay Newman 与我进行了三个月的对话，以及她在 WIRED 中对这项研究进行的[报道](https://bbs.kanxue.com/elink@782K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2%4K9i4u0W2k6q4)9J5k6h3y4G2L8g2)9J5c8Y4y4@1L8%4u0&6i4K6u0r3j5$3S2J5L8$3#2W2j5$3q4K6N6q4)9J5k6s2u0G2K9%4g2Q4x3X3c8K6L8$3&6G2M7#2)9J5k6r3c8F1M7#2)9J5k6s2u0W2j5X3W2F1k6r3W2F1k6#2)9J5k6s2k6#2L8r3&6W2M7X3q4T1K9h3I4A6N6s2W2Q4x3V1j5`.)。

非常感谢 William Robertson 对这项工作的帮助，特别是让我攻击他的家庭网络和 Sonos 设备。我还要感谢 Roku 和 Sonos 的安全团队在开发和部署固件更新方面的迅速反应，以保护他们的用户免受 DNS 重绑定攻击。

以下是我在研究中发现的有用 DNS 重绑定资源集合。你可能也会从中得到帮助！

- [斯坦福大学的原始 DNS 重绑定研究](https://bbs.kanxue.com/elink@9cbK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0M7Y4W2H3N6r3!0Q4x3X3g2K6N6r3q4F1k6X3!0J5k6q4)9J5k6h3g2V1N6g2)9J5c8X3c8F1M7#2)9J5c8R3`.`.)（2007 年）
- [与 Robert RSnake Hansen 进行 DNS 重绑定](https://bbs.kanxue.com/elink@c44K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2&6L8%4g2@1N6h3u0W2i4K6u0W2j5$3!0E0i4K6u0r3N6$3q4@1j5$3S2Q4x3@1k6$3i4K6y4p5c8V1W2c8c8@1E0u0c8e0y4r3N6U0l9`.)（精彩视频）
- [CORS, the Frontendian](https://bbs.kanxue.com/elink@4b9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6X3M7X3!0F1N6r3g2F1k6r3W2S2L8W2)9J5k6h3y4G2i4K6u0r3j5$3!0J5M7H3`.`.)
- [Luke Young 的 DEFCON 25“实现可靠的 DNS 重新绑定”讲座](https://bbs.kanxue.com/elink@087K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2&6L8%4g2@1N6h3u0W2i4K6u0W2j5$3!0E0i4K6u0r3N6$3q4@1j5$3S2Q4x3@1k6$3i4K6y4p5f1e0m8v1c8#2)9#2k6X3g2w2e0r3y4%4M7H3`.`.)
- [Ricky Lawshae 的 DEFCON 19 SOAP 和 UPnP 讲座](https://bbs.kanxue.com/elink@7dfK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2&6L8%4g2@1N6h3u0W2i4K6u0W2j5$3!0E0i4K6u0r3N6$3q4@1j5$3S2Q4x3@1k6$3i4K6y4p5k6q4b7H3j5U0g2p5g2%4p5&6j5U0R3`.)
- [UPnP Hacks 网站](https://bbs.kanxue.com/elink@37eK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4g2H3L8Y4m8Q4x3X3c8Z5j5h3y4C8M7#2)9J5k6h3!0J5k6#2)9J5c8Y4g2H3L8Y4m8Q4x3X3g2Z5N6r3#2D9)
- [亲爱的开发人员，谨防 DNS 重绑定](https://bbs.kanxue.com/elink@516K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2@1N6$3W2K6N6r3I4G2j5$3E0Q4x3X3g2U0L8$3#2Q4x3V1j5J5x3o6p5^5i4K6u0r3x3o6u0Q4x3V1j5J5z5q4)9J5c8X3c8W2j5i4u0Q4x3X3c8V1k6i4k6W2L8r3!0H3k6i4u0K6i4K6u0V1j5X3g2%4j5i4u0W2i4K6u0V1k6r3&6K6i4K6u0V1M7X3g2T1K9h3&6V1K9h3&6Y4i4K6u0r3)
- [DNS 重绑定的实际攻击](https://bbs.kanxue.com/elink@631K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2@1M7X3W2H3N6$3W2J5k6g2)9J5k6h3y4G2L8g2)9J5c8Y4y4@1j5i4c8W2i4K6u0V1L8$3k6Q4x3X3c8K6k6h3y4#2M7X3W2@1P5g2)9J5c8Y4k6W2M7Y4c8Q4x3V1k6H3M7X3q4U0N6r3W2U0j5h3I4Q4x3X3c8S2N6s2c8S2j5$3E0K6i4K6u0V1k6r3&6K6i4K6u0V1M7X3g2T1K9h3&6V1K9h3&6Y4i4K6u0r3)
- [由 Tavis Ormandy 撰写的原始暴雪游戏 DNS 重绑定错误报告](https://bbs.kanxue.com/elink@587K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1N6h3N6K6i4K6u0W2j5$3S2J5L8$3#2A6N6h3#2Q4x3X3g2G2M7X3N6Q4x3V1k6H3i4K6u0r3M7s2u0G2K9X3g2U0N6q4)9J5k6s2A6W2M7X3!0Q4x3V1k6A6M7%4y4#2k6i4y4Q4x3V1k6V1k6i4c8S2K9h3I4Q4x3@1k6A6k6q4)9K6c8o6p5@1y4K6q4Q4x3U0k6S2L8i4m8Q4x3@1u0V1k6i4y4U0i4K6y4p5x3R3`.`.)

## 原文链接

`f9fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0k6h3c8A6N6h3#2Q4x3X3g2U0L8$3#2Q4x3V1k6Q4y4o6m8T1M7X3q4F1L8X3!0F1k6r3!0J5M7$3g2&6i4K6u0r3j5i4c8@1j5h3y4C8K9h3&6Y4i4K6u0V1M7s2u0A6N6X3q4@1k6g2)9J5k6r3&6W2N6s2N6G2M7X3E0K6i4K6u0V1k6Y4u0G2L8g2)9J5k6s2c8Z5k6g2)9J5k6r3W2F1N6r3g2J5L8X3g2@1i4K6u0V1N6$3W2@1K9q4)9J5k6r3c8F1M7#2)9J5k6s2u0W2j5X3W2F1k6r3W2F1k6#2)9J5k6r3g2S2y4K6l9&6z5r3p5J5k6o6x3J5y4b7`.`

编译：看雪翻译小组 一壶葱茜

校对：看雪翻译小组 SpearMint

[[招生]科锐逆向工程师培训（2026 年 11 月 13 日实地，远程教学同时开班，第 57 期）](https://bbs.kanxue.com/thread-51839.htm)

最后于 2018-7-24 12:44

被一壶葱茜编辑，原因：校对
