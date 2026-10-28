
# [原创]Win11下利用PsSyscallProviderDispatch进行进程级SSDT HOOK

## 引言

系统服务分发表（SSDT）Hooking一直是一种强大而经典的技术，它允许开发者拦截和修改系统调用的行为。x64系统上内核补丁保护的引入，直接修改SSDT表变得异常危险，几乎必然会导致系统蓝屏（BSoD）。从Windows 10开始，微软引入了一套名为“替代系统调用（Alternative Syscalls）”或“系统调用提供者（Syscall Providers）”的机制，最初是为Windows Subsystem for Linux (WSL) 设计的。这套机制允许在进程级别指定一个自定义的系统调用处理程序。本文将深入分析其核心调度函数`PsSyscallProviderDispatch`的IDA伪代码，并详细讲解如何利用这个未公开的机制，在Windows 11上用C语言实现一个SSDT Hook。

## 正文

### 核心机制: `PsSyscallProviderDispatch` 解析

当一个线程的特定标志位被设置后，其系统调用请求将不再直接由传统的SSDT处理，而是被重定向到`PsSyscallProviderDispatch`函数。

![图片描述](upload/attach/202508/993779_ZFNSS958TV28HG8.webp)

关键检查：Pico Process: `CurrentThread->Header.Reserved1 & 4`是一个关键检查。这个标志位（在`_KTHREAD`的`MiscFlags`中，值为`0x4`）用于标识一个线程是否是“Pico Process”的一部分，例如Linux子系统（WSL）中的进程。如果是，它会走一个专门为WSL设计的特殊路径。对于我们想要Hook的普通Win32进程，这个位是0，所以我们会跳过这个if语句。

![图片描述](upload/attach/202508/993779_Z9C47ZVMWUMZ4UA.webp)

获取进程与SSN: `IoThreadToProcess`获取当前进程的`_EPROCESS`结构。`*(_DWORD *)(rsp_frame + 0x30)`从陷阱帧中取出RAX寄存器的值，即系统调用号（SSN）。

核心开关：Slot: `ProviderIndexSlot = pCurrentProcess->SyscallProviderDispatchContext.Slot;` 这是整个机制的核心开关。`_EPROCESS`结构中有一个未文档化的成员`SyscallProviderDispatchContext`，它包含一个`Slot`字段。这个`Slot`就是一个索引，它告诉内核应该去哪个“提供者”那里查找处理程序。我们实现进程级Hook的关键，就是修改目标进程`_EPROCESS`结构中这个`Slot`的值，使其指向我们自己。

全局注册表: `PspServiceDescriptorGroupTable`是一个全局的、未导出的数组，它像一个注册表，最多可以存放`0x20`（32个）系统调用提供者。

定位提供者: `(char *)&PspServiceDescriptorGroupTable + 0x18 * ProviderIndexSlot`这行代码通过`Slot`索引在全局表中定位到我们注册的`PspServiceDescriptorRow`结构。`0x18`（24字节）正好是`PspServiceDescriptorRow`结构的大小（在x64下，3个指针 * 8字节/指针）。

![图片描述](upload/attach/202508/993779_6NWPGTXD548MJH7.webp)

![图片描述](upload/attach/202508/993779_88KNUKEBXHSM8GJ.webp)

解码回调函数地址: `pfnServiceRoutine = *pProviderDescriptor + ((ServiceEntryValue >> 4) & ...)` 这行代码从我们的描述符`ServiceEntryValue`中解码出回调函数的绝对地址。

`ServiceEntryValue >> 4`: 右移4位。这说明描述符的低4位被用作标志位。因此，我们在构建描述符时，必须将回调函数的RVA（相对于驱动基址的偏移）左移4位。

`*pProviderDescriptor`: 这是我们驱动的基地址。RVA加上基地址就得到了回调函数的绝对地址。

选择分发路径: `if ( (ServiceEntryValue & 0x10) != 0 )` 这行代码检查描述符的第4位（值为`0x10`）。

这里我们以`NtCreateFile`为例子

![图片描述](upload/attach/202508/993779_WPUMTJD6GR7QPM6.webp)

可以发现正常,由于代码没有写卸载 清理等细节逻辑 并未完善 大体可以使用 ,还可以将自己的函数当作一个系统调用号使用,以及Win32k函数有的也走syscall同样可以研究进行Win32k的稳定hook , 所以由此让各位师傅们自行完善

