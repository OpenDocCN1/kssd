

# 去除花指令的代码

> 原文：[`www.pediy.com/kssd/pediy10/70322.html`](https://www.pediy.com/kssd/pediy10/70322.html)

奥运开幕前夕，发个帖子留作纪念。  
  
#define WIN32\_LEAN\_AND\_MEAN    
#include "windows.h"  
  
BYTE flowCode1\[\] = {......}; // ......表示你的花指令组合，这个需要你自己替换了。  
  
BYTE flowCode2\[\] = { ........};  
  
BYTE flowCode3\[\] = { .........};  
char szFileName\[\] = "combojiang.exe";  
  
//找出花指令的位置并去掉花指令  
void FindFlowerCodeAndRemove(LPVOID src, LPVOID flw, int nSrcLen,int nflwLen)  
{  
  \_\_asm  
  {  
    xor eax,eax  
    push esi   
    push edi   
    push ecx  
  
    cld   
    mov esi,flw  
    mov edi,src  
    mov ecx,nSrcLen  
    lodsb  
      
Start:  
    repnz scasb  
    or ecx,ecx  
    jz NotFindFlower  
    push eax  
    push ecx  
    push esi   
    push edi   
  
    lea edi,\[edi - 1\]  
    mov ecx,nflwLen  
    mov esi,flw  
        repz cmpsb  
  
    or ecx,ecx  
    jnz MYLOOP  
        
    mov ecx,nflwLen  
    sub edi,nflwLen  
    mov al,0x90  
    rep stosb  
  
MYLOOP:  
               pop edi   
    pop esi   
    pop ecx   
    pop eax  
    jmp Start  
  
NotFindFlower:  
    pop ecx  
    pop edi   
    pop esi  
  }  
}  
  
  
  
int main(int argc, char\* argv\[\])  
{  
    DWORD dwHigh;  
  HANDLE hFile = CreateFile(szFileName,GENERIC\_READ|GENERIC\_WRITE,FILE\_SHARE\_READ,0,OPEN\_EXISTING,FILE\_ATTRIBUTE\_NORMAL,NULL);    
  HANDLE hMap = CreateFileMapping(hFile,NULL,PAGE\_READWRITE,0,0,0);  
  LPVOID lpMem = MapViewOfFile(hMap,FILE\_MAP\_ALL\_ACCESS,0,0,0);  
  int nFileLen  = GetFileSize(hFile,&dwHigh);  
  
  
   FindFlowerCodeAndRemove(lpMem, flowCode1, nFileLen,23);    
   FindFlowerCodeAndRemove(lpMem, flowCode2, nFileLen,12);  
  FindFlowerCodeAndRemove(lpMem, flowCode3, nFileLen,8);  
    
  UnmapViewOfFile(lpMem);  
  CloseHandle(hMap);  
  CloseHandle(hFile);  
  return 0;  
}