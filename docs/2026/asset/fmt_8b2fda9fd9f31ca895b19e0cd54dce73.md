
# 4. 注入

对目标进程进行DLL注入，Hook `GetMoudleHandle`、`Loadlibary`等函数，让目标进程自己调用，进行IPC通信。

# 参考

<a href="elink@42aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9k6h3q4J5L8W2)9J5k6h3#2A6j5%4u0G2M7$3!0X3N6q4)9J5k6h3y4G2L8g2)9J5c8X3g2F1i4K6u0V1N6i4y4Q4x3V1k6%4K9h3&6V1L8%4N6K6i4K6u0r3N6$3W2F1x3K6u0Q4x3V1k6S2M7r3W2Q4x3V1k6%4K9h3&6@1k6i4u0F1L8q4)9J5c8X3&6K6i4K6u0V1N6$3W2F1N6r3g2J5L8X3I4Q4x3X3c8H3k6h3t1`.">PEB （winternl.h） - Win32 应用 |Microsoft 学习</a>

<a href="elink@9d2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9k6h3q4J5L8W2)9J5k6h3#2A6j5%4u0G2M7$3!0X3N6q4)9J5k6h3y4G2L8g2)9J5c8X3g2F1i4K6u0V1N6i4y4Q4x3V1k6%4K9h3&6V1L8%4N6K6i4K6u0r3N6$3W2F1x3K6u0Q4x3V1k6S2M7r3W2Q4x3V1k6%4K9h3&6@1k6i4u0F1L8q4)9J5c8X3&6K6i4K6u0V1N6$3W2F1N6r3g2J5L8X3I4Q4x3X3c8H3k6h3u0Q4y4h3k6D9k6s2u0Q4y4h3k6V1j5i4c8S2">PEB\_LDR\_DATA （winternl.h） - Win32 应用 |Microsoft 学习</a>

<a href="elink@246K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0L8X3u0D9L8$3N6K6i4K6u0W2j5$3!0E0i4K6u0r3j5X3!0C8k6i4u0F1j5W2)9J5c8Y4m8Q4x3V1j5$3y4o6l9@1y4K6V1#2i4K6u0W2K9s2c8E0L8l9`.`.">PEB及LDR链 - bokernb - 博客园 (cnblogs.com)</a>

<a href="elink@de2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8X3I4#2L8X3q4@1K9h3x3%4i4K6u0r3j5i4u0@1K9h3y4D9k6g2)9J5c8X3c8W2N6r3q4A6L8s2y4Q4x3V1j5I4x3U0b7@1y4U0l9J5y4U0p5`.">远程获取进程DLL模块地址\_获取进程模块-CSDN博客</a>

<a href="elink@24aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9k6h3q4J5L8W2)9J5k6h3#2A6j5%4u0G2M7$3!0X3N6q4)9J5k6h3y4G2L8g2)9J5c8Y4A6Z5i4K6u0V1j5$3&6Q4x3V1k6%4K9h3&6V1L8%4N6K6i4K6u0r3N6$3W2F1x3K6u0Q4x3V1k6S2M7r3W2Q4x3V1k6@1L8r3S2W2L8s2l9K6x3W2)9J5c8X3&6X3i4K6u0V1N6r3I4Z5k6h3I4H3x3K6u0Q4x3X3c8U0M7X3g2S2N6r3g2@1L8$3!0D9K9r3g2D9M7o6x3J5M7$3&6S2M7s2y4Z5L8%4b7`.">CreateToolhelp32Snapshot 函数 (tlhelp32.h) - Win32 apps | Microsoft Learn</a>

<a href="elink@9f6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9k6h3q4J5L8W2)9J5k6h3#2A6j5%4u0G2M7$3!0X3N6q4)9J5k6h3y4G2L8g2)9J5c8Y4A6Z5i4K6u0V1j5$3&6Q4x3V1k6%4K9h3&6V1L8%4N6K6i4K6u0r3N6$3W2F1x3K6u0Q4x3V1k6H3M7$3q4H3K9g2)9J5c8X3g2F1N6h3#2W2M7X3q4@1K9h3&6Y4i4K6u0V1j5h3I4D9i4K6u0V1L8h3!0V1N6h3I4W2M7#2)9J5k6r3k6G2M7W2)9J5k6r3q4Q4x3X3c8H3M7X3!0U0k6i4y4K6">枚举进程的所有模块 - Win32 apps | Microsoft Learn</a>

# 四种获取目标进程导入DLL模块地址的方法

本人在校大学生一枚，技术不太行，各位师傅将就看看。

## 1. 获取进程快照

`tlhelp32.h`头文件中，提供了`CreateToolhelp32Snapshot`API，可以获取获取指定进程以及这些进程使用的堆、模块和线程的快照。

```
HANDLE CreateToolhelp32Snapshot(
  [in] DWORD dwFlags,
  [in] DWORD th32ProcessID
);
```

![](https://bbs.kanxue.com/upload/attach/202312/991700_DA52GED3MQJ66HR.webp)

在进程快照信息中，包含着进程导入模块的信息。因此，我们可以有如下实现思路：

1. 调用`CreateToolhelp32Snapshot`指定`TH32CS_SNAPPROCESS`属性，获取进程快照句柄。
2. 使用`Process32Next`遍历进程信息，对比进程名查找到目标进程。
3. 使用`OpenProcess`验证目标进程状态。
4. 调用`CreateToolhelp32Snapshot`指定`TH32CS_SNAPMODULE`属性，获取目标进程的模块快照。
5. 使用`Module32Next`遍历模块，直到获取到目标模块信息，返回目标模块的句柄。
6. 后续干什么都可以了，例如使用`GetProcAddress`获取目标函数的地址。

```
#include <iostream>
#include <Windows.h>
#include <Tlhelp32.h>
#include <tchar.h>
#include <string>
#include <stdio.h>

HMODULE GetProcessModuleHandle(DWORD pid, CONST TCHAR* moduleName) {
    // 根据 PID 、模块名（需要写后缀，如：".dll"），获取模块入口地址
    MODULEENTRY32 moduleEntry;
    HANDLE handle = NULL;
    moudleInfoHandle = CreateToolhelp32Snapshot(TH32CS_SNAPMODULE, pid); //  获取进程快照的打开句柄，包含模块信息（TH32CS_SNAPMODULE指定）
    if (!moudleInfoHandle) {
        CloseHandle(moudleInfoHandle);
        return NULL;
    }
    ZeroMemory(&moduleEntry, sizeof(MODULEENTRY32));
    moduleEntry.dwSize = sizeof(MODULEENTRY32);
    if (!Module32First(moudleInfoHandle, &moduleEntry)) {
        CloseHandle(moudleInfoHandle);
        return NULL;
    }
    do {
        if (_tcscmp(moduleEntry.szModule, moduleName) == 0) {
            CloseHandle(moudleInfoHandle);
            return moduleEntry.hModule;
        }
    } while (Module32Next(moudleInfoHandle, &moduleEntry));
    CloseHandle(moudleInfoHandle);
    return 0;
}

int main() {
    HANDLE hProcessSnap = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0); // 进程快照句柄
    PROCESSENTRY32 process = { sizeof(PROCESSENTRY32) }; // 存放进程快照的结构体

    //  遍历进程
    while (Process32Next(hProcessSnap, &process)) {
        // 找到目标进程
        std::string s_szExeFile = process.szExeFile; // char* 转 string
        if (s_szExeFile == "xxx.exe") {
            HANDLE hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, process.th32ProcessID);
            if (hProcess)
            {
                HMODULE hMod = GetProcessModuleHandle(process.th32ProcessID, "xxx.dll");
                if (hMod)
                {
                    auto fnAddress = GetProcAddress(hMod, "xxFunc");

                    std::cout << fnAddress << "        " << GetLastError();
                }
            }
        }
    }
}
```

## 2. 枚举进程模块

按照微软官方的建议，使用`EnumProcessModules`函数枚举进程模块可以确定哪些进程加载了特定的DLL。这个函数在`psapi.h`中。以下是微软给出的实例代码。

```
BOOL EnumProcessModules(
  [in] HANDLE  hProcess,
  [out] HMODULE *lphModule,
  [in] DWORD   cb,
  [out] LPDWORD lpcbNeeded
);
```

1. 根据进程ID，调用`OpenProcess`获取进程句柄。
2. 调用`EnumProcessModules`函数枚举模块信息。模块句柄都返回到`hMods`数组中。进一步筛选即可。

```
#include <windows.h>
#include <tchar.h>
#include <stdio.h>
#include <psapi.h>

// 为了确保正确解析符号，需要将 Psapi.lib 添加到 TARGETLIBS，并使用 -DPSAPI_VERSION=1 编译

int PrintModules( DWORD processID )
{
    HMODULE hMods[1024];
    HANDLE hProcess;
    DWORD cbNeeded;
    unsigned int i;

    // 打印进程标识符。

    printf( "\n进程 ID：%u\n", processID );

    // 获取进程的句柄。

    hProcess = OpenProcess( PROCESS_QUERY_INFORMATION |
        PROCESS_VM_READ,
        FALSE, processID );
    if (NULL == hProcess)
        return 1;

    // 获取此进程中所有模块的列表。

    if( EnumProcessModules(hProcess, hMods, sizeof(hMods), &cbNeeded))
    {
        for ( i = 0; i < (cbNeeded / sizeof(HMODULE)); i++ )
        {
            TCHAR szModName[MAX_PATH];

            // 获取模块文件的完整路径。

            if ( GetModuleFileNameEx( hProcess, hMods[i], szModName,
                sizeof(szModName) / sizeof(TCHAR)))
            {
                // 打印模块名和句柄值。

                _tprintf( TEXT("\t%s (0x%08X)\n"), szModName, hMods[i] );
            }
        }
    }

    // 释放对进程的句柄。

    CloseHandle( hProcess );

    return 0;
}

int main( void )
{

    DWORD aProcesses[1024];
    DWORD cbNeeded;
    DWORD cProcesses;
    unsigned int i;

    // 获取进程标识符的列表。

    if ( !EnumProcesses( aProcesses, sizeof(aProcesses), &cbNeeded ) )
        return 1;

    // 计算返回了多少个进程标识符。

    cProcesses = cbNeeded / sizeof(DWORD);

    // 打印每个进程的模块名称。

    for ( i = 0; i < cProcesses; i++ )
    {
        PrintModules( aProcesses[i] );
    }

    return 0;
}
```

## 3. 获取PEB

`PEB +0x00c`成员`Ldr`指向`_PEB_LDR_DATA`结构，此结构的第三成员`InMemoryOrderModuleList`包含进程的已加载模块的双向链表的头部。列表中的每个项目都是指向`LDR_DATA_TABLE_ENTRY`结构的指针。

```
typedef struct _LDR_DATA_TABLE_ENTRY {
  PVOID Reserved1[2];
  LIST_ENTRY InMemoryOrderLinks;
  PVOID Reserved2[2];
  PVOID DllBase;
  PVOID EntryPoint;
  PVOID Reserved3;
  UNICODE_STRING FullDllName;
  BYTE Reserved4[8];
  PVOID Reserved5[3];
  union {
    ULONG CheckSum;
    PVOID Reserved6;
  };
  ULONG TimeDateStamp;
} LDR_DATA_TABLE_ENTRY, *PLDR_DATA_TABLE_ENTRY;
```

因此，这种方法的思路如下：

1. 调用`NtWow64QueryInformationProcess64`获取`PROCESS_BASIC_INFORMATION64(32)`结构
2. 调用`NtWow64ReadVirtualMemory64`（32位用`ReadProcessMemory`）读出`PebBaseAddress`
3. 获取到PEB，定位到`_PEB_LDR_DATA`遍历`_LDR_DATA_TABLE_ENTRY`
4. 通过`FullDllName`筛选，`DllBase`返回模块基地址。
