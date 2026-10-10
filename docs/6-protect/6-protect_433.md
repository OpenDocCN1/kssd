

# 如何中断 Themida 的 MessageBox 对话框

> 原文：[`www.pediy.com/kssd/pediy12/140298.html`](https://www.pediy.com/kssd/pediy12/140298.html)

原理：读取系统 DLL 到分配的内存里面去，然后定位到相关的 API 的代码。  
FARPROC  GetApiAddr(BYTE \*Base,DWORD Api)  
参数一：我们分配的内存基址  
参数二：函数地址减去模块基址的差值  
返回值：返回内存中得 API 的函数的地址  
作用：对原来的系统 API 函数下断点无效，因为我们已经吧全部系统 DLL 读取到内存中定位到相关的代码了，调用的是内存中得 API 函数  
  

代码:

```
#include <windows.h>
char Caption\[\]="Test";
char Text\[\]="MessageBoxA";
FARPROC  GetApiAddr(BYTE \*Base,DWORD Api)
{
  DWORD VirtualSize;
  DWORD PhysicalAddr;
  DWORD RawSize;  
  FARPROC RET=NULL;
  PIMAGE\_DOS\_HEADER DosHeader=(PIMAGE\_DOS\_HEADER)Base;
  PIMAGE\_NT\_HEADERS NtHeader=(PIMAGE\_NT\_HEADERS)((DWORD)DosHeader+(DWORD)DosHeader->e\_lfanew);
  PIMAGE\_SECTION\_HEADER SecHeader=(PIMAGE\_SECTION\_HEADER)((DWORD)NtHeader+sizeof(IMAGE\_FILE\_HEADER)+  
    NtHeader->FileHeader.SizeOfOptionalHeader+4);
  BYTE SecSum=NtHeader->FileHeader.NumberOfSections;
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
  return RET;
}
int WINAPI WinMain(HINSTANCE hInstance,HINSTANCE hPrevInstance,LPSTR lpCmdLine,int nCmdShow)
{
  DWORD dwRead;
  char SystemPath\[MAX\_PATH\]={0};
  HMODULE hMod=GetModuleHandleA("USER32.DLL");
  if(!hMod)
    hMod=LoadLibraryA("USER32.DLL");
  GetSystemDirectoryA(SystemPath,MAX\_PATH);
  lstrcat(SystemPath,"\\\\user32.dll");
  HANDLE Handle=CreateFileA(SystemPath,GENERIC\_READ,FILE\_SHARE\_READ,NULL,OPEN\_EXISTING,FILE\_ATTRIBUTE\_NORMAL,NULL);
  DWORD Size=GetFileSize(Handle,NULL);
  BYTE \*Virtual=VirtualAlloc(NULL,Size,MEM\_COMMIT,PAGE\_READWRITE);
  VirtualLock(Virtual,Size);
  ReadFile(Handle,Virtual,Size,&dwRead,NULL);
  DWORD MeAddr=(DWORD)GetProcAddress(hMod,"MessageBoxA");
  MeAddr=MeAddr-(DWORD)hMod;
  DWORD Api=(DWORD)GetApiAddr(Virtual,MeAddr);
\_asm{
    push 0
    lea eax,Caption
    push eax
    lea eax,Text
    push eax
    push 0
    call Api
  }
  return FALSE;
}

```