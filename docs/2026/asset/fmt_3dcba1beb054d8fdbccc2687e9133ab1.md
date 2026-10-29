
# [原创]Frida源码编译环境搭建

## 前言

`frida` 的官方文档写的并不是很好，有些例子好像还有些问题。这就不得不去研究它的源码了。`frida` 的源码有许多个模块，我们这只关注 <a href="elink@3bcK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6X3M7X3W2V1j5g2)9J5c8X3k6J5K9h3c8S2i4K6u0V1K9X3q4$3j5g2)9J5k6r3u0J5K9h3c8Y4k6b7`.`."><code>frida-java-bridge</code></a> 这个模块。为什么呢？这个模块实现了 **js世界到java世界的单向通道**。所以我们主要的代码在这。可以看看这篇文章对 `frida-java` 的介绍 **[frida源码阅读之frida-java](https://bbs.pediy.com/thread-229215.htm)**。这里我就记录一下 `frida-java` 的编译环境搭建。

## 环境

- VMware 12
- Ubuntu16
- Android8.0
- Google pixel（已 root）

## 步骤

1. 下载安装配置 Ubuntu16 需要使用到的软件
   1. 安装配置 JDK
   2. 安装配置 SDK
   3. 安装配置 NDK
   4. 编译安装 Nodejs
   5. 编译运行 frida-java

这里就跳过安装 Ubuntu16 虚拟机的步骤了。

## 下载安装配置 Ubuntu16 需要使用到的软件

### 配置 `apt` 国内软件源镜像

使用<a href="elink@bc2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0K9i4u0J5L8%4u0K6i4K6u0W2N6s2g2F1j5g2)9J5k6i4c8K6K9h3&6Y4K9s2g2S2i4K6u0W2k6h3c8#2i4K6u0W2j5$3&6Q4x3V1k6Z5k6h3I4H3i4K6u0r3N6h3u0#2L8Y4c8#2i4K6u0r3">清华大学开源软件镜像站</a>

将软件源的配置拷贝到：

```bash
sudo gedit /etc/apt/sources.list
```

更新 `apt`：

```bash
sudo apt-get update
```

### 安装 `curl`

```bash
sudo apt-get install curl
```

### 安装 `git`

```bash
sudo apt-get install git
```

## 安装配置 JDK

下载 jdk：

<mark class="encrypted">0bfK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3A6V1K9#2)9J5k6h3q4F1k6s2u0G2K9h3c8Q4x3X3c8K6N6s2g2V1K9h3!0Q4x3X3g2G2M7X3N6Q4x3V1j5`.</mark>

这里为了方便我在家目录创建一个 `work` 目录，有关环境我都安装到这个目录下。读者可以自行选择目录。

```bash
mkdir work
```

将下载的 jdk 解压：

```bash
tar -zxvf jdk-8u77-linux-x64.tar.gz
```

配置环境变量：

```bash
sudo gedit /etc/profile
```

加入配置如下，请修改自己的 jdk 位置：

```bash
#set java env
export JAVA_HOME=/home/fj/work/jdk1.8.0_231
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH
```

加载配置：

```bash
source /etc/profile
```

查看是否配置成功：

```bash
fj@ubuntu:~/work/jdk1.8.0_231$ java -version
java version "1.8.0_231"
Java(TM) SE Runtime Environment (build 1.8.0_231-b11)
Java HotSpot(TM) 64-Bit Server VM (build 25.231-b11, mixed mode)
```

## 安装配置 SDK

这里我借助<a href="elink@163K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3q4F1k6s2u0G2K9h3c8Q4x3X3c8K6N6s2g2V1K9h3!0Q4x3X3g2G2M7X3N6Q4x3V1j5`.">android studio</a>进行下载 sdk。

android studio 下载地址：<mark class="encrypted">7ffK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3q4F1k6s2u0G2K9h3c8Q4x3X3c8K6N6s2g2V1K9h3!0Q4x3X3g2G2M7X3N6Q4x3V1j5`.</mark>

下载后移动到 `work` 目录，解压运行：

```bash
tar -zxvf android-studio-ide-191.5977832-linux.tar.gz
cd android-studio/bin
./studio.sh
```

其他操作和 Windows 上的 `as` 一样。

安装 sdk 版本，我们下载 29 版本。

这里可以看到 `as` 下载的 sdk 在 `/home/fj/Android/Sdk`，所以我们需要配置这个目录的环境变量：

```bash
sudo gedit /etc/profile
```

配置如下，请注意修改自己 sdk 的位置：

注意配置 `build-tools/30.0.3`，因为编译时会用到 `dx`。

```bash
#set sdk
export ANDROID_SDK_HOME=/home/fj/Android/Sdk
export PATH=$PATH:${ANDROID_SDK_HOME}/tools
export PATH=$PATH:${ANDROID_SDK_HOME}/build-tools/30.0.3
export PATH=$PATH:${ANDROID_SDK_HOME}/platform-tools
```

加载配置文件：

```bash
source /etc/profile
```

运行 `dx` 命令，如果出现以下信息说明成功：

```bash
fj@ubuntu:~/work/android-ndk-r21d$ dx
error: no command specified
usage:
dx --dex [--debug] [--verbose] [--positions=<style>] [--no-locals]
[--no-optimize] [--statistics] [--[no-]optimize-list=<file>] [--no-strict]
[--keep-classes] [--output=<file>] [--dump-to=<file>] [--dump-width=<n>]
[--dump-method=<name>[\*]] [--verbose-dump] [--no-files] [--core-library]
[--num-threads=<n>] [--incremental] [--force-jumbo] [--no-warning]
[--multi-dex [--main-dex-list=<file> [--minimal-main-dex]]
[--input-list=<file>] [--min-sdk-version=<n>]
[--allow-all-interface-method-invokes]
```

## 安装配置 NDK

ndk 下载地址：<mark class="encrypted">6bbK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2k6$3!0G2k6$3I4W2i4K6u0W2j5$3&6Q4x3V1k6F1k6r3E0Q4x3V1k6V1L8%4N6F1L8r3!0S2k6s2y4Q4x3V1j5`.</mark>

下载 `android-ndk-r21d-linux-x86_64.zip` 这个版本，因为 `frida` 的 ndk 默认版本就是这个。

下载后移动到 `work` 目录，解压：

```bash
unzip android-ndk-r21d-linux-x86_64.zip
cd android-ndk-r21d
```

添加 `/etc/profile` 配置：

```bash
#set NDK env
export NDK_HOME=/home/fj/work/android-ndk-r21d
export PATH=$NDK_HOME:$PATH
```

加载配置文件：

```bash
source /etc/profile
```

查看 ndk 版本，如果出现以下信息说明成功：

```bash
fj@ubuntu:~/work/android-ndk-r21d$ ndk-build --v
GNU Make 4.2.1
Built for x86_64-pc-linux-gnu
Copyright (C) 1988-2016 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
```

## 编译安装 Nodejs

这里我使用源码安装，使用 `apt` 安装的话版本过老。不支持 `frida` 的编译，并且升级麻烦。

```bash
$ sudo git clone https://github.com/nodejs/node.git
Cloning into 'node'...
```

修改目录权限：

```bash
$ sudo chmod -R 755 node
```

使用 `./configure` 创建编译文件，并按照：

```bash
$ cd node
$ sudo ./configure
$ sudo make
$ sudo make install
```

查看 `node` 和 `npm` 版本：

```bash
fj@ubuntu:~/work/node$ npm -v
7.4.2
fj@ubuntu:~/work/node$ node -v
v16.0.0-pre
```

## 编译运行 frida-java

使用 Git 下载 `frida-java-bridge` 源码，这里我选择 `3.9.4`，因为我的 `frida` 用的就是这个版本：

```bash
fj@ubuntu:~/work$ git clone https://github.com/frida/frida-java-bridge.git --tag 3.9.4
```

`frida-java` 对应的版本在 `test/Makefile` 下可以看到：

```makefile
frida_version := 12.11.14
```

### 修改配置文件

```bash
cd frida-java-bridge/
sudo gedit test/config.mk
```

修改为如下内容：

```makefile
#修改为自己的sdk目录
ANDROID_SDK_ROOT ?= $(shell echo /home/fj/Android/Sdk)
#修改为自己的ndk目录
ANDROID_NDK_ROOT ?= /home/fj/work/android-ndk-r21d
ANDROID_ARCH ?= arm64
ANDROID_ABI ?= arm64-v8a
#因为我们前面在As中下载了29版本，所以可以不用更换
ANDROID_API_LEVEL ?= 29
ANDROID_BINDIR ?= /system/bin
ANDROID_LIBDIR ?= /system/lib64
APEX_LIBDIRS ?= /apex/com.android.runtime/$(shell basename $(ANDROID_LIBDIR)):/apex/com.android.art/$(shell basename $(ANDROID_LIBDIR))
DEBUG_PORT ?= 5042
```

一般情况修改上面我注释的部分就行了，如果你的机型的架构不一样注意修改一下。

### 进行编译
