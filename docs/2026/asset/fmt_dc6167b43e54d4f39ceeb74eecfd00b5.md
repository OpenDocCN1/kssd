
# PWN入门-23-LargeBin托梦

## 针对largebin的攻击

众所周知，程序申请堆内存一般可以分成两步，首先是通过GLibC封装的`malloc`等接口函数向GLibC提出堆内存的申请，最后GLibC会通过`brk`等系统调用向实际的物理资源管理者内核申请内存。

拿到的内存会落入`top chunk`中，然后GLibC再从`top chunk`中切割chunk给程序使用，至于程序释放chunk时，chunk会有两个出路，它们分别是进入空闲chunk的管理链表和重新合并进入`top chunk`。

是否与`top chunk`合并，取决于被释放的chunk是否与`top chunk`相邻。

你不想将堆内存交给GLibC管理也不是不行，GLibC针对接口函数在内存申请其实也是依赖`brk`与`mmap`等系统调用实现的，你可以直接通过系统调用去获取堆内存，避免堆内存落在GLibC的手中。

```
struct malloc_chumalloc_statenk {
......
mfastbinptr fastbinsY[NFASTBINS];
......
mchunkptr bins[NBINS * 2 - 2];
......
}
```

GLibC对于释放时不合入`top chunk`的chunk，会将它存入对应的链表中，进入链表需要分成两个阶段，在不考虑tcache机制的影响下，第一阶段GLibC会根据chunk的大小选择进入`fastbinsY`或`unsorted bin`。

```
_int_free
-> if ((size) <= get_max_fast())
-> atomic_store_relaxed (&av->have_fastchunks, true);
-> unsigned int idx = fastbin_index(size);
-> fb = &fastbin (av, idx);
-> p->fd = PROTECT_PTR (&p->fd, old);
-> *fb = p;
-> else if (!chunk_is_mmapped(p))
-> _int_free_merge_chunk
-> _int_free_create_chunk
-> if (nextchunk != av->top)
-> mchunkptr bck = unsorted_chunks (av);
-> mchunkptr fwd = bck->fd;
-> p->fd = fwd;
-> p->bk = bck;
-> bck->fd = p;
-> fwd->bk = p;
```

第二阶段指的是chunk的迁移，迁移一般发生在`_int_malloc`中，即程序向GLibC申请堆内存的时候。

对于`fast bin`来讲，它只会在链表不为空且可用chunk空间紧张的时候，才会通过函数`malloc_consolidate`合并`fast bin`中的空闲chunk。

空闲chunk紧张的时机有两个，一是程序向GLibC提出属于`large chunk`的堆内存大小申请，二是`top chunk`空间不够，这时候GLibC都会检查`have_fastchunks`标志位，看是否有可以合并的`fast chunk`存在。

另一种迁移发生在函数`_int_malloc`通过遍历`unsorted bin`链表查找chunk的时候，当`unsorted bin`链表无法提供与申请要求匹配的chunk时，`_int_malloc`会将chunk从`unsorted bin`中取出，然后存入`small bin`或`large bin`中。

```
_int_malloc
-> if (in_smallbin_range (nb))
-> else
-> if (atomic_load_relaxed (&av->have_fastchunks))
-> malloc_consolidate (av);
-> while ((victim = unsorted_chunks (av)->bk) != unsorted_chunks (av))
-> if (in_smallbin_range (size))
-> bck = bin_at (av, victim_index);
-> fwd = bck->fd;
-> else
-> bck = bin_at (av, victim_index);
-> fwd = bck->fd;
-> xx->fd_nextsize = xxx
-> xx->bk_nextsize = xxx
-> victim->bk = bck;
-> victim->fd = fwd;
-> fwd->bk = victim;
-> bck->fd = victim;
-> use_top:
-> if ((size) >= (nb + MINSIZE))
-> else if (atomic_load_relaxed (&av->have_fastchunks))
-> malloc_consolidate (av);
```

从上面可以看出，`unsorted bin`、`small bin`以及`large bin`三大链表的主要区别还是十分明显的。

`unsorted bin`是`small chunk`和`large chunk`在释放时的第一去处，只有当链表`unsorted bin`中chunk不能满足申请者的要求时，`_int_malloc`函数才会将空闲的chunk迁入`small bin`或`large bin`中。

```
#define unsorted_chunks(M) (bin_at (M, 1))
```

而`unsorted bin`、`small bin`以及`large bin`虽然共同使用`bins`数组，但是它们在使用区域上却有着严格的划分。

其中`unsorted bin`链表固定占用`bin_at(M, 1)`的位置。

```
#define NSMALLBINS 64
#define SMALLBIN_WIDTH MALLOC_ALIGNMENT
#define MIN_LARGE_SIZE (NSMALLBINS * SMALLBIN_WIDTH)
#define in_smallbin_range(sz) \
((unsigned long) (sz) < (unsigned long) MIN_LARGE_SIZE)
#define smallbin_index(sz) (((unsigned) (sz)) >> 4)
#define largebin_index(sz) largebin_index_64
```

你应该知道`small chunk`和`large chunk`之间的界限是由`MIN_LARGE_SIZE`的大小进行划分的，GLibC在`bins`数组使用区域上的划分上依旧延续了这个套路。

`bin_at(M, 2)`到`bin_at(M, 63)`的区域是给`small bin`使用的，剩余的区域是留给`large bin`使用的。

`large bin`与`small bin`和`unsorted bin`还有一些差别。

`unsorted bin`中存储chunk的大小并不固定，`small chunk`和`large chunk`都是可能的，而`bin_at(M, 2)`到`bin_at(M, 63)`所对应的`small bin`链表，同一链表中存储的chunk大小都是固定的，只有`small bin`链表不同时，链表中所存储的chunk大小才会有差异。

```
#define largebin_index_64(sz) \
(((((unsigned long) (sz)) >> 6) <= 48) ? 48 + (((unsigned long) (sz)) >> 6) :\
((((unsigned long) (sz)) >> 9) <= 20) ? 91 + (((unsigned long) (sz)) >> 9) :\
((((unsigned long) (sz)) >> 12) <= 10) ? 110 + (((unsigned long) (sz)) >> 12) :\
((((unsigned long) (sz)) >> 15) <= 4) ? 119 + (((unsigned long) (sz)) >> 15) :\
((((unsigned long) (sz)) >> 18) <= 2) ? 124 + (((unsigned long) (sz)) >> 18) :\
126)
```

至于`bin_at(M, 64)`开始对应的`large bin`链表，如果仔细`largebin_index`宏就会发现，该宏会将某个大小范围内的chunk归类到`bin_at(M, x)`中，所以呢，单个链表`large bin`中所存放的chunk大小并不一定是相同的。

这也是`malloc_chunk`结构体中`fd_nextsize`和`bk_nextsize`两个字段存在的原因，这两个字段专门帮助`large bin`链表链接同链表中不同大小的chunk。

## 从信息泄露开始聊

一旦你对操作系统和GLibC的机制不熟悉，误认为通过`free`接口释放后，再读写被释放的内存就会出现`SIGEGV`（无效的内存访问）崩溃，那就大错特错了。

归还到GLibC手中的堆内存，GLibC并不会立即向内核提出抛弃内存的请求，所以这些内存仍然位于程序的内存布局中，对于程序来讲，释放堆内存后将对应的缓冲区变量设置成空指针才是正道。

针对UAF产生的攻击，最为常见的就是信息泄露了，由于GLibC对`malloc_chunk`结构体中`fdxx`和`bkxx`在使用阶段和释放阶段的功能划分，导致了这些字段会在释放时写入新的内存地址数据。

假如程序并没有将缓冲区变量设置成空指针，那么`fdxx`和`bkxx`在使用阶段位于数据区的特性，就会导致内存地址数据被泄露。

原来的数据区释放后摇身一变就成了可以影响chunk的功能区，一旦程序仍在操作缓冲区变量时，就相当于改写了GLibC管理空闲chunk的规则，那么造成的安全隐患当然不会止于信息泄露这一条了。

## 偷龙转凤 - 任意地址读写环境的创建

除了信息泄露之外，再比较常见的攻击方式就是任意地址读写了。

这种任意地址地址读写的漏洞，该问题也是因为`fdxx`和`bkxx`四个在使用期和释放期中作用不明确的字段导致的。

当我们手握一个已释放chunk的读写权限，那么完成`fd`位置上内存地址的泄露当然不成问题，但想要完成任意地址读写环境的创建，还需要依赖GLibC的链表管理规则。

在GLibC的空闲chunk链表的规则中，最少会用一个`fd`字段链接其余的chunk，最多就如同`large bin`一样，会用完`fdxx`和`bkxx`四个字段。

在GLibC的概念中，`fdxx`和`bkxx`用于链接其他的chunk，程序发出堆内存的申请之后，GLibC会考虑从链表中取出chunk返回给程序使用，漏洞在这个时候会显现出来，当chunk被取出之后，GLibC有个很重要的任务就是更新链表。

更新链表的依据就是`malloc_chunk`中的`fdxx`和`bkxx`四个字段，它会将根据字段上存放的地址`A`更新链表。当程序再次申请堆内存时，申请到的chunk就是地址`A`所对应的内存区域了。

根据链表的特性和可以修改已释放chunk的异常现象，我们可以控制`fdxx`和`bkxx`四个字段，让它们指向特定的内存区域，当GLibC根据`fdxx`或`bkxx`取出新地址并读写时，就完成了向我们指定的任意内存区域进行读写数据的操作。

对于GLibC来讲，就像是头上带了绿帽，孩子（被篡改的chunk）已经不是自己的了，还蒙在鼓中，尽心尽力的为了这个孩子提供服务，等孩子长大了（被移出链表）才发现，原来这个孩子是跟自己没有关系的。

并且这个孩子还可能会被刺自己，协助黑客破坏系统。

### 消失的usortedbin攻击

从上面可以看到修改`fdxx`和`bkxx`指向任意的地址，使得取出的chunk指向特定的内存区域，然后再在该区域进行读写，就是任意地址读写形成的原因。

GLibC针对`malloc_chunk`中`fdxx`和`bkxx`四个字段的策略，为我们带来了任意地址读写的可能，这种可能性需要依赖UAF，毕竟要是程序员知道`free`接口的内存释放只是伪释放，那就一定会将变量设置成空指针，进而丧失对已释放chunk的操控能力。

GLibC早就注意到了任意地址读写的问题，这一问题产生源自于GLibC早前永远默认结构体`malloc_chunk`上`fdxx`和`bkxx`中保存的数值都是正确的。

所以现在GLibC不再保持这种默认正确的规则，它会在使用前对这些数据进行检查，只要发现数据有异常就会通过`malloc_printerr`抛出错误。

比较典型的就是`unlink_chunk`函数中的检查，该函数会根据当前释放chunk的内存地址`p`的`fd`前一个入链chunk，以及`bk`获取后一个入链chunk。

按照GLibC的规则，`p->fd`的后一个入链chunk和`p->bk`的前一个入链chunk都应该指向新释放chunk的内存地址`p`，如果不是就说明`p`上的数据是有问题的。

这种依赖相邻chunk的检查，在一定程度上确保了安全性，但如果你连相邻chunk都能伪造或控制，那就不好说了。

```
unlink_chunk
-> mchunkptr fd = p->fd;
-> mchunkptr bk = p->bk;
-> if (__builtin_expect (fd->bk != p || bk->fd != p, 0))
-> malloc_printerr ("corrupted double-linked list");
```

这种任意地址读写的情况当然不止`unlink_chunk`这一处，只要`fdxx`和`bkxx`可以被恶意控制的操作都是符合要求的。

比如程序通过`_int_malloc`申请chunk的时候，会尝试遍历`unsorted bin`链表，当`_int_malloc`发现申请内存的大小不属于`small chunk`，且最近没有被切割的chunk的时候，就会将`unsorted bin`链表中的`bk`保存的最早入链chunk的再晚一个入链的chunk取出，因为该`victim->bk`之后要么与申请大小相等直接返回给使用，要么就是移入`small bin`或`large bin`中，反正就是不在`unsorted bin`里面了。

```
_int_malloc
-> for (;;)
-> while ((victim = unsorted_chunks (av)->bk) != unsorted_chunks (av))
-> bck = victim->bk;
-> if (in_smallbin_range (nb) && bck == unsorted_chunks (av) && victim == av->last_remainder && (unsigned long) (size) > (unsigned long) (nb + MINSIZE))
-> ......
-> return p;
-> unsorted_chunks (av)->bk = bck;
-> bck->fd = unsorted_chunks (av);
```

在这个时候`victim`肯定是处于空闲状态的，假如这个时候`victim`还是可以被程序写入数据的，那么`bck = victim->bk`中的`bck`被赋值后，`bck`就会指向我们可以控制的内存区域。

后面将`victim`移除链表，并将`bck`更新成`unsorted bin`的`bk`链表的头成员时，就代表GLibC将我们设置的恶意地址看作成了新的chunk。

后面取出chunk时，取出的就是恶意地址对应的内存区域，程序拿到恶意地址后，就可以对该区域进行读写。

只不过可惜，现在已经是公元2025年了，GLibC早就针对这块进行了加固，加固的方案比较简单，仍然是根据chunk相连的特性进行检查。

在正常情况下`bck`上`fd`保存的是`victim`，但如果`bck`是被篡改过的，那么它的`fd`上保存的就很难是`victim`，所以当`bck->fd != victim`时就会抛出错误。

```
_int_malloc
-> for (;;)
-> while ((victim = unsorted_chunks (av)->bk) != unsorted_chunks (av))
-> bck = victim->bk;
-> if (__glibc_unlikely (bck->fd != victim) || __glibc_unlikely (victim->fd != unsorted_chunks (av)))
-> malloc_printerr ("malloc(): unsorted double linked list corrupted");
```

### largebin攻击的诞生

针对`large bin`进行的攻击与上方介绍的任意地址读写颇有相似之处。

在GLibC对堆的管理概念中，程序申请的chunk不能通过`unsorted bin`链表匹配时，会将原来`bk`链表上的头成员`victim`取出，并将`victim->bk`更新为`bk`链表的头成员，被移出`unsorted bin`链表的`victim`，如果不能和申请大小`nb`完全匹配，那么`victim`进入`small bin`或`large bin`链表中。
