
```c
static inline struct meta *get_meta(const unsigned char *p) {
  assert(!((uintptr_t)p & 15));
  int offset = *(const uint16_t *)(p - 2); // 得到chunk offset
  int index = p[-3] & 31;; // 得到chunk idx
  if (p[-4]) {
    assert(!offset);
    offset = *(uint32_t *)(p - 8);
    assert(offset > 0xffff);
  }
  const struct group *base = (const void *)(p - UNIT*offset - UNIT); // 通过offset 和chunk 地址计算出group地址
  const struct meta *meta = base->meta; // 从group 得到 meta 地址
  assert(meta->mem == base); // 检查meta 是否指向对应的group
  assert(index <= meta->last_idx); // 检查chunk idx 是否超过 meta 最大chunk 容量
  assert(!(meta->avail_mask & (1u<<index)));
  assert(!(meta->freed_mask & (1u<<index)));
  const struct meta_area *area = (void *)((uintptr_t)meta & -4096); // 得到meta_area 地址
  assert(area->check == ctx.secret); // 检查 check 校验值
  if (meta->sizeclass < 48) {
    // 如果属于 sizeclasses 管理的chunk 大小
    assert(offset >= size_classes[meta->sizeclass]*index);
    assert(offset < size_classes[meta->sizeclass]*(index+1));
  } else {
    assert(meta->sizeclass == 63);
  }
  if (meta->maplen) {
    assert(offset <= meta->maplen*4096UL/UNIT - 1);
  }
  return (struct meta *)meta;
}
```

#### nontrivial_free

关于`nontrivial_free()`函数很重要，这里尽量详细说明

```c
static struct mapinfo nontrivial_free(struct meta *g, int i) // i = idx
{
  uint32_t self = 1u<<i;
  int sc = g->sizeclass;
  uint32_t mask = g->freed_mask | g->avail_mask; // mask=已经被free的chunk +可使用的chunk
  if (mask+self == (2u<<g->last_idx)-1 && okay_to_free(g))
  {
    /*
      如果 mask+self == (2u<<g->last_idx)-1 代表此meta中group里的chunk 都被释放 或者 都被用了
      (2u<<g->last_idx)-1 计算出的值化成二进制，其中每位含义类似于bitmap，如果每位为1 表每位要不是被free 不然就是被
      okay_to_free 检测是否可以被释放
    */
    if (g->next)
    {
      // 如果队列中 有下一个meta
      assert(sc < 48); // 检测 sc 是不是mmap 分配的
      // 检测当前meta g 和 队列里的active[sc] meta 是否一样，一样则activate_new赋值为1
      int activate_new = (ctx.active[sc]==g);
      dequeue(&ctx.active[sc], g); // 当前meta 出队
      // 在出队操作后 ,ctx.active[sc]==meta->next 是指的刚刚出队meta 的下一个meta
      if (activate_new && ctx.active[sc])
        activate_group(ctx.active[sc]); // 如果有下一个meta 直接激活 然后修改avail_mask 标志位
    }
    return free_group(g);
  }
  else if (!mask)
  {
    // mask==0 group chunk 空间已被完全使用
    assert(sc < 48);
    // might still be active if there were no allocations after last available slot was taken.
    if (ctx.active[sc] != g) { // 如果 g 未被加入 队列ctx.ative[sc]
      queue(&ctx.active[sc], g); // 把g 加入队列
    }
  }
  a_or(&g->freed_mask, self); // 修改对应的freed_mask 标志 ，表示着对应的chunk 已被释放
  return (struct mapinfo){0};
}
```

```c
static inline void dequeue(struct meta **phead, struct meta *m)
{
  if (m->next != m) {
    m->prev->next = m->next; // 这里存在指针互写 在 prev 所指地址上 写入next指针
    m->next->prev = m->prev; // 在next所指地址上 写入prev 指针
    if (*phead == m) *phead = m->next; // 队列头如果为m 那就更新为m->next
  } else {
    *phead = 0;
  }
  m->prev = m->next = 0; // 清理m(meta)的头尾指针
}
```

**dequeue** 触发条件

> `self = 1 << idx`

下面是几种简单的触发情况

1. `avail_mask` 表示只有一个chunk 被使用 ,`freed_mask`=0，而`free` 刚好要`free` 一个`chunk`
满足 `okay_to_free()` 条件 就可以进入`dequeue` 进行出队操作
> 如`add(1,0x20)` 再`free(1)` 就会使得`meta` 被回收

2. `avail_mask`=0, `freed_mask` 表示只有 1个 chunk 没被 释放，这时释放的`chunk` 就应该是那最后一个`chunk`
如下面情况 `avail_mask ==0` `free_mask=63=00111111` `last_idx = 6`
已经释放6 个`chunk` 还有最后一个`chunk`没被释放 在释放最后一个`chunk` 时会触发`dequeue`使得对应`meta`出队

3. 如果发现这个`group`中所有的`chunk`要么被`free`, 要么是可用的, 那么就会回收掉这个`group`，调用`dequeue`从队列中出队

### 0x02 CTF 中musl题利用

一般有如下几种利用方法，核心原理都是构造假的`chunk` 索引到假的`group` 从而所引导假的`meta`
或覆盖`group` 中指向`meta` 的指针 覆盖为假的`meta` ，然后使得假的`meta` `dequeue` 最终实现`unlink`
(构造`fake_meta` 需要先泄露 `secret` 校验值)

#### dequeue 的两种流程

**一、**
通过构造假的`meta` 满足各种条件 通过以下流程
`free()->nontrivial_free()->dequeue`

这里通过`free` 到 `dequeue`

**二、**
通过`realloc` 里也带有`free`
`realloc()->free(old)->nontrivial_free()->dequeue`

#### 伪造meta后控制程序流的方法

注意: `musl` 是没有`malloc_hook`和 `free_hook` 这种一般的`hook` 位
且`musl` 程序的`IO_FILE` 结构体格式和`libc` 不一样 没有`IO_jump_t`的`vtable`
但是存在**`read`，`write`，`seek`，`close`** 四个函数指针

讲一下思路

1. 伪造`meta` 后满足各种条件 使得其进入`dequeue` 通过`unlink`,构造`prev,next` 实现任意地址指针互写
通过任意地址互写指针，向`stdout_used` 写入我们伪造的`fake_stdout`地址， 通过`IO_FILE` 劫持程序执行流
到我们布置好的`fake_stdout` 上，可以找`IO_FILE` 里的一些函数`exit` `puts`在`fake_stdout`上布置`rop_chain`然后通过栈迁移的`gadget` 利用`FSOP` 劫持程序到布置的`fake_stdout`上

2. 第二种方式更麻烦 也是伪造`fake_meta` 也是任意地址指针互写，先进行布局使得 `fake_meta` `dequeue` 实现`unlink`，
在利用指针互写 修改`fake_meta` 中的`mem`(`mem` 就是`group` 区域) ，把`mem` 修改为我们想要的地址，
然后让`fake_meta` 通过`queue` 入队，可以实现任意地址分配的，然后同样是打 `IO_FILE` 通过修改`stdout` `stdin` 和`stderr` 结构体 劫持程序流

### 0x03 补充:

前面提到过

- 第一种:如果一个`group` 中所有的`chunk` 都已经被使用，且没有`free`掉的`chunk`
- 第二种:`group` 中的`chunk` 当`free`掉最后一个`chunk`，都处于`freed`的状态

处于这两种状态的`group` 对应的`meta` 会被`dequeue` 出队

1. 第一种状态，在`malloc` 时候触发`dequeue` ，当前`meta`出队

```c
/* src/malloc/mallocng/malloc.c */
static uint32_t try_avail(struct meta **pm)
{
  struct meta *m = *pm;
  uint32_t first;
  if (!m) return 0;
  uint32_t mask = m->avail_mask;
  if (!mask)
  {
    // 这里如果mask==0，含义就是group中的chunk已被全部使用，没有空闲了
    if (!m) return 0;
    if (!m->freed_mask) // 并且没有已经释放的chunk
    {
      dequeue(pm, m); // 直接让这个meta 出队，因为group已经没有可用空间了
      m = *pm;
      if (!m) return 0;
    }
    else {
      m = m->next;
      *pm = m;
    }
    ....
  }
  .....
}
```

2. 第二种状态在`free`时候，在`nontrivial_free()`中触发`dequeue` ，当前`meta` 出队，不被使用
