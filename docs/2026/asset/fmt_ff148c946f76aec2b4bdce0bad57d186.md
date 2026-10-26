
# [原创]OD 分析被调试进程的数据来源

一直在用 OD，但是 OD 的原理是啥？那些数据怎么来的？

于是，就花了几天用 OD 调试了一下 OD，并写了点心得。呵呵。

- 【名称】OD 分析被调试进程的数据来源
- 【版本】`.\OllyICE\原版\英文原版\OLLYDBG.EXE v1.10`
- 【工具】IDA
- 【分析时间】2010-09-22

下面所贴出来的代码都是基于 `OLLYDBG.EXE v1.10` 英文版，可用 IDA 逆向参考，地址一定是匹配的。

## OD 的数据

数据表：

- `ModuleList`：模块列表。
- `ThreadList`：线程列表。
- `HeapList`：堆内存列表。
- `MemoryList`：内存列表。
- `WindowList`：窗口列表。
- `HandleList`：句柄列表。
- `PatchsList`：补丁列表。
- `CallStackList`：回调列表。
- `BreakPointList`：用户断点列表。
- `RefrencesList`：字符串参考列表。
- `RunTraceList`：运行跟踪列表。
- `SourceList`：源代码。

## 枚举

OD 的大部分数据来源，是通过对被调试进程的枚举而获取的数据。

下面是枚举的 API 描述：

### 线程枚举

可以得到当前操作系统的全部的线程，只能得到线程 ID，对线程操作需要线程句柄，句柄只能通过 `<jmp.&KERNEL32.WaitForDebugEvent>` 调试事件才可以得到。

```
hSnapShot := CreateToolHelp32SnapShot(TH32CS_SNAPTHREAD, Pid);
Thread32First(hSnapshot, lpte);
Thread32Next(hSnapshot, lpte);
```

### 进程枚举

```
CreateToolHelp32Snapshot(TH32CS_SNAPPROCESS, 0);
Process32First(FSnap, PE);
Process32Next(FSnap, PE);
```

### 模块枚举

```
hSnapShot := CreateToolHelp32SnapShot(TH32CS_SNAPMODULE, Pid);
Module32First(hSnapShot, lphl);
Module32Next(hSnapShot, lphl);
```

## OD 的这些数据的基本操作

首先，OD 通过进程枚举获取进程列表。

其次，开始创建、添加数据。

### 创建数据池

```asm
.text:00478013 loc_478013:                             ; CODE XREF: ShowProcessList+5Fj
.text:00478013                 push    0               ; FuncAddr2
.text:00478015                 push    offset FuncOfProcessTable ; FuncAddr1
.text:0047801A                 push    100h            ; ItemNum
.text:0047801F                 push    310h            ; RecordSizeOfItem
.text:00478024                 push    offset aProcessTable ; "Process table"
.text:00478029                 push    offset ProcessTableClass ; pResultDataPointer
.text:0047802E                 call    _Createsorteddata ; ：创建存储数据块(ProcessTable,ThreadTable,ModuleTable,MemTable.....)
.text:0047802E                                         ; ：pResultDataPointer= 新数据块指针
.text:0047802E                                         ;   pSrcTableName    = 数据块的名字
.text:0047802E                                         ;   RecordSizeOfItem= 每个记录的大小
.text:0047802E                                         ;   ItemNum         = 数量
.text:0047802E                                         ;   Func1        = 这个数据Table模块对应的函数
.text:0047802E                                         ;   Func2        = 这个数据Table模块的附加函数
.text:00478033                 add     esp, 18h
.text:00478036                 test    eax, eax        ; ProcessTableClass： Size=138h
.text:00478036                                         ; 004ED3FC  50 72 6F 63 65 73 73 20 74 61 62 6C 65 00 00 00  Process table...
.text:00478036                                         ;           Name
.text:00478036                                         ; 004ED40C  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
.text:00478036                                         ; ................
.text:00478036                                         ; 004ED4FC  00 00 00 00 00 00 00 00 00 01 00 00 FF FF FF FF
.text:00478036                                         ;                                   Num         Index=-1
.text:00478036                                         ; 004ED50C  7C 04 00 00 10 03 00 00 58 00 00 00 00 00 78 02
.text:00478036                                         ;                       RecordSize              Addr_Items
.text:00478036                                         ; 004ED51C  10 7C 47 00 00 00 00 00 00 00 00 00 01 00 00 00
.text:00478036                                         ;           AddrFunc
.text:00478036                                         ; 004ED52C  00 00 67 02 00 00 00 00
.text:00478036                                         ;           Pointer_FuncLists
.text:00478038                 jz      short loc_478044
.text:0047803A                 mov     eax, 1
.text:0047803F                 jmp     loc_478680
.text:00478044 ; ---------------------------------------------------------------------------
.text:00478044
.text:00478044 loc_478044:                             ; CODE XREF: ShowProcessList+90j
.text:00478044                 mov     ListViewProcessListColum1Width, 9
.text:0047804E                 mov     ListViewProcessListColum1Name, offset aProcess ; "Process"
.text:00478058                 mov     ListViewProcessListColum1Check, 0
.text:0047805F                 mov     ListViewProcessListColum2Width, 9
.text:00478069                 mov     ListViewProcessListColum2Name, offset aName_0 ; "Name"
.text:00478073                 mov     ListViewProcessListColum2Check, 0
.text:0047807A                 mov     ListViewProcessListColum3Width, 1Bh
.text:00478084                 mov     ListViewProcessListColum3Name, offset aWindow_0 ; "Window"
.text:0047808E                 mov     ListViewProcessListColum3Check, 0
.text:00478095                 mov     ListViewProcessListColum4Width, 100h
.text:0047809F                 mov     ListViewProcessListColum4Name, offset aPath_0 ; "Path"
.text:004780A9                 mov     ListViewProcessListColum4Check, 0
.text:004780B0                 mov     ListViewProcessListColumCount, 4
.text:004780BA                 push    offset ListViewProcessListColumCount
.text:004780BF                 mov     edx, IsShowListView
.text:004780C5                 mov     ListViewProcessList, edx
```

### 把数据添加到数据池

```asm
.text:00454EFC ; =============== S U B R O U T I N E =======================================
.text:00454EFC
.text:00454EFC ; 把数据添加到数据池：添加一个Record到 SortedData里面： 根据类型判断 Copy的大小。
.text:00454EFC ; 比如： Copy大小就是310h
.text:00454EFC ;   0012D3B4   004ED3FC  ASCII "Process table"
.text:00454EFC ;   0012D3B8   0012D4F0
.text:00454EFC ; Attributes: bp-based frame
.text:00454EFC
.text:00454EFC ; int __cdecl Addsorteddata(int pDestSortedData,void *src)
.text:00454EFC                 public _Addsorteddata
.text:00454EFC _Addsorteddata  proc near               ; CODE XREF: _Setbreakpointext+112p
```
