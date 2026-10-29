
# [原创]一种将 LLVM Pass 集成到 NDK 中的通用方法

关于“如何将自己写的 LLVM Pass 集成到 NDK 中”这个问题，目前网上并没有很完美的方法，并且大多数已经过时。经过我的一番折腾，最终摸索出了一种比较简单实用的方法，可供参考。

## 0x00. 概览

简单来说，本文要介绍的方法是下载 NDK 中 `llvm-android` 部分的源码，在 `llvm-android` 中的 `llvm-project` 子项目中加入自己的 Pass 后与整个项目一起编译，用编译好的文件替换掉 NDK 中的工具链。

该方法有以下优点：

- Windows、macOS、Linux 通用，各 NDK 版本也通用
- 理论上加入自己的 Pass 后不会出现不兼容的问题
- 操作过程简单易懂

当然也有缺点：

- 无法直接照搬 OLLVM、Hikari、Armariris 等现成项目的源码，需要手动做一些迁移
- 第一次编译比较耗时

## 0x01. 操作流程

本文并不是直接将最终的操作方法摆在大家面前，还讲解了为什么要这么操作，因此过程会显得较冗长，需要有一点耐心看下去。

### 1. 环境准备

如果是 Windows，首先需要准备 Linux 虚拟机进行交叉编译，因为 NDK 不支持在 Windows 上直接编译，我这里使用的是 **Ubuntu 20.04.3**；macOS 则不需要。NDK 版本我选择的是 **23.1.7779620**。

本文以 Windows + Linux 虚拟机为例讲解，macOS 下的操作大同小异。

以下使用的指令全部以 root 权限执行。

### 2. 下载 `llvm-android` 源代码

NDK 使用的是 `git-repo` 这一工具管理，而不是 `git`，所以首先需要安装 `git-repo`：

```bash
curl https://storage.googleapis.com/git-repo-downloads/repo > /usr/bin/repo
chmod a+x /usr/bin/repo
```

上面这个地址需要科学上网才能访问，我们可以换成国内的清华源：

```bash
curl https://mirrors.tuna.tsinghua.edu.cn/git/git-repo > /usr/bin/repo
chmod a+x /usr/bin/repo
```

在 `$NDK_PATH\toolchains\llvm\prebuilt\windows-x86_64\AndroidVersion.txt` 中查看 NDK 使用的 LLVM 版本，在我使用的 NDK 中，`AndroidVersion.txt` 的内容是：

```text
12.0.8
based on r416183c1
```

在对应路径中找到对应版本的 `llvm-android` 源码：

![图片描述](https://bbs.kanxue.com/upload/attach/202201/910514_AY6XK8JRE7TF4TV.png)

Google 的文档中给出了下载 `llvm-android` 源代码的方法，但这里默认下载的是最新版本：

```bash
mkdir llvm-toolchain && cd llvm-toolchain
repo init -u https://android.googlesource.com/platform/manifest -b llvm-toolchain
repo sync -c
```

我们需要做一些操作来换成我们想要的版本，在 `$NDK_PATH\toolchains\llvm\prebuilt\windows-x86_64` 目录下找到一个 `manifest_xxxx.xml` 文件，我这里是 `manifest_7714059.xml`。执行完下列指令后：

```bash
mkdir llvm-toolchain && cd llvm-toolchain
repo init -u
```

将 `manifest_7714059.xml` 复制到 `.repo/mainifests` 文件夹中（注意 `.repo` 是隐藏文件夹，并且从 Windows 复制到 Linux 会有格式转换的问题，这里建议使用 VSCode 的 SSH-Remote 插件避免上述问题）：

![图片描述](https://bbs.kanxue.com/upload/attach/202201/910514_7JRCVD7UTBZAHGG.png)

继续执行：

```bash
repo init -m manifest_7714059.xml
repo sync -c
```
