
# 得到进程的父进程

> 原文：[`www.pediy.com/kssd/pediy04/kanxue036.htm`](https://www.pediy.com/kssd/pediy04/kanxue036.htm)

上次贴的防止 `loader` 的办法中使用了 `NT` 中的 `native API`，所以无法在 `Win9x/ME` 中使用。

利用 `Win32` 提供的 `ToolHelp API`，也可以得到一个进程的父进程的 `PID`。`Win9x/ME` 和 `Win2K/XP` 都支持 `ToolHelp API`，但 `WinNT(3.51, 4.0)` 却不支持。

```c
#include <windows.h>
#include <tlhelp32.h>
#include <psapi.h>
#include <stdio.h>

#pragma comment(lib, "psapi")

void DisplayProcessInfo(DWORD pid)
{
    printf("PID = %8u ", pid);
    HANDLE hProcess = OpenProcess (PROCESS_ALL_ACCESS, FALSE, pid);
    char FileName[MAX_PATH];
    if (hProcess)
    {
        if (GetModuleFileNameEx(hProcess, NULL, FileName, MAX_PATH))
        {
            printf("%s\n", FileName);
        }
        else
        {
            printf("无法获得进程名\n");
        }
        CloseHandle (hProcess);
    }
    else
    {
        printf("无权访问该进程\n");
    }
}

void main(void)
{
    HANDLE hProcessSnap = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);
    if (hProcessSnap == INVALID_HANDLE_VALUE)
    {
        printf("无法创建进程的快照\n");
        return;
    }

    PROCESSENTRY32 pe32 = {0};
    pe32.dwSize = sizeof(PROCESSENTRY32);
    if (!Process32First(hProcessSnap, &pe32))
    {
        printf("无法获得进程列表\n");
        CloseHandle (hProcessSnap);    //少了这句
        return;
    }

    do
    {
        DisplayProcessInfo(pe32.th32ProcessID);
        DisplayProcessInfo(pe32.th32ParentProcessID);
        printf("\n");
    } while (Process32Next(hProcessSnap, &pe32));

    CloseHandle (hProcessSnap);
}
```
