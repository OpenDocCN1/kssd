

# 重映射 User32.dll 调用 MessageBox 对话框

> 原文：[`www.pediy.com/kssd/pediy12/140456.html`](https://www.pediy.com/kssd/pediy12/140456.html)

#include <windows.h>  
  
  
  
DWORD ReGetProcAddress(HMODULE hModule, HMODULE hOldModule, LPCSTR lpProcName)  
{  
  DWORD MyApiAddr;  
  DWORD ApiOffset;  
  DWORD ReApiAddr;  
  
    
  MyApiAddr = (DWORD)GetProcAddress(hOldModule, lpProcName);  
  ApiOffset = MyApiAddr - (DWORD)hOldModule;  
  ReApiAddr = (DWORD)hModule + ApiOffset;  
  
  return (ReApiAddr);  
}  
  
  
HMODULE RemapLibrary(LPCSTR lpModuleName)  
{  
  HMODULE hMod;  
  DWORD ImageSize;  
  BYTE\* Virtual;  
  PIMAGE\_DOS\_HEADER DosHeader;  
  PIMAGE\_NT\_HEADERS NtHeader;  
  
  hMod = GetModuleHandleA(lpModuleName);  
  if(!hMod)  
  {  
    return (0);  
  }  
  
  DosHeader = (PIMAGE\_DOS\_HEADER)hMod;  
  NtHeader = (PIMAGE\_NT\_HEADERS)((DWORD)DosHeader + (DWORD)DosHeader->e\_lfanew);  
  ImageSize = (DWORD)NtHeader->OptionalHeader.SizeOfImage;  
    
  Virtual = (BYTE\*)VirtualAlloc(NULL, ImageSize, MEM\_COMMIT, PAGE\_READWRITE);  
  if(!Virtual)  
  {  
    return (0);  
  }  
  
  VirtualLock(Virtual, ImageSize);  
  
  memcpy(Virtual, (PBYTE)hMod, ImageSize);  
  
  return ((HMODULE)Virtual);  
}  
  
  
typedef DWORD (WINAPI \*MSGBOX)(HWND,LPSTR,LPSTR,UINT);  
  
void main()  
{  
  HMODULE hOldMod, hNewMod;  
  MSGBOX MyMessageBox;  
  
  printf("开始动态加载 User32.dll\\n");  
  
  hOldMod = LoadLibrary("user32.dll");  
  
  printf("User32.dll 加载成功!\\n");  
  
  printf("开始重新映射 User32.dll\\n");  
  
  hNewMod = RemapLibrary("user32.dll");  
  
  printf("User32.dll 重新映射成功!\\n 按任意键继续...\\n");  
  
  getchar();  
  
  printf("开始释放原 User32.dll\\n");  
  
  FreeLibrary(hOldMod);  
  
  printf("User32.dll 释放成功!\\n 按任意键继续...\\n");  
  
  getchar();  
  
  MyMessageBox = (MSGBOX)ReGetProcAddress(hNewMod, hOldMod, "MessageBoxA");  
    
  MyMessageBox(NULL, "ReMapLibrary OK!", "Test", 0);  
  
  printf("调用 MessageBoxA 成功!");  
  
}: