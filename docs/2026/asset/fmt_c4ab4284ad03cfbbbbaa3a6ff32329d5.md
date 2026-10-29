
# Android 系统架构、启动流程与 Xposed 框架

## 1. Android 系统架构

### Android Runtime

Android 5.0 之前 Android Runtime 为 Dalvik，Android 5.0 之后 Android Runtime 为 ART。

首先我们先了解一些文件的含义：

1. `dex` 文件：Android 将所有的 `class` 文件打包形成一个 `dex` 文件，是 Dalvik 运行的程序。
2. `odex` 文件：优化过的 `dex` 文件，APK 在安装时会进行验证和优化，通过 `dexopt` 生成 `odex` 文件，加快 APK 的响应时间。
3. `oat` 文件：Android 私有 ELF 文件格式，由 `dex2oat` 处理生成，包含（原 `dex` 文件 + `dex` 翻译的本地机器指令），是 ART 虚拟机使用的文件，可以直接加载。
4. `vdex` 文件：Android 8.0 引入，包含 APK 的未压缩 `DEX` 代码，以及一些旨在加快验证速度的元数据。

下面我们从 Android 系统的发展过程中详细介绍二者的区别：

| 版本 | 虚拟机类型 | 特性 |
| --- | --- | --- |
| 2.1-4.4 | Dalvik | JIT + 解释器 |
| 5.0-7.0 | ART | AOT |
| 7.0-11 | ART | AOT + JIT + 解释器 |

下面部分参考博客：[博客地址](<https://bbs.kanxue.com/elink@ab9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6B7N6h3g2B7K9h3&6Q4x3X3g2U0L8W2)9J5c8Y4m8G2M7%4c8Q4x3V1j5$3z5o6b7@1z5e0l9K6y4K6b7^5x3o6f1^5x3U0p5^5y4e0l9&6>)

#### Android 2.2（Dalvik）

- 支持已转换成 `dex` 格式的 Android 应用，基于寄存器，指令执行更快，加载的是 `odex` 文件，采用 JIT 运行时编译。

#### JIT

JIT 即运行时编译策略，可以理解成一种运行时编译器。此时 Android 的虚拟机使用的是 Dalvik，为了加快 Dalvik 虚拟机解释 `dex` 速度，运行时动态地将执行频率很高的 `dex` 字节码翻译成本地机器码。

缺点：

1. 每次启动应用都需要重新编译。
2. 运行时比较耗电，造成电池额外的开销。

![image-20211002145934512](https://bbs.kanxue.com/upload/attach/202110/905443_NWK43FAQMD83H2G.png)

![image-20211003100041210](https://bbs.kanxue.com/upload/attach/202110/905443_AF57PEUV7DXGFB9.png)

基于 Dalvik 的虚拟机，在 APK 安装时会对 `dex` 文件进行优化，产生 `odex` 文件。启动 APP 后，运行时会利用 JIT 即时编译，处理执行频率高的一部分 `dex`，将其翻译成机器码，这样在再次调用的时候就可以直接运行机器码，从而提高了 Dalvik 翻译的速率，提高运行速度。

缺点：

1. 由于在 Dex 加载时会触发 `dexopt`，导致 Multidex 加载的时候会非常慢。
2. 由于热点代码的 Monitor 一直在运行，解释器解释的字节码会带来 CPU 和时间的消耗，会带来电量的损耗。

#### Android 4.4——ART 和 AOT

此时引入全新的虚拟机运行环境 ART 和全新的编译策略 AOT，此时 ART 和 Dalvik 是共存的，用户可以在两者之间选择。

#### Android 5.0——ART 全面取代 Dalvik

##### AOT

AOT 是一种运行前编译的策略。

缺点：

1. 应用安装和系统升级之后的应用优化比较耗时。
2. 优化后的文件会占用额外的存储空间。

##### AOT 与 JIT 区别

- JIT 是在运行时进行编译，是动态编译，并且每次运行程序的时候都需要对 `odex` 重新进行编译。
- AOT 是静态编译，应用在安装的时候会启动 `dex2oat` 过程把 `dex` 预编译成 ELF 文件，每次运行程序的时候不用重新编译，是真正意义上的本地应用。

##### JVM、Dalvik 和 ART 区别

- JVM：传统的 Java 虚拟机、基于栈、运行 `class` 文件。
- Dalvik：支持已转换成 `dex` 格式的 Android 应用，基于寄存器，指令执行更快，加载的是 `odex`（优化的 `dex`）。
- ART：第一次安装时，将 `dex` 进行 AOT（预编译），字节码预先编译成机器码，生成可执行 `oat` 文件（ELF 文件）。

![image-20211002154534641](https://bbs.kanxue.com/upload/attach/202110/905443_MNJJBGKSUG3AD9G.png)

![image-20211003101124310](https://bbs.kanxue.com/upload/attach/202110/905443_WATDYB6PWZH9AZD.png)

基于 ART 的虚拟机，会在 APK 第一次安装时，将 `dex` 进行 AOT（预编译），通过 `dex2oat` 生成 `oat` 文件，即 Android 可执行 ELF 文件，包括原 `dex` 文件和翻译后的机器码，然后启动程序后直接运行。

缺点：

1. 由于安装 APK 时触发 `dex2oat`，需要编译成 native code，导致安装时间过长。
2. 由于 `dex2oat` 生成的文件较大，会占用较多的空间。

#### Android 7.0——JIT 回归

考虑上面 AOT 的缺点，`dex2oat` 过程比较耗时且会占用额外的存储空间，Android 7.0 再次加入 JIT，形成 `AOT + JIT + 解释器` 模式。

特点：

1. 应用在安装的时候 `dex` 不会被编译。
2. 应用在运行时 `dex` 文件先通过解析器（Interpreter）后会被直接执行，与此同时，热点函数（Hot Code）会被识别并被 JIT 编译后存储在 `jit code cache` 中并生成 `profile` 文件以记录热点函数的信息。
3. 手机进入 IDLE（空闲）或者 Charging（充电）状态的时候，系统会扫描 App 目录下的 `profile` 文件并执行 AOT 过程进行编译。

混合编译模式综合了 AOT 和 JIT 的各种优点，使得应用在安装速度加快的同时，运行速度、存储空间和耗电量等指标都得到了优化。

![image-20211003102555392](https://bbs.kanxue.com/upload/attach/202110/905443_MEEBZXF5JUW7G3Z.png)

最后我们可以看下 Android 各版本 `ClassLoader` 加载 `dex` 时的 `dexopt` 过程：

![image-20211003102844820](https://bbs.kanxue.com/upload/attach/202110/905443_U5HEESE5529NAVC.png)

### 原生 C/C++ 库

许多核心 Android 系统组件和服务（例如 ART 和 HAL）构建自原生代码，需要以 C 和 C++ 编写的原生库。Android 平台提供 Java 框架 API 以向应用显示其中部分原生库的功能，我们可以通过 NDK 开发 Android 中的 C/C++ 库。

### Java API 框架

通过以 Java 语言编写的 API 使用 Android OS 的整个功能集。这些 API 形成创建 Android 应用所需的构建块，它们可简化核心模块化系统组件和服务的重复使用，包括以下组件和服务：

## 2. Android 启动流程

Android 启动流程如下图所示：

![image-20211003102844820](https://bbs.kanxue.com/upload/attach/202110/905443_ZVX94URZK2CY3XG.png)

### Loader

- Boot ROM: 当手机处于关机状态时，长按 Power 键开机，引导芯片开始从固化在 ROM 里的预设代码开始执行，然后加载引导程序到 RAM。
- Boot Loader：这是启动 Android 系统之前的引导程序，主要是检查 RAM，初始化硬件参数，拉起 Android OS。

我们长按电源键后，手机就会在 Loader 层加载引导程序，并启动引导程序，初始化参数。

### Linux 内核

1. 启动 Kernel 的 `swapper` 进程（pid=0）：该进程又称为 `idle` 进程，系统初始化过程 Kernel 由无到有开创的第一个进程，用于初始化进程管理、内存管理，加载 Display、Camera Driver、Binder Driver 等相关工作，这些模块驱动都会封装到对应的 HAL 层中。
2. 启动 `init` 进程（用户进程的祖宗）。pid=1，用来孵化用户空间的守护进程、HAL、开机动画等。
3. 启动 `kthreadd` 进程（pid=2）：是 Linux 系统的内核进程，会创建内核工作线程 `kworker`，软中断线程 `ksoftirqd`，thermal 等内核守护进程。`kthreadd` 进程是所有内核进程的鼻祖。

### 执行 init 进程

`init` 进程是 Linux 系统中用户空间的第一个进程，进程号为 1，是所有用户进程的祖先。

Linux Kernel 完成系统设置后，会首先在系统中寻找 `init.rc` 文件，并启动 `init` 进程。`init.rc` 脚本存放路径：`/system/core/rootdir/init.rc`，`init` 进程：`/system/core/init`。

`init` 进程的启动可以分为三个部分：

1. `init` 进程会孵化出 `ueventd`、`logd`、`healthd`、`installd`、`adbd`、`lmkd` 等用户守护进程。
2. `init` 进程还会启动 `ServiceManager`（Binder 服务管家）、`bootanim`（开机动画）等重要服务。
3. 解析 `init.rc` 配置文件并孵化 `zygote` 进程。`Zygote` 进程是 Android 系统的第一个 Java 进程（虚拟机进程），`zygote` 进程是所有 Java 进程的父进程。

创建 Zygote 过程：

1. 解析 `init.zygote.rc` // `parse_service()`
2. 启动 main 类型服务 // `do_class_start()`
3. 启动 zygote 服务 // `service_start()`
4. 创建 Zygote 进程 // `fork()`
5. 创建 Zygote Socket // `create_socket()`

### Zygote

`Zygote` 为孵化器，即所有 Android 应用的祖先。`Zygote` 让 VM 共享代码、低内存占用以及最小的启动时间成为可能。`Zygote` 是一个虚拟机进程，由 `init` 进程通过解析 `init.zygote.rc` 文件而创建。`zygote` 所对应的可执行程序为 `app_process`，所对应的源文件是 `App_main.cpp`，进程名为 `zygote`。

`Zygote` 作用过程：

1. 解析 `init.zygote.rc` 中的参数，创建 `AppRuntime` 并调用 `AppRuntime.start()` 方法；
2. 调用 `AndroidRuntime` 的 `startVM()` 方法创建虚拟机，再调用 `startReg()` 注册 JNI 函数；
3. 通过 JNI 方式调用 `ZygoteInit.main()`，第一次进入 Java 世界；
4. `registerZygoteSocket()` 建立 socket 通道，`zygote` 作为通信的服务端，用于响应客户端请求；
5. `preload()` 预加载通用类、drawable 和 color 资源、OpenGL 以及共享库以及 `WebView`，用于提高 app 启动效率；
6. `zygote` 完成大部分工作，接下来再通过 `startSystemServer()`，fork 得力帮手 `system_server` 进程，也是上层 framework 的运行载体。
7. `zygote` 功成身退，调用 `runSelectLoop()`（死循环），随时待命，当接收到请求创建新进程请求时立即唤醒并执行相应工作。

Android 系统流程总结：

1. 手机开机后，引导芯片启动，引导芯片开始从固化在 ROM 里的预设代码执行，加载引导程序到 RAM，BootLoader 检查 RAM，初始化硬件参数等功能；
2. 硬件等参数初始化完成后，进入到 Kernel 层，Kernel 层主要加载一些硬件设备驱动，初始化进程管理等操作。在 Kernel 中首先启动 `swapper` 进程（pid=0），用于初始化进程管理、内存管理、加载 Driver 等操作，再启动 `kthreadd` 进程（pid=2），这些是 Linux 系统的内核进程，`kthreadd` 是所有内核进程的鼻祖；
3. Kernel 层加载完毕后，硬件设备驱动与 HAL 层进行交互。初始化进程管理等操作会启动 `init` 进程，这些在 Native 层中；
4. `init` 进程（pid=1，`init` 进程是所有进程的鼻祖，第一个启动）启动后，会启动 `adbd`、`logd` 等用户守护进程，并且会启动 `servicemanager`（binder 服务管家）等重要服务，同时孵化出 `zygote` 进程，这里属于 C++ Framework，代码为 C++ 程序；
5. `zygote` 进程是由 `init` 进程解析 `init.rc` 文件后 fork 生成，它会加载虚拟机，启动 System Server（`zygote` 孵化的第一个进程）；`SystemServer` 负责启动和管理整个 Java Framework，包含 `ActivityManager`、`WindowManager`、`PackageManager`、`PowerManager` 等服务；
6. `zygote` 同时会启动相关的 APP 进程，它启动的第一个 APP 进程为 `Launcher`，然后启动 Email、SMS 等进程，所有的 APP 进程都由 `zygote` fork 生成。

## 三、Xposed 框架运行机制

我们从上文中已经详细介绍了 Android 的启动流程，如下图所示：

![image-20211003102844820](https://bbs.kanxue.com/upload/attach/202110/905443_ZFPEBJFXVJ4CYER.png)

下面我们来详细介绍 Xposed 框架的实现原理：

### 1. Xposed 实现原理

我们先进入 [Xposed 官网](<https://bbs.kanxue.com/elink@cf9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6J5L8%4k6G2z5o6V1>)，可以发现 Xposed 工程的 5 个文件夹：

![image-20211003120209903](https://bbs.kanxue.com/upload/attach/202110/905443_AUVNU72XGRAXE49.png)

具体的模块功能和作用，我们在上文 [源码编译（2）——Xposed 源码编译详解](https://bbs.pediy.com/thread-269616.htm) 已经介绍了，大家可以去参考。

上文中我们知道，Xposed 集成到 Android 源码中主要是：

1. 替换了 Android 中的虚拟机 `art`。
2. 替换了 `app_process` 以及生成的一些 `lib`、`bin` 文件等。

具体如下图所示：

![image-20211003120719433](https://bbs.kanxue.com/upload/attach/202110/905443_96JUPDYZEX726AE.png)

我们从上文中可以得知 Android 所有的用户进程都是通过 `Zygote` 孵化出来的，而 `Zygote` 的执行程序是 `app_process`。安装 Xposed，将 `app_process` 替换，然后替换对应的虚拟机，这样 `Zygote` 孵化器就变成了 Xposed 孵化器。

![image-20211003134604388](https://bbs.kanxue.com/upload/attach/202110/905443_NMDRG5MSVSY9PKZ.png)

我们先来分析一下替换后的 `app_process`：

