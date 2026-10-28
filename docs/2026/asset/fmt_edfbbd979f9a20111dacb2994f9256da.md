
![](https://bbs.kanxue.com/upload/attach/201703/706972_3g0jxjt96r16x4y.png)

图12：托管在IP 217.23.13.111 (译者注：应该是110)上带有重定向到Sundown EK的iframe的域名

## 利用popcash[.]net

一组重定向域名利用了302/303重定向，从广告商`popcash[.]net` (Alexa #165)重定向到Sundown EK的登陆页。在这些例子中，广告商并不直接链接到Sundown EK的域名，而是通过两个域进行中转。

表12列出了`popcash[.]net`能过303重定向到的域名，再通过这些域名链接到第二层域名(通常通过iframe或是另一个303重定向)并最终将用户重定向到Sundown EK的域名。

![](https://bbs.kanxue.com/upload/attach/201703/706972_cl0g96k4ctegnva.png)

表12：shadow服务器域名列表

这些域名使用了两个IP，23.238.19.56 或 173.208.245.114。

这样的重定向的一个典型的例子如图13和图14所示。

![](https://bbs.kanxue.com/upload/attach/201703/706972_6sw5kmq4f3c1tma.png)

图13：从`popcash[.]net`进行重定向链上的两个域名

![](https://bbs.kanxue.com/upload/attach/201703/706972_9kflgfx10lcsvuk.png)

图14：重定向到Sundown EK登陆页上的第二层Shadow服务器的域名

`popcash[.]net`在我们通知之后清除了这些恶意广告。

## 通过Propeller广告平台

这次攻击活动与带有下列whois信息的一组域名有关：

Registrant Name: elise wickson

Registrant [.]organization: None

Registrant Street: 4-4025 Sladeview Crescent

Registrant City: mississauga

Registrant State/Province: QC

Registrant Postal Code: L6L 5Y1

Registrant Country: CA

Registrant Phone: +1.5148852225

Registrant Name: bruno calisto

Registrant [.]organization: None

Registrant Street: 8807 PIERRE-BOUCHER

Registrant City: laval

Registrant State/Province: QC

Registrant Postal Code: H7A3R2

Registrant Country: CA

Registrant Phone: +1.5148859965

这些域名通过载入托管在Webzilla B.V托管服务上的合法广告网站作为重定向到Sundown EK域名的shadow服务器。表13列出了这些域名的完整列表。

![](https://bbs.kanxue.com/upload/attach/201703/706972_keu61zaarnh5d2q.png)

表13：这次攻击活动中的相关域名

![](https://bbs.kanxue.com/upload/attach/201703/706972_1ao6mi5xx757l40.png)

表14：这次攻击活动中的重定向广告

其他Sundown EK的恶意广告的例子如表15所示。

![](https://bbs.kanxue.com/upload/attach/201703/706972_sp4dhwhxs4kv78e.png)

表15：重定向到Sundown EK 的其他域名和广告服务

## Terror EK

与Sundown EK相似，Terror EK也利用了广告商`serve.popads[.]net`，把流量重定向到它所控制的域名。广告商被用于把流量重定向到托管在IP 144.217.84.234 上的域名，然后进一步重定向到托管在144.217.84.235 / 94.74.81.91 / 94.74.81.8上的域名。

托管在149.202.164.86上的域名的早期例子见于去年12月我们的同事发表在Trustwave 和 Malwarebytes上的文章。

在2017年1月，这次攻击活动的新域名出现在了一个不同位置的IP 上。然而，正如我们在前面所观测到的那样，Terror EK仍在使用它下载`ccminer`。

图15和图16展示了重定向到域名`onlinesalespromarketing[.]com`(托管在144.217.84.234下) 并进一步重定向到登陆页域名`onlinesalesproaffiliate4[.]us`的广告服务。

![](https://bbs.kanxue.com/upload/attach/201703/706972_80njbrds754r84q.png)

图15：`serve.popads[.]net`重定向到shadow服务器

![](https://bbs.kanxue.com/upload/attach/201703/706972_k0arm8hhpzsf6zs.png)

图16：重定向到Terror EK登陆页的shadow服务器

表16列出了使用上面提及的IP重定向到登陆页的新域名：

![](https://bbs.kanxue.com/upload/attach/201703/706972_4hsuvmgbfw2e5s6.png)

表16：在第一次攻击活动之后Terror EK使用的新域名

## 总结

对于普通用户来说，恶意广告和exploit kits仍然是一个重大的威胁。尽管我们知道这不太可能，但我们仍然强烈推荐在所有web浏览器中使用ad blockers。因此，最好的方法就是让你的浏览器和应用程序永远保持在最新版。此外，定时检查你的浏览器，看看有哪些插件正在使用，如果这些插件不是必要的话，就禁用它们。

在我们所有讨论的例子中，通过我们的multi-flow 和multi-vector检测引擎，FireEye的顾客都能够避免被感染。

更新(2017.3.17)：我们在此感谢PopCash, Adcash, Propeller Ads, AlpNames Limited and Cloudflare关闭了那些我们这个博客中讨论到的，链接到shadow服务器的欺诈账号。

这篇博客发表于Wed Mar 15 08:48:00 EDT 2017，并归类在Advanced Malware, Blog, Exploit Kits, Latest Blog Posts, Malvertising, Threat Research 和Zain Gardezi下。

原文链接：

7dfK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2X3K9i4u0W2k6i4W2W2i4K6u0W2j5$3!0E0i4K6u0r3j5X3I4G2k6#2)9J5c8Y4c8Z5M7X3g2S2N6q4)9J5k6s2u0W2M7$3g2S2M7X3y4Z5i4K6u0r3x3U0l9I4y4#2)9J5c8U0l9K6i4K6u0r3M7%4c8A6L8r3I4Q4y4h3k6Y4k6i4c8@1K9h3&6Y4i4K6g2X3M7$3g2J5N6X3g2V1i4K6u0W2K9s2c8E0L8l9`.

本文由 看雪翻译小组 梦野间 翻译

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
