
> 驱动在调用过程中使用的`InsDrv.exe`工具实现安装驱动、开启驱动、停止驱动和卸载驱动，但是在正常的执行过程中，上述步骤是放在代码中来实现的。

```c
#include <winsvc.h>

// 安装驱动

BOOL installDvr(CONST WCHAR drvPath[50], CONST WCHAR serviceName[20]) {
 // 打开服务控制管理器数据库

 SC_HANDLE schSCManager = OpenSCManager(
  NULL, // 目标计算机的名称,NULL：连接本地计算机上的服务控制管理器
  NULL, // 服务控制管理器数据库的名称，NULL：打开 SERVICES_ACTIVE_DATABASE 数据库
  SC_MANAGER_ALL_ACCESS // 所有权限
 );

 if (schSCManager == NULL) {
  CloseServiceHandle(schSCManager);
  return FALSE;
 }

 // 创建服务对象，添加至服务控制管理器数据库

 SC_HANDLE schService = CreateService(
  schSCManager, // 服务控件管理器数据库的句柄
  serviceName, // 要安装的服务的名称
  serviceName, // 用户界面程序用来标识服务的显示名称
  SERVICE_ALL_ACCESS, // 对服务的访问权限：所有全权限
  SERVICE_KERNEL_DRIVER, // 服务类型：驱动服务
  SERVICE_DEMAND_START, // 服务启动选项：进程调用 StartService 时启动
  SERVICE_ERROR_IGNORE, // 如果无法启动：忽略错误继续运行
  drvPath, // 驱动文件绝对路径，如果包含空格需要多加双引号
  NULL, // 服务所属的负载订购组：服务不属于某个组
  NULL, // 接收订购组唯一标记值：不接收
  NULL, // 服务加载顺序数组：服务没有依赖项
  NULL, // 运行服务的账户名：使用 LocalSystem 账户
  NULL // LocalSystem 账户密码
 );

 if (schService == NULL) {
  CloseServiceHandle(schService);
  CloseServiceHandle(schSCManager);
  return FALSE;
 }

 CloseServiceHandle(schService);
 CloseServiceHandle(schSCManager);
 return TRUE;
}

// 启动服务

BOOL startDvr(CONST WCHAR serviceName[20]) {
 // 打开服务控制管理器数据库

 SC_HANDLE schSCManager = OpenSCManager(
  NULL, // 目标计算机的名称,NULL：连接本地计算机上的服务控制管理器
  NULL, // 服务控制管理器数据库的名称，NULL：打开 SERVICES_ACTIVE_DATABASE 数据库
  SC_MANAGER_ALL_ACCESS // 所有权限
 );

 if (schSCManager == NULL) {
  CloseServiceHandle(schSCManager);
  return FALSE;
 }

 // 打开服务

 SC_HANDLE hs = OpenService(
  schSCManager, // 服务控件管理器数据库的句柄
  serviceName, // 要打开的服务名
  SERVICE_ALL_ACCESS // 服务访问权限：所有权限
 );

 if (hs == NULL) {
  CloseServiceHandle(hs);
  CloseServiceHandle(schSCManager);
  return FALSE;
 }

 if (StartService(hs, 0, 0) == 0) {
  CloseServiceHandle(hs);
  CloseServiceHandle(schSCManager);
  return FALSE;
 }

 CloseServiceHandle(hs);
 CloseServiceHandle(schSCManager);
 return TRUE;
}

// 停止服务

BOOL stopDvr(CONST WCHAR serviceName[20]) {
 // 打开服务控制管理器数据库

 SC_HANDLE schSCManager = OpenSCManager(
  NULL, // 目标计算机的名称,NULL：连接本地计算机上的服务控制管理器
  NULL, // 服务控制管理器数据库的名称，NULL：打开 SERVICES_ACTIVE_DATABASE 数据库
  SC_MANAGER_ALL_ACCESS // 所有权限
 );

 if (schSCManager == NULL) {
  CloseServiceHandle(schSCManager);
  return FALSE;
 }

 // 打开服务

 SC_HANDLE hs = OpenService(
  schSCManager, // 服务控件管理器数据库的句柄
  serviceName, // 要打开的服务名
  SERVICE_ALL_ACCESS // 服务访问权限：所有权限
 );

 if (hs == NULL) {
  CloseServiceHandle(hs);
  CloseServiceHandle(schSCManager);
  return FALSE;
 }

 // 如果服务正在运行

 SERVICE_STATUS status;

 if (QueryServiceStatus(hs, &status) == 0) {
  CloseServiceHandle(hs);
  CloseServiceHandle(schSCManager);
  return FALSE;
 }

 // typedef struct _SERVICE_STATUS {
 //  /* 服务指向文件的类型 */
 // DWORD dwServiceType;
 //  /* 用于通知SCM(服务控制管理器)此服务的现行状态 */
 // DWORD dwCurrentState;
 //  /* 指明服务接受什么样的控制通知 */
 // DWORD dwControlsAccepted;
 //  /* 服务用于报告启动或停止时发生的错误的错误代码 */
 // DWORD dwWin32ExitCode;
 //  /* 服务特定的错误代码，服务在启动或停止时发生错误时，该服务将返回 */
 // DWORD dwServiceSpecificExitCode;
 //  /* 服务的检查点值会定期增加，以在长时间的启动，停止，暂停或继续操作期间报告其进度 */
 // DWORD dwCheckPoint;
 //  /* 挂起的启动，停止，暂停或继续操作所需的估计时间，以毫秒为单位 */
 // DWORD dwWaitHint;
 // } SERVICE_STATUS, *LPSERVICE_STATUS;

 if (status.dwCurrentState != SERVICE_STOPPED &&
  status.dwCurrentState != SERVICE_STOP_PENDING
 ) {
  // 发送关闭服务请求

  if (ControlService(
   hs, // 服务句柄
   SERVICE_CONTROL_STOP, // 控制码：通知服务应该停止
   &status // 接收最新的服务状态信息
  ) == 0) {
   CloseServiceHandle(hs);
   CloseServiceHandle(schSCManager);
   return FALSE;
  }

  // 判断超时

  INT timeOut = 0;
  while(status.dwCurrentState != SERVICE_STOPPED){
   timeOut++;
   QueryServiceStatus(hs, &status);
   Sleep(50);
  }

  if (timeOut > 80) {
   CloseServiceHandle(hs);
   CloseServiceHandle(schSCManager);
   return FALSE;
  }
 }

 CloseServiceHandle(hs);
 CloseServiceHandle(schSCManager);
 return TRUE;
}

// 卸载驱动

BOOL unloadDvr(CONST WCHAR serviceName[20]) {
 // 打开服务控制管理器数据库

 SC_HANDLE schSCManager = OpenSCManager(
  NULL, // 目标计算机的名称,NULL：连接本地计算机上的服务控制管理器
  NULL, // 服务控制管理器数据库的名称，NULL：打开 SERVICES_ACTIVE_DATABASE 数据库
  SC_MANAGER_ALL_ACCESS // 所有权限
 );

 if (schSCManager == NULL) {
  CloseServiceHandle(schSCManager);
  return FALSE;
 }

 // 打开服务

 SC_HANDLE hs = OpenService(
  schSCManager, // 服务控件管理器数据库的句柄
  serviceName, // 要打开的服务名
  SERVICE_ALL_ACCESS // 服务访问权限：所有权限
 );

 if (hs == NULL) {
  CloseServiceHandle(hs);
  CloseServiceHandle(schSCManager);
  return FALSE;
 }

 // 删除服务

 if (DeleteService(hs) == 0) {
  CloseServiceHandle(hs);
  CloseServiceHandle(schSCManager);
  return FALSE;
 }

 CloseServiceHandle(hs);
 CloseServiceHandle(schSCManager);
 return TRUE;
}
```

# 驱动学习7--SSDT HOOK(x86 架构下)

> SSDT 的全称是`System Services Descriptor Table`，系统服务描述表。这个表就是一个把`Ring3`的`Win32 API`和`Ring0`内核`API`联系起来的。

`SSDT`并不仅仅只包含一个庞大的地址索引表，它还包含一些其他应用的信息，如：地址索引的基址、服务函数个数等。

通过修改此表的函数地址可以对常用的`Windows`函数及`API`进行`HOOK`，从而实现对一些关心的系统动作进行过滤、监控的目的。一些`HIPS`（主机入侵防御系统）、防毒软件、系统监控、注册表监控软件往往会采用此接口来实现自己的监控模块。

`SSDT`即系统服务描述表，它的结构如下：

```c
// KSYSTEM_SERVICE_TABLE 和 KSERVICE_TABLE_DESCRIPTOR
// 用来定义 SSDT 结构

typedef struct _KSYSTEM_SERVICE_TABLE
{
 PULONG ServiceTableBase; // SSDT (System Service Dispatch Table)的基地址
 PULONG ServiceCounterTableBase; // 用于 checked builds, 包含 SSDT 中每个服务被调用的次数
 ULONG NumberOfService; // 服务函数的个数, NumberOfService \* 4 就是整个地址表的大小
 ULONG ParamTableBase; // SSPT(System Service Parameter Table)的基地址
} KSYSTEM_SERVICE_TABLE, *PKSYSTEM_SERVICE_TABLE;

typedef struct _KSERVICE_TABLE_DESCRIPTOR
{
 KSYSTEM_SERVICE_TABLE ntoskrnl; // ntoskrnl.exe 的服务函数
 KSYSTEM_SERVICE_TABLE win32k; // win32k.sys 的服务函数(GDI32.dll/User32.dll 的内核支持)
 KSYSTEM_SERVICE_TABLE notUsed1;
 KSYSTEM_SERVICE_TABLE notUsed2;
}KSERVICE_TABLE_DESCRIPTOR, *PKSERVICE_TABLE_DESCRIPTOR;
```

> 内核中有两个系统服务描述符表,一个是`KeServiceDescriptorTable`（由`ntoskrnl.exe`导出）,一个是`KeServieDescriptorTableShadow`（没有导出）。

两者的区别是，`KeServiceDescriptorTable`仅有`ntoskrnel`一项，`KeServieDescriptorTableShadow`包含了`ntoskrnel`以及`win32k`。一般的`Native API`（原生`API`）的服务地址由`KeServiceDescriptorTable`分派，`gdi.dll/user.dll`的内核`API`调用服务地址由`KeServieDescriptorTableShadow`分派。还有要清楚一点的是`win32k.sys`只有在`GUI`线程中才加载，一般情况下是不加载的，所以要`Hook KeServieDescriptorTableShadow`的话，一般是用一个`GUI`程序通过`IoControlCode`来触发(想当初不明白这点，蓝屏死机了N次都想不明白是怎么回事)。（此段摘抄自网络）

内核`HOOK`分类：

![图片描述](upload/attach/202102/889500_ZUY27RSZM3SP8GK.png)

#### SSDT HOOK 原理：

1. 首先是关于内核函数地址是怎么计算的。这里以`NtOpenProcess`为例：

首先是要`IDA`加载`ntdll.dll`， 查找`NtOpenProcess`查看函数对应的`SystemId`，这个值可以理解为地址表内的一个下标。

![图片描述](upload/attach/202102/889500_JFCG858PQMNRHAD.png)

2. 查看`KeServiceDescriptorTable`（服务描述表）：

是要`WinDbg`调试本地内核或者双调试，执行命令

```
dd KeServiceDescriptorTable
```

![图片描述](upload/attach/202102/889500_U2BT78UEUBQA2SV.png)

所得到的`0x83EBDD9C`值就指向函数表首地址的指针，如下：

![图片描述](upload/attach/202102/889500_E9KCHTHXTGPR4VR.png)

3. 利用函数表首地址和`SystemId`（下标）计算出目标函数 (`NtOpenProcess`)函数对应的地址。

```
pKernelFuncAddress = [[KeServiceDescriptorTable] + SystenId*0x4]
```

![图片描述](upload/attach/202102/889500_3C5TFAASZMXZP95.png)

上述就是计算`NtOpenProcess`函数地址的方式。那么样实现`HOOK`该函数，就是将最后得出的地址`0x840529DC`替换成自己的函数地址。

### 参考文章：

[https://bbs.pediy.com/thread-40832.htm](https://bbs.pediy.com/thread-40832.htm)

> 实现对`SSDT`表的修改，首先要确保这个表示可写的。可以通过以下3种方法实现对`SSDT`的内存保护机制的修改。

修改方式：

1. 修改注册表

恢复页面保护：`HKLM\SYSTEM\CurrentControlset\Control\Session Manger\Memory Management\EnforceWriteProtection=0`

去掉页面保护：`HKLM\SYSTEM\CurrentControlset\Control\Session Manger\Memory Management\DisablePagingExecutive=1`

>2. 改变`CR0`寄存器的第一位

`Windows`对内存的分配，是采用的分页管理。其中有个`CR0`寄存器，如下图：

![图片描述](upload/attach/202102/889500_9534C3FZVRFDZZG.png)

其中第一位叫做保护属性位，控制着页的读写属性。如果为1，则页面可进行读写和执行，如果为0，则只能进行读和执行操作。`SSDT`，`IDT`的页属性默认情况下是只读可执行的。

```c
//设置为不可写
void DisableWrite()
{
 __try
 {
  __asm
  {
   mov eax, cr0
   or eax, 10000h
   mov cr0, eax
   sti
  }
 }
 __except(1)
 {
  KdPrint(("DisableWrite执行失败！"));
 }
}

// 设置为可写

void EnableWrite()
{
 __try
 {
  __asm
  {
   cli
   mov eax,cr0
   and eax,not 10000h //and eax,0FFFEFFFFh
   mov cr0,eax
  }
 }
 __except(1)
 {
  KdPrint(("EnableWrite执行失败！"));
 }
}

//设置为不可写
void DisableWrite()
{
 __try
 {
  __asm
  {
   mov eax, cr0
   or eax, 10000h
   mov cr0, eax
   sti
  }
 }
 __except(1)
 {
  KdPrint(("DisableWrite执行失败！"));
 }
}
```
