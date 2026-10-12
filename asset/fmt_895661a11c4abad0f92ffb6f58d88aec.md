
```asm
  jmp  Quit  

GetSSDTNum:  

  push  4  

  pop  eax  

  cmp  ecx, eax ;输入长度<4 跳转  

  jb  CreateAndClose  

  mov  ecx, KeServiceDescriptorTable  

  mov  ecx, [ecx+8] ;NumberOfService  

  mov  [edx], ecx  ;edx 指向 IRP.AssociatedIrp.SystemBuffer  

Quit:  

  mov  [esi+1Ch], eax  

CreateAndClose:  

  mov  edi, [esi+18h] ;IoStatus  

  xor  dl, dl  

  mov  ecx, esi  

  call  IofCompleteRequest  

  mov  eax, edi  

  pop  edi  

  pop  esi  

  ret  

wcharDeviceName wchar L(\\Device\\Swk0217\0)  

wcharSymbolicLink wchar L(\\DosDevices\\Swk0217\0)  

start proc DriverObject:dword  

  LOCAL SymbolicLinkName:UNICODE_STRING  

  LOCAL DestinationString:UNICODE_STRING  

  LOCAL DeviceObject:dword  

  and  DeviceObject, 0  

  push  esi  

  push  edi  

  mov  edi, RtlInitUnicodeString  

  push  offset wcharDeviceName ; SourceString  

  lea  eax, DestinationString  

  push  eax    ; DestinationString  

  call  edi           ; RtlInitUnicodeString  

  mov  esi, DriverObject  

  lea  eax, DeviceObject  

  push  eax    ; DeviceObject  

  push  0    ; Exclusive  

  push  0    ; DeviceCharacteristics  

  push  598347h    ; DeviceType  

  lea  eax, DestinationString  

  push  eax    ; DeviceName  

  push  0    ; DeviceExtensionSize  

  push  esi    ; DriverObject  

  call  IoCreateDevice  

  test  eax, eax  

  jl  @f  

  push  offset wcharSymbolicLink ; SourceString  

  lea  eax, SymbolicLinkName  

  push  eax    ; DestinationString  

  call  edi ; RtlInitUnicodeString  

  lea  eax, DestinationString  

  push  eax    ; DeviceName  

  lea  eax, SymbolicLinkName  

  push  eax    ; SymbolicLinkName  

  call  IoCreateSymbolicLink  

  mov  ecx, offset DispatchFunction  

  mov  [esi+70h], ecx ;IRP_MJ_DEVICE_CONTROL  

  mov  [esi+40h], ecx ;IRP_MJ_CLOSE  

  mov  [esi+38h], ecx ;IRP_MJ_CREATE  

  mov  dword ptr [esi+34h], offset swkUnLoad ;DriverObject.DriverUnLoad  

@@:  

  pop  edi  

  pop  esi  

  ret  

start endp  

end start
```

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | `swk0207.rar` |
|---|---|
