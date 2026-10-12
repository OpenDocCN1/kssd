
# 标题：

> 原文：[`www.pediy.com/kssd/pediy12/129521.html`](https://www.pediy.com/kssd/pediy12/129521.html)

PE Explorer 相信大家都知道这个软件吧!  

这是官方的介绍:  

Here's what's new in PE Explorer 1.95  

General:  

• Added the Remove Debug Information Tool.  

• Added the Remove Relocations Tool.  

Disassembler:  

• Now supports the SSE3 instruction set.  

• Upgraded the algorithm for detecting Unicode strings.  

• Improved the data analysis algorithm to produce a higher quality disassembly listing.  

Resource editor:  

• Bugfix: TypeLibs in the multi-language resources would occassionally display incorrectly.  

[点击下载](http://www.heaventools.com/download/pexsetup.exe)![`bbs.pediy.com/upload/image/?眵靡.JPG_83478.jpg`](http://bbs.pediy.com/upload/image/?眵靡.JPG_83478.jpg) ![`bbs.pediy.com/upload/image/未命名.JPG_83478.jpg`](http://bbs.pediy.com/upload/image/未命名.JPG_83478.jpg)

Montgomery 乘法是公钥算法实现中的一个核心算法，其主要作用是为模乘运算加速。

在公钥算法实现中，通常需要计算 `a mod M`、`a*b mod M`、`a^b mod M` 等，一般看见 `mod M`，最直接想到的当然是除法，可是除法运算慢且实现难，于是，就有人发明了一种不需要计算除法的计算模的方法，这就是所谓的 Montgomery 乘法。

Montgomery 乘法的数学表达是 `A*B*R^(-1) mod M`，其中，`A`、`B` 是与 `M` 同位长的大数，`R = 2^bitlen(bitlen 指 M 位长)`，`R^(-1)`是指 `R` 相对于 `M` 的模逆，即 `R^(-1)`是满足如下条件的数：`R*R^(-1) mod M = 1`；这个条件成立的充要条件是 `R` 与 `M` 互素，这一点只需要 `M` 为奇数即可，所以 Montgomery 乘法通常适用于对奇数求模。

数学最烦人也是最忌讳的是只见形式不懂实质；以下为你慢慢揭示 Montgomery 乘法的内在涵义。

从一个除法例子说起吧。

例如要计算 11111111 除以 1011（二进制表示），写出其除法过程如下：

```
1011| 11111111| 1  

        | 1011        |  

——————————————  

        1001111| 0  

            0000      |  

——————————————  

        1001111| 1  

              1011    |  

——————————————  

          100011| 0  

              0000    |  

——————————————  

          100011| 1  

               1011   |  

——————————————  

            1101 | 1  

                 1011  |        

——————————————  

             10
```

计算过程本质上是将被除数从高位归 0 直至余数小于除数的过程，归 0 过程的规则是要使用被除数减除数。例如在第一步中，是将 1111 减去 1011，即把被除数最高位归成 0，然后将余做 100 作高位与被除数低位 1111 拼成新的被除数，如此迭代。

现在来做一种新的运算。这种新运算与之前除法有两点相反：一是从低位归 0，二是归 0 规则使用加法，其运算过程如下：

```
       11111111| 1011  

                   1011|  

————————————  

     100001010|  

                 1011  |  

————————————  

     100100000|
```

从低位归 0 需要“除数”是奇数，否则，当“被除数”是奇数时，“被除数”的最低位总是无法归成 0。

低位归 0 方法总可以迭代到“被除数”的非 0 高位与“除数”相差不大，上例中，当低位归到 5 个 0 时，“被除数”的高位变为 1001，舍弃低位的 5 个 0，把高位的 1001 当做“余数”；通常，传统的 Montgomery 乘法归 0 的个数是与模数位长相等的，所以这里我们归掉 4 个 0，然后将 10010 减去 1011，得到的 111 作为“余数”。

比较上述 2 种运算的运算量，可以发现：从高位归 0 需要额外使用“比较”操作，因为需要确保每次归 0 时被除数大于除数。

以下来揭示第二种“除法”的涵义。

上述第二种“除法”实际是在被除数上加了若干除数，使得被除数的低位变为 0，而高位与除数接近（所以也可以让其比除数小），将高位当做余数。用 `S` 记被除数，`M` 记除数，`q` 记累加 `M` 的次数，`R` 记 `2^bitlen`，其中 `bitlen` 指归 0 的个数，则余数的表达式为：（`S+qM`）/`R`。

当“除数”是奇数时，“被除数”的低位总能被归成 0，所以，适当选取 `q`，总可以使得（`S+qM`）/`R` 为一个小于除数的数。

`S` 与 `S+qM` 是什么关系呢？显然，`S` 和 `S+qM` 对 `M` 是同余的。所以表达式（`S+qM`）/`R` 的涵义可以理解为：在与 `S` 同余的数中，选取一个低位有 `bitlen` 个 0，且高位小于 `M` 的数，去掉该数低位的 `bitlen` 个 0，所得的数即为（`S+qM`）/`R`。

如果了解一些模和群的知识，表达式（`S+qM`）/`R` 写成 `S*R^(-1)`就更简单，形式上的意思就变为：剩余类 `S` 与剩余类 `R^(-1)`的乘积。

对于 Montgomery 乘法 `A*B*R^(-1) mod M`，将 `S` 看做 `A*B`，涵义自然就比较清晰了。

使用 Montgomery 乘法代替 `A*B mod M` 必须要做到功能能替代 `A*B mod M`，且要有其速度上的优势，否则 Montgomery 乘法就没有存在的必要了。

先来看速度上的优势。

通过前面的说明可以知道：从低位归 0 比从高位归 0 每次迭代可以省去大数的比较操作，这是 Montgomery 乘法速度上的优势之一。

更为重要的是，Montgomery 乘法可以做大量的并行运算，而 `A*B mod M` 则必须先将 `A*B` 计算出来再计算除法。所以当使用硬件实现 Montgomery 乘法会有很高的效率。

如何使用 `A*B*R^(-1) mod M` 代替 `A*B mod M` 呢？

通常，将 `x` 映射成 `x*R mod M`，后者可以称之为 `x` 的 Montgomery 表示，可以发现，输入两个 Montgomery 表示下的乘数，使用 Montgomery 乘法运算，可以得到原乘数使用普通模乘得到的数的 Montgomery 表示。所以，要计算模乘运算，可以先将乘数化成 Montgomery 表示，然后使用 Montgomery 乘法，最后将结果脱掉 Montgomery 表示，即得普通模乘的结果；这个过程虽然复杂些，但是对于计算大量模乘的运算（例如模幂）而言，只是多了初始的入和出 Montgomery 表示的过程，但中间每次计算 Montgomery 乘法，却比计算普通模乘省下大量时间。

要将 `x` 写成 Montgomery 表示，可以计算 `x` 与 `R²` 的 Montgomery 乘法，要脱掉 Montgomery 乘法，可以计算 `x` 与 `1` 的 Montgomery 乘法。由于 `R²` 是常数，可以提前计算。

Montgomery 乘法目前常常被使用于 RSA 和 ECC 中，智能卡芯片通常使用硬件实现 Montgomery 乘法，软件实现 RSA 和 ECC 功能即可满足一般终端的使用需求。
