
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
