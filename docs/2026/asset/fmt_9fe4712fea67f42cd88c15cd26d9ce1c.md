
# [原创]`ACPI.sys`，从Windows到Bios的桥梁(1)：`windbg`跟踪win7开启ACPI

## 引入

由于windows设备驱动程序被设计成以设备堆栈(Device stack)的形式，下层驱动向上提供支持，所以我们习惯于不加思考的在设备栈的上层使用来自下层驱动的各种已有功能。当我还在做驱动程序时，经常见到这些句话：

A1."功能驱动(Fdo)收到`IRP_MJ_PNP`&`IRP_MN_START`后，会向总线驱动(Pdo)查询并正确配置所需的硬件资源，如IO Port/IRQ/Mem等，然后才开始工作"。这段话我一直感到困惑：功能驱动的硬件资源来自总线驱动，那么总线驱动的资源由谁提供？可能有人会这样释疑：由物理总线仲裁分配。

A2."设备驱动从D0进入D3，保存设备上下文以后，设备(特指硬件上的设备)将进入D3 Sleep状态"。这段话就更困惑了，为什么保存设备上下文(往往是驱动层面上的上下文)后，硬件就能下电了？

A3.OS进入Sleep后，当支持唤醒功能的设备收到中断信号后，唤醒OS。其实，进入Sleep状态后，CPU不会运行任何指令，OS和驱动程序不可能执行；另外，OS作为驱动程序的载体，不可能驱动程序先于OS提前醒来并唤醒OS的情况(驱动程序在梦游？)。那么，所谓的驱动程序Wakeup功能又是指什么？

上面有些问题看似很难有答案，另一些问题的答案又回答的很模糊，像是顺着设备堆栈的思路，把问题抛给更底层。这种不能释怀的情绪伴随了很多年(我简直是带着问题工作！)，直到我转做Bios一段时间，并无意中得到xp泄露的源码后，我觉得我可能释疑了。所以打算用几篇文章输出一下自己的观点。本文暂不回答上面3个问题，只用于引入文章的主角----作为沟通OS和Bios的桥梁，`ACPI.sys`（它是windows对ACPI spec中反复出现的OSPM的平台实现）。windows通过`ACPI.sys`，借助BIOS的实现上述功能A1/A2；GPIO信号跳变后，触发SCI中断，引起Bios执行平台唤醒功能，并在此过程中重新加载OS，最终实现上述功能A3。ACPI如此重要，值得我们好好研究。本文从两个侧面介绍win7 x86上开启ACPI的过程：Bios侧和windows侧。

## Part 1. Bios侧对开启ACPI的支持

根据ACPI spec(Version 6.2)，`FADT`表中的`SMI_CMD`和`ACPI_ENABLE`与ACPI开启有关：

![](https://bbs.kanxue.com/upload/attach/202012/399589_2R5MWT8BUZ8K429.jpg)

如spec所述，Bios在初始化过程中配置完包含`SMI_CMD`和`ACPI_ENABLE`（重点强调，这是Bios设置的，相当于提供给OS的接口。对于同一台机器不论装什么OS，这个值是固定的）的整个ACPI table，然后加载到内存。OS在引导过程中会向`SMI_CMD`指定的端口写入`ACPI_ENABLE`设定的值，触发SMI中断；SMI中断触发后会进入Bios的SMM Handler，由Bios执行开启ACPI所需的设置。到vm上看下这两项的设置：

![](https://bbs.kanxue.com/upload/attach/202012/399589_VSEBYSJKWZTXTJM.jpg)

`SMI_CMD`是`0xB2`，这是一个特殊值，Intel PCH spec上对这个端口有详细说明：当`APMC_En` bit enable后，往`APM_CNT`，就是RW中的`0xB2`写入数值就会引起SMI中断。下面截图选自Intel PCH spec：

![](https://bbs.kanxue.com/upload/attach/202012/399589_UZR494WS5ZDUF99.jpg)

![](https://bbs.kanxue.com/upload/attach/202012/399589_4FD2C8Z957ARMQX.jpg)

UEFI框架中会用下列形式注册SMI Handler，当往`0xB2`端口写入`SwSmiInputValue`时，就会触发SMI中断，并执行SMI处理函数`EnableAcpiCallback`（附注，SMI处理函数的运行对于OS而言是透明的，这句话读者自己细品）

```
  SwContext.SwSmiInputValue = (UINTN) PcdGet8 (PcdAcpiEnableSwSmi);

  Status = SwDispatch->Register (

                         SwDispatch,

                         EnableAcpiCallback,

                         &SwContext,

                         &SwHandle

                         );
```

同理，Bios为了支持windows开启ACPI，也会以同样的方式注册SMI处理函数。对于VMware，它注册的值是`0xF0`，注册后，往ACPI table `Fadt->ACPI_Enable`域写入`0xF0`，之后默默的等windows开启ACPI。

## Part 2. Windows侧开启ACPI

debug前的准备工作：

```
bcdedit /debug on

bcdedit /bootdebug on ;不开这个，windows开启ACPI的瞬间，windbg和目标机的调试会话可能会被重置
```

根据泄露的windows xp源码，可以发现Xp时代，开启ACPI的相关代码位于`ACPIEnableEnterACPIMode`函数中：

```
//busdrv\acpi\driver\shared\acpienbl.c

VOID

ACPIEnableEnterACPIMode (

    VOID

    );

/*++

Routine Description:

    This routine is called to enter ACPI mode

Arguments:

    None

Return Value:

    None

--*/
```

不过这段代码来自Xp，我们调试的目标是win7，调试前得先确认下这部分代码是否存在变化：

```
kd>x ACPI!ACPIEnableEnterACPIMode ;搜索函数名

ACPI!ACPIEnableEnterACPIMode      ;居然函数还在，看来Win7还是用这个函数开启ACPI

kd> bp acpi!ACPIEnableEnterACPIMode ;那就下断点

kd> g

Breakpoint 0 hit

ACPI!ACPIEnableEnterACPIMode:

88cdbce0 8bff            mov     edi,edi
```

当触发断点后，看下调用堆栈等信息，可以看出此时OS处于初始化Phase1阶段。

```
kd> g

Breakpoint 0 hit

ACPI!ACPIEnableEnterACPIMode:

88cc2ce0 8bff            mov     edi,edi

kd> kb

 # ChildEBP RetAddr  Args to Child              

00 8078b644 88cc2e25 00000000 8078b66c 88cfde24 ACPI!ACPIEnableEnterACPIMode

01 8078b650 88cfde24 00000000 82a0f940 88cf5ae0 ACPI!ACPIEnableInitializeACPI+0x1f

02 8078b66c 88cbe556 87d4cc60 87e548a0 00000000 ACPI!ACPIInitialize+0xe2

03 8078b69c 88d050c2 87d4cc60 886e81a0 88d04f38 ACPI!ACPIInitStartACPI+0x6a

04 8078b6c8 88cb927e 87d4cc60 886e8100 87d4cc60 ACPI!ACPIRootIrpStartDevice+0x18a

05 8078b6f8 82a7c11a 87d4cc60 87e548a0 886e8258 ACPI!ACPIDispatchIrp+0x13a

06 8078b718 82f8f531 00000000 87e535d8 87d4b008 nt!IofCallDriver+0x7e

07 8078b734 82a89506 8078b76c 82a892ea 87d4e4a0 nt!PnpAsynchronousCall+0x109

08 8078b7a0 82f976be 82a892ea 87d4e4a0 00000000 nt!PnpStartDevice+0x184

09 8078b7fc 82f96b27 00000012 00000000 87d4e4a0 nt!PnpStartDeviceNode+0x2a6

0a 8078b818 82f8b0b8 00000000 00000000 00000000 nt!PipProcessStartPhase1+0x87

0b 8078ba14 82a87f83 87df5748 00000000 8078ba50 nt!PipProcessDevNodeTree+0x1cc

0c 8078ba5c 82a87e4a 00000000 87d4cab8 00000000 nt!PnpDeviceActionWorker+0x129

0d 8078ba74 832b06ba 00000000 00000000 00000000 nt!PnpRequestDeviceAction+0x15e

0e 8078baec 832ad52b 8080f8c0 00000000 80810e50 nt!IopInitializeBootDrivers+0x414

0f 8078bb6c 832a29b7 8080f8c0 87de25e0 87de2270 nt!IoInitSystem+0x593

10 8078bc4c 82ecd012 8078bc90 8313d0da 8080f8c0 nt!Phase1InitializationDiscard+0xd67

11 8078bc54 8313d0da 8080f8c0 0010209f 00000000 nt!Phase1Initialization+0xd

12 8078bc90 82be6555 82ecd005 8080f8c0 00000000 nt!PspSystemThreadStartup+0x178

13 00000000 00000000 00000000 00000000 00000000 nt!KiThreadStartup+0x19
```

另外，调用堆栈Frame#06有个熟悉的函数调用：`IofCallDriver`，参数显示的`DeviceInst:ACPI_HAL\PNP0C08`，它就是ACPI spec中指定`ACPI.sys`(OSPM)

```
kd> !devstack 87e535d8 

  !DevObj   !DrvObj            !DevExt   ObjectName

  87d4cc60  \Driver\ACPI       87e548a0  

> 87e535d8  \Driver\ACPI_HAL   87e53690  0000006c

!DevNode 87d4b008 :

  DeviceInst is "ACPI_HAL\PNP0C08\0"  ;<---PNP0C08

  ServiceName is "ACPI"
```

![](https://bbs.kanxue.com/upload/attach/202012/399589_WZPYTTXNNFHSG66.jpg)

继续往下调试，会看到OS从全局变量`ACPI!AcpiInformation+0x04`处获得信息，感觉`ACPI!AcpiInformation`是个结构体指针：

```
kd> p

ACPI!ACPIEnableEnterACPIMode+0x81:

88cdbd61 a194bbd088      mov     eax,dword ptr [ACPI!AcpiInformation (88d0bb94)]

 

kd> u ACPI!ACPIEnableEnterACPIMode+0x81

8bf8            mov     edi,eax

a1942bcf88      mov     eax,dword ptr [ACPI!AcpiInformation (88cf2b94)]

8b4004          mov     eax,dword ptr [eax+4] ;<---从结构体偏移取值

660fb64034      movzx   ax,byte ptr [eax+34h] ;<---ACPI!AcpiInformation+4处取得的值仍然是结构体指针

0fb7c0          movzx   eax,ax
```

`ACPI!AcpiInformation`结构体长成什么样,dump出来看一下？

```
kd> dt _ACPIInformation

ACPI!_ACPIInformation

   +0x000 RootSystemDescTable : Ptr32 _RSDT_32

   +0x004 FixedACPIDescTable : Ptr32 _FADT

   +0x008 FirmwareACPIControlStructure : Ptr32 _FACS

   +0x00c DiffSystemDescTable : Ptr32 _DSDT

   +0x010 MultipleApicTable : Ptr32 _MAPIC

   +0x014 GlobalLock       : Ptr32 Uint4B

   +0x018 GlobalLockQueue  : _LIST_ENTRY

   +0x020 GlobalLockQueueLock : Uint4B

   +0x024 GlobalLockOwnerContext : Ptr32 Void

   +0x028 GlobalLockOwnerDepth : Uint4B

   +0x02c ACPIOnly         : UChar

   +0x030 PM1a_BLK         : Uint4B

   +0x034 PM1b_BLK         : Uint4B

   +0x038 PM1a_CTRL_BLK    : Uint4B

   +0x03c PM1b_CTRL_BLK    : Uint4B

   +0x040 PM2_CTRL_BLK     : Uint4B

   +0x044 PM_TMR           : Uint4B

   +0x048 GP0_BLK          : Uint4B

   +0x04c GP0_ENABLE       : Uint4B

   +0x050 GP0_LEN          : UChar

   +0x052 Gpe0Size         : Uint2B

   +0x054 GP1_BLK          : Uint4B

   +0x058 GP1_ENABLE       : Uint4B

   +0x05c GP1_LEN          : UChar

   +0x05e Gpe1Size         : Uint2B

   +0x060 GP1_Base_Index   : Uint2B

   +0x062 GpeSize          : Uint2B

   +0x064 SMI_CMD          : Uint4B

   +0x068 pm1_en_bits      : Uint2B

   +0x06a pm1_wake_mask    : Uint2B

   +0x06c pm1_wake_status  : Uint2B

   +0x06e c2_latency       : Uint2B

   +0x070 c3_latency       : Uint2B

   +0x074 ACPI_Flags       : Uint4B

   +0x078 ACPI_Capabilities : Uint4B

   +0x07c Dockable         : UChar
```

顺带一提，`dt`的输出结果中，从`ACPI!_ACPIInformation!PM1a_BLK`开始到`ACPI!_ACPIInformation!GpeSize`都是ACPI spec中定义的Fixed Hardware Register。

再dump出`ACPI!AcpiInformation->FixedACPIDescTable`的内存，可以发现，其中`smi_cmd_io_port/acpi_on_value`的值和前面RW截图中显示的值一致。
