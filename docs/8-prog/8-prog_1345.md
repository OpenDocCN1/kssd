
# 发一段卸载 DLL 模块的代码，来自 IceSword

> 原文：[`www.pediy.com/kssd/pediy10/99671.html`](https://www.pediy.com/kssd/pediy10/99671.html)

不知道谁发过类似的代码，网上搜相关信息有一篇 `DELPHI` 版的与此类似。总之，分享给大家吧，从看雪学了很多东西，最近比较迷茫。总想对看雪有点啥贡献，但总觉得拿不出手，慢慢来吧。

此代码逆向自 `IceSword` 中的卸载 `DLL` 功能，貌似还有个强力卸载，代码只有一句吧，但是效果感觉没这个好。总之，能用上的可以参考吧。

```c
----------------------------------FreeLibraryEx.h----------------------------------

// FreeLibraryEx.h

// made by lynn, reversed from IceSword

// needed : SmartHandle.h

// support w2k, xp, vista (not tested on win7)

#pragma once

#include <windows.h>

//卸载模块函数，可以再封装一下，适用版本：W2K 以上，支持 VISTA

BOOL FreeLibraryEx(HANDLE hProcess,
                  DWORD dwPid,
                  HMODULE hModule
                  );

//////////////////////////////////////////////////////////////////////////

//以下不用管了

#define RTL_DEBUG_QUERY_MODULES                          0x01

#define RTL_DEBUG_QUERY_BACKTRACES                       0x02

#define RTL_DEBUG_QUERY_HEAPS                              0x04

#define RTL_DEBUG_QUERY_HEAP_TAGS                          0x08

#define RTL_DEBUG_QUERY_HEAP_BLOCKS                          0x10

#define RTL_DEBUG_QUERY_LOCKS                              0x20

typedef struct _RTL_PROCESS_MODULE_INFORMATION
{
    ULONG Reserved[2];
    PVOID Base;
    ULONG Size;
    ULONG Flags;
    USHORT Index;
    USHORT Unknown;
    USHORT LoadCount;
    USHORT ModuleNameOffset;
    CHAR ImageName[256];
} RTL_PROCESS_MODULE_INFORMATION, *PRTL_PROCESS_MODULE_INFORMATION;

typedef struct _RTL_PROCESS_MODULES
{
    ULONG ModuleCount;
    RTL_PROCESS_MODULE_INFORMATION ModuleEntry[1];
} RTL_PROCESS_MODULES, *PRTL_PROCESS_MODULES;

typedef struct _RTL_PROCESS_HEAP_INFORMATION
{
    PVOID Base;
    ULONG Flags;
    USHORT Granularity;
    USHORT Unknown;
    ULONG Allocated;
    ULONG Committed;
    ULONG TagCount;
    ULONG BlockCount;
    ULONG Reserved[7];
    PVOID Tags;
    PVOID Blocks;
} RTL_PROCESS_HEAP_INFORMATION, *PRTL_PROCESS_HEAP_INFORMATION;

typedef struct _RTL_PROCESS_HEAPS
{
    ULONG HeapCount;
    RTL_PROCESS_HEAP_INFORMATION HeapEntry[1];
} RTL_PROCESS_HEAPS, *PRTL_PROCESS_HEAPS;

typedef struct _RTL_PROCESS_LOCK_INFORMATION
{
    PVOID Address;
    USHORT Type;
    USHORT CreatorBackTraceIndex;
    ULONG OwnerThreadId;
    ULONG ActiveCount;
    ULONG ContentionCount;
    ULONG EntryCount;
    ULONG RecursionCount;
    ULONG NumberOfSharedWaiters;
    ULONG NumberOfExclusiveWaiters;
} RTL_PROCESS_LOCK_INFORMATION, *PRTL_PROCESS_LOCK_INFORMATION;

typedef struct _RTL_PROCESS_LOCKS
{
    ULONG LockCount;
    RTL_PROCESS_LOCK_INFORMATION LockEntry[1];
} RTL_PROCESS_LOCKS, *PRTL_PROCESS_LOCKS;

typedef struct _RTL_PROCESS_BACKTRACE_INFORMATION
{
    PVOID SymbolicBackTrace;
    ULONG TraceCount;
    USHORT Index;
    USHORT Depth;
    PVOID BackTrace[16];
} RTL_PROCESS_BACKTRACE_INFORMATION, *PRTL_PROCESS_BACKTRACE_INFORMATION;

typedef struct _RTL_PROCESS_BACKTRACES
{
    ULONG CommittedMemory;
    ULONG ReservedMemory;
    ULONG NumberOfBackTraceLookups;
    ULONG NumberOfBackTraces;
    RTL_PROCESS_BACKTRACE_INFORMATION BackTraces[1];
} RTL_PROCESS_BACKTRACES, *PRTL_PROCESS_BACKTRACES;

typedef struct _RTL_DEBUG_BUFFER
{
    HANDLE SectionHandle;
    PVOID SectionBase;
    PVOID RemoteSectionBase;
    ULONG SectionBaseDelta;
    HANDLE EventPairHandle;
    ULONG Unknown[2];
    HANDLE RemoteThreadHandle;
    ULONG InfoClassMask;
    ULONG SizeOfInfo;
    ULONG AllocatedSize;
    ULONG SectionSize;
    PRTL_PROCESS_MODULES ModuleInformation;
    PRTL_PROCESS_BACKTRACES BackTraceInformation;
    PRTL_PROCESS_HEAPS HeapInformation;
    PRTL_PROCESS_LOCKS LockInformation;
    PVOID Reserved[8];
} RTL_DEBUG_BUFFER, *PRTL_DEBUG_BUFFER;

typedef PRTL_DEBUG_BUFFER (__stdcall *RTLCREATEQUERYDEBUGBUFFER)(IN ULONG Size,
                          IN BOOLEAN EventPair);

typedef DWORD (__stdcall *RTLDESTROYQUERYDEBUGBUFFER)(IN PRTL_DEBUG_BUFFER Buf);

typedef DWORD (__stdcall *RTLQUERYPROCESSDEBUGINFORMATION)(IN ULONG ProcessId,
                            IN ULONG DebugInfoMask,
                            IN OUT PRTL_DEBUG_BUFFER Buf);

bool Init();

----------------------------------FreeLibraryEx.cpp----------------------------------

// FreeLibraryEx.cpp

// 需要用到 SmartHandle.h

#include "FreeLibraryEx.h"
#include "SmartHandle.h"

RTLCREATEQUERYDEBUGBUFFER RtlCreateQueryDebugBuffer;
RTLDESTROYQUERYDEBUGBUFFER RtlDestroyQueryDebugBuffer;
RTLQUERYPROCESSDEBUGINFORMATION RtlQueryProcessDebugInformation;

bool Init()
{
    HMODULE hNtdll = GetModuleHandleA("ntdll.dll");
    RtlCreateQueryDebugBuffer = (RTLCREATEQUERYDEBUGBUFFER)GetProcAddress(hNtdll, "RtlCreateQueryDebugBuffer");
    RtlDestroyQueryDebugBuffer = (RTLDESTROYQUERYDEBUGBUFFER)GetProcAddress(hNtdll, "RtlDestroyQueryDebugBuffer");
    RtlQueryProcessDebugInformation = (RTLQUERYPROCESSDEBUGINFORMATION)GetProcAddress(hNtdll, "RtlQueryProcessDebugInformation");

    if (!(RtlCreateQueryDebugBuffer && RtlDestroyQueryDebugBuffer && RtlQueryProcessDebugInformation))
    {
        return false;
    }
    return true;
}

template <typename T, class PointedBy>
struct CCloseQueryDebugBuffer
{
    void Close(T handle)
    {
        RtlDestroyQueryDebugBuffer(handle);
    }
protected:
    ~CCloseQueryDebugBuffer()
    {
    }
};

typedef CSmartHandle<PRTL_DEBUG_BUFFER, CCloseQueryDebugBuffer>   CAutoCloseQueryDebugBuffer;

// 考虑到本函数不应该涉及提权之类就没有省略进程的 HANDLE

BOOL FreeLibraryEx( HANDLE hProcess,
                  DWORD dwPid,
                  HMODULE hModule )
{
    if (!Init())
    {
        return FALSE;
    }
    CAutoCloseQueryDebugBuffer HeapDebug = RtlCreateQueryDebugBuffer(0, FALSE);
    if (NULL == HeapDebug)
    {
        return FALSE;
    }
    //DWORD dwPid = GetProcessId(hProcess); //GetProcessId 只支持 VISTA 和 XP SP1
    if (!dwPid)
    {
        //RtlDestroyQueryDebugBuffer(HeapDebug);
        return FALSE;
    }
    DWORD status = RtlQueryProcessDebugInformation(dwPid,
        RTL_DEBUG_QUERY_MODULES,
        HeapDebug);

    if (status)
    {
        //RtlDestroyQueryDebugBuffer(HeapDebug);
        return FALSE;
    }

    PRTL_PROCESS_MODULES pm = HeapDebug->ModuleInformation;
    ULONG uTotal = pm->ModuleCount;
    PRTL_PROCESS_MODULE_INFORMATION pp = &(pm->ModuleEntry[0]);

    for (ULONG i=0; i<uTotal; ++i, ++pp)
    {
        if (hModule == pp->Base)
        {
            break;
        }
    }

    if (pp->LoadCount == 0xFFFF)    // exe module
    {
        return TRUE;
    }

    USHORT usTimes = 0;
    while (usTimes++ < pp->LoadCount)
    {
        DWORD dwId;
        CAutoGeneralHandle hRemote = CreateRemoteThread(hProcess
            , 0
            , 0
            , (LPTHREAD_START_ROUTINE)FreeLibrary
            , (LPVOID)hModule
            , 0
            , &dwId);
    }

    //RtlDestroyQueryDebugBuffer(HeapDebug);
    return TRUE;
}
```

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `FreeLibraryEx.rar`

强制解除映射模块就是 `NtUnmapViewOfSection` 嘛，不过效果不理想。一楼的代码的原理是得到 `DLL` 被 `LoadLibrary` 的次数，然后调用相同次数的 `FreeLibrary`(这里用的远线程实现跨进程卸载)

网上找了一段强制解除映射的：

```c
typedef ULONG (WINAPI *PFNNtUnmapViewOfSection)( IN HANDLE ProcessHandle,IN PVOID BaseAddress );

BOOL UnmapViewOfModule ( DWORD dwProcessId, LPVOID lpBaseAddr )
...{
```



```
HMODULE hModule = GetModuleHandle ( L"ntdll.dll" ) ;  

if ( hModule == NULL )  

    hModule = LoadLibrary ( L"ntdll.dll" ) ;  

PFNNtUnmapViewOfSection pfnNtUnmapViewOfSection = (PFNNtUnmapViewOfSection)GetProcAddress ( hModule, "NtUnmapViewOfSection" ) ;  

HANDLE hProcess = OpenProcess ( PROCESS_ALL_ACCESS, TRUE, dwProcessId ) ;  

ULONG ret = pfnNtUnmapViewOfSection ( hProcess, lpBaseAddr ) ;  

CloseHandle ( hProcess ) ;  

return ret ? false : true ;  

}
```
