
# 解析 NTFS（二）DBR\MFT 部分

> 原文：[`www.pediy.com/kssd/pediy11/112568.html`](https://www.pediy.com/kssd/pediy11/112568.html)

前面解析 MBR，得到每个分区的起始扇区号，以及每个分区的大小；至于该分区的详细信息，则取决于该分区卷加载的文件系统。比如：簇大小，扇区大小，文件系统类型，该分区使用情况，介质描述，隐含扇区，扇区总数……

解析完 MBR，跳转到引导分区的第一个扇区，即 DBR；然后从 DBR 中的 BPB 中得到改卷的基本参数信息，定位 MFT 起始簇，进而定位到文件系统元数据文件位置，对该分区上的所有文件和目录进行遍历解析。

下面分步说明：

DBR 的作用

简单的来说，计算机启动时执行完 BOIS 的启动代码，检查各硬件设备正常后，JMP 到 MBR 的引导代码进行执行；然后由 MBR 引导至活动分区的 DBR，再由 DBR 引导操作系统。如：DBR 调用 `NTLDR`，再由 `NTLDR` 调用系统内核。

一．FAT32 DBR 和 NTFS $BOOT 的统一

每个分区第 0 号扇区，也就是系统引导扇区 DBR（DOS BOOT RECORDER）。不管是 FAT32 文件系统，还是 NTFS 文件系统，第一个扇区内容都是 DBR。很多做文件系统过滤驱动的开发人员强调，如果是 FAT32 系统，那么分区开始的第一个扇区是 DBR；如果是 NTFS 系统，那么分区开始的第一个扇区是 `$BOOT` 文件。其实，这个说法是基于文件系统特征的，虽然没有错，但也混淆了视听。事实上，在 NTFS 系统上，`$BOOT` 位于分区开始的 2 个簇上，也就是 16 个扇区，8K 大小（假设每簇大小占 8 扇区）。在这 16 个扇区上，第一个扇区仍然是 DBR。所以，从分区卷扇区级上来说，NTFS 分区的第一个扇区仍然是 DBR。这样就统一了说法，也便于理解了~

二．读取 DBR，进行解析：

为什么要读 DBR 呢，因为 DBR 存放着关于文件系统的重要参数信息以及系统引导代码。读 DBR 扇区的数据，并进行解析；由前面的分析，至少有两种方法可以定位到 DBR 的位置。

关于解析 DBR，声明两个重载的函数：

`void AnalyseDBR(ULONG ulSector);` //参数为扇区号

`void AnalyseDBR(wchar_t* szVolumeLink);` //参数为分区符号链接

A. 解析 MBR，根据分区表的信息，定位到指定分区的起始扇区，读取该扇区，按照 DBR 格式进行解析，上一节已经解析了 MBR，分区信息存储在链表结构里，直接定位到第一个分区，读取 DBR，伪代码如下：

```
// 
//  定位到第一个卷,即 C 盘
// 
{
  `NextEntry` = `RemoveHeadList(&NtfsData.Partitions)`;
  `PtItem` = (`PPARTITION_ITEM`)`CONTAINING_RECORD(NextEntry,PARTITION_ITEM,ForPtChain.ListEntry)`;
  `cout`<<"\\nthe test volume's start sector:"<<`PtItem->StartSector`<<`endl`;
  `ulDbrSector` = `PtItem->StartSector`;
}
// 利用解析 MBR 的分区表信息，定位到分区开始扇区，解析 DBR
`AnalyseDBR(ulDbrSector)`;
```

详细代码见 `toysNtfs` 工程。

B. 根据分区符号链接，如：`_T("\\\\.\\C:")`，直接打开分区，进行读写；相比解析 MBR 的方式，这种方法明显直接而简单，打开分区以后，第一个扇区就是 DBR。解析 MBR 虽然麻烦，但是灵活；将整个磁盘与分区都联系起来，更容易理解磁盘结构；甚至经过深入学习，自己开发一个小型的磁盘分区工具，划分一小块空间，用来存储自己的秘密文件，该卷的解析方式只有你自己知道（有意不让 windows 识别），岂不快哉~

伪代码如下：（假设打开 C：盘）

```
// 利用分区符号链接，打开分区，进行解析
`AnalyseDBR(_T("\\\\.\\C:"))`;
```

前后两种方法解析的结果如下：

![](img/2cd8882b1e691c2f50b0bc6dc81fc6ed.jpg)

（相同的结果）

C. DBR 在磁盘上的格式和各字段的含义，在 FAT 文件系统上的 DBR 和 NTFS 上的 DBR 有所不同，下面以 NTFS 为例：

大概分以下几个部分：

跳转指令；

文件系统标志, Oem；

BPB，25 字节；

其后类似于扩展 BPB 的一些字段；

大小为 `[0x200-0x044]` 的引导代码；

结束标志：55AA

用代码结构表示如下：同样，只记录需要关心的若干字段，便于解析 NTFS 其他部分。

```
typedef struct _PACKED_BOOT_SECTOR {
  UCHAR Jump[3];
  UCHAR Oem[8];
  BIOS_PARAMETER_BLOCK PackedBpb;    //BPB
  UCHAR Unused[4];
  LONGLONG NumberSectors;   //扇区总数
  LCN MftStartLcn;    //MFT 开始簇
  LCN Mft2StartLcn;
  CHAR ClustersPerFileRecordSegment;
  UCHAR Reserved0[3];
  CHAR DefaultClustersPerIndexAllocationBuffer;
  UCHAR Reserved1[3];
  LONGLONG SerialNumber;
  ULONG Checksum;
  UCHAR BootStrap[0x200-0x044];    //引导代码
} PACKED_BOOT_SECTOR;
```

`typedef PACKED_BOOT_SECTOR *PPACKED_BOOT_SECTOR;`

DBR 中的 BPB 结构如下：

```
typedef struct BIOS_PARAMETER_BLOCK {
  USHORT BytesPerSector;
  UCHAR  SectorsPerCluster;
  USHORT ReservedSectors;
  UCHAR  Fats;
  USHORT RootEntries;
  USHORT Sectors;
  UCHAR  Media;
  USHORT SectorsPerFat;
  USHORT SectorsPerTrack;
  USHORT Heads;
  ULONG  HiddenSectors;
  ULONG  LargeSectors;
} BIOS_PARAMETER_BLOCK;
```

`typedef BIOS_PARAMETER_BLOCK *PBIOS_PARAMETER_BLOCK;`

大概能见名知意。`BPB(bois parameter block)` 是一段描述能够使可执行引导代码找到相关参数的信息。主要描述磁盘结构的细节，包括每扇区的字节数 `BytesPerSector`，每簇的扇区数 `SectorsPerCluster`；磁盘介质；每磁道扇区数，磁头数等，均为后续引导代码提供参数。

关于文件系统的信息，其实在扩展 BPB 中，也就是 BPB 之后的若干字段。这些字段中的数据使得 `NTLDR` 能够在启动时找到主文件表 `$MFT`，进而定位到其他数据文件。

至此，DBR 解析完毕，得到该卷上的 MFT 起始簇，备份 MFT 起始簇，每簇扇区数，每扇区字节数，介质等信息。经过简单的计算就能知道起始 MFT 的扇区号。

三．解析 MFT

由于用扇区来描述磁盘空间粒度比较小，处理量比较大，所以引进簇的概念。簇一般是 2 的倍数，一般 NTFS 中一个簇占 8 个连续扇区。基本上所有文件系统用簇来描述分区。

`LCN`（logical cluster number）逻辑簇号，对卷的第 1 个簇到最后一个簇进行编号。只要知道 LCN 号和簇的大小以及 NTFS 卷在物理磁盘中的绝对扇区就可以对簇进行扇区定位。

`VCN`（virtual cluster number）虚拟簇号，VCN 对特定文件的簇从头到尾进行编号，表示文件内部的相对位置，方便系统对文件中的数据进行引用。假如一个文件占用 m 个簇，那么这些簇的 VCN 就是从 0 到(m-1)。

NTFS 文件系统用文件来记录所有信息；文件由属性组成；所以对 NTFS 文件的读写，实际上是对 NTFS 属性的读写。一个文件对应一条 MFT 记录（即文件记录块），一个文件记录块固定大小为 1K，占两个扇区；所有文件的 MFT 记录都有一个文件来管理，就是 `$MFT` ,也就是 0 号 MFT 记录所指向的文件。

由前面解析 DBR 知道 MFT 起始扇区，那么读取该扇区内容，就是 `$MFT` 文件的文件记录了。

解析之前，先说说元文件，对文件系统有一个全局的认识。

NTFS 文件系统由 DBR（本身也是 `$BOOT` 的内容）和元文件组成，其他文件和目录通过文件系统管理。

元数据文件记录了一些非常重要的文件系统数据，包括用于文件定位和恢复的数据结构、引导程序数据以及整个卷的分配位图等信息。这些数据被 NTFS 文件系统理解为文件，并以文件的方式进行管理，统一了思想。

元文件对于用户是不能直接访问的，MFT 将开头的 16 个文件记录块保留用于这些元数据文件，除此之外的文件记录块才用于普通的用户文件和目录。

文件系统 16 个元文件列表：

0 `$MFT` //mft 本身
1 `$MftMirr` //mft 元数据文件的镜像，用于备份恢复
2 `$LogFile` //文件操作历史记录文件
3 `$Volume` //文件卷信息文件
4 `$AttrDef` //属性定义文件
5 `$Root(\\)` //根目录文件

其它的元文件参照其他资料，不例举了。

MFT 结构

每个文件都有一条 MFT 记录，通过读取该文件的 MFT 记录，就可以知道该文件有哪些属性，各属性的内容是多少，是否有加密，是否有隐藏存档属性，是否有命名的数据流等。

MFT 文件记录由记录头和属性部分组成；一个文件可以有多个属性，这些属性又分为属性头和属性内容。这些属性的标准信息由 `$AttrDef` 元文件定义。

属性又分为常驻属性(Resident Attr)和非常驻属性(NonResident Attr)。当文件的属性内容大于大约 700 字节时，就要使用非常驻属性。将属性内容外挂到磁盘的其他位置，这些位置由 runs 片来描述。

不同属性相同的地方在于属性头，即要么是常驻属性头，要么是非常驻属性头；不同属性的属性内容各不相同，在磁盘上的分布结构也不同，作用也不同；特别说明，这些属性是可以扩充的。对各个属性的解析，则比较繁琐。

那么解析 MFT 文件记录块的基本框架如下：

```
ReadSector( MFT 索引号, 长度)
{
  解析 MFT 记录头；（大小 0x38 字节）
  解析属性部分
  {
    Switch(属性类型)
    {
      Case:  对不同的属性做处处理，读取或修改,break;
      Default:
    }
  }
}
```

指定一个 MFT，解析属性列表：

![](img/e9116d984ca926db287c6ff9ffc81f6c.jpg)

关于 MFT 的详细结构，请参考文献资料，解析过程见代码。

无法说的太详细，因为 NTFS 内容很多，结构却一点也不复杂；展开全部细节，却又不是只言片语就能说完的，亦属力所不能及；说的太概念化，又恐读之无物。那，权当抛砖引玉吧。

程序代码写的只是一个框架，在这个框架上就可以填充各个属性的详细分析过程。这样的解析未免不太科学，但是作为学习了解文件系统在磁盘上的底层结构，却是很简单有效的。如果要理解 NTFS 文件系统工作的原理，须跟踪文件系统 IO 处理流程，了解 IO 管理器，磁盘驱动，缓存 cache 管理等相关知识。

顺着这个路径，熟悉文件的各个属性，属性之间的相互作用；用 `nt4.Src.cntfs` 代码，参考部分解析过程。

PS:程序代码只是为了学习方便而写的简单测试框架；本节主要是分析 DBR，获取 MFT 的起始簇，获取分区卷的必要参数信息；对指定的一条 MFT 文件记录进行定位和解析；如此就可以从 MFT 中读取文件的所有内容，包括：文件只读、隐藏、存档属性，创建时间，修改时间，对象 ID，数据内容，命名数据流，重解析点等信息。

那么，遗留的问题是，如何获得 MFT 索引号？MFT 号其实是文件引用号（64 位）的低 48 位，那么如何获取文件引用号？这里方法就不一了，比如：递归解析 `$Root` 跟目录文件，获取每个索引项的文件引用号，取低 48 位作为 MFT 索引号去 MFT 表中读取文件数据；第二种方法是，遍历 MFT 在磁盘上分布的区域大小，计算出总的 MFT 记录数量，然后发送控制码 `FSCTL_GET_NTFS_FILE_RECORD` 来获取文件引用号。

关于解析：首先要明确目的，才能在解析过程中对关心的文件属性信息作特殊处理；如果作为学习，那么选择一个清晰明了的框架还是比较合理的。等熟悉了基本结构，在去深入学习 NTFS 驱动源码，调试，在实际应用中开发项目，合理应用，使产品更高效；举个例子，NTFS 文件系统中，遍历磁盘上的所有文件，并获取文件在磁盘上的数据存储信息。如果是普通的递归遍历法，即 `FindFirstFile`,`FindNextFile`，那么遍历速度将比直接遍历 MFT 慢 25 倍左右。

下节具体解析文件的若干属性（数据属性，`$IndexRoot` 根目录属性）；给定文件名通过 `$ROOT` 文件查找文件引用号。

参考资料：

《数据恢复技术》 戴士剑、涂彦晖；

《数据重现，文件系统原理精解》 马林

《windows internal》，4th,Mark E.Russinovich

《windows 内核情景分析》毛德操

相关代码：

`ToysNtfs2.rar`
