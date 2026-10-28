
```c
_FX BOOL SbieDll_StartBoxedService(const WCHAR *ServiceName, BOOLEAN WithAdd) // WithAdd = False
{
    HANDLE hkey;
    SERVICE_STATUS ss;
    SERVICE_QUERY_RPL *rpl;
    ULONG retries, error;

    WCHAR text[130];
    Sbie_snwprintf(text, 130, L"StartBoxedService; name: '%s'", ServiceName);
    SbieApi_MonitorPutMsg(MONITOR_SCM, text);

    //
    // when invoked from SandboxieRpcSs to handle StartProcess,
    // specify WithAdd to add the service to the sandbox
    //

    if (WithAdd)
        Scm_AddBoxedService(ServiceName);

    //
    // get service parameters and check that the service can be started
    //

    rpl = Scm_QueryBoxedServiceByName(ServiceName, TRUE, -1);
    if (! rpl)
        return FALSE;

    // 服务处于RUNNING状态，无需启动
    if (rpl->service_status.dwCurrentState != SERVICE_STOPPED &&
        rpl->service_status.dwCurrentState != SERVICE_START_PENDING) {
        Dll_Free(rpl);
        SetLastError(ERROR_SERVICE_ALREADY_RUNNING);
        return FALSE;
    }

    // 启动的服务是个驱动，无法启动，报错。
    if (rpl->service_status.dwServiceType & SERVICE_DRIVER) {
        SbieApi_Log(2103, L"%S [%S] (StartService)", ServiceName, Dll_BoxName);
        Dll_Free(rpl);
        SetLastError(ERROR_ACCESS_DENIED);
        return FALSE;
    }

    // 如果启动的服务不是一个exe或dll，暂时不支持，报错。
    if (! (rpl->service_status.dwServiceType & SERVICE_WIN32)) {
        Dll_Free(rpl);
        SetLastError(ERROR_SERVICE_LOGON_FAILED);
        return FALSE;
    }

    //
    // set service status - start pending
    //

    hkey = Scm_OpenKeyForService(ServiceName, TRUE);
    if (! hkey) {
        error = GetLastError();
        Dll_Free(rpl);
        SetLastError(error);
        return FALSE;
    }

    //
    // indicate the service is initializing
    //

    memzero(&ss, sizeof(SERVICE_STATUS));
    ss.dwCurrentState = SERVICE_START_PENDING;
    ss.dwControlsAccepted = SERVICE_ACCEPT_STOP;
    ss.dwWaitHint = 5000;

    // 设置该服务的状态，其实就是设置注册表键对应的值
    Scm_SetServiceStatus_Internal(
        hkey, HANDLE_SERVICE_STATUS, &ss, FALSE);

    CloseHandle(hkey);

    //
    // launch the service
    //

    error = Scm_StartBoxedService2(ServiceName, rpl);

    Dll_Free(rpl);

    //
    // wait for the service to indicate it has started
    //

    if (! error) {
        error = ERROR_SERVICE_LOGON_FAILED;
        for (retries = 0; retries < 40; ++retries) {
            Sleep(500);
            rpl = Scm_QueryBoxedServiceByName(ServiceName, TRUE, 0);
            if (! rpl)
                return FALSE;

            // 查看服务是否启动成功
            if (rpl->service_status.dwCurrentState == SERVICE_RUNNING) {
                error = 0;
                break;
            }

            Dll_Free(rpl);
        }
    }

    ......
    SetLastError(0);
    return TRUE;
}
```

`Scm_StartBoxedService2`函数对应的源码如下：

```c
_FX ULONG Scm_StartBoxedService2(const WCHAR *name, SERVICE_QUERY_RPL *qrpl)
{
    SERVICE_RUN_REQ *req;
    MSG_HEADER *rpl;
    ULONG error, path_len, req_len;
    WCHAR *path;
    BOOL free_path;

    //
    // special handling for Sandboxie services:
    // either run directly (for SandboxieCrypto)
    // or build a path to the Sandboxie executable
    //

    free_path = FALSE;
    // _bits -> 后台智能传输服务，_wuauserv -> Windows更新服务，Scm_CryptSvc -> 加密服务
    // 这三个特殊服务特殊处理
    if (_wcsicmp(name, _bits) == 0           ||
        _wcsicmp(name, _wuauserv) == 0       ||
        _wcsicmp(name, Scm_CryptSvc) == 0) {

        //PROCESS_INFORMATION pi;
        STARTUPINFO si;

        const WCHAR *ProcessName;
        //BOOLEAN use_sbiesvc = TRUE;

        if (_wcsicmp(name, _bits) == 0) {
            ProcessName = SandboxieBITS;
            Scm_DeletePermissions(L"69AD4AEE-51BE-439B-A92C-86AE490E8B30");
        } else if (_wcsicmp(name, _wuauserv) == 0) {
            ProcessName = SandboxieWUAU;
            Scm_DeletePermissions(L"653C5148-4DCE-4905-9CFD-1B23662D3D9E");
        } else if (_wcsicmp(name, Scm_CryptSvc) == 0) {
            ProcessName = SandboxieCrypto;
            //use_sbiesvc = FALSE;
        } else
            ProcessName = NULL;

        si.lpReserved = NULL;
        if (SbieDll_RunFromHome(ProcessName, NULL, &si, NULL)) {
            path = (WCHAR *)si.lpReserved;
            if (path)
                free_path = TRUE;
        }
    }

    if (! free_path) {
        path = Scm_GetServiceConfigString(qrpl, 'P');
    }

    //
    // otherwise start service through SbieSvc
    //

    path_len = (wcslen(path) + 1) * sizeof(WCHAR);
    req_len = sizeof(SERVICE_RUN_REQ) + path_len;

    req = Dll_Alloc(req_len);
    req->h.length = req_len;
    req->h.msgid = MSGID_SERVICE_RUN;

    req->type = qrpl->service_status.dwServiceType;

    if (_wcsicmp(name, L"McAfee SiteAdvisor Service") == 0) {
        //
        // the McAfee SiteAdvisor Service is a WIN32_SHARE_PROCESS service
        // when it is part of the Total Protection suite, so we need to
        // make sure it will always be treated as SERVICE_WIN32_OWN_PROCESS
        //

        req->type = SERVICE_WIN32_OWN_PROCESS;
    }

    wcsncpy(req->name, name, 64);
    req->name[63] = L'\0';

    req->devmap[0] = L'\0';
    File_GetSetDeviceMap(req->devmap);

    req->path_len = path_len;
    memcpy(req->path, path, path_len);

    // 向SbieSvc发送MSGID_SERVICE_RUN请求，创建进程
    rpl = (MSG_HEADER *)SbieDll_CallServer(&req->h);
    if (! rpl)
        error = ERROR_NOT_ENOUGH_MEMORY;
    else {
        error = rpl->status;
        Dll_Free(rpl);
    }

    //
    // finish
    //

    if (free_path)
        HeapFree(GetProcessHeap(), HEAP_GENERATE_EXCEPTIONS, path);

    return error;
}
```

调用`StartService`之后，对应服务的注册表内容如下：

![scm_startservice](https://bbs.kanxue.com/upload/attach/202506/738427_F5ACBCC9BYXF95R.webp)

使用DebugView可以看到服务输出的内容，如下图：

![scm_startservice](https://bbs.kanxue.com/upload/attach/202506/738427_R6C2PDEYF7WFXDN.webp)

启动的服务进程会被再次注入`SbieDll`，其对`StartServiceCtrlDispatcher` hook的代码如下：
