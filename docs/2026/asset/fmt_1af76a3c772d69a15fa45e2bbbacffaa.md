
# [原创]Windows 程序分析工具

**Windows Process Analysis Tools (Windows程序分析工具)**

主要功能:查看进程,线程,模块,堆内存,窗口,挂起线程,结束线程,卸载模块,CPU使用率,内存使用率,附加pe文件解析,支持查看导入表,导出表,重定位表,资源表,TLS表,延迟加载等..

附加功能:`dll`注入(`win32`/`x64`)，无模块`Pe`加载注入，`R3` `inlineHook`实现进程隐藏，内置反汇编引擎实现代码注入

其他功能:分析文件`MD5`,一键隐藏窗口(`alt+win+z`),设置进程优先级,托盘管理,增删用户,内存清理,窗口重置,等等无聊的功能都写里面了...

学习了小半年的编程了 写出来还是有点成就感的![](https://bbs.kanxue.com/view/img/face/1.gif)   涉及的知识【`MFC`编程,`Windows`核心编程,`C++`,注入和`hook`】

![](https://bbs.kanxue.com/upload/attach/201611/735228_c39bce554062e777.png)![](https://bbs.kanxue.com/upload/attach/201611/735228_60ec6addbc3a9807.png)

![](https://bbs.kanxue.com/upload/attach/201611/735228_f5b2152bc48b889b.png)![](https://bbs.kanxue.com/upload/attach/201611/735228_328d60614747bee0.png)![](https://bbs.kanxue.com/upload/attach/201611/735228_3b8e77d40a31d3c5.png)![](https://bbs.kanxue.com/upload/attach/201611/735228_1096a85d0c5a7025.png)

========================================================

## 用到的Api

`EnumWindows`(枚举顶层窗口);`GetWindowLong`<|reserved_special_token_1517|>`GetClassLong`l`TerminateProcess`l`Thread32First`l`OpenThread`l`ResumeThread`l`UpdateWindow`l`CreateToolhelp32Snapshot`l`IsWow64Process`l`QueryDosDevice`l`GetLogicalDriveStrings`l`SuspendThread`l`SetPriorityClass`l`OpenProcess`l`VirtualAllocEx`l`WriteProcessMemory`l`CreateRemoteThread`l`AfxBeginThread`l`FindWindow`l`GetProcAddress`l....就不都列举出来了 详情看代码吧。 我就把隐藏进程,内存加载,代码注入的核心代码贴出来吧

========================================================华丽的***======================================================

## 进程隐藏

进程隐藏采用了`R3` 层`Hook ZwQuerySystemInformation` 这个函数,这个函数是任务管理器,`Proexp`,`Promon`,一类进程管理所获取进程的一个函数底层调用的是这个函数,我们要先把`DLL` 注入到任务管理器中,然后在`dll`里面实现`inlineHook` 这个`api` 改前五个字节跳到`dll`中自己的`NewZwQuerySystemInformation`的函数中,把要隐藏的进程从单链表中剔去,![](https://bbs.kanxue.com/view/img/face/15.gif)一点都不新鲜的东西咯..  详情代码请参考逆向工程核心原理,含有要提一点想隐藏进程是进程不在任务管理器显示,所以我们`hook`的是任务管理器`ZwQuerySystemInformation` 函数,因为书中的测试环境是`Win7 x32`所以他的任务管理器也是`32`位的,所以该方法对`Win7 x64` 六十四位的任务管理器无效,因为他们的函数参数入栈方式不同,要写段七个字节的内联汇编....  有兴趣可以私信交流下![](https://bbs.kanxue.com/view/img/face/1.gif)

### Demo

```
#include "windows.h"
#include "tchar.h"
#define STATUS_SUCCESS            (0x00000000L)
typedef LONG NTSTATUS;
typedef enum _SYSTEM_INFORMATION_CLASS {
  SystemBasicInformation = 0,
  SystemPerformanceInformation = 2,
  SystemTimeOfDayInformation = 3,
  SystemProcessInformation = 5,
  SystemProcessorPerformanceInformation = 8,
  SystemInterruptInformation = 23,
  SystemExceptionInformation = 33,
  SystemRegistryQuotaInformation = 37,
  SystemLookasideInformation = 45
} SYSTEM_INFORMATION_CLASS;
typedef struct _SYSTEM_PROCESS_INFORMATION {
  ULONG NextEntryOffset;
  ULONG NumberOfThreads;
  BYTE Reserved1[48];
  PVOID Reserved2[3];
  HANDLE UniqueProcessId;
  PVOID Reserved3;
  ULONG HandleCount;
  BYTE Reserved4[4];
  PVOID Reserved5[11];
  SIZE_T PeakPagefileUsage;
  SIZE_T PrivatePageCount;
  LARGE_INTEGER Reserved6[6];
} SYSTEM_PROCESS_INFORMATION, *PSYSTEM_PROCESS_INFORMATION;
typedef NTSTATUS(WINAPI *PFZWQUERYSYSTEMINFORMATION)
(SYSTEM_INFORMATION_CLASS SystemInformationClass,
PVOID SystemInformation,
ULONG SystemInformationLength,
PULONG ReturnLength);
#define DEF_NTDLL                       ("ntdll.dll")
#define DEF_ZWQUERYSYSTEMINFORMATION    ("ZwQuerySystemInformation")
// global variable (in sharing memory)
#pragma comment(linker, "/SECTION:.SHARE,RWS")
#pragma data_seg(".SHARE")
TCHAR g_szProcName[MAX_PATH] = { 0, };
#pragma data_seg()
// global variable
BYTE g_pOrgBytes[5] = { 0, };
NTSTATUS WINAPI NewZwQuerySystemInformation(
  SYSTEM_INFORMATION_CLASS SystemInformationClass,
  PVOID SystemInformation,
  ULONG SystemInformationLength,
  PULONG ReturnLength)
{
  NTSTATUS status;
  FARPROC pFunc;
  PSYSTEM_PROCESS_INFORMATION pCur, pPrev;
  char szProcName[MAX_PATH] = { 0, };
  // 先脱钩
  unhook_by_code(DEF_NTDLL, DEF_ZWQUERYSYSTEMINFORMATION, g_pOrgBytes);
  // o调用kernrl的ZwQuerySystemInformation
  pFunc = GetProcAddress(GetModuleHandleA(DEF_NTDLL),
    DEF_ZWQUERYSYSTEMINFORMATION);
  status = ((PFZWQUERYSYSTEMINFORMATION)pFunc)
    (SystemInformationClass, SystemInformation,
    SystemInformationLength, ReturnLength);
  if (status != STATUS_SUCCESS)
    goto __NTQUERYSYSTEMINFORMATION_END;
  // 对 SystemProcessInformation 操作
  if (SystemInformationClass == SystemProcessInformation)
  {
    // SYSTEM_PROCESS_INFORMATION类型强转
    // pCur 为单链表的头节点
    pCur = (PSYSTEM_PROCESS_INFORMATION)SystemInformation;
    while (TRUE)
    {
      // 比较进程名称
      // g_szProcName = 要隐藏的进程名称
      if (pCur->Reserved2[1] != NULL)
      {
        if (!_tcsicmp((PWSTR)pCur->Reserved2[1], g_szProcName))
        {
          // 删除隐藏进程的节点
          if (pCur->NextEntryOffset == 0)
            pPrev->NextEntryOffset = 0;
          else
            pPrev->NextEntryOffset += pCur->NextEntryOffset;
        }
        else
          pPrev = pCur;
      }
      if (pCur->NextEntryOffset == 0)
        break;
      //指向链表的下一项
      pCur = (PSYSTEM_PROCESS_INFORMATION)
        ((ULONG)pCur + pCur->NextEntryOffset);
    }
  }
__NTQUERYSYSTEMINFORMATION_END:
  // 二次勾取api
  hook_by_code(DEF_NTDLL, DEF_ZWQUERYSYSTEMINFORMATION,
    (PROC)NewZwQuerySystemInformation, g_pOrgBytes);
  return status;
}
BOOL hook_by_code(LPCSTR szDllName, LPCSTR szFuncName, PROC pfnNew, PBYTE pOrgBytes)
{
  FARPROC pfnOrg;
  DWORD dwOldProtect, dwAddress;
  BYTE pBuf[5] = { 0xE9, 0, };
  PBYTE pByte;
  // 获取ZwQuerySystemInformation地址
  pfnOrg = (FARPROC)GetProcAddress(GetModuleHandleA(szDllName), szFuncName);
  pByte = (PBYTE)pfnOrg;
  // 判断第一个字节是否已被修改
  if (pByte[0] == 0xE9)
    return FALSE;
  // 修改内存属性 前额五个字节为可写
  VirtualProtect((LPVOID)pfnOrg, 5, PAGE_EXECUTE_READWRITE, &dwOldProtect);
  // 保存修改之前的五个字节
  memcpy(pOrgBytes, pfnOrg, 5);
  // JMP 的地址
  dwAddress = (DWORD)pfnNew - (DWORD)pfnOrg - 5;
  memcpy(&pBuf[1], &dwAddress, 4);
  // 修改五个字节
  memcpy(pfnOrg, pBuf, 5);
  // 恢复内存属性
  VirtualProtect((LPVOID)pfnOrg, 5, dwOldProtect, &dwOldProtect);
  return TRUE;
}
BOOL unhook_by_code(LPCSTR szDllName, LPCSTR szFuncName, PBYTE pOrgBytes)
{
  FARPROC pFunc;
  DWORD dwOldProtect;
  PBYTE pByte;
  // 获取api地址
  pFunc = GetProcAddress(GetModuleHandleA(szDllName), szFuncName);
  pByte = (PBYTE)pFunc;
       //判断是否挂钩
  if (pByte[0] != 0xE9)
    return FALSE;
  // 修改内存属性
  VirtualProtect((LPVOID)pFunc, 5, PAGE_EXECUTE_READWRITE, &dwOldProtect);
  // 卸掉钩子
  memcpy(pFunc, pOrgBytes, 5);
  // 还原内存属性
  VirtualProtect((LPVOID)pFunc, 5, dwOldProtect, &dwOldProtect);
        return TRUE;
}
BOOL WINAPI DllMain(HINSTANCE hinstDLL, DWORD fdwReason, LPVOID lpvReserved)
{
  switch (fdwReason)
  {
    // #2. API Hooking
  case DLL_PROCESS_ATTACH:
    hook_by_code(DEF_NTDLL, DEF_ZWQUERYSYSTEMINFORMATION,
      (PROC)NewZwQuerySystemInformation, g_pOrgBytes);
    break;
    // #3. API Unhooking
  case DLL_PROCESS_DETACH:
    unhook_by_code(DEF_NTDLL, DEF_ZWQUERYSYSTEMINFORMATION,
      g_pOrgBytes);
    break;
  }
  return TRUE;
}
#ifdef __cplusplus
extern "C" {
#endif
  __declspec(dllexport) void SetProcName(LPCTSTR szProcName)
  {
    _tcscpy_s(g_szProcName, szProcName);
  }
#ifdef __cplusplus
}
#endif
```

## Dll注入

我的程序是`Win32`位的,所以只能注入`Win32`的程序,我通过在`Win32`唤起`Win64`的程序,`FindWindow`,得到窗口句柄然后发送消息,`Win64`程序我用了窗口隐藏,调用后即可销毁,这是注入的方法,借鉴一篇看雪无模块加载注入`DLL`我简单的说下我的理解,也算对他那篇文章做下解释吧..

它的文章地址:<http://bbs.pediy.com/showthread.php?p=1243996>

### Nomal Dll Inject Demo

```
//创建一个进程互斥量  防止无模块DLL多次注入
BOOL IsMutexExist(char* pstrMutex)
{
  BOOL bRet = FALSE;
  HANDLE hMutex = NULL;
  hMutex = CreateMutexA(NULL, TRUE, pstrMutex);
  if ( hMutex )
  {
    if ( GetLastError() == ERROR_ALREADY_EXISTS )
      bRet = TRUE;
    ReleaseMutex(hMutex);
    CloseHandle(hMutex);
  }
  else
  {
    bRet = TRUE;
  }
  return bRet;
}
//调用LoadPE.cpp里的函数，自行处理PE加载，把DLL在新申请的内存加载起来，并执行入口函数
void LaunchNoModule()
{
  LaunchDll(dllModuleName,NO_MODULE_MARK);
```
