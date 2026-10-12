
# 【成果 3.3】驱动整个入门流程之流程四--驱动动态加载

> 原文：[`www.pediy.com/kssd/pediy10/57265.html`](https://www.pediy.com/kssd/pediy10/57265.html)

这个课题主要是讲如何动态加载驱动，虽然现在已经存在很多的动态驱动加载器，但是使用自己编写的东西，还是有不同的感觉的。当是科普吧。

在这里，主要使用 SCM 来进行驱动的动态加载和卸载。

先介绍下 SCM，服务控制管理器(Service Control Manager ， SCM )。服务(Service)是在后台运行配套的程序，并且不需要用户交互(也就是说没有常见的用户界面或者控制台)。换句话说，一个服务就是一个始终运行于系统中的 Win32 进程，即使没有用户登陆进来也如此。SC 管理器（即服务控制管理器）可以控制服务和驱动程序。

主要涉及的函数如下：

| 名称 | 描述 |
| :--- | :--- |
| `CloseServiceHandle` | 关闭来自 `OpenSCManager()` 、 `CreateService()` 或 `OpenService()` 的句柄 |
| `ControlService` | 停止、暂停、继续、查询或通知已加载的服务 / 驱动程序 |
| `CreateService` | 加载一个服务 / 驱动程序 |
| `DeleteService` | 卸载一个服务 / 驱动程序 |
| `OpenSCManager` | 获取 SC 管理器的句柄 |
| `OpenService` | 获取一个已加载的服务 / 驱动程序的句柄 |
| `QueryServiceStatus` | 查询一个服务 / 驱动程序的属性和当前状态 |
| `StartService` | 启动一个已加载的服务 / 驱动程序 |

加载和运行一个服务需要执行的典型操作步骤：

1. 调用 `OpenSCManager()` 以获取一个管理器句柄
2. 调用 `CreateService()` 来向系统中添加一个服务
3. 调用 `StartService()` 来运行一个服务
4. 调用 `CloseServiceHandle()` 来释放管理器或服务句柄

不过我这里决定把加载驱动和启动驱动的过程分开，这样容易看明白，如果想整合也很容易，直接调用两个函数就可以了。

### 一 打开 SCM

第一步需要做的是和 SCM 交互，则要使用 `OpenSCManager` 函数。原型如下：

```c
SC_HANDLE OpenSCManager(LPCTSTR lpMachineName,
　　　　　　　　　　　　 LPCTSTR lpDatabaseName,
　　　　　　　　　　　　 DWORD dwDesiredAccess);
```

*   `lpMachineName`：电脑名。如果为 `NULL`，则为本机。
*   `lpDatabaseName`：SCM 数据库标识，应该为 `SERVICES_ACTIVE_DATABASE` 或者如果为 `NULL`，则 `SEVICES_ACTIVE_DATABASE` 数据库作为默认数据库被打开。
*   `dwDesiredAccess`：以何种权限打开。

本函数返回一个 `SC_HANDLE` 的句柄以便作为其他函数的参数来操作 SCM 数据库。

当完成对 SCM 数据库的访问后，在退出前必须使用 `CloseServiceHandle` 关闭它。原型为：

```c
BOOL CloseServiceHandle(SC_HANDLE hSCManger);
```

### 二 创建服务

增加一个 Service，需要用到 `CreateService` 函数，原型如下：

```c
SC_HANDLE CreateService(SC_HANDLE hSCManager,
　　　　　　　　　　　 LPCTSTR lpServiceName,
　　　　　　　　　　　 LPCTSTR lpDisplayName,
　　　　　　　　　　　 DWORD dwDesireAccess,
　　　　　　　　　　　 DWORD dwServiceType,
　　　　　　　　　　　 DWORD dwStartType,
　　　　　　　　　　　 DWORD dwErrorControl,
　　　　　　　　　　　 LPCTSTR lpBinaryPathName,
　　　　　　　　　　　 LPCTSTR lpLoadOrderGroup,
　　　　　　　　　　　 LPDWORD lpdwTagId,
　　　　　　　　　　　 LPCTSTR lpDependencies,
　　　　　　　　　　　 LPCTSTR lpServiceStartName,
　　　　　　　　　　　 LPCTSTR lpPassword);
```

这个函数有十三个参数：

*   `hSCManager`：是 `OpenSCManager` 得到的返回值，即服务管理器的句柄。
*   `lpServiceName`：Service 的名字，同时注册表也使用这个名字，这个 Service 的相应信息存放在 `HKEY_LOCAL_MACHINE\\SYSTEM\\CurrentControlSet\\Service\\lpServiceName` 子键下；
*   `lpDisplayName`：Service 在服务控制面板中显示的名字；
*   `dwDesireAccess`：访问 Service 的权限类型；
*   `dwServiceType`：Service 的类型，可以是 `SERVICE_FILE_SYSTEM_DRIVER`、`SERVICE_KERNEL_DRIVER`、`SERVICE_WIN32_OWN_PROCESS`、`SERVICE_WIN32_SHARE_PROCESS` 之一；
*   `dwStartType`：Service 启动类型，可以是自动(`SERVICE_AUTO_START`)、手动(`SERVICE_DEMAND_START`)、禁止(`SERVICE_DISABLED`)；
*   `dwErrorControl`：出错控制，`SERVICE_ERROR_IGNORE` 和 `SERVICE_ERROR_NORMAL` 表示出错时在日志中纪录并继续。两个参数的区别是 `SERVICE_ERROR_NORMAL` 将会在出错时向用户显示一条出错信息，`SERVICE_ERROR_SEVERE` 和 `SERVICE_ERROR_CRITICAL` 告诉系统在失败时重启；
*   `lpBinaryPathName`：驱动的路径，也是服务的路径；
*   `lpLoadOrderGroup`：如果这个服务是加载服务组中的一个，这个指针指向这者字符串数组，一般为 `NULL`；
*   `lpdwTagId`：指针，指向的变量，接收的值用於区别 `lpLoadOrderGroup` 服务组中的不同服务，一般为 `NULL`；
*   `lpDependencies`：这个 Service 所依赖的其他服务，在启动本 Service 之前需要先启动这些服务；
*   `lpServiceStartName`：帐号；
*   `lpPassword`：密码。

需要注意的是，如果要增加一个 Service，那么 `OpenSCManager` 函数中的 `dwDesiredAccess` 参数至少需要为 `SC_MANAGER_CREATE_SERVICE`。

### 三 启动服务

启动 Service，相应函数如下：

```c
BOOL StartService(SC_HANDLE hService,
　　　　　　　　 DWORD dwNumServiceArgs,
　　　　　　　　 LPCTSTR *lpServiceArgVectors);
```

*   `hService`：Service 的句柄；
*   `dwNumServiceArgs`：指定在 `lpServiceArgVectors` 指向的数组中参数的个数，如果 `lpServiceArgVectors` 为 `NULL`，这个值为 0；
*   `lpServiceArgVectors`：指向传递给 Service 的参数字符串数组的指针，可以为 `NULL`；

Service 运行之后，`ControlService` 函数用于的控制，函数原型如下：

```c
BOOL ControlService(SC_HANDLE hService,
　　　　　　　　　 DWORD dwControl,
　　　　　　　　　 LPSERVICE_STATUS lpServiceStatus);
```

*   `hService`：Service 句柄；
*   `dwControl`：`SERVICE_CONTROL_STOP`，`SERVICE_CONTROL_PAUSE`，`SERVICE_CONTROL_CONTINUE`，`SERVICE_CONTROL_INTERROGATE` 之一，也可以自定义(128~255)；
*   `lpServiceStatus`：一个 `LPSERVICE_STATUS` 的结构指针。

如果想停止服务，也需要使用该函数来进行控制。

```c
BOOL QueryServiceStatus(SC_HANDLE hService,
　　　　　　　　　　　 LPSERVICE_STATUS lpServiceStatus);
```

调用 `QueryServiceStatus` 函数和调用带 `SERVICE_CONTROL_INTERROGATE` 参数的 `ControlService` 相差无几。

使用该函数进行服务状态的查询，比如判断服务是否已经启动，服务是否停止了。

好，下面就是代码了，我使用的是 delphi7，把这些功能写在一个类中，代码很容易明白。也就不多说了。

代码:
```c
// 代码部分在原文中未提供具体内容，仅保留“代码:”提示
```



```
unit untServiceMan;



interface



uses windows, WinSvc;



type
  TServiceMan = class
  private
    function ServiceConnect:SC_HANDLE;
    function ServiceDisconn(hService:SC_HANDLE):SC_HANDLE;



    function ServiceOpen(hService:SC_HANDLE; AServName:PChar):SC_HANDLE;
    function ServiceClose(hService:SC_HANDLE):Boolean;



    function ServiceGetStatus(hService:SC_HANDLE; AServName:PChar ):DWord;



    function ServiceUninstalled(hService:SC_HANDLE; AServName:PChar):Boolean;
    function ServiceRunning(hService:SC_HANDLE; AServName:PChar):Boolean;
    function ServiceStopped(hService:SC_HANDLE; AServName:PChar):Boolean;
  public
    function ServiceInstall(AServName, ADispName, AServPath:PChar):SC_HANDLE;
    function ServiceUnInstall(AServName:PChar):DWORD;
    function ServiceStart(AServName:PChar):DWORD;
    function ServiceStop(AServName:PChar):DWORD;
  end;
  
  implementation
{ TServiceMan }



{******************************************************************
**功能：装载服务                              *
**参数 1：服务名                             *
**参数 2：服务描述                            *
**参数 3：服务文件的地址                      *
******************************************************************}
function TServiceMan.ServiceInstall(AServName, ADispName, AServPath: PChar): SC_HANDLE;
var
  hManager:SC_HANDLE;
begin
  result := 0;



  hManager := ServiceConnect;



  try
    if ServiceUninstalled(hManager, AServName) then
    begin
      result := CreateService(hManager,
                              AServName,
                              ADispName,
                              SERVICE_ALL_ACCESS,
                              SERVICE_KERNEL_DRIVER,
                              SERVICE_DEMAND_START,
                              SERVICE_ERROR_NORMAL,
                              AServPath,
                              nil,
                              nil,
                              nil,
                              nil,
                              nil);



      ServiceClose(result);            //记住关闭句柄，否则会无法卸载服务
    end;
  finally
    ServiceDisconn(hManager);
  end;
end;



{******************************************************************
**功能：关闭打开的服务句柄                                 *
**参数 1：服务句柄                               *
******************************************************************}
function TServiceMan.ServiceClose(hService: SC_HANDLE): Boolean;
begin
  if (hService <> 0) then
    result := CloseServiceHandle(hService)
  else
    result := True;
end;



{******************************************************************
**功能：打开服务管理器                                 *
**返回值为其句柄                                 *
******************************************************************}
function TServiceMan.ServiceConnect: SC_HANDLE;
begin
  result := OpenSCManager(nil, nil, SC_MANAGER_ALL_ACCESS);
end;



{******************************************************************
**功能：关闭管理器句柄                                 *
**参数 1：服务句柄                               *
******************************************************************}
function TServiceMan.ServiceDisconn(hService: SC_HANDLE): SC_HANDLE;
begin
  if (hService <> 0) then
    CloseServiceHandle(hService);
  result := 0;
end;



{******************************************************************
**功能：获得一个已经加载的服务的句柄                         *
**参数 1：管理器句柄                                *
**参数 2：服务的名称                                *
******************************************************************}
function TServiceMan.ServiceOpen(hService:SC_HANDLE; AServName: PChar): SC_HANDLE;
begin
  result := OpenService(hService, AServName, SERVICE_ALL_ACCESS);
end;



{******************************************************************
**功能：卸载服务                                 *
**参数 1：服务的名称                              *
******************************************************************}
function TServiceMan.ServiceUnInstall(AServName:PChar): DWORD;
var
  hService, hManager:SC_HANDLE;
  ServiceStatus:TServiceStatus;
begin
  result := 0;



  hManager := ServiceConnect;



  try
    if not ServiceUninstalled(hManager, AServName) then
    begin
      hService := ServiceOpen(hManager, AServName);



      if (hService <> 0) then
      begin
        ControlService(hService, SERVICE_CONTROL_STOP, ServiceStatus);      //在卸载前停止服务，否则需要停止服务才能正常卸载
        if DeleteService(hService) then
          result := 2
        else
          result := 1;
      end;
      ServiceClose(hService);
    end;
  finally
    ServiceDisconn(hManager);
  end;
end;



{******************************************************************
**功能：启动服务                                 *
**参数 1：服务的名称                              *
**返回值：    0：驱动还未安装                     *
              1：驱动已启动                       *
              2：驱动启动成功                      *
              3：驱动启动失败                      *
******************************************************************}
function TServiceMan.ServiceStart(AServName: PChar): DWORD;
var
  hService, hManager:SC_HANDLE;
begin
  result := 0;



  hManager := ServiceConnect;



  try
    if not ServiceUninstalled(hManager, AServName) then
    begin
      if not ServiceRunning(hManager, AServName) then
      begin
        hService := ServiceOpen(hManager, AServName);



        if (hService <> 0) then
          if StartService(hService, 1, AServName) then         //启动服务
            result := 2
          else
            result := 3;



        ServiceClose(hService);
      end
      else
        result := 1;
    end;
  finally
    ServiceDisconn(hManager);
  end;
end;



{******************************************************************
**功能：停止服务                                 *
**参数 1：服务的名称                              *
**返回值：    0：驱动还未安装                     *
              1：驱动已停止                       *
              2：驱动停止成功                      *
              3：驱动停止失败                      *
******************************************************************}
function TServiceMan.ServiceStop(AServName: PChar): DWORD;
var
  hService, hManager:SC_HANDLE;
  ServiceStatus:TServiceStatus;
begin
  result := 0;



  hManager := ServiceConnect;



  try
    if not ServiceUninstalled(hManager, AServName) then
    begin
      if not ServiceStopped(hManager, AServName) then
      begin
        hService := ServiceOpen(hManager, AServName);



        if (hService <> 0) then
          if ControlService(hService, SERVICE_CONTROL_STOP, ServiceStatus) then
            result := 2
          else
            result := 3;



        ServiceClose(hService);
      end
      else
        result := 1;
    end;
  finally
    ServiceDisconn(hManager);
  end;
end;



{******************************************************************
**功能：获得服务的状态                                *
**参数 1：管理器的句柄                              *
**参数 2：服务的名称                              *
******************************************************************}
function TServiceMan.ServiceGetStatus(hService:SC_HANDLE; AServName:PChar ):DWORD;
var
  hService2:SC_HANDLE;
  queryStatus:TServiceStatus;
  dwStat:DWORD;
begin
  dwStat := 0;



  if(hService > 0)then
  begin
    hService2 := ServiceOpen(hService, AServName);



    if(QueryServiceStatus(hService2, queryStatus))then
    begin
      dwStat := queryStatus.dwCurrentState;
    end;
    
    ServiceClose(hService2);
  end;
  Result := dwStat;
end;



{判断某服务是否安装，未安装返回 true，已安装返回 false} 
function TServiceMan.ServiceUninstalled(hService:SC_HANDLE; AServName:PChar):Boolean;
begin 
  Result  :=  0 = ServiceGetStatus(hService, AServName);
end; 



{判断某服务是否启动，启动返回 true，未启动返回 false} 
function TServiceMan.ServiceRunning(hService:SC_HANDLE; AServName:PChar):Boolean;
begin 
  Result := SERVICE_RUNNING = ServiceGetStatus(hService, AServName);
end; 



{判断某服务是否停止，停止返回 true，未停止返回 false} 
function TServiceMan.ServiceStopped(hService:SC_HANDLE; AServName:PChar):Boolean;
begin 
  Result := SERVICE_STOPPED = ServiceGetStatus(hService, AServName);
end;



end.


```



上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | | 驱动安装.rar |
| :--- | ---: | :--- |
