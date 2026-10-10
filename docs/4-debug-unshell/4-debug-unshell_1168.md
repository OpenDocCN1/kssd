

# vprotector

> 原文：[`www.pediy.com/kssd/pediy06/pediy6914.htm`](https://www.pediy.com/kssd/pediy06/pediy6914.htm)

vcasm 的 vprotector 程序中使用的多进程（Process）反跟踪技术很有新意，反跟踪技术中比较常见的是多线程（Thread）技术。我以前见到的多进程主要是使用一个进程调试另一个进程。这种方式只能认为是调试反跟踪，不能称为真正的多进程反跟踪技术。而 vcasm 的方法本人认为是真正意义上的多进程反跟踪技术。这里简单说明一下。  
要想利用多进程，必须处理好多个进程之间的数据交换问题和进程同步问题。如果各进程之间没有数据交换就不成其为多进程技术了，谈到进程间的数据交换，我们首先想到的是动态连接库（dll），但 vcasm 采用了一种更为简便的方法。下面就说说这种方法。  
在谈 vcasm 的数据交换的方法之前先看几个 API 函数。  
CreateFileMapping 函数用来生成一个内存文件映射对象，函数定义如下  
HANDLE CreateFileMapping(  
  HANDLE hFile,              // 映射文件句柄  
  LPSECURITY\_ATTRIBUTES lpFileMappingAttributes,  
                             // 安全属性  
  DWORD flProtect,           // 对象的保护属性  
  DWORD dwMaximumSizeHigh,   // 对象大小高 32 位  
  DWORD dwMaximumSizeLow,    //对象大小第 32 位 e  
  LPCTSTR lpName             // 文件映射对象名  
);  
当 hFile=（HANDLE）-1 时得到的不是实际文件的映射对象，而是一个操作系统分页文件返回的一个特定大小的内存块对象。这个内存块对象可以被多个进程用对象名进行操作。这就是说，可以用它来交换数据了。判断这个对象是否存在呢？这又要用到一个函数。  
DWORD GetLastError(VOID)  
当返回值等于 ERROR\_ALREADY\_EXISTS 时则说明这个对象已经存在了。  
好了，下面就用程序实际说明一下，vcasm 的程序比较复杂，我并没按原程序写代码，而是简化成了一段说明方法的代码，目的只想说明一下这种方法。线程同步的代码这里就不写了,  
有兴趣自己逆向一下。  
  
  GetModuleFileName(0,lpFilename,0x200); //取得文件名  
   然后将文件名加以变动作为 mapview 的名  
for(i=0;i<0x200;i++)  
  {  
  //  将文件名中的\\变成-  
    if(\*(char\*)(lpFilename+i)==0x5c)\*(char\*)(lpFilename+i)=0x2d;  
  }  
mHandle=CreateFileMapping((HANDLE)-1,0,PAGE\_READWRITE,0,0x400,lpFilename);  
GetLastError()必须紧跟在后面,如果中间还有其他的 api,他返回的就不知道是什么了.  
if(GetLastError()!=ERROR\_ALREADY\_EXISTS)  
  {  
    mapview=MapViewOfFile(mHandle,FILE\_MAP\_ALL\_ACCESS,0,0,0); //提交物理内存  
        sum =(char\*)((char\*)mapview+0x3ff);  
        \*sum=0;                                        //初始化累加值  
  }  
  else  
  {  
    mapview=MapViewOfFile(mHandle,FILE\_MAP\_ALL\_ACCESS,0,0,0); //提交物理内存  
        sum =(char\*)((char\*)mapview+0x3ff);  
        \*sum+=1;                                        //累加值+1  
  }  
  
   if(\*sum>5)     //大于 5 次就该结束了  
   {  
     MessageBox(0,"yes","ok",0);  
     UnmapViewOfFile(mapview);   打开的 fileview 可以不要了  
     CloseHandle(mHandle);        handle 也可以关掉了  
   }  
   else  
   {  
    还没到五次,再开一个进程  
    CreateProcess(0,GetCommandLine(),0,0,0,0,0,0,&StartupInfo,&ProcessInformation);  
   }  
   Sleep(50);    将执行权交出去,休息一下  
   ExitProcess(0);  其他的进程已经打开了,本进程就可以关掉了.  
  
vc++的代码如下：  
#include "stdafx.h"  
  
int APIENTRY WinMain(HINSTANCE hInstance,  
                     HINSTANCE hPrevInstance,  
                     LPSTR     lpCmdLine,  
                     int       nCmdShow)  
{  
     
  DWORD i;  
  LPVOID mapview;  
  HANDLE mHandle;  
  PCHAR  sum;  
  LPTSTR lpFilename=(LPTSTR)malloc(0x200);  
  STARTUPINFO StartupInfo;  
  StartupInfo.cb=sizeof(STARTUPINFO);  
  PROCESS\_INFORMATION ProcessInformation;    
  GetModuleFileName(0,lpFilename,0x200); //取得文件名  
  GetStartupInfo(&StartupInfo); //填写 StartupInfo 结构  
    for(i=0;i<0x200;i++)  
  {  
  //  将文件名中的\\变成-  
    if(\*(char\*)(lpFilename+i)==0x5c)\*(char\*)(lpFilename+i)=0x2d;  
  }  
    mHandle=CreateFileMapping((HANDLE)-1,0,PAGE\_READWRITE,0,0x400,lpFilename);  
  if(GetLastError()!=ERROR\_ALREADY\_EXISTS)  
  {  
    mapview=MapViewOfFile(mHandle,FILE\_MAP\_ALL\_ACCESS,0,0,0); //提交物理内存  
        sum =(char\*)((char\*)mapview+0x3ff);  
        \*sum=0;                                        //初始化累加值  
  }  
  else  
  {  
    mapview=MapViewOfFile(mHandle,FILE\_MAP\_ALL\_ACCESS,0,0,0); //提交物理内存  
        sum =(char\*)((char\*)mapview+0x3ff);  
        \*sum+=1;                                        //累加值+1  
  }  
  
   if(\*sum>5)  
   {  
     MessageBox(0,"yes","ok",0);  
     UnmapViewOfFile(mapview);  
     CloseHandle(mHandle);  
   }  
   else  
   {  
      
    CreateProcess(0,GetCommandLine(),0,0,0,0,0,0,&StartupInfo,&ProcessInformation);  
   }  
   Sleep(0);  
   ExitProcess(0);  
  
  return 0;  
}

 4dd466d0e1e0cc123afbe504da2462ca