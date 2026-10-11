
# 如何中断 Themida 的 MessageBox 对话框

> 原文：[`www.pediy.com/kssd/pediy14/140298.html`](https://www.pediy.com/kssd/pediy14/140298.html)

原理：读取系统 DLL 到分配的内存里面去，然后定位到相关的 API 的代码。

`FARPROC GetApiAddr(BYTE *Base,DWORD Api)`

参数一：我们分配的内存基址

参数二：函数地址减去模块基址的差值

返回值：返回内存中得 API 的函数的地址

作用：对原来的系统 API 函数下断点无效，因为我们已经吧全部系统 DLL 读取到内存中定位到相关的代码了，调用的是内存中得 API 函数

代码:

```c
#include <windows.h>
char Caption[]="Test";
char Text[]="MessageBoxA";
FARPROC  GetApiAddr(BYTE *Base,DWORD Api)
{
  DWORD VirtualSize;
  DWORD PhysicalAddr;
  DWORD RawSize;   
  FARPROC RET=NULL;
  PIMAGE_DOS_HEADER DosHeader=(PIMAGE_DOS_HEADER)Base;
  PIMAGE_NT_HEADERS NtHeader=(PIMAGE_NT_HEADERS)((DWORD)DosHeader+(DWORD)DosHeader->e_lfanew);
  PIMAGE_SECTION_HEADER SecHeader=(PIMAGE_SECTION_HEADER)((DWORD)NtHeader+sizeof(IMAGE_FILE_HEADER)+   
    NtHeader->FileHeader.SizeOfOptionalHeader+4);
  BYTE SecSum=NtHeader->FileHeader.NumberOfSections;
  while(SecSum)
  {
    PhysicalAddr=(DWORD)SecHeader->Misc.PhysicalAddress;
    VirtualSize=(DWORD)SecHeader->VirtualAddress;//>Misc.VirtualSize;
    PhysicalAddr+=VirtualSize;
    if(Api>=VirtualSize&&Api<=PhysicalAddr)
    {
      RawSize=SecHeader->PointerToRawData;
      RawSize-=VirtualSize;
      Base+=RawSize;
      Base+=Api;
    }
    SecHeader++;
    SecSum--;
  }
  RET=(FARPROC)Base;
  return RET;
}
int WINAPI WinMain(HINSTANCE hInstance,HINSTANCE hPrevInstance,LPSTR lpCmdLine,int nCmdShow)
{
  DWORD dwRead;
  char SystemPath[MAX_PATH]={0};
  HMODULE hMod=GetModuleHandleA("USER32.DLL");
  if(!hMod)
    hMod=LoadLibraryA("USER32.DLL");
  GetSystemDirectoryA(SystemPath,MAX_PATH);
  lstrcat(SystemPath,"\\user32.dll");
  HANDLE Handle=CreateFileA(SystemPath,GENERIC_READ,FILE_SHARE_READ,NULL,OPEN_EXISTING,FILE_ATTRIBUTE_NORMAL,NULL);
  DWORD Size=GetFileSize(Handle,NULL);
  BYTE *Virtual=VirtualAlloc(NULL,Size,MEM_COMMIT,PAGE_READWRITE);
  VirtualLock(Virtual,Size);
  ReadFile(Handle,Virtual,Size,&dwRead,NULL);
  DWORD MeAddr=(DWORD)GetProcAddress(hMod,"MessageBoxA");
  MeAddr=MeAddr-(DWORD)hMod;
  DWORD Api=(DWORD)GetApiAddr(Virtual,MeAddr);
_asm{
    push 0
    lea eax,Caption
    push eax
    lea eax,Text
    push eax
    push 0
    call Api
  }
  return FALSE;
}
```
