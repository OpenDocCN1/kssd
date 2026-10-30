
每个资源管理器都有一组发生在关键节点上的关联事务通知，例如从一个状态切换到另一个状态的登记。其中一个非常相关的示例是，在恢复资源管理器时，将为恢复中涉及的每个登记发送通知。如果要从用户区读取这些通知，请调用`GetNotificationResourceManager()`。这些事件被放入资源管理器（`\_KRESOURCEMANAGER.NotificationQueue`）跟踪的FIFO队列中。

从关联的`TRANSACTION\_NOTIFICATION`结构获取通知并打印有关通知的某些信息的示例如下所示。该结构后接`TRANSACTION\_NOTIFICATION\_RECOVERY\_ARGUMENT`，因此我们使用以下结构：

```.cpp
typedef struct TRANSACTION\_NOTIFICATION\_ALL\_ {
TRANSACTION\_NOTIFICATION TransNotif;
TRANSACTION\_NOTIFICATION\_RECOVERY\_ARGUMENT RecovArg;
} TRANSACTION\_NOTIFICATION\_ALL;
```

以下代码显示了如何读取通知：

```.cpp
BOOL bRet;
DWORD ReturnLength;
TRANSACTION\_NOTIFICATION\_ALL TransNotif;
ZeroMemory(&TransNotif, sizeof(TransNotif));
TransNotif.ArgumentLength = sizeof(TRANSACTION\_NOTIFICATION);
bRet = GetNotificationResourceManager(
hRM,
(PTRANSACTION\_NOTIFICATION)&TransNotif,
sizeof(TransNotif),
1000, // Millisecs
&ReturnLength,
);
printf("TransactionKey=%p, TransactionNotification=%p\n",
TransNotif.TransNotif.TransactionKey,
TransNotif.TransNotif.TransactionNotification);
printf\_guid(&TransNotif.RecovArg.EnlistmentId);
```

当您请求通知信息以及其他信息时，您会收到与拥有相关通知的确切登记相关联的GUID。

我们将进一步讨论登记和事务稍后可能转换到的实际状态。

# 结论

在这篇博客文章中，我们介绍了KTM在内核中的工作原理及其相关的KTM内核对象结构的基本背景。我们还详细介绍了如何在用户层使用与KTM相关的重要函数与KTM进行交互。

我们博客的第2 部分将深入了解CVE-2018-8611漏洞和补丁。

原文链接：`fa3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6J5k6i4y4W2j5i4u0U0K9q4)9J5k6h3&6U0j5$3N6J5L8%4g2H3i4K6u0W2j5$3!0E0i4K6u0r3x3U0l9J5x3q4)9J5c8U0l9@1i4K6u0r3x3U0N6Q4x3V1k6U0N6X3g2Q4x3X3b7J5x3o6p5^5i4K6u0V1z5o6j5I4x3g2)9J5k6r3g2^5M7r3I4G2K9i4c8A6L8X3N6Q4x3X3c8%4K9h3&6V1L8%4N6K6i4K6u0V1K9%4c8E0i4K6u0V1M7r3q4J5N6q4)9J5k6o6q4Q4x3X3b7#2i4K6u0V1M7r3q4@1j5$3S2Q4x3X3c8S2L8X3q4D9P5i4y4A6M7#2)9J5k6r3q4F1k6q4)9J5k6r3u0S2M7$3W2U0i4K6u0V1N6s2u0A6k6$3N6W2M7X3W2F1k6#2)9J5c8R3`.`.`

翻译： 本文由看雪翻译小组 fyb波 翻译

校对： 本文由看雪翻译小组 一壶葱茜 校对

# 概要

在这个由五部分组成的博客文章系列中，我们将讨论Windows内核中的本地特权提升漏洞 CVE-2018-8611的利用。最初，这个漏洞是由卡巴斯基在野外发现 0day 漏洞利用而被披露的。目前尚未发现有关此漏洞的大量公开细节发布，也未公开发布此漏洞的样本。此外，卡巴斯基发现的漏洞利用的哈希值也从未发布过。

CVE-2018-8611影响Windows内核的称为内核事务管理器（KTM）的组件，公共的安全社区尚未对此进行过多研究。CVE-2018-8611是一个内核竞争条件漏洞，也是Windows（ [bochspwn](https://bbs.kanxue.com/elink@d14K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6Y4L8$3!0Y4L8r3g2H3M7X3!0B7k6h3y4@1P5X3g2J5L8#2)9J5c8X3u0G2j5$3S2K6M7s2N6F1) 除外）上讨论得相对较少的一类bug。有趣的是，对KTM的访问不受当前系统调用沙箱过滤器的限制。这不同于通常被沙箱阻止的 win32k组件，例如 [Chrome](https://bbs.kanxue.com/elink@a49K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0K9s2u0G2L8h3W2#2L8g2)9J5k6h3N6G2L8$3N6D9k6i4y4G2N6i4u0U0k6g2)9J5k6h3y4G2L8g2)9J5c8X3y4Z5M7X3!0E0K9i4g2E0i4K6u0r3M7%4u0U0i4K6u0r3i4K6u0n7i4K6u0r3L8r3E0Y4M7W2)9J5c8X3c8G2j5%4y4Q4x3V1k6V1k6i4y4A6k6$3&6Q4x3V1k6K6j5h3&6V1j5X3!0^5i4K6u0W2L8h3c8Q4x3U0y4b7M7X3!0U0k6i4y4K6i4K6u0V1L8h3W2@1K9h3N6S2N6r3W2G2L8W2)9J5k6s2m8G2L8r3W2U0K9h3g2K6) 强制执行的 [win32k syscall 过滤器](https://bbs.kanxue.com/elink@aacK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8i4m8J5L8%4y4W2j5#2)9J5k6h3y4G2L8g2)9J5c8Y4c8W2j5$3S2Q4x3X3c8T1L8r3!0Y4i4K6u0r3N6$3W2F1x3K6u0C8i4K6u0V1M7%4W2K6N6r3g2E0i4K6u0V1j5$3q4D9L8q4)9J5k6r3k6A6L8s2c8W2M7X3W2F1k6#2)9J5k6r3c8W2k6i4m8Q4x3X3c8V1K9i4k6W2) 。这意味着这个漏洞在客户端利用场景中充当了宝贵的沙箱逃逸角色。

我们将深入研究 KTM 的内部，展示我们的补丁分析，弄清漏洞的根本原因是什么（借助卡巴斯基相对较少的技术细节），最后讨论了我们开发相当可靠的漏洞利用方法。我们能够开发一种可在x86和x64体系结构上的Windows Vista至Windows 10 1809的所有版本上使用的漏洞利用程序。

这项研究由NCC团队漏洞利用开发小组（EDG）的`Aaron Adams` 和 `Cedric Halbronn`完成。这项研究首先在 `POC2019` 上发表，可以在`这里`找到该演讲的幻灯片。

与我们的许多其他公开的 writeup 一样，我们将在进行这项研究时尝试详细解释我们的想法和方法，以帮助刚接触该领域的人们。我们试图突出一些遇到的障碍，错误和死胡同。与往常一样，我们很高兴收到建议，更正和反馈。

# 入门

## 已发布的漏洞详细信息

我们最初发现这个漏洞，是阅读了卡巴斯基于2018年12月发布的的 博客。文章指出，2018年10月，卡巴斯基（Kaspersky）一项名为自动漏洞利用防御（AEP）的技术检测到该漏洞的利用，然后将其报告给微软。这导致该漏洞于2018年12月被微软修复。

卡巴斯基博客在细节上篇幅讲述较少，但是提供了一些非常有用的提示。老实说，刚开始的时候其中的某些信息感觉没有多大用处，或者只是暂时还没有找到他们之间的关联，但是后来我们加深了对 KTM 和漏洞本身的理解后，他们提供的有关该漏洞利用的信息开始变得越来越有用。

卡巴斯基针对漏洞和利用行为的全部公开技术说明均逐字粘贴在以下两段中，当我们描述分析时，我们会不时地提及到它们：

|  |  |

| --- | --- |

| 1  2  3 | `为了滥用这个漏洞，漏洞利用程序首先创建一个命名管道，然后将其打开以用于读和写。然后，它创建一对新的事务管理器对象， 资源管理器对象，事务对象并创建大量登记对象，我们称之为“事务` `#2”。登记是一个用于事务和资源管理器之间关联的特殊对象。当事务状态改变时，KTM通知相关的资源管理器。之后，它又创造了一个登记对象，现在才对“事务 #1” 执行操作，并提交事务期间发生的所有更改。`    `在所有的初步工作准备完成后，来到漏洞利用的第二部分——触发漏洞。它创建多个线程并将它们绑定到一个CPU核心。第一个创建的线程在循环中调用 `NtQueryInformationResourceManager`，而第二个线程试图执行一次`NtRecoverResourceManager`。但是漏洞本身是在第三个线程中触发。该线程使用 `NtQueryInformationThread` 的执行技巧获取第二个线程最近一次执行的系统调用信息 `NtRecoverResourceManager`的成功执行说明已经发生条件竞争情况，并且进一步在先前创建的命名管道上执行`WriteFile`操作，导致内存崩溃。` |

这两段中包含很多信息，对于读者来说，其中的大多数现在可能没有多大意义，但是希望您继续阅读。值得注意的是，即使知道如何利用此漏洞，也不是所有的内容都有意义，但可以仅仅表示略有不同的漏洞利用方法。

在2019年初利用此漏洞，并随后在2019年10月准备博客和POC演示文稿之后，我们被告知，卡巴斯基于2019年5月在BlueHat Shanghai上提供了有关0day漏洞利用的更多 细节。我们分析了0day漏洞利用的一些技术，并将其与博客系列第5部分中的方法进行了比较。

## 测试环境

如果您想复现其中的一些工作，我们使用以下工具进行了大部分研究：

- `VMWare Workstation`: 虚拟机

- `IDA Pro`: 使用x86/x64 `HexRays` 反编译器的IDA Pro反汇编器（ 7.3版本中的功能）
