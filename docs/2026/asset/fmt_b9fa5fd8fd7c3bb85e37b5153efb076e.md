
# VT技术笔记（part 1）

最近在学习 VT 技术，想把学习过程中记录的笔记分享出来。技术不精有不对的地方还望指正。代码会发到`563K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K6L8h3q4D9L8s2A6Z5L8$3&6Y4i4K6u0r3L8i4W2$3N6q4!0q4z5q4!0n7c8W2)9&6z5g2!0q4y4q4!0n7z5q4!0m8b7g2!0q4y4q4!0n7b7W2)9&6x3#2!0q4y4g2!0n7b7g2)9&6x3#2!0q4y4q4!0n7z5q4!0m8c8q4!0q4c8W2!0n7b7#2)9^5b7#2!0q4y4#2)9&6b7W2!0m8c8g2!0q4y4g2)9^5z5g2)9^5c8q4!0q4z5q4!0n7c8W2)9&6z5q4!0q4y4g2)9&6b7#2!0m8z5q4!0q4y4W2)9&6y4W2!0n7c8q4!0q4y4g2!0n7y4#2!0m8y4g2!0q4y4q4!0n7z5q4!0m8c8q4!0q4c8W2!0n7b7#2)9^5b7#2!0q4z5q4!0n7c8W2)9&6z5q4!0q4y4W2!0n7x3W2!0m8x3g2!0q4y4g2)9^5y4W2)9&6z5g2!0q4y4g2!0m8c8g2)9^5b7#2!0q4x3#2)9^5x3q4)9^5x3W2!0q4y4W2!0m8b7#2!0m8x3W2!0q4z5q4!0n7c8W2)9^5c8i4y4@1j5i4u0Q4c8f1k6Q4b7V1y4Q4z5p5y4Q4c8e0W2Q4b7e0u0Q4z5o6c8Q4c8e0S2Q4b7f1g2Q4b7e0q4Q4c8e0S2Q4b7V1k6Q4z5e0W2Q4c8e0c8Q4b7U0S2Q4b7f1q4Q4c8e0k6Q4z5f1y4Q4z5o6S2Q4c8e0g2Q4z5o6k6Q4z5o6g2Q4c8e0g2Q4b7f1g2Q4z5p5y4Q4c8e0g2Q4b7U0N6Q4b7e0g2Q4c8e0y4Q4z5o6m8Q4z5o6t1`.

## VT概述

![image-20220313143659687](upload/attach/202203/899076_ETC3FNV7JHNM5G4.jpg)

VT技术的大致含义如上。一个 CPU 上有 vm monitor 和 guest。在 guest 中感受不到自己跑在虚拟环境中。执行普通的操作时在 guest 中的操作和未开启 VT 时的操作并没有什么不同，但是在执行一些特殊的指令或者进行一些特殊的操作，如执行`cpuid`指令、切换`cr3`指令、读写 MSR 寄存器指令、触发异常时，会将控制权交还给 vm monitor，通过 vm monitor 确定应该如何让该指令或者操作得到执行。在云服务器的搭建中可以使用该技术让一台实体机上可以跑多台虚拟机。而该技术也可以做到无痕对一些指令进行高权限的 hook。甚至可以通过 EPT 技术进行对指定地址的无痕 hook。

## VT执行的大体流程

该流程的描述出自 B 站周壑的 VT 教学视频中，讲得非常通俗易懂。VT 的开启与关闭大致分为开锁、开柜门、拔电源、选中机器、装机、开机、拔电源、关柜门、关锁 9 个步骤。这 9 个步骤的具体描述如下。此处为大体的描述，在后面会对每一个步骤的具体细节进行具体的说明。

![image-20220307081330564](upload/attach/202203/899076_497NPCJCBQQ7QGP.jpg)

1. 开锁：检测是否支持 VT
2. 开柜门：`vmxon`。需要申请一块内存然后把这块内存的`lowpart`和`highpart`作为参数调用`vmxon`。注意这块内存的开头要填入一个特定的 MSR 寄存器中的值。注意这里`vmxon`修改`cr4`寄存器之后，`cr0`寄存器的页保护和段保护位都不能再置位，如果试图修改`cr0`的页保护或者段保护位，会触发异常蓝屏。
3. 拔电源：`vmclear`
4. 选中机器：`vmptrload`，相当于一个指针。需要通过`vmptrload`来指向某个机器。如果当前有多个机器的话，在各个机器之间的切换操作也需要用到`vmptrload`。
5. 装机：设置`vmcs`。也是要申请一块内存，然后在这块内存里面填入开启虚拟机时虚拟机使用的一些寄存器。要注意这里不能直接操作这块内存区域，要用`vmwrite`来操作。因为随着版本的更新，原来在这块内存区域的东西可能后来就不在了。`vmwrite`保证兼容性。
6. 开机`vmlaunch`
7. 拔电源`vmclear`
8. 关柜门`vmoff`

## 检测是否支持VT（开锁）

在 Intel 手册 31.5 章中详细描述了 VMM 的开启过程。

![image-20220313141119292](upload/attach/202203/899076_EB2ZBTZ7PV5SX2N.jpg)

1. 通过`cpuid`指令查询 CPU 是否支持 VT
2. 通过一些特定的 MSR 寄存器（在 31.5.1 中有介绍）来确认是否支持开启 VT
3. 申请一块非分页内存，用来存放`vmxon`区域。其大小在`IA32_VMX_BASIC`这个寄存器中指定。该内存要 4KB 对齐（后 12 位全为 0）。实际申请时申请一个页的大小的非分页内存即可。
4. 初始化`vmxon`区域的版本编号（前 4 个字节）。版本编号通过`IA32_VMX_BASIC`这个寄存器的低 4 字节获取。实际实验时发现这个数为 1，理论上把上一步申请到的内存前四字节置 1 即可，但为了兼容最好还是从寄存器中取值出来。
5. 给`cr0`和`cr4`寄存器的特定位置置位，使得其满足如下要求

- `IA32_VMX_CR0_FIXED0` 寄存器中为 1 的位，在`cr0`中必须为 1
- `IA32_VMX_CR0_FIXED1` 寄存器中为 0 的位，在`cr0`中必须为 0
- `IA32_VMX_CR4_FIXED0` 寄存器中为 1 的位，在`cr4`中必须为 1
- `IA32_VMX_CR4_FIXED1` 寄存器中为 0 的位，在`cr4`中必须为 0

6. 确认`IA32_FEATURE_CONTROL`寄存器被正确置位。该寄存器的具体细节可以通过《处理器虚拟化技术》2.3.2.1 找到。

![image-20220313142937541](upload/attach/202203/899076_MT52C76KPQNXC7S.jpg)

也就是说，该寄存器的`bit0`是 lock 位，`bit2`是 outside 位。必须保证这两位均为 1 才可以开启 VT。可以大致通过读取该寄存器后将其`& 5`，判断是否两个位均为 1。

7. 执行`vmxon`指令，传进去的参数是一个指向存放`vmxon`区域的物理地址的指针。如果该指令成功，那么`rflags.cf = 0`，否则`rflags.cf = 1`。

由上可写出检测是否支持 VT 的代码如下

```cpp
//检测Bios是否开启VT
BOOLEAN VmxIsCheckSupportVTBIOS()
{
ULONG64 value = __readmsr(IA32_FEATURE_CONTROL);
return (value & 0x5) == 0x5;
}
//检测CPU是否支持VT
BOOLEAN VmxIsCheckSupportVTCPUID()
{
int cpuidinfo[4];
__cpuidex(cpuidinfo, 1, 0);
//CPUID 是否支持VT ecx.vmx第6位 如果为1，支持VT，否则不支持
return (cpuidinfo[2] >> 5) & 1;
}
//检测CR4VT是否开启，如果为1 代表已经开启过了，否则没有开启
BOOLEAN VmxIsCheckSupportVTCr4()
{
ULONG64 mcr4 = __readcr4();
//检测CR4 VT是否开启，cr4.vmxe如果第14位为1，那么VT已经被开启，否则可以开启
return ((mcr4 >> 13) & 1) == 0;
}
void checkVT()
{
if (VmxIsCheckSupportVTCPUID())
{
DbgPrintEx(77, 0, "[db]:VmxIsCheckSupportVTCPUID number = %d\r\n", KeGetCurrentProcessorNumber());
}
if (VmxIsCheckSupportVTBIOS())
{
DbgPrintEx(77, 0, "[db]:VmxIsCheckSupportVTBIOS number = %d\r\n", KeGetCurrentProcessorNumber());
}
if (VmxIsCheckSupportVTCr4())
{
DbgPrintEx(77, 0, "[db]:VmxIsCheckSupportVTCr4 number = %d\r\n", KeGetCurrentProcessorNumber());
}
}
```

## `vmxon`（开柜门）

申请一块 4KB 对齐的非分页内存，作为`vmxon`的参数。

```c
PHYSICAL_ADDRESS lowphys,heiPhy;
lowphys.QuadPart = 0;
heiPhy.QuadPart = -1;
pVcpu->VmxOnAddr = MmAllocateContiguousMemorySpecifyCache(PAGE_SIZE, lowphys, heiPhy, lowphys, MmCached);
```

设置`vmxon`区域。这一块的说明在 Intel 白皮书 24.11.5 中。这一块内存是操作系统使用的。在申请区域之后要`rtlzeromemory`一下，防止没有挂页。其他位都置零即可，之后操作系统进行虚拟机的管理的时候会使用到这块内存。我们需要设置的只有头四个字节。要把 MSR 中`IA32_VMX_BASIC`这个寄存器的低 4 字节填入`vmxon`的头 4 字节。否则`vmxon`的执行会出错。基本上这个寄存器的低 4 字节会是 1。之后随着版本的更新可能会有变化。

![image-20220307110817566](upload/attach/202203/899076_JZ8GDFKAA34ENBR.jpg)

```cpp
ULONG64 vmxBasic = __readmsr(IA32_VMX_BASIC);
*(PULONG)pVcpu->VmxOnAddr = (ULONG)vmxBasic;
```

在设置完`vmxon`区域之后，还需最后一步即可`vmxon`。需要根据 MSR 寄存器中的指示进行`cr0`和`cr4`寄存器的设置。

![](upload/attach/202203/899076_P5MMV2VG7WM5E7U.jpg)

具体解释在附录 A 中。

![image-20220307112855414](upload/attach/202203/899076_P7HRKEMZSX3VG4R.jpg)

简单解释就是在进行`vmxon`之前，`IA32_VMX_CR0_FIXED0`寄存器中为 1 的值，在`cr0`中必须为 1、`IA32_VMX_CR0_FIXED1`寄存器中为 0 的值，在`cr0`中必须为 0。`cr4`同理。那么可以通过`cr0 = cr0 | IA32_VMX_CR0_FIXED0 & IA32_VMX_CR0_FIXED1`操作将`cr0`和`cr4`进行置位。具体代码如下。

```cpp
ULONG64 vcr00 = __readmsr(IA32_VMX_CR0_FIXED0);
ULONG64 vcr01 = __readmsr(IA32_VMX_CR0_FIXED1);
ULONG64 vcr04 = __readmsr(IA32_VMX_CR4_FIXED0);
ULONG64 vcr14 = __readmsr(IA32_VMX_CR4_FIXED1);
ULONG64 mcr4 = __readcr4();
ULONG64 mcr0 = __readcr0();
mcr4 |= vcr04;
mcr4 &= vcr14;
mcr0 |= vcr00;
mcr0 &= vcr01;
__writecr0(mcr0);
__writecr4(mcr4);
```

在进行以上步骤之后，可以通过`vmxon`打开柜门。

```cpp
int error = __vmx_on(&pVcpu->VmxOnAddrPhys.QuadPart);
```

## 申请 `vmcs` 内存并 `vmclear`（拔电源）

首先申请`vmcs`内存。和`vmxon`内存类似，这一块内存的头 4 个字节也要填入`IA32_VMX_BASIC`的低 4 个字节。`vmcs`中保存的是虚拟机的各种寄存器和控制区域、VM-exit 控制区域等。在装机过程中要通过`vmwrite`进行`vmcs`的设置。

申请`vmcs`内存并设置头 4 字节为`IA32_VMX_BASIC`的低 4 字节之后，通过`vmclear`指令进行初始化。传入的参数与`vmxon`的参数类似，是指向申请的内存的物理地址的一个指针。`vmclear`的用法如下

```cpp
__vmx_vmclear(&pVcpu->VmxcsAddrPhys.QuadPart);
```

## `vmptrload`（选中机器）

`vmptrload`指令和`vmxon`、`vmclear`类似，传入的是指向`vmcs`结构的物理地址的一个指针。只有执行了这行指令之后才能通过`vmwrite`进行`vmcs`结构的填写。代码如下

```cpp
__vmx_vmptrld(&pVcpu->VmxcsAddrPhys.QuadPart);
```

## 设置 `vmcs`（装机）

![image-20220307084341235](upload/attach/202203/899076_3ZRRMJC3BEJX34P.jpg)

24 章中描述了需要设置的 VMCS 字段。大体包括如下几部分

1. Guest state fields
2. Host state fields
3. vm-control fields
   1. vm execution control
   2. vm exit control
   3. vm entry control

除这 5 个区域之外，`vmcs`还有一个区域是 VM-exit 信息区域。这个区域是只读的，存储的是 VMX 指令失败后失败代码的编号。

**`VMCS`字段的设置是 VT 中最为繁琐的一步。对于 `VMCS`中一些具体字段应该如何进行设置，会在下一篇文章中进行说明。**

## 参考文献

1. Intel 白皮书
2. 邓志《处理器虚拟化技术》
3. B 站周壑 VT 教学视频
