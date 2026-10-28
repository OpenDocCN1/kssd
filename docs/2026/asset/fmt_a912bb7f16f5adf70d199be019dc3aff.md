

# [翻译]“过滤-静音”操作：调查EDR内部通信

原文标题：Filter-Mute Operation: Investigating EDR Internal Communication

原文链接：<mark class="encrypted">e95K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6k6h3&6K6k6i4m8G2M7%4c8Q4x3X3g2U0L8$3#2Q4x3V1k6T1L8r3!0Y4i4K6u0r3x3U0l9J5x3#2)9J5c8X3k6A6L8s2c8W2M7W2)9J5k6r3#2#2N6r3g2Q4x3X3c8G2M7r3g2J5j5i4c8A6L8$3&6Q4x3X3c8A6L8Y4k6W2M7%4c8A6k6$3q4@1K9h3&6Y4i4K6u0V1k6h3c8J5i4K6u0V1K9h3&6@1k6i4u0F1j5h3I4Q4x3X3c8U0L8$3#2E0N6h3&6A6j5$3q4@1K9h3!0F1i4K6u0r3</mark>

[toc]

对于我们在2023年举办的年度内部黑客大会SenseCon，我决定研究一下Windows驱动程序与其用户模式进程之间的通信。以下是这个过程的一些细节。

## 简介

攻击者可以利用Windows内核的读/写漏洞原语来避免`EDR_Driver.sys`与其`EDR_Process.exe`之间的通信。结果是，一些EDR检测机制将被禁用，使其（部分地）对恶意有效负载变得盲目。本博文描述了一种替代方法，该方法不移除内核回调，并提供了一些建议，以防止这种“过滤-静音”攻击。

## `EDR_Process.exe`和`EDR_Driver.sys`角色概述

首先要弄清楚的问题是，EDR应用程序（`EDR_Process.exe`）如何与其EDR驱动程序（`EDR_Driver.sys`）进行通信？

在进行研究之前，我们必须了解一些EDR的基础知识；EDR代理程序在进程创建期间如何进行钩子/注入自己的DLL？

来自<a href="elink@cfeK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2K6K9h3N6F1j5h3I4Q4x3X3c8D9j5h3u0K6i4K6u0W2j5$3!0E0i4K6u0r3j5X3I4G2k6#2)9J5c8X3g2V1M7W2)9J5k6r3!0T1M7$3g2J5N6X3q4@1K9h3!0F1M7H3`.`.">Christopher Vella的EDR观察</a>中采用的**回调注入**模式是一个很好的总结。

![图片描述](upload/attach/202310/876202_6BET9G9A56UEVKG.webp)

我对正在发生的事情添加了一些注释：

1. `EDR_Driver.sys`可以订阅多种类型的内核通知。你可以将这些通知想象成你在互联网上订阅并通过电子邮件从网站收到的“新闻通讯”。例如，`EDR_Driver.sys`可以使用名为`PsSetCreateProcessNotifyRoutine`的Windows API订阅“新进程创建”通知服务，然后对系统创建的每个进程，驱动程序将接收到有关该进程的信息（父进程ID、命令行等）。

2. 用户双击运行`malware.exe`。

3. Windows调用`CreateProcessW` API将`malware.exe`加载到内存中。

4. `EDR_Driver.sys`收到一个通知，表示`malware.exe`将被生成。

5. `EDR_Driver.sys`向`EDR_Process.exe`发送日志，告知“嘿！一个名为`malware.exe`的新进程即将启动。”

6. `EDR_Process.exe`可以选择采取行动（或不采取行动）：“好的，我将通过在`ntdll.dll`中创建钩子来监控这个进程。”

7. 当`malware.exe`运行时，它调用Windows API。通过已经放置的钩子，`EDR_Process.exe`知道哪些API被调用，并可以推断出`malware.exe`正在做什么。

我们可以以<a href="elink@bceK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2A6M7X3g2V1i4K6u0W2N6r3g2S2L8g2)9J5c8X3!0X3k6X3g2F1M7$3W2$3k6g2)9J5k6s2y4W2j5%4g2J5K9i4c8&6i4K6u0r3j5$3!0V1k6g2)9J5k6r3W2F1K9X3g2U0N6r3W2G2L8W2)9J5k6s2m8J5L8$3y4W2M7%4y4Q4x3X3c8A6L8X3A6W2j5%4c8A6L8$3&6Q4x3V1k6H3M7X3!0U0k6i4y4K6i4K6u0V1K9h3&6B7k6h3y4@1K9h3!0F1">ired.team</a>网站上的一段代码作为`malware.exe`的示例。

![图片描述](upload/attach/202310/876202_VHVXA833EG9H36M.webp)

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

...

这一点非常重要；蓝队越有经验地创建自定义规则，攻击者越难以逃避或在网络中横向移动而不被发现！

现在，在深入研究内部通信之前，我想先退后一步，简化EDR的行为。`EDR_Process.exe`的内部通信（蓝色箭头）和外部通信（黄色箭头）可以用简单的概述进行可视化：

![图片描述](upload/attach/202310/876202_7NABES3Q457WP49.webp)

### 深入研究EDR的内部通信

从Windows内核内存空间，`EDR_Driver.sys`可以使用多个Windows内核API（回调）来监视并阻止恶意系统活动。例如，API `PsSetCreateProcessNotifyRoutine`例程可以通过内核回调机制生成以下"监控日志"消息：

- 日志 = 新进程创建（PID 5376），命令行为`C:\notepad.exe`

从用户模式内存空间，`EDR_Process.exe`可以向驱动程序发送行动请求并从中接收信息。例如，来自EDR安全控制台的"行动请求"可能是：

- 行动 = 拒绝名单 `C:\notepad.exe`

在下面的图中，我试图绘制用于监视目的的常见Windows内核回调。

![图片描述](upload/attach/202310/876202_FG8BMNDSJ94UCYG.webp)

在总结这一点之后，一个问题浮现在脑海中，即如何避免`EDR_Process.exe`与`EDR_Driver.sys`之间的通信？

## 使用已知技术来遮蔽EDR

最常见的遮蔽EDR传感器的技术包括：

- 移除DLL钩子（用户空间）

- 移除内核回调（内核空间）

由于我们只关注EDR的内核部分，下面是移除内核回调时会发生的情况的可视化示意图：

在将EDR回调地址清零之前：

![图片描述](upload/attach/202310/876202_CJ78QXKX4XBVXHA.webp)

在将EDR回调地址清零之后：

![图片描述](upload/attach/202310/876202_QPH63PNTUCE8EVV.webp)

我们不会详细讨论这个话题，这在<a href="elink@0e1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6P5h3&6*7j5h3y4C8i4K6u0W2k6$3W2@1K9s2g2T1i4K6u0W2K9h3!0Q4x3V1k6n7L8r3W2F1k6r3W2F1k6#2)9J5k6p5g2p5f1W2)9J5k6p5!0F1i4K6u0V1g2$3W2F1k6r3!0%4M7#2)9J5c8R3`.`.">Zach Stein的博文《在Windows上遮蔽EDR》</a>中有所涵盖。

但是你可能会注意到在下面的图中，每当你将EDR回调地址清零时，这意味着Windows将不再向`EDR_Driver.sys`发送任何通知（没有"新闻通讯"）。最终，将不再向`EDR_Process.exe`（和安全分析师控制台）发送任何事件日志！

![图片描述](upload/attach/202310/876202_6EUD8MHYXF2N7JD.webp)

## 使用另一种方法来遮蔽EDR

在研究这个话题时，我想知道如何在不进行任何回调修改的情况下避免`EDR_Process.exe`和`EDR_Driver.sys`之间的通信？我们能否阻止`EDR_Process.exe`和`EDR_Driver.sys`之间的"消息"交换？

正如我之前所说，我们希望保持在内核层面上进行讨论。我们可以想象使用这个图形表示的另一种方法：

![图片描述](upload/attach/202310/876202_MWNYAHBKGCSKGQE.webp)

当我尝试使用Windbg进行调查时，Yarden Shafir撰写了一篇关于<a href="elink@a6dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4K9h3&6V1L8%4N6K6i4K6u0V1K9h3&6@1k6i4u0F1j5h3I4K6i4K6u0W2j5$3!0E0i4K6u0r3K9h3&6$3k6i4y4@1K9h3N6S2N6r3W2F1k6#2)9J5k6r3k6A6L8s2c8W2M7W2)9J5k6r3y4G2L8h3#2#2L8X3W2U0j5i4c8A6L8$3&6Q4x3X3c8H3L8%4u0@1M7#2)9J5c8R3`.`.">调查过滤器通信端口</a>的绝妙博文，对此我受益匪浅。在应用程序和驱动程序之间的通信设置过程中，我发现了一些被操纵的Windows数据结构。

名为`FLT_SERVER_PORT_OBJECT`的数据结构引起了我的注意，因为它似乎包含了一些有趣的字段，看看你是否同意：

![图片描述](upload/attach/202310/876202_4BFXAW472QG7DF8.webp)

当我看到这个时，脑海中首先浮现的问题是，如果将`MaxConnections`设置为零，会发生什么情况？

该数据结构使用名为`FltCreateCommunicationPort`的Windows驱动程序API进行初始化：

```
NTSTATUS FLTAPI FltCreateCommunicationPort(
[in] PFLT_FILTER Filter,
[out] PFLT_PORT *ServerPort,
[in] POBJECT_ATTRIBUTES ObjectAttributes,
[in, optional] PVOID ServerPortCookie,
[in] PFLT_CONNECT_NOTIFY ConnectNotifyCallback,
[in] PFLT_DISCONNECT_NOTIFY DisconnectNotifyCallback,
[in, optional] PFLT_MESSAGE_NOTIFY MessageNotifyCallback,
[in] LONG MaxConnections
);
```

根据<a href="elink@752K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9k6h3q4J5L8W2)9J5k6h3#2A6j5%4u0G2M7$3!0X3N6q4)9J5k6h3y4G2L8g2)9J5c8X3g2F1i4K6u0V1N6i4y4Q4x3V1k6%4K9h3&6V1L8%4N6K6i4K6u0V1K9r3q4J5k6s2N6S2M7X3g2Q4x3V1k6V1M7X3W2$3k6i4u0K6i4K6u0r3k6r3c8A6i4K6u0r3k6X3I4@1K9$3g2J5L8X3g2D9i4K6u0r3L8X3k6Q4x3X3c8X3L8s2c8C8k6i4u0F1k6h3I4Q4x3X3c8X3L8s2c8U0M7X3g2S2N6r3g2U0L8$3#2E0N6h3&6A6j5$3q4@1K9h3!0F1M7r3!0J5N6l9`.`.">Microsoft的文档</a>，提供了以下信息：

![图片描述](upload/attach/202310/876202_4T4CA8H33ADKNJB.webp)

我们可以得出什么结论呢？如果我们能够将`MaxConnections`重置为零，那么它只会阻止新的连接发生。让我们来制定以下攻击计划：

步骤1：重置`MaxConnections`的值

步骤2：强制`EDR_Process.exe`重新启动（可能需要高权限，例如NT SYSTEM）

步骤3：观察EDR的行为

## 步骤1：重置`MaxConnections`的值

这一步的首要条件是我们必须具备一个内核模式的读/写原语，以便我们可以将该值设置为0。我们将使用BYOVD（Bring Your Own Vulnerable Driver）技术来实现此目的。作为第二个前提条件，我们必须找到内核内存中`MaxConnections`字段的地址，对吧？让我们看看如何获取这个地址！

我们之前讨论过的`fltmgr!_FLT_SERVER_PORT_OBJECT`结构可以通过`fltmgr!_FLT_FILTER`结构访问，而后者可以通过`fltmgr!_FLTP_FRAME`结构访问，而后者可以通过`FLTMGR!_GLOBALS`结构访问，而后者可以通过`FltMgr.sys`驱动程序访问。可以使用`NtQuerySystemInformation` Windows API从用户空间获取此内核模块的基地址。

我同意Alex Ionescu的说法：“<a href="elink@17fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2&6L8%4g2@1N6h3u0W2i4K6u0W2j5$3!0E0i4K6u0r3N6$3q4@1j5$3S2Q4x3@1k6$3i4K6y4p5y4f1S2T1L8i4m8b7b7V1E0h3c8X3M7`.">我有99个问题，但内核指针不是其中之一</a>” :-）。通过遍历Windows内核数据结构，从`FltMgr.sys`驱动程序开始直到该字段，我们可以找到`MaxConnections`的地址！

![图片描述](upload/attach/202310/876202_SYV25S8X6N5H8NX.webp)

这段内容对于博文的中间部分来说有点长，但如果你感兴趣并想知道如何使用Windbg来完成这个过程，请查看末尾的“额外步骤”部分：“\*内核遍历，10个步骤获取对`MaxConnections`的访问权限\*”。

以下是查看Windows Defender内核驱动程序详细信息时的情况：

![图片描述](upload/attach/202310/876202_DU7E33FUXTM343F.webp)

有了`MaxConnections`内存位置的信息，我们可以使用内核模式的读取原语获取当前值，并使用内核模式的写入原语将该值设置为0。

## 步骤2：强制EDR重新启动

此阶段可能会很困难，因为`EDR_Process.exe`会尽其所能保护自己。通常，该程序作为一个服务启动，并且在其终止后会重新启动，但我们不关心这一点，因为由于第1步，`EDR_Driver.sys`不允许任何连接 ;-)

个人而言，我使用自己的工具（未签名的恶意驱动程序）来执行此操作，该工具允许我们终止受保护的进程，但也可以使用Process Hacker（如果未列入拒绝列表），或者更好的是任何可利用的“进程终止驱动程序”。我强烈推荐阅读Alice Climent-Pommeret（<a href="elink@e24K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1N6$3W2@1N6r3g2J5i4K6u0W2j5$3!0E0i4K6u0r3b7h3I4A6j5$3g2o6L8r3W2E0k6h3&6@1">@AliceCliment</a>）的博文<a href="elink@7a3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2L8r3W2U0k6g2)9J5k6h3y4D9K9h3#2W2L8Y4c8Q4x3X3c8H3L8$3#2E0k6i4u0W2N6q4)9J5k6i4u0W2k6q4)9J5c8Y4m8G2M7%4c8K6i4K6u0r3M7s2u0G2j5$3g2K6M7#2)9J5k6r3E0A6L8r3I4W2M7W2)9J5k6r3c8J5K9i4k6W2M7W2)9J5c8R3`.`.">《用LOL找到并利用进程终止驱动程序，只需3000美元》</a>涵盖了这个主题！

## 步骤3：观察EDR行为

