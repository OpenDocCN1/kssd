
```c
//block存储需要初始置换的明文数据 类型 unsigned int[2];
register unsigned long work, right, leftt;
register int round;
leftt = block[0];
right = block[1];
work = ((leftt >> 4) ^ right) & 0x0f0f0f0fL;
right ^= work;
leftt ^= (work << 4);
work = ((leftt >> 16) ^ right) & 0x0000ffffL;
right ^= work;
leftt ^= (work << 16);
work = ((right >> 2) ^ leftt) & 0x33333333L;
leftt ^= work;
right ^= (work << 2);
work = ((right >> 8) ^ leftt) & 0x00ff00ffL;
leftt ^= work;
right ^= (work << 8);
right = ((right << 1) | ((right >> 31) & 1L)) & 0xffffffffL;
work = (leftt ^ right) & 0xaaaaaaaaL;
leftt ^= work;
right ^= work;
leftt = ((leftt << 1) | ((leftt >> 31) & 1L)) & 0xffffffffL;
```

---

## 代码块图解：

**对应代码块：**

```c
leftt = block[0];
right = block[1];
```

![图解1](upload/attach/201908/813468_5EPBVA4XTMTJ3SZ.png)

---

**对应代码块：**

```c
work = ((leftt >> 4) ^ right) & 0x0f0f0f0fL;
right ^= work;
leftt ^= (work << 4);
```

**对应图解：**

![图解2](upload/attach/201908/813468_3PMBJ3ZR8FGCMCB.png)

---

**对应代码块：**

```c
work = ((leftt >> 16) ^ right) & 0x0000ffffL;
right ^= work;
leftt ^= (work << 16);
```

**对应图解：**

![图解3](upload/attach/201908/813468_HKT385MRXUSE2HH.png)

---

**对应代码块：**

```c
work = ((right >> 2) ^ leftt) & 0x33333333L;
leftt ^= work;
right ^= (work << 2);
```

**对应图解：**

![图解4](upload/attach/201908/813468_V62STPJ49FM9ZBV.png)

---

**对应代码块：**

```c
work = ((right >> 8) ^ leftt) & 0x00ff00ffL;
leftt ^= work;
right ^= (work << 8);
right = ((right << 1) | ((right >> 31) & 1L)) & 0xffffffffL;
```

**对应图解：**

![图解5](upload/attach/201908/813468_SJB83M2F2EZJ3NJ.png)

---

**对应代码块：**

```c
work = (leftt ^ right) & 0xaaaaaaaaL;
leftt ^= work;
right ^= work;
leftt = ((leftt << 1) | ((leftt >> 31) & 1L)) & 0xffffffffL;
```

**对应图解：**

![图解6](upload/attach/201908/813468_7AE7DWEJDT4QRTN.png)

**重点：**在最后一个代码上，大家可能发现`leftt`这个数据块已经和初始置换表上的前32位一致了，然而最后一句代码竟然把`leftt`完美的移位给破坏了，而`right`数据块还是与初始置换表的后32位有点差距的。所以我们就找到了这个代码存在了数十年的问题，`leftt = ((leftt << 1) | ((leftt >> 31) & 1L)) & 0xffffffffL;`这句话应该用于处理`right`数据块，而不是处理`leftt`。那要怎么改呢？我们从图解中发现，如果将`right`的最后一位移动到第一位上，那么存在置换表的替换就完成了。所以改动后的最后一句代码如下：

```c
//leftt = ((leftt << 1) | ((leftt >> 31) & 1L)) & 0xffffffffL;
right = ((right >> 1) | ((right << 31))) & 0xffffffffL;
```

**在进行初始置换前，作者还将输入的明文数据进行了由小端序转换成大端序的处理，所以如果要彻底替换初始置换，还需要进行大小端序转换，然后进行初始置换的移位操作，最后不要忘了将大端序转换成小端序。**

附上修改后的加密函数：

```c
static void IP(register unsigned long *block)
//register unsigned long *block, *keys;
{
register unsigned long right, leftt, work;
leftt = block[0];
right = block[1];
work = ((leftt >> 4) ^ right) & 0x0f0f0f0fL;
right ^= work;
leftt ^= (work << 4);
work = ((leftt >> 16) ^ right) & 0x0000ffffL;
right ^= work;
leftt ^= (work << 16);
work = ((right >> 2) ^ leftt) & 0x33333333L;
leftt ^= work;
right ^= (work << 2);
work = ((right >> 8) ^ leftt) & 0x00ff00ffL;
leftt ^= work;
right ^= (work << 8);
right = ((right << 1) | ((right >> 31) & 1L)) & 0xffffffffL;
work = (leftt ^ right) & 0xaaaaaaaaL;
leftt ^= work;
right ^= work;
//leftt = ((leftt << 1) | ((leftt >> 31) & 1L)) & 0xffffffffL;
right = ((right >> 1) | ((right << 31))) & 0xffffffffL;
block[0] = leftt;
block[1] = right;
}
//将小端序转换成大端序
static void scrunch(register unsigned char *outof, register unsigned long *into)
//register unsigned char *outof;
//register unsigned long *into;
{
*into = (*outof++ & 0xffL) << 24;
*into |= (*outof++ & 0xffL) << 16;
*into |= (*outof++ & 0xffL) << 8;
*into++ |= (*outof++ & 0xffL);
*into = (*outof++ & 0xffL) << 24;
*into |= (*outof++ & 0xffL) << 16;
*into |= (*outof++ & 0xffL) << 8;
*into |= (*outof & 0xffL);
return;
}
//将大端序转换成小端序
static void unscrun(register unsigned long *outof, register unsigned char *into)
//register unsigned long *outof;
//register unsigned char *into;
{
*into++ = (*outof >> 24) & 0xffL;
*into++ = (*outof >> 16) & 0xffL;
*into++ = (*outof >> 8) & 0xffL;
*into++ = *outof++ & 0xffL;
*into++ = (*outof >> 24) & 0xffL;
*into++ = (*outof >> 16) & 0xffL;
*into++ = (*outof >> 8) & 0xffL;
*into = *outof & 0xffL;
return;
}
//对明文组进行初始IP置换
unsigned long work[2];
scrunch(PlainText, work);
IP(work);
unscrun(work, PlainText);
```

目录

- [基础部分概述：](https://bbs.kanxue.com/#基础部分概述：)
- [第一节：DES算法简介](https://bbs.kanxue.com/#第一节：des算法简介)
- [第二节：DES算法原理](https://bbs.kanxue.com/#第二节：des算法原理)
- [DES算法的Feistel框架：](https://bbs.kanxue.com/#des算法的feistel框架：)
- [DES算法的加密流程：](https://bbs.kanxue.com/#des算法的加密流程：)
- [DES算法中的初始置换](https://bbs.kanxue.com/#des算法中的初始置换)
- [DES算法中的LR分组：](https://bbs.kanxue.com/#des算法中的lr分组：)
- [DES算法中的F函数：](https://bbs.kanxue.com/#des算法中的f函数：)
- [扩展置换E：](https://bbs.kanxue.com/#扩展置换e：)
- [S盒置换：](https://bbs.kanxue.com/#s盒置换：)
- [P置换：](https://bbs.kanxue.com/#p置换：)
- [DES算法中的循环以及逆初始值置换：](https://bbs.kanxue.com/#des算法中的循环以及逆初始值置换：)
- [DES算法的密钥生成算法：](https://bbs.kanxue.com/#des算法的密钥生成算法：)
- [密钥生成算法PC-1置换](https://bbs.kanxue.com/#密钥生成算法pc-1置换)
- [密钥生成算法分组以及左移处理](https://bbs.kanxue.com/#密钥生成算法分组以及左移处理)
- [密钥生成算法PC-2置换](https://bbs.kanxue.com/#密钥生成算法pc-2置换)
- [DES算法的解密方法：](https://bbs.kanxue.com/#des算法的解密方法：)
- [基础部分总结：](https://bbs.kanxue.com/#基础部分总结：)
- [进阶部分概述：](https://bbs.kanxue.com/#进阶部分概述：)
- [优化一：将S盒和P置换表进行合并](https://bbs.kanxue.com/#优化一：将s盒和p置换表进行合并)
- [第一步：优化S盒](https://bbs.kanxue.com/#第一步：优化s盒)
- [第二步：合并S盒与P置换表](https://bbs.kanxue.com/#第二步：合并s盒与p置换表)
- [优化二：通过位运算替代初始置换](https://bbs.kanxue.com/#优化二：通过位运算替代初始置换)
- [D3DES中相关代码：](https://bbs.kanxue.com/#d3des中相关代码：)
- [代码块图解：](https://bbs.kanxue.com/#代码块图解：)

# 基础部分概述：

> - **本节目的：**这一章作为DES算法的基础部分，目的主要是整理下密码学中DES加密与解密的相关知识点，并把它们整理出来，同时还有一些不懂之处希望得到解答，后续会跟上相应的答案。
> - **阅读方法：**希望大家在浏览完本章文章后可以自己去实现一下，相信一定会对你的编程技术有所提高。(附件中提供参考代码)
> - **具备基础：**
>   (1)熟练掌握C语言
> - **学习环境：**任意C语言开发环境，一个正确的DES算法程序(方便调试，验证程序结果)

---

# 第一节：DES算法简介

**DES的英文全称是Data Encryption Standard，意思是数据加密标准。而我们本篇文章讨论的是DES的加密算法。希望大家能够将这两个名词区别开来，很多时候我们说的DES都是在指DES算法，而不是DES数据加密标准。DES算法是一种典型的分组密码，即将固定长度的明文通过一系列复杂的操作变成同样长度密文的算法。也就是运行一次DES算法只能对64位长度的数据进行加密操作，这样做的缺点就是如果处理大量数据就会花费相当长的时间，所以DES算法也就不适合对大数据进行处理。DES在现代密码学中属于对称加密算法，即该算法能够使用相同的密钥进行加密和解密。**

**DES算法在设计中使用了分组密码设计的两个原则：混淆和扩散，混淆是使密文的统计特性与密钥的取值之间的关系尽可能复杂化，以使密钥和明文以及密文之间的依赖性对密码分析者来说是无法利用的。扩散的作用就是将每一位明文的影响尽可能迅速地作用到较多的输出密文位中，以便在大量的密文中消除明文的统计结构，并且使每一位密钥的影响尽可能迅速地扩展到较多的密文位中，以防对密钥进行逐段破译。主要目的就是增加数据被破解的难度，造成雪崩效应，即在输入时即使发生了微小的变化都会对结果产生不可分辨性的变化。**

# 第二节：DES算法原理

**DES算法使用的是Feistel框架，Feistel框架的密码结构是用于分组密码中的一种对称结构，Feistel框架使用的是对同一处理流程进行多次循环的方式。DES算法就是采用的16轮循环的加密，当然循环次数越多加密的效果越好，同时时间成本也会上去，设计者需要考虑这两方面的因素进行密码设计。因为这个框架是用于处理分组密码的，我们就要了解一下什么是分组密码，分组密码是一种加解密方案，将输入的明文分组当作一个整体处理，并输出一个等长的密文分组，典型的分组大小为64位和128位，随着对加密算法安全性能的要求不断提高，分组大小也就会越来越大。DES算法所采用的分组大小为64位分组，所以输入的数据和密钥都是按照64位进行处理的，输出的密文也是64位一组。**

## DES算法的Feistel框架：

![DES的Feistel框架](https://bbs.kanxue.com/upload/attach/202002/813468_24VWUXQHSW3DQWC.png)

## DES算法的加密流程：

1. 将输入的明文进行初始置换。
2. 将初始置换后的结果按照每组32位数据分成L、R两组。
3. 将48位子密钥`k[i]`和32位`R[i]`组数据输入`F`函数进行处理，输出32位处理结果。
4. 将32位`F`函数处理结果与32位的`L[i]`组数据按位进行异或操作。
5. 将32位的异或处理结果赋值给`R[i+1]`分组，将32位的`R[i]`分组数据赋值给`L[i+1]`。

   `L[i+1] = R[i]`，

   `R[i+1] = L[i]^(F(k[i], R[i]))`

6. 对3、4、5步处理进行16次循环，得到`L[16]`，`R[16]`。
7. 将`L[16]`、`R[16]`数据进行交叉合并得到64位数据。
8. 对得到的64位数据进行逆初始值置换，得到想要的密文`Y`。

**（从这里大家也可以看出来，在进行最后一轮循环时`R[15]`被赋值给`L[16]`，`L[15]`经过一系列运算后结果赋值给了`R[16]`，这里进行了一次交换，而在逆初始值置换之前又进行了一次交换，相当于没有变化。这里其实是为了保持16轮循环处理的过程一致，方便学习。如果大家觉得需要优化下，完全可以将`R[15]`直接赋值给`R[16]`,`L[15]`经过一系列运算后结果赋值给`L[16]`，这是完全没有问题的。）**

## DES算法中的初始置换

**置换处理：**在密码学中置换是指在保持数据不变的情况下，打乱数据的位置顺序的操作称作置换，在DES算法中每个置换处理都会按照相应的置换表进行操作，置换处理在DES算法中得到了充分的运用，通过置换处理可以打乱输入数据的顺序，使输入的数据变得面目全非，当然也会造成雪崩效应，因为只要有一个数据位发生变化，就会影响到很多地方。

**初始置换表：**

![初始置换表](https://bbs.kanxue.com/upload/attach/202002/813468_BVGQF87E35ZQSSH.png)

**这个初始置换表怎么使用呢？如何通过该表进行DES初始置换呢？**

**首先我们就要把明文的64位数据按照从左到右，从上到下进行排列，形成一个8*8的矩阵。我们以初始置换表的第0行第0列为例进行讲解，在初始置换表中该位置的数值是58，我们就要将数据中的第58个Bit位移动到第一个Bit位上，这就完成了一次置换。因为这个表的大小有64位，所以就要进行64次置换才会得出结果。**

**图示：**

![置换图解](https://bbs.kanxue.com/upload/attach/202002/813468_KAGQQN6G7B6FNG8.png)

**代码演示：**
