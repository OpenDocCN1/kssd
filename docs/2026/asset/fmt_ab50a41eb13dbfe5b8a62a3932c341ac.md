
`srvnet.sys` 6.1.7601.17514

`nsa` 工具集（[使用教程](https://bbs.kanxue.com/elink@1c8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2B7K9h3q4F1M7$3S2#2i4K6u0W2j5$3!0E0i4K6u0r3M7q4)9J5c8X3p5@1x3r3p5%4k6o6p5%4k6h3g2W2j5H3`.`.)）

PS:这两个文件在`C:\Windows\System32\drivers`下

参考资料：

9a1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6J5k6i4y4W2j5i4u0U0K9q4)9J5k6h3y4Z5k6h3y4C8M7r3!0A6L8Y4c8Q4x3X3g2U0L8$3#2Q4x3V1j5J5x3o6p5%4i4K6u0r3k6i4c8W2M7X3&6S2L8r3u0D9N6h3g2Q4x3X3c8W2N6X3g2J5P5i4c8Z5K9h3&6Y4i4K6u0V1K9$3&6G2N6#2)9J5c8R3`.`.

4e8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1j5K6L8X3c8s2y4r3#2W2i4K6u0r3b7i4g2@1L8@1u0D9N6h3g2Q4x3X3c8y4f1K6p5%4i4K6u0V1x3o6p5H3

658K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5i4m8W2M7W2)9J5k6i4y4W2k6h3u0#2k6#2)9J5k6h3!0J5k6#2)9J5c8U0t1^5x3q4)9J5c8R3`.`.

### 0x01 简介

本文将介绍以下的内容

>> 漏洞完整利用流程介绍

>> 漏洞溢出部分分析

>> 漏洞触发部分的分析

>> 漏洞的内存布局的分析

### 0x02 漏洞完整利用流程

该漏洞主要是利用`smb1`和`smb2`的协议兼容问题，和`windows`在处理`fealist`结构体和`ntfeallist`结构体过程中大小计算错误导致的数据溢出漏洞。

在进行堆喷射过程中，为了实现非页内存的布局，又利用用了一个`SMB_COM_SESSION_SETUP_ANDX`计算`smbv1`和`smbv2`结构体转化的漏洞，实现了任意大小的非页内存申请，从而间接利用系统的内存管理机制实现内存布局。

漏洞触发部分，在内存溢出和堆布局的基础上实现了对`srvnet`头部结构的覆盖，其中对`MDL`指针的覆盖，使得后续发送的`srvnetbuff`内容被保存到了特定可执行的内存地址(`0xffdff000`)中，于是在释放`srvnet`链接后，处理函数会执行`0xffdff000`地址处的`shellcode`，从而实现漏洞利用。

### 0x03 溢出部分分析

这部分的漏洞分析是大部分文章都有写的，主要成因是由于`SrvOs2FeaListSizeToNt`函数在进行`fealist`到`ntfeallist`的长度计算过程中进行了一个强制类型转换，导致了四个字节的长度只覆盖了低位的两个字节，数据在转换过程中大于申请的内存空间，从而实现溢出。此处就主要介绍一下为什么会出现四个字节转两个字节的情况？

#### 基础知识

`SMB`协议中，使用一串的命令来代表执行的操作的，当传输的数据过大时，`smb`通常会有一个子命令进行传输，并用传输过程中的`TID`,`UID`,`PID`,`MID`来判断是哪一个命令的后续数据。

例如，`smbv1`中的`SMB_COM_NT_TRANSACT`命令，在传输消息过大时，便会使用`SMB_COM_NT_TRANSACT_SECONDARY`来完成后续的数据传输。

![图片描述](https://bbs.kanxue.com/upload/attach/202207/859191_QGR4AXHBTADKE79.png)

而在`smbv2`中`SMB_COM_TRANSACTION2`作为`SMB_COM_NT_TRANSACT`的扩展命令，两者的请求结构体十分相似，功能也差不多，但在计算消息内容长度`TotalDataCount`时，`SMB_COM_TRANSACTION2`使用的是`USHORT`类型（两字节），`SMB_COM_NT_TRANSACT`使用的是`ULONG`类型（四字节）。

![图片描述](https://bbs.kanxue.com/upload/attach/202207/859191_CC3KUKM6HEG43PN.png)

`NSA`工具在利用该漏洞时，先传入了一个`SMB_COM_NT_TRANSACT`命令的头，后续内容利用相同`TID`,`PID`,`UID`,`MID`的`SMB_COM_TRANSACTION2_SECONDARY`进行传输的。没加补丁之前，`windows`仅通过`TID`,`PID`,`UID`,`MID`来识别命令是否一致，而消息命令的类型是由最后一个传入的命令类型确定的。这样就造成了传入`NT_TRANSACT`消息，但实际上是运行的却是`TRANSACTION2`命令的处理流程。

![图片描述](https://bbs.kanxue.com/upload/attach/202207/859191_ZPJJQCS4FFH5BNX.png)

所以，在补丁修复中，除了修复`SrvOs2FeaListSizeToNt`的类型强转外，还同时在`ExecuteTransaction`函数中添加了一个类型比较的判断。

![图片描述](https://bbs.kanxue.com/upload/attach/202207/859191_PXB395MQYDQE4F5.png)

用到的几个结构体的定义如下：

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19 | `typedef struct _FEALIST {`  `_ULONG( cbList );`  `FEA` `list``[``1``];`  `} FEALIST;`    `typedef struct _FEA {`  `UCHAR fEA;` `/``/` `flag 标志位用于判断循环是否结束`  `UCHAR cbName;` `/``/` `名字长度`  `_USHORT( cbValue );` `/``/` `值长度`  `} FEA;`    `/``/` `ntfealist，windows中没有直接对fealist结构进行操作而是统一使用ntfealist操作`  `typedef struct _FILE_FULL_EA_INFORMATION {`  `ULONG NextEntryOffset;`  `UCHAR Flags;`  `UCHAR EaNameLength;`  `USHORT EaValueLength;`  `CHAR EaName[``1``];`  `} FILE_FULL_EA_INFORMATION,` `*``PFILE_FULL_EA_INFORMATION;` |

#### 调试过程

`trans2`接受完数据后会通过`dispatchtable`调用`srv!SrvSmbOpen2`函数对接收到的数据进行处理，函数首先会读取接收到的`transion`数据，然后获取其中`fealist`结构体的部分，将`fealist`结构体转成`Ntfealist`结构体。

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13 | `SMB_TRANS_STATUS SrvSmbOpen2 (IN OUT PWORK_CONTEXT WorkContext){`  `/``/``...`  `transaction` `=` `WorkContext``-``>Parameters.Transaction;`  `/``/``...`  `feaList` `=` `(PFEALIST)transaction``-``>InData;`  `/``/``... Convert the FEALIST to NT style.`  `status` `=` `SrvOs2FeaListToNt(`  `feaList,`  `&ntFullEa,`  `&ntFullEaBufferLength,`  `&os2EaErrorOffset`  `);`  `}` |

在`SrvOs2FeaListToNt`函数中，首先是`SrvOs2FeaListSizeToNt`对于结构体长度的强转赋值，导致`fealist`的结构体长度是错误的，所以后续计算最后一个结构体指针的地址也是错误的。在后续循环转化`ntfealist`的过程中，循环是以`fea`结构体标志位和与最后一个结构体指针地址比较进行的条件判断，结构体标志位由用户传入，可控，指针地址错误的计算，可控，所以可以精准控制溢出字节。

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28  29  30  31  32  33  34  35  36  37  38  39  40  41  42  43  44  45  46  47  48  49  50  51  52  53  54  55  56  57  58  59  60  61  62  63  64  65 | `unsigned` `int` `__stdcall SrvOs2FeaListToNt(_FEALIST` `*``FeaList, _DWORD` `*``NtFullEa, _DWORD` `*``BufferLength, _WORD` `*``EaErrorOffset)`  `{`  `int` `NtBufferLen;` `/``/` `eax`  `_FEALIST` `*``NtfeaAddr;` `/``/` `eax`  `FEA` `*``feaLast;` `/``/` `ebx`  `FEA` `*``fea;` `/``/` `esi`  `unsigned` `int` `v10;` `/``/` `esi`  `__int16 v11;` `/``/` `[esp``+``8h``] [ebp``-``4h``]`  `_FEALIST` `*``NtFeaAddr;` `/``/` `[esp``+``14h``] [ebp``+``8h``]`    `v11` `=` `0``;`  `NtBufferLen` `=` `SrvOs2FeaListSizeToNt(FeaList);` `/``/` `计此处算长度出错`  `*``BufferLength` `=` `NtBufferLen;`  `if` `( !NtBufferLen )`  `{`  `*``EaErrorOffset` `=` `0``;`  `return` `0xC098F0FF``;` `/``/` `STATUS_OS2_EA_LIST_INCONSISTENT`  `}`  `NtfeaAddr` `=` `(_FEALIST` `*``)SrvAllocateNonPagedPool(NtBufferLen,` `21``);``/``/` `用ntfea的length申请内存空间`  `*``NtFullEa` `=` `NtfeaAddr;`  `if` `( NtfeaAddr )`  `{`  `/``/` `问题就出现在了这里，cbList经过刚刚的赋值已经发生了改变，这个feaLast的指针地址远远大于fea最后一个结构体指针地址`  `feaLast` `=` `(FEA` `*``)((char` `*``)FeaList` `+` `FeaList``-``>cbList` `-` `5``);``/``/` `为了保证至少有一个fea结构 FeaList``+``Feal``-``>cbList` `-` `sizeof(Fea)`  `fea` `=` `FeaList``-``>``list``;`  `if` `( FeaList``-``>``list` `> feaLast )`  `{`  `LABEL_13:`  `if` `( fea` `=``=` `(FEA` `*``)((char` `*``)FeaList` `+` `FeaList``-``>cbList))``/``/` `如果cblist长度是``0``，那么就把Ntfea的长度也设为``0`  `{`  `NtfeaAddr``-``>cbList` `=` `0``;`  `return` `0``;`  `}`  `*``EaErrorOffset` `=` `v11` `-` `(_WORD)FeaList;`  `v10` `=` `0xC0000001``;` `/``/` `STATUS_SUCCESS`  `}`  `else`  `{`  `while` `( (fea``-``>fEA &` `0x7F``)` `=``=` `0` `)` `/``/` `判断每个fea的标志位是不是``80``或``00``，不是就跳出循环`  `{`  `/``/` `注意，这里是以标志位为循环判断基础的，而本来那个损坏的fea结构体是不在拷贝范围内的，但由于长度计算错误，会出现在拷贝的范围内。`  `NtFeaAddr` `=` `NtfeaAddr;`  `v11` `=` `(__int16)fea;`  `NtfeaAddr` `=` `(_FEALIST` `*``)SrvOs2FeaToNt(NtfeaAddr, fea);``/``/` `拷贝内存，导致溢出的部分`  `fea` `=` `(FEA` `*``)((char` `*``)fea` `+` `fea``-``>cbName` `+` `fea``-``>cbValue` `+` `5``);``/``/` `赋值下一个fea`  `if` `( fea > feaLast )` `/``/` `这个地方由于feaLast的地址计算错误，所以肯定大于fea地址`  `{`  `NtfeaAddr` `=` `NtFeaAddr;`  `goto LABEL_13;`  `}`  `}`  `*``EaErrorOffset` `=` `(_WORD)fea` `-` `(_WORD)FeaList;`  `v10` `=` `0xC000000D``;` `/``/` `STATUS_INVALID_PARAMETER`  `}`  `SrvFreeNonPagedPool(``*``NtFullEa);`  `return` `v10;`  `}`  `if` `(` `*``((_BYTE` `*``)WPP_GLOBAL_Control` `+` `29``) >``=` `2u`  `&& (``*``((_BYTE` `*``)WPP_GLOBAL_Control` `+` `32``) &` `1``) !``=` `0`  `&& KeGetCurrentIrql() <` `2u` `)`  `{`  `DbgPrint(``"SrvOs2FeaListToNt: Unable to allocate %d bytes from nonpaged pool."``,` `*``BufferLength);`  `DbgPrint(``"\n"``);`  `}`  `return` `0xC0000205``;` `/``/` `STATUS_INSUFF_SERVER_RESOURCES`  `}` |

在`SrvOs2FeaListSizeToNt`函数中，实现了两个功能，一是计算`ntfealist`结构体的长度并用于申请后续空间，二是对`fealist`结构的长度进行重新赋值，防止由于该长度被用户输入控制导致错误，在实现第二个功能的时候，由于`trans2`消息的总长度为两个字节，所以在此处进行了强转导致了最终长度计算出错。

|  |  |
| --- | --- |
