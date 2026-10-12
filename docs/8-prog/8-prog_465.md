
# 计算素数

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-580.htm`](https://www.pediy.com/kssd/pediy09/pediy09-580.htm)

## 背景

`kflnig`

在《看雪精华 7》中看到一篇关于大家都在算素数的文章，很好玩，于是乎，我自己也来试试，看看能否打破记录。我本是只菜鸟，复杂一点点的算法，就不会。好了，我们还是来看看吧。在写这篇文章前，我还不知道怎么使用 `GetTickCount()` 这个 API 呢。

我就直接贴个程序了。那篇文章里最后一个家伙恐怖的说：“计算到 260 万，340ms。”不知道他的 CPU 是什么，我的是 32 位菜羊 2.66GHz。很老的一个型号了。好久好久没有编程了，甚至快连编一个求素数的都快不会了。

首先把 `menglv` 的 VB 程序写的东西 C++化，假设他们的机器和我的相同。发现 VB 和 C++的执行速度要差上好几倍。2—10000，他在 VB 里 203 毫秒。我这边有时是 16ms 有时是 0ms。把他的数据提高到 2??一百万。那么时间是 1.9s。这绝对是不行的。

## 初始算法

我的算法运用了一个定理，以前看《离散数学》的时候看来的，学以致用嘛，大概就是说，若一个数 `n` 不能被 `sqrt(n)` 之内的素数整除，那么这个数就是素数。写好之后计算了一下时间，看来我只能来当反面教材了。我的代码如果写错了，大家告诉我。

```cpp
#include <iostream.h>
#include <windows.h>
#include <math.h>

void main()
{
    long t1, t2, n1 = 12, n2, n3, ss[200000];
    bool fg = false;
    t1 = GetTickCount();
    ss[0] = 2;  ss[1] = 3;  ss[2] = 5;  ss[3] = 7;  ss[4] = 11;  n3 = 4;

    while (n1 <= 1000000)
    {
        for (n2 = 0; n2 <= n3; n2++)
        {
            if (sqrt(n1) < ss[n2]) break;
            if (n1 % ss[n2] == 0)  { fg = false; break; }
            else fg = true;
        }
        if (fg == true) { n3++; ss[n3] = n1; fg = false; }
        n1++;
    }

    t2 = GetTickCount();
    cout << t2 - t1;
}
```

约 600ms 大一点。天啊！怎么可以，其实只要改一个语句就可以把时间缩短到 275ms 左右。一半时间啊！

将 `if (sqrt(n1) < ss[n2]) break;` 修改为 `if (n1 < ss[n2] * ss[n2]) break;`

惊奇吧！只要改这么一个语句就够了哟！以后大家要注意了可以乘解决的绝对不开方。

可是那个最嚣张的家伙说“计算到 1001989，使用时间 120ms。”如果它的机器不是大型机那么就说明，我的代码还欠优化。显然，`if (n1 < ss[n2] * ss[n2]) break;` 这句语句还是不够理想。可是这句实在是无法优化。我们必须得等价地实现一种方法。

## 优化尝试

这里的判断太要人命了。我们要设法减少。可是这句话又关系着下面一句的实现。这句话的繁，使得下面一句做了最少的运算。于是乎采取了一个折衷的算法。就是分两步计算。先算 1000 以内的素数，然后求得 1000*1000 内的素数。

```cpp
#include <iostream.h>
#include <windows.h>
#include <math.h>

void main()
{
    long t1, t2, n1 = 12, n2, n3, num, ss[200000];
    bool fg = false;
    t1 = GetTickCount();
    ss[0] = 2;  ss[1] = 3;  ss[2] = 5;  ss[3] = 7;  ss[4] = 11;  n3 = 4;

    while (n1 <= 1000)
    {
        for (n2 = 0; n2 <= n3; n2++)
        {
            if (n1 < ss[n2] * ss[n2]) break;
            if (n1 % ss[n2] == 0)  { fg = false; break; }
            else fg = true;
        }
        if (fg == true) { n3++; ss[n3] = n1; fg = false; }
        n1++;
    }

    num = n3;

    while (n1 < 1000000)
    {
        for (n2 = 0; n2 < num; n2++)
        {
            if (n1 % ss[n2] == 0)  { fg = false; break; }
            else { fg = true; }
        }
        if (fg == true) { n3++; ss[n3] = n1; fg = false; }
        n1++;
    }

    t2 = GetTickCount();
    cout << t2 - t1;
}
```

不优化则已，一优化时间哗的涨到 350ms。印证了我“可是这句话又关系着下面一句的实现。这句话的繁，使得下面一句做了最少的运算。”这句话。

怎么办？

再想办法。不考虑算法，突然想到，C 语言中可以有寄存器变量。由于外层循环很大我们可以定义一个寄存器变量。我们把 `n1` 定义成 `register n1;` 这个模样。还是没有多大变化。只是，在多次运行中出现了最短时间为 265ms 了。

再稍微改一下。

```cpp
n1 = 12; n3 = 4; num = 4;
while (n1 <= 1000000)
{
    if (n1 > ss[num] * ss[num]) num++; // 这里变了
    for (n2 = 0; n2 <= num; n2++)
    {
        if (n1 % ss[n2] == 0)  { fg = false; break; }
        else fg = true;
    }
    if (fg == true) { n3++; ss[n3] = n1; fg = false; }
    n1++;
}
```

稍微又比刚才好了一点点。缩短了 10ms。

我把我自己的方法的最好只能是这样了。好了，上面的就是我的所得了。可惜我实在无法再进一步优化了。点击 VC 中的那个“执行”和直接在 CMD 中执行会差上 10ms 左右。

`kflnig` 极限：265ms。可是据说 `GetTickCount()` 的精度不高。可能还要稍微高一点。我到 200 万数据已经和那位帅哥没得比了。所以不说了。

本来还想装一下酷，来个费马小定理。想想，作罢了。

## 精确计时方法

`GetTickCount` 表面是能精确到 1ms，实际情况上，大部分是 15ms。

程序执行效率的提高，首先是在算法的优化上，其次是代码结构之类的优化，最后才是机器指令的优化。

另外有两种更精确的方法：

```cpp
double ClockRecorder()
{
    static LARGE_INTEGER liTmp;
    static double dFrequency, dStrt = 0.0, dEnd = 0.0, dLast = 0.0;
    static bool bStart = true;

    ::QueryPerformanceCounter(&liTmp);
    if (bStart)
    {
        dStrt = double(liTmp.QuadPart);
        dLast = 0.0;
        bStart = false;
    }
    else
    {
        dEnd = double(liTmp.QuadPart);
        ::QueryPerformanceFrequency(&liTmp);
        dFrequency = double(liTmp.QuadPart);
        dLast = (dEnd - dStrt) / dFrequency;
        bStart = true;
    }
    return dLast * 1000;
}

inline unsigned __int64 GetCycleCount()
{
    unsigned __int64 retvl = 0;

    __asm _emit 0x0F
    __asm _emit 0x31
    __asm mov dword ptr retvl, eax

    return retvl;
}

unsigned __int64 CircleRecorder()
{
    static unsigned __int64 dLast = 0;
    static bool bStrt = true;
    unsigned __int64 temp = bStrt ? GetCycleCount() : dLast;
    dLast = GetCycleCount();
    bStrt = !bStrt;
    return dLast - temp;
}
```

切记：算法是最重要的。

## 高性能素数筛选算法

下面是我的精确计算 1000 万以内的素数的算法，看看速度如何。![](img/7c79845954f5fbf9e981f570ed213a8d.jpg "扮鬼脸")

```cpp
#include <stdio.h>

int searchprime(int num, int *out)
{
    int pn = 0;
    bool *mask = new bool[num + 1];
    for (int i = 2; i <= num; i++)
        if (!mask[i])
        {
            out[pn++] = i;
            if (i < 32767)
                for (int j = i * i; j <= num; j += i)
                    mask[j] = true;
        }
    delete[] mask;
    return pn;
}

void main()
{
    int *out = new int[664579];
    printf("搜索 1 ~ 10000000......");
    int num = searchprime(10000000, out);
    printf("找到%d 个素数,最后一个素数是%d.\n", num, out[num - 1]);
    delete[] out;
}
```

P4 2.5GHz: 100 万以内 0.1s, 1000 万以内 1.25s.

```cpp
int pn = 0;
bool *mask = new bool[num + 1];
memset(mask, 0, sizeof(bool) * (num + 1));
```

引用: 最初由 `jjnet` 发布

```cpp
int pn = 0;
bool *mask = new bool[num + 1];
memset(mask, 0, sizeof(bool) * (num + 1));
```

嗯，严格地说应该清 0。

我贴的那个程序原来是我用来和 Java 语言比较速度的，因为 Java 是肯定在 `new` 时清 0 的，所以 C++ 版本为了和 Java 代码保持一致就没有清 0，但实际运行得到的空间总是清过 0 的，原因不明。转过来的时候忘了这个问题了。![](img/4ec4a010bb6e6d79731981cd94fbb640.jpg "尴尬")

BTW，Java 1.42 运行的时间仅比 C++ 版的慢 1/4 左右，看来 Java 在小范围密集运算这方面还是有点优势的。

引用: 最初由 `dwing` 发布

嗯，严格地说应该清 0。

我贴的那个程序原来是我用来和 Java 语言比较速度的，因为 Java 是肯定在 `new` 时清 0 的，所以 C++ 版本为了和 Java 代码保持一致就没有清 0，但实际运行得到的空间总是清过 0 的，原因不明。转过来的时候忘了这个问题了。![](img/4ec4a010bb6e6d79731981cd94fbb640.jpg "尴尬")

BTW，Java 1.42 运行的时间仅比 C++ 版的慢 1/4 左右，看来 Java 在小范围密集运算这方面还是有点优势的。

C++ 并没有规定一定清 0，跟 heap 的管理有关。

比如在 VC 的 debug 模式下，一般都填 CD。

所以还是加上好。
