

# 浅析 windows 对象管理

> 原文：[`www.pediy.com/kssd/pediy10/74430.html`](https://www.pediy.com/kssd/pediy10/74430.html)

希望对菜鸟理解 NT 有帮助  
  
作者:Sysnap   
  
出处:http://hi.baidu.com/sysnap   
标题:Windows 对象管理  
  
  
把以前的对象笔记帖上来...后面多加了俩个 函数..来加深理解...  
一个是 ZwQueryDirectoryObject 来检测隐藏驱动  
一个是让 ZwQueryDirectoryObject 检测不了的...代码比较简单..先看前面的理论再去理解后面的函数...应该可以编译出来的..否则那个函数你可能埋怨说代码不完整...  
1 对象数据结构  
  
在对象头的前面有几个可选结构... 可以是下面的组合  
  
typedef struct \_OBJECT\_HEADER\_QUOTA\_INFO {   
ULONG PagedPoolCharge;   
ULONG NonPagedPoolCharge;   
ULONG SecurityDescriptorCharge;   
PEPROCESS ExclusiveProcess; // 拥有此对象的进程  
  
#ifdef \_WIN64   
ULONG64 Reserved; // Win64 requires these   
structures to be16 bytealigned.   
#endif   
} OBJECT\_HEADER\_QUOTA\_INFO,   
\*POBJECT\_HEADER\_QUOTA\_INFO;   
  
typedef struct \_OBJECT\_HEADER\_HANDLE\_INFO {   
union{   
POBJECT\_HANDLE\_COUNT\_DATABASE   
HandleCountDataBase;   
OBJECT\_HANDLE\_COUNT\_ENTRY SingleEntry;   
  
};   
} OBJECT\_HEADER\_HANDLE\_INFO,   
\*POBJECT\_HEADER\_HANDLE\_INFO;   
  
typedef struct \_OBJECT\_HEADER\_NAME\_INFO {   
  
POBJECT\_DIRECTORY Directory; // 对象所属的路径  
UNICODE\_STRING Name;// 对象名  
ULONG QueryReferences;   
  
#if DBG   
ULONG Reserved2;   
LONG DbgDereferenceCount;   
  
#ifdef \_WIN64   
  
ULONG64 Reserved3; // Win64 requires these   
structures to be16 bytealigned.   
#endif   
#endif   
} OBJECT\_HEADER\_NAME\_INFO, \*POBJECT\_HEADER\_   
  
  
NAME\_INFO;   
  
typedef struct \_OBJECT\_HEADER\_CREATOR\_INFO {   
  
LIST\_ENTRY TypeList; // 系统可以将所有同类型的对象串  
  
起来  
HANDLE CreatorUniqueProcess;// 对象父进程的 ID   
USHORT CreatorBackTraceIndex;   
USHORT Reserved;   
  
} OBJECT\_HEADER\_CREATOR\_INFO,   
\*POBJECT\_HEADER\_CREATOR\_INFO;   
  
对这些可选择的数据结构的总结:   
  
1\_OBJECT\_HEADER\_QUOTA\_INFO 中的 PEPROCESS ExclusiveProcess 指示了拥有此对象的进程  
2\_OBJECT\_HEADER\_HANDLE\_INFO 中存储着对象句柄相关的信息  
3\_OBJECT\_HEADER\_NAME\_INFO 有对象所属的路径和对象的名字  
4\_OBJECT\_HEADER\_CREATOR\_INFO, 系统用 LIST\_ENTRY TypeList 将所有同类型的对象串起来  
  
接着就是对象头  
  
typedef struct \_OBJECT\_HEADER {   
LONG\_PTR PointerCount;   
union{   
  
LONG\_PTR HandleCount;   
  
PVOID NextToFree;  
};  
POBJECT\_TYPE Type;  
UCHAR NameInfoOffset;  
UCHAR HandleInfoOffset;  
UCHAR QuotaInfoOffset;  
UCHAR Flags;  
  
  
union{   
  
POBJECT\_CREATE\_INFORMATION   
ObjectCreateInfo;   
PVOID QuotaBlockCharged;   
};   
  
PSECURITY\_DESCRIPTOR SecurityDescriptor;   
  
QUAD Body;   
} OBJECT\_HEADER, \*POBJECT\_HEADER;   
  
对象头结构的总结  
0 PointerCount, 引用记数,它是计算内核模式组件引用对象地址的次数  
1 HandleCount, 打开句柄计数 2Type, 对象的类型  
3 NameInfoOffset,\_OBJECT\_HEADER\_NAME\_INFO 的偏移  
4 HandleInfoOffset,\_OBJECT\_HEADER\_HANDLE\_INFO 的偏移  
5 QuotaInfoOffset,\_OBJECT\_HEADER\_QUOTA\_INFO 的偏移  
6 SecurityDescriptor, 对象的安全描述符  
  
接着就是对象体.. 不同类型的对象这里不一样... 比如驱动对象\_DRIVER\_OBJECT   
  
lkd> DT \_DRIVER\_OBJECT   
  
nt!\_DRIVER\_OBJECT   
+0x000 Type : Int2B   
+0x002 Size : Int2B   
+0x004DeviceObject :Ptr32 \_DEVICE\_OBJECT   
+0x008 Flags : Uint4B   
+0x00cDriverStart :Ptr32Void   
+0x010DriverSize :Uint4B   
+0x014DriverSection :Ptr32Void   
+0x018 DriverExtension : Ptr32 \_DRIVER\_EXTENSION   
+0x01c DriverName : \_UNICODE\_STRING   
+0x024 HardwareDatabase: Ptr32 \_UNICODE\_STRING   
+0x028FastIoDispatch :Ptr32 \_FAST\_IO\_DISPATCH   
+0x02c DriverInit : Ptr32 long   
+0x030 DriverStartIo : Ptr32 void   
+0x034 DriverUnload : Ptr32 void   
+0x038 MajorFunction : \[28\] Ptr32 long   
  
一般对象指针就是指向对象的对象体,如果得到一个对象体,想访问对象头的话. 可以使用宏 OBJECT\_TO\_OBJECT\_HEADER()   
  
2 一个例子  
  
eg:   
  
lkd> !object \\Driver\\beep   
  
Object:84fd2448Type: (86243e70) Driver  
ObjectHeader:84fd2430(oldversion)  
HandleCount:0 PointerCount:3  
Directory Object:e1864448Name: Beep  
  
  
可以看到 beep 的 ObjectHeader 的地址 84fd2430   
  
lkd> dtnt!\_object\_header 84fd2430   
  
  
nt!\_OBJECT\_HEADER   
  
+0x000PointerCount :3   
  
+0x004HandleCount :0   
  
+0x004NextToFree :(null)   
  
+0x008 Type : 0x86243e70 \_OBJECT\_TYPE   
  
+0x00c NameInfoOffset : 0x10 ''   
  
+0x00d HandleInfoOffset :0 ''   
  
+0x00e QuotaInfoOffset: 0 ''   
  
+0x00fFlags :0x32 '2'   
  
+0x010 ObjectCreateInfo : 0x00000001 \_OBJECT\_CREATE\_INFORMATION   
  
+0x010 QuotaBlockCharged : 0x00000001   
  
+0x014 SecurityDescriptor: 0xe1871aa3   
  
+0x018 Body : \_QUAD   
  
1 +0x000 PointerCount : 3 表示内核模式组件引用对象地址的次数为 3   
  
ObfReferenceObject(),ObReferenceObjectByHandle(), ObReferenceObjectByName() 和  
ObReferenceObjectByPointer() 增加的就是这个 PointerCount, ObfDereferenceObject  
  
()   
和 ObDereferenceObject() 用来减少这个值。  
删除对象跟这个值有关  
如果一个对象是临时的,那当踏的引用记数为 0 的时候,该对象会被系统删除,一个驱动程序  
可以把一个临时对象的引用记数减少到 0, 这样就可以达到删除该对象的目的  
如果一个对象是在创建时标上了 OBJ\_PERMANENT flag, 那它是一个永久的对象,如果要  
删除一个永久性的对象,需要经过下列步骤:   
1 调用 ObDereferenceObject 把一个永久性的对象的引用记数减少到 0   
2 调用 ZwOpenXxx orZwCreateXxx 来获得该永久性对象的一个句柄  
3 或得句柄后调用 ZwMakeTemporaryObject 把一个永久性的对象转化成一个临时的对象  
4 用得到的句柄调用 ZwClose 删除该对象  
所以确定一个对象是临时的和永久的由 InitializeObjectAttributes 宏的 Attributes 决定  
  
2 +0x004 HandleCount : 0 表示打开句柄数是 0   
  
3+0x008Type :0x86243e70 \_OBJECT\_TYPE 表示对象的类型  
lkd> dt\_OBJECT\_TYPE 0x86243e70   
nt!\_OBJECT\_TYPE   
  
+0x000 Mutex : \_ERESOURCE   
  
+0x038TypeList :\_LIST\_ENTRY\[0x86243ea8 -0x86243ea8\]   
  
+0x040 Name : \_UNICODE\_STRING "Driver"   
  
+0x048DefaultObject :0x80562e20   
  
+0x04cIndex :0x1a   
  
+0x050 TotalNumberOfObjects: 0x72   
  
+0x054 TotalNumberOfHandles : 0   
  
+0x058 HighWaterNumberOfObjects: 0x72   
  
+0x05c HighWaterNumberOfHandles :1   
  
+0x060 TypeInfo : \_OBJECT\_TYPE\_INITIALIZER   
  
+0x0acKey :0x76697244   
  
+0x0b0ObjectLocks :\[4\]\_ERESOURCE   
  
  
+0x040Name :\_UNICODE\_STRING"Driver" 表示对象的类型是"Driver"   
lkd> DT \_OBJECT\_TYPE\_INITIALIZER 0x86243e70+0X60   
nt!\_OBJECT\_TYPE\_INITIALIZER   
  
+0x000 Length : 0x4c   
  
+0x002UseDefaultObject:0x1 ''   
  
+0x003 CaseInsensitive : 0x1 ''   
  
+0x004 InvalidAttributes :0x100   
  
+0x008 GenericMapping : \_GENERIC\_MAPPING   
  
+0x018 ValidAccessMask :0x1f01ff   
  
+0x01c SecurityRequired :0 ''   
  
+0x01d MaintainHandleCount :0 ''   
  
+0x01e MaintainTypeList : 0 ''   
  
+0x020PoolType :0(NonPagedPool)   
  
+0x024 DefaultPagedPoolCharge :0   
  
+0x028 DefaultNonPagedPoolCharge: 0xd8   
  
+0x02c DumpProcedure : (null)   
  
+0x030 OpenProcedure : (null)   
  
+0x034 CloseProcedure : (null)   
  
+0x038DeleteProcedure:0x805833a2 voidnt!IopDeleteDriver+0   
  
+0x03c ParseProcedure : (null)   
  
+0x040SecurityProcedure:0x805f79fe longnt!SeDefaultObjectMethod+0   
  
+0x044 QueryNameProcedure : (null)   
  
+0x048 OkayToCloseProcedu   
  
这个地方的东西比较有趣,分别对应着对象的删除、lookup 、获取名字等的例程，一般对象  
不是所有的 routine 都有。这里有 IopDeleteDriver 和 SeDefaultObjectMethod   
  
4 +0x00c NameInfoOffset : 0x10 '' 表示\_OBJECT\_HEADER\_NAME\_INFO 的偏移是  
0X10   
lkd> dtnt!\_object\_header\_name\_info 84fd2430-0x10   
nt!\_OBJECT\_HEADER\_NAME\_INFO   
  
+0x000 Directory : 0xe1864448 \_OBJECT\_DIRECTORY   
  
+0x004 Name : \_UNICODE\_STRING "Beep"   
  
+0x00c QueryReferences : 1   
我们看到了对象的名字"Beep" 和对象所属的路径  
lkd>dt\_OBJECT\_DIRECTORY 0xe1864448   
nt!\_OBJECT\_DIRECTORY   
  
+0x000HashBuckets :\[37\]0xe185fb10 \_OBJECT\_DIRECTORY\_ENTRY   
  
+0x094 Lock : \_EX\_PUSH\_LOCK   
  
+0x098 DeviceMap : (null)   
  
+0x09c SessionId : 0xffffffff   
  
+0x0a0 Reserved : 0   
  
+0x0a2 SymbolicLinkUsageCount : 0   
关于对象目录慢点再讲  
  
  
5 +0x00d HandleInfoOffset: 0 ''//0 表示没有该结构  
  
+0x00e QuotaInfoOffset :0 ''//0 表示没有该结构  
  
6 +0x014 SecurityDescriptor: 0xe1871aa3 它决定了谁能对该对象进行什么样的操作  
  
lkd> dt\_SECURITY\_DESCRIPTOR 0xe1871aa3   
  
nt!\_SECURITY\_DESCRIPTOR  
+0x000 Revision : 0x80 ''  
+0x001 Sbz1 : 0x58 'X'  
+0x002 Control : 0  
+0x004 Owner : 0x00006800  
+0x008 Group : (null)  
+0x00cSacl :0x00001400 \_ACL  
+0x010 Dacl : 0x44000200 \_ACL  
  
  
7 +0x010 ObjectCreateInfo :0x00000001 \_OBJECT\_CREATE\_INFORMATION 对象标志中 OB\_FLAG\_CREATE\_INFO 被标识  
  
3 对象组织  
  
目录对象  
  
WINDOWS 中的各种类型的对象都独立地存在于系统地址空间中,怎么组织起来?   
系统利用目录对象将所有的这些对象组织起来,目录对象可以看成是存储对象的容器。它是  
一种特殊的对象,既然是对象,那就有个数据结构了,数据结构如下：  
看一下"beep" 所在的对象目录  
lkd> dt \_OBJECT\_DIRECTORY 0xe1864448   
nt!\_OBJECT\_DIRECTORY   
  
+0x000HashBuckets :\[37\]0xe185fb10 \_OBJECT\_DIRECTORY\_ENTRY  
+0x094 Lock : \_EX\_PUSH\_LOCK  
+0x098 DeviceMap : (null)  
+0x09c SessionId : 0xffffffff  
+0x0a0 Reserved : 0  
+0x0a2 SymbolicLinkUsageCount : 0  
  
  
目录对象是一个有 37 个数组元素组成的哈希树,数组元素的类型是  
lkd> dt \_OBJECT\_DIRECTORY\_ENTRY 0xe185fb10   
nt!\_OBJECT\_DIRECTORY\_ENTRY   
  
+0x000 ChainLink : 0xe1872718 \_OBJECT\_DIRECTORY\_ENTRY  
+0x004Object :0x84fd2448  
  
  
lkd> dt \_OBJECT\_DIRECTORY\_ENTRY 0xe1872718   
  
nt!\_OBJECT\_DIRECTORY\_ENTRY   
+0x000 ChainLink : 0xe1869278 \_OBJECT\_DIRECTORY\_ENTRY   
+0x004Object :0x8621c600   
  
dt nt!\_DRIVER\_OBJECT 0x8621c600 知道  
  
  
下一个驱动对象是\\Driver\\NDIS... 知道怎么连接了吧  
系统将对象名称进行一定的算法得出一个 HASH 值,系统将所有相同 HASH 值的对象链接到  
响应的数组项中  
  
类型对象  
  
系统中每种类型对象只有一个类型对象。每种类型的对象都在其对象体中存在一个指向其类  
型对象的指针，因为一种类型对象只有一个实体，所以每种类型对象的指针都是固定的，  
  
这样我们就可以通过对象体中的类型对象指针来判断和访问对象的类型了  
各个类型对象的对象体内并没有链表结构使得它们相互链接起来。但是假如对象头部前面  
有 OBJECT\_CREATOR\_INFO 结构，则相同类型的对象就可以通过它的成员 ObjectList 相互链  
接起来了。  
  
每个对象头部前面有一个 OBJECT\_CREATOR\_INFO 的可选结构。其结构如下：  
  
lkd> dt \_OBJECT\_HEADER\_CREATOR\_INFO   
  
nt!\_OBJECT\_HEADER\_CREATOR\_INFO   
+0x000 TypeList : \_LIST\_ENTRY   
+0x008 CreatorUniqueProcess :Ptr32 Void   
+0x00c CreatorBackTraceIndex :Uint2B   
+0x00e Reserved : Uint2B   
  
+0x000 TypeList : \_LIST\_ENTRY 好东西......... 呵呵  
  
  
4 操作对象  
  
可以用对象句柄来使用对象.怎么得到一个对象的句柄呢.ZwCreateXxx 和 ZwOpenXxx 就可  
以了,比如  
  
NTSTATUS   
  
ZwOpenKey(   
OUT PHANDLE KeyHandle,   
IN ACCESS\_MASK DesiredAccess,   
IN POBJECT\_ATTRIBUTES ObjectAttributes   
);   
  
DesiredAccess, 指定对对象的权限操作,比如可读,可写  
  
ObjectAttributes, 指向 OBJECT\_ATTRIBUTES 结构的一个指针, 先看下  
OBJECT\_ATTRIBUTES 的定义吧  
  
typedef struct \_OBJECT\_ATTRIBUTES{   
ULONG Length;   
HANDLE RootDirectory;   
PUNICODE\_STRING ObjectName;   
ULONG Attributes;   
PSECURITY\_DESCRIPTOR SecurityDescriptor;   
PSECURITY\_QUALITY\_OF\_SERVICE SecurityQualityService;   
  
}   
  
  
我们重要的地方在  
HANDLE RootDirectory;// 对象所在的对象目录  
PUNICODE\_STRING ObjectName;// 将被创建和打开的对象的名字  
一般我们初始话 OBJECT\_ATTRIBUTES 用 InitializeObjectAttributes   
VOID   
InitializeObjectAttributes(   
  
OUT POBJECT\_ATTRIBUTES InitializedAttributes,  
IN PUNICODE\_STRING ObjectName,  
IN ULONG Attributes,  
IN HANDLE RootDirectory,  
IN PSECURITY\_DESCRIPTOR SecurityDescriptor  
);  
  
  
一般把 RootDirectory 设为 NULL 就可以.ObjectName 给出全路劲  
或者也可以用 ZwCreateDirectoryObject 获得一个目录句柄  
比如\\DosDevices\\C:\\Directory\\File.   
目录名可以是\\DosDevices\\C:\\Directory\\   
文件名 File   
  
看几个跟对象有关的函数:   
  
NTSTATUS   
  
ObReferenceObjectByHandle(   
IN HANDLE Handle,   
IN ACCESS\_MASK DesiredAccess,   
IN POBJECT\_TYPE ObjectType OPTIONAL, // 驱动程序一般设为 NULL 就可以了  
IN KPROCESSOR\_MODE AccessMode, //KernelMode   
OUT PVOID \*Object,   
OUT POBJECT\_HANDLE\_INFORMATION HandleInformation OPTIONAL //NULL   
);   
  
由对象的句柄获得对象体,当然,有了对象体,你可以得到其对象头  
  
NTKERNELAPI   
  
NTSTATUS   
  
ObReferenceObjectByName(   
\_\_inPUNICODE\_STRING ObjectName,   
\_\_inULONG Attributes,   
\_\_in\_opt PACCESS\_STATEAccessState,   
\_\_in\_optACCESS\_MASK DesiredAccess,   
\_\_inPOBJECT\_TYPE ObjectType,   
\_\_inKPROCESSOR\_MODE AccessMode,   
\_\_inout\_opt PVOID ParseContext,   
\_\_out PVOID \*Object   
);   
  
对象名字得到对象体  
  
到此为止应该可以完了...... 当然还有很多内容... 但对象的总体应该有个概况  
下面是用来检测隐藏驱动的...VOID  Scan\_Object\_Directory()  
{  
    UNICODE\_STRING ObjectName;  
    OBJECT\_ATTRIBUTES oa ;  
    HANDLE OpenObject;  
    NTSTATUS status, ntStatus;  
    ULONG count=0;  
      
    PDIRECTORY\_BASIC\_INFORMATION  pBuffer2, pBuffer=NULL;  
    ULONG                uLength = 0x800;  
    ULONG                uContext = 0;  
    ULONG                uResult = 0;  
    WCHAR ObjName\[256\]=L"\\\\Driver\\\\";  
      
    RtlInitUnicodeString(&ObjectName, L"\\\\Driver");  
    InitializeObjectAttributes(&oa,&ObjectName, OBJ\_CASE\_INSENSITIVE,   
  
NULL, NULL);  
    status=    ZwOpenDirectoryObject(&OpenObject,DIRECTORY\_QUERY,&oa);  
    if(!NT\_SUCCESS(status))  
    {  
        DbgPrint("failed\\n");  
        return;  
    }  
      
    do  
    {  
        if (pBuffer)  
        ExFreePool(pBuffer);  
      
        uLength \*= 2;  
        pBuffer=(PDIRECTORY\_BASIC\_INFORMATION)ExAllocatePool(NonPagedPool   
  
,uLength);  
        ntStatus = ZwQueryDirectoryObject(OpenObject, pBuffer,   
  
uLength, FALSE, TRUE, &uContext, &uResult);  
    } while(ntStatus == STATUS\_MORE\_ENTRIES || ntStatus ==   
  
STATUS\_BUFFER\_TOO\_SMALL);  
      
    if (ntStatus==STATUS\_SUCCESS)  
    {  
        pBuffer2=pBuffer;  
        while (pBuffer2->ObjectName.Length!=0&&pBuffer2-  
  
\>ObjectTypeName.Length!=0)  
        {  
            wcscat(ObjName,pBuffer2->ObjectName.Buffer);  
              
            GetDriverObjByName(ObjName);  //自己实现吧..到这里有 ObjName 就够了  
              
            pBuffer2++;  
            count++;  
            RtlZeroMemory(ObjName,256);  
            wcscpy(ObjName,L"\\\\Driver\\\\");  
        }  
    }  
  
    DbgPrint("-----%d\\n",count);  
    ExFreePool(pBuffer);  
  
}  
  
  
下面是断链的..  
  
void ObDeleteObjectFromObjectDirectory(POBJECT\_DIRECTORY pObjDir) //父节点 pObjDir  
{  
    POBJECT\_DIRECTORY\_ENTRY pDirectoryEntry,pEntryPrev = NULL;  
    POBJECT\_HEADER pObjectHeader;  
    PVOID Object;  
    int i;  
    PUNICODE\_STRING unipObjectTypeName,uniHideDriveName;  
    POBJECT\_DIRECTORY tmpObjDir,pDirectory;  
      
    pDirectory = pObjDir;  
      
    for(i = 0; i < 36; i ++)  //DIR 对象的对象体(BODY)是 37 个元素的数组。  
    {  
        pDirectoryEntry = pDirectory->HashTable;  //子根目录  
        // DbgPrint("---0x%x\\n", pDirectoryEntry);  
        while(pDirectoryEntry)  
        {  
            \_\_try{  
                if(pDirectoryEntry)  
                {  
                    Object = pDirectoryEntry->pObject ; //根目录对象  
                   pObjectHeader = OBJECT\_TO\_OBJECT\_HEADER( Object ); //根目录对象头  
                   unipObjectTypeName = (PUNICODE\_STRING)((\*(ULONG\*)((ULONG)pObjectHeader+8)+0x40));  
                    //这里可以输出很多类型的对象,但我们关心的是 Driver  
                   //DbgPrint("--%ws  0x%x\\n",unipObjectTypeName->Buffer,Object);  
  
    ///////////////////////////////////////////////////////////////////////////////////////////////////////      
                    // DKOM 判断  
                    if(wcscmp(unipObjectTypeName->Buffer,L"Driver")==0)  
                    {  
                        uniHideDriveName = (PUNICODE\_STRING)((ULONG)Object+0x1c);  
                        if(wcscmp(uniHideDriveName->Buffer,L"\\\\Driver\\\\NDIS")==0)  //想隐藏的驱动对象  
                        {  
                            //也可以用 wcschr..因为我实验的结果是有时候会出现 \\Driver\\Beep?? 之类的名字..  
                            if (pEntryPrev)  //前继有了  
                                pEntryPrev->NextEntry = pDirectoryEntry->NextEntry; //匹配就开始断练了  
                        }  
                        pEntryPrev = pDirectoryEntry;   
                          
                        //    DbgPrint(" %ws\\n",uniHideDriveName->Buffer);  
  
                    }  
    ///////////////////////////////////////////////////////////////////////////////////////////////////////                  
                    if(wcscmp(unipObjectTypeName->Buffer,L"Directory")==0)  
                    {  
                        tmpObjDir = Object;  
                       ObDeleteObjectFromObjectDirectory( tmpObjDir);  //如果对象的类型是"Directory" 递归处理之  
                    }  
                }  
            }\_\_except(EXCEPTION\_EXECUTE\_HANDLER)  
            {  
             ;  
            }  
           
         pDirectoryEntry = pDirectoryEntry->NextEntry;  
     }  
  }  
  
}