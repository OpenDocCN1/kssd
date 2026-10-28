
```
FX BOOL Scm_StartServiceCtrlDispatcherX(
    const void *ServiceTable, BOOLEAN IsUnicode)
{
    static const WCHAR *ServiceName_EnvVar =
        L"00000000_" SBIE L"_SERVICE_NAME";
    WCHAR *ServiceName;
    WCHAR *Buffer;
    UNICODE_STRING uni;
    void *args[3];
    ULONG ThreadId;
    HANDLE hEvent;
    SERVICE_STATUS ss;
    NTSTATUS status;
    IO_STATUS_BLOCK iosb;

    ...... // 通过环境变量获取服务的名称，设置注册表中服务的状态
    //
    // launch ServiceMain to initialize
    //

    if (IsUnicode) {
        args[0] = ((SERVICE_TABLE_ENTRYW *)ServiceTable)->lpServiceProc;
        args[1] = ((SERVICE_TABLE_ENTRYW *)ServiceTable)->lpServiceName;
        args[2] = NULL;

    } else {
        args[0] = ((SERVICE_TABLE_ENTRYA *)ServiceTable)->lpServiceProc;
        args[1] = ((SERVICE_TABLE_ENTRYA *)ServiceTable)->lpServiceName;
        args[2] = NULL;
    }

    if (_wcsicmp(ServiceName, Scm_MsiServer) == 0) {
        Scm_IsMsiServer = TRUE;
        Scm_SetupMsiHooks();
    }

    // 执行ServiceMain函数，ServiceMain函数会设置Scm_Handler或Scm_HandlerEx
    HANDLE ThreadHandle = CreateThread(NULL, 0, Scm_ServiceMainThread, args, 0, &ThreadId);
    if (ThreadHandle)
        CloseHandle(ThreadHandle);
    else
        Scm_Stopped = TRUE;

    //
    // main loop:  wait for changes on the service key
    //

    hEvent = CreateEvent(NULL, FALSE, FALSE, NULL);
    if (! hEvent) {
        SbieApi_Log(2211, ServiceName);
        return FALSE;
    }

    Buffer = Dll_Alloc(1024);
    RtlInitUnicodeString(&uni, SBIE L"_ControlCode");

    status = NtNotifyChangeKey(
        Scm_ServiceKey, hEvent, NULL, NULL, &iosb,
        REG_NOTIFY_CHANGE_LAST_SET, FALSE, Buffer, 1024, TRUE);

    if (!NT_SUCCESS(status)) {
        SbieApi_Log(2211, ServiceName);
        return FALSE;
    }

    // 当ServiceCtrlHandler函数设置服务的状态为STOP时，Scm_Stopped = TRUE，跳出while循环
    while (! Scm_Stopped) {

        union {
            KEY_VALUE_PARTIAL_INFORMATION info;
            WCHAR info_space[64];
        } u;
        ULONG len;

        // Wait for the reg value notification event, or 1 sec.
        status = WaitForSingleObject(hEvent, 1000);

        // 当服务所在的注册表键发生改变，进入到if中
        if (NT_SUCCESS(status)) {
            Scm_DiscardKeyCache(ServiceName);
            status = NtQueryValueKey(
                Scm_ServiceKey, &uni, KeyValuePartialInformation,
                &u.info, sizeof(u), &len);

            // 发生控制命令给ServiceCtrlHandler函数
            if (NT_SUCCESS(status) &&
                u.info.Type == REG_DWORD && u.info.DataLength == 4) {
                // 执行ServiceCtrlHandler，参数是控制码
                if (Scm_Handler)
                    Scm_Handler(*(ULONG *)u.info.Data);
                // 执行ServiceCtrlHandler，参数是控制码
                else if (Scm_HandlerEx) {
                    Scm_HandlerEx(
                        *(ULONG *)u.info.Data, 0, NULL, Scm_HandlerContext);
                }
            }

            status = NtDeleteValueKey(Scm_ServiceKey, &uni);
            if (NT_SUCCESS(status)) { // if key was actually deleted, we need to reissue the reg notification
                status = NtNotifyChangeKey(
                    Scm_ServiceKey, hEvent, NULL, NULL, &iosb,
                    REG_NOTIFY_CHANGE_LAST_SET, FALSE, Buffer, 1024, TRUE);
            }
        }
    }

    //
    // if the service never started, report error
    //

    Sbie_snwprintf(text, 130, L"StartServiceCtrlDispatcher; result: %s", Scm_Started ? L"success" : L"failure");
    SbieApi_MonitorPutMsg(MONITOR_SCM, text);

    if (! Scm_Started) {
        SbieApi_Log(2211, ServiceName);
        return FALSE;
    }

    return TRUE;
}
```

对 `RegisterServiceCtrlHandler` 函数 hook 的源码主要设置 `Scm_Handler` 或 `Scm_HandlerEx`，如下：

```
_FX SERVICE_STATUS_HANDLE Scm_RegisterServiceCtrlHandlerHelp(
    UCHAR *ServiceName, ULONG_PTR HandlerProc, void *Context, ULONG Flags)
{
    if (Flags == 0) {
        Scm_Handler = (LPHANDLER_FUNCTION)HandlerProc;
    } else if (Flags == 2) {
        Scm_HandlerEx = (LPHANDLER_FUNCTION_EX)HandlerProc;
        Scm_HandlerContext = Context;
    } else {
        SbieApi_Log(2205, L"RegisterServiceCtrlHandlerHelp");
        SetLastError(ERROR_SERVICE_NOT_IN_EXE);
        return NULL;
    }
    return (SERVICE_STATUS_HANDLE)HANDLE_SERVICE_STATUS;
}
```

### 服务控制 `ControlService`

该函数可产生暂停、停止和恢复服务等效果，主要逻辑：设置服务的注册表 `SBIE_ControlCode` 键实现。上文中的 `Scm_StartServiceCtrlDispatcherX` 函数设置了 `hEvent` 来监听是否设置了 `SBIE_ControlCode`，如果设置了就调用 `Scm_Handler` 或 `Scm_HandlerEx`，通过这种方式实现了模拟 SCM 发送各种控制码给 `ServiceCtrlHandler` 函数的过程。

```
_FX BOOL Scm_ControlServiceImpl(
    SC_HANDLE hService, DWORD dwControl, SERVICE_STATUS *lpServiceStatus)
{
    NTSTATUS status;
    WCHAR *ServiceName;
    HANDLE hkey;
    SERVICE_QUERY_RPL *rpl;
    UNICODE_STRING uni;
    ULONG val, retries;
    union {
        KEY_VALUE_PARTIAL_INFORMATION info;
        WCHAR info_space[64];
    } u;

    //
    // get service name
    //

    ServiceName = Scm_GetHandleName(hService);
    if (! ServiceName)
        return FALSE;

    if (! Scm_IsBoxedService(ServiceName)) {
        if (dwControl == SERVICE_CONTROL_CONTINUE ||
            dwControl == SERVICE_CONTROL_INTERROGATE) {
            return Scm_QueryServiceStatusImpl(hService, lpServiceStatus);
        } else {
            SetLastError(ERROR_ACCESS_DENIED);
            return FALSE;
        }
    }

    //
    // verify that the service is in the running or paused state
    //

    rpl = Scm_QueryBoxedServiceByName(ServiceName, TRUE, 0);
    if (! rpl)
        return FALSE;
    val = rpl->service_status.dwCurrentState;
    Dll_Free(rpl);

    if (val == SERVICE_STOPPED) {
        SetLastError(ERROR_SERVICE_NOT_ACTIVE);
        return FALSE;
    }

    if (val != SERVICE_RUNNING && val != SERVICE_PAUSED) {
        SetLastError(ERROR_SERVICE_CANNOT_ACCEPT_CTRL);
        return FALSE;
    }

    //
    // open the key for the service and transmit the control
    //

    hkey = Scm_OpenKeyForService(ServiceName, TRUE);
    if (! hkey) {
        SetLastError(ERROR_SERVICE_REQUEST_TIMEOUT);
        return FALSE;
    }

    RtlInitUnicodeString(&uni, SBIE L"_ControlCode"); //
    val = dwControl;
    // 设置SBIE_ControlCode键对应的值，也就是ServiceCtrlHandler函数的Ctrl参数
    status = NtSetValueKey(
        hkey, &uni, 0, REG_DWORD, (BYTE *)&val, sizeof(ULONG));
    if (! NT_SUCCESS(status)) {
        NtClose(hkey);
        SetLastError(ERROR_SERVICE_REQUEST_TIMEOUT);
        return FALSE;
    }

    //
    // wait up for the value to disappear, this indicates the
    // service has picked up the control
    //
    // 多次循环，尝试等待Scm_Handler或Scm_HandlerEx回调函数执行完成
    for (retries = 0; retries < 40; ++retries) {

        Scm_DiscardKeyCache(ServiceName);
        status = NtQueryValueKey(
            hkey, &uni, KeyValuePartialInformation,
            &u.info, sizeof(u), &val);
        if (NT_SUCCESS(status)) {
            Sleep(500);
            continue;
        }
        break;
    }

    NtClose(hkey);

    if (status != STATUS_OBJECT_NAME_NOT_FOUND) {
        SetLastError(ERROR_SERVICE_REQUEST_TIMEOUT);
        return FALSE;
    }

    return Scm_QueryServiceStatusImpl(hService, lpServiceStatus);
}
```

### 服务查询

这些函数主要查询注册表中的信息，一些函数如下：

- `QueryServiceStatus`，查询服务的状态。
- `EnumServicesStatus`，枚举服务的状态。
- `GetServiceDisplayName` 和 `GetServiceKeyName`，这两个函数分别用于获取服务的 `DisplayName` 信息和服务名。
