

# 内存加载 user32.dll 运行 附源码

> 原文：[`www.pediy.com/kssd/pediy12/140650.html`](https://www.pediy.com/kssd/pediy12/140650.html)

看到有人说 LoadLibrary("user32.dll")，360 要报。(我自己试了下，没报饿。不知道为啥);  
  
于是就有想可不可以内存加载然后运行。  
思路很简单：  
1：以 PE 方式把 user32.dll(或者其它 dll)COPY 到程序中。  
2：通过重写位表，进行重定位。  
3：解决输入表问题。  
4：DLL 初始化一下。  
  
完成上面 4 步就可以调用了。我测试的 MessageBoxA，没有问题。  
后来仔细想想 360 肯定是 HOOK 的内核函数，所以好像这样没用。![](img/906f120cfe711ccbc8463fd6eb86eee9.jpg "尴尬")  

代码:

```
// CopySysDll.cpp : Defines the entry point for the console application.
//

#include "stdafx.h"
#include <Windows.h>
#include <TCHAR.H>
#include <STDLIB.H>

DWORD g\_dwRelocSecAddr = 0;
DWORD g\_dwOriginalLoadBase = 0;
DWORD g\_dwRelocSecSize = 0;
DWORD g\_dwDllMain = 0;
BOOL IsPeFromHandle( HANDLE hFile )
{
  DWORD dwReadBuf;
  DWORD dwPeFlagOffset;
  // 读 PE 头 RAW
  SetFilePointer( hFile, 0x3c, NULL, FILE\_BEGIN );
  ReadFile( hFile, &dwPeFlagOffset, 4, &dwReadBuf, NULL );
  // 读取 PE 标志
  SetFilePointer( hFile, dwPeFlagOffset, NULL, FILE\_BEGIN );
  ReadFile( hFile, &dwPeFlagOffset, 2, &dwReadBuf, NULL );
  if ( 0x4550 == dwPeFlagOffset )
    return TRUE;
  else
    return FALSE;
}

DWORD LoadFile( HANDLE hFile )
{
  DWORD dwNumOfSections;
  DWORD dwPeFlagOffset;      // PE 文件标志偏移
  DWORD dwReadBuf;       // 实际读取的字节数
  DWORD dwSizeOfHeaders;         // PE 头的总大小
  DWORD dwSizeOfImage;

  if ( !IsPeFromHandle( hFile ) )
    return 0;
  
  // 读 PE 头 RAW
  SetFilePointer( hFile, 0x3c, NULL, FILE\_BEGIN );
  ReadFile( hFile, &dwPeFlagOffset, 4, &dwReadBuf, NULL );
  // 读取映像装入内存后总尺寸
  SetFilePointer( hFile, dwPeFlagOffset+0x50, NULL, FILE\_BEGIN ); 
  ReadFile( hFile, &dwSizeOfImage, 4, &dwReadBuf, NULL );
  // 读取 pe 头的总大小
  SetFilePointer( hFile, dwPeFlagOffset+0x54, NULL, FILE\_BEGIN );
  ReadFile( hFile, &dwSizeOfHeaders, 4, &dwReadBuf, NULL );
  
  // 分配内存来装载文件
   LPVOID lpBase = VirtualAlloc( NULL, dwSizeOfImage, MEM\_COMMIT | MEM\_RESERVE, PAGE\_EXECUTE\_READWRITE );
  
  // 读 PE 文件头
  SetFilePointer( hFile, 0, NULL, FILE\_BEGIN );
  ReadFile( hFile, lpBase, dwSizeOfHeaders, &dwReadBuf, NULL );
  // 得到 PE 文件相关指针
  PIMAGE\_DOS\_HEADER pDosHeaders = (PIMAGE\_DOS\_HEADER) lpBase;
  PIMAGE\_NT\_HEADERS pNtHeaders = (PIMAGE\_NT\_HEADERS) ( pDosHeaders->e\_lfanew + (DWORD) pDosHeaders );
  PIMAGE\_OPTIONAL\_HEADER pOptionalHeader = (PIMAGE\_OPTIONAL\_HEADER) &( pNtHeaders->OptionalHeader );
  PIMAGE\_SECTION\_HEADER pSecHaeder = IMAGE\_FIRST\_SECTION( pNtHeaders );
  g\_dwOriginalLoadBase = pOptionalHeader->ImageBase; 
  g\_dwDllMain = pOptionalHeader->AddressOfEntryPoint + (DWORD)lpBase;
  // 分区块读入内存
  dwNumOfSections = pNtHeaders->FileHeader.NumberOfSections;
  for ( DWORD i = 0; i < dwNumOfSections; i++ )
  {
    DWORD dwOff = SetFilePointer( hFile, (pSecHaeder + i)->PointerToRawData, NULL, FILE\_BEGIN );
    BOOL bret = ReadFile( hFile, LPVOID( (pSecHaeder+ i)->VirtualAddress + (DWORD)lpBase), (pSecHaeder+i)->SizeOfRawData,
      &dwReadBuf, NULL );
    if ( i == dwNumOfSections - 1 )
    {
      g\_dwRelocSecAddr = (DWORD) ( (pSecHaeder+ i)->VirtualAddress + (DWORD)lpBase);
      g\_dwRelocSecSize = (DWORD) ( (pSecHaeder + i)->Misc.VirtualSize );
    }
      
  }  
  return (DWORD)lpBase;
}

// 修正重定位表
void Relocation( DWORD dwBase )
{
  // 得到实际加载地址和默认加载地址的差值
  DWORD dwNum;
  dwNum = dwBase - g\_dwOriginalLoadBase;
  
  DWORD dwMax = g\_dwRelocSecAddr + g\_dwRelocSecSize;

  while ( g\_dwRelocSecAddr < dwMax )
  {
    DWORD dwRelocRva = \*( (DWORD\*) g\_dwRelocSecAddr );
    g\_dwRelocSecAddr += 4; 
    DWORD dwRelocSize = \*( (DWORD\*) g\_dwRelocSecAddr );
    g\_dwRelocSecAddr += 4;
    // 去悼基址和大小占用的字节，一个重定位数据是 2 字节。/2 得到这组重定个数
    dwRelocSize -= 8;
    dwRelocSize /= 2;
    

    WORD wRelocInfo= 0;             // 重定位信息
    WORD wBak = 0;      
    DWORD dwDataAddr = 0;      // 重定位数据的地址
    DWORD dwData = 0;               // 重定位地址的内容
    for ( DWORD i = 0; i < dwRelocSize; i++ )
    {
      wRelocInfo = \*(WORD\*)g\_dwRelocSecAddr;
      g\_dwRelocSecAddr += 2;

      wBak = wRelocInfo;
      wBak = wBak>>12;
      
      // IMAGE\_REL\_BASED\_HIGHLOW 常量是 3,表示重定位整个地址都要修正
      if ( wBak == IMAGE\_REL\_BASED\_HIGHLOW )
      {
        // 得到低 12 位
        wRelocInfo = wRelocInfo & 0x0fff;
        dwDataAddr = dwBase + wRelocInfo + dwRelocRva;
        // 读出数据
        dwData = \*(DWORD\*)dwDataAddr;
        dwData = dwData + dwNum;
        // 写入数据，完成重定位
        \*(DWORD\*)dwDataAddr = dwData;
      }
    }
  }
}

// 修正输入表函数地址
void UpdateImportTableAddress( DWORD dwBase )
{
  // 得到输入表 IID 结构
  // 得到 PE 文件相关指针
  PIMAGE\_DOS\_HEADER pDosHeaders = (PIMAGE\_DOS\_HEADER) dwBase;
  PIMAGE\_NT\_HEADERS pNtHeaders = (PIMAGE\_NT\_HEADERS) ( pDosHeaders->e\_lfanew + (DWORD) pDosHeaders );
  PIMAGE\_OPTIONAL\_HEADER pOptionalHeader = (PIMAGE\_OPTIONAL\_HEADER) &( pNtHeaders->OptionalHeader );
  PIMAGE\_SECTION\_HEADER pSecHaeder = IMAGE\_FIRST\_SECTION( pNtHeaders );
  
  // 得到输入表
  PIMAGE\_DATA\_DIRECTORY pImaDataDir = &( pOptionalHeader->DataDirectory\[1\] );
  PIMAGE\_IMPORT\_DESCRIPTOR lpImportTab =(PIMAGE\_IMPORT\_DESCRIPTOR) LPVOID ( pImaDataDir->VirtualAddress + dwBase );
  PIMAGE\_IMPORT\_DESCRIPTOR lpImportTabBak = lpImportTab;
  
  DWORD dwNameAddr = 0;
  DWORD dwFunAddr = 0;

  while ( 0 != lpImportTab->Name )
  {
    HMODULE hMod = GetModuleHandle( (LPCTSTR)(DWORD\*)(dwBase+lpImportTab->Name) );
    if ( !hMod )
    {
      hMod = LoadLibrary( (LPCTSTR)(DWORD\*)(dwBase+lpImportTab->Name) );
    }
      
    // 取得 INT OR IAT 结构的 RAV
    DWORD\* pdwINTRav;
    DWORD\* pdwIATRav;
    pdwIATRav = (DWORD\*) ( (DWORD)dwBase+ lpImportTab->FirstThunk );
    if ( lpImportTab->OriginalFirstThunk != 0 )
      pdwINTRav = (DWORD\*) ( (DWORD)dwBase+ lpImportTab->OriginalFirstThunk );
    else
      pdwINTRav = (DWORD\*) ( (DWORD)dwBase+ lpImportTab->FirstThunk );
    
    // 函数名方式得到地址
    while ( 0 != \*pdwINTRav )
    {
      if ( 0x80000000 > \*pdwINTRav )
      {
          // dwNameAddr 是 IMAGE\_IMPORT\_BY\_NAME 结构地址，+2 的地方就是函数名字串
          dwNameAddr = \*pdwINTRav + dwBase;
          dwFunAddr = (DWORD)GetProcAddress( hMod, (LPCTSTR)(DWORD\*)(dwNameAddr+2) );
          \*pdwIATRav = dwFunAddr;
          ++pdwINTRav;
          ++pdwIATRav;
      }
      // 序列号方式
      else
      {  
        // 得到函数序列号
        DWORD dwSerialNunOfFun;
        dwSerialNunOfFun = \*pdwINTRav ^ 0x80000000;
        DWORD dwFunAddr = (DWORD)GetProcAddress( hMod, (LPCTSTR)(DWORD\*)dwSerialNunOfFun );
        \*pdwIATRav = dwFunAddr;
        ++pdwINTRav;
        ++pdwIATRav;
      }  // endif else  
    } // end while
    ++lpImportTab;
  }  // end while
}

FARPROC MyGetProcAddress(HMODULE hModule, LPCSTR lpProcName)
{
  PIMAGE\_DOS\_HEADER pDOSHeader;
  PIMAGE\_NT\_HEADERS pNTHeader;
  PIMAGE\_EXPORT\_DIRECTORY pExportDir;
  LPCSTR \*pFunctionName; 
  LPCSTR pszFunName;
  LPDWORD pFunction;
  LPWORD pIndex;
  DWORD n; 
  FARPROC ret = NULL;
  if ((INVALID\_HANDLE\_VALUE == hModule) || (NULL == lpProcName))
  {
    goto end;
  }
  pDOSHeader = (PIMAGE\_DOS\_HEADER)hModule;
  if (pDOSHeader->e\_magic != IMAGE\_DOS\_SIGNATURE)  
  {
    goto end;
  }
  pNTHeader = (PIMAGE\_NT\_HEADERS)((DWORD)pDOSHeader + (DWORD)pDOSHeader->e\_lfanew);
  if (pNTHeader->Signature != IMAGE\_NT\_SIGNATURE) 
  {
    goto end;
  }
  pExportDir = (PIMAGE\_EXPORT\_DIRECTORY)((DWORD)pDOSHeader + \\
    (DWORD)pNTHeader->OptionalHeader.DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_EXPORT\].VirtualAddress);
  if ((DWORD)pExportDir == (DWORD)pDOSHeader) 
  {
    goto end;
  }
  pFunctionName = (LPCSTR \*)((DWORD)pExportDir->AddressOfNames + (DWORD)pDOSHeader);
  pFunction = (LPDWORD)((DWORD)pExportDir->AddressOfFunctions + (DWORD)pDOSHeader);
  pIndex = (LPWORD)((DWORD)pExportDir->AddressOfNameOrdinals + (DWORD)pDOSHeader);
  n = pExportDir->NumberOfNames;
  while (n--)
  {
    pszFunName = (LPCSTR)((DWORD)\*pFunctionName + (DWORD)pDOSHeader);
    if (strcmp(pszFunName, lpProcName) == 0)
    {
      ret = (FARPROC)(pFunction\[\*pIndex\] + (DWORD)pDOSHeader);
      break;
    }
    pFunctionName++;
    pIndex++;
  }
end:  
  return ret;
}

typedef int ( \_\_stdcall \*MyMessageBoxA)( HANDLE, LPCSTR, LPCSTR, UINT );

int main(int argc, char\* argv\[\])
{
  MessageBox( NULL, \_T("现在我在 user.dll 中"), \_T("提示"), MB\_OK );
  TCHAR szSysPathBuffer\[MAX\_PATH\];
  int bRet = GetSystemDirectory( szSysPathBuffer, MAX\_PATH );
  if ( 0 != bRet )
  {
    \_tcscat( szSysPathBuffer, \_T("\\\\User32.dll") );
  }
  HANDLE hFile = CreateFile( szSysPathBuffer, GENERIC\_READ, FILE\_SHARE\_READ, NULL, OPEN\_EXISTING, FILE\_ATTRIBUTE\_NORMAL, NULL );
  LPVOID lpBase = (LPVOID)LoadFile( hFile );
  // 修正重定位
  Relocation( (DWORD)lpBase );
  // 修正输入表
  UpdateImportTableAddress( (DWORD)lpBase );
  // DLL 初始化
  \_\_asm
  {
    pushad
    push 0
    push DLL\_PROCESS\_ATTACH
    push lpBase
    mov eax, g\_dwDllMain
    call eax
    popad
  }
  
  // 哈哈全部完成现在测试下
  HMODULE hMod = (HMODULE)(DWORD)lpBase;
  MyMessageBoxA Fun;
  Fun = (MyMessageBoxA)MyGetProcAddress( hMod, \_T("MessageBoxA") );  
  TCHAR szText\[MAX\_PATH\] = {0};
  TCHAR szTemp\[10\] = {0};
  itoa( (DWORD)Fun, szTemp, 16 );
  memcpy( szText, \_T("现在 MessageBoxA 在: "), MAX\_PATH );
  \_tcscat( szText, szTemp );
  Fun( NULL, szText, \_T("成功了"), MB\_OK );

  printf("Hello World!\\n");
  return 0;
}

```

上传的附件
| ![](img/31a2c422ae120fd0b6c241a4e6e83632.jpg) || CopySysDll.7z  
 |