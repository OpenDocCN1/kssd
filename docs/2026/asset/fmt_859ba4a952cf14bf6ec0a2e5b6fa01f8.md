
5. 既然是双重释放导致的蓝屏，那么 BSOD 中的调用堆栈应该为第二次触发 `free` 的堆栈

6. 根据前五点得出的信息，想要得知 poc 的工作流程，首先得去 `0x1207F` 控制码的处理例程中去分析

7. 想要得到 `0x1207F` 控制码的处理例程，根据调用堆栈，进行跟踪即可

# 0x1207F处理例程分析

## 1.双机调试

在 `NtDeviceIoControlFile` 函数下条件断点，IO 控制码在第六个参数，也就是 `esp+18`，命令如下，之后使用 `wt` 命令即可跟踪调用堆栈（不建议，我卡死了）

所以尝试在调用较少的函数处下断，例如 `afd!AfdReturnTpInfo`，它会调用 `IoFreeMdl` 释放内存（查看 IDA 函数引用可得知，调用 `AfdReturnTpInfo` 的函数只有几个，所以可以尝试在这些函数下断，如果没成功定位（也就是两次释放的流程不同，不是都经过 `AfdReturnTpInfo`），则可以逆向分析派遣例程中的控制码，或者是使用 `wt` 命令追踪是比较稳妥的）

![](https://bbs.kanxue.com/upload/attach/202205/930159_869S2WSUENUPDZF.png)

```
0: kd> bp nt!NtDeviceIoControlFile ".if (poi(esp+18) = 0x1207F){}.else{gc;}"
0: kd> wt
```

```
1: kd> bp afd!AfdReturnTpInfo
1: kd> g
Breakpoint 0 hit
afd!AfdReturnTpInfo:
94273e03 8bff            mov     edi,edi
2: kd> kb
# ChildEBP RetAddr  Args to Child
00 9fee3a38 942578c1 875794b8 00000001 0bcaffe7 afd!AfdReturnTpInfo
01 9fee3aec 9425c2bc 8a17c958 88ba0938 9fee3b14 afd!AfdTransmitFile+0x5a3
02 9fee3afc 8463e593 88ba0938 89732418 89732418 afd!AfdDispatchDeviceControl+0x3b
03 9fee3b14 8483299f 8a17c958 89732418 897324f4 nt!IofCallDriver+0x63
04 9fee3b34 84835b71 88ba0938 8a17c958 00000000 nt!IopSynchronousServiceTail+0x1f8
05 9fee3bd0 8487c3f4 88ba0938 89732418 00000000 nt!IopXxxControlFile+0x6aa
06 9fee3c04 846451ea 00000050 00000000 00000000 nt!NtDeviceIoControlFile+0x2a
07 9fee3c04 773a70b4 00000050 00000000 00000000 nt!KiFastCallEntry+0x12a
08 0014f8d8 773a5864 7556989d 00000050 00000000 ntdll!KiFastSystemCallRet
09 0014f8dc 7556989d 00000050 00000000 00000000 ntdll!ZwDeviceIoControlFile+0xc
2: kd> kb
# ChildEBP RetAddr  Args to Child
00 9fee3a08 942568ac 875794b8 00000001 0bcaff4f afd!AfdReturnTpInfo
01 9fee3a44 94257bba 0bcaffe7 000120c3 94257a8c afd!AfdTliGetTpInfo+0x89
02 9fee3aec 9425c2bc 8a17c958 88ba0938 9fee3b14 afd!AfdTransmitPackets+0x12e
03 9fee3afc 8463e593 88ba0938 89732418 89732418 afd!AfdDispatchDeviceControl+0x3b
```

好的，经过我们的推测后下断，并启动 POC 程序，成功断下，查看调用堆栈，可以看到 `afd!AfdTransmitFile` 的调用。

所以控制码 `0x1207F` 对应的处理例程为 `AfdTransmitFile`。

第二次 `g` 之后断下，查看堆栈，`0x120C3` 对应的处理例程就肯定是 `AfdTransmitPackets` 了。

之后再 `g`，蓝屏，太好了，两次调用后蓝屏，正好对应 poc 的工作流程。

## 2.逆向分析AfdTransmitFile

定位到两个关键的处理例程之后，开始进行逆向分析，把驱动文件拷出来，使用 IDA 加载符号后分析。

`AfdTransmitFile` 的两个参数，由于它是 `fastcall`，所以它的参数在 `ecx` 与 `edx` 中。根据资料得知，`ecx` 为 `PIrp`，`edx` 为 `PIoStackLocation`（你也可以在派遣例程中逆向或者动态跟踪得到）。

查看一下它们的结构，对于 `_IO_STACK_LOCATION`，它的 `Parameters` 成员对于不同的 `IRP` 处理例程有不同的定义，下边给出的结构体是对于 `DeviceIoControl` 时的定义。

```
kd> dt _Irp
ntdll!_IRP
+0x000 Type             : Int2B
+0x002 Size             : Uint2B
+0x004 MdlAddress       : Ptr32 _MDL
+0x008 Flags            : Uint4B
+0x00c AssociatedIrp    : <unnamed-tag>
+0x010 ThreadListEntry  : _LIST_ENTRY
+0x018 IoStatus         : _IO_STATUS_BLOCK
+0x020 RequestorMode    : Char
+0x021 PendingReturned  : UChar
+0x022 StackCount       : Char
+0x023 CurrentLocation  : Char
+0x024 Cancel           : UChar
+0x025 CancelIrql       : UChar
+0x026 ApcEnvironment   : Char
+0x027 AllocationFlags  : UChar
+0x028 UserIosb         : Ptr32 _IO_STATUS_BLOCK
+0x02c UserEvent        : Ptr32 _KEVENT
+0x030 Overlay          : <unnamed-tag>
+0x038 CancelRoutine    : Ptr32     void
+0x03c UserBuffer       : Ptr32 Void
+0x040 Tail             : <unnamed-tag>

dt _IO_STACK_LOCATION
ntdll!_IO_STACK_LOCATION
+0x000 MajorFunction    : UChar
+0x001 MinorFunction    : UChar
+0x002 Flags            : UChar
+0x003 Control          : UChar
+0x004 Parameters       : <unnamed-tag>

struct {
0x4        ULONG  OutputBufferLength;
0x8        ULONG POINTER_ALIGNMENT  InputBufferLength;
0xC        ULONG POINTER_ALIGNMENT  IoControlCode;
0x10        PVOID  Type3InputBuffer;
} DeviceIoControl;

+0x014 DeviceObject     : Ptr32 _DEVICE_OBJECT
+0x018 FileObject       : Ptr32 _FILE_OBJECT
+0x01c CompletionRoutine : Ptr32 long
+0x020 Context          : Ptr32 Void
```

然后在 IDA 中插入这两个结构体的定义，如下图输入名字，点击 OK 即可。

![](https://bbs.kanxue.com/upload/attach/202205/930159_GDTGCM68UNR837Y.png)

接下来可以开始正式的分析了，我使用汇编进行分析，可能篇幅会比较长。

![](https://bbs.kanxue.com/upload/attach/202205/930159_5PCWZUHF5NCPJFM.png)

这里会有分支，通过判断 `FsContext` 的值，这个时候需要动态调一下，查看程序走了哪个分支。

```
1: kd> bp afd!AfdTransmitFile
1: kd> g
Breakpoint 0 hit
afd!AfdTransmitFile:
8db6c31e 6884000000      push 84h
1: kd> r edx
edx=89689b4c
1: kd> dt _IO_STACK_LOCATION 89689b4c
ntdll!_IO_STACK_LOCATION
+0x018 FileObject       : 0x89a33ab0 _FILE_OBJECT
1: kd> dt _FILE_OBJECT 0x89a33ab0
ntdll!_FILE_OBJECT
+0x00c FsContext        : 0x877fcbc0 Void
1: kd> dd 0x877fcbc0
877fcbc0  0004afd2 00000000 00000200 89a3f658
```

这里得到 `AFD2`，明显与 `1AFD` 不同，所以这个跳转指令生效，继续分析。

![](https://bbs.kanxue.com/upload/attach/202205/930159_UXU7GWRVVQPNRPR.png)

这里又有判断跳转，动态查看一下参数。

```
dd 89689b4c+0x8
89689b54  00000030 0001207f 00d63390 8751c4a0
```

这里得到 `30`，所以跳转生效，继续分析，又遇到两个跳转。

![](https://bbs.kanxue.com/upload/attach/202205/930159_8JF65K29SVB4BK7.png)

动态查看参数，确定流程。

```
1: kd> r ecx
ecx=89689a70
1: kd> dt _IRP 89689a70
ntdll!_IRP
+0x000 Type             : 0n6
+0x002 Size             : 0x190
+0x004 MdlAddress       : (null)
+0x008 Flags            : 0x60000
+0x00c AssociatedIrp    : <unnamed-tag>
+0x010 ThreadListEntry  : _LIST_ENTRY [ 0x897fe7fc - 0x897fe7fc ]
+0x018 IoStatus         : _IO_STATUS_BLOCK
+0x020 RequestorMode    : 1 ''
```

这里可以得到 `mode` 值为 `1`，所以跳转未生效，继续查看下一个跳转。

```
1: kd> dt _IO_STACK_LOCATION 89689b4c
ntdll!_IO_STACK_LOCATION
+0x000 MajorFunction    : 0xe ''
+0x001 MinorFunction    : 0x1f ''
+0x002 Flags            : 0x5 ''
+0x003 Control          : 0 ''
+0x004 Parameters       : <unnamed-tag>

1: kd> dd 89689b4c + 0x10
89689b5c  00d63390 8751c4a0 89a33ab0 00000000
```

这里得到 `eax+10` 也就是 `Type3InputBuffer` 的值，低 4 位为 `0`，所以跳转生效，继续分析。

这里提一下，为什么说是判断是否 4 字节对齐，看后面调用的参数。根据微软的解释：`ExRaiseDatatypeMisalignment` 例程可用于结构化异常处理，为驱动程序处理 I/O 请求时发生的不对齐数据类型引发驱动程序确定的异常。

好的，继续分析，这里又遇到跳转，可以单步到这个位置看一下这个全局变量的值，值为 `0x7FFF000`。

![](https://bbs.kanxue.com/upload/attach/202205/930159_7ZVVDRFF8JPBNGN.png)

![](https://bbs.kanxue.com/upload/attach/202205/930159_RWRTRASS986NYWQ.png)

分析到这里，有三个关键的跳转，如下。

```
InputBufferLength >= 0x30;
InputBuffer & 0x3 == 0;
InputBuffer < 0x7FFF000;
```

然后继续往下分析，遇到两个跳转，动态跟踪 `ecx = 1`，所以不跳转；动态跟踪 `ebp-80 = 0`，跳转生效。

![](https://bbs.kanxue.com/upload/attach/202205/930159_9XNDWHEV9D5XXU9.png)

```
afd!AfdTransmitFile+0x90:
8db6c3ae f7c1c8ffffff    test    ecx,0FFFFFFC8h
1: kd> r ecx
ecx=00000001
```

![](https://bbs.kanxue.com/upload/attach/202205/930159_8Z2ZPWYPHU7C83T.png)

好的，继续往下分析，这里遇到两个跳转，动态跟踪 `Afd` 全局变量的值为 `0x10`。
