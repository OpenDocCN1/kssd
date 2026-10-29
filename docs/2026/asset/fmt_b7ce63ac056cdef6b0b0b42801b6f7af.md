
Xposed 是一个模块框架，可以在不触及任何 APK 的情况下改变系统和应用程序的行为。这很棒，因为这意味着模块可以在不同版本甚至 ROM 上工作而无需任何更改（只要原始代码没有太大更改）。它也很容易撤消。由于所有更改都在内存中完成，您只需停用模块并重新启动即可恢复原始系统。还有许多其他优点，但这里只是一个优点：多个模块可以对系统或应用程序的同一部分进行更改。对于修改后的 APK，您必须选择一个。没有办法组合它们，除非作者用不同的组合构建了多个 APK。

从图1-2中还能看到它支持Android 8.1-13 的系统版本。补充说明下，Xposed旧版的API只支持的Android7，后来更新的出来的一些版本，如`v89`，是支持Android 8的版本的。

## 1.1.2 Xposed && Lsposed 框架原理

Xposed的Hook原理是从整个Android的启动流程入手而设计出来的框架，看懂Android的启动流程，也是从从按了开机键后，从硬件到软件，到底做了什么事情，我们才能更好的理解Xposed框架。

### 1.1.2.1 Android 启动流程

如图1-3所示：是Android启动的整个流程图，下面我们一一对每个节点进行介绍。

![](https://bbs.kanxue.com/upload/attach/202209/799845_RU4GJA9FCUAKVKE.png)

图1-3 Android 启动流程图

Android整系统分为四层，分别为kernel、Native、FrameWork、应用层（APP），loader也可以单独算一层，是硬件的启动加载预置项。

1. 首先当我们长按开机键（电源按钮）开机，此时会引导芯片开始从固化到ROM中的预设代码处执行，然后加载引导程序到RAM。然后启动加载的引导程序，引导程序主要做一些基本的检查，包括RAM的检查，初始化硬件的参数。

2. 到达内核层的流程后，这里初始化一些进程管理、内存管理、加载各种Driver等相关操作，如Camera Driver、Binder Driver 等。下一步就是内核线程，如软中断线程、内核守护线程。下面一层就是Native层，这里额外提一点知识，层于层之间是不可以直接通信的，所以需要一种中间状态来通信。Native层和Kernel层之间通信用的是`syscall`，Native层和Java层之间的通信是`JNI`。

3. 在Native层会初始化`init`进程，也就是用户组进程的祖先进程。`init`中加载配置文件`init.rc`，`init.rc`中孵化出`ueventd`、`logd`、`healthd`、`installd`、`lmkd`等用户守护进程。开机动画启动等操作。核心的一步是孵化出`Zygote`进程，此进程是所有APP的父进程，这也是Xposed注入的核心，同时也是Android的第一个Java进程（虚拟机进程）。

4. 进入框架层后，加载`zygote init`类，注册`zygote socket`套接字，通过此套接字来做进程通信，并加载虚拟机、类、系统资源等。`zygote`第一个孵化的进程是`system_server`进程，负责启动和管理整个Java Framework，包含`ActivityManager`、`PowerManager`等服务。

5. 应用层的所有APP都是从`zygote`孵化而来

### 1.1.2.2 Xposed 注入源码剖析

上述是Android的大致流程，接下来我们从代码执行的角度来看执行链。

从图1-3中，我们可以总结出一条启动链：

|  |  |
| --- | --- |
| 1 | `init` `=> init.rc` `=> app_process` `=> zygote` `=> ...` |

一个应用的启动，核心的步骤是Framework层的`zygote`启动，`zygote`是所有进程的父进程，也就是说，所有APP应用进程都是由`zygote`孵化而来。为什么要从Native层开始说明呢？这是因为Native层开始就是Android源码的运行过程，Xposed的注入也就是从Native层开始的。

根据Android启动流程图可知，`zygote`是由`app_process`初始化产生的。`app_process`是一个二进制可执行文件，它的表现形式是一个bin文件，它位于`/system/bin/app_process`，如图1-4所示：

![](https://bbs.kanxue.com/upload/attach/202209/799845_YUGY8VKZ8ZXKNMH.png)

图1-4 Android app_process

`app_process`是由`app_main.cpp`编译而来，它的源码路径位于`/frameworks/base/cmds/app_process/app_main.cpp`，Xposed就是将`XposedInstaller.jar`包替换原始的`app_process`来实现全局注入，所以我们每次编写完Hook代码后，需要重启手机才能生效。

- init

从Android启动的流程图可知，首先进行的是`init`进程的启动，它在源码中对应着`/system/core/init/init.cpp`文件，如图1-5所示：

![](https://bbs.kanxue.com/upload/attach/202209/799845_T32T6BCVUNSRA8Y.png)

图1-5 /system/core/init/init.cpp

C++文件的启动从`main`开始，所以我们从`main`开始追，核心代码如下所示：

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23 | `int` `main(int argc, char** argv) {`  `...`    `if` `(bootscript.empty()) {`  `/` `/` `第一次开机`  `parser.ParseConfig("/init.rc");`  `/` `/` `解析配置文件 init.rc`  `parser.set_is_system_etc_init_loaded(`  `parser.ParseConfig("/system/etc/init"));`  `parser.set_is_vendor_etc_init_loaded(`  `parser.ParseConfig("/vendor/etc/init"));`  `parser.set_is_odm_etc_init_loaded(parser.ParseConfig("/odm/etc/init"));`  `}` `else` `{`  `parser.ParseConfig(bootscript);`  `parser.set_is_system_etc_init_loaded(true);`  `parser.set_is_vendor_etc_init_loaded(true);`  `parser.set_is_odm_etc_init_loaded(true);`  `}`    `...`    `while` `(true) { ... }`    `return` `0;`  `}` |

可以看到，第一次开机后，它会解析`init.rc`配置文件，此配置文件中会创建文件，并做一些用户组分配、权限赋予的操作，它位于`/system/core/rootdir/init.rc`，如图1-6所示：

![](https://bbs.kanxue.com/upload/attach/202209/799845_2YJGSTWFJ4EMBFR.png)

图1-6 /system/core/rootdir/init.rc

此外，还会有一些触发器的操作，源码如下所示：

|  |  |
| --- | --- |
| 1  2 | `# Now we can start zygote for devices with file based encryption`  `trigger zygote-start` |

`zygote`的生成就是由触发器生成的，在稍高一点的Android版本中，会分为32位和64位的`zygote`，如图1-7所示：

![](https://bbs.kanxue.com/upload/attach/202209/799845_HBF3SWCVPBCTWHB.png)

图1-7 不同位数的zygote

这里以`init.zygote32.rc`为例做说明，其它东西都是一样的，就是架构有区别。在`init.zygote32.rc`配置文件中，有如下代码：

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14 | `service zygote` `/system/bin/app_process -Xzygote /system/bin --zygote --start-system-server`  `class main`  `priority -20`  `user root`  `group root readproc`  `socket zygote stream 660 root system`  `onrestart write /sys/android_power/request_state wake`  `onrestart write /sys/power/state on`  `onrestart restart audioserver`  `onrestart restart cameraserver`  `onrestart restart media`  `onrestart restart netd`  `onrestart restart wificond`  `writepid /dev/cpuset/foreground/tasks` |

我们只看第一句命令，这个是启动`zygote`进程的核心。这里加载了`app_process`文件，那么这个文件做了什么呢？`app_process`对应着Android源码中的`app_main.cpp`,他的位置如图1-8所示：

![](https://bbs.kanxue.com/upload/attach/202209/799845_NR5DBQJTEAF4TUV.png)

图1-8 app_main.cpp 位置

同样，我们还是看它的`main`函数，只不过这里要看他后面跟的参数是什么含义，参数匹配如图1-9所示：

![](https://bbs.kanxue.com/upload/attach/202209/799845_3FEXPV9GMGX7F2T.png)

图1-9 app_main.cpp main 参数匹配

代码中，可以看到对`zygote`做了标记，那我们继续追踪，看看哪里引用了这个`bool`变量，全局搜索后，最后匹配到了文件的末尾，代码如图1-10所示：

![](https://bbs.kanxue.com/upload/attach/202209/799845_UR28U6PAVZ6R8TN.png)

图1-10 zygote 引用

`runtime.start`加载了`com.android.internal.os.ZygoteInit`类，继续追踪`runtime.start`，最后定位到`/frameworks/base/core/jni/AndroidRuntime.cpp`，核心代码如下所示：

```
void AndroidRuntime::start(const char* className, const Vector<String8>& options, bool zygote)
{
...
/`初始化 JNI 接口`
/* start the virtual machine
*/
JniInvocation jni_invocation;
jni_invocation.Init(NULL);

/`创建 env 指针`
JNIEnv* env;
/`
if (startVm(&mJavaVM, &env, zygote) != 0) {
return;
}
onVmCreated(env);

/*
* Register android functions.
*/
if (startReg(env) < 0) {
ALOGE("Unable to register all android natives\n");
return;
}

/*
* We want to call main() with a String array with arguments
in it.
* At present we have two arguments, the class name and an option string.
* Create an array to hold them.
*/
jclass stringClass;
jobjectArray strArray;
jstring classNameStr;

stringClass = env->FindClass("java/lang/String");
assert(stringClass != NULL);
strArray = env->NewObjectArray(options.size() + 1, stringClass, NULL);
assert(strArray != NULL);
classNameStr = env->NewStringUTF(className);
assert(classNameStr != NULL);
env->SetObjectArrayElement(strArray, 0, classNameStr);

for (size_t i = 0; i < options.size(); ++i) {
jstring optionsStr = env->NewStringUTF(options.itemAt(i).string());
assert(optionsStr != NULL);
env->SetObjectArrayElement(strArray, i + 1, optionsStr);
}

/*
* Start VM.  This thread becomes the main thread of the VM, and will
* not return until the VM exits.
*/
/`开启 VM 虚拟机，这个线程变成 VM 主线程，直到 VM 退出才结束`/
/`className => com.android.internal.os.ZygoteInit`
char* slashClassName = toSlashClassName(className != NULL ? className : "");
jclass startClass = env->FindClass(slashClassName);
if (startClass == NULL) {
ALOGE("JavaVM unable to locate class '%s'\n", slashClassName);
/* keep going */
} else {
/`把 main 函数调起来`
jmethodID startMeth = env->GetStaticMethodID(startClass, "main", "([Ljava/lang/String;)V");
if (startMeth == NULL) {
ALOGE("JavaVM unable to find main() in '%s'\n", className);
/* keep going */
} else {
env->CallStaticVoidMethod(startClass, startMeth, strArray);

#if 0   // 条件编译
if (env->ExceptionCheck())
threadExitUncaughtException(env);
#endif
}
}
free(slashClassName);
...
}
```
