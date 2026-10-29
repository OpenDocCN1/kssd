
```c
struct dyld\_cache\_header
{
char magic[16]; // e.g. "dyld\_v0 i386"
uint32\_t mappingOffset; // file offset to first dyld\_cache\_mapping\_info
uint32\_t mappingCount; // number of dyld\_cache\_mapping\_info entries
uint32\_t imagesOffset; // file offset to first dyld\_cache\_image\_info
uint32\_t imagesCount; // number of dyld\_cache\_image\_info entries
uint64\_t dyldBaseAddress; // base address of dyld when cache was built
uint64\_t codeSignatureOffset; // file offset of code signature blob
uint64\_t codeSignatureSize; // size of code signature blob (zero means to end of file)
uint64\_t slideInfoOffset; // file offset of kernel slid info
uint64\_t slideInfoSize; // size of kernel slid info
uint64\_t localSymbolsOffset; // file offset of where local symbols are stored
uint64\_t localSymbolsSize; // size of local symbols information
uint8\_t uuid[16]; // unique value for each shared cache file
uint64\_t cacheType; // 1 for development, 0 for optimized
};
```

检查共享缓存空间存在则直接复制其UUID到进程的`sharedCacheUUID`中，然后直接使用该缓存。

如果不存在，就需要进行创建，创建的过程如下:

1. 如果是x86环境，需要判断当前是否为安全启动模式，则会删除之前余留的cache文件，路径为`/var/db/dyld/dyld\_shared\_cache\_$arch`

2. 打开sharedCache文件，对于IPhone路径为`/System/Library/Caches/com.apple.dyld/dyld\_shared\_cache\_$arch`

3. 读取文件的前8192字节转换为`struct dyld\_cache\_header`，并做一些合法性检查

4. 处理cache文件，主要是将mapping信息提取出来，保存为一个数组`mappings[]`

5. 调用`\_shared\_region\_map\_and\_slide\_np`映射每个mapping

mapping信息如下:

```c
struct dyld\_cache\_mapping\_info {
uint64\_t address;
uint64\_t size;
uint64\_t fileOffset;
uint32\_t maxProt;
uint32\_t initProt;
};
```

和之前提到的segment信息类似，没有feilsize，因为不存在padding。

`\_shared\_region\_map\_and\_slide\_np`函数分别处理每个mapping，并最终使用`mmap`来完成cache到内存的映射操作。

> 每个mapping info对应一个`struct shared\_file\_mapping\_np`，但是这个结构体的定义在开源代码中没找到，并且在苹果文档中也进行了隐藏，见: <mark class="encrypted">04fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2S2M7s2m8D9k6g2)9J5k6h3y4G2L8g2)9J5c8X3c8G2j5%4g2E0k6h3&6@1j5i4c8A6L8$3&6Q4x3V1k6C8k6i4u0F1k6h3I4Q4x3V1k6K6K9r3q4J5k6h3c8Q4y4h3k6X3K9h3I4W2i4K6g2X3L8h3q4H3M7r3W2F1k6#2)9#2k6X3&6H3</mark>

# 后记

本文通过对MachO文件的文件格式研究，介绍了MacOS和iOS中可执行文件的加载过程，从内核中的处理一直到动态连接器dyld的代码分析。可以看出MachO与ELF相比实现方式各有千秋，但是在内核中原生增加了对代码的签名和加密，其实ELF也很容易实现类似的功能，但开放系统需要更多考虑兼容性的问题，不像苹果可以大刀阔斧的随便改。

对于MachO的深入理解其实也有助于日常的相关研究，比如Apple Store的加密实现以及代码签名的大致原理，还有针对dyld\_cache的处理等，其中每一项都值得去深入挖掘。而且本文也没有介绍到全部的MachO特性，比如Objective-C相关的段，具体的实战部分后面有时间会再去整理一下。

# 参考资料

- [Overview of the Mach-O Executable Format][spec]

- [Mach Object Files][spec0]

- [apple/darwin-xnu][xnu]

- [opensource-apple/dyld][dyld]

- RTFSC

[m]: https://bbs.pediy.com/thread-261349.htm

[spec0]: <mark class="encrypted">90bK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3y4A6L8r3W2F1k6r3g2J5i4K6u0W2j5X3g2Q4x3V1k6V1L8$3y4K6i4K6u0r3L8X3g2^5N6q4)9J5c8V1&6W2h3q4c8e0N6r3g2H3i4K6u0r3x3#2)9J5k6e0y4Q4x3V1k6F1k6q4)9J5c8V1c8W2N6W2c8G2L8$3I4K6i4K6u0r3x3e0c8Q4y4h3k6y4j5h3y4Z5e0#2)9J5c8V1#2S2j5$3S2a6i4K6u0W2K9s2c8E0L8r3c8Q4x3V1k6A6L8X3c8W2P5q4)9J5k6h3S2@1L8h3H3`.</mark>

[spec]: <mark class="encrypted">e2bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2S2M7s2m8D9k6g2)9J5k6h3y4G2L8g2)9J5c8X3I4A6j5Y4u0S2M7Y4W2Q4x3V1k6S2M7X3y4Z5K9i4k6W2i4K6u0r3k6r3!0U0N6h3#2W2L8Y4c8S2N6r3W2G2L8W2)9J5c8W2m8W2M7X3k6G2M7X3#2S2L8X3y4W2i4K6u0r3b7$3!0F1j5$3g2H3N6s2g2S2L8q4)9J5c8V1y4G2k6r3g2r3L8$3!0@1M7s2u0A6L8Y4c8Q4x3V1k6m8M7Y4c8A6j5$3I4W2M7#2)9J5c8V1#2S2j5$3S2a6e0%4k6W2M7Y4k6A6k6i4N6Q4x3X3g2Z5N6r3#2D9</mark>

[xnu]: <mark class="encrypted">231K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6S2M7s2m8D9k6g2)9J5c8X3c8S2M7Y4N6A6L8W2)9J5k6s2S2F1N6b7`.`.</mark>

[header]: <mark class="encrypted">552K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6S2M7s2m8D9k6g2)9J5c8X3c8S2M7Y4N6A6L8W2)9J5k6s2S2F1N6g2)9J5c8X3u0D9L8$3u0Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3V1k6q4h3q4c8q4f1V1&6m8e0q4)9#2k6V1S2q4b7f1c8q4f1W2y4Q4x3V1k6E0j5h3y4Z5i4K6u0V1L8#2)9J5c8X3I4G2j5h3c8W2M7W2)9J5k6h3R3`.</mark>

[syscall]: <mark class="encrypted">ad3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4y4@1i4K6u0W2k6$3W2@1K9s2g2T1i4K6u0W2j5$3!0E0i4K6u0r3P5i4u0H3y4U0l9@1i4K6u0r3x3U0y4W2z5o6k6V1j5$3f1&6j5$3p5I4x3X3u0X3y4e0p5@1k6h3j5`.</mark>

[app2dylib]: <mark class="encrypted">047K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6Q4x3X3g2A6L8h3A6#2L8W2)9J5k6h3&6W2N6q4)9J5c8Y4m8G2M7%4c8K6i4K6u0r3j5$3!0F1N6X3g2J5N6q4)9J5k6r3W2a6f1#2)9J5k6r3q4H3M7q4)9J5k6s2c8G2i4K6u0V1k6s2W2F1j5h3#2A6j5#2)9J5k6r3I4A6j5Y4u0S2M7Y4W2Q4x3V1j5`.</mark>

[restore]: <mark class="encrypted">a67K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6@1L8$3u0W2k6Y4g2@1N6i4u0W2M7W2)9J5c8Y4u0W2M7%4c8G2M7X3g2Q4x3X3c8K6P5h3#2T1L8$3H3`.</mark>

[link]: <mark class="encrypted">412K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6\*7K9r3q4F1k6$3u0#2K9s2g2S2K9g2)9J5k6h3y4G2L8g2)9J5c8Y4m8G2M7%4c8Q4x3V1k6E0j5h3y4Z5L8#2)9J5k6r3c8&6L8X3q4E0K9h3y4Q4x3X3c8D9K9h3&6C8i4K6u0W2K9s2c8E0L8l9`.`.</mark>

[sign]: <mark class="encrypted">dcaK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3&6W2N6$3!0K6P5r3u0G2L8$3E0Q4x3X3g2U0L8$3#2Q4x3V1k6S2M7Y4c8A6j5$3I4W2M7#2)9J5c8V1y4G2k6r3g2e0K9h3N6F1K9h3&6Y4i4K6u0W2M7r3c8X3</mark>

[dyld]: <mark class="encrypted">49cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6G2M7r3g2F1M7$3!0#2M7X3y4W2i4K6u0V1j5i4m8H3L8r3g2Q4x3V1k6V1P5h3I4V1</mark>

[jianshu]: <mark class="encrypted">a2eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6B7N6h3g2B7K9h3&6Q4x3X3g2A6L8g2)9J5c8Y4m8G2M7%4c8Q4x3V1j5$3z5o6b7@1z5e0l9K6y4K6V1^5y4K6p5%4x3o6t1J5x3U0t1J5i4K6t1K6K9r3g2S2k6r3W2F1k6#2)9J5k6o6p5J5</mark>

[cache]: <mark class="encrypted">594K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2B7K9h3q4F1M7$3S2#2i4K6u0W2j5$3!0E0i4K6u0r3M7q4)9J5c8X3b7J5k6h3g2T1y4o6l9^5y4h3q4X3y4b7`.`.</mark>

[data.png]: upload/attach/202009/844554\_VVZVQAM8ND2VXQN.jpg

[opcode.png]: upload/attach/202009/844554\_3SPK6K9JA6ZWAUS.jpg

之前写过一篇[深入浅出ELF](https://bbs.pediy.com/thread-261349.htm)，作为姊妹篇这次就来聊聊MacOS的可执行文件格式MachO。

目录

- Mach-O 101

- [Header](https://bbs.kanxue.com/#header)

- [Command](https://bbs.kanxue.com/#command)

- [LC\_SEGMENT](https://bbs.kanxue.com/#lc_segment)

- [TEXT](https://bbs.kanxue.com/#text)

- [DATA](https://bbs.kanxue.com/#data)

- [LINKEDIT](https://bbs.kanxue.com/#linkedit)

- [LC\_CODE\_SIGNATURE](https://bbs.kanxue.com/#lc_code_signature)

- [LC\_DYLD\_INFO\_ONLY](https://bbs.kanxue.com/#lc_dyld_info_only)

- [LC\_XXX\_DYLIB](https://bbs.kanxue.com/#lc_xxx_dylib)

- [其他](https://bbs.kanxue.com/#其他)

- [加载过程](https://bbs.kanxue.com/#加载过程)

- [内核空间](https://bbs.kanxue.com/#内核空间)

- [load\_segment](https://bbs.kanxue.com/#load_segment)

- [load\_code\_signature](https://bbs.kanxue.com/#load_code_signature)

- [load\_dylinker](https://bbs.kanxue.com/#load_dylinker)

- [用户空间](https://bbs.kanxue.com/#用户空间)

- [linking](https://bbs.kanxue.com/#linking)

- [dyld\_cache](https://bbs.kanxue.com/#dyld_cache)

- [后记](https://bbs.kanxue.com/#后记)

- [参考资料](https://bbs.kanxue.com/#参考资料)

# Mach-O 101

在[之前的文章](https://bbs.pediy.com/thread-261349.htm)中我们介绍过，可执行文件的使命有两个，一是方便开发者在编译、链接时提供可扩展的封装结构；二是在执行时能给操作系统(内核)提供内存映射信息。MachO也不例外。

MachO本身没有什么特别的含义，就是`Mach object`的简写，而[Mach](https://bbs.kanxue.com/elink@9cfK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8W2)9J5k6i4N6A6K9$3W2H3k6h3c8A6j5g2)9J5k6h3!0J5k6#2)9J5c8Y4N6A6K9$3W2Q4x3V1k6y4j5h3y4Z5i4K6g2X3K9$3g2J5L8X3g2D9)是早期的一个微内核。和ELF一样，MachO也极具拓展性，从全局视角来看，MachO文件可以分为三个部分，分别是:

1. Mach Header: 文件头信息

2. 可变长度的LOAD\_COMMAND信息

3. 上述LOAD\_COMMAND中所用到的具体信息(segments)

这里的segment可以理解为一段连续的内存空间，拥有对应的读/写/执行权限，并且在内存中总是页对齐的。每个segment由一个或者多个section组成，section表示特定含义数据或者代码的集合(不需要页对齐)。在macOS中，通常约定segment的名称为双下划线加全大写字母(如`__TEXT`)，section的名称为双下划线加小写字母(如`__text`)。

下面对这三个部分进行分别介绍。

> 注: MachO文件结构的表示通常分为32位和64位两种，本文以64位为例，毕竟这是历史的进程。

## Header
