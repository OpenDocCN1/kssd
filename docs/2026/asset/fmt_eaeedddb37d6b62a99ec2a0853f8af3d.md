
# FART：ART环境下基于主动调用的自动化脱壳方案

## 一、App启动流程

这里以一张图说明App进程的创建流程：

![](https://bbs.kanxue.com/upload/attach/201907/632473_CAEZ8FUKNQU35VF.png)

通过`Zygote`进程到最终进入到app进程世界，我们可以看到`ActivityThread.main()`是进入App世界的大门，下面对该函数体进行简要的分析，具体分析请看文末的参考链接。

```
5379    public static void main(String[] args) {

5380        Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "ActivityThreadMain");

5381        SamplingProfilerIntegration.start();

5382

5383        // CloseGuard defaults to true and can be quite spammy.  We

5384        // disable it here, but selectively enable it later (via

5385        // StrictMode) on debug builds, but using DropBox, not logs.

5386        CloseGuard.setEnabled(false);

5387

5388        Environment.initForCurrentUser();

5389

5390        // Set the reporter for event logging in libcore

5391        EventLogger.setReporter(new EventLoggingReporter());

5392

5393        AndroidKeyStoreProvider.install();

5394

5395        // Make sure TrustedCertificateStore looks in the right place for CA certificates

5396        final File configDir = Environment.getUserConfigDirectory(UserHandle.myUserId());

5397        TrustedCertificateStore.setDefaultUserDirectory(configDir);

5398

5399        Process.setArgV0("<pre-initialized>");

5400

5401        Looper.prepareMainLooper();

5402

5403        ActivityThread thread = new ActivityThread();

5404        thread.attach(false);

5405

5406        if (sMainThreadHandler == null) {

5407            sMainThreadHandler = thread.getHandler();

5408        }

5409

5410        if (false) {

5411            Looper.myLooper().setMessageLogging(new

5412                    LogPrinter(Log.DEBUG, "ActivityThread"));

5413        }

5414

5415        // End of event ActivityThreadMain.

5416        Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);

5417        Looper.loop();

5418

5419        throw new RuntimeException("Main thread loop unexpectedly exited");

5420    }

5421}
```

对于`ActivityThread`这个类，其中的`sCurrentActivityThread`静态变量用于全局保存创建的`ActivityThread`实例，同时还提供了`public static ActivityThread currentActivityThread()`静态函数用于获取当前虚拟机创建的`ActivityThread`实例。`ActivityThread.main()`函数是java中的入口main函数,这里会启动主消息循环，并创建`ActivityThread`实例，之后调用`thread.attach(false)`完成一系列初始化准备工作，并完成全局静态变量`sCurrentActivityThread`的初始化。之后主线程进入消息循环，等待接收来自系统的消息。当收到系统发送来的`bindapplication`的进程间调用时，调用函数`handlebindapplication`来处理该请求。

```
private void handleBindApplication(AppBindData data) {

    //step 1: 创建LoadedApk对象

    data.info = getPackageInfoNoCheck(data.appInfo, data.compatInfo);

    ...

    //step 2: 创建ContextImpl对象;

    final ContextImpl appContext = ContextImpl.createAppContext(this, data.info);



    //step 3: 创建Instrumentation

    mInstrumentation = new Instrumentation();



    //step 4: 创建Application对象;在makeApplication函数中调用了newApplication，在该函数中又调用了app.attach(context)，在attach函数中调用了Application.attachBaseContext函数

    Application app = data.info.makeApplication(data.restrictedBackupMode, null);

    mInitialApplication = app;



    //step 5: 安装providers

    List<ProviderInfo> providers = data.providers;

    installContentProviders(app, providers);



    //step 6: 执行Application.Create回调

    mInstrumentation.callApplicationOnCreate(app);
```

在`handleBindApplication`函数中第一次进入了app的代码世界，该函数功能是启动一个application，并把系统收集的apk组件等相关信息绑定到application里，在创建完application对象后，接着调用了application的`attachBaseContext`方法，之后调用了application的`onCreate`函数。由此可以发现，app的`Application`类中的`attachBaseContext`和`onCreate`这两个函数是最先获取执行权进行代码执行的。这也是为什么各家的加固工具的主要逻辑都是通过替换app入口`Application`，并自实现这两个函数，在这两个函数中进行代码的脱壳以及执行权交付的原因。

## 二、APP加壳原理以及运行流程

在第一节的App启动流程中我们最终可以得出结论，app最先获得执行权限的是app中声明的`Application`类中的`attachBaseContext`和`onCreate`函数。因此，壳要想完成应用中加固代码的解密以及应用执行权的交付就都是在这两个函数上做文章。下面这张图大致讲了加壳应用的运行流程。

![](https://bbs.kanxue.com/upload/attach/201907/632473_3B54U89SWTXUKRE.png)

当壳在函数`attachBaseContext`和`onCreate`中执行完加密的dex文件的解密后，通过自定义的`Classloader`在内存中加载解密后的dex文件。为了解决后续应用在加载执行解密后的dex文件中的`Class`和`Method`的问题，接下来就是通过利用java的反射修复一系列的变量。其中最为重要的一个变量就是应用运行中的`Classloader`，只有`Classloader`被修正后，应用才能够正常的加载并调用dex中的类和方法，否则的话由于`Classloader`的双亲委派机制，最终会报`ClassNotFound`异常，应用崩溃退出，这是加固厂商不愿意看到的。由此可见`Classloader`是一个至关重要的变量，所有的应用中加载的dex文件最终都在应用的`Classloader`中。

因此，只要获取到加固应用最终通过反射设置后的`Classloader`，我们就可以通过一系列反射最终获取到当前应用所加载的解密后的内存中的Dex文件。

随着加壳技术的发展，为了对抗dex整体加固更易于内存dump来得到原始dex的问题，各加固厂商又结合hook技术，通过hook dex文件中类和方法加载执行过程中的关键流程，来实现在函数执行前才进行解密操作的指令抽取的解决方案。此时，就算是对内存中的dex整体进行了dump，但是由于其方法的最为重要的函数体中的指令被加密，导致无法对相关的函数进行脱壳。由此，`Fupk3`诞生了，该脱壳工具通过欺骗壳而主动调用dex中的各个函数，完成调用流程，让壳主动解密对应method的指令区域，从而完成对指令抽取型壳的脱壳。

## 三、现有ART环境下自动化脱壳工具及优缺点

针对虚拟机运行过程中类的加载执行流程进行修改从而完成脱壳的集大成者算是`dexhunter`。`dexhunter`分别实现了dalvik和art环境下的加固app的脱壳。然而，当前针对`dexhunter`的脱壳原理来对抗`dexhunter`的技术也不断被应用，比如添加无效类并在这些类的初始化函数加入强制退出相关的代码以及检测`dexhunter`的配置文件等。其次，当前ART环境下的脱壳技术还有基于`dex2oat`编译生成oat过程的内存中的dex的dump技术，该方法依然是整体型dump，无法应对指令抽取型加固,同时，当前一些壳对于动态加载dex的流程进行了hook，这些dex也不会走`dex2oat`流程；以及基于dex加载过程中内存中的`DexFile`结构体的dump技术。例如，在ART下通过hook `OpenMem`函数来实现在壳进行加载`DexFile`时对内存中的dex的dump的脱壳技术，以及在2017年的DEF CON 25黑客大会中，Avi Bashan和SlavaMakkaveev提出的通过修改`DexFile`的构造函数`DexFile::DexFile()`，以及`OpenAndReadMagic()`函数来实现对加壳应用的内存中的dex的dump来脱壳技术。上面这些脱壳技术均无法实现对指令抽取型壳的完全脱壳。与此同时，`F8left`实现并开源了Dalvik环境下的基于主动调用的脱壳技术，完美实现了对抗指令抽取型壳的解决方案。但是随着Android的升级，Dalvik虚拟机已经逐渐淡出了视野，当前的很多应用已经不支持安装在4.4以下系统中，这就导致`fupk3`也即将走向末路。相信`F8left`大佬也早已实现了ART环境下的基于主动调用的脱壳技术，但是却由于某些原因并未开源。本人在前人基础上，提出一种ART环境下的基于主动调用的的脱壳技术解决方案，并最后实现该解决方案，这里先分别提供arm模拟器、x86模拟器以及nexus5的脱壳镜像供广大安全研究人员测试使用(建议手头有nexus5手机的用户选择使用nexus5镜像，虚拟机下使用较慢)，并提供意见和建议，也欢迎大家共同参与到我的工作中，对`Fart`进行进一步的完善。在后续待更完善后会将该项目开源。

## 四、FART脱壳原理以及实现

FART脱壳的步骤主要分为三步：

1. 内存中`DexFile`结构体完整dex的dump
2. 主动调用类中的每一个方法，并实现对应`CodeItem`的dump
3. 通过主动调用dump下来的方法的`CodeItem`进行dex中被抽取的方法的修复

下面分别对每一步的实现原理进行介绍。

### 1. 内存中DexFile结构体完整dex的dump

该步同Avi Bashan和SlavaMakkaveev在DefCon 2017上提出的通过修改`DexFile`的构造函数`DexFile::DexFile()`，以及`OpenAndReadMagic()`函数来实现对加壳应用的内存中的dex的dump来脱壳的原理类似。不同之处在于Avi Bashan和SlavaMakkaveev是通过修改系统中`DexFile`中提供的相关函数来实现dump，实际上壳完全可以自实现一套Dex文件的内存加载机制从而绕过这种dump方法。本文提出的是通过选择合适的时机点获取到应用解密后的dex文件最终依附的`Classloader`，进而通过java的反射机制最终获取到对应的`DexFile`的结构体，并完成dex的dump。接下来主要介绍具体实现细节。

首先，对于获取`Classloader`的时机点的选择。在第一节的App启动流程以及第三节中APP加壳原理和执行流程的介绍中，可以看到，APP中的`Application`类中的`attachBaseContext`和`onCreate`函数是app中最先执行的方法。壳都是通过替换APP的`Application`类并自己实现这两个函数，并在这两个函数中实现dex的解密加载，hook系统中`Class`和`method`加载执行流程中的关键函数，最后通过反射完成关键变量如最终的`Classloader`，`Application`等的替换从而完成执行权的交付。因此，我们可以选在任意一个在`Application`的`onCreate`函数执行之后才开始被调用的任意一个函数中。众所周知，对于一个正常的应用来说，最终都要由一个个的`Activity`来展示应用的界面并和用户完成交互，那么我们就可以选择在`ActivityThread`中的`performLaunchActivity`函数作为时机，来获取最终的应用的`Classloader`。选择该函数还有一个好处在于该函数和应用的最终的application同在`ActivityThread`类中，可以很方便获取到该类的成员。

```
private Activity performLaunchActivity(ActivityClientRecord r, Intent customIntent) {

        ......

       

Activity activity = null;

        try {

            java.lang.ClassLoader cl = r.packageInfo.getClassLoader();

//下面通过application的getClassLoader()获取最终的Classloader，并开启线程，在新线程中完成内存中的dex的dump以及主动调用过程，由于该过程相对耗时，为了防止应用出现ANR，从而开启新线程，在新线程中进行，主要的工作都在getDexFilesByClassLoader_23

            //addstart

            packagename=r.packageInfo.getPackageName();

            //mInitialApplication

            //final java.lang.ClassLoader finalcl=cl

            if(mInitialApplication!=null){

		    final java.lang.ClassLoader finalcl=mInitialApplication.getClassLoader();

            new Thread(new Runnable() {

            @Override

            public void run() {              

								getDexFilesByClassLoader_23(finalcl);

                            }

                        }).start();



				}

           

            //addend

          }

}
```

`getDexFilesByClassLoader_23()`函数的主要流程就是通过一系列的反射，最终获取到当前`Classloader`中的`mCookie`，即Native层中的`DexFile`。为了在C/C++中完成对dex的dump操作。这里我们在framework层的`DexFile`类中添加两个Native函数供调用：

在文件`libcore/dalvik/src/main/java/dalvik/system/DexFile.java`中

```
private static native void dumpDexFile(String dexfilepath,Object cookie);

private static native void dumpMethodCode(String eachclassname, String methodname,Object cookie, Object method);
```

这两个函数分别用于完成内存中dex的dump以及构造主动调用链，完成方法体的dump。

在对应的c++文件中添加这两个Native函数的实现并完成注册：
