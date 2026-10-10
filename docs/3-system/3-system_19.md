

# PE 教程 4: Optional Header

> 原文：[`www.pediy.com/kssd/tutorial/chap8-1-4.htm`](https://www.pediy.com/kssd/tutorial/chap8-1-4.htm)

 

# PE 教程 4: Optional Header

我们已经学习了关于 DOS header 和 PE header 中部分成员的知识。这里是 PE header 中最后、最大或许也是最重要的成员，optional header。

回顾一下，optional header 结构是 **IMAGE\_NT\_HEADERS** 中的最后成员。包含了 PE 文件的逻辑分布信息。该结构共有 31 个域，一些是很关键，另一些不太常用。这里只介绍那些真正有用的域。

这儿有个关于 PE 文件格式的常用术语: RVA   
RVA 代表相对虚拟地址。 知道什么是虚拟地址吗？相对那些简单的概念而言，RVA 有些晦涩。简言之，RVA 是虚拟空间中到参考点的一段距离。我打赌您肯定熟悉文件偏移量: RVA 就是类似文件偏移量的东西。当然它是相对虚拟空间里的一个地址，而不是文件头部。举例说明，如果 PE 文件装入虚拟地址(VA)空间的 400000h 处，且进程从虚址 401000h 开始执行，我们可以说进程执行起始地址在 RVA 1000h。每个 RVA 都是相对于模块的起始 VA 的。  
为什么 PE 文件格式要用到 RVA 呢? 这是为了减少 PE 装载器的负担。因为每个模块多有可能被重载到任何虚拟地址空间，如果让 PE 装载器修正每个重定位项，这肯定是个梦魇。相反，如果所有重定位项都使用 RVA，那么 PE 装载器就不必操心那些东西了: 它只要将整个模块重定位到新的起始 VA。这就象相对路径和绝对路径的概念: RVA 类似相对路径，VA 就象绝对路径。

| **Field** || Meanings |
| --- | --- |
| **AddressOfEntryPoint** || PE 装载器准备运行的 PE 文件的第一个指令的 RVA。若您要改变整个执行的流程，可以将该值指定到新的 RVA，这样新 RVA 处的指令首先被执行。 |
| **imageBase** || PE 文件的优先装载地址。比如，如果该值是 400000h，PE 装载器将尝试把文件装到虚拟地址空间的 400000h 处。字眼"优先"表示若该地址区域已被其他模块占用，那 PE 装载器会选用其他空闲地址。 |
| **SectionAlignment** || 内存中节对齐的粒度。例如，如果该值是 4096 (1000h)，那么每节的起始地址必须是 4096 的倍数。若第一节从 401000h 开始且大小是 10 个字节，则下一节必定从 402000h 开始，即使 401000h 和 402000h 之间还有很多空间没被使用。 |
| **FileAlignment** || 文件中节对齐的粒度。例如，如果该值是(200h),，那么每节的起始地址必须是 512 的倍数。若第一节从文件偏移量 200h 开始且大小是 10 个字节，则下一节必定位于偏移量 400h: 即使偏移量 512 和 1024 之间还有很多空间没被使用/定义。 |
| **MajorSubsystemVersion  
MinorSubsystemVersion ** || win32 子系统版本。若 PE 文件是专门为 Win32 设计的，该子系统版本必定是 4.0 否则对话框不会有 3 维立体感。 |
| **SizeOfimage** || 内存中整个 PE 映像体的尺寸。它是所有头和节经过节对齐处理后的大小。  |
| **SizeOfHeaders** || 所有头+节表的大小，也就等于文件尺寸减去文件中所有节的尺寸。可以以此值作为 PE 文件第一节的文件偏移量。 |
| **Subsystem** || NT 用来识别 PE 文件属于哪个子系统。 对于大多数 Win32 程序，只有两类值: Windows GUI 和 Windows CUI (控制台)。 |
| **DataDirectory** || 一**IMAGE\_DATA\_DIRECTORY **结构数组。每个结构给出一个重要数据结构的 RVA，比如引入地址表等。 |

**翻译：****iamgufeng \[**[**Iczelion's Win32 Assembly Homepage**](http://win32asm.cjb.net/)**\]****\[**[**LuoYunBin's Win32 ASM Page**](http://asm.yeah.net/)**\]**

 

 4dd466d0e1e0cc123afbe504da2462ca