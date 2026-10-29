
```java
}

if (moduleInstance instanceof IXposedHookLoadPackage)

XposedBridge.hookLoadPackage(new IXposedHookLoadPackage.Wrapper((IXposedHookLoadPackage) moduleInstance));

if (moduleInstance instanceof IXposedHookInitPackageResources)

XposedBridge.hookInitPackageResources(new IXposedHookInitPackageResources.Wrapper((IXposedHookInitPackageResources) moduleInstance));

} else {

if (moduleInstance instanceof IXposedHookCmdInit) {

IXposedHookCmdInit.StartupParam param = new IXposedHookCmdInit.StartupParam();

param.modulePath = apk;

param.startClassName = startClassName;

((IXposedHookCmdInit) moduleInstance).initCmdApp(param);

...

}
```

但是我的需求就是只需要处理`handleLoadPackage`就可以了，所以给这个参数加个开关`isLoadHookLoadPackage`，我的代码如下：

```java
if (XposedBridge.isZygote) {
    if (moduleInstance instanceof IXposedHookZygoteInit) {
        IXposedHookZygoteInit.StartupParam param = new IXposedHookZygoteInit.StartupParam();
        param.modulePath = apk;
        param.startsSystemServer = startsSystemServer;
        ((IXposedHookZygoteInit) moduleInstance).initZygote(param);
    }
    if (moduleInstance instanceof IXposedHookLoadPackage)
        if (isLoadHookLoadPackage) {
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

实际上就是简单地加了一个`if`而已，然后在`main`的地方改成：

```java
XposedInit.loadModules(false);
```

最后就是在`hook handleBindApplication`的地方加上`XposedInit.loadModules(true);`：

```java
findAndHookMethod(ActivityThread.class, "handleBindApplication", "android.app.ActivityThread.AppBindData", new XC_MethodHook() {

    @Override
    protected void beforeHookedMethod(MethodHookParam param) throws Throwable {

        XposedInit.loadModules(true);

        ...
```

嗯，炒鸡简单，改源码不超过 2 分钟。

## 编译

改源码两分钟，编译五小时……

rovo89 还提供了另外一个项目：<https://github.com/rovo89/XposedTools>

因为 Xposed 编译需要依赖 AOSP 来进行编译，这个工具就是专门用来编译 Xposed 源码的，使用方法也很简单，填好 `build.conf` 然后一把梭就可以了，具体的可以查看百度。

需要注意的是，上述的源码是新版的 Xposed，就是支持 5.0+ ART 的，而我使用的是 4.4.4 的源码，模拟器也是 4.4，所以 clone 源码的时候要 `-b master` 来切换分支。然后修改的代码也大同小异，就是 `loadModule` 多了个 `startClassName` 参数，可以通过`getStartClassName()`方法获取，就是这样：

```java
XposedBridge.loadModules(getStartClassName(), true);
```

因为这个老工程是个 Eclipse 工程，XposedTool 是依赖于 Gradle，所以无法使用 XposedTool 编译。直接使用 Android Studio - Build Apk，最后把文件名改成 `XposedBridge.jar` 即可。

最后你可以选择直接修改 Xposed Installer，将其中的 `XposedBridge.jar` 替换再安装；或者在手机上安装好 Xposed，然后将修改好的 `XposedBridge.jar` 替换`/data/data/de.robv.android.xposed.installer/bin/XposedBridge.jar`，重启，就可以愉快地调插件玩了。

~~顺便 fork 了一份，并把代码上传了，虽说总共也没改几行代码，2333。。~~

~~5.0-：<mark class="encrypted">0c8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6Z5L8s2g2%4j5g2)9J5c8W2S2H3L8%4y4W2k6p5u0J5K9h3c8Y4k6g2)9J5c8Y4c8J5k6h3g2Q4x3V1k6E0j5i4y4@1k6i4u0Q4y4@1g2Q4y4@1f1`.</mark>~~

~~4.4+：<mark class="encrypted">946K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6Z5L8s2g2%4j5g2)9J5c8W2S2H3L8%4y4W2k6p5u0J5K9h3c8Y4k6g2)9J5c8Y4c8J5k6h3g2Q4x3V1k6S2M7Y4c8Q4y4@1g2Q4y4@1f1`.</mark>~~

# Xposed 的实现解析

## Xposed 简介

Xposed 已经作为 Android 的 Java 层 Hook 大哥好多年了，不仅仅是安全研究者和逆向工程师手中的神器，还拥有着很多插件开发者。在官方仓库收录的插件也已经超过了 1000+ 个，而其服务的用户以各大搞机论坛用户为主，也是不尽其数。

Xposed 的代码是完全开源的，并且代码量也不算很大。对于安全研究者来说，我觉得研究一下 Xposed 的原理还是很有必要的。

Xposed 的 GitHub 地址：

`0ccK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6J5L8%4k6G2z5o6W2Q4x3V1k6j5M7r3!0K6k6h3b7`.

`6ebK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6J5L8%4k6G2z5o6W2Q4x3V1k6j5M7r3!0K6k6h3c8n7M7X3W2V1k6$3f1`.

**本文使用 Android 4.4 的源码进行分析。**

## 如何实现全局注入

简单地讲，Xposed 是通过替换修改过的 `app_process` 来注入 Zygote，以实现全局注入的。

### Android 的启动过程

想要知道 Xposed 是如何实现全局注入的，首先要分析一下 Android 系统的启动过程，就可以很方便地理解 Xposed 选择的注入点。

我画了张图，Android 系统的启动大致如下：

![Android 系统启动流程](https://bbs.kanxue.com/./upload/attach/201801/715510_b4400wvd87wa7hp.png)

从图中可以很方便地理解从 BootLoader 到显示系统界面的过程。

### 应用孵化器：Zygote

#### Zygote 介绍

Zygote，中文是“受精卵”……实际上它就是一个孵化器，所有应用程序的进程都是由它 fork 出来的。

```text
USER     PID   PPID  VSIZE  RSS     WCHAN    PC         NAME

root      155   1     509740 41876 ffffffff b75761e0 S zygote

drm       156   1     18508  4064  ffffffff b7599ff6 S /system/bin/drmserver

media     157   1     69296  15656 ffffffff b755bff6 S /system/bin/mediaserver

install   158   1     6580   1228  c04d1048 b7507bb6 S /system/bin/installd

keystore  159   1     10172  4064  c03f4a25 b7561ff6 S /system/bin/keystore

root      160   1     1096   4     c044881c 080ddd03 S /system/xbin/su

system    187   1     81040  3860  ffffffff b75b0ff6 S /system/bin/surfaceflinger

root      343   2     0      0      c01cfbd5 00000000 S flush-8:16

root      414   62    6732   1412   c02f4452 b74c6bb6 S /system/bin/sh

root      418   62    6688   1464   c01c0a90 b752e1e0 S logcat

system    424   155   629276 46384 ffffffff b7575ff6 S system_server

u0_a44    506   155   564252 75032 ffffffff b75779eb S com.android.systemui

u0_a0     551   155   525484 25696 ffffffff b75779eb S android.process.acore

wifi      565   1     10776  2912   c01c0a90 b741d1e0 S /system/bin/wpa_supplicant

system    566   155   526468 21732 ffffffff b75779eb S com.android.settings

u0_a22    606   155   521276 24932 ffffffff b75779eb S com.android.inputmethod.latin

radio     623   155   539356 28484 ffffffff b75779eb S com.android.phone

u0_a23    637   155   557680 45252 ffffffff b75779eb S com.android.launcher

u0_a29    673   155   520300 23100 ffffffff b75779eb S com.android.music

u0_a16    693   155   521532 25876 ffffffff b75779eb S android.process.media

u0_a49    719   155   517580 18600 ffffffff b75779eb S com.android.smspush

u0_a15    806   155   521632 19920 ffffffff b75779eb S com.android.dialer

bluetooth 821   155   522828 22284 ffffffff b75779eb S com.android.bluetooth

dhcp      845   1     6656   1452   c01c0a90 b74e6ab6 S /system/bin/dhcpcd

u0_a6     916   155   527828 22036 ffffffff b75779eb S com.android.calendar

u0_a7     943   155   519636 23176 ffffffff b75779eb S com.android.providers.calendar

u0_a12    967   155   521156 24960 ffffffff b75779eb S com.android.deskclock

u0_a17    1019  155   528300 24912 ffffffff b75779eb S com.android.email

u0_a18    1078  155   524976 20436 ffffffff b75779eb S com.android.exchange

u0_a28    1168  155   530852 23100 ffffffff b75779eb S com.android.mms

u0_a32    1215  155   517576 19140 ffffffff b75779eb S com.android.onetimeinitializer

u0_a47    1251  155   517732 19072 ffffffff b75779eb S com.android.voicedialer

```

可以看到，这些包名格式的进程的 PPID（父进程 PID）都是 155，即 Zygote 进程。

Zygote 启动之后会建立一个 Socket Server，然后 fork 自身成为一个新的进程——`system_server`，并让它成为“前端代言人”。当 `system_server` 接收到打开进程的请求时，它就会通过 Socket 跟 Zygote 通讯，Zygote 就再 fork 自身成为新的进程。

#### Zygote 的启动

在 `init.rc` 中可以看到，Zygote 是通过 `app_process` 启动的：

```text
service zygote /system/bin/app_process -Xzygote /system/bin --zygote --start-system-server
```

这时 `app_process` 的任务也很简单，就是把自己的进程名变成 Zygote，然后反射调用 Zygote 的主方法，即 `com.android.internal.os.ZygoteInit.main()`。之后 Zygote 做的事情就是在介绍中说过的了。

#### Hook Zygote

那么 Xposed 就是通过修改 `app_process` 的方式来实现注入 Zygote 的。

项目地址：`54cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6J5L8%4k6G2z5o6W2Q4x3V1k6j5M7r3!0K6k6h3b7`. 即是修改 `app_process` 的源码，在 `app_main.cpp` 的 `main` 函数中可以看到。
