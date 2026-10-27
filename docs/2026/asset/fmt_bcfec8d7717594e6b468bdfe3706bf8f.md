
### `_MSUBSECTION`

```
kd> dt 0x87b312d0 _MSUBSECTION

nt!_MSUBSECTION

   +0x000 ControlArea      : 0x87b31280 _CONTROL_AREA

   +0x004 SubsectionBase   : 0x9acfc958 _MMPTE

   +0x008 NextSubsection   : 0x87b312f0 _SUBSECTION

   +0x008 NextMappedSubsection : 0x87b312f0 _MSUBSECTION

   +0x00c PtesInSubsection : 1

   +0x010 UnusedPtes       : 0

   +0x010 GlobalPerSessionHead : (null) 

   +0x014 u                : <unnamed-tag>

   +0x018 StartingSector   : 0

   +0x01c NumberOfFullSectors : 2

   +0x020 u1               : <unnamed-tag>

   +0x024 LeftChild        : 0x9acfc960 _MMSUBSECTION_NODE

   +0x028 RightChild       : 0x87b31310 _MMSUBSECTION_NODE

   +0x02c DereferenceList  : _LIST_ENTRY [ 0x3c - 0x0 ]

   +0x034 NumberOfMappedViews : 6
```

### 3. 锁页，把下面两个位都改为1，可以锁页

- `NoChange`：改为1
- `SecNoChange`：改为1

```
kd> dt 85dc2fc0+14 _MMVAD_FLAGS

nt!_MMVAD_FLAGS

   +0x000 CommitCharge     : 0y0000000000000000100 (0x4)

   +0x000 NoChange         : 0y0 		//改为1

   +0x000 VadType          : 0y010

   +0x000 MemCommit        : 0y0

   +0x000 Protection       : 0y00111 (0x7)

   +0x000 Spare            : 0y00

   +0x000 PrivateMemory    : 0y0

kd> dt 85dc2fc0+20 _MMVAD_FLAGS2

nt!_MMVAD_FLAGS2

   +0x000 FileOffset       : 0y000000000000000000000000 (0)

   +0x000 SecNoChange      : 0y0		//改为1

   +0x000 OneSecured       : 0y0

   +0x000 MultipleSecured  : 0y0

   +0x000 Spare            : 0y0

   +0x000 LongVad          : 0y0

   +0x000 ExtendableFile   : 0y0

   +0x000 Inherit          : 0y1

   +0x000 CopyOnWrite      : 0y0
```

## 8.3. 数据库页帧

### 1. `_MMPFN`

```
kd> dt _MMPFN -r1

nt!_MMPFN

   +0x000 u1               : <unnamed-tag>

      +0x000 Flink            : Uint4B

      +0x000 WsIndex          : Uint4B

      +0x000 Event            : Ptr32 _KEVENT

      +0x000 Next             : Ptr32 Void

      +0x000 VolatileNext     : Ptr32 Void

      +0x000 KernelStackOwner : Ptr32 _KTHREAD

      +0x000 NextStackPfn     : _SINGLE_LIST_ENTRY

   +0x004 u2               : <unnamed-tag>

      +0x000 Blink            : Uint4B

      +0x000 ImageProtoPte    : Ptr32 _MMPTE

      +0x000 ShareCount       : Uint4B				//当前页被映射了多少次

   +0x008 PteAddress       : Ptr32 _MMPTE

      +0x000 u                : <unnamed-tag>

   +0x008 VolatilePteAddress : Ptr32 Void

   +0x008 Lock             : Int4B

   +0x008 PteLong          : Uint4B

   +0x00c u3               : <unnamed-tag>

      +0x000 ReferenceCount   : Uint2B           //引用次数，为0的时候表示释放了

      +0x002 e1               : _MMPFNENTRY

      +0x000 e2               : <unnamed-tag>

   +0x010 OriginalPte      : _MMPTE

      +0x000 u                : <unnamed-tag>

   +0x010 AweReferenceCount : Int4B

   +0x014 u4               : <unnamed-tag>

      +0x000 PteFrame         : Pos 0, 25 Bits

      +0x000 PfnImageVerified : Pos 25, 1 Bit

      +0x000 AweAllocation    : Pos 26, 1 Bit

      +0x000 PrototypePte     : Pos 27, 1 Bit

      +0x000 PageColor        : Pos 28, 4 Bits
```

### 2. `_MMPFN`的`u3`

```
   +0x00c u3               : <unnamed-tag>

      +0x000 ReferenceCount   : Uint2B

      +0x002 e1               : _MMPFNENTRY

         +0x000 PageLocation     : Pos 0, 3 Bits  //属于哪个链表，就可以知道当前页的状态

         +0x000 WriteInProgress  : Pos 3, 1 Bit

         +0x000 Modified         : Pos 4, 1 Bit   //是否被修改过

         +0x000 ReadInProgress   : Pos 5, 1 Bit

         +0x000 CacheAttribute   : Pos 6, 2 Bits  //是否有缓存

         +0x001 Priority         : Pos 0, 3 Bits

         +0x001 Rom              : Pos 3, 1 Bit

         +0x001 InPageError      : Pos 4, 1 Bit

         +0x001 KernelStack      : Pos 5, 1 Bit

         +0x001 RemovalRequested : Pos 6, 1 Bit

         +0x001 ParityError      : Pos 7, 1 Bit

      +0x000 e2               : <unnamed-tag>

         +0x000 ReferenceCount   : Uint2B

         +0x000 VolatileReferenceCount : Int2B

         +0x002 ShortFlags       : Uint2B
```

### 3. `MmPageLocationList`

```
PMMPFNLIST MmPageLocationList[NUMBER_OF_PAGE_LISTS] = {

                                      &MmZeroedPageListHead,

                                      &MmFreePageListHead,

                                      &MmStandbyPageListHead,

                                      &MmModifiedPageListHead,

                                      &MmModifiedNoWritePageListHead,

                                      &MmBadPageListHead,

                                      NULL,

                                      NULL };
```

- `MmZeroedPageListHead`：零化页表，剩余的物理地址 都挂在这个链表上，并且这个物理地址里面的内容都是0
- `MmFreePageListHead`： 释放链表
- `MmStandbyPageListHead`：备用链表：已经被移出内存工作集的物理地址，内存内容没有被修改，可以转移到磁盘上，被其他映射使用，要使用之前，先得把数据交换到磁盘上，当 然也可以重新召唤到原工作集 继续使用
- `MmModifiedPageListHead`： 修改链表：跟备用链表一样，只是内容是被修改的，无论你是否召回到原工作集 他都要交换磁盘上
- `MmModifiedNoWritePageListHead`：修改不写出链表：跟备用链表一样，只是内容是被修改的，无论你是否召回到原工作集他都不会交换到磁盘上
- `MmBadPageListHead`：损坏链表

## 8.4. 修改页属性

### 1. 附加到进程

```
PROCESS 8809b718  SessionId: 1  Cid: 035c    Peb: 7ffdf000  ParentCid: 0578

    DirBase: 23b59000  ObjectTable: ab6de1a0  HandleCount:  13.

    Image: 1.vad.exe

PROCESS 8638fd40  SessionId: 1  Cid: 0ef0    Peb: 7ffd5000  ParentCid: 0804

    DirBase: 5fec5000  ObjectTable: a4161a98  HandleCount: 580.

    Image: cheatengine-i386.exe

kd> .process /i 8809b718  

You need to continue execution (press 'g' <enter>) for the context

to be switched. When the debugger breaks in again, you will be in

the new process context.

kd> g
```

### 2. 查看`pte`和`MMPFNDATABASE`

- `pte`的`523DF867`除掉后三位，`523DF`是页帧的索引

```
kd> !pte d0000

                 VA 000d0000

PDE at C0300000         PTE at C0000340

contains 271CE867       contains 523DF867

pfn 271ce ---DA--UWEV   pfn 523df ---DA--UWEV

kd> dd MMPFNDATABASE

841ad814  85400000 00000000 00000001 0007ffff

841ad824  0007ff7e 7ffeffff 80000000 7fff0000

841ad834  80d41000 0007ffff 80000000 86001c2c

841ad844  86001c24 86001c00 c01fffbc c03007fc

841ad854  00000000 00000001 00000002 00000001

841ad864  00000000 00000002 00000000 00000001

841ad874  00000002 00000001 00000000 00000002

841ad884  00000000 00000000 00000000 00000000
```

### 3. 查看 `+0x010 OriginalPte` 位置

```
kd> dt _MMPFN 85400000+523DF*1C -r2

   +0x010 OriginalPte      : _MMPTE

      +0x000 u                : <unnamed-tag>

         +0x000 Long             : 0x5560001

         +0x000 VolatileLong     : 0x5560001       //原型pte权限

         +0x000 Flush            : _HARDWARE_PTE

         +0x000 Hard             : _MMPTE_HARDWARE

         +0x000 Proto            : _MMPTE_PROTOTYPE

         +0x000 Soft             : _MMPTE_SOFTWARE

         +0x000 TimeStamp        : _MMPTE_TIMESTAMP

         +0x000 Trans            : _MMPTE_TRANSITION

         +0x000 Subsect          : _MMPTE_SUBSECTION

         +0x000 List             : _MMPTE_LIST

   +0x010 AweReferenceCount : 0n89522177
```

### 4. 修改

- `c0`：可读可写可执行
- `80`：可读可写
- `0`：No Access ，改成0 就变成无访问权限，可以避免注入后被检测到

```
kd> ed 85400000+523DF*1C+10 c0

kd> dt _MMPTE 85400000+523DF*1c+0x10 -r1

nt!_MMPTE

   +0x000 u                : <unnamed-tag>

      +0x000 Long             : 0xc0

      +0x000 VolatileLong     : 0xc0

      +0x000 Flush            : _HARDWARE_PTE

      +0x000 Hard             : _MMPTE_HARDWARE

      +0x000 Proto            : _MMPTE_PROTOTYPE

      +0x000 Soft             : _MMPTE_SOFTWARE

      +0x000 TimeStamp        : _MMPTE_TIMESTAMP

      +0x000 Trans            : _MMPTE_TRANSITION

      +0x000 Subsect          : _MMPTE_SUBSECTION

      +0x000 List             : _MMPTE_LIST
```

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)

[#系统底层](https://bbs.kanxue.com/forum-4-1-2.htm)
