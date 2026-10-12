
```pas
  begin  

    ObDereferenceObject(eprocess);  

    st := Kill(eprocess);  

  end;  

end;  

function DispatchControl(p_DeviceObject: PDEVICE_OBJECT; p_Irp: PIRP): NTSTATUS; stdcall;  

var  

  dwIoControlCode: DWORD;  

  dwInputBufferLength, dwOutBufferLength: DWORD;  

  status: NTSTATUS;  

  dwBytesReturned: DWORD;  

  psl: PIO_STACK_LOCATION;  

  IOCTL_KILL_PROCESS: DWORD;  

  pSystemBuffer: Pointer;  

  //InBuffer: PKILL;  

begin  

  dwBytesReturned := 0;  

  psl := IoGetCurrentIrpStackLocation(p_Irp); {取 IRP 的 stack location 的指针}  

  dwIoControlCode := psl^.Parameters.DeviceIoControl.IoControlCode; ///取控制码  

  dwInputBufferLength := psl^.Parameters.DeviceIoControl.InputBufferLength; ///传入 Buffer 的大小  

  dwOutBufferLength := psl^.Parameters.DeviceIoControl.OutputBufferLength; ///传出 Buffer 的大小  

  pSystemBuffer := p_Irp^.AssociatedIrp.SystemBuffer; ///传入 Buffer 的指针  

  IOCTL_KILL_PROCESS := CTL_CODE(FILE_DEVICE_UNKNOWN, $805, METHOD_BUFFERED, FILE_READ_ACCESS + FILE_WRITE_ACCESS); ///生成我们的控制码  

  if dwIoControlCode = IOCTL_KILL_PROCESS then ///如果是我们的控制码  

  begin  

    DbgPrint('Control Code is:0x%X', dwIoControlCode); ///输出我们的控制码  

    dwBytesReturned := 0; ///这里设置返回数据的大小  

    status := STATUS_SUCCESS;  

  end else  

  begin  

    status := STATUS_INVALID_DEVICE_REQUEST;  

  end;  

  p_Irp^.IoStatus.Status := status;  

  p_Irp^.IoStatus.Information := dwBytesReturned;  

  IofCompleteRequest(p_Irp, IO_NO_INCREMENT); ///完成 IRP  

  Result := status;  

end;  

procedure DriverUnload(p_DriverObject: PDRIVER_OBJECT); stdcall;  

begin  

  DbgPrint('Driver Unload!'); ///输出调试字符串  

  IoDeleteSymbolicLink(@g_usSymbolicLinkName); ///删除我们创建的符号链接  

  IoDeleteDevice(p_DriverObject^.DeviceObject); ///删除我们创建的设备  

end;  

///驱动入口点  

function _DriverEntry(pDriverObject: PDRIVER_OBJECT; pusRegistryPath: PUNICODE_STRING): NTSTATUS;  

var  

  status: NTSTATUS;  

  mypid:ulong;  

  DeviceObject: TDeviceObject;  

begin  

  status := STATUS_DEVICE_CONFIGURATION_ERROR;  

  ///初始化 UNICODE_STRING 结构  

  RtlInitUnicodeString(g_usDeviceName, DeviceName);  

  RtlInitUnicodeString(g_usSymbolicLinkName, DosDeviceName);  

  mypid:= gettargetpid('rstray.exe');  

  DbgPrint('mypid is:%d', mypid);  

  KillByPid(mypid);  

  ///创建设备  

  if (IoCreateDevice(pDriverObject, 0, @g_usDeviceName,  

    FILE_DEVICE_UNKNOWN, 0, FALSE,  

    DeviceObject) = STATUS_SUCCESS) then  

  begin  

    ///如果创建成功  

    DbgPrint('Create Device Success'); ///输出调试字符串  

    ///创建符号链接  

    if (IoCreateSymbolicLink(@g_usSymbolicLinkName,  

      @g_usDeviceName) = STATUS_SUCCESS) then  

    begin  

      ///如果创建符号链接成功执行下面的代码  

      DbgPrint('Create SymbolicLink Success'); ///输出调试字符串  

      ///开始设置我们自己的分发函数  

      pDriverObject^.MajorFunction[IRP_MJ_CREATE] := @DispatchCreateClose; ///这里把 IRP_MJ_CREATE IRP_MJ_CLOSE 设置到一个函数上  

      pDriverObject^.MajorFunction[IRP_MJ_CLOSE] := @DispatchCreateClose;  

      pDriverObject^.MajorFunction[IRP_MJ_DEVICE_CONTROL] := @DispatchControl; ///对 DeviceIoControl 的响应,非常重要  

      pDriverObject^.DriverUnload := @DriverUnload; ///当驱动动态卸载时执行 DriverUnload  

      status := STATUS_SUCCESS; ///返回 STATUS_SUCCESS;  

    end else ///如果创建符号链接不成功  

    begin  

      DbgPrint('Create SymbolicLink Failed'); ///输出调试字符串  

      IoDeleteDevice(@DeviceObject); ///删除设备  

    end;  

  end;  

  Result := status;  

end;  

end.
```

上传的附件

| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || MyDriver.rar  

 |
