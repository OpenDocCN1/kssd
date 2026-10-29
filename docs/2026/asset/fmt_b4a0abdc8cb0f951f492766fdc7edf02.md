
```
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28  29  30  31  32 | `/``*``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-` `Public wrappers.` `-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``*``/`    `void` `*`  `__libc_malloc (size_t bytes)`  `{`  `mstate ar_ptr;`  `void` `*``victim;`    `void` `*``(``*``hook) (size_t, const void` `*``)`  `=` `atomic_forced_read (__malloc_hook);`  `if` `(__builtin_expect (hook !``=` `NULL,` `0``))`  `return` `(``*``hook)(bytes, RETURN_ADDRESS (``0``));`    `arena_get (ar_ptr, bytes);`    `victim` `=` `_int_malloc (ar_ptr, bytes);`  `/``*` `Retry with another arena only` `if` `we were able to find a usable arena`  `before.` `*``/`  `if` `(!victim && ar_ptr !``=` `NULL)`  `{`  `LIBC_PROBE (memory_malloc_retry,` `1``, bytes);`  `ar_ptr` `=` `arena_get_retry (ar_ptr, bytes);`  `victim` `=` `_int_malloc (ar_ptr, bytes);`  `}`    `if` `(ar_ptr !``=` `NULL)`  `(void) mutex_unlock (&ar_ptr``-``>mutex);`    `assert` `(!victim || chunk_is_mmapped (mem2chunk (victim)) ||`  `ar_ptr` `=``=` `arena_for_chunk (mem2chunk (victim)));`  `return` `victim;`  `}` |
```

`malloc`实际上会直接调用这里的`__libc_malloc`函数，然后`__libc_malloc`也只不过是一层包装而已，实际上大部分的逻辑都是调用`_int_malloc`函数完成的，那么先来分析外面。

### 第一段

|  |  |

| --- | --- |

| 1  2  3  4 | `void` `*``(``*``hook) (size_t, const void` `*``)`  `=` `atomic_forced_read (__malloc_hook);`  `if` `(__builtin_expect (hook !``=` `NULL,` `0``))`  `return` `(``*``hook)(bytes, RETURN_ADDRESS (``0``));` |

定义了一个`hook`函数指针，如果`hook != NULL`则直接调用`hook`指向的内容。`hook`是为了方便开发者调试的一个东西，比如我自己写了一个`malloc`函数想测试它的性能如何，那么我在这里直接让`__malloc_hook = my_malloc`就可以直接调用我自己写的malloc函数了。但是同时它也是最容易被劫持的，刚开始我们很多题目都是靠写`__malloc_hook`为一个`onegadget`，然后调用`malloc`去`getshell`的。在`2.34`版本中，`__malloc_hook`同其它大部分的`hook`都被删除了。

### 第二段

|  |  |

| --- | --- |

| 1  2 | `arena_get (ar_ptr, bytes);`  `victim` `=` `_int_malloc (ar_ptr, bytes);` |

通过`arena_get`获得一个分配区，`arena_get`是个宏定义，定义在`arena.c`中，

|  |  |

| --- | --- |

| 1  2  3  4 | `#define arena_get(ptr, size) do { \`  `arena_lookup (ptr);                           \`  `arena_lock (ptr, size);                         \`  `}` `while` `(``0``)` |

`arena_lookup`定义如下，也是获取分配器指针。

|  |  |

| --- | --- |

| 1  2  3  4 | `#define arena_lookup(ptr) do { \`  `void` `*``vptr` `=` `NULL;                              \`  `ptr` `=` `(mstate) tsd_getspecific (arena_key, vptr);             \`  `}` `while` `(``0``)` |

然后加锁，没了，获取分配器指针这一段不是我们主要要分析的，也就不过多去解析了。

### 第三段

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13 | `/``*` `Retry with another arena only` `if` `we were able to find a usable arena`  `before.` `*``/`  `if` `(!victim && ar_ptr !``=` `NULL)`  `{`  `LIBC_PROBE (memory_malloc_retry,` `1``, bytes);`  `ar_ptr` `=` `arena_get_retry (ar_ptr, bytes);`  `victim` `=` `_int_malloc (ar_ptr, bytes);`  `}`  `if` `(ar_ptr !``=` `NULL)`  `(void) mutex_unlock (&ar_ptr``-``>mutex);`  `assert` `(!victim || chunk_is_mmapped (mem2chunk (victim)) ||`  `ar_ptr` `=``=` `arena_for_chunk (mem2chunk (victim)));`  `return` `victim;` |

它本身注释也写清楚了，在能够找到一个可用的`arena`之前尝试寻找另外一个`arena`，我这英文比较飘还请亲见谅。如果`arena`找到了但是`_int_malloc`居然返回0了，那么就重新寻找另一个分配器再次调用一次`_int_malloc`。完了之后呢，要给`arena`解锁，然后返回得到的`chunk`指针。

### `_int_malloc`源码分析

由于比较长，为了摆脱水长度的嫌疑就不给看总代码了，需要的自己找`glibc`的源码就好了，下面我一段一段分析。

### 第一段：main_arena初始化

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28  29  30 | `INTERNAL_SIZE_T nb;` `/``*` `normalized request size` `*``/`  `unsigned` `int` `idx;` `/``*` `associated` `bin` `index` `*``/`  `mbinptr` `bin``;` `/``*` `associated` `bin` `*``/`    `mchunkptr victim;` `/``*` `inspected``/``selected chunk` `*``/`  `INTERNAL_SIZE_T size;` `/``*` `its size` `*``/`  `int` `victim_index;` `/``*` `its` `bin` `index` `*``/`    `mchunkptr remainder;` `/``*` `remainder` `from` `a split` `*``/`  `unsigned` `long` `remainder_size;` `/``*` `its size` `*``/`    `unsigned` `int` `block;` `/``*` `bit` `map` `traverser` `*``/`  `unsigned` `int` `bit;` `/``*` `bit` `map` `traverser` `*``/`  `unsigned` `int` `map``;` `/``*` `current word of binmap` `*``/`    `mchunkptr fwd;` `/``*` `misc temp` `for` `linking` `*``/`  `mchunkptr bck;` `/``*` `misc temp` `for` `linking` `*``/`    `const char` `*``errstr` `=` `NULL;`  `checked_request2size (bytes, nb);`    `/``*` `There are no usable arenas.  Fall back to sysmalloc to get a chunk` `from`  `mmap.` `*``/`  `if` `(__glibc_unlikely (av` `=``=` `NULL))`  `{`  `void` `*``p` `=` `sysmalloc (nb, av);`  `if` `(p !``=` `NULL)`  `alloc_perturb (p, bytes);`  `return` `p;`  `}` |

变量定义就不用看了，源码中也都标注出来了，这里最主要就是把用户请求的`bytes`转换成最小能满足的`chunk size`，然后它的变量名应该是`nb`，这个`nb`应该是`nbytes`的缩写。

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13 | `#define request2size(req)                                         \`  `(((req)` `+` `SIZE_SZ` `+` `MALLOC_ALIGN_MASK < MINSIZE)  ?             \`  `MINSIZE :                                                      \`  `((req)` `+` `SIZE_SZ` `+` `MALLOC_ALIGN_MASK) & ~MALLOC_ALIGN_MASK)`    `/``*`  `Same,` `except` `also perform argument check` `*``/`    `#define checked_request2size(req, sz)                             \`  `if` `(REQUEST_OUT_OF_RANGE (req)) {                          \`  `__set_errno (ENOMEM);                              \`  `return` `0``;                                      \`  `}                                          \`  `(sz)` `=` `request2size (req);` |

这里原来也给注释了，这俩宏定义就是一样的，只不过做一个参数check。

这里还要注意一下那些宏定义：

|  |  |

| --- | --- |

| 1  2  3 | `__glibc_unlikely(exp)表示exp很可能为假。`  `__glibc_likely(exp)表示exp很可能为真。`  `__builtin_expect(exp,value)表示exp``=``=``value大概率成立` |

这三个宏定义在源码中经常能看到，其实它不会改编程序逻辑，只是告诉编译器这个很可能为某个值，就把否的情况作为跳转，真的情况就顺序运行下去，减少程序的跳转，一定程度上可以优化程序运行速度。或者还有一个简单粗暴的办法，你把这三个字符全都去了，不影响代码逻辑。

那么这一段的逻辑就是，如果在分配的时候`arena`为空，那就调用`sys_malloc`系统调用去请求一个`chunk`，然后`memset`这个`chunk`的数据段。

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8 | `/``*` `-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-` `Testing support` `-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``-``*``/`    `static` `int` `perturb_byte;`  `static void alloc_perturb (char` `*``p, size_t n)`  `{`  `if` `(__glibc_unlikely (perturb_byte))`  `memset (p, perturb_byte ^` `0xff``, n);`  `}` |

通常情况下`perturb_byte`为假，差不多意思就是如果你没有特殊设置，那么`data`段全为0字节，实际情况也确实是这样的。

### 第二段：fast bin的处理

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28  29  30  31  32  33 | `#define get_max_fast() global_max_fast`  `#define fastbin_index(sz) \`  `((((unsigned` `int``) (sz)) >> (SIZE_SZ` `=``=` `8` `?` `4` `:` `3``))` `-` `2``)`  `#define fastbin(ar_ptr, idx) ((ar_ptr)->fastbinsY[idx])`    `if` `((unsigned` `long``) (nb) <``=` `(unsigned` `long``) (get_max_fast ()))`  `{`  `idx` `=` `fastbin_index (nb);`  `mfastbinptr` `*``fb` `=` `&fastbin (av, idx);`  `mchunkptr pp` `=` `*``fb;`  `do`  `{`  `victim` `=` `pp;`  `if` `(victim` `=``=` `NULL)`  `break``;`  `}`  `while` `((pp` `=` `catomic_compare_and_exchange_val_acq (fb, victim``-``>fd, victim))`  `!``=` `victim);`  `if` `(victim !``=` `0``)`  `{`  `if` `(__builtin_expect (fastbin_index (chunksize (victim)) !``=` `idx,` `0``))``/``/``在malloc的时候检查了fastbin的size发现不对`  `{`  `errstr` `=` `"malloc(): memory corruption (fast)"``;`  `errout:`  `malloc_printerr (check_action, errstr, chunk2mem (victim), av);`  `return` `NULL;`  `}`  `check_remalloced_chunk (av, victim, nb);`  `void` `*``p` `=` `chunk2mem (victim);`  `alloc_perturb (p, bytes);`  `return` `p;`  `}`  `}` |

这里嘛就会判断，你申请的这个`nb`是否`<=global_max_fast`，如果成立那么就会先在`fast bin`中寻找能满足的`chunk`，并且一定是完全匹配。它先找到`av->fastbinY[idx]`观察是否为0，如果不为0则说明该`size`的`fast bin`有`chunk`，那么就做以下动作：

取出`av->fastbinY[idx]`给`victim`

链表中删除这个`victim`，然后重新接回去。

中间有一个`check`，就是判断所给`chunk`的`fastbinY`链上的`size`是否＝我需要的`size`，如果不相等那么直接报错退出。
