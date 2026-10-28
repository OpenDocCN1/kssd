
# [原创]创建远程线程，将代码注入到其它进程中执行

最近在学SDK编程，把学习过程中一些认为比较精彩的部分想留下来，第一作为以后复习的资源；第二可以给新学的朋友们一个参考，互相讨论；另外想在看雪里认识一些正在学习DDK驱动开发的朋友，交流一下，应该怎样学习驱动编程！谢谢，话不多说了！

## 创建远程线程，将代码注入到其他进程中执行

关键知识点如下：

### 关键API

```c
HANDLE CreateRemoteThread(
  HANDLE hProcess,                          // 目标进程句柄
  LPSECURITY_ATTRIBUTES lpThreadAttributes, // 安全属性
  SIZE_T dwStackSize,                       // 进程堆栈大小
  LPTHREAD_START_ROUTINE lpStartAddress,    // 进程函数
  LPVOID lpParameter,                       // 进程参数
  DWORD dwCreationFlags,                    // 创建标志
  LPDWORD lpThreadId                        // 参数返回ID
);
```

```c
BOOL WriteProcessMemory(
  HANDLE hProcess,                // 目标进程句柄
  LPVOID lpBaseAddress,           // 目标进程虚拟地址空间的地址
  LPCVOID lpBuffer,               // 需要写入的数据
  SIZE_T nSize,                   // 需要写入数据的大小
  SIZE_T * lpNumberOfBytesWritten // 实际写入数据大小
);
```

主要就是这两个函数，还有其它一个内存分配的函数如：`VirtualAllocEx`,`VirtualFreeEx`等，请读者自行参考MSDN，英文不是我的强项，上面的完全是自己的理解翻译，大牛们不要笑哦！

首先随便写一个DLL了

代码如下：

```c
#include <windows.h>
_declspec(dllexport) DWORD Example(LPSTR szMsg,DWORD dwCode);

/***************************************************************************
*DllMain
****************************************************************************/
BOOL WINAPI DllMain(
                                        HANDLE hinst,           //DLL模块句柄
                                        DWORD fdwReason,        //调用情况
                                        LPVOID lpReserved       //reserved
                                        )
{
        //在不同的情况下都会调用DllMain函数，分别处理
        switch(fdwReason)
        {
                //加载DLL
        case DLL_PROCESS_ATTACH:
                {
                        MessageBox(NULL,"DLL加载时调用","Msg.dll",MB_OK);
                        break;
                }
                //新建线程
        case DLL_THREAD_ATTACH:
                break;
                //线程退出
        case DLL_THREAD_DETACH:
                break;
                //释放DLL
        case DLL_PROCESS_DETACH:
                break;
        }
        return TRUE;
}

/***************************************************************************
*DWORD Example(LPSTR szMsg,DWORD dwCode)
*功能：导出函数,显示消息
*参数：LPSTR szMsg,字符串,DWORD dwCode，整形
****************************************************************************/
DWORD Example(LPSTR szMsg,DWORD dwCode)
{
        MessageBox(NULL,"由导出函数弹出的消息","导出函数",MB_OK);
        return 0;
}
```

把生成的DLL放到系统目录下

然后在将DLL代码注入到其它进程中执行，代码如下：

`RemoteInject.cpp`

```c
#include <windows.h>
#include <winbase.h>
#include <winnt.h>
#include <stdio.h>
#include <Tlhelp32.h>
/***************************************************************************
*BOOL LoadRometeDll(DWORD dwProcessId,LPTSTR lpszLibName)
*功能：通过创建远程线程给其他进程加载DLL
*参数：DWORD dwProcessId 目标进程PID
* LPTSTR lpszLibName Dll的路径
*返回：是否成功
****************************************************************************/
BOOL LoadRometeDll(DWORD dwProcessId,LPTSTR lpszLibName)
{
        BOOL bResult = FALSE;
        HANDLE hProcess = NULL;
        HANDLE hThread = NULL;
        PSTR pszLibFileRemote = NULL;
        DWORD cch;
        PTHREAD_START_ROUTINE pfnThreadRtn;
        _try
        {
                //获得想要注入代码的进程的句柄
                hProcess = OpenProcess(PROCESS_ALL_ACCESS,FALSE,dwProcessId);
                if (hProcess == NULL)
                        _leave;
                //计算DLL路径名需要的字节数
                cch = 1 + strlen(lpszLibName);
                //在远程线程中为路径名分配空间
                pszLibFileRemote = (PSTR)VirtualAllocEx(
                        hProcess,
                        NULL,
                        cch,
                        MEM_COMMIT,
                        PAGE_READWRITE
                        );
                if (pszLibFileRemote == NULL)
                        _leave;
                //将DLL的路径名复制到远程进程的内存空间
                if (!WriteProcessMemory(
                        hProcess,
                        (PVOID)pszLibFileRemote,
                        (PVOID)lpszLibName,
                        cch,
                        NULL
                        ))
                        _leave;
                //获得LoadLibraryA在Kernel32.dll中的真正地址
                pfnThreadRtn = (PTHREAD_START_ROUTINE)GetProcAddress(
                        GetModuleHandle(TEXT("Kernel32")),TEXT("LoadLibraryA"));
                if (pfnThreadRtn == NULL)
                        _leave;
                //创建远程线程，并通过远程线程调用用户的DLL文件
                hThread = CreateRemoteThread(hProcess,NULL,0,pfnThreadRtn,(LPVOID)pszLibFileRemote,0,NULL);
                if (hThread == NULL)
                        _leave;
                //等待远程线程终止
                WaitForSingleObject(hThread,INFINITE);
                bResult = TRUE;
        }
        _finally
        {
                //关闭句柄
                if (pszLibFileRemote != NULL)
                        VirtualFreeEx(hProcess,(PVOID)pszLibFileRemote,0,MEM_RELEASE);
                if (hThread != NULL)
                        CloseHandle(hThread);
                if (hProcess != NULL)
                        CloseHandle(hProcess);
        }
        return bResult;
}
```

```c
/***************************************************************************
*EnablePrivilege修改进程的权限
****************************************************************************/
BOOL EnablePrivilege(LPSTR name)
{
        HANDLE hToken;
        BOOL rv;
        TOKEN_PRIVILEGES priv = {1,{0,0,SE_PRIVILEGE_ENABLED}};
        LookupPrivilegeValue(0,name,&priv.Privileges[0].Luid);
        OpenProcessToken(GetCurrentProcess(),TOKEN_ADJUST_PRIVILEGES,&hToken);
        AdjustTokenPrivileges(hToken,FALSE,&priv,sizeof priv,0,0);
        rv = GetLastError() == ERROR_SUCCESS;
        CloseHandle(hToken);
        return rv;
}
```

```c
/***************************************************************************
*BOOL GetProcessIdByName(LPSTR szProcessName,LPDWORD lpPID)
*功能：通过进程名获取进程PID
*参数：LPSTR szProcessName 进程名
           LPDWORD lpPID       指向保存PID的变量
*返回是否成功
****************************************************************************/
BOOL GetProcessIdByName(LPSTR szProcessName,LPDWORD lpPID)
{
        //变量及初始化
        STARTUPINFO st;
        PROCESS_INFORMATION pi;
        PROCESSENTRY32 ps;
        HANDLE hSnapshot;
        ZeroMemory(&st,sizeof(STARTUPINFO));
        ZeroMemory(&pi,sizeof(PROCESS_INFORMATION));
        st.cb = sizeof(STARTUPINFO);
        ZeroMemory(&ps,sizeof(PROCESSENTRY32));
        ps.dwSize = sizeof(PROCESSENTRY32);
        //遍历进程
        hSnapshot = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS,0);
        if (hSnapshot == INVALID_HANDLE_VALUE)
        {
                return FALSE;
        }
        if (!Process32First(hSnapshot,&ps))
        {
                return FALSE;
        }
        do
        {
                //比较进程名
                if (lstrcmpi(ps.szExeFile,"explorer.exe") == 0)
                {
                        //找到了
                        *lpPID = ps.th32ProcessID;
                        CloseHandle(hSnapshot);
                        return TRUE;
                }
        } while (Process32Next(hSnapshot,&ps));
        //没有找到
        CloseHandle(hSnapshot);
        return FALSE;
}
```

```c
/***************************************************************************
*WinMain
****************************************************************************/
int WinMain( HINSTANCE hInstance, HINSTANCE hPrevInstance, LPSTR lpCmdLine, int nShowCmd )
{
        DWORD dwPID;
```

