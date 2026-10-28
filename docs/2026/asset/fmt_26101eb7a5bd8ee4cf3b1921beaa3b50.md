
```
FLTMGR!\_FLT\_VOLUME
+0x000 Base : \_FLT\_OBJECT
+0x030 Flags : \_FLT\_VOLUME\_FLAGS
+0x034 FileSystemType : \_FLT\_FILESYSTEM\_TYPE // 文件系统类型. 如 FLT\_FSTYPE\_NTFS=2
+0x038 DeviceObject : Ptr64 \_DEVICE\_OBJECT
+0x040 DiskDeviceObject : Ptr64 \_DEVICE\_OBJECT // 指向物理磁盘的设备对象
+0x048 FrameZeroVolume : Ptr64 \_FLT\_VOLUME
+0x050 VolumeInNextFrame : Ptr64 \_FLT\_VOLUME
+0x058 Frame : Ptr64 \_FLTP\_FRAME
+0x060 DeviceName : \_UNICODE\_STRING // "\Device\HarddiskVolume3"
+0x070 GuidName : \_UNICODE\_STRING // "\??\Volume{7338f374-e276-45bd-9b04-34c204621c43}"
+0x080 CDODeviceName : \_UNICODE\_STRING // "\Ntfs"
+0x090 CDODriverName : \_UNICODE\_STRING // "\FileSystem\Ntfs"
+0x0a0 InstanceList : \_FLT\_RESOURCE\_LIST\_HEAD
+0x120 Callbacks : \_CALLBACK\_CTRL // 将与该卷所绑定的全部 instance 所设置的回调进行串联
/\*FLTMGR!\_CALLBACK\_CTRL
+0x000 OperationLists : [50] \_LIST\_ENTRY // 每种 IRP\_MJ\_xxx 均有一个双循环链表用于串联 instance
+0x320 OperationFlags : [50] \_CALLBACK\_NODE\_FLAGS
\*/
+0x508 ContextLock : \_EX\_PUSH\_LOCK
+0x510 VolumeContexts : \_CONTEXT\_LIST\_CTRL // 管理附加到该卷上的上下文
+0x518 StreamListCtrls : \_FLT\_RESOURCE\_LIST\_HEAD
+0x598 FileListCtrls : \_FLT\_RESOURCE\_LIST\_HEAD // 管理和存储流（Stream）和文件相关的控制信息
+0x618 NameCacheCtrl : \_NAME\_CACHE\_VOLUME\_CTRL // 管理文件名缓存
+0x6d0 MountNotifyLock : \_ERESOURCE // 同步卷挂载通知
+0x738 TargetedOpenActiveCount : Int4B // 记录针对该卷的活动打开操作数量
+0x740 TxVolContextListLock : \_EX\_PUSH\_LOCK
+0x748 TxVolContexts : \_TREE\_ROOT // 管理事务性卷上下文
+0x750 SupportedFeatures : Int4B // 该卷支持的特性集合
```

## 3.5 `IRP_Ctrl`

```
FLTMGR!\_IRP\_CTRL
+0x000 Type : \_FLT\_TYPE
+0x004 Flags : Uint4B
+0x008 LookAsideProcessorIndex : Uint4B // 分配此 \_IRP\_CTRL 的 CPU 核心索引，用于 Per-CPU 内存池管理
+0x00c MajorFunction : UChar
+0x00d DataScanConflictRetryCount : UChar // 用于 防病毒扫描冲突重试（如多个 AV 同时锁文件）
+0x00e CompletionStackLength : UChar // 当前 Completion Stack 中节点数量
+0x00f NextCompletion : UChar // 下一个要执行的 Post 回调索引（从高到低执行
+0x010 CompletionStack : Ptr64 \_COMPLETION\_NODE
// FLTMGR!\_COMPLETION\_NODE 是 Windows Minifilter 架构中用于管理 Post-operation 回调的关键结构，它在 I/O 请求（IRP）完成阶段被使用，确保每个注册了 PostOperation 回调的 Minifilter 都能按正确顺序、携带正确的上下文被调用。
/\*FLTMGR!\_COMPLETION\_NODE
+0x000 IrpCtrl : Ptr64 \_IRP\_CTRL
+0x008 CallbackNode : Ptr64 \_CALLBACK\_NODE
+0x008 Filter : Ptr64 \_FLT\_FILTER
+0x010 InstanceLink : \_LIST\_ENTRY
+0x020 InstanceTrackingList : Ptr64 \_COMPLETION\_NODE\_TRACKING\_LIST
+0x028 Context : Ptr64 Void
+0x030 DataSnapshot : \_FLT\_IO\_PARAMETER\_BLOCK
+0x078 Flags : Uint2B
\*/
+0x018 SyncEvent : \_KEVENT
// 用于同步等待 IRP 完成（当 Filter 返回 FLT\_PREOP\_PENDING 后需手动完成）
+0x030 Irp : Ptr64 \_IRP
+0x030 FsFilterData : Ptr64 \_FS\_FILTER\_CALLBACK\_DATA
// 在 Fast I/O 或 FsFilter 路径中，指向 \_FS\_FILTER\_CALLBACK\_DATA（非 IRP 路径）
+0x038 AsyncCompletionRoutine : Ptr64 void // 异步完成回调
+0x040 AsyncCompletionContext : Ptr64 Void // 异步完成上下文
+0x048 InitiatingInstance : Ptr64 \_FLT\_INSTANCE // 发起当前 I/O 的 Instance
+0x050 PendingCallbackNode : Ptr64 \_CALLBACK\_NODE // 当前正在执行的 Pre 回调节点（即下一个要调用的 \_CALLBACK\_NODE）
+0x050 StartingCallbackNode : Ptr64 \_CALLBACK\_NODE // 在某些路径下表示起始回调节点
+0x058 preOp : <anonymous-tag> // 临时存储 Pre/Post 回调的返回状态或参数
+0x058 postOp : <anonymous-tag>
+0x060 PostCompletionRoutine : Ptr64 void
+0x068 DeviceObject : Ptr64 \_DEVICE\_OBJECT
+0x070 FileObject : Ptr64 \_FILE\_OBJECT
+0x078 FltWork : \_FLTP\_WORKITEM // 用于延迟处理（如 Post 回调在 DPC/Worker 线程执行）
+0x078 PendingCallbackContext : Ptr64 Void // 挂起时的回调上下文
+0x080 CachedCompletionNode : Ptr64 \_COMPLETION\_NODE
// 预分配的 Completion 节点（用于快速路径，避免内存分配）
+0x088 PendingStatus : Int4B
+0x0a0 CreateIrp : <anonymous-tag> // 仅当 MajorFunction == IRP\_MJ\_CREATE 时有效，包含额外创建参数（如 ECP 列表、Options）
+0x0a0 CloseIrp : <anonymous-tag> // 仅当 MajorFunction == IRP\_MJ\_CLOSE 时有效
+0x0c0 OperationTimestamp : \_ETW\_KERNEL\_TRACE\_TIMESTAMP // ETW 时间戳，用于性能分析
+0x0e0 TraceStatus : Int4B // ETW 跟踪状态（是否已记录事件）
+0x0e8 Data : \_FLT\_CALLBACK\_DATA
/\*FLTMGR!\_FLT\_CALLBACK\_DATA; Minifilter 回调中传入的 PFLT\_CALLBACK\_DATA，封装了 I/O 信息
+0x000 Flags : Uint4B
+0x008 Thread : Ptr64 \_KTHREAD
+0x010 Iopb : Ptr64 \_FLT\_IO\_PARAMETER\_BLOCK // 参数块
+0x018 IoStatus : \_IO\_STATUS\_BLOCK // 返回状态
+0x028 TagData : Ptr64 \_FLT\_TAG\_DATA\_BUFFER // ECP/EFS 相关标签数据
+0x030 QueueLinks : \_LIST\_ENTRY
+0x040 QueueContext : [2] Ptr64 Void
+0x030 FilterContext : [4] Ptr64 Void
+0x050 RequestorMode : Char
\*/
+0x140 WorkingParameters : \_FLT\_IO\_PARAMETER\_BLOCK
/\*FLTMGR!\_FLT\_IO\_PARAMETER\_BLOCK
+0x000 IrpFlags : Uint4B
+0x004 MajorFunction : UChar
+0x005 MinorFunction : UChar
+0x006 OperationFlags : UChar
+0x007 Reserved : UChar
+0x008 TargetFileObject : Ptr64 \_FILE\_OBJECT
+0x010 TargetInstance : Ptr64 \_FLT\_INSTANCE
+0x018 Parameters : \_FLT\_PARAMETERS // IRP\_MJ\_xxx
\*/
```

---

如有错误之处，还望指正。

MiniFilter 架构的解析将从 (1) 初始化阶段的 Frame 初始化与卷抽象-设备栈附加，(2) Filter 注册, (3) Instance 实例化, (4) 回调函数触发等方面进行较为深入、细致的分析。

# 1. VolumeAttach

`FltpFsNotificationActual` 是 `fltMgr.sys` 在响应文件系统挂载事件时，自动为 Minifilter 创建实例并附加到卷设备栈的核心路径。这是 Minifilter 实现“**挂载时自动绑定（Auto-Attach）**”机制的底层实现。`FltpFsNotificationActual` 函数的触发时机如下：

![图片描述](https://bbs.kanxue.com/upload/attach/202601/832354_2DK275RB8AMTUFS.webp)

## 1.1 Path1: Initialization

fltMgr 在系统 Phase1 初始化阶段被加载，在完成自身初始化时会遍历当前已注册的所有文件系统，并为每个文件系统设备栈挂载设备对象，创建卷抽象与 frame 进行绑定。

> `KiStartSystemThread -> PspSystemThreadStartup -> Phase1Initialization -> IoInitSystem -> IoInitSystemPreDrivers -> IopInitializeBootDrivers -> IopInitializeBuiltinDriver -> FLTMGR!GsDriverEntry -> FLTMGR!DriverEntry -> FLTMGR!FltpAttachFrame -> FLTMGR!FltpFsNotificationActual -> FLTMGR!FltpAttachToFileSystemDevice -> FLTMGR!FltpAttachDeviceObject -> nt!IoAttachDeviceToDeviceStackSafe`

filMgr 在初始化阶段，调用函数 `FltpAttachFrame` 创建一个 `FrameZeroAltitude` 的 frame。当后续注册 filter 时，fltMgr 会通过读取注册表中的 altitude 值来动态扩展 `frame->AltitudeIntervalHigh`。该函数内部调用 `IoRegisterFsRegistrationChangeMountAware` 向文件系统注册通知回调例程(`FltpFsNotification`)，当文件系统注册或注销时触发。接着，调用 `IoGetDeviceObjectPointer` 函数获取目标表设备栈顶层的设备对象(包括 `"\Device\RawDisk"` 和 `"\Device\RawCdRom"`)，调用 `FltpFsNotificationActual` 函数(获取设备对象名并调用 `FltpAttachToFileSystemDevic`)执行附加。

函数 `FltpAttachToFileSystemDevic` 调用 `FltpCreateVolumeDeviceObject` 创建设备对象用于向目标设备栈中进行插入。

> 函数 `FltpCreateVolumeDeviceObject` 调用 `IoCreateDevice` 创建设备对象，并设置 `device->DeviceExtension`(`_VOLUME_DEVICE_EXTENSION`)。接着，调用函数 `FltpInitVolume` 初始化 volume 并与 frame 进行绑定(通过 `_VOLUME_DEVICE_EXTENSION`)。

> 函数 `FltpAttachDeviceObject` 调用 `IoAttachDeviceToDeviceStackSafe` 进行设备栈插入。

> 调用 `FltpDoFilterNotificationForNewVolume`/`FltpEnumerateFileSystemVolumes` 将创建的设备对象插入文件系统设备栈。

(`FltpEnumerateFileSystemVolumes`)在 fltMgr 首次加载时，主动扫描当前系统中所有已挂载的磁盘设备，并尝试为每个磁盘设备关联 volume 抽象并将设备对象插入设备栈。

1. 调用 `IoGetDeviceAttachmentBaseRef` 获取当前设备栈栈底的设备对象，调用 `IoEnumerateDeviceObjectList` 枚举设备对象列表。

2. 遍历设备对象列表，调用 `IoGetDiskDeviceObject` 获取对应顶层磁盘设备对象指针。调用 `FltpCreateVolumeDeviceObject+FltpAttachDeviceObject` 为每个磁盘设备对象创建并绑定 volume。

3. 有条件(`dword_1C00295F0 > 5`)调用 `FltpDoFilterNotificationForNewVolume`。

(`FltpDoFilterNotificationForNewVolume`)当一个新卷被挂载时，通知所有已注册的 minifilter，并为每个符合条件的 filter 创建新的实例并附加到该卷设备栈。通过遍历 `frame->RegisteredFilters` 查找所注册的 filter，调用 `FltpEnumerateRegistryInstances` 创建 instance 并与 volume 绑定。

## 1.2 Path2: MountVolume

`DriverEntry` 中设置 `MajorFunction[IRP_MJ_FILE_SYSTEM_CONTROL]` 为 `FltpFsControl`，包含对文件系统的控制操作，如卷挂载(`IRP_MN_MOUNT_VOLUME`)。 动态挂载到新卷的设备栈中，函数调用路径如下：

> `nt!IofCallDriver -> FLTMGR!FltpFsControl -> FLTMGR!FltpFsControlMountVolume -> FLTMGR!FltpAttachDeviceObject -> nt!IoAttachDeviceToDeviceStackSafe`

函数 `FltpFsControlMountVolume`，

1. 调用 `FltpLogEventWithObjectID` 关联 volume-frame-device，设置 `_IRP_CTRL`。

2. 调用 `FltpPerformPreMountCallbacks` 执行卷挂载的前置回调函数。

3. 为向下传递的 IRP（`MountVolume` 请求）准备下一层 `_IO_STACK_LOCATION`，复制必要的参数，并注册一个在任何情况下都会被调用的完成例程（`FltpFsControlCompletion`），以便过滤管理器能在底层文件系统处理完挂载请求后进行后续处理。

![图片描述](https://bbs.kanxue.com/upload/attach/202601/832354_QP4NNVK9K6E47JK.webp)

1. 调用 `FltpAttachDeviceObject` 将设备对象插入设备栈。

2. 调用 `FltpPerformPostMountCallbacks` 执行卷挂载的后置回调函数。

3. 调用 `FltpCompleteRequest(IofCompleteRequest)` 完成 IRP 处理。
