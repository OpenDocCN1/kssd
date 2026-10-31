
# [原创]large Bin Attack学习（\_int\_malloc源码细读）

> 参考文章:

wiki:<a href="elink@6f3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0N6r3k6Q4x3X3c8%4K9h3E0A6i4K6u0W2L8%4u0Y4i4K6u0r3M7s2N6F1i4K6u0r3L8r3W2F1N6i4S2Q4x3V1k6#2M7$3g2J5i4K6u0V1L8h3!0V1k6g2)9J5c8X3S2W2j5i4m8Q4x3V1k6H3N6r3#2S2L8r3I4G2j5K6u0Q4x3V1k6D9j5i4u0Y4k6g2)9J5k6r3u0A6L8W2)9J5k6r3q4@1N6r3q4U0K9#2)9J5c8R3`.`.">Large Bin Attack - CTF Wiki (ctf-wiki.org)</a>

源码级调试glibc:<a href="elink@83eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8X3q4K6N6s2u0G2N6s2W2U0L8$3!0F1i4K6u0r3j5i4u0@1K9h3y4D9k6g2)9J5c8X3c8W2N6r3q4A6L8s2y4Q4x3V1j5#2x3U0j5$3x3U0j5^5y4b7`.`.">源码级调试glibc\_glibc cannot be compiled without optimization-CSDN博客</a>

源码分析:<a href="elink@9cbK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3L8s2g2G2L8r3g2I4K9g2)9J5c8Y4m8Q4x3V1j5I4y4e0f1J5x3o6j5J5x3g2)9J5k6h3S2@1L8h3H3`.">glibc 2.31 malloc与free 源码分析（持续更新） - PwnKi - 博客园 (cnblogs.com)</a>+<a href="elink@75dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3L8s2g2G2L8r3g2I4K9g2)9J5c8Y4m8Q4x3V1j5I4x3U0M7K6x3e0R3%4y4g2)9J5k6h3S2@1L8h3I4Q4x3U0y4Q4y4h3k6A6L8Y4c8Q4y4h3k6E0j5h3I4D9L8$3x3`.">glibc malloc源码分析 - PwnKi - 博客园 (cnblogs.com)</a>

详细拆分了`_int_malloc`的流程 并且按照功能分了标题 想要了解对应部分就直接点击标题跳转即可

第一次阅读glibc的源码然后进行分析 有错误的地方请大佬指正

## 源码分析(largebin malloc)

> 每次去看别人文章分析总结的 总感觉比较难记住 每个libc版本的区别 然后也没彻底理解一些操作 所以进行阅读源码
>
> 然后重点是检查机制部分 如果只想看重点就直接跳转到[largebin入链操作](#largebin)

- 然后在正式阅读源码之前 我们先理清楚largebin的结构（去除了头部的`fd_nextsize`/`bk_nextsize` 为了图片干净一点）

![largebin_struct](upload/attach/202403/994584_2QBMNU43EECPWAJ.jpg)

- 我们可以简化一下 去除尾链的`fd`和头链的`bk`方便我们理清逻辑

![large_struct](upload/attach/202403/994584_FRH86PQD27AKCJB.jpg)

- 大概就是这个样子 也就是bin头部通过`fd`/`bk`链接chunk size链表的头部和尾部 然后chunk size链表之间通过`fd_nextsize`/`bk_nextsize`链接

- chunksize链表中 同一个大小的chunk通过`fd`/`bk`进行链接

- 所以largebin的`fd`和`bk`和其他的双向链不同我们不能通过从bin一路通过`fd`返回到large bin的头部

### Unsortedbin的合并/入链/分配操作

### 遍历的开始（梦的开始）

> 后面的操作中最重要的就是`Victim`变量 这个变量是当前循环到的unsortedbin chunk<br>`bck`变量 也就是`bck <-------> victim` 这个关系

- 从`unsorted_chunk`最后一位开始遍历 直到碰到`unsorted_bin`的头部 我在这里 没很确定是否`unsortedbin`可不可以指向自己 我们可以调试看看

```c
while ((victim = unsorted_chunks (av)->bk) != unsorted_chunks (av)){
bck = victim->bk;
size = chunksize (victim); /* 计算 size */
// ...
}
```

### 调试

```shell
unsortedbin
all: 0x555555559680 —▸ 0x7ffff7fb9be0 (main_arena+96) ◂— 0x555555559680
```

- 然后查看chunk结构

```shell
Free chunk (unsortedbin) | PREV_INUSE
Addr: 0x555555559680
Size: 0x90 (with flag bits: 0x91)
fd: 0x7ffff7fb9be0
bk: 0x7ffff7fb9be0
```

- 查看unsortedbin的大小

```shell
pwndbg> tel 0x7ffff7fb9be0
00:0000│ rdx r10 r11 0x7ffff7fb9be0 (main_arena+96) —▸ 0x5555555597a0 ◂— 0x0
01:0008│ 0x7ffff7fb9be8 (main_arena+104) ◂— 0x0
02:0010│ 0x7ffff7fb9bf0 (main_arena+112) —▸ 0x555555559680 ◂— 0x0
03:0018│ 0x7ffff7fb9bf8 (main_arena+120) —▸ 0x555555559680 ◂— 0x0
04:0020│ 0x7ffff7fb9c00 (main_arena+128) —▸ 0x7ffff7fb9bf0 (main_arena+112) —▸ 0x555555559680 ◂— 0x0
05:0028│ 0x7ffff7fb9c08 (main_arena+136) —▸ 0x7ffff7fb9bf0 (main_arena+112) —▸ 0x555555559680 ◂— 0x0
06:0030│ 0x7ffff7fb9c10 (main_arena+144) —▸ 0x7ffff7fb9c00 (main_arena+128) —▸ 0x7ffff7fb9bf0 (main_arena+112) —▸ 0x555555559680 ◂— 0x0
07:0038│ 0x7ffff7fb9c18 (main_arena+152) —▸ 0x7ffff7fb9c00 (main_arena+128) —▸ 0x7ffff7fb9bf0 (main_arena+112) —▸ 0x555555559680 ◂— 0x0
```

- 可以发现`fd bk`都是指向的unsortedbin中第一个chunk 我们清空unsortedbin看看

```shell
pwndbg> tel 0x7ffff7fb9be0
00:0000│ rsi r11 0x7ffff7fb9be0 (main_arena+96) —▸ 0x555555559830 ◂— 0x0
01:0008│ 0x7ffff7fb9be8 (main_arena+104) —▸ 0x555555559710 ◂— 0x90
02:0010│ 0x7ffff7fb9bf0 (main_arena+112) —▸ 0x7ffff7fb9be0 (main_arena+96) —▸ 0x555555559830 ◂— 0x0
03:0018│ 0x7ffff7fb9bf8 (main_arena+120) —▸ 0x7ffff7fb9be0 (main_arena+96) —▸ 0x555555559830 ◂— 0x0
04:0020│ 0x7ffff7fb9c00 (main_arena+128) —▸ 0x7ffff7fb9bf0 (main_arena+112) —▸ 0x7ffff7fb9be0 (main_arena+96) —▸ 0x555555559830 ◂— 0x0
05:0028│ 0x7ffff7fb9c08 (main_arena+136) —▸ 0x7ffff7fb9bf0 (main_arena+112) —▸ 0x7ffff7fb9be0 (main_arena+96) —▸ 0x555555559830 ◂— 0x0
06:0030│ 0x7ffff7fb9c10 (main_arena+144) —▸ 0x7ffff7fb9c00 (main_arena+128) —▸ 0x7ffff7fb9bf0 (main_arena+112) —▸ 0x7ffff7fb9be0 (main_arena+96) —▸ 0x555555559830 ◂— ...
07:0038│ 0x7ffff7fb9c18 (main_arena+152) —▸ 0x7ffff7fb9c00 (main_arena+128) —▸ 0x7ffff7fb9bf0 (main_arena+112) —▸ 0x7ffff7fb9be0 (main_arena+96) —▸ 0x55555555983
```

- 我们会发现`fd`和`bk`都是指向了自己本身也就是`main_arena+96`这个位置

### 安全检查机制

> 这里的安全机制全是对unsortedbin中的chunk进行的检查

- 不能小于`2*SIZE_SZ`不能大于`av->system_men`也就是该分配去的内存分配总量

```c
if (__glibc_unlikely (size <= 2 * SIZE_SZ)
|| __glibc_unlikely (size > av->system_mem))
malloc_printerr ("malloc(): invalid size (unsorted)");
```

- 对next chunk(物理意义上的紧挨着)也进行一样的操作

> `mchunkptr next = chunk_at_offset (victim, size);` /* 获得指向内存空间中当前 chunk 的下一个chunk 的指针 */

```c
if (__glibc_unlikely (chunksize_nomask (next) < 2 * SIZE_SZ)|| __glibc_unlikely (chunksize_nomask (next) > av->system_mem))
malloc_printerr ("malloc(): invalid next size (unsorted)");
```

- 检查next chunk的`prev_size` 是否等于当前的chunk size

> `size = chunksize (victim);` /* 计算 size */

```c
/* 如果 next chunk 中记录前一个 chunk 大小的 prev_size 与 size 不符，则报错 */
if (__glibc_unlikely ((prev_size (next) & ~(SIZE_BITS)) != size))
malloc_printerr ("malloc(): mismatching next->prev_size (unsorted)");
```

- 检查`bck`的`fd`是否为当前chunk 或者当前chunk的`fd`是否是bin的头结点

> `bck = victim->bk;`
>
> `victim = unsorted_chunks (av)->bk)`
>
> 应该就是检查下一个chunk是否是合法的

```c
if (__glibc_unlikely (bck->fd != victim)
|| __glibc_unlikely (victim->fd != unsorted_chunks (av)))
malloc_printerr ("malloc(): unsorted double linked list corrupted");
```

- 检查当前chunk是否是free的 通过next chunk的p值

```c
/* 如果 next chunk 中的显示前一个 chunk 是否正在使用的标志位为1，*/
/* 即前一个 chunk 正在使用，则报错 */
if (__glibc_unlikely (prev_inuse (next)))
malloc_printerr ("malloc(): invalid next->prev_inuse (unsorted)");
```

### 直接返回smallbin_chunk情况

> 然后就是从unsortedbin割small chunk 如果符合条件
>
> - 所需chunk大小在smallbin的范围之内
>
> - `bck`为unsortedbin的头 也就是unsortedbin中仅有一个chunk
> - `victim`为last remainder chunk 也就是分割过一次
> - 大小刚好大于所需`nb`大小+`MINSIZE`(这里猜测就是一个最小chunk 这样才能切割)
>
> 满足以上条件 就直接分割 然后将`victim`返回给用户

```c
if (in_smallbin_range (nb) &&
bck == unsorted_chunks (av) &&
victim == av->last_remainder &&
(unsigned long) (size) > (unsigned long) (nb + MINSIZE)) {
remainder_size = size - nb;
remainder = chunk_at_offset (victim, nb);
unsorted_chunks (av)->bk = unsorted_chunks (av)->fd = remainder;
av->last_remainder = remainder;
remainder->bk = remainder->fd = unsorted_chunks (av);
if (!in_smallbin_range (remainder_size)){
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

### 从unsortedbin中移除

> 在这里已经将chunk从unsortdbin中移除

```c
unsorted_chunks (av)->bk = bck;
bck->fd = unsorted_chunks (av);
```

### 大小刚好相等情况

- 如果chunk和当前需要的chunk大小一致 则直接返回chunk 并且设置物理意义上紧挨着的下一个chunk的size中p为0也就是free状态

```c
set_inuse_bit_at_offset (victim, size);
```

- 如果开启了tcache机制 且tcache未满则将chunk放入tcache中

```c
if (tcache_nb && tcache->counts[tc_idx] < mp_.tcache_count){
tcache_put (victim, tc_idx);
return_cached = 1;
continue;
}
```

- 然后直接返回

```c
check_malloced_chunk (av, victim, nb);
void *p = chunk2mem (victim);
alloc_perturb (p, bytes);
return p; /* 返回内存指针 */
```

### 归类入链操作

> 这里主要是将unsortedbin合并后的 入small链表或者large链表的操作
>
> 这里的`fwd`和`bck`记好了 我们从unsortedbin抠出来的chunk就要合并进入`fwd`和`bck`的中间
>
> 这后面的操作往往是先让`fwd`到指定的位置 然后`bck`通过`fwd->bk`来进行的定位

### small 和 large最终入bin操作

> 这里把最后的部分 直接提前 拿出来 因为smallbin和largebin的入链操作都含这个代码
>
> largebin还有chunk size的入链操作 以及其他的复杂检查
