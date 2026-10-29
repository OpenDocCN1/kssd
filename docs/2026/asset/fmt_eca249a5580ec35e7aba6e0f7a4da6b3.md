
# 故事

作为测试的一部分，客户端提供了文档根目录列表，我在使用Intruder启动完整扫描之前，正在Burp的转发器中查看几个选择页面。一页看起来非常有趣的是`secret.html`。我提出了它的请求并获得了秘密内容：

Burp Repeater请求显示秘密内容。

这似乎很容易，但我不抱怨，所以我转向浏览器，输入URL并告诉“不，你看不到数据”。

被拒绝访问Firefox中的秘密内容。

这不是我的预期，也许它与会话相关，但在Burp的代理中查看请求看起来非常相似。Firefox引入了一些额外的标题，但这是可以预料的。

在Burp的代理中的Firefox请求。

如果不是Firefox，我可以使用`curl`查看内容吗？看到我的Repeater选项卡打开，我抓住了那里的URL并将其放到命令行：

```
$ curl -i 53bK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5&6x3W2)9J5k6e0p5$3z5q4)9J5k6e0m8Q4x3X3f1&6x3%4y4W2j5%4u0W2N6q4)9J5k6h3S2@1L8h3H3`.
```

好吧，这很奇怪，URL被破坏但我可以解决它：

```
$ curl -i fe2K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5&6x3W2)9J5k6e0p5$3z5q4)9J5k6e0m8Q4x3X3f1&6x3#2)9J5c8Y4y4W2j5%4u0W2N6q4)9J5k6h3S2@1L8h3H3`.
```

HTTP/1.1 200 OK
Content-Type: text/html
Accept-Ranges: bytes
ETag: "4177750045"
Last-Modified: Wed, 18 Apr 2018 22:36:19 GMT
Content-Length: 40
Date: Fri, 20 Apr 2018 19:28:13 GMT
Server: lighttpd/1.4.45

You are not allowed to see that content

我没想到，所以我让Burp给我一个`curl`命令，然后运行：

```
$ curl -i -s -k -X $'GET' \
    -H $'Host: 192.168.0.93' -H $'Connection: close' \
    $'b97K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5&6x3W2)9J5k6e0p5$3z5q4)9J5k6e0m8Q4x3X3f1&6x3%4y4W2j5%4u0W2N6q4)9J5k6h3S2@1L8h3I4Q4x3U0M7'.
```

什么都没有回来，有点头疼，我注意到URL又被打破了。在这一点上，我应该意识到出了问题，但我没有，我只是修复了URL并再次尝试：

```
$ curl -i -s -k -X $'GET' \
    -H $'Host: 192.168.0.93' -H $'Connection: close' \
    $'baaK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5&6x3W2)9J5k6e0p5$3z5q4)9J5k6e0m8Q4x3X3f1&6x3%4y4W2j5%4u0W2N6q4)9J5k6h3S2@1L8h3I4Q4x3U0M7'.
```

HTTP/1.1 200 OK
Content-Type: text/html
Accept-Ranges: bytes
ETag: "4177750045"
Last-Modified: Wed, 18 Apr 2018 22:36:19 GMT
Content-Length: 40
Date: Fri, 20 Apr 2018 19:29:32 GMT
Server: lighttpd/1.4.45

You are not allowed to see that content

好吧，非常奇怪的事情，Burp可以得到秘密的东西，但Firefox和curl不能，让我们试试`netcat`。我建立连接，从Repeater复制请求并...

```
$ nc 192.168.0.93 80
GET secret.html HTTP/1.1
Host: 192.168.0.93
Connection: close
```

什么都没有回来，连接保持打开但服务器没有响应。也许我有一个拼写错误所以我把请求放入一个文件并尝试使用：

```
$ cat getsecretrequest | nc 192.168.0.93 80
```

另一个挂连接，没有响应。在这一点上我真的很困惑，这就是我所拥有的：

- Burp可以获得秘密内容
- Curl无法使用Burp提供的URL获取秘密
- Burp提供的curl命令无法得到秘密
- 任何使用`netcat`的尝试都会挂起

请求之间必定存在一些差异，但我看不到它们，所以让我们降低一下，看看Wireshark。

首先，Burp请求：

Wireshark查看Burp的请求显示秘密消息

接下来由Burp创建的curl命令：

Burp用拒绝消息创建的curl命令的Wireshark视图

存在一些差异，此请求具有curl用户代理和额外的接受标头。WAF和其他简单的保护系统经常回复用户代理检查，所以也许就这么简单，让我们再次尝试卷曲删除这些额外的标题：

```
$ curl -i -s -k -X $'GET' \
    -H $'Host: 192.168.0.93' -H $'Connection: close' \
    -H $'User-Agent:' -H $'Accept:' \
    $'42dK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5&6x3W2)9J5k6e0p5$3z5q4)9J5k6e0m8Q4x3X3f1&6x3#2)9J5c8Y4y4W2j5%4u0W2N6q4)9J5k6h3S2@1L8h3I4Q4x3U0M7'.
```

由Burp创建的curl命令的Wireshark视图，没有用户再次或接受标题但是另一个拒绝消息

盯着这些请求他们看起来都一样但后来我发现了差异，Burp请求`secret.html`，curl正在请求`/secret.html`。额外的领先`/`必须是差异，这也解释了为什么URL Burp给了我和它创建的curl命令都缺少`/`。Burp能够在请求中指定页面名称时发出请求，但curl要求页面成为URL的一部分。

看到我将无法使用curl重现请求，让我们回到`netcat`，看看我们是否可以解决为什么失败。我们来看看Wireshark中的netcat连接：

通过netcat发送请求的Wireshark视图

并排这两个请求看起来与去了`secret.html`而不是`/secret.html`的请求相同，但必须有一些区别，无论多么小。十六进制视图是什么样的？

Wireshark以十六进制显示Burp请求

Wireshark以十六进制显示Burp请求

经过一番凝视我终于发现了差异，Burp请求是使用DOS行结尾（`\r\n`），netcat使用的是Unix（`\n`）。看到我在文件中得到了请求，使用vim很容易更改行结尾，只需打开文件，输入：

```
:set ff=dos
```

然后保存它。如果您不是vim用户，`dos2unix`包中的`unix2dos`应用程序也是一个选项。

转换后，让我们再试一次：

```
$ cat get_secret_request_dos | nc 192.168.0.93 80

HTTP/1.1 200 OK
Content-Type: text/html
Accept-Ranges: bytes
ETag: "4169396764"
Last-Modified: Wed, 18 Apr 2018 22:42:18 GMT
Content-Length: 43
Connection: close
Date: Fri, 20 Apr 2018 21:35:12 GMT
Server: lighttpd/1.4.45

This is top secret stuff you shouldn't see
```

大奖！现在我可以通过`netcat`重现请求，让我们检查它是否是领先`/`有所不同。我把`/`放到位并再试一次：

```
$ cat get_secret_request_dos | nc 192.168.0.93 80

HTTP/1.1 200 OK
Content-Type: text/html
Accept-Ranges: bytes
ETag: "4177750045"
Last-Modified: Wed, 18 Apr 2018 22:36:19 GMT
Content-Length: 40
Connection: close
Date: Fri, 20 Apr 2018 21:39:01 GMT
Server: lighttpd/1.4.45

You are not allowed to see that content
```

我们有它，请求`secret.html`提供访问权限，请求`/secret.html`被拒绝。这种类型的请求不能通过浏览器或任何其他将完整URL作为参数的工具来进行，该请求只能通过理解页面和主机是两个独立实体的东西来进行。

所以现在我可以重现这个问题，但我不知道为什么第一个地方存在差异。这很烦人，但我想我可以在某个时候与开发人员讨论它，看看他们是否有任何想法。

稍后... 除了进行应用程序测试之外，客户端还要求审查服务器配置，并且作为其中提供lighttpd配置的一部分，当我发现这一行时，我正在努力：

```
url.rewrite-once = (

    "^/secret.html" => "/not_permitted.html"

)
```

看起来相当简单，对以`/secret.html`开头的页面名称的任何请求都将在内部重定向到`/not_permitted.html`。但是，由于我要求`secret.html`，而不是`/secret.html`，此规则不适用于我，我不会被重定向，因此可以查看秘密内容。

知道这一点，我想知道我是否可以使用curl或浏览器查看内容。我尝试的第一个网址是：

```
65eK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5&6x3W2)9J5k6e0p5$3z5q4)9J5k6e0m8Q4x3X3f1&6x3#2)9J5c8W2)9J5k6g2)9J5c8Y4y4W2j5%4u0W2N6q4)9J5k6h3S2@1L8h3H3`.
```

但curl和我尝试的所有浏览器在发出请求之前简化了这一点，所以我最终还是使用了`/seror.html`。我尝试了各种`./`和`../`组合，都失败了，直到我最终成功了：

```
55dK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5&6x3W2)9J5k6e0p5$3z5q4)9J5k6e0m8Q4x3X3f1&6x3#2)9J5c8W2)9J5k6g2)9J5k6g2)9J5c8W2)9J5k6g2)9J5c8W2)9J5k6g2)9J5k6g2)9J5c8W2)9J5c8W2)9J5c8W2)9J5c8Y4y4W2j5%4u0W2N6q4)9J5k6h3S2@1L8h3H3`.
```

反转它直到它停止工作，我发现虽然点得到简化，但额外的斜线没有，所以后面的URL是有效的，并获取秘密数据，因为所请求的文件是`//secret.html`，它与正则表达式。

```
a12K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5&6x3W2)9J5k6e0p5$3z5q4)9J5k6e0m8Q4x3X3f1&6x3#2)9J5c8W2)9J5c8Y4y4W2j5%4u0W2N6q4)9J5k6h3S2@1L8h3H3`.
```

在确认这个URL在各种浏览器中工作后，我有一些可靠的东西，我可以在报告中作为利用的例子。它需要一段时间来完成所有工作，但这比给出Repeater的截图并说“看，我得到你的数据但我不知道如何”要好得多。

我希望您已经了解了我如何调试此漏洞的有用信息。它有助于表明计算机是确定性的，并且有一个原因背后的事情，有时它只需要一些工作来找出规则是什么。一旦你了解规则，玩游戏并获胜往往要容易得多。

# 其他Web服务器

我决定尝试对抗Apache，NGINX和IIS，这三个人都拒绝了请求，并发出了`400 Bad Request`响应。我还确认这三个人都对DOS或Unix行结尾感到满意。

## Apache

```
$ cat get_secret_request | nc 192.168.0.93 80

HTTP/1.1 400 Bad Request
Date: Fri, 20 Apr 2018 20:26:09 GMT
Server: Apache/2.4.25 (Debian)
Content-Length: 304
Connection: close
Content-Type: text/html; charset=iso-8859-1

<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>400 Bad Request</title>
</head><body>
<h1>Bad Request</h1>
<p>Your browser sent a request that this server could not understand.<br />
</p>
<hr>
<address>Apache/2.4.25 (Debian) Server at 192.168.0.93 Port 80</address>
</body></html>
```

错误日志文件中还有以下条目：

```
[Fri Apr 20 20:26:09.244512 2018] [core:error] [pid 31042] [client 192.168.0.3:47832] AH00126: Invalid URI in request GET secret.html HTTP/1.1
```

## NGINX

```
$ cat get_secret_request | nc 192.168.0.93 80

HTTP/1.1 400 Bad Request
Server: nginx/1.10.3
Date: Fri, 20 Apr 2018 19:22:39 GMT
Content-Type: text/html
Content-Length: 173
Connection: close

<html>
<head><title>400 Bad Request</title></head>
<body bgcolor="white">
<center><h1>400 Bad Request</h1></center>
<hr><center>nginx/1.10.3</center>
</body>
</html>
```

NGINX日志文件中没有条目。

## IIS

```
$ cat get_secret_request_dos | nc microsoft.com 80

HTTP/1.1 400 Bad Request
Content-Type: text/html; charset=us-ascii
Server: Microsoft-HTTPAPI/2.0
Date: Sun, 22 Apr 2018 18:00:10 GMT
Connection: close
Content-Length: 324

<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN""http://www.w3.org/TR/html4/strict.dtd">
<HTML><HEAD><TITLE>Bad Request</TITLE>
<META HTTP-EQUIV="Content-Type" Content="text/html; charset=us-ascii"></HEAD>
<BODY><h2>Bad Request - Invalid URL</h2>
<hr><p>HTTP Error 400. The request URL is invalid.</p>
</BODY></HTML>
```

我没有访问IIS日志来检查条目。

# 防御

最简单的防御措施并不是在文档根目录中存储任何您不想被浏览的东西。如果`secret.html`存储在文档根目录之外，它仍然可以被任何页面使用，但是无法在URL中引用它。

在一些实验中，`mod_access`函数似乎传递了一个已被清理的页面名称，如果有需要会添加前导斜杠，并且会删除多余的斜杠。因此即使我们发错格式错误的请求，下面的拒绝规则也会使页面返回`403 Forbidden`：

```
$HTTP["url"] =~ "^/(secret.html)$" {

    url.access-deny = ("")

}
```

然后就可以将`server.errorfile-prefix`选项设置为传递自定义403页面。

如果您想使用重写规则来修复它，最简单的方法是从正则表达式中删除前导斜杠：

```
"secret.html" => "/not_permitted.html"
```

这个规则将阻止访问名称中包含`secret.html`的任何页面。如果这是网站上唯一一个此类型的页面，那么解决方案就会生效；如果其他人拥有`view_secret.html`页面，您只是为他们创建了一大堆问题，试图弄清楚他们再也看不到其页面的原因。

下面是一个更好的规则，其意思是，从页面名称的开头开始，`secret.html`后可跟任意数量的斜杠。这可以防止原始绕过手段，如零斜杠和其后使用的一两个斜杠都是允许的。

```
"^[/]*secret.html" => "/not_permitted.html"
```

我仍然认为这不是一个完美的解决方案，因为其他能够插入页面名称中的字符也许能绕过此规则。

最后一个解决方案将取决于`secret.html`的目的。如果它是由其他页面引入的模板文件，则可以在其中构建一些逻辑性的东西，这样的话，除非以正确的方式访问这个页面，否则它不会显示其内容。例如通过执行相同的身份验证和授权检查来引用它的页面。

# 问题和漏洞的摘要

对于那些没有阅读完整帖子的人，以下内容是本文的重点：

- 对`lighttpd`的HTTP请求必须使用DOS（`\r\n`）行结尾，而不是Unix（`\n`）；
- `lighttpd`接受没有前导斜杠的请求，Apache，NGINX和IIS的页面的请求都拒绝这些请求；
