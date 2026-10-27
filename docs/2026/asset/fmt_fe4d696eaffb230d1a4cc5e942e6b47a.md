
# [原创]X64 枚举 内核 符号~~~~

```c
typedef NTSTATUS (*ZWQUERYSYSTEMINFORMATION)(
        IN ULONG        SystemInformationClass,
        OUT PVOID        SystemInformation,
        IN ULONG        Length,
        OUT PULONG        ReturnLength
);
typedef unsigned long DWORD;
typedef struct _SYSTEM_MODULE_INFORMATION_ENTRY
{
    ULONG Unknow1;
    ULONG Unknow2;
    ULONG Unknow3;
        ULONG Unknow4;
    PVOID Base;
    ULONG Size;
    ULONG Flags;
    USHORT Index;
    USHORT NameLength;
    USHORT LoadCount;
    USHORT ModuleNameOffset;
    char ImageName[256];
} SYSTEM_MODULE_INFORMATION_ENTRY, *PSYSTEM_MODULE_INFORMATION_ENTRY;
typedef struct _SYSTEM_MODULE_INFORMATION
{
    ULONG Count;//内核中以加载的模块的个数
    SYSTEM_MODULE_INFORMATION_ENTRY Module[1];
} SYSTEM_MODULE_INFORMATION, *PSYSTEM_MODULE_INFORMATION;
```

X64 枚举 内核 模块 需要的 结构体

```c
ULONG64 EnumKM(char *HighlightDrvName) 枚举 内核模块 返回 基地址
{
    ULONG NeedSize, i, ModuleCount, HLed=0, BufferSize = 0x5000;
    PVOID pBuffer = NULL;
        PCHAR pDrvName = NULL;
    NTSTATUS Result;
        ULONG64 address;
    PSYSTEM_MODULE_INFORMATION pSystemModuleInformation;
    do
    {
        //分配内存
        pBuffer = malloc( BufferSize );
        if( pBuffer == NULL )
            return 0;
        //查询模块信息
        Result = ZwQuerySystemInformation( 11, pBuffer, BufferSize, &NeedSize );
        if( Result == 0xC0000004L )
        {
            free( pBuffer );
            BufferSize *= 2;
        }
        else if( Result<0 )
        {
            //查询失败则退出
            free( pBuffer );
            return 0;
        }
    }
    while( Result == 0xC0000004L );
    pSystemModuleInformation = (PSYSTEM_MODULE_INFORMATION)pBuffer;
        //获得模块的总数量
    ModuleCount = pSystemModuleInformation->Count;
        //遍历所有的模块
    for( i = 0; i < ModuleCount; i++ )
        {
                if((ULONG64)(pSystemModuleInformation->Module[i].Base) > (ULONG64)0x8000000000000000)
                {
                        pDrvName = pSystemModuleInformation->Module[i].ImageName+pSystemModuleInformation->Module[i].ModuleNameOffset;
                        if( _stricmp(pDrvName,HighlightDrvName)==0 )
                        {
                                address = (ULONG64)pSystemModuleInformation->Module[i].Base;
                                HLed=1;
                                break;
                        }
                }
        }
        if (HLed == 0)
                return 0;
        free(pBuffer);
        return address;
}
```

```c
BOOL CALLBACK EnumSymCallBack(PSYMBOL_INFO pSymInfo, ULONG SymbolSize, PVOID UserContext)回调 函数
{
        if (strcmp((pSymInfo->Name), "PspCreateProcessNotifyRoutine") == 0)
        {
                printf("Oh,yeah! %s :%p\n", pSymInfo->Name, pSymInfo->Address);
        }
        if (strcmp((pSymInfo->Name), "PspLoadImageNotifyRoutine") == 0)
        {
                printf("Oh,yeah! %s :%p\n", pSymInfo->Name, pSymInfo->Address);
        }
        if (strcmp((pSymInfo->Name), "PspCreateThreadNotifyRoutine") == 0)
        {
                printf("Oh,yeah! %s :%p\n", pSymInfo->Name, pSymInfo->Address);
        }
        if (strcmp((pSymInfo->Name), "PspCidTable") == 0)
        {
                printf("Oh,yeah! %s :%p\n", pSymInfo->Name, pSymInfo->Address);
        }
        if (strcmp((pSymInfo->Name), "ExDestroyHandle") == 0)
        {
                printf("Oh,yeah! %s :%p\n", pSymInfo->Name, pSymInfo->Address);
        }
        return TRUE;
}
```

```c
void getallkrnladdress(ULONG64 ntkrnlmpBaseaddress){ 加载 符号链接 并枚举
        HANDLE hProcess;
        DWORD64 BaseOfDll;
        PIMAGEHLP_SYMBOL pSymbol = NULL;
        DWORD Options = SymGetOptions();
        Options = Options | SYMOPT_DEBUG;
        SymSetOptions(Options);
        hProcess = GetCurrentProcess();
        BOOL bRet = SymInitialize(hProcess, 0, FALSE);
        if (!bRet)
        {
                printf("SymInitialize error ...\n");
        }
        char SymbolPath[256];
        GetCurrentDirectoryA(sizeof(SymbolPath), SymbolPath);
        strcat(SymbolPath, "\\symbols");
        SymSetSearchPath(hProcess, SymbolPath);
        char FileName[256];
        GetSystemDirectoryA(FileName, sizeof(FileName));
        strcat(FileName, "\\ntkrnlmp.exe");
        HANDLE hFile = CreateFileA(FileName, GENERIC_READ, FILE_SHARE_READ | FILE_SHARE_WRITE, NULL, OPEN_EXISTING, 0, NULL);
        DWORD dwfilesize = GetFileSize(hFile, NULL);
        BaseOfDll = SymLoadModule64(hProcess, NULL, FileName, NULL, ntkrnlmpBaseaddress, dwfilesize);
        if (BaseOfDll == 0)
        {
                DWORD nErr = GetLastError();
        }
        SymEnumSymbols(hProcess, BaseOfDll, 0, EnumSymCallBack, 0);
        SymUnloadModule64(hProcess, BaseOfDll);
        SymCleanup(hProcess);
        for (;;);
}
```

```c
int main()   用法
{
        ULONG64 ntkrnlmpBaseaddress;
        ZwQuerySystemInformation=(ZWQUERYSYSTEMINFORMATION)GetProcAddress(LoadLibraryW(L"ntdll.dll"),"ZwQuerySystemInformation");
        ntkrnlmpBaseaddress=EnumKM("ntkrnlmp.exe");//获得 NT内核模块基地址
        getallkrnladdress(ntkrnlmpBaseaddress);
        getchar();
        return 0;
}
```

完整 SRC :801K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4m8S2L8W2)9J5k6h3u0S2K9h3c8#2i4K6u0W2j5$3!0E0i4K6u0r3M7#2)9J5c8U0q4K6K9Y4g2K9k6K6u0p5

网上代码 抄抄改改 就好了 ，听说老大 都不用硬编码了 所以 ~~~~

![](https://bbs.kanxue.com/upload/attach/201509/656312_ftvdeo80syk0h2b.jpg)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [QQ截图20150904161515.jpg](javascript:void(0);)
  （69.33kb，114次下载）
