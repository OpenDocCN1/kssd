
# Unidbg-Linker 部分源码分析（上）

- [Unidbg-Linker部分源码分析（上）](https://bbs.kanxue.com/#unidbg-linker部分源码分析上)
  - [概要](https://bbs.kanxue.com/#概要)
  - [init](https://bbs.kanxue.com/#init)
  - [load](https://bbs.kanxue.com/#load)
  - [总结](https://bbs.kanxue.com/#总结)

## 概要

我们有了 Android Linker 源码部分的解析，还学习了 Unicorn 的详细使用。如果没有看过之前文章的同学可以看下之前的文章哦。今天我们就来看下 Unidbg 是如何将一个 So 加载且跑起来的。

在 Unidbg 中，我们想要加载一个 So，一般都是通过：

```java
DalvikModule dalvikModule = vm.loadLibrary(new File("so_path"), true);
```

来加载一个 So 文件。通过这个方法，我们最终会发现 Unidbg 调用了 `AndroidElfLoader` 类的 `loadInternal` 方法。`AndroidElfLoader` 类也就充当了 Linker 的角色，所以我们今天就来分析一下这个类。

## init

我们先来分析一下 `AndroidElfLoader` 的构造方法：

```java
public AndroidElfLoader(Emulator<AndroidFileIO> emulator, UnixSyscallHandler<AndroidFileIO> syscallHandler) {
    // 调用父类构造方法，初始化emulator和syscallHandler字段
    super(emulator, syscallHandler);

    // 初始化SP
    stackSize = STACK_SIZE_OF_PAGE * emulator.getPageAlign();

    // 将栈空间mem_map映射，因为在Backend(Unicorn)中，所有需要用的内存都需要先进行映射才能够进行使用，大小就是STACK_SIZE_OF_PAGE * emulator.getPageAlign()，可读可写权限
    backend.mem_map(STACK_BASE - stackSize, stackSize, UnicornConst.UC_PROT_READ | UnicornConst.UC_PROT_WRITE);

    // 设置SP寄存器
    setStackPoint(STACK_BASE);

    // 初始化TLS(线程局部存储相关)，在libc一些系统库中是有线程局部变量的，如errno等。这里就做了相关的协处理器的初始化操作
    this.environ = initializeTLS(new String[] {
        "ANDROID_DATA=/data",
        "ANDROID_ROOT=/system"
    });
    this.setErrno(0);
}
```

上面的注释也写得很清楚了，再总结一下 `AndroidElfLoader` 初始化做了什么：

- 在类中保存了 `emulator` 和 `syscallHandler` 字段，其中 `emulator` 就是一个模拟器对象。`syscallHandler` 是 Linux 系统调用相关的处理对象。
- 初始化了栈空间和设置 SP 寄存器。
- 初始化 TLS 操作。

对 TLS 感兴趣可以阅读以下源码：

> 406K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3q4F1k6s2u0G2K9h3c8^5M7X3g2X3i4K6u0W2j5$3!0E0i4K6u0r3y4q4)9J5k6e0c8Q4x3X3f1@1i4K6g2X3M7U0q4Q4x3V1k6^5M7X3g2X3i4K6u0r3j5X3W2G2L8X3W2U0i4K6u0r3L8r3W2T1j5#2)9J5c8X3u0A6L8$3&6A6j5#2)9J5c8X3I4A6j5X3y4Q4y4h3k6A6L8X3W2@1i4K6g2X3j5$3!0E0L8h3!0F1i4K6u0W2j5%4m8H3i4K6t1K6x3e0p5I4

> d1aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3q4F1k6s2u0G2K9h3c8^5M7X3g2X3i4K6u0W2j5$3!0E0i4K6u0r3y4q4)9J5k6e0c8Q4x3X3f1@1i4K6g2X3M7U0q4Q4x3V1k6^5M7X3g2X3i4K6u0r3j5X3W2G2L8X3W2U0i4K6u0r3L8r3W2T1j5#2)9J5c8X3u0A6L8$3&6A6j5#2)9J5c8X3I4A6j5X3y4Q4y4h3k6A6L8X3W2@1i4K6g2X3j5$3!0E0L8h3!0F1i4K6u0W2j5%4m8H3i4K6t1K6x3e0p5I4

> 7d2K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3q4F1k6s2u0G2K9h3c8^5M7X3g2X3i4K6u0W2j5$3!0E0i4K6u0r3y4q4)9J5k6e0c8Q4x3X3f1@1i4K6g2X3M7U0q4Q4x3V1k6^5M7X3g2X3i4K6u0r3j5X3W2G2L8X3W2U0i4K6u0r3L8r3W2T1j5#2)9J5c8Y4m8J5K9i4k6S2N6r3g2Q4x3V1k6T1K9h3!0F1K9h3y4Q4y4h3k6@1L8s2y4Q4x3X3g2Z5i4K6t1K6z5e0l9`.

## load

那么当我们调用了 `loadLibrary` 方法后，在 Android 中最终调用了 `AndroidElfLoader` 类下的 `loadInternal` 方法，我们来分析这个方法，该类的初始化我们已经分析过了。

```java
protected final LinuxModule loadInternal(LibraryFile libraryFile, boolean forceCallInit) {
    // File对象被封装为了LibraryFile对象
    try {
        // 接着调用了loadInternal(重载)方法，继续加载流程
        LinuxModule module = loadInternal(libraryFile);

        // 处理符号(关于重定位)
        resolveSymbols(!forceCallInit);

        // callInitFunction默认true
        if (callInitFunction || forceCallInit) {
            // 调用初始化函数
            for (LinuxModule m : modules.values().toArray(new LinuxModule[0])) {
                boolean forceCall = (forceCallInit && m == module) || m.isForceCallInit();
                if (callInitFunction) {
                    m.callInitFunction(emulator, forceCall);
                } else if (forceCall) {
                    m.callInitFunction(emulator, true);
                }
                m.initFunctionList.clear();
            }
        }

        // 添加引用计数
        module.addReferenceCount();
        return module;
    } catch (IOException e) {
        throw new IllegalStateException(e);
    }
}
```

上面的函数接着调用了重载 `loadInternal` 方法继续加载，加载完成会返回一个 `LinuxModule` 对象，该对象就保存了该 So 文件加载后的信息，接着处理了符号和调用初始化函数。

我们接着来看重载 `loadInternal` 方法，代码比较长。

## 总结

我们分为上下两篇文章。我们看到上面这个 `loadInternal` 方法就是加载一个 So 主要的内容，大致内容看到这里其实已经够用了，如果想了解更多的细节就看下篇。其中的很多方法我们没有展开来讲解，接下来我们就来处理剩余的细枝末节。如果文章有错误的地方还请指正，可以加个 VX 一起交流：roy5ue 或直接在文章下方进行评论哦。
