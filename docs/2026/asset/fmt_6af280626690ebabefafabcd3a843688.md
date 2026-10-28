
- Connor McGarr (@33y0re)的博客 - <a href="elink@56eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0L8$3&6F1L8%4u0E0j5$3N6S2M7Y4u0Q4x3X3g2Y4K9i4c8Z5N6h3u0Q4x3X3g2A6L8#2)9J5c8X3S2$3j5$3W2Q4x3V1j5`.">漏洞开发：没有代码执行？没问题！活在VBS、HVCI和内核CFG时代</a>

希望您从中学到了一些东西，感谢您的阅读！

## 额外的努力：“内核遍历，获取MaxConnections的10个步骤”

好吧，您很好奇，这种黑客精神很棒！想知道如何获取MaxConnections值吗？下面的方法展示了如何使用Windbg获取名为`bindflt.sys`的驱动程序的MaxConnections值。作为提醒/帮助，请查看下面的徒步旅行地图。

![图片描述](upload/attach/202310/876202\_DEGK3ASD4MABMWG.webp)

### 第0步 - 确定入口点

起点是名为`FLTMGR!FltGlobals`的结构。您可以直接获取地址：

```
kd> ? FLTMGR!FltGlobals
```

但是，如果您需要使用内存泄漏来检索此地址，以下路径非常有效：

- 获取`fltmgr.sys`的起始地址

```
lmdvm fltmgr
```

- 获取`FltEnumerateFilters`函数的偏移量，并在该函数中获取`lea rcx, [FLTMGR!FltGlobals+0x58]`指令的偏移量

```
u FLTMGR!FltEnumerateFilters L15
```

- 计算`FLTMGR!FltGlobals`的起始地址：`0xfffff8061ea8b600`

```
? fffff806`1ea8b658 - 0x58
```

这样就可以计算出`FLTMGR!FltGlobals`的起始地址。

![图片描述](upload/attach/202310/876202\_YAERQ6YBBYM42VB.webp)

### 第1步 - 计算存储在FLTMGR!FltGlobals中的FrameList字段地址：0xfffff8061ea8b6c0

```
kd> ? fffff806`1ea8b600 + 0x58 + 0x68
```

### 第2步 - 使用指针间接引用并获取FLTMGR!\_FLTP\_FRAME中第一个帧（Links字段）的地址：0xffffca0c38c61058

```
kd> ? poi(fffff806`1ea8b6c0)
```

### 第3步 - 计算第一个帧的起始地址：0xffffca0c38c61050

```
kd> ? ffffca0c`38c61058 - 0x008
```

### 第4步 - 计算过滤器列表的地址：0xffffca0c38c61100

```
kd> ? ffffca0c`38c61050 + 0x48 + 0x68 + 0x000
```

### 第5步 - 使用指针间接引用并获取第一个过滤器的地址（PrimaryLink字段）：0xffffca0c386e8020

```
kd> ? poi(ffffca0c`38c61100)
```

### 第6步 - 计算第一个过滤器的地址（FLTMGR!\_FLT\_FILTER的基址）：0xffffca0c386e8010

```
kd> ? ffffca0c`386e8020 - 0x010
```

好的，很好。我们可以使用Windbg过滤器内核调试器（`fltkd`）命令来验证和可视化我们所在的位置，该命令名为`frames`：

![图片描述](upload/attach/202310/876202\_XKTSCRYYAXA9B8N.webp)

### 第7步 - 计算服务器端口列表的地址：0xffffca0c386e8250

```
kd> ? ffffca0c`386e8010 + 0x208 + 0x038 + 0x000
```

### 第8步 - 使用指针间接引用并获取第一个服务器端口对象的地址（FLTMGR!\_FLT\_SERVER\_PORT\_OBJECT的FilterLink地址）：0xffffca0c3eaf73f0

```
kd> ? poi(ffffca0c`386e8250)
```

### 第9步 - 计算MaxConnections字段的地址：0xffffca0c3eaf7430

```
kd> ? ffffca0c`3eaf73f0 +0x040
```

### 第10步 - 使用指针间接引用并获取MaxConnections的值：1000

```
kd> .formats poi(ffffca0c`3eaf7430)
```

或者

```
kd> dt \_FLT\_SERVER\_PORT\_OBJECT ffffca0c`3eaf73f0
```

最后！您成功了！

![图片描述](upload/attach/202310/876202\_3YVXS9TPA3SDJ2N.webp)

最后的步骤 - 将MaxConnections的值设置为零并终止`EDR_Process.exe`进程

```
kd> eq 0xffffca0c3eaf7430 0
```

然后

```
kd> !process 0 0 MsMpEng.exe
PROCESS ffffa40a23a5f340
kd> .kill ffffa40a23a5f340
```

由于对我来说，在内核内存中可视化自己所处的位置、数据结构的字段、数据结构之间的链接、应该使用的偏移量等等，这真是一场噩梦。因此，我制作了下面的地图（还包括Windbg命令）。

![图片描述](upload/attach/202310/876202\_24CSX5WARU2QW22.webp)

也许（就像我一样），您想知道那些用于计算的值是什么：是的，那些是偏移量。

偏移量可能会在Windows更新后发生变化，个人我使用了EDRSandblast的`ExtractOffsets.py`脚本的自定义版本（在EDRSnowblast中可用）。下面显示了输出示例。

![图片描述](upload/attach/202310/876202\_YKGQNKPBR45E4EA.webp)

因为我们知道方法和偏移量，我们可以自动化这个过程！PoC或者GTFO，可以在<a href="elink@797K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6a6M7X3q4F1k6$3g2Q4x3X3c8o6P5h3u0W2M7X3c8W2k6X3g2F1M7$3g2Q4x3V1k6q4c8q4u0e0L8X3!0%4j5X3I4S2M7%4c8Q4x3V1k6T1L8r3!0T1i4K6u0r3z5e0V1#2x3K6t1J5z5e0p5^5x3r3b7K6j5$3x3#2y4o6p5K6x3X3p5K6y4K6c8X3y4o6u0U0y4X3b7&6y4K6M7#2z5o6u0T1k6e0W2S2j5W2)9J5c8V1g2p5f1W2y4S2L8X3c8T1L8r3q4K6N6q4)9J5c8W2g2@1K9h3I4K6i4K6u0r3c8X3I4@1L8h3N6J5e0$3k6X3M7$3g2@1M7#2)9J5k6h3y4Q4x3U0y4x3x3e0j5%4">EDRSnowblast</a>中找到执行此操作的代码。

祝您愉快的黑客之旅！

原文标题：Filter-Mute Operation: Investigating EDR Internal Communication

原文链接：[931K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6k6h3&6K6k6i4m8G2M7%4c8Q4x3X3g2U0L8$3#2Q4x3V1k6T1L8r3!0Y4i4K6u0r3x3U0l9J5x3#2)9J5c8X3k6A6L8s2c8W2M7W2)9J5k6r3#2#2N6r3g2Q4x3X3c8G2M7r3g2J5j5i4c8A6L8$3&6Q4x3X3c8A6L8Y4k6W2M7%4c8A6k6$3q4@1K9h3&6Y4i4K6u0V1k6h3c8J5i4K6u0V1K9h3&6@1k6i4u0F1j5h3I4Q4x3X3c8U0L8$3#2E0N6h3&6A6j5$3q4@1K9h3!0F1i4K6u0r3](https://bbs.kanxue.com/elink@91aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6k6h3&6K6k6i4m8G2M7%4c8Q4x3X3g2U0L8$3#2Q4x3V1k6T1L8r3!0Y4i4K6u0r3x3U0l9J5x3#2)9J5c8X3k6A6L8s2c8W2M7W2)9J5k6r3#2#2N6r3g2Q4x3X3c8G2M7r3g2J5j5i4c8A6L8$3&6Q4x3X3c8A6L8Y4k6W2M7%4c8A6k6$3q4@1K9h3&6Y4i4K6u0V1k6h3c8J5i4K6u0V1K9h3&6@1k6i4u0F1j5h3I4Q4x3X3c8U0L8$3#2E0N6h3&6A6j5$3q4@1K9h3!0F1i4K6u0r3)

目录

- [简介](https://bbs.kanxue.com/#简介)
- [EDR\_Process.exe和EDR\_Driver.sys角色概述](https://bbs.kanxue.com/#edr_processexe和edr_driversys角色概述)
- [使用已知技术来遮蔽EDR](https://bbs.kanxue.com/#使用已知技术来遮蔽edr)
- [使用另一种方法来遮蔽EDR](https://bbs.kanxue.com/#使用另一种方法来遮蔽edr)
- [步骤1：重置MaxConnections的值](https://bbs.kanxue.com/#步骤1：重置maxconnections的值)
- [步骤2：强制EDR重新启动](https://bbs.kanxue.com/#步骤2：强制edr重新启动)
- [步骤3：观察EDR行为](https://bbs.kanxue.com/#步骤3：观察edr行为)
- [执行计划](https://bbs.kanxue.com/#执行计划)
- [如何保护/检测？](https://bbs.kanxue.com/#如何保护检测？)
- [资源和致谢](https://bbs.kanxue.com/#资源和致谢)
- [额外的努力：“内核遍历，获取MaxConnections的10个步骤”](https://bbs.kanxue.com/#额外的努力：内核遍历，获取maxconnections的10个步骤)

对于我们在2023年举办的年度内部黑客大会SenseCon，我决定研究一下Windows驱动程序与其用户模式进程之间的通信。以下是这个过程的一些细节。

## 简介

攻击者可以利用Windows内核的读/写漏洞原语来避免`EDR_Driver.sys`与其`EDR_process.exe`之间的通信。结果是，一些EDR检测机制将被禁用，使其（部分地）对恶意有效负载变得盲目。本博文描述了一种替代方法，该方法不移除内核回调，并提供了一些建议，以防止这种“过滤-静音”攻击。

## EDR\_Process.exe和EDR\_Driver.sys角色概述

首先要弄清楚的问题是，EDR应用程序（`EDR_Process.exe`）如何与其EDR驱动程序（`EDR_Driver.sys`）进行通信？

在进行研究之前，我们必须了解一些EDR的基础知识；EDR代理程序在进程创建期间如何进行钩子/注入自己的DLL？

来自[Christopher Vella的EDR观察](https://bbs.kanxue.com/elink@56fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2K6K9h3N6F1j5h3I4Q4x3X3c8D9j5h3u0K6i4K6u0W2j5$3!0E0i4K6u0r3j5X3I4G2k6#2)9J5c8X3g2V1M7W2)9J5k6r3!0T1M7$3g2J5N6X3q4@1K9h3!0F1M7H3`.`.)中采用的**回调注入**模式是一个很好的总结。

 ![图片描述](https://bbs.kanxue.com/upload/attach/202310/876202_6BET9G9A56UEVKG.webp)

我对正在发生的事情添加了一些注释：

1. `EDR_Driver.sys`可以订阅多种类型的内核通知。你可以将这些通知想象成你在互联网上订阅并通过电子邮件从网站收到的“新闻通讯”。例如，`EDR_Driver.sys`可以使用名为`PsSetCreateProcessNotifyRoutine`的Windows API订阅“新进程创建”通知服务，然后对系统创建的每个进程，驱动程序将接收到有关该进程的信息（父进程ID、命令行等）。

2. 用户双击运行`malware.exe`。

3. Windows调用`CreateProcessW` API将`malware.exe`加载到内存中。

4. `EDR_Driver.sys`收到一个通知，表示`malware.exe`将被生成。

5. `EDR_Driver.sys`向`EDR_Process.exe`发送日志，告知“嘿！一个名为`malware.exe`的新进程即将启动。”

6. `EDR_Process.exe`可以选择采取行动（或不采取行动）：“好的，我将通过在`ntdll.dll`中创建钩子来监控这个进程。”

7. 当`malware.exe`运行时，它调用Windows API。通过已经放置的钩子，`EDR_Process.exe`知道哪些API被调用，并可以推断出`malware.exe`正在做什么。

我们可以以[ired.team](https://bbs.kanxue.com/elink@c96K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2A6M7X3g2V1i4K6u0W2N6r3g2S2L8g2)9J5c8X3!0X3k6X3g2F1M7$3W2$3k6g2)9J5k6s2y4W2j5%4g2J5K9i4c8&6i4K6u0r3j5$3!0V1k6g2)9J5k6r3W2F1K9X3g2U0N6r3W2G2L8W2)9J5k6s2m8J5L8$3y4W2M7%4y4Q4x3X3c8A6L8X3A6W2j5%4c8A6L8$3&6Q4x3V1k6H3M7X3!0U0k6i4y4K6i4K6u0V1K9h3&6B7k6h3y4@1K9h3!0F1)网站上的一段代码作为`malware.exe`的示例。

 ![图片描述](https://bbs.kanxue.com/upload/attach/202310/876202_VHVXA833EG9H36M.webp)

一旦钩子放置好，EDR代理程序（`EDR_Process.exe`）就可以监视/分析`malware.exe`。以下是它可能采取的一些行动示例：

1. `EDR_Process.exe`看到`malware.exe`调用了以下Windows API：

- `OpenProcess`

- `VirtualAllocEx`

- `WriteProcessMemory`

- `CreateRemoteThread`

2. `EDR_Process.exe`将这个API调用序列分类为"恶意"并阻止（终止）该进程。

3. `EDR_Process.exe`向`EDR_C2`（安全控制台）发送日志，内容为"嘿，`malware.exe`进程已生成并被分类为恶意"。

注意：这是一种常见的EDR流程，并非唯一的工作方式，例如`EDR_Process.exe`可能只发送遥测数据，让`EDR_C2`决定是否恶意以及应采取的行动（阻止与否）。

如果EDR供应商或安全团队操作员（也称为蓝队）在EDR安全控制台中配置了"如果恶意则阻止"的规则，那么`malware.exe`进程将被`EDR_Process.exe`（或`EDR_Driver.sys`）终止。还有其他的应对措施，例如：

- 可以将Windows主机从网络中远程隔离

- 可以下载`malware.exe`文件或内存转储进行分析/逆向

- 安全分析师可以从安全控制台上对Windows主机运行命令，以进行调查

这一点非常重要；蓝队越有经验地创建自定义规则，攻击者越难以逃避或在网络中横向移动而不被发现！

现在，在深入研究内部通信之前，我想先退后一步，简化EDR的行为。`EDR_Process.exe`的内部通信（蓝色箭头）和外部通信（黄色箭头）可以用简单的概述进行可视化：

 ![图片描述](https://bbs.kanxue.com/upload/attach/202310/876202_7NABES3Q457WP49.webp)

深入研究EDR的内部通信

从Windows内核内存空间，`EDR_Driver.sys`可以使用多个Windows内核API（回调）来监视并阻止恶意系统活动。例如，API `PsSetCreateProcessNotifyRoutine`例程可以通过内核回调机制生成以下"监控日志"消息：

- 日志 = 新进程创建（PID 5376），命令行为`C:\notepad.exe`

从用户模式内存空间，`EDR_Process.exe`可以向驱动程序发送行动请求并从中接收信息。例如，来自EDR安全控制台的"行动请求"可能是：

- 行动 = 拒绝名单 `C:\notepad.exe`

在下面的图中，我试图绘制用于监视目的的常见Windows内核回调。

 ![图片描述](https://bbs.kanxue.com/upload/attach/202310/876202_FG8BMNDSJ94UCYG.webp)

在总结这一点之后，一个问题浮现在脑海中，即如何避免`EDR_Process.exe`与`EDR_Driver.sys`之间的通信？

## 使用已知技术来遮蔽EDR

最常见的遮蔽EDR传感器的技术包括：

- 移除DLL钩子（用户空间）

- 移除内核回调（内核空间）

由于我们只关注EDR的内核部分，下面是移除内核回调时会发生的情况的可视化示意图：

在将EDR回调地址清零之前：
