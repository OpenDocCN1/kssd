
```c
// 2.23 when size > global_max_fast
/*
 * consolidate backward
 */
if (!prev_inuse(p)) {
  prevsize = p->prev_size;
  size += prevsize;
  p = chunk_at_offset(p, -((long) prevsize));
  unlink(av, p, bck, fwd);
}

if (nextchunk != av->top) {
  /* get and clear inuse bit */
  nextinuse = inuse_bit_at_offset(nextchunk, nextsize);

  /* consolidate forward */
  if (!nextinuse) {
    unlink(av, nextchunk, bck, fwd);
    size += nextsize;
  }

  /*
   * Take a chunk off a bin list
   */
  #define unlink(AV, P, BK, FD) {                                            \
    FD = P->fd;                                                              \
    BK = P->bk;                                                              \
    if (__builtin_expect (FD->bk != P || BK->fd != P, 0))                    \
      malloc_printerr (check_action, "corrupted double-linked list", P, AV); \
    else {                                                                   \
      FD->bk = BK;                                                           \
      BK->fd = FD;                                                           \
      if (!in_smallbin_range (P->size)                                       \
          && __builtin_expect (P->fd_nextsize != NULL, 0)) {                 \
        if (__builtin_expect (P->fd_nextsize->bk_nextsize != P, 0)           \
            || __builtin_expect (P->bk_nextsize->fd_nextsize != P, 0))       \
          malloc_printerr (check_action,                                     \
                           "corrupted double-linked list (not small)",       \
                           P, AV);                                           \
        if (FD->fd_nextsize == NULL) {                                       \
          if (P->fd_nextsize == P)                                           \
            FD->fd_nextsize = FD->bk_nextsize = FD;                          \
          else {                                                             \
            FD->fd_nextsize = P->fd_nextsize;                                \
            FD->bk_nextsize = P->bk_nextsize;                                \
            P->fd_nextsize->bk_nextsize = FD;                                \
            P->bk_nextsize->fd_nextsize = FD;                                \
          }                                                                  \
        } else {                                                             \
          P->fd_nextsize->bk_nextsize = P->bk_nextsize;                      \
          P->bk_nextsize->fd_nextsize = P->fd_nextsize;                      \
        }                                                                    \
      }                                                                      \
    }                                                                        \
  }
}
```

为了方便，依据物理地址相邻来命名如下：

![Snipaste_2021-08-31_11-47-47](https://bbs.kanxue.com/upload/attach/202109/904686_4RAY4HDC4JT24WV.jpg)

即当 `size` 大于 `global_max_fast` 且不是 `mmap` 出来的 `chunk` 时，就会进入判断。所以这里我们进行释放用的 `chunk` 的大小就必须要大于 `global_max_fast` 才行，否则就是改掉了 `pre_inuse` 位也是直接进入 `fastbin`，不会进入判断的。

- 先依据当前 `chunk`（`chunkP`）的 `pre_inuse` 位来判断前一个 `chunk`（`preChunk`）是否处于释放状态，是则进入 `unlink`，将前一个 `chunk` 取出
- 然后判断下一个 `chunk`（`nextChunk`）是否是 `top_chunk`，是则直接与 `top_chunk` 合并。
- 若 `nextChunk` 不为 `top_chunk`，再判断下一个 `Chunk` 的再下一个 `chunk` 的 `pre_inuse` 位来判断 `nextChunk` 是否处于释放状态，若是则进入 `unlink`。

然后 `unlink` 中就不细说，就是双向循环链表解链的过程，依据 `fd` 和 `bk` 来查找并解链，但是我们的 `off-by-null` 通常不会涉及到 `nextsize` 位的使用，所以基本不用看后面的。需要注意的是，由于这里会检查，即：

```c
if (__builtin_expect (FD->bk != P || BK->fd != P, 0))
  malloc_printerr (check_action, "corrupted double-linked list", P, AV);
```

所以我们需要将进入 `unlink` 的 `chunk` 的 `fd` 和 `bk` 来进行伪造或者干脆直接释放使其直接进入 `unsortedbin` 中完成双向链表的加持。这里先讲放入 `unsortedbin` 中来获取 `fd` 和 `bk` 的方法，伪造的方法一般用在 `2.29` 及以上的高版本中，因为那时候的 `unlink` 加入了关于 `size` 位的检查，不能简单得伪造 `fd` 和 `bk`。

其次，这里还需要明白一个寻找 `chunk` 的原理。

- 寻找 `preChunk`：`preChunk_addr = chunkP_addr - chunkP->pre_size`
- 寻找 `nextChunk`：`nextChunk_addr = chunkP_addr + chunkP->size`

即以下源码，这个一直没有变化过：

```c
/* Ptr to previous physical malloc_chunk.  Only valid if !prev_inuse (P). */
#define prev_chunk(p) ((mchunkptr) (((char *) (p)) - prev_size (p)))

/* Ptr to next physical malloc_chunk. */
#define next_chunk(p) ((mchunkptr) (((char *) (p)) + chunksize (p)))

/* Get size, ignoring use bits */
#define chunksize(p) (chunksize_nomask (p) & ~(SIZE_BITS))

/* extract p's inuse bit */
#define inuse(p) \
((((mchunkptr) (((char *) (p)) + chunksize (p)))->mchunk_size) & PREV_INUSE)
```

所以，如果我们可以伪造 `pre_size` 和 `in_use` 位，就能触发向上任意寻找一个满足 `fd` 和 `bk` 为双向链表的 `chunk`，从而将中间所有的 `chunk` 都一并合并为一个 `Chunk` 释放掉。(向下合并也可以的，不过一般不常使用)

![Snipaste_2021-08-31_11-57-00](https://bbs.kanxue.com/upload/attach/202109/904686_VPT4EBR54QSNQTV.jpg)

这里就是通过释放 `chunkP`，依据 `pre_size` 向上寻找到原本已经在 `unsortedbin` 中的 `preChunk`，其 `FD` 和 `BK` 已经组成双向循环链表，可以绕过检查，所以释放 `ChunkP` 之后 `preChunk+OverlapChunk+chunkP` 都进入到 `unsortedbin` 中。但是 `OverlapChunk` 本身其实并没有被释放，我们再从 `unsortedbin` 中申请切割出 `preChunk` 大小的 `chunk`，再申请就可以得到 `OverlapChunk`。这样我们就有两个指针都指向 `OverlapChunk`，从而伪造出 `UAF`，之后我们就可以通过 `OverlapChunk` 来 `getshell` 了。

## 1.常用布局

```python
add_malloc(0xf8, '\x00'*0xf8) #0x1
add_malloc(0x68, '\x00'*0x68) #0x2
add_malloc(0xf8, '\x00'*0xf8) #0x3
add_malloc(0x68, '\x00'*0x68) #0x4
free(0x1)
edit(0x2,0x70,'\x00'*0x60+p64(0x70+0x100)+p16(0x100))
free(0x3)
```

`off-by-null` 在调试中不太好搞，所以我就借用堆溢出来假设存在 `off-by-null`，将 `chunk3` 原本的 `size` 位 `0x101` 通过 `off-by-null` 变成 `0x100` 即可。

## 2.注意事项

### (1)顺序

此外需要注意的是，需要先释放 `chunk1`，再溢出修改 `chunk3`。不然如果先修改 `chunk3`，那么释放 `chunk1` 的时候，寻找 `chunk1` 的 `nextChunk` 即 `chunk2`，判断 `chunk2` 是否处于释放状态时，会找到 `chunk3`，依据 `pre_inuse` 位发现 `chunk2` 已经处于释放状态，那么尝试进入 `unlink` 合并，但是这里的 `chunk2` 的 `fd` 和 `bk` 并没有组成双向循环链表，所以会出错。

### (2)size位的设置

- `0x100`：这里注意到上面的布局中 `size` 位为 `0x100` 和 `0x70`，这里的 `0x100` 就是为了通过 `off-by-null` 将 `0x101` 变成 `0x100` 设置的。当然设置为 `0x201`，`0x301` 通常也是一样的。
- `0x70`：这里就通常是为了方便打 `fastbin attack`，从 `_malloc_hook` 处构造 `0x7f` 字节错位用的。

# 二、更新换代

## 1.Glibc2.27

这里也不是特指 `2.27`，而指的是 `Glibc2.29` 以下的存在 `tcache` 的版本，这类版本通常需要填充满 `tcache` 再进行释放，也不需要多讲。

## 2.Glibc2.29

从这个版本开始，`off-by-null` 由于加入的检查，引入了好几种全新的利用方式。

### (1)_int_free中的变化

```c
if (!prev_inuse(p)) {
  prevsize = prev_size (p);
  size += prevsize;
  p = chunk_at_offset(p, -((long) prevsize));
  if (__glibc_unlikely (chunksize(p) != prevsize))
    malloc_printerr ("corrupted size vs. prev_size while consolidating");
  unlink_chunk (av, p);
}
```

加入的检查是

```c
if (__glibc_unlikely (chunksize(p) != prevsize))
  malloc_printerr ("corrupted size vs. prev_size while consolidating");
```

这里的 `p` 因为 `p = chunk_at_offset(p, -((long) prevsize));` 已经变成了 `preChunk`。所以这里就是检查 `preChunk->size` 是否等于 `chunkP->pre_size`。按照上面那张图的逻辑，`preChunk` 的 `size` 为 `0x101`，`chunkP` 的 `pre_size` 为 `0x170`，两个不等于，根本就无法进入 `unlink` 中，直接崩掉。

![Snipaste_2021-08-31_11-57-00](https://bbs.kanxue.com/upload/attach/202109/904686_6C8MY85JGMEB3CT.jpg)

### (2)unlink变化

首先 `unlink` 从宏定义变成了全局函数定义，名字也从 `unlink` 变成了 `unlink_chunk`，但实际内容没有变太多，只是加入了一些检查：

```c
/* Take a chunk off a bin list. */
static void
unlink_chunk (mstate av, mchunkptr p)
{
  if (chunksize (p) != prev_size (next_chunk (p)))
    malloc_printerr ("corrupted size vs. prev_size");

  mchunkptr fd = p->fd;
  mchunkptr bk = p->bk;

  if (__builtin_expect (fd->bk != p || bk->fd != p, 0))
    malloc_printerr ("corrupted double-linked list");

  fd->bk = bk;
  bk->fd = fd;
  if (!in_smallbin_range (chunksize_nomask (p)) && p->fd_nextsize != NULL)
    {
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

加入的检查，就加了一个 `if` 语句：

```c
if (chunksize (p) != prev_size (next_chunk (p)))
  malloc_printerr ("corrupted size vs. prev_size");
```

即在 `unlink` 时会检查 `nextChunk` 的 `pre_size` 是否等于 `chunkP` 的 `size`。
