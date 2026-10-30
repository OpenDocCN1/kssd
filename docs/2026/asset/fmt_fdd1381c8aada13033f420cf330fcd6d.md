
# [原创]初学Windows内核漏洞之瑞星HookCont.sys

# 一.前言

该漏洞存在于瑞星2010 `HookCont.sys`驱动中，版本低于或者等于23.0.0.5的`HookCont.sys`都存在该漏洞。

|  |  |
| --- | --- |
| **参考资料** | 《0day安全：软件漏洞分析技术》 |
| **实验环境** | Win XP SP3 |
| **实验工具** | IDA Pro，WinDbg, VS2017 |

# 二.漏洞分析

## 1.WinDbg

驱动装载以后，首先在WinDbg中查看驱动装载情况。

```
kd> lm

start    end        module name

804d8000 806e6000   nt         (pdb symbols)          d:\xpsymbols\ntkrpamp.pdb\7075F995A48A414F8F7BE9A1E0240F821\ntkrpamp.pdb

806e6000 80706d00   hal        (deferred)             

b22a5000 b22a7a00   vmmemctl   (deferred)             

b22a9000 b22ab200   HookCont   (deferred)             

b24a1000 b24b8900   dump_atapi   (deferred)             

b24b9000 b24e9000   BAPIDRV    (deferred)
```

可以看到`HookCont`被装载到0xB22A9000到0xB22AB200内存区域中。接着再看看分发函数的情况

```
kd> !drvobj HookCont 2

Driver object (81b26b98) is for:

*** ERROR: Module load completed but symbols could not be loaded for HookCont.sys

 \Driver\HookCont

DriverEntry:   b22aa066	HookCont

DriverStartIo: 00000000	

DriverUnload:  00000000	

AddDevice:     00000000	

Dispatch routines:

[00] IRP_MJ_CREATE                      b22a9c3a	HookCont+0xc3a

[01] IRP_MJ_CREATE_NAMED_PIPE           b22a9f62	HookCont+0xf62

[02] IRP_MJ_CLOSE                       b22a9c3a	HookCont+0xc3a

[03] IRP_MJ_READ                        b22a9f62	HookCont+0xf62

[04] IRP_MJ_WRITE                       b22a9f62	HookCont+0xf62

[05] IRP_MJ_QUERY_INFORMATION           b22a9f62	HookCont+0xf62

[06] IRP_MJ_SET_INFORMATION             b22a9f62	HookCont+0xf62

[07] IRP_MJ_QUERY_EA                    b22a9f62	HookCont+0xf62

[08] IRP_MJ_SET_EA                      b22a9f62	HookCont+0xf62

[09] IRP_MJ_FLUSH_BUFFERS               b22a9f62	HookCont+0xf62

[0a] IRP_MJ_QUERY_VOLUME_INFORMATION    b22a9f62	HookCont+0xf62

[0b] IRP_MJ_SET_VOLUME_INFORMATION      b22a9f62	HookCont+0xf62

[0c] IRP_MJ_DIRECTORY_CONTROL           b22a9f62	HookCont+0xf62

[0d] IRP_MJ_FILE_SYSTEM_CONTROL         b22a9f62	HookCont+0xf62

[0e] IRP_MJ_DEVICE_CONTROL              b22a9d2c	HookCont+0xd2c

[0f] IRP_MJ_INTERNAL_DEVICE_CONTROL     b22a9fa4	HookCont+0xfa4

[10] IRP_MJ_SHUTDOWN                    b22a9f62	HookCont+0xf62

[11] IRP_MJ_LOCK_CONTROL                b22a9f62	HookCont+0xf62

[12] IRP_MJ_CLEANUP                     b22a9f62	HookCont+0xf62

[13] IRP_MJ_CREATE_MAILSLOT             b22a9f62	HookCont+0xf62

[14] IRP_MJ_QUERY_SECURITY              b22a9f62	HookCont+0xf62

[15] IRP_MJ_SET_SECURITY                b22a9f62	HookCont+0xf62

[16] IRP_MJ_POWER                       b22a9f1a	HookCont+0xf1a

[17] IRP_MJ_SYSTEM_CONTROL              b22a9f62	HookCont+0xf62

[18] IRP_MJ_DEVICE_CHANGE               b22a9f62	HookCont+0xf62

[19] IRP_MJ_QUERY_QUOTA                 b22a9f62	HookCont+0xf62

[1a] IRP_MJ_SET_QUOTA                   b22a9f62	HookCont+0xf62

[1b] IRP_MJ_PNP                         804f55ce	nt!IopInvalidDeviceRequest
```

可以看到`DriverEntry`的函数地址是0xB22AA066，根据上面得到的驱动装载地址，可以算出`DriverEntry`函数的偏移地址是0x1066。而驱动偏移0xD2C就是`IRP_MJ_DEVICE_CONTROL`的对应的函数地址。

## 2.IDA Pro

以下是`DriverEntry`中比较有用的两段代码，这两段代码完成对设备的创建，对分发函数的初始化以及符号链接的创建。

```
.text:000110A9                 mov     ebx, ds:RtlInitUnicodeString

.text:000110AF                 push    esi

.text:000110B0                 push    offset DEVICE_NAME ; SourceString

.text:000110B5                 lea     eax, [ebp+DestinationString]

.text:000110B8                 push    eax             ; DestinationString

.text:000110B9                 call    ebx ; RtlInitUnicodeString

.text:000110BB                 mov     esi, [ebp+DriverObject]

.text:000110BE                 lea     eax, [ebp+DeviceObject]

.text:000110C1                 push    eax             ; DeviceObject

.text:000110C2                 push    edi             ; Exclusive

.text:000110C3                 push    edi             ; DeviceCharacteristics

.text:000110C4                 push    8300h           ; DeviceType

.text:000110C9                 lea     eax, [ebp+DestinationString]

.text:000110CC                 push    eax             ; DeviceName

.text:000110CD                 push    edi             ; DeviceExtensionSize

.text:000110CE                 push    esi             ; DriverObject            

.text:000110CF                 call    ds:IoCreateDevice
```

```
.text:000110FD                 push    1Bh

.text:000110FF                 lea     edx, [esi+DRIVER_OBJECT.MajorFunction]

.text:00011102                 pop     ecx

.text:00011103                 mov     eax, offset DispatchCommon

.text:00011108                 mov     edi, edx

.text:0001110A                 rep stosd

.text:0001110C                 mov     eax, offset DispatchCreateAndClose

.text:00011111                 mov     [edx], eax

.text:00011113                 mov     [esi+40h], eax

.text:00011116                 push    offset LINK_NAME ; SourceString

.text:0001111B                 lea     eax, [ebp+SymbolicLinkName]

.text:0001111E                 push    eax             ; DestinationString

.text:0001111F                 mov     dword ptr [esi+70h], offset DispatchIoCtrl

.text:00011126                 mov     dword ptr [esi+90h], offset DispatchPower

.text:00011130                 mov     dword ptr [esi+74h], offset DispatchInternalDeivceControl

.text:00011137                 call    ebx ; RtlInitUnicodeString

.text:00011139                 lea     eax, [ebp+DestinationString]

.text:0001113C                 push    eax             ; DeviceName

.text:0001113D                 lea     eax, [ebp+SymbolicLinkName]

.text:00011140                 push    eax             ; SymbolicLinkName

.text:00011141                 call    ds:IoCreateSymbolicLink
```

而`DEVICE_NAME`和`LINK_NAME`的内容如下

```
.text:00010FDA DEVICE_NAME     dw '\'                  ; DATA XREF: DriverEntry+4A↓o

.text:00010FDC aDeviceHookcont:

.text:00010FDC                 text "UTF-16LE", 'Device\hookcont',0

.text:00010FFC                 db 2 dup(0)

.text:00010FFE ; const WCHAR LINK_NAME

.text:00010FFE LINK_NAME       dw '\'                  ; DATA XREF: DriverEntry+B0↓o

.text:00011000 aDosdevicesHook:

.text:00011000                 text "UTF-16LE", 'DosDevices\hookcont',0

.text:00011028                 db 2 dup(0)
```

那么，根据这个符号名，就可以完成对驱动的打开。

在`DispatchComm`函数中，程序首先会将输入缓冲区，输入缓冲区长度，输出缓冲区长度保存起来。接着会对输出缓冲区的地址的合法性进行判断，如果是非法的地址，则会跳转到函数执行结束的地方执行。如果对于偏移0x60数据不清楚的可以看下这篇：[初学Windows内核漏洞之CVE-2011-2005](https://bbs.pediy.com/thread-270103.htm)。
