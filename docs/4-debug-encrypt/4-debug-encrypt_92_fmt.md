
# 怎样攻破 RSA-1024 的算法保护 ?(翻译部分)

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-705.htm`](https://www.pediy.com/kssd/pediy07/pediy7-705.htm)

## 1 介绍

为了学点基础知识,我决定去学习一些著名的算法保护以及研究它们的弱点。几年前，`Recca` 就首先发现了 `asprotect` 的弱点。我们知道他是利用了随机数发生器成功破解了 `asprotect` 的注册，所以我们也走这条路。。。

`ASProtect` - 是一种软件保护软件，能帮助软件设计者很快捷地设立软件辅助保护机制。`ASProtect` 还能提供诸如注册码或不同版本的设置。

## 2 Asprotect 1.0/1.1 [built-in key]

软件的注册部分会随机产生几个数，存放在 `.aspr` 文件里主要是由 `base 64` 编码的 `A`, `D`, `E` 和 `N` 四个数组成。（从右往左）

`A` = `INDtrZliM4t...0czFJpN42UQ==`

`D` = `U2atlST1lQ...kHcbIGwJU8=`

`E` = `EQAAAAAA...AAAAAAAA=`

`N` = `X7lD2zsvq3QW...ha6mOrdvULEAM8=`

然后注册码通过 `RSA` 函数计算产生：

1. `H1` = `RipeMD-160(A)`
2. `H2` = `MD5(Registration Information—H1)`
3. `Key` = `RSA(D,N, [H2—Registration Information—H1])`

注册码就会像这样:

`PYgt/87koSvbYPluc+/crrilfWI+ssZSU7UhgCLmK3D1C+x+`
`EX9n7ukwM5sKmI+nsH66V7L28BFTziNz5DOPLRHAqnI11wN5`
`Nd/dm0Esw20mm66V7L28BFTziNz55DOP4kzt+bie/rW4grgG`
`+e8/hsIuotMqUXguWKBnOXsoQ89Kg92T0MkB4FCZYuZQo=`

这样的注册码看上去是很安全的。。。

### 2.1 D, N 和 E 是怎样产生的呢？

全过程是通过一个 `RSAVR.DLL` 的 dll 文件进行运算的：

1. 选两个 512 位的大素数 `p` 和 `q`
   1.1 随机选取加密密钥 `e`，同样也是 512 位的
   1.2 使 `e` 和 `(p-1)(q-1)` 互素（非常耗时间）
2. 计算 `n=pq`
3. 计算 `d=exp(-1)mod ((p-1)(q-1))` (比如 `e = 17`)

如果我们选好了 `p` 和 `q` ，我们就能得到 `n`

### 2.2 怎样产生 p 和 q 呢?

```
unsigned long _rand()
{
    Seed *= 214013;
    Seed += 2531011;
    return( ((Seed >> 16) & 0x7FFF) );
}

unsigned long RandInt()
{
    for(i=0;i<4;i++) { rval = ((rval << 8) +_ rand()); }
    return(rval);
}

Seed = ( time() + ThreadId()) xor TickCount();
for(ri=0;ri<16;ri++) { BigNumber1[ri] = RandInt(); }
BigNumber1[ri] = BigNumber1[ri] xor C0000000h;
P= nextPrime(BigNumber1);

for(ri=0;ri<16;ri++) { BigNumber2[ri] = RandInt(); }
BigNumber2[ri] = BigNumber2[ri] xor C0000000h;
Q= nextPrime(BigNumber2);
```

如果我们能得到 `time() + ThreadId()) xor TickCount()`的计算值，我们就能得到 `p` 和 `q`...

### 2.3 破解

我不知道 `Recca` 是怎样找到合适的种子数的，但我认为好像穷举是唯一的解决办法

1. 选择种子数
2. 计算大数 1 和 大数 2
3. 使 `P = nextPrime(BigNumber1)` and `Q=nextPrime(BigNumber2)`
4. 如果 `n != p*q` 再尝试下一个数

但是完成全部的测试很花时间，所以基本不太可能测试所有的 2 的 32 次方个不同的数。

#### 2.31 减小测试范围

首先改进的是减小 `ThreadId()` 和 `TickCount()`，使得长度最大的种子数符合 `time()`。 最终我们就通过测试几个和公钥位数接近的种子数，来缩短所需要的时间。

#### 2.3.2 改进算法

猜试的速度还是很缓慢，所以我们不得不尝试找到一种算法，既能测试种子数又能不计算 `p` 和 `q` 的值。我们想到的办法就是修改 `nextPrime()`函数使他只检测大数 1 和大数 2 的高位部分，如果相等就认为是符合条件的数值

令

`P = BigNumber1 + M1` ( `M1` <2 的 64 次方)

`Q = BigNumber2 + M2` ( `M2` <2 的 64 次方)

则

`N = P* Q = (BigNumber1 + M1) * (BigNumber2 + M2)`

`N = BigNumber1 * BigNumber2 + M3` (`M3` <= 2 的(512+64+2)次方)

所以 `N(高位) = BigNumber1(高位) * BigNumber2(高位)`

算法：

1. 选一个种子数
2. 计算得到 `BigNumber1` 位和 `BigNumber2` 高位
3. 如果 `n` 的高位不等于 `BigNumber1` 的高位\*`BigNumber2` 的高位，就尝试下一个种子数
4. 且 `BigNumber1` 和 `BigNumber2` 不能相等。
5. 如果符合，再计算
   `P = NextPrime(BigNumber1)`
   `Q = NextPrime(BigNumber2)`
6. 如果 `N! = P* Q` 则尝试下一个数

### 2.4 例子

`Asprotect 1.1` 主程序加的是 `Asprotect 1.1` 壳。一时找不到一个更好的例子来检验我们的算法是不是有效...

从 `.aspr` 文件里得到 `n` 和 `e` 的值

`n = EB1D4EADA4815F6277519791BFFA8B4C0B872D1C436515AB`
`D9572B22BF6A03FECB4E5CC49AF1EE35C31344617A1210663056`
`90529B9CE7F13ED2D37CD7034A3EDD096853EC61243BCCAC5A58`
`800B0330A4DD85E9AA237F2F2AE60CA049B1D2777B2E0C5FF51E`
`058382A86C3EC12F7AB41642022772FF2A2D3DBA704725702199`

`e = 17`

`Asprotect` 是在 2000 年十月发布的，所以我们选择 `39000000h` 作为最小的种子数。我们在一分钟里就找到了 `398BBB72h` (是碰撞值，所以舍去)和 `399BACC4h`（一小时能检测所有的 2 的 32 次方个种子数）,所以我们很轻松地就得到了ｄ的值：

`D= l8F1EGKSQWCw9Et5klCpkm9/TIQFw0xOxibd+bQNndzGYoIX`
`4PmHXcdZtN3VWRQfuYS/cLeEf0i+kG3Cd7kaqKCkBO3xiAFgZMf`
`vW8D+bov+AfjDICITq5/Lhex7PykLGtUNnH8LSsmIDSWqldwX3Q`
`9o8U4HcJSjSJIfS4bumc=`

## 3 Asprotect 1.11c

不久 `A.S.` 就为了修补这个小问题而发布了一个新的版本。这次看来是没有人能够破解它了．．．：）

随机数是这样产生的：

```
unsigned long _rand()
{
    Seed = ( time() + ThreadId()) xor TickCount();
    Seed *= 214013;
    Seed += 2531011;
    return( ((Seed >> 16) & 0x7FFF) );
}

unsigned long RandInt()
{
    for(i=0;i<4;i++)
    { rval = ((rval << 8) +_ rand()); }
    4
    return(rval);
}

for(ri=0;ri<16;ri++) { BigNumber1[ri] = RandInt(); }
BigNumber1[ri] = BigNumber1[ri] xor C0000000h;
P= nextPrime(BigNumber1);

for(ri=0;ri<16;ri++) { BigNumber2[ri] = RandInt(); }
BigNumber2[ri] = BigNumber2[ri] xor C0000000h;
Q= nextPrime(BigNumber2);
```

这个新的种子数的每一位都被分别计算得到的。我们就必须重新写一个爆破机。但我遇到的还是相同的问题怎样才能快捷地找到合适的数而不测试所有的数。此外怎样才能方便检测 4\*2 的 512 次方个可能的大数呢？

### 3.1 减小测试范围

`GetCurrentThreadId()`是固定的值，`GetTickCount` 和 `time()`是利用瞬时时间得到的，但由于计算的速度很快，就导致 `GetTickCount` and `time()`在计算大数的时候基本是保持不变得，因此最终用这些函数计算得到的 `rand()`参数值总是一个& `0x7FFF` 的相同的值。所以我们能用 2 的 15 次方\*2 的 15 次方对的种子数来代替原来的 4\*2 的 512 次方\*2 的 512 次方个数。

只需要按照以下的格式就可以了：）

`BigNumber1 = [AorC0000000h]AAAA. . .AAAA`

`BigNumber2 = [BorC0000000h]BBBB . . .BBBB`

### 3.2 改进

每个大数都有一种已知的结构，我们就利用它去测试一个种子数是不是正确。如果 `n` 的高位除以大数 1 的高位所得到的数有类似 `(B or C0000000h)`的结构，那我们就猜这个数也因该就是 `p` 了，那是因为 `Q` 就是等于 `N/P`。现在我们就只剩下 2 的 15 次方个数来尝试了...

`P = [AorC0000000h]AAAA. . .AAA + M1`

`Q = [BorC0000000h]BBBB . . .BBBB + M2`

`N = BigNumber1 * BigNumber2 + M3`

所以 `N` 高位/`BigNumber1` 高位= `[BorC0000000h]BBBB`：

1. 选一个随机数
2. 计算大数 1 的高位
3. 如果 `n` 的高位除以大数 1 的高位不是有类似 `(B or C0000000h)`的结构，就继续尝试接下来的数
4. 如果符合那个结构，我们就把大数 1 给算出来
5. 通过这样算：`P = NextPrime(BigNumber1)`
6. 计算 `Q = N/P`

### 3.3 举例

`Asprotect 1.11c` 主程序使用 `Asprotect 1.11c` 保护的:

`N = C7D6E57A0D1C3A9752618FB497A6E4D1DCEC39EF22318F0C`
`6776E429ACBC3946F2018E643746E3817C8C389EC1D18DBC0716`
`E2D94C5C37F691A18D13D6E6F122D03D00090AF7AAEBC5B255CE`
`806D00B13B27AB93F5E25676B09D01596B57AC3C2612571EE0CD`
`02019B87ACE4564257C710FD02A9CBB7AD8C8672586F416D536D`

使用和前一次一样的算法我们就在 1 秒内就得到了那个合适的随机数值 `5454542D`（而且全部测试 2 的 15 次方个数也只用了 3 秒）

所以就得到了：

`P = D454542D5454542D...`

`Q = F0F0F088F0F0F088...`

轻松搞定 `d` 的值：

`D = HV/nbSNxR14Tvhm4bHRVey+U+qdbHQk8Q+BPfBrY`
`qZYMa14KmBhtGX4flkK+gVoGGX23485UMFwdxMMux5Aw`
`DtEsU+ZTzlQmvNX5zEuDRVg/1jZJGc7NIBltCVy+sOt+`
`iVqzBnopoHPQHrNGzDkr/615Ch40ns4iIWp3i7PbRs`

## 4 结论

是不是感到很有意思呢？这是个过时的软件了，但它却是一个很好的 `RSA1024` 被攻破的例子。

如果你想讨论关于保护，或是任何其他有意思代码的话题，随时可以联系我。下次问题我们再见...

再次感谢 `{TKM!, FFF,CoRE,GoD,UCF,DAMN, TMG, . . .}.` 

全文毕。

`4dd466d0e1e0cc123afbe504da2462ca`
