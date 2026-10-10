

# Win7 下的 GetProcAddress

> 原文：[`www.pediy.com/kssd/pediy12/132743.html`](https://www.pediy.com/kssd/pediy12/132743.html)

最近在 WIN7 下跑原来写的一个壳。发现自己写的 GetProcAddress 出错。分析了 ntdll 里的 LdrGetProcedureAddress 函数。查了一些资料发现。Win7 引入了 ApiSetMap 的机制。它把一些系统低级别的一些 API 分为几个类型并且把 kernel32.dll 与 advapi32.dll 的一部分函数放到 kernelbase.dll 里。而归类的 API 名由 api-ms-xxx 字符串的 dll 库引入。这些 DLL 在系统上并不存在而只是一个索引类似的东西存在于引出表中。起一个重定向的问题。  
在每一个进程的 PEB+0x38 偏移处就是 ApiSetMap 字段了。翻阅了一些文档，这个东西是存在于 ApiSetMap.dll 中的.apimap 节中而此节是在 WIN7 启动时在启动 1 阶段被加载到系统空间，而 0x38 是这个的用户空间映射(这些我没有去验证,有时间的朋友可以自行去验证一下)。这个结构是这样的。  

代码:

```
// ApiSetMap 结构
typedef struct \_API\_SET\_MAP\_HEADER {
 \_\_dword dwVersionNumber;
 \_\_dword dwNumberOfApiSetModules;
} API\_SET\_MAP\_HEADER, \*PAPI\_SET\_MAP\_HEADER;
 
typedef struct \_API\_SET\_MAP\_ENTRY {
 \_\_dword dwNameOfApiSetModuleRVA;
 \_\_dword dwSizeOfName;
 \_\_dword dwHostModulesRVA;
} API\_SET\_MAP\_ENTRY, \*PAPI\_SET\_MAP\_ENTRY;
 
typedef struct \_API\_SET\_MAP\_HOST\_HEADER {
 \_\_dword dwNumberOfHosts;
} API\_SET\_MAP\_HOST\_HEADER, \*PAPI\_SET\_MAP\_HOST\_HEADER;
 
typedef struct \_API\_SET\_MAP\_HOST\_ENTRY {
 \_\_dword dwNameOfImportModuleRVA;
 \_\_dword dwSizeOfImportModuleName;
 \_\_dword dwNameOfHostRVA;
 \_\_dword dwSizeOfHostName;
} API\_SET\_MAP\_HOST\_ENTRY, \*PAPI\_SET\_MAP\_HOST\_ENTRY;

```

由两个头 两个 Entry 组成。只有一个 ApiSetMapHeader,紧随其后的是 ApiSetMapHeader.dwNumberOfApiSetModules 个 ApiSetMapEntry 而在此结构 ApiSetMapEntry.dwHostModulesRVA 又是 ApiSetMapHostHeader 结构的相对于 ApiSetMapHeader 的偏移。而在 ApiSetMapHostHeader 下又有 ApiSetMapHostHeader.dwNumberOfHosts 个 ApiSetMapHostEntry 结构,在这个结构中存在着一些对应。现在发现最多是 2 个 HostEntry 结构,最少是 1 个。  
  
而我们要找的就是 ImportMoude 的名字与我们原先加载 DLL 的名字一样下面的 HostModule 的名字。大多数都是 kernel32.dll -> kernelbase.dll 的映射。  
如果只有 1 个 HostEntry 那么 ImportModule 名为空。而 HostModule 为 kernel32.dll，这个应该是为 advapi32.dll 准备的。advapi32.dll 的一部分函数被重定向到 kernel32.dll 里了。   
  
我语言组织的不太好。下面直接给出代码，一看就明白了。  
  
代码:

```
// 获取 ApiSetMap
PAPI\_SET\_MAP\_HEADER \_\_API\_\_ GetApiSetMapHeader() {
 PAPI\_SET\_MAP\_HEADER pApiSetMap = NULL;
 \_\_memory pPeb = NULL;
 // 检查是否是 Vista 以上系统
 if (GetWindowsVersion() != WIN\_VISTA)
  return NULL;
 pPeb = (\_\_memory)\_\_NtCurrentPeb\_\_();
 pApiSetMap = (PAPI\_SET\_MAP\_HEADER)\*((\_\_address \*)(pPeb + 0x38));
 return pApiSetMap;
}
 
// 私有函数这里声明一下,底下函数有要用
FARPROC \_\_INTERNAL\_FUNC\_\_ xLdrGetExportByName(\_\_memory pBaseAddress, \_\_memory pHashPoint, \_\_integer iHashSize, FPHashFunc pHashFunc, FARPROC fpLoadLibraryA);
FARPROC \_\_INTERNAL\_FUNC\_\_ xLdrGetExportByOrdinal(\_\_memory pBaseAddress, \_\_word wOrdinal, FPHashFunc pHashFunc, FARPROC fpLoadLibraryA);
\_\_INLINE\_\_ FARPROC \_\_INTERNAL\_FUNC\_\_ xLdrFixupForward(\_\_memory pBaseAddress, \_\_memory pForwardName, \_\_word wOrdinal, FPHashFunc pHashFunc, FARPROC fpLoadLibraryA) {
 \_\_char NameBuffer\[128\] = {0};
 \_\_wchar NameOfLib\[128\] = {0};
 \_\_memory pPoint = NULL;
 \_\_memory hModule = NULL;
 \_\_byte HashValue\[1024\] = {0};
 \_\_integer iHashValueSize = 0;
 FARPROC pFunction = NULL;
 // 获取加载动态库的地址
 FPLoadLibraryA pLoadLibraryA = (FPLoadLibraryA)fpLoadLibraryA;
 // 取出 DLL 与引出的函数名
 \_\_logic\_strcpy\_\_(NameBuffer, pForwardName);
 pPoint = \_\_logic\_strchr\_\_(NameBuffer, '.');
 if (pPoint) {
  \_\_char \*pProcName = NULL;
  \_\_integer iProcNameSize = 0;
  \*pPoint = 0;//使用 0 字符将 DLL 与函数名隔开,替代'.'
  // 检查是否是 ApiSetMap 的函数
  \_\_logic\_str2lower\_\_(NameBuffer);//转换为小写
  if (\_\_logic\_strncmp\_\_(NameBuffer, "api-", 4) == 0) {
   // 寻找 API SET MAP
   PAPI\_SET\_MAP\_HEADER pApiSetMapHeader = NULL;
   PAPI\_SET\_MAP\_ENTRY pApiSetMapEntry = NULL;
   \_\_dword dwNumberOfApiSetMapEntry = 0;
   \_\_dword dwNumberOfHostEntry = 0;
   \_\_wchar \*pApiSetMapEntryName = NULL;
   \_\_integer iSizeOfApiSetMapEntryName = 0;
   \_\_wchar \*pHostMapName = NULL;
   \_\_integer iSizeOfHostMapName = 0;
   \_\_wchar \*pImportMapName = NULL;
   \_\_integer iSizeOfImportMapName = 0;
   PAPI\_SET\_MAP\_HOST\_HEADER pApiSetMapHostHeader = NULL;
   PAPI\_SET\_MAP\_HOST\_ENTRY pApiSetMapHostEntry = NULL;
   \_\_integer i = 0, j = 0;
   \_\_char \*pNameBuffer = NULL;
   // 获取要匹配的库名
   pNameBuffer = (\_\_char \*)(NameBuffer + 4);
   pApiSetMapHeader = GetApiSetMapHeader();
   if (!pApiSetMapHeader) return NULL;
   pApiSetMapEntry = (PAPI\_SET\_MAP\_ENTRY)((\_\_memory)pApiSetMapHeader + sizeof(API\_SET\_MAP\_HEADER));
   dwNumberOfApiSetMapEntry = pApiSetMapHeader->dwNumberOfApiSetModules;
   for (i = 0; i < dwNumberOfApiSetMapEntry; i++, pApiSetMapEntry++) {
    \_\_char AnsiNameOfLib\[128\] = {0};
    pApiSetMapEntryName = (\_\_wchar \*)((\_\_memory)pApiSetMapHeader + pApiSetMapEntry->dwNameOfApiSetModuleRVA);
    iSizeOfApiSetMapEntryName = pApiSetMapEntry->dwSizeOfName;
    // 转换成小写
    \_\_logic\_memset\_\_(NameOfLib, 0, sizeof(\_\_wchar) \* 128);
    \_\_logic\_memset\_\_(AnsiNameOfLib, 0, 128);
    \_\_logic\_tcsncpy\_\_(NameOfLib, pApiSetMapEntryName, iSizeOfApiSetMapEntryName);
    \_\_logic\_tcs2lower\_n\_(NameOfLib, iSizeOfApiSetMapEntryName / sizeof(\_\_tchar));
    \_\_logic\_tcs2str\_n\_\_(NameOfLib, AnsiNameOfLib, iSizeOfApiSetMapEntryName / sizeof(\_\_tchar));
    // 对比是否是要查找的 Lib
    if (\_\_logic\_strcmp\_\_(AnsiNameOfLib, pNameBuffer) == 0) {
     \_\_integer iCount = 0;
     PLDR\_MODULE pLdrImportModule = NULL;
     \_\_char szImportModuleName\[128\] = {0};
     PUNICODE\_STRING pCurrImportName = NULL;
     pApiSetMapHostHeader = (PAPI\_SET\_MAP\_HOST\_HEADER)((\_\_memory)pApiSetMapHeader + pApiSetMapEntry->dwHostModulesRVA);
     dwNumberOfHostEntry = pApiSetMapHostHeader->dwNumberOfHosts;
     pApiSetMapHostEntry = (PAPI\_SET\_MAP\_HOST\_ENTRY)((\_\_memory)pApiSetMapHostHeader + sizeof(API\_SET\_MAP\_HOST\_HEADER));
     // 找到当前模块的引入模块
     pLdrImportModule = GetExistModuleInLoadModuleList(pBaseAddress);
     pCurrImportName = &(pLdrImportModule->BaseDllName);
     
     // 如果 dwNumberOfHostEntry == 1
     if (dwNumberOfHostEntry == 1) {
      // Host
      pHostMapName = (\_\_wchar \*)((\_\_memory)pApiSetMapHeader + pApiSetMapHostEntry->dwNameOfHostRVA);
      iSizeOfHostMapName = pApiSetMapHostEntry->dwSizeOfHostName;
      iCount = iSizeOfHostMapName / sizeof(\_\_tchar);
      // 转换
      \_\_logic\_tcs2str\_n\_\_(pHostMapName, NameBuffer, iCount);
      NameBuffer\[iCount\] = '\\0';
      // 跳转到
      goto \_load\_library\_get\_address;
     } else {
      // 遍历 Host 映射
      for (j = 0; j < dwNumberOfHostEntry; j++, pApiSetMapHostEntry++) {
       \_\_wchar ImportModule\[128\] = {0};
       \_\_wchar ImportModule2\[128\] = {0};
       // Import
       pImportMapName = (\_\_wchar \*)((\_\_memory)pApiSetMapHeader + pApiSetMapHostEntry->dwNameOfImportModuleRVA);
       iSizeOfImportMapName = pApiSetMapHostEntry->dwSizeOfImportModuleName;
       iCount = iSizeOfImportMapName / sizeof(\_\_tchar);
       // 如果引入模块名长度为 0
       if (iSizeOfImportMapName == 0)
        continue;
       // Host
       pHostMapName = (\_\_wchar \*)((\_\_memory)pApiSetMapHeader + pApiSetMapHostEntry->dwNameOfHostRVA);
       iSizeOfHostMapName = pApiSetMapHostEntry->dwSizeOfHostName;
       \_\_logic\_tcsncpy\_\_(ImportModule, pImportMapName, iCount);
       \_\_logic\_tcs2lower\_n\_(ImportModule, iCount);
       ImportModule\[iCount\] = \_T('\\0');
       \_\_logic\_tcsncpy\_\_(ImportModule2, pCurrImportName->Buffer, iCount);
       \_\_logic\_tcs2lower\_n\_(ImportModule2, iCount);
       ImportModule2\[iCount\] = \_T('\\0');
       // 比对 Import 模块的名字, 如果相等则获取
       if (\_\_logic\_tcsncmp\_\_(pImportMapName, ImportModule2, iCount) == 0) {
        iCount = iSizeOfHostMapName / sizeof(\_\_tchar);
        \_\_logic\_tcs2str\_n\_\_(pHostMapName, NameBuffer, iCount);
        NameBuffer\[iCount\] = '\\0';
        goto \_load\_library\_get\_address;
       }/\* end if \*/
      }/\* end for \*/
     }/\* end else \*/
    }/\* end if \*/
   }/\* end for \*/
  } else {
\_load\_library\_get\_address:
   // 加载动态库
   hModule = (\_\_memory)pLoadLibraryA(NameBuffer);
   if (!hModule) return NULL;
   // 产生哈希值
   pProcName = (\_\_char \*)(pPoint + 1);
   iProcNameSize = \_\_logic\_strlen\_\_(pPoint + 1);
   iHashValueSize = pHashFunc((\_\_memory)pProcName, iProcNameSize, HashValue);
   return xLdrGetExportByName((\_\_memory)hModule, (\_\_memory)HashValue, iHashValueSize, pHashFunc, fpLoadLibraryA);
  }/\* end else \*/
 }/\* end if \*/
 return NULL;
}
FARPROC \_\_INTERNAL\_FUNC\_\_ xLdrGetExportByOrdinal(\_\_memory pBaseAddress, \_\_word wOrdinal, FPHashFunc pHashFunc, FARPROC fpLoadLibraryA) {
 PIMAGE\_EXPORT\_DIRECTORY pExportDir;
 \_\_integer iExportDirSize;
 \_\_dword \*\*pExFunctionsPoint;
 FARPROC pFunction;
 PIMAGE\_DATA\_DIRECTORY pExportDataDirectory = ExistDataDirectory(pBaseAddress, IMAGE\_DIRECTORY\_ENTRY\_EXPORT);
 pExportDir = (PIMAGE\_EXPORT\_DIRECTORY)(pBaseAddress + pExportDataDirectory->VirtualAddress);
 if (!pExportDir) 
  return NULL;
 iExportDirSize = pExportDataDirectory->Size;
 pExFunctionsPoint = (\_\_dword \*\*)\_\_RvaToVa\_\_(pBaseAddress, pExportDir->AddressOfFunctions);
 pFunction = (FARPROC)(0 != pExFunctionsPoint\[wOrdinal - pExportDir->Base\]
    ? \_\_RvaToVa\_\_(pBaseAddress, pExFunctionsPoint\[wOrdinal - pExportDir->Base\])
    : NULL);
 if (((\_\_address)pFunction >= (\_\_address)pExportDir) &&
  ((\_\_address)pFunction < (\_\_address)pExportDir + (\_\_address)iExportDirSize))
  pFunction = xLdrFixupForward(pBaseAddress, (\_\_memory)pFunction, wOrdinal, pHashFunc, fpLoadLibraryA);
 return pFunction;
}
FARPROC \_\_INTERNAL\_FUNC\_\_ xLdrGetExportByName(\_\_memory pBaseAddress, \_\_memory pHashPoint, \_\_integer iHashSize, FPHashFunc pHashFunc, FARPROC fpLoadLibraryA) {
 \_\_word wOrdinal = 0;
 \_\_integer iDirCount = 0;
 \_\_address \*pAddrTable = NULL;
 \_\_address addrAddr = 0;
 \_\_offset ofRVA = 0;
 \_\_integer iExpDataSize = 0;
 \_\_integer i = 0;
 PIMAGE\_EXPORT\_DIRECTORY pEd = NULL;
 PIMAGE\_NT\_HEADERS pNt = NULL;
 PIMAGE\_DATA\_DIRECTORY pExportDataDirectory = NULL;
 if (pBaseAddress == NULL) return NULL;
 pNt = GetNtHeader(pBaseAddress);
 iDirCount = pNt->OptionalHeader.NumberOfRvaAndSizes;
 if (iDirCount < IMAGE\_NUMBEROF\_DIRECTORY\_ENTRIES) return FALSE;
 pExportDataDirectory = ExistDataDirectory(pBaseAddress, IMAGE\_DIRECTORY\_ENTRY\_EXPORT);
 if(!pExportDataDirectory) 
  return NULL;//确定引出表
 iExpDataSize = pExportDataDirectory->Size;
 // 从引出表获取函数地址
 pEd = (PIMAGE\_EXPORT\_DIRECTORY)\_\_RvaToVa\_\_(pBaseAddress, pExportDataDirectory->VirtualAddress); 
 /\* 
 \* 获取序数引出的函数
 \*/
 if (HIWORD((\_\_dword)pHashPoint)==0) {//以序号引出
  wOrdinal = (\_\_word)(LOWORD((\_\_dword)pHashPoint)) - pEd->Base;
 } else {
  \_\_integer iCount = 0;
  \_\_dword \*pdwNamePtr = NULL;
  \_\_word \*pwOrdinalPtr = NULL;
  iCount = (\_\_integer)(pEd->NumberOfNames);
  pdwNamePtr = (\_\_dword \*)\_\_RvaToVa\_\_(pBaseAddress, pEd->AddressOfNames);
  pwOrdinalPtr = (\_\_word \*)\_\_RvaToVa\_\_(pBaseAddress, pEd->AddressOfNameOrdinals);
  for(i = 0; i < iCount; i++) {
   \_\_byte HashValue\[1024\];
   \_\_char \*svName = NULL;
   \_\_integer iHashValueSize = 0;
   svName = (\_\_char \*)\_\_RvaToVa\_\_(pBaseAddress, \*pdwNamePtr);
   iHashValueSize = pHashFunc(svName, \_\_logic\_strlen\_\_(svName), HashValue);//进行哈希计算
   if (iHashValueSize == iHashSize) {
    if (\_\_logic\_memcmp\_\_(HashValue, pHashPoint, iHashSize) == 0) {
     wOrdinal = \*pwOrdinalPtr;
     break;
    }
   }
   pdwNamePtr++;
   pwOrdinalPtr++;
  }
  if (i == iCount) return NULL;
 }
 pAddrTable=(\_\_address \*)\_\_RvaToVa\_\_(pBaseAddress, pEd->AddressOfFunctions);
 ofRVA = pAddrTable\[wOrdinal\];
 addrAddr = (\_\_address)\_\_RvaToVa\_\_(pBaseAddress, ofRVA);
 /\*
  \* 最终判断是否是中间跳转
  \*/
 if (((\_\_address)addrAddr >= (\_\_address)pEd) &&
  ((\_\_address)addrAddr < (\_\_address)pEd + (\_\_address)iExpDataSize))
  return xLdrFixupForward(pBaseAddress, (\_\_memory)addrAddr, wOrdinal, pHashFunc, fpLoadLibraryA);
 return (FARPROC)addrAddr;
}
\_\_INLINE\_\_ FARPROC \_\_INTERNAL\_FUNC\_\_ xLdrGetProcedureAddress(\_\_memory pBaseAddress, \_\_memory pHashPoint, \_\_integer iHashSize, FPHashFunc pHashFunc, FARPROC fpLoadLibraryA) {
 FARPROC pProcedureAddress = NULL;
 \_\_dword dwOrdinal = (\_\_dword)pHashPoint;
 if (HIWORD((\_\_dword)pHashPoint)) {
  // 通过名字引出
  pProcedureAddress = xLdrGetExportByName(pBaseAddress, pHashPoint, iHashSize, pHashFunc, fpLoadLibraryA);
 } else {
  // 通过序数
  dwOrdinal &= 0x0000FFFF;
  pProcedureAddress = xLdrGetExportByOrdinal(pBaseAddress, (\_\_word)dwOrdinal, pHashFunc, fpLoadLibraryA);
 }
 return pProcedureAddress;
}

```

  
我逆向的是 win7 下 ntdll 中的位于 0x77F1E681 -> 0x77F1E6F1 -> 0x77F22193 这三个函数就是主要 ntdll 实现对 ApiSetMap 的解析了。  
  
一个邪恶的想法是是否可以在内核里直接劫持这片空间。把他们的映射关系修改我们自己的 DLL。谁有时间研究一下可行性。![](img/c1d8523a7842d7b67917c210ac6dcf18.jpg "微笑")