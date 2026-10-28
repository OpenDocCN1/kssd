
```
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
    *((_BYTE *)v4->Tail.Overlay.CurrentStackLocation + 3) |= 1u;
  HIBYTE(DeviceObject->Size) = KfAcquireSpinLock((PKSPIN_LOCK)v3 + 5);
  InterlockedDecrement((volatile LONG *)v3 + 4);
  KfReleaseSpinLock((PKSPIN_LOCK)v3 + 5, HIBYTE(DeviceObject->Size));
  return v4->IoStatus.Status;
}
//----- (000107BA) --------------------------------------------------------
int __stdcall DispatchIoControl(PDEVICE_OBJECT DeviceObject, PIRP irp)
{
  signed int v2; // [EMAIL="eax@1"]eax@1[/EMAIL]
  unsigned int v3; // [EMAIL="edx@1"]edx@1[/EMAIL]
  PIRP v4; // [EMAIL="ebx@1"]ebx@1[/EMAIL]
  int v5; // [EMAIL="esi@1"]esi@1[/EMAIL]
  //struct _IRP::$::$::$::$A02EC6A2CE86544F716F4825015773AC::_IO_STACK_LOCATION *v7; // [EMAIL="eax@1"]eax@1[/EMAIL]
  struct _IO_STACK_LOCATION *v7; // [EMAIL="eax@1"]eax@1[/EMAIL]
  int v8; // [EMAIL="eax@7"]eax@7[/EMAIL]
  v4 = irp;
  v5 = 0;
  v7 = irp->Tail.Overlay.CurrentStackLocation;
  v3 = *((_DWORD *)v7 + 1);
  v2 = *((_DWORD *)v7 + 3);
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
  PDEVICE_OBJECT v3; // [EMAIL="eax@1"]eax@1[/EMAIL]
  v3 = DeviceObject;
  ++Irp->CurrentLocation;
  //Irp->Tail.Overlay.CurrentStackLocation = (struct _IRP::$::$::$::$A02EC6A2CE86544F716F4825015773AC::_IO_STACK_LOCATION *)((char *)Irp->Tail.Overlay.CurrentStackLocation + 36);
  Irp->Tail.Overlay.CurrentStackLocation = (struct _IO_STACK_LOCATION *)((char *)Irp->Tail.Overlay.CurrentStackLocation + 36);
  return IofCallDriver(*((PDEVICE_OBJECT *)v3->DeviceExtension + 1), Irp);
}
// ALL OK, 10 function(s) have been successfully decompiled
```

以上代码虽可编译，但仍有很多中间变量，宏替换代码等，进一步优化即可得到一份符合驱动程序编写规范的源代码，留给有兴趣的朋友完成。

- [`样本.rar`](https://bbs.kanxue.com/attach-download-12619.htm)
- [`Fuzen.rar`](https://bbs.kanxue.com/attach-download-12620.htm) —— Fuzen的开源代码，`msdtx.sys`来自这里
- [`KBLOG_C.rar`](https://bbs.kanxue.com/attach-download-12623.htm) —— 主程序逆向的C代码 VC6.0编译通过
- [`KBDRV_C.rar`](https://bbs.kanxue.com/attach-download-12621.htm) —— hexrays逆向出来的伪代码，win2k下ddk编译通过
- [`KBDRV_IDB.rar`](https://bbs.kanxue.com/attach-download-12622.htm) —— 驱动的IDA 保存文件，汇编代码详细注释
- [`部分汇编代码注释.txt`](https://bbs.kanxue.com/attach-download-12624.htm) —— 主程序重点汇编代码注释

bambooooo

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

**上传的附件：**

- [`样本.rar`](javascript:void(0);)（16.72kb，181次下载）
- [`Fuzen.rar`](javascript:void(0);)（35.05kb，272次下载）
- [`KBDRV_C.rar`](javascript:void(0);)（10.77kb，234次下载）
- [`KBDRV_IDB.rar`](javascript:void(0);)（86.60kb，231次下载）
- [`KBLOG_C.rar`](javascript:void(0);)（783.83kb，268次下载）
- [`部分汇编代码注释.txt`](javascript:void(0);)（53.14kb，234次下载）
