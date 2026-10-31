
```
if (fwd != bck) -> ......
else -> ......
```

- 如果链表不是空的话，逻辑会复杂些。

首先呢，GlibC会检查，`victim`的大小`size`是否小于最早入链的chunk。

你应该还记得，`bins`数组中有好多对链表，每对链表都有`fd`和`bk`两个链表，其中链表`fd`的规则是LIFO，而`bk`链表则是FIFO，它们共同组成双向循环链表。

其中`bins[(i - 1) * 2]`存放`fd`链表的头成员，`bins[(i - 1) * 2 + 1]`存储`bk`链表的头成员，`fd`链表的头成员是最晚入链的chunk，`bk`链表的头成员是最早入链的chunk。

```
if ((size) < chunksize_nomask (bck->bk))
```

a. 当待入链的`victim`比最早入链chunk小的时候，会将最晚入链的`fwd`重新赋值成链表尾，然后再更新`fd_nextsize`和`bk_nextsize`。

其中`victim`的`fd_nextsize`指向最晚入链chunk，`bk_nextsize`则指向最晚入链chunk的`bk_nextsize`。

因为即将有更小的`victim`将要入链，所以会将最晚入链chunk的`bk_nextsize`以及另一个不同大小chunk的`fd_nextsize`更新成`victim`。

`fd_nextsize`和`bk_nextsize`是专门为`large bin`而设的，它们与`fd`和`bk`不同，`fd`和`bk`维护的是相同大小组成的chunk，`xx_nextsize`则维护同一链表中但大小不相同的chunk。

之所以说`xx_nextsize`是为`large bin`而设的，是因为同一链表中维护大小不相同的chunk这种情况，只有`large bin`链表才具备。

最后，`victim`作为新的最晚入链chunk，GLibC会让`bk`指向最早入链的`bck`，而`fd`指向链表尾`fwd`，再更新`fwd`的`bk`以及`bck`的`fd`更新为`victim`，目的是完成`victim`插入链表的操作。

你可能会好奇，`fwd->bk`应该是等价于`bins[(i - 1) * 2 + 1]`的，也就是等价于链表`bk`的头成员，在这里`victim`并非是最早入链的chunk，有什么要更新呢？

这是因为`bk`链表的FIFO原则需要为按大小排序的原则让步。

```
if ((size) < chunksize_nomask (bck->bk)) {
    fwd = bck;
    bck = bck->bk;
    victim->fd_nextsize = fwd->fd;
    victim->bk_nextsize = fwd->fd->bk_nextsize;
    fwd->fd->bk_nextsize = victim->bk_nextsize->fd_nextsize = victim;
}
victim->bk = bck;
victim->fd = fwd;
fwd->bk = victim;
bck->fd = victim;
```

b. 当`victim`大于等于最早入链的chunk时，GLibC会遍历`fd_nextsize`链表。

```
if ((size) < chunksize_nomask (bck->bk))
else -> while (size < chunksize_nomask (fwd))
    -> fwd = fwd->fd_nextsize;
```

遍历`fd_nextsize`链表是为了找到一个比`victim`大或相等的chunk。

当大小相等时，就拿出当前chunk的前一个chunk，后面`victim`插入链表时，自己作为最晚入链的chunk，`bk`当然指向链表尾，`fd`继承旧chunk的前一个chunk，它被设置成`fwd->fd`，最后建立`fwd->fd`和`victim`的联系完成插入动作。

```
if ((size) < chunksize_nomask (bck->bk))
else -> while (size < chunksize_nomask (fwd))
    -> fwd = fwd->fd_nextsize;
    -> if (size == chunksize_nomask (fwd))
        -> fwd = fwd->fd;
    -> bck = fwd->bk;

victim->bk = bck;
victim->fd = fwd;
fwd->bk = victim;
bck->fd = victim;
```

如果是大于的话，就会先将`victim`插入`xx_nextsize`维护的链表当中。

```
bck = bin_at (av, victim_index);

if ((size) < chunksize_nomask (bck->bk))
else -> while (size < chunksize_nomask (fwd))
    -> fwd = fwd->fd_nextsize;
    -> if (size == chunksize_nomask (fwd))
        -> fwd = fwd->fd;
    -> else
        -> victim->fd_nextsize = fwd;
        -> victim->bk_nextsize = fwd->bk_nextsize;
        -> fwd->bk_nextsize = victim;
        -> victim->bk_nextsize->fd_nextsize = victim;
    -> bck = fwd->bk;

victim->bk = bck;
victim->fd = fwd;
fwd->bk = victim;
bck->fd = victim;
```

- 如果链表为空就会直接将`victim`插入链表。

因为链表中没有其他的chunk，所以`xx_nextsize`只能更新成`victim`的地址。

```
-> if (fwd != bck) -> ......
-> else -> victim->fd_nextsize = victim->bk_nextsize = victim;

victim->bk = bck;
victim->fd = fwd;
fwd->bk = victim;
bck->fd = victim;
```

在这里，不管`victim`是属于`small chunk`还是`large chunk`，它都会被移除链表`unsorted bin`中，转移到`small bin`或`large bin`内。

### large bin 的匹配

经过前面的操作后，`fast chunk`、`small chunk`以及`unsorted chunk`都有了被使用的机会，但`large bin`呢？

有的，兄弟，肯定有的。

到这个时候没有匹配到的话，GLibC会针对`large bin`链表进行匹配。

首先做的自然还是判断大小，只要不属于`small bin`那就是`large bin`了。

```
if (!in_smallbin_range (nb))
```

此时GLibC会继续判断链表是否为空以及`fd`链表头的chunk大小是否超过申请大小。

```
-> bin = bin_at (av, idx);
-> if ((victim = first (bin)) != bin && (unsigned long) chunksize_nomask (victim) >= (unsigned long) (nb))
```

当链表不为空且最晚入链chunk超过申请大小时，GLibC会先针对`bk_nextsize`链表进行遍历，目的是找到一个最接近申请大小且大于申请大小的chunk。

```
while (((unsigned long) (size = chunksize (victim)) < (unsigned long) (nb)))
    -> victim = victim->bk_nextsize;
```

找到符合的chunk后，GLibC会判断当前chunk是不是最早入链，且链表中存在与当前chunk大小相同的chunk，如果是就让当前chunk变成`victim->fd`。

这么做是为了避免移除最早入链chunk后，需要再次维护链表信息。

```
if (victim != last (bin) && chunksize_nomask (victim) == chunksize_nomask (victim->fd))
    victim = victim->fd;
```

拿到可用的`victim`后，GlibC会对`victim`进行切割，并将切割后的剩余大小保存在`remainder_size`中。

然后通过`unlink_chunk`将`victim`从`large bin`链表中移出来。

```
remainder_size = size - nb;
unlink_chunk (av, victim);
```

最后会判断`remainder_size`的大小，当`remainder_size`小于`MINSIZE`时，说明切割后的大小不满足最小尺寸的要求，此时就不会切割。

然后直接使用chunk的原大小`size`偏移到高地址chunk，目的是告诉上一个chunk，当前chunk已经被使用了。

如果切割后的大小满足最小尺寸的要求，就会将切割后的部分`remainder`放入链表`unsorted bin`当中（这里通过申请大小`nd`偏移到被切割的chunk）。

处理好切割部分后，就将可用的chunk返回，交给程序进行使用。

```
if (remainder_size < MINSIZE)
    -> set_inuse_bit_at_offset (victim, size);
else
    -> remainder = chunk_at_offset (victim, nb);
    -> bck = unsorted_chunks (av);
    -> fwd = bck->fd;
    -> remainder->bk = bck;
    -> remainder->fd = fwd;
    -> bck->fd = remainder;
    -> fwd->bk = remainder;
return p;
```

### 扩大范围进行搜索

假如`large bins`提供的chunk还没有被命中怎么办呢？

不要担心，GLibC自有办法处理，接下来GlibC会用到`struct malloc_state`结构体的`binmap`成员。

#### binmap 的介绍

`binmap`是什么呢，首先它结构体中的定义可以知道，它是一个容量为4的数组，数组中记录着`bins`数组中的链表是否为空，用于辅助GLibC遍历链表。

```
#define BINMAPSHIFT      5
#define BITSPERMAP       (1U << BINMAPSHIFT)
#define BINMAPSIZE       (NBINS / BITSPERMAP)

struct malloc_state {
    ......
    unsigned int binmap[BINMAPSIZE];
    ......
};
```

下面是几个关于`binmap`的宏定义信息，这些宏揭示了`binmap`是如何辅助遍历的。

```
#define BINMAPSHIFT         5
#define idx2block(i)        ((i) >> BINMAPSHIFT)
#define idx2bit(i)          ((1U << ((i) & ((1U << BINMAPSHIFT) - 1))))
#define mark_bin(m, i)      ((m)->binmap[idx2block (i)] |= idx2bit (i))
#define unmark_bin(m, i)    ((m)->binmap[idx2block (i)] &= ~(idx2bit (i)))
#define get_binmap(m, i)    ((m)->binmap[idx2block (i)] & idx2bit (i))
```

`bins[NBINS * 2 - 2]`数组的容量是`254 = 128 * 2 - 2`，其中0到1的位置会留给`unsorted bin`使用，从下标`126 = (64 - 1) * 2`开始是`large bin`链表，下标2到下标125的空间是`small bin`链表。

```
bin_at(M, 1) -> unsorted bin
bin_at(M, 2) - bin_at(M, 63) -> small bin
bin_at(M, 64) - bin_at(M, 127) -> large bin
```

`binmap`一共有`BINMAPSIZE`元素，`BINMAPSIZE`一般为4，`binmap`将`NBINS`按照`BITSPERMAP`划分成4个block，每个block一般占有`32 = 1 << 5`个比特。

在GLibC的概念中，每个block都对应一个map，每个map都对应着32个链表。

在`bin_at`宏的眼中，一共有127个链表，`idx2block`宏可以将`bin_at`宏的索引值转换为`binmap`的block，通过block可以在`binmap`中找到map。

`idx2block`宏将`bin_at`宏使用的索引值右移5位，因为索引值最大就是127，其二进制格式为`0111 1111`，右移后刚好是block的最大值3，其余的数值右移会分别对应2，1，0。

对于map来讲，它里面存放着32个链表的标志位，标志位存在则代表链表不为空。

标志位的赋值只在`unsorted bin`中的chunk迁移到`small bin`或`large bin`时发生，`mark_bin`宏会设置`binmap[block]`中map的标志位。

可以通过`get_binmap`比较标志位，判断特定链表是否为空。

`unmark_bin`宏则是可以清除map中的比特位。

```
_init_malloc -> while ((victim = unsorted_chunks (av)->bk) != unsorted_chunks (av))
    -> mark_bin (av, victim_index);
```

map中特定链表的比特位是通过`idx2bit`宏计算的。

`binmap`被定义成了`unsigned int`，`unsigned int`一般占用32个比特位，刚好可以对应map管理的32个链表。
