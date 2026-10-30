
包含了没有任何脏页的视图的`SHARED_CACHE_MAP`结构体被保存在全局链表`CcCleanSharedCacheMapList`中。`SharedCacheMapLinks`字段用于组织这两种队列（dirty or clean）。

`SectionSize`字段决定了通过`SHARED_CACHE_MAP`映射的区段的大小。

`InitialVacbs`字段是一个内建的4 VACB指针数组，它用于映射那些小于1MB的文件区段。如果区段大小超过了1MB，一个128 VACB的指针数组会被分配并存储在`Vacbs`字段中，它指向了当前可以描述文件到32MB（也就是`128*256K`）大小的VACBs。如果区段尺寸超过32MB，则指针数组的128个指针都用来指向另一个128 VACB指针数组。这种额外的层级设计使得区段大小可以达到4GB（`128*128*256K`）。一共可以有7层VACB，尺寸也就是（`128^7*256K`），这将远高于缓存管理器提供的最大区段尺寸（2^63）。

函数`CcCreateVacbArray()`层级了VACB数组，所有的VACB数组都由一个推锁（`VacbLock`字段）保护。

`PrivateList`是一个链表的头，它用于保存和每个文件打开实例，也就是`FILE_OBJECT`相关联的`PRIVATE_CACHE_MAP`结构体。`PRIVATE_CACHE_MAP.PrivateLinks`字段用于形成链表。

调试命令`!fileobj`可以显示关于`SECTION_OBJECT_POINTER`结构体的信息，它包含一个指向`SHARED_CACHE_MAP`的指针。

### `nt!\_PRIVATE_CACHE_MAP`

缓存管理器执行一个文件的智能预阅读来提升性能。这些预阅读在每个特定文件打开的实例上都独立执行。和每个文件打开实例相关联的`PRIVATE_CACHE_MAP`结构体，保存了一个上次文件阅读操作的历史记录，它被缓存管理器用于执行智能预阅读操作。

`FILE_OBJECT.PrivateCacheMap`指向了和文件打开实例相关联的`PRIVATE_CACHE_MAP`结构体。当该文件缓存被激活时该字段由`CcInitializeCacheMap()`初始化，同样当缓存被清掉时则通过`CcUninitializeCacheMap()`完成。

`FileObject`字段，指向了和`PRIVATE_CACHE_MAP`相关联的`FILE_OBJECT`。预阅读操作仅在`FILE_OBJECT.Flags`字段的`FO_RANDOM_ACCESS`位未置位时才会执行。

`SHARED_CACHE_MAP.PrivateList`指向所有打开的特定文件的实例的`PRIVATE_CACHE_MAP`结构体。特定文件的打开实例的`PRIVATE_CACHE_MAP`结构体通过`PrivateLinks`字段链在一起。

`FileOffset1`,`FileOffset2`,`BeyondLastBye`,`BeyondLastByte2`四个字段一起用于决定在特定`FILE_OBJECT`对应的文件上阅读的模式。缓存管理器函数`CcUpdateReadHistory()`用来更新这些读操作的计数。

`Flags.ReadAheadEnabled`字段决定了预读操作对该文件打开实例来说是否是需要的。`Flags.ReadAheadActive`字段由`CcScheduleReadAhead()`设置，标志了`ReadAheadWorkItem`字段的读工作例程当前是否是活跃的（active）。

调试命令`!fileobj`显示了关于`PRIVATE_CACHE_MAP`的信息。

APIs：

- `CcInitializeCacheMap()`

- `CcUninitializeCacheMap()`

- `CcIsFileCached()`

### `nt!\_SECTION_OBJECT_POINTERS`

`SECTION_OBJECT_POINTERS`和文件对象相关联，它指向了文件映射以及该文件的缓存相关信息。一个单一文件可以有2个分离的映射，一个为可执行文件，另一个为数据文件。

`DataSectionObject`字段指向了控制区域，它是一种服务于内存管理器和文件系统之间的一个链接，它用于内存映射数据文件。

`ImageSectionObject`字段指向另一个控制区域结构，它用于内存映射可执行文件。一个数据映射由单一连续的具有相同保护属性的VA组成，一个映像映射则由可执行体不同的区段映射的多个不同保护属性的区域组成。

`SharedCacheMap`字段指向了文件的`SHARED_CACHE_MAP`结构，它描述了该文件被缓存在那里，缓存了哪些部分。

上面提到的`SECTION_OBJECT_POINTERS`结构体中的字段，是由内存管理器和文件系统驱动配合该文件对象的`SECTION_OBJECT_POINTERS`设置的。

APIs：

- `CcFlushCache()`

- `CcPurgeCacheSection()`

- `CcCoherencyFlushAndPurgeCache()`

- `MmFlushImageSection()`

- `MmForceSectionClosed()`

- `MmCanFileBeTruncated()`

- `MmDoesFileHaveUserWritableReferences()`

- `CcGetFileObjectFromSectionPtrsRef()`

- `CcSetFileSizes()`

> 译者注：本文延续[上一篇](https://bbs.pediy.com/thread-223858.htm)。由于下篇中的某些数据结构译者本人并不熟悉，所以译文可能会有一些蹩脚的地方。如果你发现了错误，烦请及时指出，玉涵百拜。

## I/O管理器

### `nt!_IRP`

`IRP`表示一个I/O请求包结构体，它用来封装执行一个特定I/O操作所需要的所有参数以及I/O操作的状态。`IRP`的表现也类似于一个线程独立调用栈因此它可以从一个线程传递到另一个线程，也可以通过驱动实现的队列传递给一个DPC例程。IRPs是Windows异步I/O处理模型的关键所在，应用程序可以发出一连串的I/O请求并继续执行其他代码，而与此同时I/O请求会被驱动或硬件设备处理。这种异步模型允许相当大的吞吐量且可以实现最佳资源利用。IRPs通过I/O管理器组件分配，由Win32 I/O APIs进行操纵。在内核中IRPs也可以通过设备驱动为I/O请求而分配。IRPs流穿过驱动的栈，在这里驱动在传递IRP给下方的驱动之前会先增加它自身的值到IRP上，这是通过调用`IoCallDriver()`完成的。一般来说在最下方的驱动栈上会通过调用`IoCompleteRequest()`来完成该IRP，该调用会引起栈上的每个驱动被通知I/O已完成并给这些驱动一个在此IRP上执行后向处理操作的机会。

IRPs包含一个固定大小的头部，即IRP数据结构和一个可变的堆栈位置，存储在`StackCount`字段，其中每个堆栈位置都通过`IO_STACK_LOCATION`结构来表示。IRPs必须包含足够多的堆栈位置，该数量不应少于设备栈上互相叠罗汉的设备对象个数，这将用于处理IRP。驱动栈上的设备对象数量存储在`DEVICE_OBJECT.StackSize`字段。IRPs一般由支持固定大小分配器的前向链表分配。因此，由I/O管理器分配的IRPs有10或4个I/O堆栈位置，这取决于IRP目标所在的堆栈上的设备对象数量。

一些IRPs通过`ThreadListEntry`字段被链入到线程中，链表头为`ETHREAD.IrpList`。

`Tail.Overlay.ListEntry`字段被驱动用来保存内部队列的IRP，典型的固定在一个有`LIST_ENTRY`结构的设备扩展结构体中，它由驱动创建`DEVICE_OBJECT`时创建。

`Tail.CompletionKey`在IRP链入到一个I/O完成端口时被使用。

调试器命令`!irp`将显示一个IRP的细节描述。`!irpfind`命令可以通过扫描non-paged pool来找到系统中所有的或者特定的IRPs集合。

APIs：

- `IoAllocateIrp()`

- `IoBuildDeviceIoControlRequest()`

- `IoFreeIrp()`

- `IoCallDriver()`

- `IoCompleteRequest()`

- `IoBuildAsynchronousFsdRequest()`

- `IoBuildSynchronousFsdRequest()`

- `IoCancelIrp()`

- `IoForwardAndCatchIrp()`

- `IoForwardIrpSynchronously()`

- `IoIsOperationSynchronous()`

- `IoMarkIrpPending()`

### `nt!_IO_STACK_LOCATION`

`IO_STACK_LOCATION`包含有关一个I/O操作的信息，该操作要求在一组驱动中执行一个特定的驱动程序。IRP包含多个内嵌的I/O堆栈位置，它们都是在IRP被分配时分配的。设备栈上有多少个驱动IRP中就至少有多少个I/O堆栈位置。I/O堆栈位置归设备所有，它们逆序出现在设备堆栈上，也就是说堆栈上最顶层的设备拥有最底层的堆栈位置，反之亦然。I/O管理器负责填充最顶层设备的I/O堆栈位置，每个设备都负责填充链中下一个设备的I/O堆栈位置。

`Parameters`字段是一个多重结构体的共用体，每个结构代表了一个对应驱动必须执行的I/O操作。共用体`Parameters`中特定结构的选择依赖于`MajorFunction`字段。该字段可用的值由`IRP_MJ_XXX`在wdm.h中定义。具体的主函数（major function）还拥有和它们相关联的副函数（minor function）。这些副函数的数量存储在`MinorFunction`字段中。例如`IRP_MN_START_DEVICE`是一个和主函数`IRP_MJ_PNP`相关的副函数码。

APIs：

- `IoGetCurrentIrpStackLocation()`

- `IoGetNextIrpStackLocation()`

- `IoCopyCurrentIrpStackLocationToNext()`

- `IoSkipCurrentIrpStackLocation()`

- `IoMarkIrpPending()`

### `nt!_DRIVER_OBJECT`

`DRIVER_OBJECT`表示一个加载到内存中的驱动映像。I/O管理器会在驱动加载到内存前创建驱动对象，此后`DriverEntry()`例程会收到一个指向该驱动对象的指针。驱动被卸载出内存后驱动对象的释放与此相似。

`MajorFunction`字段是个数组，每个元素指向一个由驱动所知的分发入口指针提供的函数。这些入口指针被I/O管理器用来分发IRPs给驱动处理。

`DriverName`字段包含对象管理器命名空间内驱动对象的名称。

`DriverStart`字段是内核虚拟地址空间中驱动加载的地址，`DriverSize`包含驱动映射的字节数，按最近页边界向上取整。

`FastIoDispatch`指向了`FAST_IO_DISPATCH`类型的结构，它包含文件系统驱动提供的例程指针。

`DriverSection`指向一个`LDR_DATA_TABLE_ENTRY`类型的数据结构，供加载器在内存中找到驱动映像。

调试器命令`!drvobj`显示了一个驱动对象的信息。

APIs：

- `IoAllocateDriverObjectExtension()`

- `IoGetDriverObjectExtension()`

### `nt!_DEVICE_OBJECT`

`DEVICE_OBJECT`用于表示一个系统中的逻辑或物理设备。和驱动对象不同的是，驱动对象在驱动加载前由I/O管理器创建，而设备对象则是由驱动本身来创建。I/O请求的目标总是设备对象而不是驱动对象。一个指向设备对象的指针会传递给驱动的分发例程以识别哪个设备对象是此I/O请求的目标。

驱动对象在创建时怀有一个设备链表。该链表固定在`DRIVER_OBJECT.DeviceObject`中，使用`NextDevice`字段来把所有的设备驱动链在一起。设备对象，同样的，有一个`DriverObject`字段用以指回拥有它的驱动对象。尽管设备对象是系统定义的数据结构它们也可以拥有驱动特定的扩展结构。该扩展数据结构也一起放在设备对象中，它在non-paged pool中分配，大小是一个特定的声明的尺寸，而用于指向扩展结构的指针就是设备对象的`DeviceExtension`字段。

设备对象可以互相叠罗汉来形成一组设备对象。`StackSize`字段标志了有多少个设备对象在该设备对象的下方。该字段也被I/O管理器用来为IRPs分配合适数量的I/O堆栈位置。`CurrentIrp`和`DeviceQueue`字段只有当驱动为设备对象使用系统管理的I/O时才有用，这非常罕见，因此`CurrentIrp`字段在大多数情况下常常被设置成NULL。`AttachedDevice`指向了下一个设备栈上更高层的设备对象。

调试器命令`!devobj`可以显示一个设备对象的信息。

APIs：

- `IoCreateDevice()`

- `IoDeleteDevice()`

- `IoCreateDeviceSecure()`

- `IoCreateSymbolicLink()`

- `IoCallDriver()`

- `IoCreateFileSpecifyDeviceObjectHint()`

- `IoAttachDevice()`

- `IoAttachDeviceToDeviceStack()`

- `IoDetachDevice()`

- `IoGetAttachedDevice()`

- `IoGetAttachedDeviceReference()`

- `IoGetLowerDeviceObject()`

- `IoGetDeviceObjectPointer()`

- `IoGetDeviceNumaNode()`

### `nt!_DEVICE_NODE`

`DEVICE_NODE`用于表示一个已经被PnP管理器枚举到的物理或逻辑设备。设备节点（nodes）是电源管理和PnP操作的目标。系统中完整的硬件设备树源于具有层级结构的设备节点。设备节点有一个父子与兄弟的关系网。用户模式在CfgMgr32.h/CfgMgr32.lib定义的配置管理器APIs用于处理这些设备节点。

`DEVICE_NODE`结构体的`Child`,`Sibling`,`Parent`以及`LastChild`字段用于存放系统中所有的设备节点，形成一个层级结构。设备节点包含至少一个由`PhysicalDeviceObject`字段指向的物理设备对象（PDO）和一个功能设备对象（FDO），以及一到多个过滤设备对象（FDOs）。`ServiceName`字段指向了一个字符串，标志了创建FDO并驱动该设备的功能驱动。`InstancePath`指向了一个字符串，该字符串唯一的特定标志一个设备实例，系统中可以有多个相同的设备实例。`State`,`PreviousState`,`StateHistory`字段的组合用来确定设备节点在其当前状态设定之前所经历的状态。

调试器命令`!devnode`可以显示`DEVICE_NODE`结构体的信息。`!devstack`命令显示了所有的设备对象，它们是单一devnode的一部分。

### `nt!_FILE_OBJECT`

`FILE_OBJECT`表示了一个打开的设备对象实例。文件对象会在一个用户模式进程调用`CreateFile()`或本地（native）API`NtCreateFile()`或内核模式驱动调用`ZwCreateFile()`时被创建。多个文件对象可以指向一个设备对象除非该设备被标记被设置为排他属性，这是通过设置`DEVICE_OBJECT`标记中的`DO_EXCLUSIVE`位实现的。

`DeviceObject`字段指向了打开该文件对象实例的设备对象。`Event`字段包含一个内嵌的事件结构体，它用于阻塞一个在设备对象上已经请求过异步I/O操作的线程以便于其所有的驱动程序执行异步IO。

`FsContext`和`FsContext2`被文件系统驱动（FSDs）所使用，它们保存文件对象特定的上下文信息。当被一个文件系统驱动使用时，`FsContext`字段指向了一个类型为`FSRTL_COMMON_FCB_HEADER`或`FSRTL_ADVANCED_FCB_HEADER`的结构体，它包含一个文件或流的信息。多重`FILE_OBJECT`的`FsContext`字段表示相同的文件（或流）的打开实例指向同一个文件控制块（FCB）。`FsContext2`字段指向了一个缓存控制块，FSD用它来存储有关文件或流的特定实例信息。

`CompletionContext`,`IrpList`以及`IrpListLock`用于文件对象和I/O完成端口相关联的场景。`CompletionContext`字段由`NtSetInformationFile()`初始化，调用参数为信息类`FileCompletionInformation`。`CompletionContext.Port`字段指向一个KQUEUE类型的结构体，它包含一个已完成并等待被取回的IRP链表。`IoCompleteRequest()`通过字段`IRP.Tail.Overlay.ListEntry`查询该链表中的IRP。

调试命令`!fileobj`显示了一个文件对象的信息。

APIs：

- `IoCreateFile()`

- `IoCreateFileEx()`

- `IoCreateFileSpecifyDeviceObjectHint()`

- `IoCreateStreamFileObject()`

- `IoCreateStreamFileObjectEx()`

- `ZwCreateFile()`

- `ZwReadFile()`
