
# 防止 loader

> 原文：[`www.pediy.com/kssd/pediy03/forum1229.htm`](https://www.pediy.com/kssd/pediy03/forum1229.htm)

通过检查进程的父进程，可以防止 `loader` 方式的破解。一般情况下普通 Win32 进程的父进程是 `explorer.exe`。如果不是，则很可能是被其它 `loader` 加载了。

在 NT/2K 下，可以利用 native API 获得进程的父进程的 PID，进而可获得父进程的程序名。

```c
#include <windows.h>
#include <psapi.h>
#include <iostream.h>

#define NTAPI __stdcall

typedef long NTSTATUS;

#define NT_SUCCESS(Status) (((NTSTATUS)(Status)) >= 0)

enum PROCESS_INFO_CLASS { ProcessBasicInformation = 0 };

typedef struct _PROCESS_BASIC_INFORMATION
{
    NTSTATUS ExitStatus;
    ULONG PebBaseAddress;
    ULONG AffinityMask;
    ULONG BasePriority;
    ULONG UniqueProcessId;
    ULONG InheritedFromUniqueProcessId;
} PROCESS_BASIC_INFORMATION, *PPROCESS_BASIC_INFORMATION;

typedef NTSTATUS (NTAPI *ZW_QUERY_INFORMATION_PROCESS)(IN HANDLE ProcessHandle, IN PROCESS_INFO_CLASS ProcessInformationClass, OUT PVOID ProcessInformation, IN ULONG ProcessInformationLength, OUT PULONG ReturnLength);

ULONG GetParentPID(HANDLE hProcess)
{
    ULONG ParentPID = (ULONG)(-1);
    __try
    {
        HMODULE hModule = GetModuleHandle("ntdll.dll");
        if (!hModule)
            __leave;
        ZW_QUERY_INFORMATION_PROCESS ZwQueryInformationProcess;
        ZwQueryInformationProcess = (ZW_QUERY_INFORMATION_PROCESS)GetProcAddress(hModule, "ZwQueryInformationProcess");
        if (!ZwQueryInformationProcess)
            __leave;
        PROCESS_BASIC_INFORMATION ProcessInfo;
        NTSTATUS Status = ZwQueryInformationProcess(hProcess, ProcessBasicInformation, &ProcessInfo, sizeof(ProcessInfo), NULL);
        if (NT_SUCCESS(Status))
        {
            ParentPID = ProcessInfo.InheritedFromUniqueProcessId;
        }
    }
    __finally
    {
        //empty
    }
    return ParentPID;
}

void main(void)
{
    LONG ParentPID = GetParentPID(GetCurrentProcess());
    cout << "Parent PID: " << ParentPID << endl;
    HANDLE hParentProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, ParentPID);
    if (hParentProcess)
    {
        char FileName[MAX_PATH];
        DWORD Len = GetModuleFileNameEx(hParentProcess, NULL, FileName, MAX_PATH);
        if (Len)
        {
            cout << "Parent EXE name: " << FileName << endl;
        }
        CloseHandle(hParentProcess);
    }
}
```

在 NT/2K 下，普通 Win32 进程的父进程还可能是 `cmd.exe`；对于 Win32 服务进程，其父进程则为 `service.exe`。

`4dd466d0e1e0cc123afbe504da2462ca`
