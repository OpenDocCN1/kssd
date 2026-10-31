
```C++
#define XA_ZERO_ENTRY xa_mk_internal(257)
/**
 * xa_is_zero() - Is the entry a zero entry?
 * @entry: Entry retrieved from the XArray
 *
 * The normal API will return NULL as the contents of a slot containing
 * a zero entry. You can only see zero entries by using the advanced API.
 *
 * Return: %true if the entry is a zero entry.
 */
static inline bool xa_is_zero(const void *entry)
{
	return unlikely(entry == XA_ZERO_ENTRY);
}

static inline __must_check
int xa_reserve(struct xarray *xa, unsigned long index, gfp_t gfp)
{
	return xa_err(xa_cmpxchg(xa, index, NULL, XA_ZERO_ENTRY, gfp));
}
```

#### Error Entry(错误条目)

- 功能：表示操作错误（如 `ENOMEM`）
- 编码：`xa_mk_internal(负数)`，范围 -4094 到 -2

```C++
static inline bool xa_is_err(const void *entry)
{
	return unlikely(xa_is_internal(entry) &&
			entry >= xa_mk_internal(-MAX_ERRNO));
}
```

#### Value Entry(第一位为1)

> 值条目，用于存储 swap/shadow 条目或标记指针。

- 场景：内存不足时，Page Cache 中的页面被回收，但需要记录“这里曾经有页面”。
- 编码紧凑，可携带回收时间、节点ID、memcg ID 等信息

##### Value Entry(纯值条目)

- 功能：存储整数值而非指针

```C++
static inline void *xa_mk_value(unsigned long v)
{
	WARN_ON((long)v < 0);
	return (void *)((v << 1) | 1);
}
/**
 * xa_to_value() - Get value stored in an XArray entry.
 * @entry: XArray entry.
 *
 * Context: Any context.
 * Return: The value stored in the XArray entry.
 */
static inline unsigned long xa_to_value(const void *entry)
{
	return (unsigned long)entry >> 1;
}
/**
 * xa_is_value() - Determine if an entry is a value.
 * @entry: XArray entry.
 *
 * Context: Any context.
 * Return: True if the entry is a value, false if it is a pointer.
 */
static inline bool xa_is_value(const void *entry)
{
	return (unsigned long)entry & 1;
}
// 列子
// 创建值条目
void *entry = xa_mk_value(123); // 将整数123编码为值条目
// 提取值
unsigned long value = xa_to_value(entry); // 返回123
// 判断是否为值条目
if (xa_is_value(entry)) {
	// 这是值条目，不是指针
}
```

##### Tagged Pointer(标记指针)

- 功能：在指针上附加标记位（0、1 或 3）

### PerBDI

> 现代Linux内核中，`bdflush`已被per-BDI（Backing Device Info）的写回线程替代。每个BDI有一个或多个`bdi_writeback`结构，每个`bdi_writeback`对应一个写回线程（通过workqueue实现）。通过这个机制来负责的 Linux 的脏数据回写。

#### `bdi_writeback` （核心数据结构）

```C++
struct bdi_writeback {
	struct backing_dev_info *bdi;	/* our parent bdi */
	unsigned long state;	/* Always use atomic bitops on this */
	unsigned long last_old_flush;	/* last old data flush */
	struct list_head b_dirty;	/* dirty inodes */
	struct list_head b_io;	/* parked for writeback */
	struct list_head b_more_io;	/* parked for more writeback */
	struct list_head b_dirty_time;	/* time stamps are dirty */
	spinlock_t list_lock;	/* protects the b_* lists */
	atomic_t writeback_inodes;	/* number of inodes under writeback */
	struct percpu_counter stat[NR_WB_STAT_ITEMS];
	unsigned long bw_time_stamp;	/* last time write bw is updated */
	unsigned long dirtied_stamp;
	unsigned long written_stamp;	/* pages written at bw_time_stamp */
	unsigned long write_bandwidth;	/* the estimated write bandwidth */
	unsigned long avg_write_bandwidth; /* further smoothed write bw, > 0 */
	/*
	 * The base dirty throttle rate, re-calculated on every 200ms.
	 * All the bdi tasks' dirty rate will be curbed under it.
	 * @dirty_ratelimit tracks the estimated @balanced_dirty_ratelimit
	 * in small steps and is much more smooth/stable than the latter.
	 */
	unsigned long dirty_ratelimit;
	unsigned long balanced_dirty_ratelimit;
	struct fprop_local_percpu completions;
	int dirty_exceeded;
	enum wb_reason start_all_reason;
	spinlock_t work_lock;	/* protects work_list & dwork scheduling */
	struct list_head work_list;
	struct delayed_work dwork;	/* work item used for writeback */
	struct delayed_work bw_dwork;	/* work item used for bandwidth estimate */
	struct list_head bdi_node;	/* anchored at bdi->wb_list */
#ifdef CONFIG_CGROUP_WRITEBACK
	struct percpu_ref refcnt;	/* used only for !root wb's */
	struct fprop_local_percpu memcg_completions;
	struct cgroup_subsys_state *memcg_css;	/* the associated memcg */
	struct cgroup_subsys_state *blkcg_css;	/* and blkcg */
	struct list_head memcg_node;	/* anchored at memcg->cgwb_list */
	struct list_head blkcg_node;	/* anchored at blkcg->cgwb_list */
	struct list_head b_attached;	/* attached inodes, protected by list_lock */
	struct list_head offline_node;	/* anchored at offline_cgwbs */
	union {
		struct work_struct release_work;
		struct rcu_head rcu;
	};
#endif
};
```

- `b_dirty`: 脏inode列表
- `b_io`: 待写回的inode列表
- `b_more_io`: 需要更多IO的inode列表
- `b_dirty_time`: 仅时间戳脏的inode列表
- `work_list`: 待处理的写回工作列表
- `dwork`: 延迟工作项，绑定到`wb_workfn`
- `work_lock`: 保护`work_list`和`dwork`调度的锁

#### `wb_writeback_work` 写回工作任务

```C++
struct wb_writeback_work {
	long nr_pages;
	struct super_block *sb;
	enum writeback_sync_modes sync_mode;
	unsigned int tagged_writepages:1;
	unsigned int for_kupdate:1;
	unsigned int range_cyclic:1;
	unsigned int for_background:1;
	unsigned int for_sync:1;	/* sync(2) WB_SYNC_ALL writeback */
	unsigned int auto_free:1;	/* free on completion */
	enum wb_reason reason;	/* why was writeback initiated? */
	struct list_head list;	/* pending work list */
	struct wb_completion *done;	/* set if the caller waits */
};
```

- `nr_pages`: 要写回的页面数
- `sync_mode`: 同步模式（`WB_SYNC_ALL`或`WB_SYNC_NONE`）
- `for_background`: 是否为后台写回
- `for_sync`: 是否为同步操作
- `reason`: 写回原因（如`WB_REASON_PERIODIC`、`WB_REASON_VMSCAN`等）
- `list`: 链接到`wb->work_list`

## PageCaches 生命周期

```Bash
┌─────────────────────────────────────────────────────────────────┐
│ 1. 创建阶段（Creation）                                         │
│                                                                 │
│ 文件系统挂载/创建文件/打开文件                                    │
│   └─> alloc_inode()                                             │
│         └─> kmem_cache_alloc(ext4_inode_cachep)                │
│               └─> init_once() [slab构造函数]                    │
│                     └─> inode_init_once()                      │
│                           └─> __address_space_init_once()      │
│                                 └─> xa_init_flags(&i_pages, ...)│
│                                       └─> i_pages初始化（空XArray）│
│                                                                 │
│   └─> inode_init_always()                                       │
│         └─> 初始化address_space其他字段                          │
│               └─> mapping->a_ops = &empty_aops                  │
│               └─> mapping->host = inode                         │
│               └─> nrpages = 0                                   │
└─────────────────────────────────────────────────────────────────┘
│
▼
┌─────────────────────────────────────────────────────────────────┐
│ 2. 使用阶段（Usage）                                            │
│                                                                 │
│ 文件读取：                                                       │
│   └─> filemap_fault() / filemap_read()                          │
│         └─> filemap_get_folio()                                 │
│               └─> 在i_pages中查找page                            │
│                     ├─> 命中：直接返回                           │
│                     └─> 未命中：                                │
│                           └─> __filemap_get_folio(FGP_CREAT)    │
│                                 └─> 分配新page                  │
│                                 └─> a_ops->read_folio()         │
│                                 └─> 从磁盘读取数据               │
│                                 └─> __filemap_add_folio()       │
│                                       └─> xas_store(&i_pages, folio)│
│                                             └─> page加入i_pages  │
│                                             └─> nrpages++       │
│                                                                 │
│ 文件写入：                                                       │
│   └─> generic_perform_write()                                   │
│         └─> write_begin()                                       │
│               └─> grab_cache_page_write_begin()                 │
│                     └─> 查找或创建page                           │
│                     └─> 写入数据到page                           │
│               └─> write_end()                                   │
│               └─> mark_page_dirty()                             │
│                     └─> 标记page为DIRTY                         │
│                                                                 │
│ 页缓存管理：                                                     │
│   ├─> 查找：xa_load(&i_pages, index)                            │
│   ├─> 添加：xas_store(&i_pages, folio)                          │
│   ├─> 删除：xas_store(&i_pages, NULL)                           │
│   └─> 标记：xa_set_mark(&i_pages, index, PAGECACHE_TAG_DIRTY)   │
└─────────────────────────────────────────────────────────────────┘
│
▼
┌─────────────────────────────────────────────────────────────────┐
│ 3. 回收阶段（Reclaim）                                          │
│                                                                 │
│ 内存压力触发：                                                   │
│   └─> kswapd / direct reclaim                                  │
│         └─> shrink_node()                                       │
│               └─> shrink_inode_list()                           │
│                     └─> inode->i_mapping->a_ops->invalidate_folio() │
│                           └─> 从i_pages中移除page                 │
│                                 └─> nrpages--                   │
│                                                                 │
│ 文件删除/截断：                                                  │
│   └─> truncate_inode_pages()                                    │
│         └─> 遍历i_pages，删除所有page                            │
│               └─> nrpages = 0                                   │
└─────────────────────────────────────────────────────────────────┘
│
▼
┌─────────────────────────────────────────────────────────────────┐
│ 4. 销毁阶段（Destruction）                                      │
│                                                                 │
│ inode引用计数降为0：                                             │
│   └─> iput(inode)                                               │
│         └─> iput_final()                                        │
│               └─> evict()                                       │
│                     └─> clear_inode()                           │
│                           └─> truncate_inode_pages_final()      │
│                                 └─> 清空i_pages中的所有page      │
│                                 └─> 等待i_pages锁               │
│                           └─> destroy_inode()                   │
│                                 └─> kmem_cache_free(ext4_inode_cachep) │
│                                       └─> i_pages随inode一起释放 │
│                                                                 │
│ 注意：i_pages本身是inode结构的一部分，                             │
│       不需要单独释放                                            │
└─────────────────────────────────────────────────────────────────┘
```

## `address_space`的创建初始化
