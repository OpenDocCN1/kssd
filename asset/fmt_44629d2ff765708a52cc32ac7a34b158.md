
# PE 文件格式

> 原文：[`www.pediy.com/kssd/tutorial/chap8-1-1.htm`](https://www.pediy.com/kssd/tutorial/chap8-1-1.htm)

# PE 教程 1: PE 文件格式一览

考虑到早期写的 PE 教程 1 是自己所有教程中最糟糕的一篇，此番决心彻底重写一篇以飨读者。

PE 的意思就是 Portable Executable（可移植的执行体）。它是 Win32 环境自身所带的执行体文件格式。它的一些特性继承自 Unix 的 Coff (common object file format)文件格式。"portable executable"（可移植的执行体）意味着此文件格式是跨 win32 平台的 : 即使 Windows 运行在非 Intel 的 CPU 上，任何 win32 平台的 PE 装载器都能识别和使用该文件格式。当然，移植到不同的 CPU 上 PE 执行体必然得有一些改变。所有 win32 执行体 (除了 `VxD` 和 16 位的 `Dll`)都使用 PE 文件格式，包括 NT 的内核模式驱动程序（kernel mode drivers）。因而研究 PE 文件格式给了我们洞悉 Windows 结构的良机。

本教程就让我们浏览一下 PE 文件格式的概要。

| **DOS MZ header** |
| :--- |
| **DOS stub** |
| **PE header** |
| **Section table** |
| **Section 1** |
| **Section 2** |
| **Section ...** |
| **Section n** |

上图是 PE 文件结构的总体层次分布。所有 PE 文件(甚至 32 位的 `DLLs`) 必须以一个简单的 DOS MZ header 开始。我们通常对此结构没有太大兴趣。有了它，一旦程序在 DOS 下执行，DOS 就能识别出这是有效的执行体，然后运行紧随 MZ header 之后的 DOS stub。DOS stub 实际上是个有效的 `EXE`，在不支持 PE 文件格式的操作系统中，它将简单显示一个错误提示，类似于字符串 "This program requires Windows" 或者程序员可根据自己的意图实现完整的 DOS 代码。通常我们也不对 DOS stub 太感兴趣: 因为大多数情况下它是由汇编器/编译器自动生成。通常，它简单调用中断 `21h` 服务 `9` 来显示字符串"This program cannot run in DOS mode"。

紧接着 DOS stub 的是`PE header`。 `PE header` 是 PE 相关结构`IMAGE_NT_HEADERS` 的简称，其中包含了许多 PE 装载器用到的重要域。当我们更加深入研究 PE 文件格式后，将对这些重要域耳目能详。执行体在支持 PE 文件结构的操作系统中执行时，PE 装载器将从 DOS MZ header 中找到 PE header 的起始偏移量。因而跳过了 DOS stub 直接定位到真正的文件头 PE header。

PE 文件的真正内容划分成块，称之为`sections`（节）。每节是一块拥有共同属性的数据，比如代码/数据、读/写等。我们可以把 PE 文件想象成一逻辑磁盘，PE header 是磁盘的 boot 扇区，而 sections 就是各种文件，每种文件自然就有不同属性如只读、系统、隐藏、文档等等。值得注意的是：节的划分是基于各组数据的共同属性：而不是逻辑概念。重要的不是数据/代码是如何使用的，如果 PE 文件中的数据/代码拥有相同属性，它们就能被归入同一节中。不必关心节中类似于"data", "code"或其他的逻辑概念: 如果数据和代码拥有相同属性，它们就可以被归入同一个节中。（译者注：节名称仅仅是个区别不同节的符号而已，类似"data", "code"的命名只为了便于识别，惟有节的属性设置决定了节的特性和功能）如果某块数据想付为只读属性，就可以将该块数据放入置为只读的节中，当 PE 装载器映射节内容时，它会检查相关节属性并置对应内存块为指定属性。

如果我们将 PE 文件格式视为一逻辑磁盘，PE header 是 boot 扇区而 sections 是各种文件，但我们仍缺乏足够信息来定位磁盘上的不同文件，譬如，什么是 PE 文件格式中等价于目录的东东？别急，那就是 PE header 接下来的数组结构`section table`（节表）。每个结构包含对应节的属性、文件偏移量、虚拟偏移量等。如果 PE 文件里有 5 个节，那么此结构数组内就有 5 个成员。因此，我们便可以把节表视为逻辑磁盘中的根目录，每个数组成员等价于根目录中目录项。

以上就是 PE 文件格式的物理分布，下面将总结一下装载一 PE 文件的主要步骤:

1. 当 PE 文件被执行，PE 装载器检查 DOS MZ header 里的 PE header 偏移量。如果找到，则跳转到 PE header。
2. PE 装载器检查 PE header 的有效性。如果有效，就跳转到 PE header 的尾部。
3. 紧跟 PE header 的是节表。PE 装载器读取其中的节信息，并采用文件映射方法将这些节映射到内存，同时付上节表里指定的节属性。
4. PE 文件映射入内存后，PE 装载器将处理 PE 文件中类似 import table（引入表）逻辑部分。

上述步骤是基于本人观察后的简述，显然还有一些不够精确的地方，但基本明晰了执行体被处理的过程。

你应该下载 LUEVELSMEYER的《PE文件格式》。该文的描述相当详细，可用作案头的参考手册。

**翻译：iamgufeng** [Iczelion's Win32 Assembly Homepage](http://win32asm.cjb.net/) [LuoYunBin's Win32 ASM Page](http://asm.yeah.net/)

`4dd466d0e1e0cc123afbe504da2462ca`
