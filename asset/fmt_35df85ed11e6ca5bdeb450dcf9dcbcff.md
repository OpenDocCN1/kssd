
# 学习文件过滤的一点总结

> 原文：[`www.pediy.com/kssd/pediy12/131448.html`](https://www.pediy.com/kssd/pediy12/131448.html)

最近遇到一些朋友，说感觉谭文的那本寒江独钓看不懂。其实不管你做什么都是这样，记得我第一次看老罗的《win32 汇编程序设计》的时候，也是完全不懂，但时间长了，看的多了，也就懂了。谭文的两本书我都买了，而且看了很多遍，感觉写的还是蛮好，如果看懂就多看几遍，哪里看不懂，结合其他的书再看，把不懂的搞懂，这样你就会慢慢懂了！如果这样还不懂，我只能说你基础太烂，还是先把基础知识搞懂吧，别搞驱动了！

记得第一次看他的寒江独钓的时候也是除了第一章可以理解外，其他的不知所云，但看过多遍，又实践过一些，然后又学习过很多相关的基础知识和内容之后，现在又回过头来看他的书，真是有一种说不出的踏实的感觉，觉得写的真的太好了！这里不带“拍马屁”的，我从不搞那个，只是觉得真的可以作为入门的好材料，还有张帆的那本《驱动开发详解》我也是看了很多遍，但还是觉得里面还有很多要学，希望将来能越来越明白，学习就是这样，当你不懂的时候，可能就是某些知识点不懂，把那些知识点捡起来，然后接着学，这样你就会慢慢的有一种恍然大悟的感觉！

好了，说了这么多废话，总结一下：静心去学，多花时间去学，不要今天学这个，明天学那个，人的时间和精力是有限的，走好自己的路就可以了！

下面就把我学习寒江独钓第七章文件系统的过滤与监控的一些总结发给一些需要帮助的人，驱动牛人和暂时不想从事驱动开发的人，可以飘过了！

![微笑](img/c1d8523a7842d7b67917c210ac6dcf18.jpg "微笑")

## 文件系统驱动

文件系统驱动主要生成两类设备：文件系统控制设备，文件系统的卷设备。

*   **文件系统控制设备**：主要任务是修改整个驱动的内部配置。
*   **文件系统的卷设备**：一个卷对应一个逻辑盘。

发送给控制设备的请求（`IRP`），一般是文件系统控制 `IRP`（主功能号为 `IRP_MJ_FILE_SYSTEM_CONTROL`）；发送给卷设备的 `IRP` 一般则是文件操作 `IRP`。过滤的目标最终是为了得到文件操作的 `IRP`，但是控制设备的 `IRP`，一般用来捕获卷设备的生成信息，所以我们要先绑定文件系统的控制设备，达到绑定文件系统的卷设备的目的。

1.  生成自己的一个控制设备，当然必须给控制设备指定名称
2.  设置普通分发函数
3.  设置快速 IO 分发函数
4.  编写一个文件系统变动回调函数，在其中绑定刚激活的文件系统控制设备(动态绑定)
5.  使用 `IoRegisterFsRegistrationChange` 调用注册这个回调函数

### 跳过文件系统识别器

...

## 文件系统控制设备的绑定

### 过滤设备扩展

```c
typedef struct _SFILTER_DEVICE_EXTENSION {
    ULONG TypeFlag;
    
    // 绑定的文件系统设备（真实设备）
    PDEVICE_OBJECT AttachedToDeviceObject;
    
    // 与文件系统设备相关的真实设备（磁盘），这个在绑定时使用
    PDEVICE_OBJECT StorageStackDeviceObject;
    
    // 如果绑定了一个卷，那么这是物理磁盘卷名；否则这是绑定的控制设备名
    UNICODE_STRING DeviceName;
    
    // 用来保存名字字符串的缓冲区
    WCHAR DeviceNameBuffer[MAX_DEVNAME_LENGTH];
    
    // The extension used by other user.
    UCHAR UserExtension[1];
} SFILTER_DEVICE_EXTENSION, *PSFILTER_DEVICE_EXTENSION;
```

### 绑定文件系统控制设备

`SfAttachToFileSystemDevice`

文件系统控制设备已经被绑定，绑定的目的是为了获得发送给文件系统控制设备的文件系统控制请求。这些 `IRP` 的主功能号是 `IRP_MJ_FILE_SYSTEM_CONTROL`，每个主功能号下一般都有次功能号。

从这些控制 `IRP` 中能得到足够的信息，确定一个卷被挂载，这样才有可能去绑定文件系统的卷设备。

当有卷被挂载或解挂载时，`SfFsControl()` 就会被系统回调。现在的任务是在这个函数中获得卷设备的相关信息并对它实行绑定，才能捕获各种针对文件的 `IRP`，从而获得监控各种文件操作的能力。

主功能号为 `IRP_MJ_FILE_SYSTEM_CONTROL` 时，有以下几个不同次功能号的 `IRP` 要处理：

1.  次功能号为 `IRP_MN_MOUNT_VOLUME`，说明一个卷被挂载，应该调用 `SfFsControlMountVolume` 来绑定一个卷
2.  次功能号为 `IRP_MN_LOAD_FILE_SYSTEM`，这个请求比较特殊，它一般出现在文件系统识别器要求加载真正的文件系统时，此时说明前面绑定了一个文件系统识别器，现在应该在这里开始绑定真正的文件系统控制设备了
3.  次功能号为 `IRP_MN_USER_FS_REQUEST`，此时可以从 `irpSp->Parameters.FileSystemControl.FsControlCode` 得到一个控制码。当控制码为 `FSCTL_DISMOUNT_VOLUME` 时，说明是一个磁盘在解挂载

**绑定流程总结：**

1.  生成一个控制设备。当然此前必须给控制设备指定名称
2.  设置分发函数和快速 IO 分发函数
3.  编写一个文件系统变动回调函数，在其中绑定刚激活的文件系统的控制设备，并注册这个回调函数
4.  编写默认的分发函数
5.  处理文件系统控制请求(`IRP` 主功能号为 `IRP_MJ_FILE_SYSTEM_CONTROL`)，在其中监控卷设备的 `Mount` 和 `Dismount`

## 文件系统卷设备的绑定

从 `IRP` 中获得 `VPB` 指针：指针 `irpSp->Parameters.MountVolume.Vpb` 是一个 `VPB`，`VPB` 是卷参数(`Volume Parameter Block`)，一个重要的数据结构，它在这里的主要作用是把实际存储媒介设备对象和文件系统上的卷设备对象联系起来。

下面的代码用 `VPB` 来做一下倒手工作，首先，从 `irpSp` 中获得文件系统卷设备的 `VPB`，然后从 `VPB` 中获得一个存储设备对象：

```c
storageStackDeviceObject = irpSp->Parameters.MountVolume.Vpb->RealDevice
```

以后可以从这个存储设备对象再得到原来的 `VPB`。这里记下存储设备，实际上是为了从存储设备对象找回 `VPB`，再找回文件系统卷设备。

**为什么要进行上面的倒手工作？**

这里的 `IRP` 是一个 `Mount` 请求，而文件系统卷设备对象实际上是这个请求完成之后才可用。因此，在这个请求还没完成之前 `irpSp->Parameter.MountVolume.Vpb->DeviceObject` 是没有意义的，必须等这个 `IRP` 完成后 `irpSp->Parameter.MountVolume.Vpb->DeviceObject` 才是需要绑定的设备对象。

但是这个 `IRP` 传递过程中，`irpSp->Parameter.MountVolume.Vpb` 可能被修改。换句话说，有可能 `IRP` 完成之后，这个指针就已经不是原来那个了，对这种情况的处理，`WDK` 文档有如下说明：

> `IrpSp->Parameter.MountVolume.Vpb` 指向一个被挂载的卷参数块（`VPB`）的指针。支持可移动介质的文件系统，可能替换掉预先传入的这个参数。在这样的文件系统上，在卷被挂载之后，这个指针可能不再有效。过滤这种文件系统的过滤驱动必须按下面的方法使用这个参数：在把 `IRP` 发送到下层驱动之前，保存 `IrpSp->Parameter.MountVolume.Vpb->RealDevice` 的值。这个卷被成功挂载之后，过滤驱动可以用这个存储设备的指针重新获得正确的 `VPB` 指针。

为此，必须先获得 `IrpSp->Parameter.MountVolume.Vpb->RealDevice` 的值保存起来，等待 `IRP` 完成之后，再从 `RealDevice` 中去获得那个 `VPB`。

### 设置完成函数并等待 `IRP` 完成 `SfFsControlMountVolume`

完成函数相应实现 `SfFsControlCompletion`。

### 绑定卷的实现

在 `SfFsControlMountVolume`，调用 `ExInitializeWorkItem` 实现 `SfFsControlMountVolumeCompleteWorker`：

```c
VOID
SfFsControlMountVolumeCompleteWorker (
    IN PFSCTRL_COMPLETION_CONTEXT Context
    )
{
    ASSERT( Context != NULL );
    SfFsControlMountVolumeComplete( Context->DeviceObject, 
                                    Context->Irp,
                                    Context->NewDeviceObject );
    ExFreePoolWithTag( Context, SFLT_POOL_TAG );
}
```

`SfFsControlMountVolumeComplete` -> `SfAttachMountedDevice` -> `SfAttachDeviceToDeviceStack`

经过上面操作，我们就完成了：

*   文件系统设备对象的创建
*   文件系统分发函数的分配
*   文件系统控制设备的绑定
*   文件系统卷设备的绑定

## 文件读/写操作的过滤

设置一个读处理函数：`DriverObject->MajorFunction[IRP_MJ_READ] = SfRead;`

### 设备对象的区分处理

如何判断？绑定 `Volume` 的代码已经在设备扩展中设置了域 `StorageDev`，如果不是(比如控制设备的绑定就没设置过)，那么判断 `StorageDev` 中是否为空，就可以知道这是否是一个文件系统卷设备。由此可见，过滤设备上的设备扩展是非常有用的，实际上，就是用来在绑定时保存任意信息，将来在过滤时能得到这些信息的一个上下文。

```c
PSFILTER_DEVICE_EXTENSION devExt = DeviceObject->DeviceExtension;

if (devExt->StorageDev != NULL)
{....}
```

### 解析读请求中的文件信息

**(1)被操作的是哪个文件**

`IRP` 当前栈空间下有一个文件对象指针，指向一个文件对象，从文件对象中可以得到文件对象的名字，不过在读操作的过程中去获得这个文件对象的名字是很不合理的。

**(2)读文件的偏移量**

```c
PIO_STACK_LOCATION irpsp = IoGetCurrentIrpStackLocation(irp);

LARGE_INTEGER offset;
ULONG length;

offset.QuadPart = irpsp->Parameters.Read.ByteOffset.QuadPart;
```

**(3)读取的文件的长度**

```c
length = irpsp->Parameters.Read.Length;
```

如果是写操作，则偏移量和长度为：

```c
offset.QuadPart = irpsp->Parameters.Write.ByteOffset.QuadPart;
length = irpsp->Parameters.Write.Length;
```

### 读请求的完成

如果是读请求，也有几个不同的次功能号，可以用如下的代码得到：

```c
PIO_STACK_LOCATION irpsp = IoGetCurrentIrpStackLocation(irp);

UCHAR minor_code = irpsp->MinorFunction;
```

主功能号为 `IRP_MJ_READ`，则次功能号为：

*   `IRP_MN_NORMAL`
*   `IRP_MN_MDL`
*   `IRP_MN_MDL_COMPLETE`

## 其它操作的过滤

### 文件对象的生存周期

**从 `IRP` 中获得文件对象的指针**

```c
PIO_STACK_LOCATION irpsp = IoGetCurrentIrpStackLocation(irp);

PFILE_OBJECT file_obj = irpsp->FileObject;
```

文件对象是在主功能号为 `IRP_MJ_CREATE` 的 `IRP` 完成之后诞生的，并在主功能号为 `IRP_MJ_CLOSE` 的 `IRP` 完成之后被销毁的。

**文件的打开和关闭**

对于一个已经存在的文件对象，判断是目录还是文件？

一般的做法是在生成(或者打开)成功时捕获这个信息，并记录在一个哈希表中，这时读者得写一个用来表示哈希表的数据结构。当然为了简便也可以完全不用哈希表，而是简单地使用链表甚至数组，只是注意保证操作时的多线程安全性，把所有认为是目录的文件对象放到一个集合里，那么在以后的过滤中，要判断一个文件对象是否是目录，只要判断它是否在这个集合中即可。

**文件的删除**

`Windows` 对文件的删除操作，第一步是发送一个请求打开文件，打开文件时必须设置为有删除的访问权限：如果打开失败，则直接导致无法删除文件。第二步是发出一个设置请求(主功能号是 `IRP_MJ_SET_INFORMATION`)，表示这个文件将被删除。第三步是关闭文件，关闭文件时被系统删除。

### 路径过滤的实现

**取得文件路径的三种情况：**

1.  在文件打开之前从打开文件的请求中提取路径。换句话说，这个文件并不一定真实存在，只是 `Windows` 在试图去打开(或者生成)某个文件了，也就是从 `FileObject->FileName` 中提取文件路径。
2.  在文件打开 `IRP` 处理结束后获取路径。
3.  在文件过滤其他 `IRP` 时（如改名，查询，设置，读，写），得到文件对象所对应的文件路径。

## 把 `sfilter` 编译成一个静态链接库

```makefile
TARGETNAME=sfilter
TARGETPATH=obj
TARGETTYPE=LIBRARY
DRIVERTYPE=FS
SOURCES=sfilter.c
```

上面只是我学习时的一个总结，没有什么其他的意思，只是发在看雪里给那些需要的人看！

最后感谢看雪这个平台，感谢在看雪认识的所有朋友们，十年前我们确实是一只菜鸟，只有拼命看书，跟前辈学习的份，十年之后，只要我们能坚持下去，我们也会成为前辈，并成为那个写书的人！

![微笑](img/c1d8523a7842d7b67917c210ac6dcf18.jpg "微笑") 贵在坚持！
