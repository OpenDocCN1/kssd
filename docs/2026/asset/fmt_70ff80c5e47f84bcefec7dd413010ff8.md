
上面已经暗示了这一点，但是事务是KTM中其他所有事情的基础。微软文档 [提供](https://bbs.kanxue.com/elink@823K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6%4K9h3^5K6x3W2)9J5c8X3E0@1L8g2)9J5c8Y4N6Z5j5i4c8Q4x3X3c8A6M7#2)9J5k6r3q4Q4x3X3c8@1M7X3q4F1M7$3q4U0N6r3W2G2L8R3`.`.) 了相当全面的高层解释。它们还在 [这里](https://bbs.kanxue.com/elink@73dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6%4K9h3^5K6x3W2)9J5c8X3E0@1L8g2)9J5c8Y4c8J5j5h3&6K6j5h3y4@1K9h3!0F1M7H3`.`.)提供了较低层次的视图。

从我们的角度来看，事务是 `_KTRANSACTION` 内核结构，与事务管理器有关联，还有一个或多个登记。它有效地代表了一些期望原子化完成的多部分工作。事务用于跟踪即将启动、正在执行或已完成的工作。

事务主要可以执行三个操作：创建，提交和回滚。提交事务意味着将部分操作转换为永久更改。回滚事务只是意味着在事务实际完成之前还原到目前为止已发生的所有部分操作。已回滚的事务无法提交，反之亦然。

以下是 `_KTRANSACTION` 结构的代码段：

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19 | `// 0x2d8 bytes (sizeof) struct _KTRANSACTION { struct _KEVENT OutcomeEvent; //0x0 ULONG cookie; //0x18 struct _KMUTANT Mutex; //0x20 [...] struct _GUID UOW; //0xb0 enum _KTRANSACTION_STATE State; //0xc0 ULONG Flags; //0xc4 struct _LIST_ENTRY EnlistmentHead; //0xc8 ULONG EnlistmentCount; //0xd8 [...] union _LARGE_INTEGER Timeout; //0x128 struct _UNICODE_STRING Description; //0x130 [...] struct _KTM* Tm; //0x200 [...] };` |

实际上，我们并没有太多使用到 `_KTRANSACTION` 结构，但是很有趣的是它跟踪了登记数量（`EnlistmentCount`），登记链表（`EnlistmentHead`）和指向关联事务管理器（`Tm`）的指针。

我们只需调用 [`CreateTransaction()`](https://bbs.kanxue.com/elink@d90K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6%4K9h3^5K6x3W2)9J5c8X3q4H3K9g2)9J5c8X3E0@1L8i4M7K6x3W2)9J5c8X3&6X3i4K6u0V1K9%4c8E0N6K6x3J5i4K6u0V1j5%4u0W2j5i4c8W2N6s2u0S2L8Y4y4S2j5%4c8A6L8$3^5`.)用户态函数来创建事务：

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9 | `HANDLE hTx = CreateTransaction( NULL, // lpTransactionAttributes 0, // UOW 0, // CreateOptions 0, // IsolationLevel 0, // IsolationFlags infinite timeout L"ExampleTx" // Description );` |

值得注意的是，大多数 KTM 内核结构都有一个`cookie`字段，可以在上面的`_KTRANSACTION`结构中看到。这些对于每种结构类型都是唯一的，因此在调试器中查找以确认我们正在查看正确的`_KTM *`类型和偏移量时非常有用。下面我们列出了有用的 KTM 类型的cookie值：

| Cookie | Object type |
| --- | --- |
| `0xb00b0001` | `_KTRANSACTION` |
| `0xb00b0002` | `_KRESOURCEMANAGER` |
| `0xb00b0003` | `_KENLISTMENT` |
| `0xb00b0004` | `_KTM` |
| `0xb00b0005` | Protocol Address Info? |
| `0xb00b0006` | Propagate Request? |

大多数`KTM *`对象还检测到[池标记](https://bbs.kanxue.com/elink@36aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6*7L8$3c8A6j5h3y4G2L8W2)9J5c8W2m8G2L8$3I4y4L8$3&6j5i4K6u0r3j5X3I4G2j5W2)9J5c8X3#2S2M7%4c8W2M7W2)9J5c8Y4u0W2M7#2)9J5c8Y4m8G2L8$3I4@1j5h3N6Q4x3X3g2@1P5s2b7`.)，允许在内核池上跟踪它们。下面我们仅显示与我们将描述的漏洞最相关的结构的标记。

| Pool tag | Object type |
| --- | --- |
| `TmTx` | `_KTRANSACTION` |
| `TmRm` | `_KRESOURCEMANAGER` |
| `TmEn` | `_KENLISTMENT` |
| `TmTm` | `_KTM` |

## 事务管理器是什么（TM）？

事务管理器基本上是一个管理一般事务的实体。它是KTM层次结构中顺序最高的部分。通常，事务管理器将包含一个或多个与其关联的资源管理器。为了实际执行任何类型的事务，必须首先创建一个事务管理器。

出于利用目的，一个重要的事务管理器概念是易失性和持久性之间的区别：

从 [MSDN](https://bbs.kanxue.com/elink@0c0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6%4K9h3^5K6x3W2)9J5c8X3E0@1L8g2)9J5c8Y4c8J5j5h3&6K6j5h3y4@1K9h3!0F1i4K6u0V1L8h3q4F1j5h3N6W2M7Y4x3`.) 总结的事务管理器：

- 持久性——具有日志并可以恢复其状态的常规事务管理器
- 易失性——这些事务管理器没有日志，无法恢复其状态

使用持久性事务管理器时，您所做的一切最终都会记录在磁盘上的日志文件中。如果您正在做类似重复尝试使用成千上万的登记实现条件竞争的事情，那么它很可能会很快填满日志并可能导致错误。通过反复试验，我们了解到使用易失性事务管理器可以解决许多问题，而这正是我们用来利用的方法。

描述事务管理器的内核结构是 `_KTM`:

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15 | `// 0x3c0 bytes (sizeof) struct _KTM { ULONG cookie; //0x0 struct _KMUTANT Mutex; //0x8 enum KTM_STATE State; //0x40 [...] ULONG Flags; //0x80 ULONG VolatileFlags; //0x84 struct _UNICODE_STRING LogFileName; //0x88 struct _FILE_OBJECT* LogFileObject; //0x98 [...] struct _KRESOURCEMANAGER* TmRm; //0x2a8 [...] };` |

与 `_KTRANSACTION` 一样，在开发利用过程中，我们无需在实践中使用过多。许多内容与日志记录（`LogFileName`，`LogFileObject`）有关，但它也跟踪所有关联的资源管理器（`TmRm`）。

`Flags`字段使用未记录的标记，在逆向时我们对其进行了如下注释：

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12 | `enum KTM_FLAGS { KTM_FLAG_VOLATILE = 0x01, KTM_FLAG_COMMIT_SYSTEM_VOLUME = 0x02, KTM_FLAG_COMMIT_SYSTEM_HIVES = 0x04, KTM_FLAG_COMMIT_LOWEST = 0x08, KTM_FLAG_THAW = 0x10, KTM_FLAG_NO_IDENTITY = 0x20, KTM_FLAG_CORRUPT_FOR_PROGRESS = 0x40, KTM_FLAG_CORRUPT_FOR_RECOVERY = 0x80, KTM_FLAG_CLUSTERED = 0x100, KTM_FLAG_UNK4000 = 0x4000 };` |

通常，创建事务管理器是您要做的第一件事，必须调用 `CreateTransactionManager()`:

|  |  |
| --- | --- |
| 1  2  3  4  5  6 | `HANDLE hTM = CreateTransactionManager( NULL, // lpTransactionAttributes NULL, // LogFileName TRANSACTION_MANAGER_VOLATILE, // CreateOptions 0 // CommitStrength );` |

## 资源管理器是什么 (RM)?

资源管理器是管理资源的实体，它管理将在完成与事务相关的某些工作中涉及的资源。例如，为了完成事务，可能必须同时涉及文件系统和注册表并进行一些工作。在这种情况下，文件系统和注册表都将拥有自己的关联资源管理器，并且每个资源管理器将负责管理该事务工作的某些部分。每个资源管理器都将通过创建与该事务相关联的登记加入事务中以完成必要的工作。在等待事务完成时，您将等待相关的两个资源管理器完成工作，然后再继续。

从利用的角度来看，我们只是创建自己的资源管理器，并加入到自己的事务中，因此它并不能真正反映大多数真实的用例。

使用 [`CreateResourceManager()`](https://bbs.kanxue.com/elink@451K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6%4K9h3^5K6x3W2)9J5c8X3q4H3K9g2)9J5c8X3E0@1L8i4M7K6x3W2)9J5c8X3&6X3i4K6u0V1K9%4c8E0N6K6x3J5i4K6u0V1j5%4u0W2j5i4c8W2M7X3g2K6L8%4g2J5j5$3g2E0j5h3&6S2k6$3g2J5) 用户层函数创建资源管理器：

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7 | `HANDLE hRM = CreateResourceManager( NULL, // lpTransactionAttributes pRMGuid, // ResourceManagerId - GUID can't be NULL RESOURCE_MANAGER_VOLATILE, // CreateOptions - No log file hTM, // TmHandle - Previously created transaction manager handle NULL, // Description );` |

资源管理器的内核结构是 `_KRESOURCEMANAGER`:

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19 | `// 0x250 bytes (sizeof) struct _KRESOURCEMANAGER { struct _KEVENT NotificationAvailable; //0x0 ULONG cookie; //0x18 enum _KRESOURCEMANAGER_STATE State; //0x1c ULONG Flags; //0x20 struct _KMUTANT Mutex; //0x28 [...] struct _KQUEUE NotificationQueue; //0x98 struct _KMUTANT NotificationMutex; //0xd8 struct _LIST_ENTRY EnlistmentHead; //0x110 ULONG EnlistmentCount; //0x120 LONG (*NotificationRoutine)(struct _KENLISTMENT* arg1, VOID* arg2, VOID* arg3, ULONG arg4, union _LARGE_INTEGER* arg5, ULONG arg6, VOID* arg7); //0x128 [...] struct _KTM* Tm; //0x168 struct _UNICODE_STRING Description; //0x170 [...] };` |

这是了解漏洞和利用的非常重要的内核结构。最重要的字段是：

- `Tm` – 指向关联的事务管理器的指针
- `Description` –与资源管理器关联的（可选）日志名称unicode字符串读取了 `Description` 字段等。
- `EnlistmentHead` –与此资源管理器关联的登记的当前列表
- `Mutex` – 锁定资源管理器，这意味着其他代码无法解析该资源管理器的登记列表 (`EnlistmentHead`)。
- `NotificationQueue` –可以从用户区查询通知包队列，以检索与登记状态更改有关的事件

`Flags`字段使用未记录的标记，在逆向时我们对其进行了如下注释：

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7 | `enum KRESOURCEMANAGER_FLAGS { KRESOURCEMANAGER_NOTIFY = 0x01, KRESOURCEMANAGER_UNK02 = 0x02, KRESOURCEMANAGER_IS_VOLATILE = 0x04, KRESOURCEMANAGER_COMMUNICATION = 0x08, KRESOURCEMANAGER_COMPLETION = 0x10 };` |

我们将在描述漏洞以及如何利用此漏洞时，进一步讨论如何使用其中的某些字段。

## 登记是什么 (En)?
