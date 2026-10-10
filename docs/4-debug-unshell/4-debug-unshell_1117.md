

# UPX3.03 脱壳机-学习版\[代码更新\]

> 原文：[`www.pediy.com/kssd/pediy12/140312.html`](https://www.pediy.com/kssd/pediy12/140312.html)

代码更新：全 C 版，自动脱掉自己写的 MessageBox,解压算法使用了徐大力大牛逆向的解压算法，主要代码如下,这个头文件就是解压算法

代码:

```
#include <windows.h>
#include <stdlib.h>

#include "upx.h"

#ifndef \_IMAGEHLP\_H 
#include "imagehlp.h"
#pragma comment ( lib, "imagehlp.lib" )
#endif

IMAGE\_DOS\_HEADER \*DosHeader        = NULL;      //DOS 头
IMAGE\_NT\_HEADERS \*NtHeader         = NULL;      //NT 头
IMAGE\_FILE\_HEADER \*FileHeader      = NULL;    //文件头
IMAGE\_OPTIONAL\_HEADER32 \*OptHeader = NULL;  //可选头
IMAGE\_SECTION\_HEADER  \*SecHeader   =NULL;
IMAGE\_IMPORT\_DESCRIPTOR \*pIID      = NULL;    //数据目录
DWORD dwImportBase;
PIMAGE\_DOS\_HEADER  GetDosHeader(LPVOID ImageBase)
{
//////////////////////////////结构声明
  PIMAGE\_DOS\_HEADER  pDosHeader = NULL;
/////////////////////////////获得 DosHeader 结构
  pDosHeader = (PIMAGE\_DOS\_HEADER)ImageBase;
  return pDosHeader;
}
PIMAGE\_NT\_HEADERS  GetNtHeader(LPVOID  ImageBase) 
{
//////////////////////////////结构声明
  PIMAGE\_DOS\_HEADER  pDosHeader = NULL;
  PIMAGE\_NT\_HEADERS  pNtHeader  = NULL;
/////////////////////////////获得 NtHeader 结构
  pDosHeader = (PIMAGE\_DOS\_HEADER)ImageBase;
  pNtHeader = (PIMAGE\_NT\_HEADERS32)((DWORD)pDosHeader+pDosHeader->e\_lfanew);
  return    pNtHeader;
}

PIMAGE\_FILE\_HEADER GetFileHeader(LPVOID  ImageBase)
{
//////////////////////////////结构声明
  PIMAGE\_DOS\_HEADER  pDosHeader     = NULL;
  PIMAGE\_NT\_HEADERS  pNtHeader      = NULL;
  PIMAGE\_FILE\_HEADER  pFileHeader   = NULL;
/////////////////////////////获得 FileHeader 结构
  pDosHeader = (PIMAGE\_DOS\_HEADER)ImageBase;
  pNtHeader = (PIMAGE\_NT\_HEADERS32)((DWORD)pDosHeader+pDosHeader->e\_lfanew);
  pFileHeader =(PIMAGE\_FILE\_HEADER)&pNtHeader->FileHeader;
  return  pFileHeader;
}
PIMAGE\_OPTIONAL\_HEADER  GetOptionalHeader(LPVOID  ImageBase)
{
//////////////////////////////结构声明
  PIMAGE\_DOS\_HEADER  pDosHeader         = NULL;
  PIMAGE\_NT\_HEADERS  pNtHeader          = NULL;
  PIMAGE\_OPTIONAL\_HEADER  pOptionHeader = NULL;
/////////////////////////////获得 OptionalHeader 结构
  pDosHeader = (PIMAGE\_DOS\_HEADER)ImageBase;
  pNtHeader = (PIMAGE\_NT\_HEADERS32)((DWORD)pDosHeader+pDosHeader->e\_lfanew);
  pOptionHeader = (PIMAGE\_OPTIONAL\_HEADER32)&pNtHeader->OptionalHeader;
  return  pOptionHeader;
}
PIMAGE\_SECTION\_HEADER GetSectionHeader(LPVOID ImageBase)
{
//////////////////////////////结构声明
  PIMAGE\_DOS\_HEADER  pDosHeader         = NULL;
  PIMAGE\_NT\_HEADERS  pNtHeader          = NULL;
  PIMAGE\_OPTIONAL\_HEADER  pOptionHeader = NULL;
  PIMAGE\_FILE\_HEADER  pFileHeader       = NULL;
  PIMAGE\_SECTION\_HEADER pSecHeader      = NULL;
/////////////////////////////获得 SectionHeader 结构
  pDosHeader = (PIMAGE\_DOS\_HEADER)ImageBase;
  pNtHeader = (PIMAGE\_NT\_HEADERS32)((DWORD)pDosHeader+pDosHeader->e\_lfanew);
  pFileHeader =(PIMAGE\_FILE\_HEADER)&pNtHeader->FileHeader; 
  pOptionHeader = (PIMAGE\_OPTIONAL\_HEADER)&pNtHeader->OptionalHeader;
  pSecHeader =(PIMAGE\_SECTION\_HEADER)((LPBYTE)pOptionHeader + pFileHeader->SizeOfOptionalHeader);
  return pSecHeader;
}
PIMAGE\_IMPORT\_DESCRIPTOR  GetFirstImportDesc(LPVOID  ImageBase)
{
//////////////////////////////结构声明
  PIMAGE\_DOS\_HEADER  pDosHeader         = NULL;
  PIMAGE\_NT\_HEADERS  pNtHeader          = NULL;
  PIMAGE\_OPTIONAL\_HEADER  pOptionHeader = NULL;
  PIMAGE\_FILE\_HEADER  pFileHeader       = NULL;
  PIMAGE\_SECTION\_HEADER pSecHeader      = NULL;
/////////////////////////////获得 OptionalHeader 结构
  pDosHeader = (PIMAGE\_DOS\_HEADER)ImageBase;
  pNtHeader = (PIMAGE\_NT\_HEADERS32)((DWORD)pDosHeader+pDosHeader->e\_lfanew);
  pFileHeader =(PIMAGE\_FILE\_HEADER)&pNtHeader->FileHeader; 
  pOptionHeader = (PIMAGE\_OPTIONAL\_HEADER)&pNtHeader->OptionalHeader;  
  pSecHeader =(PIMAGE\_SECTION\_HEADER)((LPBYTE)pOptionHeader + pFileHeader->SizeOfOptionalHeader);
  DWORD pImportOffset = (pOptionHeader->DataDirectory\[1\]).VirtualAddress ;
  
  PIMAGE\_IMPORT\_DESCRIPTOR pImportDesc = (PIMAGE\_IMPORT\_DESCRIPTOR)
  ImageRvaToVa (pNtHeader,ImageBase,pImportOffset,&pSecHeader ) ; 
  return  pImportDesc;
}
////////////////////比较是否为 PE 文件
BOOL MemCmpPe(LPVOID ImageBase)
{
  PIMAGE\_DOS\_HEADER  pDosHeader =(PIMAGE\_DOS\_HEADER)ImageBase;
    PIMAGE\_NT\_HEADERS  pNtHeader  =(PIMAGE\_NT\_HEADERS32)((DWORD)pDosHeader+pDosHeader->e\_lfanew);
  while(!ImageBase)
    return FALSE;
  while(pDosHeader->e\_magic!=IMAGE\_DOS\_SIGNATURE)
    return FALSE;
  while(pNtHeader->Signature!=IMAGE\_NT\_SIGNATURE)
      return FALSE;
  return TRUE;
}
///////////////////////////读取文件到内存
LPVOID MapFileToMemory(char \*szFileName)
{
  HANDLE hFile = CreateFile(szFileName, GENERIC\_READ, 0, NULL, OPEN\_EXISTING, FILE\_ATTRIBUTE\_NORMAL, NULL);
  if (hFile==INVALID\_HANDLE\_VALUE)
  {
    MessageBox(NULL,"不能打开文件","提示",MB\_OK);
    return NULL;
  }
  HANDLE hMap = CreateFileMapping(hFile, NULL, PAGE\_READONLY | SEC\_COMMIT, 0, 0, NULL);
  LPVOID pMap = MapViewOfFile(hMap, FILE\_MAP\_READ, 0, 0, 0);
  return pMap;
}
///////////////////////////获得对齐值
DWORD Alignment(DWORD Size,DWORD Align)
{
  if(Size%Align!=0)
    return  (Size/Align+1)\*Align;
  return Size;
}
/////////////////////修复区段名字
void ModifySecName(PIMAGE\_SECTION\_HEADER SecHeader,DWORD SecNum)
{
  int i=0;
  while(i<SecNum)
  {
    strcpy((char\*)SecHeader\[i\].Name,"UnPack");
    i++;
  }
}
///////////////////修复输入表
void ModifyImport(DWORD SecOffset,DWORD SecOffsetA,DWORD SecOffsetB)
{
  DWORD SecOffsetC=SecOffset+SecOffsetA;
  DWORD ImPort    =\*(DWORD\*)SecOffsetC;
  while(ImPort)
  {
    DWORD FuncAddr  =\*(DWORD\*)(SecOffsetC+4);
    HMODULE hMod=LoadLibraryA((char\*)SecOffsetB+ImPort);
    FuncAddr+=SecOffset;
    SecOffsetC+=8;
    while(\*(BYTE\*)SecOffsetC!=0)
    {
        ++SecOffsetC;
        \*(DWORD\*)FuncAddr=(DWORD)GetProcAddress(hMod,(char\*)SecOffsetC);
        FuncAddr+=4;
      SecOffsetC+=strlen((char\*)SecOffsetC)+1;
    }
    ++SecOffsetC;
    ImPort=\*(DWORD\*)SecOffsetC;
  }
}
////////////用于处理 0xE8,0xE9//////////////C 语言表示///////////////有点问题
void ModifyCall(BYTE \*Start,int Size)
{
  BYTE DB;
  DWORD DDb;
  DWORD FixAddr=(DWORD)Start;
  WORD DWA;
  BYTE Hi8;
  WORD Lo8;
  while(Size)
  {
    DB=\*(BYTE\*)Start-0xE8;
    ++Start;
    if(DB>1||\*Start!=3)
    {
      continue;
    }
    DDb=\*(DWORD\*)Start;
    DWA=DDb&0x0ffff;
    DWA>>=8;
    DDb&=0xffff0000;
    DDb|=DWA;
    DDb=((DDb<<0x10)|(DDb>>0x10));
    DWA=DDb&0x0ffff;
    Lo8=((WORD)DWA<<8);
    Hi8=(BYTE)(DWA>>8);
    DWA=(Lo8+Hi8);
    DDb&=0xffff0000;
    DDb|=DWA;
    DDb-=(DWORD)Start;
    \*(DWORD\*)Start=DDb+FixAddr;
    Start+=4;
    --Size;
  }
}
///////////////////设置区段对齐
void SetSectionAlignment(IMAGE\_SECTION\_HEADER  \*TempSecHeader,int SectionNum,DWORD SecValue)
{
  int AlignValue=0;
    for(int i=0;i<SectionNum;i++)
    {
    AlignValue=Alignment(TempSecHeader->Misc.VirtualSize, SecValue);
    TempSecHeader->SizeOfRawData   =AlignValue;
        TempSecHeader->Misc.VirtualSize=AlignValue;    
    TempSecHeader->PointerToRawData=TempSecHeader->VirtualAddress;
    TempSecHeader++;
    } 
}
BYTE PCode;                                                         //压缩后的代码的文件指针
BYTE \*pImage;                                                       //新的映像空间
DWORD ImageSize;                                                    //加壳后的映像大小
int WINAPI WinMain(HINSTANCE hInstance,HINSTANCE hPrevInstance,LPSTR lpCmdLine,int nCmdShow)
{
  DWORD dwWrite;
  LPBYTE lpFile=(LPBYTE)MapFileToMemory("Test.exe");
  if(MemCmpPe(lpFile)==FALSE)
  {
    MessageBoxA(NULL,"PE 签名无效","提示",MB\_OK);
  }
  DosHeader    =GetDosHeader(lpFile);
  NtHeader     =GetNtHeader(lpFile);
  FileHeader   =GetFileHeader(lpFile);
  OptHeader    =GetOptionalHeader(lpFile);
  SecHeader    =GetSectionHeader(lpFile);
  pIID         =GetFirstImportDesc(lpFile);
/////////////////////处理第一个区段，并复制到内存中
  ImageSize=Alignment(OptHeader->SizeOfImage,OptHeader->SectionAlignment);//获得加壳后的映像大小，也就是虚拟空间大小
  pImage =VirtualAlloc(NULL,ImageSize,MEM\_COMMIT,PAGE\_READWRITE);         //分配新的 PE 映像空间
  memcpy(pImage,lpFile,OptHeader->SizeOfHeaders);                         //复制 PE 头到新分配的映像控件里面
  IMAGE\_SECTION\_HEADER  \*TempSecHeader   =SecHeader;                      //临时区段结构
////////////////////依次复制区段到新的 PE 的区段里面去
  for(int i=0;i<FileHeader->NumberOfSections;i++)                         //依次复制代码到新的映像空间里面
  {
    memcpy(pImage+TempSecHeader->VirtualAddress,                        //接收代码的映像地址
      lpFile+TempSecHeader->PointerToRawData,                         //复制代码的文件偏移地址
      Alignment(TempSecHeader->SizeOfRawData,OptHeader->FileAlignment)//复制代码大小以文件大小和对齐计算
      );                                         
    ++TempSecHeader;
  }
////////////////////修正 Image 里面的 PE 头
    IMAGE\_NT\_HEADERS \*NewNtHeader         = GetNtHeader(pImage);  //NT 头
    IMAGE\_SECTION\_HEADER  \*NewSecHeader   = GetSectionHeader(pImage);
  IMAGE\_OPTIONAL\_HEADER \*NewOptHeader    =GetOptionalHeader(pImage);
////////////////////设置入口点
    NewNtHeader->OptionalHeader.AddressOfEntryPoint=0x1020;
////////////////////文件对齐方式
    NewNtHeader->OptionalHeader.SizeOfImage   =Alignment(NewOptHeader->SizeOfImage,NewOptHeader->SectionAlignment);
    NewNtHeader->OptionalHeader.FileAlignment =NewOptHeader->FileAlignment;
    NewNtHeader->OptionalHeader.SizeOfHeaders =NewSecHeader->VirtualAddress; 
////////////////////修正区段位置和大小 
  SetSectionAlignment(NewSecHeader,NewNtHeader->FileHeader.NumberOfSections,0x1000);       //修复区段对齐
////////////////////开始解压代码
  UnCompress(pImage+SecHeader->VirtualAddress,(BYTE\*)lpFile+SecHeader->PointerToRawData);      //从只读内存中得压缩代码解压回映像空间
   //DisposureE8(pImage+SecHeader->VirtualAddress,0x84);                                      //现在处理 E8
  ModifyCall(pImage+SecHeader->VirtualAddress,0x84);
////////////////////修复输入表
    ModifyImport((DWORD)pImage+0x1000,0x9000,(DWORD)pImage+0xC000); 
////////////////////修改区段名字
    ModifySecName(GetSectionHeader(pImage),3);                                                              
///////////////////写入文件
  HANDLE Handle=CreateFile("UnpackMe.exe", GENERIC\_READ|GENERIC\_WRITE, 0, NULL, CREATE\_NEW, FILE\_ATTRIBUTE\_NORMAL, NULL);
  SetFilePointer(Handle,0,0,FILE\_BEGIN);
  WriteFile(Handle,pImage,ImageSize,&dwWrite,NULL);
  MessageBox(NULL,"脱壳成功","UnpackMe",MB\_OK);
  return TRUE;
}

```

上传的附件
| ![](img/620251b33676a8e4b98ca0ece915ee8d.jpg) || UnPackMe.zip  
 |