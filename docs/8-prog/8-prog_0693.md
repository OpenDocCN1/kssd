
# Delphi 研究之驱动开发篇（六）--利用 Section 与用户模式程序通讯

> 原文：[`www.pediy.com/kssd/pediy10/66291.html`](https://www.pediy.com/kssd/pediy10/66291.html)

在进入主题之前，先来简单地看一下结构化异常处理（Structured Exception Handling, SEH）,本篇的程序需要这个东东。

## 结构化异常处理

这里我并不打算详细讲结构化异常处理，关于 SEH，在网上你能找到相关的内容，SHE 能用于所有的异常处理，也就是说，SEH 既能用于用户模式又能用于内核模式。但这两种模式下的异常处理有一个本质上的差别：

在内核模式下，借助于 `seh`，并非所有的异常都能得到处理！比如说，即使使用了 `seh`，用零作除数作除法也会使系统崩溃。最为可怕的是，引用未定义的内核内存也会导致蓝屏死机 `BSOD`。而对未定义的用户模式内存的引用异常，`seh` 却可以轻松处理。因此避免系统崩溃的唯一办法就是所编写的代码不要导致无法处理的异常。

以下是个使用结构化异常的例子：

代码:

```
unit seh;

interface

uses
  nt_status;

function _DriverEntry(pDriverObject:PDRIVER_OBJECT;
                      pusRegistryPath:PUNICODE_STRING): NTSTATUS; stdcall;   

implementation

uses
  ntoskrnl;

const
  SEH_SafePlaceCounter  = 0;
  SEH_INSTALLED         = 0;

type
  _SEH = record
    SafeEip: DWORD; { 线程继续执行的地方 }
    PrevEsp: DWORD; { 以前 esp 的值 }
    PrevEbp: DWORD; { 以前 ebp 的值 }
  end;

var
  sseh: _SEH;

function DefaultExceptionHandler(pExcept:PEXCEPTION_RECORD;
                                 pFrame:DWORD;
                                 pContext:PCONTEXT;
                                 pDispatch:DWORD): DWORD; cdecl;
begin
  DbgPrint(#13#10'SEH: An exception %08X has occured'#13#10,
           pExcept^.ExceptionCode);
  if pExcept^.ExceptionCode = $0C0000005 then
  begin
    {如果发生了 EXCEPTION_ACCESS_VIOLATION 类型的异常,}
    {则输出以下信息.}
    DbgPrint(' Access violation at address: %08X'#13#10,
             pExcept^.ExceptionAddress);
    if pExcept^.ExceptionInformation[0] <> nil then  {试图读还是写?}
    begin
      DbgPrint(' The code tried to write to address %08X'#13#10#13#10,
               DWORD(pExcept^.ExceptionInformation[4]));
    end else
    begin
      DbgPrint(' The code tried to read from address %08X'#13#10#13#10,
               DWORD(pExcept^.ExceptionInformation[4]));
    end;
  end;
  asm
    lea eax, sseh
    push (_SEH PTR [eax]).SafeEip
    push (_SEH PTR [eax]).PrevEsp
    push (_SEH PTR [eax]).PrevEbp

    mov eax, pContext
    pop (CONTEXT PTR [eax]).regEbp
    pop (CONTEXT PTR [eax]).regEsp
    pop (CONTEXT PTR [eax]).regEip
  end;
  result := 0;
end;

procedure BuggyReader; assembler;
asm
  xor eax, eax
  mov eax, [eax]  {!!! 没有 SEH 的话 - BSOD !!!}
end;

procedure BuggyWriter; assembler;
asm
  mov eax, offset MmUserProbeAddress
  mov eax, [eax]
  mov eax, [eax]

  mov byte ptr [eax], 0 {!!!没有 SEH 的话 - BSOD !!!}
end;

function _DriverEntry(pDriverObject:PDRIVER_OBJECT;
                      pusRegistryPath:PUNICODE_STRING): NTSTATUS; stdcall;
label
  SafePlace;
begin
  DbgPrint(#13#10'SEH: Entering DriverEntry'#13#10);
  { "手工"安装 SEH }
  asm
    push offset DefaultExceptionHandler  {我们的 SEH 程序}
    push fs:[0]
    mov fs:[0], esp

    mov sseh.SafeEip, offset SafePlace  {SafePlace 是处理完异常后继续执行的地方}
    mov sseh.PrevEbp, ebp
    mov sseh.PrevEsp, esp
  end;
  BuggyReader;
  BuggyWriter;

SafePlace:
  asm
    pop fs:[0]
    add esp, 4
  end;
   
  DbgPrint(#13#10'SEH: Leaving DriverEntry'#10#13);
  result := STATUS_DEVICE_CONFIGURATION_ERROR;
end;

end.
```

### 安装 SHE-Frame

由于在内核模式下我们无法直接使用 Delphi 自身的异常处理机制，因为在驱动程序中我们要自己手工安装 `SHE`，这里我们使用 Delphi 的 `BASM` 来做这件事情，了解 `SHE` 的朋友都知道，做这件事情是非常简单的。

代码:

```
asm
    push offset DefaultExceptionHandler  {我们的 SEH 程序}
    push fs:[0]
    mov fs:[0], esp

    mov sseh.SafeEip, offset SafePlace  {SafePlace 是处理完异常后继续执行的地方}
    mov sseh.PrevEbp, ebp
mov sseh.PrevEsp, esp
end;
```

为了在异常处理之后我们的处理程序能恢复线程的执行，我们应该保存 `esp`、`ebp` 寄存器的内容以及线程继续执行的地址。我们将这三项信息保存在 `seh` 结构体中并调用函数 `BuggyReader()`。`BuggyReader()` 函数试图从地址 `00000000` 读取一个 `DWORD`。

代码:

```
procedure BuggyReader; assembler;
asm
  xor eax, eax
  mov eax, [eax]  {!!! 没有 SEH 的话 - BSOD !!!}
end;
```

`nil` 指针引用是一个十分常见的错误，微软在 `00000000-0000FFFFh` 划出了 `64k` 字节的内存区，并使此区域无法访问。访问此区域中的任何一个字节都会导致 `EXCEPTION_ACCESS_VIOLATION` 类型的异常。

### 异常处理

函数 `BuggyReader()` 从地址 `00000000` 读取引发了异常，我们就进入了我们指定的处理程序。

代码:

```
function DefaultExceptionHandler(pExcept:PEXCEPTION_RECORD;
                                 pFrame:DWORD;
                                 pContext:PCONTEXT;
                                 pDispatch:DWORD): DWORD; cdecl;
begin
DbgPrint(#13#10'SEH: An exception %08X has occured'#13#10,
           pExcept^.ExceptionCode);
  if pExcept^.ExceptionCode = $0C0000005 then
  begin
    {如果发生了 EXCEPTION_ACCESS_VIOLATION 类型的异常,}
    {则输出以下信息.}
    DbgPrint(' Access violation at address: %08X'#13#10,
             pExcept^.ExceptionAddress);
    if pExcept^.ExceptionInformation[0] <> nil then  {试图读还是写?}
    begin
      DbgPrint(' The code tried to write to address %08X'#13#10#13#10,
               DWORD(pExcept^.ExceptionInformation[4]));
    end else
    begin
      DbgPrint(' The code tried to read from address %08X'#13#10#13#10,
               DWORD(pExcept^.ExceptionInformation[4]));
    end;
  end;
  asm
    lea eax, sseh
    push (_SEH PTR [eax]).SafeEip
    push (_SEH PTR [eax]).PrevEsp
    push (_SEH PTR [eax]).PrevEbp

    mov eax, pContext
    pop (CONTEXT PTR [eax]).regEbp
    pop (CONTEXT PTR [eax]).regEsp
    pop (CONTEXT PTR [eax]).regEip
  end;
  result := 0;
end;
```

我们处理的第一件事就是输出相应的调试信息，如果发生的是 `EXCEPTION_ACCESS_VIOLATION` 类型的异常，还要输出一些额外的信息。之后开始真正的异常处理（这里了使用了 `BASM`，你可以从中体会到 `BASM` 的强大功能）。

代码:

```
asm
    lea eax, sseh
    push (_SEH PTR [eax]).SafeEip
    push (_SEH PTR [eax]).PrevEsp
    push (_SEH PTR [eax]).PrevEbp

    mov eax, pContext
    pop (CONTEXT PTR [eax]).regEbp
    pop (CONTEXT PTR [eax]).regEsp
    pop (CONTEXT PTR [eax]).regEip
end;
```

在本例中异常处理只是简单地恢复 `esp`、`ebp` 寄存器的值并将 `eip` 寄存器的值置为一个能使线程安全地继续执行的地址。这些处理信息都是由系统从我们预先填充的 `seh` 结构体中取出并保存在了 `CONTEXT` 结构体中，`CONTEXT` 结构体的指针又传递给了系统。

最后异常处理函数返回 `ExceptionContinueExecution`，该值为零就是告诉系统应该恢复线程的上下文并继续执行。即 `eip` 的值等于标记 `SafePlace` 的地址，而 `esp` 和 `ebp` 寄存器的值恢复为原值，线程从标记 `SafePlace` 处继续其执行。还有一点要注意，异常处理程序是 `C` 调用约定的。

有了结构化异常处理的知识后，让我们继续我们的内核之旅。接下来的例子中我们要从驱动程序中转到用户模式内存里。这个转换最好能包含在 `SEH-frame` 里。

### 内存共享

Windows 提供了许多机制来进行进程间通讯（Interprocess Communications, IPC）:通讯缓冲、DDE、通讯窗口（`WM_COPYDATA` 就在这里）、邮槽(mailslot)、`sockets` 等等。所有这些机制都是基于文件映射对象（file-mapping object）的，该对象本身是一块两个或多个进程可以访问的内存区，用 DDK 的术语，映射文件就是 `section` 对象，不要把它和 PE 文件中的 `section` 混淆起来。

`section` 对象是最底层的通讯机制，这种对象被系统用来将可执行映象加载到内存，而缓存调度程序用它来访问缓存文件中的数据。`section` 对象还能将磁盘上的文件映射到进程的地址空间中，而且用起来不像是在用文件，而是在用内存块。

借助于 `section` 对象来共享数据的情形如下：一个进程调用函数 `CreateFileMapping()` 创建了一个内存映射文件。之后调用函数 `MapViewOfFile()`（如果层次更低就调用 `NtMapViewOfSection()`）将其视图（view）映射到自己的地址空间中，而另一个进程通过 `OpenFileMapping()` 打开这个映射文件，并将其映射到自己的地址空间中。结果同一组物理内存页变为由两个进程访问，这就使得它们能通过这个区域轻松地传递较大量的数据，一个进程对这些页内容的修改会反映到另一个进程中。

共享 `section` 这种通讯方法不止可以用在用户进程间，还可以用在驱动程序里。在下面的例子里我们用命名 `section` 来在用户进程和驱动程序之间进行通讯。

老规矩，先来看看驱动程序。

代码:



```delphi
unit SharedSection;

interface

uses
  nt_status, ntoskrnl, native, winioctl, fcall, macros;

function _DriverEntry(pDriverObject:PDRIVER_OBJECT; pusRegistryPath:PUNICODE_STRING): NTSTATUS; stdcall;

implementation

uses
  seh;

const
  SECTION_SIZE           = $1000;

var
  g_usDeviceName, g_usSymbolicLinkName, g_usSectionName: UNICODE_STRING;

function DispatchCreateClose(p_DeviceObject:PDEVICE_OBJECT; p_Irp:PIRP): NTSTATUS; stdcall;
begin
  p_Irp^.IoStatus.Status := STATUS_SUCCESS;
  p_Irp^.IoStatus.Information := 0;

  IofCompleteRequest(p_Irp, IO_NO_INCREMENT);
  result := STATUS_SUCCESS;
end;

function DispatchControl(p_DeviceObject: PDEVICE_OBJECT; p_Irp:PIRP): NTSTATUS; stdcall;
label
  SafePlace;
var
  status:NTSTATUS;
  IOCTL_SHARE_MY_SECTION: DWORD;
  psl:PIO_STACK_LOCATION;
  oa:OBJECT_ATTRIBUTES;
  hSection:HANDLE;
  pSectionBaseAddress:PVOID;
  liViewSize:LARGE_INTEGER;
begin
  IOCTL_SHARE_MY_SECTION := CTL_CODE(FILE_DEVICE_UNKNOWN, $800, 0, 0);
  psl := IoGetCurrentIrpStackLocation(p_Irp); {取 IRP 的 stack location 的指针}
  if psl^.Parameters.DeviceIoControl.IoControlCode = IOCTL_SHARE_MY_SECTION then
  begin
    {是我们控制码就开始处理}
    DbgPrint('SharedSection: Opening section object'#10#13);
    RtlInitUnicodeString(g_usSectionName, '\\BaseNamedObjects\\UserKernelSharedSection');
    InitializeObjectAttributes(oa, @g_usSectionName,
                               OBJ_CASE_INSENSITIVE, 0, nil);
    status := ZwOpenSection(@hSection,
                            SECTION_MAP_WRITE or SECTION_MAP_READ,
                            @oa);
    if status = STATUS_SUCCESS then
    begin
      DbgPrint('SharedSection: Section object opened'#13#10);
      pSectionBaseAddress := nil;
      liViewSize.HighPart := 0;
      liViewSize.LowPart := 0;
      status := ZwMapViewOfSection(hSection, HANDLE(NtCurrentProcess),
                                   pSectionBaseAddress, 0,
                                   SECTION_SIZE, nil,
                                   @liViewSize, ViewShare, 0,
                                   PAGE_READWRITE);
      if status = STATUS_SUCCESS then
      begin
        DbgPrint('SharedSection: Section mapped at address %08X'#13#10,
                 pSectionBaseAddress);
        {安装 SEH}
        asm
          push offset DefaultExceptionHandler
          push fs:[0]
          mov fs:[0], esp

          mov sseh.SafeEip, offset SafePlace
          mov sseh.PrevEbp, ebp
          mov sseh.PrevEsp, esp
        end;
        _strrev(pSectionBaseAddress);
        p_Irp^.IoStatus.Status := STATUS_SUCCESS;
        DbgPrint('SharedSection: String reversed'#13#10);
SafePlace:
        asm
          pop fs:[0]
          add esp, 4
        end;
        ZwUnmapViewOfSection(HANDLE(NtCurrentProcess), pSectionBaseAddress);
        DbgPrint('SharedSection: Section at address %08X unmapped '#13#10,
                 pSectionBaseAddress);
      end else
      begin
        DbgPrint('SharedSection: Couldn''t map view of section. Status: %08X'#13#10,
                 status);
      end;
      ZwClose(hSection);
      DbgPrint('SharedSection: Section object handle closed'#13#10);
    end else
    begin
      DbgPrint('SharedSection: Couldn''t open section. Status: %08X'#13#10, status);
    end;
  end else
  begin
    status := STATUS_INVALID_DEVICE_REQUEST;
  end;
  p_Irp^.IoStatus.Status := status;
  IofCompleteRequest(p_Irp, IO_NO_INCREMENT);
  DbgPrint('SharedSection: Leaving DispatchControl'#13#10);
  result := status;
end;

{卸载驱动}
procedure DriverUnload(p_DriverObject:PDRIVER_OBJECT); stdcall;
begin
  IoDeleteSymbolicLink(@g_usSymbolicLinkName);
  IoDeleteDevice(p_DriverObject^.DeviceObject);
end;

{驱动进入点}
function _DriverEntry(pDriverObject:PDRIVER_OBJECT; pusRegistryPath:PUNICODE_STRING): NTSTATUS;
var
  status: NTSTATUS;
  pDeviceObject: TDeviceObject;
begin
  status := STATUS_DEVICE_CONFIGURATION_ERROR;
  RtlInitUnicodeString(g_usDeviceName, '\\Device\\SharedSection');
  RtlInitUnicodeString(g_usSymbolicLinkName, '\\DosDevices\\SharedSection');

  if IoCreateDevice(pDriverObject, 0, @g_usDeviceName,
                    FILE_DEVICE_UNKNOWN, 0, TRUE,
                    pDeviceObject) = STATUS_SUCCESS then
  begin
    if IoCreateSymbolicLink(@g_usSymbolicLinkName,
                            @g_usDeviceName) = STATUS_SUCCESS then
    begin
      pDriverObject^.MajorFunction[IRP_MJ_CREATE] := @DispatchCreateClose;
      pDriverObject^.MajorFunction[IRP_MJ_CLOSE] := @DispatchCreateClose;
      pDriverObject^.MajorFunction[IRP_MJ_DEVICE_CONTROL] := @DispatchControl;
      pDriverObject^.DriverUnload := @DriverUnload;
      status := STATUS_SUCCESS;
    end else
    begin
      IoDeleteDevice(@pDeviceObject);
    end;
  end;
  result := status;
end;

end.
```

整个驱动程序非常简单，但它却是个完整意义上的驱动程序，包含了一个驱动程序所必须的各个部分。程序很简单，大家一看就明白，所以也没加什么注释，只捡几个重要的地方说明一下。

使用共享资源通常情况下需要考虑同步问题，即读写线程不能同时访问共享资源。在本例中总是只有一个线程，所以不需要同步。

代码：

```delphi
InitializeObjectAttributes(oa, @g_usSectionName,
                          OBJ_CASE_INSENSITIVE, 0, nil);
```

取得控制代码 `IOCTL_SHARE_MY_SECTION`，驱动程序尝试打开 section 对象，其名字定义为变量 `g_usSectionName`。

代码：

```delphi
status := ZwOpenSection(@hSection,
                        SECTION_MAP_WRITE or SECTION_MAP_READ,
                        @oa);
if status = STATUS_SUCCESS then
begin
  DbgPrint('SharedSection: Section object opened'#13#10);
  pSectionBaseAddress := nil;
  liViewSize.HighPart := 0;
  liViewSize.LowPart := 0;
  status := ZwMapViewOfSection(hSection, HANDLE(NtCurrentProcess),
                               pSectionBaseAddress, 0,
                               SECTION_SIZE, nil,
                               @liViewSize, ViewShare, 0,
                               PAGE_READWRITE);
end;
```

如果取得了 section 的句柄，我们就来映射它的视图。这里与应用程序中的映射视图部分几乎完全相同。但是在调用了 `ZwMapViewOfSection` 之后，变量 `pSectionBaseAddress` 保存的值位于用户地址区域中，而不是核心区域中的地址，这点我们可以预料到。这可是个原则上的问题，即对这个地址的使用将只能在这个进程的上下文中，在映射 section 的地址空间中。由于 `SharedSection` 驱动程序是单层的（您还记得 `IRP_MJ_DEVICE_CONTROL` 类型的 IRP 处理要经过驱动程序进入发出该操作的线程上下文中），所以我们位于我们的应用程序的上下文中。

这里视图的虚拟地址与应用程序中视图的地址将有所不同，但共享 section 所在的物理页是同一个。我们这里有一个内存页，其中还保存着倒着写的一行文字。

代码：

```delphi
asm
  push offset DefaultExceptionHandler
  push fs:[0]
  mov fs:[0], esp

  mov sseh.SafeEip, offset SafePlace
  mov sseh.PrevEbp, ebp
  mov sseh.PrevEsp, esp
end;
_strrev(pSectionBaseAddress);
p_Irp^.IoStatus.Status := STATUS_SUCCESS;
DbgPrint('SharedSection: String reversed'#13#10);
SafePlace:
asm
  pop fs:[0]
  add esp, 4
end;
```

以上建立 SEH-frame，并调用 `_strrev` 函数将内存里的字符串反转过来。

下面来看看在用户模式下如何加载和调用这个驱动程序。

代码：



```
program SharedSection;

{$APPTYPE CONSOLE}

uses
  SysUtils,
  Windows,
  Dialogs,
  WinSvc,
  nt\_status,
  native,
  macros,
  ntdll;

const
  SECTION\_SIZE = $1000;
  str = '.revird ecived a dna sessecorp resu neewteb yromem erahs ot euqinhcet emas eht esu nac uoy ,revewO .sessecorp resu gnoma yromem gnirahs rof desu euqinhcet nommoc a si elif gnigap eht yb dekcab elif deppam-yromem A';
  \_DELETE = $10000;

var
  hSection:HANDLE;
  liSectionSize: LARGE\_INTEGER;
  oa:OBJECT\_ATTRIBUTES;
  pSectionBaseAddress:PVOID;
  liViewSize: LARGE\_INTEGER;
  g\_usSectionName: UNICODE\_STRING;
  status:NTSTATUS;
  sTemp: array\[0..255\] of char;

function CallDriver: boolean;
var
  fOk: boolean;
  hSCManager:HANDLE;
  hService:HANDLE;
  acModulePath: string;
  \_ss:SERVICE\_STATUS;
  hDevice:HANDLE;

  dwBytesReturned: DWORD;
  IOCTL\_SHARE\_MY\_SECTION: DWORD;
  lpTemp: PChar;
begin
  fOk := false;
  IOCTL\_SHARE\_MY\_SECTION := CTL\_CODE(FILE\_DEVICE\_UNKNOWN, $800, 0, 0);
  hSCManager := OpenSCManager(nil, nil, SC\_MANAGER\_ALL\_ACCESS);
  if hSCManager <> 0 then
  begin
    acModulePath := GetCurrentDir + '\\' + ExtractFileName('SharedSection.sys');
    hService := CreateService(hSCManager, 'SharedSection',
                              'One way to share section',
                              SERVICE\_START or SERVICE\_STOP or \_DELETE,
                              SERVICE\_KERNEL\_DRIVER,
                              SERVICE\_DEMAND\_START,
                              SERVICE\_ERROR\_IGNORE,
                              PChar(acModulePath),
                              nil, nil, nil, nil, nil);

    if hService <> 0 then
    begin
      if StartService(hService, 0, lpTemp) then
      begin;
        hDevice := CreateFile(PChar('\\\\.\SharedSection'), 0, 0,
                              nil, OPEN\_EXISTING, 0, 0);
        if hDevice <> INVALID\_HANDLE\_VALUE then
        begin
          if DeviceIoControl(hDevice, IOCTL\_SHARE\_MY\_SECTION,
                            nil, 0, nil, 0, dwBytesReturned,
                            nil) then
          begin
            fOk := true;
          end else
          begin
            ShowMessage('Can''t send control code to device.');
          end;
          CloseHandle(hDevice);
        end else
        begin
          ShowMessage('Device is not present.');
        end;
        ControlService(hService, SERVICE\_CONTROL\_STOP, \_ss);
      end else
      begin
        ShowMessage('Can''t start driver.');
      end;
      DeleteService(hService);
      CloseServiceHandle(hService);
    end else
    begin
      ShowMessage('Can''t register driver.');
    end;
    CloseServiceHandle(hSCManager);
  end else
  begin
    ShowMessage('Can''t connect to Service Control Manager.');
  end;
  result := fOk;
end;

begin
  liSectionSize.HighPart := 0;
  liSectionSize.LowPart := SECTION\_SIZE;
  RtlInitUnicodeString(g\_usSectionName, '\\BaseNamedObjects\\UserKernelSharedSection');
  InitializeObjectAttributes(oa, @g\_usSectionName,
                            OBJ\_CASE\_INSENSITIVE, 0, nil);
  status := ZwCreateSection(@hSection, SECTION\_MAP\_WRITE or SECTION\_MAP\_READ,
                          @oa, @liSectionSize, PAGE\_READWRITE,
                          SEC\_COMMIT, 0);
  if status = STATUS\_SUCCESS then
  begin
    pSectionBaseAddress := nil;
    liViewSize.HighPart := 0;
    liViewSize.LowPart := 0;
    status := ZwMapViewOfSection(hSection, HANDLE(NtCurrentProcess),
                                pSectionBaseAddress, 0,
                                SECTION\_SIZE, nil, @liViewSize,
                                ViewShare, 0, PAGE\_READWRITE);
    if status = STATUS\_SUCCESS then
    begin
      //RtlInitUnicodeString(g\_szStrToReverse, str);
      strcpy(pSectionBaseAddress, PChar(str));
      if CallDriver then
      begin
        strcpy(sTemp, pSectionBaseAddress);
        ShowMessage(sTemp);
        ZwUnmapViewOfSection(HANDLE(NtCurrentProcess),
                            pSectionBaseAddress);
      end;
    end else
    begin
      ShowMessage('Can''t map section.');
    end;
    ZwClose(hSection);
  end else
  begin
    ShowMessage('Can''t create section.');
  end;

end.
```

这是个控制台应用程序，我们也只讲几个关键的地方，其他地方很容易理解。

```
liSectionSize.HighPart := 0;
liSectionSize.LowPart := SECTION\_SIZE;
```

建立 section 需要指明其大小，对于大小值使用 `LARGE_INTEGER` 类型的变量的程序来说，这个值可以超过 4GB。我们将这个值初始化为一个内存页的大小即 4KB。

代码:

```
InitializeObjectAttributes(oa, @g\_usSectionName,
              OBJ\_CASE\_INSENSITIVE, 0, nil);
```

对于宏 `InitializeObjectAttributes` 我们已经很熟悉了。后面的 `ZwCreateSection` 调用需要填充好的 `OBJECT_ATTRIBUTES` 结构体，填充工作我们用这些宏来完成。

我们准备使用的 section 应该取个名字，这样就可以用名字来打开它。section 的名字必须就 unicode 字符串，通过 `RtlInitUnicodeString(g_usSectionName, '\\BaseNamedObjects\UserKernelSharedSection');` 来创建。

section 对象在对象管理器名字空间的 `BaseNamedObjects` 目录下，用户进程创建的命名对象一般都在这个目录下。

代码:

```
status := ZwCreateSection(@hSection, SECTION\_MAP\_WRITE or SECTION\_MAP\_READ,
                      @oa, @liSectionSize, PAGE\_READWRITE,
                      SEC\_COMMIT, 0);
```

调用函数 `ZwCreateSection` 来创建命名 section 对象，其大小为 `SECTION_SIZE`，访问方式为可读写。如果 section 创建成功，则可以从变量 `hSection` 中得到其句柄。

代码:

```
if status = STATUS\_SUCCESS then
begin
  pSectionBaseAddress := nil;
  liViewSize.HighPart := 0;
  liViewSize.LowPart := 0;
  status := ZwMapViewOfSection(hSection, HANDLE(NtCurrentProcess),
                  pSectionBaseAddress, 0,
                  SECTION\_SIZE, nil, @liViewSize,
                  ViewShare, 0, PAGE\_READWRITE);
```

这样就将 section 的所有的视图都映射到内存中。这个函数的参数够多的所有的参数在 DDK 中都有详细介绍。参数 `pSectionBaseAddress` 初始化为 NULL，为更方便她映射 section，系统自己定义了映射 section 的虚拟地址，并将这个地址返给此参数。参数 `liViewSize` 初始化为零，这就定义了 section 将被全部映射。

代码:

```
.revird ecived a dna sessecorp resu neewteb yromem erahs ot euqinhcet emas eht esu nac uoy ,revewO .sessecorp resu gnoma yromem gnirahs rof desu euqinhcet nommoc a si elif gnigap eht yb dekcab elif deppam-yromem A
```

我们将一行倒着写的文字拷贝到所得的视图中。驱动程序的任务就是将这行文字翻转，使其变为可读的形式。

代码:

```
if status = STATUS\_SUCCESS then
begin
  strcpy(pSectionBaseAddress, PChar(str));
  if CallDriver then
  begin
    strcpy(sTemp, pSectionBaseAddress);
ShowMessage(sTemp);
ZwUnmapViewOfSection(HANDLE(NtCurrentProcess),
                  pSectionBaseAddress);
  end;
```

`CallDriver` 的返回值为 TRUE 表示驱动程序已成功完成自己的任务。我们来检查一下其工作的结果。在函数 `CallDriver` 中，我们完成通常的注册和启动驱动程序的操作并向驱动程序发送控制代码 `IOCTL_SHARE_MY_SECTION`。

最后 `ZwUnmapViewOfSection(HANDLE(NtCurrentProcess), pSectionBaseAddress);` 把系统恢复成初始的样子。

### 上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `KmdKit4DRls.part1.rar` \[附件请到论坛下载：[`bbs.pediy.com/showthread.php?t=66291`](http://bbs.pediy.com/showthread.php?t=66291) \] |
| --- | --- |
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `KmdKit4DRls.part2.rar` |
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `source.rar` |
