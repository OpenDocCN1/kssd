

# [原创] PageCache 详细分析（读写/写回）——基于 Linux 6.12.32 版本

> 参考文章：
>
> 总览（概念上讲解）：<mark class="encrypted">1e5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6\*7K9s2g2S2L8X3I4S2L8W2)9J5k6i4A6Z5K9h3S2#2i4K6u0W2j5$3!0E0i4K6u0r3M7q4)9J5c8U0b7K6y4U0x3I4x3K6V1H3z5l9`.`.</mark>
>
> PageCache 的产生、释放及优化：<mark class="encrypted">051K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3M7$3!0X3N6r3y4D9L8%4g2V1i4K6u0r3M7q4)9J5c8U0p5$3y4U0b7&6x3K6t1$3i4K6u0W2K9s2c8E0L8l9`.`.</mark>
>
> `address_space` 结构解析：<mark class="encrypted">933K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6\*7K9s2g2S2L8X3I4S2L8W2)9J5k6i4A6Z5K9h3S2#2i4K6u0W2j5$3!0E0i4K6u0r3M7q4)9J5c8U0f1@1x3o6R3H3y4U0R3%4y4R3`.`.</mark>
>
> `xarray` 详细解释（很厉害）：<mark class="encrypted">345K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6\*7K9s2g2S2L8X3I4S2L8W2)9J5k6i4A6Z5K9h3S2#2i4K6u0W2j5$3!0E0i4K6u0r3M7q4)9J5c8U0f1^5y4K6p5^5y4o6j5J5x3H3`.`.</mark>

本文较长，过程较为详细，请耐心阅读。本人水平有限，如果出现什么错误，请各位大佬指出。

## 版本说明

> 本文分析的版本均为：Linux 6.12.32。

## 核心数据结构

### FGP Flags

```C++
#define FGP\_ACCESSED ((\_\_force fgf\_t)0x00000001)
#define FGP\_LOCK ((\_\_force fgf\_t)0x00000002)
#define FGP\_CREAT ((\_\_force fgf\_t)0x00000004)
#define FGP\_WRITE ((\_\_force fgf\_t)0x00000008)
#define FGP\_NOFS ((\_\_force fgf\_t)0x00000010)
#define FGP\_NOWAIT ((\_\_force fgf\_t)0x00000020)
#define FGP\_FOR\_MMAP ((\_\_force fgf\_t)0x00000040)
#define FGP\_STABLE ((\_\_force fgf\_t)0x00000080)
#define FGF\_GET\_ORDER(fgf) (((\_\_force unsigned)fgf) >> 26) /\* top 6 bits \*/
#define FGP\_WRITEBEGIN (FGP\_LOCK | FGP\_WRITE | FGP\_CREAT | FGP\_STABLE)
```

- `FGP_ACCESSED`：标记页面为已访问（用于 LRU）。
- `FGP_LOCK`：返回时页面已加锁。
- `FGP_CREAT`：未命中时创建新页面。
- `FGP_WRITE`：用于写入操作。
- `FGP_NOFS`：清除 `__GFP_FS`（避免递归文件系统操作）。
- `FGP_NOWAIT`：非阻塞模式，加锁失败返回 `-EAGAIN`。
- `FGP_FOR_MMAP`：`mmap` 场景，创建后解锁。
- `FGP_STABLE`：等待页面稳定（写回完成）。

### `address_space`

```C++
struct address\_space {
struct inode \*host; // 指向所属的inode
struct xarray i\_pages; // XArray：存储页缓存的radix tree
struct rw\_semaphore invalidate\_lock; // 保护页面失效操作的读写锁
gfp\_t gfp\_mask; // 页面分配的内存标志
atomic\_t i\_mmap\_writable; // 可写内存映射计数
struct rb\_root\_cached i\_mmap; // 内存映射的RB树（用于跟踪VMA）
unsigned long nrpages; // 页缓存中的页面总数
pgoff\_t writeback\_index; // 写回操作的起始索引
const struct address\_space\_operations \*a\_ops; // 地址空间操作函数集
unsigned long flags; // 标志位（AS\_\*）
errseq\_t wb\_err; // 写回错误序列号
spinlock\_t i\_private\_lock; // 私有数据保护锁
struct list\_head i\_private\_list; // 私有数据链表
struct rw\_semaphore i\_mmap\_rwsem; // 保护i\_mmap的读写锁
void \* i\_private\_data; // 文件系统私有数据指针
}
```

- `host`：指向所属 `inode`，建立双向关联。
- `i_pages`：XArray，存储页缓存页面的 radix tree。
  - 键：页面索引（`pgoff_t`）。
  - 值：`folio/page` 指针。
  - 标志：`XA_FLAGS_LOCK_IRQ`（IRQ 安全）、`XA_FLAGS_ACCOUNT`（内存统计）。
- `invalidate_lock`：保护页面失效操作，防止与 I/O 并发。
- `i_mmap`：RB 树，记录该文件的所有内存映射（VMA）。
- `a_ops`：地址空间操作函数集，文件系统可自定义。
- `nrpages`：页缓存页面总数，用于统计和限制。
- `writeback_index`：写回起始索引，支持断点续写。

### XArray

> `i_pages` 的数据结构，用于管理 Page Cache。
>
> XArray 是基于 Radix Tree（基数树）的实现。在 Linux 内核中，XArray 相比于树，使用体验更类似于一个无限增长的数组。

#### XArray 根结构

```C++
struct xarray {
spinlock\_t xa\_lock;
/\* private: The rest of the data structure is not to be used directly. \*/
gfp\_t xa\_flags;
void \_\_rcu \* xa\_head;
};
```

- `xa_lock`：保护 XArray 的锁（IRQ 安全）。
- `xa_flags`：标志（如 `XA_FLAGS_LOCK_IRQ`、`XA_FLAGS_ACCOUNT`）。
- `xa_head`：指向根节点或直接条目（RCU 指针）。

#### 常量

```C
#ifndef XA\_CHUNK\_SHIFT
#define XA\_CHUNK\_SHIFT (IS\_ENABLED(CONFIG\_BASE\_SMALL) ? 4 : 6)
#endif
#define XA\_CHUNK\_SIZE (1UL << XA\_CHUNK\_SHIFT)
#define XA\_CHUNK\_MASK (XA\_CHUNK\_SIZE - 1)
```

- `XA_CHUNK_SHIFT`：通常为 6（64 位系统）。
- `XA_CHUNK_SIZE`：64（表示一个 `xa_node` 节点可以表示的最大槽位数量）。
- `XA_CHUNK_MASK`：63（`0x3F`，用于取模）。

#### 数据结构

```C
XArray树结构（3级示例）：
xa\_head
│
▼
[xa\_node (shift=12)]
│
┌─────────────┼─────────────┐
│ │ │
slots[0] slots[1] slots[2]
│ │ │
▼ ▼ ▼
[xa\_node] [xa\_node] NULL
(shift=6) (shift=6)
│ │
┌───┼───┐ ┌───┼───┐
│ │ │ │ │ │
slots[0] slots[1] slots[2] slots[3]
│ │ │ │ │ │
▼ ▼ ▼ ▼ ▼ ▼
folio folio folio folio folio
(0) (1) (2) (3) (4)
索引计算：
- index = 0: 0 >> 12 = 0, 0 >> 6 = 0, 0 & 63 = 0 → slots[0][0]
- index = 1: 1 >> 12 = 0, 1 >> 6 = 0, 1 & 63 = 1 → slots[0][1]
- index = 64: 64 >> 12 = 0, 64 >> 6 = 1, 64 & 63 = 0 → slots[1][0]
```

#### Node 结构

```C++
struct xa\_node {
unsigned char shift; /\* Bits remaining in each slot \*/
unsigned char offset; /\* Slot offset in parent \*/
unsigned char count; /\* Total entry count \*/
unsigned char nr\_values; /\* Value entry count \*/
struct xa\_node \_\_rcu \*parent; /\* NULL at top of tree \*/
struct xarray \*array; /\* The array we belong to \*/
union {
struct list\_head private\_list; /\* For tree user \*/
struct rcu\_head rcu\_head; /\* Used when freeing node \*/
};
void \_\_rcu \*slots[XA\_CHUNK\_SIZE];
union {
unsigned long tags[XA\_MAX\_MARKS][XA\_MARK\_LONGS];
unsigned long marks[XA\_MAX\_MARKS][XA\_MARK\_LONGS];
};
};
```

- `shift`：该层级剩余的位数（决定该节点覆盖的索引范围）。
  - `shift = 0`：叶子节点，直接存储条目。
  - `shift = 6`：覆盖 64 个索引（`2^6 = 64`）。
  - `shift = 12`：覆盖 4096 个索引（`2^12 = 4096`）。
  - `shift = 18`：覆盖 262144 个索引（`2^18 = 262144`）。
- `offset`：在父节点中的槽位偏移。
- `count`：非 NULL 槽位总数。
- `nr_values`：值条目数量。
- `parent`：父节点指针（根节点为 NULL）。
- `array`：所属的 XArray。
- `slots[]`：64 个槽位数组（每个槽位可指向子节点或条目），这里较为复杂，后文详细解释。
- `marks[]`：标记位图（用于 DIRTY、WRITEBACK 等）。

### Entry

> Node 中 `slots` 存放的 Entry，有三种基本类型：
>
> - Pointer Entry
> - Internal Entry
> - Value Entry

```C
Entry (void \*)
│
├─ [00] Pointer Entry (普通指针条目)
│ ├─ NULL Entry (空指针)
│ └─ 普通指针 (指向实际数据对象)
│
├─ [10] Internal Entry (内部条目)
│ ├─ Node Pointer (>4096)
│ ├─ Sibling Entry (0-62)
│ ├─ Retry Entry (256)
│ ├─ Zero Entry (257)
│ └─ Error Entry (-4094 ~ -2)
│
└─ [x1] Value Entry (值条目)
├─ 纯值条目 (低1位=1, 低2位=01)
└─ Tagged Pointer (低2位=11)
```

#### Pointer Entry（低两位为 00）

##### Null Entry

```C
// NULL指针本身就是NULL entry
void \*entry = NULL;
```

##### 普通指针

> 指向实际数据或子节点。

```C
// 示例：指向struct folio的指针
void \*entry = folio; // 低2位 = 00
```

#### Internal Entry（低两位为 10）

##### 转化函数

```C
static inline void \*xa\_mk\_node(const struct xa\_node \*node)
{
return (void \*)((unsigned long)node | 2);
}
/\* Private \*/
static inline struct xa\_node \*xa\_to\_node(const void \*entry)
{
return (struct xa\_node \*)((unsigned long)entry - 2);
}
```

##### Node Pointer

> 指向下一层 `xa_node`。

- 编码：`xa_mk_internal(v)`，其中 `v > 1024`（实际检查 `> 4096`）。

```C++
static inline bool xa\_is\_node(const void \*entry)
{
return xa\_is\_internal(entry) && (unsigned long)entry > 4096;
}
```

##### Sibling Entry（兄弟条目）

- 大页面（multi-index entry）时，多个索引共享同一 `folio`。
- 编码：`xa_mk_internal(offset)`，`offset` 范围为 0–62。

```C
假设一个64KB的大页面（order=4），占用16个4KB页面的索引：
索引100-115都指向同一个folio
存储方式:
i\_pages[100] = folio指针 ← 规范槽位（canonical slot）
i\_pages[101] = sibling(100) ← 指向槽位100
i\_pages[102] = sibling(100)
...
i\_pages[115] = sibling(100)
查找索引103时:
xas\_descend() → 发现sibling(100)
→ 跳转到槽位100 → 返回folio指针
// 创建函数
static inline void \*xa\_mk\_sibling(unsigned int offset)
{
return xa\_mk\_internal(offset);
}
// 识别函数
static inline bool xa\_is\_sibling(const void \*entry)
{
return IS\_ENABLED(CONFIG\_XARRAY\_MULTI) && xa\_is\_internal(entry) &&
(entry < xa\_mk\_sibling(XA\_CHUNK\_SIZE - 1));
}
// 提取函数
static inline unsigned long xa\_to\_sibling(const void \*entry)
{
return xa\_to\_internal(entry);
}
```

##### Retry Entry（重试条目）

> 标记节点正在被修改或即将释放，提示 RCU 读者重试。

- 编码：`XA_RETRY_ENTRY = xa_mk_internal(256)`。
- 节点删除时替换 slot。
- 多索引遍历时遇到节点指针。

```C++
#define XA\_RETRY\_ENTRY xa\_mk\_internal(256)
/\*\*
\* xa\_is\_retry() - Is the entry a retry entry?
\* @entry: Entry retrieved from the XArray
\*
\* Return: %true if the entry is a retry entry.
\*/
static inline bool xa\_is\_retry(const void \*entry)
{
return unlikely(entry == XA\_RETRY\_ENTRY);
}
```

##### Zero Entry（零条目）

- 功能：预留索引，表示“已分配但值为 NULL”。
- 编码：`XA_ZERO_ENTRY = xa_mk_internal(257)`。

