
# Delphi 驱动开发研究（12）-- Shared Event

> 原文：[`www.pediy.com/kssd/pediy11/107123.html`](https://www.pediy.com/kssd/pediy11/107123.html)

假如我们需要实现一个驱动程序去收集一些操作系统的统计信息，并将这些信息发送给用户层的管理程序去处理，这就需要驱动程序与管理程序交互，如何实现这种功能呢？最简单的办法就是在管理程序里创建一个计时器，这个计时器每隔一定的间隔时间（例如，每秒一次）就触发一次，通过使用 `DeviceIoControl` 函数向驱动程序发送 I/O 控制码来收集统计信息（`RegMon` 和 `FileMon` 就是这样做的）。但是如果我们的驱动程序要收集的是一些不常发生的事件信息，比如我们要收集进程创建、销毁的信息，由于系统并非每时每刻都有进程创建或者销毁，所以定时执行 `DeviceIoControl` 往往都是白忙一场，获取不到任何信息。此时我们可以考虑增加触发定时器的间隔时间（例如，把间隔时间增加到 10 秒），但是如果间隔时间过长又可能会出现驱动程序已经收集到信息但是由于管理程序的定时器尚未被触发而导致未能及时获取统计信息的情况发生。显然，比较合理的方案应该是由驱动程序来决定管理程序什么时候获取统计信息，因为只有驱动程序才知道具体的事件是什么时候发生的，当事件发生后，驱动程序以某种方式通知管理程序，有事件发生，你可以获取数据了，此时管理程序再用 `DeviceIoControl` 向驱动程序发送控制码以获取最新的统计信息数据。

事实上，Windows 给我们提供了两种比较好的方法可以让我们达到目的。一种方法是使用异步的 `DeviceIoControl`，这种方法要求用户在使用 `CreateFile` 创建对象时必须要设置 `FILE_FLAG_OVERLAPPED` 标志，同时填充一个 `OVERLAPPED` 结构并将其地址做为 `CreateFile` 的最后一个参数传递。异步 `DeviceIoControl` 调用可能返回的结果有三种：返回 `TRUE`，这说明驱动程序的分派子程序可以立即完成请求；返回 `FALSE` 并且调用 `GetLastError` 取得的错误码是 `ERROR_IO_PENDING`，这表明驱动程序的分派子程序返回 `STATUS_PENDING` 并会在稍后完成控制操作。这里要注意的是，`ERROR_IO_PENDING` 并不代表真的出错了，它仅仅是一种系统表明任何事件正在被正常处理的途径而已；如果返回 `FALSE` 并且用 `GetLastError` 取得的错误码也不是 `STATUS_PENDING`，那就是真的出错了。这里我们不打算详细讨论异步 `DeviceIoControl` 方法，有兴趣的话，您可以参考 DDK 或者 MSDN。另一种方法就是同步调用了，这种方法相对要容易些，这种方法要求在驱动程序和用户程序间共享一个事件，驱动程序通过这个事件对象向客户程序发送信号，客户程序收到信号后，就可以从驱动程序接收一些自己感兴趣的信息。

如何才能共享事件呢？你可以使用一个命名对象，然后通过名字访问这个对象。我们之前在 SharedSection 的例子中就使用过这种方法。这种方法的基本技巧就是让该命名对象一直存在于内存中，所以缺点也就显而易见了，这个命名对象对所有进程都是可见的，但是即便如些，它也比使用无名对象要些。当然，我们还有更好的方法，就是先在客户程序中用 `CreateEvent` 创建一个对象，然后调用 `DeviceIoControl` 把事件句柄传递给驱动程序。在本章中我们将使用这种方法创建一个简单的进程监视器，这个监视器可以监视进程的创建与销毁。下面是 `ProcessMon` 管理程序运行时的截图：

![](img/7610fb4dfa881a53d2f3948ccee5c6d4.jpg)

## 12.1 全局数据

我们先来看一下 `common.pas` 中定义的一些全局数据。

代码:

```
const
  IOCTL_SET_NOTIFY      = (FILE_DEVICE_UNKNOWN shl 16) or (FILE_WRITE_ACCESS shl 14) or ($800 shl 2) or METHOD_BUFFERED;
  IOCTL_REMOVE_NOTIFY    = (FILE_DEVICE_UNKNOWN shl 16) or (0 shl 14) or ($801 shl 2) or 0;
  IOCTL_GET_PROCESS_DATA = (FILE_DEVICE_UNKNOWN shl 16) or (FILE_READ_ACCESS shl 14) or ($802 shl 2) or METHOD_BUFFERED;

  IMAGE_FILE_PATH_LEN    = 512;

type
  {$RTTI EXPLICIT METHODS([]) PROPERTIES([]) FIELDS([])}
  PROCESS_DATA = packed record
    bCreate: DWORD;
    dwProcessId: DWORD;
    { full process's image file path }
    szProcessName: array[0..IMAGE_FILE_PATH_LEN - 1] of AnsiChar;
  end;
```

在这里，我们定义了三个控制码：`IOCTL_SET_NOTIFY` 使得驱动开始踪进程的创建和销毁；`IOCTL_REMOVE_NOTIFY` 的作用正好与 `IOCTL_SET_NOTIFY` 相反；`IOCTL_GET_PROCESS_DATA` 则返回保存在 `PROCESS_DATA` 结构里的进程信息，这个信息包括进程 ID、一个说明进程是被创建还是被销毁的标志以及相关进程的完整路径信息。

## 12.2 ProcessMon 的用户管理程序源码

代码:

```
unit main;

interface

uses
  Windows, Messages, SysUtils, Variants, Classes, Graphics, Controls, Forms,
  Dialogs, ComCtrls, WinSvc, common;

type
  TForm1 = class(TForm)
    lvProcessInfo: TListView;
    procedure FormActivate(Sender: TObject);
    procedure FormCreate(Sender: TObject);
    procedure FormClose(Sender: TObject; var Action: TCloseAction);
  private
    { Private declarations }
  public
    { Public declarations }
  end;

var
  Form1: TForm1;
  g_hEvent: THandle;
  g_fbExitNow: Boolean;
  g_hDevice: THandle;

implementation

uses
  GetData;

var
  g_hSCManager: THandle;
  g_hService: THandle;
  tgd: TGetData;
{$R *.dfm}

procedure TForm1.FormActivate(Sender: TObject);
var
  acModulePath: string;
  lpTemp: PChar;
  dwBytesReturned: DWORD;
begin
  g_hSCManager := OpenSCManager(nil, nil, SC_MANAGER_ALL_ACCESS);
  if g_hSCManager <> 0 then
  begin
    acModulePath := GetCurrentDir + '\\' + ExtractFileName('ProcessMon.sys');
    g_hService := CreateService(g_hSCManager, 'ProcessMon',
                               'Process creation/destruction monitor',
                               SERVICE_START or SERVICE_STOP or _DELETE,
                               SERVICE_KERNEL_DRIVER,
                               SERVICE_DEMAND_START,
                               SERVICE_ERROR_IGNORE,
                               PChar(acModulePath),
                               nil, nil, nil, nil, nil);
    if g_hService <> 0 then
    begin
      if StartService(g_hService, 0, lpTemp) then
      begin
        g_hDevice := CreateFile('\\.\ProcessMon', GENERIC_READ or GENERIC_WRITE,
                                0, nil, OPEN_EXISTING, 0, 0);
        if g_hDevice <> INVALID_HANDLE_VALUE then
        begin
          { No need it to be registered anymore }
          DeleteService(g_hService);
          { Create unnamed auto-reset event to be signalled when there is data to read. }
          g_hEvent := CreateEvent(nil, False, false, nil);
          { Create thread to wait event signalled. }
          tgd := TGetData.Create(False);
          if not DeviceIoControl(g_hDevice, IOCTL_SET_NOTIFY,
                                 @g_hEvent, SizeOf(g_hEvent), nil, 0,
                                 dwBytesReturned, nil) then
          begin
            ShowMessage('无法设置通知!');
          end;
        end else
        begin
          ShowMessage('无法打开设备!');
        end;
      end else
      begin
        ShowMessage('无法启动驱动!');
      end;
    end else
    begin
      ShowMessage('无法注册驱动!');
    end;
  end else
  begin
    ShowMessage('无法连接到 SCM!');
  end;
end;

procedure TForm1.FormClose(Sender: TObject; var Action: TCloseAction);
var
  dwBytesReturned: DWORD;
  _ss: SERVICE_STATUS;
begin
  DeviceIoControl(g_hDevice, IOCTL_REMOVE_NOTIFY,
                  nil, 0, nil, 0, dwBytesReturned, nil);

  g_fbExitNow := true;  { If exception has occured not in loop thread it should exit now. }
  SetEvent(g_hEvent);
  Sleep(100);

  CloseHandle(g_hEvent);
  CloseHandle(g_hDevice);

  ControlService(g_hService, SERVICE_CONTROL_STOP, _ss);

  DeleteService(g_hService);

  CloseServiceHandle(g_hService);
  CloseServiceHandle(g_hSCManager);
end;

procedure TForm1.FormCreate(Sender: TObject);
begin
  if(MessageDlg('由于条件所限，本驱动仅在 Windows XP sp3 上做过测试，是否继续？',
    mtWarning, [mbYes, mbNo], 0, mbYes) = mrNo) then
  begin
    Application.Terminate;
  end;
end;

end.
```

## 12.3 客户端程序说明

客户端程序比较简单，在 Form 的 OnActivate 事件处理函数中，我们加载驱动并启动它。如果启动成功则调用 `CreateEvent` 创建一个非信号态、自动重置的无名信号对象，并将其句柄保存在全局变量 `g_hEvent` 中，接下来，创建一个线程用于与内核驱动交互。这里，我使用了 Delphi 的 `TThread` 对象。

代码:

```
if not DeviceIoControl(g_hDevice, IOCTL_SET_NOTIFY,
      @g_hEvent, SizeOf(g_hEvent), nil, 0,
      dwBytesReturned, nil) then
```

接下来调用 `DeviceIoControl` 向驱动程序发送 `IOCTL_SET_NOTIFY` 控制码，并将事件对象句柄传送给驱动程序。如果成功，驱动程序就开始跟踪进程的创建和销毁，如果系统中有进程创建或者销毁，驱动程序就会设置事件对象到信号态以通知客户端程序有事情发生了，你可以获取信息了。

OnClose 事件处理函数完成退出前的清理工作，先调用 `DeviceIoControl` 函数向驱动程序发送 `IOCTL_REMOVE_NOTIFY` 控制码，让驱动程序停止跟踪，然后停止并卸载驱动。下面来看下线程对象。

## 12.4 线程对象

代码:
