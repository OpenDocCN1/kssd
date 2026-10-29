
# Windows堆初探

## Windows堆概述

很多应用程序都会有内存需求，而这些应用程序的内存需求通常是频繁且零散的，如果应用程序每次有内存需求都通过内存管理器来申请，势必会有很大资源开销，影响系统性能。因此，用户层的堆管理方式就出现了。堆可以向内存管理器一次性申请一块比较大的内存，然后根据应用程序的需求分割成对应小块。申请和释放的操作可以由堆管理器来处理，当应用程序的生命周期结束时堆也随之销毁，堆申请的内存全部还给内存管理器。

### Windows堆的发展阶段

堆管理机制的发展大致可以分为三个阶段

1. Windows 2000～Windows XP SP1：堆管理系统只考虑了完成分配任务和性能因素，没有考虑安全因素，可以比较容易发被攻击者利用。
2. Windows XP 2～Windows 2003：加入了安全因素，比如修改了块首的格式并加入安全 cookie，双向链表结点在删除时会做指针验证等。这些安全防护措施使堆溢出攻击变得非常困难，但利用一些高级的攻击技术在一定情况下还是有可能利用成功。
3. Windows Vista～Windows 7：不论在堆分配效率上还是安全与稳定性上，都是堆管理算法的一个里程碑

## 堆的数据结构(`_HEAP`)

Windows通过`HEAP`结构来记录和维护堆的管理信息，该结构位于堆的开始处。当使用`HeapCreate`函数创建堆时，该函数会返回一个堆句柄。这个句柄不同于窗口对象、内核对象的句柄需要结合句柄表使用，而是独立存在可以直接使用去访问堆，也就是说这个句柄实质上也是一个指向堆的指针。该地址所指向的数据结构就是`HEAP`，`HEAP`结构的定义如下：

|  |  |
| --- | --- |

## 堆的申请与释放

win32提供的堆申请的api函数很多，包括`LocalAlloc`，`GloabalAlloc`，`HeapAlloc`，`malloc`等函数。这些函数都殊途同归，最终调用`Ntdll.dll`中的`RtlAllocateHeap()`。同样的所有的释放函数最终都会调用`Ntdll.dll`中的`RtlDestoryHeap()`函数，这两个函数都是ring3最底层的函数。

![111](upload/attach/202311/967913_9M6CK5CADME7F7R.webp)

在申请堆内存时，`RtlAllocateHeap`函数会对申请的堆大小进行更改，加上标准堆头的大小，并对申请的大小进行位与运算后进行对齐补全。最后便如上面在堆表处所提到的，从堆表中寻找一个合适的可以分配的堆块返回。

```c
static ARENA_FREE* HEAP_FindFreeBlock(HEAP * heap, SIZE_T size,
SUBHEAP **ppSubHeap)
{
SUBHEAP* subheap;
struct list* ptr;
SIZE_T total_size;
FREE_LIST_ENTRY* pEntry = heap->freeList + get_freelist_index(size + sizeof(ARENA_INUSE));
/* Find a suitable free list, and in it find a block large enough */
ptr = &pEntry->arena.entry;
while ((ptr = list_next(&heap->freeList[0].arena.entry, ptr)))
{
ARENA_FREE* pArena = LIST_ENTRY(ptr, ARENA_FREE, entry);
SIZE_T arena_size = (pArena->size & ARENA_SIZE_MASK) +
sizeof(ARENA_FREE) - sizeof(ARENA_INUSE);
if (arena_size >= size)
{
subheap = HEAP_FindSubHeap(heap, pArena);
if (!HEAP_Commit(subheap, (ARENA_INUSE*)pArena, size)) return NULL;
*ppSubHeap = subheap;
return pArena;
}
}
/* If no block was found, attempt to grow the heap */
if (!(heap->flags & HEAP_GROWABLE))
{
WARN("Not enough space in heap %p for %08lx bytes\n", heap, size);
return NULL;
}
total_size = size + ROUND_SIZE(sizeof(SUBHEAP)) + sizeof(ARENA_INUSE) + sizeof(ARENA_FREE);
if (total_size < size) return NULL; /* overflow */
if ((subheap = HEAP_CreateSubHeap(heap, NULL, heap->flags, total_size,
max(heap->grow_size, total_size))))
{
if (heap->grow_size < 128 * 1024 * 1024) heap->grow_size *= 2;
}
else while (!subheap) /* shrink the grow size again if we are running out of space */
{
if (heap->grow_size <= total_size || heap->grow_size <= 4 * 1024 * 1024) return NULL;
heap->grow_size /= 2;
subheap = HEAP_CreateSubHeap(heap, NULL, heap->flags, total_size,
max(heap->grow_size, total_size));
}
TRACE("created new sub-heap %p of %08lx bytes for heap %p\n",
subheap, subheap->size, heap);
*ppSubHeap = subheap;
return (ARENA_FREE*)((char*)subheap->base + subheap->headerSize);
}
```

找到合适的堆表后进行断链取出(堆溢出任意地址写的利用点)

```c
int remove (ListNode * node)
{
node -> blink -> flink = node -> flink;
node -> flink -> blink = node -> blink;
return 0;
}
```

## 堆溢出的原理

在前置知识了解后我们就可以尝试理解堆溢出的利用方式。

举个例子，当我们申请了一个8字节大小的堆内存，但在使用时写入了16字节的数据，此时超出的8字节数据就会将后面堆块的堆块头淹没。如果后面的堆块是一个空闲堆块(可以构造出来)，我们可以再写入8字节数据就可以将它的`LIST_ENTRY`也淹没掉。

*构造要溢出的空闲堆块*：申请同样大小的3个堆块，将中间的第二个释放掉，由于附近没有其他闲置堆块所以不会被合并，当下次再申请同样大小的内存就可以把它拿出来。

因为溢出的内容是可控的，所以我们可以尝试伪造一个正常的堆块头和2个任意的4字节数据。溢出后我们进行相同大小的内存申请，堆管理器就会将后面的堆块进行断链取出，此时就会一次触发任意地址写操作。

下面我们以栈空间指向函数返回地址所在的栈地址为目标，将栈空间的原始返回地址改写为目标shellcode起始地址。

```c
node0是已申请的内存，假设node0用户空间为0x8字节
node0溢出8字节到空闲node1的堆头，再溢出8字节将node1的Flink和blink覆盖成目标地址和内容
node1的next覆盖成栈地址
node0(占用的) |node1(空闲的)
node0的8字节可用空间 | node1堆头 |flink |blink
0x 41 41 41 41 41 41 41 41 32 32 32 32 32 32 32 32 12 ff 2b 00 40 10 00 00
|栈地址 |函数返回地址（shellcode地址）
把node1申请走时堆管理器进行断链，执行了两步
1.node1->flink->blink = node1->blink
原本是将node2 的blink指针改为指向nodo1
现在node1->flink被覆盖成12 ff 2b 00栈地址，因此执行完这行代码后该栈地址的内容就回被替换成node1->blink，即函数返回地址。
2.node1->blink->flink = node1->flink
此时函数返回地址,也就是shellcode处前4个字节会被破坏。
```

部分被破环的4字节数据不会执行流程造成影响，但是为了shellcode能够稳定执行还是需要配合一些跳板技术来使用，有兴趣的可以搜索相关的内容。

除了攻击改写栈的函数返回地址，还可以攻击windows异常处理的seh函数地址、其他虚函数的指针、关键内存变量、PEB 中线程同步函数的入口地址等，有点像hook。

## 结语

windows堆的内存结构和管理机制基本就是这样了，从win2000到现在的win11，堆的内存分配算法和安全机制已经改变了很多，仅通过构造堆块头已经无法完成利用，但是最基础的框架和原理还是相通，剩下的就是不同版本的新结构分析和安全机制绕过的技术学习了。

## 参考资料

本文参考了以下内容：

1. 《0day安全软件漏洞分析技术》

2. [看雪论坛-Windows平台下堆溢出漏洞学习笔记](https://bbs.kanxue.com/thread-271213.htm)

3. <mark class="encrypted">dd9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6J5k6h3q4U0N6r3!0K6i4K6u0r3N6$3W2F1k6g2)9J5c8X3u0D9L8$3u0Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3V1k6V1L8r3I4K6i4K6u0r3L8Y4c8V1L8r3I4Q4x3V1k6Z5k6h3q4H3i4K6u0W2j5H3`.`.</mark>

4. <a href="elink@fb7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3h3r3W2#2P5X3S2#2d9$3W2J5j5h3E0A6M7X3q4Q4x3V1k6H3i4K6u0r3x3e0j5&6z5o6j5%4y4o6c8Q4x3X3g2Z5N6r3#2D9">博客园-Windows堆管理机制-修竹Kirakira</a>

5. <a href="elink@8bdK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4N6W2K9i4S2A6L8W2)9#2k6U0b7@1y4e0x3I4x3K6x3$3i4K6u0r3j5i4u0@1K9h3y4D9k6g2)9J5c8X3c8W2N6r3q4A6L8s2y4Q4x3V1j5I4x3U0f1J5z5o6x3K6z5o6M7`.">CSDN-磨刀砍柴Debug-堆(heap)系列\_0x04</a>
