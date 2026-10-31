
3. 当前申请的 mmap 内存数量没有超过 [`DEFAULT_MMAP_MAX`](https://bbs.kanxue.com/elink@b0dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6T1L8h3W2F1L8%4u0Q4x3V1k6Y4L8r3W2T1j5#2)9J5c8X3u0D9L8$3u0Q4x3V1k6Y4L8r3W2T1j5#2)9J5k6o6u0Q4x3X3f1J5y4#2)9J5c8X3#2S2L8r3I4G2j5#2)9J5c8X3#2S2L8r3I4G2j5#2)9J5k6h3y4Q4x3U0y4x3z5e0V1K6)，默认是 65536

mmap 内存申请暂不做具体分析，先跳过。

如果即没有可用的 arena 也没有申请到 mmap 内存，则直接返回空。

如果 `av` 不为空，则表示有可用的 arena，通过扩展 `av->top` 后从 `av->top` 中划分出所需 chunk 块。

线程申请的内存 chunk 时实际是从 arena 中的 top chunk 中划分出来的，[`sysmalloc`](https://bbs.kanxue.com/elink@079K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6T1L8h3W2F1L8%4u0Q4x3V1k6Y4L8r3W2T1j5#2)9J5c8X3u0D9L8$3u0Q4x3V1k6Y4L8r3W2T1j5#2)9J5k6o6u0Q4x3X3f1J5y4#2)9J5c8X3#2S2L8r3I4G2j5#2)9J5c8X3#2S2L8r3I4G2j5#2)9J5k6h3y4Q4x3U0y4x3x3U0t1%4x3H3`.`.) 函数用于扩展 topchunk 后分配 chunk。

如果topchunk对于申请的大小还有足够的空间，则直接触发异常：

```
/* Precondition: not enough current space to satisfy nb request */
assert((unsigned long) (old_size) < (unsigned long) (nb + MINSIZE));
```

如果 `av` 是 `thread_arena`，首先尝试调用 [`grow_heap`](https://bbs.kanxue.com/elink@ce8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6T1L8h3W2F1L8%4u0Q4x3V1k6Y4L8r3W2T1j5#2)9J5c8X3u0D9L8$3u0Q4x3V1k6Y4L8r3W2T1j5#2)9J5k6o6u0Q4x3X3f1J5y4#2)9J5c8X3#2S2L8r3I4G2j5#2)9J5c8X3q4J5k6h3&6S2i4K6u0W2j5#2)9J5x3@1H3#2x3K6l9`.) 扩展 top 堆，如果失败就调用 [`new_heap`](https://bbs.kanxue.com/elink@925K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6T1L8h3W2F1L8%4u0Q4x3V1k6Y4L8r3W2T1j5#2)9J5c8X3u0D9L8$3u0Q4x3V1k6Y4L8r3W2T1j5#2)9J5k6o6u0Q4x3X3f1J5y4#2)9J5c8X3#2S2L8r3I4G2j5#2)9J5c8X3q4J5k6h3&6S2i4K6u0W2j5#2)9J5x3@1H3@1y4e0t1`.) 创建新堆。

[`new_heap`](https://bbs.kanxue.com/elink@df0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6T1L8h3W2F1L8%4u0Q4x3V1k6Y4L8r3W2T1j5#2)9J5c8X3u0D9L8$3u0Q4x3V1k6Y4L8r3W2T1j5#2)9J5k6o6u0Q4x3X3f1J5y4#2)9J5c8X3#2S2L8r3I4G2j5#2)9J5c8X3q4J5k6h3&6S2i4K6u0W2j5#2)9J5x3@1H3@1y4e0t1`.) 中实际是利用 mmap 方式分配内存。[`new_heap`](https://bbs.kanxue.com/elink@59dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6T1L8h3W2F1L8%4u0Q4x3V1k6Y4L8r3W2T1j5#2)9J5c8X3u0D9L8$3u0Q4x3V1k6Y4L8r3W2T1j5#2)9J5k6o6u0Q4x3X3f1J5y4#2)9J5c8X3#2S2L8r3I4G2j5#2)9J5c8X3q4J5k6h3&6S2i4K6u0W2j5#2)9J5x3@1H3@1y4e0t1`.) 中限制一个 mmap 堆的最大空间默认是 [`HEAP_MAX_SIZE`](https://bbs.kanxue.com/elink@cb5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6T1L8h3W2F1L8%4u0Q4x3V1k6Y4L8r3W2T1j5#2)9J5c8X3u0D9L8$3u0Q4x3V1k6Y4L8r3W2T1j5#2)9J5k6o6u0Q4x3X3f1J5y4#2)9J5c8X3#2S2L8r3I4G2j5#2)9J5c8X3q4J5k6h3&6S2i4K6u0W2j5#2)9J5x3@1H3K6y4l9`.`.)，分配的最小堆大小为 [`HEAP_MIN_SIZE`](https://bbs.kanxue.com/elink@b12K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6T1L8h3W2F1L8%4u0Q4x3V1k6Y4L8r3W2T1j5#2)9J5c8X3u0D9L8$3u0Q4x3V1k6Y4L8r3W2T1j5#2)9J5k6o6u0Q4x3X3f1J5y4#2)9J5c8X3#2S2L8r3I4G2j5#2)9J5c8X3q4J5k6h3&6S2i4K6u0W2j5#2)9J5x3@1H3J5z5b7`.`.)：

```
#define HEAP_MIN_SIZE (32 * 1024)
#ifndef HEAP_MAX_SIZE
# ifdef DEFAULT_MMAP_THRESHOLD_MAX
#  define HEAP_MAX_SIZE (2 * DEFAULT_MMAP_THRESHOLD_MAX)
# else
#  define HEAP_MAX_SIZE (1024 * 1024) /* must be a power of two */
# endif
#endif
```

如果超过这个限制就会分配失败，[`sysmalloc`](https://bbs.kanxue.com/elink@2a9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6T1L8h3W2F1L8%4u0Q4x3V1k6Y4L8r3W2T1j5#2)9J5c8X3u0D9L8$3u0Q4x3V1k6Y4L8r3W2T1j5#2)9J5k6o6u0Q4x3X3f1J5y4#2)9J5c8X3#2S2L8r3I4G2j5#2)9J5c8X3#2S2L8r3I4G2j5#2)9J5k6h3y4Q4x3U0y4x3x3U0t1%4x3H3`.`.) 再尝试直接调用 mmap 创建所需大小的映射内存块。

如果 `av` 是 `main_arena`，则利用 brk 方式分配内存，如果 brk 失败则调用 mmap 分配内存。因为这种 brk 和 mmap 混合使用的情况所以可能会造成内存无法连续，具体 brk 分配过程不再细做分析，先跳过。

不管是 `main_arena` 还是 `thread_arena`，因为 `top_pad` 的存在：

```
{
  /* Request enough space for nb + pad + overhead */
  size = nb + mp_.top_pad + MINSIZE;
}
```

`nb` 是请求 chunk 块的大小，最小 0x20，`mp_.top_pad` 默认是 [`DEFAULT_TOP_PAD`](https://bbs.kanxue.com/elink@971K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6T1L8h3W2F1L8%4u0Q4x3V1k6Y4L8r3W2T1j5#2)9J5c8X3u0D9L8$3u0Q4x3V1k6Y4L8r3W2T1j5#2)9J5k6o6u0Q4x3X3f1J5y4#2)9J5c8Y4y4&6M7$3c8W2M7s2y4Q4x3V1k6Y4k6h3&6W2M7X3W2U0i4K6u0r3L8h3q4D9L8r3!0U0i4K6u0V1L8h3q4U0K9r3W2F1k6g2)9J5k6h3S2Q4x3U0y4x3x3K6V1`.)，即 0x20000：

```
#ifndef DEFAULT_TOP_PAD
# define DEFAULT_TOP_PAD 131072
#endif
```

所以 top 堆的最小长度为 0x20040，没有最大限制。另外 top 堆长度还要与系统 `page_size` 对齐。

最终 top 堆完成扩容操作。然后从 top 堆顶划分出 `nb` 长度的 chunk 块返回。

# 内存释放

程序调用 glibc 释放内存的入口为 [`__libc_free`](https://bbs.kanxue.com/elink@6e7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6T1L8h3W2F1L8%4u0Q4x3V1k6Y4L8r3W2T1j5#2)9J5c8X3u0D9L8$3u0Q4x3V1k6Y4L8r3W2T1j5#2)9J5k6o6u0Q4x3X3f1J5y4#2)9J5c8X3#2S2L8r3I4G2j5#2)9J5c8X3#2S2L8r3I4G2j5#2)9J5k6h3y4Q4x3U0y4x3x3K6l9^5y4b7`.`.)。

## `__libc_free`

函数原型：

```
void
__libc_free (void *mem)
```

`mem` 是要释放的内存位置。

如果 [`__free_hook`](https://bbs.kanxue.com/elink@951K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6T1L8h3W2F1L8%4u0Q4x3V1k6Y4L8r3W2T1j5#2)9J5c8X3u0D9L8$3u0Q4x3V1k6Y4L8r3W2T1j5#2)9J5k6o6u0Q4x3X3f1J5y4#2)9J5c8X3#2S2L8r3I4G2j5#2)9J5c8X3#2S2L8r3I4G2j5#2)9J5k6h3S2Q4x3U0y4x3x3e0b7K6) 位置不是 null，则调用 [`__free_hook`](https://bbs.kanxue.com/elink@5bcK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6T1L8h3W2F1L8%4u0Q4x3V1k6Y4L8r3W2T1j5#2)9J5c8X3u0D9L8$3u0Q4x3V1k6Y4L8r3W2T1j5#2)9J5k6o6u0Q4x3X3f1J5y4#2)9J5c8X3#2S2L8r3I4G2j5#2)9J5c8X3#2S2L8r3I4G2j5#2)9J5k6h3S2Q4x3U0y4x3x3e0b7K6) 返回。

调用 [`mem2chunk`](https://bbs.kanxue.com/elink@88cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6T1L8h3W2F1L8%4u0Q4x3V1k6Y4L8r3W2T1j5#2)9J5c8X3u0D9L8$3u0Q4x3V1k6Y4L8r3W2T1j5#2)9J5k6o6u0Q4x3X3f1J5y4#2)9J5c8X3#2S2L8r3I4G2j5#2)9J5c8X3#2S2L8r3I4G2j5#2)9J5k6h3y4Q4x3U0y4x3x3e0p5^5z5l9`.`.) 将 `mem` 转为 chunk。

如果 chunk 的 `IS_MMAPPED` 标记位为 1，表示为 map 申请的大内存或临时内存，需要立即释放。

否则调用 [`_int_free`](https://bbs.kanxue.com/elink@758K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6T1L8h3W2F1L8%4u0Q4x3V1k6Y4L8r3W2T1j5#2)9J5c8X3u0D9L8$3u0Q4x3V1k6Y4L8r3W2T1j5#2)9J5k6o6u0Q4x3X3f1J5y4#2)9J5c8X3#2S2L8r3I4G2j5#2)9J5c8X3#2S2L8r3I4G2j5#2)9J5k6h3y4Q4x3U0y4x3y4o6p5K6z5l9`.`.) 释放内存。

## `_int_free`

函数原型：

```
static void
_int_free (mstate av, mchunkptr p, int have_lock)
```

如果 chunk 满足存入 tcache 的条件，则将 chunk 存入 tcache 中，完成释放。

如果chunk 满足存入 fastbins 的条件，则将 chunk 存入 fastbins 中。

如果不能存入 fastbins，将 `IS_MMAPPED` 标记位为 1 的 chunk 释放归还给系统，否则对 chunk 物理连续的前后空闲 chunk 进行合并。

如果合并后的 chunk 没有到 top chunk 边界，就将 chunk 放入 unsorted bin 中，否则说明 top chunk 耗尽了直接该释放的 chunk 作为 top chunk。

最后对于经过合并操作后的 chunk 大小如果超过了 `FASTBIN_CONSOLIDATION_THRESHOLD` 阈值，则调用 [`malloc_consolidate`](https://bbs.kanxue.com/elink@5feK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6T1L8h3W2F1L8%4u0Q4x3V1k6Y4L8r3W2T1j5#2)9J5c8X3u0D9L8$3u0Q4x3V1k6Y4L8r3W2T1j5#2)9J5k6o6u0Q4x3X3f1J5y4#2)9J5c8X3#2S2L8r3I4G2j5#2)9J5c8X3#2S2L8r3I4G2j5#2)9J5k6h3y4Q4x3U0y4x3y4o6b7H3x3R3`.`.) 合并 fastbin chunk：
