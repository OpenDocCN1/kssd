
```c
_QWORD * __fastcall NtfsQueryEaUserEaList(_QWORD *a1, _FILE_FULL_EA_INFORMATION *ea_blocks_for_file, __int64 out_buf, __int64 a4, unsigned int out_buf_length, _FILE_GET_EA_INFORMATION *eaList, char a7)
{
int v8; // edi
unsigned int v9; // ebx
unsigned int padding; // er15
_FILE_GET_EA_INFORMATION *GetEa; // r12
ULONG v12; // er14
unsigned __int8 v13; // r13
_FILE_GET_EA_INFORMATION *curEaList; // rbx
unsigned int v15; // ebx
_DWORD *v16; // r13
unsigned int ea_block_size; // er14
unsigned int v18; // ebx
_FILE_FULL_EA_INFORMATION *ea_block; // rdx
char v21; // al
ULONG v22; // [rsp+20h] [rbp-38h]
unsigned int v23; // [rsp+24h] [rbp-34h] BYREF
_DWORD *v24; // [rsp+28h] [rbp-30h]
struct _STRING DestinationString; // [rsp+30h] [rbp-28h] BYREF
STRING SourceString; // [rsp+40h] [rbp-18h] BYREF
unsigned int offest; // [rsp+A0h] [rbp+48h]
v8 = 0;
*a1 = 0i64;
v24 = 0i64;
v9 = 0;
offest = 0;
padding = 0;
a1[1] = 0i64;
while ( 1 )
{ // 索引ealist中的成员，用作下面的查找。
GetEa = (_FILE_GET_EA_INFORMATION *)((char *)eaList + v9);
*(_QWORD *)&DestinationString.Length = 0i64;
DestinationString.Buffer = 0i64;
*(_QWORD *)&SourceString.Length = 0i64;
SourceString.Buffer = 0i64;
*(_QWORD *)&DestinationString.Length = GetEa->EaNameLength;
DestinationString.MaximumLength = DestinationString.Length;
DestinationString.Buffer = GetEa->EaName;
RtlUpperString(&DestinationString, &DestinationString);
if ( !(unsigned __int8)NtfsIsEaNameValid(&DestinationString) )// 检查ealist中成员的name是否有效
break;
v12 = GetEa->NextEntryOffset;
v13 = GetEa->EaNameLength;
v22 = GetEa->NextEntryOffset + v9;
for ( curEaList = eaList; ; curEaList = (_FILE_GET_EA_INFORMATION *)((char *)curEaList + curEaList->NextEntryOffset) )// 遍历查询的EaList
{
if ( curEaList == GetEa )
{
v15 = offest;
v16 = (_DWORD *)(a4 + padding + offest);
if ( (unsigned __int8)NtfsLocateEaByName(// 根据name查找对应的Ea信息
ea_blocks_for_file,
*(unsigned int *)(out_buf + 4),
&DestinationString,
&v23) )
{
ea_block = (_FILE_FULL_EA_INFORMATION *)((char *)ea_blocks_for_file + v23);
ea_block_size = ea_block->EaValueLength + ea_block->EaNameLength + 9;// 计算内存拷贝大小
if ( ea_block_size <= out_buf_length - padding )// 防溢出检查
// 两个uint32相减以后发生整数溢出绕过检查
{
memmove(v16, ea_block, ea_block_size);// 溢出点
*v16 = 0;
goto LABEL_8;
}
}
else
{
ea_block_size = GetEa->EaNameLength + 9;// 9=4(next entry offset)+1(flags)+1(name length)+2(value length)+1(null byte)
if ( ea_block_size + padding <= out_buf_length )
{
*v16 = 0;
*((_BYTE *)v16 + 4) = 0;
*((_BYTE *)v16 + 5) = GetEa->EaNameLength;
*((_WORD *)v16 + 3) = 0;
memmove(v16 + 2, GetEa->EaName, GetEa->EaNameLength);
SourceString.Length = DestinationString.Length;
SourceString.MaximumLength = DestinationString.Length;
SourceString.Buffer = (PCHAR)(v16 + 2);
RtlUpperString(&SourceString, &SourceString);
v15 = offest;
*((_BYTE *)v16 + GetEa->EaNameLength + 8) = 0;
LABEL_8:
v18 = ea_block_size + padding + v15;
offest = v18;
if ( !a7 )
{
if ( v24 )
*v24 = (_DWORD)v16 - (_DWORD)v24;
if ( GetEa->NextEntryOffset ) // 判断是ealist中是否还有其他成员
{
v24 = v16;
out_buf_length -= ea_block_size + padding;// 总长度减去已经拷贝的长度
padding = ((ea_block_size + 3) & 0xFFFFFFFC) - ea_block_size;// padding的计算
goto LABEL_26;
}
}
LABEL_12:
a1[1] = v18;
LABEL_13:
*(_DWORD *)a1 = v8;
return a1;
}
}
v21 = NtfsStatusDebugFlags;
a1[1] = 0i64;
if ( v21 )
NtfsStatusTraceAndDebugInternal(0i64, 2147483653i64, 919406i64);
v8 = -2147483643;
goto LABEL_13;
}
if ( v13 == curEaList->EaNameLength && !memcmp(GetEa->EaName, curEaList->EaName, v13) )
break;
}
if ( !v12 )
{
v18 = offest;
goto LABEL_12;
}
LABEL_26:
v9 = v22;
}
a1[1] = v9;
if ( NtfsStatusDebugFlags )
NtfsStatusTraceAndDebugInternal(0i64, 2147483667i64, 919230i64);
*(_DWORD *)a1 = -2147483629;
return a1;
}
```

`NtfsQueryEaUserEaList`从 循环遍历文件的每个 NTFS 扩展属性 (Ea)，并根据`ea_block->EaValueLength + ea_block->EaNameLength + 9`的大小从 Ea 块复制到输出缓冲区。

有一个检查确保`ea_block_size`小于或等于`out_buf_length - padding`。然后，`out_buf_length`会减去`ea_block_size`及其填充的大小。填充是通过`((ea_block_size + 3) 0xFFFFFFFC) - ea_block_size`来计算的。因为每个EA块应该填充为32位对齐。

假设文件的扩展属性中有两个扩展属性

## 正常情况下：

第一次迭代

```
EaNameLength = 5
EaValueLength = 4
ea_block_size = 9 + 5 + 4 = 18
padding = 0
```

因此18 < `out_buf_length - 0`，数据将被复制到缓冲区中

第二次迭代

```
out_buf_length = 30 - 18 + 0
out_buf_length = 12 // we would have 12 bytes left of the output buffer.
padding = ((18+3) 0xFFFFFFFC) - 18
padding = 2
```

在文件中添加一个具有相同值的第二个扩展属性。

```
EaNameLength = 5
EaValueLength = 4
ea_block_size = 9 + 5 + 4 = 18
18 <= 12 - 2 // is False.
```

由于缓冲区太小，第二次内存复制将不会发生。

## 整数溢出

```
第一个扩展属性：
EaNameLength = 5
EaValueLength = 4
第二个扩展属性：
EaNameLength = 5
EaValueLength = 47
```

第一次迭代

```
EaNameLength = 5
EaValueLength = 4
ea_block_size = 9 + 5 + 4 // 18
padding = 0
```

检查结果为:

18 <= 18 - 0 // is True and a copy of 18 occurs.

第二个扩展属性具有以下值：

```
EaNameLength = 5
EaValueLength = 47
ea_block_size = 5 + 47 + 9
ea_block_size = 137
```

结果检查将是：

`ea_block_size <= out_buf_length - padding`

137 <= 0 - 2

发生下溢，137 个字节将被复制到缓冲区末尾，从而损坏相邻内存。

查看`NtfsQueryEaUserEaList`函数的调用者`NtfsCommonQueryEa`，我们可以看到输出缓冲区是根据请求的大小在分页池上分配的

![图片描述](upload/attach/202306/865065_54U5PND6625M28W.png)

`NtfsCommonQueryEa`函数可通过`ZwQueryEaFIle`函数调用

```
// 为文件创建EA
NTSTATUS ZwSetEaFile(
[in] HANDLE FileHandle,
[out] PIO_STATUS_BLOCK IoStatusBlock,
[in] PVOID Buffer,
[in] ULONG Length
);
// 查询文件EA
NTSTATUS ZwQueryEaFile(
[in] HANDLE FileHandle, //文件句柄
[out] PIO_STATUS_BLOCK IoStatusBlock,
[out] PVOID Buffer, //扩展属性缓冲区（FILE_FULL_EA_INFORMATION结构）
[in] ULONG Length, //缓冲区大小
[in] BOOLEAN ReturnSingleEntry,
[in, optional] PVOID EaList, //指定需要查询的扩展属性
[in] ULONG EaListLength,
[in, optional] PULONG EaIndex, //指定需要查询的起始索引
[in] BOOLEAN RestartScan
);
```

可以看到输出缓冲区`Buffer`以及该缓冲区的长度都是从用户空间传入的。这意味着我们根据缓冲区的大小控制内核空间的内存分配。

该漏洞对攻击者来说：

溢出拷贝时数据和大小均可控。

可以覆盖下一个内核池块

内核池分配时大小可控，并且可以进行堆布局。

## 触发漏洞

Windows10引入了新的方式进行堆块管理，称为Segment Heap，具体可看以下论文

<mark class="encrypted">bceK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2K6M7%4c8A6j5#2)9J5k6h3!0J5k6#2)9J5c8X3#2W2k6r3W2S2i4K6u0r3f1#2y4f1d9f1x3J5x3o6t1H3i4K6u0r3f1#2y4f1d9f1y4Q4x3X3c8S2j5%4c8W2M7#2)9J5c8Y4m8G2L8$3I4Q4y4h3k6G2N6X3g2J5k6X3I4G2N6#2)9#2k6X3g2^5M7r3I4G2K9i4c8S2N6r3W2G2L8W2)9#2k6Y4y4A6L8X3y4W2i4K6g2X3N6$3W2F1k6r3!0%4M7#2)9#2k6U0p5H3i4K6g2X3x3e0W2Z5x3g2)9J5c8W2y4e0g2p5W2o6x3U0l9J5x3q4)9J5k6p5q4J5N6r3W2U0L8r3g2Q4x3X3c8H3L8$3!0D9i4K6g2X3L8%4k6W2M7X3k6D9L8%4N6Q4y4h3k6W2P5s2m8D9L8$3W2@1j5i4c8A6L8$3&6Q4y4h3k6K6K9h3&6U0k6g2)9#2k6Y4N6A6L8X3c8G2N6%4y4Q4y4h3j5I4x3q4)9#2k6U0p5&6K9o6q4Q4x3X3c8T1j5i4W2W2N6q4)9#2k6X3k6S2M7X3W2W2L8r3I4G2i4K6u0W2M7r3c8X3</mark>

EXP:<mark class="encrypted">b6dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6S2j5i4A6Z5N6h3I4A6j5h3&6Y4i4K6u0r3b7#2k6q4i4K6u0V1x3U0l9J5x3g2)9J5k6o6x3I4z5e0f1$3i4K6u0V1c8g2S2b7</mark>

仿照EXP改的可以触发溢出的POC
