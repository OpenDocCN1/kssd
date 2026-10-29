
```java
void load(String absolutePath, ClassLoader loader) {
if (absolutePath == null) {
throw new NullPointerException("absolutePath == null");
}
String error = doLoad(absolutePath, loader);
if (error != null) {
throw new UnsatisfiedLinkError(error);
}
}
public void loadLibrary(String nickname) {
loadLibrary(nickname, VMStack.getCallingClassLoader());
}
void loadLibrary(String libraryName, ClassLoader loader) {
if (loader != null) {
String filename = loader.findLibrary(libraryName);
if (filename == null) {...
```

我们对比了Android6.0下的`System.load`和`System.loadLibrary`:

```
我们可以发现System.loadLibrary()中会修改类加载器，这个在我们后面hook过程可能会报错，而Runtime.loadLibray()中有重写的方法，则可以正确实现
```

### Android 7.0：

[System.java] `java.lang.System`:

```java
public static void load(String filename) {
Runtime.getRuntime().load0(VMStack.getStackClass1(), filename);
}
public static void loadLibrary(String libname) {
Runtime.getRuntime().loadLibrary0(VMStack.getCallingClassLoader(), libname);
}
```

[Runtime.java] `java.lang.Runtime`:

```java
synchronized void load0(Class fromClass, String filename) {
if (!(new File(filename).isAbsolute())) {
throw new UnsatisfiedLinkError(
"Expecting an absolute path of the library: " + filename); }
if (filename == null) {
throw new NullPointerException("filename == null");
}
String error = doLoad(filename, fromClass.getClassLoader());
if (error != null) {
throw new UnsatisfiedLinkError(error);
}
}
public void loadLibrary(String libname, ClassLoader classLoader) {
java.lang.System.logE("java.lang.Runtime#loadLibrary(String, ClassLoader)" +
" is private and will be removed in a future Android release");
loadLibrary0(classLoader, libname);
}
```

我们可以发现不同版本的区别：

```java
Android 6.0采用的是loadLibrary,6.0之后都采用的是loadLibrary0; 同理 load函数也一样,6.0之后采用的是load0
```

同时我们分析了`loadLibrary0`：

```
1. classLoader存在时，通过classLoader.findLibrary(libraryName)来获取存放指定so文件的路径；
2. classLoader不存在时，则通过getLibPaths()接口来获取
3. 最终调用nativeLoad加载指定路径的so文件
```

## 四、各类hook技术原理分析

hook技术就是指截获进程对某个API函数的调用，使得API的执行流程转向我们实现的代码片段，从而实现我们要的功能，在Android中使用hook的方法有很多，常用的Xposed和frida hook技术、inlinehook技术、基于inlinehook的开源框架Sandhook、PLT/Got hook技术、以及当下模拟cpu的Unicorn的hook技术，下面我们将逐一介绍其原理。

### 1.Xposed hook技术

Xposed的基本原理，我在[源码编译（3）——Xposed框架定制](https://bbs.pediy.com/thread-269627.htm)中已经给大家做了详细的讲解，其主要就是Android应用进程都是由 `zygote` 进程孵化而来，zygote对应的可执行程序就是`app_process`,posed 框架通过替换系统的 `app_process` 可执行文件以及虚拟机动态链接库，让 `zygote` 在启动应用程序进程时注入框架代码，进而实现对应用程序进程的劫持。

具体怎么实现hook技术，Xposed就是通过修改了Art虚拟机，将需要hook的函数注册为Native函数，当执行这一函数时，虚拟机会优先执行Native函数，然后执行java函数，这样就成功完成了函数的hook。

![image-20210919155021588](upload/attach/202205/905443\_A6M5RGRCQ2A3HGM.png)

具体实现流程:

在 Android 系统启动的时候， `zygote` 进程加载 `XposedBridge` 将所有需要替换的 `Method` 通过 JNI 方法 `hookMethodNative` 指向 Native 方法 `xposedCallHandler` ， `xposedCallHandler` 在转入 `handleHookedMethod` 这个 Java 方法执行用户规定的 Hook Func

![image-20220517133042720](upload/attach/202205/905443\_TTUBVG5KA7YWR9W.png)

`dvmCallMethodV`会根据`accessFlags`决定调用native还是java函数，因此修改`accessFlags`后，Dalvik会认为这个函数是一个native函数，便走向了native分支也就是说Xposed在对java方法进行hook时，先将虚拟机里面这个方法的`Method`的`accessFlag`改为native对应的值，然后将该方法的`nativeFunc`指向自己实现的一个native方法，这样方法在调用时，就会调用到这个native方法，接管了控制权

其他的就详细参考上篇文章了

### 2.Frida hook技术

`frida` 也是一种动态插桩工具，原理和Xposed hook一样，也是把java method转为native method，但是Art下的实现与Dalivk有所不同，这里就需要了解ART的运行机制，这里主要参考博客：<a href="elink@82aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0j5h3u0A6L8U0l9H3y4q4)9J5k6h3N6A6N6r3S2#2j5W2)9J5k6h3W2G2i4K6u0r3x3U0l9I4z5q4)9J5c8U0l9%4i4K6u0r3x3K6q4Q4x3V1k6y4j5h3y4Q4x3U0g2q4y4q4)9J5y4f1t1^5i4K6t1#2z5p5q4Q4x3U0g2q4y4#2)9J5y4f1u0o6i4K6t1#2z5e0k6Q4x3U0g2q4z5q4)9J5y4f1q4r3i4K6t1#2z5e0q4r3M7X3W2V1j5g2)9J5c8R3`.`.">Frida源码分析</a>

ART 是一种代替 Dalivk 的新的运行时,它具有更高的执行效率。ART虚拟机执行 Java 方法主要有两种模式：quick code 模式和 Interpreter 模式

```
quick code 模式：执行 arm 汇编指令
Interpreter 模式：由解释器解释执行 Dalvik 字节码
```

即使是在quick code模式中，也有类方法可能需要以Interpreter模式执行。反之亦然。解释执行的类方法通过函数`artInterpreterToCompiledCodeBridge`的返回值调用本地机器指令执行的类方法；本地机器指令执行的类方法通过函数`GetQuickToInterpreterBridge`的返回值调用解释执行的类方法

这里引用博客中的一张图

![image-20220517133042720](upload/attach/202205/905443\_VJRV8CFKFQS475P.png)

如图，对于一个native方法，ART虚拟机会先尝试使用quickcode的模式去执行，并检查`ARTMethod`结构中的`entry_point_from_quick_compiled_code`成员，这里分3种情况：

```
1.如果函数已经存在quick code, 则指向这个函数对应的 quick code的起始地址，而当quick code不存在时，它的值则会代表其他的意义；
2.当一个 java 函数不存在 quick code时，它的值是函数 artQuickToInterpreterBridge 的地址，用以从 quick 模式切换到 Interpreter 模式来解释执行 java 函数代码；
3.当一个 java native（JNI）函数不存在 quick code时，它的值是函数 art_quick_generic_jni_trampoline 的地址，用以执行没有quick code的 jni 函数
```

因此，frida将一个java method修改jni mthod 显然是不存在quick code，这时需要将`entry_point_from_quick_compiled_code`值修改为`art_quick_generic_jni_trampoline` 的地址

总结，frida把java method改为jni method，需要修改`ARTMethod`结构体中的这几个值：

```
accessflags = native
entry_point_fromjni = 自定义代码的入口
entry_point_from_quick_compiledcode = art_quick_generic_jni_trampoline函数的地址
entry_point_frominterpreter = artInterpreterToCompiledCodeBridge函数地址
```

### 3.inlinehook 技术

#### （1）基本原理

首先，我们先介绍一下什么是inline Hook:

```
inline Hook是一种拦截目标函数调用的方法，主要用于杀毒软件、沙箱和恶意软件。一般的想法是将一个函数重定向到我们自己的函数，以便我们可以在函数执行它之前和/或之后执行处理；这可能包括：检查参数、填充、记录、欺骗返回的数据和过滤调用。
hook是通过直接修改目标函数内的代码来放置，通常是用跳转覆盖的前几个字节，允许在函数进行任何处理之前重定向执行。
```

#### （2）inlineHook组成

```
hook:一个5字节的相对跳转，在被写入目标函数以钩住它，跳转将从被钩住的函数跳转到我们的代码
proxy:这是我们指定的函数（或代码），放置在目标函数上的钩子将跳转到该函数（或代码）
Trampoline:用于绕过钩子，以便我们可以正常调用钩子函数
```

#### （3）inlineHook实现

![image-20210918161352124](upload/attach/202205/905443\_CQY5DDQWTKHP3WS.png)

从示意图上，我们可以这样理解：

```
我们将目标函数MessgeBoxA()中的地址拿出来，然后我们用重写的hook函数替换，然后我们执行完成之后，再回调到函数的执行地址出，保证程序的正常运行
```

![image-20210919152200104](upload/attach/202205/905443\_JXZH5VUZP7ETYGZ.png)

我们也可以通过上述示意图去理解inlinehook的基本原理

#### （4）Android-Inline-Hook和SandHook 技术

Android-lnline-Hook和SandHook都是基于inlinehook的两种开源框架，在Android中对native层hook，使用的比较常见，前者主要针对32位进行hook，后者即可以用于32位也可以用于64位，但是官方表示32位并未进行测试，所以应用在64位上仍然更多

### 4.PLT/GOT hook技术

前面我们已经很详细的讲述了全局偏移表（GOT）和动态链接表（PLT）,Inline Hook能Hook几乎所有函数，但是兼容性较差，不能达到上线标准,相比于inlineHook，GOT Hook兼容性比较好，可以达到上线标准，但是只能Hook基于GOT表的一些函数

GOT/PLT Hook 主要是通过解析SO文件，将待hook函数在got表的地址替换为自己函数的入口地址，这样目标进程每次调用待hook函数时，实际上是执行了我们自己的函数

这里我们还要理解GOT表中含包含了导入表和导出表

```
导出表指将当前动态库的一些函数符号保留，供外部调用
导入表中的函数实际是在该动态库中调用外部的导出函数
```

例如导入表存放的是一些其他so的函数，例如libc的open，而导出表存放的是一些共其他so调用的函数，比如自己so中编写的函数，而无论导入表还是导出表基本都是针对导出函数，针对非导出函用inlinehook更常用一些

### 5.Unicorn hook技术

Unicore是一款非常优秀的跨平台模拟执行框架，该框架可以跨平台执行Arm, Arm64 (Armv8), M68K, Mips, Sparc, & X86 (include X86_64)等指令集的原生程序，通过模拟CPU，可以实现很多强大的功能，也可以实现函数级别的Hook

参考资料：无名大佬文章[Unicorn 在 Android 的应用](https://bbs.pediy.com/thread-253868.htm#msg_header_h1_7)

nicorn 内部并没有函数的概念，它只是一个单纯的CPU， 没有HOOK_FUNCTION的callback，AndroidNativeEmu 中的函数级Hook 并不是真正意义上的Hook，它不仅能Hook存在的函数，还能Hook不存在的函数。AndroidNativeEmu 使用这种技术实现了JNI函数Hook、库函数Hook。 Jni函数是不存的，Hook它只是为了能够用Python 实现 Jni Functions。有一些库函数是存在的，Hook只是为了重新实现它

## 五、各类hook技术实操

### 1.Xposed hook实操

#### （1）环境安装

Xposed环境安装详细可以参考我写的Xposed系列文章，这里只是简单的总结一下：

```java
(1) 4.4以下Android版本安装比较简单，只需要两步即可
1.对需要安装Xposed的手机进行root
2.下载并安装xposedInstaller,之后授权其root权限，进入app点击安装即可
但是由于官网不在维护，导致无法直接通过xposedinstaller下载补丁包
（2）Android 5.0-8.0 由于5.0后出现ART，所以安装步骤分成两个部分：xposed.zip 和
XposedInstaller.apk,zip文件是框架主体，需要进入Recovery后刷入，apk文件用于Xposed管理
1.完成对手机的root，并刷入reconvery(比如twrp),使用Superroot
2.下载你对应的zip补丁包，并进入recovery刷入
3.重启手机，安装xposedInstaller并授予root权限即可
官网地址：https://dl-xda.xposed.info/framework/
（3）由于Android 8.0后，Xposed官方作者没有再对其更新，我们一般就使用国内大佬riyu的Edxposed框架
Magisk + riyu + Edxposed
```

这里我们用的是nexus5进行操作，简单演示一下android6.0的Xposed安装

资源准备：

```
asop镜像：https://developers.google.com/android/ota#hammerhead
twrp: https://twrp.me/
xposed: https://dl-xda.xposed.info/framework/
xposed installer https://repo.xposed.info/module/de.robv.android.xposed.installer
```

首先我们先下载n5镜像，然后刷机，这里我们已经安装就不再安装了

然后我们刷入 `twrp-3.4.0-0-hammerhead.img`

```
fastboot flash recovery twrp-3.4.0-0-hammerhead.img
```

![image-20210919152200104](upload/attach/202205/905443\_PTP5YTG7G8Q65Q4.png)

然后我们就可以进入recovery模式了

然后我们将Supersu拷贝进去，然后将Xposed-v89-sdk.zip拷贝进去

![image-20210919152200104](upload/attach/202205/905443\_USKZ6Q73UJ67BPY.png)
