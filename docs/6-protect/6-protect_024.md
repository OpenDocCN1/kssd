
# Win7 下的 `GetProcAddress`

> 原文：[`www.pediy.com/kssd/pediy12/132743.html`](https://www.pediy.com/kssd/pediy12/132743.html)

最近在 `WIN7` 下跑原来写的一个壳。发现自己写的 `GetProcAddress` 出错。分析了 `ntdll` 里的 `LdrGetProcedureAddress` 函数。查了一些资料发现。`Win7` 引入了 `ApiSetMap` 的机制。它把一些系统低级别的一些 `API` 分为几个类型并且把 `kernel32.dll` 与 `advapi32.dll` 的一部分函数放到 `kernelbase.dll` 里。而归类的 `API` 名由 `api-ms-xxx` 字符串的 `dll` 库引入。这些 `DLL` 在系统上并不存在而只是一个索引类似的东西存在于引出表中。起一个重定向的问题。

在每一个进程的 `PEB+0x38` 偏移处就是 `ApiSetMap` 字段了。翻阅了一些文档，这个东西是存在于 `ApiSetMap.dll` 中的 `.apimap` 节中而此节是在 `WIN7` 启动时在启动 1 阶段被加载到系统空间，而 `0x38` 是这个的用户空间映射(这些我没有去验证,有时间的朋友可以自行去验证一下)。这个结构是这样的。

代码:

```
// ApiSetMap 结构
typedef struct _API_SET_MAP_HEADER {
  _dword dwVersionNumber;
  _dword dwNumberOfApiSetModules;
} API_SET_MAP_HEADER, *PAPI_SET_MAP_HEADER;

typedef struct _API_SET_MAP_ENTRY {
  _dword dwNameOfApiSetModuleRVA;
  _dword dwSizeOfName;
  _dword dwHostModulesRVA;
} API_SET_MAP_ENTRY, *PAPI_SET_MAP_ENTRY;

typedef struct _API_SET_MAP_HOST_HEADER {
  _dword dwNumberOfHosts;
} API_SET_MAP_HOST_HEADER, *PAPI_SET_MAP_HOST_HEADER;

typedef struct _API_SET_MAP_HOST_ENTRY {
  _dword dwNameOfImportModuleRVA;
  _dword dwSizeOfImportModuleName;
  _dword dwNameOfHostRVA;
  _dword dwSizeOfHostName;
} API_SET_MAP_HOST_ENTRY, *PAPI_SET_MAP_HOST_ENTRY;
```

由两个头 两个 `Entry` 组成。只有一个 `ApiSetMapHeader`,紧随其后的是 `ApiSetMapHeader.dwNumberOfApiSetModules` 个 `ApiSetMapEntry` 而在此结构 `ApiSetMapEntry.dwHostModulesRVA` 又是 `ApiSetMapHostHeader` 结构的相对于 `ApiSetMapHeader` 的偏移。而在 `ApiSetMapHostHeader` 下又有 `ApiSetMapHostHeader.dwNumberOfHosts` 个 `ApiSetMapHostEntry` 结构,在这个结构中存在着一些对应。现在发现最多是 2 个 `HostEntry` 结构,最少是 1 个。

而我们要找的就是 `ImportMoude` 的名字与我们原先加载 `DLL` 的名字一样下面的 `HostModule` 的名字。大多数都是 `kernel32.dll` -> `kernelbase.dll` 的映射。

如果只有 1 个 `HostEntry` 那么 `ImportModule` 名为空。而 `HostModule` 为 `kernel32.dll`，这个应该是为 `advapi32.dll` 准备的。`advapi32.dll` 的一部分函数被重定向到 `kernel32.dll` 里了。

我语言组织的不太好。下面直接给出代码，一看就明白了。

代码:



```c
// 获取 ApiSetMap
PAPI_SET_MAP_HEADER __API__ GetApiSetMapHeader() {
 PAPI_SET_MAP_HEADER pApiSetMap = NULL;
 __memory pPeb = NULL;
 // 检查是否是 Vista 以上系统
 if (GetWindowsVersion() != WIN_VISTA)
  return NULL;
 pPeb = (__memory)__NtCurrentPeb__();
 pApiSetMap = (PAPI_SET_MAP_HEADER)*((__address *)(pPeb + 0x38));
 return pApiSetMap;
}
 
// 私有函数这里声明一下,底下函数有要用
FARPROC __INTERNAL_FUNC__ xLdrGetExportByName(__memory pBaseAddress, __memory pHashPoint, __integer iHashSize, FPHashFunc pHashFunc, FARPROC fpLoadLibraryA);
FARPROC __INTERNAL_FUNC__ xLdrGetExportByOrdinal(__memory pBaseAddress, __word wOrdinal, FPHashFunc pHashFunc, FARPROC fpLoadLibraryA);
__INLINE__ FARPROC __INTERNAL_FUNC__ xLdrFixupForward(__memory pBaseAddress, __memory pForwardName, __word wOrdinal, FPHashFunc pHashFunc, FARPROC fpLoadLibraryA) {
 __char NameBuffer[128] = {0};
 __wchar NameOfLib[128] = {0};
 __memory pPoint = NULL;
 __memory hModule = NULL;
 __byte HashValue[1024] = {0};
 __integer iHashValueSize = 0;
 FARPROC pFunction = NULL;
 // 获取加载动态库的地址
 FPLoadLibraryA pLoadLibraryA = (FPLoadLibraryA)fpLoadLibraryA;
 // 取出 DLL 与引出的函数名
 __logic_strcpy__(NameBuffer, pForwardName);
 pPoint = __logic_strchr__(NameBuffer, '.');
 if (pPoint) {
  __char *pProcName = NULL;
  __integer iProcNameSize = 0;
  *pPoint = 0;//使用 0 字符将 DLL 与函数名隔开,替代'.'
  // 检查是否是 ApiSetMap 的函数
  __logic_str2lower__(NameBuffer);//转换为小写
  if (__logic_strncmp__(NameBuffer, "api-", 4) == 0) {
   // 寻找 API SET MAP
   PAPI_SET_MAP_HEADER pApiSetMapHeader = NULL;
   PAPI_SET_MAP_ENTRY pApiSetMapEntry = NULL;
   __dword dwNumberOfApiSetMapEntry = 0;
   __dword dwNumberOfHostEntry = 0;
   __wchar *pApiSetMapEntryName = NULL;
   __integer iSizeOfApiSetMapEntryName = 0;
   __wchar *pHostMapName = NULL;
   __integer iSizeOfHostMapName = 0;
   __wchar *pImportMapName = NULL;
   __integer iSizeOfImportMapName = 0;
   PAPI_SET_MAP_HOST_HEADER pApiSetMapHostHeader = NULL;
   PAPI_SET_MAP_HOST_ENTRY pApiSetMapHostEntry = NULL;
   __integer i = 0, j = 0;
   __char *pNameBuffer = NULL;
   // 获取要匹配的库名
   pNameBuffer = (__char *)(NameBuffer + 4);
   pApiSetMapHeader = GetApiSetMapHeader();
   if (!pApiSetMapHeader) return NULL;
   pApiSetMapEntry = (PAPI_SET_MAP_ENTRY)((__memory)pApiSetMapHeader + sizeof(API_SET_MAP_HEADER));
   dwNumberOfApiSetMapEntry = pApiSetMapHeader->dwNumberOfApiSetModules;
   for (i = 0; i < dwNumberOfApiSetMapEntry; i++, pApiSetMapEntry++) {
    __char AnsiNameOfLib[128] = {0};
    pApiSetMapEntryName = (__wchar *)((__memory)pApiSetMapHeader + pApiSetMapEntry->dwNameOfApiSetModuleRVA);
    iSizeOfApiSetMapEntryName = pApiSetMapEntry->dwSizeOfName;
    // 转换成小写
    __logic_memset__(NameOfLib, 0, sizeof(__wchar) * 128);
    __logic_memset__(AnsiNameOfLib, 0, 128);
    __logic_tcsncpy__(NameOfLib, pApiSetMapEntryName, iSizeOfApiSetMapEntryName);
    __logic_tcs2lower__n_(NameOfLib, iSizeOfApiSetMapEntryName / sizeof(__tchar));
    __logic_tcs2str__n__(NameOfLib, AnsiNameOfLib, iSizeOfApiSetMapEntryName / sizeof(__tchar));
    // 对比是否是要查找的 Lib
    if (__logic_strcmp__(AnsiNameOfLib, pNameBuffer) == 0) {
     __integer iCount = 0;
     PLDR_MODULE pLdrImportModule = NULL;
     __char szImportModuleName[128] = {0};
     PUNICODE_STRING pCurrImportName = NULL;
     pApiSetMapHostHeader = (PAPI_SET_MAP_HOST_HEADER)((__memory)pApiSetMapHeader + pApiSetMapEntry->dwHostModulesRVA);
     dwNumberOfHostEntry = pApiSetMapHostHeader->dwNumberOfHosts;
     pApiSetMapHostEntry = (PAPI_SET_MAP_HOST_ENTRY)((__memory)pApiSetMapHostHeader + sizeof(API_SET_MAP_HOST_HEADER));
     // 找到当前模块的引入模块
     pLdrImportModule = GetExistModuleInLoadModuleList(pBaseAddress);
     pCurrImportName = &(pLdrImportModule->BaseDllName);
     
     // 如果 dwNumberOfHostEntry == 1
     if (dwNumberOfHostEntry == 1) {
      // Host
      pHostMapName = (__wchar *)((__memory)pApiSetMapHeader + pApiSetMapHostEntry->dwNameOfHostRVA);
      iSizeOfHostMapName = pApiSetMapHostEntry->dwSizeOfHostName;
      iCount = iSizeOfHostMapName / sizeof(__tchar);
      // 转换
      __logic_tcs2str__n__(pHostMapName, NameBuffer, iCount);
      NameBuffer[iCount] = '\0';
      // 跳转到
      goto _load_library_get_address;
     } else {
      // 遍历 Host 映射
      for (j = 0; j < dwNumberOfHostEntry; j++, pApiSetMapHostEntry++) {
       __wchar ImportModule[128] = {0};
       __wchar ImportModule2[128] = {0};
       // Import
       pImportMapName = (__wchar *)((__memory)pApiSetMapHeader + pApiSetMapHostEntry->dwNameOfImportModuleRVA);
       iSizeOfImportMapName = pApiSetMapHostEntry->dwSizeOfImportModuleName;
       iCount = iSizeOfImportMapName / sizeof(__tchar);
       // 如果引入模块名长度为 0
       if (iSizeOfImportMapName == 0)
        continue;
       // Host
       pHostMapName = (__wchar *)((__memory)pApiSetMapHeader + pApiSetMapHostEntry->dwNameOfHostRVA);
       iSizeOfHostMapName = pApiSetMapHostEntry->dwSizeOfHostName;
       __logic_tcsncpy__(ImportModule, pImportMapName, iCount);
       __logic_tcs2lower__n_(ImportModule, iCount);
       ImportModule[iCount] = _T('\0');
       __logic_tcsncpy__(ImportModule2, pCurrImportName->Buffer, iCount);
       __logic_tcs2lower__n_(ImportModule2, iCount);
       ImportModule2[iCount] = _T('\0');
       // 比对 Import 模块的名字, 如果相等则获取
       if (__logic_tcsncmp__(pImportMapName, ImportModule2, iCount) == 0) {
        iCount = iSizeOfHostMapName / sizeof(__tchar);
        __logic_tcs2str__n__(pHostMapName, NameBuffer, iCount);
        NameBuffer[iCount] = '\0';
        goto _load_library_get_address;
       }/* end if */
      }/* end for */
     }/* end else */
    }/* end if */
   }/* end for */
  } else {
_load_library_get_address:
   // 加载动态库
   hModule = (__memory)pLoadLibraryA(NameBuffer);
   if (!hModule) return NULL;
   // 产生哈希值
   pProcName = (__char *)(pPoint + 1);
   iProcNameSize = __logic_strlen__(pPoint + 1);
   iHashValueSize = pHashFunc((__memory)pProcName, iProcNameSize, HashValue);
   return xLdrGetExportByName((__memory)hModule, (__memory)HashValue, iHashValueSize, pHashFunc, fpLoadLibraryA);
  }/* end else */
 }/* end if */
 return NULL;
}
FARPROC __INTERNAL_FUNC__ xLdrGetExportByOrdinal(__memory pBaseAddress, __word wOrdinal, FPHashFunc pHashFunc, FARPROC fpLoadLibraryA) {
 PIMAGE_EXPORT_DIRECTORY pExportDir;
 __integer iExportDirSize;
 __dword **pExFunctionsPoint;
 FARPROC pFunction;
 PIMAGE_DATA_DIRECTORY pExportDataDirectory = ExistDataDirectory(pBaseAddress, IMAGE_DIRECTORY_ENTRY_EXPORT);
 pExportDir = (PIMAGE_EXPORT_DIRECTORY)(pBaseAddress + pExportDataDirectory->VirtualAddress);
 if (!pExportDir) 
  return NULL;
 iExportDirSize = pExportDataDirectory->Size;
 pExFunctionsPoint = (__dword **)__RvaToVa__(pBaseAddress, pExportDir->AddressOfFunctions);
 pFunction = (FARPROC)(0 != pExFunctionsPoint[wOrdinal - pExportDir->Base]
    ? __RvaToVa__(pBaseAddress, pExFunctionsPoint[wOrdinal - pExportDir->Base])
    : NULL);
 if (((__address)pFunction >= (__address)pExportDir) &&
  ((__address)pFunction < (__address)pExportDir + (__address)iExportDirSize))
  pFunction = xLdrFixupForward(pBaseAddress, (__memory)pFunction, wOrdinal, pHashFunc, fpLoadLibraryA);
 return pFunction;
}
FARPROC __INTERNAL_FUNC__ xLdrGetExportByName(__memory pBaseAddress, __memory pHashPoint, __integer iHashSize, FPHashFunc pHashFunc, FARPROC fpLoadLibraryA) {
 __word wOrdinal = 0;
 __integer iDirCount = 0;
 __address *pAddrTable = NULL;
 __address addrAddr = 0;
 __offset ofRVA = 0;
 __integer iExpDataSize = 0;
 __integer i = 0;
 PIMAGE_EXPORT_DIRECTORY pEd = NULL;
 PIMAGE_NT_HEADERS pNt = NULL;
 PIMAGE_DATA_DIRECTORY pExportDataDirectory = NULL;
 if (pBaseAddress == NULL) return NULL;
 pNt = GetNtHeader(pBaseAddress);
 iDirCount = pNt->OptionalHeader.NumberOfRvaAndSizes;
 if (iDirCount < IMAGE_NUMBEROF_DIRECTORY_ENTRIES) return FALSE;
 pExportDataDirectory = ExistDataDirectory(pBaseAddress, IMAGE_DIRECTORY_ENTRY_EXPORT);
 if(!pExportDataDirectory) 
  return NULL;//确定引出表
 iExpDataSize = pExportDataDirectory->Size;
 // 从引出表获取函数地址
 pEd = (PIMAGE_EXPORT_DIRECTORY)__RvaToVa__(pBaseAddress, pExportDataDirectory->VirtualAddress); 
 /* 
 * 获取序数引出的函数
 */
 if (HIWORD((__dword)pHashPoint)==0) {//以序号引出
  wOrdinal = (__word)(LOWORD((__dword)pHashPoint)) - pEd->Base;
 } else {
  __integer iCount = 0;
  __dword *pdwNamePtr = NULL;
  __word *pwOrdinalPtr = NULL;
  iCount = (__integer)(pEd->NumberOfNames);
  pdwNamePtr = (__dword *)__RvaToVa__(pBaseAddress, pEd->AddressOfNames);
  pwOrdinalPtr = (__word *)__RvaToVa__(pBaseAddress, pEd->AddressOfNameOrdinals);
  for(i = 0; i < iCount; i++) {
   __byte HashValue[1024];
   __char *svName = NULL;
   __integer iHashValueSize = 0;
   svName = (__char *)__RvaToVa__(pBaseAddress, *pdwNamePtr);
   iHashValueSize = pHashFunc(svName, __logic_strlen__(svName), HashValue);//进行哈希计算
   if (iHashValueSize == iHashSize) {
    if (__logic_memcmp__(HashValue, pHashPoint, iHashSize) == 0) {
     wOrdinal = *pwOrdinalPtr;
     break;
    }
   }
   pdwNamePtr++;
   pwOrdinalPtr++;
  }
  if (i == iCount) return NULL;
}
 pAddrTable=(__address *)__RvaToVa__(pBaseAddress, pEd->AddressOfFunctions);
 ofRVA = pAddrTable[wOrdinal];
 addrAddr = (__address)__RvaToVa__(pBaseAddress, ofRVA);
 /*
  * 最终判断是否是中间跳转
  */
 if (((__address)addrAddr >= (__address)pEd) &&
  ((__address)addrAddr < (__address)pEd + (__address)iExpDataSize))
  return xLdrFixupForward(pBaseAddress, (__memory)addrAddr, wOrdinal, pHashFunc, fpLoadLibraryA);
 return (FARPROC)addrAddr;
}
__INLINE__ FARPROC __INTERNAL_FUNC__ xLdrGetProcedureAddress(__memory pBaseAddress, __memory pHashPoint, __integer iHashSize, FPHashFunc pHashFunc, FARPROC fpLoadLibraryA) {
 FARPROC pProcedureAddress = NULL;
 __dword dwOrdinal = (__dword)pHashPoint;
 if (HIWORD((__dword)pHashPoint)) {
  // 通过名字引出
  pProcedureAddress = xLdrGetExportByName(pBaseAddress, pHashPoint, iHashSize, pHashFunc, fpLoadLibraryA);
 } else {
  // 通过序数
  dwOrdinal &= 0x0000FFFF;
  pProcedureAddress = xLdrGetExportByOrdinal(pBaseAddress, (__word)dwOrdinal, pHashFunc, fpLoadLibraryA);
 }
 return pProcedureAddress;
}
```



我逆向的是 `win7` 下 `ntdll` 中的位于 `0x77F1E681` -> `0x77F1E6F1` -> `0x77F22193` 这三个函数就是主要 `ntdll` 实现对 `ApiSetMap` 的解析了。

一个邪恶的想法是是否可以在内核里直接劫持这片空间。把他们的映射关系修改我们自己的 `DLL`。谁有时间研究一下可行性。![](img/c1d8523a7842d7b67917c210ac6dcf18.jpg "微笑")
