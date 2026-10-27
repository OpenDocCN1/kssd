
（请参阅 `BugCheck 0x109：CRITICAL_STRUCTURE_CORRUPTION` 的文档）。

接下来有一个指向要检查的数据的指针以及要检查的大小。

重要的值是校验和结果。

此校验和在 PatchGuard 初始化期间计算，并在 PatchGuard 检查相应结构的完整性时用作参考。

最后，此结构中的最后一项特定于必须检查的数据。

例如，对于 IDT 检查用例，此特定值将保存已用于执行的目标处理器。

一般来说，这意味着该结构在检查结构方面可能有所不同，并表明检查代码对于所有结构并不完全相同。

### PatchGuard 上下文结构中的相对条目

这些结构存储在 PatchGuard 上下文结构中的数组中。

PatchGuard 上下文结构的第一部分中存在几个条目以使用此数组：

```
0x680: Total count of critical structure in the array
0x684: Offset to next critical structure data to checksum
0x6a8: Offset to the first critical structure data
0x6ac: Current count of checked structure
```

这些信息很重要，PatchGuard 在其检查算法中使用这些信息。

### 2 - PatchGuard 上下文：初始化

PatchGuard 上下文主要由 `KiInitPatchGuardContex`t 初始化。

这个函数实际上是未命名的，但在文献中是已知的。

我们将在本节中看到存在其他方法来初始化 PatchGuard 上下文，

并且在某些情况下，设置了一些独立的系统检查方式。

#### a - KiInitPatchGuardContext: Method 0, 1, 2, 3, 4, 5, 7

如前所述，该函数负责大多数 PatchGuard 上下文的初始化。

使用哪种方法的选择取决于赋予此函数的参数。

正如我们在 `KiFilterFiberContext` 概述中所描述的，这些参数大多是随机选择的。

在本节中，我们将介绍给此函数的参数，该参数将描述 PatchGuard 检查是如何初始化和触发的。

以下是此函数的参数：

• - Arg 1：DPC 方法的索引

• - `Arg 2：调度方法`

• - Arg 3：用于确定要检查的最大大小的随机值

• - Arg 4：指向来自 `ExpLicenseWatchInitWorker` 的结构的指针（只有 4 % 的机会）

• - Arg 5：布尔值，用于确定 nt 例程的完整性是否具有待检查

在我们的例子中，最重要的参数是第二个（用于安排检查的方法）和第四个（允许更多的调度方法）。

在 `KiFilterFiberContext` 中，随机值作为第二个参数的索引，它将决定应该使用什么方法。

在本节中，我们将首先描述 `KiInitPatchGuardContext` 可能初始化的不同方法以及关于该方法的第四个参数。

然后我们将快速浏览其他参数。

##### i - 方法 0 - 插入与 DPC 链接的计时器

该方法的主要思想是 PatchGuard 将初始化 PatchGuard 上下文结构和 DPC（延迟过程调用），并将其设置在计时器结构中。

然后计时器在 0x1408A8920 附近与 `KeSetCoalescableTimer` 一起排队。

计时器在调用后将用 2' 到 2'10" 之间的第一个参数触发 DPC。

此计时器不是周期性的，必须在检查例程结束时恢复，我们将在本文后面看到

`TolerableDelay` 参数是 0 到 0.001 秒之间的随机值

##### ii - 方法 1 和 2 - 隐藏 DPC

当 `KiInitPatchGuardContext` 的第二个参数为 1 或 2 时，

PatchGuard 初始化上下文结构和 DPC 结构，但不使用计时器, 将其隐藏在内核结构 `PRCB`（进程寄存器控制块）中。

这种方法的有趣之处在于系统中的合法函数实际上负责对DPC进行排队。

• `AcpiReserved`

对于方法1，指向DPC的指针隐藏在来自PRCB的`AcpiReserved`字段：

```
loc_1408A890C:
mov rax, [rsp+2238h+KPRCB_var_308]
mov [rax+_KPRCB_.AcpiReserved], r8 ; DPC initialized by PatchGuard
jmp loc_1408A89CE
```

它在 `HalpTimerDpcRoutine` 中排队，并检查每次检查之间是否至少经过了两分钟。

为了计算最后一个队列发生的时间，它使用全局变量 `HalpTimerLastDpc`。

这个全局变量是在 `HalpTimerSchedulePeriodicQueries` 中初始化的，它的值取自全局变量 `0xFFFFF78000000014`，这与正常运行时间有关（我认为是机器的，但我不确定）。

当某个 ACPI 事件发生时调用 `HalpTimerDpcRoutine`，例如过渡到空闲状态。

• `HalReserved`

对于方法 2，指向 DPC 的指针隐藏在来自 PRCB 的 HalReserved 字段中：

```
loc_1408A88F8:
mov rax, [rsp+2238h+KPRCB_var_308]
mov [rax+(_KPRCB_.HalReserved+38h)], r8 *; DPC initialized by PatchGuard*
jmp loc_1408A89CE
```

旁注：

回想一下，当从 `ExpLicenseWatchInitWorker` 调用 `KiFilterFiberContext` 时，该字段（但该数组的条目）也用于保存指向结构 `KI_FILTER_FIBER_PARAM` 的指针。

它由 `HalpMcaQueueDpc` 排队，最短周期也为 2 分钟，并在发生 HAL 定时器时钟中断时完成检查（请参阅 `HalpTimerClockInterrupt/HalpTimerAlwaysOnClockInterrupt`）

##### iii - 方法 3 – 系统线程

这种情况需要一个指向 `KI_FILTER_FIBER_PARAM` 结构的指针，它只有 4% 的机会发生（来自函数 `ExpLicenseWatchInitWorker`，在 II - A - 2 - a 中解释）。

前面已经展示了这个结构的概述，但请记住，它包含一个指向 `PsCreateSystemThread` 函数的指针。

该指针用于在函数 `sub_1408A9518`（我们方便地将其命名为 `Pg_InitMethod3SystemThread`）中创建一个新的系统线程，

使用函数 `sub_1402CD680`（在 `KI_FILTER_FIBER_PARAM` 结构中偏移 0x10，

它是验证例程的存根，因此我们方便地将其命名为 `Pg_Method3StubToCheckRoutine_sub_1402CD680`）作为起始地址。

`Pg_InitMethod3SystemThread` 直接在 `KiInitPatchGuardContext` 的 0x1408A5B88 处调用。

需要注意的一件有趣的事情是添加了优雅的混淆。

这个想法是一些绕过用于从 ETHREAD 结构中定位条目 StartAddress 和 Win32StartAddress 以识别 PatchGuard 线程，因此在 Windows 10 中，他们使用通用函数指针修改了这些条目：

在线程创建后，PatchGuard 立即获取指向相应的指针ETHREAD（只是说没有锁）并修改 StartAddress 和 Win32StartAddress 两个字段：

```
lea rcx, Pg_FuncArray_off_1408F71E0

mov rcx, [rcx+rax*8] *; rax is a random value*

mov rax, [rsp+0A8h+var_68]

mov [rax+ETHREAD_.anonymous_1.anonymous_0.StartAddress], rcx

mov [rax+ETHREAD_.Win32StartAddress], rcx
```

为此，它首先获取 0 到 7 之间的随机值，

并在内存中偏移 `Pg_FuncArray_off_1408F71E0` 的数组中获取函数指针。

这是这个数组的内容

```
index Function name
0 KeBalanceSetManager
1 KeSwapProcessOrStack
2 ExpWorkerThread
3 PopIrpWorker
4 FsRtlWorkerThread
5 EtwpLogger
6 **Pg_Method3StubToCheckRoutine_sub_1402CD680**
```

只有最后一项是正确的，这意味着 ETHREAD 结构中的 StartAddress 和 Win32StartAddress 文件只有七分之一的机会是正确的。

##### iv - 方法 4 - 异步过程调用

第四种方法初始化一个 PatchGuard Context 结构和一个 APC 结构，并直接将其插入到现有系统线程中。

`NormalRoutine` 参数设置为 `xHalTimerWatchdogStop`，它实际上只是一个 «ret 0» 指令。

`KernelRoutine` 设置为 `KiDispatchCallout`，它将以某种方式调用验证例程，而 `RundownRoutine` 为 NULL。

这些参数设置为 0x14089555B（上下文结构中函数指针的初始化）和 0x1408A8734（为 `KiInsertQueueApc` 调用准备参数）

它选择附加到哪个线程的方式是使用带有回调 `Pg_IsStartAddressPopIrpWorkerControl_sub_1408A9B70` 的 `PsEnumProcessThreads` 完成的，该任务是查询线程起始地址并将结果与 `PopIrpWorkerControl` 进行比较。

如果要找到此类线程，则将指向 `ETHREAD` 结构的指针存储在 PatchGuard 上下文结构的偏移量 0x830 处，然后将其复制到提供给 `KeInsertQueueApc` 的 `KAPC` 结构中。

##### v - 方法 5 – 挂钩常规 DPC

与方法 3（使用系统线程）一样，此方法需要有效的 `KI_FILTER_FIBER_PARAM` 结构，否则 `KiInitPatchGuardContext` 将回退到方法 0。

对于此方法，使用结构的最后一个条目，即是指向全局变量 `KiBalanceSetManagerPeriodicDpc` 的指针。

该变量包含一个 `KDPC` 结构，其 DPC 例程在函数 `KiInitSystem` 中初始化。

这种方法的优雅之处在于它实际上是一个合法的 DPC，

它由系统每秒左右通过 `KeClockInterruptNotify` 在 0x1400619b6 处排队；

并且 PatchGuard 钩住这个合法的 DPC，

以便每 120 个队列（实际上，像许多其他方法一样，120 到 130 次之间的随机值），PatchGuard DPC 排队而不是合法的一个。

这是简化此机制代码的图表

![图片描述](upload/attach/202208/93_7MRE2R7V9K8Z7AC.png)

如果要对 PatchGuard DPC 进行排队，那么它首先会清除全局 DPC 的副本，并让验证例程在检查结束时将其设置回来。

##### vi - 方法 7 - 新的怪异方法。

（不，没有方法6，我对此没有任何解释。）

乍一看，这种方法……什么都没有。

嗯，几乎没有。

它实际上做了两件事。

第一件事是它初始化一个要排队的 DPC，但在之后立即清除它，所以永远不要排队。

第二个是它初始化一个全局 PatchGuard 上下文结构，该结构将通过系统的全局指针可用。

这个全局 PatchGuard 上下文结构实际上在内存中是明文的，并且保留在初始化函数的末尾。

在这一部分中，我们将描述我们的发现，特别是未使用的 DPC 的内容。

• 未使用的 DPC

当索引 7 被赋予 `KiInitPatchGuardContext` 时，会采用许多特定的分支。

特别是，初始化 DPC 并将例程定义为 `KiInterruptThunk` 函数之一或 `KiMachineCheckControl` 函数之一。

`KiInterruptThunk` 和 `KiMachineCheckControl` 都是一组 16 个存根，

分别对应于函数 `FsRtlTruncateSmallMcb` 和 `KiDecodeMcaFault`，

它们依次调用检查例程 `FsRtlMdlReadCompleteDevEx`。

在初始化函数 `KiInitPatchGuardContext` 中，使用的是 `KiInterruptThunk` 函数，

但我们稍后会看到其他 PatchGuard 例程中存在对 `KiMachineCheckControl` 的一些引用。

要使用这个函数数组，会生成一个从 0 到 0xf 的随机值（rdtsc & 0xf），然后用作这些存根中的索引。

尽管每个函数有 16 个存根可用，但只有两种不同类型的存根：

一种在调用检查例程之前清除 DR7（调试寄存器），

另一种则不清除。

这是 `KiInterruptThunk` 函数的两个不同存根

```
33 C0 xor eax, eax
90 nop
90 nop
90 nop
E9 F6 AF 12 00 jmp FsRtlTruncateSmallMcb
66 0F 1F 44 00 00 align 10h
```

```
33 C0 xor eax, eax
0F 23 F8 mov dr7, rax
E9 E6 AF 12 00 jmp FsRtlTruncateSmallMcb
66 0F 1F 44 00 00 align 10h
```

由于 NOP 说明，两者或完全相同的尺寸。

这两个存根重复 8 次，随机值用于选择其中一个。

对于 `KiMachineCheckControl` 函数，存根几乎相同，

不同之处在于调用 `KiDecodeMcaFault` 而不是 `FsRtlTruncateSmallMcb`。

现在，正如我们之前所说，这种方法的问题在于它似乎没有做更多的事情。

其他方法通过将 DPC 与计时器耦合或将其放在内存中的某个位置来使用 DPC，以便系统可以在某个时间点对其进行排队，但这个没有。

这是详细说明我们发现的技术分析。

尽管不能证明没有任何路径可以让这个 DPC 排队，但它会展示我们对这种方法的一些研究。

技术分析：使用 Reven 作为时间旅行调试器，我们跟踪了这个初始化的执行，找到了为什么没有这个方法的处理程序。

• 第一次检查：使用 0x10000000 测试标志

从随机选择 `KiInterruptThunk` 存根的块开始，我们在 1408A8308 处发现对标志的检查：

```
test [rsp+2238h+flag_828_on_stack_var_140], 10000000h
```

让我们分析一下这个标志是从哪里来的。

这个标志来自 PatchGuard 上下文，我们可以使用内存历史来找出它来自哪里。

通过 reven 的 Memory History 功能检查了几个 memcpy，我们发现该标志的设置在 0x140891B60。

以下是如何使用内存历史记录来查找此内容的屏幕截图：

多次使用相同的方法，这里是标志 0x10000000 的结果摘要。

如屏幕截图所示，这是负责设置标志的代码：

```
mov ecx, [r14+pg_ctx_rs4.multiple_flag_0x828]
mov eax, r12d
btr ecx, 1Ch
shl eax, 1Ch
or ecx, eax
mov edx, 2000h
mov [r14+pg_ctx_rs4.multiple_flag_0x828], ecx
```

它是根据 r12d 的值来设置的。

再次进行时间旅行调试，发现这个寄存器设置为0x140891707：

```
mov r12d, dword ptr [rsp+2238h+var_bIsMethod7_2158]*
```

同样，使用此堆栈内存位置的 Memory History 功能，我们发现它已设置为 140890A13：

```
cmp esi, 7
mov rdi, 0CCCCCCCCCCCCCCCDh
cmovz ebx, r12d *; r12 = 1*
mov dword ptr [rsp+2238h+var_bIsMethod7_2158], ebx
```

这里 esi 包含调度方法，即方法 7。

最后一条数据是 r12，但静态我们很容易发现它被硬编码为 1，与控制流无关。
