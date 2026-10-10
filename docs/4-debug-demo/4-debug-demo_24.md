

# Beginner Tutorial #10 【翻译】

> 原文：[`www.pediy.com/kssd/pediy12/23789.html`](https://www.pediy.com/kssd/pediy12/23789.html)

Beginner Tutorial #10   
By: Shub-Nigurrath /ARTeam  
http://cracking.accessroot.com  
  
Anti-tampering Techniques Theory v1.0  
   
The Target:  
none   
The Tools:  
Ollydbg 1.10   
The Protection:  
none   
Other Information:  
This tutorial will cover some basic concepts about attacking applications in general and anti-tampering techniques. Explicit aim of this tutorial is to help making a clear understanding and classification of concepts seen in the beginners tutorials on this series.   
   
Best viewed in Firefox at 1280x1024  
  
  
初学者教程 #10   
By: Shub-Nigurrath /ARTeam  
http://cracking.accessroot.com  
  
反篡改技术理论 v1.0  
  
目标：无  
工具：Ollydbg 1.10   
保护方式：无  
其他信息：这篇教程将涉及常规的攻击程序与反篡改技术的一些基本概念。帮助大家弄懂和区分本系列教程中提到的各个概念是本篇的明确目标。  
  
  
1. Introduction  
   
This tutorial is meant to cover a topic that is usually just slightly pointed in common tutorials. The argument is the anti-tampering techniques used by programs to counteract the modifications of their code. Told this way, it could seem something you never seen around, but this is the name of techniques such as CRC self-checking of an application, MD5, SHA1, and so on. Did you realize it?   
  
1.介绍  
本章会涉及一个普通教程很少提到的主题：讨论反篡改技术（即程序用于防止其代码被修改的技术）。讨论过程中，有些技术名词你以前可能还没见过，例如应用程序的 CRC 自校验、MD5、SHA1 等等。你了解它们吗？  
  
Self-checking (also called self-validation or integrity checking) is a technique in which a program, while running, checks itself to verify that it has not been modified. Usually it's useful to distinguish between static self-checking, in which the program checks its integrity only once, during startup, and dynamic self-checking, in which the program repeatedly verifies its integrity, as it is running. While self-checking alone is not sufficient to robustly protect software, is nowadays one of the most easier protections to implement and widely used either in commercial or custom protections.  
  
自校验（也叫自验证或完整性校验）就是程序运行期间校验自身代码以确认没有被修改的技术。一般来说，自校验分为静态自校验和动态自校验。静态自校验仅在程序开始运行时验证代码完整性一次；而动态自校验则是在程序运行期间反复检测。虽然单有自校验还不足以万无一失地保护软件，但它仍是现今最容易实现的保护技术之一，并且已广泛应用到商业或客户保护中。  
  
To correctly discuss this issue I'll resemble some things coming from different sources and some contributions on my own. As usual the distinction between my original work and what is coming from another source is left intentionally hidden.  
  
为了恰当讲述这个问题，我的方法会跟一些从各方面收集的例子及自己拥有的文献相似。与其他资料的差别是，我的文章会有意忽略某些问题。  
  
This tutorial presents the anti-tampering problem and some details of the CPU memory management which will help understanding the whole thing. An example in C is then built and debugged as a proof of concepts.  
  
have phun.  
  
这章教程提出了反篡改问题及 CPU 内存管理的一些细节，这些细节会帮助你理解整个过程。作为教学概念的验证，我会以 C 语言作例子进行编译和调试。  
   
  
2. Classification of Attacks  
   
"The security of a cypher scheme shouldn't rely on keeping secret the algorithm.  
The security depends only on keeping the used key secreted"  
Auguste Kerckhoffs, La Cryptographie Militaire,   
Journal des Sciences Militaires, vol.9, February 1883.  
  
Before starting to discuss the argument it is better to introduce a classification of attacks types, which helps keeping ideas clear.  
  
Broadly speaking the attacks to a system (software or not) can be classified into three categories based on the nature of malicious agent.  
  
2.攻击的分类  
  
“加密方案的安全性不应该依赖对算法的保密，而只需依靠对密钥的的保密。”  
——Auguste Kerckhoffs, La Cryptographie Militaire,   
Journal des Sciences Militaires, vol.9, February 1883.  
  
讨论开始前不如先介绍一下攻击类型的分类，使你保持清晰的概念。  
  
概括来讲，对一个系统（软件或其他）的攻击可基于恶意主体的性质而分为三类。  
  
  
The first one, the simplest to detect and handle is when the perpetrator breaches communications access controls to attack the system. The malicious agent is still under the restrictions of the communication protocol. A good example of this genre would be a standard hacker type attack. Robust access control mechanism that isolates the system hardware and software totally form the user is enough to counter this attack.   
  
第一类，入侵者以破坏通信访问控制来攻击系统是最容易检测和处理的。恶意主体仍受通信协议的约束。黑客攻击就是这类攻击手法的典型例子。用健壮的访问控制机制使用户与系统软、硬件完全隔离，就足够抵抗这类攻击。  
  
The second, comparatively more serious attack would be like a computer virus. Such attacks originate as software running on the platform. Though the attackers have breached the communications, they must still depend on the BIOS and OS interfaces. Such attacks are generally the aftermath of a class I attack, and forebode class 3 attacks. Detection of these types of attacks is facilitated by the fact that the perpetrators attack classes of files.   
  
第二类，更加危险的攻击与计算机病毒作用相似。攻击通过运行操作平台上的软件进行。尽管攻击代码已经破坏了通信系统，但他们的活动仍须依靠基本输入输出系统接口和操作系统接口。这类攻击通常在第一类攻击之后，也是第三类攻击的预兆。基于入侵者攻击文件的类的事实，检测此类攻击是不难的。  
  
Class 3 attacks are the most difficult to resolve and detect as the perpetrator is an insider. The perpetrator may modify software or hardware for the system at will. The only form of prevention for this type of attacks is to raise the technological bar to such an extent that the perpetrator would end up wasting so much time and resources that it would be a poor investment. The technical bar could be varied from no-specialized-analysis-tools to specialized-hardware-analysis tools.   
  
第三类攻击最难发现与解决，原因入侵者就是内部人员，他可以根据不同系统随意修改软件或硬件。防范此类攻击的唯一方式是提升整个防护技术体系，使得入侵者消耗了大量时间和资源却得不到回报而放弃。从非专用分析工具到专用硬件分析工具，这种防护技术体系应该是多样性的。  
  
Under the heading of software protection, there is another little classification to discuss. The techniques for protecting programs can be distinguished roughly into   
  
techniques for protecting for protecting software from reverse engineering (by obfuscation),   
  
modification (by software tamper resistance),   
  
program-based attacks (by software diversity),   
  
and BORE ? Break-Once Run Everywhere ? attacks (by architectural design).   
  
在软件保护的范围下，有其他小类去讨论。程序保护技术可初步分为：  
  
防止软件被逆向的保护技术（混淆代码）  
  
代码修改（软件抗篡改）  
  
基于程序的攻击（软件相异性）  
  
BORE（Break-Once Run Everywhere）——泄露一次各处运行的攻击（架构设计）  
  
  
  
3. Tamper Resistance Techniques and Checksumming  
   
Introduction.  
Software tamper resistance is the art of crafting a program such that it cannot be modified by a potentially malicious attacker without the attack being detected. In some respects, it is similar to fault-tolerant computing, in that potentially dangerous changes in program state are detected at runtime. For tamper resistance, however, it is assumed that intelligent, malicious attackers (rather than hardware flaws or software errors) may be responsible for such changes. Self-checking tamper resistance is distinguished in its ability to run on current unmodified commodity hardware without requiring third parties, because all the tests are performed by the application on its own code/data.   
  
3.抗篡改技术与校验和  
  
介绍  
软件抗篡改是编写程序的艺术，使代码在没有攻击检测的情况下，不会被潜在的有恶意的攻击代码所修改。在某些方面，它类似于容错处理，在程序运行期间检测存在潜在威胁的改动。由于抗篡改技术，可假定明智和暴力的入侵者将为篡改软件造成的硬件损坏或软件错误负责。自校验防篡改程序的所有应用测试基于内建代码系统，具有在不需要第三方支持的条件下固化于硬件内部的卓越性能 。  
  
There are of course different techniques for self-checking software (e.g. program or result checking and generation of the correct executable or part of code on the fly, decryption according to a calculated digest) partially derived from asymmetric signature algorithms, but we'll focus more on checksumming.  
  
当然了，带有自校验的软件也用了不同技术（例如在编程、结果检测、标准执行阶段或者代码片断空闲时，根据合适的处理解码），也有部分技术是非对称签名算法的派生，我们更多关注的是校验和技术。  
  
The standard threat model for software tamper resistance is the hostile host model. The challenge is to protect an application running on a system controlled by a malicious, intelligent user. Because such a user can in theory change any code on the computer, other software on such a system, including the operating system, is untrusted; in the case of particularly determined adversaries, even the hardware is untrusted. This situation is in contrast with the hostile client problem in which we assume a trusted host and untrusted application. The hostile client problem appears to be an easier problem to solve; numerous solutions have been developed, e.g. sandboxing and is typical of mobile agents such as smartphones or PDAs.  
  
一般对于抗软件篡改的威胁模式，来自攻击者主机中。在聪明而有恶意的用户控制的系统中保护一个应用程序是个难题。理论上，这个用户可以改变计算机的任何代码、软件和系统，包括操作系统。如果遇到特别坚决的对手，甚至连硬件也可以改动。与此对比的是，入侵者只控制客户端应用软件而主机不受其操控的情况。只有客户端受控的问题看来较易解决，针对此已经开发了众多方案。例如 sandboxing 技术和典型的可移动终端，像智能手机或者掌上电脑。  
  
Since a single checksum is relatively easy for an attacker to disable, stronger proposals rely on a network of inter-connected checksums, all of which must be disabled to defeat tamper resistance. A tester reads the area of memory occupied by code and readonly data, building up a checksum result based on the data read. A subsequent section of the code may operate on the checksum result, affecting program stability or correctness in a negative way if a checksum result is not the same as a known good value pre-computed at compile time. The sections of code which perform the checksumming operations may be further hidden using code obfuscation techniques to prevent static analysis. Ideally the effects of a bad checksum result in the program are subtle (e.g. causing mysterious failures much later in execution) thus making it much more difficult for an attacker to locate the checksum code.  
  
因为单一的校验和技术比较容易被攻击代码屏蔽，所以有人提出更安全的方案：互联校验和网络，只有令整个网络失效，才能对付这种抗篡改技术。检测码会读取装有代码和只读数据的内存区域，根据读得的数据计算其校验和。下一区域的代码则要靠上面计算的校验和结果运作，如果计算校验和的结果与程序编译时预计算的值不符，就会影响程序的稳定性和正确性。利用代码混淆技术深藏执行校验和计算的代码区，将会防止对程序进行静态分析。理论上，校验和结果有误，并非立即在程序中体现（例如在运行较长时间后才出现奇怪的错误），从而使入侵者对校验和代码的定位更加困难。  
  
Figure 1 - A good distribution of checksum blocks within a code segment.  
  
Figure 1 gives a simplified view of a typical distribution of checksum code within an application. In practise, there may be hundreds of checksum blocks hidden within the main application code. Each allows verification of the integrity of a predetermined section of the code segment. The read-only data segment may also be similarly checked.   
  
图 1 ——校验和模块在代码段中的有效分配  
  
图 1 简略示意了在程序中校验和模块的典型分配方式。事实上，也许有几百个校验和模块隐藏在主程序代码中。每个模块都对代码段中的预定区域进行完整性检验。即使是只读数据段也会进行简单检查。  
  
The checksumming code is inserted at compile time and integrated with regular execution code. The application also requires a correct checksum result for each block in order to work properly.  
  
The resulting scheme is an Interlocking Trust Model, for which each segment not only on itself but also on other segments to effectively perform its task. Thus each segment of code would be made responsible for maintaining and verifying the integrity of other segments.  
  
校验和代码在程序编译时就被嵌入，并与常规的可执行代码结合。要程序运行正常则需每个模块的校验和结果正确。  
  
综合方案就是使用连锁校验模式，使每段代码不仅仅存在于自身程序段中，而且还要置于其他程序段来有效执行。因此每个代码段将要负责维持和核对其他代码段的完整性。  
  
  
High level view of how the protection works, Testers and Guards  
  
The overall protection works like this: at runtime, a large number of embedded code fragments called  testers test small segments of code for integrity (using a linear hash function and an expected has value); if integrity fails, an appropriate response is pursued. The peices of code called on check failures are called guards and can be programmed to carry out arbitrary tasks (for example common guards are peices of code repairing the code,  erasing the breakpoints or downloading a fresh copy of the application).   
  
保护架构的高级运作模式，测试码与防护码  
  
全面的保护架构的运作如下：在程序运行期间，称为测试码（Testers）的大量嵌入代码片断会检测小型代码段的完整性（利用线性散列函数及预期值），如果检测不通过，则会作出恰当的响应。调用错误检测的代码叫防护码（Guards），它可被编写来执行任何任务，例如常见的防护码可以修复程序代码、清除断点和载入最新的应用程序副本。  
  
Two of the most common guards are checksumming and repairing.  
  
Checksum Guards: The checksum guards are programmed to check a code fragment within the software. If this code is tampered with in any way the guard takes action. The type of action the guard takes can vary in degree from logging the infraction  halting the program.   
  
两种最普遍的防护码：校验和防护码与修复防护码  
  
校验和防护码（Checksum Guards）：校验和防护码用于检测软件内部的代码片断。如果通过任何方式篡改软件代码，防护码将会采取行动。它的反应会随着非法操作的程度作出改变，并以此中断程序。  
  
Repair Guards: The repair guards can fix a tampered section of code. Thus, the changes an attacker made are wiped out and the program remains secure. The repair can be made by storing a clean copy of the segment of code the guard is protection somewhere, and then replacing the dirty copy with the clean one.   
  
The security of the guards actually comes from guards watching each other's backs. Each guard protects fragments of code, and that includes protecting other guards. Figure 1.1 shows an example of a possible network of guards.  
  
  
修复防护码（Repair Guards）：修复防护码可以修补被篡改的代码段。因此，攻击代码对程序的改动会被清除，程序仍是安全的。一个“干净”的代码段副本会放在某个受保护的区域，修复码将使用这个副本来覆盖遭篡改的代码段。  
  
防护码技术的安全性在于其环环相扣——每个防护码所保护的代码段，都包含维护其他防护码的代码。图 1.1 就是一个合理的防护码网络例子。  
  
Figure 1.1 - A possible distribution of cross-checking guards of an application's integrity.  
  
A group of guards working together implement a sophisticated protection scheme that is more resilient against attacks than a single guard. These schemes are used by the most modern protectors, such as Armadillo also: the Nanomities trick is just a way to implements a Guard scheme and an interlocking scheme, made of two processes, controlling one each other.  
  
图 1.1——防护码交叉检测程序完整性的的一种分配方式  
  
用一组防护码实现复杂的保护方案，其抗攻击的强度比单个防护码更好。一些最新的加壳软件也采用了类似机制，例如 Armadillo，它只是用了防护码与连锁校验的把戏，建立了两个相互控制的进程而已。  
  
In order to perform their duties, a network of guards is placed into the program and hooked to its execution flows in proper ways, before of the controlled code goes into execution: a repairing guard has to be inserted into a point in the control flow that is to be reached before guarded code (in execution order). In other words the guard has do dominate the guarded code. Guards can also be controlled by other guards too. Figure 1.2 reports another real more sophisticated example where a network of Guards (G) and Guarded Code (C) is presented beside it's distribution into the real application's memory.  
  
防护码网络运作在控制码执行程序之前，它会在程序中通过恰当的方式与执行流挂钩：修复防护码会在控制流到达受保护代码前（按执行次序）嵌入。换句话讲，防护码完全支配了受保护代码。当然，防护码也会受其他防护码控制。图 1.2 是另一更复杂的实例，它显示了由防护码（G）和受保护代码（C）组成的网络，也包括它们在真实程序内存中的分配方式。  
  
Figure 1.2 - A Guards graph and the corresponding memory layout of the guarded program.  
  
Defeating complex Guard Networks can be, generally speaking, a nightmare.  
  
图 1.2——防护码网络图示及受保护程序在相应内存中的布局  
  
一般来说，破解复杂的防护码网络就如恶梦一般。  
  
Anti-Tampering details  
There are several aspects of such checksumming which an attacker must keep in mind:  
  
1. Because of the overlapping network of testers, almost every checksumming block must be disabled at the same time in order for a tampering attack to be successful.   
2.The resulting value from a checksum block must remain the same as the original value determined during compilation (or all uses of the checksum value must be determined and adjusted accordingly), if the results of a checksum are used during standard program execution.   
3.The checksum values are only computed for static (i.e. runtime invariant) sections of the program.   
4.Checksumming code is obfuscated, hard to find, and the use of checksum results is also hidden.   
  
抗篡改的一些细节  
有关校验和的若干特点，入侵者必须熟记于心:  
  
1.由于测试码重叠网络的存在，要破解成功必须同时将每个校验和模块处理掉。  
2.如果校验和结果在程序正常运行时起作用，从校验和模块得到的值必须与编译时计算的值相同（或者必须求出校验和值的全部作用，而且其效果会作出相应调整）。  
3.只能对程序的静态（即运行期间不变）的区域计算校验和。  
4.校验和代码常与其他代码混淆，难于发现，其结果的效用也是隐藏的。  
  
Note that a critical (implicit) assumption of checksumming algorithms is that D(x) = I(x), where D(x) is the bit-string result of a “data read” from memory address x, and I(x) is the bit-string result of an “instruction fetch” of corresponding length from x. In  the practical everyday cases this is the situation, because D(x) and I(x) are the same portion of memory "read as data and as instructions".   
  
注意校验和算法的关键（隐含）假设是 D(x) = I(x)，D(x)是读取内存地址 x 处数据的位串结果，I(x) 则是 x 处集取指令的相应长度的位串结果。在实际日常的事例中，D(x) 和 I(x)就是内存中读取数据和指令的同一部分。  
  
If I(x) were different from D(x), then the checksumming code would always check using D(x) while the processor would always execute I(x).   
Checksumming aims to verify that the code the processor executes is the original code, and thus assumes that the code it reads is the code the processor executes.   
  
While current checksumming proposal critically rely on this assumptions there are opportunities to violate this assumption on several modern processors..  
  
如果 I(x)与 D(x)不同，当处理器不断执行 I(x)时，校验和代码会一直利用 D(x)作检测。  
计算校验和的目的是验证处理器执行的是否为原始代码，而因此推测它读取的就是处理器所执行的代码。  
  
当前的校验和方案严重依赖于上述假设，而个别现代处理器已有可能进行暴力破解了……  
  
   
4. CPU Support for Virtual Memory  
   
This section provides background information for those less familiar with the virtual memory subsystems of modern processors, including translation look-aside buffers (TLBs). Thos of you already familiar with processor architecture can jump directly to the next section.  
  
4.CPU 对虚拟内存的支持  
  
本部分提供了现代处理器的虚拟内存子系统的背景信息，其中也包括了 TLBs（旁路转换缓冲器，也译作关联高速缓存），这些信息并不常见。如果你已经对处理器结构很熟悉，可以直接跳到下一部分。  
  
Modern processors do much more than execute a sequence of instructions. Advances in processor speed and flexibility have resulted in a very complex architecture. A significant part of this complexity comes from mechanisms designed to efficiently support virtual memory. Virtual memory, first introduced in the late 1950’s, involves splitting main memory into an array of frames (pages) which can be subsequently manipulated. Virtual addresses used by an application program are mapped into physical addresses by the virtual memory system.  
  
现代处理器已不仅仅局限于顺序执行指令了。其复杂的结构带来的是高速与高适应性。这种复杂结构的重要部分就是有效支持虚拟内存的机制。50 年代末最早提出了虚拟内存机制，与此相应，把主内存划分为页帧（页面）再进行操作。在虚拟内存机制下，应用程序通过使用虚拟地址来映射物理地址。  
  
Figure 2 - Translation of Virtual Address into a Physical Address  
  
Even though the page table translation algorithm may vary slightly between processors and may sometimes be implemented in software, modern processors all use roughly the same method for translating a virtual page number to a physical frame number. Specifically, this translation is performed through the use of page tables, which are arrays that associate a selected number of virtual page numbers with physical frame numbers.  
  
图 2——把虚拟地址转换成物理地址  
  
即使页表转换算法会由于处理器的不同而略有差异，而且有时也会在软件中执行，但现代处理器大都使用相同的原理把虚拟页面号转换成物理页帧号（划分虚拟内存使用的页面称为页面（Page），划分物理内存使用的页面称为页帧（Frame），译者注）。明确地说，转换是通过页表（Page Table）实现的，页表是选定的虚拟页面码与物理页帧码相关联的数组。  
  
Because the virtual address spaces of most processes are both large and sparse, page table entries are only allocated for the portions of the address space that are actually used. To determine the physical address corresponding to a given virtual address, the appropriate page table, and the correct entry within that page table must be located.   
  
由于大部分进程的虚拟地址空间大而散，页表入口只能定位在实际使用的那部分地址空间上。为了找到指定虚拟地址所对应的物理地址，必须定位于合适的页表及其中正确的入口。  
  
For systems that uses 3-level page tables, a virtual address is divided into four fields, x1 through x4. The x1 bits (the directory offset) specify an entry in a perprocess page directory. The entry contains the address of a page map table. The x2 bits (the map offset) are used as an offset within the specified page map table, giving the address of a page table. The x3 bits (the table offset) index into the chosen page table, returning the number of a physical page frame. x4, then, specifies the offset within a physical frame that contains the data referred to by the original virtual address.   
  
This resolution process is illustrated in Figure 3. Note that if memory segments are used, segment translation typically occurs before operations involving the page table.  
  
因为系统使用了三层页表，所以一个虚拟地址被分为从 x1 到 x4 的四个区域。x1（目录偏移）指定了单进程页目录的入口，入口包含了页映射表的地址。x2（映射偏移）是指定页映射表内部偏移量，传递对应的页表地址。x3（页表偏移）根据选定页表索引，返回对应物理地址的页帧号。x4 指定了这个物理页帧内部的偏移量，它就是原虚拟地址的页内偏移。  
  
  
Figure 3 - Translation of a Virtual to Physical Address through Page Tables  
  
TLB Translation look Aside Buffer  
Because multiple memory locations must be accessed to resolve each virtual memory address, virtual address translation using page tables is a expensive operation. To speed up these mappings, a specialized high-speed associative memory store called a translation look-aside buffer (TLB) is used.   
  
图 3——通过页表将虚拟地址转换为物理地址  
  
TLB——旁路转换缓冲器  
由于分解每个虚拟内存地址都必须访问多个内存定位，用页表进行虚拟地址的转换是很繁琐的操作。为了提高映射速度，人们使用了专用的高速关联内存储存器，叫做旁路转换缓冲器（TLB）。  
  
A TLB caches recently used mappings of virtual page numbers to physical page frames. On every virtual memory access, all entries in a TLB are checked to see whether any of them contain the correct virtual page number. If an entry is found for the virtual page number, a TLB hit has occurred, and the corresponding physical page frame is immediately accessed. Otherwise, we have a TLB miss, and the appropriate page tables are consulted in the fashion discussed previously. The mapping so determined is then added to the TLB by replacing the mapping that was least recently used. Figure 4 illustrates what happens on a TLB hit.  
  
TLB 缓存用于把虚拟页面码映射到物理页帧的新技术。访问每个虚拟内存地址时,CPU 会检查 TLB 内全部入口，看是否有所需的虚拟页号。如果找到这个入口，发生 TLB 命中，就可以立即访问相应的物理页帧。否则就是 TLB 失效，要按之前所说的方式访问合适的页表，同时更新 TLB 的映射。图 4 显示了 TLB 命中的发生。  
  
  
Figure 4 - Address Translation using a TLB  
  
Because of the principal of locality, TLB translation works very well in practise. System designers have noticed, however, that code and data exhibit different patterns of locality. To prevent interference between these patterns, caches of code and data are often separated; for similar reasons, most modern CPUs have separate code and data TLBs.   
  
图 4——使用 TLB 进行地址转换  
  
由于虚拟存储系统局部性（locality）的重要性，实际上 TLB 转换机制工作良好。系统设计师已经注意到，代码与数据无论如何都会执行不同的局部性模式。为了防止不同模式之间的冲突，代码和数据的缓存经常是分离的。基于这个简单的原因，大部分现代 CPU 拥有独立的代码及数据 TLB。  
  
CPU caches mark referenced memory as code or data depending upon whether it is sent to an instruction decoder. Whenever an instruction is fetched from memory, the instruction pointer is translated via the instruction TLB into a physical address. When data is fetched or stored, the processor uses a separate data TLB for the translation. Using different TLB units for code and data allows the processor to maintain a more accurate representation of recently used memory. Separate TLB’s also protect against frequent random accesses of code (data) overwhelming both TLB’s. Because most code and data references exhibit high degrees of locality, a combination of small amounts of fast storage (e.g. onchip memory caches) and more plentiful slower storage (DRAM memory) can together approximate the performance of a larger amount of fast storage.  
  
CPU 缓存像标记代码和数据一样标记被引用的内存，是依靠判断它是否被发送到指令解码器。无论何时从内存中取一个指令，指令指针都会经指令 TLB 的翻译后指向物理地址。当数据存取时，处理器会使用独立的数据 TLB 来进行翻译。为代码和数据提供不同的 TLB 单元，能使处理器对最近使用内存的表达更为准确，也能够防止代码（数据）对整个 TLB 系统极为频繁的随机访问。由于大部分代码和数据引用的局部性程度高，少量快速存储器（例如高速缓存）与大量较慢存储器（如 DRAM 内存）的组合将接近大量快存的执行速度。  
  
Page swapping  
  
Because the memory management unit presents a virtual address space to the application running, the application need not be aware of the physical sections of memory which it actively uses. Thus even though the virtual address space of a program is contiguous, the physical regions of memory it uses may not be. This presents a great opportunity for the operating system. Not only does it allow multiple applications to be run on the system (each with its own unique virtual address space, mapping to different physical pages), but it allows the operating system to only keep in physical memory those parts of each application required at the current time.   
  
页面置换  
  
由于内存管理单元对运行中的应用程序提供了虚拟地址空间，所以应用程序就不需知道其占用的物理内存段。因此，即使程序的虚拟地址是连续的，其所在物理内存区域也不一定连续。这样就给了操作系统一个巨大的机会——不仅可以让多种程序同时运行（每个程序拥有唯一的虚拟地址空间，并映射到不同的物理页面），而且允许操作系统只保留当前每个程序需求的那部分物理内存。  
  
Since not all pages of virtual memory may map to a physical page, there must be some way for the processor to inform the OS when a virtual address does not have a physical mapping. The processor does this through the use of a page fault interrupt. The processor will store the virtual address which caused the page fault in a register, and then signal the operating system through an interrupt handler. The operating system updates the mapping of virtual to physical addresses, so that the requested virtual address can be mapped to a physical address.   
  
因为不是全部的虚拟内存页都映射到物理页，所以必定有某种方式让处理器在虚拟地址没有物理映射的时候告知操作系统。处理器通过缺页中断来进行这个操作。它把导致页面失效的虚拟地址装入寄存器中，再利用中断句柄来通知操作系统。操作系统随即更新地址映射, 以便被请求的虚拟地址能映射到物理地址。  
  
This may mean bringing the section of the program into physical memory from disk or some other external storage. The OS then signals the processor to retry the instruction by returning from the interrupt. The OS also has the choice of aborting execution of the application if it determines that the virtual address is invalid, e.g. if the virtual address refers to memory that has not been located.  
  
这意味着可以把磁盘和其他外存储器中的程序段放到物理内存中。然后操作系统通知处理器重发指令从中断处返回。假如虚拟地址无效，操作系统也能异常终止程序的执行,例如虚拟地址引用了没有定位的内存。  
  
Access Controls on Memory.  
  
Along with the translation of a virtual to physical address, the processor may implement access protection on memory regions. Since the virtual memory subsystem already splits physical memory into small areas (frames), it makes sense that the same memory management unit would also implement access control on a per-frame basis. The most important protection is that only pages that an application is allowed to access are mapped into its page table. To prevent an application from manually mapping a page into its address space, the page directory base pointer is stored in a read-only register, and the frames containing a process’s page table are themselves not accessible by the process.   
  
内存访问控制  
  
随着虚拟到物理地址的转换，处理器会在内存区执行访问保护。因为虚拟内存体系把物理内存分为多个区域（页帧），所以同样的内存管理单元对每页帧执行访问控制才会有意义。最为重要的保护是，只有程序容许访问的页面才能映射到页表。为了防止程序手动将页面映射到它的地址空间，页目录基指针被贮存在只读寄存器中，而且含有进程页表的页帧也不能用同样进程来访问。  
  
In addition, there are protection mechanisms for pages which are in a process’s address space. Each mapped page is restricted in the types of operations that may be performed on its contents: read, write, and instruction fetch (also called execute). Permitted operations are specified using control bits associated with each page table entry. Read and write are common operations on data pages, while executing code is commonly associated with a page containing executable code.  
  
Modern operating systems take advantage of the protection mechanisms implemented by the processor to distinguish various types of memory usage.  
   
此外，对于进程寻址空间中的页面也有保护机制。每个被映射的页面都有读、写以及取指令（也叫执行）的限制。能允许的操作须指定使用与每个页表入口关联的控制位。读、写数据页是普通的操作，因为执行代码通常与含可执行代码的页面相关联。  
  
现代操作系统利用处理器实现的保护机制来辨别各类内存应用。  
  
The ability to set no-execute permission on a per-page basis produces the restriction that many programs are confined to executing code from their code segment, unless they take specific action to make their data executable.   
  
Although such changes can interfere with systems that generate machine code at runtime (e.g. modern Java Virtual Machines), many types of code injection attacks can be defeated by nonexecutable data pages  
  
在每页面基础上设置无执行权限，会限制很多程序从自身代码段中运行代码，除非用特殊方法使其数据可执行。  
  
即使这种改变可以干预在运行期间生成机器码的系统（例如 JAVA 虚拟机），但通过不可执行数据页能够防御多种代码注入攻击。  
  
Table 1 - Separation of access control privileges for different page types  
  
Table 1 shows the ideal separation of privileges for different sections of an application. This separation of privileges is currently assumed in executable file formats. All processors implementing page level access controls must check for disallowed operations and signal the operating system appropriately. Most often, the operating system is signalled through the page fault interrupt, which indicates the memory reference that caused the invalid operation.  
   
表 1——不同页面类型的访问控制权划分  
   
表 1 显示了程序不同区段的理想权限分配。通常假定可执行文件格式使用这种权限分配。所有处理器要实现页面分级访问控制必须检查不允许的操作，并且通知操作系统。一般情况下，以缺页中断来通知操作系统，中断会指出引起无效操作的内存引用。  
  
  
5. The memory model of the x86 architecture, the segments  
   
In addition to supporting memory pages, the x86 can also manage memory in variable sized chunks known as segments.  
  
x86 体系的内存模式及分段  
  
x86 体系通过管理可变大小内存块（也称为段）来维持内存页面。  
  
Associated with each segment is a base address, size, permissions, and other meta-data. Together this information forms a segment descriptor. To use a given segment descriptor, its value is loaded into one of the segment registers. Other than segment descriptor numbers, the contents of these registers are inaccessible to all software. In order to update a segment register, the corresponding segment descriptor must be modified in kernel memory and then reloaded into the segment register.  
  
与每个分段相关联的信息有基地址、大小、权限和其他中间数据。这些信息的集合构成了段描述符。要使用给定的段描述符，它的值需载入段寄存器中。除了段描述符的号码以外，这些寄存器的内容对所有软件来说都是不可访问的。更新段寄存器时，与其相应的段描述符必须在核心内存中修改，然后重新载入段寄存器。  
  
A logical address consists of a segment register specifier and offset. To derive a linear address, a segment register’s segment base (named by the segment specifier) is added to the segment offset. An illustration of the complete translation mechanism for the x86 architecture is shown in Figure 6. Code reads are always relative to the code segment (CS) register, while normally, if no segment register is specified data reads use the data segment (DS) register. Through segment overrides a data read can use any segment register including CS. After obtaining a linear address, normal page table translation is done as shown in Figure 6 and Figure 7.  
  
一个逻辑地址包含了段寄存器的说明符及偏移量。为了获得线性地址，要将段寄存器的段基（以段说明符命名）加上段偏移量。图 6 显示了 x86 体系的地址完整转换机制。读代码总是与代码段（CS）寄存器有关，如果不作特别指定，读数据通常使用数据段（DS）寄存器。通过段超越可以使用包括 CS 在内的任何段寄存器来读数据。得到线性地址后，正常页表转换就完成了，如图 6、7 所示。  
  
Figure 6 - Translation from virtual to physical addresses on the x86  
  
图 6——x86 体系中虚拟地址到物理地址的转换  
  
Figure 7 - Translation of a get using segment overrides  
  
图 7——使用段超越的转换  
  
Unlike pages on the x86, segments can be set to only allow instruction reads (execute-only). Data reads and writes to an execute-only segment will generate an exception. This execute-only permission can be used to detect when an application attempts to read memory relative to CS. As soon as the exception is delivered to an OS modified for our attack, the OS can automatically modify the memory map (see Figure 7) to make it appear as if the unmodified data was present at that memory page.   
  
x86 体系中，与分页不同，分段可设置为仅允许读指令（只执行）。在只执行段中读写数据，会产生一个异常（exception）。这种只执行权限可用来检测应用程序在什么时候企图读取有关 CS 的内存。当异常送到一个被攻击并修改的操作系统时，这个操作系统能自动修改内存映射（参看图 7）使其内容出现，例如未修改的数据在内存页中呈现。  
  
Most operating systems for x86, however, now implement a flat memory model. This means that the base value for the CS and DS registers are equal; an application need not use the CS register to read its code. A flat memory model will ensure that both linear addresses are the same, resulting in the same physical address (see the dash-dot-dot line in Figure 7).  
  
大部分的 x86 操作系统，已实现内存平面内存模式（flat memory model）——意谓 CS 与 DS 寄存器的基值是相等的；也就是说程序不必用 CS 寄存器读取代码。 平面内存模式会确保两者线性地址相同，产生相同的物理地址（参考图 7 的虚线部分）。  
  
   
6. Writing and debugging a selfchecking program in C  
   
Resuming what we said till now it can be stated that: "detecting whether portions of a binary have been modified is essentially an error-detection problem; therefore, a checksum algorithm such as CRC32, MD5, or SHA1 can be used to generate a signature for an arbitrary block of code or data. This signature can then be checked at runtime to determine whether any modification has taken place."  
    
6.用 C 语言编写调试自校验程序  
  
继续我们的话题，它可以表述为：“检测机器码是否被修改本质上属于错误检测问题；为此，校验和算法（如 CRC32、MD5 或 SHA1）可对任意代码块或数据块生成标记。随后检查这个标记以确定程序是否在运行期间被修改过。”  
  
I have chosen the CRC32 algorithm both for its ease of implementation and for its speed. It is ideal for detecting changes to short sequences of bytes; however, because there are only 232 possible checksum values, and because it is not cryptographically secure, the likelihood of a collision is high, giving the attacker a realistic chance to replace code without changing the checksum.   
  
我选择 CRC32 算法做例子是由于它易于实现且执行速度快。用它检测字节短序列比较理想；但由于只有 2 的 32 次方个可能的校验和值，而且不具密码安全性——碰撞可能性高，所以给了入侵者一个可实现的机会，不改变校验和而替换代码。  
  
Implementation of a checksum API in C  
  
Here, first of all, I included a classical implementation of a CRC32 API, which I found to be very compact and useful. This is an implementation of CRC32, which consists of macros for marking the start and end of the block to be checked, as well as a function to calculate the checksum of the block. The function crc32\_calc( ) is used to compute the checksum of a buffer.  
  
用 C 语言实现校验和 API  
  
首先，这里包括了 CRC32 API 的经典实现方法，我认为非常简洁有效。为实现 CRC32，我使用了宏——用于标记块的头尾以便检测，还有计算块的校验和的函数。crc32\_calc( )函数就用于计算缓冲区的校验和。  
  
#define CRC\_START\_BLOCK(label) void label(void) { }  
#define CRC\_END\_BLOCK(label) void \_##label(void) { }  
#define CRC\_BLOCK\_LEN(label) (int)\_##label - (int)label  
#define CRC\_BLOCK\_ADDR(label) (unsigned char \*)label  
  
static unsigned long crc32\_table\[256\] = {0};  
  
#define CRC\_TABLE\_LEN 256  
#define CRC\_POLY 0xEDB88320L   
  
static int crc32(unsigned long a, unsigned long b) {  
    int idx, prev;  
  
    prev = (a >> 8) & 0x00FFFFFF;  
    idx = (a ^ b) & 0xFF;  
    return (prev ^ crc32\_table\[idx\] ^ 0xFFFFFFFF);  
}  
  
static unsigned long crc32\_table\_init(void) {  
    int i, j;  
    unsigned long crc;  
  
     
    for (i = 0; i < CRC\_TABLE\_LEN; i++) {  
        crc = i;  
        for (j = 8; j > 0; j--) {  
            if (crc & 1) crc = (crc >> 1) ^ CRC\_POLY;  
            else crc >>= 1;  
        }  
        crc32\_table\[i\] = crc;  
    }  
    return 1;  
}   
  
unsigned long crc32\_calc(unsigned char \*buf, int buf\_len) {  
    int x;  
    unsigned long crc = 0xFFFFFFFF;  
  
    if (!crc32\_table\[0\]) crc32\_table\_init( );  
    for (x = 0; x < buf\_len; x++) crc = crc32(crc, buf\[x\]);  
    return crc;  
}  
  
The following program demonstrates the use of the checksum implementation. Note that the program is first compiled with a printf( ) in main( ) that will print the checksum to stdout. As long as main( ) is linked into the program after the buffer being checked, this printf( ) can be removed and the program recompiled without the value of the checksum changing. Once the checksum is known, a hex editor can be used to patch the checksum value into the location crc32\_stored. In this example, the four bytes of the checksum are stored between two 0xBAADF00D markers that should be overwritten with random bytes before the binary is distributed. Note that the markers will be stored in little-endian order in the binary, hence the reversed ordering of the bytes in the C source.  
  
以下程序演示了校验和的执行。注意程序首先编译了 main( ) 函数中的 printf( )，显示校验和到标准输出设备。检查缓冲区后，只要 main( )函数链接到程序，这个 printf( ) 函数会被移除，而且程序会在没有校验和变化值的情况下重新编译。一旦知道校验和，就使用十六进制编辑器将校验和值插入到 crc32\_stored 的位置。在这个例子中，四个字节的校验和分别存储在两个 0xBAADF00D 标志符之间，二进制码分配前这里会随机写满数据。注意标志符的二进制码是按从小到大的顺序存储，因此在 C 源程序中次序相反。  
  
#include <stdio.h>  
  
/\* warning: replace "crc32\_stored" with the real checksum! 警告：用校验和真值替换 crc32\_stored\*/  
asm(".long 0x0DF0ADBA \\n" /\* look for 0xBAADF00D markers 寻找 0xBAADF00D 标志符\*/  
    "crc32\_stored: \\n"  
    ".long 0xFFFFFFFF \\n" /\* change this in the binary! 用二进制码更改这里\*/  
    ".long 0x0DF0ADBA \\n" /\* end marker 结束标志符 \*/  
);  
  
CRC\_START\_BLOCK(test)  
int test\_routine(int a) {  
    while (a < 12) a = (a - (a \* 3)) + 1;  
return a;  
}  
CRC\_END\_BLOCK( test )  
  
int main(int argc, char \*argv\[ \]) {  
    unsigned long crc;  
  
    crc = crc32\_calc(CRC\_BLOCK\_ADDR(test), CRC\_BLOCK\_LEN(test));  
  #ifdef TEST\_BUILD  
    /\* This printf( ) displays the CRC value that needs to be stored in the program.  
     \* The printf( ) must be removed, and the program recompiled before distribution.  
     \* 此 printf( )函数程序需要存储的 CRC 值。  
     \* 此 printf( ) 函数必需移除，分配前程序会重新编译。\*/  
    printf("CRC is %08X\\n", crc);  
  #else  
    if (crc != crc32\_stored) {  
        printf("CRC32 %#08X does not match %#08X\\n", crc, crc32\_stored);  
        return 1;  
    }  
    printf("CRC32 %#08X is OK\\n", crc);  
  #endif  
return 0;  
}  
  
You should compile this program with TEST\_BUILD defined, then execute it to obtain the CRC value that needs to be replaced for crc32\_stored in the binary. Then rebuild the program with TEST\_BUILD undefined, and modify the binary with the proper CRC value from the first run. To calculate the CRC32 of a program you can use the CRC\_calculator given with this tutorial, just drag & drop the executable over to ge the CRC value.  
  
你应定义 TEST\_BUILD 来编译这个程序，然后执行并得到 CRC 的值，此值需要替代 crc32\_stored 内的二进制码。再用未定义的 TEST\_BUILD 重建程序，把二进制码修改成第一次运行所得到的正确 CRC 值。为计算程序 CRC32 值，你可以使用本教程附带的 CRC\_calculator 软件，只需拖放可执行文件到其界面上即可得到 CRC 值。  
  
It is tempting to generate a checksum of the entire program and use this to determine whether any bytes have been changed; however, this causes a loss of granularity for the checksum and can degrade performance. Instead, as explained in previous sections is always better to generate multiple checksums for vital sections of code. These checksums can be encrypted, and they can even be supplemented with checksums of trivial blocks of code to disguise which portions of code are significant.  
  
对整个程序生成校验和并以此确定是否有字节改动过，无疑非常吸引人；然而这会引起程序颗粒度（granularity ，即数据对象的相对大小的表达式，译者注）的损失，并使其性能下降。作为替代，如之前的所说那样，在重要代码段生成多重校验和的方法会好得多。加密这些校验和值，甚至可以补充一些无关紧要的代码块校验和来伪装成重要的代码。  
  
Notes about compiling the example1 with VisualC++ 6.0  
  
The code relative to this example is included into the example1 folder. If  you open it you might notice some differencies with what I just written above. These differencies are because of not complete compliance to standard C of Visual C++ 6.0. With respect to the code explained above, there are some differencies, just because the Visual C++ 6.0 I used doesn't support the standard asm C operator. I had to use instead the Microsoft specific \_\_asm operator which has some clearly documented limitations. First of all is not possible to directly define variables using the DB directive. I have to use instead the pseudoinstruction \_emit which force the compiler to emit the specified bytes, simulating so the DB directive. The ASM part is also modified to copy the address of the crc value to a local variable usable by the C code.  
To make things shorter I'll not include this code here too, look the file for further comments and clarifications.  
  
关于用 VisualC++ 6.0 编译例 1（example1）的注意事项  
  
example1 文件夹中有这个例子的源代码。如果你打开它，你也许会注意到它跟上面我所写的有些不同。这是由于 Visual C++ 6.0 与标准 C 并不完全一致。两者的一些差别仅仅因为 Visual C++ 6.0 不支持标准 C 的操作符 asm。我只能用 \_\_asm 关键字来代替 ，关于\_\_asm 的一些限制，微软文档有清楚的说明。首先，不能用 DB 伪指令直接定义变量。我改用伪指令\_emit 强制编译器放出指定的字节来模拟 DB。经修改 ASM 部分也使 C 代码可用，其作用是复制 CRC 值的地址到一个局部变量。  
限于篇幅，这里就不引用代码了。请直接查看该文件以获得更多注释和说明。  
  
To compile use "cl crc1.c /DTEST\_BUILD" or "cl crc1.c"   
   
Note that the programs are console based, they will just close and exit when double clicking them. You need to run them from within a DOS box, to see the printed messages..   
  
编译使用"cl crc1.c /DTEST\_BUILD" 或 "cl crc1.c"  
  
注意程序是基于控制台的，双击它就会关闭退出。你须在 DOS 窗口运行它以看到显示的信息。  
  
Looking the example1 into OllyDbg  
Open with ollydbg the crc1.exe file inside example1 folder of the present tutorial's archive. Hit CTRL-B to open the Binary Search dialog and search for 0xBAADF00D hex value. You should land here:  
  
用 OllyDbg 观察例 1   
用 OllyDbg 打开本教程附带 example1 文件夹中的 crc1.exe。按 CTRL-B 打开二进制字串查找框，查找 0xBAADF00D 的 HEX 值。你会来到这里：  
  
 I properly placed the comments to help you find the correspondence with the written code (refers to the modified code used for the example1 and not to the code above).  
  
我加了必要的注释帮助你找到程序代码的对应位置（引用的是 example1 修改后的程序代码，并非上文的代码）。  
  
Place a Breakpoint with F2 at the JMP at 0x0040114C and press F9 to reach the breakpoint. First of all if you step with F8 the program at 0x00401180 you can see that the real CRC just-in-time calculated is 0x6C8673E6 (it's in ECX).  
  
按 F2 在 0x0040114C 的 JMP 处下断，按 F9 来到断点。首先用 F8 单步步进，在 0x00401180 处可以看到即时计算的 CRC 真值是 0x6C8673E6（看 ECX 处）。  
  
Well you have the opportunity then to run again the application from the beginning and to stop again the application at 0x0040114C. Now go to location 0x401152 and select "Follow in dump" option to let you view in Ollydbg the memory at that location.  
  
好了，你有机会重头开始运行程序，然后再次停在 0x0040114C 处。来到 0x401152 处并右键选择"Follow in dump"（数据窗口中跟随，译者注），在 Ollydbg 中会看到此处的内存内容。  
  
Now insert the real CRC value you've just seen before (0x6C8673E6), remember to insert it as 0xE673866C.  
  
在此插入之前看过的 CRC 真值（0x6C8673E6），记得插入的是 0xE673866C。(注意堆栈后进先出的性质,低字节优先。译者注)  
  
Now save the modifications to a new executable and run from outside Ollydbg, the application runs perfectly. I saved this second executable into crc1\_ok.exe  
  
现在保存修改到新的 EXE 文件，然后脱离 Ollydbg 运行。程序运行正常。我把这个新的 EXE 文件保存为 crc1\_ok.exe。  
  
Weel, obviously the markers are not mandatory and are useful only to easily find the proper point where to insert the real CRC value, the whole routine can be more efficiently hidden and the check can be made less evident.  
  
当然了，例 1（example1）里的标志符并非必须，它只是让你更容易找到合适的位置去插入 CRC 真值。整个例程可以隐藏得更有效，而且检测也可能更隐蔽。  
  
Analizying a more more complex example, example2  
Generally speaking to fool the protection you only have to change the JE jump (opcode 0x74) at offset 0x401183 to a JMP instruction (opcode 0xEB). The generated checksum should never be checked against a valid one; instead, the generated checksum should be used as a source of information that the program requires to execute properly. A byte within the checksum could be used as an index into a jump table, for example, or the checksum itself could be used as a key to decrypt critical code or data.  
  
分析更为复杂得例子——例 2（example2)  
  
一般来说，要破解这个保护你只能用爆破——把偏移 0x401183 处的 JE(操作码 0x74) 跳转改为 JMP（操作码 0xEB）。程序生成的校验和从不与合法值比较，相反，它被作为一个必要的信息源使程序正常运行。例如，一个不带校验和的字节会作为索引进入跳转表，而校验和自身则是加密关键代码或数据的密钥。  
  
The next program demonstrates how to use a table of function pointers to test the value of a checksum. Each nibble or half-byte in the checksum is used as an index into a 16-entry table of function pointers; only the correct table entry calls the function to check the next nibble. This method requires 8 tables of 16 function pointers so that one table is used for each nibble in the checksum.  
  
下面的程序演示了怎样利用函数指针表来测试校验和值。校验和中每个半字节（nibble）会作为索引进入一张 16 项的函数指针表，只有正确的表项才会调用函数检查下一个半字节。这个办法需要 8 张每张含 16 个函数指针的表，使每张表对应一个校验和半字节。  
  
#include <stdio.h>  
     
CRC\_START\_BLOCK(test)  
int test\_routine(int a) {  
  while (a < 12) a = (a - (a \* 3)) + 1;  
  return a;  
}  
CRC\_END\_BLOCK(test)  
     
typedef void (\*crc\_check\_fn)(unsigned long \*);  
     
static void crc\_good (unsigned long \*crc);   
static void crc\_check(unsigned long \*crc);   
static void crc\_nib2 (unsigned long \*crc);   
static void crc\_nib3 (unsigned long \*crc);   
static void crc\_nib4 (unsigned long \*crc);   
static void crc\_nib5 (unsigned long \*crc);   
static void crc\_nib6 (unsigned long \*crc);   
static void crc\_nib7 (unsigned long \*crc);   
static void crc\_nib8 (unsigned long \*crc);  
     
crc\_check\_fn b1\[16\] = {0,}, b2\[16\] = {0,}, b3\[16\] = {0,}, b4\[16\] = {0,},  
             b5\[16\] = {0,}, b6\[16\] = {0,}, b7\[16\] = {0,}, b8\[16\] = {0,};  
     
#define CRC\_TABLE\_LOOKUP(table)             \\  
          int          index = \*crc & 0x0F; \\  
          crc\_check\_fn next = table\[index\]; \\  
          \*crc >>= 4;                       \\  
          (\*next)(crc)  
     
static void crc\_check(unsigned long \*crc) { CRC\_TABLE\_LOOKUP(b1); }  
static void crc\_nib2 (unsigned long \*crc) { CRC\_TABLE\_LOOKUP(b2); }  
static void crc\_nib3 (unsigned long \*crc) { CRC\_TABLE\_LOOKUP(b3); }  
static void crc\_nib4 (unsigned long \*crc) { CRC\_TABLE\_LOOKUP(b4); }  
static void crc\_nib5 (unsigned long \*crc) { CRC\_TABLE\_LOOKUP(b5); }  
static void crc\_nib6 (unsigned long \*crc) { CRC\_TABLE\_LOOKUP(b6); }  
static void crc\_nib7 (unsigned long \*crc) { CRC\_TABLE\_LOOKUP(b7); }  
static void crc\_nib8 (unsigned long \*crc) { CRC\_TABLE\_LOOKUP(b8); }  
     
static void crc\_good(unsigned long \*crc) {  
  printf("CRC is valid.\\n");  
}  
     
int main(int argc, char \*argv\[  \]) {  
  unsigned long crc;  
     
  crc = crc32\_calc(CRC\_BLOCK\_ADDR(test), CRC\_BLOCK\_LEN(test));  
#ifdef TEST\_BUILD  
  printf("CRC32 %#08X\\n", crc);  
#else  
  crc\_check(&crc);  
#endif  
  return 0;  
}  
  
When this program is compiled with TEST\_BUILD defined, the resulting binary will print the CRC32 computed for the function test\_routine( ). If the computed CRC32 is 0xFFF7FB7C, the following table indices will represent valid function pointers: b1\[12\], b2\[7\], b3\[11\], b4\[15\], b5\[7\], b6\[15\], b7\[15\], b8\[15\]. Each of these contains a pointer to the function that will process the next nibble in the checksum, except for b8\[15\], which contains a pointer to the function that is called when the checksum has proven valid. The tables in the source can now be rewritten to reflect these correct values:  
  
当程序与 TEST\_BUILD 定义一起编译，结果代码会打印计算后的 CRC32 的值，此值随后被 test\_routine( )函数使用。假如 CRC32 的计算值是 0xFFF7FB7C，从下面的表单索引会看到有效的函数指针：b1\[12\], b2\[7\], b3\[11\], b4\[15\], b5\[7\], b6\[15\], b7\[15\], b8\[15\]。除了 b8\[15\]外，其余每个指针都指向处理校验和内下一个半字节的函数。当证实校验和有效，b8\[15\]就指向程序真正调用的函数。  
  
  
crc\_check\_fn b1\[16\] = { 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, crc\_nib2, 0, 0, 0 }, /\*C\*/  
             b2\[16\] = { 0, 0, 0, 0, 0, 0, 0, crc\_nib3, 0, 0, 0, 0, 0, 0, 0, 0 }, /\*7\*/  
             b3\[16\] = { 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, crc\_nib4, 0, 0, 0, 0 }, /\*B\*/  
             b4\[16\] = { 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, crc\_nib5 }, /\*F\*/  
             b5\[16\] = { 0, 0, 0, 0, 0, 0, 0, crc\_nib6, 0, 0, 0, 0, 0, 0, 0, 0 }, /\*7\*/  
             b6\[16\] = { 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, crc\_nib7 }, /\*F\*/  
             b7\[16\] = { 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, crc\_nib8 }, /\*F\*/  
             b8\[16\] = { 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, crc\_good }; /\*F\*/  
  
Obviously, the NULL bytes will have to be replaced with other values to disguise the fact that they are invalid entries. They can be replaced with pointers to functions that handle incorrect checksums, or they can be filled with garbage values to make the program unstable. For example:  
  
很明显，NULL 字节须替换为其他值以掩饰它们是无效项。它们可以替换为指针，指向生成错误校验和的函数；也可以填入垃圾代码使程序不稳定。例如；  
  
crc\_check\_fn b8\[16\] = { crc\_good - 64, crc\_good - 60, crc\_good - 56, crc\_good - 52,  
                        crc\_good - 48, crc\_good - 44, crc\_good - 40, crc\_good - 36,  
                        crc\_good - 32, crc\_good - 28, crc\_good - 24, crc\_good - 20,  
                        crc\_good - 16, crc\_good - 12, crc\_good - 8,  crc\_good - 4,  
                        crc\_good };  
  
In this table, the use of incrementally increasing values causes the table to appear to be valid data, as opposed to addresses in the code segment.   
  
表中，与代码段地址相对，递增的数值看起来像是有效数据。  
  
Notes about compiling the example2 with VisualC++ 6.0  
  
This time the example doesn't have any specific modification to be compiled on VC++ 6.0. Follow the same commands as before to compile it.  
Into the example2 folder there are two sources, the first crc2\_step1.c must compiled using the TEST\_BUILD define, because has an unitialized crc\_check\_fn matrix. The crc2\_step2.c instead has this matrix correctly initialized.  
  
  
关于用 VisualC++ 6.0 编译例 2（example2）的注意事项  
  
这次的例子不需任何修改即可用 VC++ 6.0 编译。编译方式如常。  
在 example2 文件夹中有两份源程序，crc2\_step1.c 必须定义 TEST\_BUILD 来编译，因为 crc\_check\_fn 矩阵没有初始化。而 crc2\_step2.c 中这个矩阵已正确初始化。  
  
Looking the example2 into OllyDbg  
  
If you compile the program crc2\_step1.c using TEST\_BUILD you should get this value 0X6C8673E6. According to what I just said before the crc\_check\_fn matrix should becomes:  
  
用 OllyDbg 观察例 2  
  
如果使用 TEST\_BUILD 编译 crc2\_step1.c，你会得到 0X6C8673E6 这个值。据前文所述，crc\_check\_fn 矩阵应变成：  
  
crc\_check\_fn b1\[16\] = { 0, 0, 0, 0, 0, 0, crc\_nib2, 0, 0, 0, 0, 0, 0, 0, 0, 0 }, /\*6\*/  
             b2\[16\] = { 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, crc\_nib3, 0 }, /\*E\*/  
             b3\[16\] = { 0, 0, 0, crc\_nib4, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0 }, /\*3\*/  
             b4\[16\] = { 0, 0, 0, 0, 0, 0, 0, crc\_nib5, 0, 0, 0, 0, 0, 0, 0, 0 }, /\*7\*/  
             b5\[16\] = { 0, 0, 0, 0, 0, 0, crc\_nib6, 0, 0, 0, 0, 0, 0, 0, 0, 0 }, /\*6\*/  
             b6\[16\] = { 0, 0, 0, 0, 0, 0, 0, 0, crc\_nib7, 0, 0, 0, 0, 0, 0, 0 }, /\*8\*/  
             b7\[16\] = { 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, crc\_nib8, 0, 0, 0 }, /\*C\*/  
             b8\[16\] = { 0, 0, 0, 0, 0, 0, crc\_good, 0, 0, 0, 0, 0, 0, 0, 0, 0 }; /\*6\*/  
  
If you try to open the crc2\_step2 into Olly you can see that it is not a so trivial task to modify the code and still obtain a valid running program. Looking for the references to the string "CRC is valid" you should land at 0x004012C4. If you place a breakpoint with F2 and press run to the program with F9 you land to the correct place. Now if you press CTRL-K to see the call stack you should easily see that this subroutine is called from the   
  
如果尝试用 OllyDbg 打开 crc2\_step2，你会发觉修改代码并保持程序运行稳定并不是件很容易的事情。寻找字串参考“CRC is valid”，你会来到 0x004012C4 处。若 F2 下断后 F9 重新运行程序，你会到达正确地址。现在按 CTRL-K 查看调用堆栈，你应该很容易看到此子程序的调用来自  
  
004012BA |. FF55 F8 CALL \[LOCAL.2\] ; crc2\_ste.004012C4  
  
This local variable is a pointer to the selected address into the function pointer addresses contained into the b8\[16\] vector.  
  
此局部变量是个指针，指向函数指针地址中选中的地址，b8\[16\]向量包含了这个函数指针地址。  
  
Well, indeed there's still space for a more intelligent reversing of this application. We will see now how crypto algorithms can be identified using the constants they use. CRC32 is a very easy algorithm but also all the other more complex are not so complex..we'll see it in the next, last, section! Have a little more patient, we arriving at the end ^\_°  
  
事实上，这个程序仍有更具技巧的逆向空间。现在我们要知道怎样利用加密算法使用的常量来鉴别加密算法。CRC32 固然是非常简单的算法，但其他更复杂的算法也不见得会很难。我们将在下一节，也是本文最后一节看到它。请耐心一点，我们来到结尾了。  
   
7. Identifying self-checking algorithms into programs  
   
Breaking the example1 program  
To introduce you to the problem let approach to the program crc1\_ok.exe inside the example1 folder in the classical way. First of all be sure to have PEiD with the KryptoAnalizer (aka KANAL) installed. Run PEiD and select the crc1\_ok.exe program, then select plugins and run the plugin just mentioned.  
You should obtain (I assume you are alredy expert on PEiD, otherwise return to Beginner tut #1) something like below:  
  
7.鉴别程序的自校验算法  
  
破解例 1（example1）的程序  
以 example1 文件夹中的 crc1\_ok.exe 来说明这个问题是典型的做法。首先确认安装了 PEiD 及其插件 KryptoAnalizer（aka KANAL）。运行 PEiD 并选中 crc1\_ok.exe，然后选择 plugins 项并运行刚才提到的插件。  
你应该看到如下的画面（这里假设你已经很熟悉 PEiD，否则请回到初学者教程#1）：  
  
The plugin reports clearly an address at 0x004010DF. Well, fire up Olly and go there to see what's happening there..  
  
插件清楚显示了一个地址：0x004010DF。打开 OllyDbg 跟进去看看究竟是怎么回事。  
  
You landed here, to get the meaning of the place where you are move a little the cursors, to realign to the actual real code (you landed in the middle of an instruction, why it will be clear in a moment). Your Olly then should show something like below:  
  
你来到了这里。为了获得选中位置的含义，你只需稍稍移动光标，即可重新整理成实际代码（你在指令中间着陆,何以它马上变得清晰呢）。你的 OllyDbg 应该显示如下：  
  
Well, we were at the address of a constant, 0xEDB88320 which was part of an instruction XOR ECX,EDB88320  
  
好，这是常量的地址，0xEDB88320 就是 XOR ECX,EDB88320 指令的一部分。  
  
Now look the CRC32 implementation I did in Section 6, you can find this line:  
  
现在看看我在第六部分所讲的 CRC32 的实现，会找到这行：  
  
#define CRC\_POLY 0xEDB88320L  
  
This is the same constant KANAL plugin found. If you have the opportunity/time to see the sources of KANAL (freely available on PEiD site) you will realize that KANAL is doing nothing more than searching for specific constants in memory, any crypto algorithm (better hashing algorithm) uses some specific constants somewhere in the algorithm.   
  
KANAL 插件也找到这个同样的常量。如果你有机会/时间去看看 KANAL 的源代码（可在 PEiD 的网站上免费获取），就会了解 KANAL 只是搜索了内存中的特殊常量。任何加密算法（包括比散列算法更好的）都会在算法的某个位置使用了特殊的常量。  
  
At this stage we then understood we landed in the middle of CRC32 algorithm implementation. What we should do is to go at the entrypoint of the CRC32 routine  that is, in the example1 sources, the crc32\_calc() function.  
  
这个阶段，我们懂得自己来到了实现 CRC32 算法的代码段中。我们要做的就是进入 CRC32 程序的入口点——例 1 源代码的 crc32\_calc()函数。  
  
The function where we landed starts here:   
  
我们着陆的函数在此处开始：  
  
0040108F /$ 55 PUSH EBP  
  
and is called here:  
  
并且在此处被调用：  
  
00401016 |. E8 74000000 CALL crc1\_ok.0040108F  
  
which is inside a function which is called here:  
  
事实上它存在于函数内部，而函数在此处被调用：  
  
00401172 |. E8 89FEFFFF CALL crc1\_ok.00401000 ; \\crc1\_ok.00401000  
  
Now you arrived at destination. Look the code where you are now (at 0x00401172) and compare it with the code shown before in the debugging section of the example1, it's the same place.   
  
你现在来到了目的地。看看你所在位置（0x00401172）的代码，再对比一下调试例 1 那节所显示的代码，它们都在同样位置。  
  
Breaking the example2 program  
We are now using the method learnt for the easyer example1 with the more complex example2. Run KANAL on the file crc2\_step2.exe into the example2 folder. You should obtain something like this:  
  
破解例 2（example2）的程序  
现在我们用破解例 1 学到的方法来对付更复杂的例 2。将 example2 文件夹里的 crc2\_step2.exe 拖放到 KANAL。你会看到如下画面：  
  
As done before go to the address shown by KANAL, go up to the entry point of the CRC32 function and you finally land here:  
  
如之前所做一样，追入 KANAL 显示的地址，来到 CRC32 函数的入口点。最后在此处着陆：  
  
004012EA |. E8 11FDFFFF CALL crc2\_ste.00401000 ; \\crc2\_ste.00401000  
  
Well, place a breakpoint at this specific address and run the program using F9. Execute the call at 0x004012EA with F8, and you should have in EAX the real CRC32 value just calculated.  
  
好，在这个特殊的地址下断，然后按 F9 运行程序。按 F8 进入 0x004012EA 处的调用，然后你会从 EAX 得到刚才计算的 CRC32 真值。  
  
Well, here's how the program is compiled and I'll leave to you the opportunity to understand how to fix it when a change is done.  
  
下图展示了程序是如何被编译的。我现在给你一个机会，使你懂得如何在改动完成时去修复。  
  
This is the routine into which the CRC32 calculation is performed. I commented it with reference to the sources we wrote in Section 6.  
  
这是执行 CRC32 运算的例程。注释以第六部分的源代码作为参考。  
  
The crc\_check() function is called at:   
  
此处调用 crc\_check()函数：  
  
004012F9 |. E8 09000000 CALL crc2\_ste.00401307 ; \\call crc\_check(&crc)  
  
and it's body is the following one:  
  
主体部分如下：  
  
The function elaborates a bit the CRC32 value computed and uses it to call the next correct function here:  
  
函数巧妙地用 CRC32 的值计算了一个位（bit），并以它调用下一正确的函数：  
  
00401336 |. FF55 F8 CALL \[LOCAL.2\] ; call the function returned at address b1\[6\], where 6 is the value found at 00401312  
  
00401336 |. FF55 F8 CALL \[LOCAL.2\] ; 调用函数返回到 b1\[6\]的位置,  数字 6 是在 00401312 处得到的。  
  
If you step with F8 and follow this call, you will see that the called function is another one, almost identical . These functions are the functions responsible to extract from the b1\[\]; b8\[\] vectors the correct function pointers, which finally will lead you to the end of the program.  
  
如果用 F8 跟入这个调用，你会看到它调用了另一个几乎相同的函数。这些函数负责取出 b1\[\]的指针。b8\[\] 向量是正确的函数指针，它会引导你到程序的最后。  
  
For example the sequence of calls with a correct  CRC32 is:   
  
例如调用正确 CRC32 的顺序是：  
  
00401307 -> 00401135 -> 0040116E -> 004011A7 -> 004011E0 -> 00401219 -> 00401252 -> 0040128B -> 004012C4   
  
Think of this example mixed with an obfuscator or compressor like AsProtect or Armadillo with a more complex hashing algorithm than CRC32 and you will get how it would be easy to write less easily crackable programs.   
  
想象一下，对这个例子使用混淆代码和压缩程序的方法，例如加上 AsProtect 或 Armadillo 的壳，再用比 CRC32 更复杂的散列算法加密，你也能用简单的办法去写一个不易破解的程序。  
   
Conclusions  
   
What I shown here is a long journey on self-checking programs and the anti-tampering techniques programs uses. I also shown the theoretical assumptions that are behind these solutions and two practical examples. You should have realized how easy is to build custom protections for programs, that combined with known commercial protectors give extra barrires to crackers. Fortunately on the other hand much programmers do not ever consider them doing our cracking-lifes much of the time easy.  
  
结语  
  
这篇长长的文章介绍了自校验程序与反篡改技术。我也在理论上对这些技术作出假设的解决方法，并使用了两个实例。你应该知道构造常规的保护方法并不难，但假如与知名的商业保护手段结合的话，则会给破解者带来更大的麻烦。幸运的是，很多程序员甚至还没有考虑过这些技术，这给我们的破解生涯带来不少便利。  
   
References  
   
I suggest the following further readings from now on to complete this argument..  
  
O'Reilly - Secure Programming Cookbook for C and C++   
There a lot of tutorials on http://tutorials.accessroot.com which fool anti-tampering protections involving hashing.   
Intel - IA-32 Intel Architecture Software Developer's Manual, volume 3: System Programming Guide   
..and essentially all the tutorials seens around (also others on our tutorials page) which often target checksummed applications..  
  
参考资料  
  
我建议从现在起进一步阅读下面的资料，完善这方面的知识：  
O'Reilly - Secure Programming Cookbook for C and C++   
http://tutorials.accessroot.com 中有许多关于破解反篡改保护与散列密码的教程。  
Intel - IA-32 Intel Architecture Software Developer's Manual, volume 3: System Programming Guide   
基本上你能找到的以校验和程序为目标的全部教程（包括我们教程版面的其他文章）。  
  
  
Greetings  
   
Thanks to the whole ARTeam:  
\[Nilrem\] \[JDog45\] \[Shub - Nigurrath\] \[MaDMAn\_H3rCuL3s\] \[Ferrari\] \[Kruger\] \[Teerayoot\] \[R@dier\] \[ThunderPwr\] \[Eggi\] \[EJ12N\]   
\[Stickman 373\] \[Bone Enterprise\]   
  
Thanks to all the people who take time to write tutorials.   
Thanks to all the people who continue to develop better tools.   
Thanks to Exetools, Woodmann, SND, TSRH, MP2K and all the others for being a great place of learning.  
Thanks also to The Codebreakers Journal, and the Anticrack forum.  
  
If you have any suggestions, comments or corrections contact me in usual places..  
   
敬礼  
  
感谢 ARTeam 小组：  
\[Nilrem\] \[JDog45\] \[Shub - Nigurrath\] \[MaDMAn\_H3rCuL3s\] \[Ferrari\] \[Kruger\] \[Teerayoot\] \[R@dier\] \[ThunderPwr\] \[Eggi\] \[EJ12N\]   
\[Stickman 373\] \[Bone Enterprise\]   
  
感谢所有花费时间编写教程的朋友。  
感谢所有继续开发出更好工具的朋友。  
感谢 Exetools, Woodmann, SND, TSRH, MP2K 等等，它们都是非常棒的学习地点。  
同样感谢 Codebreakers Journal 与 Anticrack 论坛。  
  
如果您有任何建议, 意见或更正，请在老地方与我联系。  
  
  
  
  
  
  
roxhaiy 附注：本来我想做一个完全汉化的网页，但无奈做网页水平更低（唉……![](img/d23858dd221cce728a3a61b46badb044.jpg "恐怖!")![](img/d23858dd221cce728a3a61b46badb044.jpg "恐怖!") ） ，做了一半后重新打开，里面的中文全部变成问号，到现在也不知那里出错了…………  
为了不耽误制作电子书的时间（我已经耽搁得够久了），只有先把文章汉化好的部分图片打包上传，解压后全部覆盖到 images 文件夹即可。  
  
下载地址：http://bbs.pediy.com//attachment.php?s=&attachmentid=935