
# Run `cmd.exe` as Local System

> 原文：[`www.pediy.com/kssd/pediy11/114122.html`](https://www.pediy.com/kssd/pediy11/114122.html)

由于已经申请看雪的号很久了，发现自己还是临时会员，最近又在学习内核编程，遇到了一些问题想跟高手请教一下，却不能在其他版块发帖，于是想写这篇文章来申请一下邀请码。

前段时间看了一篇文章《Running `CMD.EXE` as Local System》，引自 msdn 的博客 [http://blogs.msdn.com/b/adioltean/archive/2004/11/27/271063.aspx](http://blogs.msdn.com/b/adioltean/archive/2004/11/27/271063.aspx)，我自己实现了一下发现在 `xp` 下启动服务失败却还能使 `cmd.exe` 以 `system` 身份运行，但是在之后的操作系统上不行，于是我通过学习《深入解析 windows 操作系统》中的第四章的服务章节，自己再写了个程序发现在 `xp` 下可以启动服务成功，并能使 `cmd.exe` 以 `system` 身份运行，在 `win7` 上也能运行成功。

下面是程序的思路：

### 1. 编写服务程序

（1）任何一个应用程序都需要一个入口函数，一个服务程序必须具有服务主函数，服务主函数是服务启动时执行的入口，也是服务的主线程执行起点。服务主函数一般称作 `ServiceMain` 函数。但是服务主函数的名称与线程函数 `ThreadProc` 一样，其函数名并没有特殊要求，只是其参数接口和调用类型必须与要求一致。服务主函数的参数不是通过在命令行启动时设定的，而是通过 `SCM`（服务控制管理器）的相关 `API` `StartService` 进行传递的。

（2）`SCM` 要对服务进行管理，就必须知道服务程序的服务主函数。服务程序通过调用 `StartServiceCtrlDispatcher` 函数设置服务主函数，同时通知 `SCM`。`StartServiceCtrlDispatcher` 函数原型如下：

```
BOOL StartServiceCtrlDispatcher(const LPSERVICE_TABLE_ENTRY lpServicTable);
```

结构 `SERVICE_TABLE_ENTRY` 的原型如下：

```
typedef struct _SERVICE_TABLE_ENTRY {
    LPTSTR lpServiceName; //服务名称
    LPSERVICE_MAIN_FUNCTION lpServiceProc; //指向 ServiceMain 的函数指针
} SERVICE_TABLE_ENTRY, *LPSERVICE_TABLE_ENTRY;
```

只有将函数的指针赋给 `lpServiceProc`，再调用 `StartServiceCtrlDispatcher`，这个函数就成为了服务主函数。

（3）控制处理函数：

1）控制处理函数 `Handler`，控制处理函数用于处理 `SCM` 向服务传递的服务控制请求。控制处理函数原型如下：`VOID WINAPI Handler(DWORD fdwControl)`；与 `ServiceMain` 函数一致，其函数名没有特殊要求。

2）注册控制管理函数：`RegisterServiceCtrlHandler` 函数用于向 `SCM` 设置一个服务的控制处理函数。原型如下：

```
SERVICE_STATUS_HANDLE RegisterServiceCtrlHandler(
    LPCTSTR lpServiceName, //服务名称
    LPHANDLER_FUNCTION lpHandlerProc //为 Handler 函数指针
);
```

（4）下面是一个服务程序的流程：先填充 `SERVICE_STATUS` 结构，然后注册服务控制请求处理例程，代码如下：

```c
/***********
* VOID WINAPI SplSrvServiceStart (DWORD argc, LPTSTR *argv)
* 功能  服务启动函数
*
* 参数  未使用
***********/
VOID WINAPI SplSrvServiceStart (DWORD argc, LPTSTR *argv) 
{  
  DWORD status;  
  DWORD specificError;  
  HANDLE hThread;
  // 填充 SERVICE_STATUS 结构
  SplSrvServiceStatus.dwServiceType        = SERVICE_WIN32|SERVICE_INTERACTIVE_PROCESS;
  SplSrvServiceStatus.dwCurrentState       
    = SERVICE_START_PENDING;    // 服务在运行
  SplSrvServiceStatus.dwControlsAccepted    
    = SERVICE_ACCEPT_STOP | SERVICE_ACCEPT_PAUSE_CONTINUE; 
  SplSrvServiceStatus.dwWin32ExitCode      = 0; 
  SplSrvServiceStatus.dwServiceSpecificExitCode = 0; 
  SplSrvServiceStatus.dwCheckPoint         = 0; 
  SplSrvServiceStatus.dwWaitHint           = 0; 
  // 注册服务控制请求处理例程
  SplSrvServiceStatusHandle = RegisterServiceCtrlHandler( 
    "Sample_Srv",  // 服务名，在创建服务时使用了
    // SERVICE_WIN32_OWN_PROCESS，因此本参数被忽略。
    SplSrvServiceCtrlHandler);  // 控制请求处理例程，函数名

  if (SplSrvServiceStatusHandle == (SERVICE_STATUS_HANDLE)0) 
  { 
    SvcDebugOut(" [SPLSRV_SERVICE] RegisterServiceCtrlHandler "
      "failed %d\n", GetLastError()); 
    return; 
  }   
  // 初始化工作，本示例未使用，函数为空
  //status = SplSrvServiceInitialization(argc,argv, &specificError);   
  // 初始化出错，用户自行修改
  /*if (status != NO_ERROR) 
  { 
    SplSrvServiceStatus.dwCurrentState       = SERVICE_STOPPED; 
    SplSrvServiceStatus.dwCheckPoint         = 0; 
    SplSrvServiceStatus.dwWaitHint           = 0; 
    SplSrvServiceStatus.dwWin32ExitCode      = status; 
    SplSrvServiceStatus.dwServiceSpecificExitCode = specificError; 

    SetServiceStatus (SplSrvServiceStatusHandle, &SplSrvServiceStatus); 
    return; 
  } */
  // 初始化完成，设置运行状态
  SplSrvServiceStatus.dwCurrentState       = SERVICE_RUNNING; 
  SplSrvServiceStatus.dwCheckPoint         = 0; 
  SplSrvServiceStatus.dwWaitHint           = 0; 

  if (!SetServiceStatus (SplSrvServiceStatusHandle, &SplSrvServiceStatus)) 
  { 
    status = GetLastError(); 
    SvcDebugOut(" [SPLSRV_SERVICE] SetServiceStatus error %ld\n",status); 
  } 
  // 用户自行修改，用于完成服务的工作
  hThread=CreateThread(NULL,0,CmdService,NULL,0,NULL);
  SvcDebugOut(" [SPLSRV_SERVICE] Returning the Main Thread \n",0); 

  return; 
} 
```

线程函数如下：

```c
DWORD WINAPI CmdService(LPVOID lpParam)  
{  
        ShellExecute(0,0,TEXT("cmd.exe"),0,0,SW_SHOW);  
  return(0);  
}  
```

下面是入口主函数：

```c
int WINAPI WinMain( HINSTANCE hInstance, HINSTANCE hPrevInstance, LPSTR lpCmdLine, int nShowCmd )
{  
  // 设置 SERVICE_TABLE_ENTRY 数据结构，以 NULL 结构结束，
  // 作为 StartServiceCtrlDispatcher 函数的参数。
  SERVICE_TABLE_ENTRY   DispatchTable[] = 
  { 
    { "Sample_Srv", (LPSERVICE_MAIN_FUNCTION) SplSrvServiceStart }, 
    { NULL, NULL } 
  }; 
  if (!StartServiceCtrlDispatcher( DispatchTable)) 
  { 
    SvcDebugOut(" [SPLSRV_SERVICE] StartServiceCtrlDispatcher (%d)\n", 
      GetLastError()); 
  } 
} 
```

加为红色的类型 `SERVICE_INTERACTIVE_PROCESS`：允许该服务在控制台上显示窗口，并且接受用户的输入，刚开始我没加这个类型，运行时一直没看到 `cmd` 窗口，后来看书才明白是这样。

### 2. 编写对服务的控制和管理

通过 `OpenSCManager` 获得服务控制管理器句柄，然后用 `CreateService` 创建服务，然后是启动服务，主要过程代码如下：

```c
int WINAPI WinMain( HINSTANCE hInstance, HINSTANCE hPrevInstance, LPSTR lpCmdLine, int nShowCmd )
{
  TCHAR szBinFilePath[MAX_PATH];
  PTCHAR pTemp;

  // 构造服务可执行程序的路径
  GetModuleFileName(NULL,szBinFilePath,MAX_PATH);
  pTemp = szBinFilePath+lstrlen(szBinFilePath);
  while(*--pTemp!='\\');
  lstrcpy(pTemp,TEXT("\\Service.exe"));
   
  // 打开 SCM
  schSCManager = OpenSCManager( 
    NULL,                    // local machine 
    NULL,                    // ServicesActive database 
    SC_MANAGER_ALL_ACCESS);  // full access rights 
   
  if (NULL == schSCManager) 
  {
    wsprintf(szBuffer,TEXT("OpenSCManager failed (%d)"),GetLastError());
    MessageBox(NULL,szBuffer,NULL,MB_OK);
  }
  // 创建服务
  CreateSampleService(schSCManager, szBinFilePath, szServiceName);
  // 启动服务
  StartSampleService(schSCManager,szServiceName);
  // 停止服务
  DeleteSampleService(szServiceName);
  CloseServiceHandle(schSCManager); 
}
```

### 3. 总结

至此一个服务程序就写完了，程序附上来，第一次在看雪上发帖，有点小兴奋，希望能够得到邀请码，想在《安全编程》版块向高手请教些内核编程的问题。支持看雪！希望看雪越办越好！！！

#### 上传的附件

| 预览 | 文件名 |
| :--- | :--- |
| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) | `RuncmdasSystem.rar` |
| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) | `RuncmdasSystemStart.rar` |
