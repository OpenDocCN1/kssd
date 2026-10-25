
# 突破 Windows XP 4G 内存限制续

> 原文：[`www.pediy.com/kssd/pediy12/142776.html`](https://www.pediy.com/kssd/pediy12/142776.html)

上次经过一段时间的研究后，发现了通过修改 `ntkrnlpa.exe` 可以突破 Windows XP 4G 内存限制，完全使用 4G 以上物理内存（[`bbs.pediy.com/showthread.php?t=137830`](http://bbs.pediy.com/showthread.php?t=137830)），但是在测试过程中发现使用 USB 存储设备时会出现蓝屏等问题，后来因为太忙，也没能继续研究。最近难得短暂清闲，功夫不负有心人，花了一个多星期终于找到问题所在了，下面谈谈在探索过程中的一些心得。

## 1. 首先替换 USB 相关驱动

既然一插 U 盘就蓝屏，而使用 USB 鼠标、摄像头等都没有问题，那么首先怀疑是 `usbstor.sys` 的问题。Windows 2003 可以支持 4G 以上物理内存，那么我们将 Windows 2003 的 USB 驱动替换到 XP 上来，首先替换 `usbstor.sys`，现象依旧。那么全部替换 USB 相关驱动，包括 `usbstor.sys`、`usbport.sys`、`usbhub.sys`、`usbehci.sys` 等，XP 可以正常启动，USB 设备也能使用，但经反复测试，发现使用 U 盘时有时还是会蓝屏。

## 2. 是谁造成了内存访问越界？

种种蓝屏迹象表明，很可能是因为内存读写越界造成的。破解 4G 限制后，虽然物理地址扩展到 32 位以上，但是虚拟地址依然是 32 位的啊，使用虚拟地址的程序不可能造成内存访问越界啊。那么有哪些代码用到了物理地址呢？

首先想到的是内存分页管理，负责映射物理地址到虚拟地址空间，维护了一个物理内存的页帧数据库 `MmPfnDatabase`，会不会是因为物理地址空间扩大后，这个页帧数据库并没有扩大，而导致映射 4G 以上物理内存时发生错误呢？

一番折腾，在 `MiInitMachineDependent` 函数中找到了如下代码

代码:

```
MmFreePagesByColor[0] = (PMMCOLOR_TABLES)&MmPfnDatabase[MmHighestPossiblePhysicalPage + 1];
```

看来 XP 对这个数据库空间还是有考虑的，验证一下

代码:

```
lkd> dd MmPfnDatabase
8088b0c8  818c6000 0000ff00 00000006 0000003f
lkd> dd MmHighestPossiblePhysicalPage
8088b120  00137fff 00137fff 000f7379 00000040
```

`MMPFN` 结构体大小为 1C 字节

代码:

```
lkd> dt _MMPFN
nt!_MMPFN
   +0x000 u1                 : __unnamed
   +0x004 PteAddress         : Ptr32 _MMPTE
   +0x008 u2                 : __unnamed
   +0x00c u3                 : __unnamed
   +0x010 OriginalPte        : _MMPTE
   +0x018 u4                 : __unnamed
lkd> ?818c6000+(137fff+1)*1c
Evaluate expression: -2085724160 = 83ae6000
lkd> dd MmFreePagesByColor
80886388  83ae6000 83ae6300 c0883000 f77fffff
```

`MmPfnDatabase` 确实是预留了足够大的空间，既然 `MmPfnDatabase` 空间足够大，那就不存在页面映射错误，还会是谁会直接操作物理地址呢？

## 3. 谁动了我的物理地址？

感谢 Geoff Chappell 的一篇文章

[《Licensed Memory in 32-Bit Windows Vista》](http://www.geoffchappell.com/viewer.htm?doc=notes/windows/license/memory.htm)

其中提到了微软官方关于 Windows XP sp2 以后版本无法显示全部物理内存的答复：

[《The RAM reported by the System Properties dialog box and the System Information tool is less than you expect in Windows Vista or in Windows XP Service Pack 2 or later version](http://support.microsoft.com/kb/888137?ln=en-us)》，

从这里又链接到完整版的介绍：

[《Changes to Functionality in Microsoft Windows XP Service Pack 2

Part 3: Memory Protection Technologies》](http://technet.microsoft.com/en-us/library/bb457155.aspx)

其中讲到了

引用: The largest driver PAE compatibility issue involves direct memory access (DMA) transfers and map register allocation. Many devices that support DMA, usually 32-bit adapters, are not capable of performing 64-bit physical addressing

大意是说： PAE 兼容的最大问题是驱动程序进行的直接内存访问（DMA）传输和映射寄存器分配。很多设备都支持 DMA，但 DMA 往往为 32 位的，不能执行 64 位物理寻址。

那么我们遇到的问题很可能就是 DMA 的问题了，因为在 DMA 传输过程中，DMA 不经过页表映射，直接访问物理地址。经过我们的 4G 内存限制破解后，物理地址范围扩大到了 4G 以上，PAE 最大可以寻址 64G（36 位物理地址），如果驱动在请求 DMA 向 4G 以上物理地址传输数据时，DMA 就会将 36 位地址截断为 32 位地址，从而将数据传输到错误的内存地址，导致蓝屏，难道我们只有祈求驱动分配 4G 以下物理内存了吗？DMA 硬件都只能接受 32 位地址，在碰到 36 位地址时就无能为力了吗？当然不能！

## 4. 双缓冲 DMA 传输

引用: The Windows 2000 Server family and later provide double-buffering for the DMA transaction by providing a 32-bit address that is indicated by a map register. The device can perform the DMA transaction to the 32-bit address and the kernel copies the memory to the 64-bit address that is provided to the driver.

Windows 2000 Server family 及其以后的版本通过映射寄存器为 DMA 传输提供了双缓冲机制，也就是说，为每次请求 DMA 传输的缓冲区再分配一块位于 32 位物理地址空间的内存，让 DMA 将数据传输到这块低地址空间，内核再将这些数据复制到驱动提供的 64 位地址。所以尽管存在 DMA 硬件限制，32 位 Windows 2003 依然能够正常使用 4G 以上内存。

## 5. 难道是 USB 驱动不支持双缓冲映射？

在向 DMA 写入物理地址时将 36 位物理地址截断了？那么逆向 USB 驱动，感谢 tiamo 对 USB 体系的分析

[`bbs3.driverdevelop.com/read.p...67&skinco=wind`](http://bbs3.driverdevelop.com/read.php?tid=86767&skinco=wind)，

不然真是无从着手，USB 数据传输时的调用堆栈如下：

代码:

```
b82540e0 f805a042 81fc29dc 81840c80 817e88f4 usbehci!EHCI_SubmitTransfer+0x53
b8254128 f805a74a 81fc2028 81840b08 804e3ec4 USBPORT!USBPORT_DmaEndpointActive+0x1f0
b8254154 f805cb7c 81fc2028 00000000 804e3ec4 USBPORT!USBPORT_DmaEndpointWorker+0x140
b825417c f805e4c3 81fc2028 00000003 00000001 USBPORT!USBPORT_CoreEndpointWorker+0x6d2
b82541f0 806f2a98 81fc2028 00000000 413e504d USBPORT!USBPORT_MapTransfer+0x76f
b825421c 8052e551 821c7788 81fc205c 00000001 hal!HalAllocateAdapterChannel+0x126
b8254234 f805e6df 821c7788 81fc2028 00000001 nt!IoAllocateAdapterChannel+0x2a
b8254278 f805f4d9 81fc2028 817adad0 804e3ec4 USBPORT!USBPORT_FlushMapTransferList+0x1b1
```

`USBPORT.SYS` 通过调用 `nt!IoAllocateAdapterChannel` 请求 DMA 通道，并执行回调函数 `USBPORT_MapTransfer`，原型如下：

代码:

```
signed int __stdcall USBPORT_MapTransfer(PDEVICE_OBJECT DeviceObject, PIRP CurrentIrp, PVOID MapRegisterBase, PTRANSFER pTransfer)
```

`MapRegisterBase` 即为映射寄存器基址，其实映射寄存器并非为硬件寄存器，而是指向一块内存映射区域，`MapRegisterBase` 指向了一个 `TRANSLATION_ENTRY` 结构体的列表，

`TRANSLATION_ENTRY` 结构体如下

代码:

```
typedef struct _TRANSLATION_ENTRY {
    PVOID VirtualAddress;
    ULONG PhysicalAddress;
    ULONG Index;
} TRANSLATION_ENTRY, * PTRANSLATION_ENTRY;
```

这个结构体中的 `VirtualAddress` 和 `PhysicalAddress` 指向了系统预先分配好的 4G 以下物理内存，用来作为双缓冲的备份缓冲区。在调用 `IoAllocateAdapterChannel` 时指定要申请的映射寄存器的数量，在 `USBPORT_MapTransfer` 中就可以通过 `MapRegisterBase` 指针使用申请数量的 `TRANSLATION_ENTRY` 结构体列表。

`USBPORT_MapTransfer` 会调用 `hal!IoMapTransfer`，`hal!IoMapTransfer` 大致执行路径如下：

代码:

```
__int64 __stdcall IoMapTransfer(PADAPTER_OBJECT AdapterObject, PMDL Mdl, PVOID MapRegisterBase, PVOID CurrentVa, PULONG Length, BOOLEAN WriteToDevice)
if ( MapRegisterBase )
{
    result = HalpMapTransfer(AdapterObject, Mdl, MapRegisterBase, CurrentVa, Length, WriteToDevice);
}
else
{
    计算缓冲区 Mdl 中的连续物理内存块，返回 64 位物理基址
}
  return result;
}
```

通过 `IoMapTransfer` 得到要传递给 DMA 的物理地址，如果 `MapRegisterBase` 为 0，就直接计算缓冲区 `Mdl` 中的连续物理内存块，返回 64 位物理基址，这个地址来自于 `Mdl`，不能保证它是否位于 4G 以下物理地址。如果 `MapRegisterBase` 不为 0，则调用 `hal! HalpMapTransfer`。

在 `HalpMapTransfer` 中，

代码:

```
PHYSICAL_ADDRESS
HalpMapTransfer(
    IN PADAPTER_OBJECT AdapterObject,
    IN PMDL Mdl,
    IN PVOID MapRegisterBase,
    IN PVOID CurrentVa,
    IN OUT PULONG Length,
    IN BOOLEAN WriteToDevice
)
{
if ((ULONG) MapRegisterBase & NO_SCATTER_GATHER
            && transferLength < *Length) {

        logicalAddress = translationEntry->PhysicalAddress + pageOffset;
        translationEntry->Index = COPY_BUFFER;
        index = 0;
        transferLength = *Length;
        useBuffer = TRUE;
    }
if (useBuffer  &&  WriteToDevice) {
        HalpCopyBufferMap(
            Mdl,
            translationEntry + index,
            CurrentVa,
            *Length,
            WriteToDevice
            );
    }
}
```

如果 `MapRegisterBase` 不为 0，且满足一定条件，就设置 `useBuffer` 为 TRUE，`useBuffer` 就是启用双缓冲的标志，如果 `useBuffer` 为 TRUE，并且是从内存写入到设备（`WriteToDevice`），那么就将数据复制到映射缓存（因为读取过程不涉及到 DMA 操作，所有不需要映射缓存参与）。看来，影响是否使用双缓冲的关键参数为 `MapRegisterBase`，只有 `MapRegisterBase` 不为 0，`HalpMapTransfer` 才会启用双缓冲机制，调用 `HalpCopyBufferMap` 将 64 位物理地址缓冲区拷贝到映射寄存器指向的位于 4G 以下地址空间的缓冲存。

VMWare 里调试一下，断点下在 `USBPORT_MapTransfer`，`MapRegisterBase` 还真为 0，既然 `MapRegisterBase` 来自于 `nt!IoAllocateAdapterChannel`，那么继续到这个函数中去寻找答案，在 `nt!IoAllocateAdapterChannel` 中发现如下代码：

代码:

```
if ( NumberOfMapRegisters && AdapterObject->NeedsMapRegisters )
{
    MasterAdapter = AdapterObject->MasterAdapter;
    MapRegisterBase = (int)((char *)MasterAdapter->MapRegisterBase + 12 * MapRegisterId);
}
else
{
    AdapterObject->MapRegisterBase = 0;
    AdapterObject->NumberOfMapRegisters = 0;
}
```

从条件语句看，只要 `NumberOfMapRegisters` 为 0 或者 `AdapterObject->NeedsMapRegisters` 为 0 二者之一满足条件，`AdapterObject->MapRegisterBase = 0` 都会执行。`NumberOfMapRegisters` 为申请的映射寄存器数量，这个值肯定不为 0，那么只有 `AdapterObject->NeedsMapRegisters` 为 0 了。`AdapterObject` 结构体为 `ADAPTER_OBJECT`，是 `USBPORT.SYS` 在系统启动的时候在 `USBPORT!USBPORT_StartDevice` 函数中调用 `nt!IoGetDmaAdapter` 获取的，`nt!IoGetDmaAdapte` 最终调用了 `hal! HalGetAdapter`，原型如下：

代码:

```
PADAPTER_OBJECT HalGetAdapter(PDEVICE_DESCRIPTION DeviceDescription, PULONG NumberOfMapRegisters)
```

`DeviceDescription` 为总线驱动程序发送的一些生成 `ADAPTER_OBJECT` 的参数，`NumberOfMapRegisters` 是个指针，用来返回驱动能使用的最大的映射寄存器数量。



在 `HalGetAdapter` 函数中发现了如下代码：

代码:

```
if ( ScatterGather && (LessThan16Mb || InterfaceType == Eisa || InterfaceType == PCIBus) )
{
    MapRegistersNeed = 0;
}
```

`ScatterGather` 为分散\\聚集模式，一般都为 `TRUE`，`LessThan16Mb` 一般为 `FALSE`，现在基本没有 16M 以下内存的机器了吧，关键是 `InterfaceType`，如果是 `Eisa` 或者 `PCIBus` 任意一种，`MapRegistersNeed` 都会设为 `0`，那么双缓冲模式就不会被启动。现在我们的机器一般都是使用 PCI 总线，而 USB 总线是挂在 PCI 总线下的，所以就不会启用双缓冲了。那 `Windows 2003` 里对应的代码是怎样的呢？

代码:

```
if ( bScatterGather
    && (!HalpPhysicalMemoryMayAppearAbove4GB || DevDesc->Dma64BitAddresses)
    && (LessThan16Mb || InterfaceType == Eisa || InterfaceType == PCIBus) )
{
    MapRegistersNeed = 0;
}
```

原来 `Windows 2003` 多加了一个判断 `HalpPhysicalMemoryMayAppearAbove4GB` 和 `DevDesc->Dma64BitAddresses`，按照 2003 的逻辑，如果我们的机器物理地址空间大于 4GB 了，并且 DMA 硬件不支持 64 位地址，那么就会直接开启双缓冲，而不管总线是什么类型，或者其他的什么条件。`XP` 上这段程序是建立在只有 32 位物理地址的前提下，PCI 总线和 `Eisa` 总线的 DMA 都支持到 32 位的，所以不需要双缓冲机制。

既然关键点找到了，那就 patch 吧 `jz` 改成 `jmp`。此外还要注意 `Windows 2003` 在多个地方判断了 `HalpPhysicalMemoryMayAppearAbove4GB` 这个标志，涉及到修改的地方主要在 `HalInitSystem` 里面。由于 `XP` 原来主要工作于 32 位物理地址空间，现在地址空间变大，启用了双缓冲模式，所有相应参数要改大：

`HalpMapBufferSize` 为映射缓存大小，原来为 `0x10000`，改为 `0x30000`

`SizeOfBitMap` 为映射寄存器使用位图大小，原来为 `0x10`，改为 `0x30`

`HalInitSystem` 调用了 `HalpAllocPhysicalMemory` 申请映射缓存，物理内存少于 4G 时，申请参数为

`HalpAllocPhysicalMemory(LoaderBlock, 0x1000000u, 0x10u, 1);`

即在物理地址 `0x1000000u` 以下的空间申请 `0x10u` 页，最后一个参数表示是否 64k 对齐。而当物理内存大于 4G 时，`Windows 2003` 申请参数为

`HalpAllocPhysicalMemory(LoaderBlock, 0xFFFFFFFFu, 0x30u, 1);`

即在 4G 以下物理地址空间申请 `0x30u` 页，64k 对齐。汇编代码改动结果如下：

代码:

```
 push    1
 push    10h
 push    1000000h
 push    ebx
 mov     DefaultSizeOfBitMap, 40h
 mov     esi, 10000h
 call    _HalpAllocPhysicalMemory@16
```

 改为

代码:

```
 push    1
 push    30h
 push    FFFFFFFFh
 push    ebx
 mov     DefaultSizeOfBitMap, 4000h
 mov     esi, 30000h
 call    _HalpAllocPhysicalMemory@16
```

这就搞定了，对了，改完 `hal.dll` 一定要重新计算校验和，不然会启动失败。

搞到最后，原来还是 `hal.dll` 的问题，还白费力气，研究了半天 `usb` 驱动。既然是 `hal.dll` 的问题，为什么硬盘、网卡等等这些使用 DMA 的设备就没有引发系统蓝屏呢？难道说这些驱动在一开始发起请求的时候就总是申请 4G 以下物理内存来进行 DMA 操作？或者是其他设备的 DMA 硬件已经都支持 36 位以上寻址了？还有待去考证。

总算搞完了，希望这次补丁能够更加稳定些，还有问题的话不知道什么时候再能去解决了。在此感谢我敬爱的领导兼导师，感谢他把我领进 `ROOTKIT` 的大门，感谢他的包容和理解，感谢他对我年轻气盛的容忍和悉心开导。不玩了，好好工作，努力干活。

上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || `XP64G20111113.rar`

 |
