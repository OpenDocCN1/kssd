
8. [Linux中的物理内存管理 [二]](<mark class="encrypted">393K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6\*7K9s2g2S2L8X3I4S2L8W2)9J5k6i4A6Z5K9h3S2#2i4K6u0W2j5$3!0E0i4K6u0r3M7q4)9J5c8U0j5^5y4o6M7K6y4o6t1^5i4K6t1&6</mark>

9. <a href="elink@df8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8W2)9J5k6i4m8A6L8X3N6U0j5i4m8Q4x3X3g2U0L8$3#2Q4x3V1k6T1L8r3!0Y4i4K6u0r3L8r3W2F1N6i4S2Q4x3X3c8C8k6i4u0F1k6h3I4Q4x3X3c8$3M7#2)9J5k6r3#2W2L8h3!0J5P5g2)9J5k6r3k6J5j5h3N6E0k6h3&6@1j5i4c8A6L8$3&6Q4x3X3b7I4">Linux Kernel vs. Memory Fragmentation (Part I)</a>

10. <a href="elink@b39K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3I4A6L8Y4g2^5i4K6u0W2L8r3q4G2M7h3W2F1M7X3g2F1i4K6u0W2L8X3g2@1i4K6u0r3K9$3g2J5L8X3g2D9i4K6u0r3L8X3!0V1k6g2)9#2k6Y4y4@1j5i4c8W2M7#2)9J5c8R3`.`.">描述系统上cpu和memory的状态:`node_states`</a>

11. <a href="elink@018K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4N6W2K9i4S2A6L8W2)9#2k6U0b7J5x3U0j5J5z5e0b7@1i4K6u0r3j5i4u0@1K9h3y4D9k6g2)9J5c8X3c8W2N6r3q4A6L8s2y4Q4x3V1j5I4x3e0V1^5y4e0x3^5y4o6j5`.">Linux内存子系统——分配物理页面（`alloc_pages`）</a>

12. **<a href="elink@d9bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4W2Z5j5U0p5H3y4o6M7^5x3e0R3K6z5o6c8Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8U0p5I4x3U0t1&6z5o6V1&6y4R3`.`.">Linux内存管理(六): 分配物理内存`alloc_pages`</a>**

13. <a href="elink@079K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8X3I4G2L8X3N6%4j5h3&6Y4x3e0f1#2x3o6j5&6i4K6u0r3j5i4u0@1K9h3y4D9k6g2)9J5c8X3c8W2N6r3q4A6L8s2y4Q4x3V1j5I4x3o6f1#2z5e0f1%4y4U0x3`.">从备用类型中steal page</a>

本文首发于安全客——[链接](https://bbs.kanxue.com/elink@8aeK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2L8Y4q4#2j5h3&6C8k6g2)9J5k6h3y4G2L8g2)9J5c8Y4m8G2M7%4c8Q4x3V1k6A6k6q4)9J5c8U0t1$3x3o6j5K6y4l9`.`.)

# 一、前言

本文基于如下环境：

- Kernel Version：5.11.0
- Debugging Env：Ubuntu 20.04.02 x64(Kernel Version—5.11.0)

近来笔者计划从脏牛漏洞入手，分析Linux内核漏洞，故在开始之前学习了Linux内核中内存管理部分相关内容，下文权当笔者学习过程整理的笔记。如有不当之处，望读者不吝赐教。

# 二、层次组织

## 0x01 Node

UMA——Uniform Memory Access，NUMA——Non-uniform memory access：

![](https://bbs.kanxue.com/upload/attach/202111/817966_22DRGEE2BKCVRTT.jpg)

![](https://bbs.kanxue.com/upload/attach/202111/817966_C4XV92C7SMHGRHT.jpg)

可进一步阅读[Non-uniform memory access—Wikipedia](https://bbs.kanxue.com/elink@64fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8W2)9J5k6i4N6A6K9$3W2H3k6h3c8A6j5g2)9J5k6h3!0J5k6#2)9J5c8Y4N6A6K9$3W2Q4x3V1k6z5L8$3&6Q4x3X3c8#2L8X3W2X3L8%4u0E0i4K6g2X3L8h3g2E0L8%4u0&6i4K6g2X3j5h3y4U0k6i4y4K6)。Linux为每个节点定义了一个类型为`pg_data_t`的描述符：

```c
/*
 * On NUMA machines, each NUMA node would have a pg_data_t to describe
 * it's memory layout. On UMA machines there is a single pglist_data which
 * describes the whole memory.
 *
 * Memory statistics and page replacement data structures are maintained on a
 * per-zone basis.
 */
typedef struct pglist_data {
	/*
	 * node_zones contains just the zones for THIS node. Not all of the
	 * zones may be populated, but it is the full list. It is referenced by
	 * this node's node_zonelists as well as other node's node_zonelists.
	 */
	struct zone node_zones[MAX_NR_ZONES];

	/*
	 * node_zonelists contains references to all zones in all nodes.
	 * Generally the first zones will be references to this node's
	 * node_zones.
	 */
	struct zonelist node_zonelists[MAX_ZONELISTS];

	int nr_zones; /* number of populated zones in this node */
#ifdef CONFIG_FLAT_NODE_MEM_MAP    /* means !SPARSEMEM */
	struct page *node_mem_map;
#ifdef CONFIG_PAGE_EXTENSION
	struct page_ext *node_page_ext;
#endif
#endif
#if defined(CONFIG_MEMORY_HOTPLUG) || defined(CONFIG_DEFERRED_STRUCT_PAGE_INIT)
	/*
	 * Must be held any time you expect node_start_pfn,
	 * node_present_pages, node_spanned_pages or nr_zones to stay constant.
	 * Also synchronizes pgdat->first_deferred_pfn during deferred page
	 * init.
	 *
	 * pgdat_resize_lock() and pgdat_resize_unlock() are provided to
	 * manipulate node_size_lock without checking for CONFIG_MEMORY_HOTPLUG
	 * or CONFIG_DEFERRED_STRUCT_PAGE_INIT.
	 *
	 * Nests above zone->lock and zone->span_seqlock
	 */
	spinlock_t node_size_lock;
#endif
	unsigned long node_start_pfn;
	unsigned long node_present_pages; /* total number of physical pages */
	unsigned long node_spanned_pages; /* total size of physical page range, including holes */
	int node_id;
	wait_queue_head_t kswapd_wait;
	wait_queue_head_t pfmemalloc_wait;
	struct task_struct *kswapd; /* Protected by mem_hotplug_begin/end() */
	int kswapd_order;
	enum zone_type kswapd_highest_zoneidx;

	int kswapd_failures; /* Number of 'reclaimed == 0' runs */

#ifdef CONFIG_COMPACTION
	int kcompactd_max_order;
	enum zone_type kcompactd_highest_zoneidx;
	wait_queue_head_t kcompactd_wait;
	struct task_struct *kcompactd;
#endif
	/*
	 * This is a per-node reserve of pages that are not available
	 * to userspace allocations.
	 */
	unsigned long        totalreserve_pages;

#ifdef CONFIG_NUMA
	/*
	 * node reclaim becomes active if more unmapped pages exist.
	 */
	unsigned long        min_unmapped_pages;
	unsigned long        min_slab_pages;
#endif /* CONFIG_NUMA */

	/* Write-intensive fields used by page reclaim */
	ZONE_PADDING(_pad1_)

#ifdef CONFIG_DEFERRED_STRUCT_PAGE_INIT
	/*
	 * If memory initialisation on large machines is deferred then this
	 * is the first PFN that needs to be initialised.
	 */
	unsigned long first_deferred_pfn;
#endif /* CONFIG_DEFERRED_STRUCT_PAGE_INIT */

#ifdef CONFIG_TRANSPARENT_HUGEPAGE
	struct deferred_split deferred_split_queue;
#endif

	/* Fields commonly accessed by the page reclaim scanner */

	/*
	 * NOTE: THIS IS UNUSED IF MEMCG IS ENABLED.
	 *
	 * Use mem_cgroup_lruvec() to look up lruvecs.
	 */
	struct lruvec        __lruvec;

	unsigned long        flags;

	ZONE_PADDING(_pad2_)

	/* Per-node vmstats */
	struct per_cpu_nodestat __percpu *per_cpu_nodestats;
	atomic_long_t        vm_stat[NR_VM_NODE_STAT_ITEMS];
} pg_data_t;
```

而每个节点又可以划分为数个内存管理区——ZONE，与之相关的字段如下：

```c
struct zone node_zones[MAX_NR_ZONES]; //内存管理区描述符数组
struct zonelist node_zonelists[MAX_ZONELISTS]; //引用所有节点中内存管理区
int nr_zones; //当前节点中内存管理区数量
```

与当前节点相关字段：

```c
unsigned long node_start_pfn; //节点第一个页框的PFN
unsigned long node_present_pages; /* total number of physical pages */
unsigned long node_spanned_pages; /* total size of physical page range, including holes */
int node_id; //节点标识符
```

与`kswapd`内核线程相关字段：

```c
wait_queue_head_t kswapd_wait;
wait_queue_head_t pfmemalloc_wait;
struct task_struct *kswapd; /* Protected by mem_hotplug_begin/end() */
int kswapd_order;
enum zone_type kswapd_highest_zoneidx;

int kswapd_failures; /* Number of 'reclaimed == 0' runs */
```

## 0x02 Zone

内存管理区描述符定义如下：

