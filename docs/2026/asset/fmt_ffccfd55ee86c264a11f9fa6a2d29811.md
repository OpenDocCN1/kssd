
goto 2[link](<<mark class="encrypted">09eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8r3W2^5K9i4u0Q4x3X3g2T1L8$3!0@1L8r3W2F1i4K6u0W2j5$3!0E0i4K6u0r3L8r3W2F1N6i4S2Q4x3V1k6$3y4W2)9J5k6e0k6Q4x3V1k6K6L8%4g2J5j5$3g2Q4x3V1k6E0L8g2)9J5c8Y4y4D9N6h3u0Q4x3X3g2U0i4K6t1K6e0o6x3J5x3e0l9`.</mark>>)

如果per-CPU的partial slabs是空的

goto 4[link](<<mark class="encrypted">9f5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8r3W2^5K9i4u0Q4x3X3g2T1L8$3!0@1L8r3W2F1i4K6u0W2j5$3!0E0i4K6u0r3L8r3W2F1N6i4S2Q4x3V1k6$3y4W2)9J5k6e0k6Q4x3V1k6K6L8%4g2J5j5$3g2Q4x3V1k6E0L8g2)9J5c8Y4y4D9N6h3u0Q4x3X3g2U0i4K6t1K6e0o6x3J5x3e0x3`.</mark>>)

4. 4. allocating from per-node partial slabs (`kmem_cache_node->partial`)

如果有per-node的partial slabs：

首先将链表中的第一个脱链，并指定为active slabs[link](<<mark class="encrypted">6fdK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8r3W2^5K9i4u0Q4x3X3g2T1L8$3!0@1L8r3W2F1i4K6u0W2j5$3!0E0i4K6u0r3L8r3W2F1N6i4S2Q4x3V1k6$3y4W2)9J5k6e0k6Q4x3V1k6K6L8%4g2J5j5$3g2Q4x3V1k6E0L8g2)9J5c8Y4y4D9N6h3u0Q4x3X3g2U0i4K6t1K6e0o6t1K6x3o6V1`.</mark>>)；然后移动一些(最多`cpu_partial_slabs / 2`[link](<<mark class="encrypted">05eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8r3W2^5K9i4u0Q4x3X3g2T1L8$3!0@1L8r3W2F1i4K6u0W2j5$3!0E0i4K6u0r3L8r3W2F1N6i4S2Q4x3V1k6$3y4W2)9J5k6e0k6Q4x3V1k6K6L8%4g2J5j5$3g2Q4x3V1k6E0L8g2)9J5c8Y4y4D9N6h3u0Q4x3X3g2U0i4K6t1K6e0o6t1K6x3e0V1`.</mark>>))per-node的slabs到per-CPU的partial list[link](<<mark class="encrypted">9dbK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8r3W2^5K9i4u0Q4x3X3g2T1L8$3!0@1L8r3W2F1i4K6u0W2j5$3!0E0i4K6u0r3L8r3W2F1N6i4S2Q4x3V1k6$3y4W2)9J5k6e0k6Q4x3V1k6K6L8%4g2J5j5$3g2Q4x3V1k6E0L8g2)9J5c8Y4y4D9N6h3u0Q4x3X3g2U0i4K6t1K6e0o6t1K6x3e0x3`.</mark>>)；再去active slab重新分配。[link](<<mark class="encrypted">c2aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8r3W2^5K9i4u0Q4x3X3g2T1L8$3!0@1L8r3W2F1i4K6u0W2j5$3!0E0i4K6u0r3L8r3W2F1N6i4S2Q4x3V1k6$3y4W2)9J5k6e0k6Q4x3V1k6K6L8%4g2J5j5$3g2Q4x3V1k6E0L8g2)9J5c8Y4y4D9N6h3u0Q4x3X3g2U0i4K6t1K6e0o6x3J5x3U0l9`.</mark>>)

如果per-node partial list 为空，goto 5

5. 5. Create new slab

[allocate](<<mark class="encrypted">737K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8r3W2^5K9i4u0Q4x3X3g2T1L8$3!0@1L8r3W2F1i4K6u0W2j5$3!0E0i4K6u0r3L8r3W2F1N6i4S2Q4x3V1k6$3y4W2)9J5k6e0k6Q4x3V1k6K6L8%4g2J5j5$3g2Q4x3V1k6E0L8g2)9J5c8Y4y4D9N6h3u0Q4x3X3g2U0i4K6t1K6e0o6x3J5x3U0x3`.</mark>>) from new slab的过程：

首先从`page_alloc`中分配新的slab，并放进freelist中，并指定为active slab，然后从该slab中分配对象。

### explotion case

1. Out-of-bounds, case #1 (Shaping Slab memory)

攻击所需条件：

1. 需要一个内核bug能导致OOB；

2. 有两个不同的系统调用，一个可以分配object(`IOCTL_ALLOC`)，一个可以OOB(`IOCTL_OOB`)；

3. 能够leak或者overwrite的目标object；

4. 能将可利用的object和targetobject挨着放在一起。

攻击过程：

1. allocate 足够的targt objects 来获取新的active slab；需要填充所有的holes达到分配过程的第五步。

所以我们就需要找到有多少个holes。

但是在非特权的目标系统上，没有方法能够找到确切的数目。 `/proc/slabinfo` 和相关文件对于普通用户不可读。

而且我们可能拥有的空闲插槽数量没有上限，原因是atcive slab上的holes数量最多有每一个slab上的objects的数目。

per-CPU partials的holes数量上限是每一个slab上的objects的数目 x `cpu_partial_slabs`。

per-node partials的没有限制slabs的数量。

所以一种方式是估计，首先重现目标环境，运行相同的版本内核，运行相同的软件，然后我们通过 `cat /proc/slabinfo` 看有多少个holes。

| name | <active_objs> | <num_objs> | <objsize> | <objperslab> | <pagesperslab> |
|---|---|---|---|---|---|
| `cred_jar` | 7644 | 7644 | 192 | 21 | 1 |
| `kmalloc-8k` | 456 | 460 | 8192 | 4 | 8 |
| `kmalloc-4k` | 3118 | 3160 | 4096 | 8 | 8 |
| `kmalloc-2k` | 3621 | 3696 | 2048 | 16 | 8 |
| `kmalloc-32` | 54789 | 55808 | 32 | 128 | 1 |

`active_objs`: 已经分配的objects的数量，

`num_objs`: 现存slab中的slots的总数。

这个值不是实时更新的，只有在一个slab被分配，释放或者移动到per-node partial list时才会更新。

Shrink cache 可以获得更准确的值，

`echo 1 | sudo tee /sys/kernel/slab/kmalloc-32/shrink`

但是这样会导致这个cache释放fully free slabs。

| # name | <active_objs> | <num_objs> |
|---|---|---|
| `kmalloc-32` | 25216 | 25216 // Before shrinking. |
| `kmalloc-32` | 23132 | 24320 // After shrinking. |

比如这个就少了1000多个，这个就是不准确的，即是我们复制来环境也不准确。

1. 现在假设我们分配了足够的target objects并获得了一个新的active slab。并且新的active slab被target objects填充一部分；

2. 现在通过`IOCTL_ALLOC`操作分配一个vulnerable object；

现在分配足够的target objects填满active slab。现在slab变成full，尽管可能会变成非active，但是没关系。

现在内存看起来是这样：

```C
+-------+-------+-------+-------+-------+-------+-------+-------+
| Target| Target| Target| Vuln | Target| Target| Target| Target|
+-------+-------+-------+-------+-------+-------+-------+-------+
```

3. 现在通过`IOCTL_OOB`触发越界访问。

```C
+-------+-------+-------+-------+-------+-------+-------+-------+
| Target| Target| Target| Vuln | Target| Target| Target| Target|
+-------+-------+-------+-------+-------+-------+-------+-------+
|\_\_\_\_\_\_\_| OOB
```

（注：如果没有第一步，我们就无法破坏target，并且可能会破坏内核其他数据，后果不可控。所以第一步是为了explition的稳定。

除此之外这个exp也有一些问题，比如:

如果vuln被allocated到最后一个object，这就有概率会失败。解决的办法就是在其后多分配一个slab，然后填充target。

Migration: 进程被移动到另一个CPU上执行了。解决办法：绑定CPU的亲和性

Preempting: 另一个进程或者中断处理来抢占此CPU，解决方法：减少slab shaping的时间；使用less noisy（不那么频繁） 的cache。）

1. Out-of-bounds, case #2 （Shaping Slab memory）

需要条件：分配vulnerable objects并且立即写数据触发OOB（`IOCTL_ALLOC_AND_OOB`），

攻击过程：

1. 分配足够多的target objects以获得新的 active slab；

2. 分配一个vulnerable object并且触发OOB通过`IOCTL_ALLOC_AND_OOB`，

这有两种情况，

case #1: OOB访问的区域在free slot中，如果OOB的范围很小，没有覆盖元数据，则不会发生任何事情。可以重复进行OOB操作。

```C
+-------+-------+-------+-------+-------+-------+-------+-------+
| Target| | Target| Vuln | | Target| Target| Target|
+-------+-------+-------+-------+-------+-------+-------+-------+
|\_\_\_\_\_\_\_| OOB
```

case #2: OOB访问的区域在target object中

Success！！！ 但是也许需要很多次重试才能成功

```C
+-------+-------+-------+-------+-------+-------+-------+-------+
| Target| | Target| Vuln | Target| Target| Target| Target|
+-------+-------+-------+-------+-------+-------+-------+-------+
|\_\_\_\_\_\_\_| OOB
```

### Freeing process and explition

1. case #1: object 属于active slab，

object加入无锁的per-CPU的freelist的头部。[link](<<mark class="encrypted">15bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8r3W2^5K9i4u0Q4x3X3g2T1L8$3!0@1L8r3W2F1i4K6u0W2j5$3!0E0i4K6u0r3L8r3W2F1N6i4S2Q4x3V1k6$3y4W2)9J5k6e0k6Q4x3V1k6K6L8%4g2J5j5$3g2Q4x3V1k6E0L8g2)9J5c8Y4y4D9N6h3u0Q4x3X3g2U0i4K6t1K6e0o6x3%4y4U0j5`.</mark>>)

想象一种场景:

```C
void *ptr1 = kmalloc(128, GFP_KERNEL);
free(ptr1);
void *ptr2 = kmalloc(128, GFP_KERNEL);
free(ptr2);
void *ptr3 = kmalloc(128, GFP_KERNEL);
```

`ptr1`，`ptr2`，`ptr3`都指向同一个object。

所以这就引出第一种利用场景(UAF)

所需条件：假设我们有UAF的漏洞：

1. 分配vulnerable object （`IOCTL_ALLOC`）

2. free vulnerable object （`IOCTL_FREE`）

3. 在`IOCTL_FREE`后，读写vulnerable object的数据，（`IOCTL_UAF`）

攻击过程：

1. 通过`IOCTL_ALLOC`分配一个vulnerable object，

2. 通过`IOCTL_FREE` free vulnerable object，悬空引用仍然存在；

3. 分配一个target object，现在那个悬空指针指向它；

4. 现在能够使用`IOCTL_UAF`触发UAF访问。

2. case #2: object属于一个non-full slab

free object到所属的freelist之中。[link](<<mark class="encrypted">b6eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8r3W2^5K9i4u0Q4x3X3g2T1L8$3!0@1L8r3W2F1i4K6u0W2j5$3!0E0i4K6u0r3L8r3W2F1N6i4S2Q4x3V1k6$3y4W2)9J5k6e0k6Q4x3V1k6K6L8%4g2J5j5$3g2Q4x3V1k6E0L8g2)9J5c8Y4y4D9N6h3u0Q4x3X3g2U0i4K6t1K6e0o6x3$3y4U0p5`.</mark>>)

如果slab是per-node的，并且变成了fully free，并且node有足够的per-node slabs。该slab会被从per-node中移除并[free](<<mark class="encrypted">490K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8r3W2^5K9i4u0Q4x3X3g2T1L8$3!0@1L8r3W2F1i4K6u0W2j5$3!0E0i4K6u0r3L8r3W2F1N6i4S2Q4x3V1k6$3y4W2)9J5k6e0k6Q4x3V1k6K6L8%4g2J5j5$3g2Q4x3V1k6E0L8g2)9J5c8Y4y4D9N6h3u0Q4x3X3g2U0i4K6t1K6e0o6x3$3z5o6M7`.</mark>>)回page allocator中。
