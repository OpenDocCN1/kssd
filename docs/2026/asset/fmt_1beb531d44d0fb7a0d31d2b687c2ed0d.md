
# 特斯拉攻击案例解读：硬件逆向分析

## 一. 解读攻击案例的目的

研究攻击案例的主要目的在于“知攻”，也就是搞清楚攻击者是如何攻破车辆、实现控车的。关于对特斯拉的攻击研究，腾讯的科恩实验室贡献了几个成功的案例，按道理讲应该先解读科恩系列。但是，作为一名车联网安全工程师，我对攻击者的完整尝试路径更感兴趣，因为完整的尝试路径包含丰富的对抗信息，包括车辆的哪些防护手段有力地击退了攻击者，哪些防护手段缺失或者存在缺陷，最有效的防护手段是什么等等，而科恩的报告对这部分信息的描述甚少。因此，本文选取Pentest Partners的《Reverse Engineering Tesla Hardware》作为解读系列的开篇，一方面尝试解释清楚Pentest Partners的攻击过程，另一方面，也是我的主要目的，试图发掘真实攻击场景下，车联网安全防护技术的实际防护效果，并探索防护失效的深层次原因。

Pentest Partners的完整报告由两篇文章组成：《Reverse Engineering Tesla Hardware》、《Reverse Engineering the Tesla Firmware Update Process》，前者描述了从拆件到还原硬件架构的过程，后者描述了通过逆向FOTA发现和利用漏洞的过程。本文主要解读第一篇，该篇对车载芯片的拆解和逆向探索的过程记录详细，堪称教科书式的板级逆向。书归正传，文归正叙，下面开始解读。

## 二. Pentest Partners硬件攻击过程

### 2.1. 不设防的物理接触之一：即拆即得的PCB

为了方便逆向，研究员首先将车机拆了下来：

“We took the CID out of the Tesla to power it up and examine it.”

![](https://bbs.kanxue.com/upload/attach/202202/508256_VBCCYCBX7DVDR4N.jpg)

（实物图片均来自Pentest Partners博客，后文不再说明）

拆解后可以看到两块主要的芯片：`IC`和`CID`。`IC`即Instrument Cluster，仪表盘；`CID`即Central Information Display，中央信息显示器，即特斯拉娱乐屏背后的计算芯片。关于`IC`和`CID`的确认过程没有太多的描述，可能是根据公开信息获取，也可能是根据芯片上的标记获取。

![](https://bbs.kanxue.com/upload/attach/202202/508256_6ZPETRBVC9PFQ3U.jpg)

（该步骤还原的架构图，后文亦不再说明）

### 2.2. “自报家门”第一弹：算力担当VCM的信息确认

根据芯片上的文字和标致，研究员确认了`VCM`在`IC`和`CID`两处的存在：

![](https://bbs.kanxue.com/upload/attach/202202/508256_ZUYTQMWA6VPTQ8M.jpg)

“An NVidia VCM (Visual Compute Module) sits at the core of the CID. This is an ARM based single-board computer module that runs Linux – the operating system that drives most of the visible actions of the Tesla.

Nvidia VCM modules are used in both the CID and IC.”

![](https://bbs.kanxue.com/upload/attach/202202/508256_RX7SNTQZNST29HY.jpg)

“An Nvidia Tegra System-on-Chip (SoC) is used in the VCM. This has a multi-core ARM processor, GPU and various other audio and video accelerators.”

`VCM`的`SOC`确认为`Nvidia Tegra`，多核`ARM`处理器，包含`GPU`和`audio`、`vidio`加速器。这部分信息来自`Nvidia Tegra`的文件描述，推测来自网络。

芯片上的标致和文字明确地公开了芯片的身份信息，这种信息至少有助于理解`IC`和`CID`的电器架构。我们可以看出`VCM`为`IC`和`CID`提供了主要的运算能力，`CID`上的`VCM`主要为娱乐屏的图像、视频和音频提供算力支撑。

### 2.3. 第一次挫败“近亲”研究：没有关键共性的“远房亲戚”

确认`VCM`的`SOC`之后，研究员对Tesla `VCM`和`Audi`、`BMW`及`Lamborghini`在使用`Nvidia VCM`上的共性做了简单阐述。

“Audi, BMW and Lamborghini have all been associated with Nvidia VCMs. For example, we uncovered a similar module in an Audi Virtual Cockpit.”

![](https://bbs.kanxue.com/upload/attach/202202/508256_ZEREQM73D5W32VX.jpg)

接着，研究员对`VCM`的算力和其他`IVI`的算力做了一个简单的对比：

“It is significantly more powerful than SoCs used in other IVIs that we have examined, which is probably due to the large CID display. ”

虽然在`VCM`的共性研究和算力对比上没有发现安全的突破口，但千万不要小看这种分析，在逆向分析的过程中，通过对相同或相近对象的比较研究，极有可能发现关键信息。在板级的对比研究中没有获取关键信息，研究员进一步尝试寻找芯片级的“近亲”。

### 2.4. 第二次挫败“近亲”研究：NDA协议的功劳

研究员进一步尝试通过开发用的相近`SOC`，以及使用相同系列芯片的`SOC`寻找关于芯片管脚和布线的信息，但是发现都需要和供应商签署`NDA`才能获取。

“You can obtain technical reference manuals for the Tegra 2, 3 and 4 SoC lines, but they don’t contain ball-maps or pin-outs, so are of limited use for reverse engineering.

To gain access to documentation and forums, a valid developer account was created for Nvidia.com, but we could only find one reference for VCMs or the Jetson Pro development kit. We wrote a forum post asking about documentation but got no response. We envisage that an NDA would have to be signed to gain access to anything significant but it is unlikely that we would be able to share any of the information with third parties.”

### 2.5. “自报家门”第二弹：VCM子模块信息确认

接着研究员对`IC`和`CID`上`VCM`挂载的子模块信息做了一遍梳理，包括`NOR Flash`存储器、`eMMC`存储器、以太网控制器、`USB`收发器、`SDRAM`存储器、电源管理模块等等。其中`IC`和`CID`上的`VCM`配置并不完全一样：

1）`IC`上使用的`Tegra 2`，比`CID`上`Tegra 3`低一个版本；
2）`IC`上`eMMC`容量`4GB`，比`CID`少一半；
3）`IC`上`1GB`的`RAM`，比`CID`少一半。

这些差异表明，`CID VCM`承载的业务负载要高于`IC VCM`，即`CID VCM`为整个车机的主要算力单元。另外，两个`VCM`都挂载了以太网收发器。

### 2.6. 成本战的第一次胜利：阻止对VCM的直接分析

梳理完`VCM`上的子模块之后，研究员的下一步计划是对`VCM`做侵入性分析。虽然研究员已经通过示波器确认了电源引脚，但是`3000`到`10000`英镑的维修成本使研究员放弃了对`VCM`的直接分析。

“We believe that the pin-outs of the two VCMs were the same – voltage supply and ground appeared to be on the same pins, which was confirmed with an oscilloscope. We did not want to direct test the VCMs from the Tesla due to risk of damaging or bricking the CID or IC. This could make the car unusable, and cost somewhere between £3,000 and £10,000 to have repaired by Tesla.”

### 2.7. 被“近亲”研究突突了一小步：VCM固件被提取，零部件生命周期的锅

在前面几次寻找相近型号芯片的尝试失败后，研究员不知从哪淘来一块二手的`VCM`，这块二手`VCM`除了`RAM`容量和`IC VCM`不一样，其他子模块基本完全一致。

“we obtained a third VCM from a second-hand IC. This was the same as the VCM in the IC from the test, except it had 50% less RAM. The IC itself was a different model. We conducted invasive testing”

![](https://bbs.kanxue.com/upload/attach/202202/508256_T4PUWQQHBRVV2RY.jpg)

研究员从这块二手`TCM`的`Nor Flash`中提取了第一、第二阶段的`bootloader`，`kernel`以及`boot`相关的数据，其他文件系统相关的数据则存在于`NAND Flash`中，文件系统的其余部分位于`NAND`闪存中。大多数文件存储在压缩的只读文件系统 (`squashfs`) 中，小部分存储在读/写文件系统 (`ext4`) 中。

### 2.8. “自报家门”第二弹：SWITCH信息确认

通过`Switch`芯片上的文字和标致信息，研究员确认了`Switch`为`Marvell 88E6061`，支持`VLAN`：

![](https://bbs.kanxue.com/upload/attach/202202/508256_GFD6NTJQRY59V2E.jpg)

“A built-in Ethernet switch connects the VCM to the rest of the vehicle via a Marvell 88E6061. This five-port 10/100 switch supports VLANs. It is configured using SMI, a serial interface. The Ethernet switch is configured when the CID starts.”

### 2.9. 通信协议的第一次破防：可以被嗅探和篡改的SMI

`SMI`通信缺乏有效的安全措施，研究员可以通过逻辑分析仪嗅探`SMI`命令，并篡改`SMI`指令。虽然通过`VLAN`划分了广播域，但由于`SMI`命令缺乏完整性保护，导致可以通过篡改指令修改`VLAN`配置。

“A logic analyser could sniff the SMI commands. We could not ascertain whether they were sent by the VCM or the gateway processor. Traffic is segregated from the various components using several VLANs.

There is no security functionality associated with the SMI, for example integrity protection – it trusts all signals, meaning that communications can be tampered with or modified. VLAN configuration could be modified in this way, potentially creating new routes into the system. ”

### 2.10. “自报家门”第三弹：网关信息确认

通过网关芯片上的文字和标致信息，研究员很容易就确认了网关芯片的型号：

“Here we show the gateway with CAN transceivers. Similar devices are often found in CAN gateways. The gateway is connected to the CAN buses and the Ethernet switch. We believe the firmware on the gateway is FreeRTOS, a light-weight real-time operating system (RTOS) with multiple automotive uses.”

![](https://bbs.kanxue.com/upload/attach/202202/508256_Q99CTCCHV93E8UN.jpg)

![](https://bbs.kanxue.com/upload/attach/202202/508256_2GJN6ZA7NHBETQP.jpg)

### 2.11. “自报家门”第四弹：Modem信息确认

同样地，通过`Modem`芯片上的文字和标致，研究员很容易就确认了`Modem`的型号。

![](https://bbs.kanxue.com/upload/attach/202202/508256_GJXCVGZBWZCQAPK.jpg)

### 2.12. 软件架构的胜利：无法被找到的eCALL

研究员尝试从`CID`中寻找`eCALL`和短信相关的应用，但是没有发现任何踪迹，因此推测相关应用功能的实现均在`Modem`中。由于`Modem`没有暴露攻击接口，因此没有进一步对`Modem`进行分析。

“Most TCUs in vehicles do not allow user-accessible and swappable SIM. Embedded or inaccessible SIMS are becoming the norm. Despite further examination of the Tesla, we did not uncover any functionality for tracking or eCall running on the CID’s VCM. This suggests these safety and security features run directly on the cellular modem, as per TCUs in other vehicles.”

从攻击者的思路反思，我们在做正向设计的时候，应该尽量避免复杂的架构，好的设计是将同一功能的模块封装在同一个芯片中，并尽量减少芯片的攻击面。在实际的设计中，`eCALL`等功能模块（的部分功能）经常被放到`IVI`中（类似于这里的`CID`），这种设计明显增加了`eCALL`及`Modem`被攻击的可能性。

### 2.13. 芯片封装方式的第一次胜利：BGA阻止对Modem的侵入式分析

由于`Modem`芯片的封装方式使用的是`BGA`，这种封装方式的特点是拆卸难度和重新挂载的难度都很高，因此研究员放弃了从`Modem`芯片中还原固件的尝试。

“Recovering firmware from a Sierra Wireless AR8522 module without causing damage is extremely difficult, being as it is a tightly packed BGA module with numerous BGA chips, all contained under a soldered-on shield”

![](https://bbs.kanxue.com/upload/attach/202202/508256_A8WZS4KFWN5CZHF.jpg)

### 2.14. 芯片封装方式的第二次胜利：面朝下的WIFI和蓝牙模块

由于`WIFI`和蓝牙模块的芯片均采用正面朝下的封装，这种封装方式有效地隐藏了芯片的文字信息，因此攻击者无法直接获取芯片型号。再加上芯片使用`BGA`的封装方式（根据上下文可以推测其封装方式也为`BGA`），拆解难度极大，因此有效地阻止了研究员的进一步攻击。

“The CID employs a Parrot Wi-Fi and Bluetooth module, mounted face-down. We could not confirm the specific model because it could not be removed from the main board.”

![](https://bbs.kanxue.com/upload/attach/202202/508256_JSQFBCD3DMN62KW.jpg)

### 2.15. 不设防的物理接触之二：即拆即用的SD卡

`SD`卡没有完整性校验机制，存储升级期间使用的`firmware`，并且根据文章的描述，`SD`卡的存储内容应当为明文存储，这意味着攻击者可以很容易地从`SD`卡提取数据及向`SD`卡中写入数据。在第二阶段的攻击过程中，攻击者很好地利用了`SD`卡的不设防。
