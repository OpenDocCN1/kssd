
# Windows 服务程序编写

> 原文：[`www.pediy.com/kssd/pediy10/88110.html`](https://www.pediy.com/kssd/pediy10/88110.html)

工作中常常需要编写 Service 程序，在 linux 中叫守护进程，写出来和大家共同进步，欢迎批评指正，高手飘过。

`ServiceDemo` 采用 .net 2003 写的，没有实现复杂的功能，只是定时向事件察看器中写入当前系统的时间，个人觉得在登陆前启动一个程序在登陆桌面上还是挺有意思的。

安装方法： `ServiceDemo - install`，安装后需要在服务中手动启动 `ServiceDemo` 程序，方可在事件察看器中看到程序运行效果。

卸载方法： `ServiceDemo - unisntall`。

涉及到的主要函数

### （1）服务入口点函数（`ServiceMain`）

执行服务初始化任务。

```c
void WINAPI ServiceMain(DWORD dwArgc, LPTSTR *lpszArgv)
{
  DWORD dwErr = 0;

  // register our service control handler:
  g_hServiceStatus = RegisterServiceCtrlHandler(g_szServiceName, ServiceCtrl);

  // SERVICE_STATUS members don't change
  g_SevicStatus.dwServiceType = SERVICE_WIN32_OWN_PROCESS;
  g_SevicStatus.dwServiceSpecificExitCode = 0;

  // report the status to the service control manager.
  if (!ReportStatusToSCM(SERVICE_START_PENDING, NO_ERROR, 3000))
  {
    if (g_hServiceStatus)
      ReportStatusToSCM(SERVICE_STOPPED, dwErr, 0);
  }

  ServiceStart(dwArgc, lpszArgv);

  // try to report the stopped status to the service control manager.
  if (g_hServiceStatus)
    ReportStatusToSCM(SERVICE_STOPPED, dwErr, 0);

  return;
}
```

### （2）控制服务处理程序函数

```c
void WINAPI ServiceCtrl(DWORD dwCtrlCode)
{
  // Handle the requested control code
  switch(dwCtrlCode)
  {
    // Stop the service.
    // SERVICE_STOP_PENDING should be reported before
    // setting the Stop Event - hServerStopEvent - in
    // EsServiceStop().  This avoids a race condition
    // which may result in a 1053 - The Service did not respond...
    // error.
  case SERVICE_CONTROL_STOP:
    ReportStatusToSCM(SERVICE_STOP_PENDING, NO_ERROR, 0);
    ServiceStop();
    return;

    // Update the service status.
  case SERVICE_CONTROL_INTERROGATE:
    break;

    // invalid control code
  default:
    break;
  }

  ReportStatusToSCM(g_SevicStatus.dwCurrentState, NO_ERROR, 0);
}
```

### （3）安装服务的函数

```c
void CmdInstallService()
{
  SC_HANDLE   schService;
  SC_HANDLE   schSCManager;
  TCHAR szPath[512];

  if (GetModuleFileName(NULL, szPath, 512) == 0)
  {
    _tprintf(_T("Unable to install Service\n"));
    return;
  }

  schSCManager = OpenSCManager(  NULL,                 // machine (NULL == local)
    NULL,                 // database (NULL == default)
    SC_MANAGER_ALL_ACCESS  );  // access required

  if (schSCManager)
  {
    schService = CreateService(  schSCManager,               // SCManager database
      g_szServiceName,      // name of service
      g_szServiceName,      // name to display
      SERVICE_ALL_ACCESS,         // desired access
      SERVICE_WIN32_OWN_PROCESS | SERVICE_INTERACTIVE_PROCESS ,  // service type

      SERVICE_AUTO_START,
      SERVICE_ERROR_IGNORE,       // error control type
      szPath,                   // service's binary
      NULL,                     // no load ordering group
      NULL,                     // no tag identifier
      _T(""),        // dependencies
      NULL,                     // LocalSystem account
      NULL   );               // no password

    if (schService)
    {
      _tprintf(_T("ServiceDemo is installed.\n"));
      CloseServiceHandle(schService);
    }
    else
    {
      _tprintf(_T("CreateService failed"));
    }

    CloseServiceHandle(schSCManager);
  }
  else
    _tprintf(_T("OpenSCManager failed"));
}
```

### （4）卸载函数

```c
void CmdUninstallService()
{
  SC_HANDLE   schService;
  SC_HANDLE   schSCManager;

  schSCManager = OpenSCManager(  NULL,
    NULL,
    SC_MANAGER_ALL_ACCESS  );

  if (schSCManager)
  {
    schService = OpenService(schSCManager, g_szServiceName, SERVICE_ALL_ACCESS);

    if (schService)
    {
      // try to stop the service
      if (ControlService(schService, SERVICE_CONTROL_STOP, &g_SevicStatus))
      {
        _tprintf(_T("Stopping the service."));
        Sleep(1000);
        while(QueryServiceStatus(schService, &g_SevicStatus))
        {
          if (g_SevicStatus.dwCurrentState == SERVICE_STOP_PENDING)
          {
            _tprintf(_T("."));
            Sleep(1000);
          }
          else
            break;
        }

        if (g_SevicStatus.dwCurrentState == SERVICE_STOPPED)
        {
          _tprintf(_T("\nthe service stopped.\n"));
        }
        else
          _tprintf(_T("\nFailed to stop service.\n"));
      }

      // now remove the service
      if(DeleteService(schService))
        _tprintf(_T("the service removed.\n"));
      else
        _tprintf(_T("DeleteService failed.\n"));

      CloseServiceHandle(schService);
    }
    else
      _tprintf(_T("OpenService failed.\n"));

    CloseServiceHandle(schSCManager);
  }
  else
    _tprintf(_T("OpenSCManager failed.\n"));
}
```

关于 Service 更详细的知识可以参考 Windows internals。

代码中登记日志信息的类采用 Singleton 设计模式实现。

## 上传的附件

| 图片 | 文件 |
| :--- | :--- |
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | `ServiceDemo.rar` |
