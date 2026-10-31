
# [原创]新手眼中--xp的对象管理1

求加入能把新手煮熟的公司：`qq->1815349357[DeDf]`

`windows`以“对象”的观念，管理系统中的各种资源，于是经常见到和用到：`DRIVER_OBJECT`，`FILE_OBJECT`，`DEVICE_OBJECT`等等。

（命名的）对象被以一个树形结构组织起来，方便管理，按名顺路径查找。

这个树的根的地址，存在内核中名为`ObpRootDirectoryObject`的变量里，于是：

```
lkd> dd ObpRootDirectoryObject
80563ed8  e1009030 00000000 00000001 f772c1c0
```

可见笔者电脑里这个树根地址为`e1009030`.

`windows`描述对象时，每个对象刚好如树的叶子，对象结构本身并不包含构成树所必要的连结的成分，额外需要树杈和树枝才能构成一棵树。

`windows`对象管理里`hash`表的构造表明，对象树里每个树杈连`37`个树枝。

一图以蔽之：

![](http://bbs.pediy.com/attachment.php?attachmentid=84529&stc=1&d=1384770703)

对象的描述结构如下：

```c
typedef struct _OBJECT_HEADER
{
    LONG PointerCount;
    union
    {
        LONG HandleCount;
        PVOID NextToFree;
    };
    POBJECT_TYPE Type;
    UCHAR NameInfoOffset;
    UCHAR HandleInfoOffset;
    UCHAR QuotaInfoOffset;
    UCHAR Flags;
    union
    {
        POBJECT_CREATE_INFORMATION ObjectCreateInfo;
        PVOID QuotaBlockCharged;
    };
    PVOID SecurityDescriptor;
    QUAD Body;
} OBJECT_HEADER, *POBJECT_HEADER;
```

其实是这个样子：

```
lkd> dt _OBJECT_HEADER
nt!_OBJECT_HEADER
   +0x000 PointerCount     : Int4B
   +0x004 HandleCount      : Int4B
   +0x004 NextToFree       : Ptr32 Void
   +0x008 Type             : Ptr32 _OBJECT_TYPE
   +0x00c NameInfoOffset   : UChar
   +0x00d HandleInfoOffset : UChar
   +0x00e QuotaInfoOffset  : UChar
   +0x00f Flags            : UChar
   +0x010 ObjectCreateInfo : Ptr32 _OBJECT_CREATE_INFORMATION
   +0x010 QuotaBlockCharged : Ptr32 Void
   +0x014 SecurityDescriptor : Ptr32 Void
   +0x018 Body             : _QUAD    // 从+18这里开始就是对象的具体结构本身了
```

注意到`NameInfoOffset`，`HandleInfoOffset`，`QuotaInfoOffset`三个`UChar`各存储一个偏移值，分别代表一个与该对象相关的结构，它们都处于`OBJECT_HEADER`的上方，用`OBJECT_HEADER`的地址减去该偏移值寻址相关结构。

`windbg`里有一条命令"`!object`" ，获取对象相关信息十分方便。

```
lkd> !object e1009030
Object: e1009030  Type: (86fe9330) Directory
    ObjectHeader: e1009018 (old version)
    HandleCount: 0  PointerCount: 35
    Directory Object: 00000000  Name: \
    193 symbolic links snapped through this directory
    Hash Address  Type          Name
    ---- -------  ----          ----
     00  e100a628 Directory     ArcName
         86fc85d0 Device        Ntfs
     01  e1b464c0 Port          SeLsaCommandPort
     03  e101a490 Key           \REGISTRY
     04  850adb10 Device        UdfsCdRom
     09  e1b6d110 Directory     NLS
     10  e10091f8 SymbolicLink  DosDevices
     13  e1a921a8 Port          SeRmCommandPort
     14  e2a0cdd0 Port          LsaAuthenticationPort
         86f643d0 Device        Dfs
     16  e196f548 Directory     Driver
     18  85241030 Device        UdfsDisk
     19  e1011700 Directory     Device
     20  e1a2fdc0 Directory     Windows
     21  851607d8 Event         SAM_SERVICE_STARTED
         e1a7a698 Directory     Sessions
     22  e1b4ddc0 Directory     RPC Control
         e198cba8 Port          SmApiPort
     23  e1b74910 Directory     BaseNamedObjects
         e1000230 Directory     KernelObjects
     24  e1000088 Directory     GLOBAL??
         e1951578 Directory     FileSystem
     25  8508bc18 WaitablePort  NLAPublicPort
     26  e1009ab8 Directory     ObjectTypes
     27  e100c138 Directory     Security
         e2a2b698 Port          ErrorLogPort
     31  e1008440 SymbolicLink  SystemRoot
         85242a88 Device        Cdfs
     32  85077b50 WaitablePort  NLAPrivatePort
         e10010e0 Directory     Callback
     33  85240668 Event         SeLsaInitEvent
         852239b8 Event         UniqueSessionIdEvent
     35  e1a31c68 Directory     KnownDlls
```

附一份遍历`ObpRootDirectoryObject`下面那一层对象的代码

```c
#include "ntddk.h"
#define MAX_TABLE                    37
#define OBJECT_TO_OBJECT_HEADER(o)  CONTAINING_RECORD( (o), OBJECT_HEADER, Body )
typedef struct _OBJECT_CREATE_INFORMATION
{
    ULONG Attributes;
    PVOID RootDirectory;
    PVOID ParseContext;
    CHAR ProbeMode;
    ULONG PagedPoolCharge;
    ULONG NonPagedPoolCharge;
    ULONG SecurityDescriptorCharge;
    PVOID SecurityDescriptor;
    PSECURITY_QUALITY_OF_SERVICE SecurityQos;
    SECURITY_QUALITY_OF_SERVICE SecurityQualityOfService;
} OBJECT_CREATE_INFORMATION, *POBJECT_CREATE_INFORMATION;
typedef struct _OBJECT_HEADER
{
    LONG PointerCount;
    union
    {
        LONG HandleCount;
        PVOID NextToFree;
    };
    POBJECT_TYPE Type;
    UCHAR NameInfoOffset;
    UCHAR HandleInfoOffset;
    UCHAR QuotaInfoOffset;
    UCHAR Flags;
    union
    {
        POBJECT_CREATE_INFORMATION ObjectCreateInfo;
        PVOID QuotaBlockCharged;
    };
    PVOID SecurityDescriptor;
    QUAD Body;
} OBJECT_HEADER, *POBJECT_HEADER;
typedef struct _OBJECT_DIRECTORY_ENTRY
{
    struct _OBJECT_DIRECTORY_ENTRY *ChainLink;
    PVOID        Object;
}OBJECT_DIRECTORY_ENTRY,*POBJECT_DIRECTORY_ENTRY;
typedef struct _OBJECT_DIRECTORY{
    POBJECT_DIRECTORY_ENTRY        HashBuckets[MAX_TABLE];
    ULONG               Lock;
    PVOID               DeviceMap;
    ULONG               SessionId;
    USHORT              Reserved;
    USHORT              SymbolicLinkUsageCount;
}OBJECT_DIRECTORY,*POBJECT_DIRECTORY;
typedef struct _OBJECT_HEADER_NAME_INFO{
    POBJECT_DIRECTORY        Directory;
    UNICODE_STRING           Name;
    ULONG                    QueryReferences;
}OBJECT_HEADER_NAME_INFO,*POBJECT_HEADER_NAME_INFO;
NTSTATUS DriverEntry(PDRIVER_OBJECT pDriverObject,PUNICODE_STRING Reg_Path)
{
    POBJECT_DIRECTORY ObpRootDirectoryObject = (POBJECT_DIRECTORY)0xe1009030;
    POBJECT_DIRECTORY_ENTRY pObjDirEntry;
    PVOID pObject;
    POBJECT_HEADER pObjectHeader;
    POBJECT_HEADER_NAME_INFO pObjNameInfo;
    //
    UCHAR i;
    for (i = 0; i < MAX_TABLE; ++i)
    {
        pObjDirEntry = ObpRootDirectoryObject->HashBuckets[i];
        while (pObjDirEntry)
        {
            pObject = pObjDirEntry->Object;
            pObjectHeader = OBJECT_TO_OBJECT_HEADER(pObject);
            if (pObjectHeader->NameInfoOffset)
            {
                pObjNameInfo = (POBJECT_HEADER_NAME_INFO)((PUCHAR)pObjectHeader - pObjectHeader->NameInfoOffset);
                //KdPrint(("%08x : ", pObjectHeader));
                //KdPrint(("%08x : ", pObject));
                KdPrint(("%wZ\n", &pObjNameInfo->Name));
            }
            pObjDirEntry = pObjDirEntry->ChainLink;
        }
        //         if (pObjNameInfo->Name.Length == 22)
        //         {
        //             if(!memcmp(pObjNameInfo->Name.Buffer, L"ObjectTypes", 22))
        //                 break;
        //         }
    }
    return STATUS_UNSUCCESSFUL;
}
```

![](https://bbs.kanxue.com/upload/attach/201311/181388_rdewgevn6leer7x.jpg)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [新手眼中-XP的对象管理1.jpg](javascript:void(0);)

  （31.08kb，645次下载）
