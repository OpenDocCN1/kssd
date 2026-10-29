
```javascript
var data = Memory.alloc(1);

send(message, Memory.readByteArray(data, 1))

}

return result;

}
```

下面是保存到pcap的代码

```python
def log_pcap(pcap_file, ssl_session_id, function, src_addr, src_port,
             dst_addr, dst_port, data):
    """Writes the captured data to a pcap file.

    Args:
        pcap_file: The opened pcap file.
        ssl_session_id: The SSL session ID for the communication.
        function: The function that was intercepted ("SSL_read" or "SSL_write").
        src_addr: The source address of the logged packet.
        src_port: The source port of the logged packet.
        dst_addr: The destination address of the logged packet.
        dst_port: The destination port of the logged packet.
        data: The decrypted packet data.
    """
    t = time.time()

    if ssl_session_id not in ssl_sessions:
        ssl_sessions[ssl_session_id] = (random.randint(0, 0xFFFFFFFF),
                                        random.randint(0, 0xFFFFFFFF))
    client_sent, server_sent = ssl_sessions[ssl_session_id]

    if function == "SSL_read":
        seq, ack = (server_sent, client_sent)
    else:
        seq, ack = (client_sent, server_sent)

    for writes in (
        # PCAP record (packet) header
        ("=I", int(t)), # Timestamp seconds
        ("=I", int((t * 1000000) % 1000000)), # Timestamp microseconds
        ("=I", 40 + len(data)), # Number of octets saved
        ("=i", 40 + len(data)), # Actual length of packet
        # IPv4 header
        (">B", 0x45), # Version and Header Length
        (">B", 0), # Type of Service
        (">H", 40 + len(data)), # Total Length
        (">H", 0), # Identification
        (">H", 0x4000), # Flags and Fragment Offset
        (">B", 0xFF), # Time to Live
        (">B", 6), # Protocol
        (">H", 0), # Header Checksum
        (">I", src_addr), # Source Address
        (">I", dst_addr), # Destination Address
        # TCP header
        (">H", src_port), # Source Port
        (">H", dst_port), # Destination Port
        (">I", seq), # Sequence Number
        (">I", ack), # Acknowledgment Number
        (">H", 0x5018), # Header Length and Flags
        (">H", 0xFFFF), # Window Size
        (">H", 0), # Checksum
        (">H", 0)): # Urgent Pointer

pcap_file.write(struct.pack(writes[0], writes[1]))
pcap_file.write(data)

    if function == "SSL_read":
        server_sent += len(data)
    else:
        client_sent += len(data)

    ssl_sessions[ssl_session_id] = (client_sent, server_sent)
```

### 改造并整合：

> 我去掉了pcap的保存，直接调用脚本。把输出方式统一起来（去掉所有js的`console.log`打印。统一格式`send`到py进行输出）。其他功能都保持原有的。

## jnitrace/JNI-Frida-Hook

---

### github：

> <a href="elink@436K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6U0K9r3q4E0k6e0q4W2L8$3&6Q4x3V1k6B7L8X3W2@1M7X3q4U0k6b7`.`.">jnitrace</a>/<a href="elink@a07K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6m8M7X3g2A6P5X3g2F1i4K6u0r3d9V1&6u0i4K6u0V1c8Y4u0A6k6r3q4Q4x3X3c8t1L8$3!0C8">JNI-Frida-Hook</a>

### 功能：

> 对所有jni的函数进行hook。比如vmp中大量使用到了jni的函数来模拟java的实现。对所有的jni进行hook就可以获得一些线索。

> 这里我列了两个项目，是因为这两个我都分析了一下。jnitrace的使用和输出都非常的方便，可以打印jni函数的结构，以及所有参数和返回值，并且代码结构优美，全部用ts实现的，可以说是非常完美的hook脚本开发模式，虽然很香，但是想要整合进来并不容易。我需要的是逻辑清晰易读的js文件来方便的嵌入，并且可以简单的修改。太过复杂庞大的js不利于我整合进来，所以最终选择了简单的JNI-Frida-Hook，这个项目只是简单的hook了jni函数，打印了一下函数名，并没有详细的参数和返回值。我们可以后续再进行优化

### 实现原理：

> hook的js开发比较麻烦的问题是多文件的调用会很难处理。所以jnitrace使用了ts写脚本再生成js来解决。

> 而JNI-Frida-Hook直接使用的`require("./utils/jni_struct.js")`。然后再通过`frida-compile agent.js -o _agent.js`来将多个文件合并。

> 这里我就只讲JNI-Frida-Hook的实现了。首先需要设置hook的目标模块`library_name`以及要监控的目标函数`function_name`。

> 这里他对`android_dlopen_ext`进行hook。判断目标模块加载完成了，再进行目标函数的hook。如果不这样做，在spwan的附加的时候，就会找不到模块，因为模块还未加载。

> 遍历所有`export`符号，如果有找到设定的目标函数，就进行hook所有jni函数，并且在函数结束时，关掉所有jni函数的hook。

> 所有jni函数的hook实现就是准备所有jni函数的名称列表，然后遍历所有，然后hook的时候将`jnienv`的指针传进来，再根据jni函数名和`jnienv`的指针进行偏移，找到对应的函数地址。直接hook即可。最后`FindClass`可能比较特殊，就单独hook了。

> 这个项目的关键就是计算偏移，这里只要熟悉类对象结构的存储，再看看`jnienv`这个类的结构，就看的很明白了，贴一篇我以前写的笔记博文：<a href="elink@7c1K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3#2A6M7%4y4C8K9h3&6Y4i4K6u0W2j5$3y4Q4x3V1j5J5x3o6t1H3i4K6u0r3x3o6S2Q4x3V1j5I4x3W2)9J5c8V1y4D9j5i4y4K6f1%4c8J5N6h3y4@1N6i4u0W2i4K6u0r3">类对象的内存布局</a>

### 核心代码（简略）：

```javascript
//在要hook的模块加载完后，才调用hook代码

Interceptor.attach(Module.findExportByName(null, 'android_dlopen_ext'),{

onEnter: function(args){

// first arg is the path to the library loaded

var library_path = Memory.readCString(args[0])

//判断当前加载的模块是否是目标模块

if( library_path.includes(library_name)){

console.log("[...] Loading library : " + library_path)

library_loaded = 1

}

},

onLeave: function(args){

// if it's the library we want to hook, hooking it

if(library_loaded == 1){

console.log("[+] Loaded")

//hook目标函数

hook_jni(library_name, function_name)

library_loaded = 0

}

}

})

/*

Calculate the given funcName address from the JNIEnv pointer //计算出jni函数的地址

*/

function getJNIFunctionAdress(jnienv_addr,func_name){

//最关键的起始就是这里，根据jnienv的地址和函数名,计算出偏移，其实就是拿函数的当前索引。这个了解类对象的结构就很清楚了。

var offset = jni_struct_array.indexOf(func_name) * Process.pointerSize

// console.log("offset : 0x" + offset.toString(16))

return Memory.readPointer(jnienv_addr.add(offset))

}

// Hook all function to have an overview of the function called //hook全部jni函数

function hook_all(jnienv_addr){

jni_struct_array.forEach(function(func_name){

// Calculating the address of the function

if(!func_name.includes("reserved"))

{

var func_addr = getJNIFunctionAdress(jnienv_addr,func_name)

Interceptor.attach(func_addr,{

onEnter: function(args){

console.log("[+] Entered : " + func_name)

}

})

}

})

}
```

### 改造并整合：

> 他只针对了spawn的附加情况进行hook。我调整了下，判断是哪种附加，再进行不同方式的调用。最后统一下输出的方式。

## DroidSSLUnpinning

---

### github：

> <a href="elink@82aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6i4L8$3!0&6N6h3&6p5L8%4c8S2i4K6u0r3c8s2u0G2K9h3c8e0f1@1I4g2L8Y4m8A6L8X3&6A6L8X3M7`.">DroidSSLUnpinning</a>

### 功能：

> 主要是处理防抓包的双向验证的，客户端验证服务端的证书。这个项目可以解掉证书绑定，让中间人抓包正常运行。效果和JustTrustMe差不多。他厉害的地方在于支持各种库的解绑定。市面上大多数的绑定方式他都有处理到。下面列一下大佬的支持的库

>
> ```
> 1.SSLcontext
> 2.okhttp
> 3.webview
> 4.XUtils
> 5.httpclientandroidlib
> 6.JSSE
> 7.network_security_config (android 7.0+)
> 8.Apache Http client (support partly)
> 9.OpenSSLSocketImpl
> 10.TrustKit
> 11.Cronet
> ```

### 实现原理：

> 其实实现不难。但是关键是你要熟悉各种库的正向解绑定，知道是哪个函数来绑定的，然后将绑定函数给替换掉，直接改成空函数。所以像他这种支持这么多的，就比较厉害了。

### 核心代码（简略）：

```javascript
//代码太长。这里只简单放两种解绑定的例子
//WebView的解绑定
var WebViewClient = Java.use("android.webkit.WebViewClient");

WebViewClient.onReceivedSslError.implementation = function(webView, sslErrorHandler, sslError) {
    quiet_send("WebViewClient onReceivedSslError invoke");
    //执行proceed方法
    sslErrorHandler.proceed();
    return;
};

WebViewClient.onReceivedError.overload('android.webkit.WebView', 'int', 'java.lang.String', 'java.lang.String').implementation = function(a, b, c, d) {
    quiet_send("WebViewClient onReceivedError invoked");
    return;
};

WebViewClient.onReceivedError.overload('android.webkit.WebView', 'android.webkit.WebResourceRequest', 'android.webkit.WebResourceError').implementation = function() {
    quiet_send("WebViewClient onReceivedError invoked");
    return;
};

//okhttp的解绑定
var OkHttpClient = Java.use("com.squareup.okhttp.OkHttpClient");

OkHttpClient.setCertificatePinner.implementation = function(certificatePinner) {
    // do nothing
    quiet_send("OkHttpClient.setCertificatePinner Called!");
    return this;
};

// Invalidate the certificate pinnet checks (if "setCertificatePinner" was called before the previous invalidation)
var CertificatePinner = Java.use("com.squareup.okhttp.CertificatePinner");

CertificatePinner.check.overload('java.lang.String', '[Ljava.security.cert.Certificate;').implementation = function(p0, p1) {
    // do nothing
    quiet_send("okhttp Called! [Certificate]");
    return;
};

CertificatePinner.check.overload('java.lang.String', 'java.util.List').implementation = function(p0, p1) {
    // do nothing
    quiet_send("okhttp Called! [List]");
    return;
};
```

## stalker

---

### github：

> <a href="elink@148K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6T1L8h3q4^5x3e0t1I4i4K6u0r3M7$3E0@1M7X3q4U0k6b7`.`.">sktrace</a>

### 功能：
