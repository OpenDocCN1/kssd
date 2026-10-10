

# 机器狗 0625 技术剖析（驱动读写磁盘扇区）

> 原文：[`www.pediy.com/kssd/pediy10/67321.html`](https://www.pediy.com/kssd/pediy10/67321.html)

由于对磁盘结构不是很熟，仅仅从逆向分析的角度，通过对这个驱动的分析过程，学习了向 SCSI 端口向扇区写数据的方法。如有错误或纰漏之处，请大家指出。  
  
附件：  
1、机器狗 0625 技术剖析.pdf  
2、obj2(Macdog).idb  
  
摘要  
这是一个 6 月 23 号捕获的机器狗变种。跟之前不同的地方，主要是将以前 ring3 下写磁盘替换文件的操作使用驱动实现了，本文的主要部分也是对该部分的解析。可以看出，机器狗的作者对内核驱动以及磁盘结构都很熟悉。  
  
三、驱动分析  
1、基本信息  
文件名：%windir%\\system32\\drivers\\obj2.sys  
DeviceName：\\\\Device\\\\DogBaby  
SymbolicLinkName：\\\\DosDevices\\\\DogBaby  
  
2、摘除 ntfs 的 AttachedDevice  
  
3、恢复 disk 的 Hook  
1)、摘除 disk 的 AttachedDevice  
2)、根据\_DRIVER\_OBJECT.DriverSection 获得\_LDR\_DATA\_TABLE\_ENTRY 的链表，枚举该链表查找名为 CLASSPNP.SYS 的模块，找到则获取该模块的基址 Base 和 EntryPoint，如果 EntryPoint - Base 等于 0AE8Fh(入口的 RVA)，则取 g\_pClassInternalIoControl = Base + 4FC3h。然后注册驱动 disk 的派遣例程为\_DRIVER\_OBJECT.IrpInternalDeviceControlDispatch = pClassInternalIoControl。  
  
4、摘除 FtDisk 的 AttachedDevice  
  
5、摘除 Atapi 的 AttachedDevice  
  
6、恢复 Atapi 的 Hook  
1)、读取 atapi.sys 文件，并定位 pe 头、及块表。  
2)、定位到 init 块，从 init 块中使用模糊匹配特征 C7 ?? 30 ?? ?? ?? ?? C7 ?? 34。  
3)、找到的地址为 24968h，记为 Atapi\_24968h。  
4)、取 Atapi\_24968h + 11h 的位置，并换算成 RVA，得到 fnRVA\_IdePortDispatch。  
5)、取 Atapi\_24968h + 18h 的位置，并换算成 RVA，得到 fnRVA\_IdePortDispatchDeviceControl。  
6)、如果上面的方法获取失败，则强制转换：  
fnRVA\_IdePortDispatch = 67B4h，  
fnRVA\_IdePortDispatchDeviceControl = 0A592h。  
7)、获取这两个函数的 SVA  
fnIdePortDispatch = fnRVA\_IdePortDispatch + DriverStart，  
fnIdePortDispatchDeviceControl = fnRVA\_IdePortDispatchDeviceControl + DriverStart。  
8)、注册 Atapi 的派遣例程  
\_DRIVER\_OBJECT.IrpDeviceControlDispatch =   
fnIdePortDispatchDeviceControl，  
\_DRIVER\_OBJECT.IrpInternalDeviceControlDispatch = fnIdePortDispatch。  
  
7、定位被覆盖文件的逻辑簇号  
1)、打开文件 conime.exe，失败则打开 userinit.bat；并获取 FileObject。  
2)、根据 FileObject 获取\_FILE\_OBJECT.Vpb，再获取\_VPB.DeviceObject。  
3)、创建一个 IRP。设 MainFunction 为 IRP\_MJ\_FILE\_SYSTEM\_CONTROL。  
4)、设定\_IO\_STACK\_LOCATION.FsControlCode =   
FSCTL\_GET\_RETRIEVAL\_POINTERS  
//其它 IRP 初始化略过，详细请看 idb 文件。  
5)、调用自己实现的 IOCallDriver，获取指定文件的 RETRIEVAL\_POINTERS\_BUFFER 信息。  
结构查阅 msdn 如下：  
typedef struct RETRIEVAL\_POINTERS\_BUFFER {  
  DWORD ExtentCount;  
  LARGE\_INTEGER StartingVcn;  
  struct {  
    LARGE\_INTEGER NextVcn;  
    LARGE\_INTEGER Lcn;  
  } Extents\[1\];   
} RETRIEVAL\_POINTERS\_BUFFER,   
\*PRETRIEVAL\_POINTERS\_BUFFER;  
要注意的是按 8 个字节对齐的，所以 DWORD ExtentCount 后、LARGE\_INTEGER StartingVcn 前有 4 个字节的空间。  
ExtentCount 记录了该文件分成了 n 块存储。  
StartingVcn 记录了起始虚拟簇号。  
Extents\[i\].NextVcn 记录了下一个块的起始虚拟簇号，所以 Extents\[i\].NextVcn - Extents\[i - 1\].NextVcn 表示 i 块所占的簇数。  
Extents\[i\].Lcn 记录了第 i + 1 块的起始逻辑簇号。注意 i 是从 0 开始的。  
本机器狗使用了不严格的计算方法，仅用到了该结构中的 Extents\[0\].Lcn，即文件的起始逻辑簇号，并没有考虑分多块存储的情况。因此，理论上是存在破坏系统文件的风险的。这也是病毒不负责任的地方。  
  
8、MyRead\_WriteSector  
从这个函数名可以看出这是一个读写扇区的例程。前缀 My 表示该函数是由我标识的，用来跟 ida 自动识别的函数进行区别，以便查找。这个函数很重要，整个驱动中会调用三次，所以先解析。  
MyRead\_WriteSector(DeviceObj, IRP\_ID, pBuffer, dwSectorLowPos, SecNum);  
DeviceObj：操作设备对象，这里为 Disk。  
IRP\_ID：3 表示读取，4 表示写入。  
pBuffer：写入设备的数据地址，或读取数据的内存辞职。  
dwSectorLowPos：读取或写入的扇区号。其实扇区号为 LARGE\_INTEGER 类型的大数，但这里仅使用了低 32 位，没有使用高 32 位。  
SecNum：读取或写入的扇区数。  
下面为函数的实现介绍。  
1)、通过使用 MyIoCallDriver(病毒自己实现的 IoCallDriver)来向 Disk 设备发送服务号为 IRP\_MJ\_INTERNAL\_DEVICE\_CONTRO，通过向 SCSI 端口读写数据实现的。  
2)、填充 IRP  
mov     \[edi+\_IRP.MdlAddress\], eax; eax 为 pBuffer 对应的 Mdl  
......    ;省略部分代码  
mov     \[ebp+UserIosb\], ebx  ; ebx == 0  
lea     eax, \[ebp+UserIosb\]  
mov     \[edi+\_IRP.UserIosb\], eax  
lea     eax, \[ebp+Event\]  
mov     \[edi+\_IRP.UserEvent\], eax  
mov     \[edi+1Ch\], ebx  ; \_IRP.Information = 0  
mov     \[edi+18h\], ebx  ; \_IRP.\_IO\_STATUS\_BLOCK = 0  
or      \[edi+\_IRP.Flags\], 5 ; IRP\_SYNCHRONOUS\_API | IRP\_NOCACHE  
mov     \[edi+0Ch\], ebx  ; \_IRP.IrpCount = 0  
mov     \[edi+24h\], bl   ; \_IRP.Cancel = 0  
mov     \[edi+38h\], ebx  ; \_IRP.CancelRoutine = 0  
mov     \[edi+20h\], bl   ; \_IRP.RequestorMode = 0  
call    KeGetCurrentThread  
mov     \[edi+50h\], eax  ; \_IRP.Thread  
mov     eax, \[edi+60h\]  ; \_IRP.CurrentStackLocation  
sub     eax, 24h        ; IoGetNextStackLocation  
mov     byte ptr \[eax\], IRP\_MJ\_INTERNAL\_DEVICE\_CONTROL  
mov     \[eax+4\], esi    ; \_IO\_STACK\_LOCATION.scsi.\_SCSI\_REQUEST\_BLOCK  
mov     ecx, \[ebp+DiskDeviceObj\]  
mov     \[eax+14h\], ecx  ; \_IO\_STACK\_LOCATION.DeviceObject = DiskDeviceObj  
mov     eax, \[edi+60h\]  
sub     eax, 24h  
mov     dword ptr \[eax+1Ch\], offset fnCompletionRoutine ; \_IO\_STACK\_LOCATION.CompletionRoutine  
mov     \[eax+20h\], esi  ; \_IO\_STACK\_LOCATION.Context  
mov     \[eax+\_IO\_STACK\_LOCATION.Control\], 0E0h  
从填充的 IRP 可以看到一个重要的结构\_SCSI\_REQUEST\_BLOCK。  
3)、\_SCSI\_REQUEST\_BLOCK  
关于结构的定义可以找相关资料。用来存放向 SCSI 端口写数据的一些信息。这里看一下对这个 srb 的填充。  
mov     word ptr \[esi\], 40h ; \_SCSI\_REQUEST\_BLOCK.Length = 40h  
mov     \[esi+2\], bl     ; \_SCSI\_REQUEST\_BLOCK.Function = 0  
mov     eax, \[ebp+pBuffer\]  ;pBuffer 指向传入的参数，即存放数据的地址。  
mov     \[esi+18h\], eax  ; \_SCSI\_REQUEST\_BLOCK.DataBuffer = pBUffer  
movzx   eax, \[ebp+SecNum\]  
shl     eax, 9          ; eax = SecNum \* 512  
mov     \[esi+10h\], eax  ; \_SCSI\_REQUEST\_BLOCK.DataTransferLength = SecNum \* 512 //即写入数据的大小  
mov     byte ptr \[esi+9\], 20h ; \_SCSI\_REQUEST\_BLOCK.QueueAction = 20h  
mov     \[esi+3\], bl     ; \_SCSI\_REQUEST\_BLOCK.SrbStatus = 0  
mov     \[esi+4\], bl     ; \_SCSI\_REQUEST\_BLOCK.ScsiStatus = 0  
mov     \[esi+20h\], ebx  ; \_SCSI\_REQUEST\_BLOCK.NextSrb = NULL  
mov     \[esi+1Ch\], ecx  ; \_SCSI\_REQUEST\_BLOCK.SenseInfoBuffer = pSenseInfoBuffer  
mov     byte ptr \[esi+0Bh\], 12h ; \_SCSI\_REQUEST\_BLOCK.SenseInfoBufferLength = 12h  
xor     eax, eax  
cmp     \[ebp+IRP\_ID\], IRP\_MJ\_READ ; if(IRP\_ID != IRP\_MJ\_READ)  
setnz   al              ; {al = 1;}  
dec     eax             ; eax --;  
and     eax, 0FFFFFFC0h ; eax &= 0x0ffffffc0;  
add     eax, 80h        ; eax += 80h;  
; //if IRP\_ID == IRP\_MJ\_READ eax == 40h SRB\_FLAGS\_DATA\_IN  
; //else eax == 80h SRB\_FLAGS\_DATA\_OUT  
mov     \[esi+0Ch\], eax  ; \_SCSI\_REQUEST\_BLOCK.SrbFlags = 80h  
cmp     \[ebp+IRP\_ID\], IRP\_MJ\_READ  
jnz     short loc\_11997  
or      eax, 200h SRB\_FLAGS\_ADAPTER\_CACHE\_ENABLE  
mov     \[esi+0Ch\], eax  ; \_SCSI\_REQUEST\_BLOCK.SrbFlags = 240h  
loc\_11997:   
or      dword ptr \[esi+0Ch\], 20h ; \_SCSI\_REQUEST\_BLOCK.SrbFlags = A0h(If Read 260h)  
; 20h == SRB\_FLAGS\_DISABLE\_AUTOSENSE  
mov     eax, \[esi+10h\]  ; eax = \_SCSI\_REQUEST\_BLOCK.DataTransferLength  
shr     eax, 0Ah  
inc     eax  
mov     \[esi+14h\], eax  ; \_SCSI\_REQUEST\_BLOCK.TimeOutValue = \_SCSI\_REQUEST\_BLOCK.DataTransferLength >> 0Ah + 1  
mov     eax, \[ebp+dwSectorLowPos\]  
mov     \[esi+2Ch\], eax  ; \_SCSI\_REQUEST\_BLOCK.QueueSortKey = dwSectorLowPos  
mov     byte ptr \[esi+0Ah\], 0Ah ; \_SCSI\_REQUEST\_BLOCK.CdbLength = 0Ah  
填充这个结构读数据和写数据的 SrbFlags 分别设为 SRB\_FLAGS\_DATA\_IN | SRB\_FLAGS\_ADAPTER\_CACHE\_ENABLE |   
SRB\_FLAGS\_DISABLE\_AUTOSENSE 和 SRB\_FLAGS\_DATA\_OUT | SRB\_FLAGS\_DISABLE\_AUTOSENSE。  
4)、\_CDB  
在\_SCSI\_REQUEST\_BLOCK 结构最后是 UCHAR Cdb\[16\]; 但是 Cdb 不一定是 16 个字节，长度由\_SCSI\_REQUEST\_BLOCK.CdbLength 决定，从上面可以看出这里是 0Ah 个字节。该结构是一个联合体，这里使用的是\_CDB10 结构。  
mov     byte ptr \[esi+0Ah\], 0Ah ; \_SCSI\_REQUEST\_BLOCK.CdbLength = 0Ah  
mov     cl, \[ebp+IRP\_ID\]  
add     cl, 11h  
shl     cl, 1  
mov     \[esi+30h\], cl   ; \_CDB\[0\] = 28h  
mov     cl, \[esi+31h\]  
and     cl, 1Fh  
or      cl, 80h  
mov     \[esi+31h\], cl   ; \_CDB\[1\] = 80h  
mov     ecx, eax  
shr     ecx, 18h  
mov     \[esi+32h\], cl   ; if Read \_CDB\[2\] = 0  
mov     ecx, eax  
shr     ecx, 10h  
mov     \[esi+33h\], cl   ; if Read \_CDB\[3\] = 0  
mov     ecx, eax  
shr     ecx, 8  
mov     \[esi+34h\], cl   ; if Read \_CDB\[4\] = 0  
mov     \[esi+35h\], al   ; if Read \_CDB\[5\] = 0  
mov     ax, \[ebp+SecNum\]  
mov     \[esi+37h\], ah   ; if Read \_CDB\[7\] = 0  
cmp     ax, bx  
setnz   al  
mov     \[esi+38h\], al   ; if Read \_CDB\[8\] = 1  
从 scsi.h 中找到如下：  
// 10-byte commands  
#define SCSIOP\_READ\_FORMATTED\_CAPACITY 0x23  
#define SCSIOP\_READ\_CAPACITY       0x25  
#define SCSIOP\_READ                0x28  
#define SCSIOP\_WRITE               0x2A  
可以看出，\_CDB\[0\]为 OperationCode，28h 表示 SCSIOP\_READ，2Ah 表示 SCSIOP\_WRITE。\_CDB\[1\]按位描述了一些属性，可以参考 scsi.h，这里将这个字节设为 80h。\_CDB\[2\]到\_CDB\[5\]描述的是扇区位置，依次为 dwSectorLowPos 从高字节到低字节的值。\_CDB\[7\]和\_CDB\[8\]描述了要写书的扇区数。如果小于 0，则设为 1。  
由于本人的语言组织能力较弱，写得比较乱，详细的请看 idb 文件。这个函数将会完成向指定扇区读写数据。  
  
9、读取主分区表  
调用 My\_Read\_WriteSector(DiskDeviceObj, 3, pBuffer, 0, 1)来获取磁盘 0 号扇区的数据，大小为 1 个扇区的大小，即 512 字节。  
0 号扇区保存了主分区表，结构为：  
typedef struct \_MBR\_SECTOR  
{  
   UCHAR             BootCode\[446\];  
   PARTITION\_ENTRY   Partition\[4\];  
   USHORT           Signature;  
} MBR\_SECTOR, \*PMBR\_SECTOR;  
这里主要用到了 PARTITION\_ENTRY 结构：  
typedef struct \_PARTITION\_ENTRY  
{  
   UCHAR active;                 // 能否启动标志  
   UCHAR StartHead;               // 该分区起始磁头号  
   UCHAR StartSector;             // 起始柱面号高 2 位：6 位起始扇区号  
   UCHAR StartCylinder;           // 起始柱面号低 8 位  
   UCHAR PartitionType;           // 分区类型  
   UCHAR EndHead;                 // 该分区终止磁头号  
   UCHAR EndSector;             // 终止柱面号高 2 位：6 位终止扇区号  
   UCHAR EndCylinder;             // 终止柱面号低 8 位  
   ULONG StartLBA;               // 起始扇区号  
   ULONG TotalSector;             // 分区尺寸（总扇区数）  
} PARTITION\_ENTRY, \*PPARTITION\_ENTRY;  
其中 active 为 80h，表示该分区能够启动系统。  
PartitionType 表示分区类型，如 NTFS、FAT32 等等，如果为 FF 表示该分区为扩展分区，扩展分区会再有分区表，划分为逻辑分区。如果你有兴趣，可以查阅其它资料。  
本处依次判断 Partition\[0\]到 Partition\[3\]是否为启动分区，如果是，则计算 StartLBA，这里比较有意思的是 StartLBA\[n\] = StartLBA\[0\] + ... + StartLBA\[n\]，这个我是比较奇怪的，从《数据恢复技术》一书中，我的理解是，如果为基本分区，StartLBA 直接是分区的起始逻辑扇区，如果是扩展分区，则需要加上的保留扇区数。这里是让我感到不解的，如果自己分析，要自己测试多个系统、多种分区格式的分区表，遂放弃。希望知道的兄弟能够指点迷津，抑或是机器狗的一个 bug。但对于大多数系统，Partition\[0\]即是启动分区，Partition\[0\].StartLBA 也就是该分区的起始扇区位置。  
判断 Partition\[0\].PartitionType 如果不是 PARTITION\_TYPE\_FAT32、PARTITION\_TYPE\_FAT32\_LBA 或 PARTITION\_TYPE\_NTFS，则返回失败。  
  
10、读取启动分区的第一个扇区  
定位到启动分区后，调用 My\_Read\_WriteSector(DiskDeviceObj, 3, pBuffer, StartLBA, 1)来读取启动分区的第一个扇区的内容，即分区信息。结构如下：  
typedef struct \_BBR\_SECTOR  
{  
   USHORT JmpCode;               // 2 字节跳转指令,跳转到引导代码  
   UCHAR   NopCode;               // 1 字节 nop 指令,填充用,保证跳转指令长 3 个字节  
   UCHAR   OEMName\[8\];             // 8 字节的 OEMName  
   // 下面开始为: BPB( BIOS Parameter Block )  
   USHORT BytesPerSector;         // 每个扇区的字节数 (512 1024 2048 4096)  
   UCHAR   SectorsPerCluster;     // 每个簇的扇区数 ( 1 2 4 8 16 32 64 128 )两者相乘不能超过 32K(簇最大大小)  
   USHORT ReservedSectors;       // 从卷的第一个扇区开始的保留扇区数目,该值不能为 0，对于 FAT12/FAT16，该值通常为 1,对于 FAT32，典型值为 32  
   UCHAR   NumberOfFATs;           // 卷上 FAT 数据结构的数目，该值通常应为 2,\[NTFS 不使用 NumberOfFATs 字段，必须为 0\]  
   USHORT RootEntries;           // 对于 FAT12/FAT16,该值表示 32 字节目录项的数目,对于 FAT32，该值必须为 0；\[NTFS 不使用\]  
   USHORT NumberOfSectors16;     // 该卷上的扇区总数，该字段可以为 0，如果该字段为 0，则 NumberOfSectors32 不能为 0；对于 FAT32，该字段必须为 0 \[FAT32/NTFS 不使用该字段\]  
   UCHAR   MediaDescriptor;       // 介质类型  
   USHORT SectorsPerFAT16;       // 该字段标识一个 FAT 结构占有的扇区数（FAT12/FAT16）,对于 FAT32 卷，该字段必须为 0；\[FAT32/NTFS 不使用该字段\]  
   USHORT SectorsPerTrack;       // 用于 INT 0x13 中断的每个磁道的扇区数  
   USHORT HeadsPerCylinder;       // 用于 INT 0x13 中断的每个柱面的磁头数  
   ULONG   HiddenSectors;         // 包含该 FAT 卷的分区之前的隐藏扇区数  
   ULONG   NumberOfSectors32;     // 该字段包含该卷上的所有扇区数目，对于 FAT32，该字段不为 0；FAT12/FAT16 可根据实际大小是否超过 65536 个扇区数决定是否采用该字段； \[NTFS 不使用该字段\]  
   // 下面开始为: EBPB ( Extended BIOS Parameter Block )  
   ULONG   SectorsPerFAT32;       // 对于 FAT32，该字段包含一个 FAT 的大小，而 SectorsPerFAT16 字段必须为 0;  
} BBR\_SECTOR, \*PBBR\_SECTOR;  
  
取得分区信息之后，获取 SectorsPerCluster 和 ReservedSectors。  
判断 Partition\[0\].PartitionType 是否是 PARTITION\_TYPE\_FAT32 或 PARTITION\_TYPE\_FAT32\_LBA，如果是，则获取 SectorsPerFAT32 和 NumberOfFATs。  
  
11、计算文件对应的扇区位置  
Lcn\_HightPart 和 Lcn\_LowPart 由第 7 步获得。  
1)、如果是 FAT32 分区，dwSectorLowPos = StartLBA + ReservedSectors + SectorsPerFAT32 \* NumberOfFATs + SectorsPerCluster \* Lcn\_LowPart；  
FileSectorHighPos = Lcn\_HightPart \* SectorsPerCluster；  
2)、如果是 NTFS 分区，dwSectorLowPos = StartLBA + ReservedSectors + SectorsPerCluster \* Lcn\_LowPart；  
FileSectorHighPos = Lcn\_HightPart \* SectorsPerCluster；  
FileSectorHighPos 在将数据写入扇区时，并没有使用。  
  
12、写入病毒  
是由 DeviceIoControl 的 800C004h 号控制命令触发的，写入文件的数据由 DeviceIoControl 传入。调用 My\_Read\_WriteSector(DiskDeviceObj, 4, pBuffer, dwSectorLowPos, dwFileSize / 512 + 1)来实现向扇区的写入。  
另外 DeviceIoControl 的 800C008h 号控制命令会摘掉 ntfs 的 AttachedDevice。  
  
参考资料  
1、《数据恢复技术》  
2、http://dev.csdn.net/article/78/78564.shtm  
3、http://hi.baidu.com/ptf\_phoenix/blog/item/c5db4f1f319f6ecca7866994.html  
4、winddk  
5、msdn  
还有一些网上的其它资料，不一一列出。  
  
  
分析者：水中雁   
日期：2008.06.25   
邮箱：tigerinhm@163.com   
主页：http://hi.baidu.com/wildgoose

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || 机器狗 0625 技术剖析.rar \[附件请到论坛下载：[`bbs.pediy.com/showthread.php?t=67321`](http://bbs.pediy.com/showthread.php?t=67321) \]   
 |