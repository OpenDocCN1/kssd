
# [原创]聊聊大厂设备指纹获取和对抗&设备指纹看着一篇就够了!

## 前言

现在大厂的设备指纹层出不穷，但是想要确保**稳定性**和**唯一性**高精准其实也挺难的一件事，有的是通过设备信息比重进行的设备ID唯一值确认。比如A设备信息占比10%，B设备信息占比20%，当比重超过60%以上，设备指纹才会发生变化。这样的好处就是当你只修改某一个字段的时候，设备指纹不发生变化。还有的干脆找一个隐蔽的并且唯一的设备信息，作为缓存，每次读取缓存的方式去判断，设备信息是唯一，比如常见的有Native获取DRM，`popen cat /sys/devices/soc0/serial_number`，`svc`读取`bootid`并且保存到文件，`netlinker`获取网卡。都是很常见并且隐蔽的设备指纹。这篇文章主要介绍了各种指纹的获取情况，如何修改，站在上帝视角去俯看，攻击者和被攻击者遇到的问题。

看完这篇文章主要你能学到如下：

- 1，常用的指纹检测有哪些？
- 2，如何修改设备指纹，难点在哪里，会有哪些坑需要踩？
- 3，现在的国内的指纹面临哪些问题？

## 前奏知识

### Android IPC代理是什么？

Android本身是CS架构，客户端（client）服务端（server），我们常用的通过`context`上下文调用的API都是直接调用代理人的方式去调用的，而真正的服务端是`ActivityManagerServer`，简称`AMS`，他有很多代理，比如`PackageManager`、`ActivityManager`等，这些都是`AMS`的**代理人**。而`AMS`就是**被代理人**。代理模式是一种设计模式，代理人可以提供被代理人的部分或者全部功能，实现代码封装，做鉴权，代码安全的角度，代理模式很常用的设计模式。

`AMS`和代理们通过`Binder`进行通讯，`Binder`是什么，有什么好处这里就不详细展开了，安卓面试八股文，可以理解成进程间通讯的东西，底层实现是通过共享内存，数据传输，读取速度更快。当我们调用代理人的API得时候，本质上是通过`Binder`去发送一些数据包，和`AMS`通讯，当`AMS`收到消息以后把结果再传输给对应的代理人。然后返回给调用方。在每个`Manager`里面都有一个代理人。

之前很久之前有一种动态代理的技术，原理就是替换了里面的代理人，因为代理是一个接口，然后我们自己通过`Proxy`这个类创建一个代理，然后反射`set`回去，就可以实现常用的API拦截和`Hook`。类似`VA`的沙盒，对多开的App提供一份自己实现的代理，然后控制这些代理的返回值，以此实现沙盒相关操作。还有一种比较好的过APK签名的方法就是直接`Hook`"水管"，也就是`hook` `binder`的通讯的方法，当接收到指定事件以后，直接修改具体的结果，以此对Java层进行全量`Hook`（`binder`的通讯方法被`Hook`以后，调用者和代理人只能拿到被修改以后的结果，以此实现Java层的全量`Hook`，后面再讲签名验证的时候我再详细说。）

### 如何绕过9.0隐藏API限制？

因为在获取或者分析的时候，需要绕过反射9.0限制，这里采用的是Lsp作者的[`AndroidHiddenApiBypass`](https://bbs.kanxue.com/elink@173K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6x3f1#2m8G2M7$3g2V1i4K6u0r3b7h3&6V1M7X3!0A6k6p5S2A6k6r3c8W2L8V1q4H3K9f1u0&6M7r3q4K6M7H3`.`.)进行隐藏API的调用。

在手机装了`EDXP`或者`Magisk`，之后一般隐藏API的限制都是默认去除的。因为在`EDXP`和`Magisk`里面也需要进行使用一些隐藏API。

## 设备指纹

设备指纹主要分为三部分，Java层设备指纹，Native设备指纹，`popen`执行一些命令获取设备信息，包括一些核心的设备指纹。

这篇文章也主要围绕这三部分进行展开讨论。比如一些内核文件等信息，我也会放在Native层进行讨论。

所有的每个设备指纹我都分为两部分

`Get`（如何获取，站在开发者角度） `Mock`（如何进行修改测试，站在攻击者角度）

所有的指纹我都分为三种类型，**普通指纹，次要指纹，重要指纹。**

### Java层设备指纹

#### Setting相关（重要）

##### Get

在setting里面大家经常遇到的可能就是`android id`的获取的。API如下：

```
Settings.Secure.getString(context.getContentResolver(), Settings.Secure.ANDROID_ID)
```

但是其实Settings里面还有很多别的功能东西，常见的就是`Settings.Secure`和`Settings.Global`。

在`Settings.Global`里面其实还有一些别的字段，具体API如下。这些都是一些比较隐蔽的设备指纹。

```
Settings.Global.getString(context.getContentResolver(), "mi_health_id")
Settings.Global.getString(context.getContentResolver(), "mi_health_id")
Settings.Global.getString(context.getContentResolver(), "gcbooster_uuid")
Settings.Global.getString(context.getContentResolver(), "key_mqs_uuid")
Settings.Global.getString(context.getContentResolver(), "ad_aaid")
```

##### Mock

- 方法`Hook`

`Global`和`Secure`都是实现的`NameValueTable`接口。

```
public static String getString(ContentResolver resolver, String name) {
    return getStringForUser(resolver, name, resolver.getUserId());
}
```

底层调用的是`getStringForUser(resolver, name, resolver.getUserId())`三个参数，如果`Hook`的话可以对这个方法进行入手。

`Settings.Secure->getStringForUser` & `Settings.Global->getStringForUser`

- 内存反射

很多开发者会采用内存反射的方式去获取变量，所以仅仅是通过`Mock`方法的方式不够。如果进行`Mock`，需要将`Settings.Secure`和`Settings.Global`里面的内存变量进行修复：`Settings.Global`放了一些全局变量，`Settings.Secure`放了一些安全相关变量。`Settings.Secure->getStringForUser`和`Settings.Global->getStringForUser`的具体方法如下：

```
private static final HashSet<String> MOVED_TO_GLOBAL;

private static final NameValueCache sNameValueCache = new NameValueCache(
    CONTENT_URI,
    CALL_METHOD_GET_SYSTEM,
    CALL_METHOD_PUT_SYSTEM,
    sProviderHolder,
    System.class);

public static String getStringForUser(ContentResolver resolver, String name,
        int userHandle) {
    if (MOVED_TO_GLOBAL.contains(name)) {
        Log.w(TAG, "Setting " + name + " has moved from android.provider.Settings.Secure"
                + " to android.provider.Settings.Global.");
        return Global.getStringForUser(resolver, name, userHandle);
    }
    ...
    return sNameValueCache.getStringForUser(resolver, name, userHandle);
}

@UnsupportedAppUsage
public static String getStringForUser(ContentResolver resolver, String name,
        int userHandle) {
    if (MOVED_TO_SECURE.contains(name)) {
        Log.w(TAG, "Setting " + name + " has moved from android.provider.Settings.System"
                + " to android.provider.Settings.Secure, returning read-only value.");
        return Secure.getStringForUser(resolver, name, userHandle);
    }
    if (MOVED_TO_GLOBAL.contains(name) || MOVED_TO_SECURE_THEN_GLOBAL.contains(name)) {
        Log.w(TAG, "Setting " + name + " has moved from android.provider.Settings.System"
                + " to android.provider.Settings.Global, returning read-only value.");
        return Global.getStringForUser(resolver, name, userHandle);
    }
}
```

可以看到，整体的cache都是放在`sNameValueCache`变量和`MOVED_TO_GLOBAL`变量内部进行存储。

我们可以直接反射`MOVED_TO_GLOBAL`这个`HashSet`或者去`sNameValueCache`这个变量然后去获取这个值的话，也是很容易可以拿到最真实的值的。所以光`mock`是不够的。

比如很多大厂就是**Android高版本绕过了反射限制以后，或者判断当前手机没有API反射限制以后直接通过反射变量的方式去获取。**

`sNameValueCache`在高版本是一个对象，低版本安卓中它是一个`ArrayMap`，这块需要注意。

`sNameValueCache`修改的话可以调用API的`putStringForUser`往里面强制赋值。这么一来下次对方再通过API去调用的时候就会拿到你已经进行过`Mock`的值。所以你修改的时候需要进行判断，当前获取的值是否是你已经`Mock`过的。

#### 蓝牙网卡MAC（普通）

蓝牙的网卡不是普通的网卡，后面会介绍`netlinker`获取真实的网卡。

##### Get

主要方法就是通过`BluetoothAdapter->getAddress`。

```
public String getAddress() {
    try {
        return mManagerService.getAddress(mAttributionSource);
    } catch (RemoteException e) {
        Log.e(TAG, "", e);
    }
    return null;
}
```

##### Mock

可以看到这个方法主要是通过`IPC`的代理类方式去获取的。所以`Hook`的话尽可能先`Hook`代理的`IPC`类。先尝试反射`android.bluetooth.IBluetooth$Stub$Proxy`，然后`Hook` `IPC`里面的`getAddress`，而不是直接`Hook BluetoothAdapter->getAddress`。

因为很多大厂获取设备的指纹的时候会检测这个方法是否被`Hook`，检测也很简单，只需要获取这个`artmethod`结构体以后判断这个方法入口是否被替换。比如`Sandhook`之类的常用的`Hook`框架，低版本采用的是`inlinehook`形式，在高版本里面采用的是入口替换，可以直接获取到方法的入口的函数地址，判断一下函数所在的`so`即可。所以尽可能`Hook` `IPC`的方法。如果用`XPosed`去修改的话，还需要注意魔改，否则大厂会通过`XposedHelpers->sHookedMethodCallbacks`变量把你`Hook`的方法进行上报。

> 小技巧：这个变量是一个静态变量，所以我们只需要拿到`XposedHelpers`这个`class`即可。想要拿到`class`必须先拿到这个类的`classloader`，正常的`Xposed`是通过系统的`classloader`作为父类`classloader`，但是`edxp`这种，是一个方法内部的成员变量，没有任何地方引用这个`classloader`，所以想拿到这个`classloader`需要用到内存漫游。把内存全部的`classloader`都从内存抠出来，然后挨个去反射获取`XposedHelpers`即可。代码可参考如下：

> <https://bbs.pediy.com/thread-269094.htm>

`sHookedMethodCallbacks`里面保存了`XPosed`全部的`Hook`方法信息，用于石锤当前方法是否被`Hook`。获取被`Hook`方法具体如下：

`XposedHelpers->methodCache`不建议使用，如果攻击者使用了`XposedBridge->HookAllmethod`的话，可能会导致`Hook`方法上报的遗漏。

```
private void getHookItemDemo() {
    ArrayList<Object> choose = ChooseUtils.choose(ClassLoader.class, true);
    for (Object obj : choose) {
        ClassLoader clazzloader = (ClassLoader) obj;
        Class<?> clazzXh = null;
        Class<?> clazzBR = null;
        try {
            Log.e("Test", "-----------------------------------------------------------------");
            clazzBR = Class.forName("de.robv.android.xposed.XposedBridge", false, clazzloader);
            Field callbacksField = clazzBR.getDeclaredField("sHookedMethodCallbacks");
            callbacksField.setAccessible(true);
            Map<Member, Object> callback = (Map<Member, Object>) callbacksField.get(null);
            for (Member key : callback.keySet()) {
                Log.e("Test", "sHookedMethodCallbacks  " + key.toString());
            }
            Log.e("Test", "-----------------------------------------------------------------");
            return;
        } catch (Throwable e) {
            Log.e("Test", "find errror " + e.getMessage());
        }
    }
}
```

#### serial（普通）

这个变量在高版本里面基本已经拿不到，即使拿到了也是一个`unknow`，但是也需要兼容低版本的Android。

##### get

```
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
    return Build.getSerial();
}
```

##### Mock
