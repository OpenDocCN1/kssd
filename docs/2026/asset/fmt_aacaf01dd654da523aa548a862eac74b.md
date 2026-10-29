
```
本函数中更详细的一些调用还可以看：
https://bbs.pediy.com/thread-264199.htm
主要讲了一下对应的COW相关的处理。
# 参考
*** 特别感谢povcfe学长的linux内存管理分析，让我学到了很多 ***
<mark class="encrypted">19dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2C8k6i4u0F1k6h3I4Q4x3X3g2G2M7X3N6Q4x3V1k6V1L8$3y4Q4x3V1k6Z5N6r3#2D9i4K6u0r3L8r3q4@1k6i4y4@1i4K6u0r3j5$3!0J5k6g2)9J5k6r3q4H3K9g2)9J5c8X3#2W2L8h3!0J5P5g2)9J5k6r3q4D9L8r3!0U0j5i4c8A6L8$3&6Q4x3X3g2Z5N6r3#2D9</mark>
<mark class="encrypted">44dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6X3M7X3q4F1K9$3c8W2L8X3&6W2L8h3q4F1i4K6u0W2L8X3I4Q4x3V1j5J5x3o6p5$3i4K6u0r3x3o6N6Q4x3V1j5H3y4#2)9J5c8X3&6#2L8h3q4Q4x3X3c8V1k6h3g2H3i4K6u0V1k6r3W2$3k6g2)9J5k6s2m8S2M7Y4c8Q4x3X3b7I4i4K6u0V1N6h3#2S2i4K6u0V1L8Y4g2E0j5g2)9J5c8R3`.`.</mark>
<mark class="encrypted">9d3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6\*7K9s2g2S2L8X3I4S2L8W2)9J5k6i4A6Z5K9h3S2#2i4K6u0W2j5$3!0E0i4K6u0r3M7q4)9J5c8U0j5^5y4o6j5#2z5e0f1J5</mark>
<mark class="encrypted">e7dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8X3A6S2M7$3!0F1j5$3S2W2L8W2)9#2k6X3N6T1k6q4)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3y4K6V1@1y4U0t1H3x3e0b7`.</mark>
<mark class="encrypted">274K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4A6Z5L8%4g2@1j5h3!0H3L8%4N6W2M7W2)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3z5o6M7H3z5e0l9&6z5o6t1`.</mark>
<mark class="encrypted">16bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4A6Z5L8%4g2@1j5h3!0H3L8%4N6W2M7W2)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3z5o6R3H3x3U0f1%4x3e0t1`.</mark>
<mark class="encrypted">608K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1j5H3P5r3q4^5i4K6u0W2k6$3W2@1j5X3!0G2K9%4y4Q4x3X3g2A6L8#2)9J5c8X3I4A6L8Y4g2^5i4K6u0V1K9h3&6K6K9h3c8W2M7#2)9J5c8X3y4G2L8Y4c8W2L8Y4c8Q4x3V1k6f1K9r3g2G2M7Y4W2Q4x3V1k6D9K9h3&6#2P5q4)9J5k6s2c8Z5k6h3!0J5P5g2)9J5k6o6q4Q4x3X3g2Z5N6r3#2D9</mark>
<mark class="encrypted">aa5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6\*7K9s2g2S2L8X3I4S2L8W2)9J5k6i4A6Z5K9h3S2#2i4K6u0W2j5$3!0E0i4K6u0r3M7q4)9J5c8U0p5K6y4K6t1%4y4K6M7J5y4l9`.`.</mark>
<mark class="encrypted">8feK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6k6h3N6E0k6h3&6@1k6X3q4#2L8s2c8Q4x3X3g2U0L8$3#2Q4x3V1k6S2i4K6u0r3x3e0p5&6x3o6l9H3x3o6l9I4x3U0t1$3z5e0t1@1z5b7`.`.</mark>
<mark class="encrypted">5baK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8$3c8W2L8X3!0F1k6#2)9J5k6h3y4G2L8g2)9J5c8X3y4K6x3e0l9#2z5e0R3@1y4e0j5@1i4K6u0r3</mark>
<mark class="encrypted">05fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6J5N6r3!0S2P5q4)9J5k6h3u0D9L8$3N6Q4x3X3g2U0M7$3c8F1i4K6u0W2L8X3g2@1i4K6u0r3j5i4u0@1K9h3y4D9k6g2)9J5c8X3c8W2N6r3q4A6L8s2y4Q4x3V1j5I4x3o6R3$3y4U0x3^5z5e0S2Q4x3@1k6#2N6r3#2Q4y4h3k6E0k6h3c8A6N6h3#2Q4x3@1c8V1K9i4y4@1M7X3W2T1N6i4c8W2i4K6u0W2M7r3y4Q4y4h3k6J5k6h3I4W2N6X3q4F1N6q4)9J5k6h3&6G2L8X3g2Q4x3X3c8@1j5i4y4C8i4K6u0V1j5X3I4G2k6#2)9J5k6o6u0Q4x3U0f1%4c8h3c8W2k6X3q4#2L8s2c8Q4x3U0f1%4c8f1u0D9L8$3N6o6L8$3#2E0k6h3&6V1c8Y4u0G2L8f1#2S2j5$3S2A6L8X3g2x3k6h3q4J5L8W2m8S2K9e0u0Q4x3U0f1%4c8h3c8W2k6X3q4#2L8s2c8Q4x3X3b7J5i4K6u0W2k6i4y4K6k6h3q4J5j5$3S2Q4y4h3k6H3j5#2)9#2k6Y4u0W2L8r3g2$3j5h3&6@1i4K6t1$3k6r3g2H3N6r3S2Q4y4h3j5I4i4K6u0V1N6i4c8E0i4K6g2X3M7$3!0#2M7X3y4W2i4K6y4p5k6r3W2K6N6s2u0A6j5Y4g2@1k6g2)9J5k6i4m8U0i4K6g2X3M7X3g2D9k6i4k6S2L8Y4c8Q4x3X3g2F1L8$3&6W2i4K6u0V1N6r3q4K6K9#2)9J5k6r3u0D9L8$3N6Q4x3X3b7J5i4K6t1#2y4@1g2V1k6h3k6S2N6h3I4@1i4K6t1#2y4@1g2n7L8r3!0Y4b7$3!0E0L8h3g2F1k6p5k6J5L8$3#2y4j5h3y4Z5K9h3&6W2e0r3g2S2M7X3&6b7j5h3V1J5i4K6t1#2y4@1g2V1k6h3k6S2N6h3I4@1i4K6u0V1x3W2)9J5k6h3g2K6M7$3g2S2M7X3y4Z5i4K6g2X3M7r3y4Q4y4h3k6J5k6h3I4W2N6X3q4F1N6l9`.`.</mark>
<mark class="encrypted">441K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6I4K9h3&6Y4L8r3W2F1L8h3q4G2z5o6x3I4y4g2)9J5k6h3N6A6N6r3S2#2j5W2)9J5k6h3W2G2i4K6u0r3L8r3W2F1N6i4S2Q4x3V1j5J5x3o6p5^5i4K6u0r3x3o6y4Q4x3V1j5I4y4q4)9J5c8X3I4A6L8Y4g2^5i4K6u0V1M7r3q4Y4k6g2)9J5k6r3y4S2j5$3S2W2i4K6u0W2K9s2c8E0L8l9`.`.</mark>
<mark class="encrypted">75eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2B7K9h3q4F1M7$3S2#2i4K6u0W2j5$3!0E0i4K6u0r3M7q4)9J5c8U0S2S2z5o6j5H3x3K6y4V1k6X3y4T1x3l9`.`.</mark>
<mark class="encrypted">021K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4N6Z5z5q4)9#2k6U0t1H3x3e0q4Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8U0f1K6x3e0x3^5x3K6M7%4</mark>
<mark class="encrypted">8beK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6\*7K9s2g2S2L8X3I4S2L8W2)9J5k6i4A6Z5K9h3S2#2i4K6u0W2j5$3!0E0i4K6u0r3M7q4)9J5c8U0t1#2z5o6V1J5x3e0b7#2x3#2)9K6c8Y4g2@1L8g2)9#2k6Y4y4G2N6i4u0U0k6g2)9K6c8s2N6W2j5$3S2S2N6q4)9#2k6Y4c8A6L8h3g2D9K9h3&6W2</mark>
<mark class="encrypted">c21K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8V1k6J5k6h3g2W2e0r3W2F1N6i4S2Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8U0f1@1y4K6f1@1y4K6f1J5</mark>
<mark class="encrypted">60fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2K6L8$3S2#2i4K6u0W2j5$3!0E0i4K6u0r3j5g2)9J5c8U0t1&6y4K6R3K6x3e0R3#2x3q4)9#2k6U0b7$3y4K6M7^5y4l9`.`.</mark>
<mark class="encrypted">ed8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3j5h3c8W2M7X3q4Q4x3V1k6H3i4K6u0r3x3e0p5%4x3e0R3%4y4U0g2Q4x3X3g2Z5N6r3#2D9</mark>
<mark class="encrypted">ae0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4A6Z5N6i4W2G2L8X3M7H3x3o6k6Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8U0p5H3x3o6M7K6y4K6M7J5y4l9`.`.</mark>
<mark class="encrypted">a59K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4N6S2L8X3N6I4N6h3q4F1x3e0V1&6x3W2)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3x3e0l9#2x3o6x3$3x3U0R3J5i4K6u0r3</mark>
<mark class="encrypted">bbeK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4y4&6K9%4m8G2N6i4u0Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8U0t1@1x3o6b7@1y4U0b7I4</mark>
<a href="elink@2d9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3N6r3!0D9K9h3#2A6N6q4)9J5c8Y4m8Q4x3V1j5#2y4o6x3J5y4U0M7@1i4K6u0W2K9s2c8E0L8l9`.`.">页迁移与碎片整理</a>

本文基于现今最新的Linux内核5.13版本。

内存管理模块一直是内核中最重要的模块之一，本文希望能简单的梳理内核内存管理模块的一部分核心内容，并结合我们在漏洞利用中的一些经验，以达到加深对内核理解的效果。

目录

- [开始之前](https://bbs.kanxue.com/#开始之前)
- [NUMA与UMA/SMP](https://bbs.kanxue.com/#numa与uma/smp)
- [层次化](https://bbs.kanxue.com/#层次化)
- [页表组织](https://bbs.kanxue.com/#页表组织)
- [一次页表查询](https://bbs.kanxue.com/#一次页表查询)
- [KPTI与内核页表](https://bbs.kanxue.com/#kpti与内核页表)
- [TLB缓存](https://bbs.kanxue.com/#tlb缓存)
- page cache 页缓冲
- Inverted page tables（IPT）
- Huge page
- THP(transparent huge page)
- [页表标识位](https://bbs.kanxue.com/#页表标识位)
- [伙伴系统](https://bbs.kanxue.com/#伙伴系统)
- [overview](https://bbs.kanxue.com/#overview)
- `alloc_pages(gfp_t gfp_mask, unsigned int order)`
- `__alloc_pages(gfp_t gfp, unsigned int order, int preferred_nid,nodemask_t *nodemask)`
- `get_page_from_freelist`（从zone freelist，快分配）
- [`__alloc_pages_slowpath`（慢分配）](https://bbs.kanxue.com/#__alloc_pages_slowpath（慢分配）)
- [`__free_pages`](https://bbs.kanxue.com/#__free_pages)
- [SLAB/SLUB分配器](https://bbs.kanxue.com/#slab/slub分配器)
- [关键结构体](https://bbs.kanxue.com/#关键结构体)
- [slab_hardened缓解/加固](https://bbs.kanxue.com/#slab_hardened缓解/加固)
- [`kmem_cache_alloc`](https://bbs.kanxue.com/#kmem_cache_alloc)
- [`kmem_cache_free`](https://bbs.kanxue.com/#kmem_cache_free)
- [`do_slab_free`（快速路径）](https://bbs.kanxue.com/#do_slab_free（快速路径）)
- [`__slab_free`（慢路径）](https://bbs.kanxue.com/#__slab_free（慢路径）)
- `check_object` 与 `CONFIG_SLUB`
- [进程](https://bbs.kanxue.com/#进程)
- Page Fault
- [`handle_page_fault`](https://bbs.kanxue.com/#handle_page_fault)
- [**do_kern_addr_fault**](https://bbs.kanxue.com/#**do_kern_addr_fault**)
- [**do_user_addr_fault**](https://bbs.kanxue.com/#**do_user_addr_fault**)
- [参考](https://bbs.kanxue.com/#参考)

# 开始之前

## NUMA与UMA/SMP

我们从最顶层的说起。

假设我们有3个CPU分别是C1，C2，C3

UMA/SMP：**Uniform-Memory-Access**，**均匀存储器存取**

可以简单理解为，C1，C2，C3作为一个整体，共享所有的物理内存。每台处理器可以有私有的高速cache。

[![hYeeoj.png](https://bbs.kanxue.com/upload/attach/202109/876323_F8AXH8EAT82HBNC.jpg)](https://bbs.kanxue.com/elink@c75K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6Z5h3h3g2W2L8$3Z5`.)

NUMA：**Nonuniform-Memory-Access**，**非均匀存储器存取**

对于处理器C1，C2，C3，他们不是“共享”内存的

[![hYellV.png](https://bbs.kanxue.com/upload/attach/202109/876323_ED3H9CAYGEPZHCC.jpg)](https://bbs.kanxue.com/elink@8ffK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6Z5h3h3g2D9L8q4j5`.)

具体地说，相对CPU1，连接到 CPU1 的内存控制器的内存被认为是本地内存。而连接到CPU2)的内存被视为 CPU1 的外部或远程内存。

由于远程内存访问比本地内存访问有额外的延迟开销，因为它必须遍历互连（点对点链接）并连接到远程内存控制器。由于内存位置不同，系统会经历“不均匀”的内存访问时间。

## 层次化

Linux内核中内存组织的层次化主要是经历了 **node->zone->page** 这样一个顺序。

[![hYe3OU.png](https://bbs.kanxue.com/upload/attach/202109/876323_4VR3BDNNZPJ4YUX.jpg)](https://bbs.kanxue.com/elink@0fbK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6Z5h3h3f1K6e0#2f1`.)

可以看到，每个CPU维护了自己对应的node，而这个node就可以理解为本地内存（NUMA）。

每个node又被划分为多个Zone。

Node 在内核源码中是一个全局数组。

|  |  |
| --- | --- |
| 1  2 | `/arch/x86/mm/numa.c`  `struct pglist_data` `*node_data[MAX_NUMNODES] __read_mostly;` |

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17 | `typedef struct pglist_data {`  `/*`  `* node_zones contains just the zones`  `for`  `THIS node. Not`  `all`  `of the`  `* zones may be populated, but it` `is` `the full` `list. It` `is` `referenced by`  `* this node's node_zonelists as well as other node'` `s node_zonelists.`  `*`  `struct zone node_zones[MAX_NR_ZONES];`  `/*`  `* node_zonelists contains references to` `all` `zones` `in` `all` `nodes.`  `* Generally the first zones will be references to this node's`  `* node_zones.`  `*`  `struct zonelist node_zonelists[MAX_ZONELISTS];`  `int` `nr_zones;` `/* number of populated zones` `in` `this node` `*`  `......` |

可以看到在对应的数组中，每一个 `pglist_data` 中都包含了对应的多个 `node_zones` 以及其对应的引用。

每个zone中维护了一些比较重要的结构。

- watermark（水位）

  - 表征了我们当前zone的page的使用情况，当watermark过高时，会自动启动内存回收算法。
- spanned_pages

  - 表征当前zone中含有的page frames数目。
- `long lowmem_reserve[MAX_NR_ZONES]` 一个动态的数组，主要功能是保留一些低位内存空间，以防止当在高区有大量可释放的内存，但我们却在低区启动了OOM。也是一种预留内存。
- zone_start_pfn：当前zone起始的物理页面号。而通过zone_start_pfn+spanned_pages可获得该zone的结束物理页面号。

  |  |  |
  | --- | --- |
  | 1 | `/* zone_start_pfn === zone_start_paddr >> PAGE_SHIFT */` |
- free_area：表征当前zone中还有多少空余可供分配的page frames。

值得一提的是，其实Zone也分为不同种类的Zone（类比slab），可以通过如下方式查看：

|  |  |
| --- | --- |
| 1  2  3  4  5  6 | `root@ubuntu:~# cat /proc/zoneinfo |grep Node`  `Node 0, zone      DMA`  `Node 0, zone    DMA32`  `Node 0, zone   Normal`  `Node 0, zone  Movable`  `Node 0, zone   Device` |

接下来说说page（页），page frame（页帧/框）。这两个的关系类似鸡蛋（page）与篮子（page frame）的关系。

一般来说，一个page的大小是4K，是管理物理内存的最小单位。

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28  29  30  31  32  33  34  35  36  37  38  39  40  41  42  43  44  45  46  47  48  49  50  51  52  53  54  55  56  57  58  59  60  61  62  63  64  65  66  67  68  69  70  71  72  73  74  75  76  77  78  79  80  81  82  83  84  85  86  87  88  89  90  91  92  93  94  95  96  97  98  99  100  101  102  103  104  105  106  107  108  109  110  111  112  113  114  115  116  117  118  119  120  121 | `//mm_types.h`  `struct page {`  `unsigned long flags;` `/* Atomic flags, some possibly`  `* updated asynchronously` `*/`  `union {`  `struct {` `/* Page cache` `and` `anonymous pages` `*/`  `struct list_head lru;`  `/* See page-flags.h` `for` `PAGE_MAPPING_FLAGS` `*/`  `struct address_space` `*mapping;`  `pgoff_t index;` `/* Our offset within mapping.` `*/`  `/*`  `* @private: Mapping-private opaque data.`  `* Usually used` `for` `buffer_heads` `if` `PagePrivate.`  `* Used` `for` `swp_entry_t` `if` `PageSwapCache.`  `* Indicates order` `in` `the buddy system` `if` `PageBuddy.`  `*/`  `unsigned long private;`  `};`  `struct {` `/* page_pool used by netstack` `*/`  `/*`  `* @dma_addr: might require a` `64-bit value on`  `* 32-bit architectures.`  `*/`  `unsigned long dma_addr[2];`  `};`  `struct {` `/* slab, slob` `and` `slub` `*/`  `union {`  `struct list_head slab_list;`  `struct {` `/* Partial pages` `*/`  `struct page` `*next;`  `#ifdef CONFIG_64BIT`  `int` `pages;` `/* Nr of pages left` `*/`  `int` `pobjects;` `/* Approximate count` `*/`  `#else`  `short` `int` `pages;`  `short` `int` `pobjects;`  `#endif`  `};`  `};`  `struct kmem_cache` `*slab_cache;` `/* not` `slob` `*/`  `/* Double-word boundary` `*/`  `void` `*freelist;` `/* first free` `object` `*/`  `union {`  `void` `*s_mem;` `/* slab: first` `object` `*/`  `unsigned long counters;` `/* SLUB` `*/`  `struct {` `/* SLUB` `*/`  `unsigned inuse:16;`  `unsigned objects:15;`  `unsigned frozen:1;`  `};`  `};`  `};`  `struct {` `/* Tail pages of compound page` `*/`  `unsigned long compound_head;` `/* Bit zero` `is` `set` `*/`  `/* First tail page only` `*/`  `unsigned char compound_dtor;`  `unsigned char compound_order;`  `atomic_t compound_mapcount;`  `unsigned` `int` `compound_nr;` `/* 1` `<< compound_order` `*/`  `};`  `struct {` `/* Second tail page of compound page` `*/`  `unsigned long _compound_pad_1;` `/* compound_head` `*/`  `atomic_t hpage_pinned_refcount;` `/* For both` `global` `and` `memcg` `*/`  `struct list_head deferred_list;`  `};`  `struct {` `/* Page table pages` `*/`  `unsigned long _pt_pad_1;` `/* compound_head` `*/`  `pgtable_t pmd_huge_pte;` `/* protected by page->ptl` `*/`  `unsigned long _pt_pad_2;` `/* mapping` `*/`  `union {`  `struct mm_struct` `*pt_mm;` `/* x86 pgds only` `*/`  `atomic_t pt_frag_refcount;` `/* powerpc` `*/`  `};`  `#if ALLOC_SPLIT_PTLOCKS`  `spinlock_t` `*ptl;`  `#else`  `spinlock_t ptl;`  `#endif`  `};`  `struct {` `/* ZONE_DEVICE pages` `*/`  `/*`  `@pgmap: Points to the hosting device page` `map.`  `*/`  `struct dev_pagemap` `*pgmap;`  `void` `*zone_device_data;`  `};`  `/*`  `@rcu_head: You can use this to free a page by RCU.`  `*/`  `struct rcu_head rcu_head;`  `};`  `union {` `/* This union` `is` `4` `bytes` `in` `size.` `*/`  `atomic_t _mapcount;`  `/*`  `* If the page` `is` `neither PageSlab nor mappable to userspace,`  `* the value stored here may` `help` `determine what this page`  `* is` `used` `for. See page-flags.h` `for` `a` `list` `of page types`  `* which are currently stored here.`  `*/`  `unsigned` `int` `page_type;`  `unsigned` `int` `active;` `/* SLAB` `*/`  `int` `units;` `/* SLOB` `*/`  `};`  `/* Usage count.`  `*DO NOT USE DIRECTLY*. See page_ref.h`  `*/`  `atomic_t _refcount;`  `#ifdef CONFIG_MEMCG`  `unsigned long memcg_data;`  `#endif`  `#if defined(WANT_PAGE_VIRTUAL)`  `void` `*virtual;` `/* Kernel virtual address (NULL` `if`  `not` `kmapped, ie. highmem)` `*/`  `#endif /* WANT_PAGE_VIRTUAL */`  `#ifdef LAST_CPUPID_NOT_IN_PAGE_FLAGS`  `int _last_cpupid;`  `#endif`  `} _struct_page_alignment;` |

我们主要聊里面几个重要的成员：

- flags: 标定了page frame一些相应的属性。

  flags的格式如下：

  [![hYeJw4.png](https://bbs.kanxue.com/upload/attach/202109/876323_AK3QU7ZNCEBYPW7.jpg)](https://bbs.kanxue.com/elink@033K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6Z5h3h3g2v1N6K6b7`.)

我们主要关注最后一位flag，用于标识page的状态。

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28  29  30  31  32  33  34  35  36  37  38  39  40  41  42  43  44  45  46  47  48  49  50  51  52  53  54  55  56  57  58  59  60  61  62  63  64  65  66  67  68  69  70  71  72  73  74  75  76 | ```c ```  `enum pageflags {`  `PG_locked,` `/* Page` `is` `locked. Don't touch.` `*/`  `PG_referenced,``/``/``表示page刚刚被访问过`  `PG_uptodate,`  `PG_dirty,` `/* 是否页面数据已经被修改（脏页）`  `PG_lru,` `/* 是否处于lru链表中`  `PG_active,`  `PG_workingset,`  `PG_waiters,`  `PG_error,`  `PG_slab,` `/* 是否属于slab分配器`  `PG_owner_priv_1,` `/* Owner use. If pagecache, fs may use` `*/`  `PG_arch_1,`  `PG_reserved,`  `PG_private,` `/* If pagecache, has fs-private data` `*/`  `PG_private_2,` `/* If pagecache, has fs aux data` `*/`  `PG_writeback,` `/* page正在被写回`  `PG_head,` `/* A head page` `*/`  `PG_mappedtodisk,` `/* Has blocks allocated on-disk` `*/`  `PG_reclaim,` `/* To be reclaimed asap` `*/`  `PG_swapbacked,` `/* Page` `is` `backed by RAM/swap` `*/`  `PG_unevictable,` `/* Page` `is` `"unevictable"`  `*/`  `#ifdef CONFIG_MMU`  `PG_mlocked,` `/* Page` `is` `vma mlocked` `*/`  `#endif`  `#ifdef CONFIG_ARCH_USES_PG_UNCACHED`  `PG_uncached,` `/* Page has been mapped as uncached` `*/`  `#endif`  `#ifdef CONFIG_MEMORY_FAILURE`  `PG_hwpoison,` `/* hardware poisoned page. Don't touch` `*/`  `#endif`  `#if defined(CONFIG_IDLE_PAGE_TRACKING) && defined(CONFIG_64BIT)`  `PG_young,`  `PG_idle,`  `#endif`  `#ifdef CONFIG_64BIT`  `PG_arch_2,`  `#endif`  `__NR_PAGEFLAGS,`  `/* Filesystems` `*/`  `PG_checked` `=` `PG_owner_priv_1,`  `/* SwapBacked` `*/`  `PG_swapcache` `=` `PG_owner_priv_1,` `/* page处于swap cache中`  `/* Swap page: swp_entry_t` `in` `private` `*/`  `/* Two page bits are conscripted by FS-Cache to maintain local caching`  `* state.  These bits are` `set` `on pages belonging to the netfs's inodes`  `* when those inodes are being locally cached.`  `*/`  `PG_fscache` `=` `PG_private_2,` `/* page backed by cache` `*/`  `/* XEN` `*/`  `/* Pinned` `in` `Xen as a read-only pagetable page.` `*/`  `PG_pinned` `=` `PG_owner_priv_1,`  `/* Pinned as part of domain save (see xen_mm_pin_all()).` `*/`  `PG_savepinned` `=` `PG_dirty,`  `/* Has a grant mapping of another (foreign) domain's page.` `*/`  `PG_foreign` `=` `PG_owner_priv_1,`  `/* Remapped by swiotlb-xen.` `*/`  `PG_xen_remapped` `=` `PG_owner_priv_1,`  `/* SLOB` `*/`  `PG_slob_free` `=` `PG_private,`  `/* Compound pages. Stored` `in` `first tail page's flags` `*/`  `PG_double_map` `=` `PG_workingset,`  `/* non-lru isolated movable page` `*/`  `PG_isolated` `=` `PG_reclaim,`  `/* Only valid` `for` `buddy pages. Used to track pages that are reported` `*/`  `PG_reported` `=` `PG_uptodate,`  `};` |
