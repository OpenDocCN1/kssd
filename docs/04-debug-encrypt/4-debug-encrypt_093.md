
# RSA 与大数运算

> 原文：[`www.pediy.com/kssd/pediy05/pediy50664.htm`](https://www.pediy.com/kssd/pediy05/pediy50664.htm)

基于以下原因，俺估计 `RSA` 算法会被越来越多的共享软件采用：

1. 原理简洁易懂
2. 加密强度高
3. 专利限制已过期
4. 看雪老大三番五次呼吁共享软件采用成熟的非对称加密技术

所以，大家应该对 `RSA` 算法进行深入了解。

`RSA` 依赖大数运算，目前主流 `RSA` 算法都建立在 512 位到 1024 位的大数运算之上，所以我们在现阶段首先需要掌握 1024 位的大数运算原理。

大多数的编译器只能支持到 64 位的整数运算，即我们在运算中所使用的整数必须小于等于 64 位，即：`0xffffffffffffffff` 也就是 `18446744073709551615`，这远远达不到 `RSA` 的需要，于是需要专门建立大数运算库来解决这一问题。

最简单的办法是将大数当作字符串进行处理，也就是将大数用 10 进制字符数组进行表示，然后模拟人们手工进行“竖式计算”的过程编写其加减乘除函数。但是这样做效率很低，因为 1024 位的大数其 10 进制数字个数就有数百个，对于任何一种运算，都需要在两个有数百个元素的数组空间上做多重循环，还需要许多额外的空间存放计算的进位退位标志及中间结果。当然其优点是算法符合人们的日常习惯，易于理解。

另一种思路是将大数当作一个二进制流进行处理，使用各种移位和逻辑操作来进行加减乘除运算，但是这样做代码设计非常复杂，可读性很低，难以理解也难以调试。

于是俺琢磨了一种介于两者之间的思路：

将大数看作一个 n 进制数组，对于目前的 32 位系统而言 n 可以取值为 2 的 32 次方，即 `0x10000000`，假如将一个 1024 位的大数转化成 `0x10000000` 进制，它就变成了 32 位，而每一位的取值范围就不是 0-1 或 0-9，而是 `0-0xffffffff`。我们正好可以用一个无符号长整数来表示这一数值。所以 1024 位的大数就是一个有 32 个元素的 `unsigned long` 数组。而且 `0x100000000` 进制的数组排列与 2 进制流对于计算机来说，实际上是一回事，但是我们完全可以针对 `unsigned long` 数组进行“竖式计算”，而循环规模被降低到了 32 次之内，并且算法很容易理解。

例如大数 `18446744073709551615`，等于“`ffffffff ffffffff`”，它就相当于 10 进制的“`99`”：有两位，每位都是 `ffffffff`。
而大数 `18446744073709551616`，等于“`00000001 00000000 00000000`”，它就相当于 10 进制的“`100`”：有三位，第一位是 `1`，其它两位是 `0`。如果我们要计算 `18446744073709551616 - 18446744073709551615`，就类似于 `100-99`：

```
00000001 00000000 00000000
-             ffffffff ffffffff
--------------------------------
=             0          0          1
```

所以，可以声明大数类如下：

```cpp
/**********************************************/
//大数运算库头文件：BigInt.h
//作者：afanty@vip.sina.com
//版本：1.0 (2003.4.26)
//说明：适用于 MFC
/**********************************************/

#define BI_MAXLEN 40
#define DEC 10
#define HEX 16

class CBigInt
{
public:
    int m_nSign;    //记录大数的符号，支持负值运算
    int m_nLength;  //记录 0x10000000 进制的位数，0-40 之间，相当于 2 进制的 0-1280 位
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

    //将输入的 10 进制或 16 进制字符串转换成大数
    int InPutFromStr(CString& str, const unsigned int system);

    //将大数按 10 进制或 16 进制格式输出到字符串
    int OutPutToStr(CString& str, const unsigned int system);

    //欧几里德算法求：Y=X.Euc(A)，使满足：YX mod A = 1
    CBigInt Euc(CBigInt& A);

    //蒙哥马利算法求：Y=X.Mon(A,B)，使满足：X^A mod B = Y
    CBigInt Mon(CBigInt& A, CBigInt& B);
};
```

注意以上函数的声明格式，完全遵循普通整数运算的习惯，例如大数 `Y=X+Z` 相当于 `Y.Mov(X.Add(Z))`，这样似乎没有 `Mov(Y,Add(X,Z))` 看起来舒服，但是一旦我们重载运算符“`=`”为“`Mov`”，“`+`”为“`Add`”，则 `Y.Mov(X.Add(Z))` 的形式就等价于 `Y=X+Z`。

俺不知道其他编程语言里是否支持运算浮重载，至少这样定义函数格式在 C++ 里可以带来很大的方便。

下面让我们来实现大数类的主要成员函数：

```cpp
/**********************************************/
//大数运算库源文件：BigInt.cpp
//作者：afanty@vip.sina.com
//版本：1.0 (2003.4.26)
//说明：适用于 MFC
/**********************************************/

#include "stdafx.h"
#include "BigInt.h"

//初始化大数为 0
CBigInt::CBigInt()
{
    m_nSign=1;
    m_nLength=1;
    for(int i=0;i<BI_MAXLEN;i++)m_ulvalue[i]=0;
}

//采用缺省的解构函数
CBigInt::~CBigInt()
{
}

//大数比较，如果大数 A 位数比大数 B 多，当然 A>B
//如果位数相同，则从高位开始比较，直到分出大小
int CBigInt::Cmp(CBigInt& A)
{
    if(m_nLength>A.m_nLength)return 1;
    if(m_nLength<A.m_nLength)return -1;
    for(int i=m_nLength-1;i>=0;i--)
    {
        if(m_ulvalue[i]>A.m_ulvalue[i])return 1;
        if(m_ulvalue[i]<A.m_ulvalue[i])return -1;
    }
    return 0;
}

//照搬参数的各属性
CBigInt& CBigInt::Mov(CBigInt& A)
{
    m_nLength=A.m_nLength;
    for(int i=0;i<BI_MAXLEN;i++)m_ulvalue[i]=A.m_ulvalue[i];
    return *this;
}

//大数相加
//调用形式：N.Add(A)，返回值：N+A
//若两大数符号相同，其值相加，否则改变参数符号再调用大数相减函数
/**********************************************/
//例如：
//     A  B  C
//+       D  E
//-------------
//= S  F  G  H
//
//其中，若 C+E<=0xffffffff，则 H=C+E，carry(进位标志)=0
//     若 C+E>0xffffffff，则 H=C+E-0x100000000，carry=1
//
//     若 B+D+carry<=0xfffffff，则 G=B+D，carry=0
//     若 B+D+carry>0xfffffff，则 G=B+D+carry-0x10000000，carry=1
//
//     若 carry=0，则 F=A，S=0
//     若 carry=1，A<0xfffffff，则 F=A+1，S=0
//     若 carry=1，A=0xfffffff，则 F=0，S=1
/**********************************************/
CBigInt CBigInt::Add(CBigInt& A)
{
    CBigInt X;
    if(X.m_nSign==A.m_nSign)
    {
        X.Mov(*this);
        int carry=0;
        unsigned __int64 sum=0;
        if(X.m_nLength<A.m_nLength)X.m_nLength=A.m_nLength;
        for(int i=0;i<X.m_nLength;i++)
        {
            sum=A.m_ulvalue[i];
            sum=sum+X.m_ulvalue[i]+carry;
            X.m_ulvalue[i]=(unsigned long)sum;
            if(sum>0xffffffff)carry=1;
            else carry=0;
        }
        if(X.m_nLength<BI_MAXLEN)
        {
            X.m_ulvalue[X.m_nLength]=carry;
            X.m_nLength+=carry;
        }
        return X;
    }
    else{X.Mov(A);X.m_nSign=1-X.m_nSign;return Sub(X);}
}

//大数相减
//调用形式：N.Sub(A)，返回值：N-A
//若两大数符号相同，其值相减，否则改变参数符号再调用大数相加函数
/**********************************************/
//例如：
//     A  B  C
//-       D  E
//-------------
//=    F  G  H
//
//其中，若 C>=E，则 H=C-E，carry(借位标志)=0
//     若 C<E，则 H=C-E+0x100000000，carry=1
//
//     若 B-carry>=D，则 G=B-carry-D，carry=0
//     若 B-carry<D，则 G=B-carry-D+0x10000000，carry=1
//
//     若 carry=0，则 F=A
//     若 carry=1，A>1，则 F=A-1
//     若 carry=1，A=1，则 F=0
/**********************************************/
CBigInt CBigInt::Sub(CBigInt& A)
{
    CBigInt X;
    if(m_nSign==A.m_nSign)
    {
        X.Mov(*this);
        int cmp=X.Cmp(A);
        if(cmp==0){X.Mov(0);return X;}
        int len,carry=0;
        unsigned __int64 num;
        unsigned long *s,*d;
        if(cmp>0)
        {
            s=X.m_ulvalue;
            d=A.m_ulvalue;
            len=X.m_nLength;
        }
        if(cmp<0)
        {
            s=A.m_ulvalue;
            d=X.m_ulvalue;
            len=A.m_nLength;
            X.m_nSign=1-X.m_nSign;
        }
        for(int i=0;i<len;i++)
        {
            if((s[i]-carry)>=d[i])
            {
                X.m_ulvalue[i]=s[i]-carry-d[i];
                carry=0;
            }
            else
            {
                num=0x100000000+s[i];
                X.m_ulvalue[i]=(unsigned long)(num-carry-d[i]);
                carry=1;
            }
        }
        while(X.m_ulvalue[len-1]==0)len--;
        X.m_nLength=len;
        return X;
    }
    else{X.Mov(A);X.m_nSign=1-X.m_nSign;return Add(X);}
}

//大数相乘
//调用形式：N.Mul(A)，返回值：N*A
/**********************************************/
//例如：
//       A  B  C
//*          D  E
//----------------
//=    S  F  G  H
//+ T  I  J  K
//----------------
//= U  V  L  M  N
//
//其中，SFGH=ABC*E，TIJK=ABC*D
//
//而对于：
//     A  B  C
//*          E
//-------------
```



```
= S F G H
```

其中，若`C*E <= 0xffffffff`，则`H = C*E`，`carry`（进位标志）`= 0`

若`C*E > 0xffffffff`，则`H = (C*E) & 0xffffffff`

`carry = (C*E) / 0xffffffff`

若`B*E + carry <= 0xffffffff`，则`G = B*E + carry`，`carry = 0`

若`B*E + carry > 0xffffffff`，则`G = (B*E + carry) & 0xffffffff`

`carry = (B*E + carry) / 0xffffffff`

若`A*E + carry <= 0xffffffff`，则`F = A*E + carry`，`carry = 0`

若`A*E + carry > 0xffffffff`，则`F = (A*E + carry) & 0xffffffff`

`carry = (A*E + carry) / 0xffffffff`

`S = carry`

```
/*★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★*/
CBigInt CBigInt::Mul(CBigInt& A)
{
    CBigInt X, Y;
    unsigned __int64 mul;
    unsigned long carry;
    for(int i=0; i<A.m_nLength; i++)
    {
        Y.m_nLength = m_nLength;
        carry = 0;
        for(int j=0; j<m_nLength; j++)
        {
            mul = m_ulvalue[j];
            mul = mul * A.m_ulvalue[i] + carry;
            Y.m_ulvalue[j] = (unsigned long)mul;
            carry = (unsigned long)(mul >> 32);
        }
        if(carry && (Y.m_nLength < BI_MAXLEN))
        {
            Y.m_nLength++;
            Y.m_ulvalue[Y.m_nLength-1] = carry;
        }
        if(Y.m_nLength < BI_MAXLEN - i)
        {
            Y.m_nLength += i;
            for(int k=Y.m_nLength-1; k>=i; k--) Y.m_ulvalue[k] = Y.m_ulvalue[k-i];
            for(int k=0; k<i; k++) Y.m_ulvalue[k] = 0;
        }
        X.Mov(X.Add(Y));
    }
    if(m_nSign + A.m_nSign == 1) X.m_nSign = 0;
    else X.m_nSign = 1;
    return X;
}
```

大数相除

调用形式：`N.Div(A)`，返回值：`N/A`

除法的关键在于“试商”，然后就变成了乘法和减法

这里将被除数与除数的试商转化成了被除数最高位与除数最高位的试商

```
CBigInt CBigInt::Div(CBigInt& A)
{
    CBigInt X, Y, Z;
    int len;
    unsigned __int64 num, div;
    unsigned long carry = 0;
    Y.Mov(*this);
    while(Y.Cmp(A) > 0)
    {
        if(Y.m_ulvalue[Y.m_nLength-1] > A.m_ulvalue[A.m_nLength-1])
        {
            len = Y.m_nLength - A.m_nLength;
            div = Y.m_ulvalue[Y.m_nLength-1] / (A.m_ulvalue[A.m_nLength-1] + 1);
        }
        else if(Y.m_nLength > A.m_nLength)
        {
            len = Y.m_nLength - A.m_nLength - 1;
            num = Y.m_ulvalue[Y.m_nLength-1];
            num = (num << 32) + Y.m_ulvalue[Y.m_nLength-2];
            if(A.m_ulvalue[A.m_nLength-1] == 0xffffffff) div = (num >> 32);
            else div = num / (A.m_ulvalue[A.m_nLength-1] + 1);
        }
        else
        {
            X.Mov(X.Add(1));
            break;
        }
        Z.Mov(div);
        Z.m_nLength += len;
        for(int i=Z.m_nLength-1; i>=len; i--) Z.m_ulvalue[i] = Z.m_ulvalue[i-len];
        for(int i=0; i<len; i++) Z.m_ulvalue[i] = 0;
        X.Mov(X.Add(Z));
        Z.Mov(Z.Mul(A));
        Y.Mov(Y.Sub(Z));
    }
    if(Y.Cmp(A) == 0) X.Mov(X.Add(1));
    if(m_nSign + A.m_nSign == 1) X.m_nSign = 0;
    else X.m_nSign = 1;
    return X;
}
```

大数求模

调用形式：`N.Mod(A)`，返回值：`N%A`

求模与求商原理相同

```
CBigInt CBigInt::Mod(CBigInt& A)
{
    CBigInt X, Y;
    int len;
    unsigned __int64 num, div;
    unsigned long carry = 0;
    X.Mov(*this);
    while(X.Cmp(A) > 0)
    {
        if(X.m_ulvalue[X.m_nLength-1] > A.m_ulvalue[A.m_nLength-1])
        {
            len = X.m_nLength - A.m_nLength;
            div = X.m_ulvalue[X.m_nLength-1] / (A.m_ulvalue[A.m_nLength-1] + 1);
        }
        else if(X.m_nLength > A.m_nLength)
        {
            len = X.m_nLength - A.m_nLength - 1;
            num = X.m_ulvalue[X.m_nLength-1];
            num = (num << 32) + X.m_ulvalue[X.m_nLength-2];
            if(A.m_ulvalue[A.m_nLength-1] == 0xffffffff) div = (num >> 32);
            else div = num / (A.m_ulvalue[A.m_nLength-1] + 1);
        }
        else
        {
            X.Mov(X.Sub(A));
            break;
        }
        Y.Mov(div);
        Y.Mov(Y.Mul(A));
        Y.m_nLength += len;
        for(int i=Y.m_nLength-1; i>=len; i--) Y.m_ulvalue[i] = Y.m_ulvalue[i-len];
        for(int i=0; i<len; i++) Y.m_ulvalue[i] = 0;
        X.Mov(X.Sub(Y));
    }
    if(X.Cmp(A) == 0) X.Mov(0);
    return X;
}
```

暂时只给出了十进制字符串的转化

```
int CBigInt::InPutFromStr(CString& str, const unsigned int system=DEC)
{
    int len = str.GetLength();
    Mov(0);
    for(int i=0; i<len; i++)
    {
        Mov(Mul(system));
        int k = str[i] - 48;
        Mov(Add(k));
    }
    return 0;
}
```

暂时只给出了十进制字符串的转化

```
int CBigInt::OutPutToStr(CString& str, const unsigned int system=DEC)
{
    str = "";
    char ch;
    CBigInt X;
    X.Mov(*this);
    while(X.m_ulvalue[X.m_nLength-1] > 0)
    {
        ch = X.Mod(system) + 48;
        str.Insert(0, ch);
        X.Mov(X.Div(system));
    }
    return 0;
}
```

欧几里德算法求：`Y = X.Euc(A)`，使满足：`Y*X mod A = 1`

相当于对不定方程`ax - by = 1`求最小整数解

实际上就是初中学过的辗转相除法

```
/*★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★*/
例如：11x - 49y = 1，求 x

11 x - 49 y = 1    a)
49 % 11 = 5 -> 11 x - 5 y = 1    b)
11 % 5 = 1 -> x - 5 y = 1    c)

令 y=1 代入 c) 式 得 x=6
令 x=6 代入 b) 式 得 y=13
令 y=13 代入 a) 式 得 x=58

/*★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★★*/
CBigInt CBigInt::Euc(CBigInt& A)
{
    CBigInt X, Y;
    X.Mov(*this);
    Y.Mov(A);
    if((X.m_nLength == 1) && (X.m_ulvalue[0] == 1)) return X;
    if((Y.m_nLength == 1) && (Y.m_ulvalue[0] == 1)) { X.Mov(X.Sub(1)); return X; }
    if(X.Cmp(Y) == 1) X.Mov(X.Mod(Y));
    else Y.Mov(Y.Mod(X));
    X.Mov(X.Euc(Y));
    Y.Mov(*this);
    if(Y.Cmp(A) == 1)
    {
        X.Mov(X.Mul(Y));
        X.Mov(X.Sub(1));
        X.Mov(X.Div(A));
    }
    else
    {
        X.Mov(X.Mul(A));
        X.Mov(X.Add(1));
        X.Mov(X.Div(Y));
    }
    return X;
}
```

蒙哥马利算法求：`Y = X.Mon(A, B)`，使满足：`X^A mod B = Y`

俺估计就是高中学过的反复平方法

```
CBigInt CBigInt::Mon(CBigInt& A, CBigInt& B)
{
    CBigInt X, Y, Z;
    X.Mov(1);
    Y.Mov(*this);
    Z.Mov(A);
    while((Z.m_nLength != 1) || Z.m_ulvalue[0])
    {
        if(Z.m_ulvalue[0] & 1)
        {
            Z.Mov(Z.Sub(1));
            X.Mov(X.Mul(Y));
            X.Mov(X.Mod(B));
        }
        else
        {
            Z.Mov(Z.Div(2));
            Y.Mov(Y.Mul(Y));
            Y.Mov(Y.Mod(B));
        }
    }
    return X;
}
```

最后需要说明的是因为在`VC`里面存在一个`__int64`类型可以用来计算进位与借位值，所以将大数当作`0x100000000`进制进行运算是可能的，而在其他编译系统中如果不存在`64`位整形，则可以采用`0x40000000`进制，由于在`0x40000000`进制中，对任何两个“数字”进行四则运算，结果都在`0x3fffffff * 0x3fffffff`之间，小于`0xffffffff`，都可以用一个`32`位无符号整数来表示。事实上《楚汉棋缘》采用的`freelip`大数库正是运用了`0x40000000`进制来表示大数的，所以其反汇编后大数的值在内存中表现出来有些“奇怪”。

好了，俺的大数运算库源代码将附在本贴之后（点击下载），在`VC`中可以直接编译通过，如果大家在使用中发现有`Bug`，希望能够通知俺及时改正，谢谢！

由于代码中的“`TAB`”控制符被浏览器忽略，建议下载源码参照阅读`4dd466d0e1e0cc123afbe504da2462ca`
