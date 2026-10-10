
# 我的第一次 vc 转 shellcode 历程

> 原文：[`www.pediy.com/kssd/pediy12/142657.html`](https://www.pediy.com/kssd/pediy12/142657.html)

## C++转 shellcode 的曲折历程

没有人响应,看来写的太滥了, 自己重新排一下版, 把原代码都粘上, 希望招徕几个看客...

第一次写类似的东西，实在是没有经验，历程非常坎坷，先后用了大约两个月时间，主要是因为我之前的电脑台垃圾了，6 年前的笔记本 512MB 内存，随便抛个调试器就快死机了，也没有太多心情搞那个，就一直拖了下来，直到换了新机才来完成这一遗留问题。

开始的目的是，我这里有一个 dll 里面会暴露出两个方法， 同样进程中有多个这样的 dll， 我的目的是获得其他 dll 对应函数的地址。 正常高级语言应该是没有这样的功能的，所以选择用 pe 表查询的方式。

```cpp
// BWChess.cpp : Defines the entry point for the console application.
//  
引用: #include "stdafx.h"
#include <windows.h>
#include <winnt.h>
#include <stdio.h>
#include <Psapi.h>
#include <atlstr.h>
#include <iostream>
using namespace std;
typedef BOOL (*PENUMPROCESSMODS)( HANDLE, HMODULE*, DWORD, LPDWORD );
typedef HANDLE (*PGETCURRENTPROC)( VOID );
typedef int  (*PGETPROCADDR)(HANDLE, LPCTSTR);
typedef BOOL (WINAPI *PEnumProcessModules)( HANDLE, HMODULE *, DWORD, LPDWORD );
typedef WINBASEAPI HMODULE (WINAPI *PGetModuleHandleW)( IN LPCWSTR lpModuleName );
typedef WINBASEAPI HANDLE (WINAPI *PGetCurrentProcess)( VOID );
typedef DWORD (WINAPI *PGetModuleFileNameExW)( HANDLE hProcess, HMODULE hModule, LPSTR lpFilename, DWORD nSize );
typedef WINBASEAPI HMODULE (WINAPI *PLoadLibraryW)( IN LPCWSTR lpLibFileName );
typedef WINBASEAPI DWORD (WINAPI *PGetCurrentDirectoryW)( IN DWORD nBufferLength, OUT LPSTR lpBuffer );
typedef BOOL (*PIsTheSameDir)(TCHAR* pszPath ,HMODULE hMod, void* pfn);
typedef void* ( *PGetMethod)( BYTE *hMod, DWORD dwidx );
typedef BOOL  ( *PGetMethodEx)( BYTE *hMod, void* pfn1, void* pfn2 );
typedef __declspec(dllexport) void ( *PEXPORT1)(int x, int y);
typedef __declspec(dllexport) void ( *PEXPORT2)(int& x, int& y, int& color);
HMODULE g_hModule;
void Relocate(void* pfnissame, void* pfnsch)
{
  DWORD* phead = (DWORD*)(((BYTE*)pfnissame) + 0x1f);
  DWORD* phead2 = (DWORD*)(((BYTE*)pfnsch) + 0x0e);
  *phead2 = *phead;
  DWORD dwPos = (DWORD)(((BYTE*)pfnissame) + 0x15b);
  DWORD* dwVal = (DWORD*)(((BYTE*)pfnissame) + 0x157);
  DWORD dwPos2 = (DWORD)(((BYTE*)pfnsch) + 0x1f5);
  DWORD* dwVal2 = (DWORD*)(((BYTE*)pfnsch) + 0x1f1);
  *dwVal2 = dwPos + *dwVal - dwPos2;
}
void* GetMethod( BYTE *hMod, DWORD dwidx )
{
  if( !hMod )  
    return NULL;
  IMAGE_NT_HEADERS *pnt = (IMAGE_NT_HEADERS*)&hMod[PIMAGE_DOS_HEADER(hMod)->e_lfanew];
  IMAGE_EXPORT_DIRECTORY *exp = (IMAGE_EXPORT_DIRECTORY*)&hMod[pnt->OptionalHeader.DataDirectory->VirtualAddress];
  if( !exp->AddressOfNames )
    return NULL;
  DWORD *dwFunAddr = (DWORD*)&hMod[exp->AddressOfFunctions];
  DWORD *dwFunctions = (DWORD*)&hMod[exp->AddressOfNames];
  if( dwidx > exp->NumberOfFunctions )
    return NULL;
  return (void*)&hMod[dwFunAddr[dwidx-1]];
}
BOOL GetMethodEx( BYTE *hMod, void** pfn1, void** pfn2)
{
  if( !hMod )  
    return NULL;
  if( !pfn1 || ! pfn2)
    return FALSE;
  IMAGE_NT_HEADERS *pnt = (IMAGE_NT_HEADERS*)&hMod[PIMAGE_DOS_HEADER(hMod)->e_lfanew];
  IMAGE_EXPORT_DIRECTORY *exp = (IMAGE_EXPORT_DIRECTORY*)&hMod[pnt->OptionalHeader.DataDirectory->VirtualAddress];
  if( !exp->AddressOfNames )
    return NULL;
  DWORD *dwFunAddr = (DWORD*)&hMod[exp->AddressOfFunctions];
  DWORD *dwFunctions = (DWORD*)&hMod[exp->AddressOfNames];
  if( 2 != exp->NumberOfFunctions )
    return FALSE;
  *pfn1 = (void*)&hMod[dwFunAddr[0]];
  *pfn2 = (void*)&hMod[dwFunAddr[1]];
    return TRUE;
};
BOOL IsTheSameDir(TCHAR* pszPath ,HMODULE hMod, void* pfn)
{
  if( hMod == g_hModule )
    return FALSE;    
  TCHAR szDir[MAX_PATH] = {0};
  DWORD dw = (*(PGetCurrentDirectoryW)pfn)( (DWORD)MAX_PATH, (LPSTR)szDir );
  CString str(pszPath);
  if( 0 == str.CompareNoCase( szDir ) )
    return FALSE; //not an dll   
  str = str.Left( str.ReverseFind(_T('\\')) );
  return (0==str.CompareNoCase(szDir));
}
BOOL Searching(void* pfn1, void* pfn2, HANDLE hknl, DWORD& dwCnt, DWORD* pdwfn1, DWORD* pdwfn2, void* pfnis)
{
  if(!pfn1 || !pfn2 )
    return FALSE;
  if( !hknl )
    return FALSE;
  HMODULE hModule[1024];
  DWORD dwTotalSize=0;
  PGetCurrentProcess pgetcurrproc = (PGetCurrentProcess)(void*)(*(PGetMethod)pfn1)((BYTE*)hknl, 316);
  PGetModuleHandleW pgetmodhandle = (PGetModuleHandleW)(*(PGetMethod)pfn1)((BYTE*)hknl, 378);
  PLoadLibraryW ploadlib = (PLoadLibraryW)(*(PGetMethod)pfn1)((BYTE*)hknl, 584);
  PGetCurrentDirectoryW pgetcurrentdir = (PGetCurrentDirectoryW)(*(PGetMethod)pfn1)((BYTE*)hknl, 315);
  TCHAR szpsapi[] = { 0x50, 0x53, 0x41, 0x50, 0x49, 0x2e, 0x44, 0x4c, 0x4c, 0x00 };
  HANDLE hpsapi = (*ploadlib)( szpsapi );
  if(!hpsapi)
    return FALSE;
  PEnumProcessModules penumprocmod = (PEnumProcessModules)(*(PGetMethod)pfn1)((BYTE*)hpsapi, 5);
  PGetModuleFileNameExW pgetmodfilenameexw = (PGetModuleFileNameExW) (*(PGetMethod)pfn1)((BYTE*)hpsapi, 16);
  int nidx = 0;
  if( (*penumprocmod)((*pgetcurrproc)(), hModule, sizeof(hModule), &dwTotalSize) )
  {
    for ( int i = 0; i < (dwTotalSize / sizeof(HMODULE)); i++ )
    {
      TCHAR szModName[MAX_PATH];
      if ( (*pgetmodfilenameexw)( (HMODULE)(*pgetcurrproc)(), hModule[i], (LPSTR)szModName,
        sizeof(szModName)))
      {
        if( (*(PIsTheSameDir)pfnis)(szModName, hModule[i], (void*)pgetcurrentdir) )
          if( (*(PGetMethodEx)pfn2)( (BYTE*)hModule[i], &pdwfn1[dwCnt], &pdwfn2[dwCnt] ) )
          {
            ++dwCnt;
          }
      }
    }
  }
  return TRUE;
}
DWORD GetFunc( DWORD* pdwfn1, DWORD* pdwfn2 )
{
  TCHAR szkernel32[] = { 0x4B, 0x45, 0x52, 0x4E, 0x45, 0x4C, 0x33, 0x32, 0x2e, 0x44, 0x4c, 0x4c, 0x00 };
  HANDLE hknl = GetModuleHandle( szkernel32 );
  DWORD dwCnt = 0;
  Searching( (void*)GetMethod, (void*)GetMethodEx, hknl, dwCnt, pdwfn1, pdwfn2, IsTheSameDir );
  for( int x=0; x<dwCnt; ++x )
  {
    cout << pdwfn1[x] << "   " << pdwfn2[x] << endl;
  }
  return dwCnt;
}
int _tmain(int argc, _TCHAR* argv[])
{
  LoadLibrary( _T("D:\\workspace\\test projects\\BWChess\\Release\\sbot.dll") );  
  DWORD dwfn1[1024] = {0} ;
  DWORD dwfn2[1024] = {0} ;
  DWORD dwCnt = GetFunc(dwfn1, dwfn2);
  for( int x=0; x<dwCnt; ++x )
  {
    PEXPORT1 pfn1 = (PEXPORT1)(void*)dwfn1[x];
    PEXPORT2 pfn2 = (PEXPORT2)(void*)dwfn2[x];
    printf( "%08x - %08x\n", pfn1, pfn2 );
  }   
  getchar();
  return 0;
}
```

上面代码执行是没有问题的:

结果如图:

![](img/94373825641dfce440ebeeade68a4902.jpg)

但是这些肯定不是最终目的,目的是用 `shellcode` 形式表达, 利用 `windbg` 反汇编指令 `uf` 对 `Searching` 函数反汇编结果如下：

引用:
```
0:000> uf searching  
BWChess!Searching [d:\\testproject\\bwchess\\bwchess.cpp @ 157]:  
  157 004011f0 55              push    ebp  
  157 004011f1 8bec            mov     ebp,esp  
  157 004011f3 b848120000      mov     eax,1248h  
  157 004011f8 e803160000     ** call    BWChess!_chkstk (00402800) **  
  157 004011fd a138434000     ** mov     eax,dword ptr [BWChess!__security_cookie (00404338)] **  
  157 00401202 33c5            xor     eax,ebp  
  157 00401204 8945e4          mov     dword ptr [ebp-1Ch],eax  
  158 00401207 837d0800        cmp     dword ptr [ebp+8],0  
  158 0040120b 7406            je      BWChess!Searching+0x23 (00401213)  
BWChess!Searching+0x1d [d:\\testproject\\bwchess\\bwchess.cpp @ 158]:  
  158 0040120d 837d0c00        cmp     dword ptr [ebp+0Ch],0  
  158 00401211 7507            jne     BWChess!Searching+0x2a (0040121a)  
BWChess!Searching+0x23 [d:\\testproject\\bwchess\\bwchess.cpp @ 159]:  
  159 00401213 33c0            xor     eax,eax  
```
