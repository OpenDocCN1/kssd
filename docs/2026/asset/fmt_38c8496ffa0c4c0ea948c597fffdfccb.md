
``` 
BOOL CheckDebug()  

{  

    PIMAGE_DOS_HEADER pDosHeader;  

    PIMAGE_NT_HEADERS32 pNtHeaders;  

    PIMAGE_SECTION_HEADER pSectionHeader;  

    DWORD dwBaseImage = (DWORD)GetModuleHandle(NULL);   

    pDosHeader = (PIMAGE_DOS_HEADER)dwBaseImage;  

    pNtHeaders = (PIMAGE_NT_HEADERS32)((DWORD)pDosHeader + pDosHeader->e_lfanew);  

    pSectionHeader = (PIMAGE_SECTION_HEADER)((DWORD)pNtHeaders + sizeof(pNtHeaders->Signature) + sizeof(IMAGE_FILE_HEADER) +  

        (WORD)pNtHeaders->FileHeader.SizeOfOptionalHeader);  

    DWORD dwAddr = pSectionHeader->VirtualAddress + dwBaseImage;   

    DWORD dwCodeSize = pSectionHeader->SizeOfRawData;      

    DWORD checksum = 0;  

    __asm  

    {  

        cld  

        mov     esi, dwAddr  

        mov     ecx, dwCodeSize  

        xor eax, eax  

    checksum_loop :  

        movzx    ebx, byte ptr[esi]  

        add        eax, ebx  

        rol eax, 1  

        inc esi  

        loop       checksum_loop  

        mov checksum, eax  

    }  

    if (checksum != 0x46ea24)  

    {  

        return FALSE;  

    }  

    else  

    {  

        return TRUE;  

    }  

}
```

## 4.时钟检测

被调试时，进程的运行速度大大降低，例如，单步调试大幅降低恶意代码的运行速度，所以时钟检测是恶意代码探测调试器存在的最常用方式之一。有如下两种用时钟检测来探测调试器存在的方法。

记录一段操作前后的时间戳，然后比较这两个时间戳，如果存在滞后，则可以认为存在调试器。

记录触发一个异常前后的时间戳。如果不调试进程，可以很快处理完异常，因为调试器处理异常的速度非常慢。默认情况下，调试器处理异常时需要人为干预，这导致大量延迟。虽然很多调试器允许我们忽略异常，将异常直接返回程序，但这样操作仍然存在不小的延迟。

### 4.1使用rdtsc指令

较常用的时钟检测方法是利用rdtsc指令(操作码0x0F31)，它返回至系统重新启动以来的时钟数，并且将其作为一个64位的值存入EDX:EAX中。恶意代码运行两次rdtsc指令，然后比较两次读取之间的差值。

```
BOOL CheckDebug()  

{  

    DWORD time1, time2;  

    __asm  

    {  

        rdtsc  

        mov time1, eax  

        rdtsc  

        mov time2, eax  

    }  

    if (time2 - time1 < 0xff)  

    {  

        return FALSE;  

    }  

    else  

    {  

        return TRUE;  

    }  

}
```

### 4.2使用QueryPerformanceCounter和GetTickCount

同rdtsc指令一样，这两个Windows API函数也被用来执行一个反调试的时钟检测。使用这种方法的前提是处理器有高分辨率能力的计数器-寄存器，它能存储处理器活跃的时钟数。为了获取比较的时间差，调用两次QueryPerformanceCounter函数查询这个计数器。若两次调用之间花费的时间过于长，则可以认为正在使用调试器。GetTickCount函数返回最近系统重启时间与当前时间的相差毫秒数(由于时钟计数器的大小原因，计数器每49.7天就被重置一次)。

```
BOOL CheckDebug()  

{  

    DWORD time1 = GetTickCount();  

    __asm  

    {  

        mov     ecx,10             

        mov     edx,6                 

        mov     ecx,10  

    }  

    DWORD time2 = GetTickCount();  

    if (time2-time1 > 0x1A)  

    {  

        return TRUE;  

    }  

    else  

    {  

        return FALSE;  

    }  

}
```

## 5.判断父进程是否是explorer.exe

一般双击运行的进程的父进程都是explorer.exe，但是如果进程被调试父进程则是调试器进程。也就是说如果父进程不是explorer.exe则可以认为程序正在被调试。

```
BOOL CheckDebug()  

{  

    LONG                      status;    

    DWORD                     dwParentPID = 0;    

    HANDLE                    hProcess;    

    PROCESS_BASIC_INFORMATION pbi;    

    int pid = getpid();  

    hProcess = OpenProcess(PROCESS_QUERY_INFORMATION, FALSE, pid);    

    if(!hProcess)    

        return -1;    

    PNTQUERYINFORMATIONPROCESS  NtQueryInformationProcess = (PNTQUERYINFORMATIONPROCESS)GetProcAddress(GetModuleHandleA("ntdll"),"NtQueryInformationProcess");  

    status = NtQueryInformationProcess(hProcess,SystemBasicInformation,(PVOID)&pbi,sizeof(PROCESS_BASIC_INFORMATION),NULL);  

    PROCESSENTRY32 pe32;  

    pe32.dwSize = sizeof(pe32);   

    HANDLE hProcessSnap = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);   

    if(hProcessSnap == INVALID_HANDLE_VALUE)   

    {   

        return FALSE;   

    }  

    BOOL bMore = Process32First(hProcessSnap, &pe32);   

    while(bMore)  

    {  

        if (pbi.InheritedFromUniqueProcessId == pe32.th32ProcessID)  

        {  

            if (stricmp(pe32.szExeFile, "explorer.exe")==0)  

            {  

                CloseHandle(hProcessSnap);  

                return FALSE;  

            }  

            else  

            {  

                CloseHandle(hProcessSnap);  

                return TRUE;  

            }  

        }  

        bMore = Process32Next(hProcessSnap, &pe32);   

    }  

    CloseHandle(hProcessSnap);  

}
```

## 6.判断STARTUPINFO信息

explorer.exe创建进程的时候会把STARTUPINFO结构中的值设为0，而非explorer.exe创建进程的时候会忽略这个结构中的值，也就是结构中的值不为0。所以可以利用STARTUPINFO来判断程序是否在被调试。

```
BOOL CheckDebug()  

{  

    STARTUPINFO si;  

    GetStartupInfo(&si);  

    if (si.dwX!=0 || si.dwY!=0 || si.dwFillAttribute!=0 || si.dwXSize!=0 || si.dwYSize!=0 || si.dwXCountChars!=0 || si.dwYCountChars!=0)  

    {  

        return TRUE;  

    }  

    else  

    {  

        return FALSE;  

    }  

}
```

## 7.判断是否具有SeDebugPrivilege权限

默认情况下进程是没有SeDebugPrivilege权限的，但是当进程通过调试器启动时，由于调试器本身启动了SeDebugPrivilege权限，当调试进程被加载时SeDebugPrivilege也就被继承了。所以我们可以检测进程的SeDebugPrivilege权限来间接判断是否存在调试器，而对SeDebugPrivilege权限的判断可以用能否打开csrss.exe进程来判断。

```
BOOL CheckDebug()  

{  

    DWORD ID;  

    DWORD ret = 0;  

    PROCESSENTRY32 pe32;  

    pe32.dwSize = sizeof(pe32);   

    HANDLE hProcessSnap = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);   

    if(hProcessSnap == INVALID_HANDLE_VALUE)   

    {   

        return FALSE;   

    }  

    BOOL bMore = Process32First(hProcessSnap, &pe32);   

    while(bMore)  

    {  

        if (strcmp(pe32.szExeFile, "csrss.exe")==0)  

        {  

            ID = pe32.th32ProcessID;  

            break;  

        }  

        bMore = Process32Next(hProcessSnap, &pe32);   

    }  

    CloseHandle(hProcessSnap);   

    if (OpenProcess(PROCESS_QUERY_INFORMATION, NULL, ID) != NULL)  

    {  

        return TRUE;  

    }  

    else  

    {  

        return FALSE;  

    }  

}
```

# 三、干扰调试器的功能

恶意代码可以用一些技术来干扰调试器的正常运行。例如线程本地存储(TLS)回调、插入中断、异常等。这些技术当且仅当程序处于调试器控制之下时才试图扰乱程序的运行。

## 1.使用TLS回调

Thread Local Storage(TLS)，即线程本地存储，是Windows为解决一个进程中多个线程同时访问全局变量而提供的机制。TLS可以简单地由操作系统代为完成整个互斥过程，也可以由用户自己编写控制信号量的函数。当进程中的线程访问预先制定的内存空间时，操作系统会调用系统默认的或用户自定义的信号量函数，保证数据的完整性与正确性。下面是一个简单的TLS回调的例子，TLS_CALLBACK1函数在main函数执行前调用IsDebuggerPresent函数检查它是否正在被调试。

```
#include "stdafx.h"  

#include <stdio.h>  

#include <windows.h>  

  

void NTAPI __stdcall TLS_CALLBACK1(PVOID DllHandle, DWORD dwReason, PVOID Reserved);  

  

#ifdef _M_IX86  

#pragma comment (linker, "/INCLUDE:__tls_used")  

#pragma comment (linker, "/INCLUDE:__tls_callback")  

#else  

#pragma comment (linker, "/INCLUDE:_tls_used")  

#pragma comment (linker, "/INCLUDE:_tls_callback")  

#endif  

EXTERN_C  

#ifdef _M_X64  

#pragma const_seg (".CRT$XLB")  

const  

#else  

#pragma data_seg (".CRT$XLB")  

#endif  

  

PIMAGE_TLS_CALLBACK _tls_callback[] = { TLS_CALLBACK1,0};  

#pragma data_seg ()  

#pragma const_seg ()  

  

#include <iostream>  

  

void NTAPI __stdcall TLS_CALLBACK1(PVOID DllHandle, DWORD Reason, PVOID Reserved)  

{  

    if (IsDebuggerPresent())  

    {  

        printf("TLS_CALLBACK: Debugger Detected!\n");  

    }  

    else  

    {  

        printf("TLS_CALLBACK: No Debugger Present!\n");  

    }  

}  

  

int main(int argc, char* argv[])  

{  

    printf("233\n");  

    return 0;  

}
```

要在程序中使用TLS，必须为TLS数据单独建一个数据段，用相关数据填充此段，并通知链接器为TLS数据在PE文件头中添加数据。_tls_callback[]数组中保存了所有的TLS回调函数指针。数组必须以NULL指针结束，且数组中的每一个回调函数在程序初始化时都会被调用，程序员可按需要添加。但程序员不应当假设操作系统已何种顺序调用回调函数。如此则要求在TLS回调函数中进行反调试操作需要一定的独立性。

正常运行这个程序会打印下面的内容。

TLS_CALLBACK: No Debugger Present!

233

如果把在OllyDbg中运行，在OllyDbg暂停之前会打印下面的内容。

TLS_CALLBACK: Debugger Detected!

使用PEview查看.tls段，可以发现TLS回调函数。通常情况下，正常程序不使用.tls段，如果在可执行程序中看到.tls段，应该立即怀疑它使用了反调试技术。

![](https://bbs.kanxue.com/upload/attach/201804/734571_BZDANDDPZ3HN68Q.jpg)

在OllyDbg中选择Options->Debugging Options->Events，然后设置System break-point作为第一个暂停的位置，这样就可以让OllyDbg在TLS回调执行前暂停。

![](https://bbs.kanxue.com/upload/attach/201804/734571_VT82HFSA5HUWDUB.jpg)

在IDA Pro中按Ctrl+E快捷键看到二进制的入口点，该组合键的作用是显示应用程序所有的入口点，其中包括TLS回调。双击函数名可以浏览回调函数。

![](https://bbs.kanxue.com/upload/attach/201804/734571_QUBDAXKP2E7UFQC.jpg)

由于TLS回调已广为人知，因此同过去相比，恶意代码使用它的次数已经明显减少。为数不多的合法程序使用TLS回调，所以可执行程序中的.tls段特别突出。

## 2.利用中断
