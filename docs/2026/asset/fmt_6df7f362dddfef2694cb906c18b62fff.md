
# [原创]反爬虫之有个操作也许可以检测aiohttp、httpx，requests也尴尬？

## 前言

最近发现一个很6的东西。这个问题是在差不多半个月前，在技术群里一个老哥发现的问题，然后在群里跟大家讨论。

这个网站，请求的时候，`requests`正常：

![](https://bbs.kanxue.com/upload/attach/202304/835366_N4A2XZ8WF3EBKRR.jpg)

原始的`curl`也可以

![](https://bbs.kanxue.com/upload/attach/202304/835366_TWHXBY2QRPHRPYK.jpg)

`aiohttp`，直接报错

![](https://bbs.kanxue.com/upload/attach/202304/835366_NJV8UVD3FWK3YYQ.jpg)

`httpx`，也直接报错：

![](https://bbs.kanxue.com/upload/attach/202304/835366_DR3VCJ89PWXH8PC.jpg)

不过`httpx`的报错要明显点，这就进入了有意思的环节

## 分析

看到这里，估计就有朋友开始冷笑了：搞爬虫大部分不都用`requests`的吗，谁会用`httpx`和`aiohttp`这些啊，感觉这个是不是没啥用啊。

别急，朋友，在文章的下面会说这个，当然如果你赶时间或者觉得没啥意思，那就没必要浪费时间继续看了

### 1.httpx

拿着上面的报错，去网上搜，找到如下`issues`:

> ```
> 316K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6W2L8X3y4G2k6r3g2Q4x3V1k6Z5N6s2c8H3P5q4)9J5c8X3W2K6M7%4g2W2M7#2)9J5c8U0p5#2y4U0p5`.
>
> fe3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6W2L8X3y4G2k6r3g2Q4x3V1k6Z5N6s2c8H3P5q4)9J5c8X3W2K6M7%4g2W2M7#2)9J5c8U0p5K6y4U0x3`.
> ```

看完里面各位大佬的一顿分析，说是`hyper`库的问题：

![](https://bbs.kanxue.com/upload/attach/202304/835366_T76BVD9PJN8MFTH.jpg)

![](https://bbs.kanxue.com/upload/attach/202304/835366_MMAET25WC9GQQKR.jpg)

然后`hyper`库的开发者，如下链接回复：

> ```
> a20K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6H3P5i4c8Z5L8$3&6Q4x3X3c8Z5P5i4m8W2M7W2)9J5c8X3R3I4x3g2)9J5c8X3W2K6M7%4g2W2M7#2)9J5c8U0p5I4x3H3`.`.
> ```

大概意思是这个不是一个问题，而是`http`请求的严格性判断问题，请求头的协议，按国际标准，是不能出现 `[Cache-Control]` 这种带有特殊符号作为响应头的键名的，所以报错

而`requests`却可以，或许是因为`requests`的校验不严格，直接就放过了：

![](https://bbs.kanxue.com/upload/attach/202304/835366_URV55WSF3MA8AJZ.jpg)

而，浏览器访问也是可以的：

![](https://bbs.kanxue.com/upload/attach/202304/835366_3ZSXJU6F7HJFVVZ.jpg)

那么我个人就有理由认为

**这是一个`bug`，`httpx`和`aiohttp`都存在的`bug`**

`httpx`的作者，对这个问题那段时间确实在尝试解决，`github`机器人都想关闭了，`httpx`作者还不想放弃：

![](https://bbs.kanxue.com/upload/attach/202304/835366_UNNEKRE7B5X6UND.jpg)

且至今没解决，遇到的人还不少，至少，上个月都还有人在说这个问题

![](https://bbs.kanxue.com/upload/attach/202304/835366_JCYKE4D5NAZFRDG.jpg)

卧槽，这时间，上周都还有人在问啊：

![](https://bbs.kanxue.com/upload/attach/202304/835366_2FJ3CUC49CJWR7M.jpg)

其中也有说解决办法，看到有个老哥说改`h11`的源码，改成这样：

![](https://bbs.kanxue.com/upload/attach/202304/835366_B3VSRBQDUEGHS5U.jpg)

但是报错依然在

![](https://bbs.kanxue.com/upload/attach/202304/835366_5VCQZFTXXVKBMER.jpg)

另外有个老哥说了这个方法：

![](https://bbs.kanxue.com/upload/attach/202304/835366_YGCWQNC7AE82H3B.jpg)

```
h11.readers.header_field_re = re.compile(b"(?P<field_name>[-!#$%&'*+.^`/|~0-9a-zA-Z]+):[ \t](?P<field_value>([^\\x00\\s]+(?:[ \t]+[^\\x00\\s]+))?)[ \t]*")
```

我把这个代码直接放我代码里，报错了，根本没有这个属性

![](https://bbs.kanxue.com/upload/attach/202304/835366_Y7FW5XUFKH5QC8X.jpg)

经过一顿查阅，他换了个属性名，是这个：

![](https://bbs.kanxue.com/upload/attach/202304/835366_D3CWQSV9BDT2XVF.jpg)

```
import h11

from h11 import _readers

import re

h11._readers.header_field_re = re.compile(b"(?P<field_name>[-!#$%&'*+.^`/|~0-9a-zA-Z]+):[ \t](?P<field_value>([^\\x00\\s]+(?:[ \t]+[^\\x00\\s]+))?)[ \t]*")
```

但是我试了，换成了新的报错：

![](https://bbs.kanxue.com/upload/attach/202304/835366_WNR2E8MB4Q4TBEF.jpg)

也许是正则表达式写的不完美：

![](https://bbs.kanxue.com/upload/attach/202304/835366_8XWNN683DJ37Z84.jpg)

正则表达式通用匹配一下，然后就可以了：

![](https://bbs.kanxue.com/upload/attach/202304/835366_98P7JWWGETE8M8G.jpg)

```
import httpx

import h11

from h11 import _readers

import re

# h11._readers.header_field_re = re.compile(b"(?P<field_name>[-!#$%&'*+.^`/|~0-9a-zA-Z]+):[ \t](?P<field_value>([^\\x00\\s]+(?:[ \t]+[^\\x00\\s]+))?)[ \t]*")

h11._readers.header_field_re = re.compile(b"(?P<field_name>.*?):[ \t](?P<field_value>.*?)")

headers = {'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/111.0.0.0 Safari/537.36'}

url = ' 

# url = ' 

resp = httpx.get(url=url, headers=headers)

print(resp.text)
```

但是，我换了一个网站，也就是那个`issue`里某华某为的社区地址，不出正常的结果：

![](https://bbs.kanxue.com/upload/attach/202304/835366_ZA7GNVQZ56Y4YRC.jpg)

所以，修改正则表达式并不是一个通用的方案。

插一句，在请求的时候，就不要开代理或者抓包软件了，不然他报错更奇怪：

![](https://bbs.kanxue.com/upload/attach/202304/835366_C53APMV788R8PJX.jpg)

综上所述，也就是，只要服务器的返回头埋的坑够多，这个方法根本无法完美解决

那么这个问题，`httpx`作者就给了个这个解释就没下文了

> ```
> 0d2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6W2L8X3y4G2k6r3g2Q4x3V1k6Z5N6s2c8H3P5q4)9J5c8X3W2K6M7%4g2W2M7#2)9J5c8U0M7$3y4#2)9J5x3$3W2K6M7%4g2W2j5$3!0E0L8h3g2F1N6q4)9J5k6o6p5K6y4U0M7@1z5e0R3@1y4e0R3`.
> ```

就这么不了了之了。。。

因为根据`http`协议的响应头原理，确实不会出现这种不标准的字段

更官方的解释可以去查查`http`协议原理，或者看看以下资料：

> ```
>  
>  
> 649K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2E0L8%4A6A6L8r3I4S2i4K6u0W2L8%4u0Y4i4K6u0r3P5X3S2Q4x3X3c8o6e0W2)9J5c8X3c8G2j5%4y4Q4x3V1k6s2L8r3!0K6M7$3q4J5P5g2)9J5c8W2u0W2M7%4m8G2L8Y4y4W2i4K6g2X3K9r3g2S2k6r3g2J5
> ```

我的理解如下：

> 客户端需要先解析响应头，通过响应头的一些规范来解析返回的响应体的，既然你响应头都报错了，那后续的响应体解析自然也不会往下走了，直接报错退出。

### 2.aiohttp

对于`aiohttp`来说，像上面那么加正则表达式是无效的：

![](https://bbs.kanxue.com/upload/attach/202304/835366_7HNHJT75CRM3CVS.jpg)

![](https://bbs.kanxue.com/upload/attach/202304/835366_RF2Q2MC6KKXNG7E.jpg)

改完发现并没有用

![](https://bbs.kanxue.com/upload/attach/202304/835366_22SN2BCE8444PXF.jpg)

这还有说，改成`1`，也试了，不行

![](https://bbs.kanxue.com/upload/attach/202304/835366_3TR6MFYQYNT9EE7.jpg)

更多的就不演示了

反正就是一顿查找，发现`aiohttp`上并没有合理方案解决

## 针对的解决方案

### 1.用requests

上面有了，这里就不展示了

但是`requests`我们知道，它是不支持`http2.0`协议的

### 2.用正则表达式替换

上面也有了，这里不贴了

但是不能完美覆盖后续的响应头特殊参数

### 3.用urllib3处理

这个库用的倒不多，但可以直接请求：

![](https://bbs.kanxue.com/upload/attach/202304/835366_XNG6NNQF34RH3JA.jpg)

但是他也是个可以直接拿来请求的库，相比`requets`，那很多功能还是没有。

其实`requests`本质就是借助了`urllib3`库的

所以，这个库，不出意外应该也是不支持`http2.0`的

## 用非常规的请求库尝试

既然目前的方法，除了`requests/urllib3`，其他都不行，那我在这个基础之上，强制`http2`，是不是就可以干掉`requets`库了，`python`的常规请求库直接无可用的，直接干死这些爬虫？

想的挺美的，哈哈

### 1.用tls-client试试

虽然他是用来对抗`tls`的，试试呢：

![](https://bbs.kanxue.com/upload/attach/202304/835366_HCQ9MT8JM9GEYRW.jpg)

可行。为啥，因为他原理就是完全模拟浏览器，大概看了它的源码，用`curl_impersonate`库，打包成了一个`dll`（具体怎么打包的不可知，这部分没开源，插一句，**据群友反馈，也是因为这个`dll`，会导致内存泄漏**），然后可以直接用，上面说过了，浏览器可以访问，那这个库肯定也可以访问了

那么测试那个某为的网址：

![](https://bbs.kanxue.com/upload/attach/202304/835366_NW8M742Q4EW23WZ.jpg)

不行，被识别了，牛吧，还是得是某为啊

### 2.用curl_cffi试试

![](https://bbs.kanxue.com/upload/attach/202304/835366_U9PF38YT22MBY4K.jpg)

再来测试那个某为的网址：

![](https://bbs.kanxue.com/upload/attach/202304/835366_N5ZST5RBYDXMK95.jpg)

这么对比，看来`curl_cffi`才是获胜者啊，只能说，牛逼啊！！！

## 实现一个anti aiohttp、httpx的服务

简单的用`fastapi` 实现一个简单的服务，设置了下响应头，直接无情的嘲讽（只能设置英文状态的，要不然嘲讽力度拉满，嘿嘿）

![](https://bbs.kanxue.com/upload/attach/202304/835366_PFZ8GCX3MU5ME9S.jpg)

### 1.用httpx请求

`httpx`代码不变，只是把`url`换了，果然报错

![](https://bbs.kanxue.com/upload/attach/202304/835366_GFZ94P89QSWSDKT.jpg)

### 2.用aiohttp请求

再来看`aiohttp`，果然报错，哈哈哈哈

![](https://bbs.kanxue.com/upload/attach/202304/835366_68DQSFJXHQS5PSW.jpg)

### 2.用http.client请求

![](https://bbs.kanxue.com/upload/attach/202304/835366_828W8W98H3X48KK.jpg)

### 3.用urllib3、requests测试

![](https://bbs.kanxue.com/upload/attach/202304/835366_7X5M42KEB7SHR3W.jpg)

![](https://bbs.kanxue.com/upload/attach/202304/835366_QJN7P72854VVFS8.jpg)

还得是这哥俩啊，直接能跑

### 4.用node的request请求看看

![](https://bbs.kanxue.com/upload/attach/202304/835366_YYMY9WBWNM9KMRD.jpg)

刺激，也给防住了。

### 5.用golang 看看

刺激啊，也没有正常返回

![](https://bbs.kanxue.com/upload/attach/202304/835366_9S3SXX5KJAKS5PM.jpg)

### 6.用postman看看

`postman`也直接无返回

![](https://bbs.kanxue.com/upload/attach/202304/835366_CPT7BRBNUDANT4P.jpg)

### 7.用安卓看看
