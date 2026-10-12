
```
  fastcall IofCompleteRequest,pIrp,IO_NO_INCREMENT  
  mov eax,STATUS_SUCCESS  
  ret  
DispatchCreateClose endp  
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
;DispatchControl  
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
DispatchControl proc pDeviceObject:PDEVICE_OBJECT,pIrp:PIRP  
  LOCAL status:NTSTATUS  
  mov esi,pIrp  
  assume esi:PTR _IRP  
  IoGetCurrentIrpStackLocation esi  
  mov edi,eax  
  assume edi:PTR IO_STACK_LOCATION  
  .if [edi].Parameters.DeviceIoControl.IoControlCode == IOCTL_DEVICEIOCONTROL  
    invoke DbgPrint,$CTA0("IOCTL_DEVICEIOCONTROL IS CALL")    ;演示之用，请读者自行扩展  
  .elseif [edi].Parameters.DeviceIoControl.IoControlCode == ....   如果还有其它的请求进行处理  
  .else  
    mov status,STATUS_INVALID_DEVICE_REQUEST  
  .endif  
  assume edi:nothing  
  push status  
  pop [esi].IoStatus.Status  
  push 0  
  pop [esi].IoStatus.Information  
  assume esi:nothing  
  fastcall IofCompleteRequest,pIrp,IO_NO_INCREMENT  
  mov eax,status  
  ret  
DispatchControl endp  
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
;DriverUnload  
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
DriverUnload proc pDriverObject:PDRIVER_OBJECT  
  ;删除符号符连接  
  invoke IoDeleteSymbolicLink,addr g_usSymbolicLinkName  
  mov eax,pDriverObject  
  ;删除设备对象  
  invoke IoDeleteDevice,(DRIVER_OBJECT PTR[eax]).DeviceObject  
  ret  
DriverUnload endp  
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
;DriverEntry  
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
DriverEntry proc pDriverObject:PDRIVER_OBJECT, pusRegistryPath:PUNICODE_STRING  
  LOCAL status:NTSTATUS  
  LOCAL pDeviceObject:PDEVICE_OBJECT  
  mov status,STATUS_DEVICE_CONFIGURATION_ERROR  
  invoke IoCreateDevice,pDriverObject,0,addr g_usDeviceName,FILE_DEVICE_UNKNOWN,0,FALSE,addr pDeviceObject  
  .if eax == STATUS_SUCCESS  
    invoke IoCreateSymbolicLink,addr g_usSymbolicLinkName,addr g_usDeviceName  
    .if eax == STATUS_SUCCESS  
      mov eax,pDriverObject  
      assume eax:PTR DRIVER_OBJECT  
      mov [eax].DriverUnload,offset DriverUnload  
      mov [eax].MajorFunction[IRP_MJ_CREATE*(sizeof PVOID)],offset DispatchCreateClose  
      mov [eax].MajorFunction[IRP_MJ_CLOSE*(sizeof PVOID)],offset DispatchCreateClose  
      mov [eax].MajorFunction[IRP_MJ_DEVICE_CONTROL*(sizeof PVOID)],offset DispatchControl  
      assume eax:nothing  
      mov status,STATUS_SUCCESS  
    .else  
      invoke IoDeleteDevice,pDeviceObject  
    .endif  
  .endif  
  mov eax,status  
  ret  
DriverEntry endp  
end DriverEntry  
```

好了，代码很简单，我就不多讲了，现在我们来看看应用层的代码吧！

我们用 `RadASM` 建立一个应用程序 `DeviceControlExe`

`DeviceControlExe.inc` 的代码如下：

```
IOCTL_DEVICEIOCONTROL equ CTL_CODE(FILE_DEVICE_UNKNOWN,800h,METHOD_BUFFERED,FILE_READ_ACCESS + FILE_WRITE_ACCESS)  
```

`DeviceControlExe.asm` 的代码如下：

```
.386  
.model flat,stdcall  
option casemap:none  
include windows.inc  
include user32.inc  
include kernel32.inc  
include advapi32.inc  
includelib user32.lib  
includelib kernel32.lib  
includelib advapi32.lib  
include D:\\RadASM\\masm32\\include\\winioctl.inc       ;当使用 IOCTL 时必须包含这个头文伯  
include D:\\RadASM\\masm32\\macros\\Strings.mac  
include DeviceControlExe.inc  
.code  
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
;ControlConnection  
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
ControlConnection proc uses esi edi  
  LOCAL hDevice:HANDLE  
  LOCAL dwBytesReturned:DWORD  
  invoke CreateFile,$CTA0("\\\\.\DeviceControl"),GENERIC_READ + GENERIC_WRITE,\  
    0,NULL,OPEN_EXISTING,0,NULL  
  .if eax != INVALID_HANDLE_VALUE  
    mov hDevice,eax  
    invoke DeviceIoControl,hDevice,IOCTL_DEVICEIOCONTROL,\  
       NULL,NULL,NULL,NULL,addr dwBytesReturned,NULL  
    .if (eax != 0 ) && (dwBytesReturned == 0)  
      invoke MessageBox,NULL,$CTA0("Send Control Code is Success"),$CTA0("Success"),MB_OK  
    .else  
      invoke MessageBox,NULL,$CTA0("Send Control Code is Failed"),$CTA0("Failed"),MB_OK  
    .endif  
  .else  
    invoke MessageBox,NULL,$CTA0("Device Connection failed"),$CTA0("Failed"),MB_OK + MB_ICONSTOP  
  .endif  
  ret  
ControlConnection endp  
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
;InstallDriver  
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
InstallDriver proc  
  LOCAL hSCManager:HANDLE  
  LOCAL hService:HANDLE  
  LOCAL acModulePath[MAX_PATH]:CHAR  
  LOCAL _ss:SERVICE_STATUS  
  invoke OpenSCManager,NULL,NULL,SC_MANAGER_ALL_ACCESS  
  .if eax != NULL  
    mov hSCManager,eax  
    push eax  
    invoke GetFullPathName,$CTA0("DeviceControl.sys"),sizeof acModulePath,addr acModulePath,esp  
    pop eax  
    invoke CreateService,hSCManager,$CTA0("DeviceControl"),$CTA0("DeviceControl"),\  
      SERVICE_START + SERVICE_STOP + DELETE,SERVICE_KERNEL_DRIVER,SERVICE_DEMAND_START,\  
      SERVICE_ERROR_IGNORE,addr acModulePath,NULL,NULL,NULL,NULL,NULL  
    .if eax != NULL  
      mov hService,eax  
      invoke StartService,hService,0,NULL  
      .if eax != 0  
        invoke ControlConnection  
        invoke ControlService,hService,SERVICE_CONTROL_STOP,addr _ss  
      .else  
        invoke MessageBox,NULL,$CTA0("Cann't start driver"),$CTA0("Failed"),MB_OK + MB_ICONSTOP  
      .endif  
    .else  
      invoke MessageBox,NULL,$CTA0("Can't register driver"),$CTA0("Failed"),MB_OK + MB_ICONSTOP  
    .endif  
    invoke DeleteService,hService  
    invoke CloseServiceHandle,hSCManager  
  .else  
    invoke MessageBox,NULL,$CTA0("Cann't connect to Service Control Manager"),$CTA0("Failed"),MB_OK + MB_ICONSTOP  
  .endif  
  ret  
InstallDriver endp  
start proc  
  invoke InstallDriver  
  invoke ExitProcess,NULL  
  ret  
start endp  
end start  
```

呵呵，这样我们就完成了应用层的代码了，代码很简单，仅作为演示之用，如您有其它的需要自行添加其它功能~~

程序运行如图所示（将程序与驱动放在同一个目录下）

图 1

![](img/b945c9bda950c43d657edc9c748cf476.jpg)

好了，就先讲到这里，争取在过年之前，将驱动的相关编程全部讲完，申明：我只会讲方法，至于具体你要实现什么功能，由你自己完成，有句话说的好“师傅领进门，修行在各人”没有师傅也行，不过我想有的话可能会更快点，虽然我不算“师傅”级的人，仅是想和大家一起学习，才有这个想法，下次再见~~

(本来想把附件也传上，不过想想还是让大家多上机操作吧，熟能生巧，不要总是 `CTRL+C` 加 `CTRL+V` 的![](img/c1376e2bd59b6be48d07a70822c7e47d.jpg "伤心")~~)
