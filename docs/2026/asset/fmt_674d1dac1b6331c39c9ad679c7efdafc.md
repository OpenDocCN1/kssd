
### `GetExportApiAddr`

```c
DWORD GetExportApiAddr(char* szPath, char* szApiName, DWORD64* ExportTableAddrRva)
{
    // 映射文件到内存  找出需要的数据
    HANDLE hFile = CreateFile(szPath, GENERIC_READ, 0, NULL, OPEN_EXISTING, 0, NULL);
    HANDLE hMap = CreateFileMapping(hFile, NULL, PAGE_READONLY, 0, 0, NULL);
    DWORD64 lpBuff = MapViewOfFile(hMap, FILE_MAP_READ, 0, 0, NULL);
    PIMAGE_DOS_HEADER  pDosHeader = (PIMAGE_DOS_HEADER)lpBuff;
    PIMAGE_NT_HEADERS  pNtHeader = (PIMAGE_NT_HEADERS)((DWORD64)lpBuff + pDosHeader->e_lfanew);
    PIMAGE_EXPORT_DIRECTORY  pExportTable = lpBuff + RVA_TO_FOA(lpBuff, pNtHeader->OptionalHeader.DataDirectory[0].VirtualAddress);
    // 所有地址转为FOA
    DWORD64* pAddressOfFunctions = (DWORD64*)RVA_TO_FOA(lpBuff, pExportTable->AddressOfFunctions);
    pAddressOfFunctions = lpBuff + (DWORD64)pAddressOfFunctions;

    DWORD64* pAddressOfNames = (DWORD64*)RVA_TO_FOA(lpBuff, pExportTable->AddressOfNames);
    pAddressOfNames = lpBuff + (DWORD64)pAddressOfNames;

    DWORD64* pAddressOfNameOrdinals = (DWORD64*)RVA_TO_FOA(lpBuff, pExportTable->AddressOfNameOrdinals);
    pAddressOfNameOrdinals = lpBuff + (DWORD64)pAddressOfNameOrdinals;

    DWORD dwNumberOfNames = pExportTable->NumberOfNames;
    // 判断导出方式（序号 or 名称）
    if ((DWORD)szApiName >> 16 == 0)
    {
        DWORD Index = (DWORD64)szApiName - pExportTable->Base;
        if (Index >= pExportTable->NumberOfFunctions) return NULL;
        return (DWORD64)(lpBuff + pAddressOfFunctions[Index]);
    }
    else
    {
        DWORD64      pNameAddress;
        DWORD64   FuncRva;
        pNameAddress = (DWORD64)(lpBuff + RVA_TO_FOA(lpBuff, (DWORD)*pAddressOfNames));
        for (size_t i = 0; i < dwNumberOfNames; i++)
        {
            if (strcmp(szApiName, (char*)pNameAddress) == 0)
            {
                FuncRva = (WORD)*pAddressOfNameOrdinals;
                pAddressOfFunctions = (DWORD64)pAddressOfFunctions + FuncRva * 4;
                FuncRva = (DWORD)(*pAddressOfFunctions);
                *ExportTableAddrRva = (DWORD64)pAddressOfFunctions - lpBuff;
                *ExportTableAddrRva = FOA_TO_RVA(lpBuff, *ExportTableAddrRva);
                UnmapViewOfFile(lpBuff);
                CloseHandle(hFile);
                return FuncRva;
            }
            pAddressOfNameOrdinals = (DWORD64)pAddressOfNameOrdinals + 2;
            pAddressOfNames = (DWORD64)pAddressOfNames + 4;
            pNameAddress = (DWORD64)(lpBuff + RVA_TO_FOA(lpBuff, (DWORD)*pAddressOfNames));
        }
    }
    UnmapViewOfFile(lpBuff);
    CloseHandle(hFile);
    return NULL;
}
```

### 获取目标进程的基址

这个也是个巨坑啊，首先我选择使用遍历模块的函数，但都失败，最后经过研究得知，当你创建挂起进程的时候，由于程序没有运行，所以系统不会更新此程序在三环的快照信息（遍历的几个函数都是使用快照获取）

所以我这里给出两个办法

第一个， 使用Nt半公开函数 `NtQueryInformationProcess` ，这个函数也通常用来做反调试

```c
// 第二个参数是ProcessBasicInformation的话，则第三个参数必须为一个指针指向结构PROCESS_BASIC_INFORMATION
// 可以获取PEB ， 通过PEB获取基址
// loadlibrary  getprocaddress 获取函数地址
__kernel_entry NTSTATUS NtQueryInformationProcess(
    [in]            HANDLE           ProcessHandle,
    [in]            PROCESSINFOCLASS ProcessInformationClass,
    [out]           PVOID            ProcessInformation,
    [in]            ULONG            ProcessInformationLength,
    [out, optional] PULONG           ReturnLength
);

typedef enum _PROCESSINFOCLASS {
    ProcessBasicInformation = 0,
    ProcessDebugPort = 7, // 调式端口
    ProcessWow64Information = 26,
    ProcessImageFileName = 27,
    ProcessBreakOnTermination = 29
    ProcessDebugObjectHandle = 30, // 获取调试对象句柄，句柄为空表示未调试
    ProcessDebugFlags = 31        // 检测调试标志位为0表示处于调试状态
} PROCESSINFOCLASS;

typedef struct
{
    DWORD ExitStatus; // 接收进程终止状态
    DWORD PebBaseAddress; // 接收进程环境块地址   PEB
    DWORD AffinityMask; // 接收进程关联掩码
    DWORD BasePriority; // 接收进程的优先级类
    ULONG UniqueProcessId; // 接收进程ID
    ULONG InheritedFromUniqueProcessId; // 接收父进程ID
} PROCESS_BASIC_INFORMATION;
```

第二个办法

获取线程环境，根据病毒分析经验，在创建进程挂起时，进程的ebx会指向PEB ，在64位程序中 rdx会指向PEB，而64位PEB结构中，PEB+0x10的偏移为进程基址，好的，这样就得到了Base

```c
if (!GetThreadContext(pi.hThread, &context))
{
    printf("GetThreadContext failed (%d)\n", GetLastError());
    return 0;
}

// 获取进程基址
if (!ReadProcessMemory(pi.hProcess, (LPVOID)(context.Rdx + 0x10), &BaseAddr, sizeof(BaseAddr), &retByte))
{
    printf("ReadProcessMemory failed (%d)\n", GetLastError());
    return 0;
}
```

### 申请内存与hook

这里也有坑

根据前面的分析流程，现在应该hook导出函数地址表了，需要得到shellcode的地址，地址需要去对方进程申请

而导出表函数地址表中存放的函数的RVA ，且只有4字节，所以不得不考虑一个问题，如果申请到的地址是64位地址，大于FFFFFFFF则我们没法填入导出函数地址表（所以如果你有导出函数则进程不能大于4个G？微软这PE格式是不是该升级了）

```c
// 申请内存 （地址不能与主模块间隔超过FFFFFFFF ）
shellcodeAddr = VirtualAllocEx(pi.hProcess, BaseAddr + 0x10000000, 0x2000, MEM_COMMIT | MEM_RESERVE, PAGE_EXECUTE_READWRITE);
if (shellcodeAddr == NULL)
{
    printf("VirtualAllocEx failed (%d)\n", GetLastError());
    return 0;
}
```

这里我选择在基址附近申请一个内存，不超过4G即可，那么接下来该考虑hook的问题了

hook需要得到对方进程的VA ，前面说过了

```c
// 获得hook函数的VA
ExportFuncVA = BaseAddr + ExportFuncRVA;
// 获得导出表的VA
ExportTableVA = BaseAddr + ExportTableVA;
// hook导出函数  shellcode VA 转为 RVA  修改内存属性后写入
shellcodeRva = shellcodeAddr - BaseAddr;
```

最后要注意修改时，要调整内存保护属性

### Shellcode的编写

这个是难点，为了写出精致的shellcode，我甚至还得用汇编实现算法

另一个难点主要是Shellcode的编写需要依赖于逆向分析的结果，Shellcode以及注释在下边给出

这里64位程序不支持内联汇编，那么怎么写呢，两个办法 ，联合编译 与 使用机器码 ， 这里我选择64位的联合编译 ， 怎么实现联合编译呢 ，第一种办法如下， 第二种办法 写好汇编使用ml64汇编编译器生成obj文件，在C程序编译中链接此文件

新建一个asm后缀的文件，修改它的属性 ， 从生成中排除 否 ，自定义生成工具

![](https://bbs.kanxue.com/upload/attach/202205/930159_S69WKGJKZKVDF2R.png)

![](https://bbs.kanxue.com/upload/attach/202205/930159_79JA4F7T63ATT4N.png)

按照图中的配置即可

![](https://bbs.kanxue.com/upload/attach/202205/930159_3C8Q9BWW7YA8XDD.png)

最后需要在C文件中，extern 汇编函数，即可愉快的使用联合编译

这里给出shellcode的汇编代码 **注意不能使用绝对地址**
