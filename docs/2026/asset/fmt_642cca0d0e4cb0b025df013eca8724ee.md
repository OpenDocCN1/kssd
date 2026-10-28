
# [原创]也谈rootkit 注册表信息隐藏

呵呵，今天是我女儿的生日，写篇文章，留作纪念。闲话少说，转入正题。

注册表是Windows系统存储关于计算机配置信息的数据库，包括了系统运行时需要调用的运行方式的设置，是系统的核心。操作系统是用特殊的文件 (`Hive`文件) 来存储注册表的内容，并提供给用户相关的编程接口(`APIs`)来进行注册表的操作，例如`Advapi32.dll`中的`Registry Functions`(如：`RegEnumKey`)。在Windows操作系统中，函数的调用有着极其规范的层次结构，如下图所示是系统实现 `RegEnumKey`函数的调用体系。

注册表隐藏就是将系统呈现给用户的注册表信息进行修改，使得用户或检测工具无法直观地发现事实上存在的注册表内容。目前的注册表隐藏，大都采用的是Hook技术。根据它们不同的运行环境和模式，把常见的注册表隐藏技术称为用户态下注册表隐藏技术和核心态下注册表隐藏技术。它们通常是利用`IAT Hook`、`Inline Hook`、远程线程注入、`SSDT Hook`等方式来实现。网上比较多见的核心态隐藏技术是`ssdt hook NtEnumerateKey`和`inline hook CmEnumerateKey`。然而这些方法已经很容易被防御体系发现。

本文是通过修改`hive`内存结构中的特殊指针`hook`实现注册表项的隐藏。关于`hive`文件结构，论坛中曾经有两人写过相关的文章，一篇是炉子大牛写的《[HIVE格式解析](http://bbs.pediy.com/showthread.php?t=59174)》，另一篇是HSQ大牛写的《[注册表监控弱点演示程序 v0.2 逆向ASM源码及相关资料](http://bbs.pediy.com/showthread.php?p=438407)》,另外，Petter Nordahl-Hagen写过一个`hive`文件存取库，我整理了下，也附在本篇文章之后，方便大家查阅。

相信看过网上这几篇的朋友一定还有些疑虑，到底注册表的工作原理是怎样的呢？我们今天通过一个驱动代码来揭开它神秘的面纱。

在磁盘上，注册表并不是简单的一个大文件，而是一组称为`hive`的单独文件。每个`hive`文件包含了一颗注册表树。有一个键作为该树的根。子键和他们的值存储在根的下面。当配置管理器（注册表的执行体子系统）加载`hive`的时候，会在`HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\hivelist`子键下的注册表值中记录下每个`hive`的路径。下面是我本机注册表`hivelist`子键导出的信息：

```
[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\hivelist]

"\\REGISTRY\\MACHINE\\HARDWARE"=""

"\\REGISTRY\\MACHINE\\SECURITY"="\\Device\\HarddiskVolume1\\WINDOWS\\system32\\config\\SECURITY"

"\\REGISTRY\\MACHINE\\SOFTWARE"="\\Device\\HarddiskVolume1\\WINDOWS\\system32\\config\\SOFTWARE"

"\\REGISTRY\\MACHINE\\SYSTEM"="\\Device\\HarddiskVolume1\\WINDOWS\\system32\\config\\SYSTEM"

"\\REGISTRY\\USER\\.DEFAULT"="\\Device\\HarddiskVolume1\\WINDOWS\\system32\\config\\DEFAULT"

"\\REGISTRY\\MACHINE\\SAM"="\\Device\\HarddiskVolume1\\WINDOWS\\system32\\config\\SAM"

"\\REGISTRY\\USER\\S-1-5-20"="\\Device\\HarddiskVolume1\\Documents and Settings\\NetworkService\\NTUSER.DAT"

"\\REGISTRY\\USER\\S-1-5-20_Classes"="\\Device\\HarddiskVolume1\\Documents and Settings\\NetworkService\\Local Settings\\Application Data\\Microsoft\\Windows\\UsrClass.dat"

"\\REGISTRY\\USER\\S-1-5-19"="\\Device\\HarddiskVolume1\\Documents and Settings\\LocalService\\NTUSER.DAT"

"\\REGISTRY\\USER\\S-1-5-19_Classes"="\\Device\\HarddiskVolume1\\Documents and Settings\\LocalService\\Local Settings\\Application Data\\Microsoft\\Windows\\UsrClass.dat"

"\\REGISTRY\\USER\\S-1-5-21-1550111154-316279633-4274931122-1006"="\\Device\\HarddiskVolume1\\Documents and Settings\\Jason\\NTUSER.DAT"

"\\REGISTRY\\USER\\S-1-5-21-1550111154-316279633-4274931122-1006_Classes"="\\Device\\HarddiskVolume1\\Documents and Settings\\Jason\\Local Settings\\Application Data\\Microsoft\\Windows\\UsrClass.dat"
```

通过这个信息我们看到，在`C:\WINDOWS\system32\config`目录中存储着`HKLM`所有项的`hive`文件以及一个`HKU`的`DEFAULT` `hive`文件，其他的`HKU`下的`hive`文件存储在`c:\Documents and Settings`等子目录下。在`hivelist`记录的这些信息中，我们看到`hive`文件只能加载到`HKLM`和`HKU`下。

配置管理器从逻辑上将一个`hive`分成一些称为`block`的分配单元。

其方式类似于文件系统将一个磁盘分成簇。根据定义，注册表`block`的大小为`4096`字节（`4kb`）。当新的数据要扩展一个`hive`时，该`hive`总是按照 `block`的粒度来增加。`hive`的第一个`block`称为`base block.` 里面包含了一些全局信息，包括一个特征签名`regf`、更新的序列号，时间戳，`hive`格式版本号、校验和以及该`hive`的内部文件名（如：`\Device \HarddiskVolume1\WINDOWS\system32\config\SECURITY`）。

结构如下：

```
lkd> dt _HBASE_BLOCK

nt!_HBASE_BLOCK

   +0x000 Signature         : Uint4B

   +0x004 Sequence1         : Uint4B

   +0x008 Sequence2         : Uint4B

   +0x00c TimeStamp         : _LARGE_INTEGER

    +0x014 Major             : Uint4B

    +0x018 Minor             : Uint4B

    +0x01c Type              : Uint4B

    +0x020 Format            : Uint4B

    +0x024 RootCell          : Uint4B

    +0x028 Length            : Uint4B

    +0x02c Cluster           : Uint4B

    +0x030 FileName          : [64] UChar

   +0x070 Reserved1         : [99] Uint4B

    +0x1fc CheckSum          : Uint4B

    +0x200 Reserved2         : [894] Uint4B

    +0xff8 BootType          : Uint4B

    +0xffc BootRecover       : Uint4B
```

`windows`将一个`hive`所存储的注册表数据组织在一种称为`cell`的容器中。一个`cell`可以容纳一个键、一个值、一个安全描述符、一列子键或者一列键值。其对应的结构如下：

```
typedef struct _CELL_DATA

{

     union _u

    {

         CM_KEY_NODE       KeyNode;

         CM_KEY_VALUE      KeyValue;

         CM_KEY_SECURITY   KeySecurity;     // Variable security descriptor length

         CM_KEY_INDEX      KeyIndex;        // Variable sized structure

         CM_BIG_DATA       ValueData;       // This is only for big cells; a list of cells

                                          // all of the length CM_KEY_VALUE_BIG

         HCELL_INDEX       KeyList[1];      // Variable sized array

         WCHAR             KeyString[1];    // Variable sized array

     } u;

} CELL_DATA, *PCELL_DATA;
```

该结构中嵌套了一个联合体，用于代表`cell`可以存放的不同类型的数据。

下面给出`CELL_DATA`可以存放的几种不同类型的`cell`结构：

```
lkd> DT _CM_KEY_NODE

nt!_CM_KEY_NODE

    +0x000 Signature         : Uint2B

    +0x002 Flags             : Uint2B

    +0x004 LastWriteTime     : _LARGE_INTEGER

    +0x00c Spare             : Uint4B

    +0x010 Parent            : Uint4B

    +0x014 SubKeyCounts      : [2] Uint4B

    +0x01c SubKeyLists       : [2] Uint4B

    +0x024 ValueList         : _CHILD_LIST

    +0x01c ChildHiveReference : _CM_KEY_REFERENCE

    +0x02c Security          : Uint4B

    +0x030 Class             : Uint4B

    +0x034 MaxNameLen        : Pos 0, 16 Bits

    +0x034 UserFlags         : Pos 16, 4 Bits

    +0x034 VirtControlFlags : Pos 20, 4 Bits

    +0x034 Debug             : Pos 24, 8 Bits

    +0x038 MaxClassLen       : Uint4B

    +0x03c MaxValueNameLen   : Uint4B

    +0x040 MaxValueDataLen   : Uint4B

    +0x044 WorkVar           : Uint4B

    +0x048 NameLength        : Uint2B

    +0x04a ClassLength       : Uint2B

    +0x04c Name              : [1] Uint2B
```

```
lkd> DT _CM_KEY_VALUE

nt!_CM_KEY_VALUE

    +0x000 Signature         : Uint2B

    +0x002 NameLength        : Uint2B

    +0x004 DataLength        : Uint4B

    +0x008 Data              : Uint4B

    +0x00c Type              : Uint4B

    +0x010 Flags             : Uint2B

    +0x012 Spare             : Uint2B

    +0x014 Name              : [1] Uint2B
```

```
lkd> dt _CM_KEY_SECURITY

nt!_CM_KEY_SECURITY

    +0x000 Signature         : Uint2B

    +0x002 Reserved          : Uint2B

    +0x004 Flink             : Uint4B

    +0x008 Blink             : Uint4B

    +0x00c ReferenceCount    : Uint4B

    +0x010 DescriptorLength : Uint4B

    +0x014 Descriptor        : _SECURITY_DESCRIPTOR_RELATIVE
```

```
lkd> dt _CM_KEY_INDEX

nt!_CM_KEY_INDEX

   +0x000 Signature        : Uint2B

   +0x002 Count            : Uint2B

   +0x004 List             : [1] Uint4B
```

```
lkd> dt _CM_BIG_DATA

nt!_CM_BIG_DATA

   +0x000 Signature        : Uint2B

   +0x002 Count            : Uint2B

   +0x004 List             : Uint4B
```

如果一个`cell`要加入一个`hive`，而该`hive`必须扩充才能装下它时，系统会创建一个 称为 `bin`的分配单元。该`bin`的范围包括了新添`cell`所占空间再延续到下一个`block`边界。系统将`bin`中除了`cell`以外的部分视为可以分配给其他`cell`的空闲空间。 `bin`的头部中包含了一个签名`hbin`、一个存储了`bin`在`hive`文件中偏移量的字段以及`bin`的大小。

```
typedef struct _HBIN {

    ULONG       Signature;

    ULONG       FileOffset;     // Own file offset (used in checking)

    ULONG       Size;           // Size of bin in bytes, all inclusive

    ULONG       Reserved1[2];   // Old FreeSpace and FreeList (from 1.0)

    LARGE_INTEGER   TimeStamp; // Old Link (from 1.0). Usually trash, but

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
