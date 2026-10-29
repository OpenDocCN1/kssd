
# [原创]android 4.4 从url.openConnection到Dns解析

其实并不是要分析Dns流程的，一开始是分析Android设置wifi代理，怎么起到全局代理的作用的。

因为坏习惯，在线看源码，全凭记忆，没有做笔记的习惯，忽略了一个点，直接分析到dns了，回过头才发现分析过了。而之前群里有人问应用进程hook hosts文件能不能改dns，很久之前分析过4.1，记得是有个远程进程负责dns解析的，但是太久了，细节都忘光了。

所以分析完了wifi代理、dns后记录下吧，年纪大了记忆力真的不如以前了。

下一篇 wifi代理流程。

除了http外，ftp和webview是怎么代理的。

以及6.0以下不用root，不申请特殊权限，怎么设置wifi代理。由此引出app检测代理的必要，之前考虑实际的攻击场景可能就是钓鱼wifi，dns劫持等，所以认为一些app检测代理或者不走代理主要是应对渗透抓包，但是忽略了安卓6.0及以下版本恶意app可以设置wifi代理，把http(s)代理到服务器的，如果没有做正确的证书校验或者诱导用户安装证书，https也可以中间人。

以及除了root后监控网卡解析出http代理和使用VpnService外怎么通用的hook达到代理的目的，比如自己封装发送http协议，不使用标准api，第三方框架等、比如直接socket连接，写http。因为工作中还是有一部分这样不走寻常路的应用，往往web渗透的同事即没root机器，版本还很高（比如8.0的手机信任证书），甚至还在驻场的情况，对其进行支持就很头疼（多开框架）。

```java
URL url = new URL("63eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2T1j5h3W2V1N6g2)9J5k6h3y4G2L8b7`.`.");

Proxy proxy = new Proxy(Proxy.Type.HTTP, new InetSocketAddress("192.168.2.1", 8081));

HttpURLConnection connection = (HttpURLConnection) url.openConnection(/*proxy*/);

connection.setConnectTimeout(10000);

//                        connection.addRequestProperty("accept","text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,\*/\*;q=0.8,application/signed-exchange;v=b3");

connection.addRequestProperty("accept-encoding", "gzip, deflate, br");

//                        connection.addRequestProperty("accept-language", "zh-CN,zh;q=0.9");

connection.setRequestMethod("GET");

InputStream is = connection.getInputStream();

Map<String, List<String>> map = connection.getHeaderFields();

for (Map.Entry<String, List<String>> entry : map.entrySet()) {

Log.e("zhuo", entry.getKey()+"="+entry.getValue());

}

GZIPInputStream gis = new GZIPInputStream(is);

byte buf[] = new byte[1024];

gis.read(buf);

Log.e("zhuo", new String(buf));

connection.disconnect();
```

以上是一个简单到网络请求，而如果使用proxy的话会产生异常，我们借助异常堆栈可以较清晰的看到函数调用流程。（至于为什么产生这个异常，最后分析，而有经验的可能看下异常已经知道了）

```java
W/System.err: java.net.ConnectException: failed to connect to localhost/127.0.0.1 (port 8081) after 10000ms: isConnected failed: ECONNREFUSED (Connection refused)
W/System.err:     at libcore.io.IoBridge.isConnected(IoBridge.java:223)
at libcore.io.IoBridge.connectErrno(IoBridge.java:161)
at libcore.io.IoBridge.connect(IoBridge.java:112)
at java.net.PlainSocketImpl.connect(PlainSocketImpl.java:192)
at java.net.PlainSocketImpl.connect(PlainSocketImpl.java:459)
at java.net.Socket.connect(Socket.java:843)
at com.android.okhttp.internal.Platform.connectSocket(Platform.java:131)
at com.android.okhttp.Connection.connect(Connection.java:101)
at com.android.okhttp.internal.http.HttpEngine.connect(HttpEngine.java:294)
at com.android.okhttp.internal.http.HttpEngine.sendSocketRequest(HttpEngine.java:255)
at com.android.okhttp.internal.http.HttpEngine.sendRequest(HttpEngine.java:206)
at com.android.okhttp.internal.http.HttpURLConnectionImpl.execute(HttpURLConnectionImpl.java:345)
at com.android.okhttp.internal.http.HttpURLConnectionImpl.getResponse(HttpURLConnectionImpl.java:296)
at com.android.okhttp.internal.http.HttpURLConnectionImpl.getInputStream(HttpURLConnectionImpl.java:179)
at com.android.okhttp.internal.http.HttpsURLConnectionImpl.getInputStream(HttpsURLConnectionImpl.java:246)
```

可以发现真正的网络请求是从`getInputStream`开始。但是为什么完全了解整个流程，我们还是简单看下Url类

```java
transient URLStreamHandler streamHandler;

//仅是包装，而streamHandler的赋值是在构造函数
public URLConnection openConnection() throws IOException {
return streamHandler.openConnection(this);
}

public URL(String spec) throws MalformedURLException {
this((URL) null, spec, null);
}

//被上面的构造函数调用
public URL(URL context, String spec, URLStreamHandler handler) throws MalformedURLException {
if (spec == null) {
throw new MalformedURLException();
}
if (handler != null) {
streamHandler = handler;
}
spec = spec.trim();
protocol = UrlUtils.getSchemePrefix(spec);
int schemeSpecificPartStart = protocol != null ? (protocol.length() + 1) : 0;
// If the context URL has a different protocol, discard it because we can't use it.
if (protocol != null && context != null && !protocol.equals(context.protocol)) {
context = null;
}
// Inherit from the context URL if it exists.
if (context != null) {
set(context.protocol, context.getHost(), context.getPort(), context.getAuthority(),
context.getUserInfo(), context.getPath(), context.getQuery(),
context.getRef());
if (streamHandler == null) {
streamHandler = context.streamHandler;
}
} else if (protocol == null) {
throw new MalformedURLException("Protocol not found: " + spec);
}
if (streamHandler == null) {
//赋值
setupStreamHandler();
if (streamHandler == null) {
throw new MalformedURLException("Unknown protocol: " + protocol);
}
}
// Parse the URL. If the handler throws any exception, throw MalformedURLException instead.
try {
streamHandler.parseURL(this, spec, schemeSpecificPartStart, spec.length());
} catch (Exception e) {
throw new MalformedURLException(e.toString());
}
}

void setupStreamHandler() {
...
else if (protocol.equals("http")) {
try {
String name = "com.android.okhttp.HttpHandler";
streamHandler = (URLStreamHandler) Class.forName(name).newInstance();
} catch (Exception e) {
throw new AssertionError(e);
}
} else if (protocol.equals("https")) {
try {
String name = "com.android.okhttp.HttpsHandler";
streamHandler = (URLStreamHandler) Class.forName(name).newInstance();
} catch (Exception e) {
throw new AssertionError(e);
}
}
...
}
```

（因为基本上所有的https相关的类都是继承自http，覆盖某些方法，整体逻辑是类似的，为了少些跳转，我们直接看http相关的即可）

根据以上分析，发现最终实现类为`com.android.okhttp.HttpsHandler`，但是在源码里搜索并没有找到这个类。找到的有`external/okhttp/android/main/java/com/squareup/okhttp/HttpHandler.java`

看一下`external/okhttp/jarjar-rules.txt`

`rule com.squareup.** com.android.@1`

所以会在编译后改包名，所以就是我们要找的`HttpHandler`，而这个模块好像是早期的okhttp，集成进来改名应该是为了防止冲突，影响应用使用更新的okhttp。

```java
public class HttpHandler extends URLStreamHandler {
@Override protected URLConnection openConnection(URL url) throws IOException {
return newOkHttpClient(null /* proxy */).open(url);
}
@Override protected URLConnection openConnection(URL url, Proxy proxy) throws IOException {
if (url == null || proxy == null) {
throw new IllegalArgumentException("url == null || proxy == null");
}
return newOkHttpClient(proxy).open(url);
}
@Override protected int getDefaultPort() {
return 80;
}
protected OkHttpClient newOkHttpClient(Proxy proxy) {
OkHttpClient client = new OkHttpClient();
client.setFollowProtocolRedirects(false);
if (proxy != null) {
client.setProxy(proxy);
}
return client;
}
}

//可以看到确实如所说的http和https的关系，所以之后的分析，不在列出https专属部分
public final class HttpsHandler extends HttpHandler {
private static final List<String> ENABLED_TRANSPORTS = Arrays.asList("http/1.1");
@Override protected int getDefaultPort() {
return 443;
}
@Override
protected OkHttpClient newOkHttpClient(Proxy proxy) {
OkHttpClient client = super.newOkHttpClient(proxy);
client.setTransports(ENABLED_TRANSPORTS);
HostnameVerifier verifier = HttpsURLConnection.getDefaultHostnameVerifier();
// Assume that the internal verifier is better than the
// default verifier.
if (!(verifier instanceof DefaultHostnameVerifier)) {
client.setHostnameVerifier(verifier);
}
return client;
}
}
```

分析发现`openConnection`调用的是`OkHttpClient`的`open`函数。
