
# [原创] 现实世界的堆漏洞利用图鉴-2021-10-06更新

[TOC]

## 概述

本文将以多个真实的堆相关的漏洞以及一些高质量的研究文章为例，介绍在真实漏洞场景下的堆利用技巧，主要是介绍各种现实漏洞场景的堆布局技巧。

`vuln obj` : 存在漏洞的对象，比如存在堆溢出的对象。

`target obj`: 漏洞利用时需要篡改的对象，比如利用堆溢出时，我们想覆盖的对象。

GitHub & BLOG

```
https://www.cnblogs.com/hac425/p/15371359.html
https://github.com/hac425xxx/heap-exploitation-in-real-world
```

## A tale of two mallocs

当来利用堆溢出的时候，我们需要找一个 gadget object 来进行利用

![image-20211006121340597](upload/attach/202110/690974_GD3EVFPAGAYSSHZ.png)

常用的 gadget object 类型：

1. 有函数指针，虚表指针，可用于劫持控制流。
2. 对象中有 size/length 字段，修改这个字段可以导致后续使用对象时发送越界。
3. 对象中的指针在后面的逻辑中会被用于读、写内存等操作，利用程序的逻辑可以完成一些漏洞利用的原语。

此外利用堆漏洞非常重要的是进行堆布局，为了进行堆布局我们需要找到外部输入可控的内存操作原语，搜索内存操作原语时需要关注三个要素：

1. 内存分配的大小是否可控
2. 内存中的内容是否可控
3. 分配到的内存的生命周期是否可控，即我们能否控制其申请和释放的时机

然后根据不同的内存控制原语采用合适方式进行堆布局，常见搜索内存操作原语的思路是在代码中找含有`malloc`，`new`，`realloc`，`std::vector`，`std::string`的调用点，然后判断是否可以由外部输入触发。

作者提到进行堆布局的时候，可以选用占位式堆喷，如图所示：![image-20211006123541144](upload/attach/202110/690974_MGZBT8XD4CX3UEQ.png)

此外为了提升溢出到 gadget 的概率我们可以先把内存碎片清理了，然后再进行占位式堆喷，这样 `vuln obj` 和 `gadget obj` 大概率会相邻分配。

如果分配 gadget object 的时候会有一些小对象的分配从而导致影响内存布局，我们可以先在堆上占位一些小的内存块，然后在分配 gadget 前释放一些小的内存块，这样小对象就会落在之前占位的小内存块中，避免影响 vuln obj 和 gadget object 之间的布局。

![image-20211006123820942](upload/attach/202110/690974_5ARRT79AFK7GB7V.png)

Assuming the gadget object is the one which allocates the unwanted allocations, one way to deal with this issue is to do the following:

1. First, at the beginning of the exploit, we allocate a bunch of smaller blocks, let’s say 100 blocks of 0x100 bytes each
2. We then allocate all the placeholder sets （表示一组相邻的 overflowable 和 gadget 占位对象）
3. Then, for each placeholder set, we first free the gadget placeholder.
4. Then we free a few of the small filter allocations. These will be added to the free bins
5. Then we allocate the gadget itself, the smaller unwanted allocations will use the filter allocations we just freed, and the gadget itself will fall on the placeholder.

## CVE-2015-3864

漏洞是一个整数溢出导致的堆溢出

```
case FOURCC('t', 'x', '3', 'g'):
{
uint32_t type;
const void *data;
size_t size = 0;
if (!mLastTrack->meta->findData(
kKeyTextFormatData, &type, &data, &size)) {
size = 0;
}
if (SIZE_MAX - chunk_size <= size) { // <---- attempt to prevent overflow
return ERROR_MALFORMED;
}
uint8_t *buffer = new uint8_t[size + chunk_size];
if (size > 0) {
memcpy(buffer, data, size);
}
if ((size_t)(mDataSource->readAt(*offset, buffer + size, chunk_size))
< chunk_size) {
delete[] buffer;
buffer = NULL;
return ERROR_IO;
}
```

在分配内存前有一个检查，但由于 `chunk_size` 在 32 位系统下下也是 `uint64_t` 的，所以当 `chunk_size > SIZE_MAX` 时（比如 `0xFFFFFFFFFFFFFFFF`），会通过检查，然后下面分配的时候就会整数溢出。

从文件中解析 `chunk_size` 的代码如下

```
status_t MPEG4Extractor::parseChunk(off64_t *offset, int depth) {
uint32_t hdr[2];
if (mDataSource->readAt(*offset, hdr, 8) < 8) {
return ERROR_IO;
}
uint64_t chunk_size = ntohl(hdr[0]);
uint32_t chunk_type = ntohl(hdr[1]);
off64_t data_offset = *offset + 8;
if (chunk_size == 1) {
if (mDataSource->readAt(*offset + 8, &chunk_size, 8) < 8) {
return ERROR_IO;
}
chunk_size = ntoh64(chunk_size); // 从文件中获取 8 字节的 chunk_size
```

因此通过这个漏洞我们可以实现一个堆溢出，`vuln obj` 的大小可控，溢出的内容可控，由于是整数溢出后面可能会拷贝大量的数据（比如 `0xFFFFFFFFFFFFFFFF`），最终可能会由于访问到没有映射的内存导致进程崩溃。

因此，利用这种类型的整数溢出漏洞，我们需要找到一些路径，让溢出发生后，不会拷贝过量的数据，对于这个漏洞来说我们可能有以下两种方式：

1. 内存分配后，程序会调用 `readAt` 从文件中读取 `chunk_size` 的数据，如果能让 `readAt` 返回值小于 `chunk_size` （比如控制文件大小），就能避免溢出过量的数据。
2. 通过覆盖在具体数据拷贝之前调用的函数指针完成利用，从而避免了溢出过量数据。

该漏洞的利用采取的是第二种方式，即溢出到 `mDataSource` 对象 (`其类型` `MPEG4DataSource`), 然后把该对象的虚表劫持，从而劫持 `readAt` 这个虚函数调用。

那么利用该漏洞的 `target obj` 就是 `mDataSource` 对象，如果要利用该漏洞，`buffer` 和 `mDataSource` 的内存布局如下：

![image-20211006145738908](upload/attach/202110/690974_J6EEZKME47N3KHG.png)

下面就介绍如何利用程序中的逻辑来整理堆的布局，实现上述的布局，首先需要分析源码定位 vuln obj 和 target obj 的分配方式，当解析到 `stbl` 标签时，会分配 `mDataSource` 对象（`target obj`），当解析 `tx3g` 标签时就会分配 vuln obj 并触发溢出。

然后再看看程序用的内存分配器的一些特点，其使用的是 `jemalloc` ，`jemalloc` 的大概思路是把内存块分成大小相同的 object，然后返回给申请者，类似于内核的 slub 分配器。

由于堆分配器的实现，堆在**初始情况**下先分配的对象的地址会比后分配的对象的地址小，这时如果直接先分配 target obj 的话，`vuln obj` 就会在 target obj 的后面，因此无法溢出到 target obj.

![image-20211002221526076](upload/attach/202110/690974_KQHYTHQKGBWD7AH.png)

假如先分配 vuln obj 的话，由于漏洞触发点的**内存分配和数据溢出是一起发生的**，也是无法覆盖到 `mDataSource`。

或者我们可以利用`jemalloc`的LIFO特性，先把后面的块释放了，然后把前面的块释放了，然后再先分配 target obj 后分配 vuln obj，也可以实现上述的布局。

![image-20211004111532366](upload/attach/202110/690974_69TSHJBWZ9RE4G3.png)

`exploit` 里面的思路是采用占位对象，然后通过控制占位对象的分配与释放，来完成布局，示意图如下：

![image-20211002205439796](upload/attach/202110/690974_9JYPSHWP674UTR3.png)

1. 首先分配两个占位对象 A 和 B，且 A 和 B 的前后都是被分配出去的内存，然后释放B，再利用 `stbl` 标签分配 target 对象，这样 `mDataSource` 就会落在 B 的位置。
2. 然后释放 A ，并利用 `tx3g` 标签触发漏洞，分配`buffer`到 A 的位置，就能完成我们需要的布局，这样一溢出就可以覆盖到 `mDataSource` 对象。

要完成上述布局，关键点在于需要找到可以通过用户输入（`mp4`文件）控制 分配/释放 的代码逻辑，在 `libstagefright` 中我们可以利用 `avcC` 和 `hvcC` 来进行占位，通过这两个标签的特点为：

1. 分配任意大小的内存，内存中的内容可控（从文件读取）
2. 内存释放的时机可以通过输入文件中的特定标签控制

利用 `avcC` 和 `hvcC` 来占位的示例图如下：

![image-20211002210810170](upload/attach/202110/690974_G9Z48C7RMNXJPBA.png)

为了稳定的完成堆布局，还需要解决内存碎片的问题，上述布局成立的前提是 `avcC` 和 `hvcC` 对象是相邻的，如果内存中存在内存碎片的话，这两个对象是有可能不相邻的。

![image-20211004113906179](upload/attach/202110/690974_5QC87TT8U7EYK4S.png)

为了解决这个问题常用的方式是首先把之前的内存碎片清掉（即大量分配内存，把之前堆中碎片的内存块申请完），之后再申请内存的时候，我们分配到的内存块的顺序就会符合我们的预期了。

![image-20211004115112728](upload/attach/202110/690974_7D87NUE3CK5AVED.png)

`exploit` 中使用的是 `pssh` 标签来清理内存碎片，原因是程序在解析 `pssh` 标签时会根据标签中的数据分配内存，内存的大小和内容可控，且分配出来的内存在整个文件解析完成后才会被释放，因此直接在输入文件中放置多个 `pssh` 标签就可以完成内存碎片的清理工作。

完成堆布局后现在就可以溢出到 `mDataSource` 对象 (`其类型` `MPEG4DataSource`), 接下来的利用思路就是覆盖对象的虚表，把虚表劫持到我们伪造的需要，然后对象进行虚函数调用的时候就可以劫持`pc`。

![image-20211004120703444](upload/attach/202110/690974_J9W8JDNBBF3KKB5.png)

目前的问题是如何知道 伪造的虚表 的地址，获取地址一般有两种方式：

1. 找一个信息泄漏漏洞，或者利用漏洞构造信息泄漏，从而可以拿到堆的地址，然后把虚表指针指向堆上的可控数据位置即可。
2. 利用堆喷射技术，在某个固定的地址布置伪造的虚表数据，然后把虚表指针指向该位置即可。

这里使用的是堆喷射技术，以32位系统为例，堆喷射的思路大概是由于进程虚拟地址空间的大小限制，当程序分配大量内存时（比如 0xff000 字节时）， 尽管存在随机化，我们依然可以以极大的概率在某个具体的地址（0xf7500000）上布置我们的数据。

```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/mman.h>
#define ALLOC_SIZE 0xff000
#define ALLOC_COUNT 0x1
int main(int argc, char** argv) {
int i = 0;
char* min_ptr = (char*)0xffffffff;
char* max_ptr = (char*)0;
for (i = 0; i < ALLOC_COUNT; ++i) {
char* ptr = mmap(NULL, ALLOC_SIZE,
PROT_READ | PROT_WRITE | PROT_EXEC,
MAP_PRIVATE | MAP_ANONYMOUS,
-1, 0);
if (ptr < min_ptr) {
fprintf(stderr, "new min: %p\n", ptr);
min_ptr = ptr;
}
if (ptr + ALLOC_SIZE > max_ptr) {
fprintf(stderr, "new max: %p\n", ptr + ALLOC_SIZE);
max_ptr = ptr + ALLOC_SIZE;
}
memset(ptr, '\xcc', ALLOC_SIZE);
}
fprintf(stderr, "finished min: %p max %p\n", min_ptr, max_ptr);
((void(*)())0xf7500000)();
}
```

利用大量的内存分配可以让进程的某个地址存放我们的数据，数据的内容如何控制呢，按照我的理解，我们在进行堆喷的时候尽量按页对齐去分配，然后以页为单位进行数据布局，就可以实现**在可预测的地址，布置可控的数据**，具体的偏移和布局还需要调试确认。

`exploit` 中的堆喷射思路如下

![image-20211004135942470](upload/attach/202110/690974_7CT9UPR38VRQCG5.png)

**小结**

该漏洞的利用非常经典，主要的特点如下：

1. 触发整数溢出后，通过劫持后续可能会使用的虚函数调用来避免整数溢出后的内存拷贝访问到不可访问的内存.
2. 通过分析程序对文件的解析流程，找到了内存申请和释放的原语（`avcC` 和 `hvcC`），随后使用这两个标签进行占位并完成了 vuln obj 和 target obj 的布局。
3. 为了增大占位成功的概率，使用了 `pssh` 标签分配大量对象，清理内存碎片。
4. 为了在没有信息泄露的情况下劫持虚表，利用 `pssh` 进行堆喷射，在可预测的地址处放置 rop gadget.

主要可以学习的思路有：

1. 触发漏洞后，要思考后续的步骤，比如后续要覆盖什么，为了完成覆盖，需要什么样的内存布局。
2. 然后去程序中查找是否存在内存控制的原语，比如内存申请/释放原语，申请的大小、内存的内容、释放的时机是否可控等。
3. 然后就根据堆分配器的特性来尝试堆布局。
4. 堆喷射在32位下还是很有用的，64位下在一些情况下面也是可以用的。
5. 占位型堆布局和内存碎片的清理也非常的经典。

## CVE-2017-0781

漏洞是8字节的溢出， `vuln obj` 的大小可控。
