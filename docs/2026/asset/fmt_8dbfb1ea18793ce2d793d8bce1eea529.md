
```cpp
isXposedLoaded = xposed::initialize(zygote, startSystemServer, className, argc, argv);

if (zygote) {
    runtime.start(isXposedLoaded ? XPOSED_CLASS_DOTS_ZYGOTE : "com.android.internal.os.ZygoteInit",
                  startSystemServer ? "start-system-server" : "");
} else if (className) {
    // Remainder of args get passed to startup class main()
    runtime.mClassName = className;
    runtime.mArgC = argc - i;
    runtime.mArgV = argv + i;

    runtime.start(isXposedLoaded ? XPOSED_CLASS_DOTS_TOOLS : "com.android.internal.os.RuntimeInit",
                  application ? "application" : "tool");
}
```

可以看到，Xposed首先尝试加载自己的库，如果成功的话，就将`runtime.start`的类名替换成自己的类：`de.robv.android.xposed.XposedBridge`。因为`app_processs`是采用反射`main()`的方法来调用加载方法，所以只要在这个类中定义一个`main()`方法，就可以为所欲为了。最后只需在末尾调用一下`com.android.internal.os.ZygoteInit`的`main()`，让系统继续跑下去，就可以实现对Zygote的Hook。

## 注入应用进程

既然已经拿到了Zygote的所有权了，但是现在Application还没跑起来呢。如果这个时候就执行Hook模块，肯定是无法Hook到相关代码的。那么这时候就有必要提一下从Zygote接受到请求之后做了什么，我又简单的画了张图：

![](https://bbs.kanxue.com/./upload/attach/201801/715510_26xo2xnk8nnmxjf.png)

值得注意的是，因为fork之后的子进程会继承父进程的状态继续往下跑，所以到`handleChildProc`的时候，此时已经是子进程在执行程序了，而`ActivityThread`也是在新进程中loop。

在Activity的loop中，可以接收的消息也包括了Application的生命周期`BIND_APPLICATION`和`EXIT_APPLICATION`。处理`BIND_APPLICATION`的时候是直接调用了`handleBindApplication`方法，Xposed便是在Zygote中Hook了这个方法，并在这里面执行用户的Hook模块。因为在这个时候，已经可以拿到应用的`classLoader`了，所以将此`classLoader`封装成一个`LoadPackageParam`，再传给各个模块的`handleLoadPackage`即可。

在`XposedBridge`的`main`中可看到其调用了`initForZygote()`：

```java
protected static void main(String[] args) {
    ...

    if (isZygote) {
        XposedInit.hookResources();
        XposedInit.initForZygote();
    }

    ...
}
```

`initForZygote`中又Hook了`handleBindApplication`：

```java
static void initForZygote() throws Throwable {
    ...

    findAndHookMethod(ActivityThread.class, "handleBindApplication",
            "android.app.ActivityThread.AppBindData", new XC_MethodHook() {

        @Override
        protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
            ActivityThread activityThread = (ActivityThread) param.thisObject;

            ApplicationInfo appInfo =
                    (ApplicationInfo) getObjectField(param.args[0], "appInfo");

            String reportedPackageName =
                    appInfo.packageName.equals("android") ? "system" : appInfo.packageName;

            SELinuxHelper.initForProcess(reportedPackageName);

            ComponentName instrumentationName =
                    (ComponentName) getObjectField(param.args[0], "instrumentationName");

            if (instrumentationName != null) {
                Log.w(TAG, "Instrumentation detected, disabling framework for "
                        + reportedPackageName);

                XposedBridge.disableHooks = true;
                return;
            }

            CompatibilityInfo compatInfo =
                    (CompatibilityInfo) getObjectField(param.args[0], "compatInfo");

            if (appInfo.sourceDir == null)
                return;

            setObjectField(activityThread, "mBoundApplication", param.args[0]);
            loadedPackagesInProcess.add(reportedPackageName);

            LoadedApk loadedApk =
                    activityThread.getPackageInfoNoCheck(appInfo, compatInfo);

            XResources.setPackageNameForResDir(
                    appInfo.packageName, loadedApk.getResDir());

            XC_LoadPackage.LoadPackageParam lpparam =
                    new XC_LoadPackage.LoadPackageParam(
                            XposedBridge.sLoadedPackageCallbacks);

            lpparam.packageName = reportedPackageName;
            lpparam.processName =
                    (String) getObjectField(param.args[0], "processName");
            lpparam.classLoader = loadedApk.getClassLoader();
            lpparam.appInfo = appInfo;
            lpparam.isFirstApplication = true;

            XC_LoadPackage.callAll(lpparam);

            if (reportedPackageName.equals(INSTALLER_PACKAGE_NAME))
                hookXposedInstaller(lpparam.classLoader);
        }
    });

    ...
}
```

Xposed在Zygote进程中执行完这些之后，每次fork出来的都是`handleBindApplication`已经被Hook过的进程。所以，当每个应用进程被打开的时候，都会最开始就先执行Xposed的模块插件，然后才开始执行本身的代码。这样就实现了注入到每个应用程序来进行Hook。

## 如何实现Hook

### `findAndHookMethod`

`findAndHookMethod`想必是大家写Xposed插件最常用及最熟悉的一个方法，我们就通过这个方法来分析Xposed是如何实现的Java Hook。

```java
public static XC_MethodHook.Unhook findAndHookMethod(
        Class<?> clazz,
        String methodName,
        Object... parameterTypesAndCallback) {

    if (parameterTypesAndCallback.length == 0
            || !(parameterTypesAndCallback[parameterTypesAndCallback.length - 1]
            instanceof XC_MethodHook)) {
        throw new IllegalArgumentException("no callback defined");
    }

    XC_MethodHook callback =
            (XC_MethodHook) parameterTypesAndCallback[
                    parameterTypesAndCallback.length - 1];

    Method m = findMethodExact(
            clazz,
            methodName,
            getParameterClasses(
                    clazz.getClassLoader(),
                    parameterTypesAndCallback));

    return XposedBridge.hookMethod(m, callback);
}
```

大家知道这个方法的参数是个可变长参数列表，然后它会取最后一个参数来作为callback，即Hook时注入的代码。然后通过反射的方法来得到原本的`Method`。最后将这两个传给`hookMethod`。

```java
public static XC_MethodHook.Unhook hookMethod(
        Member hookMethod,
        XC_MethodHook callback) {

    ...

    if (newMethod) {
        Class<?> declaringClass = hookMethod.getDeclaringClass();

        int slot;
        Class<?>[] parameterTypes;
        Class<?> returnType;

        if (runtime == RUNTIME_ART) {
            slot = 0;
            parameterTypes = null;
            returnType = null;
        } else if (hookMethod instanceof Method) {
            slot = getIntField(hookMethod, "slot");
            parameterTypes = ((Method) hookMethod).getParameterTypes();
            returnType = ((Method) hookMethod).getReturnType();
        } else {
            slot = getIntField(hookMethod, "slot");
            parameterTypes = ((Constructor<?>) hookMethod).getParameterTypes();
            returnType = null;
        }

        AdditionalHookInfo additionalInfo =
                new AdditionalHookInfo(callbacks, parameterTypes, returnType);

        hookMethodNative(
                hookMethod,
                declaringClass,
                slot,
                additionalInfo);
    }

    ...
}
```

可以看到，`hookMethod`实际上是调用了一个native函数`hookMethodNative`。其他参数都很好理解，但是取的这个`slot`是个什么东西？翻了一下百度，在Dalvik的源码中`dalvik/vm/reflect.c`：

```c
static int methodToSlot(const Method* meth)
{
    ClassObject* clazz = meth->clazz;
    int slot;

    if (dvmIsDirectMethod(meth)) {
        slot = meth - clazz->directMethods;
        slot = -(slot + 1);
    } else {
        slot = meth - clazz->virtualMethods;
    }

    return slot;
}
```

原来这个`slot`就是此method在ClassObject的methods中的偏移，`directMethods`为负，`virtualMethods`为正。

### 偷梁换柱

`hookMethodNative`对应C函数`XposedBridge_hookMethodNative`。
