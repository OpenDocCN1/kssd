
虽说是php的内存管理，但是实际上是其内部zend引擎的内存管理机制。PHP采取“预分配方案”，提前向操作系统申请一个chunk（2M，利用到hugepage特性），并且将这2M内存切割为不同规格（大小）的若干内存块，当程序申请内存时，直接查找现有的空闲内存块即可；

PHP将内存分配请求分为3种情况：

huge内存：针对大于2M-4K的分配请求，直接调用mmap分配；

large内存：针对小于2M-4K，大于3K的分配请求，在chunk上查找满足条件的若干个连续page；

small内存：针对小于3K的分配请求；PHP拿出若干个页切割为8字节大小的内存块，拿出若干个页切割为16字节大小的内存块，24字节，32字节等等，将其组织成若干个空闲链表；每当有分配请求时，只在对应的空闲链表获取一个内存块即可；

## 相关结构体

在large和small两类chunk的第一个page里，会存储chunk的控制信息，这个结构体是`_zend_mm_chunk`，所有的chunk会形成一个双向链表，`zend_mm_page_map`利用位图记录512个page的使用情况，0代表空闲，1代表已经分配。`zend_mm_page_info`通过`uint32_t`存储FLAG信息，

```c
struct _zend_mm_chunk {
    zend_mm_heap *heap;
    zend_mm_chunk *next;
    zend_mm_chunk *prev;
    uint32_t free_pages; /* number of free pages */
    uint32_t free_tail; /* number of free pages at the end of chunk */
    uint32_t num;
    char reserve[64 - (sizeof(void*) * 3 + sizeof(uint32_t) * 3)];
    zend_mm_heap heap_slot; /* used only in main chunk */
    zend_mm_page_map free_map; /* 512 bits or 64 bytes */
    zend_mm_page_info map[ZEND_MM_PAGES]; /* 2 KB = 512 * 4 */
};
```

然后是[`_zend_mm_heap`](<elink@f90K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6H3K9s2m8Q4x3V1k6H3K9s2m8Q4x3X3c8K6M7X3y4Q4x3V1k6T1L8r3!0T1i4K6u0r3y4K6R3%4k6U0t1$3j5K6R3^5x3U0N6T1j5h3p5@1k6X3p5$3k6h3b7#2y4e0c8X3j5e0b7#2y4U0t1%4y4e0m8U0j5K6R3$3k6o6q4S2y4W2)9J5c8W2A6W2L8X3c8Q4x3V1k6*7k6h3&6V1i4K6g2X3j5h3I4D9L8$3y4Q4x3X3g2U0i4K6t1K6e0o6t1$3y4R3.>)，是chunk的上级管理结构，存储与堆分配相关的全局信息：

```c
struct _zend_mm_heap {
#if ZEND_MM_CUSTOM
    int use_custom_heap;
#endif
#if ZEND_MM_STORAGE
    zend_mm_storage *storage;
#endif
#if ZEND_MM_STAT
    size_t size; /* current memory usage */
    size_t peak; /* peak memory usage */
#endif
    uintptr_t shadow_key; /* free slot shadow ptr xor key */
    zend_mm_free_slot *free_slot[ZEND_MM_BINS]; /* free lists for small sizes */
#if ZEND_MM_STAT || ZEND_MM_LIMIT
    size_t real_size; /* current size of allocated pages */
#endif
#if ZEND_MM_STAT
    size_t real_peak; /* peak size of allocated pages */
#endif
#if ZEND_MM_LIMIT
    size_t limit; /* memory limit */
    int overflow; /* memory overflow flag */
#endif
    zend_mm_huge_list *huge_list; /* list of huge allocated blocks */
    zend_mm_chunk *main_chunk;
    zend_mm_chunk *cached_chunks; /* list of unused chunks */
    int chunks_count; /* number of allocated chunks */
    int peak_chunks_count; /* peak number of allocated chunks for current request */
    int cached_chunks_count; /* number of cached chunks */
    double avg_chunks_count; /* average number of chunks allocated per request */
    int last_chunks_delete_boundary; /* number of chunks after last deletion */
    int last_chunks_delete_count; /* number of deletion over the last boundary */
#if ZEND_MM_CUSTOM
    struct {
        void *(*_malloc)(size_t ZEND_FILE_LINE_DC ZEND_FILE_LINE_ORIG_DC);
        void (*_free)(void* ZEND_FILE_LINE_DC ZEND_FILE_LINE_ORIG_DC);
        void *(*_realloc)(void*, size_t ZEND_FILE_LINE_DC ZEND_FILE_LINE_ORIG_DC);
        size_t (*_gc)(void);
        void (*_shutdown)(bool full, bool silent);
    } custom_heap;
    union {
        HashTable *tracked_allocs;
        struct {
            bool poison_alloc;
            uint8_t poison_alloc_value;
            bool poison_free;
            uint8_t poison_free_value;
            uint8_t padding;
            bool check_freelists_on_shutdown;
        } debug;
    };
#endif
    pid_t pid;
    zend_random_bytes_insecure_state rand_state;
};
```

如果不方便看的话可以直接看gdb的结果：

![image](upload/attach/202504/950404_QQ5EXGUDVTQD492.png)

堆的最上层结构体是封装了`zend_mm_heap`的`zend_alloc_globals`：

```c
typedef struct _zend_alloc_globals {
    zend_mm_heap *mm_heap;
} zend_alloc_globals;
```

`alloc_globals`是类似于glibc中`main_arena`的变量，通过它即可逐步获取整个堆：

```c
static zend_alloc_globals alloc_globals;
```

## small内存

这里只介绍small类型的内存分配，而这也是与我们攻击直接相关的部分。简单来说，small类型内存的空闲链表类似于2.27下的`tcache`空闲链表，也是单链表形式，并且没有任何保护，因此只需要修改链表中任一节点，即可劫持`free`的空闲链表。它的结构类似于：

![image](upload/attach/202504/950404_XCEKUMGCU53VVVP.png)

## 源码分析

下面从源码来分析一下，当申请small类型heap时：

```c
static zend_always_inline void *zend_mm_alloc_small(zend_mm_heap *heap, int bin_num ZEND_FILE_LINE_DC ZEND_FILE_LINE_ORIG_DC)
{
#if ZEND_MM_STAT
    do {
        size_t size = heap->size + bin_data_size[bin_num];
        size_t peak = MAX(heap->peak, size);
        heap->size = size;
        heap->peak = peak;
    } while (0);
#endif
    if (EXPECTED(heap->free_slot[bin_num] != NULL)) {
        zend_mm_free_slot *p = heap->free_slot[bin_num];
        heap->free_slot[bin_num] = p->next_free_slot;
        return p;
    } else {
        return zend_mm_alloc_small_slow(heap, bin_num ZEND_FILE_LINE_RELAY_CC ZEND_FILE_LINE_ORIG_RELAY_CC);
    }
}
```

如果`free_slot`资源不够，则会调用`zend_mm_alloc_small_slow`创建一个对应大小的`free_slot`：

```c
static zend_never_inline void *zend_mm_alloc_small_slow(zend_mm_heap *heap, uint32_t bin_num ZEND_FILE_LINE_DC ZEND_FILE_LINE_ORIG_DC)
{
    zend_mm_chunk *chunk;
    int page_num;
    zend_mm_bin *bin;
    zend_mm_free_slot *p, *end;
#if ZEND_DEBUG
    bin = (zend_mm_bin*)zend_mm_alloc_pages(heap, bin_pages[bin_num], bin_data_size[bin_num] ZEND_FILE_LINE_RELAY_CC ZEND_FILE_LINE_ORIG_RELAY_CC);
#else
    bin = (zend_mm_bin*)zend_mm_alloc_pages(heap, bin_pages[bin_num] ZEND_FILE_LINE_RELAY_CC ZEND_FILE_LINE_ORIG_RELAY_CC);
#endif
    if (UNEXPECTED(bin == NULL)) {
        /* insufficient memory */
        return NULL;
    }
    chunk = (zend_mm_chunk*)ZEND_MM_ALIGNED_BASE(bin, ZEND_MM_CHUNK_SIZE);
    page_num = ZEND_MM_ALIGNED_OFFSET(bin, ZEND_MM_CHUNK_SIZE) / ZEND_MM_PAGE_SIZE;
    chunk->map[page_num] = ZEND_MM_SRUN(bin_num);
    if (bin_pages[bin_num] > 1) {
        uint32_t i = 1;
        do {
            chunk->map[page_num+i] = ZEND_MM_NRUN(bin_num, i);
            i++;
        } while (i < bin_pages[bin_num]);
    }
    /* create a linked list of elements from 1 to last */
    end = (zend_mm_free_slot*)((char*)bin + (bin_data_size[bin_num] * (bin_elements[bin_num] - 1)));
    heap->free_slot[bin_num] = p = (zend_mm_free_slot*)((char*)bin + bin_data_size[bin_num]);
    do {
        zend_mm_set_next_free_slot(heap, bin_num, p, (zend_mm_free_slot*)((char*)p + bin_data_size[bin_num]));
#if ZEND_DEBUG
        do {
            zend_mm_debug_info *dbg = (zend_mm_debug_info*)((char*)p + bin_data_size[bin_num] - ZEND_MM_ALIGNED_SIZE(sizeof(zend_mm_debug_info)));
            dbg->size = 0;
        } while (0);
#endif
        p = (zend_mm_free_slot*)((char*)p + bin_data_size[bin_num]);
    } while (p != end);
    /* terminate list using NULL */
    p->next_free_slot = NULL;
#if ZEND_DEBUG
    do {
        zend_mm_debug_info *dbg = (zend_mm_debug_info*)((char*)p + bin_data_size[bin_num] - ZEND_MM_ALIGNED_SIZE(sizeof(zend_mm_debug_info)));
        dbg->size = 0;
    } while (0);
#endif
    /* return first element */
    return bin;
}
```

释放时，直接将free的small heap链入末尾：

```c
static zend_always_inline void zend_mm_free_small(zend_mm_heap *heap, void *ptr, int bin_num)
{
    ZEND_ASSERT(bin_data_size[bin_num] >= ZEND_MM_MIN_USEABLE_BIN_SIZE);
    zend_mm_free_slot *p;
#if ZEND_MM_STAT
    heap->size -= bin_data_size[bin_num];
#endif
#if ZEND_DEBUG
    do {
        zend_mm_debug_info *dbg = (zend_mm_debug_info*)((char*)ptr + bin_data_size[bin_num] - ZEND_MM_ALIGNED_SIZE(sizeof(zend_mm_debug_info)));
        dbg->size = 0;
    } while (0);
#endif
    p = (zend_mm_free_slot*)ptr;
    zend_mm_set_next_free_slot(heap, bin_num, p, heap->free_slot[bin_num]);
    heap->free_slot[bin_num] = p;
}
```

## php堆调试

网上没有搜到比较合适的，自己写了个[phpgdb](<elink@41bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6s2k6h3g2C8b7$3#2G2M7X3g2Q4x3V1k6H3K9s2m8Y4k6r3t1.>)，目前支持4个命令。

### `pstart`

运行到php加载完所有拓展之后，此时可以设置断点。

```sh
gdb> pstart
...
gdb> b zif_some_mod_func
```

### `pheap`

查看最上层的堆信息：

![image](upload/attach/202504/950404_D5VVCNWXSR4UNAJ.png)

### `psmall`

查看small slot链表：

![image](upload/attach/202504/950404_Z5NAXCQBFHM4H53.png)

### `pelement`

查看给定地址所属于的element（最终分配的堆块）
