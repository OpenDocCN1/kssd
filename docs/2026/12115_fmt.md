
# [翻译]Rolling Your Own

在这篇文章中我将描述如何自己构造`IRP`还有`I/O`管理器提供一些分配和管理`IRP`的例程。

经常听到NT驱动程序开发者问的问题是如何在他们自己所驱动中执行一个I/O操作。这个问题可以总结为以下两点：当仅可用到一个文件对象但是`ZwXXX`例程需要一个文件句柄时如何处理I/O；为什么`ZwCreateFile`返回的句柄不能用在他们的驱动中。实际的问题是如何在他们的驱动中处理I/O操作，典型的多线程上下文中。

当然，这有很多种可行的方法解决这个问题，但一个重要的方法是在你的驱动中自己构造`IRP`。这个`IRP`包含一个I/O操作在任意上下文中完成所需要的任何信息。`IRPs`依赖于在任意上下文中都有效的`FILE_OBJECTs`和`DEVICE_OBJECTs`，而不是只在特定进程上下文中有效的`File Handle`。这篇文章描述和示范了很多方法，每一种方法都是基于DDK的，你不必使用没有文档化的Kernels APIs。

有很多原因使你需要构建一个`IRP`。可能的原因有与文件系统交互执行一个文件的I/O操作，或者是文件系统支持的内核特征。也许你的驱动要增加一个存在设备的功能，如处理NT容错设备(`FTDISK`)。也许你有两个协作的驱动需要要它们之间通信。或者也许是你在Windows NT上执行一个物理文件系统需要和Media Driver或Transport Driver交互。不管什么原因，在Windows NT下完成这些任务最好的方法是自己构建`IRPs`。

## Allocation

可以用两种方式分配`IRP`。最简单的方法是调用`IoAllocateIrp()`。I/O管理器将用适当的I/O栈单元(你在调用中指定的)分配一个`IRP`，这是最简单的并且是大多数分配I/O请求的方法。警告注意：如何你调用`IoAllocateIrp()`不用调用`IoInitializeIrp()`。在这点上DDK文档导致许多无辜的受害者误入歧途。

你的驱动可能想要创建自己的`IRP`。一个方法是用`ExAllocatePool()`在非分页内存中分配内存。你通过`IoInitializeIrp()`初始化`IRP`的形式。你的驱动可以通过`IoSizeOfIrp()`计算出分配的`IRP`的大小。只要你在非分页内存中分配了一个`IRP`，你就能调用`IoInitializeIrp()`设置`IRP`中的域。请看下面的例子：

```c
PIRP MyAllocateIrp(CCHAR NumberOfStackLocations) {
    USHORT IrpSize = IoSizeOfIrp(NumberOfStackLocations);
    Irp = ExAllocatePool(NonPagedPool, IrpSize);
    if (!Irp) {
        return 0; // failureContent provided by OSR Open Systems Resources, Inc.
    }
    IoInitializeIrp(Irp, IrpSize, NumberOfStackLocations);
    return Irp;
}
```

一般地，驱动依赖于I/O管理器分配和管理`IRP`。然而这些实例使你的驱动能更有效的分配和管理自己的`IRP`。

当NT启动的时候，I/O管理器建立两个旁视列表：一个对应于一个I/O栈单元的`IRP`，另一个对应于四个I/O栈单元的`IRP`。当你分配大栈单元的`IRP`或旁视列表为空时，I/O管理器在非分页内存中分配新的`IRP`。如果你知道你将使用的`IRP`多于四个栈单元，你能通过在你自己的自由列表中保持`IRP`提高一些效率。换句话说，你的驱动在第一次运行时能创建一个`IRP`池并且保存它在一个私有列表中。当你的驱动需要`IRP`时能在这个列表中分配，当I/O操作完成时能返回到这个列表中，从而消除了间接的分配和释放内存池。我们已经在OSR网站上的`roll.c`展示了这方面的例子代码。

你怎么确信I/O管理器将`IRP`返回到你的驱动让它能返回到你的旁视列表中？简单的方法是使用I/O完成例程。下面展示了I/O完成例程如何实际的工作即使你驱动创建的`IRP`没有栈单元也能注册一个完成例程。

这是为什么？如果你考虑一下完成例程如何使用，你就会认识到底层驱动被调用不需要完成例程(毕竟，这将是驱动完成I/O请求)。所以，底层的I/O栈单元可以用来存储下层驱动的完成例程。继续这个处理直到驱动的顶层将给我们一个能把握的额外的完成例程，这个例程对原始的`IRP`创建者是有效的。

对于`IRP`的创建者，设置I/O完成例程等同于中间层驱动的完成例程(简单的调用`IoSetCompletionRoutine()`)。我们将在这篇文章在后面描述如何构建完成例程。

## Building

只要你分配了自己的`IRP`，不管是使用`IoAllocateIrp()`还是从自己的旁视列表中，你必须初始化I/O请求指示你需要低层驱动的什么服务。当你调用下层驱动执行的时候你的驱动必须做同样的工作。简单的设置下层驱动的参数块。你的驱动要做的额外工作是初始化自己分配的`IRP`的其他域。

- `MdlAddress`：this field will point to the `MDL` containing the data (if any)
- `Flags`：any appropriate flags (c.f., `ntddk.h` for the `IRP_*` flags)
- `AssociatedIrp.SystemBuffer`：any data buffer for this I/O request
- `RequestorMode`：`UserMode` or `KernelMode`. Typically, this is `UserMode` if the arguments being passed should be validated, `KernelMode` otherwise.
- `UserBuffer`：any data buffer for this I/O request
- `Tail.Overlay.Thread`：the `PETHREAD` for the original requestor

当然，其中的某些域对于你的I/O操作可是不是必须的(如：`MdlAddress`, `AssociatedIrp.SystemBuffer`，和`UserBuffer`参数，仅仅是它们中的一个可能被你的驱动使用)。当然，你将用到的域用因I/O操作不同而变化。

`Tail.Overlay.Thread`数据结构仅仅用在明确的设备中，如可移动设备，所以系统知道如何操作"错误的弹出"如当媒体设备没有在驱动中加载的时候abort/retry/cancel对话框的出现。

`IRP`有很多不同的标志，告诉下层驱动如何解释这个I/O请求的内容。

- `IRP_NOCACHE` – data for this I/O request should be read from the actual backing media and not from cache.
- `IRP_PAGING_IO` – the I/O operation in question is performing paging I/O. This bit is used by the Memory Manager.
- `IRP_MOUNT_COMPLETION` – the I/O operation in question is performing a mount operation.
- `IRP_SYNCHRONOUS_API` – the API in question expects synchronous behavior. While synchronous behavior is advised when this bit is set, it is not required.
- `IRP_ASSOCIATED_IRP` – the `IRP` in question is associated with some larger I/O operation.
- `IRP_BUFFERED_IO` – the `AssociatedIrp.SystemBuffer` field is valid
- `IRP_DEALLOCATE_BUFFER` – the system buffer was allocated from pool and should be deallocated by the I/O Manager.
- `IRP_INPUT_OPERATION` – the I/O operation is for input. This is used by the Memory Manager to indicate a page in operation.
- `IRP_SYNCHRONOUS_PAGING_IO` – the paging operation should complete synchronously. This bit is used by the Memory Manager.
- `IRP_CREATE_OPERATION` – the `IRP` represents a file system create operation.
- `IRP_READ_OPERATION` – the `IRP` represents a read operation.
- `IRP_WRITE_OPERATION` – the `IRP` represents a write operation.
- `IRP_CLOSE_OPERATION` – the `IRP` represents a close operation.
- `IRP_DEFER_IO_COMPLETION` – the `IRP` should be processed asynchronously. While asynchronous behavior is advised when this bit is set, it is not required.

小心的使用这些标志设置`IRP`，因为这些标志将对下层驱动处理`IRP`请求产生一个根本的影响。

如上所述，你的驱动还应该设置下一个I/O栈单元。它仅仅发生的处理第一个栈单元的时候。第一个栈单元的指针通过`IoGetNextIrpStackLocation()`得到。这个调用返回下一个将要调用的驱动的栈单元你的驱动要负责初始化这个栈单元的下面这个域：

- `MajorFunction`：the function code for the I/O to be performed
- `MinorFunction`：a minor function code for the I/O. This field should be zero if there is no minor function code.
- `Flags`：any flags needed to modify the behavior of the I/O operation (c.f., `ntddk.h` for the `SL_*` flags.)
- `DeviceObject`：the device to which your driver will pass the `IRP`.

标志域用来修改低层驱动的行为当处理I/O请求的时候。可能的标志如下面所示：表３未画

最后，你的驱动必须初始化I/O操作指定的参数。如Read Write请求，需要初始化偏移，长度等。

## Completion

有时候，当你构建自己的`IRP`时你需要提供一个完成例程。完成例程的特定规则是不明确的。但是如果你错误的使用它将导致系统崩溃。

提供完成例程最重要的原因是你可以重新使用I/O操作。在DDK文档中提到的不太重要的原因是你要释放它。这排除了I/O管理器需要执行I/O完成例程，你通过在你的完成例程中返回`STATUS_MORE_PROCESSING_REQUIRED`告诉I/O管理器停止继续向上返回。

什么时候你不需要使用一个完成例程？当你不需要考虑I/O操作的完成状态，或者你不能在你的完成例程中释放`IRP`。后面的情况没有在DDK文档中指出但是非常重要。典型的，I/O管理器为一个线程创建一个I/O操作，这个`IRP`存储在线程的链表中(`ThreadListEntry`域)。当线程退出的时候允许NT清除`IRP`如果你的驱动有一个完成例程并返回`STATUS_MORE_PROCESSING_REQUIRED`，这个`IRP`可能仍然存放于线程的I/O列表中，这可能产生严重的问题。已经证明I/O管理器的某些函数将创建的`IRP`添加到线程链表中，而其它的函数不添加。那么，当构建你的完成例程时，这将是一个好机会检查你的`IRP`不在线程列表中！

只要你的完成例程返回`STATUS_MORE_PROCESSING_REQUIRED`，I/O管理器将停下来等待更多的处理。那么，你就可以做你想做的任何事情，此时你在I/O操作开始时的线程上下文中，文件句柄和用户地址都是不必须的。第二点，你不用假设你的完成例程在`PASSIVE_LEVEL`被调用，它可能在`DISPATCH_LEVEL`级被调用，原因是你完成例程的驱动可能是一个`DPC`例程。将这些记在脑子里，当你设计你自己的完成例程时，如果你需要在`IRP`完成后有更多的处理你可能需要设置一个工作例程确保它安全。

## Reuse

我们描述了你的驱动如何在旁视列表中保持`IRPs`。当你不在需要`IRP`时你的驱动可以在完成例程中设置将它返回到你的旁视列表中。然而，你可能需要做些额外的工作在`IRP`准备重用的时候。

例如，如果你调用文件系统驱动并指定了一个用户缓冲区(通过设置`Irp->Userbuffer`)，文件系统驱动可以创建一个`MDL`描述这个缓冲区。如果是那样，因为你在负责清除`IRP`，所以你有责任unmapping,unlocking并且释放与`IRP`关联的`MDL`。可以通过`MmGetSystemAddressForMdl()`得到`MDL`的地址，通过`MmUnmapLockedPages()`解除映射并通过`MmUnlockPages()`解除锁定页。

## Short-cuts

现在我们已经描述了如何构建自己的`IRPs`，我们将提到I/O管理器提供的三个短小的调用来使你容易的完成这些。这些I/O管理器提供的函数不如你自己构建`IRPs`的通用性好，但它们能快速的构建一个`IRP`并且你能在你的驱动中完成初始化工作，这些例程是：

- `IoBuildAsynchronousFsdRequest()`
- `IoBuildSynchronousFsdRequest()`
- `IoBuildDeviceIoControlRequest()`

这三个调用都没有初始化`FileObject`参数，因此如果你调用文件系统驱动，你的驱动需要设置这个域。既然你知道如何在你的驱动中构建`IRP`，你能扩充I/O管理器创建的`IRPs`来适用于你自己使用。

前面提到的，在I/O管理器的帮助函数中使用完成例程不是那么容易。在`IoBuildSynchronousFsdRequest()`和`IoBuildDeviceIoControlRequest()`中你不能在你的完成例程中释放`IRPs`，但是在`IoBuildAsynchronousFsdRequest()`中你可以这样做。这是因为前面的两个例程把`IRP`增加到线程的`IRP`链表中。因为I/O管理器不会在线程中移除`IRP`，所以唯一的选择是允许完成请求。

用上面三个函数中的任何一个都可以简单的创建一个`IRPs`，但是对于你的驱动的请求所需要的帮助例程仍有限制。`IoBuildDeviceIoControlRequest()`仅用来构建`IRP_MJ_DEVICE_CONTROL`请求，`IoBuildSynchronousFsdRequest()`仅用来支持`IRP_MJ_INTERNAL_DEVICE_CONTROL`，`IoBuildAsynchronousFsdRequest()`仅对`IRP_MJ_READ`, `IRP_MJ_WRITE`, `IRP_MJ_FLUSH_BUFFERS`和`IRP_MJ_SHUTDOWN`有效。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
