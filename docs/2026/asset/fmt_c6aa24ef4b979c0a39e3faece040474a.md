
# The House of Mind

## "The House of Mind"的学习条件

其实不光是"The House of Mind"，在学习各种堆溢出漏洞的利用方法之前，都必须对`glibc malloc()/free()`的逻辑，有相当程度的了解，《Glibc内存管理--Ptmalloc2源代码分析》这份文档，通过129页的篇幅，已经分析的非常深刻和详细（如果没有积分下载文档，也可以去看作者的博客：`e90K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2A6N6r3g2&6k6g2)9J5k6h3y4G2L8g2)9J5c8X3u0D9L8$3N6Q4x3V1k6#2M7$3g2J5i4K6u0r3L8i4q4*7K9s2g2S2L8X3M7`.），也可以看看我发过的一个帖子：[https://bbs.pediy.com/thread-271331.htm](https://bbs.pediy.com/thread-271331.htm)，先从外围了解`glibc malloc()/free()`的本质和设计目标，瞄一眼宏观的地形，再深入到茫茫的内部实现中，应该可以少迷点路。

另外，本文是对phrack杂志中一篇神作的总结和补充，所以exploit code和更完整的分析过程，请阅读原文：`290K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4m8Z5M7X3q4U0K9#2)9J5k6h3!0J5k6#2)9J5c8X3W2K6M7%4g2W2M7#2)9J5c8U0j5$3i4K6u0r3x3e0m8Q4x3X3g2Z5N6r3#2D9`。

## 什么是"The House of Mind"？

"The House of Mind"是一种堆溢出漏洞的利用方法（为什么叫这个名称我目前还不知道），可以通过构造输入数据，让漏洞程序执行攻击者期望的任意代码（不过，不是所有存在堆溢出漏洞的程序，都可以利用这种方法进行攻击，需要漏洞程序满足一定条件，稍后具体说明）。

再具体一点就是，在应用程序分配到的内存周围，很多都是glibc内部使用的内存，程序存在漏洞，攻击者就有机会通过构造输入数据，溢出glibc内部使用的变量，进一步控制`malloc()/free()`的执行逻辑，最终借glibc之手，修改某个函数对应的`got`表项（可以理解为函数指针，感兴趣也可以看看我的另外一个帖子：[https://bbs.pediy.com/thread-246373.htm](https://bbs.pediy.com/thread-246373.htm)），使其指向一段shell code（同样通过用户输入构造），这样，当漏洞程序后续执行该函数时（比如`.dtors()`函数，它会在`main()`函数结束后执行），就会触发shell code执行。

* "The House of Mind"的目标，就是欺骗`_int_free()`按如下逻辑执行：

  ![](upload/attach/202202/815036_SPYN9B7TZ23EA4G.png)

## 漏洞程序

为了满足"The House of Mind"的利用条件，作者提供了一个用于演示的漏洞程序（现实中这类漏洞当然会隐蔽的多，几乎不会存在这么饥渴难耐的想被宰割的程序）。

```c
/*
 * K-sPecial's vulnerable program
 */
#include <stdio.h>
#include <stdlib.h>
int main (void) {
char *ptr = malloc(1024); /* First allocated chunk */
char *ptr2; /* Second chunk */
/* ptr & ~(HEAP_MAX_SIZE-1) = 0x08000000 */
int heap = (int)ptr & 0xFFF00000;
_Bool found = 0;
printf("ptr found at %p\n", ptr); /* Print address of first chunk */
// i == 2 because this is my second chunk to allocate
for (int i = 2; i < 1024; i++) {
/* Allocate chunks up to 0x08100000 */
if (!found && (((int)(ptr2 = malloc(1024)) & 0xFFF00000) == \
(heap + 0x100000))) {
printf("good heap allignment found on malloc() %i (%p)\n", i, ptr2);
found = 1; /* Go out */
break;
}
}
malloc(1024); /* Request another chunk: (ptr2 != av->top) */
/* Incorrect input: 1048576 bytes */
fread (ptr, 1024 * 1024, 1, stdin);
free(ptr); /* Free first chunk */
free(ptr2); /* The House of Mind */
return(0); /* Bye */
}
```

* 按照程序逻辑，归纳程序的执行流程如下：

1. `ptr = malloc(1024);`

   `heap = (int)ptr & 0xFFF00000;` // 将`ptr`值按1M向下取整

2. 循环执行`ptr2 = malloc(1024)`，直到`(ptr2 & 0xFFF00000) == (heap + 0x100000)`

3. 再次执行一次`malloc(1024)`

4. `fread(ptr, 1024 * 1024, 1, stdin);`

   使用`fread()`函数读取用户输入，是为了减小攻击难度，如果换成`strcpy()`函数，读到`'\0'`字符就不会再读了，解决这个问题，要使用的就是另外的技术了，作者为了让大家专注于"The House of Mind"，就特地避开了更复杂的情况。

5. `free(ptr);`

   `free(ptr2);`

* 稍后就会明白为什么这样才能满足"The House of Mind"的利用条件，先看漏洞程序的内存布局（左）：

  ![](upload/attach/202202/815036_CZMZVB5HSNNPHG6.png)

* 由于glibc在每个`chunk`头部，都额外安排了4字节的`size`字段记录其大小，并将整个`chunk`的大小按8字节对齐，所以每次`malloc(1024)`实际消耗的是1032字节（`(1024+4)`按8对齐），因此在循环中，`ptr2`是按1032字节递增的，这样，假设实际运行时`ptr=0x804a008`（如果`/proc/sys/kernel/randomize_va_space`文件内容非0，`ptr`值会是随机的，即使随机化是关闭的，也跟漏洞程序代码段、数据段的长度有关，可以认为攻击者无法精确预测这个值，不过它也不会影响能否攻击成功，定个假设值只是为了后续描述方便），循环第723次时，`ptr2=0x81002a0`，跳出循环。

* 蓝色区域对应第一次`malloc(1024)`占用的内存，灰色区域对应循环中前721次`malloc(1024)`占用的内存，白色区域对应循环第722次`malloc(1024)`占用的内存，橙色区域对应循环第723次`malloc(1024)`占用的内存；

* 利用`size`字段，对任意`chunk`计算其结束位置，也就是next `chunk`的开始位置，是很容易的，如果只是为了划清各个`chunk`之间的界线，有`size`字段其实就够了，但是在释放过程中，如果当前释放`chunk`的prev `chunk`为free `chunk`，这时要是能知道prev `chunk`的大小，也是很有价值的，因为这样就可以很方便计算prev `chunk`的起始位置，进而跟当前释放`chunk`合并，相反，如果prev `chunk`为inuse `chunk`，即不需要与当前释放`chunk`合并，那也就不需要知道prev `chunk`的大小了，所以，glibc并不是始终为每个`chunk`安排一个`prev_size`字段，而是将free `chunk`的末尾4字节作为其next `chunk`的`prev_size`（一方面，既然是free `chunk`，那就表示业务层不会继续使用user data了，当然就可以被glibc内部使用；另一方面，`prev_size`相对于各个`chunk`头部的位置是确定的，向前偏移4字节就是）；

* 每个`chunk`的`size`都是`0x409`（最低3位清0为`0x408`，表示`chunk`总大小为1032字节，最低3位二进制值为001，表示A=0、M=0、P=1），是可以根据向`malloc()`传的大小推测的，在构造溢出数据时，尽量保持各个`size`的原值，另外，刚分配完时，所有`chunk`都是inuse状态，所以`prev_size`字段所占空间，都是用于user data，不管被溢出数据填充成什么内容，都不会影响glibc内部的执行逻辑。

## 攻击过程分析

`fread(ptr, 1024 * 1024, 1, stdin)`这行代码，使攻击者有机会往`0x804a008`之后的1M内存，写入任意数据，这块内存中，有很多地方保存的是`chunk->size`，由glibc内部使用，通过构造溢出数据，控制这些地方的值，就可以达到欺骗glibc的效果，甚至还可以进一步欺骗glibc，将部分user data也当作自己内部使用的内存，为此，作者构造出了上述内存布局图（右）中的数据，当漏洞程序执行`free(ptr2)`时，glibc就会按照攻击者欺骗的流程执行。

* (1) `chunk2->size = 0x40d`

  最低3位清0为`0x408`，表示`chunk`总大小为1032字节，最低3位二进制值为101，表示A=1、M=0、P=1，这是要欺骗glibc认为`chunk2`在thread arena中（`chunk2`实际在main arena中），然后进一步欺骗glibc，使其认为`chunk2`所属arena的管理结构，在输入数据可以溢出到的某个位置，这样就相当于控制了`chunk2`所属arena的管理结构的内容了（而`main_arena`是个全局变量，位于数据段，溢出数据到达不了）。

  ```c
  #define heap_for_ptr(ptr) \
  ((heap_info *)((unsigned long)(ptr) & ~(HEAP_MAX_SIZE-1)))
  #define arena_for_chunk(ptr) \
  (chunk_non_main_arena(ptr) ? heap_for_ptr(ptr)->ar_ptr : &main_arena)
  ```

  通过这段代码可以看出，glibc是将`chunk2`的地址按1M（`HEAP_MAX_SIZE`）向下对齐（即`0x8100000`），将其当作`chunk2`所属heap的地址，然后再将`heap->ar_ptr`指向的内存，作为`chunk2`所属arena的管理信息（这也是漏洞程序中用for循环分配内存，直到`ptr2`达到一定高度的原因，否则向下对齐为`0x8000000`，就不在可以溢出的范围了）。

* (2) `fake_heap->ar_ptr = 0x804a014`

  由于`ar_ptr`为`heap_info`结构的第一个成员，所以按照布局图中的构造地址，glibc会认为arena管理结构位于`0x804a014`。

* (3) `fake_arena->bins[2] = DTORS_END-12`

  根据布局图可以看出，`fake_arena`开始的8个字节，都被构造为`0x102`，剩余部分全部构造为`DTORS_END-12`，根据`malloc_state`结构的定义可知，这样构造肯定可以使`fake_arena->bins[2] = DTORS_END-12`。

  ```c
  struct malloc_state {
  /* Serialize access. */
  mutex_t mutex;
  // Should we have padding to move the mutex to its own cache line?
  #if THREAD_STATS
  /* Statistics for locking. Only used if THREAD_STATS is defined. */
  long stat_lock_direct, stat_lock_loop, stat_lock_wait;
  #endif
  /* The maximum chunk size to be eligible for fastbin */
  INTERNAL_SIZE_T max_fast; /* low 2 bits used as flags */
  /* Fastbins */
  mfastbinptr fastbins[NFASTBINS];
  /* Base of the topmost chunk -- not otherwise kept in a bin */
  mchunkptr top;
  /* The remainder from the most recent split of a small request */
  mchunkptr last_remainder;
  /* Normal bins packed as described above */
  mchunkptr bins[NBINS * 2];
  /* Bitmap of bins */
  unsigned int binmap[BINMAPSIZE];
  /* Linked list */
  struct malloc_state *next;
  /* Memory allocated from the system in this arena. */
  INTERNAL_SIZE_T system_mem;
  INTERNAL_SIZE_T max_system_mem;
  };
  ```

  使`fake_arena->bins[2] = DTORS_END-12`，是为了欺骗glibc修改`got[.dtros]`：

  ```c
  } else
  clear_inuse_bit_at_offset(nextchunk, 0);
  /*
  Place the chunk in unsorted chunk list. Chunks are
  not placed into regular bins until after they have
  been given one chance to be used in malloc.
  */
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

  关于`unsorted_chunks()`函数返回`&fake_arena->bins[0]`的设计意图，可以进入这篇帖子：[https://bbs.pediy.com/thread-271331.htm](https://bbs.pediy.com/thread-271331.htm)，看看其中的bin结构图。

* (4) `((struct malloc_chunk*)(DTORS_END-12))->bk = p`

  在(3)中已经一起解释了，是为了使`got[.dtors] = p`，代码中的`p`，对应的是布局图中的`chunk2`，所以等到`.dtors()`函数执行时，实际上是执行`chunk2`位置的`"nop; nop; jmp 0x0c"`（nop机器码为`0x90`，jmp 0x0c机器码为`0xeb0c`，所以共4字节）。

* (5) `jmp 0x0c`

  向前跳转`0x0c`偏移，是因为紧接着的4个字节，用于存放`0x40d`，再往后的8字节，会被上述代码中的`p->bk=bck`和`p->fd=fwd`两条赋值语句覆盖，所以shell code一定要构造在`chunk2->bk`之后的位置。

* (6) `fake_arena = 0x804a014`
