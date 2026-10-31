
再假设一下，`chunk 1`溢出了0x10字节，那么不知`mchunk_prev_size`会被完全控制，而且`mchunk_size`也会被完全控制。

控制`chunk 2`的`mchunk_size`，说废话就是拥有了控制`chunk 2`大小的能力，比如让`chunk 2`直接包含`chunk 3`，乃至地址更高的chunk，如果`chunk 3`还是已分配的状态，那么就会被`chunk 2`强制拉到了未分配的区域。

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9 | `| AAAA ......        |`  `| mchunk_size` `3`      `|`  `| mchunk_prev_size` `3` `|` `-``> chunk` `3`  `| AAAA ......        |`  `| mchunk_size` `2`      `|`  `| mchunk_prev_size` `2` `|` `-``> chunk` `2`  `| AAAA ......        |`  `| mchunk_size` `1`      `|`  `| mchunk_prev_size` `1` `|` `-``> chunk` `1` |

## 可靠的覆写地址

如果想要完成利用获取Shell呢，有两个比较好的覆写点，一是`got`表项，但是`got`表项可能是只读的，二是`tls_dtor_list`和`exit_function`变量。

在常规的影响中，程序退出时会调用GLibC的`exit`函数进行退出，在常规情况下，退出函数并不会做什么特别的操作，但如果注册过特殊的函数那就不一样了。

最常见的特殊退出函数注册机制就是`atexit`机制，GLibC提供`atexit`接口，该接口接受函数指针作为参数，并来到`__internal_atexit`。

它先通过`__new_exitfn`分配新的`exit_function`，并将新`exit_function`插入`__exit_funcs`链表中。

回到`__internal_atexit`后，会给新的`exit_function`填充数据。

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8 | `static struct exit_function_list initial;`  `struct exit_function_list` `*``__exit_funcs` `=` `&initial;`    `atexit`  `-``> __cxa_atexit`  `-``> __internal_atexit`  `-``> new` `=` `__new_exitfn (__exit_funcs);`  `-``> new``-``>func.cxa.fn` `=` `(void (``*``) (void` `*``,` `int``)) func;` |

`atexit`机制还会利用`.fini_array`节，GlibC提供`__attribute__`关键字，当该关键字和`destructor`合作时，编译器会将函数注册到`.fini_array`节中。

程序在初始化启动的过程中，会进入`LIBC_START_MAIN`函数，该函数的作用是主动注册`call_fini`作退出函数，而`call_fini`的作用是遍历`.fini_array`节，并调用`.fini_array`节中存储的函数。

|  |  |
| --- | --- |
| 1  2  3  4  5 | `__attribute__ ((destructor))`  `void xxx (void) {}`  `-``> .fini_array [xxx, ...]`  `LIBC_START_MAIN`  `-``> __cxa_atexit (call_fini, NULL, NULL);` |

当程序退出进入`exit`函数时，它会先进入`__run_exit_handlers`的内部查找已经注册的退出函数，它先通过`__call_tls_dtors`函数遍历`tls_dtor_list`全局变量，最后再遍历`exit_function_list`全局变量。

|  |  |
| --- | --- |
| 1  2  3  4 | `exit`  `-``> __run_exit_handlers (status, &__exit_funcs, true, true);`  `-``> __call_tls_dtors`  `-``> exit_function_list` |

`tls_dtor_list`和`exit_function_list`都是全局变量，它们的区别是什么呢？

`exit_function_list`对整个进程有效，而`tls_dtor_list`只针对线程有效。

|  |  |
| --- | --- |
| 1  2  3 | `__cxa_thread_atexit_impl`  `-``> struct dtor_list` `*``new` `=` `calloc (``1``, sizeof (struct dtor_list))`  `-``> new``-``>func` `=` `func` |

除了`__cxa_atexit`，你其实还可以发现有针对C++产生的`__cxa_finalize`，它被提供给析构函数，在析构函数退出时执行。

显然，利用这些全局变量有一个最大的好处，就是它们不仅可读可写，而且当退出程序发生时，就一定会被执行。

## 残酷的现实

现实往往不是那么友好的，针对`unlink_chunk`的利用方法很早就存在了，GLibC也针对这些问题进行了加固。

`unlink_chunk`在进行向前合并时，会检查链表是否正确，这个检查只会出现在较新版本的GLibC中，所以这种利用方法已经失效了。

|  |  |
| --- | --- |
| 1  2  3  4  5 | `unlink_chunk`  `-``> fd` `=` `p``-``>fd`  `-``> bk` `=` `p``-``>bk`  `-``>` `if` `(__builtin_expect (fd``-``>bk !``=` `p || bk``-``>fd !``=` `p,` `0``))`  `-``> malloc_printerr (``"corrupted double-linked list"``);` |

除了这个检查之外，`_int_free_merge_chunk`函数在进入`unlink_chunk`之前，检查`chunk 2`保存的`mchunk_prev_size`与`chunk 1`保存的`mchunk_size`是否一致，如果不一致就说明有问题会抛出错误。

|  |  |
| --- | --- |
| 1  2  3  4  5  6 | `_int_free_merge_chunk`  `-``>` `if` `(!prev_inuse(p))`  `-``> p` `=` `chunk_at_offset(p,` `-``((``long``) prevsize));`  `-``>` `if` `(__glibc_unlikely (chunksize(p) !``=` `prevsize))`  `-``> malloc_printerr (``"corrupted size vs. prev_size while consolidating"``);`  `-``> unlink_chunk (av, p);` |

## 关于漏洞的反思

尽管上面的利用方法已经失效了，但是并不应该影响我们思考漏洞是如何产生的，调试方法是调试器也好，亦或者说依赖源代码与打印也罢，调试最基本的需求都是尽可能的建立系统、指令、源程序之间的关系，调试器以来DWARF和ELF定义还原信息，优点是利用操作获取结果的方式更加智能，而源代码与打印的方式，虽然语义信息是原生的，但是操作起来可能没有那么智能。

上面提到了漏洞，也提到了调试，它们之间又有什么关系呢？

调试方法帮助我们建立起低级指令与高级语义间的联系，但是这个东西就像一个二维平面一样，上面给出的示例中，合并操作与赋值语句都是极为常见的操作，但它们却在特定环境下出现了漏洞层面的联系，这种特定环境依赖于释放的时机与数据的构造。

以今天的眼光来看，理解这些漏洞与利用方式可能都不是什么困难的事情，因为我们可能只是承担了一个翻译机的角色，但是在我们真正去调试程序时，又应该在掌握高级语义的情况下，站在高维空间中洞察代码的数据结构中的漏洞呢？

## 检查绕过的初探

`mchunk_prev_size`控制着向后合并的范围，而`mchunk_size`则控制着向前合并的范围以及向后合并的可行性。

但是这两个数值是可以任意设置的吗？

- `mchunk_size`设置成任意值时，不管通过公式`p + mchunk_size`得到的新地址是不是`top chunk`，造成的影响都是相似的。

如果当前chunk意外合并高地址的未释放chunk时，这些chunk的结局有两个，一是直接并入`top chunk`，二是合并进入空闲链表中。

但程序肯定还不知道这件事情，它可以继续从被动释放的chunk中读写数据，传统的UAF都是主动释放但指针未置零，因此这种情况可以算作是一种另类的UAF。

除了UAF风险外，我们还有可能从`top chunk`或链表中申请到已存在的chunk，毕竟程序眼中是释放了，但GlibC眼中还是空闲的，空闲chunk自然也就能再被分配出去。

至于如何被利用，实际上还是要取决于程序对chunk上数据的操作。

- `mchunk_prev_size`设置成任意值时，影响的就是向后合并的范围，与向前合并时轻松不同，篡改`mchunk_prev_size`进行异常向后合并需要面对合并失败的情况。

a. `chunk A`向前合并相邻的`chunk B`。

首先面临的是chunk大小的匹配检查。

因为相邻，所以`chunk B`的`mchunk_size`一定是`chunk B`到`chunk A`的偏移值，这与`chunk A`的`mchunk_prev_size`是一致的。

chunk大小的匹配检查在这个时候不会发现问题。

合并相邻低地址的chunk是一句废话吗？因为好像本来也会合并。

这可能是废话，也可能不会是，当`chunk B`空闲时，`chunk A`的`mchunk_size`的标志位`P`会是0，这个时候当然本来就应该合并。

但如果`chunk B`不空闲，且`chunk A`的`mchunk_size`的标志位`P`被抹掉了，才会造成意外的合并。

b. `chunk A`向前合并非相邻的`chunk B`。

`chunk A`的`mchunk_prev_size`被篡改后，`chunk A`找到`chunk B`当然没有什么问题，但是能不能绕过大小匹配检查就是一个严重的问题了。

在理想中，`chunk A`的`mchunk_prev_size`和`chunk B`的`mchunk_size`，指的都是`chunk A`相对于`chunk B`的偏移值，但在合并非相邻chunk时，这两数值的大小显然不会相同。

这个时候chunk大小的匹配检查在这个时候会发现问题。

除非`chunk B`的`mchunk_size`也被我们控制了。

- 向前合并时有两个检查，一是chunk的大小匹配检查，二是链表指针的检查.

此时我们假设检查1已经绕过，但是检查2又应该怎么绕过呢？

a. 在GlibC的bins链表中，有一个很有意思的特性，那就是初始元素的数值，初始数值帮助链表形成双向链表的特性，可以帮助我们完成利用。

bins数组在正式使用前会被初始化，初始化是为了变成循环链表，这与`fast bin`采用单向链表的策略有所不同。

bins数组存放着很多链表，bins数组每两个元素组成一个链表，bins数组会存放三种类型的链表，它们分别是`unsorted bin`、`small bin`以及`large bin`。

bins数组中的`bins[0]`和`bins[1]`是留给`unsorted bin`使用的，其余部分留给剩下的两种类型使用。

链表中的`bins[(i - 1) * 2]`代表`fd`成员领衔的后进先出链表。

链表中的`bin[(i - 1) * 2 + 1]`代表`bk`成员领衔的先进先出链表。

arena初始化时，GlibC会将数组中的元素初始化成`bin = bin_at(M, i)`，该宏默认返回链表尾`&bins[(i - 1) * 2] - offset(fd)`。

这个链表尾提供了一个很好的性质。

即`bin->fd`为`bins[(i-1)*2]`，`bin->bk`为`bins[(i-1)*2 1]`，即初始数值可以检索到自身第`i`对链表，进而形成双向链表。

|  |  |
| --- | --- |
| 1  2 | `xxxx``-``>xx` `=` `A`  `-``>` `*``(xxxx` `+` `xx)` `=` `A` `-``> (xxxx` `+` `xx)` `=` `&A` `-``> xxxx` `=` `&A` `-` `xx` |

因此只要被向前合并的chunk是链表中唯一的chunk，那么就可以绕过链表检查。

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8 | `mchunkptr fd` `=` `p``-``>fd;` `-``> fd` `=` `&bins[(i` `-` `1``)` `*` `2``]` `-` `offset(fd)`  `mchunkptr bk` `=` `p``-``>bk;` `-``> bk` `=` `&bins[(i` `-` `1``)` `*` `2``]` `-` `offset(fd)`    `if` `(fd``-``>bk !``=` `p || bk``-``>fd !``=` `p)`  `malloc_printerr ......`    `fd``-``>bk !``=` `p || bk``-``>fd !``=` `p`  `-``> fd``-``>bk` `=` `bk``-``>fd` `=` `bins[(i` `+` `1``)` `*` `2` `+` `1``]` `=` `p` |

但这么做好像也不对啊，被向前合并的chunk若本来就是链表中唯一的chunk，那就代表它原本就是空闲的啊，上面提到的做法本来就是符合链表规则的。

所以也就不存在合并未释放的chunk一说了。

b. 由于假设被向前合并的chunk需要是尚未释放的，但被强行拉进了合并流程中，所以chunk中`fdxx`和`bkxx`肯定是还没有被赋值的，此时检查直接使用的就是chunk中的数据，这些可控的数据显然可以帮助我们完成利用。

由于我们可以控制`p + fd`和`p + bk`的值，那么`*(p + fd/bk)`结果就可以指向恶意地址，那么`p->fd`和`p->bk`就会落入我们设定好的目的地中。

更进一步的话`fd->bk`和`bk->fd`的结果也可以被我们控制。

只不过这种恶意数据的构造需要花上不小的经历。

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7 | `fd` `=` `p``-``>fd` `-``>` `*``(p` `+` `fd)`  `bk` `=` `p``-``>bk` `-``>` `*``(p` `+` `bk)`  `if` `(fd``-``>bk !``=` `p || bk``-``>fd !``=` `p)`  `......`    `(p``-``>fd)``-``>bk` `=` `*``(``*``(chunk_addr` `+` `0x10``)` `+` `0x18``)`  `(p``-``>bk)``-``>fd` `=` `*``(``*``(chunk_addr` `+` `0x18``)` `+` `0x10``)` |

## 常规检查绕过方法的悖论

- 先来看一下向前合并相邻chunk的情况，`chunk B`是被合并。

当`chunk A`触发`OffByOne`时，`chunk A`的`mchunk_prev_size`会被控制，我们可以定位到`chunk B`，这时chunk的大小匹配检查对我们是无效的。

但是进入`unlink_chunk`进行链表检查时，会面临不小的问题。

使用链表初始值特性进行绕过，`chunk B`就必须是链表中唯一的空闲chunk，这样的话就和`chunk B`未释放的前提有所冲突。

而构造`chunk B`上的数据需要面临地址泄露以及多处数据伪造的麻烦问题。

- 再看一下合并非相邻chunk的情况。

除非你既能控制`chunk B`的`mchunk_size`的数值，否则想要绕过chunk大小匹配检查基本就是空谈。

## 检查绕过的另外一种思路

我们假设这样的一种场景，当前有三个chunk，其中`chunk 1`与`chunk 2`相邻，且位于更低的地址，同时`chunk 1`申请了`0x38`的内存大小。

至于`chunk 3`它位于比`chunk 1`与`chunk 2`都更加低的地址上。

此时三个chunk都还没有被释放。

根据前面的学习我们可以知道，GLibC实际会给`chunk 1`分配0x40的内存的空间，其中还有0x10的空间给`mchunk_size`和`mchunk_prev_size`使用。

`chunk 1`的实际可用内存只有0x30，比预期少了0x8，具备触发`OffByOne`场景的条件，此刻我们通过`read`向缓冲区变量中写入0x38长度的数据，并在0x39处写入字符`\0`，`OffByOne`场景就此触发。

`chunk 1`除了负责触发`OffByOne`之外，还需要在0x38空间的前0x10空间，写入两个相同的特殊地址，这个特殊地址就是`chunk 3`的数据区起始地址。

最后我们还要让`chunk 1`想`chunk 2`的`mchunk_prev_size`写入新的偏移值，这个偏移值就是`chunk 2`到`chunk 3`数据区的距离。
