

# 根据 CRC32 值，是否能够快速获得其原始值？

> 原文：[`www.pediy.com/kssd/pediy10/66767.html`](https://www.pediy.com/kssd/pediy10/66767.html)

有一个 CRC32 的值，是否能够快速获取其原始值？  
CRC 是多对一映射，能够回复其中的 n 个就可以了  
  
查了一些资料，没找到其源码，恳请大侠指教 

由 crchead, crcfinal 可快速得出 1 个 32bit 的值,这个 32 bit 是可逆的。  
附件是我写的一个工具。  
比如：  
crchead =  0xFFFFFFFF;  
crcfianl = 0xDEADBEEF;  
5 byte， 每个 byte 范围 0 - FF,  
则符合值为：  
004E3726D4  
01D80721A3  
026256283A  
...  
FE55E8238E  
FFC3D824F9  
  
它们的 crc 都是  
0xDEADBEEF  
  
参考：  
http://www.pediy.com/bbshtml/BBS6/pediy6937.htm

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || crc3x.rar  
 |

下面是 crc32/ crc16 求逆的代码。  
其实就是 DonQuixote 的： CRC32 碰撞的实现。做了以下改进。  
以前看过一个软件注册算法用到了 crc16 的逆，crc16 的逆就更简单了。  
  
全部采用的查表法，速度较快。  
首先构造 tab/ rf\_tab。rf\_tab 是求逆的表。  
  
  
typedef unsigned char      BYTE;  
typedef unsigned short     WORD;  
typedef unsigned int       DWORD;  
  
#define LOWORD(l)           ((WORD)(l))  
#define HIWORD(l)           ((WORD)(((DWORD)(l) >> 16) & 0xFFFF))  
#define LOBYTE(w)           ((BYTE)(w))  
#define HIBYTE(w)           ((BYTE)(((WORD)(w) >> 8) & 0xFF))  
#define MAKEWORD(a, b)      ((WORD)(((BYTE)(a)) | ((WORD)((BYTE)(b))) << 8))  
#define MAKEDWORD(a, b)     ((DWORD)(((WORD)(a)) | ((DWORD)((WORD)(b))) << 16))  
  
#define MakeWord(a,b)    MAKEWORD(b,a)  
#define MakeDWord(a,b)   MAKEDWORD(b,a)  
  
  
// crc32/ crc16 funs  
#define CRC32\_NEW   (0xFFFFFFFFUL)   
#define CRC32\_POLY  (0xEDB88320UL)  
#define CRC16\_NEW   (unsigned short)(0xFFFF)   
#define CRC16\_POLY  (unsigned short)(0x8408)   
#define CRC16\_STD\_POLY  (unsigned short)(0xA001)   
  
  
unsigned int crc32\_tab\[256\];  
unsigned short crc16\_tab\[256\];  
  
unsigned char crc32\_rf\[256\];  
unsigned char crc16\_rf\[256\];  
  
void init\_crc32\_table(unsigned int poly)  
{  
    unsigned int i,j;  
    unsigned int crc;  
    for( i = 0; i < 256; i ++ ) {  
        crc = i;  
        for( j = 0; j < 8; j ++ ){  
            if(crc & 1) crc = (crc >> 1) ^ poly;  
            else crc >>= 1;  
        }  
        crc32\_tab\[i\] = crc;  
    }  
    return;  
}  
  
void init\_crc32\_rf\_table(void)  
{  
    unsigned int i,j;  
      
    for(i=0; i<256; i++) {  
        j = HIBYTE(HIWORD(crc32\_tab\[i\]));  
        crc32\_rf\[j\] = i;  
    }  
    return;  
}  
  
/\* must init\_crc32\_rf\_table() first \*/   
#define RF(x)   crc32\_rf\[(BYTE) x\]  
#define F(x)    HIBYTE(HIWORD(crc32\_tab\[(BYTE) x\]))  
#define G(x)    LOBYTE(HIWORD(crc32\_tab\[(BYTE) x\]))  
#define H(x)    HIBYTE(LOWORD(crc32\_tab\[(BYTE) x\]))  
#define I(x)    LOBYTE(LOWORD(crc32\_tab\[(BYTE) x\]))  
  
/\*  
given crc\_head = ABCD  
find abcd;  
that crc(abcd) = WXYZ  
\*/  
unsigned int rCRC32(unsigned int WXYZ,unsigned int ABCD)  
{  
    BYTE p,o,n,m,a,b,c,d,W,X,Y,Z,A,B,C,D;  
      
    W=HIBYTE(HIWORD(WXYZ));  
    X=LOBYTE(HIWORD(WXYZ));  
    Y=HIBYTE(LOWORD(WXYZ));  
    Z=LOBYTE(LOWORD(WXYZ));  
      
    A=HIBYTE(HIWORD(ABCD));  
    B=LOBYTE(HIWORD(ABCD));  
    C=HIBYTE(LOWORD(ABCD));  
    D=LOBYTE(LOWORD(ABCD));  
      
    p=crc32\_rf\[W\];  
    o=crc32\_rf\[(BYTE) (X^G(p))\];  
    n=crc32\_rf\[(BYTE) (Y^G(o)^H(p))\];  
    m=crc32\_rf\[(BYTE) (Z^G(n)^H(o)^I(p))\];  
      
    d=m^D;  
    c=n^C^I(m);  
    b=o^B^H(m)^I(n);  
    a=p^A^G(m)^H(n)^I(o);      
    return MakeDWord(MakeWord(a,b),MakeWord(c,d));  
}  
  
  
  
#define FF(x)  HIBYTE(crc16\_tab\[(unsigned char)x\])  
#define GG(x)  LOBYTE(crc16\_tab\[(unsigned char)x\])  
  
void init\_crc16\_table(unsigned short poly)  
{  
    unsigned int i,j;  
    unsigned short crc;  
    for( i = 0; i < 256; i ++ ) {  
        crc = i;  
        for( j = 0; j < 8; j ++ ){  
            if(crc & 1) crc = (crc >> 1) ^ poly;  
            else crc >>= 1;  
        }  
        crc16\_tab\[i\] = crc;  
    }  
    return;  
}  
  
void init\_crc16\_rf\_table(void)  
{  
    unsigned int i,j;  
      
    for(i=0; i<256; i++) {  
        j = HIBYTE(crc16\_tab\[i\]);  
        crc16\_rf\[j\] = i;  
    }  
    return;  
}  
  
unsigned short rCRC16(unsigned short XY, unsigned short AB)  
{  
    BYTE X,Y,A,B,a,b,m,n;   
      
    X=HIBYTE(XY);  
    Y=LOBYTE(XY);  
    A=HIBYTE(AB);  
    B=LOBYTE(AB);  
      
    n=crc16\_rf\[X\];  
    m=crc16\_rf\[(BYTE) (Y^GG(n))\];  
      
    b=m^B;  
    a=n^A^GG(m);   
    return MakeWord(a,b);  
} 

前面的 crc3x 是可以指定范围的。  
  
比如 12 字节，len=12  
范围  
'1' - '9'  
CHAR:  
0x31 - 0x39  
想要多少个，NUM 填示例数目。  
比如我要 5 个实例，num = 5.  
  
crc32 = 0x12345678  
  
数据会自动保存在 crc32x-yyyymmdd-hh-mm.txt  
  
CRC32x-20080619-2208.txt  
116534385183  
126488432389  
131875263511  
177354617145  
188428597511