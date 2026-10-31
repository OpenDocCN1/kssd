
### (3) 先进先出

```c
#注释头

unsortedbin.bk->a;  a.bk->b     b.bk->c     c.bk->unsortedbin
unsortedbin.fd->c;  c.fd->b     b.fd->a     a.fd->unsortedbin
```

① `m = malloc(0x100);` -> `m->a`

② `n = malloc(0x100);` -> `n->b`

▲ 依据 `fd` 来遍历：

如果 `fd` 链顺序为 `A->B->C`，那么解链顺序一定是先解 `C`，再解 `B`，再解 `A`。

# 5. Tcache机制

从 `libc-2.26` 及以后都有：先进后出，最大为 `0x410`。

### (1) 结构

① 2.29以下，无 `key` 字段的 `tcache`，结构大小为 0x240，包括 `chunk` 头则占据 0x250：

```c
#注释头

typedef struct tcache_perthread_struct
{
  char counts[TCACHE_MAX_BINS];//0x40
  tcache_entry *entries[TCACHE_MAX_BINS];//0x40
} tcache_perthread_struct;
```

`A.counts`：是个数组，总共 64 个字节，对应 `tcache` 的 64 个 `bin`，每个字节代表对应 `bin` 中存在 `chunk` 的个数，所以每个字节都会小于 8，一般使用

```c
#注释头

tc_idx = csize2tidx(size);
tcache->counts[tc_idx]
```

来访问索引对应 `bin` 的 `count`。

![](https://bbs.kanxue.com/upload/attach/202108/904686_JE66B4J9ECMFF4D.jpg)

从 0x55555555b010 至 0x55555555b04f 都是 `counts` 这个数组的范围。

▲ 由于使用 `tcache` 时，不会检查 `tcache->counts[tc_idx]` 的大小是否处在 `[0,7]` 的范围，所以如果我们可以将对应 `bin` 的 `count` 改成 `[0,7]` 之外的数，这样下回再 `free` 该 `bin` 对应大小的 `chunk` 时，就不会将该 `chunk` 置入 `tcache` 中，使得 `tcache` 不满也能不进入 `tcache`。

`B.entries`：是个 `bin` 指针数组，共 64 个指针数组，每个指针 8 个字节，总计大小 0x200 字节，指针指向对应的 `bin` 中第一个 `chunk` 的首地址，这个首地址不是 `chunk` 头的首地址，而是对应数据的首地址。如果该 `bin` 为空，则该指针也为空。一般会使用 `tcache->entries[tc_idx] != NULL` 来判断是否为空。

② 2.29及以上，在 `entry` 中增加了 `key` 字段，结构体大小为 0x290，`count` 由原来的一个字节变为两个字节：

```c
#注释头

typedef struct tcache_entry
{
  struct tcache_entry *next;
  /* This field exists to detect double frees.  */
  struct tcache_perthread_struct *key; /* 新增指针 */
} tcache_entry;
```

### (2) 类似于一个比较大的 fastbins

总共 64 个 `bin`。

### (3) 归类方式

相同大小的 `chunk` 归到一类：大小范围 `[0x20, 0x410]`。每组 `bins` 中的 `chunk` 大小一定。且一组 `bin` 中最多只能有 7 个 `chunk`，如果 `free` 某大小 `bin` 的 `chunk` 数量超过 7，那么多余的 `chunk` 会按照没有 `tcache` 机制来 `free`。

### (4) 单向链表

▲ 例子：`a = malloc(0x10); b = malloc(0x10); c = malloc(0x10); d = malloc(0x10)`

`FastbinY,d,c,b,a`

① `free(a)` 之后：`tcachebins[0x20]->a; a.fd=0`

② `free(b)` 之后：`tcachebins[0x20]->b; b.fd=a; a.fd=0`

③ `free(c)` 之后：`tcachebins[0x20]->c; c.fd=b; b.fd->a; a.fd=0`

④ `free(d)` 之后：`tcachebins[0x20]->d; d.fd=c; c.fd->b; b.fd->a; a.fd=0`

★ 但是这里的 `fd` 指向的是 `chunk` 内容地址，而不是其它的 `bins` 中的 `fd` 指向的是 `chunk` 头地址。

### (5) 后进先出

后进先出，与 `fastbins` 类似。且 `tcache` 的优先级最高。

### (6) 特殊

① 当 `tcache` 某个 `bin` 被填满之后，再 `free` 相同大小的 `bin` 放到 `fastbin` 中或者 `smallbins` 中，之后连续申请 7 个该大小的 `chunk`，那么 `tcache` 中的这个 `bin` 就会被清空。之后再申请该大小的 `chunk` 就会从 `fastbins` 或者 `smallbins` 中找，如果找到了，那么返回该 `chunk` 的同时，会将该大小的 `fastbin` 或者 `smallbin` 中所有的 `chunk` 都移动到对应大小的 `tcache` 的 `bin` 中，直至填满 7 个。（移动时仍旧遵循先进后出的原则，所以移动之后 `chunk` 顺序会发生颠倒）

② `libc-2.26` 中存在 `tcache poisoning` 漏洞，即可以连续 `free(chunk)` 多次。

假如 `chunk0`、`chunk1`，然后 `free(chunk0)` 两次，这样 `tcache bin` 中就是：

`chunk0.fd -> chunk0`，即 `chunk0->chunk0`

那么第一次申请回 `chunk0`，修改 `fd` 为 `fakechunk`，`tcache bin` 中就是：

`chunk0.fd->fakechunk`，即 `chunk0->fakechunk`

之后再申请回 `chunk0`，再申请一次就是 `fakechunk` 了，实现任意地址修改。

★ 这个漏洞在 `libc2.27` 中就被修复了。

③ 从 `tcache` 中申请 `Chunk` 的时候不会检查 `size` 位，不需要构造字节错误。

### (7) 2.31下新增 stash 机制

在 `Fastbins` 处理过程中新增了一个 `Stash` 机制，每次从 `Fastbins` 取 `Chunk` 的时候会把剩下的 `Chunk` 全部依次放进对应的 `tcache`，直到 `Fastbins` 空或是 `tcache` 满。

### (8) 2.32下新增 fd 异或机制

会将 `fd` 异或上某个值。这个具体看其他文章吧，我自己调试大多都是异或 `heap_base/0x1000`，比较奇怪，具体题目具体分析。

# 6. Topchunk

`Topchunk` 不属于任何 `Bin`，在 `arena` 中属于最高地址，没有其它空闲块时，`topchunk` 就会被用于分配。

# 7. last_remainder

当请求 `small chunk` 大小内存时，如果发生分裂，则剩余的 `chunk` 保存为 `last_remainder`，放入 `unsortedbin` 中。

# 四、没有 `tcache` 的 `malloc` 和 `free` 流程

## `malloc` 流程

★ 如果是多线程情况下，那么会先进行分配区的加锁，这里就可能存在条件竞争漏洞。

1. 如果 `size` 在 `fastbins` 的范围内，则先在 `fastbins` 中查找，找到则结束，没找到就去 `unsortedbins` 中找。

2. 如果 `size` 不在 `fastbins` 范围中，而在 `smallbins` 范围中，则查找 `smallbins`（在 2.23 下如果发现 `smallbin` 链表未初始化，则会调用 `malloc_consolidate` 函数，但是实际情况在申请 `chunk` 之前都已经初始化过了，所以这个不怎么重要 `if (victim == 0) /* initialization check */ malloc_consolidate(av);` 而且这个操作从 2.27 开始已经没有了，如果能够让 `smallbin` 不初始化，或者将 `main_arena+0x88` 设置为 0），此时若 `smallbin` 找到结束，没找到去 `unsortedbins` 中找。

3. 如果 `size` 不在 `fastbins`、`smallbins` 范围中，那一定在 `largebins` 中，那么先调用 `malloc_consolidate` 函数将所有的 `fastbin` 中的 `chunk` 取出来，合并相邻的 `freechunk`，放到 `unsortedbin` 中，或者与 `topchunk` 合并。再去 `largebins` 中找，找到结束，没找到就去 `unsortedbins` 中找。

4. `unsortedbins` 中查找：

(1) 如果 `unsortedbin` 中只有 `last_remainder`，且分配的 `size` 小于 `last_remainder`，且要求的 `size` 范围为 `smallbin` 的范围，则分裂，将分裂之后的一个合适的 `chunk` 给用户，剩余的 `chunk` 变成新的 `last_remainder` 进入 `unsortedbin` 中。如果大于 `last_remainder`，或者分配的 `size` 范围为 `largebin` 的范围，则将 `last_remainder` 整理至对应 `bin` 中，跳至第 5 步。

(2) 如果 `unsortedbin` 中不只一个 `chunk`，则先整理，遍历 `unsortedbins`。如果遇到精确大小，直接返回给用户，接着整理完。如果一直没遇到过，则该过程中所有遇到的 `chunk` 都会被整理到对应的 `fastbins`、`smallbins`、`largebins` 中去。

5. `unsortedbins` 中还是找不到，则：

(1) 若当前 `size` 最开始判断是处于 `smallbins` 范围内，则再去 `smallbins` 找，这回不找精确大小，找最接近略大于 `size` 的一个固定大小的 `chunk` 给分裂，将符合 `size` 的 `chunk` 返回给用户，剩下的扔给 `unsortedbins`，作为新的 `last_remainder`。

(2) 若当前 `size` 最开始判断处于 `largebins` 范围内，则去 `largebins` 中找，和步骤 (1) 类似。

(3) 若当前 `size` 大于 `largebins` 中最大的 `chunk` 大小，那么就去 `topchunk` 来分割使用。

6. `topchunk` 分割：

(1) `topchunk` 空间够，则直接分割。

(2) `topchunk` 空间不够，那么再调用 `malloc_consolidate` 函数进行整理一下，然后利用 `brk` 或者 `mmap` 进行再拓展。

① `brk` 扩展：当申请的 `size` 小于 128K 时，使用该扩展。向高地址拓展新的 `topchunk`，一般加 0x21000，之后从新的 `topchunk` 再分配，旧的 `topchunk` 进入 `unsortedbin` 中。

② `mmap` 扩展：申请的 `size` 大于等于 `mmap` 分配阈值（最开始为 128k）时，使用该扩展，但是这种扩展申请到的 `chunk`，在释放时会调用 `munmap` 函数直接被返回给操作系统，而不会进入 `bins` 中。所以如果用指针再引用该 `chunk` 块时，就会造成 `segmentation fault` 错误。

▲ 当 `ptmalloc` munmap `chunk` 时，如果回收的 `chunk` 空间大小大于 `mmap` 分配阈值的当前值，并且小于 `DEFAULT_MMAP_THRESHOLD_MAX`（32 位系统默认为 512KB，64 位系统默认为 32MB），`ptmalloc` 会把 `mmap` 分配阈值调整为当前回收的 `chunk` 的大小，并将 `mmap` 收缩阈值（`mmap trim threshold`）设置为 `mmap` 分配阈值的 2 倍。这就是 `ptmalloc` 的对 `mmap` 分配阈值的动态调整机制，该机制是默认开启的，当然也可以用 `mallopt()` 关闭该机制。

▲ 如果将 `M_MMAP_MAX` 设置为 0，`ptmalloc` 将不会使用 `mmap` 分配大块内存。

## `free` 流程

★ 多线程情况下，`free()` 函数同样首先需要获取分配区的锁，来保证线程安全。

1. 首先判断该 `chunk` 是否为 `mmaped chunk`，如果是，则调用 `munmap()` 释放 `mmaped chunk`，解除内存空间映射，该空间不再有效。同时满足条件则调整 `mmap` 阈值。

2. 如果 `size` 位于 `fastbins` 范围内，直接放到 `fastbins` 中。

（这里有点争议，在《glibc内存管理ptmalloc源代码分析.pdf》一书中，写到：

> 如果执行到这一步，说明释放了一个与 `top chunk` 相邻的 `chunk`。则无论它有多大，都将它与 `top chunk` 合并，并更新 `top chunk` 的大小等信息。

但是实际测试，如果 `size` 位于 `fastbins` 范围内，则不管是否与 `topchunk` 相邻，都直接放到 `fastbin` 中，测试环境是 `libc2.23`，内核版本是 `Linux version 5.10.13`，也可能是某些参数的问题吧，具体题目分析就好了。）

3. 如果 `size` 不在 `fastbins` 范围内，则进行判断：

(1) 先判断前一个 `chunk_before`，如果 `chunk_before` 是 `free` 状态的，那么就将前一个 `chunk` 从其对应的 `bins` 中取出来（`unlink`），然后合并这两个 `chunk` 和 `chunk_before`。

由于还没有进入链表结构中，所以这里寻找 `chunk_before` 地址是通过当前地址减去当前 `chunk` 的 `presize` 内容，得到 `chunk_before` 的地址，从而获取其 `in_use` 位。

这也是 `presize` 唯一的用途，所以在堆溢出中，可以只要不进行 `free`，`presize` 可以任意修改。

（这里如果 `chunk_before` 是位于 `fastbins` 中则没办法合并，因为在 `fastbins` 中的 `in_use` 位不会被改变，永远是 1，在判断时始终认为该 `chunk` 是处于 `allocated` 状态的）

(2) 再判断后一个 `chunk`，如果后一个 `chunk` 是 `free` 状态，那么如步骤 (1)，合并，之后将合并后的 `chunk` 放入到 `unsortedbins` 中去。如果后一个 `chunk` 就是 `topchunk`，那么直接将这个 `chunk` 和 `topchunk` 合并完事。

之后将合并之后的 `chunk` 进行判断，（这里也可能不发生合并，但依旧会进行判断）如果 `size` 大于 `FASTBIN_CONSOLIDATION_THRESHOLD`（0x10000），那么就调用 `malloc_consolidate` 函数进行整理 `fastbins`，然后给到 `unsortedbins` 中，等待 `malloc` 时进行整理。

▲ 32位与64位区别，差不多其实，对于 0x8 和 0x10 的变化而已：

![](https://bbs.kanxue.com/upload/attach/202108/904686_MMR8E3QKEC9554N.jpg)

▲ 至于有 `tcache` 的 `malloc` 和 `free`，在各个版本其实都差不太多，除开 2.29 和 2.31、2.32 三个版本的不同检查和机制，都是判断 `count` 和链表来着，具体结合题目会比较好。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2021-8-8 23:26 被 PIG-007 编辑，原因：
