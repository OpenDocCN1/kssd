
# 硬盘清道夫 1.00

> 原文：[`www.pediy.com/kssd/pediy03/forum1235.htm`](https://www.pediy.com/kssd/pediy03/forum1235.htm)

《硬盘清道夫》的注册机

作者：`RATARICE[BCG]`

好高兴呀！终于做出了第一个注册机，虽然是极其简单的算法，但必定是我的第一次呀！希望大哥们不要笑话！

在这里要非常感谢破解勇大哥的鼎立支持！！！

```c
#include "stdio.h"
#include "string.h"

main() {
    char name[100];
    int nl, sit, a, b, c, d, i;
    
    clrscr();
    printf("************** FDISK CLEANER 1.00 crack by RATARICE[BCG]");
    printf("**************\n\n\n");
    printf("Please input you register name :\n");
    gets(name);
    
    nl = strlen(name);
    a = nl * 107;
    
    printf("\n");
    printf("The register code is :\n");
    
    for (i = 0; i < nl; i++) {
        b = 0;
        b = a + name[i];
        d = i + 1;
        c = b / d;
        printf("%d", c);
    }
}
```

`4dd466d0e1e0cc123afbe504da2462ca`
