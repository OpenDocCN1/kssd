
# [分享] Hook Android C 代码基础总结

这几天自己开始深入学习 `Cydia Substrate` 框架在 Hook Android C 上的使用，但一开始对 Android Native 的开发不怎么了解，所以直接通过学习这方面的博文容易碰壁![](https://bbs.kanxue.com/view/img/face/50.gif)，碰到的问题没有相应的思路来很好解决、实现。便先学习了 Android Native 开发方面的知识，这里给大家推荐一篇 Android Native 的学习博文，讲得很详细：[Android 开发之 JNI 入门](https://bbs.kanxue.com/elink@141K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6Q4x3X3g2U0M7$3c8F1i4K6u0W2L8X3g2@1i4K6u0r3M7$3S2#2L8r3W2S2L8X3N6Z5j5h3&6Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8U0p5^5z5e0j5@1z5o6x3#2)。

下边是我学习 Hook Android C 的知识总结，希望可以给学习这方面的初学者有些帮助，也希望大牛们可以多多指点。![](https://bbs.kanxue.com/view/img/face/1.gif)

参考学习的英文博文：[Substrate - hooking C on Android](https://bbs.kanxue.com/elink@974K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6C8L8%4A6Q4x3X3g2A6L8#2)9J5c8X3q4F1k6s2u0G2K9h3c8Q4x3X3c8K6N6h3u0K6N6s2u0S2N6r3g2Q4x3X3c8U0i4K6u0V1K9r3!0G2K9$3W2F1k6#2)9J5c8R3`.`.)

实验 APK 在附件中。

## Hook Android C 代码

### 创建一个目标 APK

编写目标项目，用于本次实操过程的 Hook 对象。

#### 1. 创建项目

```bash
android create project --target android-23 --path targetapp --package com.example.targetapp --activity Mainactivity
```

执行命令，在当前目录下创建 Android 项目。

#### 2. 编写 C 程序

在项目下创建 `jni` 目录，并新建 `targetlib.c` 文件，其代码如下：

![](https://bbs.kanxue.com/upload/attach/201609/718841_js61a8nmr4ync7q.png)

其中 `doThings()` 方法将对应 Java 层中的声明的 Native `daThings()`，而 `arc4random()` 是系统上的仿生标准库（Bionic Standard Library），生成随机数。

#### 3. 编写 Android.mk

![](https://bbs.kanxue.com/upload/attach/201609/718841_luclsrt1sz3lk5b.png)

用于编译生成 `targetlib` 库文件。

#### 4. Java 层编写

![](https://bbs.kanxue.com/upload/attach/201609/718841_o4n22gdyekwncgn.png)

#### 5. 编译、调试运行、安装程序

在 `jni` 目录下，执行 `ndk-build` 进行库文件的编译。

![](https://bbs.kanxue.com/upload/attach/201609/718841_ovgmxazbv02vk45.png)

然后在项目目录下，执行 `ant debug` 进行项目的调试运行，生成 APK。

```bash
adb install ./bin/Mainactivity-debug.apk
```

安装 APK。

程序打印结果如下：

![](https://bbs.kanxue.com/upload/attach/201609/718841_wgtcrgs5ipwx3tk.png)

#### 6. 查看库文件中的 exported symbols

```bash
nm -aDC --defined-only libs/armeabi-v7a/libtargetLib.so
```

显示库文件所暴露的符号。

![](https://bbs.kanxue.com/upload/attach/201609/718841_fcsao47uk3hp7ok.png)

忽略其中 `_` 的符号，该库文件暴露了下边两个方法：

```text
00000e95 T Java_com_example_targetapp_Mainactivity_doThings
00000e6d T getAge
```

这些暴露出来的方法将可以直接被其他程序或库文件调用。

```bash
nm -aDCu libs/armeabi-v7a/libtargetLib.so
```

显示库文件中未定义的请求符号。

![](https://bbs.kanxue.com/upload/attach/201609/718841_ln0ww5b6xe07orw.png)

这样可以查看库文件所调用其他库的符号，可以看到库文件调用了 `arc4random()`。

## 创建 Substrate 模块

编写 Substrate 扩展模块来 Hook 上边库文件中的 `arc4random()` 方法，修改其返回值，使其返回固定值。

### 1. 创建项目

```bash
android create project --target android-23 --name Hooknative --package com.example.hooknative --path Hooknative --activity Mainactivity
```

创建 Hook 项目。

### 2. 修改 AndroidManifest.xml

声明权限：`cydia.permission.SUBSTRATE`，设置 `android:installLocation` 和 `android:hasCode`。

![](https://bbs.kanxue.com/upload/attach/201609/718841_k1qxuo1b45rwion.png)

### 3. 编写 C++ 程序

在项目目录下，创建 `jni` 目录并新建 `***.cy.cpp` 文件，以最后编译生成 `.cy.so` 文件才能被 Substrate 框架所链接到。代码如下：

![](https://bbs.kanxue.com/upload/attach/201609/718841_ixk9ev3ujqhw06z.png)

通过 `MSConfig()` 方法来配置过滤对象，这里是可执行文件，参数 2 应为文件的绝对路径，再调用 `MSHookFunction()` 方法来 Hook 其中的 `arc4random()` 方法。

### 4. 编译、调试运行、安装模块

在 `jni` 目录下，执行 `ndk-build` 进行库文件的编译。

然后在项目目录下，执行 `ant debug` 进行项目的调试运行，生成 APK。

```bash
adb install ./bin/Hooknative-debug.apk
```

安装 APK。

然后通过 Substrate 链接模块，重启系统，或执行下边命令重启 Zygote：

```bash
adb shell setprop ctl.restart zygote
```

修改后，打印结果如下：

![](https://bbs.kanxue.com/upload/attach/201609/718841_eorm5fnhrswjc33.png)

## Hook 库文件内部暴露方法

编写 Substrate 模块来 Hook 库文件中暴露的内部方法，进行修改返回值操作。

可在上边 C++ 源文件中添加下边代码，进行 Hook 操作。

![](https://bbs.kanxue.com/upload/attach/201609/718841_z28dd7rgcmv1o3y.png)

这里我们使用 `lookup_symbol()` 这个定义的方法来查找 `getAge()` 方法在库文件 `libtargetLib.so` 中的地址。需要注意的是，我们的 Substrate 模块将会在作为所有进程初始化的孵化器 Zygote 中第一个被加载，这时候库文件 `libtargetLib.so` 还没被加载到进程空间，直接查找其暴露的方法是行不通的。

这里通过 `dlopen()` 方法人为地打开和加载在 Zygote 的内存中的 ELF 二进制文件，并通过 `dlsym()` 方法来找到库文件中的暴露方法。同时 Substrate 也提供 `MSGetImageByName()` 和 `MSFindSymbol()` 来实现相同的功能。

Hook 结果输出如下：

![](https://bbs.kanxue.com/upload/attach/201609/718841_e0ozs7ogh5l4dtr.png)

## Hook 内部 non-exported 方法

### 1. 将内部方法 getAge() 设置为 non-exported

在 C++ 程序中将 `getAge()` 设置为 `static`，则方法就不会暴露出来给其他程序直接调用。

通过 `nm -aDC --defined-only libs/armeabi-v7a/libtargetLib.so` 将不会打印出：

```text
00000e6d T getAge
```

当然由于 `doThings()` 方法要被 Java 层调用，所以仍是暴露的。由于 `getAge()` 设置为 `static`，所以上边的模块将不再 Hook 到该方法，结果输出如下：

![](https://bbs.kanxue.com/upload/attach/201609/718841_d42twerw4djoq8t.png)

### 2. 静态分析定位 non-exported 方法

使用 IDA Pro 工具打开 `libtargetLib.so` 库文件，可以定位到其中 `getAge()` 方法对应的汇编程序如下：

![](https://bbs.kanxue.com/upload/attach/201609/718841_mkeoi4oun6dnr4q.png)

其中的 `sub_E48` 就是对应 `getAge()`，由图可得出 `sub_E48` 对应的地址为 `0xE48`，但这不是方法 `getAge()` 在库文件中的地址，而是基于库文件基地址的偏移量，所以应先找出库文件的基地址，然后加上方法的偏移量便可确定方法的地址。

### 3. 获取库文件的基地址

通过将库文件加载到 Zygote 的内存空间，然后通过 `/proc/self/maps` 来读取其基地址。

![](https://bbs.kanxue.com/upload/attach/201609/718841_fdxpheeo33q67k6.png)

### 4. Hook non-exported 方法

![](https://bbs.kanxue.com/upload/attach/201609/718841_a3avnsx6spp2yf4.png)

其中之所以在基地址上加上 `0xE48+1`，是因为这里运行在 Thumb 体系下，`+1` 使地址为奇数，对应 Thumb。

### 5. Hook 结果如下

![](https://bbs.kanxue.com/upload/attach/201609/718841_dv8qwfwz8if3sx2.png)

---

[[招生]科锐逆向工程师培训（2026 年 11 月 13 日实地，远程教学同时开班，第 57 期）](https://bbs.kanxue.com/thread-51839.htm)

## 上传的附件

- [Image 1.png](javascript:void(0);)（83.04kb，26 次下载）
- [Image 2.png](javascript:void(0);)（49.59kb，7 次下载）
- [Image 3.png](javascript:void(0);)（99.44kb，8 次下载）
- [Image 4.png](javascript:void(0);)（28.93kb，5 次下载）
- [Image 5.png](javascript:void(0);)（84.98kb，4 次下载）
- [Image 6.png](javascript:void(0);)（60.32kb，5 次下载）
- [Image 7.png](javascript:void(0);)（31.85kb，5 次下载）
- [Image 8.png](javascript:void(0);)（43.45kb，5 次下载）
- [Image 9.png](javascript:void(0);)（53.19kb，8 次下载）
- [Image 10.png](javascript:void(0);)（89.30kb，6 次下载）
- [Image 11.png](javascript:void(0);)（105.64kb，6 次下载）
- [Image 12.png](javascript:void(0);)（58.94kb，5 次下载）
- [Image 13.png](javascript:void(0);)（64.94kb，5 次下载）
- [Image 14.png](javascript:void(0);)（45.70kb，5 次下载）
- [Image 15.png](javascript:void(0);)（93.19kb，8 次下载）
- [Image 16.png](javascript:void(0);)（64.46kb，9 次下载）
- [Image 17.png](javascript:void(0);)（29.19kb，7 次下载）
- [apk.zip](javascript:void(0);)（438.70kb，155 次下载）
