
# [原创] 逆向专版 `super-jadx` v1.0.0 发布（`jadx` 反编译工具增强版），支持类、方法、字段、变量重命名

> 几天前 `jadx` 作者放出了最新的 1.2.0 版本，首度支持了类、方法、字段的重命名特性。但仍有一些不足和缺陷，所以放出本人对 `jadx` 增强后的版本。

## 一、最新的 `jadx` 1.2.0 有哪些不足？

今天我简单试用了最新的 `jadx`，发现有以下不足：

> 1. 不支持对方法内的变量进行重命名。
> 2. 不支持自动对父、子类中 override 的其他方法进行重命名。（如果仅重命名 1 个方法，会导致代码逻辑上的错误。）
> 3. 对部分 override 方法重命名时存在 bug，会没有反映。

以上两点也刚好是我投入很多时间去解决的问题。相信后续 `jadx` 作者会逐步完善这些特性和体验。

## 二、什么是 `super-jadx`

`super-jadx` 是本人基于原生 `jadx` 进行的一个增强版本，专为 Android 逆向人员使用的版本。主要增加了如下特性：

> 1. 增加对类、方法、字段、变量重命名的支持（原生 `jadx` 不支持）。
> 2. 对 override、interface 方法进行关联重命名（原生 `jadx` 不支持）。
> 3. 重复名称检查（自动对其他 class 和 interface 的 override、interface 方法进行关联检查）（原生 `jadx` 不支持）。

![](https://bbs.kanxue.com/upload/attach/202011/721062_MMZC4EDTF478N9N.gif)

## 三、为什么要做这个 `super-jadx`

`jadx` 的搜索功能是非常好用的，但遗憾是一直不支持 UI 界面的重命名，本人一直期待作者 `skylot` 能够进行支持，在 `skylot/jadx` 的 issues 也有很多人表达了这个诉求，当时 `skylot` 多次明确表示不会增加重命名功能，对 `jadx` 的定位只是反编译代码。所以我在去年底和今年用零散的时间对 `jadx` 进行架构摸索和改造，逐步增加重命名的相关特性，这对我个人工作而言绝对是个刚需。所以我是先于 `skylot` 对 `jadx` 进行功能增强的。

## 四、开发过程中，需解决的问题，都非常耗费时间：

1. 增加对变量重命名的支持，原生 `jadx` 中完全没有对变量命名的支持。
2. 重命名后更新全局搜索缓存。
3. 分别支持搜索缓存`已建立`和`未建立`两种状态下的关联更新。
4. 对 override、interface 方法进行关联重命名。
5. 重复名称检查，需要自动对其他 class 和 interface 的 override、interface 方法进行关联检查。

后续计划增加的功能：

> 1. 代码索引持久化，下次启动无需再建立索引，直接可搜索。
> 2. 插件机制，类似 `jeb`，但是 API 接口和场景会更贴常用情况。
> 3. 重命名传导，自动对被赋值的其他变量、字段进行重命名。
> 4. 交叉引用图或其他辅助分析功能。

## 五、下载地址：

[https://github.com/pkilller/super-jadx/releases/tag/10.0](https://github.com/pkilller/super-jadx/releases/tag/10.0)

[3aaK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6H3K9$3W2D9L8r3I4W2M7W2)9J5c8Y4y4#2M7r3g2J5i4K6u0V1K9X3q4V1P5q4)9J5c8Y4u0W2L8r3g2S2M7$3g2K6i4K6u0r3N6r3q4Y4i4K6u0r3x3e0m8Q4x3X3f1H3](https://bbs.kanxue.com/elink@9f4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6H3K9$3W2D9L8r3I4W2M7W2)9J5c8Y4y4#2M7r3g2J5i4K6u0V1K9X3q4V1P5q4)9J5c8Y4u0W2L8r3g2S2M7$3g2K6i4K6u0r3N6r3q4Y4i4K6u0r3x3e0m8Q4x3X3f1H3)
