
```
GG (d, a, b, c, x( 2), S22, 0xfcefa3f8); /* 30 */
GG (c, d, a, b, x( 7), S23, 0x676f02d9); /* 31 */
GG (b, c, d, a, x(12), S24, 0x8d2a4c8a); /* 32 */

/* Round 3 */
HH (a, b, c, d, x( 5), S31, 0xfffa3942); /* 33 */
HH (d, a, b, c, x( 8), S32, 0x8771f681); /* 34 */
HH (c, d, a, b, x(11), S33, 0x6d9d6122); /* 35 */
HH (b, c, d, a, x(14), S34, 0xfde5380c); /* 36 */
HH (a, b, c, d, x( 1), S31, 0xa4beea44); /* 37 */
HH (d, a, b, c, x( 4), S32, 0x4bdecfa9); /* 38 */
HH (c, d, a, b, x( 7), S33, 0xf6bb4b60); /* 39 */
HH (b, c, d, a, x(10), S34, 0xbebfbc70); /* 40 */
HH (a, b, c, d, x(13), S31, 0x289b7ec6); /* 41 */
HH (d, a, b, c, x( 0), S32, 0xeaa127fa); /* 42 */
HH (c, d, a, b, x( 3), S33, 0xd4ef3085); /* 43 */
HH (b, c, d, a, x( 6), S34,  0x4881d05); /* 44 */
HH (a, b, c, d, x( 9), S31, 0xd9d4d039); /* 45 */
HH (d, a, b, c, x(12), S32, 0xe6db99e5); /* 46 */
HH (c, d, a, b, x(15), S33, 0x1fa27cf8); /* 47 */
HH (b, c, d, a, x( 2), S34, 0xc4ac5665); /* 48 */

/* Round 4 */
II (a, b, c, d, x( 0), S41, 0xf4292244); /* 49 */
II (d, a, b, c, x( 7), S42, 0x432aff97); /* 50 */
II (c, d, a, b, x(14), S43, 0xab9423a7); /* 51 */
II (b, c, d, a, x( 5), S44, 0xfc93a039); /* 52 */
II (a, b, c, d, x(12), S41, 0x655b59c3); /* 53 */
II (d, a, b, c, x( 3), S42, 0x8f0ccc92); /* 54 */
II (c, d, a, b, x(10), S43, 0xffeff47d); /* 55 */
II (b, c, d, a, x( 1), S44, 0x85845dd1); /* 56 */
II (a, b, c, d, x( 8), S41, 0x6fa87e4f); /* 57 */
II (d, a, b, c, x(15), S42, 0xfe2ce6e0); /* 58 */
II (c, d, a, b, x( 6), S43, 0xa3014314); /* 59 */
II (b, c, d, a, x(13), S44, 0x4e0811a1); /* 60 */
II (a, b, c, d, x( 4), S41, 0xf7537e82); /* 61 */
II (d, a, b, c, x(11), S42, 0xbd3af235); /* 62 */
II (c, d, a, b, x( 2), S43, 0x2ad7d2bb); /* 63 */
II (b, c, d, a, x( 9), S44, 0xeb86d391); /* 64 */

__asm {
    mov [x],0x67452301;
    mov [x+4],0xefcdab89;
    mov [x+8],0x98badcfe;
    mov [x+12],0x10325476;
    add [x],a;
    add [x+4],b;
    add [x+8],c;
    add [x+12],d;
}
}
__asm popad;
}
if(memcmp(MD5Result,x,16) == 0)  return true;
return false;
}
```

有一点很重要的就是，`vc` 好像不主动优化汇编代码。

速度上没有改观的主要原因可能是因为 `x` 数组的数据没有保存到寄存器中。

考虑到 `x[5-13]=0`，以及 `x[4]=x[14]=0x80`，如果把 `x[0-3]` 保存到 `mmx` 空间，我估计时间上肯定能好不少。

工作了半天，终于写出了基于 `mmx` 的代码。并且使用了一些技巧。比如，

`F(x,y,z) = (x and y) or ( not(x) and z)` 可以修改为：

`f(x,y,z) = ((y xor z) and x) xor z`

同时，每次比较 2 个密码。这点利用了 `mmx` 是 64 位的特性。不过，难以执信的是，代码速度的提速没有预计的那么明显。明天再探讨这个问题。今天困了。

代码如下：

```
//Md5 算法

typedef unsigned char UI8;

typedef char I8;

typedef unsigned long UI32;

//mm0,mm1,mm2,mm3 保存有 2 个 md5 变换的 a,b,c,d 值

//a0 和 a1 都保存在这里
#define a mm0

//b0,b1
#define b mm1

//c0,c1
#define c mm2

//d0,d1
#define d mm3

//m0,m1,m2,m3 是第一个数据
//m4,m5,m6,m7 是第二个数据

#define m2 eax
#define m6 ebx
#define m3 ecx
#define m7 edx

//m0,m4 保存在 mm4
//m1,m5 保存在 mm5

//把 ac(32)复制 2 个到 mm7(64)中去
#define Move(ac) \
  __asm mov esi,ac \
  __asm movd mm6,esi \
  __asm movd mm7,esi \
  __asm psll mm6,32 \
  __asm por mm7,mm6

//把 data(32)复制 2 个到 a0(64)中去
#define Move0(a0,data) \
  __asm mov esi,data \
  __asm movd a0,esi \
  __asm movd mm7,esi \
  __asm psll a0,32 \
  __asm por a0,mm7

#define F(x,y,z) \
  __asm movq mm6,y \
  __asm pxor mm6,z \
  __asm pand mm6,x \
  __asm pxor mm6,z

#define G(x,y,z) \
  __asm movq mm6,x \
  __asm pxor mm6,y \
  __asm pand mm6,z \
  __asm pxor mm6,y

#define H(x,y,z) \
  __asm movq mm6,x \
  __asm pxor mm6,y \
  __asm pxor mm6,z

#define I(x,y,z) \
__asm movd mm6,edi \
__asm movd mm7,edi \
__asm psllq mm6,32 \
__asm pxor mm6,mm7 \
__asm pxor mm6,z \
__asm por mm6,x \
__asm pxor mm6,y

//当 x 数组数据为 0 或者 0x80 时,有 x[4]=0x80,x[14]=0x80,x[5-13,15]=0;

//ac 为普通的 2 个 32 位组成
//s0 + s1 = 32,因为没有循环左移位
#define md5_FF0(a0,b0,c0,d0,s0,s1,ac) \
Move(ac) \
F(b0,c0,d0) \
__asm paddd a0,mm7 \
__asm paddd a0,mm6 \
__asm movq mm7,a0 __asm psrld a0,s0 __asm pslld mm7,s1 __asm por a0,mm7 \
__asm paddd a0,b0

//当 x 数组数据不为 0 时
//调用第一个数据
#define md5_FF1(a0,b0,c0,d0,s0,s1,ac) \
Move(ac) \
F(b0,c0,d0) \
__asm paddd a0,mm6 \
__asm paddd mm7,mm4 \
__asm paddd a0,mm7 \
__asm movq mm7,a0 __asm psrld a0,s0 __asm pslld mm7,s1 __asm por a0,mm7 \
__asm paddd a0,b0

//第二个数据
#define md5_FF2(a0,b0,c0,d0,s0,s1,ac) \
Move(ac) \
F(b0,c0,d0) \
__asm paddd mm7,mm5 \
__asm paddd a0,mm6 \
__asm paddd a0,mm7 \
__asm movq mm7,a0 __asm psrld a0,s0 __asm pslld mm7,s1 __asm por a0,mm7 \
__asm paddd a0,b0

//第三个数据
#define md5_FF3(a0,b0,c0,d0,s0,s1,ac) \
Move(ac) \
F(b0,c0,d0) \
__asm paddd mm7,mm6 \
__asm paddd a0,mm7 \
__asm movd mm6,eax \
__asm movd mm7,ebx \
__asm psll mm6,32 \
__asm por mm7,mm6 \
__asm paddd a0,mm7 \
__asm movq mm7,a0 __asm psrld a0,s0 __asm pslld mm7,s1 __asm por a0,mm7 \
__asm paddd a0,b0

//第四个数据
#define md5_FF4(a0,b0,c0,d0,s0,s1,ac) \
Move(ac) \
F(b0,c0,d0) \
__asm paddd mm7,mm6 \
__asm paddd a0,mm7 \
__asm movd mm6,ecx \
__asm movd mm7,edx \
__asm psll mm6,32 \
__asm por mm7,mm6 \
__asm paddd a0,mm7 \
__asm movq mm7,a0 __asm psrld a0,s0 __asm pslld mm7,s1 __asm por a0,mm7 \
__asm paddd a0,b0

//为 0
#define md5_GG0(a0,b0,c0,d0,s0,s1,ac) \
Move(ac) \
G(b0,c0,d0) \
__asm paddd a0,mm6 \
__asm paddd a0,mm7 \
__asm movq mm7,a0 __asm psrld a0,s0 __asm pslld mm7,s1 __asm por a0,mm7 \
__asm paddd a0,b0

//当 x 数组数据不为 0 时
#define md5_GG1(a0,b0,c0,d0,s0,s1,ac) \
Move(ac) \
G(b0,c0,d0) \
__asm paddd mm7,mm4 \
__asm paddd a0,mm6 \
__asm paddd a0,mm7 \
__asm movq mm7,a0 __asm psrld a0,s0 __asm pslld mm7,s1 __asm por a0,mm7 \
__asm paddd a0,b0

#define md5_GG2(a0,b0,c0,d0,s0,s1,ac) \
Move(ac) \
G(b0,c0,d0) \
__asm paddd mm7,mm5 \
__asm paddd a0,mm6 \
__asm paddd a0,mm7 \
__asm movq mm7,a0 __asm psrld a0,s0 __asm pslld mm7,s1 __asm por a0,mm7 \
__asm paddd a0,b0

#define md5_GG3(a0,b0,c0,d0,s0,s1,ac) \
Move(ac) \
G(b0,c0,d0) \
__asm paddd mm7,mm6 \
__asm paddd a0,mm7 \
__asm movd mm6,eax \
__asm movd mm7,ebx \
__asm psll mm6,32 \
__asm por mm7,mm6 \
__asm paddd a0,mm7 \
__asm movq mm7,a0 __asm psrld a0,s0 __asm pslld mm7,s1 __asm por a0,mm7 \
__asm paddd a0,b0

#define md5_GG4(a0,b0,c0,d0,s0,s1,ac) \
Move(ac) \
G(b0,c0,d0) \
__asm paddd mm7,mm6 \
__asm paddd a0,mm7 \
__asm movd mm6,ecx \
__asm movd mm7,edx \
__asm psll mm6,32 \
__asm por mm7,mm6 \
__asm paddd a0,mm7 \
__asm movq mm7,a0 __asm psrld a0,s0 __asm pslld mm7,s1 __asm por a0,mm7 \
__asm paddd a0,b0

//为 0
#define md5_HH0(a0,b0,c0,d0,s0,s1,ac) \
Move(ac) \
H(b0,c0,d0) \
__asm paddd a0,mm6 \
__asm paddd a0,mm7 \
__asm movq mm7,a0 __asm psrld a0,s0 __asm pslld mm7,s1 __asm por a0,mm7 \
__asm paddd a0,b0

//当 x 数组数据不为 0 时
#define md5_HH1(a0,b0,c0,d0,s0,s1,ac) \
Move(ac) \
H(b0,c0,d0) \
__asm paddd mm7,mm4 \
__asm paddd a0,mm6
```
