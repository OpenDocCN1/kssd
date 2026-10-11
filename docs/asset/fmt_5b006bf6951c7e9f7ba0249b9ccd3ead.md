
```
// first bin has valid value used for .log  
// correspondence testing, only meaningful  
// on disk.  
ULONG       Spare;          // this used to be MemAlloc. We don't use it anymore as we  
                            // can't afford to touch the bin (it's not residing in paged-pool  
                            // anymore, so touching it means modifying mnw pages).  
                            // Spare is used for the ShiftFreeBins Stuff - in memory only!  
//  
// Cell data goes here  
//  
} HBIN, *PHBIN;
```

所以一个 hive 的磁盘映像看起来如下图所示。

![](img/50bbc30c11177c9a1c0b797f74e4abfb.jpg)

是按照 4k 字节对齐的，由于其使用链表的结构，在内存中可以是不连续的。

为了处理内存中存放 hive 数据的地址不连续的的问题，配置管理器采取了一种类似 Windows 内存管理器将虚存地址映射到内存时所使用的策略。配置管理器用了一个两级系统。

![](img/bc6222f12963c84964af013185b6187.jpg)

如图所示，该系统以一个 `cell` 索引作为输入而后返回 `cell` 索引所在的 `block` 的内存地址以及 `cell` 所在的 `block` 的内存地址作为输出。

为了方便依据 `cell` 的索引来计算出 `cell` 的内存位置，在 Windows 内核提供了一个计算函数。这个函数的函数指针存放在一个称为 `_HHIVE` 的结构中。这个结构如下所示：

```
lkd> dt _HHIVE
nt!_HHIVE
    +0x000 Signature          : Uint4B
    +0x004 GetCellRoutine     : Ptr32      _CELL_DATA*
    +0x008 ReleaseCellRoutine : Ptr32      void
    +0x00c Allocate           : Ptr32      void*
    +0x010 Free               : Ptr32      void
    +0x014 FileSetSize        : Ptr32      unsigned char
    +0x018 FileWrite          : Ptr32      unsigned char
    +0x01c FileRead           : Ptr32      unsigned char
    +0x020 FileFlush          : Ptr32      unsigned char
    +0x024 BaseBlock          : Ptr32 _HBASE_BLOCK
    +0x028 DirtyVector        : _RTL_BITMAP
    +0x030 DirtyCount         : Uint4B
    +0x034 DirtyAlloc         : Uint4B
    +0x038 RealWrites         : UChar
    +0x03c Cluster           : Uint4B
    +0x040 Flat              : UChar
    +0x041 ReadOnly          : UChar
    +0x042 Log               : UChar
    +0x044 HiveFlags         : Uint4B
    +0x048 LogSize           : Uint4B
    +0x04c RefreshCount      : Uint4B
    +0x050 StorageTypeCount  : Uint4B
    +0x054 Version           : Uint4B
    +0x058 Storage           : [2] _DUAL
```

`GetCellRoutine` 就是计算函数，函数返回 `_CELL_DATA` 指针，即 `cell` 的地址。这个函数原型是

```
typedef struct _CELL_DATA * (*PGET_CELL_ROUTINE)(
    struct _HHIVE   *Hive,
    HCELL_INDEX Cell
    );
```

`GetCellRoutine` 是保存在 `_HHIVE` 结构中的 `PGET_CELL_ROUTINE` 类型的函数指针。参数 `HCELL_INDEX cell` 就是我们前面讲的 `cell` 索引。

而对于本函数的两个参数，又是怎样的得到呢？下面我们就谈谈两个重要的结构。即：

键对象（`key object`）和键控制块（`key control block`）

当一个应用程序打开或者创建一个注册表键的时候，对象管理器会给应用程序一个句柄，让它通过这个句柄来引用该键。该句柄所代表的 `key object` 是配置管理器在对象管理器的协助下分配的。有了对象管理器的支持，配置管理器就可以利用对象管理器所提供的安全及引用计数功能了。

对于每个打开的注册表键，配置管理器还会为它分配一个 `key control block`。一个 `key control block` 中存储了该键的完整路径名、该 `key control block` 所对应的键节点的 `cell` 索引以及一个标识用以指示在该键的最后一个句柄被关闭后，配置管理器是否要删除此 `key control block` 所对应的的 `key cell`。Windows 将所有的 `key control block` 放进一个哈希表中好用名字对存在的 `key control block` 进行快速查找。一个 `key object` 指向一个与它相关的 `key control block`，因此，如果有两个应用程序打开同一个注册表键，这两个程序各会获得一个 `key object`，而这两个 `key object` 都会指向同一个 `key control block`。

我们先看看 `key object` 的结构：

```
lkd> DT _cm_key_body
nt!_CM_KEY_BODY
    +0x000 Type               : Uint4B
    +0x004 KeyControlBlock    : Ptr32 _CM_KEY_CONTROL_BLOCK
    +0x008 NotifyBlock        : Ptr32 _CM_NOTIFY_BLOCK
    +0x00c ProcessID          : Ptr32 Void
    +0x010 Callers            : Uint4B
    +0x014 CallerAddress      : [10] Ptr32 Void
    +0x03c KeyBodyList        : _LIST_ENTRY
```

在该结构中有一项 `KeyControlBlock` ，就是配置管理器分配的 `key control block`。我们看下这个结构：

```
lkd> dt _CM_KEY_CONTROL_BLOCK
nt!_CM_KEY_CONTROL_BLOCK
    +0x000 RefCount           : Uint2B
    +0x004 ExtFlags          : Pos 0, 8 Bits
    +0x004 PrivateAlloc      : Pos 8, 1 Bit
    +0x004 Delete            : Pos 9, 1 Bit
    +0x004 DelayedCloseIndex : Pos 10, 12 Bits
    +0x004 TotalLevels       : Pos 22, 10 Bits
    +0x008 KeyHash           : _CM_KEY_HASH
    +0x008 ConvKey           : Uint4B
    +0x00c NextHash          : Ptr32 _CM_KEY_HASH
    +0x010 KeyHive           : Ptr32 _HHIVE
    +0x014 KeyCell           : Uint4B
    +0x018 ParentKcb         : Ptr32 _CM_KEY_CONTROL_BLOCK
    +0x01c NameBlock         : Ptr32 _CM_NAME_CONTROL_BLOCK
    +0x020 CachedSecurity    : Ptr32 _CM_KEY_SECURITY_CACHE
    +0x024 ValueCache        : _CACHED_CHILD_LIST
    +0x02c IndexHint         : Ptr32 _CM_INDEX_HINT_BLOCK
    +0x02c HashKey           : Uint4B
    +0x02c SubKeyCount       : Uint4B
    +0x030 KeyBodyListHead   : _LIST_ENTRY
    +0x030 FreeListEntry     : _LIST_ENTRY
    +0x038 KcbLastWriteTime  : _LARGE_INTEGER
    +0x040 KcbMaxNameLen     : Uint2B
    +0x042 KcbMaxValueNameLen : Uint2B
    +0x044 KcbMaxValueDataLen : Uint4B
    +0x048 KcbUserFlags      : Pos 0, 4 Bits
    +0x048 KcbVirtControlFlags : Pos 4, 4 Bits
    +0x048 KcbDebug          : Pos 8, 8 Bits
    +0x048 Flags             : Pos 16, 16 Bits
```

在其中有两个重要的项

`+0x014 KeyCell : Uint4B` 就是我们要隐藏的注册表项对应的 `cell` 索引。

`+0x010 KeyHive : Ptr32 _HHIVE` 就是我们前面刚刚讲过的 `_HHIVE` 结构指针。

注册表隐藏的步骤如下：

1. 打开指定名字的注册表键，获取其 `hkey`。例如：

```
// 打开指定名字的 Key
HANDLE OpenKeyByName(PCWSTR pwcsKeyName)
{
    NTSTATUS status;
    UNICODE_STRING uKeyName;
    OBJECT_ATTRIBUTES oa;
    HANDLE hKey;
    RtlInitUnicodeString(&uKeyName, pwcsKeyName);
    InitializeObjectAttributes(&oa, &uKeyName, OBJ_CASE_INSENSITIVE | OBJ_KERNEL_HANDLE, NULL, NULL);
    status = ZwOpenKey(&hKey, KEY_READ, &oa);
    if (!NT_SUCCESS(status))
    {
        DbgPrint("ZwOpenKey Failed: %lx\n", status);
        return NULL;
    }
    return hKey;
}
```

2. 根据得到的 `hkey` 句柄，得到其内核对象 `key object`，并从 `key object` 中获取键控制块（`key control block`）。例如：

```
PVOID GetKeyControlBlock(HANDLE hKey)
{
    NTSTATUS status;
    PCM_KEY_BODY KeyBody;
    PVOID KCB;
    if (hKey == NULL) return NULL;
    // 由 Key 句柄获取对象体
    status = ObReferenceObjectByHandle(hKey, KEY_READ, NULL, KernelMode, &KeyBody, NULL);
    if (!NT_SUCCESS(status))
    {
        DbgPrint("ObReferenceObjectByHandle Failed: %lx\n", status);
        return NULL;
    }
    // 对象体中含有 KeyControlBlock
    KCB = KeyBody->KeyControlBlock;
    DbgPrint("KeyControlBlock = %lx\n", KCB);
    ObDereferenceObject(KeyBody);
    return KCB;
}
```

3. 从 `KCB` 中获取前面讲过的两个重要项。

`+0x014 KeyCell : Uint4B` 就是我们要隐藏的注册表项对应的 `cell` 索引。

`+0x010 KeyHive : Ptr32 _HHIVE` 就是我们前面刚刚讲过的 `_HHIVE` 结构指针。

4. 在 `_HHIVE` 结构中获取 `GetCellRoutine` 就是计算函数，可以根据 `cell` 索引计算出 `cell` 的内存地址。

5. `hook` `GetCellRoutine`，判断如果返回的 `cell` 内存地址等于上面计算出的隐藏的 `CELL` 地址。则让其返回 `NULL`。

这就是要隐藏的 5 个步骤。

例子：这个例子是网上的，已经不清楚是哪位大侠的杰作了。在此表示感谢。

```
#include <ntddk.h>

#define GET_PTR(ptr, offset) ( *(PVOID*)( (ULONG)ptr + (offset##Offset) ) )

#define CM_KEY_INDEX_ROOT     0x6972        // ir
#define CM_KEY_INDEX_LEAF     0x696c        // il
#define CM_KEY_FAST_LEAF      0x666c        // fl
#define CM_KEY_HASH_LEAF      0x686c        // hl

// 一些 CM 的数据结构，只列出用到的开头部分
#pragma pack(1)

typedef struct _CM_KEY_NODE {
    USHORT Signature;
    USHORT Flags;
    LARGE_INTEGER LastWriteTime;
    ULONG Spare;            // used to be TitleIndex
    HANDLE Parent;
    ULONG SubKeyCounts[2];    // Stable and Volatile
    HANDLE SubKeyLists[2];    // Stable and Volatile
    // ...
} CM_KEY_NODE, *PCM_KEY_NODE;

typedef struct _CM_KEY_INDEX {
    USHORT Signature;
    USHORT Count;
    HANDLE List[1];
} CM_KEY_INDEX, *PCM_KEY_INDEX;

typedef struct _CM_KEY_BODY {
```
