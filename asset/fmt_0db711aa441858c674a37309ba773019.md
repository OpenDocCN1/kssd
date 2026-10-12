
# sentinel superpro

> 原文：[`www.pediy.com/kssd/pediy05/pediy50370.htm`](https://www.pediy.com/kssd/pediy05/pediy50370.htm)

首先用一点儿时间来谈谈什么是加密狗(dongle)，dongle 经常被认为是硬件保护，它是一个可被附加在计算机并口、串口或 USB 上的小插件，它包含厂家烧制的 EPROM 和定制的专用集成电路。dongle 保护的的原理就是软件开发者在程序里经常检查 dongle 中的单元（Cell）的并对比返回值，这种检查可以是直接读单元或者是使用某种内部算法（此时单元受到保护无法直接读）。尽管任何称职的硬件工程师都能很容易地分析出 dongle 是如何实现的，但是实际上并不用那么麻烦，dongle 保护最薄弱的环节在于应用程序要通过 dongle 厂家提供的函数库来访问 dongle，而应用程序和这些函数的连接通常是很弱的，因为软件开发人员不管这些函数如何访问 dongle，它只检查函数的返回值是表示成功还是失败，那么只要打补丁让所有函数返回成功即可，而且这些函数通常并不多，从厂家提供的 API 手册中很容易查到这些函数的定义（参数和返回值）。一般来说 dongle 解密不需要原来的正版狗。

必须指出从（西方）法律上来说，破解狗与调试独立的程序有一些细微的不同，后者是完全合法的，但前者在德国有被判为非法的案例（即使你已经购买了正版的软件），我不知道我国法律是如何规定的，这种涉嫌侵权的情况当然是民不告、官不纠，好在我们不讨论国产软件，你如果被引渡到美国或德国本站不负任何责任。

下面是一些著名 dongle 厂商的官方网站，从中可以下载用户手册，应用程序接口（API）和其它相关资料，也可以了解 dongle 技术的最新进展。

比如说，从彩虹公司提供的 API 文档中我们可以了解到 Sentinel Superpro(以下简称 `sspro`)的一些情况，`sspro` 有 64 个内存单元(memory cells)，其中 56 个可以被用户使用，这些单元中的每一个都可以被用为三种类型之一：算法、数据值和计数器。

算法(algorithm)是这样一种技术，你用 `sproQuery(queryData)` 函数访问它，其中 `queryData` 是查询值，上述函数有一个返回值，被加密的程序知道一组这样的查询值/返回值对，在需要加密的地方，用上述函数检查狗的存在和真伪。对于被指定为算法的单元，软件上是无法读和修改的，即使你是合法的用户也是如此，我理解这种技术除了增加程序复杂性以外，主要是为了对付使用模拟器技术的破解。

数据值和计数器就比较好理解了，数据值用户存储在可读写的单元中的数，用户可以用它存储自己的信息，计数器（counter）是这样一种单元，软件开发商在其软件中使用 `sproDecrement` 函数可以把其值减一，当计数器和某种活动的(active)算法关联时，计数器为零则会封闭(deactive) 这个算法。

下面是 `sspro` 的所有 API 函数：

*   `RNBOsproActivate()` 激活或反激活一个算法以便它能被 `RNBOsproQuery()`函数使用。
*   `RNBOsproDecrement()` 把计数器或一个可读写的单元减一。
*   `RNBOsproExtendedRead()` 读任一非隐藏单元的值和连接码(`access code`)
*   `RNBOsproFindFirstUnit()` 查找指定的狗。
*   `RNBOsproFindNextUnit()` 查找同一 ID 的下一个狗。
*   `RNBOsproFormatPacket()` 初始化一些数据结构，这个函数必须在其他 API 函数之前调用。
*   `RNBOsproGetVersion()` 返回驱动程序的版本。
*   `RNBOsproInitialize()` 驱动程序初始化。
*   `RNBOsproOverwrite()` 可以修改除保留单元 `00–07` 之外的任何单元的值和连接码。
*   `RNBOsproQuery()` 提供一个查询值，给应用程序返回一个加密的返回值。
*   `RNBOsproRead()` 读非隐藏单元的值。
*   `RNBOsproWrite()` 改变某单元的值。

`DesKey` 的 API 函数就更简单了：
*   `Dk2DriverInstalled` 检查驱动程序是否安装
*   `findDk2` 检查 `DK2` 狗是否安装
*   `dk2readmemory` 读 `DK2` 某个单元的内容
*   `dk2readdowncounter` 读计数器的值
*   `dk2decrementDowncounter` 计数器减一

具体的函数细节可以查看 `sspro` 和 `Deskey` 的开发者指南（本站有下载）

了解了上述函数之后，你就可以使用 `IDA` 对你的目标进行分析，然后应用相应的 `sig` 符号表（本站有下载），`IDA` 就可以在反汇编的结果中找到上述函数（如果有的话），接下来就看你的汇编语言工夫和思维是否清晰了。

先要找到以下函数被编译在什么地方：

*   `RNBOsproActivate()` 激活或反激活一个算法以便它能被 `RNBOsproQuery()`函数使用。
*   `RNBOsproDecrement()` 把计数器或一个可读写的单元减一。
*   `RNBOsproExtendedRead()` 读任一非隐藏单元的值和连接码(`access code`)
*   `RNBOsproFindFirstUnit()` 查找指定的狗。
*   `RNBOsproFindNextUnit()` 查找同一 ID 的下一个狗。
*   `RNBOsproFormatPacket()` 初始化一些数据结构，这个函数必须在其他 API 函数之前调用。
*   `RNBOsproGetVersion()` 返回驱动程序的版本。
*   `RNBOsproInitialize()` 驱动程序初始化。
*   `RNBOsproOverwrite()` 可以修改除保留单元 `00–07` 之外的任何单元的值和连接码。
*   `RNBOsproQuery()` 提供一个查询值，给应用程序返回一个加密的返回值。
*   `RNBOsproRead()` 读非隐藏单元的值。
*   `RNBOsproWrite()` 改变某单元的值。

其中 `RNBOsproInitialize` 是第一个被调用的，它初始化一些数据结构，

`RNBOsproFindFirstUnit` 是第一个读狗的函数，然后用 `RNBOsproQuery()`、

`RNBOsproRead()` 对狗进行验证。所有函数返回零表示成功。

打补丁使它们都返回零，然后对 `RNBOsproQuery`、`RNBOsproRead` 进行处理，返回它们想要的值。
