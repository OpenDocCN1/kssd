
```c
#define MIN_CHUNK_SIZE (offsetof(struct malloc_chunk, fd_nextsize))
```

chunk除了有最小值的要求外，还有对齐的要求，一般来讲，GLibC要求跟16对齐。

```c
#define MALLOC_ALIGNMENT 16
#define MALLOC_ALIGN_MASK (MALLOC_ALIGNMENT - 1)
```

### 堆箱 - bin

被释放的chunk并不会马上将内存归还给内核，GLibC的堆管理器ptmalloc会继续管理着空闲chunk，当申请内存的请求再次被提交时，ptmalloc会从空闲chunk中挑选一个合适的chunk，把它交给程序使用。

ptmalloc会根据chunk大小进行分类，同类型的chunk会放入箱子bin内收纳，bin可以分成`fast bin`、`small bin`、`large bin`、`unsorted bin`四大类。

```c
struct malloc_state
mfastbinptr fastbinsY[NFASTBINS]
mchunkptr bins[NBINS * 2 - 2]
unsigned int binmap[BINMAPSIZE]
```

bin的信息由记录堆状态的`malloc_state`结构体进行维护。

`fast bin`放在`malloc_state`的`fastbinsY`内，其余类型的bin放在`bins`内。

#### fast bin

`fast bin`用于存放比`global_max_fast`小的chunk，`global_max_fast`的大小一般都会是128，设计`fast bin`的初衷，就是快速的对小内存进行分配和释放。

```c
#define set_max_fast(s)                                                 \
global_max_fast = (((size_t) (s) <= MALLOC_ALIGN_MASK - SIZE_SZ) \
? MIN_CHUNK_SIZE / 2 : ((s + SIZE_SZ) & ~MALLOC_ALIGN_MASK))
(gdb) p global_max_fast
$38 = 128
```

`fastbinsY`在`malloc_state`中被定义成了一个数组，数组的容量是`NFASTBINS`，它的值一般都是10。

```c
struct malloc_state
mfastbinptr fastbinsY[NFASTBINS]
```

`fastbinsY`中存储的`NFASTBINS`个`fast bin`链表，链表根据chunk的大小进行区分，链表对应chunk的大小越小，该链表在数组中的排名越靠前。好处就是，`fastbinsY`数组中的元素对应的`fast bin`大小是固定的，免去了按大小进行排序的需求。

GLibC中提供了一个名为`fastbin_index`的宏，它会接收chunk的大小，然后将它右移4位后再减去2，右移四位可以消除对齐`MALLOC_ALIGNMENT`（值一般为16）的影响，减2可以消除最小值`MIN_CHUNK_SIZE`的影响，从而得到chunk大小在数组中的索引值。

```c
(gdb) p sizeof(size_t)
$39 = 8

#define INTERNAL_SIZE_T size_t
#define SIZE_SZ (sizeof (INTERNAL_SIZE_T))
#define fastbin_index(sz) \
((((unsigned int) (sz)) >> (SIZE_SZ == 8 ? 4 : 3)) - 2)
```

通过`fastbin_index`和`fast bin`两个宏，快速定位到`fastbinsY`数组中的链表。

```c
idx = fastbin_index (nb)
mfastbinptr *fb = &fastbin (av, idx)

#define fastbin(ar_ptr, idx) ((ar_ptr)->fastbinsY[idx])
```

程序申请的内存大小肯定不可能是刚好和`MALLOC_ALIGNMENT`对齐的，GlibC贴心的提供了宏`csize2tidx`，用于获取申请内存在`fastbinsY`中对应的下标找到空闲的chunk。

```c
# define csize2tidx(x) (((x) - MINSIZE + MALLOC_ALIGNMENT - 1) / MALLOC_ALIGNMENT)
```

`fast bin`之所以孤立其他的bin，是因为它需要具备一些特殊的属性。`fast chunk`的chunk信息中的`mchunk_size`不止记录了chunk的大小，低比特位还记录了chunk的属性，对于`fast chunk`来讲，比特位`PREV_INUSE`永远为1，保证`fast chunk`不会被合并。

`fast chunk`在一般情况下不会被合并，但在二般情况下就不好说了（碎片化的内存数量过多），`fast chunk`不容易被合并的特点，保障了它能被快速的释放和取出使用。

`fast bin`还有一个特点，就是它采用LIFO后进先出的策略。尽管CPU和内存的通信速度已经很快了，但相当于CPU自身的运行速度还是太慢了，所以CPU一般都会设置CPU缓存，CPU缓存的容量有限，不会像内存那样大，但是CPU和CPU缓存间的通信速度是相当快的，假如CPU经常使用的内存数据都位于CPU缓存上，那么CPU的执行效率就会大大提高，LIFO可以帮助我们达到这一目的。

一般来讲，越晚使用的内存数据留存在CPU缓存上几率越大，使用LIFO管理`fast bin`正是利用了这一点，提高CPU缓存命中率，保障`fast bin`能被快速的释放和取出使用。

`fast bin`只使用`struct malloc_state`中的`fd`成员，当chunk被释放进入链表时，会先取出`fast bin`链表指针`FB`，然后让空闲chunk的`fd`指向`FB`，最后更新链表指针为当前空闲chunk，经过这番操作后，链表最后的元素就是最近释放的chunk，之前的`fast bin`可以被`fd`成员索引，由于`fast bin`链表只使用了`fd`，所以该链表是单向的。

从空闲的`fast bin`链表中取出chunk时，当然就是进行反向操作，先拿出最晚入列的chunk，最后将`fast bin`链表更新为`fd`指向的后续链表信息。

```c
enter -> P->fd = *FB; *FB = P;
leave -> P = *FB; *FB = P->fd;
```

上面的操作只是一个假想，在实际情况中`p`并不是缓冲区变量对应的`malloc_chunk`地址，而是缓冲区变量地址`-0x10`，这样做的好处就是链表中存放的直接就是缓冲区变量地址，不需要根据`malloc_chunk`进行进一步索引。

```c
__libc_free
-> mem -> free bufferr address
-> example -> free(0x465060)
-> mem = 0x465060
-> p = mem2chunk (mem)
-> p = mem - 0x10
-> _int_free (ar_ptr, p, 0);
-> p->fd = PROTECT_PTR (&p->fd, old);
-> *fb = p;
_int_malloc
-> victim = *fb;
-> *fb = REVEAL_PTR (victim->fd);
-> void *p = chunk2mem (victim);
-> return p;
```

如果arena中拥有空闲的`fast chunk`，那么在arena对应的`struct malloc_state`，不止会发现`fastbinsY`存在不为`NULL`的元素，也会发现`malloc_state->have_fastchunks`成员的数值也不是0。

在实际情况中，如果对`fastbinsY`数组中链表上的地址进行观察，会发现实际地址会有一定量的偏移，这个偏移值是宏`PROTECT_PTR`产生的，它只保证链表中最后一个空闲chunk的地址是原始且正确的，其余的地址都会经过`PROTECT_PTR`完成地址随机化。

```c
#define PROTECT_PTR(pos, ptr) \
((__typeof (ptr)) ((((size_t) pos) >> 12) ^ ((size_t) ptr)))

commands
>p main_arena->fastbinsY
>p buf_pool[i]
>end

$112 = {0x0, 0x0, 0x0, 0x0, 0x0, 0x4059b0, 0x0, 0x0, 0x0, 0x0}
$113 = 0x405a30 "main"
$114 = {0x0, 0x0, 0x0, 0x0, 0x0, 0x405a20, 0x0, 0x0, 0x0, 0x0}
$115 = 0x405a30 "\265]@"
$120 = {0x0, 0x0, 0x0, 0x0, 0x0, 0x405a90, 0x0, 0x0, 0x0, 0x0}
$121 = 0x405aa0 "%^@"

p /x *(struct malloc_chunk *)(0x405a20)
$117 = {mchunk_prev_size = 0x0, mchunk_size = 0x71, fd = 0x405db5, bk = 0x0, fd_nextsize = 0x0, bk_nextsize = 0x0}
p /x *(struct malloc_chunk *)(0x405a90)
$122 = {mchunk_prev_size = 0x0, mchunk_size = 0x71, fd = 0x405e25, bk = 0x0, fd_nextsize = 0x0, bk_nextsize = 0x0}

| correct fd | current fd | offset (0x405xxx >> 12 = 0x405) |
| 0x4059b0   | 0x405db5   | 0x405                           |
| 0x405a20   | 0x405e25   | 0x405                           |
```

#### small bin和large bin

`small bin`和`large bin`的特殊之处在于，它们的大小并不固定，`small bin`的最大值是1024，而`large bin`的最小值则是1024。

```c
#define NSMALLBINS          64
#define SMALLBIN_WIDTH      MALLOC_ALIGNMENT
#define SMALLBIN_CORRECTION (MALLOC_ALIGNMENT > CHUNK_HDR_SZ)
#define MIN_LARGE_SIZE      ((NSMALLBINS - SMALLBIN_CORRECTION) * SMALLBIN_WIDTH)
```

GLibC中的`in_smallbin_range`宏可以判断出程序申请的内存应该划入`small bin`里面，还是划入`large bin`里面，`in_smallbin_range`宏的判断依据是`MIN_LARGE_SIZE`宏的数值大小。

```c
#define in_smallbin_range(sz)   \
((unsigned long) (sz) < (unsigned long) MIN_LARGE_SIZE)
#define bin_index(sz)           \
((in_smallbin_range (sz)) ? smallbin_index (sz) : largebin_index (sz))
```

上面展示的宏中有一个名为`SMALLBIN_CORRECTION`的宏，该宏出现在`MIN_LARGE_SIZE`宏以及`smallbin_index`宏中并影响着它们，至于`SMALLBIN_CORRECTION`宏在不在其他宏中起实际作用，取决于`MALLOC_ALIGNMENT`是否大于`CHUNK_HDR_SZ`。

GLibC会接收程序申请的内存大小，然后根据改申请的大小判断它在`bins`数组中的下标，其中`small bin`通过`smallbin_index`获取下标，`large bin`通过`largebin_index_64`获取下标。

```c
#define smallbin_index(sz) (((unsigned) (sz)) >> 4) + SMALLBIN_CORRECTION

#define largebin_index_64(sz)                                                \
(((((unsigned long) (sz)) >> 6) <= 48) ? 48 + (((unsigned long) (sz)) >> 6) :\
......)
#define largebin_index(sz) largebin_index_64
```

结构体`struct malloc_state`中的`bins[NBINS * 2 - 2]`数组元素的数量是比较奇怪的，按照道理来讲，一个`NBINS`就足够了，为什么要`NBINS * 2 - 2`呢？

首先内存有最小值的限制`MALLOC_ALIGNMENT`，所以`sz >> 4`永远都不会是0，所以`bins`数组元素的数量就变成了`NBINS - 1`，又因为`bins`数组中的链表都是双向双向链表，存储着`fd`和`bk`两个成员，所以`bins`数组就扩成了`2*(NBINS - 1)`。

由于奇怪的`bins`数组，所以GLibC提供`bin_at`接口索引数组，但不幸的是，`bin_at`接口的索引方式更加奇怪。

首先`bins[(i-1)*2]`中的`(i-1)*2`指的是`bins`数组中第`i`对`fd`和`bk`成员。

```c
#define bin_at(m, i)    \
(mbinptr) (((char *) &((m)->bins[((i) - 1) * 2])) - offsetof (struct malloc_chunk, fd))
```

找到第`i`对元素后会取出元素在数组中的内存地址，最后再减去`fd`的偏移值，这是一个很奇怪但又不奇怪的操作。

在`bins`数组存放数据的角度中，数组中的每个元素放置的都是由`stuct malloc_chunk`描述的空闲chunk信息。

但在`bins`数组的设计角度来看，每`bins[i * 2]`和`bins[i * 2 - 1]`构成了第`i`对空闲chunk链表，`bin_at`用于检索空闲chunk链表，而不是某个空闲chunk。
