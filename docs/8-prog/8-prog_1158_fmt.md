
# 枚举消息钩子

> 原文：[`www.pediy.com/kssd/pediy10/98286.html`](https://www.pediy.com/kssd/pediy10/98286.html)

把炉子大侠的代码翻译了下，代码写的很乱，没整理，很多没必要的也没去掉，凑活看。

没有原创，如有雷同，纯属巧合。主要参考如下两篇：

`http://bbs.pediy.com/showthread.php?t=55453`

`http://www.xfocus.net/articles/200408/721.html`

其他的记不清了，再此一并谢过。

```c
#include <stdio.h>
#include <conio.h>
#include <Windows.h>

typedef enum _HOOK_TYPE{
  R_WH_MSGFILTER = -1,
  R_WH_JOURNALRECORD = 0,
  R_WH_JOURNALPLAYBACK = 1,
  R_WH_KEYBOARD = 2,
  R_WH_GETMESSAGE = 3,
  R_WH_CALLWNDPROC = 4,
  R_WH_CBT = 5,
  R_WH_SYSMSGFILTER = 6,
  R_WH_MOUSE = 7,
  R_WH_HARDWARE = 8,
  R_WH_DEBUG = 9,
  R_WH_SHELL = 10,
  R_WH_FOREGROUNDIDLE = 11,
  R_WH_CALLWNDPROCRET = 12,
  R_WH_KEYBOARD_LL = 13,
  R_WH_MOUSE_LL = 14
}HOOK_TYPE;

typedef enum _HANDLE_TYPE
{
  TYPE_FREE = 0,
  TYPE_WINDOW = 1,
  TYPE_MENU = 2,
  TYPE_CURSOR = 3,
  TYPE_SETWINDOWPOS = 4,
  TYPE_HOOK = 5,
  TYPE_CLIPDATA = 6,
  TYPE_CALLPROC = 7,
  TYPE_ACCELTABLE = 8,
  TYPE_DDEACCESS = 9,
  TYPE_DDECONV = 10,
  TYPE_DDEXACT = 11,
  TYPE_MONITOR = 12,
  TYPE_KBDLAYOUT = 13,
  TYPE_KBDFILE = 14,
  TYPE_WINEVENTHOOK = 15,
  TYPE_TIMER = 16,
  TYPE_INPUTCONTEXT = 17,
  TYPE_CTYPES = 18,
  TYPE_GENERIC = 255
}HANDLE_TYPE;

typedef struct _HANDLEENTRY{
  PVOID  phead;
  ULONG  pOwner;
  BYTE  bType;
  BYTE  bFlags;
  WORD  wUniq;
}HANDLEENTRY,*PHE;

typedef struct _SERVERINFO{
  WORD wRIPFlags ;
  WORD wSRVIFlags ;
  WORD wRIPPID ;
  WORD wRIPError ;
  ULONG cHandleEntries;
}SERVERINFO,*PSERVERINFO;

typedef struct _SHAREDINFO{
  PSERVERINFO psi;
  PHE aheList;
  ULONG pDispInfo;
  ULONG ulSharedDelta;
}SHAREDINFO,*PSHAREDINFO;

typedef struct _HEAD
{
  HANDLE h;
  ULONG cLockObj;
}HEAD;

typedef struct _THROBJHEAD
{
  HEAD headinfo;
  PVOID pti;
}THROBJHEAD;

typedef  struct _DESKHEAD
{
  PVOID rpdesk;
  PBYTE pSelf ;
}DESKHEAD;

typedef struct _THRDESKHEAD
{
  THROBJHEAD ThreadObjHead ;
  DESKHEAD DesktopHead ;
}THRDESKHEAD;

typedef  struct _HOOK
{
  THRDESKHEAD tshead ;
  HOOK * phkNext ;
  HOOK_TYPE iHook;
  ULONG offPfn;
  UINT flags ;
  DWORD  ihmod ;
  PVOID ptiHooked;
  PVOID rpdesk ;
}HOOK,*PHOOK;

#define IN
#define OUT
#define OPTIONAL
#define NT_SUCCESS(Status) (((NTSTATUS)(Status)) >= 0)

typedef enum _SYSDBG_COMMAND {
  SysDbgQueryModuleInformation,
  SysDbgQueryTraceInformation,
  SysDbgSetTracepoint,
  SysDbgSetSpecialCall,
  SysDbgClearSpecialCalls,
  SysDbgQuerySpecialCalls,
  SysDbgCopyMemoryChunks_0 = 8
} SYSDBG_COMMAND, *PSYSDBG_COMMAND;

typedef struct _MEMORY_CHUNKS {
  ULONG Address;
  PVOID Data;
  ULONG Length;
}MEMORY_CHUNKS, *PMEMORY_CHUNKS;

typedef NTSTATUS (_stdcall *NTSYSTEMDEBUGCONTROL)(
            IN SYSDBG_COMMAND Command,
            IN PVOID InputBuffer,
            IN ULONG InputBufferLength,
            OUT PVOID OutputBuffer,
            IN ULONG OutputBufferLength,
            OUT PULONG ReturnLength
            );

NTSYSTEMDEBUGCONTROL NtSystemDebugControl;

typedef NTSTATUS (_stdcall *NTREADVIRTUALMEMORY)(IN HANDLE ProcessHandle, IN PVOID BaseAddress, OUT PVOID Buffer, IN ULONG NumberOfBytesToRead, OUT PULONG NumberOfBytesRead OPTIONAL);

NTREADVIRTUALMEMORY NtReadVirtualMemory;

int _tmain(int argc, _TCHAR* argv[])
{
  NTSTATUS status;
  SHAREDINFO SharedInfo;
  PSHAREDINFO pSharedInfo;
  DWORD NumberOfBytesRead;
  SERVERINFO ServerInfo;
  PHE pHandleEntry;
  PHE pHandleEntryR;
  PHOOK pHook;
  MEMORY_CHUNKS mc;
  ULONG uWowHandler =(ULONG)GetProcAddress(LoadLibraryW(L"user32.dll"), "UserRegisterWowHandlers");
  for (ULONG uAddr=uWowHandler; uAddr<=uWowHandler+0x250; uAddr++)
  {
    if (0x40c7 ==*(WORD*)uAddr && 0xb8 ==*(BYTE*)(uAddr+7))
    {
      pSharedInfo=(PSHAREDINFO)(*(DWORD*)(uAddr+8));
      printf("%x\\n", pSharedInfo);
      break;
    }
  }

  //
  NtSystemDebugControl =(NTSYSTEMDEBUGCONTROL)GetProcAddress(GetModuleHandleW(L"ntdll.dll"), "NtSystemDebugControl");

  //
  NtReadVirtualMemory =(NTREADVIRTUALMEMORY)GetProcAddress(GetModuleHandleW(L"ntdll.dll"), "NtReadVirtualMemory");

  status =NtReadVirtualMemory(GetCurrentProcess(),pSharedInfo, &SharedInfo, sizeof(SHAREDINFO), &NumberOfBytesRead);

  if (!NT_SUCCESS(status))
  {
    return 1;
  }

  status =NtReadVirtualMemory(GetCurrentProcess(), SharedInfo.psi, &ServerInfo, sizeof(SERVERINFO), &NumberOfBytesRead);

  if (!NT_SUCCESS(status))
  {
    return 1;
  }

    pHandleEntryR =(PHE)VirtualAlloc(NULL, sizeof(HANDLEENTRY) *ServerInfo.cHandleEntries, MEM_RESERVE|MEM_COMMIT, PAGE_READWRITE);
  pHandleEntry =pHandleEntryR;

  NtReadVirtualMemory(GetCurrentProcess(),SharedInfo.aheList, pHandleEntry, sizeof(HANDLEENTRY) *ServerInfo.cHandleEntries, &NumberOfBytesRead);

  if (!NT_SUCCESS(status))
  {
    return 1;
  }

  pHook =(PHOOK)VirtualAlloc(NULL, sizeof(HOOK), MEM_RESERVE|MEM_COMMIT, PAGE_READWRITE);

  for ( ; pHandleEntry< pHandleEntryR +ServerInfo.cHandleEntries; pHandleEntry++)
  {
    if (TYPE_HOOK ==pHandleEntry->bType)
    {
      mc.Address =(ULONG)pHandleEntry->phead;
      mc.Length =sizeof(HOOK);
      mc.Data =pHook;
      status =NtSystemDebugControl(SysDbgCopyMemoryChunks_0, &mc, sizeof(mc), NULL, 0, &NumberOfBytesRead);

      if (!NT_SUCCESS(status))
      {
        VirtualFree(pHook, sizeof(HOOK), MEM_DECOMMIT);
        VirtualFree(pHandleEntryR, sizeof(HANDLEENTRY) *ServerInfo.cHandleEntries, MEM_DECOMMIT);
        return 1;
      }

      switch(pHook->iHook)
      {
      case   R_WH_MSGFILTER: printf("Handle:%x; Type:WH_MSGFILTER, FunAddr:%x\\n", pHook->tshead.ThreadObjHead.headinfo.h, pHook->offPfn);break;
      case   R_WH_JOURNALRECORD: printf("Handle:%x; Type:WH_JOURNALRECORD, FunAddr:%x\\n", pHook->tshead.ThreadObjHead.headinfo.h, pHook->offPfn);break;
      case   R_WH_JOURNALPLAYBACK: printf("Handle:%x; Type:WH_JOURNALPLAYBACK, FunAddr:%x\\n", pHook->tshead.ThreadObjHead.headinfo.h, pHook->offPfn);break;
      case   R_WH_KEYBOARD: printf("Handle:%x; Type:WH_KEYBOARD, FunAddr:%x\\n", pHook->tshead.ThreadObjHead.headinfo.h, pHook->offPfn);break;
      case   R_WH_GETMESSAGE: printf("Handle:%x; Type:WH_GETMESSAGE, FunAddr:%x\\n", pHook->tshead.ThreadObjHead.headinfo.h, pHook->offPfn);break;
      case   R_WH_CALLWNDPROC: printf("Handle:%x; Type:WH_CALLWNDPROC, FunAddr:%x\\n", pHook->tshead.ThreadObjHead.headinfo.h, pHook->offPfn);break;
      case   R_WH_CBT: printf("Handle:%x; Type:WH_CBT, FunAddr:%x\\n", pHook->tshead.ThreadObjHead.headinfo.h, pHook->offPfn);break;
      case   R_WH_SYSMSGFILTER: printf("Handle:%x; Type:WH_SYSMSGFILTER, FunAddr:%x\\n", pHook->tshead.ThreadObjHead.headinfo.h, pHook->offPfn);break;
      case   R_WH_MOUSE: printf("Handle:%x; Type:WH_MOUSE, FunAddr:%x\\n", pHook->tshead.ThreadObjHead.headinfo.h, pHook->offPfn);break;
      case   R_WH_HARDWARE: printf("Handle:%x; Type:WH_HARDWARE, FunAddr:%x\\n", pHook->tshead.ThreadObjHead.headinfo.h, pHook->offPfn);break;
      case   R_WH_DEBUG: printf("Handle:%x; Type:WH_DEBUG, FunAddr:%x\\n", pHook->tshead.ThreadObjHead.headinfo.h, pHook->offPfn);break;
      case   R_WH_SHELL: printf("Handle:%x; Type:WH_SHELL, FunAddr:%x\\n", pHook->tshead.ThreadObjHead.headinfo.h, pHook->offPfn);break;
```



```
case   R\_WH\_FOREGROUNDIDLE: printf("Handle:%x; Type:WH\_FOREGROUNDIDLE, FunAddr:%x\\n", `pHook->tshead.ThreadObjHead.headinfo.h`, `pHook->offPfn`);break;

case   R\_WH\_CALLWNDPROCRET: printf("Handle:%x; Type:WH\_KEYBOARD\_LL, FunAddr:%x\\n", `pHook->tshead.ThreadObjHead.headinfo.h`, `pHook->offPfn`);break;

case   R\_WH\_KEYBOARD\_LL: printf("Handle:%x; Type:WH\_MSGFILTER, FunAddr:%x\\n", `pHook->tshead.ThreadObjHead.headinfo.h`, `pHook->offPfn`);break;

case   R\_WH\_MOUSE\_LL: printf("Handle:%x; Type:WH\_MOUSE\_LL, FunAddr:%x\\n", `pHook->tshead.ThreadObjHead.headinfo.h`, `pHook->offPfn`);break;

default: printf("Unknown\\n");

}

}//if

}

//////////////////////////////////////////////////////////////////////////

`VirtualFree`(`pHook`, `sizeof(HOOK)`, `MEM\_DECOMMIT`);

`VirtualFree`(`pHandleEntryR`, `sizeof(HANDLEENTRY)` * `ServerInfo.cHandleEntries`, `MEM\_DECOMMIT`);

`\_getch`();

return 0;

}
```
