

# 浅谈利用 RSA 算法防止非法注册机的制作

> 原文：[`www.pediy.com/kssd/pediy06/pediy6533.htm`](https://www.pediy.com/kssd/pediy06/pediy6533.htm)

　　　　　　　　　　　　　　《浅谈利用 RSA 算法防止非法注册机的制作》   
  
　　　　　　　　　　　　　　  
  
一：RSA 简介。  
  
　　RSA 公开密钥密码系统是由 R.Rivest，A.Shamir，L.Adleman 提出的，不仅仅可用于数据的加密，也可用于数字签名，其算法如下：  
  
　　1，取两个相近的大素数 p、q；  
　　2，计算 n=p\*q，z=(p-1)\*(q-1)；  
　　3，任取一个与 z 互素的整数 e；  
　　4，计算满足 e\*d=1 mod z 的整数 d；  
　　5，将明文 m 分成字符块 s 加密，每个块 s 小于 n。现设明文 m 小于 n，加密后形成密文 c。 加密、解密过程如下:  
  
　　加密：c=m^e mod n  
　　解密：m=c^d mod n  
  
　　6，(n,e)和(n,d)分别称为“公开密钥”和“秘密密钥”。根据 Euler 定理可得：  
  
　　m=c^d mod n=(m^e mod n)^d mod n=m  
  
　　现举例说明其工作过程：取两个素数 p=11，q=13，n=p\*q=11\*13=143，z=(p-1)\*(q-1)=(11-1)\*(13-1)=120，再选取与 z=120 互素的整数 e，如 e=7，现可计算出满足 7\*d=1 mod 120 的整数 d=103，即：7\*103=1 mod 120，7\*103/120 余 1，整理如下：  
  
　　p=11  
　　q=13  
　　n=143  
　　e=7  
　　d=103  
　　(n,e)=(143,7)  
　　(n,d)=(143,103)  
  
　　以数据加密为例：  
  
　　甲向乙发送机密数据信息 m=85，并已知乙的公钥(n,e)=(143,7)，于是可计算出：  
c=m^e mod n=85⁷ mod 143=123  
甲将 c 发送至乙，乙利用私钥(n,d)=(143,103)对 c 进行计算：  
m=c^d mod n=123¹⁰³ mod 143=85  
现乙已经得到甲向其要发送的机密数据信息。在这里，甲向乙发送信息，甲所拥有的仅仅是乙的公钥。  
  
  
  
  
　　以数字签名为例：  
  
　　乙要向甲发送信息，并要让甲确信此信息是由乙本人所发出的，于是，乙将能代表自己身份的编码值（如：123），利用私钥(n,d)=(143,103)进行计算，并将结果发送给甲：  
m=c^d mod n=123¹⁰³ mod 143=85  
甲接受到乙的数字签名后利用乙的公钥(n,e)=(143,7)进行计算，得出代表乙身份的编码：  
c=m^e mod n=85⁷ mod 143=123  
现甲经过验证已确信信息的发送方为乙。因为只有乙拥有私钥(n,d)，来对代表自己身份的编码 123 进行计算。在不知道乙私钥(n,d)的情况下，任何人都不会算出 85 这一签名来冒充乙。在这里，乙向甲发送信息并进行签名，甲所拥有的也仅仅是乙的公钥来验证乙的签名。  
  
　　从上面两个例子中我们可以更好得理解这一结论，即：由(n,e)加密的数据只能用(n,d)解密，反之亦然。  
  
  
二：在基于序列号保护的共享软件中应用 RSA。  
  
　　结合数字签名的实例能更好得理解这一应用：在某一共享软件中，甲想用 123 为注册名进行软件注册，他现在拥有的仅仅是存在于共享软件程序中的公钥(n,e)=(143,7)。甲现将 123 为注册名向乙提出注册申请，乙得知此申请并通过此申请后，便利用所拥有的私钥(n,d)对注册名 123 进行计算：  
m=c^d mod n=123¹⁰³ mod 143=85  
甲得到计算后的结果 85（序列号），提供给共享软件的注册程序进行计算：  
c=m^e mod n=85⁷ mod 143=123  
然后注册程序将判断计算结果 c 是否为 123（注册名），以决定注册是否通过。  
  
　　如果甲随意输入一组序列号利用公钥(n,e)进行计算，那他得到的结果将不是 123，注册也就失败了。注意，在这里，共享软件的注册程序比较的是注册名，而不是序列号。如果甲跟踪注册程序得到了他所随意输入的序列号所产生的注册名，将其提供给注册程序，那注册程序也能够通过注册，但他由于没有(n,d)，所以他无法用自己的注册名进行软件注册，也就防止了非法注册机的制作。  
  
　　将 RSA 应用于此的目的仅仅是防止非法注册机的制作，在以上描述中，n=143，包括以下 Demo 中的 n(HEX)=963251DC5A9C90D9F203A03C363BA411，以现在的计算机处理速度，能很快地将其因式分解得到相应的 p，q，再结合暴露在共享软件注册程序中的 e，从而计算出 d，那么这个共享软件的保护就完全被破解了。解决的方法是要避免 n 过短，以及结合 MD5 等加密算法……  
  
  
三：具体实践。  
  
/\* RSA Demo 1.0 版  
\* 版权所有 (C) 2004 赵春生  
\* 2004.04.25  
\* http://timw.yeah.net  
\* http://timw.126.com  
\* 本程序调用 Miracl ver 4.82 大数运算库，详见其附带手册。  
\* P,Q,N,D,E 使用 RSATool2 生成。  
\*/  
  
  
1，注册机的关键代码：  
  
void CKeyGenDlg::OnGen()   
{  
// TODO: Add your control notification handler code here  
  
/\* KeyGen 1.0 版  
\* 版权所有 (C) 2004 赵春生  
\* 2004.04.25  
\* http://timw.yeah.net  
\* http://timw.126.com  
\* 本程序调用 Miracl ver 4.82 大数运算库，详见其附带手册。  
\*/  
  
//////////////////////////////////////////////////////////////////////////  
  
//P(HEX)=E34436F5F48A227B  
//Q(HEX)=A92FA24467C4E3E3  
//N(HEX)=963251DC5A9C90D9F203A03C363BA411  
//D(HEX)=56157D29A89D77BF2F669A8F0B123CC9  
//E(HEX)=10001  
//Keysize(Bits)=128  
  
UpdateData(TRUE);  
  
int namelen=m\_name.GetWindowTextLength (); //获取 Name 的长度  
  
//务必满足：Name 的长度<=(Keysize/8)，使 M<N 成立。  
if (namelen!=0&&namelen<=16) //如果不为空且<=128/8 则开始计算 SN  
{  
miracl \*mip=mirsys(100,0);  
mip->IOBASE=16; //16 进制模式  
  
//定义并初始化变量  
big m=mirvar(0); //m 放明文：注册码 SN  
big c=mirvar(0); //c 放密文：用户名 Name  
big n=mirvar(0); //n 模数  
big d=mirvar(0); //d 私钥  
TCHAR Name\[256\]={0};  
TCHAR SN\[256\]={0};  
  
m\_name.GetWindowText (Name,namelen+1); //获取 Name  
bytes\_to\_big(namelen,Name,c); //转换成 16 进制  
cinstr(n,"963251DC5A9C90D9F203A03C363BA411"); //初始化模数 n   
cinstr(d,"56157D29A89D77BF2F669A8F0B123CC9"); //初始化私钥 d  
powmod(c,d,n,m); //计算 m=c^d mod n  
cotstr(m,SN); //将 m 以 16 进制串写入 SN  
m\_sn.SetWindowText (SN); //输出 16 进制 SN  
  
//释放内存  
mirkill(m);  
mirkill(c);  
mirkill(n);  
mirkill(d);  
mirexit();  
}  
else  
m\_sn.SetWindowText ("Error: Invalid registration name.");  
  
  
UpdateData(FALSE);  
}  
  
  
2，软件中注册程序的关键代码：  
  
void CRSAApplicationDlg::OnReg()   
{  
// TODO: Add your control notification handler code here  
  
/\* RSA Application 1.0 版  
\* 版权所有 (C) 2004 赵春生  
\* 2004.04.25  
\* http://timw.yeah.net  
\* http://timw.126.com  
\* 本程序调用 Miracl ver 4.82 大数运算库，详见其附带手册。  
\*/  
  
//////////////////////////////////////////////////////////////////////////  
  
//P(HEX)=E34436F5F48A227B  
//Q(HEX)=A92FA24467C4E3E3  
//N(HEX)=963251DC5A9C90D9F203A03C363BA411  
//D(HEX)=56157D29A89D77BF2F669A8F0B123CC9  
//E(HEX)=10001  
//Keysize(Bits)=128  
  
UpdateData(TRUE);  
  
miracl \*mip=mirsys(100,0);  
mip->IOBASE=16; //16 进制模式  
  
//定义并初始化变量  
big m=mirvar(0); //m 放明文：注册码 SN  
big c=mirvar(0); //c 放密文：用户名 Name  
big n=mirvar(0); //n 模数  
big e=mirvar(0); //e 公钥  
TCHAR Name\[256\]={0};  
TCHAR SN\[256\]={0};  
TCHAR temp\[256\]={0};  
int len=0;  
int i,j;  
  
//获取 Name  
len=m\_name.GetWindowTextLength ();  
m\_name.GetWindowText (Name,len+1);  
  
//获取 SN  
len=m\_sn.GetWindowTextLength ();  
m\_sn.GetWindowText (SN,len+1);  
  
//检查 SN 是否为 16 进制  
for (i=0,j=0;i<len;i++)  
{  
if(isxdigit(SN\[i\])==0)  
{  
j=1;  
break;  
}  
}  
  
//如果输入的 SN 为 16 进制且长度不为 0  
if (j!=1&&len!=0)  
{  
  
cinstr(m,SN); //初始化明文 m  
cinstr(n,"963251DC5A9C90D9F203A03C363BA411"); //初始化模数 n   
cinstr(e,"10001"); //初始化公钥 e  
  
//当 m<n 时  
if(compare(m,n)==-1)  
{  
powmod(m,e,n,c);//计算 c=m^e mod n  
big\_to\_bytes(256,c,temp,FALSE);//将 c 转换成数组写入 temp  
  
//释放内存  
mirkill(m);  
mirkill(c);  
mirkill(n);  
mirkill(e);  
mirexit();  
}  
else  
j=1;  
  
}  
  
  
//对 Name、temp， m、n， SN 的长度进行检查  
if(lstrcmp(Name,temp)!=0||j==1||len==0)  
MessageBox("Please check your NAME and SN, then try again.","RSA Application");  
else  
MessageBox("Congratulate!!!","Registration complete!");  
  
UpdateData(FALSE);  
  
}  
  
//////////////////////////////////////////////////////////////////////////////////  
  
编译提示：  
  
1：将 Project-Settings-Settings For(All Configuration)-C/C++中 Category 项的  
Precompiled Headers 设置成：Automatic use of precompiled headers。  
  
2：将 ms32.lib 添加到工程中。  
  
3：MIRACL 是 C 库。  
  
extern "C"  
{  
#include "miracl.h"  
#include "mirdef.h"  
}  
  
#pragma comment( lib, "ms32.lib" )   
  
  
四：以上代码在 Win2000P+SP4 + VC6+SP5 测试通过。  
　　RSA Demo 1.0 源码可从我的个人主页下载。  
　　http://timw.yeah.net  
　　http://timw.126.com  
  
14:15 2004-4-26 

 4dd466d0e1e0cc123afbe504da2462ca