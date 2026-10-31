
# [实践] Android5.1.1源码 - 让某个APP以解释执行模式运行

**作者：寻禹@阿里聚安全**

## 前言

本文的实践修改了Android5.1.1的源码。

本文只简单的讲了一下原理。在“实践”一节讲了具体做法。

本文的内容涉及Art模式下dex加载的知识，想要详细了解这部分知识可以去看老罗的文章：

[Android运行时ART简要介绍和学习计划](https://bbs.kanxue.com/elink@944K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6Q4x3X3g2U0M7$3c8F1i4K6u0W2L8X3g2@1i4K6u0r3L8s2g2G2M7$3S2W2L8X3N6&6j5h3&6Y4i4K6u0r3j5i4u0@1K9h3y4D9k6g2)9J5c8X3c8W2N6r3q4A6L8s2y4Q4x3V1j5K6z5e0t1#2y4U0R3I4x3#2)9K6c8Y4y4H3L8g2)9K6c8r3p5K6x3e0y4W2i4K6u0W2y4K6V1I4y4U0j5@1z5q4)9J5k6e0m8Q4x3X3f1H3i4K6u0W2c8X3Z5J5K9i4x3I4)

[Android运行时ART加载OAT文件的过程分析](https://bbs.kanxue.com/elink@35fK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6Q4x3X3g2U0M7$3c8F1i4K6u0W2L8X3g2@1i4K6u0r3L8s2g2G2M7$3S2W2L8X3N6&6j5h3&6Y4i4K6u0r3j5i4u0@1K9h3y4D9k6g2)9J5c8X3c8W2N6r3q4A6L8s2y4Q4x3V1j5K6z5e0x3H3y4K6R3I4x3#2)9K6c8Y4y4H3L8g2)9K6c8r3p5K6x3e0y4W2i4K6u0W2y4K6V1I4y4U0j5@1z5q4)9J5k6e0m8Q4x3X3f1H3i4K6u0W2c8X3Z5J5K9i4x3I4)

[Android运行时ART加载类和方法的过程分析](https://bbs.kanxue.com/elink@c9aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6Q4x3X3g2U0M7$3c8F1i4K6u0W2L8X3g2@1i4K6u0r3L8s2g2G2M7$3S2W2L8X3N6&6j5h3&6Y4i4K6u0r3j5i4u0@1K9h3y4D9k6g2)9J5c8X3c8W2N6r3q4A6L8s2y4Q4x3V1j5K6z5e0f1K6x3K6f1H3x3#2)9K6c8Y4y4H3L8g2)9K6c8r3p5K6x3e0y4W2i4K6u0W2y4K6V1I4y4U0j5@1z5q4)9J5k6e0m8Q4x3X3f1H3i4K6u0W2c8X3Z5J5K9i4x3I4)

[Android运行时ART执行类方法的过程分析](https://bbs.kanxue.com/elink@c9cK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6Q4x3X3g2U0M7$3c8F1i4K6u0W2L8X3g2@1i4K6u0r3L8s2g2G2M7$3S2W2L8X3N6&6j5h3&6Y4i4K6u0r3j5i4u0@1K9h3y4D9k6g2)9J5c8X3c8W2N6r3q4A6L8s2y4Q4x3V1j5@1x3o6t1^5z5e0b7H3y4g2)9K6c8Y4y4H3L8g2)9K6c8r3p5K6x3e0y4W2i4K6u0W2y4K6V1I4y4U0j5@1z5q4)9J5k6e0m8Q4x3X3f1H3i4K6u0W2c8X3Z5J5K9i4x3I4)

本文的内容涉及zygote，如果不知道zygote是什么，或者好奇zygote如何启动，可以去看老罗的文章：

[Android系统进程Zygote启动过程的源代码分析](https://bbs.kanxue.com/elink@1a2K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6Q4x3X3g2U0M7$3c8F1i4K6u0W2L8X3g2@1i4K6u0r3L8s2g2G2M7$3S2W2L8X3N6&6j5h3&6Y4i4K6u0r3j5i4u0@1K9h3y4D9k6g2)9J5c8X3c8W2N6r3q4A6L8s2y4Q4x3V1j5$3y4K6j5^5x3K6l9@1i4K6y4r3M7%4m8E0i4K6y4p5j5e0x3I4x3$3g2Q4x3X3f1%4z5e0p5$3y4U0b7^5i4K6u0W2x3q4)9J5k6e0m8Q4x3X3g2r3K9U0u0A6M7K6p5`.)

老罗的文章分析的是Android2.3的源码，所以下面提到的与zygote有关的函数在老罗的文章里面可能没有，如果想要对下面提到的与zygote有关的函数有一个简单的了解可以看我的文章：[Android5.1.1源码 - zygote fork出的子进程如何权限降级](https://bbs.kanxue.com/elink@afbK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6B7j5i4q4Q4x3X3g2S2L8r3W2T1j5h3u0S2i4K6u0W2j5$3!0E0i4K6u0r3j5$3!0E0L8i4g2F1K9i4c8&6i4K6u0r3j5i4u0@1i4K6u0r3M7$3S2G2N6#2)9K6c8Y4y4H3L8g2)9K6c8r3p5K6x3e0y4W2i4K6u0W2y4K6V1I4y4U0j5@1x3W2)9J5k6e0m8Q4x3X3f1H3i4K6u0W2f1g2g2A6y4V1u0h3i4K6t1$3j5i4u0@1K9h3y4D9k6h3W2V1i4K6y4p5x3K6l9$3)

## 原理简介

### 怎么才能让方法解释执行

在函数`ClassLinker::LinkCode`中会链接dex中的方法代码，这个函数的定义在文件"`art/runtime/class_linker.cc`"中，下面是它的源码（这里只列出了与本文有关的部分）：

在 `ClassLinker::LinkCode` 中调用了 `NeedsInterpreter` 函数判断当前方法是否要解释执行，如果返回值为 `true`，即局部变量 `enter_interpreter` 被赋值为 `true`，那么调用 `ArtMethod` 类中的 `SetEntryPointFromQuickCompiledCode` 函数并将 `GetQuickToInterpreterBridge()` 的返回值传入，`GetQuickToInterpreterBridge()` 函数返回用于解释执行的函数的入口地址，这个入口函数解释执行dex中的方法。

那么现在来看看 `NeedsInterpreter` 函数，这个函数的定义在文件 "`art/runtime/class_linker.cc`" 中，下面是它的源码：

当 "`Runtime::Current()->GetInstrumentation()->InterpretOnly()`" 返回 `true` 且不是本地方法和代理方法，那么这个函数就会返回 `true`，否则返回 `false`。

`InterpretOnly` 函数是 `Instrumentation` 类的成员函数，它的函数定义在文件 "`art/runtime/instrumentation.h`" 中，下面是它的源码：

`interpret_only_` 是类 `Instrumentation` 的成员变量，是布尔类型。可以发现 `InterpretOnly` 函数仅仅是将 "`interpret_only_`" 返回，如果将 `interpret_only_` 设置为 `true`，那么根据上文分析，所有“非本地且非代理”方法都将被解释执行。

那么如何将 `interpret_only_` 设置为 `true` 哪，在 `Instrumentation` 类中有一个 `ForceInterpretOnly` 函数，下面是这个函数的源码：

这个函数是 `Instrumentation` 类的公有成员函数，所以直接调用这个函数即可将 `interpret_only_` 设置为 `true`。

这里有一个问题，将 `interpret_only_` 设置为 `true`，那么“非本地且非代理”方法在链接代码时都将被设置成解释执行，那么会不会影响到其他的APP进程？不会，因为 `ClassLinker::LinkCode` 函数对方法的链接是在APP进程的内存中进行的，所以这个操作并不会影响到其他进程。

这里进行一个小节，当执行 "`Runtime::Current()->GetInstrumentation()->ForceInterpretOnly()`" 语句时，会把 `Instrumentation` 对象的 `interpret_only_` 成员变量设置为 `true`。那么当方法是“非本地且非代理”方法时，`NeedsInterpreter` 函数将返回 `true`，那么在 `ClassLinker::LinkCode` 函数中会将这个方法设置为解释执行。

如果要将APP中所有方法都设置为解释执行，那么就需要在链接APP的dex中的方法代码之前执行 "`Runtime::Current()->GetInstrumentation()->ForceInterpretOnly()`" 语句。

### 调用ForceInterpretOnly函数的时机

我的办法是在 `EnableDebugFeatures` 函数中调用 `ForceInterpretOnly` 函数，在这一节中会先说明Android如何执行到 `EnableDebugFeatures` 函数，然后会说明在 `EnableDebugFeatures` 函数中调用 `ForceInterpretOnly` 函数的好处。

所有的Android应用进程都是zygote fork出来的，fork APP进程时的函数调用路径：

调用完 `ForkAndSpecializeCommon` 函数后APP进程就被fork出来了。

`ForkAndSpecializeCommon` 函数定义在文件 "`frameworks/base/core/jni/com_android_internal_os_Zygote.cpp`" 中，下面它的源码：

`gCallPostForkChildHooks` 是一个全局变量，它在 `com_android_internal_os_Zygote.cpp` 文件的 `register_com_android_internal_os_Zygote` 函数中被初始化。

`env->CallStaticVoidMethod(...)` 语句调用了Java方法 "`Zygote.callPostForkChildHooks`"。

下面是 `Zygote.callPostForkChildHooks` 方法的源码，这个方法在文件 "`frameworks/base/core/java/com/android/internal/os/Zygote.java`" 中：

`VM_HOOKS` 是 `Zygote` 类的成员变量，下面是它的定义：

`VM_HOOKS.postForkChild` 调用的就是 `ZygoteHooks` 类中的成员方法 `postForkChild`，这个方法在文件 "`libcore/dalvik/src/main/java/dalvik/system/ZygoteHooks.java`" 中，下面是它的源码：

这个方法中调用了native函数 `nativePostForkChild`，`nativePostForkChild` 函数的C层代码在文件 "`/home/sowuy/android/system/art/runtime/native/dalvik_system_ZygoteHooks.cc`" 中，下面是它的源码：

我将在 `EnableDebugFeatures` 函数中调用 `ForceInterpretOnly` 函数，原因有三点：

`EnableDebugFeatures` 函数参数接收的是一个标志，我可以设置一个新的标志位用来表示是否需要调用 `ForceInterpretOnly` 函数。

每次APP启动的时候都会执行 `EnableDebugFeatures` 函数。

`EnableDebugFeatures` 函数被调用的时机好，它运行在fork出的APP进程中，并且在链接APP的dex中的方法前被调用。

## 实践

### 修改Zygote.java中的代码

`Zygote.java` 文件的位置是：`frameworks/base/core/java/com/android/internal/os/Zygote.java`，在 `Zygote` 类中添加一个成员变量：

在 `Zygote` 类 `forkAndSpecialize` 方法的开始部分添加下面的代码：

### 修改dalvik_system_ZygoteHooks.cc中的代码

`dalvik_system_ZygoteHooks.cc` 文件的位置是：`art/runtime/native/dalvik_system_ZygoteHooks.cc`，修改这个文件中的 `EnableDebugFeatures` 函数的代码。

向这个函数中添加下面的代码：

下面是对这个函数修改后的完整代码：

作者：寻禹@阿里聚安全，更多安全技术文章，请访问[阿里聚安全博客](https://bbs.kanxue.com/elink@891K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6B7j5i4q4Q4x3X3g2S2L8r3W2T1j5h3u0S2i4K6u0W2j5$3!0E0i4K6u0r3j5$3!0E0L8i4g2F1K9i4c8&6i4K6u0r3K9h3&6V1k6i4R3`.)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
