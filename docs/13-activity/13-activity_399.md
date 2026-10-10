

# 第二阶段◇第二题

> 原文：[`www.pediy.com/kssd/pediy11/124218.html`](https://www.pediy.com/kssd/pediy11/124218.html)

VS 2009 编译  
  
Input your name(1-32chars):DiKeN/iPB  
\===============================================  
Name:DiKeN/iPB  
Code:3VC0MA6Y-TQJECPTN-C5TH1Z4F-47R4WDG3  
\===============================================  
Press any key to exit!

上传的附件
| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || kg2.rar  
 |

引用: *最初由 **DiKeN**发布* ![查看帖子](img/showthread.php?p=884096#post884096) VS 2009 编译  
  
Input your name(1-32chars):DiKeN/iPB  
\===============================================  
Name:DiKeN/iPB  
Code:3VC0MA6Y-TQJECPTN-C5TH1Z4F-4... 

没看到还需要分析文档![](img/bedc2a7572d1054a9590e607b9635bdd.jpg "皱眉"), 补上  
代码:

```


class CBase32Decoder
{
public:
  CBase32Decoder(const char \*Lookuptable)
  {

  }
  char IndexOf(const char AChar)
  {
    return 0;//未实现
  }
  bool Decode(const char \*In, int InLength, unsigned char \*Out, int &OutLength)
  {
    return true;//未实现
  }

protected:
private:
  const char \*m\_LookupTable;
};

bool Encode1(unsigned char \*InData, unsigned char \*OutData)
{
  return true;//未实现
}

class CXxxxEncoder2
{
public:
  CXxxxEncoder2(unsigned short key):m\_key(key)
  {

  }
  unsigned char EncodeByte(unsigned char AByte)
  {
    return 0;//未实现
  }
protected:
private:
  unsigned short m\_key;
};

void Encode2(unsigned char \*Buffer, int Len)
{
  CXxxxEncoder2 enc2(0x11B);
  for (int i = 0; i < Len; i++)
  {
    Buffer\[i\] = enc2.EncodeByte(Buffer\[i\]);
  }
}

bool Encode3(unsigned char \*InBuffer, void \*OutBuffer, unsigned char \*InitVector)
{
  //blowfish 算法变种, CBC 模式
  return true;//未实现    
}

bool Encode4(void \*Data, int Length, void \*sig)
{
  //ECDSA 算法, 需要 crypto 库,
  unsigned char \_qx\[\] = {0x29, 0x7A, 0x4A, 0x1E, 0x5B, 0x1F, 0xC9, 0x9B};
  unsigned char \_qy\[\] = {0x88, 0x01, 0x98, 0xE3, 0x72, 0x4F, 0x9F, 0xEE};
  Integer a(-3);
  Integer b(Integer::Sign::POSITIVE, 0x22996B9C, 0x33AEEFDB);
  Integer p(Integer::Sign::POSITIVE, 0xC564EEF0, 0x70E69193);
  Integer n(Integer::Sign::POSITIVE, 0xC564EEF1, 0x9A080B07);
  Integer gx(Integer::Sign::POSITIVE, 0x2223A1D5, 0x95845FA2);//x
  Integer gy(Integer::Sign::POSITIVE, 0x1C4EEE92, 0x22DDDA62);//y
  Integer qx(\_qx, 8);//x
  Integer qy(\_qy, 8);//y    

  ECP::Point G( gx, gy );//G
  ECP::Point Q( qx, qy );//Param2    

  ECP ec(p, a, b);
  ECDSA<ECP, SHA1>::Verifier pub(ec, G, n, Q);
  return  pub.VerifyMessage((byte \*)Data, Length, sig, 0x10);
  //return true;
}

HWND hDlg;
HWND hWndName;
HWND hWndCode;

char TableMinsPos\[\] = {8, 17, 26};

void Check()
{
  //00408270
  char szName\[32+1\];//align                //输入的 Name
  char szCode\[36\];//32 + 3 + 1             //输入的 Code
  char szCode2\[32\];//32                    //去掉'-'的 Code
  unsigned char Code\[20+1\];//32\*5/8=20       //
  unsigned char EncBuf1\[16\];
  unsigned int EncBuf2\[4\];//
  CBase32Decoder base32decoder("ABCDEFGHJKMNPQRSTVWXYZ1234567890");
  int MaxLen = sizeof(Code);
  memset(szName, 0, sizeof(szName));
  memset(szCode, 0, sizeof(szCode));
  memset(szCode2, 0, sizeof(szCode2));
  memset(Code, 0, sizeof(Code));
  memset(EncBuf1, 0, sizeof(EncBuf1));
  memset(EncBuf2, 0, sizeof(EncBuf2));
  //004083C5//获取用户名
  if (0 == SendMessageA(hWndName, WM\_GETTEXT, sizeof(szName), (LPARAM)(&szName\[0\]))) return;//不合法
  //0040841B//获取注册码
  if ((sizeof(szCode) - 1) == SendMessageA(hWndName, WM\_GETTEXT, sizeof(szCode), (LPARAM)(&szCode\[0\]))) return;//不合法
  //00408438//检查注册码格式并去掉-号
  int k = 0;
  for (int i = 0; i < sizeof(szCode) - 1; i++)
  {
    int j = 0;
    for (; j < 3; j++)
    {
      if (i == TableMinsPos\[j\])
      {
        if (szCode\[i\] != '-') return;//不合法
      }
    };
    if (j == 3)
    {
      if (-1 == base32decoder.IndexOf(szCode\[i\])) return;//不是合法字符
      szCode2\[k\] = szCode\[i\];
      k++;
    };
  };
  //004084A0 //Base32 解码注册码
  if (!base32decoder.Decode(szCode2, 32, Code, MaxLen)) return ;//
  //004084C8//一个简单的算法, 名称未知, 很容易逆向
  if (!Encode1(Code, EncBuf1)) return;//
  //004084E3//一个简单的算法, 名称未知, 很容易逆向
  Encode2(EncBuf1, 16);//
  //00408511//一个简单的算法, 同上
  Encode2(&Code\[16\], 4);//
  //00408541//CBC 模式 Blowfish 算法变种
  if (!Encode3(&EncBuf1\[0\], &EncBuf2\[0\], &Code\[16\])) return ;//
  //00408563//异或校验和检查----------------------------------------关键点 1(不用分析)
  if (\*((unsigned int \*)&Code\[16\]) != (EncBuf2\[0\] ^ EncBuf2\[1\] ^ EncBuf2\[2\] ^ EncBuf2\[3\])) return;//
  //00408588//ECDSA 数据签名检查-------------------------------------关键点 2(见 Encode4)
  if (Encode4(szName, strlen(szName), &EncBuf2))
  {
    MessageBoxA(hDlg, "Congratulations! \\n You will be the keygen machine!", "Success!", MB\_OK);
  }
  //004085C5//结束
}

```

上传的附件
| ![](img/30b3588c36e23afc1abe1f92c2cce6e9.jpg) || kg2 分析文档.txt  
 |