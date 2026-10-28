
注意：本教程中Wireshark的列显示需要通过[前一个教程](https://bbs.kanxue.com/elink@ccfK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6#2L8X3W2@1y4o6u0Q4x3X3g2H3j5h3I4G2j5h3I4@1L8$3&6W2N6s2N6G2M7X3E0K6i4K6u0W2j5$3!0E0i4K6u0r3N6h3&6A6N6o6b7J5i4K6u0V1j5%4g2K6N6r3!0E0K9i4A6A6L8X3N6Q4x3X3c8%4K9i4u0W2M7$3S2S2M7X3E0Q4x3X3c8U0K9r3q4F1k6$3W2F1k6#2)9J5k6r3y4G2L8s2g2E0L8W2)9J5k6r3c8A6M7%4m8D9j5i4W2Q4x3V1j5`.)设置。你还需根据[此文](https://bbs.kanxue.com/elink@defK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6#2L8X3W2@1y4o6u0Q4x3X3g2H3j5h3I4G2j5h3I4@1L8$3&6W2N6s2N6G2M7X3E0K6i4K6u0W2j5$3!0E0i4K6u0r3N6i4y4A6L8X3N6Q4x3X3c8%4K9i4u0W2M7$3S2S2M7X3E0Q4x3X3c8V1K9i4y4H3L8r3q4&6i4K6u0V1k6X3W2D9N6r3g2J5i4K6u0V1k6i4S2H3M7X3g2K6M7$3W2G2L8Y4y4Q4x3V1j5`.) 配置Wireshark显示过滤器。

## 通过恶意邮件感染Trickbot

Trickbot通常通过恶意邮件传播。这些邮件中包含下载伪装成音频或文档的恶意文件的链接。这些文件可能是Trickbot的Windows可执行文件，也可能是Trickbot可执行文件的某种下载器。很多时候，邮件中的链接是一个zip压缩包，压缩包里的文件就包含了Tribot可执行文件或下载器。

图1是2019年9月的一个案例。在这个案例中，邮件中包含里一个下载链接，下载的文件是zip压缩包。压缩包中包含一个Windows快捷方式文件，该文件下载了Trickbot可执行文件。[此处](https://bbs.kanxue.com/elink@bd7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2E0j5h3I4%4j5i4u0W2i4K6u0V1N6s2u0S2k6X3k6A6j5#2)9J5k6r3q4F1j5h3I4&6M7$3W2K6i4K6u0W2L8X3g2@1i4K6u0r3x3U0l9I4z5g2)9J5c8U0l9&6i4K6u0r3x3U0g2Q4x3V1k6A6L8X3c8W2P5q4)9J5k6h3S2@1L8h3H3`.) 是Trickbot感染的一个pcap。

![img](https://bbs.kanxue.com/upload/attach/202001/743775_56NT2DCRXPXA645.jpg)

*图1: 2019年9月通过邮件感染Trickbot的流程图.*

从[这个页面](https://bbs.kanxue.com/elink@865K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2E0j5h3I4%4j5i4u0W2i4K6u0V1N6s2u0S2k6X3k6A6j5#2)9J5k6r3q4F1j5h3I4&6M7$3W2K6i4K6u0W2L8X3g2@1i4K6u0r3x3U0l9I4z5g2)9J5c8U0l9&6i4K6u0r3x3U0g2Q4x3V1k6A6L8X3c8W2P5q4)9J5k6h3S2@1L8h3H3`.)下载该pcap。该pcap放在一个带密码的zip压缩包：**2019-09-25-Trickbot-gtag-ono19-infection-traffic.pcap.zip** 。用解压密码**infected**解压该文件并用Wireshark打开。在Wireshark中使用**basic**过滤器查看该流量，如图2所示。

![img](https://bbs.kanxue.com/upload/attach/202001/743775_5J4H2NNG9X7R537.jpg)

*图 2: 使用Wireshark查看Trickbot感染的pcap.*

通过查看这些流量，你可以看到以下这些在近期Trickbot感染中都出现的特征：

- 一个IP地址，经检查是受感染主机
- 通过TCP端口447和449的 HTTPS/SSL/TLS 流量
- 通过TCP端口8082的HTTP流量
- 以`.png` 为后缀的HTTP请求返回的是Windows可执行文件。

这个Trickbot感染的特殊之处是对 `94fK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3c8U0K9s2u0A6M7%4c8B7j5h3&6Q4y4f1u0Q4x3X3g2Q4y4f1c8U0L8$3@1`.`的HTTP请求返回的是一个zip文件，对 `144.91.69[.]195` 的HTTP请求返回的是Windows可执行文件。对目的地是`7f7K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3c8U0K9s2u0A6M7%4c8B7j5h3&6Q4y4f1u0Q4x3X3g2Q4y4f1c8U0L8$3@1`.` 的request进行追踪HTTP流，就可以得到图4，根据图4我们可以知道返回的是一个zip压缩包。

![img](https://bbs.kanxue.com/upload/attach/202001/743775_MRXMVGZARH8TDS4.jpg)

*图3: 追踪到 1adK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3c8U0K9s2u0A6M7%4c8B7j5h3&6Q4y4f1u0Q4x3X3g2Q4y4f1c8U0L8$3#2Q4c8e0N6Q4z5f1q4Q4z5o6c8t1g2q4c8b7i4@1f1$3i4@1t1#2i4K6R3I4*

![img](https://bbs.kanxue.com/upload/attach/202001/743775_35WEMYEZHXQUUWM.jpg)

*图 4: HTTP请求返回的是zip压缩包的迹象.*

在图4中，你还可以看到zip压缩包中所包含文件的文件名，**InvoiceAndStatement.lnk**。可以根据图5和图6所示的路径，使用Wireshark从流量包中提取出该文件。

**File → Export Objects → HTTP…**

![img](https://bbs.kanxue.com/upload/attach/202001/743775_XK9GRCPTYMEZK2Y.jpg)

*图5: 从pcap提取出HTTP对象.*

![img](https://bbs.kanxue.com/upload/attach/202001/743775_UQK8DJ6JF3V629A.jpg)

*图 6:从 pcap提取出zip文件.*

在BSD,Linux，或者Mac环境下，你可以很容易的确定提取出的文件是一个zip文件，获取该文件的SHA256哈希值，并在命令行环境中提取该压缩包内容。本例中，压缩包中是一个Windows快捷方式文件，你也可以用图7的方式通过获取其哈希值进行确认。

用于确定文件类型的命令是： `file` `*[filename]*`, 用于确定文件SHA256哈希值的命令是： `shasum -a 256` `*[filename]*`。![img](https://bbs.kanxue.com/upload/attach/202001/743775_S6Q5A4R34J2UJJ6.jpg)

*图7: 检查提取的zip文件及其内容.*

到 `144.91.69[.]195` 的一个HTTP请求返回的是一个Windows可执行文件。这是Trickbot的初始Windows可执行文件。追踪这个HTTP请求的HTTP流，你就可以发现如图8和图9所示的可执行文件的迹象。如图10所示，从pcap中提取出该可执行文件。

![img](https://bbs.kanxue.com/upload/attach/202001/743775_PK4CY8BJ83NM4CY.jpg)

*图 8:追踪到 144.91.69[.]195的HTTP流*

![img](https://bbs.kanxue.com/upload/attach/202001/743775_P2FN6BJU7P2HHKW.jpg)

*图 9: 返回文件是Windows可执行文件或DLL文件的迹象.*

![img](https://bbs.kanxue.com/upload/attach/202001/743775_UW5TS9DXFYZPMAJ.jpg)

*图10: 从 pcap提取出Windows可执行文件.*

之前的感染的流量会包含经TCP端口443、447或449的HTTPS/SSL/TLS流量，以及对受感染主机IP的确认。本例中，在Trickbot可执行文件的HTTP请求之后，我们可以看到在成功经TCP 449端口连上 187.58.56[.]26 之前有几次经TCP 443端口对不同IP地址的扫描尝试。使用**basic+**过滤器，你可以看到如图11和图12所示的这些连接尝试。

![img](https://bbs.kanxue.com/upload/attach/202001/743775_Y32ZSXF9HXCZWAK.jpg)

*图11:受感染的Windows主机尝试通过443端口进行TCP连接*

![img](https://bbs.kanxue.com/upload/attach/202001/743775_24EEJZXRQ8ZAPXB.jpg)

*图12: 往下拉可以看到在经TCP 449端口成功连接到 187.58.56[.]26 之前在TCP 443端口上进行了多次尝试*

经TCP 447和449端口的到不同IP的HTTPS/SSL/TLS流量的证书数据不寻常。我们可以通过在Wireshark 2.x中用 `ssl.handshake.type == 11` 或者在Wireshark 3.x中用 `tls.handshake.type == 11` 过滤查看证书发行者信息。过滤之后转到框架细节部分并展开信息，你就可以看到如图13和图14所示的证书发行信息。

![img](https://bbs.kanxue.com/upload/attach/202001/743775_95ET94GWVNBQ465.jpg)

*图13: 过滤得到HTTPS/SSL/TLS 流量, 并展开第一个在449端口的报文信息.*

![img](https://bbs.kanxue.com/upload/attach/202001/743775_HB2K75JQWVZFTRY.jpg)

*图 14: 下拉查看在TCP 449端口的第一个报文的证书发行信息*

在图14中，我们可以看到在经TCP 449端口到187.58.56[.]26的HTTPS/SSL/TLS的证书发行数据如下：

- id-at-countryName=`AU`
- id-at-stateOrProvinceName=`Some-State`
- id-at-organizationName=`Internet Widgits Pty Ltd`

sstateOrProvinceName（Some-State）和organizationName（ Internet Widgits Pty Ltd ）并不用于合法的HTTPS/SSL/TLS流量中。所以意味着，这是一个恶意流量，而且这种不同寻常的证书发行数据并不仅用于Trickbot。用于合法的HTTPS/SSL/TLS流量的正常证书发行者信息应该是怎么样的呢？查看之前经TCP 443端口访问微软主机72.21.81.200的流量，我们就可以看到如图15所示信息。

- id-at-countryName=`US`
- id-at-stateOrProvinceName=`Washington`
- id-at-localityName=`Redmond`
- id-at-organizationName=`Microsoft Corporation`
- id-at-organizationUnitName=`Microsoft IT`
- id-at-commonName=`Microsoft IT TLS CA 2`

![img](https://bbs.kanxue.com/upload/attach/202001/743775_YPWY7QKWP8GS3W3.jpg)

*图 15: 到微软主机的合法HTTPS流量中的证书数据.*

受Trickbot感染的Windows主机会使用一系列不同的IP地址检查站点来查看其IP地址。这些站点并非恶意站点 ，而且这些流量也不是恶意的。但是，这种IP地址检查在Trickbot和其他恶意软件家族中很常见。Trickbot用的合法IP地址检查服务包括：

- `api.ip[.]sb`
- `checkip.amazonaws[.]com`
- `icanhazip[.]com`
- `ident[.]me`
- `ip.anysrc[.]net`
- `ipecho[.]net`
- `ipinfo[.]io`
- `myexternalip[.]com`
- `wtfismyip[.]com`

再次申明，对自己的IP地址检查并不是恶意的。但是，当这种行为与其他行为结合在一起就变成恶意的了，如我们在本例中看到的一样。

![img](https://bbs.kanxue.com/upload/attach/202001/743775_JQAHPMCYWBR3R6V.jpg)

*图16:在经过TCP 449端口的HTTPS/SSL/TLS通信之后，立即 对受感染的Windows主机IP地址检查。这一行为单看并非恶意，但却是Trickbot感染过程的一部分 .*

Trickbot感染通常会在TCP 8082端口产生HTTP流量，将受感染主机的系统信息及从浏览器和邮件客户端获取的密码发送到Trickbot服务器上。

用以下Wireshark过滤器查看这条数据：

```
http.request and tcp.port eq 8082
```

如图17所示，有如下几条HTTP请求：

- /ono19/BACHMANN-BTO-PC\_W617601.AC3B679F4A22738281E6D7B0C5946
  E42/81/
- 170.238.117[.]187 port 8082 – ***170.238.117[.]187*** – POST
  /ono19/BACHMANN-BTO-PC\_W617601.AC3B679F4A22738281E6D7B0C5946
  E42/83/
- 170.238.117[.]187 port 8082 – ***170.238.117[.]187*** – POST
  /ono19/BACHMANN-BTO-PC\_W617601.AC3B679F4A22738281E6D7B0C5946
  E42/81/
- 170.238.117[.]187 port 8082 – ***170.238.117[.]187:8082*** – POST
  /ono19/BACHMANN-BTO-PC\_W617601.AC3B679F4A22738281E6D7B0C5946
  E42/81/
- 170.238.117[.]187 port 8082 – ***170.238.117[.]187:8082*** – POST
  /ono19/BACHMANN-BTO-PC\_W617601.AC3B679F4A22738281E6D7B0C5946
  E42/90
- 170.238.117[.]187 port 8082 – ***170.238.117[.]187:8082*** – POST
  /ono19/BACHMANN-BTO-PC\_W617601.AC3B679F4A22738281E6D7B0C5946
  E42/90

![img](https://bbs.kanxue.com/upload/attach/202001/743775_9DXMSSHYPFDQJBW.jpg)

*图 17: Trickbot产生的经TCP 8082端口的HTTP流量.*

以81结尾的HTTP POST请求发送的是保存在web 浏览器、邮件客户端和其他应用程序中的密码。以83结尾的HTTP POST请求发送的是提交到类似web浏览器等应用程序的表格数据。以90结尾的HTTP POST请求发送的是系统相关信息。追踪这些TCP或HTTP流可以看到发送的数据内容。

![img](https://bbs.kanxue.com/upload/attach/202001/743775_T3EZUWNHCW2CEPQ.jpg)

*图 18: 被Trickbot从Chrome浏览器窃取的登录信息. 这些信息由受Trickbot感染的主机用HTTP流量经TCP 8082端口发送.*

![img](https://bbs.kanxue.com/upload/attach/202001/743775_CWMUFRU3MJ6AC97.jpg)

*图 19: 受Trickbot感染主机用HTTP流量经TCP 8082端口发送系统信息. 信息以一些正在运行的进程开始.*

![img](https://bbs.kanxue.com/upload/attach/202001/743775_E8AF8WS4F5D59FP.jpg)

*图20: 受Trickbot感染的主机使用HTTP流量经TCP 8082端口发送的其他系统数据。这张图是图19的后续.*
