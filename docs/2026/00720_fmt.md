
# [原创]在GDK8下观察glibc堆

GDK8产品主页：[Nano Code](https://bbs.kanxue.com/elink@710K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2F1j5h3&6G2j5$3!0V1k6g2)9J5k6h3y4F1i4K6u0r3i4K6t1K6i4K6u0r3k6$3c8C8z5q4)9J5c8X3W2F1k6r3g2^5)

## glibc堆布局介绍

在程序运行过程中，堆可以提供动态分配的内存，允许程序申请大小未知的内存。堆其实就是程序虚拟地址空间的一块连续的线性区域，它由低地址向高地址方向增长。我们一般称管理堆的那部分程序为堆管理器。

堆在系统中的布局可以分成两大类，分别是`main_arena`主场地和`non_main_aren`辅场地。

### `main_arena`主场地的布局

在`main_arena`中，当程序第一次动态分配内存的时候，堆会被分成两块，一块是分配区，另一块是Top Chunk，Top Chunk的起始地址亦是`top`指针所指向的位置；在之后动态分配内存的过程中，只要Top Chunk的位置还够用，分配区就会从Top Chunk中划分空间供自己使用，但是假如Top Chunk中的空间不够用了，那么就会通过`brk`机制扩展空间。

在`main_arena`中的`top`指针记录了分配区的结束位置的地址，`top`指针的上方为top chunk；而分配区的开始位置的地址，如下图所示。

![](https://bbs.kanxue.com/upload/attach/202111/929789_CJPPU9QUCE4XY8V.jpg)

### `non_main_aren`辅场地的布局

`non_main_arena`中如果分配内存数量不够大的话，那么分配区的开始位置的地址和结束位置的地址同`main_arena`中的情况是一样的；但是如果分配内存数量足够的大，那么在`non_main_arena`中的分配区会划分成1个又1个的子堆；不管`non_main_arena`是否划分了子堆，在主子堆中，分配区的开始位置永远是[`bins`指针+`mchunk_size`（去标志位；`bins`指针对应`malloc_chunk`中的`mchunk_size`）]，而其他子堆分配区的起始位置的地址为[`heap info`的地址+`0x20`]。

假如想要获取全部的子堆，就要遍历一下链表，不过这个链表并不是循环的，而且需要依靠`perv`指针找上一个，直到找到第1个为止。

**注意：主子堆的`heap info`内的`prev`指针永远是空（主子堆一般排在链表的第一个），此链表并不是循环的**

![](https://bbs.kanxue.com/upload/attach/202111/929789_NN7N8V9VAK79XX4.jpg)

### chunk的布局

无论是空闲chunk还是被分配的chunk，它们都是由`malloc_chunk`结构体进行描述，关于空闲的chunk并不会介绍太多，主要介绍一下被分配的chunk；在被分配的chunk中，假如上一个chunk尚未被释放，那么一般情况下`prev size`中不会记录任何东西（当然也有记录的时候，见下方），但是如果上一个chunk已经被释放了，那么`perv size`的大小就是上一个块的大小；而`size`中则会记录当前块的大小及当前块的属性，标志位的具体介绍见下方。

- 需要注意的是当前块的大小必是`2*SIZE_SZ`的整数倍；如果没有清除标志位，那么`size`的大小可能就不是`2*SIZE_SZ`的整数倍了。

- 比特0：记录当前块是否已经被分配；0表示不属于，1表示属于。

- 比特1：记录当前块是否是由`mmap`分配的；0表示不属于，1表示属于。

- 比特2：记录当前块是否不属于主线程；1表示不属于，0表示属于。

在`malloc_chunk`中的`fd`及`bk`字段，假如当前块处于被分配的状态，那么从`fd`开始就是数据；但是假如当前块是空闲的，那么`fd`及`bk`会分别指向下/上1个的空闲块。

`fd_nextsize`，`bk_nextsize`这两个指针指在块空闲的时候使用，指向下/上第1个大小与当前块不同的空闲块

![](https://bbs.kanxue.com/upload/attach/202111/929789_YKDXUPJMCSBR6BX.jpg)

## arena命令的基本格式

`!heap [-arena地址] [-显示选项]`

显示选项说明见下面。

## arena命令的演示

1. 首先先要和GDK8建立连接；关于建立连接的具体步骤可以参考下面的链接。

[GDK8上手指南 - Powered by MinDoc](https://bbs.kanxue.com/elink@c82K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2F1j5h3&6G2j5$3!0V1k6g2)9J5k6h3y4F1i4K6u0r3N6$3W2C8K9g2)9J5c8X3c8G2j5%4y4Q4x3V1k6Y4k6r3D9^5i4K6g2X3M7s2u0A6L8h3g2J5i4K6u0r3M7s2u0A6L8h3g2J5i4K6g2X3k6$3c8C8z5q4)9#2k6X3N6#2K9h3c8W2)

2. 输入`.sympath+ c:\temp`载入libc符号文件；如下图所示。

![](https://bbs.kanxue.com/upload/attach/202111/929789_P9JAEDBV29WUPUH.jpg)

**注意：如果还没有下载libc符号，可以通过`sudo apt install libc6-dbg`命令进行下载；完成下载以后，可以通过`sudo find /usr/lib/debug -name "libc-x.xx.so"`命令查找libc符号所在位置 [ 一般在`/usr/lib/debug/lib`目录下 ]；找到libc符号后，将其复制到主机的`c:\temp`目录下即可。**

3. 设置断点，输入`g`后，等待断点命中；如下图所示。

![](https://bbs.kanxue.com/upload/attach/202111/929789_YRS6UN2AG99EFJU.jpg)

4. 输入`x libc_so!main_arena`手动载入libc符号；如下图所示。

![](https://bbs.kanxue.com/upload/attach/202111/929789_CSTSP2QS6BVH6Q3.jpg)

5. 输入`.load ndx`载入扩展命令。

6. `!arena` => 查看所有arena的基本信息，大致列出来了`malloc_state`中的重要信息；如下图所示。

![](https://bbs.kanxue.com/upload/attach/202111/929789_8HNYBM9NQDK9289.jpg)

第1列： arena地址信息。`x`为0，代表`main_arena`；其余数字代表`non_main_arena`。

第2列： 标识了是否含有fast chunks。

第3列： `flag`标志位信息；`bit0`记录了分配区是否有fast bin chunk，`bit1`标识分配区是否能返回连续的虚拟地址空间。

`bit0`： `F`代表有，-代表没有； `bit1`： `C`代表连续，`N`代表不连续。

第4列： 线程信息；是否含有子线程。

第5、6列： 分配的内存大小信息。

7. `!arena -地址 -d` => 用于列出对应地址arena的详细信息；如下图所示。

![](https://bbs.kanxue.com/upload/attach/202111/929789_TZ88KNE9A7VMGRQ.jpg)

从图中我们可以看到，该arena是属于主线程的，并且可以看到他的地址，`next`指针指向的是下一个arena的地址；`top`指针指向top chunk的地址，我们仍可以看到`top`指针对应的`malloc_chunk`结构体内的详细信息；在往下是`bins`数组的信息；最后列出的是分配区的情况，从分配区中可以看到第一列是块对应的地址，第二列是块的属性，因为这是主线程，所以`N`全是0，假如是子线程的话`N`就是1，`M`为0代表不是内存映射，`P`代表这个块是被分配的，第三列`prev size`是上一个块的大小，由于上一个块未释放且未“越界”，因此看到的`prev size`就会是0，第四列`size`是当前块的大小。

8. `!arena -地址 -h` => 列出所有子堆的详细信息；如下图所示。

![](https://bbs.kanxue.com/upload/attach/202111/929789_KH6T472A2YBAZQP.jpg)

第2列： 子堆对应的地址（很明显可以看到低位全是0，对应的是`_heap_info`的典型地址类型）。

第3、4列： 子堆分配区的大小。

9. `!arena -地址 -s` => 统计相同大小块的信息；如下图所示。

![](https://bbs.kanxue.com/upload/attach/202111/929789_U7KZAF9F2E3SSYA.jpg)

第2列： 按块的大小进行分类。

第3列：拥有相同大小块的数量。

第4列：拥有相同大小块的总大小。

第5列：当前块在所有块当中的占比。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)
