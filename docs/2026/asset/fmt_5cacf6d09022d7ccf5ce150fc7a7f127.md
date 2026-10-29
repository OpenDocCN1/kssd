
利用`size`字段，对任意`chunk`计算其结束位置，也就是`next chunk`的开始位置，是很容易的，如果只是为了划清各个`chunk`之间的界线，有`size`字段其实就够了，但是在释放过程中，如果当前释放`chunk`的`prev chunk`为`free chunk`，这时要是能知道`prev chunk`的大小，也是很有价值的，因为这样就可以很方便计算`prev chunk`的起始位置，进而跟当前释放`chunk`合并，相反，如果`prev chunk`为`inuse chunk`，即不需要与当前释放`chunk`合并，那也就不需要知道`prev chunk`的大小了，所以，`glibc`并不是始终为每个`chunk`安排一个`prev_size`字段，而是将`free chunk`的末尾4字节作为其`next chunk`的`prev_size`（一方面，既然是`free chunk`，那就表示业务层不会继续使用`user data`了，当然就可以被`glibc`内部使用；另一方面，`prev_size`相对于各个`chunk`头部的位置是确定的，向前偏移4字节就是）；

每个`chunk`的`size`都是`0x409`（最低3位清0为`0x408`，表示`chunk`总大小为1032字节，最低3位二进制值为`001`，表示`A=0`、`M=0`、`P=1`），是可以根据向`malloc()`传的大小推测的，在构造溢出数据时，尽量保持各个`size`的原值，另外，刚分配完时，所有`chunk`都是`inuse`状态，所以`prev_size`字段所占空间，都是用于`user data`，不管被溢出数据填充成什么内容，都不会影响`glibc`内部的执行逻辑。

## 攻击过程分析

`fread(ptr, 1024 * 1024, 1, stdin)`这行代码，使攻击者有机会往`0x804a008`之后的1M内存，写入任意数据，这块内存中，有很多地方保存的是`chunk->size`，由`glibc`内部使用，通过构造溢出数据，控制这些地方的值，就可以达到欺骗`glibc`的效果，甚至还可以进一步欺骗`glibc`，将部分`user data`也当作自己内部使用的内存，为此，作者构造出了上述内存布局图（右）中的数据，当漏洞程序执行`free(ptr2)`时，`glibc`就会按照攻击者欺骗的流程执行。

### (1) `chunk2->size = 0x40d`

最低3位清0为`0x408`，表示`chunk`总大小为1032字节，最低3位二进制值为`101`，表示`A=1`、`M=0`、`P=1`，这是要欺骗`glibc`认为`chunk2`在`thread arena`中（`chunk2`实际在`main arena`中），然后进一步欺骗`glibc`，使其认为`chunk2`所属`arena`的管理结构，在输入数据可以溢出到的某个位置，这样就相当于控制了`chunk2`所属`arena`的管理结构的内容了（而`main_arena`是个全局变量，位于数据段，溢出数据到达不了）。

```c
#define heap_for_ptr(ptr) \
  ((heap_info *)((unsigned long)(ptr) & ~(HEAP_MAX_SIZE-1)))
#define arena_for_chunk(ptr) \
  (chunk_non_main_arena(ptr) ? heap_for_ptr(ptr)->ar_ptr : &main_arena)
```

通过这段代码可以看出，`glibc`是将`chunk2`的地址按1M（`HEAP_MAX_SIZE`）向下对齐（即`0x8100000`），将其当作`chunk2`所属`heap`的地址，然后再将`heap->ar_ptr`指向的内存，作为`chunk2`所属`arena`的管理信息（这也是漏洞程序中用`for`循环分配内存，直到`ptr2`达到一定高度的原因，否则向下对齐为`0x8000000`，就不在可以溢出的范围了）。

### (2) `fake_heap->ar_ptr = 0x804a014`

由于`ar_ptr`为`heap_info`结构的第一个成员，所以按照布局图中的构造地址，`glibc`会认为`arena`管理结构位于`0x804a014`。

### (3) `fake_arena->bins[2] = DTORS_END-12`

根据布局图可以看出，`fake_arena`开始的8个字节，都被构造为`0x102`，剩余部分全部构造为`DTORS_END-12`，根据`malloc_state`结构的定义可知，这样构造肯定可以使`fake_arena->bins[2] = DTORS_END-12`。

```c
struct malloc_state {
  /* Serialize access. */
  mutex_t mutex;
  /* Should we have padding to move the mutex to its own cache line? */
#if THREAD_STATS
  /* Statistics for locking.  Only used if THREAD_STATS is defined. */
  long stat_lock_direct, stat_lock_loop, stat_lock_wait;
#endif
  /* The maximum chunk size to be eligible for fastbin */
  INTERNAL_SIZE_T  max_fast; /* low 2 bits used as flags */
  /* Fastbins */
  mfastbinptr      fastbins[NFASTBINS];
  /* Base of the topmost chunk - not otherwise kept in a bin */
  mchunkptr        top;
  /* The remainder from the most recent split of a small request */
  mchunkptr        last_remainder;
  /* Normal bins packed as described above */
  mchunkptr        bins[NBINS * 2];
  /* Bitmap of bins */
  unsigned int     binmap[BINMAPSIZE];
  /* Linked list */
  struct malloc_state *next;
  /* Memory allocated from the system in this arena. */
  INTERNAL_SIZE_T system_mem;
  INTERNAL_SIZE_T max_system_mem;
};
```

使`fake_arena->bins[2] = DTORS_END-12`，是为了欺骗`glibc`修改`got[.dtros]`：

```c
}
else
  clear_inuse_bit_at_offset(nextchunk, 0);

  /* Place the chunk in unsorted chunk list. Chunks are
     not placed into regular bins until after they have
     been given one chance to be used in malloc. */
  bck = unsorted_chunks(av); // 返回：&fake_arena->bins[0]
  fwd = bck->fd; // fd位于malloc_chunk结构体8字节偏移处
  // 所以fwd = bck->fd = fake_arena->bins[2] = DTORS_END-12
  p->bk = bck;
  p->fd = fwd;
  bck->fd = p; // fake_arena->bins[2] = p
  fwd->bk = p; // bk位于malloc_chunk结构体12字节偏移处
  // 所以这里会将p，写到DTORS_END指向的内存单元，即：got[.dtors] = p

  set_head(p, size | PREV_INUSE);
  set_foot(p, size);

  check_free_chunk(av, p);
}
```

关于`unsorted_chunks()`函数返回`&fake_arena->bins[0]`的设计意图，可以进入这篇帖子：[https://bbs.pediy.com/thread-271331.htm](https://bbs.pediy.com/thread-271331.htm)，看看其中的`bin`结构图。

### (4) `((struct malloc_chunk*)(DTORS_END-12))->bk = p`

在`(3)`中已经一起解释了，是为了使`got[.dtors] = p`，代码中的`p`，对应的是布局图中的`chunk2`，所以等到`.dtors()`函数执行时，实际上是执行`chunk2`位置的`"nop; nop; jmp 0x0c"`（`nop`机器码为`0x90`，`jmp 0x0c`机器码为`0xeb0c`，所以共4字节）。

### (5) `jmp 0x0c`

向前跳转`0x0c`偏移，是因为紧跟着的4个字节，用于存放`0x40d`，再往后的8字节，会被上述代码中的`p->bk = bck`和`p->fd = fwd`两条赋值语句覆盖，所以`shell code`一定要构造在`chunk2->bk`之后的位置。

### (6) `fake_arena = 0x804a014`

`0x804a014`也就是在`(2)`中，为`fake_heap->ar_ptr`构造的值，作者一开始是将`fake_arena`构造在`0x804a010`位置的，还特地在`0x804a010`位置构造了一个`0`，作为`fake_arena->mutex`值，但是他通过调试发现，漏洞程序在执行完`free(ptr)`后，会将`0x804a014`位置清0，这样，为`fake_arena->max_fast`构造的`0x102`（为了使判断2、判断5不成立），就被覆盖了，从而使后续的攻击逻辑执行失败。作者说被覆盖的原因，可能是由于`_int_free()`结束之后，执行`mutex_unlock()`导致的：

![图片](https://bbs.kanxue.com/upload/attach/202202/815036_K9S7GDHQSRFPHHG.png)

我认为不是这个原因，通过布局图可以看出，构造数据并没有修改`chunk->size`，所以`free(ptr)`就是按照正常的`glibc`逻辑执行的，那么`mutex_unlock()`修改的一定是`main_arena`全局变量中的`mutex`，而不可能是这里，一些其它版本的`glibc`中，`0x804a010`、`0x804a014`位置分别会是`fd_nextsize`、`bk_nextsize`，如果释放的`chunk`不是`large chunk`，释放函数中就会将这两个值清0，但是碰巧的是，`glibc-2.3.6`的`malloc_chunk`结构，没有`fd_nextsize`、`bk_nextsize`成员，所以如果感兴趣，可以仔细看看代码确定一下。

### (7) `fake_arena->max_fast = 0x102`

在`(6)`中已经说过了，`free(ptr)`结束后，会将`0x804a014`位置清0，所以正好可以作为`fake_arena->mutex`值，`0x804a018`位置的`0x102`，作为`fake_arena->max_fast`值。

### (8) 最后调用的`malloc()`

漏洞程序退出循环后，又调用了一次`malloc()`，这也是为了满足`"The House of Mind"`的利用条件（使判断10不成立）。

## 攻击可行性证明

上述已经将攻击流程介绍完毕，但是实际能否成功，还要看到底能不能将`glibc`欺骗到`"fwd->bk = p;"`这一行代码上。

### 判断1

```c
/*
 * p > -size，即：p > 0-size，即：p+size > 0，在p、size都大于0的前提下，表示p+size溢出了，但是由于p和size都是无符号数，p+size >= 0恒成立，所以写成p > -size
 * 个人感觉更严谨应该是：p >= -size，因为p+size == 0，也表示溢出了（比如size=0x00000001，则p=-size=0xffffffff（取反+1）时，p+size == 0也为溢出）
 */
if (__builtin_expect ((uintptr_t) p > (uintptr_t) -size, 0) || __builtin_expect ((uintptr_t) p & MALLOC_ALIGN_MASK, 0))
```

由于`0x40d`相比于`0x409`，只是设置了`NON_MAIN_ARENA`标志位，并没有将`chunk2`的大小修改为异常值，也没有修改`ptr2`的指向，所以这个判断不成立。

### 判断2

```c
if ((unsigned long)(size) <= (unsigned long)(av->max_fast)
#if TRIM_FASTBINS
    /* If TRIM_FASTBINS set, don't place chunks bordering top into fastbins */
    && (chunk_at_offset(p, size) != av->top)
#endif
)
```

根据构造数据可知，`chunk2->size = 0x40d`（最低3位为`PREV_INUSE`、`IS_MMAPPED`、`NON_MAIN_ARENA`标志位），`av`即为图中的`fake_arena`，而`fake_arena->max_fast = 0x102`（最低2位为`FASTCHUNKS_BIT`、`NONCONTIGUOUS_BIT`标志位），所以这个判断不成立。

### 判断3

```c
else if (!chunk_is_mmapped(p))
```

由于`chunk2->size = 0x40d`，`IS_MMAPPED`标志位为`0`，表示不是直接从`mmap`内存区域分配，而是从`main arena`或者`thread arena`中分配的，所以这个判断成立。

### 判断4

```c
if (__builtin_expect (p == av->top, 0))
```

由于攻击者欺骗了`glibc`，使其认为`chunk2`属于自己构造的`fake_arena`，而不再是`main_arena`，并且通过布局图可知，`chunk2 = 0x8100298`，`fake_arena->top = DTORS_END-12`，所以这个判断不成立。

### 判断5

```c
if (__builtin_expect (contiguous (av)
  && (char *) nextchunk >= ((char *) av->top + chunksize(av->top)), 0))
```

攻击者显然不希望这个判断成立，由于这2个判断条件之间是`&&`的关系，所以只要满足`contiguous(av) == 0`，即`fake_arena->max_fast`的`NONCONTIGUOUS_BIT`标志位为`0`，而根据布局图可知，`fake_arena->max_fast = 0x102`，可以保证这个判断不成立。

### 判断6

```c
if (__builtin_expect (!prev_inuse(nextchunk), 0))
```

`prev_inuse(nextchunk)`用于判断`chunk2`是否已经是`free chunk`了，如果是，那就是`double free`，不过由于这是第一次释放`chunk2`，所以这个判断不成立。

### 判断7
