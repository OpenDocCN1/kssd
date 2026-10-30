
- `ZwWriteFile()`
- `ZwFsControlFile()`
- `ZwDeleteFile()`
- `ZwDeviceIoControlFile()`
- `ZwFlushBuffersFile()`
- `ZwOpenFile()`
- `ZwFsControlFile()`
- `ZwLockFile()`
- `ZwQueryDirectoryFile()`
- `ZwQueryEaFile()`
- `ZwCancelIoFile()`
- `ZwQueryFullAttributesFile()`
- `ZwQueryInformationFile()`
- `ZwQueryVolumeInformationFile()`
- `ZwSetEaFile()`
- `ZwSetInformationFile()`
- `ZwSetQuotaInformationFile()`
- `ZwSetVolumeInformationFile()`
- `ZwUnlockFile()`
- `ZwWriteFile()`

## 对象和句柄

### `nt!_OBJECT_HEADER`

Windows内核中的对象（Object）数据结构用来表示通用的设施比如文件、注册表键、进程、线程、设备等等。它由对象管理器管理，对象管理器是Windows内核的一个组件。所有这样的对象内部都有一个先驱的`OBJECT_HEADER`结构体，它包含对象相关的信息并用来维护对象的生命周期，允许对象有一个专有名字，通过应用访问控制来保护对象，调用对象特定类型方法以及追踪分配器的配额使用。
对象中部署在`OBJECT_HEADER`后方的数据结构与`OBJECT_HEADER`存在着部分的重叠。实际上对象的数据存放位置是从`OBJECT_HEADER`的`Body`字段开始而不是从其尾部开始。
对象头部包含了引用计数`HandleCount`和`PointerCount`，它们被对象管理器用来保存对象直到没有外部的引用指向该对象。`HandleCount`是句柄的数量，`PointerCount`是句柄和内核模式对象引用的数量。
对象头可以由一个可选的对象头引导，类似`OBJECT_HEADER_PROCESS_INFO`,`OBJECT_HEADER_QUOTA_INFO`,`OBJECT_HEADER_HANDLE_INFO`,`OBJECT_HEADER_NAME_INFO`以及`OBJECT_HEADER_CREATOR_INFO`结构，它们描述了关于对象额外的属性。`InfoMask`字段是一个位掩码，它决定了当前是哪一种前面描述的可选头结构。
`SecurityDescriptor`字段指向一个类型为`SECURITY_DESCRRIPTOR`的结构体，它包含了任意访问控制列表（DACL）和系统访问控制列表（SACL）。DACL用于检查进程的tokens是否有访问对象的权限。SACL用于审计访问对象的权限。
内核不能在IRQL高于`PASSIVE_LEVEL`的级别上删除对象。`ObpDeferObjectDeletion()`把对象链在一起，其删除操作需要被延迟到一个链表中，该链表在内核变量`ObpRemoveObjectList`中。`NextToFree`字段为此而生。
`QuotaBlockCharged`字段指向了`EPROCESS.QuotaBlock`的`EPROCESS_QUOTA_BLOCK`结构体，它被`PsChargeSharedPoolQuota()`和`PsReturnSharedPoolQuota()`函数用来追踪一个使用Non-Paged Pool和Paged Pool的特定进程。分配对象时配额总是会被分配。
调试器命令`!object`显示了存储在对象头的信息。`!obtrace`命令显示了根据对象引用trace到的数据。如果对象引用追踪在一个对象上可用，`!obja`命令可以显示任何对象的属性信息。
APIs：
- `ObReferenceObject()`
- `ObReferenceObjectByPointer()`
- `ObDereferenceObject()`

### `nt!_OBJECT_TYPE`

对对象管理器管理的每种类型的对象来说，都有一个“类型对象”结构体用于存储该类型对象的通用属性。这种“类型对象”结构体由`OBJECT_TYPE`表示。Windows 7上有大概42种不同的对象类型结构体。内核变量`ObTypeIndexTable`是一个指针数组，它的每个成员都指向一种对象类型的`OBJECT_TYPE`结构体。对于每种对象类型内核也会保存一个指向相关对象类型结构体的全局变量。例如，变量`nt!IoDeviceObjectType`指向了`DEVICE_OBJECTS`的`OBJECT_TYPE`结构体。
`OBJECT_TYPE`的`TypeInfo`字段指向了一个`OBJECT_TYPE_INITIALIZER`结构体，该结构体包含了对象类型特定的函数，它们被对象管理器用来在对象上执行各种各样的操作。
`CallbackList`字段一个特定对象类型的驱动安装的回调函数列表的头。当前只有进程和线程对象支持回调函数，它们由`TypeInfo.SupportsObjectCallbacks`字段指定。
关键字段包含了池标签(pool tag)，它用于分配该类型的对象。
调试器命令`!object ObjectTypes`用于显示系统中所有的类型对象。

### `nt!_HANDLE_TABLE_ENTRY`

Windows中的进程有它们自己的私有句柄表，它保存在内核虚拟地址空间中。`HANDLE_TABLE_ENTRY`表示了一个进程句柄表中 个体表项。句柄表在分页内存池中分配。当一个进程终止时，函数`ExSweepHandleTable()`关闭该进程句柄表中所有的句柄。
`Object`字段指向了一个对象结构体，比如File, Key, Kevent等，它们的句柄均已创建。
`GrantedAccess`字段是一个类型为`ACCESS_MASK`的位掩码，它决定了对象上特定句柄所允许的操作集合。该字段的值由`SeAccessCheck()`计算，基于调用者（可信访问）的访问请求以及对象安全描述符的DACL中的ACEs。
调试命令`!handle`可以用来查看任何进程的句柄表。`!htrace`命令可以用来显示堆栈上跟踪句柄的数据，如果句柄跟踪可用的话。
APIs：
- `ObReferenceObjectByHandle()`
- `ObReferenceObjectByHandleWithTag()`

## 内存管理

### `nt!_MDL`

`MDL`表示一个内存描述符列表结构，它描述那些已被锁定的用户或内核模式内存。它由一个固定长度的头和可变的页帧数（PFNs）组成。每一页都由MDL描述。
MDL结构包含了它描述的虚拟地址和缓冲区尺寸，对用户模式缓冲区来说它也指向了拥有该缓冲区的进程。设备驱动程序使用MDLs对硬件设备进行编程，执行DMA的传输，同时也映射用户模式缓冲区到内核模式，反之亦可。
某些类型的驱动程序，例如网络栈，在Windows支持的链式MDLs，多个MDL中描述了实质上分散的缓冲区，它们由`Next`字段链在一起。
对MDLs来说那描述了用户模式缓冲区，`Process`字段指向了进程的EPROCESS结构体，它的虚拟地址空间被MDL锁住。
如果被MDL描述的缓冲区被映射到了内核虚拟地址空间，`MappedSystemVa`就指向了内核模式的缓冲区地址。该字段仅在位`MDL_MAPPED_TO_SYSTEM_VA`或`MDL_SOURCE_IS_NONPAGED_POOL`在`MdlFlags`字段中被设置时，才是有效的。
`Size`字段包含MDL数据结构以及MDL后面跟随的整个PFN数组的尺寸。
`StartVa`字段和`ByteOffset`字段一起定义了MDL中被锁住的原始缓冲区的起始位置。`StartVa`指向了页起始，`ByteOffset`包含了从`StartVa`计算缓冲区实际起始地址的偏移字节。
`ByteCount`字段描述了MDL锁住的缓冲区尺寸。
APIs：
- `IoAllocateMdl()`
- `IoBuildPartialMdl()`
- `IoFreeMdl()`
- `MmInitializeMdl()`
- `MmSizeOfMdl()`
- `MmPrepareMdlForReuse()`
- `MmGetMdlByteCount()`
- `MmGetMdlByteOffset()`
- `MmGetMdlVirtualAddress()`
- `MmGetSystemAddressForMdl()`
- `MmGetSystemAddressForMdlSafe()`
- `MmGetMdlPfnArray()`
- `MmBuildMdlForNonPagedPool()`
- `MmProbeAndLockPages()`
- `MmUnlockPages()`
- `MmMapLockedPages()`
- `MmMapLockedPagesSpecifyCache()`
- `MmUnmapLockedPages()`
- `MmAllocatePagesForMdl()`
- `MmAllocatePagesForMdlEx()`
- `MmFreePagesFromMdl()`
- `MmMapLockedPagesWithReservedMapping()`
- `MmUnmapReservedMapping()`
- `MmAllocateMappingAddress()`
- `MmFreeMappingAddress()`

### `nt!_MMPTE`

MMPTE用来表示内存管理的页表项（PTE），它被CPU的内存管理单元（MMU）用来转换虚拟地址（VA）到物理地址（PA）。映射一个VA到PA所需要的转换级别取决于CPU类型。对x86来说它使用2级转换（PDE和PTE），x86在PAE模式下使用3级转换（PPE，PDE和PTE），x64 CPU则使用4级转换（PXE，PPE，PDE，PTE）。不同级别结构的格式，诸如PXE，PPE，PDE和PTE都是相似的，MMPTE不仅可以用于表示PTE，还可以表示这些其他转换结构。
MMPTE结构是一个多重子结构的联合体，它们被Windows内存管理器的页错误处理机制用来搜索由PTE表示的页位置。例如，当一个PTE包含了一个页的有效的物理地址且MMU可以使用这个PTE完成地址转换，那么使用的子结构就是`u.Hard`字段。
当一个页从进程的工作集中被移除时，Windows内存管理器从一个硬件视角来看会标记这个页的PTE为无效的，然后，内存管理器重新试图用该PTE来存储OS关于页的特定信息。这会导致CPU内存管理单元（MMU）不能再使用该PTE来进行地址转换。当进程试图去访问这样一个页时，CPU生成一个页错误来调用Windows的页错误例程。PTE中编码的信息现在用来定位该页并将它返还给进程的工作集，这样就解决了页错误。有个这样的例子就是PTE转换，该PTE表示一个处于待命或修改状态的页。在这种情形下，`u.Transition`子结构用来存储该页的信息。
当物理页的内容被保存在页文件中时，Windows内存管理器会修改PTE来指向页文件中的页位置，此时使用的是`u.Soft`子结构。`u.Soft.PageFileLow`字段决定了Windows支持的16个页文件中哪一个包含了该页，而`u.Soft.PageFileHigh`则包含了在该页文件中页的索引。
调试器命令`!pte`可以显示给定虚拟地址的所有级别的页表内容。

### `nt!_MMPFN`

Windows内存管理持有系统中每个物理页的信息，放置在一个叫PFN数据库的数组中。MMPFN结构体表示了该数据库中的每一个独立条目，它包含了单一物理页的信息。
变量`nt!MmPfnDatabase`指向了MMPN结构体数组，它们组成了PFN数据库。PFN数据库的条目数量为`nt!MmPfnSize`，这其中有一些额外的条目来处理热插拔内存。为了保存内存，MMPFN结构被塞得很满。每个字段的解析都是不同的，而这取决于每个页的状态。
物理页的状态存储在`u3.e1.PageLocation`，由枚举类型`nt!_MMLISTS`中的一个条目标识。
`u2.ShareCount`字段包含进程指向该页的PTE数量，如果存在共享页则应该比1大。
`u3.e2.ReferenceCount`包含该页的引用数量，如果页被锁住则也包含锁的数量。该引用计数会在`u2.ShareCount`变成0时递减。
调试命令`!pfn`可以显示给定物理页的MMPFN结构体的所有内容。

### `nt!_MMPFNLIST`

内存管理器持有处于相同状态的链在一起的物理页。这可以加速查找某种给定状态的一个或多个页的过程，例如，查找空闲页或被零化换出（zeroed out）的页。MMPFNLIST结构体持有这些链表的头。系统中有多个MMPFNLIST结构体，它们中的每个都包含一个特定状态的页并被存储在内核变量`nt!Mm<PageState>ListHead`中，这里的PageState可以表示待命(Standby)、已修改(Modified)、无写修改（ModifiedNoWrite），空闲(Free)，只读(Rom)，损坏(Bad)，零化(Zeroed)。活跃态的页面例如当前属于一个进程工作集的页面不会再任何一个链表中。
在Windows的新版本中，`nt!MmStandbyPageListHead`不再使用，取而代之的是一个具有优先级的8链表集合，存在`nt!MmStandbyPageListByPriority`。`nt!MmFreePageListHead`和`nt!MmModifiedPageListHead`也不再被使用，转而使用的是`nt!MmFreePagesByColor`和`nt!MmModifiedProcessListByColor`或`nt!MmModifiedPageListByColor`。
`MMPFN.u1.Flink`和`MMPFN.u2.Blink`字段对一个特定页来说，用于保存该页到一个双向链表中。这些链表的头存储在对应的`MMPFNLIST`结构体的`Flink`和`Blink`字段。
`ListName`字段是枚举类型MMLISTS中的一个，它标识了该链表中页的类型。
调试器命令`!memusage 8`命令显示了某个特定状态的页的数量。

### `nt!_MMWSL`

Windows中每个进程都有一个和它关联的工作集，它由那些进程不会触发页错误的页组成。工作集整理者（WST），是内存管理器的一个组件，它运行在`KeBalanceSetManager()`线程的上下文，尽力去移除进程不再使用的页并重新分配它们到有需求的其他进程。为了执行这一任务，WST需要存储关于系统中每个进程的工作集信息。该信息被保存在MMWSL结构体。每个进程的`EPROCESS.Vm.VmWorkingSetList`都指向了它的MMWSL结构。
系统中每个进程的MMWSL都在内核虚拟地址空间的超空间(HyperSpace)上一个完全相同的地址。超空间是内核虚拟地址空间的一部分，每个进程都有一个自己的进程映射，并不是所有进程共享一个单一的映射。因此，内存管理器，对任何给定的实例，只能访问当前进程的MMWSL即目前在CPU上运行进程的线程。
MMWSL结构体的`Wsle`字段指向进程工作集列表项数组的基址。数组中条目的有效值为`EPROCESS.Vm.WorkingSetSize`。

### `nt!_MMWSLE`

MMWSLE数据结构表示一个进程工作集中单一页面的工作集列表，因此每一个在进程工作集中的页都有一个MMWSLE结构。该结构被工作集整理者用来判定该特定页是否是一个潜在的可整理候选页，也就是说从进程的工作集中移除。
当一个进程试图访问一个在工作集中的页面时，CPU内存管理单元MMU会在该页对应的PTE中设置`MMPTE.Hard.Accessed`位。工作集整理者有规律的唤醒并扫描一个进程的WSLEs。在扫描期间他会检查从上一次来是否已访问过一个特定的页，这是通过检查PTE的访问位来实现的。如果该页从上次扫描以来从未被访问过，该页就通过递增`u1.e1.Age`字段逐渐老化。如果页被访问过，那么`u1.e1.Age`字段就被重置为0。当`u1.e1.Age`涨到7时，该页就被认为是一个可以被换出的候选页。
`u1.e1.VirtualPageNumber`是页虚拟地址的高20位(在x86)或52位（在x64）。
调试命令`!wsle`可以显示一个特定进程的工作集列表条目。

### `nt!_POOL_HEADER`

内核中动态内存分配是由非页池、分页池和会话池组成的。根据请求分配的大小，池分配分为小池分配（尺寸少于4K）和大池分配（尺寸大于等于4k）。池分配大小在x86系统上总是向上取整到8字节，在x64系统上向上取整到16字节。
每个小池分配都由一个池头、数据区域组成。数据区域用于存储数据，里面还有一些用于对其的padding。池头用`POOL_HEADER`结构表示，该结构包含了关于后面跟随的数据区的信息。
`BlockSize`字段包含了池块的尺寸，囊括头和任何padding字节。`PreviousSize`字段包含了前一块的尺寸（低地址毗邻的块）。这两个字段在x86上都是8的倍数，x64上都是16的倍数。`PoolTag`字段包含4字节标记，它用于标识池分配的所有者，常常用于调试。如果最高位（31位）池标签被设置，则该池分配被标记为受保护。
大池分配没有内嵌的`POOL_HEADER`，取而代之的是，池头部存储了一个分离的叫做`nt!PoolBigTable`的表。因为大池分配需要在一个页边界(4K)对齐，所以这是有必要的。
调试命令`!pool`可以显示一个给定任何地址所在的池页中的所有池块。`!vm`显示池消耗信息。`!poolused`显示消耗的字节数以及所有池标签的块数。`!poolfind`定位一个特定标签的池分配。`!poolval`检查池头是否被污染，注意到他不会检查实际的池中数据是否被污染。`!frag`显示非分页池的外部池碎片信息。输出显示了系统中所有累计可用的非分页池页空闲块（碎片）的个数以及他们占用的内存量。
APIs：
- `ExAllocatePoolWithTag()`
