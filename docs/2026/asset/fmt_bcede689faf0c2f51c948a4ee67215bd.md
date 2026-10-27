
```text
81b7d8b8 00000000 81b7f888 81bd6040 81bc4030 CLASSPNP!SubmitTransferPacket+0x82 (FPO: [Non-Fpo])
```

IDA打开雨过天晴的boot0驱动`shdbus.sys`,定位到`+0x1056e`处,位于分发函数`IrpInternalDeviceControl`内,代码如下:

![](https://bbs.kanxue.com/upload/attach/200909/57632_w599zei8op89qum.jpg)

```c
NTSTATUS IrpInternalDeviceControl(int DeviceObject, PIRP Irp)
{
  int DeviceExtension, srb, atapi_driver_object, disk_driver_object, srb_cdb, IoControlCode ;
  HANDLE CurrentTID;
  struct _IRP::$::$::$::$A02EC6A2CE86544F716F4825015773AC::_IO_STACK_LOCATION *CurrentStackLocation;
  char OperationCode;

  DeviceExtension = *(DWORD *)(DeviceObject + 0x28);
  CurrentTID = PsGetCurrentThreadId();
  PsGetCurrentProcessId();
  srb = 0;
  if ( g_disk_internal_device_control_dispatch )
  {
    disk_driver_object = *(DWORD *)(g_disk_device_object + 8);
    if ( *(DWORD *)(disk_driver_object + 0x74) != g_disk_internal_device_control_dispatch
      || *(DWORD *)(disk_driver_object + 0x48) != g_disk_internal_device_control_dispatch
       )
    {//
     // #define IRP_MJ_INTERNAL_DEVICE_CONTROL    0x0f
     // #define IRP_MJ_SCSI                       0x0f
     // 雨过天晴会不断检查自己的HOOK点,并恢复之.若2个程序同时在同一点
     // 循环检查并恢复自己的钩子,会导致系统启动后及其缓慢. 而且"雨过天晴"会结束
     // 掉与其竞争的系统线程,导致系统出现异常错误.BSOD.
     //
      *(DWORD *)(disk_driver_object + 0x74) = g_disk_internal_device_control_dispatch;
      *(DWORD *)(disk_driver_object + 0x48) = g_disk_internal_device_control_dispatch;

// disk分发例程往下发的时候,起扩展设备偏移+0x008的地方是atapi.sys的设备对象,"雨过天晴"在此进行验证.
      if ( *(DWORD *)(DeviceExtension + 8) == g_atapi_device_object )
      {
        if ( g_allowed_TID_1_0000006c
          && CurrentTID != *(HANDLE *)g_allowed_TID_1_0000006c
          && g_allowed_TID_2_ffffffff
          && CurrentTID != *(HANDLE *)g_allowed_TID_2_ffffffff
          && g_allowed_TID_3_00000070
          && CurrentTID != *(HANDLE *)g_allowed_TID_3_00000070
          && g_allowed_TID_4_00000240
          && CurrentTID != *(HANDLE *)g_allowed_TID_4_00000240 )
        {
// 不是以上4个系统线程,便会被"雨过"结束掉,并且阻止当前IRP的下发.于是就BSOD了.
          ZwTerminateProcess((HANDLE)0xFFFFFFFF, 0);
denny:
          Irp->IoStatus.Status = 0;
          IofCompleteRequest(Irp, 0);
          return 0;
        }
      }
    }
    atapi_driver_object = *(DWORD *)(g_atapi_device_object + 8);
if ( *(DWORD *)(atapi_driver_object + 0x74) != (DWORD)atapi_Proxy_dispatch ) // 在此处恢复对atapi.sys分发例程的HOOK
      memset((void *)(atapi_driver_object + 0x38), (int)atapi_Proxy_dispatch, 0x6Cu);
  }

  CurrentStackLocation = Irp->Tail.Overlay.CurrentStackLocation;
  IoControlCode = *((DWORD *)CurrentStackLocation + 3);
  srb_cdb = 0;
  if ( IoControlCode == 0x1B0012 || IoControlCode == 0x1B0011 )
  {
    srb = *((DWORD *)CurrentStackLocation + 1);
    srb_cdb = srb + 0x30;
  }
  if ( *(DWORD *)(DeviceExtension + 8) == g_atapi_device_object )
  {
    if ( g_allowed_TID_1_0000006c )
    {
      if ( CurrentTID != *(HANDLE *)g_allowed_TID_1_0000006c )
      {
        if ( !g_allowed_TID_2_ffffffff || CurrentTID != *(HANDLE *)g_allowed_TID_2_ffffffff )
        {
          if ( !g_allowed_TID_3_00000070 || CurrentTID != *(HANDLE *)g_allowed_TID_3_00000070 )
          {
            if ( !g_allowed_TID_4_00000240 || CurrentTID != *(HANDLE *)g_allowed_TID_4_00000240 )
            {
              if ( srb )
              {
                if ( !*(BYTE *)(srb + 2) )
                {
                  if ( srb_cdb )
                  {
                    OperationCode = *(BYTE *)srb_cdb;
                    if ( *(BYTE *)srb_cdb == SCSIOP_WRITE
                      || OperationCode == SCSIOP_SEND
                      || OperationCode == SCSIOP_FLUSH_BUFFER
                      || OperationCode == SCSIOP_WRITE_VERIFY
                      || OperationCode == SCSIOP_READ
                      || OperationCode == SCSIOP_RECEIVE )
                      goto denny;
                  }
                }
              }
            }
          }
        }
      }
    }
  }

  return IrpReadWrite_dep(DeviceObject, Irp);
}
```

(1) 现在调试器中将`jnz` 改成 `Jmp`, 即 `0F 84 à 90 E9`

![](https://bbs.kanxue.com/upload/attach/200909/57632_v350mvcgw32tpoo.gif)

(2) 经过初步分析,雨过天晴大致在`Disk.sys`和`Atapi.sys`层做了过滤,为了验证这一想法,我恢复掉其钩子后,自己写程序不经过文件系统层,构建`IRP`发到`DR0`上读取MBR,看是否成功.现要恢复`Disk.sys`的`IRP_MJ_READ & IRP_MJ_INTERNAL_DEVICE_CONTROL`例程 和 `Atapi.sys`的`IRP_MJ_INTERNAL_DEVICE_CONTROL`例程:

(因为我的程序是自己构建`IRP`,填充`0xf`号控制码,即`isl->MajorFunction = IRP_MJ_SCSI`,然后发送到`DR0`设备上, 那么`IRP`往下走的过程中就会调用`DR0`对应的驱动对象的分发例程,也就是`Disk.sys`的`0xf`号分发例程,而非`IRP_MJ_READ`,所以根据我的程序特性,应该恢复`Disk.sys`的`IRP_MJ_INTERNAL_DEVICE_CONTROL`历程)

```text
kd> ln CLASSPNP!ClassInternalIoControl

(f99e0fc3)   CLASSPNP!ClassInternalIoControl   |  (f99e0fc3)   CLASSPNP!ClassInternalIoControl

Exact matches:

CLASSPNP!ClassInternalIoControl = <no type information>

kd> ln CLASSPNP!ClassReadWrite

(f99dcd9b)   CLASSPNP!ClassReadWrite   |  (f99dcd9b)   CLASSPNP!ClassReadWrite

Exact matches:

    CLASSPNP!ClassReadWrite = <no type information>

kd> ed 81b7df38+0x38+0x3c f99e0fc3

kd> ed 81b7df38+0x38+0xc f99dcd9b

kd> !drvobj \driver\disk 3

…

Dispatch routines:

[00] IRP_MJ_CREATE                      f9785cd6      Shield+0x3cd6

[01] IRP_MJ_CREATE_NAMED_PIPE           f9785cd6 Shield+0x3cd6

[02] IRP_MJ_CLOSE                       f9785cd6      Shield+0x3cd6

[03] IRP_MJ_READ                        f99dcd9b    CLASSPNP!ClassReadWrite

[04] IRP_MJ_WRITE                       f9785cd6      Shield+0x3cd6

…

[0f] IRP_MJ_INTERNAL_DEVICE_CONTROL     f99e0fc3       CLASSPNP!ClassInternalIoControl

…
```

原始的Atapi.sys的分发例程如下:

```text
Dispatch routines:

[00] IRP_MJ_CREATE                      bae6d572      atapi!IdePortAlwaysStatusSuccessIrp

[01] IRP_MJ_CREATE_NAMED_PIPE           804f5282 nt!IopInvalidDeviceRequest
```
