
# 贴一个我的第三轮的 3 环方法

> 原文：[`www.pediy.com/kssd/pediy10/75891.html`](https://www.pediy.com/kssd/pediy10/75891.html)

代码:



```
#include <windows.h>
#include <tlhelp32.h>
#include <shlwapi.h>
#pragma comment (lib, "shlwapi.lib")
#pragma comment (lib, "ntdll.lib")
#pragma comment (linker, "/subsystem:windows")
#pragma comment (linker, "/entry:start")
#pragma comment (linker, "/filealign:0x200")


typedef struct \_IO\_STATUS\_BLOCK {
    union {
        int Status;
        PVOID Pointer;
    };
    ULONG Information;
} IO\_STATUS\_BLOCK, \*PIO\_STATUS\_BLOCK;

extern "C" __stdcall ZwQuerySystemInformation(
  IN int SystemInformationClass,
  IN OUT PVOID SystemInformation,
  IN ULONG SystemInformationLength,
  OUT PULONG ReturnLength  
  ); 
extern "C" __stdcall ZwQueryInformationFile(
  IN HANDLE FileHandle,
  OUT PIO\_STATUS\_BLOCK IoStatusBlock,
  OUT PVOID FileInformation,
  IN ULONG FileInformationLength,
  IN int FileInformationClass
  );

extern "C" __stdcall ZwQueryInformationThread(
  IN HANDLE ThreadHandle,
  IN int ThreadInformationClass,
  OUT PVOID ThreadInformation,
  IN ULONG ThreadInformationLength,
  OUT PULONG ReturnLength OPTIONAL
  );

void s0()
{
  HMODULE  mod;
  CreateMutex(NULL, FALSE, "HBInjectMutex");
  mod = GetModuleHandle("HBQQXX.dll");
  if (mod != NULL)
  {
    GetProcAddress(mod, "StopServiceEx");
    __asm call eax
    FreeLibrary(mod);
  }
}

int s1()
{
  HANDLE  hFile;
  DWORD  Input;
  DWORD  Output;
  DWORD  tmp;
  hFile = CreateFile("\\\\\\\\.\\\\slHBkernel32", GENERIC\_READ|GENERIC\_WRITE, 0, NULL, OPEN\_EXISTING, 0, NULL);
  if (hFile == INVALID\_HANDLE\_VALUE)
  {
    return 0;
  }
  Input = 0;
  Output = 0;
  DeviceIoControl(hFile, 0x22E00F, &Input, 4, NULL, 0, &tmp, NULL);
  DeviceIoControl(hFile, 0x22E00B, &Input, 4, &Output, 4, &tmp, NULL);
  CloseHandle(hFile);
  return 1;
}

int s2()
{
  //提权
  HANDLE hToken;
    TOKEN\_PRIVILEGES priv = {1, {0, 0, SE\_PRIVILEGE\_ENABLED}};
    if (!LookupPrivilegeValue(NULL, "SeDebugPrivilege", &priv.Privileges\[0\].Luid))
  {
    return -1;
  }
    if (!OpenProcessToken(GetCurrentProcess(), TOKEN\_ADJUST\_PRIVILEGES|TOKEN\_QUERY, &hToken))
  {
    return -2;
  }
  if (!AdjustTokenPrivileges(hToken, FALSE, &priv, sizeof(priv), 0, 0))
  {
    return -3;
  }
    CloseHandle(hToken);
  
  //找 system
  HANDLE  hC;
  DWORD  dwPid;
  BOOL  bNext;
  dwPid = 0;
  PROCESSENTRY32 p32 = {sizeof(p32)};
  hC = CreateToolhelp32Snapshot(TH32CS\_SNAPPROCESS, NULL); 
  bNext = Process32First(hC, &p32); 
  while (bNext) 
  { 
    if (lstrcmpi(p32.szExeFile, "SYSTEM") == 0)
    {
      dwPid = p32.th32ProcessID;
      break ;
    }
    bNext = Process32Next(hC, &p32); 
  } 
  CloseHandle(hC); 
  if (dwPid == 0)
  {
    return -4;
  }
  
  //找到 HB 驱动的基址和大小
  DWORD  HBBase;
  DWORD  HBSize;
  char  \*offset;
  int    Status; 
  LPBYTE  buf;
  DWORD  dwSize;
  DWORD  i;
  buf = NULL;
  dwSize = 0x2000;
  do
  {
    dwSize \*= 2;
    if (buf)
    {
      VirtualFree(buf, 0, MEM\_RELEASE);
    }
    buf = (LPBYTE)VirtualAlloc(NULL, dwSize, MEM\_COMMIT, PAGE\_READWRITE);
    Status = ZwQuerySystemInformation(11, buf, dwSize, NULL); 
  } while (Status == 0xC0000004);
  HBBase = 0;
  HBSize = 0;
  for (i=0; i<\*(LPDWORD)buf; i++)
  {
    offset = strrchr((char \*)buf+4+i\*0x11C+0x1C, '\\\\');
    if (offset != NULL)
      offset = offset + 1;
    else
      offset = (char \*)buf+4+i\*0x11C+0x1C;
    if (offset != NULL)
    {
      if (lstrcmpi(offset, "HBKernel32.sys") == 0) 
      {
        HBBase = \*(PULONG)(buf + 4 + i\*0x11C + 0x08);
        HBSize = \*(PULONG)(buf + 4 + i\*0x11C + 0x0C);
        break ;
      }
    }
  }
  VirtualFree(buf, 0, MEM\_RELEASE);
  if (HBBase == 0)
  {
    return -5;
  }
  
  //打开 system
  HANDLE  hProcess;
  hProcess = OpenProcess(PROCESS\_DUP\_HANDLE, FALSE, dwPid);
  if (hProcess == NULL)
  {
    return -6;
  }
  
  //找 HB 线程 ID
  DWORD  StartAddress;
  DWORD  dwTid;
  buf = NULL;
  dwSize = 0x2000;
  do
  {
    dwSize \*= 2;
    if (buf)
    {
      VirtualFree(buf, 0, MEM\_RELEASE);
    }
    buf = (LPBYTE)VirtualAlloc(NULL, dwSize, MEM\_COMMIT, PAGE\_READWRITE);
    Status = ZwQuerySystemInformation(5, buf, dwSize, NULL); 
  } while (Status == 0xC0000004);
  dwTid = 0;
  offset = (char \*)buf;
  while (1)
  {
    if (\*(LPDWORD)(offset+0x44) != dwPid)
    {
      offset += \*(LPDWORD)offset;
      continue ;
    }
    for (i=0; i<\*(LPDWORD)(offset+0x04); i++)
    {
      StartAddress = \*(LPDWORD)(offset+0xB8+i\*0x40+0x1C);
      if (StartAddress>HBBase && StartAddress<(HBBase+HBSize))
      {
        dwTid = \*(LPDWORD)(offset+0xB8+i\*0x40+0x24);
        break ;
      }
    }
    break ;
  }
  VirtualFree(buf, 0, MEM\_RELEASE);
  if (dwTid == 0)
  {
    CloseHandle(hProcess);
    return -7;
  }
  
  HANDLE  hThread;
  DWORD  OpenThread;
  hThread = NULL;
  OpenThread = (DWORD)GetProcAddress(GetModuleHandle("kernel32.dll"), "OpenThread");
  __asm
  {
    push dwTid
    push 0
    push THREAD\_ALL\_ACCESS
    call OpenThread
    mov hThread, eax
  }
  if (hThread == NULL)
  {
    CloseHandle(hProcess);
    return -8;
  }
  SuspendThread(hThread);
  CloseHandle(hThread);
  
  //枚举句柄
  HANDLE  hHandle;
  HANDLE  hFile;
  IO\_STATUS\_BLOCK io;
  LPBYTE  FileName;
  wchar\_t \*wname;
  buf = NULL;
  dwSize = 0x2000;
  do
  {
    dwSize \*= 2;
    if (buf)
    {
      VirtualFree(buf, 0, MEM\_RELEASE);
    }
    buf = (LPBYTE)VirtualAlloc(NULL, dwSize, MEM\_COMMIT, PAGE\_READWRITE);
    Status = ZwQuerySystemInformation(16, buf, dwSize, NULL); 
  } while (Status == 0xC0000004);
  
  FileName = (LPBYTE)VirtualAlloc(NULL, 0x1000, MEM\_COMMIT, PAGE\_READWRITE);
  for (i=0; i<\*(LPDWORD)buf; i++)
  {
    if (\*(LPDWORD)(buf+4+i\*0x10+0x00) != dwPid)
      continue ;
    hHandle = 0;
    hHandle = (HANDLE)\*(LPWORD)(buf+4+i\*0x10+0x06);
    if (\*(LPBYTE)(buf+4+i\*0x10+0x04) == 0x1C)
    {
      hFile = INVALID\_HANDLE\_VALUE;
      DuplicateHandle(hProcess, hHandle, GetCurrentProcess(), &hFile, 0, FALSE, DUPLICATE\_SAME\_ACCESS);
      if (hFile == INVALID\_HANDLE\_VALUE)
        continue ;
      memset(FileName, 0, 0x1000);
      Status = ZwQueryInformationFile(hFile, &io, FileName, 0x1000, 9);
      if (Status == 0)
      {
        wname = wcsrchr((wchar\_t \*)(FileName+4), L'\\\\');
        if (wname != NULL)
        {
          wname++;
          if (wcsicmp(wname, L"HBkernel32.sys") == 0)
          {
            CloseHandle(hFile);
            hFile = INVALID\_HANDLE\_VALUE;
            DuplicateHandle(hProcess, hHandle, GetCurrentProcess(), &hFile, 0, FALSE, DUPLICATE\_SAME\_ACCESS | DUPLICATE\_CLOSE\_SOURCE);
            if (hFile != INVALID\_HANDLE\_VALUE)
            {
              CloseHandle(hFile);
              VirtualFree(FileName, 0, MEM\_RELEASE);
              VirtualFree(buf, 0, MEM\_RELEASE);
              CloseHandle(hProcess);
              return 1;
            }
          }
        }
      }
      CloseHandle(hFile);
      hFile = INVALID\_HANDLE\_VALUE;
    }
  }
  
  VirtualFree(FileName, 0, MEM\_RELEASE);
  VirtualFree(buf, 0, MEM\_RELEASE);
  CloseHandle(hProcess);
  
  return 0;
}

int s3()
{
  HWND  hWnd;
  hWnd = FindWindow(NULL, "HBInject32");
  if (hWnd == NULL)
  {
    return 0;
  }
  SendMessage(hWnd, WM\_CLOSE, 0, 0);
  SendMessage(hWnd, WM\_QUERYENDSESSION, 0, 0);
  return 1;
}

int s4()
{
  SHDeleteValue(HKEY\_LOCAL\_MACHINE, "Software\\\\Microsoft\\\\Windows NT\\\\CurrentVersion\\\\Windows", "AppInit\_Dlls");
  SHDeleteValue(HKEY\_LOCAL\_MACHINE, "Software\\\\Microsoft\\\\Windows\\\\CurrentVersion\\\\Run", "HBService32");
  SHDeleteKey(HKEY\_LOCAL\_MACHINE, "SYSTEM\\\\ControlSet\\\\Services\\\\HBKernel32");
  SHDeleteKey(HKEY\_LOCAL\_MACHINE, "SYSTEM\\\\ControlSet001\\\\Services\\\\HBKernel32");
  SHDeleteKey(HKEY\_LOCAL\_MACHINE, "SYSTEM\\\\ControlSet002\\\\Services\\\\HBKernel32");
  SHDeleteKey(HKEY\_LOCAL\_MACHINE, "SYSTEM\\\\ControlSet003\\\\Services\\\\HBKernel32");
  SHDeleteKey(HKEY\_LOCAL\_MACHINE, "SYSTEM\\\\CurrentControlSet\\\\Services\\\\HBKernel32");
  return 1;
}

int s5()
{
  char src\[MAX\_PATH\];
  char dst\[MAX\_PATH\];
  GetTempPath(MAX\_PATH, dst);
  lstrcat(dst, "\\\\2132378.sh");
  DeleteFile(dst);
  GetSystemDirectory(src, MAX\_PATH);
  lstrcat(src, "\\\\system.exe");
  MoveFile(src, dst);
  DeleteFile(dst);
  
  GetTempPath(MAX\_PATH, dst);
  lstrcat(dst, "\\\\9345834.sh");
  DeleteFile(dst);
  GetSystemDirectory(src, MAX\_PATH);
  lstrcat(src, "\\\\HBQQXX.dll");
  MoveFile(src, dst);
  DeleteFile(dst);
  
  GetTempPath(MAX\_PATH, dst);
  lstrcat(dst, "\\\\5475451.sh");
  DeleteFile(dst);
  GetSystemDirectory(src, MAX\_PATH);
  lstrcat(src, "\\\\drivers\\\\HBKernel32.sys");
  MoveFile(src, dst);
  DeleteFile(dst);
  
  return 1;
}

void start()
{
  s0();
  if (s1() == 0)
  {
    MessageBox(0, "HBkernel32 可能不存在", "killhb", 0);
    goto home;
  }
  if (s2() < 0)
  {
    MessageBox(0, "fuck hb 驱动失败", "killhb", 0);
    goto home;
  }
  if (s3() == 0)
  {
    MessageBox(0, "清理 system.exe 失败", "killhb", 0);
    goto home;
  }
  if (s4() == 0)
  {
    MessageBox(0, "清理注册表失败", "killhb", 0);
    goto home;
  }
  if (s5() == 0)
  {
    MessageBox(0, "清理尸体失败", "killhb", 0);
    goto home;
  }
  MessageBox(0, "完成", "killhb", 0);
  
home:
  ExitProcess(0);
}
```



## 上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | | 4.rar [解压密码：pediy] |
