
# [原创]堆溢出-House of orange 学习笔记

[toc]

# House_of_orange 堆溢出

前几天把`House of orange`重新学习了一下，比照着`glibc malloc`的源码好好分析了一下，希望做到真正做到知其然亦知其所以然，其中所做的笔记如下，可能描述上有点乱，大家将就着看一下吧。同时，我也发在了我个人博客上面（<mark class="encrypted">c61K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6Q4x3X3g2D9k6h3q4F1L8%4c8W2i4K6u0W2j5$3!0E0i4K6u0r3M7$3W2E0M7o6q4W2i4K6t1&6i4K6u0o6</mark>），如果有错误的地方，请各位大牛多多指正。

## 0x00 程序描述

大名鼎鼎的`house_of_orange`。

程序逻辑还比较清晰的，一共可以`build`四次，然后每次`build`的话就是3次堆分配，两次`malloc`，一次`calloc`，其中一次`malloc`是固定分配`0x10`字节作为控制堆块，里面存放着`name`和`color`的信息，另外按输入分配`name`的大小。

![](upload/attach/201711/768814\_wdgr9ioojkkjtm7.png)

## 0x01 程序漏洞

### 1. 堆溢出

![](upload/attach/202002/768814\_UYH3NXJNVMMSWC9.jpg)

在`upgrade`函数中，修改`name`时候不顾实际`chunk`的堆大小是多少，直接进行编辑，最大可编辑`0x1000`大小，因而存在溢出。

## 0x02 漏洞利用

这里的利用思路是`4ngelboy`给出，下面就直接分析这样利用的原因。

### 1. 信息泄露（泄露libc地址）

因为程序中有堆的越界写，可以修改`top_chunk`的大小。在`malloc`源码里面如果申请的堆块大小超过了`top_chunk`的大小，将调用`sysmalloc`来进行分配。

`sysmalloc`里面针对这种情况有两种处理，一种是直接`mmap`出来一块内存，另一种是扩展`top_chunk`。

```
/*
If have mmap, and the request size meets the mmap threshold, and
the system supports mmap, and there are few enough currently
allocated mmapped regions, try to directly map this request
rather than expanding top.
*/
if ((unsigned long) (nb) >= (unsigned long) (mp_.mmap_threshold) &&
(mp_.n_mmaps < mp_.n_mmaps_max))
{
char *mm; /* return value from mmap call*/
try_mmap:
```

就是如果申请大小>=`mp_.mmap_threshold`，就会`mmap`。我们只要申请不要过大，一般不会触发这个，这个`mmap_threshold`的值为`128*1024`。

不过下面还有两个`assert`需要检查，如下：

```
old_top = av->top;
old_size = chunksize (old_top);
old_end = (char *) (chunk_at_offset (old_top, old_size));
brk = snd_brk = (char *) (MORECORE_FAILURE);
/*
If not the first time through, we require old_size to be
at least MINSIZE and to have prev_inuse set.
*/
assert ((old_top == initial_top (av) && old_size == 0) ||
((unsigned long) (old_size) >= MINSIZE &&
prev_inuse (old_top) &&
((unsigned long) old_end & pagemask) == 0));
/* Precondition: not enough current space to satisfy nb request */
assert ((unsigned long) (old_size) < (unsigned long) (nb + MINSIZE));
```

第一个`assert`就是要求修改后的`top_chunk_size`必须满足：

1. `top_chunk_size`大于`MINSIZE`（`MINSIZE`没有查到是多少，反正不要太小都行）。
2. `top_chunk`需要有`prev_inuse`的标志，就是最后一个比特为1。
3. 还有就是`(old_end & pagemask) == 0`。

`#define chunk_at_offset(p, s) ((mchunkptr) (((char *) (p)) + (s)))`

这里没有太深究，应该就是`top_chunk`需要和原来的堆页在一个页上吧。

第二个`assert`就是要求：

4. `top_chunk_size`小于申请分配的内存即可。

满足以上四个条件之后，继续往下执行，最后把原先的那个`old_top`给释放掉了，如下：

```
top (av) = chunk_at_offset (heap, sizeof (*heap));
set_head (top (av), (heap->size - sizeof (*heap)) | PREV_INUSE);
/* Setup fencepost and free the old top chunk with a multiple of
   MALLOC_ALIGNMENT in size. */
/* The fencepost takes at least MINSIZE bytes, because it might
   become the top chunk again later. Note that a footer is set
   up, too, although the chunk is marked in use. */
old_size = (old_size - MINSIZE) & ~MALLOC_ALIGN_MASK;
set_head (chunk_at_offset (old_top, old_size + 2 * SIZE_SZ), 0 | PREV_INUSE);
if (old_size >= MINSIZE)
{
set_head (chunk_at_offset (old_top, old_size), (2 * SIZE_SZ) | PREV_INUSE);
set_foot (chunk_at_offset (old_top, old_size), (2 * SIZE_SZ));
set_head (old_top, old_size | PREV_INUSE | NON_MAIN_ARENA);
_int_free (av, old_top, 1);
```

显然，这样的`free`操作的话，我们就可以得到一个`unsort_bin`，然后之后再次分配时候如果是符合`unsort_bin`大小的话，就会从`unsort_bin`里面切出来。

![](upload/attach/202002/768814\_YRXMHW3VEDAFXSY.jpg)

这样的话我们再次申请一个堆块分配到这块区域中就能泄露`libc`地址了，但是这里又有一个`trick`，如果我们分配的大小是`large_chunk`的话。`malloc`源码中还把`old_top`的堆地址放到了堆里面（没有细究原因，但是好像是large bin没有区分大小，需要有个字段来保存大小的原因吧），源码如下：

![](upload/attach/202002/768814\_GF9TJ34XK8EMVF5.jpg)

所以如果再次分配时候如果分配大小为`largebin`（也就是大于512字节）的`chunk`的话，就是可以既泄露`libc`又可以泄露`heap`。如下：

![](upload/attach/202002/768814\_5ZBZUU8PTH2KFXW.jpg)

而如果分配大小不到512字节时候是无法泄露堆地址的。

![](upload/attach/202002/768814\_XQ5XBMG85EFUSER.jpg)

### 2. 劫持流程

#### File Stream Oriented Programming

我们知道有`rop`即`return Oriented Programming`，那么其实`File Stream Oriented Programming`是一个道理的。也是一种劫持程序流程的方法，只不过方式是通过攻击`File Stream`来实现罢了。

我们先要了解`malloc`对错误信息的处理过程，`malloc_printerr`是`malloc`中用来打印错误的函数。

![](upload/attach/202002/768814\_PP5324CWYKQEXRF.jpg)

`malloc_printerr`其实是调用`__libc_message`函数之后调用`abort`函数，`abort`函数其中调用了`_IO_flush_all_lockp`，这里面用到`IO_FILE_ALL`里面的结构，采用的是虚表调用的方式。

![](upload/attach/202002/768814\_U29UVNRZWMC4KGG.jpg)

其中使用到了`IO_FILE`对象中的虚表，如果我们能够修改`IO_FILE`的内容那么就可以一定程度上劫持流程。

`IO_FILE_ALL`是一个指向`IO_FILE_plus`的结构指针，结构如下图所示，具体结构不需要太了解清晰，大概懂一些也就行。

![](upload/attach/202002/768814\_CP7DSKNKF2C9YVJ.jpg)

那么怎么劫持呢，这里又需要用到`unsorted bin attack`的知识。

`unsorted bin attack`是怎么一回事呢，其实就是在`malloc`的过程中，`unsorted bin`会从链表上卸下来（只要分配的大小不是`fastchunk`大小）。

![](upload/attach/202002/768814\_AXDDW8TA5KBJR8R.jpg)

如上代码所示，就是会把`bk+0x10`的地方写入本`unsorted_bin`的地址。

![](upload/attach/202002/768814\_P8W5VU4GGWKVZZ9.jpg)

我们通过内存断点来观察一下是如何进行的。

断点触发之后，发现`io_file_all`被修改成了指向`top_chunk`的指针，时间地址位于`main_arena`。

![](upload/attach/202002/768814\_CMEH72QG69HE5N2.jpg)

但是我们是无法控制`main_arena`的内容的，至少全部控制是不行的，那么怎么处理呢？

这里还是要牵扯到`io_file`的使用，`IO_FILE`结构中有一个字段是`chain`字段，它位于`0x60`偏移处，他指向的是下一个`IO_FILE`结构体，我们如果可以控制这个字段，就再次指定`io_file`的位置，它相当于是一个链表的结构。

这样的话又联系到`smallchunk`的问题，在拆卸`unsorted_bin`时候对属于`small_bin`的`chunk`进行了记录操作。

这个时候`IO_FILE_all`指向的正是`main_arena`的`bins`里面`unsorted_bin`的位置，那么偏移`0x60`处正好是`smallchunk`的index为6的地方，也就是满足大小为`16*6`的`chunk`，所以`upgrade`时候需要把`unsorted_bin`设置为`0x60`大小。

```
while (fp != NULL)
{
...
fp = fp->_chain;
...
if (((fp->_mode <= 0 && fp->_IO_write_ptr > fp->_IO_write_base)
#if defined _LIBC || defined _GLIBCPP_USE_WCHAR_T
|| (_IO_vtable_offset (fp) == 0
&& fp->_mode > 0 && (fp->_wide_data->_IO_write_ptr
> fp->_wide_data->_IO_write_base))
#endif
)
&& _IO_OVERFLOW (fp, EOF) == EOF)
```

因为第一个分配在`main_arena`的`IO_FILE_plus`结构的`fp->mode`等值不符合要求，就会通过`chain`跳转到下一个`IO_FILE_plus`，就是我们之前设置的`unsorted_bin`，然后需要满足一下条件：

1. `fp->mode > 0`
2. `_IO_vtable_offset(fp) == 0`
3. `fp->_wide_data->_IO_write_ptr > fp->_wide_data->_IO_write_base`

这里的话我就是把`wide_data`的`IO_write_ptr`就指向`read_end`就可以，然后就会调用虚表`+0x18`偏移处的函数了。

![](upload/attach/202002/768814\_JE46UAQ3283VD43.jpg)

## 0xFE 利用exp
