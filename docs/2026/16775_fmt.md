
# [翻译]webserver特征抓取与对抗

当敌人的军队在靠近我们时，我们就要想方设法阻止他们前进，或是把他赶走，或是让他的前进徒劳无功，或是通过虚拟的承诺延迟他的行军，然后在他的军队离开之后欺骗他。我们应该永远对能增加他人资源的行为保持警惕，打击敌人试图获得力量的行为。

-考底利耶, 政事论.

特征提取和指纹识别是一种获取目的主机操作系统，web服务器类型和版本等信息的方法。一旦被黑客获取了目的主机操作系统等信息，他就能轻易地找到特定版本的漏洞，并对它发起攻击。今天我们来看看webserver特征提取在web服务器上是怎么进行的，以及其对抗方法。在这篇文章中，我们可以看到Apache和IIS 8服务器的例子。

## Apache

假设我在Apache服务器上搭建了一个域名为`1b9K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4y4Z5N6h3&6&6j5g2)9J5k6h3y4G2L8g2!0q4y4#2)9&6b7g2)9^5y4q4!0q4y4#2!0n7c8q4)9&6x3g2!0q4y4#2!0m8b7W2)9&6z5g2!0q4x3#2)9^5x3q4)9^5x3W2!0q4z5g2!0n7b7W2)9&6x3g2!0q4y4g2!0m8c8g2!0m8x3W2!0q4z5q4)9^5x3#2!0n7c8q4!0q4z5q4!0n7c8q4!0n7b7W2!0q4y4W2)9&6z5q4)9&6x3#2!0q4y4g2)9&6b7#2!0n7x3q4!0q4y4#2)9&6y4q4!0m8z5q4!0q4y4g2)9&6x3q4)9^5y4q4!0q4y4#2!0m8y4#2)9^5c8q4!0q4y4W2)9&6y4W2!0n7z5g2!0q4y4g2!0n7b7#2)9^5c8W2!0q4y4W2)9^5z5g2!0n7c8g2!0q4y4g2)9^5z5q4!0n7x3s2N6W2j5W2!0q4y4W2)9&6b7#2)9^5c8q4!0q4y4g2)9^5b7g2!0m8x3g2!0q4y4g2)9&6z5g2!0m8z5q4!0q4y4#2)9&6b7g2)9^5y4q4!0q4y4q4!0n7c8W2!0m8x3g2!0q4y4W2)9^5x3g2!0m8c8W2!0q4x3#2)9^5x3q4)9^5x3W2!0q4y4q4!0n7c8g2)9^5b7W2!0q4y4g2!0m8y4W2)9^5x3W2!0q4c8W2!0n7b7#2)9^5b7#2!0q4z5g2!0n7b7W2)9&6x3g2!0q4y4g2!0m8c8g2!0m8x3W2!0q4z5q4)9^5x3#2!0n7c8q4!0q4y4g2!0m8y4q4)9&6c8W2!0q4y4g2)9&6b7#2!0m8z5q4!0q4y4W2)9^5z5q4)9&6x3g2!0q4y4#2)9&6b7g2)9^5y4q4!0q4y4W2)9&6b7#2)9^5c8q4!0q4y4g2)9^5b7g2!0m8x3g2!0q4y4g2)9&6z5g2!0m8z5q4!0q4y4#2)9&6b7g2)9^5y4q4!0q4y4#2!0n7c8q4)9&6x3g2!0q4y4#2!0m8b7W2)9&6z5g2!0q4y4q4!0n7z5q4)9^5b7g2!0q4z5q4!0m8c8g2!0n7c8W2!0q4z5g2)9&6y4#2!0m8c8g2!0q4y4q4!0n7z5q4)9^5x3q4!0q4y4q4!0n7z5q4!0m8b7g2!0q4y4q4!0n7z5q4)9^5c8q4!0q4y4g2!0m8c8q4)9&6z5q4!0q4y4g2)9&6b7#2!0m8z5q4!0q4y4#2)9&6b7g2)9^5y4q4!0q4z5g2!0m8x3g2!0n7y4g2!0q4z5g2)9&6c8q4!0m8x3W2!0q4c8W2!0n7b7#2)9^5b7#2!0q4y4g2)9^5x3#2)9^5c8W2!0q4y4q4!0n7z5q4)9^5b7W2!0q4z5g2)9&6c8q4!0m8x3W2!0q4z5q4!0n7c8W2)9&6z5g2!0q4y4W2!0m8x3q4!0n7y4#2!0q4x3#2)9^5x3q4)9^5x3R3``.

![](https://bbs.kanxue.com/upload/attach/201704/706972_sixfaf9ccp04rv6.jpg)

在上面的例子中，黑客试图打开名为`admin.php`的页面，而这个页面在我的服务器上并不存在，因此服务器会答复一个包含web服务器类型，操作系统和脚本语言的页面。这个页面给出了大量的信息。

通过`telnet`进行指纹识别是一种传统且通用的方法。黑客打开命令行或是终端(command line or termial)，输入命令`telnet d0fK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4y4Z5N6h3&6&6j5g2)9J5k6h3y4G2L8b7``.`. 80`。当登录成功之后，输入`head/ HTTP/1.0`，这个命令会返回服务器信息。

![](https://bbs.kanxue.com/upload/attach/201704/706972_96gj9d5eeiizd4n.jpg)

指纹识别工具有很多，我在这里只介绍一个，Id serve。让我们来看看怎么使用Id server来进行信息收集(banner grab)。

![](https://bbs.kanxue.com/upload/attach/201704/706972_x689ykl5obrbns0.jpg)

接下来我们介绍在Apache服务器上能够采取的对抗方法，至少在某种程度上预防指纹识别。我们能够在Apache web服务器的配置文件`httpd.conf`上进行一些修改来对抗指纹识别。打开 `httpd.conf`，把选项`Server Signature`的值改为`off`。这样，当某个被访问的页面不存在的时候，就不会再显示任何关于服务器的信息了。

![](https://bbs.kanxue.com/upload/attach/201704/706972_svln1b2c4x2mgc1.jpg)

在`httpd.conf`文件中，把`Server Tokens`的值由`Full`改为`Prod`，可以使其只显示最少的服务器信息，如下所示。

![](https://bbs.kanxue.com/upload/attach/201704/706972_sg8yugzgb8xy3w6.jpg)

![](https://bbs.kanxue.com/upload/attach/201704/706972_ju94q01kewgyj5k.jpg)

上示图片虽然泄漏了web服务器是Apache，但不再会显示它的版本了。用考底利耶的话来说，就是它延迟了敌人的前进。下面是我们设置的选项。

![](https://bbs.kanxue.com/upload/attach/201704/706972_6xbah4x1bqkvekl.jpg)

## IIS 8

现在，假设我们把网站的服务器由Apache改为微软web服务器IIS 8的最新版。为了预防错误页面(error page)泄漏IIS服务器的信息，我们可以设置经过定制的错误页面。现在让我们用IDserve工具对IIS 8服务器进行指纹识别。

![](https://bbs.kanxue.com/upload/attach/201704/706972_zhj3ek1hohz8o15.jpg)

上面这张图显示了服务器的版本。现在我们怎样来预防这种情况呢？微软提供了一个叫做UrlScan的免费下载软件，它能帮我们轻松地处理HTTP请求。接下来，我们打开默认路径在`C:WindowsSystem32inetservUrlscan`下的UrlScan的配置文件`UrlScan.ini`,然后它`RemoveServerHeader`的值由0改为1。

![](https://bbs.kanxue.com/upload/attach/201704/706972_vnwbzgpjea3vgxl.jpg)

如下图所示，这次不会再泄漏服务器的版本信息了。

![](https://bbs.kanxue.com/upload/attach/201704/706972_dbda4e274cxdwi9.jpg)

通过把服务器名称设置为其他的值，我们能进一步误导攻击者。具体做法是，把`RemoveServerHeader`的值改为`0`，并且把`AlternateServerName`修改为我们指定的值(本例中为Nginx)。

![](https://bbs.kanxue.com/upload/attach/201704/706972_e4rmg5c8609xebh.jpg)

这样我们就能在攻击者对我们的网站进行指纹识别的时候，误导他们。

**注意：采用这种方法并不能阻止一个意志坚定的攻击者安全部此次我们的服务器信息**

原文链接：`b6cK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3S2S2j5$3E0W2M7X3y4G2L8$3I4Q4x3X3g2U0L8$3#2Q4x3V1j5J5x3o6p5K6i4K6u0r3x3o6W2Q4x3V1k6%4k6h3u0K6k6i4u0$3k6i4u0Q4x3X3c8T1j5h3&6F1k6i4u0Y4M7X3q4T1j5X3W2F1k6#2)9J5k6r3q4F1k6q4)9J5k6r3y4G2N6h3&6@1k6i4u0E0k6h3q4K6N6i4u0W2M7#2)9J5c8R3``.

本文由看雪翻译小组 梦野间 翻译

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
