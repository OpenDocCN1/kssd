
# 利用 NtProtectVirtualMemory 结束进程

> 原文：[`www.pediy.com/kssd/pediy12/137067.html`](https://www.pediy.com/kssd/pediy12/137067.html)

原理很简单，用 `PROCESS_VM_OPERATION` 打开目标进程（没必要 `PROCESS_ALL_ACCESS`），把目标进程的 `ntdll.dll` 设为不能访问

```c
/*

 * 【作者：KiDebug】

 * 【空间：http://hi.baidu.com/KiDebug/】

 *  VC 6.0 编译出错请百度：“vc 6.0 unicode”

 */

#include <stdio.h>

#include <Windows.h>

#include <Psapi.h>

#include <Tlhelp32.h>

#pragma comment(lib,"Psapi.lib")

typedef NTSTATUS (__stdcall *RtlAdjustPrivilege_)( 

    ULONG Privilege, 

    BOOLEAN Enable, 

    BOOLEAN CurrentThread, 

    PBOOLEAN Enabled 

    );

RtlAdjustPrivilege_ RtlAdjustPrivilege = NULL;

typedef NTSTATUS (__stdcall *NtProtectVirtualMemory_)( 

    __in HANDLE ProcessHandle, 

    __inout PVOID *BaseAddress, 

    __inout PSIZE_T RegionSize, 

    __in ULONG NewProtectWin32, 

    __out PULONG OldProtect 

    );

NtProtectVirtualMemory_ NtProtectVirtualMemory = NULL;

ULONG GetPID (WCHAR* proc) 

{ 

    BOOL                working =   0; 

    PROCESSENTRY32      lppe    = {0}; 

    ULONG               targetPid=0; 

    HANDLE hSnapshot    =   CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS ,0); 

    if ( hSnapshot ) 

    { 

        lppe.dwSize = sizeof( lppe ); 

        working = Process32First( hSnapshot, &lppe ); 

        while ( working ) 

        { 

            if ( _wcsicmp( lppe.szExeFile,proc) ==0 ) 

            { 

                targetPid = lppe.th32ProcessID; 

                break; 

            } 

            working = Process32Next(hSnapshot,&lppe); 

        } 

    } 

    CloseHandle( hSnapshot ); 

    return targetPid; 

} 

void main() 

{ 

    HMODULE     ntdll; 

    MODULEINFO  ModuleInfo; 

    ntdll   =   GetModuleHandle( L"ntdll.dll" ); 

    if ( !GetModuleInformation( (HANDLE)-1, ntdll, &ModuleInfo, sizeof(MODULEINFO) ) ) 

    { 

        return; 

    }    

    BOOLEAN         Enabled; 

    RtlAdjustPrivilege =  (RtlAdjustPrivilege_)GetProcAddress( ntdll, "RtlAdjustPrivilege" ); 

    if ( RtlAdjustPrivilege ==NULL ) 

    { 

        return; 

    } 

    RtlAdjustPrivilege( 20, TRUE, FALSE, &Enabled ); 

    HANDLE hProc    =   OpenProcess( PROCESS_VM_OPERATION, FALSE, GetPID(L"services.exe") ); 

    if ( hProc ==   NULL ) 

    { 

        return; 

    } 

    NtProtectVirtualMemory =  (NtProtectVirtualMemory_)GetProcAddress( ntdll, "NtProtectVirtualMemory" ); 

    if ( NtProtectVirtualMemory ==  NULL ) 

    { 

        return; 

    } 

    ULONG   OldProtect; 

    NtProtectVirtualMemory( hProc, &ModuleInfo.lpBaseOfDll, &ModuleInfo.SizeOfImage, PAGE_NOACCESS, &OldProtect ); 

}
```
