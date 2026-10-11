
# System View 4.05

> 原文：[`www.pediy.com/kssd/pediy03/forum422.htm`](https://www.pediy.com/kssd/pediy03/forum422.htm)

本人近来使用一款在市面不常用的软件叫 `System View 4.05`，`www.elanix.com/eval.htm` 下载的。但是它是评估版，有日期限制。当使用过期后，重新安装也不行，必须重新格式化才能重新安装，真烦啊，有谁能破它啊，感谢啊！

刚才把表格长度搞错了，应为 `0x32`，不是 `32` :(

```c
#include <windows.h>
#include <stdio.h>

void main(void)
{
    long Table[0x32] =
    {
        0x00011F1B, 0x0000E298, 0x0000F2CC, 0x00008CDC,
        0x00009137, 0x0001376E, 0x00002BFD, 0x00013281,
        0x00014568, 0x00012055, 0x00003701, 0x0000B89E,
        0x00015653, 0x00013CF7, 0x0000AA62, 0x0001793F,
        0x0001596E, 0x00003AD5, 0x000174E4, 0x0000A709,
        0x0000DF96, 0x000134C0, 0x000039DF, 0x0000F759,
        0x0000CBD7, 0x00008FE2, 0x000101FA, 0x00010ACF,
        0x000083CD, 0x00008944, 0x00014ACA, 0x000148F6,
        0x0000F630, 0x000181BC, 0x00016752, 0x000076D1,
        0x00011B70, 0x00017F98, 0x00007CD1, 0x0000E2C0,
        0x00004C75, 0x0001866B, 0x000114C7, 0x00002C95,
        0x0000F146, 0x00004A3C, 0x00004B48, 0x00013FEB,
        0x00008B13, 0x0000371C
    };

    long random, Remain, sn, passwd[3], Index;

    printf("Input your S/N: ");
    scanf("%lu", &sn);
    random = GetTickCount();

    passwd[0] = 0x0000DCE7L;
    Index     = sn % 0x32L;
    Remain    = random % 0x000000B4L;
    passwd[1] = Remain + Table[Index];
    Remain    =  Table[Index] + Remain;
    passwd[2] =  Remain + 0x0001B14FL;

    printf("Your password is: ");
    for(Index = 0; Index < 3; Index++)
    {
        printf("%lu", passwd[Index]);
        if (Index == 2)  printf("\n");
        else             printf("-");
    }
}
```

好象过期之后再输入一次注册码就可以继续用了。

### `blowfish`

试试这两个断点：

1. `__vbaInStr( )`  
   相当于 C 函数 `strstr( )`。它用这个函数判断注册码是否包含“-”字符。
2. `__vbaI4Str( )`  
   相当于 C 函数 `atol( )`。   `4dd466d0e1e0cc123afbe504da2462ca`
