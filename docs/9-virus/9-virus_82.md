

# 检测 Kaspersky 沙盒之 RegSetValue 大法

> 原文：[`www.pediy.com/kssd/pediy11/106548.html`](https://www.pediy.com/kssd/pediy11/106548.html)

继上一篇的 OpenProcess 大法之后，现在放出 RegSetValue 大法。  
  
其实这些检测方法的挖掘是建立在对沙盒工作原理的了解之上，熟悉它的工作流程，知道哪里是处理难点。我认为那种完全的盲人摸象，在对原理不甚明了就胡乱测试的方法不可取。  
  
检测方法可以说是无穷无尽的~~~只要是假的就真不了~~~  
  
下面是源代码：  
  
//  
//AUTHOR:黑客守卫者  
//BLOG:http://hi.baidu.com/ihxdef  
//URL:http://hi.baidu.com/ihxdef/blog/item/0f880bf94285c704d8f9fdc7.html  
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
   // 2 for Kaspersky not installed  
  
   //  
   //Detect  
   //  
   char szKasperskyPath\[256\] = {0};  
   lstrcpy(szKasperskyPath, "SOFTWARE\\\\KasperskyLab\\\\protected");  
  
   HKEY hKaspersky = NULL;  
  
   if( RegOpenKey(HKEY\_LOCAL\_MACHINE,szKasperskyPath,&hKaspersky) != ERROR\_SUCCESS )  
   {  
      RegCloseKey(hKaspersky);  
      return 2;  
   }  
   else  
   {  
      LONG lRet = 0;  
      lRet = RegSetValue(hKaspersky,"Kaspersky",REG\_SZ,"SandBox",sizeof("SandBox"));  
  
      //  
      //Check the result  
      //  
      if( lRet == ERROR\_SUCCESS )  
      {  
         RegCloseKey(hKaspersky);  
         return 1;  
      }  
      else  
      {  
         RegCloseKey(hKaspersky);  
         return 0;  
      }  
   }  
   RegCloseKey(hKaspersky);  
  
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
   if( iRet == 2 )  
   {  
      MessageBox(NULL,"KASPERSKY NOT INSTALLED!","NOTICE",MB\_ICONSTOP);  
   }  
   else  
   {  
      MessageBox(NULL,"UNKNOWN ERROR! DAMN IT!","NOTICE",MB\_ICONSTOP);  
   }  
  
   return 0;  
}