
如果克服了`[2]`，`[2]`是我们在上一节看到的安全宏：

``` 
#define ok_address(M, a) ((char*)(a) >= (M)->least_addr)
```

就在`least_addr`字段为“从`MORECORE`或者`MMAP`中获取最少的地址”…那么任何更高的值都可以通过检验。

我们达到了`unlink`的经典步骤，在`[3]` `bin`中的"`fd`"指针指向我们控制的地址。`[4]`是段违规`segmentation violation`发生的地方，因为它尝试把`bin`地址写给`(0x41414141)->bk`。当它超出分配的地址空间时，游戏结束。

在`Ptmalloc3`中的`smallbin`损坏技巧中，恰当地使用随意地址去重写一个释放缓冲区的"`fd`"指针是很有必要的。此外，调用相同分配大小的`malloc()`，从而返还跟分配空间一样的随意地址的过程也是很重要的。

注意事项跟`2.2.1`中的一样，`F->bk`必须包含一个可写的地址，否则它将导致`[4]`中的访问违例`access violation`。

如果我们完成了所有的这些条件，`bin`中的第一个内存块`chunk`将被解开，也将触发以下的代码：

----- snip -----

``` 
        mem = chunk2mem(p);

        check_malloced_chunk(gm, mem, nb);

        goto postaction;

        .....

        postaction:

            POSTACTION(gm);

            return mem;
```
----- snip -----

我在`mspace_malloc()`和`unlink_first_small_chunk()`宏中加了临时的`printf()`语句，看下发生了些什么，结果是：

```
Starting program: /home/black/ptmalloc3/thl `perl -e 'print "A"x24 .

"\x28\xf3\xff\xbf"'` < evil.in

[mspace_malloc()]: 16 bytes <= 244

Buff1 -> [ 0xb7feefe8 ]

[mspace_malloc()]: 128 bytes <= 244

Buff2 -> [ 0xb7fef000 ]

Buff3 -> [ 0xb7fef088 ]

Buff4 -> [ 0xb7fef190 ]

[mspace_malloc()]: 128 bytes <= 244

[unlink_first_small_chunk()]: P->fd = 0xbffff328

LB1 -> [ 0xb7fef000 ]

[mspace_malloc()]: 128 bytes <= 244

[unlink_first_small_chunk()]: P->fd = 0xbffff378

LB2 -> [ 0xbffff330 ]
```

Which is your favourite hobby:

This is an evil function. You become a cool hacker if you are able to

execute it

`"244"`是`MAX_SMALL_REQUEST`的现有值，我们可以看到，这是跟`Ptmalloc2`的另外一个不同点，`Ptmalloc2`中把任何时候小于`512`字节的请求都定义为`smallbin`，而在这里，限制范围更小了。

<< 从程序员的角度来看，用户是个外部设备：

发出读请求时敲键盘。>>

                                      [ P. Williams ]

### 4.2 LargeBin方法 (TreeBin 损坏方法)

在这里，我们已经正确了解了基本概念。你们可以自己接着深入学习`Ptmalloc3`。

在`Ptmalloc3`中，`large chunks`大内存块（即大于`256`字节）存在树形结构中，每个`chunk`都有指向它父类的指针，如果它们有子类，则保留着两个指向子类的指针（左跟右）。定义此结构的代码为：

----- snip -----

``` 
struct malloc_tree_chunk {

  /* 前面四个字段必须要跟malloc_chunk 兼容*/

  size_t                    prev_foot;

  size_t                    head;

  struct malloc_tree_chunk* fd;

  struct malloc_tree_chunk* bk;

  struct malloc_tree_chunk* child[2];

  struct malloc_tree_chunk* parent;

  bindex_t                  index;

};
```
----- snip -----

如果没有什么奇异现象的话，当有内存请求长整型缓冲区`long buffer`时，`"if (bytes <=MAX_SMALL_REQUEST) {}"`语句会失败，执行的代码将是：

----- snip -----

``` 
    else {

      nb = pad_request(bytes);

      if (ms->treemap != 0 && (mem = tmalloc_large(ms, nb)) != 0) {

        check_malloced_chunk(ms, mem, nb);

        goto postaction;

      }

    }
```
----- snip -----

深入`tmalloc_large()`，我们的目标是实现以下代码：

----- snip -----

``` 
  if (v != 0 && rsize < (size_t)(m->dvsize - nb)) {

    if (RTCHECK(ok_address(m, v))) { /* split */

      .....

      if (RTCHECK(ok_next(v, r))) {

        unlink_large_chunk(m, v);

        if (rsize < MIN_CHUNK_SIZE)

          set_inuse_and_pinuse(m, v, (rsize + nb));

        else {

          set_size_and_pinuse_of_inuse_chunk(m, v, nb);

          set_size_and_pinuse_of_free_chunk(r, rsize);

          insert_chunk(m, r, rsize);

        }

        return chunk2mem(v);

        .....
```
----- snip -----

如果用跟攻击`Ptmalloc2`一样的方法去`exploit`这个应用程序，那么这个程序首先会在`"unlink_large_chunk()"`宏（类似于`"unlink_first_small_chunk()"`）处中断，这个宏最重要的代码行是：

```
    F = X->fd;  [1]
    R = X->bk;  [2]
    F->bk = R;  [3]
    R->fd = F;  [4]
```

因此我们知道了重写的`chunk`的"`fd`" 和"`bk`"指针必须要指向可写的内存地址，否则将导致无效的内存访问。

下一个个错误将发生在`"set_size_and_pinuse_of_free_chunk(r, rsize)"`，这里告诉我们重写的`chunk`的”`size`”字段必须是被用户控制的，所以再一次说明了，我们需要有漏洞的应用程序允许我们引进`NULL`字节（`N`）。

如果我们能完成这个，那么在第`3`章展现的应用程序的`"malloc(1536)"`的第一次调用将被正确执行，接下来就到第二次调用了，特别是在循环（`loop`）当中：

----- snip -----

``` 
  while (t != 0) { /* find smallest of tree or subtree */

    size_t trem = chunksize(t) - nb;

    if (trem < rsize) {

      rsize = trem;

      v = t;

    }

    t = leftmost_child(t);

  }
```
----- snip -----

当你第一次进入这个循环中，`"t"`相当于在回应缓冲区请求的`tree_bin[]`中的第一个`chunk`的地址。只要`"t"`仍然有一些子地址，循环就会一直继续，最后`"v"` (`victim`)将包含能满足请求的最小块。

挽救问题的诀窍是在第一次迭代过后退出循环。为此，我们必须创建`"leftmost_child(t)"`返回一个`"0"`值。

知道了以下定义：

``` 
#define leftmost_child(t) ((t)->child[0] != 0? (t)->child[0]:(t)->child[1])
```

唯一的方法是在栈中的某个地址中放置(`buff2->bk`)。同时必须要给`child[0]`和`child[1]`指针赋予值`"0"`，表示没有子地址。接着在`"size"`字段在`if()` sentence 中为假时，`"t"` (接着 `"v"`)将被提供。

<< 软件在循环使用之前，应该是有用的。>>

                                  [ Ralph Johnson ]

### 4.3 执行安全检验

`Ptmalloc3`应该比之前看起来安全多了，但因此你应该在编译时间中定义好页脚（`FOOTERS`）常量（这不是默认的情况）。

在第`4`章节之前我们已经看过所有`malloc_state`结构中都出现的`"magic"`参数及它的运算方式。`"prev_size"`被改称为`"prev_foot"`的原因是，如果页脚(`FOOTERS`)被定义，那么这个字段就会用来存储内存块的`mstate`和最近算出的`magic`值的异或运算的结果。

``` 
/* 设置使用中的chunk的foot为mstate和seed的异或运算值*/
#define mark_inuse_foot(M,p,s)\
(((mchunkptr)((char*)(p)+(s)))->prev_foot = ((size_t)(M) ^ mparams.magic))
```

异或运算通常会用在对称加密算法中，同时通过存储`malloc_state`地址和建立一种`cookie`去防御攻击，这个`mstate`是在以下宏中获得的：

``` 
#define get_mstate_for(p)\
  ((mstate)(((mchunkptr)((char*)(p) +\
(chunksize(p))))->prev_foot ^ mparams.magic))
```

例如，被`wrapper free()`调用的`"mspaces_free()"`函数是这样子开始的：

``` 
   #if FOOTERS
       mstate fm = get_mstate_for(p);
   #else /* FOOTERS */
       mstate fm = (mstate)msp;
   #endif /* FOOTERS */
       if (!ok_magic(fm)) {
         USAGE_ERROR_ACTION(fm, p);
         return;
       }
```

如果我们损坏了一个分配的`chunk`的字节头（因此涉及到`prev_foot`字段），那么当这个`chunk`被释放时，`get_mstate_for()`将返回错误的分配区，此时`ok_magic()`将会测试那个区的”`magic`”值并中止应用程序。

另外，我们必须注意的是如果`fm->magic`的读数因为`get_mstate_for()`获得错误值的原因而导致分段错误，那么现在的`flow`可能在`USAGE_ERROR_ACTION()`调用之前中断。

如何处理这个`cookie`和概率分析问题从而预测出它在执行中的数值已经是个老话题了，我们再此不再赘述。尽管有人能想起`PaX`案例，也许是一个重写的指针可以指向`chunk`的”`size`”字段以外，并通过进一步的`STRxxx()` 或者 `MEMxxx()`调用，不用改变`"prev_foot"`就能粉碎它们的数据。`Skape`在`Windows`平台上“减少`gs cookies`的有效熵” `[4]`的工作做的非常棒，它可以给你一些独特的想法去应用。谁知道呢，这都取决于测试的应用程序的可攻击性和内在要求。

根据这个防护，`THoL`的优势在哪呢？非常明显，目标`chunk`在被释放之后被损坏，从而通过了完整性检验。

不管怎样，总会有办法解决这些问题的。首先，如果我们都知道内存分配应该是在进栈前就分配好的（`no memory allocation should proceed belonging to a stack location`），我们可以像下面这样简单执行下：

``` 
#define STACK_ADDR 0xbff00000
#define ok_address(M, a) (((char*)(a) >= (M)->least_addr)\
                          && ((a) <= STACK_ADDR))
```

接着应用程序在我们成功`exploit`之前中止了，同时`((a) >> 20) == 0xbff)`的检验应该也生效了。这只是个例子，你系统中相关的堆栈位置可能会很不一样，是一个严格的限制性保护。

每个看了源代码的人或许都会注意到`Ptmalloc3`的`unlink...( )`宏忽略掉了灌输在`glibc`中的经典测验：检验双向链表。我们没有考虑这点是因为我们知道一个真正的实现会考虑并添加这个完整性检验。但我这里就不再啰嗦了，除非将来出现了基于`Ptmalloc3`的`glibc`。

本概述的结论是一些在`Maleficarum & Des-Maleficarum`论文中细述的技巧不适用于`Ptmalloc3`。例如其中一个是`The House of Force`（暴力之屋）。记得它需要同时重写`wilderness chunk`的”`size`”字段和用户定义的大小请求。这在`Ptmalloc2`是有可能的因为`top chunk`的`size`是这样子读取的：

``` 
    victim = av->top;
    size = chunksize(victim);
```

不幸的是现在`Ptmalloc3`把这值存在`"malloc_state"`并直接通过以下方式读取：

``` 
   size_t rsize = (g)m->topsize // gm for dlmalloc( ), m for
                                // mspace_malloc( )
```

总之，回顾一下`"malloc.c"`开始之前的注释是非常值得的：

`"这只是安全的一方面--这些检验不会，`

`也不能检测到所有可能的编程错误".`

  << 没有全局架构或者全局构思的编程就像是只使用一只手电筒探索洞穴：你不知道所经之处，不知道前进方向，并且你根本不知道你身在何处>>

                                                 [ Danny Thorpe ]

### 4.3.1 安全堆分配器(乌托邦派)

首先，没有办法能发明出一个完全安全的`"堆分配器"`，这是不可能的（注意：你可以设计一个世界上最安全的分配器但如果时间太慢=>没用）。主要规则（已经相当明显了）意味着控制结构，或者更简单点，字节头（`header`）不能分配到临近数据的地方。创建一个可以添加`8`个字节到头地址从而直接存取数据的宏是非常简单的，但绝对不是一个安全的选择。

尽管这个问题会被解决，但仍然会有人认为：如果没有任意代码的执行，那么损坏另外一个分配好的`chunk`的数据是没用的，但如果这些缓冲区包含的是一些必须有保证的完整数据呢？（财务信息，其他..）

那么我们就接着讲到在已分配的内存碎片中使用`cookie`的必要性，虽然这明显是有副作用的。为了达到保持对齐的目标，`cookie`（假设`4`个字节）将会是每个分配的`chunk`的最后`4`个字节，因为如果把它们放到两个`chunk`之间，将需要更复杂及更慢的管理。

除此以外，我们可以借鉴下`Bruce Perens` 著作的`"Electric Fence - Red-Zone memory allocator"[5]`。他的保护方法是：

   - `Anti Double Frees`:

        ``` 
        if ( slot->mode != ALLOCATED ) {

                if ( internalUse && slot->mode == INTERNAL_USE )
                        .....

                else {

                        EF_Abort("free(%a): freeing free memory.",address);
        ```
