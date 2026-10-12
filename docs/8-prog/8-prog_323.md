
# 简单 hook api 的实现

> 原文：[`www.pediy.com/kssd/pediy10/85719.html`](https://www.pediy.com/kssd/pediy10/85719.html)

小菜最近在学习 pe 文件结构和 windows 编程 就写了如下代码 检验一下学习成果

```cpp
////////////////////////////////////  
//HookDemo.cpp 文件  
#include <windows.h>  
#include <stdio.h>  

// 挂钩指定模块 hMod 对 MessageBoxA 的调用  
BOOL SetHookApi(HMODULE hMod);  

// 定义 MessageBoxA 函数原型  
typedef int (WINAPI \*PFNMESSAGEBOX)(HWND, LPCSTR, LPCSTR, UINT uType);  

// 保存 MessageBoxA 函数的真实地址  
PROC g_orgProc = (PROC)MessageBoxA;  

void main()  
{  
  // 调用原 API 函数  
  ::MessageBox(NULL, "原函数", "HookDemo", 0);  
  // 挂钩后再调用  
  SetHook(::GetModuleHandle(NULL));  
  ::MessageBox(NULL, "原函数", "HookDemo", 0);  
}  

// 用于替换 MessageBoxA 的自定义函数  
int WINAPI MyMessageBoxA(HWND hWnd, LPCSTR lpText, LPCSTR lpCaption, UINT uType)  
{  
  return ((PFNMESSAGEBOX)g_orgProc)(hWnd, "新函数", "HookDemo", uType);  
}  

BOOL SetHookApi(HMODULE hMod)  
{  
  IMAGE_DOS_HEADER* pDosHeader = (IMAGE_DOS_HEADER*)hMod;  
  IMAGE_OPTIONAL_HEADER * pOptHeader =  
    (IMAGE_OPTIONAL_HEADER *)((BYTE*)hMod + pDosHeader->e_lfanew + 24);  
  IMAGE_IMPORT_DESCRIPTOR* pImportDesc = (IMAGE_IMPORT_DESCRIPTOR*)  
    ((BYTE*)hMod + pOptHeader->DataDirectory[IMAGE_DIRECTORY_ENTRY_IMPORT].VirtualAddress);  
  // 在导入表中查找 user32.dll 模块。因为 MessageBoxA 函数从 user32.dll 模块导出  
  while(pImportDesc->FirstThunk)  
  {  
    char* pszDllName = (char*)((BYTE*)hMod + pImportDesc->Name);  
    if(lstrcmpiA(pszDllName, "user32.dll") == 0)  
    {  
      break;  
    }  
    pImportDesc++;  
  }  
  if(pImportDesc->FirstThunk)  
  {  
    // 一个 IMAGE_THUNK_DATA 就是一个双字，它指定了一个导入函数  
    // 调入地址表其实是 IMAGE_THUNK_DATA 结构的数组，也就是 DWORD 数组  
    IMAGE_THUNK_DATA* pThunk = (IMAGE_THUNK_DATA*)  
        ((BYTE*)hMod + pImportDesc->FirstThunk);  
    while(pThunk->u1.Function)  
    {  
      // lpAddr 指向的内存保存了函数的地址  
      DWORD* lpAddr = (DWORD*)&(pThunk->u1.Function);  
      if(*lpAddr == (DWORD)g_orgProc)  
      {   
        DWORD dwOldProtect;  
        MEMORY_BASIC_INFORMATION mb;  
        VirtualQuery(lpAddr, &mb, sizeof(mb));  
        VirtualProtect(lpAddr, sizeof(DWORD), PAGE_READWRITE, &dwOldProtect);  
        // 修改 IAT 表项，使其指向我们自定义的函数，相当于“*lpAddr = (DWORD)MyMessageBoxA;”  
        DWORD* lpNewProc = (DWORD*)MyMessageBoxA;  
        ::WriteProcessMemory(::GetCurrentProcess(),  
            lpAddr, &lpNewProc, sizeof(DWORD), NULL);  
        VirtualProtect(lpAddr, sizeof(DWORD), dwOldProtect, 0);  
        return TRUE;  
      }  
      pThunk++;  
    }  
  }  
  return FALSE;  
}
```
