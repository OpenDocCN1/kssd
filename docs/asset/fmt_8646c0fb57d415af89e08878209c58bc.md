
# 实现 `GetModuleHandle()`

> 原文：[`www.pediy.com/kssd/pediy12/130167-1.html`](https://www.pediy.com/kssd/pediy12/130167-1.html)

```c
// 实现 GetModuleHandle()函数

// DLL 名称对应的地址

// 返回模块的地址

HMODULE SelfGetModuleHandle(LPCSTR lpszDllName)

{

  PPEB pPEB = NULL;

  __asm {

    mov eax, fs:[30h]; // fs:[30h]指向 PEB

    mov pPEB, eax;

  }

  if (pPEB == NULL) return NULL;

  PPEB_LDR_DATA pLdr = pPEB->LoaderData;

  PLIST_ENTRY pFlink = (PLIST_ENTRY)(pLdr->InLoadOrderModuleList.Flink);

  PLIST_ENTRY pBlink = (PLIST_ENTRY)(pLdr->InLoadOrderModuleList.Blink);

  if(NULL == lpszDllName)

  {

    PLDR_DATA_TABLE_ENTRY pModuleTab = (PLDR_DATA_TABLE_ENTRY)pFlink;

    return (HMODULE)pModuleTab->DllBase;

  }

  WCHAR lpwszName[MAX_PATH] = {0};

  int nLen = strlen(lpszDllName) + 1;

  int nwLen = MultiByteToWideChar(CP_ACP, 0, lpszDllName, nLen, NULL, 0);

    MultiByteToWideChar(CP_ACP, 0, lpszDllName, nLen, lpwszName, nwLen);

  for (PLIST_ENTRY pLink = pFlink; pLink!=pBlink; pLink=pLink->Flink)

  {

    PLDR_DATA_TABLE_ENTRY pModuleTab = (PLDR_DATA_TABLE_ENTRY)pLink;

    // 不区分大小写比较

    if (_wcsicmp(pModuleTab->BaseDllName.Buffer, lpwszName) == 0)

      return (HMODULE)pModuleTab->DllBase;

  }

  return NULL;

}
```
