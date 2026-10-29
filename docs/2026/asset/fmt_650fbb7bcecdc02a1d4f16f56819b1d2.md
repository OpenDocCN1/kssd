
```xml
<network-security-config xmlns:tools="605K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4y4U0K9r3g2E0j5i4y4Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2j5$3!0E0i4K6u0r3N6r3!0G2L8s2x3.">
<!--证书校验-->
<domain-config>
<domain includeSubdomains="true">xxxx.com</domain>
<trust-anchors>
<certificates src="@raw/cert"/>
</trust-anchors>
</domain-config>
<!--公钥校验-->
<domain-config>
<domain includeSubdomains="true">xxxx.com</domain>
<pin-set expiration="2099-01-01" tools:ignore="MissingBackupPin">
<pin digest="SHA-256">vzXV96/gpZMyyNNhyTdjtX0/NUVYTtmYqWcVVaUtTdQ=</pin>
</pin-set>
</domain-config>
</network-security-config>
```

`AndroidManifest.xml`：

```xml
<application android:networkSecurityConfig="@xml/network_security_config">
```

使用`webview`请求上面配置的域名时，系统会自动校验证书信息；

```java
private class MyWebViewClient extends WebViewClient {
    private String checkflag="checkCerts";
    // 是否忽略证书校验

    public void setCheckflag(String checkflag) {
        this.checkflag = checkflag;
    }

    @Override
    public void onReceivedSslError(WebView view, SslErrorHandler handler, SslError error) {
        if("trustAllCerts".equals(checkflag)){//忽略证书校验
            handler.proceed();
        }else {
            handler.cancel();
            Toast.makeText(MainActivity.this, "证书异常，停止访问", Toast.LENGTH_SHORT).show();
        }
    }
}
```

### 双向认证

双向认证是基于单向认证（客户端只认证服务端）的基础上，添加了服务端对客户端证书的认证。除了客户端会对服务端证书进行认证外，服务端还会去认证客户端的证书是否有效。

#### 证书认证与证书锁定(SSL pinning)的区别

这里需要区分下证书认证和证书锁定的不同，android系统默认帮我们预置了150多个可信证书，这些证书可以在设置->安全->信任的凭据中看到。https特点就是证书认证，因为服务端用的证书是从android认可的证书颁发机构购买的证书，而在android中也已经内置了这些机构的信任证书，所以默认情况下是可以直接访问服务器而无需在客户端设置证书信息，证书认证是对android预置的这些证书进行认证，所以通常情况下抓包时需要导入抓包工具的证书到系统中，让系统信任抓包软件的证书。而证书锁定(SSL pinning)是对开发者自己的私有证书进行安全校验。

SSL证书大概分为三类。

1. 由可信证书颁发机构颁发的证书，比如Symantec，Go Daddy等机构，约150多个。在手机"设置->安全->信任的凭据"中可以查看。
2. 不是可信证书机构颁发的证书，比如抓包软件的证书，12306网站的证书等，都是非可信证书机构颁发的证书
3. 公司自己颁发的证书。

这三类证书中，只有第一种在使用时不会出现安全提示，不会抛出异常。

### 反抓包防护

所有的防护都不是一劳永逸的，防护只能提高逆向的门槛，而不能完全解决风险，经过多年的对抗，攻击者早已研究出如何绕过前面的抓包防护了，看下攻击者用的比较多的一些绕过方法。

**一、透明代理(路由重定向)**，可以对抗WIFI代理检测、设置不走系统代理、VPN检测。

原理：HTTP/HTTPS的默认端口是80和443，我们在BurpSuite上设置透明代理并监听这两个端口。在透明代理中应用会认为我们用BurpSuite模拟服务器开放端口就是真实服务器，实际上是将手机的TCP协议的路由重定向到了我们电脑IP地址中进而BurpSuite会进行代理服务器转发。

详情请看常见抓包方案中的透明代理+抓包内容；

**二、对于证书锁定的绕过可以采用hook的方式。**

在手机已经root的情况下，可以通过xposed框架下的JustTustMe/SSLUnping或者分析代码定制hook来对抗证书锁定，至于双向认证的话还需要分析应用中客户端证书的位置以及使用密码，并将找到的证书导入到抓包应用中。

#### 证书锁定绕过

1. 利用Xposed+JustTrustMe插件直接绕过证书锁定

   原理：JustTrustMe的原理是将前面提到的，常见的HTTP请求库中用于设置校验证书的相关API进行了Hook，创建一个自定义的未校验证书的`X509TrustManager`对象并替换掉应用安全有证书校验的`X509TrustManager`对象，达到绕过的效果。

2. 通过Frida绕过SSL单向验证

   使用Frida绕过的思路也是对客户端的验证函数进行hook，因为SSLPinning的原理就是内置一个安全的证书，当客户端与服务端通信时验证服务端的证书是否安全，这样我们就可以通过hook将这个验证函数一直返回通过，那么就达到了绕过验证的目的。

#### 双向证书校验绕过

因为服务端会验证客户端的证书，所以应用一般会把证书和证书秘钥存放在app中，一般在`app/assert`目录或`raw`目录下，后缀名一般为`.p12`或`.pks`。找到客户端的证书并导入到抓包工具中再进行抓包就行了。有些情况证书可能会有加密，这就需要手动去逆向解密或逆向密码了。

下面利用soul来实际操作下，这里使用的抓包工具是Charles-proxy，别的工具思路是一样的，只是最后导入证书的位置有所差异。

在没有绕过SSL pinning的情况下，我们发现利用抓包工具对应用的数据包进行抓取，什么数据也看不到。在使用Xposed+JustTrustMe绕过单向验证。

![Image](https://bbs.kanxue.com/upload/attach/202212/753425_EXEQ83KF43VTTB6.png)

可以看到，开启JustTrustMe后可以抓取到一些数据，但服务端返回400报错，提示"No required SSL certificate was sent"。根据这个提示说明服务端开启了双向认证机制，要求客户端发送所需的SSL证书，所以还需要去绕过服务端的双向认证机制。

既然服务端会对客户端的证书进行验证，那么客户端证书肯定在客户端会保留一份，那么能否直接在`apk`中找到呢。实际可以在`apk`的`assets`目录下发现确实存在我们需要的证书。

![Image](https://bbs.kanxue.com/upload/attach/202212/753425_KVPVZUGTFB93U6G.png)

现在有了客户端的证书，但这还不够，因为使用该证书的时候会提示我们输入密码，还需要找到证书的密码，思路和刚才一样，在应用中找。

把应用`apk`丢到`jadx`中解析，然后利用该工具的搜索功能直接搜索关键字`"client.p12"`，很快就能找到证书相关的逻辑。

![image-20220922122629419](https://bbs.kanxue.com/upload/attach/202212/753425_CB3PW986WSRSYYF.png)

在`v4_1.load`函数的第二个参数`v1`就是密码，通过向上回溯分析，发现`v1`来源于`getStorePassword`函数，该函数是一个`jni`函数，通过名字大概可以猜到。

![image-20220922123934474](https://bbs.kanxue.com/upload/attach/202212/753425_J7GMGYK7AAT2D49.png)

接着分析该函数所在的类可以知道该类会加载一个`soul-netsdk`动态库，那么就知道了`getStorePassword`的实现应该就在这个动态库中。

![Image](https://bbs.kanxue.com/upload/attach/202212/753425_K5QR7UC9U3FSUJ7.png)

从`apk`中找到这个动态库并丢到IDA中解析可以找到下面这个函数，可以看到该函数直接返回了一个字符串，到此为止得到这个字符串就是该证书的密码`"soulapp123!@#1"`。

![download](https://bbs.kanxue.com/upload/attach/202212/753425_FC469Y73VDC3P78.png)

最后把我们前面获取到的证书导入到抓包工具中，让工具使用我们提供的证书。通过以下的选项打开我们的导入窗口。

Proxy -> SSL Proxying Settings

![image-20220922123544492](https://bbs.kanxue.com/upload/attach/202212/753425_3FN6W6ZPERTG442.png)

一切顺利之后点击OK，然后再次进行抓包。

![Image](https://bbs.kanxue.com/upload/attach/202212/753425_T5KJ2NDX6JPTJZA.png)

可以看到我们成功抓到了数据包，并且也可以看到里面的内容。

### 技术选型

看了前面几种抓包方案，那么如何选择适合我们的方案呢？我们的需求在于如何自动化，所以交互越少的方案是我们的优选方案；这里对前面几种方案进行了简单对比：

- 方案一：WIFI代理/透明代理，在手机设备上设置代理服务器，将设备上的数据转发至代理服务器(PC)端，以此完成对应用的网络请求捕获；
  - 优点：数据包捕获的很完整，PC端可以配合一些扫描工具，完成对应用接口进行扫描；(xary+burp)
  - 缺点：存在对抗问题，需要增加绕过抓包防护的功能；
- 方案二：VPN隧道，基于NetBare框架自实现一个VPN代理应用安装在手机设备上，通过拦截虚拟网卡对应用网络请求进行拦截；
  - 优点：数据捕获的很完整，并且不依赖PC端抓包工具，因为框架开源所以更灵活，对单个应用的数据包进行自定义格式保存，用作后续的各种分析，比如漏洞扫描，合规检测；
  - 缺点：存在对抗问题，需要添加抓包防护绕过的功能；
- 方案三：HOOK获取，基于Hook，对目标应用中的一些常见通信函数进行hook，直接抓取原始数据；
  - 优点：基本不存在抓包对抗问题，可以将捕获的数据保存下来，用作后续的各种分析，比如漏洞扫描、合规检测；
  - 缺点：捕获数据的完整度依赖hook点，存在请求覆盖不全，数据包不完整的问题；

从数据捕获的完整性来看，方案一和方案二都符合我们的要求，但考虑到方案一需要依赖代理服务器，并且需要配置抓包工具，部署起来比较复杂，而方案二只用在手机设备上安装一个应用即可，所以最终没考虑方案一直接选择的是方案二，同时结合方案三的方式来实现对API资产进行收集。

## 自动化测试

在确定了应用API资产收集方案后，再来看第二个问题，如何对应用内业务逻辑自动化触发？

即在不人工介入的情况下自动对应用内的业务逻辑进行触发，使客户端与服务端进行网络请求，再结合前面的捕获方案，完成应用内API接口的收集工作。在一番调研后得知，如果想实现该需求可以使用自动化测试相关的框架，所以对目前比较流行的自动化测试框架进行简单的了解学习。

### 自动化测试框架

自动化测试就是通过机器代替人工来对目标进行测试，测试人员通过编写测试脚本并在脚本的实际运行中添加对业务逻辑的判断，实现测试自动化。一般QA团队用的较多，软件安全有时也会用到，像一些做游戏外挂、抢红包、视频刷赞、刷阅读量、爬虫的也都会用到这类技术。常见自动化测试框架：`Monkey`、`MonkeyRunner`、`UIAutomator`、`UIAutomator2`、`Appium`、`AirTest+Poco`。

在对上面几款测试框架进行检测使用后，发现其实这些测试框架操作控件实现自动化测试的方式都大同小异。主要分为以下两种：

- 一种是通过随机生成测试事件流并发送给目标系统来完成对目标的测试，这种方式因为是随机生成的事件流，事件流不可控，所以用的比较少，一般用来做稳定性测试。代表框架有`Monkey`。
- 另一种则是通过脚本的方式生成可控的事件流，之后测试框架按一定顺序发送给目标系统来完成对目标的测试，相对第一种因为事件是测试人员自己生成的所以测试的更精准。

对于第二种脚本的生成又分为两种方式：

最常见的就是脚本录制，主要思想是记录人为操作时控件的坐标位置和发生的事件，之后通过回放录制的脚本来完成测试事件流，现在很多测试框架都提供了比较方便的录制回放功能；

另一种是通过工具（比如源码、`UIAutomatorviewer`等）分析UI布局来获取测试界面的控件布局、找到目标控件的ID、名称、描述或位置等信息。测试人员再编写脚本来让测试框架得到控件对象，并对控件对象执行一系列事件操作，像`Robotium`、`UIAutomator2`等，这里以`UIAutomator2`举例，下图为一个应用的控件布局。

![image-20221101102855621](https://bbs.kanxue.com/upload/attach/202212/753425_J2KJUP7BEFKF4GC.png)

### 控件遍历

在实际情况中，我们需要测试的应用是未知的，如何在不介入人工的情况下对未知的应用进行自动化测试？

这种情况下，随机事件的自动化方案不适合，又因为目标应用不是固定的，录制脚本的自动化方案也不适用，能选的就只有通过分析UI布局来实现自动化测试的方案。

最终决定采用`UIAutomator2`来通过控件遍历的方式进行自动化测试，遍历逻辑如下图。

1. 安装测试应用，通过`adb`启动应用；
2. 解析UI布局，获取界面中具备点击属性的控件对象；
3. 计算当前页面Hash值，对可点击控件进行模拟点击；
4. 根据点击前后页面的Hash值，判断是否进入新页面，进行深度优先遍历；
   - 相同进入下一控件模拟点击；
   - 不同进入结束条件判断，不结束则跳到第二步；
5. 结束遍历；

![](https://bbs.kanxue.com/upload/attach/202212/753425_R9E56X26WM8K5CS.png)

其中的一个难点就在于对页面的判断，判断点击后是否进入到新的页面，这里列出几个判断依据：

- 点击前后页面的`Activity`名
- 点击前后页面的可点击控件数
- 点击前后页面截图的相似度

还有个需要规避的问题：业务层级遍历的越深整体测试的时间越长，因此需在全面与效率之间取出一个较能接受的中间值用作遍历深度，同时设置超时时间，在满足一定时长后自动终止测试。

![image-20221101101624285](https://bbs.kanxue.com/upload/attach/202212/753425_TGRXDKAC53WGV4U.png)

## 优化点

1. 对于一些应用的抓包防护需要处理；
2. 自动化遍历对于一些复杂页面，比如登录时处理还待优化；
3. 对于做了API接口防护，比如具备签名、数据加密的接口还不支持；

## 参考

- [HttpClient配置SSL绕过https证书以及双向认证](https://bbs.kanxue.com/elink@3c0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3P5i4N6T1L8h3q4K6N6r3g2J5i4K6u0r3M7q4)9J5c8U0p5@1y4U0R3@1z5e0j5^5i4K6u0W2K9s2c8E0L8l9`.`.)
- [Android安全之绕过WebView SSL Pinning抓HTTPS数据](https://bbs.kanxue.com/elink@7a9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6*7K9s2g2S2L8X3I4S2L8W2)9J5k6i4A6Z5K9h3S2#2i4K6u0W2j5$3!0E0i4K6u0r3M7q4)9J5c8U0f1J5z5o6f1H3x3e0M7@1z5b7`.`.)
- [安卓 https 证书校验和绕过](https://bbs.kanxue.com/elink@930K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0L8%4m8&6k6Y4g2@1N6i4u0W2i4K6u0W2j5$3!0E0i4K6u0r3j5X3I4G2k6%4y4Q4x3X3c8V1k6i4c8S2K9h3I4K6i4K6u0r3x3U0l9J5x3e0l9#2x3e0t1H3z5e0f1H3y4e0b7$3x3e0W2T1)
- [Android APP漏洞之战(6)—HTTP/HTTPs通信漏洞详解](https://bbs.pediy.com/thread-270634.htm)
