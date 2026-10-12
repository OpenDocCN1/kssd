
# 在 VB 中如何处理中文名

> 原文：[`www.pediy.com/kssd/pediy05/pediy50400.htm`](https://www.pediy.com/kssd/pediy05/pediy50400.htm)

我们知道在处理字符串时,VB 把一个汉字当成一个字符来进行出来,结果导致按一般情况做出来的 `key` 不支持中文,所以我们要做出支持中文的 `VBkey`,就必须正确的得到组成一个汉字的两个 `ASCII` 码,如果得到呢?这里有两种方法:

第一种:

1. 使用 `Strconv()` 函数进行字符串转换.
   - 该函数的作用是对输入的字符串进行编码转换,返回值是一个字符串.该函数原型如下:
   - `StrConv(string, conversion, LCID)`
   - `string`:要转换的字符串(必须)
   - `conversion`:整数类型,表示要进行何种转换(必须)
   - `LCID`:本地 `LocaleID`,相当于语种类型(可选)
   - 如果我们要获得组成汉字的两个 `ASCII` 码,`conversion` 可以选择为 `vbFromUnicode`,表示将 `Unicode` 转换成本地缺省的代码页,如果你的是中文 `windows`,就可以通过下一步获得组成汉字的两个 `ASCII` 码了.(其他语种的 `windows` 也许不可以,欢迎测试)

2. 提取 `ASCII` 码:
   - 由于 `Strconv` 的返回值是一个字符串,如何得到字符串中每个字符的 `ASCII` 码了,很多人会想到用 `Mid$` 函数或 `MidB$` 函数,可惜在这里如果用这两个函数的话,你会发现你有很多字符无法得到,为什么呢?因为 `Mid$()`、`MidB$()` 都无法实现对大于 `ASCII` 码大于 127 的字符的截取,所以我们必须采用其他的方法.在 `VB` 中 `Byte` 类型的数组可以直接跟字符串数组进行相互赋值,如果将字符串赋给一个 `Byte` 类型的数组,如果该字符串中的字符是汉字(必须是转换后的),则用一个数组的两个元素来保存该字符的 `ASCII` 码,否则用一个数组元素.
   - 比如:

```vb
dim a() as byte
a=strconv("我 a",vbFromUnicode)
'则 a(0)=206,a(1)=210,a(2)=97 注意:数组下标是从 0 开始
```

综合一、二步,就可以写出完整的代码:

```vb
dim AscArray() as byte
dim strSN as string
dim i as long

AscArray=strconv(strSN,vbFromUnicode)

for i=0 to Ubound(AscArray)
  '处理过程
next i
```

第二种方法:

- 直接采用 `ASC()` 函数,对 `ASC()` 的返回值进行处理就可以准确的得到组成一个汉字的两个字节的 `ASCII` 码.
- 如:

```vb
dim HighAsc as byte
dim LowAsc as byte
dim lngAscii as long

lngAscii=ASC("我")

HighAsc=int((65536+lngAscii)/256)

LowAsc=(65536+lngAscii) mod 256

'结果 HighAsc=206,LowAsc=210
```

本文结束.

再加一个：如果得到含中文字符的字符串的长度

用 `LenB(strconv(字符串,vbFromUnicode))` 就可以得到该字符串的长度。

4dd466d0e1e0cc123afbe504da2462ca
