
# [原创]Delphi研究之驱动开发篇（三）--一个完整的驱动程序示例

## Delphi研究之驱动开发篇（三）

（注：本篇的原理部分均摘自罗云彬大侠翻译的驱动开发教程）

在前面的两篇教程中我们写了三个玩具驱动程序，为什么说是玩具驱动呢？因为它们确确实实是驱动程序，而且也能完成一些有趣的功能，但是它们都不完整，没有同用户交流的功能，这一篇就让我们来完成一个简单的全功能驱动程序。

在写程序之前，我们有必要了解一些基础知识。

在用户模式下，我们可以通过访问某个地址来直接调用`dll`中的函数，但是在内核模式下，从系统的稳定性考虑，这样做是非常危险的。所以，系统提供了和内核模式通讯的媒介——`I/O管理器`，它是`I/O子系统`的部件之一。`I/O管理器`将应用程序、系统部件和设备连接起来，并定义了一个架构来支持设备驱动程序。

一般来说，用户模式的操作都被转换成了对具体硬件设备的`I/O`操作，仅对于某些设备，设备由驱动程序来创建和控制，这些设备就是虚拟设备。当然，创建这些设备并不意味着你创造了什么硬件，而仅仅是在内存中创建了一个新的对象而已。每个对象和一个物理设备或者逻辑设备对应，用于描述它们的特征。

创建设备后，驱动程序告诉`I/O管理器`：“这里有个我控制的设备，如果你收到了操作这个设备的`I/O`请求的话，直接发给我好了，剩下的由我来搞定！”。驱动程序知道如何对自己管理的设备进行`I/O`操作，`I/O管理器`唯一的职责在于创建`I/O`请求并把它发送给适当的设备驱动程序。用户模式的代码不知道（也不必知道）其中的细节，也不用知道究竟是哪个驱动程序在管理哪个设备。

下面先让我们来看一下用户模式下的控制程序：

```
program VirToPhys;



{$APPTYPE CONSOLE}



uses

  SysUtils, Windows, WinSvc, Dialogs, nt_status;



const

  NUM_DATA_ENTRY =4;

  DATA_SIZE = sizeof(DWORD) * NUM_DATA_ENTRY;

  _DELETE = $10000;



var

  hSCManager:THANDLE;

  hService:THANDLE;

  acModulePath: array [0..MAX_PATH] of char;

  _ss:SERVICE_STATUS;

  hDevice:THANDLE;



  adwInBuffer: array [0..NUM_DATA_ENTRY] of DWORD;

  adwOutBuffer: array [0..NUM_DATA_ENTRY] of DWORD;

  dwBytesReturned:DWORD;

  IOCTL_GET_PHYS_ADDRESS: DWORD;

  lpTemp: PChar;

  iRetValue: boolean;



{生成控制码}  

function CTL_CODE(DeviceType, Func, Method, Access: DWORD): DWORD;

begin

  result := (((DeviceType) SHL 16) OR ((Access) SHL 14) OR ((Func) SHL 2) OR (Method));

end;



begin

  IOCTL_GET_PHYS_ADDRESS := CTL_CODE(FILE_DEVICE_UNKNOWN,

                            $800, METHOD_BUFFERED,

                            FILE_READ_ACCESS + FILE_WRITE_ACCESS);

  hSCManager := OpenSCManager(nil, nil, SC_MANAGER_ALL_ACCESS);

  if hSCManager <> 0 then

  begin

    GetFullPathName(PChar('VirtToPhys.sys'), sizeof(acModulePath), acModulePath, lpTemp);

    hService := CreateService(hSCManager, 'VirtToPhys', 'Virtual To Physical Address Converter',

                              SERVICE_START + SERVICE_STOP + _DELETE, SERVICE_KERNEL_DRIVER,

                              SERVICE_DEMAND_START, SERVICE_ERROR_IGNORE, acModulePath,

                              nil, nil, nil, nil, nil);

    if hService <> 0 then

    begin

      {驱动程序的DriverEntry过程将被调用}

      if StartService(hService, 0, lpTemp) = true then

      begin

        {驱动程序将接收IRP_MJ_CREATE I/O请求包(IRP)}

        hDevice := CreateFile('\\.\slVirtToPhys', GENERIC_READ+GENERIC_WRITE,

                              0, nil, OPEN_EXISTING, 0, 0);

        if hDevice <> INVALID_HANDLE_VALUE  then

        begin

          {准备送往驱动程序的数据包}

          adwInBuffer[0] := GetModuleHandle(nil);

          adwInBuffer[1] := GetModuleHandle('kernel32.dll');

          adwInBuffer[2] := GetModuleHandle('user32.dll');

          adwInBuffer[3] := GetModuleHandle('comctl32.dll');

          {驱动程序将接收IRP_MJ_DEVICE_CONTROL I/O请求包}

          iRetValue := DeviceIoControl(hDevice, IOCTL_GET_PHYS_ADDRESS,

                                       @adwInBuffer, sizeof(adwInBuffer),

                                       @adwOutBuffer, sizeof(adwOutBuffer),

                                       dwBytesReturned, nil);

          if (iRetValue = true) and (dwBytesReturned <> 0) then

          begin

            {取程序名}

            GetModuleFileName(adwInBuffer[0], acModulePath, sizeof(acModulePath));

            ShowMessage(Format('Modules          BASE     Physical'#13#10 +

                               '----------------------------------'#13#10 +

                               '%s    %8.8X %8.8X'#13#10 +

                               'kernel32.dll     %8.8X %8.8X'#13#10 +

                               'user32.dll       %8.8X %8.8X'#13#10 +

                               'comctl32.dll     %8.8X %8.8x',

                               [ExtractFileName(acModulePath), adwInBuffer[0], adwOutBuffer[0],

                                adwInBuffer[1], adwOutBuffer[1],

                                adwInBuffer[2], adwOutBuffer[2],

                                adwInBuffer[2], adwOutBuffer[2]]));

          end else

          begin

            ShowMessage('Can''t send control code to device.');

          end;

          {Driver will receive IRP of type IRP_MJ_CLOSE}

          CloseHandle(hDevice);

        end else

        begin

          ShowMessage('Device is not present.');

        end;

        {DriverUnload proc in our driver will be called}

        ControlService(hService, SERVICE_CONTROL_STOP, _ss);

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

end.
```

这个代码包括了注册和启动驱动程序，以及作为客户端程序和设备进行通讯的代码。代码将输入的数据发送给设备，并将设备返回的数据格式化并显示出来。

驱动被装载后，`VirtToPhys`驱动程序创建了一个名为`devVirtToPhys`的设备，内部设备名称无法被Win32应用程序使用，因此，如果我们的驱动程序对应的设备对象希望被用户模式的代码打开的话，就必须在`\??`目录中创建一个符号连接，指向`\Device`目录中的设备对象，然后，当调用者需要获取设备句柄时，`I/O管理器`就能够找到它。

`VirtToPhys`驱动程序在`\??`目录中创建了指向`devVirtToPhys`设备的符号连接`slVirtToPhys`，真实设备的全名是`\Device\devVirtToPhys`，这样，当`StartService`函数执行后，系统中就多了三个新的对象：`\Driver\VirtToPhys`驱动、`\Device\devVirtToPhys`设备和符号连接`\??\slVirtToPhys`。

现在来看看控制程序源代码，当驱动被启动后，我们只需要使用`CreateFile`函数来打开驱动，以此获得一个文件句柄。函数原型如下：

```
function CreateFile(lpFileName: PChar; 

                    dwDesiredAccess, dwShareMode: DWORD;

                    lpSecurityAttributes: PSecurityAttributes;dwCreationDisposition, dwFlagsAndAttributes: DWORD;

                    hTemplateFile: THandle): THandle; stdcall;
```

这个函数可以创建或者打开一个已存在的对象，而不仅仅是文件。函数的参数描述如下：

◎ `lpFileName`--指向以0结尾的表示设备名称的字符串，这里要用到指向设备对象的符号连接名

◎ `dwDesiredAccess`--指定访问设备的方式，可以有两个取值：`GENERIC_READ`表示写操作，允许将数据写到设备中；`GENERIC_WRITE`表示读操作，允许从设备读取数据，这两个值可以合并起来使用

◎ `dwShareMode`--指定设备是否可以被共享，0表示设备不能被共享，这样并发的访问会失败，直到句柄被关闭为止；要共享设备的话，可以指定下面的两个值：`FILE_SHARE_READ`表示可以并发地读取设备，`FILE_SHARE_WRITE`表示可以并发地写设备

◎ `lpSecurityAttributes`--指向`SECURITY_ATTRIBUTES`结构的指针，在此无用，所以可以指定为`NULL`

◎ `dwCreationDistribution`--指明当文件存在或不存在时函数采取的动作，对于设备来说，这个参数应该使用`OPEN_EXISTING`

◎ `dwFlagsAndAttributes`--文件属性，在这里总是使用0

◎ `hTemplateFile`--指定文件模板的句柄，在这里总是使用0

如果`CreateFile`函数成功地创建或者打开了指定的设备，那么返回值就是设备句柄，否则返回值是`INVALID_HANDLE_VALUE`。我们这样使用`CreateFile`函数：

```
hDevice := CreateFile('\\.\slVirtToPhys', GENERIC_READ+GENERIC_WRITE,0, nil, OPEN_EXISTING, 0, 0);
```

来看看第一个参数，这是指向符号连接名称的字符串指针，名称格式是`\\.\slVirtToPhys`，`\\.\`是Win32中定义本地计算机的方法，`CreateFile`函数实际上是`NtCreateFile`函数的封装（位于`%SystemRoot%\System32\ntdll.dll`中），后者将访问定向到系统服务中去（注意不要和Win32服务进程混淆）。

`NtCreateFile`将本地计算机的别名`\\.\`用`\??`代替（这样`\\.\slVirtToPhys`就变成了`\??\slVirtToPhys`）并调用内核的`ObOpenObjectByName`函数，通过符号连接名称，`ObOpenObjectByName`函数找到`\Device\devVirtToPhys`对象并返回它的指针。`NtCreateFile`使用这个指针创建新的文件对象并返回句柄。

操作系统把所有的`I/O`操作请求抽象化成一个虚拟的文件，隐藏了目标设备的`I/O`操作可能在结构上并不等同于文件的事实，然后由驱动程序负责将对虚拟文件的请求转换成对具体硬件的请求，这种抽象可以推广到所有用户进程和设备之间的界面，所有对这种虚拟文件的读写操作都被当作是简单的流操作来处理。

在`CreateFile`返回前，`I/O管理器`创建了`IRP_MJ_CREATE`类型的`IRP`并将其传给驱动处理，驱动中负责响应这个`IRP`的子程序代码会在发起`I/O`请求（就是调用`CreateFile`函数的代码）的线程环境中执行，该线程的`IRQL`等于`PASSIVE_LEVEL`。如果驱动的子程序成功返回，那么对象管理器在进程的句柄表中为文件对象创建一个句柄，然后将其一层层地返回，直到返回到`CreateFile`函数。

现在来总结一下，`\\.\slVirtToPhys`会被转换成符号连接`\??\slVirtToPhys`，最终用来找到需要的`\Device\devVirtToPhys`设备。然后可以从负责维护该设备的驱动程序中取得设备对象`DEVICE_OBJECT`，接下来`I/O管理器`将`IRP_MJ_CREATE`请求传递给驱动，驱动程序知道如何处理这个请求。如果驱动打算处理该请求，那么它返回成功代码，这样对象管理器创建对应这个设备的虚拟文件句柄并将它返回给用户模式代码。

句柄和符号连接为间接访问系统资源提供服务，这种“间接”方式将应用程序和系统的数据结构隔离开来。

`CreateFile`函数返回有效的设备句柄后，我们将它保存在`hDevice`变量中，现在可以用`ReadFile`、`WriteFile`以及`DeviceIoControl`函数来和设备通讯了。`DeviceIoControl`函数是用来和设备通讯的通用函数，它的原型如下：

```
function DeviceIoControl(hDevice: THandle; 

    dwIoControlCode: DWORD; 

    lpInBuffer: Pointer;          

    nInBufferSize: DWORD; 

    lpOutBuffer: Pointer;  

    nOutBufferSize: DWORD;

    var lpBytesReturned: DWORD; 

    lpOverlapped: POverlapped): BOOL; stdcall;
```

◎ `hDevice`--设备的句柄
