
# Call(2)分析结束

到此 我们已经可以分析出这个CrackMe得基本算法了 不妨列出个等式方便分析：

CrackMe实际上是对比`MD5`的`Hash`运算结果与`RSA`算法中的密文

若相等则认为注册成功,根据上面的分析 `RSA`的加密

公式为`c = (m ^ e) mod n` 换中说法就是比较`MD5`的`Hash`运算结果是否等于

`(M^E) MOD N`

所以： `(Registeration Code ^ E ) MOD N = MD5(Name + Organization)`

即： `Registeration Code = [ MD5( Name + Organization ) ^ D ] MOD N`

因为： `MD5(娃娃[CCG])=3E32771176B5CE99ED7C920DD28EADD6`

得出等式：`Registeration Code`
= `[ 3E32771176B5CE99ED7C920DD28EADD6 ^ 1E2D9B52ADCBC20DCCDE3C721AA740E83 ] MOD`
`24DFDA27FA14D3F27DDF62CEA5D2381F9`

这时就需要用到`BigInt Calculator`了 设 `X＝82674165571952145171965745053779799510`
`Y＝641818296898666730290105568133836443267` `Z＝784232236484777663526392470596418241017`

然后计算 `X^Y%Z`

得出结果为：`362487173057627226939435819421597555030`

在其转换成十六进制得到：`110B47D7782077078380A409E3295D956`

字符串`110B47D7782077078380A409E3295D956`即为有效Key 分析完成～～～～～～～

给出两组有效Key

方便各位加深理解

Name： 娃娃

Organization： `[CCG]`

Registeration Code： `110B47D7782077078380A409E3295D956`

Name： `NYDoll`

Organization： `CHiNA CrACKiNG GrOUp`

Registeration

Code： `188B6DE7B64A99CA1DFE50084A6372ADE`

# 注册机(VC源代码)

```c
/*
 Lockless CrackMe #4 注册机  娃娃/[CCG]制作

RSA算法的源代码引用自tE! 的 PowerStrip keygenerator

 MD5算法引用 Colin Plumb 和 John Walker

编写的模板

 程序使用 freelip 的动态连接库

 */

#include <windows.h>

#include <lip.h>

#include <memory.h>
#define DLG_MAIN
                      111
#define BT_GENERATE
    1008
#define EDIT_NAME
          1009
#define EDIT_CODE
                1010
#define EDIT_ORG
            1011
typedef unsigned long uint32;
struct MD5Context {
    uint32 buf[4];
         uint32 bits[2];
         unsigned char in[64];
};
void
MD5Transform(uint32*, uint32*);
void byteReverse(buf, longs)
  unsigned char *buf; unsigned longs;
{
     uint32 t;
     do {
     t = (uint32) ((unsigned) buf[3]
<< 8 | buf[2]) << 16 |
         ((unsigned)
buf[1] << 8 | buf[0]);
     *(uint32 *) buf = t;
    buf += 4;
     } while (--longs);
}
void MD5Init(ctx)
     struct MD5Context *ctx;
{
    ctx->buf[0] = 0x67452301;
     ctx->buf[1] = 0xefcdab89;
     ctx->buf[2] = 0x98badcfe;
     ctx->buf[3] =
0x10325476;
     ctx->bits[0] = 0;
     ctx->bits[1]
= 0;
}
void MD5Update(ctx, buf, len)
     struct
MD5Context *ctx; unsigned char *buf; unsigned len;
{
     uint32
t;
     t = ctx->bits[0];
     if ((ctx->bits[0]
= t + ((uint32) len << 3)) < t)
     ctx->bits[1]++;
    /* Carry from low to high */
     ctx->bits[1]
+= len >> 29;
     t = (t >> 3) & 0x3f;    /*
Bytes already in shsInfo->data */
     /* Handle any leading
odd-sized chunks */
     if (t) {
     unsigned
char *p = (unsigned char *) ctx->in + t;
     t = 64 -
t;
     if (len < t) {
memcpy(p, buf, len);
         return;
  }
     memcpy(p, buf, t);
     byteReverse(ctx->in,
16);
     MD5Transform(ctx->buf, (uint32 *) ctx->in);
    buf += t;
     len -= t;
}
     /* Process data in 64-byte chunks */
while (len >= 64) {
     memcpy(ctx->in, buf, 64);
  byteReverse(ctx->in, 16);
     MD5Transform(ctx->buf,
(uint32 *) ctx->in);
     buf += 64;
     len
-= 64;
     }
     /* Handle any remaining bytes
of data. */
     memcpy(ctx->in, buf, len);
}

void MD5Final(digest, ctx)
     unsigned char digest[16]; struct
MD5Context *ctx;
{
     unsigned count;
unsigned char *p;
     count = (ctx->bits[0] >> 3) & 0x3F;
     p = ctx->in + count;
     *p++ = 0x80;
  count = 64 - 1 - count;
     /* Pad out to 56 mod 64
*/
     if (count < 8) {
     memset(p, 0,
count);
     byteReverse(ctx->in, 16);
     MD5Transform(ctx->buf,
(uint32 *) ctx->in);
     memset(ctx->in, 0, 56);
  } else {
     memset(p, 0, count - 8);
}
     byteReverse(ctx->in, 14);
     ((uint32 *)
ctx->in)[14] = ctx->bits[0];
     ((uint32 *) ctx->in)[15] = ctx->bits[1];
     MD5Transform(ctx->buf, (uint32 *) ctx->in);
  byteReverse((unsigned char *) ctx->buf, 4);
     memcpy(digest,
ctx->buf, 16);
     memset(ctx, 0, sizeof(ctx));
}

/* The four core functions - F1 is optimized somewhat
*/
/* #define F1(x, y, z) (x & y | ~x & z) */
#define F1(x,
y, z) (z ^ (x & (y ^ z)))
#define F2(x, y, z) F1(z, x, y)
#define
F3(x, y, z) (x ^ y ^ z)
#define F4(x, y, z) (y ^ (x | ~z))
/* This
is the central step in the MD5 algorithm. */
#define MD5STEP(f, w, x, y,
z, data, s) \
     ( w += f(x, y, z) + data,  w = w<<s
| w>>(32-s),  w += x )
void MD5Transform(buf, in)
uint32 buf[4]; uint32 in[16];
{
     register uint32 a, b,
c, d;
     a = buf[0];
     b = buf[1];
    c = buf[2];
     d = buf[3];
MD5STEP(F1, a, b, c, d, in[0] + 0xd76aa478, 7);
     MD5STEP(F1,
d, a, b, c, in[1] + 0xe8c7b756, 12);
     MD5STEP(F1, c, d, a, b,
in[2] + 0x242070db, 17);
     MD5STEP(F1, b, c, d, a, in[3] + 0xc1bdceee,
22);
     MD5STEP(F1, a, b, c, d, in[4] + 0xf57c0faf, 7);
  MD5STEP(F1, d, a, b, c, in[5] + 0x4787c62a, 12);
     MD5STEP(F1,
c, d, a, b, in[6] + 0xa8304613, 17);
     MD5STEP(F1, b, c, d, a,
in[7] + 0xfd469501, 22);
     MD5STEP(F1, a, b, c, d, in[8] + 0x698098d8,
7);
     MD5STEP(F1, d, a, b, c, in[9] + 0x8b44f7af, 12);
  MD5STEP(F1, c, d, a, b, in[10] + 0xffff5bb1, 17);
     MD5STEP(F1,
b, c, d, a, in[11] + 0x895cd7be, 22);
     MD5STEP(F1, a, b, c,
d, in[12] + 0x6b901122, 7);
     MD5STEP(F1, d, a, b, c, in[13]
+ 0xfd987193, 12);
     MD5STEP(F1, c, d, a, b, in[14] + 0xa679438e,
17);
     MD5STEP(F1, b, c, d, a, in[15] + 0x49b40821, 22);
     MD5STEP(F2, a, b, c, d, in[1] + 0xf61e2562, 5);
  MD5STEP(F2, d, a, b, c, in[6] + 0xc040b340, 9);
     MD5STEP(F2,
c, d, a, b, in[11] + 0x265e5a51, 14);
     MD5STEP(F2, b, c, d,
a, in[0] + 0xe9b6c7aa, 20);
     MD5STEP(F2, a, b, c, d, in[5] +
0xd62f105d, 5);
     MD5STEP(F2, d, a, b, c, in[10] + 0x02441453,
9);
     MD5STEP(F2, c, d, a, b, in[15] + 0xd8a1e681, 14);
    MD5STEP(F2, b, c, d, a, in[4] + 0xe7d3fbc8, 20);
MD5STEP(F2, a, b, c, d, in[9] + 0x21e1cde6, 5);
     MD5STEP(F2,
d, a, b, c, in[14] + 0xc33707d6, 9);
     MD5STEP(F2, c, d, a, b,
in[3] + 0xf4d50d87, 14);
     MD5STEP(F2, b, c, d, a, in[8] + 0x455a14ed,
20);
     MD5STEP(F2, a, b, c, d, in[13] + 0xa9e3e905, 5);
    MD5STEP(F2, d, a, b, c, in[2] + 0xfcefa3f8, 9);
MD5STEP(F2, c, d, a, b, in[7] + 0x676f02d9, 14);
     MD5STEP(F2,
b, c, d, a, in[12] + 0x8d2a4c8a, 20);
     MD5STEP(F3, a, b,
c, d, in[5] + 0xfffa3942, 4);
     MD5STEP(F3, d, a, b, c, in[8]
+ 0x8771f681, 11);
     MD5STEP(F3, c, d, a, b, in[11] + 0x6d9d6122,
16);
     MD5STEP(F3, b, c, d, a, in[14] + 0xfde5380c, 23);
    MD5STEP(F3, a, b, c, d, in[1] + 0xa4beea44, 4);
MD5STEP(F3, d, a, b, c, in[4] + 0x4bdecfa9, 11);
     MD5STEP(F3,
c, d, a, b, in[7] + 0xf6bb4b60, 16);
     MD5STEP(F3, b, c, d, a,
in[10] + 0xbebfbc70, 23);
     MD5STEP(F3, a, b, c, d, in[13] +
0x289b7ec6, 4);
     MD5STEP(F3, d, a, b, c, in[0] + 0xeaa127fa,
11);
     MD5STEP(F3, c, d, a, b, in[3] + 0xd4ef3085, 16);
    MD5STEP(F3, b, c, d, a, in[6] + 0x04881d05, 23);
MD5STEP(F3, a, b, c, d, in[9] + 0xd9d4d039, 4);
     MD5STEP(F3,
d, a, b, c, in[12] + 0xe6db99e5, 11);
     MD5STEP(F3, c, d, a,
b, in[15] + 0x1fa27cf8, 16);
     MD5STEP(F3, b, c, d, a, in[2]
+ 0xc4ac5665, 23);
     MD5STEP(F4, a, b, c, d, in[0] + 0xf4292244,
6);
     MD5STEP(F4, d, a, b, c, in[7] + 0x432aff97, 10);
  MD5STEP(F4, c, d, a, b, in[14] + 0xab9423a7, 15);
     MD5STEP(F4,
b, c, d, a, in[5] + 0xfc93a039, 21);
     MD5STEP(F4, a, b, c, d,
in[12] + 0x655b59c3, 6);
     MD5STEP(F4, d, a, b, c, in[3] + 0x8f0ccc92,
10);
     MD5STEP(F4, c, d, a, b, in[10] + 0xffeff47d, 15);
```
