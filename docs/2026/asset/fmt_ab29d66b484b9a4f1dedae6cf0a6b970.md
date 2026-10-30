
`AllocContainer` -> `AddContainer` -> `AddSymbol` -> `ExtendMetadataBlock` //根据错误码`0xC0000023`，调用`ExtendMetadataBlock`函数

`ExtendMetadataBlock`函数105行调用`CClfsBaseFile::GetControlRecord`获取控制记录地址（`_CLFS_CONTROL_RECORD`），控制记录存在于`ClfsMetaBlockControl`块中。在`CreateLogFile`执行过程中，通过构造特定的字段使的`CreateLogFile`校验和为0，因此本次获取的控制记录实则为`ClfsMetaBlockControlShadow`块的。下面分析`GetControlRecord`函数，查看控制记录是如何被替换的。

![图片描述](upload/attach/202308/875946\_JXXA3QBVF8VM3ZP.png)

`图：ExtendMetadataBlock部分代码`

`GetControlRecord`函数18行调用`CClfsBaseFile::AcquireMetadataBlock`函数加载`ClfsMetaBlockControl`块，随后进行一系列检查，最终将`ClfsMetaBlockControl`中`CLFS_CONTROL_RECORD`的地址写入到参数2。

![图片描述](upload/attach/202308/875946\_KKZMWTY9WV2Y3PH.png)

`图：GetControlRecord代码`

`AcquireMetadataBlock`中调用了一个虚函数，在windbg中跟踪为`CClfsBaseFilePersisted::ReadMetadataBlock`函数。

![图片描述](upload/attach/202308/875946\_QAPHBAZQARMT6VP.png)

`图：AcquireMetadataBlock代码`

`ReadMetadataBlock`函数31至52行代码读取参数2指定的元数据块到分配的池内存，随后在58行调用`ClfsDecodeBlock`函数进行解码（因为`CreateLogFile`函数处理流程中`ClfsMetaBlockControl`校验置为了0，因此该函数会返回错误码）。73行代码再次调用`ReadMetadataBlock`函数，加载当前元数据块的shadow块。随后通过一系列检查到达75行，此时`v8`小于0，将跳转到117行代码开始执行。117~122代码，释放了`ReadMetadataBlock`为当前元数据块申请的池空间，然后使用其shadow块地址代替当前元数据块。

![图片描述](upload/attach/202308/875946\_Q2DZWPYBX9TG7DE.png)

`图：ReadMetadataBlock部分代码`

![图片描述](upload/attach/202308/875946\_T3SG3ZM983ZD39A.png)

`图：ReadMetadataBlock部分代码`

`ClfsDecodeBlock`函数先判断校验和是否为0，这里为0然后跳转到16行继续判断。通过`ReadMetadataBlock`的函数调用可知`a4`值为`0x10`，然后`MajorVersion`值为`0xF`，因此该检测失败，直接返回错误代码`0xC01A000A`。至此`ReadMetadataBlock`函数使用`ClfsMetaBlockControlShadow`块替换了`ClfsMetaBlockControl`块。

![图片描述](upload/attach/202308/875946\_ND58MWKVKQAV497.png)

`图：ClfsDecodeBlock代码`

![图片描述](upload/attach/202308/875946\_63B975XKD4MMY9Z.png)

`图：默认MajorVersion值`

回到`ExtendMetadataBlock`函数，可以得知105行代码获取的实际为`ClfsMetaBlockControlShadow`块。随后通过检测，跳转到`LABEL_41`处。

![图片描述](upload/attach/202308/875946\_HGTKU4EHRMRZRRU.png)

`图：ExtendMetadataBlock部分代码`

跳转到`LABEL_41`后，在208行调用了`CClfsBaseFilePersisted::WriteMetadataBlock`函数。需要注意的是`ClfsMetaBlockControlShadow`块中的`iFlushBlock`值为`0x13`，即`WriteMetadataBlock`的参数2值为`0x13`。

![图片描述](upload/attach/202308/875946\_FZJAGRPAV9GWT2F.png)

`图：ExtendMetadataBlock部分代码`

此时查看`WriteMetadataBlock`处理逻辑。31行代码使用24乘以参数2作为`this+0x30`的偏移，随后取得数据。在`ReadImage`中可以得知`this+0x30`的内存空间为`0x90`，而`24*0x13`结果为`0x1C8`，已经超出了`0x90`，从而取得了错误的地址。然后从错误地址的`0x28`偏移取得偏移B，然后将错误地址的偏移B的数据进行+1。此时通过漏洞已经触发，实现了数据自增1。

![图片描述](upload/attach/202308/875946\_3XHGQ2YQBC6TSZ3.png)

`图：WriteMetadataBlock部分代码`

## 任意函数执行

《样本分析》中指出`CreateLogFile`函数会调用`CClfsBaseFilePersisted::CheckSecureAccess`函数。`CheckSecureAccess`函数的55行起会遍历容器，然后64行获取`pContainer`值，并在69行和78行调用二次虚函数。因此可以创建一个日志文件，其包含了精心构造的容器，然后利用该漏洞修改容器的偏移，使其再次调用`CreateLogFile`时，遍历的容器为构造的容器，然后`pContainer`值指向用户空间中构造虚函数布局，这样调用虚函数时便可以执行我们想要执行的函数。由此构造函数利用链（函数利用链可以查看《CVE-2022-37969 技术分析》，该文章详细的介绍了相关内容），实现权限提升。

![图片描述](upload/attach/202308/875946\_8HYQ32ETMXW4NQ3.png)

`图：CheckSecureAccess部分代码`

## 权限提升思路

权限提升过程涉及到池空间分配、利用管道属性实现任意地址写入等操作，这里仅简单介绍下思路，不在详细展开，具体过程可以查看《样本分析》。

1. 创建一个包含容器的日志文件A，然后容器地址偏移`0x100`处构造一个`pContainer`成员有值的容器。

2. 通过`NtQuerySystemInformation`查询`0x7A00`大小的CLFS数据池。

3. 调用`CreateLogFile`函数将日志文件A加载到内核。

4. 通过`NtQuerySystemInformation`查询`0x7A00`大小的CLFS数据池，查询到新增的池地址，这个地址为日志文件A的`ClfsMetaBlockGeneral`块地址（该块的大小为`0x7A00`）。

5. 创建多个漏洞利用日志文件B，构造文件数据，使其最终可以在`WriteMetadataBlock`中实现越界数据访问，并进行数据自增。

6. 构造池布局空间。申请大量的匿名管道，然后将13个日志文件A的`ClfsMetaBlockGeneral`块地址写入到管道，使其申请`0x90`的池空间。

7. 关闭匿名管道，释放池空间，然后调用`CreateLogFile`函数将多个日志文件B加载到内核，然后在将13个日志文件A的`ClfsMetaBlockGeneral`块地址写入到管道，这样在日志文件B的`clsfhelp`（`this+0x30`）的周围就是构造的日志文件A的`ClfsMetaBlockGeneral`块地址。

8. 构造函数执行内存布局。

9. 调用`AddLogContainer`函数触发漏洞，修改日志文件A的`ClfsMetaBlockGeneral`块中`rgContainer`数据，使其容器偏移增加`0x100`。增加`0x100`的原因是利用漏洞增加的是后24位的数据，因此自增1，等于增加了`0x100`偏移。

10. 调用`CreateLogFile`函数再次加载日志文件A，随后通过`CheckSecureAccess`函数利用构造的容器，然后根据函数执行内存布局利用管道属性，实现任意数据写入，然后替换了进程的TOKEN从而实现提权。

# 漏洞修复

根据上面分析可以得知触发漏洞是由于`WriteMetadataBlock`的`ClfsEncodeBlock`函数将校验和置为了0，然后在触发错误时，仍更新了`ClfsMetaBlockControl`块数据，从而导致下一次加载`ClfsMetaBlockControl`块时使用了导致越界访问的`ClfsMetaBlockControlShadow`块。因此修复后的`WriteMetadataBlock`函数在82行对`ClfsEncodeBlock`的返回值进行了检测，在出现异常时不调用`WriteSector`函数，从而使得`ClfsMetaBlockControlShadow`不会被使用。《样本分析》中描述了其他的修复内容，这里就不再重复陈述。

![图片描述](upload/attach/202308/875946\_TXV64D6W8DHGXUJ.png)

`图：修复后的WriteMetadataBlock部分代码`

# 参考

<mark class="encrypted">25eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0M7q4)9J5k6i4N6W2K9i4S2A6L8W2)9J5k6i4q4I4i4K6u0W2j5$3!0E0i4K6u0r3M7#2)9J5c8W2q4D9M7%4b7$3b7#2S2Q4y4h3k6\*7x3f1p5$3z5e0S2f1N6Y4k6^5i4K6u0V1j5V1W2c8</mark> （CVE-2023-28252在野提权漏洞样本分析）

<mark class="encrypted">bd2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6A6L8$3&6W2M7$3y4#2x3o6l9%4i4K6u0r3j5$3I4X3M7#2)9J5k6r3c8G2j5%4y4Q4x3V1j5`.</mark> （CLFS格式）

<mark class="encrypted">a0aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2\*7M7$3y4S2L8r3g2J5i4K6u0W2j5$3!0E0i4K6u0r3j5X3I4G2k6%4y4Q4x3V1k6K6k6h3y4#2M7X3W2@1P5g2)9J5k6s2u0W2M7$3g2S2M7X3y4Z5i4K6u0r3N6r3g2U0K9r3&6A6j5$3q4D9i4K6u0V1j5h3&6S2L8s2W2K6K9i4y4Q4x3X3c8%4K9h3&6V1L8%4N6K6i4K6u0V1j5$3I4X3M7#2)9J5k6s2A6W2M7X3!0Q4x3X3c8V1j5i4W2Q4x3X3c8$3N6h3I4F1k6i4u0S2j5X3W2D9K9i4c8&6i4K6u0V1j5%4k6W2i4K6u0V1x3U0l9J5x3W2)9J5k6o6x3%4z5e0j5&6i4K6u0V1M7r3q4J5N6l9`.`.</mark> （CVE-2022-37969 技术分析 上）

<mark class="encrypted">1b2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2\*7M7$3y4S2L8r3g2J5i4K6u0W2j5$3!0E0i4K6u0r3j5X3I4G2k6%4y4Q4x3V1k6K6k6h3y4#2M7X3W2@1P5g2)9J5k6s2u0W2M7$3g2S2M7X3y4Z5i4K6u0r3N6r3g2U0K9r3&6A6j5$3q4D9i4K6u0V1j5h3&6S2L8s2W2K6K9i4y4Q4x3X3c8%4K9h3&6V1L8%4N6K6i4K6u0V1j5$3I4X3M7#2)9J5k6s2A6W2M7X3!0Q4x3X3c8V1j5i4W2Q4x3X3c8$3N6h3I4F1k6i4u0S2j5X3W2D9K9i4c8&6i4K6u0V1j5%4k6W2i4K6u0V1x3U0l9J5x3W2)9J5k6o6x3%4z5e0j5&6i4K6u0V1M7r3q4J5N6o6u0Q4x3X3c8W2P5s2m8D9L8$3W2@1i4K6u0V1j5h3&6S2L8s2W2K6K9i4y4Q4c8f1k6Q4b7V1y4Q4z5o6S2o6g2V1g2Q4x3X3b7J5x3o6t1J5i4K6u0V1x3K6M7&6y4U0V1`.</mark> 技术分析 下）

# 简介

红雨滴团队的《CVE-2023-28252在野提权漏洞样本分析》文章详细的分析了恶意文件并定位到漏洞触发点，但由于文章的主要目的是分析恶意文件而非讲解漏洞成因，因此漏洞成因方面的内容偏少。该文的主要内容是分析`clfs.sys`内的函数，以了解漏洞触发过程。由于笔者刚入门，如果文章出现错误，请多多包涵。

# 实验环境

- Windows 10 22H2
- IDA

![image.png](https://bbs.kanxue.com/upload/attach/202308/875946_3WPNA7PDEB96CGB.jpg)

# CLFS

## 简述

通用日志文件系统（Common Log File System，缩写CLFS）是一个通用目的的日志文件系统，它可以从内核模式或用户模式的应用程序访问，用以构建一个高性能的事务日志。

## 文件格式

文件格式的类型定义可以参考`ionescu007`的文章，下面进行简单介绍。

元数据块类型为6种，其中Shadow块为备份块，当主块存在异常时，会使用Shadow块。类型定义如下：

| 1  2  3  4  5  6  7  8  9 | `typedef enum _CLFS_METADATA_BLOCK_TYPE`  `{`  `ClfsMetaBlockControl,`  `ClfsMetaBlockControlShadow,`  `ClfsMetaBlockGeneral,`  `ClfsMetaBlockGeneralShadow,`  `ClfsMetaBlockScratch,`  `ClfsMetaBlockScratchShadow`  `} CLFS_METADATA_BLOCK_TYPE,` `*``PCLFS_METADATA_BLOCK_TYPE;` |
| --- | --- |

每个元数据块都以`CLFS_LOG_BLOCK_HEADER`结构起始，后面跟着元数据块特有的结构与数据，简单示意图如下：

![图片描述](https://bbs.kanxue.com/upload/attach/202308/875946_8WDF7PDNBAPD8TX.png)

该漏洞涉及`ClfsMetaBlockControl`和`ClfsMetaBlockGeneral`块，其涉及的主要类型定义如下：

|  |  |
| --- | --- |
