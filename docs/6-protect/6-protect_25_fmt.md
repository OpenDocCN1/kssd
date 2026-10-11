
# 实现函数 `GetProcAddress()`

> 原文：[`www.pediy.com/kssd/pediy12/130167.html`](https://www.pediy.com/kssd/pediy12/130167.html)

通过对 PE 结构的学习，对导入表和导出表相对比较熟悉了。这里自己实现了 ANSI 的 `GetProcAddress()`和 `GetModuleHandle()`函数，UNICODE 的可以仿照来实现。

系统创建进程时根据导入表来加载 DLL 文件。而如果要调用 DLL 的函数，如果导入表中有函数名和地址，则可以直接调用；否则需要通过 `LoadLibrary()`和 `GetProcAddress()`来调用。其中 `LoadLibrary()`完成的功能是：如果 DLL 不在当前进程内存中，则加载并设置引用值；否则直接返回 DLL 的地址并增加引用值。`GetProcAddress()`的功能是：查找映射到内存中的 DLL 的导出表，返回对应函数的地址。由于 DLL 存在索引和函数名称导出两种方式，所以对 DLL 函数的查找也应支持这两种方式，如果按索引查找则按 `SelfGetProcAddress(hModule, (LPCSTR)0x01)`形式进行，如果按名称查找则按 `SelfGetProcAddress(hModule, “MessageBoxA”)`进行。

```
// 实现 GetProcAddress()函数
// hModule 是 LoadLibrary()函数返回的地址
// lpProcName 是函数字符串的地址
// 返回函数对应的地址

DWORD SelfGetProcAddress(HMODULE hModule, LPCSTR lpProcName)
{
  if (hModule==NULL || lpProcName==NULL)
    return NULL;
  BYTE* btBase = (BYTE*)hModule;
  PIMAGE_DOS_HEADER pDosHdr = (PIMAGE_DOS_HEADER)btBase;
  PIMAGE_NT_HEADERS pNtHdrs = (PIMAGE_NT_HEADERS)(btBase+pDosHdr->e_lfanew);
  PIMAGE_FILE_HEADER pFileHdr = (PIMAGE_FILE_HEADER)(&pNtHdrs->FileHeader);
  PIMAGE_OPTIONAL_HEADER pOptHdr = (PIMAGE_OPTIONAL_HEADER)(&pNtHdrs->OptionalHeader);  
  IMAGE_DATA_DIRECTORY DataDir = (IMAGE_DATA_DIRECTORY)(pOptHdr->DataDirectory[0]);
  if (!DataDir.VirtualAddress || !DataDir.Size)
    return NULL;
  PIMAGE_EXPORT_DIRECTORY pExpDir = (PIMAGE_EXPORT_DIRECTORY)(btBase+DataDir.VirtualAddress);
  if (!pExpDir || !pExpDir->NumberOfNames || !pExpDir->NumberOfFunctions)
    return NULL;
  PDWORD pNamesAddr = (PDWORD)(btBase + pExpDir->AddressOfNames);
  PWORD  pOrdisAddr = (PWORD)(btBase + pExpDir->AddressOfNameOrdinals);
  PDWORD pFuncsAddr = (PDWORD)(btBase + pExpDir->AddressOfFunctions);
  if((DWORD)lpProcName <= 0x0000FFFF) // 按索引查找函数地址
  {
    DWORD nIndex = (DWORD)lpProcName - pExpDir->Base;
    if (nIndex < pExpDir->NumberOfFunctions)
      return (DWORD)btBase+pFuncsAddr[nIndex];
  }
  else // 按函数名查找函数地址
  {
    for(unsigned i = 0; i<pExpDir->NumberOfNames; i++)
    {
      PCSTR pszName = (PCSTR)(btBase+pNamesAddr[i]);
      WORD nIndex = pOrdisAddr[i];
      if(strcmp(lpProcName, pszName) == 0)
        return (DWORD)btBase+pFuncsAddr[nIndex];
    }
  }
   return   NULL;
}
```

PEB 是进程环境块，存放进程信息。TEB 是线程环境块，存放线程信息。FS 段寄存器指向当前的 TEB 结构，在 TEB 偏移 `0x30`处是 PEB 指针，通过这个指针即可取得 PEB 的地址。不过可能你的编程环境中没有 PEB 和 TEB 的定义，网络上能搜到相关的定义。
