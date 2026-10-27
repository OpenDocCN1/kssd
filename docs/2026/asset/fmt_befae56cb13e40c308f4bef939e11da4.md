
`SIDT`是 Intel x86 指令，用于检测旧单核处理器上的虚拟机存在。这个指令被 “ScoopyNG” 以及 “Redpill” 使用，来检测虚拟机的存在。该命令将中断描述符表寄存器（`IDTR`）的内容存放到指定内存地址中。中断描述符表（`IDT`）是一种重要的操作系统数据结构，用于处理来自硬件（如键盘和鼠标）的中断命令。hypervisor 会将虚拟化操作系统的 `IDT` 的地址移动到 hypervisor 可预测的地址。Klein 和 Rutkowska 都发现VMware在当时将 `IDT` 存储在地址`0xFFXXXXXX`上，从而在单核处理器上检测 VMware 虚拟机。

然而，随着多核处理器的引入，这种虚拟机检测技术不再可靠，因为处理器内的每个核心都具有唯一的中断描述符表，而每个表的地址都不同。此外，在我们对 VMware 的虚拟机测试`SIDT`命令时，即使只分配了一个处理器核心，`IDT` 的地址也从未出现在`0xFFXXXXXX`的地址上。因此，这种虚拟机检测方法已失效。

## 2.4 `SGDT`（已失效）

`SGDT`是 Intel x86 指令，用于将全局描述符表寄存器（`GDTR`）的内容存放到指定内存地址中。`GDT` 是处理器使用的数据结构，用于定义有关进程内存的详细信息，例如基址、大小和内存权限。这种虚拟机检测方法曾被“ScoopyNG”使用。与`SIDT`指令类似，当在基于VMware的虚拟机中运行时，该指令会返回`0xFFXXXXXX`的值。然而，这个指令也面临着 `SIDT` 指令引入多核处理器后遇到的同样问题。在使用 VMware Workstation 测试这种检测方法时，返回的值并不是`0xFFXXXXXX`。因此，这种检测方法已失效。

## 2.5 `SLDT`（已失效）

`SLDT` 是 Intel x86 指令，用于将本地描述符表寄存器（`LDTR`）的内容存放到指定内存地址。`LDT`包含有关进程内存段的信息，例如代码段、数据段以及堆。当在虚拟机中执行此命令时，将返回非零值，而在非虚拟化环境中返回`0`。“ScoopyNG”和恶意软件“Conficker”曾使用过这种虚拟机检测方法。我们在 VMware Workstation、VirtualBox、XEN、HyperV 和 KVM/QEMU 虚拟机上测试了此命令，发现所有虚拟机都返回`0`。这种检测方法已失效。

## 2.6 `STR`（已失效）

`STR`是 Intel x86 指令，它返回当前执行进程的任务状态寄存器（`TSR`）内容到指定内存地址。任务状态段 `TSS` 用于存放操作系统用于任务切换的数据，其中包括进程 threads 和 registry 信息，供处理器操作系统处理线程调度。Klein 发现在虚拟机内运行此命令时，它返回格式为 `0x0040XXXX` 的地址。对VMware Workstation、VirtualBox、XEN、HyperV和KVM/QEMU虚拟机上测试此命令，发现这种检测方法已失效。

## 2.7 `SMSW`（已失效）

`SMSW`是 Intel x86命令之一，用于将机器状态字返回到指定的内存地址。机器状态字的信息来自`CR0`寄存器，与 CPU 用于指示是否启用某些功能（如内存分页）的 flags 有关。Quist 发现，在VMware虚拟机内运行此命令，当 `EAX=0xCCCCCCCC` 时，将返回 `0x8001XXXX`。经测试在VMware虚拟机内和物理机上测试此命令都会返回值 `0x8005XXXX`。因此这种检测方法已失效。

## 2.8 VM “Synthetic Instructions”（已失效）

Traut在2003年提出了一种使用“合成CPU指令”的虚拟机检测方法。这些指令仅被虚拟机支持，在物理机上执行时会触发无效指令异常。由于这些是无效的CPU指令，因此它们未在Intel或AMD的程序员手册中记录。这些指令的格式为`0FC7C8XXXX`。其中最`VMCPUID`指令被“Necurs”僵尸网络用于检查虚拟机的存在。

| Mnemonic | Opcode | Mnemonic | Opcode |
|:---------:|:--------------:|:--------:|:--------------:|
| `VMCPUID` | `0F C6 28 01 00` | `VMPOPFD` | `0F C6 28 02 01` |
| `VMGETINFO` | `0F C6 28 00 00` | `VMCLI` | `0F C6 28 02 02` |
| `VMSETINFO` | `0F C6 28 00 01` | `VMSTI` | `0F C6 28 02 03` |
| `VMDXDSBL` | `0F C6 28 00 02` | `VMIRETD` | `0F C6 28 02 04` |
| `VMDXENBL` | `0F C6 28 00 03` | `VMSGDT` | `0F C6 28 03 00` |
| `VMHLT` | `0F C6 28 01 01` | `VMSIDT` | `0F C6 28 03 01` |
| `VMSPLAF` | `0F C6 28 01 02` | `VMSLDT` | `0F C6 28 03 02` |
| `VMPUSHFD` | `0F C6 28 02 00` | `VMSTR` | `0F C6 28 03 03` |
| `VMSDTE` | `0F C6 28 04 00` | - | - |

在 VMware Workstation、VirtualBox、XEN、HyperV 和 KVM/QEMU 虚拟化平台上测试 `VMCPUID` 指令的代码如下：

```
bool IsVmcpuidSupported()
{
bool supported = true;
void\* mempool;
char vmcpuid[] = "\x0F\xC7\xC8\x01\x00"; // VMCPUID OPCODE
mempool = malloc(sizeof(vmcpuid));
memcpy(mempool, &vmcpuid, sizeof(vmcpuid));
DWORD prevAccess;
VirtualProtect(mempool, sizeof(vmcpuid), PAGE\_EXECUTE\_READ, &prevAccess);
\_\_try {
((void(\*)())mempool)();
}
\_\_except(EXCEPTION\_EXECUTE\_HANDLER) {
supported = false;
}
return supported;
}
```

结果所有测试的虚拟化平台都不支持 `VMCPUID` 命令。实际上，这个命令被错误地解释为两个命令：

```
CMPXCHG8B EAX
ADD DWORD PTR DS:[EAX],EAX
```

## 3. 其他检测方法

## 3.1 Thermal Zone Temperature

恶意软件“Win32/gravityRAT”采用了一种通过检测系统 thermal zone 来检测虚拟机存在的方法。通过使用下面的powershell脚本，可以轻松检查机器是否支持 `MSAcip\_ThermalZoneTemperature` 对象。如果此脚本没有返回结果，则假定该计算机在虚拟机中，因为虚拟机默认不支持热力监控。由于Thermal zone sensors 可以在系统的BIOS或UEFI中禁用和启用，在多个物理机上测试结果不一致，所以该方法并不可靠。

```
function Get-AntiVMwithTemperature {
$t = Get-WmiObject MSAcip\_ThermalZoneTemperature -Namespace "root/wmi"
$valorTempKelvin = $t.CurrentTemperature / 10
$valorTempCelsius = $valorTempKelvin - 273.15
$valorTempFashrenheit = (9/5) \* $valorTempCelsius + 32
return $valorTempCelsius.ToString() + " C : " + valorTempFashrenheit.ToString() + \
" F : " + $valorTempKelvin.ToString() + "K"
}
```

Ref:

<a href="elink@e69K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6h3u0#2k6$3q4U0N6r3W2$3k6i4m8J5L8$3y4W2M7%4y4Q4x3X3g2E0k6h3c8A6N6h3#2Q4x3X3g2U0L8$3#2Q4x3V1k6S2L8Y4c8A6i4K6u0V1N6X3#2Q4x3X3c8@1k6h3y4Z5L8X3W2I4N6h3g2K6i4K6u0V1N6$3W2@1K9q4)9J5k6r3#2K6j5h3y4H3K9g2)9J5k6s2c8Z5k6i4u0E0j5h3I4\*7L8$3&6W2N6r3g2E0M7r3g2J5j5i4c8#2M7X3g2Q4x3X3b7K6x3X3y4X3k6h3g2U0k6r3p5^5x3o6t1`.">Anti-VM Technique with MSAcpi\_ThermalZoneTemperature</a>

## 3.2 IP Timestamp Patterns

这种虚拟机检测方法是由Noorafize等人发现的 ，它基于虚拟机无法像物理机一样保持准确的时间。虚拟机通常通过与主机计算机共享时间来进行时间同步，以尝试伪装成物理机，但来自虚拟机的IP报文时间戳存在不一致性。研究人员发现，由于虚拟机需要与 hypervisor 进行交互，导致虚拟机的时间戳信息略有不同。为了使用此检测方法，攻击者需要向目标系统发送大量的IP / ICMP 数据包以获取足够的信息，以推断目标机器是否为虚拟机。

Ref:

M. Noorafiza, H. Maeda, T. Kinoshita and R. uda, "Virtual Machines Detection Methods using IP Timestamps pattern Characteristic," vol. 8, 2016

## 3.3 CPU Detail Anomalies

通过检查CPU物理核心数、逻辑核心数和cache容量与实际产品配置的不一致性来检测虚拟机。

Ref：Mettrick D. Virtual machine detection through Central Processing Unit (CPU) detail anomalies[D]. , 2022.

## 3.4 Physical memory resource maps

在 Windows 中，设备驱动程序需要访问硬件资源（如内存、I/O 端口、中断等），这些资源的地址通常是硬件相关的，因此在不同的系统上地址可能会发生变化。为了方便驱动程序的编写和移植，Windows 内核提供了资源映射机制，将硬件相关的地址转换为系统相关的地址，从而使驱动程序可以在不同的系统上运行而不需要进行修改。内存翻译就是其中一种资源映射类型，它将硬件设备的物理内存地址映射为系统中的虚拟内存地址，使驱动程序可以通过虚拟地址来访问设备内存。

可以通过比对虚拟地址的不同检测虚拟机环境。测试发现Hyper-V虚拟机具有与 `00001000-000a0000` 匹配的物理内存映射区域。VirtualBox虚拟机具有与 `00001000-009f000` 匹配的物理内存映射区域。

Ref:

<a href="elink@d75K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9j5h3u0K6i4K6u0W2L8X3g2@1N6r3W2@1N6h3c8W2i4K6u0W2j5$3!0E0i4K6u0r3j5X3I4G2k6#2)9J5c8Y4k6E0i4K6u0V1k6r3g2@1k6h3y4@1K9h3!0F1i4K6u0V1N6s2u0A6j5$3E0K6i4K6u0V1M7r3q4J5N6q4)9J5k6o6q4Q4x3X3c8H3K9s2W2K6K9h3y4S2L8q4)9J5k6r3#2W2L8h3!0J5P5g2)9J5k6s2u0W2M7$3!0#2M7X3y4W2i4K6u0V1L8h3q4H3M7#2)9J5c8R3`.`.">VM Detection Tricks, Part 1: Physical memory resource maps</a>

## 3.5 Driver Thread Fingerprinting

驱动程序可能创建一定数量的线程，并具有可预测的属性，因此这些属性可以用于指纹识别并构建有用的检测启发式算法。

Ref:

<a href="elink@ac9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9j5h3u0K6i4K6u0W2L8X3g2@1N6r3W2@1N6h3c8W2i4K6u0W2j5$3!0E0i4K6u0r3j5X3I4G2k6#2)9J5c8Y4k6E0i4K6u0V1k6r3g2@1k6h3y4@1K9h3!0F1i4K6u0V1N6s2u0A6j5$3E0K6i4K6u0V1M7r3q4J5N6q4)9J5k6o6u0Q4x3X3c8V1M7X3W2$3k6i4u0Q4x3X3c8@1K9s2u0W2j5h3c8Q4x3X3c8X3K9h3&6Y4k6i4u0H3M7X3W2F1N6r3W2F1k6#2)9J5c8R3`.`.">VM Detection Tricks, Part 2: Driver Thread Fingerprinting</a>

## 3.6 基于鼠标行为的方法

恶意软件 Win32/Okrum 使用了基于行为的一些方法来检测沙箱：

- `GetTickCount`函数调用两次，间隔20秒休眠。如果`GetTickCount`值没有改变（即时间已经加速），恶意软件将终止自身。

- 连续两次调用`GetCursorPos`函数。如果鼠标在X轴上的位置发生了变化（即鼠标位置是随机生成的），恶意软件将终止自身。

- 调用`GetGlobalMemoryStatusEx`。如果实际物理内存的数量小于1.5GB，恶意软件将终止自身

- 只有左键（物理）鼠标按钮按下至少三次后（在无限循环中查询`GetAsyncKeyState`），负载才开始。

Ref：

<a href="elink@cd0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2%4k6h3I4A6N6X3g2K6k6h3y4#2M7X3W2@1P5g2)9J5k6h3y4G2L8g2)9J5c8Y4N6H3i4K6u0V1j5$3!0F1N6r3g2F1N6q4)9J5c8Y4g2H3L8r3!0S2k6s2y4Q4x3V1j5J5x3o6p5&6i4K6u0r3x3o6N6Q4x3V1k6q4f1@1g2f1i4K6g2X3e0$3E0J5N6h3#2Q4y4h3k6S2L8X3c8Q4y4h3k6w2k6i4c8J5K9h3y4S2L8W2)9J5k6i4m8V1k6R3`.`.">ESET\_Okrum\_and\_Ketrican.pdf</a>

&nbsp;

## 参考

1. <a href="elink@4dcK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0P5h3&6W2N6q4)9J5k6h3y4G2L8g2)9J5c8X3q4@1N6r3q4U0K9#2)9J5k6s2c8W2j5$3S2F1K9i4q4#2k6i4y4Q4x3X3c8Z5j5h3&6V1M7#2)9J5k6r3!0F1i4K6u0r3L8h3q4D9N6$3q4J5k6g2)9J5k6r3q4F1N6r3W2Q4x3X3c8$3L8g2)9J5k6s2c8W2j5$3S2F1K9i4q4#2k6i4y4Q4x3V1j5`."><mark class="encrypted">a98K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0P5h3&6W2N6q4)9J5k6h3y4G2L8g2)9J5c8X3q4@1N6r3q4U0K9#2)9J5k6s2c8W2j5$3S2F1K9i4q4#2k6i4y4Q4x3X3c8Z5j5h3&6V1M7#2)9J5k6r3!0F1i4K6u0r3L8h3q4D9N6$3q4J5k6g2)9J5k6r3q4F1N6r3W2Q4x3X3c8$3L8g2)9J5k6s2c8W2j5$3S2F1K9i4q4#2k6i4y4Q4x3V1j5`.</mark></a>

2. <a href="elink@81aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2H3N6s2y4W2j5%4g2J5K9i4c8&6i4K6u0W2j5$3!0E0i4K6u0r3N6%4N6Q4x3X3c8W2L8W2)9J5c8X3q4F1j5h3I4&6N6r3W2U0M7#2)9J5c8X3q4F1N6r3W2K6j5h3&6V1j5X3!0^5i4K6u0V1N6r3g2U0K9r3&6A6M7i4g2W2M7#2)9J5c8R3`.`."><mark class="encrypted">faeK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2H3N6s2y4W2j5%4g2J5K9i4c8&6i4K6u0W2j5$3!0E0i4K6u0r3N6%4N6Q4x3X3c8W2L8W2)9J5c8X3q4F1j5h3I4&6N6r3W2U0M7#2)9J5c8X3q4F1N6r3W2K6j5h3&6V1j5X3!0^5i4K6u0V1N6r3g2U0K9r3&6A6M7i4g2W2M7#2)9J5c8R3`.`.</mark></a>
