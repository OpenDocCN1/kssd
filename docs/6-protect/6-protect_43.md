

# 无 windows.h 的 Windows API 调用

> 原文：[`www.pediy.com/kssd/pediy12/143901.html`](https://www.pediy.com/kssd/pediy12/143901.html)

学校的 oj 系统禁止包含 windows.h，而又想用一些 API，  
在参考了看雪 Cryin 的帖子【原创】Win 7 下定位 kernel32.dll 基址及 shellcode 编写  
（http://bbs.pediy.com/showthread.php?t=122260）后，加上自己对 PE 结构的认识，写出了这几行代码，本着共享精神，特来献丑  
  
不解释，直接上代码  

代码:

```
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

//pe 常量
#define DOS\_HEADER\_SIZE          64
#define PE\_HEADER\_SIZE          248
#define SECTION\_SIZE          40
#define SECTION\_NAME\_SIZE        8
#define IED\_SIZE          40
#define SECOND\_DATA\_DIR\_LIST\_OFFSET            0x78

//用到的类型定义
typedef unsigned long       DWORD;
typedef unsigned int    UINT;
typedef char \*        LPSTR;
typedef const char \*    LPCSTR,\*LPCTSTR;
typedef void \*        HMODULE;
typedef void \*        HINSTANCE;
typedef void \*        HWND;
#define WINAPI              \_\_stdcall

//函数类型
typedef int        T\_GetKernelBase();
typedef void \*  WINAPI      T\_GetProcAddress (
                          HMODULE hModule,
                          LPCSTR lpProcName
                          );
typedef DWORD   WINAPI      T\_GetCurrentDirectoryA(
                             DWORD nBufferLength,
                             LPSTR lpBuffer
                             );

typedef void \*  WINAPI  T\_GetModuleHandleA(
                       LPCSTR lpModuleName
                       );
typedef HINSTANCE WINAPI  T\_LoadLibrary( LPCSTR lpLibFileName);

//ShellCode 函数
char szGetKernelBase\[\] = {
'\\x56', '\\x33', '\\xC0', '\\x64', '\\x8B', 
'\\x40', '\\x30', '\\x85', '\\xC0', '\\x78', '\\x0F', 
'\\x8B', '\\x40', '\\x0C', '\\x8B', '\\x70', '\\x1C', 
'\\xAD', '\\x8B', '\\x40', '\\x08', '\\xEB', '\\x0C', 
'\\x90', '\\x90', '\\x90', '\\x8B', '\\x40', '\\x34', 
'\\x8D', '\\x40', '\\x7C', '\\x8B', '\\x40', '\\x3C', 
'\\x5E', '\\xC3', '\\x90'
};

int GetAPIGetProcAddressOffset(int hKernel32)
{
  char \*pMem = (char \*)hKernel32;
  int  \*piOffset = NULL;
  int   iETFileOffset = 0;
  pMem += DOS\_HEADER\_SIZE - 4;

  //取得 pe 头偏移
  piOffset = (int \*)pMem;

  pMem = (char \*)hKernel32 + \*piOffset;
  pMem += SECOND\_DATA\_DIR\_LIST\_OFFSET;

  //取得输出表偏移
  piOffset = (int \*)pMem;


  //pMem 指向输出表
  pMem = (char \*)hKernel32 + \*piOffset;


  int  \*piAddressOfName      = NULL;
  int  \*piPointerOfName      = NULL;

  int  \*piAddressOfFunctions    = NULL;
  int  \*piPointerOfFunctions    = NULL;

  char \*pszAddressOfName      = NULL;

  piAddressOfFunctions  = (int \*)(pMem + 28);
  piAddressOfName      = (int \*)(pMem + 32);

  piPointerOfName      = (int \*)(hKernel32 + 

\*piAddressOfName);
  piPointerOfFunctions    = (int \*)(hKernel32 + 

\*piAddressOfFunctions);

  pszAddressOfName    = (char \*)(hKernel32 + 

\*piPointerOfName);

  while (strcmp(pszAddressOfName,"GetProcAddress") != 0)
  {
    piPointerOfName++;      //= (int \*)

(hKernel32 + \*piAddressOfName);
    piPointerOfFunctions++;
    pszAddressOfName    = (char \*)(hKernel32 + 

\*piPointerOfName);
  } 

  return (\*piPointerOfFunctions);
}

int main()
{
  int     hKernel32Base = NULL;
  void    \*pTmp = szGetKernelBase;
  T\_GetKernelBase \*pGetKernelBase = (T\_GetKernelBase \*)pTmp;
  hKernel32Base = pGetKernelBase();

  int ProcAddressAPIOffset = 0;
  ProcAddressAPIOffset = GetAPIGetProcAddressOffset(hKernel32Base);

  T\_GetProcAddress \*pGetProcAddress = (T\_GetProcAddress \*)
    (hKernel32Base + ProcAddressAPIOffset);

  T\_LoadLibrary \*pLoadLibraryA = (T\_LoadLibrary \*)
    pGetProcAddress((void \*)hKernel32Base,"LoadLibraryA");

  //虽然没必要，但是作为演示，还是来一次
  HMODULE hDll = (HMODULE)pLoadLibraryA("kernel32.dll");
  T\_GetCurrentDirectoryA \*pGetCurrentDirectoryA =(T\_GetCurrentDirectoryA \*)
    pGetProcAddress(hDll,"GetCurrentDirectoryA");
  char buf\[1024\];
  pGetCurrentDirectoryA(1024,buf);
  printf("当前目录是:\\n%s\\n",buf);
  return 0;
    
}

```