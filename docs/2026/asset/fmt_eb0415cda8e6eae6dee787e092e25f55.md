
# [翻译]开源情报搜集101

渗透测试基本都是从信息收集阶段开始的。这篇文章介绍了如何搜集互联网上公开信息来对目标有个大概的了解。收集的数据可以用来识别服务器、域名、版本号、漏洞、错误配置、发现可被攻击的点以及敏感信息泄漏等。继续读下去吧。

通过开源情报收集技术可以找到很多数据，尤其是那些有大量在线服务的公司，总会有些代码泄漏、在论坛上提问泄漏技术细节、被遗忘的二级域名、包含元数据的PDF文件（可以用来帮助搞定一个网站）。甚至简单的谷歌搜索都会找到一些有趣的东西。拿到目标名字（域名）后，我们可以做下面这些事：（排名不分先后）

## 1. Whois查询

找到管理员联系信息和其他邮件地址。这些邮件地址经常也是网站的注册用户。通过一些泄漏的数据库或一些像[HaveIBeenPwned](https://bbs.kanxue.com/elink@d8aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Z5j5i4k6W2K9h3u0W2k6h3&6H3N6$3&6W2k6q4)9J5k6h3y4G2L8g2)9J5c8R3`.`.)的搜索服务来查看某个邮箱的相关密码是否已被泄漏。

![](http://bbs.pediy.com/upload/attach/201705/735073_9qw8zx7xd9rskfg.png)

在[8c9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Z5j5i4k6W2K9h3u0W2k6h3&6H3N6$3&6W2k6q4)9J5k6h3y4G2L8b7`.`.](https://bbs.kanxue.com/elink@0c9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Z5j5i4k6W2K9h3u0W2k6h3&6H3N6$3&6W2k6q4)9J5k6h3y4G2L8g2)9J5c8R3`.`.) 搜索的一个泄漏密码的例子。

除了邮箱地址，`whois`查询还可以返回IP历史信息，域名过期时间甚至手机号码（用来做社会工程学攻击）。

![](https://bbs.kanxue.com/upload/attach/201705/735073_g22rkzt01extxr6.png)

查询`whois`记录 ，`whois.domaintools.com` 是一个不错的选择。

## 2. 谷歌高级搜索

使用"`site`"来限制到特定的域名，寻找`php`（或者其他文件类型的文件）、`txt`或日志文件：

```
site:*.example.org ext:php | ext:txt | ext:log
```

有几次我们用这种方法找到了一些有趣的文件（比如日志文件），这些文件包含敏感信息和Web应用的完整路径。你可组合这些搜索语句，使用减号来去处一些特定的搜索结构。

![](https://bbs.kanxue.com/upload/attach/201705/735073_ho8itdmhenbgi8b.png)

用谷歌搜索寻找`phpinfo()`文件。

## 3. 搜索目标域名的文档

文件类型包括 `PDF`、`Excel`、`Word`和`PPT`。这些文档可能包含着一些可以帮助你进行其他攻击的信息。很多情况下，这些文档属性里包含的元数据（作者等）正好就是网站应用的一个用户名。

```
site:*.example.org ext:pdf | ext:doc| ext:docx | ext:ppt | ext:pptx | ext:xls | ext:xlsx | ext:csv
```

你可以下载这些文件到本地，然后用一些元数据提取软件来提取元数据，或者查看每个文件的属性来看看是否有一些有用的信息。

如果想了解更多关于谷歌高级搜索，可以参考：[bd6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2Y4L8$3!0Y4L8r3g2Q4x3X3g2U0L8#2)9J5k6h3W2F1i4K6u0r3j5h3c8$3j5h3&6U0k6h3c8Q4y4h3k6K6k6h3q4J5j5$3R3`.](https://bbs.kanxue.com/elink@8bcK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2Y4L8$3!0Y4L8r3g2Q4x3X3g2U0L8#2)9J5k6h3W2F1i4K6u0r3j5h3c8$3j5h3&6U0k6h3c8Q4y4h3k6K6k6h3q4J5j5$3R3`.)。另外，[Google Hacking Database](https://bbs.kanxue.com/elink@a53K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2W2P5s2m8D9L8$3W2@1i4K6u0V1k6r3u0Q4x3X3g2U0L8$3#2Q4x3V1k6Y4L8$3!0Y4L8r3g2Q4x3X3c8Z5j5h3y4C8K9h3&6Y4i4K6u0V1k6r3q4@1j5h3u0S2M7$3g2Q4x3V1j5`.)（现在搬到`exploit-db`上了）提供一些预先设定好的搜索语句帮助你搜索一些有趣的东西。

![](https://bbs.kanxue.com/upload/attach/201705/735073_ehqa4hbhqra45t4.png)

此图为`exploit-db`上的 Google Hacking Database。

## 4. 检查`robots.txt`文件

可以找到一些隐藏的、有趣的目录。很多购物网站、框架和内容管理系统有着很好的目录结构。所以管理员目录是`/admin`或者`/administrator`。如果不是，`robots.txt`文件很可能包含着你想要寻找的目录。

![](https://bbs.kanxue.com/upload/attach/201705/735073_6vz0po7a9hszq4l.png)

上图是一个流行网站的`robots.txt`文件。

## 5. 查看HTML源码

来寻找 购物/内容管理系统/框架等信息。识别这些应用类型可以帮助缩小攻击范围（有漏洞的部分：插件和主题等）。比如，如果你看页面信息看到 `wp-content` 你就可以确定这个一个WordPress网站了。

有很多浏览器插件可以帮助来识别网站使用的框架。Firefox的`Wappalyzer`可以很好的识别服务器类型、客户端框架和第三方插件等。

![](https://bbs.kanxue.com/upload/attach/201705/735073_i5qa9k0j77olap0.png)

`wappalyzer`对`80bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2%4L8%4u0V1M7s2u0W2M7%4y4Q4x3X3g2G2M7X3N6Q4c8e0N6Q4z5f1q4Q4z5o6c8Q4c8e0S2Q4b7f1k6Q4z5o6k6Q4c8e0g2Q4z5o6S2Q4b7f1u0Q4c8e0k6Q4z5o6y4Q4z5o6g2Q4c8e0g2Q4z5o6k6Q4b7U0g2Q4c8e0y4Q4z5o6m8Q4z5o6t1`.

## 6. 第三方提供商信息

通常情况下，如果你的目标网站是由第三方提供商创建的，那么你就很可能会在网站主页底部看到各种“Powered by 第三方开发公司”。

使用这个来跟踪搜索承包商（第三方提供商）的网站也可能收获不少，浏览网站可能会发现网站使用的框架和版本号。承包商很可能在客户的网站上留了`test`/`admin`账户。

根据我的经验，很多网站管理员/开发者经常使用的密码是他们公司名字的变形（客户的公司或承包商的公司），在后面跟一些数字或者特殊字符（可能没有特殊字符）。比如，如果承包商名字是“Example Developers ”，然后 `001Example`、`Example001`、`00example`、`example00`等很可能就会是目标网站使用的密码。

## 7. 查看LinkedIn信息

来寻找目标公司的高级管理员、经理和非技术职员。通常，弱密码都是那些非技术管理者使用的。查看网站“关于我们”部分，也可以找到一些“软目标”（非技术的管理者）。

基于找到的一些邮箱地址，可以推导出用户名的格式。一旦用户名格式确定了，就可以得到一组邮件地址和对应的用户名，这些可以用来进行其他攻击，包括登录暴力破解，甚至[弱密码重置功能](https://bbs.kanxue.com/elink@079K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5i4m8H3M7$3g2U0j5$3!0Q4x3X3g2U0L8$3#2Q4x3V1k6E0j5i4y4K6i4K6u0V1j5h3y4U0L8%4g2F1N6q4)9J5k6s2m8%4L8X3W2F1k6#2)9J5k6r3!0J5i4K6u0V1K9r3!0%4i4K6u0V1N6$3g2Q4x3X3c8Z5j5h3y4C8k6h3c8Q4x3X3c8E0N6h3I4@1K9i4m8D9k6g2)9J5k6s2g2K6k6i4u0Q4x3X3c8S2j5$3y4G2N6h3&6@1M7#2)9J5k6s2g2K6K9h3&6Y4i4K6u0V1N6$3g2S2K9#2)9J5k6s2u0W2M7$3g2@1i4K6u0V1N6r3!0C8k6h3&6K6i4K6u0V1k6X3!0J5i4K6u0V1M7r3q4K6M7%4N6G2M7X3c8K6i4K6u0V1j5K6u0V1y4X3x3H3z5o6x3I4x3K6M7%4)。（不止一次，我们发现搜索邮箱地址和可能的用户名，再加上弱密码，就可以完全搞定目标网站。）

## 8. 进行IP地址相关检查

很多时候网站会因为和其它一个比较弱的网站托管在同一个服务器上而受到影响。使用IP地址反查，你可以找到其它的相关目标。[Bing有个很出色IP搜索功能](https://bbs.kanxue.com/elink@d44K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Z5k6h3I4H3i4K6u0W2j5X3W2F1k6#2)9J5k6h3#2A6j5%4u0G2M7$3!0X3N6q4)9J5k6h3y4G2L8g2)9J5c8W2)9J5x3$3q4H3k6i4S2Q4x3V1j5I4z5q4)9J5c8X3g2F1i4K6u0V1g2g2y4Q4x3V1j5I4x3o6l9H3x3g2)9J5c8W2)9J5k6o6p5`.)。

![](https://bbs.kanxue.com/upload/attach/201705/735073_oxitjx141n4lbss.png)

Bing的IP搜索功能可以用来寻找在同一服务器上的网站。

[you get signal](https://bbs.kanxue.com/elink@317K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4W2G2N6h3N6W2N6s2y4A6k6$3&6S2L8q4)9J5k6h3y4G2L8g2)9J5c8Y4c8G2L8$3I4K6i4K6u0r3N6$3g2T1i4K6u0V1M7$3W2@1k6i4y4Q4x3X3c8G2L8W2)9J5k6s2N6W2j5W2)9J5k6s2y4W2M7Y4k6W2M7W2)9J5c8R3`.`.) 和 [IP Address](https://bbs.kanxue.com/elink@0beK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3W2H3i4K6u0V1j5h3c8V1M7X3g2K6M7#2)9J5k6h3!0J5k6#2)9J5c8Y4u0W2N6X3g2J5M7$3g2Q4x3X3c8D9L8$3!0C8N6i4m8Q4x3V1k6J5k6i4k6W2M7Y4y4W2i4K6u0V1K9i4m8Q4x3X3g2H3K9s2l9`.) 也提供`lookup`反向查询服务。

![](https://bbs.kanxue.com/upload/attach/201705/735073_o2no9m5t2jf8zhr.png)

在 You Get singal，你可以提供域名和IP地址。

作为检查IP地址的一部分，注意域名的`A`记录和`PTR`记录也很重要。有时候，因为配置错误，使用`PTR`或`A`记录的网站可能会连接到其他的网站。这些信息可以通过`nslookup`和`dig`命令来获取：

```
dig PTR example.org
nslookup example.org
```

## 9. 枚举二级域名

来寻找容易入手或者比较弱的入口。评估目标客户的线上资产，二级域名枚举是非常重要的一步，不管是由于他们业务需求分配的，还是因为错误配置导致的。

进行二级域名枚举有很多工具，比如：`dnsrecon`，`subbrute`，`knock.py`，使用谷歌`site`操作符或者像`dnsdumpster`和`virustotal.com`这些网站。这些工具多使用大量的常见二级域名比如：`admin`、`pages`、`people`、`hr`、`downloads`、`blog`、`dev`、`staging`等等。这些二级域名跟在主域名后面——`example.org`，生成一系列可能的二级域名像`admin.exapmle.org`，`pages.example.org`，`people`等等。可以通过进行DNS查询来判断这些猜测的域名是否存在。

![](https://bbs.kanxue.com/upload/attach/201705/735073_red1196o8a7jecp.png)

使用`dnsrecon`来暴力破解二级域名。

## 10. 寻找不同请求的HTTP状态码和响应头

合法的页面、不存在的页面、跳转的页面、目录名称等。寻找响应头里微小的变化、额外的空格和冗余值等。

![](https://bbs.kanxue.com/upload/attach/201705/735073_7i5fmxszste72wu.png)

对`X-Frame-Options`头做了很细小的修改，额外的空格使得该头部无效。

另外，注意`CSP`头，它们包含着允许加载脚本的域名和源。有时，`CSP`列出的域名或`CDN`上不安全是脚本是你执行`XSS`payload的唯一方式:)
