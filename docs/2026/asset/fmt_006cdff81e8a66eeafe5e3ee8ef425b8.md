
# sFilter.c 程序代码

接下来是 `sFilter.c` 的程序代码了。

我希望大家能够注意几个忽略的地方：

1. `ShadowCopyVolume` --- 也就是卷影。
2. `FastIo` 函数。
3. 文件系统控制设备对象的绑定。
4. 文件系统卷设备的绑定。
5. `SfCreate` 函数的一个小细节。

我看过楚狂人写的《寒江独钓Windows内核安全编程》这本书。这本书的第7章讲了 `sFilter`，但是这本书没有把 Windows 很重要一个系统服务说清楚。就是我上面所的卷影。

`sFilter.c` 的所有代码如下：

```c
#include "sfilter.h"

//指定每个函数的内存页面属性

#ifdef ALLOC_PRAGMA

#pragma alloc_text(INIT, DriverEntry)

#if DBG && WINVER >= 0x0501

#pragma alloc_text(PAGE, DriverUnload)

#endif

#pragma alloc_text(INIT, SfReadDriverParameters)

#pragma alloc_text(PAGE, SfFsNotification)

#pragma alloc_text(PAGE, SfCreate)

#pragma alloc_text(PAGE, SfCleanupClose)

#pragma alloc_text(PAGE, SfFsControl)

#pragma alloc_text(PAGE, SfFsControlMountVolume)

#pragma alloc_text(PAGE, SfFsControlMountVolumeComplete)

#pragma alloc_text(PAGE, SfFsControlLoadFileSystem)

#pragma alloc_text(PAGE, SfFsControlLoadFileSystemComplete)

#pragma alloc_text(PAGE, SfFastIoCheckIfPossible)

#pragma alloc_text(PAGE, SfFastIoRead)

#pragma alloc_text(PAGE, SfFastIoWrite)

#pragma alloc_text(PAGE, SfFastIoQueryBasicInfo)

#pragma alloc_text(PAGE, SfFastIoQueryStandardInfo)

#pragma alloc_text(PAGE, SfFastIoLock)

#pragma alloc_text(PAGE, SfFastIoUnlockSingle)

#pragma alloc_text(PAGE, SfFastIoUnlockAll)

#pragma alloc_text(PAGE, SfFastIoUnlockAllByKey)

#pragma alloc_text(PAGE, SfFastIoDeviceControl)

#pragma alloc_text(PAGE, SfFastIoDetachDevice)

#pragma alloc_text(PAGE, SfFastIoQueryNetworkOpenInfo)

#pragma alloc_text(PAGE, SfFastIoMdlRead)

#pragma alloc_text(PAGE, SfFastIoPrepareMdlWrite)

#pragma alloc_text(PAGE, SfFastIoMdlWriteComplete)

#pragma alloc_text(PAGE, SfFastIoReadCompressed)

#pragma alloc_text(PAGE, SfFastIoWriteCompressed)

#pragma alloc_text(PAGE, SfFastIoQueryOpen)

#pragma alloc_text(PAGE, SfAttachDeviceToDeviceStack)

#pragma alloc_text(PAGE, SfAttachToFileSystemDevice)

#pragma alloc_text(PAGE, SfDetachFromFileSystemDevice)

#pragma alloc_text(PAGE, SfAttachToMountedDevice)

#pragma alloc_text(PAGE, SfIsAttachedToDevice)

#pragma alloc_text(PAGE, SfIsAttachedToDeviceW2K)

#pragma alloc_text(PAGE, SfIsShadowCopyVolume)

#if WINVER >= 0x0501

#pragma alloc_text(INIT, SfLoadDynamicFunctions)

#pragma alloc_text(INIT, SfGetCurrentVersion)

#pragma alloc_text(PAGE, SfEnumerateFileSystemVolumes)

#pragma alloc_text(PAGE, SfIsAttachedToDeviceWXPAndLater)

#endif

#endif

PDRIVER_OBJECT gSFilterDriverObject = NULL;                                //保存由/O管理器生成并传入的驱动对象

PDEVICE_OBJECT gSFilterControlDeviceObject = NULL;                //保存由本过滤驱动生成的控制设备对象

FAST_MUTEX gSfilterAttachLock;                                                        //定义一个快速互斥结构变量(对象)

static const PCHAR DeviceTypeNames[] = {

    "",

    "BEEP",

    "CD_ROM",

    "CD_ROM_FILE_SYSTEM",

    "CONTROLLER",

    "DATALINK",

    "DFS",

    "DISK",

    "DISK_FILE_SYSTEM",

    "FILE_SYSTEM",

    "INPORT_PORT",

    "KEYBOARD",

    "MAILSLOT",

    "MIDI_IN",

    "MIDI_OUT",

    "MOUSE",

    "MULTI_UNC_PROVIDER",

    "NAMED_PIPE",

    "NETWORK",

    "NETWORK_BROWSER",

    "NETWORK_FILE_SYSTEM",

    "NULL",

    "PARALLEL_PORT",

    "PHYSICAL_NETCARD",

    "PRINTER",

    "SCANNER",

    "SERIAL_MOUSE_PORT",

    "SERIAL_PORT",

    "SCREEN",

    "SOUND",

    "STREAMS",

    "TAPE",

    "TAPE_FILE_SYSTEM",

    "TRANSPORT",

    "UNKNOWN",

    "VIDEO",

    "VIRTUAL_DISK",

    "WAVE_IN",

    "WAVE_OUT",

    "8042_PORT",

    "NETWORK_REDIRECTOR",

    "BATTERY",

    "BUS_EXTENDER",

    "MODEM",

    "VDM",

    "MASS_STORAGE",

    "SMB",

    "KS",

    "CHANGER",

    "SMARTCARD",

    "ACPI",

    "DVD",

    "FULLSCREEN_VIDEO",

    "DFS_FILE_SYSTEM",

    "DFS_VOLUME",

    "SERENUM",

    "TERMSRV",

    "KSEC"

};

NTSTATUS DriverEntry(IN PDRIVER_OBJECT DriverObject, IN PUNICODE_STRING RegistryPath)
{
    PFAST_IO_DISPATCH fastIoDispatch;        //定义FAST_IO_DISPATCH结构变量
    UNICODE_STRING nameString;                        //定义名字串结构变量
    NTSTATUS status;                                        //状态码
    ULONG i;

#if WINVER >= 0x0501                //条件编译,如果OS版本是WinXP以上,编译这两句,否则不编译
    SfLoadDynamicFunctions();
    SfGetCurrentVersion();
#endif

    SfReadDriverParameters( RegistryPath );

    gSFilterDriverObject = DriverObject;        //将I/O管理器传入的驱动对象保存到全局变量gSFilterDriverObject中

#if DBG && WINVER >= 0x0501                                        //若OS版本是xp以上且生成checked版,编译这些语句,否则不编译
    if (NULL != gSfDynamicFunctions.EnumerateDeviceObjectList)
        {
                gSFilterDriverObject->DriverUnload = DriverUnload;                                //注册驱动卸载函数
        }
#endif

        ExInitializeFastMutex( &gSfilterAttachLock );                                //初始化"FastMutex(快速互斥)"对象,以后多线程只能互斥访问它

        //创建控制设备名称
        RtlInitUnicodeString( &nameString, L"\\FileSystem\\Filters\\SFilter" );                //用来创建文件系统控制设备对象

        //创建控制设备对象
    status = IoCreateDevice( DriverObject,
                                                                0,                                                                                                //没有 设备扩展
                                                                &nameString,                                                                        //设备名:   FileSystem\\Filters\\SFilter
                                                                FILE_DEVICE_DISK_FILE_SYSTEM,                                        //设备类型: 磁盘文件系统
                                                                FILE_DEVICE_SECURE_OPEN,                                                //设备特征: 对发送到CDO的打开请求进行安全检查
                                                                FALSE,                                                                                        //生成一个在用户模式下使用的设备
                                                                &gSFilterControlDeviceObject );                                        //接收生成的"控制设备对象"

        if (status == STATUS_OBJECT_PATH_NOT_FOUND)                                                                //判断是否 未找到路径
        {
                RtlInitUnicodeString( &nameString, L"\\FileSystem\\SFilterCDO" );                //重新创建 控制设备名称
                status = IoCreateDevice( DriverObject, 0,
                                                                &nameString,                                                                                        //设备名: FileSystem\\SFilterCDO
                                                                FILE_DEVICE_DISK_FILE_SYSTEM,
                                                                FILE_DEVICE_SECURE_OPEN,
                                                                FALSE,
                                                                &gSFilterControlDeviceObject );                                                        //接收生成的 控制设备对象
        }

        if (!NT_SUCCESS( status ))                                                                                                                        //判断IoCreateDevice调用是否成功
        {
                KdPrint(( "SFilter!DriverEntry: Error creating control device object \"%wZ\", status=%08x\n", &nameString, status ));
                return status;                //错误返回(创建CDO失败)
        }

        //注册默认派遣函数
        for (i = 0; i <= IRP_MJ_MAXIMUM_FUNCTION; i++)
        {
                DriverObject->MajorFunction[i] = SfPassThrough;
        }

        //注册具体派遣函数
    DriverObject->MajorFunction[IRP_MJ_CREATE] = SfCreate;
    DriverObject->MajorFunction[IRP_MJ_CREATE_NAMED_PIPE] = SfCreate;
    DriverObject->MajorFunction[IRP_MJ_CREATE_MAILSLOT] = SfCreate;
    DriverObject->MajorFunction[IRP_MJ_FILE_SYSTEM_CONTROL] = SfFsControl;
}
```
