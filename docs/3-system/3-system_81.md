

# 查看 PE 文件重定位表---C 源码

> 原文：[`www.pediy.com/kssd/pediy12/132257.html`](https://www.pediy.com/kssd/pediy12/132257.html)

正在学习 PE 文件结构相关知识，学习到重定位表时使用 PEDIY 想查看某文件的重定位表，却发现貌似没有这项功能，正好就动手锻炼一下自己，也便于理解学到的东西，在此分享下……  
注：控制台程序，显示的同时将重定位表写入到文件：PE 文件名+.RelocTable.Log  
另外请教如何写出体积较小的窗口程序，学习了 MFC，写出的窗口程序必须打包库文件，一打包就得 2M 多……  
  
源码如下（编译环境：Win 7+VS2005）：  
(如有不妥之处，望高手指出，不胜感激……![](img/906f120cfe711ccbc8463fd6eb86eee9.jpg "尴尬")）  

代码:

```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <windows.h>


#define PE\_HEAD\_OFFSET        0x3C    //相对文件头
#define IMAGE\_SETION\_HEAD\_OFFSET  0xF8    //相对 PE 头
#define IMAGE\_SETION\_NUMBER\_OFFSET  0x06    //相对 PE 头

int IsPE(FILE \*ExeFile)
{
  char MZ\[2\]={0};
  fseek(ExeFile,0,0);
  fread(MZ,sizeof(char),2,ExeFile);
  if(MZ\[0\]=='M' && MZ\[1\]=='Z')
    return 1;
  return 0;
}

int IsMemoryZero(void \*Buffer,int MemorySize)  //不全为 0 返回假，全为 0 返回真
{
  char \*TempBuffer=(char \*)Buffer;
  int i;
  for(i=0;i<MemorySize;i++)
  {
    if(TempBuffer\[i\]!=0)
      break;
  }
  return !(i<MemorySize);
}

int GetPEHeadAddr(FILE \*ExeFile)
{
  fseek(ExeFile,PE\_HEAD\_OFFSET,0);
  int PEHeadOffset=0;
  fread(&PEHeadOffset,sizeof(int),1,ExeFile);
  return PEHeadOffset;
}

int RVAToOffset(int RVA,FILE \*ExeFile)
{
  int PEHeadOffset=GetPEHeadAddr(ExeFile);
  IMAGE\_SECTION\_HEADER ImgSetHead={0};
  WORD NumberOfSetions=0;
  fseek(ExeFile,PEHeadOffset+IMAGE\_SETION\_NUMBER\_OFFSET,0);
  fread(&NumberOfSetions,sizeof(WORD),1,ExeFile);
  fseek(ExeFile,IMAGE\_SETION\_HEAD\_OFFSET+PEHeadOffset,0);
  int i;
  for(i=0;i<NumberOfSetions;i++)
  {
    fread(&ImgSetHead,sizeof(IMAGE\_SECTION\_HEADER),1,ExeFile);
    if(ImgSetHead.VirtualAddress<=RVA && RVA<ImgSetHead.VirtualAddress+ImgSetHead.SizeOfRawData)
      break;
  }
  if(i>=NumberOfSetions)
    return 0;
  return ImgSetHead.PointerToRawData-ImgSetHead.VirtualAddress+RVA;
}

int OffsetToRVA(int Offset,FILE \*ExeFile)
{
  int PEHeadOffset=GetPEHeadAddr(ExeFile);
  IMAGE\_SECTION\_HEADER ImgSetHead={0};
  WORD NumberOfSetions=0;
  fseek(ExeFile,PEHeadOffset+IMAGE\_SETION\_NUMBER\_OFFSET,0);
  fread(&NumberOfSetions,sizeof(WORD),1,ExeFile);
  fseek(ExeFile,IMAGE\_SETION\_HEAD\_OFFSET+PEHeadOffset,0);
  int i;
  for(i=0;i<NumberOfSetions;i++)
  {
    fread(&ImgSetHead,sizeof(IMAGE\_SECTION\_HEADER),1,ExeFile);
    if(ImgSetHead.PointerToRawData<=Offset && Offset<ImgSetHead.PointerToRawData+ImgSetHead.SizeOfRawData)
      break;
  }
  if(i>=NumberOfSetions)
    return 0;
  return ImgSetHead.VirtualAddress+Offset-ImgSetHead.PointerToRawData;
}

void GetImageOption(FILE \*ExeFile,IMAGE\_OPTIONAL\_HEADER32 \*ImgOpt)
{
  int PEHead=GetPEHeadAddr(ExeFile);
  fseek(ExeFile,PEHead+0x18,0);
  fread(ImgOpt,sizeof(IMAGE\_OPTIONAL\_HEADER32),1,ExeFile);
}

void ShowRelocTable(char \*ExeName,char \*LogName)
{
  FILE \*ExeFile=fopen(ExeName,"rb+");
  if(!IsPE(ExeFile))
  {
    printf("不是有效的 PE 文件！\\n");
    return;
  }

  FILE \*LogFile=fopen(LogName,"w+");
  fseek(LogFile,0,0);
  IMAGE\_OPTIONAL\_HEADER32 ImgOpt={0};
  GetImageOption(ExeFile,&ImgOpt);
  int RelocTableAddr=RVAToOffset(ImgOpt.DataDirectory\[5\].VirtualAddress,ExeFile);
  IMAGE\_BASE\_RELOCATION ImgRelTab={0};
  fseek(ExeFile,RelocTableAddr,0);
  fread(&ImgRelTab,sizeof(IMAGE\_BASE\_RELOCATION),1,ExeFile);
  while(!IsMemoryZero(&ImgRelTab,sizeof(IMAGE\_BASE\_RELOCATION)))
  {
    int n=(ImgRelTab.SizeOfBlock-sizeof(IMAGE\_BASE\_RELOCATION))/2;
    printf("重定位页面起始地址：%X，本项重定位块长度：%X，本项重定位条目：%d\\n",ImgRelTab.VirtualAddress,ImgRelTab.SizeOfBlock,n);
    fprintf(LogFile,"重定位页面起始地址：%X，本项重定位块长度：%X，本项重定位条目：%d\\n",ImgRelTab.VirtualAddress,ImgRelTab.SizeOfBlock,n);
    for(int i=0;i<n;i++)
    {
      WORD RelData=0;
      fread(&RelData,sizeof(WORD),1,ExeFile);
      printf("高 4 位标志：%d，\\t 低 12 位偏移：%4X，\\t 重定位位置：%8X\\n",(RelData & 0xF000)/0x1000,RelData & 0x0FFF,ImgRelTab.VirtualAddress+(RelData & 0x0FFF));
      fprintf(LogFile,"高 4 位标志：%d，\\t 低 12 位偏移：%4X，\\t 重定位位置：%8X\\n",(RelData & 0xF000)/0x1000,RelData & 0x0FFF,ImgRelTab.VirtualAddress+(RelData & 0x0FFF));
    }
    //printf("本项重定位表已结束，按任意键查看下一项……\\n\\n");
    //getchar();
    fread(&ImgRelTab,sizeof(IMAGE\_BASE\_RELOCATION),1,ExeFile);
  }
  fclose(LogFile);
  fclose(ExeFile);
  printf("已将重定位表写入日志文件：%s，按任意键退出……",LogName);
}

void main()
{
  char ExeName\[100\]={0};
  char LogName\[100\]={0};
  printf("请输入要查询的 EXE 文件名：\\n");
  scanf("%s",ExeName);
  strcpy(LogName,ExeName);
  strcat(LogName,".RelocTable.Log");
  getchar();
  ShowRelocTable(ExeName,LogName);
  getchar();
}

```