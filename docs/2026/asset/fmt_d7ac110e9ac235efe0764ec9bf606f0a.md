
```
#include <Windows.h>

#include <tlhelp32.h>

#include <stdio.h>

#include <iostream>

#include <string>

DWORD findPidByName(const char* pname) //获得pid函数，对上面的查找进行改进

{

    HANDLE h;

    PROCESSENTRY32 procSnapshot;

    h = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);

    procSnapshot.dwSize = sizeof(PROCESSENTRY32);



    do

    {

        if (!_stricmp(procSnapshot.szExeFile, pname))

        {

            DWORD pid = procSnapshot.th32ProcessID;

            CloseHandle(h);

#ifdef _DEBUG

            printf(("[+]进程地址: %ld\n"), pid);

#endif

            return pid;

        }

    } while (Process32Next(h, &procSnapshot));



    CloseHandle(h);

    return 0;

}



typedef DWORD(WINAPI* pRtlCreateUserThread)(    //函数申明

    IN HANDLE                     ProcessHandle,

    IN PSECURITY_DESCRIPTOR     SecurityDescriptor,

    IN BOOL                     CreateSuspended,

    IN ULONG                    StackZeroBits,

    IN OUT PULONG                StackReserved,

    IN OUT PULONG                StackCommit,

    IN LPVOID                    StartAddress,

    IN LPVOID                    StartParameter,

    OUT HANDLE                     ThreadHandle,

    OUT LPVOID                    ClientID

    );



DWORD RtlCreateUserThread(LPCSTR pszLibFile, DWORD dwProcessId)

{

    pRtlCreateUserThread RtlCreateUserThread = NULL;

    HANDLE  hRemoteThread = NULL;



    HANDLE hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, dwProcessId); //打开目标进程pid

    if (hProcess == NULL)

    {

        printf("[-] Error: Could not open process for PID (%d).\n", dwProcessId);

        exit(1);

    }



    LPVOID LoadLibraryAddress = (LPVOID)GetProcAddress(GetModuleHandle("kernel32.dll"), "LoadLibraryA");  //load函数地址

    if (LoadLibraryAddress == NULL)

    {

        printf(("[-] Error: Could not find LoadLibraryA function inside kernel32.dll library.\n"));

        exit(1);

    }



    RtlCreateUserThread = (pRtlCreateUserThread)GetProcAddress(GetModuleHandle(("ntdll.dll")), ("RtlCreateUserThread"));//获取Rtl函数地址

    if (RtlCreateUserThread == NULL)

    {

        exit(1);

    }



#ifdef _DEBUG

    printf(("[+]RtlCreateUserThread函数地址： 0x%08x\n"), (UINT)RtlCreateUserThread);

    printf(("[+]LoadLibraryA函数地址： 0x%08x\n"), (UINT)LoadLibraryAddress);

#endif



    DWORD dwSize = (strlen(pszLibFile) + 1) * sizeof(char);



    LPVOID lpBaseAddress = VirtualAllocEx(hProcess, NULL, dwSize, MEM_COMMIT | MEM_RESERVE, PAGE_EXECUTE_READWRITE);//开辟一段地址

    if (lpBaseAddress == NULL)

    {

        printf(("[-] Error: Could not allocate memory inside PID (%d).\n"), dwProcessId);

        exit(1);

    }



    BOOL bStatus = WriteProcessMemory(hProcess, lpBaseAddress, pszLibFile, dwSize, NULL);  //目标dll写入

    if (bStatus == 0)

    {

        printf(("[-] Error: Could not write any bytes into the PID (%d) address space.\n"), dwProcessId);

        return(1);

    }



    bStatus = (BOOL)RtlCreateUserThread(

        hProcess,

        NULL,

        0,

        0,

        0,

        0,

        LoadLibraryAddress,  //shellcode

        lpBaseAddress,

        &hRemoteThread,

        NULL);

    if (bStatus < 0)

    {

        printf(("[-]注入失败\n"));

        return(1);

    }

    else

    {

        printf(("[+]注入成功...\n"));

        WaitForSingleObject(hRemoteThread, INFINITE);



        CloseHandle(hProcess);

        VirtualFreeEx(hProcess, lpBaseAddress, dwSize, MEM_RELEASE);

        return(0);

    }



    return(0);

}

int main()

{

    const char* name = ("菜单dome.exe");



    DWORD pId = findPidByName(name);

    LPCSTR location = ("S:\\panny_dll.dll");



    RtlCreateUserThread(location, pId);

}
```

还是运行`菜单dome.exe`：

![](https://bbs.kanxue.com/upload/attach/202004/856851_KGCU34WR2C7EK45.jpg)

![](https://bbs.kanxue.com/upload/attach/202004/856851_RD5N82VSKSZWDYJ.jpg)

（其实变蓝就已经在提示有`dll`注入了）

[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)
