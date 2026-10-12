
# 检测 Kaspersky 沙盒之 `DeleteFile` 大法

> 原文：[`www.pediy.com/kssd/pediy11/106552.html`](https://www.pediy.com/kssd/pediy11/106552.html)

继前一篇的 `OpenProcess` 大法，上一篇的 `RegSetValue` 大法，现在放出 `DeleteFile` 大法。

本例中的 `DeleteFile` 可以换成 `CreateFile`、`SetFileAttributes` 等等。。。原理相同，请自由发挥。。。

```c
//
//AUTHOR: 黑客守卫者
//BLOG: http://hi.baidu.com/ihxdef
//URL: http://hi.baidu.com/ihxdef/blog/item/a7e259391475d8c9d562259b.html
//

#include <windows.h>
#include <stdio.h>
#include <tlhelp32.h>

//
//Define
//
int DetectSandBox(void);

//
//Routine
//
int DetectSandBox(void)
{
   //
   //Routine Description:
   //
   //This routine detect if is run in real OS or SandBox.
   //
   //Arguments:
   //
   //None
   //
   //Return Value:
   //
   // -1 for error
   // 0 for run in real OS
   // 1 for run in SandBox
   //
   //Detect
   //
   char strCurrentFile[MAX_PATH] = {0};
   GetModuleFileName(NULL, strCurrentFile, MAX_PATH);
   BOOL bRet = FALSE;
   bRet = DeleteFile(strCurrentFile);
   if( bRet == TRUE )
   {
      return 1;
   }
   else
   {
      return 0;
   }
   return -1;
}

//
//Entry
//
int main(void)
{
   int iRet = DetectSandBox();
   if( iRet == 1 )
   {
      MessageBox(NULL, "RUN IN SANDBOX! DAMN IT!", "NOTICE", MB_ICONSTOP);
   }
   else
   if( iRet == 0 )
   {
      MessageBox(NULL, "RUN IN REAL OS!", "NOTICE", MB_ICONINFORMATION);
   }
   else
   {
      MessageBox(NULL, "UNKNOWN ERROR! DAMN IT!", "NOTICE", MB_ICONSTOP);
   }
   return 0;
}
```
