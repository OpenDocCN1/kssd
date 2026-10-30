
理解登记的最好方法是将其视为某个资源管理器为完成事务而执行某项工作的承诺。该资源管理器已登记到事务中，这意味着所有其他登记都需要协调状态，以便完成事务。一个登记可以看作是与事务相关联的一项工作。只要该登记没有转换为只读登记，就需要指示它已完成其工作的给定阶段，这最终允许事务转换到下一个状态。是否可以进行回滚或恢复还将取决于与事务相关的某些或所有登记的状态。我们稍后将更详细地描述这些状态。

`_KENLISTMENT` 的内核结构如下：

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16 | `/` `/` `0x1e0` `bytes (sizeof)` `struct _KENLISTMENT` `{` `ULONG cookie;` `/` `/` `0x0` `struct _KTMOBJECT_NAMESPACE_LINK NamespaceLink;` `/` `/` `0x8` `struct _GUID EnlistmentId;` `/` `/` `0x30` `struct _KMUTANT Mutex;` `/` `/` `0x40` `struct _LIST_ENTRY NextSameTx;` `/` `/` `0x78` `struct _LIST_ENTRY NextSameRm;` `/` `/` `0x88` `struct _KRESOURCEMANAGER*` `ResourceManager;` `/` `/` `0x98` `struct _KTRANSACTION*` `Transaction;` `/` `/` `0xa0` `enum _KENLISTMENT_STATE State;` `/` `/` `0xa8` `ULONG Flags;` `/` `/` `0xac` `ULONG NotificationMask;` `/` `/` `0xb0` `[...]` `};` |

最重要的字段是：

- `Transaction` – 登记实际为其工作的事务
- `Flags` – 指示登记类型和某些内部登记状态的掩码（例如，`KENLISTMENT_IS_NOTIFIABLE`是一个内部“状态”，指示此登记是否已被通知某些事务状态更改）
- `Mutex` – 锁定登记并防止其他代码操纵它
- `State` – 登记与事务相关的当前事务状态（例如“PrePrepared”）
- `NotificationMask` – 哪些通知应排队到与此登记相关的资源管理器
- `NextSameRm` – 与同一资源管理器关联的登记的链表。 这就是列表项头部为 `_KRESOURCEMANAGER.EnlistmentHead`

`Flags`字段使用未记录的标记，在逆向时我们对其进行了如下注释：

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10 | `enum KENLISTMENT_FLAGS {` `KENLISTMENT_SUPERIOR` `=` `0x01`,` `KENLISTMENT_RECOVERABLE` `=` `0x02`,` `KENLISTMENT_FINALIZED` `=` `0x04`,` `KENLISTMENT_FINAL_NOTIFICATION` `=` `0x08`,` `KENLISTMENT_OUTCOME_REQUIRED` `=` `0x10`,` `KENLISTMENT_HAS_SUPERIOR_SUB` `=` `0x20`,` `KENLISTMENT_IS_NOTIFIABLE` `=` `0x80`,` `KENLISTMENT_DELETED` `=` `0x80000000` `};` |

要创建登记，请调用 `CreateEnlistment()` 函数。您必须为先前创建的资源管理器和事务指定一个句柄。

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8 | `hEn` `=` `CreateEnlistment(` `NULL,` `/` `/` `lpEnlistmentAttributes` `hRM,` `/` `/` `ResourceManagerHandle` `-` `Existing resource manager handle` `hTx,` `/` `/` `TransactionHandle` `-` `Existing transaction manager handle` `0x39ffff0f`,` `/` `/` `NotificationMask` `-` `Special value to receive` `all` `possible notifications` `0`,` `/` `/` `CreateOptions` `NULL` `/` `/` `EnlistmentKey` `);` |

在这里，我们将继续讨论一些暂时使我们放慢速度的事情。微软 文档将`TRANSACTION_NOTIFY_MASK = 0x3fffffff`指定为指示所有有效位的掩码，我们希望仅使用它来接收所有通知。但是，我们在调用`CreateEnlistment()`时总是会出错，事实证明，当`TRANSACTION_NOTIFY_MASK`指定的标志组合作为您要接收的所有通知集传递时，本身就是无效的组合。我们也不想只指定文档化的位，或者将所有位进行“或”运算，因为可能会有未文档化的通知。我们逆向了逻辑，注意到一个名为`TmpIsNotificationmaskValid()`的函数所做的测试：

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22 | `char __fastcall TmpIsNotificationMaskValid(enum NOTIFICATION_MASK NotificationMask, char CreateOptionsHasSuperior)` `{` `char result;` `result` `=` `TRUE;` `if` `( CreateOptionsHasSuperior )` `{` `/` `/` `0x207` `=` `TRANSACTION_NOTIFY_SINGLE_PHASE_COMMIT|TRANSACTION_NOTIFY_COMMIT|` `/` `/` `TRANSACTION_NOTIFY_PREPARE|TRANSACTION_NOTIFY_PREPREPARE` `if` `( !(NotificationMask &` `8`) || NotificationMask &` `0x207` `)` `return` `FALSE;` `}` `else` `if` `( !(NotificationMask &` `8`)` `|| NotificationMask &` `0x60000F0` `|| NotificationMask &` `2` `&& !(NotificationMask &` `4`)` `|| _bittest(&NotificationMask,` `9u`) && !(NotificationMask &` `2`)``/` `bittest(` `9` `)` `=` `TRANSACTION_NOTIFY_SINGLE_PHASE_COMMIT` `|| !(NotificationMask &` `4`) && !(NotificationMask &` `1`) )``/` `1` `=` `TRANSACTION_NOTIFY_PREPREPARE` `{` `return` `FALSE;` `}` `return` `result;` `}` |

在我们的例子中，我们实际上并没有使用“高级”登记，因此我们只对第二个 `if` 条件感兴趣。`TRANSACTION_NOTIFY_MASK` 为 `0x3FFFFFFF`，但我们需要确保`0x60000F0`中的位没有设置。其余测试将确保如果将一个位置1，则不会设置另一位。因此，我们只需执行“设置所有可能的路由”，这意味着我们只需要避免使用`0x60000F0`位：

|  |  |
| --- | --- |
| 1  2 | `python -c "print('0x%x' % (0x3FFFFFFF & (~0x60000F0)))"` `0x39ffff0f` |

这就是为什么我们在前面的示例中传递 `0x39ffff0f` 的原因。

### 高级登记

上面我们简要介绍了一些与“高级”登记有关的逻辑，并表示我们不会故意使用它们。但是，值得一提的是，尽管我们没有明确地将它们用于开发利用，但我们仍会滥用一些与逻辑相关的功能来帮助我们进行调试。

高级登记用于“分布式”事务场景，其中一些登记分散在某个分布式渠道中，而一个登记负责其他登记。我们认为，在 MSDN KTM 门户上对它们的描述有些欠缺，因为没有一个地方可以简洁地对其进行解释。它在传递各种函数描述时更容易被提及。

最重要的是，当您试图了解KTM API并选择要使用的函数时，了解某些功能仅适用于高级登记，而另一些功能用于非高级登记是很有用的, 这可能会有些混乱。特别是，仅高级函数的命名通常看起来像您想使用的名称。区别的方法是任何以`Enlistment`结尾的 API 名称通常只能在高级登记中使用。但是，如果API名称以`Complete`结尾，则适用于非高级登记。API文档中对此进行了详细的技术描述，但是如果您不熟悉KTM，并不一定总是很清楚。例如，对于诸如 `CommitEnlistment()`之类的内容，它的含义如下：

|  |  |
| --- | --- |
| 1 | `提交与此登记句柄关联的事务。这个函数由通信资源管理器（有时称为高级事务管理器）使用。` |

将它与 `CommitComplete()`相对比，后者表示以下内容：

|  |  |
| --- | --- |
| 1 | `表示资源管理器（RM）已完成提交 事务管理器（TM）请求的事务。` |

下面是两个简短的列表，显示了转换登记状态的常用函数。

| Non-superior enlistment | Superior enlistment |
| --- | --- |
| `PrePrepareComplete()` | `PrePrepareEnlistment()` |
| `PrepareComplete()` | `PrepareEnlistment()` |
