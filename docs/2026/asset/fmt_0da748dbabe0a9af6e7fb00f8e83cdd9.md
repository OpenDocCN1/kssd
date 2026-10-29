
1. `recursiveLoadLibraries`
2. `recursiveUpdateDepth`
3. `recursiveRebase`
4. `recursiveBind`
5. `weakBind`
6. `recursiveGetDOFSections`

根据名字不难看出其作用，其中大部分函数名称带`recursive`，这是因为动态库本身也可能会依赖其他的动态库，因此需要递归进行处理（当然循环依赖会有对应的处理）。其中`recursiveUpdateDepth`不太直观，其实作用只是为了对镜像进行排序，被依赖的库会出现在依赖者之前。

## dyld_cache

在上面第4步中说到要加载共享缓存库，这是个什么东西呢？这一步的目的其实是为了加速动态库的加载过程。对于我们自己编译的macOS命令行程序可能还好，但是对于图形界面应用来说，每个应用启动时需要加载的动态库可能有上百个，而其中很大一部分是系统库，比如`UIKit`、`Foundation`等。因此苹果就事先把这些常用的库打包成缓存，程序启动时候直接映射到内存中，而无需逐个执行繁琐的处理和解析。

映射共享缓存库的函数为`mapSharedCache`，首先检查共享缓存库是否已经映射过：

```
static int __attribute__((noinline)) _shared_region_check_np(uint64_t* start_address)

{

    if ( gLinkContext.sharedRegionMode == ImageLoader::kUseSharedRegion ) 

        return syscall(294, start_address);

    return -1;

}

```

294号系统调用定义在内核中（`bsd/kern/syscalls.master`）：

```
294    AUE_NULL    ALL    { int shared_region_check_np(uint64_t *start_address) NO_SYSCALL_STUB; }

295    AUE_NULL    ALL    { int shared_region_map_np(int fd, uint32_t count, const struct shared_file_mapping_np *mappings) NO_SYSCALL_STUB; }

```

内核中的实现也比较简单，忽略错误检查，关键的代码如下：

```
int shared_region_check_np(

    __unused struct proc            *p,

    struct shared_region_check_np_args    *uap,

    __unused int                *retvalp) {

  // ...

  shared_region = vm_shared_region_get(current_task());

    if (shared_region != NULL) {

            /* retrieve address of its first mapping... */

        kr = vm_shared_region_start_address(shared_region,

                            &start_address);

            /* ... and give it to the caller */

        error = copyout(&start_address,

                    (user_addr_t) uap->start_address,

                    sizeof (start_address));

    // ...

        vm_shared_region_deallocate(shared_region);

  }

}

```

其内部实现姑且不管，继续回到用户空间，所返回的地址可以强制转换为`dyld_cache_header`格式：

```
struct dyld_cache_header

{

    char        magic[16];                // e.g. "dyld_v0    i386"

    uint32_t    mappingOffset;            // file offset to first dyld_cache_mapping_info

    uint32_t    mappingCount;            // number of dyld_cache_mapping_info entries

    uint32_t    imagesOffset;            // file offset to first dyld_cache_image_info

    uint32_t    imagesCount;            // number of dyld_cache_image_info entries

    uint64_t    dyldBaseAddress;        // base address of dyld when cache was built

    uint64_t    codeSignatureOffset;    // file offset of code signature blob

    uint64_t    codeSignatureSize;        // size of code signature blob (zero means to end of file)

    uint64_t    slideInfoOffset;        // file offset of kernel slid info

    uint64_t    slideInfoSize;            // size of kernel slid info

    uint64_t    localSymbolsOffset;        // file offset of where local symbols are stored

    uint64_t    localSymbolsSize;        // size of local symbols information

    uint8_t        uuid[16];                // unique value for each shared cache file

    uint64_t    cacheType;                // 1 for development, 0 for optimized

};

```

检查共享缓存空间存在则直接复制其UUID到进程的`sharedCacheUUID`中，然后直接使用该缓存。

如果不存在，就需要进行创建，创建的过程如下：

1. 如果是x86环境，需要判断当前是否为安全启动模式，则会删除之前余留的cache文件，路径为`/var/db/dyld/dyld_shared_cache_$arch`
2. 打开sharedCache文件，对于IPhone路径为`/System/Library/Caches/com.apple.dyld/dyld_shared_cache_$arch`
3. 读取文件的前8192字节转换为`struct dyld_cache_header`，并做一些合法性检查
4. 处理cache文件，主要是将mapping信息提取出来，保存为一个数组`mappings[]`
5. 调用`_shared_region_map_and_slide_np`映射每个mapping

mapping信息如下：

```
struct dyld_cache_mapping_info {

    uint64_t    address;

    uint64_t    size;

    uint64_t    fileOffset;

    uint32_t    maxProt;

    uint32_t    initProt;

};

```

和之前提到的segment信息类似，没有feilsize，因为不存在padding。

`_shared_region_map_and_slide_np`函数分别处理每个mapping，并最终使用`mmap`来完成cache到内存的映射操作。

> 每个mapping info对应一个`struct shared_file_mapping_np`，但是这个结构体的定义在开源代码中没找到，并且在苹果文档中也进行了隐藏，见：d04K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2S2M7s2m8D9k6g2)9J5k6h3y4G2L8g2)9J5c8X3c8G2j5%4g2E0k6h3&6@1j5i4c8A6L8$3&6Q4x3V1k6C8k6i4u0F1k6h3I4Q4x3V1k6K6K9r3q4J5k6h3c8Q4y4h3k6X3K9h3I4W2i4K6g2X3L8h3q4H3M7r3W2F1k6#2)9#2k6X3&6H3

## 后记

本文通过对MachO文件的文件格式研究，介绍了MacOS和iOS中可执行文件的加载过程，从内核中的处理一直到动态连接器dyld的代码分析。可以看出MachO与ELF相比实现方式各有千秋，但是在内核中原生增加了对代码的签名和加密，其实ELF也很容易实现类似的功能，但开放系统需要更多考虑兼容性的问题，不像苹果可以大刀阔斧的随便改。

对于MachO的深入理解其实也有助于日常的相关研究，比如Apple Store的加密实现以及代码签名的大致原理，还有针对dyld_cache的处理等，其中每一项都值得去深入挖掘。而且本文也没有介绍到全部的MachO特性，比如Objective-C相关的段，具体的实战部分后面有时间会再去整理一下。

## 参考资料

- [Overview of the Mach-O Executable Format](https://bbs.kanxue.com/elink@2c5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2S2M7s2m8D9k6g2)9J5k6h3y4G2L8g2)9J5c8X3I4A6j5Y4u0S2M7Y4W2Q4x3V1k6S2M7X3y4Z5K9i4k6W2i4K6u0r3k6r3!0U0N6h3#2W2L8Y4c8S2N6r3W2G2L8W2)9J5c8W2m8W2M7X3k6G2M7X3#2S2L8X3y4W2i4K6u0r3b7$3!0F1j5$3g2H3N6s2g2S2L8q4)9J5c8V1y4G2k6r3g2r3L8$3!0@1M7s2u0A6L8Y4c8Q4x3V1k6m8M7Y4c8A6j5$3I4W2M7#2)9J5c8V1#2S2j5$3S2a6e0%4k6W2M7Y4k6A6k6i4N6Q4x3X3g2Z5N6r3#2D9)
- [Mach Object Files](https://bbs.kanxue.com/elink@457K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3y4A6L8r3W2F1k6r3g2J5i4K6u0W2j5X3g2Q4x3V1k6V1L8$3y4K6i4K6u0r3L8X3g2^5N6q4)9J5c8V1&6W2h3q4c8e0N6r3g2H3i4K6u0r3x3#2)9J5k6e0y4Q4x3V1k6F1k6q4)9J5c8V1c8W2N6W2c8G2L8$3I4K6i4K6u0r3x3e0c8Q4y4h3k6y4j5h3y4Z5e0#2)9J5c8V1#2S2j5$3S2a6i4K6u0W2K9s2c8E0L8r3c8Q4x3V1k6A6L8X3c8W2P5q4)9J5k6h3S2@1L8h3H3`.)
- [apple/darwin-xnu](https://bbs.kanxue.com/elink@460K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6S2M7s2m8D9k6g2)9J5c8X3c8S2M7Y4N6A6L8W2)9J5k6s2S2F1N6b7`.`.)
- [opensource-apple/dyld](https://bbs.kanxue.com/elink@ff5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6G2M7r3g2F1M7$3!0#2M7X3y4W2i4K6u0V1j5i4m8H3L8r3g2Q4x3V1k6V1P5h3I4V1)
- RTFSC

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于  2020-9-6 20:53

被evilpan编辑

，原因：

[#系统底层](https://bbs.kanxue.com/forum-4-1-2.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)
