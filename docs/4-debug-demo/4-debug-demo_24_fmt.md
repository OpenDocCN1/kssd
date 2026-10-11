
# Beginner Tutorial #10 【翻译】

> 原文：[`www.pediy.com/kssd/pediy12/23789.html`](https://www.pediy.com/kssd/pediy12/23789.html)

**Beginner Tutorial #10**

By: `Shub-Nigurrath` / `ARTeam`

`http://cracking.accessroot.com`

`Anti-tampering Techniques Theory v1.0`

**The Target:** `none`

**The Tools:** `Ollydbg` `1.10`

**The Protection:** `none`

**Other Information:** This tutorial will cover some basic concepts about attacking applications in general and anti-tampering techniques. Explicit aim of this tutorial is to help making a clear understanding and classification of concepts seen in the beginners tutorials on this series.

*Best viewed in `Firefox` at `1280x1024`*

**初学者教程 #10**

By: `Shub-Nigurrath` / `ARTeam`

`http://cracking.accessroot.com`

**反篡改技术理论 v1.0**

**目标：** `none`

**工具：** `Ollydbg` `1.10`

**保护方式：** `none`

**其他信息：** 这篇教程将涉及常规的攻击程序与反篡改技术的一些基本概念。帮助大家弄懂和区分本系列教程中提到的各个概念是本篇的明确目标。

## 1. Introduction

This tutorial is meant to cover a topic that is usually just slightly pointed in common tutorials. The argument is the anti-tampering techniques used by programs to counteract the modifications of their code. Told this way, it could seem something you never seen around, but this is the name of techniques such as `CRC` self-checking of an application, `MD5`, `SHA1`, and so on. Did you realize it?

### 1.介绍

本章会涉及一个普通教程很少提到的主题：讨论反篡改技术（即程序用于防止其代码被修改的技术）。讨论过程中，有些技术名词你以前可能还没见过，例如应用程序的 `CRC` 自校验、`MD5`、`SHA1` 等等。你了解它们吗？

`Self-checking` (also called `self-validation` or `integrity checking`) is a technique in which a program, while running, checks itself to verify that it has not been modified. Usually it's useful to distinguish between `static self-checking`, in which the program checks its integrity only once, during startup, and `dynamic self-checking`, in which the program repeatedly verifies its integrity, as it is running. While `self-checking` alone is not sufficient to robustly protect software, is nowadays one of the most easier protections to implement and widely used either in commercial or custom protections.

自校验（也叫自验证或完整性校验）就是程序运行期间校验自身代码以确认没有被修改的技术。一般来说，自校验分为静态自校验和动态自校验。静态自校验仅在程序开始运行时验证代码完整性一次；而动态自校验则是在程序运行期间反复检测。虽然单有自校验还不足以万无一失地保护软件，但它仍是现今最容易实现的保护技术之一，并且已广泛应用到商业或客户保护中。

To correctly discuss this issue I'll resemble some things coming from different sources and some contributions on my own. As usual the distinction between my original work and what is coming from another source is left intentionally hidden.

为了恰当讲述这个问题，我的方法会跟一些从各方面收集的例子及自己拥有的文献相似。与其他资料的差别是，我的文章会有意忽略某些问题。

This tutorial presents the anti-tampering problem and some details of the `CPU` memory management which will help understanding the whole thing. An example in `C` is then built and debugged as a proof of concepts.

`have phun`.

这章教程提出了反篡改问题及 `CPU` 内存管理的一些细节，这些细节会帮助你理解整个过程。作为教学概念的验证，我会以 `C` 语言作例子进行编译和调试。

## 2. Classification of Attacks

"The security of a cypher scheme shouldn't rely on keeping secret the algorithm. The security depends only on keeping the used key secreted"

— `Auguste Kerckhoffs`, `La Cryptographie Militaire`, `Journal des Sciences Militaires`, vol.9, `February 1883`.

Before starting to discuss the argument it is better to introduce a classification of attacks types, which helps keeping ideas clear.

Broadly speaking the attacks to a system (software or not) can be classified into three categories based on the nature of malicious agent.

### 2.攻击的分类

“加密方案的安全性不应该依赖对算法的保密，而只需依靠对密钥的的保密。”

——`Auguste Kerckhoffs`, `La Cryptographie Militaire`, `Journal des Sciences Militaires`, vol.9, `February 1883`.

讨论开始前不如先介绍一下攻击类型的分类，使你保持清晰的概念。

概括来讲，对一个系统（软件或其他）的攻击可基于恶意主体的性质而分为三类。

The first one, the simplest to detect and handle is when the perpetrator breaches communications access controls to attack the system. The malicious agent is still under the restrictions of the communication protocol. A good example of this genre would be a standard hacker type attack. Robust access control mechanism that isolates the system hardware and software totally form the user is enough to counter this attack.

第一类，入侵者以破坏通信访问控制来攻击系统是最容易检测和处理的。恶意主体仍受通信协议的约束。黑客攻击就是这类攻击手法的典型例子。用健壮的访问控制机制使用户与系统软、硬件完全隔离，就足够抵抗这类攻击。

The second, comparatively more serious attack would be like a computer virus. Such attacks originate as software running on the platform. Though the attackers have breached the communications, they must still depend on the `BIOS` and `OS` interfaces. Such attacks are generally the aftermath of a class I attack, and forebode class 3 attacks. Detection of these types of attacks is facilitated by the fact that the perpetrators attack classes of files.

第二类，更加危险的攻击与计算机病毒作用相似。攻击通过运行操作平台上的软件进行。尽管攻击代码已经破坏了通信系统，但他们的活动仍须依靠基本输入输出系统接口（`BIOS`）和操作系统接口（`OS`）。这类攻击通常在第一类攻击之后，也是第三类攻击的预兆。基于入侵者攻击文件的类的事实，检测此类攻击是不难的。

Class 3 attacks are the most difficult to resolve and detect as the perpetrator is an insider. The perpetrator may modify software or hardware for the system at will. The only form of prevention for this type of attacks is to raise the technological bar to such an extent that the perpetrator would end up wasting so much time and resources that it would be a poor investment. The technical bar could be varied from no-specialized-analysis-tools to specialized-hardware-analysis tools.

第三类攻击最难发现与解决，原因入侵者就是内部人员，他可以根据不同系统随意修改软件或硬件。防范此类攻击的唯一方式是提升整个防护技术体系，使得入侵者消耗了大量时间和资源却得不到回报而放弃。从非专用分析工具到专用硬件分析工具，这种防护技术体系应该是多样性的。

Under the heading of software protection, there is another little classification to discuss. The techniques for protecting programs can be distinguished roughly into:

- techniques for protecting software from reverse engineering (by obfuscation)
- modification (by software tamper resistance)
- program-based attacks (by software diversity)
- and `BORE` (Break-Once Run Everywhere) attacks (by architectural design)

在软件保护的范围下，有其他小类去讨论。程序保护技术可初步分为：

- 防止软件被逆向的保护技术（混淆代码）
- 代码修改（软件抗篡改）
- 基于程序的攻击（软件相异性）
- `BORE`（Break-Once Run Everywhere）——泄露一次各处运行的攻击（架构设计）

## 3. Tamper Resistance Techniques and Checksumming

### Introduction.

Software tamper resistance is the art of crafting a program such that it cannot be modified by a potentially malicious attacker without the attack being detected. In some respects, it is similar to fault-tolerant computing, in that potentially dangerous changes in program state are detected at runtime. For tamper resistance, however, it is assumed that intelligent, malicious attackers (rather than hardware flaws or software errors) may be responsible for such changes. Self-checking tamper resistance is distinguished in its ability to run on current unmodified commodity hardware without requiring third parties, because all the tests are performed by the application on its own code/data.

### 3.抗篡改技术与校验和

#### 介绍

软件抗篡改是编写程序的艺术，使代码在没有攻击检测的情况下，不会被潜在的有恶意的攻击代码所修改。在某些方面，它类似于容错处理，在程序运行期间检测存在潜在威胁的改动。由于抗篡改技术，可假定明智和暴力的入侵者将为篡改软件造成的硬件损坏或软件错误负责。自校验防篡改程序的所有应用测试基于内建代码系统，具有在不需要第三方支持的条件下固化于硬件内部的卓越性能。

There are of course different techniques for self-checking software (e.g. program or result checking and generation of the correct executable or part of code on the fly, decryption according to a calculated digest) partially derived from asymmetric signature algorithms, but we'll focus more on checksumming.

当然了，带有自校验的软件也用了不同技术（例如在编程、结果检测、标准执行阶段或者代码片断空闲时，根据合适的处理解码），也有部分技术是非对称签名算法的派生，我们更多关注的是校验和技术。



# Software Tamper Resistance and Anti-Tampering

## Threat Model
The standard threat model for software tamper resistance is the **hostile host model**. The challenge is to protect an application running on a system controlled by a malicious, intelligent user. Because such a user can in theory change any code on the computer, other software on such a system, including the operating system, is untrusted; in the case of particularly determined adversaries, even the hardware is untrusted. This situation is in contrast with the **hostile client** problem in which we assume a trusted host and untrusted application. The hostile client problem appears to be an easier problem to solve; numerous solutions have been developed, e.g. `sandboxing` and is typical of mobile agents such as smartphones or `PDAs`.

一般对于抗软件篡改的威胁模式，来自攻击者主机中。在聪明而有恶意的用户控制的系统中保护一个应用程序是个难题。理论上，这个用户可以改变计算机的任何代码、软件和系统，包括操作系统。如果遇到特别坚决的对手，甚至连硬件也可以改动。与此对比的是，入侵者只控制客户端应用软件而主机不受其操控的情况。只有客户端受控的问题看来较易解决，针对此已经开发了众多方案。例如 `sandboxing` 技术和典型的可移动终端，像智能手机或者掌上电脑。

## Checksum Implementation
Since a single checksum is relatively easy for an attacker to disable, stronger proposals rely on a network of inter-connected checksums, all of which must be disabled to defeat tamper resistance. A tester reads the area of memory occupied by code and readonly data, building up a checksum result based on the data read. A subsequent section of the code may operate on the checksum result, affecting program stability or correctness in a negative way if a checksum result is not the same as a known good value pre-computed at compile time. The sections of code which perform the checksumming operations may be further hidden using code obfuscation techniques to prevent static analysis. Ideally the effects of a bad checksum result in the program are subtle (e.g. causing mysterious failures much later in execution) thus making it much more difficult for an attacker to locate the checksum code.

因为单一的校验和技术比较容易被攻击代码屏蔽，所以有人提出更安全的方案：互联校验和网络，只有令整个网络失效，才能对付这种抗篡改技术。检测码会读取装有代码和只读数据的内存区域，根据读得的数据计算其校验和。下一区域的代码则要靠上面计算的校验和结果运作，如果计算校验和的结果与程序编译时预计算的值不符，就会影响程序的稳定性和正确性。利用代码混淆技术深藏执行校验和计算的代码区，将会防止对程序进行静态分析。理论上，校验和结果有误，并非立即在程序中体现（例如在运行较长时间后才出现奇怪的错误），从而使入侵者对校验和代码的定位更加困难。

`Figure 1` - A good distribution of checksum blocks within a code segment.

`Figure 1` gives a simplified view of a typical distribution of checksum code within an application. In practise, there may be hundreds of checksum blocks hidden within the main application code. Each allows verification of the integrity of a predetermined section of the code segment. The read-only data segment may also be similarly checked.

图 1 ——校验和模块在代码段中的有效分配

图 1 简略示意了在程序中校验和模块的典型分配方式。事实上，也许有几百个校验和模块隐藏在主程序代码中。每个模块都对代码段中的预定区域进行完整性检验。即使是只读数据段也会进行简单检查。

The checksumming code is inserted at compile time and integrated with regular execution code. The application also requires a correct checksum result for each block in order to work properly.

The resulting scheme is an Interlocking Trust Model, for which each segment not only on itself but also on other segments to effectively perform its task. Thus each segment of code would be made responsible for maintaining and verifying the integrity of other segments.

校验和代码在程序编译时就被嵌入，并与常规的可执行代码结合。要程序运行正常则需每个模块的校验和结果正确。

综合方案就是使用连锁校验模式，使每段代码不仅仅存在于自身程序段中，而且还要置于其他程序段来有效执行。因此每个代码段将要负责维持和核对其他代码段的完整性。

## Protection Architecture: Testers and Guards
### High Level View of How the Protection Works
The overall protection works like this: at runtime, a large number of embedded code fragments called `testers` test small segments of code for integrity (using a linear hash function and an expected has value); if integrity fails, an appropriate response is pursued. The peices of code called on check failures are called `guards` and can be programmed to carry out arbitrary tasks (for example common guards are peices of code repairing the code, erasing the breakpoints or downloading a fresh copy of the application).

保护架构的高级运作模式，测试码与防护码

全面的保护架构的运作如下：在程序运行期间，称为测试码（`Testers`）的大量嵌入代码片断会检测小型代码段的完整性（利用线性散列函数及预期值），如果检测不通过，则会作出恰当的响应。调用错误检测的代码叫防护码（`Guards`），它可被编写来执行任何任务，例如常见的防护码可以修复程序代码、清除断点和载入最新的应用程序副本。

### Types of Guards
Two of the most common guards are checksumming and repairing.

#### Checksum Guards
The checksum guards are programmed to check a code fragment within the software. If this code is tampered with in any way the guard takes action. The type of action the guard takes can vary in degree from logging the infraction halting the program.

两种最普遍的防护码：校验和防护码与修复防护码

校验和防护码（`Checksum Guards`）：校验和防护码用于检测软件内部的代码片断。如果通过任何方式篡改软件代码，防护码将会采取行动。它的反应会随着非法操作的程度作出改变，并以此中断程序。

#### Repair Guards
The repair guards can fix a tampered section of code. Thus, the changes an attacker made are wiped out and the program remains secure. The repair can be made by storing a clean copy of the segment of code the guard is protection somewhere, and then replacing the dirty copy with the clean one.

修复防护码（`Repair Guards`）：修复防护码可以修补被篡改的代码段。因此，攻击代码对程序的改动会被清除，程序仍是安全的。一个“干净”的代码段副本会放在某个受保护的区域，修复码将使用这个副本来覆盖遭篡改的代码段。

The security of the guards actually comes from guards watching each other's backs. Each guard protects fragments of code, and that includes protecting other guards. `Figure 1.1` shows an example of a possible network of guards.

防护码技术的安全性在于其环环相扣——每个防护码所保护的代码段，都包含维护其他防护码的代码。图 1.1 就是一个合理的防护码网络例子。

`Figure 1.1` - A possible distribution of cross-checking guards of an application's integrity.

A group of guards working together implement a sophisticated protection scheme that is more resilient against attacks than a single guard. These schemes are used by the most modern protectors, such as `Armadillo` also: the `Nanomities` trick is just a way to implements a Guard scheme and an interlocking scheme, made of two processes, controlling one each other.

图 1.1——防护码交叉检测程序完整性的的一种分配方式

用一组防护码实现复杂的保护方案，其抗攻击的强度比单个防护码更好。一些最新的加壳软件也采用了类似机制，例如 `Armadillo`，它只是用了防护码与连锁校验的把戏，建立了两个相互控制的进程而已。

### Network of Guards
In order to perform their duties, a network of guards is placed into the program and hooked to its execution flows in proper ways, before of the controlled code goes into execution: a repairing guard has to be inserted into a point in the control flow that is to be reached before guarded code (in execution order). In other words the guard has do dominate the guarded code. Guards can also be controlled by other guards too. `Figure 1.2` reports another real more sophisticated example where a network of Guards (`G`) and Guarded Code (`C`) is presented beside it's distribution into the real application's memory.

防护码网络运作在控制码执行程序之前，它会在程序中通过恰当的方式与执行流挂钩：修复防护码会在控制流到达受保护代码前（按执行次序）嵌入。换句话讲，防护码完全支配了受保护代码。当然，防护码也会受其他防护码控制。图 1.2 是另一更复杂的实例，它显示了由防护码（`G`）和受保护代码（`C`）组成的网络，也包括它们在真实程序内存中的分配方式。

`Figure 1.2` - A Guards graph and the corresponding memory layout of the guarded program.

Defeating complex Guard Networks can be, generally speaking, a nightmare.

图 1.2——防护码网络图示及受保护程序在相应内存中的布局

一般来说，破解复杂的防护码网络就如恶梦一般。

## Anti-Tampering Details
There are several aspects of such checksumming which an attacker must keep in mind:

1. Because of the overlapping network of testers, almost every checksumming block must be disabled at the same time in order for a tampering attack to be successful.
2. The resulting value from a checksum block must remain the same as the original value determined during compilation (or all uses of the checksum value must be determined and adjusted accordingly), if the results of a checksum are used during standard program execution.
3. The checksum values are only computed for static (i.e. runtime invariant) sections of the program.
4. Checksumming code is obfuscated, hard to find, and the use of checksum results is also hidden.

抗篡改的一些细节

有关校验和的若干特点，入侵者必须熟记于心:

1. 由于测试码重叠网络的存在，要破解成功必须同时将每个校验和模块处理掉。
2. 如果校验和结果在程序正常运行时起作用，从校验和模块得到的值必须与编译时计算的值相同（或者必须求出校验和值的全部作用，而且其效果会作出相应调整）。
3. 只能对程序的静态（即运行期间不变）的区域计算校验和。
4. 校验和代码常与其他代码混淆，难于发现，其结果的效用也是隐藏的。



注意校验和算法的一个关键（隐含）假设是 `D(x) = I(x)`，其中 `D(x)` 是从内存地址 `x` “读取数据”的位串结果，而 `I(x)` 是从 `x` 处获取相同长度指令的位串结果。在实际的日常情况下，确实如此，因为 `D(x)` 和 `I(x)` 是内存中“作为数据和指令读取”的同一部分。

如果 `I(x)` 与 `D(x)` 不同，那么校验和代码将始终使用 `D(x)` 进行检查，而处理器将始终执行 `I(x)`。

校验和旨在验证处理器执行的代码是否为原始代码，因此假设它读取的代码就是处理器执行的代码。

尽管当前的校验和提案严重依赖此假设，但在一些现代处理器上存在破坏该假设的机会。

## 4. CPU 对虚拟内存的支持

本节为不熟悉现代处理器虚拟内存子系统（包括转换后备缓冲器（TLB））的读者提供背景信息。如果您对处理器架构已经熟悉，可以直接跳到下一节。

现代处理器所做的远不止执行指令序列。处理器速度和灵活性的进步导致了非常复杂的架构。这种复杂性的重要部分来自于旨在高效支持虚拟内存的机制。虚拟内存最早于 20 世纪 50 年代末引入，涉及将主内存划分为帧（页面）数组，随后可以进行操作。应用程序使用的虚拟地址由虚拟内存系统映射到物理地址。

图 2 - 虚拟地址到物理地址的转换

尽管页表转换算法在不同处理器之间可能略有不同，有时也可能通过软件实现，但现代处理器在将虚拟页号转换为物理帧号时都使用大致相同的方法。具体来说，这种转换是通过使用页表完成的，页表是将选定数量的虚拟页号与物理帧号关联起来的数组。

由于大多数进程的虚拟地址空间既大又稀疏，页表条目仅针对实际使用的地址空间部分进行分配。为了确定与给定虚拟地址对应的物理地址，必须定位到合适的页表及该页表中的正确条目。

对于使用 3 层页表的系统，虚拟地址被分为四个字段：`x1` 到 `x4`。`x1` 位（目录偏移量）指定每进程页目录中的一个条目。该条目包含页映射表的地址。`x2` 位（映射偏移量）用作指定页映射表内的偏移量，从而给出页表的地址。`x3` 位（表偏移量）索引到选定的页表中，返回物理页帧的编号。然后，`x4` 指定包含原始虚拟地址所引用数据的物理帧内的偏移量。

图 3 - 通过页表将虚拟地址转换为物理地址

此解析过程在图 3 中展示。请注意，如果使用内存段，段转换通常发生在涉及页表的操作之前。

### TLB 转换后备缓冲器

由于解析每个虚拟内存地址都必须访问多个内存位置，使用页表进行虚拟地址转换是一个昂贵的操作。为了加速这些映射，使用了一种名为转换后备缓冲器（TLB）的专用高速关联内存存储。

TLB 缓存最近使用的虚拟页号到物理页帧的映射。在每次虚拟内存访问时，都会检查 TLB 中的所有条目，以查看其中任何一个是否包含正确的虚拟页号。如果找到该虚拟页号的条目，则发生 TLB 命中，并立即访问对应的物理页帧。否则，发生 TLB 失效，并以之前讨论的方式咨询合适的页表。然后，通过替换最近最少使用的映射，将确定的映射添加到 TLB 中。图 4 说明了 TLB 命中时发生的情况。

图 4 - 使用 TLB 进行地址转换

由于局部性原理，TLB 转换在实际应用中效果很好。然而，系统设计者已经注意到，代码和数据表现出不同的局部性模式。为了防止这些模式之间的干扰，代码和数据的缓存通常被分离；基于类似的原因，大多数现代 CPU 拥有独立的代码 TLB 和数据 TLB。

CPU 缓存根据引用的内存是否被发送到指令解码器，将其标记为代码或数据。每当从内存中获取指令时，指令指针通过指令 TLB 转换为物理地址。当获取或存储数据时，处理器使用独立的数据 TLB 进行转换。为代码和数据使用不同的 TLB 单元，使处理器能够更准确地表示最近使用的内存。独立的 TLB 还防止代码（或数据）的频繁随机访问压倒整个 TLB 系统。由于大多数代码和数据引用表现出高程度的局部性，少量快速存储（例如片上内存缓存）与大量较慢存储（DRAM 内存）的组合可以共同近似更大数量快速存储的性能。

页面交换



# 内存管理与访问控制

由于内存管理单元为运行中的应用提供了虚拟地址空间，因此应用程序无需感知其实际使用的物理内存段。尽管程序的虚拟地址空间是连续的，但其所占用的物理内存区域不一定连续。这为操作系统提供了巨大的机会：不仅允许多个应用程序同时运行（每个应用程序拥有唯一的虚拟地址空间，映射到不同的物理页面），还允许操作系统仅将当前所需的部分保留在物理内存中。

## 页面置换

由于并非所有虚拟内存页都映射到物理页，处理器必须有一种方式在虚拟地址没有物理映射时通知操作系统。处理器通过缺页中断来完成此操作。它将导致缺页中断的虚拟地址存入寄存器，然后通过中断处理程序通知操作系统。操作系统随即更新虚拟地址到物理地址的映射，以便请求的虚拟地址能映射到物理地址。

这可能意味着从磁盘或其他外部存储将程序的某部分调入物理内存。然后，操作系统通过从中断返回来通知处理器重试该指令。如果操作系统确定虚拟地址无效（例如，虚拟地址引用了未分配的内存），它也可以选择终止应用程序的执行。

## 内存访问控制

在将虚拟地址转换为物理地址的同时，处理器可能会在内存区域上实施访问保护。由于虚拟内存子系统已经将物理内存划分为小的区域（页帧），由内存管理单元按页帧粒度实施访问控制是合理的。

最重要的保护机制是：只有允许应用程序访问的页面才会被映射到其页表中。为了防止应用程序手动将页面映射到其地址空间，页目录基指针存储在只读寄存器中，且包含进程页表的页帧本身对该进程不可访问。

此外，对于进程地址空间中的页面也有保护机制。每个映射的页都受到其内容可执行操作类型的限制：读、写以及取指令（也称为执行）。允许的操作通过与各页表项关联的控制位来指定。读和写通常是数据页的常见操作，而执行代码通常与包含可执行代码的页面相关联。

现代操作系统利用处理器实现的保护机制来区分不同类型的内存使用。

在每页基础上设置无执行权限（no-execute），会限制许多程序仅能从其代码段执行代码，除非它们采取特定措施使其数据可执行。虽然这种改变可能会干扰在运行时生成机器码的系统（例如现代 Java 虚拟机），但不可执行数据页能够防御多种代码注入攻击。

**表 1 - 不同页面类型的访问控制权限分离**

表 1 显示了应用程序不同区段的理想权限分配。当前可执行文件格式假定使用这种权限分配。所有实现页面级访问控制的处理器都必须检查不允许的操作，并相应地通知操作系统。通常，通过缺页中断通知操作系统，该中断会指出导致无效操作的内存引用。

# 5. x86 架构的内存模型：段

除了支持内存页，`x86` 还可以管理可变大小的内存块，称为段（segments）。

与每个段相关联的信息包括基地址、大小、权限和其他元数据。这些信息共同构成段描述符。要使用给定的段描述符，其值需加载到某个段寄存器中。除了段描述符编号外，这些寄存器的内容对所有软件都是不可访问的。为了更新段寄存器，对应的段描述符必须在内核内存中修改，然后重新加载到段寄存器中。

逻辑地址由段寄存器说明符和偏移量组成。为了得出线性地址，需将段寄存器的段基（由段说明符命名）加到段偏移量上。`x86` 架构的完整转换机制如图 6 所示。代码读取始终相对于代码段（`CS`）寄存器，而通常，如果没有指定段寄存器，数据读取使用数据段（`DS`）寄存器。通过段超越（segment overrides），数据读取可以使用包括 `CS` 在内的任何段寄存器。获得线性地址后，正常进行页表转换，如图 6 和图 7 所示。

**图 6 - `x86` 中从虚拟地址到物理地址的转换**

**图 7 - 使用段超越的转换**

与 `x86` 中的页不同，段可以设置为仅允许指令读取（只执行）。在只执行段中进行数据读取和写入将产生异常。这种只执行权限可用于检测应用程序何时试图相对于 `CS` 读取内存。一旦异常被投递给被攻击修改的操作系统，该操作系统可以自动修改内存映射（见图 7），使其看起来未修改的数据存在于该内存页中。

然而，大多数用于 `x86` 的操作系统现在都实现了平坦内存模型（flat memory model）。这意味着 `CS` 和 `DS` 寄存器的基值相等；应用程序无需使用 `CS` 寄存器来读取其代码。平坦内存模型将确保两个线性地址相同，从而得出相同的物理地址（见图 7 中的点划线）。



大部分的 x86 操作系统，已实现内存平面内存模式（flat memory model）——意谓 CS 与 DS 寄存器的基值是相等的；也就是说程序不必用 CS 寄存器读取代码。 平面内存模式会确保两者线性地址相同，产生相同的物理地址（参考图 7 的虚线部分）。  

### 6. Writing and debugging a selfchecking program in C  

Resuming what we said till now it can be stated that: "detecting whether portions of a binary have been modified is essentially an error-detection problem; therefore, a checksum algorithm such as CRC32, MD5, or SHA1 can be used to generate a signature for an arbitrary block of code or data. This signature can then be checked at runtime to determine whether any modification has taken place."  

### 6. 用 C 语言编写调试自校验程序  

继续我们的话题，它可以表述为：“检测机器码是否被修改本质上属于错误检测问题；为此，校验和算法（如 `CRC32`、`MD5` 或 `SHA1`）可对任意代码块或数据块生成标记。随后检查这个标记以确定程序是否在运行期间被修改过。”  

`I` `have` `chosen` `the` `CRC32` `algorithm` `both` `for` `its` `ease` `of` `implementation` `and` `for` `its` `speed.` `It` `is` `ideal` `for` `detecting` `changes` `to` `short` `sequences` `of` `bytes;` `however,` `because` `there` `are` `only` `2` `^` `32` `possible` `checksum` `values,` `and` `because` `it` `is` `not` `cryptographically` `secure,` `the` `likelihood` `of` `a` `collision` `is` `high,` `giving` `the` `attacker` `a` `realistic` `chance` `to` `replace` `code` `without` `changing` `the` `checksum.`  

我选择 `CRC32` 算法做例子是由于它易于实现且执行速度快。用它检测字节短序列比较理想；但由于只有 2 的 32 次方个可能的校验和值，而且不具密码安全性——碰撞可能性高，所以给了入侵者一个可实现的机会，不改变校验和而替换代码。  

### Implementation of a checksum API in C  

Here, first of all, I included a classical implementation of a `CRC32` API, which I found to be very compact and useful. This is an implementation of `CRC32`, which consists of macros for marking the start and end of the block to be checked, as well as a function to calculate the checksum of the block. The function `crc32_calc()` is used to compute the checksum of a buffer.  

### 用 C 语言实现校验和 API  

首先，这里包括了 `CRC32` API 的经典实现方法，我认为非常简洁有效。为实现 `CRC32`，我使用了宏——用于标记块的头尾以便检测，还有计算块的校验和的函数。`crc32_calc()` 函数就用于计算缓冲区的校验和。  

```c
#define CRC_START_BLOCK(label) void label(void) { }  

#define CRC_END_BLOCK(label) void _##label(void) { }  

#define CRC_BLOCK_LEN(label) (int)_##label - (int)label  

#define CRC_BLOCK_ADDR(label) (unsigned char *)label  

static unsigned long crc32_table[256] = {0};  

#define CRC_TABLE_LEN 256  

#define CRC_POLY 0xEDB88320L  

static int crc32(unsigned long a, unsigned long b) {  

    int idx, prev;  

    prev = (a >> 8) & 0x00FFFFFF;  

    idx = (a ^ b) & 0xFF;  

    return (prev ^ crc32_table[idx] ^ 0xFFFFFFFF);  

}  

static unsigned long crc32_table_init(void) {  

    int i, j;  

    unsigned long crc;  

    for (i = 0; i < CRC_TABLE_LEN; i++) {  

        crc = i;  

        for (j = 8; j > 0; j--) {  

            if (crc & 1) crc = (crc >> 1) ^ CRC_POLY;  

            else crc >>= 1;  

        }  

        crc32_table[i] = crc;  

    }  

    return 1;  

}  

unsigned long crc32_calc(unsigned char *buf, int buf_len) {  

    int x;  

    unsigned long crc = 0xFFFFFFFF;  

    if (!crc32_table[0]) crc32_table_init();  

    for (x = 0; x < buf_len; x++) crc = crc32(crc, buf[x]);  

    return crc;  

}
```

The following program demonstrates the use of the checksum implementation. Note that the program is first compiled with a `printf()` in `main()` that will print the checksum to `stdout`. As long as `main()` is linked into the program after the buffer being checked, this `printf()` can be removed and the program recompiled without the value of the checksum changing. Once the checksum is known, a hex editor can be used to patch the checksum value into the location `crc32_stored`. In this example, the four bytes of the checksum are stored between two `0xBAADF00D` markers that should be overwritten with random bytes before the binary is distributed. Note that the markers will be stored in little-endian order in the binary, hence the reversed ordering of the bytes in the C source.  

以下程序演示了校验和的执行。注意程序首先编译了 `main()` 函数中的 `printf()`，显示校验和到标准输出设备。检查缓冲区后，只要 `main()` 函数链接到程序，这个 `printf()` 函数会被移除，而且程序会在没有校验和变化值的情况下重新编译。一旦知道校验和，就使用十六进制编辑器将校验和值插入到 `crc32_stored` 的位置。在这个例子中，四个字节的校验和分别存储在两个 `0xBAADF00D` 标志符之间，二进制码分配前这里会随机写满数据。注意标志符的二进制码是按从小到大的顺序存储，因此在 C 源程序中次序相反。  

```c
#include <stdio.h>  

/* warning: replace "crc32_stored" with the real checksum! 警告：用校验和真值替换 crc32_stored */  

asm(".long 0x0DF0ADBA \n" /* look for 0xBAADF00D markers 寻找 0xBAADF00D 标志符 */  

    "crc32_stored: \n"  

    ".long 0xFFFFFFFF \n" /* change this in the binary! 用二进制码更改这里 */  

    ".long 0x0DF0ADBA \n" /* end marker 结束标志符 */  

);  

CRC_START_BLOCK(test)  

int test_routine(int a) {  

    while (a < 12) a = (a - (a * 3)) + 1;  

    return a;  

}  

CRC_END_BLOCK(test)  

int main(int argc, char *argv[]) {  

    unsigned long crc;  

    crc = crc32_calc(CRC_BLOCK_ADDR(test), CRC_BLOCK_LEN(test));  

#ifdef TEST_BUILD  

    /* This printf() displays the CRC value that needs to be stored in the program.  
     
    * The printf() must be removed, and the program recompiled before distribution.  
     
    * 此 printf() 函数程序需要存储的 CRC 值。  
     
    * 此 printf() 函数必需移除，分配前程序会重新编译。 */  

    printf("CRC is %08X\n", crc);  

#else  

    if (crc != crc32_stored) {  

        printf("CRC32 %#08X does not match %#08X\n", crc, crc32_stored);  

        return 1;  

    }  

    printf("CRC32 %#08X is OK\n", crc);  

#endif  

    return 0;  

}
```

You should compile this program with `TEST_BUILD` defined, then execute it to obtain the CRC value that needs to be replaced for `crc32_stored` in the binary. Then rebuild the program with `TEST_BUILD` undefined, and modify the binary with the proper CRC value from the first run. To calculate the CRC32 of a program you can use the `CRC_calculator` given with this tutorial, just drag & drop the executable over to get the CRC value.  

你应定义 `TEST_BUILD` 来编译这个程序，然后执行并得到 CRC 的值，此值需要替代 `crc32_stored` 内的二进制码。再用未定义的 `TEST_BUILD` 重建程序，把二进制码修改成第一次运行所得到的正确 CRC 值。为计算程序 CRC32 值，你可以使用本教程附带的 `CRC_calculator` 软件，只需拖放可执行文件到其界面上即可得到 CRC 值。  

It is tempting to generate a checksum of the entire program and use this to determine whether any bytes have been changed; however, this causes a loss of granularity for the checksum and can degrade performance. Instead, as explained in previous sections is always better to generate multiple checksums for vital sections of code. These checksums can be encrypted, and they can even be supplemented with checksums of trivial blocks of code to disguise which portions of code are significant.  

对整个程序生成校验和并以此确定是否有字节改动过，无疑非常吸引人；然而这会引起程序颗粒度（granularity ，即数据对象的相对大小的表达式，译者注）的损失，并使其性能下降。作为替代，如之前的所说那样，在重要代码段生成多重校验和的方法会好得多。加密这些校验和值，甚至可以补充一些无关紧要的代码块校验和来伪装成重要的代码。  

### Notes about compiling the example1 with VisualC++ 6.0  

The code relative to this example is included into the `example1` folder. If you open it you might notice some differencies with what I just written above. These differencies are because of not complete compliance to standard C of Visual C++ 6.0. With respect to the code explained above, there are some differencies, just because the Visual C++ 6.0 I used doesn't support the standard `asm` C operator. I had to use instead the Microsoft specific `__asm` operator which has some clearly documented limitations. First of all is not possible to directly define variables using the `DB` directive. I have to use instead the pseudoinstruction `_emit` which force the compiler to emit the specified bytes, simulating so the `DB` directive. The `ASM` part is also modified to copy the address of the `crc` value to a local variable usable by the C code.  

To make things shorter I'll not include this code here too, look the file for further comments and clarifications.  

### 关于用 VisualC++ 6.0 编译例 1（example1）的注意事项  

`example1` 文件夹中有这个例子的源代码。如果你打开它，你也许会注意到它跟上面我所写的有些不同。这是由于 Visual C++ 6.0 与标准 C 并不完全一致。两者的一些差别仅仅因为 Visual C++ 6.0 不支持标准 C 的操作符 `asm`。我只能用 `__asm` 关键字来代替 ，关于 `__asm` 的一些限制，微软文档有清楚的说明。首先，不能用 `DB` 伪指令直接定义变量。我改用伪指令 `_emit` 强制编译器放出指定的字节来模拟 `DB`。经修改 `ASM` 部分也使 C 代码可用，其作用是复制 `CRC` 值的地址到一个局部变量。  

限于篇幅，这里就不引用代码了。请直接查看该文件以获得更多注释和说明。  

要编译使用 `"cl crc1.c /DTEST_BUILD"` 或 `"cl crc1.c"`  



### 1. Running the Program

Note that the programs are console based, they will just close and exit when double clicking them. You need to run them from within a DOS box, to see the printed messages.

编译使用`cl crc1.c /DTEST_BUILD`或`cl crc1.c`

注意程序是基于控制台的，双击它就会关闭退出。你须在DOS窗口运行它以看到显示的信息。

### 2. Observing Example 1 with OllyDbg

#### 2.1 English Explanation

Open with`ollydbg`the`crc1.exe`file inside`example1`folder of the present tutorial's archive. Hit`CTRL-B`to open the Binary Search dialog and search for`0xBAADF00D`hex value. You should land here:

I properly placed the comments to help you find the correspondence with the written code (refers to the modified code used for the`example1`and not to the code above).

Place a Breakpoint with`F2`at the`JMP`at`0x0040114C`and press`F9`to reach the breakpoint. First of all if you step with`F8`the program at`0x00401180`you can see that the real CRC just-in-time calculated is`0x6C8673E6`(it's in`ECX`).

Well you have the opportunity then to run again the application from the beginning and to stop again the application at`0x0040114C`. Now go to location`0x401152`and select"Follow in dump"option to let you view in`Ollydbg`the memory at that location.

Now insert the real CRC value you've just seen before(`0x6C8673E6`), remember to insert it as`0xE673866C`.

Now save the modifications to a new executable and run from outside`Ollydbg`, the application runs perfectly. I saved this second executable into`crc1_ok.exe`

Weel, obviously the markers are not mandatory and are useful only to easily find the proper point where to insert the real CRC value, the whole routine can be more efficiently hidden and the check can be made less evident.

#### 2.2 Chinese Explanation

用`OllyDbg`观察例1

用`OllyDbg`打开本教程附带`example1`文件夹中的`crc1.exe`。按`CTRL-B`打开二进制字串查找框，查找`0xBAADF00D`的HEX值。你会来到这里：

我加了必要的注释帮助你找到程序代码的对应位置（引用的是`example1`修改后的程序代码，并非上文的代码）。

按`F2`在`0x0040114C`的`JMP`处下断，按`F9`来到断点。首先用`F8`单步步进，在`0x00401180`处可以看到即时计算的CRC真值是`0x6C8673E6`（看`ECX`处）。

好了，你有机会重头开始运行程序，然后再次停在`0x0040114C`处。来到`0x401152`处并右键选择"Follow in dump"（数据窗口中跟随，译者注），在`Ollydbg`中会看到此处的内存内容。

在此插入之前看过的CRC真值（`0x6C8673E6`），记得插入的是`0xE673866C`。（注意堆栈后进先出的性质，低字节优先。译者注）

现在保存修改到新的EXE文件，然后脱离`Ollydbg`运行。程序运行正常。我把这个新的EXE文件保存为`crc1_ok.exe`。

当然了，例1（`example1`）里的标志符并非必须，它只是让你更容易找到合适的位置去插入CRC真值。整个例程可以隐藏得更有效，而且检测也可能更隐蔽。

### 3. Analyzing a More Complex Example

#### 3.1 Example 2 Analysis

Generally speaking to fool the protection you only have to change the`JE`jump (opcode`0x74`) at offset`0x401183`to a`JMP`instruction (opcode`0xEB`). The generated checksum should never be checked against a valid one; instead, the generated checksum should be used as a source of information that the program requires to execute properly. A byte within the checksum could be used as an index into a jump table, for example, or the checksum itself could be used as a key to decrypt critical code or data.

分析更为复杂得例子——例2（`example2`）

一般来说，要破解这个保护你只能用爆破——把偏移`0x401183`处的`JE`（操作码`0x74`）跳转改为`JMP`（操作码`0xEB`）。程序生成的校验和从不与合法值比较，相反，它被作为一个必要的信息源使程序正常运行。例如，一个不带校验和的字节会作为索引进入跳转表，而校验和自身则是加密关键代码或数据的密钥。

#### 3.2 Function Pointer Table Example

The next program demonstrates how to use a table of function pointers to test the value of a checksum. Each nibble or half-byte in the checksum is used as an index into a 16-entry table of function pointers; only the correct table entry calls the function to check the next nibble. This method requires 8 tables of 16 function pointers so that one table is used for each nibble in the checksum.

下面的程序演示了怎样利用函数指针表来测试校验和值。校验和中每个半字节（nibble）会作为索引进入一张16项的函数指针表，只有正确的表项才会调用函数检查下一个半字节。这个办法需要8张每张含16个函数指针的表，使每张表对应一个校验和半字节。

```c
#include <stdio.h>

CRC_START_BLOCK(test)

int test_routine(int a) {
  while (a < 12) a = (a - (a * 3)) + 1;
  return a;
}

CRC_END_BLOCK(test)

typedef void (*crc_check_fn)(unsigned long *);

static void crc_good (unsigned long *crc);
static void crc_check(unsigned long *crc);
static void crc_nib2 (unsigned long *crc);
static void crc_nib3 (unsigned long *crc);
static void crc_nib4 (unsigned long *crc);
static void crc_nib5 (unsigned long *crc);
static void crc_nib6 (unsigned long *crc);
static void crc_nib7 (unsigned long *crc);
static void crc_nib8 (unsigned long *crc);

crc_check_fn b1[16] = {0,}, b2[16] = {0,}, b3[16] = {0,}, b4[16] = {0,},
                  b5[16] = {0,}, b6[16] = {0,}, b7[16] = {0,}, b8[16] = {0,};

#define CRC_TABLE_LOOKUP(table)           \
            int         index = *crc & 0x0F; \
            crc_check_fn next = table[index]; \
            *crc >>= 4;                   \
            (*next)(crc)

static void crc_check(unsigned long *crc) { CRC_TABLE_LOOKUP(b1); }
static void crc_nib2 (unsigned long *crc) { CRC_TABLE_LOOKUP(b2); }
static void crc_nib3 (unsigned long *crc) { CRC_TABLE_LOOKUP(b3); }
static void crc_nib4 (unsigned long *crc) { CRC_TABLE_LOOKUP(b4); }
static void crc_nib5 (unsigned long *crc) { CRC_TABLE_LOOKUP(b5); }
static void crc_nib6 (unsigned long *crc) { CRC_TABLE_LOOKUP(b6); }
static void crc_nib7 (unsigned long *crc) { CRC_TABLE_LOOKUP(b7); }
static void crc_nib8 (unsigned long *crc) { CRC_TABLE_LOOKUP(b8); }

static void crc_good(unsigned long *crc) {
  printf("CRC is valid.\n");
}

int main(int argc, char *argv[  ]) {
  unsigned long crc;
  crc = crc32_calc(CRC_BLOCK_ADDR(test), CRC_BLOCK_LEN(test));

#ifdef TEST_BUILD
  printf("CRC32 %#08X\n", crc);
#else
  crc_check(&crc);
#endif
  return 0;
}
```

When this program is compiled with`TEST_BUILD`defined, the resulting binary will print the`CRC32`computed for the function`test_routine()`. If the computed`CRC32`is`0xFFF7FB7C`, the following table indices will represent valid function pointers:`b1[12]`, `b2[7]`, `b3[11]`, `b4[15]`, `b5[7]`, `b6[15]`, `b7[15]`, `b8[15]`. Each of these contains a pointer to the function that will process the next nibble in the checksum, except for`b8[15]`, which contains a pointer to the function that is called when the checksum has proven valid. The tables in the source can now be rewritten to reflect these correct values:

当程序与`TEST_BUILD`定义一起编译，结果代码会打印计算后的`CRC32`的值，此值随后被`test_routine()`函数使用。假如`CRC32`的计算值是`0xFFF7FB7C`，从下面的表单索引会看到有效的函数指针：`b1[12]`, `b2[7]`, `b3[11]`, `b4[15]`, `b5[7]`, `b6[15]`, `b7[15]`, `b8[15]`。除了`b8[15]`外，其余每个指针都指向处理校验和内下一个半字节的函数。当证实校验和有效，`b8[15]`就指向程序真正调用的函数。

```c
crc_check_fn b1[16] = { 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, crc_nib2, 0, 0, 0 }, /*C*/
                  b2[16] = { 0, 0, 0, 0, 0, 0, 0, crc_nib3, 0, 0, 0, 0, 0, 0, 0, 0 }, /*7*/
                  b3[16] = { 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, crc_nib4, 0, 0, 0, 0 }, /*B*/
                  b4[16] = { 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, crc_nib5 }, /*F*/
                  b5[16] = { 0, 0, 0, 0, 0, 0, 0, crc_nib6, 0, 0, 0, 0, 0, 0, 0, 0 }, /*7*/
                  b6[16] = { 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, crc_nib7 }, /*F*/
                  b7[16] = { 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, crc_nib8 }, /*F*/
                  b8[16] = { 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, crc_good }; /*F*/
```

Obviously, the NULL bytes will have to be replaced with other values to disguise the fact that they are invalid entries. They can be replaced with pointers to functions that handle incorrect checksums, or they can be filled with garbage values to make the program unstable. For example:

很明显，NULL字节须替换为其他值以掩饰它们是无效项。它们可以替换为指针，指向生成错误校验和的函数；也可以填入垃圾代码使程序不稳定。例如；

```c
crc_check_fn b8[16] = { crc_good - 64, crc_good - 60, crc_good - 56, crc_good - 52,
                        crc_good - 48, crc_good - 44, crc_good - 40, crc_good - 36,
                        crc_good - 32, crc_good - 28, crc_good - 24, crc_good - 20,
                        crc_good - 16, crc_good - 12, crc_good - 8,  crc_good - 4,
                        crc_good };
```



在这张表中，递增的数值使该表看起来像是有效数据，这与代码段中的地址相对。

Notes about compiling the `example2` with `VisualC++` 6.0

This time the `example` doesn't have any specific modification to be compiled on `VC++` 6.0. Follow the same commands as before to compile it.

Into the `example2` folder there are two sources, the first `crc2_step1.c` must be compiled using the `TEST_BUILD` define, because it has an uninitialized `crc_check_fn` matrix. The `crc2_step2.c` instead has this matrix correctly initialized.

**Notes about compiling the `example2` with `VisualC++` 6.0**

这次的例子不需任何修改即可用 `VC++` 6.0 编译。编译方式如常。

在 `example2` 文件夹中有两份源程序，`crc2_step1.c` 必须定义 `TEST_BUILD` 来编译，因为 `crc_check_fn` 矩阵没有初始化。而 `crc2_step2.c` 中这个矩阵已正确初始化。

Looking at the `example2` in `OllyDbg`

If you compile the program `crc2_step1.c` using `TEST_BUILD` you should get this value `0x6C8673E6`. According to what I just said before, the `crc_check_fn` matrix should become:

**用 `OllyDbg` 观察例 2 (`example2`)**

如果使用 `TEST_BUILD` 编译 `crc2_step1.c`，你会得到 `0x6C8673E6` 这个值。据前文所述，`crc_check_fn` 矩阵应变成：

```c
crc_check_fn b1[16] = { 0, 0, 0, 0, 0, 0, crc_nib2, 0, 0, 0, 0, 0, 0, 0, 0, 0 }, /*6*/
                  b2[16] = { 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, crc_nib3, 0 }, /*E*/
                  b3[16] = { 0, 0, 0, crc_nib4, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0 }, /*3*/
                  b4[16] = { 0, 0, 0, 0, 0, 0, 0, crc_nib5, 0, 0, 0, 0, 0, 0, 0, 0 }, /*7*/
                  b5[16] = { 0, 0, 0, 0, 0, 0, crc_nib6, 0, 0, 0, 0, 0, 0, 0, 0, 0 }, /*6*/
                  b6[16] = { 0, 0, 0, 0, 0, 0, 0, 0, crc_nib7, 0, 0, 0, 0, 0, 0, 0 }, /*8*/
                  b7[16] = { 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, crc_nib8, 0, 0, 0 }, /*C*/
                  b8[16] = { 0, 0, 0, 0, 0, 0, crc_good, 0, 0, 0, 0, 0, 0, 0, 0, 0 }; /*6*/
```

If you try to open the `crc2_step2` in `Olly` you can see that it is not a so trivial task to modify the code and still obtain a valid running program. Looking for the references to the string `"CRC is valid"` you should land at `0x004012C4`. If you place a breakpoint with `F2` and press run to the program with `F9` you land to the correct place. Now if you press `CTRL-K` to see the call stack you should easily see that this subroutine is called from the

如果尝试用 `OllyDbg` 打开 `crc2_step2`，你会发觉修改代码并保持程序运行稳定并不是件很容易的事情。寻找字串参考 `"CRC is valid"`，你会来到 `0x004012C4` 处。若 `F2` 下断后 `F9` 重新运行程序，你会到达正确地址。现在按 `CTRL-K` 查看调用堆栈，你应该很容易看到此子程序的调用来自

```asm
004012BA |.  FF55 F8   CALL [LOCAL.2]    ; crc2_ste.004012C4
```

This local variable is a pointer to the selected address into the function pointer addresses contained into the `b8[16]` vector.

此局部变量是个指针，指向函数指针地址中选中的地址，`b8[16]` 向量包含了这个函数指针地址。

Well, indeed there's still space for a more intelligent reversing of this application. We will see now how crypto algorithms can be identified using the constants they use. `CRC32` is a very easy algorithm but also all the other more complex are not so complex..we'll see it in the next, last, section! Have a little more patient, we arriving at the end ^_\_

事实上，这个程序仍有更具技巧的逆向空间。现在我们要知道怎样利用加密算法使用的常量来鉴别加密算法。`CRC32` 固然是非常简单的算法，但其他更复杂的算法也不见得会很难。我们将在下一节，也是本文最后一节看到它。请耐心一点，我们来到结尾了。

7. Identifying self-checking algorithms into programs

Breaking the `example1` program

To introduce you to the problem let approach to the program `crc1_ok.exe` inside the `example1` folder in the classical way. First of all be sure to have `PEiD` with the `KryptoAnalizer` (aka `KANAL`) installed. Run `PEiD` and select the `crc1_ok.exe` program, then select plugins and run the plugin just mentioned.

You should obtain (I assume you are already expert on `PEiD`, otherwise return to Beginner tut #1) something like below:

**7. 鉴别程序的自校验算法**

**Breaking the `example1` program**

以 `example1` 文件夹中的 `crc1_ok.exe` 来说明这个问题是典型的做法。首先确认安装了 `PEiD` 及其插件 `KryptoAnalizer`（aka `KANAL`）。运行 `PEiD` 并选中 `crc1_ok.exe`，然后选择 plugins 项并运行刚才提到的插件。

你应该看到如下的画面（这里假设你已经很熟悉 `PEiD`，否则请回到初学者教程#1）：

The plugin reports clearly an address at `0x004010DF`. Well, fire up `Olly` and go there to see what's happening there..

插件清楚显示了一个地址：`0x004010DF`。打开 `OllyDbg` 跟进去看看究竟是怎么回事。

You landed here, to get the meaning of the place where you are move a little the cursors, to realign to the actual real code (you landed in the middle of an instruction, why it will be clear in a moment). Your `Olly` then should show something like below:

你来到了这里。为了获得选中位置的含义，你只需稍稍移动光标，即可重新整理成实际代码（你在指令中间着陆,何以它马上变得清晰呢）。你的 `OllyDbg` 应该显示如下：

Well, we were at the address of a constant, `0xEDB88320` which was part of an instruction `XOR ECX,EDB88320`

好，这是常量的地址，`0xEDB88320` 就是 `XOR ECX,EDB88320` 指令的一部分。

Now look the `CRC32` implementation I did in Section 6, you can find this line:

现在看看我在第六部分所讲的 `CRC32` 的实现，会找到这行：

```c
#define CRC_POLY 0xEDB88320L
```

This is the same constant `KANAL` plugin found. If you have the opportunity/time to see the sources of `KANAL` (freely available on `PEiD` site) you will realize that `KANAL` is doing nothing more than searching for specific constants in memory, any crypto algorithm (better hashing algorithm) uses some specific constants somewhere in the algorithm.

`KANAL` 插件也找到这个同样的常量。如果你有机会/时间去看看 `KANAL` 的源代码（可在 `PEiD` 的网站上免费获取），就会了解 `KANAL` 只是搜索了内存中的特殊常量。任何加密算法（包括比散列算法更好的）都会在算法的某个位置使用了特殊的常量。

At this stage we then understood we landed in the middle of `CRC32` algorithm implementation. What we should do is to go at the entrypoint of the `CRC32` routine that is, in the `example1` sources, the `crc32_calc()` function.

这个阶段，我们懂得自己来到了实现 `CRC32` 算法的代码段中。我们要做的就是进入 `CRC32` 程序的入口点——例 1 源代码的 `crc32_calc()` 函数。

The function where we landed starts here:

我们着陆的函数在此处开始：

```asm
0040108F /$ 55 PUSH EBP
```

and is called here:

并且在此处被调用：

```asm
00401016 |. E8 74000000 CALL crc1_ok.0040108F
```

which is inside a function which is called here:

事实上它存在于函数内部，而函数在此处被调用：

```asm
00401172 |. E8 89FEFFFF CALL crc1_ok.00401000 ; \\crc1_ok.00401000
```

Now you arrived at destination. Look the code where you are now (at `0x00401172`) and compare it with the code shown before in the debugging section of the `example1`, it's the same place.

你现在来到了目的地。看看你所在位置（`0x00401172`）的代码，再对比一下调试例 1 那节所显示的代码，它们都在同样位置。

Breaking the `example2` program

We are now using the method learnt for the easyer `example1` with the more complex `example2`. Run `KANAL` on the file `crc2_step2.exe` into the `example2` folder. You should obtain something like this:

**Breaking the `example2` program**

现在我们用破解例 1 学到的方法来对付更复杂的例 2。将 `example2` 文件夹里的 `crc2_step2.exe` 拖放到 `KANAL`。你会看到如下画面：

As done before go to the address shown by `KANAL`, go up to the entry point of the `CRC32` function and you finally land here:

如之前所做一样，追入 `KANAL` 显示的地址，来到 `CRC32` 函数的入口点。最后在此处着陆：

```asm
004012EA |. E8 11FDFFFF CALL crc2_ste.00401000 ; \\crc2_ste.00401000
```

Well, place a breakpoint at this specific address and run the program using `F9`. Execute the call at `0x004012EA` with `F8`, and you should have in `EAX` the real `CRC32` value just calculated.

好，在这个特殊的地址下断，然后按 `F9` 运行程序。按 `F8` 进入 `0x004012EA` 处的调用，然后你会从 `EAX` 得到刚才计算的 `CRC32` 真值。

Well, here's how the program is compiled and I'll leave to you the opportunity to understand how to fix it when a change is done.

下图展示了程序是如何被编译的。我现在给你一个机会，使你懂得如何在改动完成时去修复。

This is the routine into which the `CRC32` calculation is performed. I commented it with reference to the sources we wrote in Section 6.

这是执行 `CRC32` 运算的例程。注释以第六部分的源代码作为参考。

The `crc_check()` function is called at:

此处调用 `crc_check()` 函数：

```asm
004012F9 |. E8 09000000 CALL crc2_ste.00401307 ; \\call crc_check(&crc)
```

and it's body is the following one:

主体部分如下：

The function elaborates a bit the `CRC32` value computed and uses it to call the next correct function here:

函数巧妙地用 `CRC32` 的值计算了一个位（bit），并以它调用下一正确的函数：

```asm
00401336 |. FF55 F8 CALL [LOCAL.2] ; call the function returned at address b1[6], where 6 is the value found at 00401312
```

00401336 |. FF55 F8 CALL [LOCAL.2] ; 调用函数返回到 `b1[6]` 的位置, 数字 6 是在 `00401312` 处得到的。




如果你按 `F8` 跟踪这个调用，你会看到它调用了另一个几乎相同的函数。这些函数负责从 `b1[]`；`b8[]` 向量中提取正确的函数指针，最终引导你到达程序的末尾。

如果用 `F8` 跟入这个调用，你会看到它调用了另一个几乎相同的函数。这些函数负责取出 `b1[]` 的指针。`b8[]` 向量是正确的函数指针，它会引导你到程序的最后。

例如，调用正确 `CRC32` 的顺序是：

```
00401307 -> 00401135 -> 0040116E -> 004011A7 -> 004011E0 -> 00401219 -> 00401252 -> 0040128B -> 004012C4
```

想象一下，将上述例子与如 `AsProtect` 或 `Armadillo` 这样的混淆器或压缩器结合，并使用比 `CRC32` 更复杂的哈希算法，你就能明白编写更难破解的程序有多容易。

### 结语

本文介绍了自校验程序以及程序所使用的反篡改技术。我还阐述了解决这些方案背后的理论假设以及两个实际示例。你应该已经意识到，构建自定义的程序保护其实很容易，将它们与已知的商业保护工具相结合，会给破解者增加额外的障碍。幸运的是，另一方面，许多程序员甚至从未考虑过这些问题，这使得我们的破解生涯在很多时候变得相对容易。

### 参考资料

我建议你从现在开始阅读以下进一步阅读材料，以完善这部分知识：

`O'Reilly - Secure Programming Cookbook for C and C++`

`http://tutorials.accessroot.com` 中有很多教程，这些教程欺骗了涉及哈希的反篡改保护。

`Intel - IA-32 Intel Architecture Software Developer's Manual, volume 3: System Programming Guide`

... 以及基本上你能找到的所有目标为校验和应用的教程（包括我们教程版面上的其他文章）。

### 敬礼

感谢整个 `ARTeam` 小组：

`[Nilrem]` `[JDog45]` `[Shub - Nigurrath]` `[MaDMAn_H3rCuL3s]` `[Ferrari]` `[Kruger]` `[Teerayoot]` `[R@dier]` `[ThunderPwr]` `[Eggi]` `[EJ12N]`

`[Stickman 373]` `[Bone Enterprise]`

感谢所有花时间编写教程的人们。

感谢所有继续开发更好工具的人们。

感谢 `Exetools`、`Woodmann`、`SND`、`TSRH`、`MP2K` 以及其他所有人，他们是极佳的学习场所。

同时也感谢 `The Codebreakers Journal` 以及 `Anticrack` 论坛。

如果您有任何建议、意见或更正，请在通常的地方联系我。

`roxhaiy` 附注：我本想做一份完全汉化的网页，但无奈我的网页制作水平更低（唉…… `![](img/d23858dd221cce728a3a61b46badb044.jpg "恐怖!")` `![](img/d23858dd221cce728a3a61b46badb044.jpg "恐怖!")` ），做到一半后重新打开，里面的中文全部变成了问号，至今我也不知道哪里出错了…………

为了不耽误制作电子书的时间（我已经拖延得够久了），只能先将翻译好的部分图片打包上传，解压后全部覆盖到 `images` 文件夹即可。

下载地址：`http://bbs.pediy.com//attachment.php?s=&attachmentid=935`
