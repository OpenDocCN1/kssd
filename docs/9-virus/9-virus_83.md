

# 检测 Kaspersky 沙盒之 OpenProcess 大法

> 原文：[`www.pediy.com/kssd/pediy11/106514.html`](https://www.pediy.com/kssd/pediy11/106514.html)

最近看到不少人讨论如何检测程序是否运行于 Kaspersky 的沙盒中。  
  
有的人想到 Sleep(XXXXXXXX);；有的人想到访问一个不存在的网络地址然后根据结果判断；有的人想到通过复杂运算折腾死虚拟机。。。 。。。  
  
花样层出不穷，其实这些方法非常山寨，而且还不稳定、不准确。  
  
下面贴出我的源代码~~~  
  
//  
//AUTHOR:黑客守卫者  
//BLOG:http://hi.baidu.com/ihxdef  
//url:http://hi.baidu.com/ihxdef/blog/item/87e8a2a62f535d9ed043585e.html  
//  
  
#include <windows.h>  
#include <stdio.h>  
#include <tlhelp32.h>  
  
//  
//Define  
//  
int DetectSandBox(void);  
  
//  
//Routine  
//  
int DetectSandBox(void)  
{  
   //  
   //Routine Description:  
   //  
   //This routine detect if is run in real OS or SandBox.  
   //Tested in win xp.  
   //Not for win Vista or later version  
  
   //  
   //Arguments:  
   //  
   //None  
  
   //  
   //Return Value:  
   //  
   // -1 for error  
   // 0 for run in real OS  
   // 1 for run in SandBox  
  
   //  
   //Adjust Token Privileges  
   //  
    HANDLE hToken = NULL;  
    LUID sedebugnameValue;  
    TOKEN\_PRIVILEGES tkp;  
  
    if(!OpenProcessToken(GetCurrentProcess(),TOKEN\_ADJUST\_PRIVILEGES | TOKEN\_QUERY, &hToken))  
   {  
        return -1;  
    }  
    if(!LookupPrivilegeValue(NULL, SE\_DEBUG\_NAME, &sedebugnameValue))  
   {  
        CloseHandle(hToken);  
        return -1;  
    }  
    tkp.PrivilegeCount = 1;  
    tkp.Privileges\[0\].Luid = sedebugnameValue;  
    tkp.Privileges\[0\].Attributes = SE\_PRIVILEGE\_ENABLED;  
    if (!AdjustTokenPrivileges(hToken, FALSE, &tkp, sizeof(tkp), NULL, NULL))  
   {  
        CloseHandle(hToken);  
        return -1;  
    }  
  
   //  
   //Detect SandBox  
   //  
   DWORD dwProcCount = 0;  
   DWORD dwFaultCount = 0;  
  
   HANDLE hSnap=CreateToolhelp32Snapshot(TH32CS\_SNAPPROCESS,0);  
   PROCESSENTRY32 pe;  
   pe.dwSize=sizeof(PROCESSENTRY32);  
  
   if(hSnap)  
   {  
      Process32First(hSnap,&pe);  
      do  
      {  
         if(  
            pe.th32ProcessID == GetCurrentProcessId() ||  
            pe.th32ProcessID == 0 ||  
            pe.th32ProcessID == 4  
         )  
         {  
            continue;  
         }  
  
         HANDLE hProc = NULL;  
         hProc = OpenProcess( PROCESS\_CREATE\_THREAD|PROCESS\_VM\_OPERATION|PROCESS\_VM\_WRITE,FALSE,pe.th32ProcessID);        
         if( !hProc )  
         {  
            dwFaultCount++;  
         }  
         CloseHandle(hProc);  
  
         dwProcCount++;  
      }  
      while(Process32Next(hSnap,&pe));  
      CloseHandle(hSnap);  
   }  
   else  
   {  
      return -1;  
   }  
  
   //  
   //Check the result  
   //  
   if( (dwProcCount - dwFaultCount) <= 4 )  
   {  
      return 1;  
   }  
   else  
   {  
      return 0;  
   }  
  
   return -1;  
}  
  
//  
//Entry  
//  
int main(void)  
{  
   int iRet = DetectSandBox();  
   if( iRet == 1 )  
   {  
      MessageBox(NULL,"RUN IN SANDBOX! DAMN IT!","NOTICE",MB\_ICONSTOP);  
   }  
   else  
   if( iRet == 0 )  
   {  
      MessageBox(NULL,"RUN IN REAL OS!","NOTICE",MB\_ICONINFORMATION);  
   }  
   else  
   {  
      MessageBox(NULL,"UNKNOWN ERROR! DAMN IT!","NOTICE",MB\_ICONSTOP);  
   }  
  
   return 0;  
}