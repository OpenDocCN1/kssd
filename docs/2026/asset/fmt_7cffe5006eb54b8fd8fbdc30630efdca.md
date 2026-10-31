
在`GLibC`的管理概念中，`fast bin`是极其特殊的一个，这种特殊体现在多个方面。

- 第一点就是存放的位置，它和`tcache`是唯二拥有单独存放区域的链表。

像`unsorted bin`、`small bin`、`large bin`这三类链表需要放在`bins`数组中进行存储，而`fast bin`链表则是可以独占一个`fastbinsY`数组。

更不用说独占一个全局变量`tcahce`的天龙人`tcache`链表了。

- 第二点体现在`fast chunk`在插入和取出的时机上。

不管是在申请的`_int_malloc`阶段，还是释放的`_int_free`，使用`fast bin`链表的优先级都是相当高的。

对于`GLibC`提供的`malloc`函数来讲，不管是释放还是申请内存时，它的优先级都是仅次于`tcache`的。

这一点正好体现了`fast bin`的设计原则，快速响应较小chunk的分配与释放。

```
__libc_malloc -> if (tc_idx < mp_.tcache_bins && tcache != NULL && tcache->counts[tc_idx] > 0) -> tcache_get -> _int_malloc -> if ((nb) <= (get_max_fast ())) -> ...... -> ......
_int_free -> if (tcache != NULL && tc_idx < mp_.tcache_bins) -> ...... -> if ((size) <= get_max_fast()) -> ......
```

- 第三点体现在`fast bin`链表的结构上。

像`unsorted bin`、`small bin`、`large bin`使用的是由`fd`和`bk`两个字段组成的双向循环链表。

其中`fd`根据chunk入链时间的倒序对链表进行维护，链表头是最晚入链的chunk，而字段`bk`则根据chunk入链时间的正序维护链表，链表头是最早入链的chunk。

像`fast bin`链表就只使用`fd`组成单向链表。

这么做的好处就是，程序每次获取到的都是最晚进入`fast bin`链表，能有更大的几率取出仍位于CPU缓存上的数据，优化程序的运行速度。

除了只维护`fd`的单向链表外，`fast bin`链表应该还是唯一的，会在存取数据上会针对多线程进行区分的链表。

```
_int_malloc -> if ((nb) <= (get_max_fast ())) -> if (SINGLE_THREAD_P) -> *fb = REVEAL_PTR (victim->fd); -> else -> REMOVE_FB (fb, pp, victim);
_int_free -> if ((size) <= get_max_fast()) -> if (SINGLE_THREAD_P) -> p->fd = PROTECT_PTR (&p->fd, old); -> *fb = p; -> else -> do { old2 = old; p->fd = PROTECT_PTR (&p->fd, old); } while ((old = catomic_compare_and_exchange_val_rel (fb, p, old2)) != old2);
```

从上面可以看到，`GLibC`会使用`SINGLE_THREAD_P`宏判断程序是否为多线程。

在单线程时会使用`REVEAL_PTR`直接将最晚入链的chunk移出链表，当chunk入链时也会直接操作链表。

但在多线程时，`GLibC`会使用`REMOVE_FB`将chunk移出链表，而且在chunk入链时会进入`do { ...... } while`循环语句中。

其实，`REMOVE_FB`宏也是通过`do { ...... } while`循环语句实现的。

但为什么在多线程中需要使用循环语句操作`fast bin`链表呢？

```
catomic_compare_and_exchange_val_acq(mem, newval, oldval) -> if (*mem == oldval) -> *mem = newval -> return oldval

#define REVEAL_PTR(ptr)  PROTECT_PTR (&ptr, ptr)
#define REMOVE_FB(fb, victim, pp)       \
do                                  \
{                                   \
  victim = pp;                    \
  if (victim == NULL)             \
    break;                      \
  pp = REVEAL_PTR (victim->fd);    \
}                                   \
while ((pp = catomic_compare_and_exchange_val_acq (fb, pp, victim)) != victim);
```

进出链表时使用的循环语句，会通过`catomic_compare_and_exchange_val_rel`宏推进循环，该宏的作用并不复杂，它接受三个参数，当`*mem`等于`oldval`时，就更新`*mem`，最后不管`*mem`是否等于`oldval`，都会返回`oldval`。

至于`atomic`，如果你熟悉多线程编程，就应该知道当多个线程访问同一对象时，为了确保线程间不冲突，会添加锁进行保护，就像一个人上厕所时要把门锁上，这样别人就进不来了，同样的，锁也会保证多线程有序的访问同一对象。

但是锁也会带来很多问题，比如常见的死锁，为了避免锁带来的问题，就产生了一个名为无锁编程的概念。无锁编程的基石是原子化`atomic`操作，原子操作指的是执行过程中不会被打断。

当线程使用无锁编程时，会利用原子操作访问对象，这时自然就不用担心其他线程和自己争抢访问对象了。

不过啊，所谓的原子的操作指的是一种轻量级的锁，不如在x86架构中，提供一种名为`lock`的指令，执行指令前会先通过`lock`加锁。

在了解`catomic_compare_and_exchange_val_rel`宏之后，我们可以知道上面的循环语句并不会进行多次循环，而且在当前的场景中，该宏一定会给`*mem`赋新值。

```
mchunkptr pp;
victim = *fb;
REMOVE_FB (fb, pp, victim);
```

显然，在这个时候`do { ...... } while`循环语句的秘密已经解开了，它就是在多线程环境下，针对链表进行的无锁编程。

- 最后一点体现在`fast chunk`对`PREV_INUSE`标志位的使用上。

`P`标志只会出现在`mchunk_size`和`mchunk_prev_size`中，在`malloc.c`代码中，可以经常看到`set_head(xx, xx | PREV_INUSE)`的语句。

是的，`set_head`宏是专门为操控`mchunk_size`而准备的。因为`set_head`经常会给`mchunk_size`添加`P`位。

`mchunk_size`上带着`PREV_INUSE`标志位是常态，它只会在非`fast chunk`类型且非`mmap`方式分配的chunk释放时，才会通过`clear_inuse_bit_at_offset`清空前一个`nextchunk`的`mchunk_size`的`P`位。

目的是告诉`nextchunk`，当`nextchunk`被释放时，它可以进行向后合并。

所以`mchunk_size`的`PREV_INUSE`标志位是“出厂时的默认设置”，`PREV_INUSE`标志位想要被清空，那就必须`prevchunk`被释放。

不过这种情况并不会涉及到`fast chunk`，`fast chunk`在释放时，并不会与其他的空闲chunk进行合并。

```
_int_free -> if ((size) <= get_max_fast ()) -> ...... -> else if (!chunk_is_mmapped(p)) -> _int_free_merge_chunk -> _int_free_create_chunk -> if (nextchunk != av->top) -> if (!nextinuse) -> ...... -> else -> clear_inuse_bit_at_offset(nextchunk, 0);
```

当然`fast chunk`也并不是绝对不合并，`GLibC`会通过`malloc_consolidate`接口c尝试合并`fast chunk`。

最常见的场景，就是当程序进入`_int_malloc`后，会检查`have_fastchunks`标记，如果发现arena中存在`fast chunk`就会开始合并。

```
_int_malloc -> if (atomic_load_relaxed (&av->have_fastchunks)) -> malloc_consolidate (av);

malloc_consolidate -> if (nextchunk != av->top) -> if (!nextinuse) -> ...... -> else -> clear_inuse_bit_at_offset(nextchunk, 0);
```

至于`mchunk_prev_size`的`P`位，甚至可以说它的所有标志位都是无效的，这是因为在`GLibC`中，`mchunk_prev_size`只在chunk被切割后，记录上一个chunk的大小。

```
#define set_foot(p, s)  (((mchunkptr) ((char *) (p) + (s)))->mchunk_prev_size = (s))

set_foot (remainder, remainder_size);
```

## 单向链表的隐患

`fast bin`只使用`fd`维护链表，好处就是写出了cache友好的代码，并且会让链表中空闲chunk间的相对关系不那么复杂。

但坏处就是，仅通过`fd`维护的成员，只会保证相邻chunk间具有链接关系，至于指向的内容是否正确就不得而知了。

而当使用`fd`和`bk`维护的双向链表时，不止可以保证相邻chunk间具有链接关系，而且还可以通过双向链表确认指向内容的正确性。

```
mchunkptr fd = p->fd;
mchunkptr bk = p->bk;
if (fd->bk != p || bk->fd != p)
  malloc_printerr ("corrupted double-linked list");
```

还有一点，就是不管是针对单向链表还是双向循环链表的检查，都是停留只检查两相邻的chunk间的关系，不会遍历整个链表进行检查。

## chunk重复释放问题

当程序将chunk释放后，按照道理来讲，程序就要将缓冲区变量的指针设置成`NULL`，如果不设置成空指针，就代表程序还有机会操作缓冲区变量。

首当其中的隐患就是UAF，由于程序还掌握着已释放chunk的内存地址，所以就还可以对已释放的chunk进行读写。

其次就是重复释放chunk了，未被置空的缓冲区变量可以被再次释放，但是chunk被重复释放会有什么安全隐患呢？

在了解重复释放的危害之前，我们先来探讨下`fast chunk`是怎么完成重复释放的。

### 重复释放的形成

重复释放也被称作是`Double Free`，即便你不熟悉二进制安全，也可能会经常听到过该漏洞的大名。

在没有任何安全检查的防范下，重复释放`fast chunk`不会收到任何阻力。

### 针对重复释放的加固

但是`Double Free`漏洞由来已久，`GLibC`也早就针对这块内容进行了加固，在版本较新的`GLibC`中，想要无痛的完成重复释放，还需要我们加深下对堆管理策略的理解。

在`GLibC`的`malloc.c`代码中，存在着多个针对`Double Free`漏洞的检查，这些检查主要集中在`_int_free`函数中。

显然，这是为chunk的释放而专门准备的。

```
#define contiguous(M)          (((M)->flags & NONCONTIGUOUS_BIT) == 0)

_int_free -> if ((size) <= get_max_fast ()) -> unsigned int idx = fastbin_index(size); -> fb = &fastbin (av, idx); -> mchunkptr old = *fb, old2; -> if (old == p) -> malloc_printerr ("double free or corruption (fasttop)"); -> else if (!chunk_is_mmapped(p)) -> _int_free_merge_chunk -> if (p == av->top) -> malloc_printerr ("double free or corruption (top)"); -> if (contiguous (av) && (char *) nextchunk >= ((char *) av->top + chunksize(av->top))) -> malloc_printerr ("double free or corruption (out)"); -> if (__glibc_unlikely (!prev_inuse(nextchunk))) -> malloc_printerr ("double free or corruption (!prev)");
```

`_int_free`在释放过程中，针对`Double Free`漏洞的检查分成两个部分。

- 第一部分就是针对`fast chunk`进行的检查，这个检查比较简单，主要操作就是获取`fast bin`链表中最晚入链的`old`，并将`old`与被释放的chunk地址进行比较，如果它们相同，`GLibC`就判定为重复释放，并抛出异常让程序终止运行。

至于`fast bin`链表中的其余空闲chunk是否和最新被释放的chunk重复，那就检查不到了，遍历整个链表的负担难免有些太重了。

- 第二部分是针对`unsorted chunk`、`small chunk`、`large chunk`的检查。

`_int_free_merge_chunk`函数中针对重复释放的检查可以分成三步。

a. 第一步是针对`top chunk`进行的，`top chunk`是一切chunk的源头，所有被使用的chunk最先都是从`top chunk`中分割出来的，因此它永远都是空闲的。

所以当`GLibC`发现被释放chunk的指针等同于`top chunk`时，就会判断为重复释放并通过`malloc_printerr`抛出异常。

你可能会好奇，`fast bin`链表就不用担心被释放的指针是`top chunk`吗？

对的！`fast bin`链表不用担心这个问题。

`fast bin`链表有一个天然的保护屏障，那就是`fast chunk`的大小限制。
