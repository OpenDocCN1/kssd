
# 几个标准 Hash 算法的说明及实现（一）

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-469.htm`](https://www.pediy.com/kssd/pediy07/pediy7-469.htm)

作者：冲出宇宙

说明：作者水平所限，恐怕错误比较多，请兄弟们多多包涵。并提出指正。

昨天刚在这里发了一个帖子，今天就发现被转移到其他地方去了。思考之后，决定把自己写的几个标准的 Hash 函数代码和说明发布到这里来。这下不会把我的东西转移了吧？

首先看看美国的 `FIPS` 下的 Hash 算法标准：

1. `ripemd`
2. `sha`
3. `tiger`
4. `whirlpool`

`md5` 以前好像也是这个里面的标准。不知道是不是真的是。反正现在已经不是了。`md5` 的代码网络上铺天盖地都是的，我们将不讨论和 `md5` 有关的话题。需要代码的请看：

`http://lotusroots.bokee.com` ，那里也包含了 hash 算法的更详细的讨论。

### 1. ripemd

顾名思义 `ripemd` 就是成熟 md 算法，是 Hans Dobbertin 等 3 人在 `md4`, `md5` 的基础上，于 1996 年提出来的。算法共有 4 个标准 `128`、`160`、`256` 和 `320`，其对应输出长度分别为 16 字节、20 字节、32 字节和 40 字节。不过，让人难以致信的是 `ripemd` 的设计者们根本就没有真正设计 256 和 320 位这 2 种标准，他们只是在 128 位和 160 位的基础上，修改了初始参数和 `s-box` 来达到输出为 256 和 320 位的目的。所以，256 位的强度和 128 相当，而 320 位的强度和 160 位相当。`ripemd` 建立在 md 的基础之上，所以，其添加数据的方式和 `md5` 完全一样。

这里只列出 `ripemd128` 的代码。需要其他代码的兄弟可以下载附件。

其参考代码如下：

`ripemd.h` 文件：

```c
#ifndef Goodzzp_RipeMd_05_10_20_12
#define Goodzzp_RipeMd_05_10_20_12

//ripemd128,160,256,320
//program be Goodzzp
//05,10,20
//128 and 160 are referred:
//http://homes.esat.md_kuleuven.be/~bosselae/ripemd160.html

//256 and 320 are not guarranteed to be safe than 128 or 160
//referred: Crypto++5.21

#include "define.h"

//128-320 需要的 5 论变换
#define md_F(x, y, z)    (x ^ y ^ z)
#define md_G(x, y, z)    (z ^ (x & (y^z)))
#define md_H(x, y, z)    (z ^ (x | ~y))
#define md_I(x, y, z)    (y ^ (z & (x^y)))
#define md_J(x, y, z)    (x ^ (y | ~z))

//常量
#define md_k0 0
#define md_k1 0x5a827999UL
#define md_k2 0x6ed9eba1UL
#define md_k3 0x8f1bbcdcUL
#define md_k4 0xa953fd4eUL
#define md_k5 0x50a28be6UL
#define md_k6 0x5c4dd124UL
#define md_k7 0x6d703ef3UL
#define md_k8 0x7a6d76e9UL
#define md_k9 0

// for 128 and 256
#define md1_Subround(f, a, b, c, d, x, s, md_k)        \
 a += f(b, c, d) + x + md_k;\
 a = rotlFixed((UI32)a, s);

//...begin ripemd128
//RipeMd128 散列类
class RipeMD128
{
public:
 //输出结果的长度（字节）
 static UI32 OutDataLength()
 {
  return 128/8;
 }

 //RipeMD128 变换函数
 //out:输出，长度为 16，要求事先已经分配了内存
 //in:输入
 //length:输入值的长度
 void Hash(UI8 *out,const UI8 *in,UI32 length)
 {
  UI32 i = length>>6,j=(length&0x3f),md_k;
  //对数据 64 64 个字节的计算
  for(md_k=0;md_k<i;md_k++)
  {
   StepTransform((UI8 *)(in + md_k * 64),64,length);
  }
  //最后计算尾数
  StepTransform((UI8 *)(in + 64 * i),j,length);
  //拷贝输出
  memcpy(out,MDbuf,16);
  //恢复 m_state 值
  MDbuf[0] = 0x67452301UL;
  MDbuf[1] = 0xefcdab89UL;
  MDbuf[2] = 0x98badcfeUL;
  MDbuf[3] = 0x10325476UL;
 }

 //初始化
 //这里就是 4 个刚开始的值
 RipeMD128()
 {
  MDbuf[0] = 0x67452301UL;
  MDbuf[1] = 0xefcdab89UL;
  MDbuf[2] = 0x98badcfeUL;
  MDbuf[3] = 0x10325476UL;
 }

private:
 //每步的变换函数
 //输入:
 //  data:   要处理的数据块（不大于 64 字节）
 //  dataBlockLen: 数据块的长度
 //  dataTotalLen: 要处理的所有数据块的总长度
 //输出结果保存在 m_state 里面
 void StepTransform(UI8 *data,UI32 dataBlockLen, UI32 dataTotalLen)
 {
  UI8 buffer[64];
  UI32 len=dataTotalLen*8;
  memset(buffer,0,64);//清空数据为 0
  memcpy(buffer,data,dataBlockLen);//拷贝数据到缓冲
  if(dataBlockLen <64) //需要增加数据
  {
   if(dataBlockLen<56)//当前数据是最后若干个，而且不需要增加一次变换
   {
    //添加 1 和 0
    buffer[dataBlockLen]=0x80;
    //添加长度
    buffer[56]=(UI8)(len&0xff);
    len>>=8;
    buffer[57]=(UI8)(len&0xff);
    len>>=8;
    buffer[58]=(UI8)(len&0xff);
    len>>=8;
    buffer[59]=(UI8)(len&0xff);
    //变换
    FirstTransform((UI32*)buffer);
    CoreTransform();
   }
   else if(dataBlockLen>=56)
   {
    //添加 1 和 0
    buffer[dataBlockLen]=0x80;
    //变换
    FirstTransform((UI32*)buffer);
    CoreTransform();
    //添加长度
    memset(buffer,0,64);
    buffer[56]=(UI8)(len&0xff);
    len>>=8;
    buffer[57]=(UI8)(len&0xff);
    len>>=8;
    buffer[58]=(UI8)(len&0xff);
    len>>=8;
    buffer[59]=(UI8)(len&0xff);
    //变换
    FirstTransform((UI32*)buffer);
    CoreTransform();
   }
  }
  else if(dataBlockLen == 64)
  {
   //变换
   FirstTransform((UI32*)buffer);
   CoreTransform();
  }
 }

 //把 64 字节的原始数据 data 进行初步转化到 m_data 中去
 void FirstTransform(UI32 *data)
 {
  memcpy(X,data,64);
 }

 //核心变换
 void CoreTransform()
 {
  UI32 a1, b1, c1, d1, a2, b2, c2, d2;
  a1 = a2 = MDbuf[0];
  b1 = b2 = MDbuf[1];
  c1 = c2 = MDbuf[2];
  d1 = d2 = MDbuf[3];
  md1_Subround(md_F, a1, b1, c1, d1, X[ 0], 11, md_k0);
  md1_Subround(md_F, d1, a1, b1, c1, X[ 1], 14, md_k0);
  md1_Subround(md_F, c1, d1, a1, b1, X[ 2], 15, md_k0);
  md1_Subround(md_F, b1, c1, d1, a1, X[ 3], 12, md_k0);
  md1_Subround(md_F, a1, b1, c1, d1, X[ 4],  5, md_k0);
  md1_Subround(md_F, d1, a1, b1, c1, X[ 5],  8, md_k0);
  md1_Subround(md_F, c1, d1, a1, b1, X[ 6],  7, md_k0);
  md1_Subround(md_F, b1, c1, d1, a1, X[ 7],  9, md_k0);
  md1_Subround(md_F, a1, b1, c1, d1, X[ 8], 11, md_k0);
  md1_Subround(md_F, d1, a1, b1, c1, X[ 9], 13, md_k0);
  md1_Subround(md_F, c1, d1, a1, b1, X[10], 14, md_k0);
  md1_Subround(md_F, b1, c1, d1, a1, X[11], 15, md_k0);
  md1_Subround(md_F, a1, b1, c1, d1, X[12],  6, md_k0);
  md1_Subround(md_F, d1, a1, b1, c1, X[13],  7, md_k0);
  md1_Subround(md_F, c1, d1, a1, b1, X[14],  9, md_k0);
  md1_Subround(md_F, b1, c1, d1, a1, X[15],  8, md_k0);
  md1_Subround(md_G, a1, b1, c1, d1, X[ 7],  7, md_k1);
  md1_Subround(md_G, d1, a1, b1, c1, X[ 4],  6, md_k1);
  md1_Subround(md_G, c1, d1, a1, b1, X[13],  8, md_k1);
  md1_Subround(md_G, b1, c1, d1, a1, X[ 1], 13, md_k1);
  md1_Subround(md_G, a1, b1, c1, d1, X[10], 11, md_k1);
  md1_Subround(md_G, d1, a1, b1, c1, X[ 6],  9, md_k1);
  md1_Subround(md_G, c1, d1, a1, b1, X[15],  7, md_k1);
  md1_Subround(md_G, b1, c1, d1, a1, X[ 3], 15, md_k1);
  md1_Subround(md_G, a1, b1, c1, d1, X[12],  7, md_k1);
  md1_Subround(md_G, d1, a1, b1, c1, X[ 0], 12, md_k1);
  md1_Subround(md_G, c1, d1, a1, b1, X[ 9], 15, md_k1);
  md1_Subround(md_G, b1, c1, d1, a1, X[ 5],  9, md_k1);
  md1_Subround(md_G, a1, b1, c1, d1, X[ 2], 11, md_k1);
  md1_Subround(md_G, d1, a1, b1, c1, X[14],  7, md_k1);
  md1_Subround(md_G, c1, d1, a1, b1, X[11], 13, md_k1);
  md1_Subround(md_G, b1, c1, d1, a1, X[ 8], 12, md_k1);
  md1_Subround(md_H, a1, b1, c1, d1, X[ 3], 11, md_k2);
  md1_Subround(md_H, d1, a1, b1, c1, X[10], 13, md_k2);
  md1_Subround(md_H, c1, d1, a1, b1, X[14],  6, md_k2);
  md1_Subround(md_H, b1, c1, d1, a1, X[ 4],  7, md_k2);
  md1_Subround(md_H, a1, b1, c1, d1, X[ 9], 14, md_k2);
  md1_Subround(md_H, d1, a1, b1, c1, X[15],  9, md_k2);
  md1_Subround(md_H, c1, d1, a1, b1, X[ 8], 13, md_k2);
  md1_Subround(md_H, b1, c1, d1, a1, X[ 1], 15, md_k2);
  md1_Subround(md_H, a1, b1, c1, d1, X[ 2], 14, md_k2);
  md1_Subround(md_H, d1, a1, b1, c1, X[ 7],  8, md_k2);
  md1_Subround(md_H, c1, d1, a1, b1, X[ 0], 13, md_k2);
  md1_Subround(md_H, b1, c1, d1, a1, X[ 6],  6, md_k2);
  md1_Subround(md_H, a1, b1, c1, d1, X[13],  5, md_k2);
  md1_Subround(md_H, d1, a1, b1, c1, X[11], 12, md_k2);
  md1_Subround(md_H, c1, d1, a1, b1, X[ 5],  7, md_k2);
  md1_Subround(md_H, b1, c1, d1, a1, X[12],  5, md_k2);
  md1_Subround(md_I, a1, b1, c1, d1, X[ 1], 11, md_k3);
  md1_Subround(md_I, d1, a1, b1, c1, X[ 9], 12, md_k3);
  md1_Subround(md_I, c1, d1, a1, b1, X[11], 14, md_k3);
  md1_Subround(md_I, b1, c1, d1, a1, X[10], 15, md_k3);
```



```cpp
  md1_Subround(md_I, a1, b1, c1, d1, X[ 0], 14, md_k3);
  md1_Subround(md_I, d1, a1, b1, c1, X[ 8], 15, md_k3);
  md1_Subround(md_I, c1, d1, a1, b1, X[12],  9, md_k3);
  md1_Subround(md_I, b1, c1, d1, a1, X[ 4],  8, md_k3);
  md1_Subround(md_I, a1, b1, c1, d1, X[13],  9, md_k3);
  md1_Subround(md_I, d1, a1, b1, c1, X[ 3], 14, md_k3);
  md1_Subround(md_I, c1, d1, a1, b1, X[ 7],  5, md_k3);
  md1_Subround(md_I, b1, c1, d1, a1, X[15],  6, md_k3);
  md1_Subround(md_I, a1, b1, c1, d1, X[14],  8, md_k3);
  md1_Subround(md_I, d1, a1, b1, c1, X[ 5],  6, md_k3);
  md1_Subround(md_I, c1, d1, a1, b1, X[ 6],  5, md_k3);
  md1_Subround(md_I, b1, c1, d1, a1, X[ 2], 12, md_k3);
  md1_Subround(md_I, a2, b2, c2, d2, X[ 5],  8, md_k5);
  md1_Subround(md_I, d2, a2, b2, c2, X[14],  9, md_k5);
  md1_Subround(md_I, c2, d2, a2, b2, X[ 7],  9, md_k5);
  md1_Subround(md_I, b2, c2, d2, a2, X[ 0], 11, md_k5);
  md1_Subround(md_I, a2, b2, c2, d2, X[ 9], 13, md_k5);
  md1_Subround(md_I, d2, a2, b2, c2, X[ 2], 15, md_k5);
  md1_Subround(md_I, c2, d2, a2, b2, X[11], 15, md_k5);
  md1_Subround(md_I, b2, c2, d2, a2, X[ 4],  5, md_k5);
  md1_Subround(md_I, a2, b2, c2, d2, X[13],  7, md_k5);
  md1_Subround(md_I, d2, a2, b2, c2, X[ 6],  7, md_k5);
  md1_Subround(md_I, c2, d2, a2, b2, X[15],  8, md_k5);
  md1_Subround(md_I, b2, c2, d2, a2, X[ 8], 11, md_k5);
  md1_Subround(md_I, a2, b2, c2, d2, X[ 1], 14, md_k5);
  md1_Subround(md_I, d2, a2, b2, c2, X[10], 14, md_k5);
  md1_Subround(md_I, c2, d2, a2, b2, X[ 3], 12, md_k5);
  md1_Subround(md_I, b2, c2, d2, a2, X[12],  6, md_k5);
  md1_Subround(md_H, a2, b2, c2, d2, X[ 6],  9, md_k6);
  md1_Subround(md_H, d2, a2, b2, c2, X[11], 13, md_k6);
  md1_Subround(md_H, c2, d2, a2, b2, X[ 3], 15, md_k6);
  md1_Subround(md_H, b2, c2, d2, a2, X[ 7],  7, md_k6);
  md1_Subround(md_H, a2, b2, c2, d2, X[ 0], 12, md_k6);
  md1_Subround(md_H, d2, a2, b2, c2, X[13],  8, md_k6);
  md1_Subround(md_H, c2, d2, a2, b2, X[ 5],  9, md_k6);
  md1_Subround(md_H, b2, c2, d2, a2, X[10], 11, md_k6);
  md1_Subround(md_H, a2, b2, c2, d2, X[14],  7, md_k6);
  md1_Subround(md_H, d2, a2, b2, c2, X[15],  7, md_k6);
  md1_Subround(md_H, c2, d2, a2, b2, X[ 8], 12, md_k6);
  md1_Subround(md_H, b2, c2, d2, a2, X[12],  7, md_k6);
  md1_Subround(md_H, a2, b2, c2, d2, X[ 4],  6, md_k6);
  md1_Subround(md_H, d2, a2, b2, c2, X[ 9], 15, md_k6);
  md1_Subround(md_H, c2, d2, a2, b2, X[ 1], 13, md_k6);
  md1_Subround(md_H, b2, c2, d2, a2, X[ 2], 11, md_k6);
  md1_Subround(md_G, a2, b2, c2, d2, X[15],  9, md_k7);
  md1_Subround(md_G, d2, a2, b2, c2, X[ 5],  7, md_k7);
  md1_Subround(md_G, c2, d2, a2, b2, X[ 1], 15, md_k7);
  md1_Subround(md_G, b2, c2, d2, a2, X[ 3], 11, md_k7);
  md1_Subround(md_G, a2, b2, c2, d2, X[ 7],  8, md_k7);
  md1_Subround(md_G, d2, a2, b2, c2, X[14],  6, md_k7);
  md1_Subround(md_G, c2, d2, a2, b2, X[ 6],  6, md_k7);
  md1_Subround(md_G, b2, c2, d2, a2, X[ 9], 14, md_k7);
  md1_Subround(md_G, a2, b2, c2, d2, X[11], 12, md_k7);
  md1_Subround(md_G, d2, a2, b2, c2, X[ 8], 13, md_k7);
  md1_Subround(md_G, c2, d2, a2, b2, X[12],  5, md_k7);
  md1_Subround(md_G, b2, c2, d2, a2, X[ 2], 14, md_k7);
  md1_Subround(md_G, a2, b2, c2, d2, X[10], 13, md_k7);
  md1_Subround(md_G, d2, a2, b2, c2, X[ 0], 13, md_k7);
  md1_Subround(md_G, c2, d2, a2, b2, X[ 4],  7, md_k7);
  md1_Subround(md_G, b2, c2, d2, a2, X[13],  5, md_k7);
  md1_Subround(md_F, a2, b2, c2, d2, X[ 8], 15, md_k9);
  md1_Subround(md_F, d2, a2, b2, c2, X[ 6],  5, md_k9);
  md1_Subround(md_F, c2, d2, a2, b2, X[ 4],  8, md_k9);
  md1_Subround(md_F, b2, c2, d2, a2, X[ 1], 11, md_k9);
  md1_Subround(md_F, a2, b2, c2, d2, X[ 3], 14, md_k9);
  md1_Subround(md_F, d2, a2, b2, c2, X[11], 14, md_k9);
  md1_Subround(md_F, c2, d2, a2, b2, X[15],  6, md_k9);
  md1_Subround(md_F, b2, c2, d2, a2, X[ 0], 14, md_k9);
  md1_Subround(md_F, a2, b2, c2, d2, X[ 5],  6, md_k9);
  md1_Subround(md_F, d2, a2, b2, c2, X[12],  9, md_k9);
  md1_Subround(md_F, c2, d2, a2, b2, X[ 2], 12, md_k9);
  md1_Subround(md_F, b2, c2, d2, a2, X[13],  9, md_k9);
  md1_Subround(md_F, a2, b2, c2, d2, X[ 9], 12, md_k9);
  md1_Subround(md_F, d2, a2, b2, c2, X[ 7],  5, md_k9);
  md1_Subround(md_F, c2, d2, a2, b2, X[10], 15, md_k9);
  md1_Subround(md_F, b2, c2, d2, a2, X[14],  8, md_k9);
  c1        = MDbuf[1] + c1 + d2;
  MDbuf[1] = MDbuf[2] + d1 + a2;
  MDbuf[2] = MDbuf[3] + a1 + b2;
  MDbuf[3] = MDbuf[0] + b1 + c2;
  MDbuf[0] = c1;
}

private:
  UI32 MDbuf[4];  //保存有 ripemd 输出值
  UI32 X[16];    //保存有每步变换时对输入的初始化转化值
};

//...end ripemd128
```

这里是附件：

附件包含有 2 个文件：

`define.h` 定义了数据结构和操作

`ripemd.h` 定义了所有 `ripemd` 类

附件: `ripemd.rar`

明天再来继续发布，敬请等待。

### SHA 系列 Hash 算法

`SHA` 系列 `Hash` 算法是由美国国家安全局（US National Security Agency）设计的。完整的说，`SHA` 共有 6 个标准。分别是：`SHA-0`(1992 年)、`SHA-1`(1995 年)、`SHA224`(2002 年)、`SHA256`（2000 年）、`SHA-384`（2000 年）和 `SHA-512`（2000 年）。关于 `SHA` 算法最权威的文档当属 `FIPS P180` 系列，最新的是 2002 年修订的 `FIPS P180-2`。参考地址为：`http://csrc.nist.gov/publications/`。

`SHA` 的具体描述大家自己去下载文档来看就行了。`SHA` 与普通 `Hash` 算法最大的不同在于它不是专门为 `Intel cpu` 设计的，所以，其整数的表示方式和 `Intel cpu` 目前的低字节在前的方法不一样。以下只列出了 `SHA-1` 算法的代码。

```cpp
#ifndef Goodzzp_SHA_05_10_12_16
#define Goodzzp_SHA_05_10_12_16

//sha-1,224,256,384,512
//program by Goodzzp
//2005.10.13
//Notice:
// this sha.h file referred crypto++5.21 and SHA standard(Federal Information
// Processing Standards Publication 180-2 with SHA224 notice) 2002 Auguest 1
#include "define.h"

//....beigin sha-1
#define sha_blk0(i) (W[i] = m_data[i])
#define sha_blk1(i) (W[i&15] = rotlFixed(W[(i+13)&15]^W[(i+8)&15]^W[(i+2)&15]^W[i&15],1))
#define sha_f1(x,y,z) (z^(x&(y^z)))
#define sha_f2(x,y,z) (x^y^z)
#define sha_f3(x,y,z) ((x&y)|(z&(x|y)))
#define sha_f4(x,y,z) (x^y^z)

/* (R0+R1), R2, R3, R4 are the different operations used in SHA1 */
#define sha_R0(v,w,x,y,z,i) z+=sha_f1(w,x,y)+sha_blk0(i)+0x5A827999+rotlFixed(v,5);w=rotlFixed(w,30);
#define sha_R1(v,w,x,y,z,i) z+=sha_f1(w,x,y)+sha_blk1(i)+0x5A827999+rotlFixed(v,5);w=rotlFixed(w,30);
#define sha_R2(v,w,x,y,z,i) z+=sha_f2(w,x,y)+sha_blk1(i)+0x6ED9EBA1+rotlFixed(v,5);w=rotlFixed(w,30);
#define sha_R3(v,w,x,y,z,i) z+=sha_f3(w,x,y)+sha_blk1(i)+0x8F1BBCDC+rotlFixed(v,5);w=rotlFixed(w,30);
#define sha_R4(v,w,x,y,z,i) z+=sha_f4(w,x,y)+sha_blk1(i)+0xCA62C1D6+rotlFixed(v,5);w=rotlFixed(w,30);

//sha1 algorithm
class SHA1
{
public:
  //输出结果的长度（字节）
  static UI32 OutDataLength()
  {
    return 4*5;
  }
  //SHA-1 变换函数
  //out:输出，长度为 20，要求事先已经分配了内存
  //in:输入
  //length:输入值的长度
  void Hash(UI8 *out,const UI8 *in,UI32 length)
  {
    UI32 i = length>>6,j=(length&0x3f),k;
    //对数据 64 64 个字节的计算
    for(k=0;k<i;k++)
    {
      StepTransform((UI8 *)(in + k * 64),64,length);
    }
    //最后计算尾数
    StepTransform((UI8 *)(in + 64 * i),j,length);
    //拷贝输出
    for(i=0;i<4;i++)  //反转
    {
      m_state[i] = ((m_state[i]&0xff) << 24) | ((m_state[i]&0xff00) << 8) |
        ((m_state[i]&0xff0000) >> 8) | ((m_state[i]&0xff000000)>>24);
    }
    memcpy(out,m_state,4*5);
    //恢复 m_state 值
    m_state[0] = 0x67452301L;
    m_state[1] = 0xEFCDAB89L;
    m_state[2] = 0x98BADCFEL;
    m_state[3] = 0x10325476L;
    m_state[4] = 0xC3D2E1F0L;
  }
  SHA1()  //initialize data
```



```cpp
  {
    m_state[0] = 0x67452301L;
    m_state[1] = 0xEFCDAB89L;
    m_state[2] = 0x98BADCFEL;
    m_state[3] = 0x10325476L;
    m_state[4] = 0xC3D2E1F0L;
  }

private:
  //每步的变换函数
  //输入:
  //    data:      要处理的数据块（不大于 64 字节）
  //    dataBlockLen:  数据块的长度
  //    dataTotalLen:  要处理的所有数据块的总长度
  //输出结果保存在 m_state 里面
  void StepTransform(UI8 *data, UI32 dataBlockLen, UI32 dataTotalLen)
  {
    UI8 buffer[64];
    UI32 len=dataTotalLen*8;

    memset(buffer,0,64);//清空数据为 0
    memcpy(buffer,data,dataBlockLen);//拷贝数据到缓冲
    if(dataBlockLen <64)  //需要增加数据
    {
      if(dataBlockLen<56)//当前数据是最后若干个，而且不需要增加一次变换
      {
        //添加 1 和 0
        buffer[dataBlockLen]=0x80;
        //添加长度
        buffer[63]=(UI8)(len&0xff);
        len>>=8;
        buffer[62]=(UI8)(len&0xff);
        len>>=8;
        buffer[61]=(UI8)(len&0xff);
        len>>=8;
        buffer[60]=(UI8)(len&0xff);
        //变换
        FirstTransform((UI32*)buffer);
        CoreTransform();
      }
      else if(dataBlockLen>=56)
      {
        //添加 1 和 0
        buffer[dataBlockLen]=0x80;
        //变换
        FirstTransform((UI32*)buffer);
        CoreTransform();
        //添加长度
        memset(buffer,0,64);
        buffer[63]=(UI8)(len&0xff);
        len>>=8;
        buffer[62]=(UI8)(len&0xff);
        len>>=8;
        buffer[61]=(UI8)(len&0xff);
        len>>=8;
        buffer[60]=(UI8)(len&0xff);
        //变换
        FirstTransform((UI32*)buffer);
        CoreTransform();
      }
    }
    else if(dataBlockLen == 64)
    {
      //变换
      FirstTransform((UI32*)buffer);
      CoreTransform();
    }
  }

  //把 64 字节的原始数据 data 进行初步转化到 m_data 中去
  void FirstTransform(UI32 *data)
  {
    //拷贝
    memcpy(m_data,data,64);
    //反转，因为 sha-1 是低位在后面的存储方法，所以，需要把 abcd efgh 变成 dcba hgfe。
    for(UI32 i=0;i<16;i++)
    {
      m_data[i] = ((m_data[i]&0xff) << 24) | ((m_data[i]&0xff00) << 8) |
        ((m_data[i]&0xff0000) >> 8) | ((m_data[i]&0xff000000)>>24);
    }
    //根据这 16 个生成其它 64 个 UI32
    for(UI32 j=16;j<80;j++)
    {
      m_data[j] = rotlFixed(m_data[j-3]^m_data[j-8]^m_data[j-14]^m_data[j-16],1);
    }
  }

  //核心变换
  void CoreTransform()
  {
    UI32 W[16];
    UI32 a = m_state[0];
    UI32 b = m_state[1];
    UI32 c = m_state[2];
    UI32 d = m_state[3];
    UI32 e = m_state[4];

    /* 4 rounds of 20 operations each. Loop unrolled. */
    sha_R0(a,b,c,d,e, 0); sha_R0(e,a,b,c,d, 1); sha_R0(d,e,a,b,c, 2); sha_R0(c,d,e,a,b, 3);
    sha_R0(b,c,d,e,a, 4); sha_R0(a,b,c,d,e, 5); sha_R0(e,a,b,c,d, 6); sha_R0(d,e,a,b,c, 7);
    sha_R0(c,d,e,a,b, 8); sha_R0(b,c,d,e,a, 9); sha_R0(a,b,c,d,e,10); sha_R0(e,a,b,c,d,11);
    sha_R0(d,e,a,b,c,12); sha_R0(c,d,e,a,b,13); sha_R0(b,c,d,e,a,14); sha_R0(a,b,c,d,e,15);

    sha_R1(e,a,b,c,d,16); sha_R1(d,e,a,b,c,17); sha_R1(c,d,e,a,b,18); sha_R1(b,c,d,e,a,19);

    sha_R2(a,b,c,d,e,20); sha_R2(e,a,b,c,d,21); sha_R2(d,e,a,b,c,22); sha_R2(c,d,e,a,b,23);
    sha_R2(b,c,d,e,a,24); sha_R2(a,b,c,d,e,25); sha_R2(e,a,b,c,d,26); sha_R2(d,e,a,b,c,27);
    sha_R2(c,d,e,a,b,28); sha_R2(b,c,d,e,a,29); sha_R2(a,b,c,d,e,30); sha_R2(e,a,b,c,d,31);
    sha_R2(d,e,a,b,c,32); sha_R2(c,d,e,a,b,33); sha_R2(b,c,d,e,a,34); sha_R2(a,b,c,d,e,35);
    sha_R2(e,a,b,c,d,36); sha_R2(d,e,a,b,c,37); sha_R2(c,d,e,a,b,38); sha_R2(b,c,d,e,a,39);

    sha_R3(a,b,c,d,e,40); sha_R3(e,a,b,c,d,41); sha_R3(d,e,a,b,c,42); sha_R3(c,d,e,a,b,43);
    sha_R3(b,c,d,e,a,44); sha_R3(a,b,c,d,e,45); sha_R3(e,a,b,c,d,46); sha_R3(d,e,a,b,c,47);
    sha_R3(c,d,e,a,b,48); sha_R3(b,c,d,e,a,49); sha_R3(a,b,c,d,e,50); sha_R3(e,a,b,c,d,51);
    sha_R3(d,e,a,b,c,52); sha_R3(c,d,e,a,b,53); sha_R3(b,c,d,e,a,54); sha_R3(a,b,c,d,e,55);
    sha_R3(e,a,b,c,d,56); sha_R3(d,e,a,b,c,57); sha_R3(c,d,e,a,b,58); sha_R3(b,c,d,e,a,59);

    sha_R4(a,b,c,d,e,60); sha_R4(e,a,b,c,d,61); sha_R4(d,e,a,b,c,62); sha_R4(c,d,e,a,b,63);
    sha_R4(b,c,d,e,a,64); sha_R4(a,b,c,d,e,65); sha_R4(e,a,b,c,d,66); sha_R4(d,e,a,b,c,67);
    sha_R4(c,d,e,a,b,68); sha_R4(b,c,d,e,a,69); sha_R4(a,b,c,d,e,70); sha_R4(e,a,b,c,d,71);
    sha_R4(d,e,a,b,c,72); sha_R4(c,d,e,a,b,73); sha_R4(b,c,d,e,a,74); sha_R4(a,b,c,d,e,75);
    sha_R4(e,a,b,c,d,76); sha_R4(d,e,a,b,c,77); sha_R4(c,d,e,a,b,78); sha_R4(b,c,d,e,a,79);

    /* Add the working vars back into context.m_state[] */
    m_state[0] += a;
    m_state[1] += b;
    m_state[2] += c;
    m_state[3] += d;
    m_state[4] += e;
  }

private:
  UI32 m_state[5];  //保存有 sha-1 输出值
  UI32 m_data[80];  //保存有每步变换时对输入的初始化转化值
};

typedef SHA1 SHA;

//...end sha-1
```

还有，需要申明的是，作者给出的任何代码都经过基本测试。并不存在剽窃任何人代码的事情。作者的目的是为了做出中国第一个密码学算法库。打算包含所有已经成为标准或者十分流行的散列、加密、签名、检验及身份认证算法。目前已经完成了`Hash`算法部分。还有`AES`，`RSA`等。作者主要的算法和代码参考源是各种标准文件以及`crypto++5.21`代码库。这个库也是国外的代码库，目前由一个中国人维护。关于`SHA`方面，`crpto++5.21`没有包含新的`SHA-224`，因为这个库比较老了。本文作者实现了目前所有的`SHA`标准。因为大家对这些东西的关注太少了。我决定不公开所有代码了。需要的可以给我发 Email。

`4dd466d0e1e0cc123afbe504da2462ca`
