
其次，你还需要进入“设置”并启用“使用持久性地址”。使用此设置时，当您停止 OnionShare 服务器然后再次启动它时，您将获得与上次相同的洋葱地址。这使您可以灵活地重新启动计算机并再次启动 OnionShare，而无需切换 OnionShare 地址。

最后，你可能希望在无头 Linux 服务器上运行匿名保管箱，而不是你普通日常工作的电脑。OnionShare 附带两个二进制文件，`onionshare`（命令行版）和`onionshare-gui`（图形版）。它们都共享相同的配置文件 - Linux 下的`~/.config/onionshare/onionshare.json`文件。因此，使用 OnionShare 的图形版本，选择您想要的所有设置，然后将`onionshare.json`复制到您的服务器，并像这样运行 OnionShare：

```
$ onionshare --receive

OnionShare 2.0 | 505K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6G2L8X3W2G2L8Y4y4Z5j5i4u0W2i4K6u0W2L8%4u0Y4i4K6u0r3

Connecting to the Tor network: 100% - Done

Setting up onion service on port 17612.

 * Running on a01K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5J5y4#2)9J5k6e0m8Q4x3X3f1H3i4K6u0W2x3g2)9K6b7e0p5%4y4U0p5J5i4K6u0r3 (Press CTRL+C to quit)



Files sent to you appear in this folder: /home/user/OnionShare



Warning: Receive mode lets people upload files to your computer. Some files can potentially take control of your computer if you open them. Only open things from people you trust, or if you know what you are doing.



Give this address to the sender:

e61K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4j5J5L8%4S2H3L8$3I4Z5N6Y4W2V1x3$3E0Z5y4o6c8V1M7Y4b7$3k6%4c8*7y4e0N6$3y4%4N6%4N6h3^5J5N6s2N6V1k6r3y4%4x3%4S2Z5K9h3@1J5K9$3N6#2x3Y4k6$3j5$3g2%4N6Y4q4V1i4K6u0W2L8$3&6A6L8$3^5`.



Press Ctrl+C to stop the server

127.0.0.1 - - [18/Feb/2019 10:48:04] "GET / HTTP/1.1" 200 -

127.0.0.1 - - [18/Feb/2019 10:48:06] "GET /static/css/style.css HTTP/1.1" 200 -

127.0.0.1 - - [18/Feb/2019 10:48:07] "GET /static/js/receive-noscript.js HTTP/1.1" 200 -

127.0.0.1 - - [18/Feb/2019 10:48:07] "GET /static/js/receive.js HTTP/1.1" 200 -

127.0.0.1 - - [18/Feb/2019 10:48:07] "GET /static/js/jquery-3.3.1.min.js HTTP/1.1" 200 -

127.0.0.1 - - [18/Feb/2019 10:48:07] "GET /static/img/logo.png HTTP/1.1" 200 -

127.0.0.1 - - [18/Feb/2019 10:48:08] "GET /static/img/logo_large.png HTTP/1.1" 200 -

127.0.0.1 - - [18/Feb/2019 10:48:10] "GET /static/img/favicon.ico HTTP/1.1" 200 -

Feb 18, 10:48AM: Upload of total size 102.1 KiB is starting

=> 101.9 KiB       2016_tax_return.pdf

Received: /home/user/OnionShare/2019-02-18/10.48.18/2016_tax_return.pdf

127.0.0.1 - - [18/Feb/2019 10:48:20] "POST /upload-ajax HTTP/1.1" 200 -

127.0.0.1 - - [18/Feb/2019 10:48:21] "GET /static/img/ajax.gif HTTP/1.1" 200 -

```

### 丰富的语言

![img](https://bbs.kanxue.com/upload/attach/202001/665727_P9H9P5WHJCN8KVK.jpg)

最后，OnionShare 2 最有意思的事情之一是它现在已被翻译成十二种新语言，使得相当多的人可以接触到 OnionShare。OnionShare 2 翻译的一些主要语言包括西班牙语（4 亿母语人士），葡萄牙语（2.15 亿），俄语（1.7 亿）和日语（1.3 亿）。（这些数字[来自 Babbel 杂志](https://bbs.kanxue.com/elink@58aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2T1j5h3u0T1k6h3I4Q4x3X3g2U0L8$3#2Q4x3V1k6W2L8W2)9J5c8X3#2S2k6$3q4*7K9h3&6W2i4K6u0r3N6r3S2W2i4K6u0V1x3e0m8Q4x3X3c8E0L8%4y4@1i4K6u0V1M7%4m8G2K9$3g2F1i4K6u0V1L8r3q4F1k6%4g2S2k6$3g2K6i4K6u0V1K9h3&6Q4x3X3c8@1K9r3g2Q4x3X3c8%4L8%4u0D9k6l9`.`.)。）

新语言包括：孟加拉语（বাংলা），加泰罗尼亚语（Català），丹麦语（Dansk），法语（Français），希腊语（Ελληνικά），意大利语（Italiano），日语（日本語），波斯语（فارسی），巴西葡萄牙语（Português Brasil），俄语（Русский），西班牙语（Español）和瑞典语（Svenska），未来将包括更多语言。如果您是母语为非英语的人并且有兴趣为 OnionShare 做贡献，您可以在[维基上找到帮助说明](https://bbs.kanxue.com/elink@17fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6E0K9h3y4S2K9r3k6D9k6h3g2Q4x3V1k6G2L8X3W2G2L8Y4y4Z5j5i4u0W2i4K6u0r3N6$3W2C8K9g2)9J5c8W2c8J5j5h3&6K6L8r3q4@1K9h3&6Y4)。

希望你喜欢 OnionShare 2!

翻译：看雪翻译小组 一壶葱茜

校对：看雪翻译小组 Nxe

原文地址：a72K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0K9h3y4S2K9r3k6D9k6h3g2Q4x3X3g2U0L8$3#2Q4x3V1j5J5x3o6p5&6i4K6u0r3x3o6u0Q4x3V1k6G2L8X3W2G2L8Y4y4Z5j5i4u0W2i4K6u0V1x3W2)9J5c8R3`.`.

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
