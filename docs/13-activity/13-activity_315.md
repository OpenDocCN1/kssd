

# 奇虎 360 第 4 题的解法

> 原文：[`www.pediy.com/kssd/pediy10/78970.html`](https://www.pediy.com/kssd/pediy10/78970.html)

编译环境 vc6,直接 release 编译可以得到 sys 文件  
再修改子系统 1 个字节可以做为题目需求的 exe 和 dll  
  
我只用了一行内联 asm,不知道还有没有完全不用内联 asm 的方法  
  

代码:

```
#include <windows.h>

#pragma comment (linker, "/subsystem:native")
//#pragma comment (linker, "/subsystem:windows")
#pragma comment (linker, "/entry:real\_main")
#pragma comment (linker, "/FIXED:NO")
#pragma comment (linker, "/ALIGN:0x1000")
#pragma comment(linker, "/BASE:0x400000")

typedef DWORD (\_cdecl \*DBGPRINT)(char \*Format,...);
typedef DWORD (WINAPI \*OUTPUTDEBUGSTRING)(char \*lpOut);

DWORD GetModBase(DWORD x)
{
  DWORD p = x;
  DWORD dwTmp;

  while (p)
  {
    if (\*(WORD \*)p == 0x5A4D)
    {
      dwTmp = \*(DWORD \*)(p+0x3C) + p;
      if (\*(DWORD \*)dwTmp == 0x4550)
      {
        return p;
      }
    }
    p -= 0x10000;
  }

  return 0;
}

PVOID WINAPI GetExportByName(PVOID BaseAddress,char \*FuncName)
{
  PIMAGE\_NT\_HEADERS pNt;
  PIMAGE\_EXPORT\_DIRECTORY  ExportDir;
  DWORD ExportDirSize;
  PDWORD \*ExFunctions;
  PDWORD \*ExNames;
  USHORT \*ExOrdinals;
  PVOID ExName;
  DWORD Ordinal;
  PVOID Function = NULL;
  int minn,maxn;

  pNt = (PIMAGE\_NT\_HEADERS)(\*(DWORD \*)((DWORD)BaseAddress + 0x3C) + (DWORD)BaseAddress);
  if (pNt->OptionalHeader.Magic != 0x10B)
  {
    return NULL;
  }
  ExportDir = (PIMAGE\_EXPORT\_DIRECTORY)(pNt->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_EXPORT\].VirtualAddress + (DWORD)BaseAddress);
  ExportDirSize = pNt->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_EXPORT\].Size;
  if (ExportDir == NULL)
  {
    return NULL;
  }
  if (ExportDir->AddressOfNames == 0)
  {
    return NULL;
  }
  
  ExNames = (PDWORD \*)((DWORD)BaseAddress + ExportDir->AddressOfNames);
  ExOrdinals = (USHORT \*)((DWORD)BaseAddress + ExportDir->AddressOfNameOrdinals);
  ExFunctions = (PDWORD \*)((DWORD)BaseAddress + ExportDir->AddressOfFunctions);

  minn = 0;
  maxn = ExportDir->NumberOfNames - 1;
  while (minn <= maxn)
  {
        int mid;
        int res;
    
        mid = (minn + maxn) / 2;
    
        ExName = (PVOID)((DWORD)BaseAddress + (DWORD)ExNames\[mid\]);
        res = strcmp((PCHAR)ExName, (PCHAR)FuncName);
        if (res == 0)
    {
      Ordinal = ExOrdinals\[mid\];
      Function = (PVOID)((DWORD)BaseAddress + (DWORD)ExFunctions\[Ordinal\]);
      if (((DWORD)Function >= (DWORD)ExportDir) &&
        ((DWORD)Function < (DWORD)ExportDir + (DWORD)ExportDirSize))
      {
        return NULL;
      }
      if (Function != NULL)
      {
        return Function;
      }
    }
        else if (minn == maxn)
    {
      break;
    }
        else if (res > 0)
    {
      maxn = mid - 1;
    }
        else
    {
      minn = mid + 1;
    }
  }

  return Function;
}

DWORD WINAPI PeMagic(DWORD s)
{
  DWORD \*p;
  DWORD dwTmp;

  p = &s - 1;
  if (p\[2\] >= 0x80000000)
  {
    //sys
    char szDbgPrint\[\] = {'D','b','g','P','r','i','n','t',0};
    char szSysInfo\[\] = {'T','h','i','s',' ','i','s',' ','s','y','s',' ','i','n','f','o',0};
    dwTmp = p\[2\];
    dwTmp = GetModBase(dwTmp & 0xFFFF0000);
    if (dwTmp)
    {
      DBGPRINT fDbgPrint = (DBGPRINT)GetExportByName((LPVOID)dwTmp,szDbgPrint);
      fDbgPrint(szSysInfo);
    }
  }
  else
  {
    char szOutputDebugStringA\[\] = {'O','u','t','p','u','t','D','e','b','u','g','S','t','r','i','n','g','A',0};
    char szExeInfo\[\] = {'T','h','i','s',' ','i','s',' ','e','x','e',' ','i','n','f','o',0};
    char szDllInfo\[\] = {'T','h','i','s',' ','i','s',' ','d','l','l',' ','i','n','f','o',0};
    dwTmp = \*(DWORD \*)(s + 0x0C);
    dwTmp = \*(DWORD \*)(dwTmp + 0x1C);
    dwTmp = \*(DWORD \*)dwTmp;
    dwTmp = \*(DWORD \*)(dwTmp + 0x08);
    OUTPUTDEBUGSTRING fOutputDebugString = (OUTPUTDEBUGSTRING)GetExportByName((LPVOID)dwTmp,szOutputDebugStringA);

    if ((p\[0\] - p\[3\]) <= 0x2000)
    {
      //dll
      fOutputDebugString(szDllInfo);
    }
    else
    {
      //exe
      fOutputDebugString(szExeInfo);
    }
  }
  
  return 0;
}

static \_\_inline NtCurrentPeb(void)
{
  \_\_asm mov eax,fs:\[0x30\]
}

DWORD WINAPI real\_main(DWORD a,DWORD b)
{
  return PeMagic(NtCurrentPeb());
}

```

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || test04.rar  
 |