
# [原创]源码编译（2）——Xposed源码编译详解

# 源码编译（2）——Xposed源码编译详解

[TOC]

## 一、前言

在上篇帖子中我们完成了Android 6.0 从源码下载到源码编译的全过程，详细可以参考[源码编译（1）——Android6.0源码编译详解](https://bbs.pediy.com/thread-269575.htm)，本文主要讲解Xposed框架源码编译到源码定制的详细过程，实验环境主要是Android 6.0，在此过程中踩过不少坑，遇到各种问题，好在后面都一一解决了，这个过程中尤其感谢一些大佬的博客贴子，从中学习到了很多。本文将从基于我们编译的Android 6.0，从Xposed源码编译到Xposed源码定制，详细记录全过程。此实验分为两部分：本文Xposed源码编译和下文Xposed源码定制

**致谢：**

首先感谢世界美景大佬的[定制Xposed框架](https://bbs.pediy.com/thread-255836.htm)，从里面学习到对Xposed框架特征的修改，但是由于个人水平有限，大佬的贴子不够详细，不能完整复现，经过搜索发现肉丝大佬的基于此的两篇详细的贴子讲解：<a href="elink@d41K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0M7q4)9J5k6i4N6W2K9i4S2A6L8W2)9J5k6i4q4I4i4K6u0W2j5$3!0E0i4K6u0r3M7#2)9J5c8X3x3&6y4%4A6G2g2s2S2d9M7V1g2W2h3f1I4$3c8o6S2k6N6@1W2g2g2W2p5`.">来自高纬的对抗：魔改XPOSED过框架检测(上)</a>和<a href="elink@03aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0M7q4)9J5k6i4N6W2K9i4S2A6L8W2)9J5k6i4q4I4i4K6u0W2j5$3!0E0i4K6u0r3M7#2)9J5c8W2W2m8e0f1y4J5f1g2y4A6x3p5I4r3d9V1N6z5d9i4N6n7z5i4q4t1c8p5p5`.">来自高纬的对抗：魔改XPOSED过框架检测(下)</a>，本文的Xposed定制详细参考了大佬的贴子，一些bug也是从中得到了解决，而Xposed编译则参考博客：<a href="elink@76cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2B7K9h3q4F1M7$3S2#2i4K6u0W2j5$3!0E0i4K6u0r3M7q4)9J5c8U0j5@1y4K6q4T1j5h3t1@1z5h3y4T1x3b7`.`.">xposed源码编译与集成</a>，关于Xposed安装以及问题解决则参考：[学习篇-xposed框架及高版本替代方案](https://bbs.pediy.com/thread-264160.htm)

## 二、Xposed源码编译

### 1.实验准备

Xposed源码编译的过程中，我们需要详细的了解Xposed中不同模块对应的版本以及适配的Android 版本情况，这样能够帮助我们减少错误，本文实验的环境搭建为：

|**实验平台：**|**Ubuntu 16.0.4 (运行内存4GB+CPU 4+磁盘300G)**|

| -------------------- | ----------------------------------------------- |
|**Android源码：**|**android-6.0.1_r1**|

|**实验手机：**|**Google nexus5**|

|**XposedBridge：**|**XposedBridge-art**|

|**XposedInstaller:**|**3.1.5**|

### 2.编译流程

我们在学习Xposed的源码编译之前，我们先了解Xposed个各个模块的含义，详细参考官网：<a href="elink@0fbK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6J5L8%4k6G2z5o6V1`.">Xposed官网</a>

![image-20211001164001995](upload/attach/202110/905443_5YBYQ58GR7STS6N.png)

各个模块的含义（这里参考上文肉丝大佬博客（1））：

```java
XposedInstaller：下载安装Xposed.zip刷机包、下载安装和管理模块
XposedBridge：位于Java层的API提供者，模块调用功能时首先就是到这里，然后再“转发”到Native方法
Xposed：位于Native层的Xposed实际实现，实现方法替换的实际逻辑等功能，主要是在app_process上进行的二次开发
android_art：在原版art上进行的二次开发，目录及文件基本上与原版art相同，稍加修改提供对Xposed的支持
XposedTools:XposedInstaller下载的那个刷机zip包，就是用XposedTools编译打包出来的
```

我们了解Xposed的各个模块的作用，接着了解一下Xposed框架初始化的工作原理：

![image-20211001165501914](upload/attach/202110/905443_VHETHJWY7W96XRC.png)

Xposed源码编译的流程可以参考下图：

![image-20211001163008900](upload/attach/202110/905443_2AKMVWW5CTPDPSU.png)

### 3.XposedBridge

#### （1）源码下载

我们从官网上下载<a href="elink@ee1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6J5L8%4k6G2z5o6W2Q4x3V1k6j5M7r3!0K6k6h3c8n7M7X3W2V1k6$3g2Q4x3V1k6@1M7X3g2W2i4K6u0r3j5i4u0@1">XposedBridge</a>，这里我们最终下载的是Xposed_art

```java
git clone https://github.com/rovo89/XposedBridge.git
```

![image-20211001163008900](upload/attach/202110/905443_VEWHM7WFV6P3CCG.png)

#### （2）编译生成XposedBridge.jar

我们在生成`XposedBridge.jar`的过程中有两种方式：`mmm`编译和Android Studio编译（这里优先推荐使用AndroidStudio编译）

##### <1>mmm编译

首先我们需要在XposedBridge工程中编写`Android.mk`文件

```java
Android.mk是Android提供的一种makefile文件，用来指定诸如编译生成so库名、引用的头文件目录、需要编译的.c/.cpp文件和.a静态库文件等。编写android.mk文件，将编译需要的源代码，编译生成的结果形式等信息告知ndk build编译系统。Android.mk文件内容如下所示。
```

```java
LOCAL_PATH:= $(call my-dir)
include $(CLEAR_VARS)
LOCAL_DEX_PREOPT = false
LOCAL_MODULE := XposedBridge
LOCAL_SRC_FILES := \
$(call all-java-files-under, src) \
$(call all-java-files-under, lib/apache-commons-lang/external) \
$(call all-java-files-under, hiddenapistubs/src)
include $(BUILD_JAVA_LIBRARY)
$(LOCAL_INTERMEDIA_TARGETS): PRIVATE_EXTRA_JAR_ARGS := -C "$(LOCAL_PATH)" \
assets -C "#(LOCAL_PATH)" NOTICE.txt
include $(call all-makefiles-under, $(LOCAL_PATH))
```

这里我们需要注意版本，针对不同版本的XposedBridge，我们需要修改`LOCAL_SRC_FILES`中的文件选项，关于`Android.mk`的文件介绍，大家可以参考官网地址：<a href="elink@d3aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2j5$3!0E0i4K6u0r3L8X3c8C8i4K6u0r3k6%4g2A6k6r3g2K6i4K6u0r3j5h3&6V1M7X3!0A6k6q4)9#2k6X3#2C8i4K6y4r3K9r3I4Q4x3@1c8\*7K9q4)9J5k6r3y4F1">Android.mk</a>

![image-20211001163008900](upload/attach/202110/905443_N2S3HT8V735NMZH.png)

我们将`XposedBridge`文件夹放在Android源码目录下

![image-20211001163008900](upload/attach/202110/905443_9MT8PH5NDEEJKQH.png)

然后我们使用`mmm`针对模块编译：

```java
##初始化环境
source build/envsetup.sh
##选择设备
lunch 19
##编译
mmm /home/tom/SourceCode/android-6.0.1_r1/XposedBridge/
```

![image-20211001163008900](upload/attach/202110/905443_544DUG3JXKWPZ6J.png)

我们就可以在`/out/target/product/hammerhead/system/framework/`文件夹下看见生成的`XposedBridge`包

![image-20211001163008900](upload/attach/202110/905443_DXRKFY49G4FBSF4.png)

##### <2>android Studio编译

我们先从官网上下载源码，然后我们直接用Android Studio打开

![image-20211001163008900](upload/attach/202110/905443_6NM96FX2U4J8AUQ.png)

然后我们等待编译器默认配置，此时如果你的编译器没有安装sdk23，可能会报错

![image-20211001172232509](upload/attach/202110/905443_X5NNDQE7UHJRXD7.png)

这里我们进入File→Settings→Android SDK→勾选Android 6.0(Marshmallow)→Apply→OK，就会开始下载Sources for Android 23

![image-20211001172401367](upload/attach/202110/905443_3K46DP8PGMWHNR5.png)

下载完成后，我们只需要将编译器重新关闭，再次打开即可

其他的问题只需要按照编译器提示的缺失文件，点击安装，静静等待配置就可以了，详细的错误解决也可以参考肉丝的博客（一）

然后我们点击`build下Make Project或Rebuild Project`就可以编译成功了，生成路径`build/outputs/apk/app-release-unsigned.apk`

![image-20211001172401367](upload/attach/202110/905443_8Q4X29H62H2KRQR.png)

这里就编译生成不签名的apk，然后我们可以把apk重命名`XposedBridge.jar`

**编译模块开发api.jar**

如果我们需要提供给app调用的api，那么我们可以需要执行android studio的gradle project的other中的generateAPI选项，会在目录`build/api`中生成`apk.jar`

![image-20211001172401367](upload/attach/202110/905443_FJ5JDB52TF2VFY5.png)

![image-20211001173925091](upload/attach/202110/905443_X4U36XCA8ZTZE7F.png)

这里的`api-sources.jar`和`XposedBridge`中的就是一样的

我们在这里就可以获得`XposedBridge.jar`和`api.jar`

#### （3）路径存放

我们android studio产生的`XposedBridge.jar`就可以放置在`/out/target/product/hammerhead/system/framework/`文件夹下，而`mmm`编译的自动存防在该路径下，就不需要管

![image-20211001163008900](upload/attach/202110/905443_DXRKFY49G4FBSF4.png)

### 4.android_art和Xposed

#### （1）源码下载

我们从Xposed官网下载`android_art`和`Xposed`源码，我们这里是android6.0，因此下载Xposed_sdk23，参考地址：<a href="elink@8a1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6J5L8%4k6G2z5o6V1`.">Xposed官网地址</a>

```
xposed文件夹包括xposed修改之后的app_process等应用程序和libxposed_*.so等库文件的源码；art文件夹主要包括xposed修改之后的art虚拟机的文件。
```

![image-20211001163008900](upload/attach/202110/905443_FJQWMTH9W4BS2TP.png)

#### （2）android_art

我们将我们下载的`android_art`文件夹重命名为`art`文件夹，然后替换android源码下的`art`文件夹，这就相当于替换了android系统的虚拟机

![image-20211001163008900](upload/attach/202110/905443_M7G9KPRPMQ8USHD.png)

#### （3）xposed

将下载的`xposed`文件夹移动到`/SourceCode/Android-6.0.1_r1/frameworks/base/cmds/`文件夹下

![image-20211001163008900](upload/attach/202110/905443_BWYJK8GZ8S62C83.png)

注：这里我们注意下载的Xposed的首字母为大写，我们需要改成`xposed`，不然后面会报错，具体我们在错误专区中介绍

### 5.XposedTools

#### （1）源码下载

我们从官网上下载XposedTools，官网地址：<a href="elink@183K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6J5L8%4k6G2z5o6W2Q4x3V1k6j5M7r3!0K6k6h3c8f1L8$3!0D9M7H3`.`.">XposedTools</a>

![image-20211001163008900](upload/attach/202110/905443_K6WKMXKFDZZWUCE.png)

#### （2）build.conf文件配置

```
build.conf文件是一个配置文件，会被build.pl读取使用。build文件中主要定义了编译xposed的结果的存放路径，android源码的路径，还有一些版本信息之类的值。
将下载的zip包解压，在文件夹下创建build.conf文件
```

```
[General]
outdir = /home/tom/SourceCode/XposedBridge
[Build]
version = 65(custom build by author )
makeflags = -j4
[AospDir]
23 =/home/tom/SourceCode/android-6.0.1_r1
```

```java
outdir指向的路径是xposed编译之后生成的文件的存放路径。
version的值和后边的参数是自己设置的。编译之后会写入到xposed.prop文件中，后面会显示在我们的XposedInstaller中
makeflags是设置在后边编译过程中使用的线程数。
AospDir指向的路径是android源码存放的路径，前边的值是源码对应的SDK的版本号。这里我指向的是android6.0.1的源码，对应的android sdk的版本是23。
```

我们还需要在`outdir`目录下创建一个`java`文件夹，然后将上面生成的`XposedBridge.jar`放到`java`文件夹下

![image-20211001163008900](upload/attach/202110/905443_QCJUQNEV8T7JCBX.png)

#### （3）build.pl编译Xposed源码

`build.pl`文件在`xposed tools`文件夹下，在文件中有对`build.pl`文件的参数的设置规则等信息
