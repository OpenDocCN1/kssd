
```c
/* place chunk in bin */
if (in_smallbin_range (size)) //如果是smallbin的大小就放到smallbin
  {
    victim_index = smallbin_index (size);
    bck = bin_at (av, victim_index);
    fwd = bck->fd;
  }
else                                                    //如果是largebin的大小，那么：
  {
    victim_index = largebin_index (size);//根据size获取对应的largebin索引
    bck = bin_at (av, victim_index); //获取largebin表头
    fwd = bck->fd; //获取对应索引largebin的第一个chunk（循环链表的head->next）

    /* maintain large bins in sorted order */
    if (fwd != bck) //当第一个不等于最后一个（即当前的largebin不空）
      {
        /* Or with inuse bit to speed comparisons */
        size |= PREV_INUSE;
        /* if smaller than smallest, bypass loop below */
        assert (chunk_main_arena (bck->bk)); //是否在main_arena?（主线程）
        if ((unsigned long) (size) < (unsigned long) chunksize_nomask (bck->bk))//bck->bk储存的是当前索引的largebin中大小最小的chunk，如果我们要插入的chunk比这个大小还小，那么就要插入largebin的尾部。
          {
            fwd = bck; //fwd此时为largebin表头
            bck = bck->bk; //bck设置为largebin中最后一个的chunk

            victim->fd_nextsize = fwd->fd;//由于我们要插入的在末尾，比他小的就是循环回去的第一个chunk
            victim->bk_nextsize = fwd->fd->bk_nextsize;//比他大的就是之前的最小的那个

            //原来链表的第一个chunk的bk指向此时新插入的最后一个chunk
            fwd->fd->bk_nextsize = victim->bk_nextsize->fd_nextsize = victim;
          }

        // 如果不是插入尾部，那么我们要找到这个chunk应该插入的位置
        else
          {
            assert (chunk_main_arena (fwd));
            //使用这个while循环尝试从链表头部开始遍历，直到找到一个比victim大或等于的chunk退出while
            while ((unsigned long) size < chunksize_nomask (fwd))
              {
                fwd = fwd->fd_nextsize; //取下一个
                assert (chunk_main_arena (fwd));//检查分配区
              }

            //如果找到了跟他想的等
            if ((unsigned long) size == (unsigned long) chunksize_nomask (fwd))
              /* Always insert in the second position. */
              fwd = fwd->fd;//直接将victim插入他的后面（通过fd），不修改nextsize指针。

            //如果大小不一样(即此时fwd是相邻的大于victim的chunk)
            //需要构造nextsize双向链表，构造新节点,victim作为堆头
            else
              {
                //比victim小的指向fwd
                //比victim大的指向fwd的bk_nextsize（比fwd大的那个）
                //相当于插入了fwd与fwd->bk_nextsize之间
                victim->fd_nextsize = fwd;
                victim->bk_nextsize = fwd->bk_nextsize;

                if (__glibc_unlikely (fwd->bk_nextsize->fd_nextsize != fwd))//检查size链完整性
                  malloc_printerr ("malloc(): largebin double linked list corrupted (nextsize)");
                //对应的去改fwd的相关指针成链
                fwd->bk_nextsize = victim;
                victim->bk_nextsize->fd_nextsize = victim;
                //插入完成
              }

            bck = fwd->bk;
            if (bck->fd != fwd)
              malloc_printerr ("malloc(): largebin double linked list corrupted (bk)");
          }
      }
    else
      victim->fd_nextsize = victim->bk_nextsize = victim;//此时victim为唯一的chunk，也要做循环链表
  }
//放到对应的 bin 中，构成 bk<-->victim<-->fwd。
mark_bin (av, victim_index); //标识bitmap
victim->bk = bck;
victim->fd = fwd;
fwd->bk = victim;
bck->fd = victim;
```

贴一下插入过程大佬的总结：

1. 找到当前要插入的`chunk`对应的`largebin`的`index`，并定位该`index`中的最小的`chunk`（`bck`）和最大的`chunk`（`fwd`）。
2. 如果`fwd`等于`bck`，表明当前链表为空，则直接将该`chunk`插入，并设置该`chunk`为该大小堆块的堆头，将`bk_nextsize`和`fd_nextsize`赋值为它本身。
3. 如果`fwd`不等于`bck`，表明当前链表已经存在`chunk`，要做的就是找到当前`chunk`对应的位置将其插入。首先判断其大小是否小于最小`chunk`的`size`，`(size) < (bck->bk->size)`，如果小于则说明该`chunk`为当前链表中最小的`chunk`，即插入位置在链表末尾，无需遍历链表，直接插入到链表的末尾，且该`chunk`没有对应的堆头，设置该`chunk`为相应堆大小堆的堆头，将`bk_nextsize`指向比它大的堆头，`fd_nextsize`指向双链表的第一个节点即最大的堆头。
4. 如果当前`chunk`的`size`不是最小的`chunk`，则从双链表的第一个节点即最大的`chunk`的堆头开始遍历，通过`fd_nextsize`进行遍历，由于`fd_nextsize`指向的是比当前堆头小的堆头，因此可以加快遍历速度。直到找到小于等于要插入的`chunk`的`size`。
5. 如果找到的`chunk`的`size`等于要插入`chunk`的`size`，则说明当前要插入的`chunk`的`size`已经存在堆头，那么只需将该`chunk`插入到堆头的下一个节点。
6. 如果找到的`chunk`的`size`小于当前要插入`chunk`的`size`，则说明当前插入的`chunk`不存在堆头，因此该`chunk`会成为堆头插入到该位置，设置`fd_nextsize`与`bk_nextsize`。

### largebin的取出相关操作

```c
/* If a large request, scan through the chunks of current bin in sorted order to find smallest that fits.  Use the skip list for this. */
if (!in_smallbin_range (nb))//如果不在samllbin大小中
  {
    bin = bin_at (av, idx); //找到申请的size对应的largebin链表

    /* skip scan if empty or largest chunk is too small */
    if ((victim = first (bin)) != bin && //此时victim为链表的第一个节点
        (unsigned long) (victim->size) >= (unsigned long) (nb)) //第一步，判断链表的第一个结点，即最大的chunk是否大于要申请的size
      {
        //进入这里时，已经确定链表第一个节点——即最大的chunk大于要申请的size，那么我们就应该从这一条链中取，问题就是取这一条链上的哪一个？
        victim = victim->bk_nextsize; //本来victim是链中最大的那个，现在我们要从小往遍历，那么victim->bk_nextsize就循环回了链中最小的那个
        while (((unsigned long) (size = chunksize (victim)) < (unsigned long) (nb))) //第二步，从最小的chunk开始，反向遍历 chunk size链表，直到找到第一个大于等于所需chunk大小的chunk退出循环
          victim = victim->bk_nextsize;//victim取相邻的更大size的chunk

        /* Avoid removing the first entry for a size so that the skip list does not have to be rerouted. */
        if (victim != last (bin) && victim->size == victim->fd->size) //第三步，申请的chunk对应的chunk存在多个结点，则申请相应堆头的下个结点，不申请堆头。
          victim = victim->fd; //出现相同大小时堆头作为次优先申请

        remainder_size = size - nb;
        unlink (av, victim, bck, fwd); //第四步，largebin unlink 操作

        /* Exhaust */
        if (remainder_size < MINSIZE) //第五步，如果剩余的空间小于MINSIZE，则将该空间直接给用户
          {
            set_inuse_bit_at_offset (victim, size);
            if (av != &main_arena)
              victim->size |= NON_MAIN_ARENA;
          }
        /* Split */
        else
          {
            remainder = chunk_at_offset (victim, nb); //第六步，如果当前剩余空间还可以构成chunk，则将剩余的空间放入到unsorted bin中（切割后）。

            /* We cannot assume the unsorted list is empty and therefore have to perform a complete insert here. */
            bck = unsorted_chunks (av);//bck是ub头
            fwd = bck->fd; //fwd是ub第一个chunk
            if (__glibc_unlikely (fwd->bk != bck))
              {
                errstr = "malloc(): corrupted unsorted chunks";
                goto errout;
              }
            remainder->bk = bck;
            remainder->fd = fwd;
            bck->fd = remainder;
            fwd->bk = remainder;
            //以上操作完成后lastremainder被插入ub，成为新的链首元素
            //如果不在smallbin范围，那么nextsize指针置空
            if (!in_smallbin_range (remainder_size))
              {
                remainder->fd_nextsize = NULL;
                remainder->bk_nextsize = NULL;
              }

            set_head (victim, nb | PREV_INUSE |
                      (av != &main_arena ? NON_MAIN_ARENA : 0));
            set_head (remainder, remainder_size | PREV_INUSE);
            set_foot (remainder, remainder_size);
          }
        check_malloced_chunk (av, victim, nb);
        void *p = chunk2mem (victim);
        alloc_perturb (p, bytes);
        return p;
      }
  }
```

贴一下申请过程大佬的总结：

1. 找到当前要申请的空间对应的`largebin`链表，判断第一个结点即最大结点的大小是否大于要申请的空间，如果小于则说明`largebin`中没有合适的堆块，需采用其他分配方式。
2. 如果当前`largebin`中存在合适的堆块，则从最小堆块开始，通过`bk_nextsize`反向遍历链表，找到大于等于当前申请空间的结点。
3. 为减少操作，判断找到的相应结点（堆头）的下个结点是否是相同大小的堆块，如果是的话，将目标设置为该堆头的第二个结点，以此减少将`fd_nextsize`与`bk_nextsize`赋值的操作。
4. 调用`unlink`将目标`largebin chunk`从双链表中取下。
5. 判断剩余空间是否小于`MINSIZE`，如果小于直接返回给用户。
6. 否则将剩余的空间构成新的`chunk`放入到`unsorted bin`中。

## 伪造Largebin的`bk_nextsize`（取出时）
