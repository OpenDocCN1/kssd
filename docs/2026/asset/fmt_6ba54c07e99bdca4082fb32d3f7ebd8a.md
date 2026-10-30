
使用 `PipeAttribution` 构造任意地址读后，修改 `_WNF_NAME_INSTANCE` 结构体内的指针 `_WNF_STATE_DATA` 实现任意地址写

使用使用 `NtQueryWnfStateData` 和 `NtUpDateWnfStateData` API 来造成任意地址的读写（需要构造 `AllocateSize` 和 `DataSize` 成员）

<https://bbs.kanxue.com/thread-271140.htm>

漏洞在 windows 的 NTFS 文件系统驱动上 (`C:\Windows\System32\drivers\ntfs.sys`) 的 `NtfsQueryEaUserEaList` 函数中

NTFS 文件系统允许为每一个文件额外存储若干个键值对属性，称之为 EA(Extend Attribution)。可以通过 `ZwSetEaFile` 为文件创建 EA，`ZwQueryEaFile` 查询文件 EA

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13 | `typedef struct _FILE_GET_EA_INFORMATION {` `ULONG NextEntryOffset;` `UCHAR EaNameLength;` `CHAR EaName[1];` `} FILE_GET_EA_INFORMATION,` `*PFILE_GET_EA_INFORMATION;` `typedef struct _FILE_FULL_EA_INFORMATION {` `ULONG NextEntryOffset;` `UCHAR Flags;` `UCHAR EaNameLength;` `USHORT EaValueLength;` `CHAR EaName[1];` `} FILE_FULL_EA_INFORMATION,` `*PFILE_FULL_EA_INFORMATION;` |

## 漏洞成因

泄露的 NT5.1 中有 `NtfsQueryEaUserEaList` 的源码

### NT5.1 泄露的源码
