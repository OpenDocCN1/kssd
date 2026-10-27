
# [原创]ring3下过某游戏的HS保护

标 题: 【原创】ring3下过某游戏的HS保护

作 者: `fbbttfbb`

时 间: 2012-03-04,13:06:33

声明：本文只为研究技术,如使用本文中的源码及技术产生了侵权或违法行为,本人概不负责。如有侵犯你版权的行为，请联系我，本文将立即改正!

## 说明

`HS`有`ring0`驱动保护及`ring3`下保护,这个方法是在不加载驱动下,就轻松过`HS`游戏保护,是自己这几天研究的成果,现在发表出来.游戏是韩国的名品

### <1>下载附件中的`ehsvc.dll`文件替换原来游戏HS目录下的文件

### <2>HOOK `CreateProcess`, `CreateFile`, `NtSetInformationThread`

`HOOK CreateProcess`的目的是让`HS`的`hsupdate.exe`(`HS`更新进程)进程不让起来

``` 
BOOL WINAPI Mine_CreateProcessA(        LPCSTR lpApplicationName,
                                                                        LPSTR lpCommandLine,
                                                                        LPSECURITY_ATTRIBUTES lpProcessAttributes,
                                                                        LPSECURITY_ATTRIBUTES lpThreadAttributes,
                                                                        BOOL bInheritHandles,
                                                                        DWORD dwCreationFlags,
                                                                        LPVOID lpEnvironment,
                                                                        LPCSTR lpCurrentDirectory,
                                                                        LPSTARTUPINFOA lpStartupInfo,
                                                                        LPPROCESS_INFORMATION lpProcessInformation)
{
        CString strtemp= lpCommandLine;
        strtemp.MakeLower();
        if(strtemp.Find("hsupdate.exe") !=-1)
                return FALSE;
        BOOL bret = FALSE;
        g_hook_CreateProcessA.UnHook();
        bret = CreateProcessA(        lpApplicationName,
                lpCommandLine,
                lpProcessAttributes,
                lpThreadAttributes,
                bInheritHandles,
                dwCreationFlags,
                lpEnvironment,
                lpCurrentDirectory,
                lpStartupInfo,
                lpProcessInformation);
        g_hook_CreateProcessA.Hook();
        return bret;
}
```

``` 
BOOL WINAPI Mine_CreateProcessW(        LPCWSTR lpApplicationName,
                                                                        LPWSTR lpCommandLine,
                                                                        LPSECURITY_ATTRIBUTES lpProcessAttributes,
                                                                        LPSECURITY_ATTRIBUTES lpThreadAttributes,
                                                                        BOOL bInheritHandles,
                                                                        DWORD dwCreationFlags,
                                                                        LPVOID lpEnvironment,
                                                                        LPCWSTR lpCurrentDirectory,
                                                                        LPSTARTUPINFOW lpStartupInfo,
                                                                        LPPROCESS_INFORMATION lpProcessInformation)
{
        USES_CONVERSION;
        CString strtemp=W2A(lpCommandLine);
        strtemp.MakeLower();
        if(strtemp.Find("hsupdate.exe") !=-1)
                return FALSE;
        BOOL bret=FALSE;
        g_hook_CreateProcessW.UnHook();
        bret = CreateProcessW(        lpApplicationName,
                lpCommandLine,
                lpProcessAttributes,
                lpThreadAttributes,
                bInheritHandles,
                dwCreationFlags,
                lpEnvironment,
                lpCurrentDirectory,
                lpStartupInfo,
                lpProcessInformation);
        g_hook_CreateProcessW.Hook();
        return bret;
}
```

`HOOK CreateFile`的目的是让`EagleXNt.sys`(`HS`驱动)不起来

``` 
HANDLE WINAPI Mine_CreateFileA(LPCSTR lpFileName,
                                                           DWORD dwDesiredAccess,
                                                           DWORD dwShareMode,
                                                           LPSECURITY_ATTRIBUTES lpSecurityAttributes,
                                                           DWORD dwCreationDisposition,
                                                           DWORD dwFlagsAndAttributes,
                                                           HANDLE hTemplateFile)
{
        int pos = string(lpFileName).find("EagleXNt");
        if(pos!=-1)
                return NULL;
        HANDLE hhh = NULL;
        g_hook_CreateFileA.UnHook();
        hhh = CreateFileA(lpFileName,
                dwDesiredAccess,
                dwShareMode,
                lpSecurityAttributes,
                dwCreationDisposition,
                                                                dwFlagsAndAttributes,
                                hTemplateFile
                                                                );
        g_hook_CreateFileA.Hook();
        return hhh;
}
```

``` 
HANDLE WINAPI Mine_CreateFileW(LPCWSTR lpFileName,
                                                           DWORD dwDesiredAccess,
                                                           DWORD dwShareMode,
                                                           LPSECURITY_ATTRIBUTES lpSecurityAttributes,
                                                           DWORD dwCreationDisposition,
                                                           DWORD dwFlagsAndAttributes,
                                                           HANDLE hTemplateFile)
{
        int pos = wstring(lpFileName).find(L"EagleXNt");
        if(pos!=-1)
                return NULL;
        HANDLE hhh = NULL;
        g_hook_CreateFileW.UnHook();
        hhh = CreateFileW(lpFileName,
                dwDesiredAccess,
                dwShareMode,
                lpSecurityAttributes,
                dwCreationDisposition,
                                                                dwFlagsAndAttributes,
                                hTemplateFile
                                                                );
        g_hook_CreateFileW.Hook();
        return hhh;
}
```

`HOOK NtSetInformationThread`的目的是让线程可以调试

``` 
DWORD  WINAPI Mine_NtSetInformationThread(
                                                                                  HANDLE hThread,
                                                                                  DWORD ThreadInformationClass,
                                                                                  PVOID ThreadInformation,
                                                                                  ULONG ThreadInformationLength
                                                                                  )
{
        if (ThreadInformationClass == 17)
    {
        //HideFromDebugger
        return 1;
    }
        DWORD hhh = NULL;
        g_hook_NtSetInformationThread.UnHook();
        hhh = NtSetInformationThread(hThread,
                ThreadInformationClass,
                ThreadInformation,
                ThreadInformationLength);
        g_hook_NtSetInformationThread.Hook();
        return hhh;
}
```

### <3>暂停HS ring3下的保护线程

暂停`HS ring3`下的保护线程这里需要注意,只有在游戏启动后,进入游戏画面或者登陆画面,`HS ring3`下的保护线程才起来然后就暂停,这里在线程里处理

``` 
unsigned   WINAPI   threadproc(LPVOID   lpparam)
{
        g_stopthread = 0;
        BOOL Bretsus = FALSE;
        //等待35秒, 只有在游戏启动后,进入游戏画面或者登陆画面,HS ring3下的保护线程才//起来然后就暂停
        Sleep(35000);
        DWORD pid = GetCurrentProcessId();
        HANDLE   h   =   CreateToolhelp32Snapshot   (TH32CS_SNAPTHREAD,   pid);
        THREADENTRY32   te;
        te.dwSize   =   sizeof   (te);
        if(Thread32First   (h,   &te))
        {
                do
                {
                        Bretsus = SuspendHsThread(te.th32ThreadID, pid );
                }while   (Thread32Next   (h,   &te));
        };
        CloseHandle(h);
```
