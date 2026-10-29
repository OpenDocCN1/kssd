
# [翻译]如何利用iOS 7 bootchain 中的递归堆栈溢出漏洞

## 目录表

- 0 - 引言
- 1 - 研究
  - 1.0 - 谁是谁?
  - 1.1 - ASL拓展
  - 1.2 - 叫我自己的名字
  - 1.3 - 看见未见的
- 2 - 漏洞利用
  - 2.0 - 躲避子弹
  - 2.1 - 过河烧桥
  - 2.2 - 直捣黄龙
  - 2.3 - 清扫战场
  - 2.4 - 放手一搏
- 3 - 结论
- 4 - 引用
- 5 - 源码

---

## 0 - 引言

这篇文章旨在阐释如何利用iOS 7 bootchain中的递归栈溢出的bug。无需漏洞利用的前置知识，只需稍微了解一些向下增长小端栈的机器的工作原理和一些ARM汇编的基础知识。不需要特定的硬件（线缆或调试设备）来实现对`iBoot`的确定性控制。在本文中我们将使用`iPhone5,2/11B554a`作为示例。

有两个看似无关的东西导致了这一漏洞的发现。

- 其中一个原因是我隐约记得2010年的年中，丰田因为问题固件引发意想不到的加速丑闻而饱受指责。这其中一些错误被认为源自于破坏全局变量的堆栈溢出[1]。
- 另一件事是Joshua Hill的“破碎的梦”演示，它在第85页有这个”BootROM漏洞利用方法”，提到“递归堆栈溢出”[2]。

## 1 - 研究

### 1.0 - 谁是谁?

iPhone的bootchain的一个简短的总结

```
NAND/Normal +------+ +-------+
------------> | LLB | --> | iBoot | --> OS
+-----------+ / +------+ +-------+
| SecureROM |<
+-----------+ \ +------+ +-------+
------------> | iBSS | --> | iBEC | --> Restore
USB/DFU +------+ +-------+
```

SuffelRM是一小块掩模ROM或者写保护闪存。这是设备重启后第一个运行的东西。

`LLB`是Low Level Bootloader，负责硬件加载和加载主引导程序。

`iBoot`是主引导程序，主引导程序几乎做了你能想到的所有事情：USB、恢复模式，等等。它还处理FS访问，因为它需要从系统分区中读取内核并引导OS。

`iBSS`是`LLB`的DFU副本。它处理FS访问，因为它需要在升级过程中读取一个特殊的“恢复”分区。

实际上，引导逻辑比这更复杂一些，例如`iBoot`可以回退到恢复模式，它将接受一个`IBEC`，但我们不关注这一点，因为已经超出本文的范围。

由于它们的性质和目的，每个引导加载阶段带来了复杂性的增加。例如：`LLB`/`iBSS`不需要了解文件系统，但如上所述，`iBoot`/`iBEC`必须这样做。从统计上看，一个组件越复杂，其bug的几率就越高，因此，我们想要f寻找可轻易实现的目标：

### 1.1 - ASL拓展

首先，我们需要研究我们的目标，这听起来很复杂，但实际上不是。`iBoot`并不意味着可重定位，它被设计为在其首选的负载地址运行，即，`memory_base + memory_size - 1MB`，也就是：`0x5FF00000` 或 `0x9FF00000` 或 `0xBFF00000`。这意味着没有`IASLR`（iBoot地址空间布局没有随机化），这是好事。

在一个越狱的手机上iOS已经完全启动之后，我们应尽快dump `iBoot`。运气好的话，我们会发现`iBoot`仍然在自己的位置，几乎完好无损。为了dump，我们将使用winocm的`ios-kexec-utils`（3）工具，下面我们把它叫做`kloader`和它的小弟弟`kdumper`。

请注意，被dump的镜像与解密的镜像略有不同。好的是，数据在运行时保持不变，显著的例外是这个列表。我们通过对这些列表头进行清理，并基于旧的`iBoots`（如iPhone 4 `iBoot`）进行更多的清理工作，因为我们有那些的解密密钥[4]。

```
+------------------+
| ARM reset vector |
|                  |
| CODE             |
|                  |
+------------------+
|                  |
| DATA             |
|                  |
| +--------------+ |
| | btree data   | |
| +--------------+ |
| +--------------+ |
| | allocator    | |
| | meta-data    | |
| +--------------+ |
|                  |
+------------------+
|                  |
| HEAP            |
| +--------------+ |
| |task structure| |
| |--------------| |
| | task stack   | |
| +--------------+ |
| stuff           |
| +--------------+ |
| |task structure| |
| |--------------| |
| | task stack   | |
| +--------------+ |
| stuff           |
|                  |
+------------------+
```

我们可以在上面的图中观察到任务堆栈被放置在数据后面，可以推断，破坏这些堆栈中的一个将导致数据损坏。刚开始，只有一个任务运行：引导任务。然后创建一些其他任务，如`idle`和`main`，然后又会产生更多的任务：`poweroff`, `USB`等，并运行它们。每个任务都有固定的堆栈大小：`0x1C00`字节。

还有一个简单的调度器管理这些任务，这些任务也执行一些轻量级的完整性检查，比如验证ARM重置向量基地址和创建任务的堆栈基地址。任务调度器是协作的，因此我们可以安全地假设一旦任务运行，它将继续这样做直到一些发生异常，例如IRQ。简而言之，堆栈检查不影响我们，只要`task_yield()`不在我们堆栈时运行。

在多次dump `iBoot`之后，我们不仅可以看到基地址，而且可以看到它的整个内存布局是完全可预测的，从漏洞利用的角度来看，这可太好了。我们可以找到堆中的代码、数据，甚至各种结构。当然，我们需要关注那些堆栈，这就是我们的目标，对吧？

### 1.2 - 叫我自己的名字

请记住，递归堆栈溢出并不一定意味着直接递归，一个la函数`F()`调用它自己。这些都很难被发现，但并不总是有用的。例如，`ResolvePathToCatalogEntry()`被限制最大深度为64，因此对于这个目的是无用的。

让我们寻找`F() -> G() -> ... -> F()`链，因为那些仍然会被算做递归。虽然在大量文件中发现这些有点困难，但是有一些图形算法能帮我们：我们有一个调用图，我们需要在这个调用图中标识“循环”。一个相当简单和有用的算法是Tarjan的强连通分量算法[5]。实际上，SCC可能包含多个循环，但是没关系；实际上，假设SCCs已经足够好了。ONS被正确识别，我们可以运行脚本[6]并打印出我们的调用图“循环”。

我们在找那些小巧的SCC。就像`F()->G()->F()`，因为那些容易跟踪。下面这个就不错：

```
ReadExtent()
memalign(64)
ReadExtentsEntry()/ReadBTreeEntry()
memalign(blockSize)
ReadExtent()
```

上面的`memalign()`调用本身并没有对递归做出贡献，但它们在后面有重要作用 - `memalign`只是一个奇特的`malloc`。另外，请记住，没有太多*其他的事*发生，这是很好的。

一个HFS+的快速入门会让我们更好地了解正在发生的事情。有关HFS+[7]卷内所有文件和文件夹的信息保存在目录文件中。目录文件是一个包含记录的B树[8]，每个记录最多跟踪文件每个分支的8个范围[9]。这被称为程度密度。一旦所有8个扩展区都用完，附加扩展区就会记录在扩展区溢出文件中。Extents溢出文件是另一个B-tree，它将记录分配给每个扩展文件的分配块。

在`iBoot`中发现的HFS+实现使用相同的`ReadExtent()`函数来读取目录和扩展溢出范围。我们观察到，如果在读取扩展溢出范围时超出了范围密度，则`ReadExtent()`将无限递归。

请注意:碰巧，一个过时版本的`iBoot` HFS+驱动程序是公开的[10]。源代码对漏洞利用必要的，但它可能有助于了解一些。

好消息：我们在`main`任务中找到了一个递归堆栈溢出，堆栈是高度可预测的。

坏消息：它会烧穿堆并且很快击中DATA，而且一旦触发无法阻止。

更坏的消息：`iBoot`堆（和部分数据）将完全失效。

### 1.3 - 看见未见的

当时，我只有一个设备易受这个bug的影响，我很担心失去它。只是盲目地拨弄`iBoot`是愚蠢的，所以我写了一个“iBoot loader”命名为`iloader`（附在本文中的源代码）。它的目的是在用户态伪造运行`iBoot`，利用`iBoot`的运行时内存布局的高度可预测的特性。

由于我们在用户态上是伪造的，所以我们必须跳过硬件。现在回想一下，堆栈上分配了任务堆栈，跳过整个代码块可能跳过一些`alloc`。为了弥补这一点，我们需要确保`iloader`保持主任务堆栈在`iBoot`基地址上的正确偏移。我们`kloader`修改了`iBoot`，并打印出“主”任务的栈顶例程中的堆栈指针：

`SP = BASE + 0x581d4`

对齐我们的堆栈，然后需要在创建主任务之前`iloader`以正确大小调用`iBoot`分配器一次。

```
Real iBoot heap:                            iloader iBoot heap:

|          |                                |          |
+--------------------+                     +--------------------+
| heap block used by |                     |                    |
| things we skipped  |                     |                    |
+--------------------+                     |                    |
+--------------------+                     |                    |
| heap block used by |                     | dummy allocation   |
| things we skipped  |                     | required to align  |
+--------------------+                     | the stack pointer  |
+--------------------+                     |                    |
| heap block used by |                     |                    |
| things we skipped  |                     |                    |
+--------------------+                     +--------------------+
+--------------------+                     +--------------------+
| main task structure|                     | main task structure|
+--------------------+                     +--------------------+
| main task stack    |                     | main task stack    |
+--------------------+                     +--------------------+
|                    |                     |                    |
```

## 2 - 漏洞利用

### 2.0 - 躲避子弹

在寻找触发漏洞的方法之前，有两件事要做。

考虑：

1. 我们必须避免什么？
2. 我们应该瞄准什么？

回答第1题，是的，在我们的路径上有些东西。还记得那些`malloc`调用吗？`malloc`函数使用`enter_critical_section()`/`exit_critical_section()`来防止竞争条件。那两个函数执行`task::irq_disable_count`而且大于999的话就会进入应急模式，触发递归将会清除他的路径上的一切，包括我们的`main`任务，也包括`task::irq_disable_count`。所以我们需要确定`main_task->irq_disable_count`的位置被小于1000的值所重写。

To answer #2, yes there is something that could achieve an arbitrary write.

Remember those `malloc` calls again? Yeah, it follows that we should target the allocator metadata. We have to put some values there and coerce the allocator into an arbitrary write.

回答第2题，是的，有些东西可以实现任意的写入。还记得那些`malloc`调用吗？对，我们应该以分配器元数据为目标。我们在其中放一些值，并将分配器强制任意写入。

```
|          |
+------------------+
|                  |
| DATA             |
|                  |
| +--------------+ |
| |btree headers | | <-+ steer the allocator here
| +--------------+ | |
| +--------------+ | |
| | allocator    | | --+
| | meta-data    | | <-+ land here
| +--------------+ | |
|                  |
+------------------+ |
|                    |
| HEAP               |
| +--------------+   |
| |task structure|   | <-- this gets nuked, avoid irq_disable_count
| |--------------|   |
| | task stack   |   | <-- this is where we start
| +--------------+   |
| stuff              |
```

要想在这些约束下起舞，我们需要确保在非常精确的点上开始递归。我们跨越分配器元数据的确切位置是由每个递归占用多少堆栈空间（在下面的例子中是208个字节）和起始SP。我们需要足够接近来操作分配器，但不能太接近，否则它会恐慌。我们稍后会看到，块“bin”阵列是甜蜜的地方着陆。SP调谐可以通过两种方法实现：

- `try both ${boot-path} / ${boot-ramdisk} and choose whichever is best`
  `as both can be set from within iOS and are preserved across reboots.`
- `${boot-path}` 和 `${boot-ramdisk}`两个都试试，选择哪个是最好的，因为两者都可以从iOS内部设置，并在重新启动时保存。
- `ResolvePathToCatalogEntry()`下探目录层次结构，并为`ReadExtent`提供不同的起始`SP`，但记住它具有最深64的深度限制。

### 2.1 - 过河烧桥

`iBoot`以简单的方式访问文件系统。它在"装载"分区时缓存两个BTree头，然后使用`ReadExtent()`用于文件读取以及目录扫描。

递归可以通过两种方式开始：

- 给根节点设置一个很高的值，或者
- 使用长路径，并且在部分地钻下该路径之后，从索引节点的`BTRAGE`记录中获取高节点数。

从一个简单的HFS+系统开始，有一个相当平坦的Btree:

```
trigger
|
v
+------+       +------------+
|HEADER|------>| root block |
+------+       +------------+
                |    ...    |
                +--------------+
                       v         v
                    +-----+     +-----+
                    |     | ... |     |
                    +-----+     +-----+
```

上述BTree布局允许我们仅通过根节点触发递归。叶节点不能用于触发延迟递归，因为`BTNodeDescriptor::kind`在`Read ReadBTreeEntry()`中被检查，以防我们得到一个具有高编号的当前节点的任意索引节点。

配置示例就像这样：

```
HFSPlusVolumeHeader::catalogFile.logicalSize = 0xFFFFFE00;
HFSPlusVolumeHeader::extentsFile.logicalSize = 0x3FFC000;

Catalog BTree header:
BTHeaderRec::nodeSize = 512;
BTHeaderRec::totalNodes = 0x7FFFFF;
BTHeaderRec::rootNode = 0x7FFE; // initial trigger

Extents BTree header:
BTHeaderRec::nodeSize = 16384;
BTHeaderRec::totalNodes = 0xFFF;
BTHeaderRec::rootNode = 0x500; // must be big, but LSB must be zero
```

注意:这个Btree头的大部分空间都没有使用，也没有被`iBoot`检查,所以我们可以使用这个空间来放我们的payload。

上述策略虽然简单，却证明是相当不灵活的，因为我们不能调整启动SP太多。如前所述，我们希望使用`ResolvePathToCatalogEntry()`来为我们修复SP，这意味着在路径处理过程中触发任意点的递归。我们不得不求助于稍微不同的布局：复制根块，更新HFS+头，以在新根开始，并将克隆点内的所有记录都保留到原始根块。这将导致更高的树，但中间层不具有叶约束，但它仍然有效的HFS+（虽然略微浪费）。

```
+------------+
+--->| root copy |<-- trigger here
|    +------------+
|    |    ...    |
|    v          v
+------+       +------------+
|HEADER|--+    | root block |
+------+  |    +------------+
          |    |    ...    |
          +--------------+
                      v    v
                    +-----+ +-----+
                    |     | ... |     |
                    +-----+ +-----+
```

