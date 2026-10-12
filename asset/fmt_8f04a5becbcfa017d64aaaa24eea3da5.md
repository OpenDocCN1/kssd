
# NTFS 学习笔记

> 原文：[`www.pediy.com/kssd/pediy11/126204.html`](https://www.pediy.com/kssd/pediy11/126204.html)

最近学习 NTFS 分区格式，学的很郁闷，看了一天没看懂![](img/110d920e4e703968a2a60c986d363163.jpg "尴尬")，今天又看了半天终于看懂了，完全可以手工去定位文件。附件中是我搜集到的资料，看雪上也有几个。

为了总结自己学习的东西，也为了检验自己，把过程贴出来。

以下是我自己总结的内容

NTFS 与 FAT 有着很大的区别，首先它有一个 `MFT` 表(`MasterFileTable`),这个表中都是一个个的文件记录结构  每个文件记录是 1KB（`400H`），每个文件记录或者多个文件记录来描述一个文件的信息，`MFT` 中有 16 个系统使用的文件记录和 8 个保留的记录，每个文件记录是采用属性-属性的值来描述这个文件的，并且所有的属性分为了很多类型。

类型  操作系统  描述

`0x10`    `STANDARD_INFORMATION`

`0x20`    `ATTRIBUTE_LIST`

`0x30`    `FILE_NAME`

`0x40`  `NT`  `VOLUME_VERSION`

`0x40`  `2K`  `OBJECT_ID`

`0x50`    `SECURITY_DESCRIPTOR`

`0x60`    `VOLUME_NAME`

`0x70`    `VOLUME_INFORMATION`

`0x80`    `DATA`

`0x90`    `INDEX_ROOT`

`0xA0`    `INDEX_ALLOCATION`

`0xB0`    `BITMAP`

`0xC0`  `NT`  `SYMBOL_LINK`

`0xC0`  `2K`  `REPARSE_POINT`

`0xD0`    `EA_INFORMATION`

`0xE0`    `EA`

`0xF0`  `NT`  `PROPERTY_SET`

`0x100`  `2K`  `LOGGED_UNTILITY_STREAM`

这些属性就像类一样，可以建立有名字的属性也可以是无名字的属性。要描述一个属性，要有一个属性头，属性头有 2 种，非常驻属性头和常驻属性头对应着 2 种不同的属性，常驻属性是数据在文件记录中的，非常驻属性是数据在别的地方，非常驻属性的数据要借助于一种叫流的东西来实现，流其实是描述的一组簇。

在虚拟机中添加一块硬盘，格式化为 NTFS 盘符为 `E`  建立文件 `E:\\dir\\123456789.txt`

写入”`liutong`”数据

因为只分了一个分区，所以很容易通过 `MBR` 找到 `DBR`   截图如下:

![](img/8c1bb1d4e109ee5e7e8c5563d54c1afa.jpg)

通过 `DBR` 中的 `BPB` 得到每簇的大小为 8 扇区（`1000H`），`MFT` 的簇号为 `40000H`,这个簇号是逻辑簇号，从 `DBR` 开始算，所以计算出 `MFT` 的偏移是:`7E00+40000*1000=40007E00h`

`MFT` 截图如下

![](img/78015401794c7d6c03ea6af342c8105b.jpg)

每个文件记录都是以一个"`FILE`”开始的。

要想找到我们的文件先要找到 `dir` 目录在哪里，可以先通过 `$root` 这个文件记录来查找

`$root` 在 `MFT` 中偏移为 5，可以计算它的实际偏移为:`5*400h+40007e00 = 40009200`

`$root` 文件记录截图

![](img/3ed63266e143f4fea000ef638934aa62.jpg)

然后可以通过 `root` 文件的 `INDEX_ALLOCATION` 属性来得到 `dir` 目录的信息, `INDEX_ALLOCATION` 属性的结构是  属性头+`IA` 索引块+索引项，由于是非常驻属性（在属性头中表明），所以后面的 `IA` 索引块和索引项在外部，不是紧随其后。

`INDEX_ALLOCATION` 截图

![](img/124529e79b826ae8a9ef2c6570289d14.jpg)

通过图片得这个属性它还有个名字 `$I30`  ,名字后面是数据流描述了，`31 01 09 FD 09`   代表流的起始虚拟簇为 `09FD09`  大小为 1 个簇 因为最后一个 `09` 后面为 0  所以就只描述了一个流   虚拟簇是相对于属性头中的 `ATTR_StartVCN` 这个字段来讲的  因为 `ATTR_StartVCN` 为 0  所以 `09FD09`  也就是逻辑簇   可以计算出偏移为

`9FD09*1000+7e00 = 9fd10e00`   这个地址就是 `IA` 索引块+索引项的地址截图如下

![](img/8e5d7c01add90f958e4fd068d1500742.jpg)

每个流只有一个 `IA` 索引块 每个索引项 索引一个文件或目录，索引项中记录了文件或者目录的名称  所以直接搜索“`dir`”  Unicode 形式的

`DIR` 目录的索引项如下

![](img/76661cde646225466de24c7f272617de.jpg)

可以得到 `dir` 这个文件在 `MFT` 中对应的索引号为 `1E`

计算偏移为 `40007E00h+1E*400 = 4000F600`  通过这个文件记录的 `Index_Root` 属性，也可以通过 `Index_Allocation` 属性来获得 `123456789.txt` 的信息，通过 `Index_Root` 是在目录或者文件不大的情况下才能获得。所以 `Index_Root` 它是一个常驻属性，属性和属性的值都在文件记录中。

`Index_Root`:

![](img/2ed794e8fbb722ad5d4e3cef863d4f4c.jpg)

可以看到这个属性有 `130H` 的大小,有个名字  `$I30`  ,名字后面就是属性的实体,后面是 16 字节的信息头+16 字节的索引块头+索引项（如果存在的话）  ，可以看到有 2 个索引项  一个是 长文件名，一个是短文件名  ，无论如何  这 2 个都是一样的  得到文件对应 `MFT` 的索引是 `22`

计算偏移得 `40007E00h+22*400 = 40010600h` ,可以通过这个文件记录的 `DATA` 属性来得到数据

`DATA` 属性截图

![](img/ff1fab3a32a32a0d4d8dd2dc5cd55600.jpg)

`80` 是 `DATA` 属性的代码，这是个常驻属性，也就是说数据在文件记录中，属性头后面就是数据了  即”`liutong`” ,如果数据量过大，这个属性就会变成非常驻属性，数据在别处.

我把文件扩展到 2870 个字节    `DATA` 属性如下

非常驻 `DATA` 属性截图:

![](img/12339e78f905e7d3f5f16093cd6dc375.jpg)

`400107f0` 处就为数据流描述  ，只有一个 虚拟簇号为 `09FD07`  `ATTR_StartVCN` 为 0   所以 `09FD07`  是逻辑簇号

计算偏移可以得到 `9FD07*1000+7e00 = 9FD0EE00`   是数据的偏移

数据截图

![](img/8375b1a32a98ff0ec64aafd2d10055f2.jpg)

`[ATTACH][ATTACH][ATTACH][ATTACH][ATTACH]WINDOWS 平台下磁盘逻辑扇区的直接读写.doc[/ATTACH][/ATTACH][/ATTACH][/ATTACH][/ATTACH]`

上传的附件

| ![](img/58e0728fcb44eb7297cf87d68539ed4e.jpg) || NTFS and boot code.doc |

| ![](img/58e0728fcb44eb7297cf87d68539ed4e.jpg) || NTFS 白皮书.doc |

| ![](img/668feaf58c8e7ddb6fbdbf839a5a19db.jpg) || NTFS 文件系统剖析.pdf |

| ![](img/668feaf58c8e7ddb6fbdbf839a5a19db.jpg) || NTFS 系统存储介质上文件操作痕迹分析.pdf |

| ![](img/58e0728fcb44eb7297cf87d68539ed4e.jpg) || 解析 NTFS 底层结构.doc |
