
# 很多句话让 XueTr 卸载不了我们的驱动（ObjectType HOOK）

> 原文：[`www.pediy.com/kssd/pediy12/134415.html`](https://www.pediy.com/kssd/pediy12/134415.html)

对于 `ObjectType` `HOOK`，可以参考 `MJ0011` 和 `sudami` 的文章：

*   `ObjectType` `HOOK` 干涉注册表操作(bypass `Icesword`, `gmer`, `NIAP`, etc.)  
    `http://www.xfocus.net/articles/200802/966.html`
*   一种 `Object` `hook` 的思路和实现过程  
    `http://forum.eviloctal.com/thread-33688-1-1.html`

两篇文章的共同点都是去 `HOOK` 系统已经生成的 `object type` 里面的例程，系统怎么去用这些例程呢？仍然以上篇的《一句话让 XueTr 卸载不了我们的驱动》为例，驱动卸载时，函数调用如下：

`kd> kp`

```
ChildEBP RetAddr 
ee5deb30 805b1bde nt!IopDeleteDriver 
ee5deb4c 80523bf1 nt!ObpRemoveObjectRoutine+0xe0 
ee5deb70 804f5778 nt!ObfDereferenceObject+0x5f 
ee5dec14 8057a83d nt!IopUnloadDriver+0x28a 
ee5dec24 8053e6d8 nt!NtUnloadDriver+0xf 
ee5dec24 80500231 nt!KiFastCallEntry+0xf8 
ee5deca0 804f55df nt!ZwUnloadDriver+0x11 
ee5ded48 8057a83d nt!IopUnloadDriver+0xf1 
ee5ded58 8053e6d8 nt!NtUnloadDriver+0xf 
```

`ObpRemoveObjectRoutine` 定义如下：

```c
VOID
ObpRemoveObjectRoutine (
    IN   PVOID   Object,
    IN   BOOLEAN CalledOnWorkerThread
    )
```

`Object` 是我们的驱动对象 `DriverObject`，有如下的代码：

```c
    ObjectHeader = OBJECT_TO_OBJECT_HEADER( Object );
    ObjectType = ObjectHeader->Type;
    if (ObjectType->TypeInfo.DeleteProcedure) {
#if DBG
        KIRQL SaveIrql;
#endif
        ObpBeginTypeSpecificCallOut( SaveIrql );
        if (!CalledOnWorkerThread) {
            ObjectHeader->Flags |= OB_FLAG_DELETED_INLINE;
        }
        (*(ObjectType->TypeInfo.DeleteProcedure))(Object);
        ObpEndTypeSpecificCallOut( SaveIrql, "Delete", ObjectType, Object );
    }
```

可以看到是从 `Object` 取 `ObjectHeader`，然后从 `ObjectHeader` 取 `ObjectType`，从 `ObjectType` 取 `TypeInfo.DeleteProcedure`。没有直接取系统已经生成的 `ObjectType`。

因此，我们可以自己构造一个 `ObjectType`，里面填充自己定义的例程，然后覆盖 `Object` 上面的 `ObjectHeader` 里面的 `ObjectType`，这样系统取到的 `TypeInfo.DeleteProcedure` 就是我们自己定义的例程了。

测试代码如下：

```c
/*
* 作者：KiDebug
* 空间：http://hi.baidu.com/KiDebug/
*/

#include <ntddk.h>

UCHAR  My_OBJECT_TYPE[0x200];

void testUnload(IN PDRIVER_OBJECT DriverObject)
{
}

NTSTATUS testDefaultHandler(IN PDEVICE_OBJECT DeviceObject, IN PIRP Irp)
{
  Irp->IoStatus.Status = STATUS_NOT_SUPPORTED;
  Irp->IoStatus.Information = 0;
  IoCompleteRequest(Irp, IO_NO_INCREMENT);
  return Irp->IoStatus.Status;
}

void DoNothing()
{
}

NTSTATUS DriverEntry(IN PDRIVER_OBJECT DriverObject, IN PUNICODE_STRING  RegistryPath)
{
  ULONG i;
  for (i = 0; i <= IRP_MJ_MAXIMUM_FUNCTION; i++)
    DriverObject->MajorFunction[i] = testDefaultHandler;
  DriverObject->DriverUnload = testUnload;
  __asm
  {
    mov eax,DriverObject
    mov esi,[eax-10h]      //+0x008 Type   +0x018 Body  两者相差 0x10
    lea edi,My_OBJECT_TYPE
    mov ecx,64h        //_OBJECT_TYPE 共 0x190 个字节，0x64 个 DWORD
    rep movsd        //把 Driver 的系统自带的_OBJECT_TYPE 复制到 My_OBJECT_TYPE 里面
    lea ebx,My_OBJECT_TYPE
    mov ecx,DoNothing       
    mov [ebx+98h],ecx     //+0x038 DeleteProcedure
    mov [eax-10h],ebx     //把 My_OBJECT_TYPE 覆盖成自己 DriverObject 头上的 Type
  };

  return STATUS_SUCCESS;
}
```

用 `InstDrv.exe` 加载编译后的驱动，依次点击安装、启动，然后用 `XueTr` 查看“驱动模块”，可以看到 `test.sys`，可以在右键菜单中点击“卸载驱动（危险）”，但刷新后 `test.sys` 依然还在。

把 `My_OBJECT_TYPE` 覆盖掉 `XueTr.sys` 驱动对象上面的 `ObjectHeader` 里面的 `ObjectType`，`XueTr` 功能没影响，能干啥干啥。

这样的 `HOOK` 用 `XueTr` 查不到，可以推广到其他的对象中去，比如说注册表、进程等等。

既然能加载驱动，能进内核了，那么神马都是浮云了：看到什么就可以 `HOOK` 什么，反过来看到 `HOOK` 了什么，就可以反 `HOOK` 什么。
