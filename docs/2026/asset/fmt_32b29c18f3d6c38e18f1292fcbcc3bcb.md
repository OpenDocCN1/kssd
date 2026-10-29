
但是我的需求就是只需要处理`handleLoadPackage`就可以了，所以给这个参数加个开关`isLoadHookLoadPackage`，我的代码如下

```
if (XposedBridge.isZygote) {
        if (moduleInstance instanceof IXposedHookZygoteInit) {
            IXposedHookZygoteInit.StartupParam param = new IXposedHookZygoteInit.StartupParam();
            param.modulePath = apk;
            param.startsSystemServer = startsSystemServer;
            ((IXposedHookZygoteInit) moduleInstance).initZygote(param);
        }
        if (moduleInstance instanceof IXposedHookLoadPackage)
            if(isLoadHookLoadPackage)
            {
                XposedBridge.hookLoadPackage(new IXposedHookLoadPackage.Wrapper((IXposedHookLoadPackage) moduleInstance));
            }
        if (moduleInstance instanceof IXposedHookInitPackageResources)
            XposedBridge.hookInitPackageResources(new IXposedHookInitPackageResources.Wrapper((IXposedHookInitPackageResources) moduleInstance));
    } else {
        if (moduleInstance instanceof IXposedHookCmdInit) {
            IXposedHookCmdInit.StartupParam param = new IXposedHookCmdInit.StartupParam();
            param.modulePath = apk;
            param.startClassName = startClassName;
            ((IXposedHookCmdInit) moduleInstance).initCmdApp(param);
        }
    }
```

实际上就是简单的加了一个`if`而已，然后在`main`的地方改成

```
XposedInit.loadModules(false);
```

最后就是在hook `handleBindApplication`的地方加上`XposedInit.loadModules(true);`

```
findAndHookMethod(ActivityThread.class, "handleBindApplication", "android.app.ActivityThread.AppBindData", new XC_MethodHook() {
    @Override
    protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
        XposedInit.loadModules(true);
...
```

嗯，炒鸡简单，改源码不超过2分钟。

## 编译

改源码两分钟，编译五小时..

rovo89还提供了另外一个项目:449K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6J5L8%4k6G2z5o6W2Q4x3V1k6j5M7r3!0K6k6h3c8f1L8$3!0D9M7H3`.`.

因为Xposed编译需要依赖AOSP来进行编译，这个工具就是专门用来编译Xposed源码的，使用方法也很简单，填好`build.conf`然后一把梭就可以了，具体的可以查看百度。

需要注意的是，上叙的源码是新版的xposed，就是支持5.0+art的，而我使用的是4.4.4的源码，模拟器也是4.4，所以clone源码的时候要`-b master`来切换分支。然后修改的代码也大同小异，就是`loadmodule`多了个`startClassName`参数，可以通过`getStartClassName()`方法获取，就是这样：

`XposedBridge.loadModules(getStartClassName(),true);`

因为这个老工程是个Eclipse工程，XposedTool是依赖于Gradle，所以无法使用XposedTool编译。直接使用Android Studio - Build Apk最后把文件名改成`XposedBridge.jar`即可，

最后你可以选择直接修改Xposed Installer将其中的`XposedBridge.jar`替换再安装，或者在手机上安装好Xposed，然后将修改好的`XposedBridge.jar`替换`/data/data/de.robv.android.xposed.installer/bin/XposedBridge.jar`，重启，就可以愉快的调插件玩耍了。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于  2018-2-24 01:40

被葫芦娃编辑

，原因：
