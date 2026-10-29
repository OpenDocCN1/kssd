
[474]: <mark class="encrypted">bc5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2L8X3c8J5L8$3W2V1i4K6u0W2k6$3!0G2k6$3I4W2M7$3!0#2M7X3y4W2i4K6u0W2j5$3!0E0i4K6u0r3M7r3I4S2N6r3k6G2M7X3#2Q4x3V1k6X3M7X3q4E0k6i4N6G2M7X3E0K6i4K6u0r3j5X3q4K6k6g2)9J5c8W2)9J5b7W2)9J5c8U0f1$3z5h3x3K6x3o6t1K6k6U0R3K6z5h3u0U0j5e0l9%4y4$3y4V1x3$3y4U0j5$3g2X3x3r3p5K6j5X3g2X3z5h3x3K6x3h3q4X3y4U0y4Q4x3U0f1#2c8g2)9J5y4e0t1I4i4K6u0r3</mark>

# 总结

本文算是笔者学习 Android 反序列化漏洞的一个笔记，按照时间线记录了几个公开的经典反序列化漏洞，并且介绍相关的修复策略以及绕过方法。从中我们可以看到，`Parcel` 作为轻量级的序列化方案，许多操作都需要手动管理，这导致了许多读写不匹配的问题，虽然后续引进了 `LazyBundle` 优化，但又引发了新的内存管理问题，使得传统二进制的 UAF 甚至 Double Free 类漏洞可以在 Java 世界重现。

漏洞本身只是一个引子，指导我们未来的研究方向。从这些漏洞中，我们也可以发现 Security By Default 的重要性，比如序列化 Java 容器导致的类型擦除问题和 `Bundle` 的序列化问题。在后期虽然尝试进行了缓释，但由于历史负担，很多不安全的设计只能缝缝补补延续下去，这也是后续挖掘和审计其他产品漏洞的一个重要着手点。

# 参考资料

- [ReparcelBug][rb]
- [ReparcelBug2][rb2]
- [LeakValue][lv]
- [Blackhat EU22 Android parcels: the bad, the good and the better - Introducing Android’s Safer Parcel][bh]

[me]: <mark class="encrypted">b0aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2N6X3W2D9M7r3q4F1i4K6u0W2j5$3!0E0i4K6u0r3x3U0l9J5x3q4)9J5c8U0l9%4i4K6u0r3x3e0q4Q4x3V1k6S2L8X3c8J5L8$3W2V1i4K6u0V1K9i4m8U0i4K6u0V1N6r3W2H3M7#2)9J5c8R3`.`.</mark>

[parcel]: <mark class="encrypted">0f8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2j5$3!0E0i4K6u0r3k6%4g2A6k6r3g2Q4x3V1k6U0L8$3#2H3L8$3&6W2L8Y4c8K6i4K6u0r3j5h3y4@1K9i4k6A6N6r3W2W2M7#2)9J5c8Y4m8S2M7X3y4W2L8r3q4T1L8r3g2K6i4K6u0V1j5h3&6V1i4K6u0V1j5Y4g2F1k6r3I4W2M7H3`.`.</mark>

[hidden]: <mark class="encrypted">832K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2^5k6r3q4Q4x3X3c8V1k6i4k6W2L8r3!0H3k6i4u0K6i4K6u0W2j5$3!0E0i4K6u0r3j5Y4W2H3j5i4y4K6i4K6u0V1K9r3W2V1k6r3g2F1i4K6u0V1j5i4m8A6M7#2)9J5c8R3`.`.</mark>

[bh]: <mark class="encrypted">e90K9s2c8@1M7s2y4Q4x3@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2T1L8r3q4U0K9$3S2S2N6q4)9J5k6h3y4G2L8g2)9J5c8X3g2#2i4K6u0V1x3U0u0Q4x3V1k6T1M7X3W2W2k6X3W2F1k6%4y4Q4x3V1k6K6j5$3S2W2k6s2g2D9k6g2)9J5c8X3W2F1k6r3g2^5i4K6u0W2K9s2c8E0L8q4)9J5x3$3q4F1k6s2u0G2K9h3c8Q4x3X3c8H3j5i4u0U0k6h3I4K6i4K6u0V1N6r3S2W2i4K6u0V1j5X3q4V1i4K6u0V1N6r3S2W2i4K6u0V1k6$3!0G2k6q4)9J5k6r3q4F1k6q4)9J5k6s2c8Z5k6g2)9J5k6r3u0W2N6s2c8W2M7W2)9J5k6q4)9J5k6q4)9J5k6r3W2F1N6s2u0G2k6s2g2U0K9h3&6Y4i4K6u0V1j5h3&6V1M7X3!0A6k6s2y4Q4x3X3c8K6j5h3k6W2M7W2)9J5k6s2m8S2M7X3y4W2L8q4)9J5k6o6t1^5y4o6l9@1</mark>

[rb]: <mark class="encrypted">e3cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6E0K9h3y4Z5j5h3I4T1k6h3c8F1j5i4u0K6K9$3W2Q4x3V1k6d9k6i4m8S2M7X3y4W2L8p5u0#2k6H3`.`.</mark>

[rb2]: <mark class="encrypted">55bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6E0K9h3y4Z5j5h3I4T1k6h3c8F1j5i4u0K6K9$3W2Q4x3V1k6d9k6i4m8S2M7X3y4W2L8p5u0#2k6K6t1`.</mark>

[lv]: <mark class="encrypted">79eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6E0K9h3y4Z5j5h3I4T1k6h3c8F1j5i4u0K6K9$3W2Q4x3V1k6x3k6h3q4C8g2X3q4D9N6h3f1`.</mark>

[img1.png]: `upload/attach/202303/844554_89EH3NQH4F2372Q.jpg` "From: Android Parcels: The Bad, the Good and the Better - Introducing Android's Safer Parcel"

[bin.png]: `upload/attach/202303/844554_K2X5MST8XX25V3W.jpg` "Bundle 序列化"

> 本文初发表至笔者[博客](https://bbs.kanxue.com/elink@7deK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2N6X3W2D9M7r3q4F1i4K6u0W2j5$3!0E0i4K6u0r3)，为防止不可抗力，故也在看雪上发布一下备份。

Java 在历史上出现过许多反序列化的漏洞，但大部分出自 J2EE 的组件。即便是 FastJSON 这种漏洞，似乎也很少看到在 Android 中被实际的触发和利用。本文即为对历史上曾出现过的 Android Java 反序列化漏洞的分析和研究记录。

## 目录

- [前言](https://bbs.kanxue.com/#前言)
- `Parcel 101`
- [AIDL](https://bbs.kanxue.com/#aidl)
- [Parcelable](https://bbs.kanxue.com/#parcelable)
- [内存布局](https://bbs.kanxue.com/#内存布局)
- [String](https://bbs.kanxue.com/#string)
- [Array](https://bbs.kanxue.com/#array)
- [Parcelable](https://bbs.kanxue.com/#parcelable)
- [FileDescriptor](https://bbs.kanxue.com/#filedescriptor)
- [Binder](https://bbs.kanxue.com/#binder)
- [其他](https://bbs.kanxue.com/#其他)
- [Bundle](https://bbs.kanxue.com/#bundle)
- [序列化](https://bbs.kanxue.com/#序列化)
- [存储](https://bbs.kanxue.com/#存储)
- `反序列化与 Bundle 风水`
- `自修改 Bundle`
- [漏洞利用](https://bbs.kanxue.com/#漏洞利用)
- [漏洞修复](https://bbs.kanxue.com/#漏洞修复)
- [CVE-2021-0928](https://bbs.kanxue.com/#cve-2021-0928)
- [漏洞介绍](https://bbs.kanxue.com/#漏洞介绍)
- `Android 广播`
- `Java 类型擦除`
- [其他](https://bbs.kanxue.com/#其他)
- [漏洞修复](https://bbs.kanxue.com/#漏洞修复)
- [LeakValue](https://bbs.kanxue.com/#leakvalue)
- `深入 LazyValue`
- `Parcel 内存管理`
- `Parcel UAF`
- [漏洞利用](https://bbs.kanxue.com/#漏洞利用)
- [总结](https://bbs.kanxue.com/#总结)
- [参考资料](https://bbs.kanxue.com/#参考资料)

# 前言

序列化和反序列化是指将内存数据结构转换为字节流，通过网络传输或者保存到磁盘，然后再将字节流恢复为内存对象的过程。在 Web 安全领域，出现过很多反序列化漏洞，比如 PHP 反序列化、Java 反序列化等。由于在反序列化的过程中触发了非预期的程序逻辑，从而被攻击者用精心构造的字节流触发并利用漏洞从而最终实现任意代码执行等目的。

Android 中除了传统的 Java 序列化机制，还有一个特殊的序列化方法，即 [Parcel](https://bbs.kanxue.com/elink@7dbK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2j5$3!0E0i4K6u0r3k6%4g2A6k6r3g2Q4x3V1k6U0L8$3#2H3L8$3&6W2L8Y4c8K6i4K6u0r3j5h3y4@1K9i4k6A6N6r3W2W2M7#2)9J5c8Y4m8S2M7X3y4W2L8r3q4T1L8r3g2K6i4K6u0V1j5h3&6V1i4K6u0V1j5Y4g2F1k6r3I4W2M7H3`.`.)。根据官方文档的介绍，`Parcelable` 和 `Bundle` 对象主要的作用是用于跨进程边界的数据传输（IPC/Binder），但 `Parcel` 并不是一个通用的序列化方法，因此不建议开发者将 `Parcel` 数据保存到磁盘或者通过网络传输。

作为 IPC 传输的数据结构，`Parcel` 的设计初衷是轻量和高效，因此缺乏完善的安全校验。这就引发了历史上出现过多次的 Android 反序列化漏洞，本文就按照时间线对其进行简单的分析和梳理。

> 注：本文中所展现的 AOSP 示例代码，如无特殊说明则都来自文章发表时的 master 分支。

# Parcel 101

在介绍漏洞之前，我们还是按照惯例先来了解下基础知识。对于有过 Android 开发或者逆向分析经验的同学应该对 `Parcel` 都不陌生，但通常也很少直接使用该类去序列化/反序列化数据然后进行 IPC 通信，而是通过 AIDL 等方法去自动生成模版，然后集成实现对应接口。

## AIDL

关于 AIDL 开发的示例可以参考 [Android 进程间通信与逆向分析](https://bbs.kanxue.com/elink@e80K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2N6X3W2D9M7r3q4F1i4K6u0W2j5$3!0E0i4K6u0r3x3U0l9J5x3q4)9J5c8U0l9%4i4K6u0r3x3e0q4Q4x3V1k6S2L8X3c8J5L8$3W2V1i4K6u0V1K9i4m8U0i4K6u0V1N6r3W2H3M7#2)9J5c8R3`.`.) 一文，简单来说，假设有以下 AIDL 文件：

|  |  |
| --- | --- |
| 1 2 3 4 5 6 7 8 9 10 | `package com.evilpan;` `interface IFooService {` `parcelable Person {` `String name;` `int age;` `boolean gender;` `}` `String foo(int a, String b, in Person c);` `}` |

那么生成的（Java）模版大致结构如下：

|  |  |
| --- | --- |
