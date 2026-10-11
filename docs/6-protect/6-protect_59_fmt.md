
# 完美实现 GetProcAddress

> 原文：[`www.pediy.com/kssd/pediy11/121226.html`](https://www.pediy.com/kssd/pediy11/121226.html)

我们知道 `kernel32.dll` 里有一个 `GetProcAddress` 函数，可以找到模块中的函数地址，函数原型是这样的：

`WINBASEAPI` `FARPROC` `WINAPI` `GetProcAddress(`
`IN HMODULE hModule,`
`IN LPCSTR lpProcName`
`)`

`hModule` 是模块的句柄，说白了就是内存中 `dll` 模块的首地址

`lpProcName` 一般指函数名称的字符串地址，也可能是指序号，如何区分呢？

我们这样

```
if (((DWORD)lpProcName & 0xFFFF0000) == 0)
{
  //这里是序号导出的;
}
else
{
  //这里是函数名称导出的
}
```

最终真找到匹配的函数地址，然后返回就 ok 了，但是前提是你需要了解 PE 的导出表

下面简单说一下,首先从 PE 里找到下面这个结构，如果不知道如何找的话，建议在坛子里搜索一下 PE 结构解析的文章，找到这个结构应该还是很简单的

`typedef struct _IMAGE_EXPORT_DIRECTORY {`
`DWORD Characteristics;`
`DWORD TimeDateStamp;`
`WORD MajorVersion;`
`WORD MinorVersion;`
`DWORD Name;`
`DWORD Base;`
`DWORD NumberOfFunctions;`
`DWORD NumberOfNames;`
`DWORD AddressOfFunctions;     // RVA from base of image`
`DWORD AddressOfNames;         // RVA from base of image`
`DWORD AddressOfNameOrdinals;  // RVA from base of image`
`} IMAGE_EXPORT_DIRECTORY, *PIMAGE_EXPORT_DIRECTORY;`

具体什么意思呢？

虽然， `kanxue` 老大在这里写的很漂亮了都

[`www.pediy.com/tutorial/chap8/Chap8-1-7.htm`](http://www.pediy.com/tutorial/chap8/Chap8-1-7.htm)

我还是打算嗦一下

`Base` 函数以序号导出的时候的序号基数，从这个数开始递增

`NumberOfFunctions` 本 `dll` 一共有多少个导出函数，不管是以序号还是以函数名导出

`NumberOfNames` 本 `dll` 中以能够以函数名称导出的函数个数（注意，说一下，其实函数里的每一个函数都能通过序号导出，但是为了兼容性等等，也给一些函数提供用函数名称来导出）

`AddressOfFunctions` 指向一个 `DWORD` 数组首地址，共有 `NumberOfFunctions` 个元素，每一个元素都是一个函数地址

`AddressOfNames` 指向一个 `DWORD` 数组首地址，共有 `NumberOfNames` 个元素，每一个元素都是一个字符串（函数名字符串）首地址

`AddressOfNameOrdinals` 指向一个 `WORD` 数组首地址，共有 `NumberOfNames` 个元素，每一个元素都是一个函数序号

我们说的最后俩数组，其实是一种一一对应的关系，假如分别叫 `dwNames[]` 和 `dwNamesOrdinals[]`，

假如 `dwNames[5]`的值（这个指是一个地址，前面都说了）指向的字符串等于“`GetValue`”,那么 `dwNamesOrdinals[5]`的值（这个指是一个序号，前面都说了），就是 `GetValue` 导出函数的序号啦，那么怎样找到地址呢？

这时候就需要用到第一个数组了，假如名字叫 `dwFuncAddress[]`, `GetValue` 的导出地址就是

`dwFuncAddress[dwNamesOrdinals[5]] + 模块基址`

好了，嗦了这么多，看代码：

```
DWORD MyGetProcAddress(
    HMODULE hModule,    // handle to DLL module
    LPCSTR lpProcName   // function name
    )
{
    
    int i=0;
    PIMAGE_DOS_HEADER pImageDosHeader = NULL;
    PIMAGE_NT_HEADERS pImageNtHeader = NULL;
    PIMAGE_EXPORT_DIRECTORY pImageExportDirectory = NULL;
    
    pImageDosHeader=(PIMAGE_DOS_HEADER)hModule;
    pImageNtHeader=(PIMAGE_NT_HEADERS)((DWORD)hModule+pImageDosHeader->e_lfanew);
    pImageExportDirectory=(PIMAGE_EXPORT_DIRECTORY)((DWORD)hModule+pImageNtHeader->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_EXPORT].VirtualAddress);
    
    DWORD *pAddressOfFunction = (DWORD*)(pImageExportDirectory->AddressOfFunctions + (DWORD)hModule);
    DWORD *pAddressOfNames = (DWORD*)(pImageExportDirectory->AddressOfNames + (DWORD)hModule);
    DWORD dwNumberOfNames = (DWORD)(pImageExportDirectory->NumberOfNames);
    DWORD dwBase = (DWORD)(pImageExportDirectory->Base);
    
    WORD *pAddressOfNameOrdinals = (WORD*)(pImageExportDirectory->AddressOfNameOrdinals + (DWORD)hModule);
    
    //这个是查一下是按照什么方式（函数名称 or 函数序号）来查函数地址的 
    DWORD dwName = (DWORD)lpProcName;
    if ((dwName & 0xFFFF0000) == 0)
    {
      goto xuhao;
    }
    for (i=0; i<(int)dwNumberOfNames; i++)
    {
      char *strFunction = (char *)(pAddressOfNames[i] + (DWORD)hModule);
      if (lstrcmp(lpProcName, strFunction) == 0)
      {
        return (pAddressOfFunction[pAddressOfNameOrdinals[i]] + (DWORD)hModule);
      }
    }
    return 0;
    //这个是通过以序号的方式来查函数地址的
xuhao:
    if (dwName < dwBase || dwName > dwBase + pImageExportDirectory->NumberOfFunctions - 1)
    {
      return 0;
    }
    return (pAddressOfFunction[dwName - dwBase] + (DWORD)hModule);
}
```

好了，测试一下，

```
//我们写的函数返回的地址
DWORD dw1 = MyGetProcAddress(LoadLibrary("user32.dll"), "MessageBoxA");
```

```
//系统函数返回的地址
DWORD dw2 = (DWORD)GetProcAddress(LoadLibrary("user32.dll"), "MessageBoxA");
```

哈，发现一样，突然非常有成就感！！

再试试序号查找

```
//我们写的函数返回的地址
DWORD dw1 = MyGetProcAddress(LoadLibrary("user32.dll"), (LPCSTR)0x110);
```

```
//系统函数返回的地址
DWORD dw2 = (DWORD)GetProcAddress(LoadLibrary("user32.dll"), (LPCSTR)0x110);
```

我们发现还是一样，成就感更大啦。。哈哈（其实 `kernel32.dll` 的 `0x110` 是 `GetComputerNameExW` 的序号，自己可以用 `LordPE` 查一下）

突然有一天有人说 你的这个函数不行，然后给你举了个例子，于是你测试了一下，下面是例子

```
DWORD a1 = (DWORD)MyGetProcAddress(LoadLibrary("kernel32.dll"), "HeapFree");
DWORD a2 = (DWORD)GetProcAddress(LoadLibrary("kernel32.dll"), "HeapFree");
```

于是 我们就苦思冥想，依然不得其解。。。

但是我发现 `a1` 表示的地址的内容是一个字符串 `"NTDLL.RtlFreeHeap"` 似乎不能用巧合来说这个问题，难道是返回了这个字符串 还要我们再 `Load` 一下 `Ntdll` 然后再找一个 `RtlFreeHeap` 的地址吗？好了先试验一下 果然 `ntdll.dll` 中的 `RtlFreeHeap` 的地址 和 `a2` 的值的一样的，

似乎印证了什么东西，

好吧 `OD` 拿来 开启逆向 `kernel32.GetProcAddress` 搞了一会头晕了，看不出头绪

我老大很有才，他去翻了翻 `win2000` 的源码 偶也 一目了然

Win2K 源码




# FARPROC: `GetProcAddress`

## 函数原型

```c
FARPROC
GetProcAddress(
    HMODULE hModule,
    LPCSTR lpProcName
    )
```

## 例程描述

本函数检索由 `lpProcName` 参数指向名称的函数的内存地址。`GetProcAddress` 函数在由 `hModule` 参数指定的模块中查找该函数，或者如果 `hModule` 为 `NULL`，则在当前进程关联的模块中查找。该函数必须是一个导出函数；模块的定义文件必须包含用于该函数的适当 `EXPORTS` 行。

如果 `lpProcName` 参数是一个序号值，且指定序号的函数在模块中不存在，`GetProcAddress` 仍可能返回非 `NULL` 值。在函数可能不存在的情况下，指定函数时应使用名称而不是序号值。

仅使用 `GetProcAddress` 检索属于库模块的导出函数的地址。

函数名的拼写（由 `lpProcName` 指向）必须与源库定义（`.DEF`）文件中出现的拼写完全相同。函数可以在定义文件中重命名。使用大小写敏感的匹配。

## 参数

- `hModule`：标识包含该函数的可执行文件的模块。值为 `NULL` 引用与用于创建当前进程的映像文件相关的模块句柄。
- `lpProcName`：指向函数名，或包含函数的序号值。如果是序号值，该值必须位于低序字，高序字必须为零。字符串必须是空终止的字符串。

## 返回值

如果函数成功，返回值指向函数的入口点。返回值为 `NULL` 表示错误，并使用 `GetLastError` 函数获取扩展错误状态。

```c
{
    NTSTATUS Status;
    PVOID ProcedureAddress;
    STRING ProcedureName;

    //+ by blueapplez
    //这应该是按函数名称查找
    //+ by blueapplez
    if ( (ULONG_PTR)lpProcName > 0xffff ) {
        RtlInitString(&ProcedureName,lpProcName);
        Status = LdrGetProcedureAddress(
                    BasepMapModuleHandle( hModule, FALSE ),
                    &ProcedureName,
                    0L,
                    &ProcedureAddress
                    );
        }
    //+ by blueapplez
    //这应该是按函数序号查找
    //+ by blueapplez
    else {
        Status = LdrGetProcedureAddress(
                    BasepMapModuleHandle( hModule, FALSE ),
                    NULL,
                    PtrToUlong((PVOID)lpProcName),
                    &ProcedureAddress
                    );
        }
    if ( !NT_SUCCESS(Status) ) {
        BaseSetLastNTError(Status);
        return NULL;
        }
    else {
        if ( ProcedureAddress == BasepMapModuleHandle( hModule, FALSE ) ) {
            if ( (ULONG_PTR)lpProcName > 0xffff ) {
                Status = STATUS_ENTRYPOINT_NOT_FOUND;
                }
            else {
                Status = STATUS_ORDINAL_NOT_FOUND;
                }
            BaseSetLastNTError(Status);
            return NULL;
            }
        else {
            return (FARPROC)ProcedureAddress;
            }
        }
}
```

# `LdrGetProcedureAddress` 分析

`LdrGetProcedureAddress` 是什么呢？

继续看。

## 代码

```c
NTSTATUS
LdrGetProcedureAddress (
    IN PVOID DllHandle,
    IN PANSI_STRING ProcedureName OPTIONAL,
    IN ULONG ProcedureNumber OPTIONAL,
    OUT PVOID *ProcedureAddress
    )
{

    return LdrpGetProcedureAddress(DllHandle,ProcedureName,ProcedureNumber,ProcedureAddress,TRUE);

}
```

`LdrpGetProcedureAddress` 就太长了，先主要说一下他里面的一个关键函数 `LdrpSnapThunk` 的关键的一段，能不能看懂我就不管了，反正我基本看不懂。

## 代码

```c
  else 
  {
           Addr = (PULONG)((ULONG_PTR)DllBase + (ULONG)ExportDirectory->AddressOfFunctions);
           Thunk->u1.Function = ((ULONG_PTR)DllBase + Addr[OrdinalNumber]);
    //+ by blueapplez
    //这段是判断一下返回的地址是否越界
    //越界了 就返回原来的值
    //+ by blueapplez
           if (Thunk->u1.Function > (ULONG_PTR)ExportDirectory &&
                Thunk->u1.Function < ((ULONG_PTR)ExportDirectory + ExportSize)
              ) 
       {
            UNICODE_STRING UnicodeString;
            ANSI_STRING ForwardDllName;
            PVOID ForwardDllHandle;
            PUNICODE_STRING ForwardProcName;
            ULONG ForwardProcOrdinal;

    //+ by blueapplez
    //如果没有越界，就从那个地址中查找'.'，找到了就再做一次 Load Dll，和 GetProcAddress
    //没有找到'.' 就返回原来的值
    //+ by blueapplez
            ImportString = (PSZ)Thunk->u1.Function;
            ForwardDllName.Buffer = ImportString,
            ForwardDllName.Length = (USHORT)(strchr(ImportString, '.') - ImportString);
            ForwardDllName.MaximumLength = ForwardDllName.Length;
            st = RtlAnsiStringToUnicodeString(&UnicodeString, &ForwardDllName, TRUE);

            if (NT_SUCCESS(st)) {
#if defined (WX86)
                if (Wx86ProcessInit) {
                    NtCurrentTeb()->Wx86Thread.UseKnownWx86Dll = RtlImageNtHeader(DllBase)->FileHeader.Machine
                                                                 == IMAGE_FILE_MACHINE_I386;

                }
#endif
```

## 补充源码与测试

好了，到此已经告一段落，下面给出源码。有人会问，怎样做的 dll 才会出现查找导出表的地址的时候返回一个字符串呢？其实就是在 `.def` 文件的 `EXPORTS` 后加一句就成（这个我老大试验了 n 久才搞定）加上 `MsgBox = user32.MessageBoxA`。

## 自定义 `MyGetProcAddress` 实现

```c
DWORD MyGetProcAddress(
           HMODULE hModule,    // handle to DLL module
           LPCSTR lpProcName   // function name
           )
{
  int i=0;
  char *pRet = NULL;
  PIMAGE_DOS_HEADER pImageDosHeader = NULL;
  PIMAGE_NT_HEADERS pImageNtHeader = NULL;
  PIMAGE_EXPORT_DIRECTORY pImageExportDirectory = NULL;
  
  pImageDosHeader=(PIMAGE_DOS_HEADER)hModule;
  pImageNtHeader=(PIMAGE_NT_HEADERS)((DWORD)hModule+pImageDosHeader->e_lfanew);
  pImageExportDirectory=(PIMAGE_EXPORT_DIRECTORY)((DWORD)hModule+pImageNtHeader->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_EXPORT].VirtualAddress);
  
  DWORD dwExportRVA = pImageNtHeader->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_EXPORT].VirtualAddress;
  DWORD dwExportSize = pImageNtHeader->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_EXPORT].Size;
  
  DWORD *pAddressOfFunction = (DWORD*)(pImageExportDirectory->AddressOfFunctions + (DWORD)hModule);
  DWORD *pAddressOfNames = (DWORD*)(pImageExportDirectory->AddressOfNames + (DWORD)hModule);
  DWORD dwNumberOfNames = (DWORD)(pImageExportDirectory->NumberOfNames);
  DWORD dwBase = (DWORD)(pImageExportDirectory->Base);
  
  WORD *pAddressOfNameOrdinals = (WORD*)(pImageExportDirectory->AddressOfNameOrdinals + (DWORD)hModule);
  
  //这个是查一下是按照什么方式（函数名称 or 函数序号）来查函数地址的
  DWORD dwName = (DWORD)lpProcName;
  if ((dwName & 0xFFFF0000) == 0)
  {
    goto xuhao;
  }
  
  for (i=0; i<(int)dwNumberOfNames; i++)
  {
    char *strFunction = (char *)(pAddressOfNames[i] + (DWORD)hModule);
    if (strcmp(strFunction, (char *)lpProcName) == 0)
    {
      pRet = (char *)(pAddressOfFunction[pAddressOfNameOrdinals[i]] + (DWORD)hModule);
      goto _exit11;
    }
  }
  //这个是通过以序号的方式来查函数地址的
xuhao:
  if (dwName < dwBase || dwName > dwBase + pImageExportDirectory->NumberOfFunctions - 1)
  {
    return 0;
  }
  pRet = (char *)(pAddressOfFunction[dwName - dwBase] + (DWORD)hModule);
_exit11:
       //判断得到的地址有没有越界
   if ((DWORD)pRet<dwExportRVA+(DWORD)hModule || (DWORD)pRet > dwExportRVA+ (DWORD)hModule + dwExportSize)
   {
     return (DWORD)pRet;
   }
  char pTempDll[100] = {0};
  char pTempFuction[100] = {0};
  lstrcpy(pTempDll, pRet);
  char *p = strchr(pTempDll, '.');
  if (!p)
  {
    return (DWORD)pRet;
  }
  *p = 0;
  lstrcpy(pTempFuction, p+1);
  lstrcat(pTempDll, ".dll");
  HMODULE h = LoadLibrary(pTempDll);
  if (h == NULL)
  {
    return (DWORD)pRet;
  }
  return MyGetProcAddress(h, pTempFuction);
}
```

现在测试下



```
DWORD a1 = (DWORD)MyGetProcAddress(LoadLibrary("kernel32.dll"), (LPCSTR)"HeapFree");
DWORD a2 = (DWORD)GetProcAddress(LoadLibrary("kernel32.dll"), (LPCSTR)"HeapFree");
```

发现值一样了

ps.如发现问题 欢迎批评指正。
