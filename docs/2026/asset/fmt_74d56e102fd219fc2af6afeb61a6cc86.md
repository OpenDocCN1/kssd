
在分配之前，程序初始化出了一块`0x5637b467a000-0x5637b469b000`共 `0x21000`（132 * 1024）大小的内存，作为初始的`heap`，初始`heap`有 132kb 的可读可写区，再申请的时候，可以从这132kb 的空间中申请，直到用完。用完之后可以继续扩展 `brk` 的位置。同样，当 `top chunk` 上有足够大的内存时，`brk` 的值也会缩小。

申请一块 1000 byte 大小的`chunk`，这块`chunk`会从`heap`段中获取，释放也不会直接归还系统，而是分情况讨论。如果与`top chunk`物理相邻，就会合并进 `top chunk`，没有物理相邻则会进入对应的`bin`中。

![](https://bbs.kanxue.com/upload/attach/202204/882497_99DT4YA3J7TD2N7.png)

![](https://bbs.kanxue.com/upload/attach/202204/882497_DKKGBBHM4F9UCBH.png)

在 `thread1` 进行 `malloc` 之前，映射了一块区域，我不知道这个是在干嘛，有师傅知道可以教教我。

![](https://bbs.kanxue.com/upload/attach/202204/882497_NKVR49JYNSG4GX9.png)

![](https://bbs.kanxue.com/upload/attach/202204/882497_FESA869GSZXXSUB.png)

而在 `thread1` 进行 `malloc` 之后，由于非主分配区只能访问进程的`mmap`映射区域，调用`mmap`函数向系统申请`HEAP_MAX_SIZE`大小的内存，在32位机器上是1M，64位机器是64M。这64M的堆内存被映射进进程`mmap`映射段的地址空间。这64M中仅有132kb的可读可写区，称为这个线程的堆内存，也就是`thread arena`。

|  |  |
|---|---|
| 1 2 3 4 5 6 7 | `pwndbg> info macro HEAP_MAX_SIZE` `Defined at` `/home/pukrquq/Downloads/glibc-2.34/malloc/arena.c:32` `included at` `/home/pukrquq/Downloads/glibc-2.34/malloc/malloc.c:2013` `#define HEAP_MAX_SIZE (2 * DEFAULT_MMAP_THRESHOLD_MAX)` `pwndbg> info macro DEFAULT_MMAP_THRESHOLD_MAX` `Defined at` `/home/pukrquq/Downloads/glibc-2.34/malloc/malloc.c:978` `#define DEFAULT_MMAP_THRESHOLD_MAX (4 * 1024 * 1024 * sizeof(long))` |

注意：当用户请求大小超过 128 KB，并且当 `arena` 中没有足够的空间来满足用户请求时，使用 `mmap` 系统调用（而不是使用 `sbrk()` 函数）分配内存，不管请求是从 `main arena` 还是 `thread arena` 发出的。如果小于128kb，则`main_arena`会使用`sbrk()`函数来扩展`heap`段。如果是`main_arena` 调用 `mmap`函数分配内存，当 `free` 该内存时，主分配区会直接调用 `munmap()` 将该内存归还给系统。

![](https://bbs.kanxue.com/upload/attach/202204/882497_2JZMDECU8H46QEM.png)

`thread1` `free`掉内存之后，放入对应的`thread arena` `bins`中。

了解了这些之后，可以重新认识我们的 `house_of_mind_fastbin` 了。

这个技术就在于伪造一个 `arena`，然后不断增加 `brk` 的值来扩展`heap`段，再修改 `chunk` 的 `non_main_arena` 标志位。那么再次 `free` 的时候，这块 `fast bin` `chunk` 就会进入我们伪造的 `arena` 的`fast bin`中了。下面可以调试看看。

### POC

how2heap源码：

|  |  |
|---|---|
