
# 从内核层保护文件不被删除

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-158.htm`](https://www.pediy.com/kssd/pediy09/pediy09-158.htm)

> 前几天发了一篇删除正在运行的程序文件的代码片，其实都平常写 demo 留下的东西，如果大家有兴趣，我会找时间慢慢整理一些出来和大家分享。同样是块 demo 代码，前面有兄弟问为什么不整完整一些，在这里解释一下:  

> 1. 因为要分享是一种思路或技巧，不想被一些有心人不劳而获的 A 走（虽然我也是 A 别人的，呵呵）  

> 2. 关键的，如果发完整的领导就该找我喝茶了，希望能理解  

>   

> 好了，回到正题是来，保护文件不被删除的方法有很多，例如：`FSD Dispatch Hook`、文件过滤驱动、在内核层打开文件占用等等，这里要介绍的是另一种方法，`Inline Hook IofCallDriver` 大法，用这个方法来保护文件确实有点大材小用（它可以做的事情实在太多了，充分发挥想像力吧），并且在性能上可能会有点问题，但它有个优势是可以躲过一些工具的查杀。  

> 它保护文件的基本原理是：  

> 当系统要删除一个文件时，它会向 `ntfs` 或 `fastfat` 驱动发送一个 `MajorFunction` 为 `IRP_MJ_SET_INFORMATION` 的 `IRP`，而用来传递 `Irp` 的函数是 `IoCallDriver`，该函数是一个宏，它再调用 `IofCallDriver`，所以我们只要 `Inline Hook IofCallDriver` 函数就可以拦截删除文件请求。大概步骤如下：  

> 1.`Inline Hook IofCallDriver` 函数  

> 2.判断 `sp->MajorFunction == IRP_MJ_SET_INFORMATION` 如果不相等，则跳出  

> 3.判断 `sp->Parameters.SetFile.FileInformationClass == FileDispositionInformation`，如果不相等，则跳出。  

> 4.判断 `DeviceObject->DriverObject.DriverName` 是否为 `\FileSystem\ntfs` 或 `\FileSystem\fastfat`，因为我们要拦截的是发往 `ntfs` 和 `fastfat` 驱动的 `Irp` 请求，所以如果不相等，则跳出。  

> 5.判断 `sp->FileObject.FileName` 是否是我们要保护的文件（注意：这里取到的不是全路径，需要全路径要另作处理），如果不相等，则跳出  

> 6.上面所有条件都符合时，设置 `Irp->IoStatus.Information = 0;Irp->IoStatus.Status = STATUS_NOT_SUPPORTED;` 并调用 `IoCompleteRequest` 完成 `Irp`，`Irp` 不再往下传了。  

>   

> `IofCallDriver` 函数的原型  

```
NTSTATUS
__fastcall
IofCallDriver(
IN PDEVICE_OBJECT DeviceObject,
IN OUT PIRP Irp
);
```
> 这是一个 `fastcall` 函数，取参数的时候要注意一下。

> 下面是代码片段：

```
#define PROLOG __asm { pushad } __asm { pushfd }
#define RETURN __asm { popfd } __asm { popad } __asm { pop eax } __asm{ mov eax, 0xC000000D } __asm{ ret }
#define EPILOG __asm { popfd } __asm { popad } __asm { ret }

_declspec( naked )
NTSTATUS
__fastcall
IofCallDriver_Hook(
        IN PDEVICE_OBJECT DeviceObject,
        IN OUT PIRP Irp
        )
{
  PROLOG
  __asm
  {
    push ecx
    pop dword ptr g_HookParam.DeviceObject
    push edx
    pop dword ptr g_HookParam.Irp
  }
  if (IsNeedProtect(g_HookParam.DeviceObject, g_HookParam.Irp))
  {
    dprintf("Protect start...\n");
    RETURN
  }
  
Exit0:
  EPILOG

}

int IsNeedProtect(DEVICE_OBJECT *DeviceObject, PIRP Irp)
{
  NTSTATUS status = STATUS_INVALID_PARAMETER;
  int nResult = FALSE;
  DRIVER_OBJECT  *DriverObject = DeviceObject->DriverObject;
  WCHAR      *pwsz = NULL;
  IO_STACK_LOCATION *sp;
  FILE_OBJECT    *FileObject = NULL;

  sp = IoGetNextIrpStackLocation(Irp);
  PROCESS_ERROR(sp);
  
  if (sp->MajorFunction != IRP_MJ_SET_INFORMATION)
  {
    goto Exit0;
  }

  if (sp->Parameters.SetFile.FileInformationClass != FileDispositionInformation)
  {
    goto Exit0;
  }

  FileObject = sp->FileObject;
  __try
  {    
    pwsz = wcsrchr(DriverObject->DriverName.Buffer, L'\\');
    PROCESS_ERROR(pwsz);
    pwsz ++;

    if (_wcsnicmp(pwsz, L"ntfs", 4) &&
      _wcsnicmp(pwsz, L"fastfat", 7))
    {
      goto Exit0;
    }

    dprintf("IofCallDriver: Delete file %ws\n", FileObject->FileName.Buffer);

    pwsz = wcsrchr(FileObject->FileName.Buffer, L'\\');
    if (pwsz)
    {
      pwsz ++;
    }
    else
    {
      pwsz = FileObject->FileName.Buffer;
    }

    if (!_wcsnicmp(pwsz, L"test.exe", 8))
    {
      Irp->IoStatus.Information = 0;
      Irp->IoStatus.Status = STATUS_NOT_SUPPORTED;
      
      IoCompleteRequest(Irp, IO_NO_INCREMENT);
      nResult = TRUE;
      goto Exit0;
    }

  }
  __except(1)
  {
    goto Exit0;
  }

Exit0:
  return nResult;
}
```

> 结束。![](img/b1b6155d5e6bc11d8426f301b224afc3.jpg "124.gif") ![](img/768c8e05987a6949d1bb5f7378212ee0.jpg "狂笑")

`4dd466d0e1e0cc123afbe504da2462ca`
