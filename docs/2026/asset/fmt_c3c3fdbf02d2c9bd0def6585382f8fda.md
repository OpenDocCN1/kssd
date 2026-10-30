
```c
else
            {
              victim_index = largebin_index (size);
              bck = bin_at (av, victim_index);
              fwd = bck->fd;
              ....
              ....
              ....
              // 如果size<large bin中最后一个chunk即最小的chunk，就直接插到最后
                  if ((unsigned long) (size)
                      < (unsigned long) chunksize_nomask (bck->bk))
                    {
                      fwd = bck;
                      bck = bck->bk;
                      victim->fd_nextsize = fwd->fd;
                      victim->bk_nextsize = fwd->fd->bk_nextsize;
                      fwd->fd->bk_nextsize = victim->bk_nextsize->fd_nextsize = victim;
                    }
                  else
                    {
                      assert (chunk_main_arena (fwd));
                // 否则正向遍历，fwd起初是large bin第一个chunk，也就是最大的chunk。
              // 直到满足size>=large bin chunk size
                      while ((unsigned long) size < chunksize_nomask (fwd))
                        {
                          fwd = fwd->fd_nextsize;//fd_nextsize指向比当前chunk小的下一个chunk
                          assert (chunk_main_arena (fwd));
                        }
                      if ((unsigned long) size
                          == (unsigned long) chunksize_nomask (fwd))
                        /* Always insert in the second position.  */
                        fwd = fwd->fd;
                      else
                  // 插入
                        {
                          victim->fd_nextsize = fwd;
                          victim->bk_nextsize = fwd->bk_nextsize;
                          fwd->bk_nextsize = victim;
                          victim->bk_nextsize->fd_nextsize = victim;
                        }
                      bck = fwd->bk;
                    }
                }
              else
                victim->fd_nextsize = victim->bk_nextsize = victim;
            }
          mark_bin (av, victim_index);
          victim->bk = bck;
          victim->fd = fwd;
          fwd->bk = victim;
          bck->fd = victim;
```

当找到插入的位置后，看源码里具体的插入操作。

注意 large bin 要维持两个双向链表，多了一个 chunk size 链表，所以要在两个链表中插入。

```c
victim->fd_nextsize = fwd;
victim->bk_nextsize = fwd->bk_nextsize;
fwd->bk_nextsize = victim;
victim->bk_nextsize->fd_nextsize = victim;
....
....
victim->bk = bck;
victim->fd = fwd;
fwd->bk = victim;
bck->fd = victim;
```

在此题中，`fwd` 只可能是我们放入 large bin 的唯一一个 chunk，而它的 `bk_nextsize` 和 `bk` 都是我们可以控制的（如上一步的改写）。

```c
victim->bk_nextsize = fwd->bk_nextsize;
victim->bk_nextsize->fd_nextsize = victim;

----> fwd->bk_nextsize->fd_nextsize = victim
```

`victim` 就是我们要插入的堆地址。

`bk_nextsize` 被写为 `0x13370800-0x20-0x18-5`，那么 `*(0x13370800-0x20-0x18-5+0x20) = victim`。

```c
fwd->bk = victim;
```

`bk` 被写为 `0x13370800-0x20+8`，那么 `*(0x13370800 - 0x20 + 8) = victim`。

当第一个 chunk 从 unsorted bin 插入到 large bin 之后，再到 unsorted bin 的下一个 chunk，如果不满足分配则插入到 large bin 中。

而下一个 chunk 是我们伪造的（`0x13370800-0x20`）。

而这个地方已经有值了，也就是我们写入的 `*(0x13370800-0x20-0x18-5+0x20) = *0x133707e3 = victim`。

```c
pwndbg> x/gx 0x133707e3
0x133707e3:    0x000056213c4c8060
```

chunk 的 `size`，即 `0x13370800-0x20+0x8=0x133707e8`，就是 `\x55` 或者 `\x56`。

```c
# if the heap address starts with "0x56", you win
    alloc(0x48)     #2
```

之所以要求是 `\x56`，因为需要满足一个检查。

```c
assert (!mem || chunk_is_mmapped (mem2chunk (mem)) ||
        av == arena_for_chunk (mem2chunk (mem)));
```

即 chunk 的 mmap 标志位置位。

之前我调试的时候都是打开的 ASLR，现在关掉看一下，多运行几次总能有一次成功的。

![图片描述](https://bbs.kanxue.com/upload/attach/201902/93_YR23FUP2U59WTQW.png)

去掉标志位，那么它的大小就是 `0x50`，就满足 `alloc(0x48)`，就会返回给我们，成功返回 `0x13370800-0x10` 之后，就是传统的做法了。

### leak heap, leak libc, 覆盖 `free_book` 值为 `system`

## getshell

### exp
