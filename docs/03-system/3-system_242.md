
# R3 读文件获取原始 SSDT Shadow 里地址

> 原文：[`www.pediy.com/kssd/pediy10/101165.html`](https://www.pediy.com/kssd/pediy10/101165.html)

方法不是原创，先声明，免得被骂，代码是自己写的。

参考资料是：`http://hi.baidu.com/robin\_dev/blog/item/5a593fca5449ec18be09e6cf.html`

在这里郑重感谢！！！这个帖子给了个方法，无码，不过有这个了实现倒是不难，我自己写了一个，看到看雪论坛上也没这个方法的代码，所以在这分享出来，写得很烂，希望各位指正，喷我也没关系。

我封装成了一个类（个人的习惯啊），比较简单，感谢教主等人的帮助。

类声明：

```
//file:SDTShadowRestore.h

#ifndef SDTSHADOWRESTORE_H

#define SDTSHADOWRESTORE_H

#include <windows.h>
#include <Tchar.h>

typedef struct _SYSTEM_SERVICE_TABLE 
{ 
   PVOID   ServiceTableBase; 
   PULONG  ServiceCounterTableBase; 
   ULONG   NumberOfService; 
   ULONG   ParamTableBase; 
}SYSTEM_SERVICE_TABLE,*PSYSTEM_SERVICE_TABLE;

typedef struct _SERVICE_DESCRIPTOR_TABLE
{
  PVOID KiServiceTabe;
  PVOID W32pServiceTable;
  PVOID Reserved_1;
  PVOID Reserved_2;
}SERVICE_DESCRIPTOR_TABLE,*PSERVICE_DESCRIPTOR_TABLE;


class SDTShadowRestore
{
public:
  SDTShadowRestore(void)
  {
    m_dwWin32kBase = 0;
    m_dwW32pServiceTable = 0;
    m_KeServiceDescriptorTableShadow = 0;
    this->Init();
  }
  ~SDTShadowRestore(void)
  {
    if(m_pKernelName)
    {
      ::GlobalFree(m_pKernelName);
    }
  }
  bool Init();               //初始化该类
  DWORD FindW32pServiceTable();     //得到原始 W32pServiceTable 内存地址
  DWORD GetAddressSSDTShadow();              //得到 KeServiceDescriptorTableShadow 在内核中的内存地址
  DWORD GetServiceAddressById(DWORD ServiceId);        //获取服务原始地址
private:
  void AnsiToPTSTR(PTSTR DesStr, char *SourceStr, DWORD cbDesStr);
  
  DWORD GetProcFromIAT(LPCTSTR szProcName);      //根据函数名在 IAT 中查找函数地址
private:
  PIMAGE_OPTIONAL_HEADER m_pWin32kOptionalHeader;            //可选头的地址，非 RVA
  HMODULE m_hWin32kModule;               //自己加载的 Win32k.sys 内核模块的模块句柄
  LPTSTR m_pKernelName;           //内核核心模块名
  HMODULE m_hKernelModule;            //内核核心模块 ntoskrnl.exe 或者 ntkrnlpa.exe
  DWORD m_dwKernelBase;          //内核基址
  DWORD m_dwWin32kBase;          //Win32k.sys 基址
  PSERVICE_DESCRIPTOR_TABLE m_KeServiceDescriptorTableShadow;         //内核中 SSDT Shadow 的真实内存地址
  DWORD m_dwW32pServiceTable;     //原始 W32pServiceTaable 表真实地址
};

#endif    
```

类实现：

```
//file:SDTShadowRestore.cpp

#include "SDTShadowRestore.h"

bool SDTShadowRestore::Init()
{
    // Load Kernel Module
    m_hKernelModule = ::GetModuleHandle(NULL);
    if(!m_hKernelModule)
    {
        return false;
    }
    m_dwKernelBase = (DWORD)m_hKernelModule;

    // Get Kernel Name
    // ... (Implementation details omitted in original snippet but structurally present in source logic)
    // Assuming initialization logic continues here
    
    return true;
}

DWORD SDTShadowRestore::FindW32pServiceTable()
{
    // Implementation for finding W32pServiceTable
    // Uses GetProcAddress or similar mechanisms on loaded Win32k.sys
    return m_dwW32pServiceTable;
}

DWORD SDTShadowRestore::GetAddressSSDTShadow()
{
    // Implementation for getting KeServiceDescriptorTableShadow address
    // Usually offsets from kernel base
    return (DWORD)m_KeServiceDescriptorTableShadow;
}

DWORD SDTShadowRestore::GetServiceAddressById(DWORD ServiceId)
{
    // Implementation for getting original service address by ID
    return 0;
}
```



```cpp
#include "SDTShadowRestore.h"

typedef struct _SYSTEM_MODULE_INFORMATION  // Information Class 11
{
  ULONG Reserved[2];  //+0
  PVOID Base;         //+08h
  ULONG Size;         //+0ch
  ULONG Flags;        //+10h
  USHORT Index;       //+14h
  USHORT Unknown;     //+16h
  USHORT LoadCount;   //+18h
  USHORT ModuleNameOffset; //+1Ah
  CHAR  ImageName[256];   //+1Ch
} SYSTEM_MODULE_INFORMATION, *PSYSTEM_MODULE_INFORMATION;


typedef struct _tagSysModuleList 
{
  ULONG ulCount;
  SYSTEM_MODULE_INFORMATION smi;
} SYSMODULELIST, *PSYSMODULELIST;


typedef DWORD SYSTEM_INFORMATION_CLASS;

#define  STATUS_INFO_LEN_MISMATCH   0xc0000004

typedef LONG (_stdcall *pFnZwQuerySystemInformation ) 
( IN SYSTEM_INFORMATION_CLASS SystemInformationClass, 
 IN OUT PVOID SystemInformation, 
 IN ULONG SystemInformationLength, 
 OUT PULONG ReturnLength  );




void SDTShadowRestore::AnsiToPTSTR(PTSTR DesStr, char *SourceStr, DWORD cbDesStr)
{
#ifndef _UNICODE
  lstrcpy(DesStr, SourceStr);
#else
  MultiByteToWideChar(CP_ACP, 0, SourceStr, -1, DesStr, cbDesStr);
#endif

}

bool SDTShadowRestore::Init()
{
  pFnZwQuerySystemInformation ZwQuerySystemInformation = (pFnZwQuerySystemInformation)::GetProcAddress(::LoadLibrary(_T("ntdll.dll")), "ZwQuerySystemInformation");
  PSYSMODULELIST pSysModuleList = NULL;
  DWORD ReturnLength = 0, dwRet = 0;
  TCHAR ModuleName[MAX_PATH];
  dwRet = ZwQuerySystemInformation(11, (PVOID)pSysModuleList, 0, &ReturnLength);
  if(dwRet == STATUS_INFO_LEN_MISMATCH)
  {
    pSysModuleList = (PSYSMODULELIST)::GlobalAlloc(GPTR, ReturnLength);
    ZwQuerySystemInformation(11, (PVOID)pSysModuleList, ReturnLength, NULL);           //获取系统加载模块信息
    PSYSTEM_MODULE_INFORMATION pSystemModuleInformation = &(pSysModuleList->smi);
    m_dwKernelBase = (DWORD)(pSystemModuleInformation->Base);                 //获取系统内核核心模块 ntoskrnl.exe 加载基址,PAE 模式为 ntkrnlpa.exe
    char* pAnsiKernelName = pSystemModuleInformation->ModuleNameOffset + pSystemModuleInformation->ImageName;
    m_pKernelName = (PTSTR)::GlobalAlloc(GPTR, sizeof(TCHAR)*(strlen(pAnsiKernelName)+1));                 
    this->AnsiToPTSTR(m_pKernelName, pAnsiKernelName, sizeof(TCHAR)*(strlen(pAnsiKernelName)+1));    //获取内核核心模块名称(以此判定是否为 PAE 模式)
    this->m_hKernelModule = ::LoadLibraryEx(m_pKernelName, 0, DONT_RESOLVE_DLL_REFERENCES);      //在用户态加载内核核心模块
    this->m_hWin32kModule = ::LoadLibraryEx(_T("Win32k.sys"), 0, DONT_RESOLVE_DLL_REFERENCES);   //在用户态加载 Win32k.sys 模块
    for(DWORD index = 0; index < pSysModuleList->ulCount; index++)               //通过循环获取 Win32k.sys 在内核加载的基址，后面重定位 Shadow SSDT 中函数地址用到
    {
      TCHAR szModuleName[50];
      AnsiToPTSTR(szModuleName, (pSystemModuleInformation->ImageName + pSystemModuleInformation->ModuleNameOffset), 50);
      if(lstrcmp(_tcsupr(szModuleName), _T("WIN32K.SYS")) == 0)
      {
        this->m_dwWin32kBase = (DWORD)(pSystemModuleInformation->Base);
        break;
      }
      pSystemModuleInformation++;
    }
    if(pSysModuleList)
    {
      ::GlobalFree(pSysModuleList);
    }
  }
  this->m_pWin32kOptionalHeader = (PIMAGE_OPTIONAL_HEADER)((DWORD)m_hWin32kModule + ((PIMAGE_DOS_HEADER)m_hWin32kModule)->e_lfanew + 0x18);     //Win32k.sys 可选头
  return true;
}


DWORD SDTShadowRestore::GetAddressSSDTShadow()         //得到 KeServiceDescriptorTableShadow 在内核中的内存地址
{
  if(m_dwWin32kBase)
  {
    DWORD dwKeAddSystemServiceTable = (DWORD)GetProcAddress(m_hKernelModule, "KeAddSystemServiceTable");
    for(DWORD dwCurAddress = dwKeAddSystemServiceTable; dwCurAddress < dwKeAddSystemServiceTable + 0x100; dwCurAddress++)
    {
      /*通过定位
      8d8840355580    lea     ecx,nt!KeServiceDescriptorTableShadow (80553540)[eax]
      833900          cmp     dword ptr [ecx],0
      */
      if(*(PWORD)dwCurAddress == 0x888d && *(PWORD)(dwCurAddress + 6) == 0x3983)
      {
        m_KeServiceDescriptorTableShadow = (PSERVICE_DESCRIPTOR_TABLE)(*(PDWORD)(dwCurAddress + 2) - (DWORD)m_hKernelModule + m_dwKernelBase);
        return (DWORD)m_KeServiceDescriptorTableShadow;         //该地址位于内核空间不可访问
      }
    }
  }
  return NULL;
}


DWORD SDTShadowRestore::FindW32pServiceTable()     //得到原始 W32pServiceTable 内存地址
{
  //获取 Win32k.sys 得 DriverEntry 地址
  DWORD dwEntryPoint = this->m_pWin32kOptionalHeader->AddressOfEntryPoint + (DWORD)m_hWin32kModule;
  //从 IAT 中获取 KeAddSystemServiceTable 的地址
  DWORD dwKeAddSystemServiceTable = GetProcFromIAT(_T("KeAddSystemServiceTable"));
  /*IDA 反汇编结果：
  68 80 A2 99 BF         push    offset off_BF99A280
  FF 15 58 D4 98 BF      call    ds:KeAddSystemServiceTable
  */
  //通过 call    ds:KeAddSystemServiceTable 的定位，该定位应该比较准确
  for(DWORD dwCurAddress = dwEntryPoint; dwCurAddress < dwEntryPoint + 0x1000; dwCurAddress++)
  {
    if(*(PWORD)dwCurAddress == 0x15ff )
    {   
      //计算出加载后的 ds:KeAddSystemServiceTable 地址，然后该地址中存放的即是 KeAddSystemServiceTable 真实入口地址
      DWORD dwFunAddress = *(PDWORD)(*(PULONG)(dwCurAddress + 2) - m_pWin32kOptionalHeader->ImageBase + (DWORD)m_hWin32kModule);
      if(dwKeAddSystemServiceTable == dwFunAddress)
      {
        //将该地址结合内核中 Win32k 的加载地址进行重定位
        m_dwW32pServiceTable = *(PDWORD)(dwCurAddress - 4) - m_pWin32kOptionalHeader->ImageBase + (DWORD)m_hWin32kModule;
        return m_dwW32pServiceTable;
      }
    }
  }
  return 0;
}


DWORD SDTShadowRestore::GetServiceAddressById(DWORD ServiceId)       //获取服务原始地址
{
  if(!m_dwW32pServiceTable)
  {
    FindW32pServiceTable();
  }
  if(m_dwW32pServiceTable)
  {
    return *(PDWORD)(m_dwW32pServiceTable + ServiceId * 4) - m_pWin32kOptionalHeader->ImageBase + m_dwWin32kBase;
  }
}

DWORD SDTShadowRestore::GetProcFromIAT(LPCTSTR szProcName)     //根据函数名在 IAT 中查找函数地址
{
  PIMAGE_IMPORT_DESCRIPTOR pImageTable = (PIMAGE_IMPORT_DESCRIPTOR)(m_pWin32kOptionalHeader->DataDirectory[IMAGE_DIRECTORY_ENTRY_IMPORT].VirtualAddress + (DWORD)m_hWin32kModule);
  
  TCHAR szDllName[20];
  while(pImageTable->Name || pImageTable->FirstThunk || pImageTable->OriginalFirstThunk)
  {
    AnsiToPTSTR(szDllName, (char*)(pImageTable->Name + (DWORD)m_hWin32kModule), 20);
    if(lstrcmp(_tcsupr(szDllName), _T("NTOSKRNL.EXE")) == 0)
    {
      break;
    }
    pImageTable++;
  }
  PIMAGE_THUNK_DATA pOrgFirstThunk = (PIMAGE_THUNK_DATA)(pImageTable->OriginalFirstThunk + (DWORD)m_hWin32kModule);
  PIMAGE_THUNK_DATA pFirstThunk = (PIMAGE_THUNK_DATA)(pImageTable->FirstThunk + (DWORD)m_hWin32kModule);
  TCHAR szFunName[30];
  for(DWORD index = 0; &(pOrgFirstThunk[index]); index++)
  {
    if(!(pOrgFirstThunk[index].u1.Ordinal & IMAGE_ORDINAL_FLAG32 ))
    {
      PIMAGE_IMPORT_BY_NAME pFunName = (PIMAGE_IMPORT_BY_NAME)(pOrgFirstThunk[index].u1.ForwarderString + (DWORD)m_hWin32kModule);   
      AnsiToPTSTR(szFunName, (char*)(pFunName->Name), 30);
      if(lstrcmp(szFunName, szProcName) == 0)
      {
        return pFirstThunk[index].u1.Function;
      }
    }
  }
  return 0;   
}
```

关键位置做了点解释，其他位置略过了  

然后关于`ShadowSSDT`中函数名的定位问题，稍微说下，教主以及其他牛牛们好像都说没办法定位，只能通过解析`PDB`和硬编码，我用的硬编码  

使用`IDA`就可以看到函数名，然后自己复制弄出来就`OK`了，`XP SP2`下是`667`个，手工复制不现实，我写了个小程序来解析出函数名：  

代码:



```
#include <windows.h>
#include <iostream>

using namespace std;

int main()
{
CHAR SrcFileName[MAX_PATH],DesFileName[MAX_PATH];
cout << "please input the file name of src:" <<endl;
cin >> SrcFileName;
cout << "please input the file name of des:" <<endl;
cin >> DesFileName;
HANDLE hSrcFile = ::CreateFileA(SrcFileName, GENERIC_READ, FILE_SHARE_READ, NULL, OPEN_EXISTING, FILE_ATTRIBUTE_NORMAL, NULL);

if(!hSrcFile)
{
   return 1;
}
HANDLE hSrcMap = ::CreateFileMappingA(hSrcFile, NULL, PAGE_READONLY, 0, 0, NULL);

if(!hSrcMap)
{
   return 1;
}
LPVOID lpSrcMapAddress = ::MapViewOfFile(hSrcMap, FILE_MAP_READ, 0, 0, 0);
char* pSrc = (char*)lpSrcMapAddress;
HANDLE hDesFile = ::CreateFileA(DesFileName, GENERIC_WRITE|GENERIC_READ, FILE_SHARE_READ, NULL, CREATE_ALWAYS, FILE_ATTRIBUTE_NORMAL, NULL);
DWORD dwFileSizeHigh = 0;
DWORD dwFileSizeLow = ::GetFileSize(hSrcFile, &dwFileSizeHigh);
HANDLE hDesMap = ::CreateFileMappingA(hDesFile, NULL, PAGE_READWRITE, dwFileSizeHigh, dwFileSizeLow, NULL);
DWORD a = ::GetLastError();
LPVOID lpDesMapAddress = ::MapViewOfFile(hDesMap, FILE_MAP_WRITE, 0, 0, 0);
char* pDes = (char*)lpDesMapAddress;
bool bFlag = false;
while(pSrc - lpSrcMapAddress < dwFileSizeLow )
{
   //加上*(pSrc - 1) == ' '(此处不是空，是空格字符)是为了防止对此类中间存在'_'符号的错误解析，_NtGdiHT_Get8BPPFormatPalette@16
   if(*pSrc == '_' && *(pSrc - 1) == ' ')
   {
    bFlag = true;
    *pDes = 'L';
    pDes++;
    *pDes = '"';
    pDes++;
   }
   else if(*pSrc == '@')
   {
    bFlag = false;
    *pDes = '"';
    pDes++;
    *pDes = ',';
    pDes++;
    *pDes = 13;
    pDes++; 
    *pDes = 10;
    pDes++;
   }
   else
   {
    if(bFlag)
    {
     *pDes = *pSrc;
     pDes++;
    }
   }
   pSrc++;
}
::UnmapViewOfFile(lpSrcMapAddress);
::UnmapViewOfFile(lpDesMapAddress);
::CloseHandle(hSrcMap);
::CloseHandle(hDesMap);
::CloseHandle(hSrcFile);
::CloseHandle(hDesFile);
return 0;
}
```

希望对有需要的人有点用啊
