

# 庆祝编程版块开张，发个简单的 loader

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-734.htm`](https://www.pediy.com/kssd/pediy07/pediy7-734.htm)

这是前两天写的一个小工具，用来替换 SoftICE 的 Loader 的功能，因为它在我的系统上总是不好用，郁闷，类似 PELord 的 Break Enter 的功能，原理是在入口点插入 int3 断点，从而实现入口处在 SoftICE 中断下。当然前提是你要先在 SoftICE 中设置断点 bpint 3. 有 bug 的话给我写信，目前不支持 dll 的导入，以后可能会考虑加上。  
  
  
#include<stdio.h>  
#include<windows.h>  
#include<winuser.h>  
  
#pragma comment(lib,"user32.lib")  
  
DWORD EntryPoint(LPVOID ImageBase)     //获得文件入口点的虚拟地址  
{  
  PIMAGE\_DOS\_HEADER pDH=NULL;  
  PIMAGE\_NT\_HEADERS pNtH=NULL;  
  PIMAGE\_OPTIONAL\_HEADER pOH=NULL;  
  if(!ImageBase)  
    return NULL;  
  pDH=(PIMAGE\_DOS\_HEADER)ImageBase;  
  if(pDH->e\_magic!=IMAGE\_DOS\_SIGNATURE)  
    return NULL;  
  pNtH=(PIMAGE\_NT\_HEADERS)((DWORD)pDH+pDH->e\_lfanew);  
  if(pNtH->Signature!=IMAGE\_NT\_SIGNATURE)  
    return NULL;  
  pOH=&pNtH->OptionalHeader;  
  return pOH->AddressOfEntryPoint+pOH->ImageBase;  
}  
  
int main(int argc, char \*argv\[\])  
{  
    BYTE  dbInt3=0xCC;    
  DWORD OldProtect;  
  BYTE  Ori;  
  DWORD ReadByte;  
  DWORD Entry;  
  HANDLE hfile, hMap, p\_MapFile;   
  char szBuffer\[100\]={0};  
  if(argc<=1)  
  {  
    printf("Useage: SIloader <FileName> \\nINT 3 Breakpoint Tool for SoftICE, Code by Marxixing@tom.com at 06/04/2005\\n");  
    return -1;  
  }  
  if((hfile = CreateFile(argv\[1\], GENERIC\_READ|GENERIC\_WRITE, FILE\_SHARE\_READ|FILE\_SHARE\_WRITE, NULL,\\  
                       OPEN\_EXISTING, FILE\_ATTRIBUTE\_NORMAL, NULL))==INVALID\_HANDLE\_VALUE)   
  {   
    printf("Can not open file, error code %d\\n",GetLastError());   
    ExitProcess(NULL);  
  }   
    
  if ((hMap = CreateFileMapping(hfile, NULL, PAGE\_READWRITE, 0, 0, NULL))==NULL)   
  {   
    printf("Can not create file mapping!\\n");   
    CloseHandle(hfile);  
  }   
    
  if ((p\_MapFile = MapViewOfFile(hMap, FILE\_MAP\_WRITE, 0, 0, 0))==NULL)   
  {   
    printf("Can not map file!\\n");   
    CloseHandle(hMap);  
  }  
    
  Entry=EntryPoint(p\_MapFile);  
  
  UnmapViewOfFile(p\_MapFile);  
  CloseHandle(hMap);  
  CloseHandle(hfile);  
    
  STARTUPINFO stStartUp;  
  PROCESS\_INFORMATION stProcInfo;  
  GetStartupInfo(&stStartUp);  
  if(!CreateProcess(argv\[1\],NULL,NULL,NULL,NULL,CREATE\_SUSPENDED,\\  
                    NULL,NULL,&stStartUp,&stProcInfo))   //进程创建时处于挂起状态  
  {  
    printf("CreateProcess fail! error code %d\\n",GetLastError());  
      ExitProcess(NULL);  
  }  
    
  if(ReadProcessMemory(stProcInfo.hProcess,(LPVOID)Entry,&Ori,1,&ReadByte)==NULL)  //读取入口点原来的内容  
    {  
    printf("ReadProcessMemory Function fail, error code %d\\n",GetLastError());  
    TerminateProcess(stProcInfo.hProcess,-1);  
  }  
  if(WriteProcessMemory(stProcInfo.hProcess,(LPVOID)Entry,&dbInt3,1,NULL)==NULL)   //写入 int3 断点  
    {  
    printf("WriteProcessMemory Function fail, error code %d\\n",GetLastError());  
    TerminateProcess(stProcInfo.hProcess,-1);  
  }  
  
  wsprintf(szBuffer,"%c\*\*\[SIloader\]\*\*\*\*\*\*\*\*\*%c\\n%c  Type \\"EB EIP %X\\"   %c\\n%c\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*%c\\n",201,187,186,Ori,186,200,188);  
  OutputDebugString(szBuffer);    //向调试器发出修改当前指令的信息  
  
  if(ResumeThread(stProcInfo.hThread)==-1)      //恢复进程的运行，会中断在 SoftICE 中  
  {  
    printf("ResumeThread Function fail, error code %d\\n",GetLastError());  
    TerminateProcess(stProcInfo.hProcess,-1);  
  }  
    
  CloseHandle(&stProcInfo.dwThreadId);  
  CloseHandle(&stProcInfo.dwProcessId);  
  
  return 1;  
}  
  
实际中发现了一些问题，修正一下。 

marxixing 兄是我灵感的源泉，他写过这种 loader，看的小弟一时兴起，也想做一个。顺便再回顾一下 PE 文件格式，以前看过好多遍只不过从没实践过，都忘的差不多了。  
，在不在？  
不要怪我盗窃你的思路哦![](img/710b5589904d0d2bf58a67748fca40e1.jpg)   
改天小弟给你倒茶赔罪行不？![](img/710b5589904d0d2bf58a67748fca40e1.jpg)   
  
好了，进入正题  
marxixing 兄做的 loader 是通过文件映射再创建挂起进程的方式实现的。  
我的这个纯粹是从 PE 文件中修改，通过一个个指针，我指指指。。。。  
  
有几个问题需要提一下：  
1）。在使用 fread( )后指针会后移。  
2）。程序对那些经过人工修改过的程序会失败，比如把区块名去掉（OH，NO。。）  
3）。由于本人菜，还没开发什么大的项目，所以对平台差异没考虑。。。  
     所以如果有这方面问题都反馈给 marxixing 兄吧，![](img/710b5589904d0d2bf58a67748fca40e1.jpg)  
  
程序很简单，我也加了一些注释，大家浏览一下就可以了。  
  
#include "stdafx.h"  
#include <iostream.h>  
#include <fstream.h>  
#include <windows.h>  
#include <stdio.h>  
#include <stdlib.h>  
  
int main(int argc, char \*argv\[\])  
{  
  if ( argc <= 1)  
    exit(0) ;  
  
  FILE \*file ;  
  if ( ( file = fopen( argv\[1\], "r+" ) ) == NULL )  
  {  
    cout << "ErrorCode:" << GetLastError() << endl ;  
    exit (0) ;  
  }  
  
  //检测“MZ”标志  
  char pDosHeadSign\[2\] ;  
  fread ( &pDosHeadSign, 1, 2, file ) ;  
  if ( !(pDosHeadSign\[0\] == 'M' && pDosHeadSign\[1\] == 'Z' ) )   
  {  
    cout << "Invaliable PE file!" << endl ;  
    fclose ( file ) ;  
    exit(0) ;  
  }  
  
  
  //定位到偏移文件头 0X3C  
  int result = fseek ( file, 0X3C, SEEK\_SET ) ;  
  if( result )  
  {  
    cout << "Seek file Error!" << endl ;  
    fclose ( file ) ;  
    exit (0) ;  
  }  
  
  
  //取得 PEHead 的偏移值  
  DWORD pPEHeadOffset ;  
  fread ( &pPEHeadOffset, sizeof(DWORD), 1, file ) ;  
  
    
  //定位到 PEHead，同时检测“PE”标志  
  char pPEHeadSign\[2\] ;  
  fseek ( file, pPEHeadOffset, SEEK\_SET ) ;  
  fread ( &pPEHeadSign, 2, 1, file ) ;  
  if ( !( pPEHeadSign\[0\] == 'P' && pPEHeadSign\[1\] == 'E' ) )  
  {  
    cout << "Invaliable PE file!" << endl ;  
    fclose ( file ) ;  
    exit(0) ;  
  }  
  
  //取得区块数目, 由于前面取 PE 标志时已经后移两个字节 ==> pPEHeadOffset + 0X6 - 0X2 )  
  WORD NumberOfSection ;  
  fseek ( file, 0x4, SEEK\_CUR ) ;  
  fread ( &NumberOfSection, sizeof(WORD), 1, file ) ;  
  cout << "NumberOfSection: " << NumberOfSection << endl ;  
  
  
  //取得 IMAGE\_OPENTION\_HEADER32 结构的大小（pPEHeadOffset + 0X14 ) - 0X6 -0X2 == 0XC  
  fseek ( file, 0XC, SEEK\_CUR ) ;  
  WORD SizeOfOptionalHeader ;  
  fread ( &SizeOfOptionalHeader, sizeof(WORD), 1, file ) ;  
  cout << "SizeOfOptionalHeader: 0X" << hex << SizeOfOptionalHeader << endl ;  
  
  
  //取得程序执行的入口地址（RVA）(后移 4 个字节）  
  //定位到 ( pPEHeadOffset + 0X28 ) -0X14 - sizeof(WORD) ==> 0X12  
  fseek ( file, 0X12, SEEK\_CUR ) ;  
  DWORD AddressOfEntryPoint ;  
  fread ( &AddressOfEntryPoint, sizeof(DWORD), 1, file ) ;  
  cout << "AddressOfEntryPoint(RVA):0X" << hex << AddressOfEntryPoint << endl ;  
  
  
  //定位到块表 ( pPEHeadOffset + SizeOfFileHead(0X18) + SizeOfOptionalHeader )   
  DWORD SectionTableOffset = pPEHeadOffset + 0X18 + SizeOfOptionalHeader ;  
  cout << "SectionTableOffset:0X" << hex << SectionTableOffset << endl ;  
  fseek ( file, SectionTableOffset, SEEK\_SET ) ;  
  
  char SectionName\[8\] = {0} ;  
  for ( int i = 0; i < NumberOfSection; i++ )  
  {  
    fread ( &SectionName, 8, 1, file ) ;  
  
    if ( !strcmp( SectionName, ".text" ) )  
    {  
      DWORD MemoryOffset, FileOffset, EntryPointFileOffset ;  
      fseek ( file, 4, SEEK\_CUR ) ;  
      fread ( &MemoryOffset, sizeof(DWORD), 1, file ) ;  
  
      fseek ( file, 4, SEEK\_CUR ) ;  
      fread ( &FileOffset, sizeof(DWORD), 1, file ) ;  
  
      EntryPointFileOffset = AddressOfEntryPoint - ( MemoryOffset - FileOffset ) ;  
      cout << "EntryPointFileOffset: 0X" << EntryPointFileOffset << endl ;  
  
      char EntryPointFirstByte ;  
      fseek ( file, EntryPointFileOffset, SEEK\_SET ) ;  
      fread ( &EntryPointFirstByte, sizeof(char), 1, file ) ;  
      cout << "EntryPointFirstByte:0X" << hex << (int)EntryPointFirstByte ;  
      cout << "----" << "You shoud keep this value in mind!" << endl ;  
  
      byte Int3 = 0xCC ;  
      fseek ( file, EntryPointFileOffset, SEEK\_SET ) ;  
      fwrite ( &Int3, sizeof(byte), 1, file ) ;  
      cout << "\\"Int3\\" insert success" << endl ;   
        
      break ;  
    }    
    else  
    {  
      fseek ( file, 0X20, SEEK\_CUR ) ;  
    }    
  }  
  
  fclose ( file ) ;  
  
  cout << "Press \\"ENTER\\" key to leave...." << endl ;  
  getchar();  
  
  return 0 ;  
}

 4dd466d0e1e0cc123afbe504da2462ca