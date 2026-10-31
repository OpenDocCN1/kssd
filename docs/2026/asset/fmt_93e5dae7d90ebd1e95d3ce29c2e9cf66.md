
| 1  2  3  4  5  6  7  8  9  10  11 | `xxxx``-``xxxx r``-``-``p` `00000000` `08``:``30` `10865`                     `user_process`  `xxxx``-``xxxx r``-``xp` `00001000` `08``:``30` `10865`                     `user_process`  `xxxx``-``xxxx r``-``-``p` `00002000` `08``:``30` `10865`                     `user_process`  `xxxx``-``xxxx rw``-``p` `00002000` `08``:``30` `10865`                     `user_process`  `7ffff7fbf000``-``7ffff7fc3000` `r``-``-``p` `00000000` `00``:``00` `0`         `[vvar]`  `7ffff7fc3000``-``7ffff7fc5000` `r``-``xp` `00000000` `00``:``00` `0`         `[vdso]`  `7ffff7fc5000``-``7ffff7fc6000` `r``-``-``p` `00000000` `08``:``30` `12512`     `ld``-``linux``-``x86``-``64.so``.``2`  `7ffff7fc6000``-``7ffff7ff1000` `r``-``xp` `00001000` `08``:``30` `12512`     `ld``-``linux``-``x86``-``64.so``.``2`  `7ffff7ff1000``-``7ffff7ffb000` `r``-``-``p` `0002c000` `08``:``30` `12512`     `ld``-``linux``-``x86``-``64.so``.``2`  `7ffff7ffb000``-``7ffff7fff000` `rw``-``p` `00036000` `08``:``30` `12512`     `ld``-``linux``-``x86``-``64.so``.``2`  `7ffffffde000``-``7ffffffff000` `rw``-``p` `00000000` `00``:``00` `0`         `[stack]` |

这几个部分完成映射后，LD会开始通过`mmap`映射其他的部分，其中主要的部分就是程序依赖的动态链接库。
要知道，Linux下查找动态链接库除了依赖`ld.so.conf`中配置的路径外，还会依赖缓存`/etc/ld.so.cache`，LD在自己运行起来后，会将文件`/etc/ld.so.cache`也映射到虚拟文件系统中，用于辅助查找动态链接库。

| 1  2  3  4  5 | `7ffff7c00000``-``7ffff7e12000` `r``-``-``p` `00000000` `08``:``30` `12715`     `/``usr``/``lib``/``x86_64``-``linux``-``gnu``/``libc.so.``6`  `7ffff7fb7000``-``7ffff7fbd000` `r``-``-``p` `00000000` `08``:``30` `433`       `/``etc``/``ld.so.cache`  `7ffff7fbd000``-``7ffff7fbf000` `rw``-``p` `00000000` `00``:``00` `0`  `7ffff7fbf000``-``7ffff7fc3000` `r``-``-``p` `00000000` `00``:``00` `0`         `[vvar]`  `7ffff7fc3000``-``7ffff7fc5000` `r``-``xp` `00000000` `00``:``00` `0`         `[vdso]` |

不过呢，文件`/etc/ld.so.cache`在动态链接库查找任务完成之后，会通过`unmmap`释放掉，这就导致动态链接库和LD间的内存不是连续的，当程序首次通过`mmap`申请内存时，可能会从获得一个动态链接库和LD间的空闲内存，而之后通过`mmap`申请得到的内存，则会位于动态链接库的下方。
所以想要控制`chunk A`的`mchunk_size`来覆盖`chunk B`，造成`chunk A`释放连带着`chunk B`一块释放时，你需要保证`chunk A`和`chunk B`的内存区域是相连的，不然的话，你很有可能会将`libc.so.6`或其他动态连接库的内存释放掉，造成程序正常使用动态链接库时出现段错误。

# 示例讲解

上面给出了程序的源代码。

```
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28  29  30  31  32  33  34  35  36  37  38  39  40  41  42  43  44  45  46  47  48  49  50  51  52  53  54  55  56  57  58  59  60  61  62  63  64  65  66  67  68  69  70 | `#include <stdlib.h>`  `#include <stdio.h>`  `#include <stdbool.h>`  `#include <unistd.h>`    `#define MAX_READ_LEN            0x30`  `#define MMAP_ALLOC_LEN          0x100000`  `#define CHUNK_HDR_LEN           0x10`  `#define FLAGS_CLEAR             0xFFFFFFFFFD`    `#define SAFE_FREE(x)            do {                \`  `if` `(x)          \`  `free(x);    \`  `x` `=` `NULL;       \`  `}` `while``(``0``)`    `static void vuln(``long``*` `ptr)`  `{`  `unsigned` `long` `chunk_len, write_start_addr, end_addr;`  `int` `idx;`    `idx` `=` `0``;`    `printf(``"please input index\n"``);`  `scanf(``"%d"``, &idx);`    `chunk_len` `=` `ptr[``-``1``] & FLAGS_CLEAR;`  `write_start_addr` `=` `(unsigned` `long``)ptr` `+` `idx` `*` `sizeof(``long``);`  `end_addr` `=` `(unsigned` `long``)ptr` `+` `chunk_len` `-` `CHUNK_HDR_LEN;`    `if` `(write_start_addr >``=` `end_addr) {`  `return``;`  `}`    `printf(``"please input something\n"``);`  `read(STDIN_FILENO, &ptr[idx], sizeof(``long``));`  `}`    `int` `main(void)`  `{`  `long``*` `poniter_1,` `*` `poniter_2,` `*` `poniter_3,` `*` `poniter_4;`  `bool``*` `check_flag;`    `poniter_1` `=` `(``long``*``)malloc(MMAP_ALLOC_LEN);`  `printf(``"prt1 addr = 0x%lx, chunk len = 0x%lx\n"``, (unsigned` `long``)poniter_1, poniter_1[``-``1``]);`    `poniter_2` `=` `(``long``*``)malloc(MMAP_ALLOC_LEN);`  `printf(``"prt2 addr = 0x%lx, chunk len = 0x%lx\n"``, (unsigned` `long``)poniter_2, poniter_2[``-``1``]);`    `check_flag` `=` `(``bool``*``)poniter_2;`  `*``check_flag` `=` `false;`    `poniter_3` `=` `(``long``*``)malloc(MMAP_ALLOC_LEN);`  `printf(``"prt3 addr = 0x%lx, chunk len = 0x%lx\n"``, (unsigned` `long``)poniter_3, poniter_3[``-``1``]);`    `vuln(poniter_3);`  `printf(``"ptr3 new chunk size = 0x%lx\n"``, poniter_3[``-``1``]);`  `SAFE_FREE(poniter_3);`    `poniter_4` `=` `(``long``*``)malloc(``3` `*` `MMAP_ALLOC_LEN);`  `printf(``"prt4 addr = 0x%lx, chunk len = 0x%lx\n"``, (unsigned` `long``)poniter_4, poniter_4[``-``1``]);`    `vuln(poniter_4);`    `if` `(``*``check_flag) {`  `system(``"/bin/sh"``);`  `}`    `return` `0``;`  `}` |
```

从程序的源代码中可以看到，程序通过`malloc`触发了四次`mmap`，其中`poniter_3`是有机会篡改`mchunk_size`的，因为函数`vuln`中只检查了上界，但没有检查下界，数组索引值可以是负数的，当索引值为1时，`ptr[idx]`指向的就是`mchunk_size`字段所在的位置，篡改`poniter_3`的`mchunk_size`，可以将`poniter_2`一块释放，当`poniter_4`申请时，就有机会申请到和`poniter_2`重叠的区域。
`poniter_2`上存放着一个名为`check_flag`的变量，该变量可以控制程序执行`system("/bin/sh")`，只不过`check_flag`在默认情况下永远都是`false`的状态，但当`poniter_4`和`poniter_2`重叠时，就有机会改写`check_flag`为`true`的状态，导致程序执行`system("/bin/sh")`。

## exploit构造

根据上方的分析构造出下面的exploit。

|  |  |

| --- | --- |
