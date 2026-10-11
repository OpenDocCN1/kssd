
# 【分享+讨论】对 XOR_password 及 XOR_cryptanalyiss 等相关论文研讨。(No. 2)

> 原文：[`www.pediy.com/kssd/pediy10/98540.html`](https://www.pediy.com/kssd/pediy10/98540.html)

这篇论文即将发表在台湾唯一一个 `information security` 的期刊，信息安全通讯 15 卷第 4 期，出刊时间约为 2009 年 10 月 10 日前后。

以下是论文的内容。

![](img/46080ef561d55b4d5e8815155443cd71.jpg)

以下的内容是实验的结果，根据论文的算法，实际以 `Language C` 设计出来，最后提出我们的攻击法，来证明这个算法是存在漏洞。

![](img/3b6b1f41aa2bde035f67771031e797ba.jpg)

由以上内容知道，本方法确实比 [【分享】An improved signature scheme without using one-way Hash functions](http://bbs.pediy.com/showthread.php?p=684694#post684694) 6 楼那篇论文 `Cryptanalysis of a signature scheme without using one-way Hash functions.pdf` 的方法还更有效率。

这是 source code，主要是验证我们的推论，各位可以参考上面的说明；当初没有优化，简单易懂，请见谅。

代码:

```
#include <stdio.h>
#include <windows.h>
#include <stdlib.h>
#include <math.h>

long GetPowAndMod(long g, long x, long p);

int main()
{
    long g, x, p, y, m, k, s, r, t, gsk, j, M;
    printf("Input g, x, p (Separated By Space) : ");
    scanf("%ld%ld%ld", &g, &x, &p);
    fflush(stdin);
    y = GetPowAndMod(g, x, p) % p;
    printf("y : %ld\r\n", y);

    printf("Input m : ");
    scanf("%ld", &m);
    m = m % (p - 1);
    s = GetPowAndMod((y + m), m, p) % p;
    printf("s : %ld\r\n", s);

    printf("Input k : ");
    scanf("%ld", &k);
    gsk = GetPowAndMod(g, k, p) % p;
    for (j = 1 ; ; j++)
        if ((j * gsk % p) == 1)
            break;
    r = (m * s * j) % p;
    printf("r : %ld\r\n", r);

    for (j = 1 ; ; j++)
        if ((j * x % (p - 1)) == 1)
            break;
    t = ((((j * (k - (r ^ s))) % (p - 1))) - s) % p;
    
    if (t < 0)
    {
        t += (p - 1);
        printf("t : %ld\r\n", t % p);
    }
    else
    {
        printf("t : %ld\r\n", t % p);
    }
    
    for (j = 1 ; ; j++)
        if ((j * s % p) == 1)
            break;
    M = (((((GetPowAndMod(y, s + t, p) * r) % p) * GetPowAndMod(g, r ^ s, p)) % p) * j) % p;
    printf("M : %ld\r\n", M);

    t = 2 * s + t;
    r = -r;
    s = -s;
    for (j = 1 ; ; j++)
        if ((j * -s % p) == 1)
            break;
    M = (((((GetPowAndMod(y, s + t, p) * -r) % p) * GetPowAndMod(g, r ^ s, p)) % p) * j) % p;
    printf("M' : %ld\r\n", M);

    printf("Input r', s', t' (Separated By Space) : ");
    scanf("%ld%ld%ld", &r, &s, &t);
    for (j = 1 ; ; j++)
        if ((j * -s % p) == 1)
            break;
    M = (((((GetPowAndMod(y, s + t, p) * -r) % p) * GetPowAndMod(g, r ^ s, p)) % p) * j) % p;
    printf("M'' : %ld\r\n", M);

    system("pause");
        return 0;
}

long GetPowAndMod(long g, long x, long p)
{
    long Temp = g;

    if (x > 2)
        return (GetPowAndMod(g, x - 2, p) % p) * (g * g % p);
    else
    {
        for (long j = 1 ; j < x ; j++)
            Temp *= g;
        return Temp;
    }
}
```
