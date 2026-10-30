
```c
#define KMALLOC_MAX_CACHE_SIZE (1UL << KMALLOC_SHIFT_HIGH)
#define KMALLOC_SHIFT_HIGH (PAGE_SHIFT + 1)
#define PAGE_SHIFT 12
static __always_inline
void *__do_kmalloc_node(size_t size, gfp_t flags, int node, unsigned long caller)
{
...
if (unlikely(size > KMALLOC_MAX_CACHE_SIZE)) {
ret = __kmalloc_large_node(size, flags, node);
...
ret = __kmem_cache_alloc_node(s, flags, node, size, caller);
...
}
```
在使用`kmalloc`分配内存堆块的过程中，

如果分配`size`大小大于`KMALLOC_MAX_CACHE_SIZE`

那么将会使用`alloc_page()`直接从伙伴系统申请页面，`kfree`也是同理

```c
void kfree(const void *object)
{
struct folio *folio;
struct slab *slab;
struct kmem_cache *s;
trace_kfree(_RET_IP_, object);
if (unlikely(ZERO_OR_NULL_PTR(object)))
return;
folio = virt_to_folio(object);
if (unlikely(!folio_test_slab(folio))) {
free_large_kmalloc(folio, (void *)object);
return;
}
slab = folio_slab(folio);
s = slab->slab_cache;
__kmem_cache_free(s, (void *)object, _RET_IP_);
}
EXPORT_SYMBOL(kfree);
```

但某些对象即使小于`KMALLOC_MAX_CACHE_SIZE`,同样会由于开发人员减少开销的操作来直接使用`alloc_page()`,例如`PTE page`

除此之外，当对象使用`alloc_pages()`分配或者释放的时候，如果大小小于`order 3`,则会从`PCP list`(以前也被叫做冷热页，这里主要是为了防止多次访问全局的buddysystem的时候造成的阻塞，因此每个`CPU`拥有自己的`free page`列表)里面进行分配或者释放，也就是说当内核申请页表空间的时候首先将会使用`alloc_pages()`在当前`CPU`的`pcp list`里面分配页面

```
alloc_pages alloc_pages_node
│ │
▼ ▼
alloc_pages_node ──────────►__alloc_pages_node __alloc_pages_node
│ │
▼ │
__alloc_pages ◄─────────────────────────┘
│
┌──────────────────────┴─────────┬───────────────────────────────────┐
▼ ▼ ▼
prepare_alloc_pages get_page_from_freelist alloc_pages_slowpath
get zonelist info alloc new pages reclaim pages
│ alloc new pages
▼
rmqueue
│
┌─────────────┴──────────────┐
│ │
▼ ▼
rmqueue_pcplist rmqueue_buddysystem
order < 3 allocation allocate from buddysystem
│
┌─────┴──────────────────┐
empty(pcp list) not empty
▼ ▼
rmqueu_bulk return pcp_list chunk
reload pcp from buddysystem
```

详细的内存管理可以查看[内存管理浅析](elink@117K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6H3k6h3W2%4K9i4c8Z5K9r3q4G2i4K6u0r3d9r3q4U0K9$3g2J5i4K6u0V1g2h3&6A6N6X3g2J5M7$3W2@1P5g2)9J5k6r3!0X3i4K6u0V1M7r3g2A6N6$3W2@1K9r3S2S2L8#2)9J5c8X3u0D9L8$3u0Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3V1k6K6P5i4y4@1k6h3#2Q4x3V1k6E0k6h3#2G2M7Y4W2Q4y4h3k6E0j5h3&6S2k6$3g2E0k6h3&6@1i4K6u0W2L8h3b7`.)

目前目标:

1. 分配指定大小`skb`,
2. 准备双重释放该`skb`，
3. 第一次释放`skb`,延迟第二次释放`skb`
4. 分配页表, 占领该`skb`
5. 第二次释放`skb`,此时页表也被释放
6. 重新申请该页

目前可以获取的情报:

1. 分配的`skb`能够被页表所占领
2. 页表由`alloc_pages()`分配，从`pcp list`得到
3. `skb`如果小于`8K`,则将从`slab allocator`分配
4. `skb`如果大于`8K`且小于`8x4K`,则从`page allocator`分配或者释放，但是是从`pcp list`进行分配
5. 如果`skb`大于`8x4k`, 同样从`page allocator`分配,直接从`buddysystem`获取
6. 如果`skb`采用3分配和释放，则`skb`释放后只会回到`slab`当中，无法造成`skb`和`PTE page`占用同一页
7. 如果`skb`采用4分配和释放，则`skb`虽然是从`pcp list`, 但是`pcp list`里面也有`order`差异，也无法造成共同占用
8. `skb`采用5,则`skb`将会释放回`buddysystem`当中，此时有可能造成共同占用，当`pcp list`没有符合的页面就将从`buddysystem`获取

综上，`skb`将会以`order-4`(16x4KB)的形态出击

而为了造成`pcp list`没有符合的页面，需要首先耗尽其中所有的页

内核函数`rmqueue_bulk()`由`__rmqueue_pcplist()`分配，当`pcplist`为空的时候，就将使用该函数来填充`rmqueue_bulk`,

该函数将从`buddysystem`获取页面来填充`pcplist`

因此这里可以堆喷`PTE page`来消耗该`pcplist`

## IP fragment trick

![ip 头部](upload/attach/202507/955986_Q8W9Z8SPHKW4Q3A.png)

使用了`IP`包分片的技巧来巧妙的进行延迟的`double free`

`IP`分片这一块是`408`网络的重点部分:), 如果记得不太详细可以直接查看该[博客](elink@b98K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5h3y4C8k6i4c8H3N6i4y4Z5k6i4u0K6i4K6u0W2L8X3g2@1i4K6u0r3j5X3I4G2k6#2)9J5c8X3W2H3i4K6u0V1k6Y4u0S2k6$3#2W2L8Y4c8S2N6r3W2G2L8W2)9J5k6r3W2F1i4K6u0V1k6r3g2@1j5h3W2D9i4K6u0r3)

在上面基础知识我们知道如果传递`IP`分片到目标机器，首先并不会直接送入上层进行分析，而是在内核维持一个`skb`队列，

等待队列满了之后再递交上层处理

我们知道在上面的漏洞点中，我们可以在处理`netfilter hook`函数的时候将某个`skb`进行一次释放，而释放过后依旧会走完全程，

在通过漏洞释放完`skb`之后，`skb->len`字段将会被随机数覆盖，这是内核的保护机制,此时二次释放该`skb`可能会导致内核`panic`

因此何时触发二次释放就十分重要，

这里存在两个技巧：

1. 分片包在内核维持的队列有时间限制，一旦超过了这个时间限制，则该队列的`skb`包则会全部释放，
这个时间限制由内核的`sysfs`文件系统掌控,位于`/proc/sys/net/ipv4/ipfrag_time`,而设置他并不需要`root`权限，
因此我们如果将该值调大，那完全可以创建一个无限大的窗口供我们使用
2. 仔细阅读`ip_frag_queue`的代码，当他检查该分片头部的时候如果`offset==len`,那么将会判断是错误包，直接释放该`skb`队列,并且不会检测之前放入`queue`的头部是否正确

整合该两点，我们就可以通过构造`IP`头，在任意我们想要的时机对一个`skb`进行`double free`

```
set ipfrag_time=9999 │
│ │
│ │
│ │
▼ │ 
sendto ip fragment skb1 │
│ │ ─┐
freed by exploit ▼ │ │
│ ▼ │
waiting in queue │ │ i can do everthing i want
│ │ ▼
│ │ │
▼ │ │
│ │ ─┘
│ │ sendto ip fragment skb2
└──────────┬─┴───── but offset == len
│
▼
free skb1 skb2 in current cpu
(double free)
```

## 堆喷PTE

需要堆喷`PTE`,在`dirtypagetable`里面就已经讲解到,可以通过用户使用`mmap`来分配页表,

但是`mmap`的时候并不是分配页表的时候，他会首先在内核`task_struct`中建立`VMA`,当用户访问`VMA`区域内的地址的时候才会出发`page fault`,

然后造成页表分配

但是如果选择的映射区域不准确，则可能会造成一系列分配例如`PUD`,`PMD`的额外分配,这样在我们堆喷的过程中就可能造成噪音从而导致利用失败

那么我们就可以选择舍弃一部分`PTE`,那就是首先通过分配某个`PTE`的第一个`page`来提前将`PMD`和`PUD`分配完毕,例子如下

```
PUD PMD PTE page
┌────►┌────────┐ ┌──►┌─────────┐ ┌──►┌─────────┐ ┌─────────┐
PGD │ │ ├──┘ │ ├─┘ │ ├──►│ aaaaaaa │
┌───────┐ │ ├────────┤ ├─────────┤ ├─────────┤ │ │
│ ├────┘ │ │ │ │ │ │ │ │ │
├───────┤ │ │ │ │ │ │ │ │ │
│ │ │ │ │ │ │ │ │ │ │
│ │ └────────┘ └─────────┘ └─────────┘ └─────────┘
└───────┘
```

对于提前的`VMA`写入就将导致`PUD`和`PMD`的分配，接下来当继续访问在`PUD/PMD`可表示的虚拟地址范围内的地址就可以保证稳定分配`PTE`

## Dirty pageDirectory

作者由`dirtypagetable`启发而来,简单介绍一下`dirty pagetable`的原理在于:

> [!NOTE]
> 当存在`UAF`的时候,将漏洞堆块和`PTE`页表分配到一起，然后就可以通过操作漏洞堆块来影响`PTE`页表，从而导致任意物理地址写
> 但是这个步骤需要漏洞堆块稳定能操作，比如说是稳定写入字节

```
step1 ────free the obj─► step2 ──mmap──────► step3
┌───────────┐ ┌─────────────┐ ┌───────────┐
│vuln object│ │ │ │ PTE │
└───────────┘ └─────────────┘ └───────────┘
▲
│
ACCESS PTE BY VULN OBJECT
```

但可惜的是有时候无法获取一个稳定的漏洞利用模块，比如这次漏洞的`skb`，因此作者仅仅使用一次漏洞触发，

之后的操作完全是对于用户虚拟地址的访问来达成任意物理地址读写

实际操作起来的原理十分的简单，仅仅是使`PMD`和`PTE`均分配到同一个物理页面，这样做导致的后果就是:

> [!IMPORTANT]
> 原来`PTE`访问的`page`页面会变成`PMD`访问到的`PTE`页面，所以如果能够在原来`PTE`访问的`page`页面伪造合适的页表项,则就可以通过`PMD`来执行任意物理地址读写操作

```
┌───► PUD0 ────► PMD0 ───────► PTE0 ──────► page0 ───► ???
│ │ │ │
│ │ │ │
│ │ │ │
PGD ──┤ ┌─same phys──┘ ┌─same phys─┘ ┌─?──┘
│ │ │ │
│ │ │ │
└───► PUD1 ────► PMD1(PTE0)───► PTE1(page0) ───► page1(???)
```

## TLB 刷新

[TLB](elink@149K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2L8W2)9J5k6i4N6A6K9$3W2H3k6h3c8A6j5g2)9J5k6h3!0J5k6#2)9J5c8Y4N6A6K9$3W2Q4x3V1k6f1M7X3q4F1M7$3I4S2N6r3W2G2L8W2)9#2k6X3I4G2L8$3E0S2M7$3W2V1k6g2)9#2k6X3u0#2k6X3k6W2M7R3`.`. )用来缓存虚拟地址到物理地址的映射，从而可以使得加快页表访存,

所以在`dirty pagedirectory`的工作下我们需要清空`TLB`来保证该技术的稳定性,而这一步骤也可以被称为刷新`TLB`

这里该作者提出的刷新方法是在堆喷`PMD`和`PTE`的时候，可以将他们标记为共享页面，然后通过创建子进程，然后该子进程调用`munmap()`刷新内存`TLB`,从而导致父进程的`TLB`也会进行刷新，该操作在需要我们不断修改页表的环境下是极其重要的

最后让子进程进入睡眠

# 漏洞利用流程

接下来讲解具体流程:

1. 创建用户和网络命名空间，在其中可以让我们执行特权操作
2. 做环境准备，例如绑定`cpu`核等
3. 启动环回网卡,然后禁用`rpf`(反向路径转发),因为后续需要使用特制`ip`头
4. 构造`netfilter hook`, 这里`hook`点为`NF_INET_PRE_ROUTING`,具体原因可以参考基础知识环节
5. 预分配`PUD`，`PMD`,`PTE`，`socket`等结构体，来减少后续漏洞利用噪音的产生，这里虚拟地址的计算通过几个巧妙的宏来实现

```c
#define _pte_index_to_virt(i) (i << 12)
#define _pmd_index_to_virt(i) (i << 21)
#define _pud_index_to_virt(i) (i << 30)
#define _pgd_index_to_virt(i) (i << 39)
#define INDEX_TO_VIRT(pud_index, pmd_index, pte_index, page_index, byte_index) \
((void*)(\_pgd_index_to_virt((size_t)(pud_index)) + \_pud_index_to_virt((size_t)(pmd_index)) + \
\_pmd_index_to_virt((size_t)(pte_index)) + \_pte_index_to_virt((size_t)(page_index)) + (size_t)(byte_index)))
...
void *addr = mmap((void *)INDEX_TO_VIRT(1, 0, 0, 0, 0), 0x2000, PROT_READ | PROT_WRITE, MAP_FIXED | MAP_SHARED | MAP_ANONYMOUS, -1, 0);
if (addr == MAP_FAILED) {
perror("mmap failed"); // perror 会打印出具体的错误原因
exit(EXIT_FAILURE);
}
// page fault: allocate PUD PMD PTE page
*(size_t *)INDEX_TO_VIRT(1, 0, 0, 0, 0) = (size_t)0xDEADBEEF;
// pre allocation: only create the vma (mostly PTE page, and some PMD page)
for(size_t i = 0; i < CONFIG_PTE_SPRAY_AMOUNT; i++){
void *retv = mmap((void *)INDEX_TO_VIRT(2, 0, i, 0, 0), 0x2000, PROT_READ | PROT_WRITE, MAP_FIXED | MAP_SHARED | MAP_ANONYMOUS, -1, 0);
if (addr == MAP_FAILED) {
perror("mmap failed"); // perror 会打印出具体的错误原因
exit(EXIT_FAILURE);
}
}
// occur page fault
for(size_t i = 0; i < CONFIG_PTE_SPRAY_AMOUNT/CONFIG_PAGE_ENTRY_NUM; i++){
*(char *)INDEX_TO_VIRT(2, i, 0, 0, 0) = '0x41';
}
```

6. 分配大量`udp skb`, 仅仅用来放在`double free`之间来防止错误
