
# [原创]win7 32位进程隐藏（过PChunter检测）

总结一下进程隐藏的一些基本套路吧，适合萌新。

关于进程隐藏，网上说的最多的就是hook `NtQuerySystemInformation`吧，要么就是在内核去操作`ActiveProcessLinks`链表，这其实都是一个原理，都是去操作内核的活动进程链表，这个方法也着实有效

```
kd> dt _eprocess

ntdll!_EPROCESS

   +0x000 Pcb              : _KPROCESS

   +0x098 ProcessLock      : _EX_PUSH_LOCK

   +0x0a0 CreateTime       : _LARGE_INTEGER

   +0x0a8 ExitTime         : _LARGE_INTEGER

   +0x0b0 RundownProtect   : _EX_RUNDOWN_REF

   +0x0b4 UniqueProcessId  : Ptr32 Void

   +0x0b8 ActiveProcessLinks : _LIST_ENTRY

   +0x0c0 ProcessQuotaUsage : [2] Uint4B

	…
```

关于进程的活动链表：`ActiveProcessLinks`为双向循环链表，隐藏时只需获取需要隐藏进程的`EPROCESS`结构，进而获取循环链表地址，做链表删除结点操作，将当前进程的结点结构从链表中删除即可达到隐藏当前进程的目的。

另外，内核中的全局变量`PsActiveProcessHead`也指向这个系统的`EPROCESS`链表。

```
kd> ln PsActiveProcessHead

(83f5cf18)   nt!PsActiveProcessHead   |  (83f5cf20)   nt!PsReaperWorkItem

Exact matches:

    nt!PsActiveProcessHead = 

kd> dd 83f5cf18

83f5cf18  85fe37d8 83bf6d30 00000000 00000000

kd> !object 85fe37d8-b8

Object: 85fe3720  Type: (85fe3cb0) Process

    ObjectHeader: 85fe3708 (new version)

HandleCount: 3  PointerCount: 140
```

可见`PsActiveProcessHead`为系统进程system的`ActiveProcessLinks`字段的值。但是，单凭这一个操作只能在任务管理器里面隐藏进程，pchunter下会把你挖出来还把你标红。

除了摘除链表外，还需要操作进程的所有线程对象。获取进程的所有线程对象（`ThreadListHead`字段指向当前进程的线程链表），在线程对象中有一个指向其进程`EPROCESS`的指针，将指针修改为`explorer.exe`的`EPROCESS`地址；注意不要直接清零，而是选择一个确实存在`EPROCESS`地址，不然即使不蓝屏，隐藏的进程也死了。这一步完成后接着再来。。

最后一步，抹去`PspCidTable`中记录的隐藏进程所对应的`EPROCESS`地址，直接在对应地址上填0即可，当然在进程结束时需要将该数值写会，应为系统在进程结束时会检查`PspCidTable`中对应的进程对象，没有的话直接蓝屏。这里就需要定位`PspCidTable`的地址，和如何在表中找到进程对象。

定位`PspCidTable`的方法当然是通过特征码的扫描，通过`PsLookUpProcessByProcessId`定位即可；

```
kd> u PsLookupProcessByProcessId l 20

nt!PsLookupProcessByProcessId:

84076575 8bff            mov     edi,edi

84076577 55              push    ebp

84076578 8bec            mov     ebp,esp

8407657a 83ec0c          sub     esp,0Ch

840765a6 8a03            mov     al,byte ptr [ebx]

…

840765ac 750b            jne     nt!PsLookupProcessByProcessId+0x44 (840765b9)

840765ae 8bd3            mov     edx,ebx

840765b0 e83d8ee3ff      call    nt!ObReferenceObjectSafe (83eaf3f2)

840765b5 84c0            test    al,al

840765b7 7502            jne     nt!PsLookupProcessByProcessId+0x46 (840765bb)

840765b9 33db            xor     ebx,ebx

840765bb a134cff583      mov     eax,dword ptr [nt!PspCidTable (83f5cf34)]

840765c0 33c9            xor     ecx,ecx

…
```

`83f5cf34`指向一个`_handle_table`结构体

```
kd> dt _handle_table 83f5cf34

nt!_HANDLE_TABLE

   +0x000 TableCode        : 0x87a01210

   +0x004 QuotaProcess     : (null) 

   +0x008 UniqueProcessId  : 0x80000020 Void

   +0x00c HandleLock       : _EX_PUSH_LOCK

   +0x010 HandleTableList  : _LIST_ENTRY [ 0x80000300 - 0x80000024 ]

   +0x018 HandleContentionEvent : _EX_PUSH_LOCK

   +0x01c DebugInfo        : (null) 

   +0x020 ExtraInfoPages   : 0n0

   +0x024 Flags            : 0

   +0x024 StrictFIFO       : 0y0

   +0x028 FirstFreeHandle  : 0

   +0x02c LastFreeHandleEntry : 0x00000113 _HANDLE_TABLE_ENTRY

   +0x030 HandleCount      : 0

   +0x034 NextHandleNeedingPool : 0

   +0x038 HandleCountHighWatermark : 0x83f0d35a
```

`TableCode`的值为`PspCidTable`的一个指针

```
kd> dd 0x87a01210 

87a01210 9453a001 00000000 00000000 00000000
```

可以看到指针指向的地址是`0x9453a001`，注意这个地址最低位表示当前表的级数，由此可见当前系统`PspCidTable`有两层，我们掩去最低位

```
kd> dd 9453a000

9453a000  87a04000 9454b000 00000000 00000000
```

得到两个地址`87a04000`，`9454b000`即为一级表的地址，

```
kd> dd 87a04000 

87a04000  00000000 fffffffe 85fe3721 00000000

87a04010  85fe3419 00000000 85ffebb9 00000000

87a04020  85ffe8e1 00000000 85ffe609 00000000

87a04030  85fefac9 00000000 85fef7f1 00000000

87a04040  85fef519 00000000 85fef241 00000000

87a04050  85ff0021 00000000 85ff0d49 00000000

87a04060  85ff0a71 00000000 85ff0799 00000000

87a04070  85ff04c1 00000000 85ff1021 00000000
```

```
typedef struct _HANDLE_TABLE_ENTRY

{

    union

    {

        PVOID Object;

        UINT32 ObAttributes;

        struct _HANDLE_TABLE_ENTRY *InfoTable;

        UINT32 Value;

    };

    union

    {

        UINT32 GrantedAccess;

        struct

        {

            USHORT GrantedAccessIndex;

            USHORT CreatorBackTraceIndex;

        };

        INT32 NextFreeTableEntry;

    };

} HANDLE_TABLE_ENTRY, *PHANDLE_TABLE_ENTRY;
```

每8个字节为一组，比如`85fe3721 00000000`为一组，`85fe3721`为对象，`00000000`表示对象的一些属性，这一组的索引为1，故其代表的进程pid为4即为system

在解析`PspCidTable`时需要考虑`PspCidTable`的多级结构，在查询时注意表中的值需要抹掉低三位（`x & 0xfffffff8`）后才是真正的对象，同样在驱动退出需要恢复时需要将对象的第三位还原（`x | 0x00000001`）

最后附上代码
