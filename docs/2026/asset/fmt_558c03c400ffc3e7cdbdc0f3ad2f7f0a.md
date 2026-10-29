
可以在 Android linker 的源码中搜索“`load_bias`”，可以找到很多详细的注释说明，也可以参考 linker 中对 `load_bias_` 变量的赋值程序逻辑。

### 目标 ELF 使用的编译选项对 hook 有什么影响？

会有一些影响。

对于外部函数的调用，可以分为 3 中情况：

1. 直接调用。无论编译选项如何，都可以被 hook 到。外部函数地址始终保存在 `.got` 中。

2. 通过全局函数指针调用。无论编译选项如何，都可以被 hook 到。外部函数地址始终保存在 `.data` 中。

3. 通过局部函数指针调用。如果编译选项为 `-O2`（默认值），调用将被优化为直接调用（同情况 1）。如果编译选项为 `-O0`，则在执行 hook 前已经被赋值到临时变量中的外部函数的指针，通过 PLT 方式无法 hook；对于执行 hook 之后才被赋值的，可以通过 PLT 方式 hook。

一般情况下，产品级的 ELF 很少会使用 `-O0` 进行编译，所以也不必太纠结。但是如果你希望你的 ELF 尽量不被别人 PLT hook，那可以试试使用 `-O0` 来编译，然后尽量早的将外部函数的指针赋值给局部函数指针变量，之后一直使用这些局部函数指针来访问外部函数。

总之，查看 C/C++ 的源代码对这个问题的理解没有意义，需要查看使用不同的编译选项后，生成的 ELF 的反汇编输出，比较它们的区别，才能知道哪些情况由于什么原因导致无法被 PLT hook。

### hook 时遇到偶发的段错误是什么原因？如何处理？

我们有时会遇到这样的问题：

* 读取 `/proc/self/maps` 后发现某个内存区域的访问权限为**可读**，当我们读取该区域的内容做 ELF 文件头校验时，发生了段错误（sig: `SIGSEGV`, code: `SEGV_ACCERR`）。

* 已经用 `mprotect()` 修改了某个内存区域的访问权限为**可写**，`mprotect()` 返回修改成功，然后再次读取 `/proc/self/maps` 确认对应内存区域的访问权限确实为**可写**，执行写入操作（替换函数指针，执行 hook）时发生段错误（sig: `SIGSEGV`, code: `SEGV_ACCERR`）。

* 读取和验证 ELF 文件头成功了，根据 ELF 头中的相对地址值，进一步读取 PHT 或者 `.dynamic` section 时发生段错误（sig: `SIGSEGV`, code: `SEGV_ACCERR` 或 `SEGV_MAPERR`）。

可能的原因是：

* 进程的内存空间是多线程共享的，我们在执行 hook 时，其他线程（甚至 linker）可能正在执行 `dlclose()`，或者正在用 `mprotect()` 修改这块内存区域的访问权限。

* 不同厂家、机型、版本的 Android ROM 可能有未公开的行为，比如在某些情况下对某些内存区域存在**写保护**或者**读保护**机制，而这些保护机制并不反应在 `/proc/self/maps` 的内容中。

问题分析：

* 读内存时发生段错误其实是无害的。

* 我在 hook 执行的流程中，需要直接通过计算内存地址的方式来写入数据的地方只有一处：即替换函数指针的最关键的那一行。只要其他地方的逻辑没有错误，这里就算写入失败了，也不会对其他内存区域造成破坏。

* 加载运行安卓平台的 APP 进程时，加载器已经向我们注入了 signal handler 的注册逻辑，以便 APP 崩溃时与系统的 `debuggerd` 守护进程通讯，`debuggerd` 使用 `ptrace` 调试崩溃进程，获取需要的崩溃现场信息，记录到 tombstone 文件中，然后 APP 自杀。

* 系统会精确的把段错误信号发送给“发生段错误的线程”。

* 我们希望能有一种隐秘的，且可控的方式来避免段错误引起 APP 崩溃。

先明确一个观点：不要只从应用层程序开发的角度来看待段错误，段错误不是洪水猛兽，它只是内核与用户进程的一种正常的交流方式。当用户进程访问了无权限或未 mmap 的虚拟内存地址时，内核向用户进程发送 `SIGSEGV` 信号，来通知用户进程，仅此而已。只要段错误的发生位置是可控的，我们就可以在用户进程中处理它。

解决方案：

* 当 hook 逻辑进入我们认为的危险区域（直接计算内存地址进行读写）之前，通过一个全局 `flag` 来进行标记，离开危险区域后将 `flag` 复位。

* 注册我们自己的 signal handler，只捕获段错误。在 signal handler 中，通过判断 `flag` 的值，来判断当前线程逻辑是否在危险区域中。如果是，就用 `siglongjmp` 跳出 signal handler，直接跳到我们预先设置好的“危险区域以外的下一行代码处”；如果不是，就恢复之前加载器向我们注入的 signal handler，然后直接返回，这时系统会再次向我们的线程发送段错误信号，由于已经恢复了之前的 signal handler，这时会进入默认的系统 signal handler 中走正常逻辑。

* 我们把这种机制简称为：SFP (segmentation fault protection，段错误保护)

* 注意：SFP 需要一个开关，让我们随时能够开启和关闭它。在 APP 开发调试阶段，SFP 应该始终被关闭，这样就不会错过由于编码失误导致的段错误，这些错误是应该被修复的；在正式上线后 SFP 应该被开启，这样能保证 APP 不会崩溃。（当然，以采样的形式部分关闭 SFP，用以观察和分析 hook 机制本身导致的崩溃，也是可以考虑的）

具体代码可以参考 `xhook` 中的实现，在源码中搜索 `siglongjmp` 和 `sigsetjmp`。

### ELF 内部函数之间的调用能 hook 吗？

我们这里介绍的 hook 方式为 PLT hook，不能做 ELF 内部函数之间调用的 hook。

inline hook 可以做到，你需要先知道想要 hook 的内部函数符号名（symbol name）或者地址，然后可以 hook。

有很多开源和非开源的 inline hook 实现，比如：

* substrate：`<mark class="encrypted">bf0K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3y4&6k6r3W2S2M7%4g2T1M7%4c8J5j5i4c8W2i4K6u0W2j5$3!0E0i4K6u0r3</mark>`

* frida：`<mark class="encrypted">efaK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2X3M7X3W2V1j5g2)9J5k6i4u0W2i4K6u0r3</mark>`

inline hook 方案强大的同时可能带来以下的问题：

* 由于需要直接解析和修改 ELF 中的机器指令（汇编码），对于不同架构的处理器、处理器指令集、编译器优化选项、操作系统版本可能存在不同的兼容性和稳定性问题。

* 发生问题后可能难以分析和定位，一些知名的 inline hook 方案是闭源的。

* 实现起来相对复杂，难度也较大。

* 未知的坑相对较多，这个可以自行 google。

建议如果 PLT hook 够用的话，就不必尝试 inline hook 了。

## 联系作者

`caikelun#qiyi.com` （请用 @ 替换 #）

## 许可证

Copyright (c) 2018, 爱奇艺, Inc. All rights reserved.

本文使用 <a href="elink@494K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0M7X3g2S2N6r3W2$3k6h3y4G2L8h3#2G2L8Y4y4Q4x3X3g2G2M7X3N6Q4x3V1k6D9K9h3y4W2L8Y4y4W2M7#2)9J5c8X3u0&6i4K6u0r3y4q4)9J5k6e0m8Q4x3V1k6D9k6h3N6S2L8r3y4G2k6r3f1`.">Creative Commons 许可证</a> 授权。

# github地址：

[a43K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6A6M7h3W2&6K9g2)9J5c8Y4S2Z5L8$3!0C8](https://bbs.kanxue.com/elink@36aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6A6M7h3W2&6K9g2)9J5c8Y4S2Z5L8$3!0C8)

# 特征

- 支持 Android 4.0 (含) 以上版本 (API level >= 14)。

- 支持 `armeabi`，`armeabi-v7a`，`arm64-v8a`，`x86` 和 `x86_64`。

- 支持 **ELF HASH** 和 **GNU HASH** 索引的符号。

- 支持 **SLEB128** 编码的重定位信息。

- 支持通过正则表达式批量设置 hook 信息。

- **不**需要 ROOT 权限。

- 不依赖于任何的第三方动态库。

- 纯 C 的代码。比较小的库体积。

作者居然还写了一篇安卓PLT hook的小白入门文章，还有配套的演示代码，原文在：[bfbK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6A6M7h3W2&6K9g2)9J5c8Y4S2t1L8$3!0C8i4K6u0r3j5X3I4G2j5W2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8X3c8G2j5%4y4Q4x3V1k6G2N6X3g2J5N6X3W2W2N6#2)9J5c8X3q4F1k6s2u0G2K9h3c8Q4y4h3k6H3L8s2c8Q4y4h3k6Z5L8$3!0C8i4K6g2X3L8%4k6W2M7Y4k6A6k6i4N6Q4x3X3g2\*7K9q4)9J5k6p5y4z5i4K6u0W2L8h3b7`.](https://bbs.kanxue.com/elink@566K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6A6M7h3W2&6K9g2)9J5c8Y4S2t1L8$3!0C8i4K6u0r3j5X3I4G2j5W2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8X3c8G2j5%4y4Q4x3V1k6G2N6X3g2J5N6X3W2W2N6#2)9J5c8X3q4F1k6s2u0G2K9h3c8Q4y4h3k6H3L8s2c8Q4y4h3k6Z5L8$3!0C8i4K6g2X3L8%4k6W2M7Y4k6A6k6i4N6Q4x3X3g2*7K9q4)9J5k6p5y4z5i4K6u0W2L8h3b7`.)

# Android PLT hook 概述

## 获取代码和资源

你始终可以从 [这里](https://bbs.kanxue.com/elink@443K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6A6M7h3W2&6K9g2)9J5c8Y4S2t1L8$3!0C8i4K6u0r3j5X3I4G2j5W2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8X3c8G2j5%4y4Q4x3V1k6G2N6X3g2J5N6X3W2W2N6#2)9J5c8X3q4F1k6s2u0G2K9h3c8Q4y4h3k6H3L8s2c8Q4y4h3k6Z5L8$3!0C8i4K6g2X3L8%4k6W2M7Y4k6A6k6i4N6Q4x3X3g2*7K9q4)9J5k6p5y4z5i4K6u0W2L8h3b7`.) 访问本文的最新版本。

文中使用的示例代码可以从 [这里](https://bbs.kanxue.com/elink@6f5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6A6M7h3W2&6K9g2)9J5c8Y4S2t1L8$3!0C8i4K6u0r3N6s2u0W2k6g2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8X3c8G2j5%4y4Q4x3V1k6G2N6X3g2J5N6X3W2W2N6#2)9J5c8X3y4G2k6r3f1`.) 获取。文中提到的 `xhook` 开源项目可以从 [这里](https://bbs.kanxue.com/elink@309K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6A6M7h3W2&6K9g2)9J5c8Y4S2t1L8$3!0C8) 获取。

## 开始

### 新的动态库

我们有一个新的动态库：`libtest.so`。

> 头文件 `test.h`

```
#ifndef TEST_H

#define TEST_H 1

#ifdef __cplusplus
extern "C" {
#endif

void say_hello();

#ifdef __cplusplus
}
#endif

#endif
```

> 源文件 `test.c`

```
#include <stdlib.h>
#include <stdio.h>

void say_hello()
{
    char *buf = malloc(1024);
    if(NULL != buf)
    {
        snprintf(buf, 1024, "%s", "hello\n");
        printf("%s", buf);
    }
}
```

`say_hello` 的功能是在终端打印出 `hello\n` 这6个字符（包括结尾的 `\n`）。

我们需要一个测试程序：`main`。

> 源文件 `main.c`

```
#include <test.h>

int main()
{
    say_hello();
    return 0;
}
```

编译它们分别生成 `libtest.so` 和 `main`。运行一下：

```
caikelun@debian:~$ adb push ./libtest.so ./main /data/local/tmp
caikelun@debian:~$ adb shell "chmod +x /data/local/tmp/main"
caikelun@debian:~$ adb shell "export LD_LIBRARY_PATH=/data/local/tmp; /data/local/tmp/main"
hello
caikelun@debian:~$
```

太棒了！`libtest.so` 的代码虽然看上去有些愚蠢，但是它居然可以正确的工作，那还有什么可抱怨的呢？赶紧在新版 APP 中开始使用它吧！

遗憾的是，正如你可能已经发现的，`libtest.so` 存在严重的内存泄露问题，每调用一次 `say_hello` 函数，就会泄露 1024 字节的内存。新版 APP 上线后崩溃率开始上升，各种诡异的崩溃信息和报障信息跌撞而至。

### 面临的问题

幸运的是，我们修复了 `libtest.so` 的问题。可是以后怎么办呢？我们面临 2 个问题：

1. 当测试覆盖不足时，如何及时发现和准确定位线上 APP 的此类问题？

2. 如果 `libtest.so` 是某些机型的系统库，或者第三方的闭源库，我们如何修复它？如果监控它的行为？

### 怎么做？

如果我们能对动态库中的函数调用做 hook（替换，拦截，窃听，或者你觉得任何正确的描述方式），那就能够做到很多我们想做的事情。比如 hook `malloc`，`calloc`，`realloc` 和 `free`，我们就能统计出各个动态库分配了多少内存，哪些内存一直被占用没有释放。

这真的能做到吗？答案是：hook 我们自己的进程是完全可以的。hook 其他进程需要 root 权限（对于其他进程，没有 root 权限就没法修改它的内存空间，也没法注入代码）。幸运的是，我们只要 hook 自己就够了。

## ELF

### 概述

ELF（Executable and Linkable Format）是一种行业标准的二进制数据封装格式，主要用于封装可执行文件、动态库、object 文件和 core dumps 文件。

使用 google NDK 对源代码进行编译和链接，生成的动态库或可执行文件都是 ELF 格式的。用 `readelf` 可以查看 ELF 文件的基本信息，用 `objdump` 可以查看 ELF 文件的反汇编输出。
