
```
1: kd> bp afd!AfdTransmitFile
1: kd> g
Breakpoint 0 hit
afd!AfdTransmitFile:
8db6c31e 6884000000 push 84h
1: kd> r edx
edx=89689b4c
1: kd> dt \_IO\_STACK\_LOCATION 89689b4c
ntdll!\_IO\_STACK\_LOCATION
+0x018 FileObject : 0x89a33ab0 \_FILE\_OBJECT
1: kd> dt \_FILE\_OBJECT 0x89a33ab0
ntdll!\_FILE\_OBJECT
+0x00c FsContext : 0x877fcbc0 Void
1: kd> dd 0x877fcbc0
877fcbc0 0004afd2 00000000 00000200 89a3f658
```

这里得到`AFD2`， 明显与 `1AFD`不同， 所以这个跳转指令生效 ，继续分析

![](upload/attach/202205/930159\_UXU7GWRVVQPNRPR.png)

这里又有判断跳转，动态查看一下参数

```
dd 89689b4c+0x8
89689b54 00000030 0001207f 00d63390 8751c4a0
```

这里得到`30`，所以跳转生效，继续分析，又遇到两个跳转

![](upload/attach/202205/930159\_8JF65K29SVB4BK7.png)

动态查看参数，确定流程

```
1: kd> r ecx
ecx=89689a70
1: kd> dt \_IRP 89689a70
ntdll!\_IRP
+0x000 Type : 0n6
+0x002 Size : 0x190
+0x004 MdlAddress : (null)
+0x008 Flags : 0x60000
+0x00c AssociatedIrp : <unnamed-tag>
+0x010 ThreadListEntry : \_LIST\_ENTRY [ 0x897fe7fc - 0x897fe7fc ]
+0x018 IoStatus : \_IO\_STATUS\_BLOCK
+0x020 RequestorMode : 1 ''
```

这里可以得到 `mode`值为`1`，所以跳转未生效 ，继续查看下一个跳转

```
1: kd> dt \_IO\_STACK\_LOCATION 89689b4c
ntdll!\_IO\_STACK\_LOCATION
+0x000 MajorFunction : 0xe ''
+0x001 MinorFunction : 0x1f ''
+0x002 Flags : 0x5 ''
+0x003 Control : 0 ''
+0x004 Parameters : <unnamed-tag>
1: kd> dd 89689b4c + 0x10
89689b5c 00d63390 8751c4a0 89a33ab0 00000000
```

这里得到 `eax+10` 也就是 `Type3InputBuffer`的值 低4位为`0` ， 所以跳转生效，继续分析

这里提一下，为什么说是判断是否4字节对齐，看后面调用的参数 ，根据微软的解释：`**ExRaiseDatatypeMisalignment**` 例程可用于结构化异常处理，为驱动程序处理 I/O 请求时发生的不对齐数据类型引发驱动程序确定的异常。

好的，继续分析，这里又遇到跳转，可以单步到这个位置看一下这个全局变量的值 值为`0x7FFF000`

![](upload/attach/202205/930159\_7ZVVDRFF8JPBNGN.png)

![](upload/attach/202205/930159\_RWRTRASS986NYWQ.png)

分析到这里，有三个关键的跳转，如下

```c
InputBufferLength >= 0x30;
InputBuffer & 0x3 == 0;
InputBuffer < 0x7FFF000;
```

然后继续往下分析，遇到两个跳转，动态跟踪 `ecx = 1` ，所以不跳转 ， 动态跟踪 `ebp-80 = 0` ， 跳转生效

![](upload/attach/202205/930159\_9XNDWHEV9D5XXU9.png)

```
afd!AfdTransmitFile+0x90:
8db6c3ae f7c1c8ffffff test ecx,0FFFFFFC8h
1: kd> r ecx
ecx=00000001
```

![](upload/attach/202205/930159\_8Z2ZPWYPHU7C83T.png)

好的，继续往下分析，这里遇到两个跳转，动态跟踪 `Afd`全局变量的值为 `0x10`

![](upload/attach/202205/930159\_F2N2KMQGYBUGYWT.png)

![](upload/attach/202205/930159\_ES953T3N7QYW6WZ.png)

`edx+8` 的值为 `200`

```
1: kd> dd 0x877fcbc0
877fcbc0 0004afd2 00000000 00000200 89a3f658
```

之后调用了 `AfdTliGetTpInfo`函数 ，`ecx`为参数 值为`3` ，也就是 `AfdTliGetTpInfo(3)` 的调用

通过之前的堆栈调用，可以得知，这个函数在第二次`free`的时候被调用，所以这个函数也需要重点分析一下

## 3.逆向分析AfdTliGetTpInfo

![](upload/attach/202205/930159\_QV7QMRUKRDJHSPR.png)

首先看到一个函数调用，比较陌生，不着急，一步一步来，根据书籍提示 查看 《使用 Lookaside List 分配内存》 这篇文章

得到函数原型与实现如下

```c
PVOID p = ExAllocateFromNPagedLookasideList(lookaside);//从 non-paged 链节点里分配内存，返回内存块到 p
PVOID ExAllocateFromNPagedLookasideList(PNPAGED\_LOOKASIDE\_LIST Lookaside)
{
PSLIST\_ENTRY ListEntry;
// TotalAllocates 记录分配的次数，包括成功和失败
Lookaside->L.TotalAllocates++;
// 从节点里读取内存块链
ListEntry = ExInterlockedPopEntrySList(&Lookaside->L.ListHead);
// 分配内存有两个情形：
// 1. 返回 lookaside list 节点（SLIST\_ENTRY）
// 2. 返回内存，假如 lookaside list 为空
if (ListEntry == NULL)
{
// AllocateMisses 记录从 lookaside list 中分配失败的次数 也就是从pool中分配的次数
Lookaside->L.AllocateMisses++;
// 调用 lookaside list 节点中的 Allocate() 例程
return Lookaside->L.Allocate(Lookaside->L.Type,
Lookaside->L.Size,
Lookaside->L.Tag);
}
else
return ListEntry;
}
```

同时得到一些信息，代码使用 `ExInterlockedPopEntrySList()` 例程来读取节点内存块地址 ，如果`lookaside list` 为空， 则调用

`Lookaside`结构中的默认分配内存例程 `Allocate`进行分配内存，否则直接返回`lookaside list`节点

继续查看得知`Allocate`的定义

`PALLOCATE_FUNCTION Allocate` 自定义的分配函数，缺省使用 `ExAllocatePoolWithTag()`

那么现在需要去跟踪一下，程序走了哪个流程

![](upload/attach/202205/930159\_VG8D5UXER9YR2XB.png)

这里可以看到，返回值为`0`，也就是程序进入了 自定义的分配内存函数，继续跟踪查看函数与参数

![](upload/attach/202205/930159\_2VRXAG7XPHFU33H.png)

可以看到这里调用了 `AfdAllocateTpInfo`，参数分别为 `0` ， `108` ， `c6646641` ， 根据`Lookaside`结构可知参数的意义

```c
typdef struct \_NPAGED\_LOOKASIDE\_LIST
{
GENERAL\_LOOKASIDE L;
KSPIN\_LOCK Lock;
} NPAGED\_LOOKASIDE\_LIST, \*PNPAGED\_LOOKASIDE\_LIST;
typedef struct \_GENERAL\_LOOKASIDE {
SLIST\_HEADER ListHead; // 单向链表，用来挂接 pool block
USHORT Depth; // lookaside 的 depth，缺省为 4
USHORT MaximumDepth; // maximum depth，缺省为 256
ULONG TotalAllocates; // 记录进行多少次分配
ULONG AllocateMisses; // 记录分配从 lookaside list 中分配失败次数
ULONG TotalFrees; // 记录进行多少次释放
ULONG FreeMisses; // 记录从 lookaside list 中释放 miss 次数
POOL\_TYPE Type; // pool type 类型，与 ExAllocatePoolWithTag() 参数一致
ULONG Tag; // 用于 ExAllocatePoolWithTag()
ULONG Size; // 用于 ExAllocatePoolWithTag()
PALLOCATE\_FUNCTION Allocate; // 自定义的分配函数，缺省使用 ExAllocatePoolWithTag()
PFREE\_FUNCTION Free; // 自定义的释放函数，缺省使用 ExFreePool()
LIST\_ENTRY ListEntry; // 双向 link list
ULONG LastTotalAllocates; // 记录上一次 Allocate 次数
ULONG LastAllocateMisses; // 记录上一次 Allocate miss 次数
ULONG Future[2]; // 保留
} GENERAL\_LOOKASIDE, \*PGENERAL\_LOOKASIDE;
```

调用函数与参数 `Lookaside->L.Allocate(Lookaside->L.Type, Lookaside->L.Size, Lookaside->L.Tag)`

这里可以得知 `0`为`pool type` 类型 ，也就是非分页内存 ， `0x108`为分配的`size` ， `c6646641` 为 `tag`

所以这里可以得到 `AfdAllocateTpInfo` 的调用模型 ，返回值通过函数名，推测为`TpInfo`的指针

```c
TpInfo\* tpinfo = AfdAllocateTpInfo(POOL\_TYPE PoolType, SIZE\_T NumberOfBytes, ULONG Tag);
```

那么事情真的是这样吗，去`AfdAllocateTpInfo` 中逆向分析一波

## 4.逆向分析AfdAllocateTpInfo

这个函数比较短，以下是它的反编译代码

```c
PVOID \_\_stdcall AfdAllocateTpInfo(POOL\_TYPE PoolType, SIZE\_T NumberOfBytes, ULONG Tag)
{
PVOID TpInfo; // esi
TpInfo = ExAllocatePoolWithTagPriority(0, 0x108, 0xc6646641 , 0);
//首先分配一块内存，参数还是之前那三个，第四个优先级不用管
if ( TpInfo )
AfdInitializeTpInfo(TpInfo, AfdDefaultTpInfoElementCount, AfdTdiStackSize, 1);
//调用AfdInitializeTpInfo 初始化这个结构体
return TpInfo; //返回这个结构体的指针
}
```

得到的信息 `AfdDefaultTpInfoElementCount = 3`

初始化`tpinfo`的函数先不看，需要的时候再来分析 ，接下来返回`AfdTliGetTpInfo`继续分析

下边为`ExAllocateFromNPagedLookasideLif`返回的数据 ，留着备用

```
1: kd> dd 896540c8 L70
896540c8 00000000 00000000 00000000 00000000
896540d8 00000000 00000000 00000000 00000000
896540e8 89654158 00000000 00000000 00000000
896540f8 00000002 00000000 00000000 00000000
89654108 00000000 00000000 00000000 00000000
89654118 00000000 00000000 00000000 00000000
89654128 00000000 00000000 00000000 00000000
89654138 896541a0 896541b8 00000000 00000000
89654148 00000000 00000000 00000000 00000000
89654158 899653a8 8977dd48 8977a360 8977dd40
89654168 89654170 02010000 88920b90 88920b90
89654178 8977a360 88920b88 89654188 02010001
89654188 8977aa28 8977aa28 8977a360 8977aa20
89654198 896541a0 02010002 88bc0000 88bc1c20
896541a8 896540c8 88bc1c18 00000000 00000000
896541b8 88b00001 88b0da08 896540c8 88b0da00
896541c8 00000000 00000000
```

## 5.继续分析AfdTliGetTpInfo

```c
DWORD \*\_\_fastcall AfdTliGetTpInfo(unsigned int a1)
{
\_DWORD \*tpinfo; // eax
\_DWORD \*tpinfo1; // esi
tpinfo = ExAllocateFromNPagedLookasideList((PNPAGED\_LOOKASIDE\_LIST)&AfdGlobalData[6].NumberOfSharedWaiters);
tpinfo1 = tpinfo;
if ( !tpinfo )
return 0;
tpinfo[2] = 0;
tpinfo[3] = 0;
tpinfo[4] = tpinfo + 3;
tpinfo[5] = 0;
tpinfo[6] = tpinfo + 5;
tpinfo[13] = 0;
\*((\_BYTE \*)tpinfo + 51) = 0;
tpinfo[9] = 0;
tpinfo[11] = -1;
tpinfo[15] = 0;
tpinfo[1] = 0;
if ( a1 > AfdDefaultTpInfoElementCount ) //这里 a1为上层传进来的3 AfdDefaultTpInfoElementCount = 3
{ //所以没有进if
tpinfo[8] = ExAllocatePoolWithQuotaTag((POOL\_TYPE)16, 0x18 \* a1, 0xC6646641);
\*((\_BYTE \*)tpinfo1 + 50) = 1;
}
return tpinfo1; //返回一个局部变量tpinfo
}
```

下面为此函数返回的数据，留着备用

```
1: kd> dd 896540c8 l70
896540c8 00000000 00000000 00000000 00000000
896540d8 896540d4 00000000 896540dc 00000000
896540e8 89654158 00000000 00000000 ffffffff
896540f8 00000002 00000000 00000000 00000000
89654108 00000000 00000000 00000000 00000000
89654118 00000000 00000000 00000000 00000000
89654128 00000000 00000000 00000000 00000000
89654138 896541a0 896541b8 00000000 00000000
89654148 00000000 00000000 00000000 00000000
89654158 899653a8 8977dd48 8977a360 8977dd40
89654168 89654170 02010000 88920b90 88920b90
89654178 8977a360 88920b88 89654188 02010001
89654188 8977aa28 8977aa28 8977a360 8977aa20
89654198 896541a0 02010002 88bc0000 88bc1c20
896541a8 896540c8 88bc1c18 00000000 00000000
896541b8 88b00001 88b0da08 896540c8 88b0da00
896541c8 00000000 00000000
```
