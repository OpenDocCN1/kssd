

# 提取一段简易的算法

> 原文：[`www.pediy.com/kssd/pediy11/122146.html`](https://www.pediy.com/kssd/pediy11/122146.html)

代码已经贴在三楼 

从机动车驾驶人考试培训系统 2010（速成版）提取出来的算法  
  
这里感谢一下 Crack Online DDK 群里的 fvlkejr  教了我一个函数 strtol  原先不知道 有这样的函数 呵呵  
  
  
加密部分  
  

代码:

```
BOOL Xor\_Crypt(LPSTR Crypted,size\_t  nBuffSize,LPSTR SourceChar,LPSTR sKey)
{
  char TempChar\[255\];
  int InitNum = 0xA7;
  int DataLen = strlen(SourceChar);
  int nKeyLen = strlen(sKey);
  int nIndex = 0;
  BYTE TempNum = 0;

  RtlZeroMemory(Crypted,nBuffSize);

  for(int i= 0;i< DataLen;i++)
  {
    //sprintf\_s(TempChar,"%02X",InitNum);
    sprintf\_s(TempChar,255,"%02X",InitNum);
    StrCat(Crypted,TempChar);
    TempNum = (SourceChar\[i\] + InitNum) % (signed int)0xFFu;
    if(nIndex >= nKeyLen) nIndex = 0;
    InitNum = sKey\[nIndex\] ^ TempNum;
    nIndex++;
  }

  //sprintf(TempChar,"%02X",InitNum);
  sprintf\_s(TempChar,255,"%02X",InitNum);
  StrCat(Crypted,TempChar);

  return TRUE;
}

```

  
解密部分  
  
代码:

```
BOOL Xor\_DeCrypt(char \* DeCrypted,size\_t nBuffSize,char \*SourceChar,char \* sKey)
{
  char sMidKey\[4\];
  RtlZeroMemory(sMidKey,4);
  RtlCopyMemory(sMidKey,SourceChar,2);//首先取出来密钥
  int nMidKey = strtol(sMidKey,NULL,16);//转换成十六进制
  int TempNum = 0;
  int SourceLen = strlen(SourceChar);
  int nKeyLen = strlen(sKey);
  RtlZeroMemory(DeCrypted,nBuffSize); 
  BYTE b\_Result = 0;
  char szResults = 0x00;
  int nIndex =0;
  int DeCryptCharIndex = 0;
  for(int i = 2;i < SourceLen;i+=2)
  {
    RtlZeroMemory(sMidKey,4);
    RtlCopyMemory(sMidKey,SourceChar+i,2);
    TempNum = strtol(sMidKey,NULL,16); 
    if(nIndex >= nKeyLen) nIndex = 0;   
    b\_Result = sKey\[nIndex\] ^ TempNum;
    if (b\_Result > nMidKey )
      szResults = b\_Result - nMidKey;
    else
      szResults = b\_Result + 0xFF - nMidKey;
    DeCrypted\[DeCryptCharIndex\] = szResults;
    nMidKey = TempNum;
    nIndex++;
    DeCryptCharIndex++;
  }

  return TRUE;
}

```

上传的附件
| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || CrackSoftTest.rar  
 |