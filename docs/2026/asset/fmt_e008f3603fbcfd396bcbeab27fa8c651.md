
`0x804a014`也就是在(2)中，为`fake_heap->ar_ptr`构造的值，作者一开始是将`fake_arena`构造在`0x804a010`位置的，还特地在`0x804a010`位置构造了一个`0`，作为`fake_arena->mutex`值，但是他通过调试发现，漏洞程序在执行完`free(ptr)`后，会将`0x804a014`位置清0，这样，为`fake_arena->max_fast`构造的`0x102`（为了使判断2、判断5不成立），就被覆盖了，从而使后续的攻击逻辑执行失败。作者说被覆盖的原因，可能是由于`_int_free()`结束之后，执行`mutex_unlock()`导致的：

![](upload/attach/202202/815036_K9S7GDHQSRFPHHG.png)

我认为不是这个原因，通过布局图可以看出，构造数据并没有修改`chunk->size`，所以`free(ptr)`就是按照正常的glibc逻辑执行的，那么`mutex_unlock()`修改的一定是`main_arena`全局全量中的`mutex`，而不可能是这里，一些其它版本的glibc中，`0x804a010`、`0x804a014`位置分别会是`fd_nextsize`、`bk_nextsize`，如果释放的chunk不是large chunk，释放函数中就会将这两个值清0，但是碰巧的是，glibc-2.3.6的`malloc_chunk`结构，没`fd_nextsize`、`bk_nextsize`成员，所以如果感兴趣，可以仔细看看代码确定一下。

* (7) `fake_arena->max_fast = 0x102`

  在(6)中已经说过了，`free(ptr)`结束后，会将`0x804a014`位置清0，所以正好可以作为`fake_arena->mutex`值，`0x804a018`位置的`0x102`，作为`fake_arena->max_fast`值。

* (8) 最后调用的`malloc()`

  漏洞程序退出循环后，又调用了一次`malloc()`，这也是为了满足"The House of Mind"的利用条件（使判断10不成立）。

## 攻击可行性证明

上述已经将攻击流程介绍完毕，但是实际能否成功，还要看到底能不能将glibc欺骗到`fwd->bk = p;`这一行代码上。

### 判断1

```c
/*
 * p > -size， 即：p > 0-size，即：p+size > 0，在p、size都大于0的前提下，表示p+size溢出了，但是由于p和size都是无符号数，p+size >= 0恒成立，所以写成p > -size
 * 个人感觉更严谨应该是：p >= -size，因为p+size == 0，也表示溢出了（比如size=0x00000001，则p=-size=0xffffffff（取反+1）时，p+size==0也为溢出）
 */
if (__builtin_expect ((uintptr_t) p > (uintptr_t) -size, 0)
|| __builtin_expect ((uintptr_t) p & MALLOC_ALIGN_MASK, 0))
```

由于`0x40d`相比于`0x409`，只是设置了`NON_MAIN_ARENA`标志位，并没有将`chunk2`的大小修改为异常值，也没有修改`ptr2`的指向，所以这个判断不成立。

### 判断2

```c
if ((unsigned long)(size) <= (unsigned long)(av->max_fast)
#if TRIM_FASTBINS
/*
If TRIM_FASTBINS set, don't place chunks
bordering top into fastbins
*/
&& (chunk_at_offset(p, size) != av->top)
#endif
)
```

根据构造数据可知，`chunk2->size = 0x40d`（最低3位为`PREV_INUSE`、`IS_MMAPPED`、`NON_MAIN_ARENA`标志位），`av`即为图中的`fake_arena`，而`fake_arena->max_fast = 0x102`（最低2位为`FASTCHUNKS_BIT`、`NONCONTIGUOUS_BIT`标志位），所以这个判断不成立。

### 判断3

```c
else if (!chunk_is_mmapped(p))
```

由于`chunk2->size = 0x40d`，`IS_MMAPPED`标志位为0，表示不是直接从`mmap`内存区域分配，而是从main arena或者thread arena中分配的，所以这个判断成立。

### 判断4

```c
if (__builtin_expect (p == av->top, 0))
```

由于攻击者欺骗了glibc，使其认为`chunk2`属于自己构造的`fake_arena`，而不再是`main_arena`，并且通过布局图可知，`chunk2 = 0x8100298`，`fake_arena->top = DTORS_END-12`，所以，这个判断不成立。

### 判断5

```c
if (__builtin_expect (contiguous (av)
&& (char *) nextchunk
>= ((char *) av->top + chunksize(av->top)), 0))
```

攻击者显然不希望这个判断成立，由于这2个判断条件之间是`&&`的关系，所以只要满足`contiguous(av) == 0`，即`fake_arena->max_fast`的`NONCONTIGUOUS_BIT`标志位为0，而根据布局图可知，`fake_arena->max_fast = 0x102`，可以保证这个判断不成立。

### 判断6

```c
if (__builtin_expect (!prev_inuse(nextchunk), 0))
```

`prev_inuse(nextchunk)`用于判断`chunk2`是否已经是free chunk了，如果是，那就是double free，不过由于这是第一次释放`chunk2`，所以，这个判断不成立。

### 判断7

```c
if (__builtin_expect (nextchunk->size <= 2 * SIZE_SZ, 0)
|| __builtin_expect (nextsize >= av->system_mem, 0))
```

作者使用的shell code很短，不会将`chunk2->nextchunk->size`覆盖为异常值，即使shell code很长，也可以通过`jmp`跳过`nextchunk->size`字段，另外`av->system_mem`，在攻击者可以构造的范围，所以，可以保证这个判断不成立。

### 判断8

```c
if (!prev_inuse(p))
```

`chunk2->size = 0x40d`，`PREV_INUSE`标志位为1，显然不会通过这个判断，将`chunk2`与其前一个chunk合并。

### 判断9

```c
if (nextchunk != av->top)
```

和控制判断4的道理一样，由于`av->top`的值，是受攻击者控制的，所以相应也很容易控制这个判断，使其成立。

### 判断10

```c
if (!nextinuse)
```

`chunk2->nextchunk`，也就是漏洞程序中最后一次执行`malloc()`分配的chunk，它这时显然还没有释放，所以这个判断不成立，从而最终欺骗glibc执行到`else`分支中的代码（如果漏洞程序中没有最后一次`malloc()`，应该也能攻击成功，因为那样的话，`chunk2`后面就是top chunk，而top chunk一定是inuse状态的，这是通过它顶部的fencepost标记的）。

## glibc改造

随着各种攻击技术的出现，glibc其实一直都在改造，以上看到的这些判断，很多就是为了缓解攻击，但glibc的改造，是受限于两个因素的：

- 不能影响正常逻辑

- 不能影响性能

  比如业务层调用`free()`，glibc会将释放chunk放在相应的缓存链表中，而判断是否double free，只会拿当前释放chunk和链表头中的第一个chunk，进行地址对比，而不会遍历整个链表对比。

所以，glibc只能缓存攻击，根本避免被攻击，在业务层的源头就要开始防范。

## "The House of Mind"的学习条件

其实不光是"The House of Mind"，在学习各种堆溢出漏洞的利用方法之前，都必须对glibc `malloc()`/`free()`的逻辑，有相当程度的了解，《Glibc内存管理--Ptmalloc2源代码分析》这份文档，通过129页的篇幅，已经分析的非常深刻和详细（如果没有积分下载文档，也可以去看作者的博客：[f24K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2A6N6r3g2&6k6g2)9J5k6h3y4G2L8g2)9J5c8X3u0D9L8$3N6Q4x3V1k6#2M7$3g2J5i4K6u0r3L8i4q4*7K9s2g2S2L8X3M7.](<https://bbs.kanxue.com/elink@897K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2A6N6r3g2&6k6g2)9J5k6h3y4G2L8g2)9J5c8X3u0D9L8$3N6Q4x3V1k6#2M7$3g2J5i4K6u0r3L8i4q4*7K9s2g2S2L8X3M7.>)），也可以看看我发过的一个帖子：<https://bbs.pediy.com/thread-271331.htm>，先从外围了解glibc `malloc()`/`free()`的本质和设计目标，瞄一眼宏观的地形，再深入到茫茫的内部实现中，应该可以少迷点路。

另外，本文是对phrack杂志中一篇神作的总结和补充，所以exploit code和更完整的分析过程，请阅读原文：[085K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4m8Z5M7X3q4U0K9#2)9J5k6h3!0J5k6#2)9J5c8X3W2K6M7%4g2W2M7#2)9J5c8U0j5$3i4K6u0r3x3e0m8Q4x3X3g2Z5N6r3#2D9](<https://bbs.kanxue.com/elink@7f9K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4m8Z5M7X3q4U0K9#2)9J5k6h3!0J5k6#2)9J5c8X3W2K6M7%4g2W2M7#2)9J5c8U0j5$3i4K6u0r3x3e0m8Q4x3X3g2Z5N6r3#2D9>)。

## 什么是"The House of Mind"？

"The House of Mind"是一种堆溢出漏洞的利用方法（为什么叫这个名称我目前还不知道），可以通过构造输入数据，让漏洞程序执行攻击者期望的任意代码（不过，不是所有存在堆溢出漏洞的程序，都可以利用这种方法进行攻击，需要漏洞程序满足一定条件，稍后具体说明）。

再具体一点就是，在应用程序分配到的内存周围，很多都是glibc内部使用的内存，程序存在漏洞，攻击者就有机会通过构造输入数据，溢出glibc内部使用的变量，进一步控制`malloc()`/`free()`的执行逻辑，最终借glibc之手，修改某个函数对应的got表项（可以理解为函数指针，感兴趣也可以看看我的另外一个帖子：<https://bbs.pediy.com/thread-246373.htm>），使其指向一段shell code（同样通过用户输入构造），这样，当漏洞程序后续执行该函数时（比如`.dtors()`函数，它会在`main()`函数结束后执行），就会触发shell code执行。

- "The House of Mind"的目标，就是欺骗`_int_free()`按如下逻辑执行：

  ![](https://bbs.kanxue.com/upload/attach/202202/815036_SPYN9B7TZ23EA4G.png)

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

- 按照程序逻辑，归纳程序的执行流程如下：

1. `ptr = malloc(1024);`

   `heap = (int)ptr & 0xFFF00000; // 将ptr值按1M向下取整`

2. 循环执行`ptr2 = malloc(1024)`，直到`(ptr2 & 0xFFF00000) == (heap + 0x100000)`

3. 再次执行一次`malloc(1024)`

4. `fread(ptr, 1024 * 1024, 1, stdin);`

   使用`fead()`函数读取用户输入，是为了减小攻击难度，如果换成`strcpy()`函数，读到`'\0'`字符就不会再读了，解决这个问题，要使用的就是另外的技术了，作者为了让大家专注于"The House of Mind"，就特地避开了更复杂的情况。

5. `free(ptr);`

   `free(ptr2);`

- 稍后就会明白为什么这样才能满足"The House of Mind"的利用条件，先看漏洞程序的内存布局（左）：

  ![](https://bbs.kanxue.com/upload/attach/202202/815036_CZMZVB5HSNNPHG6.png)

  - 由于glibc在每个chunk头部，都额外安排了4字节的`size`字段记录其大小，并将整个chunk的大小按8字节对齐，所以每次`malloc(1024)`实际消耗的是1032字节（`(1024+4)`按8对齐），因此在循环中，`ptr2`是按1032字节递增的，这样，假设实际运行时`ptr = 0x804a008`（如果`/proc/sys/kernel/randomize_va_space`文件内容非0，`ptr`值会是随机的，即使随机化是关闭的，也跟漏洞程序代码段、数据段的长度有关，可以认为攻击者无法精确预测这个值，不过它也不会影响能否攻击成功，定个假设值只是为了后续描述方便），循环第723次时，`ptr2 = 0x81002a0`，跳出循环。

  - 蓝色区域对应第一次`malloc(1024)`占用的内存，灰色区域对应循环中前721次`malloc(1024)`占用的内存，白色区域对应循环第722次`malloc(1024)`占用的内存，橙色区域对应循环第723次`malloc(1024)`占用的内存；
