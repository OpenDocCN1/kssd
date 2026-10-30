
详见：[3baK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2K6M7%4c8A6j5#2)9J5k6h3!0J5k6#2)9J5c8X3#2W2k6r3W2S2i4K6u0r3f1#2y4f1d9f1x3J5x3o6t1H3i4K6u0r3f1#2y4f1d9f1y4Q4x3X3c8S2j5%4c8W2M7#2)9J5c8Y4m8G2L8$3I4Q4y4h3k6G2N6X3g2J5k6X3I4G2N6#2)9#2k6X3g2^5M7r3I4G2K9i4c8S2N6r3W2G2L8W2)9#2k6Y4y4A6L8X3y4W2i4K6g2X3N6$3W2F1k6r3!0%4M7#2)9#2k6U0p5H3i4K6g2X3x3e0W2Z5x3g2)9J5c8W2y4e0g2p5W2o6x3U0l9J5x3q4)9J5k6p5q4J5N6r3W2U0L8r3g2Q4x3X3c8H3L8$3!0D9i4K6g2X3L8%4k6W2M7X3k6D9L8%4N6Q4y4h3k6W2P5s2m8D9L8$3W2@1j5i4c8A6L8$3&6Q4y4h3k6K6K9h3&6U0k6g2)9#2k6Y4N6A6L8X3c8G2N6%4y4Q4y4h3j5I4x3q4)9#2k6U0p5&6K9o6q4Q4x3X3c8T1j5i4W2W2N6q4)9#2k6X3k6S2M7X3W2W2L8r3I4G2i4K6u0W2M7r3c8X3](<https://bbs.kanxue.com/elink@7c0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2K6M7%4c8A6j5#2)9J5k6h3!0J5k6#2)9J5c8X3#2W2k6r3W2S2i4K6u0r3f1#2y4f1d9f1x3J5x3o6t1H3i4K6u0r3f1#2y4f1d9f1y4Q4x3X3c8S2j5%4c8W2M7#2)9J5c8Y4m8G2L8$3I4Q4y4h3k6G2N6X3g2J5k6X3I4G2N6#2)9#2k6X3g2^5M7r3I4G2K9i4c8S2N6r3W2G2L8W2)9#2k6Y4y4A6L8X3y4W2i4K6g2X3N6$3W2F1k6r3!0%4M7#2)9#2k6U0p5H3i4K6g2X3x3e0W2Z5x3g2)9J5c8W2y4e0g2p5W2o6x3U0l9J5x3q4)9J5k6p5q4J5N6r3W2U0L8r3g2Q4x3X3c8H3L8$3!0D9i4K6g2X3L8%4k6W2M7X3k6D9L8%4N6Q4y4h3k6W2P5s2m8D9L8$3W2@1j5i4c8A6L8$3&6Q4y4h3k6K6K9h3&6U0k6g2)9#2k6Y4N6A6L8X3c8G2N6%4y4Q4y4h3j5I4x3q4)9#2k6U0p5&6K9o6q4Q4x3X3c8T1j5i4W2W2N6q4)9#2k6X3k6S2M7X3W2W2L8r3I4G2i4K6u0W2M7r3c8X3>)

中文翻译：[201K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5i4m8W2M7W2)9J5k6i4y4W2k6h3u0#2k6#2)9J5k6h3!0J5k6#2)9J5c8U0p5%4y4o6x3](<https://bbs.kanxue.com/elink@6a9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5i4m8W2M7W2)9J5k6i4y4W2k6h3u0#2k6#2)9J5k6h3!0J5k6#2)9J5c8U0p5%4y4o6x3>)

### POOL_HEADER

`POOL_HEADER` 在池中，适合单个页面的所有块都以 `POOL_HEADER` 结构开头，`POOL_HEADER` 包含分配器所需信息和 Tag 信息。当试图在 Windows 内核中利用堆溢出漏洞时，首先要覆盖的就是 `POOL_HEADER` 结构。攻击者有两个选择：重写一个正确的 `POOL_HEADER` 结构，并用来攻击下一个块的数据，或者直接攻击 `POOL_HEADER` 结构。

```c
struct POOL_HEADER {
    char PreviousSize; // 之前的块的大小除以16
    char PoolIndex; // PoolDescriptor数组中的索引
    char BlockSize; // 当前分配的大小除以16
    char PoolType; // 包含分配类型信息的位域
    int PoolTag;
    Ptr64 ProcessBilled; // 指向分配内存的进程的KPROCESS的指针，只有PoolType中包含PoolQuota标志时，才设置此字段。
};
```

`PoolType` 是一个位域，存储若干信息：

- 使用的内存类型，可以是 `NonPagedPool`、`PagedPool`、`SessionPool` 或 `NonPagedPoolNx`；
- 如果分配是关键的（bit 1）并且必须成功，那么当分配失败，就会触发 BugCheck；
- 如果分配与缓存大小对齐（bit 2）；
- 如果分配使用了 `PoolQuota` 机制（bit 3）；
- 其他未文档化的机制。

```c
NonPagedPool = 0
PagedPool = 1
NonPagedPoolMustSucceed = 2
DontUseThisType = 3
NonPagedPoolCacheAligned = 4
PagedPoolCacheAligned = 5
NonPagedPoolCacheAlignedMustSucceed = 6
MaxPoolType = 7
PoolQuota = 8
NonPagedPoolSession = 20h
PagedPoolSession = 21h
NonPagedPoolMustSucceedSession = 22h
DontUseThisTypeSession = 23h
NonPagedPoolCacheAlignedSession = 24h
PagedPoolCacheAlignedSession = 25h
NonPagedPoolCacheAlignedMustSSession = 26h
NonPagedPoolNx = 200h
NonPagedPoolNxCacheAligned = 204h
NonPagedPoolSessionNx = 220h
```

## 相对偏移地址读写

### WNF

`WNF` (Windows Notification Facility) 是 Windows 中的一个通知系统。应用程序可以订阅特定类型的事件（`StateName` 标识），在每次状态更改时可以进行通知。

`WNF` 利用相关文章：

[c03K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4H3L8r3q4&6k6i4u0Q4x3X3g2F1k6i4c8Q4x3V1j5I4y4o6f1H3x3K6l9^5y4o6q4Q4x3X3c8f1K9r3g2Q4x3X3c8%4K9h3&6V1L8%4N6K6i4K6u0V1L8X3!0@1K9h3k6A6j5$3q4@1K9h3!0F1i4K6u0V1k6X3q4U0K9h3I4A6N6s2W2Q4x3X3g2Z5N6r3#2D9](<https://bbs.kanxue.com/elink@02bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4H3L8r3q4&6k6i4u0Q4x3X3g2F1k6i4c8Q4x3V1j5I4y4o6f1H3x3K6l9^5y4o6q4Q4x3X3c8f1K9r3g2Q4x3X3c8%4K9h3&6V1L8%4N6K6i4K6u0V1L8X3!0@1K9h3k6A6j5$3q4@1K9h3!0F1i4K6u0V1k6X3q4U0K9h3I4A6N6s2W2Q4x3X3g2Z5N6r3#2D9)>)

[93cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2M7i4g2S2M7X3E0K6L8r3q4T1i4K6u0W2j5$3!0E0i4K6u0r3M7r3I4S2P5h3W2F1k6#2)9J5k6s2N6A6N6r3S2Q4x3X3c8@1K9r3g2Q4x3X3c8%4K9h3&6V1L8%4N6K6i4K6u0V1L8X3!0@1K9h3k6A6j5$3q4@1K9h3!0F1i4K6u0V1k6X3q4U0K9h3I4A6N6s2W2Q4x3X3c8%4L8X3k6Q4x3X3g2Z5N6r3#2D9](<https://bbs.kanxue.com/elink@9e0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2M7i4g2S2M7X3E0K6L8r3q4T1i4K6u0W2j5$3!0E0i4K6u0r3M7r3I4S2P5h3W2F1k6#2)9J5k6s2N6A6N6r3S2Q4x3X3c8@1K9r3g2Q4x3X3c8%4K9h3&6V1L8%4N6K6i4K6u0V1L8X3!0@1K9h3k6A6j5$3q4@1K9h3!0F1i4K6u0V1k6X3q4U0K9h3I4A6N6s2W2Q4x3X3c8%4L8X3k6Q4x3X3g2Z5N6r3#2D9)>)

结构体 `_WNF_STATE_DATA` 大可以由用户自定义

```c
struct _WNF_STATE_DATA {
    struct _WNF_NODE_HEADER Header; // 0x0
    ULONG AllocatedSize; // 0x4 // 分配的内核池大小
    ULONG DataSize; // 0x8 // 当前数据大小
    ULONG ChangeStamp; // 0xc
};
```

用户可以通过 `NtCreateWnfStateName` 创建一个 `WNF` 对象实例，实例的数据结构为 `_WNF_NAME_INSTANCE`；`WNF` 对象大小为 `0xb8`（`_WNF_NAME_INSTANCE` + `POOL_HEADER`）。

```c
typedef NTSTATUS (NTAPI * __NtCreateWnfStateName)(
    _Out_ PWNF_STATE_NAME StateName,
    _In_ WNF_STATE_NAME_LIFETIME NameLifetime,
    _In_ WNF_DATA_SCOPE DataScope,
    _In_ BOOLEAN PersistData,
    _In_opt_ PCWNF_TYPE_ID TypeId,
    _In_ ULONG MaximumStateSize,
    _In_ PSECURITY_DESCRIPTOR SecurityDescriptor
);
```

```c
struct _WNF_NAME_INSTANCE {
    struct _WNF_NODE_HEADER Header; // 0x0
    struct _EX_RUNDOWN_REF RunRef; // 0x8
    struct _RTL_BALANCED_NODE TreeLinks; // 0x10
    struct _WNF_STATE_NAME_STRUCT StateName; // 0x28
    struct _WNF_SCOPE_INSTANCE * ScopeInstance; // 0x30
    struct _WNF_STATE_NAME_REGISTRATION StateNameInfo; // 0x38
    struct _WNF_LOCK StateDataLock; // 0x50
    struct _WNF_STATE_DATA * StateData; // 0x58
    ULONG CurrentChangeStamp; // 0x60
    VOID * PermanentDataStore; // 0x68
    struct _WNF_LOCK StateSubscriptionListLock; // 0x70
    struct _LIST_ENTRY StateSubscriptionListHead; // 0x78
    struct _LIST_ENTRY TemporaryNameListEntry; // 0x88
    struct _EPROCESS * CreatorProcess; // 0x98
    LONG DataSubscribersCount; // 0xa0
    LONG CurrentDeliveryCount; // 0xa4
};
```

`NtUpdateWnfStateData` 可以往对象里写入数据，使用 `_WNF_STATE_DATA` 结构存储写入的内容；

```c
typedef NTSTATUS (NTAPI * __NtUpdateWnfStateData)(
    _In_ PWNF_STATE_NAME StateName,
    _In_reads_bytes_opt_(Length) const VOID * Buffer,
    _In_opt_ ULONG Length,
    _In_opt_ PCWNF_TYPE_ID TypeId,
    _In_opt_ const PVOID ExplicitScope,
    _In_ WNF_CHANGE_STAMP MatchingChangeStamp,
    _In_ ULONG CheckStamp
);
```

通过 `NtQueryWnfStateData` 可以读取之前写入的数据，通过 `NtDeleteWnfStateData` 可以释放掉这个对象。`NtDeleteWnfStateData` 会调用 `ExpWnfDeleteStateData`。

```c
typedef NTSTATUS (NTAPI * __NtQueryWnfStateData)(
    _In_ PWNF_STATE_NAME StateName,
    _In_opt_ PWNF_TYPE_ID TypeId,
    _In_opt_ const VOID * ExplicitScope,
    _Out_ PWNF_CHANGE_STAMP ChangeStamp,
    _Out_writes_bytes_to_opt_(*BufferSize, *BufferSize) PVOID Buffer,
    _Inout_ PULONG BufferSize
);
```

`DataSize` 表示内存中 `_WNF_STATE_DATA` 结构的实际数据的大小，并用于 `NtQueryWnfStateData` 函数内的边界检查。`_WNF_STATE_DATA` 结构存储写入的内容的内存复制操作发生在函数 `ExpWnfReadStateData` 中。
