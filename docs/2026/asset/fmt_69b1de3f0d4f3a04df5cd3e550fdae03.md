
```
static anti_fgets() {
    const tag = 'anti_fgets';
    const fgetsPtr = Module.findExportByName(null, 'fgets');
    DMLog.i(Anti.tag, 'fgets addr: ' + fgetsPtr);
    if (null == fgetsPtr) {
        return;
    }
    var fgets = new NativeFunction(fgetsPtr, 'pointer', ['pointer', 'int', 'pointer']);
    Interceptor.replace(fgetsPtr, new NativeCallback(function (buffer, size, fp) {
        if (null == this) {
            return 0;
        }
        var logTag = null;
        // 进入时先记录现场
        const lr = FCCommon.getLR(this.context);
        // 读取原 buffer
        var retval = fgets(buffer, size, fp);
        var bufstr = (buffer as NativePointer).readCString();

        if (null != bufstr) {
            if (bufstr.indexOf("TracerPid:") > -1) {
                buffer.writeUtf8String("TracerPid:\t0");
                logTag = 'TracerPid';
            }
            //State:    S (sleeping)
            else if (bufstr.indexOf("State:\tt (tracing stop)") > -1) {
                buffer.writeUtf8String("State:\tS (sleeping)");
                logTag = 'State';
            }
            // ptrace_stop
            else if (bufstr.indexOf("ptrace_stop") > -1) {
                buffer.writeUtf8String("sys_epoll_wait");
                logTag = 'ptrace_stop';
            }
            else if (bufstr.indexOf(") t") > -1) {
                buffer.writeUtf8String(bufstr.replace(") t", ") S"));
                logTag = 'stat_t';
            }
            // SigBlk
            else if (bufstr.indexOf('SigBlk:') > -1) {
                buffer.writeUtf8String('SigBlk:\t0000000000001204');
                logTag = 'SigBlk';
            }
            if (logTag) {
                DMLog.i(tag + " " + logTag, bufstr + " -> " + buffer.readCString() + ' lr: ' + lr + "(" + FCCommon.getModuleByAddr(lr) + ")");
            }
        }
        return retval;
    }, 'pointer', ['pointer', 'int', 'pointer']));
}
```

**FridaContainer 调用：**

```
FCAnd.anti.anti_debug();
```

**小结：**

当上面的方法无法绕过反调试时，可以再 Hook 一些常用的退出函数来定位反调点，比如 hook `exit`、`kill`，再总结出一些其他过反调的方法，思路类似。

## anti-ssl-pinning

我们也在 FridaContainer 里面集成了 Frida 版的 JustTrustMe 来过 SSL Pinning 检测。

此部分代码主要借鉴了：294K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0L8$3c8W2M7$3S2S2M7X3g2Q4x3X3g2X3M7X3W2V1j5g2)9J5k6i4u0W2i4K6u0r3i4K6b7H3j5h3E0S2j5X3f1I4i4K6u0r3k6Y4u0A6k6r3q4Q4x3X3c8E0N6h3I4@1K9i4m8D9k6g2)9J5k6s2g2F1M7r3W2F1L8X3W2F1k6#2)9J5c8R3`.`.

支持 20 种类库的SSL 验证绕过：

- `TrustManager` (Android < 7)
- `TrustManagerImpl` (Android > 7)
- OkHTTPv3 (quadruple bypass)
- Trustkit (triple bypass)
- Appcelerator Titanium
- `OpenSSLSocketImpl` Conscrypt
- `OpenSSLEngineSocketImpl` Conscrypt
- `OpenSSLSocketImpl` Apache Harmony
- PhoneGap `sslCertificateChecker`
- IBM MobileFirst `pinTrustedCertificatePublicKey` (double bypass)
- IBM WorkLight (ancestor of MobileFirst) `HostNameVerifierWithCertificatePinning` (quadruple bypass)
- Conscrypt `CertPinManager`
- CWAC-Netsecurity (unofficial back-port pinner for Android<4.2) `CertPinManager`
- Worklight Androidgap `WLCertificatePinningPlugin`
- Netty `FingerprintTrustManagerFactory`
- Squareup `CertificatePinner` [OkHTTP<v3] (double bypass)
- Squareup `OkHostnameVerifier` [OkHTTP v3] (double bypass)
- Android `WebViewClient` (double bypass)
- Apache Cordova `WebViewClient`
- Boye `AbstractVerifier`

**FridaContainer 调用：**

```
FCAnd.anti.anti_ssl_unpinning();
```

**小结：**

主要的出发点就是想摆脱 Xposed 框架，调试时减少被检测的风险，要绕过检测只绕 Frida 一个就好了。

## dump dex

这里的 dump dex 是 dump 二代壳，通过 hook art so 的 `art::DexFile::OpenCommon` (`_ZN3art7DexFile10OpenCommonEPKhjRKNSt3__112basic_stringIcNS3_11char_traitsIcEENS3_9allocatorIcEEEEjPKNS_10OatDexFileEbbPS9_PNS0_12VerifyResultE`) dump 动态加载的 dex 文件。

通过 frida 动态加载自定义的 dex，在 dex 中实现类的枚举和主动加载，来尝试加载所有的类，使其触发 dex 动态加载，再通过之前的 hook ，将加载的 dex dump 下来。

**FridaContainer 调用：**

```
FCAnd.dump_dex_common();
```

也有通过搜索内存的方式将 dex dump 出来 [FRIDA-DEXDump](https://bbs.kanxue.com/elink@971K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6Z5L8s2g2%4j5g2)9J5c8V1k6d9d9f1c8m8i4K6u0V1c8p5g2j5c8s2g2E0M7l9`.`.)

## multi-dex

有时候遇到动态加载的 dex 又不是用的 application 的 `loader`，`Java.use` 可能会提示找不到类，遇到这种情况可以用修改 java `loader` 的方式来应对。

比如遇到利用 `InMemoryDexClassLoader` 来加载内存中的 dex 时，可以 Hook 它，当其触发时先走原流程，让其动态加载 dex ，然后利用此时的 `loader` object 修改 `Java.classFactory.loader`，再使用 `Java.use` 来获取类就可以了，使用后记得恢复现场，否则会崩溃。

**动态加载实例：**

![动态加载实例](https://bbs.kanxue.com/upload/attach/202101/357319_EBAB3A4EXY7XAUT.png)

**完整代码：**

```
function anti_InMemoryDexClassLoader(callbackfunc) {
    // dalvik.system.InMemoryDexClassLoader
    const InMemoryDexClassLoader = Java.use('dalvik.system.InMemoryDexClassLoader');
    InMemoryDexClassLoader.$init.overload('java.nio.ByteBuffer', 'java.lang.ClassLoader')
        .implementation = function (buff, loader) {
            this.$init(buff, loader);
            var oldcl = Java.classFactory.loader;
            Java.classFactory.loader = this;
            callbackfunc();
            Java.classFactory.loader = oldcl;
            // 恢复现场
        }
}
```

**使用：**

```
FCAnd.anti.anti_InMemoryDexClassLoader(function(){
    const cls = Java.use('find/same/multi/dex/class');
    ...
});
```

## frida 的 java 反射调用

有时 dex 使用了一些非常规的动态加载方式，找 `loader` 定位起来也比较麻烦。假设我们只想调用某 jni 函数，看输入输出值的话，还可以用 frida java 反射的方式调用。

**整体调用流程：**

```
// 获取 so 基址
var base = Module.findBaseAddress('libxxxx.so');
// 根据偏移获取 jni 函数地址
var jnifunc_ptr = libsgmainso.add(0xE729);
// 声明 jni 函数
var jnifunc = new NativeFunction(jnifunc_ptr, 'pointer', ['pointer', 'pointer', 'int', 'pointer']);
// ********* 拼装 obj *********
// JNIEnv
var env = Java.vm.getEnv();
// 调用
var retval = jnifunc(env.handle, ptr(0), 10401, obj);
```

**拼装Obj：**

```
// staticVaMethod 实现 Integer.valueOf(7)
const Integer_jcls = env.findClass('java/lang/Integer');
const Integer_valueOf = env.getStaticMethodId(Integer_jcls, 'valueOf', '(I)Ljava/lang/Integer;');
const invorkeStaticOjbectMethod = env.staticVaMethod('pointer', ['int']);
var pIn2 = invorkeStaticOjbectMethod(env.handle, Integer_jcls, Integer_valueOf, 7);
// 利用 constructor | vaMethod 组装 HashMap obj
// new HashMap().put('INPUT', 'xxxxxxxxxx')
const HashMap_jcls = env.findClass('java/util/HashMap');
const invokeHashmap_constructor = env.constructor([]);
const HashMap_init = env.getMethodId(HashMap_jcls, '<init>', '()V');
var HashMap_obj = invokeHashmap_constructor(env.handle, HashMap_jcls, HashMap_init);
const HashMap_put = env.getMethodId(HashMap_jcls, 'put', '(Ljava/lang/Object;Ljava/lang/Object;)Ljava/lang/Object;');
const invokeOjbectMethod = env.vaMethod('pointer', ['pointer', 'pointer']);
invokeOjbectMethod(env.handle, HashMap_obj, HashMap_put, env.newStringUtf('INPUT'), env.newStringUtf('xxxxxxxxxx'));
```

上面都是在反射调用的角度简单介绍了 `staticVaMethod`, `vaMethod`, `constructor` 的使用方法。

## trace java methods

用 Frida 也可以实现 java 层的 trace 。

因为基于 Frida 框架，如果直接 trace 所有的类效率太慢，也容易崩溃。所以这里是以白名单的方式实现的。核心方法就是枚举所有类，按过滤名单，匹配需要 trace 的类，Hook 目标类的所有方法（可指定），在方法被调用时，将其入参和返回值记录下来。

**核心逻辑：**

遍历所有类，Hook 白名单中的类及方法。

**调用方式：**
