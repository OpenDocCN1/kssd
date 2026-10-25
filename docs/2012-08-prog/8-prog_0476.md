
# C 语言一题

> 原文：[`www.pediy.com/kssd/pediy11/121637.html`](https://www.pediy.com/kssd/pediy11/121637.html)

![](img/070d6550041e5346c13283420b7fb91e.jpg)

为了方便大家辨别，我人肉 OCR 一下。

2.71：

You just started working for a company that is implementing a set of procedures to operatate on a data structure where 4 signed bytes are packed into a 32-bit unsigned. Bytes within the word are numbered from 0 (least significant) to 3 (most significant). You have been assigned the task of implementing a function for a machine using two's-complement arithmetic and arithemetic right shifts with following prototype:

```c
/* Declaration of data type where 4 bytes are packed into an unsigned */
typedef unsigned packet_t;

/* Extract byte from word. Return as signed integer */
int xbyte(packet_t word, int bytenum);
```

That is, the function will extract the designed byte and sign extend it to be a 32-bit int.

Your predecessor (who was fired for incompetence) wrote following cod

```c
/* Failed attempt at xbyte */
int xbyte(packet_t word, int bytenum)
{
    return (word >> (bytenum << 3)) & 0xFF;
}
```

A. What is wroing with this code?
B. Give a correct implementation of the function that uses only left and righ shifts, along with one subtraction.

嗯，是 2 次，搞错了

```c
int xbyte(unsigned word, int bytenum)
{
    unsigned y = 0;
    y = (unsigned char)(word >> (bytenum << 3));
    unsigned char sign = (y >> 7) << 7;
    unsigned char val = (unsigned char)((unsigned char)y << 1) >> 1;
    return val - sign;
}
```

引用: *最初由 **foxabu**发布* ![查看帖子](img/showthread.php?p=867265#post867265) 显然不能够类型转换，否则不如自定义一个 `struct`，或者拷贝到 `char[]` 里面返回 `[bytenum]` 好了。

嗯，改了下

```c
int xbyte(unsigned word, int bytenum)
{
    unsigned y = (word >> (bytenum << 3));
    y <<= 24;
    y >>= 24;
    unsigned sign = y >> 7;
    sign <<= 8;
    return y - sign;
}
```

引用: *最初由 **wil**发布* ![查看帖子](img/showthread.php?p=867268#post867268) 嗯，改了下

```c
int xbyte(unsigned word, int bytenum)
{
    unsigned y = (word >> (bytenum << 3));
    y <<= 24;
    y >>= 24;
    unsigned sign = y &... 
```

其实这个题的本意是考眼力：

for a machine using two's-complement arithmetic and **arithemetic right shifts** with

不过 @wil 的解法也是对滴，在此膜拜一下。
