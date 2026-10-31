
# [原创]Windows XP(sp0~sp2某一版本)内核函数`CmpCheckRecursionAndRecordThreadInfo`崩溃BUG

作者：MJ0011 转载请注明

这是Windows XP 从SP0开始存在的一个问题，直到SP2的某一版本（至少在5.1.2600.3093之后）上才修复的这一问题

## 问题位置

问题出现在内核函数 `ntoskrnl!CmpCheckRecursionAndRecordThreadInfo`中

此函数使用了系统函数`ExAcquireFastMutex`来申请一个快速互斥体，然后在进行一些操作后，使用了`ExReleaseFastMutex`来释放这个Mutex

`ExReleaseFastMutex`在一定情况下会调用`KeSetEventBoostPriority`函数,这个函数在双核下会调用`KeAcquireQueuedSpinLockRaiseToSynch`函数提升到同步级别，而在单核下则调用`KeRaiseIrqlToDpcLevel`提升到DPC Level，接着会去操作FastMutex的相关结构内容（`DISPATCH_HEADER`）,此时，如果`pFastMutex`指向的是一个分页内存，将导致BSOD

作者：MJ0011 转载请注明

`CmpCheckRecursionAndRecordThreadInfo`实际上会在`CmpCallCallbacks`时调用，是Cm回调机制为了防止递归而加入的一种机制

## 触发条件

这个问题将发生在第三方驱动调用系统函数 `CmRegisterCallback`时发生，该函数作为一种WINDOWS提供的标准安全的注册表监视方式，有众多安全软件在使用（例如， bitfender 2008,2009等），在SP0~SP1或SP2非全补丁的系统上，用户将面临蓝屏风险

`CmRegisterCallback`将申请一个`CM_CALLBACK_CONTEXT_BLOCK`的上下文结构的分页内存，其中FastMutex就作为一个结构体包含在这个上下文结构中。

实际这个问题的发生几率并不是特别的高，原因在于`CmpCheckRecursionAndRecordThreadInfo`在Release Mutex之前没多久刚调用过`ExAcquireFastMutex`,内存在此时仍是换入的，只有在Acquire之后， 提升IRQL 之前，发生了换出，才会触发此问题。但CmCallback本身的触发率很高，因此仍是一个较严重的问题

## 修复说明

SP2的全补丁和SP3已经修补了这个问题，微软修改了`CM_CALLBACK_CONTEXT_BLOCK`的结构，将原本包含在此结构中的`FAST_MUTEX`结构体，使用一块非分页内存单独存放，而将该结构体的指针存放在`CM_CALLBACK_CONTEXT_BLOCK`结构中

此问题在微软的KB知识库中有包含：381K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4y4#2M7s2m8G2M7Y4c8Q4x3X3g2E0K9h3y4J5L8%4y4G2k6Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6C8j5W2)9J5c8U0V1K6y4U0b7#2y4W2)9J5c8Y4A6Z5i4K6u0V1j5$3^5`.

但这个网页只提供了给SP2用户的临时补丁

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
