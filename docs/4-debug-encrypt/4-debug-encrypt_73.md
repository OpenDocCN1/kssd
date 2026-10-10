

# crypto++5.0 应用一例 

> 原文：[`www.pediy.com/kssd/pediy05/pediy50374.htm`](https://www.pediy.com/kssd/pediy05/pediy50374.htm)

/\*   
目标为娃娃\[CCG\]的在论坛的文章   
一篇关于密码学的入门级破解实例-BiSHoP's CrackMe4 (30 千字)   
地址见 http://www.chat001.com/forum/crackforum/245341.html   
  
根据娃娃老鸟(三年算老鸟了吧)的结论   
Registeration Code = \[ MD5( Name + Organization ) ^ D \] MOD N   
已知 N , D 也已经用 RSATools 算出   
\*/   
  
#include <iostream>   
#include <string.h>   
#include "../crypto50/hex.h"        //HexEncoder   
#include "../crypto50/md5.h"        // MD5   
#include "../crypto50/filters.h"       
#include "../crypto50/files.h"        //FileSink   
#include "../crypto50/rsa.h"        //rsa   
#include "../crypto50/nbtheory.h"    //Integer  大数   
  
int main(int argc, char\* argv\[\])   
{   
    using namespace std;   
    using namespace CryptoPP;   
  
    MD5 md5;   
    string strDigest;        //md5 摘要   
    RSAFunction  rsa;        //rsa   
    Integer plaintext;        //明文    即注册码       
    char name\[100\],organization\[100\];    //姓名,组织   
  
    cout << "Name: ";        //输入姓名   
    ws(cin);   
    cin.getline(name,100);   
  
    cout << "Organization: ";    //输入组织   
    ws(cin);   
    cin.getline(organization,100);   
  
    //计算摘要,结果以 16 进制方式放在 strDigest 中   
    HashFilter hashFilter(md5, new HexEncoder(new StringSink(strDigest)));       
    hashFilter.Put((const byte\*)name, strlen(name));                    //put name   
    hashFilter.Put((const byte\*)organization, strlen(organization));    //put organization   
    hashFilter.MessageEnd();           
  
    strDigest="0x"+strDigest;           
    string letmeat="李嘉欣恶啃到掉渣,让我啃吧,让我啃吧!";    //==>小心花指令   
    Integer c((const char\*) strDigest.data());            //c 以 16 进制初始化   
    Integer d("0x1E2D9B52ADCBC20DCCDE3C721AA740E83");    //d   
    Integer n("0x24DFDA27FA14D3F27DDF62CEA5D2381F9");    //n   
  
    rsa.Initialize(n,d);        //初始化   
    plaintext = rsa.ApplyFunction(c);    //解密 得到明文   
  
//由于未知道有没有方便的方法输出 Integer 对应的十六进制,所以只好一个一个 byte 的干了:<   
  
    byte\* buffer=new byte\[1024\];               
    for (int i=0;i<plaintext.ByteCount();i++)       
        buffer\[plaintext.ByteCount()-1-i\]=plaintext.GetByte(i);  //注意高位在后   
       
    buffer\[plaintext.ByteCount()\]=0;   
  
    //构造一条 pipeline  一般地 pipeline 的头为 Source  尾为 Sink 中间为各种的 filter 类   
    StringSource(buffer,plaintext.ByteCount(), true,           
        new HexEncoder(                        //编码为 16 进制   
            new FileSink(cout)));                //输出到屏幕   
    cout << endl;   
  
    return 0;   
}   
  
/\*   
  
我的注册码  (算娃娃的那两个注册码发现在 code 的前面比他的多了个 0)   
name=yyxzz   
organization=\[CCG\]   
code=D54E3A1976B790C3FCCE022B777FA875   
  
  
有兴趣的朋友可参考如下网站或资料   
crypto++作者 weidai(相信是一个具有华人血统的 snooker 高手)  其中的 faq 不可不看   
        http://www.eskimo.com/~weidai/cryptlib.html   
doxygen 的主页    http://www.trolocsis.com/crypto++/index.html   
CryptoPPGuide.chm 在 google 上 虽然有点过时,但还是很有用的   
  
  
                                yyxzz\[CCG\] 2002.12.29   
\*/  4dd466d0e1e0cc123afbe504da2462ca