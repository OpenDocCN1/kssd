
# [原创]一种基于JDWP动态注入代码的方案

## 前言

在逆向开发中，一般都需要对目标App进行代码注入。主流的代码注入工具是`Frida`，这个工具能稳定高效实现`java`代码`hook`和`native`代码`hook`，不过缺点是需要使用Root设备，而且用`js`开发，入门门槛较高。最近发现一种非Root环境下对Debug App进行代码注入的方案，原理是利用Java调试框架，通过调试器与目标虚拟机之间通讯，实现对虚拟机进程的修改。

## JPDA框架和JDWP协议

Java SE从1.2.2版本以后推出了`JPDA`框架（Java Platform Debugger Architecture，Java平台调试体系结构）。`JPDA`定义了一套独立且完整的调试体系，它由三个相对独立的模块组成，分别为：

* `JVM TI`：Java虚拟机工具接口（被调试者）。被调试者运行在我们想要调试的虚拟机上，它可以通过`JVM TI`这个标准接口监控当前虚拟机的信息。
* `JDWP`：Java Debug Wire Protocol，Java调试协议（通道）。在调试者和被调试者之间，通过`JDWP`传输层传输消息。
* `JDI`：Java Debug Interface，Java调试接口（调试者）。调试者定义了用户可以使用的调试接口，用户可以通过这些接口对被调试虚拟机发送调试命令，同时显示调试结果。

![JPDA整体架构](upload/attach/202202/849838_DABHGR9EV6PDUTG.png)

Reference: `030K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8%4u0S2j5$3I4W2i4K6u0W2j5$3!0E0i4K6u0r3K9X3q4$3j5i4y4W2i4K6u0r3y4#2)9J5c8X3c8G2j5%4y4Q4x3V1k6@1k6h3y4Z5L8X3!0@1k6i4y4Q4x3V1k6Y4N6h3W2V1k6i4y4Q4x3V1k6B7M7r3c8S2i4K6u0r3j5i4u0U0K9r3W2@1k6h3y4@1N6i4u0W2i4K6u0W2K9s2c8E0L8l9`.`.

其中，`JDWP`协议是用于调试器与目标虚拟机之间进行调试交互的通信协议。

`JDWP`大致分为两个阶段：握手和应答。握手是在传输层连接建立完成后，做的第一件事：

调试器发送 14 bytes 的字符串"`JDWP-Handshake`"到目标虚拟机，虚拟机回复"`JDWP-Handshake`" ，从而完成握手。

握手完成后，调试器就可以向虚拟机发送命令了。`JDWP`是通过命令（`command`）和回复（`reply`）进行通信，这与 HTTP 有些相似。`JDWP` 本身是无状态的，因此对 `command` 出现的顺序并不受限制。

`JDWP` 有两种基本的包（`packet`）类型：命令包（`command packet`）和回复包（`reply packet`）。

调试器和目标虚拟机都有可能发送 `command packet`。调试器通过发送 `command packet` 获取虚拟机的信息以及控制程序的执行。虚拟机通过发送 `command packet` 通知调试器某些事件的发生，如到达断点或是产生异常。

`Reply packet` 是用来回复 `command packet` 该命令是否执行成功，如果成功 `reply packet` 还有可能包含 `command packet` 请求的数据，比如当前的线程信息或者变量的值。从虚拟机发送的事件消息是不需要回复的。

数据包部分`JDWP`协议按照功能大致分为`18`组命令，包含了虚拟机、引用类型、对象、线程、方法、堆栈、事件等不同类型的操作命令。

`ART`虚拟机对`JDWP`协议的支持基本是完整的，具体信息可以参考`ART-JDWP`中所支持的消息。

## JDWP协议的实现

`JDWP`协议内容比较多，要自行实现协议内容工作量还是比较大。庆幸的是，国外已有大神将`JDWP`协议大部分用`python`实现好了，我们只需要直接使用即可，非常方便。

`python`实现的`jdwp`协议源码地址：`jdwp-shellifier`

## 基于JDWP的代码注入方案

### Native代码注入

既然利用`JDWP`可以让调试器跟虚拟机进行交互，我们可以通过调用基于`JDWP`协议的相关接口，向虚拟机进程中注入代码。假如只是注入`c/c++`代码的话，实现起来很轻松，我们在App进程启动时加上一个断点，在断点处执行加载`so`的代码即可，流程如下：

1. 利用`JDWP`的命令，在App进程启流程的某个方法中添加断点，使得App启动前能执行到我们注入的指令。为了使注入的代码尽早执行，这里选择`android.app.LoadedApk.makeApplication`方法处加断点；
2. 在断点处通过`jdwp`协议执行下面的Java代码：

```
Runtime.getRuntime().load("data/data/package_name/libnative_injecter.so")
```

3. 注入的`so`加载完成后，继续App的启动流程;

这样`native`代码就注入到了目标App中。

### Java代码注入

通过`jdwp`协议封装的接口可以实现`java`代码的注入，通过这种方式注入少量Java代码还比较轻松，大量`java`代码都用`jdwp`来实现，难度将会非常大。

我们可以将`java`代码编译成的是`dex`文件，然后用`c/c++`实现`dex`文件的加载以及`dex`方法的执行，便可实现`java`代码的注入。

在插件化开发中，加载`dex`文件大致有两种方案，一种是多`ClassLoader`方案，一种是单`ClassLoader`方案。多`ClassLoader`方案就是根据插件`dex`路径，每个插件构造自己的`DexClassLoader`，然后用这个`classLoader`加载插件中的类。单`ClassLoader`就是将插件的`ClassLoader`里的`Element`合并到App的`ClassLoader`中，然后使用App的`ClassLoader`来加载插件里的类。

这里我们选择单`ClassLoader`的方案，具体步骤如下:

1. 根据插件`dex`或者`Apk`路径，反射调用`DexPathList.java`的`makePathElements`静态方法，构造出来一个用于类加载的`Element`数组；
2. 获取App的`Classloader`，这是一个`BaseDexClassLoader`对象，反射获取成员变量`pathList`，其类型时`DexPathList`，再反射获取成员变量`dexElements`，得到一个`Element`数组；
3. 将第一步获取到的`Element`数组合并到第二步获取到的`dexElements`对象对应的`Element`数组中;
4. 最后用App的`classLoader`加载插件`Apk`中的类，并执行插件入口方法；

以上流程需要用`c/c++`来实现。

### Xposed模块加载器的注入

为了在注入的代码中更方便地修改被注入App Java代码，我们希望注入的代码能够给App代码加钩子。因此，在注入代码中接入了稳定性较好的一个Android Art Hook库: `SandHook`。

接入这个Hook库的方法有两种：

1. 方法一：在每个需要动态注入的插件工程中接入`SandHook`，然后在插件工程中使用Xposed Api来Hook Java代码；
2. 方法二：将Xposed Api的`SandHook`所有Java代码(`dex`文件)和Native代码(`so`文件)注入到目标App中，并增加加载Xposed插件的相关逻辑。注入的插件工程就可以按照一个Xposed Module工程模式来开发，这样能显著降低插件工程的接入成本。

方案二优势更明显，因此这里采用了方案二来实现。

最终，在利用`JDWP`协议注入的`so`文件中，需要实现以下功能：

1. 将加载Xposed插件的功能编译出`dex`文件，用`dex`构造出来新的`Element`合并到App的`ClassLoader`中，同时`so`路径也要合并到App的`ClassLoader` `nativeBianryPath`中；
2. 调用`dex`文件中的加载指定Xposed模块的方法，完成外置插件的动态注入；

整体流程大致如此，不过其中还有不少细节需要处理，比如，`SandHook`初始化时需要传App Context，但是我们这个注入流程是在`LoadedApk.makeApplication`之前，此时App的Context并没有创建出来，因此，需要通过反射主动构造出一个Context对象：

```
LoadedApk loadedApk = ActivityThread.currentActivityThread().mBoundApplication.info;
ContextImpl appContext = ContextImpl.createAppContext(activityThread, loadedApk);
```

还有，为了能够加载Xposed插件中的`so`库，在加载插件`Apk`之前，需要将`Apk`中的`so`文件拷贝到`data/data`目录下，并将`so`路径传给`DexClassLoader` 构造方法的最后一个参数。为了更高效地拷贝`so`，这里反射调用了Framework里的内部类`NativeLibraryHelper`。App安装时的`so`拷贝就是用`NativeLibraryHelper`实现的，具体拷贝操作在`native`层完成，效率更高。

另外，Android9及以上的系统限制了App对隐藏Api的调用。我们可以在注入`so`的`JNI_Onload`函数中加入以下代码，便可简单绕过这种限制：

```
static void BypassHiddenApi(JNIEnv *env) {
jclass vmRumtime_class = env->FindClass("dalvik/system/VMRuntime");
void *getRuntime_art_method = env->GetStaticMethodID(vmRumtime_class,
"getRuntime",
"()Ldalvik/system/VMRuntime;");
jobject vmRuntime_instance = env->CallStaticObjectMethod(vmRumtime_class, (jmethodID)getRuntime_art_method);
jstring mystring = env->NewStringUTF("L");
jclass cls = env->FindClass("java/lang/String");
jobjectArray jarray = env->NewObjectArray(1, cls, nullptr);
env->SetObjectArrayElement(jarray, 0, mystring);
void *setHiddenApiExemptions_art_method = env->GetMethodID(vmRumtime_class,
"setHiddenApiExemptions",
"([Ljava/lang/String;)V");
env->CallVoidMethod(vmRuntime_instance, (jmethodID)setHiddenApiExemptions_art_method, jarray);
}
```

## 使用方法
