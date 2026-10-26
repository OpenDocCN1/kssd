
```cpp
__asm add a0,b0

//当x数组数据不为0时

#define md5_HH1(a0,b0,c0,d0,s0,s1,ac) \
        H(b0,c0,d0) \
        __asm movd edi,mm0 \
        __asm add a0,esi \
        __asm add edi,ac \
        __asm add a0,edi \
        __asm rol a0,s1 \
__asm add a0,b0

#define md5_HH2(a0,b0,c0,d0,s0,s1,ac) \
        H(b0,c0,d0) \
        __asm movd edi,mm1 \
        __asm add a0,esi \
        __asm add edi,ac \
        __asm add a0,edi \
        __asm rol a0,s1 \
__asm add a0,b0

#define md5_HH3(a0,b0,c0,d0,s0,s1,ac) \
        H(b0,c0,d0) \
        __asm movd edi,mm2 \
        __asm add a0,esi \
        __asm add edi,ac \
        __asm add a0,edi \
        __asm rol a0,s1 \
__asm add a0,b0

#define md5_HH4(a0,b0,c0,d0,s0,s1,ac) \
        H(b0,c0,d0) \
        __asm movd edi,mm3 \
        __asm add a0,esi \
        __asm add edi,ac \
        __asm add a0,edi \
        __asm rol a0,s1 \
__asm add a0,b0

//第一个II,就是不需要加任何东西上去

#define md5_II0(a0,b0,c0,d0,s0,s1,ac) \
        I(b0,c0,d0) \
        __asm add a0,esi \
        __asm add a0,ac \
        __asm rol a0,s1 \
__asm add a0,b0

//当x数组数据不为0时

#define md5_II1(a0,b0,c0,d0,s0,s1,ac) \
        I(b0,c0,d0) \
        __asm movd edi,mm0 \
        __asm add a0,esi \
        __asm add edi,ac \
        __asm add a0,edi \
        __asm rol a0,s1 \
__asm add a0,b0

#define md5_II2(a0,b0,c0,d0,s0,s1,ac) \
        I(b0,c0,d0) \
        __asm movd edi,mm1 \
        __asm add a0,esi \
        __asm add edi,ac \
        __asm add a0,edi \
        __asm rol a0,s1 \
__asm add a0,b0

#define md5_II3(a0,b0,c0,d0,s0,s1,ac) \
        I(b0,c0,d0) \
        __asm movd edi,mm2 \
        __asm add a0,esi \
        __asm add edi,ac \
        __asm add a0,edi \
        __asm rol a0,s1 \
__asm add a0,b0

#define md5_II4(a0,b0,c0,d0,s0,s1,ac) \
        I(b0,c0,d0) \
        __asm movd edi,mm3 \
        __asm add a0,esi \
        __asm add edi,ac \
        __asm add a0,edi \
        __asm rol a0,s1 \
__asm add a0,b0

///////////////////////////////////////

// QQ解密正式开始

//////////////////////////////////////

//作者：冲出宇宙

//2005，11，1

//////////////////////////////////////

UI8 MD5Result[16];        //md5加密后的结果

UI32 LoopNum;                //循环次数

UI32 QQNum;                        //qq号码

UI32 PwdNum;                //已经对多少个字符串进行了检查

CString Pwd;                //当前正确的密码

CString FileQQ,FilePwd;        //密码文件和密码字典文件

DWORD DecodeThreadId;

HANDLE DecodeThreadHandle;        //解密线程

CQQDecoderCtrl *Me;

//密码是否正确，每次测试2个密码

bool IsPasswordCorrect(char* sPwd0)

{

        UI32 i,dataIn0[4];

        MD5 md5;

        //第一轮

        md5.Hash((UI8*)dataIn0,(UI8*)sPwd0,strlen(sPwd0));

        __asm pushad;

        //准备输入数据

        __asm

        {

                //1,m_data[0]

                mov esi,[dataIn0];

                movd mm0,esi;

                //2,

                mov esi,[dataIn0+4];

                movd mm1,esi;

                //3,

                mov esi,[dataIn0+8];

                movd mm2,esi;

                //4,

                mov esi,[dataIn0+12];

                movd mm3,esi;

        }

        //主要轮次

        __asm

        {

                push ebp;

                mov ebp,LoopNum;

        }

Begin:

        __asm

        {

                dec ebp;

                cmp ebp,0;

                je End;

                __asm

                {

                        mov a,0x67452301;

                        mov b,0xefcdab89;

                        mov c,0x98badcfe;

                        mov d,0x10325476;

                }

                md5_FF1 (a, b, c, d,  md5_S11, md5_S110,0xd76aa478);

                md5_FF2 (d, a, b, c,  md5_S12,  md5_S120,0xe8c7b756);

                md5_FF3 (c, d, a, b,  md5_S13, md5_S130,0x242070db);

                md5_FF4 (b, c, d, a,  md5_S14, md5_S140,0xc1bdceee);

                md5_FF0 (a, b, c, d,  md5_S11,  md5_S110,0xf57c102f);

                md5_FF0 (d, a, b, c, md5_S12, md5_S120,0x4787c62a);

                md5_FF0 (c, d, a, b, md5_S13, md5_S130,0xa8304613);

                md5_FF0 (b, c, d, a, md5_S14, md5_S140,0xfd469501);

                md5_FF0 (a, b, c, d, md5_S11,md5_S110, 0x698098d8);

                md5_FF0 (d, a, b, c, md5_S12, md5_S120,0x8b44f7af);

                md5_FF0 (c, d, a, b, md5_S13,  md5_S130,0xffff5bb1);

                md5_FF0 (b, c, d, a, md5_S14, md5_S140,0x895cd7be);

                md5_FF0 (a, b, c, d, md5_S11, md5_S110,0x6b901122);

                md5_FF0 (d, a, b, c, md5_S12, md5_S120,0xfd987193);

                md5_FF0 (c, d, a, b,md5_S13, md5_S130,0xa679440e);

                md5_FF0 (b, c, d, a, md5_S14,  md5_S140,0x49b40821);

                md5_GG2 (a, b, c, d,md5_S21, md5_S210,0xf61e2562);

                md5_GG0 (d, a, b, c, md5_S22, md5_S220,0xc040b340);

                md5_GG0 (c, d, a, b, md5_S23, md5_S230,0x265e5a51);

                md5_GG1 (b, c, d, a,md5_S24, md5_S240,0xe9b6c7aa);

                md5_GG0 (a, b, c, d, md5_S21, md5_S210,0xd62f105d);

                md5_GG0 (d, a, b, c, md5_S22, md5_S220, 0x2441453);

                md5_GG0 (c, d, a, b, md5_S23, md5_S230,0xd8a1e681);

                md5_GG0 (b, c, d, a, md5_S24, md5_S240,0xe7d3fc48);

                md5_GG0 (a, b, c, d, md5_S21, md5_S210,0x21e1cde6);

                md5_GG0 (d, a, b, c, md5_S22, md5_S220,0xc3370856);

                md5_GG4 (c, d, a, b,md5_S23, md5_S230,0xf4d50d87);

                md5_GG0 (b, c, d, a, md5_S24, md5_S240,0x455a14ed);

                md5_GG0 (a, b, c, d, md5_S21, md5_S210,0xa9e3e905);

                md5_GG3 (d, a, b, c,md5_S22, md5_S220,0xfcefa3f8);

                md5_GG0 (c, d, a, b, md5_S23, md5_S230,0x676f02d9);

                md5_GG0 (b, c, d, a, md5_S24,md5_S240, 0x8d2a4c8a);

                md5_HH0 (a, b, c, d, md5_S31, md5_S310,0xfffa3942);

                md5_HH0 (d, a, b, c, md5_S32, md5_S320,0x8771f681);

                md5_HH0 (c, d, a, b, md5_S33, md5_S330,0x6d9d6122);

                md5_HH0 (b, c, d, a, md5_S34, md5_S340,0xfde5388c);

                md5_HH2 (a, b, c, d,md5_S31, md5_S310,0xa4beea44);

                md5_HH0 (d, a, b, c, md5_S32, md5_S320,0x4bded029);

                md5_HH0 (c, d, a, b, md5_S33, md5_S330,0xf6bb4b60);

                md5_HH0 (b, c, d, a, md5_S34, md5_S340,0xbebfbc70);

                md5_HH0 (a, b, c, d, md5_S31, md5_S310,0x289b7ec6);

                md5_HH1 (d, a, b, c,md5_S32, md5_S320,0xeaa127fa);

                md5_HH4 (c, d, a, b,md5_S33, md5_S330,0xd4ef3085);

                md5_HH0 (b, c, d, a, md5_S34, md5_S340, 0x4881d05);

                md5_HH0 (a, b, c, d, md5_S31, md5_S310,0xd9d4d039);

                md5_HH0 (d, a, b, c, md5_S32, md5_S320,0xe6db99e5);

                md5_HH0 (c, d, a, b, md5_S33, md5_S330,0x1fa27cf8);

                md5_HH3 (b, c, d, a,md5_S34, md5_S340,0xc4ac5665);

                md5_II1 (a, b, c, d,md5_S41, md5_S410,0xf4292244);

                md5_II0 (d, a, b, c, md5_S42,  md5_S420,0x432aff97);

                md5_II0 (c, d, a, b, md5_S43, md5_S430,0xab942427);

                md5_II0 (b, c, d, a, md5_S44, md5_S440,0xfc93a039);

                md5_II0 (a, b, c, d, md5_S41, md5_S410,0x655b59c3);

                md5_II4 (d, a, b, c,md5_S42, md5_S420,0x8f0ccc92);

                md5_II0 (c, d, a, b, md5_S43, md5_S430,0xffeff47d);

                md5_II2 (b, c, d, a, md5_S44, md5_S440,0x85845dd1);

                md5_II0 (a, b, c, d, md5_S41, md5_S410,0x6fa87e4f);

                md5_II0 (d, a, b, c, md5_S42, md5_S420,0xfe2ce6e0);

                md5_II0 (c, d, a, b, md5_S43,md5_S430, 0xa3014314);
```
