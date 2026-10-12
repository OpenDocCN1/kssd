
# 注入 DLL 之 ANSI 版

> 原文：[`www.pediy.com/kssd/pediy10/92631.html`](https://www.pediy.com/kssd/pediy10/92631.html)

Jeffrey 在《Windows 核心编程》中的第二十二章给出了一个注入 DLL 的实例，但其中包含了异常处理、字符串安全操作函数库等技术。虽然这是好事，但会增加阅读和理解难度（也许只有我有这种感觉）。因此，我使用 C 重写了一遍。这是 ANSI 版本，看看反响如何，以后会推出 UNICODE 版本。

现在将其贴于此，以飨读者。也感谢看雪提供一个获取知识的平台。谢谢

-----------------------------------

**injec.cpp**

// 此为 ASCII 版本

```c
#include "CommFile.h"
#include "resource.h"
#include <tchar.h>
#include <TlHelp32.h>
#include <windows.h>
#include <windowsx.h>
#include <malloc.h>
//#include <StrSafe.h>
#pragma once

BOOL EnjectDLL(DWORD dwProcessId, char *szDllFileName) {
  BOOL fOK = FALSE;
  HANDLE hthSnapshot = NULL;
  HANDLE hProcess = NULL, hThread = NULL;
  hthSnapshot = CreateToolhelp32Snapshot(TH32CS_SNAPMODULE, dwProcessId);
  if (hthSnapshot == INVALID_HANDLE_VALUE)
    return fOK;

  MODULEENTRY32 me = { sizeof(me) };
  BOOL bFound = FALSE;
  BOOL bMoreMods = Module32First(hthSnapshot, &me);
  while (bMoreMods) {
    if (!lstrcmp(me.szModule, szDllFileName)) {
      bFound = TRUE;
      break;
    } else
      bMoreMods = Module32Next(hthSnapshot, &me);
  }

  if (!bFound) {
    chMB("not found the wanted-release dll");
    return FALSE;
  }

  if (bFound)
    hProcess = OpenProcess(
      PROCESS_QUERY_INFORMATION |
      PROCESS_CREATE_THREAD |
      PROCESS_VM_OPERATION, // For CreateRemoteThread
      FALSE, dwProcessId);
  if (hProcess == NULL)
    chMB("进程打不开？？？");

  PTHREAD_START_ROUTINE pfnThreadRtn = (PTHREAD_START_ROUTINE)
    GetProcAddress(GetModuleHandle(TEXT("Kernel32")), "FreeLibrary"); // FreeLibrary 没有 FreeLibraryA 的

  hThread = CreateRemoteThread(hProcess, NULL, 0,
    pfnThreadRtn, me.modBaseAddr, 0, NULL);

  // Wait for the remote thread to terminate
  WaitForSingleObject(hThread, INFINITE);

  fOK = TRUE;
  if (hthSnapshot != NULL)
    CloseHandle(hthSnapshot);
  if (hThread != NULL)
    CloseHandle(hThread);
  if (hProcess != NULL)
    CloseHandle(hProcess);

  fOK = TRUE; // Everything executed successfully
  return fOK;
}

BOOL InjectDLL(DWORD nThreadID, char *szDllFileName) {
  HANDLE hProcess = NULL, hThread = NULL;
  char *pBaseAddr;
  BOOL fOK;

  hProcess = ::OpenProcess(PROCESS_QUERY_INFORMATION |   // Required by Alpha
    PROCESS_CREATE_THREAD |   // For CreateRemoteThread
    PROCESS_VM_OPERATION |   // For VirtualAllocEx/VirtualFreeEx
    PROCESS_VM_WRITE,        // For WriteProcessMemory
    FALSE, nThreadID);

  if (hProcess == NULL) {
    chMB("该进程不存在或无法打开!");
    return FALSE;
  }

  int cb = lstrlen(szDllFileName) + 1;
  pBaseAddr = (char *)VirtualAllocEx(hProcess, NULL, cb, MEM_COMMIT, PAGE_READWRITE);

  // Copy the DLL's pathname to the remote process' address space
  WriteProcessMemory(hProcess, pBaseAddr,
    (PVOID)szDllFileName, cb, NULL);

  PTHREAD_START_ROUTINE pfnThreadRtn = (PTHREAD_START_ROUTINE)
    GetProcAddress(GetModuleHandle(TEXT("Kernel32")), "LoadLibraryA");

  hThread = CreateRemoteThread(hProcess, NULL, 0,
    pfnThreadRtn, pBaseAddr, 0, NULL);

  WaitForSingleObject(hThread, INFINITE);

  VirtualFreeEx(hProcess, pBaseAddr, 0, MEM_RELEASE);

  if (hThread != NULL)
    CloseHandle(hThread);
  if (hProcess != NULL)
    CloseHandle(hProcess);

  return (fOK = TRUE);
}

BOOL Dlg_InitDialog(HWND hwnd, HWND hwndFocus, LPARAM lParam) {
  chSETDLGICONS(hwnd, IDI_ICON);
  return (TRUE);
}

void Dlg_OnCommand(HWND hwnd, int id, HWND hwndCtl, UINT codeNotify) {
  switch (id) {
  case IDCANCEL:
    EndDialog(hwnd, id);
    break;
  case IDC_INJECT:
    char DllPath[MAX_PATH];
    GetModuleFileName(NULL, DllPath, sizeof(DllPath));
    PTSTR pFilename = _tcsrchr(DllPath, TEXT('\\')) + 1;
    lstrcpyn(pFilename, TEXT("walk.dll"), sizeof(DllPath) - (pFilename - DllPath));

    UINT nThreadID;
    nThreadID = GetDlgItemInt(hwnd, IDC_Thread, NULL, FALSE);
    if (nThreadID == 0)
      nThreadID = GetCurrentProcessId();

    if (InjectDLL(nThreadID, DllPath)) {
      chMB("DLL Injection successful.");
    }

    if (EnjectDLL(nThreadID, "walk.dll"))
      chMB("DLL Enjection is successful");
    break;
  }
}

INT_PTR WINAPI Dlg_Proc(HWND hwnd, UINT uMsg, WPARAM wParam, LPARAM lParam) {
  switch (uMsg) {
    chHANDLE_DLGMSG(hwnd, WM_INITDIALOG, Dlg_InitDialog);
    chHANDLE_DLGMSG(hwnd, WM_COMMAND, Dlg_OnCommand);
  }
  return (FALSE);
}

int WINAPI WinMain(HINSTANCE hinstExe, HINSTANCE, PTSTR pszCmdLine, int) {
  DialogBoxParam(hinstExe, MAKEINTRESOURCE(IDD_INJECTOR),
    NULL, Dlg_Proc, _ttoi(pszCmdLine));
  return (0);
}
```

-----------------------------------

至于 `.rc` 文件、`CommFile.h` 文件和 `resource.h` 可参照《Windows 核心编程》中给出的实例源代码。如果在编译时出现不识别某些函数而报错，估计您使用的是 VC6.0 的原装库，建议升级 SDK。当然还需要那个 `.dll` 文件（也在该书的源码中，我改名为 `walk.dll`，名字好记）。

高手飘过即可。
