
其中首先通过`GetXposedHookInfo()`函数调用`GetEntryPointFromJniPtrSize()`函数获取`Xposed`在设置函数`hook`时保存在`entry_point_from_jni_`中的`XposedHookInfo`对象信息。

```C++
const XposedHookInfo* GetXposedHookInfo() {
    DCHECK(IsXposedHookedMethod());
    return reinterpret_cast<const XposedHookInfo*>(GetEntryPointFromJniPtrSize(sizeof(void*)));
}
const XposedHookInfo* hook_info = soa.DecodeMethod(method)->GetXposedHookInfo();
```

然后拼接参数并通过`CallStaticObjectMethodA()`函数`JNI`调用`XposedBridge.handleHookedMethod(Member method, int originalMethodId, Object additionalInfoObj, Object thisObject, Object[] args)`函数这样就再次回到`Java`层中。

此时再次回到`XposedBridge`的源码，找到对应`handleHookedMethod`函数的实现会发现：

1. 先调用了`CallBack`中的`beforeHookedMethod`函数。

```java
// call "before method" callbacks
int beforeIdx = 0;
do {
    try {
        ((XC_MethodHook) callbacksSnapshot[beforeIdx]).beforeHookedMethod(param);
    } catch (Throwable t) {
        XposedBridge.log(t);

        // reset result (ignoring what the unexpectedly exiting callback did)
        param.setResult(null);
        param.returnEarly = false;
        continue;
    }

    if (param.returnEarly) {
        // skip remaining "before" callbacks and corresponding "after" callbacks
        beforeIdx++;
        break;
    }
} while (++beforeIdx < callbacksLength);
```

2. 然后通过`invokeOriginalMethodNative`调用原函数。

```java
// call original method if not requested otherwise
if (!param.returnEarly) {
    try {
        param.setResult(invokeOriginalMethodNative(method, originalMethodId,
                additionalInfo.parameterTypes, additionalInfo.returnType, param.thisObject, param.args));
    } catch (InvocationTargetException e) {
        param.setThrowable(e.getCause());
    }
}
```

3. 最后执行所有的`afterHookedMethod`回调。

```java
// call "after method" callbacks
int afterIdx = beforeIdx - 1;
do {
    Object lastResult = param.getResult();
    Throwable lastThrowable = param.getThrowable();
    try {
        ((XC_MethodHook) callbacksSnapshot[afterIdx]).afterHookedMethod(param);
    } catch (Throwable t) {
        XposedBridge.log(t);

        // reset to last result (ignoring what the unexpectedly exiting callback did)
        if (lastThrowable == null)
            param.setResult(lastResult);
        else
            param.setThrowable(lastThrowable);
    }
} while (--afterIdx >= 0);
```

其中`invokeOriginalMethodNative`函数则是通过保存的`reflected_method`反射对象对原函数进行反射调用，也就是通过原`art`函数`InvokeMethod()`函数进行调用。

```C++
jobject XposedBridge_invokeOriginalMethodNative(JNIEnv* env, jclass, jobject javaMethod,
        jint isResolved, jobjectArray, jclass, jobject javaReceiver, jobjectArray javaArgs) {
    ScopedFastNativeObjectAccess soa(env);
    if (UNLIKELY(!isResolved)) {
        ArtMethod* artMethod = ArtMethod::FromReflectedMethod(soa, javaMethod);
        if (LIKELY(artMethod->IsXposedHookedMethod())) {
            javaMethod = artMethod->GetXposedHookInfo()->reflected_method;
        }
    }
#if PLATFORM_SDK_VERSION >= 23
    return InvokeMethod(soa, javaMethod, javaReceiver, javaArgs);
#else
    return InvokeMethod(soa, javaMethod, javaReceiver, javaArgs, true);
#endif
}
```

## 5. 参考资料

<mark class="encrypted">b3dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4N6W2K9i4S2A6L8W2)9#2k6U0b7%4z5o6R3K6y4U0x3$3i4K6u0r3j5i4u0@1K9h3y4D9k6g2)9J5c8X3c8W2N6r3q4A6L8s2y4Q4x3V1j5I4x3o6V1H3x3e0R3@1y4o6l9`.</mark>

<mark class="encrypted">64aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2k6$3N6#2L8X3y4D9k6g2)9J5k6h3N6A6N6r3S2#2j5W2)9J5k6h3W2G2i4K6u0r3x3U0l9I4z5q4)9J5c8U0l9J5i4K6u0r3x3o6c8Q4x3V1k6^5M7r3!0K6k6h3c8Q4x3X3c8S2M7Y4c8Q4x3X3c8Z5L8$3!0C8i4K6u0V1i4K6t1#2c8e0k6Q4x3U0g2n7y4g2)9J5y4e0R3#2i4K6t1#2c8e0k6Q4x3U0f1&6c8g2)9J5y4e0V1H3i4K6u0r3</mark>

https://bbs.pediy.com/thread-257844.htm

目录

- Xposed ART原理分析笔记

- 0. 写在前面

- 1. Xposed结构

- 2. `Xposed`注入进程

- 3. `Xposed` `hook`函数

- 4. 被`hook`的函数执行流程分析

- 5. 参考资料

---

# Xposed ART原理分析笔记

## 0. 写在前面

这篇文章是我在看`Xposed`源码时所做的一些笔记内容，中间也参考了一些前人的分析文章这里都在最后列出了，除此之外还有一些内容纯属个人理解，如果有错误欢迎各位大佬斧正，感谢！

## 1. Xposed结构

Xposed相关源码: `9cdK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6J5L8%4k6G2z5o6V1`

1. `XposedBridge`: Xposed提供的jar文件，app_process启动过程中会加载该jar包，其他的Modules的开发都是基于该jar包。
2. `Xposed`: Xposed的C++部分，主要是用来替换`/system/bin/app_process`，并为`XposedBridge`提供JNI方法。
3. `XposedInstaller`: Xposed的安装包，提供对基于Xposed框架的Modules的管理。
4. `android_art`: Xposed修改的art部分源码。

## 2. `Xposed` 注入进程

`Xposed`注入进程的方式是通过`zygote`进程在`fork`出`APP`进程时使之加载`XposedBridge.jar`。

根据`Xposed`的`Android.mk`文件可知在5.0以上的手机上编译的是`app_main2.cpp`。

```makefile
ifeq (1,$(strip $(shell expr $(PLATFORM_SDK_VERSION) >= 21)))
# if Android_Version >= 5.0
LOCAL_SRC_FILES := app_main2.cpp
LOCAL_MULTILIB := both
LOCAL_MODULE_STEM_32 := app_process32_xposed
LOCAL_MODULE_STEM_64 := app_process64_xposed
else
LOCAL_SRC_FILES := app_main.cpp
LOCAL_MODULE_STEM := app_process_xposed
endif
```

这里选取`Android 7.1.1_r6`的`app_main.cpp`文件与`app_main2.cpp`做对比。

从`main`函数看起。

![图片描述](https://bbs.kanxue.com/upload/attach/202104/715334_2CCXG3XP6PGEFPP.png)

从文件的对比可以发现，`app_main2.cpp`中多出了对于`Xposed::handleOptions`的调用，而这个函数主要是对`--xposedversion`的处理以及是否测试模式的启用，真正运行时无需理会。

```C++
bool handleOptions(int argc, char* const argv[]) {
    // version 信息
    parseXposedProp();

    if (argc == 2 && strcmp(argv[1], "--xposedversion") == 0) {
        printf("Xposed version: %s\n", xposedVersion);
        return true;
    }

    if (argc == 2 && strcmp(argv[1], "--xposedtestsafemode") == 0) {
        printf("Testing Xposed safemode trigger\n");

        if (detectSafemodeTrigger(shouldSkipSafemodeDelay())) {
            printf("Safemode triggered\n");
        } else {
            printf("Safemode not triggered\n");
        }
        return true;
    }

    // From Lollipop coding, used to override the process name
    argBlockStart = argv[0];
    uintptr_t start = reinterpret_cast<uintptr_t>(argv[0]);
    uintptr_t end = reinterpret_cast<uintptr_t>(argv[argc - 1]);
    end += strlen(argv[argc - 1]) + 1;
    argBlockLength = end - start;

    return false;
}
```

在`main`函数的最后，增加了对`Xposed`的加载与`runtimeStart`函数的调用。

![图片描述](https://bbs.kanxue.com/upload/attach/202104/715334_P6QHKFKE8JT7K8P.png)

在`initialize`函数最终返回`XposedBridge.jar`是否成功加入环境变量的标志和`XposedInstaller`是否成功运行的标志。

首先对`xposed`这个结构体变量进行赋值。

```C++
struct XposedShared {
    // Global variables
    bool zygote;
    bool startSystemServer;
    const char* startClassName;
    uint32_t xposedVersionInt;
    bool isSELinuxEnabled;
    bool isSELinuxEnforcing;
    uid_t installer_uid;
    gid_t installer_gid;

    // Provided by runtime-specific library, used by executable
    void (*onVmCreated)(JNIEnv* env);

#if XPOSED_WITH_SELINUX
    // Provided by the executable, used by runtime-specific library
    int (*zygoteservice_accessFile)(const char* path, int mode);
    int (*zygoteservice_statFile)(const char* path, struct stat* st);
    char* (*zygoteservice_readFile)(const char* path, int* bytesRead);
#endif
};
XposedShared* xposed = new XposedShared;
xposed->zygote = zygote;
xposed->startSystemServer = startSystemServer;
xposed->startClassName = className;
xposed->xposedVersionInt = xposedVersionInt;

#if XPOSED_WITH_SELINUX
    xposed->isSELinuxEnabled = is_selinux_enabled() == 1;
    xposed->isSELinuxEnforcing = xposed->isSELinuxEnabled && security_getenforce() == 1;
#else
    xposed->isSELinuxEnabled = false;
    xposed->isSELinuxEnforcing = false;
#endif  // XPOSED_WITH_SELINUX
```

然后在`logcat`中通过`printStartupMarker`函数和`printRomInfo`函数打印一些信息。
