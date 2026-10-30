
用来创建一个WNF对象，该函数会在0环创建一个`WNF_NAME_INSTANCE`对象，大小为`0xb8`(`WNF_NAME_INSTANCE` + `POOL_HEADER`)。

![图片描述](upload/attach/202201/848410_HNFMW5E6QWSQDV2.png)

（`NtCreateWnfStateName`函数片段）

### `NtUpdateWnfStateData`

```
typedef NTSTATUS (NTAPI * __NtUpdateWnfStateData)(
_In_ PWNF_STATE_NAME StateName,
_In_reads_bytes_opt_(Length) const VOID * Buffer,
_In_opt_ ULONG Length,
_In_opt_ PCWNF_TYPE_ID TypeId,
_In_opt_ const PVOID ExplicitScope,
_In_ WNF_CHANGE_STAMP MatchingChangeStamp,
_In_ ULONG CheckStamp);
```

更新WNF StateData，当`Length`小于`StateData->AllocateSize`时会根据`Length`大小来分配内核池，否则会将`Buffer`中的数据拷贝到内核池中。

```
if (!v12 && (a1->PermanentDataStore || (_DWORD)v6) || (v13 = v12) != 0i64 && v12->AllocatedSize < (unsigned int)v6)
{
......
if (((*(_DWORD *)&a1->StateName >> 4) & 3) != 3 || PsInitialSystemProcess == (PEPROCESS)a1->CreatorProcess)
{
v21 = (_WNF_STATE_DATA *)ExAllocatePoolWithTag(PagedPool, (unsigned int)(v6 + 16), 0x20666E57u);
v25 = v21;
}
else
{
......
v21 = (_WNF_STATE_DATA *)ExAllocatePoolWithQuotaTag((POOL_TYPE)9, (unsigned int)(v6 + 16), 0x20666E57u);
......
}
......
memmove(&v13[1], v7, v6);
v13->DataSize = v6;
v13->ChangeStamp = i;
v15 = a1->PermanentDataStore;
......
（NtUpdateWnfStateData函数片段）
```

### `NtQueryWnfStateData`

```
typedef NTSTATUS (NTAPI * __NtQueryWnfStateData)(
_In_ PWNF_STATE_NAME StateName,
_In_opt_ PWNF_TYPE_ID TypeId,
_In_opt_ const VOID * ExplicitScope,
_Out_ PWNF_CHANGE_STAMP ChangeStamp,
_Out_writes_bytes_to_opt_(*BufferSize, *BufferSize) PVOID Buffer,
_Inout_ PULONG BufferSize);
```

查询指定`stateName`对应的`stateData`， 当`BufferSize`小于`StateData->DataSize`时，该函数会调用失败，并返回`C0000023`。

```
*a2 = v11->ChangeStamp;
*a5 = v11->DataSize;
v12 = v11->DataSize;
if ( a4 < v12 )
{
v14 = 0xC0000023;
}
else
{
memmove(a3, &v11[1], v12);
v14 = 0;
}
（NtQueryWnfStateData函数片段）
```

## 利用思路

### 相对内存读写

进行堆喷射，在0环中造成以下的内存布局。

![图片描述](upload/attach/202201/848410_BG3TWXX8UU49PU8.png)

利用`Ntfs` Chunk覆盖`StateData`中的`DataSize`成员，后续就可以使用`NtQueryWnfStateData` API来读取`NAME INSTACE`对象中的内容。覆盖`StateData`中的`AllocateSize`成员，后续就可以使用`NtUpDateWnfStateData` API来修改`NAME INSTACE`对象中的内容。

### 任意内存读写

利用 State Data Chunk来覆盖Name Instance chunk中的`StateData`指针，后续使用`NtQueryWnfStateData`和`NtUpDateWnfStateData` API来造成任意地址的读写（需要构造`AllocateSize`和`DataSize`成员）。

### 利用过程

1. 按照如上所示进行内核池布局。
2. 利用相对内存读取，读取`NAME INSTACE`对象中的内容。（`NAME INSTANCE`对象中有两个比较重要的成员`StateName`和`CreatorProcess`，前者由于所有的`NAME INSTANCE`对象都保存在一个排序二叉树中，破坏了`StateName`成员会导致系统无法找到相对应的`NAME INSTANCE`对象，并且在进行喷射后我们也无法确定究竟是哪一个`StateName`对应的对象发生了溢出，所以通过该方法可以准确定位到发生了溢出的`NAME INSTANCE`对象。后者标识了当前进程的`Eprocess`对象，可以通过该对象来遍历所有进程的`Eprocess`结构）。
3. 利用相对内存写入，修改`NAME INSTANCE`对象中的`StateData`成员为`CreatorProcess`（需要注意`DataSize`成员，该成员直接影响读取的字节数和是否能够成功读取）。
4. 利用任意内存读取遍历系统进程。
5. 找到对应的系统进程后，利用任意内存读取获取系统进程的`token`。
6. 利用任意内存写入，修改当前进程的`token`（需要注意`AllocateSize`成员）。

## 效果演示

![图片描述](upload/attach/202201/848410_T6RKHUAB5RWDVQ4.png)

## EXP

EXP已在`Github`上开源，目前只在`win 1903`上经过测试，稳定性大概在`80%`。

<a href="elink@b3aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6S2j5i4A6Z5N6h3I4A6j5h3&6Y4i4K6u0r3b7#2k6q4i4K6u0V1x3U0l9J5x3g2)9J5k6o6x3I4z5e0f1$3i4K6u0V1c8g2S2b7">github传送门</a>

## 参考文章

<a href="elink@eb6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6J5k6i4y4W2j5i4u0U0K9q4)9J5k6h3&6U0j5$3N6J5L8%4g2H3i4K6u0W2j5$3!0E0i4K6u0r3x3U0l9J5x3g2)9J5c8U0l9%4i4K6u0r3x3e0g2Q4x3V1k6U0N6X3g2Q4x3X3b7J5x3o6t1I4i4K6u0V1x3K6p5&6y4e0k6Q4x3X3c8W2P5s2m8D9L8$3W2@1K9h3&6Y4i4K6u0V1N6r3S2W2i4K6u0V1N6$3W2F1k6r3!0%4M7#2)9J5k6r3E0W2M7X3&6W2L8q4)9J5k6r3&6@1k6Y4y4Q4x3X3c8%4K9i4c8Z5i4K6u0V1N6$3&6X3i4K6u0V1M7r3q4J5N6q4)9J5k6o6q4Q4x3V1j5`.">CVE-2021-31956</a>

# CVE-2021-31956

## 漏洞概括

`CVE-2021-31956`是由`Windows Ntfs`组件系统存在整形溢出所导致，该漏洞可导致本地权限提升。

## 漏洞复现环境

![图片描述](https://bbs.kanxue.com/upload/attach/202201/848410_EEJTFNYZVGNF26S.png)

## 漏洞成因

该漏洞发生在`ntfs.sys`中的`NtfsQueryEaUserEaList`函数中

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28  29  30  31  32  33  34  35  36  37  38  39  40  41  42  43  44  45  46  47  48  49  50  51  52  53  54  55  56  57  58  59  60  61  62  63  64  65  66  67  68  69  70  71  72 | `_QWORD` `*``__fastcall NtfsQueryEaUserEaList(_QWORD` `*``a1,`  `FILE_FULL_EA_INFORMATION` `*``CurrentEas,`  `__int64 a3, __int64 PEaBuffer,`  `unsigned` `int` `UserBufferLength,`  `FILE_GET_EA_INFORMATION` `*``pUserEaList,`  `char a7)`  `{`  `. . . . . .`  `while` `(` `1` `)`  `{`  `/``/` `索引ealist中的成员，用作下面的查找。`  `v11` `=` `(FILE_GET_EA_INFORMATION` `*``)((char` `*``)pUserEaList` `+` `v9);`  `*``(_QWORD` `*``)&DestinationString.Length` `=` `0i64``;`  `DestinationString.``Buffer` `=` `0i64``;`  `*``(_QWORD` `*``)&SourceString.Length` `=` `0i64``;`  `SourceString.``Buffer` `=` `0i64``;`  `*``(_QWORD` `*``)&DestinationString.Length` `=` `v11``-``>EaNameLength;`  `DestinationString.MaximumLength` `=` `DestinationString.Length;`  `DestinationString.``Buffer` `=` `v11``-``>EaName;`  `RtlUpperString(&DestinationString, &DestinationString);`  `/``/` `检查ealist中成员的name是否有效`  `if` `( !(unsigned __int8)NtfsIsEaNameValid(&DestinationString) )`  `break``;`  `v12` `=` `v11``-``>NextEntryOffset;`  `v13` `=` `v11``-``>EaNameLength;`  `v22` `=` `v11``-``>NextEntryOffset` `+` `v9;`  `/``/` `遍历查询的EaList`  `for` `( curEaList` `=` `pUserEaList; ; curEaList` `=` `(FILE_GET_EA_INFORMATION` `*``)((char` `*``)curEaList`  `+` `curEaList``-``>NextEntryOffset) )`  `{`  `if` `( curEaList` `=``=` `v11 )`  `{`    `v15` `=` `offset;`  `/``/` `v16 分配的内核池`  `v16` `=` `(_DWORD` `*``)(PEaBuffer` `+` `padding` `+` `offset);`      `/``/` `根据name查找对应的Ea信息`  `if` `( NtfsLocateEaByName((__int64)CurrentEas,` `*``(_DWORD` `*``)(a3` `+` `4``), &DestinationString, &FeaOffset) )`  `{`  `ea_block` `=` `(FILE_FULL_EA_INFORMATION` `*``)((char` `*``)CurrentEas` `+` `FeaOffset);`  `/``/` `计算内存拷贝大小`  `RawEaSize` `=` `ea_block``-``>EaValueLength` `+` `ea_block``-``>EaNameLength` `+` `9``;`  `/``/``防溢出检查`  `if` `( RawEaSize <``=` `UserBufferLength` `-` `padding )`  `{`  `/``/``溢出点`  `memmove(v16, ea_block, RawEaSize);`  `*``v16` `=` `0``;`  `goto LABEL_8;`  `}`  `}`  `. . . . .`  `if` `( !a7 )`  `{`  `if` `( v24 )`  `*``v24` `=` `(_DWORD)v16` `-` `(_DWORD)v24;`  `/``/``判断是ealist中是否还有其他成员`  `if` `( v11``-``>NextEntryOffset )`  `{`  `v24` `=` `v16;`  `/``/` `总长度减去已经拷贝的长度`  `UserBufferLength` `-``=` `RawEaSize` `+` `padding;`  `/``/``padding的计算`  `padding` `=` `((RawEaSize` `+` `3``) &` `0xFFFFFFFC``)` `-` `RawEaSize;`  `goto LABEL_26;`  `}`  `}`  `. . . . .`  `}`  `(NtfsQueryEaUserEaList函数片段)` |

上面的代码片段在循环遍历文件中的每个EA拓展属性，并将其拷贝到堆中，每次拷贝的大小为`ea_block->EaValueLenght` + `ea_blocal->EaNameLength` + `9`。其中`ea_block`的结构如下：

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7 | `typedef struct _FILE_FULL_EA_INFORMATION {`  `ULONG  NextEntryOffset;` `/``/``下一个同类型结构的偏移，若是左后一个则为``0``。`  `UCHAR  Flags;`  `UCHAR  EaNameLength;` `/``/``eanam数组的长度，不包含``0``终止字符。`  `USHORT EaValueLength;` `/``/``数组中每个ea值的长度`  `CHAR   EaName[``1``];`  `} FILE_FULL_EA_INFORMATION,` `*``PFILE_FULL_EA_INFORMATION;` |

在每次拷贝前有一个判断溢出的检查（`RawEaSize <= UserBufferLength - padding`），`UserBufferLength`是由参数传入并在每次循环中递减，`padding`由（`padding = ((RawEaSize + 3) & 0xFFFFFFFC) - RawEaSize`）计算而来，该表达式只会存在4个结果（`0, 1, 2,3`）。

内存拷贝的目的地址（`v16`）由参数传入，该参数是在`NtfsCommonQueryEa`函数中分配的内核池。

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21 | `. . . . .`  `IrpSp` `=` `IoGetCurrentIrpStackLocation( Irp );`  `. . . . .`  `UserBufferLength` `=` `IrpSp``-``>Parameters.QueryEa.Length;`  `. . . . .`  `if` `(` `*``(_BYTE` `*``)(a2` `+` `64``) )`  `{`  `v34` `=` `v14;`  `v4` `=` `ExAllocatePoolWithTag((POOL_TYPE)``17``, UserBufferLength,` `0x4546744Eu``);`  `v28` `=` `v4;`  `v24` `=` `1``;`  `}`  `memset(v4,` `0``, v10);`  `. . . . . .`    `if` `( v33 )`  `{`  `v15` `=` `NtfsQueryEaUserEaList(&v33, v30, (__int64)v27, (__int64)v4, v10, v33, v39);`  `}`  `. . . . . .`  `（NtfsCommonQueryEa函数片段）` |
