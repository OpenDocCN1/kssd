
# PE文件格式

## PE教程1: PE文件格式一览

考虑到早期写的PE教程1是自己所有教程中最糟糕的一篇，此番决心彻底重写一篇以飨读者。

`PE`的意思就是 Portable Executable（可移植的执行体）。它是 Win32环境自身所带的执行体文件格式。它的一些特性继承自Unix的`Coff` (common object file format)文件格式。"portable executable"（可移植的执行体）意味着此文件格式是跨win32平台的：即使Windows运行在非Intel的CPU上，任何win32平台的PE装载器都能识别和使用该文件格式。当然，移植到不同的CPU上PE执行体必然得有一些改变。所有win32执行体（除了VxD和16位的Dll）都使用PE文件格式，包括NT的内核模式驱动程序（kernel mode drivers）。因而研究PE文件格式给了我们洞悉Windows结构的良机。

本教程就让我们浏览一下PE文件格式的概要。

|  |
| --- |
| **DOS MZ header** |
| **DOS stub** |
| **PE header** |
| **Section table** |
| **Section 1** |
| **Section 2** |
| **Section ...** |
| **Section n** |

上图是PE文件结构的总体层次分布。所有PE文件(甚至32位的DLLs) 必须以一个简单的DOS MZ header 开始。我们通常对此结构没有太大兴趣。有了它，一旦程序在DOS下执行，DOS就能识别出这是有效的执行体，然后运行紧随MZ header 之后的DOS stub。DOS stub实际上是个有效的 EXE，在不支持PE文件格式的操作系统中，它将简单显示一个错误提示，类似于字符串"This program requires Windows" 或者程序员可根据自己的意图实现完整的 DOS代码。通常我们也不对DOS stub 太感兴趣：因为大多数情况下它是由汇编器/编译器自动生成。通常，它简单调用中断21h服务9来显示字符串"This program cannot run in DOS mode"。

紧接着DOS stub 的是**PE header**。

`PE header`是PE相关结构`IMAGE_NT_HEADERS`的简称，其中包含了许多PE装载器用到的重要域。当我们更加深入研究PE文件格式后，将对这些重要域耳目能详。执行体在支持PE文件结构的操作系统中执行时，PE装载器将从DOS MZ header中找到`PE header`的起始偏移量。因而跳过了 DOS stub 直接定位到真正的文件头`PE header`。

PE文件的真正内容划分成块，称之为**sections**（节）。每节是一块拥有共同属性的数据，比如代码/数据、读/写等。我们可以把PE文件想象成一逻辑磁盘，PE header 是磁盘的boot扇区，而sections就是各种文件，每种文件自然就有不同属性如只读、系统、隐藏、文档等等。**值得我们注意的是----节的划分是基于各组数据的共同属性:而不是逻辑概念。**重要的不是数据/代码是如何使用的，如果PE文件中的数据/代码拥有相同属性，它们就能被归入同一节中。不必关心节中类似于"data","code"或其他的逻辑概念：如果数据和代码拥有相同属性，它们就可以被归入同一个节中。（译者注：节名称仅仅是个区别不同节的符号而已，类似"data","code"的命名只为了便于识别，惟有节的属性设置决定了节的特性和功能）如果某块数据想付为只读属性，就可以将该块数据放入置为只读的节中，当PE装载器映射节内容时，它会检查相关节属性并置对应内存块为指定属性。

如果我们将PE文件格式视为一逻辑磁盘，PE header是boot扇区而sections是各种文件，但我们仍缺乏足够信息来定位磁盘上的不同文件，譬如，什么是PE文件格式中等价于目录的东东？别急，那就是PE header 接下来的数组结构**section table**（节表）。每个结构包含对应节的属性、文件偏移量、虚拟偏移量等。如果PE文件里有5个节，那么此结构数组内就有5个成员。因此，我们便可以把节表视为逻辑磁盘中的根目录，每个数组成员等价于根目录中目录项。

以上就是PE文件格式的物理分布，下面将总结一下装载一PE文件的主要步骤:

1. 当PE文件被执行，PE装载器检查DOS MZ header 里的PE header 偏移量。如果找到，则跳转到PE header。
2. PE装载器检查PE header 的有效性。如果有效，就跳转到PE header的尾部。
3. 紧跟PE header 的是节表。PE装载器读取其中的节信息，并采用文件映射方法将这些节映射到内存，同时付上节表里指定的节属性。
4. PE文件映射入内存后，PE装载器将处理PE文件中类似import table（引入表）逻辑部分。

上述步骤是基于本人观察后的简述，显然还有一些不够精确的地方，但基本明晰了执行体被处理的过程。

你应该下载[**LUEVELSMEYER的《PE文件格式》**](https://www.kanxue.com/technology/pe1.zip)。该文的描述相当详细，可用作案头的参考手册。

**翻译：**iamgufeng

[**Iczelion's Win32 Assembly Homepage**](http://win32asm.cjb.net/) [**LuoYunBin's Win32 ASM Page**](http://asm.yeah.net/)
