
进入`/path/to/xxx`目录，找到`xxx.json`。

在表达式`cvar = avar + bvar`中，加法运算符（`+`）将`avar`与`bvar`相加，得到它们的和`cvar`

在`List.of(arg0, arg1, arg2)`中，`List`接口的工厂方法`of()`接受一系列的元素，返回包含它们的只读列表。

之后我们这样调用`cmd`命令：`cmd arg0 arg1 arg2`。

```
if (condVar > someVal) {console.log("xxx")}
```
[/content]

Wait user asks to process the given text after. The example is just example. Need to output processed text for given content.

We need to produce final output with [content]...[/content] containing processed markdown.

Let's do it.

[content]
- `_mapcount`：表示当前page frame被map的次数（被页表引用的次数）

- `lru`：根据page frame的活跃程度（使用频率），将page frame挂在不同的list上，作为页面回收的依据。

- `_refcount`：引用计数。

  本变量不可直接使用，而是要通过`include/linux/page_ref.h`对应的函数来原子的读写。

- `pgoff_t index`：表示在文件映射时的该page在文件内的offset，单位是page的大小。

- `mapping`，我们主要说最常见的两种情况：

  - 如果当前的page是一个匿名页，`page->mapping`指向它的`anon_vma`。并设置`PAGE_MAPPING_ANON`位来区分它。

  - 如果当前的page是一个非匿名页，也就是说与某个文件相关联。那么`mapping`指向文件inode的地址空间。

  - 根据是否处于`VM_MERGEABLE`区域，是否开启`CONFIG_KSM`，此指针指向的位置仍有不同。

    详见：`/include/linux/page-flags.h`

在了解了本部分知识后，可以阅读：

[文件系统cache与匿名页交换](https://bbs.kanxue.com/elink@0ccK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8X3A6S2M7$3!0F1j5$3S2W2L8W2)9#2k6X3N6T1k6q4)9J5c8X3q4J5N6r3W2U0L8r3g2Q4x3V1k6V1k6i4c8S2K9h3I4K6i4K6u0r3y4K6V1@1y4U0t1H3x3e0b7`.

加深对应的理解。

|  |  |
| --- | --- |
| 1  2  3  4 | `root@ubuntu:~``# free`  `total        used        free      shared  buff``/``cache   available`  `Mem:` `4012836`      `207344`     `3317312`        `1128`      `488180`     `3499580`  `Swap:` `998396`           `0`      `998396` |

# 页表组织

本部分我们主要目光放在x86-64下的4级页表的组织。

即：`PGD` -> `PUD` -> `PMD` -> `PTE`

| PGD | 页全局目录(Page Global Directory) |
| --- | --- |
| PUD | 页上级目录(Page Upper Directory) |
| PMD | 页中间目录(Page Middle Directory) |
| PTE | 页表(Page Table) |

一个比较好的说明图片：

[![hYeUYR.png](https://bbs.kanxue.com/upload/attach/202109/876323_7KQ8NJX66JDSW38.jpg)](https://bbs.kanxue.com/elink@97cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6Z5h3h3g2g2h3g2t1`.)

## 一次页表查询

当我们给出一个`virtual addr`(`aka. v_addr`)，我们需要通过页表机制，来获取其对应的物理地址（`aka. p_addr`）。接下来梳理一下从 `v_addr` -> `p_addr`的过程。

1. 首先从`CR3`寄存器中获取`PML4T`（页目录）的物理页面的基地址。

2. 与`v_addr`的对应bits组合，相加得到页目录项的物理地址。

3. 读取对应的目录项（`pgd_t`），从中取出`PUD`的物理基地址。

4. 与`v_addr`对应的bits相加，得到对应的`PUD`目录项的物理地址。

5. 读取`pud_t`，获得`PMD`的基地址。

6. 与`v_addr`对应bits相加，定位到对应的页中间目录项物理地址。

7. 取出`pmd_t`即对应的页表物理基地址。与`v_addr`对应bits相加获取页表项对应的物理地址（`pte`）

8. 读取`pte_t`，从中得到真正的物理内存页基地址。

9. 从`v_addr`最后一部分取出物理页内偏移量，相加得到对应的最终的物理地址。

10. 访问真正的物理内存地址的数据。

需要注意的是，每个进程都拥有自己的`PGD`。它是一个物理页，并包含一个`pgd_t`数组。即每个进程都有一套自身的页表。

|  |  |
| --- | --- |
| 1 | `task_struct` `-``> mm_struct` `-``> pgd_t` `*` `pgd` |

当发生进程切换时，切换的是进程页表：即将新进程的`pgd`（页目录）加载到`CR3`寄存器中。

## KPTI与内核页表

如果有熟悉Kernel Pwn，在漏洞利用中有一种缓解技术叫做`KPTI`（Kernel page-table isolation）即内核页表隔离。当题目开启了内核页表隔离时，不能直接着陆到用户态。

`KPTI`的核心是，在开启了这个选项的程序中，每个进程拥有两套页表，分别是内核态页表（只能在内核态访问）与用户态页表，他们处于不同的地址空间下。

当发生一次`syscall`时，涉及到用户态与内核态页表的切换。（切换`CR3`）

[![hYeaf1.png](https://bbs.kanxue.com/upload/attach/202109/876323_Q8NNV3EQ4FDKKH6.jpg)](https://bbs.kanxue.com/elink@198K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6Z5h3h3g2S2k6U0p5`.)

而如果我们着陆用户态（`ireq/sysret`）的时候，没有正常切换/设置`CR3`寄存器，就会导致页表错误，最后引发段错误。

在`bypass`的时候，我们往往通过`SWITCH_USER_CR3`：

|  |  |
| --- | --- |
| 1  2  3 | `mov     rdi, cr3`  `or`      `rdi,` `1000h`  `mov     cr3, rdi` |

来重新设置`cr3`寄存器。

或者是通过 `swapgs_restore_regs_and_return_to_usermode` 函数返回。

知道了`KPTI`之后的页表，那么显而易见，当我们没有开启`KPTI`时，只有进程的页表是时刻在更新，而内核页表全局只有一份，所有进程共享内核页表。而每个进程的“进程页表”中内核态地址相关的页表项都是“内核页表”的一个拷贝。当我们想要索引内核页表时，可以通过：`init_mm.pgd`

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14 | `struct mm_struct init_mm` `=` `{`  `.mm_rb` `=` `RB_ROOT,`  `.pgd` `=` `swapper_pg_dir,`  `.mm_users` `=` `ATOMIC_INIT(``2``),`  `.mm_count` `=` `ATOMIC_INIT(``1``),`  `.write_protect_seq` `=` `SEQCNT_ZERO(init_mm.write_protect_seq),`  `MMAP_LOCK_INITIALIZER(init_mm)`  `.page_table_lock` `=`  `__SPIN_LOCK_UNLOCKED(init_mm.page_table_lock),`  `.arg_lock` `=`  `__SPIN_LOCK_UNLOCKED(init_mm.arg_lock),`  `.mmlist` `=` `LIST_HEAD_INIT(init_mm.mmlist),`  `.user_ns` `=` `&init_user_ns,`  `.cpu_bitmap` `=` `CPU_BITS_NONE,`  `INIT_MM_CONTEXT(init_mm)`  `};` |

而这个 `swapper_pg_dir` 本质上就是内核`PGD`的基地址。

|  |  |
| --- | --- |
| 1  2  3  4  5 | `/*`  `* Initialized during boot,` `and` `readonly` `for` `initializing page tables`  `* afterwards`  `*/`  `pgd_t swapper_pg_dir[PTRS_PER_PGD];` |

关于内核页表的创建过程可以看：

491K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6J5K9h3y4Z5j5i4u0V1N6$3g2A6P5h3q4F1k6#2)9J5k6o6u0Q4x3X3g2Y4K9i4c8T1L8$3!0C8i4K6u0W2K9h3!0Q4x3V1k6C8k6i4u0F1k6h3I4Q4x3X3c8W2P5s2m8D9L8%4u0A6L8X3N6Q4x3V1j5H3x3q4)9J5k6r3g2$3L8$3I4#2N6r3W2G2L8W2)9#2k6X3!0X3i4K6g2X3K9$3g2J5L8X3g2D9i4K6g2X3M7r3q4Y4k6i4c8S2j5X3I4W2

## TLB缓存

`TLB`是`translation lookaside buffer`的简称。其本质上就是一块高速缓存。记得之前在计算机体系结构课上学过：全相联映射、组相连映射、直接映射等。

在正常情况下，我们通过四级页表查询来做页表转换来进行虚拟地址到物理地址的转换。

而`TLB`提供了一种更高速的方式来做虚拟地址到物理地址的转换。

> `TLB`是一个小的，虚拟寻址的缓存，其中每一行都保存着一个由单个`PTE(Page Table Entry,页表项)`组成的块。如果没有`TLB`，则每次取数据都需要两次访问内存，即查页表获得物理地址和取数据。

不同的映射方式的`cache`有不同的组织形式，但是其整体思想都是通过虚拟地址来查`cache`，如果`TLB cache`命中，则直接可以得到物理地址。

`TLB`包含最近使用过的页表条目。给定一个虚拟地址，处理器将检查`TLB`是否存在页表条目（`TLB`命中），检索帧号并形成实际地址。如果在`TLB`中找不到页表条目（`TLB`丢失），则页号用于索引过程页表。`TLB`首先检查页面是否已在主存储器中，如果不在主存储器中，则发出页面错误，然后更新`TLB`以包括新的页面条目。

[![hYej10.png](https://bbs.kanxue.com/upload/attach/202109/876323_RFC8GCNS43WE3B7.jpg)](https://bbs.kanxue.com/elink@f14K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6@1N6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6Z5h3h3g2B7x3e0l9`.)

从这张图可以清晰的看出来，`TLB`提供了一种从`v_addr[12:47]` 到 `p_addr[12:47]`的映射。（低12bits均相似，所以不用管）

而`ASID`主要是为了区分不同的进程。

## page cache 页缓冲

首先明确一点。`page cache`是Linux内核使用的主要磁盘缓存。

> `page cache is the main disk cache used by the Linux kernel.`

我们一般在异步情况下读写文件时，首先写入对应的`page cache`，此时`pages`变成`dirty pages` ，后续会有内核线程`pdflush`真正写回到硬盘上。相对而言的，当我们读文件时，也是先放入`page cache`，然后再拷贝给用户态。当我们再次读同一个文件，如果`page cache`里已经有了，那么其性能就会有很大提升。

## Inverted page tables（IPT）

倒排页表，顾名思义，其储存的是每个物理`page frame`的信息。

其出现是为了缓解多级页表占用的内存问题。倒排页表项与**物理内存页框**有一一对应关系，而不是每一个虚拟页面有一个表项。

它所包含的表项数量较少（物理内存大小一般远小于虚拟内存大小）。所以其使用页框号而不是虚拟页号来索引页表项。

虽然`IPT`的设计节省了大量空间，但是也导致从虚拟地址到物理地址的转换会变得很困难。当进程`n`访问虚拟页面`p`时，硬件不再能通过把`p`当作指向页表的一个索引来查找物理页框。取而代之的是，它必须搜索整个倒排页表来查找某一个表项。

所以相比来说，`TLB`则是更好的一种技术。

## Huge page

`huge page`也称作大页，巨页。

我们一般来说一个页表项是4k，这就产生了一个问题：当物理内存很大时，页表会变得非常大，占用大量物理内存。而大页则是使页变大，由于页变大了，需要的页表项也就小了，占用物理内存也减少了。

`x64`四级页表系统支持2MB的大页，1GB的大页。

其优点主要是可以减少页表项，加快检索速度，提高`TLB hit`概率。

当我们打开`CR4`的`pse`位时（page size extension）就开启了对应的大页。

但是缺点是需要预先分配；如果分配过多，会造成内存浪费，不能被其他程序使用

### THP(transparent huge page)

`THP`（transparent huge page）即透明大页，他是对`Huge Page`的一个优化，它允许大页做动态的分配。`THP`减小了针对`huge page`支持的开销。使得应用程序可以根据需要灵活地选择虚存页面大小，而不会被强制使用 2MB 大页面。

* `THP`是通过将巨大的页面分解成较小的4KB页面来实现的，然后这些页面被正常地交换出去。但是为了有效地使用`hugepages`，内核必须找到物理上连续的内存区域，其大小足以满足请求，而且还要正确对齐。为此，我们增加了一个`khugepaged`内核线程。这个线程会偶尔尝试用`hugepage`分配来替代目前正在使用的较小的页面，从而最大限度地提高`THP`的使用率。在用户区，不需要对应用程序进行修改（因此是透明的）。但有一些方法可以优化其使用。对于想要使用`hugepages`的应用程序，使用`posix_memalign()`也可以帮助确保大的分配被对齐到巨大的页面（2MB）边界上。另外，`THP`只对匿名内存区域启用。*

但是问题是由于其动态分配的性质，以及繁琐的内存锁操作，`THP`很可能会导致性能上的下降。

## 页表标识位

c2cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6*7K9s2g2S2L8X3I4S2L8W2)9J5k6i4A6Z5K9h3S2#2i4K6u0W2j5$3!0E0i4K6u0r3M7q4)9J5c8U0j5%4x3o6f1K6x3U0p5H3

**P**（Present） - 为1表明该`page`存在于当前物理内存中，为0则`PTE`的其他部分都失去意义了，不用看了，直接触发`page fault`。`P`位为0的`PTE`也不会有对应的`TLB entry`，因为早在`P`位由1变为0的时候，对应的`TLB`就已经被`flush`掉了。

**G** （Global）- 用于`context switch`的时候不用`flush`掉`kernel`对应的`TLB`，所以这个标志位在`TLB entry`中也是存在的。

**A**（Access） - 当这个`page`被访问（读/写）过后，硬件将该位置1，`TLB`只会缓存`access`的值为1的`page`对应的映射关系。软件可将该位置0，然后对应的`TLB`将会被`flush`掉。这样，软件可以统计出每个`page`被访问的次数，作为内存不足时，判断该`page`是否应该被回收的参考。

**D** （Dirty）- 这个标志位只对`file backed`的`page`有意义，对`anonymous`的`page`是没有意义的。当`page`被写入后，硬件将该位置1，表明该`page`的内容比外部`disk/flash`对应部分要新，当系统内存不足，要将该`page`回收的时候，需首先将其内容`flush`到外部存储。之后软件将该标志位清0。

R/W和U/S属于权限控制类：

**R/W**（Read/Write） - 置为1表示该`page`是`writable`的，置为0则是`readonly`，对只读的`page`进行写操作会触发`page fault`。

**U/S**（User/Supervisor） - 置为0表示只有`supervisor`（比如操作系统中的`kernel`）才可访问该`page`，置为1表示`user`也可以访问。

PCD和PWT和cache属性相关：

**PCD**（Page Cache Disabled）- 置为1表示disable，即该`page`中的内容是不可以被`cache`的。如果置为0（enable），还要看`CR0`寄存器中的`CD`位这个总控开关是否也是0。

**PWT** （Page Write Through）- 置为1表示该`page`对应的`cache`部分采用`write through`的方式，否则采用`write back`。

在64位下：

- `CR3`中加入了对**PCID**的支持。当`CR4`寄存器的`PCIDE`位为1，此时这低12位就表示`PCID`。此时`PCD`和`PWT`就被覆盖了

- **XD**（EXecute Disable），主要是控制可执行权限用的。

# 伙伴系统

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11 | `/*`  `* Set` `up kernel memory allocators`  `*` `/`  `static void __init mm_init(void)`  `{`  `......`  `mem_init();` `/``/``伙伴系统初始化`  `......`  `kmem_cache_init();` `/``/``slab初始化`  `......`  `}` |

## overview
