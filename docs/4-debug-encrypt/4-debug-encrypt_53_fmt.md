
# MD5 的介绍，算法和实现

> 原文：`www.pediy.com/kssd/pediy04/kanxue202.htm`

## 简介

MD5 的全称是 `Message-Digest Algorithm 5`，在 90 年代初由 `MIT` 的计算机科学实验室和 `RSA Data Security Inc` 发明，经 `MD2`、`MD3` 和 `MD4` 发展而来。

`Message-Digest` 泛指字节串(Message)的 `Hash` 变换，就是把一个任意长度的字节串变换成一定长的大整数。请注意我使用了“字节串”而不是“字符串”这个词，是因为这种变换只与字节的值有关，与字符集或编码方式无关。

`MD5` 将任意长度的“字节串”变换成一个 `128bit` 的大整数，并且它是一个不可逆的字符串变换算法。（我刚开始还愚蠢的认为 `MD5` 是可逆的算法 感谢 `Stkman` 大哥的讲解）换句话说就是，即使你看到源程序和算法描述，也无法将一个 `MD5` 的值变换回原始的字符串，从数学原理上说，是因为原始的字符串有无穷多个，这有点象不存在反函数的数学函数。

`MD5` 的典型应用是对一段 `Message`(字节串)产生 `fingerprint`(指纹)，以防止被“篡改”。举个例子，你将一段话写在一个叫 `readme.txt` 文件中，并对这个 `readme.txt` 产生一个 `MD5` 的值并记录在案，然后你可以传播这个文件给别人，别人如果修改了文件中的任何内容，你对这个文件重新计算 `MD5` 时就会发现。如果再有一个第三方的认证机构，用 `MD5` 还可以防止文件作者的“抵赖”，这就是所谓的数字签名应用。

`MD5` 还广泛用于加密和解密技术上，在很多操作系统中，用户的密码是以 `MD5` 值（或类似的其它算法）的方式保存的， 用户 `Login` 的时候，系统是把用户输入的密码计算成 `MD5` 值，然后再去和系统中保存的 `MD5` 值进行比较，而系统并不“知道”用户的密码是什么。

一些黑客破获这种密码的方法是一种被称为“跑字典”的方法。有两种方法得到字典，一种是日常搜集的用做密码的字符串表，另一种是用排列组合方法生成的，先用 `MD5` 程序计算出这些字典项的 `MD5` 值，然后再用目标的 `MD5` 值在这个字典中检索。

即使假设密码的最大长度为 `8`，同时密码只能是字母和数字，共 `26+26+10=62` 个字符，排列组合出的字典的项数则是 `P(62,1)+P(62,2)….+P(62,8)`，那也已经是一个很天文的数字了，存储这个字典就需要 `TB` 级的磁盘组，而且这种方法还有一个前提，就是能获得目标账户的密码 `MD5` 值的情况下才可以。

在软件的加密保护中 很多软件采用 `MD5` 保护 但是由于 `MD5` 算法为不可逆算法 所以所有的软件都是使用 `MD5` 算法作为一个加密的中间步骤，比如对用户名做一个 `MD5` 变换,结果再进行一个可逆的加密变换，做注册机时也只要先用 `MD5` 变换，然后再用一个逆算法。所以对于破解者来说只要能看出是 `MD5` 就很容易了。

`MD5` 代码的特点明显，跟踪时很容易发现，如果软件采用 `MD5` 算法，在数据初始化的时候必然用到以下的四个常数

`0x67452301`
`0xefcdab89`
`0x98badcfe`
`0x10325476`

若常数不等 则可能是变形的 `MD5` 算法 或者根本就不是这个算法。在内存了也就是

`01 23 45 67 89 ab cd ef fe dc ......32 10` `16` 个字节

## MD5 算法

### 第一步：增加填充

增加 `padding` 使得数据长度（`bit` 为单位）模 `512` 为 `448`。如果数据长度正好是模 `512` 为 `448`，增加 `512` 个填充 `bit`，也就是说填充的个数为 `1-512`。第一个 `bit` 为 `1`，其余全部为 `0`。

### 第二步：补足长度

将数据长度转换为 `64bit` 的数值，如果长度超过 `64bit` 所能表示的数据长度的范围，值保留最后 `64bit`，增加到前面填充的数据后面，使得最后的数据为 `512bit` 的整数倍。也就是 `32bit` 的 `16` 倍的整数倍。在 `RFC1321` 中，`32bit` 称为一个 `word`。

### 第三步：初始化变量

用到 `4` 个变量，分别为 `A`、`B`、`C`、`D`，均为 `32bit` 长。初始化为：

`A: 01 23 45 67`
`B: 89 ab cd ef`
`C: fe dc ba 98`
`D: 76 54 32 10`

### 第四步：数据处理

首先定义 `4` 个辅助函数：

`F(X,Y,Z) = XY v not(X) Z`
`G(X,Y,Z) = XZ v Y not(Z)`
`H(X,Y,Z) = X xor Y xor Z`
`I(X,Y,Z) = Y xor (X v not(Z))`

其中：`XY` 表示按位与，`X v Y` 表示按位或，`not(X)`表示按位取反。`xor` 表示按位异或。

函数中的 `X`、`Y`、`Z` 均为 `32bit`。

定义一个需要用到的数组：`T(i)`，`i` 取值 `1-64`，`T(i)` 等于 `abs(sin(i))` 的 `4294967296` 倍的整数部分，`i` 为弧度。

假设前三步处理后的数据长度为 `32*16*Nbit`

### 第五步：输出

最后得到的 `ABCD` 为输出结果，共 `128bit`。`A` 为低位，`D` 为高位。

## MD5 在编程中的实现

下面来看看如何在 `C` 语言和 `VB` 中实现 `MD5` 算法

```c
/*

#ifndef PROTOTYPES

#define PROTOTYPES 0

#endif

typedef unsigned char *POINTER;

typedef unsigned short int UINT2;

typedef unsigned long int UINT4;

#if PROTOTYPES

#define PROTO_LIST(list) list

#else

#define PROTO_LIST(list) ()

#endif

——— MD5.h ————————

typedef struct {

  UINT4 state[4];

  UINT4 count[2];

  unsigned char buffer[64];

} MD5_CTX;

void MD5Init PROTO_LIST ((MD5_CTX *));

void MD5Update PROTO_LIST

  ((MD5_CTX *, unsigned char *, unsigned int));

void MD5Final PROTO_LIST ((unsigned char [16], MD5_CTX *));

******* MD5C.C *******

#include "global.h"

#include "md5.h"

#define S11 7

#define S12 12

#define S13 17

#define S14 22

#define S21 5

#define S22 9

#define S23 14

#define S24 20

#define S31 4

#define S32 11

#define S33 16

#define S34 23

#define S41 6

#define S42 10

#define S43 15

#define S44 21

static void MD5Transform PROTO_LIST ((UINT4 [4], unsigned char [64]));

static void Encode PROTO_LIST

  ((unsigned char *, UINT4 *, unsigned int));

static void Decode PROTO_LIST

  ((UINT4 *, unsigned char *, unsigned int));

static void MD5_memcpy PROTO_LIST ((POINTER, POINTER, unsigned int));

static void MD5_memset PROTO_LIST ((POINTER, int, unsigned int));

static unsigned char PADDING[64] = {

  0x80, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,

  0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0,

  0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0

};

/* 定义 F G H I 为四个基数

#define F(x, y, z) (((x) & (y)) | ((~x) & (z)))

#define G(x, y, z) (((x) & (z)) | ((y) & (~z)))

#define H(x, y, z) ((x) ^ (y) ^ (z))

#define I(x, y, z) ((y) ^ ((x) | (~z)))

#define ROTATE_LEFT(x, n) (((x) << (n)) | ((x) >> (32-(n))))

#define FF(a, b, c, d, x, s, ac) { \

(a) += F ((b), (c), (d)) + (x) + (UINT4)(ac); \

(a) = ROTATE_LEFT ((a), (s)); \

(a) += (b); \

  }

#define GG(a, b, c, d, x, s, ac) { \

(a) += G ((b), (c), (d)) + (x) + (UINT4)(ac); \

(a) = ROTATE_LEFT ((a), (s)); \

(a) += (b); \

  }

#define HH(a, b, c, d, x, s, ac) { \

(a) += H ((b), (c), (d)) + (x) + (UINT4)(ac); \

(a) = ROTATE_LEFT ((a), (s)); \

(a) += (b); \

  }

#define II(a, b, c, d, x, s, ac) { \

(a) += I ((b), (c), (d)) + (x) + (UINT4)(ac); \

(a) = ROTATE_LEFT ((a), (s)); \

(a) += (b); \

  }

/*开始进行 MD5 计算

void MD5Init (context)

MD5_CTX *context;

{

  context->count[0] = context->count[1] = 0;

  /* 在这里定义四个常数，也就是我们刚才讲到的四个特征数.

  context->state[0] = 0x67452301;

  context->state[1] = 0xefcdab89;

  context->state[2] = 0x98badcfe;

  context->state[3] = 0x10325476;

}

void MD5Update (context, input, inputLen)

MD5_CTX *context;

unsigned char *input;

unsigned int inputLen;

{

  unsigned int i, index, partLen;

  index = (unsigned int)((context->count[0] >> 3) & 0x3F);

  if ((context->count[0] += ((UINT4)inputLen << 3)) < ((UINT4)inputLen << 3))

context->count[1]++;

  context->count[1] += ((UINT4)inputLen >> 29);

  partLen = 64 - index;

  if (inputLen >= partLen) {

MD5_memcpy

  ((POINTER)&context->buffer[index], (POINTER)input, partLen);

MD5Transform (context->state, context->buffer);

for (i = partLen; i + 63 < inputLen; i += 64)

  MD5Transform (context->state, &input[i]);

index = 0;

  }

  else

i = 0;

  MD5_memcpy

((POINTER)&context->buffer[index], (POINTER)&input[i],

  inputLen-i);

}

void MD5Final (digest, context)

unsigned char digest[16];

MD5_CTX *context;

{

  unsigned char bits[8];

  unsigned int index, padLen;

  Encode (bits, context->count, 8);

  index = (unsigned int)((context->count[0] >> 3) & 0x3f);

  padLen = (index < 56) ? (56 - index) : (120 - index);

  MD5Update (context, PADDING, padLen);

  MD5Update (context, bits, 8);

  Encode (digest, context->state, 16);

  MD5_memset ((POINTER)context, 0, sizeof (*context));

}

static void MD5Transform (state, block)

UINT4 state[4];

unsigned char block[64];

{

  UINT4 a = state[0], b = state[1], c = state[2], d = state[3], x[16];

  Decode (x, block, 64);

  /* 第一轮循环 */

  FF (a, b, c, d, x[ 0], S11, 0xd76aa478); /* 1 */

  FF (d, a, b, c, x[ 1], S12, 0xe8c7b756); /* 2 */

  FF (c, d, a, b, x[ 2], S13, 0x242070db); /* 3 */

  FF (b, c, d, a, x[ 3], S14, 0xc1bdceee); /* 4 */

  FF (a, b, c, d, x[ 4], S11, 0xf57c0faf); /* 5 */

  FF (d, a, b, c, x[ 5], S12, 0x4787c62a); /* 6 */

  FF (c, d, a, b, x[ 6], S13, 0xa8304613); /* 7 */

  FF (b, c, d, a, x[ 7], S14, 0xfd469501); /* 8 */
```



```vb
FF (a, b, c, d, x[ 8], S11, 0x698098d8); /* 9 */   
FF (d, a, b, c, x[ 9], S12, 0x8b44f7af); /* 10 */   
FF (c, d, a, b, x[10], S13, 0xffff5bb1); /* 11 */   
FF (b, c, d, a, x[11], S14, 0x895cd7be); /* 12 */   
FF (a, b, c, d, x[12], S11, 0x6b901122); /* 13 */   
FF (d, a, b, c, x[13], S12, 0xfd987193); /* 14 */   
FF (c, d, a, b, x[14], S13, 0xa679438e); /* 15 */   
FF (b, c, d, a, x[15], S14, 0x49b40821); /* 16 */   
/* 第二轮循环 */   
GG (a, b, c, d, x[ 1], S21, 0xf61e2562); /* 17 */   
GG (d, a, b, c, x[ 6], S22, 0xc040b340); /* 18 */   
GG (c, d, a, b, x[11], S23, 0x265e5a51); /* 19 */   
GG (b, c, d, a, x[ 0], S24, 0xe9b6c7aa); /* 20 */   
GG (a, b, c, d, x[ 5], S21, 0xd62f105d); /* 21 */   
GG (d, a, b, c, x[10], S22,  0x2441453); /* 22 */   
GG (c, d, a, b, x[15], S23, 0xd8a1e681); /* 23 */   
GG (b, c, d, a, x[ 4], S24, 0xe7d3fbc8); /* 24 */   
GG (a, b, c, d, x[ 9], S21, 0x21e1cde6); /* 25 */   
GG (d, a, b, c, x[14], S22, 0xc33707d6); /* 26 */   
GG (c, d, a, b, x[ 3], S23, 0xf4d50d87); /* 27 */   
GG (b, c, d, a, x[ 8], S24, 0x455a14ed); /* 28 */   
GG (a, b, c, d, x[13], S21, 0xa9e3e905); /* 29 */   
GG (d, a, b, c, x[ 2], S22, 0xfcefa3f8); /* 30 */   
GG (c, d, a, b, x[ 7], S23, 0x676f02d9); /* 31 */   
GG (b, c, d, a, x[12], S24, 0x8d2a4c8a); /* 32 */   
/* 第三轮循环 */   
HH (a, b, c, d, x[ 5], S31, 0xfffa3942); /* 33 */   
HH (d, a, b, c, x[ 8], S32, 0x8771f681); /* 34 */   
HH (c, d, a, b, x[11], S33, 0x6d9d6122); /* 35 */   
HH (b, c, d, a, x[14], S34, 0xfde5380c); /* 36 */   
HH (a, b, c, d, x[ 1], S31, 0xa4beea44); /* 37 */   
HH (d, a, b, c, x[ 4], S32, 0x4bdecfa9); /* 38 */   
HH (c, d, a, b, x[ 7], S33, 0xf6bb4b60); /* 39 */   
HH (b, c, d, a, x[10], S34, 0xbebfbc70); /* 40 */   
HH (a, b, c, d, x[13], S31, 0x289b7ec6); /* 41 */   
HH (d, a, b, c, x[ 0], S32, 0xeaa127fa); /* 42 */   
HH (c, d, a, b, x[ 3], S33, 0xd4ef3085); /* 43 */   
HH (b, c, d, a, x[ 6], S34,  0x4881d05); /* 44 */   
HH (a, b, c, d, x[ 9], S31, 0xd9d4d039); /* 45 */   
HH (d, a, b, c, x[12], S32, 0xe6db99e5); /* 46 */   
HH (c, d, a, b, x[15], S33, 0x1fa27cf8); /* 47 */   
HH (b, c, d, a, x[ 2], S34, 0xc4ac5665); /* 48 */   
/* 第四轮循环 */   
II (a, b, c, d, x[ 0], S41, 0xf4292244); /* 49 */   
II (d, a, b, c, x[ 7], S42, 0x432aff97); /* 50 */   
II (c, d, a, b, x[14], S43, 0xab9423a7); /* 51 */   
II (b, c, d, a, x[ 5], S44, 0xfc93a039); /* 52 */   
II (a, b, c, d, x[12], S41, 0x655b59c3); /* 53 */   
II (d, a, b, c, x[ 3], S42, 0x8f0ccc92); /* 54 */   
II (c, d, a, b, x[10], S43, 0xffeff47d); /* 55 */   
II (b, c, d, a, x[ 1], S44, 0x85845dd1); /* 56 */   
II (a, b, c, d, x[ 8], S41, 0x6fa87e4f); /* 57 */   
II (d, a, b, c, x[15], S42, 0xfe2ce6e0); /* 58 */   
II (c, d, a, b, x[ 6], S43, 0xa3014314); /* 59 */   
II (b, c, d, a, x[13], S44, 0x4e0811a1); /* 60 */   
II (a, b, c, d, x[ 4], S41, 0xf7537e82); /* 61 */   
II (d, a, b, c, x[11], S42, 0xbd3af235); /* 62 */   
II (c, d, a, b, x[ 2], S43, 0x2ad7d2bb); /* 63 */   
II (b, c, d, a, x[ 9], S44, 0xeb86d391); /* 64 */   
state[0] += a;   
state[1] += b;   
state[2] += c;   
state[3] += d;   
MD5_memset ((POINTER)x, 0, sizeof (x));   
}   

static void Encode (output, input, len)   
unsigned char *output;   
UINT4 *input;   
unsigned int len;   
{   
  unsigned int i, j;   
  for (i = 0, j = 0; j < len; i++, j += 4) {   
output[j] = (unsigned char)(input[i] & 0xff);   
output[j+1] = (unsigned char)((input[i] >> 8) & 0xff);   
output[j+2] = (unsigned char)((input[i] >> 16) & 0xff);   
output[j+3] = (unsigned char)((input[i] >> 24) & 0xff);   
  }   
}   

static void Decode (output, input, len)   
UINT4 *output;   
unsigned char *input;   
unsigned int len;   
{   
  unsigned int i, j;   
  for (i = 0, j = 0; j < len; i++, j += 4)   
output[i] = ((UINT4)input[j]) | (((UINT4)input[j+1]) << 8) |   
  (((UINT4)input[j+2]) << 16) | (((UINT4)input[j+3]) << 24);   
}   

static void MD5_memcpy (output, input, len)   
POINTER output;   
POINTER input;   
unsigned int len;   
{   
  unsigned int i;   
  for (i = 0; i < len; i++) output[i] = input[i];   
}   

static void MD5_memset (output, value, len)   
POINTER output;   
int value;   
unsigned int len;   
{   
  unsigned int i;   
  for (i = 0; i < len; i++)   
((char *)output)[i] = (char)value;   
}
```

————————————————C 代码结束——————————

### 在 VB 中实现 MD5 算法

——————————————————————————————————————————

```vb
Option Explicit   
Dim w1 As String, w2 As String, w3 As String, w4 As String   

Function MD5F(ByVal tempstr As String, ByVal w As String, ByVal X As String, ByVal y As String, ByVal z As String, ByVal Xin As String, ByVal qdata As String, ByVal rots As Integer)   
MD5F = BigMod32Add(RotLeft(BigMod32Add(BigMod32Add(w, tempstr), BigMod32Add(Xin, qdata)), rots), X)   
End Function   

Sub MD5F1(w As String, ByVal X As String, ByVal y As String, ByVal z As String, ByVal Xin As String, ByVal qdata As String, ByVal rots As Integer)   
Dim tempstr As String   
    tempstr = BigXOR(z, BigAND(X, BigXOR(y, z)))   
    w = MD5F(tempstr, w, X, y, z, Xin, qdata, rots)   
End Sub   

Sub MD5F2(w As String, ByVal X As String, ByVal y As String, ByVal z As String, ByVal Xin As String, ByVal qdata As String, ByVal rots As Integer)   
Dim tempstr As String   
    tempstr = BigXOR(y, BigAND(z, BigXOR(X, y)))   
    w = MD5F(tempstr, w, X, y, z, Xin, qdata, rots)   
End Sub   

Sub MD5F3(w As String, ByVal X As String, ByVal y As String, ByVal z As String, ByVal Xin As String, ByVal qdata As String, ByVal rots As Integer)   
Dim tempstr As String   
    tempstr = BigXOR(X, BigXOR(y, z))   
    w = MD5F(tempstr, w, X, y, z, Xin, qdata, rots)   
End Sub   

Sub MD5F4(w As String, ByVal X As String, ByVal y As String, ByVal z As String, ByVal Xin As String, ByVal qdata As String, ByVal rots As Integer)   
Dim tempstr As String   
    tempstr = BigXOR(y, BigOR(X, BigNOT(z)))   
    w = MD5F(tempstr, w, X, y, z, Xin, qdata, rots)   
End Sub   

Function MD5_Calc(ByVal hashthis As String) As String   
ReDim buf(0 To 3) As String   
ReDim Xin(0 To 15) As String   
Dim tempnum As Integer, tempnum2 As Integer, loopit As Integer, loopouter As Integer, loopinner As Integer   
Dim a As String, b As String, c As String, d As String   
    tempnum = 8 * Len(hashthis)   
    hashthis = hashthis + Chr$(128) 'Add binary 10000000   
    tempnum2 = 56 - Len(hashthis) Mod 64   
    If tempnum2 < 0 Then   
        tempnum2 = 64 + tempnum2   
    End If   
    hashthis = hashthis + String$(tempnum2, Chr$(0))   
    For loopit = 1 To 8   
        hashthis = hashthis + Chr$(tempnum Mod 256)   
        tempnum = tempnum - tempnum Mod 256   
        tempnum = tempnum / 256   
    Next loopit   
    buf(0) = "67452301"   
    buf(1) = "efcdab89"   
    buf(2) = "98badcfe"   
    buf(3) = "10325476"   
For loopouter = 0 To Len(hashthis) / 64 - 1   
        a = buf(0)   
        b = buf(1)   
        c = buf(2)   
        d = buf(3)   
        ' Get the 512 bits   
        For loopit = 0 To 15   
            Xin(loopit) = ""   
            For loopinner = 1 To 4   
                Xin(loopit) = Hex$(Asc(Mid$(hashthis, 64 * loopouter + 4 * loopit + loopinner, 1))) + Xin(loopit)   
                If Len(Xin(loopit)) Mod 2 Then Xin(loopit) = "0" + Xin(loopit)   
            Next loopinner   
        Next loopit   
' 第一轮循环   
MD5F1 a, b, c, d, Xin(0), "d76aa478", 7   
MD5F1 d, a, b, c, Xin(1), "e8c7b756", 12   
```



`MD5F1 c, d, a, b, Xin(2), "242070db", 17`

`MD5F1 b, c, d, a, Xin(3), "c1bdceee", 22`

`MD5F1 a, b, c, d, Xin(4), "f57c0faf", 7`

`MD5F1 d, a, b, c, Xin(5), "4787c62a", 12`

`MD5F1 c, d, a, b, Xin(6), "a8304613", 17`

`MD5F1 b, c, d, a, Xin(7), "fd469501", 22`

`MD5F1 a, b, c, d, Xin(8), "698098d8", 7`

`MD5F1 d, a, b, c, Xin(9), "8b44f7af", 12`

`MD5F1 c, d, a, b, Xin(10), "ffff5bb1", 17`

`MD5F1 b, c, d, a, Xin(11), "895cd7be", 22`

`MD5F1 a, b, c, d, Xin(12), "6b901122", 7`

`MD5F1 d, a, b, c, Xin(13), "fd987193", 12`

`MD5F1 c, d, a, b, Xin(14), "a679438e", 17`

`MD5F1 b, c, d, a, Xin(15), "49b40821", 22`

' 第二轮循环
`MD5F2 a, b, c, d, Xin(1), "f61e2562", 5`
`MD5F2 d, a, b, c, Xin(6), "c040b340", 9`
`MD5F2 c, d, a, b, Xin(11), "265e5a51", 14`
`MD5F2 b, c, d, a, Xin(0), "e9b6c7aa", 20`
`MD5F2 a, b, c, d, Xin(5), "d62f105d", 5`
`MD5F2 d, a, b, c, Xin(10), "02441453", 9`
`MD5F2 c, d, a, b, Xin(15), "d8a1e681", 14`
`MD5F2 b, c, d, a, Xin(4), "e7d3fbc8", 20`
`MD5F2 a, b, c, d, Xin(9), "21e1cde6", 5`
`MD5F2 d, a, b, c, Xin(14), "c33707d6", 9`
`MD5F2 c, d, a, b, Xin(3), "f4d50d87", 14`
`MD5F2 b, c, d, a, Xin(8), "455a14ed", 20`
`MD5F2 a, b, c, d, Xin(13), "a9e3e905", 5`
`MD5F2 d, a, b, c, Xin(2), "fcefa3f8", 9`
`MD5F2 c, d, a, b, Xin(7), "676f02d9", 14`
`MD5F2 b, c, d, a, Xin(12), "8d2a4c8a", 20`

' 第三轮循环
`MD5F3 a, b, c, d, Xin(5), "fffa3942", 4`
`MD5F3 d, a, b, c, Xin(8), "8771f681", 11`
`MD5F3 c, d, a, b, Xin(11), "6d9d6122", 16`
`MD5F3 b, c, d, a, Xin(14), "fde5380c", 23`
`MD5F3 a, b, c, d, Xin(1), "a4beea44", 4`
`MD5F3 d, a, b, c, Xin(4), "4bdecfa9", 11`
`MD5F3 c, d, a, b, Xin(7), "f6bb4b60", 16`
`MD5F3 b, c, d, a, Xin(10), "bebfbc70", 23`
`MD5F3 a, b, c, d, Xin(13), "289b7ec6", 4`
`MD5F3 d, a, b, c, Xin(0), "e27fa", 11`
`MD5F3 c, d, a, b, Xin(3), "d4ef3085", 16`
`MD5F3 b, c, d, a, Xin(6), "04881d05", 23`
`MD5F3 a, b, c, d, Xin(9), "d9d4d039", 4`
`MD5F3 d, a, b, c, Xin(12), "e6db99e5", 11`
`MD5F3 c, d, a, b, Xin(15), "1fa27cf8", 16`
`MD5F3 b, c, d, a, Xin(2), "c4ac5665", 23`

' 第四轮循环
`MD5F4 a, b, c, d, Xin(0), "f4292244", 6`
`MD5F4 d, a, b, c, Xin(7), "432aff97", 10`
`MD5F4 c, d, a, b, Xin(14), "ab9423a7", 15`
`MD5F4 b, c, d, a, Xin(5), "fc93a039", 21`
`MD5F4 a, b, c, d, Xin(12), "655b59c3", 6`
`MD5F4 d, a, b, c, Xin(3), "8f0ccc92", 10`
`MD5F4 c, d, a, b, Xin(10), "ffeff47d", 15`
`MD5F4 b, c, d, a, Xin(1), "85845dd1", 21`
`MD5F4 a, b, c, d, Xin(8), "6fa87e4f", 6`
`MD5F4 d, a, b, c, Xin(15), "fe2ce6e0", 10`
`MD5F4 c, d, a, b, Xin(6), "a3014314", 15`
`MD5F4 b, c, d, a, Xin(13), "4e0811a1", 21`
`MD5F4 a, b, c, d, Xin(4), "f7537e82", 6`
`MD5F4 d, a, b, c, Xin(11), "bd3af235", 10`
`MD5F4 c, d, a, b, Xin(2), "2ad7d2bb", 15`
`MD5F4 b, c, d, a, Xin(9), "eb86d391", 21`

`buf(0) = BigAdd(buf(0), a)`
`buf(1) = BigAdd(buf(1), b)`
`buf(2) = BigAdd(buf(2), c)`
`buf(3) = BigAdd(buf(3), d)`
`Next loopouter`

`hashthis = ""`
`For loopit = 0 To 3`
`For loopinner = 3 To 0 Step -1`
`hashthis = hashthis + Chr(Val("&H" + Mid$(buf(loopit), 1 + 2 \* loopinner, 2)))`
`Next loopinner`
`Next loopit`

`MD5_Calc = hashthis`
`End Function`

`Function BigMod32Add(ByVal value1 As String, ByVal value2 As String) As String`
`BigMod32Add = Right$(BigAdd(value1, value2), 8)`
`End Function`

`Public Function BigAdd(ByVal value1 As String, ByVal value2 As String) As String`
`Dim valueans As String`
`Dim loopit As Integer, tempnum As Integer`

`tempnum = Len(value1) - Len(value2)`
`If tempnum < 0 Then`
`value1 = Space$(Abs(tempnum)) + value1`
`ElseIf tempnum > 0 Then`
`value2 = Space$(Abs(tempnum)) + value2`
`End If`

`tempnum = 0`
`For loopit = Len(value1) To 1 Step -1`
`tempnum = tempnum + Val("&H" + Mid$(value1, loopit, 1)) + Val("&H" + Mid$(value2, loopit, 1))`
`valueans = Hex$(tempnum Mod 16) + valueans`
`tempnum = Int(tempnum / 16)`
`Next loopit`

`If tempnum <> 0 Then`
`valueans = Hex$(tempnum) + valueans`
`End If`

`BigAdd = Right(valueans, 8)`
`End Function`

`Public Function RotLeft(ByVal value1 As String, ByVal rots As Integer) As String`
`Dim tempstr As String`
`Dim loopit As Integer, loopinner As Integer`
`Dim tempnum As Integer`

`rots = rots Mod 32`

`If rots = 0 Then`
`RotLeft = value1`
`Exit Function`
`End If`

`value1 = Right$(value1, 8)`
`tempstr = String$(8 - Len(value1), "0") + value1`
`value1 = ""`

' 转换成二进制形式
`For loopit = 1 To 8`
`tempnum = Val("&H" + Mid$(tempstr, loopit, 1))`
`For loopinner = 3 To 0 Step -1`
`If tempnum And 2 ^ loopinner Then`
`value1 = value1 + "1"`
`Else`
`value1 = value1 + "0"`
`End If`
`Next loopinner`
`Next loopit`
`tempstr = Mid$(value1, rots + 1) + Left$(value1, rots)`

' 转换为十六进制
`value1 = ""`
`For loopit = 0 To 7`
`tempnum = 0`
`For loopinner = 0 To 3`
`If Val(Mid$(tempstr, 4 \* loopit + loopinner + 1, 1)) Then`

`tempnum = tempnum + 2 ^ (3 - loopinner)`
`End If`
`Next loopinner`
`value1 = value1 + Hex$(tempnum)`
`Next loopit`

`RotLeft = Right(value1, 8)`
`End Function`

`Function BigAND(ByVal value1 As String, ByVal value2 As String) As String`
`Dim valueans As String`
`Dim loopit As Integer, tempnum As Integer`

`tempnum = Len(value1) - Len(value2)`
`If tempnum < 0 Then`
`value2 = Mid$(value2, Abs(tempnum) + 1)`
`ElseIf tempnum > 0 Then`
`value1 = Mid$(value1, tempnum + 1)`
`End If`

`For loopit = 1 To Len(value1)`
`valueans = valueans + Hex$(Val("&H" + Mid$(value1, loopit, 1)) And Val("&H" + Mid$(value2, loopit, 1)))`
`Next loopit`

`BigAND = valueans`
`End Function`

`Function BigNOT(ByVal value1 As String) As String`
`Dim valueans As String`
`Dim loopit As Integer`

`value1 = Right$(value1, 8)`
`value1 = String$(8 - Len(value1), "0") + value1`
`For loopit = 1 To 8`
`valueans = valueans + Hex$(15 Xor Val("&H" + Mid$(value1, loopit, 1)))`
`Next loopit`

`BigNOT = valueans`
`End Function`

`Function BigOR(ByVal value1 As String, ByVal value2 As String) As String`
`Dim valueans As String`
`Dim loopit As Integer, tempnum As Integer`

`tempnum = Len(value1) - Len(value2)`
`If tempnum < 0 Then`
`valueans = Left$(value2, Abs(tempnum))`
`value2 = Mid$(value2, Abs(tempnum) + 1)`
`ElseIf tempnum > 0 Then`
`valueans = Left$(value1, Abs(tempnum))`
`value1 = Mid$(value1, tempnum + 1)`



```
>     End If
>
>     For loopit = 1 To Len(value1)
>         valueans = valueans + Hex$(Val("&H" + Mid$(value1, loopit, 1)) Or Val("&H" + Mid$(value2, loopit, 1)))
>     Next loopit
>
>     BigOR = valueans
> End Function
>
> Function BigXOR(ByVal value1 As String, ByVal value2 As String) As String
> Dim valueans As String
> Dim loopit As Integer, tempnum As Integer
>
>     tempnum = Len(value1) - Len(value2)
>     If tempnum < 0 Then
>         valueans = Left$(value2, Abs(tempnum))
>         value2 = Mid$(value2, Abs(tempnum) + 1)
>     ElseIf tempnum > 0 Then
>         valueans = Left$(value1, Abs(tempnum))
>         value1 = Mid$(value1, tempnum + 1)
>     End If
>
>     For loopit = 1 To Len(value1)
>         valueans = valueans + Hex$(Val("&H" + Mid$(value1, loopit, 1)) Xor Val("&H" + Mid$(value2, loopit, 1)))
>     Next loopit
>
>     BigXOR = Right(valueans, 8)
> End Function
>
> ————————————————————————————————————————————
```

呵呵 洋洋洒洒的一大篇可是我查找了不少资料才写出来 `^\_^` 大概可以当成我们学校的 Assignment 了 呵呵 感谢大家看完 最后给大家出到题吧 大家可以试试自己的运气啊 就是：

`51E5D4BD3323A02CCCDD0472AE2DC20B` 这组数是我通过 `MD5` 算法加密一组字符串后产生的一组密码 大家猜猜我加密的字符串是什么 `^\_^` 提示一下————加空格一共 `20` 位 `^\_^`

`^\_^` `88`

仅以次文献给我们可爱的组织 `CCG` 希望它能蒸蒸日上！

`娃娃(NYDoll)`

属于中国破解组织 `CHiNA CrACKiNG GrOUp` `4dd466d0e1e0cc123afbe504da2462ca`
