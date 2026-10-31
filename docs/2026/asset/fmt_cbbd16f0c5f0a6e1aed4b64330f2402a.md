
```c
struct malloc_context {
    uint64_t secret; // 和 meta_area 头的 check 是同一个值，就是校验值
#ifndef PAGESIZE
    size_t pagesize;
#endif
    int init_done; // 是否初始化标记
    unsigned mmap_counter; // 记录有多少 mmap 的内存的数量
    struct meta *free_meta_head; // 被 free 的 meta 头，这里 meta 管理使用了队列和双向循环链表
    struct meta *avail_meta; // 指向可用 meta 数组
    size_t avail_meta_count, avail_meta_area_count, meta_alloc_shift;
    struct meta_area *meta_area_head, *meta_area_tail;
    unsigned char *avail_meta_areas;
    struct meta *active[48]; // 记录着可用的 meta
    size_t usage_by_class[48];
    uint8_t unmap_seq[32], bounces[32];
    uint8_t seq;
    uintptr_t brk;
};
```

![image-20210919154755122](upload/attach/202208/907645_6CTKFR52SW7JFP3.jpg)

**小总结一下**

- musl 中堆的管理由 `meta` 管理 `group`，`group` 管理 `chunk`。
- 在 `free` 或者 `malloc chunk` 的时候，又是从 `chunk` 到 `group` 再到 `meta`，从小到大索引。
- `meta` 间通过 `meta` 中 `prev`、`next` 结构形成循环链表连接。

![image-20210920011323565](upload/attach/202208/907645_H5U456FZCBMYMRZ.jpg)

### 0x01 释放与分配

如果不想看源码，可以跳下面看总结。

#### `malloc`

源码路径：

> `/src/malloc/mallocng/malloc.c`

**源码：**

```c
void *malloc(size_t n)
{
    if (size_overflows(n)) return 0; // 最大申请空间限制
    struct meta *g;
    uint32_t mask, first;
    int sc;
    int idx;
    int ctr;

    if (n >= MMAP_THRESHOLD) { // size >= 阈值，会直接通过 mmap 申请空间
        size_t needed = n + IB + UNIT; // UNIT 0x10，IB 4，定义在 meta.h 里，这里 UNIT + IB 是一个基本头的大小
        void *p = mmap(0, needed, PROT_READ | PROT_WRITE, MAP_PRIVATE | MAP_ANON, -1, 0); // 新 mmap group 空间
        if (p == MAP_FAILED) return 0;

        wrlock();
        step_seq();
        g = alloc_meta();
        if (!g) { // 如果申请 meta 失败，会把刚刚 mmap 出来的 group 回收
            unlock();
            munmap(p, needed); // 回收 group
            return 0;
        }

        g->mem = p; // mem = group 地址
        g->mem->meta = g; // group 头部指向 meta（g 为 meta）
        g->last_idx = 0; // mmap 的 group last_idx 默认值 = 0
        g->freeable = 1;
        g->sizeclass = 63; // mmap 申请的 sizeclass 都为 63
        g->maplen = (needed + 4095) / 4096;
        g->avail_mask = g->freed_mask = 0;
        ctx.mmap_counter++; // mmap 内存记载数量++
        idx = 0;
        goto success;
    }

    // 否则直接根据传入 size，转换成 size_classes 的对应大小的下标
    sc = size_to_class(n);
    rdlock();
    g = ctx.active[sc]; // 从现有的 active 中取出对应 sc 的 meta，不同 sc 对应不同的 meta

    /*
     * 如果从 ctx.active 中没找到对应的 meta，会执行下面的 if 分支。
     * 这里 !g <=> g == 0，说明 ctx.active[sc] 没有对应的 meta。
     */
    if (!g && sc >= 4 && sc < 32 && sc != 6 && !(sc & 1) && !ctx.usage_by_class[sc]) {
        size_t usage = ctx.usage_by_class[sc | 1]; // 如果在 ctx.active 没找到，就使用更大 size group 的 meta

        // if a new group may be allocated, count it toward
        // usage in deciding if we can use coarse class.
        if (!ctx.active[sc | 1] || (!ctx.active[sc | 1]->avail_mask
            && !ctx.active[sc | 1]->freed_mask))
            usage += 3;

        if (usage <= 12)
            sc |= 1;
        g = ctx.active[sc];
    }

    for (;;) {
        mask = g ? g->avail_mask : 0;
        first = mask & -mask;
        if (!first) break;

        if (RDLOCK_IS_EXCLUSIVE || !MT)
            g->avail_mask = mask - first;
        else if (a_cas(&g->avail_mask, mask, mask - first) != mask)
            continue;

        idx = a_ctz_32(first);
        goto success;
    }

    upgradelock();
    idx = alloc_slot(sc, n);

    /*
     * 如果当前 group 不满足，就会来到这里：
     * alloc_slot 从 group 中取出对应大小 chunk 的 idx。
     *
     * 这里先从对应 sc 的 ctx.active[sc] 中找对应的 meta 的 group
     * 是否有空闲 chunk 可以使用，再从队列中其他 meta 的 group 中找。
     * 如果队列中其他 meta 的 group 有可利用的 chunk，就使用；
     * 如果没有，就重新分配一个新的 group。
     */
    if (idx < 0) {
        unlock();
        return 0;
    }

    g = ctx.active[sc]; // 取出 sc 对应 active meta

success:
    ctr = ctx.mmap_counter;
    unlock();
    return enframe(g, idx, n, ctr); // 从对应 meta 中的 group 取出第 idx 号 chunk，n = size
}
```

**关键：**一般分配先进入这个循环：

```c
for (;;) {
    mask = g ? g->avail_mask : 0; // 先检查 g 所指 meta 是否存在，若存在 mask = g->avail_mask

    first = mask & -mask; // 这里只有 mask = 0 时，first 才会为 0

    if (!first) break; // mask 为 0，first = 0，无可用空闲 chunk，跳出循环

    if (RDLOCK_IS_EXCLUSIVE || !MT) // 如果是排它锁，那么下面保证成功
        g->avail_mask = mask - first;
    else if (a_cas(&g->avail_mask, mask, mask - first) != mask)
        // 成功找到并设置 avail_mask 之后，continue 后设置 idx，然后跳出
        continue;

    idx = a_ctz_32(first);
    goto success;
}
```

```c
upgradelock();
```

如果：

```c
idx = alloc_slot(sc, n);
```

#### `alloc_slot`

```c
static int alloc_slot(int sc, size_t req)
{
    // 尝试从 active 中找到合适可用的
    uint32_t first = try_avail(&ctx.active[sc]);
    if (first) return a_ctz_32(first);

    // 如果没找到，重新创建一个 meta，然后重新分配一个 size 大小对应 sc 的 group，
    // 给这个新分配的 meta
    struct meta *g = alloc_group(sc, req);
    if (!g) return -1;

    g->avail_mask--;
    queue(&ctx.active[sc], g); // 把新 meta 加入队列
    return 0;
}
```

#### `try_avail`

```c
static uint32_t try_avail(struct meta **pm)
{
    struct meta *m = *pm;
    uint32_t first;

    if (!m) return 0;

    uint32_t mask = m->avail_mask;
    if (!mask) { // mask = m->avail_mask，!mask 表示没有可用的 chunk
        if (!m->freed_mask) { // if (!m->freed_mask) <=> 没有已经释放的 chunk
            /*
             * 进入这个分支的条件：
             * 既没有可用的 chunk，也没有被释放还未回收的 chunk，
             * 即 chunk 都被使用，且都没被释放。
             */
            dequeue(pm, m); // freed_mask == avail_mask == 0，group 空间已满，让对应的 meta 出队
            m = *pm;
            if (!m) return 0;
        } else {
            /*
             * 这里 else 表示的是：无可用空闲 chunk，但是有已经释放的 chunk。
             * free 释放的 chunk 不能马上被复用。
             */
            /*
             * 进入这个分支的条件：
             * 没有可用的 chunk，有被释放还未回收的 chunk。
             *
             * 有点好奇这里，如果达成这个条件，然后利用指针互写，
             * 修改 m->next 伪造的 meta，是不是就可以制造 fake meta 入队的假象。
             * 若 meta 链表中没有，一般 meta 的 next 和 prev 都是指向自己。
             */
            m = m->next;
            *pm = m;
        }

        mask = m->freed_mask;

        // 如果这个 meta 的 group 只含有一个 chunk，且被释放就跳过；
        // 或者这个 meta 的 group 根本不能被释放，如 mmap 的 group last_idx = 0，freeable = 1
        if (mask == (2u << m->last_idx) - 1 && m->freeable) {
            m = m->next;
            *pm = m;
            mask = m->freed_mask;
        }

        // activate more slots in a not-fully-active group
        // if needed, but only as a last resort. prefer using
        // any other group with free slots. this avoids
        // touching & dirtying as-yet-unused pages.
        if (!(mask & ((2u << m->mem->active_idx) - 1))) {
            if (m->next != m) { // 如果这个 meta 后还有 meta，就切换到下一个 meta
                m = m->next;
                *pm = m;
            } else {
                int cnt = m->mem->active_idx + 2;
                int size = size_classes[m->sizeclass] * UNIT;
                int span = UNIT + size * cnt;

                // activate up to next 4k boundary
                while ((span ^ (span + size - 1)) < 4096) { // 页对齐
                    cnt++;
                    span += size;
                }

                if (cnt > m->last_idx + 1)
                    cnt = m->last_idx + 1;

                m->mem->active_idx = cnt - 1;
            }
        }

        mask = activate_group(m); // 这里是给 m 的 avail_mask 打上标记
        assert(mask);
        decay_bounces(m->sizeclass);
    }

    first = mask & -mask;
    // 若 mask % 2 == 0，则 first 是结果中能整除这个偶数的最大的 2 的幂；
    // 若 mask % 2 == 1，则 first 永远为 1
    m->avail_mask = mask - first;
    return first;
}
```

#### `malloc` 的流程

**一、判断是否超过 size 阈值**

- 先检查申请的 `chunk` 的 `needed size` 是否超过最大申请限制。
- 检查申请的 `needed` 是否超过需要 `mmap` 分配的阈值，超过就用 `mmap` 分配一个 `group` 来给 `chunk` 使用。
- 若是 `mmap`，则设置各种标记。

**二、分配 chunk**

1. 若申请的 `chunk` 没超过阈值，就从 `active` 队列找管理对应 `size` 大小的 `meta`。

2. 关于找对应 `size` 的 `meta`，这里有两种情况：

   - 如果 `active` 对应 `size` 的 `meta` 位置上为空，没找到，那么尝试先找 `size` 更大的 `meta`。
   - 如果 `active` 对应 `size` 的 `meta` 位置上有对应的 `meta`，尝试从这个 `meta` 中的 `group` 找到可用的 `chunk`。这里是 `malloc` 中的那个循环：`for (;;)`。这里不清楚的建议看 `malloc` 源码分析。
   - 如果通过循环里的 `meta->avail_mask` 判断当前 `group` 中是否有空闲 `chunk`：
     - 有，就直接修改 `meta->avail_mask`，然后利用 `enframe(g, idx, n, ctr)` 从对应 `meta` 中的 `group` 取出第 `idx` 号 `chunk` 分配。
     - 无，`break` 跳出循环。
   - 跳出循环后执行 `idx = alloc_slot(sc, n);`，`alloc_slot` 有三种分配方式：
     - **使用 group 中被 free 的 chunk**
     - **从队列中其他 meta 的 group 中找**
     - **如果都不行，就重新分配一个新的 group，对应一个新的 meta**

3. `enframe(g, idx, n, ctr)` 取出对应 `meta` 中对应 `idx` 的 `chunk`。

> 仔细观察分配的过程，我们也可以看出为什么 `free` 的 `chunk` 不能立即回收使用，因为有空闲的 `chunk` 时，分配 `chunk` 是直接设置 `meta->avail_mask`。

> 然后直接 `enframe()` 从 `group` 中取出 `chunk` 即可，不会设置 `meta->freed`。

#### `free`

源码路径：

> `/src/malloc/mallocng/malloc.c`
