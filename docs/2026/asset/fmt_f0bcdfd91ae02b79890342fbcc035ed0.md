
# [原创]pchunter逆向笔记

之前分析pchunter时的笔记，懒得整理了，凑合看

## 1. pchunter调试

IDA打开驱动文件

设置调试器

![](https://bbs.kanxue.com/upload/attach/201904/105259_REJDCBAGAFPXY9H.png)

![](https://bbs.kanxue.com/upload/attach/201904/105259_XS9BK463X665YB2.png)

运行vmmon

启动vmware虚拟机系统，

运行调试器

![](https://bbs.kanxue.com/upload/attach/201904/105259_YYG4G5M8FSG7UZM.png)

```
com:pipe,resets=0,reconnect,port=\\.\pipe\kd_windows_7_x64
com:pipe,resets=0,reconnect,port=\\.\pipe\kd_Windows_7_x64
```

prot值可以从vmmon处观察

其实都一样

`.sympath srv*229K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3#2K6k6r3I4Q4x3X3g2E0K9h3y4J5L8%4y4G2k6Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6V1L8%4N6F1L8r3!0S2k6q4)9J5c8Y4y4&6L8h3u0G2L8s2y4Q4x3@1u0p5i4K6y4m8i4K6g2o6L8r3!0U0j5h3I4K6P5h3#2T1L8$3I4K6`

`dt _driver_object` 不好使先设置符号路径，仍然不好使用 `.reload` 命令重新加载符号

找到 `driverEntry` 的 `iocontrol` 例程，找到里面的 `call eax` 为处理某个功能号，在此处下断点，`dd rbx`，查看功能号，输入输出等。

驱动分析技巧：驱动就是主要根据结构特征来定位，`driver_object`、`device_object`，各种的 `device_extension` 可以参考 `ddk sample` 和 `reactos`

功能号：`0x13` 号为判断地址是否可写

功能号：`31` 列出所有的模块名称

通过 `DirverObject` 找到 `DriverSection`, 枚举 `KLDR_DATA_TABLE_ENTRY` 中的 `InLoadOrderLinks` 双向链表, 获取加载的模块名称

```
//
qword_FFFFF8800389E558 = (__int64)DriverObject;

WINDBG>dt _driver_object
nt!_DRIVER_OBJECT
+0x000 Type : Int2B
+0x002 Size : Int2B
+0x008 DeviceObject : Ptr64 _DEVICE_OBJECT
+0x010 Flags : Uint4B
+0x018 DriverStart : Ptr64 Void
+0x020 DriverSize : Uint4B
+0x028 DriverSection : Ptr64 Void
+0x030 DriverExtension : Ptr64 _DRIVER_EXTENSION
+0x038 DriverName : _UNICODE_STRING
+0x048 HardwareDatabase : Ptr64 _UNICODE_STRING
+0x050 FastIoDispatch : Ptr64 _FAST_IO_DISPATCH
+0x058 DriverInit : Ptr64 long
+0x060 DriverStartIo : Ptr64 void
+0x068 DriverUnload : Ptr64 void
+0x070 MajorFunction : [28] Ptr64 long
```

```
dt _device_object

v8 = qword_FFFFF8800389FF58; //driver_object dt _driver_object
if ( !qword_FFFFF8800389FF58 )
    v8 = **(_QWORD **)(qword_FFFFF8800389E558 + 0x28); //driver_section

DriverSection输出出来是以下结构体 // dt _LDR_DATA_TABLE_ENTRY

WINDBG>dt _LDR_DATA_TABLE_ENTRY
nt!_LDR_DATA_TABLE_ENTRY
+0x000 InLoadOrderLinks : _LIST_ENTRY
+0x010 InMemoryOrderLinks : _LIST_ENTRY
+0x020 InInitializationOrderLinks : _LIST_ENTRY
+0x030 DllBase : Ptr64 Void
+0x038 EntryPoint : Ptr64 Void
+0x040 SizeOfImage : Uint4B
+0x048 FullDllName : _UNICODE_STRING
+0x058 BaseDllName : _UNICODE_STRING
+0x068 Flags : Uint4B
+0x06c LoadCount : Uint2B
+0x06e TlsIndex : Uint2B
+0x070 HashLinks : _LIST_ENTRY
+0x070 SectionPointer : Ptr64 Void
+0x078 CheckSum : Uint4B
+0x080 TimeDateStamp : Uint4B
+0x080 LoadedImports : Ptr64 Void
+0x088 EntryPointActivationContext : Ptr64 _ACTIVATION_CONTEXT
+0x090 PatchInformation : Ptr64 Void
+0x098 ForwarderLinks : _LIST_ENTRY
+0x0a8 ServiceTagLinks : _LIST_ENTRY
+0x0b8 StaticLinks : _LIST_ENTRY
+0x0c8 ContextInformation : Ptr64 Void
+0x0d0 OriginalBase : Uint8B
+0x0d8 LoadTime : _LARGE_INTEGER
```

```
{
    if ( v8 <= MmSystemRangeStart )
        break;
    ++v11;
    v10 = 0;
    if ( v11 > 0x1000 || *(_QWORD *)(v8 + 0x30) == qword_FFFFF8800389E568 )
        break;
    v8 = *(_QWORD *)v8; //InLoadOrderLinks
}
```

```
//

WINDBG>dt _LDR_DATA_TABLE_ENTRY
nt!_LDR_DATA_TABLE_ENTRY
+0x000 InLoadOrderLinks : _LIST_ENTRY
+0x010 InMemoryOrderLinks : _LIST_ENTRY
+0x020 InInitializationOrderLinks : _LIST_ENTRY
+0x030 DllBase : Ptr64 Void
+0x038 EntryPoint : Ptr64 Void
+0x040 SizeOfImage : Uint4B
+0x048 FullDllName : _UNICODE_STRING
+0x058 BaseDllName : _UNICODE_STRING
+0x068 Flags : Uint4B
+0x06c LoadCount : Uint2B
+0x06e TlsIndex : Uint2B
+0x070 HashLinks : _LIST_ENTRY
+0x070 SectionPointer : Ptr64 Void
+0x078 CheckSum : Uint4B
+0x080 TimeDateStamp : Uint4B
+0x080 LoadedImports : Ptr64 Void
+0x088 EntryPointActivationContext : Ptr64 _ACTIVATION_CONTEXT
+0x090 PatchInformation : Ptr64 Void
+0x098 ForwarderLinks : _LIST_ENTRY
+0x0a8 ServiceTagLinks : _LIST_ENTRY
+0x0b8 StaticLinks : _LIST_ENTRY
+0x0c8 ContextInformation : Ptr64 Void
+0x0d0 OriginalBase : Uint8B
+0x0d8 LoadTime : _LARGE_INTEGER

v24 = *(_WORD *)(v8 + 0x48) & 0xFFFE;
memmove((void *)(v17 + 36), *(const void **)(v8 + 0x50), (unsigned int)v24);
```

### 点击内核钩子，FSD

枚举下列模块，记录下他们的加载地址和镜像大小

![](https://bbs.kanxue.com/upload/attach/201904/105259_FKEVZEPNDGJ6EH6.png)

单机键盘选项卡

读取原始函数地址

1. 打开文件，读取文件
2. 分配镜像大小，按内存展开
3. 找到 `OEP`
4. 在 `oep` 函数内存中通过特征码，找到 `irp` 处理函数的偏移

![](https://bbs.kanxue.com/upload/attach/201904/105259_W2MA9PNGBHJRM9S.png)

### 内核钩子选项卡下的内核钩子

从磁盘中读取文件

根据导出表来检查钩子

根据区段来检查钩子，判断区段是否可读，`0x20000020` is executable contains code

![](https://bbs.kanxue.com/upload/attach/201904/105259_82HHYXQF9XMDR4S.png)

### IDT hook

`35` 号功能

`KeGetProcessorNumberFromIndex`

`KeSetSystemGroupAffinityThread`

`dt _KPCR`

`dt _KPRCB`

`(GS:[0x20] is the "KPCR.CurrentPrcb" pointer`

`d27K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4y4W2j5%4g2J5K9i4c8W2j5h3#2Q4x3X3g2U0L8$3#2Q4x3V1k6K6k6h3y4#2M7X3W2@1P5h3&6W2N6%4y4Q4x3V1j5$3g2e0l9H3c8o6m8m8e0U0g2s2i4K6u0W2K9s2c8E0L8l9`.

@0环的 `ETHREAD` 结构体是记录线程的相关信息，`EPROCESS` 结构体是记录进程相关的信息，同样我们每个CPU也有一个结构体来记录每个CPU的状态，这个结构体就是 `KPCR` 结构体，`KPCR` 结构体如下，下面该结构体中几个主要的成员。

先 `KeStartDynamicProcessor` 函数处开始，(`rva = 0x4f6a70`) 从内存中搜索，搜索不到的话通过 `kpcr` 来获取原函数地址，搜索的函数为 `sub_FFFFxxxxxx1e50()`

![](https://bbs.kanxue.com/upload/attach/201904/105259_KH5HPAZTQ8Z8PNN.png)

`278a48` ntoskernel的地址加上这个数，就是上面的 `v1`

`v1` 的指向的值加上 `6` 就是了

`__int64 sub_1405A62E0()` ntoskernel对处理器的处理

在 `KeStartDynamicProcessor` 函数中找到 `v10 = sub_1401A0B10(v7, v6, v5, v9);`

在 `sub_1401A0B10` 函数中找到

`v7 = sub_1403FA9D0(&v33, &Dst, v8, v13, (unsigned __int16)v23, v4, dword_1402B10C0, SHIDWORD(v26), v14, v9, v10);`

在 `sub_1403FA9D0` 中定位到 `1403FAA68`

`1403FAA68` 为找地址 `278a48` 的关键点，该地址位于 `hal *__usercall sub_1403FA9D0` 函数处

读取 `sidt` 寄存器来获取中段处理历程，获取挂钩后的地址

`pchunter` 的不一定准，它的结果和 `windbg !idt` 输出有不一致的地方，`windbg` 能找到的 `51` 号，在 `pci` 驱动中，`pchunter` 找不到

### 内核选项卡

枚举过滤驱动的功能号 `45`

枚举的驱动：

```
\FileSystem\RAW \Driver\KbdClass \Driver\i8042prt
\Driver\Tcpip \Driver\nsiproxy \Driver\tdx
\Driver\Mouclass \Driver\NDIS \Driver\PnpManager
```

`ObReferenceObjectByName` 获取驱动对象，枚举 `device object` 的 `attchached device`，获取 `attached device` 的驱动对象名、设备名和驱动路径

`dt _DEVICE_OBJECT`

`dt _DRIVER_OBJECT`

### 工作线程队列

功能号：`0xc3`

在 `ntoskrnl` 中，`rva 0x8cc01` 为 `ExQueueWorkItem`

在函数中找到 `rva 0x21D600`，此处为 `ExWorkerQueue` 全局变量，枚举 `ExWorkerQueue` 可以得到工作线程队列，枚举 `WORK_QUEUE_ITEM`

参考9.4 内核劳务线程 reactos `76bK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0G2L8$3E0Q4x3X3f1#2x3h3y4@1L8#2)9J5k6h3y4G2L8g2)9J5c8X3q4J5N6q4)9J5c8U0t1H3x3o6V1I4x3W2)9J5c8U0p5%4y4o6j5@1z5q4)9J5k6h3S2@1L8g2!0q4c8W2!0n7b7#2)9^5b7H3`.

`ExWorkerQueue` 是全局数组

一共三类

```
typedef enum _WORK_QUEUE_TYPE {
    CriticalWorkQueue,
    DelayedWorkQueue,
    HyperCriticalWorkQueue,
    MaximumWorkQueue
} WORK_QUEUE_TYPE;

WINDBG>dt _KQUEUE 0xFFFFF80004076600
ntdll!_KQUEUE
+0x000 Header : _DISPATCHER_HEADER
+0x018 EntryListHead : _LIST_ENTRY [ 0xfffff800`04076618 - 0xfffff800`04076618 ]
+0x028 CurrentCount : 0
+0x02c MaximumCount : 1
+0x030 ThreadListHead : _LIST_ENTRY [ 0xfffffa80`018fd208 - 0xfffffa80`018fed28 ]
```

`ThreadListHead` 是这类劳务线程链表，`nt!_KTHREAD.QueueListEntry` 的队列

`!exqueue` 命令显示了关于工作队列和工作线程的详细信息。

### hal回调

`haldispatch table:`

功能号：`0xf0`

`ntoskrnl.exe` 导出全局变量：`HalDispatchTable`

从此处 `HalDispatchTable+8` 的地址处枚举 `0x15` 个地址

`halPrivateDispatchTable:`

功能号：`0xf1`

`ntoskrnl.exe` 导出变量：`HalPrivateDispatchTable`

从此处 `HalPrivateDispatchTable+8` 的地址处枚举 `0x2D` 个地址

`halAcpiDispatchTable:`

功能号：`0xf2`

找到 `HalInitializeProcessor` 函数的地址

在 `.data` 数据段搜索特征码 `1212238880`（`48414C20h`）

找到 rva 为 `0x254c0` 的全局变量

+8 处开始计算地址

原函数地址处的也是从此处复制过去的。有什么用呢

### wdf

`wdf01000` 派发函数：

功能号：`0xaf`

当前函数地址通过 `driverobject` 获得 `\Driver\wdf01000`

原始函数地址通过，打开 `SystemRoot\System32\drivers\wdf01000.sys`，通过 `oep` 查找原函数地址

下图为 IDA 反汇编 `wdf01000.sys` 入口点的部分截图

找到 `wdf` 中 `IRP_MJ_CREATE`、`IRP_MJ_CLEANUP`、`IRP_MJ_CLOSE`，这几个派发函数位于 `wdf01000.sys` 中

![](https://bbs.kanxue.com/upload/attach/201904/105259_DCMDVFZDQURYRZM.png)

找到 `rva 0x91148`，此 `rva` 后面后 `4` 个常量地址，这四个函数是干嘛的？

保存后面四个函数的特征码

派发函数好多在 `ntoskrnl` 中，派遣函数地址都是填的下面的地址

`ntoskernel` 导出表中找到 `iocreatedriver`，在该函数中找到 `rva 0x661d4`，该函数名称为 `IopInvalidDeviceRequest`

![](https://bbs.kanxue.com/upload/attach/201904/105259_7MP7988VXKQ6W8D.png)
