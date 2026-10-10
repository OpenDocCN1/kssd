

# 实现函数 GetModuleHandle()

> 原文：[`www.pediy.com/kssd/pediy12/130167-1.html`](https://www.pediy.com/kssd/pediy12/130167-1.html)

// 实现 GetModuleHandle()函数  
// DLL 名称对应的地址  
// 返回模块的地址  
HMODULE SelfGetModuleHandle(LPCSTR lpszDllName)  
{  
  PPEB pPEB = NULL;  
  \_\_asm {   
    mov eax, fs:\[30h\]; // fs:\[30h\]指向 PEB  
    mov pPEB, eax;  
  }  
  if (pPEB == NULL) return NULL;  
  
  PPEB\_LDR\_DATA pLdr = pPEB->LoaderData;  
  PLIST\_ENTRY pFlink = (PLIST\_ENTRY)(pLdr->InLoadOrderModuleList.Flink);  
  PLIST\_ENTRY pBlink = (PLIST\_ENTRY)(pLdr->InLoadOrderModuleList.Blink);  
  if(NULL == lpszDllName)  
  {  
    PLDR\_DATA\_TABLE\_ENTRY pModuleTab = (PLDR\_DATA\_TABLE\_ENTRY)pFlink;   
    return (HMODULE)pModuleTab->DllBase;  
  }  
  
  WCHAR lpwszName\[MAX\_PATH\] = {0};  
  int nLen = strlen(lpszDllName) + 1;   
  int nwLen = MultiByteToWideChar(CP\_ACP, 0, lpszDllName, nLen, NULL, 0);  
    MultiByteToWideChar(CP\_ACP, 0, lpszDllName, nLen, lpwszName, nwLen);   
  
  for (PLIST\_ENTRY pLink = pFlink; pLink!=pBlink; pLink=pLink->Flink)  
  {  
    PLDR\_DATA\_TABLE\_ENTRY pModuleTab = (PLDR\_DATA\_TABLE\_ENTRY)pLink;  
  
    // 不区分大小写比较  
    if (\_wcsicmp(pModuleTab->BaseDllName.Buffer, lpwszName) == 0)   
      return (HMODULE)pModuleTab->DllBase;   
  }  
  
  return NULL;  
}