
# [原创]green-dill: 一个用于linux kernel调试的gdb 插件

`dill` 是一个用于linux kernel调试的gdb 插件，基于gdb 的python api开发。

项目地址：<mark class="encrypted">7e7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6p5k6e0c8V1b7%4t1H3N6#2)9J5c8X3N6J5k6h3g2F1i4K6u0V1k6r3W2D9L8l9`.`.</mark>

主要功能包括如下：

（1）`kbase` 获取当前的内核基址：

![](upload/attach/202005/589842_52A4UWZY58NWMG9.jpg)

（2） `kstruct` 得到特定的数据结构，对于结构中存在`struct`和`union`类型的，进行递归解析，展开整个结构：

![](upload/attach/202005/589842_5463EY3T8BFS4SZ.jpg)

以及某个字段在数据结构中的偏移：

![](upload/attach/202005/589842_VQGF8C39QQJSZ93.jpg)

（3）`ktask` 获取某个进程的`task_struct` 地址

![](upload/attach/202005/589842_NYBZP4TS92CBZZH.jpg)

（4）`kcache` 获得特定类型`kmem_cache` 的`freelist`, `partial`

![](upload/attach/202005/589842_ARX64BDCHG6KMGJ.jpg)

输出主要分成四个部分：

1. 关于`kmem_cache`结构的一些信息
2. `kmem_cache_cpu` 上的`freelist`
3. `kmem_cache_cpu` partial上各个page挂的`freelist`
4. `kmem_cache_node` partial上各个page挂的`freelist`

以上属于插件静态解析部分，主要根据各个数据结构之间的关系进行解析

（5）`dill` 跟踪某个进程的堆分配，以及释放，该部分主要是对分配和释放函数打上断点，在函数开头下断点时获取函数参数，在函数结束下断点时获取返回值，以此来获取需要的信息。

![](upload/attach/202005/589842_RC4UUVT5M6SHKNZ.jpg)

`dill` 模块的功能包括：

（1）`dill run` ：对要跟踪的函数打上断点

（2）`dill process_name` : 跟踪对应进程的堆分配和释放，会输出分配/释放的slab类型，以及堆块地址

（3）`dill on/off` ：对输出日志进行开启/关闭

各个结构之间的数据关系图：

![](upload/attach/202005/589842_B5T6CVY7EA7QSA5.png)

**（注：上图来源网上）**

内核管理页面使用了2个算法：伙伴算法和slub算法，伙伴算法以页为单位管理内存，但在大多数情况下，程序需要的并不是一整页，而是几个、几十个字节的小内存。于是需要另外一套系统来完成对小内存的管理，这就是slub系统。slub系统运行在伙伴系统之上，为内核提供小内存管理的功能。

通过`cat /proc/slabinfo` 查看系统当前可用的`kmem_cache`，目前我们关注一般`kmalloc`申请空间的类型：

![](upload/attach/202005/589842_PGH8T6MQMCFUSA3.png)

每个`kmem_cache`结构都表示一种类型，`kmem_cache->name`表示cache的名称，`kmem_cache->size`表示cache的大小。每个cache通过`kmem_cache->list.next`指向下一个cache，形成一个链表。我们找到全局变量 `slab_caches`就能遍历该链表，找到对应的cache信息。

内存分配释放相关的结构如下：

```
kmem_cache
- kmem_cache_cpu
- freelist
- partial
- kmem_cache_node
- partial
```

### kmem_cache_cpu freelist

`freelist`成员指向第一个可用内存obj首地址。类似fastbin，在开头保存下一个obj的地址，采用 FIFO 进行分配和回收

### kmem_cache_cpu partial

`kmem_cache_cpu->partial`指向的是一个page，每个page通过`page->next`链接，形成链表，而每个page下都挂着一个`freelist`，如果`kmem_cache_cpu` freelist 上的空闲obj分配完就会从这分配。

### kmem_cache_node partial

`kmem_cache_node->partial`指向的也是一个page，但每个page是通过`page->lru`链接，其他和`kmem_cache_cpu` partial类似。值得注意的是node中的slab是所有cpu共享的，而per cpu是每个cpu独占的。

### kmem_cache_node full

`kmem_cache_node->full` 指向的是自身的地址，full slab没有链表来管理，只有full变成partial才会被接入partial链中。

### 总结

`kmalloc`分配内存顺序如下：`kmem_cache_cpu` freelist -> `kmem_cache_cpu` partial -> p`kmem_cache_node` partial

如果前一个没有空闲的object，依次找下一个。

目前实验环境下插件可以正常运行，但跟踪复杂程序时，在释放的地方下断点容易崩，后面再慢慢完善。

## 参考链接

<mark class="encrypted">0dfK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6b7j5h3!0D9L8@1#2G2L8Y4c8A6y4o6u0Q4x3V1k6K6j5h3I4@1i4K6u0r3N6s2u0W2k6g2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8X3c8G2j5%4x3`.</mark>

<mark class="encrypted">413K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6L8%4g2J5j5$3g2%4j5i4u0W2i4K6u0W2L8%4u0Y4i4K6u0r3k6$3c8T1i4K6u0r3L8$3&6D9K9h3&6W2k6r3!0U0M7#2)9J5c8X3N6V1j5W2)9J5c8W2m8&6N6r3S2G2L8W2)9J5k6p5q4b7d9g2)9J5k6h3S2@1L8h3H3`.</mark>

<mark class="encrypted">a42K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2L8Y4q4#2j5h3&6C8k6g2)9J5k6h3y4G2L8g2)9J5c8Y4m8G2M7%4c8Q4x3V1k6A6k6q4)9J5c8U0t1H3x3o6p5$3x3b7`.`.</mark>

<mark class="encrypted">1a4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0P5g2)9J5k6h3!0K6j5$3S2A6L8X3q4Q4x3X3g2F1k6i4c8Q4x3V1k6X3K9h3I4W2L8%4m8@1K9h3!0F1M7#2)9J5c8X3u0D9L8$3N6Q4x3V1j5I4y4U0x3H3x3K6b7$3</mark>

`dill` 是一个用于linux kernel调试的gdb 插件，基于gdb 的python api开发。

项目地址：a5aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6p5k6e0c8V1b7%4t1H3N6#2)9J5c8X3N6J5k6h3g2F1i4K6u0V1k6r3W2D9L8l9`.`.

主要功能包括如下：

（1）`kbase` 获取当前的内核基址：

![](https://bbs.kanxue.com/upload/attach/202005/589842_52A4UWZY58NWMG9.jpg)

（2） `kstruct` 得到特定的数据结构，对于结构中存在`struct`和`union`类型的，进行递归解析，展开整个结构：

![](https://bbs.kanxue.com/upload/attach/202005/589842_5463EY3T8BFS4SZ.jpg)

以及某个字段在数据结构中的偏移：

![](https://bbs.kanxue.com/upload/attach/202005/589842_VQGF8C39QQJSZ93.jpg)

（3）`ktask` 获取某个进程的`task_struct` 地址

![](https://bbs.kanxue.com/upload/attach/202005/589842_NYBZP4TS92CBZZH.jpg)

（4）`kcache` 获得特定类型`kmem_cache` 的`freelist`, `partial`

![](https://bbs.kanxue.com/upload/attach/202005/589842_ARX64BDCHG6KMGJ.jpg)

输出主要分成四个部分：

1. 关于`kmem_cache`结构的一些信息
2. `kmem_cache_cpu` 上的`freelist`
3. `kmem_cache_cpu` partial上各个page挂的`freelist`
4. `kmem_cache_node` partial上各个page挂的`freelist`

以上属于插件静态解析部分，主要根据各个数据结构之间的关系进行解析

（5）`dill` 跟踪某个进程的堆分配，以及释放，该部分主要是对分配和释放函数打上断点，在函数开头下断点时获取函数参数，在函数结束下断点时获取返回值，以此来获取需要的信息。

![](https://bbs.kanxue.com/upload/attach/202005/589842_RC4UUVT5M6SHKNZ.jpg)

`dill` 模块的功能包括：

（1）`dill run` ：对要跟踪的函数打上断点

（2）`dill process_name` : 跟踪对应进程的堆分配和释放，会输出分配/释放的slab类型，以及堆块地址

（3）`dill on/off` ：对输出日志进行开启/关闭

各个结构之间的数据关系图：

![](https://bbs.kanxue.com/upload/attach/202005/589842_B5T6CVY7EA7QSA5.png)

**(注：上图来源网上)**

内核管理页面使用了2个算法：伙伴算法和slub算法，伙伴算法以页为单位管理内存，但在大多数情况下，程序需要的并不是一整页，而是几个、几十个字节的小内存。于是需要另外一套系统来完成对小内存的管理，这就是slub系统。slub系统运行在伙伴系统之上，为内核提供小内存管理的功能。

通过`cat /proc/slabinfo` 查看系统当前可用的`kmem_cache`，目前我们关注一般`kmalloc`申请空间的类型：

![](https://bbs.kanxue.com/upload/attach/202005/589842_PGH8T6MQMCFUSA3.png)

每个`kmem_cache`结构都表示一种类型，`kmem_cache->name`表示cache的名称，`kmem_cache->size`表示cache的大小。每个cache通过`kmem_cache->list.next`指向下一个cache，形成一个链表。我们找到全局变量 `slab_caches`就能遍历该链表，找到对应的cache信息。

内存分配释放相关的结构如下：

```
kmem_cache

    - kmem_cache_cpu 

        - freelist

        - partial

    - kmem_cache_node

        - partial

```

### kmem_cache_cpu freelist

`freelist`成员指向第一个可用内存obj首地址。类似fastbin，在开头保存下一个obj的地址，采用 FIFO 进行分配和回收

### kmem_cache_cpu partial

`kmem_cache_cpu->partial`指向的是一个page，每个page通过`page->next`链接，形成链表，而每个page下都挂着一个`freelist`，如果`kmem_cache_cpu` freelist 上的空闲obj分配完就会从这分配。

### kmem_cache_node partial

`kmem_cache_node->partial`指向的也是一个page，但每个page是通过`page->lru`链接，其他和`kmem_cache_cpu` partial类似。值得注意的是node中的slab是所有cpu共享的，而per cpu是每个cpu独占的。

### kmem_cache_node full

`kmem_cache_node->full` 指向的是自身的地址，full slab没有链表来管理，只有full变成partial才会被接入partial链中。

### 总结

`kmalloc`分配内存顺序如下：`kmem_cache_cpu` freelist -> `kmem_cache_cpu` partial -> p`kmem_cache_node` partial

如果前一个没有空闲的object，依次找下一个。

目前实验环境下插件可以正常运行，但跟踪复杂程序时，在释放的地方下断点容易崩，后面再慢慢完善。

## 参考链接

5a8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6b7j5h3!0D9L8@1#2G2L8Y4c8A6y4o6u0Q4x3V1k6K6j5h3I4@1i4K6u0r3N6s2u0W2k6g2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8X3c8G2j5%4x3`.

a04K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6L8%4g2J5j5$3g2%4j5i4u0W2i4K6u0W2L8%4u0Y4i4K6u0r3k6$3c8T1i4K6u0r3L8$3&6D9K9h3&6W2k6r3!0U0M7#2)9J5c8X3N6V1j5W2)9J5c8W2m8&6N6r3S2G2L8W2)9J5k6p5q4b7d9g2)9J5k6h3S2@1L8h3H3`.

2dfK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2L8Y4q4#2j5h3&6C8k6g2)9J5k6h3y4G2L8g2)9J5c8Y4m8G2M7%4c8Q4x3V1k6A6k6q4)9J5c8U0t1H3x3o6p5$3x3b7`.`.

0f9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0P5g2)9J5k6h3!0K6j5$3S2A6L8X3q4Q4x3X3g2F1k6i4c8Q4x3V1k6X3K9h3I4W2L8%4m8@1K9h3!0F1M7#2)9J5c8X3u0D9L8$3N6Q4x3V1j5I4y4U0x3H3x3K6b7$3

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2020-5-7 18:51

被笔墨编辑，原因： 添加原创
