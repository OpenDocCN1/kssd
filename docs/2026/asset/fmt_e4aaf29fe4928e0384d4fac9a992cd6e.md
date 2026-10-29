
# [原创]黑苹果Hackintosh 10.13.5 High Sierra i7 8700k z370

## 一、配置

* CPU i7 8700K
* 主板 技嘉（GIGABYTE） Z370 AORUS Ultra Gaming
* 内存 G.SKILL DDR4 3200MHz 8G*2
* SSD 三星 SM961 256G
* 显卡 七彩虹（Colorful） GTX1070Ti Vulcan X TOP 8G
* WiFi&Bluetooth `BCM943602CS`
* 显示器 DELL P2715q 4K

先说说配置，轻度游戏玩家，比如我，强烈建议上INTEL CPU + 技嘉Z370 + A卡（4K入门建议蓝宝石的RX560），原生驱动是很爽的，就像我在这里使用的蓝牙是`BCM943602CS`一样，其实其他品牌估计30块钱就搞定了，多花了120，但是这里我选择是原生支持的BCM，对于新手来说，如果你进系统的第一时间能使用会省你不少麻烦(尤其是`BCM943602CS`直接能WiFi上网,Air Drop传文件，后面发现网卡也是直驱的，这钱花的没什么卵用。。。吃土.jpg)。

### 1. 下载系统

系统盘有很多种做法，APP STORE上下载下来，会到你的Application里有一个安装程序。

下面这篇文章主要介绍通过 `UniBeast`（下载好安装包可以用它制作安装盘） 和 `MultiBeast`（帮你给U盘开EFI，并且根据你指定的硬件配置相关驱动） 来制作装载盘。

*讲道理这其实就够了，中途我绕了一些弯路，也学了不少工具，其中 `Clover`(会帮你给你指定的盘制作EFI分区，和`MultiBeast`非常类似) 和 `clover configurator`（可以帮你找到指定盘的EFI，非常具体的设置你的`config.plist`，还有说明） 是相当优秀的工具，优化你的系统时一定会用到（非必须）*

`cf6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2@1L8$3&6&6L8h3q4U0P5o6R3$3i4K6u0W2j5$3!0E0i4K6u0r3N6r3S2J5k6h3q4V1M7#2)9J5c8Y4g2F1K9h3u0W2j5i4y4@1i4K6u0V1K9h3&6K6N6r3q4D9L8q4)9J5k6r3#2S2j5$3!0K6i4K6u0V1K9r3W2Y4K9q4)9J5k6s2y4A6k6i4u0J5j5g2)9J5k6r3!0F1i4K6u0V1j5h3&6&6i4K6u0V1M7%4g2H3M7r3!0J5N6r3g2V1i4K6u0V1K9h3&6@1k6h3I4Q4x3X3c8T1j5i4y4W2k6q4)9J5k6s2m8U0i4K6u0W2x3U0x3#2y4o6M7@1i4K6u0r3`

## 二、安装

### 2. BIOS推荐设置

```
Save & Exit >Load Optimized Defaults
M.I.T. >Advanced Memory Settings Extreme Memory Profile (X.M.P.) : Profile1
BIOS > FastBoot: Disabled
BIOS > LAN PXE Boot Option ROM: Disabled
BIOS > Storage Boot Option Control: UEFI
BIOS > CSM Support [Disabled]
Peripherals > TrustedComputing > Security Device Support:Disable
Peripherals > Network Stack Configuration > Network Stack:Disabled
Peripherals > USB Configuration > Legacy USB Support: Auto
Peripherals > USB Configuration > XHCI Hand-off: Enabled
Chipset > Vt-d: Disabled
Chipset > Wake on LAN Enable: Disabled
Chipset > IOAPIC 24-119 Entries: Enabled
Peripherals > Initial Display Output: PCIe 1 Slot
Chipset > Integrated Graphics: Disabled （这个我好像设置的auto，不重要）
```

### 3. MultiBeast

```
Quick Start > UEFI Boot Mode
Audio: Leave All Unchecked.Skip installing any audio kexts for now (more on that later)
Disk: Leave All Unchecked.
Misc: Leave the FakeSMC (greyed out) option as the only install
Network: I selected IntelMausiEthernet.kext
USB: I selected 7/8/9 USB Support
Bootloaders >Clover UEFI Boot Mode
Customize: System Definition > iMac14,2​
```

EFI里有个`config.plist` 关注一下 boot，argument有没有 `-v`，可以帮你输出log，有时装不下去可以有错误信息去搜索。

### 4. 安装步骤

* F12 后面一直用UEFI 进U盘（装载盘），弹clover引导页面选择从U盘安装系统，进去会是白苹果走一半会提示熟悉的安装界面，安装很快，2分钟左右自动重启，硬盘里面已经有装一半的系统了。
* 这时候clover选从第二个，从硬盘启动安装macOS，这个大概20分钟，后面再重启一次就安装好了。
* 第三次根据macOS的引导进行用户名等设置，就进系统了。之前选的`BCM943602CS`，这时就能直使用WiFi，下载驱动，省的U盘拷来拷去。
* 接下来画面很戳，因为没安装显卡驱动，安装完重启，世界又清晰了。`Nvidia驱动链接`
* 声卡，网卡驱动配置（完全不用配置直接能用...）

### 5. 关闭下载程序验证

打开终端，输入 `sudo spctl --master-disable`，回车，输入密码

### 6. Z370 USB3.1（非必须）

`Kext Placement`

1. Open `Clover Configurator`
2. Select `Mount EFI` under TOOLS
3. Click `Mount Partition` for your boot drive
4. Click `Open Partition` to open your EFI folder.
5. Open your `config.plist` from your EFI folder using `Clover Configurator`
7. Click `Kernel and Ket Patches`
8. Under the `KextstoPatch` tab, make sure you have the following three patches (you can copy/paste below):

![图片描述](upload/attach/201902/93_YERXKMF626S3R7F.png)

USB设置

*Patch #1*

* Name*: `AppleAHCIPort`
* Find*: `45787465 726E616C`
* Replace*: `496E7465 726E616C`
* Comment*: `External Icons Patch`

*Patch #2*

* Name*: `com.apple.driver.usb.AppleUSBXHCI`
* Find* [HEX]: `837d940f 0f839704 0000`
* Replace* [HEX]: `837d940f 90909090 9090`
* Comment: `Remove USB port limit in XHCI kext`

*Patch #3*

* Name*: `com.apple.driver.usb.AppleUSBXHCIPCI`
* Find* [HEX]: `837D8C10`
* Replace* `[HEX]: 837D8C1B`
* Comment: `Change 15 port limit to 26 in 10.13 XHCI kext`

## 给硬盘做一个ESP引导，进系统摆脱U盘（装载盘）

`clover configurator` 里面 `Mount EFI` -> `Mount Partition` -> `Open Partition` 打开你U盘（装载盘）里正常能跑起系统的EFI拷出来，再用同样的方法打开你的本地硬盘，把U盘的EFI放进去，从此你就摆脱U盘了，不过这个U盘在你胡乱设置挂了以后，可以把你的系统载起来，然后把你备份的EFI恢复起来，建议你留一个这样的装载盘在身边。

![图片描述](upload/attach/201902/93_QFCEFC9AFNTZS5M.png)

## 其他问题

因为大名鼎鼎APPLE居然不认识i7 8700k显示unknow，BBEdit 修改 “UnknownCPUKind” 为“Intel Core i7-8700K”。*我比较无聊，就设置了i9 7900X 反正又不要钱 吐舌.jpg*

![图片描述](upload/attach/201902/93_UPGS8RRZG7ADTMS.png)

来自星星的Mac Pro

```
/System/Library/PrivateFrameworks/AppleSystemInfo.framework/Versions/A/Resources/zh_CN.lproj/AppleSystemInfo.strings
```

请一定记住在`clover configurator`改任何设置，一定要提前保存EFI文件，即使系统进不去了，可以通过U盘引导进去以后恢复EFI。

内存显示错误或者打不开“关于本机”，在`clover configurator`修改在SMBIOS，设置正确的内存型号

## 性能测试

### Benchmarks

![](upload/attach/201807/738665_HNCKJZSC22YMX8Z.jpg)

![](upload/attach/201807/738665_7EX4FJEVRZKUNCJ.jpg)

#### Mac性能天梯图对比一下（只放前几名吧）

##### 单核

![](upload/attach/201807/738665_QT6VF46WATPK3B4.png)

##### 多核

![](upload/attach/201807/738665_DPTCWEJ87SD7UU6.png)

PS:Xeon 核心多是叼啊...

### Cinebench

![](upload/attach/201807/738665_3DC7D8HB9NSGDRS.png)

![](upload/attach/201807/738665_55GHHZURRTDVVAB.jpg)

### 磁盘性能

![](upload/attach/201807/738665_FH8H5529NNHBF8M.jpg)

讲道理这个SM961性能真心爆棚，可惜MacOS现在还不支持SSD Raid0。刚好这块Z370有2个M.2接口，WIN10下两块SM961写入速度稳稳2900MB/s，读取3200MB/S。

## 参考:

`ef9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2@1L8$3&6&6L8h3q4U0P5o6R3$3i4K6u0W2j5$3!0E0i4K6u0r3N6r3S2J5k6h3q4V1M7#2)9J5c8Y4y4#2j5$3y4W2M7%4y4Q4x3X3c8Y4K9h3N6S2j5Y4W2@1k6g2)9J5k6s2Z5K6y4K6m8Q4x3X3c8Y4j5h3#2A6L8X3N6Q4x3X3b7%4i4K6u0V1K9h3&6@1k6h3I4Q4x3X3c8U0L8%4u0W2i4K6u0V1K9e0N6Q4x3X3b7^5y4K6l9H3K9#2)9J5k6s2u0^5i4K6u0V1y4e0R3H3i4K6u0V1x3Y4S2Q4x3X3c8V1k6h3I4D9i4K6u0V1M7o6t1%4x3e0g2I4i4K6u0V1y4r3E0Q4x3X3b7$3x3r3S2*7i4K6u0W2x3U0f1J5z5e0R3&6i4K6u0r3`

`c7dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0L8r3W2E0j5X3A6G2k6g2)9J5k6i4c8$3i4K6u0r3L8h3q4U0L8%4y4Q4x3X3c8Z5K9h3N6Z5i4K6u0V1M7$3W2W2M7Y4u0S2i4K6u0V1N6X3g2J5M7$3W2G2L8W2)9J5k6o6p5H3i4K6u0V1x3e0y4Q4x3X3b7J5i4K6u0V1j5$3!0X3k6X3g2W2i4K6u0V1L8r3q4C8k6g2)9J5k6s2y4&6M7%4c8W2L8g2)9J5k6r3u0#2K9h3I4V1i4K6u0V1K9h3&6@1M7X3!0Q4x3V1j5`.`

性能天梯

`8e0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1M7X3!0%4M7$3g2J5i4K6u0W2k6$3g2W2K9$3u0W2L8X3y4Z5i4K6u0W2j5$3!0E0i4K6u0r3L8h3q4U0i4K6u0V1j5X3g2F1j5$3S2E0j5i4u0C8M7H3`.` .`

## 感谢：

*种地老头* ，*雨生* 和 *黑苹果研究学院（QQ群）*提供的帮助，探索还在继续...

### 一、配置：

- CPU i7 8700K
- 主板 技嘉（GIGABYTE） Z370 AORUS Ultra Gaming
- 内存 G.SKILL DDR4 3200MHz 8G*2
- SSD 三星 SM961 256G
- 显卡 七彩虹（Colorful） GTX1070Ti Vulcan X TOP 8G
- WiFi&Bluetooth `BCM943602CS`
- 显示器 DELL P2715q 4K

先说说配置，轻度游戏玩家，比如我，强烈建议上INTEL CPU + 技嘉Z370 + A卡（4K入门建议蓝宝石的RX560），原生驱动是很爽的，就像我在这里使用的蓝牙是`BCM943602CS`一样，其实其他品牌估计30块钱就搞定了，多花了120，但是这里我选择是原生支持的BCM，对于新手来说，如果你进系统的第一时间能使用会省你不少麻烦(尤其是`BCM943602CS`直接能WiFi上网,Air Drop传文件，后面发现网卡也是直驱的，这钱花的没什么卵用。。。吃土.jpg)。

1. 下载系统

系统盘有很多种做法，APP STORE上下载下来，会到你的Application里有一个安装程序。

下面这篇文章主要介绍通过**`UniBeast`**（下载好安装包可以用它制作安装盘） 和 **`MultiBeast`**（帮你给U盘开EFI，并且根据你指定的硬件配置相关驱动） 来制作装载盘。

***讲道理这其实就够了，中途我绕了一些弯路，也学了不少工具，其中** `Clover`(会帮你给你指定的盘制作EFI分区，和`MultiBeast`非常类似) 和 `clover configurator`（可以帮你找到指定盘的EFI，非常具体的设置你的`config.plist`，还有说明）* 是相当优秀的工具，优化你的系统时一定会用到（非必须）*

`9fcK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2@1L8$3&6&6L8h3q4U0P5o6R3$3i4K6u0W2j5$3!0E0i4K6u0r3N6r3S2J5k6h3q4V1M7#2)9J5c8Y4g2F1K9h3u0W2j5i4y4@1i4K6u0V1K9h3&6K6N6r3q4D9L8q4)9J5k6r3#2S2j5$3!0K6i4K6u0V1K9r3W2Y4K9q4)9J5k6s2y4A6k6i4u0J5j5g2)9J5k6r3!0F1i4K6u0V1j5h3&6&6i4K6u0V1M7%4g2H3M7r3!0J5N6r3g2V1i4K6u0V1K9h3&6@1k6h3I4Q4x3X3c8T1j5i4y4W2k6q4)9J5k6s2m8U0i4K6u0W2x3U0x3#2y4o6M7@1i4K6u0r3`

### 二、安装

1. BIOS推荐设置
