
# VFP 的 APP 文件头最后一个 WORD 表示的校验和是怎样计算出来的啊

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-812.htm`](https://www.pediy.com/kssd/pediy08/pediy8-812.htm)

在论坛里有个人也问了同样的问题！

引用:  *最初由** 行者孙 **发布*  

在论坛里有个人也问了同样的问题，他说自己后来解决了，而且给出了汇编的代码，可我汇编看不懂  

，哪位高手能说下计算的过程吗？ 

老流氓的定义：![](img/77a5f7cfd3ec840c98be3aecc80df838.jpg)  

```cpp
//---------------------------------------------------------------------------  

//  

// FOX Header struct define  

//  

//---------------------------------------------------------------------------  

typedef struct  
{  WORD  Magic;           // 00,FOX 文件标识：FE F2  
  BYTE  Flag;           // 02,FOX 类型："加密类型" EE 或者 "未加密类型"FF  
  WORD  Version;         // 03,FOX 版本  
  WORD  NumOfFiles;         // 05,文件列表中定义的文件个数  
  WORD  OrdOfMainPrg;               // 07,主文件的序号  
  DWORD  EndPosOfBlob;               // 09,字符串信息的结束位置  
  DWORD  StartPosOfBlob;               // 0D,字符串信息的起始位置  
  DWORD  SizeOfBlob;         // 11,字符串信息的长度  
  BYTE  Reserved\[0x12\];               // 15,保留字节  
  WORD  CheckSum;         // 27,校验和  
} FOX\_HEADER,\* PFOX\_HEADER;  

WORD CalcCheckSum(PFOX\_HEADER pFoxHeader)  
{  return CalcFoxCheckSum(pFoxHeader,0xFEF2,0x25);  
}  

WORD SwapByte(WORD wCheckSum)  
{  
  return (((BYTE)wCheckSum << 8) + ((wCheckSum >> 8) & 0x0FF)) & 0x0FFFF;  
}  

WORD CalcFoxCheckSum(PFOX\_HEADER pFoxHeader,WORD wInitKey,int nSize)  
{   
  
  PBYTE pHeader = (PBYTE)pFoxHeader;  
  int i = nSize;  
  WORD wCheckSum = wInitKey;  
  do  
  {  
    wCheckSum = SwapByte(wCheckSum);  
    wCheckSum = (\*pHeader) ^ wCheckSum;  
    wCheckSum ^= (((BYTE)wCheckSum >> 4) & 0x0F);  
    wCheckSum ^= (((BYTE)wCheckSum ^ (wCheckSum << 7)) << 5 );  
    pHeader ++;  
    i --;  
  }while(i > 0);  
  return wCheckSum;  
} 
```

`4dd466d0e1e0cc123afbe504da2462ca`
