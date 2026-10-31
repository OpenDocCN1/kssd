
# 《RSA与大数运算》

**标 题:** 《RSA与大数运算》

**发信人:** afanty

**时 间:** 2003/04/28 04:50pm

**详细信息:**

基于以下原因，俺估计RSA算法会被越来越多的共享软件采用：

1、原理简洁易懂

2、加密强度高

3、专利限制已过期

4、看雪老大三番五次呼吁共享软件采用成熟的非对称加密技术

所以，大家应该对RSA算法进行深入了解。

RSA依赖大数运算，目前主流RSA算法都建立在512位到1024位的
大数运算之上，所以我们在现阶段首先需要掌握1024位的大数
运算原理。

大多数的编译器只能支持到64位的整数运算，即我们在运算中
所使用的整数必须小于等于64位，即：`0xffffffffffffffff`

也就是18446744073709551615，这远远达不到RSA的需要，于是
需要专门建立大数运算库来解决这一问题。

最简单的办法是将大数当作字符串进行处理，也就是将大数用
10进制字符数组进行表示，然后模拟人们手工进行“竖式计算”
的过程编写其加减乘除函数。但是这样做效率很低，因为1024
位的大数其10进制数字个数就有数百个，对于任何一种运算，
都需要在两个有数百个元素的数组空间上做多重循环，还需要
许多额外的空间存放计算的进位退位标志及中间结果。当然其
优点是算法符合人们的日常习惯，易于理解。

另一种思路是将大数当作一个二进制流进行处理，使用各种移
位和逻辑操作来进行加减乘除运算，但是这样做代码设计非常
复杂，可读性很低，难以理解也难以调试。

于是俺琢磨了一种介于两者之间的思路：

将大数看作一个n进制数组，对于目前的32位系统而言n可以取
值为2的32次方，即`0x10000000`，假如将一个1024位的大数转
化成`0x10000000`进制，它就变成了32位，而每一位的取值范围
就不是0-1或0-9，而是0-`0xffffffff`。我们正好可以用一个无
符号长整数来表示这一数值。所以1024位的大数就是一个有32
个元素的`unsigned long`数组。而且`0x100000000`进制的数组排列
与2进制流对于计算机来说，实际上是一回事，但是我们完全
可以针对`unsigned long`数组进行“竖式计算”，而循环规模
被降低到了32次之内，并且算法很容易理解。

例如大数18446744073709551615，等于`ffffffff ffffffff`，
它就相当于10进制的“99”：有两位，每位都是`ffffffff`。

而大数18446744073709551616，等于`00000001 00000000 00000000`，它就相当于10进制的“100”：有三位，第一位是
1，其它两位是0。如果我们要计算18446744073709551616
-18446744073709551615，就类似于100-99：

```
   00000001 00000000 00000000
-
  ffffffff ffffffff
-----------------------------
=
 0         0        1
```

所以，可以声明大数类如下：

```
//大数运算库头文件：BigInt.h
//作者：afanty@vip.sina.com
//版本：1.0
//2003.4.26
//说明：适用于MFC
```
```c
#define BI_MAXLEN 40
#define DEC 10
#define HEX 16

class CBigInt
{
public:
    int m_nSign;    //记录大数的符号，支持负值运算
    int m_nLength;  //记录0x10000000进制的位数，0-40之间，相当于2进制的0-1280位
    unsigned long m_ulvalue[BI_MAXLEN];   //记录每一位的“数字”
    CBigInt();
    ~CBigInt();

    //将大数赋值为另一个大数
    CBigInt& Mov(CBigInt& A);

    //将大数赋值为编译器能够理解的任何整形常数或变量
    CBigInt& Mov(unsigned __int64 A);

    //比较两个大数大小
    int Cmp(CBigInt& A);

    //计算两个大数的和
    CBigInt Add(CBigInt& A);

    //重载函数以支持大数与普通整数相加
    CBigInt Add(long A);

    //计算两个大数的差
    CBigInt Sub(CBigInt& A);

    //重载函数以支持大数与普通整数相减
    CBigInt Sub(long A);

    //计算两个大数的积
    CBigInt Mul(CBigInt& A);

    //重载函数以支持大数与普通整数相乘
    CBigInt Mul(long A);

    //计算两个大数的商
    CBigInt Div(CBigInt& A);

    //重载函数以支持大数与普通整数相除
    CBigInt Div(long A);

    //计算两个大数相除的余数
    CBigInt Mod(CBigInt& A);

    //重载函数以支持大数与普通整数相除求模
    long Mod(long A);

    //将输入的10进制或16进制字符串转换成大数
    int InPutFromStr(CString& str, const unsigned int system);

    //将大数按10进制或16进制格式输出到字符串
    int OutPutToStr(CString& str, const unsigned int system);

    //欧几里德算法求：Y=X.Euc(A)，使满足：YX mod A = 1
    CBigInt Euc(CBigInt& A);

    //蒙哥马利算法求：Y=X.Mon(A,B)，使满足：X^A mod B = Y
    CBigInt Mon(CBigInt& A, CBigInt& B);
};
```

注意以上函数的声明格式，完全遵循普通整数运算的习惯，例如大数
`Y=X+Z`
相当于 `Y.Mov(X.(Add(Z)))`，这样似乎没有`Mov(Y,Add(X,Z))`
看起来舒服，但是一旦我们重载运算符“=”为“`Mov`”，“+”为“`Add`”，
则`Y.Mov(X.(Add(Z)))`的形式就等价于
`Y=X+Z`。

俺不知道其他编程语言里是否支持运算浮重载，至少这样定义函数格式
在C++里可以带来很大的方便。

下面让我们来实现大数类的主要成员函数：

```
//大数运算库源文件：BigInt.cpp
//作者：afanty@vip.sina.com
//版本：1.0
//2003.4.26
//说明：适用于MFC
```
```c
#include "stdafx.h"
#include "BigInt.h"

//初始化大数为0
CBigInt::CBigInt()
{
    m_nSign=1;
    m_nLength=1;
    for(int i=0;i<BI_MAXLEN;i++) m_ulvalue[i]=0;
}

//采用缺省的解构函数
CBigInt::~CBigInt()
{
}

//大数比较，如果大数A位数比大数B多，当然A>B
//如果位数相同，则从高位开始比较，直到分出大小
int CBigInt::Cmp(CBigInt& A)
{
    if(m_nLength>A.m_nLength) return 1;
    if(m_nLength<A.m_nLength) return -1;
    for(int i=m_nLength-1;i>=0;i--)
    {
        if(m_ulvalue[i]>A.m_ulvalue[i]) return 1;
        if(m_ulvalue[i]<A.m_ulvalue[i]) return -1;
    }
    return 0;
}

//照搬参数的各属性
CBigInt& CBigInt::Mov(CBigInt& A)
{
    m_nLength=A.m_nLength;
    for(int i=0;i<BI_MAXLEN;i++) m_ulvalue[i]=A.m_ulvalue[i];
    return *this;
}

//大数相加
//调用形式：N.Add(A)，返回值：N+A
//若两大数符号相同，其值相加，否则改变参数符号再调用大数相减函数
```
例如：

```
     A  B  C
 +       D  E
--------------
= S  F  G  H
```

其中，若`C+E<=0xffffffff`，则`H=C+E`，`carry`进位标志`=0`
若`C+E>0xffffffff`，则`H=C+E-0x100000000`，`carry=1`
若`B+D+carry<=0xffffffff`，则`G=B+D+carry`，`carry=0`
若`B+D+carry>0xffffffff`，则`G=B+D+carry-0x100000000`
