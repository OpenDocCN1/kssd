
# 修改已加载 DLL 的模块名和路径

> 原文：[`www.pediy.com/kssd/pediy10/69730.html`](https://www.pediy.com/kssd/pediy10/69730.html)

标题：【原创】修改已加载 DLL 的模块名和路径

作者：achillis

时间：2008-07-31,16:50

链接：`http://bbs.pediy.com/showthread.php?t=69730`

简单说就是修改 `PEB` 中的内容，从修改 `PEB` 中的进程路径想到的，纯属小玩意儿，大牛牛飘过～

首先说说从 `PEB` 中获取已加载的模块信息，这个不清楚的可以参考下面这篇文章：

`http://bbs.pediy.com/showthread.php?t=59932`

里面讲得很清楚了，实在好文，我不再多说哈

在获取到每一个模块的信息时，需要用到以下结构：

```c
typedef struct _LDR_MODULE
{
    LIST_ENTRY          InLoadOrderModuleList;
    LIST_ENTRY          InMemoryOrderModuleList;
    LIST_ENTRY          InInitializationOrderModuleList;
    void*               BaseAddress;
    void*               EntryPoint;
    ULONG               SizeOfImage;
    UNICODE_STRING    FullDllName;    //全路径
    UNICODE_STRING      BaseDllName;  //模块名称
    ULONG               Flags;
    SHORT               LoadCount;
    SHORT               TlsIndex;
    HANDLE              SectionHandle;
    ULONG               CheckSum;
    ULONG               TimeDateStamp;
} LDR_MODULE, *PLDR_MODULE;
```

加红的两项就是我们要修改的内容了．用代码说话：

```c
#include <windows.h>
#include <stdio.h>

typedef struct _UNICODE_STRING
{
    USHORT Length;
    USHORT MaximumLength;
    PWSTR Buffer;
} UNICODE_STRING, *PUNICODE_STRING;

typedef struct _PEB_LDR_DATA
{
    ULONG Length;
    BOOLEAN Initialized;
    PVOID SsHandle;
    LIST_ENTRY InLoadOrderModuleList;
    LIST_ENTRY InMemoryOrderModuleList;
    LIST_ENTRY InInitializationOrderModuleList;
} PEB_LDR_DATA,*PPEB_LDR_DATA;

typedef struct _LDR_MODULE
{
    LIST_ENTRY          InLoadOrderModuleList;
    LIST_ENTRY          InMemoryOrderModuleList;
    LIST_ENTRY          InInitializationOrderModuleList;
    void*               BaseAddress;
    void*               EntryPoint;
    ULONG               SizeOfImage;
    UNICODE_STRING    FullDllName;
    UNICODE_STRING      BaseDllName;
    ULONG               Flags;
    SHORT               LoadCount;
    SHORT               TlsIndex;
    HANDLE              SectionHandle;
    ULONG               CheckSum;
    ULONG               TimeDateStamp;
} LDR_MODULE, *PLDR_MODULE;

int main(int argc, char* argv[])
{
    PEB_LDR_DATA *pPEBLDR;
    LDR_MODULE *pLdrMod;
    LIST_ENTRY *pListEntry,*pStart;
    DWORD dwKernelBase;
    PWSTR fakemodulename=L"假名字啊";
    PWSTR fakemodulepath=L"这是假路径，看能骗到谁";

    _asm
    {
        mov eax,fs:[0x30] //TEB->PEB
        mov eax,[eax+0xC] //PEB->Ldr
        mov pPEBLDR,eax
    }

    printf("准备改名字了...\n");
    dwKernelBase=(DWORD)GetModuleHandle("kernel32.dll");
    printf("Base of kernel32.dll is 0x%08x\n",dwKernelBase);
    pStart=pListEntry=(LIST_ENTRY*)(PUCHAR)&(pPEBLDR->InLoadOrderModuleList);

    do
    {
        pListEntry=pListEntry->Flink;
        pLdrMod=(LDR_MODULE*)(pListEntry);
        if ((DWORD)pLdrMod->BaseAddress==dwKernelBase)
        {
            printf("Found kernel32.dll...");
            lstrcpyW(pLdrMod->BaseDllName.Buffer,fakemodulename);
            lstrcpyW(pLdrMod->FullDllName.Buffer,fakemodulepath);
            printf("Change OK.\n");
            break;
        }
    } while(pListEntry!=pStart);

    printf("现在你可以用工具来看看本进程的模块列表\n");
    while (1)
    {
        Sleep(1000);
    }

    return 0;
}
```

不喜欢结构的可以自己用偏移代替～～

附效果图,修改了 `user32.dll` 的，(用 `RKU` 查看的)：

![](img/aafe18ac3f578b53827468a1c3f74442.jpg)

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | `ChangeModulePath.rar` |
| :--- | :--- |
