
首先选择 Windows 7 的一个缺点是，我们首先构建了可在Vista和Windows 7上运行的写原语。但是，事实证明它由于Windows 8及更高版本上的 <a href="elink@55dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4M7#2)9J5k6i4c8W2j5$3S2F1k6i4c8Q4x3X3g2E0K9h3y4J5L8%4y4G2k6Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6K6M7X3c8Q4x3V1j5J5x3o6p5K6i4K6u0r3x3e0q4Q4x3V1j5H3y4W2)9J5c8Y4y4G2k6Y4c8%4j5i4u0W2i4K6u0V1k6r3g2X3k6h3&6K6k6g2)9J5k6s2y4S2k6X3g2Q4x3X3c8#2L8X3I4A6L8X3E0A6L8X3N6Q4x3X3c8S2L8X3c8Q4x3X3c8J5k6h3k6W2M7X3g2F1j5$3g2Q4x3X3c8U0L8%4g2F1N6q4)9J5k6r3S2S2M7X3c8W2L8X3W2F1k6#2)9J5c8R3`.`.">缓解措施</a>而失败，从而迫使我们重新考虑我们的方法。这些决定总是需要权衡的。通常最好不要过度分析并继续下去。

鉴于我们的逆向工作主要是在Windows 7上完成的，本文中几乎所有代码段均来自经过反编译的Windows 7 `ntoskrnl.exe` 二进制文件。如果在其他Windows版本中存在一些值得注意的行为差异，我们会尝试提及，但由于该漏洞相当复杂，我们可能会忽略一些细节。因此，如果您选择复现，请注意当您从 Windows Vista 移植到 Win10 时，您会发现在代码中的结构布局，偏移量方面遇到细微差异。如果存在这些差异，则在移植漏洞利用程序时，有时会需要作特殊处理。

还需要注意的是，在Windows 8及更高版本上，通过补丁对比分析 `tm.sys` 不会导致其他 `ntoskrnl.exe` 更改或漏洞出现在您的结果中，因此，在Windows 8及更高版本上进行补丁对比分析 KTM 漏洞要更加容易一些。

我们专门逆向了 Windows 7 x64 版本的 `ntoskrnl.exe`（版本号为 `6.1.7601.24291`）。

# 了解Windows内核事务管理器（KTM）

## 文档

在查看了补丁（稍后我们将描述）之后，并没有真正理解我们所看到的内容，我们决定首先需要尽可能多地了解KTM 是什么。我们选择从逆向大多数主要内核API和系统调用实现开始，同时建立我们自己的一组用户层代码示例以探索 API。我们知道最终我们将能够重用许多小样例代码进行利用编写，这使我们能够缓慢地建立起坚实的理解基础。

与许多其他MSDN页面不同， <a href="elink@ae7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6%4K9h3^5K6x3W2)9J5c8X3E0@1L8g2)9J5c8X3E0W2M7X3&6W2L8q4)9J5k6s2c8J5j5h3&6K6j5h3y4@1K9h3!0F1i4K6u0V1L8h3q4F1j5h3N6W2M7W2)9J5k6s2m8G2M7Y4c8S2L8l9`.`.">MSDN KTM 门户网站</a>上很少有公开的可用的KTM代码示例，因此我们没有很多可重用的代码段。

另一个资源是 <a href="elink@821K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3X3c8Z5j5i4u0V1N6$3q4J5k6g2)9J5c8X3c8J5K9i4k6W2M7Y4y4Q4x3V1k6C8k6i4u0F1k6h3I4Q4x3V1k6A6L8Y4c8J5L8$3c8#2j5%4c8A6L8$3&6Q4x3X3c8@1L8#2)9J5k6r3E0@1L8b7`.`.">MSDN 内核模式KTM门户 </a> ，它从内核方面讨论KTM内部。在理解新概念时，它非常有用，但也很麻烦。刚开始不必理解所有内容，但确实值得一读。另外请注意，我们只对内核API感兴趣，以便更好地理解概念，但由于我们本地的特权提升漏洞利用只能调用用户层KTM API，因此我们将无法直接调用它们。

因此，我们编写的大部分内容都是通过反复试验或通过逆向KTM组件使用API的方式得出的。即使MSDN上的KTM文档非常详尽，但在没有工作代码段来演示概念之间的相关性的情况下，对所有术语进行整理还是有些令人困惑。

我们还找到了有关KTM总体工作原理的很好概述，而官方文档中没有，这是Microsoft在2005年至2007年之间发布的3个视频系列，作为“深入”系列中有关Vista新发行版Windows技术的一部分的视频：

- <a href="elink@1a9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0K9r3q4F1L8X3g2D9z5g2)9J5k6h3#2K6k6r3&6Q4x3X3g2U0L8$3#2Q4x3V1k6e0K9r3!0%4M7#2)9J5c8V1N6G2K9h3&6Y4i4K6u0n7c8r3g2W2M7q4)9J5c8W2c8J5j5h3&6K6j5h3y4@1K9h3!0F1j5h3I4Q4x3X3c8h3K9i4y4@1j5g2)9J5k6p5E0W2M7X3&6W2L8q4)9J5k6q4c8J5j5h3&6K6j5h3y4@1K9h3!0F1i4K6u0V1e0h3q4F1j5h3N6W2M7W2)9J5k6r3q4F1k6q4)9J5k6r3k6J5K9h3g2F1k6s2y4Q4x3X3c8f1P5p5k6Q4x3X3c8f1P5q4t1`.">Transactional Vista: Kernel Transaction Manager and friends (TxF, TxR)</a>

- <a href="elink@08bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0K9r3q4F1L8X3g2D9z5g2)9J5k6h3#2K6k6r3&6Q4x3X3g2U0L8$3#2Q4x3V1k6e0K9r3!0%4M7#2)9J5c8V1q4d9b7$3q4K6N6q4)9J5b7Y4N6A6N6r3S2Q4x3V1u0d9L8$3&6Q4x3V1u0v1j5h3y4G2j5Y4y4Q4x3V1k6m8f1V1y4S2M7%4c8Q4x3X3c8f1M7X3q4F1M7$3q4U0N6r3W2G2L8X3q4D9i4K6u0V1c8X3W2D9k6g2)9J5k6q4y4&6M7%4c8W2L8g2)9J5k6r3q4F1k6q4)9J5k6q4u0W2k6$3W2K6N6s2u0&6">ARCast – Transactional File System and Registry</a>

- <a href="elink@799K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0K9r3q4F1L8X3g2D9z5g2)9J5k6h3#2K6k6r3&6Q4x3X3g2U0L8$3#2Q4x3V1k6e0K9r3!0%4M7#2)9J5c8V1N6G2K9h3&6Y4i4K6u0n7c8r3g2W2M7q4)9J5c8W2y4#2M7X3g2F1k6s2u0S2i4K6u0V1g2X3g2J5L8h3q4Q4x3X3c8h3K9i4y4@1j5g2)9J5k6q4c8J5j5h3&6K6j5h3y4@1K9h3!0F1j5h3I4Q4x3X3c8r3K9h3I4W2i4K6u0V1f1%4W2K6N6r3g2E0">Surendra Verma: Vista Transactional File System</a>

最近发现一些名为 <a href="elink@ec6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6X3N6h3#2A6K9K6m8Q4x3X3g2U0L8$3#2Q4x3V1j5J5x3o6p5&6i4K6u0r3x3o6g2Q4x3V1j5J5y4q4)9J5c8X3!0$3k6i4u0$3K9h3g2%4i4K6u0V1L8$3k6Q4x3X3c8H3M7X3!0@1L8$3&6Q4x3X3c8T1L8%4c8Q4x3X3c8S2L8X3!0@1K9r3g2J5i4K6u0V1L8r3!0S2k6r3g2J5i4K6u0V1K9h3&6Q4x3X3c8@1K9r3g2Q4x3X3c8%4K9h3I4V1i4K6u0r3">"Proton Bot"</a>的恶意软件使用 KTM 函数试图绕过 API监控/钩子之类的东西。不过，它似乎并没有真正利用KTM的事务性来实现其他任何目的。

## 什么是内核事务管理器（KTM）？

KTM 是Windows Vista中添加的一种技术，用于引入“事务操作”概念。Windows本身至少在Windows注册表和NTFS文件系统中使用此组件。这个概念特别存在于数据库领域，例如SQL等。想法是，某些给定的操作（称为“事务”）可能需要跨多个资源完成大量相关工作。这样的事务在较高的层次上代表着单个工作，需要有效地实现原子化，如果任何一个部分失败，则整个事务都会失败。

这种类型的“事务”对于复杂的多进程操作尤其重要。自动取款机（ATM）试图协调自己的现金和请求者的账户信息，这是一个常见的例子，可能受益于原子交易的概念。安装软件是另一个例子，在中断时，它需要回滚所有更改，因为比如说用户取消安装。

这意味着，如果操作的任何部分最终失败，则可以回滚已完成的与该事务相关的所有其他工作，使其看起来好像从未发生过一样。只有当与事务相关的所有工作都成功时，该事务才实际完成。

KTM的概念是从某些故障中恢复并回滚某些事务。我们稍后将详细介绍它们，但现在只需注意，如果与事务相关的某个工作失败，系统的设计将通知参与该事务的所有其他工作人员，以确保他们知道某些事情已经失败，或者他们都需要重新同步到某个商定的状态。

<a href="elink@333K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6%4K9h3^5K6x3W2)9J5c8X3E0@1L8g2)9J5c8X3E0W2M7X3&6W2L8q4)9J5k6s2c8J5j5h3&6K6j5h3y4@1K9h3!0F1i4K6u0V1L8h3q4F1j5h3N6W2M7W2)9J5k6s2m8G2M7Y4c8S2L8l9`.`.">MSDN KTM 门户</a> 是微软如何提供api以允许用户层客户端软件处理事务的主要参考。

我们对KTM中涉及的四个主要组件感兴趣，我们将更详细地描述它们。在整个文档中，我们在下一节中交替使用它们的长格式和短格式名称：

- 资源管理器 (RM)

- 事务管理器 (TM)

- 事务 (Tx)

- 登记 (En)

## 事务是什么（Tx）？

上面已经暗示了这一点，但是事务是KTM中其他所有事情的基础。微软文档 <a href="elink@343K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6%4K9h3^5K6x3W2)9J5c8X3E0@1L8g2)9J5c8Y4N6Z5j5i4c8Q4x3X3c8A6M7#2)9J5k6r3q4Q4x3X3c8@1M7X3q4F1M7$3q4U0N6r3W2G2L8R3`.`.">提供</a> 提供了相当全面的高层解释。它们还在 <a href="elink@a28K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6%4K9h3^5K6x3W2)9J5c8X3E0@1L8g2)9J5c8Y4c8J5j5h3&6K6j5h3y4@1K9h3!0F1M7H3`.`.">这里</a>提供了较低层次的视图。

从我们的角度来看，事务是 `_KTRANSACTION` 内核结构，与事务管理器有关联，还有一个或多个登记。它有效地代表了一些期望原子化完成的多部分工作。事务用于跟踪即将启动、正在执行或已完成的工作。

事务主要可以执行三个操作：创建，提交和回滚。提交事务意味着将部分操作转换为永久更改。回滚事务只是意味着在事务实际完成之前还原到目前为止已发生的所有部分操作。已回滚的事务无法提交，反之亦然。

以下是 <a href="f7bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2$3k6i4u0Y4K9h3I4A6N6i4y4H3M7X3!0B7k6h3y4@1i4K6u0W2j5$3!0E0i4K6u0r3K9$3g2J5L8X3g2D9M7#2)9J5c8Y4R3$3y4q4)9J5c8W2N6A6L8X3c8G2N6%4x3">`_KTRANSACTION`</a>（7 | 2008R2/SP1/_KTRANSACTION）结构的代码段：

```cpp
//0x2d8 bytes (sizeof)
struct _KTRANSACTION
{
struct _KEVENT OutcomeEvent; //0x0
ULONG cookie; //0x18
struct _KMUTANT Mutex; //0x20
[...]
struct _GUID UOW; //0xb0
enum _KTRANSACTION_STATE State; //0xc0
ULONG Flags; //0xc4
struct _LIST_ENTRY EnlistmentHead; //0xc8
ULONG EnlistmentCount; //0xd8
[...]
union _LARGE_INTEGER Timeout; //0x128
struct _UNICODE_STRING Description; //0x130
[...]
struct _KTM* Tm; //0x200
[...]
};
```

实际上，我们并没有太多使用到 `_KTRANSACTION` 结构，但是很有趣的是它跟踪了登记数量（`EnlistmentCount`），登记链表（`EnlistmentHead`）和指向关联事务管理器（`Tm`）的指针。

我们只需调用 <a href="elink@b6dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6%4K9h3^5K6x3W2)9J5c8X3q4H3K9g2)9J5c8X3E0@1L8i4M7K6x3W2)9J5c8X3&6X3i4K6u0V1K9%4c8E0N6K6x3J5i4K6u0V1j5%4u0W2j5i4c8W2N6s2u0S2L8Y4y4S2j5%4c8A6L8$3^5`.">`CreateTransaction()`</a>用户态函数来创建事务：

```cpp
HANDLE hTx = CreateTransaction(
NULL, // lpTransactionAttributes
0, // UOW
0, // CreateOptions
0, // IsolationLevel
0, // IsolationFlags
0, // infinite timeout
L"ExampleTx" // Description
);
```

值得注意的是，大多数 KTM 内核结构都有一个`cookie`字段，可以在上面的`_KTRANSACTION`结构中看到。这些对于每种结构类型都是唯一的，因此在调试器中查找以确认我们正在查看正确的`_KTM *`类型和偏移量时非常有用。下面我们列出了有用的 KTM 类型的`cookie`值：

| Cookie | Object type |
| :----------- | :--------------------- |
| `0xb00b0001` | `_KTRANSACTION` |
| `0xb00b0002` | `_KRESOURCEMANAGER` |
| `0xb00b0003` | `_KENLISTMENT` |
| `0xb00b0004` | `_KTM` |
| `0xb00b0005` | Protocol Address Info? |
| `0xb00b0006` | Propagate Request? |
