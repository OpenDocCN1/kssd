
Trickbot经HTTP GET请求发送了很多以`.png`结尾的Windows可执行文件。当受感染的Windows主机是Active Directory环境中的客户端时，这些Trickbot可执行文件可用于感染易受攻击的域控制器（DC）。

在Wireshark中用如下过滤器可以找到这些URL。

`http.request and ip contains .png`

![img](https://bbs.kanxue.com/upload/attach/202001/743775_TXJ2NYMHHQRESVD.jpg)

*图 21:过滤查找用.png结尾的URL发送的Trickbot可执行文件*

追踪图21中三条TCP或HTTP流量的任一条，就可以看到类似之前图9所示的表明是Windows可执行文件的迹象。在本例中，HTTP respond header表明返回的是一张图片，即使它很明显是一个Windows可执行文件或DLL文件。

![img](https://bbs.kanxue.com/upload/attach/202001/743775_QUT9F2C92DHEQCM.jpg)

*图 22: 通过URL发送的以.png结尾的Windows可执行文件*

根据本教程前部分所述，可以从Wireshark中提取出这些文件，并确定他们是Windows可执行文件，及获取他们的SHA256哈希值。

## 通过其他恶意软件分发Trickbot

Trickbot经常由其他恶意软件分发。Trickbot通常伴随着Emotet出现，但是也会在IceID和Ursnif感染后出现。

因为Emotet经常分发Trickbot，我们现在来看一个发生在2019年9月，由Emotet感染Trickbot的例子9J5k6r3q4F1j5h3I4&6M7$3W2K6i4K6u0W2L8X3g2@1i4K6u0r3x3U0l9I4z5g2)9J5c8U0l9&6i4K6u0r3x3U0g2Q4x3V1k6A6L8X3c8W2P5o6u0Q4x3X3g2Z5N6r3#2D9)。去年我们已经在Palo Alto Networks中揭露过Emotet感染Trickbot，所以在本文中我们更关注Trickbot的行为。

![img](https://bbs.kanxue.com/upload/attach/202001/743775_2CZRXR6GHPW7ZDV.jpg)

*图 23: 具有Trickbot活动的Emotet简化流程图*.

从这个链接下载pcap。文件名为`2019-09-25-Emotet-infection-with-Trickbot-in-AD-environment.pcap.zip`，解压密码为`infected`。解压后在Wireshark中打开。用`basic`过滤器查看如图24所示的基于web的感染流量。

![img](https://bbs.kanxue.com/upload/attach/202001/743775_B7PAFPTFBSZBUVE.jpg)

*图24:过滤 Emotet+Trickbot 感染的web流量.*

经验丰富的分析师通常可以识别Emotet生成的流量和Trickbot生成的流量。感染Emotet后会产生从服务器发回加密数据的HTTP流量。和感染Trickbot后产生的HTTPS/SSL/TLS有很大不同。图25展示了Emotet和Trickbot感染流量的不同。

![img](https://bbs.kanxue.com/upload/attach/202001/743775_3F5K4665UEMJ8Z3.jpg)

*图 25: Emotet和 Trickbot 流量的差异.*

本例感染发生在Active Directory环境中，10.9.25.102是受感染Windows 客户端，10.9.25.9是DC。在后续的流量中，可以看到DC表现出了受Trickbot感染的迹象，如图26所示。

![img](https://bbs.kanxue.com/upload/attach/202001/743775_YDQFGMSA8U86FBZ.jpg)

*图26: 在 DC上的Trickbot行为.*

感染是如何从客户端传到DC的呢？ Trickbot利用某一版本的EternalBlue exploit用微软的SMP协议传输。在本例中，受感染的Windows客户端经TCP 445端口发送了几次信息给在10.9.25.9的DC，从这些流量中可以取出Trickbot可执行文件（从`185.98.87[.]185/wredneg2.png`）。在DC向`185.98.87[.]185`发送数据之前用`basic+`过滤器查看10.9.25.102到10.9.25.9流量中的SYN段，如图27所示。

![img](https://bbs.kanxue.com/upload/attach/202001/743775_J3YH75EFWQQWUF5.jpg)

*图27: 在DC从196.98.87 [。] 185 / wredneg2.png检索到Trickbot EXE之前，找到从10.9.25.102处的客户端到10.9.25.9处的DC（以灰色显示）的流量。*

追踪其中一个TCP流，例如经TCP 49321端口从10.9.25.102到10.9.25.9 445TCP端口这一条。这是一条很不常见的从客户端发往DC的流量，所以它很可能包含 EternalBlue exploit 。图28就是这一条流量。

![img](https://bbs.kanxue.com/upload/attach/202001/743775_YF6AA6Z3HB46374.jpg)

*图28:不常见的从客户端经TCP 445端口发文DC的流量，很可能是EternalBlue*

*类exploit*

## 结论

本教程通过分析2019年9月产生的两个 pcap展示了如何检查是否感染了Trickbot恶意软件。更多近期产生的包含Trickbot行为的pcap可以移步 [malware-traffic-analysis.net](https://bbs.kanxue.com/elink@16aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2E0j5h3I4%4j5i4u0W2i4K6u0V1N6s2u0S2k6X3k6A6j5#2)9J5k6r3q4F1j5h3I4&6M7$3W2K6i4K6u0W2L8X3g2@1i4K6u0r3K9h3&6V1k6i4S2Q4x3X3g2Z5N6r3#2D9).

更多关于Wireshark教程，可以看我们之前的教程：

- [Customizing Wireshark – Changing Your Column Display](https://bbs.kanxue.com/elink@347K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6#2L8X3W2@1y4o6u0Q4x3X3g2H3j5h3I4G2j5h3I4@1L8$3&6W2N6s2N6G2M7X3E0K6i4K6u0W2j5$3!0E0i4K6u0r3N6h3&6A6N6o6b7J5i4K6u0V1j5%4g2K6N6r3!0E0K9i4A6A6L8X3N6Q4x3X3c8%4K9i4u0W2M7$3S2S2M7X3E0Q4x3X3c8U0K9r3q4F1k6$3W2F1k6#2)9J5k6r3y4G2L8s2g2E0L8W2)9J5k6r3c8A6M7%4m8D9j5i4W2Q4x3V1j5`.)

- [Using Wireshark – Display Filter Expressions](https://bbs.kanxue.com/elink@aaeK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6#2L8X3W2@1y4o6u0Q4x3X3g2H3j5h3I4G2j5h3I4@1L8$3&6W2N6s2N6G2M7X3E0K6i4K6u0W2j5$3!0E0i4K6u0r3N6i4y4A6L8X3N6Q4x3X3c8%4K9i4u0W2M7$3S2S2M7X3E0Q4x3X3c8V1K9i4y4H3L8r3q4&6i4K6u0V1k6X3W2D9N6r3g2J5i4K6u0V1k6i4S2H3M7X3g2K6M7$3W2G2L8Y4y4Q4x3V1j5`.)

- [Using Wireshark: Identifying Hosts and Users](https://bbs.kanxue.com/elink@c55K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6#2L8X3W2@1y4o6u0Q4x3X3g2H3j5h3I4G2j5h3I4@1L8$3&6W2N6s2N6G2M7X3E0K6i4K6u0W2j5$3!0E0i4K6u0r3N6i4y4A6L8X3N6Q4x3X3c8%4K9i4u0W2M7$3S2S2M7X3E0Q4x3X3c8A6k6r3g2F1N6r3W2X3P5h3W2F1k6#2)9J5k6r3S2G2M7%4c8K6i4K6u0V1j5h3&6V1i4K6u0V1N6i4y4W2M7Y4y4Q4x3V1j5`.)

- [Using Wireshark: Exporting Objects from a Pcap](https://bbs.kanxue.com/elink@1d6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6#2L8X3W2@1y4o6u0Q4x3X3g2H3j5h3I4G2j5h3I4@1L8$3&6W2N6s2N6G2M7X3E0K6i4K6u0W2j5$3!0E0i4K6u0r3N6i4y4A6L8X3N6Q4x3X3c8%4K9i4u0W2M7$3S2S2M7X3E0Q4x3X3c8W2P5s2m8G2M7Y4c8A6L8X3N6Q4x3X3c8G2j5X3A6W2j5%4c8K6i4K6u0V1k6Y4u0G2L8g2)9J5k6r3q4Q4x3X3c8H3j5$3q4H3i4K6u0r3)

原文地址：原文地址： 465K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6#2L8X3W2@1y4o6u0Q4x3X3g2H3j5h3I4G2j5h3I4@1L8$3&6W2N6s2N6G2M7X3E0K6i4K6u0W2j5$3!0E0i4K6u0r3N6$3W2J5k6i4y4Z5j5i4u0C8i4K6u0V1N6s2g2@1L8%4u0A6j5h3I4Q4x3X3c8W2P5r3q4E0K9h3&6A6L8X3N6Q4x3X3c8@1M7X3W2U0K9$3u0G2N6q4)9J5k6r3W2F1k6X3g2U0N6r3W2G2L8Y4y4Q4x3V1j5`.

编译：看雪翻译小组 lumou

校对：看雪翻译小组 梦野间

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
