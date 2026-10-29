
# 安卓API自动化安全扫描

## 背景

### 解决的问题

在日常的移动端安全审计，自动化审计一直停留在应用客户端，对于安卓应用中的网络API接口长期处于空白阶段，该方案主要想解决实际工作中移动安审自动化覆盖范围不全遗漏掉API相关内容的问题，同时对公司APP端的资产进行梳理，进一步完善公司移动应用SDL流程缺失的环节。

## API自动化扫描

### 扫描流程

先说整体的扫描流程：

一、业务测试人员提交APK到检测平台;

二、检测平台对APK进行静态分析结合动态监控的方式完成API资产的收集；

- 静态分析获取APK中静态的API信息；
- 动态监控通过模拟点击+VPN代理的方式捕获应用运行时与服务端的交互API信息；

三、检测平台完成API资产收集后开始进行请求数据的清洗，包括去除重复、无效的API信息；

四、将处理过的API资产信息发送给WEB扫描器，进行应用API的自动化扫描。

### 项目框架

框架图：

![image-20221125110611962](upload/attach/202212/753425_UGC7M2XWTGRUKBW.png)

在整个过程中，主要需要解决的有以下两个问题：

- 如何实现API资产的自动化收集；
- 如何对应用内业务逻辑自动化触发。

## API资产收集

首先来看第一个问题，对于应用内API资产的收集有两个思路，一个是静态解析应用内的字符串，通过正则表达式的方式来识别出应用内的API资产，另一个思路是对应用的网络通信进行捕获来获取。鉴于一般应用的url链接都是动态拼接出来的，并且纯静态分析很难解析出API请求对应的数据格式，所以这里主要对第二种思路进行实践。

如何实现APP网络通信的捕获？

聊个老生常谈的话题：抓包，相信做过移动安全审计的小伙伴应该知道，App应用抓包不管是在协议分析还是渗透测试中都属于比较重要的一环，在拿到需要审计的应用后，首先会对应用是否加固进行检测，除了加固检测第二步就是对应用与服务端的请求进行抓包然后再根据请求数据包的内容来展开更深层次的审计。根据目的不同分析方向也有些差别，像协议分析主要是去apk中定位相应的处理函数或者其算法逻辑，对于渗透测试更多的则是对数据包中关键字段进行修改来检测服务端是否存在鉴权、越权、SQL注入等问题，但多数情况下渗透测试也会涉及到协议分析相关的工作。

### HTTP介绍

经常听到http协议、https协议，那它们有什么区别？`http`即超文本传输协议，采用明文的方式去传输数据，而`https`是`http`的升级，`https`在`http`协议的基础上加上了`SSL/TLS`功能，`http`协议负责建立客户端与服务端的网络通信，而`SSL/TLS`则负责通信的安全，包括传输数据的加密与身份的认证。

`SSL`与`TLS`的区别：`TLS`是`SSL`迭代的版本，因为`HTTP`协议传输的数据都是未加密的，所以为了保证这些明文数据能够进行安全传输，网景公司设计了`SSL`(Secure Sockets Layer)协议用于对`HTTP`协议传输的数据进行加密，`SSL`目前的版本是3.0。互联网标准化组织ISOC接替网景公司对`SSL` 3.0进行了升级，衍生出了`TLS`1.0(Transport Layer Security)，因此可以理解为`TLS`1.0=`SSL`3.1，目前`TLS`版本支持1.3。

![image-20221031175806360](upload/attach/202212/753425_KSCP555ZQECG6NS.png)

### 安卓通信框架

对于安卓应用，有以下几种较常见的http通信方式，Apache的`HttpClient`类、Java提供的`HttpsURLConnection`类、Android提供的`WebView`以及第三方库比如`OkHttp`、`Retrofit2`。

其中`HttpClinet`在安卓6.0的时候就已经被废弃，安卓官方推荐使用`HttpsURLConnection`，但`OkHttp`和`Retrofit2`使用起来更方便、功能更多，所以大部分应用都采用`OkHttp`来实现网络通信。

### 常见抓包方案

#### 抓包原理

即中间人攻击，就是在通信双方的中间建立一个代理服务器。让客户端以为这个代理服务器就是真正的服务端，这样客户端一切的请求都会先发给中间服务器并由中间服务器代理转发给真实的服务端，而真实服务端的响应也都会被中间服务器接收，再由中间服务器转发给客户端。这样这个中间服务器就可对客户端与真实服务端通信的数据进行拦截、监听、篡改、重放等操作。

![image-20221031180121756](upload/attach/202212/753425_MCKN4EH26X28VP6.png)

#### 抓包工具

对于HTTP的抓包工具在PC端的主要有`BurpSuitePro`、`Fiddler4`、`Charles`等工具，客户端主要有`HttpCanary`(小黄鸟)，对于一些不使用`HTTP/HTTPS`协议传输数据的app，还可以使用`tcpdump`来进行抓包，抓到的包可以使用`wireshark`工具进行解析，最后就是基于`frida`的`r0capture`也比较好用。

![image-20221031181734051](upload/attach/202212/753425_7RQFC6RWR9KY66W.png)

#### WIFI代理

1. 首先将安装了客户端应用的手机与安装了抓包工具的PC处于同一网段，也就是连上同一wifi；
2. 其次将手机的wifi设置为手动代理，代理主机IP设置为PC的IP，端口设置为8888（随便设置，只要和抓包工具一致）；
4. 然后打开PC上的抓包工具并配置监听端口开始进行代理抓包工作；
5. 最后还需要通过手机浏览器访问以下抓包工具的证书下载地址，进行证书安装，安卓完成后即可进行抓包。

- <a href="elink@afeK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0#2M7Y4l9`.">BurpSuite\_pro</a>
- <a href="elink@8fcK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3y4Z5L8s2y4Q4x3X3g2H3M7X3!0Q4x3V1k6K6M7$3H3`.">Charles-proxy</a>
- <a href="elink@3a7K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3W2H3N6U0c8Q4x3X3g2X3K9h3c8V1L8r3g2J5i4K6y4m8z5o6R3^5z5q4)9J5c8R3`.`.">Fiddler</a>

注意：当应用的`targetSdkVersion`到28后会发现在Android7.0及以上机型上抓不到包，主要原因是应用不再信任客户端用户自己安装的证书，除非App应用自身明确开启用户证书信任的功能。

来看下7.0之前App的配置与7.0及之后的配置有什么不一样。

6.0的`"res/xml/network_security_config.xml"`文件

```xml
<base-config cleartextTrafficPermitted="true">
<trust-anchors>
<certificates src="system" />
</trust-anchors>
</base-config>
```

7.0的`"res/xml/network_security_config.xml"`文件

```xml
<base-config cleartextTrafficPermitted="true">
<trust-anchors>
<certificates src="system" />
<certificates src="user" />
</trust-anchors>
</base-config>
```

因为我们自己安装的证书属于user域，可以看到配置取消了对user域证书的信任。

想要在7.0及7.0以上的手机上进行抓包，可以从以下几个点入手。

1. 最简单的方法，就是重新配置，修改配置文件支持对user域证书的信任。
   - 具体配置详情请看官方介绍:<mark class="encrypted">b82K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2j5$3!0E0i4K6u0r3N6s2u0S2K9h3&6A6L8X3N6Q4x3V1k6S2M7Y4c8A6j5$3I4W2M7#2)9J5c8Y4y4W2j5%4g2J5K9i4c8&6i4K6u0V1j5$3!0F1k6X3W2Y4</mark>
2. 将我们的证书添加到系统证书的目录，这样我们的证书就是系统证书，也就会被信任了。
   - 方法一：`/system/etc/security/cacerts`目录包含每个已安装根证书的问题，在有root的情况下重新挂载`/system`目录，并将我们的证书拷贝到该目录。
   - 方法二：使用Magisk自定义模块<a href="elink@a81K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6z5g2V1W2e0e0%4y4W2j5%4g2J5K9i4c8&6i4K6u0r3e0h3q4Y4K9i4y4C8g2s2u0#2M7%4c8g2M7$3g2J5b7$3g2J5N6s2x3`.">MagiskTrustUserCerts</a>，将任何用户证书识别为系统证书。

#### VPN隧道

除了可以通过WIFI设置代理进行抓包还有一种方式就是在安卓设备上创建VPN隧道来配合抓包工具进行抓包，VPN隧道属于七层协议的网络层，设备在开启VPN后会多出一个网络接口，相当于多加了个虚拟网卡，所有流量都会走这个新增的虚拟网卡，这样应用层和传输层的请求数据就都能捕获。

开启VPN可以使用`postern`应用来完成，然后配置下代理服务器地址以及规则，即可在PC端使用抓包软件进行抓包。

![image-20220920104631965](upload/attach/202212/753425_99EZUR4UB8FG6TS.png)

如果觉得上面的设置流程较麻烦，那么直接选择`HttpCanary`(小黄鸟)吧，该应用的抓包原理也是基于VPN实现的，安装完即配置完，比较方便，配合<a href="elink@9d1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2L8X3I4Q4x3X3g2A6L8X3E0Q4x3V1j5`.">AnLink</a>投屏工具用起来更丝滑。需要注意的点：有些应用会检测是否存在`HttpCanary`，所以可以先将目标软件打开后再启动`HttpCanary`应用进行抓包。

#### 透明代理

最后一种叫透明代理(路由重定向)，顾名思义就是可以让客户端感觉不到代理的存在。该方法主要依赖linux上的`iptables`命令行工具的流量转发功能，用户不需要设置代理服务器，设置下默认网关即可。当设备访问外部网络时，客户端的数据包会被转发到设置的默认网关上，通过默认网关的路由，最终到达目标服务器。该方案可以适用在设备不支持WIFI代理设置并且不能安装第三方应用的情况下，比如对机车进行抓包时。

首先需要在手机上使用`su`权限设置将设备所有tcp流量转发到指定IP(`172.20.10.9`)的电脑上。

```
iptables -t nat -A OUTPUT -d 0.0.0.0/0 -p tcp -j DNAT --to 172.20.10.9 #设置重定向
iptables -t nat -D OUTPUT -d 0.0.0.0/0 -p tcp -j DNAT --to 172.20.10.9 #取消重定向
# 可以将0.0.0.0/0替换成指定IP可以极度精细化控制每一条流。
```

然后再重定向的电脑上开启`BurpSuite`并配置对`HTTP/HTTPS`的默认端口`80`和`443`进行监听。

![image-20220920111721526](upload/attach/202212/753425_2HNCM6MZR3WGG6S.png)

### 抓包防护

根据前面的内容我们可以看到，攻击者想要抓包其实是很容易的，那作为开发者如何去增加应用的防护能力，来避免应用的业务数据被中间人抓包获取呢？这里整理了以下几种方法供大家参考；

- 方法一：使用系统API进行常规检测，包括是否设置了WIFI代理，是否开启VPN等；
- 方法二：通过设置让APP请求时不用系统代理来绕过WIFI代理抓包；
- 方法三：自定义Sooket实现HTTP/HTTPS；
- 方法四：使用证书校验的方式来验证服务端是否为信任的服务端(双向认证和证书锁定)；

实际情况中遇到的比较多的有上面的第一种，第二种和第四种，第三种目前遇到的较少。

#### WIFI代理检测

最基础的抓包就是设置WIFI代理进行抓包，所以就可以从WIFI代理入手，检测当前的WIFI环境是否安全，是否被设置了代理服务器和代理端口，当检测到存在代理时就判定为用户正在使用抓包，这样我们就能采取一些措施，比如退出应用；

WIFI代理的检测代码如下：

```java
public static boolean isWifiProxy(Context context) {
final boolean IS_ICS_OR_LATER = Build.VERSION.SDK_INT >= Build.VERSION_CODES.ICE_CREAM_SANDWICH; // 判断安卓版本
String proxyAddress;
int proxyPort;
if (IS_ICS_OR_LATER) {
proxyAddress = System.getProperty("http.proxyHost"); // 获取代理主机
String portStr = System.getProperty("http.proxyPort"); // 获取代理端口
proxyPort = Integer.parseInt((portStr != null ? portStr : "-1"));
} else {
proxyAddress = android.net.Proxy.getHost(context);
proxyPort = android.net.Proxy.getPort(context);
}
Log.i("代理信息", "proxyAddress :" + proxyAddress + "prot :" + proxyPort);
return (!TextUtils.isEmpty(proxyAddress)) && (proxyPort != -1);
}
```

#### 设置不走系统代理

对于WIFI代理抓包，除了可以通过检测进行防护，还可以通过网络请求时设置不走系统代理来绕过，检测代码如下：

```java
public void run() {
Looper.prepare();
OkHttpClient okHttpClient = new OkHttpClient.Builder().
proxy(Proxy.NO_PROXY). // 使用此参数，可绕过系统代理直接发包
build();
Request request = new Request.Builder()
.url("http://www.baidu.com")
.build();
Response response = null;
try {
response = okHttpClient.newCall(request).execute();
Toast.makeText(this, Objects.requireNonNull(response.body()).string(), Toast.LENGTH_SHORT).show();
} catch (IOException e) {
e.printStackTrace();
}
Looper.loop();
}
```

#### VPN检测

VPN检测的检测有两种方式，因为创建VPN隧道会新增一个网络接口，一般是`"tun0"`或`"ppp0"`，所以VPN的第一种检测原理是对设备网络接口进行检测，当发现存在`"tun0"`或`"ppp0"`的接口时，则判定存在VPN抓包。

```java
public boolean isInVPN() {
try {
Enumeration<NetworkInterface> networkInterfaces = NetworkInterface.getNetworkInterfaces();
while(networkInterfaces.hasMoreElements()){
String name = networkInterfaces.nextElement().getName();
if (name.equals("tun0") || name.equals("ppp0")) {
return true;
}
}
} catch(SocketException e) {
e.printStackTrace();
}
return false;
}
```

第二种检测则比较简单，直接使用系统服务，获取当前网络的状态来判定是否使用了VPN。
