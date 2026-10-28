
```c
    case FLT_FSTYPE_FS_REC:

        DbgPrint("FileSystemType :%ls\n",L"\\FileSystem\\Fs_rec.");

        break;

    case FLT_FSTYPE_INCD:

        DbgPrint("FileSystemType :%ls\n",L"\\FileSystem\\InCDfs.");

        break;

    case FLT_FSTYPE_INCD_FAT:

        DbgPrint("FileSystemType :%ls\n",L"\\FileSystem\\InCDFat.");

        break;

    case FLT_FSTYPE_EXFAT:

        DbgPrint("FileSystemType :%ls\n",L"\\FileSystem\\exfat.");

        break;

    case FLT_FSTYPE_PSFS:

        DbgPrint("FileSystemType :%ls\n",L"\\FileSystem\\psfs.");

        break;

    case FLT_FSTYPE_GPFS:

        DbgPrint("FileSystemType :%ls\n",L"\\FileSystem\\gpfs.");

        break;

    case FLT_FSTYPE_NPFS:

        DbgPrint("FileSystemType :%ls\n",L"\\FileSystem\\npfs.");

        break;

    case FLT_FSTYPE_MSFS:

        DbgPrint("FileSystemType :%ls\n",L"\\FileSystem\\msfs.");

        break;

    case FLT_FSTYPE_CSVFS:

        DbgPrint("FileSystemType :%ls\n",L"\\FileSystem\\csvfs.");

        break;

    case FLT_FSTYPE_REFS:

        DbgPrint("FileSystemType :%ls\n",L"\\FileSystem\\refs.");

        break;

    case FLT_FSTYPE_OPENAFS:

        DbgPrint("FileSystemType :%ls\n",L"\\FileSystem\\AFSRedirector.");

        break;

    default:

        DbgPrint("FileSystemType :%ls\n",L"发生错误!");

        break;

    }

    //这里也要用FltObjectDereference再释放一下?

    FltObjectDereference(pv);

    ExFreePoolWithTag(Buffer, TAG);

}

void EnumerateInstances(PFLT_FILTER pf)

{

    //FltEnumerateInstances

    //FltGetInstanceInformation 这个获取的全是数字,所以放弃使用.

    //FltEnumerateInstanceInformationByFilter 这个获取的全是数字,所以放弃使用.

    //FltEnumerateInstanceInformationByVolume 这个获取的全是数字,所以放弃使用.

    NTSTATUS status = STATUS_SUCCESS;

    PFLT_INSTANCE  * InstanceList = 0;

    ULONG  InstanceListSize = 0;

    ULONG  NumberInstancesReturned = 0;

    ULONG i;

    status = FltEnumerateInstances(0, pf, InstanceList, InstanceListSize, &NumberInstancesReturned);

    if (!NT_SUCCESS( status ))

    {

        if (status != STATUS_BUFFER_TOO_SMALL)

        {

            return;

        }

    }

    InstanceListSize = sizeof (PFLT_INSTANCE) * NumberInstancesReturned * 2;//多申请一倍.

    InstanceList = (PFLT_INSTANCE *)ExAllocatePoolWithTag(NonPagedPool, InstanceListSize, TAG);

    if (InstanceList == NULL) {

        return;

    }

    RtlZeroMemory(InstanceList,InstanceListSize);

    status = FltEnumerateInstances(0, pf, InstanceList, InstanceListSize, &NumberInstancesReturned);

    if (!NT_SUCCESS( status ))

    {

        ExFreePoolWithTag(InstanceList, TAG);

        return;

    }

    for (i = 0; i< NumberInstancesReturned; i++)

    {

        //打印每个实例的信息.

        //相信和卷设备是一样的,转换为卷设备再打印,这里就不打印详细信息的,只打印实例的地址.

        DbgPrint("PFLT_FILTER:%p\tInstances:%p\n", pf, InstanceList[i]);

        FltObjectDereference(InstanceList[i]);

    }

    //这里也要用FltObjectDereference再释放一下?

    FltObjectDereference(pf);

    ExFreePoolWithTag(InstanceList, TAG);

}

void EnumerateVolumes(PFLT_FILTER pf)

{

    //FltEnumerateVolumes这个获取全了,要枚举.

    //FltGetVolumeInformation

    //FltEnumerateVolumeInformation这个不用,是获取单个的,要循环.

    NTSTATUS status = STATUS_SUCCESS;

    PFLT_VOLUME  * VolumeList = 0;

    ULONG  VolumeListSize = 0;

    ULONG  NumberVolumesReturned = 0;

    ULONG i;

    status = FltEnumerateVolumes(pf, VolumeList, VolumeListSize, &NumberVolumesReturned);

    if (!NT_SUCCESS( status ))

    {

        if (status != STATUS_BUFFER_TOO_SMALL)

        {

            return;

        }

    }

    VolumeListSize = sizeof (PFLT_VOLUME) * NumberVolumesReturned * 2;//多申请一倍.

    VolumeList = (PFLT_VOLUME  *)ExAllocatePoolWithTag(NonPagedPool, VolumeListSize, TAG);

    if (VolumeList == NULL) {

        return;

    }

    RtlZeroMemory(VolumeList,VolumeListSize);

    status = FltEnumerateVolumes(pf, VolumeList, VolumeListSize, &NumberVolumesReturned);

    if (!NT_SUCCESS( status ))

    {

        ExFreePoolWithTag(VolumeList, TAG);

        return;

    }

    for (i = 0; i< NumberVolumesReturned; i++)

    {

        //打印每个卷设备的信息.

        PrintVolumeStandardInformation(VolumeList[i]);

        FltObjectDereference(VolumeList[i]);

    }

    //这里也要用FltObjectDereference再释放一下?

    FltObjectDereference(pf);

    ExFreePoolWithTag(VolumeList, TAG);

}

VOID DriverUnload(IN PDRIVER_OBJECT DriverObject)

{

    /*

    没有此函数是停止不了此驱动的,但是可以删除驱动.

    在别的地方设置一下,如注册表回调保护驱动入口的第二个参数,可以让本驱动删除不了.

    如果这样,就在一个消息中卸载了:恢复回调入口和删除注册表的保护.

    然后应用层再卸载,上面的消息中还可以加密码验证.

    */

}

DRIVER_INITIALIZE DriverEntry;

#pragma alloc_text(INIT, DriverEntry)//#pragma INITCODE

NTSTATUS DriverEntry (_In_ PDRIVER_OBJECT DriverObject, _In_ PUNICODE_STRING RegistryPath)

{

    NTSTATUS status = STATUS_SUCCESS;

    PFLT_FILTER * FilterList;

    ULONG FilterListSize = 0;

    ULONG NumberFiltersReturned = 0;

    ULONG i;

    UNREFERENCED_PARAMETER( RegistryPath );

    KdBreakPoint();//DbgBreakPoint()

    DriverObject->DriverUnload = DriverUnload;

    //Because filters can register at any time, two calls to FltEnumerateFilters are not guaranteed to return the same result.

    //确保两次调用FltEnumerateFilters期间不要加载或者卸载minifilter.建议使用rundown机制.

    status = FltEnumerateFilters(0,FilterListSize,&NumberFiltersReturned);

    if (!NT_SUCCESS( status )) //#define STATUS_BUFFER_TOO_SMALL          ((NTSTATUS)0xC0000023L)

    {

        if (status != STATUS_BUFFER_TOO_SMALL)

        {

            return status;

        }

    }

    //建议每次成功的调用之后都调用:VOID FltObjectDereference(_Inout_  PVOID FltObject);

    FilterListSize = sizeof (PFLT_FILTER) * NumberFiltersReturned * 2;//多申请一倍.

    FilterList = (PFLT_FILTER *)ExAllocatePoolWithTag(NonPagedPool, FilterListSize, TAG);

    if (FilterList == NULL) {

        return status;

    }

    RtlZeroMemory(FilterList,FilterListSize);

    status = FltEnumerateFilters(FilterList,FilterListSize,&NumberFiltersReturned);

    if (!NT_SUCCESS( status ))

    {

        ExFreePoolWithTag(FilterList, TAG);

        return status;

    }

    //卸载所有已经注册的minifilter.理论上比抹去未公开的结构好.这里暂时注释掉.

    //for (i = 0;i < NumberFiltersReturned;i++)

    //{

    //    FltUnregisterFilter(FilterList[i]);//有的驱动会永远停止在这里.

    //}

    /*

    在这里可以列举一些minifilter的信息

    甚至每个minifilter过滤的每个卷设备的信息.

    PFLT_FILTER是个为公开的数据结构,各个版本的结构不一.

    0: kd> vertarget

    Windows 7 Kernel Version 7601 (Service Pack 1) MP (2 procs) Free x64

    Built by: 7601.18229.amd64fre.win7sp1_gdr.130801-1533

    Machine Name:

    Kernel base = 0xfffff800`0185f000 PsLoadedModuleList = 0xfffff800`01aa26d0

    Debug session time: Wed Oct 23 16:25:06.039 2013 (UTC + 8:00)

    System Uptime: 0 days 0:04:08.664

    0: kd> dt _FLT_FILTER fffffa80`04306c60

    fltmgr!_FLT_FILTER

    +0x000 Base             : _FLT_OBJECT

    +0x020 Frame            : 0xfffffa80`03f30630 _FLTP_FRAME

    +0x028 Name             : _UNICODE_STRING "TFsFlt"

    +0x038 DefaultAltitude  : _UNICODE_STRING "389700"

    +0x048 Flags            : 2 ( FLTFL_FILTERING_INITIATED )

    +0x050 DriverObject     : 0xfffffa80`042fde70 _DRIVER_OBJECT

    +0x058 InstanceList     : _FLT_RESOURCE_LIST_HEAD

    +0x0d8 VerifierExtension : (null)

    +0x0e0 VerifiedFiltersLink : _LIST_ENTRY [ 0x00000000`00000000 - 0x00000000`00000000 ]

    +0x0f0 FilterUnload     : 0xfffff880`0245f320     long  +0

    +0x0f8 InstanceSetup    : 0xfffff880`0248d0ec     long  +0

    +0x100 InstanceQueryTeardown : 0xfffff880`0248d118     long  +0

    +0x108 InstanceTeardownStart : (null)

    +0x110 InstanceTeardownComplete : 0xfffff880`0248d138     void  +0

    +0x118 SupportedContextsListHead : 0xfffffa80`04306890 _ALLOCATE_CONTEXT_HEADER

    +0x120 SupportedContexts : [6] (null)

    +0x150 PreVolumeMount   : (null)

    +0x158 PostVolumeMount  : (null)

    +0x160 GenerateFileName : (null)

    +0x168 NormalizeNameComponent : (null)
```
