
# 发个 RadASM 用的标准控制驱动模版

> 原文：[`www.pediy.com/kssd/pediy11/105136.html`](https://www.pediy.com/kssd/pediy11/105136.html)

使用方法： 把下面的内容用记事本另存为`"RadASM\\Masm\\Templates\\标准控制驱动.tpl"`即可，方便写控制设备对象驱动。
![](img/110d920e4e703968a2a60c986d363163.jpg "尴尬")

引用: `Driver` (`.sys`)

```
Driver
标准控制驱动框架

[*BEGINPRO*]

[*BEGINDEF*]

[MakeDef]

Menu=0,1,1,1,0,0,0,0,0,0,0,0,0,0,0,0

1=4,O,$B\\RC.EXE /v,1

2=3,O,$B\\ML.EXE /nologo /c /coff /I"$I",2

3=16,O,$B\\LINK.EXE /nologo /driver /base:0x10000 /align:32 /subsystem:native /out:"$16",3

4=0,0,,5

5=rsrc.obj,O,$B\\CVTRES.EXE,rsrc.res

6=*.obj,O,$B\\ML.EXE /c /coff /nologo /I"$I",*.asm

7=0,0,"$E\\OllyDbg",5

[MakeFiles]

0=Driver.rap
1=Driver.rc
2=Driver.asm
3=Driver.obj
4=Driver.res
5=Driver.exe
6=Driver.def
7=Driver.dll
8=Driver.txt
9=Driver.lib
10=Driver.mak
11=Driver.hla
12=Driver.com
13=Driver.ocx
14=Driver.idl
15=Driver.tlb
16=Driver.sys

[Resource]
[StringTable]
[Accel]
[VerInf]
[Group]

Group=Added files,Assembly,Resources,Misc,Modules

1=2

2=2

[*ENDDEF*]

[*BEGINTXT*]

Driver.Asm

.386

.model flat, stdcall
option casemap:none
include driver.inc
.const

CCOUNTED_UNICODE_STRING  "\\\Device\\\devVirtToPhys", g_usDeviceName, 4 ;驱动程序名称
CCOUNTED_UNICODE_STRING  "\\\??\\\slVirtToPhys", g_usSymbolicLinkName, 4;符号连接名称

.code

GetPhysicalAddress proc  uses esi edi pInputBuffer,inBufLength,pOutputBuffer,outBufLength
 ;四个参数分别为用户层的输入数据指针,输入长度,输出数据指针,输出长度
 ;你可以在下面的代码中使用。
  ;下面的代码你可能有用
  ; invoke PsGetCurrentProcess  ;取用户层进程 EPROCES
  ; invoke PsGetCurrentThread   ;取用户层线程 EPROCESS
  ;you code
  ;mov eax ,STATUS_BUFFER_TOO_SMALL  ;缓冲区长度不足
  mov eax,STATUS_SUCCESS
  ret
GetPhysicalAddress endp

;:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
;                    调度创建关闭
;:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
DispatchCreateClose proc pDeviceObject:PDEVICE_OBJECT, pIrp:PIRP
;当用户模式用：
; CreateFile， 让设备处理
; CloseHandle，关闭设备句柄
  mov eax, pIrp
  assume eax:ptr _IRP
  mov [eax].IoStatus.Status, STATUS_SUCCESS
  and [eax].IoStatus.Information, 0
  assume eax:nothing
  fastcall IofCompleteRequest, pIrp, IO_NO_INCREMENT
  mov eax, STATUS_SUCCESS
  ret
DispatchCreateClose endp

;:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
;这里是应用程序发出 DeviceIoControl 指令时，驱动处理子程序。
;:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
DispatchControl proc uses esi edi ebx pDeviceObject:PDEVICE_OBJECT, pIrp:PIRP
local status:NTSTATUS
LOCAL pInputBuffer,inBufLength,pOutputBuffer,outBufLength
  ;pIrp
  mov esi, pIrp
  assume esi:ptr _IRP
        ;pIrp_Stack
  IoGetCurrentIrpStackLocation esi
  mov edi, eax
  assume edi:ptr IO_STACK_LOCATION
  ;IoGetCurrentIrpStackLocation 宏取出_IRP 结构的 IO_STACK_LOCATION 的指针(指向 I/O stack)。
  ;这个栈的层次取决于这个 IRP 需要经过多少层的驱动处理。我们的单层驱动程序在这里只有一个值。
   push  [edi].Parameters.DeviceIoControl.InputBufferLength
   pop   inBufLength
         push  [edi].Parameters.DeviceIoControl.OutputBufferLength
         pop   outBufLength
         push [esi].AssociatedIrp.SystemBuffer
         pop  pInputBuffer
         push  [esi].UserBuffer
         pop   pOutputBuffer
   ;判断用户层的驱动控制码，这里设置的是 800h，在 inc 文件里定义
   .if [edi].Parameters.DeviceIoControl.IoControlCode == IOCTL_GET_PHYS_ADDRESS
         invoke GetPhysicalAddress,pInputBuffer,inBufLength,pOutputBuffer,outBufLength
       mov status, eax
  .else
       mov status, STATUS_INVALID_DEVICE_REQUEST
  .endif
  assume edi:nothing
  push status
  pop [esi].IoStatus.Status
  push outBufLength
  pop [esi].IoStatus.Information
  assume esi:nothing
  fastcall IofCompleteRequest, pIrp, IO_NO_INCREMENT
  mov eax, status
  ret
DispatchControl endp

;:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
;                    驱动程序卸载回调函数
;:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
DriverUnload proc pDriverObject:PDRIVER_OBJECT
  invoke IoDeleteSymbolicLink, addr g_usSymbolicLinkName
  mov eax, pDriverObject
  invoke IoDeleteDevice, (DRIVER_OBJECT PTR [eax]).DeviceObject
  ret
DriverUnload endp

.code
;.code INIT
;所有这样标记的代码将被放入 PE 文件的 INIT 节区中，表明是 Discardable 的。
;在驱动程序初始化后，这部分代码就再也用不着了。
;INIT 节区中的代码可以在 DriverEntry 过程返回后被丢弃，系统会自己决定在合适的时候丢弃它。
;:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
;                    驱动入口
;:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
DriverEntry proc pDriverObject:PDRIVER_OBJECT, pusRegistryPath:PUNICODE_STRING
;
local status:NTSTATUS
local pDeviceObject:PDEVICE_OBJECT
  mov status, STATUS_DEVICE_CONFIGURATION_ERROR
        ;创建虚拟设备
  invoke IoCreateDevice, pDriverObject, 0, addr g_usDeviceName, FILE_DEVICE_UNKNOWN, 0, FALSE, addr pDeviceObject
  .if eax == STATUS_SUCCESS
  ;建立符号链接
    invoke IoCreateSymbolicLink, addr g_usSymbolicLinkName, addr g_usDeviceName
    .if eax == STATUS_SUCCESS
      mov eax, pDriverObject
      assume eax:ptr DRIVER_OBJECT;指定 pDriverObject 的参数的类型为 DRIVER_OBJECT 结构
      ;指定用户模式下卸载驱动时的处理地址
      mov [eax].DriverUnload,  offset DriverUnload
      ;指定用户模式调用 CreateFile 时的处理地址
      mov [eax].MajorFunction[IRP_MJ_CREATE*(sizeof PVOID)],offset DispatchCreateClose
      ;指定用户模式调用 CloseHandle 时的处理地址
      mov [eax].MajorFunction[IRP_MJ_CLOSE*(sizeof PVOID)],offset DispatchCreateClose
      ;指定用户模式调用 DeviceIoControl 时的处理地址
      mov [eax].MajorFunction[IRP_MJ_DEVICE_CONTROL*(sizeof PVOID)],offset DispatchControl
      assume eax:nothing
      mov status, STATUS_SUCCESS
    .else
      invoke IoDeleteDevice, pDeviceObject
    .endif
  .endif
  mov eax, status
  ret
DriverEntry endp

end DriverEntry

[*ENDTXT*]

[*BEGINTXT*]

Driver.Inc
;:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
;            要用到的头文件定义
;:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
include w2k\\ntstatus.inc
include w2k\\ntddk.inc
include w2k\\ntoskrnl.inc
include w2k\\w2kundoc.inc

includelib \\RadASM\\masm32\\lib\\w2k\\ntoskrnl.lib

include \\RadASM\\masm32\\Macros\\Strings.mac

;控制代码
IOCTL_GET_PHYS_ADDRESS  equ CTL_CODE(FILE_DEVICE_UNKNOWN, 800h, METHOD_BUFFERED, FILE_READ_ACCESS + FILE_WRITE_ACCESS)

[*ENDTXT*]

[*ENDPRO*]
```

实现的效果如下图：

![](img/5b2ac254395d90b5b886f97c49071e28.jpg)

![](img/7e5528e74e3a3c4c3d0a5d4829c21d4f.jpg)

![](img/d39a34b70c36ce35a645d6317eb90b66.jpg)

![](img/0c55816e681e24cf40f0999d48c0af6c.jpg)
