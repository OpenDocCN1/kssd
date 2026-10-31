
# 自己整理出来的一套Go流量蜜罐劫持库

开发http/https流量蜜罐有一段时间了，"借鉴"了不少的项目，终于成了项目之集大成者O(∩\_∩)O。

项目使用Go语言开发，地址如下:

[2baK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6X3K9Y4q4A6M7$3u0S2i4K6u0r3e0i4W2t1L8$3&6W2P5g2m8J5L8%4S2&6](https://bbs.kanxue.com/elink@17bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6X3K9Y4q4A6M7$3u0S2i4K6u0r3e0i4W2t1L8$3&6W2P5g2m8J5L8%4S2&6)

## 项目介绍

> 写着玩玩，一款高性能http、socks代理蜜罐，用于捕捉恶意攻击流量。
>
> high performance，easy to use，honey proxy.

个人水平不咋地，也不知道有啥问题，希望大家多多反馈Bug。

## 参考项目

[elazarl/goproxy: An HTTP proxy library for Go (github.com)](https://bbs.kanxue.com/elink@815K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6W2L8r3q4*7j5i4u0D9i4K6u0r3k6$3!0H3M7X3!0^5P5b7`.`.)

[projectdiscovery/proxify: Swiss Army knife Proxy tool for HTTP/HTTPS traffic capture, manipulation, and replay on the go. (github.com)](https://bbs.kanxue.com/elink@056K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6H3M7X3!0B7k6h3y4@1k6r3W2K6j5$3!0$3k6i4u0&6i4K6u0r3M7s2u0G2P5r3W2X3P5b7`.`.)

[v2ray/v2ray-core: A platform for building proxies to bypass network restrictions. (github.com)](https://bbs.kanxue.com/elink@f01K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6$3x3Y4u0S2P5g2)9J5c8Y4j5J5M7X3q4&6i4K6u0V1j5$3!0J5k6b7`.`.)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2022-9-16 22:57

被fjqisba编辑，原因：

[#开源分享](https://bbs.kanxue.com/forum-41-1-134.htm)
