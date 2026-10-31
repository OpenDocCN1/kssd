
```c
for (;;) {
  mask = g ? g->avail_mask : 0; // 先检查g所指meta是否存在，若存在mask = g->avail_mask
  first = mask & -mask; // 这里只有mask=0时，first才会为0
  if (!first) break; // mask为0，first=0，无可用空闲chunk，跳出循环
  if (RDLOCK_IS_EXCLUSIVE || !MT) // 如果是排它锁, 那么下面保证成功
    g->avail_mask = mask - first;
  else if (a_cas(&g->avail_mask, mask, mask - first) != mask) // 成功找到并设置avail_mask之后,continue 后设置idx，然后跳出
    continue;
  idx = a_ctz_32(first);
  goto success;
}
upgradelock();
如果 idx = alloc_slot(sc, n);
```

## alloc_slot

```c
static int alloc_slot(int sc, size_t req)
{
  // 尝试从限制active 中找到合适可用的
  uint32_t first = try_avail(&ctx.active[sc]);
  if (first) return a_ctz_32(first);

  // 如果没找到 重新创造一个meta，然后重新分配一个size大小对应sc的group，给这个新分配的meta
  struct meta *g = alloc_group(sc, req);
  if (!g) return -1;

  g->avail_mask--;
  queue(&ctx.active[sc], g);
  // 把新meta 加入队列
  return 0;
}
```

## try_avail

```c
static uint32_t try_avail(struct meta **pm)
{
  struct meta *m = *pm;
  uint32_t first;
  if (!m) return 0;
  uint32_t mask = m->avail_mask;
  if (!mask) // mask = m->avail_mask (!mask) 表示没有可用的chunk了
  {
    if (!m->freed_mask) // if (!m->freed_mask) <= 没有已经释放的chunk
    {
      /* 进入这个分支的条件:既没有可用的chunk，也没有被释放还未回收的chunk，即chunk都被使用，且都没被释放 */
      dequeue(pm, m); // freed_mask==avail_mask==0, group 空间已满 让对应的meta 出队
      m = *pm;
      if (!m) return 0;
    }
    /* 这里else表示的是:无可用空闲chunk，但是有已经释放的chunk
    !!! free释放的chunk 不能马上被复用的 !!!
    */
    else
    {
      /* 进入这个分支的条件:没有可用的chunk，有被释放还未回收的chunk。
      有点好奇这里，如果达成这个条件，然后利用指针互写，修改m->next 伪造的meta，是不是就可以制造fake meta 入队的假象
      若meta链表中没有，一般meta 的next和prev 都是指向自己
      */
      m = m->next;
      *pm = m;
    }

    mask = m->freed_mask;
    // 如果这个meta 的group 只含有一个chunk ，且被释放就跳过，
    // 或者 这个meta 的group 根本不能被释放 如mmap 的 group last_idx = 0 freeable=1
    if (mask == (2u << m->last_idx) - 1 && m->freeable)
    {
      m = m->next;
      *pm = m;
      mask = m->freed_mask;
    }

    // activate more slots in a not-fully-active group
    // if needed, but only as a last resort. prefer using
    // any other group with free slots. this avoids
    // touching & dirtying as-yet-unused pages.
    if (!(mask & ((2u << m->mem->active_idx) - 1)))
    {
      if (m->next != m)
      {
        // 如果这个meta 后还有meta 就切换到 下一个meta
        m = m->next;
        *pm = m;
      }
      else
      {
        int cnt = m->mem->active_idx + 2;
        int size = size_classes[m->sizeclass] * UNIT;
        int span = UNIT + size * cnt;
        // activate up to next 4k boundary
        while ((span ^ (span + size - 1)) < 4096) // 页对齐
        {
          cnt++;
          span += size;
        }
        if (cnt > m->last_idx + 1) cnt = m->last_idx + 1;
        m->mem->active_idx = cnt - 1;
      }
    }
    mask = activate_group(m); // 这里是给 m的 avail_mask 打上标记
    assert(mask);
    decay_bounces(m->sizeclass);
  }
  first = mask & -mask; // 若 mask%2==0 则first=结果是能整除这个偶数的最大的2的幂 若 mask%2==1 则first永远为1
  m->avail_mask = mask - first;
  return first;
}
```

## malloc 的流程

**一、判断是否超过size 阈值**
- 先检查申请的chunk的 `needed` `size` 是否超过最大申请限制
- 检查申请的`needed` 是否超过需要`mmap` 的分配的阈值 超过就用`mmap` 分配一个`group` 来给`chunk`使用
- 若是`mmap` 则设置各种标记

**二、分配chunk**
1. 若申请的chunk 没超过阈值 就从`active` 队列找管理对应`size`大小的`meta`
2. 关于找对应`size`的`meta` 这里有两种情况:
   - 如果`active` 对应`size`的`meta` 位置上为空，没找到那么尝试先找`size`更大的`meta`
   - 如果`active` 对应`size`的`meta`位置上有对应的`meta`，尝试从这个`meta`中的`group`找到可用的`chunk`(这里`malloc` 那个循环:`for (;;)`)
     - 如果通过循环里，通过`meta->avail_mask` 判断当前`group` 中是否有空闲`chunk`
       - 有，就直接修改`meta->avail_mask`，然后利用`enframe(g, idx, n, ctr)` 从对应`meta` 中的`group` 取出 第`idx`号`chunk`分配
       - 无，`break` 跳出循环
     - 跳出循环后执行`idx = alloc_slot(sc, n)` `alloc_slot`有三种分配方式
       - **使用`group`中被`free`的`chunk`**
       - **从队列中其他`meta`的`group` 中找**
       - 如果都不行就重新分配一个新的`group` 对应一个新的`meta`
3. `enframe(g, idx, n, ctr)` 取出 对应`meta` 中对应`idx` 的`chunk`

> 仔细观察分配的过程，我们也可以看出为什么`free` 的`chunk`不能立即回收使用，因为有空闲的`chunk`的时候，分配`chunk`是直接设置`meta->avail_mask`
>
> 然后直接`enframe()` 直接从`group`中取出 `chunk`即可，不会设置`meta->freed`

## free

源码路径
> `/src/malloc/mallocng/malloc.c`

```c
void free(void *p)
{
  if (!p) return;

  struct meta *g = get_meta(p); // 通过chunk p 用get_meta得到对应的meta
  int idx = get_slot_index(p); // 得到对应chunk的 idx
  size_t stride = get_stride(g); // 得到sizeclasses 中对应chunk类型的size

  unsigned char *start = g->mem->storage + stride * idx;
  unsigned char *end = start + stride - IB;
  /* start = g->mem->storage(得到group中第一个chunk地址) + stride*idx(加上对应chunk偏移);
  start 就为对应p(chunk)的起始地址
  end 对应结束地址
  */

  get_nominal_size(p, end); // 算出真实大小
  uint32_t self = 1u << idx, all = (2u << g->last_idx) - 1; // 设置bitmap 标志
  ((unsigned char *)p)[-3] = 255;
  *(uint16_t *)((char *)p - 2) = 0;
  if (((uintptr_t)(start - 1) ^ (uintptr_t)end) >= 2 * PGSZ && g->last_idx) {
    unsigned char *base = start + (-(uintptr_t)start & (PGSZ - 1));
    size_t len = (end - base) & -PGSZ;
    if (len) madvise(base, len, MADV_FREE);
  }

  // atomic free without locking if this is neither first or last slot
  for (;;) {
    uint32_t freed = g->freed_mask;
    uint32_t avail = g->avail_mask;
    uint32_t mask = freed | avail; // 将释放的chunk 和 现在可用的 chunk 加起来
    assert(!(mask & self));
    if (!freed || mask + self == all) break;
    // !freed 没有被释放的chunk，mask+self==all说明释放了当前chunk所有chunk 都将被回收
    // 此group 会被弹出队列
    if (!MT)
      g->freed_mask = freed + self; // 设置free_mask 表示chunk 被释放
    else if (a_cas(&g->freed_mask, freed, freed + self) != freed)
      continue;
    return;
  }

  wrlock();
  struct mapinfo mi = nontrivial_free(g, idx); // 含有meta 操作 ，内有unlink 是漏洞利用的关键
  unlock();
  if (mi.len) munmap(mi.base, mi.len);
}
```

## free 的流程

**free**：
- 通过`get_meta(p)`得到`meta` (`get_meta` 是通过`chunk` 对应的`offset` 索引到对应的`group` 再索引到`meta`) 下面会详细介绍`get_meta`
- 通过`get_slot_index(p)`得到对应`chunk`的 `idx`
- `->` 通过`get_nominal_size(p, end)` 算出真实大小
- 重置`idx` 和 `offset` `idx` 被置为`0xff` 标记`chunk`
- 修改`freed_mask` 标记`chunk`被释放
- 最后调用`nontrivial_free` 完成关于`meta`一些剩余操作 (注意进入`nontrivial_free` 是在`for`循环外 还未设置)

**细节**：
- `释放chunk的时候，先只会修改freed_mask,不会修改avail_mask，说明chunk 在释放后，不会立即被复用`
- 注意进入`nontrivial_free` 是在`for`循环外 还未设置`freed_mask` 跳出循环的条件是 `if (!freed || mask + self == all) break;`
- `free` 中`chunk` 的起始位置可以通过 `chunk`的`idx` 定位

## get_meta
