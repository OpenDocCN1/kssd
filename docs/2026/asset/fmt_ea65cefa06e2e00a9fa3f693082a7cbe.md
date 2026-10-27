
# [原创]Windows内核学习笔记之进程（下）

> **上篇：**[**Windows内核学习笔记之进程（上）**](https://bbs.pediy.com/thread-270756.htm)

## 五.句柄与句柄表

### 1.句柄

Windows执行体实现了一套对象机制来管理各种资源或实体。每种对象都有一个类型对象，类型对象定义了该类对象的一些特性和方法。对象管理器也定义了一个全局名字空间，提供了根据名称来解析对象的同一机制。类型对象通过提供自定义的`Parse`方法可以扩展此名字空间。对象管理器中的对象是执行体对象，它位于系统空间，考虑到安全性，在进程空间不能直接通过地址来引用它们。

在Windows系统中需要使用到句柄(handle)来管理进程中的对象引用。当一个进程利用名称来创建或打开一个对象时，将获得一个句柄，该句柄指向所创建或打开的对象。以后，该进程无须使用名称来引用对象，使用此句柄即可访问。这样即保证了安全性，也提高了引用对象的效率。当两个应用程序以共享方式打开了同一个文件，那么，它们将分别得到各种的句柄，且都可以通过句柄操作该文件。尽管两个应用程序得到的句柄的值并不相同，但是这两个句柄所指的文件却是同一个。因此，句柄只是一个对象引用，同一个对象在不同的环境下可能有不同的引用（句柄）值。

> 在Windows系统中，句柄是进程范围内的对象引用，换句话说，句柄仅在一个进程范围内才有效。一个进程的句柄传递给另一个进程后，句柄值将不再有效

### 2.进程句柄表

实际上，Windows支持的句柄是一个索引，指向该进程句柄表中的一个表项。进程句柄表由`EPROCESS`结构中的`ObjectTable`域来指向。句柄表第一项索引是4，第二项索引是8，依次类推。一个进程的句柄表包含了所有已被该进程打开的对象的指针。`ObjectTable`的类型为`HANDLE_TABLE`，该结构定义如下：

``` 
kd> dt _HANDLE_TABLE

ntdll!_HANDLE_TABLE

   +0x000 TableCode        : Uint4B

   +0x004 QuotaProcess     : Ptr32 _EPROCESS

   +0x008 UniqueProcessId  : Ptr32 Void

   +0x00c HandleTableLock  : [4] _EX_PUSH_LOCK

   +0x01c HandleTableList  : _LIST_ENTRY

   +0x024 HandleContentionEvent : _EX_PUSH_LOCK

   +0x028 DebugInfo        : Ptr32 _HANDLE_TRACE_DEBUG_INFO

   +0x02c ExtraInfoPages   : Int4B

   +0x030 FirstFree        : Uint4B

   +0x034 LastFree         : Uint4B

   +0x038 NextHandleNeedingPool : Uint4B

   +0x03c HandleCount      : Int4B

   +0x040 Flags            : Uint4B

   +0x040 StrictFIFO       : Pos 0, 1 Bit
```

| **偏移** | **名称** | **作用** |
| --- | --- | --- |
| 0x000 | `TableCode` | 指向句柄表的存储结构 |
| 0x004 | `QuotaProcess` | 句柄表的内存资源记录在此进程中 |
| 0x008 | `UniqueProcessId` | 创建进程的ID，用于回调函数 |
| 0x00C | `HandleTableLock` | 句柄表锁，仅在句柄表扩展时使用 |
| 0x01C | `HandleTableList` | 所有句柄表形成一个链表，链表头为全局变量`HandleTableListHead` |
| 0x024 | `HandleContentionEvent` | 若在访问句柄时发生竞争，则在此推锁上等待 |
| 0x02C | `DebugInfo` | 调试信息，仅当调试句柄时才有意义 |
| 0x030 | `ExtraInfoPages` | 审计信息所占用的页面数量 |
| 0x034 | `FirstFree` | 空闲链表表头句柄索引 |
| 0x038 | `NextHandleNeedingPool` | 下一次句柄表扩展的起始句柄索引 |
| 0x03C | `HandleCount` | 正在使用的句柄表项的数量 |
| 0x040 | `Flags` | 标志域 |
| 0x040 | `StrictFIFO` | 是否使用FIFO风格的重用，即先释放先重用 |

`TableCode`域是一个指针，指向句柄表最高层表项页面，它的低2位代表了当前句柄表的层数，具体情况如下

- 0：句柄表只有一层，此时进程最多容纳512个句柄
- 1：句柄表有两层，此时进程最多可容纳512 \* 1024 个句柄
- 2：句柄表有三层，三层树结构最多可容纳的句柄数是512 \* 1024 \* 1024，但是Windows执行体限定每个进程的句柄数不得超过2^24=16777246

下图显示了这三种情形，实际上，在每个最底层句柄表页面中，第一个句柄表项都有特殊用途，真正供进程使用的句柄表项是511个

![](https://bbs.kanxue.com/upload/attach/202112/835440_DHHPGDYRJB4MQCJ.jpg)

最低层句柄表每一项所指的都是一个句柄表项，句柄表项结构为`HANDLE_TABLE_ENTRY`，占8个字节，定义如下

``` 
kd> dt _HANDLE_TABLE_ENTRY

nt!_HANDLE_TABLE_ENTRY

   +0x000 Object           : Ptr32 Void

   +0x000 ObAttributes     : Uint4B

   +0x000 InfoTable        : Ptr32 _HANDLE_TABLE_ENTRY_INFO

   +0x000 Value            : Uint4B

   +0x004 GrantedAccess    : Uint4B

   +0x004 GrantedAccessIndex : Uint2B

   +0x006 CreatorBackTraceIndex : Uint2B

   +0x004 NextFreeTableEntry : Int4B
```

`Object`指针所指的就是句柄所代表的内核对象，它的最低3位有特殊的含义

| **位数** | **名称** | **含义** |
| --- | --- | --- |
| 第0位 | `OBJ_PROTECT_CLOSE` | 表示调用者是否允许关闭该句柄 |
| 第1位 | `OBJ_INHERIT` | 指示该进程所创建的子进程是否可以继承该句柄 |
| 第2位 | `OBJ_AUDIT_OBJECT_CLOSE` | 指示关闭该对象时是否产生一个审计事件 |

因此，想要获得句柄对象的地址需要将`Object`的低3位清0，但是此时所得到的对象地址指向的是对象头，偏移0x18的地址才是对象的真正地址。

接下来通过WinDbg来查找句柄对象来验证上述内容，首先使用如下代码来创建test进程，而该进程则是要得到进程PID为1488的进程句柄

``` 
#include <cstdio>

#include <windows.h>

 

int main()

{

        HANDLE handle = OpenProcess(PROCESS_ALL_ACCESS, FALSE, 1812);

 

        printf("%X\n", handle);



	       system("pause");

        return 0;

}
```

此时这个进程的PID是一个记事本进程的PID

![](https://bbs.kanxue.com/upload/attach/202112/835440_FK5NSZSNAVT9EB4.jpg)

在WinDbg中查找test进程的`EPROCESS`地址

![](https://bbs.kanxue.com/upload/attach/202112/835440_CTKSJKXK3ZUU3FC.jpg)

通过该地址来获得`ObjectTable`的值，该值保存了此进程`HANDLE_TABLE`的地址

![](https://bbs.kanxue.com/upload/attach/202112/835440_GPFGTF8N9MPGGME.jpg)

根据该值来获得`TableCode`

![](https://bbs.kanxue.com/upload/attach/202112/835440_3RDKG7MMC6GN4BY.jpg)

该值的低2位为0，所以这是一个单层句柄表结构，所以`TableCode`所指的地址保存了512个`HANDLE_TABLE_ENTRY`。而要得知打开的记事本进程对象是在句柄表中的位置，就需要通过句柄值进行索引，下图可以知道此次的句柄值为0x7E8，由于句柄的索引是从4开始每次递增4，所以将7E8除以4得到1FA就是句柄表的索引

![](https://bbs.kanxue.com/upload/attach/202112/835440_TKTYU3QUNMC24YP.jpg)

有了索引就可以在句柄表中查找对应的句柄对应的对象头地址

![](https://bbs.kanxue.com/upload/attach/202112/835440_ABHSXAY4NR4D6RY.jpg)

此时将低3位清0就得到了对象头地址，偏移0x18处就是句柄对应的内核对象

``` 
kd> dt 0x81bd3330 + 0x18 _EPROCESS

ntdll!_EPROCESS

   +0x000 Pcb              : _KPROCESS

   +0x06c ProcessLock      : _EX_PUSH_LOCK

   +0x070 CreateTime       : _LARGE_INTEGER 0x1d7f193`acd47dd4

   +0x078 ExitTime         : _LARGE_INTEGER 0x0

   +0x080 RundownProtect   : _EX_RUNDOWN_REF

   +0x084 UniqueProcessId  : 0x00000714 Void

   +0x088 ActiveProcessLinks : _LIST_ENTRY [ 0x81d5a6c0 - 0x81b41c10 ]

   +0x090 QuotaUsage       : [3] 0xa78

   +0x09c QuotaPeak        : [3] 0xaf0

   +0x0a8 CommitCharge     : 0x192

   +0x0ac PeakVirtualSize  : 0x2451000

   +0x0b0 VirtualSize      : 0x1fbe000

   +0x0b4 SessionProcessLinks : _LIST_ENTRY [ 0x81d5a6ec - 0x81b41c3c ]

   +0x0bc DebugPort        : (null) 

   +0x0c0 ExceptionPort    : 0xe138c5a8 Void

   +0x0c4 ObjectTable      : 0xe2e92558 _HANDLE_TABLE

   +0x0c8 Token            : _EX_FAST_REF

   +0x0cc WorkingSetLock   : _FAST_MUTEX

   +0x0ec WorkingSetPage   : 0x9cf3

   +0x0f0 AddressCreationLock : _FAST_MUTEX

   +0x110 HyperSpaceLock   : 0

   +0x114 ForkInProgress   : (null) 

   +0x118 HardwareTrigger  : 0

   +0x11c VadRoot          : 0x81b411b8 Void

   +0x120 VadHint          : 0x81953f68 Void

   +0x124 CloneRoot        : (null) 

   +0x128 NumberOfPrivatePages : 0xce

   +0x12c NumberOfLockedPages : 0

   +0x130 Win32Process     : 0xe3b2ce68 Void

   +0x134 Job              : (null) 

   +0x138 SectionObject    : 0xe31fe450 Void

   +0x13c SectionBaseAddress : 0x01000000 Void

   +0x140 QuotaBlock       : 0x819f39d0 _EPROCESS_QUOTA_BLOCK

   +0x144 WorkingSetWatch  : (null) 

   +0x148 Win32WindowStation : 0x0000003c Void

   +0x14c InheritedFromUniqueProcessId : 0x000004e0 Void

   +0x150 LdtInformation   : (null) 

   +0x154 VadFreeHint      : (null) 

   +0x158 VdmObjects       : (null) 

   +0x15c DeviceMap        : 0xe17ce150 Void

   +0x160 PhysicalVadList  : _LIST_ENTRY [ 0x81bd34a8 - 0x81bd34a8 ]

   +0x168 PageDirectoryPte : _HARDWARE_PTE_X86

   +0x168 Filler           : 0

   +0x170 Session          : 0xf89d3000 Void

   +0x174 ImageFileName    : [16]  "notepad.exe"
```

偏移0x174的地址保存了进程对象的名称为`notepad.exe`，证明上述内容正确。

### 3.创建句柄

从上面的内容可以知道，要为一个进程创建句柄并将其插入句柄表的时候，需要为句柄准备一个临时的`HANDLE_TABLE_ENTRY`数据结构，使其指向这个对象的头部，然后将该结构插入到相应的句柄表中。这里就有两个问题，一个是填充`HANDLE_TABLE_ENTRY`数据结构，用来表示要插入的对象，另一个是在相应的句柄表中将其插入。

实现句柄创建的函数为`ObpCreateHandle`，在函数最开始就是取出对象的对象类型和对象头赋值到相应的变量
