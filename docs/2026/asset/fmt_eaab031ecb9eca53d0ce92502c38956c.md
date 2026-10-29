
### 批量hook

```javascript
// 批量hook
function traceClass(clsname) {
try {
var target = Java.use(clsname);
// 获取本类所有函数（注意getMethods这个是获取本类和父类中的函数。）
var methods = target.class.getDeclaredMethods();
methods.forEach(function (method) {
var methodName = method.getName();
// 获取所有重载
var overloads = target[methodName].overloads;
overloads.forEach(function (overload) {
// 参数的类型
var proto = "(";
overload.argumentTypes.forEach(function (type) {
proto += type.className + ", ";
});
if (proto.length > 1) {
proto = proto.substr(0, proto.length - 2);
}
proto += ")";
log("hooking: " + clsname + "." + methodName + proto);
// hook 函数
overload.implementation = function () {
var args = [];
var tid = getTid();
var tName = getTName();
for (var j = 0; j < arguments.length; j++) {
args[j] = arguments[j] + ""
}
// 函数进入时的参数啥的在里面通过send传给py
enter(tid, tName, clsname, methodName + proto, args);
var retval = this[methodName].apply(this, arguments);
// 函数结束时的返回值在里面通过send传给py
exit(tid, "" + retval);
return retval;
}
});
});
} catch (e) {
log("'" + clsname + "' hook fail: " + e)
}
}
//匹配符合要求的类。并且不在黑名单中的
if (Java.available) {
Java.perform(function () {
log('ZenTracer Start...');
//在js被读取时，会替换这里的数据
var matchRegEx = {MATCHREGEX};
var blackRegEx = {BLACKREGEX};
Java.enumerateLoadedClasses({
onMatch: function (aClass) {
for (var index in matchRegEx) {
// console.log(matchRegEx[index]);
if (match(matchRegEx[index], aClass)) {
var is_black = false;
for (var i in blackRegEx) {
if (match(blackRegEx[i], aClass)) {
is_black = true;
log(aClass + "' black by '" + blackRegEx[i] + "'");
break;
}
}
if (is_black) {
break;
}
log(aClass + "' match by '" + matchRegEx[index] + "'");
traceClass(aClass);
}
}
},
onComplete: function () {
log("Complete.");
}
});
});
}
```

### 改造并整合

> 优化界面显示，优化类名的输入环节。每次附加进程后。都将所有类名都保存下来。这里就可以选择之前缓存下来的所有类数据。然后根据输入智能过滤。可以快捷方便的找到自己想要hook的类。快捷添加操作。将经常要hook的类放在里面。就可以迅速的hook了。

### 相关贴图

![](https://bbs.kanxue.com/upload/attach/202106/659397_455YX7TP9Q7A6MP.png)

## r0capture

---

### github

> [r0capture](https://bbs.kanxue.com/elink@352K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6J5x3s2W2K6N6h3g2Q4x3V1k6J5x3r3y4S2M7s2c8#2M7X3f1`.)

### 功能

> 安卓应用抓包的通杀脚本。并且可以过证书校验解绑定ssl pining。可以导出客户端ssl证书。可以导出pcap文件

### 实现原理

> 通过分析http https tcp udp ssl的系统框架或者第三方框架的调用流程。找到底层调用的地方进行hook。就可以在一定程度上通杀了。关于调用流程的分析的详细过程可以看看我之前整理的另一篇文章[android抓包学习的整理和归纳](https://bbs.pediy.com/thread-267940.htm)。
>
> 证书的导出是选择一个证书会调用的时机，函数`getPrivateKey`和函数`getCertificateChain`。hook后。取出私钥和证书内容。重新设置密码导出新的证书。
>
> 定位sslpinning证书绑定的位置，是通过hook类型`File`的构造函数，并且打印出调用堆栈。然后在里面匹配证书绑定函数是否在调用链。这是一种技巧。在其他场合同样可以使用类似的技巧来找到关键代码的位置。
>
> 可以将抓包结果保存为pcap数据。便捷于一些擅长使用网卡抓包工具的人导入分析（例如wireshark）。这个关键是需要熟悉网络数据包的组成结构，然后按照格式写入文件。可以参考他的这个顺便学习一下数据包的组成。

### 核心代码（简略，网络抓包相关的就不贴了，太多了）

```javascript
// 导出证书到指定路径,并使用新密码
function storeP12(pri, p7, p12Path, p12Password) {
var X509Certificate = Java.use("java.security.cert.X509Certificate")
var p7X509 = Java.cast(p7, X509Certificate);
var chain = Java.array("java.security.cert.X509Certificate", [p7X509])
var ks = Java.use("java.security.KeyStore").getInstance("PKCS12", "BC");
ks.load(null, null);
ks.setKeyEntry("client", pri, Java.use('java.lang.String').$new(p12Password).toCharArray(), chain);
try {
var out = Java.use("java.io.FileOutputStream").$new(p12Path);
ks.store(out, Java.use('java.lang.String').$new(p12Password).toCharArray())
} catch (exp) {
console.log(exp)
}
}
// 在服务器校验客户端的情形下，帮助dump客户端证书，并保存为p12的格式，证书密码为r0ysue
Java.use("java.security.KeyStore$PrivateKeyEntry").getPrivateKey.implementation = function () {
var result = this.getPrivateKey()
var packageName = Java.use("android.app.ActivityThread").currentApplication().getApplicationContext().getPackageName();
storeP12(this.getPrivateKey(), this.getCertificate(), '/sdcard/Download/' + packageName + uuid(10, 16) + '.p12', 'r0ysue');
var message = {};
message["function"] = "dumpClinetCertificate=>" + '/sdcard/Download/' + packageName + uuid(10, 16) + '.p12' + '   pwd: r0ysue';
message["stack"] = Java.use("android.util.Log").getStackTraceString(Java.use("java.lang.Throwable").$new());
var data = Memory.alloc(1);
send(message, Memory.readByteArray(data, 1))
return result;
}

// SSLpinning helper 帮助定位证书绑定的关键代码
Java.use("java.io.File").$init.overload('java.io.File', 'java.lang.String').implementation = function (file, cert) {
var result = this.$init(file, cert)
// 打印堆栈
var stack = Java.use("android.util.Log").getStackTraceString(Java.use("java.lang.Throwable").$new());
// 匹配证书绑定的函数是否在调用链中
if (file.getPath().indexOf("cacert") >= 0 && stack.indexOf("X509TrustManagerExtensions.checkServerTrusted") >= 0) {
var message = {};
message["function"] = "SSLpinning position locator => " + file.getPath() + " " + cert;
message["stack"] = stack;
var data = Memory.alloc(1);
send(message, Memory.readByteArray(data, 1))
}
return result;
}
```

下面是保存到pcap的代码
