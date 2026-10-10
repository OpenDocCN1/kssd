

# 使用 GetStartupInfo 检查自己是否被"调试"

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-288.htm`](https://www.pediy.com/kssd/pediy08/pediy8-288.htm)

在使用 CreateProcess 创建进程时,需要传递   
STARTUPINFO 的结构的指针,  
而常常我们并不会一个一个设置其结构的值,  
连把其他不用的值清 0 都会忽略,  
而 ollydbg 也这样做了,  
我们可以使用 GetStartupInfo 检查启动信息,  
如果很多值为"不可理解"的,那么就说明自己不是由 explorer 来创建的.(explorer.exe 使用 shell32 中 ShellExecute 的来运行程序, ShellExecute 会清不用的值)  
  
还有一点 ollydbg 会向 STARTUPINFO 中的   dwFlags 设置 STARTF\_FORCEOFFFEEDBACK,而 explorer 不会  
  
![](img/9e41176b4900ac914e77ca870db1e3dd.jpg)  
  
////////////////////////  
//ex  
  
#include <windows.h>  
#include <stdio.h>  
  
#pragma comment(linker, "/subsystem:windows /entry:main")  
  
int main()  
{  
  STARTUPINFO si;  
    
  GetStartupInfo(&si);  
  
  if (   
    (si.dwX != 0) ||  
    (si.dwY != 0) ||  
    (si.dwXCountChars != 0) ||  
    (si.dwYCountChars != 0) ||  
    (si.dwFillAttribute != 0) ||  
    (si.dwXSize != 0) ||  
    (si.dwYSize != 0) ||  
    (si.dwFlags & STARTF\_FORCEOFFFEEDBACK)  
    )  
  {  
    MessageBox(NULL, "found debugger!", NULL, 0);  
  }  
  else  
  {  
    MessageBox(NULL, "no found debugger!", NULL, 0);  
  }  
    
  return 0;  
}  

 4dd466d0e1e0cc123afbe504da2462ca