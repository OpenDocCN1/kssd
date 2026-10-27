
```C
int howManyBits(int x) {
int sign=x>>31;
x = (sign&~x)|(~sign&x);
int b16=(!!((x>>16)&0xffff))<<4;
x=x>>b16;
x=x&0xffff;
int b8=(!!((x>>8)&0xff))<<3;
x=x>>b8;
int b4=(!!((x>>4)&0xf))<<2;
x=x>>b4;
int b2=(!!((x>>2)&0x3))<<1;
x=x>>b2;
int b1=(!!((x>>1)&0x1))<<0;
x=x>>b1;
return b16+b8+b4+b2+b1+X+1;
}
```

整数阶段到此完结了就。

### floatScale2

```C
/*
* floatScale2 - Return bit-level equivalent of expression 2*f for
* floating point argument f.
* Both the argument and result are passed as unsigned int's, but
* they are to be interpreted as the bit-level representation of
* single-precision floating point values.
* When argument is NaN, return argument
* Legal ops: Any integer/unsigned operations incl. ||, &&. also if, while
* Max ops: 30
* Rating: 4
*/
unsigned floatScale2(unsigned uf) {
return 2;
}
```

就是返回一个浮点数乘 `2`。首先排除掉那些特殊值： `NAN，INF` 和 `0`，乘二返回指数 `+1` 即可，

#### 答案

```c
unsigned floatScale2(unsigned uf) {
int exp = (uf&0x7f800000)>>23;
int sign = uf&(1<<31);
if(exp==0) return uf<<1|sign;
if(exp==255) return uf;
exp++;
if(exp==255) return 0x7f800000|sign;
return (exp<<23)|(uf&0x807fffff);
}
```

### floatFloat2Int

```c
/*
* floatFloat2Int - Return bit-level equivalent of expression (int) f
* for floating point argument f.
* Argument is passed as unsigned int, but
* it is to be interpreted as the bit-level representation of a
* single-precision floating point value.
* Anything out of range (including NaN and infinity) should return
* 0x80000000u.
* Legal ops: Any integer/unsigned operations incl. ||, &&. also if, while
* Max ops: 30
* Rating: 4
*/
int floatFloat2Int(unsigned uf) {
return 2;
}
```

实现一个类似是强制转换的东西吧，首先还是按照浮点数的规则把各个字段提取出来，然后特判移位就行了。

#### 答案

```c
int floatFloat2Int(unsigned uf) {
int s=uf>>31;
int exp=((uf&0x7f800000)>>23)-127;
int frac=(uf&0x007fffff)|0x00800000;
if(!(uf&0x7fffffff)) return 0;
if(exp>31)return 0x80000000;
if(exp<0) return 0;
if(exp>23)frac<<=(exp-23);
else frac>>=(23-exp);
if(!((frac>>31)^s)) return frac;
else if(frac>>31) return 0x80000000;
else return ~frac+1;
}
```

### floatPower2

```c
/*
* floatPower2 - Return bit-level equivalent of the expression 2.0^x
* (2.0 raised to the power x) for any 32-bit integer x.
*
* The unsigned value that is returned should have the identical bit
* representation as the single-precision floating-point number 2.0^x.
* If the result is too small to be represented as a denorm, return
* 0. If too large, return +INF.
*
* Legal ops: Any integer/unsigned operations incl. ||, &&. Also if, while
* Max ops: 30
* Rating: 4
*/
unsigned floatPower2(int x) {
return 2;
}
```

实现运算 `pow(2,x)`，返回结果为 `int` 虽然我们可以直接 `1<<x`，但是它要求浮点计算。

首先防止溢出先判断，因为是 `2` 的整数次方，所以它的结果就是 `(exp+127)<<23`，然后判断无穷小和溢出即可，这个比较简单。

#### 答案

```c
unsigned floatPower2(int x) {
int INF = 0xff<<23;
int exp = x + 127;
if(exp <= 0) return 0;
if(exp >= 255) return INF;
return exp << 23;
}
```

需要注意，由于最后一个题目测试数据比较多，因此要修改时限才能通过。我也不知道我算法有什么问题，这 `O(1)` 的复杂度也不知道上哪优化去。

### result

![6](upload/attach/202206/919002_BWCKHKT4M8KU8EV.jpg)

## 小结

啥也不说了 `CSAPP` 永远的神，入门二进制必备书籍，二刷都能感觉学到了很多以前没学到的东西，接下来没以前认真看的也都得去看一遍了。

之前 `CS:APP` 没有好好学，今天下定决心来重新看一遍，并把实验好好做一遍以巩固自身。

<!--more-->

## 信息存储

### 数字表示

二进制：区别于我们平时计数使用的十进制，二进制使用的是逢二进一原则，而我们的十进制则是逢十进一，比如我们十进制的 `9+1` 中的 `9`，答案应该是十，但是十应该进一，因此得出了我们常规的答案 `10`。在二进制里面，每一位只要大于等于 `2` 则都要向高位进一。为了方便表示，还衍生出了二进制的子类，比如八进制，十六进制等，主要是二进制向这R

这些进制转换较为容易，而计算机平时又都处理二进制数据，因此就出现了这些常见的进制计数。

### 信息存储

大多数计算机使用的都是8位（在计算机中，除特殊说明外，一位均指的是二进制的位）的块，或者叫字节，字节是作为计算机可寻址的最小单位。一般来说我们并不习惯于将一个字节写成八位二进制的数，而是会写成两位十六进制的数。十六进制与二进制之间的转换也会十分容易，举个例子：

|  |  |

| --- | --- |

| 1  2 | `0x99``-``>``0b10011001`  `0x88``-``>``0b10001000` |

我们可以发现我们并不用像十进制那样权值相加或者是除二取余那么麻烦，我们把一位十六进制视为四位二进制即可，这样我们在转换的时候就是直接每一位分别转了，可以看出十分的方便。

#### 字数据大小

每台计算机都有一个字长，字长为计算机 `CPU` 一次能处理的最大数据，也有一种说法是能表示的最大内存。其实意思是差不多的，比如我们都知道 `32` 位的计算机最多能装 `4GB` 的内存，再多它也是只能使用这么多的内存，那是因为 `CPU` 要访问内存的时候，也只能使用一个 `32` 位的数据来表示地址，32位的数能表示的数的个数也就是 `2^32` 这么多，而地址指示的单位是字节，所以最多就是 `2^32` 字节，那就是熟知的 `4GB` 了。

`C` 语言中 `sizeof()` 会返回一个对象所占的字节数，我们对比输出下 `32` 位机子和 `64` 位机子的各个基本数据类型所占的字节数。我们不必找两个机子，只需要在 `64` 位的机子上分别编译 `32` 位和 `64` 位的程序即可。

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11 | `#include<stdio.h>`    `int` `main(){`  `printf(``"char:%d\n"``,sizeof(char));`  `printf(``"short:%d\n"``,sizeof(short));`  `printf(``"int:%d\n"``,sizeof(``int``));`  `printf(``"long:%d\n"``,sizeof(``long``));`  `printf(``"char *:%d\n"``,sizeof(char` `*``));`  `printf(``"float:%d\n"``,sizeof(``float``));`  `printf(``"double:%d\n"``,sizeof(double));`  `}` |

`32`位的结果：

![1](https://bbs.kanxue.com/upload/attach/202206/919002_HNRVN7NKW6K8K83.jpg)

`64` 位的结果：

![2](https://bbs.kanxue.com/upload/attach/202206/919002_8M2RA4W5C9NXEFG.jpg)

我们总结出如下表格

| C声明 | 32位字节数 | 64位字节数 |

| --- | --- | --- |

| char | 1 | 1 |

| short | 2 | 2 |

| int | 4 | 4 |

| long | 4 | 4 |

| char \* | 4 | 8 |

| float | 4 | 4 |

| double | 8 | 8 |

其实有些时候， `long` 的字节数也会随机器字长有所变化的，只是好像某个版本区分了 `64` 位整数就叫 `long long` 而 `long` 地位与 `int` 一致了。但是除了这个问题，可以发现只有指针类型的数据会随着机器字长发生变化，这也如我们所说，机器字长决定了指针就多少位，决定了有多少个地址，能用多少内存，多出的内存机器就无法做区分了。

### 寻址和字节顺序

对于多字节的数据类型我们必须确定两点：

1. 这个对象地址在哪里

2. 这个对象中的字节按什么顺序排列

比如一个 `int` 它有四个字节，那么我定义 `int a=0x12345678`。首先它一定是连续排列的，因此我们确定一个 `int` 的地址只需要确定它的最高字节的位置即可确定整个 `int` 的位置。假如 `int` 的地址是在 `0x100` 的，那么它应该怎么排列这些字节呢？

我们最初可能按照惯性，认为它是按照如下方式存储的：

|  |  |

| --- | --- |

| 1  2 | `字节：``12`  `34`  `56`  `78`  `地址：``100` `101` `102` `103` |

这个也很符合我们的书写规则，但是实时却恰恰相反，在现在大部分的机器中，它是反着存储的。也就是

|  |  |

| --- | --- |

| 1  2 | `字节：``78`  `56`  `34`  `12`  `地址：``100` `101` `102` `103` |

这两种存储方式我们分别叫大端序和小端序。

- 大端序：最高有效字节在低地址

- 小端序：最高有效字节在高地址

在我们书写汇编语言的时候，要写一个值通常也是以小端序的方式书写的。

比如我给 `eax` 寄存器加上 `0x12345678` 你会发现它的字节码是：

|  |  |

| --- | --- |

| 1  2 | `mov    $``0x12345678``,``%``eax`  `b8` `78` `56` `34` `12` |

我们通过定义以下的函数以十六进制来逐个显示内存中的字节。

|  |  |

| --- | --- |

| 1  2  3  4  5  6 | `void show_bytes(void` `*``start,size_t` `len``){`  `size_t i;`  `for``(i``=``0``;i<``len``;i``+``+``)`  `printf(``"%.2x "``,``*``(unsigned char` `*``)(start``+``i));`  `printf(``"\n"``);`  `}` |

其中 `start` 参数为该对象的起始地址，`len` 参数为显示的字节数，可以任意定义变量调用这个函数来查看这个对象在内存中的排布，理解浮点数的时候这个函数非常有用。

来查看下面的一个例子：

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12 | `void show_bytes(void` `*``start,size_t` `len``){`  `size_t i;`  `for``(i``=``0``;i<``len``;i``+``+``)`  `printf(``"%.2x "``,``*``(unsigned char` `*``)(start``+``i));`  `printf(``"\n"``);`  `}`  `int` `main(){`  `int` `x``=``0x3039``;`  `show_bytes(&x,sizeof(x));`  `float` `y``=``x;`  `show_bytes(&y,sizeof(y));`  `}` |

可以发现浮点数的输出在整数形态下为 `0x4640E400`。与整数的存储有着截然不同的结果，但是我们对这个结果的二进制适当移位就会发现它们有13个相匹配的位。

|  |  |

| --- | --- |

| 1  2  3  4 | `0`   `0`   `0`   `0`   `3`   `0`   `3`   `9`  `0000000000000000001000000111001`  `4`   `6`   `4`   `0`   `E` `4`   `0`   `0`  `01000110010000001110010000000000` |

我们也可以看到13个位刚好就是 `int` 形式下的低 `13` 位，这并不是巧合，大家可以试试看其它数，并且可以看看它最多能匹配多少位的整数，但是要注意结论需要有一般性，特殊的数字符合并不能得出什么结论，只能说是特性。

### 表示字符串

字符串的定义就是一个以 `null` 字符结尾的字符数组，如果将字符串看成一个数据类型的话，那么它是以大端形式存储的，但是其实实际上应该说是数组是大端存储的。我们在书写的时候，一般下标 `0`是最小的，但是我们习惯把它称为高位，高位在低地址便是大端序。比如 `char s[]="1234";` 那么 `s[0]='1',s[1]='2',s[2]='3',s[3]=4,s[4]=NULL`。请注意末尾的空字节也会包括在字符串里面，但是我们算长度不会算上这个字节。在 `gcc` 编译器中，我们编译如下的程序：

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7 | `#include<stdio.h>`  `char s[]``=``"0123456789ABCDEF"``;`  `int` `main(){`  `char buf[``16``];`  `strcpy(buf,s);`  `printf(``"%s"``,s);`  `}` |

编译器会发出可能存在溢出的警告，因为在拷贝的时候会多携带一个 `0` 字节过来，这在后面堆利用中也是很常见的 `off by null` 的手段。

再比如我们用这样的方式：

|  |  |

| --- | --- |

| 1  2  3  4  5 | `#include<stdio.h>`  `char s[]``=``"0123456789ABCDEF"``;`  `int` `main(){`  `printf(``"%d"``,sizeof(s));`  `}` |

运行我们也可以发现这个 `char` 数组占用了 `17` 个字节的空间。

### 布尔代数简介

布尔代数是一个建立在二元集合集合 `G={0,1}` 上的定义。

#### 非

> 非0即为1，非非0即为0。

#### 与

> 全1为1，不全1为0

#### 或

> 全0为0，不全0为1

#### 异或

> 相同为0，不同为1

---

#### 结论

1. 非0为1，非1为0

2. 1与1为1，1与0为0，0与1为0，0与0为0

3. 1或1为1，1或0为1，0或1为1，0或0为0

4. 1异或1为0，1异或0为1，0异或1为1，0异或0为0

在C语言中也有一个类型是 `bool`，它们有特殊的运算符：

- 与：&&

- 或：||

- 非：!
