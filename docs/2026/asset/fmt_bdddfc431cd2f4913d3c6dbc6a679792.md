
### PE 重定位处理

```
PIMAGE_DOS_HEADER pDos = (PIMAGE_DOS_HEADER)chBaseAddress;

PIMAGE_NT_HEADERS pNt = (PIMAGE_NT_HEADERS)(chBaseAddress + pDos->e_lfanew);

PIMAGE_BASE_RELOCATION pLoc = (PIMAGE_BASE_RELOCATION)(chBaseAddress + pNt->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_BASERELOC].VirtualAddress);
```
```c
//判断是否有重定位表

if ((char*)pLoc == (char*)pDos)
{
    return TRUE;
}
```
```c
while ((pLoc->VirtualAddress + pLoc->SizeOfBlock) != 0) //开始扫描重定位表
{
    WORD *pLocData = (WORD *)((PBYTE)pLoc + sizeof(IMAGE_BASE_RELOCATION));
    //计算需要修正的重定位项（地址）的数目
    int nNumberOfReloc = (pLoc->SizeOfBlock - sizeof(IMAGE_BASE_RELOCATION)) / sizeof(WORD);

    for (int i = 0; i < nNumberOfReloc; i++)
    {
        // 每个WORD由两部分组成。高4位指出了重定位的类型，WINNT.H中的一系列IMAGE_REL_BASED_xxx定义了重定位类型的取值。
        // 低12位是相对于VirtualAddress域的偏移，指出了必须进行重定位的位置。

        if ((DWORD)(pLocData[i] & 0x0000F000) == 0x00003000) //这是一个需要修正的地址
        {
            DWORD* pAddress = (DWORD *)((PBYTE)pDos + pLoc->VirtualAddress +                                                                    (pLocData[i] & 0x0FFF));    
            DWORD dwDelta = (DWORD)pDos - pNt->OptionalHeader.ImageBase;
            *pAddress += dwDelta;
        }
    }

    //转移到下一个节进行处理
    pLoc = (PIMAGE_BASE_RELOCATION)((PBYTE)pLoc + pLoc->SizeOfBlock);
}
```
```c
return TRUE;
```

### 6.根据PE结构的导入表，加载所需的dll，并获取导入函数的地址并写入导入表中

```
PIMAGE_DOS_HEADER pDos = (PIMAGE_DOS_HEADER)chBaseAddress;

PIMAGE_NT_HEADERS pNt = (PIMAGE_NT_HEADERS)(chBaseAddress + pDos->e_lfanew);

PIMAGE_IMPORT_DESCRIPTOR pImportTable = (PIMAGE_IMPORT_DESCRIPTOR)((DWORD)pDos +
    pNt->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_IMPORT].VirtualAddress);
```
```c
// 循环遍历DLL导入表中的DLL及获取导入表中的函数地址
char *lpDllName = NULL;
HMODULE hDll = NULL;
PIMAGE_THUNK_DATA lpImportNameArray = NULL;
PIMAGE_IMPORT_BY_NAME lpImportByName = NULL;
PIMAGE_THUNK_DATA lpImportFuncAddrArray = NULL;
FARPROC lpFuncAddress = NULL;
DWORD i = 0;

while (TRUE)
{
    if (0 == pImportTable->OriginalFirstThunk)
    {
        break;
    }

    // 获取导入表中DLL的名称并加载DLL
    lpDllName = (char *)((DWORD)pDos + pImportTable->Name);
    hDll = GetModuleHandleA(lpDllName);
    if (NULL == hDll)
    {
        hDll = LoadLibraryA(lpDllName);
        if (NULL == hDll)
        {
            pImportTable++;
            continue;
        }
    }

    i = 0;
    // 获取OriginalFirstThunk以及对应的导入函数名称表首地址
    lpImportNameArray = (PIMAGE_THUNK_DATA)((DWORD)pDos + pImportTable->OriginalFirstThunk);
    // 获取FirstThunk以及对应的导入函数地址表首地址
    lpImportFuncAddrArray = (PIMAGE_THUNK_DATA)((DWORD)pDos + pImportTable->FirstThunk);
    while (TRUE)
    {
        if (0 == lpImportNameArray[i].u1.AddressOfData)
        {
            break;
        }

        // 获取IMAGE_IMPORT_BY_NAME结构
        lpImportByName = (PIMAGE_IMPORT_BY_NAME)((DWORD)pDos + lpImportNameArray[i].u1.AddressOfData);

        // 判断导出函数是序号导出还是函数名称导出
        if (0x80000000 & lpImportNameArray[i].u1.Ordinal)
        {
            // 序号导出
            lpFuncAddress = GetProcAddress(hDll, (LPCSTR)                           (lpImportNameArray[i].u1.Ordinal & 0x0000FFFF));
        }
        else
        {
            // 名称导出
            lpFuncAddress = GetProcAddress(hDll, (LPCSTR)lpImportByName->Name);
        }
        lpImportFuncAddrArray[i].u1.Function = (DWORD)lpFuncAddress;
        i++;
    }

    pImportTable++;
}
```

### 7.修改PE文件的加载基址

```
PIMAGE_DOS_HEADER pDos = (PIMAGE_DOS_HEADER)chBaseAddress;

PIMAGE_NT_HEADERS pNt = (PIMAGE_NT_HEADERS)(chBaseAddress + pDos->e_lfanew);

pNt->OptionalHeader.ImageBase = (ULONG32)chBaseAddress;
```

### 8.跳转到PE的入口点处执行

```
PIMAGE_DOS_HEADER pDos = (PIMAGE_DOS_HEADER)chBaseAddress;

PIMAGE_NT_HEADERS pNt = (PIMAGE_NT_HEADERS)(chBaseAddress + pDos->e_lfanew);

char* ExeEntry = (char*)(chBaseAddress + pNt->OptionalHeader.AddressOfEntryPoint);

// 跳转到入口点处执行

__asm
{
    mov eax, ExeEntry
    jmp eax
}
```

通过上述代码，成功加载并运行程序：

![](https://bbs.kanxue.com/upload/attach/201901/829513_UUANC634TNN5JR8.png)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2019-2-27 19:25

被Iam0x17编辑，原因：

上传的附件：

- [main.rar](javascript:void(0);)

  （2.48kb，405次下载）
