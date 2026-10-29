
# [原创] Android 第二代加固（支持 Android 4.4–8.1）

## 介绍

代码放在 GitHub 上：<mark class="encrypted">d1aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6%4L8%4S2A6K9s2g2S2L8X3&6A6M7$3A6S2i4K6u0r3b7X3q4F1k6$3y4D9k6b7`.`.</mark>

第二代加固使用的是内存动态加载 `Dex`，也就是不落地加载，可以将 `Dex` 加密放在 `Apk` 中，在内存中实现解密。

## 兼容性

测试可以支持 Android 4.4–8.1 版本，目前还不能支持重写了 `Application` 类的 `Apk`。

## 原理

Dalvik 下的动态加载方法使用的是我这篇文章的方案：<https://bbs.pediy.com/thread-215078.htm>。

ART 下提供了 2 种方案：

1. 方案一是调用 `libart` 下的 `OpenMemory` 函数。如何将 Java 的 `mCookie` 和 C 层的 `cookie` 联系起来是一个难点。
2. 方案二是使用 ELF Hook 来实现。由于在 Nougat+ 上 `dlsym` `OpenMemory` 失败，还有 `dex_location` 和 `dex_cache_location` 的路径检查，使用方案一有些问题。

## 最后

Android 二代加固尽管脱壳不是很难，但是这种技术还是一些加固厂商的基础，很多在这基础上添加保护 `so`，以及类抽取等功能。

## 介绍

代码放在 GitHub 上：<mark class="encrypted">f12K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6%4L8%4S2A6K9s2g2S2L8X3&6A6M7$3A6S2i4K6u0r3b7X3q4F1k6$3y4D9k6b7`.`.</mark>

第二代加固使用的是内存动态加载 `Dex`，也就是不落地加载，可以将 `Dex` 加密放在 `Apk` 中，在内存中实现解密。

## 兼容性

测试可以支持 Android 4.4–8.1 版本，目前还不能支持重写了 `Application` 类的 `Apk`。

## 原理

Dalvik 下的动态加载方法使用的是我这篇文章的方案：<https://bbs.pediy.com/thread-215078.htm>。

ART 下提供了 2 种方案：

1. 方案一是调用 `libart` 下的 `OpenMemory` 函数。如何将 Java 的 `mCookie` 和 C 层的 `cookie` 联系起来是一个难点。
2. 方案二是使用 ELF Hook 来实现。由于在 Nougat+ 上 `dlsym` `OpenMemory` 失败，还有 `dex_location` 和 `dex_cache_location` 的路径检查，使用方案一有些问题。

## 最后

Android 二代加固尽管脱壳不是很难，但是这种技术还是一些加固厂商的基础，很多在这基础上添加保护 `so`，以及类抽取等功能。

[招生]科锐逆向工程师培训（2026 年 11 月 13 日实地，远程教学同时开班，第 57 期）

最后于 2018-3-20 09:07

被寒号鸟二代编辑

，原因：
