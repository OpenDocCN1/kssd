
# [原创]#30天写作挑战#Tcache Attack原理学习

**ubuntu 18.04 下测试**

## tcache介绍

源码看不动，说一下通过实验得到的：

同一大小的`chunk` `free`之后前 7 个会放到一个`tcache`链表里面，不同大小的放在不同的链表中

![image.png](https://bbs.kanxue.com/upload/attach/202009/837755_N3ZXVJU6XZF873S.jpg "image.png")

最大能放`0x408`的，再大就要按照原本的那样放到`unsortedbin`中了

![image.png](https://bbs.kanxue.com/upload/attach/202009/837755_HJP3TJHNXVCU558.jpg "image.png")

程序再次申请内存块的时候首先判断在`tchche`中是否存在，如果存在的话会先从`tcache`中拿

## tcache_dup

```
#include <stdio.h>

#include <stdlib.h>



int main()

{

	fprintf(stderr, "先申请一块内存\n");

	int *a = malloc(8);



	fprintf(stderr, "申请的内存地址是: %p\n", a);

	fprintf(stderr, "对这块内存地址 free两次\n");

	free(a);

	free(a);



	fprintf(stderr, "这时候链表是这样的 [ %p, %p ].\n", a, a);

	fprintf(stderr, "接下来再去 malloc 两次: [ %p, %p ].\n", malloc(8), malloc(8));

    fprintf(stderr, "ojbk\n");

	return 0;

}
```

`gcc -g tcache_dup.c`

运行之后的结果：

![image.png](https://bbs.kanxue.com/upload/attach/202009/837755_J4S4PUHXDN8QQWS.jpg "image.png")

一开始申请了`0x8`大小的`chunk`，后来`free`了两次

![image.png](https://bbs.kanxue.com/upload/attach/202009/837755_6HBEFXJ26BNZXUN.jpg "image.png")

然后再去申请的话也会申请这俩在`tcache`中的，所以后面输出的`malloc`的地址还是一样的

## tcache_house_of_spirit

```
#include <stdio.h>

#include <stdlib.h>



int main()

{

    malloc(1);

    unsigned long long *a;

    unsigned long long fake_chunks[10];

    fprintf(stderr, "fake_chunks[1] 在 %p\n", &fake_chunks[1]);

    fprintf(stderr, "fake_chunks[1] 改成 0x40 \n");

    fake_chunks[1] = 0x40;

    fprintf(stderr, "把 fake_chunks[2] 的地址赋给 a, %p.\n", &fake_chunks[2]);

    a = &fake_chunks[2];

    fprintf(stderr, "free 掉 a\n");

    free(a);

    fprintf(stderr, "再去 malloc(0x30)，在可以看到申请来的结果在: %p\n", malloc(0x30));

    fprintf(stderr, "ojbk\n");

}
```

首先取了一个数组

![image.png](https://bbs.kanxue.com/upload/attach/202009/837755_SG92CN32VYG4BUK.jpg "image.png")

然后把数组的`index1`改成了`0x40`（`chunk`的大小）

![image.png](https://bbs.kanxue.com/upload/attach/202009/837755_XVTPGFTN8RCUTHS.jpg "image.png")

然后把数组的`index2`的地址赋给了`a`，之后去`free(a)`，再去申请回来的话就会把`a`申请回来，这样就申请了`fake_chunks[2]`那里

![image.png](https://bbs.kanxue.com/upload/attach/202009/837755_R8YWRHWKWR4PXD5.jpg "image.png")

## tcache_poisoning

```
#include <stdio.h>

#include <stdlib.h>

#include <stdint.h>



int main()

{

    setbuf(stdin, NULL);

    setbuf(stdout, NULL);

    size_t stack_var;

    printf("定义了一个变量 stack_var，我们想让程序 malloc 到这里 %p.\n", (char *)&stack_var);



    printf("接下来申请两个 chunk\n");

    intptr_t *a = malloc(128);

    printf("chunk a 在: %p\n", a);

    intptr_t *b = malloc(128);

    printf("chunk b 在: %p\n", b);



    printf("free 掉这两个 chunk\n");

    free(a);

    free(b);



    printf("现在 tcache 那个链表是这样的 [ %p -> %p ].\n", b, a);

    printf("我们把 %p 的前 %lu 字节（也就是 fd/next 指针）改成 stack_var 的地址：%p", b, sizeof(intptr_t), &stack_var);

    b[0] = (intptr_t)&stack_var;

    printf("现在 tcache 链表是这样的 [ %p -> %p ].\n", b, &stack_var);



    printf("然后一次 malloc : %p\n", malloc(128));

    printf("现在 tcache 链表是这样的 [ %p ].\n", &stack_var);



    intptr_t *c = malloc(128);

    printf("第二次 malloc: %p\n", c);

    printf("ojbk\n");



    return 0;

}
```

程序首先定义了一个变量，我们想要做的就是让程序`malloc`到变量那里

`malloc`两个`chunk`，然后`free`掉，这时候链表及内存布局是这样的

![image.png](https://bbs.kanxue.com/upload/attach/202009/837755_J83XYGY7Z8JRMRM.jpg "image.png")

![image.png](https://bbs.kanxue.com/upload/attach/202009/837755_EXS8FGZEHV8CD8C.jpg "image.png")

然后把`b`的`fd`指针改成那个变量地址

![image.png](https://bbs.kanxue.com/upload/attach/202009/837755_7JX5MJ4VX2W6TKS.jpg "image.png")

这时候`tcache`的链表是这样的

![image.png](https://bbs.kanxue.com/upload/attach/202009/837755_S699T98NQ6T8FYR.jpg "image.png")

然后连续申请两个`chunk`，来看一下申请到的地址是啥样的

![image.png](https://bbs.kanxue.com/upload/attach/202009/837755_W666BYP5FZQPCRC.jpg "image.png")

## tcache_stashing_unlink_attack

```
#include <stdio.h>

#include <stdlib.h>



int main(){

    unsigned long stack_var[0x10] = {0};

    unsigned long *chunk_lis[0x10] = {0};

    unsigned long *target;

    unsigned long *pp;



    fprintf(stderr, "stack_var 是我们希望分配到的地址，我们首先把 &stack_var[2] 写到 stack_var[3] 来绕过 glibc 的 bck->fd=bin（即 fake chunk->bk 应该是一个可写的地址）\n");

    stack_var[3] = (unsigned long)(&stack_var[2]);

    fprintf(stderr, "修改之后 fake_chunk->bk 是:%p\n",(void*)stack_var[3]);

    fprintf(stderr, "stack_var[4] 的初始值是:%p\n",(void*)stack_var[4]);

    fprintf(stderr, "现在申请 9 个 0x90 的 chunk\n");



    for(int i = 0;i < 9;i++){

        chunk_lis[i] = (unsigned long*)malloc(0x90);

    }

    fprintf(stderr, "先释放 6 个，这 6 个都会放到 tcache 里面\n");



    for(int i = 3;i < 9;i++){

        free(chunk_lis[i]);

    }

    fprintf(stderr, "接下来的释放的三个里面第一个是最后一个放到 tcache 里面的，后面的都会放到 unsortedbin 中\n");

    

    free(chunk_lis[1]);

    //接下来的就是放到 unsortedbin 了

    free(chunk_lis[0]);

    free(chunk_lis[2]);

    fprintf(stderr, "接下来申请一个大于 0x90 的 chunk，chunk0 和 chunk2 都会被整理到 smallbin 中\n");

    malloc(0xa0);//>0x90

    

    fprintf(stderr, "然后再去从 tcache 中申请两个 0x90 大小的 chunk\n");

    malloc(0x90);

    malloc(0x90);



    fprintf(stderr, "假设有个漏洞，可以把 victim->bk 的指针改写成 fake_chunk 的地址: %p\n",(void*)stack_var);

    chunk_lis[2][1] = (unsigned long)stack_var;

    fprintf(stderr, "现在 calloc 申请一个 0x90 大小的 chunk，他会把一个 smallbin 里的 chunk0 返回给我们，另一个 smallbin 的 chunk2 将会与 tcache 相连.\n");

    pp = calloc(1,0x90);

    

    fprintf(stderr, "这时候我们的 fake_chunk 已经放到了 tcache bin[0xa0] 这个链表中，它的 fd 指针现在指向下一个空闲的块: %p， bck->fd 已经变成了 libc 的地址: %p\n",(void*)stack_var[2],(void*)stack_var[4]);

    target = malloc(0x90);  

    fprintf(stderr, "再次 malloc 0x90 可以看到申请到了 fake_chunk: %p\n",(void*)target); 

    fprintf(stderr, "ojbk\n");

    return 0;

}
```

在`tcache`有剩余（不够 7 个）的时候，`smallbin`中的相同大小空闲块会放入`tcache`中，这时候也会出现`unlink`操作

**`calloc`在分配时不会用`tcache bin`的**

首先把 7 个放到`tcache bin`中，剩下两个放在`unsorted bin`中

`0x555555757250`是`chunk0`，`0x555555757390`是`chunk2`

![image.png](https://bbs.kanxue.com/upload/attach/202009/837755_5455SK2EKPEB6FJ.jpg "image.png")

这时候去申请一个`0xa0`大小的`chunk`，那俩在`unsorted bin`中的`chunk`整理放在了`small bin`中

![image.png](https://bbs.kanxue.com/upload/attach/202009/837755_V4PD8UHMZBZ93KW.jpg "image.png")

再去申请两个`0x90`大小的会从`tcache bin`中分配，这时候`tcache`还剩 5 个

![image.png](https://bbs.kanxue.com/upload/attach/202009/837755_Q4X7XF3NV56FQ7W.jpg "image.png")

然后把`chunk2`的`bk`改成`&stack_var`，在一开始是这样的

![image.png](https://bbs.kanxue.com/upload/attach/202009/837755_JK7WNDHYEXBXNUW.jpg "image.png")

改后：

![image.png](https://bbs.kanxue.com/upload/attach/202009/837755_ZAQH3Z9UTK52U8P.jpg "image.png")

另外此时的`bins`

![image.png](https://bbs.kanxue.com/upload/attach/202009/837755_QYW3KPB3PJMAGWN.jpg "image.png")

使用`calloc`去申请`0x90`大小的`chunk`会把`0x555555757250`申请出去，这时候如果`tcachebin`还有空闲的位置，剩下的`smallbin`从最后一个`0x7fffffffddc0`开始顺着`bk`链接到`tcachebin`中

[7b7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1j5i4W2B7N6h3&6Q4x3X3g2@1L8%4m8Q4x3V1j5J5x3o6t1H3i4K6u0r3x3o6u0Q4x3V1j5H3y4#2)9J5c8Y4y4E0j5h3I4D9j5X3W2F1在unlink的时候存在的漏洞/](https://bbs.kanxue.com/elink@4d5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1j5i4W2B7N6h3&6Q4x3X3g2@1L8%4m8Q4x3V1j5J5x3o6t1H3i4K6u0r3x3o6u0Q4x3V1j5H3y4#2)9J5c8Y4y4E0j5h3I4D9j5X3W2F1i4@1f1#2i4K6W2o6i4@1p5^5N6h3&6D9K9h3&6C8i4@1f1%4i4K6W2m8i4K6R3@1i4@1f1$3i4K6V1%4i4@1t1$3i4@1f1#2i4K6R3H3i4K6V1&6i4@1f1#2i4@1q4p5i4K6V1^5i4@1f1#2i4K6W2o6i4@1p5^5i4@1f1%4i4K6W2m8i4K6R3@1i4@1f1$3i4@1u0o6i4K6S2r3i4@1f1$3i4@1t1@1i4K6W2q4i4K6u0r3)

![image.png](https://bbs.kanxue.com/upload/attach/202009/837755_437VDZ72JJF5U2J.jpg "image.png")

`tcache`是后进先出的，所以这时候再去申请就拿到了`0x7fffffffddc0`这个地址的`chunk`

![image.png](https://bbs.kanxue.com/upload/attach/202009/837755_N2JFXKCZ87P5M8U.jpg "image.png")

参考：

278K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K6K9r3g2D9L8s2m8Z5K9i4y4Z5i4K6u0r3K9r3!0%4x3X3S2W2j5i4m8Q4x3V1k6@1M7X3g2W2i4K6u0r3L8h3q4K6N6r3g2J5i4K6u0r3k6$3I4A6j5X3y4Q4y4h3j5J5i4K6u0W2x3U0j5` .

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于  2020-9-8 12:09

被yichen115编辑

，原因： 加参考链接
