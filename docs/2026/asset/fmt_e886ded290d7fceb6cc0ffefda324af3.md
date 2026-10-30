
这里最关注的是头戴设备的主板。许多有趣的硬件组件放在上面，他还有许多调试连接口。幸运的是，大多数微控制器不属于GA情况，所以我们必须先找到SWD接口。下图是`LPC11U35F`。

![](https://bbs.kanxue.com/upload/attach/201808/805111_DDDXWMYJAV4JJX3.jpg)

To understand the functions performed by each microcontroller and rebuild connections between the components we used a solderer and a multi-meter.

要知道每个微控制器的功能，重建每个组件间的连接，我们使用一个焊接工具和一个万用表。

![](https://bbs.kanxue.com/upload/attach/201808/805111_G46RMNVPE2Y7JHP.jpg)

We disassembled the motherboard, rang out the paths with our multi-meter, and made the scheme of connection between the components (see Fig. 12).

我们拆散主板，使用万用表激活路径并组合各个组件间的连接（见下图）。

![](https://bbs.kanxue.com/upload/attach/201808/805111_C4295ATJH82N5X6.jpg)

As we remember, the headset has USB-Hub `SMSC USB5537B` that connects up to 7 devices via 1 USB. 6 of the available ports are occupied by the main components of the headset, while the remaining external one is vacant for connecting additional equipment. The scheme illustrates that it is impossible to get into the video output processing from the headset side since it is a PC that is responsible for the procedures. The FPGA monitors the status of 32 headset sensors, processes all the data sent by them, and passes it via the UART to the NXP `LPC11U35F/401` microcontroller.

我们记得，头戴设备有一个USB-hub `SMSC USB5537B`基于一个USB连接最多七个设备。有效接口中的6个连接着头戴设备的主要组件，剩下一个空余来连接附加设备。该结构说明不可能从头戴设备一侧介入视频输出处理过程，当由一个PC负责处理该过程。FPGA监控32个头戴设备传感器的状态，处理它们发送的数据，并通过UART传输到`LPC11U35F/401`微控制器。

### What conclusions can be drawn here?

现在能得到什么结论？

The first thing that came to our heads here is that the video stream cannot be affected in any way since it is processed by a PC, which sends the rendered image to the headset. Consequently, Attack Scenarios 2 and 3 are inapplicable.

首先，当由PC负责处理视频时，视频流无法被任何方法介入，PC直接发送图像到头戴设备。随后，攻击情景2和3无法应用。

Second, we found out that the chip (`NXP LPC11U35F/401`) is responsible for tracking the position of the headset and sends them via USB to a PC. Therefore, a cybercriminal can modify the system that process position data so that a user’s virtual position is different from the real one. This case corresponds to Attack Scenario 1.

其次，芯片`NXP LPC11U35F/401`负责追踪头戴设备位置信息并基于USB发送到PC。因此，网络罪犯可以修改处理位置数据的系统导致用户的虚拟位置与真实位置不相同。这相当于攻击情景1。

Then, we had to find out what firmware was stored on the microcontroller. All microcontrollers and memory had been unsoldered. So, we needed to read data from them. With the help of our soldering station we soldered microcontrollers to the TQFP adapter (see Fig. 13):

我们还知道了固件存储在微控制器中。所有微控制器和内存已经被焊开。所以，我们需要读它们的数据。借助焊接的帮助，我们焊接微控制器到TQFP适配器。

![](https://bbs.kanxue.com/upload/attach/201808/805111_VY87FY5B7UETUAY.jpg)

Unfortunately, we could not find adapters for the SPI memory chips since it had fewer pins. So, we just soldered a pin header to them.

不幸，我们找不到SPI内存芯片的适配器，它的针脚太少。所以，我只焊接了一个针脚到它们。

![](https://bbs.kanxue.com/upload/attach/201808/805111_3MT7NV469MEJBAZ.jpg)

`ChipProg481` and a bunch of wires came in handy for extracting firmware!

`ChipProg481`和一束线用于提取固件。

![](https://bbs.kanxue.com/upload/attach/201808/805111_H8NPTHCFSYVCCMB.jpg)

We extracted data from a controller for a change. However, all the components there were all right, so we did not have to unsolder them. The procedure of extracting data from two microcontrollers was absolutely seamless, thanks to BlackMagick Probe v2 connected to SWDs.

我们改为从一个控制器提取数据。然而，所有组件都是安装好的，我们无需拆开它们。从两个微控制器中提取数据绝对无缝衔接，得益于使用黑科技探针v2连接到SWD。

![](https://bbs.kanxue.com/upload/attach/201808/805111_XASYZBJ686Y2QT2.jpg)

According to the collected data, we built a table to make it clear what firmware was used and what functionality was performed on each microcontroller.

根据搜集的数据，我们构建了一个表清晰说明每个固件的功能和每个微控制器负责的功能。

![](https://bbs.kanxue.com/upload/attach/201808/805111_RWM7B2W67HG5SVU.jpg)

|  |  |  |
| --- | --- | --- |
| 转储源 | StreamVR | 附加信息 |
| Micron N25Q032A13ESE40E 32Mb | Micron N25Q032A13ESE40E 32Mb | 在StreamVR固件中没有找到地址区间0xf800-0xf9c1df间的内容 |
| STM32F072 | DisplayBin tools/lighthouse/firmware/htc/APP-000000000 0200160.bin | 在StreamVR固件中没有找到地址区间0x0-0x26c8间的内容 |
| Micron M25P40 4Mb | DisplayBin tools/lighthouse/fir mware/htc/APP-000000000 0200160.bin | - |
| nRF24LU1P | DisplayBin tools/lighthouse/firmware/vr\_controller/archive/htc\_vrc\_dongle\_1461100729\_2016\_04\_19.bin | - |
| NXP 11U35F | WatchmanBin tools/lighthouse/firmware/lighthouse\_rx\_watchman/archive/htc\_watchman\_1462663157\_2016\_05\_07.bin | 配置文件 |
| Micron N25Q032A13ESE40E 32Mb | WatchmanFPGABin tools/lighthouse/firmware/lighthouse\_rx\_watchman/archive/htc\_pre\_watchman\_262\_fpga.bin | PNG\_1 Green\_4A7A16BB004239\_mura\_analyzes.mc PNG\_2 Green\_4A8A16B8004487\_mura\_analyzes.mc. |

# Security aspect

# 安全形势

The security problems of HTC Vive came to the spotlight quite early – at the updating stage. According to the microcontroller documentation, all microcontrollers support updating via USB. Judging by firmware files, it was clear for us that firmware is a binary file with no checksums. Therefore, adversaries can modify firmwares to their liking and upload them to the headset. Also, each microcontroller has sufficient space (45 Kb on average) for the modified code to be put in there. Change, upload, enjoy!

HTC Vive的安全问题很早就出现了，早到产品处于更新阶段。根据微控制器文档，所有微控制器支持通过USB更新。通过固件文件判断，可以确定固件文件是没有完整性校验的二进制文件。因此，攻击者可以任意修改固件并上传到头戴设备。同样，每个微控制器有足够的空间（平均45 Kb）放置增加的代码。修改-上传-发送有趣的是！

For example, in the NXP LPC11U35 documentation the following is stated:

例如，NXP `LPC11U35`的文档中有如下说明：

- In-System Programming (ISP) and In-Application Programming (IAP) via on-chip bootloader software.

- ROM-based USB drivers. Flash updates via USB supported.

- ROM-based 32-bit integer division routines

- 基于芯片内bootloader软件的系统内编程（In-System Programming (`ISP`)）和应用内编程（In-Application Programming (`IAP`)）

- 基于ROM的USB驱动。支持基于USB快速更新。

- 基于ROM的32位整数分配惯例

and in the `STM32F072` documentation:

`STM32F072`文档内有：

- The boot loader is located in System Memory. It is used to reprogram the Flash memory by

- using USART on pins PA14/PA15, or PA9/PA10 or I2C on pins PB6/PB7 or through the USB

- DFU interface.

- Bootloader位于系统内存。用于通过USART的PA14/PA15针脚或PA9/PA10针脚或PB6/PB7针脚或通过USB DFU接口对flash内存进行重编程

`nRF24LU1P` says:

`nRF24LU1P`文档说:

The `nRF24LU1P` bootloader allows you to program the `nRF24LU1+` through the USB interface. The bootloader is pre-programmed into the `nRF24LU1+` flash memory and automatically starts when power is applied.

`nRF24LU1P`准许通过USB接口对`nRF24LU1P`进行编程。Bootloader预编程在`nRF24LU1P` flash内存，在上电时自动启动。

As it was written above, there is a special `lighthouse_watchman_update.exe` console tool available. A mere malicious code can use it to conduct an attack or implement the functionality of the tool in itself.

正如上面写的，有一个特殊的控制台工具`lighthouse_watchman_update.exe`。恶意代码可以使用它发起攻击或使用它自动的功能。

As seen from `HTC_Vive_Tracker_Developer_Guidelines_v1.3`, the commands below are used for updating the device:

在`HTC_Vive_Tracker_Developer_Guidelines_v1.3`中可以看到，下面的命令用于更新设备：

- Update MCU’s firmware:

  - `lighthouse_watchman_update -mnt tracker_mcu_filename.bin`

- Update FPGA’s firmware:

  - `lighthouse_watchman_update -fnt tracker_fpga_filename.bin`

- Update RF’s firmware:

  - `lighthouse_watchman_update -rnt tracker_rf_filename.bin`

- 更新MCU的固件:

  - `lighthouse_watchman_update -mnt tracker_mcu_filename.bin`

- 更新FPGA的固件:

  - `lighthouse_watchman_update -fnt tracker_fpga_filename.bin`

- 更新RF的固件:

  - `lighthouse_watchman_update -rnt tracker_rf_filename.bin`

If it is possible to update the software of HTC Vive, an attacker can add malicious code that can affect the operation of the system and potentially lead to physical injuries of a device owner or cause other inconveniences.

如果能更新HTC Vive的软件，攻击者可以增加恶意代码影响系统运行，造成使用者受伤或引起其它不便。

# HTC Vive Pro
