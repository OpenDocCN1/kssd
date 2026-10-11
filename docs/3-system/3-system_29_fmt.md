
# PE 节 load 时的对齐问题

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-135.htm`](https://www.pediy.com/kssd/pediy09/pediy09-135.htm)

> 【 标题 】 PE 节 load 时的对齐问题  
> 【 作者 】 `linxer`  
> 【 环境 】 `xp sp2`  
> 【 Q Q 】 `3568599`  
> 【 声明 】 俺系初级选手，高手略过。失误之处敬请诸位大侠赐教！

本问题的研究源至 `upack 0.32` load 后虚拟脱壳，发现自己写的 load 程序有很大问题，`upack 0.32` 加壳后程序虚拟机中跑不起来，连入口处反汇编出来的代码都不对......后来，发现是 section load 不对。经过本人反复改写节表头信息(`IMAGE_SECTION_HEADER`),然后用 OD 加载，看其怎么 load，发现以下规律,以前曾在论坛上发过一段 PE load 的代码，当时，对节表对齐属性该怎么效验也不清楚，那个时候，在 load `FSG1.33` 时，发现有些节属性是不对齐的，但 windows 却能让这个程序跑起来，心中甚是不解，后来没有办法，干脆把节对齐属性效验部分给屏蔽了(其实那个时候是自己的程序写的不对，效验太严格了)

前几天，在 `expressor v1.2` 虚拟脱壳后 dump 时，发现 dump 后程序 windows 无法 load，我用 `OllyDump` 插件把 OD 中脱壳后程序 dump 出来也有这样的问题，后来发现是：`expressor v1.2` 加壳后程序，其 `SizeOfImage` 不为 `SectionAlignment` 整数倍，我写的那个 dump 是在相对 PE 头 `SizeOfImage` 处建新节的(从以前看的 `OllyDump` 插件源码学的，不过，目前最新的源码好像修正这个 bug 了，可能是我用的插件比较老吧，才 load 不起来)，这样就出现了最后一个节没对齐，windows load 不起来。

下面结合我用 `upack v0.32` 加壳后程序说明之，高手请飘过，内容很简单。

以下示例中对齐属性如下：

```
00000078    00100000    DD 00001000          ;  SectionAlignment = 1000
0000007C    00020000    DD 00000200          ;  FileAlignment = 200
```

说下 `FileAlignment` 字段吧，这个字段在有些程序可以看到是 `0x1000` 等，我通过试验，好像发现这个字段不管是什么值，系统都用 `0x200`，不过它必须大于等于 `0x200`，小于等于 `0x1000`，并为 `0x200` 整数倍

为了说明问题，这里定义两个变量：

```
SectionAlignmentMask = SectionAlignment - 1;
FileAlignmentMask = 0x200 - 1;
```

如不加说明，以下几个示例均非最后一个节

### 示例 1：

节表信息如下：

```
00000138    2E 55 70 61>ASCII ".Upack"       ; SECTION
00000140    00800000    DD 00008000          ;  VirtualSize = 8000 (32768.)
00000144    00100000    DD 00001000          ;  VirtualAddress = 1000    //按一般书上说，这个要对齐
00000148    B7000000    DD 000000B7          ;  SizeOfRawData = B7 (183.)   //按一般书上说，这个也要对齐
0000014C    11000000    DD 00000011          ;  PointerToRawData = 11
00000150    00000000    DD 00000000          ;  PointerToRelocations = 0
00000154    00000000    DD 00000000          ;  PointerToLineNumbers = 0
00000158    0000        DW 0000              ;  NumberOfRelocations = 0
0000015A    0000        DW 0000              ;  NumberOfLineNumbers = 0
0000015C    600000E0    DD E0000060          ;  Characteristics = CODE|INITIALIZED_DATA|EXECUTE|READ|WRITE
```

上面这个节，貌似：将磁盘中偏移 `0x11`，大小 `0xb7` 的内容拷到内存中 `VirtualAddress=0x1000` 处，当然这个节在内存中大小是 `0x8000`，未填充部分全填 0

但，其实不是这样的，windows 在 load 的时候是将磁盘中偏移 `0x00`，大小 `0x200` 的内容拷到内存中去了，出乎我这个菜鸟的意料了

### 示例 2：

节表信息如下：

```
00000138    2E 55 70 61>ASCII ".Upack"       ; SECTION
00000140    00800000    DD 00008000          ;  VirtualSize = 8000 (32768.)
00000144    00100000    DD 00001000          ;  VirtualAddress = 1000
00000148    B7020000    DD 000002B7          ;  SizeOfRawData = 2B7 (695.)
0000014C    f1000000    DD 000000f1          ;  PointerToRawData = f1
00000150    00000000    DD 00000000          ;  PointerToRelocations = 0
00000154    00000000    DD 00000000          ;  PointerToLineNumbers = 0
00000158    0000        DW 0000              ;  NumberOfRelocations = 0
0000015A    0000        DW 0000              ;  NumberOfLineNumbers = 0
0000015C    600000E0    DD E0000060          ;  Characteristics = CODE|INITIALIZED_DATA|EXECUTE|READ|WRITE
```

看下这个 win 怎么 load 到内存的，它将磁盘中偏移 `0x00`，大小 `0x400` 的内容拷到内存中去了

好了，看了上面两个例子，你应该知道它是怎么从磁盘中拷数据出去了吧，其思路用代码表示如下：

```
SizeOfRawData = (SizeOfRawData + FileAlignmentMask) & (0xffffffff ^ FileAlignmentMask);
PointerToRawData = PointerToRawData & (0xffffffff ^ FileAlignmentMask);
```

### 示例 3：

节表信息如下：

```
00000138    2E 55 70 61>ASCII ".Upack"       ; SECTION
00000140    00810000    DD 00008100          ;  VirtualSize = 8100 (33024.)
00000144    00100000    DD 00001000          ;  VirtualAddress = 1000
00000148    B7020000    DD 000002B7          ;  SizeOfRawData = 2B7 (695.)
0000014C    f1000000    DD 000000f1          ;  PointerToRawData = f1
00000150    00000000    DD 00000000          ;  PointerToRelocations = 0
00000154    00000000    DD 00000000          ;  PointerToLineNumbers = 0
00000158    0000        DW 0000              ;  NumberOfRelocations = 0
0000015A    0000        DW 0000              ;  NumberOfLineNumbers = 0
0000015C    600000E0    DD E0000060          ;  Characteristics = CODE|INITIALIZED_DATA|EXECUTE|READ|WRITE
```

上面这个例子把 `VirtualSize` 改成没有对齐了，发现程序图标也没了，windows 还 load 不起来了，这个地方为什么不能运行是因为我们把 `VirtualSize` 调大了，占了第二个节的地方了，如果调小的话，则可以运行，可以看出这个字段的效验是不跨越其它节地界，这个字段只要不跨越其它节地界，为任何值都可以，0 也不例外，这个值除了用来效验，没有其它用

### 示例 4：

节表信息如下：

```
00000138    2E 55 70 61>ASCII ".Upack"       ; SECTION
00000140    00800000    DD 00008000          ;  VirtualSize = 8000 (32768.)
00000144    00110000    DD 00001100          ;  VirtualAddress = 1100
00000148    B7000000    DD 000000B7          ;  SizeOfRawData = B7 (183.)
0000014C    11000000    DD 00000011          ;  PointerToRawData = 11
00000150    00000000    DD 00000000          ;  PointerToRelocations = 0
00000154    00000000    DD 00000000          ;  PointerToLineNumbers = 0
00000158    0000        DW 0000              ;  NumberOfRelocations = 0
0000015A    0000        DW 0000              ;  NumberOfLineNumbers = 0
0000015C    600000E0    DD E0000060          ;  Characteristics = CODE|INITIALIZED_DATA|EXECUTE|READ|WRITE
```

这个示例跟示例 3 一样，图标没了，windows 也 load 不起来了，`VirtualAddress` 这个字段也是要严格效验的

好了，下面来看最后一个节的情况(以下均为最后一个节了)

### 示例 5：

节表信息如下：

```
00000160    2E 72 73 72>ASCII ".rsrc"        ; SECTION
00000168    00500000    DD 00005000          ;  VirtualSize = 5000 (20480.)
0000016C    00900000    DD 00009000          ;  VirtualAddress = 9000
00000170    48320000    DD 00003248          ;  SizeOfRawData = 3248 (12872.)
00000174    00020000    DD 00000200          ;  PointerToRawData = 200
00000178    00000000    DD 00000000          ;  PointerToRelocations = 0
0000017C    00000000    DD 00000000          ;  PointerToLineNumbers = 0
00000180    0000        DW 0000              ;  NumberOfRelocations = 0
00000182    0000        DW 0000              ;  NumberOfLineNumbers = 0
00000184    600000E0    DD E0000060          ;  Characteristics = CODE|INITIALIZED_DATA|EXECUTE|READ|WRITE
```

这个节是 `upack v0.32` 加壳后程序的最后一个节，是好的，可见 `SizeOfRawData` 也没有对齐，这说明最后一个节 `SizeOfRawData` 可以不对齐

### 示例 6：这个示例我们让 `PointerToRawData` 不对齐看看

节表信息如下：

```
00000160    2E 72 73 72>ASCII ".rsrc"        ; SECTION
00000168    00500000    DD 00005000          ;  VirtualSize = 5000 (20480.)
0000016C    00900000    DD 00009000          ;  VirtualAddress = 9000
00000170    48320000    DD 00003248          ;  SizeOfRawData = 3248 (12872.)
00000174    10020000    DD 00000210          ;  PointerToRawData = 210
00000178    00000000    DD 00000000          ;  PointerToRelocations = 0
0000017C    00000000    DD 00000000          ;  PointerToLineNumbers = 0
00000180    0000        DW 0000              ;  NumberOfRelocations = 0
00000182    0000        DW 0000              ;  NumberOfLineNumbers = 0
00000184    600000E0    DD E0000060          ;  Characteristics = CODE|INITIALIZED_DATA|EXECUTE|READ|WRITE
```

这么一改，程序图标又没了，windows 也 load 不起来了，这是因为我们加大了 `PointerToRawData`,导致 `SizeOfRawData + PointerToRawData > 文件长度`了，所以 windows 不让 load 了，如果将其改小的话，可以 load，但是会出现什么错误视具体程序定

下面还是冗余的改下 `VirtualSize` 和 `VirtualAddress` 字段吧，看看如何

### 示例 7：

节表信息如下：

```
00000160    2E 72 73 72>ASCII ".rsrc"        ; SECTION
00000168    00500000    DD 00005000          ;  VirtualSize = 5000 (20480.)
0000016C    00910000    DD 00009100          ;  VirtualAddress = 9100
```



`00000170`    `48320000`    `DD` `00003248`          ;  `SizeOfRawData` = `3248` (`12872.`)  

`00000174`    `00020000`    `DD` `00000200`          ;  `PointerToRawData` = `200`  

`00000178`    `00000000`    `DD` `00000000`          ;  `PointerToRelocations` = `0`  

`0000017C`    `00000000`    `DD` `00000000`          ;  `PointerToLineNumbers` = `0`  

`00000180`    `0000`        `DW` `0000`              ;  `NumberOfRelocations` = `0`  

`00000182`    `0000`        `DW` `0000`              ;  `NumberOfLineNumbers` = `0`  

`00000184`    `600000E0`    `DD` `E0000060`          ;  `Characteristics` = `CODE|INITIALIZED_DATA|EXECUTE|READ|WRITE`  

让 `VirtualAddress` 不对齐后，程序图标又没了，`windows` 也 `load` 不起来了，可见最后一个节的 `VirtualAddress` 也有严格的效验  

### 示例 8

节表信息如下：

`00000160`    `2E` `72` `73` `72` > `ASCII` `".rsrc"`        ;  `SECTION`  
`00000168`    `10500000`    `DD` `00005010`          ;  `VirtualSize` = `5010` (`20496.`)  
`0000016C`    `00900000`    `DD` `00009000`          ;  `VirtualAddress` = `9000`  
`00000170`    `48320000`    `DD` `00003248`          ;  `SizeOfRawData` = `3248` (`12872.`)  
`00000174`    `00020000`    `DD` `00000200`          ;  `PointerToRawData` = `200`  
`00000178`    `00000000`    `DD` `00000000`          ;  `PointerToRelocations` = `0`  
`0000017C`    `00000000`    `DD` `00000000`          ;  `PointerToLineNumbers` = `0`  
`00000180`    `0000`        `DW` `0000`              ;  `NumberOfRelocations` = `0`  
`00000182`    `0000`        `DW` `0000`              ;  `NumberOfLineNumbers` = `0`  
`00000184`    `600000E0`    `DD` `E0000060`          ;  `Characteristics` = `CODE|INITIALIZED_DATA|EXECUTE|READ|WRITE`  

让 `VirtualSize` 不对齐后，程序图标还有(我们是加大了 `VirtualSize`，如果减小，估计就会没了，555，失算了，把它改成 `0`，还有图标)，但是 `windows` `load` 不起来了，如果把它改成 `0x4500` 也可以运行，这里调大了，导致最后一个节的 `VirtualAddress` + `VirtualSize` > `SizeOfImage` 了，因此 `load` 不起来，这种情况 `windows` 也是要效验的  

下面给出这些属性的效验代码：  

```c
//pe check 函数  
//合法返回 0，否则-1  
//lpExePEBuff 是 pe 文件地址  
//nExePELen 是 pe 文件长度  

long pe_check(char *lpExePEBuff, unsigned long nExePELen)  
{  
  unsigned long lSectionNum;  
  unsigned long lFileAlign;  
  unsigned long lFileAlignMask;  
  unsigned long lSectionAlignMask;  
  unsigned long nPointerToRawData;  
  unsigned long nSizeOfRawData;  
  unsigned long nSizeOfImage;  
  unsigned long nIndex;  

  IMAGE_DOS_HEADER *imDos_Headers;  
  IMAGE_NT_HEADERS * pINH;  
   
  IMAGE_SECTION_HEADER *pISH;  
  IMAGE_SECTION_HEADER * pOldEndISH;  

  //MZ PE 标志效验省去  
  imDos_Headers = (IMAGE_DOS_HEADER *)lpExePEBuff;  
  pINH = (IMAGE_NT_HEADERS *)((char *)lpExePEBuff + imDos_Headers->e_lfanew);//NT 头指针地址  
   
  //取 PE 文件的节数量  
  lSectionNum = pINH->FileHeader.NumberOfSections;  
   
  pISH = (IMAGE_SECTION_HEADER *)((char *)pINH + sizeof(DWORD) + sizeof(IMAGE_FILE_HEADER) + pINH->FileHeader.SizeOfOptionalHeader);  
  pOldEndISH = pISH + lSectionNum - 1;  


  //SizeOfImage 按对齐属性向上取整  
  nSizeOfImage = pINH->OptionalHeader.SizeOfImage;  
  //各节在磁盘中的对齐掩码  
  lSectionAlignMask = pINH->OptionalHeader.SectionAlignment - 1;  //各节在 load 后内存中的对齐掩码  
  nSizeOfImage = (nSizeOfImage + lSectionAlignMask) & (0xffffffff ^ lSectionAlignMask);  

  //效验最后节 VirtualSize 是否越界 SizeOfImage  
  if(pOldEndISH->VirtualAddress + pOldEndISH->Misc.VirtualSize > nSizeOfImage)  
  {  
    return -1;  
  }  

  //貌似 FileAlignment 无用，系统好像是用 0x200，只要是 0x200 整数倍，不大于 0x1000，程序都可以运行  
  lFileAlign = g_pINH->OptionalHeader.FileAlignment;  
  if(lFileAlign < 0x200 && lFileAlign > 0x1000)  
  {  
    return -1;  
  }  
  if(lFileAlign % 0x200)  
  {  
    return -1;  
  }  
  lFileAlign = 0x200;  
  
  lFileAlignMask = lFileAlign - 1;               
  
  for(nIndex = 0; nIndex < lSectionNum; nIndex++, pISH++)  
  {  
    //效验 VirtualAddress 对齐属性  
    if(pISH->VirtualAddress & lSectionAlignMask)  
    {  
      //出现非法节  
      return -1;  
    }  

    //效验要装载磁盘数据是否超出文件大小  
    if(pISH->PointerToRawData + pISH->SizeOfRawData > nExePELen)  
    {  
      return -1;  
    }  

    //效验 VirtualSize 是否跨入下一节地界  
    if(pISH != pOldEndISH)  
    {  
      if(pISH->VirtualAddress + pISH->Misc.VirtualSize > (pISH + 1)->VirtualAddress)  
      {  
        //出现非法节  
        return -1;  
      }  
    }  

    //不管对不对齐均重算 PE 节磁盘地址，磁盘中大小信息  
    nPointerToRawData = pISH->PointerToRawData & (0xffffffff ^ lFileAlignMask);  
    nSizeOfRawData = (pISH->SizeOfRawData + lFileAlignMask) & (0xffffffff ^ lFileAlignMask);  
    //nSizeOfRawData 要跟磁盘中剩余长度比较，取小者  
    if(nSizeOfRawData > nExePELen - nPointerToRawData)  
    {  
      nSizeOfRawData = nExePELen - nPointerToRawData;  
    }  

    if(pISH != pOldEndISH)  
    {  

      //发现这个 VirtualSize 没有用，用 0 程序也可以运行，这里效验如下  
      nVirtualSize = (pISH + 1)->VirtualAddress - pISH->VirtualAddress;  
      //nSizeOfRawData 大于 nVirtualSize 也不行  
      if(nSizeOfRawData > nVirtualSize)  
      {  
        return -1;  
      }  
    }  

    //如果要 load 的话，是用 nSizeOfRawData/nPointerToRawData  
  }  
  
  return 0;  
}
```

### 总结

1. `VirtualSize` 可以不对齐，但是它表示的值不能跨到其它节地界  
2. `VirtualAddress` 在任何节中也都要对齐  
3. `SizeOfRawData` 没有对齐要求，如不对齐，按这个公式重算：`SizeOfRawData = (SizeOfRawData + FileAlignmentMask) & (0xffffffff ^ FileAlignmentMask);`  
4. `PointerToRawData` 没有对齐要求，如不对齐，按这个公式重算：`PointerToRawData = PointerToRawData & (0xffffffff ^ FileAlignmentMask);`  

修正了这个帖子中程序的一处 bug，这个 bug 估计很多向我一样的菜鸟都会犯吧，那就是怎么求节表开始地址：  

我以前一直用这个方法求节表开始地址：`PE` 标志地址 + `sizeof(IMAGE_NT_HEADERS);`  

刚才 `load` `upack v0.36` 时，发现不是这样的，跟踪了下 `PETools`，发现是用这个方法求的：`PE` 标志地址 + `sizeof(DWORD) + sizeof(IMAGE_FILE_HEADER) + pINH->FileHeader.SizeOfOptionalHeader;`  

哈哈，看来今天收获不小：）

> 引用: *最初由 **linxer**发布* `[查看帖子](img/showthread.php?p=305287#post305287)` 修正了这个帖子中程序的一处 bug，这个 bug 估计很多向我一样的菜鸟都会犯吧，那就是怎么求节表开始地址：  
>  
> 我以前一直用这个方法求节表开始地址：`PE` 标志地址 + `sizeof(IMAGE_NT_HEADERS);`  
>  
> 刚才 `load` `upack v0.36` 时，发现不是这样的，跟踪了下 `PETools`... 

根本原因在于,在某些被加壳的程序里，`FileHeader.SizeOfOptionalHeader` 的值并不是标准的 `0xE0`

> 引用: *最初由 **InNULL**发布* `[查看帖子](img/showthread.php?p=364289#post364289)` 貌似有道理，那些制造最小字节 `PE` 的程序，不是都用 `32` 字节文件对齐吗！

制造最小 `PE` 文件,貌似跟文件对齐属性关系不大,文件对齐属性主要用来决定怎么 `load` 文件的,刚来修正了下文章,加入了对 `SizeOfImage` 的向上取整,也搞了一个小错误,目前这小段程序应该经历了近 400 个小版本壳的考验,潜在的错误还有待发掘(水平有限,没逆过 `windows` 的 `loader`,基本上都是总结的,错误在所难免),目前工作良好,当然这都是在 `xp sp2` 上的测试结果,在其它平台上怎么样,没有测试过,尤其是 `2000` 系统

下面对 4 楼给出的文字进行评论下,测试平台 `xp sp2`,其它平台不清楚,没测试

以下引用大牛的话------------------------------

1. 如果 `FileAlignment` 小于 `200h`，则要求 `FileAlignment == SectionAlignment >= 2`  
2. 如果 `FileAlignment` 小于 `200h`，则要求 `VirtualAddress == PointerToRawData`

满足以上两个条件,在 `xp sp2` 上程序无法运行,提示不是有效 `win32` 应用程序

3. `VirtualSize <= SizeOfRawData`

第三个条件不满足,在 `xp sp2` 上程序可以正常运行

4. `SizeOfHeaders < SizeOfImage`

5. `NumberOfRvaAndSizes >= 2` 数据目录结构的数量要求不小于 `2`

6. 节表和输入表都要求有一个结束的全 `0` 成员

第 6 个条件,在 `xp sp2` 平台上,节表和输入表都可以不以全 `0` 成员结束,程序可以正常运行

再次声明,以上只是盗版 `xp sp2` 的测试结果,其它平台没有测试过

不同的操作系统是有点不一样的，可能是 `2000` 的系统判断没那么严格

看看 `windows` 源码里面的 `imagechk.c` 就知道大概是怎么回事了

比如

```c
  NumberOfSubsections = NtHeader->FileHeader.NumberOfSections;  
  PreferredImageBase = NtHeader->OptionalHeader.ImageBase;  
   
  //  
  // At this point the object table is read in (if it was not  
  // already read in) and may displace the image header.  
  //  
   
  OffsetToSectionTable = sizeof(ULONG) + sizeof(IMAGE_FILE_HEADER) + NtHeader->FileHeader.SizeOfOptionalHeader;  
   
  SectionTableEntry = (PIMAGE_SECTION_HEADER)((PCHAR)NtHeader + OffsetToSectionTable);  
   
  if (ImageAlignment < PageSize)  
  {  
    // The image header is no longer valid, TempPte is
```



```c
// used to indicate that this image alignment is  
// less than a PageSize.  

//  
// Loop through all sections and make sure there is no  
// unitialized data.  
//  
  
while (NumberOfSubsections > 0)  
{  
  if (SectionTableEntry->Misc.VirtualSize == 0)  
  {  
    SectionVirtualSize = SectionTableEntry->SizeOfRawData;  
  }  
  else  
  {  
    SectionVirtualSize = SectionTableEntry->Misc.VirtualSize;  
  }  
  

  //  
  // If the pointer to raw data is zero and the virtual size  
  // is zero, OR, the section goes past the end of file, OR  
  // the virtual size does not match the size of raw data, then  
  // return an error.  
  //  
  

  if (((SectionTableEntry->PointerToRawData != SectionTableEntry->VirtualAddress))  
    ||  
    ((SectionTableEntry->SizeOfRawData + SectionTableEntry->PointerToRawData) >  
    FileInfo.nFileSizeLow)  
    ||  
    (SectionVirtualSize > SectionTableEntry->SizeOfRawData))  
  {  
    printf("invalid BSS/Trailingzero section/file size\\n");  
    

    ImageOk = FALSE;  
    goto NeImage;  
  }  
  SectionTableEntry += 1;  
  NumberOfSubsections -= 1;  
}  
goto PeReturnSuccess;  
}  
```

其实没什么秘密

`4dd466d0e1e0cc123afbe504da2462ca`
