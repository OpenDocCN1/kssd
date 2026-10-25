
# 对 sizeof() 操作符的理解

> 原文：[`www.pediy.com/kssd/pediy10/64014.html`](https://www.pediy.com/kssd/pediy10/64014.html)

本来想在 `google` 上看看 `#pragma pack(1)` 是什么意思的，却看到它还会对 `sizeof` 操作符有影响，所以就想搞清楚它们。

在 `google` 上有[很多文章(可以先看看)](http://blog.csdn.net/fengsanshao/archive/2007/03/18/1533036.aspx)都讲了它们的关系，可是都感觉很乱，所以自己总结了一下。

`#pragma pack(x)` 是通知编译器内存对齐的指令

`sizeof()` 是返回一个对象或类型名的字节长度的操作符

### 文字叙述:

```
如果定义了 #pragma pack(x), 判断 x 是否小于结构中最大的数据类型字节数
是: 结构中各个数据类型按 x 对齐,
否: 按默认对齐
没有定义 #pragma pack(x), 按默认对齐

默认对齐是这样子的:
如果当前数据类型不是最后一个, 而且当前数据类型字节数小于下一个数据类型字节数, 则当前数据类型字节数按下一个数据类型字节数对齐
最后, 结构总字节数按最大数据类型字节数对齐
```

### 伪代码叙述:

```c
void 按默认对齐()
{
    for(i=0;i<数据类型个数;++i)
    {
        if(当前数据类型<>最后一个数据类型&&当前数据类型字节数<下一个数据类型字节数)
        当前数据类型字节数按下一个数据类型字节数对齐;
    }
    结构总字节数按最大数据类型字节数对齐；
}

if(#pragma pack(x)==exist)
{
    if(x<结构中最大数据类型字节数)
    结构中各个数据类型按 x 对齐;
    else
    按默认对齐();
}
else
按默认对齐();
```

大家可以做做实验验证一下![](img/f43c1e514327696dccbb0da997496b2a.jpg "扮鬼脸")

我提供一下网络上别人的实验:

[来源](http://blog.vckbase.com/zhangjw_cn/archive/2005/08/09/10701.html)

```c
struct sample1
{
    char a;        /// sizeof(char) = 1
    double b;      /// sizeof(double) = 8
};

///default 16
#pragma pack( 4 )   12
#pragma pack( 2 )   10
#pragma pack( 1 )   9
#pragma pack( 16 ) 16

struct sample2
{
    char a;     ///1
    int b;      ///4
};

#pragma pack( 8 )    8
#pragma pack( 16 )   8

struct sample3
{
    char a;  ///1
    int b;   ///4
    char c;  ///1
};

///default              ///12
#pragma pack( 4 )   ///12
#pragma pack( 2 )   ///08
#pragma pack( 1 )   ///06
#pragma pack( 16 ) ///12

struct sample4
{
    char a;     ///1
    int b;      ///4
    double c;   ///8
    char d;     ///1
};

///default:            ///24
#pragma pack( 4 )   ///20
#pragma pack( 2 )   ///16
#pragma pack( 1 )   ///14
#pragma pack( 16 ) ///24

struct sample5
{
    char a;     ///1
    double c;   ///8
    int b;      ///4
    char d;     ///1
};
```

编译器给出结果：24、20、16、14、24
