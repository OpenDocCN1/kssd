
`static void malloc_consolidate(mstate av) {` `mfastbinptr *fb;` `/* current fastbin being consolidated */` `mfastbinptr *maxfb;` `/* last fastbin (for loop control) */` `mchunkptr p;` `/* current chunk being consolidated */` `mchunkptr nextp;` `/* next chunk to consolidate */` `mchunkptr unsorted_bin;` `/* bin header */` `mchunkptr first_unsorted;` `/* chunk to link to */` 
`/* These have same use as in free() */` `mchunkptr nextchunk;` `INTERNAL_SIZE_T size;` `INTERNAL_SIZE_T nextsize;` `INTERNAL_SIZE_T prevsize;` `int nextinuse;` `// 设置 av->have_fastchunks 为 false（0）` `atomic_store_relaxed(&av->have_fastchunks, false);` `// 取出 unsortedbin chunk` `unsorted_bin = unsorted_chunks(av);`

```
/*
Remove each chunk from fast bin and consolidate it, placing it
then in unsorted bin. Among other reasons for doing this,
placing in unsorted bin avoids needing to calculate actual bins
until malloc is sure that chunks aren't immediately going to be
reused anyway.
移除fastbin中的chunk，然后合并，放到unsortedbin
*/
```
`// 取出最大的 fastbin` `maxfb = &fastbin(av, NFASTBINS - 1);` `// 取出最小的 fastbin` `fb = &fastbin(av, 0);` `do` `{` `p = atomic_exchange_acq(fb, NULL);` `// 从最小的fb到最大的fb进行遍历，有chunk就进入处理` `if (p != 0)` `{` `// 遍历每一个 fastbin chunk` `do` `{` `// 安全检查：p 需要是内存对齐的` `if (__glibc_unlikely(misaligned_chunk(p)))` `malloc_printerr("malloc_consolidate(): " "unaligned fastbin chunk detected");` `// 获取 fastbin 索引` `unsigned int idx = fastbin_index(chunksize(p));` `// 安全检查：该fastbin的大小检查，不能是其他大小` `if ((&fastbin(av, idx)) != fb)` `malloc_printerr("malloc_consolidate(): invalid chunk size");` `}` `// 检查 prev_inuse 位为 1` `check_inuse_chunk(av, p);` `// 解密 next 指针，拿到next chunk地址` `nextp = REVEAL_PTR(p->fd);`

`/* Slightly streamlined version of consolidation code in free() */`
`// 轻量线性版本的free()的consolidation`
`// 获取大小` `size = chunksize(p);` `// 获取next chunk 及其 size` `nextchunk = chunk_at_offset(p, size);` `nextsize = chunksize(nextchunk);` `// 如果prev_inuse==0，意味着上一个chunk是空闲的normal chunk，向上（低地址）合并` `if (!prev_inuse(p))` `{` `// 获取 prev_size，计算合并后大小 size，获取prev chunk ptr` `prevsize = prev_size(p);` `size += prevsize;` `p = chunk_at_offset(p, -((long)prevsize));` `// 安全检查：如果chunk size和next chunk 的 prev_size不一致，报错` `if (__glibc_unlikely(chunksize(p) != prevsize))` `malloc_printerr("corrupted size vs. prev_size in fastbins");` `// 双链表断链 prev chunk` `unlink_chunk(av, p);` `}` `// 如果下一个chunk不是top chunk` `if (nextchunk != av->top)` `{` `// 判断再下一个chunk的prev_inuse` `nextinuse = inuse_bit_at_offset(nextchunk, nextsize);`

`// 如果是0，表示next chunk是空闲的` `if (!nextinuse)` `{` `// 大小合并，断链` `size += nextsize;` `unlink_chunk(av, nextchunk);` `}` `else` `// 清除next chunk的prev_inuse位` `clear_inuse_bit_at_offset(nextchunk, 0);`

`// 插入到 unsortedbin 的前面` `// 取出unsortedbin中的第一个` `first_unsorted = unsorted_bin->fd;` `// 第一个设置成新的chunk` `unsorted_bin->fd = p;` `// 原本第一个的上一个设置成新的chunk` `first_unsorted->bk = p;`

`// 如果是largebin size chunk，就清空nextsize位` `if (!in_smallbin_range(size))` `{` `p->fd_nextsize = NULL;` `p->bk_nextsize = NULL;` `}` `// 设置标志位，完成插入操作` `set_head(p, size | PREV_INUSE);` `p->bk = unsorted_bin;` `p->fd = first_unsorted;` `set_foot(p, size);` `}` `// 如果下一个chunk是top chunk` `else` `{` `// 合并到top chunk` `size += nextsize;` `set_head(p, size | PREV_INUSE);` `av->top = p;` `}`

`}` `while ((p = nextp) != 0);` `}` `}` `while (fb++ != maxfb);` `}`

简单来说，就是轻量版本的`free`里的`consolidation`过程，是针对`fastbin chunk`的，流程如下：

1. 设置`av->have_fastchunks`为`false`
2. 遍历每一个`fastbin`
3. 遇到`fastbin chunk`，就进行合并
   1. 安全检查：该`chunk`的大小和该`bin`的大小需要匹配
   2. 向上合并（低地址），能合就合
   3. 向下合并（高地址），能合就合
      1. 如果不是`top chunk`，就插入到`unsortedbin`的前面
      2. 如果是`top chunk`，就合并到`top chunk`中

### 进入 `malloc_consolidation` 的条件是什么？

1. `_int_malloc`中，当不能从`fastbin`中申请，且申请大小不属于`small size`时，如果当前`arena`有`fastbin chunk`，就会进行调用：

```
else {
 // largebin 中取出
 idx = largebin_index(nb);
 if (atomic_load_relaxed(&av->have_fastchunks))
 malloc_consolidate(av);
}
```

2. `_int_malloc`中，当无法通过`top chunk`分配，且`arena`中有`fastbin chunk`时，就会进行调用：

```
else if (atomic_load_relaxed(&av->have_fastchunks))
{
 malloc_consolidate(av);
// 合并操作
/* restore original bin index */
// 保存原本bin索引
if (in_smallbin_range(nb))
 idx = smallbin_index(nb);
else
 idx = largebin_index(nb);
}
```

3. `_int_free`中，释放到`unsortedbin`进行`consolidation`的过程中，在向前向后合并完成了以后，如果合并的大小超过`0x10000`，就检测`fastbin chunk`并进行合并：

```
if ((unsigned long)(size) >= FASTBIN_CONSOLIDATION_THRESHOLD)
// 阈值：65536
{
 // 如果fastbins存在，就合并
 if (atomic_load_relaxed(&av->have_fastchunks))
 malloc_consolidate(av);
```

### `fastbin chunk` 进行合并有哪些安全检查？

和 `libc 2.23` 相比，新增了3个检查：

1. 在遍历的时候，会检查`chunk`的内存对齐
2. 在遍历的时候，会检查`chunk`的大小和`fastbin`的大小是否匹配
3. **在向前合并的时候，会检查`prev_size`和前一个`chunk`的`size`是否相同**

```
// 如果prev_inuse==0，意味着上一个chunk是空闲的normal chunk，向上（低地址）合并
if (!prev_inuse(p))
{
 // 获取 prev_size，计算合并后大小 size，获取prev chunk ptr
 prevsize = prev_size(p);
 size += prevsize;
 p = chunk_at_offset(p, -((long)prevsize));
 // 安全检查：如果chunk size和next chunk 的 prev_size不一致，报错
 if (__glibc_unlikely(chunksize(p) != prevsize))
 malloc_printerr("corrupted size vs. prev_size in fastbins");
 // 双链表断链 prev chunk
 unlink_chunk(av, p);
}
```

## 实验：`hitbctf2018 - netupig`

实验环境：`libc 2.23`

> 这个实验我做了2天，还是思想太局限了，没收集好题目给出的信息，就在硬做，看到大佬的`exp`，恍然大悟，实在精彩！精彩！

### 题目分析

保护：没有`PIE`，以及`Partial RELRO`，意味着`got`表可以作为利用目标

```
Arch: amd64-64-little
RELRO: Partial RELRO
Stack: Canary found
NX: NX enabled
PIE: No PIE (0x3fe000)
```

`main`函数：很简洁，三个功能，其中没有任何地方能打印

```
void __fastcall __noreturn main(__int64 a1, char **a2, char **a3)
{
 int choose;
// [rsp+4h] [rbp-Ch] BYREF
 unsigned __int64 v4;
// [rsp+8h] [rbp-8h]

 v4 = __readfsqword(0x28u);
 sub_4008DC(a1, a2, a3);
 sub_400950();
 while ( 1 )
 {
 __isoc99_scanf("%d", &choose);
 getchar();
 switch ( choose )
 {
 case 2:
 fp_free();
 break;
 case 3:
 fp_edit();
 break;
 case 1:
 fp_malloc();
// 1:0x10
// 2:0x80
// 3:0xa00000
 break;
 }
 }
}
```

`malloc`的函数：根据输入的选项申请固定大小的内存
