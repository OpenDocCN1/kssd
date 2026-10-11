
# ring3 & ring0 通用 InlineHook 代码(修补)

> 原文：[`www.pediy.com/kssd/pediy10/71480.html`](https://www.pediy.com/kssd/pediy10/71480.html)

修补加强了下，支持多核，多线程

一晚上的战绩，希望尊重版权，写一个 `ring3` 下的例子，`ring0` 下就不写了，很简单

代码：

```c
#include <stdio.h>
#include "InlineHook.h"

typedef void (*__stdcall *__Sleep)(DWORD);
__Sleep      realSleep = NULL;

VOID
__stdcall
MySleep(
  IN DWORD dwMilliseconds
  )
{
  printf("Sleep(%d) Called\n", dwMilliseconds);

  return realSleep(dwMilliseconds);
}

int main(int argc, char* argv[])
{
  InlineHook(Sleep, MySleep, &realSleep);
  Sleep(10);
  UnInlineHook(Sleep, realSleep);
  return 0;
}
```

主文件代码如下

代码：

```c
/*++

Copyright (c) 2008/08/27 By CoolDiyer

Abstract:

Ring3 all-purpose inline hook module

--*/

#if !defined(AFX_INLINEHOOK_H_INCLUDED)
#define AFX_INLINEHOOK_H_INCLUDED

#ifdef WIN32
#define RING3
#endif

#ifdef RING3
#include <windows.h>
#else
#include <windef.h>
#endif

#include "LDasm.h"

#ifdef RING3
#define __malloc(_s)  VirtualAlloc(NULL, _s, MEM_COMMIT, PAGE_EXECUTE_READWRITE)
#define __free(_p)    VirtualFree(_p, 0, MEM_RELEASE)
#define JMP_SIZE    5
#else
#define __malloc(_s)  ExAllocatePool(NonPagedPool, _s)
#define __free(_p)    ExFreePool(_p)
#define JMP_SIZE    7
#endif

#ifdef RING3

BOOL
WriteReadOnlyMemory(
  LPBYTE  lpDest,
  LPBYTE  lpSource,
  ULONG  Length
  )
/*++
  写只读内存, ring3 下就不要锁了
--*/
{
  BOOL        bRet;
  DWORD        dwOldProtect;
  bRet = FALSE;

  // 使前几个字节的内存可写
  if (!VirtualProtect(lpDest, Length, PAGE_READWRITE, &dwOldProtect))
  {
    return bRet;
  }

  memcpy(lpDest, lpSource, Length);

  bRet = VirtualProtect(lpDest, Length, dwOldProtect, &dwOldProtect);

  return  bRet;
}

#else

NTSTATUS
WriteReadOnlyMemory(
  LPBYTE  lpDest,
  LPBYTE  lpSource,
  ULONG  Length
  )
/*++
  写只读内存(源于 Mark 代码)
--*/
{
  NTSTATUS  status;
  KSPIN_LOCK  spinLock;
  KIRQL    oldIrql;
  PMDL    pMdlMemory;
  LPBYTE    lpWritableAddress;

  status = STATUS_UNSUCCESSFUL;

  pMdlMemory = IoAllocateMdl(lpDest, Length, FALSE, FALSE, NULL);

  if (NULL == pMdlMemory) return status;

  MmBuildMdlForNonPagedPool(pMdlMemory);
    MmProbeAndLockPages(pMdlMemory, KernelMode, IoWriteAccess);
  lpWritableAddress = MmMapLockedPages(pMdlMemory, KernelMode);
    if (NULL != lpWritableAddress)
  {
    oldIrql  = 0;
    KeInitializeSpinLock(&spinLock);
    KeAcquireSpinLock(&spinLock, &oldIrql);

    memcpy(lpWritableAddress, lpSource, Length);

    KeReleaseSpinLock(&spinLock, oldIrql);
    MmUnmapLockedPages(lpWritableAddress, pMdlMemory);

    status = STATUS_SUCCESS;
  }

  MmUnlockPages(pMdlMemory);
    IoFreeMdl(pMdlMemory);

  return  status;
}

#endif

BOOL 
GetPatchSize(
  IN  void *Proc,      /* 需要 Hook 的函数地址 */
  IN  DWORD dwNeedSize,  /* Hook 函数头部占用的字节大小 */
  OUT LPDWORD lpPatchSize  /* 返回根据函数头分析需要修补的大小 */
  )
/*++
  计算函数头需要 Patch 的大小
--*/
{
  DWORD  Length;
  PUCHAR  pOpcode;
  DWORD  PatchSize = 0;

  if (!Proc || !lpPatchSize)
  {
    return FALSE;
  }

  do
  {
    Length = SizeOfCode(Proc, &pOpcode);
    if ((Length == 1) && (*pOpcode == 0xC3)) break;
    if ((Length == 3) && (*pOpcode == 0xC2)) break;
    Proc = (PVOID)((DWORD)Proc + Length);

    PatchSize += Length;
    if (PatchSize >= dwNeedSize)
    {
      break;
    }

  } while (Length);

  *lpPatchSize = PatchSize;

  return  TRUE;
}

BOOL
InlineHook(
  IN  void *OrgProc,      /* 需要 Hook 的函数地址 */
  IN  void *NewProc,    /* 代替被 Hook 函数的地址 */
  OUT  void **RealProc    /* 返回原始函数的入口地址 */
  )
/*++
  对函数进行 Inline Hook
--*/
{
  DWORD  dwPatchSize;  // 得到需要 patch 的字节大小
  DWORD  dwOldProtect;   
  LPVOID  lpHookFunc;    // 分配的 Hook 函数的内存
  DWORD  dwBytesNeed;  // 分配的 Hook 函数的大小
  LPBYTE  lpPatchBuffer;  // jmp 指令的临时缓冲区

  if (!OrgProc || !NewProc || !RealProc)
  {
    return FALSE;
  }
  // 得到需要 patch 的字节大小
  if (!GetPatchSize(OrgProc, JMP_SIZE, &dwPatchSize))
  {
    return FALSE;
  }

  /*
  0x00000800         0x00000800    sizeof(DWORD)  // dwPatchSize
  JMP  / FAR 0xAABBCCDD    E9 DDCCBBAA    JMP_SIZE
  ...              ...        dwPatchSize    // Backup instruction
  JMP  / FAR 0xAABBCCDD    E9 DDCCBBAA    JMP_SIZE
  */

  dwBytesNeed = sizeof(DWORD) + JMP_SIZE + dwPatchSize + JMP_SIZE;

  lpHookFunc = __malloc(dwBytesNeed);

  // 备份 dwPatchSize 到 lpHookFunc
  *(DWORD *)lpHookFunc = dwPatchSize;

  // 跳过开头的 4 个字节
  lpHookFunc = (LPVOID)((DWORD)lpHookFunc + sizeof(DWORD));


  // 开始 backup 函数开头的字   
  memcpy((BYTE *)lpHookFunc + JMP_SIZE, OrgProc, dwPatchSize);

  lpPatchBuffer = __malloc(dwPatchSize);

  // NOP 填充
  memset(lpPatchBuffer, 0x90, dwPatchSize);

#ifdef RING3
  // jmp 到 Hook
  *(BYTE *)lpHookFunc = 0xE9;
  *(DWORD*)((DWORD)lpHookFunc + 1) = (DWORD)NewProc - (DWORD)lpHookFunc - JMP_SIZE;

  // 跳回原始
  *(BYTE*)((DWORD)lpHookFunc + 5 + dwPatchSize) = 0xE9;
  *(DWORD*)((DWORD)lpHookFunc + 5 + dwPatchSize + 1) = ((DWORD)OrgProc + dwPatchSize) - ((DWORD)lpHookFunc + JMP_SIZE + dwPatchSize) - JMP_SIZE;


  // jmp 
  *(BYTE *)lpPatchBuffer = 0xE9;
  // 注意计算长度的时候得用 OrgProc
  *(DWORD*)(lpPatchBuffer + 1) = (DWORD)lpHookFunc - (DWORD)OrgProc - JMP_SIZE;

#else
   
  // jmp 到 Hook
  *(BYTE *)lpHookFunc = 0xEA;
  *(DWORD*)((DWORD)lpHookFunc + 1) = (DWORD)NewProc;
  *(WORD*)((DWORD)lpHookFunc + 5) = 0x08;
   
  // 跳回原始
  *(BYTE*)((DWORD)lpHookFunc + JMP_SIZE + dwPatchSize) = 0xEA;
  *(DWORD*)((DWORD)lpHookFunc + JMP_SIZE + dwPatchSize + 1) = ((DWORD)OrgProc + dwPatchSize);
  *(WORD*)((DWORD)lpHookFunc + JMP_SIZE + dwPatchSize + 5) = 0x08;

  // jmp far
  *(BYTE *)lpPatchBuffer = 0xEA;
   
  // 跳到 lpHookFunc 函数
  *(DWORD*)(lpPatchBuffer + 1) = (DWORD)lpHookFunc;
  *(WORD*)(lpPatchBuffer + 5) = 0x08;
#endif

  WriteReadOnlyMemory(OrgProc, lpPatchBuffer, dwPatchSize);
   
  __free(lpPatchBuffer);


  *RealProc = (DWORD)lpHookFunc + JMP_SIZE;

  return  TRUE;
}

void UnInlineHook(
  void *OrgProc,    /* 需要恢复 Hook 的函数地址 */
  void *RealProc  /* 原始函数的入口地址 */
  )
/*++
  恢复对函数进行的 Inline Hook
--*/
{
  DWORD  dwPatchSize;
  DWORD  dwOldProtect;
  LPBYTE  lpBuffer;
   
  // 找到分配的空间
  lpBuffer = (DWORD)RealProc - (sizeof(DWORD) + JMP_SIZE);
  // 得到 dwPatchSize
  dwPatchSize = *(DWORD *)lpBuffer;

  WriteReadOnlyMemory(OrgProc, RealProc, dwPatchSize);

  // 释放分配的跳转函数的空间
  __free(lpBuffer);
}
#endif // !defined(AFX_INLINEHOOK_H_INCLUDED)
```

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | | `InlineHook.rar`
