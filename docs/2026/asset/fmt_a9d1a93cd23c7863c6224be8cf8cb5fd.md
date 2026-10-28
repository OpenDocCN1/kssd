
反检测最重要的原则不是把某个特定的 bit 伪造好，而是**所有的返回值必须一致**。如果你让 `CPUID` 返回“SVM 不支持”，那 `EFER.SVME` 必须返回 `0`，`VM_CR` 必须返回锁定状态，`VMRUN` 执行必须触发 `#UD`——只要有一个地方矛盾，检测方就能判断出有 Hypervisor 在运行。

## 十一、多核、TLB、中断——那些会让你蓝屏的坑

### 11.1 每 CPU 独立结构

每个 CPU 核心需要独立的：VMCB（Guest+Host）、HSAVE、Host 栈、NPT 页表。NPT 的 PML4 和 PDPT 层可以共享（因为它们映射的是全局地址空间），但 PD 和 PT 层需要独立——因为同一时刻，CPU A 可能在执行 Hook（NPT 指向影子页 1），而 CPU B 在读同一函数（NPT 指向影子页 0）。

启动 SVM 时，通过 `KeSetSystemAffinityThread` 依次把线程绑定到每个 CPU，在该 CPU 上执行 `VMRUN`。

### 11.2 TLB 一致性

每次修改 NPT PTE 后，必须刷新 TLB。否则 CPU 可能使用缓存的旧翻译，导致：

- 切到影子页 1 后还在执行影子页 0 的内容（Hook 不触发）
- 切回影子页 0 后还在执行影子页 1 的内容（暴露时间窗口变长）

AMD SVM 通过 VMCB 中的 `TlbControl` 字段控制 TLB 刷新：

- `0x01`：Flush 当前 ASID 的所有 TLB 条目
- `0x03`：Flush 所有 ASID 的所有 TLB 条目
- `0x07`：Flush 当前 ASID 的所有 TLB 条目，包括全局页

设置 `TlbControl` 后，下次 `VMRUN` 时硬件自动完成刷新，不需要执行 `INVLPGA` 指令。每个 CPU 使用不同的 ASID（`GuestAsid = cpuIndex + 1`），这样可以利用 ASID 标记避免不必要的全局刷新。

### 11.3 VMEXIT 期间的中断

VMEXIT 发生时，硬件自动禁用中断（相当于执行了 `CLGI`）。在 `VmExitHandler` 执行期间，中断是关闭的，这意味着：

- VMEXIT 处理必须尽快完成，不能做耗时操作（不能调用会 `Wait` 的内核函数，不能打印日志到串口等慢设备）
- 需要分配内存时只能用预分配的页池，不能调用 `ExAllocatePool`
- `VmExitHandler` 中不能持有自旋锁后再尝试获取另一个锁（死锁风险）

如果需要在 VMEXIT 中做复杂操作，应该把工作通过队列 DPC/工作项延后到 IRQL 降低后执行。

### 11.4 NPF 注入的 Guest `#PF`

如果 NPF 不是 Hook 引起的（比如 Guest 真的发生了缺页），你需要向 Guest 注入一个 `#PF` 异常，让 Guest 的缺页处理函数来处理。事件注入通过 VMCB 的 `EventInj` 字段：

```c
EVENTINJ inj = {0};

inj.Vector = EXCEPTION_PF;  // 14

inj.Type = 3;               // Exception type

inj.Valid = 1;              // 注入有效

inj.ErrorCode = pfErrorCode; // 和正常#PF一样的错误码

vmcb->ControlArea.EventInj = inj;
```

`VMRUN` 时硬件会自动把这个异常注入到 Guest 中，Guest 的 `#PF` 处理函数正常执行，跟真实发生缺页一模一样。

### 11.5 电源状态

Windows 休眠（S3）时 CPU 状态会丢失，唤醒后 VMCB/NPT 等状态无效。需要注册 `\Callback\PowerState` 回调：

- 休眠前：在每个 CPU 上通过 `VMMCALL` 让 VM 循环退出，清除 `EFER.SVME`
- 唤醒后：重新初始化 VMCB/NPT，重新进入 VM

不处理电源状态的后果：唤醒后导致三重故障。

## 十二、不同 Hook 设计方案的对比与取舍

了解了基本方案后，你需要知道：NPT Hook 并非只有一种实现方式，不同方案有不同的性能和隐蔽性。

### 方案 1：影子页切换 + `INT3`（本文主讲的方案）

```text
默认：NPT→影子页0(NX=1) → NPF取指 → 切影子页1(NX=0) → INT3 → #BP → 跳跳板 → 复位
```

- **优点**：实现简单，稳定，影子页 1 暴露窗口极短（单条 `INT3` 的时间），隐藏了跳板的位置。
- **缺点**：每次 Hook 触发产生两次 VMEXIT（NPF + BP），开销略大。

### 方案 2：影子页切换 + `jmp`

不修改页内容，而是准备两套 NPT：

- NPT-A（不可执行视图）：W=0，NX=1（禁止执行）
- NPT-B（读/执行视图）：被 Hook 页映射到含 `jmp` 的影子页，W=0，NX=0。

NPF 时根据错误码切换 `VMCB.nCR3` 指向不同 NPT 根，利用 ASID 标记避免 TLB flush。

- **优点**：不需要 `INT3`，不需要 BP 拦截；TLB 条目可以同时缓存两套翻译（因为 ASID 不同），减少 flush 开销。
- **缺点**：实现更复杂（需要覆盖 2 到 3 条汇编指令，容易出错），且跳板位置暴露，存在隐蔽性问题。

## 十三、工程实现清单——从零开始要写哪些模块

以下是一个 NPT Hook 框架的完整模块清单，按照实现顺序排列：

### 阶段 1：基础骨架（不做 Hook，先让 Hypervisor 跑起来）

| # | 模块 | 内容 |
| --- | --- | --- |
| 1 | CPUID 检测 | 检测 AMD/SVM/NPT/SVMDIS |
| 2 | 物理内存分配器 | 物理连续页分配（`MmAllocateContiguousMemory`）、页池（预分配 + 位图管理） |
| 3 | 汇编入口 | `.asm` 文件：VM 循环、`VMRUN` 进入、VMEXIT 保存/恢复寄存器、`VMMCALL` 封装 |
| 4 | VMCB 管理 | VMCB 分配、Control Area 初始化、State Save Area 填充（读当前寄存器） |
| 5 | Host 状态 | Host 栈分配、Host VMCB、HSAVE、MSRPM/IOPM |
| 6 | NPT 页表 | 四级页表构建（2MB 大页恒等映射）、大页拆分、地址翻译、PTE 修改、懒更新 |
| 7 | VMEXIT 分发 | 基本的 ExitCode switch-case，处理 NPF（先全部注入 Guest `#PF`）、MSR 直通、SHUTDOWN 蓝屏 |
| 8 | `DriverEntry` + 启动线程 | 异步启动、逐核绑定亲和性、`StartSVM` |
| 9 | 基础 MSR 拦截 | 拦截 EFER 写（保留 SVME 位） |
| 10 | 电源回调 | Sleep 前 Suspend、唤醒后 Resume |

**里程碑**：系统在 Hypervisor 下稳定运行，功能跟没虚拟化一样。

### 阶段 2：内存隐藏能力

| # | 模块 | 内容 |
| --- | --- | --- |
| 11 | 零页隐藏 | `SetNestedPageProtection(Hide=TRUE)` 映射零页 |
| 12 | PE 段规划 | `code_seg/data_seg` 把 Hypervisor 代码和数据分到专用段，4KB 对齐 |
| 13 | 自身隐藏 | 把 VMCB/NPT/Host 栈/私有数据段/代码段全部 Hide |

**里程碑**：Hypervisor 自身代码和数据对 Guest 不可见。

### 阶段 3：Hook 框架

| # | 模块 | 内容 |
| --- | --- | --- |
| 14 | Hook 管理器 | `HOOK_INFO/HOOK_FUNC` 结构、链表管理、`AddHook`/`RemoveHook` |
| 15 | 影子页管理 | `CreateShadowPage`（分配 + `memcpy` + 隐藏）、`SetGuestShadowPage`（切换 NPT 映射） |
| 16 | x64 指令长度解码器 | Legacy prefix → REX → Opcode → ModRM/SIB → Immediate 长度计算 |
| 17 | 跳板生成 | 汇编跳板模板、`AllocateJmpTrampoline`（分配 + `memcpy` 模板 + 填 3 个地址 + 复制序言） |
| 18 | `#BP` 拦截 | `HandleBP`：匹配 RIP、改写 Guest RIP 到跳板、复位影子页 |
| 19 | NPF Hook 触发 | `HandleNPF` 中 `Id=1` 分支：查找 Hook、切到影子页 1 |
| 20 | `VMMCALL` 处理 | `ResetShadows` 复位影子页 |
| 21 | 复位线程 | 高优先级系统线程，周期 `VMMCALL` 复位影子页 0 |
| 22 | Hook 安装/卸载 API | `InstallHook`/`UninstallHook` 完整流程 |

**里程碑**：可以 Hook 任意内核函数，读原函数看到干净字节，执行流被重定向。

### 阶段 4：反检测与优化（可选，根据需求）

| # | 模块 | 内容 |
| --- | --- | --- |
| 23 | CPUID 伪造 | 拦截 CPUID、清除 Hypervisor 位、伪造 SVM 状态 |
| 24 | EFER/RDMSR 伪造 | 让 Guest 读 EFER 看到 SVME=0 |
| 25 | SVM 指令 `#UD` | CPL split + SVME=0 时 `VMRUN`/`VMSAVE` 等触发 `#UD` |
| 26 | RDTSC 虚拟化 | 拦截 RDTSC，补偿 VMEXIT 时间差 |
| 27 | TLB 优化 | ASID 管理、减少不必要的全局 flush |
| 28 | VMMCALL 认证 | 防伪造的 hypercall 调用（cookie 机制） |

## 十四、调试方法论

开发 Hypervisor 级别的代码，蓝屏是常态。以下是实战中总结的调试策略：

### 14.1 分阶段验证

不要一次写完所有模块再测试。按阶段 1 → 2 → 3 的顺序，每完成一个阶段就验证：

- 阶段 1：能启动 VM 并稳定运行 10 分钟以上，运行各种软件不蓝屏。
- 阶段 2：在 Guest 中尝试读 Hypervisor 地址，确认读到 0。
- 阶段 3：先 Hook 一个自己写的简单调用 `DbgPrintEx` 的函数，确认 Hook 触发和跳板正常，再 Hook 复杂内核函数。

### 14.2 双机内核调试

用两台物理机通过串口/网络/USB 做内核调试（WinDbg）。Hypervisor 的 Bug 几乎不可能在单机上调试——因为一旦蓝屏你就什么都看不到了。

设置方法：

- 被调试机：`bcdedit /debug on`；Set com port。
- 调试机：WinDbg → Attach to kernel → Connect com。

### 14.3 日志策略

VMEXIT 中不能直接 `DbgPrint`（太慢且在高 IRQL）。使用环形缓冲区日志：

- 预分配一块 4KB–16KB 的非分页内存作为 log buffer。
- VMEXIT 中写日志到 buffer（无锁或 per-CPU buffer）。
- 在 PASSIVE_LEVEL 的线程中定期把 buffer 输出到 `DbgPrint`。
- 关键路径用 `DbgPrint` 直接打（启动阶段、Hook 安装阶段，这些不在 VMEXIT 中）。

### 14.4 常见 Bug 排查

| 症状 | 可能原因 |
| --- | --- |
| VMRUN 后立刻蓝屏、三重故障、无响应 | VMCB.StateSaveArea 填错了（尤其是 CR3、段寄存器、EFER.SVME），亦或者是处理特定 VMEXIT 时未更新 Rip |
| 运行几秒后随机蓝屏 | NPT 映射错误（比如漏了某些物理内存区域的映射） |
| Hook 第一次触发正常，第二次蓝屏 | 影子页复位失败；跳板 OriginalCode 指令截断 |
| 多核环境一个核正常，其他核蓝屏 | 每 CPU 结构没独立分配；NPT 页表共享了不该共享的层 |
| 休眠唤醒后蓝屏 | 电源回调没正确 Suspend/Resume |
| Hook 后系统变卡 | VMEXIT 太多（检查拦截位，不要拦截不需要的事件）；复位线程间隔太短 |
| Guest 能读到 Hypervisor 内存 | Hide 后忘了 Flush TLB；某些页没有加入隐藏列表 |

## 十五、写在最后

### 技术的本质

NPT Hook 之所以“无痕”，不是因为它用了什么魔法，而是因为它把 Hook 从“修改代码”这个维度搬到了“控制内存映射”这个维度。在传统模型里，代码和它所在的物理页是绑定的——你改了代码字节，任何人读那个地址都能看到。但在虚拟化模型里，“地址”和“物理内容”的绑定关系是 Hypervisor 可以随时切换的，而且这个切换发生在 MMU 硬件层面，Guest 的任何软件操作都无法绕过。

这种思想——**多加一层抽象，在这层抽象里做手脚**——其实在计算机科学里无处不在。NPT Hook 只是“加一层”思想在安全领域的一个具体应用。

### 性能代价

每次 Hook 触发涉及两次 VMEXIT（NPF + BP），每次 VMEXIT 大约 500–2000 个 CPU 周期。对于不频繁调用的函数（进程创建、文件打开、注册表操作），这个开销完全可以忽略。但对于每秒调用数万次的函数（如 `NtQueryInformationProcess`、某些图形函数），累计开销也可能会很显著，但整体影响可以在能接受的范围内。

### 为什么理解这些很重要

即使你不打算自己写一个 Hypervisor，理解 NPT Hook 的原理也很有价值：

- 安全软件开发者需要知道攻击者可能用什么手段绕过你的检测。
- 逆向工程师需要理解为什么有些代码“读起来和执行起来不一样”。
- 内核开发者需要理解硬件虚拟化提供的能力边界。
- 这种“控制映射关系而非内容”的思想可以启发很多其他场景的设计。

### 延伸阅读

- **AMD Architecture Programmer's Manual Volume 2**：SVM/NPT 的权威参考，所有实现细节都在这里。
- **AMD64 Architecture Programmer's Manual Volume 3**：指令集参考，手写反汇编引擎和跳板时必查。
- **SimpleSvmHook/DdiMon**：开源参考实现，读源码是最好的学习方式。

---

*本文讲解的是技术原理和实现思路。请将所学用于正当的安全研究和系统开发。*

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2026-7-7 12:14

被漫雾编辑，原因：开放可见范围

[#系统内核](https://bbs.kanxue.com/forum-41-1-131.htm)

[#驱动开发](https://bbs.kanxue.com/forum-41-1-132.htm)

[#HOOK/注入](https://bbs.kanxue.com/forum-41-1-133.htm)

[#虚拟化](https://bbs.kanxue.com/forum-41-1-136.htm)
