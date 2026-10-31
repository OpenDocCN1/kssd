
# [原创]Mimalloc分析

## 前言

在今年的HWS中遇到了一道堆，与往常不同的是，c语言标准库中的`malloc`分配器更换成了`mimalloc`，于是打算来了解一下这个分配器，查看一下这个分配器要如何利用。

同时由于本人水平不足，对于`mimalloc`源码的分析不到位，很多地方也是一知半解，只能起到一个面向pwn解题的分析。

个人认为相比起用`ptmalloc`锻炼源码分析能力，`mimalloc`更加轻量化，更能起到练手的一个效果。

## 编译环境配置

本小节用来指引如何在ubuntu中配置c语言调用`mimalloc`库，如果没有需要自己编译题目来调试的，就不需要看。

<a href="elink@cc5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6E0K9h3y4J5L8%4y4G2k6Y4c8Q4x3V1k6E0K9h3#2S2L8r3I4G2j5H3`.`.">Microsoft / Mimalloc：Mimalloc是一款紧凑的通用分配器，具有出色的性能。 (github.com)</a>

首先`git clone`库到本地

```bash
git clone https://github.com/microsoft/mimalloc.git
```

随后创建一个目录用来存放构建后的项目

```bash
cmake ../mimalloc/
make
sudo make install
```

接着是比较麻烦的一步，因为在执行完`make install`后，官方文档是说会在`/usr/include`中安装头文件，但是我实测是没有的，所以我们需要手动把头文件复制到对应目录中

```bash
sudo cp -r /home/chen/mimalloc/include/* /usr/include/
```

随后我们在使用`gcc`编译的时候，在后面加上`-lmimalloc`就可以成功编译了

```bash
gcc -o test ./test.c -lmimalloc
```

如果需要用到`pwndbg`进行源码调试的，需要编译`.so`文件的时候加上`-DCMAKE_BUILD_TYPE=Debug`，参照官方文档。

## 结构分析

对于每一个线程，都有对应的内存用来管理线程，我们称其为TLD。

TLD主要由两个部分组成`segment`和`heap`，我们先来介绍`segment`。

![image-20230720232504990](upload/attach/202308/983628_E6VYFFVTB79VA22.jpg)

这里的`page`就是实际分配给用户的内存，而第一个`page`的大小会小于其他`page`，是因为`segment`头部用来存放了当前`segment`的信息，占用了`page`的一部分空间。

![image-20230720232605058](upload/attach/202308/983628_HEHVMDVXKUHEU7V.jpg)

具体的成员我个人认为没有值得关注的，后面如果遇到了再说，这里先暂时记住`segment`的起始地址就是由于`mi_malloc`多分配的一块内存地址。

![image-20230727230229146](upload/attach/202308/983628_59G64J2S33XGS2D.jpg)

`heap`重点的成员有三个，前面两个是用于存放空闲的内存块。

`pages_free_direct`用于小于1024的内存块

`thread_delayed_free`是用于满页释放的，稍后会提及。

![image-20230721004423205](upload/attach/202308/983628_NCX5YTUCF93SFRD.jpg)

通过`pwndbg`直接观察，会发现实际上重要信息存放是位于`segment`，`heap`就起到了一个索引的功能。

索引到的结构我们称之为内存页，其主要的成员就四个。

![image-20230721010343554](upload/attach/202308/983628_7HRQR3AFMM6WJ3B.jpg)
![image-20230721004846559](upload/attach/202308/983628_2N6ZTST9QUMECDA.jpg)

红框圈起来的就是`free`链表，蓝框圈起来的是`local free`链表。

你会发现，和常规的`malloc`不同，并不是被申请过的内存块被释放后才会放入到链表中。

当我们申请一个内存块后，当前`page`剩下的会划分成内存块放入到`free`链表中。

当申请过的内存块释放后，会进入`local free`链表。

随后我们来观察一下实际分配给用户的内存空间。

![image-20230721010905578](upload/attach/202308/983628_HXN9F3H74ZYFFJN.jpg)

此时我申请的是`0x80`大小的内存空间，可以看到此时的`free`链表就已经成型了。

## 源码分析

### `mi_malloc`

我们来分析一下`mi_malloc`函数的源码

```c
extern inline void* _mi_heap_malloc_zero(mi_heap_t* heap, size_t size, bool zero) mi_attr_noexcept {
    return _mi_heap_malloc_zero_ex(heap, size, zero, 0);
}
mi_decl_nodiscard extern inline mi_decl_restrict void* mi_heap_malloc(mi_heap_t* heap, size_t size) mi_attr_noexcept {
    return _mi_heap_malloc_zero(heap, size, false);
}
mi_decl_nodiscard extern inline mi_decl_restrict void* mi_malloc(size_t size) mi_attr_noexcept {
    return mi_heap_malloc(mi_prim_get_default_heap(), size);
}
```

`mimalloc`通过多次跳转指向了`_mi_heap_malloc_zero_ex`函数，我们来分析一下这个函数的源码

```c
// The main allocation function
extern inline void* _mi_heap_malloc_zero_ex(mi_heap_t* heap, size_t size, bool zero, size_t huge_alignment) mi_attr_noexcept {
    if mi_likely(size <= MI_SMALL_SIZE_MAX) {
        mi_assert_internal(huge_alignment == 0);
        return mi_heap_malloc_small_zero(heap, size, zero);
    }
    else {
        mi_assert(heap != NULL);
        mi_assert(heap->thread_id == 0 || heap->thread_id == _mi_thread_id()); // heaps are thread local
        void* const p = _mi_malloc_generic(heap, size + MI_PADDING_SIZE, zero, huge_alignment); // note: size can overflow but it is detected in malloc_generic
        mi_track_malloc(p, size, zero);
#if MI_STAT > 1
        if (p != NULL) {
            if (!mi_heap_is_initialized(heap)) { heap = mi_prim_get_default_heap(); }
            mi_heap_stat_increase(heap, malloc, mi_usable_size(p));
        }
#endif
#if MI_DEBUG > 3
        if (p != NULL && zero) {
            mi_assert_expensive(mi_mem_is_zero(p, size));
        }
#endif
        return p;
    }
}
```

首先是对要申请`chunk`的大小进行了一个判断，如果小于`MI_SMALL_SIZE_MAX`就会跳转到`samll chunk`的申请，同时进行了一个断言检测。

`MI_SMALL_SIZE_MAX`的值定义在`mimalloc.h`中，可以看到是1024（64位的情况下，和指针字节大小有关系）。

```c
#define MI_SMALL_WSIZE_MAX (128)
#define MI_SMALL_SIZE_MAX (MI_SMALL_WSIZE_MAXsizeof(void))
```

我们先来分析大于1024的内存分配逻辑。

对于`heap`指针是否为空和线程id进行了检查。

随后跳转到`_mi_malloc_generic`函数中进行内存分配，`mi_track_malloc`函数是用来将内存块的信息存储在`track`跟踪器中，方便调试。

接着使用了条件编译语句，根据`MI_STAT`的值来决定是否记录更新`heap`的数据，用于调试。

`mi_heap_stat_increase`函数用于更新`heap`的统计数据，`mi_usable_size`函数用于计算内存块的实际大小（不是申请的大小）。

如果`MI_DEBUG`的值设置为3，那么就会调用`mi_assert_expensive`函数检测条件是否成立，这里的条件是调用`mi_mem_is_zero`来检测分配的内存块的前`size`个字节是否为0。`MI_DEBUG`的默认值为0，常规情况下我们并不用担心`fake_chunk`被检测出来。

### `_mi_malloc_generic`

随后我们进入`_mi_malloc_generic`函数，查看一下分析的主要逻辑

```c
void* _mi_malloc_generic(mi_heap_t* heap, size_t size, bool zero, size_t huge_alignment) mi_attr_noexcept
{
    mi_assert_internal(heap != NULL);
    // initialize if necessary
    if mi_unlikely(!mi_heap_is_initialized(heap)) {
        heap = mi_heap_get_default(); // calls mi_thread_init
        if mi_unlikely(!mi_heap_is_initialized(heap)) { return NULL; }
    }
    mi_assert_internal(mi_heap_is_initialized(heap));
    // call potential deferred free routines
    _mi_deferred_free(heap, false);
    // free delayed frees from other threads (but skip contended ones)
    _mi_heap_delayed_free_partial(heap);
    // find (or allocate) a page of the right size
    mi_page_t* page = mi_find_page(heap, size, huge_alignment);
    if mi_unlikely(page == NULL) { // first time out of memory, try to collect and retry the allocation once more
        mi_heap_collect(heap, true /* force */);
        page = mi_find_page(heap, size, huge_alignment);
    }
    if mi_unlikely(page == NULL) { // out of memory
        const size_t req_size = size - MI_PADDING_SIZE; // correct for padding_size in case of an overflow on `size`
        _mi_error_message(ENOMEM, "unable to allocate memory (%zu bytes)\n", req_size);
        return NULL;
    }
    mi_assert_internal(mi_page_immediate_available(page));
    mi_assert_internal(mi_page_block_size(page) >= size);
    // and try again, this time succeeding! (i.e. this should never recurse through _mi_page_malloc)
    if mi_unlikely(zero && page->xblock_size == 0) {
        // note: we cannot call _mi_page_malloc with zeroing for huge blocks; we zero it afterwards in that case.
        void* p = _mi_page_malloc(heap, page, size, false);
        mi_assert_internal(p != NULL);
        _mi_memzero_aligned(p, mi_page_usable_block_size(page));
        return p;
    }
    else {
        return _mi_page_malloc(heap, page, size, zero);
    }
}
```

最开始对于`heap`是否初始化了进行一个检测，如果没有初始化则进行初始化。

随后调用`_mi_deferred_free`将本线程所有标记释放的内存块加入到延迟释放列表中，随后批量释放，这一操作是为了提高性能。

调用`_mi_heap_delayed_free_partial`函数，释放其他线程已经标记释放的内存块，但跳过正在争用的内存块。

接着调用`mi_find_page`函数寻找可用的`page`，如果没有找到则将空闲的内存块回收后再次查找。

如果最后还是没有找到空闲的`page`，就说明空间不足，触发断言输出报错。

如果找到了可用的`page`，接着对于`page`的立即可用性进行检测，以及检测`page`的大小是否满足`size`的需求，这里的检测依赖的是`page`的`xblock`成员，其存储的是当前`page`中的内存块大小。

接着是性能优化的问题，通过`xblock`和`zero`参数来决定用哪种办法来清零内存块的内容。

### `_mi_page_malloc`
