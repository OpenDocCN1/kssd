
大概就是这个样子 也就是`bin`头部通过`fd`/`bk`链接`chunk size`链表的头部和尾部 然后`chunk size`链表之间通过`fd_nextsize`/`bk_nextsize`链接

`chunksize`链表中 同一个大小的`chunk`通过`fd`/`bk`进行链接

所以`largebin`的`fd`和`bk`和其他的双向链不同我们不能通过从`bin`一路通过`fd`返回到`large bin`的头部

# Unsortedbin的合并/入链/分配操作

## 遍历的开始（梦的开始）

> 后面的操作中最重要的就是`Victim`变量 这个变量是当前循环到的`unsortedbin` `chunk` `bck`变量 也就是`bck` <-------> `victim` 这个关系

从`unsorted_chunk`最后一位开始遍历 直到碰到`unsorted_bin`的头部 我在这里 没很确定是否`unsortedbin`可不可以指向自己 我们可以调试看看

```
while ((victim = unsorted_chunks(av)->bk) != unsorted_chunks(av)) {
    bck = victim->bk;
    size = chunksize(victim);
    /* 计算 size */
    // ...
}
```

## 调试

```
unsortedbin
all: 0x555555559680 —▸ 0x7ffff7fb9be0 (main_arena+96) ◂— 0x555555559680
```

然后查看`chunk`结构

```
Free chunk (unsortedbin) | PREV_INUSE
Addr: 0x555555559680
Size: 0x90 (with flag bits: 0x91)
fd: 0x7ffff7fb9be0
bk: 0x7ffff7fb9be0
```

查看`unsortedbin`的大小

```
pwndbg> tel  0x7ffff7fb9be0
00:0000│ rdx r10 r11 0x7ffff7fb9be0 (main_arena+96) —▸ 0x5555555597a0 ◂— 0x0
01:0008│             0x7ffff7fb9be8 (main_arena+104) ◂— 0x0
02:0010│             0x7ffff7fb9bf0 (main_arena+112) —▸ 0x555555559680 ◂— 0x0
03:0018│             0x7ffff7fb9bf8 (main_arena+120) —▸ 0x555555559680 ◂— 0x0
04:0020│             0x7ffff7fb9c00 (main_arena+128) —▸ 0x7ffff7fb9bf0 (main_arena+112) —▸ 0x555555559680 ◂— 0x0
05:0028│             0x7ffff7fb9c08 (main_arena+136) —▸ 0x7ffff7fb9bf0 (main_arena+112) —▸ 0x555555559680 ◂— 0x0
06:0030│             0x7ffff7fb9c10 (main_arena+144) —▸ 0x7ffff7fb9c00 (main_arena+128) —▸ 0x7ffff7fb9bf0 (main_arena+112) —▸ 0x555555559680 ◂— 0x0
07:0038│             0x7ffff7fb9c18 (main_arena+152) —▸ 0x7ffff7fb9c00 (main_arena+128) —▸ 0x7ffff7fb9bf0 (main_arena+112) —▸ 0x555555559680 ◂— 0x0
```

可以发现`fd` `bk`都是指向的`unsortedbin`中第一个`chunk` 我们清空`unsortedbin`看看

```
pwndbg> tel 0x7ffff7fb9be0

00:0000│ rsi r11 0x7ffff7fb9be0 (main_arena+96) —▸ 0x555555559830 ◂— 0x0

01:0008│         0x7ffff7fb9be8 (main_arena+104) —▸ 0x555555559710 ◂— 0x90

02:0010│         0x7ffff7fb9bf0 (main_arena+112) —▸ 0x7ffff7fb9be0 (main_arena+96) —▸ 0x555555559830 ◂— 0x0

03:0018│         0x7ffff7fb9bf8 (main_arena+120) —▸ 0x7ffff7fb9be0 (main_arena+96) —▸ 0x555555559830 ◂— 0x0

04:0020│         0x7ffff7fb9c00 (main_arena+128) —▸ 0x7ffff7fb9bf0 (main_arena+112) —▸ 0x7ffff7fb9be0 (main_arena+96) —▸ 0x555555559830 ◂— 0x0

05:0028│         0x7ffff7fb9c08 (main_arena+136) —▸ 0x7ffff7fb9bf0 (main_arena+112) —▸ 0x7ffff7fb9be0 (main_arena+96) —▸ 0x555555559830 ◂— 0x0

06:0030│         0x7ffff7fb9c10 (main_arena+144) —▸ 0x7ffff7fb9c00 (main_arena+128) —▸ 0x7ffff7fb9bf0 (main_arena+112) —▸ 0x7ffff7fb9be0 (main_arena+96) —▸ 0x555555559830 ◂— ...

07:0038│         0x7ffff7fb9c18 (main_arena+152) —▸ 0x7ffff7fb9c00 (main_arena+128) —▸ 0x7ffff7fb9bf0 (main_arena+112) —▸ 0x7ffff7fb9be0 (main_arena+96) —▸ 0x55555555983
```

我们会发现`fd`和`bk`都是指向了自己本身也就是`main_arena+96`这个位置

## 安全检查机制

> 这里的安全机制全是对`unsortedbin`中的`chunk`进行的检查

不能小于`2 * SIZE_SZ`不能大于`av->system_mem`也就是该分配去的内存分配总量

```
if (__glibc_unlikely (size <= 2 * SIZE_SZ)
|| __glibc_unlikely (size > av->system_mem))
malloc_printerr ("malloc(): invalid size (unsorted)");
```

对`next chunk`（物理意义上的紧挨着）也进行一样的操作

> `mchunkptr next = chunk_at_offset (victim, size);` /* 获得指向内存空间中当前 chunk 的下一个chunk 的指针 */

```
if (__glibc_unlikely (chunksize_nomask (next) < 2 * SIZE_SZ) || __glibc_unlikely (chunksize_nomask (next) > av->system_mem))
malloc_printerr ("malloc(): invalid next size (unsorted)");
```

检查`next chunk`的`prev_size` 是否等于当前的`chunk size`

> `size = chunksize (victim);` /* 计算 size */

```
/* 如果 next chunk 中记录前一个 chunk 大小的 prev_size 与 size 不符，则报错 */
if (__glibc_unlikely ((prev_size (next) & ~(SIZE_BITS)) != size))
malloc_printerr ("malloc(): mismatching next->prev_size (unsorted)");
```

检查`bck`的`fd`是否为当前`chunk` 或者当前`chunk`的`fd`是否是`bin`的头结点

> `bck = victim->bk;`

> `victim = unsorted_chunks (av)->bk)`

> 应该就是检查下一个`chunk`是否是合法的

```
if (__glibc_unlikely (bck->fd != victim)
|| __glibc_unlikely (victim->fd != unsorted_chunks (av)))
malloc_printerr ("malloc(): unsorted double linked list corrupted");
```

检查当前`chunk`是否是`free`的 通过`next chunk`的`p`值

```
/* 如果 next chunk 中的显示前一个 chunk 是否正在使用的标志位为1，*/
/* 即前一个 chunk 正在使用，则报错 */
if (__glibc_unlikely (prev_inuse (next)))
malloc_printerr ("malloc(): invalid next->prev_inuse (unsorted)");
```

## 直接返回smallbin_chunk情况

> 然后就是从`unsortedbin`割`small chunk` 如果符合条件

> - 所需`chunk`大小在`smallbin`的范围之内
> - `bck`为`unsortedbin`的头 也就是`unsortedbin`中仅有一个`chunk`
> - `victim`为`last remainder chunk` 也就是分割过一次
> - 大小刚好大于所需`nb`大小+`MINSIZE`（这里猜测就是一个最小`chunk` 这样才能切割）

> 满足以上条件 就直接分割 然后将`victim`返回给用户

```
if (in_smallbin_range (nb) &&
    bck == unsorted_chunks (av) &&
    victim == av->last_remainder &&
    (unsigned long) (size) > (unsigned long) (nb + MINSIZE)) {
    remainder_size = size - nb;
    remainder = chunk_at_offset (victim, nb);
    unsorted_chunks (av)->bk = unsorted_chunks (av)->fd = remainder;
    av->last_remainder = remainder;
    remainder->bk = remainder->fd = unsorted_chunks (av);
    if (!in_smallbin_range (remainder_size)) {
        remainder->fd_nextsize = NULL;
        remainder->bk_nextsize = NULL;
    }
    set_head (victim, nb | PREV_INUSE | (av != &main_arena ? NON_MAIN_ARENA : 0));
    set_head (remainder, remainder_size | PREV_INUSE);
    set_foot (remainder, remainder_size);
    check_malloced_chunk (av, victim, nb);
    void *p = chunk2mem (victim);
    alloc_perturb (p, bytes);
    return p;
}
```

## 从unsortedbin中移除

> 在这里已经将`chunk`从`unsortdbin`中移除

```
unsorted_chunks (av)->bk = bck;
bck->fd = unsorted_chunks (av);
```

## 大小刚好相等情况

如果`chunk`和当前需要的`chunk`大小一致 则直接返回`chunk` 并且设置物理意义上紧挨着的下一个`chunk`的`size`中`p`为0也就是`free`状态

```
set_inuse_bit_at_offset (victim, size);
```

如果开启了`tcache`机制 且`tcache`未满则将`chunk`放入`tcache`中

```
if (tcache_nb && tcache->counts[tc_idx] < mp_.tcache_count) {
    tcache_put (victim, tc_idx);
    return_cached = 1;
    continue;
}
```

然后直接返回

```
check_malloced_chunk (av, victim, nb);
void *p = chunk2mem (victim);
alloc_perturb (p, bytes);
return p;
/* 返回内存指针 */
```

## 归类入链操作

> 这里主要是将`unsortedbin`合并后的 入`small`链表或者`large`链表的操作

> 这里的`fwd`和`bck`记好了 我们从`unsortedbin`抠出来的`chunk`就要合并进入`fwd`和`bck`的中间

> 这后面的操作往往是先让`fwd`到指定的位置 然后`bck`通过`fwd->bk`来进行的定位

## small 和 large最终入bin操作

> 这里把最后的部分 直接提前 拿出来 因为`smallbin`和`largebin`的入链操作都含这个代码

> `largebin`还有`chunk size`的入链操作 以及其他的复杂检查

```
mark_bin (av, victim_index);
victim->bk = bck;
victim->fd = fwd;
fwd->bk = victim;
bck->fd = victim;
```

## smallbin的fwd bck赋值

如果属于`small bin`则进行`fwd`和`bck`的赋值

> `small bin` 的链表表头赋值给 `bck`: `bck = bin_at (av, victim_index);`

> 首个`chunk`赋值给`fwd` : `fwd = bck->fd;`

```
if (in_smallbin_range (size)) {
    victim_index = smallbin_index (size);
    bck = bin_at (av, victim_index);
    fwd = bck->fd;
}
```

## largebin 入bin链和chunk size链

如果属于`large_bins`同理进行赋值 然后判断该插入什么合适的位置

> 因为`largebin`是按照大小进行的排序 由大到小 所以最小的在链表最后

```
victim_index = largebin_index (size);
bck = bin_at (av, victim_index);
fwd = bck->fd;
```

判断`large`是否有空闲`chunk`:

```
if (fwd != bck)
```

如果当前`chunk`比最后一位`chunk`还小则直接加入链表末尾

> `bck`是头 `bck->bk`应该就是最后一位
