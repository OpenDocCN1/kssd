
Jscript 的开发工程师曾经写过一篇关于 Jscript 垃圾回收机制的设计性描述，有兴趣的读者可以参考：

```text
784K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3P5X3S2Q4x3X3c8U0L8W2)9J5c8X3q4J5j5$3S2A6N6X3g2Q4x3V1k6T1L8r3!0Y4M7#2)9J5c8X3g2J5K9h3y4D9K9i4m8H3k6i4u0@1i4K6u0r3K9r3!0%4i4K6u0V1k6r3!0Q4x3X3c8@1K9r3g2Q4x3X3c8K6j5%4u0A6M7s2c8Q4x3X3c8Y4j5i4u0T1j5h3N6W2i4K6u0V1j5$3!0D9L8r3g2U0N6r3!0J5M7#2)9J5k6s2N6G2M7X3D9
```

## Jscript 中 GC 机制的具体实现

关于 Jscript 中 GC 机制的具体实现，国外研究员 Sudhakar Verma 写过一篇很棒的分析。为方便读者理解，笔者在这里对其进行注释说明。读者也可以在文末参考链接“Jscript 的垃圾回收机制”一节中找到这篇文章并阅读。

Jscript 中的 `variant` 存在一个个 `GCBlock` 结构中，每个 `GCBlock` 结构如下：

```cpp
struct GcBlock {
    struct GcBlock* prev;
    struct GcBlock* next;
    VARIANT mem[100];
};
```

当定义变量时，Jscript 引擎内部会申请一些 `GCBlock`，并初始化其内部的 `variant` 结构。

申请第一个 `GCBlock` 的相关调用如下：

```text
GcBlockFactory::GcBlockFactory

GcContext::EnsureGc
    GcContext::New
        GcContext::GcContext
        GcContext::Init
```

当有需要时，更多 `GCBlock` 被申请，申请过程如下：

```text
GcAlloc::PvarAlloc
    GcBlockFactory::PblkAlloc // Jscript 中有一个全局变量，这个变量是一个大小为 50 的 GCBlock 缓存链表的头部，当申请一个 GCBlock 时，优先从该链表申请
    GcBlock::Link
```

申请到的每一个 `GCBlock` 都会链接到一个双向链表（区别于缓存链表的另一个链表），遍历链表可以访问到所有 `GCBlock` 中的所有 `variant`。

当在 Jscript 代码中手动调用 `CollectGarbge` 函数时，Jscript 引擎会调用 `JsCollectGarbage` 函数，`JsCollectGarbage` 进一步会调用 `GcContext::Collect` 函数，紧随 `Mark`、`Scavenge` 和 `Sweep` 三个过程。

### Mark

`Mark` 阶段会遍历 `GCBlock` 链表，并遍历每个 `GCBlock` 中的所有 `variant`，对所有 `variant` 的第 12 位（定义第 1 位为最低位）置位 1（`|= 0x800`）。这个过程涉及的调用如下：

```text
GcContext::SetMark
    GcAlloc::SetMark
```

### Scavenge

`Mark` 完毕后是 `Scavenge` 阶段。这个阶段的任务是捡拾，即把所有还在使用中的对象从 `Mark` 阶段“抢救”回来。方法是从每一种不同的 `Root` 开始遍历，得到所有可遍历到的 Jscript 对象和 `variant`，并将所有遍历到的 `variant` 的第 12 位置为 0（`&= 0xf7ff`）。相关函数的调用顺序如下：

```text
ObjectRegistration::ScavengeRoots
    FncObj::ScavengeCore
        GcContext::ScavengeVar

VarStack::ScavengeRoots

ObjectRegistration::ScavengeRoots
    ScrFncObj::ScavengeCore

ObjectRegistration::ScavengeRoots
    VAR::Scavenge
        GcContext::ScavengeVar
    NameTbl::ScavengeCore
        NameList::ScavengeRoots

ScavVarList::ScavengeRoots
    GcContext::ScavengeVar

GCRootStack::ScavengeRoots
```

### Sweep

最后是清理阶段。这个阶段会再次遍历 `GCBlock` 链表，并遍历每个 `GCBlock` 中的所有 `variant`，对所有第 12 位为 1 的 `variant` 进行对应的清理。

当一个 `GCBlock` 内的所有 `variant` 都被清理时，这个 `GCBlock` 并不会被立即释放，只会对一个全局计数加 1，并将这个 `GCBlock` 加入上述缓存链表。当全局计数大于等于 50 时，Jscript 会将多余的 `GCBlock` 摘链后直接进行 `delete`。相关过程如下所示：

```text
GcContext::Reclaim(0x4f96030, 0x2)
    GcAlloc::ReclaimGarbage(0x4f9c580)

    // 作者举了一个 64 位下的例子进行说明，此时 variant 的第 12 位为 1（0x808）

    00000000`04f9ce20  00000000`00000808 00000000`01ab9ff8
    00000000`04f9ce30  00000000`00000000

        VariantClear(0x4f9ce20)

        // 到这里 Flag 已经被清除，即将清理相应的 variant

        00000000`04f9ce20  00000000`00000008 00000000`01ab9ff8
        00000000`04f9ce30  00000000`00000000

        VariantClear+6b returns 0x0
        GcContext::Adapt(0x4f96030, 0x1, 0x11)
        GcContext::Adapt+77 returns 0x6

    GcAlloc::ReclaimGarbage+24a returns 0x6
    GcContext::NormalizeBuckets(0x4f96030)
    GcContext::NormalizeBuckets+4c returns 0x24b978

GcContext::Reclaim+128 returns 0x1
GcContext::CollectCore+1bb returns 0x1
GcContext::Collect+43 returns 0x1
JsCollectGarbage+29 returns 0x0
```

现在，读者应该可以比较清晰地了解 Jscript 中的垃圾回收机制。

## 一波三折的漏洞修复过程

提及微软对此类漏洞的修复，比较早的一个修复方案要追溯到 `CVE-2018-8353`。这个漏洞是 Ivan Fratric 发现的，成因和本文中讨论的几个漏洞完全一致。微软当时的修复方案是将造成 `lastIndex` 加入 `Scavenge`（捡拾）列表，大致如下：

```text
VAR::Scavenge(RegExpObj.lastIndex, GcContextObj)
```

很明显，这是一种创可贴式的修补。它解决了当前问题，但没有解决此类问题。随后的 `CVE-2018-8653`、`CVE-2019-1367`、`CVE-2019-1429` 都是创可贴式的修补，直到 `CVE-2020-0674` 的修补方案出现。

在 `CVE-2020-0674` 的补丁中，微软在 Jscript 中加入了一个新的函数 `ScrFncObj::PerformCall`。之前对 `CallWithFrameOnStack` 和 `CallWithFrameOnHeap` 的调用都改为间接调用 `ScrFncObj::PerformCall` 进入。在 `ScrFncObj::PerformCall` 中，会将函数参数都加入到 `root` 列表中，这样就从根本上杜绝了 `CVE-2020-0674` 这类漏洞的产生。

笔者在进行补丁分析的过程中，查到启明星辰的一篇文章。这篇文章对 `CVE-2020-0674` 补丁进行了比较细致的分析，文末参考链接“补丁分析”处给出了文章的网址，读者可以进行参考。

有意思的是，在 `CVE-2019-1367` 的补丁中，微软在 Jscript 引入了一个 `GcContext::IsLegacyGCEnabled()` 函数。如伪码所示，这个函数用来检查是否开启旧的 GC 机制，方法是检查 `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Policies` 项下一个 `REG_DWORD` 类型的键：`ee1ca8aa-4402-4da1-bbe2-69a09c483a56` 的值是否为 1。若为 1，则不对导致漏洞的对象进行 GC 追踪。

```cpp
int __stdcall GcContext::InitIsLegacyGCEnabled(
    union _RTL_RUN_ONCE* a1,
    void* a2,
    void** a3
) {
    DWORD cbData;
    HKEY phkResult;
    DWORD Type;
    BYTE Data[4];

    *(_DWORD*)Data = 0;
    cbData = 4;
    Type = 0;
    GcContext::s_fIsLegacyGCEnabled = 0;

    if (!RegOpenKeyExA(
        HKEY_LOCAL_MACHINE,
        "System\\CurrentControlSet\\Policies",
        0,
        0x20119u,
        &phkResult
    )) {
        if (
            !RegQueryValueExA(
                phkResult,
                "6310d37c-a685-4133-81b7-01b42749b9b4",
                0,
                &Type,
                Data,
                &cbData
            ) &&
            *(_DWORD*)Data == 1 &&
            Type == REG_DWORD
        ) {
            GcContext::s_fIsLegacyGCEnabled = 1;
        }

        RegCloseKey(phkResult);
    }

    return 1;
}
```

笔者在打了 `CVE-2020-0674` 补丁的机器上新建了上述注册表项和键，并设为 1，发现 `CVE-2020-0674` 的补丁不再起作用，由此推测该注册表可能是为了兼容性问题所设。

## 写在最后

本文中笔者回顾了近两年出现的 4 个 Jscript 零日漏洞，借助 PoC 解释了这些漏洞的成因，分析了其中一个漏洞的在野利用方式，并对相关修补方案做了简要分析。

随着 `CVE-2020-0674` 的补丁方案的引入，此类 Jscript UAF 基本宣告落幕。这几个案例清楚地告诉我们：对漏洞修复工作者来说，如果对一类通用漏洞中的某个 Case 采用创可贴式的修复，不深挖根源，就会导致本文中这样连续绕过的故事，最后于时间和精力上都是巨大的开销。

## 参考链接

- `CVE-2018-8353`

  https://bbs.pediy.com/thread-257127.htm

  ```text
  97fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1N6h3N6K6i4K6u0W2j5$3S2J5L8$3#2A6N6h3#2Q4x3X3g2G2M7X3N6Q4x3V1k6H3i4K6u0r3M7s2u0G2K9X3g2U0N6q4)9J5k6s2A6W2M7X3!0Q4x3V1k6A6M7%4y4#2k6i4y4Q4x3V1k6V1k6i4c8S2K9h3I4Q4x3@1k6A6k6q4)9K6c8o6p5#2z5o6M7
  ```

- `CVE-2018-8653`

  ```text
  143K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2N6r3g2@1M7X3q4F1k6g2)9J5k6h3y4G2L8g2)9J5c8V1q4F1j5h3I4&6M7$3W2K6i4K6g2X3L8$3k6Q4y4h3k6o6g2V1g2Q4y4h3j5J5x3o6p5^5i4K6g2X3z5o6j5#2x3#2)9#2k6V1#2W2L8h3!0J5P5g2)9#2k6V1#2S2L8X3q4Y4k6h3#2W2L8Y4c8Q4x3X3g2Z5N6r3#2D9

  ba1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2E0j5$3q4X3k6h3g2Q4x3X3g2U0L8$3#2Q4x3V1k6T1L8r3!0Y4M7#2)9J5c8X3!0@1K9r3g2J5i4K6u0V1j5X3I4G2k6%4y4Q4x3V1k6E0j5$3q4X3k6h3g2Q4x3X3c8D9j5h3u0K6i4K6u0r3K9h3g2Q4x3X3c8K6j5%4u0A6M7s2c8A6L8X3N6Q4x3X3c8X3L8r3q4%4i4K6u0V1M7%4c8A6L8r3I4Q4x3X3c8S2i4K6u0V1N6r3S2J5k6h3q4@1i4K6u0V1N6r3!0Q4x3X3c8#2L8Y4m8S2N6r3y4Z5k6h3c8Q4x3X3c8K6P5i4y4@1k6h3#2K6i4K6u0V1j5h3&6S2L8s2W2\*7K9h3&6Y4i4K6u0V1j5%4k6W2i4K6u0V1x3U0l9I4z5q4)9J5k6o6R3$3y4e0y4Q4x3V1j5
  ```

- `CVE-2019-1367`

  ```text
  d71K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4L8$3!0Y4L8r3g2H3M7X3!0B7k6h3y4@1P5X3g2J5L8#2)9J5k6h3u0D9L8$3N6K6M7r3!0@1i4K6u0W2j5$3!0E0i4K6u0r3M7q4)9J5c8Y4u0U0j5g2)9J5k6r3y4$3k6g2)9J5k6o6t1H3x3e0W2Q4x3X3b7I4x3K6j5%4i4K6u0W2K9s2c8E0L8l9
  ```

  ```text
  ba6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5$3!0F1k6X3W2S2L8Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6A6L8Y4c8W2M7X3&6W2N6q4)9J5k6r3g2^5M7r3I4G2M7X3g2J5i4K6u0V1j5%4k6W2i4K6u0V1x3U0l9I4z5g2)9J5k6o6p5K6y4U0N6Q4x3X3c8W2P5s2m8D9L8$3W2@1j5i4c8A6L8$3&6Q4x3X3c8H3j5i4u0@1i4K6u0V1x3g2)9J5k6o6N6X3k6U0l9^5j5U0N6V1j5$3x3^5j5R3
  ```

  ```text
  bbdK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5$3!0F1k6X3W2S2L8Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6A6L8Y4c8W2M7X3&6W2N6q4)9J5k6r3g2^5M7r3I4G2M7X3g2J5i4K6u0V1j5%4k6W2i4K6u0V1x3U0l9I4z5g2)9J5k6o6p5K6y4U0N6Q4x3X3c8W2P5s2m8D9L8$3W2@1j5i4c8A6L8$3&6Q4x3X3c8H3j5i4u0@1i4K6u0V1x3W2)9J5k6o6R3I4y4o6x3J5y4o6u0T1y4e0M7^5x3l9
  ```

- `CVE-2019-1429`

  ```text
  858K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1N6h3N6K6i4K6u0W2j5$3S2J5L8$3#2A6N6h3#2Q4x3X3g2G2M7X3N6Q4x3V1k6H3i4K6u0r3M7s2u0G2K9X3g2U0N6q4)9J5k6s2A6W2M7X3!0Q4x3V1k6A6M7%4y4#2k6i4y4Q4x3V1k6V1k6i4c8S2K9h3I4Q4x3@1k6A6k6q4)9K6c8o6p5&6y4o6M7
  ```

- `CVE-2019-0674`

  https://bbs.pediy.com/thread-259500.htm
