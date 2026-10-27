
```
kd> rdmsr C0000082

msr[c0000082] = fffff800`03ebf780

    

kd> u fffff800`03ebf780

nt!KiSystemCall64:

fffff800`03ebf780 0f01f8          swapgs

fffff800`03ebf783 654889242510000000 mov   qword ptr gs:[10h],rsp

fffff800`03ebf78c 65488b2425a8010000 mov   rsp,qword ptr gs:[1A8h]

fffff800`03ebf795 6a2b            push    2Bh

fffff800`03ebf797 65ff342510000000 push    qword ptr gs:[10h]

fffff800`03ebf79f 4153            push    r11

fffff800`03ebf7a1 6a33            push    33h

fffff800`03ebf7a3 51              push    rcx



    

kd> uf fffff800`03ef3780

    .

    .

    fffff800`03ef3a72 4c8d1587be1f00  lea     r10,[nt!KeServiceDescriptorTable (fffff800`040ef900)]

	fffff800`03ef3a79 4c8d1d40bf1f00  lea     r11,[nt!KeServiceDescriptorTableShadow (fffff800`040ef9c0)]

    .

    .
```

//用下一行函数地址-起始地址，算出下一行偏移位761

```
kd> ?fffff800`03ef3a79-fffff800`03ef3780

Evaluate expression: 761 = 00000000`000002f9
```

## X64_SSDT表遍历

DriverMain.c

```
#include <ntifs.h>



typedef struct _KeServiceTable

{

	PLONG offsetTablel;

	ULONG64 number;

	ULONG64 funNumber;

	PUCHAR funArgTable;

}KeServiceTable,*PKeServiceTable;





void EnumFun()

{

	//1.获取KeServiceTable地址

	ULONG64 KiSystemFun = __readmsr(0XC0000082);

	//+761是KeServiceTable下一行地址

	PLONG offsetAddr = KiSystemFun + 761 - 4;

	LONG offset = *offsetAddr;

	PKeServiceTable serviceTable = (PKeServiceTable)(offset + KiSystemFun + 761);



	for (size_t i = 0; i < serviceTable->funNumber; i++)

	{

		LONG funoffset = serviceTable->offsetTablel[i] >> 4;

		LONG64 ssdtFun = funoffset + (LONG64)(serviceTable->offsetTablel);

		DbgPrintEx(77, 0, "[db]:%p\r\n", ssdtFun);

	}

}





VOID DriverUnload(PDRIVER_OBJECT pDriverObject)

{

	

}



NTSTATUS DriverEntry(PDRIVER_OBJECT pDriver, PUNICODE_STRING pReg)

{



	EnumFun();



	pDriver->DriverUnload = DriverUnload;



	return STATUS_SUCCESS;

}
```

### 9.8.X64全局句柄表

**Win7全局句柄表解析**

- 0级表每个元素占16个字节，总共可以存256个
- 查找`csrss.exe`进程，pid为416。
- `416/4 = 104 < 256`，说明在第0个表。索引为16进制`0x68`

```
kd> dq PspCidTable

fffff800`0406c988  fffff8a0`00004890 00000000`00000000

fffff800`0406c998  ffffffff`80000020 00000000`00000101

fffff800`0406c9a8  ffffffff`800002cc ffffffff`80000024

fffff800`0406c9b8  00000000`00000000 00000000`00000113

fffff800`0406c9c8  00000000`00000000 00000000`00000000

fffff800`0406c9d8  fffff800`03fd4650 00000000`00000000

fffff800`0406c9e8  00000000`00000000 00000000`00000000

fffff800`0406c9f8  00000000`00000000 00000000`00000008
```

```
kd> dt _HANDLE_TABLE fffff8a0`00004890

nt!_HANDLE_TABLE

   +0x000 TableCode        : 0xfffff8a0`010c0001

   +0x008 QuotaProcess     : (null) 

   +0x010 UniqueProcessId  : (null) 

   +0x018 HandleLock       : _EX_PUSH_LOCK

   +0x020 HandleTableList  : _LIST_ENTRY [ 0xfffff8a0`000048b0 - 0xfffff8a0`000048b0 ]

   +0x030 HandleContentionEvent : _EX_PUSH_LOCK

   +0x038 DebugInfo        : (null) 

   +0x040 ExtraInfoPages   : 0n0

   +0x044 Flags            : 1

   +0x044 StrictFIFO       : 0y1

   +0x048 FirstFreeHandle  : 0x5cc

   +0x050 LastFreeHandleEntry : 0xfffff8a0`0195f630 _HANDLE_TABLE_ENTRY

   +0x058 HandleCount      : 0x2b7

   +0x05c NextHandleNeedingPool : 0xc00

   +0x060 HandleCountHighWatermark : 0x2d9
```

```
kd> dq 0xfffff8a0`010c0000

fffff8a0`010c0000  fffff8a0`00005000 fffff8a0`010c1000

fffff8a0`010c0010  fffff8a0`0195f000 00000000`00000000

fffff8a0`010c0020  00000000`00000000 00000000`00000000

fffff8a0`010c0030  00000000`00000000 00000000`00000000

fffff8a0`010c0040  00000000`00000000 00000000`00000000

fffff8a0`010c0050  00000000`00000000 00000000`00000000

fffff8a0`010c0060  00000000`00000000 00000000`00000000

fffff8a0`010c0070  00000000`00000000 00000000`00000000
```

```
kd> dt _HANDLE_TABLE_ENTRY fffff8a0`00005000+0x10*0x68

nt!_HANDLE_TABLE_ENTRY

   +0x000 Object           : 0xfffffa80`03685b01 Void

   +0x000 ObAttributes     : 0x3685b01

   +0x000 InfoTable        : 0xfffffa80`03685b01 _HANDLE_TABLE_ENTRY_INFO

   +0x000 Value            : 0xfffffa80`03685b01

   +0x008 GrantedAccess    : 0

   +0x008 GrantedAccessIndex : 0

   +0x00a CreatorBackTraceIndex : 0

   +0x008 NextFreeTableEntry : 0
```

```
kd> dt _EPROCESS 0xfffffa80`03685b00

   +0x2e0 ImageFileName    : [15]  "csrss.exe"
```

**Win10全局句柄表解析**

- 0级表每个元素占16个字节，总共可以存256个
- 查找`smss.exe`进程，pid为512。
- `512/4 = 128 < 256`，说明在第0个表。索引为16进制`0x80`
- 最后的`_HANDLE_TABLE_ENTRY_INFO`地址要先右移4位再`&0xFFFFFFFFFFFFFFF0`

```
kd> dq PspCidTable

fffff802`5a573528  ffffb784`82205d40 ffffa38b`78ab2a70

fffff802`5a573538  ffffa38b`78aad220 00000000`00000000

fffff802`5a573548  00010000`00000000 0001017d`00001000

fffff802`5a573558  00000000`00009705 00000000`00000000

fffff802`5a573568  00000000`00000000 00000000`00000000

fffff802`5a573578  00000000`00000000 00000000`00000000

fffff802`5a573588  00000000`00000000 ffffa38b`78aade80

fffff802`5a573598  00000000`00000000 00000000`00000000
```

```
kd> dt _HANDLE_TABLE ffffb784`82205d40

ntdll!_HANDLE_TABLE

   +0x000 NextHandleNeedingPool : 0x2000

   +0x004 ExtraInfoPages   : 0n0

   +0x008 TableCode        : 0xffffb784`89acc001

   +0x010 QuotaProcess     : (null) 

   +0x018 HandleTableList  : _LIST_ENTRY [ 0xffffb784`82205d58 - 0xffffb784`82205d58 ]

   +0x028 UniqueProcessId  : 0

   +0x02c Flags            : 1

   +0x02c StrictFIFO       : 0y1

   +0x02c EnableHandleExceptions : 0y0

   +0x02c Rundown          : 0y0

   +0x02c Duplicated       : 0y0

   +0x02c RaiseUMExceptionOnInvalidHandleClose : 0y0

   +0x030 HandleContentionEvent : _EX_PUSH_LOCK

   +0x038 HandleTableLock  : _EX_PUSH_LOCK

   +0x040 FreeLists        : [1] _HANDLE_TABLE_FREE_LIST

   +0x040 ActualEntry      : [32]  ""

   +0x060 DebugInfo        : (null) 
```

```
kd> dq 0xffffb784`89acc000

ffffb784`89acc000  ffffb784`8222a000 ffffb784`89acd000

ffffb784`89acc010  ffffb784`8a5ff000 ffffb784`8b6cd000

ffffb784`89acc020  ffffb784`8c3fd000 ffffb784`8cbfe000

ffffb784`89acc030  ffffb784`8ca35000 ffffb784`8b8fc000

ffffb784`89acc040  00000000`00000000 00000000`00000000

ffffb784`89acc050  00000000`00000000 00000000`00000000

ffffb784`89acc060  00000000`00000000 00000000`00000000

ffffb784`89acc070  00000000`00000000 00000000`00000000
```

```
kd> dt _HANDLE_TABLE_ENTRY ffffb784`8222a000+0x10*0x80

ntdll!_HANDLE_TABLE_ENTRY

   +0x000 VolatileLowValue : 0n-6662098147688252097

   +0x000 LowValue         : 0n-6662098147688252097

   +0x000 InfoTable        : 0xa38b7908`3040fd3f _HANDLE_TABLE_ENTRY_INFO

   +0x008 HighValue        : 0n0

   +0x008 NextFreeHandleEntry : (null) 

   +0x008 LeafHandleValue  : _EXHANDLE

   +0x000 RefCountField    : 0n-6662098147688252097

   +0x000 Unlocked         : 0y1

   +0x000 RefCnt           : 0y0111111010011111 (0x7e9f)

   +0x000 Attributes       : 0y000

   +0x000 ObjectPointerBits : 0y10100011100010110111100100001000001100000100 (0xa38b7908304)

   +0x008 GrantedAccessBits : 0y0000000000000000000000000 (0)

   +0x008 NoRightsUpgrade  : 0y0

   +0x008 Spare1           : 0y000000 (0)

   +0x00c Spare2           : 0
```

`_HANDLE_TABLE_ENTRY_INFO`地址：`0xa38b7908`3040fd3f`

计算出进程地址

```
0xa38b7908`3040fd3f >> 4 & 0xFFFFFFFFFFFFFFF0 = 0xFFFFA38B79083040
```

查看进程

```
kd> dt _EPROCESS FFFFA38B79083040

    +0x450 ImageFileName    : [15]  "smss.exe"
```

传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！

最后于 2022-8-24 02:09

被zhang_derek编辑，原因：

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)

[#系统底层](https://bbs.kanxue.com/forum-4-1-2.htm)
