
# CRC-32 的算法小议

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-306.htm`](https://www.pediy.com/kssd/pediy08/pediy8-306.htm)

现在网络上广为流传的 `CRC32` 的实现代码，是基于 `little endian` 的。

幸运地是，`X86CPU` 和 `Ethernet` 都是基于 `little endian` 的。

`little endian` 从小头开始吃鸡蛋，将 `the most significant` 放在相对低位。

一块以 `little endian` 存放和传输方式的数据块（我当然说的是 `x86` 环境），

如何适应 `CRC32` 模型？

解决方法：

数据块的每一个字节内，相对低位是多项式码中的相对高位；

数据块的字节间，相对低位的字节是多项式码中的相对高位。

举个例子：

```c
/*  C  */

char    String[]={a,b,c,d,e,f};

```

```asm
;  MASM

String   db   'a','b','c','d','e','f'

```

```text
; 在 x86 环境中的内存分布

0x61    |   0x62   |  0x63    |  0x64   | 0x65    | 0x66
0110 0001 0110 0010  0110 0011 0110 0100 0110 0101 0110 0110

```

```text
; 表示成多项式，！！！
1000 0110 0100 0110 1100 0110 0010 0110 1010 0110 0110 0110......
; 为什么我加省略号，后面还可能接有更多的字节。
```

为什么选择这种方法，（1）后面可以有更多的字节，而只需要简单地往这个队列添加；（2）方便实现，还有，跟贴吧！！！

## CRC 的本质

`CRC` 的本质是多项式的模 2 运算，如果你关心它运算的过程，你就知道它的各种可能的变体。

### 模 2 运算

模 2 运算的加法和减法的结果都是相同，可以用逻辑异或来表示；

模 2 加减运算的特点：每一位的结果和上位或下位无关。

模 2 除法的余数总比除数少 1 位。

模 2 运算没有进位和退位的概念。

模 2 除法的上商原则是：余数最高位为 1,上 1,余数最高位为 0,上 0.

这一结论可以推至比特位：

任何数与 `0x0000000` 异或，结果不变；

任何数与 `0xfffffff` 异或，结果为反，相当于 `not` 指令。

### CRC\_Table 表

*   `CRC16_Table`
*   `CRC_Table` 是一个元素数目为 256,类型为 `unsigned short` 的一维数列
*   `CRC32_Table`
*   `CRC_Table` 是一个元素数目为 256,类型为 `unsigned int` 的一维数列

256＝2⁸，多项式的最高字节一共有 8 位

```c
for(i=0;i<256;i++)
{
    register = i;        /* the top byte of the polynominal */
    for(j=0;j<8;j++)
    /* the ordinal of bits in a byte: 7, 6, 5, ... */
    {
        if ( register & 1 )    /* dividable */
            register = ( register >> 1 ) ^ 0xEDB88320;
        else
            register >>= 1;
    }
    crc_table[i]=register;
}

char CRC_TABLE={y0,y1,y2,y3,y4,...,y255};
```

可以看作一个一对一的映射 `f(index)=y`;

`index` 是被除数，`y` 是余数，映射关系是经过 8 次模 2 除法运算。

```c
index = ( crc ^ char [n] ) & 0x000000ff;

crc = crc_table[index] ^ crc>>8;
```

一次运算，指 8 次有 8 比特位参加的模 2 除法运算。

`crc ^ char [n]`

本次运算的被除数，是上次运算的余数和 `char[n]`的逻辑加

`crc_table[index] ^ crc >> 8`

最后的结果等于本次运算的结果异或上次运算的余数，

（余数需经过移位）

引用: *最初由 `tnttools` 发布*

现在网络上广为流传的 `CRC32` 的实现代码，是基于 `little endian` 的，

幸运地是，`X86CPU` 和 `Ethernet` 都是基于 `little endian` 的。

`little endian` 从小头开始吃鸡蛋，将最不重要地放在最前面。

一块以 `little endian` 存放和传输方式的数据块（我当然说的是 `x86` 环境），

........

这不叫“最不重要”![](img/70b1b1b49da6fb968b36803b1ec24864.jpg)

`the least significant bit` 这叫最低位

`the most significant bit` 这叫最高位

也没什么最前面之说，这叫“最左边”(`little endian`)或者“最右边”(`big endian`)

`the left-most bit position`

`the right-most bit position`

这篇文章是你原创的么？

回应 `cnbragon` － 请大家先弄清楚这些：

字节内的顺序，字节之间的顺序，字 `word` 之间的顺序

标题中的字 `word`，

指在 `CPU` 环境下表示 `CPU` 处理和存储的单元。

（１）不是！！！比特位！！！

字节内的顺序，如 `cnbragon` 所说：

`the least significant bit`

`the most significant bit`

`the left-most bit position`

`the right-most bit position`

请 `cnbragon` 注意，`little endian` 所描述的数据存储方式的单位不是比特位，而至少是字节或是字节的倍数，不要和 `CPU` 的移位指令弄混。

（２）

我所说的最重要，最不重要指的是英文原文

`the most significant  and the least significant`

后面特意没有加 `byte`, `word` 等名词，是因为在不同的语境下，

这些都有可能。

```text
BYTE

UNICODE
```

按 `Little Endian` 存储如果是 `0xAA`, `0xBB`,

那么 `Big Endian` 存储就是 `0xBB`, `0xAA`

```text
WORD
```

这里的 `WORD`，指１６比特位

按 `little endian` 存储如果是 `0xAA`, `0xBB`, `0xCC`, `0xDD`

那么按 `Big Endian` 存储就是 `0xCC`, `0xDD`, `0xAA`, `0xBB`

（３）

关于最前面和最后面

一个队列，你说最前面和最后面，没有什么不妥，

说最左边和最右边也没有什么错。

这篇文章被看雪老大列为精华，我也觉得意外。

因为这只是我学习算法过程中的一个小小的思考过程，文章太短了，

但我觉得如果你真正想看懂网上 `CRC32` 代码的实现，这篇文章绝对有帮助。

当时在看 `CRC32` 代码实现时，不想作个 `copypaster`，反复思考：

Ⅰ.为什么要用表来实现

解决方法：

你既可以静态填表，也可以动态生成那个表。

Ⅱ.为什么要这样写代码，以字节为单位，为什么要移８位

所有思考的结果就汇成了这样一篇小文章，`cnbragon` 前辈说它不是原创，也对：我生来就什么都不知道，都是后天学习的结果，全部都是 `COPY|paste` 到脑袋中的。

这篇文章被看雪老大列为精华，我再次表示意外。

![](img/70b1b1b49da6fb968b36803b1ec24864.jpg)

我不是前辈。这篇文章加为精华实属正常，没有必要意外，我也替你感到高兴

(1)、（2）说话不要这么肯定，建议你看看标准文档后再来反驳：

`little-endian`

`Describes a computer architecture in which, within a given 16- or 32-bit word, bytes at lower addresses have lower significance (the word is stored ‘little-end-first’). The PDP-11 and VAX families of computers and Intel microprocessors and a lot of communications and networking hardware are little-endian. See big-endian, middle-endian, NUXI problem. The term is sometimes used to describe the ordering of units other than bytes; most often, bits within a byte.`

请你注意最后一句话。再说，没说你 `little-endian` 理解有错误，只是指出你专业英文翻译中的不当之处：`the most significant  and the least significant` 翻译成最重要和最不重要有点贻笑大方了。

（3）关于最前面和最后面，这种说法是不准确的，只能说你考虑的不是很全面

假如在内存中有如下的数据存储形式：

`0x00405000 ab cd ef 01 23 45`

人家可以理解成从 `ab` 到 `45` 的方向是由后到前或者理解成从 `45` 到 `ab` 的方向是由后到前，必须指出是左还是右才不易引起混淆

在 `NIST` 的 `fips 180-2` 中，有这样一段话：

`Throughout this specification, the “big-endian” convention is used when expressing both 32- and 64-bit words, so that within each word, the most significant bit is stored in the left-most bit position.`

这些知识都是已经有标准的定论的基本概念，根本不需要做过多的讨论。当然如果你愿意按照自己的理解也无可厚非，只是希望你能多读读一些标准的东西，再对照自己的想法，以达到理解上没有偏差，做学问应当如此。

已看，你正确，我失误。多谢！！！

我发表文章的原因之一：就是希望有人指出我的错误。

孟子问“独乐乐,与人乐乐,孰乐"

答：与人乐乐，乐。
