
# 用 symbol 来获得 ShadowSSDT 的原始地址和函数名

> 原文：[`www.pediy.com/kssd/pediy10/94676.html`](https://www.pediy.com/kssd/pediy10/94676.html)

在网上看了下，获得 ShadowSSDT 的函数名和原始地址的方法和文章不是很多。比较简单的应该算是设计张函数名表和用 symbol 的方法。

个人觉得用 symbol 来获得 ShadowSSDT 还是比较方便的，而且自己也不用去创建一张表，何乐而不为。^\_^

这办法简单的原因是我只需要一个 `win32.sys`，`win32.pdb`，以及调用不到 10 个的 API 就能完成工作。

好，来看看怎么调用这些函数。

1. 调用 `SymInitialize` 初始化 `Dbghelp` 这系列的函数。
2. 调用 `ImageLoad` 读取文件 `win32.sys`。
3. 调用 `SymLoadModule` 来读取 pdb 文件。
4. 调用 `SymEnumSymbols` 枚举符号，其中一个参数是回调函数 `SymEnumSymbolsProc`，that's the key。
5. 调用 `ImageUnload` 和 `SymCleanup` 做一些清理工作。

`SymEnumSymbolsProc` 中有一个系统会帮忙填充一个为 `PSYMBOL_INFO` 结构的参数。在这个结构中我主要用到的是 `Name` 和 `Address`。

OK，看一下我的思路。

首先是获得 `W32pServiceTable` 的地址。熟悉 ShadowSSDT 都知道，`W32pServiceTable` 是在内核初始化用来填充 ShadowSSDT 的表。

然后，知道了 `W32pServiceTable` 的地址，后面的事情也很容易。用 `SymEnumSymbolsProc` 把 ShadowSSDT 每个函数的地址和函数名保存下来。

说起来有点复杂，但代码很简单的。

代码:

```c
#include <windows.h>
#include <stdio.h>
#include <Dbghelp.h>

#pragma comment(lib, "Dbghelp.lib")
#pragma comment(lib, "Imagehlp.lib")

extern "C" {

  PLOADED_IMAGE
    IMAGEAPI
    ImageLoad(
    PCSTR DllName,
    PCSTR DllPath
    );

  BOOL 
    IMAGEAPI 
    ImageUnload(
    PLOADED_IMAGE LoadedImage   
    );

}

//
//用于存放得到的 ShadowSSDT 的函数和函数名
//
typedef struct _SHADOWFUNC
{
  CHAR FuncName[100];
  DWORD FuncAddr;

}SHADOWFUNC, *PSHADOWFUNC;

DWORD    g_W32pServiceTable = 0;
SHADOWFUNC  g_ShadowFunc[667] = {0};

//
//回调函数，用于获得 ShadowSSDT 的函数和函数名
//
BOOL CALLBACK SymEnumSymbolsProc(
                     PSYMBOL_INFO  pSymInfo,
                     ULONG  SymbolSize,
                     PVOID  UserContext
                     )
{
  PDWORD  pW32pServiceTable = NULL;
  INT    i = 0;
  if (!UserContext)
  {
    if (strstr(pSymInfo->Name, "W32pServiceTable"))
    {
      printf("%s, %#x\n", pSymInfo->Name, pSymInfo->Address);
      g_W32pServiceTable = (DWORD)pSymInfo->Address;
    }
  }
  else
  {
    pW32pServiceTable = (PDWORD)UserContext;
    for (i = 0; i < 667; i++)
    {
      if (*(pW32pServiceTable + i) == (DWORD)pSymInfo->Address)
      {
        g_ShadowFunc[i].FuncAddr = (DWORD)pSymInfo->Address;
        lstrcpyn(g_ShadowFunc[i].FuncName, pSymInfo->Name, 100);          
      }
    }
  }

  return TRUE;
}

void main()
{
  INT        i = 0;
  PDWORD      pW32pServiceTable = 0;
  HANDLE      hHandle = 0;
  PLOADED_IMAGE  ploadImage = {0};
  DWORD      dwload = 0;

  hHandle = GetCurrentProcess();
  SymInitialize(hHandle, NULL, TRUE);

  ploadImage = ImageLoad("win32k.sys", NULL);

  dwload = SymLoadModule (hHandle, ploadImage->hFile, 
    "win32k.sys", "win32k.pdb", 
    0, ploadImage->SizeOfImage);


  SymEnumSymbols(hHandle, dwload, NULL, SymEnumSymbolsProc, NULL);
  if (g_W32pServiceTable)
  {
    pW32pServiceTable = (PDWORD)(g_W32pServiceTable - dwload 
      + (DWORD)ploadImage->MappedAddress);
    SymEnumSymbols(hHandle, dwload, NULL, SymEnumSymbolsProc, pW32pServiceTable);
  }

  for (i = 0; i < 667; i++)
  {
    printf("%03d, 0x%08X, %s\n", i, g_ShadowFunc[i].FuncAddr, g_ShadowFunc[i].FuncName);
  }

  ImageUnload(ploadImage);
  SymCleanup(hHandle);
}
```

先放张图片吧：

![](img/9c47d86c44bf22b196c2a3a96948c404.jpg)

嗯，很成功~~~hoho~~~![](img/022c916b9660e467a8663bd57a4ae6d1.jpg "狂笑")

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || ShadowSSDT.rar  

 |
