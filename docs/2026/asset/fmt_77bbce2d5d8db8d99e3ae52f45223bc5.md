
<mark class="encrypted">cc0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8Y4q4I4i4K6g2X3y4o6p5@1y4e0x3J5z5o6g2Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8U0V1%4y4U0p5K6y4e0R3^5</mark>

通过这道题又复习和捋清楚了一些 `bin` 的存储方式：

主线程的 `Main_arena` 保存在 `libc.so` 的数据区里，其中包括了 `fastbinsY` 和 `bins`。

`fastbin` 和 `Unsortedbin` 是后进先出，其他 `bins` 是先进先出。

`fastbin` 只用到 `fd` 指针，后进来的 `chunk` 放在链表头。`fd` 指向上一个进来链表的节点。第一个进来的 `chunk` 的 `fd` 指向的是特殊的“0”，代表前面没有 `chunk`。

而且 `fastbin` 里面的 `chunk` 不会进行合并操作，只有当调用 `malloc_consolidate()` 的时候才会取出来与相邻的 `free chunk` 合并，所以 `fastbin` 的 `chunk` 的下一个 `chunk` 的 `PREV_INUSE` 始终为 1，处于使用状态。

以下我们通过两个参考程序：

参考程序：

```c
#include <unistd.h>
#include <stdlib.h>
#include <malloc.h>

int main()
{
    int size = 0x10;
    int size2 = 0x20;
    int *p1 = malloc(size);
    int *p2 = malloc(size);
    int *p3 = malloc(size2);
    sleep(0); // 只为了程序打断点，没有其他作用
    free(p1);
    free(p2);
    free(p3);
    return 0;
}
```

其他 `Bin` 如 `Unsorted bin`，用到 `fd` 和 `bk` 指针，是先进先出，后进来的 `chunk` 放在链表头。后进来的 `chunk` 的 `bk` 指向 `mainarena+88`，即 `unsortedbin` 数组的存储位置；`fd` 指向上一个进来链表的节点。第一个进来的 `chunk` 的 `fd` 指向 `mainarena+88`，即 `unsortedbin` 数组的存储位置。

参考程序：

```c
#include <unistd.h>
#include <stdlib.h>
#include <malloc.h>

int main()
{
    int size = 0x100;
    int *p1 = malloc(size);
    int *temp = malloc(size); // 防止 p1 与 p2 合并
    int *p2 = malloc(size);
    int *p3 = malloc(size); // 防止 p2 被 top chunk 合并
    sleep(0);
    free(p1);
    free(p2);
    return 0;
}
```

先把 `ld` 和 `Libc` 给换成题目给的：

```bash
patchelf --set-interpreter ./glibc-all-in-one/libs/2.26-0ubuntu2_amd64/ld-2.26.so --replace-needed ./glibc-all-in-one/libs/2.27-3ubuntu1_amd64/libc.so.6 ./glibc-all-in-one/libs/2.26-0ubuntu2_amd64/libc-2.26.so houseofAtum
```

# 程序分析

这里只进行一些简单的分析，其他的博客分析得很详细了：

```text
bigeast@ubuntu:~/Desktop/ctf$ ./houseofAtum
1. new
2. edit
3. delete
4. show
```

`main` 函数反编译如下：

```c
int __cdecl __noreturn main(int argc, const char **argv, const char **envp)
{
    int v3; // eax
    initialize(argc, argv, envp);
    while (1)
    {
        while (1)
        {
            while (1)
            {
                v3 = menu();
                if (v3 != 2)
                    break;
                edit();
            }
            if (v3 > 2)
                break;
            if (v3 != 1)
                goto LABEL_13;
            alloc();
        }
        if (v3 == 3)
        {
            del();
        }
        else
        {
            if (v3 != 4)
LABEL_13:
                exit(0);
            show();
        }
    }
}
```

`alloc` 函数如下：

```c
int alloc()
{
    int i; // [rsp+Ch] [rbp-4h]
    for (i = 0; i <= 1 && notes[i]; ++i)
        ;
    if (i == 2)
        return puts("Too many notes!");
    printf("Input the content:");
    notes[i] = malloc(0x48uLL);
    readn(notes[i], 72LL);
    return puts("Done!");
}
```

这里 `ull` 表示无符号长整形，`ll` 表示长整型，就是 8 字节。

这里 `72 = 0x48`，`72LL` 表示用 8 字节来存储 `72`。没有在字符串末尾添加 `\x00`，而且没有初始化，可能存在泄漏。利用 `visit()` 或者 `show()` 函数打印的时候就能泄漏了。

`del` 函数如下：

```c
unsigned __int64 del()
{
    int v1; // [rsp+0h] [rbp-10h]
    char v2[2]; // [rsp+6h] [rbp-Ah] BYREF
    unsigned __int64 v3; // [rsp+8h] [rbp-8h]

    v3 = __readfsqword(0x28u);
    printf("Input the idx:");
    v1 = getint();
    if (v1 >= 0 && v1 <= 1 && notes[v1])
    {
        free((void *)notes[v1]);
        printf("Clear?(y/n):");
        readn(v2, 2uLL);
        if (v2[0] == 121)
            notes[v1] = 0LL;
        puts("Done!");
    }
    else
    {
        puts("No such note!");
    }
    return __readfsqword(0x28u) ^ v3;
}
```

当在 `Clear?(y/n)` 中选择 `n` 的时候，不会清空 `notes` 数组中的指针，而 `edit` 和 `show` 都是通过这个来判断一个 `note` 是否存在。

## 漏洞利用的参考程序

参考链接：`8e1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0K9r3q4F1k6$3!0U0K9r3g2F1i4K6u0W2k6$3W2@1K9s2g2T1i4K6u0W2K9h3!0Q4x3V1j5J5x3o6p5^5i4K6u0V1x3e0q4Q4x3X3b7J5y4W2)9J5k6r3u0U0N6r3k6Q4x3X3b7J5x3o6p5^5i4K6u0W2K9s2c8E0L8l9`.

受上文的启发，虽然他的图画错了（头节点不应该指向其 `fd` 而应该指向 `chunk` 头）。

### 实验该参考程序过程发现了一个小现象：

- 当 `malloc(0x20)`，分配的 `chunk` 的 `size` 为 `0x21`；
- 当 `malloc(0x28)`，分配的 `chunk` 的 `size` 为 `0x21`；
- 当 `malloc(0x29)`，分配的 `chunk` 的 `size` 为 `0x41`。

`0x20 = 32` 字节，是分配一个 `chunk` 的最小空间 = `presize + size + fd + bk = 32` 字节。`size` 后面多 1 表示上一个 `chunk` 的状态。可以看到当 `malloc(0x28)`，显示的 `size` 仍然为 `0x21`，肯定是和后一个 `chunk` 的 `presize` 复用了。

```c
#include <unistd.h>
#include <stdlib.h>
#include <malloc.h>

void main() {
    void *a = malloc(0x28);
    void *b = malloc(0x28);
    // fill the tcache
    for (int i = 0; i < 7; i++) {
        free(a);
    }
    sleep(0);
    free(b); // fast bin
    // What will happen with this:
    free(a); // fast bin
}
```

`free(b)` 后：

```text
pwndbg> heap
Allocated chunk | PREV_INUSE
Addr: 0x555555757000
Size: 0x251

Free chunk (tcache) | PREV_INUSE
Addr: 0x555555757250
Size: 0x31
fd: 0x555555757260

Free chunk (fastbins) | PREV_INUSE
Addr: 0x555555757280
Size: 0x31
fd: 0x00

Top chunk | PREV_INUSE
Addr: 0x5555557572b0
Size: 0x20d51

pwndbg> bins
tcachebins
0x30 [7]: 0x555555757260 ◂— 0x555555757260 /* '`ruUUU' */
fastbins
0x20: 0x0
0x30: 0x555555757280 ◂— 0x0
0x40: 0x0
0x50: 0x0
0x60: 0x0
0x70: 0x0
0x80: 0x0
unsortedbin
all: 0x0
smallbins
empty
largebins
empty
```

`free(a)` 之后：

发现 `a` 也进了 `fastbin` 里面，而且其 `fd` 指向了 `b` 的 `presize` 字段。这样我们就可以通过 `malloc` 从 `tcache bin` 里得到 `a` 的 `fd`，从而修改 `b` 的 `presize`，甚至 `presize` 后面的 `size` 等内容。

```text
pwndbg> heap
Allocated chunk | PREV_INUSE
Addr: 0x555555757000
Size: 0x251

Free chunk (fastbins) | PREV_INUSE
Addr: 0x555555757250
Size: 0x31
fd: 0x555555757280

Free chunk (fastbins) | PREV_INUSE
Addr: 0x555555757280
Size: 0x31
fd: 0x00

Top chunk | PREV_INUSE
Addr: 0x5555557572b0
Size: 0x20d51

pwndbg> bins
tcachebins
0x30 [7]: 0x555555757260 —▸ 0x555555757280 ◂— 0x0
fastbins
0x20: 0x0
0x30: 0x555555757250 —▸ 0x555555757280 ◂— 0x0
0x40: 0x0
0x50: 0x0
0x60: 0x0
0x70: 0x0
0x80: 0x0
unsortedbin
all: 0x0
smallbins
empty
largebins
empty
```

## 漏洞利用思路

我们的目标是最终执行 `system('/bin/sh')`，而且是通过堆来完成。在《HITB CTF 2018 gundam》中，我们通过动态获得 `libc` 基地址，进而计算出 `_free_hook` 地址和 `system` 地址，想方设法在 `_free_hook` 地址处写入 `system` 地址，再创建 1 个内容为 `'/bin/sh'` 的 `chunk`，然后释放，就可以触发 `_free_hook`，最终执行 `system('/bin/sh')`。这道题同样可以采取这种思路。

1. 要动态获得 `libc` 基地址，就要用到 `unsorted bin`，在 `tcache` 的 `count` 为 7 的情况下，将符合 `unsorted bin` 大小的 `chunk` 释放到 `unsorted bin` 中。该 `chunk` 的前向指针 `fd` 和后向指针 `bk` 的值就是要泄露的地址（详细分析见上一篇《HITB CTF 2018 gundam 分析》）。
2. 题目中创建的 `house of Atum` 的 `chunk` 大小为 `0x51`。显然，这样 `chunk` 释放后只能进入 `tcache bin` 和 `fastbin`，要想使其进入 `unsorted bin`，就得改变指定 `chunk` 的大小。
3. 要在 `_free_hook` 地址处写入 `system` 地址，就得构建 1 个以 `_free_hook` 地址为数据区地址的 `chunk`，即 `_free_hook-0x10` 为起始地址的 `chunk`，以 `system` 地址作为内容参数。

以下分析和调试过程基于以上 3 点考虑，通过对堆块 `chunk` 的灵活操作，成功执行并获得 shell。

### 泄露 Chunk 的 `fd` 地址

连续释放同一个 `chunk` 7 次后，此时通过 `show` 即可获得 `chunk0` 的 `fd` 的地址，书本中记为 `heap_addr`：

```text
tcachebin[7] -> chunk0.fd <- chunk0.fd
fastbin[] ：null
```

### 伪造 chunk

动态获得了 `heap_addr`，即 `chunk0` 的 `next` 指针，后面该如何利用？
