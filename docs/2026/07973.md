
# [原创]Hyperplatform VT HOOK阅读笔记

放假期间看了一下Hyperplatform的VT框架，代码还是相当值得阅读学习的，在此分享下学习心得~，期间感谢@leeqwind大佬的悉心指导

源码戳这里:`[DdiMon](https://bbs.kanxue.com/elink@84aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6@1j5h3&6V1j5i4y4S2N6q4)9J5c8V1c8V1K9f1#2G2L8R3`.`.`)

## 0x01 前言

现在绝大多数CPU都已经支持虚拟化，目前这部分技术运用于`VMWare`，`VirtualBox`，`ifyou`大佬专注的`Hyper-V`... 还有除这些产品之外的 漏斗挖掘，游戏保护以及核晶防护(^\_^ 是啥我也说不清楚，总之很牛逼)。跟进程隔离机制类似，虚拟化技术将进程隔离的概念扩大到了操作系统，即若干个隔离的操作系统互不影响。而其中的工作原理想必大家也都有所了解，LZ对这方面的概念也不是很清楚，所以只是简单介绍下这个概念。即由`VMM`管理着所有`VM`的资源请求，`VM`即我们所说的客户机，当客户机需要寻址某个页面内存时，实际上除了Windows内核的页表寻址机制之外，还需要经过一个页面表的转换来使得每个客户机操作系统访问不同的物理内存，从而达到隔离的目的。那么这里就引出了新页表，它被称之为扩展页表（`Extend Page Table`）。当然这只是一方面，虚拟化处理的事情远远大于这个机制。`VMM`使用`VMXON`区域对一些数据进行记录和维护，而每个`VM`也需要有自己对应的`VMCS`区域，一个`VM`发生异常通过`vmexit`指令来到`VMM`，`VMM`通过该`VM`的`VMCS`区域了解它到底是发生了什么异常以决定是否对他进行处理。入门级有个这样的概念就差不多了，剩下的慢慢消化《处理器虚拟化技术》就行了

## 0x02 虚拟化处理器核心

虚拟化是CPU的硬件支持，因此多核情况下只能对每个核心进行虚拟化处理。这里主要看 `VmInitialization` ---> `VmpStartVm` ---> `AsmInitializeVm` + `VmpInitializeVm`这块逻辑。主要要提及的是这里用到了汇编

`AsmInitializeVm`

去调用

`VmpInitializeVm` 派发函数。这个是因为驱动在`DriverEntry`入口点调用的处理器虚拟化函数，当为每一个CPU准备好VM结构后会调用到 `vmlaunch`指令，这个指令会直接call入Guest机器环境中，如果不明确给Guest确定他的代码入口点为`DriverEntry`的后半段以及栈区是后半段的栈区 的话那么驱动就无法完成后续的执行工作，系统就会进入`HANG`的状态。

检测CPU的虚拟化支持以及CPU对`EPT`的支持这里就不再做仔细介绍了，详细的大家翻阅Inter手册会比我说的好~

这里需要详细说明的是`EPT`的构造这块逻辑...... 首先明确下一个事情，就是Hyperplatform中的虚拟化处理的是整个内核空间的，架设的VM也就有一台，而这台VM的内核空间也就是主机的内核空间，因此`EPT`的构造就需要达到这样一个目的：`GUEST`使用的`GPA`在被`EPT`转换后依然访问的是原始地址，也就是`GPA==HPA`。这样当CPU处理在`GUEST`模式下的时候访问`nt!NtDeviceIoControlFile`不至于发生数据紊乱而访问到`win32k!NtUserBuildHwndList+0x17d`。因此对于x64的四级页面表的构造函数`EptInitialization`的原理就在于此，就是把物理地址`A`对应的`EPT`项填为`A`的物理页面项。注意他调用的`EptpConstructTables`设计的比较好，可以递归构造每一级的页面表

紧接着就是堆栈的构建，这块堆栈就是`GUEST`与`HOST`交互时进行数据传输时使用的堆栈内存；在调用`vmlaunch`之前，需要初始化`VMM`用于管理`VM`的`VMCS`域，（必须调用完`__vmx_vmptrld`设置`VMCS`之后才可以用`vmwrite`写`vmcs`域）

其实很多`VMCS`域的填充就是当前系统下一些寄存器的值，但是有几个域还是非常重要的:

`VmcsField::kExceptionBitmap`域，决定了`VM_EXIT`时可以捕获哪些异常

`VmcsField::kGuestRsp`域，决定了`GUEST`模式下执行的堆栈地址

`VmcsField::kGuestRip`域，决定了`GUEST`模式下执行的入口点地址

`VmcsField::kGuestSysenterEsp`

域，决定了

`GUEST`模式下快速系统调用的堆栈地址

`VmcsField::kGuestSysenterEip`

域，决定了`GUEST`模式下快速系统调用入口点地址

`VmcsField::kHostRsp`域，决定了`GUEST`退出到`VMM`时的堆栈地址

`VmcsField::kHostRip`域，

决定了`GUEST`退出到`VMM`时的入口地址

```
error |= UtilVmWrite(VmcsField::kExceptionBitmap, exception_bitmap);

error |= UtilVmWrite(VmcsField::kGuestRsp, guest_stack_pointer);
error |= UtilVmWrite(VmcsField::kGuestRip, guest_instruction_pointer);

error |= UtilVmWrite(VmcsField::kGuestSysenterEsp, UtilReadMsr(Msr::kIa32SysenterEsp));
error |= UtilVmWrite(VmcsField::kGuestSysenterEip, UtilReadMsr(Msr::kIa32SysenterEip));

error |= UtilVmWrite(VmcsField::kHostRsp, vmm_stack_pointer);
error |= UtilVmWrite(VmcsField::kHostRip, reinterpret_cast<ULONG_PTR>(AsmVmmEntryPoint));
```

## 0x03 利用VT绕过PG执行Hook

![](https://bbs.kanxue.com/upload/attach/201804/747296_EMY78Q2NAGXNZV3.jpg)

1. 为目标函数创建两个页面（如果有重合的页面，那么复用同一个），一个具有只可执行属性 `shadow_exec`；另外一个可读可写可执行 `shadow_rw`；并为每一个Hook点构建Trampline函数 【Original Code Bytes】+ 【jmp Target+offset】; 并将

`shadow_exec`页面中函数所在偏移位置写入一个字节 `0xCC`。然后调用`vmcall`指令退回`VMM`状态（只有在该状态下才可以修改`EPT`），将Hook函数所在页面的`EPT`项替换为

`shadow_exec` (含有`INT 3`指令的不可读写页面)；那么这样下次如果函数被调用时就会触发`#BP`，相应的`VM_EXIT`退回

`VMM` 状态时就可以修改`GuestRip`，替换到`Fake_xxx`；紧接着`Fake_xxx`再调用下原函数就完成了第一步。

2. 那么既然现在函数所在的物理页属性已经被修改为不可读写状态（虽然被Patch成了`INT 3`），但是PatchGuard检测时就会触发`EptVolation`，`VM_EXIT`到`VMM`状态，那么这个时候将函数所在的`EPT`项替换为`shadow_rw`页面（可读可写可执行，代码没有被Patch过）就可以巧妙的躲过PG的检测；但是问题来了，那么这样的话后面Hook点就失效了~

3. 所以在替换成

`shadow_rw`的物理页的同时，还设置了`VmcsField::kCpuBasedVmExecControl`域里面的一个标志位叫做 `MTF`（Monitor Trap Flag），这个标志位的作用是单步执行完一条执行之后触发一次`VM_EXIT`（reason:`MonitorTrapFlag`） ，那么在这次`VMM`执行机会中就可以再而将Hook点所在的 `shadow_rw` 页面替换为

`shadow_exec` 所在物理页，完成偷梁换柱的操作；从此Hook点将保持下去。

## 0x04 存在的问题

虚拟化的强大之处在于他可以接管Guest内部的几乎所有的内部机制，那么这样的话就存在效率的问题，同样利用VT绕过PG检测时不断的来回替换物理页也是很消耗时间的一个方法；其次，目前Hyperplatform这个VT框架目前只能够以独占方式运行，如果存在多种这样的模块，那么只能是第一个生效，后续的都不能够再次虚拟化处理器核心；当然可能还有一些其他方面的问题，本文只讨论到这里~

如果有说的不到位的地方，还请大佬们指出~

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2019-1-2 14:11

被FaEry编辑，原因：
