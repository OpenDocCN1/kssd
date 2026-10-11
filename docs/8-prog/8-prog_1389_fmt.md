
# 360 的 ARP 防火墙简单分析

> 原文：[`www.pediy.com/kssd/pediy12/130864.html`](https://www.pediy.com/kssd/pediy12/130864.html)

360 的 ARP 防护墙驱动分析：

```c
NTSTATUS __stdcall DriverEntry(PDRIVER_OBJECT DriverObject, PUNICODE_STRING RegistryPath)
{
  if (!golobal_cookies || golobal_cookies == 0xBB40E64E)
  {
    golobal_cookies = (unsigned int)&golobal_cookies ^ KeTickCount.LowPart;
    if (&golobal_cookies == (int *)KeTickCount.LowPart)
      golobal_cookies = 0xBB40E64Eu;
  }
  return main(DriverObject, RegistryPath);
}
```

`windows` 的自带程序好像都有这种类似的入口代码

```c
// 真正的执行入口
NTSTATUS __stdcall main(PDRIVER_OBJECT DriverObject, PUNICODE_STRING RegistryPath)
{
  NTSTATUS result; // eax@4
  PDRIVER_OBJECT Driver_Object; // ebx@7
  NTSTATUS status; // eax@8
  unsigned int status_; // esi@9
  int tcpip_module_list; // esi@14
  struct _IMAGE_INFO img_info; // [sp+4h] [bp-48h]@13
  char tcpip_sys; // [sp+18h] [bp-34h]@14
  UNICODE_STRING SymbolicLinkName; // [sp+20h] [bp-2Ch]@5
  char antiarp_1003; // [sp+28h] [bp-24h]@8
  UNICODE_STRING DeviceName; // [sp+30h] [bp-1Ch]@7
  UNICODE_STRING DestinationString; // [sp+38h] [bp-14h]@5
  ULONG MinorVersion; // [sp+40h] [bp-Ch]@13
  ULONG BuildNumber; // [sp+44h] [bp-8h]@13
  PDEVICE_OBJECT DeviceObject; // [sp+48h] [bp-4h]@7

  if (
    InitSafeBootMode
    || (KeInitializeSpinLock(&SpinLock), InitAllSpinLock(&local_spinlock) < 0)
    || InitPackStruct((struct_a1 *)&unk_14500, 'Pack', 0x123u, 0xC8Au) < 0)
    return 0xC0000001u;

  RtlInitUnicodeString(&DestinationString, L"\\Device\\360AntiArp");
  IoCreateDevice(DriverObject, 256u, &DestinationString, 34u, 0, 0, &::DeviceObject);
  RtlInitUnicodeString(&SymbolicLinkName, L"\\DosDevices\\360AntiArp");

  if (IoCreateSymbolicLink(&SymbolicLinkName, &DestinationString) < 0)
    IoDeleteDevice(::DeviceObject);

  RtlInitUnicodeString(&DeviceName, L"\\Device\\360AntiArp1003");
  Driver_Object = DriverObject;
  result = IoCreateDevice(DriverObject, 0x100u, &DeviceName, 0x22u, 0, 0, &DeviceObject);

  if (result >= 0)
  {
    RtlInitUnicodeString((PUNICODE_STRING)&antiarp_1003, L"\\DosDevices\\360AntiArp1003");
    status = IoCreateSymbolicLink((PUNICODE_STRING)&antiarp_1003, &DeviceName);

    if (status < 0)
    {
      status_ = status;
    LABEL_12:
      IoDeleteDevice(DeviceObject);
      return status_;
    }

    Driver_Object->MajorFunction[14] = (PDRIVER_DISPATCH)ControlDispatch;
    Driver_Object->MajorFunction[0] = (PDRIVER_DISPATCH)ControlDispatch;
    Driver_Object->MajorFunction[2] = (PDRIVER_DISPATCH)ControlDispatch;

    if (!sub_123DA())
    {
      IoDeleteSymbolicLink((PUNICODE_STRING)&antiarp_1003);
      status_ = 0xC0000017u;
      goto LABEL_12;
    }

    img_info.Properties = 0;
    img_info.ImageBase = 0;
    img_info.ImageSelector = 0;
    img_info.ImageSize = 0;
    img_info.ImageSectionNumber = 0;

    PsGetVersion((PULONG)&DriverObject, &MinorVersion, &BuildNumber, 0);
    // //
    // WIN7 和 Vista 版本
    // //

    if (
      DriverObject == (PDRIVER_OBJECT)6
      && (RtlInitUnicodeString((PUNICODE_STRING)&tcpip_sys, L"TCPIP.SYS"),
          (tcpip_module_list = EnumerateSystemMoudledLists(
                                (int)Driver_Object->DriverSection,
                                (PUNICODE_STRING)&tcpip_sys)) != 0))
    {
      GetTcpIpModuleInformation((struct_module_list *)tcpip_module_list, &img_info);
      hookNdisRegisterProtocol((struct_module_list *)(tcpip_module_list + 36), 0, &img_info);
    }
    else
    {
      PsSetLoadImageNotifyRoutine((PLOAD_IMAGE_NOTIFY_ROUTINE)hookNdisRegisterProtocol);
    }

    hookIOCreateDevice();
    result = 0;
  }
  return result;
}
```

通过代码可以知道，其面是一些初始化操作和自身结构体的填充，然后创建设备和设备的符号连接

真正开始位置是

```c
PsGetVersion((PULONG)&DriverObject, &MinorVersion, &BuildNumber, 0);
```

通过版本号是 `6` 就是 `vista` 和 `w7` 的处理过程

同过 `driverobject->DriverSection` 指向系统加载的模块列表，遍历这个链表查找 `TCPIP.sys` 这个模块(代码如下：)

```c
if (
  MaxVersiom == 6
  && (RtlInitUnicodeString((PUNICODE_STRING)&tcpip_sys, L"TCPIP.SYS"),
      (tcpip_module_list = EnumerateSystemMoudledLists(
                            (int)Driver_Object->DriverSection,
                            (PUNICODE_STRING)&tcpip_sys)) != 0))
{
  GetTcpIpModuleInformation((struct_module_list *)tcpip_module_list, &img_info);
  hookNdisRegisterProtocol((struct_module_list *)(tcpip_module_list + 36), 0, &img_info);
}
```

如果系统是 `vista` 和 `w7` 系统调用 `ZwQuerySystemInformation` 遍历查找 `TCPIP.sys` 的模块信息通过然后从查找 `NdisRegisterProtocol` 或者 `NdisRegisterProtocolDriver` 的函数地址然后 `hook` 这两个函数

```c
if (MajorVersion < 6)
  Tcpip_imagebase = GetFuncNameFormNdis(
                      "NdisRegisterProtocol",
                      "NDIS.SYS",
                      &DestinationString,
                      (int)&func_addr,
                      (int)&func_index);
else
  Tcpip_imagebase = GetNdisFuncAddress_Win7(
                      "NdisRegisterProtocolDriver",
                      "ndis.sys",
                      (int)img_info_tcpip->ImageBase,
                      (int)&func_addr,
                      (int)&func_index);
```

如果最大版本不是 `6`

```c
PsSetLoadImageNotifyRoutine((PLOAD_IMAGE_NOTIFY_ROUTINE)hookNdisRegisterProtocol);
```

设置加载模块的回调，这个回调也是查找 `NdisRegisterProtocol` 或者 `NdisRegisterProtocolDriver` 的函数地址然后 `hook` 这两个函数

接下来回调用

```c
bool __cdecl hookIOCreateDevice()
{
  bool result; // eax@1
  STRING Func_IoCreateDevice; // [sp+0h] [bp-Ch]@3
  int module_flags; // [sp+8h] [bp-4h]@1

  result = GetSystemModules((int)&module_flags);

  if (result)
  {
    RtlInitAnsiString(&Func_IoCreateDevice, "IoCreateDevice");
    old_Io_CreateDevice = int HookIOCreateDevice(module_flags, (int)&Func_IoCreateDevice, (int)insteadoffunc);
    result = old_Io_CreateDevice != 0;
  }
  return result;
}
```

这个函数用来 `hook` 函数 `IoCreateDevice`，看一下替换这个函数的 `fake_IoCreateDevice`

```c
int fake_IoCreateDevice(PDRIVER_OBJECT driver, int a2, PVOID Device_Name, int a4, int a5, unsigned __int8 a6, int out_device_object)
{
  int status; // eax@1
  size_t len; // eax@5
  KIRQL v9; // al@7
  int bak_statsu; // [sp+Ch] [bp-8h]@1
  KSPIN_LOCK SpinLock; // [sp+10h] [bp-4h]@7

  status = old_Io_CreateDevice(driver, a2, Device_Name, a4, a5, a6, out_device_object);
  bak_statsu = status;

  if (status >= 0)
  {
    if (driver != (PDRIVER_OBJECT)Self_DeviceObejct)
    {
      if (MmIsAddressValid(Device_Name))
      {
        if (MmIsAddressValid(*((PVOID *)Device_Name + 1)))
        {
          len = wcslen(L"\\Device\\NPF_");
          if (!wcsncmp(*(const wchar_t **)Device_Name + 1, L"\\Device\\NPF_", len))
          {
            if (MmIsAddressValid(driver->MajorFunction[4]))
            {
              KeInitializeSpinLock(&SpinLock);
              v9 = KfAcquireSpinLock(&SpinLock);
              Real_WriteDispatch = (int (__stdcall *)(_DWORD, _DWORD))driver->MajorFunction[4];
              if ((_WORD)NtBuildNumber >= 6000)
                Real_ReadDispatch = (int (__stdcall *)(_DWORD, _DWORD))driver->MajorFunction[3];

              driver->MajorFunction[4] = (PDRIVER_DISPATCH)hookWriteDispatch;
              if ((_WORD)NtBuildNumber >= 6000)
```



```
          driver->MajorFunction[3] = (PDRIVER_DISPATCH)hookReadDispatch;  

          Self_DeviceObejct = (int)driver;  

          KfReleaseSpinLock(&SpinLock, v9);  

        }  

      }  

    }  

  }  

  status = bak_statsu;  

}  

return status;  

}  
```

当一个设备创建成功后。如果设备名称包括`"\\Device\\NPF_"`然后根据系统版本版本号，用对应函数`hook`这个创建设备的`IRP_MJ_READ`和`IRP_MJ_WRITE`例程(疑惑：好像`winpcap`打开的设备名称好像都是这个开头的，难道它只是防止`winpcap`？)

我们看看`IRP_MJ_READ`读例程

```
int __stdcall hookReadDispatch(PDEVICE_OBJECT device_obj, PIRP irp)  

{  

  int result; // eax@4  

  LARGE_INTEGER Interval; // [sp+0h] [bp-Ch]@1  

  int v4; // [sp+8h] [bp-4h]@4  

  Interval = (LARGE_INTEGER)0xFFFFFFFFFFF85EE0ui64;  

  if ( !Real_ReadDispatch )  

    KeBugCheck(0x7900001u);  

  if ( !irp->Tail.Overlay.CurrentStackLocation->FileObject->FsContext )  

  {  

    while ( 1 )  

      KeDelayExecutionThread(0, 0, &Interval);  

  }  

  result = Real_ReadDispatch(device_obj, irp);  

  v4 = result;  

  return result;  

}  
```

替换写的例程是   判断数据链路层协议类型是不是`0x608`，是不是`ARP`的协议，然后获取使用该设备的进程的信息

```
signed int __stdcall GetProcessInfoAndDealWith(PEPROCESS eproc)  

{  

  PSYSTEM_PROCESSES process; // ebx@1  

  PSYSTEM_PROCESSES i; // esi@3  

  ULONG ReturnLength; // [sp+8h] [bp-110h]@2  

  STRING process_name; // [sp+Ch] [bp-10Ch]@5  

  KIRQL NewIrql; // [sp+14h] [bp-104h]@8  

  __int16 v7; // [sp+110h] [bp-8h]@9  

  CHAR v8; // [sp+112h] [bp-6h]@9  

  int v9; // [sp+114h] [bp-4h]@1  

  v9 = golobal_cookies;  

  process = (PSYSTEM_PROCESSES)ExAllocatePoolWithTag(0, 0x50000u, 0x70333630u);  

  if ( !process )  

    return 0;  

  if ( ZwQuerySystemInformation(SystemProcessesAndThreadsInformation, process, 0x50000u, &ReturnLength) )  

  {  

LABEL_12:  

    ExFreePool(process);  

    return 0;  

  }  

  for ( i = process;  

        (PEPROCESS)i->ProcessId != eproc || RtlUnicodeStringToAnsiString(&process_name, &i->ProcessName, 1u);  

        i = (PSYSTEM_PROCESSES)((char *)i + i->NextEntryDelta) )  

  {  

    if ( !i->NextEntryDelta )  

      goto LABEL_12;  

  }  

  memset(&NewIrql, 0, 0x100u);  

  if ( strlen(process_name.Buffer) <= 0x100 )  

  {  

    memcpy(&NewIrql, process_name.Buffer, strlen(process_name.Buffer) + 1);  

  }  

  else  

  {  

    memcpy(&NewIrql, process_name.Buffer, 0xFCu);  

    v7 = *((_WORD *)process_name.Buffer + 126);  

    v8 = process_name.Buffer[254];  

  }  

  RtlFreeAnsiString(&process_name);  

  ExFreePool(process);  

  return IRP_Message_Dispatch(&NewIrql);  

}  
```

`IRP_Message_Dispatch`负责处理是否`arp`欺骗
