
## 1.3 Path3: RegisterFileSystem

> `nt!IoRegisterFileSystem` -> `FLTMGR!FltpFsNotification` -> `FLTMGR!FltpFsNotificationActual` -> `FLTMGR!FltpAttachToFileSystemDevice` -> `FLTMGR!FltpAttachDeviceObject` -> `nt!IoAttachDeviceToDeviceStackSafe`

函数 `FltpFsNotification` 的逻辑较为简单，调用 `FltpFsNotificationActual` 执行设备栈附加。

# 2. Registration

函数 `FltRegisterFilter` 用于向 MiniFiltter 注册 filter，除了初始化 `_FLT_REGISTRATION` 结构体中所设置的参数外，还包括：验证 filter 的安全性(`FltpInitializeFilterVerifier`)、根据注册表所设置的 altitude 按顺序插入 `frame->RegisteredFilters` 链表。

需要注意的是，`_FLT_FILTER` 结构体的大小会依据所注册回调函数数量而发生拓展。

![图片描述](https://bbs.kanxue.com/upload/attach/202601/832354_SDRR78972QB6PBB.webp)

### 2.1 FltpGetInstanceAltitude

函数 `FltpGetInstanceAltitude` 中，读取注册表中实例的键用于获取 Altitude。

1. 内部调用 `FltpOpenInstancesRegistryKey` 打开对应的注册表键，路径为 `\\Registry\\Machine\\System\\CurrentControlSet\\Services\\Instance`.

    ![图片描述](https://bbs.kanxue.com/upload/attach/202602/832354_Z9D8J4RYV6DEC2A.webp)

2. 查询名为 `DEFAULTINSTANCE` 的子键，接着查询该子键的 `Altitude` 值，并返回字符串格式。

    ![图片描述](https://bbs.kanxue.com/upload/attach/202602/832354_T9ZMTFFQPPWK8XN.webp)

    ![图片描述](https://bbs.kanxue.com/upload/attach/202602/832354_9GW7JNS2N3N6QCM.webp)

这也解释了为什么 MiniFilter 驱动安装时的 INF 文件的重要性。

参见： [有偿求解决,文件过滤驱动函数FltRegisterFilter调用失败返回0xc0000034的问题](https://bbs.kanxue.com/thread-262815.htm)

### 2.2 FltpFindFrameForFilter

1. 当 `flttMgr` 中还未创建 `frame` 时，调用函数 `FltpAttachFrame` 设置并初始化 `frame`，填充 altitude，完成设备栈的插入。`frame->AltitudeIntervalLow` 默认为0，`frame->AltitudeIntervalHigh` 为当前 altitude。

    ![图片描述](https://bbs.kanxue.com/upload/attach/202601/832354_U57VSGTMGST3BJA.webp)

2. 当 `frame` 存在时，若 altitude 不在当前 `frame` 的范围内，更新当前 `frame->AltitudeIntervalHigh`。因此，`fltMgr` 只有一个 altitude 可动态扩展的 `frame`。

    ![图片描述](https://bbs.kanxue.com/upload/attach/202601/832354_9RY7WJSJ3HJU64Q.webp)

### 2.3 FltpLinkFilterIntoFrame

函数 `FltpLinkFilterIntoFrame` 依据 altitude 插入 `frame->RegisteredFilters` 链表。altitude 的值决定了 filter 在设备栈中的被调用顺序。

![图片描述](https://bbs.kanxue.com/upload/attach/202601/832354_T7WW8BQRZT3SSKT.webp)

## 3. FltStartFiltering

函数 `FltStartFiltering` 用于创建 instance 并于 volume 进行关联。当设置 InstanceSetupCallback 回调时的函数执行路径如下：

> `FltStartFiltering` -> `FltpDoVolumeNotificationForNewFilter` -> `FltpEnumerateRegistryInstances` -> `FltpCreateInstanceFromName` -> `FltpInitInstance` -> `FltpDoInstanceSetupNotification` -> `self_func`

函数 `FltpEnumerateRegistryInstances` 通过查询注册表的 instance 子键数来决定所需创建 `_FLT_INSTANCE` 对象数量 (`FltpOpenInstancesRegistryKey` -> `\\Registry\\Machine\\System\\CurrentControlSet\\Services\\Instance`)。接着，调用 `FltpCreateInstanceFromName` 创建 instance 对象并与 volume 绑定。

![图片描述](https://bbs.kanxue.com/upload/attach/202601/832354_Q2RSSSNGBCBHWKX.webp)

instance 子键数量由 INF 文件进行定义。

![图片描述](https://bbs.kanxue.com/upload/attach/202601/832354_KAQXSTKAXPK3B3R.webp)

虽然当前仅有一个 instance 子键，但上层调用函数 `FltpEnumerateRegistryInstances` 通过遍历当前 `frame->AttachedVolumes` 来为每个 volume 绑定 instance。因此，即使仅设置了一个 instance，需要为每个 volume 均绑定一个 instance 对象。

![图片描述](https://bbs.kanxue.com/upload/attach/202601/832354_3WNQ76X9W68J43H.webp)

### 3.1 FltpInitInstance

函数 `FltpInitInstance` 初始化 instance 并依据 MajorFunction 将回调插入 `volume->Callbacks` 中：

1. 函数 `FltpNumCallbackNodes` 获取当前 filter 的有效回调数。由于 filter 所注册的回调函数数量不固定，因此申请 `_FLT_INSTANCE` 结构体的内存时需要获取有效回调函数数量。`fiilter->Operations` 实际上指向一个 `_FLT_OPERATION_REGISTRATION` 数组，以 `0x80` 为结束标志。类似的，`instance->CallbackNodes` 提供了一个长度为 50 的 `_CALLBACK_NODE` 指针表用于快速索引，而 `_CALLBACK_NODE` 结构体紧随 instance。

    ![图片描述](https://bbs.kanxue.com/upload/attach/202601/832354_662NXD3BEPTWY94.webp)

2. 函数 `FltpDoInstanceSetupNotification` 调用 `filter->InstanceSetup` 执行回调函数。

3. 函数 `FltpSetCallbacksForInstance` 设置回调函数组。

4. 按照 MajorFunction 将 instance 的回调插入 volume 中。

![图片描述](https://bbs.kanxue.com/upload/attach/202601/832354_TN2CHCF4MJDRB5R.webp)

## Callback

`FltpCreate` 函数中 ，通过 `_DEVICE_OBJECT::_DRIVER_EXTENSION::_FLT_VOLUME` 解析设备对象所属的卷，完成 `_IRP_CALL_CTRL` 结构体的初始化(将作为函数 `FltpPassThroughInternal` 的参数)。此处的 `_DRIVER_EXTENSION` 实际上是 `FLTMGR!_VOLUME_DEVICE_EXTENSION`，由驱动程序自行定义，用于存储驱动的私有数据。

![图片描述](https://bbs.kanxue.com/upload/attach/202601/832354_GXVV3F3EWTHYQNW.webp)

结构体 `_VOLUME_DEVICE_EXTENSION` 的定义如下：

|  |  |

| --- | --- |

| 1  2  3  4  5  6 | `FLTMGR!_VOLUME_DEVICE_EXTENSION`  `+0x000 Type             : _FLT_TYPE`  `+0x008 AttachedToDeviceObject : Ptr64 _DEVICE_OBJECT` `// 所附加的设备对象`  `+0x010 Frame            : Ptr64 _FLTP_FRAME`  `+0x018 VolumeAccessLock : _FAST_MUTEX`  `+0x050 Volume           : Ptr64 _FLT_VOLUME` `// 关联私有数据结构(卷), 用于下层函数调用` |

结构体 `_IRP_CALL_CTRL` 用于管理单个 IRP 在 MiniFilter 中的处理过程，包含当前 I/O 请求的上下文，其定义如下：

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7 | `FLTMGR!_IRP_CALL_CTRL`  `+0x000 Volume           : Ptr64 _FLT_VOLUME`  `+0x008 Irp              : Ptr64 _IRP`  `+0x010 IrpCtrl          : Ptr64 _IRP_CTRL`  `+0x018 StartingCallbackNode : Ptr64 _CALLBACK_NODE` `// 处理此 irp 的第一个回调节点`  `+0x020 OperationStatusCallbackListHead : _SINGLE_LIST_ENTRY` `// 状态回调链表（如完成通知）`  `+0x028 Flags            : ICC_FLAGS` |

函数 `FltpPassThroughInternal` 和 `FltpLegacyProcessingAfterPreCallbacksCompleted` 内部调用 Pre/Post 回调函数。

![图片描述](https://bbs.kanxue.com/upload/attach/202601/832354_RMEF475GB24MPQZ.webp)

回调函数被触发时的调用路径：

- Pre：`IofCallDriver` -> `FLTMGR!FltpCreate` -> `FLTMGR!FltpPassThroughInternal` -> `FLTMGR!FltpPerformPreCallbacks` -> `PreFunc`

- Post：`IofCallDriver` -> `FLTMGR!FltpCreate` -> `FltpLegacyProcessingAfterPreCallbacksCompleted` -> `FltpPassThroughCompletionWorker` -> `FltpPerformPostCallbacks` -> `PostFunc`

> `FltpPerformPreCallbacks` 是 `fltmgr.sys` 内部用于按 Altitude 从高到低依次调用 Minifilter Pre-operation 回调的核心函数，是 Minifilter 实现“有序过滤”的执行中枢。它本身不派发 IRP 到文件系统，但决定了哪些过滤器有机会拦截或修改 I/O 请求。

# 3. Appendix

## 3.1 Frame

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27 | `FLTMGR!_FLTP_FRAME`  `+0x000 Type             : _FLT_TYPE`  `+0x008 Links            : _LIST_ENTRY` `// 指向 FltGlobals_0xc0 ???`  `+0x018 FrameID          : Uint4B`  `+0x020 AltitudeIntervalLow : _UNICODE_STRING` `// 该 Frame 覆盖的 Altitude 下限`  `+0x030 AltitudeIntervalHigh : _UNICODE_STRING` `// 该 Frame 覆盖的 Altitude 上限` `// 当 Minifilter 注册时，FLTMGR 根据其 Altitude 字符串找到对应的 Frame，并将其加入 RegisteredFilters。`  `+0x040 LargeIrpCtrlStackSize : UChar`  `+0x041 SmallIrpCtrlStackSize : UChar`  `+0x048 RegisteredFilters : _FLT_RESOURCE_LIST_HEAD` `// 该 Frame 内所有已注册 Minifilter 的链表头` `// I/O 请求按此顺序调用 Pre 回调，反序调用 Post 回调`  `+0x0c8 AttachedVolumes  : _FLT_RESOURCE_LIST_HEAD` `// 该 Frame 中所有` `Filter` `已 attach 的 卷（Volume）列表. 实际是 _FLT_VOLUME 对象的集合, 用于卷级资源管理.`  `+0x148 MountingVolumes  : _LIST_ENTRY` `// 正在挂载过程中的卷临时链表（避免竞态）`  `+0x158 AttachedFileSystems : _FLT_MUTEX_LIST_HEAD` `// 已 attach 的文件系统设备（如 \Device\Ntfs）列表`  `+0x1a8 ZombiedFltObjectContexts : _FLT_MUTEX_LIST_HEAD` `// 僵尸”上下文列表` `// 当 File/Stream 关闭但上下文仍有引用时暂存于此, 待引用归零后释放, 防止 Use-After-Free`  `+0x1f8 KtmResourceManagerHandle : Ptr64 Void` `// KTM（Kernel Transaction Manager）资源管理器句柄`  `+0x200 KtmResourceManager : Ptr64 _KRESOURCEMANAGER` `// 指向内核 KTM 资源管理器对象, 用于支持事务 NTFS（TxF）通知`  `+0x208 FilterUnloadLock : _ERESOURCE`  `+0x270 DeviceObjectAttachLock : _FAST_MUTEX`  `+0x2a8 Prcb             : Ptr64 _FLT_PRCB` `// 指向 Filter Manager 的 Per-Processor Control Block, 存储 CPU 局部数据（如 IRP 控制块缓存）`  `+0x2b0 PrcbPoolToFree   : Ptr64 Void` `// 待释放的 PRCB 内存池（延迟释放）`  `+0x2b8 LookasidePoolToFree : Ptr64 Void` `// 待释放的 Lookaside List 内存`  `+0x2c0 IrpCtrlStackProfiler : _FLTP_IRPCTRL_STACK_PROFILER` `// IRP 控制栈使用分析器（调试/性能调优）`  `+0x400 SmallIrpCtrlLookasideList : _NPAGED_LOOKASIDE_LIST` `// 小型 IRP 控制块（IrpCtrl）的快速分配池`  `+0x480 LargeIrpCtrlLookasideList : _NPAGED_LOOKASIDE_LIST` `// 大型 IrpCtrl 分配池`  `+0x500 ReserveIrpCtrls  : _RESERVE_IRPCTRL` `// 预留的 IrpCtrl（用于内存压力下保证关键路径不失败）` |

## 3.2 Filter
