
```
/* With rounding and alignment, the bins are...

   idx 0   bytes 0..24 (64-bit) or 0..12 (32-bit)

   idx 1   bytes 25..40 or 13..20

   idx 2   bytes 41..56 or 21..28

   etc.  */
```

由于`tcache`的增加和删除非常简洁，因此速度很快，但另一方面这也意味着缺乏各种安全检查和`mitigation`，在利用时候也格外方便。

## 分配与释放

在分析分配和释放的具体流程上，最好的方法其实是阅读源码(`RTFSC`)，也有的人喜欢看一步步的文字解释。我这里按照自己阅读源码的过程将其简化为伪代码，或许可以更直观一些。

> 再次声明，所分析的`libc`版本为`2.27`

### `_int_malloc`

该函数代码近千行，梳理主要逻辑的伪代码如下：

```
void *_int_malloc(size) {

    size_t nb = req2size(size);

    chunkptr victim;

    if (nb <= get_max_fast()) {

        // 定位对应的fastbin，并在头部取出(FILO)

        // 如果开启tcache，还会将bin中其他相同大小的chunk全部转移到tcache中

    }

    if (in_smallbin_range(size)) {

        // 定位对应的smallbin，并在尾部取出，即bin->bk(FIFO)

        // 如果开启tcache，还会将bin中其他相同大小的chunk全部转移到tcache中

    } else {

        // 请求为large request，因此需要先清理fastbin

        malloc_consolidate(av);

    }

    // 在unsorted_bin中搜索

    foreach(chunk : unsorted_bin) {

        if (is_small && only_chunk && chunk_size > nb) {

            // 分裂这个chunk，更新last_remainder并返回

        }

        if (chunk_size == nb) {

            // 如果没开启tcache，直接返回

            // 如果开启tcache，则先放到tcache，再次循环从tcache中取

        } 

        // 只有一次机会，不是合适的chunk，直接释放到对应的bin中

        if (in_smallbin_range(size)) {

            // 放到对应的smallbin 头部

        } else {

            // 放到对应的largbin中，按照大小排序

        }

        if (in_tcache) return tcache_get();

    }

    if (!in_smallbin_range (nb)) {

        // 在largebin中搜索，找到就返回

    }

    for (bin in bins) {

        // 通过bitmap在smallbin中搜索，找到就返回

    }

use_top:

    // 都找不到合适的chunk，中top_chunk中分裂

    // 如果不足以分离，通过sysalloc扩充top_chunk

}
```

值得一提的是`tcache`的搜索在`_int_malloc`之前，优先级最高：

```
void *__libc_malloc(size) {

    if (in_tcache) return tcache_get();

    return _int_malloc(size);

}
```

### `_int_free`

释放逻辑的伪代码如下：

```
static void
_int_free (mstate av, mchunkptr p) {

    size_t size = chunksize (p);

    if (tcache) { 

        // 放到tcachebin中

    }

    if (size <= get_max_fast()) {

        // 将chunk放到对应fastbin的头部

    } else if (!chunk_is_mmapped(p)) {

        if (!prev_inuse(p)) {

            // 后向合并(backward)

            p = prev_chunk(p);

            unlink(av, p, bck, fwd);

        }

        if (nextchunk != av->top) {

            if (!nextinuse) {

                // 前向合并(forward)

                unlink(av, nextchunk, bck, fwd);

                size += nextsize;

            }

            // 将chunk放到unsorted bin中

        } else {

            // 将chunk与top_chunk合并，并成为新的top_chuck

            av->top = p;

        }

        if (size > FASTBIN_CONSOLIDATION_THRESHOLD) {

            // 如果释放的空间太大，就通过systrim回收给系统

            malloc_consolidate(av);

            systrim(heap);

        }

    } else {

        munmap_chunk(p);

    }

}
```

`free`中这些操作大都是围绕两个目的：

1. 减少内部碎片
2. 加快下次分配的速度

记住这两点，那么其中的许多操作也就比较容易理解了。

另外由于篇幅原因，分析中省略了各种安全相关的`assert`，因为这和具体的`libc`版本有关，实际利用布局时需要根据具体的代码进行适配。

# 漏洞利用

漏洞利用的核心是在产生非预期的操作时，通过内存布局实现读写或者控制流劫持。堆漏洞利用也一样，我们的目标是通过小小的漏洞去构造出蝴蝶效应，在代码和数据的有序交互中开辟出一条新的道路。

`glibc`的漏洞利用已经有很多优秀的资料了，比如：

- [heap-exploitation gitbook](https://bbs.kanxue.com/elink@26eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Z5k6h3q4H3i4K6u0V1k6i4S2H3L8r3!0A6N6r3q4@1K9h3!0F1i4K6u0W2k6r3S2S2N6X3q4D9K9$3q4H3K9h3I4Q4x3X3g2U0L8$3#2Q4x3V1j5`.)

- [ctf-wiki/glibc-heap](https://bbs.kanxue.com/elink@870K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0N6r3k6Q4x3X3c8%4K9h3E0A6i4K6u0W2k6$3W2@1K9s2g2T1i4K6u0W2K9h3!0Q4x3V1k6U0N6r3k6Q4x3X3c8%4K9h3E0A6i4K6u0r3M7s2N6F1i4K6u0r3L8r3W2F1N6i4S2Q4x3V1k6Y4L8r3W2T1j5#2)9J5k6r3S2W2j5i4l9`.)

- [heap-exploitation by angleboy](https://bbs.kanxue.com/elink@e27K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3q4F1k6$3g2D9j5X3!0&6i4K6u0W2L8r3!0Y4k6r3!0%4L8W2)9J5k6h3y4G2L8g2)9J5c8Y4m8G2M7%4c8K6i4K6u0r3x3U0V1I4z5e0R3K6i4K6u0V1K9r3g2S2M7q4)9J5k6r3g2^5M7r3I4G2K9i4c8S2N6r3W2G2L8R3`.`.)

- [how2heap by shellphish](https://bbs.kanxue.com/elink@b1fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K6K9r3g2D9L8s2m8Z5K9i4y4Z5i4K6u0r3K9r3!0%4x3X3S2W2j5i4l9`.)

- [Heap overflow using Malloc Maleficarum](https://bbs.kanxue.com/elink@0f0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6M7r3I4G2K9i4c8X3N6h3&6Q4x3X3g2%4L8%4u0V1M7s2u0W2M7%4y4Q4x3X3g2U0L8$3#2Q4x3V1j5J5x3o6p5#2i4K6u0r3x3o6y4Q4x3V1j5H3y4q4)9J5c8X3S2W2j5i4m8Q4x3X3c8G2N6X3g2J5k6X3I4G2N6#2)9J5k6s2g2K6K9h3&6Y4i4K6u0V1L8h3q4D9L8r3!0U0i4K6u0V1L8h3q4D9k6h3k6A6j5$3q4J5N6h3#2Q4x3V1j5`.)

- [Extra Heap Exploitation by dangokyo](https://bbs.kanxue.com/elink@602K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1j5h3&6Y4L8$3E0&6L8#2)9J5k6h3#2W2i4K6u0r3M7r3!0K6N6q4)9J5k6r3I4A6M7%4c8Q4x3V1j5`.)

- ...

因此我们主要介绍几个典型，而不深入各个House的细节之中。

## unlink primitive

`Binning`是基于双链表而构造的，而在双链表中一个典型操作就是删除某个结点，也就是`unlink`操作。以`fd`和`bk`作为链表指针为例，`unlink`的实现一般是：

```
void unlink(chunk *c) {

    FD = c->fd;

    BK = c->bk;

    FD->bk = BK;

    BK->fd = FD;

}
```

本质是让前后的结点绕过自己而进行连结。这个操作本身没有问题，但是却经常被攻击者滥用。

假设`fd`和`bk`的偏移分别为`off_fd`和`off_bk`，那么上述操作转换为内存读写是：

```
FD = *(c + fd_off);

BK = *(c + bk_off);

*(FD + bk_off) = BK; // 3

*(BK + fd_off) = FD; // 4
```

如果攻击者可以控制被`unlink`的`chunk`中的内容，即`FD`、`BK`的值是可控的话，那么就可以构造`FD`为目的地址`x-bk_off`，且`BK`为攻击者控制的地址`y`。则第3步就可以简化成：

```
*x = y;
```

如果`x`是某个已知会被调用的函数指针地址，那就可以将其覆盖为`BK`，令`BK`指向我们的`shellcode`即可实现控制流劫持。当然这时候`BK+fd_off`处会被修改，实际构造`shellcode`可以跳过这几个字节。

其实除了`ptmalloc`，很多代码中只要有双向链表都会有类似的`unlink`操作，比如之前安卓内核中的的`CVE-2019-2215`，所以这类常见的`pattern`还是值得深入研究的。

## 单字节溢出

堆的单字节溢出在现实中通常更为常见。涉及到堆越界的基本上都和堆的数据结构(头部内容)强相关，所以这里着重讨论`gblic`的情况。

在可以溢出1字节的情况下，我们可以修改的区域为相邻的下一个`chunk`的`size`字段，因为`prev_size`实际上是在当前`chunk`的`data`部分的。溢出的思路有以下几种：

**一、 拓展被释放块**：当溢出的下一个`chunk`已经被释放并且存放在`unsorted bin`中时，通修改一字节将其扩大，这样下一次分配该`chunk`时后面的块就会被包含从而造成进一步溢出。

```
  0x100   0x100    0x80

|-------|-------|-------|

|   A   |   B   |   C   |   初始状态

|-------|-------|-------|

|   A   |   B   |   C   |   释放 B

|-------|-------|-------|

|   A   |   B   |   C   |   溢出 B 的 size 为 0x180

|-------|-------|-------|

|   A   |   B   |   C   |   malloc(0x180-8)

|-------|-------|-------|   C 块被覆盖

        |<-- malloced-->|
```

**二、拓展已分配块**：当溢出块的下一块为使用中的块，则需要合理控制溢出的字节，使其被释放时的合并操作能够顺利进行，例如直接加上下一块的大小使其完全被覆盖。

下一次分配对应大小时，即可取得已经被扩大的块，并造成进一步溢出。

```
  0x100   0x100    0x80

|-------|-------|-------|

|   A   |   B   |   C   |   初始状态

|-------|-------|-------|

|   A   |   B   |   C   |   溢出 B 的 size 为 0x180 (C->prev_size)

|-------|-------|-------|

|   A   |   B   |   C   |   释放 B

|-------|-------|-------|

|   A   |   B   |   C   |   malloc(0x180-8)

|-------|-------|-------|   C 块被覆盖

        |<-- malloced-->|
```

**三、`poison_null_byte`**：收缩被释放块，使其在之后分裂此块时将无法正确更新后一块的`prev_size`，导致释放时出现重叠的堆块。常用于`null byte`溢出中。

```
  0x100       0x210       0x80

|-------|---------------|-------|

|   A   |       B       |   C   |   初始状态

|-------|---------------|-------|

|   A   |       B       |   C   |   释放 B

|-------|---------------|-------|

|   A   |       B       |   C   |   溢出 B 的 size 为 0x200

|-------|---------------|-------|   之后的 malloc 操作没有更新 C 的 prev_size

         0x100  0x80

|-------|------|-----|--|-------|

|   A   |  B1  | B2  |  |   C   |   malloc(0x100-8), malloc(0x80-8)

|-------|------|-----|--|-------|

|   A   |  B1  | B2  |  |   C   |   释放 B1

|-------|------|-----|--|-------|

|   A   |  B1  | B2  |  |   C   |   释放 C，C 将与 B1 合并

|-------|------|-----|--|-------|  

|   A   |  B1  | B2  |  |   C   |   malloc(0x180-8)

|-------|------|-----|--|-------|   从B1开始，B2 将被覆盖

        |<-malloced->|
```

通常`B2`是一个目标结构体，包含我们想要控制的有价值的指针。

**四、 house_of_einherjar**：同样是`null byte`溢出，不过覆盖后的下一个块`prev_size`可以找到前一个合法的块，并与其进行合并。

```
  0x100   0x100   0x101

|-------|-------|-------|

|   A   |   B   |   C   |   初始状态

|-------|-------|-------|

|   A   |   B   |   C   |   释放 A

|-------|-------|-------|

|   A   |   B   |   C   |   溢出 B，覆盖 C 块的 size 为 0x200，并使其 prev_size 为 0x200

|-------|-------|-------|

|   A   |   B   |   C   |   释放 C

|-------|-------|-------|

|   A   |   B   |   C   |   C 将与 A 合并

|-------|-------|-------|   B 块被重叠

|<----- malloced ------>|
```

## 其他技巧
