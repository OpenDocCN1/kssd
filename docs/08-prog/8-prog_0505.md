
# 小议 Fletcher's checksum

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-719.htm`](https://www.pediy.com/kssd/pediy08/pediy8-719.htm)

将一些标准或实现语焉不详的地方或难于理解的地方说清楚，不希望有任何死角。

欢迎大牛指正。

By: TnTTools

## 参数集合

利用参数集合来描述具体的算法，是我从 `Ross` 处得到的灵感。

`http://www.ross.net/crc/`

| 参数 | 说明 |
| :--- | :--- |
| **Name** | 算法的名字，一般是约定俗成的，可不是我随意起的。 |
| **Block** | 指处理单元（块）的长度。<br>`8-bit` 可用于处理 `ANSI` 字符，`16-bit` 可用于处理 `Unicode` 字符。<br>标准中 `8-bit Fletcher`, `16-bit Fletcher` 就是指此参数。 |
| **Chksum** | 指校验和的长度。<br>`A` 和 `B` 的长度一般相同：`LengthA=LengthB=Chksum/2`。<br>`Adler-8`, `Adler-16`, `Adler-32` 其中的数字 `8`, `16`, `32` 就是指此参数。 |
| **InitA, InitB** | `A` 和 `B` 的初始化值 |
| **Modulo(Base)** | 模，`Base` 见于 `adler32.c` in `zlib`<br>一般地 `Fletcher's checksum` 的模是 `65535`；<br>而 `Adler's checksum` 的模变为 `65521`。 |

示例配置：

```
Name: Adler-32
ChkSum: 32(LengthA,LengthB: 16)
InitB,InitA: 0x0000, 0x0001
Modulo: 65521
```

## 数学原理

`Fletcher's` 的数学原理非常简单：和，数列的和。

```
; original
A = Sum.
B = Sum of sum.

; More practical
A = InitA + D[1] + D[2] + D[3] + ... + D[n]
B = InitB + nInitA + nD[1] + (n-1)D[2] + (n-2)D[3] + ... + D[n]
```

## 实现

`RFC1146` 中的实现太过简略，没有实用价值。

实用的代码，可参见

`Fletcher's`
`http://en.wikipedia.org/wiki/Fletcher%27s_checksum`

`Adler's`

(1)
`adler32.c`

(2)
`http://en.wikipedia.org/wiki/Adler-32`

外循环遍历字符数列，内循环实现不溢出条件下的具体加法操作。

下面说明一下 `360` 和 `5552` 这两个数字的来历。

### 360

`360` 是...算法中 `A` 和 `B` 都不会溢出时内循环最大次数 `N` 的最大整数值。

`Block=16`，`Chksum=32`, `Modulo=65535`

**推导过程**

`B` 和 `A` 执行相同次数的加法操作，`B` 明显比 `A` 大
只要 `B` 不溢出，`A` 也不会溢出。

```
B = InitB + N*InitA + N*D[1]+(N-1)*D[2]+(N-2)*D[3]+...+D[N]
MaxD[x]=0x0000FFFF，
MaxInitA=Modulo-1=0x0000FFFF
MaxInitB=Modulo-1=0x0000FFFF

B(max) = 0x0000FFFF * N * (N+1)/2 + (N+1)* 0x0000FFFF
B(max) <= 0xFFFFFFFF

解不等方程式：

0xFFFFFFFF >= 0x0000FFFF * N * (N+1)/2 + (N+1)* 0x0000FFFF

<=>

N*N + 3N - (65537-1)*2 <= 0
N*N + 3N - 131072 <= 0

解得：

-363.54 <= N <= +360.54
```

### 5552

`5552` 是...算法中 `A` 和 `B` 都不会溢出时内循环最大次数 `N` 的最大整数值。

`Block=8`, `Chksum=32`, `Modulo=65535`

**推导过程**

```
B = InitB + N*InitA + N*D[1]+(N-1)*D[2]+(N-2)*D[3]+...+D[N]
MaxD[x]=0x000000FF
MaxInitA=Modulo-1=0x0000FFFF
MaxInitB=Modulo-1=0x0000FFFF

B(max) = 0x000000FF * N * (N+1)/2 + (N+1)* 0x0000FFFF
B(max) <= 0xFFFFFFFF

解不等方程式：

0xFFFFFFFF >= 0x000000FF * N * (N+1)/2 + (N+1)* 0x0000FFFF

<=>

N*N + (1+0x202)N <= ( 0x1010101 - 0x101 )*2
N*N + 515*N <= 33685504

解得：

-6067.13 <= N <= + 5552.13
```

## 组合

如果有两个 `Fletcher's checksum`，如何组合它们。相关的数学原理为

```
A1 = InitA + P[1] + P[2] + ... + P[m]
B1 = InitB + mInitA + mP[1] + (m-1)P[2] + ... + P[m]

A2 = InitA + Q[1] + Q[2] + ... + Q[n] 
B2 = InitB + nInitA + nQ[1] + (n-1)Q[2] + ... + Q[n] 

A = InitA + P[1] + ... + P[m] + Q[1] + ... + Q[n]        
  = A1 + A2 - InitA 

B = InitB + (m+n)InitA + (m+n)P[1] + ... + (1+n)P[m] + nQ[1] + ... + Q[n] 
  = B1 + B2 + n*(A1 - InitA) - InitB 
```

`adler32` 的实现代码可参见 `adler32_combine()` in `adler32.c` in `zlib 1.2.3`.

`4dd466d0e1e0cc123afbe504da2462ca`
