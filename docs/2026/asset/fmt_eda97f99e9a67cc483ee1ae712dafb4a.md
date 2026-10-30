
越界写的位置是 `signatureArray - 2`，主要受到偏移 `0x68` 位置的 `signatureOffset` 的影响；越界写的数据是 `\*(curSectorSize + blockHeader + 0x1FE)`，主要受到偏移 `0x4` 位置的 `nSector` 的影响，就是因为这里的数值太大才导致了崩溃的发生。

还有一些偏移位置的数据会影响程序流程，这里暂时不关注。

### 总结

- 写入位置：`blockHeader + \*(blockHeader + 0x68) + idx * 2`
- 写入数据：`\*(blockHeader + idx * 0x200 + 0x1FE)`

根据上面对于堆块结构的分析，如果 `blockHeader + 【offset】` 中 `offset % 16` 的结果是奇数，那么数据来源就是相对固定的堆块头部，否则数据来源就是未初始化的数据空间。

所以如果不考虑超出这块内存空间（即内部全部是 `0x10` 堆块的空间）的情况：写入数据所在的位置正位于堆块头部偏移 `0x0E` 的位置，该位置的数据绝大部分情况下均为 `0x0000`；写入位置可以通过类似堆喷射的方式进行控制。

## 5. 总结

通过以上分析可知，`CVE-2022-30220` 漏洞是由于对保存在堆块头部中的 `cBlocks` 字段的检查不够充分造成的，通过修改 BLF 文件中该字段数值，可以导致堆越界写的发生。由于堆空间结构的限制，越界写的位置与写入数据受到限制，如果想要进行漏洞利用，可能需要对堆空间结构更详细的分析，并通过修改 BLF 文件内容以及类似堆喷射的方式对数据进行控制。

## 6. 参考资料

1. [CLFS Internals](elink@d17K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6A6L8$3&6W2M7$3y4#2x3o6l9%4i4K6u0r3j5$3I4X3M7#2)9J5k6r3c8G2j5%4x3`.)

2. <mark class="encrypted">9e4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6D9K9h3u0&6j5h3I4Q4x3V1k6D9K9h3u0X3M7$3y4D9k6Y4y4Q4x3V1k6T1L8r3!0T1i4K6u0r3L8h3q4A6L8W2)9J5c8X3c8G2j5%4g2E0k6h3&6S2N6r3W2G2L8W2)9J5c8V1y4G2L8h3#2G2L8W2)9J5y4e0t1H3e0r3!0Y4i4K6t1#2x3U0m8r3K9h3I4W2i4K6t1#2x3U0m8e0P5i4y4@1k6h3#2Q4x3U0f1J5x3q4)9J5z5p5y4x3c8W2y4Q4x3U0W2Q4x3X3g2S2M7$3y4A6K9h3c8G2j5H3`.`.</mark>

3. Windows Internal editon 6th

原文链接：029K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0M7q4)9J5k6i4N6W2K9i4S2A6L8W2)9J5k6i4q4I4i4K6u0W2j5$3!0E0i4K6u0r3M7#2)9J5c8X3#2C8e0r3A6m8P5h3q4D9L8K6f1#2f1r3#2V1e0r3u0*7e0h3&6j5z5h3M7`.

## 1. 漏洞概述

| **项目** | **详情** |
| --- | --- |
| **名称** | Windows Common Log File System Driver 提权漏洞 |
| **简介** | CLFS 驱动中存在堆越界写漏洞，利用该漏洞可以实现本地提权 |
| **影响版本** | 该漏洞影响所有受支持的 Windows 操作系统 |
| **编号** | `CVE-2022-30220` |

## 2. CLFS基础

*注：以下所有涉及的数据结构根据参考资料整理得到，由于没有官方文档，所以可能存在未知或同名字段，但不影响此次漏洞分析。*

### 2.1 概念

`CLFS`(`Common Log File System`) 是一个日志框架，其他应用程序可以通过 `CLFS` 创建、保存和读取日志数据。“日志(log)”既可以表示一个抽象的概念，也可以表示一个物理上的文件，`CLFS` 对一条日志以及保存它的物理存储空间做了区分，因此物理空间的管理和日志的管理是分开的。`CLFS` 通过使用流(`stream`)和容器(`container`)在 `NTFS` 之上构建了一个虚拟的抽象层。

### 2.2 日志的存储

一个 `CLFS` 日志的存储由两部分组成：

- 包含元数据(`metadata`)的 base log file(`BLF`)

  `BLF`文件大小通常为 `64KB`，但是可以根据需要增长，其中包含了日志存储所需的一些元信息，例如日志的起始位置、容器的大小和路径、日志名称、客户端信息等。考虑到日志恢复的需要，`BLF` 文件中包含了一份元数据的拷贝，使用 `dump count` 参数来识别哪份信息是最新的。

- 最多 `1023` 个包含真正数据的容器文件

  容器是一个活动的日志流在空间上的基础分配单元，同一日志中的容器大小一致，是 `512KB`（一个扇区的大小）的倍数，最大为 `4GB`。`CLFS` 客户端通过增加和删除容器实现日志流的扩大和缩小。在实现时，`CLFS` 把容器当作 `BLF` 所在卷上的一个连续文件，通过在逻辑上将多个容器串在一起，形成包含一条日志的单个逻辑顺序磁盘区。初始化的时候一条日志至少要分配两个容器。

可以把整个日志存储看作是文件系统中一个卷的概念

#### 2.2.1 Log Blocks

`CLFS` 使用日志块对记录进行组织管理，每个日志块由多个 `512` 字节的扇区组成，之后对日志数据的读取和写入操作都在日志块上进行。

每个日志块的开头有一个 `_CLFS_LOG_BLOCK_HEADER`：

```
typedef struct _CLFS_LOG_BLOCK_HEADER
{
UCHAR MajorVersion;
UCHAR MinorVersion;
UCHAR Usn;
UCHAR StreamNum;
USHORT TotalSectorCount;
USHORT ValidSectorCount;
ULONG Padding;
ULONG Checksum;
ULONG Flags;
ULONG Unknown2;
CLFS_LSN CurrentLsn;
CLFS_LSN NextLsn;
ULONG RecordOffsets[16];
ULONG SignaturesOffset;
ULONG Unknown3;
} CLFS_LOG_BLOCK_HEADER;
```

当日志写入硬盘的时候，日志块会进行编码，编码状态的日志块中，每个扇区结尾处有一个 `2` 字节的扇区签名(`sector signature`)：

```
[Sector Block Type][Usn]
```

由于在编码阶段每个扇区的最后两个字节被改写成了扇区签名，因此需要对原始数据进行备份。日志块的最后一个扇区结尾处，有一个签名数组，这个数组中就保存了原始数据，由 `_CLFS_LOG_BLOCK_HEADER` 中的 `SignaturesOffset` 表示。

在写入日志的时候，可以比较每个扇区结尾签名位置的数据是否和签名数组中的数据一致来判断写入是否成功。

#### 2.2.2 BLF文件

`BLF` 由六种不同的元数据块组成，每种元数据块只包含对应类型的数据，其中 `shadow block` 中保存的是对应元数据块的备份，同样使用 `dump count` 参数说明数据块的新旧。

- `Control Block`：包含了有关布局(`layout`)、扩展(`extend`)区域以及截断(`truncate`)区域的信息
- `Base Block`：包含了符号表信息，其中包括该`BLF`有关的客户端、容器和安全上下文信息
- `Truncate Block`：包含了因为截断操作而需要对扇区进行更改的客户端信息，以及具体更改的扇区字节
- `Control Block Shadow`
- `Base Block Shadow`
- `Truncate Block Shadow`

这里主要关注其中的 `Control Block`，这个日志块中的记录遵循以下数据结构：

```
typedef struct _CLFS_CONTROL_RECORD
{
CLFS_METADATA_RECORD_HEADER hdrControlRecord;
ULONGLONG ullMagicValue;
ULONG Version;
CLFS_EXTEND_STATE eExtendState;
USHORT iExtendBlock;
USHORT iFlushBlock;
ULONG cNewBlockSectors;
ULONG cExtendStartSectors;
ULONG cExtendSectors;
CLFS_TRUNCATE_CONTEXT cxTruncate;
ULONG cBlocks;
ULONG cReserved;
CLFS_METADATA_BLOCK rgBlocks[6];
} CLFS_CONTROL_RECORD;
```

其中的 `cBlocks` 表示整个文件中包含的日志块数量：

![image1.png](https://bbs.kanxue.com/./upload/attach/202207/959053_4T3VEPAKN3YZWU6.png)

`rgBlocks` 中保存了每个日志块的大小信息：

```
typedef struct _CLFS_METADATA_BLOCK
{
ULONGLONG pbImage; // 指向内存中数据的指针，在BLF文件中为0
ULONG cbImage; // 日志块大小
ULONG cbOffset; // 偏移
CLFS_METADATA_BLOCK_TYPE eBlockType;
ULONG Padding;
} CLFS_METADATA_BLOCK;
```

## 3. 补丁对比

对比 `clfy.sys` 文件发现只有 `CClfsBaseFilePersisted::ReadImage` 函数存在不同，仔细检查代码，发现修改主要位于 `cBlocks > 6` 的情况：

![image2.png](https://bbs.kanxue.com/./upload/attach/202207/959053_TQAJHXHMJ7FWAPW.png)

修复之后，如果 `BLF` 文件中 `cBlocks` 数值大于 `6`，`ReadImage` 函数会直接出错返回。

## 4. 漏洞调试分析

### 4.1 漏洞原理

根据补丁分析结果，我们生成一个 `BLF` 文件，并手动将 `cBlocks` 修改成一个较大值 `0x19`（后期调试发现大于等于 `0x20` 无法通过检查），加载 `BLF` 文件后，系统崩溃（需要多次测试）。

```
kd> g
KDTARGET: Refreshing KD connection
KDTARGET: Refreshing KD connection
*** Fatal System Error:
0x00000050 (0xFFFF800C3905118E,0x0000000000000000,0xFFFFF8006B2A8140,0x0000000000000000)
Driver at fault:
***
CLFS.SYS - Address FFFFF8006B2A8140 base at FFFFF8006B2A0000, DateStamp 0c6e6b39
.Break instruction exception - code 80000003 (first chance)
A fatal system error has occurred.
Debugger entered on first try; Bugcheck callbacks have not been invoked.
A fatal system error has occurred.
nt!DbgBreakPointWithStatus:
fffff800*6b804c70 cc int 3
```

查看此时的函数调用栈：
