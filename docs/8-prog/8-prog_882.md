
# 老生常谈-远程线程注入

> 原文：[`www.pediy.com/kssd/pediy10/98944.html`](https://www.pediy.com/kssd/pediy10/98944.html)

闲来无聊,总结前人技术

## 主要函数

- `CreateRemoteThread()`
- `LoadLibrary()`
- `Thread()`
- `VirtualAllocEx()`

函数原型就不写了~偷懒....

## 线程注入基本步骤

1. 提升权限通常为 `DEBUG`
2. 调用 `OpenProcess` 函数打开目标进程,返回进程句柄
3. 申请空间,写入要注入的 `DLL` 名,返回内存空间首地址
4. 调用 `GetProcAddress` 函数得到 `LoadLibrary` 函数地址
5. 调用 `CreateRemoteThread` 函数创建并启动线程

## 逐步实现

### 1. 提升权限

首先,提升权限

```c
int EnableDebugPriv(const char * name)  //提升进程为 DEBUG 权限
{
  HANDLE hToken;
  TOKEN_PRIVILEGES tp;
  LUID luid;
  if(!OpenProcessToken(GetCurrentProcess(),
    TOKEN_ADJUST_PRIVILEGES|TOKEN_QUERY,
    &hToken) )
  {
    printf("OpenProcessToken error\n");
    return 1;
  }
  if(!LookupPrivilegeValue(NULL,name,&luid))
  {
    printf("LookupPrivilege error!\n");
  }
  tp.PrivilegeCount = 1;
  tp.Privileges[0].Attributes =SE_PRIVILEGE_ENABLED;
  tp.Privileges[0].Luid = luid;
  if(!AdjustTokenPrivileges(hToken,0,&tp,sizeof(TOKEN_PRIVILEGES),NULL,NULL) )
  {
    printf("AdjustTokenPrivileges error!\n");
    return 1;
  }
  return 0;
}
```

该函数用于提升进程为 `DEBUG` 权限,函数入口为进程名

### 2. 得到进程句柄

接下来,得到进程句柄

```c
OpenProcess(PROCESS_ALL_ACCESS,FALSE,dwRemoteProcessId) //取得所有权
```

### 3. 申请内存空间并写入路径

现在就开始向系统申请所需内存空间

```c
char *pszLibFileRemote;
  pszLibFileRemote=(char *)VirtualAllocEx( hRemoteProcess, 
                  NULL, lstrlen(DllFullPath)+1, 
                  MEM_COMMIT, PAGE_READWRITE);
  if(pszLibFileRemote==NULL)
  {
    printf("VirtualAllocEx error\n");
    return FALSE;
  }
  if(WriteProcessMemory(hRemoteProcess,
    pszLibFileRemote,(void *)DllFullPath,lstrlen(DllFullPath)+1,NULL) == 0)
  {
    printf("WriteProcessMemory error\n");
    return FALSE;
  }
```

### 4. 获取 LoadLibraryA 地址

在往后就是得到 `LoadLibraryA` 函数地址

```c
  PTHREAD_START_ROUTINE pfnStartAddr=(PTHREAD_START_ROUTINE)
    GetProcAddress(GetModuleHandle(TEXT("Kernel32")),"LoadLibraryA");
  if(pfnStartAddr == NULL)
  {
    printf("GetProcAddress error\n");
    return FALSE;
  }
```

### 5. 启动远程线程

最后就是启动线程了呵呵

```c
  if( (hRemoteThread = CreateRemoteThread(hRemoteProcess,NULL,0, 
    pfnStartAddr,pszLibFileRemote,0,NULL))==NULL)
  {
    printf("CreateRemoteThread error\n");
    return FALSE;
  }
  return TRUE;
}
```

## 完整源代码

### 头文件

```c
#include <windows.h>
#include <tlhelp32.h>
#include <stdio.h>

#ifndef FUN_H
#define FUN_H

int EnableDebugPriv(const char * name)  //提升进程为 DEBUG 权限
{
  HANDLE hToken;
  TOKEN_PRIVILEGES tp;
  LUID luid;
  //打开进程令牌环
  if(!OpenProcessToken(GetCurrentProcess(),
    TOKEN_ADJUST_PRIVILEGES|TOKEN_QUERY,
    &hToken) )
  {
    printf("OpenProcessToken error\n");
    return 1;
  }
  //获得进程本地唯一 ID
  if(!LookupPrivilegeValue(NULL,name,&luid))
  {
    printf("LookupPrivilege error!\n");
  }
  tp.PrivilegeCount = 1;
  tp.Privileges[0].Attributes =SE_PRIVILEGE_ENABLED;
  tp.Privileges[0].Luid = luid;
  //调整进程权限
  if(!AdjustTokenPrivileges(hToken,0,&tp,sizeof(TOKEN_PRIVILEGES),NULL,NULL) )
  {
    printf("AdjustTokenPrivileges error!\n");
    return 1;
  }
  return 0;
}

BOOL InjectDll(const char *DllFullPath, const DWORD dwRemoteProcessId) //注入函数
{
  HANDLE hRemoteProcess;
  //获得调试权限
  if(EnableDebugPriv(SE_DEBUG_NAME))
  {
    printf("add privilege error");
    return FALSE;
  }
  //打开目标进程
  if((hRemoteProcess=OpenProcess(PROCESS_ALL_ACCESS,FALSE,dwRemoteProcessId))==NULL)
  {
    printf("OpenProcess error\n");
    return FALSE;
  }
  char *pszLibFileRemote;
    //申请存放 dll 文件名的路径
  pszLibFileRemote=(char *)VirtualAllocEx( hRemoteProcess, 
                  NULL, lstrlen(DllFullPath)+1, 
                  MEM_COMMIT, PAGE_READWRITE);
  if(pszLibFileRemote==NULL)
  {
    printf("VirtualAllocEx error\n");
    return FALSE;
  }
  //把 dll 的完整路径写入到内存，
  if(WriteProcessMemory(hRemoteProcess,
    pszLibFileRemote,(void *)DllFullPath,lstrlen(DllFullPath)+1,NULL) == 0)
  {
    printf("WriteProcessMemory error\n");
    return FALSE;
  }
  //得到 LoadLibraryA 函数地址
  PTHREAD_START_ROUTINE pfnStartAddr=(PTHREAD_START_ROUTINE)
    GetProcAddress(GetModuleHandle(TEXT("Kernel32")),"LoadLibraryA");
  if(pfnStartAddr == NULL)
  {
    printf("GetProcAddress error\n");
    return FALSE;
  }
  HANDLE hRemoteThread;
  //启动远程线程
  if( (hRemoteThread = CreateRemoteThread(hRemoteProcess,NULL,0, 
    pfnStartAddr,pszLibFileRemote,0,NULL))==NULL)
  {
    printf("CreateRemoteThread error\n");
    return FALSE;
  }
  return TRUE;
}

DWORD GetProcessID(char *ProcessName)  //获得进程 PID
{
  PROCESSENTRY32 pe32;
  pe32.dwSize=sizeof(pe32);
  //获得系统内所有进程快照
  HANDLE hProcessSnap=CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS,0);
  if(hProcessSnap==INVALID_HANDLE_VALUE)
  {
    printf("CreateToolhelp32Snapshot error");
    return 0;
  }
  //枚举列表中的第一个进程
  BOOL bProcess=Process32First(hProcessSnap,&pe32);
  while(bProcess)
  {
    //比较找到的进程名和我们要查找的进程名，一样则返回进程 id
    if(strcmp(strupr(pe32.szExeFile),strupr(ProcessName))==0)
      return pe32.th32ProcessID;
    //继续查找
    bProcess=Process32Next(hProcessSnap,&pe32);
  }
  CloseHandle(hProcessSnap);
  return 0;
}
#endif
```

### 主函数文件

```c
#include "fun.h"

int APIENTRY WinMain(HINSTANCE hInstance,
                    HINSTANCE hPrevInstance,
                    LPSTR lpCmdLine,
                    int nCmdShow)
{
  char Path[255];
  char DllPath[255];
  //得到 widnows 系统路径
    GetSystemDirectory(Path,sizeof(Path));
  //0x00 截断字符，得到盘符
  Path[3]=0x00;
  //得到 IE 带路径文件名
  strcat(Path,"Program Files\\Internet Explorer\\iexplore.exe");
  //启动 IE，为了防止系统中没有 IE 进程
    WinExec(Path,SW_HIDE);
  //暂停两秒，等待 IE 启动
  Sleep(2000);
  //得到 IE 进程
  DWORD Pid=GetProcessID("iexplore.exe");
  //得到程序自身路径
  GetCurrentDirectory(sizeof(DllPath),DllPath);
  //得到 DLL 带路径文件名
  strcat(DllPath,"\\test.dll");
  //注入 IE 进程
    InjectDll(DllPath,Pid);
  return 0;
}
```

## 补充

另外有个东西

本进程的 `LoadLibrary` 函数地址可以这样获得

```c
PTHREAD_START_ROUTINE p=(PTHREAD_START_ROUTINE)GetProcAddress(GetModuleHandle(TEXT("kernel32")),"LoadLibraryA");
```

该函数成功返回句柄,否则返回零蛋
