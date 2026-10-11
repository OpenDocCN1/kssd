
# PE 结构中插入一段补丁程序 (C 语言版)

> 原文：[`www.pediy.com/kssd/pediy12/143612.html`](https://www.pediy.com/kssd/pediy12/143612.html)

前些天我发了一篇帖子，《PE 结构中插入一段补丁程序》，有人说那个用汇编写的，可读性太差了，所以今天我重新写了这篇帖子，把原来的程序改成了 C 语言，程序中有详尽的注释，可为那些想学习写 PE 补丁程序的人们一些参考，当然鄙人是个菜鸟，如有不妥指出，还请高人指点指点。

源代码如下：

```c
/*
 *   本程序的功能:在 PE 空隙中插入一小段补丁程序，PE 被加载后，补丁程序先于原程序运行，后将执行权转移给原程序
 *   补丁在 PE 结构中的大致位置:
 *                   --------------------------------
 *                   |             DOS 头           |
 *                   --------------------------------
 *                   |             DOS Stub         |
 *                   --------------------------------
 *                   |             补丁程序         |
 *                   --------------------------------
 *                   |             PE 头            |
 *                   --------------------------------
 *                   |             节表             |
 *                   --------------------------------
 *                   |              节              |
 *                   --------------------------------
 *
 *   注意:为了使程序字节变得更小，在这里补丁程序和 DOS Stub 部分是重合的
 *        本程序在 Windows XP SP2 VC6.0 下编译通过
 *        请在测试本程序时关闭 360 或者将其添加为信任,否则会给您的测试带来不便
 */

#include <windows.h>

#include <stdio.h>

char cObjectFile[]="C:\\PE\\PE.exe";

char cSaveFile[]="C:\\PE\\newPE.exe";

char *szFileBuffer=NULL;

char *szOldFile=NULL;

//实用补丁程序字节码

DWORD dwPatchSize;

char* cPatch;

/*
 *    请将补丁字节码粘贴在这里,后面跟随若干个 90,请确保该_cPatch 的缓冲区比较小,
 *    如果缓冲区过大，可能会使补丁程序无法正常跳转到原程序入口,
 *    或者使补丁不能正常插入到原程序中
 *    建议缓冲区大小为 8 的倍数
 */

//待用补丁程序字节码

DWORD _dwPatchSize=192;//192=12*16

char* _cPatch=new char[_dwPatchSize]=//本补丁的功能是弹出一个 cmd,其默认 msvcrt.dll 已被加载

"\x55"                                     
"\x8b\xec"                                 
"\x33\xff"                                 
"\x57"                                      
"\x83\xec\x08"                             
"\xc6\x45\xfe\x6d"                         
"\xc6\x45\xfd\x6f"                         
"\xc6\x45\xfc\x63"                         
"\xc6\x45\xfb\x2e"                         
"\xc6\x45\xfa\x64"                         
"\xc6\x45\xf9\x6e"                         
"\xc6\x45\xf8\x61"                         
"\xc6\x45\xf7\x6d"                         
"\xc6\x45\xf6\x6d"                         
"\xc6\x45\xf5\x6f"                         
"\xc6\x45\xf4\x63"                         
"\x8d\x45\xf4"                             
"\x50"                                      
"\xb8\xc7\x93\xbf\x77"                     
"\xff\xd0"                                  
"\x90\x90\x90\x90\x90\x90\x90\x90"        
"\x90\x90\x90\x90\x90\x90\x90\x90"        
"\x90\x90\x90\x90\x90\x90\x90\x90"        
"\x90\x90\x90\x90\x90\x90\x90\x90"        
"\x90\x90\x90\x90\x90\x90\x90\x90"        
"\x90\x90\x90\x90\x90\x90\x90\x90"        
"\x90\x90\x90\x90\x90\x90\x90\x90"        
"\x90\x90\x90\x90\x90\x90\x90\x90"        
"\x90\x90\x90\x90\x90\x90\x90\x90"        
"\x90\x90\x90\x90\x90\x90\x90\x90"        
"\x90\x90\x90\x90\x90\x90\x90\x90"        
"\x90\x90\x90\x90\x90\x90\x90\x90"        
"\x90\x90\x90\x90\x90\x90\x90\x90"        
"\x90\x90\x90\x90\x90\x90\x90\x90"        
"\x90\x90\x90\x90\x90\x90\x90\x90"        
"\x90\x90\x90\x90\x90\x90\x90\x90";

DWORD ddPatchRVA;//补丁起始地址 RVA

DWORD ddPatchFOA;//补丁文件 FOA

////////////////////////////
//PE 信息

DWORD dwe_lfanew=0x3c;//常量

DWORD ddPEHeaderSize=248;//PE 头的大小-常量

DWORD dwSectionHeaderFileOffset=20;//节表头到文件偏移项的偏移-常量

DWORD dwSectionHeaderSize=40;//一个节表项的大小-常量

DWORD ddHeaderAndSectionSize;//头+节表在文件中总大小:DOS 头+PE 头+节表

LONG e_lfanew=0;

DWORD dwDosSize=0;//DOS 头和 DOS Stub 的总大小

DWORD ddAddressOfEntryPoint;//程序执行入口 RVA

WORD wNumberOfSection;//PE 中节的数量

DWORD dwSectionTableStart;//节表起始处的 FOA

DWORD dwDataDirectorySize=0;//数据目录数组的总大小

DWORD ddNumberOfRvaAndSizes;//数据目录项目个数

DWORD ddSegmentOffset;//节偏移

DWORD ddSectionAlignment;//内存中节的对齐粒度

DWORD ddFileAlignment;//文件中节的对齐粒度

char* ddtextName=new char[6];//.text 节的名字

DWORD ddtextSizeOfRawData;//.text 节的在文件中对齐后的尺寸

DWORD ddtextFileOffset;//.text 节在文件中的偏移

DWORD ddtextRVA;//.text 节的 RVA

DWORD ddAddressOfEntryPointXX;//原始程序入口 RVA 与新入口 RVA 跳转时的 XX

BYTE byE9=233;//长跳转指令字节码 E9 XXXXXXXX

//用 C 语言编写在 PE 间隙中插入程序

void ModifyExe_C(char* file)

{

  //首先计算实用补丁程序大小和待用补丁大小程序的差异，如果差异过大，则不允许插入补丁程序

  if(_dwPatchSize>dwPatchSize)

  {

    printf("不允许插入补丁程序\n");

    return ;

  }

  else

  {

      cPatch=new char[dwPatchSize];

    memset(cPatch,144,dwPatchSize);//144=90h

    memcpy(cPatch,_cPatch,_dwPatchSize);

  }

  HANDLE h;

  DWORD readed;

  h=CreateFile(file,GENERIC_READ | GENERIC_WRITE,0,NULL,OPEN_ALWAYS,FILE_ATTRIBUTE_NORMAL,NULL);

  //获得文件的大小

  DWORD ddFileSize;

  ddFileSize=GetFileSize(h,NULL);

  //读取原始文件

  szOldFile=new char[ddFileSize+1];

  ReadFile(h,szOldFile,ddFileSize,&readed,NULL);

  //新文件的内容缓冲区

  szFileBuffer=new char[ddFileSize+dwPatchSize];

  //复制 DOS 头+PE 头+节表+多于部分

  memcpy(szFileBuffer,szOldFile,ddPatchFOA);

  //复制补丁程序

  memcpy(szFileBuffer+ddPatchFOA,cPatch,dwPatchSize);

  //插入剩余部分字节

  memcpy(szFileBuffer+ddPatchFOA+dwPatchSize,szOldFile+ddtextFileOffset,ddFileSize-ddtextFileOffset);

  PIMAGE_NT_HEADERS pImage_nt_headers=NULL;

  //修改程序执行入口 RVA-AddressOfEntryPoint

  pImage_nt_headers=(PIMAGE_NT_HEADERS)(szFileBuffer+e_lfanew);

  pImage_nt_headers->OptionalHeader.AddressOfEntryPoint = ddPatchFOA;

  //修改跳转到原始程序入口 RVA 的字节码 E9 XX

  void* address = (void *)(szFileBuffer+ddPatchFOA+dwPatchSize-5);//+sizeof(IMAGE_OPTIONAL_HEADER)+dwPatchSize-5;

  memcpy(address,&byE9,1);

  address = (void *)(szFileBuffer+ddPatchFOA+dwPatchSize-4);

  memcpy(address,&ddAddressOfEntryPointXX,4);

  //关闭文件

  CloseHandle(h);

  //写入新文件中

  h=CreateFile(cSaveFile,GENERIC_READ | GENERIC_WRITE,0,NULL,CREATE_ALWAYS,FILE_ATTRIBUTE_NORMAL,NULL);

  WriteFile(h,szFileBuffer,ddFileSize+dwPatchSize,&readed,NULL);

  CloseHandle(h);

}

//用 C 语言写的获取 PE 的信息

void GetPEInformation_C(char *file)

{

     HANDLE h;

  DWORD readed;

  h=CreateFile(file,GENERIC_READ | GENERIC_WRITE,0,NULL,OPEN_ALWAYS,FILE_ATTRIBUTE_NORMAL,NULL);

  //获得文件的大小

  DWORD dwFileSize;

  dwFileSize=GetFileSize(h,NULL);

  //读取原始文件

  szOldFile=new char[dwFileSize+1];

  ReadFile(h,szOldFile,dwFileSize,&readed,NULL);

  PIMAGE_DOS_HEADER   pImage_dos_header=NULL;

  PIMAGE_NT_HEADERS32 pImage_nt_headers32=NULL;

  PIMAGE_SECTION_HEADER pImage_section_Header=NULL;

  //计算 DOS 头和 DOS Stub 的总大小

  pImage_dos_header = (PIMAGE_DOS_HEADER)szOldFile;

  e_lfanew=pImage_dos_header->e_lfanew;

  //获得原始程序入口 RVA

  pImage_nt_headers32 = (PIMAGE_NT_HEADERS32)(szOldFile+e_lfanew);

  ddAddressOfEntryPoint = pImage_nt_headers32->OptionalHeader.AddressOfEntryPoint;

  //获得内存中节的对齐粒度

  ddSectionAlignment = pImage_nt_headers32->OptionalHeader.SectionAlignment;

  //获得文件中节的对齐粒度

  ddFileAlignment = pImage_nt_headers32->OptionalHeader.FileAlignment;

  //计算节偏移

  ddSegmentOffset = ddSectionAlignment - ddFileAlignment;

  //获得 PE 中节的数量

  wNumberOfSection = pImage_nt_headers32->FileHeader.NumberOfSections;

  //数据目录数组的目录项数一般为 16

  ddNumberOfRvaAndSizes = 16;

  //计算.text 节在文件的偏移

  //计算.text 节在文件中对齐后的尺寸

  //计算.text 节的 RVA

  char pName[IMAGE_SIZEOF_SHORT_NAME];

  for(int i=0;i<wNumberOfSection;i++)

  {

       pImage_section_Header = (PIMAGE_SECTION_HEADER)(szOldFile+e_lfanew+sizeof(IMAGE_NT_HEADERS32)+i*sizeof(IMAGE_SECTION_HEADER));

     memset(pName,0,IMAGE_SIZEOF_SHORT_NAME);

     memcpy(pName,pImage_section_Header->Name,IMAGE_SIZEOF_SHORT_NAME);

     if(strcmp(pName,".text")==0)

     {

       memcpy(ddtextName,pName,IMAGE_SIZEOF_SHORT_NAME);

       ddtextFileOffset = pImage_section_Header->PointerToRawData;

       ddtextSizeOfRawData = pImage_section_Header->SizeOfRawData;

       ddtextRVA = pImage_section_Header->VirtualAddress;

       break;

     }

  }

  //计算头+节表在文件中总大小:DOS 头+PE 头+节表
```
