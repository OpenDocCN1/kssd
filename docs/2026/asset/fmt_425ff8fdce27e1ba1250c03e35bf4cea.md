
- `libopenjdk.so` 与文件重定向、文件状态有关
- `libopenjdkjvm.so` 与文件访问有关
- `libandroid_runtime.so` 与文件访问有关
- `libcutils.so` 与 `SystemProperties` 访问有关

3. 如果有遗漏的库可以调用 `NativeHook.relinkLibrary()` 重新重定位该库

## 其它模块调用

查看 <a href="elink@d66K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K6j5h3&6X3k6h3&6Y4b7h3&6V1M7X3!0A6k6q4)9J5c8V1k6S2K9$3g2j5M7r3!0K6k6h3b7.">FakeXposed</a> 说明文档

## 软件使用截图

- 软件状态 ![home](
- 应用配置，长按开启/关闭 ![package_configuration](upload/attach/202102/573677_BVAEJ5GHGRA4SQY.jpg)
- 应用配置，长按开启/关闭 ![package_configuration](
- 应用配置，长按开启/关闭 ![package_configuration](upload/attach/202102/573677_FRGK6CF6W93D46N.jpg)
- 对应功能配置 ![package_hidden](
- 应用配置，长按开启/关闭 ![package_configuration](upload/attach/202102/573677_TYVM3WRJ8TRHFVV.jpg) ![dlsym_hidden](
- 应用配置，长按开启/关闭 ![package_configuration](upload/attach/202102/573677_R9SDYC78B32RTJQ.jpg)

## FakeXposed

一个通用多功能的 `Xposed` 隐藏器，采用 `Native` 与 `Java` 结合来做到双向屏蔽检测，提供高度自由化为每个应用配置不同属性。它不仅仅局限于屏蔽 `Xposed` 检测，还提供更多更加高级的功能，如 `maps` 文件自定义屏蔽各种检测、完整的文件重定向功能、访问权限控制、`JNI` 方法监控、动态符号查找屏蔽 `dlsym` 等等，还可以提供给其它模块在进程内动态添加或修改配置。开源地址 [910K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K6j5h3&6X3k6h3&6Y4b7h3&6V1M7X3!0A6k6q4)9J5c8V1k6S2K9$3g2j5M7r3!0K6k6h3b7.](<https://bbs.kanxue.com/elink@c99K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K6j5h3&6X3k6h3&6Y4b7h3&6V1M7X3!0A6k6q4)9J5c8V1k6S2K9$3g2j5M7r3!0K6k6h3b7>). 目前问题还较多不够稳定，欢迎加群讨论

## 原理简要介绍

- `Native Hook` 使用我的另一开源项目 [fake-linker](<https://bbs.kanxue.com/elink@cd3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K6j5h3&6X3k6h3&6Y4b7h3&6V1M7X3!0A6k6q4)9J5c8X3k6S2K9$3g2Q4x3X3c8D9K9h3&6C8k6i4t1>)，`Java Hook` 使用 `Xposed` 框架，大部分功能都是由 `Native Hook` 来完成，`Xposed` 不限于原版 `Xposed`、`EdXposed`、`VirtualXposed`等等

- 内部提供 `类`、`堆栈类`、`应用`、`环境变量`、`全局系统属性`、`Android Global属性`、`Runtime.exec拦截`、`文件访问/重定向等`、`符号拦截`，各种属性的隐藏和修改，下面我将简单介绍一些原理，代码都是基于 Android 最新源码主分支，旧分支一些变化不是太大自行分析即可

### 类/堆栈类隐藏

- Hook `Class.forName()`、`ClassLoader.loadClass()`、`Throwable.getStackTrace()` 方法，判断隐藏类加载则抛出异常或删除该元素。目前我在测试 `EdXposed` 中只有部分情况会走该回调，可能框架处理了有关部分

### 应用/组件隐藏

- 使用动态代理 `PackageManager`、`ActivityManager`、`ActivityTaskManager` 屏蔽常见会使用到获取其它应用属性的方法，如：`getInstalledPackages`、`getInstalledApplications`、`getRunningServices`、`getTasks` 等等。应用进程本身就是通过 `Binder` 与 `system_server` 服务进程通信，进程内只存在一个 `IBinger` 对象，因此非常适合使用动态代理，这里屏蔽掉几乎所有能够访问其它应用的方式，具体查看源码 [`HookSystemComponent`](<https://bbs.kanxue.com/elink@5cfK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K6j5h3&6X3k6h3&6Y4b7h3&6V1M7X3!0A6k6q4)9J5c8V1k6S2K9$3g2j5M7r3!0K6k6h3c8Q4x3V1k6T1L8r3!0T1i4K6u0r3L8h3q4A6L8W2)9J5c8X3q4H3M7q4)9J5c8Y4y4J5j5#2)9J5c8X3#2S2K9h3&6Q4x3V1k6B7j5i4k6S2i4K6u0r3j5$3!0E0i4K6u0r3M7$3q4F1k6X3g2F1k6$3q4F1k6s2u0G2K9h3c8Q4x3V1k6^5M7q4)9J5c8X3S2G2L8$3E0K6i4K6u0r3d9r3!0G2K9#2y4&6M7%4c8W2L8f1y4G2L8i4m8G2L8X3g2F1N6q4)9J5k6h3A6S2N6X3p5>).。

- `PackageManager` 源码在 `ActivityThread.getPackageManager`：

```java
static volatile IPackageManager sPackageManager;

public static IPackageManager getPackageManager() {
    if (sPackageManager != null) {
        return sPackageManager;
    }
    final IBinder b = ServiceManager.getService("package");
    sPackageManager = IPackageManager.Stub.asInterface(b);
    return sPackageManager;
}
```

因此只需要使用反射修改 `sPackageManager` 静态变量即可

- `ActivityManager` 源码 `AndroidO` 以上在 `ActivityManager.IActivityManagerSingleton`，`AndroidO` 以下在 `ActivityManagerNative.gDefault`，都是一个单例对象

```java
private static final Singleton<IActivityManager> IActivityManagerSingleton =
        new Singleton<IActivityManager>() {
    @Override
    protected IActivityManager create() {
        final IBinder b = ServiceManager.getService(Context.ACTIVITY_SERVICE);
        final IActivityManager am = IActivityManager.Stub.asInterface(b);
        return am;
    }
};
```

同样反射修改 `Singleton` 里面的对象即可

- `ActivityTaskManager` 是 `AndroidQ` 以上新增的一个服务，修改方法同 `ActivityManager`

### 环境变量修改

- `Java` 调用 `System.getenv()`、`System.getenv(String)` 源码如下

```java
public static java.util.Map<String, String> getenv() {
    SecurityManager sm = getSecurityManager();
    if (sm != null) {
        sm.checkPermission(new RuntimePermission("getenv.*"));
    }

    return ProcessEnvironment.getenv();
}

public static String getenv(String name) {
    if (name == null) {
        throw new NullPointerException("name == null");
    }

    return Libcore.os.getenv(name);
}
```

其最终调用两个函数 `Libcore.os.environ()`、`Libcore.os.getenv(String)`

```java
public final class Libcore {
    private Libcore() { }

    public static final Os rawOs = new Linux();
    public static volatile Os os = new BlockGuardOs(rawOs);
    ...
}

public final class Linux implements Os {
    Linux() { }

    public native String getenv(String name);
    public native String[] environ();
    ...
}
```

不同版本 `Libcore.os` 的实现对象类名不一样，但是区别很小，而 `native` 中访问到是 `libc` 导出变量 `environ`、导出函数 `getenv`，因此通过 `Native Hook` 拦截 `getenv` 函数即可拦截对应 Java `System.getenv(String)` 调用，而 `System.getenv()` 调用是直接使用 `environ` 变量，因此暂时采用 `Java Hook` 替换该 `Map` 对象，通常情况下应用是很少使用到 `System.getenv` 非系统环境变量的，一些软件检测才会使用，因此后续可能会直接修改 `environ` 变量中的值

### 全局属性 `SystemProperties` 修改

- `Java` 反射使用 `SystemProperties.get` 系列方法

```java
@NonNull
@SystemApi
public static String get(@NonNull String key, @Nullable String def) {
    if (TRACK_KEY_ACCESS) onKeyAccess(key);
    return native_get(key, def);
}
...

@FastNative
@UnsupportedAppUsage(maxTargetSdk = Build.VERSION_CODES.P)
private static native String native_get(String key, String def);
```

调用 `android_os_SystemProperties.cpp` 中的方法
