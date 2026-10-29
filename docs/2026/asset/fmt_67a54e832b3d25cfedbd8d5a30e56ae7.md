
首先将目标`so`拷贝出来，新应用的包名需与原来的一致，并配置好`so`所需环境，在`application`中调用四哥的`PMS`签名爆破。对于一般的项目来说，绝大多数的`so`是能调用成功的。

这里简单提供一下目标签名的获取：

|  |  |
| --- | --- |
| 1–22 | **Frida：**<br><br>```javascript<br>function call_signature(){<br>    Java.perform(function(){<br>        var context = Java.use("android.app.ActivityThread").currentApplication().getApplicationContext();<br>        var packageInfo = context.getPackageManager().getPackageInfo(context.getPackageName(), 64);<br>        // .signatures[0]<br>        var Signatures = packageInfo.signatures.value;<br>        console.log("Signatures: ", Signatures.length, Signatures[0].toCharsString())<br>    })<br>}<br>```<br><br>**Xposed：**<br><br>大多数情况下`xposed hook`提示`class not find`的原因是`classloader`不对<br><br>```java<br>XposedHelpers.findAndHookMethod("com.kwai.hotfix.loader.app.TinkerApplication", loadPackageParam.classLoader,<br>    "attachBaseContext", Context.class, new XC_MethodHook() {<br>    @Override<br>    protected void beforeHookedMethod(MethodHookParam param) throws Throwable {<br>        Context context = (Context) param.args[0];<br>        String signature = context.getPackageManager().getPackageInfo(context.getPackageName(), PackageManager.GET_SIGNATURES).signatures[0].toCharsString();<br>        LogUtil.d(TAG, "PackageName" + context.getPackageName() + "   signature: " + signature);<br>        super.beforeHookedMethod(param);<br>    }<br>});<br>``` |

但是这里我们调用返回的却是`null`。这说明了一个问题，我们的环境相对于`so`原本的环境，必然有某些地方不一致，`so`仍能校验出来。且不同于某些`0335`之类的图片签名，也不同于某些在`assets`目录中读取文件的`so`，我们的`demo`并未发生崩溃，也没有其他异常信息。我们已经加好了签名爆破，那么就有可能是`apk`的完整性问题了。

### 3.2 分析问题

上文猜测有可能是`apk`的完整性保护问题。那么我们应该如何绕过呢？

我们知道一个`release`版本的`apk`是需要签名才能打包的，并且会在`META-INF`文件下生成三个签名文件：`MANIFEST.MF`、`CERT.SF`、`CERT.RSA`。其中`CERT.RSA`文件以签名的方式“包含”了前两个文件，并且也会将公钥证书一同写进去。（名称解释：`RSA`算法为非对称加密，通常来说，以公钥加密、私钥解密称为加解密过程；以私钥加密、公钥解密称为签名、验证的过程。）

而里面包含的公钥证书，我们可以通过`openssl`命令提取：

```bash
openssl pkcs7 -inform DER -in CERT.RSA -print_certs | openssl x509 -outform DER -out CERT.cer
```

提取出来的`CERT.cer`证书文件，在`010`中打开，可以发现其`hex`值正是在`Android`中获取的签名值（`toCharsString()`）。

当然，我们并不确定`so`中是否真的去校验这个签名值，更简单的方式可能直接校验`CERT.RSA`的`MD5`，而不用去提取证书。但是有一点我们可以确定，`so`中一定会通过某种方式获取到原始`apk`文件。

### 3.3 getPackageCodePath

一个`apk`在安装成功后，会在`/data/app/包名`下存有原始`apk`的信息，且这个路径是动态变化的。一般获取`apk`的完整路径，都会使用`context.getPackageCodePath()`这个方法来获取。因此，我们的目标就变成了如何`hook`掉`getPackageCodePath`，并转移其指向到原本的`apk`。

当然，比较直接的方式就是通过`Frida`或者`Xposed`进行`hook`，但是使用这两者都需要额外的成本，尤其是如果搭建模拟器服务时，会更加复杂。不过`getPackageCodePath`方法既然是在`Java`层，我们便可以利用`Java`厉害的“武器”——反射，来手动`hook`掉它。

## 四、源码分析

`Android`中`context`的实现类是`ContextImpl`，因此`context`通过`ContextWrapper`调用到`ContextImpl`的`getPackageCodePath()`方法：

```java
// ContextImpl.java
@Override
public String getPackageCodePath() {
    if (mPackageInfo != null) {
        return mPackageInfo.getAppDir();
    }
    throw new RuntimeException("Not supported in system context");
}
```

这里的`mPackageInfo`的类型是`LoadedApk`，`getAppDir()`返回的是其字段`mAppDir`。也就是说，如果我们能在运行时，动态替换掉`LoadedApk`，并把`mAppDir`字段设置为虚假的`apk`路径，那么是否就能够绕过`apk`的完整性验证？

这里我们可以参考插件化的思想，应用启动时先加载代理`application`，在创建代理`application`的过程中，替换为目标`application`，并修改其值。因此我们需要了解一个`application`是如何创建的。

### 4.1 Application 的创建过程

首先，这张图很好的概括了，当一个应用启动后，`Launcher`进程首先会通过`binder IPC`的方式将请求转发到远端服务`AMS`中（`system_server`进程），接着又会通过`socket`，在`Zygote`进程中`fork`出我们的应用进程，也就是所谓所有的应用程序都是由`zygote`孵化而来，最终会通过反射调用到`ActivityThread.main()`方法。

![](https://bbs.kanxue.com/upload/attach/202107/804803_EB93SMJD83JQ7M9.png)（[图来源](https://bbs.kanxue.com/elink@840K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3N6A6N6s2W2#2j5h3&6Q4x3X3g2U0L8$3#2Q4x3V1j5J5x3o6p5$3i4K6u0r3x3o6y4Q4x3V1j5J5y4W2)9J5c8X3q4H3M7q4)9J5k6s2m8J5L8$3y4W2M7%4y4Q4x3X3c8U0M7X3g2S2N6r3g2Q4x3V1j5`.)）

因此，我们从`ActivityThread.main()`方法开始，作为分析`application`创建的入口点：

```java
// android.app.ActivityThread
public static void main(String[] args) {
    ......
    thread.attach(false);
}

private void attach(boolean system) {
    ......
    if (!system) {
        final IActivityManager mgr = ActivityManager.getService();
        try {
            mgr.attachApplication(mAppThread);
        } catch (RemoteException ex) {
            throw ex.rethrowFromSystemServer();
        }
    }
}
```

在`attach`方法中，`getService()`最终会通过`binder`驱动在`server_manager`中获取已注册好的远端服务`AMS`的对象，并调用其（`AMS`）`attachApplication`方法。

```java
// ActivityManagerService.java
@Override
public final void attachApplication(IApplicationThread thread) {
    synchronized (this) {
        int callingPid = Binder.getCallingPid();
        final long origId = Binder.clearCallingIdentity();
        attachApplicationLocked(thread, callingPid);
        Binder.restoreCallingIdentity(origId);
    }
}

private final boolean attachApplicationLocked(IApplicationThread thread, int pid) {
    ......
    if (app.instr != null) {
        thread.bindApplication(processName, appInfo, providers,
                app.instr.mClass,
                profilerInfo, app.instr.mArguments,
                app.instr.mWatcher,
                app.instr.mUiAutomationConnection, testMode,
                mBinderTransactionTrackingEnabled, enableTrackAllocation,
                isRestrictedBackupMode || !normalMode, app.persistent,
                new Configuration(getGlobalConfiguration()), app.compat,
                getCommonServicesLocked(app.isolated),
                mCoreSettingsObserver.getCoreSettingsLocked(),
                buildSerial);
    }
}
```

调用到`bindApplication`方法中：

![](https://bbs.kanxue.com/upload/attach/202107/804803_EH62HYB7BE97K92.png)

这个方法会创建一个`AppBindData`对象，并通过`handler`的方式发送出去，这个消息将会在**主线程**的`handleMessage`里面接收，此时`switch`的分支是`BIND_APPLICATION`，接着就走到`handleBindApplication`并把新创建的`AppBindData`对象传进去。

![](https://bbs.kanxue.com/upload/attach/202107/804803_S9SE7URCJDUHZWE.png)

在`handleBindApplication`中做了最关键的事情就是`makeApplication`方法，这个方法是在`data.info`也就是`LoadedApk`中：

```java
private void handleBindApplication(AppBindData data) {
    ......
    data.info = getPackageInfoNoCheck(data.appInfo, data.compatInfo);
    ......
    Application app;
    try {
        app = data.info.makeApplication(data.restrictedBackupMode, null);
        mInitialApplication = app;
    }
    ......
}
```

![](https://bbs.kanxue.com/upload/attach/202107/804803_GETKV72VUPGXZFR.png)

`makeApplication`这个方法非常关键，它代表了一个`application`是如何创建的。

1. 首先判断当前进程的`application`是否为空，也就是判断当前进程的`application`是否已经被加载过；如果加载过，则直接返回。
2. 然后会获取此`application`的全限定类名，获取`classloader`，通过第 2 处创建出`application`。
3. 最后把创建出来的`application`添加进集合，并返回创建出的`application`，也就是第 3 处。

来看一下第 2 处的`newApplication`中做了什么？

![](https://bbs.kanxue.com/upload/attach/202107/804803_NKV5HA3QYX9QQPK.png)

![](https://bbs.kanxue.com/upload/attach/202107/804803_CSSDFBWVFC8VFFY.png)

发现最终会调用到`attachBaseContext`方法，也就是通常在应用层面，继承`Application`复写的那个方法，所以`attachBaseContext`被调用的时机其实还是很早的，此时第 2、3 处的代码还没有执行到。**因此我们可以在代理`application`中复写`attachBaseContext`，在此方法内通过反射，重新调用一次`makeApplication`，并把新的`application`替换掉原有的`application`，并完成相应属性的替换工作。**

### 4.2 hook application

这里我们会通过大量的反射获取成员对象、调用方法，因此下边的表格表示了我们需要反射获取的全部属性\方法，如有忘记可以上下查看。

| 重要字段/方法 | 所属类 | 描述 |
| --- | --- | --- |
| `currentActivityThread()` | `android.app.ActivityThread` | 该方法返回一个全局的`ActivityThread`对象 |
| `mBoundApplication` | `android.app.ActivityThread` | 类型为`ActivityThread`的静态内部类`AppBindData`，包含着存有`apk`类的信息、`LoadedApk`、`ApplicationInfo`等 |
| `info` | `ActivityThread$AppBindData` | 类型为`LoadedApk`，表示一个`apk`在当前内存的信息 |
| `mApplication` | `android.app.LoadedApk` | 类型为`Application`，表示当前进程的`Application`，在`makeApplication()`过程中，最先判断此值是否为空 |
| `mInitialApplication` | `android.app.ActivityThread` | `Application`，`makeApplication()`的返回结果会赋给此值 |
| `mAllApplications` | `android.app.ActivityThread` | `List<Application>`，`makeApplication()`过程中，将新创建的`application`保存到集合中 |
| `mApplicationInfo` | `android.app.LoadedApk` | `ApplicationInfo`，封装了一个应用的基本信息 |
| `appInfo` | `ActivityThread$AppBindData` | `ApplicationInfo`，同上 |

上节提到，我们需要做的就是重新调用`makeApplication`创建出一个新的`application`，来看核心代码：
